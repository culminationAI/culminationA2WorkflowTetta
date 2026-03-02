> [!TIP]
> Оптимизация запросов: Query Expansion, HyDE, декомпозиция. Дополнено примерами кода. Верифицировано 2026-02-26.

# Query Optimization: Мастерство поиска

Даже самая лучшая база данных не поможет, если запрос сформулирован плохо. Оптимизация запроса — это 50% успеха RAG.

## 1. Query Expansion (Расширение)
Пользователи часто задают неполные вопросы.
- **Multi-query**: Генерация вариаций. "Как работает RAG?" -> "Архитектура RAG", "Пайплайн извлечения и генерации", "Retrieval-Augmented Generation basics".
- **Synonym Expansion**: Автоматическое добавление терминов-синонимов.

```python
import anthropic

client = anthropic.Anthropic()

def expand_query(original_query: str, n_variants: int = 3) -> list[str]:
    """Генерация вариаций запроса через LLM для Multi-Query Retrieval."""
    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": (
                f"Сгенерируй {n_variants} разных формулировки этого вопроса "
                f"для поиска в базе знаний. Только формулировки, по одной на строку.\n\n"
                f"Вопрос: {original_query}"
            )
        }]
    )
    variants = response.content[0].text.strip().split("\n")
    return [original_query] + [v.strip() for v in variants if v.strip()]

# Пример
queries = expand_query("Как Иван относится к своему отцу?")
# ["Как Иван относится к своему отцу?",
#  "Отношения Ивана с отцом",
#  "Чувства Ивана к отцу и семейные конфликты",
#  "Влияние отца на поведение Ивана"]
```

## 2. HyDE (Hypothetical Document Embeddings)
Техника HyDE (Gao et al., 2022, [arXiv:2212.10496](https://arxiv.org/abs/2212.10496)), ставшая стандартом к 2026 году.
1.  LLM получает вопрос: "Как Иван победил дракона?".
2.  LLM генерирует «фейковый» ответ на основе своих общих знаний: "Иван использовал меч и магический щит...".
3.  Мы делаем векторный поиск, используя этот **ответ**, а не вопрос.
- **Почему это работает**: Вектор ответа и вектор документов в базе находятся ближе друг к другу, чем вектор вопроса.

```python
from sentence_transformers import SentenceTransformer
from qdrant_client import QdrantClient, models
import anthropic

e5 = SentenceTransformer("intfloat/multilingual-e5-large")
qdrant = QdrantClient(url="http://localhost:6333")
claude = anthropic.Anthropic()

def hyde_search(question: str, collection: str, top_k: int = 10):
    """HyDE: генерируем гипотетический ответ, ищем по его эмбеддингу."""
    # Шаг 1: LLM генерирует гипотетический ответ
    response = claude.messages.create(
        model="claude-sonnet-4-5-20250514",
        max_tokens=200,
        messages=[{
            "role": "user",
            "content": f"Напиши короткий фактический ответ на вопрос (2-3 предложения):\n{question}"
        }]
    )
    hypothetical_doc = response.content[0].text

    # Шаг 2: Эмбеддинг гипотетического ответа (passage, не query!)
    embedding = e5.encode(
        f"passage: {hypothetical_doc}",
        normalize_embeddings=True
    ).tolist()

    # Шаг 3: Поиск в Qdrant
    results = qdrant.query_points(
        collection_name=collection,
        query=embedding,
        limit=top_k
    )
    return results.points
```

## 3. Query Decomposition (Декомпозиция)
Сложный вопрос "Как политика компании повлияла на экологию и прибыль в 2025 году?" разбивается на подвопросы:
1.  "Какова политика компании по экологии?"
2.  "Какова была прибыль в 2025?"
3.  "Какие экологические штрафы были в 2025?"
Система ищет по каждому отдельно и синтезирует финальный ответ.

```python
def decompose_and_search(complex_query: str, collection: str):
    """Декомпозиция сложного запроса на подвопросы с раздельным поиском."""
    # Шаг 1: Декомпозиция
    response = claude.messages.create(
        model="claude-sonnet-4-5-20250514",
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": (
                "Разбей этот сложный вопрос на 2-4 простых подвопроса. "
                "Только подвопросы, по одному на строку.\n\n"
                f"Вопрос: {complex_query}"
            )
        }]
    )
    sub_queries = [q.strip() for q in response.content[0].text.strip().split("\n") if q.strip()]

    # Шаг 2: Поиск по каждому подвопросу
    all_results = {}
    for sq in sub_queries:
        embedding = e5.encode(f"query: {sq}", normalize_embeddings=True).tolist()
        results = qdrant.query_points(
            collection_name=collection,
            query=embedding,
            limit=5
        )
        all_results[sq] = results.points

    return all_results
```

## 4. Борьба с "Lost in the Middle"
Модели плохо помнят середину промпта (Liu et al., 2023, [arXiv:2307.03172](https://arxiv.org/abs/2307.03172)).
- **Context Filtering**: Удаление из извлеченных документов всего, что не относится к делу.
- **Prompt Reordering**: Помещение самых ценных фактов в самое начало или самый конец контекста.

```python
def reorder_for_llm(scored_docs: list[tuple[str, float]]) -> list[str]:
    """Переупорядочивание: лучшие документы в начало и конец, слабые — в середину.

    Решает проблему 'Lost in the Middle' (Liu et al., 2023).
    """
    sorted_docs = sorted(scored_docs, key=lambda x: x[1], reverse=True)
    texts = [doc for doc, _ in sorted_docs]

    if len(texts) <= 2:
        return texts

    # Чередование: лучший -> худший -> второй лучший -> ...
    reordered = []
    left, right = 0, len(texts) - 1
    toggle = True
    while left <= right:
        if toggle:
            reordered.append(texts[left])
            left += 1
        else:
            reordered.append(texts[right])
            right -= 1
        toggle = not toggle

    return reordered
```

---
*Следующая глава: [Оценка качества RAG](07-rag-evaluation-metrics.md)*

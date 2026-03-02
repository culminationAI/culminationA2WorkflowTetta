> [!TIP]
> Модульная архитектура RAG: компоненты, маршрутизация, чанкинг. Дополнено примерами кода. Верифицировано 2026-02-26.

# Modular RAG: Архитектура конструктора

В 2026 году RAG перестал быть линейным пайплайном. Современная система — это граф модулей, которые адаптируются под каждый запрос.

## 1. Компоненты Modular RAG
Система состоит из взаимозаменяемых блоков:

### A. Query Transformation (Преобразование)
Прежде чем искать, мы улучшаем запрос:
- **Multi-query**: Генерация 3-5 вариаций вопроса для расширения поиска.
- **HyDE (Hypothetical Document Embeddings)**: Техника из работы Gao et al. (2022, arXiv:2212.10496). LLM пишет гипотетический ответ, и мы ищем документы, похожие на этот ответ (часто работает лучше, чем поиск по вопросу).
- **Query Rewriting**: Очистка запроса от лишних слов.

```python
# Query Transformation: Multi-query + HyDE
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("intfloat/multilingual-e5-large")

def multi_query(original: str, llm) -> list[str]:
    """Генерация вариаций запроса через LLM."""
    prompt = f"Сгенерируй 3 перефразировки вопроса:\n{original}"
    variations = llm.generate(prompt)  # ["вариант 1", "вариант 2", "вариант 3"]
    return [original] + variations

def hyde_search(query: str, llm, client) -> list:
    """HyDE: поиск по гипотетическому ответу."""
    hypothetical = llm.generate(f"Напиши короткий ответ на: {query}")
    # Кодируем как passage (не query!), потому что ищем похожие документы
    vector = model.encode(f"passage: {hypothetical}", normalize_embeddings=True)
    return client.query_points(
        collection_name="memories",
        query=vector.tolist(),
        limit=10,
    )
```

### B. Routing (Маршрутизация)
Интеллектуальный диспетчер, который выбирает источник данных:
- **Vector DB**: Для неструктурированных знаний.
- **Graph DB**: Для анализа связей и цепочек событий.
- **SQL DB**: Для четких табличных данных.
- **Search API**: Для актуальной информации из веба.

### C. Logic/Agentic Loop
RAG внутри агента может работать итеративно:
1. `Retrieve` -> 2. `Evaluate` -> 3. `Is enough?` -> No -> 4. `Expand Query` -> Back to 1.

## 2. Преимущества модульности
- **Гибкость**: Можно заменить векторную базу (например, перейти с Pinecone на Qdrant) без переписывания всей логики.
- **Масштабируемость**: Каждый модуль можно оптимизировать отдельно (например, ускорить только процесс эмбеддинга).
- **Стоимость**: Мы не запускаем тяжелые графовые поиски там, где достаточно простого ключевого слова.

## 3. Практический кейс: Продвинутый Chunking
Модульный подход позволяет использовать разные стратегии нарезки данных:
- **Small-to-Big**: Храним мелкие куски (для поиска), ссылаемся на большие (для контекста).
- **Recursive Character Split**: Интеллектуальный сплиттер, который «дышит» вместе с текстом.

```python
# Small-to-Big Chunking с хранением parent_id
from qdrant_client.models import PointStruct
import uuid

def chunk_with_parent(text: str, chunk_size: int = 200, parent_size: int = 1000) -> list:
    """Нарезка: мелкие чанки для поиска, крупные для контекста."""
    parents = [text[i:i+parent_size] for i in range(0, len(text), parent_size)]
    points = []
    for parent_idx, parent in enumerate(parents):
        parent_id = str(uuid.uuid4())
        children = [parent[j:j+chunk_size] for j in range(0, len(parent), chunk_size)]
        for child in children:
            vector = model.encode(f"passage: {child}", normalize_embeddings=True)
            points.append(PointStruct(
                id=str(uuid.uuid4()),
                vector=vector.tolist(),
                payload={
                    "text": child,
                    "parent_text": parent,  # полный контекст
                    "parent_id": parent_id,
                },
            ))
    return points
```

---
*Следующая глава: [Концепции GraphRAG](03-graphrag-concepts.md)*

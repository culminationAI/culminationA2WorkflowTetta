> [!TIP]
> Эволюция RAG: от Naive к Advanced. Дополнено примерами кода. Верифицировано 2026-02-26.

# От Naive к Advanced RAG: Эволюция поиска

RAG (Retrieval-Augmented Generation) прошел три стадии развития. Понимание каждой из них критично для построения ультимативной системы.

## 1. Naive RAG (2023)
Классическая схема: **Indexing -> Retrieval -> Generation**.
- **Проблема**: Векторный поиск часто находит «похожие» по смыслу, но бесполезные для ответа куски текста.
- **Риски**: Высокий уровень галлюцинаций из-за шума в контексте.

```python
# Naive RAG: простой векторный поиск через Qdrant
from qdrant_client import QdrantClient
from sentence_transformers import SentenceTransformer

client = QdrantClient(url="http://localhost:6333")
model = SentenceTransformer("intfloat/multilingual-e5-large")

query_vector = model.encode("query: Что чувствует герой?", normalize_embeddings=True)

# Qdrant v1.17: query_points вместо deprecated search()
results = client.query_points(
    collection_name="memories",
    query=query_vector.tolist(),
    limit=5,
)
# results.points содержит найденные документы
context = "\n".join([p.payload["text"] for p in results.points])
```

## 2. Advanced RAG (2024-2025)
Вводит дополнительные этапы для повышения точности (Pre-retrieval и Post-retrieval).

### A. Hybrid Search (Золотой стандарт)
Комбинация векторного поиска (Semantic) и ключевых слов (BM25/Keyword).
- **Зачем**: Векторы понимают «смысл», но ключевые слова важны для поиска точных имен, артикулов и терминов.

### B. Re-ranking (Переранжирование)
После того как мы достали 100 «похожих» документов, мы используем специальную модель (Cross-Encoder, например BGE-Reranker), чтобы выбрать 5 по-настоящему лучших.
- **Результат**: Значительное повышение качества ответов.

```python
# Advanced RAG: двухэтапный поиск (retrieve + rerank)
from FlagEmbedding import FlagReranker

# Этап 1: широкий поиск (top-100)
results = client.query_points(
    collection_name="memories",
    query=query_vector.tolist(),
    limit=100,
)

# Этап 2: реранкинг через BGE-v2-m3
reranker = FlagReranker("BAAI/bge-reranker-v2-m3", use_fp16=True)
pairs = [[query_text, p.payload["text"]] for p in results.points]
scores = reranker.compute_score(pairs, normalize=True)

# Берём top-5 после реранкинга
ranked = sorted(zip(results.points, scores), key=lambda x: x[1], reverse=True)[:5]
```

### C. Context Optimization
- **Parent-Document Retrieval**: Ищем по мелким кускам (Chunk), но отдаем модели весь абзац (Parent), чтобы не терять контекст.
- **Semantic Splitting**: Разбиение текста не по количеству знаков, а по смысловым паузам.

## 3. Modular RAG (2025-2026)
Система превращается в набор независимых модулей (Routing, Rewrite, Retrieve, Rerank).
- **Search Routing**: Система понимает запрос и решает: "Для этого вопроса мне нужен Neo4j, а для этого -- Qdrant".
- **Self-Correction**: Агент видит, что извлеченная информация не полная, и делает новый запрос ("Multi-step retrieval").

```python
# Modular RAG: маршрутизация запроса к нужному источнику
def route_query(query: str) -> str:
    """Определяет источник данных по типу запроса."""
    # Паттерны для графовых запросов (связи, цепочки)
    graph_patterns = ["связан", "отношения", "через кого", "цепочка"]
    if any(p in query.lower() for p in graph_patterns):
        return "neo4j"
    return "qdrant"

# Пример: агентная петля с самокоррекцией
async def agentic_retrieve(query: str, max_rounds: int = 3) -> list:
    """Итеративный поиск с проверкой достаточности контекста."""
    context = []
    for round_num in range(max_rounds):
        source = route_query(query)
        if source == "neo4j":
            new_facts = await search_graph(query)
        else:
            new_facts = await search_vectors(query)
        context.extend(new_facts)
        if is_sufficient(context, query):  # LLM оценивает достаточность
            break
        query = expand_query(query, context)  # расширяем запрос
    return context
```

---
*Следующая глава: [Модульная архитектура RAG](02-modular-rag-architecture.md)*

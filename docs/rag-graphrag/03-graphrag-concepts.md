> [!TIP]
> GraphRAG: фундамент, Local/Global Search, DRIFT. Дополнено примерами Cypher и Python. Верифицировано 2026-02-26.

# GraphRAG: Фундамент и Поиск

GraphRAG — это "убийца" проблем классического векторного поиска. Он позволяет LLM понимать структуру знаний, а не только их близость.

## 1. Зачем нужен GraphRAG?
Обычный RAG страдает от:
- **Lost in the Middle**: В большом контексте LLM теряет детали.
- **Multi-hop blind spot**: Традиционный поиск не может связать "А знает Б" и "Б знает С" в одном запросе.
- **Global Understanding**: Векторный поиск не может ответить на вопрос "О чем вся эта база документов?".

## 2. Microsoft GraphRAG: Два режима поиска
Microsoft предложила революционную схему, ставшую стандартом в 2025 году.

### A. Local Search (Локальный поиск)
Фокусируется на конкретных сущностях.
- **Как работает**: Ищет ключевые узлы (например, "Компания X") и собирает всё их окружение (1-hop, 2-hop связи).
- **Итог**: Идеально для фактов о конкретных людях, местах или событиях.

```cypher
// Local Search: подграф вокруг героя (2 прыжка)
// Neo4j 2026: elementId() вместо deprecated id()
MATCH (hero:Hero {name: $hero_name})
CALL {
    WITH hero
    MATCH (hero)-[r1]-(n1)-[r2]-(n2)
    WHERE r1.weight >= 3.0
    RETURN n1, r1, n2, r2
    LIMIT 50
}
RETURN hero, n1, r1, n2, r2
```

### B. Global Search (Глобальный поиск)
Фокусируется на общей картине.
- **Как работает (Leiden Algorithm)**: Граф разбивается на «сообщества» (кластеры). LLM заранее пишет резюме для каждого сообщества. При поиске мы опрашиваем эти резюме.
- **Итог**: Ответы на вопросы типа "Какие главные темы обсуждались в отчетах за прошлый год?".

```cypher
// Global Search: кластеры сообществ через GDS (Louvain/Leiden)
// Шаг 1: построить проекцию графа
CALL gds.graph.project(
    'social-graph',
    ['Hero', 'NPC'],
    {
        KNOWS: { properties: 'weight' },
        TRUSTS: { properties: 'weight' }
    }
)

// Шаг 2: запустить Leiden для выявления сообществ
CALL gds.leiden.write('social-graph', {
    writeProperty: 'communityId',
    relationshipWeightProperty: 'weight'
})

// Шаг 3: извлечь резюме сообщества
MATCH (n) WHERE n.communityId = $community_id
RETURN labels(n) AS type, n.name AS name, n.summary AS summary
ORDER BY n.importance DESC
LIMIT 20
```

## 3. DRIFT Search (Продвинутый гибрид)
Новейшая техника (2025), которая динамически переключается между локальным и глобальным режимами в процессе рассуждения. Агент начинает с узкой детали и «уходит в глубину» или «поднимается на уровень выше» по мере необходимости.

## 4. Графовое преимущество (Graph Advantage)
- **Explainability**: Мы точно видим, почему агент пришел к такому ответу (через связи в Neo4j).
- **Data Integrity**: Мы можем наложить жесткие правила (Constraints) на граф, чтобы избежать галлюцинаций.

```python
# Python: извлечение подграфа для GraphRAG через Neo4j async driver
from neo4j import AsyncGraphDatabase

async def local_search(hero_name: str, depth: int = 2) -> dict:
    """Извлекает подграф вокруг героя для контекста LLM."""
    driver = AsyncGraphDatabase.driver(
        "bolt://localhost:7687",
        auth=("neo4j", "password"),
    )
    query = """
        MATCH path = (h:Hero {name: $name})-[*1..$depth]-(n)
        WHERE ALL(r IN relationships(path) WHERE r.weight >= 2.0)
        RETURN path
        LIMIT 100
    """
    async with driver.session() as session:
        result = await session.run(query, name=hero_name, depth=depth)
        records = [record async for record in result]
    await driver.close()
    return records
```

---
*Следующая глава: [Создание Графа Знаний](04-knowledge-graph-construction.md)*

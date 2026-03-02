> [!TIP]
> Построение графа знаний: экстракция, Entity Resolution, динамическая схема. Дополнено примерами кода. Верифицировано 2026-02-26.

# Knowledge Graph Construction: Извлечение знаний LLM

GraphRAG бесполезен без качественного графа. В этом разделе мы разберем «как» построить граф знаний из сырого текста, используя современные LLM-пайплайны.

## 1. Конвейер экстракции (The Extraction Pipeline)
Согласно паттернам 2026 года, процесс состоит из 4-х стадий:

### A. Intelligent Chunking
Текст бьется на куски, учитывая семантические границы. Каждый кусок должен содержать законченную мысль, чтобы LLM видела связи между сущностями.

### B. Entity & Relation Extraction (0-shot/Few-shot)
Мы подаем чанк в LLM с инструкцией: "Найди всех людей, места и события. Опиши, как они связаны".
- **Инновация 2025**: Использование **Discriminative Models** для быстрой разметки ENTITY и **Generative Models** для описания связей (RELATION).

### C. Relationship Bottleneck
Извлечение связей сложнее, чем имен.
- **Техника**: Chain-of-Thought Extraction. Сначала LLM перечисляет сущности, потом ищет связи между ними, потом проверяет их на логичность.

```python
# Экстракция сущностей и связей через LLM
import json

EXTRACTION_PROMPT = """Проанализируй текст и извлеки сущности и связи в JSON:
{
  "entities": [{"name": "...", "type": "Person|Location|Event|Emotion", "description": "..."}],
  "relations": [{"source": "...", "target": "...", "type": "...", "weight": 0.0-1.0}]
}

Текст: {text}"""

async def extract_triples(text: str, llm) -> dict:
    """Извлекает тройки (subject, relation, object) из текста."""
    response = await llm.generate(EXTRACTION_PROMPT.format(text=text))
    triples = json.loads(response)
    # Фильтруем по confidence
    triples["relations"] = [r for r in triples["relations"] if r["weight"] >= 0.7]
    return triples
```

```cypher
// Загрузка извлечённых троек в Neo4j
UNWIND $entities AS entity
MERGE (n {name: entity.name})
SET n:{type} = entity.type,
    n.description = entity.description,
    n.updated_at = datetime()

// Создание связей
UNWIND $relations AS rel
MATCH (a {name: rel.source}), (b {name: rel.target})
CALL apoc.merge.relationship(a, rel.type, {weight: rel.weight}, {}, b, {}) YIELD rel AS r
RETURN count(r)
```

## 2. Entity Resolution (Разрешение сущностей)
Это критический этап дедупликации.
- **Проблема**: В одном тексте написано "Гарри Поттер", в другом "Гарри", в третьем "Мальчик, который выжил".
- **Решение**: Семантическая группировка. Мы используем векторные эмбеддинги (E5/BGE), чтобы найти «похожие» имена, а затем просим LLM подтвердить: "Это один и тот же человек?".

```python
# Entity Resolution через эмбеддинги E5
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer("intfloat/multilingual-e5-large")

def find_duplicates(entities: list[str], threshold: float = 0.85) -> list[tuple]:
    """Находит потенциальные дубликаты среди извлечённых сущностей."""
    vectors = model.encode(
        [f"passage: {e}" for e in entities],
        normalize_embeddings=True,
    )
    # Cosine similarity через dot product (векторы нормализованы)
    sim_matrix = np.dot(vectors, vectors.T)
    duplicates = []
    for i in range(len(entities)):
        for j in range(i + 1, len(entities)):
            if sim_matrix[i][j] > threshold:
                duplicates.append((entities[i], entities[j], float(sim_matrix[i][j])))
    return duplicates

# Пример: find_duplicates(["Гарри Поттер", "Гарри", "Мальчик, который выжил"])
# -> [("Гарри Поттер", "Гарри", 0.91), ...]
```

## 3. Schema Inference (Динамическая схема)
В отличие от классических БД, современные GraphRAG системы могут расширять схему графа на лету. 
- Если LLM видит новый тип связи "ненавидит", она создает этот тип в графе без предварительного DDL.

## 4. Качество экстракции
- **Extraction Noise**: Избегайте сохранения случайных местоимений.
- **Confidence Scoring**: Каждой тройке (Subject-Relation-Object) присваивается балл уверенности. Данные с баллом ниже 0.7 не попадают в основной граф.

---
*Следующая глава: [Гибридные стратегии RAG](05-hybrid-rag-strategies.md)*

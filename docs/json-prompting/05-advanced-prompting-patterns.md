> [!TIP]
> Верифицировано 2026-02-26. Паттерны промптинга -- корректны, добавлен пример Pydantic v2.

# Advanced JSON Patterns: Умная структура

В 2026 году мы используем JSON не только для вывода, но и для того, чтобы заставить модель думать лучше. Вот паттерны, которые отличают профи от новичков.

## 1. Паттерн "Internal Reasoning" (CoT в JSON)
Обычный JSON заставляет модель отвечать сразу. Но сложные задачи требуют размышления.
**Решение**: Добавьте поле `"reasoning"` в начало схемы.
```json
{
  "reasoning": "Здесь ИИ подробно объясняет логику своего решения на 200 слов...",
  "answer": "Финальный короткий ответ для программы"
}
```
*Почему это работает*: Когда ИИ заполняет поле `reasoning`, он активирует свои механизмы логики (Chain-of-Thought). К моменту заполнения поля `answer` модель уже "разобралась" в задаче.

### Pydantic v2 реализация
```python
from pydantic import BaseModel, Field

class ReasonedAnswer(BaseModel):
    reasoning: str = Field(
        description="Подробное рассуждение модели (200+ слов)"
    )
    answer: str = Field(
        description="Финальный короткий ответ"
    )
    confidence: float = Field(
        ge=0.0, le=1.0,
        description="Уверенность модели в ответе (0.0 - 1.0)"
    )
```

## 2. Паттерн "Step-by-Step State"
Если вам нужно выполнить цепочку действий, опишите это как массив шагов.
```json
"steps": [
  { "step_num": 1, "action": "Поиск в БД", "result": "Найдено 3 записи" },
  { "step_num": 2, "action": "Синтез", "result": "Данные объединены" }
]
```

### Pydantic v2 реализация
```python
from pydantic import BaseModel, Field

class Step(BaseModel):
    step_num: int
    action: str
    result: str

class StepByStepPlan(BaseModel):
    goal: str = Field(description="Цель плана")
    steps: list[Step] = Field(description="Шаги выполнения")
```

## 3. Рекурсивный JSON (Дерево Мыслей)
Используйте вложенность для описания иерархий. Это идеально подходит для извлечения сущностей из текста:
```json
"entities": [
  {
    "name": "Apple",
    "type": "Company",
    "sub_entities": [ { "name": "iPhone", "type": "Product" } ]
  }
]
```

### Pydantic v2 реализация
```python
from __future__ import annotations
from pydantic import BaseModel

class Entity(BaseModel):
    name: str
    type: str
    sub_entities: list[Entity] = []

class ExtractionResult(BaseModel):
    entities: list[Entity]
```

## 4. Паттерн "Confidence Score"
Просите ИИ оценить свою уверенность в каждом поле.
```json
"date": "2024-05-20",
"date_confidence": 0.98
```
Если уверенность ниже 0.5, ваша программа может отправить этот результат на проверку человеку.

## 5. Few-shot JSON Examples
Хотя Structured Outputs работают хорошо, предоставление 1-2 примеров в промпте (в формате JSON) помогает модели лучше понять "стиль" заполнения полей (например, как сокращать слова или в каком формате писать время).

---
*Следующая глава: [Troubleshooting & Best Practices](06-troubleshooting-and-best-practices.md)*

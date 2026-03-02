> [!TIP]
> Верифицировано 2026-02-26. Исправлено: Claude не имеет JSON Mode -- использует output_config.format с JSON Schema.

# JSON Mode vs Structured Outputs: Конец эры галлюцинаций

Долгое время получение валидного JSON от ИИ было "игрой в рулетку". В 2026 году мы имеем технологии, которые гарантируют корректность на 100%.

## 1. JSON Mode (OpenAI)
Это режим, при котором модель старается выдавать валидный JSON.
- **Как это работает**: Вы устанавливаете `response_format: { "type": "json_object" }`, и OpenAI гарантирует синтаксически валидный JSON.
- **Проблема**: Она гарантирует, что это *какой-то* JSON, но не гарантирует, что в нем будут нужные вам поля. Модель всё еще может забыть поле `"price"` или вернуть `"true"` вместо `true`.

> **Claude (Anthropic) не имеет отдельного "JSON Mode"** как у OpenAI. Вместо этого Claude использует `output_config.format` с JSON Schema, что сразу обеспечивает и синтаксическую валидность, и соответствие схеме (эквивалент Structured Outputs). См. раздел 2a ниже.

## 2. Structured Outputs (Стандарт 2026)
Это качественный скачок. Теперь вы передаете модели **JSON Schema**.
- **Гарантия**: Модель физически не может сгенерировать токен, который не соответствует вашей схеме.
- **Как это достигнуто**: Провайдеры используют **Constrained Decoding** на уровне нейронной сети. Если по схеме ожидается число, сеть даже не будет рассматривать буквы как варианты для следующего токена.

### 2a. Structured Outputs в Claude (Anthropic)
Claude поддерживает два способа гарантированного JSON:

**Способ 1: `output_config.format` (рекомендуется)**
```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Опиши персонажа"}],
    output_config={
        "format": {
            "type": "json_schema",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "age": {"type": "integer"},
                    "traits": {
                        "type": "array",
                        "items": {"type": "string"}
                    }
                },
                "required": ["name", "age", "traits"],
                "additionalProperties": False
            }
        }
    }
)

import json
result = json.loads(response.content[0].text)
```

**Способ 2: Strict Tool Use**
```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    tool_choice={"type": "tool", "name": "extract_data"},
    tools=[{
        "name": "extract_data",
        "description": "Извлеки данные о персонаже",
        "strict": True,
        "input_schema": {
            "type": "object",
            "properties": {
                "name": {"type": "string"},
                "age": {"type": "integer"}
            },
            "required": ["name", "age"],
            "additionalProperties": False
        }
    }],
    messages=[{"role": "user", "content": "Иван, 34 года, инженер"}]
)

result = response.content[0].input  # {"name": "Иван", "age": 34}
```

### 2b. Structured Outputs в OpenAI
```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "character",
            "strict": True,
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "age": {"type": "integer"}
                },
                "required": ["name", "age"],
                "additionalProperties": False
            }
        }
    },
    messages=[{"role": "user", "content": "Опиши персонажа Иван, 34 года"}]
)
```

## 3. Сравнение технологий
| Характеристика | JSON Mode (OpenAI) | Structured Outputs (OpenAI) | Claude output_config | Claude Strict Tools |
| :--- | :--- | :--- | :--- | :--- |
| **Синтаксис** | Гарантирован | Гарантирован | Гарантирован | Гарантирован |
| **Наличие полей** | Нет | **Гарантировано** | **Гарантировано** | **Гарантировано** |
| **Типы данных** | Нет | **Гарантировано** | **Гарантировано** | **Гарантировано** |
| **Надежность** | ~95% | **100%** | **100%** | **100%** |
| **Сложность** | Низкая | Средняя | Средняя | Средняя |

## 4. Когда что использовать?
- **JSON Mode (OpenAI)**: Быстрые прототипы или когда схема постоянно меняется.
- **Structured Outputs**: Единственный выбор для production-систем, где ошибки парсинга недопустимы.
- **Claude output_config.format**: Основной способ для Claude -- гарантированный JSON по схеме.
- **Claude Strict Tool Use**: Когда нужен вызов функций с гарантированной схемой входных параметров.

## 5. Влияние на промпт
При использовании Structured Outputs вам больше не нужно писать в промпте "be careful with commas" или "don't include markdown". Модель просто не сможет выдать ничего лишнего. Это экономит кучу места в контекстном окне.

---
*Следующая глава: [Мастерство JSON Schema](03-json-schema-mastery.md)*

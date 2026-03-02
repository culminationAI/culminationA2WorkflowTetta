> [!TIP]
> Верифицировано 2026-02-26. Исправлено: Pydantic v2 (@field_validator), Instructor (from_openai).

# Pydantic & Instructor: Python-путь к ИИ

Писать RAW JSON схемы -- это долго и неудобно. В 2026 году стандарт для Python-разработчиков -- это **Pydantic v2** в связке с библиотекой **Instructor**.

## 1. Что такое Pydantic?
Это самая популярная библиотека для валидации данных в Python. Вы описываете структуру данных как обычный класс:
```python
from pydantic import BaseModel, Field

class UserProfile(BaseModel):
    name: str = Field(description="Полное имя пользователя")
    age: int = Field(ge=18, description="Возраст (минимум 18)")
```

## 2. Магия Instructor
**Instructor** -- это библиотека, которая "наклеивает" ваш Pydantic-класс прямо на вызов LLM.
```python
import instructor
from openai import OpenAI

# Pydantic v2 + Instructor (актуальный API)
client = instructor.from_openai(OpenAI())

user = client.chat.completions.create(
    model="gpt-4o",
    response_model=UserProfile,  # Модель ИИ вернет сразу объект UserProfile!
    messages=[{"role": "user", "content": "Иван, 25 лет"}]
)

print(user.name)  # Работает автодополнение в IDE!
```

### Instructor для Anthropic Claude
```python
import instructor
import anthropic

client = instructor.from_anthropic(anthropic.Anthropic())

user = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    response_model=UserProfile,
    messages=[{"role": "user", "content": "Мария, 30 лет"}]
)
```

> **Важно:** `instructor.patch()` -- deprecated. Используйте `instructor.from_openai()`, `instructor.from_anthropic()` или `instructor.from_provider()` в зависимости от провайдера.

## 3. Почему это круто?
- **IDE Support**: Вы получаете автодополнение и проверку типов в VS Code или PyCharm.
- **Auto-Retry**: Если ИИ ошибся и выдал невалидный JSON, Instructor может автоматически отправить ошибку обратно модели: "Ты ошибся в поле age, это должна быть строка, а не число. Попробуй еще раз".
- **Documentation**: Поля `Field(description=...)` автоматически превращаются в `description` в JSON Schema, которую видит ИИ.

## 4. Кастомная валидация (Pydantic v2)
В Pydantic v2 используется `@field_validator` вместо устаревшего `@validator`:
```python
from pydantic import BaseModel, field_validator

class UserProfile(BaseModel):
    name: str
    age: int

    @field_validator("name")
    @classmethod
    def name_must_be_capitalized(cls, v: str) -> str:
        if not v[0].isupper():
            raise ValueError("Имя должно начинаться с заглавной буквы")
        return v
```
Если ИИ выдаст "иван", валидатор выбросит ошибку, и Instructor попросит модель исправить это.

### Валидация нескольких полей (model_validator)
```python
from pydantic import BaseModel, model_validator

class DateRange(BaseModel):
    start: str
    end: str

    @model_validator(mode="after")
    def check_dates(self) -> "DateRange":
        if self.start > self.end:
            raise ValueError("start должен быть раньше end")
        return self
```

> **Миграция Pydantic v1 -> v2:**
> - `@validator("field")` -> `@field_validator("field")` + `@classmethod`
> - `@root_validator` -> `@model_validator(mode="before"|"after")`
> - `class Config:` -> `model_config = ConfigDict(...)`
> - `.dict()` -> `.model_dump()`
> - `.json()` -> `.model_dump_json()`

## 5. Другие языки
- **TypeScript**: Используйте **Zod** -- это аналог Pydantic для мира JS/TS.
- **Gemini**: Имеет встроенную поддержку типизации через Python-классы (Type Hints).

---
*Следующая глава: [Advanced Prompting Patterns](05-advanced-prompting-patterns.md)*

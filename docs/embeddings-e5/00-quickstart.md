> [!CAUTION]
> Создано Manus/Gemini без верификации. Известные проблемы:
> - Размер модели "1.2 GB" неточен (~2.24 GB fp32, ~1.1 GB fp16)

# E5 Эмбеддинги: Быстрый старт

Этот гайд поможет тебе запустить и протестировать модель `multilingual-e5-large` за 5 минут.

## 1. Установка
Тебе понадобится Python 3.9+ и библиотека `sentence-transformers`.

```bash
pip install sentence-transformers torch
```

## 2. Твой первый код
Создай файл `test_e5.py` и вставь туда этот код:

```python
from sentence_transformers import SentenceTransformer

# 1. Загрузка модели (вес около 1.2 GB)
model = SentenceTransformer('intfloat/multilingual-e5-large')

# 2. Подготовка данных с префиксами
# ВАЖНО: всегда добавляй 'query:' для вопросов и 'passage:' для документов
docs = [
    "passage: Кошки любят спать на солнце.",
    "passage: Графовые базы данных хранят связи между узлами."
]
query = "query: Где спят коты?"

# 3. Генерация векторов
doc_embeddings = model.encode(docs, normalize_embeddings=True)
query_embedding = model.encode(query, normalize_embeddings=True)

# 4. Проверка схожести (dot product для нормализованных векторов = cosine similarity)
import torch
similarities = torch.matmul(torch.tensor(query_embedding), torch.tensor(doc_embeddings).T)

print(f"Схожесть с первым домом: {similarities[0]:.4f}")
print(f"Схожесть со вторым домом: {similarities[1]:.4f}")
```

## 3. Что ты только что сделал?
1.  **Загрузил модель**: Она превращает слова в числа (вектор из 1024 чисел).
2.  **Использовал префиксы**: Это "переключает" режим работы модели (поиск vs хранение).
3.  **Нормализовал векторы**: Теперь их можно сравнивать по углу (косинусное расстояние).

## 4. Следующие шаги
- [ ] Прочитай о [префиксах](02-prefixes-and-usage.md) для продвинутого поиска.
- [ ] Узнай, как [оптимизировать](03-optimizations.md) модель для медленного железа.
- [ ] Посмотри, как настроить [Qdrant](../qdrant/index.md) для этих векторов.

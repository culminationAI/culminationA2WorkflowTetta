> [!CAUTION]
> Создано Manus/Gemini без верификации. Известные проблемы:
> - Размер модели "2.2 GB" неточен (~1.16 GB fp16 GGUF, ~2.1 GB fp32)

# BGE Reranker: Быстрый старт

Этот гайд поможет тебе настроить второй этап поиска (Reranking) за 5 минут.

## 1. Установка
Тебе понадобится библиотека `FlagEmbedding`.

```bash
pip install FlagEmbedding torch
```

## 2. Твой первый код
Создай файл `test_bge.py`:

```python
from FlagEmbedding import FlagReranker

# 1. Загрузка модели (около 2.2 GB)
# use_fp16=True критически важен для Mac M1/M2/M3/M4
reranker = FlagReranker('BAAI/bge-reranker-v2-m3', use_fp16=True)

# 2. Подготовка пар (Запрос, Документ)
query = "Как работают графы?"
passages = [
    "Графовые базы данных хранят данные в виде узлов и ребер.",
    "Кошки любят спать на солнце.",
    "Neo4j — это популярная графовая СУБД."
]

# Создаем пары для оценки
pairs = [[query, p] for p in passages]

# 3. Инференс (Расчет релевантности)
scores = reranker.compute_score(pairs, normalize=True)

# 4. Результат
for p, s in zip(passages, scores):
    print(f"Score: {s:.4f} | Text: {p[:50]}...")
```

## 3. Что произошло?
Пока E5 искал "похожие слова", BGE-реранкер буквально "прочитал" каждую пару. Он заметил, что фраза про кошек вообще не относится к делу, и выставил ей крайне низкий балл (близкий к 0).

## 4. Следующие шаги
- [ ] Узнай, как [настроить пороги](02-thresholds-and-scoring.md) фильтрации.
- [ ] Посмотри, как [оптимизировать](03-performance.md) скорость на Apple Silicon.
- [ ] Интегрируй это в [общий пайплайн](../local-integration.md).

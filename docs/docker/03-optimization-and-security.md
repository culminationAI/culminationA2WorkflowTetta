> [!TIP]
> Создано Manus/Gemini, верифицировано и исправлено Claude (2026-02-26).

# Docker: Оптимизация и Безопасность

Чтобы проект летал на Mac M4 и был защищен в продакшене, следуй этим правилам.

## 1. Multi-Stage Builds (Магия Маленьких Образов)
Мы собираем приложение в два этапа: на первом ставим всё "тяжелое", на втором — оставляем только готовый код.

```dockerfile
# Stage 1: Build
FROM python:3.11-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim as runtime
WORKDIR /app
# Копируем установленные пакеты из первого этапа
COPY --from=builder /root/.local /root/.local
COPY . .
# Обновляем PATH
ENV PATH=/root/.local/bin:$PATH

CMD ["python", "main.py"]
```
- **Результат**: Образ весит в 3-5 раз меньше. Нет лишних компиляторов и мусора.

## 2. Секреты производительности на Mac M4
Если ты работаешь на Apple Silicon, Docker Desktop требует настройки:

1.  **VirtioFS**: Включи в настройках. Это ускоряет чтение файлов с твоего диска (например, если ты правишь код и он сразу синхронизируется в контейнер).
2.  **Rosetta for x86/amd64**: Если тебе нужно запустить образ, у которого нет версии под ARM, включи поддержку Rosetta в настройках Docker. Но старайся всегда искать `arm64` версии.
3.  **VMM (Virtual Machine Manager)**: В Docker Desktop 4.35+ выбери "Docker VMM". Это самая быстрая реализация на сегодня.

## 3. Безопасность (Non-Root User)
По умолчанию Docker запускает всё от имени `root`. Это опасно.
**Лучшая практика:**
```dockerfile
RUN useradd -m myuser
USER myuser
```
Теперь, даже если злоумышленник взломает твоё приложение, он не получит прав суперпользователя на твоей машине.

## 4. Чек-лист перед запуском
- [ ] Использован `.dockerignore` (исключи `.git`, `__pycache__`, `venv`).
- [ ] Все зависимости закреплены версиями в `requirements.txt`.
- [ ] Секретные ключи (API Keys) передаются через `environment`, а не хранятся в Dockerfile.

---
*Поздравляем! Ты освоил Docker на уровне Ultimate Reference.*

> [!TIP]
> Создано Manus/Gemini, верифицировано и исправлено Claude (2026-02-26).
> Исправлено: убран deprecated `version`, `neo4j:enterprise` заменён на `community`, healthcheck через `cypher-shell`.

# Docker Compose: Оркестрация стека

Docker Compose позволяет запускать Neo4j, Qdrant и твоё приложение одной командой, настраивая связи между ними автоматически.

## 1. Файл docker-compose.yml

Эталонный конфиг для проекта. Учитывает зависимости и проверки готовности (healthchecks).

> **Важно:** Поле `version` убрано намеренно. Docker Compose v2 (начиная с v2.27.0) его не требует и выдаёт предупреждение, если оно присутствует. Compose-файл описывается [Compose Specification](https://docs.docker.com/compose/compose-file/), а не устаревшими версиями формата.

```yaml
services:
  # Векторная БД Qdrant
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_data:/qdrant/storage
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:6333/healthz"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Графовая БД Neo4j
  neo4j:
    image: neo4j:community
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      - NEO4J_AUTH=neo4j/password
      - NEO4J_PLUGINS=["apoc"]
    volumes:
      - neo4j_data:/data
    healthcheck:
      test: ["CMD", "cypher-shell", "-u", "neo4j", "-p", "password", "RETURN 1"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Наше Python Приложение
  app:
    build: .
    depends_on:
      qdrant:
        condition: service_healthy
      neo4j:
        condition: service_healthy
    environment:
      - QDRANT_HOST=qdrant
      - NEO4J_URI=bolt://neo4j:7687

volumes:
  neo4j_data:
  qdrant_data:
```

### Что изменилось по сравнению с исходной версией

| Было | Стало | Почему |
| :--- | :--- | :--- |
| `version: '3.8'` | убрано | Deprecated в Compose v2.27+, не нужен |
| `neo4j:5.12-enterprise` | `neo4j:community` | Enterprise требует лицензию; для MVP и разработки используем Community |
| `neo4j status` (healthcheck) | `cypher-shell "RETURN 1"` | `neo4j status` проверяет процесс, а не готовность БД к запросам |
| `./qdrant_data` (bind mount) | `qdrant_data` (named volume) | Named volumes управляются Docker, проще переносить и чистить |
| Порт `6334` не проброшен | Проброшен `6334:6334` | gRPC-порт Qdrant нужен для Python-клиента |
| `NEO4J_PLUGINS=["apoc", "gds"]` | `NEO4J_PLUGINS=["apoc"]` | GDS недоступен в Community Edition |

## 2. Основные команды Compose

| Команда | Действие |
| :--- | :--- |
| `docker compose up -d` | Запустить всё в фоновом режиме |
| `docker compose down` | Остановить и удалить контейнеры (volumes сохраняются) |
| `docker compose down -v` | Остановить и удалить контейнеры вместе с volumes |
| `docker compose ps` | Статус всех сервисов в стеке |
| `docker compose logs -f app` | Следить за логами приложения |
| `docker compose build --no-cache` | Пересобрать образы с нуля |

## 3. Почему `depends_on` недостаточно?

Обычный `depends_on` просто ждёт запуска контейнера. Но база данных может запускаться 30 секунд.
- Мы используем **condition: service_healthy**.
- Приложение `app` начнёт запускаться только тогда, когда Neo4j и Qdrant ответят: "Я готов принимать запросы".
- Healthcheck Neo4j через `cypher-shell` гарантирует, что Bolt-протокол (порт 7687) действительно принимает соединения. Команда `neo4j status` лишь проверяет, что процесс запущен, но не что БД готова к запросам.

---
*Дополнительно: [Оптимизация и безопасность](03-optimization-and-security.md)*

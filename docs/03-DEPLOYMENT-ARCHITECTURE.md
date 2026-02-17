# Архитектура развёртывания

**Навигация:** [README](../README.md) | [Меню](00-INDEX.md) | [← Назад](02-TECH-STACK.md) | [Далее →](04-SYNC-FLOW.md)

---

Диаграмма и описание компонентов.

---

## Диаграмма развёртывания

```mermaid
flowchart TB
    subgraph Client [Client Layer]
        API[Rails API]
    end

    subgraph Orchestration [Orchestration]
        Temporal[Temporal Server]
        Workers[Temporal Workers]
    end

    subgraph Data [Data Layer]
        PG[(PostgreSQL)]
        Redis[(Redis)]
    end

    subgraph Extract [Extract - дорого]
        Crawlers[Ruby Crawler Gems]
        CrawlersNote[Instagram, YouTube, TikTok]
    end

    subgraph Transform [Transform]
        Local[Validate, Normalize, Types]
        External[ML / ImageRecognition API]
    end

    subgraph Observability [Observability]
        OTEL[OpenTelemetry Collector]
        Grafana[Grafana]
        NewRelic[NewRelic]
    end

    API --> Temporal
    Workers --> Temporal
    Workers --> PG
    Workers --> Crawlers
    Workers --> Local
    Workers --> External
    Workers --> OTEL
    OTEL --> Grafana
    OTEL --> NewRelic
```

---

## Extract — самый дорогой этап

Данные из соцсетей получать **дороже всего**:

- **Постраничная загрузка** — всё кроме профилей (посты, подписчики, лайки, комменты)
- **Долго** — в среднем ~1000 запросов на профиль; бывает на 2 порядка больше или меньше (крайне редко, чаще близко к середине)
- **Платно** — API limits, rate limits

Поэтому Extract выделен отдельно и требует:

- Батчинг
- Rate limiting по сети
- Курсоры (since_id, updated_since) для докачки
- Checkpoints для рестарта

---

## Компоненты

| Компонент | Назначение |
|-----------|------------|
| **Rails API** | Точка входа, приём запросов на синк |
| **Temporal Server** | Оркестрация workflow (при варианте A) |
| **Temporal Workers** | Выполнение Activity (Extract, Transform, Load) |
| **PostgreSQL** | Данные, job_calls, service_calls | 
| **Redis** | Sidekiq / Temporal (опционально) |
| **Crawler Gems** | Extract из Instagram, YouTube, TikTok |
| **Transform (локально)** | Валидация, нормализация, приведение типов |
| **Transform (внешне)** | ML, ImageRecognition — HTTP API, polling |
| **OpenTelemetry** | Трейсинг, метрики |
| **Grafana / NewRelic** | Дашборды, алерты |

---

## Вариант без Temporal (Sidekiq Flow)

При выборе варианта B:

- Temporal Server и Workers заменяются на **Sidekiq** + **Sidekiq Flow**
- Redis — обязателен для Sidekiq
- Polling-service — cron или Sidekiq scheduled job для опроса ML/ImageRecognition
- Остальная схема сохраняется

---

## Инфраструктура

- **Kubernetes** — деплой
- **Docker** — образы приложений
- **Yandex Cloud** — хостинг

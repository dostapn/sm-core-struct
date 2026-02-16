# Технологический стек

**Навигация:** [00](00-INDEX.md) | [01](01-ORCHESTRATOR-CHOICE.md) | [02](02-TECH-STACK.md) | [03](03-DEPLOYMENT-ARCHITECTURE.md) | [04](04-SYNC-FLOW.md) | [05](05-DATA-CONTRACTS.md) | [06](06-OBSERVABILITY.md) | [07](07-RESILIENCE-RESTART.md) | [← Назад](01-ORCHESTRATOR-CHOICE.md) | [Далее →](03-DEPLOYMENT-ARCHITECTURE.md)

---

Варианты сервисных решений под разные оркестраторы и графы взаимодействия систем.

---

## Граф взаимодействия (общий)

```mermaid
flowchart TB
    subgraph Client [Client]
        API[Rails API]
    end

    subgraph Orchestrator [Orchestrator]
        T[Temporal / Sidekiq Flow / Cron]
    end

    subgraph Workers [Workers]
        W[Ruby Workers]
    end

    subgraph Extract [Extract]
        C[Crawler Gems]
    end

    subgraph Transform [Transform]
        L[Validate, Normalize]
        M[ML / ImageRecognition API]
    end

    subgraph Polling [Polling Service]
        P[Cron / Job]
    end

    subgraph Storage [Storage]
        PG[(PostgreSQL)]
        Redis[(Redis)]
    end

    API --> T
    T --> W
    W --> C
    W --> L
    W --> M
    M -.->|async, job_id| P
    P -.->|poll status| M
    W --> PG
    W --> Redis
```

---

## Граф взаимодействия (вариант A: Temporal)

```mermaid
sequenceDiagram
    participant API as Rails API
    participant Temporal as Temporal Server
    participant Worker as Temporal Worker
    participant Crawler as Crawler Gems
    participant ML as ML/ImageRecognition
    participant PG as PostgreSQL

    API->>Temporal: Start Workflow
    Temporal->>Worker: Schedule Activity
    Worker->>Crawler: Extract
    Crawler-->>Worker: Raw data
    Worker->>PG: write_to_raw
    Worker->>ML: Submit job (async)
    ML-->>Worker: job_id
    loop Polling
        Worker->>ML: GET /jobs/{id}
        ML-->>Worker: status (pending/done)
    end
    Worker->>PG: write_to_normalized
    Worker-->>Temporal: Activity done
```

---

## Граф взаимодействия (вариант B: Sidekiq Flow)

```mermaid
flowchart LR
    Cron[Cron] --> Sidekiq[Sidekiq]
    Sidekiq --> Flow[Sidekiq Flow DAG]
    Flow --> Job1[fetch_profile]
    Flow --> Job2[fetch_posts]
    Flow --> Job3[Transform]
    Job1 --> Redis[(Redis)]
    Job2 --> Redis
    Job3 --> PG[(PostgreSQL)]
    Polling[Cron: poll ML] --> Job3
```

---

## Вариант A: Temporal + Rails

| Компонент | Технология |
|-----------|------------|
| **Оркестратор** | Temporal (self-hosted) |
| **ЯП / фреймворк** | Ruby 4, Rails 8 |
| **Workers** | Temporal Workers (Ruby SDK) |
| **Краулеры** | Локальные гемы (Instagram, YouTube, TikTok) |
| **Transform (локально)** | Rails, валидация, нормализация |
| **Transform (внешне)** | ML/ImageRecognition — HTTP API, polling |
| **Очереди / хранилище** | PostgreSQL (Temporal + данные), Redis (опционально) |
| **Observability** | OpenTelemetry, Grafana, NewRelic |

**Оркестратор во главе:** да. Temporal управляет workflow, Activity вызывают краулеры и Transform.

---

## Вариант B: Sidekiq Flow + polling-service

| Компонент | Технология |
|-----------|------------|
| **Оркестратор** | Sidekiq Flow (DAG) |
| **ЯП / фреймворк** | Ruby 4, Rails 8 |
| **Workers** | Sidekiq workers |
| **Краулеры** | Локальные гемы (Instagram, YouTube, TikTok) |
| **Transform** | Rails + внешние ML/ImageRecognition (API, polling) |
| **Polling** | Cron или Sidekiq scheduled job — опрос результатов (1 мин — несколько дней) |
| **Очереди / хранилище** | Redis (Sidekiq), PostgreSQL (данные, sync_jobs) |
| **Observability** | OpenTelemetry, Grafana, NewRelic |

**Оркестратор во главе:** Sidekiq Flow — DAG. Checkpoints и resume — вручную (sync_jobs, cursor).

---

## Вариант C: Общий (оркестратор-агностик)

| Компонент | Технология |
|-----------|------------|
| **Оркестратор** | Не выбран (Cron + Sidekiq как минимум) |
| **ЯП / фреймворк** | Ruby 4, Rails 8 |
| **Workers** | Sidekiq workers |
| **Краулеры** | Локальные гемы |
| **Transform** | Локальные шаги + внешние ML (API) |
| **Очереди / хранилище** | Redis, PostgreSQL |
| **Observability** | OpenTelemetry, Grafana, NewRelic |

**Оркестратор во главе:** нет. Cron создаёт job'ы; Sidekiq выполняет; sync_jobs/sync_tasks хранят состояние. Подходит для MVP до выбора оркестратора.

---

## Общие компоненты

### Языки и фреймворки

- **Ruby 4 / Rails 8** — основной ETL, API, краулеры
- **Python** — ML, ImageRecognition (отдельные сервисы), общение через HTTP API

### Краулеры

- **Instagram, YouTube, TikTok** — локальная разработка гемов
- Интеграция — через API, постраничная загрузка, батчи

### Transform

- **Локальные:** валидация, приведение типов, нормализация, маппинг классов
- **Внешние (опционально):** ML, ImageRecognition — распознавание картинок, текста, упоминаний (async API, polling)

### Инфраструктура

- **Kubernetes** — оркестрация контейнеров
- **Docker** — образы
- **Yandex Cloud** — хостинг (переход с AWS)

### Observability

- **Grafana** — дашборды, метрики (дашборды — отдельная задача)
- **NewRelic** — APM, трейсинг
- **OpenTelemetry** — единый источник телеметрии

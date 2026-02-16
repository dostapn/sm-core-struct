# ETL Social Sync — База знаний и план создания сервиса

**Навигация:** [00](00-INDEX.md) | [01](01-ORCHESTRATOR-CHOICE.md) | [02](02-TECH-STACK.md) | [03](03-DEPLOYMENT-ARCHITECTURE.md) | [04](04-SYNC-FLOW.md) | [05](05-DATA-CONTRACTS.md) | [06](06-OBSERVABILITY.md) | [07](07-RESILIENCE-RESTART.md) | [Далее →](01-ORCHESTRATOR-CHOICE.md)

---

Пополняемый документ: архитектура, решения и инструкции по созданию ETL-сервиса синхронизации данных из соцсетей (профили, посты, метрики).

---

## Навигация по документам

| Документ | Описание |
|----------|----------|
| [01-ORCHESTRATOR-CHOICE](01-ORCHESTRATOR-CHOICE.md) | Сравнение оркестраторов по критериям; полезный инструментарий |
| [02-TECH-STACK](02-TECH-STACK.md) | Варианты стеков (Temporal / Kiba+Sidekiq / общий), компоненты |
| [03-DEPLOYMENT-ARCHITECTURE](03-DEPLOYMENT-ARCHITECTURE.md) | Диаграмма развёртывания, Extract как дорогой этап |
| [04-SYNC-FLOW](04-SYNC-FLOW.md) | Флоу синка: Extract → Transform (полный) → Load |
| [05-DATA-CONTRACTS](05-DATA-CONTRACTS.md) | Форматы данных, sync_jobs/sync_tasks |
| [06-OBSERVABILITY](06-OBSERVABILITY.md) | Логи, трейсы, инциденты |
| [07-RESILIENCE-RESTART](07-RESILIENCE-RESTART.md) | Идемпотентность, checkpoints, докачка |

---

## План создания сервиса

1. **Согласовать название** — ETLooky, Siphon, Data Siphon и др.
2. **Выбрать оркестратор** — Temporal / Kiba+Sidekiq / общий (см. 01, 02).
3. **Развернуть инфраструктуру** — Kubernetes, Docker, Yandex Cloud.
4. **Реализовать Extract** — Ruby-гемы под Instagram, YouTube, TikTok.
5. **Реализовать Transform** — валидация, нормализация, интеграция с ML/ImageRecognition (API, polling).
6. **Реализовать Load** — raw zone, normalized tables.
7. **Настроить Observability** — OpenTelemetry, Grafana, NewRelic.
8. **Реализовать Resilience** — checkpoints, докачка, backfills.

---

## Контекст

- **Цель**: синк профилей, подписок, постов, лайков, комментов и прочих метрик.
- **Платформы**: Instagram, YouTube, TikTok (локальные гемы).
- **Масштаб**: тысячи–сотни тысяч постов на профиль; сотни тысяч профилей; в перспективе миллионы.
- **Стек**: Ruby 4 / Rails 8 основной; Python — ML/ImageRecognition, общение через API.

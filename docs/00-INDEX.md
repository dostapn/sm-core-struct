# ETL Social Sync — База знаний и план создания сервиса

**Навигация:** [README](../README.md) | [Меню](00-INDEX.md) | [Далее →](01-ORCHESTRATOR-CHOICE.md)

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
| [05-DATA-CONTRACTS](05-DATA-CONTRACTS.md) | Форматы данных, JobCall/ServiceCall |
| [06-OBSERVABILITY](06-OBSERVABILITY.md) | Логи, трейсы, инциденты |
| [07-RESILIENCE-RESTART](07-RESILIENCE-RESTART.md) | Идемпотентность, checkpoints, докачка |
| [08-IMPLEMENTATION-ROADMAP](08-IMPLEMENTATION-ROADMAP.md) | Пошаговый роадмап от нуля до синка (промпт-постановка) |

---

## План создания сервиса

1. **Название:** SM Core (sm-core) — Social Monitoring Core.
2. **Оркестратор:** MVP на Sidekiq + cron → апгрейд до Temporal (см. 02, 08).
3. **Развернуть инфраструктуру** — Docker, Yandex Cloud, Kubernetes.
4. **Реализовать Extract** — looky-gem-insteon (Instagram), атомарные джобы.
5. **Реализовать Transform** — валидация, нормализация (ML/ImageRecognition — позже).
6. **Реализовать Load** — SocialProfile, Follower, Subscription, JobCall, ServiceCall.
7. **Observability:** NewRelic (MVP), OpenTelemetry — позже.
8. **Resilience** — JobCall, ServiceCall, курсоры, докачка (см. 07).

---

## Контекст

- **Цель**: синк профилей, подписок, постов, лайков, комментов и прочих метрик.
- **Платформы**: Instagram, YouTube, TikTok (локальные гемы).
- **Масштаб**: тысячи–сотни тысяч постов на профиль; сотни тысяч профилей; в перспективе миллионы.
- **Стек**: Ruby 4 / Rails 8 основной; Python — ML/ImageRecognition, общение через API.

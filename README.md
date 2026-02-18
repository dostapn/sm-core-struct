# SM Core (sm_core)

Social Monitoring Core — архитектура и база знаний ETL-сервиса синхронизации данных из соцсетей.

## О проекте

Документация по проектированию ETL-пайплайна: профили, посты, метрики из Instagram, YouTube, TikTok. Оркестрация (Temporal / Sidekiq Flow), resilience, observability.

## Документация

| Документ | Описание |
|----------|----------|
| [00-INDEX](docs/00-INDEX.md) | Оглавление, план создания сервиса |
| [01-ORCHESTRATOR-CHOICE](docs/01-ORCHESTRATOR-CHOICE.md) | Сравнение оркестраторов |
| [02-TECH-STACK](docs/02-TECH-STACK.md) | Варианты стеков, графы взаимодействия |
| [03-DEPLOYMENT-ARCHITECTURE](docs/03-DEPLOYMENT-ARCHITECTURE.md) | Архитектура развёртывания |
| [04-SYNC-FLOW](docs/04-SYNC-FLOW.md) | Флоу синка |
| [05-DATA-CONTRACTS](docs/05-DATA-CONTRACTS.md) | Контракты данных |
| [06-OBSERVABILITY](docs/06-OBSERVABILITY.md) | Логи, трейсы, инциденты |
| [07-RESILIENCE-RESTART](docs/07-RESILIENCE-RESTART.md) | Идемпотентность, докачка |
| [08-IMPLEMENTATION-ROADMAP](08-IMPLEMENTATION-ROADMAP.md) | Пошаговый роадмап от нуля до синка (промпт-постановка) |

## AI-prompt

[ai-prompt.md](ai-prompt.md) - базовый промпт с описанием структуры

## Стек

Ruby 4 / Rails 8 · Temporal / Sidekiq Flow · Kubernetes · Yandex Cloud

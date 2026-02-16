# Выбор оркестратора

**Навигация:** [00](00-INDEX.md) | [01](01-ORCHESTRATOR-CHOICE.md) | [02](02-TECH-STACK.md) | [03](03-DEPLOYMENT-ARCHITECTURE.md) | [04](04-SYNC-FLOW.md) | [05](05-DATA-CONTRACTS.md) | [06](06-OBSERVABILITY.md) | [07](07-RESILIENCE-RESTART.md) | [← Назад](00-INDEX.md) | [Далее →](02-TECH-STACK.md)

---

Объективное сравнение оркестраторов по критериям и список полезного инструментария.

---

## Сравнение по критериям

| Критерий | Temporal | Prefect | Dagster | Airflow | Kestra | Kiba | Sidekiq Flow |
|----------|----------|---------|---------|---------|--------|------|--------------|
| **Долгие workflow (дни)** | Да (Continue-As-New) | Durable execution | Ограничено | Нет | Частично | Polling-service* | Polling-service* |
| **Ruby SDK** | Да (официальный) | Нет | Нет | Нет | Нет | Нативный | Нативный |
| **Resume с места сбоя** | Встроено | Да | Retry | Retry | Retry | Вручную | Вручную |
| **Heartbeat для long-running** | Да | Частично | Да | Да | — | Нет | Нет |
| **Child workflows / батчи** | Да | Да | Да | SubDAG | Да | Kiba Pro** | DAG |
| **OpenTelemetry** | Да | Да | Да | Да | Да | Вручную | Вручную |
| **Стоимость** | Self-hosted: бесплатно | OSS бесплатно | OSS бесплатно | OSS бесплатно | OSS бесплатно | Pro €950/год** | Бесплатно |
| **Инфраструктура** | Temporal Server + DB | Prefect Server | Dagster | Airflow + DB | Kestra Server | Redis + Sidekiq | Redis + Sidekiq |

\* **Polling-service:** можно обойтись отдельным сервисом опроса результатов (cron/job). Время обработки — от 1 минуты до нескольких дней; опрос выполняется самостоятельно.  
\** **Kiba Pro:** часть функционала (ParallelTransform, bulk upsert) — платная. Kiba OSS — бесплатен, но без батчей/параллелизма.

---

## Детальный разбор по оркестраторам

| Оркестратор | Плюсы | Минусы | Стоимость | Снижение затрат | Оверинжиниринг? |
|-------------|-------|--------|-----------|-----------------|-----------------|
| **Temporal** | Ruby SDK, Continue-As-New, Child Workflows, resume, heartbeat, polling samples | Операционная сложность (кластер) | Self-hosted: бесплатно; Cloud: $100+/мес | Self-hosted в Kubernetes | Нет |
| **Prefect** | Durable execution, Python-native, гибкость | Нет Ruby SDK | OSS бесплатно | — | Не применимо (нет Ruby) |
| **Dagster** | Asset-first, lineage, тестируемость | Нет Ruby SDK | OSS бесплатно | — | Не применимо (нет Ruby) |
| **Airflow** | Стандарт индустрии, экосистема | Нет Ruby SDK, нет долгих workflow | OSS бесплатно | — | Возможно, для простых пайплайнов |
| **Kestra** | YAML, декларативность | Нет Ruby SDK | OSS бесплатно | — | Не применимо (нет Ruby) |
| **Kiba** | Ruby-native, мало зависимостей, ETL DSL | Нет долгих workflow; батчи — только в Pro (€950/год) | OSS бесплатно; Pro платно | Kiba OSS + своя логика батчей и polling | Нет, если процессы короткие |
| **Sidekiq Flow** | Ruby-native, DAG, бесплатно | Нет resume, heartbeat; polling — своя реализация | Бесплатно | — | Нет |

---

## Рекомендации по сценариям

### Вариант A: Temporal (self-hosted)

**Когда выбирать:** долгие процессы, resume без ручной логики, heartbeat, готовность поднять кластер.

**Плюсы:** Ruby SDK, Continue-As-New, Child Workflows, polling samples.  
**Минусы:** Операционная сложность.

---

### Вариант B: Sidekiq Flow + polling-service

**Когда выбирать:** минимальная инфраструктура, готовность реализовать polling-service (опрос результатов 1 мин — несколько дней).

**Плюсы:** Ruby-native, Redis + Sidekiq.  
**Минусы:** Resume, checkpoints — вручную.

---

### Вариант C: Общий (оркестратор-агностик)

**Когда выбирать:** сначала проектируем процессы, оркестратор выберем позже.

---

## Полезный инструментарий (несравнимое)

| Инструмент | Назначение | Когда использовать |
|------------|------------|-------------------|
| **OpenTelemetry** | Distributed tracing, метрики, единый стандарт телеметрии | Для сквозного трейсинга запросов и шагов ETL |
| **Grafana** | Дашборды, алерты | Визуализация метрик и логов (дашборды — отдельная задача) |
| **NewRelic** | APM, трейсинг | Если уже в стеке |
| **lakeFS** | Версионирование данных в DWH | При необходимости «поворота» на другую версию данных; выглядит избыточным на старте, но полезно при росте |
| **Redis** | Очереди, кэш, locks | Sidekiq, Temporal (опционально) |
| **PostgreSQL** | Данные, sync_jobs, sync_tasks | Хранение состояния и результатов |
| **Kiba Common** | CSV, SQL sources/destinations | Расширение Kiba OSS |
| **Sidekiq Pro Batches** | DAG поверх Sidekiq, callbacks | Если уже есть Sidekiq Pro — альтернатива Sidekiq Flow |

---

## Официальные ресурсы

- [Temporal](https://temporal.io/) — [Ruby SDK](https://github.com/temporalio/sdk-ruby) | [samples-ruby](https://github.com/temporalio/samples-ruby)
- [Kiba ETL](https://www.kiba-etl.org/) | [Kiba Pro](https://www.kiba-etl.org/kiba-pro)
- [Sidekiq Flow](https://github.com/ArizenHQ/sidekiq_flow)
- [Prefect](https://www.prefect.io/)
- [Dagster](https://dagster.io/)
- [Apache Airflow](https://airflow.apache.org/)
- [Kestra](https://kestra.io/)

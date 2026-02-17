# Observability

**Навигация:** [README](../README.md) | [Меню](00-INDEX.md) | [← Назад](05-DATA-CONTRACTS.md) | [Далее →](07-RESILIENCE-RESTART.md)

---

Логи, трейсы, инциденты.

---

## Корреляционные идентификаторы

- **trace_id** — глобальный на каждый workflow run
- Каждый шаг и внешний вызов логирует: `trace_id` + `step_name` + `input_id` (batch_id)
- Логи — в централизованное хранилище (ELK, Loki, Datadog, OpenTelemetry-совместимое)

---

## Structured logging

JSON с полями:

- `workflow_id`
- `job_call_id` (JobCall.id для трейсинга)
- `step`
- `status`
- `error_type`
- `external_job_id`
- `attempt`
- `duration_ms`

**Не логировать** сырые payload'ы — только ссылку/ключи (S3 key, record id) и минимум PII.

---

## Tracing

- **OpenTelemetry SDK** + экспортер (Jaeger, Tempo, Datadog)
- Один trace: запрос → создание workflow → шаги ETL → внешние сервисы

**Span-атрибуты:**

- `etl.stage` — extract, transform, load
- `etl.source` — instagram, youtube, tiktok
- `ml.model_name` — при вызове ML
- `retry_count`
- `external_service`

---

## Инциденты и алерты

- **SLA/timeout** для шагов и workflow в оркестраторе
- При превышении или ошибках — эвент в PagerDuty, Opsgenie, Slack
- Статусы (success, failed, skipped) и retries видны в UI (Temporal, Dagster, Prefect, Sidekiq)

---

## Инструменты

- **Grafana** — дашборды, метрики (дашборды — отдельная задача, настраиваются по запросу)
- **NewRelic** — APM, трейсинг
- **OpenTelemetry Collector** — приём и экспорт телеметрии

# Инструкция для ИИ-агента: SM Core

Загружать как промпт при реализации и рефакторинге. Следовать при написании кода.

---

## 1. Разделение слоёв

| Слой | Директория | Назначение |
|------|------------|------------|
| API | `app/operations` | Controller → Operation. Бизнес-логика HTTP-запросов |
| Фоновые задачи | `app/services` | Job → Service. Логика синка, ETL, джоб |

Базовый класс `app/services/application_service.rb` — опционально, для общей логики (concern).

---

## 2. Сервисы: неймспейс и иерархия

**Неймспейс по платформе:** `instagram/`, позже `youtube/`, `tiktok/`.

**Профиль — корневая сущность.** Followings, posts, reels, highlights привязаны к профилю и вложены под него.

```
app/services/
  application_service.rb
  instagram/
    social_profile_service.rb       # Instagram::SocialProfileService
    social_profile/
      subscriptions_service.rb     # Instagram::SocialProfile::SubscriptionsService
      # posts_service.rb            # позже
      # reels_service.rb            # позже
      # highlights_service.rb       # позже
```

Обоснование: дочерние сущности не существуют без профиля. Вложенность `profile/` отражает принадлежность и упрощает добавление posts, reels, highlights в тот же неймспейс.

---

## 3. Методы в сервисах

В каждом сервисе — два уровня:

- **`sync`** — общий: полный синк (триггер цепочки джоб или цикл)
- **`sync_page`** — частный: одна страница, один запрос к API (атомарно)

Пример `Instagram::SocialProfile::SubscriptionsService`:
- `sync(social_profile_id:)` — запуск полного синка (ставит первую джобу или цикл sync_page)
- `sync_page(social_profile_id:, end_cursor:, api_provider:)` — одна страница followings

---

## 4. Джобы

Неймспейс `Instagram::`:

- `app/jobs/instagram/sync_social_account_job.rb` → `Instagram::SyncSocialAccountJob`
- `app/jobs/instagram/sync_subscriptions_job.rb` → `Instagram::SyncSubscriptionsJob`

Джоба вызывает метод сервиса (`sync` или `sync_page`).

**ETL-модель:** джоба при старте создаёт JobCall; каждый вызов сервиса (API, DB) создаёт ServiceCall (job_call_id). Подзадача (след. страница) → ServiceCall с parent_service_call_id.

---

## 5. Спеки

Зеркало app: `spec/services/instagram/`, `spec/services/instagram/social_profile/`, `spec/jobs/instagram/`, `spec/operations/`.

---

## 6. Комментарии

Для каждого класса, модуля и метода — **обязательные комментарии на русском**, 1–2 строки. Описание назначения и, при необходимости, ключевой логики.

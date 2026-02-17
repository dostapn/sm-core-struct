# Контракты данных

**Навигация:** [README](../README.md) | [Меню](00-INDEX.md) | [← Назад](04-SYNC-FLOW.md) | [Далее →](06-OBSERVABILITY.md)

---

Форматы данных, API, схема JobCall/ServiceCall.

---

## Входные данные

- **ID профилей** — список entity_id для синка
- **Параметры синка** — since_id, until_id, last_synced_at, scope (profile | posts | comments | likes | all)

---

## Выходные данные (SM Core)

- **SocialProfile** — platform, ext_id, username, full_name, bio, avatar, url, followers_count, follows_count, posts_count, engagement_rate, raw_data
- **Follower** — platform, ext_id, username, full_name, avatar, raw_data
- **Subscription** — social_profile_id, follower_id, active, followed_at, unfollowed_at
- **Посты** — id, profile_id, caption (одна на пост), media (фото, видео — много элементов), created_at, …
- **Лайки, комменты** — id, post_id, author_id, content, created_at, …
- **Метрики** — подписчики, просмотры, engagement и т.п.

Форматы уточняются при реализации под каждую платформу (Instagram, YouTube, TikTok).

---

## API бэкенда

- **REST** или **GraphQL** — по выбору
- MVP: `GET /api/v1/bloggers` (поиск). Запуск синка, статус — в будущем.

---

## Контракт внешних Transform (ML, ImageRecognition)

- **Запрос:** идентификатор элемента (post_id, media_id), тип (image, text)
- **Ответ:** асинхронный — job_id, статус (pending | processing | done | failed)
- **Polling:** GET /jobs/{job_id} — статус и результат при done

---

## Модель JobCall (вызов джобы)

| Поле | Тип | Описание |
|------|-----|----------|
| id | bigint | PK |
| subject_id | bigint | FK, nullable (MVP: social_profile_id) |
| kind | string | profile, followings, posts, … |
| status | string | статус выполнения¹ |
| started_at, finished_at | timestamp | |
| created_at, updated_at | timestamp | |

---

## Модель ServiceCall (вызов сервиса внутри джобы)

| Поле | Тип | Описание |
|------|-----|----------|
| id | bigint | PK |
| job_call_id | bigint | FK |
| parent_service_call_id | bigint | FK, nullable — предыдущий этап (пагинация) |
| kind | string | profile, followings_page, … |
| status | string | статус выполнения¹ |
| cursor | jsonb | курсоры, токены пагинации |
| result_summary | jsonb | метаданные результата |
| error_message | text | при failed |
| started_at, finished_at | timestamp | |
| created_at, updated_at | timestamp | |

---

## ¹ Статус выполнения

| Статус | Описание |
|--------|----------|
| **pending** | В очереди, ещё не начат |
| **running** | Выполняется |
| **succeeded** | Успешно завершён |
| **failed** | Ошибка |
| **partial** | Частичный успех (часть данных получена, можно докачать) |
| **skipped** | Пропущено (напр. уже актуально) |

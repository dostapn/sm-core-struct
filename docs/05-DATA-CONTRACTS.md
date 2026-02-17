# Контракты данных

**Навигация:** [README](../README.md) | [Меню](00-INDEX.md) | [← Назад](04-SYNC-FLOW.md) | [Далее →](06-OBSERVABILITY.md)

---

Форматы данных, API, схема sync_jobs/sync_tasks.

---

## Входные данные

- **ID профилей** — список entity_id для синка
- **Параметры синка** — since_id, until_id, last_synced_at, scope (profile | posts | comments | likes | all)

---

## Выходные данные

- **Профили** — атрибуты профиля (id, username, bio, followers_count, …)
- **Посты** — id, profile_id, caption (одна на пост), media (фото, видео — много элементов), created_at, …
- **Лайки, комменты** — id, post_id, author_id, content, created_at, …
- **Метрики** — подписчики, просмотры, engagement и т.п.

Форматы уточняются при реализации под каждую платформу (Instagram, YouTube, TikTok).

---

## API бэкенда

- **REST** или **GraphQL** — по выбору
- Endpoints: запуск синка, статус job, результаты

---

## Контракт внешних Transform (ML, ImageRecognition)

- **Запрос:** идентификатор элемента (post_id, media_id), тип (image, text)
- **Ответ:** асинхронный — job_id, статус (pending | processing | done | failed)
- **Polling:** GET /jobs/{job_id} — статус и результат при done

---

## Модель sync_jobs

| Поле | Тип | Описание |
|------|-----|----------|
| id | bigint | PK |
| network | string | instagram, youtube, tiktok |
| entity_id | string | ID профиля/аккаунта в соцсети |
| cursor | jsonb | since_id, until_id, page_token и т.п. |
| status | enum | статус выполнения¹ |
| error_type | string | тип ошибки при failed |
| error_message | text | сообщение об ошибке |
| previous_job_id | bigint | FK на предыдущий job при рестарте/докачке |
| started_at | timestamp | |
| finished_at | timestamp | |

---

## Модель sync_tasks (опционально, гранулярно)

| Поле | Тип | Описание |
|------|-----|----------|
| id | bigint | PK |
| job_id | bigint | FK → sync_jobs |
| kind | enum | profile, posts, comments, likes |
| external_id | string | post_id, comment_id и т.п. |
| status | enum | статус выполнения¹ |
| attempts | int | количество попыток |
| last_error | text | |
| last_synced_at | timestamp | |

---

## ¹ Статус выполнения

| Статус | Описание |
|--------|----------|
| **pending** | В очереди, ещё не начат |
| **running** | Выполняется |
| **succeeded** | Успешно завершён |
| **failed** | Ошибка |
| **partial** | Частичный успех (часть данных получена, можно докачать) |
| **skipped** | Пропущено (напр. уже актуально; в основном для sync_tasks) |

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
    profile_service.rb              # Instagram::ProfileService
    profile/
      followings_service.rb         # Instagram::Profile::FollowingsService
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

Пример `Instagram::Profile::FollowingsService`:
- `sync_page(profile_id:, end_cursor:, api_provider:)` — одна страница followings
- `sync(profile_id:)` — запуск полного синка (ставит первую джобу или цикл sync_page)

---

## 4. Джобы

Неймспейс `Instagram::`:

- `app/jobs/instagram/sync_profile_job.rb` → `Instagram::SyncProfileJob`
- `app/jobs/instagram/sync_followings_page_job.rb` → `Instagram::SyncFollowingsPageJob`

Джоба вызывает метод сервиса (`sync` или `sync_page`).

---

## 5. Спеки

Зеркало app: `spec/services/instagram/`, `spec/services/instagram/profile/`, `spec/jobs/instagram/`, `spec/operations/`.

---

## 6. Комментарии

Для каждого класса, модуля и метода — **обязательные комментарии на русском**, 1–2 строки. Описание назначения и, при необходимости, ключевой логики.

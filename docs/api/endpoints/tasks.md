# API: задачи

Ниже описаны основные операции с задачами в рамках API v1. Все пути относительно базового URL `https://api.tasktracker.example.com/v1`.

## Общие соглашения

- Идентификатор задачи в URL — **глобальный UUID** (`task_id`), а человекочитаемый номер (`ALF-42`) возвращается в поле `sequence_key`.
- Заголовок `Content-Type: application/json` обязателен для тел с JSON.

---

## Список задач в проекте

`GET /projects/{project_id}/tasks`

### Параметры пути

| Параметр | Описание |
|----------|----------|
| `project_id` | UUID проекта |

### Query-параметры

| Параметр | Описание |
|----------|----------|
| `status` | Фильтр по коду статуса, например `in_progress` |
| `assignee_id` | UUID исполнителя |
| `label` | Метка (можно повторять для нескольких меток) |
| `page`, `per_page` | Пагинация |

### Пример запроса

```http
GET /v1/projects/550e8400-e29b-41d4-a716-446655440000/tasks?status=in_progress&per_page=10 HTTP/1.1
Host: api.tasktracker.example.com
Authorization: Bearer tt_live_xxxxxxxx
Accept: application/json
```

### Пример ответа `200 OK`

```json
{
  "data": [
    {
      "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
      "sequence_key": "ALF-12",
      "title": "Добавить валидацию формы заказа",
      "status": "in_progress",
      "priority": "high",
      "assignee": {
        "id": "user-uuid-1",
        "display_name": "Иван Петров"
      },
      "due_at": "2026-04-15T18:00:00Z",
      "updated_at": "2026-04-10T09:30:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "per_page": 10,
    "total_count": 34,
    "total_pages": 4
  }
}
```

---

## Получить задачу

`GET /tasks/{task_id}`

### Пример ответа `200 OK`

```json
{
  "data": {
    "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "project_id": "550e8400-e29b-41d4-a716-446655440000",
    "sequence_key": "ALF-12",
    "title": "Добавить валидацию формы заказа",
    "description": "## Критерии\n- [ ] Клиентская валидация\n- [ ] Серверная валидация",
    "description_format": "markdown",
    "status": "in_progress",
    "priority": "high",
    "assignee_id": "user-uuid-1",
    "labels": ["frontend", "orders"],
    "due_at": "2026-04-15T18:00:00Z",
    "created_at": "2026-04-08T11:00:00Z",
    "updated_at": "2026-04-10T09:30:00Z"
  }
}
```

---

## Создать задачу

`POST /projects/{project_id}/tasks`

### Тело запроса

| Поле | Тип | Обязательное | Описание |
|------|-----|----------------|----------|
| `title` | string | да | Заголовок, 1–500 символов |
| `description` | string | нет | Текст в Markdown |
| `status` | string | нет | Код статуса; по умолчанию начальный статус проекта |
| `priority` | string | нет | `low`, `medium`, `high`, `urgent` |
| `assignee_id` | string (UUID) | нет | Исполнитель |
| `due_at` | string (ISO 8601) | нет | Срок |
| `labels` | array of string | нет | Метки |

### Пример запроса

```http
POST /v1/projects/550e8400-e29b-41d4-a716-446655440000/tasks HTTP/1.1
Host: api.tasktracker.example.com
Authorization: Bearer tt_live_xxxxxxxx
Content-Type: application/json
Idempotency-Key: 7c9e6679-7425-40de-944b-e07fc1f90ae7

{
  "title": "Исправить отображение НДС в корзине",
  "description": "См. скриншот во вложениях интерфейса.",
  "priority": "medium",
  "labels": ["bug", "checkout"]
}
```

### Пример ответа `201 Created`

```json
{
  "data": {
    "id": "new-task-uuid",
    "sequence_key": "ALF-13",
    "title": "Исправить отображение НДС в корзине",
    "status": "todo",
    "priority": "medium",
    "created_at": "2026-04-10T10:00:00Z",
    "updated_at": "2026-04-10T10:00:00Z"
  }
}
```

!!! note "Ошибки валидации"
    Ответ `400` с `code: "VALIDATION_ERROR"` и массивом `details` по полям — см. [обзор API](../overview.md).

---

## Обновить задачу

`PATCH /tasks/{task_id}`

Передавайте только изменяемые поля.

### Пример тела

```json
{
  "status": "done",
  "assignee_id": null
}
```

### Пример ответа `200 OK`

```json
{
  "data": {
    "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
    "sequence_key": "ALF-12",
    "status": "done",
    "assignee_id": null,
    "updated_at": "2026-04-10T10:15:00Z"
  }
}
```

---

## Удалить задачу

`DELETE /tasks/{task_id}`

Требуются права администратора проекта или владельца организации.

### Пример ответа `204 No Content`

Тело пустое.

!!! warning "Политика удаления"
    В некоторых организациях удаление через API отключено — в этом случае вернётся `403` с пояснением.

---

## Комментарии к задаче

`GET /tasks/{task_id}/comments` — список комментариев.

`POST /tasks/{task_id}/comments` — добавить комментарий.

### Пример создания комментария

```json
{
  "body": "Готово к ревью, ветка `feature/ALF-12-validation`."
}
```

Ответ `201` содержит объект комментария с полями `id`, `body`, `author`, `created_at`.

---

Краткий обзор лимитов и аутентификации: [Обзор API](../overview.md).

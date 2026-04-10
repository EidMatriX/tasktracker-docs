# Настройка окружения разработчика

Инструкция для разработчиков, которые собирают **бэкенд и веб-клиент TaskTracker** из исходников. Предполагается ОС Linux или Windows с WSL2; для macOS шаги аналогичны.

## Требования

| Компонент | Версия (ориентир) |
|-----------|-------------------|
| Python | 3.11+ |
| Node.js | 20 LTS |
| PostgreSQL | 15+ |
| Redis | 7+ |
| Docker (опционально) | 24+ |

!!! tip "Быстрый старт через Docker"
    В репозитории есть `docker-compose.yml`: поднимает PostgreSQL, Redis и миграции. Команда: `docker compose up -d` из корня проекта (см. README в репозитории).

## Клонирование репозитория

```bash
git clone https://github.com/example/tasktracker.git
cd tasktracker
```

## Бэкенд (Python)

### Виртуальное окружение

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -U pip
pip install -r requirements-dev.txt
```

### Переменные окружения

Создайте файл `.env` в каталоге `backend/` на основе примера:

```bash
cp backend/.env.example backend/.env
```

Минимальный набор переменных:

```env
DATABASE_URL=postgresql://tasktracker:tasktracker@localhost:5432/tasktracker
REDIS_URL=redis://localhost:6379/0
SECRET_KEY=сгенерируйте-случайную-строку-минимум-32-символа
JWT_ISSUER=tasktracker-dev
ALLOWED_HOSTS=localhost,127.0.0.1
```

!!! warning "Секреты"
    Не коммитьте `.env`. Для локальных ключей подойдёт `openssl rand -hex 32`.

### Миграции и суперпользователь

```bash
cd backend
alembic upgrade head
python manage.py create_superuser --email admin@local.test
```

### Запуск API

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Проверка: `http://127.0.0.1:8000/health` должен вернуть `{"status":"ok"}`.

## Фронтенд (веб-клиент)

```bash
cd frontend
npm ci
cp .env.example .env.local
```

В `.env.local` укажите URL API:

```env
VITE_API_BASE_URL=http://127.0.0.1:8000/v1
```

Запуск dev-сервера:

```bash
npm run dev
```

Приложение откроется на `http://localhost:5173` (порт может отличаться — смотрите вывод в терминале).

## Тесты

### Бэкенд

```bash
cd backend
pytest -q --cov=app
```

### Фронтенд

```bash
cd frontend
npm run test
npm run lint
```

## Сборка документации (MkDocs)

Документация пользователя и API собирается из каталога `docs/` в корне монорепозитория (или из отдельного репозитория docs — уточните в README вашего форка).

```bash
pip install mkdocs-material
mkdocs serve
```

Откройте в браузере адрес, который выведет MkDocs (обычно `http://127.0.0.1:8000` — не путайте с портом бэкенда; при конфликте используйте `mkdocs serve -a 127.0.0.1:8001`).

## Полезные команды

| Команда | Назначение |
|---------|------------|
| `pre-commit install` | Хуки перед коммитом (линтер, форматирование) |
| `docker compose logs -f api` | Логи API-контейнера |

!!! note "Ветвление"
    Новые фичи оформляйте от `main` в ветках `feature/краткое-описание`; в PR указывайте ссылку на задачу в трекере.

## Дальнейшие шаги

- Ознакомьтесь с [обзором API](../api/overview.md) для контрактов интеграции.
- Правила код-ревью и CI описаны в `CONTRIBUTING.md` в репозитории (при наличии).

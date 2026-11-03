# Todo App Backend

REST API для менеджера задач с категориями. Написан на FastAPI, данные хранятся в PostgreSQL, всё запускается через Docker Compose.

## Стек

- **Python 3.14**, **FastAPI**, **Uvicorn**
- **SQLAlchemy 2** (ORM) и **PostgreSQL 18**
- **Pydantic v2**
- **Docker** и **Docker Compose**

## Возможности

- создание, просмотр, изменение и удаление задач;
- создание, просмотр, изменение и удаление категорий;
- отметка задачи как выполненной;
- валидация входных данных и коды ответов (`201`, `204`, `404`, `422`);
- автоматическая документация API в Swagger UI.

## Архитектура


```
HTTP-запрос → routers → services → repositories → models → PostgreSQL
                  ↑           ↑
               schemas    исключения (TaskNotFound, CategoryNotFound)
```

| Слой | Директория | Задача |
|---|---|---|
| Роутеры | `app/api/routers` | принимают HTTP-запросы, превращают ошибки сервиса в HTTP-коды |
| Сервисы | `app/services` | бизнес-логика, управление транзакциями |
| Репозитории | `app/repositories` | запросы к базе данных |
| Модели | `app/models` | описание таблиц (SQLAlchemy ORM) |
| Схемы | `app/schemas` | формат входных и выходных данных API (Pydantic) |

Зависимости (сессия БД и сервисы) передаются через `Depends`.

## Структура проекта

```
todo-app-backend/
├── app/
│   ├── api/
│   │   ├── routers/        # эндпоинты /tasks и /categories
│   │   └── dependencies.py # внедрение сервисов
│   ├── core/config.py      # настройки из переменных окружения
│   ├── db/session.py       # подключение к БД
│   ├── models/             # ORM-модели
│   ├── repositories/       # доступ к данным
│   ├── schemas/            # Pydantic-схемы
│   ├── services/           # бизнес-логика
│   └── main.py             # точка входа
├── Dockerfile
├── docker-compose.yaml
├── requirements.txt
└── .env.example
```

## Запуск через Docker Compose

1. Клонируйте репозиторий:

   ```bash
   git clone https://github.com/andruyxaParavoz/todo-app-backend.git
   cd todo-app-backend
   ```

2. Создайте файл `.env` по образцу `.env.example` и задайте пароль базы данных:

   ```
   POSTGRES_PASSWORD=your_password
   ```

3. Запустите проект:

   ```bash
   docker compose up -d --build
   ```

4. Откройте документацию API: <http://localhost:8080/docs>

Остановить проект можно командой `docker compose stop`, а остановить и удалить контейнеры — `docker compose down`. Данные базы при этом сохраняются в volume `pgdata`.

## Локальный запуск без Docker

Нужен Python 3.12+ и запущенный PostgreSQL. Базу проще всего поднять из того же compose-файла: `docker compose up -d postgres`. Она будет доступна на порту `15432`.

```bash
python -m venv venv

# Windows (PowerShell)
.\venv\Scripts\Activate.ps1
# Linux / macOS
source venv/bin/activate

pip install -r requirements.txt
uvicorn app.main:app --port 8080 --reload
```

Адрес базы задаётся переменной окружения `DATABASE_URL`. Если она не задана, используется `postgresql+psycopg://postgres:admin@localhost:15432/postgres`.

## Переменные окружения

| Переменная | Использование | Сводка |
|---|---|---|
| `POSTGRES_PASSWORD` | `.env`, docker-compose | пароль пользователя `postgres` |
| `DATABASE_URL` | бэкенд | адрес БД |

## API

### Задачи

| Метод | Путь | Описание | Код |
|---|---|---|---|
| `GET` | `/tasks` | список всех задач | `200` |
| `POST` | `/tasks` | создать задачу | `201` |
| `PATCH` | `/tasks/{task_id}` | изменить название или статус | `200` |
| `DELETE` | `/tasks/{task_id}` | удалить задачу | `204` |

Пример создания задачи:

```http
POST /tasks
Content-Type: application/json

{
  "title": "Сходить в магазин"
}
```

Ответ:

```json
{
  "id": "3f1c2a9e-7b4d-4e8a-9c21-5d6f7a8b9c0d",
  "title": "Сходить в магазин",
  "completed": false
}
```

Отметить задачу выполненной:

```http
PATCH /tasks/3f1c2a9e-7b4d-4e8a-9c21-5d6f7a8b9c0d
Content-Type: application/json

{
  "completed": true
}
```

### Категории

| Метод | Путь | Описание | Код |
|---|---|---|---|
| `GET` | `/categories` | список всех категорий | `200` |
| `POST` | `/categories` | создать категорию (`{"name": "..."}`) | `201` |
| `PATCH` | `/categories/{category_id}` | переименовать категорию | `200` |
| `DELETE` | `/categories/{category_id}` | удалить категорию | `204` |

Если объект с указанным `id` не найден, API возвращает `404`. Если данные в запросе некорректны (например, пустое название), возвращается `422`.

## Фронтенд

API рассчитан на работу с фронтендом на `http://localhost:3000`, этот адрес разрешён в настройках CORS.  
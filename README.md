<div align="center">

# Todo App Backend

REST API для менеджера задач с категориями. Написан на FastAPI, данные хранятся в PostgreSQL, всё запускается через Docker Compose.

[![Python](https://img.shields.io/badge/Python-3.14-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.x-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white)](https://www.sqlalchemy.org/)
<br>
[![Pydantic](https://img.shields.io/badge/Pydantic-v2-E92063?style=for-the-badge&logo=pydantic&logoColor=white)](https://docs.pydantic.dev/)
[![Docker](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Swagger](https://img.shields.io/badge/Swagger_UI-85EA2D?style=for-the-badge&logo=swagger&logoColor=black)](https://swagger.io/tools/swagger-ui/)

[![Last commit](https://img.shields.io/github/last-commit/andruyxaParavoz/todo-app-backend?style=flat-square)](https://github.com/andruyxaParavoz/todo-app-backend/commits)
[![Repo size](https://img.shields.io/github/repo-size/andruyxaParavoz/todo-app-backend?style=flat-square)](https://github.com/andruyxaParavoz/todo-app-backend)
[![Top language](https://img.shields.io/github/languages/top/andruyxaParavoz/todo-app-backend?style=flat-square)](https://github.com/andruyxaParavoz/todo-app-backend)

[Стек](#стек) •
[Архитектура](#архитектура) •
[Запуск](#запуск-через-docker-compose) •
[API](#api)

</div>

---

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


```mermaid
flowchart LR
    C([HTTP-запрос]) --> R[routers]
    R --> S[services]
    S --> Rep[repositories]
    Rep --> M[models]
    M --> DB[(PostgreSQL)]
    Sch[[schemas]] -.-> R
    S -. исключения TaskNotFound, CategoryNotFound .-> R
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

<details>
<summary>Показать структуру</summary>

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

</details>

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

> [!WARNING]
> Команда `docker compose down -v` удалит volume вместе со всеми данными базы.

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

| Переменная | Использование | Описание |
|---|---|---|
| `POSTGRES_PASSWORD` | `.env`, docker-compose | пароль пользователя `postgres` |
| `DATABASE_URL` | бэкенд | строка подключения к PostgreSQL |

## API

### Задачи

| Метод | Путь | Описание | Код |
|:---:|---|---|:---:|
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/tasks` | список всех задач | `200` |
| ![POST](https://img.shields.io/badge/POST-49CC90?style=flat-square) | `/tasks` | создать задачу | `201` |
| ![PATCH](https://img.shields.io/badge/PATCH-50E3C2?style=flat-square) | `/tasks/{task_id}` | изменить название или статус | `200` |
| ![DELETE](https://img.shields.io/badge/DELETE-F93E3E?style=flat-square) | `/tasks/{task_id}` | удалить задачу | `204` |

<details>
<summary>Примеры запросов</summary>

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

</details>

### Категории

| Метод | Путь | Описание | Код |
|:---:|---|---|:---:|
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/categories` | список всех категорий | `200` |
| ![POST](https://img.shields.io/badge/POST-49CC90?style=flat-square) | `/categories` | создать категорию (`{"name": "..."}`) | `201` |
| ![PATCH](https://img.shields.io/badge/PATCH-50E3C2?style=flat-square) | `/categories/{category_id}` | переименовать категорию | `200` |
| ![DELETE](https://img.shields.io/badge/DELETE-F93E3E?style=flat-square) | `/categories/{category_id}` | удалить категорию | `204` |

Если объект с указанным `id` не найден, API возвращает `404`. Если данные в запросе некорректны (например, пустое название), возвращается `422`.

## Фронтенд

API рассчитан на работу с фронтендом на `http://localhost:3000`, этот адрес разрешён в настройках CORS.  
# Smart Spend

Приложение для управления расходами на базе FastAPI с модульной архитектурой.

## Команда проекта
1. Тимлид-Разработчик: Лаврентьев Никита	 (skitchen1992@gmail.com)
2. Software Engineer: Артемов Гоша	()
3. Software Engineer: Лукьянчук Владислав	 ()
4. Software Engineer: Цыбизов Дмитрий	 ()
5. Software Engineer Паниклов Арсений (arseniy.paniklov@gmail.com)

## Быстрый старт

### Через Docker (рекомендуется):

```bash
# 1. Создайте файл .env на основе .env.example (см. раздел "Настройка переменных окружения")
cp .env.example .env

# 2. Запустите приложение
make docker-build    # Собрать Docker образы (только при первом запуске или изменении зависимостей)
make docker-up       # Запустить контейнеры (PostgreSQL + FastAPI)
make docker-migrate  # Применить миграции БД
```

Приложение будет доступно по адресу: http://localhost:8000

### Локально:

```bash
# 1. Установите Poetry (если еще не установлен)
curl -sSL https://install.python-poetry.org | python3 -

# 2. Установите зависимости
poetry install

# 3. Создайте файл .env на основе .env.example (см. раздел "Настройка переменных окружения")
cp .env.example .env

# 4. Активируйте виртуальное окружение и запустите
poetry shell
make dev  # Режим разработки с автоперезагрузкой
```

## Установка и настройка

### Требования

- Python 3.11+
- Poetry (для локальной разработки)
- Docker и Docker Compose (для Docker-запуска)

### Настройка переменных окружения

**Важно:** Файл `.env` обязателен для работы приложения!

1. Скопируйте шаблон:

   ```bash
   cp .env.example .env
   ```

2. Отредактируйте `.env` и установите следующие переменные:

   **Обязательные переменные:**

   - `DATABASE_URL` - URL подключения к базе данных в формате `postgresql://user:password@host:port/database`
   - `SECRET_KEY` - секретный ключ для JWT токенов (минимум 32 символа)
   - `POSTGRES_USER` - имя пользователя PostgreSQL (для Docker)
   - `POSTGRES_PASSWORD` - пароль пользователя PostgreSQL (для Docker)
   - `POSTGRES_DB` - имя базы данных (для Docker)

   **Опциональные переменные:**

   - `CORS_ORIGINS` - разрешенные источники для CORS, разделенные запятыми (по умолчанию: `http://localhost:3000,http://localhost:8000`)

3. **Важно:** Файл `.env` должен быть в `.gitignore` и не коммититься в репозиторий!

### Приоритет переменных окружения

Переменные окружения загружаются в следующем порядке приоритета:

1. **Переменные окружения системы** (наивысший приоритет)
2. **Файл `.env`** в корне проекта
3. **Значения по умолчанию** в коде (низший приоритет)

## Запуск

### 🐳 Запуск через Docker

**Важно:** Убедитесь, что Docker Desktop запущен перед выполнением команд!

#### Когда нужно пересобирать Docker образы:

Пересборка образа требуется в следующих случаях:

- **Первый запуск проекта** - обязательно собрать образ перед запуском
- **Изменения в зависимостях Python** - при изменении `pyproject.toml` или `poetry.lock`
- **Изменения в Dockerfile** - при любых изменениях в `Dockerfile` или `Dockerfile.dev`
- **Изменения в системных зависимостях** - при необходимости установки новых системных библиотек

**Когда НЕ нужно пересобирать образ:**

- **Изменения в коде приложения** - благодаря volume mount изменения в Python коде применяются автоматически
- **Изменения в переменных окружения** - достаточно перезапустить контейнеры (`make docker-restart`)

#### Прямые команды docker compose:

> **Примечание:** Используйте `docker compose` (без дефиса) - это встроенная команда Docker CLI.

```bash
docker compose up -d              # Запустить в фоне
docker compose up                 # Запустить с выводом логов
docker compose down               # Остановить
docker compose logs -f app        # Логи приложения
docker compose exec app bash      # Войти в контейнер
```

### Локальный запуск (без Docker):

```bash
make dev          # Запустить в режиме разработки (с автоперезагрузкой)
make run          # Запустить приложение
make prod         # Запустить в production режиме
make help         # Показать все доступные команды
```

**Через Poetry скрипты:**

```bash
poetry run dev    # Режим разработки
poetry run start  # Production режим
```

## Работа с базой данных

### Проверка статуса PostgreSQL в Docker:

```bash
# Проверить статус контейнера БД
docker compose ps db

# Проверить логи контейнера БД
docker compose logs db

# Проверить healthcheck контейнера
docker compose exec db pg_isready -U ${POSTGRES_USER}
```

### Подключение к базе данных

#### Через psql (в Docker):

```bash
# Подключиться к PostgreSQL в контейнере
docker compose exec db psql -U ${POSTGRES_USER} -d ${POSTGRES_DB}

# Выполнить SQL запрос напрямую
docker compose exec db psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} -c "SELECT version();"

# Показать список баз данных
docker compose exec db psql -U ${POSTGRES_USER} -c "\l"

# Показать список таблиц
docker compose exec db psql -U ${POSTGRES_USER} -d ${POSTGRES_DB} -c "\dt"
```

#### Через dBeaver:

**Важно:** Убедитесь, что контейнер с базой данных запущен (`docker compose up -d db` или `make docker-up`).

**Параметры подключения:**

- **Host:** `localhost`
- **Port:** `5432`
- **Database:** значение `POSTGRES_DB` из `.env`
- **Username:** значение `POSTGRES_USER` из `.env`
- **Password:** значение `POSTGRES_PASSWORD` из `.env`

**Примечания:**

- Порт `5432` проброшен из Docker контейнера на хост (см. `docker-compose.yml`)
- Если подключение не удается, убедитесь, что контейнер БД запущен: `docker compose ps db`
- Для просмотра логов БД: `docker compose logs db`

### Устранение проблем:

**Если контейнер БД не запускается:**

```bash
# Перезапустить контейнер БД
docker compose restart db

# Просмотреть детальные логи
docker compose logs -f db
```

**Если нужно пересоздать базу данных:**

```bash
# Остановить и удалить контейнеры с volumes
docker compose down -v

# Запустить заново
docker compose up -d db
make docker-migrate
```

**Проверка переменных окружения:**

```bash
# Проверить переменные в Docker контейнере приложения
docker compose exec app env | grep DATABASE_URL

# Проверить переменные в Docker контейнере БД
docker compose exec db env | grep POSTGRES
```

## Структура проекта

```
smart_spend/
├── app/
│   ├── main.py                    # Точка входа в приложение
│   │
│   ├── core/                      # Базовые системные компоненты (инфраструктура)
│   │   ├── config.py              # Конфигурация, переменные окружения
│   │   ├── db.py                  # Подключение к БД (SQLAlchemy)
│   │   ├── security.py            # Авторизация, JWT, хэширование паролей
│   │   ├── dependencies.py        # Зависимости (Depends)
│   │   ├── exceptions.py          # Кастомные исключения
│   │   ├── dto/                   # Общие DTO (Data Transfer Objects)
│   │   ├── guards/                # Проверка доступа (аналог NestJS Guards)
│   │   ├── pipes/                 # Валидация и преобразование данных
│   │   └── core_module.py         # Инициализация ядра приложения
│   │
│   ├── shared/                    # Общие утилиты и базовые классы
│   │   ├── base_model.py          # Базовые ORM-модели
│   │   ├── mixins.py              # Повторно используемые классы (CRUD)
│   │   └── utils.py               # Хелперы, форматирование, экспорт
│   │
│   ├── modules/                    # Основные бизнес-модули
│   │   ├── users/                 # Модуль пользователей
│   │   │   ├── models.py          # ORM-модель пользователя
│   │   │   ├── schemas.py         # Pydantic-схемы (DTO)
│   │   │   ├── repository.py      # CRUD и работа с БД
│   │   │   ├── service.py         # Бизнес-логика пользователей
│   │   │   └── router.py          # REST API по пользователям
│   │   │
│   │   ├── groups/                # Модуль групп
│   │   │   ├── models.py          # ORM-модель группы
│   │   │   ├── schemas.py         # Pydantic-схемы (DTO)
│   │   │   ├── repository.py      # CRUD и работа с БД
│   │   │   ├── service.py         # Бизнес-логика групп
│   │   │   └── router.py          # REST API по группам
│   │   │
│   │   └── analytics/             # Модуль аналитики
│   │       ├── service.py         # Бизнес-логика аналитики
│   │       └── router.py          # REST API по аналитике
│   │
│   └── tests/                     # Тесты (pytest + httpx)
│       └── test_main.py
│
├── alembic/                       # Миграции базы данных
│   ├── env.py
│   └── versions/                  # Каталог версий миграций
│
├── Dockerfile                     # Docker образ для production
├── Dockerfile.dev                 # Docker образ для разработки
├── docker-compose.yml             # Конфигурация Docker Compose
├── .dockerignore                  # Исключения для Docker build
├── .env.example                   # Пример переменных окружения
├── Makefile                       # Команды для быстрого запуска
├── pyproject.toml                 # Конфигурация Poetry
└── README.md
```

## Архитектура

Проект использует модульную архитектуру, похожую на NestJS:

- **Core** - инфраструктурные компоненты (БД, безопасность, конфигурация)
- **Shared** - общие утилиты и базовые классы
- **Modules** - бизнес-модули с полным циклом (models → schemas → repository → service → router)

Каждый модуль следует принципу разделения ответственности:

- **models.py** - ORM модели (SQLAlchemy)
- **schemas.py** - DTO схемы (Pydantic)
- **repository.py** - работа с БД (CRUD операции)
- **service.py** - бизнес-логика
- **router.py** - HTTP endpoints (FastAPI)

## Разработка

### Быстрые команды (через Makefile):

**Локальная разработка:**

```bash
make install          # Установить зависимости
make dev              # Запустить в режиме разработки
make test             # Запустить тесты
make lint             # Проверить код линтером
make format           # Отформатировать код
make type-check       # Проверить типы
make check            # Запустить все проверки (lint + type-check)
make migrate          # Применить миграции БД
make migrate-create MESSAGE="описание"  # Создать новую миграцию
make migrate-downgrade # Откатить последнюю миграцию
make clean            # Очистить кэш и временные файлы
make db-reset         # Сбросить БД и применить миграции заново
make shell            # Активировать виртуальное окружение
```

**Docker команды:**

```bash
make docker-build          # Собрать Docker образы
make docker-up            # Запустить контейнеры
make docker-down          # Остановить контейнеры
make docker-restart       # Перезапустить контейнеры
make docker-logs          # Показать логи
make docker-shell         # Войти в контейнер приложения
make docker-migrate       # Применить миграции в Docker
make docker-migrate-create MESSAGE="описание"  # Создать миграцию в Docker
make docker-test          # Запустить тесты в Docker
make docker-clean         # Остановить и удалить все
```

### Детальные команды:

**Запуск тестов:**

```bash
make test
# или
poetry run pytest
```

**Форматирование кода:**

```bash
make format
# или
poetry run black app/
poetry run ruff check --fix app/
```

**Проверка типов:**

```bash
make type-check
# или
poetry run mypy app/
```

**Миграции БД:**

_Локально:_

```bash
make migrate-create MESSAGE="описание"
make migrate
make migrate-downgrade
```

_В Docker:_

```bash
make docker-migrate-create MESSAGE="описание"
make docker-migrate
```

_Напрямую через Alembic:_

```bash
poetry run alembic revision --autogenerate -m "description"
poetry run alembic upgrade head
poetry run alembic downgrade -1
```

## API Документация

После запуска приложения документация доступна по адресам:

- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc

### Базовые эндпоинты

- `GET /` - Главная страница
- `GET /health` - Проверка здоровья приложения
- `GET /docs` - Swagger UI документация
- `GET /redoc` - ReDoc документация

### Модули (в разработке)

Все модули имеют базовую структуру, но эндпоинты еще не реализованы:

- `/api/v1/users` - Модуль пользователей (TODO)
- `/api/v1/groups` - Модуль групп (TODO)
- `/api/v1/analytics` - Модуль аналитики (TODO)

## Зависимости

### Production

- FastAPI - веб-фреймворк
- SQLAlchemy - ORM
- Alembic - миграции БД
- Pydantic - валидация данных
- python-jose - JWT токены
- passlib - хэширование паролей
- psycopg2-binary - драйвер PostgreSQL

### Development

- pytest - тестирование
- black - форматирование кода
- ruff - линтер
- mypy - проверка типов

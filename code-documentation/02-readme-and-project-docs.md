# README и документация проекта / README and Project Docs

---

## RU

### README.md — лицо проекта

README — первое, что видит любой человек, открывший репозиторий. Он должен отвечать на вопросы: что это, зачем, как запустить.

---

### Структура хорошего README

```markdown
# Название проекта

Одно предложение: что это и зачем.

## Требования

- .NET 9 SDK
- PostgreSQL 15+
- Docker (опционально)

## Быстрый старт

\`\`\`bash
git clone https://github.com/you/myproject
cd myproject
cp appsettings.Example.json appsettings.Development.json
# Заполни ConnectionStrings в appsettings.Development.json
dotnet run --project src/MyProject.WebApi
\`\`\`

Приложение будет доступно на http://localhost:5000
Swagger UI: http://localhost:5000/swagger

## Конфигурация

| Параметр | Описание | Пример |
|---|---|---|
| `ConnectionStrings:Default` | Строка подключения к PostgreSQL | `Host=localhost;Database=mydb;...` |
| `Telegram:BotToken` | Токен Telegram-бота | `123456:ABC-DEF...` |

## Архитектура

Проект использует Clean Architecture. Подробнее: docs-vault/architecture/

## Разработка

Гайд для контрибьюторов: [CONTRIBUTING.md](CONTRIBUTING.md)

## Лицензия

MIT
```

---

### Что обязательно в README

```
✓ Что это за проект (1–2 предложения)
✓ Как запустить локально (конкретные команды, без "и так далее")
✓ Список требований (версии runtime, инструменты)
✓ Как настроить конфигурацию
✓ Ссылка на CONTRIBUTING.md (если проект командный)
```

```
✗ Не нужно: история компании, биографии авторов
✗ Не нужно: подробное описание каждого модуля (это в docs-vault/)
✗ Не нужно: копирование документации из кода
```

---

### appsettings.Example.json

Никогда не коммить реальные секреты. Вместо этого — пример файла конфигурации:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Port=5432;Database=mydb;Username=user;Password=REPLACE_ME"
  },
  "Telegram": {
    "BotToken": "REPLACE_ME"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```

Добавить в `.gitignore`:
```
appsettings.Development.json
appsettings.Production.json
appsettings.Local.json
```

Оставить в git только `appsettings.json` (публичные дефолты) и `appsettings.Example.json`.

---

### CONTRIBUTING.md — для команды

Документ, который читает любой, кто хочет внести изменение в проект.

```markdown
# Contributing

## Требования для разработки

- .NET 9 SDK
- PostgreSQL (или Docker)
- Rider / Visual Studio / VS Code

## Настройка окружения

1. Клонировать репозиторий
2. Скопировать `appsettings.Example.json` → `appsettings.Development.json`
3. Заполнить строку подключения к БД
4. Применить миграции: `dotnet ef database update --project src/MyProject.Infrastructure`
5. Запустить: `dotnet run --project src/MyProject.WebApi`

## Как работать с кодом

### Ветки
- `feature/название` — новый функционал
- `fix/название` — исправление бага
- `chore/название` — обновление зависимостей, CI

### Коммиты
Используем Conventional Commits. Подробнее: docs-vault/git/01-commit-messages.md

### Pull Requests
- Открывай PR в ветку `dev` (не в `main`)
- Описание PR: что сделано и почему
- Минимум 1 аппрув перед merge
- Автор PR сам разрешает конфликты

## Тесты

\`\`\`bash
dotnet test                           # все тесты
dotnet test --filter Category=Unit    # только unit
\`\`\`

Новый функционал — с тестами. Без тестов PR не принимается.

## Форматирование

\`\`\`bash
dotnet format                         # применить
dotnet format --verify-no-changes     # проверить без изменений
\`\`\`

CI проверяет форматирование автоматически.

## Архитектура проекта

Кратко: Clean Architecture, четыре слоя (Domain, Application, Infrastructure, WebApi).
Подробнее: docs-vault/architecture/
Принятые решения: docs-vault/adr/
```

---

### Архитектурный обзор (для онбординга)

Отдельный документ для нового разработчика — не теория, а конкретика:

```markdown
# Обзор архитектуры проекта

## Что делает система

MyProject — бот для рассылки объявлений в Telegram-каналы.
Администраторы создают объявления через Web UI или API.
Бот публикует их в настроенные каналы.

## Структура проекта

\`\`\`
src/
├── Domain/           Сущности: Announcement, Channel, User
├── Application/      Команды и хэндлеры (CQRS через MediatR)
├── Infrastructure/   EF Core + PostgreSQL, Telegram.Bot клиент
└── WebApi/           REST API, Swagger, авторизация (JWT)
\`\`\`

## Основной поток создания объявления

1. POST /announcements → AnnouncementsController
2. → MediatR → CreateAnnouncementHandler
3. → валидация (FluentValidation)
4. → сохранение в БД (IAnnouncementRepository)
5. → публикация через TelegramSender
6. ← 201 Created с Id

## Где что искать

| Задача | Где смотреть |
|---|---|
| Добавить эндпоинт | WebApi/Controllers/ |
| Добавить бизнес-логику | Application/Features/ |
| Изменить модель БД | Domain/Entities/ + новая миграция |
| Изменить работу с Telegram | Infrastructure/Telegram/ |

## Конфигурация

Все настройки через appsettings.json и переменные окружения.
Секреты через dotnet user-secrets (локально) или переменные окружения (сервер).

## Принятые решения

Все архитектурные решения зафиксированы в docs-vault/adr/
```

---

## EN

### README.md

The README is the first thing anyone sees when opening the repository. It must answer: what is this, why does it exist, how do I run it.

Required: project description (1–2 sentences), how to run locally (exact commands), requirements (runtime versions, tools), configuration guide, link to CONTRIBUTING.md.

Skip: company history, author biographies, detailed per-module descriptions (those go in docs-vault/).

### appsettings.Example.json

Never commit real secrets. Provide an example config file with placeholder values. Add actual config files to `.gitignore`. Only `appsettings.json` (public defaults) and `appsettings.Example.json` should be in git.

### CONTRIBUTING.md

Covers: dev environment setup (exact steps), branching convention, commit format, PR process (target branch, description, review requirements), test requirements, formatting commands.

### Architecture Overview

A concrete, project-specific document for onboarding — not theory. Includes: what the system does, project structure, main data flow through the system (with specific class names), where to find things (lookup table), configuration approach, link to ADR.

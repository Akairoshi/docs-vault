# Что сделать до написания первой строки / Before You Write a Single Line

---

## RU

### Почему это важно

Большинство проблем в проектах не возникают во время написания кода. Они закладываются **до** него: непонятные требования, неверно выбранная архитектура, отсутствие договорённостей в команде.

Час, потраченный на подготовку, экономит дни на исправление.

---

### Шаг 1 — Понять задачу

Прежде чем что-то проектировать, ответь на вопросы:

**Что это за система?**
- Для кого она? (конечные пользователи, другие сервисы, внутренняя команда)
- Какую проблему она решает?
- Что будет считаться успехом?

**Функциональные требования:**
Что система _делает_. Перечисли конкретные действия:
- Пользователь может зарегистрироваться
- Администратор может создать объявление
- Система отправляет уведомление при новом сообщении

**Нефункциональные требования:**
Как система _работает_:
- Сколько пользователей одновременно?
- Какое время отклика приемлемо?
- Нужна ли высокая доступность (uptime)?
- Есть ли требования к безопасности (GDPR, аутентификация)?
- Нужны ли аудит и логирование?

**Для одного разработчика:** запиши ответы в `docs-vault/README.md` или отдельный `BRIEF.md`. Через месяц ты скажешь себе спасибо.

**Для команды:** проведи встречу, зафиксируй ответы, дай всем прочитать и согласовать. Разногласия в понимании требований на этом этапе — норма; разногласия в разгар разработки — катастрофа.

---

### Шаг 2 — Определить границы системы

Нарисуй (буквально, на листе или в draw.io) что входит в систему, а что нет.

```
Граница системы:
┌──────────────────────────────────────┐
│          MyApp                       │
│  ┌──────────┐   ┌─────────────────┐  │
│  │  WebAPI  │   │   Background    │  │
│  │          │   │   Worker        │  │
│  └──────────┘   └─────────────────┘  │
└──────────────────────────────────────┘
         │                │
    ┌────▼────┐     ┌──────▼──────┐
    │  Database│    │  Telegram   │
    │ (внешнее)│    │  API (внешнее)│
    └──────────┘    └─────────────┘
```

Что в системе — твоя ответственность. Что снаружи — внешние зависимости, которые могут упасть, измениться, стоить денег.

---

### Шаг 3 — Выбрать архитектуру

На основе требований выбери подход. Зафиксируй в ADR (см. `docs-vault/adr/`).

Подсказка для большинства новых проектов:
- Команда 1–3 человека, старт → Монолит + Clean Architecture
- Сложный домен → Clean Architecture + CQRS
- CRUD без сложной логики → Sliced Architecture или Minimal API

---

### Шаг 4 — Настроить рабочее окружение

До первого коммита:

```bash
# Создать .gitignore
dotnet new gitignore

# Инициализировать git
git init
git add .
git commit -m "chore: initial project setup"

# Создать .editorconfig для единого форматирования
```

Пример `.editorconfig` для C#:

```ini
root = true

[*.cs]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

---

### Шаг 5 — Договориться о соглашениях (для команды)

До первого коммита команда должна договориться о:

| Тема | Пример соглашения |
|---|---|
| Ветки | `main`, `dev`, `feature/*`, `fix/*` |
| Коммиты | Conventional Commits (см. `docs-vault/git/`) |
| Именование | PascalCase для классов, camelCase для переменных |
| Язык кода | Английский (идентификаторы, комментарии) |
| Code review | Минимум 1 аппрув перед merge |
| Тесты | Новый функционал — с тестами |

Зафиксируй в `CONTRIBUTING.md` в корне репозитория.

---

### Шаг 6 — Создать скелет проекта

Не пиши функциональный код, пока нет скелета. Скелет — это:
- Структура папок и проектов
- Подключение DI, конфигурации, логирования
- Одна заглушка-endpoint, которая возвращает `200 OK`
- CI/CD pipeline (хотя бы сборка и запуск тестов)

Только после этого — пиши функциональный код.

---

## EN

### Why This Matters

Most project problems don't occur during coding. They are built in **before** it: unclear requirements, wrong architecture choice, lack of team agreements.

One hour spent on preparation saves days of fixes.

### Step 1 — Understand the Task

**Functional requirements:** What the system _does_. List concrete actions.

**Non-functional requirements:** How the system _behaves_: how many concurrent users, acceptable response time, uptime, security requirements (GDPR, authentication), audit and logging needs.

**Solo developer:** Write answers in `docs-vault/README.md` or a separate `BRIEF.md`. You'll thank yourself in a month.

**Team:** Hold a meeting, document the answers, let everyone read and agree. Disagreements about requirements at this stage are normal; disagreements mid-development are a disaster.

### Step 2 — Define System Boundaries

Draw (literally, on paper or in draw.io) what is inside the system and what isn't. What's inside is your responsibility. What's outside are external dependencies that can fail, change, or cost money.

### Step 3 — Choose an Architecture

Based on requirements, choose an approach. Record in an ADR (see `docs-vault/adr/`).

### Step 4 — Set Up the Working Environment

Before the first commit: create `.gitignore`, initialize git, create `.editorconfig` for consistent formatting.

### Step 5 — Agree on Conventions (for teams)

Before the first commit, the team must agree on: branching strategy, commit format, naming conventions, code language (English for identifiers), code review requirements, test policy.

Document in `CONTRIBUTING.md` at the root of the repository.

### Step 6 — Create the Project Skeleton

Don't write functional code until there's a skeleton: folder and project structure, DI/config/logging wired up, one stub endpoint returning `200 OK`, CI/CD pipeline (at minimum: build and run tests).

Only after this — write functional code.

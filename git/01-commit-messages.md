# Сообщения коммитов / Commit Messages

---

## RU

### Почему это важно

Коммит — это не просто сохранение. Это запись в историю проекта. Через год ты (или другой разработчик) будет читать `git log` и пытаться понять, почему код выглядит именно так.

Плохая история:
```
fix
wip
changes
asdfg
fix 2
ok now works
```

Хорошая история:
```
feat: add announcement creation endpoint
fix: handle empty channel ID in validator
refactor: extract notification sender to separate service
docs: add deployment instructions to README
chore: update EF Core to 9.0.1
```

---

### Conventional Commits

Стандарт, который используется в большинстве профессиональных проектов.

Формат:
```
<тип>(<область>): <описание>

[тело — необязательно]

[футер — необязательно]
```

**Типы:**

| Тип | Когда использовать |
|---|---|
| `feat` | Новая функциональность |
| `fix` | Исправление бага |
| `refactor` | Рефакторинг (не баг, не фича) |
| `docs` | Изменения в документации |
| `test` | Добавление или исправление тестов |
| `chore` | Обновление зависимостей, конфигурация, CI |
| `style` | Форматирование, пробелы (не логика) |
| `perf` | Улучшение производительности |
| `ci` | Изменения в CI/CD |
| `build` | Изменения системы сборки |
| `revert` | Откат предыдущего коммита |

---

### Правила хорошего сообщения

**1. Первая строка — краткое описание, до 72 символов**

```
# Хорошо
feat: add user registration endpoint

# Плохо — слишком длинно
feat: add the user registration endpoint that handles email validation and sends a confirmation email
```

**2. Используй повелительное наклонение (что делает коммит, а не что ты сделал)**

```
# Хорошо
fix: handle null reference in user service

# Плохо
fix: fixed null reference in user service
fix: fixing null reference in user service
```

**3. Тело коммита — объясняй ПОЧЕМУ, не ЧТО**

ЧТО — видно из diff. ПОЧЕМУ — не видно нигде, кроме сообщения коммита.

```
fix: use UTC for all timestamp comparisons

Previously, comparisons used local server time, which caused
incorrect expiration checks when server timezone differed
from UTC. All stored timestamps are UTC, so comparisons
must use DateTime.UtcNow.
```

**4. Футер — для ссылок на задачи и breaking changes**

```
feat: add role-based access control

Closes #42
BREAKING CHANGE: /api/users now requires Authorization header
```

---

### Примеры реальных коммитов

```bash
# Новая функция
git commit -m "feat(announcements): add POST /announcements endpoint"

# Баг
git commit -m "fix(auth): return 401 instead of 500 on invalid token"

# Рефакторинг
git commit -m "refactor(notifications): extract INotificationSender interface"

# Зависимости
git commit -m "chore: update packages to latest stable versions"

# Тест
git commit -m "test(announcements): add unit tests for CreateAnnouncementHandler"

# Документация
git commit -m "docs: add API authentication guide to README"
```

---

### Область (scope)

Область — необязательна, но полезна в больших проектах. Это название модуля, компонента или слоя, которого касается коммит:

```
feat(auth): ...
fix(announcements): ...
refactor(infrastructure): ...
chore(ci): ...
```

---

### Атомарные коммиты

Один коммит = одно логическое изменение.

```
# Плохо — один большой коммит на всё
git commit -m "feat: add announcements, fix auth bug, update readme"

# Хорошо — три отдельных коммита
git commit -m "feat: add announcement creation endpoint"
git commit -m "fix(auth): handle expired token gracefully"
git commit -m "docs: update README with new API endpoints"
```

Если тебе трудно описать коммит одной строкой — возможно, он делает слишком много.

---

### Практика для одного разработчика

Даже в одиночку коммиты имеют смысл:
- `git log` становится дневником разработки
- `git bisect` работает эффективно (поиск коммита, который сломал что-то)
- Легче делать `git revert` конкретного изменения

Минимальная дисциплина: не коммить «всё сразу в конце дня». Коммить каждое законченное логическое изменение.

---

### Практика для команды

- Настройте `commitlint` или `husky` для автоматической проверки формата коммитов
- Ведите `CHANGELOG.md`, который генерируется из коммитов (инструмент: `conventional-changelog`)
- Договоритесь о языке сообщений (EN рекомендуется — universal)

---

## EN

### Why It Matters

A commit is a record in the project's history. A year from now, you (or another developer) will read `git log` trying to understand why the code looks the way it does.

### Conventional Commits

Standard format: `<type>(<scope>): <description>`

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`, `perf`, `ci`, `build`, `revert`.

### Rules for Good Messages

1. First line is a short summary, under 72 characters.
2. Use imperative mood: "add feature" not "added feature" or "adds feature."
3. Body explains WHY, not WHAT — the diff shows what changed.
4. Footer references issues and breaking changes.

### Atomic Commits

One commit = one logical change. If you can't describe it in one line, it probably does too much.

### Solo vs Team

**Solo:** git log becomes a development diary. git bisect works effectively. Easier to revert specific changes. Commit each finished logical change — not everything at end of day.

**Team:** Use `commitlint` or `husky` for automatic format validation. Generate `CHANGELOG.md` from commits. Agree on language (English recommended).

# Рабочий процесс с Git / Git Workflow

---

## RU

### Полный цикл от задачи до merge

Это практический гайд — последовательность команд и решений в реальной работе.

---

### 1. Начало работы над задачей

```bash
# Убедиться, что у тебя актуальный main
git checkout main
git pull origin main

# Создать ветку
git checkout -b feature/add-announcement-endpoint
```

---

### 2. Работа: коммить часто, атомарно

Не копи изменения. Каждое законченное логическое действие — отдельный коммит.

```bash
# Посмотреть что изменилось
git status
git diff

# Добавить конкретные файлы (не git add . без проверки)
git add src/MyProject.Application/Announcements/CreateAnnouncementCommand.cs
git add src/MyProject.Application/Announcements/CreateAnnouncementHandler.cs
git commit -m "feat(announcements): add create announcement command and handler"

# Следующее изменение
git add src/MyProject.WebApi/Controllers/AnnouncementsController.cs
git commit -m "feat(announcements): add POST /announcements endpoint"

# Тесты
git add tests/MyProject.Application.Tests/Announcements/CreateAnnouncementHandlerTests.cs
git commit -m "test(announcements): add unit tests for create handler"
```

---

### 3. Держать ветку актуальной

Если разработка занимает несколько дней — регулярно подтягивай изменения из main:

```bash
git fetch origin
git rebase origin/main
```

`rebase` предпочтительнее `merge` для поддержания чистой истории.

Если возник конфликт:
```bash
# Git покажет конфликтующие файлы
# Открой их, разреши конфликт вручную, затем:
git add <файл>
git rebase --continue

# Если хочешь отменить rebase:
git rebase --abort
```

---

### 4. Перед открытием Pull Request: привести историю в порядок

Иногда в процессе работы накапливаются «черновые» коммиты. Перед PR их можно схлопнуть:

```bash
# Интерактивный rebase последних N коммитов
git rebase -i HEAD~3
```

Откроется редактор, где для каждого коммита можно выбрать:
- `pick` — оставить как есть
- `squash` / `s` — слить с предыдущим
- `reword` / `r` — изменить сообщение
- `drop` / `d` — удалить коммит

**Осторожно:** не делай rebase коммитов, которые уже запушены и видны другим.

---

### 5. Отправить ветку и открыть Pull Request

```bash
git push origin feature/add-announcement-endpoint
```

Открой PR на GitHub/GitLab. Описание PR:

```markdown
## Что сделано
Добавлен эндпоинт `POST /announcements` для создания объявлений.

## Почему
Задача #47: пользователи должны иметь возможность создавать объявления через API.

## Как проверить
1. Запустить проект
2. POST /announcements с телом `{"title": "Test", "body": "Body", "channelId": 1}`
3. Ожидаемый ответ: 201 Created с Id нового объявления

## Особое внимание
Валидация channelId — проверяет существование канала в БД перед сохранением.
```

---

### 6. После merge: cleanup

```bash
# Вернуться на main
git checkout main
git pull origin main

# Удалить локальную ветку
git branch -d feature/add-announcement-endpoint

# Удалить remote ветку (если не удалилась автоматически)
git push origin --delete feature/add-announcement-endpoint
```

---

### Полезные команды на каждый день

```bash
# Посмотреть историю красиво
git log --oneline --graph --all

# Найти коммит, который сломал что-то (бинарный поиск)
git bisect start
git bisect bad                  # текущий коммит — сломан
git bisect good <хэш>           # этот коммит — работал
# Git будет переключаться между коммитами, ты отвечаешь good/bad
git bisect reset                # закончить bisect

# Отменить последний коммит (сохранив изменения в рабочей директории)
git reset HEAD~1

# Временно спрятать незакоммиченные изменения
git stash
git stash pop                   # вернуть обратно

# Посмотреть кто написал эту строку и когда
git blame src/MyProject.WebApi/Controllers/AnnouncementsController.cs

# Найти коммит по тексту сообщения
git log --grep="announcement"

# Найти в истории по изменению кода
git log -S "CreateAnnouncementHandler"
```

---

### Hotfix: срочное исправление в продакшне

```bash
# Бранчуемся от main (не от dev!)
git checkout main
git pull origin main
git checkout -b hotfix/fix-null-in-validator

# Фиксим, коммитим
git commit -m "fix(validator): handle null channel ID without throwing"

# Пушим, открываем PR прямо в main
git push origin hotfix/fix-null-in-validator
# → merge в main → deploy

# Синхронизируем dev (если используется)
git checkout dev
git merge main
```

---

### .gitignore для .NET

```gitignore
# Build artifacts
bin/
obj/
*.user
*.suo

# Visual Studio
.vs/
*.rsuser

# JetBrains Rider
.idea/
*.sln.iml

# Secrets
appsettings.Production.json
appsettings.Local.json
*.env
.env

# Logs
logs/
*.log

# OS
.DS_Store
Thumbs.db
```

Генерация: `dotnet new gitignore` создаёт стандартный .gitignore для .NET.

---

## EN

### Full Cycle from Task to Merge

**Start:** pull latest main, create feature branch.

**Work:** commit often and atomically. Each logical unit = one commit. Use `git status` and `git diff` before every `git add`.

**Stay current:** `git fetch origin && git rebase origin/main` regularly when the branch lives more than a day.

**Before PR:** use `git rebase -i` to clean up draft commits. Don't rebase commits already visible to others.

**Open PR:** description explains what, why, and how to verify.

**After merge:** pull main, delete local and remote feature branch.

### Useful Daily Commands

`git log --oneline --graph --all` — visual history.
`git bisect` — binary search for the breaking commit.
`git reset HEAD~1` — undo last commit, keep changes.
`git stash` / `git stash pop` — temporarily hide uncommitted changes.
`git blame` — see who wrote which line and when.
`git log --grep` / `git log -S` — search history by message or code change.

### Hotfix

Branch from `main` (not `dev`). Fix, commit, PR directly to `main`, deploy. Then sync `dev` from `main`.

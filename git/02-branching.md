# Стратегии веток / Branching Strategies

---

## RU

### Зачем нужны ветки

Ветка — это изолированная линия разработки. Ты работаешь над фичей, не трогая стабильный код. Если что-то пошло не так — ветку можно удалить, и ничего не сломается.

---

### Стратегия 1: Feature Branch Workflow (рекомендуется для большинства)

Подходит для: команды от 1 до ~15 человек, одна основная кодовая база.

```
main ──────────────────────────────────────────► (всегда стабильный)
        │                        │
        └── feature/auth ────────┘  (merge через PR)
             └── feature/announcements ──────────┘
```

**Правила:**
- `main` всегда в рабочем состоянии, всегда можно задеплоить
- Каждая задача — отдельная ветка
- Ветка живёт максимум несколько дней
- Merge только через Pull Request (даже для одного разработчика — хорошая привычка)

**Именование веток:**

```
feature/add-announcement-endpoint
fix/null-reference-in-validator
chore/update-ef-core
docs/add-deployment-guide
refactor/extract-notification-service
```

**Жизненный цикл ветки:**

```bash
# Создать ветку
git checkout main
git pull origin main
git checkout -b feature/add-announcements

# Работа...
git add .
git commit -m "feat(announcements): add domain entity"
git commit -m "feat(announcements): add create handler"
git commit -m "test(announcements): add unit tests for handler"

# Отправить на remote
git push origin feature/add-announcements

# Открыть Pull Request → code review → merge
# Удалить ветку после merge
git branch -d feature/add-announcements
git push origin --delete feature/add-announcements
```

---

### Стратегия 2: Git Flow

Подходит для: продуктов с регулярными релизами (раз в 2 недели, раз в месяц).

```
main ─────────────────────────────────────────► (релизы)
      │                                  │
dev ──┼──────────────────────────────────┼─────► (текущая разработка)
      │           │              │       │
      └─ feature ─┘    release ─┘  hotfix ─┘
```

Ветки:
- `main` — только стабильные релизы с тегами
- `dev` — текущая разработка
- `feature/*` — фичи, бранчуются от `dev`, мержатся в `dev`
- `release/*` — подготовка релиза (баги, версии), мержится в `main` и `dev`
- `hotfix/*` — срочные фиксы в продакшне, бранчуются от `main`

**Когда использовать:** если у вас версионированный продукт и несколько параллельных релизов.

**Когда не использовать:** если деплой идёт непрерывно (continuous delivery) — Git Flow добавляет лишнюю сложность.

---

### Стратегия 3: Trunk-Based Development

Подходит для: зрелые команды, continuous delivery, много автоматизации.

```
main ────────────────────────────────────────────► (единственная ветка)
        ↑     ↑      ↑      ↑      ↑
      коммит  ↑    коммит коммит коммит
         short-lived feature branch (< 1 дня)
```

Разработчики коммитят прямо в `main` или через очень короткоживущие ветки (часы, не дни). Требует:
- Мощного CI (каждый коммит проходит все тесты)
- Feature flags (незаконченный код скрыт за флагом)
- Высокой культуры тестирования

**Не рекомендуется** без этих условий.

---

### Для одного разработчика

Feature Branch Workflow работает и в одиночку:
- Дисциплинирует: фича делается отдельно, `main` всегда рабочий
- Pull Request на самого себя — способ сделать финальную проверку перед merge
- История в `main` чистая

Минимальная версия: просто ветки без строгих PR:
```bash
git checkout -b feature/my-thing
# работа
git checkout main
git merge feature/my-thing --no-ff
git branch -d feature/my-thing
```

`--no-ff` (no fast-forward) создаёт merge commit, который сохраняет историю «эта группа коммитов — одна фича».

---

### Для команды

Договоритесь о:
- Какую стратегию используете
- Как называются ветки (формат)
- Кто может мержить в `main`
- Нужна ли защита ветки `main` (branch protection rules на GitHub/GitLab)

Настройка защиты `main` на GitHub: Settings → Branches → Add rule → Require pull request reviews.

---

## EN

### Feature Branch Workflow (recommended for most)

Suitable for: teams of 1 to ~15 people, single codebase.

`main` is always stable and deployable. Each task gets its own branch. Branches live for at most a few days. Merge only through Pull Requests.

Branch naming: `feature/`, `fix/`, `chore/`, `docs/`, `refactor/` prefix followed by a short description.

### Git Flow

Suitable for: products with regular releases (bi-weekly, monthly).

Branches: `main` (tagged releases), `dev` (current development), `feature/*` (from dev, back to dev), `release/*` (prep work), `hotfix/*` (from main).

Use when: versioned product, multiple parallel releases.
Don't use when: continuous delivery — Git Flow adds unnecessary complexity.

### Trunk-Based Development

Suitable for: mature teams, continuous delivery, high automation.

Developers commit directly to `main` or through very short-lived branches (hours, not days). Requires strong CI, feature flags, and a strong testing culture.

### Solo Developer

Feature Branch Workflow works solo too. It disciplines you: features are isolated, `main` stays working. Use `--no-ff` on merge to preserve feature grouping in history.

### Team Setup

Agree on strategy, branch naming format, who can merge to `main`, and enable branch protection rules on GitHub/GitLab.

# Настройка проекта / Project Setup

---

## RU

### Структура Solution в .NET

Правильная структура проекта закладывается один раз и потом живёт годами. Плохая структура — это постоянный источник путаницы.

Стандартная структура для .NET проекта с Clean Architecture:

```
MyProject/
├── src/
│   ├── MyProject.Domain/           # Сущности, интерфейсы, исключения
│   ├── MyProject.Application/      # Use cases, DTO, валидация
│   ├── MyProject.Infrastructure/   # EF Core, внешние сервисы, реализации
│   └── MyProject.WebApi/           # Controllers, Middleware, Program.cs
├── tests/
│   ├── MyProject.Domain.Tests/
│   ├── MyProject.Application.Tests/
│   └── MyProject.Integration.Tests/
├── docs/                           # Документация (или docs-vault/)
├── .editorconfig
├── .gitignore
├── README.md
├── CONTRIBUTING.md                 # Соглашения для команды
└── MyProject.sln
```

Создать скелет:

```bash
mkdir MyProject && cd MyProject
dotnet new sln -n MyProject

# Проекты
dotnet new classlib -n MyProject.Domain -o src/MyProject.Domain
dotnet new classlib -n MyProject.Application -o src/MyProject.Application
dotnet new classlib -n MyProject.Infrastructure -o src/MyProject.Infrastructure
dotnet new webapi -n MyProject.WebApi -o src/MyProject.WebApi

# Тесты
dotnet new xunit -n MyProject.Domain.Tests -o tests/MyProject.Domain.Tests
dotnet new xunit -n MyProject.Application.Tests -o tests/MyProject.Application.Tests
dotnet new xunit -n MyProject.Integration.Tests -o tests/MyProject.Integration.Tests

# Добавить все в solution
dotnet sln add src/MyProject.Domain
dotnet sln add src/MyProject.Application
dotnet sln add src/MyProject.Infrastructure
dotnet sln add src/MyProject.WebApi
dotnet sln add tests/MyProject.Domain.Tests
dotnet sln add tests/MyProject.Application.Tests
dotnet sln add tests/MyProject.Integration.Tests

# Ссылки между проектами
dotnet add src/MyProject.Application reference src/MyProject.Domain
dotnet add src/MyProject.Infrastructure reference src/MyProject.Application
dotnet add src/MyProject.WebApi reference src/MyProject.Infrastructure
dotnet add src/MyProject.WebApi reference src/MyProject.Application
dotnet add tests/MyProject.Application.Tests reference src/MyProject.Application
```

---

### Git: начало

```bash
git init
dotnet new gitignore   # стандартный .gitignore для .NET
git add .
git commit -m "chore: initial project structure"
```

Если проект командный — сразу создай репозиторий на GitHub/GitLab и настрой remote:

```bash
git remote add origin https://github.com/yourname/myproject.git
git branch -M main
git push -u origin main
```

---

### Ветки

Стандартная стратегия для большинства проектов:

| Ветка | Назначение |
|---|---|
| `main` | Стабильный продакшн-код |
| `dev` | Текущая разработка (если нужен промежуточный буфер) |
| `feature/название` | Новая функциональность |
| `fix/название` | Исправление бага |
| `chore/название` | Технические задачи (обновление зависимостей, CI и т.п.) |

**Один разработчик:** можно работать прямо в `main` для мелких правок, но ветки для фич — хорошая привычка.

**Команда:** `main` защищён от прямых пушей. Всё через Pull Request с code review.

---

### .editorconfig

Файл `.editorconfig` в корне репозитория унифицирует форматирование у всех разработчиков независимо от IDE.

```ini
root = true

[*]
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.cs]
indent_style = space
indent_size = 4

[*.{json,yml,yaml}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

---

### Directory.Build.props

Файл в корне Solution для общих настроек всех проектов — чтобы не повторять одно и то же в каждом `.csproj`:

```xml
<!-- Directory.Build.props -->
<Project>
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <LangVersion>latest</LangVersion>
  </PropertyGroup>
</Project>
```

`TreatWarningsAsErrors` — важная настройка. Предупреждения компилятора — это ранние сигналы о проблемах. Если их игнорировать, они накапливаются.

---

### appsettings и секреты

Никогда не коммить секреты (пароли, ключи API, строки подключения к БД) в репозиторий.

```
appsettings.json          ← в git (только публичные настройки)
appsettings.Development.json  ← в git (без секретов)
appsettings.Production.json   ← НЕ в git, или пустой шаблон
```

Для разработки используй `dotnet user-secrets`:

```bash
cd src/MyProject.WebApi
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:Default" "Host=localhost;Database=mydb;Username=user;Password=secret"
```

Секреты хранятся вне папки проекта и не попадают в git.

---

### CONTRIBUTING.md

Если в проекте будет больше одного человека, создай `CONTRIBUTING.md` в корне:

```markdown
# Contributing

## Ветки
- Новый функционал: `feature/название`
- Баг: `fix/название`

## Коммиты
Используем Conventional Commits. Подробнее: docs-vault/git/

## Code Review
- Минимум 1 аппрув перед merge
- Автор PR сам разрешает конфликты

## Тесты
- Новый код — с тестами
- Запуск: `dotnet test`

## Форматирование
- .editorconfig применяется автоматически
- Проверь: `dotnet format --verify-no-changes`
```

---

## EN

### Solution Structure

Standard structure for a .NET project with Clean Architecture — src/ for production code (Domain, Application, Infrastructure, WebApi), tests/ for test projects, docs/ for documentation.

Use `dotnet new sln`, `dotnet new classlib`, `dotnet new webapi`, `dotnet new xunit` to scaffold. Wire references with `dotnet add reference`.

### Git Setup

Initialize git, use `dotnet new gitignore` for a standard .NET gitignore, make an initial commit, push to remote.

Branch strategy: `main` (stable), `dev` (optional buffer), `feature/*`, `fix/*`, `chore/*`.

### .editorconfig

Place at repository root to unify formatting across all developers regardless of IDE.

### Directory.Build.props

Shared MSBuild settings at solution root. Enables `Nullable`, `ImplicitUsings`, `TreatWarningsAsErrors` for all projects without repeating in each `.csproj`.

### Secrets

Never commit secrets. Use `dotnet user-secrets` for local development. Use environment variables or a secrets manager in production.

### CONTRIBUTING.md

Document branching, commit format, code review requirements, test policy, and formatting rules for the team.

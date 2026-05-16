# Чеклисты / Checklists

---

## RU

Чеклисты применяются в конкретные моменты: перед стартом работы, перед PR, перед деплоем. Не нужно проходить все каждый раз — выбирай подходящий момент.

---

### Чеклист: перед тем как взять задачу

```
[ ] Я понимаю, что нужно сделать (могу объяснить одним предложением)
[ ] Я знаю критерий готовности (как понять, что сделано)
[ ] Я понимаю, что затронет это изменение
[ ] У меня нет блокеров (зависимостей, которых нет)
[ ] Задача достаточно мала, чтобы завершить её за 1–2 дня
    (если нет — разбей на подзадачи)
```

---

### Чеклист: перед написанием кода

```
[ ] Я набросал план: что делаю, какие файлы затрагиваю
[ ] Я понял граничные случаи (пустые данные, нулевые значения, ошибки сети)
[ ] Я создал ветку от актуального main/dev
[ ] Я знаю, как буду тестировать результат
```

---

### Чеклист: перед открытием Pull Request

```
[ ] dotnet build — без ошибок и предупреждений
[ ] dotnet test — все тесты зелёные
[ ] dotnet format --verify-no-changes — форматирование в порядке
[ ] Нет закомментированного кода
[ ] Нет Console.WriteLine / Debug.WriteLine оставленных случайно
[ ] Нет hardcoded строк: паролей, URL, ключей, конфигов
[ ] Нет файлов, которые не должны быть в PR (appsettings.Production, .env)
[ ] Описание PR: что сделано и зачем, не просто пересказ кода
[ ] Коммиты осмысленные (не "fix", не "wip", не "asd")
[ ] Я сам прочитал свой diff перед отправкой
```

---

### Чеклист: перед деплоем на сервер

```
[ ] Все тесты прошли в CI
[ ] Миграции БД готовы и проверены
[ ] Переменные окружения настроены на сервере
[ ] Секреты НЕ в коде и НЕ в репозитории
[ ] Есть резервная копия БД (перед применением миграций)
[ ] Rollback-план: что делать, если что-то пойдёт не так
[ ] Команда знает о деплое (если проект командный)
[ ] Health check работает после деплоя
```

---

### Чеклист: code review (ревьювер)

```
[ ] Логика правильная (не только стиль)
[ ] Граничные случаи обработаны
[ ] Нет явных проблем с безопасностью (SQL injection, открытые эндпоинты, секреты)
[ ] Нет утечек памяти или ресурсов (незакрытые соединения, потоки)
[ ] Тесты есть и они проверяют именно то, что нужно
[ ] Наименования понятны без пояснений
[ ] Изменение не нарушает архитектурных границ
```

---

### Чеклист: онбординг нового разработчика

```
[ ] Новый человек прочитал README.md
[ ] Новый человек прочитал CONTRIBUTING.md
[ ] Настроено рабочее окружение (dotnet, IDE, git)
[ ] Есть доступ к репозиторию
[ ] Есть доступ к трекеру задач
[ ] Есть доступ к dev-окружению (БД, конфигурация)
[ ] Проведена короткая экскурсия по архитектуре проекта
[ ] Первая задача — небольшая и некритичная
```

---

## EN

Checklists apply at specific moments: before picking up a task, before writing code, before opening a PR, before deployment, during code review, and when onboarding a new developer.

Use the appropriate checklist for the moment — you don't need all of them every time.

**Before taking a task:** understand what needs to be done, know the definition of done, identify dependencies and blockers, confirm the task is small enough to finish in 1–2 days.

**Before writing code:** sketch a plan, identify edge cases, create a branch from current main/dev.

**Before opening a PR:** build passes, tests pass, formatting clean, no leftover debug output, no hardcoded secrets, PR description explains context not just what changed, read your own diff.

**Before deployment:** CI green, migrations ready, environment variables configured, secrets out of code, database backup exists, rollback plan defined.

**During code review:** logic is correct, edge cases handled, no security issues, no resource leaks, tests cover the right things, names are clear without explanation, architectural boundaries respected.

**Onboarding:** README and CONTRIBUTING read, environment set up, repository and tracker access granted, short architecture walkthrough given, first task is small and non-critical.

# Что документировать / What to Document

---

## RU

### Уровни документации

Документация существует на нескольких уровнях. Каждый уровень отвечает на свой вопрос.

| Уровень | Что описывает | Где живёт |
|---|---|---|
| Проект | Зачем проект, как запустить, как контрибьютить | README.md, CONTRIBUTING.md |
| Архитектура | Почему такая структура, какие решения приняты | docs-vault/adr/, docs-vault/architecture/ |
| Модуль / компонент | Что делает этот кусок системы | Папка с README или заголовочный комментарий |
| Публичный API | Что принимает/возвращает каждый метод | XML-документация (///) |
| Сложная логика | Почему код написан именно так | Inline-комментарии (//) |

---

### Что точно нужно документировать

**Публичные интерфейсы и их методы**

Интерфейс — это контракт. Документируй каждый метод: что он делает, что принимает, что возвращает, что бросает.

```csharp
/// <summary>
/// Репозиторий для управления объявлениями.
/// </summary>
public interface IAnnouncementRepository
{
    /// <summary>
    /// Возвращает объявление по идентификатору.
    /// </summary>
    /// <returns>Объявление или <c>null</c> если не найдено.</returns>
    Task<Announcement?> GetByIdAsync(int id, CancellationToken ct = default);

    /// <summary>
    /// Возвращает страницу объявлений, отсортированных по дате создания (убыв.).
    /// </summary>
    /// <param name="page">Номер страницы, начиная с 1.</param>
    /// <param name="pageSize">Размер страницы. Максимум 100.</param>
    Task<PagedResult<Announcement>> GetPagedAsync(int page, int pageSize, CancellationToken ct = default);
}
```

**Нетривиальные бизнес-правила**

Если правило не очевидно — объясни. Правило «пользователь не может создать объявление в канале, в котором он заблокирован» не вытекает автоматически из кода.

```csharp
// Проверяем статус пользователя в канале до создания объявления.
// Заблокированный пользователь получает Result.Failure, а не исключение,
// чтобы контроллер мог вернуть 403, а не 500.
var membership = await _channelRepo.GetMembershipAsync(command.UserId, command.ChannelId, ct);
if (membership?.Status == MembershipStatus.Banned)
    return Result.Failure("User is banned in this channel");
```

**Внешние зависимости и их особенности**

```csharp
// Telegram Bot API ограничивает частоту: 30 сообщений/сек глобально
// и 1 сообщение/сек в один чат. При превышении — ошибка 429.
// Задержка определяется полем retry_after в ответе API.
```

**Конфигурационные параметры**

```csharp
public class TelegramOptions
{
    /// <summary>
    /// Токен бота. Получается в @BotFather. Формат: <c>123456:ABC-DEF...</c>
    /// </summary>
    public string BotToken { get; init; } = string.Empty;

    /// <summary>
    /// Таймаут polling-запроса в секундах. Рекомендуемое значение: 25–60.
    /// Значение 0 отключает long polling (только short polling).
    /// </summary>
    public int PollingTimeout { get; init; } = 30;
}
```

---

### Что НЕ нужно документировать

```csharp
// Плохо — очевидно из кода
/// <summary>
/// Возвращает Id.
/// </summary>
public int Id { get; set; }

// Плохо — просто пересказ имени метода
/// <summary>
/// Метод отправки сообщения.
/// </summary>
public async Task SendMessageAsync(string message) { ... }
```

Если документация ничего не добавляет к имени — она не нужна.

---

### Документация для одного разработчика

Даже работая один, документируй:

- **README.md** — как запустить проект. Через полгода ты забудешь.
- **ADR** — почему принял то или иное решение.
- **XML-документация** — для публичных методов в библиотечных слоях. IntelliSense будет показывать подсказки.
- **Комментарии** — для нетривиальной логики.

Не нужно одному: формальные описания процессов (CONTRIBUTING.md), если ты единственный разработчик.

---

### Документация для команды

Команде дополнительно нужны:

- **CONTRIBUTING.md** — соглашения, процесс, как сделать PR
- **Архитектурный обзор** — документ с объяснением структуры проекта для онбординга
- **Runbook** — что делать при типичных инцидентах (как перезапустить сервис, где смотреть логи)
- **API документация** — Swagger/OpenAPI, который актуален автоматически

Принцип: документация должна быть рядом с кодом, чтобы не устаревала. Swagger генерируется из кода. XML-документация — часть кода. ADR — в репозитории рядом с кодом.

---

### Что быстро устаревает (избегай)

- Документы в Confluence или Notion, которые обновляются вручную
- Диаграммы в отдельных файлах без привязки к коду
- Подробные описания «как работает метод X» — они всегда отстают от кода

Пиши документацию, которая автоматически остаётся актуальной (генерируется из кода) или очень медленно устаревает (архитектурные решения, концепции).

---

## EN

### Documentation Levels

Documentation exists at multiple levels, each answering a different question:

Project level (README, CONTRIBUTING) — why the project exists, how to run it, how to contribute.
Architecture level (ADR, architecture docs) — why this structure, what decisions were made.
Module/component level — what this part of the system does.
Public API level (XML docs) — what each method accepts, returns, throws.
Complex logic level (inline comments) — why the code is written this way.

### What to Document

Public interfaces and their methods — document contract, parameters, return values, exceptions.

Non-trivial business rules — if a rule isn't obvious from the code, explain it.

External dependency quirks — rate limits, error behaviors, API constraints.

Configuration parameters — what each setting controls, acceptable values, format.

### What Not to Document

Properties and methods where the name says everything. Documentation that merely restates the name adds nothing.

### Solo Developer

Document: README (how to run), ADR (why decisions were made), XML docs for public library-layer methods, inline comments for non-trivial logic.

Skip: formal team process documents (CONTRIBUTING, Runbook) if you're the only developer.

### Team

Add: CONTRIBUTING (conventions and PR process), architecture overview (for onboarding), Runbook (how to handle typical incidents), and API documentation via Swagger/OpenAPI.

Principle: documentation should live next to the code to stay current. Swagger generates from code. XML docs are part of the code. ADR lives in the repository.

### What Goes Stale Quickly

Confluence/Notion pages updated manually, diagrams in separate files, detailed descriptions of how a method works. These always lag behind the code.

Write documentation that either auto-generates from code or ages very slowly (architectural decisions, concepts, principles).

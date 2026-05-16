# Практические паттерны / Practical Patterns

---

## RU

### Паттерны как инструменты, не цели

Паттерн — это имя для решения распространённой проблемы. Паттерны не применяются «чтобы было правильно». Они применяются, когда есть конкретная проблема, которую паттерн решает.

Для каждого паттерна ниже указано: какую проблему решает и когда применять.

---

### Repository

**Проблема:** бизнес-логика напрямую зависит от EF Core или SQL. Замена хранилища требует правки везде. Тесты требуют реальной БД.

**Решение:** интерфейс, который описывает операции с данными. Реализация — за ним.

```csharp
// Интерфейс в Domain или Application
public interface IAnnouncementRepository
{
    Task<Announcement?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<List<Announcement>> GetByChannelAsync(int channelId, CancellationToken ct = default);
    Task AddAsync(Announcement announcement, CancellationToken ct = default);
    Task DeleteAsync(int id, CancellationToken ct = default);
}

// Реализация в Infrastructure
public class AnnouncementRepository : IAnnouncementRepository
{
    private readonly AppDbContext _context;

    public AnnouncementRepository(AppDbContext context)
        => _context = context;

    public async Task<Announcement?> GetByIdAsync(int id, CancellationToken ct)
        => await _context.Announcements.FindAsync(new object[] { id }, ct);

    public async Task AddAsync(Announcement announcement, CancellationToken ct)
    {
        await _context.Announcements.AddAsync(announcement, ct);
        await _context.SaveChangesAsync(ct);
    }
}
```

**В тесте:**
```csharp
var repoMock = new Mock<IAnnouncementRepository>();
repoMock.Setup(r => r.GetByIdAsync(1, default))
        .ReturnsAsync(new Announcement { Id = 1, Title = "Test" });

var handler = new GetAnnouncementHandler(repoMock.Object);
```

**Когда применять:** всегда, когда Domain или Application слой работает с данными.

---

### Result / Either

**Проблема:** исключения используются для ожидаемых ошибок (не найдено, не валидно). Это усложняет поток управления и заставляет ловить исключения в контроллерах.

**Решение:** тип `Result<T>`, который явно несёт либо успех, либо ошибку.

```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public string? Error { get; }

    private Result(T value) { IsSuccess = true; Value = value; }
    private Result(string error) { IsSuccess = false; Error = error; }

    public static Result<T> Success(T value) => new(value);
    public static Result<T> Failure(string error) => new(error);
}

// Использование в Handler
public async Task<Result<int>> Handle(CreateAnnouncementCommand cmd, CancellationToken ct)
{
    var channel = await _channelRepo.GetByIdAsync(cmd.ChannelId, ct);
    if (channel is null)
        return Result<int>.Failure($"Channel {cmd.ChannelId} not found");

    var announcement = new Announcement(cmd.Title, cmd.Body, cmd.ChannelId);
    await _announcementRepo.AddAsync(announcement, ct);

    return Result<int>.Success(announcement.Id);
}

// В контроллере
var result = await _mediator.Send(command, ct);
return result.IsSuccess
    ? CreatedAtAction(nameof(GetById), new { id = result.Value }, null)
    : NotFound(result.Error);
```

**Когда применять:** для ожидаемых ошибок бизнес-логики. Исключения оставить для непредвиденных ситуаций (потеря соединения с БД, NPE).

---

### Options Pattern

**Проблема:** конфигурация разбросана по коду, жёстко задана или берётся через `IConfiguration["Key"]` напрямую — не типобезопасно.

**Решение:** строго типизированный класс настроек, зарегистрированный через `IOptions<T>`.

```csharp
// Класс настроек
public class TelegramOptions
{
    public const string SectionName = "Telegram";

    /// <summary>Токен бота из @BotFather.</summary>
    public string BotToken { get; init; } = string.Empty;

    /// <summary>Таймаут polling-запроса в секундах.</summary>
    public int PollingTimeoutSeconds { get; init; } = 30;
}

// Регистрация
services.Configure<TelegramOptions>(
    configuration.GetSection(TelegramOptions.SectionName));

// Использование
public class TelegramSender
{
    private readonly TelegramOptions _options;

    public TelegramSender(IOptions<TelegramOptions> options)
        => _options = options.Value;
}
```

```json
// appsettings.json
{
  "Telegram": {
    "BotToken": "...",
    "PollingTimeoutSeconds": 30
  }
}
```

**Когда применять:** для любой внешней конфигурации (API ключи, таймауты, URL адреса).

---

### Decorator

**Проблема:** нужно добавить сквозную функциональность (логирование, кэширование, retry) к существующему сервису, не изменяя его код.

**Решение:** класс-обёртка, реализующий тот же интерфейс.

```csharp
// Оригинальный сервис
public class AnnouncementRepository : IAnnouncementRepository
{
    public async Task<Announcement?> GetByIdAsync(int id, CancellationToken ct)
    {
        // запрос в БД
    }
}

// Декоратор с кэшированием
public class CachedAnnouncementRepository : IAnnouncementRepository
{
    private readonly IAnnouncementRepository _inner;
    private readonly IMemoryCache _cache;

    public CachedAnnouncementRepository(
        IAnnouncementRepository inner,
        IMemoryCache cache)
    {
        _inner = inner;
        _cache = cache;
    }

    public async Task<Announcement?> GetByIdAsync(int id, CancellationToken ct)
    {
        var key = $"announcement:{id}";
        if (_cache.TryGetValue(key, out Announcement? cached))
            return cached;

        var result = await _inner.GetByIdAsync(id, ct);
        if (result is not null)
            _cache.Set(key, result, TimeSpan.FromMinutes(5));

        return result;
    }
}

// Регистрация
services.AddScoped<AnnouncementRepository>();
services.AddScoped<IAnnouncementRepository>(sp =>
    new CachedAnnouncementRepository(
        sp.GetRequiredService<AnnouncementRepository>(),
        sp.GetRequiredService<IMemoryCache>()));
```

**Когда применять:** кэширование, логирование, retry, метрики — всё, что добавляется поверх, не меняя суть.

---

### MediatR Pipeline Behavior

**Проблема:** нужна сквозная логика (валидация, логирование) для всех команд/запросов, но не хочется дублировать её в каждом Handler.

**Решение:** Pipeline Behavior в MediatR — middleware для команд/запросов.

```csharp
// Behavior для валидации
public class ValidationBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
        => _validators = validators;

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken ct)
    {
        if (!_validators.Any())
            return await next();

        var context = new ValidationContext<TRequest>(request);
        var failures = _validators
            .Select(v => v.Validate(context))
            .SelectMany(r => r.Errors)
            .Where(e => e is not null)
            .ToList();

        if (failures.Count > 0)
            throw new ValidationException(failures);

        return await next();
    }
}

// Регистрация
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
services.AddTransient(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
```

**Когда применять:** когда одна и та же логика нужна для многих команд/запросов.

---

### Чего избегать: антипаттерны

**Service Locator** — получение зависимостей через `IServiceProvider` внутри класса:
```csharp
// Плохо
public class AnnouncementService
{
    private readonly IServiceProvider _sp;

    public async Task DoSomething()
    {
        var repo = _sp.GetRequiredService<IAnnouncementRepository>();  // антипаттерн
    }
}
```
Используй инъекцию через конструктор.

**God Object** — один класс, который знает и умеет всё. Признак: `AnnouncementManager` с 20+ методами разной ответственности.

**Anemic Domain Model** — сущности без поведения, только с полями. Вся логика в сервисах. Признак: у `Announcement` только get/set, а метод `Publish()` живёт в `AnnouncementService`.

```csharp
// Лучше: поведение в сущности
public class Announcement
{
    public AnnouncementStatus Status { get; private set; }

    public void Publish()
    {
        if (Status != AnnouncementStatus.Draft)
            throw new DomainException("Only draft announcements can be published");

        Status = AnnouncementStatus.Published;
    }
}
```

---

## EN

### Patterns as Tools, Not Goals

A pattern is a name for a solution to a common problem. Apply patterns when you have the specific problem they solve, not to "do it right."

### Repository

**Problem:** business logic depends directly on EF Core. Replacing storage or writing tests requires a real database.
**Solution:** an interface describing data operations; implementation lives behind it.
**Apply when:** Domain or Application layer needs to work with data.

### Result / Either

**Problem:** exceptions are thrown for expected business errors (not found, validation failed), complicating control flow.
**Solution:** `Result<T>` type that explicitly carries either success+value or failure+error.
**Apply when:** for expected business errors. Keep exceptions for unexpected situations (DB connection loss, null references).

### Options Pattern

**Problem:** configuration scattered through code, hardcoded or accessed via `IConfiguration["Key"]` without type safety.
**Solution:** strongly-typed settings class registered via `IOptions<T>`.
**Apply when:** any external configuration — API keys, timeouts, URLs.

### Decorator

**Problem:** need to add cross-cutting behavior (caching, logging, retry) to a service without changing its code.
**Solution:** wrapper class implementing the same interface, delegating to the inner implementation.
**Apply when:** caching, logging, retry, metrics — anything layered on top without changing the core behavior.

### MediatR Pipeline Behavior

**Problem:** cross-cutting logic (validation, logging) needed for all commands/queries without duplicating in every Handler.
**Solution:** Pipeline Behavior — middleware for MediatR requests.
**Apply when:** the same logic is needed across many commands/queries.

### Antipatterns to Avoid

**Service Locator:** resolving dependencies via `IServiceProvider` inside a class. Use constructor injection instead.

**God Object:** one class with 20+ methods of different responsibilities.

**Anemic Domain Model:** entities with only get/set fields, all behavior in service classes. Move behavior to the entity where it naturally belongs.

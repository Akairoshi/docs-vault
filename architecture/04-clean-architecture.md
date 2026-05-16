# Clean Architecture

---

## RU

### Суть

Clean Architecture (Роберт Мартин, «дядя Боб») — это архитектурный подход, в котором **бизнес-логика не зависит ни от чего внешнего**: ни от фреймворка, ни от базы данных, ни от UI.

Главное правило — **Dependency Rule**: зависимости направлены только внутрь. Внутренние слои ничего не знают о внешних.

```
┌────────────────────────────────────────┐
│            Infrastructure              │
│   ┌────────────────────────────────┐   │
│   │          Application           │   │
│   │   ┌────────────────────────┐   │   │
│   │   │        Domain          │   │   │
│   │   │  (Entities, Rules)     │   │   │
│   │   └────────────────────────┘   │   │
│   └────────────────────────────────┘   │
└────────────────────────────────────────┘
         ↑ зависимости идут внутрь
```

---

### Слои

**Domain** — ядро системы. Сущности, доменные правила, доменные исключения, value objects. Не зависит ни от чего.

**Application** — use cases (сценарии использования). Интерфейсы репозиториев и сервисов. DTO и команды/запросы (если используется CQRS). Зависит только от Domain.

**Infrastructure** — реализации интерфейсов из Application: EF Core, внешние HTTP-клиенты, файловая система, очереди. Зависит от Application и Domain.

**Presentation (WebApi)** — контроллеры, middleware, конфигурация DI. Зависит от Application (вызывает use cases).

---

### Структура проекта

```
src/
├── Domain/
│   ├── Entities/
│   │   └── User.cs
│   ├── Exceptions/
│   │   └── UserNotFoundException.cs
│   └── ValueObjects/
│       └── Email.cs
│
├── Application/
│   ├── Interfaces/
│   │   └── IUserRepository.cs
│   ├── Users/
│   │   ├── GetUserByIdQuery.cs
│   │   ├── GetUserByIdHandler.cs
│   │   └── UserDto.cs
│   └── DependencyInjection.cs
│
├── Infrastructure/
│   ├── Persistence/
│   │   ├── AppDbContext.cs
│   │   └── UserRepository.cs
│   └── DependencyInjection.cs
│
└── WebApi/
    ├── Controllers/
    │   └── UsersController.cs
    ├── Program.cs
    └── appsettings.json
```

---

### Пример кода

**Domain — сущность и value object:**

```csharp
// Domain/ValueObjects/Email.cs
public record Email
{
    public string Value { get; }

    public Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value) || !value.Contains('@'))
            throw new ArgumentException("Invalid email format.", nameof(value));

        Value = value.ToLowerInvariant();
    }
}

// Domain/Entities/User.cs
public class User
{
    public int Id { get; private set; }
    public string Name { get; private set; }
    public Email Email { get; private set; }

    private User() { } // для EF Core

    public User(string name, Email email)
    {
        Name = name;
        Email = email;
    }

    public void ChangeName(string newName)
    {
        if (string.IsNullOrWhiteSpace(newName))
            throw new ArgumentException("Name cannot be empty.");
        Name = newName;
    }
}
```

**Application — интерфейс и use case:**

```csharp
// Application/Interfaces/IUserRepository.cs
public interface IUserRepository
{
    Task<User?> GetByIdAsync(int id, CancellationToken ct = default);
    Task AddAsync(User user, CancellationToken ct = default);
    Task SaveChangesAsync(CancellationToken ct = default);
}

// Application/Users/GetUserByIdQuery.cs
public record GetUserByIdQuery(int UserId);

public record UserDto(int Id, string Name, string Email);

// Application/Users/GetUserByIdHandler.cs
public class GetUserByIdHandler
{
    private readonly IUserRepository _repo;

    public GetUserByIdHandler(IUserRepository repo)
    {
        _repo = repo;
    }

    public async Task<UserDto?> HandleAsync(GetUserByIdQuery query, CancellationToken ct = default)
    {
        var user = await _repo.GetByIdAsync(query.UserId, ct);
        if (user is null) return null;

        return new UserDto(user.Id, user.Name, user.Email.Value);
    }
}
```

**Infrastructure — реализация репозитория:**

```csharp
// Infrastructure/Persistence/UserRepository.cs
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _db;

    public UserRepository(AppDbContext db)
    {
        _db = db;
    }

    public async Task<User?> GetByIdAsync(int id, CancellationToken ct = default)
        => await _db.Users.FindAsync(new object[] { id }, ct);

    public async Task AddAsync(User user, CancellationToken ct = default)
        => await _db.Users.AddAsync(user, ct);

    public Task SaveChangesAsync(CancellationToken ct = default)
        => _db.SaveChangesAsync(ct);
}
```

**WebApi — контроллер:**

```csharp
// WebApi/Controllers/UsersController.cs
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly GetUserByIdHandler _handler;

    public UsersController(GetUserByIdHandler handler)
    {
        _handler = handler;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id, CancellationToken ct)
    {
        var result = await _handler.HandleAsync(new GetUserByIdQuery(id), ct);
        return result is null ? NotFound() : Ok(result);
    }
}
```

**Регистрация зависимостей:**

```csharp
// Infrastructure/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services, IConfiguration config)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(config.GetConnectionString("Default")));

        services.AddScoped<IUserRepository, UserRepository>();

        return services;
    }
}

// Program.cs
builder.Services.AddInfrastructure(builder.Configuration);
builder.Services.AddScoped<GetUserByIdHandler>();
```

---

### Что даёт Clean Architecture

- **Тестируемость** — `GetUserByIdHandler` тестируется с mock-репозиторием без поднятия БД
- **Независимость от фреймворка** — Domain и Application не знают о ASP.NET
- **Замена инфраструктуры** — переход с EF Core на Dapper не затрагивает Application и Domain
- **Понятная структура** — новый разработчик сразу видит, где бизнес-логика

---

### Для одного / для команды

**Один разработчик:**
Поначалу кажется избыточным. Но уже на среднем проекте окупается — тесты пишутся быстро, рефакторинг безопасен.

**Несколько разработчиков:**
Слои становятся естественными зонами ответственности. Один человек может работать над Infrastructure, не зная деталей Domain.

**Команда:**
Clean Architecture делает код-ревью предметным: «этот код не должен находиться в Domain, потому что он зависит от IEmailSender» — это конкретная проверяемая претензия. Можно автоматизировать проверку через [NetArchTest](https://github.com/BenMorris/NetArchTest):

```csharp
// ArchitectureTests.cs
[Test]
public void Domain_Should_Not_DependOn_Application()
{
    var result = Types.InAssembly(DomainAssembly)
        .Should().NotHaveDependencyOn("MyApp.Application")
        .GetResult();

    result.IsSuccessful.Should().BeTrue();
}
```

---

## EN

### Concept

Clean Architecture (Robert C. Martin, "Uncle Bob") is an architectural approach where **business logic depends on nothing external** — not the framework, not the database, not the UI.

The main rule is the **Dependency Rule**: dependencies point only inward. Inner layers know nothing about outer layers.

### Layers

**Domain** — the system's core. Entities, domain rules, domain exceptions, value objects. Depends on nothing.

**Application** — use cases. Repository and service interfaces. DTOs and commands/queries (if using CQRS). Depends only on Domain.

**Infrastructure** — implementations of Application interfaces: EF Core, external HTTP clients, file system, queues. Depends on Application and Domain.

**Presentation (WebApi)** — controllers, middleware, DI configuration. Depends on Application (calls use cases).

### What Clean Architecture Gives You

- **Testability** — handlers are tested with mock repositories, no database required
- **Framework independence** — Domain and Application know nothing about ASP.NET
- **Infrastructure swap** — moving from EF Core to Dapper doesn't touch Application or Domain
- **Clear structure** — new developers immediately see where business logic lives

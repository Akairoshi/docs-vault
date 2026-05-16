# Слоёная архитектура / Layered (N-Tier) Architecture

---

## RU

### Суть

Система делится на горизонтальные слои. Каждый слой выполняет одну роль и взаимодействует только со слоем непосредственно под ним.

Классическая трёхслойная схема:

```
┌─────────────────────────────┐
│     Presentation Layer      │  ← Controllers, API endpoints, Views
├─────────────────────────────┤
│      Business Layer         │  ← Services, Use Cases, бизнес-правила
├─────────────────────────────┤
│       Data Layer            │  ← Repositories, DbContext, ORM
└─────────────────────────────┘
```

Запрос идёт сверху вниз. Данные возвращаются снизу вверх. Слои не «прыгают» через уровни.

---

### Роли слоёв

**Presentation Layer** — принимает входящий запрос, валидирует его формат, вызывает бизнес-логику, формирует ответ. Не содержит логику предметной области.

**Business Layer** — вся логика приложения. Правила, расчёты, оркестрация. Не знает, откуда пришёл запрос (HTTP, CLI, очередь) и где хранятся данные.

**Data Layer** — общается с базой данных, внешними API, файловой системой. Возвращает данные в виде объектов домена или DTO.

---

### Пример структуры проекта на .NET

```
MyApp/
├── MyApp.Web/               ← Presentation: Controllers, Program.cs
├── MyApp.Application/       ← Business: Services, Interfaces
├── MyApp.Infrastructure/    ← Data: Repositories, EF DbContext
└── MyApp.Domain/            ← Общие модели, сущности (опционально)
```

Пример кода:

```csharp
// Presentation Layer — Controller
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;

    public UsersController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetById(int id)
    {
        var user = await _userService.GetByIdAsync(id);
        return user is null ? NotFound() : Ok(user);
    }
}

// Business Layer — Service
public class UserService : IUserService
{
    private readonly IUserRepository _repo;

    public UserService(IUserRepository repo)
    {
        _repo = repo;
    }

    public async Task<UserDto?> GetByIdAsync(int id)
    {
        var user = await _repo.GetByIdAsync(id);
        if (user is null) return null;

        return new UserDto(user.Id, user.Name, user.Email);
    }
}

// Data Layer — Repository
public class UserRepository : IUserRepository
{
    private readonly AppDbContext _db;

    public UserRepository(AppDbContext db)
    {
        _db = db;
    }

    public async Task<User?> GetByIdAsync(int id)
        => await _db.Users.FindAsync(id);
}
```

---

### Направление зависимостей

Слои зависят **только вниз**. Никогда — вверх.

```
Web → Application → Infrastructure
```

Это значит: `UserService` знает об `IUserRepository`, но не знает о `UserRepository` (конкретной реализации). Конкретная реализация регистрируется через DI в точке входа:

```csharp
// Program.cs
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IUserService, UserService>();
```

---

### Для одного / для команды

**Один разработчик:**
Слоёная архитектура — хорошая отправная точка. Она проста и хорошо известна. Применяй её даже в небольших проектах, чтобы с самого начала не смешивать HTTP-логику с запросами к БД.

**Несколько разработчиков:**
Слои естественно распределяются между людьми: один работает над контроллерами, другой — над сервисами, третий — над репозиториями. Интерфейсы позволяют работать параллельно.

**Команда:**
Договоритесь о правилах раз и навсегда: что можно делать в каждом слое, что нельзя. Зафиксируйте в ADR. Настройте линтер или архитектурные тесты (например, NetArchTest) для проверки соблюдения правил.

---

### Ограничения слоёной архитектуры

Классическая слоёная архитектура имеет один известный недостаток: **бизнес-логика зависит от инфраструктуры**. `Application` знает об `Infrastructure` напрямую.

Это приемлемо для небольших проектов, но при росте создаёт проблемы с тестированием и сменой технологий. Для решения этой проблемы существует Clean Architecture (см. `04-clean-architecture.md`).

---

## EN

### Concept

The system is divided into horizontal layers. Each layer has one role and interacts only with the layer directly below it.

Classic three-tier scheme:

```
┌─────────────────────────────┐
│     Presentation Layer      │  ← Controllers, API endpoints, Views
├─────────────────────────────┤
│      Business Layer         │  ← Services, Use Cases, business rules
├─────────────────────────────┤
│       Data Layer            │  ← Repositories, DbContext, ORM
└─────────────────────────────┘
```

Requests flow top-down. Data returns bottom-up. Layers don't skip levels.

---

### Layer Roles

**Presentation Layer** — receives incoming requests, validates their format, calls business logic, formats the response. Contains no domain logic.

**Business Layer** — all application logic. Rules, calculations, orchestration. Doesn't know where the request came from (HTTP, CLI, queue) or where data is stored.

**Data Layer** — communicates with the database, external APIs, file system. Returns data as domain objects or DTOs.

---

### Direction of Dependencies

Layers depend **only downward**. Never upward.

```
Web → Application → Infrastructure
```

This means: `UserService` knows about `IUserRepository`, but not about `UserRepository` (the concrete implementation). The concrete implementation is registered via DI at the entry point.

---

### Limitations of Layered Architecture

Classic layered architecture has one well-known drawback: **business logic depends on infrastructure**. `Application` knows about `Infrastructure` directly.

This is acceptable for small projects, but as the system grows it creates testing and technology-swap problems. Clean Architecture solves this (see `04-clean-architecture.md`).

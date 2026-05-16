# CQRS и Event Sourcing / CQRS and Event Sourcing

---

## RU

### CQRS — Command Query Responsibility Segregation

**Суть:** разделить операции чтения (Query) и записи (Command) на разные модели и потоки выполнения.

Без CQRS один сервис обслуживает и чтение, и запись:

```csharp
public interface IUserService
{
    Task<UserDto> GetByIdAsync(int id);       // Query
    Task CreateUserAsync(CreateUserDto dto);   // Command
    Task UpdateEmailAsync(int id, string email); // Command
}
```

С CQRS они разделены:

```
Command → CommandHandler → изменяет состояние (Write Model)
Query  → QueryHandler  → читает данные    (Read Model)
```

---

### Зачем разделять

**Чтение и запись — разные задачи:**

- Запись требует валидации, бизнес-правил, транзакций
- Чтение требует скорости, проекций, часто денормализованных данных

При едином сервисе модель компромиссная: ни для чтения, ни для записи не оптимальна.

При CQRS каждая сторона оптимизируется независимо.

---

### Пример реализации (без MediatR)

```csharp
// Command
public record CreateUserCommand(string Name, string Email);

public record CreateUserResult(int UserId);

// Command Handler
public class CreateUserHandler
{
    private readonly IUserRepository _repo;

    public CreateUserHandler(IUserRepository repo)
    {
        _repo = repo;
    }

    public async Task<CreateUserResult> HandleAsync(
        CreateUserCommand command, CancellationToken ct = default)
    {
        var email = new Email(command.Email); // доменная валидация
        var user = new User(command.Name, email);

        await _repo.AddAsync(user, ct);
        await _repo.SaveChangesAsync(ct);

        return new CreateUserResult(user.Id);
    }
}

// Query
public record GetUserByIdQuery(int UserId);

public record UserDto(int Id, string Name, string Email);

// Query Handler
public class GetUserByIdHandler
{
    private readonly IUserRepository _repo;

    public GetUserByIdHandler(IUserRepository repo)
    {
        _repo = repo;
    }

    public async Task<UserDto?> HandleAsync(
        GetUserByIdQuery query, CancellationToken ct = default)
    {
        var user = await _repo.GetByIdAsync(query.UserId, ct);
        return user is null ? null : new UserDto(user.Id, user.Name, user.Email.Value);
    }
}
```

---

### CQRS с MediatR

MediatR — популярная библиотека, которая реализует паттерн Mediator и удобно используется с CQRS:

```bash
dotnet add package MediatR
```

```csharp
// Query
public record GetUserByIdQuery(int UserId) : IRequest<UserDto?>;

// Handler
public class GetUserByIdHandler : IRequestHandler<GetUserByIdQuery, UserDto?>
{
    private readonly IUserRepository _repo;

    public GetUserByIdHandler(IUserRepository repo)
    {
        _repo = repo;
    }

    public async Task<UserDto?> Handle(
        GetUserByIdQuery request, CancellationToken cancellationToken)
    {
        var user = await _repo.GetByIdAsync(request.UserId, cancellationToken);
        return user is null ? null : new UserDto(user.Id, user.Name, user.Email.Value);
    }
}

// Controller
[HttpGet("{id}")]
public async Task<IActionResult> GetById(int id, CancellationToken ct)
{
    var result = await _mediator.Send(new GetUserByIdQuery(id), ct);
    return result is null ? NotFound() : Ok(result);
}

// Program.cs
builder.Services.AddMediatR(cfg =>
    cfg.RegisterServicesFromAssembly(typeof(GetUserByIdHandler).Assembly));
```

---

### Event Sourcing

**Суть:** вместо хранения текущего состояния объекта хранится **последовательность событий**, которые к нему привели.

Обычное хранение:

```
users таблица:
| id | name  | email            | balance |
|----|-------|------------------|---------|
| 1  | Alice | alice@example.com| 500     |
```

Event Sourcing:

```
events таблица:
| id | aggregate_id | type                | data                        |
|----|-------------|---------------------|-----------------------------|
| 1  | user-1      | UserRegistered      | {name: Alice, email: ...}   |
| 2  | user-1      | BalanceDeposited    | {amount: 1000}              |
| 3  | user-1      | BalanceWithdrawn    | {amount: 500}               |
```

Текущее состояние получается воспроизведением (replay) всех событий.

---

### Когда Event Sourcing уместен

**Уместен:**
- Финансовые системы (нужна полная история операций)
- Системы аудита
- Сложный доменный объект с богатой историей

**Не уместен:**
- CRUD-приложения
- Простые сервисы без требований к истории
- Команда не знакома с подходом

Event Sourcing — сложный инструмент. Не применяй его по умолчанию.

---

### Связь CQRS и Event Sourcing

CQRS и Event Sourcing часто используются вместе, но это **независимые паттерны**.

- CQRS можно применять без Event Sourcing
- Event Sourcing хорошо сочетается с CQRS, потому что при replay событий удобно строить отдельные read-модели

---

### Для одного / для команды

**Один разработчик:**
CQRS без MediatR — разумная структура даже для одного человека. Каждый handler — отдельный файл, легко находить, легко тестировать.

**Несколько разработчиков:**
Handlers параллельно разрабатываются разными людьми без конфликтов — каждый handler изолирован.

**Команда:**
Соглашение по именованию команд и запросов — обязательно. Пример:
- Команды: `CreateUserCommand`, `UpdateEmailCommand`, `DeleteOrderCommand`
- Запросы: `GetUserByIdQuery`, `ListActiveUsersQuery`
- Handlers: `CreateUserHandler`, `GetUserByIdHandler`

---

## EN

### CQRS — Command Query Responsibility Segregation

**Concept:** separate read operations (Query) and write operations (Command) into distinct models and execution paths.

```
Command → CommandHandler → changes state   (Write Model)
Query  → QueryHandler  → reads data       (Read Model)
```

**Why separate them:**

- Writes require validation, business rules, transactions
- Reads require speed, projections, often denormalized data

With a unified service, the model is a compromise: not optimal for either reads or writes. With CQRS, each side is optimized independently.

### Event Sourcing

**Concept:** instead of storing the current state of an object, store the **sequence of events** that led to it.

Current state is obtained by replaying all events.

**When Event Sourcing is appropriate:**
- Financial systems (complete history of operations required)
- Audit systems
- Complex domain objects with rich history

**When it's not appropriate:**
- CRUD applications
- Simple services without history requirements
- Team unfamiliar with the approach

Event Sourcing is a complex tool. Don't apply it by default.

### Relationship Between CQRS and Event Sourcing

CQRS and Event Sourcing are often used together, but they are **independent patterns**.

- CQRS can be used without Event Sourcing
- Event Sourcing pairs well with CQRS because replaying events makes it convenient to build separate read models

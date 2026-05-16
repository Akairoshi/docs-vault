# Структура папок и файлов / Folder Structure

---

## RU

### Принципы организации

Структура папок — это навигационная система. Хорошая структура отвечает на вопрос «где мне искать X?» за секунды. Плохая структура — это когда ответ «поищи везде».

Два подхода к организации:

**По слоям (layer-first):**
```
src/
├── Controllers/
├── Services/
├── Repositories/
└── Models/
```

**По функциям/доменам (feature-first):**
```
src/
├── Announcements/
│   ├── AnnouncementsController.cs
│   ├── AnnouncementService.cs
│   └── AnnouncementRepository.cs
└── Users/
    ├── UsersController.cs
    └── UserService.cs
```

Layer-first прост для старта, но плохо масштабируется. Feature-first требует дисциплины, но код, связанный по смыслу, лежит рядом.

В Clean Architecture используется гибрид: проекты разделены по слоям, внутри каждого слоя — по доменам.

---

### Эталонная структура: Clean Architecture в .NET

```
MyProject/
│
├── src/
│   │
│   ├── MyProject.Domain/
│   │   ├── Entities/
│   │   │   ├── Announcement.cs
│   │   │   ├── Channel.cs
│   │   │   └── User.cs
│   │   ├── Enums/
│   │   │   └── AnnouncementStatus.cs
│   │   ├── Exceptions/
│   │   │   ├── DomainException.cs
│   │   │   └── ChannelNotFoundException.cs
│   │   ├── Interfaces/
│   │   │   ├── IAnnouncementRepository.cs
│   │   │   └── IChannelRepository.cs
│   │   └── ValueObjects/
│   │       └── TelegramChannelId.cs
│   │
│   ├── MyProject.Application/
│   │   ├── Features/
│   │   │   ├── Announcements/
│   │   │   │   ├── Commands/
│   │   │   │   │   ├── CreateAnnouncement/
│   │   │   │   │   │   ├── CreateAnnouncementCommand.cs
│   │   │   │   │   │   ├── CreateAnnouncementHandler.cs
│   │   │   │   │   │   └── CreateAnnouncementValidator.cs
│   │   │   │   │   └── DeleteAnnouncement/
│   │   │   │   │       ├── DeleteAnnouncementCommand.cs
│   │   │   │   │       └── DeleteAnnouncementHandler.cs
│   │   │   │   └── Queries/
│   │   │   │       └── GetAnnouncement/
│   │   │   │           ├── GetAnnouncementQuery.cs
│   │   │   │           ├── GetAnnouncementHandler.cs
│   │   │   │           └── AnnouncementDto.cs
│   │   │   └── Users/
│   │   │       └── Commands/
│   │   │           └── RegisterUser/
│   │   │               ├── RegisterUserCommand.cs
│   │   │               └── RegisterUserHandler.cs
│   │   ├── Common/
│   │   │   ├── Interfaces/
│   │   │   │   ├── ICurrentUserContext.cs
│   │   │   │   └── INotificationSender.cs
│   │   │   ├── Models/
│   │   │   │   ├── Result.cs
│   │   │   │   └── PagedResult.cs
│   │   │   └── Behaviors/
│   │   │       ├── ValidationBehavior.cs
│   │   │       └── LoggingBehavior.cs
│   │   └── DependencyInjection.cs       ← регистрация сервисов Application слоя
│   │
│   ├── MyProject.Infrastructure/
│   │   ├── Persistence/
│   │   │   ├── AppDbContext.cs
│   │   │   ├── Migrations/
│   │   │   ├── Configurations/          ← EF Core entity configurations
│   │   │   │   └── AnnouncementConfiguration.cs
│   │   │   └── Repositories/
│   │   │       ├── AnnouncementRepository.cs
│   │   │       └── ChannelRepository.cs
│   │   ├── Telegram/
│   │   │   ├── TelegramSender.cs
│   │   │   └── TelegramOptions.cs
│   │   ├── Identity/                    ← если есть аутентификация
│   │   │   └── JwtTokenGenerator.cs
│   │   └── DependencyInjection.cs       ← регистрация сервисов Infrastructure слоя
│   │
│   └── MyProject.WebApi/
│       ├── Controllers/
│       │   ├── AnnouncementsController.cs
│       │   └── UsersController.cs
│       ├── Middleware/
│       │   ├── ExceptionHandlingMiddleware.cs
│       │   └── RequestLoggingMiddleware.cs
│       ├── Filters/                     ← если нужны action filters
│       ├── Models/                      ← Request/Response модели (не DTO из Application!)
│       │   └── Announcements/
│       │       └── CreateAnnouncementRequest.cs
│       ├── Program.cs
│       └── appsettings.json
│
├── tests/
│   ├── MyProject.Domain.Tests/
│   │   └── Entities/
│   │       └── AnnouncementTests.cs
│   ├── MyProject.Application.Tests/
│   │   └── Features/
│   │       └── Announcements/
│   │           └── CreateAnnouncementHandlerTests.cs
│   └── MyProject.Integration.Tests/
│       └── Controllers/
│           └── AnnouncementsControllerTests.cs
│
├── docs/                                ← или docs-vault/
├── .editorconfig
├── .gitignore
├── Directory.Build.props
├── README.md
├── CONTRIBUTING.md
└── MyProject.sln
```

---

### Что куда класть: правила

**Domain — только чистая бизнес-модель:**
- Сущности (Entities) — объекты с идентичностью (User, Announcement)
- Value Objects — объекты без идентичности (Email, Money, TelegramChannelId)
- Enums — перечисления бизнес-состояний
- Исключения — доменные ошибки
- Интерфейсы репозиториев — `IAnnouncementRepository` (но не реализация!)

Чего НЕТ в Domain: EF Core, HTTP, Telegram, JSON, DI.

---

**Application — координация, use cases:**
- Commands и Queries (CQRS через MediatR)
- Handlers — логика выполнения команды/запроса
- Validators — правила валидации (FluentValidation)
- DTO — объекты передачи данных (что возвращает API)
- Interfaces сервисов, которых нет в Domain (ICurrentUserContext, IEmailSender)
- Pipeline Behaviors (логирование, валидация как pipeline)

Чего НЕТ в Application: EF Core, HTTP, конкретные реализации сервисов.

---

**Infrastructure — всё внешнее:**
- Реализации репозиториев (работа с EF Core)
- AppDbContext + конфигурации сущностей
- Миграции
- Клиенты внешних API (Telegram, почта, SMS)
- Реализации инфраструктурных интерфейсов (IJwtGenerator и т.п.)

---

**WebApi — точка входа:**
- Controllers — только маршрутизация и преобразование запроса/ответа
- Middleware — обработка исключений, логирование запросов, аутентификация
- Program.cs — конфигурация и запуск
- Request/Response модели — что принимает и возвращает HTTP (отдельно от DTO!)

Контроллер не должен содержать бизнес-логику. Его задача:
1. Принять HTTP-запрос
2. Создать команду/запрос
3. Отправить через MediatR
4. Вернуть HTTP-ответ

```csharp
// Хорошо — контроллер как тонкий адаптер
[HttpPost]
public async Task<IActionResult> Create(
    [FromBody] CreateAnnouncementRequest request,
    CancellationToken ct)
{
    var command = new CreateAnnouncementCommand(request.Title, request.Body, request.ChannelId);
    var result = await _mediator.Send(command, ct);

    return result.IsSuccess
        ? CreatedAtAction(nameof(GetById), new { id = result.Value }, null)
        : BadRequest(result.Error);
}
```

---

### Именование файлов: итоговая таблица

| Тип | Суффикс | Пример |
|---|---|---|
| Команда (CQRS) | `Command` | `CreateAnnouncementCommand.cs` |
| Запрос (CQRS) | `Query` | `GetAnnouncementQuery.cs` |
| Обработчик | `Handler` | `CreateAnnouncementHandler.cs` |
| Валидатор | `Validator` | `CreateAnnouncementValidator.cs` |
| DTO | `Dto` | `AnnouncementDto.cs` |
| HTTP Request модель | `Request` | `CreateAnnouncementRequest.cs` |
| HTTP Response модель | `Response` | `AnnouncementResponse.cs` |
| Репозиторий (интерфейс) | `IXxxRepository` | `IAnnouncementRepository.cs` |
| Репозиторий (реализация) | `Repository` | `AnnouncementRepository.cs` |
| Конфигурация EF Core | `Configuration` | `AnnouncementConfiguration.cs` |
| Middleware | `Middleware` | `ExceptionHandlingMiddleware.cs` |
| Background сервис | `Worker` / `Service` | `AnnouncementPublisherWorker.cs` |
| Options/Settings | `Options` | `TelegramOptions.cs` |
| Расширения DI | `DependencyInjection` | `DependencyInjection.cs` (в корне слоя) |

---

### DependencyInjection.cs — соглашение по регистрации

Каждый слой регистрирует свои сервисы в одном месте:

```csharp
// Infrastructure/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseNpgsql(configuration.GetConnectionString("Default")));

        services.AddScoped<IAnnouncementRepository, AnnouncementRepository>();
        services.AddScoped<IChannelRepository, ChannelRepository>();
        services.AddSingleton<ITelegramSender, TelegramSender>();

        return services;
    }
}
```

```csharp
// WebApi/Program.cs
builder.Services
    .AddApplication()       // из Application/DependencyInjection.cs
    .AddInfrastructure(builder.Configuration);  // из Infrastructure/DependencyInjection.cs
```

Это держит Program.cs чистым и позволяет каждому слою управлять своей регистрацией.

---

## EN

### Organization Principles

Folder structure is a navigation system. Good structure answers "where do I find X?" in seconds.

Two approaches: layer-first (all Controllers together, all Services together) and feature-first (all Announcement-related code together). Layer-first is simple to start but scales poorly. Feature-first requires discipline but keeps related code co-located.

Clean Architecture uses a hybrid: projects split by layer, within each project organized by domain/feature.

### What Goes Where

**Domain:** pure business model — Entities, Value Objects, Enums, domain Exceptions, repository Interfaces (not implementations). No EF Core, HTTP, external SDKs.

**Application:** use cases — Commands, Queries, Handlers, Validators, DTOs, infrastructure interface definitions (ICurrentUserContext, IEmailSender), pipeline Behaviors. No EF Core, no HTTP, no concrete implementations.

**Infrastructure:** everything external — repository implementations, AppDbContext, Migrations, EF entity Configurations, external API clients (Telegram, email), JWT generation.

**WebApi:** entry point — Controllers (routing and HTTP adaptation only), Middleware, Program.cs, HTTP Request/Response models (separate from Application DTOs).

### Controller Rule

Controllers contain no business logic. Their job: receive HTTP request → create command/query → send via MediatR → return HTTP response.

### File Naming Summary

Commands → `CreateAnnouncementCommand.cs`. Handlers → `CreateAnnouncementHandler.cs`. Validators → `CreateAnnouncementValidator.cs`. DTOs → `AnnouncementDto.cs`. HTTP models → `CreateAnnouncementRequest.cs` / `AnnouncementResponse.cs`. Repository interfaces → `IAnnouncementRepository.cs`. Repository implementations → `AnnouncementRepository.cs`. DI registration → `DependencyInjection.cs` at each layer root.

### DependencyInjection.cs Convention

Each layer registers its own services in a single `DependencyInjection.cs` file exposing an `AddXxx()` extension method on `IServiceCollection`. Program.cs chains these calls, staying clean.

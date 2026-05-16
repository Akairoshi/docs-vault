# SOLID и принципы хорошего кода / SOLID and Design Principles

---

## RU

### Что такое расширяемость и поддерживаемость

**Расширяемость** — способность добавить новую функциональность, не переписывая существующую.

**Поддерживаемость** — способность понять, изменить и исправить код спустя месяцы без боли.

Это не абстрактные понятия. Это конкретные вещи, которые либо есть, либо нет:
- Можно добавить новый тип уведомлений, не трогая существующие?
- Можно понять, что делает метод, прочитав только его?
- Можно изменить способ хранения данных, не переписывая бизнес-логику?

---

### SOLID

Пять принципов, которые делают код расширяемым. Это не правила, которые нужно соблюдать формально — это инструменты для решения конкретных проблем.

---

#### S — Single Responsibility Principle (SRP)

**Класс должен иметь одну причину для изменения.**

Если класс меняется, когда изменяется бизнес-логика И когда изменяется способ хранения данных — у него две ответственности.

```csharp
// Плохо: AnnouncementService делает всё
public class AnnouncementService
{
    public async Task CreateAsync(string title, string body)
    {
        // Валидация
        if (string.IsNullOrWhiteSpace(title))
            throw new ArgumentException("Title required");

        // Сохранение в БД
        var announcement = new Announcement { Title = title, Body = body };
        _context.Announcements.Add(announcement);
        await _context.SaveChangesAsync();

        // Отправка в Telegram
        await _botClient.SendTextMessageAsync(chatId, $"{title}\n{body}");

        // Отправка email
        await _emailClient.SendAsync("admin@site.com", title, body);
    }
}

// Хорошо: каждый класс — одна ответственность
public class CreateAnnouncementHandler          // координация
public class AnnouncementRepository             // сохранение
public class TelegramNotificationSender         // Telegram
public class EmailNotificationSender            // Email
public class CreateAnnouncementValidator        // валидация
```

Практический признак нарушения SRP: класс нужно изменить по нескольким не связанным причинам.

---

#### O — Open/Closed Principle (OCP)

**Класс открыт для расширения, закрыт для изменения.**

Добавление новой функциональности не должно требовать изменения существующего кода.

```csharp
// Плохо: каждый новый тип уведомления — правка метода
public class NotificationService
{
    public async Task SendAsync(Announcement announcement, string type)
    {
        if (type == "telegram")
        {
            // отправить в Telegram
        }
        else if (type == "email")
        {
            // отправить email
        }
        else if (type == "sms") // добавили SMS — правим существующий класс
        {
            // отправить SMS
        }
    }
}

// Хорошо: новый тип — новый класс, старые не трогаем
public interface INotificationSender
{
    Task SendAsync(Announcement announcement, CancellationToken ct);
}

public class TelegramNotificationSender : INotificationSender { ... }
public class EmailNotificationSender : INotificationSender { ... }
public class SmsNotificationSender : INotificationSender { ... }  // новый — добавили, старые не тронули

public class NotificationService
{
    private readonly IEnumerable<INotificationSender> _senders;

    public async Task SendAllAsync(Announcement announcement, CancellationToken ct)
    {
        foreach (var sender in _senders)
            await sender.SendAsync(announcement, ct);
    }
}
```

---

#### L — Liskov Substitution Principle (LSP)

**Подтип должен быть полностью заменяем базовым типом.**

Если метод принимает `INotificationSender`, любая реализация должна работать корректно. Если реализация бросает `NotImplementedException` или меняет контракт — это нарушение LSP.

```csharp
// Плохо: реализация нарушает контракт интерфейса
public class NoOpNotificationSender : INotificationSender
{
    public Task SendAsync(Announcement announcement, CancellationToken ct)
    {
        throw new NotImplementedException();  // нарушение LSP
    }
}

// Хорошо: no-op реализация, которая честно ничего не делает
public class NoOpNotificationSender : INotificationSender
{
    public Task SendAsync(Announcement announcement, CancellationToken ct)
        => Task.CompletedTask;
}
```

---

#### I — Interface Segregation Principle (ISP)

**Клиент не должен зависеть от методов, которые он не использует.**

Жирные интерфейсы с десятками методов заставляют реализации писать много пустых заглушек.

```csharp
// Плохо: один большой интерфейс
public interface IUserService
{
    Task<User> GetByIdAsync(int id);
    Task CreateAsync(CreateUserDto dto);
    Task DeleteAsync(int id);
    Task SendPasswordResetEmailAsync(string email);
    Task<List<Role>> GetRolesAsync(int userId);
    Task AssignRoleAsync(int userId, int roleId);
}

// Хорошо: разделить по зонам ответственности
public interface IUserReader
{
    Task<User?> GetByIdAsync(int id);
}

public interface IUserWriter
{
    Task CreateAsync(CreateUserDto dto);
    Task DeleteAsync(int id);
}

public interface IUserRoleManager
{
    Task<List<Role>> GetRolesAsync(int userId);
    Task AssignRoleAsync(int userId, int roleId);
}
```

Каждый потребитель зависит только от того, что ему нужно.

---

#### D — Dependency Inversion Principle (DIP)

**Модули верхнего уровня не зависят от модулей нижнего уровня. Оба зависят от абстракций.**

```csharp
// Плохо: высокоуровневый код зависит от конкретной реализации
public class CreateAnnouncementHandler
{
    private readonly SqlAnnouncementRepository _repository;  // конкретный класс

    public CreateAnnouncementHandler()
    {
        _repository = new SqlAnnouncementRepository();  // жёсткая зависимость
    }
}

// Хорошо: зависимость от интерфейса, реализация снаружи
public class CreateAnnouncementHandler
{
    private readonly IAnnouncementRepository _repository;  // интерфейс

    public CreateAnnouncementHandler(IAnnouncementRepository repository)
    {
        _repository = repository;  // инъекция через конструктор
    }
}
```

DIP — основа для тестируемости. Если зависишь от интерфейса — в тесте можно подставить мок.

---

### DRY — Don't Repeat Yourself

Каждый кусок знания должен иметь одно авторитетное место в системе.

```csharp
// Плохо: валидация Title продублирована в нескольких местах
public class CreateAnnouncementHandler
{
    public async Task Handle(CreateAnnouncementCommand cmd)
    {
        if (string.IsNullOrWhiteSpace(cmd.Title) || cmd.Title.Length > 255)
            throw new ValidationException("Invalid title");
        // ...
    }
}

public class UpdateAnnouncementHandler
{
    public async Task Handle(UpdateAnnouncementCommand cmd)
    {
        if (string.IsNullOrWhiteSpace(cmd.Title) || cmd.Title.Length > 255)  // дубль
            throw new ValidationException("Invalid title");
        // ...
    }
}

// Хорошо: одно место для правил валидации
public class AnnouncementTitleValidator
{
    public bool IsValid(string title)
        => !string.IsNullOrWhiteSpace(title) && title.Length <= 255;
}
```

Но DRY — не повод создавать абстракции там, где два похожих куска кода просто совпадают случайно. Если они могут разойтись в разные стороны — дублирование лучше неверной абстракции.

---

### YAGNI — You Aren't Gonna Need It

Не строй то, что может когда-нибудь понадобиться.

```csharp
// Плохо: "на будущее" добавляем поддержку нескольких баз данных
public interface IAnnouncementStore
{
    Task SaveAsync(Announcement a);
}
public class SqlAnnouncementStore : IAnnouncementStore { ... }
public class MongoAnnouncementStore : IAnnouncementStore { ... }   // никто не просил
public class RedisAnnouncementStore : IAnnouncementStore { ... }  // никто не просил
```

Добавляй абстракцию, когда появляется вторая реализация — не заранее.

---

### Практические признаки хорошего кода

- Метод читается без комментариев
- Можно понять, что делает класс, не читая все его методы
- Добавление новой функциональности не требует правки 10 файлов
- Тест можно написать без поднятия всего приложения
- Имена переменных и методов не требуют объяснений

---

## EN

### What Extensibility and Maintainability Mean

**Extensibility:** can you add new functionality without rewriting existing code?

**Maintainability:** can you understand, change, and fix code months later without pain?

These are concrete observable properties, not abstract ideals.

### SOLID

**S — Single Responsibility:** a class has one reason to change. Practical test: if a class changes for two unrelated reasons, split it.

**O — Open/Closed:** open for extension, closed for modification. Add new behavior via new classes, not by editing existing ones. Interfaces with multiple implementations are the primary tool.

**L — Liskov Substitution:** any implementation must be fully substitutable for its interface. Implementations that throw `NotImplementedException` or change the contract violate LSP.

**I — Interface Segregation:** don't force clients to depend on methods they don't use. Split fat interfaces into focused ones.

**D — Dependency Inversion:** high-level modules depend on abstractions, not concrete implementations. Inject dependencies through constructors. This is the foundation of testability.

### DRY — Don't Repeat Yourself

Each piece of knowledge has one authoritative location. But don't create abstractions for coincidentally similar code that might diverge — duplication is better than a wrong abstraction.

### YAGNI — You Aren't Gonna Need It

Don't build what might someday be needed. Add abstractions when the second implementation appears, not in advance.

### Signs of Good Code

Method is readable without comments. Class purpose is clear without reading all its methods. Adding a feature doesn't require editing 10 files. Tests can be written without starting the full application. Names don't require explanation.

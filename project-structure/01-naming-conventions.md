# Соглашения об именовании / Naming Conventions

---

## RU

### Зачем соглашения

Соглашения об именовании — это не вкусовщина. Это способ передать информацию о том, что такое этот элемент кода, без чтения его реализации. Читатель смотрит на `IUserRepository` и понимает: это интерфейс репозитория. Смотрит на `userRepository` — это экземпляр. Смотрит на `MAX_RETRIES` — константа.

Нарушение соглашений создаёт когнитивную нагрузку: мозг тратит ресурсы на распознавание, а не на понимание логики.

---

### C# — стандартные соглашения Microsoft

#### PascalCase

Применяется к:
- Классам, структурам, записям (record)
- Интерфейсам (+ префикс `I`)
- Методам
- Свойствам
- Событиям
- Публичным полям
- Enum и их значениям
- Пространствам имён

```csharp
public class AnnouncementService { }
public interface IAnnouncementRepository { }
public record CreateAnnouncementCommand(string Title, string Body);
public enum AnnouncementStatus { Draft, Published, Archived }

public class UserService
{
    public event EventHandler UserCreated;

    public async Task<User> GetByIdAsync(int id) { }
    public string DisplayName { get; set; }
}
```

#### camelCase

Применяется к:
- Локальным переменным
- Параметрам методов
- Приватным полям (с префиксом `_`)

```csharp
public class AnnouncementService
{
    private readonly IAnnouncementRepository _repository;  // поле: _ + camelCase
    private readonly ILogger<AnnouncementService> _logger;

    public AnnouncementService(IAnnouncementRepository repository)
    {
        _repository = repository;  // параметр: camelCase
    }

    public async Task ProcessAsync(int announcementId)
    {
        var announcement = await _repository.GetByIdAsync(announcementId);  // локальная: camelCase
        var isPublished = announcement?.Status == AnnouncementStatus.Published;
    }
}
```

#### Константы

В C# константы пишутся PascalCase (не SCREAMING_SNAKE_CASE как в Java):

```csharp
// Правильно
private const int MaxRetryCount = 3;
public const string DefaultCulture = "en-US";

// Неправильно (стиль Java/C)
private const int MAX_RETRY_COUNT = 3;
```

---

### Правила именования по типам

#### Классы

- Существительное или существительное + прилагательное
- Отражает ответственность, не реализацию

```csharp
// Хорошо
public class AnnouncementService { }
public class TelegramMessageSender { }
public class UserRepository { }

// Плохо — слишком общо
public class Manager { }
public class Helper { }
public class Utils { }
public class Data { }
```

#### Интерфейсы

Префикс `I` + существительное (способность или роль):

```csharp
public interface IAnnouncementRepository { }  // роль
public interface INotificationSender { }      // способность
public interface ICurrentUserContext { }      // роль

// Плохо
public interface AnnouncementRepository { }   // нет I — как отличить от класса?
public interface IDoStuff { }                 // непонятно
```

#### Методы

Глагол или глагол + существительное. Асинхронные методы — суффикс `Async`:

```csharp
// Синхронные
public User GetById(int id) { }
public bool Validate(string input) { }
public void Send(Message message) { }
public List<User> FindByRole(Role role) { }

// Асинхронные — обязательно Async
public async Task<User> GetByIdAsync(int id) { }
public async Task SendAsync(Message message) { }
public async Task<bool> ExistsAsync(int id) { }

// Плохо
public async Task<User> GetById(int id) { }   // async без Async в названии
public async Task DoTheThing() { }            // непонятно что делает
```

#### Свойства

Существительное или прилагательное + существительное. Булевы свойства — `Is`, `Has`, `Can`, `Should`:

```csharp
public string Title { get; set; }
public DateTime CreatedAt { get; init; }
public int RetryCount { get; private set; }

public bool IsPublished { get; set; }
public bool HasAttachments { get; }
public bool CanEdit { get; }
```

#### Переменные и параметры

Конкретные, описывающие содержимое, не тип:

```csharp
// Хорошо
var announcement = await _repo.GetByIdAsync(id);
var activeUsers = users.Where(u => u.IsActive).ToList();
int retryCount = 0;

// Плохо
var a = await _repo.GetByIdAsync(id);   // однобуквенные (кроме i, j в циклах)
var obj = GetSomething();               // 'obj' ничего не говорит
var list = GetUsers();                  // тип вместо содержимого
```

Исключение: `i`, `j`, `k` в коротких циклах — допустимо.

#### Обобщённые параметры (generics)

`T` если один, иначе описательный с префиксом `T`:

```csharp
public class Repository<T> { }
public class Result<TValue, TError> { }
public Task<TResult> ExecuteAsync<TRequest, TResult>(TRequest request) { }
```

---

### Именование файлов и папок

- Один публичный тип — один файл
- Имя файла = имя типа: `AnnouncementService.cs`, `IAnnouncementRepository.cs`
- Папки — PascalCase, отражают домен или слой

```
Features/
├── Announcements/
│   ├── CreateAnnouncementCommand.cs
│   ├── CreateAnnouncementHandler.cs
│   ├── GetAnnouncementQuery.cs
│   └── AnnouncementValidator.cs
└── Users/
    ├── RegisterUserCommand.cs
    └── RegisterUserHandler.cs
```

---

### Именование в тестах

Конвенция: `МетодКоторыйТестируем_Условие_ОжидаемыйРезультат`

```csharp
public class CreateAnnouncementHandlerTests
{
    [Fact]
    public async Task Handle_ValidCommand_ReturnsSuccessWithId() { }

    [Fact]
    public async Task Handle_EmptyTitle_ReturnsValidationError() { }

    [Fact]
    public async Task Handle_NonExistentChannel_ReturnsChannelNotFoundError() { }
}
```

---

### Язык идентификаторов

**Рекомендация:** английский для всего кода — классов, методов, переменных, комментариев.

Причины:
- Код читают инструменты, которые не понимают кириллицу
- GitHub, Stack Overflow, ChatGPT — весь контекст на английском
- Новый разработчик из другой страны сможет разобраться

Если команда полностью русскоязычная и проект никогда не выйдет за её пределы — русский в комментариях допустим. Но в идентификаторах — только английский.

---

## EN

### Why Conventions Matter

Naming conventions communicate what a code element is without reading its implementation. A reader sees `IUserRepository` and knows: this is a repository interface. Sees `userRepository` — this is an instance. Violating conventions creates cognitive load — the brain spends resources on recognition instead of understanding logic.

### C# Microsoft Conventions

**PascalCase:** classes, structs, records, interfaces (+ `I` prefix), methods, properties, events, public fields, enums and their values, namespaces.

**camelCase:** local variables, method parameters, private fields (with `_` prefix).

**Constants:** PascalCase in C# (not SCREAMING_SNAKE_CASE).

### By Type

**Classes:** noun or adjective+noun reflecting responsibility, not implementation. Avoid: Manager, Helper, Utils, Data.

**Interfaces:** `I` prefix + noun (role or capability).

**Methods:** verb or verb+noun. Async methods: `Async` suffix (required).

**Properties:** noun or adjective+noun. Booleans: `Is`, `Has`, `Can`, `Should` prefix.

**Variables:** describe content, not type. Avoid single letters (except `i`, `j`, `k` in short loops).

**Generics:** `T` for single type parameter, descriptive `T`-prefixed names for multiple.

### Files and Folders

One public type per file. File name = type name. Folders use PascalCase and reflect domain or layer.

### Test Naming

Convention: `MethodUnderTest_Condition_ExpectedResult`.

### Language

English for all identifiers. Reasons: tooling compatibility, universal context (GitHub, StackOverflow, AI tools), accessibility to developers from any background.

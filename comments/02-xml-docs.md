# XML-документация в C# / XML Documentation

---

## RU

### Что такое XML-документация

В C# существует специальный формат комментариев с `///`, который:
- Отображается в IDE как подсказка при наведении (IntelliSense)
- Может быть экспортирован в HTML-документацию (DocFX, Sandcastle)
- Читается генераторами документации API (Swagger/OpenAPI, NSwag)

```csharp
/// <summary>
/// Создаёт новое объявление и сохраняет его в базе данных.
/// </summary>
/// <param name="command">Команда с данными объявления.</param>
/// <param name="cancellationToken">Токен отмены операции.</param>
/// <returns>
/// Результат операции. При успехе содержит Id созданного объявления.
/// </returns>
/// <exception cref="ChannelNotFoundException">
/// Выбрасывается если канал с указанным ChannelId не существует.
/// </exception>
public async Task<Result<int>> Handle(
    CreateAnnouncementCommand command,
    CancellationToken cancellationToken)
```

---

### Основные теги

**`<summary>`** — краткое описание. Отображается в IntelliSense. Обязателен.

```csharp
/// <summary>
/// Отправляет объявление в Telegram-канал.
/// </summary>
public async Task SendAsync(Announcement announcement) { ... }
```

**`<param>`** — описание параметра.

```csharp
/// <param name="channelId">Идентификатор Telegram-канала.</param>
/// <param name="message">Текст сообщения. Поддерживает HTML-разметку.</param>
```

**`<returns>`** — описание возвращаемого значения.

```csharp
/// <returns>
/// <c>true</c> если сообщение успешно отправлено; <c>false</c> при ошибке API.
/// </returns>
```

**`<exception>`** — какие исключения может бросать метод.

```csharp
/// <exception cref="ArgumentNullException">
/// Если <paramref name="message"/> равен null или пустой строке.
/// </exception>
/// <exception cref="TelegramApiException">
/// При ошибке взаимодействия с Telegram API.
/// </exception>
```

**`<remarks>`** — дополнительные пояснения, которые не вписываются в summary.

```csharp
/// <remarks>
/// Метод потокобезопасен. Использует ReaderWriterLockSlim для защиты кэша.
/// При первом вызове инициализирует внутренний кэш — это может занять до 500 мс.
/// </remarks>
```

**`<example>`** — пример использования.

```csharp
/// <example>
/// <code>
/// var sender = new TelegramSender(botToken);
/// await sender.SendAsync(channelId: -100123456789, message: "Hello!");
/// </code>
/// </example>
```

**`<inheritdoc>`** — наследовать документацию от базового класса или интерфейса.

```csharp
public interface IAnnouncementRepository
{
    /// <summary>
    /// Возвращает объявление по идентификатору.
    /// </summary>
    Task<Announcement?> GetByIdAsync(int id);
}

public class AnnouncementRepository : IAnnouncementRepository
{
    /// <inheritdoc/>
    public async Task<Announcement?> GetByIdAsync(int id) { ... }
}
```

**`<see>` и `<seealso>`** — ссылки на другие члены.

```csharp
/// <summary>
/// Создаёт объявление. Для удаления используйте <see cref="DeleteAnnouncementAsync"/>.
/// </summary>
```

**`<paramref>` и `<typeparamref>`** — ссылки на параметры внутри текста.

```csharp
/// <returns>
/// <c>null</c> если элемент с <paramref name="id"/> не найден.
/// </returns>
```

---

### Что документировать обязательно

Документируй всё, что является публичным API:

```
✓ public классы, интерфейсы, record'ы
✓ public методы (особенно в интерфейсах)
✓ public свойства с нетривиальной семантикой
✓ public константы и enum-значения
✓ Исключения, которые метод может бросать
```

Не обязательно документировать:
```
✗ private методы (только если сложные — тогда обычный // комментарий)
✗ тривиальные свойства типа Id, Name
✗ override методов с /// <inheritdoc/>
```

---

### Полный пример: публичный сервис

```csharp
/// <summary>
/// Сервис для управления объявлениями в Telegram-каналах.
/// </summary>
/// <remarks>
/// Все операции записываются в лог аудита.
/// Сервис требует наличия зарегистрированного бота в системе.
/// </remarks>
public class AnnouncementService : IAnnouncementService
{
    /// <summary>
    /// Создаёт новое объявление и публикует его в указанном канале.
    /// </summary>
    /// <param name="title">
    /// Заголовок объявления. Не может быть пустым. Максимум 255 символов.
    /// </param>
    /// <param name="body">Тело объявления. Поддерживает HTML-разметку Telegram.</param>
    /// <param name="channelId">
    /// Идентификатор канала. Канал должен существовать в системе.
    /// </param>
    /// <param name="cancellationToken">Токен отмены.</param>
    /// <returns>
    /// Результат операции. При успехе содержит Id созданного объявления.
    /// </returns>
    /// <exception cref="ValidationException">
    /// Если <paramref name="title"/> пустой или превышает 255 символов.
    /// </exception>
    /// <exception cref="ChannelNotFoundException">
    /// Если канал с указанным <paramref name="channelId"/> не существует.
    /// </exception>
    public async Task<Result<int>> CreateAsync(
        string title,
        string body,
        int channelId,
        CancellationToken cancellationToken = default)
    {
        // реализация
    }

    /// <summary>
    /// Возвращает объявление по идентификатору.
    /// </summary>
    /// <param name="id">Идентификатор объявления.</param>
    /// <returns>
    /// Объявление, или <c>null</c> если не найдено.
    /// </returns>
    public async Task<Announcement?> GetByIdAsync(int id) { ... }
}
```

---

### Настройка генерации XML-файла

Чтобы IDE видела документацию при ссылке на сборку из другого проекта, включи генерацию XML-файла:

```xml
<!-- В .csproj -->
<PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <!-- Ошибка если публичный член без документации: -->
    <NoWarn>1591</NoWarn>  <!-- убери эту строку если хочешь получать предупреждения -->
</PropertyGroup>
```

Или включи предупреждения для всех недокументированных публичных членов:

```xml
<PropertyGroup>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <!-- CS1591: Missing XML comment for publicly visible type or member -->
</PropertyGroup>
```

---

### Swagger и XML-документация

Если проект — API, XML-документация подтягивается в Swagger автоматически:

```csharp
// Program.cs
builder.Services.AddSwaggerGen(options =>
{
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});
```

После этого `<summary>` и `<remarks>` из кода появятся в Swagger UI.

---

## EN

### What Is XML Documentation

C# supports `///` comments that display in IDE IntelliSense tooltips, can be exported to HTML documentation (DocFX), and are read by API documentation generators (Swagger/OpenAPI).

### Core Tags

`<summary>` — short description, shown in IntelliSense. Required for public members.

`<param name="...">` — parameter description.

`<returns>` — description of the return value.

`<exception cref="...">` — exceptions the method may throw.

`<remarks>` — additional notes that don't fit in summary (thread safety, performance notes, constraints).

`<example>` with `<code>` — usage example.

`<inheritdoc/>` — inherit documentation from base class or interface. Use on implementations to avoid duplication.

`<see cref="...">` — cross-references to other members.

`<paramref name="...">` — reference a parameter by name within text.

### What to Document

Always document: public classes, interfaces, records, methods (especially on interfaces), properties with non-trivial semantics, constants, enum values, and exceptions a method may throw.

Skip: private methods (use plain `//` if complex), trivial Id/Name properties, overrides with `<inheritdoc/>`.

### XML File Generation

Enable in `.csproj`: `<GenerateDocumentationFile>true</GenerateDocumentationFile>`. This lets other projects in the solution see your documentation in IntelliSense.

### Swagger Integration

Pass the generated XML file path to `IncludeXmlComments()` in `AddSwaggerGen()` to have `<summary>` and `<remarks>` appear in Swagger UI.

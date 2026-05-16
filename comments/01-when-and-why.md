# Когда и зачем писать комментарии / When and Why to Comment

---

## RU

### Главный принцип

Комментарий объясняет **ПОЧЕМУ**, а не **ЧТО**.

Что делает код — видно из самого кода (если он написан понятно). Почему он делает именно это — часто не видно нигде. Это и есть задача комментария.

---

### Когда комментарии НЕ нужны

**Плохой комментарий — это комментарий, который дублирует код:**

```csharp
// Плохо: комментарий повторяет код
// Увеличиваем счётчик на 1
counter++;

// Плохо: очевидно из имени метода
// Возвращаем пользователя по Id
public User GetUserById(int id) { ... }

// Плохо: вместо нормального имени переменной
// d — количество дней
int d = 30;
```

Вместо такого комментария — дай переменной нормальное имя:

```csharp
int sessionExpirationDays = 30;
```

---

### Когда комментарии НУЖНЫ

**1. Нетривиальная бизнес-логика**

```csharp
// Telegram API возвращает ошибку 429 без Retry-After заголовка
// для некоторых типов запросов. В этом случае используем
// фиксированную задержку 3 секунды, которая эмпирически
// достаточна для сброса rate limit.
if (response.StatusCode == HttpStatusCode.TooManyRequests
    && !response.Headers.Contains("Retry-After"))
{
    await Task.Delay(TimeSpan.FromSeconds(3));
}
```

**2. Намеренное нарушение очевидного подхода**

```csharp
// Намеренно не используем async здесь: этот метод вызывается
// из конструктора через Task.Run, и async/await в конструкторе
// приводит к дедлоку в конкретном контексте синхронизации.
// См. issue #234.
public void Initialize()
{
    Task.Run(() => LoadCacheAsync()).Wait();
}
```

**3. Магические числа с объяснением**

```csharp
// BCrypt рекомендует work factor 12 как баланс между
// безопасностью и производительностью (2^12 итераций).
// Значение ниже 10 считается небезопасным.
private const int BcryptWorkFactor = 12;
```

**4. TODO и FIXME — с контекстом**

```csharp
// TODO: заменить на кэширование через IMemoryCache после задачи #89
// Сейчас каждый запрос идёт в БД — приемлемо на текущем объёме
var roles = await _repository.GetUserRolesAsync(userId);

// FIXME: при count > 10_000 запрос занимает > 5 сек.
// Нужен индекс по полю CreatedAt или пагинация на уровне запроса.
var all = await _context.Announcements.ToListAsync();
```

**5. Контракты и ограничения, которые не видны из сигнатуры**

```csharp
// Метод потокобезопасен. Внутренний словарь защищён ReaderWriterLockSlim.
// Можно вызывать из нескольких потоков одновременно.
public string? GetCachedValue(string key) { ... }

// Не вызывать повторно для одного и того же userId в рамках одного запроса.
// Метод делает запрос к внешнему API с ограничением 1 req/user/sec.
public async Task<UserProfile> FetchProfileAsync(int userId) { ... }
```

---

### Комментарии, которые нужно удалить

**Закомментированный код:**

```csharp
// Плохо — это мусор. Если код не нужен — удали его.
// История есть в git.
// var result = OldMethod(x);
// if (result != null) return result;
var result = NewMethod(x);
```

**Устаревшие комментарии — хуже отсутствия:**

```csharp
// Возвращает список активных пользователей
// (УСТАРЕЛО: теперь возвращает всех, включая удалённых — кто-то обновил метод
// но не обновил комментарий)
public List<User> GetUsers() { ... }
```

Устаревший комментарий активно вводит в заблуждение. Лучше никакого, чем неправильный.

---

### Стиль комментариев

- Пиши полными предложениями
- Начинай с заглавной буквы
- Не заканчивай точкой в коротких фразах (но в длинных объяснениях — можно)
- Язык: договорись в команде. EN — универсально, RU — если команда только русскоязычная

```csharp
// Хорошо
// Retry с экспоненциальной задержкой при временных ошибках сети

// Плохо
// retry логика для сети
```

---

### Отношение к комментариям: правило «будущего разработчика»

Представь, что через год незнакомый человек будет читать этот код в 23:00, пытаясь понять, почему что-то сломалось. Что ему нужно знать, что не видно из кода?

Напиши это.

---

## EN

### Main Principle

Comments explain **WHY**, not **WHAT**.

What the code does is visible from the code itself (if written clearly). Why it does exactly that is often invisible. That's the comment's job.

### When NOT to Comment

Don't duplicate the code in words. Don't explain variable names — fix the names instead. A comment that restates what the code obviously does is noise.

### When to Comment

**Non-trivial business logic** — external API quirks, retry strategies, domain rules that aren't obvious.

**Intentional violations of the obvious approach** — if you're doing something that looks wrong but isn't, explain why, and reference the relevant issue.

**Magic numbers** — constants with non-obvious values need an explanation of their origin and acceptable range.

**TODO/FIXME** — always include context: what needs to be done and why it wasn't done now. Reference a ticket number.

**Contracts not visible from the signature** — thread safety guarantees, rate limit constraints, call frequency restrictions.

### Comments to Delete

**Commented-out code** — delete it, git has the history.

**Outdated comments** — worse than no comment. An incorrect comment actively misleads. If you update code, update the comment or delete it.

### The "Future Developer" Rule

Imagine an unfamiliar person reading this code at 11pm trying to understand why something is broken. What do they need to know that isn't visible from the code? Write that.

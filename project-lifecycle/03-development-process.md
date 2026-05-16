# Процесс разработки / Development Process

---

## RU

### От задачи до готового кода

Хаотичная разработка — это когда садишься и сразу начинаешь писать код. Она работает для задач в 10 строк. Для всего остального она порождает код, который сложно читать, тестировать и изменять.

Структурированный процесс применим и одному человеку, и команде. Разница только в инструментах коммуникации.

---

### Шаг 1 — Понять задачу до конца

Перед тем как открыть редактор, ответь:

- Что именно нужно сделать? Сформулируй одним предложением.
- Что является входными данными и что выходными?
- Какие граничные случаи (edge cases) существуют?
- Как понять, что задача выполнена? (критерий готовности)

**Один разработчик:** записывай ответы в задаче или в TODO-комментарии.

**Команда:** эти вопросы — часть Definition of Done. Задача без критерия готовности не берётся в работу.

---

### Шаг 2 — Продумать решение до кода

Не обязательно долго. Достаточно набросать:

```
Что нужно сделать:
- Добавить эндпоинт POST /announcements
- Принять DTO с полями Title, Body, ChannelId
- Провалидировать (Title не пустой, ChannelId существует)
- Сохранить в БД
- Вернуть 201 Created с Id

Затронутые файлы:
- AnnouncementsController.cs (новый endpoint)
- CreateAnnouncementCommand.cs (новая команда)
- CreateAnnouncementHandler.cs (новый handler)
- AnnouncementValidator.cs (новый или расширить)
```

Это занимает 5 минут. Зато во время написания кода ты не блуждаешь — ты реализуешь уже принятое решение.

---

### Шаг 3 — Написать тест первым (или сразу после)

Не обязательно строгий TDD, но тест должен быть написан **до или сразу после** логики, не «потом когда-нибудь».

Тест до кода (TDD):
```csharp
[Fact]
public async Task Handle_ValidCommand_CreatesAnnouncement()
{
    // Arrange
    var command = new CreateAnnouncementCommand("Test Title", "Body", channelId: 1);
    var handler = new CreateAnnouncementHandler(_repoMock.Object);

    // Act
    var result = await handler.Handle(command, CancellationToken.None);

    // Assert
    result.IsSuccess.Should().BeTrue();
    _repoMock.Verify(r => r.AddAsync(It.IsAny<Announcement>()), Times.Once);
}
```

Если тест написан до кода — он не пройдёт. Это нормально. Цель — написать минимум кода, чтобы тест прошёл.

---

### Шаг 4 — Реализовать

Пиши код итерационно:

1. Минимально работающая версия (happy path)
2. Обработка ошибок и граничных случаев
3. Рефакторинг (название переменных, дублирование, читаемость)

Не рефакторить и добавлять фичи одновременно. Это разные операции.

---

### Шаг 5 — Проверить самому перед code review

Перед тем как открыть Pull Request, пройди по чеклисту:

```
[ ] Код компилируется без предупреждений (dotnet build)
[ ] Все тесты проходят (dotnet test)
[ ] Форматирование в порядке (dotnet format --verify-no-changes)
[ ] Нет закомментированного кода или TODO без тикета
[ ] Нет console.log / Debug.WriteLine оставленных случайно
[ ] Нет жёстко прописанных строк (hardcoded strings) — пароли, URL, конфиги
[ ] Коммиты в порядке (осмысленные сообщения, нет "fix", "wip", "asd")
```

---

### Шаг 6 — Code Review (для команды)

**Автор PR:**
- Описывает, что сделано и почему (не пересказ diff, а контекст)
- Указывает, где хотел бы особого внимания
- Сам прочитал свой diff перед отправкой

**Ревьювер:**
- Проверяет логику, а не только стиль
- Задаёт вопросы вместо утверждений: «почему здесь List, а не IEnumerable?» вместо «сделай IEnumerable»
- Отмечает хорошее тоже — review не должен быть только критикой

**Правило:** ревьювер не блокирует PR из-за личных предпочтений стиля, если в команде есть соглашение и линтер.

---

### Шаг 7 — Merge и сопровождение

После merge:
- Удали ветку (feature-ветки не должны жить вечно)
- Убедись, что CI прошёл на `main`
- Обнови задачу в трекере

---

### Итоговый цикл

```
Задача → Понять → Спроектировать → Написать тест → Реализовать
      → Проверить → (Code Review) → Merge → Удалить ветку
```

---

## EN

### From Task to Finished Code

Chaotic development — opening your editor and immediately writing code — works for 10-line tasks. For everything else it produces code that is hard to read, test, and change.

### Step 1 — Fully Understand the Task

Before opening the editor: what exactly needs to be done (one sentence), what are the inputs and outputs, what edge cases exist, and what is the definition of done?

**Solo:** write answers in the task or as TODO comments.
**Team:** these questions are part of the Definition of Done. A task without a completion criterion doesn't get picked up.

### Step 2 — Think Before Coding

A 5-minute sketch of what files will be touched and what the flow looks like prevents wandering during implementation.

### Step 3 — Write the Test First (or Immediately After)

Not strict TDD required, but the test must be written before or immediately after the logic — not "sometime later."

### Step 4 — Implement Iteratively

1. Minimum working version (happy path)
2. Error handling and edge cases
3. Refactoring (names, duplication, readability)

Do not refactor and add features simultaneously. These are separate operations.

### Step 5 — Self-Review Before Pull Request

Build without warnings, all tests pass, formatting clean, no leftover debug output, no hardcoded secrets or URLs, commits are meaningful.

### Step 6 — Code Review

**Author:** describes what was done and why (context, not diff summary). Points out areas wanting attention. Read your own diff before submitting.

**Reviewer:** checks logic not just style. Asks questions instead of making demands. Notes good things too.

### Step 7 — Merge and Follow-up

Delete the branch. Confirm CI passes on main. Update the task in the tracker.

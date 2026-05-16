# Как выбрать архитектуру / How to Choose an Architecture

---

## RU

### Нет универсально правильного ответа

Выбор архитектуры — это набор компромиссов. Нет архитектуры, которая лучше всех во всех ситуациях. Есть архитектура, которая **лучше всего подходит для конкретного контекста**.

Контекст определяется:
- Размером команды
- Стадией проекта (MVP, зрелый продукт)
- Требованиями к масштабированию
- Сложностью домена

---

### Вопросы для выбора

Перед выбором архитектуры задай себе эти вопросы:

**1. Насколько сложен домен?**

Если большая часть логики — это CRUD (создать, прочитать, обновить, удалить) без сложных правил, Clean Architecture избыточна. Достаточно слоёной архитектуры или даже Minimal API с прямыми вызовами к БД.

Если домен богатый — сложные правила, жизненный цикл сущностей, много бизнес-правил — Clean Architecture или DDD оправданы.

**2. Как быстро нужно запуститься?**

MVP, прототип, хакатон → монолит, простая слоёная архитектура.
Долгосрочный продукт → вложи время в архитектуру сейчас.

**3. Сколько людей будет работать с кодом?**

1–2 человека → монолит, слоёная архитектура.
5+ человек → модульный монолит с явными границами.
Несколько независимых команд → рассмотри микросервисы (но только если есть зрелая инфраструктура).

**4. Есть ли требования к независимому масштабированию?**

Нет → монолит или модульный монолит.
Да → микросервисы (или просто горизонтальное масштабирование монолита — оно работает дольше, чем кажется).

**5. Насколько зрела инфраструктура?**

Микросервисы требуют: CI/CD для каждого сервиса, centralized logging (ELK, Grafana Loki), distributed tracing (Jaeger, OpenTelemetry), service discovery, API gateway. Если этого нет — микросервисы будут болью.

---

### Дерево решений

```
Стартуешь новый проект?
│
├── Да → Команда < 5 человек?
│         ├── Да → Монолит со слоёной архитектурой
│         └── Нет → Модульный монолит
│
└── Нет → Текущая архитектура вызывает проблемы?
          ├── Нет → Не меняй
          └── Да → В чём проблема?
                    ├── Сложно разобраться в коде → Разбей на модули
                    ├── Невозможно масштабировать → Выдели горячие части в сервисы
                    └── Тесты медленные/хрупкие → Внедри Clean Architecture
```

---

### Распространённые ошибки

**Ошибка 1: Начать с микросервисов**

Типичная мотивация: «мы хотим масштабироваться». Типичный результат: полгода на инфраструктуру вместо продукта, потом оказывается, что границы между сервисами выбраны неправильно и менять дорого.

Правило: **начни с монолита. Выдели сервисы когда почувствуешь боль**, а не заранее.

**Ошибка 2: Применять Clean Architecture везде**

Clean Architecture — это накладные расходы. Для небольшого API с 5 эндпоинтами это лишний слой абстракций без выгоды.

**Ошибка 3: Менять архитектуру без причины**

«Я прочитал про CQRS и хочу переписать» — плохая причина. Хорошая причина — конкретная проблема, которую решает эта архитектура.

**Ошибка 4: Смешивать архитектурные стили**

Если начали с Clean Architecture, применяйте её везде последовательно. Половина проекта на Clean Architecture, половина на классическом MVC — хуже, чем любой из них отдельно.

---

### Рекомендация по умолчанию

Для большинства новых проектов на .NET:

```
Монолит + Clean Architecture внутри + возможность разделения на модули позже
```

Это даёт:
- Простой деплой (монолит)
- Тестируемую бизнес-логику (Clean Architecture)
- Возможность расти (чёткие границы)

---

### Зафиксируй решение

Каждый выбор архитектуры — это ADR. Зафиксируй:
- Что выбрано
- Почему именно это
- Какие альтернативы рассматривались
- Какие последствия

Пример: `docs-vault/adr/ADR-0001-clean-architecture.md`

---

## EN

### There Is No Universally Right Answer

Architecture selection is a set of trade-offs. There is no architecture that is best in all situations. There is only the architecture that **fits a specific context best**.

Context is defined by:
- Team size
- Project stage (MVP, mature product)
- Scaling requirements
- Domain complexity

### Questions to Guide Your Choice

**1. How complex is the domain?**
If most logic is CRUD without complex rules, Clean Architecture is overkill. A layered architecture or even Minimal API with direct DB calls is enough.
If the domain is rich — complex rules, entity lifecycles, many business rules — Clean Architecture or DDD is justified.

**2. How fast do you need to ship?**
MVP, prototype, hackathon → monolith, simple layered architecture.
Long-term product → invest in architecture now.

**3. How many people will work with the code?**
1–2 people → monolith, layered architecture.
5+ people → modular monolith with explicit boundaries.
Multiple independent teams → consider microservices (only if infrastructure is mature).

**4. Are there requirements for independent scaling?**
No → monolith or modular monolith.
Yes → microservices (or just horizontal scaling of the monolith — it works longer than you think).

### Common Mistakes

**Mistake 1: Starting with microservices**
Typical motivation: "we want to scale." Typical result: six months on infrastructure instead of product, then discovering that service boundaries were wrong and are expensive to change.
Rule: **start with a monolith. Extract services when you feel pain**, not in advance.

**Mistake 2: Applying Clean Architecture everywhere**
Clean Architecture has overhead. For a small API with 5 endpoints, it's an extra abstraction layer with no benefit.

**Mistake 3: Changing architecture without a reason**
"I read about CQRS and want to rewrite" is a bad reason. A good reason is a specific problem that architecture solves.

### Default Recommendation

For most new .NET projects:

```
Monolith + Clean Architecture inside + ability to split into modules later
```

This gives you:
- Simple deployment (monolith)
- Testable business logic (Clean Architecture)
- Room to grow (clear boundaries)

---
aliases: ["Чистая архитектура", "Clean Architecture"]
tags: ["#архитектурный-паттерн", "#frontend", "#backend", "#ddd", "#separation-of-concerns"]
---

# Clean-архитектура

## Обзор

Clean Architecture (Чистая архитектура) - это подход к построению программных систем, предложенный Робертом Мартином (Uncle Bob). Она направлена на создание приложений, которые являются:
- Тестируемыми
- Независимыми от UI
- Независимыми от базы данных
- Независимыми от любой внешней агентности

## Основные принципы

### 1. Зависимости направлены внутрь
- Внешние слои зависят от внутренних, но не наоборот
- Внутренние слои не знают о внешних

### 2. Правило инверсии зависимостей
- Зависимости должны быть направлены к абстракциям, а не к конкретным реализациям
- Абстракции не должны зависеть от деталей

### 3. Слои архитектуры
- Каждый слой имеет четко определенную ответственность
- Слои отделены друг от друга границами

## Структура слоев

```
┌─────────────────────────────────────┐
│            Frameworks &             │
│         Drivers Layer               │
├─────────────────────────────────────┤
│            Interface                │
│         Adapters Layer              │
├─────────────────────────────────────┤
│            Use Cases                │
│           (Interactors)             │
├─────────────────────────────────────┤
│            Entities                 │
│          (Business Rules)           │
└─────────────────────────────────────┘
```

### Внешний слой (Frameworks & Drivers)
- Веб-фреймворки (Express, Django)
- Базы данных
- UI-фреймворки
- Внешние API

### Слой адаптеров (Interface Adapters)
- Преобразует данные между слоями
- Контроллеры, презентеры
- DTO (Data Transfer Objects)

### Слой вариантов использования (Use Cases)
- Бизнес-логика приложения
- Оркестрация операций
- Не зависит от UI или базы данных

### Внутренний слой (Entities)
- Бизнес-правила
- Доменные объекты
- Наиболее стабильная часть системы

## Пример реализации на JavaScript

```javascript
// Внутренний слой - Entities (Доменные объекты)
class User {
  constructor(id, name, email) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
  }

  validate() {
    if (!this.name || this.name.length < 3) {
      throw new Error('Имя должно содержать не менее 3 символов');
    }
    if (!this.email || !this.isValidEmail(this.email)) {
      throw new Error('Некорректный email');
    }
  }

  isValidEmail(email) {
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return re.test(email);
  }
}

// Слой вариантов использования (Use Cases)
class UserRegistrationUseCase {
  constructor(userRepository, emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  async execute(userData) {
    // Создаем пользователя
    const user = new User(
      userData.id || Date.now().toString(),
      userData.name,
      userData.email
    );

    // Валидируем
    user.validate();

    // Проверяем, существует ли уже пользователь
    const existingUser = await this.userRepository.findByEmail(user.email);
    if (existingUser) {
      throw new Error('Пользователь с таким email уже существует');
    }

    // Сохраняем пользователя
    const savedUser = await this.userRepository.save(user);

    // Отправляем приветственное письмо
    await this.emailService.sendWelcomeEmail(savedUser.email, savedUser.name);

    return savedUser;
  }
}

// Абстракции для репозитория и сервиса
class UserRepositoryInterface {
  async save(user) {
    throw new Error('Метод save должен быть реализован');
  }

  async findByEmail(email) {
    throw new Error('Метод findByEmail должен быть реализован');
  }
}

class EmailServiceInterface {
  async sendWelcomeEmail(to, name) {
    throw new Error('Метод sendWelcomeEmail должен быть реализован');
  }
}

// Слой адаптеров - реализации интерфейсов
class InMemoryUserRepository extends UserRepositoryInterface {
  constructor() {
    super();
    this.users = new Map();
  }

  async save(user) {
    this.users.set(user.id, user);
    return user;
  }

  async findByEmail(email) {
    for (const user of this.users.values()) {
      if (user.email === email) {
        return user;
      }
    }
    return null;
  }
}

class ConsoleEmailService extends EmailServiceInterface {
  async sendWelcomeEmail(to, name) {
    console.log(`Отправляем приветственное письмо на ${to} для ${name}`);
    return Promise.resolve();
  }
}

// Внешний слой - контроллер (например, для Express.js)
class UserController {
  constructor(userRegistrationUseCase) {
    this.userRegistrationUseCase = userRegistrationUseCase;
  }

  async registerUser(req, res) {
    try {
      const userData = req.body;
      const user = await this.userRegistrationUseCase.execute(userData);
      res.status(201).json({ user });
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

// Использование
const userRepository = new InMemoryUserRepository();
const emailService = new ConsoleEmailService();
const userRegistrationUseCase = new UserRegistrationUseCase(
  userRepository,
  emailService
);
const userController = new UserController(userRegistrationUseCase);

// Пример вызова (в реальном приложении это будет через Express.js роут)
// userController.registerUser({ body: { name: 'Иван', email: 'ivan@example.com' } }, { status: () => ({ json: console.log }) });
```

## Преимущества

- **Независимость от фреймворков** - приложение не зависит от внешних библиотек
- **Тестируемость** - бизнес-логика легко тестируется изолированно
- **Независимость от UI** - можно легко изменить интерфейс без изменения бизнес-логики
- **Независимость от базы данных** - можно легко сменить СУБД
- **Независимость от внешних агентов** - система не зависит от внешних API

## Недостатки

- **Сложность для простых приложений** - может быть избыточной для небольших проектов
- **Бойлерплейт** - требует создания множества промежуточных слоев и интерфейсов
- **Кривая обучения** - требует понимания принципов проектирования

## Когда использовать

- В проектах среднего и крупного размера
- Когда важна тестируемость и поддерживаемость
- В долгосрочных проектах с высоким уровнем изменений
- Когда необходимо обеспечить независимость от внешних факторов

## Связанные концепции

- [[Архитектурные-паттерны]] - общее понятие о паттернах архитектуры
- [[Domain-Driven-Design]] - подход к проектированию, часто используемый вместе с Clean Architecture
- [[Dependency-Inversion]] - один из принципов SOLID, лежащий в основе Clean Architecture
- [[Separation-of-Concerns]] - принцип разделения ответственностей
- [[MVC-паттерн]] - альтернативный подход к архитектуре
- [[Feature-Sliced-Design]] - современный подход к архитектуре frontend-приложений

## Заключение

Clean Architecture предоставляет мощный подход к построению масштабируемых и поддерживаемых приложений. Несмотря на дополнительную сложность, она окупается в долгосрочной перспективе за счет гибкости и тестируемости.
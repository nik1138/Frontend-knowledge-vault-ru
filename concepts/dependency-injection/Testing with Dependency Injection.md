---
tags: 
  - programming
  - testing
  - dependency-injection
  - unit-testing
  - mocks
---

# Тестирование с использованием внедрения зависимостей

## Введение

Одним из ключевых преимуществ внедрения зависимостей (DI) является упрощение тестирования. DI позволяет легко заменять реальные зависимости на мок-объекты, что делает модульное тестирование более эффективным и изолированным.

## Принципы тестирования с DI

### 1. Изоляция компонентов

С помощью DI мы можем тестировать компоненты изолированно, без необходимости запускать всю систему:

```javascript
// Тестируемый класс
class UserService {
  constructor(database, emailService) {
    this.database = database;
    this.emailService = emailService;
  }
  
  async registerUser(userData) {
    const user = await this.database.save(userData);
    await this.emailService.sendWelcomeEmail(user.email);
    return user;
  }
}

// Тест с мок-зависимостями
describe('UserService', () => {
  let userService;
  let mockDatabase;
  let mockEmailService;
  
  beforeEach(() => {
    mockDatabase = {
      save: jest.fn().mockResolvedValue({ id: '1', email: 'test@example.com' })
    };
    
    mockEmailService = {
      sendWelcomeEmail: jest.fn().mockResolvedValue(true)
    };
    
    userService = new UserService(mockDatabase, mockEmailService);
  });
  
  it('should register user and send welcome email', async () => {
    const userData = { name: 'John', email: 'john@example.com' };
    const result = await userService.registerUser(userData);
    
    expect(mockDatabase.save).toHaveBeenCalledWith(userData);
    expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith('test@example.com');
    expect(result.id).toBe('1');
  });
});
```

### 2. Легкость подстановки моков

DI делает подстановку мок-объектов тривиальной задачей:

```javascript
// Реализация для продакшена
class RealPaymentService {
  async processPayment(amount, cardDetails) {
    // Реальная обработка платежа
  }
}

// Мок для тестирования
class MockPaymentService {
  constructor() {
    this.processedPayments = [];
  }
  
  async processPayment(amount, cardDetails) {
    this.processedPayments.push({ amount, cardDetails, status: 'success' });
    return { success: true, transactionId: 'mock-123' };
  }
}

// В тестах легко подставляем мок
const paymentService = new MockPaymentService();
const orderService = new OrderService(paymentService);

// Проверяем поведение
expect(paymentService.processedPayments).toHaveLength(1);
```

## Типы тестовых двойников

### 1. Моки (Mocks)

Моки не только имитируют поведение, но и проверяют взаимодействия:

```javascript
class MockNotificationService {
  constructor() {
    this.sentNotifications = [];
    this.verifyCallCount = 0;
  }
  
  async sendNotification(message, recipient) {
    this.sentNotifications.push({ message, recipient, timestamp: Date.now() });
    this.verifyCallCount++;
    return { success: true };
  }
  
  verifyNotificationSent(expectedMessage, expectedRecipient) {
    const notification = this.sentNotifications.find(n => 
      n.message === expectedMessage && n.recipient === expectedRecipient
    );
    return !!notification;
  }
}

// Использование в тесте
test('should send notification when order is placed', async () => {
  const mockNotificationService = new MockNotificationService();
  const orderService = new OrderService(mockNotificationService);
  
  await orderService.placeOrder({ id: '1', customer: 'John' });
  
  expect(mockNotificationService.verifyCallCount).toBe(1);
  expect(mockNotificationService.verifyNotificationSent(
    'Your order has been placed', 
    'John'
  )).toBe(true);
});
```

### 2. Заглушки (Stubs)

Заглушки предоставляют предопределенные ответы:

```javascript
class ConfigStub {
  constructor(overrides = {}) {
    this.config = {
      apiEndpoint: 'https://api.example.com',
      timeout: 5000,
      ...overrides
    };
  }
  
  get(key) {
    return this.config[key];
  }
}

// Использование в тесте
test('should use correct API endpoint', () => {
  const configStub = new ConfigStub({ apiEndpoint: 'https://test-api.com' });
  const apiService = new ApiService(configStub);
  
  expect(apiService.getEndpoint()).toBe('https://test-api.com');
});
```

### 3. Фейки (Fakes)

Фейки имеют рабочую реализацию, но упрощенную:

```javascript
class InMemoryUserRepository {
  constructor() {
    this.users = new Map();
    this.nextId = 1;
  }
  
  async save(user) {
    const id = user.id || this.nextId++;
    const userWithId = { ...user, id: id.toString() };
    this.users.set(id.toString(), userWithId);
    return userWithId;
  }
  
  async findById(id) {
    return this.users.get(id) || null;
  }
  
  async findAll() {
    return Array.from(this.users.values());
  }
}

// Использование в интеграционных тестах
test('should create and retrieve user', async () => {
  const repository = new InMemoryUserRepository();
  const userService = new UserService(repository);
  
  const createdUser = await userService.createUser({ name: 'John' });
  const retrievedUser = await userService.getUser(createdUser.id);
  
  expect(retrievedUser.name).toBe('John');
});
```

## Интеграция с DI контейнерами

### Тестовый контейнер

Создание специального контейнера для тестирования:

```javascript
class TestDIContainer {
  constructor() {
    this.services = new Map();
  }
  
  register(name, instance) {
    this.services.set(name, instance);
  }
  
  resolve(name) {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Test service ${name} not found`);
    }
    return service;
  }
  
  // Метод для быстрой регистрации моков
  registerMock(name, mockImplementation) {
    this.register(name, mockImplementation);
  }
}

// Использование в тестах
describe('OrderService', () => {
  let container;
  let orderService;
  
  beforeEach(() => {
    container = new TestDIContainer();
    
    // Регистрируем моки
    container.registerMock('paymentService', {
      processPayment: jest.fn().mockResolvedValue({ success: true })
    });
    
    container.registerMock('inventoryService', {
      checkAvailability: jest.fn().mockResolvedValue(true),
      reserveItems: jest.fn().mockResolvedValue(true)
    });
    
    // Регистрируем тестируемый сервис с зависимостями
    orderService = new OrderService(
      container.resolve('paymentService'),
      container.resolve('inventoryService')
    );
  });
  
  test('should process order successfully', async () => {
    const order = { items: [{ id: '1', quantity: 2 }] };
    const result = await orderService.processOrder(order);
    
    expect(result.success).toBe(true);
    expect(container.resolve('paymentService').processPayment).toHaveBeenCalled();
  });
});
```

## Паттерны тестирования с DI

### 1. Arrange-Act-Assert с DI

```javascript
describe('ReportService', () => {
  // Arrange
  let reportService;
  let mockDatabase;
  let mockFormatter;
  
  beforeEach(() => {
    mockDatabase = createMockDatabase();
    mockFormatter = createMockFormatter();
    reportService = new ReportService(mockDatabase, mockFormatter);
  });
  
  test('should generate formatted report', async () => {
    // Arrange
    const testData = [{ id: 1, name: 'Item 1' }];
    mockDatabase.getData.mockResolvedValue(testData);
    mockFormatter.format.mockReturnValue('formatted data');
    
    // Act
    const result = await reportService.generateReport('sales');
    
    // Assert
    expect(mockDatabase.getData).toHaveBeenCalledWith('sales');
    expect(mockFormatter.format).toHaveBeenCalledWith(testData);
    expect(result).toBe('formatted data');
  });
});
```

### 2. Builder Pattern для тестовых данных

```javascript
class ServiceBuilder {
  constructor() {
    this.dependencies = new Map();
  }
  
  withDatabase(mockDb) {
    this.dependencies.set('database', mockDb);
    return this;
  }
  
  withLogger(mockLogger) {
    this.dependencies.set('logger', mockLogger);
    return this;
  }
  
  withConfig(config) {
    this.dependencies.set('config', config);
    return this;
  }
  
  build(serviceClass) {
    const db = this.dependencies.get('database') || this.createDefaultDb();
    const logger = this.dependencies.get('logger') || this.createDefaultLogger();
    const config = this.dependencies.get('config') || this.createDefaultConfig();
    
    return new serviceClass(db, logger, config);
  }
  
  createDefaultDb() {
    return { query: jest.fn(), save: jest.fn() };
  }
  
  createDefaultLogger() {
    return { log: jest.fn(), error: jest.fn() };
  }
  
  createDefaultConfig() {
    return { timeout: 5000 };
  }
}

// Использование
test('should handle user registration', async () => {
  const service = new ServiceBuilder()
    .withDatabase(createMockUserDatabase())
    .withLogger(createMockLogger())
    .build(UserService);
  
  // Тестирование
  const result = await service.registerUser({ name: 'John' });
  expect(result.success).toBe(true);
});
```

## Тестирование различных сценариев

### 1. Тестирование ошибок

```javascript
test('should handle database errors gracefully', async () => {
  const mockDatabase = {
    save: jest.fn().mockRejectedValue(new Error('Database connection failed'))
  };
  
  const userService = new UserService(mockDatabase, createMockEmailService());
  
  await expect(userService.registerUser({ name: 'John' }))
    .rejects.toThrow('Database connection failed');
  
  expect(mockDatabase.save).toHaveBeenCalledWith({ name: 'John' });
});
```

### 2. Тестирование взаимодействия

```javascript
test('should send email only when user is successfully saved', async () => {
  const mockDatabase = {
    save: jest.fn().mockResolvedValue({ id: '1', email: 'test@example.com' })
  };
  
  const mockEmailService = {
    sendWelcomeEmail: jest.fn().mockResolvedValue(true)
  };
  
  const userService = new UserService(mockDatabase, mockEmailService);
  
  await userService.registerUser({ name: 'John', email: 'test@example.com' });
  
  // Проверяем, что email отправлен только после успешного сохранения
  expect(mockDatabase.save).toHaveBeenCalled();
  expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith('test@example.com');
  expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledTimes(1);
});
```

## Инструменты для тестирования с DI

### Использование популярных библиотек

```javascript
// С использованием Jest
describe('UserService with Jest', () => {
  let userService;
  
  beforeEach(() => {
    // Автоматическое создание моков
    const mockDatabase = {
      save: jest.fn(),
      findById: jest.fn()
    };
    
    const mockEmailService = {
      sendWelcomeEmail: jest.fn()
    };
    
    userService = new UserService(mockDatabase, mockEmailService);
  });
  
  test('should call all dependencies correctly', async () => {
    const userData = { name: 'John', email: 'john@example.com' };
    userService.database.save.mockResolvedValue({ id: '1', ...userData });
    
    await userService.registerUser(userData);
    
    expect(userService.database.save).toHaveBeenCalledWith(userData);
    expect(userService.emailService.sendWelcomeEmail).toHaveBeenCalledWith('john@example.com');
  });
});

// С использованием Sinon.js
const sinon = require('sinon');

describe('PaymentService with Sinon', () => {
  let sandbox;
  let paymentService;
  
  beforeEach(() => {
    sandbox = sinon.createSandbox();
    
    const mockGateway = {
      charge: sandbox.stub().resolves({ success: true })
    };
    
    paymentService = new PaymentService(mockGateway);
  });
  
  afterEach(() => {
    sandbox.restore();
  });
  
  test('should verify payment gateway interaction', async () => {
    await paymentService.processPayment(100, 'card-123');
    
    sinon.assert.calledOnce(paymentService.gateway.charge);
    sinon.assert.calledWith(paymentService.gateway.charge, 100, 'card-123');
  });
});
```

## Лучшие практики тестирования с DI

### 1. Используйте явные моки

```javascript
// Хорошо: явные моки с понятным поведением
const createMockDatabase = () => ({
  save: jest.fn().mockResolvedValue({ id: '1', name: 'John' }),
  findById: jest.fn().mockResolvedValue({ id: '1', name: 'John' }),
  update: jest.fn().mockResolvedValue(true)
});

// Плохо: неявные моки с неопределенным поведением
const createMockDatabase = () => ({
  save: jest.fn(), // Что возвращает?
  findById: jest.fn() // Что возвращает?
});
```

### 2. Тестируйте взаимодействия, а не реализацию

```javascript
// Хорошо: тестирование взаимодействия
test('should save user and send notification', async () => {
  const mockDatabase = createMockDatabase();
  const mockNotifier = createMockNotifier();
  const service = new UserService(mockDatabase, mockNotifier);
  
  await service.createUser({ name: 'John' });
  
  expect(mockDatabase.save).toHaveBeenCalled();
  expect(mockNotifier.send).toHaveBeenCalledWith('User created: John');
});

// Плохо: тестирование внутренней реализации
test('should set user property correctly', () => {
  const service = new UserService(mockDatabase, mockNotifier);
  service.createUser({ name: 'John' });
  
  // Тестирование внутреннего состояния - плохая практика
  expect(service._currentUser.name).toBe('John');
});
```

### 3. Используйте Test-Specific Subclass когда необходимо

```javascript
class TestableUserService extends UserService {
  constructor(database, emailService) {
    super(database, emailService);
    this.processedUsers = [];
  }
  
  async registerUser(userData) {
    const user = await super.registerUser(userData);
    this.processedUsers.push(user);
    return user;
  }
}

// Использование в тесте
test('should track processed users', async () => {
  const service = new TestableUserService(mockDatabase, mockEmailService);
  
  await service.registerUser({ name: 'John' });
  await service.registerUser({ name: 'Jane' });
  
  expect(service.processedUsers).toHaveLength(2);
});
```

## Заключение

Тестирование с использованием внедрения зависимостей значительно упрощает создание надежных, изолированных тестов. DI позволяет легко подставлять моки, заглушки и фейки, что делает тестирование более управляемым и предсказуемым.

Ключевые преимущества:
- Упрощение изоляции тестируемых компонентов
- Легкость подстановки тестовых двойников
- Возможность тестирования различных сценариев
- Повышение надежности тестов

См. также:
- [[Внедрение-зависимостей]]
- [[DI Patterns and Implementation]]
- [[Testing Principles]]
- [[Unit Testing]]
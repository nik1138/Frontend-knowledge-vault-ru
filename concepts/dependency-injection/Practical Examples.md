---
tags: 
  - programming
  - dependency-injection
  - practical-examples
  - use-cases
  - architecture
---

# Практические примеры внедрения зависимостей

## Введение

В этой статье мы рассмотрим практические примеры использования внедрения зависимостей (DI) в реальных сценариях разработки. Мы изучим, как DI применяется в различных архитектурных подходах, в веб-приложениях, микросервисах и других контекстах.

## Пример 1: Веб-приложение с многоуровневой архитектурой

Рассмотрим типичное веб-приложение с тремя слоями: контроллеры, сервисы и репозитории.

```javascript
// Интерфейс репозитория
class IUserRepository {
  async findById(id) { throw new Error('Not implemented'); }
  async save(user) { throw new Error('Not implemented'); }
  async findAll() { throw new Error('Not implemented'); }
}

// Конкретная реализация репозитория
class DatabaseUserRepository extends IUserRepository {
  constructor(databaseConnection) {
    super();
    this.db = databaseConnection;
  }
  
  async findById(id) {
    const query = 'SELECT * FROM users WHERE id = ?';
    return await this.db.query(query, [id]);
  }
  
  async save(user) {
    if (user.id) {
      const query = 'UPDATE users SET name = ?, email = ? WHERE id = ?';
      return await this.db.query(query, [user.name, user.email, user.id]);
    } else {
      const query = 'INSERT INTO users (name, email) VALUES (?, ?)';
      return await this.db.query(query, [user.name, user.email]);
    }
  }
  
  async findAll() {
    const query = 'SELECT * FROM users';
    return await this.db.query(query);
  }
}

// Сервисный слой
class UserService {
  constructor(userRepository, logger, validator) {
    this.repository = userRepository;
    this.logger = logger;
    this.validator = validator;
  }
  
  async getUser(id) {
    this.logger.info(`Getting user with id: ${id}`);
    
    if (!this.validator.isValidId(id)) {
      throw new Error('Invalid user ID');
    }
    
    const user = await this.repository.findById(id);
    this.logger.info(`Retrieved user: ${user ? user.name : 'not found'}`);
    
    return user;
  }
  
  async createUser(userData) {
    this.logger.info('Creating new user');
    
    if (!this.validator.isValidUserData(userData)) {
      throw new Error('Invalid user data');
    }
    
    const user = await this.repository.save(userData);
    this.logger.info(`Created user with id: ${user.insertId}`);
    
    return user;
  }
}

// Контроллер
class UserController {
  constructor(userService) {
    this.userService = userService;
  }
  
  async handleGetUser(req, res) {
    try {
      const user = await this.userService.getUser(req.params.id);
      if (user) {
        res.json(user);
      } else {
        res.status(404).json({ error: 'User not found' });
      }
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
  
  async handleCreateUser(req, res) {
    try {
      const user = await this.userService.createUser(req.body);
      res.status(201).json(user);
    } catch (error) {
      res.status(400).json({ error: error.message });
    }
  }
}

// Контейнер зависимостей
class DIContainer {
  constructor() {
    this.services = new Map();
    this.singletons = new Map();
  }
  
  register(name, factory, options = {}) {
    this.services.set(name, {
      factory,
      singleton: options.singleton || false
    });
  }
  
  resolve(name) {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Service ${name} not registered`);
    }
    
    if (service.singleton && !this.singletons.has(name)) {
      this.singletons.set(name, service.factory(this));
    }
    
    return service.singleton ? this.singletons.get(name) : service.factory(this);
  }
}

// Регистрация всех зависимостей
const container = new DIContainer();

// Регистрация базовых сервисов как синглтоны
container.register('database', () => createDatabaseConnection(), { singleton: true });
container.register('logger', () => new Logger(), { singleton: true });
container.register('validator', () => new UserValidator());

// Регистрация репозитория
container.register('userRepository', (container) => 
  new DatabaseUserRepository(container.resolve('database')));

// Регистрация сервиса
container.register('userService', (container) => 
  new UserService(
    container.resolve('userRepository'),
    container.resolve('logger'),
    container.resolve('validator')
  ));

// Регистрация контроллера
container.register('userController', (container) => 
  new UserController(container.resolve('userService')));

// Использование
const userController = container.resolve('userController');
```

## Пример 2: Микросервис с внешними зависимостями

Рассмотрим микросервис, который взаимодействует с внешними API и базами данных.

```typescript
// Интерфейсы для внешних сервисов
interface IEmailService {
  sendEmail(to: string, subject: string, body: string): Promise<boolean>;
}

interface INotificationService {
  sendNotification(userId: string, message: string): Promise<void>;
}

interface IUserExternalService {
  getUserProfile(userId: string): Promise<any>;
}

// Реализации внешних сервисов
class EmailService implements IEmailService {
  constructor(private smtpConfig: any) {}
  
  async sendEmail(to: string, subject: string, body: string): Promise<boolean> {
    // Реализация отправки email через SMTP
    console.log(`Sending email to ${to}: ${subject}`);
    return true; // Симуляция успешной отправки
  }
}

class NotificationService implements INotificationService {
  constructor(private pushNotificationClient: any) {}
  
  async sendNotification(userId: string, message: string): Promise<void> {
    // Реализация отправки push-уведомления
    console.log(`Sending notification to user ${userId}: ${message}`);
  }
}

class UserExternalService implements IUserExternalService {
  constructor(private httpClient: any, private userServiceUrl: string) {}
  
  async getUserProfile(userId: string): Promise<any> {
    // Вызов внешнего сервиса пользователей
    return await this.httpClient.get(`${this.userServiceUrl}/users/${userId}`);
  }
}

// Основной сервис микросервиса
class OrderService {
  constructor(
    private emailService: IEmailService,
    private notificationService: INotificationService,
    private userExternalService: IUserExternalService,
    private orderRepository: any, // Репозиторий заказов
    private logger: any
  ) {}
  
  async processOrder(orderData: any): Promise<any> {
    try {
      this.logger.info(`Processing order: ${orderData.id}`);
      
      // Получаем профиль пользователя из внешнего сервиса
      const userProfile = await this.userExternalService.getUserProfile(orderData.userId);
      
      // Сохраняем заказ в базе данных
      const order = await this.orderRepository.save(orderData);
      
      // Отправляем уведомления
      await this.emailService.sendEmail(
        userProfile.email,
        'Order Confirmation',
        `Your order #${order.id} has been confirmed.`
      );
      
      await this.notificationService.sendNotification(
        orderData.userId,
        `Your order #${order.id} has been confirmed.`
      );
      
      this.logger.info(`Order ${order.id} processed successfully`);
      return order;
    } catch (error) {
      this.logger.error(`Error processing order: ${error.message}`);
      throw error;
    }
  }
}

// Фабрика для создания сервисов
class ServiceFactory {
  static createOrderService(container: DIContainer): OrderService {
    return new OrderService(
      container.resolve('emailService'),
      container.resolve('notificationService'),
      container.resolve('userExternalService'),
      container.resolve('orderRepository'),
      container.resolve('logger')
    );
  }
}
```

## Пример 3: Тестирование с DI

Показываем, как DI упрощает тестирование:

```javascript
// Тестируемый сервис
class PaymentService {
  constructor(paymentGateway, transactionLogger, fraudDetector) {
    this.paymentGateway = paymentGateway;
    this.transactionLogger = transactionLogger;
    this.fraudDetector = fraudDetector;
  }
  
  async processPayment(paymentData) {
    // Проверка на мошенничество
    const isFraud = await this.fraudDetector.check(paymentData);
    if (isFraud) {
      throw new Error('Fraudulent transaction detected');
    }
    
    // Обработка платежа
    const result = await this.paymentGateway.charge(paymentData.amount, paymentData.card);
    
    // Логирование транзакции
    await this.transactionLogger.log({
      paymentId: result.id,
      amount: paymentData.amount,
      status: result.success ? 'success' : 'failed'
    });
    
    return result;
  }
}

// Тесты с моками
describe('PaymentService', () => {
  let paymentService;
  let mockGateway;
  let mockLogger;
  let mockFraudDetector;
  
  beforeEach(() => {
    mockGateway = {
      charge: jest.fn()
    };
    
    mockLogger = {
      log: jest.fn()
    };
    
    mockFraudDetector = {
      check: jest.fn()
    };
    
    paymentService = new PaymentService(mockGateway, mockLogger, mockFraudDetector);
  });
  
  test('should process valid payment successfully', async () => {
    // Настройка моков
    mockFraudDetector.check.mockResolvedValue(false);
    mockGateway.charge.mockResolvedValue({ id: 'txn-123', success: true });
    
    const paymentData = { amount: 100, card: '4111111111111111' };
    const result = await paymentService.processPayment(paymentData);
    
    expect(mockFraudDetector.check).toHaveBeenCalledWith(paymentData);
    expect(mockGateway.charge).toHaveBeenCalledWith(100, '4111111111111111');
    expect(mockLogger.log).toHaveBeenCalledWith({
      paymentId: 'txn-123',
      amount: 100,
      status: 'success'
    });
    expect(result.success).toBe(true);
  });
  
  test('should reject fraudulent payment', async () => {
    mockFraudDetector.check.mockResolvedValue(true);
    
    const paymentData = { amount: 100, card: '4111111111111111' };
    
    await expect(paymentService.processPayment(paymentData))
      .rejects.toThrow('Fraudulent transaction detected');
      
    expect(mockGateway.charge).not.toHaveBeenCalled();
    expect(mockLogger.log).not.toHaveBeenCalled();
  });
});
```

## Пример 4: Интеграция с фреймворками

### Express.js с DI

```javascript
const express = require('express');

// Middleware для внедрения контейнера DI
function diMiddleware(container) {
  return (req, res, next) => {
    req.container = container;
    next();
  };
}

// Фабрика контроллеров
function createUserController(container) {
  const userService = container.resolve('userService');
  
  return {
    async getUser(req, res) {
      try {
        const user = await userService.getUser(req.params.id);
        if (user) {
          res.json(user);
        } else {
          res.status(404).json({ error: 'User not found' });
        }
      } catch (error) {
        res.status(500).json({ error: error.message });
      }
    },
    
    async createUser(req, res) {
      try {
        const user = await userService.createUser(req.body);
        res.status(201).json(user);
      } catch (error) {
        res.status(400).json({ error: error.message });
      }
    }
  };
}

// Настройка приложения
const app = express();
app.use(express.json());

// Создание и настройка контейнера
const container = setupDIContainer(); // функция настройки контейнера

// Использование middleware
app.use(diMiddleware(container));

// Регистрация маршрутов
const userController = createUserController(container);
app.get('/users/:id', userController.getUser.bind(userController));
app.post('/users', userController.createUser.bind(userController));

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Использование с Clean Architecture

```javascript
// Внешний слой (фреймворк/драйверы)
class UserController {
  constructor(userInteractor) {
    this.interactor = userInteractor;
  }
  
  async handleGetUser(request, response) {
    const { userId } = request.params;
    
    try {
      const output = await this.interactor.getUser({ userId });
      response.json(output.user);
    } catch (error) {
      response.status(500).json({ error: error.message });
    }
  }
}

// Средний слой (юзкейсы/интеракторы)
class GetUserInteractor {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }
  
  async getUser(request) {
    const user = await this.userRepository.findById(request.userId);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    return {
      user: {
        id: user.id,
        name: user.name,
        email: user.email
      }
    };
  }
}

// Внутренний слой (сущности/бизнес-логика)
class User {
  constructor(id, name, email) {
    this.id = id;
    this.name = name;
    this.email = email;
  }
  
  updateEmail(newEmail) {
    if (!this.isValidEmail(newEmail)) {
      throw new Error('Invalid email');
    }
    this.email = newEmail;
  }
  
  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

// Регистрация в контейнере для Clean Architecture
function setupCleanArchitectureDI() {
  const container = new DIContainer();
  
  // Репозитории (внешние границы)
  container.register('userRepository', (container) => 
    new DatabaseUserRepository(container.resolve('database')));
  
  // Интеракторы (бизнес-логика)
  container.register('getUserInteractor', (container) => 
    new GetUserInteractor(container.resolve('userRepository')));
  
  // Контроллеры (входные границы)
  container.register('userController', (container) => 
    new UserController(container.resolve('getUserInteractor')));
  
  return container;
}
```

## Пример 5: Управление жизненным циклом зависимостей

```javascript
class AdvancedDIContainer {
  constructor() {
    this.registrations = new Map();
    this.singletonInstances = new Map();
    this.scopedInstances = new Map();
    this.currentScope = null;
  }
  
  register(name, factory, lifecycle = 'transient') {
    this.registrations.set(name, { factory, lifecycle });
  }
  
  resolve(name) {
    const registration = this.registrations.get(name);
    if (!registration) {
      throw new Error(`Registration for ${name} not found`);
    }
    
    switch (registration.lifecycle) {
      case 'singleton':
        if (!this.singletonInstances.has(name)) {
          this.singletonInstances.set(name, registration.factory(this));
        }
        return this.singletonInstances.get(name);
        
      case 'scoped':
        if (!this.currentScope) {
          throw new Error('No active scope for scoped dependency');
        }
        if (!this.currentScope.has(name)) {
          this.currentScope.set(name, registration.factory(this));
        }
        return this.currentScope.get(name);
        
      case 'transient':
      default:
        return registration.factory(this);
    }
  }
  
  createScope() {
    const oldScope = this.currentScope;
    this.currentScope = new Map();
    
    return {
      resolve: (name) => this.resolve(name),
      dispose: () => {
        // Очистка ресурсов в области видимости
        if (this.currentScope) {
          for (const [name, instance] of this.currentScope) {
            if (instance && typeof instance.dispose === 'function') {
              instance.dispose();
            }
          }
        }
        this.currentScope = oldScope;
      }
    };
  }
}

// Пример использования с жизненным циклом запроса
const container = new AdvancedDIContainer();

// Регистрация сервисов с разными жизненными циклами
container.register('config', () => loadConfig(), 'singleton');
container.register('database', (container) => new Database(container.resolve('config')), 'singleton');
container.register('userRepository', (container) => new UserRepository(container.resolve('database')), 'transient');
container.register('requestLogger', (container) => new RequestLogger(), 'scoped');

// Обработка HTTP-запроса с областью видимости
function handleRequest(req, res) {
  const scope = container.createScope();
  
  try {
    const requestLogger = scope.resolve('requestLogger');
    requestLogger.logRequest(req);
    
    const userRepository = scope.resolve('userRepository');
    // Обработка запроса...
    
  } finally {
    scope.dispose(); // Очистка области видимости
  }
}
```

## Заключение

Практические примеры показывают, как внедрение зависимостей применяется в реальных сценариях:

1. **Многоуровневые архитектуры** - DI помогает разделять ответственность между слоями
2. **Микросервисы** - DI облегчает интеграцию с внешними сервисами
3. **Тестирование** - DI позволяет легко подставлять моки и тестовые двойники
4. **Фреймворки** - DI интегрируется с различными фреймворками и платформами
5. **Clean Architecture** - DI поддерживает принципы разделения слоев
6. **Управление жизненным циклом** - DI позволяет управлять временем жизни объектов

Правильное использование DI делает приложения более гибкими, тестируемыми и поддерживаемыми. Ключ к успеху - это понимание того, когда и как использовать DI в конкретных сценариях.

См. также:
- [[Внедрение-зависимостей]]
- [[DI Patterns and Implementation]]
- [[Testing with Dependency Injection]]
- [[DI in Different Languages]]
- [[DI Frameworks Comparison]]
- [[Clean Architecture]]
- [[SOLID Principles]]
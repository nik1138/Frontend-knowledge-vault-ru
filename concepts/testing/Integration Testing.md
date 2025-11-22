---
aliases: ["Интеграционное тестирование", "Integration Tests", "Тестирование интеграции"]
tags: ["#testing", "#integration-testing", "#quality", "#development", "#best-practices"]
---

# Интеграционное тестирование (Integration Testing)

## Обзор

Интеграционное тестирование - это уровень тестирования программного обеспечения, который проверяет взаимодействие между различными компонентами или модулями системы. В отличие от модульного тестирования, которое проверяет отдельные части изолированно, интеграционное тестирование фокусируется на том, как эти части работают вместе и взаимодействуют друг с другом.

## Цели интеграционного тестирования

Основные цели интеграционного тестирования включают:

- Проверка корректного взаимодействия между компонентами
- Обнаружение проблем в интерфейсах между модулями
- Проверка потоков данных между компонентами
- Убедиться, что интегрированная система работает в соответствии с требованиями
- Проверка производительности при взаимодействии компонентов

## Типы интеграционного тестирования

### Big Bang интеграция

Все компоненты объединяются одновременно и тестируются как единое целое. Этот подход подходит для небольших систем:

**Преимущества:**
- Простота реализации
- Не требует промежуточных шагов

**Недостатки:**
- Сложно локализовать ошибки
- Высокий риск неудачного результата

### Top-Down интеграция

Тестирование начинается с верхних уровней архитектуры и постепенно включает нижестоящие компоненты:

```javascript
// Пример Top-Down подхода
class UserController {
  constructor(userService) {
    this.userService = userService;
  }
  
  async getUserProfile(userId) {
    return await this.userService.getUserWithDetails(userId);
  }
}

// Тестирование начинается с контроллера
describe('UserController', () => {
  let controller;
  let mockUserService;
  
  beforeEach(() => {
    mockUserService = {
      getUserWithDetails: jest.fn()
    };
    controller = new UserController(mockUserService);
  });
  
  test('should return user profile from service', async () => {
    const mockUser = { id: 1, name: 'John', details: {} };
    mockUserService.getUserWithDetails.mockResolvedValue(mockUser);
    
    const result = await controller.getUserProfile(1);
    
    expect(result).toEqual(mockUser);
    expect(mockUserService.getUserWithDetails).toHaveBeenCalledWith(1);
  });
});
```

### Bottom-Up интеграция

Тестирование начинается с нижних уровней и постепенно поднимаются к верхним:

```javascript
// Пример Bottom-Up подхода
class DatabaseService {
  async findUser(id) {
    // Реализация работы с базой данных
    return await db.users.findById(id);
  }
}

class UserService {
  constructor(dbService) {
    this.dbService = dbService;
  }
  
  async getUserWithDetails(id) {
    const user = await this.dbService.findUser(id);
    // Добавление деталей
    return { ...user, details: await this.getUserDetails(id) };
  }
}

// Сначала тестируем DatabaseService, затем UserService
describe('DatabaseService integration', () => {
  let dbService;
  
  beforeAll(() => {
    // Настройка реальной базы данных для тестирования
    dbService = new DatabaseService();
  });
  
  test('should find user by id', async () => {
    const user = await dbService.findUser(1);
    expect(user).toBeDefined();
    expect(user.id).toBe(1);
  });
});
```

### Sandwich (Hybrid) интеграция

Комбинация Top-Down и Bottom-Up подходов, тестирование начинается с середины архитектуры:

```javascript
// Пример комбинированного подхода
describe('Integration Testing - Sandwich Approach', () => {
  // Тестируем средний уровень (сервисы) с реальными зависимостями
  let userService;
  let notificationService;
  
  beforeAll(async () => {
    // Настройка реальных сервисов для тестирования
    userService = new UserService(realDatabase);
    notificationService = new NotificationService(realEmailProvider);
  });
  
  test('should send notification when user is created', async () => {
    const newUser = { name: 'John Doe', email: 'john@example.com' };
    
    await userService.createUser(newUser);
    
    // Проверка, что уведомление было отправлено
    expect(notificationService.sentNotifications).toContainEqual({
      to: newUser.email,
      subject: 'Welcome!'
    });
  });
});
```

## Подходы к интеграционному тестированию

### Тестирование API

API интеграционные тесты проверяют взаимодействие между различными сервисами через HTTP:

```javascript
const request = require('supertest');
const app = require('../app');
const { User, Order } = require('../models');

describe('API Integration Tests', () => {
  beforeAll(async () => {
    // Настройка тестовой базы данных
    await setupTestDatabase();
  });
  
  afterAll(async () => {
    await cleanupTestDatabase();
  });
  
  test('should create order and send notification', async () => {
    // Создание пользователя
    const user = await User.create({
      name: 'John Doe',
      email: 'john@example.com'
    });
    
    // Вызов API для создания заказа
    const response = await request(app)
      .post('/api/orders')
      .send({
        userId: user.id,
        items: [{ productId: 1, quantity: 2 }],
        total: 100
      })
      .expect(201);
    
    // Проверка ответа
    expect(response.body).toMatchObject({
      userId: user.id,
      status: 'pending',
      total: 100
    });
    
    // Проверка, что заказ был создан в базе
    const createdOrder = await Order.findByPk(response.body.id);
    expect(createdOrder).not.toBeNull();
    expect(createdOrder.userId).toBe(user.id);
  });
});
```

### Тестирование базы данных

Тестирование взаимодействия между приложением и базой данных:

```javascript
describe('Database Integration Tests', () => {
  let dbConnection;
  let userRepository;
  let orderRepository;
  
  beforeAll(async () => {
    dbConnection = await createTestDatabaseConnection();
    userRepository = new UserRepository(dbConnection);
    orderRepository = new OrderRepository(dbConnection);
  });
  
  afterAll(async () => {
    await dbConnection.close();
  });
  
  test('should create user and related orders', async () => {
    // Транзакция для обеспечения целостности данных
    const transaction = await dbConnection.beginTransaction();
    
    try {
      // Создание пользователя
      const user = await userRepository.create({
        name: 'John Doe',
        email: 'john@example.com'
      }, { transaction });
      
      // Создание заказов для пользователя
      const order1 = await orderRepository.create({
        userId: user.id,
        total: 50
      }, { transaction });
      
      const order2 = await orderRepository.create({
        userId: user.id,
        total: 75
      }, { transaction });
      
      // Подтверждение транзакции
      await transaction.commit();
      
      // Проверка создания данных
      const createdUser = await userRepository.findById(user.id);
      expect(createdUser).not.toBeNull();
      
      const userOrders = await orderRepository.findByUserId(user.id);
      expect(userOrders).toHaveLength(2);
      
    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  });
});
```

### Тестирование внешних сервисов

Тестирование взаимодействия с внешними API и сервисами:

```javascript
const nock = require('nock');

describe('External Service Integration Tests', () => {
  let paymentService;
  let notificationService;
  
  beforeAll(() => {
    // Мокирование внешнего API
    nock('https://api.payment-provider.com')
      .post('/charge')
      .reply(200, { success: true, transactionId: 'txn_123' });
    
    nock('https://api.email-service.com')
      .post('/send')
      .reply(200, { messageId: 'msg_456' });
    
    paymentService = new PaymentService();
    notificationService = new NotificationService();
  });
  
  test('should process payment and send confirmation', async () => {
    const paymentResult = await paymentService.charge({
      amount: 100,
      cardToken: 'tok_abc'
    });
    
    expect(paymentResult.success).toBe(true);
    expect(paymentResult.transactionId).toBe('txn_123');
    
    const notificationResult = await notificationService.sendConfirmation(
      'customer@example.com',
      paymentResult.transactionId
    );
    
    expect(notificationResult.messageId).toBe('msg_456');
  });
});
```

## Инструменты для интеграционного тестирования

### Docker для тестирования инфраструктуры

Использование Docker для создания изолированных тестовых окружений:

```yaml
# docker-compose.test.yml
version: '3.8'
services:
  test-db:
    image: postgres:13
    environment:
      POSTGRES_DB: test_db
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
    ports:
      - "5433:5432"
  
  test-redis:
    image: redis:6
    ports:
      - "6380:6379"
  
  test-app:
    build: .
    environment:
      DATABASE_URL: postgresql://test_user:test_password@test-db:5432/test_db
      REDIS_URL: redis://test-redis:6379
    depends_on:
      - test-db
      - test-redis
```

### TestContainers

TestContainers - это библиотека для тестирования с использованием Docker-контейнеров:

```java
// Пример использования TestContainers в Java
@Testcontainers
public class DatabaseIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:13")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");
    
    private DataSource dataSource;
    
    @BeforeEach
    void setUp() {
        dataSource = DataSourceBuilder.create()
                .url(postgres.getJdbcUrl())
                .username(postgres.getUsername())
                .password(postgres.getPassword())
                .build();
    }
    
    @Test
    void shouldSaveAndRetrieveUser() {
        // Тестирование с реальной базой данных
        UserRepository userRepository = new UserRepository(dataSource);
        
        User user = new User("John Doe", "john@example.com");
        userRepository.save(user);
        
        User retrievedUser = userRepository.findById(user.getId());
        
        assertThat(retrievedUser).isNotNull();
        assertThat(retrievedUser.getName()).isEqualTo("John Doe");
    }
}
```

## Практики интеграционного тестирования

### Тестирование с реальными зависимостями

```javascript
// Тестирование с реальной базой данных
describe('Real Database Integration Tests', () => {
  let app;
  let server;
  let testDatabase;
  
  beforeAll(async () => {
    // Запуск приложения с реальной конфигурацией
    app = await createApp({
      database: process.env.TEST_DATABASE_URL,
      redis: process.env.TEST_REDIS_URL
    });
    
    server = app.listen(3001);
    testDatabase = new TestDatabaseManager();
    await testDatabase.setup();
  });
  
  afterAll(async () => {
    await server.close();
    await testDatabase.cleanup();
  });
  
  beforeEach(async () => {
    await testDatabase.clearData();
  });
  
  test('should handle complete user registration flow', async () => {
    // Тестирование полного потока: API -> БД -> Email сервис
    const response = await request(app)
      .post('/api/register')
      .send({
        name: 'John Doe',
        email: 'john@example.com',
        password: 'password123'
      });
    
    expect(response.status).toBe(201);
    
    // Проверка, что пользователь создан в базе
    const user = await User.findOne({ where: { email: 'john@example.com' } });
    expect(user).not.toBeNull();
    
    // Проверка, что отправлено подтверждение по email
    const emailJobs = await EmailQueue.getCompletedJobs();
    expect(emailJobs).toHaveLength(1);
    expect(emailJobs[0].data.to).toBe('john@example.com');
  });
});
```

### Использование Test Data Builders

```javascript
// Паттерн Test Data Builder
class TestDataBuilder {
  static createUser(overrides = {}) {
    return {
      name: 'John Doe',
      email: `john${Date.now()}@example.com`,
      password: 'password123',
      ...overrides
    };
  }
  
  static createOrder(userId, overrides = {}) {
    return {
      userId,
      items: [{ productId: 1, quantity: 1, price: 100 }],
      total: 100,
      status: 'pending',
      ...overrides
    };
  }
}

// Использование в тестах
test('should process order with discount for premium user', async () => {
  const user = TestDataBuilder.createUser({ isPremium: true });
  const createdUser = await userService.createUser(user);
  
  const order = TestDataBuilder.createOrder(createdUser.id, { total: 100 });
  const processedOrder = await orderService.processOrder(order);
  
  expect(processedOrder.total).toBe(90); // 10% скидка для премиум пользователей
});
```

## Организация интеграционных тестов

### Структура тестов

```
tests/
├── integration/
│   ├── api/
│   │   ├── user.test.js
│   │   ├── order.test.js
│   │   └── auth.test.js
│   ├── database/
│   │   ├── user.repository.test.js
│   │   └── order.repository.test.js
│   ├── external/
│   │   ├── payment.test.js
│   │   └── notification.test.js
│   └── fixtures/
│       ├── database.js
│       └── server.js
```

### Настройка и очистка тестов

```javascript
// Фикстуры для интеграционных тестов
const TestSetup = {
  async setup() {
    // Настройка тестовой базы данных
    await setupTestDatabase();
    
    // Запуск тестового сервера
    this.app = await createApp(testConfig);
    this.server = this.app.listen(0); // Использование случайного порта
    
    // Ожидание готовности сервера
    await new Promise(resolve => {
      this.server.on('listening', resolve);
    });
    
    return this.app;
  },
  
  async teardown() {
    if (this.server) {
      await this.server.close();
    }
    await cleanupTestDatabase();
  },
  
  getServerAddress() {
    const address = this.server.address();
    return `http://localhost:${address.port}`;
  }
};

// Использование в тестах
describe('Integration Tests', () => {
  let serverAddress;
  
  beforeAll(async () => {
    await TestSetup.setup();
    serverAddress = TestSetup.getServerAddress();
  });
  
  afterAll(async () => {
    await TestSetup.teardown();
  });
  
  test('should handle complete API flow', async () => {
    const response = await fetch(`${serverAddress}/api/users`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: 'John', email: 'john@example.com' })
    });
    
    expect(response.status).toBe(201);
  });
});
```

## Лучшие практики

### Изолированные тестовые данные

Каждый тест должен использовать изолированные данные, чтобы избежать влияния одного теста на другой:

```javascript
describe('Isolated Test Data', () => {
  let testUser;
  
  beforeEach(async () => {
    // Создание уникальных тестовых данных для каждого теста
    testUser = await createUser({
      email: `test${Date.now()}@example.com`,
      name: 'Test User'
    });
  });
  
  afterEach(async () => {
    // Очистка данных после каждого теста
    if (testUser) {
      await deleteUser(testUser.id);
    }
  });
  
  test('should update user profile', async () => {
    await updateUser(testUser.id, { name: 'Updated Name' });
    
    const updatedUser = await getUserById(testUser.id);
    expect(updatedUser.name).toBe('Updated Name');
  });
});
```

### Проверка состояния системы

Интеграционные тесты должны проверять не только ответы, но и состояние системы:

```javascript
test('should create order and update inventory', async () => {
  // Исходное состояние
  const initialInventory = await getInventory(productId);
  
  // Выполнение действия
  await createOrder({ items: [{ productId, quantity: 5 }] });
  
  // Проверка результата
  const finalInventory = await getInventory(productId);
  expect(finalInventory.quantity).toBe(initialInventory.quantity - 5);
  
  // Проверка создания заказа
  const orders = await getOrdersByProductId(productId);
  expect(orders).toHaveLength(1);
});
```

## Интеграция с CI/CD

### Запуск интеграционных тестов в CI

```yaml
# .github/workflows/integration-tests.yml
name: Integration Tests
on: [push, pull_request]

jobs:
  integration-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:6
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379
```

## Антипаттерны интеграционного тестирования

### Слишком много моков

Использование чрезмерного количества моков в интеграционных тестах сводит их смысл к нулю:

```javascript
// Плохо: Все мокировано, это не интеграционный тест
test('mocked integration test', async () => {
  const mockDb = jest.fn();
  const mockEmail = jest.fn();
  const mockPayment = jest.fn();
  
  // Все зависимости замокированы - это модульный тест
});
```

### Недостаточное покрытие потоков

```javascript
// Плохо: Тестирование только успешного сценария
test('should process order', async () => {
  // Только happy path
});

// Хорошо: Покрытие всех сценариев
test('should process order successfully', async () => {
  // Happy path
});

test('should handle payment failure', async () => {
  // Обработка ошибок
});

test('should rollback on database error', async () => {
  // Обработка отката транзакции
});
```

## Заключение

Интеграционное тестирование играет критически важную роль в обеспечении качества программного обеспечения, проверяя, как различные компоненты системы работают вместе. Эффективные интеграционные тесты помогают обнаружить проблемы на ранних стадиях разработки, снижают риски при развертывании и обеспечивают надежность приложений.

Связанные темы: [[Unit Testing]], [[End-to-End Testing]], [[Testing Strategies]], [[CI/CD Best Practices]], [[Quality Assurance]]

Теги: #testing #integration-testing #quality #development #best-practices #api-testing #database-testing
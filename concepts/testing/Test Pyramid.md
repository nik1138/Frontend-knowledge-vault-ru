---
aliases: [Тестовая пирамида, Пирамида тестирования]
tags: [testing, quality-assurance, software-testing, automation]
---

# Тестовая пирамида

## Обзор

Тестовая пирамида (Test Pyramid) - это концепция, предложенная Майком Кунсом (Mike Cohn), которая описывает оптимальное распределение различных типов автоматизированных тестов в проекте. Пирамида демонстрирует, что большинство тестов должны быть модульными (unit), меньшее количество - интеграционными, и наименьшее количество - сквозными (end-to-end).

```
                E2E Tests (Few)
                     /|\
                    / | \
                   /  |  \
                  /   |   \
                 /    |    \
                /     |     \
               /      |      \
              /       |       \
             /        |        \
            /         |         \
           /          |          \
          /           |           \
         /            |            \
        /             |             \
       /              |              \
      /               |               \
     /                |                \
    /                 |                 \
   /                  |                  \
  /                   |                   \
 /                    |                    \
/                     |                     \
Unit Tests (Many)  Integration Tests (Some)
```

## Компоненты тестовой пирамиды

### Модульные тесты (Unit Tests)

Модульные тесты - это основа пирамиды, составляющие 70-80% всех тестов. Они тестируют отдельные компоненты системы изолированно.

#### Характеристики модульных тестов:
- **Быстрые**: Обычно выполняются за миллисекунды
- **Изолированные**: Не зависят от внешних систем
- **Надежные**: Маловероятны ложные срабатывания
- **Легко поддерживаемые**: Просты в написании и обслуживании
- **Часто запускаемые**: Выполняются при каждом изменении кода

#### Пример модульного теста

```javascript
// Пример модульного теста для функции сложения
function add(a, b) {
  return a + b;
}

describe('Add function', () => {
  test('should add two positive numbers correctly', () => {
    const result = add(2, 3);
    expect(result).toBe(5);
  });

  test('should handle negative numbers', () => {
    const result = add(-1, 1);
    expect(result).toBe(0);
  });

  test('should handle decimal numbers', () => {
    const result = add(0.1, 0.2);
    expect(result).toBeCloseTo(0.3);
  });
});
```

#### Пример модульного теста для класса

```javascript
// Класс калькулятора
class Calculator {
  add(a, b) {
    return a + b;
  }

  subtract(a, b) {
    return a - b;
  }

  multiply(a, b) {
    return a * b;
  }

  divide(a, b) {
    if (b === 0) {
      throw new Error('Division by zero');
    }
    return a / b;
  }
}

// Модульные тесты для калькулятора
describe('Calculator', () => {
  let calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  test('should add two numbers correctly', () => {
    const result = calculator.add(2, 3);
    expect(result).toBe(5);
  });

  test('should subtract two numbers correctly', () => {
    const result = calculator.subtract(5, 3);
    expect(result).toBe(2);
  });

  test('should throw error when dividing by zero', () => {
    expect(() => {
      calculator.divide(5, 0);
    }).toThrow('Division by zero');
  });
});
```

### Интеграционные тесты (Integration Tests)

Интеграционные тесты составляют средний уровень пирамиды (15-20% всех тестов). Они проверяют взаимодействие между компонентами системы.

#### Характеристики интеграционных тестов:
- **Медленнее модульных**: Обычно выполняются за секунды
- **Требуют больше ресурсов**: Может потребоваться база данных или внешние API
- **Сложнее в отладке**: Ошибки могут быть вызваны несколькими компонентами
- **Проверяют взаимодействие**: Убеждаются, что компоненты работают вместе

#### Пример интеграционного теста

```javascript
// Пример интеграционного теста для сервиса пользователей
const User = require('../models/User');
const UserService = require('../services/UserService');
const Database = require('../config/database');

describe('UserService Integration', () => {
  beforeAll(async () => {
    await Database.connect();
  });

  afterAll(async () => {
    await Database.disconnect();
  });

  beforeEach(async () => {
    await User.deleteMany({});
  });

  test('should create user and return user object', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      password: 'securepassword'
    };

    const user = await UserService.createUser(userData);
    
    expect(user).toBeDefined();
    expect(user.name).toBe(userData.name);
    expect(user.email).toBe(userData.email);
    
    // Проверка, что пользователь действительно сохранен в БД
    const savedUser = await User.findOne({ email: userData.email });
    expect(savedUser).toBeDefined();
    expect(savedUser.name).toBe(userData.name);
  });

  test('should not create user with duplicate email', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      password: 'securepassword'
    };

    await UserService.createUser(userData);
    
    await expect(
      UserService.createUser(userData)
    ).rejects.toThrow('User with this email already exists');
  });
});
```

### Сквозные тесты (End-to-End Tests)

Сквозные тесты составляют вершину пирамиды (5-10% всех тестов). Они проверяют полные сценарии использования приложения от начала до конца.

#### Характеристики E2E тестов:
- **Самые медленные**: Могут занимать минуты
- **Наиболее хрупкие**: Часто ломаются при небольших изменениях интерфейса
- **Наиболее дорогие**: Требуют больше ресурсов и времени
- **Наиболее ценные**: Проверяют пользовательский опыт

#### Пример E2E теста

```javascript
// Пример E2E теста с использованием Cypress
describe('User Registration Flow', () => {
  beforeEach(() => {
    cy.visit('/register');
  });

  it('should register a new user successfully', () => {
    cy.get('[data-cy=username]').type('testuser');
    cy.get('[data-cy=email]').type('test@example.com');
    cy.get('[data-cy=password]').type('Password123!');
    cy.get('[data-cy=confirmPassword]').type('Password123!');
    cy.get('[data-cy=terms]').check();
    cy.get('[data-cy=submit]').click();

    cy.url().should('include', '/dashboard');
    cy.get('[data-cy=welcome-message]').should('contain', 'Welcome, testuser!');
  });

  it('should show validation errors for invalid input', () => {
    cy.get('[data-cy=submit]').click();

    cy.get('[data-cy=email-error]').should('contain', 'Email is required');
    cy.get('[data-cy=password-error]').should('contain', 'Password is required');
  });
});
```

## Пропорции и распределение

### Классическая пропорция пирамиды

```
Модульные тесты:    70%
Интеграционные:     20%
Сквозные:           10%
```

### Альтернативные подходы

#### Пирамида тестирования в микросервисах

```
Модульные тесты:    60% (модульные тесты сервисов)
Интеграционные:     25% (тесты взаимодействия между сервисами)
Сквозные:           15% (тесты пользовательских сценариев)
```

#### Пирамида тестирования для API

```
Модульные тесты:    75% (тесты отдельных функций и контроллеров)
Интеграционные:     20% (тесты взаимодействия с БД и внешними API)
Сквозные:            5% (тесты полного пользовательского сценария)
```

## Стратегии реализации

### 1. Начало с модульных тестов

```javascript
// Начинаем с модульных тестов для бизнес-логики
class OrderProcessor {
  constructor(inventoryService, paymentService) {
    this.inventoryService = inventoryService;
    this.paymentService = paymentService;
  }

  async processOrder(order) {
    // Проверяем наличие товара
    const isAvailable = await this.inventoryService.checkAvailability(
      order.items
    );
    
    if (!isAvailable) {
      throw new Error('Items not available');
    }

    // Обрабатываем оплату
    const paymentResult = await this.paymentService.processPayment(
      order.total,
      order.paymentMethod
    );

    if (!paymentResult.success) {
      throw new Error('Payment failed');
    }

    // Обновляем инвентарь
    await this.inventoryService.reserveItems(order.items);

    return {
      orderId: generateOrderId(),
      status: 'processed',
      paymentId: paymentResult.paymentId
    };
  }
}

// Модульные тесты для OrderProcessor
describe('OrderProcessor', () => {
  let orderProcessor;
  let mockInventoryService;
  let mockPaymentService;

  beforeEach(() => {
    mockInventoryService = {
      checkAvailability: jest.fn(),
      reserveItems: jest.fn()
    };
    
    mockPaymentService = {
      processPayment: jest.fn()
    };
    
    orderProcessor = new OrderProcessor(
      mockInventoryService,
      mockPaymentService
    );
  });

  test('should process order successfully when items are available', async () => {
    const order = {
      items: [{ id: 1, quantity: 2 }],
      total: 100,
      paymentMethod: 'credit_card'
    };

    mockInventoryService.checkAvailability.mockResolvedValue(true);
    mockPaymentService.processPayment.mockResolvedValue({
      success: true,
      paymentId: 'payment_123'
    });

    const result = await orderProcessor.processOrder(order);

    expect(result.status).toBe('processed');
    expect(mockInventoryService.reserveItems).toHaveBeenCalledWith(order.items);
  });
});
```

### 2. Постепенное добавление интеграционных тестов

```javascript
// Интеграционный тест для проверки взаимодействия между сервисами
describe('Order Processing Integration', () => {
  let app;
  let server;

  beforeAll(async () => {
    app = await createApp();
    server = app.listen(3000);
  });

  afterAll(async () => {
    await server.close();
  });

  test('should process order through HTTP API', async () => {
    const orderData = {
      items: [{ id: 1, quantity: 2 }],
      total: 100,
      paymentMethod: 'credit_card'
    };

    const response = await request(app)
      .post('/api/orders')
      .send(orderData)
      .expect(200);

    expect(response.body.status).toBe('processed');
    
    // Проверяем, что заказ действительно сохранен в БД
    const order = await Order.findById(response.body.orderId);
    expect(order).toBeDefined();
  });
});
```

### 3. Создание ключевых E2E тестов

```javascript
// E2E тест для ключевого пользовательского сценария
describe('Complete Purchase Flow', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  test('user can complete purchase from product selection to order confirmation', async () => {
    // 1. Пользователь заходит на сайт
    await page.goto('https://example.com');
    
    // 2. Поиск и выбор продукта
    await page.fill('input[name="search"]', 'laptop');
    await page.click('button[type="submit"]');
    await page.click('a[href="/product/123"]');
    
    // 3. Добавление в корзину
    await page.click('button[data-cy="add-to-cart"]');
    
    // 4. Переход к оформлению заказа
    await page.click('a[href="/cart"]');
    await page.click('button[data-cy="checkout"]');
    
    // 5. Ввод данных доставки
    await page.fill('input[name="address"]', '123 Main St');
    await page.fill('input[name="city"]', 'New York');
    
    // 6. Ввод данных оплаты
    await page.fill('input[name="card-number"]', '4111111111111111');
    await page.fill('input[name="expiry"]', '12/25');
    await page.fill('input[name="cvv"]', '123');
    
    // 7. Подтверждение заказа
    await page.click('button[data-cy="place-order"]');
    
    // 8. Проверка подтверждения
    await expect(page).toHaveURL(/confirmation/);
    await expect(page.locator('h1')).toContainText('Order Confirmed');
  });
});
```

## Примеры для разных технологий

### Тестовая пирамида для React-приложения

```javascript
// Модульный тест компонента
import { render, screen } from '@testing-library/react';
import { Calculator } from './Calculator';

test('renders calculator with buttons', () => {
  render(<Calculator />);
  expect(screen.getByText('0')).toBeInTheDocument();
  expect(screen.getByText('+')).toBeInTheDocument();
  expect(screen.getByText('-')).toBeInTheDocument();
});

// Интеграционный тест с контекстом
import { render, fireEvent } from '@testing-library/react';
import { CalculatorProvider } from './CalculatorContext';
import { Calculator } from './Calculator';

test('calculator performs addition', () => {
  render(
    <CalculatorProvider>
      <Calculator />
    </CalculatorProvider>
  );
  
  fireEvent.click(screen.getByText('2'));
  fireEvent.click(screen.getByText('+'));
  fireEvent.click(screen.getByText('3'));
  fireEvent.click(screen.getByText('='));
  
  expect(screen.getByText('5')).toBeInTheDocument();
});

// E2E тест с Cypress
describe('React App E2E', () => {
  it('should allow user to calculate values', () => {
    cy.visit('/calculator');
    cy.get('[data-cy=button-2]').click();
    cy.get('[data-cy=operator-plus]').click();
    cy.get('[data-cy=button-3]').click();
    cy.get('[data-cy=equals]').click();
    cy.get('[data-cy=result]').should('have.text', '5');
  });
});
```

### Тестовая пирамида для Node.js API

```javascript
// Модульный тест сервиса
const UserService = require('../services/UserService');

describe('UserService', () => {
  test('should hash password before saving', async () => {
    const plainPassword = 'password123';
    const hashedPassword = await UserService.hashPassword(plainPassword);
    
    expect(hashedPassword).not.toBe(plainPassword);
    expect(await UserService.verifyPassword(plainPassword, hashedPassword)).toBe(true);
  });
});

// Интеграционный тест контроллера
const request = require('supertest');
const app = require('../app');

describe('User Controller', () => {
  test('should create user successfully', async () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      password: 'password123'
    };

    const response = await request(app)
      .post('/api/users')
      .send(userData)
      .expect(201);

    expect(response.body.user.name).toBe(userData.name);
    expect(response.body.user.email).toBe(userData.email);
  });
});
```

## Практические рекомендации

### 1. Мониторинг соотношения тестов

```javascript
// Скрипт для анализа соотношения тестов
const fs = require('fs');
const path = require('path');

function analyzeTestPyramid(testDir) {
  const files = fs.readdirSync(testDir);
  
  const unitTests = files.filter(file => 
    file.includes('.unit.test.js') || file.includes('.spec.js')
  ).length;
  
  const integrationTests = files.filter(file => 
    file.includes('.integration.test.js') || file.includes('.int.spec.js')
  ).length;
  
  const e2eTests = files.filter(file => 
    file.includes('.e2e.test.js') || file.includes('.e2e.spec.js')
  ).length;
  
  const total = unitTests + integrationTests + e2eTests;
  
  console.log('Test Pyramid Analysis:');
  console.log(`Unit Tests: ${unitTests} (${((unitTests/total)*100).toFixed(2)}%)`);
  console.log(`Integration Tests: ${integrationTests} (${((integrationTests/total)*100).toFixed(2)}%)`);
  console.log(`E2E Tests: ${e2eTests} (${((e2eTests/total)*100).toFixed(2)}%)`);
  
  return {
    unit: unitTests,
    integration: integrationTests,
    e2e: e2eTests,
    total: total
  };
}

// Использование
const pyramid = analyzeTestPyramid('./tests');
```

### 2. Проверка качества тестов

```javascript
// Пример проверки качества модульных тестов
function validateUnitTestQuality(testFunction) {
  // Проверяем, что тест изолирован
  const isIsolated = testFunction.toString().includes('mock') || 
                     testFunction.toString().includes('spy') ||
                     testFunction.toString().includes('stub');
  
  // Проверяем, что тест быстрый
  const hasTimeout = testFunction.toString().includes('timeout') ||
                     testFunction.toString().includes('wait');
  
  // Проверяем, что тест тестирует одну вещь
  const assertionCount = (testFunction.toString().match(/expect\(/g) || []).length;
  
  return {
    isolated: isIsolated,
    fast: !hasTimeout,
    focused: assertionCount <= 3, // Не более 3 проверок на тест
    quality: isIsolated && !hasTimeout && assertionCount <= 3
  };
}
```

### 3. Автоматизация проверки пирамиды

```yaml
# GitHub Actions для проверки соотношения тестов
name: Test Pyramid Check
on: [push, pull_request]

jobs:
  check-pyramid:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
      - run: npm ci
      - name: Analyze test pyramid
        run: node scripts/check-test-pyramid.js
      - name: Validate proportions
        run: |
          node scripts/validate-test-proportions.js
          if [ $? -ne 0 ]; then
            echo "Test pyramid proportions are not optimal"
            exit 1
          fi
```

## Преимущества и недостатки

### Преимущества тестовой пирамиды

- **Быстрая обратная связь**: Множество быстрых модульных тестов
- **Экономия ресурсов**: Меньше ресурсов требуется для выполнения тестов
- **Легкая отладка**: Ошибки легко локализуются
- **Надежность**: Меньше ложных срабатываний
- **Покрытие**: Хорошее покрытие кода

### Возможные недостатки

- **Сложность понимания**: Новичкам может быть сложно понять
- **Неправильное применение**: Могут создавать только модульные тесты
- **Недостаточное покрытие пользовательских сценариев**: Слишком мало E2E тестов
- **Переусложнение**: Может быть сложно поддерживать баланс

## Альтернативные модели

### Пирамида тестирования по Лизе Криспин

```
Сквозные тесты (Customer Tests) - 10%
Интеграционные тесты (Service Tests) - 20%
Модульные тесты (Unit Tests) - 70%
```

### Модель "леденец на палочке"

```
Сквозные тесты - 10%
Интеграционные тесты - 10%
Модульные тесты - 80%
```

### Модель "обратная пирамида" (для некоторых специфичных случаев)

```
Сквозные тесты - 50%
Интеграционные тесты - 30%
Модульные тесты - 20%
```

## Заключение

Тестовая пирамида остается фундаментальным принципом в автоматизации тестирования, обеспечивая баланс между скоростью, надежностью и покрытием. Успешная реализация пирамиды тестирования требует:

- Понимания пропорций и целей каждого уровня тестирования
- Правильного подхода к написанию тестов каждого типа
- Регулярного анализа и оптимизации соотношения тестов
- Интеграции в процесс CI/CD
- Обучения команды принципам тестирования

Правильно реализованная тестовая пирамида значительно повышает качество программного обеспечения и ускоряет процесс разработки за счет быстрой обратной связи и надежного покрытия.

[[Testing Strategies]] [[Quality Assurance]] [[Automated Testing]] [[CI/CD Pipeline]]
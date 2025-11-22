---
aliases: ["Модульное тестирование", "Unit Tests", "Тестирование компонентов"]
tags: ["#testing", "#unit-testing", "#quality", "#development", "#best-practices"]
---

# Модульное тестирование (Unit Testing)

## Обзор

Модульное тестирование - это метод тестирования программного обеспечения, при котором отдельные компоненты или модули приложения тестируются изолированно. Цель модульного тестирования - проверить, что каждый модуль работает правильно по отдельности, что позволяет обнаружить и устранить ошибки на ранних стадиях разработки.

## Принципы модульного тестирования

### FIRST принципы

- **Fast** (Быстрые): Модульные тесты должны выполняться быстро, чтобы разработчики могли запускать их часто
- **Independent** (Независимые): Тесты не должны зависеть друг от друга и могут выполняться в любом порядке
- **Repeatable** (Повторяемые): Результаты тестов должны быть одинаковыми в любой среде
- **Self-validating** (Самопроверяемые): Тесты должны автоматически определять, пройден тест или нет
- **Timely** (Своевременные): Тесты должны быть написаны вовремя, желательно перед тестируемым кодом

### AAA паттерн

AAA (Arrange-Act-Assert) - это распространенный паттерн структурирования модульных тестов:

1. **Arrange** (Подготовка): Подготовка данных и объектов для теста
2. **Act** (Действие): Выполнение тестируемой функции или метода
3. **Assert** (Проверка): Проверка результата на соответствие ожиданиям

```javascript
// Пример теста по паттерну AAA
describe('Calculator', () => {
  test('should add two numbers correctly', () => {
    // Arrange
    const calculator = new Calculator();
    const a = 5;
    const b = 3;
    
    // Act
    const result = calculator.add(a, b);
    
    // Assert
    expect(result).toBe(8);
  });
});
```

## Фреймворки для модульного тестирования

### Jest

Jest - это популярный фреймворк для тестирования JavaScript и TypeScript приложений:

```javascript
// Пример использования Jest
describe('UserService', () => {
  let userService;
  let userRepository;
  
  beforeEach(() => {
    userRepository = new MockUserRepository();
    userService = new UserService(userRepository);
  });
  
  describe('getUserById', () => {
    test('should return user when user exists', async () => {
      // Arrange
      const userId = 1;
      const expectedUser = { id: userId, name: 'John Doe' };
      userRepository.findById.mockResolvedValue(expectedUser);
      
      // Act
      const result = await userService.getUserById(userId);
      
      // Assert
      expect(result).toEqual(expectedUser);
      expect(userRepository.findById).toHaveBeenCalledWith(userId);
    });
    
    test('should throw error when user does not exist', async () => {
      // Arrange
      const userId = 999;
      userRepository.findById.mockResolvedValue(null);
      
      // Act & Assert
      await expect(userService.getUserById(userId))
        .rejects
        .toThrow('User not found');
    });
  });
});
```

### Mocha + Chai

Mocha - это гибкий фреймворк для тестирования, часто используемый с библиотекой утверждений Chai:

```javascript
const { expect } = require('chai');
const Calculator = require('../src/Calculator');

describe('Calculator', function() {
  let calculator;
  
  beforeEach(function() {
    calculator = new Calculator();
  });
  
  describe('add', function() {
    it('should add two positive numbers correctly', function() {
      // Arrange
      const a = 5;
      const b = 3;
      const expected = 8;
      
      // Act
      const result = calculator.add(a, b);
      
      // Assert
      expect(result).to.equal(expected);
    });
    
    it('should handle negative numbers', function() {
      // Arrange & Act
      const result = calculator.add(-5, 3);
      
      // Assert
      expect(result).to.equal(-2);
    });
  });
});
```

### JUnit (Java)

JUnit - это фреймворк для модульного тестирования в Java:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    
    @Test
    public void shouldAddTwoNumbersCorrectly() {
        // Arrange
        Calculator calculator = new Calculator();
        
        // Act
        int result = calculator.add(5, 3);
        
        // Assert
        assertEquals(8, result, "5 + 3 should equal 8");
    }
    
    @Test
    public void shouldHandleZeroCorrectly() {
        // Arrange
        Calculator calculator = new Calculator();
        
        // Act & Assert
        assertEquals(5, calculator.add(5, 0));
        assertEquals(5, calculator.add(0, 5));
    }
}
```

## Паттерны модульного тестирования

### Arrange-Act-Assert (AAA)

Уже рассмотрен выше, это основной паттерн организации тестов.

### Given-When-Then

Альтернативный подход, особенно популярный в BDD:

```javascript
describe('Shopping Cart', () => {
  test('should calculate total price correctly', () => {
    // Given - начальное состояние
    const cart = new ShoppingCart();
    const item1 = { price: 10, quantity: 2 };
    const item2 = { price: 15, quantity: 1 };
    
    // When - выполнение действия
    cart.addItem(item1);
    cart.addItem(item2);
    const total = cart.getTotalPrice();
    
    // Then - проверка результата
    expect(total).toBe(35); // (10*2) + (15*1) = 35
  });
});
```

### Object Mother

Паттерн для создания тестовых объектов:

```javascript
class UserObjectMother {
  static createValidUser() {
    return {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      age: 30
    };
  }
  
  static createInvalidUser() {
    return {
      id: null,
      name: '',
      email: 'invalid-email',
      age: -1
    };
  }
}

// Использование в тестах
test('should validate user correctly', () => {
  const validUser = UserObjectMother.createValidUser();
  const invalidUser = UserObjectMother.createInvalidUser();
  
  expect(validateUser(validUser)).toBe(true);
  expect(validateUser(invalidUser)).toBe(false);
});
```

## Мокирование и стабирование

### Мокирование

Мокирование позволяет заменить зависимости реального объекта контролируемыми заглушками:

```javascript
// Мокирование внешней зависимости
jest.mock('../services/EmailService');

const EmailService = require('../services/EmailService');
const UserService = require('./UserService');

describe('UserService', () => {
  test('should send welcome email when user registers', async () => {
    // Arrange
    const mockEmailService = new EmailService();
    const userService = new UserService(mockEmailService);
    const newUser = { id: 1, email: 'user@example.com' };
    
    // Настройка мока
    mockEmailService.sendWelcomeEmail.mockResolvedValue(true);
    
    // Act
    await userService.registerUser(newUser);
    
    // Assert
    expect(mockEmailService.sendWelcomeEmail)
      .toHaveBeenCalledWith(newUser.email);
  });
});
```

### Спай (Spy)

Spy позволяет отслеживать вызовы методов без изменения их поведения:

```javascript
test('should call logger when processing order', () => {
  const logger = {
    log: jest.fn()
  };
  
  const orderProcessor = new OrderProcessor(logger);
  const order = { id: 1, amount: 100 };
  
  orderProcessor.process(order);
  
  // Проверяем, что метод был вызван
  expect(logger.log).toHaveBeenCalledWith(
    `Processing order ${order.id}`
  );
  expect(logger.log).toHaveBeenCalledTimes(1);
});
```

### Stub

Stub предоставляет предопределенные ответы для методов:

```javascript
// Создание stub для API сервиса
const createApiStub = () => ({
  getUsers: jest.fn().mockResolvedValue([
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' }
  ]),
  getUserById: jest.fn().mockResolvedValue({ id: 1, name: 'John' })
});

test('should fetch users from API', async () => {
  const apiStub = createApiStub();
  const userService = new UserService(apiStub);
  
  const users = await userService.getAllUsers();
  
  expect(users).toHaveLength(2);
  expect(apiStub.getUsers).toHaveBeenCalled();
});
```

## TDD (Test-Driven Development)

TDD - это практика разработки, при которой тесты пишутся до реализации кода. Процесс включает три этапа:

1. **Red** (Красный): Написание теста, который заведомо не проходит
2. **Green** (Зеленый): Написание минимального кода для прохождения теста
3. **Refactor** (Рефакторинг): Улучшение кода без изменения его поведения

```javascript
// Пример TDD процесса

// 1. Red: Написание теста (тест не проходит)
describe('StringCalculator', () => {
  test('should return 0 for empty string', () => {
    const calculator = new StringCalculator();
    expect(calculator.add('')).toBe(0);
  });
});

// 2. Green: Минимальная реализация
class StringCalculator {
  add(input) {
    return 0; // Просто чтобы тест прошел
  }
}

// 3. Refactor: Улучшение кода (добавление новых тестов и реализации)
class StringCalculator {
  add(input) {
    if (input === '') return 0;
    if (input === '1') return 1;
    return 0;
  }
}

// Продолжение TDD цикла...
```

## Покрытие кода (Code Coverage)

Покрытие кода показывает, какой процент кода приложения охвачен тестами:

```javascript
// Пример настройки покрытия в Jest
// jest.config.js
module.exports = {
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  coverageReporters: ['text', 'lcov', 'html']
};
```

### Типы покрытия

- **Statement coverage**: Процент выполненных строк кода
- **Branch coverage**: Процент пройденных ветвлений в коде
- **Function coverage**: Процент вызванных функций
- **Line coverage**: Процент выполненных строк кода

## Практики и шаблоны тестирования

### Параметризованные тесты

Тестирование с различными входными данными:

```javascript
describe('Validator', () => {
  test.each([
    ['john@example.com', true],
    ['invalid-email', false],
    ['', false],
    ['user@domain.co.uk', true]
  ])('should validate email %s as %s', (email, expected) => {
    expect(validateEmail(email)).toBe(expected);
  });
});
```

### Тестирование исключений

```javascript
test('should throw error for negative numbers', () => {
  const calculator = new Calculator();
  
  expect(() => {
    calculator.sqrt(-1);
  }).toThrow('Cannot calculate square root of negative number');
  
  // Для асинхронных функций
  expect(async () => {
    await calculator.asyncOperation(-1);
  }).rejects.toThrow('Invalid input');
});
```

### Тестирование асинхронного кода

```javascript
test('should fetch user data correctly', async () => {
  // Arrange
  const mockUser = { id: 1, name: 'John' };
  global.fetch = jest.fn().mockResolvedValue({
    json: jest.fn().mockResolvedValue(mockUser)
  });
  
  // Act
  const user = await fetchUser(1);
  
  // Assert
  expect(user).toEqual(mockUser);
  expect(global.fetch).toHaveBeenCalledWith('/api/users/1');
});
```

## Качество модульных тестов

### Характеристики хороших тестов

1. **Быстрые**: Тесты должны выполняться быстро
2. **Надежные**: Не должны давать ложные срабатывания
3. **Читаемые**: Легко понимаемые другими разработчиками
4. **Поддерживаемые**: Легко изменяемые при изменении кода
5. **Полные**: Проверяют все важные сценарии

### Антипаттерны тестирования

- **Зависимые тесты**: Один тест зависит от результата другого
- **Случайные тесты**: Результаты меняются от запуска к запуску
- **Тесты с побочными эффектами**: Меняют состояние системы
- **Слишком сложные тесты**: Требуют много времени для понимания
- **Повторяющиеся тесты**: Дублируют проверки

## Организация тестов

### Структура тестов

```javascript
describe('UserService', () => {
  // Подготовка данных
  let userService;
  let userRepository;
  
  beforeEach(() => {
    userRepository = new MockUserRepository();
    userService = new UserService(userRepository);
  });
  
  describe('getUserById', () => {
    test('should return user when exists', async () => {
      // Тестовая логика
    });
    
    test('should throw error when not found', async () => {
      // Тестовая логика
    });
  });
  
  describe('createUser', () => {
    test('should create user with valid data', async () => {
      // Тестовая логика
    });
  });
});
```

### Именование тестов

Хорошие имена тестов должны описывать:
- Что тестируется
- При каких условиях
- Что должно произойти

```javascript
// Плохо
test('test1', () => { ... });

// Хорошо
test('should return user profile when valid ID provided', () => { ... });
test('should throw validation error when email is invalid', () => { ... });
```

## Интеграция с CI/CD

### Запуск тестов в CI

```yaml
# .github/workflows/test.yml
name: Unit Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16.x, 18.x]
    
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

### Проверка покрытия

```bash
# Команда для проверки покрытия
npm test -- --coverage --coverageThreshold='{"global":{"lines":80,"functions":80,"branches":80}}'
```

## Лучшие практики

### Практики написания тестов

1. **Тестирование поведения, а не реализации**: Тесты должны проверять, что код делает, а не как
2. **Одна проверка на тест**: Каждый тест должен проверять одну конкретную вещь
3. **Изолированные тесты**: Тесты не должны зависеть от внешних факторов
4. **Самодокументирующиеся тесты**: Имена тестов должны быть понятными
5. **Минимизация моков**: Использовать моки только когда необходимо

### Организация тестов

- Разделение тестов по функциональности
- Использование описательных имен
- Поддержание чистоты тестовой среды
- Регулярное обновление тестов при изменении кода

## Заключение

Модульное тестирование является важной частью процесса разработки программного обеспечения, обеспечивающей качество, надежность и поддерживаемость кода. Правильное применение принципов модульного тестирования, использование подходящих фреймворков и соблюдение лучших практик позволяет создавать надежные и легко сопровождаемые приложения.

Связанные темы: [[Integration Testing]], [[End-to-End Testing]], [[Testing Strategies]], [[CI/CD Best Practices]], [[Code Quality]]

Теги: #testing #unit-testing #quality #development #best-practices #tdd #mocking #coverage
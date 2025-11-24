---
aliases: ["Mock типы", "Mock Types"]
tags: [typescript, testing, mock, типизация]
---

# Mock-типы в TypeScript

## Обзор

Mock-типы в TypeScript - это специальные типы, используемые для создания mock-объектов в тестах. Они позволяют заменить реальные зависимости на контролируемые объекты, что упрощает тестирование изолированных компонентов и функций.

## Зачем нужны Mock-типы?

Mock-типы важны для тестирования по следующим причинам:

1. **Изоляция тестируемого кода** - Отделяет тестируемую логику от внешних зависимостей
2. **Контроль над поведением зависимостей** - Позволяет задавать ожидаемое поведение mock-объектов
3. **Предсказуемость тестов** - Устраняет влияние внешних факторов на результаты тестов
4. **Ускорение тестов** - Избегает медленных операций вроде сетевых вызовов или работы с базой данных

## Основные концепции

### Создание Mock-типов

Mock-типы создаются на основе существующих интерфейсов или классов. Вот несколько подходов:

#### Использование Partial для создания mock-объектов

```typescript
interface UserService {
  getUser(id: number): Promise<User>;
  createUser(userData: CreateUserRequest): Promise<User>;
  updateUser(id: number, userData: UpdateUserRequest): Promise<User>;
  deleteUser(id: number): Promise<void>;
}

// Тип для mock-объекта
type MockUserService = Partial<UserService>;

// Создание mock-объекта
const mockUserService: MockUserService = {
  getUser: jest.fn().mockResolvedValue({ id: 1, name: 'John Doe', email: 'john@example.com' }),
  createUser: jest.fn().mockResolvedValue({ id: 2, name: 'Jane Doe', email: 'jane@example.com' })
};
```

#### Создание конкретного типа для mock-объекта

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

interface CreateUserRequest {
  name: string;
  email: string;
}

interface UpdateUserRequest {
  name?: string;
  email?: string;
}

// Тип для mock-объекта с методами-заглушками
type UserServiceMock = {
  getUser: jest.MockedFunction<(id: number) => Promise<User>>;
  createUser: jest.MockedFunction<(userData: CreateUserRequest) => Promise<User>>;
  updateUser: jest.MockedFunction<(id: number, userData: UpdateUserRequest) => Promise<User>>;
  deleteUser: jest.MockedFunction<(id: number) => Promise<void>>;
};

// Создание mock-объекта с типизацией
const createUserServiceMock = (): UserServiceMock => ({
  getUser: jest.fn(),
  createUser: jest.fn(),
  updateUser: jest.fn(),
  deleteUser: jest.fn()
});
```

## Практические примеры

### Mock-классы

```typescript
// Реальный класс
class DatabaseService {
  async findUser(id: number): Promise<User | null> {
    // Реализация обращения к базе данных
    return null;
  }

  async saveUser(user: User): Promise<User> {
    // Реализация сохранения в базу данных
    return user;
  }
}

// Mock-класс
class MockDatabaseService {
  private users: User[] = [];

  async findUser(id: number): Promise<User | null> {
    return this.users.find(user => user.id === id) || null;
  }

  async saveUser(user: User): Promise<User> {
    this.users.push(user);
    return user;
  }
}

// Использование в тесте
describe('UserService', () => {
  let userService: UserService;
  let mockDatabase: MockDatabaseService;

  beforeEach(() => {
    mockDatabase = new MockDatabaseService();
    userService = new UserService(mockDatabase);
  });

  it('должен создать пользователя', async () => {
    const newUser: User = { id: 999, name: 'Test User', email: 'test@example.com' };
    
    const result: User = await userService.createUser(newUser);
    
    expect(result).toEqual(newUser);
  });
});
```

### Использование Jest Mocks с типизацией

```typescript
// Сервис, который будем мокать
interface NotificationService {
  sendEmail(recipient: string, message: string): Promise<boolean>;
  sendSMS(phone: string, message: string): Promise<boolean>;
}

class UserNotificationService {
  constructor(private notificationService: NotificationService) {}

  async notifyUser(user: User, message: string): Promise<boolean> {
    if (user.email) {
      return await this.notificationService.sendEmail(user.email, message);
    }
    return false;
  }
}

// Тест с типизированным mock
describe('UserNotificationService', () => {
  let mockNotificationService: jest.Mocked<NotificationService>;
  let userNotificationService: UserNotificationService;

  beforeEach(() => {
    // Создание типизированного mock
    mockNotificationService = {
      sendEmail: jest.fn(),
      sendSMS: jest.fn()
    };

    userNotificationService = new UserNotificationService(mockNotificationService);
  });

  it('должен отправить email пользователю', async () => {
    const user: User = { id: 1, name: 'John', email: 'john@example.com' };
    const message = 'Test message';
    
    mockNotificationService.sendEmail.mockResolvedValue(true);

    const result: boolean = await userNotificationService.notifyUser(user, message);

    expect(mockNotificationService.sendEmail).toHaveBeenCalledWith('john@example.com', message);
    expect(result).toBe(true);
  });
});
```

### Mock-типы для сложных объектов

```typescript
// Сложный интерфейс
interface PaymentProcessor {
  processPayment(paymentData: PaymentData): Promise<PaymentResult>;
  refundPayment(paymentId: string): Promise<RefundResult>;
  validatePayment(paymentData: PaymentData): Promise<ValidationResult>;
}

interface PaymentData {
  amount: number;
  currency: string;
  cardNumber: string;
  expiryDate: string;
  cvv: string;
}

interface PaymentResult {
  success: boolean;
  transactionId: string;
  message: string;
}

interface RefundResult {
  success: boolean;
  refundId: string;
  message: string;
}

interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

// Тип для mock-объекта
type MockPaymentProcessor = {
  processPayment: jest.MockedFunction<PaymentProcessor['processPayment']>;
  refundPayment: jest.MockedFunction<PaymentProcessor['refundPayment']>;
  validatePayment: jest.MockedFunction<PaymentProcessor['validatePayment']>;
};

// Фабрика для создания mock-объекта
const createPaymentProcessorMock = (): MockPaymentProcessor => ({
  processPayment: jest.fn(),
  refundPayment: jest.fn(),
  validatePayment: jest.fn()
});

// Использование в тесте
describe('OrderService', () => {
  let orderService: OrderService;
  let mockPaymentProcessor: MockPaymentProcessor;

  beforeEach(() => {
    mockPaymentProcessor = createPaymentProcessorMock();
    orderService = new OrderService(mockPaymentProcessor);
  });

  it('должен обработать оплату заказа', async () => {
    const order: Order = { id: 1, total: 100, items: [] };
    const paymentData: PaymentData = {
      amount: 100,
      currency: 'USD',
      cardNumber: '1234567890123456',
      expiryDate: '12/25',
      cvv: '123'
    };
    
    const paymentResult: PaymentResult = {
      success: true,
      transactionId: 'txn_123456',
      message: 'Payment successful'
    };
    
    mockPaymentProcessor.processPayment.mockResolvedValue(paymentResult);

    const result: PaymentResult = await orderService.processOrderPayment(order, paymentData);

    expect(mockPaymentProcessor.processPayment).toHaveBeenCalledWith(paymentData);
    expect(result.success).toBe(true);
  });
});
```

## Лучшие практики

1. **Создавайте mock-типы на основе реальных интерфейсов** - Это обеспечивает совместимость
2. **Используйте типизированные mock-функции** - Это позволяет TypeScript проверять корректность использования
3. **Создавайте фабрики для mock-объектов** - Это упрощает создание mock-объектов в разных тестах
4. **Документируйте поведение mock-объектов** - Это помогает другим разработчикам понять, как использовать mock-объекты

## Связанные темы

- [[Типизация-тестов]] - Типизация в контексте тестирования
- [[Тестирование-типов]] - Проверка корректности определений типов
- [[Инструменты-для-тестирования]] - Инструменты, используемые для тестирования TypeScript кода
- [[Практики-тестирования]] - Общие практики тестирования в TypeScript

## Вывод

Mock-типы являются важным инструментом для тестирования TypeScript-приложений. Они позволяют создавать изолированные тесты с контролируемым поведением зависимостей, что значительно упрощает разработку и поддержку кода.
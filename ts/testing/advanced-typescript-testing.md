# Тестирование TypeScript приложений

Тестирование TypeScript приложений включает в себя unit-тестирование, интеграционное тестирование и end-to-end тестирование с использованием возможностей системы типов для повышения надежности и качества тестов.

## Основы тестирования TypeScript

### 1. Настройка тестовой среды

```json
// package.json
{
  "devDependencies": {
    "@types/jest": "^29.0.0",
    "@types/node": "^18.0.0",
    "jest": "^29.0.0",
    "ts-jest": "^29.0.0",
    "typescript": "^4.8.0",
    "@types/supertest": "^2.0.12",
    "@types/express": "^4.17.14"
  },
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

```json
// jest.config.json
{
  "preset": "ts-jest",
  "testEnvironment": "node",
  "roots": [
    "<rootDir>/src"
  ],
  "testMatch": [
    "**/__tests__/**/*.+(ts|tsx|js)",
    "**/?(*.)+(spec|test).+(ts|tsx|js)"
  ],
  "transform": {
    "^.+\\.(ts|tsx)$": "ts-jest"
  },
  "moduleNameMapper": {
    "^@/(.*)$": "<rootDir>/src/$1"
  },
  "collectCoverageFrom": [
    "src/**/*.{ts,tsx}",
    "!src/**/*.d.ts"
  ]
}
```

### 2. Базовые unit-тесты

```typescript
// src/calculator.ts
export class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
  
  subtract(a: number, b: number): number {
    return a - b;
  }
  
  multiply(a: number, b: number): number {
    return a * b;
  }
  
  divide(a: number, b: number): number {
    if (b === 0) {
      throw new Error("Деление на ноль недопустимо");
    }
    return a / b;
  }
}

// src/calculator.test.ts
import { Calculator } from './calculator';

describe('Calculator', () => {
  let calculator: Calculator;

  beforeEach(() => {
    calculator = new Calculator();
  });

  describe('add', () => {
    it('should add two positive numbers correctly', () => {
      const result = calculator.add(2, 3);
      expect(result).toBe(5);
    });

    it('should add negative numbers correctly', () => {
      const result = calculator.add(-2, -3);
      expect(result).toBe(-5);
    });

    it('should handle zero correctly', () => {
      expect(calculator.add(5, 0)).toBe(5);
      expect(calculator.add(0, 5)).toBe(5);
    });
  });

  describe('divide', () => {
    it('should divide numbers correctly', () => {
      expect(calculator.divide(10, 2)).toBe(5);
    });

    it('should throw error when dividing by zero', () => {
      expect(() => calculator.divide(10, 0)).toThrow("Деление на ноль недопасно");
    });
  });
});
```

## Типобезопасное тестирование

### 1. Использование типов в тестах

```typescript
// src/userService.ts
export interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

export interface UserRepository {
  findById(id: number): Promise<User | null>;
  save(user: User): Promise<User>;
  delete(id: number): Promise<boolean>;
}

export class UserService {
  constructor(private repository: UserRepository) {}

  async getUserProfile(userId: number): Promise<User | null> {
    const user = await this.repository.findById(userId);
    return user;
  }

  async updateUser(userId: number, updates: Partial<User>): Promise<User | null> {
    const existingUser = await this.repository.findById(userId);
    if (!existingUser) {
      return null;
    }

    const updatedUser = { ...existingUser, ...updates };
    return await this.repository.save(updatedUser);
  }
}

// src/userService.test.ts
import { UserService, User, UserRepository } from './userService';

// Типобезопасный mock репозитория
class MockUserRepository implements UserRepository {
  private users: User[] = [
    { id: 1, name: 'John Doe', email: 'john@example.com', isActive: true },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', isActive: false }
  ];

  async findById(id: number): Promise<User | null> {
    const user = this.users.find(u => u.id === id);
    return user || null;
  }

  async save(user: User): Promise<User> {
    const index = this.users.findIndex(u => u.id === user.id);
    if (index !== -1) {
      this.users[index] = user;
    } else {
      this.users.push(user);
    }
    return user;
  }

  async delete(id: number): Promise<boolean> {
    const initialLength = this.users.length;
    this.users = this.users.filter(u => u.id !== id);
    return this.users.length < initialLength;
  }
}

describe('UserService', () => {
  let userService: UserService;
  let mockRepository: MockUserRepository;

  beforeEach(() => {
    mockRepository = new MockUserRepository();
    userService = new UserService(mockRepository);
  });

  describe('getUserProfile', () => {
    it('should return user when user exists', async () => {
      const user = await userService.getUserProfile(1);
      
      expect(user).toEqual({
        id: 1,
        name: 'John Doe',
        email: 'john@example.com',
        isActive: true
      });
    });

    it('should return null when user does not exist', async () => {
      const user = await userService.getUserProfile(999);
      expect(user).toBeNull();
    });
  });

  describe('updateUser', () => {
    it('should update user successfully', async () => {
      const updatedUser = await userService.updateUser(1, { name: 'John Updated' });
      
      expect(updatedUser).toEqual({
        id: 1,
        name: 'John Updated',
        email: 'john@example.com',
        isActive: true
      });
    });

    it('should return null when user does not exist', async () => {
      const result = await userService.updateUser(999, { name: 'Non-existent' });
      expect(result).toBeNull();
    });
  });
});
```

### 2. Тестирование с Generics

```typescript
// src/genericRepository.ts
export interface Entity {
  id: number;
}

export class GenericRepository<T extends Entity> {
  private entities: T[] = [];
  private nextId: number = 1;

  async findById(id: number): Promise<T | null> {
    const entity = this.entities.find(e => e.id === id);
    return entity || null;
  }

  async save(entity: T): Promise<T> {
    if (entity.id === 0) {
      // Новый элемент
      entity.id = this.nextId++;
      this.entities.push(entity);
    } else {
      // Обновление существующего
      const index = this.entities.findIndex(e => e.id === entity.id);
      if (index !== -1) {
        this.entities[index] = entity;
      }
    }
    return entity;
  }

  async findAll(): Promise<T[]> {
    return [...this.entities];
  }
}

// src/user.ts (реализует Entity)
export interface User extends Entity {
  name: string;
  email: string;
}

// src/genericRepository.test.ts
import { GenericRepository } from './genericRepository';
import { User } from './user';

describe('GenericRepository', () => {
  let userRepository: GenericRepository<User>;

  beforeEach(() => {
    userRepository = new GenericRepository<User>();
  });

  it('should create and retrieve user with correct type', async () => {
    const newUser: User = { id: 0, name: 'Test User', email: 'test@example.com' };
    const savedUser = await userRepository.save(newUser);
    
    expect(savedUser).toHaveProperty('id');
    expect(savedUser).toHaveProperty('name');
    expect(savedUser).toHaveProperty('email');
    expect(savedUser.id).toBeGreaterThan(0); // ID был назначен
    
    const retrievedUser = await userRepository.findById(savedUser.id);
    expect(retrievedUser).toEqual(savedUser);
  });

  it('should handle multiple entities of the same type', async () => {
    const users: User[] = [
      { id: 0, name: 'User 1', email: 'user1@example.com' },
      { id: 0, name: 'User 2', email: 'user2@example.com' }
    ];

    const savedUsers = await Promise.all(users.map(user => userRepository.save(user)));
    const allUsers = await userRepository.findAll();

    expect(allUsers).toHaveLength(2);
    expect(allUsers).toEqual(savedUsers);
  });
});
```

## Продвинутые паттерны тестирования

### 1. Паттерн "Test Data Builder"

```typescript
// src/testDataBuilder.ts
export class UserBuilder {
  private data: Partial<User> = {
    id: 0,
    name: 'Default Name',
    email: 'default@example.com'
  };

  withId(id: number): UserBuilder {
    this.data.id = id;
    return this;
  }

  withName(name: string): UserBuilder {
    this.data.name = name;
    return this;
  }

  withEmail(email: string): UserBuilder {
    this.data.email = email;
    return this;
  }

  withActiveStatus(isActive: boolean): UserBuilder {
    this.data.isActive = isActive;
    return this;
  }

  build(): User {
    if (!this.data.name || !this.data.email) {
      throw new Error('Name and email are required');
    }
    
    return {
      id: this.data.id || 0,
      name: this.data.name,
      email: this.data.email,
      isActive: this.data.isActive ?? true
    } as User;
  }

  buildMany(count: number): User[] {
    return Array.from({ length: count }, (_, i) => 
      new UserBuilder()
        .withId(i + 1)
        .withName(`User ${i + 1}`)
        .withEmail(`user${i + 1}@example.com`)
        .build()
    );
  }
}

// Использование в тестах
describe('UserService with builders', () => {
  it('should handle multiple users correctly', async () => {
    const users = new UserBuilder().buildMany(3);
    const mockRepository = new MockUserRepository();
    const userService = new UserService(mockRepository);

    // Сохраняем пользователей
    for (const user of users) {
      await mockRepository.save(user);
    }

    // Проверяем, что все пользователи доступны
    for (const user of users) {
      const retrieved = await userService.getUserProfile(user.id);
      expect(retrieved).toEqual(user);
    }
  });
});
```

### 2. Паттерн "Page Object" для UI тестов

```typescript
// src/pageObjects/userListPage.ts (для интеграционных тестов)
export class UserListPage {
  constructor(private document: Document) {}

  getUserRows(): NodeListOf<Element> {
    return this.document.querySelectorAll('[data-testid="user-row"]');
  }

  getUserNames(): string[] {
    const rows = this.getUserRows();
    return Array.from(rows).map(row => 
      row.querySelector('[data-testid="user-name"]')?.textContent || ''
    );
  }

  findUserRow(name: string): Element | null {
    const rows = this.getUserRows();
    return Array.from(rows).find(row => 
      row.querySelector('[data-testid="user-name"]')?.textContent === name
    ) || null;
  }

  clickEditButton(userName: string): void {
    const row = this.findUserRow(userName);
    if (row) {
      const editButton = row.querySelector('[data-testid="edit-button"]') as HTMLButtonElement;
      if (editButton) {
        editButton.click();
      }
    }
  }
}

// src/pageObjects/userListPage.test.ts
describe('UserListPage', () => {
  let mockDocument: Document;
  let page: UserListPage;

  beforeEach(() => {
    // Создаем mock DOM
    mockDocument = {
      querySelectorAll: jest.fn(),
      querySelector: jest.fn()
    } as unknown as Document;
    
    page = new UserListPage(mockDocument);
  });

  it('should find user by name', () => {
    // Мокаем возвращаемые значения
    const mockRow = {
      querySelector: (selector: string) => {
        if (selector === '[data-testid="user-name"]') {
          return { textContent: 'John Doe' };
        }
        return null;
      }
    } as Element;

    (mockDocument.querySelectorAll as jest.Mock).mockReturnValue([mockRow]);

    const foundRow = page.findUserRow('John Doe');
    expect(foundRow).toBe(mockRow);
  });
});
```

### 3. Тестирование асинхронного кода

```typescript
// src/asyncService.ts
export class AsyncService {
  async fetchData(id: number): Promise<string> {
    // Симуляция асинхронной операции
    return new Promise(resolve => {
      setTimeout(() => resolve(`Data for ${id}`), 100);
    });
  }

  async fetchMultipleData(ids: number[]): Promise<string[]> {
    const promises = ids.map(id => this.fetchData(id));
    return Promise.all(promises);
  }

  async fetchWithRetry(id: number, maxRetries: number = 3): Promise<string> {
    let lastError: Error;

    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        return await this.fetchData(id);
      } catch (error) {
        lastError = error as Error;
        if (attempt === maxRetries) {
          throw lastError;
        }
        // Задержка перед повторной попыткой
        await new Promise(resolve => setTimeout(resolve, 100 * (attempt + 1)));
      }
    }

    throw lastError!;
  }
}

// src/asyncService.test.ts
import { AsyncService } from './asyncService';

describe('AsyncService', () => {
  let asyncService: AsyncService;

  beforeEach(() => {
    asyncService = new AsyncService();
  });

  it('should fetch data asynchronously', async () => {
    const result = await asyncService.fetchData(1);
    expect(result).toBe('Data for 1');
  });

  it('should fetch multiple data concurrently', async () => {
    const result = await asyncService.fetchMultipleData([1, 2, 3]);
    expect(result).toEqual(['Data for 1', 'Data for 2', 'Data for 3']);
  });

  it('should handle async errors with retries', async () => {
    // Мокаем метод для тестирования повторных попыток
    const originalFetchData = asyncService.fetchData;
    let callCount = 0;
    
    jest.spyOn(asyncService, 'fetchData').mockImplementation(async (id: number) => {
      callCount++;
      if (callCount < 3) {
        throw new Error('Network error');
      }
      return originalFetchData.call(asyncService, id);
    });

    const result = await asyncService.fetchWithRetry(1, 3);
    expect(result).toBe('Data for 1');
    expect(callCount).toBe(3);
  });

  it('should timeout if retries are exhausted', async () => {
    jest.spyOn(asyncService, 'fetchData').mockImplementation(async () => {
      throw new Error('Network error');
    });

    await expect(asyncService.fetchWithRetry(1, 2)).rejects.toThrow('Network error');
  });
});
```

## Тестирование с моками и стабами

### 1. Использование Jest моков

```typescript
// src/notificationService.ts
export interface NotificationSender {
  send(recipient: string, message: string): Promise<boolean>;
}

export class EmailNotificationSender implements NotificationSender {
  async send(recipient: string, message: string): Promise<boolean> {
    // Реальная отправка email
    console.log(`Sending email to ${recipient}: ${message}`);
    return true;
  }
}

export class SMSNotificationSender implements NotificationSender {
  async send(recipient: string, message: string): Promise<boolean> {
    // Реальная отправка SMS
    console.log(`Sending SMS to ${recipient}: ${message}`);
    return true;
  }
}

export class NotificationService {
  constructor(private sender: NotificationSender) {}

  async sendNotification(recipient: string, message: string): Promise<boolean> {
    return await this.sender.send(recipient, message);
  }
}

// src/notificationService.test.ts
import { 
  NotificationService, 
  NotificationSender, 
  EmailNotificationSender 
} from './notificationService';

describe('NotificationService with Jest mocks', () => {
  let mockSender: jest.Mocked<NotificationSender>;
  let notificationService: NotificationService;

  beforeEach(() => {
    // Создаем типобезопасный мок
    mockSender = {
      send: jest.fn()
    } as jest.Mocked<NotificationSender>;

    notificationService = new NotificationService(mockSender);
  });

  it('should send notification using the sender', async () => {
    mockSender.send.mockResolvedValue(true);

    const result = await notificationService.sendNotification('test@example.com', 'Hello');

    expect(mockSender.send).toHaveBeenCalledWith('test@example.com', 'Hello');
    expect(result).toBe(true);
  });

  it('should handle sender errors', async () => {
    mockSender.send.mockRejectedValue(new Error('Send failed'));

    await expect(
      notificationService.sendNotification('test@example.com', 'Hello')
    ).rejects.toThrow('Send failed');
  });

  it('should work with real implementation', async () => {
    const realSender = new EmailNotificationSender();
    const realService = new NotificationService(realSender);

    // Тестируем с реальной реализацией
    const consoleSpy = jest.spyOn(console, 'log').mockImplementation();

    const result = await realService.sendNotification('test@example.com', 'Hello');
    
    expect(result).toBe(true);
    expect(consoleSpy).toHaveBeenCalledWith(
      'Sending email to test@example.com: Hello'
    );

    consoleSpy.mockRestore();
  });
});
```

### 2. Создание моков вручную

```typescript
// src/apiClient.ts
export interface ApiClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
  put<T>(url: string, data: any): Promise<T>;
  delete(url: string): Promise<void>;
}

export class HttpClient implements ApiClient {
  async get<T>(url: string): Promise<T> {
    const response = await fetch(url);
    return response.json();
  }

  async post<T>(url: string, data: any): Promise<T> {
    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }

  async put<T>(url: string, data: any): Promise<T> {
    const response = await fetch(url, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
    return response.json();
  }

  async delete(url: string): Promise<void> {
    await fetch(url, { method: 'DELETE' });
  }
}

// src/apiClient.test.ts
import { ApiClient } from './apiClient';

// Ручной мок API клиента
class MockApiClient implements ApiClient {
  private responses: Map<string, any> = new Map();
  private requestLog: Array<{ method: string; url: string; data?: any }> = [];

  setResponse(url: string, response: any): void {
    this.responses.set(url, response);
  }

  async get<T>(url: string): Promise<T> {
    this.requestLog.push({ method: 'GET', url });
    const response = this.responses.get(url);
    if (response === undefined) {
      throw new Error(`No mock response for GET ${url}`);
    }
    return response as T;
  }

  async post<T>(url: string, data: any): Promise<T> {
    this.requestLog.push({ method: 'POST', url, data });
    const response = this.responses.get(url);
    if (response === undefined) {
      throw new Error(`No mock response for POST ${url}`);
    }
    return response as T;
  }

  async put<T>(url: string, data: any): Promise<T> {
    this.requestLog.push({ method: 'PUT', url, data });
    const response = this.responses.get(url);
    if (response === undefined) {
      throw new Error(`No mock response for PUT ${url}`);
    }
    return response as T;
  }

  async delete(url: string): Promise<void> {
    this.requestLog.push({ method: 'DELETE', url });
  }

  getRequestLog(): Array<{ method: string; url: string; data?: any }> {
    return [...this.requestLog];
  }

  clear(): void {
    this.responses.clear();
    this.requestLog = [];
  }
}

describe('API Client with manual mocks', () => {
  let mockClient: MockApiClient;

  beforeEach(() => {
    mockClient = new MockApiClient();
  });

  it('should make correct API calls', async () => {
    mockClient.setResponse('/api/users/1', { id: 1, name: 'John' });

    const user = await mockClient.get<{ id: number; name: string }>('/api/users/1');

    expect(user).toEqual({ id: 1, name: 'John' });
    expect(mockClient.getRequestLog()).toEqual([
      { method: 'GET', url: '/api/users/1' }
    ]);
  });

  it('should handle POST requests correctly', async () => {
    mockClient.setResponse('/api/users', { id: 2, name: 'Jane' });

    const newUser = await mockClient.post<{ id: number; name: string }>(
      '/api/users',
      { name: 'Jane' }
    );

    expect(newUser).toEqual({ id: 2, name: 'Jane' });
    expect(mockClient.getRequestLog()).toEqual([
      { method: 'POST', url: '/api/users', data: { name: 'Jane' } }
    ]);
  });
});
```

## Тестирование ошибок и исключений

### 1. Тестирование обработки ошибок

```typescript
// src/errorService.ts
export class ErrorService {
  async riskyOperation(data: any): Promise<any> {
    if (!data) {
      throw new Error('Data is required');
    }

    if (typeof data !== 'object') {
      throw new TypeError('Data must be an object');
    }

    if (data.errorFlag) {
      throw new Error('Simulated error');
    }

    return { success: true, data };
  }

  validateInput(input: string): string {
    if (!input) {
      throw new Error('Input is required');
    }

    if (input.length < 3) {
      throw new Error('Input too short');
    }

    return input.toUpperCase();
  }
}

// src/errorService.test.ts
import { ErrorService } from './errorService';

describe('ErrorService', () => {
  let errorService: ErrorService;

  beforeEach(() => {
    errorService = new ErrorService();
  });

  describe('riskyOperation', () => {
    it('should throw error for null data', async () => {
      await expect(errorService.riskyOperation(null))
        .rejects
        .toThrow('Data is required');
    });

    it('should throw TypeError for non-object data', async () => {
      await expect(errorService.riskyOperation('not-an-object'))
        .rejects
        .toThrow(TypeError);
    });

    it('should throw specific error when errorFlag is true', async () => {
      await expect(errorService.riskyOperation({ errorFlag: true }))
        .rejects
        .toThrow('Simulated error');
    });

    it('should return success for valid data', async () => {
      const result = await errorService.riskyOperation({ valid: true });
      expect(result).toEqual({ 
        success: true, 
        data: { valid: true } 
      });
    });
  });

  describe('validateInput', () => {
    it('should throw error for empty input', () => {
      expect(() => errorService.validateInput(''))
        .toThrow('Input is required');
    });

    it('should throw error for short input', () => {
      expect(() => errorService.validateInput('ab'))
        .toThrow('Input too short');
    });

    it('should return uppercase for valid input', () => {
      const result = errorService.validateInput('hello');
      expect(result).toBe('HELLO');
    });
  });

  // Тестирование конкретных типов ошибок
  it('should throw specific error types', () => {
    expect(() => errorService.validateInput(''))
      .toThrowError(Error);

    expect(() => errorService.validateInput('ab'))
      .toThrowError(Error);
  });
});
```

### 2. Тестирование асинхронных ошибок

```typescript
// src/asyncErrorService.ts
export class AsyncErrorService {
  async fetchWithError(url: string): Promise<any> {
    if (url === 'error') {
      throw new Error('Network error');
    }
    
    return { data: 'success' };
  }

  async retryOperation(maxRetries: number, shouldFailUntil: number = 2): Promise<string> {
    let attempt = 0;
    
    while (attempt < maxRetries) {
      attempt++;
      
      if (attempt <= shouldFailUntil) {
        throw new Error(`Attempt ${attempt} failed`);
      }
      
      return `Success on attempt ${attempt}`;
    }
    
    throw new Error(`All ${maxRetries} attempts failed`);
  }
}

// src/asyncErrorService.test.ts
import { AsyncErrorService } from './asyncErrorService';

describe('AsyncErrorService', () => {
  let asyncErrorService: AsyncErrorService;

  beforeEach(() => {
    asyncErrorService = new AsyncErrorService();
  });

  it('should handle network errors', async () => {
    await expect(asyncErrorService.fetchWithError('error'))
      .rejects
      .toThrow('Network error');
  });

  it('should handle successful requests', async () => {
    const result = await asyncErrorService.fetchWithError('success');
    expect(result).toEqual({ data: 'success' });
  });

  it('should retry and eventually succeed', async () => {
    // Мокаем текущее время для предсказуемого тестирования
    const originalSetTimeout = setTimeout;
    const setTimeoutMock = (fn: Function, time: number) => originalSetTimeout(fn, 1); // Ускоряем для теста
    
    jest
      .spyOn(global, 'setTimeout')
      .mockImplementation(setTimeoutMock as any);

    const result = await asyncErrorService.retryOperation(5, 2);
    expect(result).toBe('Success on attempt 3');

    // Восстанавливаем оригинальный setTimeout
    jest.clearAllMocks();
  });

  it('should fail after max retries', async () => {
    await expect(asyncErrorService.retryOperation(2, 5))
      .rejects
      .toThrow('All 2 attempts failed');
  });

  // Тестирование с использованием done callback для лучшей обработки ошибок
  it('should handle errors with done callback', (done) => {
    asyncErrorService.fetchWithError('error')
      .then(() => {
        done(new Error('Expected promise to be rejected'));
      })
      .catch(error => {
        expect(error.message).toBe('Network error');
        done();
      });
  });
});
```

## Интеграционное тестирование

### 1. Тестирование взаимодействия компонентов

```typescript
// src/userModule.ts (демонстрация модуля)
import { UserRepository, UserService } from './userService';
import { NotificationService, NotificationSender } from './notificationService';

export interface UserModuleDependencies {
  userRepository: UserRepository;
  notificationSender: NotificationSender;
}

export class UserModule {
  private userService: UserService;
  private notificationService: NotificationService;

  constructor(private deps: UserModuleDependencies) {
    this.userService = new UserService(deps.userRepository);
    this.notificationService = new NotificationService(deps.notificationSender);
  }

  async registerUser(userData: Omit<User, 'id'>): Promise<User> {
    // Создаем пользователя
    const newUser: User = { id: Date.now(), ...userData } as User;
    const savedUser = await this.deps.userRepository.save(newUser);

    // Отправляем уведомление
    await this.notificationService.sendNotification(
      savedUser.email,
      `Welcome, ${savedUser.name}!`
    );

    return savedUser;
  }

  async deactivateUser(userId: number): Promise<boolean> {
    const user = await this.userService.getUserProfile(userId);
    if (!user) {
      return false;
    }

    const updatedUser = await this.userService.updateUser(userId, { isActive: false });
    
    if (updatedUser) {
      await this.notificationService.sendNotification(
        updatedUser.email,
        'Your account has been deactivated'
      );
    }

    return !!updatedUser;
  }
}

// src/userModule.test.ts
import { UserModule, UserModuleDependencies } from './userModule';
import { User } from './user';
import { MockUserRepository } from './userService.test'; // используем ранее созданный мок
import { NotificationSender } from './notificationService';

class MockNotificationSender implements NotificationSender {
  sentNotifications: Array<{ recipient: string; message: string }> = [];

  async send(recipient: string, message: string): Promise<boolean> {
    this.sentNotifications.push({ recipient, message });
    return true;
  }

  clear(): void {
    this.sentNotifications = [];
  }
}

describe('UserModule Integration', () => {
  let mockUserRepository: MockUserRepository;
  let mockNotificationSender: MockNotificationSender;
  let userModule: UserModule;

  beforeEach(() => {
    mockUserRepository = new MockUserRepository();
    mockNotificationSender = new MockNotificationSender();
    
    const deps: UserModuleDependencies = {
      userRepository: mockUserRepository,
      notificationSender: mockNotificationSender
    };
    
    userModule = new UserModule(deps);
  });

  it('should register user and send welcome notification', async () => {
    const userData = { name: 'John Doe', email: 'john@example.com', isActive: true };
    const result = await userModule.registerUser(userData);

    // Проверяем, что пользователь был сохранен
    expect(result.name).toBe('John Doe');
    expect(result.email).toBe('john@example.com');
    expect(result.isActive).toBe(true);

    // Проверяем, что уведомление было отправлено
    expect(mockNotificationSender.sentNotifications).toHaveLength(1);
    expect(mockNotificationSender.sentNotifications[0]).toEqual({
      recipient: 'john@example.com',
      message: 'Welcome, John Doe!'
    });
  });

  it('should deactivate user and send notification', async () => {
    // Сначала регистрируем пользователя
    const userData = { name: 'Jane Doe', email: 'jane@example.com', isActive: true };
    const user = await userModule.registerUser(userData);

    // Затем деактивируем
    const result = await userModule.deactivateUser(user.id);

    expect(result).toBe(true);

    // Проверяем, что пользователь деактивирован
    const updatedUser = await mockUserRepository.findById(user.id);
    expect(updatedUser?.isActive).toBe(false);

    // Проверяем, что отправлены оба уведомления
    expect(mockNotificationSender.sentNotifications).toHaveLength(2);
    expect(mockNotificationSender.sentNotifications[1]).toEqual({
      recipient: 'jane@example.com',
      message: 'Your account has been deactivated'
    });
  });

  it('should return false when deactivating non-existent user', async () => {
    const result = await userModule.deactivateUser(999);
    expect(result).toBe(false);
    expect(mockNotificationSender.sentNotifications).toHaveLength(0);
  });
});
```

## Тестирование с использованием fixtures

### 1. Создание тестовых данных

```typescript
// src/testFixtures.ts
export const USER_FIXTURES = {
  validUser: {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    isActive: true
  },
  inactiveUser: {
    id: 2,
    name: 'Jane Smith',
    email: 'jane@example.com',
    isActive: false
  },
  newUser: {
    name: 'New User',
    email: 'newuser@example.com',
    isActive: true
  }
};

export const API_RESPONSE_FIXTURES = {
  userResponse: {
    status: 200,
    data: USER_FIXTURES.validUser,
    message: 'User found'
  },
  usersListResponse: {
    status: 200,
    data: [USER_FIXTURES.validUser, USER_FIXTURES.inactiveUser],
    message: 'Users found'
  },
  errorResponse: {
    status: 404,
    message: 'User not found'
  }
};

export const VALIDATION_FIXTURES = {
  validInputs: [
    'valid@example.com',
    'test.user@domain.co.uk',
    'user+tag@example.org'
  ],
  invalidInputs: [
    'invalid-email',
    'missing@.com',
    '@missing-domain.com',
    'space in@domain.com'
  ]
};

// src/testHelpers.ts
import { USER_FIXTURES } from './testFixtures';

export function createTestUser(overrides: Partial<User> = {}): User {
  return {
    ...USER_FIXTURES.validUser,
    ...overrides,
    id: overrides.id || Date.now()
  };
}

export function createMultipleTestUsers(count: number): User[] {
  return Array.from({ length: count }, (_, i) => 
    createTestUser({ id: i + 1, name: `User ${i + 1}` })
  );
}

export function expectUserToMatch(user: User, expected: Partial<User>): void {
  Object.entries(expected).forEach(([key, value]) => {
    expect((user as any)[key]).toEqual(value);
  });
}
```

### 2. Использование fixtures в тестах

```typescript
// src/advancedTesting.test.ts
import { 
  USER_FIXTURES, 
  API_RESPONSE_FIXTURES, 
  VALIDATION_FIXTURES 
} from './testFixtures';
import { 
  createTestUser, 
  createMultipleTestUsers, 
  expectUserToMatch 
} from './testHelpers';
import { UserService, User } from './userService';

describe('Advanced Testing with Fixtures', () => {
  let userService: UserService;
  let mockRepository: MockUserRepository;

  beforeEach(() => {
    mockRepository = new MockUserRepository();
    userService = new UserService(mockRepository);
  });

  describe('User creation with fixtures', () => {
    it('should create user from fixture', async () => {
      const newUser = createTestUser();
      mockRepository.users.push(newUser);

      const retrievedUser = await userService.getUserProfile(newUser.id);
      expect(retrievedUser).toEqual(newUser);
    });

    it('should handle multiple users from fixtures', async () => {
      const users = createMultipleTestUsers(5);
      mockRepository.users.push(...users);

      const allUsers = await mockRepository.findAll();
      expect(allUsers).toHaveLength(5);
      
      allUsers.forEach((user, index) => {
        expectUserToMatch(user, { 
          id: index + 1, 
          name: `User ${index + 1}` 
        });
      });
    });
  });

  describe('Validation testing with fixtures', () => {
    it('should validate valid email formats', () => {
      VALIDATION_FIXTURES.validInputs.forEach(email => {
        expect(email).toMatch(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
      });
    });

    it('should reject invalid email formats', () => {
      VALIDATION_FIXTURES.invalidInputs.forEach(email => {
        expect(email).not.toMatch(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
      });
    });
  });

  describe('API response testing', () => {
    it('should handle success response', () => {
      const response = API_RESPONSE_FIXTURES.userResponse;
      expect(response.status).toBe(200);
      expect(response.data).toBeDefined();
      expect(response.message).toBe('User found');
    });

    it('should handle error response', () => {
      const response = API_RESPONSE_FIXTURES.errorResponse;
      expect(response.status).toBe(404);
      expect(response.message).toBe('User not found');
    });
  });

  describe('Complex scenario testing', () => {
    it('should handle user lifecycle with fixtures', async () => {
      // Создаем пользователя
      const user = createTestUser(USER_FIXTURES.newUser);
      
      // Сохраняем через репозиторий (имитация работы сервиса)
      mockRepository.users.push(user);
      
      // Получаем пользователя
      const retrieved = await userService.getUserProfile(user.id);
      expect(retrieved).toEqual(user);
      
      // Обновляем пользователя
      const updated = await userService.updateUser(user.id, { name: 'Updated Name' });
      expect(updated?.name).toBe('Updated Name');
      
      // Проверяем, что пользователь был обновлен в репозитории
      const updatedFromRepo = await mockRepository.findById(user.id);
      expect(updatedFromRepo?.name).toBe('Updated Name');
    });
  });
});
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки в тестировании

```typescript
// ОШИБКА: Тесты с побочными эффектами
// let globalCounter = 0;
// 
// test('bad test', () => {
//   globalCounter++; // Побочный эффект
//   expect(globalCounter).toBe(1);
// });

// ЛУЧШЕ: Изолированные тесты
test('good test', () => {
  const counter = 0;
  expect(counter).toBe(0);
});

// ОШИБКА: Зависимость от порядка выполнения
// test('first', () => {
//   globalValue = 10;
// });
// 
// test('second', () => {
//   expect(globalValue).toBe(10); // Зависит от первого теста
// });

// ЛУЧШЕ: Независимые тесты
test('test with setup', () => {
  const value = 10;
  expect(value).toBe(10);
});

// ОШИБКА: Слишком общие тесты
// test('should work', () => {
//   // Проверяет слишком много всего
// });

// ЛУЧШЕ: Конкретные, целенаправленные тесты
test('should add two numbers correctly', () => {
  const result = 2 + 3;
  expect(result).toBe(5);
});
```

### 2. Лучшие практики тестирования

```typescript
// 1. Используйте описательные названия тестов
test('should return user profile when user exists', async () => {
  // тестовая логика
});

// 2. Тестируйте один сценарий за раз
test('should handle division by zero error', () => {
  // Только проверка деления на ноль
});

// 3. Используйте beforeEach для настройки тестов
describe('UserService', () => {
  let userService: UserService;
  let mockRepository: MockUserRepository;

  beforeEach(() => {
    mockRepository = new MockUserRepository();
    userService = new UserService(mockRepository);
  });

  test('should find existing user', async () => {
    // тест использует подготовленные переменные
  });
});

// 4. Используйте fixtures для сложных данных
const complexUserData = {
  id: 1,
  profile: {
    personal: { name: 'John', age: 30 },
    contact: { email: 'john@example.com', phone: '123-456-7890' },
    address: { street: '123 Main St', city: 'Anytown' }
  }
};

test('should handle complex user data', () => {
  // используем fixture
});
```

## Лучшие практики

1. **Используйте типобезопасные моки** - для предотвращения ошибок при изменении API
2. **Создавайте переиспользуемые fixtures** - для упрощения написания тестов
3. **Тестируйте поведение, а не реализацию** - фокус на публичном API
4. **Изолируйте тесты** - каждый тест должен быть независимым
5. **Используйте описательные названия** - для понимания цели теста
6. **Тестируйте граничные условия** - нули, пустые значения, максимальные/минимальные значения
7. **Покрывайте ошибочные сценарии** - для повышения надежности

## Связи с другими концепциями

- [[ts/type-system/type-inference|Вывод типов]] - как типизация помогает в тестировании
- [[ts/utility-types/index|Утилиты типов]] - для создания вспомогательных типов в тестах
- [[testing/unit-testing]] - основы unit тестирования
- [[testing/integration-testing]] - интеграционное тестирование
- [[react/testing]] - тестирование React компонентов
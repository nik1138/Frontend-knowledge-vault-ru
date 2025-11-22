# Тестирование в TypeScript

## Введение в тестирование TypeScript-кода

Тестирование в TypeScript включает в себя различные уровни проверки кода, от модульного тестирования отдельных функций до интеграционного тестирования взаимодействия между компонентами. TypeScript предоставляет дополнительные возможности для тестирования благодаря своей системе типов.

## Виды тестирования

### Модульное тестирование (Unit Testing)

Модульное тестирование проверяет отдельные компоненты системы изолированно.

```ts
// calculator.ts
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
      throw new Error("Division by zero");
    }
    return a / b;
  }
}
```

```ts
// calculator.test.ts
import { Calculator } from './calculator';

describe('Calculator', () => {
  let calculator: Calculator;
  
  beforeEach(() => {
    calculator = new Calculator();
  });
  
  test('should add two numbers correctly', () => {
    expect(calculator.add(2, 3)).toBe(5);
    expect(calculator.add(-1, 1)).toBe(0);
  });
  
  test('should subtract two numbers correctly', () => {
    expect(calculator.subtract(5, 3)).toBe(2);
    expect(calculator.subtract(0, 5)).toBe(-5);
  });
  
  test('should multiply two numbers correctly', () => {
    expect(calculator.multiply(3, 4)).toBe(12);
    expect(calculator.multiply(-2, 3)).toBe(-6);
  });
  
  test('should divide two numbers correctly', () => {
    expect(calculator.divide(10, 2)).toBe(5);
    expect(calculator.divide(7, 1)).toBe(7);
  });
  
  test('should throw error when dividing by zero', () => {
    expect(() => calculator.divide(10, 0)).toThrow("Division by zero");
  });
});
```

## Типобезопасное тестирование

### Использование типов в тестах

```ts
// user-service.ts
export interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

export class UserService {
  private users: User[] = [];
  
  addUser(user: Omit<User, 'id'>): User {
    const newUser: User = {
      ...user,
      id: this.users.length + 1,
      isActive: user.isActive ?? true
    };
    this.users.push(newUser);
    return newUser;
  }
  
  getUserById(id: number): User | undefined {
    return this.users.find(user => user.id === id);
  }
  
  getAllUsers(): readonly User[] {
    return [...this.users]; // возвращаем копию массива для защиты
  }
}
```

```ts
// user-service.test.ts
import { UserService, User } from './user-service';

describe('UserService', () => {
  let userService: UserService;
  
  beforeEach(() => {
    userService = new UserService();
  });
  
  test('should add a new user with correct properties', () => {
    const userData = {
      name: 'John Doe',
      email: 'john@example.com',
      isActive: true
    };
    
    const newUser = userService.addUser(userData);
    
    // Проверяем, что возвращаемый объект имеет правильный тип
    expect(newUser).toMatchObject({
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      isActive: true
    });
    
    // Проверяем, что тип возвращаемого значения корректен
    const typedUser: User = newUser;
    expect(typedUser.id).toBe(1);
  });
  
  test('should retrieve user by id', () => {
    const user = userService.addUser({
      name: 'Jane Doe',
      email: 'jane@example.com'
    });
    
    const retrievedUser = userService.getUserById(user.id);
    
    expect(retrievedUser).toEqual(user);
  });
  
  test('should return undefined for non-existent user', () => {
    const user = userService.getUserById(999);
    expect(user).toBeUndefined();
  });
});
```

## Тестирование асинхронного кода

```ts
// api-client.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
}

export class ApiClient {
  async get<T>(url: string): Promise<ApiResponse<T>> {
    const response = await fetch(url);
    const data = await response.json();
    return { data, status: response.status };
  }
  
  async post<T>(url: string, body: any): Promise<ApiResponse<T>> {
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(body)
    });
    const data = await response.json();
    return { data, status: response.status };
  }
}
```

```ts
// api-client.test.ts
import { ApiClient } from './api-client';

// Мокаем fetch
global.fetch = jest.fn() as jest.MockedFunction<typeof fetch>;

describe('ApiClient', () => {
  let apiClient: ApiClient;
  
  beforeEach(() => {
    apiClient = new ApiClient();
    (global.fetch as jest.MockedFunction<typeof fetch>).mockClear();
  });
  
  test('should make GET request and return typed response', async () => {
    const mockData = { id: 1, name: 'Test' };
    (global.fetch as jest.MockedFunction<typeof fetch>)
      .mockResolvedValueOnce({
        json: () => Promise.resolve(mockData),
        status: 200
      } as Response);
    
    const response = await apiClient.get('/test');
    
    expect(response).toEqual({
      data: mockData,
      status: 200
    });
    
    // Проверяем, что тип данных корректен
    const typedData: { id: number; name: string } = response.data;
    expect(typedData.id).toBe(1);
  });
  
  test('should make POST request with correct headers and body', async () => {
    const postData = { name: 'New Item' };
    const expectedResponse = { id: 1, ...postData };
    
    (global.fetch as jest.MockedFunction<typeof fetch>)
      .mockResolvedValueOnce({
        json: () => Promise.resolve(expectedResponse),
        status: 201
      } as Response);
    
    const response = await apiClient.post('/items', postData);
    
    expect(global.fetch).toHaveBeenCalledWith('/items', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(postData)
    });
    
    expect(response.data).toEqual(expectedResponse);
  });
});
```

## Мокирование зависимостей

```ts
// user-repository.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export interface UserRepository {
  findById(id: number): Promise<User | null>;
  save(user: User): Promise<User>;
  findAll(): Promise<User[]>;
}

export class DatabaseUserRepository implements UserRepository {
  async findById(id: number): Promise<User | null> {
    // Реализация получения пользователя из базы данных
    return null; // Заглушка
  }
  
  async save(user: User): Promise<User> {
    // Реализация сохранения пользователя в базу данных
    return user; // Заглушка
  }
  
  async findAll(): Promise<User[]> {
    // Реализация получения всех пользователей
    return []; // Заглушка
  }
}
```

```ts
// user-service-with-repository.ts
import { UserRepository, User } from './user-repository';

export class UserService {
  constructor(private userRepository: UserRepository) {}
  
  async getUserProfile(userId: number): Promise<User | null> {
    return await this.userRepository.findById(userId);
  }
  
  async createUser(userData: Omit<User, 'id'>): Promise<User> {
    const newUser: User = {
      ...userData,
      id: Date.now() // В реальности id генерируется базой данных
    };
    
    return await this.userRepository.save(newUser);
  }
}
```

```ts
// user-service-with-repository.test.ts
import { UserService } from './user-service-with-repository';
import { UserRepository, User } from './user-repository';

// Создаем мок-реализацию репозитория
const mockUserRepository: jest.Mocked<UserRepository> = {
  findById: jest.fn(),
  save: jest.fn(),
  findAll: jest.fn()
};

describe('UserService with mocked repository', () => {
  let userService: UserService;
  const mockUser: User = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com'
  };
  
  beforeEach(() => {
    userService = new UserService(mockUserRepository);
    jest.clearAllMocks();
  });
  
  test('should retrieve user profile', async () => {
    mockUserRepository.findById.mockResolvedValue(mockUser);
    
    const result = await userService.getUserProfile(1);
    
    expect(mockUserRepository.findById).toHaveBeenCalledWith(1);
    expect(result).toEqual(mockUser);
  });
  
  test('should create new user', async () => {
    const userData = {
      name: 'Jane Doe',
      email: 'jane@example.com'
    };
    
    mockUserRepository.save.mockResolvedValue(mockUser);
    
    const result = await userService.createUser(userData);
    
    expect(mockUserRepository.save).toHaveBeenCalledWith({
      ...userData,
      id: expect.any(Number)
    });
    expect(result).toEqual(mockUser);
  });
});
```

## Тестирование с типизированными фикстурами

```ts
// test-fixtures.ts
export const createUser = (overrides: Partial<User> = {}): User => ({
  id: 1,
  name: 'Test User',
  email: 'test@example.com',
  ...overrides
});

export const createUsers = (count: number): User[] => {
  return Array.from({ length: count }, (_, index) => 
    createUser({ id: index + 1, name: `User ${index + 1}` })
  );
};
```

```ts
// user-service-extended.test.ts
import { UserService } from './user-service-with-repository';
import { UserRepository } from './user-repository';
import { createUser, createUsers } from './test-fixtures';

const mockUserRepository: jest.Mocked<UserRepository> = {
  findById: jest.fn(),
  save: jest.fn(),
  findAll: jest.fn()
};

describe('UserService with fixtures', () => {
  let userService: UserService;
  
  beforeEach(() => {
    userService = new UserService(mockUserRepository);
    jest.clearAllMocks();
  });
  
  test('should handle user with custom properties', async () => {
    const specialUser = createUser({ 
      name: 'Special User', 
      email: 'special@example.com' 
    });
    
    mockUserRepository.findById.mockResolvedValue(specialUser);
    
    const result = await userService.getUserProfile(1);
    
    expect(result?.name).toBe('Special User');
    expect(result?.email).toBe('special@example.com');
  });
  
  test('should work with multiple users', async () => {
    const users = createUsers(3);
    mockUserRepository.findAll.mockResolvedValue(users);
    
    const allUsers = await mockUserRepository.findAll();
    
    expect(allUsers).toHaveLength(3);
    expect(allUsers[0].name).toBe('User 1');
    expect(allUsers[1].name).toBe('User 2');
    expect(allUsers[2].name).toBe('User 3');
  });
});
```

## Ключевые концепции

- **Типобезопасность в тестах**: использование типов для проверки корректности
- **Мокирование**: замена зависимостей для изолированного тестирования
- **Асинхронные тесты**: тестирование асинхронного кода с правильной обработкой
- **Фикстуры**: подготовленные данные для тестов
- **Покрытие типов**: убедиться, что все типы правильно протестированы

## Дальнейшее изучение

- [[../type-system/type-guards|Тип-гарды]]
- [[../generics/index|Обобщения]]
- [[Утилиты типов TypeScript|Утилиты типов]]
- [[../async-await/async-typescript-comprehensive|Асинхронный TypeScript]]
- [[Обработка ошибок в TypeScript|Обработка ошибок]]
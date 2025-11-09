# Модули в TypeScript

## Основы модульной системы

Модули в TypeScript позволяют организовывать код в отдельные файлы и управлять зависимостями между ними. Каждый файл в TypeScript считается модулем, если он содержит операторы `import` или `export`.

## Синтаксис экспорта

### Named экспорты

```ts
// math.ts
export const PI = 3.14159;

export function add(a: number, b: number): number {
  return a + b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

// Можно экспортировать по-разному
function subtract(a: number, b: number): number {
  return a - b;
}

export { subtract };
```

### Default экпорт

```ts
// calculator.ts
class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
  
  multiply(a: number, b: number): number {
    return a * b;
  }
}

export default Calculator;
```

### Mixed экспорты

```ts
// utils.ts
export const MAX_VALUE = 100;

export function validate(value: number): boolean {
  return value <= MAX_VALUE;
}

class Validator {
  static isValid(value: number): boolean {
    return value > 0 && value <= MAX_VALUE;
  }
}

export default Validator;
```

## Синтаксис импорта

### Named импорты

```ts
// main.ts
import { add, multiply, PI } from './math';
import { MAX_VALUE, validate } from './utils';
import Validator from './utils';

console.log(add(5, 3)); // 8
console.log(PI); // 3.14159
console.log(Validator.isValid(50)); // true
```

### Импорт всего модуля

```ts
import * as MathUtils from './math';
import * as Validators from './utils';

console.log(MathUtils.add(10, 5)); // 15
console.log(Validators.MAX_VALUE); // 100
```

### Импорт с переименованием

```ts
import { add as addNumbers, multiply as multiplyNumbers } from './math';

console.log(addNumbers(2, 3)); // 5
console.log(multiplyNumbers(4, 5)); // 20
```

### Реэкспорт

```ts
// index.ts
export { add, multiply } from './math';
export { default as Calculator } from './calculator';
export * from './utils'; // экспорт всего
```

## Типизация модулей

### Определение типов для модулей

```ts
// types.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export type UserRole = 'admin' | 'user' | 'guest';
```

### Использование типов из других модулей

```ts
// user-service.ts
import { User, UserRole } from './types';

export class UserService {
  private users: User[] = [];
  
  addUser(user: User): void {
    this.users.push(user);
  }
  
  getUserById(id: number): User | undefined {
    return this.users.find(user => user.id === id);
  }
  
  changeRole(user: User, role: UserRole): User {
    return { ...user, role };
  }
}
```

## Динамические импорты

```ts
// dynamic-import.ts
async function loadModule(moduleName: string) {
  switch (moduleName) {
    case 'math':
      const math = await import('./math');
      console.log(math.add(2, 3));
      break;
    case 'utils':
      const utils = await import('./utils');
      console.log(utils.validate(50));
      break;
    default:
      throw new Error(`Module ${moduleName} not found`);
  }
}

// Использование
loadModule('math');
```

## Обработка асинхронных операций с модулями

```ts
// async-module.ts
interface ModuleWithDefault {
  default: any;
}

async function loadConfig(): Promise<ModuleWithDefault> {
  try {
    const configModule = await import('./config.json');
    return configModule;
  } catch (error) {
    console.error('Failed to load config:', error);
    return { default: {} };
  }
}
```

## Практические примеры

### Пример: API сервис

```ts
// api-client.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

export class ApiClient {
  private baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  async get<T>(endpoint: string): Promise<ApiResponse<T>> {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    const data = await response.json();
    return {
      data,
      status: response.status,
      message: response.statusText
    };
  }
  
  async post<T>(endpoint: string, body: any): Promise<ApiResponse<T>> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(body)
    });
    const data = await response.json();
    return {
      data,
      status: response.status,
      message: response.statusText
    };
  }
}

export default new ApiClient('https://api.example.com');
```

### Использование сервиса

```ts
// main.ts
import api, { ApiResponse } from './api-client';

interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(userId: number): Promise<User> {
  const response: ApiResponse<User> = await api.get(`/users/${userId}`);
  
  if (response.status !== 200) {
    throw new Error(`Failed to fetch user: ${response.message}`);
  }
  
  return response.data;
}

// Использование
fetchUser(1)
  .then(user => console.log(user))
  .catch(error => console.error(error));
```

## Ключевые концепции

- **Named exports/imports**: именованные экспорты и импорты
- **Default exports/imports**: экспорт и импорт по умолчанию
- **Dynamic imports**: динамическая загрузка модулей
- **Module resolution**: как TypeScript находит модули
- **Type declarations**: объявление типов для модулей

## Дальнейшее изучение

- [[./module-systems|Системы модулей]]
- [[./import-export|Синтаксис импорта/экспорта]]
- [[./module-resolution|Разрешение модулей]]
- [[./bundling|Сборка модулей]]
- [[./dynamic-imports|Динамические импорты]]
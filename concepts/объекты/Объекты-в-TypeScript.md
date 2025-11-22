---
aliases: [TS-Objects, TypeScript-Objects, Объекты-TS]
tags: [typescript, frontend, programming, objects]
---

# Объекты в TypeScript

TypeScript значительно расширяет возможности работы с объектами по сравнению с чистым JavaScript, добавляя систему типов, которая позволяет создавать более надежный и предсказуемый код.

## Базовые типы объектов

### Встроенные типы объектов

```typescript
// Объявление объекта с конкретными свойствами
const person: { name: string; age: number; isActive: boolean } = {
  name: 'Иван',
  age: 30,
  isActive: true
};
```

### Интерфейсы

```typescript
interface Person {
  name: string;
  age: number;
  email?: string; // необязательное свойство
  readonly id: number; // только для чтения
}

const user: Person = {
  name: 'Иван',
  age: 30,
  id: 12345
};

// user.id = 67890; // Ошибка: нельзя изменить readonly свойство
```

### Типы объектов

```typescript
type Person = {
  name: string;
  age: number;
  email?: string;
};

// Можно использовать как алиас для сложных типов
type Address = {
  street: string;
  city: string;
  country: string;
};

type User = Person & {
  address: Address;
  permissions: string[];
};
```

## Вложенные объекты и массивы

```typescript
interface User {
  id: number;
  profile: {
    firstName: string;
    lastName: string;
    contact: {
      email: string;
      phone?: string;
    };
  };
  roles: string[];
}

const user: User = {
  id: 1,
  profile: {
    firstName: 'Иван',
    lastName: 'Иванов',
    contact: {
      email: 'ivan@example.com'
    }
  },
  roles: ['admin', 'user']
};
```

## Индексные типы

```typescript
// Объект с произвольными строковыми ключами
const config: { [key: string]: string | number } = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
};

// Или с использованием Record
const settings: Record<string, any> = {
  theme: 'dark',
  language: 'ru',
  notifications: true
};

// Строго определенные ключи
const permissions: Record<'read' | 'write' | 'delete', boolean> = {
  read: true,
  write: false,
  delete: false
};
```

## Методы объектов и функции

```typescript
interface Calculator {
  value: number;
  add(num: number): Calculator;
  subtract(num: number): Calculator;
  multiply(num: number): Calculator;
  getValue(): number;
}

const calculator: Calculator = {
  value: 0,
  
  add(num: number) {
    this.value += num;
    return this;
  },
  
  subtract(num: number) {
    this.value -= num;
    return this;
  },
  
  multiply(num: number) {
    this.value *= num;
    return this;
  },
  
  getValue() {
    return this.value;
  }
};
```

## Опциональные свойства и свойства только для чтения

```typescript
interface Config {
  apiUrl: string;
  timeout?: number; // опциональное свойство
  readonly version: string; // только для чтения
  features?: {
    darkMode?: boolean;
    notifications?: boolean;
  };
}

const config: Config = {
  apiUrl: 'https://api.example.com',
  version: '1.0.0'
};

// config.version = '2.0.0'; // Ошибка: нельзя изменить readonly свойство
```

## Деструктуризация с типами

```typescript
interface ApiResponse {
  data: any;
  status: number;
  message?: string;
}

function handleResponse({ data, status, message = 'OK' }: ApiResponse) {
  console.log(`Статус: ${status}, Данные:`, data, `Сообщение: ${message}`);
}

const response: ApiResponse = {
  data: { id: 1, name: 'Тест' },
  status: 200
};

handleResponse(response);
```

## Generic-объекты

```typescript
interface KeyValuePair<T, U> {
  key: T;
  value: U;
}

const pair: KeyValuePair<string, number> = {
  key: 'count',
  value: 42
};

// Generic функция для работы с объектами
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: 'Иван', age: 30, city: 'Москва' };
const name = getProperty(person, 'name'); // строка
const age = getProperty(person, 'age');   // число
```

## Partial, Required, Pick, Omit

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Partial - делает все свойства опциональными
const partialUser: Partial<User> = {
  name: 'Иван'
  // остальные свойства опциональны
};

// Required - делает все свойства обязательными (полезно если в базовом типе есть опциональные)
type UserWithRequiredEmail = Required<Pick<User, 'email'>> & Omit<User, 'email'>;

// Pick - выбирает только указанные свойства
const userPreview: Pick<User, 'id' | 'name'> = {
  id: 1,
  name: 'Иван'
};

// Omit - исключает указанные свойства
const userWithoutPassword: Omit<User, 'password'> = {
  id: 1,
  name: 'Иван',
  email: 'ivan@example.com'
};
```

##_mapped types_

```typescript
// Создание типа с опциональными свойствами
type Optional<T> = {
  [P in keyof T]?: T[P];
};

// Создание типа только для чтения
type ReadOnly<T> = {
  readonly [P in keyof T]: T[P];
};

// Создание типа с префиксом для всех ключей
type Proxify<T> = {
  [P in keyof T]: { get(): T[P], set(value: T[P]): void }
};

interface Person {
  name: string;
  age: number;
}

const optionalPerson: Optional<Person> = {
  name: 'Иван'
  // age опционально
};

const readOnlyPerson: ReadOnly<Person> = {
  name: 'Иван',
  age: 30
};
// readOnlyPerson.name = 'Петр'; // Ошибка: нельзя изменить readonly свойство
```

## Классы и объекты

```typescript
class User {
  constructor(
    public id: number,
    public name: string,
    private email: string,
    protected role: string = 'user'
  ) {}
  
  public getInfo(): { id: number; name: string; role: string } {
    return {
      id: this.id,
      name: this.name,
      role: this.role
    };
  }
  
  protected setEmail(email: string): void {
    this.email = email;
  }
}

const user = new User(1, 'Иван', 'ivan@example.com');
const info = user.getInfo(); // правильно типизированный объект
```

## Асинхронные объекты и промисы

```typescript
interface ApiClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
  put<T>(url: string, data: any): Promise<T>;
  delete<T>(url: string): Promise<T>;
}

class HttpClient implements ApiClient {
  async get<T>(url: string): Promise<T> {
    const response = await fetch(url);
    return response.json() as Promise<T>;
  }
  
  async post<T>(url: string, data: any): Promise<T> {
    const response = await fetch(url, {
      method: 'POST',
      body: JSON.stringify(data),
      headers: {
        'Content-Type': 'application/json'
      }
    });
    return response.json() as Promise<T>;
  }
  
  // остальные методы...
}
```

## Практические рекомендации

### Использование интерфейсов для API-ответов

```typescript
interface UserApiResponse {
  data: {
    id: number;
    name: string;
    email: string;
    created_at: string;
  };
  success: boolean;
  message?: string;
}

async function fetchUser(id: number): Promise<UserApiResponse> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

### Создание типов для конфигурации

```typescript
interface AppConfig {
  api: {
    baseUrl: string;
    timeout: number;
    retries: number;
  };
  ui: {
    theme: 'light' | 'dark';
    language: string;
    animations: boolean;
  };
  features: {
    [key: string]: boolean;
  };
}

const config: AppConfig = {
  api: {
    baseUrl: process.env.API_URL || 'https://api.example.com',
    timeout: 5000,
    retries: 3
  },
  ui: {
    theme: 'light',
    language: 'ru',
    animations: true
  },
  features: {
    notifications: true,
    darkMode: false
  }
};
```

## Заключение

TypeScript значительно улучшает работу с объектами за счет строгой типизации, что позволяет избежать многих ошибок на этапе компиляции. Правильное использование интерфейсов, generic-типов и утилитарных типов делает код более надежным и поддерживаемым.

См. также:
- [[Объекты-в-JavaScript]]
- [[Типы-в-TypeScript]]
- [[Объекты-в-React-компонентах]]
- [[Объекты-и-их-свойства]]
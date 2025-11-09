# Обработка ошибок в TypeScript

## Введение в обработку ошибок

Обработка ошибок в TypeScript включает в себя типобезопасные способы управления исключениями и ошибками, которые могут возникнуть во время выполнения программы. TypeScript предоставляет дополнительные возможности по сравнению с JavaScript, позволяя более эффективно обрабатывать ошибки.

## Типы ошибок в TypeScript

### Системные ошибки

```ts
// Ошибки компиляции TypeScript
// const message: string = 42; // Ошибка: Type '42' is not assignable to type 'string'
```

### Ошибки выполнения

```ts
// Ошибки, возникающие во время выполнения
function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error("Division by zero is not allowed");
  }
  return a / b;
}

try {
  const result = divide(10, 0);
} catch (error) {
  if (error instanceof Error) {
    console.error(error.message); // "Division by zero is not allowed"
  }
}
```

## Типобезопасная обработка ошибок

### Использование Union Types для результатов с ошибками

```ts
type Success<T> = { success: true; data: T };
type Failure = { success: false; error: string };
type Result<T> = Success<T> | Failure;

function safeDivide(a: number, b: number): Result<number> {
  if (b === 0) {
    return { success: false, error: "Division by zero" };
  }
  return { success: true, data: a / b };
}

// Использование
const result = safeDivide(10, 2);
if (result.success) {
  console.log(`Result: ${result.data}`); // Result: 5
} else {
  console.error(`Error: ${result.error}`);
}
```

### Типы Never и Unknown

```ts
// never - тип для функций, которые никогда не возвращают значение
function throwError(message: string): never {
  throw new Error(message);
}

// unknown - безопасная альтернатива any
function processValue(value: unknown): string {
  if (typeof value === 'string') {
    return value.toUpperCase();
  } else if (value === null) {
    return "NULL value";
  } else if (value === undefined) {
    return "UNDEFINED value";
  } else {
    return "UNKNOWN type";
  }
}
```

## Async/Await и обработка ошибок

```ts
interface User {
  id: number;
  name: string;
}

async function fetchUser(id: number): Promise<Result<User>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const user: User = await response.json();
    return { success: true, data: user };
  } catch (error) {
    if (error instanceof Error) {
      return { success: false, error: error.message };
    } else {
      return { success: false, error: "Unknown error occurred" };
    }
  }
}

// Использование
async function handleUser(id: number): Promise<void> {
  const result = await fetchUser(id);
  if (result.success) {
    console.log(`User: ${result.data.name}`);
  } else {
    console.error(`Failed to fetch user: ${result.error}`);
  }
}
```

## Тип-гарды для обработки ошибок

```ts
// Пользовательский тип-гард
function isErrorLike(value: unknown): value is { message: string } {
  return (
    typeof value === 'object' &&
    value !== null &&
    'message' in value &&
    typeof (value as any).message === 'string'
  );
}

function handleUnknownError(error: unknown): string {
  if (isErrorLike(error)) {
    return error.message;
  } else if (typeof error === 'string') {
    return error;
  } else {
    return "An unknown error occurred";
  }
}
```

## Обработка ошибок в API

```ts
interface ApiResponse<T> {
  data?: T;
  error?: string;
  status: number;
}

class ApiClient {
  async request<T>(url: string): Promise<ApiResponse<T>> {
    try {
      const response = await fetch(url);
      const data = await response.json();
      
      if (!response.ok) {
        return { error: data.message || "Request failed", status: response.status };
      }
      
      return { data, status: response.status };
    } catch (error) {
      return { 
        error: error instanceof Error ? error.message : "Network error", 
        status: 0 
      };
    }
  }
}

// Использование
const api = new ApiClient();
api.request<User>('/api/users/1')
  .then(response => {
    if (response.error) {
      console.error(`API Error: ${response.error}`);
    } else {
      console.log(`User data: ${JSON.stringify(response.data)}`);
    }
  });
```

## Обработка ошибок в middleware

```ts
type Middleware<T> = (data: T) => Result<T>;

function validateUser(user: any): Result<User> {
  if (!user || typeof user !== 'object') {
    return { success: false, error: "Invalid user object" };
  }
  
  if (typeof user.id !== 'number' || user.id <= 0) {
    return { success: false, error: "Invalid user id" };
  }
  
  if (typeof user.name !== 'string' || user.name.trim() === '') {
    return { success: false, error: "Invalid user name" };
  }
  
  return { success: true, data: { id: user.id, name: user.name } };
}

function processUser(userData: unknown): Result<User> {
  if (typeof userData !== 'object' || userData === null) {
    return { success: false, error: "User data must be an object" };
  }
  
  return validateUser(userData);
}
```

## Кастомные ошибки

```ts
class ValidationError extends Error {
  constructor(public field: string, public value: any) {
    super(`Validation error in field "${field}" with value: ${value}`);
    this.name = 'ValidationError';
  }
}

class NetworkError extends Error {
  constructor(public url: string, public status: number) {
    super(`Network error for ${url}: ${status}`);
    this.name = 'NetworkError';
  }
}

// Использование
function validateEmail(email: string): void {
  if (!email.includes('@')) {
    throw new ValidationError('email', email);
  }
}

try {
  validateEmail('invalid-email');
} catch (error) {
  if (error instanceof ValidationError) {
    console.error(`Field: ${error.field}, Value: ${error.value}`);
  }
}
```

## Ключевые концепции

- **Типобезопасность**: использование типов для предотвращения ошибок
- **Result-типы**: безопасная обработка результатов с возможными ошибками
- **Тип-гарды**: проверка типов во время выполнения
- **Кастомные ошибки**: создание специфичных для домена ошибок
- **Async обработка**: обработка ошибок в асинхронном коде

## Дальнейшее изучение

- [[../type-system/type-guards|Тип-гарды]]
- [[../type-system/type-assertions-narrowing|Утверждения и сужение типов]]
- [[../async-await/async-typescript-comprehensive|Асинхронный TypeScript]]
- [[../utility-types/index|Утилиты типов]]
- [[../generics/index|Обобщения]]
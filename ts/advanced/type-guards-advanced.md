---
aliases: ["Продвинутые тип-гарды", "TS Advanced Type Guards"]
tags: [typescript, type-guards, advanced, patterns]
---

# Продвинутые тип-гарды

## Введение

Тип-гарды - это выражения, которые проверяют тип переменной в рантайме и обеспечивают сужение типа в соответствующей ветке кода. Продвинутые тип-гарды позволяют создавать более сложные и гибкие проверки типов, что особенно полезно при работе с объединениями типов (union types) и неизвестными данными.

## Базовые тип-гарды

### Простые пользовательские тип-гарды

```typescript
// Простой пользовательский тип-гард
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}

function processValue(value: unknown) {
  if (isString(value)) {
    // В этой ветке TypeScript знает, что value - это string
    return value.toUpperCase(); // можно вызывать строковые методы
  } else if (isNumber(value)) {
    // В этой ветке TypeScript знает, что value - это number
    return value.toFixed(2); // можно вызывать числовые методы
  }
  return 'Unknown type';
}

// Примеры использования
console.log(processValue('hello')); // 'HELLO'
console.log(processValue(42)); // '42.00'
console.log(processValue(true)); // 'Unknown type'
```

### Тип-гарды для объектов

```typescript
// Тип-гард для проверки наличия свойства
function hasProperty<T extends string>(obj: unknown, prop: T): obj is Record<T, unknown> {
  return typeof obj === 'object' && obj !== null && prop in obj;
}

// Более конкретный тип-гард для объекта пользователя
interface User {
  id: number;
  name: string;
  email: string;
}

function isUser(obj: unknown): obj is User {
  if (typeof obj !== 'object' || obj === null) {
    return false;
  }
  
  const user = obj as Record<string, unknown>;
  return (
    typeof user.id === 'number' &&
    typeof user.name === 'string' &&
    typeof user.email === 'string'
  );
}

// Пример использования
const data: unknown = { id: 1, name: 'John', email: 'john@example.com' };

if (isUser(data)) {
  // В этой ветке TypeScript знает, что data - это User
  console.log(data.name); // безопасный доступ к свойствам
}
```

## Продвинутые тип-гарды

### Тип-гарды с дженериками

```typescript
// Универсальный тип-гард для проверки типа
function isOfType<T>(value: unknown, typeGuard: (v: unknown) => v is T): value is T {
  return typeGuard(value);
}

// Тип-гард для массивов
function isArray<T>(value: unknown, elementGuard: (v: unknown) => v is T): value is T[] {
  return Array.isArray(value) && value.every(elementGuard);
}

// Пример использования
const stringArray: unknown = ['hello', 'world'];
const numberArray: unknown = [1, 2, 3];

if (isArray(stringArray, isString)) {
  // В этой ветке TypeScript знает, что stringArray - это string[]
  console.log(stringArray.map(s => s.toUpperCase()));
}

if (isArray(numberArray, isNumber)) {
  // В этой ветке TypeScript знает, что numberArray - это number[]
  console.log(numberArray.map(n => n.toFixed(2)));
}
```

### Тип-гарды для проверки форм

```typescript
// Тип-гард для проверки форм с определенной структурой
interface LoginForm {
  email: string;
  password: string;
}

interface RegisterForm {
  email: string;
  password: string;
  confirmPassword: string;
}

function isLoginForm(data: unknown): data is LoginForm {
  if (typeof data !== 'object' || data === null) {
    return false;
  }
  
  const form = data as Record<string, unknown>;
  return (
    typeof form.email === 'string' &&
    typeof form.password === 'string' &&
    form.email.includes('@') // простая валидация email
  );
}

function isRegisterForm(data: unknown): data is RegisterForm {
  if (!isLoginForm(data)) {
    return false;
  }
  
  const form = data as LoginForm & Record<string, unknown>;
  return (
    typeof form.confirmPassword === 'string' &&
    form.password === form.confirmPassword
  );
}

// Использование в рантайме
const formData: unknown = { email: 'test@example.com', password: '123456', confirmPassword: '123456' };

if (isRegisterForm(formData)) {
  // В этой ветке TypeScript знает, что formData - это RegisterForm
  console.log('Valid register form:', formData);
} else if (isLoginForm(formData)) {
  // В этой ветке TypeScript знает, что formData - это LoginForm
  console.log('Valid login form:', formData);
}
```

### Тип-гарды для проверки дат

```typescript
// Тип-гард для проверки даты
function isDate(value: unknown): value is Date {
  return value instanceof Date && !isNaN(value.getTime());
}

// Тип-гард для проверки строки даты ISO
function isDateString(value: unknown): value is string {
  if (typeof value !== 'string') {
    return false;
  }
  
  const date = new Date(value);
  return !isNaN(date.getTime()) && date.toISOString() === value;
}

// Комбинированный тип-гард для дат
function isDateLike(value: unknown): value is Date | string {
  return isDate(value) || isDateString(value);
}

// Пример использования
function formatDateTime(value: unknown): string {
  if (isDate(value)) {
    // value - это Date
    return value.toLocaleString();
  } else if (isDateString(value)) {
    // value - это строка в формате ISO
    return new Date(value).toLocaleString();
  }
  return 'Invalid date';
}
```

## Практические примеры

### Тип-гарды для валидации API ответов

```typescript
// Тип-гард для проверки успешного ответа API
interface SuccessResponse<T> {
  success: true;
  data: T;
}

interface ErrorResponse {
  success: false;
  error: string;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function isSuccessResponse<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
  return response.success === true;
}

function isErrorResponse<T>(response: ApiResponse<T>): response is ErrorResponse {
  return response.success === false;
}

// Пример использования
async function fetchUser(id: number): Promise<User> {
  const response: ApiResponse<User> = await fetch(`/api/users/${id}`).then(r => r.json());
  
  if (isSuccessResponse(response)) {
    // В этой ветке TypeScript знает, что response - это SuccessResponse<User>
    return response.data;
  } else if (isErrorResponse(response)) {
    // В этой ветке TypeScript знает, что response - это ErrorResponse
    throw new Error(response.error);
  }
  
  throw new Error('Unexpected response format');
}
```

### Тип-гарды для проверки конфигурации

```typescript
// Интерфейс для конфигурации
interface AppConfig {
  apiUrl: string;
  timeout: number;
  debug: boolean;
  features: string[];
}

// Тип-гард для проверки конфигурации
function isAppConfig(obj: unknown): obj is AppConfig {
  if (typeof obj !== 'object' || obj === null) {
    return false;
  }
  
  const config = obj as Record<string, unknown>;
  return (
    typeof config.apiUrl === 'string' &&
    typeof config.timeout === 'number' &&
    typeof config.debug === 'boolean' &&
    Array.isArray(config.features) &&
    config.features.every(f => typeof f === 'string')
  );
}

// Тип-гард для частичной конфигурации
function isPartialAppConfig(obj: unknown): obj is Partial<AppConfig> {
  if (typeof obj !== 'object' || obj === null) {
    return false;
  }
  
  const config = obj as Record<string, unknown>;
  return (
    config.apiUrl === undefined || typeof config.apiUrl === 'string' &&
    config.timeout === undefined || typeof config.timeout === 'number' &&
    config.debug === undefined || typeof config.debug === 'boolean' &&
    config.features === undefined || Array.isArray(config.features) &&
    (config.features === undefined || config.features.every(f => typeof f === 'string'))
  );
}

// Пример использования
const rawConfig: unknown = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  debug: true,
  features: ['feature1', 'feature2']
};

if (isAppConfig(rawConfig)) {
  // В этой ветке TypeScript знает, что rawConfig - это AppConfig
  console.log('Valid configuration:', rawConfig);
} else {
  console.error('Invalid configuration');
}
```

### Тип-гарды для проверки событий

```typescript
// Тип-гард для проверки пользовательских событий
interface CustomEventPayload {
  type: string;
  data: Record<string, any>;
}

function isCustomEventPayload(obj: unknown): obj is CustomEventPayload {
  if (typeof obj !== 'object' || obj === null) {
    return false;
  }
  
  const event = obj as Record<string, unknown>;
  return (
    typeof event.type === 'string' &&
    typeof event.data === 'object' &&
    event.data !== null
  );
}

// Тип-гард для проверки события с определенным типом
function isEventType<T extends string>(obj: unknown, eventType: T): obj is CustomEventPayload & { type: T } {
  return isCustomEventPayload(obj) && obj.type === eventType;
}

// Пример использования
const event: unknown = { type: 'USER_LOGIN', data: { userId: 123, timestamp: Date.now() } };

if (isEventType(event, 'USER_LOGIN')) {
  // В этой ветке TypeScript знает, что event - это CustomEventPayload с type 'USER_LOGIN'
  console.log('User logged in:', event.data.userId);
}
```

## Сложные примеры

### Тип-гарды для проверки сложных структур данных

```typescript
// Тип-гард для проверки древовидной структуры
interface TreeNode<T> {
  value: T;
  children?: TreeNode<T>[];
}

function isTreeNode<T>(obj: unknown, valueGuard: (v: unknown) => v is T): obj is TreeNode<T> {
  if (typeof obj !== 'object' || obj === null) {
    return false;
  }
  
  const node = obj as Record<string, unknown>;
  if (!valueGuard(node.value)) {
    return false;
  }
  
  if (node.children !== undefined) {
    if (!Array.isArray(node.children)) {
      return false;
    }
    
    // Рекурсивная проверка дочерних узлов
    return node.children.every(child => isTreeNode(child, valueGuard));
  }
  
  return true;
}

// Пример использования
const treeData: unknown = {
  value: 'root',
  children: [
    { value: 'child1' },
    { 
      value: 'child2', 
      children: [
        { value: 'grandchild' }
      ] 
    }
  ]
};

if (isTreeNode(treeData, isString)) {
  // В этой ветке TypeScript знает, что treeData - это TreeNode<string>
  console.log('Valid tree structure:', treeData.value);
}
```

### Тип-гарды для проверки функций

```typescript
// Тип-гард для проверки функций с определенной сигнатурой
function isFunction<T extends (...args: any[]) => any>(value: unknown): value is T {
  return typeof value === 'function';
}

// Тип-гард для проверки функции с определенными параметрами и возвращаемым типом
function isSpecificFunction<Args extends any[], Return>(
  value: unknown,
  paramGuards: { [K in keyof Args]: (arg: unknown) => arg is Args[K] },
  returnGuard: (ret: unknown) => ret is Return
): value is (...args: Args) => Return {
  if (!isFunction(value)) {
    return false;
  }
  
  // В реальных приложениях проверка сигнатуры функции может быть сложной
  // Этот пример показывает концепцию
  return true;
}

// Пример использования
const processData = (data: string, count: number): boolean => {
  return data.length > count;
};

function checkFunction(fn: unknown) {
  if (isFunction(fn)) {
    // В этой ветке TypeScript знает, что fn - это функция
    console.log('Function received');
  }
}
```

### Комбинированные тип-гарды

```typescript
// Утилита для комбинации тип-гардов
function combineGuards<T1, T2>(
  guard1: (value: unknown) => value is T1,
  guard2: (value: unknown) => value is T2
): (value: unknown) => value is (T1 & T2) {
  return (value: unknown): value is (T1 & T2) => {
    return guard1(value) && guard2(value);
  };
}

// Утилита для альтернативных тип-гардов
function eitherGuard<T1, T2>(
  guard1: (value: unknown) => value is T1,
  guard2: (value: unknown) => value is T2
): (value: unknown) => value is (T1 | T2) {
  return (value: unknown): value is (T1 | T2) => {
    return guard1(value) || guard2(value);
  };
}

// Пример использования
const isStringOrNumber = eitherGuard(isString, isNumber);

const value: unknown = 42;
if (isStringOrNumber(value)) {
  // В этой ветке TypeScript знает, что value - это string | number
  console.log('Value is string or number:', value);
}
```

## Заключение

Продвинутые тип-гарды позволяют создавать надежные системы проверки типов в рантайме, что особенно важно при работе с внешними API, пользовательским вводом и другими источниками неизвестных данных. Они обеспечивают безопасность типов и улучшают автодополнение в IDE.

См. также: [[discriminated-unions]] | [[generic-constraints-advanced]] | [[conditional-types-tricks]]
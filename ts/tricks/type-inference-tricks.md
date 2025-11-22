---
aliases: ["Хитрости с выводом типов TypeScript", "TS Type Inference Tricks"]
tags: [typescript, type-inference, advanced]
---

# Хитрости с выводом типов TypeScript

## Введение

Вывод типов в TypeScript позволяет компилятору автоматически определять типы переменных, параметров функций и возвращаемых значений. Понимание механизмов вывода типов помогает писать более читаемый и безопасный код.

## Основы вывода типов

### Простой вывод типов

```typescript
// TypeScript автоматически выводит тип
const userName = 'John'; // string
const userAge = 30; // number
const isActive = true; // boolean

// Вывод типов для объектов
const user = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};
// user: { id: number; name: string; email: string; }
```

### Вывод типов для функций

```typescript
// Вывод возвращаемого типа
function getUser(id: number) {
  return {
    id,
    name: `User ${id}`,
    email: `user${id}@example.com`
  };
}
// Возвращаемый тип: { id: number; name: string; email: string; }

// Вывод типов параметров через дженерики
function identity<T>(arg: T): T {
  return arg;
}

const result = identity('hello'); // T выводится как string
```

## Продвинутые примеры

### Вывод типов через infer

```typescript
// Использование infer для извлечения типов
type ElementType<T> = T extends (infer U)[] ? U : T;

type ArrayElement = ElementType<string[]>; // string
type NonArray = ElementType<string>; // string

// Извлечение возвращаемого типа функции
type ReturnOf<T> = T extends (...args: any[]) => infer R ? R : never;

function createUser(): User {
  return { id: 1, name: 'John', email: 'john@example.com', password: 'secret', createdAt: new Date(), updatedAt: new Date() };
}

type CreatedUser = ReturnOf<typeof createUser>; // User
```

### Вывод типов в дженериках

```typescript
// Вывод типов для кортежей
function tuple<T extends unknown[]>(...args: T): T {
  return args;
}

const result1 = tuple('hello', 42, true);
// result1: [string, number, boolean]

// Вывод типов с ограничениями
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

const user = { id: 1, name: 'John', email: 'john@example.com' };
const userName = getProperty(user, 'name'); // string
const userId = getProperty(user, 'id'); // number
```

### Вывод типов в условиях

```typescript
// Вывод типов в зависимости от условий
function processValue<T extends string | number>(value: T): T extends string ? string[] : number[] {
  if (typeof value === 'string') {
    return value.split('') as any;
  } else {
    return [value] as any;
  }
}

const stringResult = processValue('hello'); // string[]
const numberResult = processValue(42); // number[]
```

## Практические примеры

### Вывод типов для функций обратного вызова

```typescript
// Вывод типов для функций обратного вызова
function processData<T>(
  data: T,
  callback: (item: T) => string
): string {
  return callback(data);
}

const result2 = processData(42, num => `Number: ${num}`);
// callback принимает number, возвращает string
```

### Вывод типов в массивах и объектах

```typescript
// Вывод типов для сложных структур
const users = [
  { id: 1, name: 'John', role: 'admin' },
  { id: 2, name: 'Jane', role: 'user' }
] as const;

type UserType = typeof users[number];
// { readonly id: number; readonly name: string; readonly role: "admin" | "user"; }

type RoleType = UserType['role'];
// "admin" | "user"
```

### Вывод типов для асинхронных функций

```typescript
// Вывод типов для асинхронных функций
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// TypeScript выводит, что fetchUser возвращает Promise<User>
const userPromise = fetchUser(1);
// userPromise: Promise<User>

// Извлечение типа из Promise
type AwaitedUser = Awaited<ReturnType<typeof fetchUser>>;
// User
```

### Вывод типов в функциях высшего порядка

```typescript
// Вывод типов в функциях высшего порядка
function createMapper<T, U>(fn: (item: T) => U) {
  return function(items: T[]): U[] {
    return items.map(fn);
  };
}

const stringToNumber = createMapper((s: string) => s.length);
const lengths = stringToNumber(['hello', 'world']); // number[]
```

## Продвинутые паттерны

### Вывод типов с использованием перегрузок

```typescript
// Перегрузки для лучшего вывода типов
function handleValue(x: number): number;
function handleValue(x: string): string;
function handleValue(x: boolean): boolean;
function handleValue(x: number | string | boolean) {
  return x;
}

const numResult = handleValue(42); // number
const strResult = handleValue('hello'); // string
const boolResult = handleValue(true); // boolean
```

### Вывод типов для объектов с динамическими ключами

```typescript
// Вывод типов для объектов с динамическими ключами
function createRecord<K extends string, V>(key: K, value: V) {
  return { [key]: value } as Record<K, V>;
}

const record = createRecord('name', 'John');
// record: { name: string; }
```

### Вывод типов в условных выражениях

```typescript
// Вывод типов в условных выражениях
function getValue<T>(condition: boolean, value: T, defaultValue: T): T {
  return condition ? value : defaultValue;
}

const result3 = getValue(true, 'hello', 'world'); // string
const result4 = getValue(false, 42, 0); // number
```

### Вывод типов для Union Types

```typescript
// Вывод типов для Union Types
type ApiResponse = { success: true; data: User } | { success: false; error: string };

function handleResponse(response: ApiResponse) {
  if (response.success) {
    // В этом блоке TypeScript знает, что response - это { success: true; data: User }
    console.log(response.data.name); // можно безопасно обращаться к data
  } else {
    // А в этом - { success: false; error: string }
    console.log(response.error); // можно безопасно обращаться к error
  }
}
```

### Вывод типов для дженериков с ограничениями

```typescript
// Вывод типов с использованием extends
function getPropertySafely<T extends { [key: string]: any }, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const userWithOptional = { id: 1, name: 'John', email: undefined as string | undefined };
const email = getPropertySafely(userWithOptional, 'email'); // string | undefined
```

## Заключение

Эффективное использование вывода типов позволяет писать более читаемый и безопасный код на TypeScript. Понимание механизмов вывода типов помогает создавать гибкие и типобезопасные API, а также улучшает опыт разработки за счет лучшего автодополнения и проверки типов.

См. также: [[advanced-utility-types]] | [[conditional-types-tricks]] | [[mapped-types-tricks]]
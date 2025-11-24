---
aliases: ["Infer", "Инфер", "Type Inference"]
tags: [typescript, advanced-types, infer, type-inference]
---

# Infer (Инфер) в TypeScript

Ключевое слово `infer` в TypeScript используется в условных типах для объявления переменной типа, которая будет выведена из типа в определенном контексте. Оно позволяет "захватывать" и использовать типы из сложных структур данных, что делает условные типы гораздо более мощными и гибкими.

## Основы использования infer

Ключевое слово `infer` может использоваться только в контексте условных типов (`T extends U ? X : Y`). Оно позволяет захватить тип из "правой" части условия.

```ts
type ExtractType<T> = T extends (infer U)[] ? U : T;

type A = ExtractType<string[]>;     // string
type B = ExtractType<number>;       // number (условие не выполняется)
type C = ExtractType<(string | number)[]>;  // string | number
```

## Извлечение возвращаемого типа функции

Один из самых распространенных случаев использования `infer` - это извлечение возвращаемого типа функции:

```ts
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { name: "Alice", age: 30 };
}

type User = GetReturnType<typeof getUser>;  // { name: string; age: number; }
```

## Извлечение параметров функции

Можно также извлекать типы параметров функции:

```ts
type GetParameters<T> = T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number, active: boolean) {
  return `Hello, ${name}!`;
}

type GreetParams = GetParameters<typeof greet>;  // [string, number, boolean]
```

## Извлечение типа элемента массива

```ts
type ArrayElementType<T> = T extends (infer U)[] ? U : never;

type T1 = ArrayElementType<number[]>;        // number
type T2 = ArrayElementType<string[]>;        // string
type T3 = ArrayElementType<(number | string)[]>;  // number | string
```

## Использование infer с несколькими переменными

Можно использовать несколько `infer` в одном условном типе:

```ts
type FuncInfo<T> = T extends (arg: infer A, callback: infer C) => infer R
  ? { arg: A; callback: C; return: R }
  : never;

function processData(data: string, onComplete: (result: boolean) => void): number {
  return 42;
}

type Info = FuncInfo<typeof processData>;
// { arg: string; callback: (result: boolean) => void; return: number }
```

## Продвинутые примеры

### Извлечение типа Promise

```ts
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type T1 = UnwrapPromise<Promise<string>>;           // string
type T2 = UnwrapPromise<Promise<number>>;           // number
type T3 = UnwrapPromise<Promise<Promise<boolean>>>; // Promise<boolean>

// Для рекурсивного извлечения
type DeepUnwrapPromise<T> = T extends Promise<infer U> 
  ? DeepUnwrapPromise<U> 
  : T;

type T4 = DeepUnwrapPromise<Promise<Promise<boolean>>>;  // boolean
```

### Извлечение типа из объекта

```ts
type PropType<T, K extends keyof T> = T[K] extends infer U ? U : never;

interface User {
  id: number;
  name: string;
  email: string;
}

type NameType = PropType<User, 'name'>;  // string
type IdType = PropType<User, 'id'>;      // number
```

### Извлечение типа из кортежа

```ts
type FirstElement<T> = T extends [infer U, ...any[]] ? U : never;

type T1 = FirstElement<[string, number, boolean]>;  // string
type T2 = FirstElement<[number]>;                   // number
type T3 = FirstElement<[]>;                         // never
```

### Извлечение типа из аргументов функции

```ts
type LastParameter<T> = T extends (...args: [...infer Args]) => any
  ? Args extends [...infer _, infer Last] ? Last : never
  : never;

function example(a: string, b: number, c: boolean) {}

type LastArg = LastParameter<typeof example>;  // boolean
```

## Практические применения

### 1. Создание утилит для асинхронных функций

```ts
type AsyncFunctionResult<T> = T extends () => Promise<infer U>
  ? U
  : T extends (...args: any[]) => Promise<infer U>
    ? U
    : never;

async function fetchUser() {
  return { id: 1, name: "John", email: "john@example.com" };
}

type UserResult = AsyncFunctionResult<typeof fetchUser>;  // { id: number; name: string; email: string; }
```

### 2. Обработка типов с разными структурами

```ts
type Unwrap<T> = 
  T extends (infer U)[] ? U : 
  T extends Promise<infer U> ? U : 
  T extends (...args: any[]) => infer U ? U : 
  T;

type T1 = Unwrap<string[]>;        // string
type T2 = Unwrap<Promise<number>>; // number
type T3 = Unwrap<() => boolean>;   // boolean
type T4 = Unwrap<Date>;            // Date
```

### 3. Извлечение типов из обобщенных структур

```ts
type ExtractGeneric<T> = T extends Map<infer K, infer V> 
  ? { key: K; value: V } 
  : T extends Set<infer E> 
    ? { element: E } 
    : never;

const myMap = new Map<string, number>();
type MapTypes = ExtractGeneric<typeof myMap>;  // { key: string; value: number }

const mySet = new Set<boolean>();
type SetTypes = ExtractGeneric<typeof mySet>;  // { element: boolean }
```

## Ограничения и особенности

### 1. Ограничения на количество infer

В одном условном выражении может быть несколько `infer`, но каждый должен быть уникальным:

```ts
// Правильно
type MultiInfer<T> = T extends [infer A, infer B, infer C] ? [A, B, C] : never;

// Ошибка - дублирование
// type BadInfer<T> = T extends [infer A, infer A] ? A : never;  // Ошибка!
```

### 2. Ограничения на контекст использования

`infer` можно использовать только в правой части условия `extends` в условных типах:

```ts
// Правильно
type Valid<T> = T extends (infer U)[] ? U : never;

// Ошибка
// type Invalid<T> = (infer U)[] extends T ? U : never;  // Ошибка!
```

### 3. Ограничения на рекурсию

Слишком сложные структуры с `infer` могут привести к ошибке из-за ограничений на глубину рекурсии TypeScript.

## Связанные концепции

- [[Условные-типы]] - основа для использования infer
- [[Сопоставляющие-типы]] - часто используют infer для извлечения типов
- [[Template Literal Types]] - могут использоваться вместе с infer
- [[Generic Types]] - основа для условных типов с infer
- [[Keyof]] - часто используется в сочетании с infer

## Заключение

Ключевое слово `infer` является мощным инструментом для создания сложных типов в TypeScript. Оно позволяет извлекать и использовать типы из сложных структур данных, что делает условные типы гораздо более гибкими и мощными. Понимание и умелое использование `infer` открывает возможности для создания сложных типовых утилит и систем типизации.
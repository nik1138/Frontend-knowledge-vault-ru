# Условные типы в TypeScript

Условные типы (conditional types) - это мощная возможность TypeScript, позволяющая определять типы, которые зависят от условий. Они работают по принципу "если T соответствует U, то использовать тип X, иначе использовать тип Y", что делает систему типов динамичной и выразительной.

## Основной синтаксис

Условные типы используют синтаксис `T extends U ? X : Y`, где:
- `T` - входной тип
- `U` - тип для проверки
- `X` - тип, возвращаемый если условие истинно
- `Y` - тип, возвращаемый если условие ложно

```typescript
type IsString<T> = T extends string ? true : false;

type T0 = IsString<string>;    // true
type T1 = IsString<number>;    // false
type T2 = IsString<"hello">;   // true (литерал строки)
```

## Простые примеры условных типов

### Базовые проверки

```typescript
// Проверка на null или undefined
type NonNullable<T> = T extends null | undefined ? never : T;

type T0 = NonNullable<string>;           // string
type T1 = NonNullable<string | null>;   // string
type T2 = NonNullable<undefined>;       // never

// Проверка на функцию
type IsFunction<T> = T extends (...args: any[]) => any ? "function" : "not function";

type T3 = IsFunction<(x: number) => string>;  // "function"
type T4 = IsFunction<string>;                 // "not function"
```

### Извлечение типов из объединений

```typescript
// Извлечение типов из объединения
type Extract<T, U> = T extends U ? T : never;
type Exclude<T, U> = T extends U ? never : T;

type T5 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T6 = Exclude<"a" | "b" | "c", "a" | "f">;  // "b" | "c"
```

## Вывод типов с помощью infer

Одна из самых мощных возможностей условных типов - использование `infer` для вывода типов внутри условных выражений:

```typescript
// Извлечение возвращаемого типа функции
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

type T7 = ReturnType<() => string>;           // string
type T8 = ReturnType<(x: number) => number>;  // number
type T9 = ReturnType<number>;                 // any (не функция)

// Извлечение типа параметра функции
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

type T10 = Parameters<(a: number, b: string) => void>;  // [number, string]
type T11 = Parameters<string>;                          // never (не функция)
```

### Продвинутый вывод типов

```typescript
// Извлечение типа элемента массива
type ElementOf<T> = T extends (infer U)[] ? U : T;

type T12 = ElementOf<string[]>;      // string
type T13 = ElementOf<number[]>;     // number
type T14 = ElementOf<string>;       // string (не массив)

// Извлечение типа промиса
type PromiseType<T> = T extends Promise<infer U> ? U : T;

type T15 = PromiseType<Promise<string>>;  // string
type T16 = PromiseType<string>;          // string (не промис)
```

## Практические применения

### 1. Создание утилит типов

```typescript
// Создание типа для выбора свойств определенного типа
type PickByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};

interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
  createdAt: Date;
}

type StringProps = PickByType<User, string>;  
// { name: string; email: string }

type BooleanProps = PickByType<User, boolean>;  
// { isActive: boolean }
```

### 2. Преобразование типов

```typescript
// Преобразование объекта: все функции становятся асинхронными
type Asyncify<T> = {
  [K in keyof T]: T[K] extends (...args: infer A) => infer R
    ? (...args: A) => Promise<R>
    : T[K]
};

interface Api {
  getUser: (id: number) => User;
  saveUser: (user: User) => boolean;
  validate: () => void;
}

type AsyncApi = Asyncify<Api>;
// {
//   getUser: (id: number) => Promise<User>;
//   saveUser: (user: User) => Promise<boolean>;
//   validate: () => Promise<void>;
// }
```

### 3. Работа с API ответами

```typescript
// Извлечение типа данных из ответа API
type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
}

type GetData<T> = T extends ApiResponse<infer U> ? U : never;

type UserResponse = ApiResponse<User>;
type ExtractedUser = GetData<UserResponse>;  // User
```

## Распределительные условные типы

Когда условный тип применяется к объединению типов, он становится распределительным, то есть применяется к каждому члену объединения:

```typescript
type ToArray<T> = T extends any ? T[] : never;

type T17 = ToArray<string | number>;  // string[] | number[]

// Без распределения результат был бы: (string | number)[]
```

### Контроль распределения

Иногда нужно отключить распределение, для этого можно обернуть тип в кортеж:

```typescript
// Распределение включено
type Distributive<T> = T extends string ? "yes" : "no";
type T18 = Distributive<number | string>;  // "yes" | "no"

// Распределение отключено
type NonDistributive<T> = [T] extends [string] ? "yes" : "no";
type T19 = NonDistributive<number | string>;  // "no"
```

## Сложные паттерны

### 1. Рекурсивные условные типы

```typescript
// Углубленное извлечение типа из вложенных промисов
type Awaited<T> = T extends Promise<infer U> 
  ? Awaited<U> 
  : T;

type T20 = Awaited<Promise<Promise<string>>>;  // string
```

### 2. Условные типы с несколькими условиями

```typescript
// Сложная логика с цепочкой условий
type TypeName<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";

type T21 = TypeName<string>;     // "string"
type T22 = TypeName<() => void>; // "function"
type T23 = TypeName<Date>;      // "object"
```

### 3. Условные типы для работы с массивами

```typescript
// Извлечение типа элемента, учитывая возможные вложенные массивы
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;

type T24 = Flatten<number[][][]>;     // number
type T25 = Flatten<string[][]>;       // string
type T26 = Flatten<boolean>;          // boolean
```

## Встроенные условные типы

TypeScript предоставляет несколько встроенных условных типов:

```typescript
// NonNullable<T> - исключает null и undefined
type T27 = NonNullable<string | null | undefined>;  // string

// Extract<T, U> - извлекает типы из T, которые присутствуют в U
type T28 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"

// Exclude<T, U> - исключает типы из T, которые присутствуют в U
type T29 = Exclude<"a" | "b" | "c", "a" | "f">;  // "b" | "c"

// ReturnType<T> - возвращаемый тип функции
type T30 = ReturnType<() => string>;  // string

// InstanceType<T> - тип экземпляра класса
class MyClass {
  constructor(public name: string) {}
}
type T31 = InstanceType<typeof MyClass>;  // MyClass
```

## Практические примеры из реальной разработки

### 1. Создание типобезопасного события

```typescript
interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  error: Error;
}

type TypedEventHandlers<T extends keyof EventMap> = {
  [K in T]: (data: EventMap[K]) => void
};

// Использование
const handlers: TypedEventHandlers<"user_login" | "error"> = {
  user_login: (data) => console.log(`User ${data.userId} logged in`),
  error: (error) => console.error(error.message)
};
```

### 2. Типобезопасная работа с асинхронными функциями

```typescript
// Извлечение типа возвращаемого значения асинхронной функции
type AsyncReturnType<T extends (...args: any) => any> = 
  T extends (...args: any) => Promise<infer U> 
    ? U 
    : T extends (...args: any) => infer U 
      ? U 
      : any;

async function getUser(id: number): Promise<User> {
  // ...
  return { id, name: "Alice", email: "alice@example.com" };
}

type GetUserResult = AsyncReturnType<typeof getUser>;  // User
```

### 3. Условные типы для работы с опциональными полями

```typescript
// Определение, какие поля обязательны, а какие опциональны
type RequiredKeys<T> = {
  [K in keyof T]-?: T extends Record<K, T[K]> ? K : never
}[keyof T];

type OptionalKeys<T> = {
  [K in keyof T]-?: T extends Record<K, T[K]> ? never : K
}[keyof T];

interface ComplexUser {
  id: number;        // обязательное
  name: string;      // обязательное
  email?: string;    // опциональное
  avatar?: string;   // опциональное
}

type ReqKeys = RequiredKeys<ComplexUser>;  // "id" | "name"
type OptKeys = OptionalKeys<ComplexUser>;  // "email" | "avatar"
```

## Ограничения и лучшие практики

### 1. Производительность

Сложные условные типы могут замедлять компиляцию:

```typescript
// Избегайте чрезмерно сложных вложенных условий
type BadExample<T> = T extends string 
  ? T extends "a" 
    ? "A" 
    : T extends "b" 
      ? "B" 
      : "OTHER" 
  : T extends number 
    ? T extends 0 
      ? "ZERO" 
      : "NUMBER" 
    : "OTHER";
```

### 2. Читаемость

Разбивайте сложные условные типы на несколько шагов:

```typescript
// Сложный для понимания тип
type Complex<T> = T extends Array<infer U> 
  ? U extends object 
    ? { [K in keyof U]: U[K] extends Function ? string : U[K] } 
    : U 
  : T;

// Лучше разбить на несколько типов
type TransformFunction<T> = T extends Function ? string : T;
type TransformObject<T> = { [K in keyof T]: TransformFunction<T[K]> };
type ProcessArray<T> = T extends Array<infer U> 
  ? U extends object 
    ? TransformObject<U> 
    : U 
  : T;
```

### 3. Ограничения infer

`infer` можно использовать только в правой части условного типа:

```typescript
// Правильно
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

// Неправильно - ошибка компиляции
// type BadType<T> = infer R extends T ? R : any;
```

## Лучшие практики

1. **Начинайте с простых условных типов** - постепенно переходите к сложным
2. **Используйте именованные типы** - давайте понятные имена для условных типов
3. **Документируйте сложные типы** - добавляйте комментарии к сложным условным типам
4. **Тестируйте с разными входными типами** - убедитесь, что типы работают корректно
5. **Избегайте излишней сложности** - сложные условные типы могут затруднить понимание кода

## Связи с другими концепциями

- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - часто используются совместно с условными типами
- [[ts/type-system/type-inference|Вывод типов]] - основа для использования infer
- [[ts/generics/TS Generics|Обобщения]] - условные типы часто параметризованы
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - продвинутые паттерны с условными типами
- [[ts/utility-types/index|Утилиты типов]] - многие утилиты реализованы через условные типы
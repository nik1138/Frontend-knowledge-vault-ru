# TypeScript Type System Complete Overview

## Полнейшее руководство по системе типов TypeScript

Система типов TypeScript - это комплексная структура, объединяющая все концепции типов для обеспечения безопасности типов, надежности кода и лучшего опыта разработки.

## Архитектурная карта типовой системы

```
                    ┌─────────────────────────────────────────────────────────┐
                    │              TypeScript Type System                     │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Core Types    │               │      Type Operations     │                                    │  Type Analysis   │
  │                 │               │                          │                                    │                 │
  │ - Primitive     │◄──────────────┤ - Type Inference         │◄───────────────────────────────────┤ - Type          │
  │   Types         │               │ - Narrowing & Widening   │                                    │   Compatibility │
  │ - Object Types  │               │ - Type Guards            │                                    │ - Structural    │
  │ - Function      │               │ - Assertions             │                                    │   Typing        │
  │   Types         │               │ - Control Flow Analysis  │                                    │ - Nominal       │
  │ - Union/        │               │ - Duck Typing            │                                    │   Typing        │
  │   Intersection  │               │ - Contextual Typing      │                                    └─────────────────┘
  │ - Literal Types │               │ - Widening Rules         │
  └─────────────────┘               └─────────────────┬────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │           Advanced Type           │
        │                           │            Operations             │
        │                           │                                 │
        │                           │ - Conditional Types               │
        │                           │ - Mapped Types                    │
        │                           │ - Template Literal Types          │
        │                           │ - Keyof & Lookup Types            │
        │                           │ - Indexed Access Types            │
        │                           │ - Utility Types                   │
        │                           └───────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                           Type Safety                                 │
        │                           │                                                                         │
        │                           │ - Strict Mode & Flags                                                    │
        │                           │ - null/undefined Handling                                                │
        │                           │ - strictNullChecks                                                       │
        │                           │ - noImplicitAny                                                          │
        │                           │ - strictPropertyInitialization                                            │
        └───────────────────────────┤ - strictFunctionTypes                                                     │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Основные компоненты системы типов

### 1. Базовая структура типов
```typescript
// Примитивные типы
let str: string = "hello";
let num: number = 42;
let bool: boolean = true;

// Объектные типы
let obj: { name: string; age: number } = { name: "Alice", age: 30 };

// Функциональные типы
let greet: (name: string) => string;
greet = (name: string) => `Hello, ${name}`;

// Специальные типы
let anyValue: any = "anything";
let unknownValue: unknown = "something";
let neverValue: never = (() => { throw new Error("always throws"); })();
let voidValue: void = undefined;
```

### 2. Система вывода типов
```typescript
// Вывод типов переменных
let inferredString = "hello";     // string
let inferredNumber = 42;          // number
let inferredBoolean = true;       // boolean

// Вывод типов функций
function add(a, b) {              // параметры и возвращаемый тип могут быть выведены
  return a + b;                   // в зависимости от использования
}

// Контекстное определение типов
const items = [
  { name: "Alice", age: 30 },    // тип: { name: string; age: number }[]
  { name: "Bob", age: 25 }
];

// Лучшее общее типовое определение
let mixed = [1, "hello", true];   // (number | string | boolean)[]
```

### 3. Сужение типов (Type Narrowing)
```typescript
// Сужение через typeof
function processValue(value: string | number) {
  if (typeof value === "string") {
    // value: string в этом блоке
    return value.toUpperCase();
  }
  // value: number в этом блоке
  return value.toFixed(2);
}

// Сужение через проверку на null/undefined
function processUser(user: { name: string } | null) {
  if (user) {
    // user: { name: string } в этом блоке
    return user.name.toUpperCase();
  }
  return "Anonymous";
}

// Сужение через instanceof
function formatDate(date: Date | number) {
  if (date instanceof Date) {
    // date: Date в этом блоке
    return date.toISOString();
  }
  // date: number в этом блоке
  return new Date(date).toISOString();
}

// Сужение через in оператор
interface Bird { fly(): void; }
interface Fish { swim(): void; }

function move(pet: Bird | Fish) {
  if ("fly" in pet) {
    // pet: Bird в этом блоке
    pet.fly();
  } else {
    // pet: Fish в этом блоке
    pet.swim();
  }
}
```

### 4. Дополнительные возможности сужения
```typescript
// Тип-предикаты (User-Defined Type Guards)
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function processValue(value: string | number) {
  if (isString(value)) {
    // value: string в этом блоке
    return value.toUpperCase();
  }
  // value: number в этом блоке
  return value.toFixed(2);
}

// Assertion Functions
function assert(condition: boolean, message: string): asserts condition {
  if (!condition) {
    throw new Error(message);
  }
}

function processNumber(value: number | string) {
  assert(typeof value === "number", "Value must be a number");
  // После assert, TypeScript знает, что value - number
  return value.toFixed(2);
}

// Доказательства (Proofs)
function assertString(value: unknown): asserts value is string {
  if (typeof value !== "string") {
    throw new Error("Value must be a string");
  }
}

function processValue2(value: unknown) {
  assertString(value);
  // После assert, TypeScript знает, что value - string
  return value.toUpperCase();
}
```

### 5. Продвинутая система типов
```typescript
// Условные типы
type TypeName<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<true>;    // "boolean"
type T2 = TypeName<() => void>;  // "object"

// Сопоставляющие типы (Mapped Types)
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Writable<T> = {
  -readonly [P in keyof T]: T[P];
};

// Шаблонные литеральные типы
type EventName = "click" | "scroll" | "mousemove";
type HandlerName = { [K in EventName as `on${Capitalize<K>}`]: (e: Event) => void };

type S = HandlerName;
// {
//   onclick: (e: Event) => void;
//   onscroll: (e: Event) => void;
//   onmousemove: (e: Event) => void;
// }

type FieldName = "name" | "age";
type ValidationRules = { [K in FieldName as `${K}Validator`]: (value: any) => boolean };
```

## Совместимость типов

### Структурное типирование
```typescript
// TypeScript использует структурное типирование
interface Named {
  name: string;
}

class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

let n: Named;
n = new Person("Alice"); // OK, структуры совпадают

// Совместимость функций
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

// y может быть присвоена x, т.к. x игнорирует лишние параметры
x = y; // OK

// x НЕ может быть присвоена y, т.к. y требует больше параметров
// y = x; // ОШИБКА

// Совместимость возвращаемых значений
let f = () => ({ name: "Alice" });
let g = () => ({ name: "Alice", age: 30 });

// f может быть присвоена g, т.к. { name: string } является подтипом { name: string, age: number }
g = f; // OK

// g НЕ может быть присвоена f, т.к. { name: string, age: number } не может быть использован как { name: string }
// f = g; // ОШИБКА
```

### Контрвариантность и ковариантность
```typescript
// Пример контравариантности параметров функций
interface Animal { name: string; }
interface Dog extends Animal { breed: string; }

let animalConsumer: (animal: Animal) => void;
let dogConsumer: (dog: Dog) => void;

// dogConsumer может принимать только Dog, но потребителя животных можно передать Dog
// Поэтому он может быть присвоен более общему потребителю
animalConsumer = dogConsumer; // OK

// animalConsumer может принимать любое животное, включая Dog
// Но dogConsumer может не работать с другими животными
// dogConsumer = animalConsumer; // ОШИБКА в зависимости от настроек

// Ковариантность возвращаемых значений
let getAnimal: () => Animal;
let getDog: () => Dog;

// getDog возвращает Dog, который является Animal
// Поэтому его можно присвоить более общему типу
getAnimal = getDog; // OK

// getAnimal может вернуть любое животное, не обязательно Dog
// getDog = getAnimal; // ОШИБКА
```

## Практические паттерны системы типов

### 1. Дискриминированные объединения
```typescript
interface LoadingState {
  status: "loading";
  progress?: number;
}

interface SuccessState {
  status: "success";
  data: string;
}

interface ErrorState {
  status: "error";
  error: string;
}

type State = LoadingState | SuccessState | ErrorState;

function handleState(state: State) {
  switch (state.status) {
    case "loading":
      // state: LoadingState
      console.log(`Loading... ${state.progress ?? 0}%`);
      break;
    case "success":
      // state: SuccessState
      console.log(`Success: ${state.data}`);
      break;
    case "error":
      // state: ErrorState
      console.log(`Error: ${state.error}`);
      break;
  }
}
```

### 2. Утилиты системы типов
```typescript
// Создание собственных утилит
type NonNullable<T> = T extends null | undefined ? never : T;

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

type Extract<T, U> = T extends U ? T : never;

type Exclude<T, U> = T extends U ? never : T;

// Примеры использования
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

type PublicUser = Pick<User, "id" | "name" | "email">;
// { id: number; name: string; email: string; }

type PrivateUser = Omit<User, "password">;
// { id: number; name: string; email: string; createdAt: Date; updatedAt: Date; }

type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];

type UserStringKeys = StringKeys<User>; // "name" | "email" | "password"
```

### 3. Преобразование типов
```typescript
// Глубокие преобразования
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object 
    ? T[P] extends Function 
      ? T[P] 
      : DeepPartial<T[P]> 
    : T[P]
};

type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object 
    ? T[P] extends Function 
      ? T[P] 
      : DeepReadonly<T[P]> 
    : T[P]
};

interface NestedConfig {
  api: {
    url: string;
    timeout: number;
    auth: {
      username: string;
      password: string;
    };
  };
  ui: {
    theme: {
      primary: string;
      secondary: string;
    };
    lang: string;
  };
}

type PartialConfig = DeepPartial<NestedConfig>;
// Все свойства становятся необязательными рекурсивно

type ReadonlyConfig = DeepReadonly<NestedConfig>;
// Все свойства становятся readonly рекурсивно
```

## Настройки системы типов

### Строгие флаги
```typescript
// В tsconfig.json:
{
  "compilerOptions": {
    "strict": true,                    // Включает все строгие проверки
    "strictNullChecks": true,          // null и undefined явно отличаются от других типов
    "noImplicitAny": true,             // Запрещает вывод типа any
    "strictFunctionTypes": true,       // Более строгая проверка типов функций
    "strictBindCallApply": true,       // Проверка типов для bind/call/apply
    "strictPropertyInitialization": true, // Проверка инициализации свойств
    "noImplicitThis": true,            // Явная проверка использования this
    "useUnknownInCatchVariables": true // catch переменные как unknown
  }
}
```

### Строгая проверка NULL
```typescript
// Без strictNullChecks (не рекомендуется)
let str: string = "hello";
str = null; // OK (плохо!)

// С strictNullChecks (рекомендуется)
let strDefinite: string | null = "hello";
strDefinite = null; // OK
// strDefinite = undefined; // OK если не strictNullChecks или если string | undefined

function processString(str: string | null) {
  // if (str) { // проверка на truthiness
  if (str !== null && str !== undefined) { // явная проверка
    // str: string в этом блоке
    return str.toUpperCase();
  }
  return "";
}
```

## Сравнение с другими системами типов

### Nominal vs Structural Typing
```typescript
// TypeScript использует структурное типирование
interface Duck {
  quack(): void;
  swim(): void;
}

interface Dog {
  bark(): void;
  swim(): void;
}

// В номинальной системе типов эти интерфейсы были бы разными
// В структурной системе TypeScript они могут быть совместимы
// если один содержит все свойства другого

class Animal {
  swim(): void { }
}

class DuckImpl extends Animal implements Duck {
  quack(): void { }
  // swim наследуется от Animal
}

// Это возможно благодаря структурному типированию
let duck: Duck = new DuckImpl(); // OK
```

### Совместимость с JavaScript
```typescript
// TypeScript должен быть совместим с JavaScript
let jsArray = [1, 2, 3]; // Тип: number[]
// JavaScript: [1, 2, 3, "mixed"] - разрешено в JS
// TypeScript: защищает нас от этого

// Совместимость с динамическими данными
function processJson(data: any) {
  // any отключает проверки типов
  return data.property.subProperty.value;
  // Могут быть ошибки выполнения
}

function processJsonSafe(data: unknown) {
  // unknown требует проверок
  if (typeof data === "object" && data !== null && "property" in data) {
    // Проверка структуры
    const obj = data as { property: unknown };
    if (typeof obj.property === "object" && obj.property !== null && "subProperty" in obj.property) {
      // Дальнейшая проверка...
      return (obj.property as { subProperty: { value: string } }).subProperty.value;
    }
  }
  throw new Error("Invalid data structure");
}
```

## Лучшие практики системы типов

### 1. Баланс между безопасностью и гибкостью
```typescript
// Слишком широкий тип
function badExample(data: any): any {
  return data.process(); // Нет безопасности типов
}

// Слишком узкий тип
function tooNarrow(data: { name: string; age: number }) {
  // Работает только с этими двумя полями
}

// Хороший баланс
type Processable = { name: string } & Record<string, any>;
function goodExample(data: Processable) {
  return data.name.toUpperCase();
}
```

### 2. Использование строгих проверок
```typescript
// Включите строгие флаги
interface Config {
  apiUrl: string;
  timeout: number;
}

function createRequest(config: Config) {
  if (!config.apiUrl) { // Проверка на пустые значения
    throw new Error("Base URL is required");
  }
  
  // Использование с уверенностью в типах
  return fetch(config.apiUrl, { timeout: config.timeout });
}
```

### 3. Документирование типов
```typescript
/**
 * Результат операции
 * @template T Тип успешного результата
 */
type Result<T> = 
  | { success: true; data: T }
  | { success: false; error: Error };

/**
 * Асинхронная операция с типобезопасным результатом
 */
async function safeExecute<T>(operation: () => Promise<T>): Promise<Result<T>> {
  try {
    const data = await operation();
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}
```

## Связь с другими концепциями TypeScript
- [[types]] - базовая система типов
- [[TypeScript Advanced Types]] - продвинутые возможности
- [[TS Generics]] - обобщенные типы
- [[TS Interfaces and Classes]] - объектно-ориентированные типы
- [[TypeScript Modules]] - типизация модулей
- [[Functional Programming in TS]] - функциональная типизация
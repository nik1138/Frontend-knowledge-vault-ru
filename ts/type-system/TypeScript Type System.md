# TypeScript Type System

Система типов TypeScript - это комплексная архитектура, обеспечивающая статическую типизацию для JavaScript. Она включает в себя структурную типизацию, вывод типов, совместимость типов и продвинутые возможности для моделирования сложных доменных областей.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        TypeScript Type System                           │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │  Core Types     │               │     Type Relations       │                                    │  Advanced      │
  │                 │               │                          │                                    │  Features      │
  │ - Primitives    │◄──────────────┤ - Type Inference         │─────────────────────────────────────►│ - Conditional   │
  │ - Objects       │               │ - Type Compatibility     │                                    │   Types         │
  │ - Functions     │               │ - Structural Typing      │                                    │ - Mapped        │
  │ - Arrays        │               │ - Duck Typing            │                                    │   Types         │
  │ - Tuples        │               │ - Type Guards            │                                    │ - Template      │
  │ - Literal Types │               │ - Narrowing/Widening     │                                    │   Literals      │
  │ - Union/        │               │ - Type Assertions        │                                    │ - Advanced      │
  │   Intersection  │               │ - Compatibility Rules    │                                    │   Utilities     │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │        Type Safety            │
        │                           │                               │
        │                           │ - strictNullChecks            │
        │                           │ - noImplicitAny               │
        │                           │ - strictFunctionTypes         │
        │                           │ - strictBindCallApply         │
        │                           │ - strictPropertyInitialization │
        │                           │ - noImplicitReturns           │
        │                           │ - noUncheckedIndexedAccess    │
        │                           │ - strict mode flags           │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                        Type Analysis                              │
        │                           │                                                                 │
        │                           │ - Control Flow Analysis                                             │
        │                           │ - Type Guards & Predicates                                          │
        │                           │ - Exhaustiveness Checking                                           │
        │                           │ - Discriminant Properties                                           │
        │                           │ - Narrowing with typeof/instanceof                                  │
        │                           │ - Type Widening/Rewinding                                           │
        └───────────────────────────┤ - Liskov Substitution Principle                                     │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Основные компоненты системы типов

### 1. Примитивные типы
```typescript
// Базовые примитивные типы
let booleanValue: boolean = true;
let numberValue: number = 42;
let stringValue: string = "hello";
let symbolValue: symbol = Symbol("unique");
let bigintValue: bigint = 123456789012345678901234567890n;
let undefinedValue: undefined = undefined;
let nullValue: null = null;

// Типы, производные от примитивов
let objectType: object = { name: "Alice" };
let anyType: any = "any value";
let unknownType: unknown = "unknown value";
let neverType: never; // никогда не присваивается
let voidType: void = undefined; // обычно для возвращаемого значения
```

### 2. Объектные типы
```typescript
// Интерфейсы
interface User {
  name: string;
  age: number;
  email?: string; // необязательное свойство
  readonly id: number; // свойство только для чтения
}

// Типы-объединения (Type Aliases)
type UserProfile = {
  name: string;
  preferences: {
    theme: string;
    notifications: boolean;
  };
};

// Классы
class Person {
  constructor(public name: string, public age: number) {}
  
  greet(): string {
    return `Hello, my name is ${this.name}`;
  }
}

// Ключевые особенности объектных типов
const user: User = {
  name: "Alice",
  age: 30,
  id: 123
  // email опционально, поэтому может быть пропущено
};

// user.id = 456; // ОШИБКА: id только для чтения
```

### 3. Функциональные типы
```typescript
// Типы функций
type GreetingFunction = (name: string) => string;

const greet: GreetingFunction = (name: string): string => {
  return `Hello, ${name}!`;
};

// Функции с несколькими параметрами
type CalculatorFunction = (a: number, b: number) => number;

const add: CalculatorFunction = (a: number, b: number): number => a + b;
const multiply: CalculatorFunction = (a: number, b: number): number => a * b;

// Функции с callback'ами
type Processor<T> = (item: T, callback: (result: T) => void) => void;

const processItem: Processor<string> = (item, callback) => {
  const processed = item.toUpperCase();
  callback(processed);
};
```

## Правила совместимости типов

### Структурная типизация
```typescript
// TypeScript использует структурную (duck) типизацию
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
n = new Person("Alice"); // OK: структура совпадает

// Совместимость функций
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

// y может быть присвоена x, потому что x игнорирует лишние параметры
x = y; // OK
// y = x; // ОШИБКА: x не предоставляет все параметры, которые ожидает y

// Совместимость возвращаемых значений (ковариантность)
let f = () => ({ name: "Alice" });
let g = () => ({ name: "Alice", age: 30 });

// g может быть присвоена f, потому что { name: string, age: number } может использоваться как { name: string }
f = g; // OK
// Обратное не всегда безопасно:
// g = f; // ОШИБКА: { name: string } не обязательно имеет age
```

### Совместимость объединений
```typescript
// Совместимость объединений типов
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

// Доступ к общим свойствам объединения
function getSmallPet(): Fish | Bird {
  // случайное создание питомца
  return Math.random() > 0.5 ? { fly() {}, layEggs() {} } : { swim() {}, layEggs() {} };
}

let pet = getSmallPet();
pet.layEggs(); // OK: оба типа имеют layEggs

// pet.swim(); // ОШИБКА: не все типы в объединении имеют swim
// pet.fly();  // ОШИБКА: не все типы в объединении имеют fly
```

## Вывод типов (Type Inference)

### Базовый вывод типов
```typescript
// Вывод типов переменных
let x = 3;        // тип number
let y = "hello";  // тип string
let z = true;     // тип boolean

// Вывод типов в функциях
function add(a, b) {  // параметры: any, возвращаемое: any
  return a + b;
}

// Явная типизация для лучшей безопасности
function typedAdd(a: number, b: number): number {
  return a + b;
}

// Вывод типов возвращаемого значения
function getRandomValue() {
  if (Math.random() > 0.5) {
    return "string";
  }
  return 42;
}
// Тип возвращаемого значения: string | number
```

### Контекстный вывод типов
```typescript
// Контекстный вывод для функций обратного вызова
const names = ["Alice", "Bob", "Charlie"];

// Тип параметра "s" выводится из типа массива names
names.forEach(function (s) {
  console.log(s.toUppercase()); // ОШИБКА: toUppercase не является методом string
});

// Правильное использование:
names.forEach(function (s) {
  console.log(s.toUpperCase()); // OK: тип s — string
});

// Контекстный вывод для объектов
const person = {
  name: "Alice",    // тип: string
  age: 30,         // тип: number
  isActive: true    // тип: boolean
};

// Контекстный вывод для функций
const processUser = (user: { name: string; age: number }) => {
  // Тип параметра user определен структурой
  return user.name.toUpperCase();
};
```

## Совместимость функций

### Параметры функций
```typescript
// Контравариантность параметров
interface Empty {}
let empty: Empty = {};
interface NotEmpty { name: string }
let notEmpty: NotEmpty = { name: "Alice" }

// Контравариантность: функция, ожидающая меньшего, может принимать большее
let x: (arg: Empty) => void;
let y: (arg: NotEmpty) => void;

x = y; // OK: x может принять все, что может y, и больше
// y = x; // ОШИБКА: x может получить Empty в месте, где y ожидает NotEmpty

// Пример с более сложными функциями
type ComparisonFunction = (a: number, b: number) => boolean;

function ascending(a: number, b: number): boolean {
  return a < b;
}

function descending(a: number, b: number): boolean {
  return a > b;
}

// Все эти функции совместимы с ComparisonFunction
const compare: ComparisonFunction = ascending;
const sortAsc = (fn: ComparisonFunction) => (a: number, b: number) => fn(a, b);
```

### Возвращаемые значения
```typescript
// Ковариантность возвращаемых значений
type Getter<T> = () => T;

let g1: Getter<Empty>;
let g2: Getter<NotEmpty>;

// g2 можно присвоить g1, потому что NotEmpty может использоваться как Empty
g1 = g2; // OK
// g2 = g1; // ОШИБКА: Empty не всегда может использоваться как NotEmpty
```

## Сужение типов (Type Narrowing)

### Проверки типов во время выполнения
```typescript
// Проверка typeof
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    // В этой области TypeScript знает, что padding является number
    return " ".repeat(padding) + value;
  }
  // В этой области TypeScript знает, что padding является string
  return padding + value;
}

// Проверка instanceof
interface Padder {
  getPaddingString(): string;
}

class SpaceRepeatingPadder implements Padder {
  constructor(private numSpaces: number) {}
  getPaddingString() {
    return " ".repeat(this.numSpaces);
  }
}

class StringPadder implements Padder {
  constructor(private value: string) {}
  getPaddingString() {
    return this.value;
  }
}

function greetPadder(padder: Padder) {
  if (padder instanceof SpaceRepeatingPadder) {
    // Здесь TypeScript знает, что padder - SpaceRepeatingPadder
    console.log("Space padding:", padder.getPaddingString());
  } else if (padder instanceof StringPadder) {
    // Здесь TypeScript знает, что padder - StringPadder
    console.log("String padding:", padder.getPaddingString());
  }
}
```

### in оператор
```typescript
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

function move(pet: Fish | Bird): void {
  if ("swim" in pet) {
    // pet имеет тип Fish в этой ветке
    pet.swim();
  } else {
    // pet имеет тип Bird в этой ветке
    pet.fly();
  }
}
```

### Утверждения типов и защитники типов
```typescript
// Type predicate functions
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

// Типоохранитель (type guard)
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function isNumber(value: unknown): value is number {
  return typeof value === "number";
}

function processValue(value: unknown) {
  if (isString(value)) {
    // В этой области TypeScript знает, что value - string
    return value.toUpperCase(); // OK: string метод
  } else if (isNumber(value)) {
    // В этой области TypeScript знает, что value - number
    return value.toFixed(2); // OK: number метод
  }
  return "unknown type";
}

// Использование
const pet: Fish | Bird = getSmallPet();

if (isFish(pet)) {
  pet.swim(); // OK: TypeScript знает, что это Fish
} else {
  pet.fly(); // OK: TypeScript знает, что это Bird
}
```

## Продвинутые возможности системы типов

### Условные типы
```typescript
// Условные типы: T extends U ? X : Y
type TypeName<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  T extends Function ? "function" :
  "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<"a">;     // "string" (литеральный тип)
type T2 = TypeName<() => void>;  // "function"
type T3 = TypeName<string[]>;    // "object" (не "array")
```

### Сопоставляющие типы
```typescript
// Сопоставляющие типы: создание новых типов из существующих
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Примеры использования
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;
// { title: string; completed: boolean; }

type TodoPartial = Partial<Todo>;
// { title?: string; description?: string; completed?: boolean; }

type TodoReadonly = Readonly<Todo>;
// { readonly title: string; readonly description: string; readonly completed: boolean; }
```

### Обобщенные типы
```typescript
// Обобщенные функции
function identity<T>(arg: T): T {
  return arg;
}

// Использование
let output1 = identity<string>("hello");    // string
let output2 = identity("hello");            // string (вывод типа)

// Обобщенные классы
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = (x, y) => x + y;
```

### Ограничения обобщений (Generic Constraints)
```typescript
// Базовое ограничение
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // OK: теперь мы знаем, что у arg есть свойство length
  return arg;
}

// loggingIdentity(3); // ОШИБКА: number не имеет length
loggingIdentity({ length: 10, value: 3 }); // OK

// Ограничение с ключами
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };
getProperty(x, "a"); // OK
// getProperty(x, "m"); // ОШИБКА: "m" не является ключом x
```

## Практические примеры

### Система типов для API
```typescript
// Типобезопасная система API
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  timestamp: number;
}

type ApiSuccess<T> = {
  success: true;
  data: T;
  timestamp: number;
};

type ApiError = {
  success: false;
  error: string;
  timestamp: number;
};

type ApiResponseImproved<T> = ApiSuccess<T> | ApiError;

// Функция с выводом типов
async function apiCall<T>(url: string): Promise<ApiResponseImproved<T>> {
  try {
    const response = await fetch(url);
    const data = await response.json();
    
    if (response.ok) {
      return { success: true as const, data, timestamp: Date.now() };
    } else {
      return { success: false as const, error: data.message, timestamp: Date.now() };
    }
  } catch (error) {
    return { success: false as const, error: (error as Error).message, timestamp: Date.now() };
  }
}

// Использование
type User = { id: number; name: string };
const userResponse = await apiCall<User>("/api/user/123");

if (userResponse.success) {
  // TypeScript знает, что в этой ветке есть data
  console.log(userResponse.data.name); // OK
} else {
  // TypeScript знает, что в этой ветке есть error
  console.error(userResponse.error); // OK
}
```

### Дискриминированные объединения
```typescript
// Дискриминированные объединения для безопасной работы с разными состояниями
interface LoadingState {
  status: "loading";
  progress?: number;
}

interface SuccessState<T> {
  status: "success";
  data: T;
}

interface ErrorState {
  status: "error";
  error: string;
}

type AppState<T> = LoadingState | SuccessState<T> | ErrorState;

function handleAppState<T>(state: AppState<T>) {
  switch (state.status) {
    case "loading":
      // state автоматически имеет тип LoadingState
      const progress = state.progress || 0;
      console.log(`Loading... ${progress}%`);
      break;
      
    case "success":
      // state автоматически имеет тип SuccessState<T>
      console.log("Data:", state.data);
      break;
      
    case "error":
      // state автоматически имеет тип ErrorState
      console.error("Error:", state.error);
      break;
      
    default:
      // Если все возможные варианты учтены, 
      // этот код никогда не выполнится
      // TypeScript может проверить исчерпание
      const _exhaustive: never = state;
      throw new Error(`Unhandled state: ${_exhaustive}`);
  }
}
```

### Типы-утилиты
```typescript
// Определение собственных утилит
type Nullable<T> = T | null | undefined;

type NonNullable<T> = T & {};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

type ReadOnly<T> = {
  readonly [P in keyof T]: T[P];
};

// Комбинирование утилит
type OptionalReadOnly<T> = ReadOnly<Partial<T>>;

interface User {
  id: number;
  name: string;
  email: string;
  preferences?: {
    theme: string;
    notifications: boolean;
  };
}

type OptionalUserFields = Partial<User>;
type ReadOnlyUser = ReadOnly<User>;
type OptionalReadOnlyUser = OptionalReadOnly<User>;
```

## Строгая типизация

### Strict Mode Flags
```json
{
  "compilerOptions": {
    "strict": true,                    // Включает все строгие проверки
    "strictNullChecks": true,          // null и undefined не совместимы с другими типами
    "noImplicitAny": true,             // Запрещает вывод типа any
    "strictFunctionTypes": true,       // Более строгая проверка функций
    "strictBindCallApply": true,       // Проверка типов для bind/call/apply
    "strictPropertyInitialization": true, // Проверка инициализации свойств
    "noImplicitThis": true,            // Явная проверка использования this
    "useUnknownInCatchVariables": true // Catch переменные как unknown
  }
}
```

### Практическое применение строгой типизации
```typescript
// Строгая проверка null/undefined
function getLength(str: string | null | undefined): number {
  // return str.length; // ОШИБКА с strictNullChecks
  if (str != null) { // проверка на null и undefined
    return str.length;
  }
  return 0;
}

// Строгая проверка this
class StrictClass {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  
  // Неявное использование this теперь проверяется
  getName(): string {
    return this.name; // OK: this.name определен
  }
  
  brokenGetName(): string {
    const fn = () => this.name; // OK: стрелочная функция захватывает this
    return fn();
  }
}
```

## Лучшие практики

### 1. Явная типизация для API
```typescript
// Хорошо: явная типизация для публичных API
function calculateTotal(
  items: { price: number; quantity: number }[],
  taxRate: number
): number {
  return items.reduce(
    (total, item) => total + item.price * item.quantity,
    0
  ) * (1 + taxRate);
}

// Плохо: неясные типы
// function calculateTotal(items, taxRate) { return items.reduce(...) * (1 + taxRate); }
```

### 2. Понятные имена типов
```typescript
// Понятные имена для объединений
type Status = 'active' | 'inactive' | 'pending';
type UserRole = 'admin' | 'user' | 'guest';

// Понятные имена для сложных типов
type UserPermissions = {
  canRead: boolean;
  canWrite: boolean;
  canDelete: boolean;
};
```

### 3. Использование интерфейсов vs типов-объединений
```typescript
// Используйте интерфейсы для расширения
interface User {
  name: string;
  email: string;
}

interface AdminUser extends User {
  permissions: string[];
}

// Используйте типы-объединения для комбинаций
type HttpResponse = SuccessResponse | ErrorResponse;
type InputElement = HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement;
```

### 4. Типобезопасные enum'ы
```typescript
// Использование строковых литеральных типов вместо enum'ов для большей гибкости
type Direction = "up" | "down" | "left" | "right";

// Вместо числовых enum'ов
// enum Direction { Up, Down, Left, Right }
```

## Ошибки и ловушки

### Частые ошибки новичков
```typescript
// 1. Предположение, что типы влияют на выполнение
let value: number | string = "hello";
// typeof value в JavaScript всё равно возвращает runtime-тип "string"
// TypeScript проверяет типы только во время компиляции

// 2. Использование any вместо правильной типизации
function processData(data: any): any { // ПЛОХО
  return data.process();
}

// Лучше: конкретные типы или unknown
function processDataBetter<T extends { process(): any }>(data: T): ReturnType<T['process']> {
  return data.process();
}

// 3. Непонимание структурной типизации
interface Named {
  name: string;
}

interface Person {
  name: string;
  age: number;
}

const person: Person = { name: "Alice", age: 30 };
const named: Named = person; // OK: Person структурно совместим с Named
// const specific: Person = named; // ОШИБКА: Named не содержит age
```

### Комплексные типы
```typescript
// Избегайте чрезмерно сложных типов
// ПЛОХО: трудно читать и понимать
type OverlyComplex<T> = T extends Array<infer U> 
  ? U extends object 
    ? U extends Function 
      ? never 
      : { [K in keyof U as string extends K ? never : K]: U[K] }
    : T
  : T;

// ЛУЧШЕ: разбить на более простые типы-утилиты
type ObjectProperties<T> = Pick<T, { [K in keyof T]: T[K] extends Function ? never : K }[keyof T]>;
type NonFunctionArray<T> = T extends Array<infer U> ? U extends Function ? never : U[] : T;
```

## Современные тенденции

### Type-First Development
```typescript
// Разработка сначала типов, потом реализации
interface UserApi {
  getUser(id: number): Promise<User>;
  createUser(userData: CreateUserRequest): Promise<User>;
  updateUser(id: number, updateData: UpdateUserRequest): Promise<User>;
}

interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

interface UpdateUserRequest {
  name?: string;
  email?: string;
}

// Реализация после определения типов
class UserService implements UserApi {
  async getUser(id: number): Promise<User> {
    // реализация
    return {} as User;
  }
  
  async createUser(userData: CreateUserRequest): Promise<User> {
    // реализация
    return {} as User;
  }
  
  async updateUser(id: number, updateData: UpdateUserRequest): Promise<User> {
    // реализация
    return {} as User;
  }
}
```

### Type-Level Programming
```typescript
// Программирование на уровне типов
type Concat<T extends any[], U extends any[]> = [...T, ...U];

type T = Concat<[1, 2], [3, 4]>; // [1, 2, 3, 4]

// Утилиты для работы со строками на уровне типов
type Join<T extends string[], D extends string> = 
  T extends [infer F, ...infer R] 
    ? R extends [] 
      ? F 
      : F extends string 
        ? R extends string[] 
          ? `${F}${D}${Join<R, D>}` 
          : never 
        : never 
    : '';

type Path = Join<['users', 'profile', 'settings'], '/'>; // "users/profile/settings"
```

## Рекомендации по использованию

| Сценарий | Рекомендуемый подход |
|----------|---------------------|
| Публичный API | Явная типизация всех параметров и возвратов |
| Внутренние утилиты | Вывод типов, но с понятными интерфейсами |
| Работа с внешними библиотеками | Определения типов или type assertions |
| Обработка данных | Дискриминированные объединения и type guards |
| Функциональное программирование | Обобщения и условные типы |
| Объектно-ориентированное программирование | Интерфейсы и наследование типов |

## Связь с другими концепциями
- [[../types/index]] - базовая система типов
- [[../generics/index]] - обобщенное программирование
- [[Advanced TypeScript Types]] - продвинутые возможности типа
- [[../interfaces-classes/index]] - объектно-ориентированная типизация
- [[../functional-programming/index]] - функциональная типизация
- [[../modules/index]] - типизация модулей
- [[../ecosystem/tools]] - инструменты для работы с типами
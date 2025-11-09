# Type Inference

Вывод типов (Type Inference) в TypeScript - это способность компилятора автоматически определять типы переменных, параметров функций, возвращаемых значений и других выражений без явного указания аннотаций типов.

## Основные принципы вывода типов

### 1. Вывод при инициализации
```typescript
// TypeScript автоматически выводит типы из инициализирующих значений
let x = 3;           // тип number
let y = "hello";     // тип string  
let z = true;        // тип boolean

const obj = {
  name: "Alice",     // name: string
  age: 30,          // age: number
  active: true      // active: boolean
};                 // тип: { name: string; age: number; active: boolean; }

let items = [1, 2, 3]; // тип: number[]
// Если массив пустой, выводится: never[] (если не strictNullChecks: false, то any[])
```

### 2. Контекстный вывод (Contextual Typing)
```typescript
// TypeScript использует окружающий контекст для вывода типов
window.onmousedown = function(mouseEvent) {
  // mouseEvent автоматически выводится как MouseEvent
  console.log(mouseEvent.button);  // OK
  // console.log(mouseEvent.kangaroo); // ОШИБКА: нет такого свойства
};

// Контекстный вывод для обработчиков событий
document.addEventListener("click", (event) => {
  // event автоматически выводится как Event
  console.log(event.type); // OK
});

// Контекстный вывод для callback функций
const numbers = [1, 2, 3, 4, 5];
const evens = numbers.filter(n => n % 2 === 0);
// n автоматически выводится как number
// evens выводится как number[]

// Контекстный вывод для функций
interface FunctionConfig {
  callback: (value: string) => boolean;
}

const config: FunctionConfig = {
  callback: (value) => {  // value автоматически string
    return value.length > 0;
  }
};
```

## Вывод в функциях

### Возвращаемые типы
```typescript
// Вывод возвращаемого типа
function add(a: number, b: number) {
  return a + b;  // возвращаемый тип: number (выведен из a + b)
}

function greet(name: string) {
  return `Hello, ${name}!`;  // возвращаемый тип: string
}

function processItems(items: string[]) {
  return items.map(item => item.toUpperCase());  // возвращаемый тип: string[]
}

// Вывод типов в стрелочных функциях
const multiply = (a: number, b: number) => a * b;
// возвращаемый тип: number (выведен из a * b)

// Вывод для асинхронных функций
async function fetchUserData(id: number) {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return data;  // возвращаемый тип: Promise<unknown> (в зависимости от response.json())
}
```

### Вывод параметров в callback'ах
```typescript
// Вывод параметров в функциях высшего порядка
function processWithCallback<T, U>(
  items: T[],
  callback: (item: T, index: number) => U
): U[] {
  return items.map(callback);
}

const strings = ["hello", "world", "typescript"];
const lengths = processWithCallback(strings, (str, index) => {
  // str выводится как string (из strings)
  // index выводится как number
  return `${str}-${index}`; // тип возвращаемого значения: string
});
// lengths имеет тип: string[]

// Вывод через Promise
function executeAfterDelay<T>(value: T, delay: number): Promise<T> {
  return new Promise(resolve => {
    setTimeout(() => resolve(value), delay);
  });
}

const result = executeAfterDelay("hello", 1000);
// result имеет тип: Promise<string> (T выведено из "hello")
```

## Вывод в классах

### Свойства и методы
```typescript
class Rectangle {
  // Вывод типов для свойств из конструктора
  constructor(
    public width: number,    // width: number
    public height: number,   // height: number
    private color = "white" // color: string (default value)
  ) {}
  
  // Вывод возвращаемого типа
  getArea() {
    return this.width * this.height; // number
  }
  
  // Вывод возвращаемого типа для вычисляемых свойств
  get description() {
    return `Rectangle ${this.width}x${this.height} (${this.color})`; // string
  }
  
  // Вывод типов параметров методов
  setSize(width: number, height: number) {
    this.width = width;
    this.height = height;
    // возвращаемый тип: void
  }
}

const rect = new Rectangle(10, 20);
// rect тип: Rectangle
// rect.width тип: number
// rect.height тип: number
// rect.getArea() тип: number
```

## Вывод с обобщениями

### Вывод типов параметров
```typescript
// Вывод generic параметров
function identity<T>(arg: T): T {
  return arg;
}

// T выводится из аргумента
let output1 = identity("hello");    // T = string, output1 = string
let output2 = identity(42);         // T = number, output2 = number
let output3 = identity({x: 1});     // T = {x: number}, output3 = {x: number}

// Вывод с несколькими параметрами
function mapObject<T, U>(
  obj: T,
  mapper: (value: any, key: keyof T) => U
): {[K in keyof T]: U} {
  const result = {} as {[K in keyof T]: U};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      result[key] = mapper(obj[key], key);
    }
  }
  return result;
}

const original = { a: 1, b: 2, c: 3 };
const doubled = mapObject(original, (value) => value * 2);
// doubled: { a: number; b: number; c: number }
// value выводится как number из оригинального объекта
```

### Вывод с ограничениями
```typescript
interface Lengthwise {
  length: number;
}

function getProperty<T extends Lengthwise, K extends keyof T>(obj: T, key: K): T[K] {
  console.log(obj.length); // OK: T гарантированно имеет length
  return obj[key];
}

const lengthyObject = { length: 10, name: "Alice" };
const nameProperty = getProperty(lengthyObject, "name"); 
// T выводится как { length: number; name: string }
// K выводится как "name"
// Возвращаемый тип: string (тип свойства "name")

// Вывод при работе с массивами
function findElement<T>(
  array: T[],
  predicate: (element: T, index: number, array: T[]) => boolean
): T | undefined {
  return array.find(predicate);
}

const numbers = [1, 2, 3, 4, 5];
const found = findElement(numbers, (num, idx) => num > 3);
// T выводится как number из numbers
// found имеет тип number | undefined
```

## Вывод с объединениями

### Вывод из объединений
```typescript
// Вывод при работе с объединениями
function processValue(value: string | number) {
  if (typeof value === "string") {
    // Здесь value имеет тип string (сужение)
    return value.toUpperCase(); // OK: string метод
  } else {
    // Здесь value имеет тип number (сужение)
    return value.toFixed(2); // OK: number метод
  }
  // Возвращаемый тип: string (объединение: string | string => string)
}

// Вывод из условных выражений
function getValue(condition: boolean) {
  if (condition) {
    return "hello"; // string
  } else {
    return 42;      // number
  }
  // Возвращаемый тип: string | number (объединение всех возможных возвратов)
}

// Вывод в логических выражениях
const result = Math.random() > 0.5 ? "success" : 0;
// Тип: string | number (объединение типов обеих ветвей)
```

## Вывод в сложных структурах

### Вывод возвращаемого типа объектов
```typescript
// Вывод типов для создаваемых объектов
function createUser(name: string, email: string) {
  return {
    id: Math.random(),           // number
    name,                        // string (from parameter)
    email,                       // string (from parameter)
    createdAt: new Date(),      // Date
    isActive: true,             // boolean
    profile: {                  // nested object
      bio: "",
      avatar: null
    }
  };
  // Возвращаемый тип: {
  //   id: number;
  //   name: string;
  //   email: string;
  //   createdAt: Date;
  //   isActive: boolean;
  //   profile: { bio: string; avatar: null; };
  // }
}

// Вывод для функции, возвращающей функцию
function createAdder(x: number) {
  return function(y: number) {  // y: number (из параметра внутренней функции)
    return x + y;               // number (выведен из x + y)
  };
  // Тип возвращаемой функции: (y: number) => number
}

const addFive = createAdder(5);
// addFive: (y: number) => number

const result = addFive(3);
// result: number
```

### Вывод в маппингах и сопоставлениях
```typescript
// Вывод при сопоставлении типов
type KeysMatching<T, V> = {
  [K in keyof T]: T[K] extends V ? K : never
}[keyof T];

// Вывод из маппинга
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
  isActive: boolean;
}

type StringKeys = KeysMatching<User, string>;
// Вывод: "name" | "email" (ключи, чьи значения являются string)

// Вывод при использовании встроенных утилит
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; ... }

type RequiredUser = Required<PartialUser>;
// Возвращается к исходному User (если все поля были опциональны)
```

## Контрольный вывод типов

### Лучшее общее типовое определение (Best Common Type)
```typescript
// Когда TypeScript должен выбрать тип из нескольких возможных
let zoo = [new Rhino(), new Elephant(), new Snake()];
// Что должно быть типом zoo?

// TypeScript использует "best common type algorithm" для определения T
// в Array<T>. Для типов Rhino, Elephant, Snake, 
// выводится тип: (Rhino | Elephant | Snake)[]

// Явное указание типа для лучшего контроля
let betterZoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
// где Animal - родительский класс всех животных

// Вывод при смешивании типов
let mixed = [1, "hello", true];  // тип: (number | string | boolean)[]
let specific: number[] = [1, 2, 3]; // явный тип для контроля
```

### Контекстное определение типов (Contextual Typing)
```typescript
// Контекстное определение влияет на вывод типов
interface Named {
  name: string;
}

class Person {
  name: string;
  constructor(name: string) { this.name = name; }
}

let persons: Named[] = [
  new Person("Alice"),  // OK: Person может быть назначен как Named
  new Person("Bob")     // OK: Person может быть назначен как Named
];

// Без контекста - выводится только структура
const anonymousPerson = new Person("Charlie");
// тип: Person, а не Named
```

## Вывод в асинхронном коде

### Promise и async/await
```typescript
// Вывод типов в асинхронных функциях
async function fetchUser(id: number) {
  const response = await fetch(`/api/users/${id}`);
  const userData = await response.json();
  return userData;  // тип: Promise<unknown>
}

// Вывод из возвращаемого значения
const userPromise = fetchUser(123);
// userPromise: Promise<unknown>
// для получения конкретного типа нужна явная аннотация или generics

// Более точный вывод с generics
async function fetchTypedUser<T>(id: number, typeDef: T): Promise<T> {
  const response = await fetch(`/api/users/${id}`);
  const userData = await response.json();
  return userData as T;
}

// Использование с типом
interface ApiUser {
  id: number;
  name: string;
  email: string;
}

const typedUser = fetchTypedUser(123, {} as ApiUser);
// typedUser: Promise<ApiUser>
```

### Вывод в цепочках Promise
```typescript
// Вывод в цепочках промисов
const userPromise = fetchUser(123)
  .then(user => user.name)        // здесь user: unknown, но name: any
  .then(name => name.toUpperCase()) // здесь name: any, но toUpperCase возвращает string
  .catch(err => {
    console.error(err);
    return "Anonymous";           // тип: string
  });

// Результат: Promise<string>
```

## Вывод с функциональными паттернами

### Вывод в композиции функций
```typescript
// Вывод типов в функциональной композиции
const pipe = <T>(value: T, ...fns: Array<(arg: any) => any>): any =>
  fns.reduce((acc, fn) => fn(acc), value);

// Более точный вывод с generics
const betterPipe = <T, U>(value: T, fn: (arg: T) => U): U => fn(value);

const add = (a: number) => (b: number): number => a + b;
const multiply = (a: number) => (b: number): number => a * b;

// Вывод через композицию
const addFive = add(5);      // (b: number) => number
const multByTwo = multiply(2); // (b: number) => number

const addFiveThenMultByTwo = (x: number) => multByTwo(addFive(x)); // number => number
```

### Вывод в функциях высшего порядка
```typescript
// Вывод типов в функциях высшего порядка
const withLogging = <T extends Function>(fn: T): T => {
  return ((...args: any[]) => {
    console.log(`Calling function with args:`, args);
    const result = fn(...args);
    console.log(`Function returned:`, result);
    return result;
  }) as T;
};

// Вывод типа для обернутой функции
const addLogger = withLogging((a: number, b: number) => a + b);
// addLogger: (a: number, b: number) => number

const result = addLogger(5, 3); // number
```

## Ограничения вывода типов

### Когда вывод не работает
```typescript
// 1. Недостаточно информации
let someValue; // тип: any (или unknown если strict)
someValue = "hello";
someValue = 42; // OK: any может быть чем угодно

// 2. Сложные условные типы могут не выводиться
function complexFunction(value: any) {
  if (typeof value === 'string') {
    // В сложных условиях TypeScript может не вывести тип точно
  }
}

// 3. Возврат из различных мест может затруднить вывод
function ambiguousReturn(condition: boolean) {
  if (condition) {
    return { type: "success", data: "ok" };
  } else {
    return { type: "error", message: "failed" };
  }
  // Вывод: { type: "success"; data: string; } | { type: "error"; message: string; }
}
```

### Явные аннотации для лучшего контроля
```typescript
// Иногда необходимо явно указать тип для лучшего вывода
const callbacks: Array<(data: string) => void> = [];

callbacks.push((data) => {
  // data автоматически выводится как string (из типа массива)
  console.log(data.toUpperCase()); // OK: string метод
});

// Явная типизация для сложных структур
interface ComplexConfig {
  apiUrl: string;
  timeout: number;
  retries: number;
  features: Array<{ name: string; enabled: boolean }>;
}

const config: ComplexConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
  features: [
    { name: "feature1", enabled: true },  // тип выводится из ComplexConfig
    { name: "feature2", enabled: false }
  ]
};
```

## Расширенные примеры вывода

### Вывод с keyof и индексными типами
```typescript
// Вывод типов с использованием keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = {
  name: "Alice",
  age: 30,
  email: "alice@example.com"
};

const userName = getProperty(user, "name");      // string
const userAge = getProperty(user, "age");        // number  
const userEmail = getProperty(user, "email");    // string

// Вывод через маппинг
type UserKeys = keyof typeof user;  // "name" | "age" | "email"
```

### Вывод в обобщенных классах
```typescript
// Вывод типов в конструкторе обобщенного класса
class Container<T> {
  constructor(public value: T) {}
  
  getValue(): T {
    return this.value;
  }
  
  map<U>(transform: (value: T) => U): Container<U> {
    return new Container(transform(this.value));
  }
}

// Вывод типа при создании экземпляра
const stringContainer = new Container("hello");
// stringContainer: Container<string>

const numberContainer = new Container(42);
// numberContainer: Container<number>

// Вывод при использовании методов
const transformed = stringContainer.map(s => s.length);
// transformed: Container<number>
```

## Лучшие практики вывода типов

### 1. Полагайтесь на вывод, но будьте осторожны с any
```typescript
// Хорошо: полагаемся на вывод, где это безопасно
const users = [
  { id: 1, name: "Alice", email: "alice@example.com" },
  { id: 2, name: "Bob", email: "bob@example.com" }
];
// users: { id: number; name: string; email: string; }[]

// Плохо: когда не хватает информации
const dynamicData = [];
dynamicData.push({ id: 1, name: "Test" });
// dynamicData: never[] (если strict) или any[] (если не strict)
```

### 2. Используйте явные типы для API
```typescript
// Хорошо: явные типы для публичных API
function publicAPI(userData: { name: string; email: string }): { success: boolean; id: number } {
  return { success: true, id: 123 };
}

// Менее хорошо: полагаемся на вывод
// function publicAPI(userData: any) { return { success: true, id: 123 }; }
```

### 3. Используйте type assertions для уточнения при необходимости
```typescript
// Когда вывод не достаточно точный
const rawData: any = getRawDataFromAPI();
const typedData = rawData as { users: User[]; total: number };
// Явное утверждение типа для лучшей безопасности
```

## Современные тенденции

### Вывод с Template Literal Types
```typescript
// Новые возможности TypeScript 4.1+ позволяют более точный вывод
type EventName = `on${'click' | 'hover' | 'focus'}`;
const handlers: Record<EventName, (e: Event) => void> = {
  onclick: (e) => console.log(e),
  onhover: (e) => console.log(e),
  onfocus: (e) => console.log(e)
};

// Вывод типов свойств на основе шаблонных литералов
const eventName: EventName = 'onclick'; // выводит тип корректно
```

### Вывод с условными типами
```typescript
// Условные типы позволяют более умный вывод
type Unpromisify<T> = T extends Promise<infer U> ? U : T;

async function getData() {
  return "Hello";
}

const promiseResult = getData();    // Promise<string>
const awaitedResult = await getData(); // string (выведен через Unpromisify)
```

## Ошибки и ловушки

### Частые ошибки с выводом типов
```typescript
// 1. Неполный вывод в сложных выражениях
interface ComplexType {
  data: { [key: string]: string | number | boolean };
}

// ПЛОХО: не все поля могут быть выведены корректно
const inferred = {
  data: { name: "test" } // Выведен как { [key: string]: string }
  // Но ожидается: { [key: string]: string | number | boolean }
};

// 2. Вывод any при отсутствии контекста
function handleError(error: any) {
  // error может быть чем угодно - вывод не помогает
  console.log(error.message); // потенциальная ошибка времени выполнения
}

// 3. Неправильный вывод в циклах
const items: (string | number)[] = ["hello", 42, "world"];
let firstItem; // тип: any
for (const item of items) {
  firstItem = item; // не помогает выводу типа
  break;
}
// ЛУЧШЕ:
const firstItemCorrect = items[0]; // string | number (корректный вывод)
```

## Сравнение с другими подходами

| Подход | Когда использовать | Преимущества | Недостатки |
|--------|-------------------|-------------|------------|
| Вывод типов | При очевидных типах значения | Меньше бойлерплейта, чище код | Может быть неточным |
| Явная типизация | Для API, сложных типов | Ясность, контроль, документация | Более многословно |
| Вывод + проверка | Комбинация обоих подходов | Баланс между удобством и безопасностью | Требует опыта |

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - как выводимые типы проверяются на совместимость
- [[../generics/index]] - вывод generic параметров
- [[../advanced-types/conditional-types]] - условные типы и вывод
- [[../advanced-types/mapped-types]] - сопоставляющие типы и вывод
- [[../functional-programming/index]] - функциональные паттерны с выводом типов
- [[../interfaces-classes/index]] - вывод в классах и интерфейсах
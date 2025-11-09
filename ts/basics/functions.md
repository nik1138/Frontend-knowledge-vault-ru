# TypeScript Functions

Функции в TypeScript расширяют функции JavaScript дополнительными возможностями статической типизации. Это включает аннотации типов параметров, возвращаемых значений и различные продвинутые функциональные паттерны.

## Базовая типизация функций

### Аннотации параметров и возвращаемого значения
```typescript
// Простая функция с типизацией
function add(x: number, y: number): number {
  return x + y;
}

// Функциональное выражение
const subtract = (x: number, y: number): number => {
  return x - y;
};

// Стрелочная функция
const multiply = (x: number, y: number): number => x * y;

// Вызов функций
const result = add(5, 3); // number
```

### Необязательные параметры
```typescript
function buildName(firstName: string, lastName?: string): string {
  if (lastName) {
    return `${firstName} ${lastName}`;
  } else {
    return firstName;
  }
}

const name1 = buildName("Bob"); // OK
const name2 = buildName("Bob", "Adams"); // OK
// const name3 = buildName("Bob", undefined, "Jr."); // Ошибка: слишком много аргументов
```

### Параметры по умолчанию
```typescript
function multiplyWithDefault(x: number, y: number = 10): number {
  return x * y;
}

// Использование
multiplyWithDefault(5); // 50 (y = 10 по умолчанию)
multiplyWithDefault(5, 2); // 10

// Параметры по умолчанию могут быть необязательными без ?
function greet(name: string, greeting: string = "Hello", punctuation: string = "!"): string {
  return `${greeting}, ${name}${punctuation}`;
}
```

### Остальные параметры (rest parameters)
```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, curr) => acc + curr, 0);
}

// Использование
sum(1, 2, 3, 4, 5); // 15
sum(); // 0
sum(42); // 42

// Комбинация с другими параметрами
function join(separator: string, ...strings: string[]): string {
  return strings.join(separator);
}

join(", ", "apple", "banana", "cherry"); // "apple, banana, cherry"
```

## Типизация функций

### Типы функций
```typescript
// Явное объявление типа функции
let myAdd: (x: number, y: number) => number;

// Присвоение функции
myAdd = function(x: number, y: number): number { return x + y; };

// Или с интерфейсом
interface MathFunction {
  (x: number, y: number): number;
}

const addFunc: MathFunction = (a, b) => a + b;
```

### Контекстное определение типов
```typescript
// Контекстно-зависимый вывод типов
const names = ["Alice", "Bob", "Charlie"];
names.forEach(function (s) {
  // Тип параметра s выводится как string
  console.log(s.toUppercase()); // Ошибка: toUppercase не существует (демонстрирует проверку)
});

names.forEach((s) => console.log(s.toUpperCase())); // OK: тип s выводится как string
```

### Типизация callback'ов
```typescript
// Простой callback
function delayedCall(callback: () => void, delay: number): void {
  setTimeout(callback, delay);
}

delayedCall(() => console.log("Delayed!"), 1000);

// Callback с параметрами
function processItems<T>(items: T[], callback: (item: T, index: number) => void): void {
  items.forEach(callback);
}

processItems([1, 2, 3], (num, idx) => {
  console.log(`${idx}: ${num}`);
});
```

## Продвинутые паттерны

### Перегрузки функций
```typescript
// Перегрузка функций позволяет иметь разные сигнатуры для одной функции
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    // Правильная дата m/d/y
    return new Date(y, mOrTimestamp, d);
  } else {
    // Дата с timestamp
    return new Date(mOrTimestamp);
  }
}

const d1 = makeDate(12345678); // Date
const d2 = makeDate(5, 5, 5); // Date
// const d3 = makeDate(1, 2); // Ошибка: нет подходящей перегрузки
```

### Функции как объекты
```typescript
// Функции могут иметь свойства
function greet(name: string): string {
  return `Hello, ${name}!`;
}

greet.count = 0; // Ошибка: у функции нет свойства count

// Правильное объявление с дополнительными свойствами
interface Greeter {
  (name: string): string;
  count: number;
}

const greeter: Greeter = ((name: string) => `Hello, ${name}!`) as Greeter;
greeter.count = 0;
```

### Оператор this в функциях
```typescript
interface User {
  id: number;
  name: string;
  getName(this: User): string; // Явное указание типа this
}

const user: User = {
  id: 1,
  name: "Alice",
  getName() {
    // Здесь this имеет тип User
    return this.name;
  }
};

// При передаче метода как callback, потеряется связь с this
// const getName = user.getName; // Ошибка при вызове без контекста

// Использование стрелочной функции сохраняет this
const api = {
  user: user,
  getName: () => user.name // this не зависит от вызова
};
```

## Обобщенные функции

### Обобщенные функции
```typescript
// Простая обобщенная функция
function identity<T>(arg: T): T {
  return arg;
}

// Вызов с явным указанием типа
let output1 = identity<string>("myString");

// Вызов с выводом типа
let output2 = identity(42); // T выводится как number

// Обобщенная функция с ограничениями
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // OK: теперь мы знаем, что у arg есть свойство length
  return arg;
}
```

### Обобщенные callback'и
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };
let propName: "a" | "b" | "c" | "d" = "a";

// Вывод типов в обобщенных функциях
const value = getProperty(x, propName); // number
const specificValue = getProperty(x, "a"); // number
```

## Функциональные паттерны

### Чистые функции
```typescript
// Чистая функция: всегда возвращает одинаковый результат для одинаковых аргументов
function pureAdd(a: number, b: number): number {
  return a + b;
}

// Не чистая функция: зависит от внешнего состояния
let multiplier = 2;
function impureMultiply(a: number): number {
  return a * multiplier; // Результат зависит от внешней переменной
}
```

### Высшие порядки функций
```typescript
// Функция, возвращающая другую функцию
function createAdder(x: number): (y: number) => number {
  return function(y: number): number {
    return x + y;
  };
}

const add5 = createAdder(5);
const result = add5(3); // 8

// Функция, принимающая другую функцию
function applyOperation<T>(value: T, operation: (v: T) => T): T {
  return operation(value);
}

const doubled = applyOperation(5, x => x * 2); // 10
```

### Каррирование
```typescript
// Каррированная функция
function addCurry(a: number): (b: number) => number {
  return function(b: number): number {
    return a + b;
  };
}

// Или с стрелочной функцией
const addArrow = (a: number) => (b: number) => a + b;

const add5Curry = addCurry(5);
const result = add5Curry(3); // 8

// Каррирование с несколькими параметрами
const multiplyAll = (a: number) => (b: number) => (c: number) => a * b * c;
const multiplyBy2 = multiplyAll(2);
const multiplyBy2And3 = multiplyBy2(3);
const finalResult = multiplyBy2And3(4); // 24
```

## Функции и объекты

### Методы объекта
```typescript
interface Counter {
  count: number;
  increment(): void;
  decrement(): void;
  getCount(): number;
}

const counter: Counter = {
  count: 0,
  increment() {
    this.count++; // this указывает на counter
  },
  decrement() {
    this.count--;
  },
  getCount() {
    return this.count;
  }
};
```

### Функции в массивах
```typescript
// Типизация массива функций
const operations: ((x: number) => number)[] = [
  x => x + 1,
  x => x * 2,
  x => x - 1
];

let result = 5;
for (const op of operations) {
  result = op(result);
}
// result = ((5 + 1) * 2) - 1 = 11
```

## Асинхронные функции

### Promise и async/await
```typescript
// Асинхронная функция
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const userData = await response.json();
  return userData as User;
}

// Использование
async function displayUserInfo(id: number): Promise<void> {
  try {
    const user = await fetchUser(id);
    console.log(`User: ${user.name}`);
  } catch (error) {
    console.error('Failed to fetch user:', error);
  }
}
```

### Типизация Promise
```typescript
// Функция, возвращающая Promise
function delayedValue<T>(value: T, delay: number): Promise<T> {
  return new Promise(resolve => {
    setTimeout(() => resolve(value), delay);
  });
}

// Использование
delayedValue("Hello", 1000).then(result => {
  // result: string
  console.log(result.toUpperCase());
});
```

## Практические примеры

### Валидация параметров
```typescript
interface ValidationConfig {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
}

function validateString(
  value: string, 
  config: ValidationConfig
): { isValid: boolean; error?: string } {
  
  if (config.required && !value) {
    return { isValid: false, error: "Value is required" };
  }
  
  if (config.minLength && value.length < config.minLength) {
    return { isValid: false, error: `Minimum length is ${config.minLength}` };
  }
  
  if (config.maxLength && value.length > config.maxLength) {
    return { isValid: false, error: `Maximum length is ${config.maxLength}` };
  }
  
  if (config.pattern && !config.pattern.test(value)) {
    return { isValid: false, error: "Value does not match pattern" };
  }
  
  return { isValid: true };
}
```

### Утилита для композиции функций
```typescript
// Тип для функций композиции
type ComposableFn<T, U> = (input: T) => U;

// Простая композиция двух функций
function compose<A, B, C>(
  f: ComposableFn<B, C>,
  g: ComposableFn<A, B>
): ComposableFn<A, C> {
  return function(x: A): C {
    return f(g(x));
  };
}

// Использование
const addOne = (x: number): number => x + 1;
const double = (x: number): number => x * 2;
const addOneThenDouble = compose(double, addOne);

addOneThenDouble(5); // double(addOne(5)) = double(6) = 12
```

## Лучшие практики

1. **Указывайте возвращаемый тип для публичных API** - особенно если он не очевиден
2. **Используйте необязательные параметры вместо нескольких перегрузок** - когда это возможно
3. **Типизируйте callback'и явно** - особенно когда они передаются в другие функции
4. **Используйте обобщения для переиспользуемых функций** - для типовой безопасности
5. **Документируйте сложные сигнатуры функций** - особенно с перегрузками
6. **Используйте стрелочные функции для сохранения контекста this** - когда нужно

## Связь с другими концепциями
- [[../basics/type-annotations]] - основы типизации функций
- [[TS Generics]] - обобщенные функции
- [[Functional Programming in TS]] - функциональные паттерны
- [[../advanced-types/conditional-types]] - продвинутая типизация функций
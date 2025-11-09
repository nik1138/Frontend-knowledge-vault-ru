# Частичное применение

Частичное применение - это техника, при которой функция вызывается с меньшим количеством аргументов, чем она ожидает, в результате чего возвращается новая функция, ожидающая оставшиеся аргументы.

## Содержание

- [Частичное применение](#частичное-применение)
  - [Содержание](#содержание)
  - [Понятие частичного применения](#понятие-частичного-применения)
  - [Различие между каррированием и частичным применением](#различие-между-каррированием-и-частичным-применением)
  - [Реализация частичного применения](#реализация-частичного-применения)
  - [Практические примеры](#практические-примеры)
  - [Паттерны использования](#паттерны-использования)
    - [Создание специализированных функций](#создание-специализированных-функций)
    - [Конфигурация функций](#конфигурация-функций)
  - [Лучшие практики](#лучшие-практики)
    - [1. Явное указание типов](#1-явное-указание-типов)
    - [2. Избегайте избыточного частичного применения](#2-избегайте-избыточного-частичного-применения)
    - [3. Комбинирование с другими функциональными техниками](#3-комбинирование-с-другими-функциональными-техниками)
  - [Распространенные ошибки](#распространенные-ошибки)
    - [1. Неправильное понимание концепции](#1-неправильное-понимание-концепции)
    - [2. Потеря контекста](#2-потеря-контекста)
    - [3. Изменяемые аргументы](#3-изменяемые-аргументы)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Понятие частичного применения

Частичное применение позволяет создавать новые функции путем фиксации некоторых аргументов существующей функции. Это мощная техника для создания специализированных версий общих функций.

```typescript
// Функция с несколькими аргументами
const multiply = (a: number, b: number, c: number): number => a * b * c;

// Частичное применение с фиксацией первого аргумента
const multiplyByTwo = (b: number, c: number): number => multiply(2, b, c);
console.log(multiplyByTwo(3, 4)); // 24

// Частичное применение с фиксацией первых двух аргументов
const multiplyByTwoAndThree = (c: number): number => multiply(2, 3, c);
console.log(multiplyByTwoAndThree(4)); // 24
```

## Различие между каррированием и частичным применением

Хотя каррирование и частичное применение часто путают, это разные концепции:

- **Каррирование** - это процесс преобразования функции с n аргументами в цепочку из n функций с одним аргументом
- **Частичное применение** - это вызов функции с меньшим количеством аргументов, чем она ожидает

```typescript
// Каррирование
const curriedAdd = (a: number) => (b: number) => (c: number): number => a + b + c;
const addFive = curriedAdd(2)(3); // Частично примененная функция
console.log(addFive(5)); // 10

// Частичное применение без каррирования
const add = (a: number, b: number, c: number): number => a + b + c;
const partiallyAppliedAdd = (b: number, c: number): number => add(2, b, c);
console.log(partiallyAppliedAdd(3, 5)); // 10
```

## Реализация частичного применения

Мы можем создать универсальную функцию для частичного применения аргументов к любой функции.

```typescript
// Универсальная функция частичного применения
function partial<T extends any[], U extends any[], R>(
  fn: (...args: [...T, ...U]) => R,
  ...args: T
): (...remainingArgs: U) => R {
  return (...remainingArgs: U) => fn(...args, ...remainingArgs);
}

// Пример использования
const greet = (greeting: string, name: string, punctuation: string): string =>
  `${greeting}, ${name}${punctuation}`;

// Частичное применение первого аргумента
const greetWithHello = partial(greet, "Привет");
console.log(greetWithHello("Иван", "!")); // "Привет, Иван!"

// Частичное применение первых двух аргументов
const greetIvan = partial(greet, "Привет", "Иван");
console.log(greetIvan("?")); // "Привет, Иван?"
```

## Практические примеры

Рассмотрим практические примеры использования частичного применения в реальных приложениях.

```typescript
// Пример 1: Работа с массивами
const filterBy = <T>(predicate: (item: T) => boolean) => (array: T[]): T[] => 
  array.filter(predicate);

const mapWith = <T, R>(fn: (item: T) => R) => (array: T[]): R[] => 
  array.map(fn);

// Создание специализированных функций через частичное применение
const isEven = (n: number): boolean => n % 2 === 0;
const filterEven = filterBy(isEven);

const double = (n: number): number => n * 2;
const mapDouble = mapWith(double);

// Использование
const numbers = [1, 2, 3, 4, 5, 6];
console.log(filterEven(numbers)); // [2, 4, 6]
console.log(mapDouble(numbers)); // [2, 4, 6, 8, 10, 12]

// Пример 2: Работа с API
interface ApiRequest {
  method: string;
  url: string;
  headers?: Record<string, string>;
  body?: any;
}

const makeRequest = (
  baseUrl: string,
  defaultHeaders: Record<string, string>,
  request: ApiRequest
): Promise<Response> => {
  const fullUrl = `${baseUrl}${request.url}`;
  const headers = { ...defaultHeaders, ...request.headers };
  
  return fetch(fullUrl, {
    method: request.method,
    headers,
    body: request.body ? JSON.stringify(request.body) : undefined
  });
};

// Создание специализированных клиентов через частичное применение
const createApiClient = partial(
  makeRequest,
  "https://api.example.com",
  { "Content-Type": "application/json" }
);

// Использование
const apiClient = createApiClient;
apiClient({
  method: "GET",
  url: "/users"
});

apiClient({
  method: "POST",
  url: "/users",
  body: { name: "Иван", email: "ivan@example.com" }
});
```

## Паттерны использования

### Создание специализированных функций

Частичное применение часто используется для создания специализированных версий общих функций.

```typescript
// Общая функция для форматирования дат
const formatDate = (
  locale: string,
  options: Intl.DateTimeFormatOptions,
  date: Date
): string => {
  return new Intl.DateTimeFormat(locale, options).format(date);
};

// Специализированные функции через частичное применение
const formatRuDate = partial(
  formatDate,
  "ru-RU",
  { year: "numeric", month: "long", day: "numeric" }
);

const formatUsTime = partial(
  formatDate,
  "en-US",
  { hour: "2-digit", minute: "2-digit" }
);

// Использование
const now = new Date();
console.log(formatRuDate(now)); // "8 ноября 2025 г."
console.log(formatUsTime(now)); // "11:58 PM"
```

### Конфигурация функций

Частичное применение полезно для создания функций с предустановленной конфигурацией.

```typescript
// Функция с конфигурацией
interface LoggerConfig {
  prefix: string;
  timestamp: boolean;
  level: "debug" | "info" | "warn" | "error";
}

const log = (
  config: LoggerConfig,
  message: string,
  ...args: any[]
): void => {
  const timestamp = config.timestamp ? `[${new Date().toISOString()}]` : "";
  const prefix = config.prefix ? `[${config.prefix}]` : "";
  const level = `[${config.level.toUpperCase()}]`;
  
  console.log(`${timestamp}${prefix}${level} ${message}`, ...args);
};

// Создание логгеров с разной конфигурацией
const createLogger = (prefix: string) => partial(
  log,
  { prefix, timestamp: true, level: "info" }
);

const debugLogger = partial(
  log,
  { prefix: "DEBUG", timestamp: false, level: "debug" }
);

const errorLogger = partial(
  log,
  { prefix: "ERROR", timestamp: true, level: "error" }
);

// Использование
const appLogger = createLogger("APP");
appLogger("Приложение запущено");

debugLogger("Отладочная информация");
errorLogger("Произошла ошибка", new Error("Тестовая ошибка"));
```

## Лучшие практики

### 1. Явное указание типов

При использовании частичного применения важно явно указывать типы для обеспечения безопасности типов.

```typescript
// Хорошо: явные типы
const add = (a: number, b: number, c: number): number => a + b + c;
const addFive = partial(add, 2, 3); // Тип: (c: number) => number

// Плохо: неявные типы могут привести к ошибкам
const addUntyped: any = (a: number, b: number, c: number): number => a + b + c;
const addFiveUntyped = partial(addUntyped, 2, 3); // Тип: any
```

### 2. Избегайте избыточного частичного применения

Не создавайте слишком много специализированных функций, если они используются редко.

```typescript
// Хорошо: создание специализированной функции для частого использования
const formatCurrency = partial(
  Intl.NumberFormat,
  "ru-RU",
  { style: "currency", currency: "RUB" }
);

// Плохо: создание специализированной функции для однократного использования
const addSpecificNumbers = partial((a: number, b: number) => a + b, 42, 18);
```

### 3. Комбинирование с другими функциональными техниками

Частичное применение хорошо сочетается с другими функциональными техниками.

```typescript
// Комбинирование частичного применения с композицией
const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B): ((a: A) => C) => 
  (a: A) => f(g(a));

const multiply = (a: number, b: number): number => a * b;
const add = (a: number, b: number): number => a + b;

// Частичное применение
const multiplyByTwo = partial(multiply, 2);
const addTen = partial(add, 10);

// Композиция
const multiplyByTwoAndAddTen = compose(addTen, multiplyByTwo);
console.log(multiplyByTwoAndAddTen(5)); // 20 (5 * 2 + 10)
```

## Распространенные ошибки

### 1. Неправильное понимание концепции

```typescript
// Ошибка: путаница между каррированием и частичным применением
const add = (a: number, b: number): number => a + b;
const addFive = (b: number): number => add(5, b); // Это частичное применение
const curriedAdd = (a: number) => (b: number): number => a + b; // Это каррирование
```

### 2. Потеря контекста

```typescript
// Ошибка: потеря контекста при частичном применении методов
class Calculator {
  private value: number = 0;
  
  add(n: number): this {
    this.value += n;
    return this;
  }
  
  getValue(): number {
    return this.value;
  }
}

const calc = new Calculator();
// Ошибка: потеря контекста
const addTen = partial(calc.add, 10);
// addTen(); // Ошибка: this будет undefined

// Решение: использование bind или стрелочных функций
const addTenBound = calc.add.bind(calc, 10);
```

### 3. Изменяемые аргументы

```typescript
// Ошибка: изменение аргументов после частичного применения
const processArray = (multiplier: number, arr: number[]): number[] => 
  arr.map(n => n * multiplier);

const numbers = [1, 2, 3];
const doubleArray = partial(processArray, 2);

console.log(doubleArray(numbers)); // [2, 4, 6]

// Ошибка: изменение исходного массива может повлиять на результат
numbers.push(4);
console.log(doubleArray(numbers)); // [2, 4, 6, 8] - неожиданный результат

// Решение: создание копии массива
const processArraySafe = (multiplier: number, arr: number[]): number[] => 
  [...arr].map(n => n * multiplier);
```

## Связи с другими концепциями

- [[ts/functional-programming/currying/Каррирование|Каррирование]] - Основная страница раздела
- [[ts/functional-programming/currying/implementation|Реализация каррирования]] - Различные способы реализации каррирования
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/testing/index|Тестирование]] - Тестирование функций с частичным применением

## Следующие шаги

- [[ts/functional-programming/currying/practical-examples|Практические примеры]] - Реальные примеры использования каррирования
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация функций с частичным применением
- [[ts/testing/index|Тестирование]] - Тестирование функций с частичным применением
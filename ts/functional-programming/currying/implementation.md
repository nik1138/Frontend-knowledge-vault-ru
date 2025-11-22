# Реализация каррирования

В этом разделе мы рассмотрим различные способы реализации каррирования в TypeScript, включая ручную реализацию, автоматическое каррирование и использование библиотек.

## Содержание

- [Реализация каррирования](#реализация-каррирования)
  - [Содержание](#содержание)
  - [Ручная реализация](#ручная-реализация)
  - [Универсальная функция каррирования](#универсальная-функция-каррирования)
  - [Каррирование с сохранением типов](#каррирование-с-сохранением-типов)
  - [Использование библиотек](#использование-библиотек)
  - [Производительность](#производительность)
  - [Практические примеры](#практические-примеры)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Ручная реализация

Самый простой способ реализовать каррирование - это вручную преобразовать функцию с несколькими аргументами в цепочку функций с одним аргументом.

```typescript
// Функция без каррирования
const multiply = (a: number, b: number, c: number): number => a * b * c;

// Ручное каррирование
const curriedMultiply = (a: number) => (b: number) => (c: number): number => a * b * c;

// Использование
console.log(curriedMultiply(2)(3)(4)); // 24

// Частичное применение
const multiplyByTwo = curriedMultiply(2);
const multiplyByTwoAndThree = multiplyByTwo(3);
console.log(multiplyByTwoAndThree(4)); // 24
```

## Универсальная функция каррирования

Мы можем создать универсальную функцию, которая автоматически каррирует любую функцию с фиксированным числом аргументов.

```typescript
// Универсальная функция каррирования
function curry<T extends any[], R>(fn: (...args: T) => R): Curried<T, R> {
  return function curried(...args: any[]): any {
    if (args.length >= fn.length) {
      return fn.apply(null, args);
    }
    
    return (...nextArgs: any[]) => curried(...args, ...nextArgs);
  } as Curried<T, R>;
}

// Тип для каррированной функции
type Curried<T extends any[], R> = 
  T extends [infer First, ...infer Rest] 
    ? (arg: First) => Curried<Rest, R>
    : R;

// Пример использования
const addThree = (a: number, b: number, c: number): number => a + b + c;
const curriedAddThree = curry(addThree);

console.log(curriedAddThree(1)(2)(3)); // 6
console.log(curriedAddThree(1, 2)(3)); // 6
console.log(curriedAddThree(1)(2, 3)); // 6
console.log(curriedAddThree(1, 2, 3)); // 6
```

## Каррирование с сохранением типов

Для более сложных случаев мы можем создать реализацию каррирования с полной поддержкой типов.

```typescript
// Расширенная реализация каррирования с сохранением типов
type CurryV1<F> = 
  F extends (...args: infer A) => infer R
    ? A extends [infer First, ...infer Rest]
      ? (arg: First) => CurryV1<(...args: Rest) => R>
      : R
    : never;

function curryV1<F extends (...args: any[]) => any>(fn: F): CurryV1<F> {
  return function curried(...args: any[]): any {
    if (args.length >= fn.length) {
      return fn.apply(null, args);
    }
    
    return (...nextArgs: any[]) => curried(...args, ...nextArgs);
  } as CurryV1<F>;
}

// Пример с типизированной функцией
const formatUser = (name: string, age: number, city: string): string => 
  `${name}, ${age} лет, ${city}`;

const curriedFormatUser = curryV1(formatUser);

// Типы сохраняются
const withName = curriedFormatUser("Иван"); // (age: number) => (city: string) => string
const withNameAndAge = withName(25); // (city: string) => string
const result = withNameAndAge("Москва"); // string

console.log(result); // "Иван, 25 лет, Москва"
```

## Использование библиотек

Существуют библиотеки, которые предоставляют готовые реализации каррирования с полной поддержкой типов.

```typescript
// Пример с использованием библиотеки Ramda
import * as R from 'ramda';

// Функция с несколькими аргументами
const greet = (greeting: string, separator: string, emphasis: string, name: string): string =>
  `${greeting}${separator} ${name}${emphasis}`;

// Каррирование с помощью Ramda
const curriedGreet = R.curry(greet);

// Использование
console.log(curriedGreet("Привет", ",", "!", "Иван")); // "Привет, Иван!"
console.log(curriedGreet("Привет")(",", "!")("Иван")); // "Привет, Иван!"
console.log(curriedGreet("Привет", ",")("!")("Иван")); // "Привет, Иван!"
```

## Производительность

Каррирование может повлиять на производительность из-за дополнительных вызовов функций. Важно учитывать это при разработке.

```typescript
// Измерение производительности каррированных функций
function performanceTest() {
  const multiply = (a: number, b: number, c: number): number => a * b * c;
  const curriedMultiply = curry(multiply);
  
  const iterations = 1000000;
  
  // Тест обычной функции
  console.time("Обычная функция");
  for (let i = 0; i < iterations; i++) {
    multiply(2, 3, 4);
  }
  console.timeEnd("Обычная функция");
  
  // Тест каррированной функции
  console.time("Каррированная функция");
  for (let i = 0; i < iterations; i++) {
    curriedMultiply(2)(3)(4);
  }
  console.timeEnd("Каррированная функция");
  
  // Тест частично примененной функции
  console.time("Частично примененная функция");
  const partiallyApplied = curriedMultiply(2)(3);
  for (let i = 0; i < iterations; i++) {
    partiallyApplied(4);
  }
  console.timeEnd("Частично примененная функция");
}

// performanceTest();
```

## Практические примеры

Рассмотрим практические примеры использования каррирования в реальных приложениях.

```typescript
// Пример 1: Каррирование для работы с API
interface ApiConfig {
  baseUrl: string;
  apiKey: string;
}

interface RequestConfig {
  method: string;
  endpoint: string;
  data?: any;
}

const createApiRequest = (config: ApiConfig) => (request: RequestConfig) => {
  const url = `${config.baseUrl}${request.endpoint}`;
  console.log(`Запрос к ${url} с методом ${request.method}`);
  // Здесь был бы реальный запрос к API
  return Promise.resolve({ status: 200, data: request.data || {} });
};

// Создание API клиента с конфигурацией
const apiRequest = createApiRequest({
  baseUrl: "https://api.example.com",
  apiKey: "secret-key"
});

// Использование
apiRequest({ method: "GET", endpoint: "/users" });
apiRequest({ method: "POST", endpoint: "/users", data: { name: "Иван" } });

// Пример 2: Каррирование для работы с массивами
const filterBy = <T>(predicate: (item: T) => boolean) => (array: T[]): T[] => 
  array.filter(predicate);

const mapWith = <T, R>(fn: (item: T) => R) => (array: T[]): R[] => 
  array.map(fn);

// Создание специализированных функций
const filterEven = filterBy((n: number) => n % 2 === 0);
const double = (n: number) => n * 2;
const mapDouble = mapWith(double);

// Композиция операций
const processNumbers = (numbers: number[]) => 
  mapDouble(filterEven(numbers));

console.log(processNumbers([1, 2, 3, 4, 5, 6])); // [4, 8, 12]
```

## Связи с другими концепциями

- [[ts/functional-programming/currying/Каррирование|Каррирование]] - Основная страница раздела
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация каррированных функций
- [[Тестирование в TypeScript|Тестирование]] - Тестирование каррированных функций

## Следующие шаги

- [[ts/functional-programming/currying/partial-application|Частичное применение]] - Использование частично примененных функций
- [[ts/functional-programming/currying/practical-examples|Практические примеры]] - Реальные примеры использования каррирования
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация каррированных функций
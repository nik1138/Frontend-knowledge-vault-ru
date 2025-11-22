# Реализация композиции функций

В этом разделе мы рассмотрим различные способы реализации композиции функций в TypeScript, включая базовую композицию, универсальные функции композиции и продвинутые техники.

## Содержание

- [Реализация композиции функций](#реализация-композиции-функций)
  - [Содержание](#содержание)
  - [Базовая композиция](#базовая-композиция)
  - [Универсальная функция композиции](#универсальная-функция-композиции)
  - [Композиция с сохранением типов](#композиция-с-сохранением-типов)
  - [Пайплайн композиция](#пайплайн-композиция)
  - [Композиция с обработкой ошибок](#композиция-с-обработкой-ошибок)
  - [Производительность](#производительность)
  - [Практические примеры](#практические-примеры)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Базовая композиция

Самый простой способ реализовать композицию - это вручную объединить две функции.

```typescript
// Базовая композиция двух функций
const compose2 = <A, B, C>(
  f: (b: B) => C,
  g: (a: A) => B
): ((a: A) => C) => {
  return (a: A) => f(g(a));
};

// Пример использования
const add = (x: number): number => x + 1;
const multiply = (x: number): number => x * 2;

const addAndMultiply = compose2(multiply, add);
console.log(addAndMultiply(5)); // 12 (5 + 1 = 6, 6 * 2 = 12)

const multiplyAndAdd = compose2(add, multiply);
console.log(multiplyAndAdd(5)); // 11 (5 * 2 = 10, 10 + 1 = 11)
```

## Универсальная функция композиции

Мы можем создать универсальную функцию, которая компонует любое количество функций.

```typescript
// Универсальная функция композиции (справа налево)
type Compose = {
  <A, B, C>(f: (b: B) => C, g: (a: A) => B): (a: A) => C;
  <A, B, C, D>(f: (c: C) => D, g: (b: B) => C, h: (a: A) => B): (a: A) => D;
  <A, B, C, D, E>(f: (d: D) => E, g: (c: C) => D, h: (b: B) => C, i: (a: A) => B): (a: A) => E;
  (...fns: Function[]): Function;
};

const compose: Compose = (...fns: Function[]) => {
  if (fns.length === 0) {
    return <T>(x: T) => x;
  }
  
  if (fns.length === 1) {
    return fns[0];
  }
  
  return fns.reduceRight((prevFn, nextFn) => (...args: any[]) => prevFn(nextFn(...args)));
};

// Пример использования
const add = (x: number): number => x + 1;
const multiply = (x: number): number => x * 2;
const square = (x: number): number => x * x;

const complexOperation = compose(square, multiply, add);
console.log(complexOperation(3)); // 64 ((3 + 1) * 2) ^ 2 = (4 * 2) ^ 2 = 8 ^ 2 = 64
```

## Композиция с сохранением типов

Для более сложных случаев мы можем создать реализацию композиции с полной поддержкой типов.

```typescript
// Расширенная реализация композиции с сохранением типов
type ComposeWithTypes = {
  <A, B>(f1: (a: A) => B): (a: A) => B;
  <A, B, C>(f2: (b: B) => C, f1: (a: A) => B): (a: A) => C;
  <A, B, C, D>(f3: (c: C) => D, f2: (b: B) => C, f1: (a: A) => B): (a: A) => D;
  <A, B, C, D, E>(f4: (d: D) => E, f3: (c: C) => D, f2: (b: B) => C, f1: (a: A) => B): (a: A) => E;
  <A, B, C, D, E, F>(f5: (e: E) => F, f4: (d: D) => E, f3: (c: C) => D, f2: (b: B) => C, f1: (a: A) => B): (a: A) => F;
  (...fns: Function[]): Function;
};

const composeWithTypes: ComposeWithTypes = (...fns: Function[]) => {
  return fns.reduceRight((prevFn, nextFn) => (...args: any[]) => prevFn(nextFn(...args)));
};

// Пример с типизированными функциями
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserDto {
  id: number;
  fullName: string;
  emailAddress: string;
}

const normalizeUser = (user: User): UserDto => ({
  id: user.id,
  fullName: user.name.trim(),
  emailAddress: user.email.toLowerCase()
});

const validateUser = (user: UserDto): UserDto & { isValid: boolean } => ({
  ...user,
  isValid: user.fullName.length > 0 && user.emailAddress.includes("@")
});

const formatUser = (user: UserDto & { isValid: boolean }): string => 
  `${user.fullName} (${user.emailAddress}) - ${user.isValid ? "Валидный" : "Невалидный"}`;

// Композиция с сохранением типов
const processUser = composeWithTypes(formatUser, validateUser, normalizeUser);

const user: User = {
  id: 1,
  name: "  Иван  ",
  email: "IVAN@EXAMPLE.COM"
};

console.log(processUser(user)); // "Иван (ivan@example.com) - Валидный"
```

## Пайплайн композиция

Альтернативный подход к композиции - пайплайн (слева направо), который может быть более интуитивным.

```typescript
// Пайплайн композиция (слева направо)
type Pipe = {
  <A, B>(f1: (a: A) => B): (a: A) => B;
  <A, B, C>(f1: (a: A) => B, f2: (b: B) => C): (a: A) => C;
  <A, B, C, D>(f1: (a: A) => B, f2: (b: B) => C, f3: (c: C) => D): (a: A) => D;
  <A, B, C, D, E>(f1: (a: A) => B, f2: (b: B) => C, f3: (c: C) => D, f4: (d: D) => E): (a: A) => E;
  (...fns: Function[]): Function;
};

const pipe: Pipe = (...fns: Function[]) => {
  return fns.reduce((prevFn, nextFn) => (...args: any[]) => nextFn(prevFn(...args)));
};

// Пример использования
const add = (x: number): number => x + 1;
const multiply = (x: number): number => x * 2;
const square = (x: number): number => x * x;

const complexOperationPipe = pipe(add, multiply, square);
console.log(complexOperationPipe(3)); // 64 ((3 + 1) * 2) ^ 2 = (4 * 2) ^ 2 = 8 ^ 2 = 64

// Сравнение с compose
const complexOperationCompose = compose(square, multiply, add);
console.log(complexOperationCompose(3)); // 64 - тот же результат, но порядок функций обратный
```

## Композиция с обработкой ошибок

Мы можем создать композицию, которая обрабатывает ошибки и возвращает результат в виде Either типа.

```typescript
// Either тип для обработки ошибок
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

// Композиция функций, возвращающих Either
const composeEither = <E, A, B, C>(
  f: (b: B) => Either<E, C>,
  g: (a: A) => Either<E, B>
): ((a: A) => Either<E, C>) => {
  return (a: A) => {
    const result = g(a);
    if (result._tag === "Left") {
      return result;
    }
    return f(result.right);
  };
};

// Пример использования
const parseNumber = (str: string): Either<string, number> => {
  const num = Number(str);
  return isNaN(num) ? left(`Невозможно преобразовать "${str}" в число`) : right(num);
};

const sqrt = (num: number): Either<string, number> => {
  return num < 0 ? left(`Невозможно вычислить квадратный корень из ${num}`) : right(Math.sqrt(num));
};

const formatResult = (num: number): Either<string, string> => {
  return right(`Результат: ${num.toFixed(2)}`);
};

const processNumber = composeEither(formatResult, composeEither(sqrt, parseNumber));

console.log(processNumber("16")); // { _tag: "Right", right: "Результат: 4.00" }
console.log(processNumber("-4")); // { _tag: "Left", left: "Невозможно вычислить квадратный корень из -4" }
console.log(processNumber("abc")); // { _tag: "Left", left: "Невозможно преобразовать \"abc\" в число" }
```

## Производительность

Композиция может повлиять на производительность из-за дополнительных вызовов функций. Важно учитывать это при разработке.

```typescript
// Измерение производительности композиции
function performanceTest() {
  const add = (x: number): number => x + 1;
  const multiply = (x: number): number => x * 2;
  const square = (x: number): number => x * x;
  
  const composedFunction = compose(square, multiply, add);
  const pipedFunction = pipe(add, multiply, square);
  
  const iterations = 1000000;
  
  // Тест обычных последовательных вызовов
  console.time("Последовательные вызовы");
  for (let i = 0; i < iterations; i++) {
    square(multiply(add(i)));
  }
  console.timeEnd("Последовательные вызовы");
  
  // Тест композиции
  console.time("Композиция");
  for (let i = 0; i < iterations; i++) {
    composedFunction(i);
  }
  console.timeEnd("Композиция");
  
  // Тест пайплайна
  console.time("Пайплайн");
  for (let i = 0; i < iterations; i++) {
    pipedFunction(i);
  }
  console.timeEnd("Пайплайн");
}

// performanceTest();
```

## Практические примеры

Рассмотрим практические примеры использования композиции в реальных приложениях.

```typescript
// Пример 1: Обработка данных пользователя
interface RawUser {
  id: string;
  firstName: string;
  lastName: string;
  email: string;
  birthDate: string;
  salary: string;
}

interface ProcessedUser {
  id: number;
  fullName: string;
  email: string;
  age: number;
  salary: number;
}

// Функции для обработки данных
const parseId = (user: RawUser): RawUser & { id: number } => ({
  ...user,
  id: parseInt(user.id, 10)
});

const formatName = (user: RawUser & { id: number }): RawUser & { id: number, fullName: string } => ({
  ...user,
  fullName: `${user.firstName} ${user.lastName}`
});

const calculateAge = (user: RawUser & { id: number, fullName: string }): RawUser & { id: number, fullName: string, age: number } => ({
  ...user,
  age: new Date().getFullYear() - new Date(user.birthDate).getFullYear()
});

const parseSalary = (user: RawUser & { id: number, fullName: string, age: number }): ProcessedUser => ({
  ...user,
  salary: parseFloat(user.salary)
});

// Композиция обработки данных
const processUser = pipe(parseId, formatName, calculateAge, parseSalary);

const rawUser: RawUser = {
  id: "1",
  firstName: "Иван",
  lastName: "Иванов",
  email: "ivan@example.com",
  birthDate: "1990-01-01",
  salary: "50000.50"
};

console.log(processUser(rawUser));
// {
//   id: 1,
//   firstName: "Иван",
//   lastName: "Иванов",
//   email: "ivan@example.com",
//   birthDate: "1990-01-01",
//   salary: "50000.50",
//   fullName: "Иван Иванов",
//   age: 35,
//   salary: 50000.5
// }

// Пример 2: Обработка массивов данных
const filterBy = <T>(predicate: (item: T) => boolean) => (array: T[]): T[] => 
  array.filter(predicate);

const mapWith = <T, R>(fn: (item: T) => R) => (array: T[]): R[] => 
  array.map(fn);

const sortBy = <T>(keyFn: (item: T) => any) => (array: T[]): T[] => 
  [...array].sort((a, b) => {
    const aKey = keyFn(a);
    const bKey = keyFn(b);
    if (aKey < bKey) return -1;
    if (aKey > bKey) return 1;
    return 0;
  });

interface Product {
  id: number;
  name: string;
  category: string;
  price: number;
  inStock: boolean;
}

const products: Product[] = [
  { id: 1, name: "Ноутбук", category: "Электроника", price: 50000, inStock: true },
  { id: 2, name: "Смартфон", category: "Электроника", price: 30000, inStock: false },
  { id: 3, name: "Книга", category: "Книги", price: 500, inStock: true },
  { id: 4, name: "Планшет", category: "Электроника", price: 20000, inStock: true }
];

// Композиция операций над массивом
const getAvailableElectronics = pipe(
  filterBy((p: Product) => p.inStock),
  filterBy((p: Product) => p.category === "Электроника"),
  sortBy((p: Product) => p.price),
  mapWith((p: Product) => `${p.name} - ${p.price} руб.`)
);

console.log(getAvailableElectronics(products));
// ["Планшет - 20000 руб.", "Ноутбук - 50000 руб."]
```

## Связи с другими концепциями

- [[ts/functional-programming/composition/Композиция_функций|Композиция функций]] - Основная страница раздела
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/functional-programming/currying|Каррирование]] - Техника создания функций с одним аргументом
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация композиции функций
- [[Тестирование в TypeScript|Тестирование]] - Тестирование композиции функций

## Следующие шаги

- [[ts/functional-programming/composition/data-flow|Поток данных]] - Управление потоком данных через композицию
- [[ts/functional-programming/composition/practical-examples|Практические примеры]] - Реальные примеры использования композиции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация композиции функций
- [[Тестирование в TypeScript|Тестирование]] - Тестирование композиции функций
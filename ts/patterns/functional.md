# Functional Programming Patterns in TypeScript

Функциональные паттерны программирования в TypeScript используют концепции функционального программирования для создания более надежного, предсказуемого и поддерживаемого кода. Эти паттерны базируются на использовании чистых функций, неизменяемости и композиции.

## Основные функциональные концепции

### Чистые функции (Pure Functions)
```typescript
// Чистые функции - основа функционального программирования
// Они всегда возвращают одинаковый результат для одинаковых аргументов
// и не имеют побочных эффектов

// Чистая функция
function add(a: number, b: number): number {
  return a + b;
}

// Также чистая функция
const multiply = (a: number, b: number): number => a * b;

// Нечистая функция - зависит от внешнего состояния
let multiplier = 2;
const impureMultiply = (a: number): number => a * multiplier;

// Нечистая функция - имеет побочный эффект
const logAndMultiply = (a: number, b: number): number => {
  console.log(`Multiplying ${a} by ${b}`); // побочный эффект
  return a * b;
};

// Чистая версия
const multiplyWithLog = (a: number, b: number): { result: number; log: string } => ({
  result: a * b,
  log: `Multiplying ${a} by ${b}`
});
```

### Неизменяемость (Immutability)
```typescript
// Работа с неизменяемыми данными
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly preferences: readonly string[];
}

// Чистая функция для обновления пользователя
const updateUser = (user: User, updates: Partial<User>): User => ({
  ...user,
  ...updates,
  // Обработка вложенных объектов
  preferences: updates.preferences ? [...updates.preferences] : [...user.preferences]
});

// Использование
const originalUser: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  preferences: ["email_notifications", "push_notifications"]
};

const updatedUser = updateUser(originalUser, {
  name: "Alice Smith",
  preferences: ["push_notifications"]
});

// originalUser не изменен
console.log(originalUser.name); // "Alice"
console.log(updatedUser.name);  // "Alice Smith"
```

## Продвинутые функциональные паттерны

### Функции высшего порядка
```typescript
// Функции, которые принимают или возвращают другие функции
type Transformer<T, U> = (value: T) => U;
type Predicate<T> = (value: T) => boolean;
type Consumer<T> = (value: T) => void;

// Каррирование
const curry = <A, B, C>(fn: (a: A, b: B) => C): ((a: A) => (b: B) => C) =>
  (a: A) => (b: B) => fn(a, b);

const add = (a: number, b: number): number => a + b;
const curriedAdd = curry(add);
const addFive = curriedAdd(5);
const result = addFive(3); // 8

// Частичное применение
const partial = <T extends any[], U extends any[], R>(
  fn: (...args: [...T, ...U]) => R,
  ...partialArgs: T
): ((...args: U) => R) => 
  (...args: U) => fn(...partialArgs, ...args);

const multiplyBy = partial(multiply, 2); // (b: number) => number
const doubled = multiplyBy(5); // 10

// Композиция функций
const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B): (a: A) => C =>
  (x: A) => f(g(x));

const toUpperCase = (s: string): string => s.toUpperCase();
const getLength = (s: string): number => s.length;
const upperCaseLength = compose(getLength, toUpperCase);

console.log(upperCaseLength("hello")); // 5

// Цепочка функций
const pipe = <T>(value: T, ...fns: Array<(arg: any) => any>): any =>
  fns.reduce((acc, fn) => fn(acc), value);

const processText = (text: string): number =>
  pipe(
    text,
    (s: string) => s.trim(),
    (s: string) => s.toLowerCase(),
    (s: string) => s + "!",
    (s: string) => toUpperCase(s),
    (s: string) => getLength(s)
  );

console.log(processText("  Hello World  ")); // 13
```

### Functor Pattern
```typescript
// Интерфейс функтора
interface Functor<T> {
  map<U>(fn: (value: T) => U): Functor<U>;
}

// Простой функтор Maybe (аналог Option в других языках)
interface Maybe<T> extends Functor<T> {
  isNone(): boolean;
  isSome(): boolean;
  getOrElse(defaultValue: T): T;
  flatMap<U>(fn: (value: T) => Maybe<U>): Maybe<U>;
}

class None<T> implements Maybe<T> {
  map<U>(fn: (value: T) => U): Maybe<U> {
    return new None<U>(); // fmap f nothing = nothing
  }
  
  isNone(): boolean {
    return true;
  }
  
  isSome(): boolean {
    return false;
  }
  
  getOrElse(defaultValue: T): T {
    return defaultValue;
  }
  
  flatMap<U>(fn: (value: T) => Maybe<U>): Maybe<U> {
    return new None<U>();
  }
}

class Some<T> implements Maybe<T> {
  constructor(private value: T) {}
  
  map<U>(fn: (value: T) => U): Maybe<U> {
    return new Some(fn(this.value));
  }
  
  isNone(): boolean {
    return false;
  }
  
  isSome(): boolean {
    return true;
  }
  
  getOrElse(defaultValue: T): T {
    return this.value;
  }
  
  flatMap<U>(fn: (value: T) => Maybe<U>): Maybe<U> {
    return fn(this.value);
  }
}

// Создание Maybe из значения
const maybe = <T>(value: T | null | undefined): Maybe<T> =>
  value == null ? new None<T>() : new Some(value);

// Использование
const processUser = (input: string | null): string => {
  return maybe(input)
    .flatMap(name => name.length > 0 ? new Some(name) : new None<string>())
    .map(name => name.toUpperCase())
    .getOrElse("UNKNOWN_USER");
};

console.log(processUser("alice")); // "ALICE"
console.log(processUser(null));    // "UNKNOWN_USER"
console.log(processUser(""));      // "UNKNOWN_USER"
```

### Either/Result для обработки ошибок
```typescript
// Тип Either для результатов с ошибками
type Either<L, R> = Left<L> | Right<R>;

interface Left<L> {
  readonly type: 'Left';
  readonly value: L;
}

interface Right<R> {
  readonly type: 'Right';
  readonly value: R;
}

const Left = <L, R = never>(value: L): Either<L, R> => ({
  type: 'Left',
  value
});

const Right = <R, L = never>(value: R): Either<L, R> => ({
  type: 'Right',
  value
});

// Функция для безопасного деления
const safeDivide = (dividend: number, divisor: number): Either<string, number> => {
  if (divisor === 0) {
    return Left("Division by zero");
  }
  return Right(dividend / divisor);
};

// Работа с Either
const calculate = (a: number, b: number, c: number): Either<string, number> => {
  return safeDivide(a, b)
    .map(result => safeDivide(result, c))
    .fold(
      error => Left(error), // если ошибка в первой операции
      intermediate => // если первая операция успешна
        safeDivide(intermediate, c) // вторая операция
    );
};

// Использование Either
const result1 = safeDivide(10, 2); // Right(5)
const result2 = safeDivide(10, 0); // Left("Division by zero")

// Обработка результата
const handleResult = <T>(either: Either<string, T>): void => {
  either.fold(
    error => console.error("Error:", error),
    value => console.log("Success:", value)
  );
};

// Расширение интерфейса Either с методами
declare global {
  interface Either<L, R> {
    map<U>(fn: (value: R) => U): Either<L, U>;
    flatMap<U>(fn: (value: R) => Either<L, U>): Either<L, U>;
    fold<U>(leftFn: (l: L) => U, rightFn: (r: R) => U): U;
    getOrElse(defaultValue: R): R;
  }
}

// Реализация методов
// (в реальности это делается через объявление интерфейса в глобальном скоупе)
```

### Монада State
```typescript
// Монада для управления состоянием
class State<S, A> {
  constructor(private runFn: (state: S) => [A, S]) {}
  
  static of<S, A>(value: A): State<S, A> {
    return new State(state => [value, state]);
  }
  
  static get<S>(): State<S, S> {
    return new State(state => [state, state]);
  }
  
  static put<S>(newState: S): State<S, void> {
    return new State(() => [undefined, newState]);
  }
  
  static modify<S>(fn: (state: S) => S): State<S, void> {
    return new State(state => [undefined, fn(state)]);
  }
  
  map<B>(fn: (a: A) => B): State<S, B> {
    return new State(state => {
      const [value, newState] = this.runFn(state);
      return [fn(value), newState];
    });
  }
  
  flatMap<B>(fn: (a: A) => State<S, B>): State<S, B> {
    return new State(state => {
      const [value, newState] = this.runFn(state);
      return fn(value).runFn(newState);
    });
  }
  
  run(initialState: S): [A, S] {
    return this.runFn(initialState);
  }
}

// Пример использования для счетчика
type Counter = number;

const increment: State<Counter, void> = State.modify<Counter>(count => count + 1);
const decrement: State<Counter, void> = State.modify<Counter>(count => count - 1);
const getCount: State<Counter, Counter> = State.get<Counter>();

// Комбинация операций
const counterProgram: State<Counter, number> = increment
  .flatMap(() => increment)
  .flatMap(() => decrement)
  .flatMap(() => getCount.map(val => val * 2));

const [finalValue, finalState] = counterProgram.run(0); // [2, 1]
```

## Практические функциональные утилиты

### Мемоизация
```typescript
// Утилита для мемоизации функций
const memoize = <T extends (...args: any[]) => any>(fn: T): T => {
  const cache = new Map<string, any>();
  
  return function(...args: any[]): any {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  } as T;
};

// Мемоизированная функция Фибоначчи
const fibonacci = (n: number): number => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
};

const memoizedFibonacci = memoize(fibonacci);

// Мемоизация с кастомным ключом
const memoizeWithKey = <T extends (...args: any[]) => any>(
  fn: T,
  keyGenerator: (...args: Parameters<T>) => string = (...args) => JSON.stringify(args)
): T => {
  const cache = new Map<string, any>();
  
  return function(...args: any[]): any {
    const key = keyGenerator(...args as Parameters<T>);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  } as T;
};
```

### Curry и Partial Application
```typescript
// Универсальный каррирующий механизм
type Curry2<T, U, R> = (t: T) => (u: U) => R;
type Curry3<T, U, V, R> = (t: T) => (u: U) => (v: V) => R;
type Curry4<T, U, V, W, R> = (t: T) => (u: U) => (v: V) => (w: W) => R;

// Каррирование для функций с 2 параметрами
const curry2 = <T, U, R>(fn: (t: T, u: U) => R): Curry2<T, U, R> =>
  (t: T) => (u: U) => fn(t, u);

// Каррирование для функций с 3 параметрами
const curry3 = <T, U, V, R>(fn: (t: T, u: U, v: V) => R): Curry3<T, U, V, R> =>
  (t: T) => (u: U) => (v: V) => fn(t, u, v);

// Универсальное каррирование (ограничено)
const createCurry = <T extends Function>(fn: T): any => {
  const arity = fn.length;
  
  return function resolver(...args: any[]) {
    if (args.length >= arity) {
      return fn.apply(this, args);
    }
    
    return function(...nextArgs: any[]) {
      return resolver.apply(this, args.concat(nextArgs));
    };
  };
};

// Использование
const multiplyThree = (a: number, b: number, c: number): number => a * b * c;
const curriedMultiply = createCurry(multiplyThree);

const timesTwo = curriedMultiply(2);          // (b: number, c: number) => number
const timesTwoThree = timesTwo(3);            // (c: number) => number  
const result = timesTwoThree(4);              // 24
```

### Комбинаторы
```typescript
// Базовые комбинаторы
const identity = <T>(x: T): T => x;
const constant = <T>(x: T) => (): T => x;
const flip = <T, U, R>(fn: (a: T, b: U) => R) => (b: U, a: T): R => fn(a, b);

// Комбинатор композиции (B)
const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B): (a: A) => C => 
  (x: A): C => f(g(x));

// Комбинатор аппликации (S)
const apply = <A, B>(f: (a: A) => B, g: A): B => f(g);

// Комбинатор постоянства (K)
const always = <T>(x: T) => <U>(y: U): T => x;

// Использование комбинаторов
const numbers = [1, 2, 3, 4, 5];

// С использованием функций высшего порядка
const result1 = numbers
  .map(x => x * 2)
  .filter(x => x > 4)
  .reduce((acc, x) => acc + x, 0);

// С использованием композиции
const double = (x: number) => x * 2;
const isGreaterThan = (threshold: number) => (x: number) => x > threshold;
const sum = (arr: number[]) => arr.reduce((a, b) => a + b, 0);

const processNumbers = compose(
  compose(
    sum,
    filter(isGreaterThan(4))
  ),
  map(double)
);

// const result2 = processNumbers(numbers);
```

## Функции для работы с массивами

### Функции высшего порядка для массивов
```typescript
// Использование встроенных методов массивов функционально
const map = <T, U>(fn: (item: T) => U) => (arr: T[]): U[] => arr.map(fn);
const filter = <T>(predicate: (item: T) => boolean) => (arr: T[]): T[] => arr.filter(predicate);
const reduce = <T, U>(fn: (acc: U, item: T) => U, initial: U) => (arr: T[]): U => 
  arr.reduce(fn, initial);

// Комбинирование
const doubleAndFilter = (arr: number[]) => 
  pipe(
    arr,
    map(x => x * 2),
    filter(x => x > 10)
  ); // применит сначала map, потом filter

// Функция для группировки
const groupBy = <T, K extends keyof any>(
  keySelector: (item: T) => K
) => (arr: T[]): Record<K, T[]> => {
  return arr.reduce((groups, item) => {
    const key = keySelector(item);
    if (!groups[key]) {
      groups[key] = [];
    }
    groups[key].push(item);
    return groups;
  }, {} as Record<K, T[]>);
};

// Функция для агрегации
interface AggregationConfig<T> {
  selector: (item: T) => number;
  aggregator: (values: number[]) => number;
}

const aggregate = <T>(config: AggregationConfig<T>) => (arr: T[]): number => {
  const values = arr.map(config.selector);
  return config.aggregator(values);
};

// Использование
const users = [
  { name: "Alice", age: 30, salary: 50000 },
  { name: "Bob", age: 25, salary: 45000 },
  { name: "Charlie", age: 35, salary: 60000 }
];

const usersByAge = groupBy((user: typeof users[0]) => 
  user.age < 30 ? "young" : user.age < 40 ? "middle" : "senior"
)(users);

// Результат: { young: [Bob], middle: [Alice, Charlie], senior: [] }

const avgSalary = aggregate({
  selector: (user: typeof users[0]) => user.salary,
  aggregator: (salaries: number[]) => salaries.reduce((a, b) => a + b, 0) / salaries.length
})(users); // 51666.66...
```

### Рекурсивные функции
```typescript
// Рекурсивные функции без побочных эффектов
const factorial = (n: number): number => {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
};

// Хвостовая рекурсия (оптимизируется компилятором)
const factorialTR = (n: number, acc: number = 1): number => {
  if (n <= 1) return acc;
  return factorialTR(n - 1, n * acc);
};

// Обход деревьев функционально
interface TreeNode<T> {
  value: T;
  children: TreeNode<T>[];
}

const traverse = <T>(node: TreeNode<T>, visitor: (value: T) => void): void => {
  visitor(node.value);
  node.children.forEach(child => traverse(child, visitor));
};

// Преобразование дерева
const transformTree = <T, U>(
  node: TreeNode<T>, 
  transformer: (value: T) => U
): TreeNode<U> => ({
  value: transformer(node.value),
  children: node.children.map(child => transformTree(child, transformer))
});

// Поиск в дереве
const findInTree = <T>(
  node: TreeNode<T>, 
  predicate: (value: T) => boolean
): T | undefined => {
  if (predicate(node.value)) {
    return node.value;
  }
  
  for (const child of node.children) {
    const found = findInTree(child, predicate);
    if (found) return found;
  }
  
  return undefined;
};
```

## Практические паттерны

### Скан (scan) - аккумуляция с историей
```typescript
// Функция scan (как reduce, но возвращает промежуточные результаты)
const scan = <T, U>(
  fn: (acc: U, value: T) => U, 
  initial: U
) => (arr: T[]): U[] => {
  const result: U[] = [initial];
  
  for (let i = 0; i < arr.length; i++) {
    const nextValue = fn(result[i], arr[i]);
    result.push(nextValue);
  }
  
  return result.slice(1); // возвращаем без начального значения
};

// Пример: накопление суммы
const numbers = [1, 2, 3, 4, 5];
const cumulativeSum = scan((acc, value) => acc + value, 0)(numbers);
// [1, 3, 6, 10, 15] - накопительные суммы

// Практическое использование: история состояния
interface StateChange {
  type: string;
  payload: any;
}

const applyChanges = (state: any, change: StateChange) => {
  switch (change.type) {
    case 'SET_VALUE':
      return { ...state, [change.payload.key]: change.payload.value };
    case 'INCREMENT':
      return { ...state, [change.payload.key]: (state[change.payload.key] || 0) + 1 };
    default:
      return state;
  }
};

const stateHistory = (changes: StateChange[]) => 
  scan(applyChanges, {})(changes);

const changes: StateChange[] = [
  { type: 'SET_VALUE', payload: { key: 'name', value: 'Alice' } },
  { type: 'SET_VALUE', payload: { key: 'age', value: 30 } },
  { type: 'INCREMENT', payload: { key: 'age' } }
];

const history = stateHistory(changes);
// [
//   { name: 'Alice' },
//   { name: 'Alice', age: 30 },
//   { name: 'Alice', age: 31 }
// ]
```

### Трансдьюсеры (Transducers)
```typescript
// Простой трансдьюсер для эффективной композиции трансформаций
type Reducer<T, U> = (acc: U, value: T) => U;
type Transformer<T> = {
  [Symbol.iterator](): Iterator<T>;
  next(value: T): T;
  result(): T;
};

type Transducer<Input, Output> = <T>(reducer: Reducer<Output, T>) => Reducer<Input, T>;

// Трансдьюсер для map
const mapX = <T, U>(fn: (value: T) => U): Transducer<T, U> => 
  (reducer: Reducer<U, any>) => (acc: any, value: T) => 
    reducer(acc, fn(value));

// Трансдьюсер для filter  
const filterX = <T>(predicate: (value: T) => boolean): Transducer<T, T> =>
  (reducer: Reducer<T, any>) => (acc: any, value: T) =>
    predicate(value) ? reducer(acc, value) : acc;

// Композиция трансдьюсеров
const composeX = <T, U, V>(
  t1: Transducer<U, V>, 
  t2: Transducer<T, U>
): Transducer<T, V> => 
  (reducer: Reducer<V, any>) => t2(t1(reducer));

// Использование трансдьюсеров
const transducer = composeX(
  mapX((x: number) => x * 2),
  filterX((x: number) => x > 0)
);

// Применение к массиву
const reduceWithTransducer = <T, U>(
  transducer: Transducer<T, U>,
  reducer: Reducer<U, any>,
  initial: any,
  source: T[]
): any => {
  const steppedReducer = transducer(reducer);
  return source.reduce(steppedReducer, initial);
};

const result = reduceWithTransducer(
  transducer,
  (acc: number[], val: number) => [...acc, val],
  [],
  [-1, 1, 2, 3]
); // [2, 4, 6] - только положительные, умноженные на 2
```

## Функциональные паттерны для обработки данных

### Обработка коллекций с сохранением типа
```typescript
// Функция для безопасного получения значения из массива
const safeGet = <T>(index: number) => (arr: T[]): T | undefined => {
  return index >= 0 && index < arr.length ? arr[index] : undefined;
};

// Функция для получения первого элемента
const head = <T>(arr: T[]): T | undefined => safeGet(0)(arr);

// Функция для получения хвоста массива (все кроме первого)
const tail = <T>(arr: T[]): T[] => arr.length > 0 ? arr.slice(1) : [];

// Функция для получения последнего элемента
const last = <T>(arr: T[]): T | undefined => 
  arr.length > 0 ? arr[arr.length - 1] : undefined;

// Функция для получения всех кроме последнего
const init = <T>(arr: T[]): T[] => 
  arr.length > 0 ? arr.slice(0, -1) : [];

// Функция для проверки, все ли элементы соответствуют условию
const all = <T>(predicate: (item: T) => boolean) => (arr: T[]): boolean => 
  arr.every(predicate);

// Функция для проверки, есть ли хоть один элемент, соответствующий условию
const any = <T>(predicate: (item: T) => boolean) => (arr: T[]): boolean => 
  arr.some(predicate);

// Использование
const numbers = [1, 2, 3, 4, 5];
const positive = (n: number) => n > 0;

console.log(head(numbers));        // 1
console.log(tail(numbers));        // [2, 3, 4, 5]
console.log(last(numbers));        // 5
console.log(init(numbers));        // [1, 2, 3, 4]
console.log(all(positive)(numbers)); // true
console.log(any(n => n > 4)(numbers)); // true
```

### Функции для работы с опциональными значениями
```typescript
// Утилиты для безопасной работы с undefine/null значениями
const isNotNull = <T>(value: T | null | undefined): value is T => 
  value !== null && value !== undefined;

const isNull = <T>(value: T | null | undefined): value is null | undefined => 
  value === null || value === undefined;

// Безопасное извлечение значения с дефолтным
const withDefault = <T>(defaultValue: T) => (value: T | null | undefined): T => 
  isNotNull(value) ? value : defaultValue;

// Безопасное преобразование
const mapNullable = <T, U>(fn: (value: T) => U) => 
  (value: T | null | undefined): U | null => 
    isNotNull(value) ? fn(value) : null;

// Упрощение цепочки операций с опциональными значениями
interface NullableMonad<T> {
  value: T | null | undefined;
  map<U>(fn: (value: T) => U): NullableMonad<U>;
  flatMap<U>(fn: (value: T) => NullableMonad<U>): NullableMonad<U>;
  orElse(defaultValue: T): T;
}

class Nullable<T> implements NullableMonad<T> {
  constructor(public value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): Nullable<T> {
    return new Nullable(value);
  }
  
  map<U>(fn: (value: T) => U): Nullable<U> {
    return isNotNull(this.value) 
      ? new Nullable(fn(this.value)) 
      : new Nullable(null);
  }
  
  flatMap<U>(fn: (value: T) => NullableMonad<U>): Nullable<U> {
    return isNotNull(this.value) 
      ? fn(this.value) as Nullable<U>
      : new Nullable(null);
  }
  
  orElse(defaultValue: T): T {
    return isNotNull(this.value) ? this.value : defaultValue;
  }
}

// Использование
const result = Nullable.of("hello world")
  .map(str => str.split(" "))
  .map(words => words[0])
  .map(word => word.toUpperCase())
  .orElse("DEFAULT");

console.log(result); // "HELLO"
```

## Функциональное ядро приложения

### Функциональный IoC контейнер
```typescript
// Простая функциональная система внедрения зависимостей
interface DependencyContainer {
  get<T>(token: string): T;
  register<T>(token: string, factory: () => T): void;
  registerSingleton<T>(token: string, factory: () => T): void;
}

// Функциональная реализация контейнера
const createContainer = (): DependencyContainer => {
  const factories = new Map<string, () => any>();
  const singletonInstances = new Map<string, any>();
  
  return {
    register<T>(token: string, factory: () => T): void {
      factories.set(token, factory);
    },
    
    registerSingleton<T>(token: string, factory: () => T): void {
      factories.set(token, () => {
        if (!singletonInstances.has(token)) {
          singletonInstances.set(token, factory());
        }
        return singletonInstances.get(token);
      });
    },
    
    get<T>(token: string): T {
      const factory = factories.get(token);
      if (!factory) {
        throw new Error(`Dependency not found: ${token}`);
      }
      return factory();
    }
  };
};

// Использование
interface Database {
  query(sql: string): any[];
}

interface UserService {
  getUser(id: number): any;
}

const createDb = (): Database => ({
  query: (sql: string) => []
});

const createUserService = (db: Database): UserService => ({
  getUser: (id: number) => db.query(`SELECT * FROM users WHERE id = ${id}`)[0]
});

const container = createContainer();
container.registerSingleton('db', createDb);
container.register('userService', () => createUserService(container.get('db')));

const userService = container.get<UserService>('userService');
```

### Состояние и эффекты функционально
```typescript
// Функциональный подход к управлению состоянием
type StateAction<TState, TPayload = any> = (state: TState, payload?: TPayload) => TState;

interface Store<TState> {
  getState(): TState;
  dispatch<TPayload>(action: StateAction<TState, TPayload>, payload?: TPayload): void;
  subscribe(listener: () => void): () => void;
}

const createStore = <TState>(initialState: TState): Store<TState> => {
  let state = initialState;
  const listeners: Array<() => void> = [];
  
  return {
    getState(): TState {
      return state;
    },
    
    dispatch<TPayload>(action: StateAction<TState, TPayload>, payload?: TPayload): void {
      state = action(state, payload);
      listeners.forEach(listener => listener());
    },
    
    subscribe(listener: () => void): () => void {
      listeners.push(listener);
      
      return () => {
        const index = listeners.indexOf(listener);
        if (index > -1) {
          listeners.splice(index, 1);
        }
      };
    }
  };
};

// Пример использования
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

interface TodoState {
  todos: Todo[];
  filter: 'all' | 'active' | 'completed';
}

const initialTodoState: TodoState = {
  todos: [],
  filter: 'all'
};

const addTodo = (state: TodoState, text: string): TodoState => ({
  ...state,
  todos: [
    ...state.todos,
    { id: Date.now(), text, completed: false }
  ]
});

const toggleTodo = (state: TodoState, id: number): TodoState => ({
  ...state,
  todos: state.todos.map(todo =>
    todo.id === id ? { ...todo, completed: !todo.completed } : todo
  )
});

const store = createStore(initialTodoState);
store.dispatch(addTodo, "Learn functional programming");
```

## Лучшие практики

### 1. Используйте чистые функции везде, где возможно
```typescript
// ПЛОХО: функция зависит от внешнего состояния
let multiplier = 1;
const badMultiply = (x: number) => x * multiplier;

// ХОРОШО: все зависимости передаются явно
const goodMultiply = (x: number, multiplier: number) => x * multiplier;
```

### 2. Предпочтите иммутабельные структуры данных
```typescript
// Используйте spread оператор и другие методы для создания новых объектов
const updateUser = (user: User, updates: Partial<User>) => ({
  ...user,
  ...updates,
  preferences: updates.preferences ? [...updates.preferences] : [...user.preferences]
});
```

### 3. Используйте композицию для создания сложной логики
```typescript
// Композиция простых функций для создания сложной логики
const processUserData = pipe(
  validateUser,
  enrichUserData,
  transformForStorage,
  sanitizeSensitiveData
);
```

### 4. Тестируйте чистые функции легко
```typescript
// Чистые функции легко тестировать
const formatCurrency = (amount: number, currency: string = "USD"): string => 
  new Intl.NumberFormat("en-US", { 
    style: "currency", 
    currency 
  }).format(amount);

// Тестирование тривиально
expect(formatCurrency(1000, "USD")).toBe("$1,000.00");
```

## Современные тенденции

### Функциональные реактивные паттерны
```typescript
// Комбинирование функционального и реактивного программирования
interface Stream<T> {
  map<U>(fn: (value: T) => U): Stream<U>;
  filter(predicate: (value: T) => boolean): Stream<T>;
  subscribe(observer: (value: T) => void): void;
}

class ValueStream<T> implements Stream<T> {
  constructor(private value: T) {}
  
  map<U>(fn: (value: T) => U): ValueStream<U> {
    return new ValueStream(fn(this.value));
  }
  
  filter(predicate: (value: T) => boolean): ValueStream<T> {
    return predicate(this.value) ? new ValueStream(this.value) : new ValueStream(undefined as any);
  }
  
  subscribe(observer: (value: T) => void): void {
    observer(this.value);
  }
}
```

### Использование с современными библиотеками
```typescript
// Использование функциональных утилит с библиотеками
import { pipe } from 'ramda';

const processApiData = pipe(
  filter((item: any) => item.active),
  map((item: any) => ({ ...item, processed: true })),
  reduce((acc, item) => [...acc, item], [])
);
```

## Заключение

Функциональные паттерны программирования в TypeScript позволяют:

1. **Создавать предсказуемый код** - чистые функции не имеют скрытых зависимостей
2. **Обеспечивать легкую тестируемость** - функции не зависят от состояния
3. **Повышать надежность** - неизменяемость предотвращает случайные мутации
4. **Упрощать параллелизм** - отсутствие побочных эффектов упрощает многопоточность
5. **Повышать переиспользуемость** - небольшие чистые функции легко комбинируются

Функциональное программирование не требует отказа от других парадигм - оно может эффективно сочетаться с объектно-ориентированным и императивным подходами в рамках одного проекта.

## Связь с другими концепциями
- [[../type-system/advanced-types]] - типы для функциональных паттернов
- [[../generics/advanced]] - обобщения в функциональных паттернах
- [[../advanced-types/mapped-types]] - сопоставляющие типы для трансформаций
- [[../design-patterns/behavioral-patterns]] - поведенческие паттерны в функциональном стиле
- [[../modules/function-composition]] - модульная композиция функций
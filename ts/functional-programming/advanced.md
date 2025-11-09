# Advanced Functional Programming in TypeScript

Продвинутое функциональное программирование в TypeScript использует более сложные паттерны и концепции для создания надежного, тестируемого и поддерживаемого кода с минимальными побочными эффектами.

## Продвинутые функциональные концепции

### Категории и функторы
```typescript
// Интерфейс функтора
interface Functor<T> {
  map<U>(fn: (value: T) => U): Functor<U>;
}

// Простой функтор для массива
class ArrayFunctor<T> implements Functor<T[]> {
  constructor(private values: T[]) {}
  
  map<U>(fn: (value: T) => U): ArrayFunctor<U[]> {
    return new ArrayFunctor(this.values.map(fn));
  }
  
  get(): T[] {
    return this.values;
  }
}

// Использование
const numbers = new ArrayFunctor([1, 2, 3]);
const doubled = numbers.map(x => x * 2);
console.log(doubled.get()); // [2, 4, 6]

// Функтор Maybe
interface Maybe<T> extends Functor<T> {
  isNothing(): boolean;
  isJust(): boolean;
  map<U>(fn: (value: T) => U): Maybe<U>;
  flatMap<U>(fn: (value: T) => Maybe<U>): Maybe<U>;
}

const Nothing = (): Maybe<never> => ({
  isNothing: () => true,
  isJust: () => false,
  map: () => Nothing(),
  flatMap: () => Nothing(),
  getOrElse: <T>(defaultValue: T): T => defaultValue
}) as Maybe<never>;

const Just = <T>(value: T): Maybe<T> => ({
  isNothing: () => false,
  isJust: () => true,
  map: (fn) => Just(fn(value)),
  flatMap: (fn) => fn(value),
  getOrElse: () => value
}) as Maybe<T>;

interface Maybe<T> {
  isNothing(): boolean;
  isJust(): boolean;
  map<U>(fn: (value: T) => U): Maybe<U>;
  flatMap<U>(fn: (value: T) => Maybe<U>): Maybe<U>;
  getOrElse(defaultValue: T): T;
}

// Практическое использование Maybe
const safeDivide = (a: number, b: number): Maybe<number> => {
  return b === 0 ? Nothing() : Just(a / b);
};

const result = safeDivide(10, 2)
  .map(x => x * 2)  // 10
  .flatMap(x => safeDivide(x, 5)); // Just(2)

console.log(result.getOrElse(0)); // 2
```

### Моноиды и моны
```typescript
// Интерфейс моноида
interface Monoid<T> {
  empty(): T;
  combine(a: T, b: T): T;
}

// Пример: моноид для строк
const StringMonoid: Monoid<string> = {
  empty: () => "",
  combine: (a, b) => a + b
};

// Пример: моноид для чисел (сложение)
const SumMonoid: Monoid<number> = {
  empty: () => 0,
  combine: (a, b) => a + b
};

// Пример: моноид для чисел (умножение)
const ProductMonoid: Monoid<number> = {
  empty: () => 1,
  combine: (a, b) => a * b
};

// Мой монада
class Identity<T> {
  constructor(private value: T) {}
  
  static of<T>(value: T): Identity<T> {
    return new Identity(value);
  }
  
  map<U>(fn: (value: T) => U): Identity<U> {
    return new Identity(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => Identity<U>): Identity<U> {
    return fn(this.value);
  }
  
  get(): T {
    return this.value;
  }
}

// Использование
const result1 = Identity.of(5)
  .map(x => x * 2)
  .map(x => x + 10)
  .get(); // 20

// Цепочка операций
const computation = Identity.of(10)
  .flatMap(x => Identity.of(x * 2))
  .flatMap(x => Identity.of(x + 5))
  .get(); // 25
```

## Иммутабельные структуры данных

### Иммутабельный список
```typescript
interface ImmutableList<T> {
  readonly head: T | undefined;
  readonly tail: ImmutableList<T> | undefined;
  isEmpty(): boolean;
  prepend(value: T): ImmutableList<T>;
  append(value: T): ImmutableList<T>;
  map<U>(fn: (value: T) => U): ImmutableList<U>;
  filter(predicate: (value: T) => boolean): ImmutableList<T>;
  reduce<U>(fn: (acc: U, value: T) => U, initial: U): U;
}

class EmptyList<T> implements ImmutableList<T> {
  readonly head: undefined = undefined;
  readonly tail: undefined = undefined;
  
  isEmpty(): boolean {
    return true;
  }
  
  prepend(value: T): ImmutableList<T> {
    return new ListNode(value, this);
  }
  
  append(value: T): ImmutableList<T> {
    return new ListNode(value, this);
  }
  
  map<U>(fn: (value: T) => U): ImmutableList<U> {
    return new EmptyList<U>();
  }
  
  filter(predicate: (value: T) => boolean): ImmutableList<T> {
    return new EmptyList<T>();
  }
  
  reduce<U>(fn: (acc: U, value: T) => U, initial: U): U {
    return initial;
  }
}

class ListNode<T> implements ImmutableList<T> {
  constructor(
    readonly head: T,
    readonly tail: ImmutableList<T>
  ) {}
  
  isEmpty(): boolean {
    return false;
  }
  
  prepend(value: T): ImmutableList<T> {
    return new ListNode(value, this);
  }
  
  append(value: T): ImmutableList<T> {
    return new ListNode(value, new EmptyList<T>()).prependList(this);
  }
  
  private prependList(other: ImmutableList<T>): ImmutableList<T> {
    if (other.isEmpty()) {
      return this;
    } else {
      return this.prependList(other.tail!).prepend(other.head!);
    }
  }
  
  map<U>(fn: (value: T) => U): ImmutableList<U> {
    return new ListNode(
      fn(this.head),
      this.tail.map(fn)
    );
  }
  
  filter(predicate: (value: T) => boolean): ImmutableList<T> {
    const filteredTail = this.tail.filter(predicate);
    
    if (predicate(this.head)) {
      return new ListNode(this.head, filteredTail);
    } else {
      return filteredTail;
    }
  }
  
  reduce<U>(fn: (acc: U, value: T) => U, initial: U): U {
    return this.tail.reduce(fn, fn(initial, this.head));
  }
}

// Использование
const list = new EmptyList<number>()
  .prepend(3)
  .prepend(2)
  .prepend(1); // [1, 2, 3]

const doubled = list.map(x => x * 2); // [2, 4, 6]
const evens = list.filter(x => x % 2 === 0); // [2]
const sum = list.reduce((acc, x) => acc + x, 0); // 6
```

### Иммутабельное дерево
```typescript
interface ImmutableTree<T> {
  value: T;
  left: ImmutableTree<T> | null;
  right: ImmutableTree<T> | null;
  insert(value: T): ImmutableTree<T>;
  contains(value: T): boolean;
  traverse(): T[];
}

class TreeNode<T> implements ImmutableTree<T> {
  constructor(
    public value: T,
    public left: ImmutableTree<T> | null = null,
    public right: ImmutableTree<T> | null = null
  ) {}
  
  insert(newValue: T): ImmutableTree<T> {
    if (newValue < this.value) {
      return new TreeNode(
        this.value,
        this.left ? this.left.insert(newValue) : new TreeNode(newValue),
        this.right
      );
    } else if (newValue > this.value) {
      return new TreeNode(
        this.value,
        this.left,
        this.right ? this.right.insert(newValue) : new TreeNode(newValue)
      );
    } else {
      // Значение уже существует, возвращаем текущее дерево
      return this;
    }
  }
  
  contains(target: T): boolean {
    if (target === this.value) {
      return true;
    } else if (target < this.value) {
      return this.left ? this.left.contains(target) : false;
    } else {
      return this.right ? this.right.contains(target) : false;
    }
  }
  
  traverse(): T[] {
    const result: T[] = [];
    
    if (this.left) {
      result.push(...this.left.traverse());
    }
    
    result.push(this.value);
    
    if (this.right) {
      result.push(...this.right.traverse());
    }
    
    return result;
  }
}

// Использование
let tree = new TreeNode(5);
tree = tree.insert(3);
tree = tree.insert(7);
tree = tree.insert(2);
tree = tree.insert(4);

console.log(tree.contains(4)); // true
console.log(tree.contains(6)); // false
console.log(tree.traverse()); // [2, 3, 4, 5, 7]
```

## Продвинутые функциональные паттерны

### Композиция функций
```typescript
// Функция композиции (справа налево)
const compose = <T, U, V>(f: (x: U) => V, g: (x: T) => U) => (x: T): V => f(g(x));

// Композиция нескольких функций
const pipe = <T>(value: T, ...fns: Array<(arg: any) => any>): any =>
  fns.reduce((acc, fn) => fn(acc), value);

// Пример использования
const add = (a: number) => (b: number): number => a + b;
const multiply = (a: number) => (b: number): number => a * b;
const power = (exponent: number) => (base: number): number => Math.pow(base, exponent);

// Композиция функций
const calculate = compose(
  compose(power(2), multiply(3)), // (x * 3) ^ 2
  add(5)                          // x + 5
);

// Или с pipe
const resultPipe = pipe(
  2,                // начальное значение
  add(5),          // 2 + 5 = 7
  multiply(3),     // 7 * 3 = 21
  power(2)         // 21^2 = 441
); // 441

console.log(calculate(2)); // ((2 + 5) * 3) ^ 2 = 441
console.log(resultPipe);   // 441
```

### Каррирование и частичное применение
```typescript
// Универсальная функция каррирования
const curry = <T extends any[], R>(
  fn: (...args: T) => R
): Curry<T, R> => 
  function(this: any, ...args: any[]): any {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return curry(
        fn.bind(this, ...args)
      );
    }
  };

type Curry<T extends any[], R> = 
  T extends [infer A] 
    ? (a: A) => R
    : T extends [infer A, infer B] 
      ? (a: A) => (b: B) => R  
      : T extends [infer A, infer B, infer C]
        ? (a: A) => (b: B) => (c: C) => R
        : T extends [infer A, infer B, infer C, infer D]
          ? (a: A) => (b: B) => (c: C) => (d: D) => R
          : T extends [infer A, infer B, infer C, infer D, infer E]
            ? (a: A) => (b: B) => (c: C) => (d: D) => (e: E) => R
            : (...args: T) => R;

// Примеры
const multiply3 = (a: number, b: number, c: number): number => a * b * c;
const curriedMultiply3 = curry(multiply3);

const multiplyBy2 = curriedMultiply3(2);        // (b: number, c: number) => number
const multiplyBy2and3 = multiplyBy2(3);         // (c: number) => number  
const result = multiplyBy2and3(4);              // 2 * 3 * 4 = 24

// Частичное применение
const partial = <T extends any[], U extends any[], R>(
  fn: (...args: [...T, ...U]) => R,
  ...partialArgs: T
) => (...args: U): R => fn(...partialArgs, ...args);

const format = (prefix: string, value: number, suffix: string): string => 
  `${prefix}${value}${suffix}`;

const dollars = partial(format, "$", 0); // (suffix: string) => string
const resultDollars = dollars(".00");    // "$0.00"
```

### Линзы (Lenses)
```typescript
// Простая реализация линз
interface Lens<S, A> {
  get: (state: S) => A;
  set: (value: A, state: S) => S;
}

// Создание линзы для доступа к свойству объекта
const prop = <S, K extends keyof S>(key: K): Lens<S, S[K]> => ({
  get: (obj: S): S[K] => obj[key],
  set: (value: S[K], obj: S): S => ({ ...obj, [key]: value })
});

// Композиция линз
const composeLens = <S, A, B>(
  lens1: Lens<A, B>,
  lens2: Lens<S, A>
): Lens<S, B> => ({
  get: (state: S) => lens1.get(lens2.get(state)),
  set: (value: B, state: S) => lens2.set(lens1.set(value, lens2.get(state)), state)
});

// Пример использования
interface Address {
  street: string;
  city: string;
}

interface Person {
  name: string;
  age: number;
  address: Address;
}

const addressLens = prop<Person, "address">("address");
const cityLens = prop<Address, "city">("city");
const addressCityLens = composeLens(cityLens, addressLens);

const person: Person = {
  name: "Alice",
  age: 30,
  address: {
    street: "123 Main St",
    city: "New York"
  }
};

// Чтение
const city = addressCityLens.get(person); // "New York"

// Изменение
const updatedPerson = addressCityLens.set("Los Angeles", person);
// { name: "Alice", age: 30, address: { street: "123 Main St", city: "Los Angeles" } }

console.log(updatedPerson.address.city); // "Los Angeles"
```

## Функциональное реактивное программирование (FRP)

### Простые монады-обертки
```typescript
// Монада для асинхронных вычислений (упрощенная версия Future/Eff)
class Future<T> {
  constructor(private promise: Promise<T>) {}
  
  static of<T>(value: T): Future<T> {
    return new Future(Promise.resolve(value));
  }
  
  static from<T>(promise: Promise<T>): Future<T> {
    return new Future(promise);
  }
  
  map<U>(fn: (value: T) => U): Future<U> {
    return new Future(this.promise.then(fn));
  }
  
  flatMap<U>(fn: (value: T) => Future<U>): Future<U> {
    return new Future(this.promise.then(value => fn(value).promise));
  }
  
  run(): Promise<T> {
    return this.promise;
  }
}

// Использование
const asyncCalculation = Future.from(fetch('/api/data'))
  .flatMap(response => Future.from(response.json()))
  .map(data => data.items.length);

asyncCalculation.run().then(length => {
  console.log(`Items count: ${length}`);
});
```

### Монада Reader для зависимостей
```typescript
// Монада Reader для внедрения зависимостей
class Reader<R, A> {
  constructor(private runFn: (env: R) => A) {}
  
  static of<R, A>(value: A): Reader<R, A> {
    return new Reader(() => value);
  }
  
  map<B>(fn: (value: A) => B): Reader<R, B> {
    return new Reader((env: R) => fn(this.runFn(env)));
  }
  
  flatMap<B>(fn: (value: A) => Reader<R, B>): Reader<R, B> {
    return new Reader((env: R) => fn(this.runFn(env)).runFn(env));
  }
  
  run(env: R): A {
    return this.runFn(env);
  }
  
  local<B extends R>(transform: (env: R) => B): Reader<B, A> {
    return new Reader((env: B) => this.runFn(transform(env)));
  }
}

// Пример использования для конфигурации
interface AppConfig {
  apiUrl: string;
  timeout: number;
  debug: boolean;
}

const configReader = Reader.of<AppConfig, AppConfig>({} as AppConfig);

// Функция, которая зависит от конфигурации
const getApiUrl = new Reader<AppConfig, string>(config => config.apiUrl);

const timeoutReader = new Reader<AppConfig, number>(config => config.timeout);

// Комбинация вычислений
const appLogic = getApiUrl.flatMap(url => 
  timeoutReader.map(timeout => `${url} with ${timeout}ms timeout`)
);

// Выполнение с реальной конфигурацией
const config: AppConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  debug: true
};

const result = appLogic.run(config);
console.log(result); // "https://api.example.com with 5000ms timeout"
```

## Практические применения

### Валидация с использованием Either
```typescript
// Тип Either для обработки ошибок
type Either<L, R> = Left<L> | Right<R>;

interface Left<L> {
  readonly type: 'Left';
  readonly value: L;
}

interface Right<R> {
  readonly type: 'Right';
  readonly value: R;
}

const Left = <L>(value: L): Either<L, never> => ({
  type: 'Left',
  value
});

const Right = <R>(value: R): Either<never, R> => ({
  type: 'Right',
  value
});

// Простая система валидации
interface ValidationRules<T> {
  [K in keyof T]: (value: T[K]) => boolean;
}

interface ValidationResult<T> {
  success: boolean;
  errors: string[];
  data: T | null;
}

// Валидация с использованием Either
const validateField = <T, K extends keyof T>(
  obj: T,
  field: K,
  validator: (value: T[K]) => boolean,
  errorMessage: string
): Either<string[], T[K]> => {
  if (validator(obj[field])) {
    return Right(obj[field]);
  } else {
    return Left([errorMessage]);
  }
};

// Комбинирование валидаций
const combineValidations = <T>(
  validations: Either<string, T>[]
): Either<string[], T[]> => {
  const errors: string[] = [];
  const values: T[] = [];
  
  for (const validation of validations) {
    if (validation.type === 'Left') {
      errors.push(...validation.value);
    } else {
      values.push(validation.value);
    }
  }
  
  return errors.length > 0 ? Left(errors) : Right(values);
};

// Пример использования
interface User {
  name: string;
  age: number;
  email: string;
}

const user: User = { name: "", age: -5, email: "invalid-email" };

const validations = [
  validateField(user, "name", (n) => n.length > 0, "Name is required"),
  validateField(user, "age", (a) => a >= 0 && a <= 150, "Age must be between 0 and 150"),
  validateField(user, "email", (e) => e.includes("@"), "Email must contain @")
];

const validationResult = combineValidations(validations);

if (validationResult.type === 'Right') {
  console.log("Valid:", validationResult.value);
} else {
  console.log("Errors:", validationResult.value);
  // ["Name is required", "Age must be between 0 and 150", "Email must contain @"]
}
```

### Генераторы и функциональные эффекты
```typescript
// Использование генераторов для управления эффектами
type Effect<T> = () => T;

class IO<T> {
  constructor(private effect: Effect<T>) {}
  
  static of<T>(value: T): IO<T> {
    return new IO(() => value);
  }
  
  static from<T>(effect: Effect<T>): IO<T> {
    return new IO(effect);
  }
  
  map<U>(fn: (value: T) => U): IO<U> {
    return new IO(() => fn(this.effect()));
  }
  
  flatMap<U>(fn: (value: T) => IO<U>): IO<U> {
    return new IO(() => fn(this.effect()).effect());
  }
  
  run(): T {
    return this.effect();
  }
}

// Пример чистой функции для работы с DOM
const querySelectorIO = <T extends Element>(selector: string): IO<T | null> =>
  IO.from(() => document.querySelector(selector) as T | null);

const setTextContentIO = (element: HTMLElement, text: string): IO<void> =>
  IO.from(() => { element.textContent = text; });

// Композиция эффектов
const updateHeader = (newText: string) =>
  querySelectorIO<HTMLHeadingElement>('h1')
    .flatMap(header => {
      if (header) {
        return setTextContentIO(header, newText);
      } else {
        return IO.of(void 0);
      }
    });

// Выполнение эффекта (только в конце цепочки для безопасности)
// updateHeader("New Title").run(); // Только при вызове run() происходит эффект
```

## Лучшие практики

### Иммутабельность
```typescript
// Всегда возвращайте новые объекты, а не изменяйте существующие
const updateObject = <T extends Record<string, any>>(
  obj: T, 
  updates: Partial<T>
): T => ({
  ...obj,
  ...updates
});

// Использование Object.freeze для дополнительной защиты
const makeImmutable = <T>(obj: T): Readonly<T> => Object.freeze(obj);

// Пример безопасной мутации (на самом деле копирования)
interface State {
  items: string[];
  count: number;
}

const addItem = (state: State, item: string): State => ({
  ...state,
  items: [...state.items, item],
  count: state.count + 1
});

const removeItem = (state: State, index: number): State => ({
  ...state,
  items: [
    ...state.items.slice(0, index),
    ...state.items.slice(index + 1)
  ],
  count: state.count - 1
});
```

### Чистые функции
```typescript
// Функции должны быть детерминистичными и не иметь побочных эффектов
const pureCalculate = (a: number, b: number, operation: 'add' | 'mul'): number => {
  switch (operation) {
    case 'add': return a + b;
    case 'mul': return a * b;
  }
};

// Избегайте функций с побочными эффектами
// ПЛОХО:
const badCalculate = (a: number, b: number): number => {
  console.log("Calculating..."); // побочный эффект
  const result = a + b;
  localStorage.setItem('lastCalc', result.toString()); // побочный эффект
  return result;
};

// ХОРОШО:
const goodCalculate = (a: number, b: number): number => a + b;
```

### Типобезопасные функции
```typescript
// Используйте конкретные типы вместо any
type OperationType = 'add' | 'subtract' | 'multiply' | 'divide';

// Более безопасно чем string
const performOperation = (
  a: number, 
  b: number, 
  op: OperationType
): number => {
  switch (op) {
    case 'add': return a + b;
    case 'subtract': return a - b;
    case 'multiply': return a * b;
    case 'divide': 
      if (b === 0) throw new Error("Division by zero");
      return a / b;
  }
};
```

## Заключение

Продвинутое функциональное программирование в TypeScript позволяет:

1. **Создавать чистый код** с минимальными побочными эффектами
2. **Обеспечивать безопасность типов** в функциональных паттернах
3. **Создавать переиспользуемые утилиты** через композицию
4. **Легко тестировать функции** благодаря их чистоте
5. **Создавать надежные системы** с помощью монад и функторов

Хотя функциональный подход может показаться сложнее в изучении, он предоставляет мощные инструменты для создания надежного и поддерживаемого кода.

## Связь с другими концепциями
- [[../types/functional-types]] - функциональные типы в TypeScript  
- [[../advanced-types/conditional-types]] - условные типы для функций
- [[../generics/advanced-patterns]] - обобщенные функциональные паттерны
- [[../patterns/functional]] - паттерны функционального программирования
# Распространенные монады

В функциональном программировании существует множество стандартных монад, каждая из которых решает определенные задачи по управлению эффектами вычислений. Рассмотрим наиболее распространенные монады и их применение.

## Содержание

- [Распространенные монады](#распространенные-монады)
  - [Содержание](#содержание)
  - [Maybe/Option монада](#maybeoption-монада)
  - [Either монада](#either-монада)
  - [IO монада](#io-монада)
  - [State монада](#state-монада)
  - [Reader монада](#reader-монада)
  - [Writer монада](#writer-монада)
  - [List монада](#list-монада)
  - [Future/Promise монада](#futurepromise-монада)
  - [Практические примеры](#практические-примеры)
  - [Выбор подходящей монады](#выбор-подходящей-монады)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Maybe/Option монада

Maybe (в Haskell) или Option (в других языках) монада используется для работы с потенциально отсутствующими значениями, избегая null pointer exceptions.

```typescript
// Maybe монада
type Maybe<T> = T | null | undefined;

class MaybeMonad<T> {
  private constructor(private value: T | null | undefined) {}
  
  static of<T>(value: T | null | undefined): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static just<T>(value: T): MaybeMonad<T> {
    return new MaybeMonad(value);
  }
  
  static nothing<T>(): MaybeMonad<T> {
    return new MaybeMonad<T>(null);
  }
  
  map<U>(fn: (value: T) => U): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return new MaybeMonad<U>(fn(this.value));
  }
  
  flatMap<U>(fn: (value: T) => MaybeMonad<U>): MaybeMonad<U> {
    if (this.value === null || this.value === undefined) {
      return new MaybeMonad<U>(null);
    }
    return fn(this.value);
  }
  
  getValue(): T | null | undefined {
    return this.value;
  }
  
  isNothing(): boolean {
    return this.value === null || this.value === undefined;
  }
  
  isJust(): boolean {
    return !this.isNothing();
  }
  
  orElse(defaultValue: T): T {
    return this.isNothing() ? defaultValue : this.value!;
  }
  
  filter(predicate: (value: T) => boolean): MaybeMonad<T> {
    if (this.isNothing() || !predicate(this.value!)) {
      return MaybeMonad.nothing();
    }
    return this;
  }
}

// Примеры использования Maybe монады
const safeHead = <T>(arr: T[]): MaybeMonad<T> => {
  return arr.length > 0 ? MaybeMonad.just(arr[0]) : MaybeMonad.nothing();
};

const safePropertyAccess = <T, K extends keyof T>(obj: T, key: K): MaybeMonad<T[K]> => {
  return obj ? MaybeMonad.of(obj[key]) : MaybeMonad.nothing();
};

// Композиция безопасных операций
const getUserCity = (users: any[]): MaybeMonad<string> => {
  return safeHead(users)
    .flatMap(user => safePropertyAccess(user, 'address'))
    .flatMap(address => safePropertyAccess(address, 'city'));
};

// Использование
const users = [
  { name: "Иван", address: { city: "Москва" } },
  { name: "Мария" }
];

const city1 = getUserCity(users);
const city2 = getUserCity([]);

console.log(city1.getValue()); // "Москва"
console.log(city2.getValue()); // null
```

## Either монада

Either монада используется для обработки ошибок, где Left представляет ошибку, а Right - успешный результат.

```typescript
// Either монада
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

class EitherMonad<E, A> {
  private constructor(private value: Either<E, A>) {}
  
  static of<E, A>(value: Either<E, A>): EitherMonad<E, A> {
    return new EitherMonad(value);
  }
  
  static right<E, A>(a: A): EitherMonad<E, A> {
    return new EitherMonad(right(a));
  }
  
  static left<E, A>(e: E): EitherMonad<E, A> {
    return new EitherMonad(left(e));
  }
  
  map<B>(fn: (a: A) => B): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return new EitherMonad(right(fn(this.value.right)));
  }
  
  flatMap<B>(fn: (a: A) => EitherMonad<E, B>): EitherMonad<E, B> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(this.value.left));
    }
    return fn(this.value.right);
  }
  
  getValue(): Either<E, A> {
    return this.value;
  }
  
  isLeft(): boolean {
    return this.value._tag === "Left";
  }
  
  isRight(): boolean {
    return this.value._tag === "Right";
  }
  
  mapLeft<F>(fn: (e: E) => F): EitherMonad<F, A> {
    if (this.value._tag === "Left") {
      return new EitherMonad(left(fn(this.value.left)));
    }
    return new EitherMonad(right(this.value.right));
  }
  
  fold<B>(onLeft: (e: E) => B, onRight: (a: A) => B): B {
    return this.value._tag === "Left" 
      ? onLeft(this.value.left) 
      : onRight(this.value.right);
  }
  
  // Дополнительные методы для удобства
  orElse<B>(other: EitherMonad<E, B>): EitherMonad<E, A | B> {
    if (this.isRight()) {
      return this as any;
    }
    return other;
  }
}

// Примеры использования Either монады
const parseJSON = (str: string): EitherMonad<Error, any> => {
  try {
    return EitherMonad.right(JSON.parse(str));
  } catch (error) {
    return EitherMonad.left(new Error(`Ошибка парсинга JSON: ${error}`));
  }
};

const validateUser = (data: any): EitherMonad<string, { name: string; email: string }> => {
  if (!data.name || !data.email) {
    return EitherMonad.left("Отсутствуют обязательные поля: name или email");
  }
  
  if (!data.email.includes("@")) {
    return EitherMonad.left("Неверный формат email");
  }
  
  return EitherMonad.right({
    name: data.name,
    email: data.email
  });
};

const processUserData = (jsonStr: string): EitherMonad<Error | string, { name: string; email: string }> => {
  return parseJSON(jsonStr)
    .flatMap(validateUser);
};

// Использование
const validJson = '{"name": "Иван", "email": "ivan@example.com"}';
const invalidJson = '{"name": "Иван"}';
const malformedJson = '{"name": "Иван", "email":}';

console.log(processUserData(validJson).getValue());
// { _tag: "Right", right: { name: "Иван", email: "ivan@example.com" } }

console.log(processUserData(invalidJson).getValue());
// { _tag: "Left", left: "Отсутствуют обязательные поля: name или email" }

console.log(processUserData(malformedJson).getValue());
// { _tag: "Left", left: Error: Ошибка парсинга JSON: ... }
```

## IO монада

IO монада используется для работы с побочными эффектами, делая их явными и управляемыми.

```typescript
// IO монада
class IOMonad<A> {
  constructor(private computation: () => A) {}
  
  static of<A>(value: A): IOMonad<A> {
    return new IOMonad(() => value);
  }
  
  static from<A>(computation: () => A): IOMonad<A> {
    return new IOMonad(computation);
  }
  
  map<B>(fn: (a: A) => B): IOMonad<B> {
    return new IOMonad(() => fn(this.computation()));
  }
  
  flatMap<B>(fn: (a: A) => IOMonad<B>): IOMonad<B> {
    return new IOMonad(() => fn(this.computation()).computation());
  }
  
  run(): A {
    return this.computation();
  }
  
  // Дополнительные методы для удобства
  tap(fn: (a: A) => void): IOMonad<A> {
    return new IOMonad(() => {
      const result = this.computation();
      fn(result);
      return result;
    });
  }
}

// Примеры использования IO монады
const readLine = (): IOMonad<string> => {
  return IOMonad.from(() => {
    // В реальном приложении здесь был бы реальный ввод
    return "Привет, мир!";
  });
};

const writeLine = (message: string): IOMonad<void> => {
  return IOMonad.from(() => {
    console.log(message);
  });
};

const getCurrentTime = (): IOMonad<Date> => {
  return IOMonad.from(() => new Date());
};

const formatTime = (date: Date): string => {
  return date.toISOString();
};

// Композиция IO операций
const program = readLine()
  .map(str => str.toUpperCase())
  .flatMap(str => writeLine(`Обработанная строка: ${str}`))
  .flatMap(() => getCurrentTime())
  .map(formatTime)
  .flatMap(time => writeLine(`Текущее время: ${time}`));

// Выполнение программы
program.run();
```

## State монада

State монада используется для работы с изменяемым состоянием в функциональном стиле.

```typescript
// State монада
class StateMonad<S, A> {
  constructor(private run: (state: S) => [A, S]) {}
  
  static of<S, A>(value: A): StateMonad<S, A> {
    return new StateMonad(state => [value, state]);
  }
  
  static get<S>(): StateMonad<S, S> {
    return new StateMonad(state => [state, state]);
  }
  
  static put<S>(state: S): StateMonad<S, void> {
    return new StateMonad(() => [undefined, state]);
  }
  
  static modify<S>(fn: (state: S) => S): StateMonad<S, void> {
    return new StateMonad(state => [undefined, fn(state)]);
  }
  
  map<B>(fn: (a: A) => B): StateMonad<S, B> {
    return new StateMonad(state => {
      const [value, newState] = this.run(state);
      return [fn(value), newState];
    });
  }
  
  flatMap<B>(fn: (a: A) => StateMonad<S, B>): StateMonad<S, B> {
    return new StateMonad(state => {
      const [value, newState] = this.run(state);
      return fn(value).run(newState);
    });
  }
  
  evaluate(initialState: S): A {
    const [result] = this.run(initialState);
    return result;
  }
  
  execute(initialState: S): S {
    const [, newState] = this.run(initialState);
    return newState;
  }
}

// Примеры использования State монады
interface CounterState {
  count: number;
  history: number[];
}

const increment = (): StateMonad<CounterState, number> => {
  return StateMonad.modify<CounterState>(state => ({
    count: state.count + 1,
    history: [...state.history, state.count + 1]
  })).flatMap(() => StateMonad.get<CounterState>()).map(state => state.count);
};

const decrement = (): StateMonad<CounterState, number> => {
  return StateMonad.modify<CounterState>(state => ({
    count: state.count - 1,
    history: [...state.history, state.count - 1]
  })).flatMap(() => StateMonad.get<CounterState>()).map(state => state.count);
};

const getCount = (): StateMonad<CounterState, number> => {
  return StateMonad.get<CounterState>().map(state => state.count);
};

// Композиция операций со состоянием
const counterProgram = StateMonad.of<CounterState, void>(undefined)
  .flatMap(() => increment())
  .flatMap(() => increment())
  .flatMap(() => decrement())
  .flatMap(() => getCount());

// Выполнение программы
const initialState: CounterState = { count: 0, history: [] };
const result = counterProgram.evaluate(initialState);
const finalState = counterProgram.execute(initialState);

console.log("Результат:", result); // 1
console.log("Финальное состояние:", finalState); // { count: 1, history: [1, 2, 1] }
```

## Reader монада

Reader монада используется для работы с неизменяемым окружением, которое передается через всю цепочку вычислений.

```typescript
// Reader монада
class ReaderMonad<R, A> {
  constructor(private run: (env: R) => A) {}
  
  static of<R, A>(value: A): ReaderMonad<R, A> {
    return new ReaderMonad(() => value);
  }
  
  static ask<R>(): ReaderMonad<R, R> {
    return new ReaderMonad(env => env);
  }
  
  static local<R1, R2, A>(fn: (r1: R1) => R2, reader: ReaderMonad<R2, A>): ReaderMonad<R1, A> {
    return new ReaderMonad(env => reader.run(fn(env)));
  }
  
  map<B>(fn: (a: A) => B): ReaderMonad<R, B> {
    return new ReaderMonad(env => fn(this.run(env)));
  }
  
  flatMap<B>(fn: (a: A) => ReaderMonad<R, B>): ReaderMonad<R, B> {
    return new ReaderMonad(env => fn(this.run(env)).run(env));
  }
  
  runReader(env: R): A {
    return this.run(env);
  }
}

// Примеры использования Reader монады
interface Config {
  apiUrl: string;
  apiKey: string;
  timeout: number;
}

interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
}

interface AppEnvironment {
  config: Config;
  database: DatabaseConfig;
}

const getApiUrl = (): ReaderMonad<AppEnvironment, string> => {
  return ReaderMonad.ask<AppEnvironment>().map(env => env.config.apiUrl);
};

const getDatabaseHost = (): ReaderMonad<AppEnvironment, string> => {
  return ReaderMonad.ask<AppEnvironment>().map(env => env.database.host);
};

const createApiRequest = (endpoint: string): ReaderMonad<AppEnvironment, string> => {
  return ReaderMonad.ask<AppEnvironment>().map(env => 
    `${env.config.apiUrl}${endpoint}?apiKey=${env.config.apiKey}`
  );
};

// Композиция операций с окружением
const appProgram = ReaderMonad.of<AppEnvironment, string>("Начало")
  .flatMap(() => getApiUrl())
  .flatMap(url => ReaderMonad.of(`URL API: ${url}`))
  .flatMap(message => getDatabaseHost().map(host => `${message}, Хост БД: ${host}`));

// Выполнение программы с окружением
const environment: AppEnvironment = {
  config: {
    apiUrl: "https://api.example.com",
    apiKey: "secret-key",
    timeout: 5000
  },
  database: {
    host: "localhost",
    port: 5432,
    username: "admin"
  }
};

const result = appProgram.runReader(environment);
console.log(result); // "URL API: https://api.example.com, Хост БД: localhost"
```

## Writer монада

Writer монада используется для накопления дополнительной информации (например, логов) во время вычислений.

```typescript
// Writer монада
class WriterMonad<W, A> {
  constructor(private value: A, private log: W[]) {}
  
  static of<W, A>(value: A): WriterMonad<W, A> {
    return new WriterMonad(value, []);
  }
  
  static tell<W>(log: W): WriterMonad<W, void> {
    return new WriterMonad(undefined, [log]);
  }
  
  map<B>(fn: (a: A) => B): WriterMonad<W, B> {
    return new WriterMonad(fn(this.value), this.log);
  }
  
  flatMap<B>(fn: (a: A) => WriterMonad<W, B>): WriterMonad<W, B> {
    const result = fn(this.value);
    return new WriterMonad(result.value, [...this.log, ...result.log]);
  }
  
  runWriter(): [A, W[]] {
    return [this.value, this.log];
  }
  
  execWriter(): W[] {
    return this.log;
  }
  
  mapWriter<V, B>(fn: (a: A, w: W[]) => [B, V[]]): WriterMonad<V, B> {
    const [newValue, newLog] = fn(this.value, this.log);
    return new WriterMonad(newValue, newLog);
  }
}

// Примеры использования Writer монады
type LogEntry = string;

const add = (a: number, b: number): WriterMonad<LogEntry, number> => {
  return WriterMonad.of(a + b)
    .flatMap(result => 
      WriterMonad.tell<LogEntry>(`Сложение: ${a} + ${b} = ${result}`)
        .map(() => result)
    );
};

const multiply = (a: number, b: number): WriterMonad<LogEntry, number> => {
  return WriterMonad.of(a * b)
    .flatMap(result => 
      WriterMonad.tell<LogEntry>(`Умножение: ${a} * ${b} = ${result}`)
        .map(() => result)
    );
};

const divide = (a: number, b: number): WriterMonad<LogEntry, number> => {
  if (b === 0) {
    return WriterMonad.tell<LogEntry>(`Ошибка: деление на ноль ${a} / ${b}`)
      .map(() => NaN);
  }
  
  return WriterMonad.of(a / b)
    .flatMap(result => 
      WriterMonad.tell<LogEntry>(`Деление: ${a} / ${b} = ${result}`)
        .map(() => result)
    );
};

// Композиция операций с логированием
const calculationProgram = WriterMonad.of<LogEntry, number>(10)
  .flatMap(x => add(x, 5))
  .flatMap(x => multiply(x, 2))
  .flatMap(x => divide(x, 3));

// Выполнение программы
const [result, log] = calculationProgram.runWriter();

console.log("Результат:", result); // 10
console.log("Лог:");
log.forEach(entry => console.log("  ", entry));
// Лог:
//   Сложение: 10 + 5 = 15
//   Умножение: 15 * 2 = 30
//   Деление: 30 / 3 = 10
```

## List монада

List монада используется для работы с недетерминированными вычислениями, где каждая операция может вернуть несколько результатов.

```typescript
// List монада
class ListMonad<A> {
  constructor(private values: A[]) {}
  
  static of<A>(value: A): ListMonad<A> {
    return new ListMonad([value]);
  }
  
  static fromArray<A>(values: A[]): ListMonad<A> {
    return new ListMonad(values);
  }
  
  static empty<A>(): ListMonad<A> {
    return new ListMonad([]);
  }
  
  map<B>(fn: (a: A) => B): ListMonad<B> {
    return new ListMonad(this.values.map(fn));
  }
  
  flatMap<B>(fn: (a: A) => ListMonad<B>): ListMonad<B> {
    return new ListMonad(
      this.values.flatMap(value => fn(value).values)
    );
  }
  
  runList(): A[] {
    return this.values;
  }
  
  isEmpty(): boolean {
    return this.values.length === 0;
  }
  
  head(): A | undefined {
    return this.values[0];
  }
  
  tail(): ListMonad<A> {
    return new ListMonad(this.values.slice(1));
  }
}

// Примеры использования List монады
const numbers = ListMonad.fromArray([1, 2, 3]);
const letters = ListMonad.fromArray(['a', 'b', 'c']);

// Декартово произведение
const pairs = numbers.flatMap(n => 
  letters.map(l => [n, l])
);

console.log(pairs.runList());
// [[1, 'a'], [1, 'b'], [1, 'c'], [2, 'a'], [2, 'b'], [2, 'c'], [3, 'a'], [3, 'b'], [3, 'c']]

// Недетерминированные вычисления
const safeDivideAll = (dividends: number[], divisor: number): ListMonad<number> => {
  if (divisor === 0) {
    return ListMonad.empty();
  }
  
  return ListMonad.fromArray(dividends.map(d => d / divisor));
};

const operations = ListMonad.of([10, 20, 30])
  .flatMap(numbers => safeDivideAll(numbers, 2))
  .flatMap(result => ListMonad.fromArray([result, result * 2]));

console.log(operations.runList()); // [5, 10, 10, 20, 15, 30]
```

## Future/Promise монада

Future/Promise монада используется для работы с асинхронными вычислениями.

```typescript
// Future монада (упрощенная реализация)
class FutureMonad<A> {
  constructor(private computation: () => Promise<A>) {}
  
  static of<A>(value: A): FutureMonad<A> {
    return new FutureMonad(() => Promise.resolve(value));
  }
  
  static fromPromise<A>(promise: Promise<A>): FutureMonad<A> {
    return new FutureMonad(() => promise);
  }
  
  map<B>(fn: (a: A) => B): FutureMonad<B> {
    return new FutureMonad(() => this.computation().then(fn));
  }
  
  flatMap<B>(fn: (a: A) => FutureMonad<B>): FutureMonad<B> {
    return new FutureMonad(() => 
      this.computation().then(a => fn(a).computation())
    );
  }
  
  runFuture(): Promise<A> {
    return this.computation();
  }
  
  // Дополнительные методы для удобства
  tap(fn: (a: A) => void): FutureMonad<A> {
    return new FutureMonad(() => 
      this.computation().then(a => {
        fn(a);
        return a;
      })
    );
  }
  
  catchError<B>(fn: (error: any) => B): FutureMonad<A | B> {
    return new FutureMonad(() => 
      this.computation().catch(fn)
    );
  }
}

// Примеры использования Future монады
const fetchUserData = (id: number): FutureMonad<any> => {
  return FutureMonad.fromPromise(
    new Promise(resolve => {
      setTimeout(() => {
        resolve({ id, name: `Пользователь ${id}`, email: `user${id}@example.com` });
      }, 100);
    })
  );
};

const fetchUserPosts = (userId: number): FutureMonad<any[]> => {
  return FutureMonad.fromPromise(
    new Promise(resolve => {
      setTimeout(() => {
        resolve([
          { id: 1, userId, title: "Пост 1" },
          { id: 2, userId, title: "Пост 2" }
        ]);
      }, 150);
    })
  );
};

const processUserWithPosts = (userId: number): FutureMonad<any> => {
  return fetchUserData(userId)
    .flatMap(user => 
      fetchUserPosts(user.id)
        .map(posts => ({ ...user, posts }))
    )
    .tap(result => console.log("Обработанный пользователь:", result))
    .catchError(error => ({ error: `Ошибка обработки пользователя: ${error}` }));
};

// Выполнение асинхронной программы
processUserWithPosts(1)
  .runFuture()
  .then(result => {
    console.log("Финальный результат:", result);
  });
```

## Практические примеры

Рассмотрим комплексный пример использования различных монад в реальном приложении.

```typescript
// Комплексный пример: обработка заказа в интернет-магазине
interface Product {
  id: number;
  name: string;
  price: number;
  inStock: boolean;
}

interface OrderItem {
  productId: number;
  quantity: number;
}

interface Order {
  id: number;
  items: OrderItem[];
  userId: number;
}

interface User {
  id: number;
  name: string;
  email: string;
  address?: string;
}

// Состояние приложения
interface AppState {
  products: Product[];
  users: User[];
  orders: Order[];
}

// Использование различных монад для обработки заказа
const findProduct = (productId: number): StateMonad<AppState, MaybeMonad<Product>> => {
  return StateMonad.get<AppState>().map(state => {
    const product = state.products.find(p => p.id === productId);
    return MaybeMonad.of(product);
  });
};

const findUser = (userId: number): StateMonad<AppState, MaybeMonad<User>> => {
  return StateMonad.get<AppState>().map(state => {
    const user = state.users.find(u => u.id === userId);
    return MaybeMonad.of(user);
  });
};

const validateStock = (product: Product, quantity: number): EitherMonad<string, Product> => {
  if (!product.inStock) {
    return EitherMonad.left(`Товар "${product.name}" отсутствует на складе`);
  }
  
  if (quantity <= 0) {
    return EitherMonad.left("Количество должно быть положительным");
  }
  
  return EitherMonad.right(product);
};

const calculateTotal = (items: OrderItem[]): StateMonad<AppState, EitherMonad<string, number>> => {
  return StateMonad.get<AppState>().map(state => {
    let total = 0;
    
    for (const item of items) {
      const product = state.products.find(p => p.id === item.productId);
      if (!product) {
        return EitherMonad.left(`Товар с ID ${item.productId} не найден`);
      }
      
      if (!product.inStock) {
        return EitherMonad.left(`Товар "${product.name}" отсутствует на складе`);
      }
      
      total += product.price * item.quantity;
    }
    
    return EitherMonad.right(total);
  });
};

const createOrder = (userId: number, items: OrderItem[]): StateMonad<AppState, EitherMonad<string, Order>> => {
  return findUser(userId).flatMap(maybeUser => {
    if (maybeUser.isNothing()) {
      return StateMonad.of(EitherMonad.left(`Пользователь с ID ${userId} не найден`));
    }
    
    return calculateTotal(items).map(eitherTotal => {
      if (eitherTotal.isLeft()) {
        return eitherTotal;
      }
      
      const total = eitherTotal.getValue()._tag === "Right" ? eitherTotal.getValue().right : 0;
      
      // Создание заказа
      const newOrder: Order = {
        id: Date.now(), // Простой способ генерации ID
        items,
        userId
      };
      
      return EitherMonad.right(newOrder);
    });
  });
};

// Пример использования
const initialState: AppState = {
  products: [
    { id: 1, name: "Ноутбук", price: 50000, inStock: true },
    { id: 2, name: "Смартфон", price: 30000, inStock: false },
    { id: 3, name: "Книга", price: 500, inStock: true }
  ],
  users: [
    { id: 1, name: "Иван", email: "ivan@example.com" },
    { id: 2, name: "Мария", email: "maria@example.com", address: "Москва" }
  ],
  orders: []
};

const orderItems: OrderItem[] = [
  { productId: 1, quantity: 1 },
  { productId: 3, quantity: 2 }
];

const orderProgram = createOrder(1, orderItems);

const result = orderProgram.evaluate(initialState);
console.log("Результат создания заказа:", result.getValue());
```

## Выбор подходящей монады

Выбор подходящей монады зависит от типа эффекта, который нужно обработать:

1. **Maybe/Option** - для работы с потенциально отсутствующими значениями
2. **Either** - для обработки ошибок с информацией о них
3. **IO** - для работы с побочными эффектами
4. **State** - для работы с изменяемым состоянием
5. **Reader** - для работы с неизменяемым окружением
6. **Writer** - для накопления дополнительной информации
7. **List** - для недетерминированных вычислений
8. **Future/Promise** - для асинхронных вычислений

```typescript
// Пример выбора монады в зависимости от контекста
const chooseMonadExample = () => {
  // Для потенциально отсутствующих значений - Maybe
  const safeDivision = (a: number, b: number): MaybeMonad<number> => {
    return b === 0 ? MaybeMonad.nothing() : MaybeMonad.just(a / b);
  };
  
  // Для обработки ошибок - Either
  const parseInteger = (str: string): EitherMonad<string, number> => {
    const num = parseInt(str, 10);
    return isNaN(num) 
      ? EitherMonad.left(`"${str}" не является целым числом`)
      : EitherMonad.right(num);
  };
  
  // Для побочных эффектов - IO
  const getCurrentTimestamp = (): IOMonad<number> => {
    return IOMonad.from(() => Date.now());
  };
  
  // Для состояния - State
  const incrementCounter = (): StateMonad<{ count: number }, number> => {
    return StateMonad.modify<{ count: number }>(state => ({ count: state.count + 1 }))
      .flatMap(() => StateMonad.get<{ count: number }>())
      .map(state => state.count);
  };
  
  // Для окружения - Reader
  const getDatabaseConnection = (): ReaderMonad<{ dbUrl: string }, string> => {
    return ReaderMonad.ask<{ dbUrl: string }>().map(env => env.dbUrl);
  };
  
  // Для логирования - Writer
  const loggedCalculation = (a: number, b: number): WriterMonad<string, number> => {
    return WriterMonad.tell<string>(`Вычисление: ${a} + ${b}`)
      .flatMap(() => WriterMonad.of(a + b));
  };
  
  // Для недетерминированных вычислений - List
  const possibleResults = (): ListMonad<number> => {
    return ListMonad.fromArray([1, 2, 3, 4, 5]);
  };
  
  // Для асинхронных операций - Future
  const asyncOperation = (): FutureMonad<string> => {
    return FutureMonad.fromPromise(
      new Promise(resolve => setTimeout(() => resolve("Результат"), 1000))
    );
  };
};
```

## Связи с другими концепциями

- [[ts/functional-programming/functors-monads/Функторы_и_монады|Функторы и монады]] - Основная страница раздела
- [[ts/functional-programming/functors-monads/functors|Функторы]] - Базовая концепция функторов
- [[ts/functional-programming/functors-monads/monads|Монады]] - Подробное изучение монад
- [[ts/functional-programming/functors-monads/practical-examples|Практические примеры]] - Реальные примеры использования
- [[ts/error-handling/index|Обработка ошибок]] - Обработка ошибок в функциональном стиле
- [[ts/testing/index|Тестирование]] - Тестирование монад
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация монад

## Следующие шаги

- [[ts/functional-programming/functors-monads/practical-examples|Практические примеры]] - Реальные примеры использования
- [[ts/error-handling/index|Обработка ошибок]] - Расширенная обработка ошибок в функциональном стиле
- [[ts/testing/index|Тестирование]] - Тестирование монад
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация монад
- [[ts/functional-programming/Функциональное_программирование|Функциональное программирование]] - Обзор всех концепций функционального программирования
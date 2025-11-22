# Функциональное программирование в TypeScript

Функциональное программирование (ФП) в TypeScript позволяет создавать более предсказуемые, тестируемые и надежные приложения. Оно основывается на использовании чистых функций, неизменяемых данных и функций высшего порядка.

## Основные концепции функционального программирования

### 1. Чистые функции

```typescript
// Чистая функция: всегда возвращает одинаковый результат для одинаковых аргументов
// и не имеет побочных эффектов
function add(a: number, b: number): number {
  return a + b;
}

// Не чистая функция: зависит от внешнего состояния
let multiplier = 2;
function impureMultiply(a: number): number {
  return a * multiplier; // Зависит от внешней переменной
}

// Чистая альтернатива
function pureMultiply(a: number, multiplier: number): number {
  return a * multiplier;
}

// Еще примеры чистых функций
function capitalize(str: string): string {
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}

function isEven(num: number): boolean {
  return num % 2 === 0;
}

function doubleArray(numbers: number[]): number[] {
  return numbers.map(n => n * 2);
}
```

### 2. Неизменяемость (Immutability)

```typescript
// Использование readonly для создания неизменяемых структур
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly preferences: readonly string[];
}

// Функция для обновления пользователя без изменения оригинала
function updateUser(user: User, updates: Partial<User>): User {
  return {
    ...user,
    ...updates
  };
}

// Функция для добавления предпочтения без изменения оригинального массива
function addPreference(user: User, preference: string): User {
  return {
    ...user,
    preferences: [...user.preferences, preference]
  };
}

// Использование Object.freeze для глубокой неизменяемости
function deepFreeze<T>(obj: T): T {
  Object.getOwnPropertyNames(obj).forEach(prop => {
    const value = (obj as any)[prop];
    if (value && typeof value === "object") {
      deepFreeze(value);
    }
  });
  return Object.freeze(obj) as T;
}
```

## Функции высшего порядка

### 1. Определение и использование

```typescript
// Функция высшего порядка принимает функцию как аргумент или возвращает функцию
type Predicate<T> = (item: T) => boolean;
type Transformer<T, U> = (item: T) => U;

// Функция высшего порядка: принимает предикат
function findFirst<T>(array: T[], predicate: Predicate<T>): T | undefined {
  for (const item of array) {
    if (predicate(item)) {
      return item;
    }
  }
  return undefined;
}

// Использование
const numbers = [1, 2, 3, 4, 5];
const firstEven = findFirst(numbers, n => n % 2 === 0); // 2

// Функция высшего порядка: возвращает функцию
function createMultiplier(factor: number): (value: number) => number {
  return function(value: number): number {
    return value * factor;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(4)); // 12
```

### 2. Композиция функций

```typescript
// Утилита для композиции функций справа налево
function compose<T, U, V>(f: (x: U) => V, g: (x: T) => U): (x: T) => V {
  return function(x: T): V {
    return f(g(x));
  };
}

// Универсальная композиция нескольких функций
function pipe<T, R>(...fns: Array<Function>): (value: T) => R {
  return function(value: T): R {
    return fns.reduce((acc, fn) => fn(acc), value as any) as R;
  };
}

// Примеры использования
const add1 = (x: number) => x + 1;
const multiply2 = (x: number) => x * 2;
const toString = (x: number) => x.toString();

// Композиция: сначала умножаем на 2, потом прибавляем 1
const add1AfterMultiply2 = compose(add1, multiply2);
console.log(add1AfterMultiply2(5)); // 11

// Цепочка: 5 -> multiply2 -> add1 -> toString
const result = pipe(multiply2, add1, toString)(5);
console.log(result); // "11"
```

## Продвинутые функциональные паттерны

### 1. Каррирование

```typescript
// Каррированная функция: принимает аргументы по одному
function curry<A, B, C>(fn: (a: A, b: B) => C): (a: A) => (b: B) => C {
  return function(a: A): (b: B) => C {
    return function(b: B): C {
      return fn(a, b);
    };
  };
}

// Пример использования
function add(a: number, b: number): number {
  return a + b;
}

const curriedAdd = curry(add);
const add5 = curriedAdd(5); // (b: number) => number
console.log(add5(3)); // 8

// Каррирование для более сложных функций
function formatUserMessage(
  greeting: string, 
  punctuation: string, 
  name: string
): string {
  return `${greeting}, ${name}${punctuation}`;
}

const curriedFormat = curry(formatUserMessage);
const greetInEnglish = curriedFormat("Hello");
const greetWithExclamation = greetInEnglish("!");
const greetJohn = greetWithExclamation;
console.log(greetJohn("John")); // "Hello, John!"
```

### 2. Частичное применение

```typescript
// Утилита для частичного применения функции
function partial<T extends Function, U extends any[]>(
  fn: T, 
  ...presetArgs: U
): (...args: any[]) => any {
  return function(...laterArgs: any[]) {
    return fn(...presetArgs, ...laterArgs);
  };
}

// Пример использования
function multiply(a: number, b: number, c: number): number {
  return a * b * c;
}

const multiplyBySix = partial(multiply, 2, 3);
console.log(multiplyBySix(4)); // 24 (2 * 3 * 4)

// Частичное применение с методами объекта
const logger = {
  log: function(level: string, message: string) {
    console.log(`[${level}] ${message}`);
  }
};

const errorLogger = partial(logger.log, "ERROR");
errorLogger("Something went wrong"); // "[ERROR] Something went wrong"
```

### 3. Мемоизация

```typescript
// Простая мемоизация для функций с одним аргументом
function memoize<T, U>(fn: (arg: T) => U): (arg: T) => U {
  const cache = new Map<T, U>();
  
  return function(arg: T): U {
    if (cache.has(arg)) {
      return cache.get(arg)!;
    }
    
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}

// Мемоизация для функций с несколькими аргументами
function memoizeMulti<T extends any[], R>(fn: (...args: T) => R): (...args: T) => R {
  const cache = new Map<string, R>();
  
  return function(...args: T): R {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Пример использования
const expensiveOperation = memoize((n: number) => {
  console.log(`Вычисляем для ${n}...`);
  return n * n;
});

console.log(expensiveOperation(5)); // "Вычисляем для 5...", 25
console.log(expensiveOperation(5)); // 25 (из кэша)
```

## Практические применения

### 1. Работа с коллекциями

```typescript
// Утилиты для функциональной работы с массивами
function map<T, U>(array: T[], fn: (item: T) => U): U[] {
  return array.reduce((acc, item) => [...acc, fn(item)], [] as U[]);
}

function filter<T>(array: T[], predicate: Predicate<T>): T[] {
  return array.reduce((acc, item) => predicate(item) ? [...acc, item] : acc, [] as T[]);
}

function reduce<T, U>(array: T[], fn: (acc: U, item: T) => U, initial: U): U {
  return array.reduce(fn, initial);
}

function flatMap<T, U>(array: T[], fn: (item: T) => U[]): U[] {
  return array.reduce((acc, item) => [...acc, ...fn(item)], [] as U[]);
}

// Пример использования
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

const products: Product[] = [
  { id: 1, name: "Laptop", price: 1000, category: "Electronics" },
  { id: 2, name: "Book", price: 20, category: "Education" },
  { id: 3, name: "Phone", price: 800, category: "Electronics" },
  { id: 4, name: "Desk", price: 300, category: "Furniture" }
];

// Найти все электронные товары с ценой > 500, вернуть их имена в верхнем регистре
const expensiveElectronics = pipe(
  (items: Product[]) => filter(items, p => p.category === "Electronics"),
  (items: Product[]) => filter(items, p => p.price > 500),
  (items: Product[]) => map(items, p => p.name.toUpperCase())
)(products);

console.log(expensiveElectronics); // ["LAPTOP", "PHONE"]
```

### 2. Обработка асинхронных данных

```typescript
// Функции для функциональной обработки промисов
async function asyncMap<T, U>(
  items: T[], 
  asyncFn: (item: T) => Promise<U>
): Promise<U[]> {
  return Promise.all(items.map(asyncFn));
}

async function asyncFilter<T>(
  items: T[], 
  asyncPredicate: (item: T) => Promise<boolean>
): Promise<T[]> {
  const results = await Promise.all(items.map(async item => ({
    item,
    result: await asyncPredicate(item)
  })));
  
  return results.filter(r => r.result).map(r => r.item);
}

// Пример использования
interface UserData {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(id: number): Promise<UserData> {
  // Симуляция API вызова
  await new Promise(resolve => setTimeout(resolve, 100));
  return { id, name: `User ${id}`, email: `user${id}@example.com` };
}

async function isPremiumUser(user: UserData): Promise<boolean> {
  // Симуляция проверки премиум статуса
  await new Promise(resolve => setTimeout(resolve, 50));
  return user.id % 2 === 0; // Четные ID - премиум пользователи
}

async function getPremiumUserNames(): Promise<string[]> {
  const userIds = [1, 2, 3, 4, 5, 6];
  
  return pipe(
    asyncMap,
    (users: UserData[]) => asyncFilter(users, isPremiumUser),
    (premiumUsers: UserData[]) => premiumUsers.map(u => u.name)
  )(userIds, fetchUser);
}

// Использование
getPremiumUserNames().then(names => console.log(names)); // ["User 2", "User 4", "User 6"]
```

### 3. Валидация данных

```typescript
// Тип для результата валидации
type ValidationResult<T> = {
  success: true;
  value: T;
} | {
  success: false;
  errors: string[];
};

// Функции валидации
type Validator<T> = (value: T) => ValidationResult<T>;

// Комбинаторы валидации
function combineValidations<T>(
  ...validators: Validator<T>[]
): Validator<T> {
  return function(value: T): ValidationResult<T> {
    const errors: string[] = [];
    
    for (const validator of validators) {
      const result = validator(value);
      if (!result.success) {
        errors.push(...result.errors);
      }
    }
    
    if (errors.length > 0) {
      return { success: false, errors };
    }
    
    return { success: true, value } as ValidationResult<T>;
  };
}

// Простые валидаторы
const required: Validator<string> = (value) => {
  if (value.trim() === "") {
    return { success: false, errors: ["Обязательное поле"] };
  }
  return { success: true, value };
};

const minLength = (min: number): Validator<string> => (value) => {
  if (value.length < min) {
    return { success: false, errors: [`Минимум ${min} символов`] };
  }
  return { success: true, value };
};

const emailFormat: Validator<string> = (value) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(value)) {
    return { success: false, errors: ["Неверный формат email"] };
  }
  return { success: true, value };
};

// Комбинирование валидаторов
const emailValidator = combineValidations(
  required,
  emailFormat
);

const passwordValidator = combineValidations(
  required,
  minLength(8)
);

// Валидация формы
interface LoginForm {
  email: string;
  password: string;
}

function validateLoginForm(form: LoginForm): ValidationResult<LoginForm> {
  const emailResult = emailValidator(form.email);
  const passwordResult = passwordValidator(form.password);
  
  const errors: string[] = [];
  
  if (!emailResult.success) {
    errors.push(`Email: ${emailResult.errors.join(', ')}`);
  }
  
  if (!passwordResult.success) {
    errors.push(`Password: ${passwordResult.errors.join(', ')}`);
  }
  
  if (errors.length > 0) {
    return { success: false, errors };
  }
  
  return { success: true, value: form };
}

// Использование
const form: LoginForm = { email: "test@example.com", password: "12345678" };
const validation = validateLoginForm(form);

if (validation.success) {
  console.log("Форма валидна:", validation.value);
} else {
  console.log("Ошибки валидации:", validation.errors);
}
```

## Продвинутые паттерны

### 1. Моноиды и функторы

```typescript
// Интерфейс для моноида
interface Monoid<T> {
  empty: T;
  concat: (a: T, b: T) => T;
}

// Пример моноида для строк
const stringMonoid: Monoid<string> = {
  empty: "",
  concat: (a, b) => a + b
};

// Пример моноида для чисел (сложение)
const sumMonoid: Monoid<number> = {
  empty: 0,
  concat: (a, b) => a + b
};

// Пример моноида для чисел (умножение)
const productMonoid: Monoid<number> = {
  empty: 1,
  concat: (a, b) => a * b
};

// Функция для свертки с использованием моноида
function fold<T>(monoid: Monoid<T>, items: T[]): T {
  return items.reduce(monoid.concat, monoid.empty);
}

// Использование
const strings = ["Hello", " ", "World", "!"];
const concatenated = fold(stringMonoid, strings); // "Hello World!"

const numbers = [1, 2, 3, 4, 5];
const sum = fold(sumMonoid, numbers); // 15
const product = fold(productMonoid, numbers); // 120

// Интерфейс для функтора (упрощенный)
interface Functor<T> {
  map<U>(fn: (value: T) => U): Functor<U>;
}

// Пример функтора - Maybe (для безопасной работы с null/undefined)
type Maybe<T> = { type: 'just', value: T } | { type: 'nothing' };

function mapMaybe<T, U>(maybe: Maybe<T>, fn: (value: T) => U): Maybe<U> {
  if (maybe.type === 'nothing') {
    return { type: 'nothing' };
  }
  return { type: 'just', value: fn(maybe.value) };
}

function just<T>(value: T): Maybe<T> {
  return { type: 'just', value };
}

function nothing<T>(): Maybe<T> {
  return { type: 'nothing' };
}

// Использование
const safeDivide = (a: number, b: number): Maybe<number> => {
  if (b === 0) return nothing();
  return just(a / b);
};

const result = mapMaybe(
  safeDivide(10, 2),
  x => x * 3
); // Just(15)

const errorResult = mapMaybe(
  safeDivide(10, 0),
  x => x * 3
); // Nothing
```

### 2. Трансдьюсеры

```typescript
// Тип для трансдьюсера
type Transducer<A, B> = <C>(reducer: (acc: C, item: B) => C) => (acc: C, item: A) => C;

// Функции для создания трансдьюсеров
function mapTransducer<T, U>(fn: (item: T) => U): Transducer<T, U> {
  return function(reducer) {
    return function(acc, item) {
      return reducer(acc, fn(item));
    };
  };
}

function filterTransducer<T>(predicate: Predicate<T>): Transducer<T, T> {
  return function(reducer) {
    return function(acc, item) {
      if (predicate(item)) {
        return reducer(acc, item);
      }
      return acc;
    };
  };
}

function takeTransducer<T>(count: number): Transducer<T, T> {
  let taken = 0;
  return function(reducer) {
    return function(acc, item) {
      if (taken < count) {
        taken++;
        return reducer(acc, item);
      }
      return acc;
    };
  };
}

// Применение трансдьюсеров
function transduce<T, U, R>(
  transducer: Transducer<T, U>,
  reducer: (acc: R, item: U) => R,
  initial: R,
  items: T[]
): R {
  const transformedReducer = transducer(reducer);
  return items.reduce(transformedReducer, initial);
}

// Композиция трансдьюсеров
const composeTransducers = <A, B, C>(
  t1: Transducer<A, B>,
  t2: Transducer<B, C>
): Transducer<A, C> => {
  return function(reducer) {
    return t1(t2(reducer));
  };
};

// Пример использования
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Композиция трансдьюсеров: отфильтровать четные, умножить на 2, взять первые 3
const composed = composeTransducers(
  composeTransducers(
    filterTransducer((n: number) => n % 2 === 0),
    mapTransducer((n: number) => n * 2)
  ),
  takeTransducer(3)
);

const result = transduce(
  composed,
  (acc: number[], item: number) => [...acc, item],
  [] as number[],
  numbers
);

console.log(result); // [4, 8, 12] - первые 3 четных числа, умноженных на 2
```

## Практические примеры из реальной разработки

### 1. Система трансформации данных

```typescript
// Интерфейс для трансформации данных
interface DataTransformer<T, U> {
  transform: (data: T) => U;
  validate: (data: U) => boolean;
}

// Утилита для цепочки трансформаций
class TransformationChain<T> {
  constructor(private data: T) {}
  
  map<U>(transformer: (value: T) => U): TransformationChain<U> {
    return new TransformationChain(transformer(this.data));
  }
  
  filter(predicate: Predicate<T>): TransformationChain<T> {
    if (predicate(this.data)) {
      return this;
    }
    // Возвращаем цепочку с undefined или подходящим значением по умолчанию
    return new TransformationChain(undefined as any);
  }
  
  reduce<U>(reducer: (acc: U, item: T) => U, initial: U): U {
    return reducer(initial, this.data);
  }
  
  get(): T {
    return this.data;
  }
}

// Пример использования
interface RawUser {
  id: string;
  name: string;
  email: string;
  age: string;
  isActive: string;
}

interface ProcessedUser {
  id: number;
  name: string;
  email: string;
  age: number;
  isActive: boolean;
}

const rawUsers: RawUser[] = [
  { id: "1", name: "John Doe", email: "john@example.com", age: "30", isActive: "true" },
  { id: "2", name: "Jane Smith", email: "jane@example.com", age: "25", isActive: "false" }
];

const processedUsers = rawUsers.map(user => 
  new TransformationChain(user)
    .map(u => ({
      id: parseInt(u.id),
      name: u.name.trim(),
      email: u.email.toLowerCase(),
      age: parseInt(u.age),
      isActive: u.isActive.toLowerCase() === 'true'
    }))
    .get() as ProcessedUser
);
```

### 2. Функциональная система событий

```typescript
// Функциональная система обработки событий
type EventHandler<T> = (event: T) => void;

interface EventProcessor<T> {
  filter: (predicate: Predicate<T>) => EventProcessor<T>;
  map: <U>(transformer: Transformer<T, U>) => EventProcessor<U>;
  subscribe: (handler: EventHandler<T>) => void;
}

class FunctionalEventEmitter<T> implements EventProcessor<T> {
  private handlers: EventHandler<T>[] = [];
  
  constructor(private events: T[] = []) {}
  
  filter(predicate: Predicate<T>): FunctionalEventEmitter<T> {
    const filteredEvents = this.events.filter(predicate);
    return new FunctionalEventEmitter(filteredEvents);
  }
  
  map<U>(transformer: Transformer<T, U>): FunctionalEventEmitter<U> {
    const transformedEvents = this.events.map(transformer);
    return new FunctionalEventEmitter(transformedEvents);
  }
  
  subscribe(handler: EventHandler<T>): void {
    this.handlers.push(handler);
    this.events.forEach(event => handler(event));
  }
  
  emit(event: T): void {
    this.handlers.forEach(handler => handler(event));
  }
}

// Использование
interface UserEvent {
  type: 'login' | 'logout' | 'profile_update';
  userId: number;
  timestamp: Date;
}

const events: UserEvent[] = [
  { type: 'login', userId: 1, timestamp: new Date() },
  { type: 'login', userId: 2, timestamp: new Date() },
  { type: 'logout', userId: 1, timestamp: new Date() }
];

const emitter = new FunctionalEventEmitter(events);

// Подписка на события входа
emitter
  .filter(event => event.type === 'login')
  .subscribe(event => console.log(`User ${event.userId} logged in at ${event.timestamp}`));
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки

```typescript
// Ошибка 1: Мутирование данных в функциональном стиле
const users = [{ id: 1, name: "John" }];
// ПЛОХО: мутируем оригинальный массив
// users.push({ id: 2, name: "Jane" });

// ХОРОШО: создаем новый массив
const newUsers = [...users, { id: 2, name: "Jane" }];

// Ошибка 2: Неэффективное использование функций высшего порядка
// ПЛОХО: несколько проходов по массиву
const result1 = users.filter(u => u.name.length > 3);
const result2 = result1.map(u => u.name.toUpperCase());
const result3 = result2.filter(name => name.startsWith('J'));

// ХОРОШО: один проход с композицией
const result = pipe(
  (items: typeof users) => items.filter(u => u.name.length > 3),
  (items: typeof users) => items.map(u => u.name.toUpperCase()),
  (names: string[]) => names.filter(name => name.startsWith('J'))
)(users);

// Ошибка 3: Неправильное использование мемоизации
// ПЛОХО: мемоизация для функций с побочными эффектами
const badMemoized = memoize((x: number) => {
  console.log("Вычисляем..."); // Это побочный эффект!
  return x * 2;
});
```

### 2. Лучшие практики

```typescript
// 1. Используйте неизменяемые структуры данных
interface ReadonlyUser {
  readonly id: number;
  readonly name: string;
  readonly preferences: readonly string[];
}

// 2. Создавайте чистые функции
const calculateTotal = (prices: readonly number[]): number => {
  return prices.reduce((sum, price) => sum + price, 0);
};

// 3. Используйте функции высшего порядка для повторяющихся паттернов
const withLogging = <T extends Function>(fn: T): T => {
  return ((...args: any[]) => {
    console.log(`Вызов функции с аргументами:`, args);
    const result = fn(...args);
    console.log(`Результат:`, result);
    return result;
  }) as any;
};

// 4. Комбинируйте функции для создания более сложной логики
const processUser = pipe(
  (user: any) => ({ ...user, name: user.name.trim() }),
  (user: any) => ({ ...user, email: user.email.toLowerCase() }),
  (user: any) => ({ ...user, isValid: user.email.includes('@') })
);
```

## Лучшие практики

1. **Предпочитайте чистые функции** - они более предсказуемы и легче тестируются
2. **Используйте неизменяемые структуры данных** - для предотвращения неожиданных изменений
3. **Применяйте композицию функций** - для создания сложной логики из простых компонентов
4. **Используйте каррирование и частичное применение** - для создания специализированных функций
5. **Избегайте побочных эффектов** - особенно в чистых функциях
6. **Документируйте функциональные паттерны** - особенно когда они неочевидны

## Связи с другими концепциями

- [[ts/type-system/type-inference|Вывод типов]] - как TypeScript помогает с типизацией функций
- [[ts/generics/TS Generics|Обобщения]] - основа для универсальных функциональных утилит
- [[ts/advanced-types/conditional-types|Условные типы]] - для сложных функциональных преобразований
- [[Утилиты типов TypeScript|Утилиты типов]] - для работы с функциональными паттернами
- [[js/functional-programming]] - основы функционального программирования в JavaScript
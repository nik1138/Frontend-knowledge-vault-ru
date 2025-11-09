# Functional Programming in TypeScript

Функциональное программирование в TypeScript использует функции как основные строительные блоки программ, применяя концепции неизменяемости, чистых функций и композиции для создания надежного и поддерживаемого кода.

## Основы функционального программирования

### Чистые функции
Чистые функции не имеют сайд-эффектов и всегда возвращают одинаковый результат для одинаковых аргументов:

```typescript
// Чистая функция
function add(a: number, b: number): number {
  return a + b;
}

// Чистая функция с объектами (неизменяемая операция)
function updateUser(user: { name: string; age: number }, newName: string) {
  return { ...user, name: newName };
}

// Не чистая функция (сайд-эффект)
function logAdd(a: number, b: number): number {
  console.log("Adding:", a, b); // Сайд-эффект
  return a + b;
}
```

### Неизменяемость (Immutability)
```typescript
// Использование readonly для создания неизменяемых структур
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
}

// Манипуляции с массивами без изменения оригинала
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2);  // [2, 4, 6, 8], numbers не изменен
const evens = numbers.filter(n => n % 2 === 0);  // [2, 4], numbers не изменен

// Использование as const для литеральных типов
const weekdays = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"] as const;
// Тип: readonly ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"]
```

## Высшие порядки функций

### Функции, возвращающие функции
```typescript
// Функция, возвращающая другую функцию
function makeAdder(x: number): (y: number) => number {
  return function(y: number): number {
    return x + y;
  };
}

// Или с стрелочной функцией
const makeMultiplier = (factor: number) => (value: number): number => value * factor;

const add5 = makeAdder(5);
const multiplyBy10 = makeMultiplier(10);

console.log(add5(3)); // 8
console.log(multiplyBy10(4)); // 40
```

### Функции, принимающие другие функции
```typescript
// Функция, принимающая другую функцию
function applyOperation(x: number, y: number, operation: (a: number, b: number) => number): number {
  return operation(x, y);
}

// Использование
const result1 = applyOperation(5, 3, add); // 8
const result2 = applyOperation(5, 3, (a, b) => a * b); // 15

// Более сложный пример
function processArray<T, U>(
  array: T[], 
  transformer: (item: T) => U, 
  filter: (item: T) => boolean
): U[] {
  return array.filter(filter).map(transformer);
}

// Использование
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const doubledEvens = processArray(
  numbers, 
  n => n * 2,  // удвоить
  n => n % 2 === 0  // только четные
); // [4, 8, 12, 16, 20]
```

## Встроенные функции массивов

### Основные операции
```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

const products: Product[] = [
  { id: 1, name: 'Laptop', price: 1000, category: 'Electronics' },
  { id: 2, name: 'Book', price: 20, category: 'Education' },
  { id: 3, name: 'Phone', price: 800, category: 'Electronics' },
  { id: 4, name: 'Desk', price: 300, category: 'Furniture' }
];

// map - преобразование
const productNames: string[] = products.map(p => p.name);

// filter - фильтрация
const expensiveProducts: Product[] = products.filter(p => p.price > 500);

// reduce - агрегация
const totalPrice: number = products.reduce((sum, p) => sum + p.price, 0);

// find - поиск
const laptop: Product | undefined = products.find(p => p.name === 'Laptop');

// sort - сортировка (не мутирует оригинальный массив при правильном использовании)
const sortedByPrice = [...products].sort((a, b) => a.price - b.price);
```

### Цепочки операций
```typescript
// Цепочка операций - функциональный подход
const avgElectronicsPrice: number = products
  .filter(p => p.category === 'Electronics')
  .map(p => p.price)
  .reduce((sum, price, _, arr) => sum + price / arr.length, 0);

// Или с вычислением промежуточных значений
const electronicsStats = products
  .filter(p => p.category === 'Electronics')
  .reduce((acc, product) => ({
    count: acc.count + 1,
    totalPrice: acc.totalPrice + product.price,
    avgPrice: (acc.totalPrice + product.price) / (acc.count + 1)
  }), { count: 0, totalPrice: 0, avgPrice: 0 });
```

## Каррирование (Currying)

### Реализация каррирования
```typescript
// Утилита для каррирования
const curry = <T, U, V>(fn: (a: T, b: U) => V) => 
  (a: T) => (b: U): V => fn(a, b);

// Пример использования
const multiply = (a: number, b: number) => a * b;
const curriedMultiply = curry(multiply);
const multiplyBy2 = curriedMultiply(2);

console.log(multiplyBy2(5)); // 10

// Каррирование функции с тремя параметрами
const curry3 = <T, U, V, R>(fn: (a: T, b: U, c: V) => R) => 
  (a: T) => (b: U) => (c: V): R => fn(a, b, c);

const format = (prefix: string, suffix: string, value: string) => 
  `${prefix}${value}${suffix}`;

const curriedFormat = curry3(format);
const xmlTag = curriedFormat("<")(">");
const wrapped = xmlTag("div"); // "<div>"
```

## Композиция функций

### Простая композиция
```typescript
// Утилита для композиции двух функций
const compose = <T, U, V>(f: (x: U) => V, g: (x: T) => U) => 
  (x: T): V => f(g(x));

// Пример: преобразование строки -> длина -> удвоенная длина
const stringToDoubledLength = compose(
  (n: number) => n * 2,
  (s: string) => s.length
);

console.log(stringToDoubledLength("hello")); // 10

// Композиция нескольких функций
const pipe = <T>(value: T, ...fns: Array<(arg: any) => any>) =>
  fns.reduce((acc, fn) => fn(acc), value);

// Использование pipe
const result = pipe(
  "hello world",
  (s: string) => s.toUpperCase(),
  (s: string) => s.split(' '),
  (arr: string[]) => arr.length
); // 2
```

### Продвинутая композиция
```typescript
// Типизированная композиция для большего количества функций
type Compose = {
  <A, B>(fn: (a: A) => B): (a: A) => B;
  <A, B, C>(fn2: (b: B) => C, fn1: (a: A) => B): (a: A) => C;
  <A, B, C, D>(fn3: (c: C) => D, fn2: (b: B) => C, fn1: (a: A) => B): (a: A) => D;
  // и так далее...
};

const compose: Compose = (...fns: any[]) => (initial: any) => 
  fns.reverse().reduce((acc, fn) => fn(acc), initial);

// Создание цепочки обработки данных
const processUserData = compose(
  (users: any[]) => users.map(u => ({ id: u.id, name: u.name.toUpperCase() })),
  (users: any[]) => users.filter(u => u.active),
  (data: any) => data.users
);
```

## Функциональные утилиты

### Maybe/Option тип
```typescript
type Maybe<T> = T | null | undefined;

// Функции для работы с Maybe
const mapMaybe = <T, U>(value: Maybe<T>, fn: (v: T) => U): Maybe<U> => 
  value != null ? fn(value) : null;

const filterMaybe = <T>(value: Maybe<T>, predicate: (v: T) => boolean): Maybe<T> => 
  value != null && predicate(value) ? value : null;

// Пример использования
const user: Maybe<{ profile?: { age?: number } }> = { profile: { age: 25 } };

const userAge = mapMaybe(user, u => u.profile)
                ?.age; // TypeScript безопасно разбирает цепочку
```

### Either тип для обработки ошибок
```typescript
type Either<L, R> = { type: 'Left', value: L } | { type: 'Right', value: R };

const left = <L, R>(value: L): Either<L, R> => ({ type: 'Left', value });
const right = <L, R>(value: R): Either<L, R> => ({ type: 'Right', value });

const mapEither = <L, R, S>(
  either: Either<L, R>,
  fn: (value: R) => S
): Either<L, S> => 
  either.type === 'Right' 
    ? right(fn(either.value)) 
    : left(either.value);

// Пример использования
const divide = (a: number, b: number): Either<string, number> => 
  b === 0 
    ? left("Division by zero") 
    : right(a / b);

const result = mapEither(divide(10, 2), x => x * 2); // Right(10)
const error = mapEither(divide(10, 0), x => x * 2); // Left("Division by zero")

// Разбор Either
const handleResult = <L, R>(either: Either<L, R>, onSuccess: (r: R) => void, onError: (l: L) => void) => {
  if (either.type === 'Right') {
    onSuccess(either.value);
  } else {
    onError(either.value);
  }
};
```

## Продвинутые функциональные паттерны

### Функторы
```typescript
// Простой функтор для массива
const fmap = <T, U>(fn: (t: T) => U, arr: T[]): U[] => arr.map(fn);

// Функтор для Maybe
const maybeFmap = <T, U>(fn: (t: T) => U, maybe: Maybe<T>): Maybe<U> => 
  maybe != null ? fn(maybe) : null;
```

### Мемоизация
```typescript
// Простая мемоизация
const memoize = <T extends (...args: any[]) => any>(fn: T): T => {
  const cache = new Map<string, any>();
  
  return function(...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  } as T;
};

// Пример использования
const expensiveOperation = (n: number) => {
  // Сложная операция
  return n * n * n;
};

const memoizedExpensive = memoize(expensiveOperation);
```

## Практические примеры

### Обработка данных
```typescript
interface Order {
  id: number;
  userId: number;
  amount: number;
  status: 'pending' | 'completed' | 'cancelled';
  createdAt: Date;
}

// Функциональная обработка заказов
const processOrders = (orders: Order[]) => 
  orders
    .filter(order => order.status === 'completed')
    .filter(order => order.amount > 100)
    .map(order => ({ ...order, processedAt: new Date() }))
    .sort((a, b) => b.createdAt.getTime() - a.createdAt.getTime());

// Типизированная версия
const createOrderProcessor = <T extends Order>(customFilters: ((order: T) => boolean)[] = []) => 
  (orders: T[]): T[] => 
    orders
      .filter(order => customFilters.every(filter => filter(order)))
      .sort((a, b) => b.amount - a.amount);
```

### Валидация
```typescript
interface ValidationResult<T> {
  success: boolean;
  errors: string[];
  data?: T;
}

// Функции валидации
const required = (value: any): boolean => value != null && value !== '';
const minLength = (min: number) => (value: string): boolean => value.length >= min;
const emailFormat = (value: string): boolean => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);

// Композиция валидаций
const validateField = <T>(
  value: T, 
  ...validators: ((v: T) => boolean)[]
): ValidationResult<T> => {
  const errors = validators
    .map(validate => ({ validate, result: validate(value) }))
    .filter(({ result }) => !result)
    .map(() => "Validation failed");
  
  return {
    success: errors.length === 0,
    errors,
    data: errors.length === 0 ? value : undefined
  };
};
```

### Функциональное построение объектов
```typescript
// Builder pattern в функциональном стиле
const createUser = (name: string) => ({ name });
const withAge = (age: number) => (user: { name: string }) => ({ ...user, age });
const withEmail = (email: string) => (user: { name: string; age?: number }) => ({ ...user, email });

// Использование
const finalUser = [withAge(30), withEmail("user@example.com")]
  .reduce((acc, fn) => fn(acc), createUser("John"));
// Результат: { name: "John", age: 30, email: "user@example.com" }
```

## Лучшие практики

### Чистота функций
```typescript
// Хорошо: чистая функция
const calculateTax = (amount: number, rate: number): number => amount * rate;

// Плохо: функция с сайд-эффектами
let totalTax = 0;
const calculateTaxWithSideEffect = (amount: number, rate: number): number => {
  const tax = amount * rate;
  totalTax += tax; // сайд-эффект!
  return tax;
};
```

### Использование иммутабельных структур
```typescript
// Хорошо: неизменяемые структуры
const updateState = <T extends Record<string, any>>(
  state: T, 
  updates: Partial<T>
): T => ({ ...state, ...updates });

// Плохо: мутация
const badUpdateState = <T extends Record<string, any>>(
  state: T, 
  updates: Partial<T>
): T => {
  Object.assign(state, updates); // мутация!
  return state;
};
```

### Композиция вместо наследования
```typescript
// Функциональная композиция вместо наследования
const canFly = (bird: any) => ({ ...bird, fly: () => console.log("Flying!") });
const canSwim = (bird: any) => ({ ...bird, swim: () => console.log("Swimming!") });

const duck = canSwim(canFly({ name: "Donald" }));
// duck может и летать, и плавать
```

## Функциональные библиотеки

### Создание собственных утилит
```typescript
// Ramda-подобные утилиты
const prop = <K extends string, T extends Record<K, any>>(key: K) => (obj: T): T[K] => obj[key];

const propEq = <K extends string, V>(key: K, value: V) => <T extends Record<K, V>>(obj: T): boolean =>
  obj[key] === value;

const map = <T, U>(fn: (item: T) => U) => (array: T[]): U[] => array.map(fn);

// Использование
const users = [{ name: 'Alice', age: 30 }, { name: 'Bob', age: 25 }];
const getNames = map(prop('name')); // (users: User[]) => string[]
const adults = users.filter(propEq('age', 30)); // [{ name: 'Alice', age: 30 }]

const names = getNames(users); // ['Alice', 'Bob']
```

## Связь с другими концепциями

### С обобщениями (Generics)
```typescript
// Функциональные утилиты часто используют обобщения
const identity = <T>(x: T): T => x;

const constant = <T>(value: T) => (): T => value;

const composeWithGenerics = <A, B, C>(f: (b: B) => C, g: (a: A) => B) => (x: A): C => f(g(x));
```

### С типами утилит (Utility Types)
```typescript
// Функциональное преобразование типов
type MapType<T, F> = { [K in keyof T]: F extends (x: T[K]) => infer R ? R : never };

const transformObject = <T extends Record<string, any>, F extends (x: any) => any>(
  obj: T,
  transformer: F
): MapType<T, F> => {
  const result = {} as MapType<T, F>;
  for (const key in obj) {
    result[key] = transformer(obj[key]);
  }
  return result;
};
```

## Лучшие практики

1. **Предпочитайте чистые функции** - они проще в тестировании и отладке
2. **Используйте неизменяемые структуры данных** - избегайте мутаций
3. **Применяйте композицию функций** - для создания сложной логики из простых частей
4. **Используйте функции высшего порядка** - для абстракции и повторного использования
5. **Используйте типизацию** - для документирования функций и обеспечения безопасности
6. **Избегайте лишней сложности** - функциональный код не должен быть сложнее необходимого

## Связь с другими концепциями
- [[types]] - типизация функций и возвращаемых значений
- [[TS Generics]] - обобщенные функциональные утилиты
- [[../advanced-types/conditional-types]] - условные типы в функциональных утилитах
- [[TypeScript Modules]] - организация функциональных утилит в модули
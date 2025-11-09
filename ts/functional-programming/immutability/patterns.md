# Паттерны неизменяемости

Паттерны неизменяемости - это повторяющиеся подходы и техники для работы с неизменяемыми структурами данных. Эти паттерны помогают создавать предсказуемый, тестируемый и поддерживаемый код.

## Основные паттерны неизменяемости

### 1. Persistent Data Structures (Персистентные структуры данных)

Персистентные структуры данных сохраняют предыдущие версии при внесении изменений. Вместо модификации существующих данных создаются новые версии с общими частями.

```typescript
// Пример персистентного стека
class PersistentStack<T> {
  constructor(
    private readonly _head: T | null = null,
    private readonly _tail: PersistentStack<T> | null = null
  ) {}
  
  push(value: T): PersistentStack<T> {
    return new PersistentStack(value, this);
  }
  
  pop(): PersistentStack<T> | null {
    return this._tail;
  }
  
  peek(): T | null {
    return this._head;
  }
  
  isEmpty(): boolean {
    return this._head === null;
  }
  
  toArray(): T[] {
    const result: T[] = [];
    let current: PersistentStack<T> | null = this;
    
    while (current && !current.isEmpty()) {
      result.push(current.peek()!);
      current = current.pop();
    }
    
    return result.reverse();
  }
  
  static empty<T>(): PersistentStack<T> {
    return new PersistentStack<T>();
  }
}

// Использование
const stack1 = PersistentStack.empty<number>()
  .push(1)
  .push(2)
  .push(3);

const stack2 = stack1.pop()!; // Удаляем 3
const stack3 = stack2.push(4); // Добавляем 4

console.log(stack1.toArray()); // [1, 2, 3]
console.log(stack2.toArray()); // [1, 2]
console.log(stack3.toArray()); // [1, 2, 4]
```

### 2. Structural Sharing (Структурное разделение)

Структурное разделение позволяет нескольким версиям структуры данных совместно использовать общие части, что снижает потребление памяти.

```typescript
// Пример структурного разделения с массивами
class ImmutableList<T> {
  private constructor(private readonly _items: readonly T[]) {}
  
  get length(): number {
    return this._items.length;
  }
  
  get(index: number): T | undefined {
    return this._items[index];
  }
  
  push(item: T): ImmutableList<T> {
    // Совместное использование оригинального массива
    return new ImmutableList([...this._items, item]);
  }
  
  pop(): ImmutableList<T> {
    if (this._items.length === 0) {
      return this;
    }
    
    // Совместное использование части оригинального массива
    return new ImmutableList(this._items.slice(0, -1));
  }
  
  set(index: number, value: T): ImmutableList<T> {
    if (index < 0 || index >= this._items.length) {
      return this;
    }
    
    const newItems = [...this._items];
    newItems[index] = value;
    
    return new ImmutableList(newItems);
  }
  
  slice(start?: number, end?: number): ImmutableList<T> {
    return new ImmutableList(this._items.slice(start, end));
  }
  
  concat(other: ImmutableList<T>): ImmutableList<T> {
    return new ImmutableList([...this._items, ...other._items]);
  }
  
  static empty<T>(): ImmutableList<T> {
    return new ImmutableList([]);
  }
  
  static of<T>(...items: T[]): ImmutableList<T> {
    return new ImmutableList(items);
  }
}

// Использование с демонстрацией структурного разделения
const list1 = ImmutableList.of(1, 2, 3, 4, 5);
const list2 = list1.push(6);
const list3 = list1.push(7);

// list1 и list2 совместно используют элементы [1, 2, 3, 4, 5]
// list1 и list3 совместно используют элементы [1, 2, 3, 4, 5]
```

### 3. Copy-on-Write (Копирование при записи)

Copy-on-Write откладывает копирование данных до момента их изменения, что повышает эффективность.

```typescript
// Пример Copy-on-Write для массивов
class CopyOnWriteArray<T> {
  private _data: T[];
  private _copied: boolean = false;
  
  constructor(data: T[]) {
    this._data = data;
  }
  
  private ensureCopied(): void {
    if (!this._copied) {
      this._data = [...this._data];
      this._copied = true;
    }
  }
  
  get(index: number): T {
    return this._data[index];
  }
  
  set(index: number, value: T): void {
    this.ensureCopied();
    this._data[index] = value;
  }
  
  push(...items: T[]): number {
    this.ensureCopied();
    return this._data.push(...items);
  }
  
  pop(): T | undefined {
    this.ensureCopied();
    return this._data.pop();
  }
  
  slice(start?: number, end?: number): T[] {
    // Для операций чтения не нужно копировать
    return this._data.slice(start, end);
  }
  
  toArray(): T[] {
    return [...this._data];
  }
}

// Использование
const originalData = [1, 2, 3, 4, 5];
const cowArray = new CopyOnWriteArray(originalData);

// Операции чтения не вызывают копирование
console.log(cowArray.slice(0, 3)); // [1, 2, 3] - без копирования

// Операции записи вызывают копирование
cowArray.set(0, 10); // Теперь данные скопированы
cowArray.push(6); // Работаем с копией

console.log(originalData); // [1, 2, 3, 4, 5] - оригинальный массив не изменился
console.log(cowArray.toArray()); // [10, 2, 3, 4, 5, 6] - копия изменена
```

## Паттерны обновления неизменяемых структур

### 1. Lens Pattern (Паттерн линз)

Линзы предоставляют функциональный способ доступа и обновления вложенных свойств.

```typescript
// Базовый интерфейс линзы
interface Lens<S, A> {
  get: (state: S) => A;
  set: (value: A, state: S) => S;
}

// Создание линзы
const lens = <S, A>(get: (s: S) => A, set: (a: A, s: S) => S): Lens<S, A> => ({
  get,
  set
});

// Композиция линз
const composeLens = <A, B, C>(lens1: Lens<A, B>, lens2: Lens<B, C>): Lens<A, C> => ({
  get: (a) => lens2.get(lens1.get(a)),
  set: (c, a) => lens1.set(lens2.set(c, lens1.get(a)), a)
});

// Пример использования с пользователем
interface Address {
  readonly street: string;
  readonly city: string;
  readonly zipCode: string;
}

interface User {
  readonly id: number;
  readonly name: string;
  readonly address: Address;
}

// Линзы для доступа к свойствам
const addressLens: Lens<User, Address> = lens(
  (user) => user.address,
  (address, user) => ({ ...user, address })
);

const cityLens: Lens<Address, string> = lens(
  (address) => address.city,
  (city, address) => ({ ...address, city })
);

// Композиция линз
const userCityLens = composeLens(addressLens, cityLens);

// Использование
const user: User = {
  id: 1,
  name: 'John Doe',
  address: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001'
  }
};

// Чтение
const city = userCityLens.get(user); // 'New York'

// Обновление
const updatedUser = userCityLens.set('Los Angeles', user);
console.log(updatedUser.address.city); // 'Los Angeles'
```

### 2. Builder Pattern (Паттерн строителя)

Строитель предоставляет удобный способ создания сложных неизменяемых объектов пошагово.

```typescript
// Функциональный Builder для сложных объектов
class UserBuilder {
  private user: Partial<User> = {};
  
  static create(): UserBuilder {
    return new UserBuilder();
  }
  
  withId(id: number): this {
    this.user.id = id;
    return this;
  }
  
  withName(name: string): this {
    this.user.name = name;
    return this;
  }
  
  withEmail(email: string): this {
    this.user.email = email;
    return this;
  }
  
  withAddress(address: Address): this {
    this.user.address = address;
    return this;
  }
  
  build(): User {
    if (!this.user.id || !this.user.name || !this.user.email || !this.user.address) {
      throw new Error('Missing required fields');
    }
    
    return Object.freeze({
      id: this.user.id,
      name: this.user.name,
      email: this.user.email,
      address: Object.freeze(this.user.address)
    });
  }
}

// Использование
const user = UserBuilder.create()
  .withId(1)
  .withName('John Doe')
  .withEmail('john@example.com')
  .withAddress({
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001'
  })
  .build();
```

### 3. Factory Pattern (Фабричный паттерн)

Фабрики предоставляют централизованный способ создания неизменяемых объектов.

```typescript
// Фабрика для создания пользователей
class UserFactory {
  static create(
    id: number,
    name: string,
    email: string,
    address: Omit<Address, 'zipCode'> & { zipCode?: string }
  ): User {
    return Object.freeze({
      id,
      name,
      email,
      address: Object.freeze({
        street: address.street,
        city: address.city,
        zipCode: address.zipCode || '00000'
      })
    });
  }
  
  static createAdmin(
    id: number,
    name: string,
    email: string
  ): User {
    return this.create(id, name, email, {
      street: 'Admin Street 1',
      city: 'Admin City',
      zipCode: 'ADMIN'
    });
  }
  
  static createGuest(id: number): User {
    return this.create(id, 'Guest', 'guest@example.com', {
      street: 'Guest Street',
      city: 'Guest City'
    });
  }
}

// Использование
const regularUser = UserFactory.create(
  1,
  'John Doe',
  'john@example.com',
  { street: '123 Main St', city: 'New York', zipCode: '10001' }
);

const adminUser = UserFactory.createAdmin(2, 'Admin', 'admin@example.com');
const guestUser = UserFactory.createGuest(3);
```

## Паттерны работы с коллекциями

### 1. Pipeline Pattern (Паттерн конвейера)

Конвейер позволяет объединять операции над коллекциями в цепочку.

```typescript
// Pipeline для цепочки операций над массивами
const pipe = <T>(value: T, ...fns: Array<(arg: T) => T>): T =>
  fns.reduce((acc, fn) => fn(acc), value);

// Утилиты для работы с массивами в стиле pipeline
const ArrayPipeline = {
  map: <T, U>(fn: (item: T) => U) => 
    (array: readonly T[]): readonly U[] => 
      Object.freeze(array.map(fn)),
  
  filter: <T>(predicate: (item: T) => boolean) => 
    (array: readonly T[]): readonly T[] => 
      Object.freeze(array.filter(predicate)),
  
  sort: <T>(compareFn?: (a: T, b: T) => number) => 
    (array: readonly T[]): readonly T[] => 
      Object.freeze([...array].sort(compareFn)),
  
  take: <T>(count: number) => 
    (array: readonly T[]): readonly T[] => 
      Object.freeze(array.slice(0, count)),
  
  skip: <T>(count: number) => 
    (array: readonly T[]): readonly T[] => 
      Object.freeze(array.slice(count)),
  
  groupBy: <T, K extends string | number>(
    keyFn: (item: T) => K
  ) => (array: readonly T[]): Record<K, readonly T[]> => 
    array.reduce((groups, item) => {
      const key = keyFn(item);
      return {
        ...groups,
        [key]: [...(groups[key] || []), item]
      };
    }, {} as Record<K, readonly T[]>)
};

// Пример использования pipeline
const numbers = Object.freeze([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

const result = pipe(
  numbers,
  ArrayPipeline.filter(n => n % 2 === 0), // [2, 4, 6, 8, 10]
  ArrayPipeline.map(n => n * 2),          // [4, 8, 12, 16, 20]
  ArrayPipeline.sort((a, b) => b - a),    // [20, 16, 12, 8, 4]
  ArrayPipeline.take(3)                   // [20, 16, 12]
);

console.log(result); // [20, 16, 12]
```

### 2. Batch Operations Pattern (Паттерн пакетных операций)

Пакетные операции позволяют выполнять множественные изменения за один проход.

```typescript
// Batch операции для множественных изменений
const ArrayBatch = {
  // Множественное обновление
  updateMany: <T>(
    array: readonly T[],
    updates: { index: number; value: T }[]
  ): readonly T[] => {
    const updateMap = new Map(updates.map(u => [u.index, u.value]));
    
    return Object.freeze(array.map((item, index) => 
      updateMap.has(index) ? updateMap.get(index)! : item
    ));
  },
  
  // Множественное удаление
  removeMany: <T>(
    array: readonly T[],
    indices: number[]
  ): readonly T[] => {
    const indicesSet = new Set(indices);
    
    return Object.freeze(array.filter((_, index) => !indicesSet.has(index)));
  },
  
  // Множественное добавление
  insertMany: <T>(
    array: readonly T[],
    inserts: { index: number; items: T[] }[]
  ): readonly T[] => {
    // Сортируем вставки по индексу в обратном порядке
    // чтобы индексы не сдвигались при вставке
    const sortedInserts = [...inserts].sort((a, b) => b.index - a.index);
    
    let result = [...array];
    
    for (const { index, items } of sortedInserts) {
      result.splice(index, 0, ...items);
    }
    
    return Object.freeze(result);
  }
};

// Пример использования
const numbers = Object.freeze([1, 2, 3, 4, 5]);

const updated = ArrayBatch.updateMany(numbers, [
  { index: 1, value: 20 },
  { index: 3, value: 40 }
]); // [1, 20, 3, 40, 5]

const filtered = ArrayBatch.removeMany(numbers, [1, 3]); // [1, 3, 5]

const withInserts = ArrayBatch.insertMany(numbers, [
  { index: 2, items: [25, 26] },
  { index: 0, items: [0] }
]); // [0, 1, 2, 25, 26, 3, 4, 5]
```

## Паттерны оптимизации неизменяемости

### 1. Memoization Pattern (Паттерн мемоизации)

Мемоизация кэширует результаты вычислений для повышения производительности.

```typescript
// Мемоизация для чистых функций
const memoize = <T extends (...args: any[]) => any>(fn: T): T => {
  const cache = new Map<string, ReturnType<T>>();
  
  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
};

// Мемоизация для методов классов
const memoizeMethod = <T>(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) => {
  const originalMethod = descriptor.value;
  const cache = new Map<string, any>();
  
  descriptor.value = function(...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
};

// Пример использования
class ExpensiveCalculator {
  @memoizeMethod
  fibonacci(n: number): number {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
  
  @memoizeMethod
  complexCalculation(a: number, b: number, c: number): number {
    // Сложные вычисления
    return Math.pow(a, b) + Math.sqrt(c);
  }
}

const calculator = new ExpensiveCalculator();
console.log(calculator.fibonacci(10)); // Вычисляется
console.log(calculator.fibonacci(10)); // Берется из кэша
```

### 2. Lazy Evaluation Pattern (Паттерн ленивых вычислений)

Ленивые вычисления откладывают выполнение операций до момента их необходимости.

```typescript
// Lazy вычисления для массивов
class LazyArray<T> {
  private readonly _thunk: () => readonly T[];
  private _value: readonly T[] | null = null;
  private _evaluated: boolean = false;
  
  constructor(thunk: () => readonly T[]) {
    this._thunk = thunk;
  }
  
  private evaluate(): readonly T[] {
    if (!this._evaluated) {
      this._value = this._thunk();
      this._evaluated = true;
    }
    
    return this._value!;
  }
  
  map<U>(fn: (item: T) => U): LazyArray<U> {
    return new LazyArray(() => this.evaluate().map(fn));
  }
  
  filter(predicate: (item: T) => boolean): LazyArray<T> {
    return new LazyArray(() => this.evaluate().filter(predicate));
  }
  
  take(count: number): LazyArray<T> {
    return new LazyArray(() => this.evaluate().slice(0, count));
  }
  
  toArray(): readonly T[] {
    return this.evaluate();
  }
  
  static of<T>(...items: T[]): LazyArray<T> {
    return new LazyArray(() => Object.freeze(items));
  }
  
  static from<T>(items: readonly T[]): LazyArray<T> {
    return new LazyArray(() => Object.freeze([...items]));
  }
}

// Пример использования
const lazyArray = LazyArray.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
  .filter(n => n % 2 === 0)  // Ленивое вычисление
  .map(n => n * 2)           // Ленивое вычисление
  .take(3);                  // Ленивое вычисление

// Вычисления происходят только здесь
console.log(lazyArray.toArray()); // [4, 8, 12]
```

## Распространенные ошибки и лучшие практики

### 1. Избегайте поверхностного копирования

```typescript
// Ошибка: поверхностное копирование
const updateUserBad = (user: User, city: string): User => ({
  ...user,
  address: {
    ...user.address,
    city // Это правильно
  }
});

// Но если address содержит вложенные объекты, это может быть проблемой
interface ComplexAddress {
  readonly street: string;
  readonly city: string;
  readonly coordinates: {
    readonly lat: number;
    readonly lng: number;
  };
}

interface ComplexUser {
  readonly id: number;
  readonly name: string;
  readonly address: ComplexAddress;
}

// Ошибка: неполная копия вложенных объектов
const updateCoordinatesBad = (
  user: ComplexUser, 
  lat: number
): ComplexUser => ({
  ...user,
  address: {
    ...user.address,
    coordinates: {
      lat, // Забыли lng
      lng: user.address.coordinates.lng
    }
  }
});

// Правильно: полная копия
const updateCoordinatesGood = (
  user: ComplexUser, 
  lat: number
): ComplexUser => ({
  ...user,
  address: {
    ...user.address,
    coordinates: {
      ...user.address.coordinates,
      lat
    }
  }
});
```

### 2. Используйте утилиты для глубокого копирования

```typescript
// Утилита для глубокого копирования
const deepFreeze = <T>(obj: T): T => {
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  if (Array.isArray(obj)) {
    return Object.freeze(obj.map(item => deepFreeze(item))) as unknown as T;
  }
  
  const frozenObj = {} as T;
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      (frozenObj as any)[key] = deepFreeze((obj as any)[key]);
    }
  }
  
  return Object.freeze(frozenObj);
};

// Использование
const complexObject = {
  user: {
    id: 1,
    name: 'John',
    preferences: {
      theme: 'light',
      notifications: true
    }
  },
  settings: {
    language: 'en',
    timezone: 'UTC'
  }
};

const frozenObject = deepFreeze(complexObject);
// Все уровни объекта теперь заморожены
```

### 3. Создавайте специализированные утилиты

```typescript
// Специализированные утилиты для работы с неизменяемыми структурами
const ImmutableUtils = {
  // Глубокое обновление по пути
  deepUpdate<T>(obj: T, path: (string | number)[], value: any): T {
    if (path.length === 0) return value;
    
    const [head, ...tail] = path;
    const current = (obj as any)[head];
    
    return {
      ...obj,
      [head]: tail.length === 0 
        ? value 
        : this.deepUpdate(current, tail, value)
    } as T;
  },
  
  // Глубокое слияние объектов
  deepMerge<T>(obj1: T, obj2: Partial<T>): T {
    const result = { ...obj1 } as any;
    
    for (const key in obj2) {
      if (obj2.hasOwnProperty(key)) {
        if (
          obj2[key] !== null && 
          typeof obj2[key] === 'object' && 
          !Array.isArray(obj2[key]) &&
          result[key] !== null &&
          typeof result[key] === 'object'
        ) {
          result[key] = this.deepMerge(result[key], obj2[key] as any);
        } else {
          result[key] = obj2[key];
        }
      }
    }
    
    return result as T;
  },
  
  // Безопасное обновление массивов
  updateArray<T>(
    array: readonly T[], 
    index: number, 
    updater: (item: T) => T
  ): readonly T[] {
    if (index < 0 || index >= array.length) {
      return array;
    }
    
    const newArray = [...array];
    newArray[index] = updater(array[index]);
    
    return Object.freeze(newArray);
  }
};

// Пример использования
const user: User = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  address: {
    street: '123 Main St',
    city: 'New York',
    zipCode: '10001'
  }
};

const updatedUser = ImmutableUtils.deepUpdate(
  user,
  ['address', 'city'],
  'Los Angeles'
);

const mergedUser = ImmutableUtils.deepMerge(user, {
  name: 'Jane Doe',
  address: {
    zipCode: '90210'
  }
});
```

## Связи с другими концепциями

- [[ts/functional-programming/immutability/Неизменяемость|Неизменяемость]] - Основная страница раздела
- [[ts/functional-programming/immutability/immutable-objects|Неизменяемые объекты]] - Работа с неизменяемыми объектами
- [[ts/functional-programming/immutability/immutable-arrays|Неизменяемые массивы]] - Работа с неизменяемыми массивами
- [[ts/functional-programming/immutability/immutable-classes|Неизменяемые классы]] - Создание неизменяемых классов
- [[ts/functional-programming/pure-functions|Чистые функции]] - Работа с чистыми функциями
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции

## Следующие шаги

- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Освоение функций, принимающих или возвращающих другие функции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
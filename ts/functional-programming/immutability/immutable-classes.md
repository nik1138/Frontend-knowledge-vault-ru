# Неизменяемые классы

Неизменяемые классы - это классы, экземпляры которых не могут быть изменены после создания. Все свойства таких классов являются readonly, а методы возвращают новые экземпляры вместо изменения существующих.

## Создание неизменяемых классов

### Базовый пример неизменяемого класса

```typescript
// Базовый неизменяемый класс
class ImmutablePoint {
  constructor(
    public readonly x: number,
    public readonly y: number
  ) {}
  
  // Методы возвращают новые экземпляры
  move(dx: number, dy: number): ImmutablePoint {
    return new ImmutablePoint(this.x + dx, this.y + dy);
  }
  
  scale(factor: number): ImmutablePoint {
    return new ImmutablePoint(this.x * factor, this.y * factor);
  }
  
  // Геттеры для вычисляемых свойств
  get distanceFromOrigin(): number {
    return Math.sqrt(this.x * this.x + this.y * this.y);
  }
  
  // Статические фабричные методы
  static origin(): ImmutablePoint {
    return new ImmutablePoint(0, 0);
  }
  
  static fromPolar(radius: number, angle: number): ImmutablePoint {
    return new ImmutablePoint(
      radius * Math.cos(angle),
      radius * Math.sin(angle)
    );
  }
}

// Использование
const point = new ImmutablePoint(3, 4);
const movedPoint = point.move(1, 2); // Новый экземпляр
const scaledPoint = point.scale(2); // Новый экземпляр

console.log(point.x, point.y); // 3, 4 (оригинал не изменился)
console.log(movedPoint.x, movedPoint.y); // 4, 6
console.log(scaledPoint.x, scaledPoint.y); // 6, 8
```

### Неизменяемый класс с вложенными объектами

```typescript
// Неизменяемый класс с вложенными объектами
class ImmutableUser {
  constructor(
    public readonly id: number,
    public readonly name: string,
    public readonly email: string,
    public readonly preferences: ImmutableUserPreferences
  ) {}
  
  // Методы для обновления свойств
  withName(name: string): ImmutableUser {
    return new ImmutableUser(
      this.id,
      name,
      this.email,
      this.preferences
    );
  }
  
  withEmail(email: string): ImmutableUser {
    return new ImmutableUser(
      this.id,
      this.name,
      email,
      this.preferences
    );
  }
  
  withPreferences(preferences: ImmutableUserPreferences): ImmutableUser {
    return new ImmutableUser(
      this.id,
      this.name,
      this.email,
      preferences
    );
  }
  
  // Методы для обновления вложенных объектов
  withTheme(theme: 'light' | 'dark'): ImmutableUser {
    return new ImmutableUser(
      this.id,
      this.name,
      this.email,
      this.preferences.withTheme(theme)
    );
  }
  
  withNotifications(notifications: boolean): ImmutableUser {
    return new ImmutableUser(
      this.id,
      this.name,
      this.email,
      this.preferences.withNotifications(notifications)
    );
  }
}

class ImmutableUserPreferences {
  constructor(
    public readonly theme: 'light' | 'dark',
    public readonly notifications: boolean
  ) {}
  
  withTheme(theme: 'light' | 'dark'): ImmutableUserPreferences {
    return new ImmutableUserPreferences(theme, this.notifications);
  }
  
  withNotifications(notifications: boolean): ImmutableUserPreferences {
    return new ImmutableUserPreferences(this.theme, notifications);
  }
}

// Использование
const user = new ImmutableUser(
  1,
  'John Doe',
  'john@example.com',
  new ImmutableUserPreferences('light', true)
);

const updatedUser = user
  .withName('Jane Doe')
  .withTheme('dark');

console.log(user.name); // 'John Doe' (оригинал не изменился)
console.log(updatedUser.name); // 'Jane Doe'
console.log(updatedUser.preferences.theme); // 'dark'
```

## Паттерны создания неизменяемых классов

### Builder паттерн

```typescript
// Builder для неизменяемых классов
class ImmutableUserBuilder {
  private id: number = 0;
  private name: string = '';
  private email: string = '';
  private preferences: ImmutableUserPreferences = 
    new ImmutableUserPreferences('light', true);
  
  static create(): ImmutableUserBuilder {
    return new ImmutableUserBuilder();
  }
  
  withId(id: number): this {
    this.id = id;
    return this;
  }
  
  withName(name: string): this {
    this.name = name;
    return this;
  }
  
  withEmail(email: string): this {
    this.email = email;
    return this;
  }
  
  withPreferences(preferences: ImmutableUserPreferences): this {
    this.preferences = preferences;
    return this;
  }
  
  build(): ImmutableUser {
    if (!this.id || !this.name || !this.email) {
      throw new Error('Missing required fields');
    }
    
    return new ImmutableUser(
      this.id,
      this.name,
      this.email,
      this.preferences
    );
  }
}

// Использование
const user = ImmutableUserBuilder.create()
  .withId(1)
  .withName('John Doe')
  .withEmail('john@example.com')
  .withPreferences(new ImmutableUserPreferences('dark', false))
  .build();
```

### Фабричные методы

```typescript
// Неизменяемый класс с фабричными методами
class ImmutableRectangle {
  constructor(
    public readonly width: number,
    public readonly height: number
  ) {}
  
  // Фабричные методы
  static square(side: number): ImmutableRectangle {
    return new ImmutableRectangle(side, side);
  }
  
  static fromDimensions(width: number, height: number): ImmutableRectangle {
    return new ImmutableRectangle(width, height);
  }
  
  static fromAreaAndRatio(area: number, ratio: number): ImmutableRectangle {
    const width = Math.sqrt(area * ratio);
    const height = area / width;
    return new ImmutableRectangle(width, height);
  }
  
  // Методы для создания новых экземпляров
  scale(factor: number): ImmutableRectangle {
    return new ImmutableRectangle(
      this.width * factor,
      this.height * factor
    );
  }
  
  rotate(): ImmutableRectangle {
    return new ImmutableRectangle(this.height, this.width);
  }
  
  // Геттеры для вычисляемых свойств
  get area(): number {
    return this.width * this.height;
  }
  
  get perimeter(): number {
    return 2 * (this.width + this.height);
  }
  
  get isSquare(): boolean {
    return this.width === this.height;
  }
}

// Использование
const rect1 = ImmutableRectangle.square(5);
const rect2 = ImmutableRectangle.fromDimensions(3, 4);
const rect3 = ImmutableRectangle.fromAreaAndRatio(20, 2);

console.log(rect1.area); // 25
console.log(rect2.area); // 12
console.log(rect3.area); // 20
```

### Value Object паттерн

```typescript
// Value Object - объект, идентичность которого определяется его значениями
class ImmutableMoney {
  constructor(
    public readonly amount: number,
    public readonly currency: string
  ) {
    if (amount < 0) {
      throw new Error('Amount cannot be negative');
    }
  }
  
  // Операции с деньгами
  add(other: ImmutableMoney): ImmutableMoney {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    
    return new ImmutableMoney(this.amount + other.amount, this.currency);
  }
  
  subtract(other: ImmutableMoney): ImmutableMoney {
    if (this.currency !== other.currency) {
      throw new Error('Cannot subtract different currencies');
    }
    
    const result = this.amount - other.amount;
    if (result < 0) {
      throw new Error('Result cannot be negative');
    }
    
    return new ImmutableMoney(result, this.currency);
  }
  
  multiply(factor: number): ImmutableMoney {
    if (factor < 0) {
      throw new Error('Factor cannot be negative');
    }
    
    return new ImmutableMoney(this.amount * factor, this.currency);
  }
  
  // Сравнение value objects
  equals(other: ImmutableMoney): boolean {
    return this.amount === other.amount && this.currency === other.currency;
  }
  
  // Форматирование
  toString(): string {
    return `${this.amount} ${this.currency}`;
  }
  
  // Фабричные методы
  static zero(currency: string): ImmutableMoney {
    return new ImmutableMoney(0, currency);
  }
  
  static fromString(amount: string, currency: string): ImmutableMoney {
    const parsedAmount = parseFloat(amount);
    if (isNaN(parsedAmount)) {
      throw new Error('Invalid amount');
    }
    
    return new ImmutableMoney(parsedAmount, currency);
  }
}

// Использование
const price = new ImmutableMoney(100, 'USD');
const tax = new ImmutableMoney(10, 'USD');
const total = price.add(tax);

console.log(price.toString()); // "100 USD"
console.log(total.toString()); // "110 USD"
console.log(price.equals(total)); // false
```

## Работа с коллекциями неизменяемых объектов

### Неизменяемая коллекция

```typescript
// Неизменяемая коллекция объектов
class ImmutableCollection<T> {
  private readonly items: readonly T[];
  
  constructor(items: readonly T[] = []) {
    this.items = Object.freeze([...items]);
  }
  
  // Получение элементов
  get length(): number {
    return this.items.length;
  }
  
  get(index: number): T | undefined {
    return this.items[index];
  }
  
  toArray(): readonly T[] {
    return this.items;
  }
  
  // Операции добавления
  add(item: T): ImmutableCollection<T> {
    return new ImmutableCollection([...this.items, item]);
  }
  
  addAll(items: readonly T[]): ImmutableCollection<T> {
    return new ImmutableCollection([...this.items, ...items]);
  }
  
  // Операции удаления
  remove(predicate: (item: T) => boolean): ImmutableCollection<T> {
    return new ImmutableCollection(
      this.items.filter(item => !predicate(item))
    );
  }
  
  removeAt(index: number): ImmutableCollection<T> {
    if (index < 0 || index >= this.items.length) {
      return this;
    }
    
    return new ImmutableCollection([
      ...this.items.slice(0, index),
      ...this.items.slice(index + 1)
    ]);
  }
  
  // Операции обновления
  updateAt(index: number, updater: (item: T) => T): ImmutableCollection<T> {
    if (index < 0 || index >= this.items.length) {
      return this;
    }
    
    const updatedItems = [...this.items];
    updatedItems[index] = updater(this.items[index]);
    
    return new ImmutableCollection(updatedItems);
  }
  
  // Функции высшего порядка
  map<U>(fn: (item: T) => U): ImmutableCollection<U> {
    return new ImmutableCollection(this.items.map(fn));
  }
  
  filter(predicate: (item: T) => boolean): ImmutableCollection<T> {
    return new ImmutableCollection(this.items.filter(predicate));
  }
  
  reduce<U>(reducer: (accumulator: U, item: T) => U, initialValue: U): U {
    return this.items.reduce(reducer, initialValue);
  }
  
  find(predicate: (item: T) => boolean): T | undefined {
    return this.items.find(predicate);
  }
  
  // Статические фабричные методы
  static empty<T>(): ImmutableCollection<T> {
    return new ImmutableCollection<T>();
  }
  
  static of<T>(...items: T[]): ImmutableCollection<T> {
    return new ImmutableCollection(items);
  }
}

// Использование
const numbers = ImmutableCollection.of(1, 2, 3, 4, 5);
const doubled = numbers.map(n => n * 2);
const evens = doubled.filter(n => n % 2 === 0);
const sum = evens.reduce((acc, n) => acc + n, 0);

console.log(numbers.toArray()); // [1, 2, 3, 4, 5]
console.log(doubled.toArray()); // [2, 4, 6, 8, 10]
console.log(evens.toArray()); // [2, 4, 6, 8, 10]
console.log(sum); // 30
```

### Специализированная коллекция для пользователей

```typescript
// Специализированная коллекция для пользователей
class ImmutableUserCollection {
  private readonly users: readonly ImmutableUser[];
  private readonly userMap: ReadonlyMap<number, ImmutableUser>;
  
  constructor(users: readonly ImmutableUser[] = []) {
    this.users = Object.freeze([...users]);
    this.userMap = new Map(users.map(user => [user.id, user]));
  }
  
  // Получение пользователей
  get length(): number {
    return this.users.length;
  }
  
  get(id: number): ImmutableUser | undefined {
    return this.userMap.get(id);
  }
  
  getAll(): readonly ImmutableUser[] {
    return this.users;
  }
  
  // Поиск пользователей
  findByName(name: string): ImmutableUser | undefined {
    return this.users.find(user => user.name === name);
  }
  
  findByEmail(email: string): ImmutableUser | undefined {
    return this.users.find(user => user.email === email);
  }
  
  // Фильтрация пользователей
  filterByTheme(theme: 'light' | 'dark'): ImmutableUserCollection {
    return new ImmutableUserCollection(
      this.users.filter(user => user.preferences.theme === theme)
    );
  }
  
  filterByNotifications(notifications: boolean): ImmutableUserCollection {
    return new ImmutableUserCollection(
      this.users.filter(user => user.preferences.notifications === notifications)
    );
  }
  
  // Добавление пользователей
  add(user: ImmutableUser): ImmutableUserCollection {
    if (this.userMap.has(user.id)) {
      throw new Error(`User with id ${user.id} already exists`);
    }
    
    return new ImmutableUserCollection([...this.users, user]);
  }
  
  // Обновление пользователей
  update(id: number, updater: (user: ImmutableUser) => ImmutableUser): ImmutableUserCollection {
    const index = this.users.findIndex(user => user.id === id);
    
    if (index === -1) {
      throw new Error(`User with id ${id} not found`);
    }
    
    const updatedUser = updater(this.users[index]);
    const updatedUsers = [...this.users];
    updatedUsers[index] = updatedUser;
    
    return new ImmutableUserCollection(updatedUsers);
  }
  
  // Удаление пользователей
  remove(id: number): ImmutableUserCollection {
    const index = this.users.findIndex(user => user.id === id);
    
    if (index === -1) {
      return this;
    }
    
    const updatedUsers = [
      ...this.users.slice(0, index),
      ...this.users.slice(index + 1)
    ];
    
    return new ImmutableUserCollection(updatedUsers);
  }
  
  // Статические фабричные методы
  static empty(): ImmutableUserCollection {
    return new ImmutableUserCollection();
  }
  
  static fromArray(users: readonly ImmutableUser[]): ImmutableUserCollection {
    return new ImmutableUserCollection(users);
  }
}

// Использование
const users = ImmutableUserCollection.empty()
  .add(new ImmutableUser(1, 'John Doe', 'john@example.com', new ImmutableUserPreferences('light', true)))
  .add(new ImmutableUser(2, 'Jane Smith', 'jane@example.com', new ImmutableUserPreferences('dark', false)));

const darkThemeUsers = users.filterByTheme('dark');
const user1 = users.get(1);

const updatedUsers = users.update(1, user => user.withName('John Smith'));
```

## Преимущества неизменяемых классов

### 1. Потокобезопасность

```typescript
// Потокобезопасное использование неизменяемых классов
class ThreadSafeCounter {
  private readonly value: number;
  
  constructor(value: number = 0) {
    this.value = value;
  }
  
  increment(): ThreadSafeCounter {
    return new ThreadSafeCounter(this.value + 1);
  }
  
  decrement(): ThreadSafeCounter {
    return new ThreadSafeCounter(this.value - 1);
  }
  
  getValue(): number {
    return this.value;
  }
}

// Многопоточное использование безопасно
const counter = new ThreadSafeCounter(0);
const counter1 = counter.increment();
const counter2 = counter.increment();

console.log(counter.getValue()); // 0 (оригинал не изменился)
console.log(counter1.getValue()); // 1
console.log(counter2.getValue()); // 1
```

### 2. Упрощенное тестирование

```typescript
// Легко тестируемые неизменяемые классы
class ImmutableCalculator {
  constructor(public readonly initialValue: number) {}
  
  add(value: number): ImmutableCalculator {
    return new ImmutableCalculator(this.initialValue + value);
  }
  
  multiply(value: number): ImmutableCalculator {
    return new ImmutableCalculator(this.initialValue * value);
  }
  
  subtract(value: number): ImmutableCalculator {
    return new ImmutableCalculator(this.initialValue - value);
  }
  
  divide(value: number): ImmutableCalculator {
    if (value === 0) {
      throw new Error('Division by zero');
    }
    
    return new ImmutableCalculator(this.initialValue / value);
  }
}

// Тестирование
const calc = new ImmutableCalculator(10);
const result = calc.add(5).multiply(2).subtract(3);

console.assert(result.initialValue === 27); // (10 + 5) * 2 - 3 = 27
console.assert(calc.initialValue === 10); // Оригинал не изменился
```

### 3. Кэширование и мемоизация

```typescript
// Кэширование вычисляемых свойств
class ImmutableCircle {
  private _area: number | null = null;
  private _circumference: number | null = null;
  
  constructor(public readonly radius: number) {
    if (radius < 0) {
      throw new Error('Radius cannot be negative');
    }
  }
  
  get area(): number {
    if (this._area === null) {
      this._area = Math.PI * this.radius * this.radius;
    }
    
    return this._area;
  }
  
  get circumference(): number {
    if (this._circumference === null) {
      this._circumference = 2 * Math.PI * this.radius;
    }
    
    return this._circumference;
  }
  
  scale(factor: number): ImmutableCircle {
    return new ImmutableCircle(this.radius * factor);
  }
}

// Использование
const circle = new ImmutableCircle(5);
console.log(circle.area); // Вычисляется первый раз
console.log(circle.area); // Берется из кэша
console.log(circle.circumference); // Вычисляется первый раз
console.log(circle.circumference); // Берется из кэша
```

## Распространенные ошибки

### 1. Попытка изменения свойств напрямую

```typescript
// Ошибка: попытка изменения readonly свойств
const point = new ImmutablePoint(3, 4);
// point.x = 5; // Ошибка компиляции
// point.y = 6; // Ошибка компиляции
```

### 2. Изменение вложенных изменяемых объектов

```typescript
// Ошибка: изменение вложенных изменяемых объектов
class BadImmutableUser {
  constructor(
    public readonly id: number,
    public readonly name: string,
    public readonly preferences: { theme: string; notifications: boolean } // Изменяемый объект!
  ) {}
}

const badUser = new BadImmutableUser(1, 'John', { theme: 'light', notifications: true });
// badUser.preferences.theme = 'dark'; // Это изменит оригинальный объект!

// Правильно: использовать неизменяемые вложенные объекты
class GoodImmutableUser {
  constructor(
    public readonly id: number,
    public readonly name: string,
    public readonly preferences: ImmutableUserPreferences // Неизменяемый объект
  ) {}
}
```

### 3. Неправильная реализация методов обновления

```typescript
// Ошибка: возврат this вместо нового экземпляра
class BadImmutablePoint {
  constructor(
    public readonly x: number,
    public readonly y: number
  ) {}
  
  move(dx: number, dy: number): BadImmutablePoint {
    // this.x += dx; // Ошибка: нельзя изменить readonly свойство
    // this.y += dy; // Ошибка: нельзя изменить readonly свойство
    return this; // Ошибка: возвращаем оригинальный экземпляр
  }
}

// Правильно: создание нового экземпляра
class GoodImmutablePoint {
  constructor(
    public readonly x: number,
    public readonly y: number
  ) {}
  
  move(dx: number, dy: number): GoodImmutablePoint {
    return new GoodImmutablePoint(this.x + dx, this.y + dy);
  }
}
```

## Лучшие практики

### 1. Использование readonly модификаторов

```typescript
// Хорошо: все свойства readonly
class ImmutablePerson {
  constructor(
    public readonly name: string,
    public readonly age: number,
    public readonly email: string
  ) {}
}

// Плохо: отсутствие readonly модификаторов
class MutablePerson {
  constructor(
    public name: string,
    public age: number,
    public email: string
  ) {}
}
```

### 2. Валидация в конструкторе

```typescript
// Валидация входных данных
class ImmutableEmail {
  private readonly _value: string;
  
  constructor(email: string) {
    if (!this.isValidEmail(email)) {
      throw new Error('Invalid email format');
    }
    
    this._value = email.toLowerCase();
  }
  
  private isValidEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
  
  get value(): string {
    return this._value;
  }
  
  equals(other: ImmutableEmail): boolean {
    return this._value === other._value;
  }
  
  toString(): string {
    return this._value;
  }
}
```

### 3. Создание утилит для работы с неизменяемыми классами

```typescript
// Утилиты для работы с неизменяемыми классами
const ImmutableUtils = {
  // Глубокое сравнение
  deepEqual<T>(obj1: T, obj2: T): boolean {
    if (obj1 === obj2) return true;
    
    if (obj1 == null || obj2 == null) return obj1 === obj2;
    
    if (typeof obj1 !== 'object' || typeof obj2 !== 'object') {
      return obj1 === obj2;
    }
    
    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);
    
    if (keys1.length !== keys2.length) return false;
    
    for (const key of keys1) {
      if (!keys2.includes(key)) return false;
      
      const val1 = (obj1 as any)[key];
      const val2 = (obj2 as any)[key];
      
      if (!this.deepEqual(val1, val2)) return false;
    }
    
    return true;
  },
  
  // Создание копии с обновленными свойствами
  update<T>(obj: T, updates: Partial<T>): T {
    return { ...obj, ...updates } as T;
  }
};

// Пример использования
const user1 = new ImmutableUser(1, 'John', 'john@example.com', new ImmutableUserPreferences('light', true));
const user2 = new ImmutableUser(1, 'John', 'john@example.com', new ImmutableUserPreferences('light', true));

console.log(ImmutableUtils.deepEqual(user1, user2)); // true
```

## Связи с другими концепциями

- [[ts/functional-programming/immutability/Неизменяемость|Неизменяемость]] - Основная страница раздела
- [[ts/functional-programming/immutability/immutable-objects|Неизменяемые объекты]] - Работа с неизменяемыми объектами
- [[ts/functional-programming/immutability/immutable-arrays|Неизменяемые массивы]] - Работа с неизменяемыми массивами
- [[ts/functional-programming/pure-functions|Чистые функции]] - Работа с чистыми функциями
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции

## Следующие шаги

- [[ts/functional-programming/immutability/patterns|Паттерны неизменяемости]] - Распространенные паттерны работы с неизменяемыми структурами
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Освоение функций, принимающих или возвращающих другие функции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
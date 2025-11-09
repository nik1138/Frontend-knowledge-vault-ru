# Типы классов в TypeScript

Типы классов в TypeScript позволяют определять формы объектов с состоянием и поведением, обеспечивая структурную типизацию для классов, их экземпляров и конструкторов. Понимание типов классов критически важно для объектно-ориентированного программирования в TypeScript.

## Основные концепции типов классов

### 1. Простые классы и их типы

```typescript
class Person {
  name: string;
  age: number;
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  
  greet(): string {
    return `Hello, my name is ${this.name}`;
  }
}

// Тип экземпляра класса
const person: Person = new Person("Alice", 30);
console.log(person.greet()); // "Hello, my name is Alice"

// Тип конструктора
type PersonConstructor = new (name: string, age: number) => Person;
const PersonFactory: PersonConstructor = Person;
```

### 2. Типы конструкторов

```typescript
// Тип конструктора для класса
interface PersonConstructor {
  new (name: string, age: number): Person;
}

// Фабрика, создающая экземпляры
const createPerson: (ctor: PersonConstructor, name: string, age: number) => Person = 
  (ctor, name, age) => new ctor(name, age);

const newPerson = createPerson(Person, "Bob", 25);
```

## Наследование классов

### 1. Простое наследование

```typescript
class Animal {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  move(distance: number = 0): void {
    console.log(`${this.name} moved ${distance}m.`);
  }
}

class Dog extends Animal {
  breed: string;
  
  constructor(name: string, breed: string) {
    super(name); // вызов конструктора родительского класса
    this.breed = breed;
  }
  
  bark(): void {
    console.log("Woof! Woof!");
  }
  
  // Переопределение метода
  move(distance = 5) {
    console.log("Running...");
    super.move(distance); // вызов родительского метода
  }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.bark(); // "Woof! Woof!"
dog.move(10); // "Running... Buddy moved 10m."
```

### 2. Совместимость типов при наследовании

```typescript
class Cat extends Animal {
  constructor(name: string) {
    super(name);
  }
  
  meow(): void {
    console.log("Meow!");
  }
}

// Благодаря наследованию, Dog и Cat совместимы с Animal
const animals: Animal[] = [
  new Dog("Buddy", "Golden Retriever"),
  new Cat("Whiskers")
];

// Полиморфизм: вызов методов родительского класса
animals.forEach(animal => animal.move(5));
```

## Модификаторы доступа

### 1. Public, private, protected

```typescript
class BankAccount {
  public accountNumber: string;
  protected balance: number;
  private pin: string;
  
  constructor(accountNumber: string, initialBalance: number, pin: string) {
    this.accountNumber = accountNumber;
    this.balance = initialBalance;
    this.pin = pin;
  }
  
  // Публичный метод
  public deposit(amount: number): void {
    if (amount > 0) {
      this.balance += amount;
    }
  }
  
  // Защищенный метод (доступен в классе и подклассах)
  protected getBalance(): number {
    return this.balance;
  }
  
  // Приватный метод (доступен только в этом классе)
  private validatePin(inputPin: string): boolean {
    return this.pin === inputPin;
  }
  
  public withdraw(amount: number, inputPin: string): boolean {
    if (this.validatePin(inputPin) && amount <= this.balance) {
      this.balance -= amount;
      return true;
    }
    return false;
  }
}

class SavingsAccount extends BankAccount {
  private interestRate: number;
  
  constructor(
    accountNumber: string, 
    initialBalance: number, 
    pin: string, 
    interestRate: number
  ) {
    super(accountNumber, initialBalance, pin);
    this.interestRate = interestRate;
  }
  
  // Может обращаться к protected полю и методу
  public addInterest(): void {
    const interest = this.getBalance() * this.interestRate;
    this.deposit(interest);
  }
}

const account = new SavingsAccount("12345", 1000, "1234", 0.05);
account.deposit(500); // OK: public
// account.balance; // Ошибка: protected
// account.validatePin("1234"); // Ошибка: private
```

## Абстрактные классы

### 1. Определение абстрактных классов

```typescript
abstract class Shape {
  protected color: string;
  
  constructor(color: string) {
    this.color = color;
  }
  
  // Конкретный метод
  public getColor(): string {
    return this.color;
  }
  
  // Абстрактный метод (должен быть реализован в подклассах)
  public abstract calculateArea(): number;
  
  // Абстрактный метод с возвращаемым значением
  public abstract getPerimeter(): number;
}

class Circle extends Shape {
  private radius: number;
  
  constructor(color: string, radius: number) {
    super(color);
    this.radius = radius;
  }
  
  public calculateArea(): number {
    return Math.PI * this.radius ** 2;
  }
  
  public getPerimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rectangle extends Shape {
  private width: number;
  private height: number;
  
  constructor(color: string, width: number, height: number) {
    super(color);
    this.width = width;
    this.height = height;
  }
  
  public calculateArea(): number {
    return this.width * this.height;
  }
  
  public getPerimeter(): number {
    return 2 * (this.width + this.height);
  }
}

// const shape = new Shape("red"); // Ошибка: нельзя создать экземпляр абстрактного класса
const circle = new Circle("red", 5);
const rectangle = new Rectangle("blue", 4, 6);

console.log(circle.calculateArea()); // Площадь круга
console.log(rectangle.calculateArea()); // Площадь прямоугольника
```

## Статические члены

### 1. Статические свойства и методы

```typescript
class Grid {
  static origin = { x: 0, y: 0 };
  
  calculateDistanceFromOrigin(point: { x: number; y: number; }): number {
    const xDist = point.x - Grid.origin.x;
    const yDist = point.y - Grid.origin.y;
    return Math.sqrt(xDist ** 2 + yDist ** 2);
  }
}

const grid = new Grid();
console.log(grid.calculateDistanceFromOrigin({ x: 3, y: 4 })); // 5
console.log(Grid.origin); // { x: 0, y: 0 } - доступ к статическому свойству
```

### 2. Статические фабричные методы

```typescript
class User {
  private constructor(
    public id: number,
    public name: string,
    public email: string
  ) {}
  
  static createAdmin(name: string, email: string): User {
    return new User(Date.now(), name, email);
  }
  
  static createGuest(): User {
    return new User(-1, "Guest", "guest@example.com");
  }
  
  static fromJSON(json: string): User {
    const data = JSON.parse(json);
    return new User(data.id, data.name, data.email);
  }
}

const admin = User.createAdmin("Admin", "admin@example.com");
const guest = User.createGuest();
const userFromJson = User.fromJSON('{"id": 1, "name": "John", "email": "john@example.com"}');
```

## Обобщенные классы

### 1. Простые обобщенные классы

```typescript
class Queue<T> {
  private elements: T[] = [];
  
  constructor(private maxSize: number = Infinity) {}
  
  public enqueue(element: T): void {
    if (this.size() >= this.maxSize) {
      throw new Error("Queue is full");
    }
    this.elements.push(element);
  }
  
  public dequeue(): T | undefined {
    return this.elements.shift();
  }
  
  public size(): number {
    return this.elements.length;
  }
  
  public peek(): T | undefined {
    return this.elements[0];
  }
}

// Использование обобщенного класса
const stringQueue = new Queue<string>(5);
stringQueue.enqueue("Hello");
stringQueue.enqueue("World");

const numberQueue = new Queue<number>();
numberQueue.enqueue(1);
numberQueue.enqueue(2);
```

### 2. Обобщенные классы с ограничениями

```typescript
interface Lengthwise {
  length: number;
}

class ArrayUtils<T extends Lengthwise> {
  static logLength<T extends Lengthwise>(arg: T): T {
    console.log(arg.length); // Теперь мы знаем, что у arg есть свойство length
    return arg;
  }
  
  static getFirstElement<T extends Lengthwise>(arr: T[]): T | undefined {
    return arr[0];
  }
}

// Работает с любыми типами, у которых есть свойство length
ArrayUtils.logLength("Hello"); // 5
ArrayUtils.logLength([1, 2, 3]); // 3
ArrayUtils.logLength({ length: 10, name: "Alice" }); // 10

// const result = ArrayUtils.logLength(3); // Ошибка: number не имеет length
```

## Совместимость типов классов

### 1. Структурная типизация классов

```typescript
class Point {
  x: number;
  y: number;
  
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

class Point2D {
  x: number;
  y: number;
  
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
  
  distanceFromOrigin(): number {
    return Math.sqrt(this.x ** 2 + this.y ** 2);
  }
}

// Совместимость: структура совпадает
const point: Point = new Point2D(1, 2); // OK
// Но не наоборот без дополнительных полей
// const point2d: Point2D = new Point(1, 2); // Ошибка: у Point нет distanceFromOrigin()
```

### 2. Совместимость конструкторов

```typescript
interface PointConstructor {
  new (x: number, y: number): { x: number; y: number };
}

class Point3D {
  x: number;
  y: number;
  z: number;
  
  constructor(x: number, y: number, z: number = 0) {
    this.x = x;
    this.y = y;
    this.z = z;
  }
}

// Point3D совместим с PointConstructor, так как имеет нужные свойства
const createPoint: PointConstructor = Point3D;
const point = new createPoint(1, 2); // { x: number; y: number; z: number; }
```

## Практические применения

### 1. Репозитории данных

```typescript
interface Entity {
  id: number;
}

class Repository<T extends Entity> {
  protected entities: T[] = [];
  
  public findById(id: number): T | undefined {
    return this.entities.find(entity => entity.id === id);
  }
  
  public findAll(): T[] {
    return [...this.entities];
  }
  
  public save(entity: T): T {
    if (entity.id === 0) {
      // Новый элемент
      entity.id = this.generateId();
      this.entities.push(entity);
    } else {
      // Обновление существующего
      const index = this.entities.findIndex(e => e.id === entity.id);
      if (index !== -1) {
        this.entities[index] = entity;
      }
    }
    return entity;
  }
  
  public delete(id: number): boolean {
    const index = this.entities.findIndex(entity => entity.id === id);
    if (index !== -1) {
      this.entities.splice(index, 1);
      return true;
    }
    return false;
  }
  
  private generateId(): number {
    return this.entities.length > 0 
      ? Math.max(...this.entities.map(e => e.id)) + 1 
      : 1;
  }
}

interface User extends Entity {
  name: string;
  email: string;
}

class UserRepository extends Repository<User> {
  public findByEmail(email: string): User | undefined {
    return this.entities.find(user => user.email === email);
  }
  
  public setActiveStatus(id: number, active: boolean): void {
    const user = this.findById(id);
    if (user) {
      // Дополнительная логика установки статуса
    }
  }
}
```

### 2. Система событий

```typescript
abstract class EventEmitter {
  private listeners: { [event: string]: Function[] } = {};
  
  public on(event: string, callback: Function): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(callback);
  }
  
  public emit(event: string, ...args: any[]): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(callback => callback(...args));
    }
  }
  
  public off(event: string, callback: Function): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      const index = eventListeners.indexOf(callback);
      if (index > -1) {
        eventListeners.splice(index, 1);
      }
    }
  }
}

class UserService extends EventEmitter {
  private users: User[] = [];
  
  public addUser(user: User): void {
    this.users.push(user);
    this.emit('userAdded', user);
  }
  
  public removeUser(id: number): void {
    const index = this.users.findIndex(user => user.id === id);
    if (index !== -1) {
      const [removedUser] = this.users.splice(index, 1);
      this.emit('userRemoved', removedUser);
    }
  }
}
```

### 3. Система конфигурации

```typescript
abstract class Configurable<T = any> {
  protected config: T;
  
  constructor(initialConfig: T) {
    this.config = { ...initialConfig };
  }
  
  public set<K extends keyof T>(key: K, value: T[K]): void {
    this.config[key] = value;
  }
  
  public get<K extends keyof T>(key: K): T[K] {
    return this.config[key];
  }
  
  public update(newConfig: Partial<T>): void {
    this.config = { ...this.config, ...newConfig };
  }
  
  protected getConfig(): T {
    return { ...this.config }; // Возвращаем копию для защиты
  }
}

interface ServerConfig {
  host: string;
  port: number;
  ssl: boolean;
  timeout: number;
}

class Server extends Configurable<ServerConfig> {
  constructor(config: ServerConfig) {
    super(config);
  }
  
  public start(): void {
    const config = this.getConfig();
    console.log(`Сервер запущен на ${config.host}:${config.port}`);
  }
  
  public enableSSL(): void {
    this.set('ssl', true);
  }
  
  public setPort(port: number): void {
    if (port < 1 || port > 65535) {
      throw new Error('Неверный номер порта');
    }
    this.set('port', port);
  }
}
```

## Продвинутые паттерны

### 1. Паттерн "Строитель" (Builder)

```typescript
class UserBuilder {
  private user: Partial<User> = {};
  
  public withId(id: number): this {
    this.user.id = id;
    return this;
  }
  
  public withName(name: string): this {
    this.user.name = name;
    return this;
  }
  
  public withEmail(email: string): this {
    this.user.email = email;
    return this;
  }
  
  public withAge(age: number): this {
    // Можно добавить валидацию
    if (age < 0) {
      throw new Error("Возраст не может быть отрицательным");
    }
    this.user['age'] = age; // Предполагаем, что в интерфейсе User есть age
    return this;
  }
  
  public build(): User {
    if (!this.user.id || !this.user.name || !this.user.email) {
      throw new Error("Не все обязательные поля заполнены");
    }
    return this.user as User;
  }
}

// Использование
const user = new UserBuilder()
  .withId(1)
  .withName("John Doe")
  .withEmail("john@example.com")
  .build();
```

### 2. Паттерн "Фабрика" (Factory)

```typescript
abstract class Animal {
  abstract makeSound(): string;
  abstract move(): string;
}

class Dog extends Animal {
  makeSound(): string {
    return "Woof!";
  }
  
  move(): string {
    return "Running on four legs";
  }
}

class Bird extends Animal {
  makeSound(): string {
    return "Tweet!";
  }
  
  move(): string {
    return "Flying with wings";
  }
}

type AnimalType = 'dog' | 'bird';

class AnimalFactory {
  static create(type: AnimalType): Animal {
    switch (type) {
      case 'dog':
        return new Dog();
      case 'bird':
        return new Bird();
      default:
        throw new Error(`Неизвестный тип животного: ${type}`);
    }
  }
}

// Использование
const dog = AnimalFactory.create('dog');
console.log(dog.makeSound()); // "Woof!"
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки

```typescript
// Ошибка: забыли вызвать super() в конструкторе наследника
class WrongDog extends Animal {
  breed: string;
  
  constructor(name: string, breed: string) {
    // Ошибка: нужно сначала вызвать super()
    this.breed = breed; // Ошибка TS2377: Constructors for derived classes must contain a 'super' call
    super(name);
  }
}

// Правильный вариант
class CorrectDog extends Animal {
  breed: string;
  
  constructor(name: string, breed: string) {
    super(name); // Сначала вызываем родительский конструктор
    this.breed = breed; // Потом инициализируем свои поля
  }
}
```

### 2. Лучшие практики

```typescript
// 1. Используйте абстрактные классы для определения контрактов
abstract class DataProcessor<T> {
  abstract process(data: T): any;
  abstract validate(data: T): boolean;
  
  public execute(data: T): any {
    if (!this.validate(data)) {
      throw new Error("Данные не прошли валидацию");
    }
    return this.process(data);
  }
}

// 2. Используйте обобщения для создания универсальных классов
class Container<T> {
  private item: T | null = null;
  
  public set(item: T): void {
    this.item = item;
  }
  
  public get(): T | null {
    return this.item;
  }
  
  public has(): boolean {
    return this.item !== null;
  }
}

// 3. Защищайте внутреннее состояние класса
class SafeCounter {
  private _count: number = 0;
  
  public get count(): number {
    return this._count;
  }
  
  public increment(): void {
    this._count++;
  }
  
  public reset(): void {
    this._count = 0;
  }
}
```

## Практические примеры из реальной разработки

### 1. Классы для работы с API

```typescript
abstract class ApiClient<T> {
  protected baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  abstract get(id: number): Promise<T>;
  abstract getAll(): Promise<T[]>;
  abstract create(item: Omit<T, 'id'>): Promise<T>;
  abstract update(id: number, item: Partial<T>): Promise<T>;
  abstract delete(id: number): Promise<void>;
  
  protected async request(url: string, options: RequestInit = {}): Promise<any> {
    const response = await fetch(`${this.baseUrl}${url}`, {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  }
}

interface Product {
  id: number;
  name: string;
  price: number;
}

class ProductApiClient extends ApiClient<Product> {
  async get(id: number): Promise<Product> {
    return this.request(`/${id}`);
  }
  
  async getAll(): Promise<Product[]> {
    return this.request('/');
  }
  
  async create(item: Omit<Product, 'id'>): Promise<Product> {
    return this.request('/', {
      method: 'POST',
      body: JSON.stringify(item)
    });
  }
  
  async update(id: number, item: Partial<Product>): Promise<Product> {
    return this.request(`/${id}`, {
      method: 'PUT',
      body: JSON.stringify(item)
    });
  }
  
  async delete(id: number): Promise<void> {
    await this.request(`/${id}`, { method: 'DELETE' });
  }
}
```

### 2. Система управления сессиями

```typescript
interface SessionData {
  userId: number;
  expiresAt: Date;
  permissions: string[];
}

class SessionManager {
  private sessions: Map<string, SessionData> = new Map();
  
  public createSession(userId: number, permissions: string[] = []): string {
    const sessionId = this.generateSessionId();
    const expiresAt = new Date();
    expiresAt.setDate(expiresAt.getDate() + 7); // Сессия действует неделю
    
    this.sessions.set(sessionId, {
      userId,
      expiresAt,
      permissions
    });
    
    return sessionId;
  }
  
  public validateSession(sessionId: string): boolean {
    const session = this.sessions.get(sessionId);
    if (!session) {
      return false;
    }
    
    return session.expiresAt > new Date();
  }
  
  public getSessionData(sessionId: string): SessionData | null {
    if (!this.validateSession(sessionId)) {
      return null;
    }
    
    return { ...this.sessions.get(sessionId)! }; // Возвращаем копию
  }
  
  public destroySession(sessionId: string): boolean {
    return this.sessions.delete(sessionId);
  }
  
  private generateSessionId(): string {
    return Math.random().toString(36).substring(2, 15) + 
           Math.random().toString(36).substring(2, 15);
  }
}
```

## Лучшие практики

1. **Используйте абстрактные классы для определения контрактов** - когда нужно задать структуру, но не реализацию
2. **Предпочитайте композицию наследованию** - особенно при создании гибких архитектур
3. **Используйте модификаторы доступа для защиты инкапсуляции** - не делайте всё public без необходимости
4. **Используйте обобщения для создания универсальных классов** - это делает классы более переиспользуемыми
5. **Документируйте интерфейсы классов** - особенно когда они используются как типы
6. **Рассматривайте интерфейсы вместо абстрактных классов** - когда нужна только форма, без реализации

## Связи с другими концепциями

- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]] - основы ООП в TypeScript
- [[ts/generics/TS Generics|Обобщения]] - основа для универсальных классов
- [[ts/type-system/type-compatibility|Совместимость типов]] - как классы проверяются на совместимость
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - альтернатива наследованию в некоторых случаях
- [[js/classes]] - JavaScript классы, на которых основаны типы
# Creational Patterns

Порождающие паттерны (Creational Patterns) отвечают за создание объектов. В TypeScript они обеспечивают гибкие и типобезопасные способы создания экземпляров классов, управления состоянием и инстанцированием сложных объектов.

## Основные порождающие паттерны

### [Factory Pattern](factory.md)
Определяет интерфейс для создания объекта, но позволяет подклассам решать, какой класс инстанцировать.

### [Abstract Factory](abstract-factory.md)
Предоставляет интерфейс для создания семейств связанных или зависимых объектов без указания их конкретных классов.

### [Builder Pattern](builder.md)
Разделяет конструирование сложного объекта от его представления, чтобы один и тот же процесс конструирования мог создавать разные представления.

### [Singleton Pattern](singleton.md)
Гарантирует, что класс имеет только один экземпляр, и предоставляет глобальную точку доступа к нему.

### [Prototype Pattern](prototype.md)
Определяет создание новых объектов путем клонирования существующего объекта, а не создания новых экземпляров.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Creational Design Patterns                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                             ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   Factory       │               │      Abstract Factory    │                                    │   Builder       │
  │  Pattern        │               │                          │                                    │  Pattern        │
  │                 │               │ - Family of Products     │                                    │                 │
  │ - Object Creation │───────────────┤ - Related Factories      │─────────────────────────────────────►│ - Step-by-step  │
  │ - Flexible      │               │ - Common Interface       │                                    │   Construction  │
  │   Instantiation │               │ - Cross-platform         │                                    │ - Complex Objects │
  │ - Type Safety   │               │ - Consistent Products    │                                    │ - Fluent API    │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │          Singleton              │
        │                           │                               │
        │                           │ - Single Instance             │
        │                           │ - Global Access               │
        │                           │ - Controlled Instantiation    │
        │                           │ - Thread Safety               │
        │                           │ - Lazy/Eager Initialization   │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                           Prototype                                 │
        │                           │                                                                 │
        │                           │ - Clone Existing Object                                             │
        │                           │ - Avoid Expensive Creation                                            │
        │                           │ - Custom Clone Logic                                                │
        │                           │ - Deep/Shadow Copy                                                  │
        │                           │ - Registry Pattern                                                  │
        └───────────────────────────┤ - Alternative to Constructor                                         │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Factory Pattern

### Простая фабрика
```typescript
interface Animal {
  speak(): void;
}

class Dog implements Animal {
  speak(): void {
    console.log("Woof!");
  }
}

class Cat implements Animal {
  speak(): void {
    console.log("Meow!");
  }
}

// Перечисление или union для типов животных
type AnimalType = 'dog' | 'cat';

// Простая фабрика
class SimpleAnimalFactory {
  static create(type: AnimalType): Animal {
    switch (type) {
      case 'dog':
        return new Dog();
      case 'cat':
        return new Cat();
      default:
        throw new Error(`Unknown animal type: ${type}`);
    }
  }
}

// Использование
const myDog = SimpleAnimalFactory.create('dog'); // Dog
const myCat = SimpleAnimalFactory.create('cat'); // Cat
myDog.speak(); // "Woof!"
```

### Фабричный метод
```typescript
abstract class AnimalCreator {
  // Шаблонный метод
  public createAnimal(): Animal {
    const animal = this.factoryMethod();
    this.setupAnimal(animal);
    return animal;
  }

  // Фабричный метод
  protected abstract factoryMethod(): Animal;

  private setupAnimal(animal: Animal): void {
    console.log("Setting up animal habitat...");
    animal.speak(); // подготовка включает проверку
  }
}

class DogCreator extends AnimalCreator {
  protected factoryMethod(): Animal {
    return new Dog();
  }
}

class CatCreator extends AnimalCreator {
  protected factoryMethod(): Animal {
    return new Cat();
  }
}

// Использование
const dogCreator = new DogCreator();
const dog = dogCreator.createAnimal(); // Создаст Dog с подготовкой

const catCreator = new CatCreator();
const cat = catCreator.createAnimal(); // Создаст Cat с подготовкой
```

### Обобщенная фабрика
```typescript
interface Product {
  getId(): string;
}

class ConcreteProductA implements Product {
  constructor(private id: string) {}
  
  getId(): string {
    return `ProductA-${this.id}`;
  }
}

class ConcreteProductB implements Product {
  constructor(private id: string) {}
  
  getId(): string {
    return `ProductB-${this.id}`;
  }
}

// Обобщенная фабрика с типами
interface ProductFactory<T extends Product> {
  create(id: string): T;
}

class ProductFactoryA implements ProductFactory<ConcreteProductA> {
  create(id: string): ConcreteProductA {
    return new ConcreteProductA(id);
  }
}

class ProductFactoryB implements ProductFactory<ConcreteProductB> {
  create(id: string): ConcreteProductB {
    return new ConcreteProductB(id);
  }
}

// Фабрика фабрик
class FactoryProducer {
  static getFactory<T extends Product>(
    type: 'A' | 'B'
  ): ProductFactory<T> {
    switch (type) {
      case 'A':
        return new ProductFactoryA() as ProductFactory<T>;
      case 'B':
        return new ProductFactoryB() as ProductFactory<T>;
      default:
        throw new Error(`Unknown factory type: ${type}`);
    }
  }
}

// Использование
const factoryA = FactoryProducer.getFactory<ConcreteProductA>('A');
const productA = factoryA.create('123');
console.log(productA.getId()); // "ProductA-123"

const factoryB = FactoryProducer.getFactory<ConcreteProductB>('B');
const productB = factoryB.create('456');
console.log(productB.getId()); // "ProductB-456"
```

## Abstract Factory

### Абстрактная фабрика для GUI
```typescript
// Продукты для кнопки
interface Button {
  render(): string;
  onClick(): void;
}

// Продукты для чекбокса
interface Checkbox {
  render(): string;
  toggle(): void;
}

// Windows реализации
class WindowsButton implements Button {
  render(): string {
    return "Rendering Windows button";
  }
  
  onClick(): void {
    console.log("Windows button clicked");
  }
}

class WindowsCheckbox implements Checkbox {
  render(): string {
    return "Rendering Windows checkbox";
  }
  
  toggle(): void {
    console.log("Windows checkbox toggled");
  }
}

// Mac реализации
class MacButton implements Button {
  render(): string {
    return "Rendering Mac button";
  }
  
  onClick(): void {
    console.log("Mac button clicked");
  }
}

class MacCheckbox implements Checkbox {
  render(): string {
    return "Rendering Mac checkbox";
  }
  
  toggle(): void {
    console.log("Mac checkbox toggled");
  }
}

// Абстрактная фабрика
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

class WindowsFactory implements GUIFactory {
  createButton(): Button {
    return new WindowsButton();
  }
  
  createCheckbox(): Checkbox {
    return new WindowsCheckbox();
  }
}

class MacFactory implements GUIFactory {
  createButton(): Button {
    return new MacButton();
  }
  
  createCheckbox(): Checkbox {
    return new MacCheckbox();
  }
}

// Приложение, использующее абстрактную фабрику
class Application {
  private button: Button;
  private checkbox: Checkbox;
  
  constructor(factory: GUIFactory) {
    this.button = factory.createButton();
    this.checkbox = factory.createCheckbox();
  }
  
  renderUI(): string {
    return this.button.render() + "\n" + this.checkbox.render();
  }
  
  handleInteraction(): void {
    this.button.onClick();
    this.checkbox.toggle();
  }
}

// Использование
const windowsApp = new Application(new WindowsFactory());
console.log(windowsApp.renderUI());

const macApp = new Application(new MacFactory());
console.log(macApp.renderUI());
```

### Фабрика с зависимостями
```typescript
interface DatabaseConnection {
  connect(): void;
  query(sql: string): any[];
}

interface Cache {
  get(key: string): any;
  set(key: string, value: any): void;
}

// Абстрактная фабрика инфраструктуры
interface InfrastructureFactory {
  createDatabase(): DatabaseConnection;
  createCache(): Cache;
}

// Реализации для Production
class ProductionDatabase implements DatabaseConnection {
  connect(): void { console.log("Connecting to production DB"); }
  query(sql: string): any[] { return []; }
}

class RedisCache implements Cache {
  get(key: string): any { return null; }
  set(key: string, value: any): void {}
}

class ProductionInfrastructureFactory implements InfrastructureFactory {
  createDatabase(): DatabaseConnection {
    return new ProductionDatabase();
  }
  
  createCache(): Cache {
    return new RedisCache();
  }
}

// Реализации для Development
class MockDatabase implements DatabaseConnection {
  connect(): void { console.log("Mock DB connection"); }
  query(sql: string): any[] { return [{ id: 1, name: "Mock" }]; }
}

class InMemoryCache implements Cache {
  private store = new Map<string, any>();
  
  get(key: string): any { return this.store.get(key); }
  set(key: string, value: any): void { this.store.set(key, value); }
}

class DevelopmentInfrastructureFactory implements InfrastructureFactory {
  createDatabase(): DatabaseConnection {
    return new MockDatabase();
  }
  
  createCache(): Cache {
    return new InMemoryCache();
  }
}

// Сервис, использующий фабрику
class UserService {
  private db: DatabaseConnection;
  private cache: Cache;
  
  constructor(factory: InfrastructureFactory) {
    this.db = factory.createDatabase();
    this.cache = factory.createCache();
  }
  
  async getUser(id: number) {
    const cached = this.cache.get(`user:${id}`);
    if (cached) return cached;
    
    this.db.connect();
    const [user] = this.db.query(`SELECT * FROM users WHERE id = ${id}`);
    this.cache.set(`user:${id}`, user);
    return user;
  }
}

// Использование
const prodService = new UserService(new ProductionInfrastructureFactory());
const devService = new UserService(new DevelopmentInfrastructureFactory());
```

## Builder Pattern

### Классический Builder
```typescript
interface Car {
  wheels: number;
  engine: string;
  color: string;
  doors: number;
}

class CarBuilder {
  private car: Car = { wheels: 0, engine: "", color: "white", doors: 0 };
  
  setWheels(wheels: number): this {
    if (wheels < 2 || wheels > 24) {
      throw new Error("Invalid number of wheels");
    }
    this.car.wheels = wheels;
    return this;
  }
  
  setEngine(engine: string): this {
    this.car.engine = engine;
    return this;
  }
  
  setColor(color: string): this {
    this.car.color = color;
    return this;
  }
  
  setDoors(doors: number): this {
    if (doors < 2 || doors > 5) {
      throw new Error("Invalid number of doors");
    }
    this.car.doors = doors;
    return this;
  }
  
  build(): Car {
    if (!this.isValid()) {
      throw new Error("Invalid car configuration");
    }
    return { ...this.car }; // возвращаем копию
  }
  
  private isValid(): boolean {
    return this.car.wheels > 0 && this.car.engine.length > 0 && this.car.doors > 0;
  }
}

// Использование
const car = new CarBuilder()
  .setWheels(4)
  .setEngine("V8")
  .setColor("red")
  .setDoors(2)
  .build();

console.log(car); // { wheels: 4, engine: "V8", color: "red", doors: 2 }
```

### Builder с обобщениями
```typescript
interface Buildable<T> {
  build(): T;
}

// Общая схема для билдера
abstract class GenericBuilder<T, B extends Buildable<T>> {
  protected data: Partial<T> = {};
  
  set<K extends keyof T>(key: K, value: T[K]): this {
    this.data[key] = value;
    return this;
  }
  
  abstract build(): T;
}

// Конкретный билдер
interface UserConfig {
  name: string;
  email: string;
  age: number;
  isActive: boolean;
}

class UserBuilder extends GenericBuilder<UserConfig, UserBuilder> {
  setName(name: string): this {
    return this.set('name', name);
  }
  
  setEmail(email: string): this {
    return this.set('email', email);
  }
  
  setAge(age: number): this {
    return this.set('age', age);
  }
  
  setIsActive(isActive: boolean): this {
    return this.set('isActive', isActive);
  }
  
  build(): UserConfig {
    if (!this.data.name || !this.data.email) {
      throw new Error("Name and email are required");
    }
    
    return {
      name: this.data.name,
      email: this.data.email,
      age: this.data.age ?? 0,
      isActive: this.data.isActive ?? false
    } as UserConfig;
  }
}

// Использование
const user = new UserBuilder()
  .setName("Alice")
  .setEmail("alice@example.com")
  .setAge(30)
  .setIsActive(true)
  .build();

console.log(user); // { name: "Alice", email: "alice@example.com", age: 30, isActive: true }
```

### Director для Builder
```typescript
// Директор управляет процессом строительства
interface Meal {
  drink: string;
  food: string;
  dessert: string;
}

class MealBuilder {
  private meal: Meal = { drink: "", food: "", dessert: "" };
  
  setDrink(drink: string): this { this.meal.drink = drink; return this; }
  setFood(food: string): this { this.meal.food = food; return this; }
  setDessert(dessert: string): this { this.meal.dessert = dessert; return this; }
  
  build(): Meal { return { ...this.meal }; }
}

class MealDirector {
  // Предопределенные комбинации
  buildBreakfast(): Meal {
    return new MealBuilder()
      .setDrink("Coffee")
      .setFood("Toast")
      .setDessert("Fruit")
      .build();
  }
  
  buildLunch(): Meal {
    return new MealBuilder()
      .setDrink("Juice")
      .setFood("Sandwich")
      .setDessert("Cookie")
      .build();
  }
  
  buildDinner(): Meal {
    return new MealBuilder()
      .setDrink("Wine")
      .setFood("Steak")
      .setDessert("Ice Cream")
      .build();
  }
}

// Использование
const director = new MealDirector();
const breakfast = director.buildBreakfast();
const lunch = director.buildLunch();
const dinner = director.buildDinner();
```

## Singleton Pattern

### Базовый синглтон
```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection;
  private isConnected = false;
  
  private constructor() {
    // Приватный конструктор предотвращает прямое создание
  }
  
  static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
  
  connect(): void {
    if (!this.isConnected) {
      console.log("Connecting to database...");
      this.isConnected = true;
    }
  }
  
  disconnect(): void {
    if (this.isConnected) {
      console.log("Disconnecting from database...");
      this.isConnected = false;
    }
  }
  
  query(sql: string): any[] {
    if (!this.isConnected) {
      throw new Error("Database not connected");
    }
    console.log(`Executing query: ${sql}`);
    return [];
  }
}

// Использование
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();

console.log(db1 === db2); // true - один и тот же экземпляр
db1.connect();
db2.query("SELECT * FROM users"); // Работает, т.к. соединение открыто
```

### Синглтон с обобщениями
```typescript
// Обобщенный синглтон
abstract class Singleton<T> {
  private static instances: Map<Function, any> = new Map();
  
  static getInstance<T>(this: new () => T): T {
    if (!Singleton.instances.has(this)) {
      Singleton.instances.set(this, new this());
    }
    return Singleton.instances.get(this);
  }
}

class Logger extends Singleton<Logger> {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

class ConfigManager extends Singleton<ConfigManager> {
  private config: Record<string, any> = {};
  
  set(key: string, value: any): void {
    this.config[key] = value;
  }
  
  get(key: string): any {
    return this.config[key];
  }
}

// Использование
const logger1 = Logger.getInstance();
const logger2 = Logger.getInstance();
console.log(logger1 === logger2); // true

const config1 = ConfigManager.getInstance();
const config2 = ConfigManager.getInstance();
console.log(config1 === config2); // true
console.log(logger1 === config1); // false - разные типы
```

### Синглтон с асинхронной инициализацией
```typescript
class AsyncSingleton {
  private static instance: AsyncSingleton;
  private static initializing: boolean = false;
  private static initialized: boolean = false;
  private data: any;
  
  private constructor() {}
  
  static async getInstance(): Promise<AsyncSingleton> {
    if (AsyncSingleton.instance) {
      return AsyncSingleton.instance;
    }
    
    if (AsyncSingleton.initializing) {
      // Ждем завершения инициализации
      while (!AsyncSingleton.initialized) {
        await new Promise(resolve => setTimeout(resolve, 10));
      }
      return AsyncSingleton.instance!;
    }
    
    AsyncSingleton.initializing = true;
    AsyncSingleton.instance = new AsyncSingleton();
    await AsyncSingleton.instance.initialize();
    AsyncSingleton.initialized = true;
    AsyncSingleton.initializing = false;
    
    return AsyncSingleton.instance;
  }
  
  private async initialize(): Promise<void> {
    // Симуляция асинхронной инициализации
    await new Promise(resolve => setTimeout(resolve, 100));
    this.data = { initialized: true, timestamp: Date.now() };
  }
  
  getData(): any {
    return this.data;
  }
}

// Использование
async function example() {
  const [inst1, inst2] = await Promise.all([
    AsyncSingleton.getInstance(),
    AsyncSingleton.getInstance()
  ]);
  
  console.log(inst1 === inst2); // true - всегда один экземпляр
  console.log(inst1.getData()); // { initialized: true, timestamp: ... }
}
```

## Prototype Pattern

### Классический прототип
```typescript
interface Prototype<T> {
  clone(): T;
}

class ConcretePrototype implements Prototype<ConcretePrototype> {
  constructor(
    public name: string,
    public data: any[],
    public config: Record<string, any>
  ) {}
  
  clone(): ConcretePrototype {
    // Поверхностное копирование
    return new ConcretePrototype(
      this.name,
      [...this.data], // копируем массив
      { ...this.config } // копируем объект
    );
  }
  
  deepClone(): ConcretePrototype {
    // Глубокое копирование
    return new ConcretePrototype(
      this.name,
      JSON.parse(JSON.stringify(this.data)),
      JSON.parse(JSON.stringify(this.config))
    );
  }
}

// Использование
const original = new ConcretePrototype("Original", [1, 2, 3], { debug: true });
const clone = original.clone();

console.log(original === clone); // false - разные экземпляры
console.log(original.data === clone.data); // false - разные массивы
console.log(original.config === clone.config); // false - разные объекты

clone.name = "Clone";
clone.data.push(4);

console.log(original.data); // [1, 2, 3] - неизменен
console.log(clone.data); // [1, 2, 3, 4]
```

### Прототип с обобщениями
```typescript
interface ClonableObject<T> {
  clone(): T;
  deepClone?(): T;
}

class Configuration implements ClonableObject<Configuration> {
  constructor(
    public name: string,
    public settings: Map<string, any>,
    public features: Set<string>
  ) {}
  
  clone(): Configuration {
    return new Configuration(
      this.name,
      new Map(this.settings), // клонируем Map
      new Set(this.features)  // клонируем Set
    );
  }
  
  deepClone(): Configuration {
    // Для более сложных объектов
    return new Configuration(
      this.name,
      this.deepCloneMap(this.settings),
      new Set(this.features) // Set уже содержит примитивы
    );
  }
  
  private deepCloneMap(map: Map<string, any>): Map<string, any> {
    const clonedMap = new Map<string, any>();
    map.forEach((value, key) => {
      clonedMap.set(key, this.deepCloneValue(value));
    });
    return clonedMap;
  }
  
  private deepCloneValue(value: any): any {
    if (value === null || typeof value !== 'object') {
      return value;
    }
    
    if (value instanceof Date) {
      return new Date(value.getTime());
    }
    
    if (Array.isArray(value)) {
      return value.map(item => this.deepCloneValue(item));
    }
    
    if (value instanceof Map) {
      return this.deepCloneMap(value);
    }
    
    if (value instanceof Set) {
      return new Set(Array.from(value).map(item => this.deepCloneValue(item)));
    }
    
    // Обычный объект
    const clonedObj: any = {};
    for (const key in value) {
      if (value.hasOwnProperty(key)) {
        clonedObj[key] = this.deepCloneValue(value[key]);
      }
    }
    return clonedObj;
  }
}

// Использование
const baseConfig = new Configuration(
  "default", 
  new Map([["theme", "light"], ["lang", "en"]]), 
  new Set(["feature1", "feature2"])
);

const customConfig = baseConfig.clone();
customConfig.name = "custom";
customConfig.settings.set("theme", "dark");
customConfig.features.add("feature3");

console.log(baseConfig.name); // "default"
console.log(baseConfig.settings.get("theme")); // "light"
console.log(baseConfig.features.has("feature3")); // false

console.log(customConfig.name); // "custom"  
console.log(customConfig.settings.get("theme")); // "dark"
console.log(customConfig.features.has("feature3")); // true
```

### Registry паттерн
```typescript
// Реестр прототипов
class PrototypeRegistry<T extends ClonableObject<T>> {
  private prototypes: Map<string, T> = new Map();
  
  register(key: string, prototype: T): void {
    this.prototypes.set(key, prototype);
  }
  
  unregister(key: string): void {
    this.prototypes.delete(key);
  }
  
  clone(key: string): T | undefined {
    const prototype = this.prototypes.get(key);
    return prototype ? prototype.clone() : undefined;
  }
  
  create(key: string, customizer?: (obj: T) => void): T | undefined {
    const cloned = this.clone(key);
    if (cloned && customizer) {
      customizer(cloned);
    }
    return cloned;
  }
  
  getAllKeys(): string[] {
    return Array.from(this.prototypes.keys());
  }
}

// Использование реестра
const registry = new PrototypeRegistry<Configuration>();

// Регистрируем базовые конфигурации
registry.register("web-default", new Configuration(
  "web", 
  new Map([["port", 3000], ["ssl", false]]), 
  new Set(["auth", "cors"])
));

registry.register("mobile-default", new Configuration(
  "mobile",
  new Map([["api-version", "v1"], ["cache-enabled", true]]), 
  new Set(["offline", "push-notifications"])
));

// Создание новых конфигов на основе прототипов
const webConfig = registry.create("web-default", (config) => {
  config.settings.set("port", 8080);
  config.settings.set("ssl", true);
});

const mobileConfig = registry.clone("mobile-default");
if (mobileConfig) {
  mobileConfig.name = "mobile-prod";
  mobileConfig.settings.set("api-version", "v2");
}

console.log(webConfig?.settings.get("port")); // 8080
console.log(mobileConfig?.name); // "mobile-prod"
console.log(registry.getAllKeys()); // ["web-default", "mobile-default"]
```

## Функциональные порождающие паттерны

### Фабричные функции
```typescript
// Функциональная альтернатива класс-базированным фабрикам
interface User {
  id: number;
  name: string;
  email: string;
  role: string;
}

interface UserFactoryOptions {
  defaultRole: string;
  requireEmail: boolean;
}

// Фабричная функция
function createUserFactory(options: UserFactoryOptions) {
  return function(name: string, email?: string): User {
    if (options.requireEmail && !email) {
      throw new Error("Email is required");
    }
    
    return {
      id: Math.floor(Math.random() * 1000000),
      name,
      email: email || "no-email@example.com",
      role: options.defaultRole
    };
  };
}

// Создание конкретных фабрик
const adminUserFactory = createUserFactory({ 
  defaultRole: "admin", 
  requireEmail: true 
});

const guestUserFactory = createUserFactory({ 
  defaultRole: "guest", 
  requireEmail: false 
});

// Использование
const admin = adminUserFactory("Alice", "alice@example.com");
const guest = guestUserFactory("Bob");
```

### Builder как цепочка функций
```typescript
// Функциональный Builder
interface UserProfile {
  name: string;
  age: number;
  email: string;
  preferences: {
    theme: string;
    notifications: boolean;
  };
}

function createUserProfile(): UserProfile {
  return {
    name: "",
    age: 0,
    email: "",
    preferences: {
      theme: "light",
      notifications: true
    }
  };
}

function setName(profile: UserProfile, name: string): UserProfile {
  return { ...profile, name };
}

function setAge(profile: UserProfile, age: number): UserProfile {
  return { ...profile, age };
}

function setEmail(profile: UserProfile, email: string): UserProfile {
  return { ...profile, email };
}

function setTheme(profile: UserProfile, theme: string): UserProfile {
  return {
    ...profile,
    preferences: { ...profile.preferences, theme }
  };
}

function setNotifications(profile: UserProfile, enabled: boolean): UserProfile {
  return {
    ...profile,
    preferences: { ...profile.preferences, notifications: enabled }
  };
}

// Композиция
const buildProfile = (
  name: string,
  age: number,
  email: string,
  theme: string,
  notifications: boolean
): UserProfile => 
  [setName, setAge, setEmail, setTheme, setNotifications]
    .reduce((profile, fn) => fn(profile, ...argumentsFor(fn)), 
            createUserProfile());

// Более элегантная цепочка
const pipe = <T>(value: T, ...fns: Array<(arg: T) => T>): T =>
  fns.reduce((acc, fn) => fn(acc), value);

function createUserBuilder() {
  const profile = createUserProfile();
  
  return {
    setName: (name: string) => ({
      ...this,
      build: () => setName(profile, name)
    }),
    setAge: (age: number) => ({
      ...this,
      build: () => setAge(profile, age)
    }),
    setEmail: (email: string) => ({
      ...this,
      build: () => setEmail(profile, email)
    }),
    setTheme: (theme: string) => ({
      ...this,
      build: () => setTheme(profile, theme)
    }),
    setNotifications: (enabled: boolean) => ({
      ...this,
      build: () => setNotifications(profile, enabled)
    })
  };
}
```

## Практические примеры

### Dependency Injection Container
```typescript
interface Service {
  name: string;
}

class DatabaseService implements Service {
  name = "DatabaseService";
  connect() { console.log("Connected to database"); }
}

class EmailService implements Service {
  name = "EmailService";
  send() { console.log("Sending email"); }
}

// Стратегия инъекции
type InjectionStrategy = 'singleton' | 'transient' | 'scoped';

interface ServiceRegistration {
  token: string;
  useClass?: new () => Service;
  useFactory?: (...deps: Service[]) => Service;
  useValue?: Service;
  strategy: InjectionStrategy;
  dependencies?: string[];
}

class DIContainer {
  private services = new Map<string, Service>();
  private registrations = new Map<string, ServiceRegistration>();
  
  register(registration: ServiceRegistration): void {
    this.registrations.set(registration.token, registration);
  }
  
  resolve<T extends Service>(token: string): T {
    const registration = this.registrations.get(token);
    if (!registration) {
      throw new Error(`Service not registered: ${token}`);
    }
    
    // Одиночки кэшируем, остальные создаем каждый раз
    if (registration.strategy === 'singleton' && this.services.has(token)) {
      return this.services.get(token) as T;
    }
    
    let service: Service;
    
    if (registration.useValue) {
      service = registration.useValue;
    } else if (registration.useFactory) {
      const dependencies = registration.dependencies?.map(dep => this.resolve(dep)) || [];
      service = registration.useFactory(...dependencies);
    } else if (registration.useClass) {
      const dependencies = registration.dependencies?.map(dep => this.resolve(dep)) || [];
      service = new registration.useClass(...dependencies);
    } else {
      throw new Error(`Invalid registration for token: ${token}`);
    }
    
    if (registration.strategy === 'singleton') {
      this.services.set(token, service);
    }
    
    return service as T;
  }
}

// Использование
const container = new DIContainer();

container.register({
  token: 'DatabaseService',
  useClass: DatabaseService,
  strategy: 'singleton'
});

container.register({
  token: 'EmailService', 
  useClass: EmailService,
  strategy: 'transient'
});

container.register({
  token: 'UserService',
  useFactory: (db: DatabaseService, email: EmailService) => {
    class UserService implements Service {
      name = "UserService";
      constructor(private database: DatabaseService, private email: EmailService) {}
    }
    return new UserService(db, email);
  },
  strategy: 'singleton',
  dependencies: ['DatabaseService', 'EmailService']
});

const userService = container.resolve<UserService>('UserService');
```

## Лучшие практики

### 1. Используйте обобщения для переиспользуемых фабрик
```typescript
// Хорошо: обобщенная фабрика
interface Identifiable {
  id: string;
}

class GenericRepository<T extends Identifiable> {
  private items: Map<string, T> = new Map();
  
  create(factory: () => T): T {
    const item = factory();
    this.items.set(item.id, item);
    return item;
  }
  
  findById(id: string): T | undefined {
    return this.items.get(id);
  }
}
```

### 2. Обеспечивайте типобезопасность
```typescript
// Хорошо: типобезопасный builder
interface ValidationResult<T> {
  success: boolean;
  data?: T;
  errors?: string[];
}

class ValidatedBuilder<T> {
  constructor(private validator: (data: T) => ValidationResult<T>) {}
  
  build(rawData: any): ValidationResult<T> {
    const result: T = rawData as T;
    return this.validator(result);
  }
}
```

### 3. Используйте строительную цепочку для сложных объектов
```typescript
interface ComplexConfig {
  server: {
    host: string;
    port: number;
    ssl: { enabled: boolean; cert?: string; key?: string };
  };
  database: {
    url: string;
    poolSize: number;
  };
  features: {
    [key: string]: boolean;
  };
}

// Builder для сложной конфигурации
class ComplexConfigBuilder {
  private config: Partial<ComplexConfig> = {};
  
  setServer(host: string, port: number): this {
    this.config.server = { host, port, ssl: { enabled: false } };
    return this;
  }
  
  enableSSL(cert?: string, key?: string): this {
    if (this.config.server) {
      this.config.server.ssl = { enabled: true, cert, key };
    }
    return this;
  }
  
  setDatabase(url: string, poolSize: number): this {
    this.config.database = { url, poolSize };
    return this;
  }
  
  addFeature(name: string, enabled: boolean): this {
    if (!this.config.features) {
      this.config.features = {};
    }
    this.config.features[name] = enabled;
    return this;
  }
  
  build(): ComplexConfig {
    // Валидация перед возвратом
    if (!this.config.server || !this.config.database) {
      throw new Error("Server and database configuration required");
    }
    
    return this.config as ComplexConfig;
  }
}
```

## Ограничения и подводные камни

### 1. Singleton и тестируемость
```typescript
// ПЛОХО: трудно тестировать из-за глобального состояния
class BadService {
  private static instance: BadService;
  
  static getInstance() {
    if (!BadService.instance) {
      BadService.instance = new BadService(new RealAPI());
    }
    return BadService.instance;
  }
  
  fetchData() {
    // hard to mock RealAPI
  }
}

// ХОРОШО: инжекция зависимостей
class GoodService {
  constructor(private api: APIInterface) {}
  
  static createForTest(testApi: APIInterface) {
    return new GoodService(testApi);
  }
  
  fetchData() {
    return this.api.fetchData();
  }
}
```

### 2. Сложные Builder'ы
```typescript
// Избегайте чрезмерной сложности в Builder'ах
// Слишком много условной логики может привести к багам
class OverengineeredBuilder {
  // Не стоит делать слишком сложную внутреннюю логику
  private validate(): void {
    // Лучше вынести валидацию отдельно
  }
  
  private normalize(): void {
    // Лучше не иметь сложной нормализации
  }
}
```

## Современные тенденции

### Builder с Union Types
```typescript
// Использование Union Types для безопасного построения
type Status = 'draft' | 'published' | 'archived';

interface DraftPost {
  status: 'draft';
  title: string;
  content: string;
}

interface PublishedPost {
  status: 'published';
  title: string;
  content: string;
  publishedAt: Date;
}

interface ArchivedPost {
  status: 'archived';
  title: string;
  content: string;
  publishedAt: Date;
  archivedAt: Date;
}

type Post = DraftPost | PublishedPost | ArchivedPost;

class PostBuilder {
  private data: Partial<Post> = { status: 'draft' };
  
  setTitle(title: string): this {
    this.data.title = title;
    return this;
  }
  
  setContent(content: string): this {
    this.data.content = content;
    return this;
  }
  
  publish(): PostBuilder & { build(): PublishedPost } {
    this.data.status = 'published';
    (this.data as PublishedPost).publishedAt = new Date();
    return this as any; // unsafe cast для демонстрации паттерна
  }
  
  archive(): PostBuilder & { build(): ArchivedPost } {
    if (this.data.status !== 'published') {
      throw new Error('Cannot archive non-published post');
    }
    this.data.status = 'archived';
    (this.data as ArchivedPost).archivedAt = new Date();
    return this as any;
  }
  
  build(): Post {
    if (!this.data.title || !this.data.content) {
      throw new Error('Title and content required');
    }
    
    switch (this.data.status) {
      case 'draft':
        return {
          status: 'draft',
          title: this.data.title!,
          content: this.data.content!
        } as DraftPost;
      case 'published':
        return {
          status: 'published',
          title: this.data.title!,
          content: this.data.content!,
          publishedAt: (this.data as PublishedPost).publishedAt!
        } as PublishedPost;
      case 'archived':
        return {
          status: 'archived',
          title: this.data.title!,
          content: this.data.content!,
          publishedAt: (this.data as ArchivedPost).publishedAt!,
          archivedAt: (this.data as ArchivedPost).archivedAt!
        } as ArchivedPost;
      default:
        throw new Error('Invalid status');
    }
  }
}
```

## Заключение

Порождающие паттерны в TypeScript обеспечивают гибкие и типобезопасные способы создания объектов. Ключевые преимущества:

1. **Изолированность создания объектов** - инкапсуляция логики создания
2. **Типобезопасность** - гарантии компилятора при создании
3. **Гибкость** - возможность создания разных вариантов объектов
4. **Тестируемость** - возможность мокирования и инъекции зависимостей
5. **Поддерживаемость** - централизация логики создания объектов

Каждый паттерн имеет свои сценарии использования и должен выбираться в зависимости от конкретных требований архитектуры приложения.

## Связь с другими концепциями
- [[../generics/index]] - обобщенные паттерны создания
- [[TS Interfaces and Classes]] - объектно-ориентированные паттерны
- [[Advanced TypeScript Types]] - продвинутые типы для паттернов
- [[../functional-programming/index]] - функциональные паттерны создания
- [[TypeScript Modules]] - организация паттернов в модули
# TypeScript Classes

Классы в TypeScript расширяют возможности ES2015 классов, добавляя статическую типизацию. Они предоставляют способ создания объектов с определенными свойствами и методами, а также поддерживают наследование, инкапсуляцию и полиморфизм.

## Базовый синтаксис класса

### Простой класс
```typescript
class Greeter {
  greeting: string;

  constructor(message: string) {
    this.greeting = message;
  }

  greet(): string {
    return "Hello, " + this.greeting;
  }
}

let greeter = new Greeter("world");
console.log(greeter.greet()); // "Hello, world"
```

### Параметры конструктора
```typescript
class Octopus {
  readonly name: string;
  readonly numberOfLegs: number = 8;

  constructor(name: string) {
    this.name = name;
  }
}
```

## Модификаторы доступа

### public, private, protected
```typescript
class Animal {
  private name: string;
  protected age: number;
  public species: string;

  constructor(name: string, age: number, species: string) {
    this.name = name;
    this.age = age;
    this.species = species;
  }

  // Методы по умолчанию public
  move(distance: number = 0) {
    console.log(`${this.name} moved ${distance}m.`);
  }
}

class Dog extends Animal {
  constructor(name: string, age: number) {
    super(name, age, "Canine");
  }

  // Может получить доступ к protected age
  getAge(): number {
    return this.age;
  }

  // Не может получить доступ к private name
  // getName() { return this.name; } // Ошибка
}
```

### Сокращённый синтаксис параметров
```typescript
class Person {
  constructor(
    private name: string,
    protected age: number,
    public email: string
  ) {}

  getDetails(): string {
    return `${this.name}, ${this.age}, ${this.email}`;
  }
}

const person = new Person("Alice", 30, "alice@example.com");
// person.name недоступен извне (private)
// person.age недоступен извне (protected)  
// person.email доступен извне (public)
```

## Статические свойства и методы

### Статические члены класса
```typescript
class Grid {
  static origin = { x: 0, y: 0 };

  constructor(public scale: number) {}

  calculateDistanceFromOrigin(point: { x: number; y: number }) {
    let xDist = point.x - Grid.origin.x;
    let yDist = point.y - Grid.origin.y;
    return Math.sqrt(xDist * xDist + yDist * yDist) * this.scale;
  }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({ x: 1, y: 1 })); // 1.4142...
console.log(grid2.calculateDistanceFromOrigin({ x: 1, y: 1 })); // 7.0710...
```

## Абстрактные классы

### Абстрактные классы и методы
```typescript
abstract class Department {
  constructor(public name: string) {}

  printName(): void {
    console.log("Department name: " + this.name);
  }

  abstract printMeeting(): void; // Должен быть реализован в производных классах
}

class AccountingDepartment extends Department {
  constructor() {
    super("Accounting and Auditing"); // Конструкторы в производных классах должны вызывать super()
  }

  printMeeting(): void {
    console.log("The Accounting Department meets each Monday at 10am.");
  }

  generateReports(): void {
    console.log("Generating accounting reports...");
  }
}

// let dept = new Department(); // Ошибка: не может создать экземпляр абстрактного класса
let dept = new AccountingDepartment(); // OK
dept.printName();
dept.printMeeting();
dept.generateReports();
```

## Наследование

### Классы-наследники
```typescript
class Animal {
  name: string;
  constructor(name: string) { 
    this.name = name; 
  }
  
  move(distance: number = 0) {
    console.log(`${this.name} moved ${distance}m.`);
  }
}

class Snake extends Animal {
  constructor(name: string) {
    super(name); // Вызов конструктора родительского класса
  }
  
  move(distance: number = 5) {
    console.log("Slithering...");
    super.move(distance); // Вызов метода родительского класса
  }
}

class Horse extends Animal {
  constructor(name: string) { 
    super(name); 
  }
  
  move(distance: number = 45) {
    console.log("Galloping...");
    super.move(distance);
  }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move(); // "Slithering... Sammy the Python moved 5m."
tom.move(34); // "Galloping... Tommy the Palomino moved 34m."
```

## Геттеры и сеттеры

### Использование геттеров и сеттеров
```typescript
class Employee {
  private _fullName: string = "";

  get fullName(): string {
    return this._fullName;
  }

  set fullName(newName: string) {
    if (newName && newName.length > 10) {
      throw new Error("Имя слишком длинное");
    }
    
    this._fullName = newName;
  }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
  console.log(employee.fullName);
}
```

## Ссылки на классы и конструкторы

### Использование интерфейсов для конструкторов
```typescript
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;
}

interface ClockInterface {
  tick(): void;
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
  return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
    console.log("beep beep");
  }
}

class AnalogClock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
    console.log("tick tock");
  }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

## Обобщенные классы

### Классы с обобщениями
```typescript
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };

let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };
```

## Практические примеры

### Менеджер состояния
```typescript
class StateManager<T> {
  private state: T;
  private listeners: Array<(state: T) => void> = [];

  constructor(initialState: T) {
    this.state = initialState;
  }

  getState(): T {
    return this.state;
  }

  setState(newState: T | ((prevState: T) => T)): void {
    const newStateValue = typeof newState === 'function' ? newState(this.state) : newState;
    this.state = newStateValue;
    this.notifyListeners();
  }

  subscribe(listener: (state: T) => void): () => void {
    this.listeners.push(listener);
    
    // Возвращаем функцию отписки
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }

  private notifyListeners(): void {
    this.listeners.forEach(listener => listener(this.state));
  }
}

// Использование
interface AppState {
  user: { name: string; id: number } | null;
  theme: 'light' | 'dark';
}

const stateManager = new StateManager<AppState>({
  user: null,
  theme: 'light'
});

stateManager.subscribe(state => {
  console.log('State updated:', state);
});
```

### Репозиторий с обобщениями
```typescript
abstract class Repository<T, ID> {
  protected items: Map<ID, T> = new Map();

  abstract findById(id: ID): T | undefined;
  abstract save(entity: T, id: ID): void;
  abstract delete(id: ID): void;
  abstract findAll(): T[];
}

class User {
  constructor(public id: number, public name: string, public email: string) {}
}

class UserRepository extends Repository<User, number> {
  findById(id: number): User | undefined {
    return this.items.get(id);
  }

  save(entity: User, id: number): void {
    this.items.set(id, entity);
  }

  delete(id: number): void {
    this.items.delete(id);
  }

  findAll(): User[] {
    return Array.from(this.items.values());
  }
}

// Использование
const userRepository = new UserRepository();
const user = new User(1, "Alice", "alice@example.com");
userRepository.save(user, user.id);
const foundUser = userRepository.findById(1);
```

## Продвинутые паттерны

### Фабрика классов
```typescript
interface Animal {
  name: string;
  makeSound(): void;
}

class Dog implements Animal {
  constructor(public name: string) {}
  makeSound() { console.log("Woof!"); }
}

class Cat implements Animal {
  constructor(public name: string) {}
  makeSound() { console.log("Meow!"); }
}

type AnimalClass<T extends Animal = Animal> = new (name: string) => T;

class AnimalFactory {
  static create<T extends Animal>(AnimalClass: AnimalClass<T>, name: string): T {
    return new AnimalClass(name);
  }
}

const dog = AnimalFactory.create(Dog, "Buddy");
const cat = AnimalFactory.create(Cat, "Whiskers");
```

### Mixin паттерн
```typescript
// Базовый класс
class Component {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
}

// Тип для функции-миксина
type Constructor<T = {}> = new (...args: any[]) => T;

// Миксин для логирования
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

// Миксин для сложных объектов
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    
    activate() {
      this.isActive = true;
    }
    
    deactivate() {
      this.isActive = false;
    }
  };
}

// Создание классов с миксинами
const TimestampedComponent = Timestamped(Component);
const FullComponent = Activatable(TimestampedComponent);

const myComponent = new FullComponent("test");
console.log(myComponent.name);        // "test"
console.log(myComponent.timestamp);   // 1234567890123
console.log(myComponent.isActive);    // false
myComponent.activate();
console.log(myComponent.isActive);    // true
```

## Лучшие практики

1. **Используйте модификаторы доступа** - ограничивайте доступ к внутренним деталям
2. **Предпочитайте интерфейсы абстрактным классам** - когда не нужна реализация
3. **Используйте readonly для неизменяемых свойств** - особенно в конструкторе
4. **Делайте методы и свойства как можно более закрытыми** - начинайте с private
5. **Используйте абстрактные классы для определения контрактов с частичной реализацией**
6. **Ограничивайте наследование** - часто композиция лучше наследования

## Связь с другими концепциями
- [[../interfaces-classes/interfaces]] - как классы могут реализовывать интерфейсы
- [[TS Generics]] - обобщенные классы
- [[../type-system/type-compatibility]] - совместимость классов
- [[../advanced-types/discriminated-unions]] - использование классов как дискриминантов
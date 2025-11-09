# Duck Typing

Duck Typing (утиная типизация) - это концепция, согласно которой "если это выглядит как утка, плавает как утка и крякает как утка, то это, вероятно, утка". В контексте TypeScript, это означает, что объекты рассматриваются как совместимые, если они имеют одинаковую структуру (свойства и методы), независимо от их статического типа или наследования.

## Понимание Duck Typing

### Основная концепция
Duck Typing в TypeScript является основой структурной типизации. В отличие от номинальной типизации (где типы сравниваются по их именам), TypeScript использует структурную типизацию, где два типа считаются совместимыми, если они имеют одинаковую структуру.

```typescript
// Пример базовой утиной типизации
interface Duck {
  name: string;
  swim(): void;
  quack(): void;
}

interface Dog {
  name: string;
  swim(): void;
  bark(): void;
}

// Функция ожидает Duck, но accepts любой объект с такой же структурой
function makeDuckSwim(duck: Duck) {
  duck.swim();
  duck.quack();
}

// Утки
const realDuck = {
  name: "Donald",
  swim() { console.log("Swimming like a duck"); },
  quack() { console.log("Quacking like a duck"); }
};

// Собака, но с методами утки
const dogLikeDuck = {
  name: "Buddy",
  swim() { console.log("Swimming like a dog"); },
  quack() { console.log("Barking like a duck"); } // имитирует quack
};

// Оба работают!
makeDuckSwim(realDuck);     // OK
makeDuckSwim(dogLikeDuck);  // OK - несмотря на то, что это Dog-подобный объект
```

## Структурная типизация в TypeScript

### Совместимость объектов
```typescript
interface Named {
  name: string;
}

class Person {
  name: string;
  age: number;
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

class Cat {
  name: string;
  breed: string;
  
  constructor(name: string, breed: string) {
    this.name = name;
    this.breed = breed;
  }
}

let x: Named;

// Все эти присваивания разрешены, потому что все классы имеют свойство name: string
x = new Person("Alice", 30);  // OK
x = new Cat("Fluffy", "Persian");  // OK

// И даже простой объект
x = { name: "Anonymous" };  // OK
```

### Совместимость функций
```typescript
// Сигнатуры функций также подчиняются утиной типизации
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

// В отношениях типов TypeScript рассматривает, что y может быть присвоена x
// потому что x игнорирует дополнительные параметры (вариантность параметров)
x = y; // OK - x может принять y, несмотря на дополнительный параметр

// Но не наоборот
// y = x; // ОШИБКА - y требует больше параметров

// Совместимость возвращаемых значений (ковариантность)
let f = () => ({ name: "Alice" });
let g = () => ({ name: "Alice", age: 30 });

// g может быть присвоена f, так как { name: string, age: number } 
// может быть использован везде, где ожидается { name: string }
f = g; // OK

// f НЕ может быть присвоена g, так как { name: string } 
// не содержит всех свойств { name: string, age: number }
// g = f; // ОШИБКА
```

## Примеры утиной типизации

### 1. Совместимость интерфейсов
```typescript
interface Point2D {
  x: number;
  y: number;
}

interface Point3D {
  x: number;
  y: number;
  z: number;
}

// point3D совместим с Point2D, так как имеет все необходимые свойства
const point2D: Point2D = { x: 1, y: 2 };
const point3D: Point3D = { x: 1, y: 2, z: 3 };

// Point3D может быть использован как Point2D
const another2D: Point2D = point3D; // OK - 3D точка может использоваться как 2D

// Но не наоборот
// const another3D: Point3D = point2D; // ОШИБКА - у 2D точки нет z

// Использование в функциях
function log2DCoordinates(point: Point2D) {
  console.log(`x: ${point.x}, y: ${point.y}`);
}

log2DCoordinates(point3D); // OK - 3D точка подходит для 2D
```

### 2. Совместимость классов
```typescript
class Animal {
  name: string;
  constructor(name: string) { this.name = name; }
  move() { console.log(`${this.name} moving`); }
}

class Bird extends Animal {
  fly() { console.log(`${this.name} flying`); }
}

class Fish extends Animal {
  swim() { console.log(`${this.name} swimming`); }
}

// Использование с базовым классом
function makeAnimalMove(animal: Animal) {
  animal.move();
}

const bird = new Bird("Tweety");
const fish = new Fish("Nemo");

makeAnimalMove(bird); // OK - Bird наследует Animal
makeAnimalMove(fish); // OK - Fish наследует Animal

// Но также работает и с объектами, которые "выглядят как Animal"
const animalLike = {
  name: "Mystery creature",
  move() { console.log("Moving mysteriously"); }
};

makeAnimalMove(animalLike); // OK - несмотря на отсутствие наследования
```

### 3. Совместимость функций-членов
```typescript
interface Greetable {
  name: string;
  greet(): string;
}

class Person implements Greetable {
  constructor(public name: string) {}
  greet() { return `Hello, I'm ${this.name}`; }
}

class Robot {
  constructor(public name: string) {}
  greet() { return `Beep boop, designation ${this.name}`; }
  repair() { console.log("Repairing..."); }
}

// Robot не реализует Greetable, но имеет ту же структуру
const person: Greetable = new Person("Alice"); // OK
const robot: Greetable = new Robot("R2D2"); // OK - структурно совместим

function welcome(greeter: Greetable) {
  console.log(greeter.greet());
}

welcome(person); // "Hello, I'm Alice"
welcome(robot); // "Beep boop, designation R2D2"
```

## Продвинутая утиная типизация

### 1. Совместимость с индексными сигнатурами
```typescript
interface Dictionary {
  [key: string]: number;
}

interface NumericObject {
  x: number;
  y: number;
}

// NumericObject может быть использован как Dictionary
const numericObj: NumericObject = { x: 1, y: 2 };
const dict: Dictionary = numericObj; // OK

// Но Dictionary не может быть использован как NumericObject
const dictionary: Dictionary = { a: 1, b: 2, x: 10, y: 20 };
const coord: NumericObject = dictionary; // OK - если содержит x и y

// Проверка на подмножество
const incompleteDict: Dictionary = { x: 1 }; // OK
// const incompleteCoord: NumericObject = incompleteDict; // ОШИБКА - нет y
```

### 2. Совместимость с необязательными свойствами
```typescript
interface Config {
  host: string;
  port: number;
  ssl?: boolean; // необязательное свойство
}

// Объект без ssl также совместим
const basicConfig: Config = {
  host: "localhost",
  port: 3000
  // ssl отсутствует, но OK
};

// Объект с ssl также совместим
const fullConfig: Config = {
  host: "example.com",
  port: 443,
  ssl: true
};

// И даже объект с дополнительными свойствами
const extendedConfig: Config = {
  host: "prod.example.com",
  port: 443,
  ssl: true,
  timeout: 5000, // дополнительное свойство
  retries: 3      // дополнительное свойство
};
```

## Практические примеры

### 1. Работа с внешними библиотеками
```typescript
// Представим, что внешняя библиотека ожидает определенный интерфейс
interface LibraryUser {
  id: number;
  name: string;
  email: string;
  authenticate(): boolean;
}

// А у нас есть наш собственный класс
class AppUser {
  constructor(
    public id: number,
    public name: string, 
    public email: string,
    private token: string
  ) {}
  
  authenticate(): boolean {
    // логика аутентификации
    return this.token.length > 0;
  }
  
  refreshToken() {
    // обновление токена
  }
}

// Наш AppUser структурно совместим с LibraryUser
const appUser = new AppUser(1, "Alice", "alice@example.com", "token123");
const libUser: LibraryUser = appUser; // OK

// Мы можем передать наш AppUser в библиотечную функцию
function useLibrary(user: LibraryUser) {
  if (user.authenticate()) {
    console.log(`Welcome ${user.name}`);
  }
}

useLibrary(appUser); // OK - несмотря на отсутствие явного наследования
```

### 2. Мокирование для тестирования
```typescript
// Оригинальный сервис
interface PaymentService {
  charge(amount: number, currency: string): Promise<boolean>;
  refund(transactionId: string): Promise<boolean>;
}

// Реализация
class ActualPaymentService implements PaymentService {
  async charge(amount: number, currency: string): Promise<boolean> {
    // реальная логика оплаты
    return true;
  }
  
  async refund(transactionId: string): Promise<boolean> {
    // реальная логика возврата
    return true;
  }
}

// Мок для тестирования
class MockPaymentService {
  charges: { amount: number; currency: string }[] = [];
  
  async charge(amount: number, currency: string): Promise<boolean> {
    this.charges.push({ amount, currency });
    return true;
  }
  
  async refund(transactionId: string): Promise<boolean> {
    return true;
  }
}

// В тестах можем использовать мок вместо реального сервиса
function testPaymentFlow(paymentService: PaymentService) {
  // тестовая логика
  return paymentService.charge(100, "USD");
}

const mockService = new MockPaymentService();
testPaymentFlow(mockService); // OK - структурно совместимо
```

### 3. Плагины и расширения
```typescript
// Интерфейс для плагина
interface Plugin {
  name: string;
  version: string;
  init(): void;
  destroy(): void;
}

// Разные реализации плагинов
class LoggerPlugin implements Plugin {
  name = "Logger";
  version = "1.0.0";
  
  init() { console.log("Logger initialized"); }
  destroy() { console.log("Logger destroyed"); }
  log(message: string) { console.log(message); }
}

class AnalyticsPlugin {
  name = "Analytics";
  version = "2.0.0";
  
  init() { console.log("Analytics initialized"); }
  destroy() { console.log("Analytics destroyed"); }
  track(event: string) { console.log(`Tracking: ${event}`); }
}

// Система плагинов
class PluginManager {
  private plugins: Plugin[] = [];
  
  addPlugin(plugin: Plugin) {
    this.plugins.push(plugin);
    plugin.init();
  }
  
  shutdown() {
    this.plugins.forEach(p => p.destroy());
  }
}

const manager = new PluginManager();
manager.addPlugin(new LoggerPlugin());      // OK
manager.addPlugin(new AnalyticsPlugin());   // OK - структурно совместим
```

## Ограничения и подводные камни

### 1. Приватные и защищенные свойства
```typescript
class Animal {
  name: string;
  private species: string;
  
  constructor(name: string, species: string) {
    this.name = name;
    this.species = species;
  }
}

class Person {
  name: string;
  private species: string; // та же структура, но private
  
  constructor(name: string, species: string) {
    this.name = name;
    this.species = species;
  }
}

let animal: Animal;
let person: Person;

// Это работает, потому что структура одинакова
animal = new Person("John", "Homo sapiens"); // OK
person = new Animal("Fluffy", "Canis lupus"); // OK

// Но если классы не связаны иерархией, поведение может отличаться
class Cat {
  name: string;
  private species: string;
  // private property не влияет на структурную типизацию
  // при одинаковом имени и типе
  
  constructor(name: string, species: string) {
    this.name = name;
    this.species = species;
  }
}

// Все три класса структурно совместимы
animal = new Cat("Whiskers", "Felis catus"); // OK
```

### 2. Статические свойства
```typescript
class Base {
  static type = "base";
  instanceProp: string;
  
  constructor(prop: string) {
    this.instanceProp = prop;
  }
}

class Derived {
  static type = "derived"; // другое значение, но OK
  instanceProp: string;
  
  constructor(prop: string) {
    this.instanceProp = prop;
  }
}

let base: Base;
// base = new Derived("test"); // OK для инстансов

// Но статические свойства не учитываются в структурной типизации
// Base и Derived структурно совместимы по инстансам
```

### 3. Типизация массивов
```typescript
class Cat {
  meow() { console.log("meow"); }
}

class Dog {
  bark() { console.log("woof"); }
}

// Массивы типизированы строго по типу
const cats: Cat[] = [];
const dogs: Dog[] = [];

// TypeScript считает массивы инвариантными, поэтому:
// cats = dogs; // ОШИБКА - несмотря на утиную типизацию

// Но если элементы массива совместимы:
const catlike: { meow(): void }[] = dogs.map(dog => ({
  meow: () => console.log("fake meow")
})); // OK
```

## Практические советы

### 1. Явное указание типов для безопасности
```typescript
// Хотя утиная типизация гибкая, иногда явные типы важны
interface Config {
  apiUrl: string;
  timeout: number;
}

// Явно указываем тип, чтобы избежать передачи неподходящих объектов
function setupApi(config: Config) {
  console.log(`Connecting to ${config.apiUrl} with ${config.timeout}ms timeout`);
}

// Это позволит передать структурно совместимый объект
setupApi({ 
  apiUrl: "https://api.example.com", 
  timeout: 5000,
  extraProp: "ignored" // будет проигнорировано
});

// Но если нам важна строгость:
function setupApiStrict(config: Required<Config>) {
  // теперь все свойства обязательны
}
```

### 2. Использование branded types для дополнительной безопасности
```typescript
// Для критичных ситуаций можно использовать branded types
type UserId = string & { readonly brand: unique symbol };
type OrderId = string & { readonly brand: unique symbol };

// Эти типы структурно совместимы как строки, но не совместимы друг с другом
const userId: UserId = "user123" as UserId;
const orderId: OrderId = "order456" as OrderId;

// userId = orderId; // ОШИБКА - несмотря на утиную типизацию
// orderId = userId; // ОШИБКА

// Но оба можно использовать как обычные строки
function processString(str: string) {
  console.log(str.toUpperCase());
}

processString(userId);  // OK
processString(orderId); // OK
```

## Лучшие практики

### 1. Используйте интерфейсы для документирования контрактов
```typescript
// Вместо жесткой зависимости от класса
interface DataProcessor {
  process(data: any): any;
  validate(data: any): boolean;
}

// Компонент работает с любым объектом, реализующим контракт
function useProcessor(processor: DataProcessor) {
  // использование
}
```

### 2. Используйте утиную типизацию для гибкости API
```typescript
interface Drawable {
  x: number;
  y: number;
  draw(): void;
}

// API может принимать любой объект с нужной структурой
function render(drawables: Drawable[]) {
  drawables.forEach(item => item.draw());
}

// Разработчики могут передавать свои собственные классы/объекты
class Circle {
  x = 10;
  y = 20;
  radius = 5;
  draw() { console.log("Drawing circle"); }
}

const circle = new Circle();
// circle можно использовать с render, если явно указать Drawable
const drawableCircle: Drawable = circle; // OK
```

## Сравнение с другими системами типизации

### Duck Typing vs Nominal Typing
```typescript
// TypeScript (структурная типизация - утиная типизация)
interface IFlyable {
  fly(): void;
}

class Airplane {
  fly() { console.log("Airplane flying"); }
}

class Bird {
  fly() { console.log("Bird flying"); }
}

// Оба класса могут быть использованы как IFlyable без явного наследования
const plane: IFlyable = new Airplane(); // OK
const bird: IFlyable = new Bird();      // OK

// В языках с номинальной типизацией (например, Java):
// class Airplane и class Bird должны явно implement IFlyable
// чтобы быть совместимыми
```

## Совместимость с другими концепциями TypeScript

### 1. С обобщениями
```typescript
// Утиная типизация работает с обобщениями
interface Comparable<T> {
  compareTo(other: T): number;
}

function max<T extends Comparable<T>>(a: T, b: T): T {
  return a.compareTo(b) > 0 ? a : b;
}

// Любой объект с методом compareTo может быть использован
class NumberContainer implements Comparable<NumberContainer> {
  constructor(public value: number) {}
  
  compareTo(other: NumberContainer): number {
    return this.value - other.value;
  }
}

const cont1 = new NumberContainer(10);
const cont2 = new NumberContainer(20);
const maxContainer = max(cont1, cont2); // OK
```

### 2. С условными типами
```typescript
// Утиная типизация влияет на вывод типов в условных типах
type IsDuckTyped<T> = T extends { quack(): any; swim(): any } ? true : false;

type Result1 = IsDuckTyped<{ quack(): void; swim(): void; name: string }>; // true
type Result2 = IsDuckTyped<{ bark(): void; run(): void }>; // false
```

## Заключение

Duck Typing (структурная типизация) в TypeScript:

1. **Обеспечивает гибкость** - можно использовать объекты без явного наследования
2. **Упрощает интеграцию** - легче работать с внешними библиотеками
3. **Улучшает тестируемость** - проще создавать моки
4. **Требует внимательности** - может привести к неожиданным совместимостям

Понимание утиной типизации критически важно для эффективного использования TypeScript, особенно при работе с интерфейсами, тестированием и интеграцией с внешними системами.

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - общая система совместимости типов
- [[structural-typing]] - структурная типизация как основа утиной типизации
- [[interfaces]] - как интерфейсы используют утиную типизацию
- [[type-narrowing]] - как сужение типов взаимодействует с утиной типизацией
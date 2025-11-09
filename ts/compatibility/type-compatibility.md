# TypeScript Type Compatibility

Совместимость типов в TypeScript определяет, когда одно значение может быть использовано в позиции, ожидаемой другим типом. TypeScript использует структурную (устиную) типизацию, где совместимость определяется формой значений, а не их именами.

## Основные принципы совместимости

### Структурная типизация
```typescript
// TypeScript использует структурную типизацию
interface Named {
  name: string;
}

class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

// Совместимость: структура совпадает, несмотря на разные имена
let named: Named = new Person("Alice"); // OK

// Совместимость объектов
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

let point2D: Point2D = { x: 1, y: 2 };
let point3D: Point3D = { x: 1, y: 2, z: 3 };

// Point3D совместим с Point2D, так как имеет все необходимые свойства
let p2: Point2D = point3D; // OK
// Point2D НЕ совместим с Point3D, так как отсутствует z
// let p3: Point3D = point2D; // ОШИБКА
```

### Совместимость функций

#### Вариантность параметров
```typescript
// Параметры функций контравариантны (в более широком типе)
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

// y может быть присвоена x, потому что x игнорирует дополнительные параметры
x = y; // OK

// x НЕ может быть присвоена y, потому что y ожидает больше параметров
// y = x; // ОШИБКА

// Пример с иерархией типов
interface Empty {}
interface NotEmpty { name: string; }

let empty: Empty = {};
let notEmpty: NotEmpty = { name: "Alice" };

// Функция, принимающая более широкий тип (Empty)
let f1 = (arg: Empty) => {};
// Функция, принимающая более узкий тип (NotEmpty)
let f2 = (arg: NotEmpty) => {};

// f1 может принимать все, что может f2, и больше
f1 = f2; // OK - контравариантность
// f2 = f1; // ОШИБКА - не всегда можно передать Empty в место, ожидающее NotEmpty
```

#### Ковариантность возвращаемых значений
```typescript
// Возвращаемые значения ковариантны (в более узком типе)
let g1 = (): Empty => ({});
let g2 = (): NotEmpty => ({ name: "Alice" });

// g2 может быть присвоена g1, потому что NotEmpty может использоваться как Empty
g1 = g2; // OK
// g2 = g1; // ОШИБКА - Empty не всегда подходит там, где ожидается NotEmpty
```

### Совместимость классов

#### Приватные и защищенные члены
```typescript
class Animal {
  private name: string;
  constructor(name: string) { this.name = name; }
}

class Cat extends Animal {
  constructor(name: string) { super(name); }
}

class Dog {
  private name: string;
  constructor(name: string) { this.name = name; }
}

let animal: Animal;
let cat: Cat;
let dog: Dog;

animal = new Cat("Whiskers"); // OK - наследование
// animal = new Dog("Buddy"); // ОШИБКА - разные приватные поля

// Dog и Animal не связаны иерархией, 
// поэтому их экземпляры несовместимы из-за разного private name
```

### Совместимость generics

#### Простая совместимость обобщений
```typescript
interface GenericIdentityFn<T> {
  (arg: T): T;
}

let identity = <T>(arg: T): T => arg;

// Совместимость обобщенных типов
let genericFn: GenericIdentityFn<number> = identity; // OK
let stringIdentity: GenericIdentityFn<string> = identity; // OK

// Пример с более конкретными типами
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

// Совместимость только с типами, имеющими length
loggingIdentity({ length: 10, value: 3 }); // OK
// loggingIdentity(3); // ОШИБКА - number не имеет length
```

## Совместимость примитивов

### Совместимость null и undefined
```typescript
// В strict mode null и undefined несовместимы с другими типами
let str: string;
// str = null; // ОШИБКА при strictNullChecks
// str = undefined; // ОШИБКА при strictNullChecks

// Совместимость с объединениями
let possiblyNull: string | null = "hello";
possiblyNull = null; // OK

// Совместимость с undefined
let possiblyUndefined: number | undefined;
possiblyUndefined = undefined; // OK
possiblyUndefined = 42; // OK
```

### Совместимость с any и unknown
```typescript
let anyType: any;
let unknownType: unknown;
let stringType: string = "hello";
let numberType: number = 42;

// any совместим с любыми типами
anyType = stringType; // OK
anyType = numberType; // OK
stringType = anyType; // OK
numberType = anyType; // OK

// unknown совместим с собой и может быть присвоен any
unknownType = stringType; // OK
unknownType = numberType; // OK
anyType = unknownType; // OK
// stringType = unknownType; // ОШИБКА - требует проверки или утверждения
```

## Совместимость объединений и пересечений

### Совместимость объединений
```typescript
interface Dog { name: string; bark(): void; }
interface Cat { name: string; meow(): void; }

let dog: Dog = { name: "Buddy", bark: () => {} };
let cat: Cat = { name: "Whiskers", meow: () => {} };

// Совместимость с объединением
let pet: Dog | Cat = dog; // OK
pet = cat; // OK

// При доступе к свойствам - только общие свойства доступны без проверки
// pet.bark(); // ОШИБКА - не все варианты имеют bark
// pet.meow(); // ОШИБКА - не все варианты имеют meow
pet.name; // OK - обе сущности имеют name

// Использование type guards для безопасного доступа
function makePetSound(pet: Dog | Cat) {
  if ("bark" in pet) {
    pet.bark(); // OK - TypeScript знает, что это Dog
  } else {
    pet.meow(); // OK - TypeScript знает, что это Cat
  }
}
```

### Совместимость пересечений
```typescript
interface Contact {
  name: string;
  phoneNumber: string;
}

interface Employee {
  id: number;
  department: string;
}

type EmployeeContact = Contact & Employee;

let contact: Contact = { name: "Alice", phoneNumber: "123-456" };
let employee: Employee = { id: 1, department: "Engineering" };

// Создание пересечения типов
let empContact: EmployeeContact = {
  ...contact,
  ...employee
}; // OK - имеет все необходимые свойства

// Совместимость: экземпляр пересечения должен удовлетворять всем типам в пересечении
let verifiedContact: Contact = empContact; // OK - имеет все свойства Contact
let verifiedEmployee: Employee = empContact; // OK - имеет все свойства Employee
```

## Совместимость функций высшего порядка

### Совместимость функций в функциях
```typescript
type Callback = (data: string) => void;
type AsyncCallback = (data: string, error?: Error) => void;

let normalCallback: Callback = (data) => console.log(data);
let asyncCallback: AsyncCallback = (data, error) => {
  if (error) console.error(error);
  else console.log(data);
};

// asyncCallback может быть присвоена normalCallback (дополнительные параметры игнорируются)
normalCallback = asyncCallback; // OK
// asyncCallback = normalCallback; // ОШИБКА - ожидает ошибку

// Пример с функциями, возвращающими функции
type StringProcessor = (input: string) => string;
type StringValidator = (input: string) => boolean;

function createValidator(validator: StringValidator): StringProcessor {
  return (input: string) => {
    if (validator(input)) {
      return input;
    }
    return "INVALID";
  };
}

// Совместимость функций как параметров
const isLong: StringValidator = (s) => s.length > 10;
const processor = createValidator(isLong); // OK
```

## Совместимость массивов

### Совместимость элементов массива
```typescript
let stringArray: string[] = ["hello", "world"];
let readonlyStringArray: readonly string[] = ["readonly", "array"];

// Массив может быть присвоен readonly массиву
readonlyStringArray = stringArray; // OK
// stringArray = readonlyStringArray; // ОШИБКА - readonly не может быть изменяемым

// Совместимость объединений в массивах
type StringOrNumber = string | number;
let stringOrNumberArray: StringOrNumber[] = [1, "hello", 2];
let numberArray: number[] = [1, 2, 3];

// numberArray совместим с StringOrNumber[] (number является подтипом string | number)
stringOrNumberArray = numberArray; // OK
// numberArray = stringOrNumberArray; // ОШИБКА - не все элементы могут быть number
```

## Практические случаи совместимости

### Совместимость с интерфейсами и реализациями
```typescript
interface Logger {
  log(message: string): void;
}

interface TimestampedLogger extends Logger {
  logWithTimestamp(message: string): void;
}

class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
  
  // Дополнительные методы разрешены
  info(message: string): void {
    console.log(`[INFO] ${message}`);
  }
}

class AdvancedLogger implements TimestampedLogger {
  log(message: string): void {
    console.log(`[ADVANCED] ${message}`);
  }
  
  logWithTimestamp(message: string): void {
    console.log(`[${new Date().toISOString()}] ${message}`);
  }
}

// Совместимость с базовыми интерфейсами
let baseLogger: Logger = new ConsoleLogger(); // OK
baseLogger = new AdvancedLogger(); // OK - AdvancedLogger реализует Logger

// Совместимость с производными интерфейсами
let timestampedLogger: TimestampedLogger = new AdvancedLogger(); // OK
// timestampedLogger = new ConsoleLogger(); // ОШИБКА - нет logWithTimestamp
```

### Совместимость с декораторами и миксинами
```typescript
type Constructor<T = {}> = new (...args: any[]) => T;

// Миксин для добавления логирования
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

interface Point {
  x: number;
  y: number;
}

class PointClass {
  constructor(public x: number, public y: number) {}
}

// Создание класса с миксином
const TimestampedPoint = Timestamped(PointClass);

let point: Point = new PointClass(1, 2); // OK
let timestampedPoint = new TimestampedPoint(1, 2); // OK
// timestampedPoint имеет тип, созданный миксином

// Совместимость: экземпляр с таймстемпом также является Point
let anotherPoint: Point = timestampedPoint; // OK
```

## Совместимость с функциями обратного вызова

### Совместимость callback функций
```typescript
interface HandlerEvent {
  type: string;
  timestamp: number;
}

interface ClickEvent extends HandlerEvent {
  type: "click";
  x: number;
  y: number;
}

interface HoverEvent extends HandlerEvent {
  type: "hover";
  element: HTMLElement;
}

// Обработчик базового события
function handleEvent(event: HandlerEvent) {
  console.log(`Event at ${event.timestamp}: ${event.type}`);
}

// Обработчик конкретного события
function handleClick(event: ClickEvent) {
  console.log(`Click at (${event.x}, ${event.y})`);
}

// handleClick может быть использован везде, где ожидается HandlerEvent
// из-за ковариантности возвращаемого типа и контравариантности параметров
handleEvent = handleClick; // OK - ClickEvent является HandlerEvent

// Но не наоборот - более конкретный обработчик не может принимать базовые события
// handleClick = handleEvent; // ОШИБКА - HandlerEvent не всегда является ClickEvent
```

## Лучшие практики совместимости

### 1. Использование интерфейсов для контрактов
```typescript
// Вместо жесткой зависимости от конкретных классов
interface ConfigProvider {
  get(key: string): string | undefined;
  set(key: string, value: string): void;
}

// Функция работает с любым объектом, реализующим интерфейс
function useConfig(provider: ConfigProvider) {
  const apiUrl = provider.get("apiUrl");
  console.log("API URL:", apiUrl);
}

// Разные реализации могут использоваться с тем же интерфейсом
class EnvConfig implements ConfigProvider {
  get(key: string): string | undefined {
    return process.env[key];
  }
  
  set(key: string, value: string): void {
    process.env[key] = value;
  }
}

const envConfig = new EnvConfig();
useConfig(envConfig); // OK
```

### 2. Использование условных типов для проверки совместимости
```typescript
// Проверка совместимости типов во время компиляции
type IsCompatible<T, U> = T extends U ? true : false;

type Result1 = IsCompatible<string, string | number>; // true
type Result2 = IsCompatible<string | number, string>; // false
type Result3 = IsCompatible<{ name: string }, { name: string; age: number }>; // false
type Result4 = IsCompatible<{ name: string; age: number }, { name: string }>; // true
```

### 3. Использование utility types для обеспечения совместимости
```typescript
// Использование Partial для совместимости с частичными объектами
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

function updateUser(id: number, updates: Partial<User>) {
  // Обновление только указанных полей
  console.log(`Updating user ${id} with:`, updates);
}

// Совместимость: можно передать только часть свойств
updateUser(1, { name: "New Name" }); // OK
updateUser(2, { email: "new@example.com", isActive: true }); // OK
```

## Ограничения и подводные камни

### Контравариантность параметров в методах классов
```typescript
class A { a: string = "a"; }
class B extends A { b: string = "b"; }
class C extends A { c: string = "c"; }

class Foo {
  foo(x: A) { console.log("Foo"); }
}

class Bar extends Foo {
  // ОШИБКА при строгой проверке: параметр не может быть более конкретным
  // foo(x: C) { console.log("Bar"); } // ОШИБКА - это нарушает контравариантность
  
  // ПРАВИЛЬНО: параметр должен быть таким же или более широким
  foo(x: A) { console.log("Bar override"); } // OK
}
```

### Совместимость функций с разным числом параметров
```typescript
type BinaryFunction = (a: number, b: number) => number;
type UnaryFunction = (a: number) => number;

let binary: BinaryFunction = (a, b) => a + b;
let unary: UnaryFunction = (a) => a * 2;

// В JavaScript: лишние параметры игнорируются
// В TypeScript: это определяется системой типов
unary = binary; // OK в TypeScript (лишний параметр игнорируется)
// binary = unary; // ОШИБКА - ожидается больше параметров
```

## Сравнение подходов

### Структурная vs Номинальная типизация
```typescript
// Структурная (TypeScript)
interface Bird { 
  fly(): void; 
  layEggs(): void; 
}

interface Fish { 
  swim(): void; 
  layEggs(): void; 
}

// Эти типы могут использоваться в местах, где нужен общий интерфейс
interface EggLayer { 
  layEggs(): void; 
}

function incubate(creature: EggLayer) {
  creature.layEggs();
}

incubate({ fly: () => {}, layEggs: () => {} } as Bird); // OK
incubate({ swim: () => {}, layEggs: () => {} } as Fish); // OK

// В номинальной системе (Java/C#) это не было бы возможно без явного наследования
```

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - детали системы совместимости
- [[../type-system/duck-typing]] - утиная типизация как основа
- [[../interfaces-classes/type-compatibility]] - совместимость классов
- [[../generics/variance]] - вариативность в обобщениях
- [[../advanced-types/type-guards]] - проверки типов для обеспечения совместимости
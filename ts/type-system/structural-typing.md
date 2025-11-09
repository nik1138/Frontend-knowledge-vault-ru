# Structural Typing in TypeScript

Структурная типизация (Structural Typing) - это система типизации, при которой совместимость типов определяется их структурой (формой), а не их именем или происхождением. TypeScript использует структурную типизацию (иногда называемую "утиной типизацией" - "duck typing"), что отличает его от номинальной типизации в языках вроде Java и C#.

## Основные концепции

### Структурная vs Номинальная типизация
```typescript
// Номинальная типизация (как в Java/C#):
// Два типа совместимы только если имеют общее наследование или явное объявление связи

// Структурная типизация (как в TypeScript):
// Два типа совместимы если имеют одинаковую структуру (поля и методы)

// Пример структурной типизации
interface Named {
  name: string;
}

class Person {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Dog {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

let n: Named;

// Оба класса могут быть присвоены переменной типа Named
// потому что имеют структуру (свойство name: string)
n = new Person("Alice"); // OK
n = new Dog("Buddy");    // OK - несмотря на другое имя класса

// И даже простые объекты
n = { name: "Anonymous" }; // OK - структура совпадает
```

### Компоновка объектов
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

const point2D: Point2D = { x: 1, y: 2 };
const point3D: Point3D = { x: 1, y: 2, z: 3 };

// Point3D может быть использован как Point2D (имеет все требуемые поля)
const another2D: Point2D = point3D; // OK
// Point2D НЕ может быть использован как Point3D (отсутствует z)
// const another3D: Point3D = point2D; // ОШИБКА
```

## Совместимость объектов

### Подмножества и надмножества
```typescript
interface Square {
  size: number;
}

interface Rectangle {
  width: number;
  height: number;
}

interface Shape {
  color: string;
}

interface SquareWithColor extends Square, Shape {}

// Компоновка структур
const square: Square = { size: 5 };
const rect: Rectangle = { width: 10, height: 5 };
const shape: Shape = { color: "red" };

// Совместимость через структуру
const mixed = { size: 10, color: "blue" };
const squareWithColor: SquareWithColor = mixed; // OK
```

### Дополнительные свойства
```typescript
interface BasicConfig {
  name: string;
}

// Объект с дополнительными свойствами
const extendedConfig = {
  name: "app",
  version: "1.0.0",    // дополнительное свойство
  debug: true          // дополнительное свойство
};

// Может быть присвоен базовому интерфейсу
const config: BasicConfig = extendedConfig; // OK: имеет все необходимые свойства
// Но обратное НЕВЕРНО:
// const extended: typeof extendedConfig = config; // ОШИБКА: отсутствуют свойства

// Ограничение: нельзя присваивать литералы с "лишними" свойствами напрямую
// const wrong: BasicConfig = { name: "test", extra: "property" }; // ОШИБКА в strict mode
// Но можно через переменную:
const temp = { name: "test", extra: "property" };
const correct: BasicConfig = temp; // OK
```

## Совместимость функций

### Структурная типизация функций
```typescript
// Функции совместимы если:
// 1. Параметры совместимы (контравариантно)
// 2. Возвращаемое значение совместимо (ковариантно)

let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

// x может быть присвоена y? ДА: x игнорирует лишние параметры
x = y; // OK

// y может быть присвоена x? НЕТ: y требует больше параметров
// y = x; // ОШИБКА

// Возвращаемые значения (ковариантность)
let f = () => ({ name: "Alice" });
let g = () => ({ name: "Alice", age: 30 });

// g может быть присвоена f? ДА: { name: string, age: number } может использоваться как { name: string }
f = g; // OK

// f может быть присвоена g? НЕТ: { name: string } не может всегда использоваться как { name: string, age: number }
// g = f; // ОШИБКА
```

### Контравариантность параметров
```typescript
interface Empty {}
interface NotEmpty {
  name: string;
}

let x1 = (a: Empty) => {};
let y1 = (b: NotEmpty) => {};

// Это работает: функция, которая принимает более общий тип, может принимать более конкретный
// (поскольку она может обработать все, что может обработать более конкретный тип)
x1 = y1; // OK

// Это не работает: функция, которая ожидает конкретный тип, не может обработать более общий
// y1 = x1; // ОШИБКА
```

### Совместимость методов
```typescript
class Animal {
  feed() {}
}

class Dog extends Animal {
  bark() {}
}

class Cat extends Animal {
  meow() {}
}

let animalActions: (animal: Animal) => void;
let dogActions: (dog: Dog) => void;

// Контравариантность: параметры могут быть более общими
animalActions = dogActions; // OK: (animal: Animal) => void может принять (dog: Dog) => void
// dogActions = animalActions; // ОШИБКА: (dog: Dog) => void не может гарантировать (animal: Animal) => void

// Практический пример с коллбэками
interface EventCallback<T> {
  (event: T): void;
}

const animalCallback: EventCallback<Animal> = (animal) => {
  animal.feed();
};

const dogCallback: EventCallback<Dog> = (dog) => {
  dog.feed();
  dog.bark();
};

// animalCallback может использоваться там, где ожидается EventCallback<Dog>
// поскольку любой код, ожидающий Dog, может обработать Animal
const callbacks: EventCallback<Dog>[] = [animalCallback]; // OK
// const dogCallbacks: EventCallback<Animal>[] = [dogCallback]; // ОШИБКА
```

## Совместимость классов

### Структурная совместимость классов
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
}

let p: Point = new Point2D(1, 2); // OK: структура совпадает

// Приватные и защищенные члены влияют на структурную совместимость
class Animal {
  private name: string;
  constructor(name: string) { this.name = name; }
}

class Cat {
  private name: string; // Другой класс с такой же структурой, но приватное поле
  constructor(name: string) { this.name = name; }
}

// let animal: Animal = new Cat("Whiskers"); // ОШИБКА: приватные поля в разных классах

// Приватные поля из одной иерархии разрешены
class Dog extends Animal {
  constructor(name: string) { 
    super(name); // наследует приватное поле name
  }
}

let dog: Animal = new Dog("Buddy"); // OK: из одной иерархии
```

### Защищенные члены
```typescript
class Base {
  protected value: number;
  constructor(value: number) {
    this.value = value;
  }
}

class Derived extends Base {
  constructor(value: number) {
    super(value);
  }
  
  getValue(): number {
    return this.value; // может получить доступ к protected
  }
}

class Other extends Base {
  constructor(value: number) {
    super(value);
  }
  
  compare(other: Base): boolean {
    // return this.value === other.value; // ОШИБКА: не может получить доступ к другому экземпляру
    return this.value === (other as Derived).value; // "неправильно", но возможно
  }
}

let base: Base = new Derived(1); // OK
let derived: Derived = new Base(1) as Derived; // ОШИБКА: нельзя напрямую, нужен cast
```

## Совместимость обобщений

### Структурная типизация с обобщениями
```typescript
interface GenericIdentityFn<T> {
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity; // OK
// myIdentity: (arg: number) => number

// Совместимость между разными обобщенными типами
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // OK: теперь мы знаем, что у arg есть свойство length
  return arg;
}

// loggingIdentity может принимать как string (имеет length), так и any[] (имеет length)
const str = loggingIdentity("hello"); // OK: string имеет length
const arr = loggingIdentity([1, 2, 3]); // OK: any[] имеет length
```

### Вариантность в обобщениях
```typescript
// TypeScript использует инвариантность для типа параметров обобщений
interface KeyValuePair<K, V> {
  key: K;
  value: V;
}

let pair1: KeyValuePair<string, number> = { key: "age", value: 30 };
let pair2: KeyValuePair<string, any> = { key: "metadata", value: { complex: true } };

// pair2 = pair1; // ОШИБКА: несмотря на то, что number может быть использован как any
// pair1 = pair2; // ОШИБКА: несмотря на то, что string может быть использован как string

// Но если использовать covariant отношение:
interface ReadOnlyPair<K, V> {
  readonly key: K;
  readonly value: V;
}

let readOnlyPair1: ReadOnlyPair<string, number> = { key: "age", value: 30 };
let readOnlyPair2: ReadOnlyPair<string, any> = { key: "metadata", value: { complex: true } };

readOnlyPair2 = readOnlyPair1; // OK: ковариантность readonly свойств
// readOnlyPair1 = readOnlyPair2; // ОШИБКА: все еще инвариантно для K

// Вариантность с массивами
let strings: string[] = ["hello", "world"];
let stringsAsAny: any[] = strings; // OK: массивы ковариантны для элементов

// Но это потенциально небезопасно:
// stringsAsAny[0] = 5; // OK для TS, но ломает strings[0] как string
// console.log(strings[0].toUpperCase()); // ОШИБКА во время выполнения!
```

## Продвинутые примеры

### Совместимость с индексными сигнатурами
```typescript
interface Dictionary {
  [key: string]: string;
}

interface NumberDictionary {
  [key: string]: number;
  length: number;    // OK: это число, совместимо с индексной сигнатурой
  name: string;      // ОШИБКА: строка несовместима с number
}

// Это будет работать:
interface StringNumericDictionary {
  [key: string]: number | string;
  name: string;      // OK: string является подмножеством number | string
  length: number;    // OK: number является подмножеством number | string
}

const dict: Dictionary = {};
dict["hello"] = "world"; // OK

const numDict: StringNumericDictionary = { name: "test", length: 10 };
numDict["dynamic"] = 42; // OK: number
numDict["another"] = "value"; // OK: string
```

### Совместимость функций с разными сигнатурами
```typescript
// Совместимость через структуру
interface FunctionSignature {
  (x: string): number;
  (x: number): string;
}

const overloadedFunction: FunctionSignature = (x: string | number): any => {
  if (typeof x === "string") {
    return x.length;
  }
  return x.toString();
};

// Совместимость с перегрузками
function createFunction(): FunctionSignature {
  return (x: string | number): any => {
    if (typeof x === "string") {
      return x.length;
    }
    return x.toString();
  };
}
```

### Совместимость с функциональными интерфейсами
```typescript
// Совместимость с функциональными типами
type CompareFunction<T> = (a: T, b: T) => number;

interface ICompareFunction<T> {
  (a: T, b: T): number;
}

const compareNumbers: CompareFunction<number> = (a, b) => a - b;
const compareStrings: ICompareFunction<string> = (a, b) => a.localeCompare(b);

// Совместимость между функциональным типом и интерфейсом
const func1: ICompareFunction<number> = compareNumbers; // OK
const func2: CompareFunction<string> = compareStrings;  // OK
```

## Практические применения

### Совместимость с внешними библиотеками
```typescript
// TypeScript не требует точного соответствия импортам
// если структура совместима

// Допустим, библиотека экспортирует:
interface ExternalInterface {
  name: string;
  initialize(): void;
  destroy(): void;
}

// В нашем коде:
interface OurInterface {
  name: string;
  initialize(): void;
  destroy(): void;
  // дополнительные поля разрешены
  optionalField?: boolean;
}

// Совместимость: объект с нашей структурой может использоваться 
// там, где ожидается внешний интерфейс
const ourImplementation: OurInterface = {
  name: "Our Implementation",
  initialize() { console.log("Initialized"); },
  destroy() { console.log("Destroyed"); },
  optionalField: true
};

// Может быть использован в библиотечной функции, ожидая ExternalInterface
function useExternalLib(obj: ExternalInterface) {
  obj.initialize();
  console.log(obj.name);
  obj.destroy();
}

useExternalLib(ourImplementation); // OK: структурно совместимо
```

### Совместимость в API
```typescript
// Совместимость при работе с API ответами
interface ApiResponse {
  success: boolean;
  data?: any;
  error?: string;
}

interface UserResponse {
  success: boolean; // тот же ключ и тип
  data?: {          // тот же ключ, но более конкретный тип
    id: number;
    name: string;
  };
  error?: string;   // тот же ключ и тип
  // дополнительные поля разрешены
  timestamp?: number;
}

// Потому что UserResponse имеет все поля ApiResponse и все их типы совместимы
const response: ApiResponse = {
  success: true,
  data: { id: 1, name: "Alice" },
  timestamp: Date.now() // лишнее поле разрешено
} as UserResponse; // или через интерфейс
```

### Типобезопасная работа с несовместимыми структурами
```typescript
// Решение проблем с несовместимостью через адаптеры
interface LegacyUser {
  id: number;
  name: string;
  email_address: string;
}

interface ModernUser {
  id: number;
  name: string;
  email: string;
}

// Адаптер для совместимости
class UserAdapter {
  static toModern(legacy: LegacyUser): ModernUser {
    return {
      id: legacy.id,
      name: legacy.name,
      email: legacy.email_address
    };
  }
  
  static fromModern(modern: ModernUser): LegacyUser {
    return {
      id: modern.id,
      name: modern.name,
      email_address: modern.email
    };
  }
}

// Использование
const legacyUser: LegacyUser = { id: 1, name: "Alice", email_address: "alice@example.com" };
const modernUser: ModernUser = UserAdapter.toModern(legacyUser);

// Теперь modernUser может использоваться в современном коде
```

## Проблемы и ловушки

### Частые ошибки с структурной типизацией
```typescript
// Проблема: структурная типизация может быть слишком либеральной
interface ConfigA {
  url: string;
  timeout: number;
}

interface ConfigB {
  url: string;
  timeout: number;
}

const configA: ConfigA = { url: "https://api.a.com", timeout: 5000 };
const configB: ConfigB = configA; // ОК: структурно совместимо
// Но ConfigA и ConfigB могут быть предназначены для разных целей!

// Обходное решение: брендирование типов
type Branded<T, B> = T & { readonly __brand: B };

type ConfigA = Branded<{ url: string; timeout: number }, 'ConfigA'>;
type ConfigB = Branded<{ url: string; timeout: number }, 'ConfigB'>;

// const configB2: ConfigB = configA as ConfigA; // ОШИБКА: бренды разные
```

### Совместимость с null/undefined
```typescript
// В strict mode null/undefined несовместимы с другими типами
interface User {
  name: string;
}

// let user: User = null; // ОШИБКА в strict mode
// let user: User | null = null; // OK

// Практическое применение:
function findUser(id: number): User | null {
  // поиск пользователя
  return null; // если не найден
}

const foundUser = findUser(123);
if (foundUser) { // сужение типа
  // foundUser: User в этой ветке
  console.log(foundUser.name); // безопасный доступ
}
// foundUser: User | null в остальном коде
```

### Совместимость массивов и кортежей
```typescript
// Совместимость массивов
const numbers: number[] = [1, 2, 3];
const numbersAsObjects: (number | string)[] = numbers; // OK: number подходит под number | string

// Совместимость кортежей
const tuple: [string, number] = ["hello", 42];
const pair: [string, number] = tuple; // OK: та же структура

// Но кортеж несовместим с массивом определенного типа
const stringArray: string[] = ["hello", "world"];
// const tuple2: [string, string] = stringArray; // ОШИБКА: нет гарантии длины/типов
```

## Современные паттерны

### Брендированные типы для безопасности
```typescript
// Использование branded типов для предотвращения смешивания
type Brand<K, T> = K & { __brand: T };

type Email = Brand<string, "Email">;
type Password = Brand<string, "Password">;
type UserId = Brand<number, "UserId">;

function validateEmail(email: string): email is Email {
  const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  return isValid;
}

// Использование
function createUser(email: Email, password: Password, id: UserId): boolean {
  return true;
}

// createUser("not-an-email", "123", 1); // ОШИБКА: нет брендов
const email = "test@example.com" as Email;
const password = "secret" as Password; 
const userId = (123 as any) as UserId;

createUser(email, password, userId); // OK: правильные бренды
```

### Использование с tagged unions
```typescript
// Структурная типизация работает отлично с дискриминированными объединениями
interface LoadingState {
  status: "loading";
  progress?: number;
}

interface SuccessState {
  status: "success";
  data: string;
}

interface ErrorState {
  status: "error";
  error: string;
}

type State = LoadingState | SuccessState | ErrorState;

function handleState(state: State) {
  // TypeScript может сузить тип благодаря структурной типизации и общему полю status
  switch (state.status) {
    case "loading":
      // state имеет тип LoadingState
      console.log(state.progress ?? 0);
      break;
    case "success":
      // state имеет тип SuccessState
      console.log(state.data);
      break;
    case "error":
      // state имеет тип ErrorState
      console.log(state.error);
      break;
  }
}
```

## Лучшие практики

### 1. Понимание принципов
```typescript
// Ясно понимайте, что структурная типизация:
// - Основана на форме, а не на имени
// - Оценивает совместимость по содержимому
// - Позволяет больше свободы, чем номинальная типизация
```

### 2. Использование интерфейсов для контрактов
```typescript
// Используйте интерфейсы для определения четких контрактов
interface UserRepository {
  findById(id: number): Promise<User | null>;
  save(user: User): Promise<void>;
}

// Даже если реализация имеет дополнительные методы, 
// она остается совместимой с интерфейсом
```

### 3. Осторожность с лишними свойствами
```typescript
// Будьте осторожны при присвоении литералов объектов
// TypeScript проверяет лишние свойства при прямом присвоении
const user1 = { name: "Alice", id: 1 };
const user2: { name: string } = user1; // OK: через переменную

// Но напрямую:
// const user3: { name: string } = { name: "Alice", id: 1 }; // ОШИБКА в strict mode
```

### 4. Использование readonly для контроля изменений
```typescript
// readonly помогает предотвратить нежелательные изменения структуры
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

// Это гарантирует, что объекты Config не будут изменены
```

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - общие правила совместимости типов
- [[duck-typing]] - концепция "утиная типизация"
- [[interfaces]] - интерфейсы и структурная типизация
- [[classes]] - классы и структурная типизация
- [[generics]] - обобщения и вариянтность
- [[advanced-types/discriminated-unions]] - объединения и структурная проверка
- [[functional-programming/typing]] - функциональная типизация
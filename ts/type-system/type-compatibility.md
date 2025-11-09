# Совместимость типов в TypeScript

## Введение

Совместимость типов в TypeScript определяет, когда одно значение может быть присвоено переменной другого типа. В отличие от многих других языков программирования, TypeScript использует структурную типизацию, при которой совместимость типов определяется их структурой, а не именем или иерархией наследования. Это позволяет создавать гибкие и переиспользуемые типы.

## Структурная типизация

Структурная типизация означает, что два типа считаются совместимыми, если у них одинаковая структура, независимо от их именования:

```typescript
interface Bird {
  name: string;
  fly(): void;
}

interface Fish {
  name: string;
  swim(): void;
}

class Sparrow {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  fly() {
    console.log(`${this.name} летит`);
  }
}

// Sparrow совместим с Bird, потому что имеет те же свойства и методы
const bird: Bird = new Sparrow("Воробей"); // OK

// Это также работает, даже если Sparrow не наследуется от Fish
// const fish: Fish = new Sparrow("Воробей"); // ОШИБКА: нет метода swim
```

### Совместимость объектных типов

```typescript
interface Point {
  x: number;
  y: number;
}

interface Point3D {
  x: number;
  y: number;
  z: number;
}

const point2D: Point = { x: 1, y: 2 };
const point3D: Point3D = { x: 1, y: 2, z: 3 };

// Point3D совместим с Point (но не наоборот)
const p: Point = point3D; // OK: Point3D имеет все свойства Point
// const p3d: Point3D = point2D; // ОШИБКА: point2D не имеет свойства z

// Практический пример
interface Config {
  apiUrl: string;
  timeout: number;
}

interface DetailedConfig {
  apiUrl: string;
  timeout: number;
  retries: number;
  debug: boolean;
}

// Можно использовать DetailedConfig там, где ожидается Config
function setup(config: Config) {
  console.log(`API: ${config.apiUrl}, timeout: ${config.timeout}`);
}

const detailedConfig: DetailedConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
  debug: true
};

setup(detailedConfig); // OK: DetailedConfig совместим с Config
```

## Совместимость функций

Совместимость функций в TypeScript определяется тремя аспектами: параметрами, возвращаемым значением и типами исключений.

### Совместимость параметров

```typescript
// Более конкретная функция может быть присвоена менее конкретной
type Input = { value: string };
type Data = { value: string; timestamp: Date };

let inputHandler: (data: Input) => void;
let dataHandler: (data: Data) => void;

// dataHandler может быть присвоен inputHandler
// потому что Data имеет все свойства Input
inputHandler = dataHandler; // OK

// Обратное не работает
// dataHandler = inputHandler; // ОШИБКА: Input не имеет timestamp

// Пример с параметрами
function forEach<T>(items: T[], callback: (item: T) => void): void {
  for (const item of items) {
    callback(item);
  }
}

const items = [1, 2, 3];

// Можно передать функцию с более общей сигнатурой
forEach(items, (item) => console.log(item)); // OK: параметр number
forEach(items, (item: number | string) => console.log(item)); // OK: параметр более общий
```

### Совместимость возвращаемых значений

```typescript
interface Animal { name: string }
interface Dog extends Animal { breed: string }

let animalFetcher: () => Animal;
let dogFetcher: () => Dog;

// dogFetcher может быть присвоен animalFetcher
// потому что Dog является подтипом Animal
animalFetcher = dogFetcher; // OK

// Обратное не работает
// dogFetcher = animalFetcher; // ОШИБКА: Animal не обязательно Dog
```

### Совместимость функций с разным количеством параметров

```typescript
let simpleFunction: (x: number) => number;
let complexFunction: (x: number, y: number) => number;

// TypeScript позволяет передавать функцию с большим количеством параметров
// в место, где ожидается функция с меньшим количеством параметров
simpleFunction = complexFunction; // OK: лишние параметры игнорируются

// Но не наоборот
// complexFunction = simpleFunction; // ОШИБКА: не хватает параметров

// Пример на практике
function callFunction(fn: (x: number) => number, value: number): number {
  return fn(value);
}

const addFive = (x: number, y: number = 5): number => x + y;
const result = callFunction(addFive, 10); // OK: лишний параметр игнорируется
```

## Совместимость массивов и подтипов

```typescript
interface Bird {
  name: string;
  fly(): void;
}

interface Eagle extends Bird {
  dive(): void;
}

let birdArray: Bird[];
let eagleArray: Eagle[];

// Совместимость массивов не инвариантна, а ковариантна для чтения
// eagleArray = birdArray; // ОШИБКА: не безопасно для записи
birdArray = eagleArray; // OK: Eagle[] может быть использован как Bird[]

// Но это может привести к проблемам при записи
// birdArray.push({ name: "Странная птица", fly: () => {} }); // OK по типу
// Теперь eagleArray содержит не Eagle!

// Поэтому TypeScript запрещает такое присваивание при строгой проверке
```

## Совместимость классов

Классы в TypeScript совместимы, если их экземпляры совместимы. Статические части не учитываются при проверке совместимости:

```typescript
class Animal {
  feet: number;
  constructor(name: string, numFeet: number) { }
}

class Size {
  feet: number;
  constructor(meters: number) { }
}

let a: Animal;
let s: Size;

a = s;  // OK: структура совпадает
s = a;  // OK: структура совпадает

// Совместимость с интерфейсами
interface Robot {
  name: string;
}

class Android {
  name: string = "";
  version: number = 0;
}

const robot: Robot = new Android(); // OK: Android имеет все свойства Robot
```

## Совместимость enum

Enum'ы совместимы с number, а number совместимы с enum'ами:

```typescript
enum Status { Ready, Waiting }
enum Color { Red, Blue, Green }

let statusEnum: Status = Status.Ready;
let colorEnum: Color = Color.Red;

// Enum'ы не совместимы друг с другом
// statusEnum = colorEnum; // ОШИБКА: Enum'ы несовместимы
// colorEnum = statusEnum; // ОШИБКА: Enum'ы несовместимы

// Но enum совместим с number
let num: number = Status.Ready; // OK
statusEnum = 1; // OK: присвоение числа enum'у
```

## Совместимость с any, void, never

```typescript
// Совместимость с any
let anyValue: any;
let stringValue: string = "строка";

anyValue = stringValue; // OK
stringValue = anyValue; // OK: any может быть присвоен любому типу

// Совместимость с void
function returnVoid(): void {
  return undefined; // или просто return
}

let fn: () => void;
fn = returnVoid; // OK

// Совместимость с never
function throwError(): never {
  throw new Error("Ошибка");
}

// never может быть присвоен любому типу
let str: string = throwError(); // OK, но код не выполнится
```

## Практические примеры

### Совместимость в API клиентах

```typescript
interface ApiResponse {
  success: boolean;
  data: any;
}

interface UserResponse extends ApiResponse {
  data: {
    id: number;
    name: string;
    email: string;
  };
}

interface ProductResponse extends ApiResponse {
  data: {
    id: number;
    title: string;
    price: number;
  };
}

function handleResponse(response: ApiResponse) {
  if (response.success) {
    console.log("Данные:", response.data);
  } else {
    console.log("Ошибка запроса");
  }
}

// Можно передавать более конкретные типы
const userResponse: UserResponse = {
  success: true,
  data: { id: 1, name: "Алексей", email: "alex@example.com" }
};

handleResponse(userResponse); // OK: UserResponse совместим с ApiResponse
```

### Совместимость в системах плагинов

```typescript
interface Plugin {
  name: string;
  version: string;
  initialize(): void;
}

interface AdvancedPlugin extends Plugin {
  settings: Record<string, any>;
  onBeforeExecute?(): void;
  onAfterExecute?(): void;
}

function registerPlugin(plugin: Plugin) {
  console.log(`Регистрация плагина: ${plugin.name}`);
  plugin.initialize();
}

const advancedPlugin: AdvancedPlugin = {
  name: "Расширенный плагин",
  version: "1.0.0",
  settings: { debug: true },
  initialize() {
    console.log("Инициализация расширенного плагина");
  }
};

registerPlugin(advancedPlugin); // OK: AdvancedPlugin совместим с Plugin
```

## Связь с другими концепциями

- [[type-system/structural-typing|Структурная типизация]] - основа системы совместимости
- [[type-system/type-compatibility|Совместимость типов]] - более глубокое рассмотрение темы
- [[interfaces-classes/interfaces|Интерфейсы]] - как они влияют на совместимость
- [[advanced-types/conditional-types|Условные типы]] - для создания гибких систем типов

## Лучшие практики

1. **Понимайте разницу между номинальной и структурной типизацией** - TypeScript использует структурную типизацию
2. **Будьте осторожны с массивами и изменяемыми данными** - совместимость может привести к ошибкам
3. **Используйте интерфейсы для определения контрактов** - для лучшей совместимости
4. **Проверяйте совместимость в сложных иерархиях типов** - особенно при использовании обобщений

## Заключение

Совместимость типов - ключевая концепция TypeScript, обеспечивающая гибкость при сохранении безопасности типов. Понимание принципов совместимости позволяет создавать более переиспользуемые и надежные типы, а также избегать распространенных ошибок при работе с системой типов.
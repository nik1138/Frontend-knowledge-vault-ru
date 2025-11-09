# Структурная типизация в TypeScript

Структурная типизация (structural typing) - это система типизации, при которой совместимость типов определяется их структурой (членами), а не именами или декларацией. Это ключевое отличие TypeScript от языков с номинальной типизацией, где совместимость определяется именем типа.

## Основные принципы структурной типизации

В структурно типизированном языке два типа считаются совместимыми, если они имеют одинаковую структуру, независимо от их имен. Это означает, что объект может использоваться везде, где ожидается другой тип, если он имеет все необходимые свойства.

```typescript
interface Point {
  x: number;
  y: number;
}

class Point2D {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

// Совместимость: структура совпадает, несмотря на разные имена и типы
const myPoint: Point = new Point2D(1, 2);  // OK
```

## Сравнение с номинальной типизацией

### Номинальная типизация (например, Java, C#)

```java
// В языке с номинальной типизацией
class Point { int x, y; }
class Point2D { int x, y; }

// Point p = new Point2D(); // ОШИБКА: несовместимые типы
```

### Структурная типизация (TypeScript)

```typescript
interface Point { x: number; y: number; }
class Point2D { x: number; y: number; constructor(x: number, y: number) { this.x = x; this.y = y; }}

const p: Point = new Point2D(1, 2); // OK: структура совпадает
```

## Принципы совместимости

### Совместимость объектов

Объект типа S совместим с типом T, если S содержит все члены T (и, возможно, больше):

```typescript
interface Named {
  name: string;
}

interface Person extends Named {
  age: number;
}

const person: Person = { name: "Alice", age: 30 };
const named: Named = person;  // OK: Person имеет все свойства Named

// Но не наоборот:
// const person2: Person = named;  // ОШИБКА: у Named нет свойства age
```

### Совместимость функций

Функции совместимы, если их сигнатуры совпадают. В контексте функций применяется правило контравариантности для параметров и ковариантности для возвращаемых значений:

```typescript
// Более "широкая" функция может заменить более "узкую"
type GetPerson = () => Person;
type GetNamed = () => Named;

const getPerson: GetPerson = () => ({ name: "Alice", age: 30 });
const getNamed: GetNamed = getPerson;  // OK: Person может использоваться как Named

// Параметры: контравариантны
type PersonHandler = (p: Person) => void;
type NamedHandler = (n: Named) => void;

// PersonHandler может использоваться как NamedHandler
const handlePerson: PersonHandler = (p) => console.log(p.name, p.age);
const handleNamed: NamedHandler = handlePerson;  // OK: Person может использоваться как Named
```

## Практические примеры

### 1. Совместимость анонимных объектов

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
}

// Анонимный объект совместим с интерфейсом, если имеет необходимые свойства
const config: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3  // Дополнительные свойства разрешены
};  // OK: структура совпадает
```

### 2. Совместимость с классами

```typescript
interface Drawable {
  draw(): void;
}

class Circle {
  radius: number;
  constructor(radius: number) {
    this.radius = radius;
  }
  
  draw() {
    console.log(`Drawing circle with radius ${this.radius}`);
  }
  
  getArea() {
    return Math.PI * this.radius ** 2;
  }
}

const shape: Drawable = new Circle(5);  // OK: Circle имеет метод draw()
```

### 3. Совместимость массивов

```typescript
interface HasName {
  name: string;
}

class Employee {
  name: string;
  department: string;
  
  constructor(name: string, department: string) {
    this.name = name;
    this.department = department;
  }
}

const employees: Employee[] = [
  new Employee("Alice", "Engineering"),
  new Employee("Bob", "Marketing")
];

// Массив Employee[] совместим с HasName[]
const names: HasName[] = employees;  // OK: каждый Employee имеет свойство name
```

## Утиная типизация

Структурная типизация в TypeScript также известна как "утиная типизация" по принципу: "Если выглядит как утка, плавает как утка и крякает как утка, то это, вероятно, утка".

```typescript
interface Duck {
  swim(): void;
  quack(): void;
}

class Dog {
  swim() { console.log("Dog swimming"); }
  quack() { console.log("Dog making duck sound"); }
  bark() { console.log("Dog barking"); }
}

const duck: Duck = new Dog();  // OK: Dog имеет все методы Duck
```

## Продвинутые паттерны

### 1. Совместимость с опциональными свойствами

```typescript
interface BasicConfig {
  apiUrl: string;
}

interface FullConfig {
  apiUrl: string;
  timeout?: number;
  retries?: number;
}

const basic: BasicConfig = { apiUrl: "https://api.com" };
const full: FullConfig = basic;  // OK: опциональные свойства могут отсутствовать
```

### 2. Совместимость функций с разными параметрами

```typescript
type ThreeParams = (a: number, b: number, c: number) => number;
type TwoParams = (a: number, b: number) => number;

// Функция с меньшим количеством параметров может использоваться там, где ожидается больше
const addTwo: TwoParams = (a, b) => a + b;
const addThree: ThreeParams = addTwo;  // OK: третий параметр игнорируется
```

### 3. Совместимость с индексными сигнатурами

```typescript
interface Dictionary {
  [key: string]: string;
}

const obj = { name: "Alice", city: "London" };
const dict: Dictionary = obj;  // OK: все свойства строки и значения строки
```

## Ограничения и ловушки

### 1. Строгая проверка объектных литералов

```typescript
interface Options {
  width: number;
  height: number;
}

// Объектный литерал проверяется строже
// const opt: Options = { width: 100, height: 200, extra: true };  // ОШИБКА

// Но переменная с таким же содержимым будет OK
const objWithExtra = { width: 100, height: 200, extra: true };
const opt: Options = objWithExtra;  // OK: избыточные свойства игнорируются
```

### 2. Совместимость enum и number

```typescript
enum Status { Ready, Waiting }

const status = Status.Ready;
const num: number = status;  // OK: enum совместим с number
const status2: Status = num;  // OK: number совместим с enum (в TypeScript)
```

### 3. Совместимость void

```typescript
interface Callback {
  (result: string): void;
}

const fn: Callback = (result: string) => {
  return result.length;  // OK: возвращаемое значение игнорируется
};
```

## Преимущества структурной типизации

1. **Гибкость**: можно использовать объекты разных типов, если они имеют одинаковую структуру
2. **Легкость мокирования**: легко создавать mock-объекты для тестирования
3. **Интероперабельность**: легче интегрировать с JavaScript-библиотеками
4. **Меньше boilerplate кода**: не нужно создавать много наследуемых классов

## Недостатки и соображения

1. **Меньше безопасности**: может быть сложнее обнаружить логические ошибки
2. **Нечеткая семантика**: имя типа не всегда отражает намерение
3. **Сложность отладки**: сложнее понять, какие типы используются на самом деле

## Практические применения

### 1. Создание гибких API

```typescript
interface Plugin {
  name: string;
  activate: () => void;
}

// Любой объект с нужной структурой может быть плагином
const myPlugin = {
  name: "MyPlugin",
  activate: () => console.log("Activated"),
  extraMethod: () => console.log("Extra")  // Дополнительные методы разрешены
};

function registerPlugin(plugin: Plugin) {
  console.log(`Registering ${plugin.name}`);
  plugin.activate();
}

registerPlugin(myPlugin);  // OK: структура совпадает
```

### 2. Работа с внешними библиотеками

```typescript
// Интерфейс для описания формы данных из внешней библиотеки
interface ExternalDataFormat {
  id: number;
  title: string;
  metadata: Record<string, any>;
}

// Наш внутренний формат может быть другим, но структурно совместимым
interface InternalFormat {
  id: number;
  title: string;
  metadata: Record<string, any>;
  createdAt: Date;
}

// Внешняя библиотека возвращает ExternalDataFormat
const externalData: ExternalDataFormat[] = getFromExternalLib();

// Мы можем использовать эти данные, даже если наш внутренний формат более подробный
const internalData: InternalFormat[] = externalData.map(item => ({
  ...item,
  createdAt: new Date()
}));
```

## Лучшие практики

1. **Используйте интерфейсы для определения контрактов**
2. **Будьте осторожны с избыточными свойствами в литералах**
3. **Явно указывайте типы в API, чтобы избежать неоднозначности**
4. **Используйте `Pick`, `Omit` и другие утилиты для создания производных типов**

## Связи с другими концепциями

- [[ts/type-system/type-compatibility|Совместимость типов]] - более общее понятие
- [[ts/type-system/duck-typing|Утиная типизация]] - частный случай структурной типизации
- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]] - как использовать структурную типизацию
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - продвинутые паттерны
- [[ts/basics/control-flow|Анализ потока управления]] - как структурная типизация взаимодействует с сужением типов
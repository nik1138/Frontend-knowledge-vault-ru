# TypeScript Interfaces

Интерфейсы в TypeScript определяют контракты в вашем коде и могут использоваться в качестве подсказок для компилятора о форме объекта. Они описывают, какие свойства должен иметь объект, а также их типы.

## Базовое использование

### Простой интерфейс
```typescript
interface User {
  name: string;
  age: number;
}

function greet(user: User): string {
  return `Hello, ${user.name}`;
}

// Использование
const person: User = { name: "Alice", age: 30 };
console.log(greet(person)); // "Hello, Alice"
```

### Необязательные свойства
```typescript
interface User {
  name: string;
  age: number;
  email?: string; // Необязательное свойство
}

// Оба варианта валидны:
const user1: User = { name: "Alice", age: 30 };                    // OK
const user2: User = { name: "Bob", age: 25, email: "bob@example.com" }; // OK
```

### Только для чтения свойства
```typescript
interface User {
  readonly id: number; // Только для чтения
  name: string;
}

let user: User = { id: 123, name: "Alice" };
// user.id = 456; // Ошибка: нельзя изменить readonly свойство
```

## Расширенные возможности

### Расширение интерфейсов (наследование)
```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  employeeId: number;
  department: string;
}

const employee: Employee = {
  name: "Alice",
  age: 30,
  employeeId: 123,
  department: "Engineering"
};
```

### Множественное наследование
```typescript
interface Shape {
  color: string;
}

interface PenStroke {
  penWidth: number;
}

interface Square extends Shape, PenStroke {
  sideLength: number;
}

const square: Square = {
  color: "blue",
  penWidth: 1,
  sideLength: 5
};
```

## Функциональные типы в интерфейсах

### Определение сигнатур функций
```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string): boolean {
  return source.search(subString) !== -1;
};
```

### Индексные сигнатуры
```typescript
// Строковые индексные сигнатуры
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

// Словарь с произвольными строковыми ключами
interface NumberDictionary {
  [index: string]: number;
  length: number;    // Тип этого свойства должен быть подмножеством индексной сигнатуры
  // name: string;   // Ошибка: тип string несовместим с number
}
```

### Гибридные типы
```typescript
interface Counter {
  (start: number): string;
  interval: number;
  reset(): void;
}

function getCounter(): Counter {
  let counter = (function (start: number) { return String(start); }) as Counter;
  counter.interval = 123;
  counter.reset = function () { };
  return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

## Интерфейсы и классы

### Реализация интерфейсов классами
```typescript
interface ClockInterface {
  currentTime: Date;
  setTime(d: Date): void;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  
  setTime(d: Date) {
    this.currentTime = d;
  }
  
  constructor(h: number, m: number) {
    // дополнительная инициализация
  }
}
```

### Интерфейсы для конструкторов
```typescript
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  
  setTime(d: Date) {
    this.currentTime = d;
  }
  
  constructor(h: number, m: number) {}
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
  return new ctor(hour, minute);
}
```

## Расширение интерфейсов классов

### Интерфейс может расширять класс
```typescript
class Control {
  private state: any;
}

interface SelectableControl extends Control {
  select(): void;
}

class Button extends Control implements SelectableControl {
  select() { }
}

class TextBox extends Control {
  select() { }
}

// Ошибка: Image не наследует Control
// class Image implements SelectableControl {
//   select() { }
// }
```

## Обобщенные интерфейсы

### Интерфейсы с обобщениями
```typescript
interface GenericIdentityFn<T> {
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

### Обобщенные интерфейсы с ограничениями
```typescript
interface Lengthwise {
  length: number;
}

interface GenericFn<T extends Lengthwise> {
  (arg: T): T;
}

function genericFunction<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // OK: теперь мы знаем, что у arg есть свойство length
  return arg;
}

// genericFunction(3); // Ошибка: number не имеет length
genericFunction({ length: 10, value: 3 }); // OK
```

## Практические примеры

### Интерфейсы для конфигурации
```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  retries: number;
}

function setup(config: Config): void {
  console.log(`Setting up API at ${config.apiUrl}`);
}

// Позволяет передавать объект с нужными свойствами
setup({
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
});
```

### Интерфейсы для API ответов
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  timestamp: number;
}

interface User {
  id: number;
  name: string;
  email: string;
}

// Использование в API вызовах
async function fetchUser(id: number): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

### Интерфейсы для функций обратного вызова
```typescript
interface EventHandler<T> {
  (event: T): void;
}

interface ButtonClickEvent {
  x: number;
  y: number;
  target: HTMLElement;
}

interface Component {
  onClick: EventHandler<ButtonClickEvent>;
  onMouseOver?: EventHandler<MouseEvent>;
}
```

## Операции с интерфейсами

### Повторное объявление (declaration merging)
```typescript
interface User {
  name: string;
}

interface User {
  age: number;
}

// Результирующий интерфейс объединяет оба объявления
// equivalent to: interface User { name: string; age: number; }

const user: User = { name: "Alice", age: 30 }; // OK
```

### Интерфейсы vs Типы
```typescript
// Интерфейс - может быть расширен
interface User {
  name: string;
}
interface User {  // OK: дополнительное объявление
  age: number;
}

// Тип - не может быть повторно объявлен
type UserType = {
  name: string;
};
// type UserType = { age: number; };  // Ошибка: повторное объявление

// Интерфейсы могут расширять типы и наоборот
type Name = { name: string };
interface UserWithAge extends Name {
  age: number;
}
```

## Продвинутые паттерны

### Интерфейсы для функций-генераторов
```typescript
interface Generator<T> {
  next(): { value: T; done: boolean };
  return?(value?: T): { value: T; done: boolean };
  throw?(e?: any): { value: T; done: boolean };
}
```

### Интерфейсы для асинхронных операций
```typescript
interface AsyncResult<T, E = Error> {
  success: true;
  data: T;
} | {
  success: false;
  error: E;
}

async function safeOperation<T>(fn: () => Promise<T>): Promise<AsyncResult<T>> {
  try {
    const data = await fn();
    return { success: true, data };
  } catch (error) {
    return { success: false, error } as AsyncResult<T>;
  }
}
```

## Лучшие практики

1. **Используйте интерфейсы для публичных API** - они лучше документируют контракты
2. **Предпочитайте интерфейсы типам-псевдонимам** - для объектных форм
3. **Давайте интерфейсам понятные имена** - отражающие их назначение
4. **Используйте наследование интерфейсов** - для построения иерархий
5. **Избегайте очень сложных интерфейсов** - разбивайте на более мелкие
6. **Документируйте интерфейсы** - особенно сложные или неочевидные

## Связь с другими концепциями
- [[../interfaces-classes/classes]] - как интерфейсы связаны с классами
- [[../generics/generic-interfaces]] - обобщенные интерфейсы
- [[../types/type-aliases]] - сравнение с типами-псевдонимами  
- [[TypeScript Advanced Types]] - продвинутые возможности интерфейсов
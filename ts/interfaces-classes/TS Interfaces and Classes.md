# TypeScript Interfaces and Classes

Интерфейсы и классы в TypeScript предоставляют мощные средства для определения форм и поведения объектов, а также для организации кода в объектно-ориентированном стиле.

## Основные концепции

### [Interfaces](interfaces.md)
Определение форм и контрактов для объектов

### [Classes](classes.md)
Определение объектов с состоянием и поведением

### [Inheritance](inheritance.md)
Наследование классов и интерфейсов

### [Access Modifiers](access-modifiers.md)
Управление доступом к членам классов

### [Abstract Classes](abstract-classes.md)
Абстрактные классы и методы

### [Encapsulation](encapsulation.md)
Сокрытие данных и внутренней реализации

## Сравнение интерфейсов и классов

| Feature | Interface | Class |
|---------|-----------|-------|
| Определение формы | ✅ | через реализацию |
| Реализация | ❌ (только определения) | ✅ |
| Наследование | `extends` | `extends` |
| Использование `new` | ❌ | ✅ |
| Статические члены | ❌ | ✅ |
| Конструкторы | ❌ | ✅ |

## Примеры использования

### Интерфейс для объекта
```typescript
interface User {
  name: string;
  email: string;
  readonly id: number; // только для чтения
  login?(): void;     // необязательный метод
}

const user: User = {
  name: "Alice",
  email: "alice@example.com",
  id: 123
};
```

### Класс с наследованием
```typescript
interface Animal {
  name: string;
  makeSound(): void;
}

class Dog implements Animal {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  makeSound(): void {
    console.log("Woof!");
  }
}

class Cat extends AnimalBase {
  purr(): void {
    console.log("Purr...");
  }
}
```

## Расширенные возможности

### Гибридный интерфейс (метод и объект)
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
```

### Интерфейсы для функций
```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string): boolean {
  return source.search(subString) !== -1;
};
```

### Классы как типы и значения
```typescript
class Greeter {
  greeting: string;
  
  constructor(message: string) {
    this.greeting = message;
  }
  
  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter: Greeter; // как тип
let greeterInstance = new Greeter("world"); // как значение
```

## Совместимость и реализация

### Интерфейс реализует класс
```typescript
class Point {
  x: number = 0;
  y: number = 0;
}

interface Point3D extends Point {
  z: number;
}

const point3D: Point3D = { x: 1, y: 2, z: 3 };
```

### Класс реализует несколько интерфейсов
```typescript
interface Drawable {
  draw(): void;
}

interface Resizable {
  resize(width: number, height: number): void;
}

class Window implements Drawable, Resizable {
  draw() { /* реализация */ }
  resize(width: number, height: number) { /* реализация */ }
}
```

## Продвинутые паттерны

### Защита типов через интерфейсы
```typescript
interface Bird {
  type: 'bird';
  fly(): void;
}

interface Fish {
  type: 'fish';
  swim(): void;
}

type Animal = Bird | Fish;

function move(animal: Animal) {
  switch (animal.type) {
    case 'bird':
      animal.fly(); // TypeScript знает, что это Bird
      break;
    case 'fish':
      animal.swim(); // TypeScript знает, что это Fish
      break;
  }
}
```

### Интерфейсы для словарей
```typescript
interface StringArray {
  [index: number]: string;
}

interface NumberDictionary {
  [index: string]: number;
  length: number;    // OK, length это число
  name: string;      // Ошибка: тип несовместим
}
```

## Лучшие практики

1. Используйте интерфейсы для определения форм
2. Используйте классы для определения поведения и состояния
3. Интерфейсы могут быть повторно открыты для добавления свойств
4. Предпочитайте композицию наследованию

## Связь с другими концепциями
- [[types]] - основы типизации для интерфейсов и классов
- [[TS Generics]] - обобщенные интерфейсы и классы
- [[TypeScript Advanced Types]] - расширенные возможности типизации
- [[../type-system/type-compatibility]] - совместимость типов
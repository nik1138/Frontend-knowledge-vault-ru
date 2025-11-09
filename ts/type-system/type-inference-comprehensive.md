# Вывод типов в TypeScript

Вывод типов (type inference) - это способность компилятора TypeScript автоматически определять типы переменных, параметров функций и возвращаемых значений без явного указания аннотаций типов. Это одна из ключевых особенностей TypeScript, которая делает язык более удобным в использовании, сохраняя при этом строгую типизацию.

## Основные принципы вывода типов

TypeScript использует алгоритмы вывода типов, основанные на контексте использования и присваиваемых значениях. Вывод типов работает в следующих случаях:

1. При инициализации переменных и членов
2. При установлении значений по умолчанию параметров
3. При возвращении значений из функций
4. При определении типов литералов

### Простой вывод типов

```typescript
// TypeScript выводит тип number
let x = 3;

// TypeScript выводит тип string
let y = "hello";

// TypeScript выводит тип boolean
let z = true;
```

### Вывод типов в функциях

```typescript
// TypeScript выводит возвращаемый тип number
function add(a: number, b: number) {
  return a + b;  // Возвращаемый тип выводится как number
}

// TypeScript выводит тип параметра для функции обратного вызова
const names = ["Alice", "Bob", "Charlie"];
names.forEach(s => console.log(s));  // s выводится как string
```

## Контекстный вывод

Контекстный вывод (contextual typing) - это более сложная форма вывода типов, при которой тип выражения определяется по контексту, в котором оно используется.

```typescript
// Контекстный вывод для объектов
window.onmousedown = function(mouseEvent) {
  // mouseEvent выводится как MouseEvent благодаря контексту
  console.log(mouseEvent.button);  // OK
  // console.log(mouseEvent.kangaroo); // Ошибка: нет такого свойства
};

// Контекстный вывод для массивов
const handlers: ((e: MouseEvent) => void)[] = [];
handlers.push((mouseEvent) => console.log(mouseEvent.clientX));  // mouseEvent выводится как MouseEvent
```

## Лучшие практики вывода типов

### 1. Явные аннотации для API

Для функций, которые являются частью публичного API, рекомендуется использовать явные аннотации типов:

```typescript
// Хорошо: явная аннотация для API функции
function getUser(id: number): Promise<User> {
  // реализация
}

// Не рекомендуется для API: полагаться только на вывод типов
function getUser(id) {  // Тип параметра и возвращаемого значения неясен
  // реализация
}
```

### 2. Вывод типов для локальных переменных

Для локальных переменных и вспомогательных функций можно полагаться на вывод типов:

```typescript
function processUsers(users: User[]) {
  // Тип переменной usersCount выводится как number
  const usersCount = users.length;
  
  // Тип переменной activeUsers выводится как User[]
  const activeUsers = users.filter(user => user.isActive);
  
  // Тип переменной names выводится как string[]
  const names = activeUsers.map(user => user.name);
  
  return names;
}
```

## Вывод типов в сложных структурах

### Объектные типы

```typescript
// TypeScript выводит сложный вложенный тип
const config = {
  server: {
    host: "localhost",
    port: 3000,
    ssl: false
  },
  database: {
    url: "mongodb://localhost:27017/myapp",
    options: {
      useNewUrlParser: true,
      useUnifiedTopology: true
    }
  }
};

// Тип config выводится как:
// {
//   server: {
//     host: string;
//     port: number;
//     ssl: boolean;
//   };
//   database: {
//     url: string;
//     options: {
//       useNewUrlParser: boolean;
//       useUnifiedTopology: boolean;
//     };
//   };
// }
```

### Массивы и кортежи

```typescript
// Вывод типов для массивов
const numbers = [1, 2, 3];  // number[]
const strings = ["a", "b", "c"];  // string[]
const mixed = [1, "hello", true];  // (number | string | boolean)[]

// Вывод типов для кортежей
const tuple = [1, "hello", true];  // [number, string, boolean] (если определено как кортеж)
```

## Вывод типов в условных выражениях

TypeScript может выводить типы на основе условных выражений:

```typescript
interface Bird {
  kind: "bird";
  flyingSpeed: number;
}

interface Horse {
  kind: "horse";
  runningSpeed: number;
}

type Animal = Bird | Horse;

function moveAnimal(animal: Animal) {
  // TypeScript понимает, что если animal.kind === "bird", 
  // то animal должен быть типа Bird
  if (animal.kind === "bird") {
    console.log(animal.flyingSpeed);  // OK: TypeScript знает, что это Bird
  } else {
    console.log(animal.runningSpeed); // OK: TypeScript знает, что это Horse
  }
}
```

## Вывод типов в дженериках

```typescript
// TypeScript выводит тип параметра T для функции
function createArray<T>(length: number, element: T): T[] {
  return Array.from({ length }, () => element);
}

// Тип T выводится как string
const strings = createArray(3, "hello");

// Тип T выводится как number
const numbers = createArray(5, 42);
```

## Ограничения вывода типов

### 1. Недостаточно информации

```typescript
// В некоторых случаях TypeScript не может вывести тип
function f() {
  return { x: 1, y: 2 };  // Возвращаемый тип: { x: number; y: number; }
}

// Но если присвоить результат переменной, тип может быть уточнен
const point = f();  // Тип: { x: number; y: number; }

// Или можно использовать утверждение типа
const specificPoint = f() as { x: number; y: number; z?: number };
```

### 2. Рекурсивные структуры

```typescript
// В рекурсивных структурах может потребоваться явное указание типа
interface Node {
  value: number;
  left?: Node;
  right?: Node;
}

// TypeScript может не справиться с выводом для сложных рекурсивных структур
const tree: Node = {
  value: 1,
  left: {
    value: 2,
    left: { value: 3 }
  },
  right: {
    value: 4
  }
};
```

## Расширенный вывод типов

### Вывод типов с помощью infer

В условных типах можно использовать `infer` для вывода типов:

```typescript
// Условный тип с выводом
type ElementType<T> = T extends (infer U)[] ? U : T;

type T0 = ElementType<string[]>;  // string
type T1 = ElementType<number>;    // number
```

### Вывод типов в промисах

```typescript
// TypeScript может выводить тип разрешаемого значения промиса
async function fetchData() {
  const response = await fetch('/api/data');
  const data = await response.json();  // Тип выводится как any, но может быть уточнен
  return data;
}

// Более безопасный подход с явным указанием типа
async function fetchTypedData(): Promise<{ id: number; name: string }> {
  const response = await fetch('/api/data');
  const data = await response.json();  // Все еще any, но возвращаемый тип указан
  return data as { id: number; name: string };
}
```

## Сравнение с другими языками

### Вывод типов vs. C# var

В C# ключевое слово `var` требует инициализации при объявлении:

```csharp
var name = "John";  // OK
// var surname;  // Ошибка: var требует инициализации
```

В TypeScript вывод типов работает аналогично, но также применяется в других контекстах:

```typescript
let name = "John";  // OK
// let surname;  // surname: any (не ошибка, но тип any)
```

### Вывод типов vs. F# типизация

F# использует более агрессивную систему вывода типов:

```fsharp
let add x y = x + y  // Тип функции выводится как int -> int -> int
```

TypeScript требует больше явных указаний, особенно для параметров функций, но все же имеет мощный вывод типов.

## Практические применения

### 1. Уменьшение дублирования кода

```typescript
// Без вывода типов - много повторений
const user: User = { name: "Alice", age: 30 } as User;
const users: User[] = [user] as User[];
const activeUsers: User[] = users.filter((u: User) => u.age > 18) as User[];

// С выводом типов - код чище
const user = { name: "Alice", age: 30 };  // Тип выводится
const users = [user];  // Тип выводится как { name: string; age: number; }[]
const activeUsers = users.filter(u => u.age > 18);  // Тип выводится
```

### 2. Повышение читаемости кода

```typescript
// Явные аннотации могут перегружать код
function complexOperation(input: ComplexInputType): ComplexOutputType {
  const intermediateResult: ComplexIntermediateType = processInput(input);
  const finalResult: ComplexOutputType = transformResult(intermediateResult);
  return finalResult;
}

// Вывод типов упрощает код
function complexOperation(input: ComplexInputType) {
  const intermediateResult = processInput(input);  // Тип выводится
  const finalResult = transformResult(intermediateResult);  // Тип выводится
  return finalResult;  // Возвращаемый тип выводится
}
```

## Связи с другими концепциями

- [[ts/type-system/type-compatibility|Совместимость типов]] - как вывод типов взаимодействует с совместимостью
- [[ts/generics/TS Generics|Обобщения]] - вывод типов в обобщенных функциях
- [[ts/advanced-types/conditional-types|Условные типы]] - использование infer для вывода типов
- [[ts/basics/type-annotations|Аннотации типов]] - основы типизации
- [[ts/type-system/type-guards|Защитники типов]] - сужение типов после вывода
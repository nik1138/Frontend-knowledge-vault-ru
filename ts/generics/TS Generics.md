# TypeScript Generics

Generics (обобщения) - это мощная функция TypeScript, позволяющая создавать компоненты, которые работают с различными типами без потери информации о типах. Они обеспечивают переиспользуемость кода при сохранении строгой типизации.

## Основные концепции

### [Generic Functions](generic-functions.md)
Создание функций, которые работают с любыми типами

### [Generic Interfaces](generic-interfaces.md)
Интерфейсы, параметризованные типами

### [Generic Classes](generic-classes.md)
Классы с обобщенными параметрами

### [Generic Constraints](constraints.md)
Ограничения на типы параметров обобщений

### [Multiple Type Parameters](multiple-parameters.md)
Использование нескольких параметров типов

## Простые примеры

### Обобщенная функция
```typescript
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("hello");    // тип string
let output2 = identity(42);                 // тип number (вывод)
```

### Обобщенный интерфейс
```typescript
interface GenericIdentityFn<T> {
  (arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

### Обобщенный класс
```typescript
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = (x, y) => x + y;
```

## Ограничения обобщений

### Constraint с интерфейсом
```typescript
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // OK, теперь мы знаем, что у arg есть свойство length
  return arg;
}

loggingIdentity({ length: 10, value: 3 }); // OK
// loggingIdentity(3); // Ошибка: number не имеет length
```

### Использование ключей объекта
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3 };
getProperty(x, "a"); // OK
// getProperty(x, "m"); // Ошибка: "m" не является ключом x
```

## Продвинутые паттерны

### Использование классов как типов
```typescript
function create<T>(c: { new(): T }): T {
  return new c();
}
```

### Вариативные обобщения (T extends any[])
```typescript
function tuple<T extends any[]>(...args: T): T {
  return args;
}

const result = tuple("hello", 42); // [string, number]
```

## Встроенные обобщенные типы

TypeScript предоставляет множество встроенных обобщенных типов:

- `Array<T>` - массив элементов типа T
- `Promise<T>` - промис, разрешающийся значением типа T
- `Partial<T>` - делает все свойства T необязательными
- `Readonly<T>` - делает все свойства T только для чтения
- `Pick<T, K>` - создает тип с подмножеством свойств T
- `Record<K, T>` - объект с ключами типа K и значениями типа T

### Примеры встроенных обобщений
```typescript
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

const todo1: Partial<Todo> = { title: "Buy milk" }; // Все свойства необязательны

const todo2: Readonly<Todo> = {
  title: "Delete me",
  description: "This task will be deleted",
  completed: false
};
// todo2.completed = true; // Ошибка: не может быть изменено

type TodoPreview = Pick<Todo, "title" | "completed">;
// TodoPreview = { title: string; completed: boolean }

type TodoInfo = Omit<Todo, "completed">;
// TodoInfo = { title: string; description: string }
```

## Практические применения

### Коллекции
```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
}

const numberStack = new Stack<number>();
const stringStack = new Stack<string>();
```

### API клиенты
```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

async function fetchUser(id: number): Promise<ApiResponse<User>> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

## Лучшие практики

1. Используйте обобщения для создания переиспользуемых компонентов
2. Применяйте ограничения (constraints) для безопасности типов
3. Используйте встроенные обобщенные типы для часто используемых паттернов
4. Обобщения не должны усложнять код без необходимости

## Связь с другими концепциями
- [[advanced-types/mapped-types]] - преобразование типов с помощью обобщений
- [[advanced-types/conditional-types]] - условные типы, часто используемые с обобщениями
- [[TS Interfaces and Classes]] - обобщенные интерфейсы и классы
- [[type-system/type-inference]] - вывод типов в обобщениях
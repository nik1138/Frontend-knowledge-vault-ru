# TypeScript Advanced Types

Продвинутые типы в TypeScript открывают возможности для создания очень выразительных и мощных систем типов. Эти типы позволяют создавать сложные манипуляции с типами во время компиляции.

## Основные концепции

### [Union and Intersection Types](union-intersection.md)
Объединения и пересечения для сложных типов

### [Literal Types](literal-types.md)
Использование конкретных значений в качестве типов

### [Type Guards](type-guards.md)
Специальные функции для уточнения типов

### [Discriminated Unions](discriminated-unions.md)
Объединения с общим дискриминирующим полем для безопасного анализа

### [Type Aliases](type-aliases.md)
Создание собственных именованных типов

## Расширенные концепции

### [Mapped Types](mapped-types.md)
Создание новых типов путем перебора и преобразования существующих

### [Conditional Types](conditional-types.md)
Типы, определяемые условными выражениями

### [Template Literal Types](template-literal-types.md)
Создание строковых типов с помощью шаблонных литералов

### [Indexed Access Types](indexed-access-types.md)
Доступ к типам свойств по их именам

### [Keyof Operator](keyof-operator.md)
Получение объединения строковых литералов всех публичных свойств типа

## Практические примеры

### Mapped Types для создания производных типов
```typescript
interface Person {
  name: string;
  age: number;
  email: string;
}

// Создание типа со всеми свойствами только для чтения
type ReadonlyPerson = {
  readonly [P in keyof Person]: Person[P]
};

// Создание типа с только необязательными свойствами
type PartialPerson = {
  [P in keyof Person]?: Person[P]
};

// Создание типа с определенными свойствами
type PickPerson = Pick<Person, 'name' | 'email'>;
```

### Conditional Types
```typescript
type TypeName<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<true>;    // "boolean"
```

### Discriminated Unions
```typescript
interface Bird {
  type: 'bird';
  wingspan: number;
}

interface Dog {
  type: 'dog';
  breed: string;
}

type Animal = Bird | Dog;

function handleAnimal(animal: Animal) {
  if (animal.type === 'bird') {
    // Здесь TypeScript знает, что это Bird
    console.log(animal.wingspan);
  } else {
    // Здесь TypeScript знает, что это Dog
    console.log(animal.breed);
  }
}
```

## Встроенные типы утилиты

### Основные типы утилиты
```typescript
// Partial<T> - делает все свойства необязательными
interface Todo {
  title: string;
  description: string;
}
type OptionalTodo = Partial<Todo>; // { title?: string; description?: string }

// Required<T> - делает все свойства обязательными
type RequiredTodo = Required<OptionalTodo>;

// Readonly<T> - делает все свойства только для чтения
type ImmutableTodo = Readonly<Todo>;

// Pick<T, K> - выбирает подмножество свойств
type TodoPreview = Pick<Todo, 'title'>; // { title: string }

// Omit<T, K> - исключает свойства
type TodoWithoutTitle = Omit<Todo, 'title'>; // { description: string }
```

### Продвинутые типы утилиты
```typescript
// Exclude<T, U> - исключает типы из T, которые присутствуют в U
type T0 = Exclude<"a" | "b" | "c", "a" | "c">; // "b"

// Extract<T, U> - извлекает типы из T, которые присутствуют в U
type T1 = Extract<"a" | "b" | "c", "a" | "c">; // "a" | "c"

// NonNullable<T> - исключает null и undefined
type T2 = NonNullable<string | number | undefined>; // string | number

// Parameters<T> - тип параметров функции
type T3 = Parameters<(s: string, n: number) => void>; // [string, number]

// ReturnType<T> - тип возвращаемого значения функции
type T4 = ReturnType<() => string>; // string
```

## Сложные паттерны

### Типы утилиты собственного производства
```typescript
// Создание типа для глубокого ReadOnly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P]
};

interface Nested {
  a: number;
  b: {
    c: string;
    d: boolean;
  }
}

type DeepReadonlyNested = DeepReadonly<Nested>;
// Теперь все вложенные объекты также только для чтения
```

### Template Literal Types
```typescript
// Создание строкового объединения из объединения литералов
type Color = 'red' | 'blue';
type Quantity = 'one' | 'two';

type SeussFish = `${Quantity | Cap}${Color}`;
// "oneRed" | "oneBlue" | "twoRed" | "twoBlue"

// Функция для изменения регистра свойств
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface Person {
  name: string;
  age: number;
}

type GetterPerson = Getters<Person>;
// { getName: () => string; getAge: () => number }
```

## Лучшие практики

1. Используйте продвинутые типы осознанно - они могут усложнить код
2. Начинайте с встроенных типов утилиты
3. Документируйте сложные типы-утилиты
4. Тестируйте сужение типов в объединениях
5. Используйте дискриминированные объединения для безопасной обработки данных

## Связь с другими концепциями
- [[TS Generics]] - продвинутые типы часто используют обобщения
- [[type-system/type-guards]] - сужение типов в объединениях
- [[generics/constraints]] - ограничения в условных типах
- [[types/type-aliases]] - создание сложных именованных типов
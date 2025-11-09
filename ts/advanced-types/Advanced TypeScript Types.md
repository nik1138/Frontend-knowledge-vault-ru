# Advanced TypeScript Types

Продвинутые типы в TypeScript предоставляют мощные возможности для создания выразительных и безопасных систем типов. Они позволяют моделировать сложные доменные области и обеспечивать строгую типизацию на этапе компиляции.

## Категории продвинутых типов

### [Conditional Types](conditional-types.md)
Типы, определяемые условными выражениями и позволяющие создавать типы, которые выбираются в зависимости от условий.

### [Mapped Types](mapped-types.md)  
Создание новых типов путем перебора и преобразования свойств существующего типа.

### [Template Literal Types](template-literal-types.md)
Создание строковых типов с помощью шаблонных литералов и преобразований регистров.

### [Discriminated Unions](discriminated-unions.md)
Объединения типов с общим дискриминирующим полем, позволяющие безопасно сужать типы.

### [Index Types](indexed-types.md)
Доступ к типам свойств по их именам с использованием оператора keyof и индексного доступа.

### [Utility Types](utility-types.md)
Встроенные типы-утилиты для распространенных преобразований типов.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Advanced TypeScript Types                        │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                             ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │  Conditional    │               │      Mapped              │                                    │ Template        │
  │  Types          │               │  Types                 │                                    │ Literal         │
  │                 │               │                          │                                    │ Types           │
  │ - Conditional   │◄──────────────┤ - Property Mapping       │─────────────────────────────────────►│ - String        │
  │   Expressions   │               │ - Key Remapping        │                                    │   Manipulation  │
  │ - Distributive  │               │ - Type Transformation    │                                    │ - Case          │
  │   Conditionals  │               │ - Utility Types          │                                    │   Transformation│
  │ - Infer         │               │ - Inference              │                                    │ - Pattern       │
  │ - Conditional   │               │ - Constraint             │                                    │   Matching      │
  │   Assignments   │               │ - Variance              │                                    │ - Type-Safe     │
  │ - Conditional   │               │ - Advanced Patterns      │                                    │   DSLs          │
  │   Functions     │               │ - Deep Transformations   │                                    │ - URL Patterns  │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │       Discriminated           │
        │                           │       Unions                  │
        │                           │                               │
        │                           │ - Discriminant Properties       │
        │                           │ - Exhaustiveness Checking       │
        │                           │ - Switch Statements             │
        │                           │ - Nested Discrimination         │
        │                           │ - Tagged Unions                 │
        │                           │ - Runtime Validation            │
        │                           │ - Serialization                 │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                          Index Types                              │
        │                           │                                                                 │
        │                           │ - keyof Operator                                                    │
        │                           │ - Indexed Access Types                                              │
        │                           │ - Lookup Types                                                     │
        │                           │ - Conditional Lookups                                               │
        │                           │ - Type Queries                                                     │
        │                           │ - Reflection-like Behavior                                          │
        │                           │ - Runtime Type Information                                          │
        │                           │ - Type-Safe Property Access                                         │
        └───────────────────────────┤ - Key-Based Transformations                                         │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Conditional Types

### Базовый синтаксис
```typescript
type TypeName<T> = 
  T extends string ? "string" :
  T extends number ? "number" :
  T extends boolean ? "boolean" :
  T extends undefined ? "undefined" :
  "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<"a">;     // "string" (литеральный строковый тип)
type T2 = TypeName<true>;    // "boolean"
type T3 = TypeName<() => void>;  // "object"
type T4 = TypeName<string[]>;    // "object"
```

### Распределительные условные типы
```typescript
// Условный тип распределяется по объединениям
type ToArray<T> = T extends any ? T[] : never;

type T0 = ToArray<string | number>;  // string[] | number[]

// Это эквивалентно:
// string extends any ? string[] : never | number extends any ? number[] : never
// string[] | number[]
```

### Infer оператор
```typescript
// Извлечение типов из других типов
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Num = GetReturnType<() => number>; // number
type Str = GetReturnType<(x: string) => string>; // string

// Извлечение типов из обобщений
type GetFirst<T> = T extends [infer First, ...any[]] ? First : never;
type First = GetFirst<[string, number, boolean]>; // string

// Извлечение типов из объединений
type Flatten<T> = T extends Array<infer Item> ? Item : T;
type Flat = Flatten<string[]>; // string
```

## Mapped Types

### Базовые сопоставляющие типы
```typescript
// Создание типа с измененными свойствами
type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

// Пример использования
interface User {
  id: number;
  name: string;
  email: string;
}

type OptionalUser = Partial<User>; // все свойства необязательны
type WritableUser = Mutable<Readonly<User>>; // все свойства доступны для записи
```

### Перемещение ключей
```typescript
// Изменение ключей с помощью as
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
// }
```

## Template Literal Types

### Строковые операции
```typescript
// Преобразование регистра
type T0 = Uppercase<"hello">;  // "HELLO"
type T1 = Lowercase<"HELLO">;  // "hello"
type T2 = Capitalize<"hello">; // "Hello"
type T3 = Uncapitalize<"Hello">; // "hello"

// Комбинация с маппингом
type EventName = `on${Capitalize<string>}`;
type HandlerName = `handle${Capitalize<string>}`;

type ClickEvent = "click";
type OnClick = `on${Capitalize<ClickEvent>}`; // "onClick"
```

## Discriminated Unions

### Защита типов
```typescript
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
  // TypeScript может сузить тип благодаря дискриминанту
  switch (state.status) {
    case "loading":
      // Здесь state - LoadingState
      console.log(`Loading... ${state.progress ?? 0}%`);
      break;
    case "success":
      // Здесь state - SuccessState
      console.log(`Success: ${state.data}`);
      break;
    case "error":
      // Здесь state - ErrorState
      console.log(`Error: ${state.error}`);
      break;
  }
}
```

## Index Types

### Keyof оператор
```typescript
type Point = { x: number; y: number };
type PointKeys = keyof Point; // "x" | "y"

type ArrayKeys = keyof string[]; // "length" | "push" | "pop" | ... (все методы массива)

// Использование с обобщениями
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const x = getProperty({ x: 1, y: 2 }, "x"); // number
```

### Indexed Access Types
```typescript
type Data = { a: string; b: number; c: boolean };
type A = Data["a"]; // string
type AB = Data["a" | "b"]; // string | number
type All = Data[keyof Data]; // string | number | boolean

// Глубокий доступ
type Response = {
  data: {
    users: {
      id: number;
      name: string;
    }[];
  };
};

type UserName = Response["data"]["users"][number]["name"]; // string
```

## Utility Types

### Встроенные утилиты
```typescript
// Основные утилиты
type Partial<T> = { [P in keyof T]?: T[P] };
type Required<T> = { [P in keyof T]-?: T[P] };
type Readonly<T> = { readonly [P in keyof T]: T[P] };

type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;

type NonNullable<T> = T extends null | undefined ? never : T;
type Parameters<T> = T extends (...args: infer P) => any ? P : never;
type ReturnType<T> = T extends (...args: any) => infer R ? R : any;
```

### Создание собственных утилит
```typescript
// Глубокое преобразование в необязательные поля
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Глубокая неизменяемость
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Типы только для чтения (без методов)
type DataOnly<T> = {
  [P in keyof T as T[P] extends Function ? never : P]: T[P]
};
```

## Практические примеры

### Создание типобезопасного API
```typescript
// API с дискриминированными объединениями
interface CreateAction {
  type: 'create';
  entity: string;
  data: Record<string, any>;
}

interface UpdateAction {
  type: 'update';
  entity: string;
  id: number;
  updates: Partial<Record<string, any>>;
}

interface DeleteAction {
  type: 'delete';
  entity: string;
  id: number;
}

type Action = CreateAction | UpdateAction | DeleteAction;

function handleAction(action: Action) {
  switch (action.type) {
    case 'create':
      // action.entity, action.data
      console.log(`Creating ${action.entity}`, action.data);
      break;
    case 'update':
      // action.entity, action.id, action.updates
      console.log(`Updating ${action.entity} #${action.id}`, action.updates);
      break;
    case 'delete':
      // action.entity, action.id
      console.log(`Deleting ${action.entity} #${action.id}`);
      break;
  }
}
```

### Типобезопасная работа с API
```typescript
// Тип для описания структуры API ответа
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: number;
    message: string;
  };
}

type SuccessResponse<T> = ApiResponse<T> & { success: true; data: T };
type ErrorResponse = ApiResponse<never> & { success: false; error: { code: number; message: string } };

// Использование условных типов для определения типа возврата
type ApiCallResult<T> = Promise<SuccessResponse<T> | ErrorResponse>;

// Тип для извлечения данных из успешного ответа
type GetDataFromSuccess<T> = T extends SuccessResponse<infer Data> ? Data : never;

// Утилита для проверки успешного ответа
type IsSuccess<T> = T extends SuccessResponse<any> ? true : false;
```

## Продвинутые паттерны

### Глубокие трансформации
```typescript
// Глубокое удаление определенных типов
type FilterOut<T, F> = T extends F ? never : T;

// Глубокое преобразование функций
type FunctionResult<T> = T extends (...args: infer Args) => infer Return 
  ? (...args: Args) => Promise<Return> 
  : T;

// Тип для преобразования всех функций в асинхронные
type Asyncify<T> = {
  [K in keyof T]: T[K] extends (...args: infer Args) => infer Return
    ? (...args: Args) => Promise<Awaited<Return>>
    : T[K]
};
```

### Рекурсивные сопоставляющие типы
```typescript
// Глубокое преобразование с сохранением структуры
type DeepTransform<T, F extends (value: any) => any> = T extends object
  ? {
      [K in keyof T]: T[K] extends object 
        ? T[K] extends Function 
          ? F extends (value: T[K]) => infer R ? R : T[K]
          : DeepTransform<T[K], F>
        : F extends (value: T[K]) => infer R ? R : T[K]
    }
  : F extends (value: T) => infer R ? R : T;

// Пример использования
type StringifyValues<T> = DeepTransform<T, (value: any) => string>;

interface User {
  name: string;
  age: number;
  profile: {
    hobby: string;
    preferences: {
      theme: 'light' | 'dark';
      notifications: boolean;
    };
  };
}

type StringifiedUser = StringifyValues<User>;
// Все значения будут строками, сохраняя структуру
```

## Лучшие практики

### 1. Использование infer для извлечения типов
```typescript
// Правильно извлекать типы из сложных структур
type GetPromiseType<T> = T extends Promise<infer U> ? U : T;
type GetArrayType<T> = T extends (infer U)[] ? U : T;
type GetFunctionReturn<T> = T extends (...args: any[]) => infer U ? U : never;
```

### 2. Ограничение сложности условных типов
```typescript
// Не делать условные типы слишком сложными
// Это может замедлить компиляцию
type VeryComplex<T> = T extends SomeVeryLongAndComplexCondition
  ? VeryComplexBranch1<T>
  : T extends AnotherComplexCondition
    ? VeryComplexBranch2<T>
    : T extends YetAnotherCondition
      ? VeryComplexBranch3<T>
      : SimpleFallback<T>;
```

### 3. Использование встроенных утилит
```typescript
// Предпочитайте встроенные утилиты собственным реализациям
// Вместо создания собственного Partial<T>:
type MyPartial<T> = { [K in keyof T]?: T[K] }; // избегайте

// Используйте встроенное:
type BetterPartial<T> = Partial<T>; // предпочитайте
```

## Ограничения и подводные камни

### 1. Производительность
```typescript
// Слишком сложные рекурсивные типы могут замедлить компиляцию
// type VeryDeepTransform<T> = { [K in keyof T]: VeryDeepTransform<T[K]> }; // может быть медленным
```

### 2. Ограничения inference
```typescript
// Infer не работает во всех контекстах
type BrokenInfer<T> = T extends { [K in keyof T]: infer U } ? U : T; // не работает
type ProperInfer<T> = T extends { a: infer U } ? U : T; // работает
```

## Современные тенденции

### Типы-утилиты для фреймворков
```typescript
// Примеры из реальных библиотек
type Awaited<T> =
  T extends null | undefined ? T :
  T extends object & { then(onfulfilled: infer F): any } ? F extends (value: infer V) => any ? Awaited<V> : never :
  T;

// Тип для получения типа аргумента функции
type ArgumentTypes<F extends Function> = F extends (...args: infer A) => any ? A : never;

// Тип для получения типа последнего аргумента
type LastArgument<F extends Function> = 
  ArgumentTypes<F> extends [...any[], infer Last] ? Last : never;
```

### Совместимость с новыми версиями TypeScript
```typescript
// Для новых версий TypeScript появляются новые возможности
// Satisfies operator (TS 4.9+)
type Config = { hostname: string; port: number };
const config = {
  hostname: "localhost",
  port: 3000,
  debug: true // доп. св-во разрешено
} satisfies Config; // type-check для обязательных полей
```

## Сравнение подходов

| Тип | Когда использовать | Преимущества | Недостатки |
|-----|-------------------|-------------|------------|
| Conditional | Когда выбор типа зависит от условий во время компиляции | Гибкость, безопасность типов | Сложность, производительность |
| Mapped | Для преобразования всех свойств типа | Мощность, переиспользуемость | Сложность понимания |
| Template Literal | Для строковых типов и DSL | Точная типизация строк | Ограничения в сложных строках |
| Discriminated Unions | Для безопасной работы с объединениями | Безопасность, исчерпывающая проверка | Необходимость общего поля |
| Index Types | Для динамического доступа к свойствам | Безопасность при динамическом доступе | Сложнее для новичков |

## Связь с другими концепциями
- [[../type-system/type-inference]] - как продвинутые типы влияют на вывод
- [[../generics/advanced]] - сложные обобщения с продвинутыми типами
- [[../functional-programming/advanced]] - функциональные альтернативы
- [[../modules/type-safety]] - безопасность типов в модулях
- [[../ecosystem/tools]] - инструменты для работы с продвинутыми типами
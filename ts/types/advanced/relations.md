# Связи между типами TypeScript

## Архитектурная схема связей типов

```
                    ┌─────────────────────────────────────────────────────────┐
                    │                  TypeScript Types                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────┐
        ▼                                     ▼                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                    ┌─────────────────┐
  │ Primitive Types │               │     Literal Types        │                    │  Union Types    │
  │                 │               │                          │                    │                 │
  │ - string        │◄──────────────┤ - String Literals        │───────────────────►│ - A | B         │
  │ - number        │               │ - Number Literals        │                    │ - Type Narrowing│
  │ - boolean       │               │ - Boolean Literals       │                    │ - Discriminated │
  │ - symbol        │               │ - Template Literals      │                    │   Unions        │
  │ - bigint        │               │ - Type Transformations   │                    │ - Exhaustiveness│
  │ - null          │               │ - Case Manipulation      │                    │   Checks        │
  │ - undefined     │               └─────────────────┬────────┘                    └─────────────────┘
  └─────────────────┘                                 │
        │                                             │
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │    Intersection Types           │
        │                           │                                 │
        │                           │ - A & B (Combine Properties)    │
        │                           │ - Type Composition              │
        │                           │ - Mixins Pattern                │
        │                           │ - Enhanced Types                │
        │                           └─────────────────┬───────────────┘
        │                                             │
        │                           ┌─────────────────▼───────────────┐
        │                           │        Special Types          │
        │                           │                                 │
        │                           │ - any (Escape Hatch)            │
        │                           │ - unknown (Safe any)            │
        │                           │ - never (Bottom Type)           │
        │                           │ - void (No Return)              │
        │                           │ - null/undefined (Special)      │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────┐
        │                           │                    Type Aliases                               │
        │                           │                                                               │
        │                           │ - Custom Type Names                                         │
        │                           │ - Generic Type Aliases                                        │
        │                           │ - Complex Type Abstractions                                   │
        │                           │ - Interface vs Type Alias Comparison                          │
        │                           └───────────────────────────────────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────┐
        │                           │                   Object Types                                │
        │                           │                                                               │
        │                           │ - Interfaces & Objects                                        │
        │                           │ - Index Signatures                                            │
        │                           │ - Function Types in Objects                                   │
        │                           │ - Optional & Readonly Properties                              │
        └───────────────────────────┤ - Getters & Setters                                           │
                                    └───────────────────────────────────────────────────────────────┘
```

## Подробные связи между типами

### 1. Primitive ↔ Literal Types
```typescript
// Литералы расширяют примитивы конкретными значениями
type Status = "active" | "inactive";  // строковый литерал из string
type Count = 1 | 2 | 3;              // числовой литерал из number
type Yes = true;                     // булевый литерал из boolean

// Примитивы как супертипы литералов
function processStatus(status: "active" | "inactive"): string {
  // status также является string (супертип)
  return status.toUpperCase();
}
```

### 2. Literal ↔ Template Literal Types
```typescript
// Шаблонные литералы используют примитивные литералы
type HttpMethod = "GET" | "POST";
type Resource = "users" | "posts";
type Endpoint = `${HttpMethod}/${Resource}`; // "GET/users" | "GET/posts" | "POST/users" | "POST/posts"

// Преобразования регистров
type EventName = `on${Capitalize<"click" | "hover">}`; // "onClick" | "onHover"
```

### 3. Union ↔ Intersection Types
```typescript
// Объединения и пересечения решают разные задачи
interface A { a: string; }
interface B { b: number; }
interface C { a: string; b: number; c: boolean; }

// Объединение: значение может быть A ИЛИ B
type UnionType = A | B; // { a: string; } | { b: number; }

// Пересечение: значение должно быть A И B  
type IntersectionType = A & B; // { a: string; b: number; }

// Практический пример различий
const unionValue: UnionType = { a: "hello" }; // OK
const intersectionValue: IntersectionType = { a: "hello", b: 42 }; // OK

// Сужение типов в объединениях не работает в пересечениях
function handleUnion(union: UnionType) {
  if ('a' in union) { // Проверка свойства для сужения типа
    union.a; // OK: это A
  }
  // else: это B
}
```

### 4. Special Types ↔ All Other Types
```typescript
// unknown - супертип всех типов
function processUnknown(value: unknown) {
  // value может быть чем угодно
  if (typeof value === "string") {
    // теперь value имеет тип string
    return value.toUpperCase();
  }
  return "";
}

// never - подтип всех типов
function createNever(): never {
  throw new Error("Always throws");
}

// never может быть присвоено любому типу
let str: string = createNever(); // OK
let num: number = createNever(); // OK

// any отключает проверку типов полностью
function handleAny(value: any) {
  return value.something.arbitrary(); // Никаких проверок типов
}
```

### 5. Type Aliases ↔ All Type Categories
```typescript
// Псевдонимы могут объединять все типы
type ComplexAlias = {
  // примитивы
  name: string;
  age: number;
  
  // литералы
  status: "active" | "inactive";
  
  // объединения
  id: string | number;
  
  // пересечения
  config: { debug: boolean; } & { theme: "light" | "dark"; };
  
  // специальные типы
  callback: (() => void) | null;
  
  // объектные типы
  address: {
    street: string;
    city: string;
  };
} | [string, number, boolean]; // или кортеж
```

## Интеграционные паттерны

### 1. Дискриминированные объединения
```typescript
// Объединения + литералы + сужение типов
interface Loading { type: "loading"; progress?: number; }
interface Success<T> { type: "success"; data: T; }
interface Error { type: "error"; message: string; }

type ApiResponse<T> = Loading | Success<T> | Error;

function handleResponse<T>(response: ApiResponse<T>) {
  // TypeScript использует литеральный тип 'type' как дискриминант
  switch (response.type) {
    case "loading":
      // response автоматически имеет тип Loading
      console.log(response.progress || 0);
      break;
    case "success":
      // response автоматически имеет тип Success<T>
      console.log(response.data);
      break;
    case "error":
      // response автоматически имеет тип Error
      console.log(response.message);
      break;
  }
}
```

### 2. Сопоставляющие типы с литералами
```typescript
// Пересечения + шаблонные литералы + сопоставляющие типы
type CreateGetters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface User {
  name: string;
  age: number;
}

type UserGetters = CreateGetters<User>;
// {
//   getName: () => string;
//   getAge: () => number;
// }
```

### 3. Глубокое преобразование типов
```typescript
// Комбинация: пересечение, рекурсия, условные типы, литералы
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : DeepReadonly<T[K]> 
    : T[K]
};

type ComplexObject = {
  user: {
    name: string;
    preferences: {
      theme: "light" | "dark";
      notifications: boolean;
    };
  };
  items: string[];
};

type ReadonlyComplex = DeepReadonly<ComplexObject>;
// Все свойства становятся readonly рекурсивно
```

## Практические интеграционные примеры

### 1. Функциональный пайплайн с типизацией
```typescript
// Объединения + специальные типы + псевдонимы
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

type AsyncOperation<T> = () => Promise<T>;

type PipelineStep<T, U> = (input: T) => U | Promise<U>;

async function pipe<T, U>(
  initial: T | Promise<T>,
  ...steps: PipelineStep<any, any>[]
): Promise<Result<U>> {
  try {
    const result = await steps.reduce(
      async (acc, step) => step(await acc),
      Promise.resolve(initial) as Promise<any>
    );
    return { success: true, data: result };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}
```

### 2. Типизированная система событий
```typescript
// Литералы + шаблонные литералы + объединения + условные типы
type EventType = "user" | "system" | "data";
type Action = "login" | "logout" | "update" | "error";

type EventName = `${EventType}:${Action}`;
// "user:login" | "user:logout" | "user:update" | "user:error" | ...

type EventData<T extends EventName> = 
  T extends "user:login" ? { userId: number; timestamp: Date; } :
  T extends "user:error" ? { error: string; context?: any; } :
  T extends "system:update" ? { version: string; changes: string[]; } :
  Record<string, any>;

interface EventBus {
  emit<T extends EventName>(event: T, data: EventData<T>): void;
  on<T extends EventName>(event: T, handler: (data: EventData<T>) => void): void;
}
```

### 3. Система валидации с типами
```typescript
// Объединения + литералы + пересечения + псевдонимы
type ValidationRule = "required" | "minLength" | "maxLength" | "pattern";
type FieldName = "username" | "email" | "password";

type ValidationError = `${FieldName}.${ValidationRule}`;
// "username.required" | "username.minLength" | ...

interface ValidationErrors {
  [key: ValidationError]: string;
}

type Validator<T> = (value: T) => boolean;

type FieldValidator<T> = {
  [K in keyof T]: Validator<T[K]>[];
};

type ValidationResult<T> = {
  valid: boolean;
  errors: Partial<Record<keyof T, string[]>>;
  data: T;
};
```

## Лучшие практики интеграции

### 1. Порядок применения типов
1. **Начинайте с литеральных типов** для ограничений
2. **Используйте объединения** для альтернатив
3. **Применяйте пересечения** для комбинации функциональности
4. **Добавьте специальные типы** для крайних случаев
5. **Создавайте псевдонимы** для повторного использования

### 2. Оптимизация производительности
```typescript
// Избегайте чрезмерной вложенности
// ПЛОХО: слишком сложное объединение
type OverlyComplex = 
  (A & B) | 
  (C & D & E) | 
  ((F | G) & H) | 
  ...

// ЛУЧШЕ: разбиение на именованные компоненты
type Component1 = A & B;
type Component2 = C & D & E;  
type Component3 = (F | G) & H;
type FinalType = Component1 | Component2 | Component3;
```

### 3. Документирование сложных интеграций
```typescript
/**
 * Represents a validated form field
 * - Uses literal types to restrict possible validation states
 * - Uses union types to represent success/error states  
 * - Uses intersection types to combine validation rules with data
 */
type ValidatedField<T> = {
  value: T;
  errors: string[];
  isValid: boolean;
} & {
  [K in "required" | "minLength" | "maxLength" as `check${Capitalize<K>}`]: (value: T) => boolean;
};
```

## Связь с другими концепциями TypeScript

- **[[../type-system/type-inference]]** - как TypeScript выводит сложные интегрированные типы
- **[[../type-system/type-compatibility]]** - совместимость между объединенными типами
- **[[../advanced-types/conditional-types]]** - условная типизация в интеграциях
- **[[../generics/index]]** - обобщенные интеграции типов
- **[[../interfaces-classes/index]]** - объектно-ориентированные интеграции
- **[[../modules/index]]** - организация типов в модулях
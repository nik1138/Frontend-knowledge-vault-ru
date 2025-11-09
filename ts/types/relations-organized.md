# Связи между типами TypeScript

## Архитектурная схема организованных типов

```
                    ┌─────────────────────────────────────────────────────────┐
                    │                  TypeScript Types                       │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────┐
        ▼                                     ▼                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                    ┌─────────────────┐
  │   Basics        │               │      Objects             │                    │  Advanced       │
  │                 │               │                          │                    │                 │
  │ - Primitive     │◄──────────────┤ - Object Types           │───────────────────►│ - Special Types │
  │   Types         │               │ - Union Types            │                    │ - Template      │
  │ - Literal       │               │ - Intersection Types     │                    │   Literals      │
  │   Types         │               │                          │                    │ - Type Aliases  │
  │                 │               │                          │                    │ - Relations     │
  └─────────────────┘               └─────────────────┬────────┘                    │ - Overview      │
        │                                             │                             └─────────────────┘
        │                                             │
        └─────────────────────────────────────────────┼─────────────────────────────────────────────────┘
                                                      │
                                    ┌─────────────────▼─────────────────┐
                                    │        Type System Features       │
                                    │                                   │
                                    │ - Type Inference & Narrowing      │
                                    │ - Structural Typing &             │
                                    │   Compatibility                   │
                                    │ - Generics & Constraints          │
                                    │ - Module Integration              │
                                    └───────────────────────────────────┘
```

## Подробные связи между категориями

### 1. Basics ↔ Objects
```typescript
// Литералы → Объединения
type Status = "active" | "inactive";  // литералы в объединении
let userStatus: Status = "active";

// Примитивы → Объектные свойства
interface User {
  name: string;      // string примитив
  age: number;       // number примитив
  isActive: boolean; // boolean примитив
  status: Status;    // литеральный тип
}
```

### 2. Objects ↔ Advanced
```typescript
// Объединения + специальные типы для безопасных API
type ApiResponse<T> = 
  | { success: true; data: T }           // пересечение объектов
  | { success: false; error: string };   // объединение с литералом

// Шаблонные литералы + объекты
type EventHandlers<T> = {
  [K in keyof T as `on${Capitalize<string & K>}Change`]: (value: T[K]) => void;
};

interface FormState {
  username: string;
  password: string;
}

type FormEventHandlers = EventHandlers<FormState>;
// {
//   onUsernameChange: (value: string) => void;
//   onPasswordChange: (value: string) => void;
// }
```

### 3. Basics ↔ Advanced
```typescript
// Литералы → Шаблонные литералы
type HttpMethod = "GET" | "POST";           // литерал
type Endpoint = `/${HttpMethod}/users`;     // шаблонный литерал

// Примитивы → Псевдонимы
type ID = number | string;                  // объединение примитивов
type Timestamp = number;                    // псевдоним для примитива
type Entity<T> = { id: ID; createdAt: Timestamp; data: T }; // комбинация
```

## Интеграционные паттерны

### 1. Дискриминированные объединения (Basics + Objects)
```typescript
// Литеральные типы как дискриминанты в объединениях объектов
interface Loading { type: "loading"; progress?: number; }
interface Success<T> { type: "success"; data: T; }
interface Error { type: "error"; message: string; }

type ApiResponse<T> = Loading | Success<T> | Error;  // объединение объектов с литералами

function handleResponse<T>(response: ApiResponse<T>) {
  switch (response.type) {  // литеральный тип как дискриминант
    case "loading":
      console.log(response.progress || 0);
      break;
    case "success":
      console.log(response.data);  // тип данных автоматически определен
      break;
    case "error":
      console.log(response.message);  // тип ошибки автоматически определен
      break;
  }
}
```

### 2. Сопоставляющие типы с преобразованиями (Objects + Advanced)
```typescript
// Пересечения + шаблонные литералы + сопоставляющие типы
type CreateGetters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

type CreateSetters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void
};

type EnhancedObject<T> = CreateGetters<T> & CreateSetters<T>;

interface User {
  name: string;
  age: number;
}

type UserAccessors = EnhancedObject<User>;
// {
//   getName: () => string;  setName: (value: string) => void;
//   getAge:  () => number;  setAge:  (value: number) => void;
// }
```

### 3. Глубокие преобразования (Все категории)
```typescript
// Комплексное использование всех категорий
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : DeepReadonly<T[K]>  // рекурсия с проверкой объекта
    : T[K]  // примитивы остаются как есть
};

type EntityId = string & { readonly brand: unique symbol };  // литерал + псевдоним
type Timestamp = number;  // псевдоним

interface User {
  id: EntityId;
  name: string;
  profile: {
    age: number;
    settings: {
      theme: "light" | "dark";  // литеральный тип
      notifications: boolean;
    };
  };
}

type ReadonlyUser = DeepReadonly<User>;
// Все вложенные объекты становятся readonly
```

## Практические применения

### 1. Система валидации (Все категории)
```typescript
// Комплексная система валидации с использованием всех концепций
type ValidationRule = "required" | "minLength" | "maxLength" | "pattern";
type FieldName = "username" | "email" | "password";

type ValidationErrorPath = `${FieldName}.${ValidationRule}`;  // шаблонный литерал

interface ValidationResult<T> {
  data: T;
  errors: Partial<Record<ValidationErrorPath, string[]>>;  // пересечение: объект + индексная сигнатура
  isValid: boolean;
}

type Validator<T> = (value: T) => boolean;

type FieldValidators<T> = {
  [K in keyof T]: Validator<T[K]>[];  // сопоставляющий тип
};

type ValidateFn<T> = (data: T) => ValidationResult<T>;

// Пример использования
type User = {
  username: string;
  email: string;
  password: string;
};

const userValidators: FieldValidators<User> = {
  username: [
    (s) => s.length > 0,  // required
    (s) => s.length >= 3  // minLength
  ],
  email: [
    (s) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(s)  // pattern
  ],
  password: [
    (s) => s.length > 0,  // required
    (s) => s.length >= 8  // minLength
  ]
};
```

### 2. Типизированная система событий (Все категории)
```typescript
// Система событий с полной типизацией
type System = "user" | "auth" | "system";
type Action = "create" | "update" | "delete" | "login";

type EventName = `${System}:${Action}`;  // шаблонный литерал

type EventData<T extends EventName> = 
  T extends "user:create" ? { userId: string; userData: object; } :
  T extends "auth:login" ? { userId: string; timestamp: number; } :
  T extends "system:update" ? { version: string; changes: string[]; } :
  Record<string, any>;  // универсальный тип для остальных случаев

interface EventBus {
  // объединение в возвращаемом типе
  emit<T extends EventName>(event: T, data: EventData<T>): void;
  on<T extends EventName>(event: T, handler: (data: EventData<T>) => void): void;
}

// Псевдоним для упрощения использования
type TypedEventBus<SystemEvents extends string> = {
  [K in SystemEvents as `on${Capitalize<K>}`]: (handler: (data: any) => void) => void;
};
```

## Лучшие практики интеграции

### 1. Порядок проектирования типов
1. **Начинайте с примитивов и литералов** (basics) - определите основные значения
2. **Создавайте объектные структуры** (objects) - компонуйте данные
3. **Добавляйте продвинутую типизацию** (advanced) - безопасность и гибкость

### 2. Оптимизация производительности
```typescript
// Избегайте чрезмерной вложенности при интеграции типов
// ПЛОХО: слишком много уровней
type BadIntegration = DeepTransform<ComplexMapping<UnionOfIntersections<BasicLiterals>>>;

// ЛУЧШЕ: постепенное усложнение
type BaseFields = "name" | "id" | "type";
type FieldPaths = `${BaseFields}.${"input" | "output"}`;
type FieldMapping = Partial<Record<FieldPaths, string>>;
type ProcessedMapping = { [K in keyof FieldMapping]: FieldMapping[K][] };
```

### 3. Документирование сложных интеграций
```typescript
/**
 * Интеграция типов для валидации форм:
 * - Basics: литеральные типы для имен полей и правил
 * - Objects: объединения для различных состояний формы
 * - Advanced: шаблонные литералы для путей ошибок, сопоставляющие типы для валидаторов
 */
type FormValidationIntegration<FormData> = {
  data: FormData;
  errors: Partial<Record<`${keyof FormData & string}.${ValidationRule}`, string[]>>;
  validators: { [K in keyof FormData]: ((value: FormData[K]) => boolean)[] };
  isValid: boolean;
};
```

## Эволюция концепций в проекте

### 1. Начальный этап (Basics)
- Примитивные типы для простых значений
- Литеральные типы для ограниченных наборов значений

### 2. Рост сложности (Objects)  
- Объектные типы для структур данных
- Объединения для альтернативных состояний
- Пересечения для композиции функциональности

### 3. Зрелая система (Advanced)
- Специальные типы для безопасности
- Шаблонные литералы для динамических имен
- Псевдонимы для абстракции сложности
- Сопоставляющие типы для преобразований

## Связь с остальной системой TypeScript

- **[[../type-system/type-inference]]** - как система выводит типы из комбинаций
- **[[../type-system/type-compatibility]]** - совместимость между интегрированными типами  
- **[[../advanced-types/conditional-types]]** - условная типизация в интеграциях
- **[[TS Generics]]** - обобщения для универсальных интеграций
- **[[TS Interfaces and Classes]]** - объектно-ориентированные интеграции
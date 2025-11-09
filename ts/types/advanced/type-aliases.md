# Type Aliases

Псевдонимы типов (Type Aliases) позволяют создавать новые имена для типов. Это удобный способ для создания более читаемых и переиспользуемых описаний типов, особенно для сложных типов или объединений.

## Основы псевдонимов типов

### Простые псевдонимы
```typescript
// Простой псевдоним для примитивного типа
type Name = string;

let username: Name = "Alice";

// Псевдоним для объединения типов
type ID = number | string;

function findUser(id: ID): string {
  return typeof id === "string" ? id.toUpperCase() : `ID_${id}`;
}

// Псевдоним для сложного типа
type Callback = (error: Error | null, result?: any) => void;

let callback: Callback = (err, result) => {
  if (err) {
    console.error(err.message);
  } else {
    console.log(result);
  }
};
```

### Псевдонимы для объектных типов
```typescript
// Псевдоним для сложного объектного типа
type User = {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
};

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  isActive: true
};

// Вложенные псевдонимы
type Address = {
  street: string;
  city: string;
  zipCode: string;
};

type DetailedUser = User & {
  address: Address;
  preferences: {
    theme: "light" | "dark";
    notifications: boolean;
  };
};
```

## Псевдонимы с обобщениями

### Обобщенные псевдонимы
```typescript
// Обобщенный псевдоним
type Container<T> = { value: T };

const stringContainer: Container<string> = { value: "hello" };
const numberContainer: Container<number> = { value: 42 };

// Сложный обобщенный псевдоним
type Response<T, E = Error> = {
  success: true;
  data: T;
} | {
  success: false;
  error: E;
};

type ApiResponse<T> = Response<T, { message: string; code: number }>;

const successResponse: ApiResponse<string> = {
  success: true,
  data: "Operation completed"
};

const errorResponse: ApiResponse<string> = {
  success: false,
  error: { message: "Something went wrong", code: 500 }
};
```

### Псевдонимы с ограничениями
```typescript
// Псевдоним с ограничением обобщения
type FilteredArray<T extends { id: number }> = T[];

interface Entity {
  id: number;
  name: string;
}

interface Person extends Entity {
  age: number;
}

const entities: FilteredArray<Entity> = [
  { id: 1, name: "Entity 1" },
  { id: 2, name: "Entity 2" }
];

const people: FilteredArray<Person> = [
  { id: 1, name: "Alice", age: 30 },
  { id: 2, name: "Bob", age: 25 }
];
```

## Практические применения

### Псевдонимы для конфигурации
```typescript
// Псевдонимы для сложных конфигураций
type Environment = "development" | "staging" | "production";

type LogLevel = "debug" | "info" | "warn" | "error";

type DatabaseConfig = {
  host: string;
  port: number;
  name: string;
  username: string;
  password: string;
  ssl: boolean;
};

type AppConfig = {
  environment: Environment;
  logLevel: LogLevel;
  database: DatabaseConfig;
  features: {
    [key: string]: boolean;
  };
  timeouts: {
    api: number;
    db: number;
    socket: number;
  };
};

const config: AppConfig = {
  environment: "development",
  logLevel: "debug",
  database: {
    host: "localhost",
    port: 5432,
    name: "myapp",
    username: "user",
    password: "pass",
    ssl: false
  },
  features: {
    newUI: true,
    analytics: false
  },
  timeouts: {
    api: 5000,
    db: 3000,
    socket: 10000
  }
};
```

### Псевдонимы для функций
```typescript
// Псевдонимы для сложных сигнатур функций
type AsyncOperation<T> = () => Promise<T>;

type Middleware<T> = (input: T, next: (value: T) => T) => T;

type EventListener<T> = (event: T) => void;

// Складываемые псевдонимы
type AsyncMiddleware<T> = (input: T, next: AsyncOperation<T>) => Promise<T>;

// Использование
const asyncOp: AsyncOperation<string> = async () => {
  // какая-то асинхронная операция
  return "result";
};

const logger: Middleware<string> = (input, next) => {
  console.log("Input:", input);
  return next(input);
};
```

## Сравнение с интерфейсами

### Когда использовать псевдонимы vs интерфейсы
```typescript
// Псевдонимы лучше для:
// 1. Примитивов и объединений
type Status = "active" | "inactive" | "pending";
type NumericId = number | string;

// 2. Кортежей
type HttpResponse = [number, string, Date];

// 3. Сложных типов, которые нельзя выразить интерфейсом
type JsonValue = string | number | boolean | null | JsonObject | JsonValue[];
interface JsonObject {
  [key: string]: JsonValue;
}

// Интерфейсы лучше для:
// 1. Расширения через наследование
interface BaseUser {
  id: number;
  name: string;
}

interface AdminUser extends BaseUser {
  permissions: string[];
}

// 2. Декларативного слияния
interface User {
  name: string;
}

interface User {
  age: number;
}

// Результат: { name: string; age: number; }

// Псевдонимы НЕ могут быть повторно объявлены
// type User = { name: string; }  // OK
// type User = { age: number; }   // ОШИБКА: нельзя объявить повторно
```

### Преобразование между интерфейсами и псевдонимами
```typescript
// Интерфейс в псевдоним
interface UserInterface {
  id: number;
  name: string;
}

type UserAlias = UserInterface;

// Псевдоним в интерфейс (ограниченно)
type UserObject = {
  id: number;
  name: string;
};

interface UserExtended extends UserObject {
  email: string;
}
```

## Продвинутые паттерны

### Псевдонимы с условными типами
```typescript
// Псевдонимы, использующие условные типы
type NonNullable<T> = T extends null | undefined ? never : T;

type DefinedUser = NonNullable<User | null>; // User

// Псевдонимы для извлечения типов
type GetReturnType<T extends (...args: any[]) => any> = T extends (...args: any[]) => infer R ? R : any;

type StringGetter = () => string;
type NumberGetter = () => number;

type StringReturn = GetReturnType<StringGetter>; // string
type NumberReturn = GetReturnType<NumberGetter>; // number
```

### Псевдонимы для сопоставляющих типов
```typescript
// Псевдоним для создания необязательных свойств
type Optional<T> = {
  [K in keyof T]?: T[K];
};

type OptionalUser = Optional<User>;

// Псевдоним для создания только для чтения свойств
type ReadOnly<T> = {
  readonly [K in keyof T]: T[K];
};

type ReadOnlyUser = ReadOnly<User>;

// Комбинированный псевдоним
type PartialReadonly<T> = {
  readonly [K in keyof T]?: T[K];
};

type PartialReadonlyUser = PartialReadonly<User>;
```

## Псевдонимы для функциональных паттернов

### Функторы и монады как псевдонимы
```typescript
// Типизированные функциональные паттерны
type Maybe<T> = T | null | undefined;

type Either<L, R> = { type: "Left"; value: L } | { type: "Right"; value: R };

type Functor<T> = {
  map<U>(fn: (value: T) => U): Functor<U>;
};

// Псевдоним для результата операции
type OperationResult<T> = Either<Error, T>;

function safeDivide(a: number, b: number): OperationResult<number> {
  if (b === 0) {
    return { type: "Left", value: new Error("Division by zero") };
  }
  return { type: "Right", value: a / b };
}
```

## Лучшие практики

### Именование псевдонимов
```typescript
// Хорошие имена для псевдонимов
type UserId = number;
type Email = string;
type Permissions = string[];
type UserSettings = { theme: string; language: string; };

// Соглашения об именовании
type ApiResponse<T> = { success: boolean; data?: T; error?: string; };
type EventHandler<T> = (event: T) => void;
type Predicate<T> = (value: T) => boolean;

// Избегайте абстрактных имен
// type Data = any;        // плохо
// type Info = string;     // плохо
// type Stuff = {};        // плохо
```

### Структурирование сложных типов
```typescript
// Разбиение сложных типов на более мелкие псевдонимы
type Timestamp = number;
type UUID = string;
type Role = "admin" | "user" | "moderator";

type AuditInfo = {
  createdAt: Timestamp;
  createdBy: UserId;
  updatedAt: Timestamp;
  updatedBy: UserId;
};

type UserPermissions = {
  roles: Role[];
  allowedActions: string[];
  restrictions: string[];
};

type CompleteUser = User & AuditInfo & UserPermissions;
```

## Ошибки и ловушки

### Рекурсивные псевдонимы
```typescript
// Осторожность с рекурсивными типами
type InfiniteRecursion = InfiniteRecursion[]; // ОШИБКА: бесконечная рекурсия

// Безопасная рекурсия для деревьев
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[];
};

// Или с Union
type JSONValue = 
  | string 
  | number 
  | boolean 
  | null 
  | JSONObject 
  | JSONValue[];

interface JSONObject {
  [key: string]: JSONValue;
}
```

### Слишком широкие псевдонимы
```typescript
// ПЛОХО: слишком широкий псевдоним
type Data = any;

// ЛУЧШЕ: специфический псевдоним
type UserData = { name: string; email: string; };

// ИДЕАЛЬНО: точно определенный псевдоним
type ValidatedUserData = {
  name: string;
  email: string;
  age: number;
  preferences: {
    theme: "light" | "dark";
    notifications: boolean;
  };
};
```

## Практические примеры

### Псевдонимы для API
```typescript
// Система типов для API
type HttpStatusCode = 200 | 201 | 400 | 401 | 404 | 500;

type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";

type ApiError = {
  code: HttpStatusCode;
  message: string;
  details?: string;
};

type ApiResponse<T> = {
  status: HttpStatusCode;
  data?: T;
  error?: ApiError;
};

type ApiEndpoint<T> = {
  method: HttpMethod;
  path: string;
  response: new () => T;
};

// Использование
type UserResponse = ApiResponse<{ id: number; name: string; }>;
```

### Псевдонимы для событий
```typescript
// Типизация системы событий
type EventName = 
  | `user:${"login" | "logout" | "update"}`
  | `file:${"upload" | "download" | "delete"}`
  | `system:${"start" | "stop" | "restart"}`;

type EventHandler<T extends EventName> = (data: EventData<T>) => void;

type EventData<T extends EventName> = 
  T extends "user:login" ? { userId: number; timestamp: Date } :
  T extends "file:upload" ? { filename: string; size: number } :
  T extends "system:stop" ? { reason?: string; timestamp: Date } :
  Record<string, any>;

// Псевдоним для системы событий
type EventSystem = {
  emit<T extends EventName>(event: T, data: EventData<T>): void;
  on<T extends EventName>(event: T, handler: EventHandler<T>): void;
};
```

## Связь с другими концепциями
- [[../interfaces-classes/interfaces]] - сравнение с интерфейсами
- [[union-types]] - часто используется с объединениями
- [[intersection-types]] - комбинирование типов
- [[../advanced-types/mapped-types]] - преобразование типов через псевдонимы
- [[literal-types]] - использование с литеральными типами
- [[../generics/index]] - обобщенные псевдонимы
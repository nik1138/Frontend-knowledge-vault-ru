# TypeScript Types Complete Overview

## Полное руководство по системе типов TypeScript

Система типов TypeScript - это многоуровневая иерархическая структура, обеспечивающая статическую типизацию, безопасность типов и мощные возможности для моделирования сложных доменных областей.

## Архитектурное понимание

```
                    ┌─────────────────────────────────────────────────────────┐
                    │                  TypeScript Type System                 │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────────────────────────────────────┐    ┌─────────────────┐
  │   Core Types    │               │                    Advanced Types                        │    │  Functional     │
  │                 │               │                                                          │    │   Patterns      │
  │ - Primitives    │               │ - Literal Types (String, Number, Boolean, Template)      │    │                 │
  │ - Objects       │               │ - Union Types (A | B) with Discrimination                │    │ - Type-Level    │
  │ - Functions     │               │ - Intersection Types (A & B) with Composition            │    │   Programming   │
  │ - Arrays        │               │ - Special Types (any, unknown, never, void)              │    │ - Type Safety   │
  │ - Tuples        │               │ - Type Aliases for Abstraction                           │    │ - Immutability  │
  └─────────────────┘               │ - Mapped & Conditional Types for Transformation          │    │ - Composition   │
        │                           │ - Utility Types for Common Operations                    │    └─────────────────┘
        │                           └─────────────────────────┬────────────────────────────────┘
        │                                                     │
        │                           ┌─────────────────────────▼────────────────────────────────┐
        │                           │                   Type System Features                   │
        │                           │                                                          │
        │                           │ - Type Inference & Narrowing                             │
        │                           │ - Structural Typing & Compatibility                      │
        │                           │ - Generics & Constraints                                   │
        │                           │ - Modules & Declaration Merging                          │
        │                           │ - Compile-time vs Runtime Considerations                 │
        │                           └──────────────────────────────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                          Ecosystem Integration                          │
        │                           │                                                                         │
        │                           │ - Compatibility with JavaScript (d.ts files, ambient types)           │
        │                           │ - Framework Integration (React, Vue, Angular, Node.js)                │
        │                           │ - API Design & Documentation                                        │
        │                           │ - Build Tools & Compilation (tsc, babel, webpack)                     │
        │                           │ - Runtime Validation & Error Handling                             │
        │                           └─────────────────────────────────────────────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                          Best Practices                                 │
        │                           │                                                                         │
        │                           │ - Type Safety vs Flexibility Balance                                    │
        │                           │ - Performance Considerations                                           │
        │                           │ - Maintainability & Code Organization                                   │
        │                           │ - Team Adoption & Learning Curve                                        │
        └───────────────────────────┤ - Error Prevention & Early Detection                                    │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Глубокая интеграция концепций

### 1. Модель данных с полной типизацией

```typescript
// Демонстрация интеграции всех концепций
type EntityId = string & { readonly brand: unique symbol };
type Timestamp = number;
type Status = "active" | "inactive" | "pending";

interface BaseEntity {
  id: EntityId;
  createdAt: Timestamp;
  updatedAt: Timestamp;
}

interface User extends BaseEntity {
  name: string;
  email: string;
  status: Status;
  profile?: {
    age: number;
    preferences: {
      theme: "light" | "dark";
      notifications: boolean;
    };
  };
}

type AdminUser = User & {
  permissions: string[];
  role: "admin";
};

type UserWithValidation = User & {
  validationErrors?: string[];
  isValid: boolean;
};

type ApiResponse<T> = 
  | { success: true; data: T; timestamp: Timestamp }
  | { success: false; error: string; code: number; timestamp: Timestamp };

type UserService = {
  getUser: (id: EntityId) => Promise<ApiResponse<User>>;
  createUser: (userData: Omit<User, keyof BaseEntity>) => Promise<ApiResponse<User>>;
  updateUser: (id: EntityId, updates: Partial<User>) => Promise<ApiResponse<User>>;
  validateUser: (user: User) => UserWithValidation;
};

// Псевдоним для сложного типа
type EnhancedUserService = UserService & {
  auditLog: (action: `${"create" | "update" | "delete"}_user`, userId: EntityId) => void;
  rateLimit: (maxRequests: number, windowMs: number) => void;
};
```

### 2. Система событий с полной типизацией

```typescript
// Интеграция: литералы, шаблонные литералы, объединения, условные типы
type System = "user" | "auth" | "payment" | "notification";
type Action = "create" | "update" | "delete" | "login" | "logout";
type EventType = `${System}:${Action}`;

type EventData<T extends EventType> = 
  T extends "user:create" ? { userId: EntityId; userData: User; } :
  T extends "auth:login" ? { userId: EntityId; timestamp: Timestamp; } :
  T extends "payment:create" ? { paymentId: EntityId; amount: number; userId: EntityId; } :
  Record<string, any>;

interface TypedEventBus {
  emit<T extends EventType>(event: T, data: EventData<T>): void;
  on<T extends EventType>(event: T, handler: (data: EventData<T>) => void): void;
  once<T extends EventType>(event: T, handler: (data: EventData<T>) => void): void;
  off<T extends EventType>(event: T, handler: (data: EventData<T>) => void): void;
}

// Использование
const eventBus: TypedEventBus = {
  // реализация
};

eventBus.emit("user:create", { 
  userId: "usr_123" as EntityId, 
  userData: { 
    id: "usr_123" as EntityId, 
    name: "Alice", 
    email: "alice@example.com", 
    status: "active",
    createdAt: Date.now(),
    updatedAt: Date.now()
  } 
});

// ОШИБКА: тип данных проверяется на соответствие eventType
// eventBus.emit("user:create", { wrongField: "value" });
```

### 3. Конфигурационная система

```typescript
// Интеграция: пересечения, литералы, псевдонимы, условные типы
type Environment = "development" | "staging" | "production";
type LogLevel = "debug" | "info" | "warn" | "error";

interface BaseConfig {
  environment: Environment;
  logLevel: LogLevel;
  timeout: number;
}

interface DevelopmentConfig extends BaseConfig {
  environment: "development";
  debugMode: true;
  hotReload: boolean;
  apiMocking: boolean;
}

interface ProductionConfig extends BaseConfig {
  environment: "production";
  debugMode: false;
  compression: boolean;
  caching: {
    enabled: true;
    ttl: number;
  };
}

type Config = DevelopmentConfig | ProductionConfig;

function createConfig(env: Environment): Config {
  const base: BaseConfig = {
    environment: env,
    logLevel: env === "production" ? "info" : "debug",
    timeout: env === "production" ? 5000 : 30000
  };

  if (env === "development") {
    return {
      ...base,
      environment: "development",
      debugMode: true,
      hotReload: true,
      apiMocking: true
    } as DevelopmentConfig;
  } else {
    return {
      ...base,
      environment: "production" as const,
      debugMode: false,
      compression: true,
      caching: {
        enabled: true,
        ttl: 3600
      }
    } as ProductionConfig;
  }
}
```

## Эволюция типизации

### 1. От простого к сложному

```typescript
// Уровень 1: Базовая типизация
let name: string = "Alice";
let age: number = 30;

// Уровень 2: Объединения и литералы
type Status = "active" | "inactive";
let status: Status = "active";

// Уровень 3: Интерфейсы и объекты
interface Person { name: string; age: number; status: Status; }
let person: Person = { name: "Alice", age: 30, status: "active" };

// Уровень 4: Обобщения
interface Container<T> { value: T; }
let personContainer: Container<Person> = { value: person };

// Уровень 5: Условные типы и сопоставления
type Optional<T> = { [K in keyof T]?: T[K] };
type OptionalPerson = Optional<Person>;

// Уровень 6: Сложные интеграции
type ValidatedField<T> = {
  value: T;
  errors: string[];
  isValid: boolean;
} & {
  [K in "required" | "minLength" | "maxLength" as `check${Capitalize<K>}`]: (value: T) => boolean;
};
```

### 2. Практические эволюционные пути

```typescript
// Эволюция API клиента: от простого к сложному
// Уровень 1: Простой клиент
function simpleGet(url: string): Promise<any> {
  return fetch(url).then(res => res.json());
}

// Уровень 2: Типизированный клиент
function typedGet<T>(url: string): Promise<T> {
  return fetch(url).then(res => res.json()) as Promise<T>;
}

// Уровень 3: Клиент с валидацией
type ApiResponse<T> = { success: true; data: T; } | { success: false; error: string; };
function validatedGet<T>(url: string): Promise<ApiResponse<T>> {
  return fetch(url)
    .then(res => res.json())
    .then(data => ({ success: true, data } as ApiResponse<T>))
    .catch(error => ({ success: false, error: error.message } as ApiResponse<T>));
}

// Уровень 4: Полностью типизированный клиент
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = `/${string}`;
type ApiClient = {
  [M in Lowercase<HttpMethod>]: <T>(endpoint: Endpoint, data?: any) => Promise<ApiResponse<T>>;
};

const apiClient: ApiClient = {
  get: validatedGet,
  post: (endpoint, data) => fetch(endpoint, { method: "POST", body: JSON.stringify(data) }).then(r => r.json()),
  put: (endpoint, data) => fetch(endpoint, { method: "PUT", body: JSON.stringify(data) }).then(r => r.json()),
  delete: (endpoint) => fetch(endpoint, { method: "DELETE" }).then(r => r.json())
};
```

## Лучшие практики для сложных систем

### 1. Архитектура типов
- Начинайте с простых примитивов и постепенно усложняйте
- Используйте псевдонимы для документирования сложных типов
- Применяйте иерархии типов, а не плоские структуры
- Документируйте интеграции между типами

### 2. Производительность
- Избегайте чрезмерно сложных условных типов
- Тестируйте время компиляции при сложных преобразованиях
- Используйте именованные типы вместо анонимных объединений

### 3. Поддерживаемость
- Создавайте переиспользуемые утилиты
- Используйте модули для организации типов
- Документируйте сложные интеграции типов
- Пишите тесты типов (type tests)

## Завершающие рекомендации

TypeScript - это не просто надстройка над JavaScript, а полноценная система типизации, которая позволяет создавать безопасные, надежные и легко поддерживаемые приложения. Ключ к успеху - понимание связей между концепциями и правильное применение их в зависимости от контекста.

Основные принципы:
1. **Начинайте с простого** и постепенно усложняйте
2. **Думайте на уровне типов** при проектировании архитектуры
3. **Используйте систему типов** для предотвращения ошибок, а не просто для документирования
4. **Интегрируйте типы** в процесс разработки (IDE, проверки, CI/CD)
5. **Обучайте команду** паттернам безопасного использования сложных типов
# Literal Types

Литеральные типы (Literal Types) в TypeScript позволяют использовать конкретные значения как типы. Это означает, что вы можете указать не просто "string", а конкретную строку "hello", или не просто "number", а конкретное число 42.

## Строковые литеральные типы

### Основное использование
```typescript
// Строковые литеральные типы
type Direction = "up" | "down" | "left" | "right";
type HttpStatus = "success" | "error" | "pending";

function move(direction: Direction) {
  // direction может быть только одним из трех значений
  console.log(`Moving ${direction}`);
}

// Использование
move("up");    // OK
move("down");  // OK
// move("around"); // Ошибка: "around" не является допустимым значением Direction

// Практический пример
type UserRole = "admin" | "user" | "guest";
type Permission = "read" | "write" | "delete";

function checkPermission(role: UserRole, permission: Permission): boolean {
  switch (role) {
    case "admin":
      return true;  // админ имеет все права
    case "user":
      return permission === "read" || permission === "write";
    case "guest":
      return permission === "read";
  }
}
```

### Литеральные типы в объектах
```typescript
interface ApiResponse {
  status: "success" | "error";
  message: string;
}

// Более сложный пример
interface UserAction {
  type: "LOGIN" | "LOGOUT" | "UPDATE_PROFILE";
  payload: string;
  timestamp: Date;
}

const loginAction: UserAction = {
  type: "LOGIN",
  payload: "user123",
  timestamp: new Date()
};
```

## Числовые литеральные типы

### Использование числовых литералов
```typescript
// Числовые литеральные типы
type DiceValue = 1 | 2 | 3 | 4 | 5 | 6;
type HttpStatusCodes = 200 | 201 | 400 | 401 | 404 | 500;

function rollDice(): DiceValue {
  const value = Math.floor(Math.random() * 6) + 1;
  return value as DiceValue; // Приведение типа
}

function handleResponse(status: HttpStatusCodes) {
  switch (status) {
    case 200:
      console.log("Success");
      break;
    case 400:
      console.log("Bad Request");
      break;
    case 404:
      console.log("Not Found");
      break;
    default:
      console.log("Server Error");
  }
}

// Использование в функциях
function setStatusCode(code: HttpStatusCodes): void {
  // код может быть только одним из разрешенных значений
  console.log(`Status set to: ${code}`);
}

setStatusCode(200); // OK
// setStatusCode(300); // Ошибка: 300 не является допустимым значением
```

## Булевы литеральные типы

### Использование true/false как типов
```typescript
type YesNo = true | false;
type FeatureFlag = "enabled" | "disabled";

function configureFeature(enabled: true | false) {
  if (enabled) {
    console.log("Feature enabled");
  } else {
    console.log("Feature disabled");
  }
}

// Практический пример с настройками
interface AppConfig {
  debug: true | false;
  production: boolean; // или просто boolean
  logging: "verbose" | "normal" | "none";
}

const config: AppConfig = {
  debug: false,
  production: true,
  logging: "normal"
};
```

## Смешанные литеральные типы

### Комбинирование разных литералов
```typescript
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Resource = "users" | "posts" | "comments";
type Endpoint = `${HttpMethod}/${Resource}`;

// Результат: все возможные комбинации
// "GET/users" | "GET/posts" | "GET/comments" |
// "POST/users" | "POST/posts" | "POST/comments" |
// и т.д.

function callEndpoint(endpoint: Endpoint) {
  console.log(`Calling: ${endpoint}`);
}

callEndpoint("GET/users");    // OK
callEndpoint("POST/posts");   // OK
// callEndpoint("PATCH/users"); // Ошибка: PATCH не входит в HttpMethod
```

## Литеральные типы с дженериками

### Обобщения с литеральными типами
```typescript
type ValidKey<T> = T extends string ? `${T}_id` : never;

type UserId = ValidKey<"user">;    // "user_id"
type PostId = ValidKey<"post">;    // "post_id"

// Сложный пример с ограничениями
function createValidator<T extends string | number>(
  allowedValues: T[]
) {
  return (value: T): boolean => {
    return allowedValues.includes(value);
  };
}

const statusValidator = createValidator(["active", "inactive", "pending"] as const);
console.log(statusValidator("active"));   // true
// console.log(statusValidator("unknown")); // Ошибка: "unknown" не в allowedValues
```

## Автоматический вывод литеральных типов

### Вывод литеральных типов
```typescript
// По умолчанию TypeScript выводит литеральные типы для констант
const direction = "left";  // тип: "left", не string
const status = true;       // тип: true, не boolean
const count = 42;          // тип: 42, не number

// Вывод в массивах
const directions = ["up", "down", "left", "right"];
// тип: string[], потому что массив может быть изменен

// Для получения литеральных типов используйте as const
const directionsConst = ["up", "down", "left", "right"] as const;
// тип: readonly ["up", "down", "left", "right"]

// В объектах
const config = {
  theme: "dark",
  notifications: true,
  maxRetries: 3
}; // тип: { theme: string; notifications: boolean; maxRetries: number }

const configConst = {
  theme: "dark",
  notifications: true,
  maxRetries: 3
} as const;
// тип: { readonly theme: "dark"; readonly notifications: true; readonly maxRetries: 3 }
```

## Практические применения

### API Responses
```typescript
interface ApiResponse<T> {
  status: "success" | "error";
  code: 200 | 400 | 401 | 404 | 500;
  message: string;
  data?: T;
}

type SuccessResponse<T> = ApiResponse<T> & { 
  status: "success"; 
  code: 200;
  data: T;
};

type ErrorResponse = ApiResponse<never> & { 
  status: "error"; 
  data?: never;
};

function handleApiResponse<T>(response: ApiResponse<T>): T | Error {
  if (response.status === "success") {
    // TypeScript знает, что в этом случае response.code === 200
    // и response.data определенно существует
    return response.data!; // non-null assertion
  } else {
    return new Error(response.message);
  }
}
```

### Конфигурационные объекты
```typescript
type Environment = "development" | "staging" | "production";
type LogLevel = "debug" | "info" | "warn" | "error";

interface AppConfig {
  environment: Environment;
  logLevel: LogLevel;
  apiTimeout: 5000 | 10000 | 30000;
  retryAttempts: 1 | 2 | 3 | 5;
}

const config: AppConfig = {
  environment: "production",
  logLevel: "error",
  apiTimeout: 10000,
  retryAttempts: 3
};
```

### Функциональные перегрузки
```typescript
function formatValue(value: string, type: "uppercase"): string;
function formatValue(value: string, type: "lowercase"): string;
function formatValue(value: number, type: "currency"): string;
function formatValue(value: string | number, type: "uppercase" | "lowercase" | "currency"): string {
  if (type === "uppercase" && typeof value === "string") {
    return value.toUpperCase();
  } else if (type === "lowercase" && typeof value === "string") {
    return value.toLowerCase();
  } else if (type === "currency" && typeof value === "number") {
    return `$${value.toFixed(2)}`;
  }
  throw new Error("Invalid type");
}

// Использование
formatValue("hello", "uppercase"); // "HELLO"
formatValue(123.45, "currency");   // "$123.45"
```

## Продвинутые паттерны

### Литеральные типы с условными типами
```typescript
type GetReturnType<T extends string> = 
  T extends "number" ? number :
  T extends "string" ? string :
  T extends "boolean" ? boolean :
  T extends "array" ? any[] : never;

function createValue<T extends "number" | "string" | "boolean" | "array">(
  type: T,
  value: GetReturnType<T>
): GetReturnType<T> {
  return value;
}

const numValue = createValue("number", 42);    // number
const strValue = createValue("string", "hi");  // string
```

### Типизация событий
```typescript
type KnownEvents = 
  | `user:${"login" | "logout" | "update"}`
  | `file:${"upload" | "download" | "delete"}`
  | `system:${"start" | "stop" | "restart"}`;

interface EventHandler {
  <T extends KnownEvents>(
    event: T,
    callback: T extends `user:${infer Action}` ? (user: string) => void :
              T extends `file:${infer Action}` ? (filename: string) => void :
              () => void
  ): void;
}
```

## Лучшие практики

1. **Используйте литеральные типы для ограниченных наборов значений** - особенно в API и конфигурациях
2. **Комбинируйте с объединениями для создания флагов** - `status: "active" | "inactive"`
3. **Используйте `as const` для вывода литеральных типов** - из объектов и массивов
4. **Избегайте слишком большого количества литеральных типов** - это может усложнить код
5. **Документируйте смысл литеральных значений** - особенно в сигнатурах функций

## Связь с другими концепциями
- [[template-literal-types]] - комбинация с шаблонными литералами
- [[discriminated-unions]] - часто используются в дискриминированных объединениях
- [[../type-system/type-compatibility]] - как литералы совместимы с базовыми типами
- [[advanced-types/conditional-types]] - условная типизация с литералами
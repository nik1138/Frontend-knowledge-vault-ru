# Перечисления (Enums) в TypeScript

Перечисления (enums) - это способ определения набора именованных констант. Они позволяют создавать более читаемый и типобезопасный код, особенно когда есть фиксированный набор значений, с которыми нужно работать.

## Основной синтаксис

Перечисления создаются с помощью ключевого слова `enum`:

```typescript
enum Direction {
  Up,
  Down,
  Left,
  Right
}

// Использование
let move: Direction = Direction.Up;
console.log(move); // Выведет: 0 (по умолчанию первый элемент имеет значение 0)
```

## Числовые перечисления

### Автоматическая нумерация

```typescript
enum Status {
  Pending,    // 0
  Success,    // 1
  Error       // 2
}

// Можно начать нумерацию с определенного значения
enum StatusCode {
  NotFound = 404,
  ServerError = 500,
  Success = 200  // Обратите внимание: порядок не важен
}

let status = StatusCode.NotFound;  // 404
```

### Ручная нумерация

```typescript
enum FileAccess {
  None = 0,
  Read = 2,
  Write = 4,
  ReadWrite = Read | Write  // 6 (побитовое ИЛИ)
}

// Часто используемый паттерн - битовые флаги
enum Permission {
  Read = 1 << 0,    // 1
  Write = 1 << 1,   // 2
  Execute = 1 << 2, // 4
  All = Read | Write | Execute // 7
}
```

## Строковые перечисления

```typescript
enum HttpStatus {
  OK = "OK",
  NotFound = "NOT_FOUND",
  ServerError = "SERVER_ERROR"
}

let responseStatus: HttpStatus = HttpStatus.OK;
console.log(responseStatus); // "OK"

// Смешанные перечисления (не рекомендуется)
enum MixedEnum {
  A = 1,
  B = "test",  // После числового значения нужно явно указать все последующие
  C = 2
}
```

## Обратное сопоставление

В числовых перечислениях TypeScript создает обратное сопоставление:

```typescript
enum Direction {
  Up = 0,
  Down = 1,
  Left = 2,
  Right = 3
}

// Обратное сопоставление
console.log(Direction[0]); // "Up"
console.log(Direction[Direction.Up]); // "Up"

// Это работает только для числовых перечислений
enum StringEnum {
  A = "Apple",
  B = "Banana"
}

// Обратное сопоставление не работает для строковых перечислений
// console.log(StringEnum["Apple"]); // undefined
```

## Перечисления в функциях и условиях

### Использование в switch

```typescript
enum MessageType {
  Info,
  Warning,
  Error
}

function handleMessage(type: MessageType, message: string): void {
  switch (type) {
    case MessageType.Info:
      console.log(`[INFO] ${message}`);
      break;
    case MessageType.Warning:
      console.warn(`[WARNING] ${message}`);
      break;
    case MessageType.Error:
      console.error(`[ERROR] ${message}`);
      break;
  }
}

handleMessage(MessageType.Info, "Процесс запущен");
```

### Проверка значений перечисления

```typescript
function isValidStatus(value: number): value is MessageType {
  return !isNaN(value) && !!MessageType[value];
}

// Использование
if (isValidStatus(2)) {
  console.log("Это допустимое значение перечисления");
}
```

## Вычисляемые и константные члены

### Константные члены

```typescript
enum FileAccess {
  // Константные выражения
  None = 0,
  Read = 1 << 0,      // 1
  Write = 1 << 1,     // 2
  Execute = 1 << 2,   // 4
  ReadWrite = Read | Write // 3
}
```

### Вычисляемые члены

```typescript
function getDefaultValue(): number {
  return 5;
}

enum ComputedEnum {
  A = 1,
  B = getDefaultValue(),  // Вычисляемое значение
  C = Math.random(),      // Вычисляемое значение
  D                 // Если не указано значение, будет 6 (предыдущее + 1)
}
```

## Практические применения

### 1. Состояния приложения

```typescript
enum LoadingState {
  Idle = "idle",
  Loading = "loading",
  Success = "success",
  Error = "error"
}

interface AppState {
  data: any;
  loadingState: LoadingState;
}

function handleState(state: AppState): void {
  switch (state.loadingState) {
    case LoadingState.Idle:
      console.log("Ожидание запроса...");
      break;
    case LoadingState.Loading:
      console.log("Загрузка данных...");
      break;
    case LoadingState.Success:
      console.log("Данные успешно загружены:", state.data);
      break;
    case LoadingState.Error:
      console.error("Ошибка загрузки данных");
      break;
  }
}
```

### 2. Типы пользователей

```typescript
enum UserRole {
  Guest = "guest",
  User = "user",
  Admin = "admin",
  SuperAdmin = "super_admin"
}

interface User {
  id: number;
  name: string;
  role: UserRole;
}

function canAccessAdminPanel(user: User): boolean {
  return user.role === UserRole.Admin || user.role === UserRole.SuperAdmin;
}

function canDeleteUser(deleter: User, target: User): boolean {
  // SuperAdmin может удалять всех
  if (deleter.role === UserRole.SuperAdmin) return true;
  
  // Admin может удалять только обычных пользователей
  if (deleter.role === UserRole.Admin) {
    return target.role === UserRole.User;
  }
  
  return false;
}
```

### 3. Конфигурация приложения

```typescript
enum Environment {
  Development = "development",
  Staging = "staging",
  Production = "production"
}

enum LogLevel {
  Debug = 0,
  Info = 1,
  Warning = 2,
  Error = 3
}

interface AppConfig {
  environment: Environment;
  logLevel: LogLevel;
  apiUrl: string;
}

function createConfig(env: Environment): AppConfig {
  switch (env) {
    case Environment.Development:
      return {
        environment: Environment.Development,
        logLevel: LogLevel.Debug,
        apiUrl: "http://localhost:3000/api"
      };
    case Environment.Staging:
      return {
        environment: Environment.Staging,
        logLevel: LogLevel.Info,
        apiUrl: "https://staging-api.example.com"
      };
    case Environment.Production:
      return {
        environment: Environment.Production,
        logLevel: LogLevel.Error,
        apiUrl: "https://api.example.com"
      };
  }
}
```

## Константные перечисления

Константные перечисления (`const enum`) компилируются в inline-значения:

```typescript
const enum Direction {
  Up,
  Down,
  Left,
  Right
}

// При компиляции это будет преобразовано в:
// let direction = 0 /* Up */;
let direction = Direction.Up;

// Это улучшает производительность, так как устраняет обращения к объекту перечисления
```

## Объединения строковых литералов как альтернатива

Иногда лучше использовать объединения строковых литералов вместо перечислений:

```typescript
// Вместо перечисления
enum StatusEnum {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
  Pending = "PENDING"
}

// Можно использовать объединение
type StatusType = "ACTIVE" | "INACTIVE" | "PENDING";

// Преимущества объединений:
// 1. Меньше сгенерированного кода
// 2. Лучшая совместимость с системой типов
// 3. Можно легко расширять
type ExtendedStatus = StatusType | "ARCHIVED" | "DELETED";
```

## Продвинутые паттерны

### 1. Перечисления с методами

```typescript
enum HttpStatus {
  OK = 200,
  NOT_FOUND = 404,
  SERVER_ERROR = 500
}

// Расширение перечисления с помощью namespace
namespace HttpStatus {
  export function isSuccess(code: number): boolean {
    return code >= 200 && code < 300;
  }
  
  export function isError(code: number): boolean {
    return code >= 400;
  }
  
  export function getMessage(code: HttpStatus): string {
    switch (code) {
      case HttpStatus.OK: return "OK";
      case HttpStatus.NOT_FOUND: return "Not Found";
      case HttpStatus.SERVER_ERROR: return "Server Error";
      default: return "Unknown";
    }
  }
}

console.log(HttpStatus.getMessage(HttpStatus.NOT_FOUND)); // "Not Found"
```

### 2. Перечисления для событий

```typescript
enum AppEvent {
  UserLogin = "USER_LOGIN",
  UserLogout = "USER_LOGOUT",
  DataUpdate = "DATA_UPDATE",
  ErrorOccurred = "ERROR_OCCURRED"
}

interface EventPayloadMap {
  [AppEvent.UserLogin]: { userId: number; timestamp: Date };
  [AppEvent.UserLogout]: { userId: number; duration: number };
  [AppEvent.DataUpdate]: { key: string; oldValue: any; newValue: any };
  [AppEvent.ErrorOccurred]: { message: string; stack?: string };
}

function emitEvent<T extends AppEvent>(
  event: T,
  payload: EventPayloadMap[T]
): void {
  console.log(`Событие: ${event}`, payload);
}

// Использование
emitEvent(AppEvent.UserLogin, { 
  userId: 123, 
  timestamp: new Date() 
});
```

## Практические примеры из реальной разработки

### 1. Работа с API и HTTP статусами

```typescript
enum HttpStatusCode {
  // 2xx Успешные
  OK = 200,
  CREATED = 201,
  ACCEPTED = 202,
  
  // 3xx Перенаправления
  MOVED_PERMANENTLY = 301,
  FOUND = 302,
  
  // 4xx Ошибки клиента
  BAD_REQUEST = 400,
  UNAUTHORIZED = 401,
  FORBIDDEN = 403,
  NOT_FOUND = 404,
  
  // 5xx Ошибки сервера
  INTERNAL_SERVER_ERROR = 500,
  NOT_IMPLEMENTED = 501,
  BAD_GATEWAY = 502,
  SERVICE_UNAVAILABLE = 503
}

class ApiClient {
  async request(url: string): Promise<any> {
    try {
      const response = await fetch(url);
      
      if (response.status === HttpStatusCode.OK) {
        return await response.json();
      } else if (response.status === HttpStatusCode.NOT_FOUND) {
        throw new Error(`Ресурс не найден: ${url}`);
      } else if (response.status === HttpStatusCode.UNAUTHORIZED) {
        // Обработка неавторизованного доступа
        this.handleUnauthorized();
      } else if (HttpStatusCode.isClientError(response.status)) {
        throw new Error(`Ошибка клиента: ${response.status}`);
      } else if (HttpStatusCode.isServerError(response.status)) {
        throw new Error(`Ошибка сервера: ${response.status}`);
      }
    } catch (error) {
      throw error;
    }
  }
  
  private handleUnauthorized(): void {
    // Логика обработки неавторизованного доступа
    console.log("Пользователь не авторизован");
  }
}

// Добавление статических методов к перечислению через namespace
namespace HttpStatusCode {
  export function isClientError(status: number): boolean {
    return status >= 400 && status < 500;
  }
  
  export function isServerError(status: number): boolean {
    return status >= 500 && status < 600;
  }
  
  export function isSuccess(status: number): boolean {
    return status >= 200 && status < 300;
  }
}
```

### 2. Управление состоянием игры

```typescript
enum GameState {
  MainMenu = "MAIN_MENU",
  Playing = "PLAYING",
  Paused = "PAUSED",
  GameOver = "GAME_OVER",
  Settings = "SETTINGS"
}

enum GameAction {
  StartGame = "START_GAME",
  PauseGame = "PAUSE_GAME",
  ResumeGame = "RESUME_GAME",
  EndGame = "END_GAME",
  ChangeSettings = "CHANGE_SETTINGS"
}

interface GameContext {
  state: GameState;
  score: number;
  level: number;
  playerLives: number;
}

function handleGameAction(
  context: GameContext, 
  action: GameAction
): GameContext {
  switch (action) {
    case GameAction.StartGame:
      if (context.state === GameState.MainMenu) {
        return { ...context, state: GameState.Playing };
      }
      return context;
      
    case GameAction.PauseGame:
      if (context.state === GameState.Playing) {
        return { ...context, state: GameState.Paused };
      }
      return context;
      
    case GameAction.ResumeGame:
      if (context.state === GameState.Paused) {
        return { ...context, state: GameState.Playing };
      }
      return context;
      
    case GameAction.EndGame:
      return { 
        ...context, 
        state: GameState.GameOver,
        playerLives: 0 
      };
      
    default:
      return context;
  }
}
```

## Ограничения и особенности

### 1. Взаимодействие с JavaScript

```typescript
// При компиляции enum превращается в JavaScript объект
enum Direction { Up, Down, Left, Right }

// Это компилируется в:
// var Direction = {};
// Direction[Direction["Up"] = 0] = "Up";
// Direction[Direction["Down"] = 1] = "Down";
// Direction[Direction["Left"] = 2] = "Left";
// Direction[Direction["Right"] = 3] = "Right";
```

### 2. Совместимость типов

```typescript
enum StatusA { Success, Error }
enum StatusB { Success, Error }

let statusA: StatusA = StatusA.Success;
// let statusB: StatusB = statusA;  // Ошибка: перечисления несовместимы
```

### 3. Ограничения на значения

```typescript
// Значения перечисления должны быть константами или вычисляемыми выражениями
enum ValidEnum {
  A = 1,
  B = 2,
  C = A + B  // OK: вычисляемое выражение
}

// Следующее будет ошибкой:
// enum InvalidEnum {
//   A = "hello".length,  // строковые методы не разрешены в вычисляемых значениях
// }
```

## Лучшие практики

1. **Используйте строковые перечисления для API** - они более безопасны при сериализации
2. **Используйте числовые перечисления для внутренней логики** - они компактнее
3. **Предпочитайте объединения строковым литералам** - для более гибкой системы типов
4. **Используйте `const enum` для производительности** - когда не нужен рефлексивный доступ
5. **Документируйте значения перечислений** - особенно когда они имеют бизнес-смысл
6. **Избегайте слишком большого количества значений** - перечисления должны быть легко читаемы

## Связи с другими концепциями

- [[ts/types/literal-types|Литеральные типы]] - альтернатива строковым перечислениям
- [[ts/types/union-intersection-types|Объединения типов]] - часто используются вместе с перечислениями
- [[ts/advanced-types/discriminated-unions|Дискриминированные объединения]] - более гибкая альтернатива перечислениям
- [[ts/type-system/type-compatibility|Совместимость типов]] - как перечисления взаимодействуют с системой типов
- [[js/objects]] - перечисления компилируются в JavaScript объекты
# Дискриминированные объединения в TypeScript

Дискриминированные объединения (discriminated unions), также известные как объединения с метками (tagged unions) или суммарные типы, - это мощный паттерн типизации, который позволяет создавать типобезопасные структуры данных, представляющие разные состояния или типы с общей "дискриминирующей" меткой. Этот паттерн особенно полезен для представления состояний, результатов операций и различных сценариев поведения.

## Основная концепция

Дискриминированные объединения состоят из:
1. Объединения типов (union), где каждый тип имеет общее свойство (дискриминант)
2. Общего свойства, которое отличает один тип от другого
3. Возможности сужать типы на основе значения дискриминанта

```typescript
interface Loading {
  state: "loading";
}

interface Success {
  state: "success";
  data: string;
}

interface Error {
  state: "error";
  message: string;
}

type Status = Loading | Success | Error;

function handleStatus(status: Status) {
  if (status.state === "loading") {
    // Здесь TypeScript знает, что status - это Loading
    console.log("Загрузка...");
  } else if (status.state === "success") {
    // Здесь TypeScript знает, что status - это Success
    // и что у него есть свойство data
    console.log(`Данные: ${status.data}`);
  } else {
    // Здесь TypeScript знает, что status - это Error
    // и что у него есть свойство message
    console.log(`Ошибка: ${status.message}`);
  }
}
```

## Простые примеры

### 1. Базовое использование

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  side: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Square | Triangle;

function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    case "triangle":
      return 0.5 * shape.base * shape.height;
  }
}
```

### 2. Использование с if/else

```typescript
type ApiResponse<T> = 
  | { type: "success"; data: T }
  | { type: "error"; error: string }
  | { type: "loading" };

function handleResponse<T>(response: ApiResponse<T>) {
  if (response.type === "success") {
    // response имеет тип { type: "success"; data: T }
    console.log("Успешный ответ:", response.data);
  } else if (response.type === "error") {
    // response имеет тип { type: "error"; error: string }
    console.error("Ошибка:", response.error);
  } else {
    // response имеет тип { type: "loading" }
    console.log("Загрузка...");
  }
}
```

## Продвинутые паттерны

### 1. Множественные дискриминанты

```typescript
interface User {
  type: "user";
  name: string;
  email: string;
}

interface Admin {
  type: "admin";
  name: string;
  email: string;
  permissions: string[];
}

interface Guest {
  type: "guest";
  sessionId: string;
}

type Account = User | Admin | Guest;

function getDisplayName(account: Account): string {
  if (account.type === "guest") {
    return `Гость (${account.sessionId})`;
  }
  
  // Для user и admin поле name доступно
  if (account.type === "admin") {
    return `${account.name} (администратор)`;
  }
  
  // Остается только user
  return account.name;
}
```

### 2. Вложенные дискриминированные объединения

```typescript
interface NetworkSuccess {
  status: "success";
  data: string;
}

interface NetworkError {
  status: "error";
  error: string;
  code: number;
}

interface NetworkLoading {
  status: "loading";
  progress?: number;
}

type NetworkResponse = NetworkSuccess | NetworkError | NetworkLoading;

interface CacheHit {
  source: "cache";
  data: string;
}

interface CacheMiss {
  source: "network";
  response: NetworkResponse;
}

type DataSource = CacheHit | CacheMiss;

function handleDataSource(source: DataSource) {
  if (source.source === "cache") {
    // source - это CacheHit
    console.log("Данные из кэша:", source.data);
  } else {
    // source - это CacheMiss, и у него есть response
    const response = source.response;
    
    if (response.status === "success") {
      // response - это NetworkSuccess
      console.log("Сетевой ответ (успех):", response.data);
    } else if (response.status === "error") {
      // response - это NetworkError
      console.log(`Сетевой ответ (ошибка): ${response.error}, код: ${response.code}`);
    } else {
      // response - это NetworkLoading
      const progress = response.progress ?? 0;
      console.log(`Загрузка... (${progress}%)`);
    }
  }
}
```

## Практические применения

### 1. Управление состоянием приложения

```typescript
interface IdleState {
  status: "idle";
}

interface LoadingState {
  status: "loading";
  progress?: number;
}

interface SuccessState<T> {
  status: "success";
  data: T;
}

interface ErrorState {
  status: "error";
  message: string;
}

type AppState<T> = 
  | IdleState 
  | LoadingState 
  | SuccessState<T> 
  | ErrorState;

function renderApp<T>(state: AppState<T>) {
  switch (state.status) {
    case "idle":
      return "Готов к работе";
    case "loading":
      const progress = state.progress ? ` (${state.progress}%)` : "";
      return `Загрузка${progress}...`;
    case "success":
      return `Данные: ${JSON.stringify(state.data)}`;
    case "error":
      return `Ошибка: ${state.message}`;
  }
}
```

### 2. Работа с формами

```typescript
interface UnsubmittedForm {
  status: "unsubmitted";
  formData: Record<string, any>;
}

interface ValidatingForm {
  status: "validating";
  formData: Record<string, any>;
}

interface ValidForm {
  status: "valid";
  formData: Record<string, any>;
  validatedData: Record<string, any>;
}

interface InvalidForm {
  status: "invalid";
  formData: Record<string, any>;
  errors: Record<string, string[]>;
}

interface SubmittedForm {
  status: "submitted";
  formData: Record<string, any>;
  submissionId: string;
}

type FormState = 
  | UnsubmittedForm
  | ValidatingForm
  | ValidForm
  | InvalidForm
  | SubmittedForm;

function canSubmitForm(state: FormState): boolean {
  return state.status === "valid" || state.status === "invalid";
}

function getSubmitButtonLabel(state: FormState): string {
  switch (state.status) {
    case "unsubmitted":
      return "Отправить";
    case "validating":
      return "Проверка...";
    case "valid":
      return "Отправить";
    case "invalid":
      return "Исправить и отправить";
    case "submitted":
      return "Отправлено";
  }
}
```

### 3. Обработка результатов асинхронных операций

```typescript
interface PendingResult {
  status: "pending";
}

interface SuccessResult<T> {
  status: "success";
  value: T;
}

interface FailureResult {
  status: "failure";
  error: Error;
}

type AsyncResult<T> = PendingResult | SuccessResult<T> | FailureResult;

function handleAsyncResult<T>(result: AsyncResult<T>) {
  if (result.status === "pending") {
    console.log("Операция в процессе...");
    return;
  }
  
  if (result.status === "success") {
    console.log("Успешный результат:", result.value);
    return;
  }
  
  // Остается только failure
  console.error("Ошибка операции:", result.error.message);
}
```

## Исчерпывающая проверка

Одно из важных преимуществ дискриминированных объединений - возможность обеспечить исчерпывающую проверку всех возможных случаев:

```typescript
// Функция, которая должна обрабатывать все возможные типы
function exhaustiveCheck(value: never): never {
  throw new Error(`Необработанный тип: ${value}`);
}

function processShape(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return `Круг с радиусом ${shape.radius}`;
    case "square":
      return `Квадрат со стороной ${shape.side}`;
    case "triangle":
      return `Треугольник с основанием ${shape.base} и высотой ${shape.height}`;
    default:
      // Если добавить новый тип в Shape без обновления этой функции,
      // TypeScript выдаст ошибку, потому что тип shape не будет never
      return exhaustiveCheck(shape);
  }
}
```

## Связь с другими паттернами

### 1. Комбинация с условными типами

```typescript
// Утилита для извлечения типа по дискриминанту
type ExtractByTag<T, TagName, TagValue> = T extends { [K in TagName]: TagValue } ? T : never;

type SquareOnly = ExtractByTag<Shape, "kind", "square">;
// { kind: "square"; side: number; }
```

### 2. Использование с сопоставленными типами

```typescript
// Создание типа, который содержит только свойства, специфичные для каждого варианта
type SpecificProperties<T extends { type: string }> = {
  [K in T["type"]]: Extract<T, { type: K }>
};

type StatusSpecific = SpecificProperties<ApiResponse<any>>;
// {
//   success: { type: "success"; data: any; };
//   error: { type: "error"; error: string; };
//   loading: { type: "loading"; };
// }
```

## Практические примеры из реальной разработки

### 1. Работа с API ответами

```typescript
interface ApiSuccess<T> {
  kind: "success";
  data: T;
  statusCode: 200 | 201;
}

interface ApiError {
  kind: "error";
  error: string;
  statusCode: number;
  details?: string;
}

interface ApiLoading {
  kind: "loading";
  progress?: number;
}

interface ApiIdle {
  kind: "idle";
}

type ApiCall<T> = ApiSuccess<T> | ApiError | ApiLoading | ApiIdle;

async function makeApiCall<T>(url: string): Promise<ApiCall<T>> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      return {
        kind: "error",
        error: "Network error",
        statusCode: response.status
      };
    }
    
    const data: T = await response.json();
    return {
      kind: "success",
      data,
      statusCode: response.status as 200 | 201
    };
  } catch (error) {
    return {
      kind: "error",
      error: (error as Error).message,
      statusCode: 500
    };
  }
}

function renderApiResult<T>(result: ApiCall<T>) {
  switch (result.kind) {
    case "idle":
      return "Ожидание запроса...";
    case "loading":
      return `Загрузка... ${result.progress ? `${result.progress}%` : ''}`;
    case "success":
      return `Данные: ${JSON.stringify(result.data)}`;
    case "error":
      return `Ошибка ${result.statusCode}: ${result.error}`;
  }
}
```

### 2. Система событий с типизацией

```typescript
interface UserLoginEvent {
  type: "user_login";
  userId: number;
  timestamp: Date;
}

interface UserLogoutEvent {
  type: "user_logout";
  userId: number;
  duration: number;
}

interface ErrorEvent {
  type: "error";
  error: string;
  stack?: string;
}

interface PageViewEvent {
  type: "page_view";
  url: string;
  referrer?: string;
}

type AppEvent = UserLoginEvent | UserLogoutEvent | ErrorEvent | PageViewEvent;

function handleEvent(event: AppEvent) {
  switch (event.type) {
    case "user_login":
      console.log(`Пользователь ${event.userId} вошел в систему`);
      break;
    case "user_logout":
      console.log(`Пользователь ${event.userId} вышел, провел ${event.duration} сек`);
      break;
    case "error":
      console.error(`Ошибка: ${event.error}${event.stack ? `\n${event.stack}` : ''}`);
      break;
    case "page_view":
      console.log(`Просмотр страницы: ${event.url}${event.referrer ? ` (с ${event.referrer})` : ''}`);
      break;
  }
}
```

## Ограничения и лучшие практики

### 1. Выбор подходящего дискриминанта

```typescript
// Хорошо: понятный и уникальный дискриминант
interface GoodExample {
  status: "loading" | "success" | "error";
  // другие поля
}

// Не рекомендуется: общий дискриминант, который может совпадать
interface BadExample {
  name: "loading" | "success" | "error"; // имя может совпадать с другими сущностями
  // другие поля
}
```

### 2. Избегайте слишком сложных дискриминантов

```typescript
// Сложный дискриминант может затруднить понимание
interface OverlyComplex {
  metadata: {
    typeInfo: {
      category: "user" | "admin" | "guest";
      version: 1 | 2;
    };
  };
  // другие поля
}

// Лучше использовать простой дискриминант
interface BetterComplex {
  type: "user" | "admin" | "guest";
  version: 1 | 2;
  // другие поля
}
```

### 3. Используйте консистентные имена дискриминантов

```typescript
// Используйте одинаковые имена дискриминантов в рамках одного объединения
type Response = 
  | { kind: "success"; data: any }  // используем kind
  | { kind: "error"; message: string }  // согласовано с предыдущим
  // а не { type: "loading" } - это создаст путаницу
```

## Лучшие практики

1. **Используйте понятные имена дискриминантов** - `type`, `kind`, `status` - распространенные варианты
2. **Делайте дискриминанты обязательными** - они должны присутствовать во всех членах объединения
3. **Используйте литеральные типы для дискриминантов** - это обеспечивает строгую типизацию
4. **Обеспечивайте исчерпывающую проверку** - используйте `never` для проверки всех случаев
5. **Документируйте варианты объединения** - особенно в больших объединениях
6. **Рассматривайте дискриминированные объединения для управления состоянием** - они отлично подходят для этого

## Связи с другими концепциями

- [[ts/advanced-types/conditional-types|Условные типы]] - часто используются для работы с дискриминированными объединениями
- [[ts/type-system/type-guards|Защитники типов]] - механизм сужения типов
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - могут использоваться для преобразования дискриминированных объединений
- [[ts/basics/control-flow|Анализ потока управления]] - основа для сужения типов
- [[ts/utility-types/index|Утилиты типов]] - вспомогательные типы для работы с объединениями
---
aliases: ["Дискриминированные объединения", "TS Discriminated Unions"]
tags: [typescript, unions, advanced, patterns]
---

# Дискриминированные объединения

## Введение

Дискриминированные объединения (discriminated unions), также известные как tagged unions или sum types, - это мощный паттерн в TypeScript, который позволяет создавать безопасные и понятные типы для работы с данными, которые могут принимать несколько различных форм.

## Основы дискриминированных объединений

### Простое объединение с дискриминантом

```typescript
// Простое объединение с дискриминантом
type Status = 'success' | 'error' | 'loading';

interface SuccessState {
  status: 'success';
  data: string;
}

interface ErrorState {
  status: 'error';
  error: string;
}

interface LoadingState {
  status: 'loading';
  progress?: number;
}

type State = SuccessState | ErrorState | LoadingState;

// Функция, использующая дискриминированные объединения
function handleState(state: State) {
  switch (state.status) {
    case 'success':
      // В этой ветке TypeScript знает, что state - это SuccessState
      console.log('Data:', state.data);
      break;
      
    case 'error':
      // В этой ветке TypeScript знает, что state - это ErrorState
      console.log('Error:', state.error);
      break;
      
    case 'loading':
      // В этой ветке TypeScript знает, что state - это LoadingState
      console.log('Loading...', state.progress || 0);
      break;
  }
}
```

### Дискриминированные объединения в UI компонентах

```typescript
// Объединение для различных типов уведомлений
interface SuccessNotification {
  type: 'success';
  message: string;
  action?: () => void;
}

interface ErrorNotification {
  type: 'error';
  message: string;
  error: Error;
}

interface WarningNotification {
  type: 'warning';
  message: string;
  dismissible: boolean;
}

interface InfoNotification {
  type: 'info';
  message: string;
  duration?: number;
}

type Notification = SuccessNotification | ErrorNotification | WarningNotification | InfoNotification;

// Компонент уведомления
function NotificationComponent({ notification }: { notification: Notification }) {
  switch (notification.type) {
    case 'success':
      return (
        <div className="notification success">
          <span>{notification.message}</span>
          {notification.action && <button onClick={notification.action}>Action</button>}
        </div>
      );
      
    case 'error':
      return (
        <div className="notification error">
          <span>{notification.message}</span>
          <details>
            <summary>Error details</summary>
            <pre>{notification.error.stack}</pre>
          </details>
        </div>
      );
      
    case 'warning':
      return (
        <div className={`notification warning ${notification.dismissible ? 'dismissible' : ''}`}>
          <span>{notification.message}</span>
        </div>
      );
      
    case 'info':
      return (
        <div className="notification info">
          <span>{notification.message}</span>
          {notification.duration && <span>Auto-dismiss in {notification.duration}s</span>}
        </div>
      );
  }
}
```

## Продвинутые примеры

### Дискриминированные объединения с дженериками

```typescript
// Объединение для асинхронных операций с дженериками
interface IdleState<T = any> {
  status: 'idle';
  data?: T;
}

interface PendingState {
  status: 'pending';
  startTime: number;
}

interface SuccessState<T> {
  status: 'success';
  data: T;
  timestamp: number;
}

interface ErrorState<E = Error> {
  status: 'error';
  error: E;
  timestamp: number;
}

type AsyncState<T, E = Error> = 
  | IdleState<T>
  | PendingState
  | SuccessState<T>
  | ErrorState<E>;

// Функция для обработки асинхронного состояния
function handleAsyncState<T, E extends Error>(state: AsyncState<T, E>) {
  switch (state.status) {
    case 'idle':
      if (state.data) {
        console.log('Has cached data:', state.data);
      } else {
        console.log('No data yet, starting operation...');
      }
      break;
      
    case 'pending':
      const elapsed = Date.now() - state.startTime;
      console.log(`Loading... ${elapsed}ms elapsed`);
      break;
      
    case 'success':
      console.log('Success at:', new Date(state.timestamp).toISOString());
      console.log('Data:', state.data);
      break;
      
    case 'error':
      console.log('Error at:', new Date(state.timestamp).toISOString());
      console.log('Error:', state.error.message);
      break;
  }
}
```

### Дискриминированные объединения для действий (actions)

```typescript
// Объединение для Redux-подобных действий
interface IncrementAction {
  type: 'INCREMENT';
  amount: number;
}

interface DecrementAction {
  type: 'DECREMENT';
  amount: number;
}

interface ResetAction {
  type: 'RESET';
}

interface SetUserAction {
  type: 'SET_USER';
  payload: {
    id: number;
    name: string;
    email: string;
  };
}

interface LogoutAction {
  type: 'LOGOUT';
}

type AppAction = 
  | IncrementAction
  | DecrementAction
  | ResetAction
  | SetUserAction
  | LogoutAction;

// Редьюсер, использующий дискриминированные объединения
interface AppState {
  count: number;
  user: { id: number; name: string; email: string } | null;
}

function appReducer(state: AppState, action: AppAction): AppState {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + action.amount };
      
    case 'DECREMENT':
      return { ...state, count: state.count - action.amount };
      
    case 'RESET':
      return { ...state, count: 0 };
      
    case 'SET_USER':
      return { ...state, user: action.payload };
      
    case 'LOGOUT':
      return { ...state, user: null };
      
    // Это обеспечивает исчерпывающую проверку
    default:
      // Если TypeScript выдает ошибку здесь, значит, 
      // мы не обработали какой-то тип действия
      const exhaustiveCheck: never = action;
      throw new Error(`Unhandled action type: ${exhaustiveCheck}`);
  }
}
```

## Практические примеры

### Дискриминированные объединения для API ответов

```typescript
// Объединение для различных типов API ответов
interface SuccessResponse<T> {
  type: 'success';
  data: T;
  status: number;
}

interface ErrorResponse {
  type: 'error';
  error: {
    message: string;
    code: string;
    details?: Record<string, any>;
  };
  status: number;
}

interface LoadingResponse {
  type: 'loading';
  progress?: number;
}

interface CacheResponse<T> {
  type: 'cache';
  data: T;
  timestamp: number;
  source: 'memory' | 'localstorage' | 'indexeddb';
}

type ApiResponse<T> = 
  | SuccessResponse<T>
  | ErrorResponse
  | LoadingResponse
  | CacheResponse<T>;

// Обработка API ответа
async function handleApiResponse<T>(response: ApiResponse<T>): Promise<T | null> {
  switch (response.type) {
    case 'success':
      console.log(`Success: ${response.status} status`);
      return response.data;
      
    case 'error':
      console.error(`API Error: ${response.error.message} (code: ${response.error.code})`);
      throw new Error(response.error.message);
      
    case 'loading':
      console.log(`Loading... ${response.progress || 0}%`);
      return null;
      
    case 'cache':
      console.log(`Cache hit from ${response.source}`);
      console.log(`Cached at: ${new Date(response.timestamp).toISOString()}`);
      return response.data;
      
    default:
      const exhaustiveCheck: never = response;
      throw new Error(`Unhandled response type: ${exhaustiveCheck}`);
  }
}
```

### Дискриминированные объединения для форм

```typescript
// Объединение для различных состояний формы
interface PristineFormState<T> {
  status: 'pristine';
  values: T;
}

interface ValidFormState<T> {
  status: 'valid';
  values: T;
  errors: Record<string, never>; // Нет ошибок
}

interface InvalidFormState<T> {
  status: 'invalid';
  values: T;
  errors: Record<string, string[]>;
}

interface SubmittingFormState<T> {
  status: 'submitting';
  values: T;
  errors?: Record<string, string[]>;
}

interface SubmittedFormState<T> {
  status: 'submitted';
  values: T;
  result: 'success' | 'error';
  message?: string;
}

type FormState<T> = 
  | PristineFormState<T>
  | ValidFormState<T>
  | InvalidFormState<T>
  | SubmittingFormState<T>
  | SubmittedFormState<T>;

// Обработка состояния формы
function renderForm<T>(state: FormState<T>) {
  switch (state.status) {
    case 'pristine':
      return (
        <div>
          <h3>Form is ready</h3>
          <pre>{JSON.stringify(state.values, null, 2)}</pre>
        </div>
      );
      
    case 'valid':
      return (
        <div>
          <h3>Form is valid</h3>
          <button type="submit">Submit</button>
        </div>
      );
      
    case 'invalid':
      return (
        <div>
          <h3>Form has errors</h3>
          {Object.entries(state.errors).map(([field, errors]) => (
            <div key={field} className="error">
              {field}: {errors.join(', ')}
            </div>
          ))}
          <button type="submit" disabled>Submit</button>
        </div>
      );
      
    case 'submitting':
      return (
        <div>
          <h3>Submitting...</h3>
          {state.errors && Object.entries(state.errors).map(([field, errors]) => (
            <div key={field} className="error">
              {field}: {errors.join(', ')}
            </div>
          ))}
        </div>
      );
      
    case 'submitted':
      return (
        <div>
          <h3>Submission {state.result}</h3>
          {state.message && <p>{state.message}</p>}
        </div>
      );
  }
}
```

### Дискриминированные объединения для конфигурации

```typescript
// Объединение для различных типов конфигурации
interface DefaultConfig {
  type: 'default';
  features: string[];
  timeout: number;
}

interface DevelopmentConfig {
  type: 'development';
  features: string[];
  timeout: number;
  debug: boolean;
  mockData: boolean;
}

interface ProductionConfig {
  type: 'production';
  features: string[];
  timeout: number;
  analytics: {
    trackingId: string;
    enabled: boolean;
  };
}

interface TestConfig {
  type: 'test';
  features: string[];
  timeout: number;
  mockData: boolean;
  mockDelay: number;
}

type AppConfig = 
  | DefaultConfig
  | DevelopmentConfig
  | ProductionConfig
  | TestConfig;

// Функция для настройки приложения на основе конфигурации
function setupApp(config: AppConfig) {
  switch (config.type) {
    case 'development':
      console.log('Setting up development environment');
      if (config.debug) {
        console.log('Debug mode enabled');
      }
      break;
      
    case 'production':
      console.log('Setting up production environment');
      if (config.analytics.enabled) {
        console.log('Analytics enabled with ID:', config.analytics.trackingId);
      }
      break;
      
    case 'test':
      console.log('Setting up test environment');
      console.log('Mock delay:', config.mockDelay);
      break;
      
    case 'default':
      console.log('Setting up with default configuration');
      break;
      
    default:
      const exhaustiveCheck: never = config;
      throw new Error(`Unhandled config type: ${exhaustiveCheck}`);
  }
  
  console.log('Features enabled:', config.features);
  console.log('Timeout set to:', config.timeout);
}
```

## Сложные примеры

### Дискриминированные объединения для системы типов

```typescript
// Объединение для различных типов данных в системе типов
interface StringType {
  type: 'string';
  minLength?: number;
  maxLength?: number;
  pattern?: string;
}

interface NumberType {
  type: 'number';
  min?: number;
  max?: number;
  integer?: boolean;
}

interface BooleanType {
  type: 'boolean';
}

interface ObjectType {
  type: 'object';
  properties: { [key: string]: DataType };
  required?: string[];
}

interface ArrayType {
  type: 'array';
  items: DataType;
  minItems?: number;
  maxItems?: number;
}

interface UnionType {
  type: 'union';
  variants: DataType[];
}

type DataType = 
  | StringType
  | NumberType
  | BooleanType
  | ObjectType
  | ArrayType
  | UnionType;

// Функция для валидации данных на основе типа
function validateData(data: unknown, dataType: DataType): boolean {
  switch (dataType.type) {
    case 'string':
      if (typeof data !== 'string') return false;
      if (dataType.minLength && data.length < dataType.minLength) return false;
      if (dataType.maxLength && data.length > dataType.maxLength) return false;
      if (dataType.pattern && !new RegExp(dataType.pattern).test(data)) return false;
      return true;
      
    case 'number':
      if (typeof data !== 'number') return false;
      if (dataType.min !== undefined && data < dataType.min) return false;
      if (dataType.max !== undefined && data > dataType.max) return false;
      if (dataType.integer && !Number.isInteger(data)) return false;
      return true;
      
    case 'boolean':
      return typeof data === 'boolean';
      
    case 'object':
      if (typeof data !== 'object' || data === null) return false;
      
      const obj = data as Record<string, unknown>;
      for (const [key, propType] of Object.entries(dataType.properties)) {
        if (dataType.required?.includes(key) && !(key in obj)) {
          return false;
        }
        
        if (key in obj && !validateData(obj[key], propType)) {
          return false;
        }
      }
      return true;
      
    case 'array':
      if (!Array.isArray(data)) return false;
      if (dataType.minItems !== undefined && data.length < dataType.minItems) return false;
      if (dataType.maxItems !== undefined && data.length > dataType.maxItems) return false;
      return data.every(item => validateData(item, dataType.items));
      
    case 'union':
      return dataType.variants.some(variant => validateData(data, variant));
      
    default:
      const exhaustiveCheck: never = dataType;
      throw new Error(`Unhandled data type: ${exhaustiveCheck}`);
  }
}
```

### Дискриминированные объединения для событий

```typescript
// Объединение для различных типов событий в приложении
interface UserLoginEvent {
  type: 'user:login';
  userId: number;
  timestamp: number;
  userAgent: string;
}

interface UserLogoutEvent {
  type: 'user:logout';
  userId: number;
  timestamp: number;
  reason?: 'timeout' | 'manual' | 'error';
}

interface PageViewEvent {
  type: 'page:view';
  userId: number;
  url: string;
  referrer?: string;
  timestamp: number;
}

interface ApiCallEvent {
  type: 'api:call';
  userId: number;
  endpoint: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  status: number;
  duration: number;
  timestamp: number;
}

interface ErrorEvent {
  type: 'app:error';
  error: {
    message: string;
    stack?: string;
    type: 'client' | 'server' | 'network';
  };
  timestamp: number;
  userId?: number;
}

type AppEvent = 
  | UserLoginEvent
  | UserLogoutEvent
  | PageViewEvent
  | ApiCallEvent
  | ErrorEvent;

// Обработчик событий
function handleEvent(event: AppEvent) {
  switch (event.type) {
    case 'user:login':
      console.log(`User ${event.userId} logged in at ${new Date(event.timestamp).toISOString()}`);
      break;
      
    case 'user:logout':
      console.log(`User ${event.userId} logged out at ${new Date(event.timestamp).toISOString()}`);
      if (event.reason) {
        console.log(`Reason: ${event.reason}`);
      }
      break;
      
    case 'page:view':
      console.log(`Page view: ${event.url} by user ${event.userId}`);
      if (event.referrer) {
        console.log(`Referrer: ${event.referrer}`);
      }
      break;
      
    case 'api:call':
      console.log(`API call to ${event.endpoint} took ${event.duration}ms with status ${event.status}`);
      break;
      
    case 'app:error':
      console.error(`Application error: ${event.error.message}`);
      if (event.error.stack) {
        console.error(event.error.stack);
      }
      if (event.userId) {
        console.log(`User affected: ${event.userId}`);
      }
      break;
      
    default:
      const exhaustiveCheck: never = event;
      throw new Error(`Unhandled event type: ${exhaustiveCheck}`);
  }
}
```

## Заключение

Дискриминированные объединения - мощный инструмент для создания безопасных и понятных типов в TypeScript. Они особенно полезны при работе с состояниями, действиями, ответами API и другими данными, которые могут принимать несколько различных форм. Правильное использование дискриминированных объединений улучшает безопасность типов и делает код более понятным.

См. также: [[type-guards-advanced]] | [[generic-constraints-advanced]] | [[conditional-types-tricks]]
# Обработка ошибок в TypeScript

Обработка ошибок в TypeScript включает в себя как традиционные JavaScript подходы, так и дополнительные возможности, предоставляемые системой типов TypeScript. Правильная обработка ошибок позволяет создавать более надежные и предсказуемые приложения.

## Основы обработки ошибок

### 1. Традиционные подходы

```typescript
// try-catch блоки
function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error("Деление на ноль недопустимо");
  }
  return a / b;
}

function safeDivide(a: number, b: number): number | null {
  try {
    return divide(a, b);
  } catch (error) {
    console.error("Ошибка деления:", error);
    return null;
  }
}

// Использование
const result1 = safeDivide(10, 2);  // 5
const result2 = safeDivide(10, 0);  // null
```

### 2. Типобезопасные ошибки

```typescript
// Типобезопасный результат с обработкой ошибок
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

function safeOperation<T, E = Error>(
  operation: () => T
): Result<T, E> {
  try {
    const data = operation();
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error as E };
  }
}

// Использование
const result = safeOperation(() => {
  // Опасная операция
  return divide(10, 2);
});

if (result.success) {
  console.log("Результат:", result.data);
} else {
  console.error("Ошибка:", result.error.message);
}
```

## Продвинутые паттерны обработки ошибок

### 1. Паттерн "Either" (альтернатива Result)

```typescript
// Тип Either для представления одного из двух возможных значений
type Either<L, R> = 
  | { type: 'left'; value: L }    // ошибка
  | { type: 'right'; value: R };  // успех

function parseNumber(str: string): Either<string, number> {
  const num = Number(str);
  if (isNaN(num)) {
    return { type: 'left', value: `Невозможно преобразовать "${str}" в число` };
  }
  return { type: 'right', value: num };
}

// Использование
const parsed = parseNumber("42");
if (parsed.type === 'right') {
  console.log("Число:", parsed.value * 2);
} else {
  console.error("Ошибка парсинга:", parsed.value);
}

// Функция для цепочки операций
function chain<L, M, R>(
  either: Either<L, M>,
  fn: (value: M) => Either<L, R>
): Either<L, R> {
  if (either.type === 'left') {
    return either;
  }
  return fn(either.value);
}

// Пример цепочки
const resultChain = chain(
  parseNumber("10"),
  (num) => parseNumber((num * 2).toString())
);

if (resultChain.type === 'right') {
  console.log("Результат:", resultChain.value); // 20
}
```

### 2. Кастомные ошибки с типами

```typescript
// Базовый класс кастомной ошибки
abstract class CustomError extends Error {
  abstract code: string;
  abstract httpStatus: number;
  
  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
  }
}

// Конкретные ошибки
class ValidationError extends CustomError {
  code = 'VALIDATION_ERROR';
  httpStatus = 400;
  
  constructor(
    public field: string,
    message: string
  ) {
    super(message);
    Object.setPrototypeOf(this, ValidationError.prototype);
  }
}

class NotFoundError extends CustomError {
  code = 'NOT_FOUND_ERROR';
  httpStatus = 404;
  
  constructor(
    public resource: string,
    id: string | number
  ) {
    super(`${resource} с ID ${id} не найден`);
    Object.setPrototypeOf(this, NotFoundError.prototype);
  }
}

class NetworkError extends CustomError {
  code = 'NETWORK_ERROR';
  httpStatus = 500;
  
  constructor(
    public url: string,
    public originalError: Error
  ) {
    super(`Ошибка сети при запросе к ${url}: ${originalError.message}`);
    Object.setPrototypeOf(this, NetworkError.prototype);
  }
}

// Функция для проверки типа ошибки
function isErrorOfType<T extends CustomError>(
  error: unknown,
  errorType: new (...args: any[]) => T
): error is T {
  return error instanceof errorType;
}

// Использование
function processUserData(data: any): Result<any, CustomError> {
  if (!data.email) {
    return { 
      success: false, 
      error: new ValidationError('email', 'Email обязателен') 
    };
  }
  
  if (!data.name) {
    return { 
      success: false, 
      error: new ValidationError('name', 'Имя обязательно') 
    };
  }
  
  return { 
    success: true, 
    data: { ...data, id: Date.now() } 
  };
}

// Обработка ошибок
const userData = processUserData({ email: 'test@example.com' });
if (!userData.success) {
  if (isErrorOfType(userData.error, ValidationError)) {
    console.error(`Ошибка валидации поля ${userData.error.field}: ${userData.error.message}`);
  } else if (isErrorOfType(userData.error, NotFoundError)) {
    console.error(`Ошибка 404: ${userData.error.message}`);
  }
}
```

### 3. Типобезопасная обработка асинхронных ошибок

```typescript
// Типобезопасная обработка асинхронных операций
async function asyncOperation<T, E = Error>(
  operation: () => Promise<T>
): Promise<Result<T, E>> {
  try {
    const data = await operation();
    return { success: true, data };
  } catch (error) {
    return { success: false, error: error as E };
  }
}

// Универсальный обработчик асинхронных ошибок
class AsyncErrorHandler {
  static async handle<T, E = Error>(
    operation: () => Promise<T>,
    onError?: (error: E) => void
  ): Promise<Result<T, E>> {
    try {
      const data = await operation();
      return { success: true, data };
    } catch (error) {
      const typedError = error as E;
      if (onError) {
        onError(typedError);
      }
      return { success: false, error: typedError };
    }
  }
  
  // Метод для повторных попыток
  static async retry<T, E = Error>(
    operation: () => Promise<T>,
    maxRetries: number = 3,
    delay: number = 1000
  ): Promise<Result<T, E>> {
    let lastError: E | null = null;
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        const data = await operation();
        return { success: true, data };
      } catch (error) {
        lastError = error as E;
        
        if (attempt === maxRetries) {
          break;
        }
        
        // Задержка перед повторной попыткой
        await new Promise(resolve => setTimeout(resolve, delay * (attempt + 1)));
      }
    }
    
    return { success: false, error: lastError! };
  }
}

// Использование
async function example() {
  const result = await AsyncErrorHandler.handle(
    async () => {
      // Асинхронная операция
      const response = await fetch('/api/data');
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      return response.json();
    },
    (error: Error) => {
      console.error('Ошибка асинхронной операции:', error.message);
    }
  );
  
  if (result.success) {
    console.log('Данные получены:', result.data);
  } else {
    console.error('Ошибка:', result.error);
  }
  
  // Пример с повторными попытками
  const retryResult = await AsyncErrorHandler.retry(
    async () => {
      const response = await fetch('/api/flaky-endpoint');
      if (!response.ok) throw new Error('Network error');
      return response.json();
    },
    3, // максимум 3 попытки
    1000 // задержка 1 секунда
  );
  
  if (retryResult.success) {
    console.log('Данные после повторных попыток:', retryResult.data);
  }
}
```

## Обработка ошибок в API

### 1. Типобезопасные HTTP ошибки

```typescript
// Интерфейсы для HTTP ответов
interface HttpResponse<T> {
  status: number;
  headers: Headers;
  data: T;
}

interface HttpError {
  status: number;
  message: string;
  details?: any;
}

type HttpResult<T> = Result<HttpResponse<T>, HttpError>;

// Клиент API с обработкой ошибок
class ApiClient {
  private baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  async get<T>(endpoint: string): Promise<HttpResult<T>> {
    try {
      const response = await fetch(`${this.baseUrl}${endpoint}`);
      
      const data = await response.json().catch(() => ({}));
      
      if (!response.ok) {
        return {
          success: false,
          error: {
            status: response.status,
            message: response.statusText,
            details: data
          }
        };
      }
      
      return {
        success: true,
        data: {
          status: response.status,
          headers: response.headers,
          data
        }
      };
    } catch (error) {
      return {
        success: false,
        error: {
          status: 500,
          message: error instanceof Error ? error.message : 'Неизвестная ошибка',
          details: null
        }
      };
    }
  }
  
  async post<T, R>(endpoint: string, data: T): Promise<HttpResult<R>> {
    try {
      const response = await fetch(`${this.baseUrl}${endpoint}`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
      });
      
      const responseData = await response.json().catch(() => ({}));
      
      if (!response.ok) {
        return {
          success: false,
          error: {
            status: response.status,
            message: response.statusText,
            details: responseData
          }
        };
      }
      
      return {
        success: true,
        data: {
          status: response.status,
          headers: response.headers,
          data: responseData
        }
      };
    } catch (error) {
      return {
        success: false,
        error: {
          status: 500,
          message: error instanceof Error ? error.message : 'Неизвестная ошибка',
          details: null
        }
      };
    }
  }
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(userId: number): Promise<User | null> {
  const apiClient = new ApiClient('https://api.example.com');
  const result = await apiClient.get<User>(`/users/${userId}`);
  
  if (result.success) {
    return result.data.data;
  } else {
    console.error(`Ошибка получения пользователя ${userId}:`, result.error);
    
    // Обработка специфичных HTTP ошибок
    switch (result.error.status) {
      case 404:
        console.log('Пользователь не найден');
        return null;
      case 500:
        console.log('Внутренняя ошибка сервера');
        return null;
      default:
        console.log('Неизвестная ошибка');
        return null;
    }
  }
}
```

### 2. Middleware для обработки ошибок

```typescript
// Тип для обработчиков ошибок
type ErrorHandler<T = any> = (error: T, context?: any) => void;

// Менеджер обработчиков ошибок
class ErrorManager {
  private handlers: ErrorHandler[] = [];
  
  addHandler(handler: ErrorHandler): void {
    this.handlers.push(handler);
  }
  
  removeHandler(handler: ErrorHandler): void {
    const index = this.handlers.indexOf(handler);
    if (index > -1) {
      this.handlers.splice(index, 1);
    }
  }
  
  async handle(error: any, context?: any): Promise<void> {
    for (const handler of this.handlers) {
      try {
        await handler(error, context);
      } catch (handlerError) {
        console.error('Ошибка в обработчике ошибок:', handlerError);
      }
    }
  }
}

// Примеры обработчиков
const consoleErrorHandler: ErrorHandler = (error, context) => {
  console.error('Ошибка:', error);
  if (context) {
    console.error('Контекст:', context);
  }
};

const logErrorHandler: ErrorHandler = async (error, context) => {
  // Логирование в систему
  console.log(`[ERROR] ${new Date().toISOString()}:`, {
    message: error.message || error,
    stack: error.stack,
    context
  });
};

const notificationErrorHandler: ErrorHandler = async (error, context) => {
  // Отправка уведомлений о критических ошибках
  if (error instanceof CustomError && error.httpStatus >= 500) {
    console.log(`Критическая ошибка: ${error.message}`);
    // Здесь могла бы быть отправка уведомления в Slack, email и т.д.
  }
};

// Использование
const errorManager = new ErrorManager();
errorManager.addHandler(consoleErrorHandler);
errorManager.addHandler(logErrorHandler);
errorManager.addHandler(notificationErrorHandler);

// В приложении
async function riskyOperation() {
  try {
    // Опасная операция
    throw new Error("Что-то пошло не так");
  } catch (error) {
    await errorManager.handle(error, { operation: 'riskyOperation', timestamp: Date.now() });
  }
}
```

## Обработка ошибок в компонентах (React пример)

### 1. Error Boundary с TypeScript

```typescript
import React from 'react';

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class TypedErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ComponentType<{ error: Error }> },
  ErrorBoundaryState
> {
  constructor(props: { children: React.ReactNode; fallback?: React.ComponentType<{ error: Error }> }) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo): void {
    console.error('Error Boundary поймал ошибку:', error, errorInfo);
  }

  render(): React.ReactNode {
    if (this.state.hasError && this.state.error) {
      const Fallback = this.props.fallback || DefaultErrorFallback;
      return <Fallback error={this.state.error} />;
    }

    return this.props.children;
  }
}

const DefaultErrorFallback: React.FC<{ error: Error }> = ({ error }) => (
  <div style={{ padding: '20px', border: '1px solid red', backgroundColor: '#ffe6e6' }}>
    <h2>Произошла ошибка</h2>
    <p>{error.message}</p>
    <button onClick={() => window.location.reload()}>Перезагрузить страницу</button>
  </div>
);

// Использование
const App: React.FC = () => {
  return (
    <TypedErrorBoundary fallback={DefaultErrorFallback}>
      <div>
        {/* Компоненты приложения */}
        <ProblematicComponent />
      </div>
    </TypedErrorBoundary>
  );
};

const ProblematicComponent: React.FC = () => {
  const [shouldThrow, setShouldThrow] = React.useState(false);
  
  if (shouldThrow) {
    throw new Error('Искусственная ошибка для демонстрации');
  }
  
  return (
    <div>
      <p>Работающий компонент</p>
      <button onClick={() => setShouldThrow(true)}>Вызвать ошибку</button>
    </div>
  );
};
```

### 2. Хук для асинхронных операций с обработкой ошибок

```typescript
import { useState, useEffect, useCallback } from 'react';

interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useAsync<T>(
  asyncFunction: () => Promise<T>,
  deps: React.DependencyList = []
): [AsyncState<T>, () => void] {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: false,
    error: null
  });

  const execute = useCallback(() => {
    setState({ data: null, loading: true, error: null });
    
    asyncFunction()
      .then(data => {
        setState({ data, loading: false, error: null });
      })
      .catch(error => {
        setState({
          data: null,
          loading: false,
          error: error instanceof Error ? error.message : 'Неизвестная ошибка'
        });
      });
  }, deps);

  useEffect(() => {
    execute();
  }, deps);

  return [state, execute];
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

const UserComponent: React.FC<{ userId: number }> = ({ userId }) => {
  const [state, refetch] = useAsync<User>(
    () => fetch(`/api/users/${userId}`).then(res => res.json()),
    [userId]
  );

  if (state.loading) return <div>Загрузка...</div>;
  if (state.error) return (
    <div>
      <p>Ошибка: {state.error}</p>
      <button onClick={refetch}>Повторить</button>
    </div>
  );
  if (!state.data) return <div>Нет данных</div>;

  return (
    <div>
      <h2>{state.data.name}</h2>
      <p>{state.data.email}</p>
      <button onClick={refetch}>Обновить</button>
    </div>
  );
};
```

## Практические примеры из реальной разработки

### 1. Система валидации форм с обработкой ошибок

```typescript
// Интерфейс для ошибок валидации
interface ValidationError {
  field: string;
  message: string;
  code: string;
}

// Тип для результата валидации
type ValidationResult<T> = {
  success: true;
  data: T;
} | {
  success: false;
  errors: ValidationError[];
};

// Базовый класс валидатора
abstract class BaseValidator<T> {
  protected errors: ValidationError[] = [];
  
  protected addError(field: string, message: string, code: string = 'VALIDATION_ERROR'): void {
    this.errors.push({ field, message, code });
  }
  
  protected clearErrors(): void {
    this.errors = [];
  }
  
  abstract validate(data: T): ValidationResult<T>;
}

// Валидатор пользователя
interface UserFormData {
  name: string;
  email: string;
  age: number;
  password: string;
}

class UserValidator extends BaseValidator<UserFormData> {
  validate(data: UserFormData): ValidationResult<UserFormData> {
    this.clearErrors();
    
    // Валидация имени
    if (!data.name || data.name.trim().length < 2) {
      this.addError('name', 'Имя должно содержать не менее 2 символов', 'NAME_TOO_SHORT');
    }
    
    // Валидация email
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!data.email || !emailRegex.test(data.email)) {
      this.addError('email', 'Неверный формат email', 'INVALID_EMAIL');
    }
    
    // Валидация возраста
    if (data.age < 18 || data.age > 120) {
      this.addError('age', 'Возраст должен быть от 18 до 120', 'INVALID_AGE');
    }
    
    // Валидация пароля
    if (!data.password || data.password.length < 8) {
      this.addError('password', 'Пароль должен содержать не менее 8 символов', 'PASSWORD_TOO_SHORT');
    }
    
    if (this.errors.length > 0) {
      return { success: false, errors: this.errors };
    }
    
    return { success: true, data };
  }
}

// Использование
const validator = new UserValidator();
const validationResult = validator.validate({
  name: 'John',
  email: 'invalid-email',
  age: 15,
  password: '123'
});

if (!validationResult.success) {
  console.log('Ошибки валидации:');
  validationResult.errors.forEach(error => {
    console.log(`- ${error.field}: ${error.message} (${error.code})`);
  });
} else {
  console.log('Данные валидны:', validationResult.data);
}
```

### 2. Система логирования ошибок

```typescript
// Интерфейсы для логирования
interface LogEntry {
  timestamp: Date;
  level: 'debug' | 'info' | 'warn' | 'error';
  message: string;
  context?: Record<string, any>;
  error?: Error;
}

interface Logger {
  debug(message: string, context?: Record<string, any>): void;
  info(message: string, context?: Record<string, any>): void;
  warn(message: string, context?: Record<string, any>): void;
  error(message: string, error?: Error, context?: Record<string, any>): void;
}

class TypedLogger implements Logger {
  private logEntries: LogEntry[] = [];
  private maxEntries: number;
  
  constructor(maxEntries: number = 1000) {
    this.maxEntries = maxEntries;
  }
  
  debug(message: string, context?: Record<string, any>): void {
    this.log('debug', message, undefined, context);
  }
  
  info(message: string, context?: Record<string, any>): void {
    this.log('info', message, undefined, context);
  }
  
  warn(message: string, context?: Record<string, any>): void {
    this.log('warn', message, undefined, context);
  }
  
  error(message: string, error?: Error, context?: Record<string, any>): void {
    this.log('error', message, error, context);
  }
  
  private log(
    level: LogEntry['level'], 
    message: string, 
    error?: Error, 
    context?: Record<string, any>
  ): void {
    const entry: LogEntry = {
      timestamp: new Date(),
      level,
      message,
      error,
      context
    };
    
    this.logEntries.push(entry);
    
    // Ограничение количества записей
    if (this.logEntries.length > this.maxEntries) {
      this.logEntries = this.logEntries.slice(-this.maxEntries);
    }
    
    // Вывод в консоль
    this.outputToConsole(entry);
  }
  
  private outputToConsole(entry: LogEntry): void {
    const output = `[${entry.level.toUpperCase()}] ${entry.message}`;
    
    switch (entry.level) {
      case 'error':
        if (entry.error) {
          console.error(output, entry.error, entry.context || {});
        } else {
          console.error(output, entry.context || {});
        }
        break;
      case 'warn':
        console.warn(output, entry.context || {});
        break;
      case 'info':
        console.info(output, entry.context || {});
        break;
      case 'debug':
        console.debug(output, entry.context || {});
        break;
    }
  }
  
  getLogs(): LogEntry[] {
    return [...this.logEntries];
  }
  
  getErrors(): LogEntry[] {
    return this.logEntries.filter(entry => entry.level === 'error');
  }
  
  clear(): void {
    this.logEntries = [];
  }
}

// Использование
const logger = new TypedLogger();

logger.info('Приложение запущено');
logger.error('Ошибка подключения к базе данных', new Error('Connection timeout'));
logger.warn('Устаревшая версия API используется', { 
  context: { endpoint: '/api/old', version: 'v1' } 
});

// Получение логов
const errors = logger.getErrors();
console.log('Найдено ошибок:', errors.length);
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки

```typescript
// ОШИБКА: Необработанные исключения в асинхронных функциях
// async function badExample() {
//   throw new Error("Это исключение не будет обработано");
// }

// ЛУЧШЕ: Обработка асинхронных ошибок
async function goodExample(): Promise<Result<void, Error>> {
  try {
    // Асинхронная операция
    await new Promise((resolve, reject) => {
      setTimeout(() => reject(new Error("Ошибка")), 1000);
    });
    return { success: true, data: undefined };
  } catch (error) {
    return { success: false, error: error as Error };
  }
}

// ОШИБКА: Использование any для ошибок
// try {
//   riskyOperation();
// } catch (error: any) { // Не используем any
//   console.log(error.code); // Может вызвать ошибку во время выполнения
// }

// ЛУЧШЕ: Типобезопасная обработка
try {
  riskyOperation();
} catch (error) {
  if (error instanceof Error) {
    console.log(error.message); // Типобезопасно
  }
}
```

### 2. Лучшие практики

```typescript
// 1. Используйте типобезопасные результаты вместо исключений для ожидаемых ошибок
type ApiResponse<T> = Result<T, ApiError>;

// 2. Создавайте специфичные типы ошибок
class ValidationError extends Error {
  constructor(public field: string, message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

// 3. Используйте централизованное логирование
const logger = new TypedLogger();

// 4. Обрабатывайте ошибки на соответствующем уровне абстракции
async function serviceLayer(): Promise<Result<User, ServiceError>> {
  try {
    const data = await dataLayer();
    return { success: true, data };
  } catch (error) {
    logger.error('Ошибка на уровне сервиса', error as Error);
    return { success: false, error: new ServiceError(error) };
  }
}

// 5. Не забывайте обрабатывать ошибки в асинхронных цепочках
async function handleAsyncErrors() {
  const result = await asyncOperation();
  if (result.success) {
    // Обработка успешного результата
  } else {
    // Обработка ошибки
    logger.error('Асинхронная операция завершена с ошибкой', result.error);
  }
}
```

## Лучшие практики

1. **Используйте типобезопасные результаты** - для ожидаемых ошибок вместо исключений
2. **Создавайте специфичные типы ошибок** - для лучшей дебаггинга и обработки
3. **Централизуйте логирование ошибок** - для последовательного мониторинга
4. **Обрабатывайте ошибки на соответствующем уровне** - не перехватывайте ошибки слишком рано
5. **Используйте Error Boundary в UI** - для изоляции ошибок компонентов
6. **Тестируйте сценарии с ошибками** - включая их в unit-тесты

## Связи с другими концепциями

- [[ts/type-system/type-inference|Вывод типов]] - как TypeScript помогает с типизацией ошибок
- [[ts/advanced-types/conditional-types|Условные типы]] - для создания сложных типов ошибок
- [[Утилиты типов TypeScript|Утилиты типов]] - такие как NonNullable, Exclude и другие
- [[ts/async-await/async-typescript-comprehensive|Async/Await]] - обработка асинхронных ошибок
- [[react/error-boundaries]] - обработка ошибок в React компонентах
- [[testing/error-handling]] - тестирование обработки ошибок
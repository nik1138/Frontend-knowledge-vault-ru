---
aliases: ["Примеры типизации для middleware", "TS Middleware Types"]
tags: [typescript, middleware, patterns, architecture]
---

# Примеры типизации для middleware

## Введение

Middleware - важный паттерн в разработке приложений, особенно в контексте систем управления состоянием и обработки HTTP-запросов. Типизация middleware обеспечивает безопасность типов и улучшает понимание потока данных в приложении.

## Базовые паттерны типизации middleware

### Простая цепочка middleware

```typescript
// Тип для действия (в контексте Redux-подобных систем)
interface Action {
  type: string;
  payload?: any;
}

// Тип для состояния
interface State {
  [key: string]: any;
}

// Тип для функции dispatch
type Dispatch = (action: Action) => void;

// Тип для функции next (следующего middleware в цепочке)
type Next = (action: Action) => void;

// Тип для middleware
type Middleware = (api: { dispatch: Dispatch; getState: () => State }) => (next: Next) => (action: Action) => any;

// Пример простого middleware для логирования
const loggerMiddleware: Middleware = (api) => (next) => (action) => {
  console.log('Dispatching action:', action);
  console.log('Previous state:', api.getState());
  
  const result = next(action);
  
  console.log('Next state:', api.getState());
  return result;
};
```

### Типизация middleware с дженериками

```typescript
// Универсальная типизация middleware с дженериками
type GenericMiddleware<S = any, A extends Action = Action, R = any> = (
  api: {
    dispatch: (action: A) => R;
    getState: () => S;
  }
) => (next: (action: A) => R) => (action: A) => R;

// Типизированное middleware для асинхронных операций
const asyncMiddleware: GenericMiddleware<any, Action | ((...args: any[]) => any)> = 
  (api) => (next) => (action) => {
    if (typeof action === 'function') {
      // Если action - функция, вызываем её с dispatch и getState
      return action(api.dispatch, api.getState);
    }
    
    return next(action);
  };
```

## Продвинутые примеры

### Типизация middleware для валидации

```typescript
// Интерфейс для ошибок валидации
interface ValidationError {
  field: string;
  message: string;
}

// Middleware для валидации действий
const validationMiddleware = <T extends Action>(
  validator: (action: T) => ValidationError[] | null
): GenericMiddleware<any, T> => {
  return (api) => (next) => (action) => {
    const errors = validator(action as T);
    
    if (errors && errors.length > 0) {
      console.error('Validation errors:', errors);
      // Можем бросить ошибку или отправить специальное действие
      api.dispatch({
        type: 'VALIDATION_ERROR',
        payload: errors
      });
      return; // Прерываем цепочку
    }
    
    return next(action);
  };
};

// Пример валидатора
interface LoginAction extends Action {
  type: 'LOGIN';
  payload: {
    email: string;
    password: string;
  };
}

const loginValidator = (action: LoginAction): ValidationError[] | null => {
  if (action.type !== 'LOGIN') return null;
  
  const errors: ValidationError[] = [];
  const { email, password } = action.payload;
  
  if (!email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    errors.push({ field: 'email', message: 'Invalid email format' });
  }
  
  if (!password || password.length < 8) {
    errors.push({ field: 'password', message: 'Password must be at least 8 characters' });
  }
  
  return errors.length > 0 ? errors : null;
};

const loginValidationMiddleware = validationMiddleware(loginValidator);
```

### Middleware для обработки асинхронных операций

```typescript
// Типы для асинхронных операций
interface AsyncAction<T, P = any> {
  type: string;
  payload?: P;
  meta?: {
    asyncId: string;
    startTime: number;
  };
}

// Middleware для отслеживания асинхронных операций
const asyncTrackingMiddleware: GenericMiddleware<any, AsyncAction<any>> = 
  (api) => (next) => (action) => {
    if (action.meta?.asyncId) {
      // Отправляем действие о начале асинхронной операции
      api.dispatch({
        type: `${action.type}_START`,
        payload: { asyncId: action.meta.asyncId }
      });
      
      // Отслеживаем завершение
      const result = next(action);
      
      // В реальном приложении здесь может быть логика для отслеживания завершения
      Promise.resolve(result).then(() => {
        api.dispatch({
          type: `${action.type}_SUCCESS`,
          payload: { asyncId: action.meta?.asyncId }
        });
      }).catch((error) => {
        api.dispatch({
          type: `${action.type}_FAILURE`,
          payload: { asyncId: action.meta?.asyncId, error }
        });
      });
      
      return result;
    }
    
    return next(action);
  };
```

### Типизация HTTP middleware (Express-подобное)

```typescript
// Типы для HTTP middleware
type RequestHandler<T = any, ResBody = any, ReqBody = any, ReqQuery = any> = (
  req: Request<T, ResBody, ReqBody, ReqQuery>,
  res: Response<ResBody>,
  next: NextFunction
) => void | Promise<void>;

interface Request<T = any, ResBody = any, ReqBody = any, ReqQuery = any> {
  body: ReqBody;
  query: ReqQuery;
  params: T;
  headers: { [key: string]: string };
  method: string;
  url: string;
}

interface Response<T = any> {
  status(code: number): Response<T>;
  json(data: T): void;
  send(data: any): void;
}

type NextFunction = (err?: any) => void;

// Типизированное middleware для аутентификации
const authMiddleware: RequestHandler<{ userId: string }, any, any, any> = 
  (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      res.status(401).json({ error: 'Authorization token required' });
      return;
    }
    
    // Проверка токена (в реальном приложении)
    const userId = verifyToken(token);
    
    if (!userId) {
      res.status(401).json({ error: 'Invalid token' });
      return;
    }
    
    // Добавляем userId к параметрам запроса
    req.params.userId = userId;
    next();
  };

// Вспомогательная функция для проверки токена
function verifyToken(token: string): string | null {
  // Реализация проверки токена
  return '123'; // В реальном приложении здесь будет логика проверки
}
```

## Практические примеры

### Middleware для кеширования

```typescript
// Интерфейс для кеша
interface Cache {
  get(key: string): any;
  set(key: string, value: any, ttl?: number): void;
  has(key: string): boolean;
  delete(key: string): void;
}

// Middleware для кеширования результатов действий
const createCacheMiddleware = (cache: Cache): GenericMiddleware => {
  return (api) => (next) => (action) => {
    // Проверяем, нужно ли кешировать это действие
    if (action.type.endsWith('_REQUEST') && action.payload?.useCache) {
      const cacheKey = `${action.type}:${JSON.stringify(action.payload)}`;
      
      if (cache.has(cacheKey)) {
        // Если результат в кеше, возвращаем его
        const cachedResult = cache.get(cacheKey);
        api.dispatch({
          type: `${action.type.replace('_REQUEST', '')}_SUCCESS`,
          payload: cachedResult
        });
        return cachedResult;
      }
    }
    
    // Выполняем оригинальное действие
    const result = next(action);
    
    // Если действие успешно и требует кеширования
    if (action.type.endsWith('_SUCCESS') && action.meta?.cacheKey) {
      cache.set(action.meta.cacheKey, result, action.meta.ttl);
    }
    
    return result;
  };
};

// Простая реализация кеша
class SimpleCache implements Cache {
  private store = new Map<string, { value: any; expiry: number }>();
  
  get(key: string) {
    const item = this.store.get(key);
    if (!item) return undefined;
    
    if (Date.now() > item.expiry) {
      this.store.delete(key);
      return undefined;
    }
    
    return item.value;
  }
  
  set(key: string, value: any, ttl: number = 300000) { // 5 минут по умолчанию
    this.store.set(key, { value, expiry: Date.now() + ttl });
  }
  
  has(key: string) {
    return this.get(key) !== undefined;
  }
  
  delete(key: string) {
    this.store.delete(key);
  }
}

const cache = new SimpleCache();
const cacheMiddleware = createCacheMiddleware(cache);
```

### Middleware для обработки ошибок

```typescript
// Типы для обработки ошибок
interface ErrorPayload {
  message: string;
  stack?: string;
  code?: string;
  status?: number;
}

// Middleware для централизованной обработки ошибок
const errorHandlingMiddleware: GenericMiddleware<any, Action & { error?: ErrorPayload }> = 
  (api) => (next) => (action) => {
    try {
      return next(action);
    } catch (error) {
      console.error('Middleware error:', error);
      
      // Отправляем действие об ошибке
      api.dispatch({
        type: 'GLOBAL_ERROR',
        error: {
          message: error instanceof Error ? error.message : 'Unknown error',
          stack: error instanceof Error ? error.stack : undefined,
          code: error instanceof Error ? (error as any).code : undefined
        }
      });
      
      // В реальном приложении можно также отправить ошибку в систему мониторинга
      // reportError(error);
      
      // В зависимости от требований приложения, можно продолжить выполнение или нет
      throw error;
    }
  };

// Middleware для отложенного выполнения действий
const debounceMiddleware = (delay: number = 300): GenericMiddleware => {
  let timeoutId: NodeJS.Timeout | null = null;
  
  return (api) => (next) => (action) => {
    // Если действие должно быть отложено
    if (action.meta?.debounce) {
      if (timeoutId) {
        clearTimeout(timeoutId);
      }
      
      timeoutId = setTimeout(() => {
        next(action);
      }, delay);
      
      // Возвращаем промис, чтобы можно было отменить при необходимости
      return new Promise((resolve) => {
        timeoutId = setTimeout(() => {
          const result = next(action);
          resolve(result);
        }, delay);
      });
    }
    
    return next(action);
  };
};
```

### Middleware для логирования с типами

```typescript
// Интерфейс для конфигурации логирования
interface LoggingConfig {
  level: 'debug' | 'info' | 'warn' | 'error';
  includeState?: boolean;
  includeActions?: boolean;
  filter?: (action: Action) => boolean;
}

// Типизированное middleware для расширенного логирования
const createLoggingMiddleware = (config: LoggingConfig = { level: 'info' }): GenericMiddleware => {
  return (api) => (next) => (action) => {
    // Применяем фильтр, если он задан
    if (config.filter && !config.filter(action)) {
      return next(action);
    }
    
    const startTime = Date.now();
    
    // Логируем начало обработки действия
    if (config.includeActions) {
      console.log(`%c${action.type}`, 'color: #03A9F4; font-weight: bold', action.payload);
    }
    
    const prevState = config.includeState ? api.getState() : undefined;
    
    try {
      const result = next(action);
      const duration = Date.now() - startTime;
      
      // Логируем результат
      if (config.level === 'debug') {
        console.log(`%c${action.type} (in ${duration}ms)`, 'color: #00C851; font-weight: bold', {
          prevState,
          nextState: config.includeState ? api.getState() : undefined,
          action,
          result
        });
      }
      
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      
      console.error(`%c${action.type} (in ${duration}ms)`, 'color: #F44336; font-weight: bold', {
        error: error instanceof Error ? error.message : error,
        prevState,
        action
      });
      
      throw error;
    }
  };
};

// Пример использования
const loggingMiddleware = createLoggingMiddleware({
  level: 'debug',
  includeState: true,
  includeActions: true,
  filter: (action) => !action.type.includes('@@INIT') // Фильтруем служебные действия
});
```

## Заключение

Типизация middleware позволяет создавать более надежные и предсказуемые системы обработки данных. Правильно спроектированные типы обеспечивают безопасность, улучшают автодополнение в IDE и упрощают рефакторинг. Используйте эти паттерны как основу для создания типизированных middleware в ваших приложениях.

См. также: [[state-management-types]] | [[dependency-injection-types]] | [[testing-types]]
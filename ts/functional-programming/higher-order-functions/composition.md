# Комбинирование подходов

Комбинирование функций высшего порядка позволяет создавать мощные и гибкие решения. Это включает в себя комбинацию функций, принимающих другие функции, и функций, возвращающих функции.

## Композиция функций высшего порядка

### Базовая композиция

Композиция функций высшего порядка объединяет несколько функций для создания новой функции с расширенной функциональностью.

```typescript
// Базовая композиция двух функций высшего порядка
const composeHOF = <A, B, C>(
  f: (fn: (a: A) => B) => (a: A) => B,
  g: (fn: (b: B) => C) => (b: B) => C
) => {
  return <T>(fn: (a: A) => C) => 
    f((a: A) => g((b: B) => fn(b))(a));
};

// Пример: комбинация логгирования и тайминга
const withLogging = <T>(fn: (...args: any[]) => T) => 
  (...args: any[]): T => {
    console.log(`Calling function with args: ${JSON.stringify(args)}`);
    const result = fn(...args);
    console.log(`Function returned: ${result}`);
    return result;
  };

const withTiming = <T>(fn: (...args: any[]) => T) => 
  (...args: any[]): T => {
    const start = Date.now();
    const result = fn(...args);
    const end = Date.now();
    console.log(`Function took ${end - start}ms`);
    return result;
  };

// Композиция функций высшего порядка
const withLoggingAndTiming = <T>(fn: (...args: any[]) => T) => 
  withTiming(withLogging(fn));

// Пример использования
const add = (a: number, b: number): number => a + b;
const enhancedAdd = withLoggingAndTiming(add);

console.log(enhancedAdd(2, 3));
// Calling function with args: [2,3]
// Function returned: 5
// Function took 0ms
// 5
```

### Цепочка функций высшего порядка

Цепочка функций высшего порядка позволяет применять несколько преобразований последовательно.

```typescript
// Функция для создания цепочки функций высшего порядка
const chainHOF = <T>(...hofFunctions: Array<(fn: T) => T>) => {
  return (fn: T): T => {
    return hofFunctions.reduceRight((acc, hof) => hof(acc), fn);
  };
};

// Примеры функций высшего порядка
const withValidation = <T>(validator: (args: any[]) => boolean) => 
  (fn: (...args: any[]) => T) => 
    (...args: any[]): T => {
      if (!validator(args)) {
        throw new Error('Validation failed');
      }
      return fn(...args);
    };

const withRetry = <T>(maxRetries: number) => 
  (fn: (...args: any[]) => T) => 
    (...args: any[]): T => {
      let lastError: unknown;
      
      for (let i = 0; i <= maxRetries; i++) {
        try {
          return fn(...args);
        } catch (error) {
          lastError = error;
          
          if (i < maxRetries) {
            console.log(`Retry ${i + 1}/${maxRetries}`);
          }
        }
      }
      
      throw lastError;
    };

const withCache = <T>() => {
  const cache = new Map<string, T>();
  
  return (fn: (...args: any[]) => T) => 
    (...args: any[]): T => {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        console.log('Cache hit');
        return cache.get(key)!;
      }
      
      const result = fn(...args);
      cache.set(key, result);
      console.log('Cache miss');
      return result;
    };
};

// Создание цепочки
const enhancedFunction = chainHOF(
  withLogging,
  withTiming,
  withCache(),
  withRetry(3),
  withValidation((args: any[]) => args.every(arg => typeof arg === 'number'))
);

// Пример использования
const divide = (a: number, b: number): number => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};

const safeDivide = enhancedFunction(divide);

try {
  console.log(safeDivide(10, 2)); // Первый вызов
  console.log(safeDivide(10, 2)); // Второй вызов (из кэша)
  console.log(safeDivide(10, 0)); // Ошибка с повторными попытками
} catch (error) {
  console.error(error);
}
```

## Комбинирование разных типов функций высшего порядка

### Функции, принимающие и возвращающие функции

```typescript
// Функция высшего порядка, принимающая функцию и возвращающая функцию
const createProcessor = <T, U, V>(
  transformer: (value: T) => U
) => (
  handler: (value: U) => V
): (value: T) => V => {
  return (value: T) => handler(transformer(value));
};

// Пример использования
const parseNumber = (str: string): number => {
  const num = Number(str);
  if (isNaN(num)) throw new Error(`Invalid number: ${str}`);
  return num;
};

const formatResult = (num: number): string => `Result: ${num.toFixed(2)}`;

const processString = createProcessor(parseNumber)(formatResult);

console.log(processString('42')); // "Result: 42.00"
console.log(processString('3.14159')); // "Result: 3.14"
```

### Фабрики функций высшего порядка

```typescript
// Фабрика для создания функций высшего порядка
const createMiddlewareFactory = <T>() => {
  return {
    // Функция высшего порядка для логгирования
    createLogger: (prefix: string) => 
      (next: (data: T) => T) => 
        (data: T): T => {
          console.log(`[${prefix}] Before:`, data);
          const result = next(data);
          console.log(`[${prefix}] After:`, result);
          return result;
        },
    
    // Функция высшего порядка для валидации
    createValidator: (predicate: (data: T) => boolean, errorMessage: string) => 
      (next: (data: T) => T) => 
        (data: T): T => {
          if (!predicate(data)) {
            throw new Error(errorMessage);
          }
          return next(data);
        },
    
    // Функция высшего порядка для трансформации
    createTransformer: (transformer: (data: T) => T) => 
      (next: (data: T) => T) => 
        (data: T): T => next(transformer(data))
  };
};

// Пример использования
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
  readonly age: number;
}

const userMiddlewareFactory = createMiddlewareFactory<User>();

const userLogger = userMiddlewareFactory.createLogger('USER');
const userValidator = userMiddlewareFactory.createValidator(
  user => user.age >= 0 && user.age <= 150,
  'Invalid age'
);
const userTransformer = userMiddlewareFactory.createTransformer(
  user => ({ ...user, name: user.name.toUpperCase() })
);

// Комбинирование middleware
const processUser = (user: User): User => {
  console.log('Processing user:', user);
  return { ...user, processed: true } as User & { processed: boolean };
};

const enhancedProcessUser = userLogger(
  userValidator(
    userTransformer(processUser)
  )
);

// Использование
const user: User = { id: 1, name: 'john doe', email: 'john@example.com', age: 30 };

try {
  const result = enhancedProcessUser(user);
  console.log('Result:', result);
} catch (error) {
  console.error('Error:', error);
}
```

## Практические примеры

### Система обработки запросов

```typescript
// Интерфейсы для системы обработки запросов
interface Request {
  readonly method: string;
  readonly url: string;
  readonly headers: Record<string, string>;
  readonly body?: any;
}

interface Response {
  status: number;
  headers: Record<string, string>;
  body?: any;
}

// Тип для обработчика запросов
type RequestHandler = (req: Request, res: Response) => void;

// Тип для middleware
type Middleware = (next: RequestHandler) => RequestHandler;

// Функция для применения middleware
const applyMiddleware = (middlewares: Middleware[]) => 
  (handler: RequestHandler): RequestHandler => {
    return middlewares.reverse().reduce((acc, middleware) => middleware(acc), handler);
  };

// Функции высшего порядка для создания middleware
const createLoggerMiddleware = (): Middleware => 
  (next: RequestHandler) => 
    (req: Request, res: Response) => {
      console.log(`${req.method} ${req.url}`);
      const start = Date.now();
      next(req, res);
      const end = Date.now();
      console.log(`Response: ${res.status} (${end - start}ms)`);
    };

const createAuthMiddleware = (tokenValidator: (token: string) => boolean): Middleware => 
  (next: RequestHandler) => 
    (req: Request, res: Response) => {
      const token = req.headers.authorization?.split(' ')[1];
      
      if (!token || !tokenValidator(token)) {
        res.status = 401;
        res.body = { error: 'Unauthorized' };
        return;
      }
      
      next(req, res);
    };

const createCorsMiddleware = (allowedOrigins: string[]): Middleware => 
  (next: RequestHandler) => 
    (req: Request, res: Response) => {
      const origin = req.headers.origin;
      
      if (origin && allowedOrigins.includes(origin)) {
        res.headers['Access-Control-Allow-Origin'] = origin;
      }
      
      next(req, res);
    };

// Создание обработчика с middleware
const apiHandler: RequestHandler = (req, res) => {
  res.status = 200;
  res.body = { message: 'Hello, World!' };
};

const enhancedApiHandler = applyMiddleware([
  createLoggerMiddleware(),
  createCorsMiddleware(['http://localhost:3000']),
  createAuthMiddleware(token => token === 'valid-token')
])(apiHandler);

// Использование
const request: Request = {
  method: 'GET',
  url: '/api/hello',
  headers: {
    'authorization': 'Bearer valid-token',
    'origin': 'http://localhost:3000'
  }
};

const response: Response = { status: 200, headers: {} };

enhancedApiHandler(request, response);
console.log(response);
```

### Система валидации данных

```typescript
// Типы для системы валидации
type ValidationError = string;
type ValidationResult<T> = 
  | { isValid: true; value: T }
  | { isValid: false; errors: ValidationError[] };

type Validator<T> = (value: T) => ValidationResult<T>;

// Функции высшего порядка для создания валидаторов
const createRequiredValidator = <T>(fieldName: string): Validator<T | undefined> => 
  (value: T | undefined) => {
    if (value === undefined || value === null) {
      return { isValid: false, errors: [`${fieldName} is required`] };
    }
    
    return { isValid: true, value };
  };

const createMinLengthValidator = (minLength: number, fieldName: string): Validator<string> => 
  (value: string) => {
    if (value.length < minLength) {
      return { isValid: false, errors: [`${fieldName} must be at least ${minLength} characters`] };
    }
    
    return { isValid: true, value };
  };

const createPatternValidator = (pattern: RegExp, fieldName: string): Validator<string> => 
  (value: string) => {
    if (!pattern.test(value)) {
      return { isValid: false, errors: [`${fieldName} has invalid format`] };
    }
    
    return { isValid: true, value };
  };

const createCompositeValidator = <T>(...validators: Validator<T>[]): Validator<T> => 
  (value: T) => {
    const results = validators.map(validator => validator(value));
    
    const validResults = results.filter(r => r.isValid) as Array<{ isValid: true; value: T }>;
    const invalidResults = results.filter(r => !r.isValid) as Array<{ isValid: false; errors: string[] }>;
    
    if (invalidResults.length > 0) {
      const allErrors = invalidResults.flatMap(r => r.errors);
      return { isValid: false, errors: allErrors };
    }
    
    return validResults[0] || { isValid: true, value };
  };

// Создание специализированных валидаторов
const validateEmail = createCompositeValidator(
  createRequiredValidator('Email'),
  createPatternValidator(/^[^\s@]+@[^\s@]+\.[^\s@]+$/, 'Email')
);

const validatePassword = createCompositeValidator(
  createRequiredValidator('Password'),
  createMinLengthValidator(8, 'Password')
);

const validateUsername = createCompositeValidator(
  createRequiredValidator('Username'),
  createMinLengthValidator(3, 'Username'),
  createPatternValidator(/^[a-zA-Z0-9_]+$/, 'Username')
);

// Комбинирование валидаторов для сложных объектов
interface UserInput {
  username: string;
  email: string;
  password: string;
}

const validateUserInput = (input: UserInput): ValidationResult<UserInput> => {
  const usernameValidation = validateUsername(input.username);
  if (!usernameValidation.isValid) {
    return { isValid: false, errors: usernameValidation.errors };
  }
  
  const emailValidation = validateEmail(input.email);
  if (!emailValidation.isValid) {
    return { isValid: false, errors: emailValidation.errors };
  }
  
  const passwordValidation = validatePassword(input.password);
  if (!passwordValidation.isValid) {
    return { isValid: false, errors: passwordValidation.errors };
  }
  
  return { isValid: true, value: input };
};

// Использование
const validInput: UserInput = {
  username: 'john_doe',
  email: 'john@example.com',
  password: 'password123'
};

const invalidInput: UserInput = {
  username: 'jo',
  email: 'invalid-email',
  password: '123'
};

console.log(validateUserInput(validInput));
// { isValid: true, value: { username: 'john_doe', email: 'john@example.com', password: 'password123' } }

console.log(validateUserInput(invalidInput));
// { isValid: false, errors: [
//   'Username must be at least 3 characters',
//   'Email has invalid format',
//   'Password must be at least 8 characters'
// ] }
```

### Система обработки событий

```typescript
// Типы для системы событий
type EventHandler<T> = (event: T) => void;

interface EventEmitter<T> {
  on(handler: EventHandler<T>): void;
  off(handler: EventHandler<T>): void;
  emit(event: T): void;
}

// Функции высшего порядка для создания обработчиков событий
const createFilteredHandler = <T>(
  predicate: (event: T) => boolean,
  handler: EventHandler<T>
): EventHandler<T> => {
  return (event: T) => {
    if (predicate(event)) {
      handler(event);
    }
  };
};

const createDebouncedHandler = <T>(
  delay: number,
  handler: EventHandler<T>
): EventHandler<T> => {
  let timeoutId: NodeJS.Timeout;
  
  return (event: T) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => handler(event), delay);
  };
};

const createThrottledHandler = <T>(
  delay: number,
  handler: EventHandler<T>
): EventHandler<T> => {
  let lastExecution = 0;
  
  return (event: T) => {
    const now = Date.now();
    
    if (now - lastExecution >= delay) {
      handler(event);
      lastExecution = now;
    }
  };
};

const createMappedHandler = <T, U>(
  mapper: (event: T) => U,
  handler: EventHandler<U>
): EventHandler<T> => {
  return (event: T) => {
    handler(mapper(event));
  };
};

// Реализация EventEmitter
class SimpleEventEmitter<T> implements EventEmitter<T> {
  private handlers: EventHandler<T>[] = [];
  
  on(handler: EventHandler<T>): void {
    this.handlers.push(handler);
  }
  
  off(handler: EventHandler<T>): void {
    const index = this.handlers.indexOf(handler);
    if (index !== -1) {
      this.handlers.splice(index, 1);
    }
  }
  
  emit(event: T): void {
    this.handlers.forEach(handler => handler(event));
  }
}

// Пример использования
interface MouseEvent {
  readonly type: string;
  readonly x: number;
  readonly y: number;
  readonly timestamp: number;
}

const mouseEventEmitter = new SimpleEventEmitter<MouseEvent>();

// Создание специализированных обработчиков
const clickHandler = createFilteredHandler(
  event => event.type === 'click',
  event => console.log(`Click at (${event.x}, ${event.y})`)
);

const debouncedMoveHandler = createDebouncedHandler(
  100,
  event => console.log(`Mouse moved to (${event.x}, ${event.y})`)
);

const throttledScrollHandler = createThrottledHandler(
  500,
  event => console.log(`Scroll event at ${event.timestamp}`)
);

const mappedPositionHandler = createMappedHandler(
  event => ({ x: event.x, y: event.y }),
  position => console.log(`Position: ${position.x}, ${position.y}`)
);

// Регистрация обработчиков
mouseEventEmitter.on(clickHandler);
mouseEventEmitter.on(debouncedMoveHandler);
mouseEventEmitter.on(throttledScrollHandler);
mouseEventEmitter.on(mappedPositionHandler);

// Вызов событий
mouseEventEmitter.emit({ type: 'click', x: 100, y: 200, timestamp: Date.now() });
mouseEventEmitter.emit({ type: 'move', x: 150, y: 250, timestamp: Date.now() });
mouseEventEmitter.emit({ type: 'scroll', x: 0, y: 100, timestamp: Date.now() });
```

## Преимущества комбинирования подходов

### 1. Модульность и переиспользуемость

```typescript
// Модульные функции высшего порядка
const withErrorHandling = <T>(errorHandler: (error: unknown) => T) => 
  (fn: () => T) => 
    (): T => {
      try {
        return fn();
      } catch (error) {
        return errorHandler(error);
      }
    };

const withDefaultValue = <T>(defaultValue: T) => 
  (fn: () => T | undefined) => 
    (): T => {
      const result = fn();
      return result !== undefined ? result : defaultValue;
    };

const withTransformation = <T, U>(transformer: (value: T) => U) => 
  (fn: () => T) => 
    (): U => transformer(fn());

// Комбинирование для создания специализированных функций
const safeParseInt = withErrorHandling(() => 0)(
  withDefaultValue(0)(
    withTransformation((str: string) => parseInt(str, 10))(
      () => '42'
    )
  )
);

console.log(safeParseInt()); // 42
```

### 2. Гибкость конфигурации

```typescript
// Фабрика для создания конфигурируемых обработчиков
const createConfigurableProcessor = <T, U>(config: {
  validator?: (value: T) => boolean;
  transformer?: (value: T) => U;
  errorHandler?: (error: unknown) => U;
}) => {
  return (processor: (value: T) => U) => 
    (value: T): U => {
      // Валидация
      if (config.validator && !config.validator(value)) {
        throw new Error('Validation failed');
      }
      
      try {
        // Трансформация
        const transformedValue = config.transformer ? config.transformer(value) : value as unknown as U;
        
        // Обработка
        return processor(transformedValue);
      } catch (error) {
        if (config.errorHandler) {
          return config.errorHandler(error);
        }
        throw error;
      }
    };
};

// Создание специализированных процессоров
const numberProcessor = createConfigurableProcessor({
  validator: (value: string) => /^\d+$/.test(value),
  transformer: (value: string) => parseInt(value, 10),
  errorHandler: () => 0
});

const processNumber = numberProcessor((num: number) => num * 2);

console.log(processNumber('42')); // 84
console.log(processNumber('invalid')); // 0 (из-за errorHandler)
```

### 3. Расширяемость

```typescript
// Расширяемая система плагинов
interface Plugin<T> {
  name: string;
  apply: (next: (data: T) => T) => (data: T) => T;
}

const createPluginSystem = <T>() => {
  const plugins: Plugin<T>[] = [];
  
  return {
    addPlugin: (plugin: Plugin<T>) => {
      plugins.push(plugin);
    },
    
    applyPlugins: (handler: (data: T) => T): (data: T) => T => {
      return plugins.reduceRight((acc, plugin) => plugin.apply(acc), handler);
    }
  };
};

// Примеры плагинов
const loggingPlugin: Plugin<string> = {
  name: 'logging',
  apply: (next) => (data) => {
    console.log('Before:', data);
    const result = next(data);
    console.log('After:', result);
    return result;
  }
};

const uppercasePlugin: Plugin<string> = {
  name: 'uppercase',
  apply: (next) => (data) => next(data.toUpperCase())
};

const reversePlugin: Plugin<string> = {
  name: 'reverse',
  apply: (next) => (data) => next(data.split('').reverse().join(''))
};

// Использование
const pluginSystem = createPluginSystem<string>();

pluginSystem.addPlugin(loggingPlugin);
pluginSystem.addPlugin(uppercasePlugin);
pluginSystem.addPlugin(reversePlugin);

const processString = pluginSystem.applyPlugins((str) => `Processed: ${str}`);

console.log(processString('hello'));
// Before: hello
// After: Processed: OLLEH
// Processed: OLLEH
```

## Распространенные ошибки

### 1. Неправильный порядок применения

```typescript
// Ошибка: неправильный порядок применения функций высшего порядка
const withValidationFirst = <T>(validator: (value: T) => boolean) => 
  (fn: (value: T) => T) => 
    (value: T): T => {
      if (!validator(value)) {
        throw new Error('Validation failed');
      }
      return fn(value);
    };

const withLoggingFirst = <T>(fn: (value: T) => T) => 
  (value: T): T => {
    console.log('Before:', value);
    const result = fn(value);
    console.log('After:', result);
    return result;
  };

// Неправильный порядок - логгирование происходит до валидации
const processWithError = withLoggingFirst(
  withValidationFirst((n: number) => n > 0)((n: number) => n * 2)
);

// processWithError(-5); // Логгирование "-5", затем ошибка валидации

// Правильно: валидация должна происходить до логгирования
const processCorrectly = withValidationFirst((n: number) => n > 0)(
  withLoggingFirst((n: number) => n * 2)
);

// processCorrectly(-5); // Ошибка валидации без логгирования
```

### 2. Утечки памяти в замыканиях

```typescript
// Ошибка: потенциальная утечка памяти
const createMemoryLeakFunction = () => {
  const largeDataSet = new Array(1000000).fill('data'); // Большой набор данных
  
  return (processor: (data: string[]) => number) => 
    (): number => processor(largeDataSet); // Замыкание захватывает весь набор данных
};

// Правильно: минимизация захваченных данных
const createEfficientFunction = () => {
  let largeDataSet = new Array(1000000).fill('data');
  
  // Обработка данных
  const processedResult = processLargeDataSet(largeDataSet);
  
  // Освобождение ссылки на большие данные
  largeDataSet = null as any;
  
  return (processor: (data: number) => number) => 
    (): number => processor(processedResult);
};

const processLargeDataSet = (data: string[]): number => {
  return data.length;
};
```

### 3. Непредсказуемое поведение с асинхронными операциями

```typescript
// Ошибка: непредсказуемое поведение с асинхронными функциями высшего порядка
const createAsyncHOFBad = (prefix: string) => {
  return async (fn: () => Promise<string>) => {
    const result = await fn();
    // prefix может измениться к моменту выполнения
    return `${prefix}: ${result}`;
  };
};

// Правильно: захват значений в момент создания
const createAsyncHOFGood = (prefix: string) => {
  const capturedPrefix = prefix; // Явный захват
  
  return async (fn: () => Promise<string>) => {
    const result = await fn();
    return `${capturedPrefix}: ${result}`;
  };
};
```

## Лучшие практики

### 1. Явное управление порядком

```typescript
// Хорошо: явное управление порядком применения
const createPipeline = <T>(...hofFunctions: Array<(fn: T) => T>) => {
  return (fn: T): T => {
    return hofFunctions.reduceRight((acc, hof) => hof(acc), fn);
  };
};

// Плохо: неявный порядок, зависящий от реализации
const applyHOFsBad = <T>(fn: T, ...hofFunctions: Array<(fn: T) => T>): T => {
  return hofFunctions.reduce((acc, hof) => hof(acc), fn);
};
```

### 2. Минимизация побочных эффектов

```typescript
// Хорошо: минимизация побочных эффектов
const createPureHOF = <T>(transformer: (value: T) => T) => 
  (fn: (value: T) => T) => 
    (value: T): T => fn(transformer(value)); // Чистая функция

// Плохо: функции высшего порядка с побочными эффектами
const createImpureHOF = <T>(transformer: (value: T) => T) => 
  (fn: (value: T) => T) => 
    (value: T): T => {
      console.log('Processing value:', value); // Побочный эффект
      const transformed = transformer(value);
      const result = fn(transformed);
      console.log('Result:', result); // Побочный эффект
      return result;
    };
```

### 3. Типобезопасность

```typescript
// Хорошо: строгая типизация
type HOF<T, U> = (fn: (arg: T) => U) => (arg: T) => U;

const createTypedHOF = <T, U>(transformer: (value: T) => T): HOF<T, U> => 
  (fn: (arg: T) => U) => 
    (arg: T): U => fn(transformer(arg));

// Плохо: слабая типизация
const createUntypedHOF = (transformer: Function) => 
  (fn: Function) => 
    (arg: any): any => fn(transformer(arg));
```

## Связи с другими концепциями

- [[ts/functional-programming/higher-order-functions/Функции_высшего_порядка|Функции высшего порядка]] - Основная страница раздела
- [[ts/functional-programming/higher-order-functions/functions-as-arguments|Функции как аргументы]] - Функции, принимающие другие функции
- [[ts/functional-programming/higher-order-functions/functions-as-results|Функции как результаты]] - Функции, возвращающие другие функции
- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/pure-functions|Чистые функции]] - Работа с чистыми функциями
- [[ts/functional-programming/immutability|Неизменяемость]] - Работа с неизменяемыми структурами данных

## Следующие шаги

- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/testing/index|Тестирование]] - Тестирование функций высшего порядка
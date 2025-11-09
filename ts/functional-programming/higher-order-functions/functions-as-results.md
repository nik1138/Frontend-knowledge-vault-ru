# Функции как результаты

Функции, возвращающие другие функции в качестве результата, являются вторым основным видом функций высшего порядка. Они позволяют создавать специализированные функции и реализовывать паттерны замыкания.

## Основные концепции

### Что такое функция как результат?

Функция как результат - это функция, которая возвращает другую функцию. Возвращаемая функция часто "захватывает" переменные из внешней области видимости, создавая замыкание.

```typescript
// Базовый пример функции, возвращающей функцию
const createMultiplier = (factor: number) => {
  return (value: number): number => value * factor;
};

// Пример использования
const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// Более краткая запись
const createAdder = (addend: number) => (value: number): number => value + addend;

const addFive = createAdder(5);
console.log(addFive(10)); // 15
```

## Замыкания

### Понятие замыкания

Замыкание - это комбинация функции и лексического окружения, в котором эта функция была объявлена. Замыкания позволяют функциям "запоминать" переменные из внешней области видимости.

```typescript
// Пример замыкания
const createCounter = () => {
  let count = 0; // Переменная в лексическом окружении
  
  return (): number => {
    return ++count; // Замыкание захватывает переменную count
  };
};

// Пример использования
const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1 (независимый счетчик)
console.log(counter1()); // 3
```

### Практические примеры замыканий

```typescript
// Фабрика функций валидации
const createValidator = <T>(predicate: (value: T) => boolean, errorMessage: string) => {
  return (value: T): { isValid: boolean; error?: string } => {
    const isValid = predicate(value);
    return {
      isValid,
      error: isValid ? undefined : errorMessage
    };
  };
};

// Создание специализированных валидаторов
const validateEmail = createValidator(
  (value: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  'Invalid email format'
);

const validateMinLength = (minLength: number) => 
  createValidator(
    (value: string) => value.length >= minLength,
    `Minimum length is ${minLength}`
  );

const validatePassword = validateMinLength(8);

// Использование
console.log(validateEmail('user@example.com')); 
// { isValid: true }

console.log(validateEmail('invalid-email')); 
// { isValid: false, error: 'Invalid email format' }

console.log(validatePassword('12345')); 
// { isValid: false, error: 'Minimum length is 8' }
```

### Конфигурируемые функции

```typescript
// Функция для создания конфигурируемых обработчиков
const createApiHandler = (config: { baseUrl: string; timeout: number; retries: number }) => {
  return async (endpoint: string, options: RequestInit = {}): Promise<Response> => {
    const url = `${config.baseUrl}${endpoint}`;
    const controller = new AbortController();
    
    // Таймаут
    const timeoutId = setTimeout(() => controller.abort(), config.timeout);
    
    try {
      let lastError: unknown;
      
      for (let i = 0; i <= config.retries; i++) {
        try {
          const response = await fetch(url, {
            ...options,
            signal: controller.signal
          });
          
          clearTimeout(timeoutId);
          return response;
        } catch (error) {
          lastError = error;
          
          if (i < config.retries) {
            // Задержка перед повторной попыткой
            await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
          }
        }
      }
      
      throw lastError;
    } finally {
      clearTimeout(timeoutId);
    }
  };
};

// Создание специализированных обработчиков
const apiHandler = createApiHandler({
  baseUrl: 'https://api.example.com',
  timeout: 5000,
  retries: 3
});

// Использование
// const response = await apiHandler('/users');
// const userData = await response.json();
```

## Каррирование

### Что такое каррирование?

Каррирование - это процесс преобразования функции с несколькими аргументами в последовательность функций с одним аргументом.

```typescript
// Функция без каррирования
const add = (a: number, b: number): number => a + b;
console.log(add(2, 3)); // 5

// Каррированная функция
const curriedAdd = (a: number) => (b: number): number => a + b;
console.log(curriedAdd(2)(3)); // 5

// Частичное применение
const addTwo = curriedAdd(2);
console.log(addTwo(3)); // 5

// Универсальная функция каррирования
const curry = <T extends any[], R>(fn: (...args: T) => R) => 
  (...args: any[]): any => {
    if (args.length >= fn.length) {
      return fn(...args as T);
    }
    
    return (...nextArgs: any[]) => 
      curry(fn)(...args, ...nextArgs);
  };

// Пример использования
const multiply = (a: number, b: number, c: number): number => a * b * c;
const curriedMultiply = curry(multiply);

console.log(curriedMultiply(2)(3)(4)); // 24
console.log(curriedMultiply(2, 3)(4)); // 24
console.log(curriedMultiply(2)(3, 4)); // 24
console.log(curriedMultiply(2, 3, 4)); // 24
```

### Типизация каррированных функций

```typescript
// Типы для каррирования
type Curry<F> = F extends (...args: infer A) => infer R
  ? A extends [infer First, ...infer Rest]
    ? (first: First) => Curry<(...args: Rest) => R>
    : R
  : never;

// Каррированная функция с типизацией
const curryTyped = <F extends (...args: any[]) => any>(fn: F): Curry<F> => 
  ((...args: any[]) => {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    
    return (...nextArgs: any[]) => 
      curryTyped(fn)(...args, ...nextArgs);
  }) as Curry<F>;

// Примеры
const addThree = (a: number, b: number, c: number): number => a + b + c;
const curriedAddThree = curryTyped(addThree);

// Типизированные вызовы
const result1 = curriedAddThree(1)(2)(3); // number
const addOne = curriedAddThree(1); // (b: number) => (c: number) => number
const addOneAndTwo = curriedAddThree(1)(2); // (c: number) => number

// Каррирование с конкретными типами
interface User {
  id: number;
  name: string;
  email: string;
}

const createUser = (id: number, name: string, email: string): User => ({
  id,
  name,
  email
});

const curriedCreateUser = curryTyped(createUser);
const createUserWithId = curriedCreateUser(1);
const createUserWithName = createUserWithId('John Doe');
const user = createUserWithName('john@example.com');
```

## Фабрики функций

### Создание специализированных функций

```typescript
// Фабрика для создания функций форматирования
const createFormatter = (locale: string, options: Intl.DateTimeFormatOptions) => {
  const formatter = new Intl.DateTimeFormat(locale, options);
  
  return (date: Date): string => formatter.format(date);
};

// Создание специализированных форматтеров
const usDateFormatter = createFormatter('en-US', {
  year: 'numeric',
  month: '2-digit',
  day: '2-digit'
});

const euDateTimeFormatter = createFormatter('en-GB', {
  year: 'numeric',
  month: '2-digit',
  day: '2-digit',
  hour: '2-digit',
  minute: '2-digit',
  second: '2-digit'
});

const ruFullFormatter = createFormatter('ru-RU', {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  hour: '2-digit',
  minute: '2-digit'
});

// Использование
const date = new Date('2023-12-25T15:30:00');
console.log(usDateFormatter(date)); // 12/25/2023
console.log(euDateTimeFormatter(date)); // 25/12/2023, 15:30:00
console.log(ruFullFormatter(date)); // понедельник, 25 декабря 2023 г., 15:30
```

### Фабрики для обработки данных

```typescript
// Фабрика для создания функций обработки массивов
const createArrayProcessor = <T, U>(transformations: Array<(item: T) => U>) => {
  return (array: readonly T[]): readonly U[] => {
    return array.reduce((result, item) => {
      return [...result, ...transformations.map(transform => transform(item))];
    }, [] as U[]);
  };
};

// Создание специализированных процессоров
const numberProcessor = createArrayProcessor<number, number | string>([
  n => n * 2,
  n => n.toString(),
  n => n % 2 === 0 ? 'even' : 'odd'
]);

const stringProcessor = createArrayProcessor<string, string>([
  s => s.toUpperCase(),
  s => s.toLowerCase(),
  s => s.charAt(0).toUpperCase() + s.slice(1)
]);

// Использование
const numbers = Object.freeze([1, 2, 3, 4, 5]);
const strings = Object.freeze(['hello', 'world', 'typescript']);

console.log(numberProcessor(numbers)); 
// [2, '1', 'odd', 4, '2', 'even', 6, '3', 'odd', 8, '4', 'even', 10, '5', 'odd']

console.log(stringProcessor(strings));
// ['HELLO', 'hello', 'Hello', 'WORLD', 'world', 'World', 'TYPESCRIPT', 'typescript', 'Typescript']
```

## Композиция функций

### Создание композиторов

```typescript
// Функция для создания композиции двух функций
const compose = <A, B, C>(f: (b: B) => C, g: (a: A) => B) => 
  (a: A): C => f(g(a));

// Функция для создания цепочки функций
const pipe = <T>(...fns: Array<(arg: T) => T>) => 
  (value: T): T => fns.reduce((acc, fn) => fn(acc), value);

// Фабрика для создания специализированных композиторов
const createValidatorChain = <T>(...validators: Array<(value: T) => boolean>) => {
  return (value: T): boolean => validators.every(validator => validator(value));
};

// Создание специализированных валидаторов
const validateUser = createValidatorChain(
  (user: User) => user.name.length > 0,
  (user: User) => user.email.includes('@'),
  (user: User) => user.age >= 0 && user.age <= 150
);

// Использование
const validUser: User = { id: 1, name: 'John Doe', email: 'john@example.com', age: 30 };
const invalidUser: User = { id: 2, name: '', email: 'invalid-email', age: -5 };

console.log(validateUser(validUser)); // true
console.log(validateUser(invalidUser)); // false
```

### Мемоизация как функция высшего порядка

```typescript
// Функция высшего порядка для мемоизации
const memoize = <T extends (...args: any[]) => any>(fn: T): T => {
  const cache = new Map<string, ReturnType<T>>();
  
  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
};

// Создание мемоизированных функций
const expensiveCalculation = memoize((n: number): number => {
  console.log(`Calculating for ${n}`);
  return n * n * n;
});

console.log(expensiveCalculation(5)); // Calculating for 5, 125
console.log(expensiveCalculation(5)); // 125 (без повторного вычисления)
console.log(expensiveCalculation(3)); // Calculating for 3, 27
```

## Паттерны использования

### Stateful Functions (Функции с состоянием)

```typescript
// Функция, возвращающая функции с внутренним состоянием
const createStateMachine = <T>(initialState: T) => {
  let currentState = initialState;
  
  return {
    getState: (): T => currentState,
    setState: (newState: T): void => {
      currentState = newState;
    },
    transition: (reducer: (state: T) => T): T => {
      currentState = reducer(currentState);
      return currentState;
    }
  };
};

// Пример использования
const counterMachine = createStateMachine(0);

console.log(counterMachine.getState()); // 0
counterMachine.setState(5);
console.log(counterMachine.getState()); // 5
counterMachine.transition(state => state + 1);
console.log(counterMachine.getState()); // 6
```

### Configuration Functions (Функции конфигурации)

```typescript
// Функция, возвращающая сконфигурированный обработчик
const createLogger = (config: { level: string; prefix: string }) => {
  return (message: string, level: string = 'info'): void => {
    if (config.level === 'debug' || (config.level === 'info' && level !== 'debug')) {
      console.log(`[${config.prefix}] ${level.toUpperCase()}: ${message}`);
    }
  };
};

// Создание специализированных логгеров
const appLogger = createLogger({ level: 'info', prefix: 'APP' });
const debugLogger = createLogger({ level: 'debug', prefix: 'DEBUG' });

appLogger('Application started'); // [APP] INFO: Application started
debugLogger('Debug information', 'debug'); // [DEBUG] DEBUG: Debug information
```

### Factory Pattern (Фабричный паттерн)

```typescript
// Фабрика для создания обработчиков разных типов
type Handler<T> = (data: T) => void;

const createHandlerFactory = <T>() => {
  const handlers = new Map<string, Handler<T>>();
  
  return {
    register: (type: string, handler: Handler<T>): void => {
      handlers.set(type, handler);
    },
    
    create: (type: string): Handler<T> | undefined => {
      return handlers.get(type);
    },
    
    handle: (type: string, data: T): void => {
      const handler = handlers.get(type);
      if (handler) {
        handler(data);
      }
    }
  };
};

// Пример использования
interface UserEvent {
  type: string;
  userId: number;
  timestamp: Date;
}

const userEventHandlerFactory = createHandlerFactory<UserEvent>();

userEventHandlerFactory.register('login', (event) => {
  console.log(`User ${event.userId} logged in`);
});

userEventHandlerFactory.register('logout', (event) => {
  console.log(`User ${event.userId} logged out`);
});

const loginHandler = userEventHandlerFactory.create('login');
if (loginHandler) {
  loginHandler({ type: 'login', userId: 1, timestamp: new Date() });
}
```

## Преимущества функций как результатов

### 1. Инкапсуляция состояния

```typescript
// Инкапсуляция состояния через замыкания
const createBankAccount = (initialBalance: number) => {
  let balance = initialBalance;
  
  return {
    deposit: (amount: number): void => {
      if (amount > 0) {
        balance += amount;
      }
    },
    
    withdraw: (amount: number): boolean => {
      if (amount > 0 && amount <= balance) {
        balance -= amount;
        return true;
      }
      return false;
    },
    
    getBalance: (): number => balance
  };
};

// Использование
const account = createBankAccount(1000);
account.deposit(500);
console.log(account.getBalance()); // 1500
console.log(account.withdraw(200)); // true
console.log(account.getBalance()); // 1300
// balance недоступна напрямую извне
```

### 2. Специализация функций

```typescript
// Создание специализированных функций
const createSearchFunction = <T>(predicate: (item: T) => boolean) => {
  return (array: readonly T[]): readonly T[] => array.filter(predicate);
};

// Специализированные функции поиска
const findAdults = createSearchFunction((user: User) => user.age >= 18);
const findActiveUsers = createSearchFunction((user: User) => user.active !== false);
const findUsersWithEmail = createSearchFunction((user: User) => user.email.length > 0);

// Использование
const users: readonly User[] = Object.freeze([
  { id: 1, name: 'John', email: 'john@example.com', age: 25, active: true },
  { id: 2, name: 'Jane', email: 'jane@example.com', age: 17, active: true },
  { id: 3, name: 'Bob', email: '', age: 30, active: false }
]);

console.log(findAdults(users)); 
// [{ id: 1, name: 'John', email: 'john@example.com', age: 25, active: true }, 
//  { id: 3, name: 'Bob', email: '', age: 30, active: false }]

console.log(findActiveUsers(users)); 
// [{ id: 1, name: 'John', email: 'john@example.com', age: 25, active: true }, 
//  { id: 2, name: 'Jane', email: 'jane@example.com', age: 17, active: true }]

console.log(findUsersWithEmail(users)); 
// [{ id: 1, name: 'John', email: 'john@example.com', age: 25, active: true }, 
//  { id: 2, name: 'Jane', email: 'jane@example.com', age: 17, active: true }]
```

### 3. Композиция и переиспользуемость

```typescript
// Композиция функций, возвращающих функции
const createMiddleware = <T>(...middlewares: Array<(next: (data: T) => T) => (data: T) => T>) => {
  return (handler: (data: T) => T): (data: T) => T => {
    return middlewares.reverse().reduce((acc, middleware) => middleware(acc), handler);
  };
};

// Примеры middleware
const loggingMiddleware = <T>(next: (data: T) => T) => (data: T): T => {
  console.log('Before:', data);
  const result = next(data);
  console.log('After:', result);
  return result;
};

const timingMiddleware = <T>(next: (data: T) => T) => (data: T): T => {
  const start = Date.now();
  const result = next(data);
  const end = Date.now();
  console.log(`Execution time: ${end - start}ms`);
  return result;
};

// Создание сконфигурированного обработчика
const enhancedHandler = createMiddleware(
  loggingMiddleware,
  timingMiddleware
)((n: number) => n * 2);

console.log(enhancedHandler(5));
// Before: 5
// Execution time: 0ms
// After: 10
// 10
```

## Распространенные ошибки

### 1. Неправильное использование замыканий в циклах

```typescript
// Ошибка: неправильное использование замыканий
const createFunctionsBad = () => {
  const functions: Array<() => number> = [];
  
  for (var i = 0; i < 3; i++) { // var вместо let!
    functions.push(() => i); // Все функции захватывают одно и то же значение i
  }
  
  return functions;
};

const badFunctions = createFunctionsBad();
console.log(badFunctions[0]()); // 3 (а не 0!)
console.log(badFunctions[1]()); // 3 (а не 1!)
console.log(badFunctions[2]()); // 3 (а не 2!)

// Правильно: использование let или IIFE
const createFunctionsGood = () => {
  const functions: Array<() => number> = [];
  
  for (let i = 0; i < 3; i++) { // let создает новую переменную для каждой итерации
    functions.push(() => i);
  }
  
  return functions;
};

const goodFunctions = createFunctionsGood();
console.log(goodFunctions[0]()); // 0
console.log(goodFunctions[1]()); // 1
console.log(goodFunctions[2]()); // 2
```

### 2. Утечки памяти

```typescript
// Ошибка: потенциальная утечка памяти
const createLeakyFunction = () => {
  const largeData = new Array(1000000).fill('data'); // Большой массив
  
  return () => {
    // Функция захватывает largeData, даже если не использует его
    return 'result';
  };
};

// Правильно: освобождение ненужных данных
const createNonLeakyFunction = () => {
  let largeData = new Array(1000000).fill('data');
  
  // Обработка данных
  const result = processLargeData(largeData);
  
  // Освобождение ссылки на большие данные
  largeData = null as any;
  
  return () => result;
};

const processLargeData = (data: any[]) => {
  // Обработка данных
  return data.length;
};
```

### 3. Непредсказуемое поведение с асинхронными операциями

```typescript
// Ошибка: непредсказуемое поведение с асинхронными операциями
const createAsyncFunctionBad = (prefix: string) => {
  return async (message: string): Promise<string> => {
    // Асинхронная операция может захватить измененное значение prefix
    await new Promise(resolve => setTimeout(resolve, 100));
    return `${prefix}: ${message}`;
  };
};

// Правильно: захват значений в момент создания
const createAsyncFunctionGood = (prefix: string) => {
  const capturedPrefix = prefix; // Явный захват значения
  
  return async (message: string): Promise<string> => {
    await new Promise(resolve => setTimeout(resolve, 100));
    return `${capturedPrefix}: ${message}`;
  };
};
```

## Лучшие практики

### 1. Явное захватывание переменных

```typescript
// Хорошо: явное захватывание переменных
const createProcessor = (multiplier: number) => {
  const capturedMultiplier = multiplier; // Явное захватывание
  
  return (value: number): number => value * capturedMultiplier;
};

// Плохо: неявное захватывание
const createProcessorBad = (multiplier: number) => {
  return (value: number): number => value * multiplier; // Неявное захватывание
};
```

### 2. Минимизация захваченных данных

```typescript
// Хорошо: минимизация захваченных данных
const createEfficientFunction = (config: { multiplier: number; offset: number }) => {
  const { multiplier, offset } = config; // Деструктуризация для минимизации
  
  return (value: number): number => value * multiplier + offset;
};

// Плохо: захват всего объекта
const createInefficientFunction = (config: { multiplier: number; offset: number }) => {
  return (value: number): number => value * config.multiplier + config.offset;
};
```

### 3. Чистота возвращаемых функций

```typescript
// Хорошо: возвращаемые функции являются чистыми
const createPureFunction = (factor: number) => {
  return (value: number): number => value * factor; // Чистая функция
};

// Плохо: возвращаемые функции имеют побочные эффекты
const createImpureFunction = (factor: number) => {
  let callCount = 0;
  
  return (value: number): number => {
    callCount++; // Побочный эффект
    console.log(`Called ${callCount} times`); // Побочный эффект
    return value * factor;
  };
};
```

## Связи с другими концепциями

- [[ts/functional-programming/higher-order-functions/Функции_высшего_порядка|Функции высшего порядка]] - Основная страница раздела
- [[ts/functional-programming/higher-order-functions/functions-as-arguments|Функции как аргументы]] - Функции, принимающие другие функции
- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/pure-functions|Чистые функции]] - Работа с чистыми функциями
- [[ts/functional-programming/immutability|Неизменяемость]] - Работа с неизменяемыми структурами данных

## Следующие шаги

- [[ts/functional-programming/higher-order-functions/composition|Комбинирование подходов]] - Сложные функции высшего порядка
- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
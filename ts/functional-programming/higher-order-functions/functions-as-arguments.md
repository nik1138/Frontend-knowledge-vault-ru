# Функции как аргументы

Функции, принимающие другие функции в качестве аргументов, являются одним из основных видов функций высшего порядка. Они позволяют параметризовать поведение и создавать гибкие, переиспользуемые компоненты.

## Основные концепции

### Что такое функция как аргумент?

Функция как аргумент - это функция, которая принимает одну или несколько других функций в качестве параметров. Эти переданные функции часто называют callback-функциями или функциями обратного вызова.

```typescript
// Базовый пример функции, принимающей функцию как аргумент
const applyOperation = <T>(value: T, operation: (val: T) => T): T => {
  return operation(value);
};

// Пример использования
const double = (n: number): number => n * 2;
const square = (n: number): number => n * n;

const result1 = applyOperation(5, double); // 10
const result2 = applyOperation(5, square); // 25
```

## Стандартные функции высшего порядка

### Map

Функция `map` применяет заданную функцию к каждому элементу коллекции и возвращает новую коллекцию с результатами.

```typescript
// Реализация map для массивов
const map = <T, U>(array: readonly T[], fn: (item: T) => U): readonly U[] => {
  const result: U[] = [];
  
  for (let i = 0; i < array.length; i++) {
    result.push(fn(array[i]));
  }
  
  return Object.freeze(result);
};

// Примеры использования
const numbers = Object.freeze([1, 2, 3, 4, 5]);
const doubled = map(numbers, n => n * 2); // [2, 4, 6, 8, 10]
const strings = map(numbers, n => n.toString()); // ['1', '2', '3', '4', '5']

// Работа с объектами
interface User {
  readonly id: number;
  readonly name: string;
  readonly age: number;
}

const users: readonly User[] = Object.freeze([
  { id: 1, name: 'John', age: 30 },
  { id: 2, name: 'Jane', age: 25 },
  { id: 3, name: 'Bob', age: 35 }
]);

const userNames = map(users, user => user.name); // ['John', 'Jane', 'Bob']
const userAges = map(users, user => user.age); // [30, 25, 35]
```

### Filter

Функция `filter` создает новую коллекцию, содержащую только те элементы, которые удовлетворяют заданному предикату.

```typescript
// Реализация filter для массивов
const filter = <T>(array: readonly T[], predicate: (item: T) => boolean): readonly T[] => {
  const result: T[] = [];
  
  for (let i = 0; i < array.length; i++) {
    if (predicate(array[i])) {
      result.push(array[i]);
    }
  }
  
  return Object.freeze(result);
};

// Примеры использования
const numbers = Object.freeze([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

const evens = filter(numbers, n => n % 2 === 0); // [2, 4, 6, 8, 10]
const odds = filter(numbers, n => n % 2 !== 0); // [1, 3, 5, 7, 9]
const greaterThanFive = filter(numbers, n => n > 5); // [6, 7, 8, 9, 10]

// Работа с объектами
const adults = filter(users, user => user.age >= 30); 
// [{ id: 1, name: 'John', age: 30 }, { id: 3, name: 'Bob', age: 35 }]

const nameStartsWithJ = filter(users, user => user.name.startsWith('J'));
// [{ id: 1, name: 'John', age: 30 }, { id: 2, name: 'Jane', age: 25 }]
```

### Reduce

Функция `reduce` применяет функцию к аккумулятору и каждому элементу коллекции (слева направо), возвращая одно результирующее значение.

```typescript
// Реализация reduce для массивов
const reduce = <T, U>(
  array: readonly T[], 
  reducer: (accumulator: U, currentValue: T) => U, 
  initialValue: U
): U => {
  let accumulator = initialValue;
  
  for (let i = 0; i < array.length; i++) {
    accumulator = reducer(accumulator, array[i]);
  }
  
  return accumulator;
};

// Примеры использования
const numbers = Object.freeze([1, 2, 3, 4, 5]);

const sum = reduce(numbers, (acc, n) => acc + n, 0); // 15
const product = reduce(numbers, (acc, n) => acc * n, 1); // 120

// Создание объекта из массива
const userMap = reduce(
  users, 
  (acc, user) => ({ ...acc, [user.id]: user }), 
  {} as Record<number, User>
);
// { 1: { id: 1, name: 'John', age: 30 }, ... }

// Группировка по возрасту
const usersByAge = reduce(
  users,
  (acc, user) => {
    const ageGroup = Math.floor(user.age / 10) * 10;
    return {
      ...acc,
      [ageGroup]: [...(acc[ageGroup] || []), user]
    };
  },
  {} as Record<number, User[]>
);
// { 20: [{ id: 2, name: 'Jane', age: 25 }], 30: [{ id: 1, name: 'John', age: 30 }, { id: 3, name: 'Bob', age: 35 }] }
```

## Пользовательские функции высшего порядка

### Функции для обработки данных

```typescript
// Функция для применения нескольких операций к данным
const pipeOperations = <T>(data: T, ...operations: Array<(data: T) => T>): T => {
  return operations.reduce((result, operation) => operation(result), data);
};

// Пример использования
const processData = (numbers: readonly number[]): readonly number[] => 
  pipeOperations(
    numbers,
    arr => filter(arr, n => n > 0),           // Только положительные числа
    arr => map(arr, n => n * 2),              // Удвоить
    arr => filter(arr, n => n <= 20)          // Только числа <= 20
  );

const input = Object.freeze([-2, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
const result = processData(input); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

### Функции для валидации

```typescript
// Функция для комбинирования валидаторов
const combineValidators = <T>(
  ...validators: Array<(value: T) => boolean>
): (value: T) => boolean => {
  return (value: T) => validators.every(validator => validator(value));
};

// Базовые валидаторы
const isNotEmpty = (value: string): boolean => value.length > 0;
const isEmail = (value: string): boolean => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
const hasMinLength = (min: number) => (value: string): boolean => value.length >= min;
const hasMaxLength = (max: number) => (value: string): boolean => value.length <= max;

// Комбинированные валидаторы
const validateEmail = combineValidators(isNotEmpty, isEmail);
const validatePassword = combineValidators(
  isNotEmpty, 
  hasMinLength(8), 
  hasMaxLength(128)
);

// Пример использования
console.log(validateEmail('user@example.com')); // true
console.log(validateEmail('invalid-email')); // false
console.log(validatePassword('password123')); // true
console.log(validatePassword('123')); // false
```

### Функции для обработки ошибок

```typescript
// Функция для безопасного выполнения операций
const safeExecute = <T, R>(
  operation: (value: T) => R,
  errorHandler: (error: unknown) => R
): (value: T) => R => {
  return (value: T) => {
    try {
      return operation(value);
    } catch (error) {
      return errorHandler(error);
    }
  };
};

// Пример использования
const parseJSON = safeExecute(
  (str: string) => JSON.parse(str),
  (error) => {
    console.error('Failed to parse JSON:', error);
    return null;
  }
);

console.log(parseJSON('{"name": "John"}')); // { name: 'John' }
console.log(parseJSON('invalid json')); // null (с ошибкой в консоли)
```

## Паттерны использования

### Middleware паттерн

```typescript
// Middleware для обработки запросов
interface Request {
  readonly method: string;
  readonly url: string;
  readonly headers: Record<string, string>;
  readonly body?: any;
}

interface Response {
  readonly status: number;
  readonly headers: Record<string, string>;
  readonly body?: any;
}

type Middleware = (
  req: Request, 
  res: Response, 
  next: () => void
) => void;

// Функция для применения middleware
const applyMiddleware = (
  middlewares: Middleware[]
): ((req: Request, res: Response) => void) => {
  return (req: Request, res: Response) => {
    let index = 0;
    
    const next = () => {
      if (index < middlewares.length) {
        const middleware = middlewares[index++];
        middleware(req, res, next);
      }
    };
    
    next();
  };
};

// Примеры middleware
const loggerMiddleware: Middleware = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
};

const authMiddleware: Middleware = (req, res, next) => {
  const token = req.headers.authorization;
  if (token && token.startsWith('Bearer ')) {
    next();
  } else {
    res.status = 401;
  }
};

const errorHandlerMiddleware: Middleware = (req, res, next) => {
  try {
    next();
  } catch (error) {
    res.status = 500;
    res.body = { error: 'Internal Server Error' };
  }
};

// Использование
const requestHandler = applyMiddleware([
  loggerMiddleware,
  authMiddleware,
  errorHandlerMiddleware
]);

const request: Request = {
  method: 'GET',
  url: '/api/users',
  headers: { authorization: 'Bearer token123' }
};

const response: Response = { status: 200, headers: {} };

requestHandler(request, response);
```

### Event Handler паттерн

```typescript
// Система обработки событий
type EventHandler<T> = (event: T) => void;

class EventEmitter<T> {
  private handlers: EventHandler<T>[] = [];
  
  // Добавление обработчика
  on(handler: EventHandler<T>): void {
    this.handlers.push(handler);
  }
  
  // Удаление обработчика
  off(handler: EventHandler<T>): void {
    const index = this.handlers.indexOf(handler);
    if (index !== -1) {
      this.handlers.splice(index, 1);
    }
  }
  
  // Вызов всех обработчиков
  emit(event: T): void {
    this.handlers.forEach(handler => handler(event));
  }
  
  // Функция высшего порядка для создания обработчиков с фильтрацией
  onFiltered(
    predicate: (event: T) => boolean,
    handler: EventHandler<T>
  ): void {
    this.on((event: T) => {
      if (predicate(event)) {
        handler(event);
      }
    });
  }
}

// Пример использования
interface UserEvent {
  readonly type: 'login' | 'logout' | 'update';
  readonly userId: number;
  readonly timestamp: Date;
}

const userEventEmitter = new EventEmitter<UserEvent>();

// Обычные обработчики
userEventEmitter.on(event => {
  console.log(`User ${event.userId} performed ${event.type}`);
});

// Обработчики с фильтрацией
userEventEmitter.onFiltered(
  event => event.type === 'login',
  event => {
    console.log(`User ${event.userId} logged in at ${event.timestamp}`);
  }
);

// Вызов событий
userEventEmitter.emit({
  type: 'login',
  userId: 1,
  timestamp: new Date()
});
```

## Преимущества функций как аргументов

### 1. Гибкость и переиспользуемость

```typescript
// Гибкая функция сортировки
const sortBy = <T>(array: readonly T[], comparator: (a: T, b: T) => number): readonly T[] => {
  return Object.freeze([...array].sort(comparator));
};

// Разные способы сортировки
const numbers = Object.freeze([3, 1, 4, 1, 5, 9, 2, 6]);

const ascending = sortBy(numbers, (a, b) => a - b); // [1, 1, 2, 3, 4, 5, 6, 9]
const descending = sortBy(numbers, (a, b) => b - a); // [9, 6, 5, 4, 3, 2, 1, 1]

const usersByName = sortBy(users, (a, b) => a.name.localeCompare(b.name));
```

### 2. Композиция поведения

```typescript
// Функции для создания специализированных операций
const createMultiplier = (factor: number) => (value: number): number => value * factor;
const createAdder = (addend: number) => (value: number): number => value + addend;

// Композиция функций
const double = createMultiplier(2);
const addTen = createAdder(10);

const processNumber = (n: number): number => addTen(double(n));

console.log(processNumber(5)); // 20 (5 * 2 + 10)
```

### 3. Тестирование

```typescript
// Легко тестируемые функции высшего порядка
const createValidator = <T>(predicate: (value: T) => boolean) => 
  (value: T): boolean => predicate(value);

// Тестирование
const isPositive = createValidator((n: number) => n > 0);
const isEven = createValidator((n: number) => n % 2 === 0);

console.assert(isPositive(5) === true);
console.assert(isPositive(-5) === false);
console.assert(isEven(4) === true);
console.assert(isEven(5) === false);
```

## Распространенные ошибки

### 1. Неправильная передача контекста

```typescript
// Ошибка: потеря контекста
class Calculator {
  private base: number = 10;
  
  add(value: number): number {
    return this.base + value;
  }
}

const calc = new Calculator();
const numbers = Object.freeze([1, 2, 3, 4, 5]);

// Ошибка: this будет undefined
// const results = map(numbers, calc.add); // Ошибка!

// Правильно: привязка контекста
const results = map(numbers, calc.add.bind(calc));
// Или использование стрелочной функции
const results2 = map(numbers, n => calc.add(n));
```

### 2. Мутация данных в callback-функциях

```typescript
// Ошибка: мутация данных
const processArrayBad = <T>(array: readonly T[], processor: (item: T) => T): readonly T[] => {
  const result: T[] = [];
  
  for (let i = 0; i < array.length; i++) {
    const processed = processor(array[i]);
    result.push(processed);
    // Ошибка: если processor мутирует array[i], это нарушает неизменяемость
  }
  
  return Object.freeze(result);
};

// Правильно: предполагаем, что processor не мутирует данные
const processArrayGood = <T>(array: readonly T[], processor: (item: T) => T): readonly T[] => {
  return Object.freeze(array.map(processor));
};
```

### 3. Неправильная обработка асинхронных операций

```typescript
// Ошибка: асинхронные операции в синхронных функциях высшего порядка
const asyncMapBad = <T, U>(array: readonly T[], asyncFn: (item: T) => Promise<U>): readonly U[] => {
  // Это не будет работать, так как map ожидает синхронную функцию
  return array.map(asyncFn); // Возвращает Promise<U>[]
};

// Правильно: отдельная функция для асинхронных операций
const asyncMap = async <T, U>(array: readonly T[], asyncFn: (item: T) => Promise<U>): Promise<readonly U[]> => {
  const results: U[] = [];
  
  for (let i = 0; i < array.length; i++) {
    results.push(await asyncFn(array[i]));
  }
  
  return Object.freeze(results);
};

// Использование
const asyncDouble = async (n: number): Promise<number> => {
  // Имитация асинхронной операции
  return new Promise(resolve => setTimeout(() => resolve(n * 2), 100));
};

// asyncMap([1, 2, 3, 4, 5], asyncDouble).then(results => {
//   console.log(results); // [2, 4, 6, 8, 10]
// });
```

## Лучшие практики

### 1. Явная типизация

```typescript
// Хорошо: явная типизация функций высшего порядка
type Predicate<T> = (value: T) => boolean;
type Mapper<T, U> = (value: T) => U;
type Reducer<T, U> = (accumulator: U, currentValue: T) => U;

const filterTyped = <T>(array: readonly T[], predicate: Predicate<T>): readonly T[] => {
  return Object.freeze(array.filter(predicate));
};

const mapTyped = <T, U>(array: readonly T[], mapper: Mapper<T, U>): readonly U[] => {
  return Object.freeze(array.map(mapper));
};

const reduceTyped = <T, U>(array: readonly T[], reducer: Reducer<T, U>, initialValue: U): U => {
  return array.reduce(reducer, initialValue);
};
```

### 2. Чистота функций

```typescript
// Хорошо: чистые функции высшего порядка
const createAdder = (addend: number) => (value: number): number => value + addend;

// Плохо: функции с побочными эффектами
const createImpureAdder = (addend: number) => {
  let callCount = 0;
  
  return (value: number): number => {
    callCount++; // Побочный эффект
    console.log(`Called ${callCount} times`); // Побочный эффект
    return value + addend;
  };
};
```

### 3. Композиция вместо наследования

```typescript
// Хорошо: композиция функций высшего порядка
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

const enhancedFunction = withTiming(withLogging((a: number, b: number) => a + b));

// Плохо: сложная иерархия классов
class BaseProcessor {
  process(value: number): number {
    return value;
  }
}

class LoggingProcessor extends BaseProcessor {
  process(value: number): number {
    console.log(`Processing ${value}`);
    return super.process(value);
  }
}

class TimingProcessor extends LoggingProcessor {
  process(value: number): number {
    const start = Date.now();
    const result = super.process(value);
    const end = Date.now();
    console.log(`Processing took ${end - start}ms`);
    return result;
  }
}
```

## Связи с другими концепциями

- [[ts/functional-programming/higher-order-functions/Функции_высшего_порядка|Функции высшего порядка]] - Основная страница раздела
- [[ts/functional-programming/higher-order-functions/functions-as-results|Функции как результаты]] - Функции, возвращающие другие функции
- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
- [[ts/functional-programming/pure-functions|Чистые функции]] - Работа с чистыми функций
- [[ts/testing/index|Тестирование]] - Тестирование функций высшего порядка

## Следующие шаги

- [[ts/functional-programming/higher-order-functions/functions-as-results|Функции как результаты]] - Функции, возвращающие другие функции
- [[ts/functional-programming/higher-order-functions/composition|Комбинирование подходов]] - Сложные функции высшего порядка
- [[ts/functional-programming/currying|Каррирование]] - Преобразование функций с несколькими аргументами
- [[ts/functional-programming/composition|Композиция функций]] - Объединение функций для создания новых функций
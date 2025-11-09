# Побочные эффекты

Побочные эффекты - это изменения во внешнем мире, которые происходят при выполнении функции. Отсутствие побочных эффектов - второе ключевое свойство чистых функций.

## Что такое побочные эффекты?

Побочный эффект - это любое взаимодействие функции с внешним миром, которое изменяет состояние программы или окружения. Побочные эффекты делают функции нечистыми и усложняют их тестирование и отладку.

```typescript
// Функция с побочными эффектами
const processUser = (user: User): void => {
  console.log(`Processing user ${user.name}`); // Вывод в консоль
  localStorage.setItem('lastUser', JSON.stringify(user)); // Изменение localStorage
  sendAnalyticsEvent('user_processed', { userId: user.id }); // Отправка аналитики
  user.lastProcessed = new Date(); // Изменение входного параметра
};
```

## Типы побочных эффектов

### 1. Ввод-вывод (I/O)

```typescript
// Чтение из файла
const readFile = (filename: string): string => {
  return fs.readFileSync(filename, 'utf8'); // Побочный эффект
};

// Запись в файл
const writeFile = (filename: string, content: string): void => {
  fs.writeFileSync(filename, content); // Побочный эффект
};

// Сетевые запросы
const fetchData = async (url: string): Promise<any> => {
  const response = await fetch(url); // Побочный эффект
  return response.json();
};

// Вывод в консоль
const logMessage = (message: string): void => {
  console.log(message); // Побочный эффект
};
```

### 2. Изменение внешнего состояния

```typescript
// Глобальные переменные
let globalCounter = 0;
const incrementGlobal = (): number => {
  return ++globalCounter; // Побочный эффект
};

// Изменение DOM
const updateDOM = (elementId: string, content: string): void => {
  const element = document.getElementById(elementId);
  if (element) {
    element.textContent = content; // Побочный эффект
  }
};

// Изменение входных параметров
const mutateUser = (user: User): void => {
  user.lastLogin = new Date(); // Побочный эффект
};
```

### 3. Генерация случайных значений

```typescript
// Случайные числа
const getRandomNumber = (): number => Math.random(); // Побочный эффект

// Случайные строки
const generateUUID = (): string => crypto.randomUUID(); // Побочный эффект

// Текущее время
const getCurrentTime = (): Date => new Date(); // Побочный эффект
```

### 4. Исключения

```typescript
// Функция, которая может выбрасывать исключения
const divide = (a: number, b: number): number => {
  if (b === 0) {
    throw new Error("Division by zero"); // Побочный эффект
  }
  return a / b;
};
```

## Минимизация побочных эффектов

### 1. Разделение чистых и нечистых функций

```typescript
// Нечистая функция с побочными эффектами
const processUserDataImpure = (users: User[]): void => {
  console.log('Processing users...'); // Побочный эффект
  users.forEach(user => {
    user.processed = true; // Изменение входного параметра
    localStorage.setItem(`user_${user.id}`, JSON.stringify(user)); // Побочный эффект
  });
};

// Разделение на чистую и нечистую части
const transformUser = (user: User): User => ({
  ...user,
  processed: true,
  processedAt: new Date()
});

const saveUser = (user: User): void => {
  localStorage.setItem(`user_${user.id}`, JSON.stringify(user));
};

const logProcessing = (): void => {
  console.log('Processing users...');
};

const processUserDataPure = (users: User[]): User[] => 
  users.map(transformUser);

// Композиция чистой и нечистой логики
const processData = (users: User[]): void => {
  logProcessing();
  const transformedUsers = processUserDataPure(users);
  transformedUsers.forEach(saveUser);
};
```

### 2. Использование монад для управления эффектами

```typescript
// Maybe монада для обработки неопределенных значений
type Maybe<T> = T | null | undefined;

const mapMaybe = <T, U>(value: Maybe<T>, fn: (v: T) => U): Maybe<U> => 
  value != null ? fn(value) : null;

const flatMapMaybe = <T, U>(value: Maybe<T>, fn: (v: T) => Maybe<U>): Maybe<U> => 
  value != null ? fn(value) : null;

// Either монада для обработки ошибок
type Either<L, R> = 
  | { type: 'Left'; value: L }
  | { type: 'Right'; value: R };

const left = <L, R>(value: L): Either<L, R> => ({ type: 'Left', value });
const right = <L, R>(value: R): Either<L, R> => ({ type: 'Right', value });

const mapEither = <L, R, S>(
  either: Either<L, R>,
  fn: (value: R) => S
): Either<L, S> => 
  either.type === 'Right' 
    ? right(fn(either.value)) 
    : left(either.value);

const flatMapEither = <L, R, S>(
  either: Either<L, R>,
  fn: (value: R) => Either<L, S>
): Either<L, S> => 
  either.type === 'Right' 
    ? fn(either.value) 
    : left(either.value);

// Пример использования
const safeDivide = (a: number, b: number): Either<string, number> => 
  b === 0 
    ? left("Division by zero") 
    : right(a / b);

const result = flatMapEither(safeDivide(10, 2), x => safeDivide(x, 5));
// Right(1)
```

### 3. Использование IO монады

```typescript
// IO монада для управления побочными эффектами
class IO<T> {
  constructor(private effect: () => T) {}
  
  static of<T>(value: T): IO<T> {
    return new IO(() => value);
  }
  
  static from<T>(effect: () => T): IO<T> {
    return new IO(effect);
  }
  
  map<U>(fn: (value: T) => U): IO<U> {
    return new IO(() => fn(this.effect()));
  }
  
  flatMap<U>(fn: (value: T) => IO<U>): IO<U> {
    return new IO(() => fn(this.effect()).effect());
  }
  
  run(): T {
    return this.effect();
  }
}

// Пример использования
const readLine = (): IO<string> => 
  IO.from(() => {
    // В реальном приложении здесь будет чтение из stdin
    return "Hello, World!";
  });

const writeLine = (message: string): IO<void> => 
  IO.from(() => {
    console.log(message);
  });

const program = readLine()
  .flatMap(input => writeLine(`You entered: ${input}`));

// Побочные эффекты происходят только при вызове run()
// program.run(); // You entered: Hello, World!
```

### 4. Использование функций высшего порядка

```typescript
// Функция высшего порядка для добавления логгирования
const withLogging = <T>(fn: (...args: any[]) => T) => 
  (...args: any[]): T => {
    console.log(`Calling function with args: ${JSON.stringify(args)}`);
    const result = fn(...args);
    console.log(`Function returned: ${result}`);
    return result;
  };

// Пример использования
const add = (a: number, b: number): number => a + b;
const loggedAdd = withLogging(add);

console.log(loggedAdd(2, 3)); 
// Calling function with args: [2,3]
// Function returned: 5
// 5
```

## Преобразование функций с побочными эффектами

### 1. Возврат действий вместо выполнения

```typescript
// Функция с побочными эффектами
const saveToLocalStorageImpure = (key: string, value: any): void => {
  localStorage.setItem(key, JSON.stringify(value));
  console.log(`Saved ${key} to localStorage`);
};

// Чистая версия (возвращает действие вместо выполнения)
interface SaveAction {
  type: 'SAVE_TO_LOCAL_STORAGE';
  key: string;
  value: any;
}

const createSaveAction = (key: string, value: any): SaveAction => ({
  type: 'SAVE_TO_LOCAL_STORAGE',
  key,
  value
});

// Выполнение действия отдельно
const executeSaveAction = (action: SaveAction): void => {
  localStorage.setItem(action.key, JSON.stringify(action.value));
  console.log(`Saved ${action.key} to localStorage`);
};
```

### 2. Использование конфигурации

```typescript
// Функция с побочными эффектами
const fetchDataImpure = async (url: string): Promise<any> => {
  const response = await fetch(url);
  return response.json();
};

// Чистая версия (возвращает Promise без выполнения)
const createFetchPromise = (url: string): Promise<any> => 
  fetch(url).then(response => response.json());

// Или возвращаем описание действия
interface FetchAction {
  type: 'FETCH_DATA';
  url: string;
}

const createFetchAction = (url: string): FetchAction => ({
  type: 'FETCH_DATA',
  url
});
```

### 3. Использование замыканий

```typescript
// Функция с побочными эффектами
const createLoggerImpure = (prefix: string) => 
  (message: string): void => {
    console.log(`${prefix}: ${message}`);
  };

// Чистая версия
const createLogger = (prefix: string) => 
  (message: string): string => 
    `${prefix}: ${message}`;

// Использование
const logger = createLogger('[INFO]');
const logMessage = logger('Application started');
console.log(logMessage); // [INFO]: Application started
```

## Паттерны управления побочными эффектами

### 1. Effect pattern

```typescript
// Тип для описания эффектов
type Effect<T> = () => T;

// Утилиты для работы с эффектами
const runEffect = <T>(effect: Effect<T>): T => effect();

const mapEffect = <T, U>(effect: Effect<T>, fn: (value: T) => U): Effect<U> => 
  () => fn(effect());

const chainEffect = <T, U>(effect: Effect<T>, fn: (value: T) => Effect<U>): Effect<U> => 
  () => fn(effect())();

// Пример использования
const getCurrentTimeEffect: Effect<Date> = () => new Date();

const formatTimeEffect = mapEffect(
  getCurrentTimeEffect,
  date => date.toISOString()
);

console.log(runEffect(formatTimeEffect)); // 2023-01-01T00:00:00.000Z
```

### 2. Command pattern

```typescript
// Интерфейс команды
interface Command<T> {
  execute(): T;
}

// Конкретные команды
class LogCommand implements Command<void> {
  constructor(private message: string) {}
  
  execute(): void {
    console.log(this.message);
  }
}

class SaveToStorageCommand implements Command<void> {
  constructor(
    private key: string,
    private value: any
  ) {}
  
  execute(): void {
    localStorage.setItem(this.key, JSON.stringify(this.value));
  }
}

// Композиция команд
class CompositeCommand implements Command<void> {
  constructor(private commands: Command<void>[]) {}
  
  execute(): void {
    this.commands.forEach(command => command.execute());
  }
}

// Использование
const commands: Command<void>[] = [
  new LogCommand('Starting process...'),
  new SaveToStorageCommand('status', 'running'),
  new LogCommand('Process completed')
];

const process = new CompositeCommand(commands);
process.execute();
```

## Лучшие практики

### 1. Явное разделение чистых и нечистых функций

```typescript
// Хорошо: явное разделение
// Чистая функция
const calculateTotal = (items: { price: number; quantity: number }[]): number =>
  items.reduce((total, item) => total + (item.price * item.quantity), 0);

// Нечистая функция
const saveOrder = async (order: Order): Promise<void> => {
  await fetch('/api/orders', {
    method: 'POST',
    body: JSON.stringify(order)
  });
};

// Композиция
const processOrder = async (items: OrderItem[]): Promise<void> => {
  const total = calculateTotal(items); // Чистая функция
  const order = { items, total, createdAt: new Date() }; // Чистое создание
  await saveOrder(order); // Нечистая функция
};
```

### 2. Минимизация побочных эффектов

```typescript
// Плохо: много побочных эффектов в одной функции
const processUserDataBad = (users: User[]): void => {
  users.forEach(user => {
    console.log(`Processing ${user.name}`);
    user.processed = true;
    user.processedAt = new Date();
    localStorage.setItem(`user_${user.id}`, JSON.stringify(user));
    sendAnalytics(`processed_user_${user.id}`);
  });
};

// Хорошо: минимизация побочных эффектов
const markUserAsProcessed = (user: User): User => ({
  ...user,
  processed: true,
  processedAt: new Date()
});

const processUsersPure = (users: User[]): User[] => 
  users.map(markUserAsProcessed);

const saveUsers = (users: User[]): void => {
  users.forEach(user => {
    localStorage.setItem(`user_${user.id}`, JSON.stringify(user));
  });
};

const logProcessing = (users: User[]): void => {
  users.forEach(user => {
    console.log(`Processing ${user.name}`);
  });
};

const processUserDataGood = (users: User[]): void => {
  logProcessing(users);
  const processedUsers = processUsersPure(users);
  saveUsers(processedUsers);
};
```

### 3. Использование неизменяемых структур

```typescript
// Хорошо: неизменяемость
interface User {
  readonly id: number;
  readonly name: string;
  readonly email: string;
}

const updateUser = (user: User, newName: string): User => ({
  ...user,
  name: newName
});

// Плохо: изменение оригинального объекта
const updateUserImpure = (user: User, newName: string): User => {
  user.name = newName; // Изменение оригинального объекта
  return user;
};
```

## Связи с другими концепциями

- [[ts/functional-programming/pure-functions/Чистые_функции|Чистые функции]] - Основная страница раздела
- [[ts/functional-programming/pure-functions/deterministic-functions|Детерминированные функции]] - Первое свойство чистых функций
- [[ts/functional-programming/immutability|Неизменяемость]] - Работа с неизменяемыми структурами данных
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/testing/index|Тестирование]] - Тестирование функций без побочных эффектов

## Следующие шаги

- [[ts/functional-programming/immutability|Неизменяемость]] - Работа с неизменяемыми структурами данных
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Освоение функций, принимающих или возвращающих другие функции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
# Поток данных в композиции функций

Управление потоком данных - ключевой аспект функционального программирования. Композиция функций позволяет создавать четкие и предсказуемые потоки данных через приложение.

## Содержание

- [Поток данных в композиции функций](#поток-данных-в-композиции-функций)
  - [Содержание](#содержание)
  - [Понятие потока данных](#понятие-потока-данных)
  - [Направления потока](#направления-потока)
  - [Трансформация данных](#трансформация-данных)
  - [Обработка ошибок в потоке](#обработка-ошибок-в-потоке)
  - [Асинхронные потоки](#асинхронные-потоки)
  - [Практические примеры](#практические-примеры)
  - [Паттерны управления потоком](#паттерны-управления-потоком)
    - [1. Middleware паттерн](#1-middleware-паттерн)
    - [2. Chain of Responsibility паттерн](#2-chain-of-responsibility-паттерн)
  - [Лучшие практики](#лучшие-практики)
    - [1. Явное управление потоком](#1-явное-управление-потоком)
    - [2. Единообразие в направлении потока](#2-единообразие-в-направлении-потока)
    - [3. Обработка ошибок на уровне потока](#3-обработка-ошибок-на-уровне-потока)
  - [Связи с другими концепциями](#связи-с-другими-концепциями)
  - [Следующие шаги](#следующие-шаги)

## Понятие потока данных

Поток данных в функциональном программировании - это последовательность преобразований данных, где выход одной функции становится входом следующей функции. Это создает предсказуемый и легко отслеживаемый путь данных через приложение.

```typescript
// Пример потока данных
const inputData = "  Привет, Мир!  ";

// Поток: очистка -> преобразование -> форматирование
const clean = (str: string): string => str.trim();
const toUpperCase = (str: string): string => str.toUpperCase();
const addExclamation = (str: string): string => `${str}!`;

const processText = (text: string): string => 
  addExclamation(toUpperCase(clean(text)));

console.log(processText(inputData)); // "ПРИВЕТ, МИР!!"

// Альтернативная реализация с pipe
const processTextWithPipe = pipe(clean, toUpperCase, addExclamation);
console.log(processTextWithPipe(inputData)); // "ПРИВЕТ, МИР!!"
```

## Направления потока

Существует два основных направления потока данных в композиции:

1. **Справа налево (compose)** - функции применяются в порядке справа налево
2. **Слева направо (pipe)** - функции применяются в порядке слева направо

```typescript
// Справа налево (compose)
const composeFlow = compose(addExclamation, toUpperCase, clean);
console.log(composeFlow(inputData)); // "ПРИВЕТ, МИР!!"

// Слева направо (pipe)
const pipeFlow = pipe(clean, toUpperCase, addExclamation);
console.log(pipeFlow(inputData)); // "ПРИВЕТ, МИР!!"

// Визуализация потока:
// compose(f, g, h)(x) = f(g(h(x)))
// pipe(h, g, f)(x) = f(g(h(x)))

// Пример с более сложным потоком
interface UserInput {
  name: string;
  age: string;
  email: string;
}

interface ProcessedUser {
  name: string;
  age: number;
  email: string;
  isValid: boolean;
}

const parseAge = (input: UserInput): UserInput & { age: number } => ({
  ...input,
  age: parseInt(input.age, 10)
});

const validateEmail = (input: UserInput & { age: number }): UserInput & { age: number, isValid: boolean } => ({
  ...input,
  isValid: input.email.includes("@")
});

const formatName = (input: UserInput & { age: number, isValid: boolean }): ProcessedUser => ({
  ...input,
  name: input.name.trim().toLowerCase()
});

// Поток справа налево
const processUserCompose = compose(formatName, validateEmail, parseAge);

// Поток слева направо
const processUserPipe = pipe(parseAge, validateEmail, formatName);

const userInput: UserInput = {
  name: "  ИВАН  ",
  age: "25",
  email: "ivan@example.com"
};

console.log(processUserCompose(userInput));
console.log(processUserPipe(userInput));
```

## Трансформация данных

Композиция позволяет создавать сложные трансформации данных, комбинируя простые функции.

```typescript
// Пример сложной трансформации данных
interface RawData {
  id: string;
  timestamp: string;
  values: string[];
  metadata: {
    source: string;
    tags: string[];
  };
}

interface ProcessedData {
  id: number;
  date: Date;
  values: number[];
  source: string;
  tagCount: number;
}

// Функции трансформации
const parseId = (data: RawData): RawData & { id: number } => ({
  ...data,
  id: parseInt(data.id, 10)
});

const parseDate = (data: RawData & { id: number }): RawData & { id: number, date: Date } => ({
  ...data,
  date: new Date(data.timestamp)
});

const parseValues = (data: RawData & { id: number, date: Date }): RawData & { id: number, date: Date, values: number[] } => ({
  ...data,
  values: data.values.map(v => parseFloat(v))
});

const extractSource = (data: RawData & { id: number, date: Date, values: number[] }): RawData & { id: number, date: Date, values: number[], source: string } => ({
  ...data,
  source: data.metadata.source
});

const countTags = (data: RawData & { id: number, date: Date, values: number[], source: string }): ProcessedData => ({
  ...data,
  tagCount: data.metadata.tags.length
});

// Композиция трансформаций
const processData = pipe(parseId, parseDate, parseValues, extractSource, countTags);

const rawData: RawData = {
  id: "123",
  timestamp: "2023-01-01T00:00:00Z",
  values: ["1.5", "2.3", "3.7"],
  metadata: {
    source: "sensor-1",
    tags: ["temperature", "indoor", "critical"]
  }
};

console.log(processData(rawData));
// {
//   id: 123,
//   timestamp: "2023-01-01T00:00:00Z",
//   values: [1.5, 2.3, 3.7],
//   metadata: { source: "sensor-1", tags: ["temperature", "indoor", "critical"] },
//   date: 2023-01-01T00:00:00.000Z,
//   source: "sensor-1",
//   tagCount: 3
// }
```

## Обработка ошибок в потоке

Важно учитывать обработку ошибок в потоке данных, особенно при работе с внешними данными.

```typescript
// Either тип для безопасной обработки ошибок
type Either<E, A> = 
  | { _tag: "Left"; left: E }
  | { _tag: "Right"; right: A };

const left = <E, A>(e: E): Either<E, A> => ({ _tag: "Left", left: e });
const right = <E, A>(a: A): Either<E, A> => ({ _tag: "Right", right: a });

// Функции с обработкой ошибок
const safeParseInt = (str: string): Either<string, number> => {
  const result = parseInt(str, 10);
  return isNaN(result) ? left(`Невозможно преобразовать "${str}" в число`) : right(result);
};

const safeParseFloat = (str: string): Either<string, number> => {
  const result = parseFloat(str);
  return isNaN(result) ? left(`Невозможно преобразовать "${str}" в число с плавающей точкой`) : right(result);
};

const validatePositive = (num: number): Either<string, number> => {
  return num > 0 ? right(num) : left(`Число должно быть положительным, получено: ${num}`);
};

// Композиция функций с обработкой ошибок
const processNumberWithErrorHandling = (str: string): Either<string, number> => {
  const parsed = safeParseFloat(str);
  if (parsed._tag === "Left") {
    return parsed;
  }
  
  return validatePositive(parsed.right);
};

console.log(processNumberWithErrorHandling("5.5")); // { _tag: "Right", right: 5.5 }
console.log(processNumberWithErrorHandling("-3")); // { _tag: "Left", left: "Число должно быть положительным, получено: -3" }
console.log(processNumberWithErrorHandling("abc")); // { _tag: "Left", left: "Невозможно преобразовать "abc" в число с плавающей точкой" }

// Расширенная композиция с Either
const composeEither = <E, A, B, C>(
  f: (b: B) => Either<E, C>,
  g: (a: A) => Either<E, B>
): ((a: A) => Either<E, C>) => {
  return (a: A) => {
    const result = g(a);
    if (result._tag === "Left") {
      return result;
    }
    return f(result.right);
  };
};

const processNumberPipeline = composeEither(validatePositive, safeParseFloat);
console.log(processNumberPipeline("10")); // { _tag: "Right", right: 10 }
console.log(processNumberPipeline("-5")); // { _tag: "Left", left: "Число должно быть положительным, получено: -5" }
```

## Асинхронные потоки

Асинхронные потоки данных требуют специальной обработки, так как функции возвращают Promise.

```typescript
// AsyncEither тип для асинхронной обработки ошибок
type AsyncEither<E, A> = Promise<Either<E, A>>;

// Вспомогательные функции
const asyncLeft = <E, A>(e: E): AsyncEither<E, A> => 
  Promise.resolve(left(e));

const asyncRight = <E, A>(a: A): AsyncEither<E, A> => 
  Promise.resolve(right(a));

// Асинхронные функции
const fetchUserData = async (id: number): AsyncEither<string, any> => {
  try {
    // Симуляция API вызова
    if (id <= 0) {
      return asyncLeft("Неверный ID пользователя");
    }
    
    // Симуляция данных пользователя
    return asyncRight({
      id,
      name: `Пользователь ${id}`,
      email: `user${id}@example.com`
    });
  } catch (error) {
    return asyncLeft(`Ошибка при получении данных пользователя: ${error}`);
  }
};

const validateUser = (user: any): Either<string, any> => {
  if (!user.name || !user.email) {
    return left("Неверные данные пользователя");
  }
  return right(user);
};

const formatUser = (user: any): any => ({
  ...user,
  displayName: user.name.toUpperCase()
});

// Композиция асинхронных функций
const processUserAsync = async (id: number): AsyncEither<string, any> => {
  const userData = await fetchUserData(id);
  
  if (userData._tag === "Left") {
    return userData;
  }
  
  const validatedUser = validateUser(userData.right);
  if (validatedUser._tag === "Left") {
    return asyncLeft(validatedUser.left);
  }
  
  return asyncRight(formatUser(validatedUser.right));
};

// Использование
processUserAsync(1).then(result => {
  if (result._tag === "Right") {
    console.log("Обработанный пользователь:", result.right);
  } else {
    console.log("Ошибка:", result.left);
  }
});

processUserAsync(-1).then(result => {
  if (result._tag === "Right") {
    console.log("Обработанный пользователь:", result.right);
  } else {
    console.log("Ошибка:", result.left); // "Неверный ID пользователя"
  }
});
```

## Практические примеры

Рассмотрим практические примеры управления потоком данных в реальных приложениях.

```typescript
// Пример 1: Обработка HTTP запросов
interface HttpRequest {
  method: string;
  url: string;
  headers: Record<string, string>;
  body?: string;
}

interface HttpResponse {
  status: number;
  headers: Record<string, string>;
  body: string;
}

interface ProcessedRequest {
  method: string;
  path: string;
  queryParams: Record<string, string>;
  body?: any;
}

// Функции обработки запросов
const parseUrl = (request: HttpRequest): HttpRequest & { path: string, queryParams: Record<string, string> } => {
  const url = new URL(request.url, "http://localhost");
  const queryParams: Record<string, string> = {};
  
  url.searchParams.forEach((value, key) => {
    queryParams[key] = value;
  });
  
  return {
    ...request,
    path: url.pathname,
    queryParams
  };
};

const parseBody = (request: HttpRequest & { path: string, queryParams: Record<string, string> }): ProcessedRequest => {
  let body: any = undefined;
  
  if (request.body) {
    try {
      body = JSON.parse(request.body);
    } catch (e) {
      // Игнорируем ошибки парсинга
    }
  }
  
  return {
    method: request.method,
    path: request.path,
    queryParams: request.queryParams,
    body
  };
};

const validateMethod = (request: ProcessedRequest): Either<string, ProcessedRequest> => {
  const allowedMethods = ["GET", "POST", "PUT", "DELETE"];
  return allowedMethods.includes(request.method) 
    ? right(request) 
    : left(`Неподдерживаемый метод: ${request.method}`);
};

const addTimestamp = (request: ProcessedRequest): ProcessedRequest & { timestamp: number } => ({
  ...request,
  timestamp: Date.now()
});

// Композиция обработки запросов
const processRequest = (request: HttpRequest): Either<string, ProcessedRequest & { timestamp: number }> => {
  const parsedRequest = pipe(parseUrl, parseBody)(request);
  const validatedRequest = validateMethod(parsedRequest);
  
  if (validatedRequest._tag === "Left") {
    return validatedRequest;
  }
  
  return right(addTimestamp(validatedRequest.right));
};

// Использование
const httpRequest: HttpRequest = {
  method: "POST",
  url: "/api/users?filter=active&limit=10",
  headers: {
    "Content-Type": "application/json"
  },
  body: '{"name": "Иван", "email": "ivan@example.com"}'
};

const result = processRequest(httpRequest);
if (result._tag === "Right") {
  console.log("Обработанный запрос:", result.right);
} else {
  console.log("Ошибка обработки:", result.left);
}

// Пример 2: Обработка данных из формы
interface FormInput {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
  age: string;
}

interface ValidatedUser {
  username: string;
  email: string;
  password: string;
  age: number;
}

// Функции валидации
const validateUsername = (input: FormInput): Either<string, FormInput & { username: string }> => {
  const username = input.username.trim();
  if (username.length < 3) {
    return left("Имя пользователя должно содержать минимум 3 символа");
  }
  return right({ ...input, username });
};

const validateEmail = (input: FormInput & { username: string }): Either<string, FormInput & { username: string, email: string }> => {
  const email = input.email.trim();
  if (!email.includes("@")) {
    return left("Неверный формат email");
  }
  return right({ ...input, email });
};

const validatePassword = (input: FormInput & { username: string, email: string }): Either<string, FormInput & { username: string, email: string, password: string }> => {
  if (input.password !== input.confirmPassword) {
    return left("Пароли не совпадают");
  }
  if (input.password.length < 8) {
    return left("Пароль должен содержать минимум 8 символов");
  }
  return right({ ...input, password: input.password });
};

const validateAge = (input: FormInput & { username: string, email: string, password: string }): Either<string, ValidatedUser> => {
  const age = parseInt(input.age, 10);
  if (isNaN(age) || age < 18) {
    return left("Возраст должен быть числом не менее 18");
  }
  return right({
    username: input.username,
    email: input.email,
    password: input.password,
    age
  });
};

// Композиция валидации формы
const validateForm = (input: FormInput): Either<string, ValidatedUser> => {
  const usernameValidation = validateUsername(input);
  if (usernameValidation._tag === "Left") {
    return usernameValidation;
  }
  
  const emailValidation = validateEmail(usernameValidation.right);
  if (emailValidation._tag === "Left") {
    return emailValidation;
  }
  
  const passwordValidation = validatePassword(emailValidation.right);
  if (passwordValidation._tag === "Left") {
    return passwordValidation;
  }
  
  return validateAge(passwordValidation.right);
};

// Использование
const formInput: FormInput = {
  username: "ivan",
  email: "ivan@example.com",
  password: "password123",
  confirmPassword: "password123",
  age: "25"
};

const validationResult = validateForm(formInput);
if (validationResult._tag === "Right") {
  console.log("Валидные данные:", validationResult.right);
} else {
  console.log("Ошибка валидации:", validationResult.left);
}
```

## Паттерны управления потоком

### 1. Middleware паттерн

```typescript
// Middleware паттерн для управления потоком
interface Middleware<T> {
  (input: T): Either<string, T>;
}

const applyMiddleware = <T>(...middlewares: Middleware<T>[]) => 
  (input: T): Either<string, T> => {
    let result: Either<string, T> = right(input);
    
    for (const middleware of middlewares) {
      if (result._tag === "Left") {
        break;
      }
      
      result = middleware(result.right);
    }
    
    return result;
  };

// Пример использования middleware
const logMiddleware: Middleware<any> = (input) => {
  console.log("Обработка:", input);
  return right(input);
};

const validateMiddleware: Middleware<{ value: number }> = (input) => {
  if (input.value < 0) {
    return left("Значение не может быть отрицательным");
  }
  return right(input);
};

const transformMiddleware: Middleware<{ value: number }> = (input) => {
  return right({ ...input, value: input.value * 2 });
};

const pipeline = applyMiddleware(logMiddleware, validateMiddleware, transformMiddleware);

console.log(pipeline({ value: 5 })); // { _tag: "Right", right: { value: 10 } }
console.log(pipeline({ value: -1 })); // { _tag: "Left", left: "Значение не может быть отрицательным" }
```

### 2. Chain of Responsibility паттерн

```typescript
// Chain of Responsibility паттерн
interface Handler<T> {
  setNext(handler: Handler<T>): Handler<T>;
  handle(request: T): Either<string, T>;
}

abstract class AbstractHandler<T> implements Handler<T> {
  private nextHandler: Handler<T> | null = null;
  
  setNext(handler: Handler<T>): Handler<T> {
    this.nextHandler = handler;
    return handler;
  }
  
  handle(request: T): Either<string, T> {
    if (this.nextHandler) {
      return this.nextHandler.handle(request);
    }
    
    return right(request);
  }
}

class ValidationHandler extends AbstractHandler<{ data: any }> {
  handle(request: { data: any }): Either<string, { data: any }> {
    if (!request.data) {
      return left("Данные отсутствуют");
    }
    
    return super.handle(request);
  }
}

class ProcessingHandler extends AbstractHandler<{ data: any }> {
  handle(request: { data: any }): Either<string, { data: any }> {
    // Обработка данных
    return super.handle(request);
  }
}

class LoggingHandler extends AbstractHandler<{ data: any }> {
  handle(request: { data: any }): Either<string, { data: any }> {
    console.log("Обработаны данные:", request.data);
    return super.handle(request);
  }
}

// Использование
const validationHandler = new ValidationHandler();
const processingHandler = new ProcessingHandler();
const loggingHandler = new LoggingHandler();

validationHandler.setNext(processingHandler).setNext(loggingHandler);

console.log(validationHandler.handle({ data: "test" })); // Успешная обработка
console.log(validationHandler.handle({})); // Ошибка валидации
```

## Лучшие практики

### 1. Явное управление потоком

```typescript
// Хорошо: явное управление потоком
const processUserData = pipe(
  validateInput,
  transformData,
  saveToDatabase,
  sendNotification
);

// Плохо: неявный поток
const processUserDataBad = (input: any) => {
  const validated = validateInput(input);
  const transformed = transformData(validated);
  const saved = saveToDatabase(transformed);
  return sendNotification(saved);
};
```

### 2. Единообразие в направлении потока

```typescript
// Хорошо: единообразное использование pipe
const dataPipeline = pipe(
  fetchData,
  processData,
  formatOutput
);

// Плохо: смешивание compose и pipe
const dataPipelineBad = compose(
  formatOutput,
  pipe(processData, fetchData)
);
```

### 3. Обработка ошибок на уровне потока

```typescript
// Хорошо: централизованная обработка ошибок
const safePipeline = (input: any): Either<string, any> => {
  try {
    return right(pipe(
      validate,
      transform,
      save
    )(input));
  } catch (error) {
    return left(`Ошибка обработки: ${error}`);
  }
};

// Плохо: разрозненная обработка ошибок
const unsafePipeline = (input: any) => {
  const validated = validate(input); // Может бросить исключение
  const transformed = transform(validated); // Может бросить исключение
  return save(transformed); // Может бросить исключение
};
```

## Связи с другими концепциями

- [[ts/functional-programming/composition/Композиция_функций|Композиция функций]] - Основная страница раздела
- [[ts/functional-programming/composition/implementation|Реализация композиции]] - Различные способы реализации композиции функций
- [[ts/functional-programming/higher-order-functions|Функции высшего порядка]] - Функции, принимающие или возвращающие другие функции
- [[ts/functional-programming/currying|Каррирование]] - Техника создания функций с одним аргументом
- [[Обработка ошибок в TypeScript|Обработка ошибок]] - Обработка ошибок в функциональном стиле
- [[Тестирование в TypeScript|Тестирование]] - Тестирование потоков данных

## Следующие шаги

- [[ts/functional-programming/composition/practical-examples|Практические примеры]] - Реальные примеры использования композиции
- [[ts/functional-programming/functors-monads|Функторы и монады]] - Расширенные функциональные концепции
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация потоков данных
- [[Тестирование в TypeScript|Тестирование]] - Тестирование композиции функций
- [[Обработка ошибок в TypeScript|Обработка ошибок]] - Расширенная обработка ошибок в функциональном стиле
# Типы функций в TypeScript

Типы функций - это способ описания сигнатуры функций в TypeScript, включая типы параметров, возвращаемого значения и специфические особенности функций, такие как контекст вызова и обработка аргументов. Понимание типов функций критически важно для создания типобезопасного кода.

## Основные концепции типов функций

### 1. Простые функциональные типы

```typescript
// Определение функционального типа
type GreetFunction = (name: string) => string;

// Реализация функции, соответствующей типу
const greet: GreetFunction = (name) => {
  return `Hello, ${name}!`;
};

// Использование
const message = greet("Alice"); // "Hello, Alice!"
```

### 2. Альтернативные способы определения

```typescript
// С использованием интерфейса
interface GreetInterface {
  (name: string): string;
}

const greetWithInterface: GreetInterface = (name) => {
  return `Hello, ${name}!`;
};

// С использованием типа с параметрами
type AddFunction = (a: number, b: number) => number;

const add: AddFunction = (a, b) => a + b;
```

## Параметры функций

### 1. Обязательные параметры

```typescript
type FullNameFunction = (firstName: string, lastName: string) => string;

const getFullName: FullNameFunction = (firstName, lastName) => {
  return `${firstName} ${lastName}`;
};

// getFullName("John"); // Ошибка: недостаточно аргументов
// getFullName("John", "Doe", "Jr."); // Ошибка: слишком много аргументов
```

### 2. Необязательные параметры

```typescript
type FullNameWithOptionalFunction = (firstName: string, lastName?: string) => string;

const getFullNameWithOptional: FullNameWithOptionalFunction = (firstName, lastName) => {
  if (lastName) {
    return `${firstName} ${lastName}`;
  }
  return firstName;
};

// Использование
console.log(getFullNameWithOptional("John")); // "John"
console.log(getFullNameWithOptional("John", "Doe")); // "John Doe"
```

### 3. Параметры по умолчанию

```typescript
type GreetWithDefaultFunction = (name: string, greeting?: string) => string;

// Обратите внимание: параметр по умолчанию не делает его опциональным в типе
const greetWithDefault = (name: string, greeting: string = "Hello"): string => {
  return `${greeting}, ${name}!`;
};

// Тип для функции с параметром по умолчанию
type GreetWithDefaultType = (name: string, greeting?: string) => string;
```

### 4. Остаточные параметры (rest parameters)

```typescript
type SumFunction = (...numbers: number[]) => number;

const sum: SumFunction = (...numbers) => {
  return numbers.reduce((acc, curr) => acc + curr, 0);
};

// Использование
console.log(sum(1, 2, 3, 4, 5)); // 15
console.log(sum()); // 0
```

## Возвращаемые значения

### 1. Примитивные возвращаемые типы

```typescript
type NumberFunction = () => number;
type StringFunction = () => string;
type BooleanFunction = () => boolean;

const getNumber: NumberFunction = () => 42;
const getString: StringFunction = () => "Hello";
const getBoolean: BooleanFunction = () => true;
```

### 2. Объектные возвращаемые типы

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type UserFactory = (id: number, name: string, email: string) => User;

const createUser: UserFactory = (id, name, email) => {
  return { id, name, email };
};
```

### 3. Объединения возвращаемых типов

```typescript
type DivideFunction = (a: number, b: number) => number | null;

const divide: DivideFunction = (a, b) => {
  if (b === 0) {
    return null;
  }
  return a / b;
};

// Использование
const result = divide(10, 2); // number | null
if (result !== null) {
  console.log(`Результат: ${result}`);
}
```

## Совместимость функций

### 1. Совместимость параметров (контравариантность)

```typescript
interface Animal { name: string; }
interface Dog extends Animal { breed: string; }

type AnimalFunction = (animal: Animal) => void;
type DogFunction = (dog: Dog) => void;

// DogFunction может принимать всё, что может принять Animal
// Поэтому DogFunction может быть присвоена переменной типа AnimalFunction
const animalFunc: AnimalFunction = (animal) => console.log(animal.name);
const dogFunc: DogFunction = (dog) => console.log(dog.breed);

// Это работает благодаря контравариантности параметров
const compatibleFunc: AnimalFunction = (dog: Dog) => console.log(dog.breed); // OK
```

### 2. Совместимость возвращаемых значений (ковариантность)

```typescript
type AnimalProducer = () => Animal;
type DogProducer = () => Dog;

// DogProducer может быть присвоена переменной AnimalProducer
// потому что Dog является подтипом Animal
const produceDog: DogProducer = () => ({ name: "Buddy", breed: "Golden Retriever" });
const produceAnimal: AnimalProducer = produceDog; // OK: ковариантность
```

### 3. Количество параметров

```typescript
type FlexibleFunction = (x: number, y?: number, z?: number) => number;

const flexible: FlexibleFunction = (x, y = 0, z = 0) => x + y + z;

// Все эти вызовы допустимы
console.log(flexible(1)); // 1
console.log(flexible(1, 2)); // 3
console.log(flexible(1, 2, 3)); // 6
```

## Функции обратного вызова (callbacks)

### 1. Простые коллбэки

```typescript
type CallbackFunction = (result: string) => void;

const processData = (data: string, callback: CallbackFunction) => {
  // Обработка данных
  const processed = data.toUpperCase();
  callback(processed);
};

// Использование
processData("hello", (result) => {
  console.log(`Результат: ${result}`); // "Результат: HELLO"
});
```

### 2. Асинхронные коллбэки

```typescript
type AsyncCallback = (error: Error | null, result?: any) => void;

const fetchData = (url: string, callback: AsyncCallback) => {
  // Симуляция асинхронной операции
  setTimeout(() => {
    if (Math.random() > 0.5) {
      callback(null, { id: 1, name: "John" });
    } else {
      callback(new Error("Network error"));
    }
  }, 1000);
};

// Использование
fetchData("/api/user", (error, result) => {
  if (error) {
    console.error("Ошибка:", error.message);
  } else {
    console.log("Данные:", result);
  }
});
```

## Обобщенные функциональные типы

### 1. Простые обобщенные функции

```typescript
type IdentityFunction<T> = (arg: T) => T;

const identity: IdentityFunction<number> = (arg) => arg;
const identityStr: IdentityFunction<string> = (arg) => arg;

// Или функция, которая может работать с любым типом
const genericIdentity: <T>(arg: T) => T = (arg) => arg;
```

### 2. Обобщенные функции с несколькими типами

```typescript
type TransformFunction<T, U> = (arg: T) => U;

const stringToNumber: TransformFunction<string, number> = (str) => parseInt(str);
const numberToString: TransformFunction<number, string> = (num) => num.toString();
const stringToBoolean: TransformFunction<string, boolean> = (str) => str.length > 0;
```

### 3. Обобщенные функции с ограничениями

```typescript
interface Lengthwise {
  length: number;
}

type LengthwiseFunction<T extends Lengthwise> = (arg: T) => number;

const getPropertyLength: LengthwiseFunction<string> = (str) => str.length;
const getArrayLength: LengthwiseFunction<any[]> = (arr) => arr.length;

// const getNumberLength: LengthwiseFunction<number> = (num) => num; // Ошибка: number не имеет length
```

## Практические применения

### 1. Работа с массивами и коллбэками

```typescript
type Predicate<T> = (value: T, index?: number, array?: T[]) => boolean;
type Transformer<T, U> = (value: T, index?: number, array?: T[]) => U;

// Типобезопасная реализация filter
const myFilter = <T>(array: T[], predicate: Predicate<T>): T[] => {
  const result: T[] = [];
  for (let i = 0; i < array.length; i++) {
    if (predicate(array[i], i, array)) {
      result.push(array[i]);
    }
  }
  return result;
};

// Типобезопасная реализация map
const myMap = <T, U>(array: T[], transformer: Transformer<T, U>): U[] => {
  const result: U[] = [];
  for (let i = 0; i < array.length; i++) {
    result.push(transformer(array[i], i, array));
  }
  return result;
};

// Использование
const numbers = [1, 2, 3, 4, 5];
const evenNumbers = myFilter(numbers, n => n % 2 === 0); // [2, 4]
const doubledNumbers = myMap(numbers, n => n * 2); // [2, 4, 6, 8, 10]
```

### 2. Система событий

```typescript
type EventHandler<T = void> = (data: T) => void;

interface EventSystem {
  on<T>(event: string, handler: EventHandler<T>): void;
  emit<T>(event: string, data: T): void;
  off<T>(event: string, handler: EventHandler<T>): void;
}

class SimpleEventSystem implements EventSystem {
  private handlers: { [event: string]: Function[] } = {};
  
  on<T>(event: string, handler: EventHandler<T>) {
    if (!this.handlers[event]) {
      this.handlers[event] = [];
    }
    this.handlers[event].push(handler);
  }
  
  emit<T>(event: string, data: T) {
    const eventHandlers = this.handlers[event];
    if (eventHandlers) {
      eventHandlers.forEach(handler => handler(data));
    }
  }
  
  off<T>(event: string, handler: EventHandler<T>) {
    const eventHandlers = this.handlers[event];
    if (eventHandlers) {
      const index = eventHandlers.indexOf(handler as any);
      if (index > -1) {
        eventHandlers.splice(index, 1);
      }
    }
  }
}
```

### 3. Валидация данных

```typescript
interface Validator<T> {
  (value: T): boolean;
}

interface ValidationError {
  field: string;
  message: string;
}

type ValidationFunction<T> = (value: T) => ValidationError[] | null;

// Создание валидатора с помощью функционального типа
const createValidator = <T>(
  field: string, 
  validator: Validator<T>, 
  message: string
): ValidationFunction<T> => {
  return (value: T) => {
    if (!validator(value)) {
      return [{ field, message }];
    }
    return null;
  };
};

// Использование
const emailValidator = createValidator(
  "email",
  (email: string) => email.includes("@") && email.includes("."),
  "Неверный формат email"
);

const errors = emailValidator("invalid-email"); // [{ field: "email", message: "Неверный формат email" }]
```

## Продвинутые паттерны

### 1. Перегрузки функций

```typescript
// Определение перегрузок
function format(value: string): string;
function format(value: number): string;
function format(value: boolean): string;
// Реализация
function format(value: string | number | boolean): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  } else if (typeof value === "number") {
    return value.toFixed(2);
  } else {
    return value ? "TRUE" : "FALSE";
  }
}

// Использование
const str = format("hello");    // string
const num = format(42);         // string
const bool = format(true);      // string
```

### 2. Типы функций с this

```typescript
interface User {
  name: string;
  age: number;
}

interface UserMethods {
  this: User;
  greet(): string;
  haveBirthday(): void;
}

const user: User = {
  name: "Alice",
  age: 30
};

const greetMethod: UserMethods = function() {
  return `Hello, I'm ${this.name} and I'm ${this.age} years old`;
};

const birthdayMethod: UserMethods = function() {
  this.age += 1;
};

// Привязка методов к пользователю
user["greet"] = greetMethod.bind(user);
user["haveBirthday"] = birthdayMethod.bind(user);
```

### 3. Типы стрелочных функций

```typescript
// Стрелочные функции не имеют собственного this
type ArrowFunction<T, U> = (input: T) => U;

const processString: ArrowFunction<string, number> = (str) => str.length;
const processNumber: ArrowFunction<number, boolean> = (num) => num > 0;

// Сравнение с обычной функцией
type RegularFunction<T, U> = (input: T) => U;
// Для стрелочных и обычных функций типы совпадают в этом случае
```

## Ошибки и лучшие практики

### 1. Часто встречающиеся ошибки

```typescript
// Ошибка: функция ожидает больше параметров, чем предоставлено
type BinaryOperation = (a: number, b: number) => number;
// const op: BinaryOperation = (a) => a; // Ошибка: недостаточно параметров

// Ошибка: возвращаемое значение не соответствует типу
type NumberToBoolean = (n: number) => boolean;
const numToBool: NumberToBoolean = (n) => n; // Ошибка: number несовместим с boolean
```

### 2. Использование void

```typescript
type VoidFunction = () => void;

const logMessage: VoidFunction = () => {
  console.log("Сообщение");
  // Функция не возвращает значение (или неявно возвращает undefined)
};

const returnNumber: () => number = () => 42;
// const invalid: VoidFunction = returnNumber; // Ошибка: number несовместим с void
```

## Практические примеры из реальной разработки

### 1. Работа с API

```typescript
interface ApiRequestConfig {
  url: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  data?: any;
}

interface ApiResponse<T> {
  data: T;
  status: number;
  headers: any;
}

type ApiFunction<T> = (config: ApiRequestConfig) => Promise<ApiResponse<T>>;

const apiCall: ApiFunction<any> = async (config) => {
  const response = await fetch(config.url, {
    method: config.method,
    body: config.data ? JSON.stringify(config.data) : undefined,
    headers: {
      'Content-Type': 'application/json'
    }
  });
  
  const data = await response.json();
  return {
    data,
    status: response.status,
    headers: {}
  };
};

// Использование
interface User {
  id: number;
  name: string;
}

const getUser: ApiFunction<User> = (config) => apiCall<User>(config);
```

### 2. Система middleware

```typescript
interface Context {
  request: any;
  response: any;
  [key: string]: any;
}

type Middleware<T extends Context = Context> = (
  ctx: T,
  next: () => Promise<void>
) => Promise<void> | void;

const loggerMiddleware: Middleware = async (ctx, next) => {
  console.log(`Запрос: ${ctx.request.method} ${ctx.request.url}`);
  await next();
  console.log(`Ответ: ${ctx.response.status}`);
};

const authMiddleware: Middleware = async (ctx, next) => {
  if (!ctx.request.headers.authorization) {
    ctx.response.status = 401;
    ctx.response.body = { error: "Unauthorized" };
    return;
  }
  await next();
};
```

## Лучшие практики

1. **Используйте точные типы для параметров** - избегайте any, когда это возможно
2. **Определяйте типы для коллбэков** - это улучшает автодополнение и проверку типов
3. **Используйте обобщения для универсальных функций** - это делает функции более гибкими
4. **Будьте осторожны с контравариантностью параметров** - особенно при работе с иерархиями типов
5. **Документируйте сложные функциональные типы** - особенно когда они используют обобщения
6. **Предпочитайте стрелочные функции для коллбэков** - чтобы избежать проблем с this

## Связи с другими концепциями

- [[ts/generics/TS Generics|Обобщения]] - основа для универсальных функциональных типов
- [[ts/type-system/type-compatibility|Совместимость типов]] - как функции проверяются на совместимость
- [[ts/advanced-types/conditional-types|Условные типы]] - могут использоваться в функциональных типах
- [[ts/basics/functions|Функции]] - основы работы с функциями в TypeScript
- [[js/functions]] - JavaScript функции, на которых основаны типы
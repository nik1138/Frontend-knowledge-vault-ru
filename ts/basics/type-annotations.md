# Аннотации типов в TypeScript

## Введение

Аннотации типов - это способ явного указания типа переменной, параметра функции, возвращаемого значения или свойства объекта в TypeScript. Они обеспечивают безопасность типов и помогают разработчикам понимать структуру данных в коде. Аннотации типов позволяют TypeScript проверять типы во время компиляции и предоставлять автодополнение в редакторах кода.

## Основные аннотации типов

### Аннотации для переменных

```typescript
// Аннотации для примитивных типов
let name: string = "Алексей";
let age: number = 30;
let isActive: boolean = true;

// Массивы
let numbers: number[] = [1, 2, 3, 4, 5];
let names: string[] = ["Алексей", "Мария", "Иван"];

// Альтернативный синтаксис для массивов
let scores: Array<number> = [85, 92, 78];

// Объекты
let user: { name: string; age: number; isActive: boolean } = {
  name: "Алексей",
  age: 30,
  isActive: true
};
```

### Аннотации для функций

```typescript
// Аннотация параметров и возвращаемого значения
function greet(name: string): string {
  return `Привет, ${name}!`;
}

// Функция без возвращаемого значения
function logMessage(message: string): void {
  console.log(message);
}

// Функция с несколькими параметрами
function add(a: number, b: number): number {
  return a + b;
}

// Функция с необязательным параметром
function buildName(firstName: string, lastName?: string): string {
  if (lastName) {
    return `${firstName} ${lastName}`;
  } else {
    return firstName;
  }
}

// Функция с параметром по умолчанию
function multiply(x: number, y: number = 1): number {
  return x * y;
}

// Функция с rest параметром
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, curr) => acc + curr, 0);
}
```

### Аннотации для объектов и интерфейсов

```typescript
// Использование интерфейса
interface User {
  name: string;
  age: number;
  email: string;
  isActive: boolean;
}

let user: User = {
  name: "Алексей",
  age: 30,
  email: "alex@example.com",
  isActive: true
};

// Вложенные объекты
interface Address {
  street: string;
  city: string;
  country: string;
}

interface FullUser {
  id: number;
  personalInfo: {
    name: string;
    age: number;
  };
  contact: {
    email: string;
    phone: string;
  };
  address: Address;
}

let fullUser: FullUser = {
  id: 1,
  personalInfo: {
    name: "Алексей",
    age: 30
  },
  contact: {
    email: "alex@example.com",
    phone: "+7 (495) 123-45-67"
  },
  address: {
    street: "ул. Примерная, 1",
    city: "Москва",
    country: "Россия"
  }
};
```

## Сложные аннотации типов

### Объединения типов (Union Types)

```typescript
// Переменная может быть строкой или числом
let value: string | number;
value = "строка";
value = 42;

// Использование в функциях
function formatValue(input: string | number): string {
  if (typeof input === "string") {
    return input.toUpperCase();
  } else {
    return input.toFixed(2);
  }
}

// Объединения с литеральными типами
type Theme = "light" | "dark";
type HttpStatus = 200 | 201 | 400 | 401 | 404 | 500;

function applyTheme(theme: Theme): void {
  console.log(`Применение темы: ${theme}`);
}
```

### Псевдонимы типов (Type Aliases)

```typescript
// Создание псевдонима для сложного типа
type UserIdentifier = string | number;

function getUserById(id: UserIdentifier): User | null {
  // Реализация получения пользователя
  return null;
}

// Псевдоним для функционального типа
type Callback = (error: Error | null, result?: any) => void;

// Псевдоним для сложного объектного типа
type ApiResponse<T> = {
  success: boolean;
  data?: T;
  error?: string;
  timestamp: Date;
};

// Использование обобщенного псевдонима
type UserResponse = ApiResponse<User>;
```

### Обобщения (Generics)

```typescript
// Обобщенная функция
function identity<T>(arg: T): T {
  return arg;
}

// Использование обобщенной функции
let output1 = identity<string>("привет"); // Тип: string
let output2 = identity<number>(42);       // Тип: number

// Обобщенные интерфейсы
interface GenericContainer<T> {
  value: T;
  isValid: boolean;
}

let stringContainer: GenericContainer<string> = {
  value: "тест",
  isValid: true
};

let numberContainer: GenericContainer<number> = {
  value: 123,
  isValid: false
};

// Обобщенные классы
class GenericList<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  get(index: number): T | undefined {
    return this.items[index];
  }

  size(): number {
    return this.items.length;
  }
}
```

## Практические примеры

### Аннотации для асинхронных функций

```typescript
// Аннотации для Promise
async function fetchUserData(id: number): Promise<User> {
  // Симуляция асинхронного запроса
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        name: "Алексей",
        age: 30,
        email: "alex@example.com",
        isActive: true
      });
    }, 1000);
  });
}

// Использование async/await с аннотациями
async function processUser(id: number): Promise<string> {
  try {
    const user: User = await fetchUserData(id);
    return `Пользователь: ${user.name}`;
  } catch (error: unknown) {
    if (error instanceof Error) {
      throw new Error(`Ошибка получения пользователя: ${error.message}`);
    }
    throw new Error("Неизвестная ошибка при получении пользователя");
  }
}
```

### Аннотации для коллбэков и функций высшего порядка

```typescript
// Аннотация для функции, принимающей коллбэк
function processArray<T, U>(
  array: T[],
  callback: (item: T, index: number) => U
): U[] {
  return array.map(callback);
}

// Использование
const numbers = [1, 2, 3, 4, 5];
const doubled = processArray<number, number>(numbers, (num) => num * 2);

// Аннотация для функции высшего порядка
type Predicate<T> = (value: T) => boolean;

function filterArray<T>(array: T[], predicate: Predicate<T>): T[] {
  return array.filter(predicate);
}

// Использование
const filteredNumbers = filterArray(numbers, (num) => num > 2);
```

### Аннотации для обработчиков событий

```typescript
// Аннотации для обработчиков событий
interface EventHandler<T = void> {
  (data?: T): void;
}

interface UserEvent {
  userId: number;
  action: string;
}

type UserEventHandler = EventHandler<UserEvent>;

const handleUserLogin: UserEventHandler = (event) => {
  if (event) {
    console.log(`Пользователь ${event.userId} выполнил действие: ${event.action}`);
  }
};

// Аннотации для DOM событий
function setupClickHandler(element: HTMLElement, handler: (event: MouseEvent) => void): void {
  element.addEventListener('click', handler);
}
```

## Связь с другими концепциями

- [[type-system/type-inference|Вывод типов]] - как TypeScript автоматически определяет типы без аннотаций
- [[types/type-aliases-comprehensive|Псевдонимы типов]] - для создания пользовательских типов
- [[ts/generics/index|Обобщения]] - для создания гибких аннотаций типов
- [[interfaces-classes/interfaces|Интерфейсы]] - для определения структур объектов

## Лучшие практики

1. **Аннотируйте возвращаемые значения функций** - особенно в публичных API
2. **Используйте интерфейсы или типы для сложных объектов** - вместо встроенных аннотаций
3. **Аннотируйте параметры функций** - для обеспечения безопасности типов
4. **Используйте обобщения для гибкости** - когда типы неизвестны заранее
5. **Не перегружайте код избыточными аннотациями** - используйте вывод типов, когда это возможно

## Заключение

Аннотации типов - фундаментальная особенность TypeScript, обеспечивающая безопасность типов и улучшающая качество кода. Правильное использование аннотаций типов делает код более понятным, предотвращает ошибки и улучшает опыт разработки за счет автодополнения и проверки типов.
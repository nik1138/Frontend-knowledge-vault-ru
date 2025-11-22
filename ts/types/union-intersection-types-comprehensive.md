# Объединения и пересечения типов в TypeScript - Полное руководство

## Введение

Объединения и пересечения типов - это мощные возможности системы типов TypeScript, которые позволяют создавать новые типы из существующих. Эти концепции позволяют моделировать сложные структуры данных и поведение, обеспечивая гибкость и безопасность типов. В этом руководстве мы подробно рассмотрим обе концепции и их практическое применение.

## Объединения типов (Union Types)

Объединения типов позволяют переменной иметь значение одного из нескольких типов. Используется оператор `|`.

### Основное использование объединений

```typescript
// Переменная может быть строкой или числом
let value: string | number;
value = "строка"; // OK
value = 42;       // OK
// value = true;  // ОШИБКА: boolean не входит в объединение

// Функция с параметром объединения
function formatId(id: string | number): string {
  return `ID: ${id}`;
}

console.log(formatId("abc")); // "ID: ID: abc"
console.log(formatId(123));   // "ID: ID: 123"
```

### Работа с объединениями типов

При работе с объединениями типов можно использовать только те свойства и методы, которые существуют во всех типах объединения:

```typescript
function printValue(value: string | number): void {
  // Следующие методы доступны для обоих типов:
  console.log(`Тип: ${typeof value}`);
  console.log(`Значение: ${value}`);
  
  // ОШИБКА: Метод toUpperCase() существует только у string
  // value.toUpperCase(); // ОШИБКА
  
  // Нужна проверка типа:
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // OK
  } else {
    console.log(value.toFixed(2)); // OK для чисел
  }
}
```

### Объединения с литеральными типами

Объединения часто используются с литеральными типами для создания ограниченных наборов значений:

```typescript
// Определение ограниченного набора значений
type Theme = "light" | "dark";
type HttpStatus = 200 | 201 | 400 | 401 | 404 | 500;

function applyTheme(theme: Theme): void {
  console.log(`Применение темы: ${theme}`);
}

applyTheme("light"); // OK
applyTheme("dark");  // OK
// applyTheme("blue"); // ОШИБКА: "blue" не является допустимым значением Theme

// Комбинирование с другими типами
type ApiResponse = {
  status: HttpStatus;
  message: string;
  data?: any;
};
```

## Пересечения типов (Intersection Types)

Пересечения типов позволяют объединить несколько типов в один, содержащий все свойства всех типов. Используется оператор `&`.

### Основное использование пересечений

```typescript
// Определение двух отдельных типов
type Name = {
  firstName: string;
  lastName: string;
};

type Age = {
  age: number;
};

type Contact = {
  email: string;
  phone: string;
};

// Создание пересечения типов
type Person = Name & Age & Contact;

const person: Person = {
  firstName: "Алексей",
  lastName: "Иванов",
  age: 30,
  email: "alex@example.com",
  phone: "+7 (495) 123-45-67"
};
```

### Практическое применение пересечений

```typescript
// Определение отдельных функциональностей
type Timestamps = {
  createdAt: Date;
  updatedAt: Date;
};

type SoftDelete = {
  deletedAt?: Date;
};

type Versioned = {
  version: number;
};

// Комбинирование функциональностей в один тип
type Entity = Timestamps & SoftDelete & Versioned & {
  id: number;
  name: string;
};

const user: Entity = {
  id: 1,
  name: "Пользователь",
  createdAt: new Date(),
  updatedAt: new Date(),
  version: 1
};
```

## Сложные примеры объединений и пересечений

### Объединения объектных типов

```typescript
// Определение различных типов пользователей
type Admin = {
  type: "admin";
  adminLevel: number;
  permissions: string[];
};

type User = {
  type: "user";
  email: string;
  lastLogin: Date;
};

type Guest = {
  type: "guest";
  sessionId: string;
};

// Объединение типов пользователей
type UserType = Admin | User | Guest;

function handleUser(user: UserType): void {
  // Использование дискриминированного объединения
  switch (user.type) {
    case "admin":
      console.log(`Админ уровень: ${user.adminLevel}`);
      break;
    case "user":
      console.log(`Почта: ${user.email}`);
      break;
    case "guest":
      console.log(`Сессия: ${user.sessionId}`);
      break;
  }
}
```

### Пересечение с объединением

```typescript
// Определение базовых типов
type HasId = { id: number };
type HasName = { name: string };

// Создание объединения
type Status = "active" | "inactive";

// Пересечение объединения с объектным типом
type EntityWithStatus = HasId & HasName & { status: Status };

const entity: EntityWithStatus = {
  id: 1,
  name: "Тестовая сущность",
  status: "active"
};
```

## Дискриминированные объединения

Дискриминированные объединения (также называемые объединениями с меткой) - это паттерн, который позволяет TypeScript понимать, какой конкретный тип представлен в объединении.

```typescript
// Объединение с дискриминирующим полем
type LoadingState = 
  | { status: "loading" }
  | { status: "error"; error: string }
  | { status: "success"; data: any };

function handleLoadingState(state: LoadingState): void {
  // TypeScript знает, какие поля доступны в зависимости от значения status
  switch (state.status) {
    case "loading":
      console.log("Загрузка...");
      // state.error и state.data недоступны здесь
      break;
    case "error":
      console.log(`Ошибка: ${state.error}`);
      // state.data недоступно здесь
      break;
    case "success":
      console.log(`Данные: ${state.data}`);
      // state.error недоступно здесь
      break;
  }
}
```

## Практические примеры

### Обработка API-ответов с объединениями

```typescript
// Типы для ответов API
type SuccessResponse<T> = {
  success: true;
  data: T;
};

type ErrorResponse = {
  success: false;
  error: string;
  errorCode: number;
};

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

function handleApiResponse<T>(response: ApiResponse<T>): void {
  if (response.success) {
    // TypeScript знает, что это SuccessResponse
    console.log("Данные:", response.data);
  } else {
    // TypeScript знает, что это ErrorResponse
    console.log(`Ошибка ${response.errorCode}: ${response.error}`);
  }
}
```

### Конфигурационные объекты с пересечениями

```typescript
// Определение различных частей конфигурации
type ServerConfig = {
  host: string;
  port: number;
};

type DatabaseConfig = {
  dbUrl: string;
  dbName: string;
};

type AuthConfig = {
  secret: string;
  expiresIn: string;
};

// Комбинирование конфигураций
type AppConfig = ServerConfig & DatabaseConfig & AuthConfig;

const config: AppConfig = {
  host: "localhost",
  port: 3000,
  dbUrl: "mongodb://localhost:27017",
  dbName: "myapp",
  secret: "my-secret-key",
  expiresIn: "1h"
};
```

## Продвинутые паттерны

### Условные типы с объединениями

```typescript
// Утилита для извлечения типа по ключу из объединения
type PropType<T, K extends string> = T extends { [k in K]: any } ? T[K] : never;

// Пример использования
type A = { type: 'A', value: string };
type B = { type: 'B', value: number };
type Union = A | B;

type Values = PropType<Union, 'value'>; // string | number
```

### Рекурсивные объединения

```typescript
// Тип для представления вложенной структуры
type TreeNode = {
  value: string;
  children?: TreeNode[];
};

// Рекурсивное объединение для плоской структуры
type FlatNode = { value: string; parentId?: number } | { value: string; parentId?: number }[];

const flatData: FlatNode = [
  { value: "корень", parentId: undefined },
  { value: "дочерний1", parentId: 1 },
  { value: "дочерний2", parentId: 1 }
];
```

## Связь с другими концепциями

- [[type-system/type-guards|Тип-гарды]] - как проверять типы в объединениях
- [[types/literal-types-comprehensive|Литеральные типы]] - часто используются в объединениях
- [[interfaces-classes/interfaces|Интерфейсы]] - могут быть объединены с помощью пересечений
- [[advanced-types/discriminated-unions-comprehensive|Дискриминированные объединения]] - продвинутый паттерн использования объединений
- [[Утилиты типов TypeScript|Утилиты типов]] - для работы с объединениями и пересечениями

## Лучшие практики

1. **Используйте дискриминированные объединения** для безопасной работы с объединениями объектных типов
2. **Избегайте слишком сложных объединений**, которые трудно понять и поддерживать
3. **Используйте литеральные типы в объединениях** для создания ограниченных наборов допустимых значений
4. **Придерживайтесь принципа наименьшего типа** - используйте наиболее конкретный тип, который удовлетворяет требованиям

## Заключение

Объединения и пересечения типов - мощные инструменты для создания гибких и выразительных типов в TypeScript. Они позволяют точно моделировать сложные структуры данных и поведение, обеспечивая при этом безопасность типов и автодополнение в редакторах кода. Правильное использование этих концепций делает код более надежным и понятным.
# Утилиты типов в TypeScript

## Введение

Утилиты типов - это встроенные в TypeScript типы, которые позволяют выполнять распространенные преобразования типов. Они особенно полезны для работы с обобщениями и позволяют создавать гибкие и переиспользуемые типы. Утилиты типов реализованы как обобщенные типы, которые принимают типы в качестве аргументов и возвращают преобразованные типы.

## Основные утилиты типов

### Partial<T>

Преобразует все свойства типа в необязательные:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Создание частичного типа
type PartialUser = Partial<User>;
// Результат: { id?: number; name?: string; email?: string; age?: number; }

// Практическое применение - обновление объекта
function updateUser(id: number, updates: Partial<User>): User {
  const user = getUserById(id);
  return { ...user, ...updates };
}

// Использование
const updatedUser = updateUser(1, { name: "Новое имя" }); // Только name обновляется
```

### Required<T>

Делает все свойства типа обязательными:

```typescript
interface OptionalUser {
  id?: number;
  name?: string;
  email?: string;
}

type RequiredUser = Required<OptionalUser>;
// Результат: { id: number; name: string; email: string; }

// Практическое применение
function validateUser(user: RequiredUser): boolean {
  return user.id !== undefined && 
         user.name !== undefined && 
         user.email !== undefined;
}
```

### Readonly<T>

Делает все свойства типа только для чтения:

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  retries: number;
}

type ReadonlyConfig = Readonly<Config>;
// Все свойства теперь только для чтения

const config: ReadonlyConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
};

// config.timeout = 10000; // ОШИБКА: Невозможно изменить свойство только для чтения
```

### Pick<T, K>

Создает тип, выбирая только указанные свойства K из типа T:

```typescript
interface Employee {
  id: number;
  name: string;
  email: string;
  department: string;
  salary: number;
  startDate: Date;
}

// Выбор только необходимых свойств
type EmployeePreview = Pick<Employee, 'id' | 'name' | 'email'>;
// Результат: { id: number; name: string; email: string; }

// Практическое применение - создание DTO
type CreateUserDto = Pick<Employee, 'name' | 'email' | 'department'>;

const newUser: CreateUserDto = {
  name: "Алексей",
  email: "alex@example.com",
  department: "Разработка"
};
```

### Omit<T, K>

Создает тип, исключая указанные свойства K из типа T:

```typescript
// Создание типа без чувствительных данных
type PublicEmployee = Omit<Employee, 'salary'>;
// Результат: { id: number; name: string; email: string; department: string; startDate: Date; }

// Использование в API ответах
type EmployeeResponse = Omit<Employee, 'salary' | 'startDate'>;
```

### Record<K, T>

Определяет тип объекта, где ключи принадлежат типу K, а значения - типу T:

```typescript
// Создание словаря
type PageMap = Record<string, string>;
const pages: PageMap = {
  home: "/",
  about: "/about",
  contact: "/contact"
};

// С использованием объединения для ключей
type Status = 'active' | 'inactive' | 'pending';
type UserStatusMap = Record<Status, User[]>;

const userStatusMap: UserStatusMap = {
  active: [],
  inactive: [],
  pending: []
};
```

## Продвинутые утилиты типов

### Exclude<T, U>

Исключает из T все типы, которые присутствуют в U:

```typescript
type AvailableColors = 'red' | 'green' | 'blue' | 'yellow';
type BasicColors = 'red' | 'green';

type AdvancedColors = Exclude<AvailableColors, BasicColors>;
// Результат: 'blue' | 'yellow'

// Практическое применение
type AllPermissions = 'read' | 'write' | 'delete' | 'admin';
type UserPermissions = 'read';
type AdminPermissions = Exclude<AllPermissions, UserPermissions>;
// 'write' | 'delete' | 'admin'
```

### Extract<T, U>

Извлекает из T все типы, которые также присутствуют в U:

```typescript
type Mixed = string | number | boolean | Date;
type Primitives = string | number;

type Extracted = Extract<Mixed, Primitives>;
// Результат: string | number
```

### NonNullable<T>

Исключает типы null и undefined из T:

```typescript
type MaybeString = string | null | undefined;
type DefinitelyString = NonNullable<MaybeString>;
// Результат: string
```

### Parameters<T>

Извлекает типы параметров функции:

```typescript
function createUser(name: string, age: number, email: string): User {
  return { id: 1, name, email, age: age };
}

type CreateUserParams = Parameters<typeof createUser>;
// Результат: [name: string, age: number, email: string]

// Практическое применение
function callWithDefaults(fn: (...args: CreateUserParams) => User) {
  return fn("Аноним", 0, "unknown@example.com");
}
```

### ConstructorParameters<T>

Извлекает типы параметров конструктора:

```typescript
class DatabaseConnection {
  constructor(host: string, port: number, username: string, password: string) {
    // ...
  }
}

type ConnectionParams = ConstructorParameters<typeof DatabaseConnection>;
// Результат: [host: string, port: number, username: string, password: string]
```

### ReturnType<T>

Извлекает возвращаемый тип функции:

```typescript
function getUserInfo(id: number) {
  return {
    id,
    name: "Имя пользователя",
    email: "user@example.com",
    isActive: true
  };
}

type UserInfo = ReturnType<typeof getUserInfo>;
// Результат: { id: number; name: string; email: string; isActive: boolean; }
```

### InstanceType<T>

Извлекает тип экземпляра конструктора:

```typescript
class ApiResponse<T> {
  constructor(public data: T, public timestamp: Date = new Date()) {}
}

type ResponseInstance = InstanceType<typeof ApiResponse<string>>;
// Результат: ApiResponse<string>
```

## Практические примеры

### Создание кастомных утилит

```typescript
// Утилита для создания типа с обязательными полями, кроме указанных
type PartialExcept<T, K extends keyof T> = Pick<T, K> & Partial<Omit<T, K>>;

interface UserProfile {
  id: number;
  name: string;
  email: string;
  avatar: string;
  bio: string;
}

// Требуется только id и name, остальные поля опциональны
type RequiredProfile = PartialExcept<UserProfile, 'id' | 'name'>;
// Результат: { id: number; name: string; } & { email?: string; avatar?: string; bio?: string; }
```

### Утилита для создания "грязных" полей

```typescript
// Создание типа для отслеживания измененных полей
type DirtyFields<T> = {
  [K in keyof T]?: boolean;
};

type UserDirtyFields = DirtyFields<User>;
// Результат: { id?: boolean; name?: boolean; email?: boolean; age?: boolean; }
```

### Утилита для глубокого изменения типа

```typescript
// Глубокое изменение типа на только для чтения
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

interface Address {
  street: string;
  city: string;
}

interface Person {
  name: string;
  address: Address;
}

type ReadonlyPerson = DeepReadonly<Person>;
// Все свойства, включая вложенные, будут только для чтения
```

## Связь с другими концепциями

- [[Продвинутые дженерики в TypeScript|Обобщения]] - утилиты типов являются обобщенными типами
- [[advanced-types/conditional-types|Условные типы]] - основа для реализации многих утилит
- [[advanced-types/mapped-types|Сопоставляющие типы]] - используются для реализации утилит
- [[types/type-aliases-comprehensive|Псевдонимы типов]] - для создания пользовательских утилит

## Лучшие практики

1. **Используйте встроенные утилиты типов** - они оптимизированы и проверены
2. **Создавайте кастомные утилиты для повторяющихся паттернов** - улучшает читаемость кода
3. **Документируйте сложные утилиты типов** - особенно если они используют условные или сопоставляющие типы
4. **Тестируйте утилиты типов** - убедитесь, что они возвращают ожидаемые типы

## Заключение

Утилиты типов - мощный инструмент для преобразования и манипуляции типами в TypeScript. Они позволяют создавать гибкие, переиспользуемые и безопасные типы, значительно упрощая работу с обобщениями и сложными структурами данных.
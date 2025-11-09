# Встроенные утилиты для выбора свойств

TypeScript предоставляет утилиты, которые позволяют выбирать или исключать определенные свойства из типов объектов.

## Pick<T, K>

Выбирает определенные свойства `K` из типа `T`.

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

interface User {
  name: string;
  age: number;
  email: string;
  password: string;
}

type UserProfile = Pick<User, "name" | "email">;
// {
//   name: string;
//   email: string;
// }

// Практическое применение
function getUserProfile(user: User): UserProfile {
  return {
    name: user.name,
    email: user.email
  };
}

// Для API ответов
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
  timestamp: Date;
  internalDebugInfo: string; // Не должно быть в публичном API
}

type PublicApiResponse<T> = Pick<ApiResponse<T>, 
  "data" | "status" | "message" | "timestamp"
>;
```

## Omit<T, K>

Исключает определенные свойства `K` из типа `T`.

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

interface User {
  name: string;
  age: number;
  email: string;
  password: string;
}

type PublicUser = Omit<User, "password">;
// {
//   name: string;
//   age: number;
//   email: string;
// }

// Практическое применение
function sendUserToClient(user: User): PublicUser {
  const { password, ...publicUser } = user;
  return publicUser;
}

// Для баз данных
interface DatabaseUser {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
  lastLoginAt: Date;
}

type UserForUpdate = Omit<DatabaseUser, 
  "id" | "createdAt" | "lastLoginAt"
>;
// Исключаем поля, которые не должны обновляться клиентом
```

## Extract<T, U>

Извлекает из типа `T` те типы, которые могут быть присвоены типу `U`.

```typescript
type Extract<T, U> = T extends U ? T : never;

type StringOrNumber = string | number | boolean;
type StringsAndNumbers = Extract<StringOrNumber, string | number>;
// string | number

// Практическое применение
type EventTypes = "click" | "focus" | "blur" | "hover" | "submit";
type MouseEvents = Extract<EventTypes, "click" | "hover">;
// "click" | "hover"

// С объединениями интерфейсов
interface AdminUser {
  role: "admin";
  permissions: string[];
}

interface RegularUser {
  role: "user";
  permissions: string[];
}

interface GuestUser {
  role: "guest";
  permissions: [];
}

type User = AdminUser | RegularUser | GuestUser;
type PrivilegedUser = Extract<User, { role: "admin" | "user" }>;
// AdminUser | RegularUser
```

## Exclude<T, U>

Исключает из типа `T` те типы, которые могут быть присвоены типу `U`.

```typescript
type Exclude<T, U> = T extends U ? never : T;

type StringOrNumber = string | number | boolean;
type NonString = Exclude<StringOrNumber, string>;
// number | boolean

// Практическое применение
type Status = "pending" | "loading" | "success" | "error";
type FinalStatus = Exclude<Status, "pending" | "loading">;
// "success" | "error"

// С типами свойств
interface User {
  name: string;
  age: number;
  email: string;
  password: string;
}

type NonSensitiveKeys = Exclude<keyof User, "password">;
// "name" | "age" | "email"

type NonSensitiveUser = Pick<User, NonSensitiveKeys>;
// {
//   name: string;
//   age: number;
//   email: string;
// }
```

## PickByType<T, U>

Пользовательская утилита для выбора свойств определенного типа.

```typescript
type PickByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface User {
  name: string;
  age: number;
  isActive: boolean;
  lastLogin: Date;
  score: number;
}

type StringProperties = PickByType<User, string>;
// {
//   name: string;
// }

type NumberProperties = PickByType<User, number>;
// {
//   age: number;
//   score: number;
// }

type BooleanProperties = PickByType<User, boolean>;
// {
//   isActive: boolean;
// }
```

## OmitByType<T, U>

Пользовательская утилита для исключения свойств определенного типа.

```typescript
type OmitByType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K];
};

interface User {
  name: string;
  age: number;
  isActive: boolean;
  lastLogin: Date;
  score: number;
}

type NonStringProperties = OmitByType<User, string>;
// {
//   age: number;
//   isActive: boolean;
//   lastLogin: Date;
//   score: number;
// }

type NonFunctionProperties = OmitByType<User, Function>;
// Исключает все функции (если бы они были)
```

## RequiredPick<T, K>

Пользовательская утилита, которая делает выбранные свойства обязательными.

```typescript
type RequiredPick<T, K extends keyof T> = T & {
  [P in K]-?: T[P];
};

interface User {
  name?: string;
  age?: number;
  email?: string;
  password?: string;
}

type UserWithRequiredEmail = RequiredPick<User, "email">;
// {
//   name?: string;
//   age?: number;
//   email: string; // Стало обязательным
//   password?: string;
// }

// Практическое применение
function sendEmail(user: RequiredPick<User, "email">) {
  // TypeScript гарантирует, что email существует
  console.log(`Sending email to ${user.email}`);
}
```

## PartialPick<T, K>

Пользовательская утилита, которая делает выбранные свойства опциональными.

```typescript
type PartialPick<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface User {
  name: string;
  age: number;
  email: string;
  password: string;
}

type UserWithOptionalEmail = PartialPick<User, "email" | "password">;
// {
//   name: string;
//   age: number;
//   email?: string; // Стало опциональным
//   password?: string; // Стало опциональным
// }

// Практическое применение
function updateUser(
  id: string, 
  updates: PartialPick<User, "email" | "password">
) {
  // Можно обновить email и password, но не обязательно
}
```

## Ограничения и особенности

1. `Pick` и `Omit` работают только с ключами существующих типов
2. `Extract` и `Exclude` работают с объединениями типов
3. Пользовательские утилиты могут быть сложны для понимания
4. Некоторые утилиты могут не работать корректно с индексными сигнатурами

## Связанные концепции

- [[ts/utility-types/built-in-object-transformations|Утилиты для преобразования объектов]]
- [[ts/advanced-types/mapped-types|Сопоставленные типы]]
- [[ts/advanced-types/conditional-types|Условные типы]]
- [[ts/type-system/keyof-operator|Оператор keyof]]
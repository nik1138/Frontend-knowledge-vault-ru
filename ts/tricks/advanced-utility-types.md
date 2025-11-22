---
aliases: ["Продвинутые утилитарные типы TypeScript", "TS Advanced Utility Types"]
tags: [typescript, utility-types, advanced]
---

# Продвинутые утилитарные типы TypeScript

## Введение

Продвинутые утилитарные типы в TypeScript позволяют создавать сложные типы с помощью встроенных утилитарных типов и их комбинаций. Эти типы особенно полезны при создании типобезопасных библиотек и при работе с API.

## Основные продвинутые утилитарные типы

### `Pick` и `Omit` в комбинации

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

// Выбираем только нужные поля
type PublicUser = Pick<User, 'id' | 'name' | 'email'>;

// Исключаем приватные поля
type SafeUser = Omit<User, 'password'>;

const publicUser: PublicUser = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com'
};

const safeUser: SafeUser = {
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  createdAt: new Date(),
  updatedAt: new Date()
};
```

### `Extract` и `Exclude`

```typescript
type Status = 'pending' | 'active' | 'inactive' | 'deleted';

// Извлекаем только активные статусы
type ActiveStatus = Extract<Status, 'active' | 'pending'>;

// Исключаем неактивные статусы
type ValidStatus = Exclude<Status, 'inactive' | 'deleted'>;

const activeStatus: ActiveStatus = 'active'; // Только 'active' или 'pending'
const validStatus: ValidStatus = 'active'; // Только 'active' или 'pending'
```

### `NonNullable`

```typescript
type MaybeUser = User | null | undefined;

// Убираем null и undefined
type DefiniteUser = NonNullable<MaybeUser>;

const user: DefiniteUser = {} as User; // Теперь точно User, без null/undefined
```

### `Parameters` и `ConstructorParameters`

```typescript
function createUser(name: string, age: number, email?: string): User {
  return {
    id: 1,
    name,
    email: email || '',
    password: '',
    createdAt: new Date(),
    updatedAt: new Date()
  };
}

class UserService {
  constructor(private apiUrl: string, private timeout: number) {}
  
  getUser(id: number): Promise<User> {
    return fetch(`${this.apiUrl}/users/${id}`).then(res => res.json());
  }
}

// Получаем типы параметров функции
type CreateUserParams = Parameters<typeof createUser>;
// [string, number, (string | undefined)?]

// Получаем типы параметров конструктора
type UserServiceParams = ConstructorParameters<typeof UserService>;
// [string, number]
```

### `ReturnType`

```typescript
// Получаем тип возвращаемого значения
type CreateUserReturn = ReturnType<typeof createUser>;
// User

// Использование с асинхронными функциями
type GetUserReturn = ReturnType<UserService['getUser']>;
// Promise<User>
```

## Практические примеры

### Создание типа для обновления сущности

```typescript
// Создаем тип, где все поля опциональны
type PartialUpdate<T> = {
  [P in keyof T]?: T[P];
};

type UserUpdate = PartialUpdate<Omit<User, 'id'>>;

const updateUserPayload: UserUpdate = {
  name: 'Jane Doe',
  // email может быть не указан
  // остальные поля не обязательны
};
```

### Создание типа для создания сущности

```typescript
// Убираем поля, которые устанавливаются автоматически
type UserCreation = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;

const newUser: UserCreation = {
  name: 'Jane Doe',
  email: 'jane@example.com',
  password: 'securePassword'
};
```

### Утилита для создания "плоских" типов

```typescript
// Преобразование типа с опциональными полями в тип, где поля обязательны
type RequiredKeys<T> = {
  [K in keyof T]-?: T[K];
};

type UserRequired = RequiredKeys<User>;
```

### Утилита для извлечения типов из массива

```typescript
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
] as const;

type UserFromArray = typeof users[number];
// { readonly id: number; readonly name: string; }

type UserId = UserFromArray['id'];
// number
```

## Сложные примеры

### Утилита для создания типа с обязательными полями

```typescript
// Создаем тип, где указанные поля обязательны, остальные опциональны
type RequiredOnly<T, K extends keyof T> = Required<Pick<T, K>> & Partial<Omit<T, K>>;

interface FormValues {
  name: string;
  email: string;
  phone?: string;
  address?: string;
}

// Создаем тип, где только name и email обязательны
type RequiredForm = RequiredOnly<FormValues, 'name' | 'email'>;

const formData: RequiredForm = {
  name: 'John',
  email: 'john@example.com'
  // phone и address опциональны
};
```

### Утилита для создания типа с исключенными полями

```typescript
// Утилита для создания типа с исключенными полями и оставшимися как опциональные
type OptionalExcept<T, K extends keyof T> = Pick<T, K> & Partial<Omit<T, K>>;

type UserWithRequiredId = OptionalExcept<User, 'id'>;

const userWithId: UserWithRequiredId = {
  id: 1
  // остальные поля опциональны
};
```

## Заключение

Продвинутые утилитарные типы позволяют создавать сложные типы с минимальным дублированием кода. Они особенно полезны при создании типобезопасных API и библиотек. Используйте их для повышения надежности кода и улучшения автодополнения в IDE.

См. также: [[conditional-types-tricks]] | [[mapped-types-tricks]] | [[type-inference-tricks]]
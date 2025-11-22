---
aliases: ["Хитрости с отображенными типами TypeScript", "TS Mapped Types Tricks"]
tags: [typescript, mapped-types, advanced]
---

# Хитрости с отображенными типами TypeScript

## Введение

Отображенные типы (Mapped Types) позволяют создавать новые типы, трансформируя существующие типы. Они особенно полезны для создания вариаций существующих типов с измененными модификаторами доступа или свойствами.

## Основы отображенных типов

### Простой пример отображенного типа

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Создаем тип, где все поля опциональны
type OptionalUser = {
  [K in keyof User]?: User[K];
};

const optionalUser: OptionalUser = {
  // Все поля опциональны
  name: 'John'
};
```

### Использование модификаторов `+` и `-`

```typescript
// Делаем все поля обязательными (по умолчанию, но явно указано)
type RequiredUser = {
  [K in keyof User]-?: User[K];
};

// Делаем все поля опциональными
type AllOptionalUser = {
  [K in keyof User]+?: User[K];
};
```

## Продвинутые примеры

### Создание типа с определенными полями как readonly

```typescript
// Делаем только указанные поля readonly
type ReadonlyFields<T, K extends keyof T> = {
  [P in K]: Readonly<T[P]>;
} & {
  [P in Exclude<keyof T, K>]: T[P];
};

type UserWithReadonlyId = ReadonlyFields<User, 'id'>;

const userWithReadonlyId: UserWithReadonlyId = {
  id: 1, // readonly
  name: 'John',
  email: 'john@example.com',
  password: 'secret'
};

// userWithReadonlyId.id = 2; // Ошибка: нельзя изменить readonly поле
```

### Отображенные типы с преобразованием ключей

```typescript
// Преобразование ключей в строку с префиксом
type Prefixed<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Prefixed<User>;

// Результат:
// {
//   getId: () => number;
//   getName: () => string;
//   getEmail: () => string;
//   getPassword: () => string;
// }

const userGetters: UserGetters = {
  getId: () => 1,
  getName: () => 'John',
  getEmail: () => 'john@example.com',
  getPassword: () => 'secret'
};
```

### Фильтрация полей по типу

```typescript
// Извлекаем только поля определенного типа
type FilterByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

type StringFields = FilterByType<User, string>;
// { name: string; email: string; password: string; }

type NumberFields = FilterByType<User, number>;
// { id: number; }
```

### Создание типа только с функциями

```typescript
// Извлекаем только функции из объекта
type FunctionsOnly<T> = {
  [K in keyof T as T[K] extends Function ? K : never]: T[K];
};

interface UserService {
  getUser(id: number): Promise<User>;
  updateUser(id: number, data: Partial<User>): Promise<User>;
  name: string;
  timeout: number;
}

type UserMethods = FunctionsOnly<UserService>;
// { getUser: (id: number) => Promise<User>; updateUser: (id: number, data: Partial<User>) => Promise<User> }
```

## Практические примеры

### Создание типа для обновления сущности

```typescript
// Создаем тип для обновления, где все поля опциональны, кроме указанных
type UpdateType<T, R extends keyof T = never> = {
  [K in keyof T]: K extends R ? T[K] : T[K] | undefined;
};

type UserUpdate = UpdateType<User, 'id'>;

const userUpdate: UserUpdate = {
  id: 1, // обязательное поле
  name: 'John', // опционально
  email: undefined // может быть undefined
};
```

### Создание типа с преобразованными полями

```typescript
// Создаем тип, где все строковые поля становятся строками или null
type StringToNullable<T> = {
  [K in keyof T]: T[K] extends string ? string | null : T[K];
};

type NullableUser = StringToNullable<User>;

const nullableUser: NullableUser = {
  id: 1,
  name: 'John',
  email: null, // теперь может быть null
  password: 'secret'
};
```

### Создание типа с переименованными полями

```typescript
// Переименование полей с использованием маппинга
type RenameFields<T, R extends Record<string, string>> = {
  [K in keyof T as K extends keyof R ? R[K & keyof R] : K]: T[K];
};

interface OldUser {
  userId: number;
  userName: string;
  userEmail: string;
}

type NewUser = RenameFields<OldUser, { userId: 'id'; userName: 'name'; userEmail: 'email' }>;
// { id: number; name: string; email: string; }
```

### Условное добавление полей

```typescript
// Условное добавление полей в зависимости от типа
type WithTimestamp<T> = T & { createdAt: Date; updatedAt: Date };

type UserWithTimestamp = WithTimestamp<Omit<User, 'createdAt' | 'updatedAt'>>;

const userWithTimestamp: UserWithTimestamp = {
  id: 1,
  name: 'John',
  email: 'john@example.com',
  password: 'secret',
  createdAt: new Date(),
  updatedAt: new Date()
};
```

## Сложные примеры

### Отображенные типы с рекурсией

```typescript
// Глубокое преобразование в readonly
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : DeepReadonly<T[K]> 
    : T[K]
};

interface ComplexUser {
  id: number;
  profile: {
    name: string;
    settings: {
      theme: string;
      notifications: boolean;
    };
  };
  friends: User[];
}

type ReadonlyComplexUser = DeepReadonly<ComplexUser>;
```

### Создание типа с исключением полей

```typescript
// Исключение полей с определенным типом
type ExcludeFieldsOfType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K];
};

type UserWithoutStrings = ExcludeFieldsOfType<User, string>;
// { id: number; }
```

### Создание типа с полями, удовлетворяющими условию

```typescript
// Создаем тип с полями, которые не являются функциями и не являются undefined
type NonFunctionNonUndefined<T> = {
  [K in keyof T as T[K] extends Function | undefined ? never : K]: T[K];
};

interface MixedObject {
  id: number;
  name: string;
  method: () => void;
  optional?: string;
  callback?: () => void;
}

type CleanObject = NonFunctionNonUndefined<MixedObject>;
// { id: number; name: string; }
```

### Создание типа с полями, содержащими определенные значения

```typescript
// Создаем тип с полями, которые могут содержать только определенные значения
type RestrictedValues<T, V> = {
  [K in keyof T]: T[K] extends V ? T[K] : never;
};

type UserWithStringValues = RestrictedValues<User, string>;
// { name: string; email: string; password: string; }
```

## Заключение

Отображенные типы - мощный инструмент для создания гибких и адаптивных типов в TypeScript. Они позволяют трансформировать существующие типы различными способами, что особенно полезно при создании типобезопасных API и библиотек.

См. также: [[advanced-utility-types]] | [[conditional-types-tricks]] | [[type-inference-tricks]]
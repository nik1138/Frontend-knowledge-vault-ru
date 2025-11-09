# Встроенные утилиты для преобразования объектов

TypeScript предоставляет несколько встроенных утилит, которые позволяют преобразовывать типы объектов, изменяя их свойства различными способами.

## Partial<T>

Делает все свойства типа `T` опциональными.

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P];
};

interface User {
  name: string;
  age: number;
  email: string;
}

type PartialUser = Partial<User>;
// {
//   name?: string;
//   age?: number;
//   email?: string;
// }

// Практическое применение
function updateUser(id: string, updates: Partial<User>) {
  // Можно передать только те поля, которые нужно обновить
}

updateUser("123", { name: "John" }); // ✅
updateUser("123", { age: 30, email: "john@example.com" }); // ✅
```

## Required<T>

Делает все свойства типа `T` обязательными.

```typescript
type Required<T> = {
  [P in keyof T]-?: T[P];
};

interface PartialUser {
  name?: string;
  age?: number;
  email?: string;
}

type CompleteUser = Required<PartialUser>;
// {
//   name: string;
//   age: number;
//   email: string;
// }

// Практическое применение
function createUser(user: Required<PartialUser>) {
  // Все поля обязательны
}
```

## Readonly<T>

Делает все свойства типа `T` только для чтения.

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

interface User {
  name: string;
  age: number;
}

type ReadonlyUser = Readonly<User>;
// {
//   readonly name: string;
//   readonly age: number;
// }

// Практическое применение
const user: Readonly<User> = {
  name: "John",
  age: 30
};

// user.name = "Jane"; // ❌ Ошибка: нельзя изменить readonly свойство
```

## Mutable<T> (пользовательская утилита)

Обратная операция к `Readonly<T>` - делает все свойства изменяемыми.

```typescript
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

interface ReadonlyUser {
  readonly name: string;
  readonly age: number;
}

type MutableUser = Mutable<ReadonlyUser>;
// {
//   name: string;
//   age: number;
// }

// Практическое применение
const user: ReadonlyUser = {
  name: "John",
  age: 30
};

// Создаем изменяемую копию
const mutableUser: MutableUser = { ...user };
mutableUser.name = "Jane"; // ✅ Можно изменить
```

## DeepReadonly<T>

Рекурсивная версия `Readonly<T>`, которая делает readonly все вложенные свойства.

```typescript
type DeepReadonly<T> = T extends object 
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> } 
  : T;

interface User {
  name: string;
  address: {
    street: string;
    city: string;
  };
}

type DeepReadonlyUser = DeepReadonly<User>;
// {
//   readonly name: string;
//   readonly address: {
//     readonly street: string;
//     readonly city: string;
//   };
// }

// Практическое применение
const user: DeepReadonly<User> = {
  name: "John",
  address: {
    street: "123 Main St",
    city: "New York"
  }
};

// user.address.street = "456 Oak Ave"; // ❌ Ошибка: readonly свойство
```

## DeepPartial<T>

Рекурсивная версия `Partial<T>`, которая делает опциональными все вложенные свойства.

```typescript
type DeepPartial<T> = T extends object 
  ? { [P in keyof T]?: DeepPartial<T[P]> } 
  : T;

interface User {
  name: string;
  address: {
    street: string;
    city: string;
  };
}

type DeepPartialUser = DeepPartial<User>;
// {
//   name?: string;
//   address?: {
//     street?: string;
//     city?: string;
//   };
// }

// Практическое применение
function updateDeepUser(id: string, updates: DeepPartial<User>) {
  // Можно обновить любые вложенные свойства
}

updateDeepUser("123", {
  address: {
    city: "Boston"
  }
}); // ✅ Можно обновить только город
```

## RequiredWithDefaults<T, Defaults>

Комбинированная утилита, которая делает определенные свойства обязательными и устанавливает для них значения по умолчанию.

```typescript
type RequiredWithDefaults<T, Defaults extends Partial<T>> = 
  Required<Pick<T, keyof Defaults>> & Omit<T, keyof Defaults>;

interface User {
  name?: string;
  age?: number;
  email?: string;
  isActive?: boolean;
}

type UserWithDefaults = RequiredWithDefaults<User, {
  isActive: true;
}>;

// {
//   isActive: boolean; // Стало обязательным
//   name?: string;
//   age?: number;
//   email?: string;
// }

// Практическое применение
function createUserWithDefaults(
  user: RequiredWithDefaults<User, { isActive: true }>
): User {
  return {
    ...user,
    isActive: user.isActive ?? true
  };
}
```

## Ограничения и особенности

1. Встроенные утилиты работают только с типами объектов
2. Некоторые утилиты могут быть сложны для понимания при глубокой вложенности
3. При использовании с большими объектами могут замедлять компиляцию
4. Не все утилиты поддерживают корректную работу с индексными сигнатурами

## Связанные концепции

- [[ts/advanced-types/mapped-types|Сопоставленные типы]]
- [[ts/utility-types/built-in-property-selection|Утилиты для выбора свойств]]
- [[ts/utility-types/built-in-function-utilities|Утилиты для работы с функциями]]
- [[ts/advanced-types/conditional-types|Условные типы]]
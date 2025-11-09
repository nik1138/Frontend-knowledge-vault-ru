# Сопоставленные типы (Mapped Types)

Сопоставленные типы в TypeScript позволяют создавать новые типы на основе существующих, преобразуя их свойства. Это мощный механизм для автоматического создания типов с измененными характеристиками.

## Основная концепция

Сопоставленные типы используют синтаксис индексных типов и ключевое слово `in` для итерации по ключам существующего типа:

```typescript
type MappedType<T> = {
  [P in keyof T]: T[P];
};
```

## Встроенные сопоставленные типы

TypeScript предоставляет несколько встроенных сопоставленных типов:

### 1. Partial<T>

Делает все свойства типа опциональными:

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
```

### 2. Required<T>

Делает все свойства типа обязательными:

```typescript
type Required<T> = {
  [P in keyof T]-?: T[P];
};

interface PartialUser {
  name?: string;
  age?: number;
}

type CompleteUser = Required<PartialUser>;
// {
//   name: string;
//   age: number;
// }
```

### 3. Readonly<T>

Делает все свойства типа только для чтения:

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
```

### 4. Pick<T, K>

Выбирает определенные свойства из типа:

```typescript
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

interface User {
  name: string;
  age: number;
  email: string;
}

type UserName = Pick<User, 'name'>;
// {
//   name: string;
// }
```

### 5. Omit<T, K>

Исключает определенные свойства из типа:

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

interface User {
  name: string;
  age: number;
  email: string;
}

type UserWithoutEmail = Omit<User, 'email'>;
// {
//   name: string;
//   age: number;
// }
```

## Создание собственных сопоставленных типов

### 1. Изменение модификаторов

Можно добавлять или удалять модификаторы `readonly` и `?`:

```typescript
// Удаление readonly
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

// Добавление обязательности
type Concrete<T> = {
  [P in keyof T]-?: T[P];
};
```

### 2. Преобразование типов свойств

Можно изменять типы свойств:

```typescript
// Сделать все свойства строками
type Stringify<T> = {
  [P in keyof T]: string;
};

// Сделать все свойства функциями
type ToFunctions<T> = {
  [P in keyof T]: () => T[P];
};
```

### 3. Добавление новых свойств

Можно добавлять новые свойства к существующим типам:

```typescript
type WithId<T> = T & { id: string };

type WithTimestamp<T> = T & { 
  createdAt: Date; 
  updatedAt: Date; 
};
```

## Продвинутые примеры

### 1. Создание типа для обновления

```typescript
type UpdateType<T> = {
  [P in keyof T]?: T[P] | null;
};

interface User {
  name: string;
  age: number;
  email: string;
}

type UserUpdate = UpdateType<User>;
// {
//   name?: string | null;
//   age?: number | null;
//   email?: string | null;
// }
```

### 2. Преобразование в Promise

```typescript
type ToPromise<T> = {
  [P in keyof T]: Promise<T[P]>;
};

interface ApiData {
  users: User[];
  posts: Post[];
}

type AsyncApiData = ToPromise<ApiData>;
// {
//   users: Promise<User[]>;
//   posts: Promise<Post[]>;
// }
```

### 3. Создание типа для валидации

```typescript
type ValidationRules<T> = {
  [P in keyof T]: {
    required?: boolean;
    minLength?: number;
    maxLength?: number;
    pattern?: RegExp;
  };
};

interface User {
  name: string;
  age: number;
  email: string;
}

type UserValidation = ValidationRules<User>;
```

## Условные сопоставленные типы

Можно комбинировать сопоставленные типы с условными типами:

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
// Все вложенные свойства становятся readonly
```

## Практическое применение

### 1. Конфигурационные объекты

```typescript
type ConfigOptions<T> = {
  [P in keyof T]: {
    value: T[P];
    description: string;
    required: boolean;
  };
};
```

### 2. API ответы

```typescript
type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
  timestamp: Date;
};
```

## Ограничения и особенности

1. Сопоставленные типы работают только с типами объектов
2. Не могут добавлять новые ключи, которых нет в исходном типе
3. Могут быть сложными для понимания при глубокой вложенности
4. Требуют хорошего понимания keyof и индексных типов

## Связанные концепции

- [[ts/type-system/keyof-operator|Оператор keyof]]
- [[ts/advanced-types/conditional-types|Условные типы]]
- [[ts/type-system/indexed-access-types|Индексные типы]]
- [[ts/generics/TS Generics|Дженерики]]
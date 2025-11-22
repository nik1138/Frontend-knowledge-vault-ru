---
aliases: ["Хитрости с условными типами TypeScript", "TS Conditional Types Tricks"]
tags: [typescript, conditional-types, advanced]
---

# Хитрости с условными типами TypeScript

## Введение

Условные типы в TypeScript позволяют создавать типы, которые выбирают один из двух возможных типов в зависимости от условия. Это мощный инструмент для создания гибких и адаптивных типов.

## Основы условных типов

### Синтаксис условных типов

```typescript
// T extends U ? X : Y
// Если T можно присвоить U, то результат X, иначе Y

type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>; // true
type Test2 = IsString<number>; // false
```

### Условные типы с дженериками

```typescript
// Создаем тип, который возвращает строку, если T - строка, иначе возвращает T
type Stringify<T> = T extends string ? string : T;

type Test3 = Stringify<string>; // string
type Test4 = Stringify<number>; // number
type Test5 = Stringify<boolean>; // boolean
```

## Расширенные примеры

### Условные типы с массивами

```typescript
// Создаем тип, который извлекает тип элемента из массива
type ElementOf<T> = T extends (infer U)[] ? U : T;

type ArrayElement = ElementOf<string[]>; // string
type NonArray = ElementOf<string>; // string
```

### Условные типы с функциями

```typescript
// Извлекаем возвращаемый тип функции
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser(): User {
  return { id: 1, name: 'John', email: 'john@example.com', password: 'secret', createdAt: new Date(), updatedAt: new Date() };
}

type UserReturnType = GetReturnType<typeof getUser>; // User
```

### Условные типы с объектами

```typescript
// Извлекаем тип поля объекта
type FieldValue<T, K extends keyof T> = T[K] extends infer U ? U : never;

interface Product {
  id: number;
  name: string;
  price: number;
  isActive: boolean;
}

type ProductName = FieldValue<Product, 'name'>; // string
type ProductPrice = FieldValue<Product, 'price'>; // number
```

## Практические применения

### Фильтрация типов в объединениях

```typescript
// Удаляем null и undefined из объединения типов
type FilterOut<T, U> = T extends U ? never : T;

type Filtered = FilterOut<string | number | null | undefined, null | undefined>;
// string | number
```

### Создание типа для обработки асинхронных значений

```typescript
// Извлекаем тип из Promise
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type AsyncUser = UnwrapPromise<Promise<User>>;
// User

// Извлекаем тип из вложенных Promise
type UnwrapAll<T> = T extends Promise<infer U> 
  ? U extends Promise<any> 
    ? UnwrapAll<U> 
    : U 
  : T;

type DeepPromise = UnwrapAll<Promise<Promise<User>>>;
// User
```

### Условные типы для проверки свойств

```typescript
// Проверяем, есть ли у объекта определенное свойство
type HasProperty<T, K extends string> = K extends keyof T ? true : false;

type UserHasName = HasProperty<User, 'name'>; // true
type UserHasAge = HasProperty<User, 'age'>; // false
```

### Создание типа для извлечения функций из объекта

```typescript
// Извлекаем только функции из объекта
type FunctionProperties<T> = {
  [K in keyof T as T[K] extends Function ? K : never]: T[K]
};

interface UserService {
  getUser(id: number): Promise<User>;
  updateUser(id: number, data: Partial<User>): Promise<User>;
  name: string;
  timeout: number;
}

type UserMethods = FunctionProperties<UserService>;
// { getUser: (id: number) => Promise<User>; updateUser: (id: number, data: Partial<User>) => Promise<User> }
```

## Продвинутые паттерны

### Условные типы с рекурсией

```typescript
// Глубокое преобразование объекта в неизменяемый
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : DeepReadonly<T[K]> 
    : T[K]
};

interface DeepObject {
  user: {
    profile: {
      name: string;
      settings: {
        theme: string;
      };
    };
  };
}

type ReadonlyDeepObject = DeepReadonly<DeepObject>;
// Все вложенные объекты становятся readonly
```

### Условные типы для валидации форм

```typescript
// Создаем тип, который проверяет, все ли обязательные поля заполнены
type RequiredFields<T> = {
  [K in keyof T]-?: T[K] extends undefined ? never : T[K];
};

interface Form {
  name: string;
  email: string;
  phone?: string;
  address?: string;
}

// Проверяем, что все обязательные поля (не опциональные) заполнены
type ValidatedForm<T> = {
  [K in keyof T]: T[K] extends undefined ? never : T[K];
} & {
  [K in keyof T as undefined extends T[K] ? never : K]: T[K];
};

// Более простой вариант - проверка на отсутствие undefined
type NonOptional<T> = {
  [K in keyof T]: Exclude<T[K], undefined>;
};
```

### Условные типы для маппинга типов

```typescript
// Преобразование типов в зависимости от их формы
type TransformType<T> = T extends string 
  ? `transformed_${T}`
  : T extends number 
    ? `${T}_number`
    : T extends boolean
      ? T extends true
        ? 'true_value'
        : 'false_value'
      : T;

type TransformedString = TransformType<'hello'>; // "transformed_hello"
type TransformedNumber = TransformType<42>; // "42_number"
type TransformedBoolean = TransformType<true>; // "true_value"
```

## Заключение

Условные типы - мощный инструмент для создания гибких и адаптивных типов в TypeScript. Они позволяют создавать типы, которые изменяются в зависимости от входных параметров, что особенно полезно при создании библиотек и фреймворков.

См. также: [[advanced-utility-types]] | [[mapped-types-tricks]] | [[type-inference-tricks]]
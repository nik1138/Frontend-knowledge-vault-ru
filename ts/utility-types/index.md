# Утилиты типов TypeScript

Утилиты типов - это встроенные и пользовательские типы-помощники, которые позволяют преобразовывать и манипулировать типами в TypeScript. Они значительно упрощают работу с типами и помогают создавать более гибкие и переиспользуемые типы.

## Содержание раздела

1. [[ts/utility-types/built-in-object-transformations|Встроенные утилиты для преобразования объектов]] - Partial, Required, Readonly и другие
2. [[ts/utility-types/built-in-property-selection|Встроенные утилиты для выбора свойств]] - Pick, Omit, Extract, Exclude
3. [[ts/utility-types/built-in-function-utilities|Встроенные утилиты для работы с функциями]] - ReturnType, Parameters и другие
4. [[ts/utility-types/built-in-string-utilities|Встроенные утилиты для работы со строками]] - Uppercase, Lowercase и другие
5. [[ts/utility-types/other-built-in-utilities|Другие встроенные утилиты]] - Record, NonNullable, ThisType, Awaited

## Введение

Утилиты типов - это одна из самых мощных возможностей системы типов TypeScript. Они позволяют создавать новые типы на основе существующих, преобразуя их различными способами. Это делает систему типов не только статической, но и выразительной, позволяя описывать сложные отношения между типами.

## Основные категории встроенных утилит

### 1. Преобразование свойств объектов

Эти утилиты позволяют изменять характеристики свойств объектов:

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

// Partial - делает все свойства опциональными
type PartialUser = Partial<User>;

// Required - делает все свойства обязательными
type RequiredUser = Required<Partial<User>>;

// Readonly - делает все свойства только для чтения
type ReadonlyUser = Readonly<User>;
```

### 2. Выбор и исключение свойств

Эти утилиты позволяют выбирать или исключать определенные свойства:

```typescript
interface User {
  name: string;
  age: number;
  email: string;
  password: string;
}

// Pick - выбирает определенные свойства
type UserProfile = Pick<User, "name" | "email">;

// Omit - исключает определенные свойства
type PublicUser = Omit<User, "password">;
```

### 3. Работа с функциями

Эти утилиты позволяют извлекать информацию о функциях:

```typescript
function getUser(id: string): User {
  // ...
}

// ReturnType - получает тип возвращаемого значения
type UserReturnType = ReturnType<typeof getUser>;

// Parameters - получает типы параметров
type UserParams = Parameters<typeof getUser>;
```

### 4. Работа со строками

Эти утилиты позволяют преобразовывать строковые типы:

```typescript
// Uppercase - преобразует в верхний регистр
type Upper = Uppercase<"hello">; // "HELLO"

// Lowercase - преобразует в нижний регистр
type Lower = Lowercase<"HELLO">; // "hello"

// Capitalize - делает первую букву заглавной
type Cap = Capitalize<"hello">; // "Hello"

// Uncapitalize - делает первую букву строчной
type Uncap = Uncapitalize<"Hello">; // "hello"
```

## Пользовательские утилиты

Кроме встроенных утилит, можно создавать собственные типы-помощники:

```typescript
// Утилита для создания глубоко неизменяемых типов
type DeepReadonly<T> = T extends object 
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> } 
  : T;

// Утилита для выбора свойств определенного типа
type PickByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

// Утилита для получения типа асинхронной функции
type AsyncReturnType<T extends (...args: any) => Promise<any>> = 
  T extends (...args: any) => Promise<infer R> ? R : never;
```

## Практическое применение

### 1. Создание гибких API

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
}

// Для создания, обновления и отображения пользователей
type CreateUserDto = Omit<User, "id" | "password">;
type UpdateUserDto = Partial<Omit<User, "id" | "password">>;
type PublicUser = Omit<User, "password">;
```

### 2. Конфигурационные объекты

```typescript
interface AppConfig {
  apiUrl: string;
  timeout: number;
  debug: boolean;
  retries: number;
}

// Опциональная конфигурация
type PartialConfig = Partial<AppConfig>;

// Конфигурация с значениями по умолчанию
type DefaultConfig = Required<PartialConfig>;
```

### 3. Работа с API ответами

```typescript
interface ApiResponse<T> {
  data: T | null;
  error: string | null;
  status: number;
}

// Для извлечения типа данных из ответа
type ExtractData<T> = T extends ApiResponse<infer U> ? U : never;

// Для создания строгих типов ответов
type StrictApiResponse<T> = {
  data: NonNullable<T>;
  error: null;
  status: 200;
};
```

## Комбинирование утилит

Утилиты можно комбинировать для создания сложных преобразований:

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

// Создаем тип для обновления пользователя
type UserUpdate = Partial<Omit<User, "id" | "createdAt" | "updatedAt">>;
// {
//   name?: string;
//   email?: string;
//   password?: string;
// }

// Создаем тип для профиля пользователя
type UserProfile = Pick<User, "name" | "email">;
// {
//   name: string;
//   email: string;
// }

// Создаем тип для безопасного API ответа
type SafeUserResponse = Omit<User, "password">;
// {
//   id: string;
//   name: string;
//   email: string;
//   createdAt: Date;
//   updatedAt: Date;
// }
```

## Ограничения и лучшие практики

### 1. Производительность

Сложные комбинации утилит могут замедлять компиляцию:

```typescript
// Избегайте чрезмерно сложных вложенных утилит
type ComplexType = DeepPartial<DeepReadonly<DeepRequired<SomeType>>>;

// Лучше разбить на несколько шагов
type Step1 = DeepPartial<SomeType>;
type Step2 = DeepReadonly<Step1>;
type Step3 = DeepRequired<Step2>;
```

### 2. Читаемость

Слишком сложные типы могут быть трудны для понимания:

```typescript
// Сложный для понимания тип
type Complex = Pick<Partial<Record<keyof User, NonNullable<User[keyof User]>>>, "name" | "email">;

// Более читаемый вариант
type UserFields = keyof User;
type NonNullUserFields = NonNullable<User[UserFields]>;
type UserRecord = Record<UserFields, NonNullUserFields>;
type PartialUserRecord = Partial<UserRecord>;
type UserProfile = Pick<PartialUserRecord, "name" | "email">;
```

## Связи с другими концепциями

Утилиты типов тесно связаны с другими аспектами системы типов TypeScript:

- [[ts/advanced-types/conditional-types|Условные типы]] - Многие утилиты основаны на условных типах
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - Основа для большинства утилит
- [[ts/advanced-types/template-literal-types|Типы-шаблонные литералы]] - Используются в строковых утилитах
- [[ts/type-system/type-inference|Вывод типов]] - Многие утилиты используют infer

## Рекомендации по изучению

1. Начните с простых встроенных утилит (Partial, Pick, Omit)
2. Освойте работу с утилитами для функций (ReturnType, Parameters)
3. Изучите строковые утилиты (Uppercase, Lowercase и т.д.)
4. Практикуйтесь в создании собственных утилит
5. Учитесь комбинировать утилиты для сложных преобразований
6. Изучите исходный код встроенных утилит для понимания их реализации

## Следующие шаги

- [[ts/advanced-generics/Продвинутые_дженерики|Продвинутые дженерики]] - Сложные паттерны с дженериками
- [[ts/advanced-types/conditional-types|Условные типы]] - Типы, зависящие от условий
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - Преобразование существующих типов
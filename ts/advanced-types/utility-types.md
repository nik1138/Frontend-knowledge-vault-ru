# Utility Types

Утилиты типов (Utility Types) в TypeScript - это встроенные и пользовательские типы, которые предоставляют полезные трансформации типов для повседневного использования. Они помогают быстро создавать производные типы из существующих.

## Встроенные утилиты TypeScript

### [Partial<T>](partial.md)
Делает все свойства типа T необязательными.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
}

type PartialUser = Partial<User>;
// Результат: { id?: number; name?: string; email?: string; isActive?: boolean; }

// Практическое применение: обновление объекта
function updateUser(id: number, updates: Partial<User>) {
  // Обновление только указанных полей
  console.log(`Updating user ${id} with:`, updates);
}

const partialData: PartialUser = { name: "New Name", email: "new@example.com" };
updateUser(123, partialData); // OK
```

### [Required<T>](required.md)
Делает все свойства типа T обязательными.

```typescript
interface UserWithOptional {
  id?: number;
  name?: string;
  email?: string;
}

type RequiredUser = Required<UserWithOptional>;
// Результат: { id: number; name: string; email: string; }

// Все поля теперь обязательны
const user: RequiredUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com"
}; // Все поля обязательны
```

### [Readonly<T>](readonly.md)
Делает все свойства типа T только для чтения.

```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  retryCount: number;
}

type ReadonlyConfig = Readonly<Config>;
// Все свойства помечены как readonly

const config: ReadonlyConfig = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retryCount: 3
};

// config.timeout = 10000; // ОШИБКА: не может быть изменено
```

### [Pick<T, K>](pick.md)
Создает тип с подмножеством свойств K из типа T.

```typescript
// Выбор только нужных свойств
type UserPreview = Pick<User, "id" | "name">;
// Результат: { id: number; name: string; }

const preview: UserPreview = {
  id: 1,
  name: "Alice"
}; // OK, остальные поля не требуются

// Практическое применение: формы
interface UserForm {
  id: number;
  name: string;
  email: string;
  password: string;
  confirmPassword: string;
}

type LoginFormFields = Pick<UserForm, "email" | "password">;
// { email: string; password: string; }
```

### [Omit<T, K>](omit.md)
Создает тип, исключая свойства K из типа T.

```typescript
// Исключение чувствительных полей
type PublicUser = Omit<User, "password" | "email">;
// Результат: { id: number; name: string; isActive: boolean; }

const publicInfo: PublicUser = {
  id: 1,
  name: "Alice",
  isActive: true
}; // Не содержит password или email

// Практическое применение: DTO (Data Transfer Objects)
interface UserEntity {
  id: number;
  name: string;
  email: string;
  password: string; // чувствительное поле
  createdAt: Date;
  updatedAt: Date;
}

type UserDto = Omit<UserEntity, "password">;
// { id: number; name: string; email: string; createdAt: Date; updatedAt: Date; }
```

## Продвинутые встроенные утилиты

### [Exclude<T, U>](exclude.md)
Исключает из T типы, которые присутствуют в U.

```typescript
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
type T2 = Exclude<string | number | (() => void), Function>;  // string | number

// Практическое применение: фильтрация	union типов
type Status = "active" | "inactive" | "pending" | "deleted";
type ActiveStatus = Exclude<Status, "deleted">; // "active" | "inactive" | "pending"

// Фильтрация типов функций
type NonFunctionProps<T> = {
  [K in keyof T as T[K] extends Function ? never : K]: T[K]
};
```

### [Extract<T, U>](extract.md)
Извлекает из T типы, которые присутствуют в U.

```typescript
type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1 = Extract<string | number | (() => void), Function>;  // () => void
type T2 = Extract<string | number | boolean, string | boolean>;  // string | boolean

// Практическое применение: извлечение функциональных свойств
interface ApiService {
  getData: () => Promise<any>;
  post: (data: any) => Promise<any>;
  config: { timeout: number };
  name: string;
}

type MethodNames = Extract<keyof ApiService, "getData" | "post">; // "getData" | "post"
type FunctionProps<T> = {
  [K in keyof T as T[K] extends Function ? K : never]: T[K]
};

type ServiceMethods = FunctionProps<ApiService>;
// { getData: () => Promise<any>; post: (data: any) => Promise<any>; }
```

### [NonNullable<T>](nonnullable.md)
Исключает типы `null` и `undefined` из T.

```typescript
type T0 = NonNullable<string | number | undefined>; // string | number
type T1 = NonNullable<string[] | null | undefined>; // string[]

// Практическое применение: работа с nullable полями
interface ApiResponse<T> {
  data: T | null;
  error?: string | null;
}

type ValidResponse<T> = {
  data: NonNullable<ApiResponse<T>["data"]>;
  error?: NonNullable<ApiResponse<T>["error"]>;
};
```

### [Parameters<T>](parameters.md)
Получает типы параметров функции.

```typescript
declare function f1(arg: { a: number; b: string }): void;
type T0 = Parameters<() => string>;           // []
type T1 = Parameters<(s: string) => void>;    // [string]
type T2 = Parameters<typeof f1>;              // [{ a: number, b: string }]
type T3 = Parameters<any>;                    // unknown[]
type T4 = Parameters<never>;                  // never

// Практическое применение: типобезопасные декораторы
function withLogging<T extends (...args: any[]) => any>(
  fn: T
): (...args: Parameters<T>) => ReturnType<T> {
  return function(...args: Parameters<T>): ReturnType<T> {
    console.log("Calling function with args:", args);
    const result = fn(...args);
    console.log("Function returned:", result);
    return result;
  };
}
```

### [ConstructorParameters<T>](constructor-parameters.md)
Получает типы параметров конструктора класса.

```typescript
type T0 = ConstructorParameters<ErrorConstructor>; // [string?]
type T1 = ConstructorParameters<FunctionConstructor>; // string[]
type T2 = ConstructorParameters<RegExpConstructor>; // [string, string?]

// Практическое применение: создание экземпляров через factory
class HttpClient {
  constructor(
    public baseUrl: string,
    public timeout: number = 5000,
    public retries: number = 3
  ) {}
}

type HttpClientParams = ConstructorParameters<typeof HttpClient>;
// [string, number?, number?] - типы параметров конструктора

function createHttpClient(...args: HttpClientParams): HttpClient {
  return new HttpClient(...args);
}
```

### [ReturnType<T>](return-type.md)
Получает тип возвращаемого значения функции.

```typescript
declare function f1(): { a: number; b: string };
type T0 = ReturnType<() => string>;                   // string
type T1 = ReturnType<(s: string) => void>;            // void
type T2 = ReturnType<(<T>() => T)>;                   // {}
type T3 = ReturnType<(<T extends U, U extends number[]>() => T)>; // number[]
type T4 = ReturnType<typeof f1>;                      // { a: number, b: string }
type T5 = ReturnType<any>;                            // any
type T6 = ReturnType<never>;                          // any
// type T7 = ReturnType<string>;                        // ОШИБКА
// type T8 = ReturnType<Function>;                      // ОШИБКА

// Практическое применение: типобезопасная работа с API
async function fetchUsers(): Promise<{ id: number; name: string }[]> {
  // логика получения пользователей
  return [{ id: 1, name: "Alice" }];
}

type UsersResponse = ReturnType<typeof fetchUsers>;
// Promise<{ id: number; name: string; }[]>

type UnwrappedUsers = Awaited<UsersResponse>;
// { id: number; name: string; }[]
```

## Продвинутые встроенные утилиты

### [InstanceType<T>](instance-type.md)
Получает тип экземпляра класса.

```typescript
class C {
  x = 0;
  y = 0;
}

type T0 = InstanceType<typeof C>;      // C
type T1 = InstanceType<any>;           // any
type T2 = InstanceType<never>;         // any
// type T3 = InstanceType<string>;       // ОШИБКА

// Практическое применение: фабрики
function createInstance<T extends new (...args: any[]) => any>(
  constructor: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new constructor(...args);
}

const instance = createInstance(HttpClient, "https://api.example.com", 3000);
// Тип: HttpClient
```

### [ThisParameterType<T>](this-parameter-type.md)
Получает тип параметра `this` функции.

```typescript
function toHex(this: Number) {
  return this.toString(16);
}

type T0 = ThisParameterType<typeof toHex>;  // Number

// Практическое применение: обертки методов
function withThisBinding<T, R>(
  method: (this: T, ...args: any[]) => R,
  context: T
): (...args: Parameters<(this: T, ...args: any[]) => R>) => R {
  return function(...args: Parameters<(this: T, ...args: any[]) => R>): R {
    return method.apply(context, args);
  };
}
```

## Создание пользовательских утилит

### Основные пользовательские утилиты

```typescript
// DeepPartial - глубокая версия Partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : DeepPartial<T[K]> 
    : T[K]
};

interface DeepObject {
  user: {
    name: string;
    profile: {
      age: number;
      settings: {
        theme: string;
      };
    };
  };
}

type DeepPartialObject = DeepPartial<DeepObject>;
// { user?: { name?: string; profile?: { age?: number; settings?: { theme?: string; }; }; }; }

// DeepReadonly - глубокая версия Readonly
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : DeepReadonly<T[K]> 
    : T[K]
};

// Mutable (обратный Readonly)
type Mutable<T> = {
  -readonly [K in keyof T]: T[K]
};

// OptionalKeys - получение ключей опциональных свойств
type OptionalKeys<T> = {
  [K in keyof T]: {} extends Pick<T, K> ? K : never
}[keyof T];

interface ExampleType {
  required: string;
  optional?: number;
  alsoOptional?: boolean;
}

type OptionalProps = OptionalKeys<ExampleType>; // "optional" | "alsoOptional"

// RequiredKeys - получение ключей обязательных свойств
type RequiredKeys<T> = {
  [K in keyof T]: {} extends Pick<T, K> ? never : K
}[keyof T];

type RequiredProps = RequiredKeys<ExampleType>; // "required"
```

### Утилиты для работы с полями

```typescript
// Get - получение типа значения по пути
type Get<T, K extends string> = K extends `${infer First}.${infer Rest}`
  ? First extends keyof T
    ? Get<T[First], Rest>
    : never
  : K extends keyof T
    ? T[K]
    : never;

interface NestedObject {
  user: {
    profile: {
      name: string;
      address: {
        street: string;
        city: string;
      };
    };
  };
}

type UserNameType = Get<NestedObject, "user.profile.name">; // string
type StreetType = Get<NestedObject, "user.profile.address.street">; // string

// Merge - объединение двух объектов
type Merge<T, U> = {
  [K in keyof T | keyof U]: K extends keyof U ? U[K] : K extends keyof T ? T[K] : never;
};

type Merged = Merge<{ a: number; b: string }, { b: boolean; c: number }>;
// { a: number; b: boolean; c: number; } (b из второго типа переопределяет b из первого)

// Diff - разница между двумя типами
type Diff<T, U> = Pick<T, Exclude<keyof T, keyof U>>;

type Difference = Diff<User, { id: number }>;
// { name: string; email: string; isActive: boolean; } (id исключен)
```

### Утилиты для функций

```typescript
// Overwrite - перезапись свойств
type Overwrite<T, U> = Pick<T, Exclude<keyof T, keyof U>> & U;

type OverwrittenUser = Overwrite<User, { name: string; age: number }>;
// { id: number; email: string; isActive: boolean; name: string; age: number; }

// Values - получение типа значений всех свойств
type Values<T> = T[keyof T];

type UserValues = Values<User>; // number | string | boolean

// UnionToIntersection - преобразование объединения в пересечение
type UnionToIntersection<U> = 
  (U extends any ? (k: U) => void : never) extends 
  (k: infer I) => void ? I : never;

type T = UnionToIntersection<{ a: string } | { b: number } | { c: boolean }>;
// { a: string } & { b: number } & { c: boolean }
// что равно: { a: string; b: number; c: boolean; }

// ToTuple - преобразование объединения в кортеж
type ToTuple<T> = [T] extends [never] ? [] : [...ts: ToTuple<Exclude<T, [T] extends [never] ? any : T extends T ? T : never>>];
```

## Практические примеры использования

### Пользовательские утилиты для API

```typescript
// Тип для создания DTO с обязательными полями
type RequiredDto<T> = Omit<T, keyof Partial<T>> & Required<Partial<T>>;

// Тип для исключения полей в update операциях
type Updatable<T> = Omit<T, 'id' | 'createdAt'>;

// Тип для создания нового объекта (без ID и timestamps)
type CreationDto<T> = Omit<T, 'id' | 'createdAt' | 'updatedAt'>;

// Пример использования
interface UserEntity {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

type CreateUserDto = CreationDto<UserEntity>;
// { name: string; email: string; password: string; }

type UpdateUserDto = Partial<Updatable<UserEntity>>;
// { name?: string; email?: string; password?: string; updatedAt?: Date; }
```

### Утилиты для валидации

```typescript
// Тип для указания обязательных полей
type WithRequired<T, K extends keyof T> = T & Required<Pick<T, K>>;

type UserWithRequiredEmail = WithRequired<User, 'email'>;
// User с обязательным полем email

// Тип для указания опциональных полей
type WithOptional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type UserWithOptionalEmail = WithOptional<User, 'email'>;
// User с необязательным полем email
```

### Утилиты для конфигурации

```typescript
// Тип для опциональной конфигурации
type Configurable<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface ServerConfig {
  host: string;
  port: number;
  ssl: boolean;
  timeout: number;
  debug: boolean;
}

type OptionalConfig = Configurable<ServerConfig, 'ssl' | 'timeout' | 'debug'>;
// { host: string; port: number; ssl?: boolean; timeout?: number; debug?: boolean; }

// Тип для строго определенной конфигурации
type StrictConfig<T> = {
  [K in keyof T]: [T[K]] extends [never] ? T[K] : T[K]
};
```

## Продвинутые паттерны

### Утилиты с условными типами

```typescript
// ConditionalPick - выбор свойств по условию
type ConditionalPick<T, Condition> = {
  [K in keyof T as T[K] extends Condition ? K : never]: T[K]
};

type StringProps = ConditionalPick<User, string>;
// { name: string; email: string; }

// ConditionalOmit - исключение свойств по условию
type ConditionalOmit<T, Condition> = {
  [K in keyof T as T[K] extends Condition ? never : K]: T[K]
};

type NonStringProps = ConditionalOmit<User, string>;
// { id: number; isActive: boolean; }

// Filter - фильтрация типов с условием
type Filter<T, K extends keyof T, Condition> = {
  [P in K as T[P] extends Condition ? P : never]: T[P]
};

// Пример: получить только числовые поля
type NumericFields<T> = {
  [K in keyof T as T[K] extends number ? K : never]: T[K]
};

type UserNumericFields = NumericFields<User>;
// { id: number; }
```

### Утилиты для преобразования

```typescript
// Преобразование в CamelCase ключей
type CamelCase<S extends string> = S extends `${infer P1}_${infer P2}${infer P3}`
  ? `${Lowercase<P1>}${Uppercase<P2>}${CamelCase<P3>}`
  : Lowercase<S>;

type SnakeToCamelCase<T> = {
  [K in keyof T as K extends string ? CamelCase<K> : K]: T[K]
};

interface SnakeCaseUser {
  user_id: number;
  first_name: string;
  last_name: string;
}

type CamelCaseUser = SnakeToCamelCase<SnakeCaseUser>;
// { userId: number; firstName: string; lastName: string; }

// Преобразование с добавлением префикса/суффикса
type WithPrefix<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K]
};

type PrefixedUser = WithPrefix<User, "api_">;
// { api_id: number; api_name: string; api_email: string; api_isActive: boolean; }
```

## Лучшие практики

### 1. Используйте встроенные утилиты когда возможно
```typescript
// Хорошо: используете встроенные утилиты
type ReadOnlyUser = Readonly<User>;
type PartialConfig = Partial<ServerConfig>;

// Избегайте переизобретения колеса
// ПЛОХО: реализуете свой Partial
type MyPartial<T> = { [K in keyof T]?: T[K] }; // используйте Partial<T> вместо
```

### 2. Документируйте сложные пользовательские утилиты
```typescript
/**
 * DeepReadonly - делает все свойства T и всех вложенных объектов только для чтения
 * Не применяется к функциям (оставляются как есть)
 */
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : DeepReadonly<T[K]> 
    : T[K]
};
```

### 3. Тестируйте пользовательские утилиты
```typescript
// Пример тестирования пользовательской утилиты
type TestUser = { id: number; name: string; profile: { age: number; } };

type Expected = { 
  readonly id: number; 
  readonly name: string; 
  readonly profile: { 
    readonly age: number; 
  }; 
};

type Actual = DeepReadonly<TestUser>;

// Должно быть true если DeepReadonly работает правильно
type Test = [Expected] extends [Actual] ? true : false;
```

## Ограничения и подводные камни

### 1. Производительность
```typescript
// Сложные рекурсивные утилиты могут замедлить компиляцию
// type VeryComplexUtility<T> = /* сложная рекурсивная логика */; // избегайте чрезмерной сложности
```

### 2. Ограничения infer
```typescript
// Infer не всегда работает в сложных сценариях
type BrokenInfer<T> = T extends Record<infer K, infer V> ? [K, V] : never;
// Может не работать как ожидается со сложными структурами
```

### 3. Сложность дебага
```typescript
// Слишком сложные утилиты трудно отлаживать
// Лучше разбивать на более простые утилиты
```

## Современные тенденции

### Утилиты в библиотеках
```typescript
// Примеры из популярных библиотек
// fp-ts, io-ts, zod и другие предоставляют свои утилиты

// В TypeScript 4.1+ появились Template Literal Types
type GetterName<T extends string> = `get${Capitalize<T>}`;

type NameGetter = GetterName<"name">; // "getName"
type EmailGetter = GetterName<"email">; // "getEmail"
```

### Утилиты для работы с JSON
```typescript
// Тип для безопасной работы с JSON
type JsonPrimitive = string | number | boolean | null;
type JsonValue = JsonPrimitive | JsonObject | JsonValue[];
type JsonObject = { [key: string]: JsonValue };

type JsonSerializable<T> = T extends JsonValue ? T : never;

// Проверка, может ли тип быть сериализован в JSON
type IsJsonSerializable<T> = [T] extends [JsonSerializable<T>] ? true : false;
```

## Сравнение с другими подходами

| Утилита | Назначение | Аналог в других языках |
|---------|------------|------------------------|
| Partial | Опциональные свойства | Option<T> в Rust, optional<T> в C++ |
| Readonly | Только для чтения | const в C++, readonly в Kotlin |
| Pick | Выбор подмножества | select в LINQ, pick в Ruby |
| Omit | Исключение свойств | except в Ruby, без аналога в C++ |
| Exclude/Extract | Фильтрация union типов | Set operations в математике |
| Parameters/ReturnType | Интроспекция функций | Reflection в Java/C#, RTTI в C++ |

## Связь с другими концепциями
- [[../type-system/conditional-types]] - основа для сложных утилит
- [[../type-system/mapped-types]] - сопоставляющие типы для создания утилит
- [[../generics/advanced]] - обобщения для утилит
- [[../functional-programming/type-level]] - функциональное программирование на уровне типов
- [[../modules/type-safe-apis]] - типобезопасные API с использованием утилит
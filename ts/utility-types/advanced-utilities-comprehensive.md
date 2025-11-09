# Расширенные утилиты типов в TypeScript

Утилиты типов в TypeScript - это встроенные и пользовательские типы-помощники, которые позволяют создавать новые типы на основе существующих. Они значительно упрощают работу с типами и помогают создавать более гибкие и переиспользуемые типы.

## Встроенные утилиты типов

### 1. Основные утилиты преобразования

```typescript
// Partial<T> - делает все свойства опциональными
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

type PartialUser = Partial<User>;
// Эквивалентно: { id?: number; name?: string; email?: string; age?: number; }

// Required<T> - делает все свойства обязательными
type OptionalUser = { id?: number; name?: string; };
type RequiredOptionalUser = Required<OptionalUser>;
// Эквивалентно: { id: number; name: string; }

// Readonly<T> - делает все свойства только для чтения
type ReadonlyUser = Readonly<User>;
// Все свойства User становятся readonly

// Pick<T, K> - выбирает подмножество свойств
type UserPreview = Pick<User, "name" | "email">;
// { name: string; email: string; }

// Omit<T, K> - исключает определенные свойства
type UserWithoutId = Omit<User, "id">;
// { name: string; email: string; age: number; }
```

### 2. Утилиты для работы с функциями

```typescript
// ReturnType<T> - извлекает тип возвращаемого значения функции
function getUser(id: number): User {
  return { id, name: "John", email: "john@example.com", age: 30 };
}

type GetUserReturnType = ReturnType<typeof getUser>; // User

// Parameters<T> - извлекает типы параметров функции
type GetUserParams = Parameters<typeof getUser>; // [number]

// ConstructorParameters<T> - извлекает типы параметров конструктора
class UserClass {
  constructor(public id: number, public name: string) {}
}

type UserClassParams = ConstructorParameters<typeof UserClass>; // [number, string]

// InstanceType<T> - извлекает тип экземпляра класса
type UserClassInstance = InstanceType<typeof UserClass>; // UserClass
```

### 3. Утилиты для фильтрации типов

```typescript
// Exclude<T, U> - исключает типы из T, которые присутствуют в U
type T0 = Exclude<"a" | "b" | "c", "a" | "c">; // "b"

// Extract<T, U> - извлекает типы из T, которые присутствуют в U
type T1 = Extract<"a" | "b" | "c", "a" | "c">; // "a" | "c"

// NonNullable<T> - исключает null и undefined
type T2 = NonNullable<string | number | undefined>; // string | number

// ThisParameterType<T> - извлекает тип this параметра функции
function thisFunction(this: { name: string }, message: string) {
  return `${this.name}: ${message}`;
}

type ThisType = ThisParameterType<typeof thisFunction>; // { name: string }

// OmitThisParameter<T> - удаляет тип this параметра
type FunctionWithoutThis = OmitThisParameter<typeof thisFunction>; // (message: string) => string
```

### 4. Утилиты для работы со строками

```typescript
// Uppercase<T> - преобразует строковые литералы в верхний регистр
type Greeting = "hello world";
type UpperGreeting = Uppercase<Greeting>; // "HELLO WORLD"

// Lowercase<T> - преобразует строковые литералы в нижний регистр
type LowerGreeting = Lowercase<Greeting>; // "hello world"

// Capitalize<T> - делает первую букву заглавной
type CapitalGreeting = Capitalize<Greeting>; // "Hello world"

// Uncapitalize<T> - делает первую букву строчной
type UncapGreeting = Uncapitalize<Uppercase<Greeting>>; // "hELLO WORLD"
```

## Пользовательские утилиты типов

### 1. Утилиты для глубоких преобразований

```typescript
// Глубокое преобразование в readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object 
    ? T[P] extends Function 
      ? T[P] 
      : DeepReadonly<T[P]> 
    : T[P]
};

interface NestedUser {
  id: number;
  profile: {
    name: string;
    contact: {
      email: string;
      phone: string;
    };
  };
}

type DeepReadonlyNestedUser = DeepReadonly<NestedUser>;
// Все вложенные объекты тоже будут readonly

// Глубокое преобразование в частичный тип
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object 
    ? T[P] extends Function 
      ? T[P] 
      : DeepPartial<T[P]> 
    : T[P]
};

type DeepPartialNestedUser = DeepPartial<NestedUser>;
// Все свойства, включая вложенные, становятся опциональными
```

### 2. Утилиты для работы с ключами и значениями

```typescript
// Извлечение только строковых ключей
type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never
}[keyof T];

type UserStringKeys = StringKeys<User>; // "name" | "email"

// Извлечение только числовых ключей
type NumberKeys<T> = {
  [K in keyof T]: T[K] extends number ? K : never
}[keyof T];

type UserNumberKeys = NumberKeys<User>; // "id" | "age"

// Извлечение значений по ключам
type ValuesOf<T, K extends keyof T> = T[K];

type UserNameType = ValuesOf<User, "name">; // string

// Создание типа с префиксами к ключам
type AddPrefix<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K]
};

type PrefixedUser = AddPrefix<User, "api">;
// { apiId: number; apiName: string; apiEmail: string; apiAge: number; }
```

### 3. Утилиты для валидации и фильтрации

```typescript
// Утилита для выбора свойств определенного типа
type PropertiesOfType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};

type StringProperties = PropertiesOfType<User, string>;
// { name: string; email: string; }

type NumberProperties = PropertiesOfType<User, number>;
// { id: number; age: number; }

// Утилита для исключения свойств определенного типа
type ExcludePropertiesOfType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K]
};

type NonStringProperties = ExcludePropertiesOfType<User, string>;
// { id: number; age: number; }
```

## Продвинутые пользовательские утилиты

### 1. Утилиты для работы с кортежами и массивами

```typescript
// Извлечение типа элемента массива
type ElementOf<T> = T extends (infer U)[] ? U : T;

type ArrayElement = ElementOf<string[]>; // string
type NonArrayElement = ElementOf<number>; // number

// Извлечение первого элемента кортежа
type First<T extends any[]> = T extends [infer U, ...any[]] ? U : never;

type FirstElement = First<[string, number, boolean]>; // string

// Извлечение последнего элемента кортежа
type Last<T extends any[]> = T extends [...any[], infer U] ? U : never;

type LastElement = Last<[string, number, boolean]>; // boolean

// Утилита для объединения кортежей
type Concat<T extends any[], U extends any[]> = [...T, ...U];

type Combined = Concat<[string, number], [boolean, Date]>; // [string, number, boolean, Date]
```

### 2. Утилиты для работы с промисами

```typescript
// Извлечение типа разрешаемого значения промиса
type Awaited<T> = T extends Promise<infer U> 
  ? U extends Promise<any> 
    ? Awaited<U> 
    : U 
  : T;

type ResolvedType = Awaited<Promise<Promise<string>>>; // string

// Утилита для создания типа с разрешенными промисами
type Resolved<T> = T extends Promise<infer U> 
  ? U 
  : T extends {} 
    ? { [K in keyof T]: Resolved<T[K]> } 
    : T;

// Утилита для создания асинхронной версии функции
type AsyncFunction<T extends (...args: any[]) => any> = 
  (...args: Parameters<T>) => Promise<Awaited<ReturnType<T>>>;

function syncFunction(x: number, y: string): boolean {
  return x > 0 && y.length > 0;
}

type AsyncSyncFunction = AsyncFunction<typeof syncFunction>;
// (...args: [number, string]) => Promise<boolean>
```

### 3. Утилиты для создания условных типов

```typescript
// Утилита для выбора типа на основе условия
type If<C extends boolean, T, F> = C extends true ? T : F;

type TrueType = If<true, string, number>; // string
type FalseType = If<false, string, number>; // number

// Утилита для проверки, является ли тип примитивом
type IsPrimitive<T> = 
  T extends string | number | boolean | bigint | symbol | undefined | null 
    ? true 
    : false;

type IsStringPrimitive = IsPrimitive<string>; // true
type IsObjectPrimitive = IsPrimitive<User>; // false

// Утилита для получения типа значения по ключу с возможностью безопасного доступа
type SafeGet<T, K> = K extends keyof T ? T[K] : never;

type UserName = SafeGet<User, "name">; // string
type Invalid = SafeGet<User, "invalid">; // never
```

## Практические применения утилит типов

### 1. Создание типобезопасных мапперов

```typescript
// Утилита для создания маппинга типов
type TypeMap<T> = {
  [K in keyof T]: (value: T[K]) => any;
};

interface ApiData {
  userId: number;
  userName: string;
  isActive: boolean;
  createdAt: string;
}

type Processors = TypeMap<ApiData>;
// {
//   userId: (value: number) => any;
//   userName: (value: string) => any;
//   isActive: (value: boolean) => any;
//   createdAt: (value: string) => any;
// }

// Универсальный маппер
function createMapper<T>() {
  return {
    map: <U>(obj: T, processors: { [K in keyof T]: (value: T[K]) => U[K] }): U => {
      const result = {} as U;
      for (const key in processors) {
        if (processors.hasOwnProperty(key)) {
          result[key] = processors[key](obj[key]);
        }
      }
      return result;
    }
  };
}
```

### 2. Типобезопасная валидация форм

```typescript
// Утилита для определения обязательных полей
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K
}[keyof T];

// Утилита для определения опциональных полей
type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never
}[keyof T];

interface FormData {
  name: string;
  email: string;
  age?: number;
  consent?: boolean;
}

type RequiredFields = RequiredKeys<FormData>; // "name" | "email"
type OptionalFields = OptionalKeys<FormData>; // "age" | "consent"

// Утилита для создания типа валидации формы
type FormValidation<T> = {
  [K in keyof T]: {
    value: T[K];
    isValid: boolean;
    error?: string;
  }
};

type ValidatedFormData = FormValidation<FormData>;
// {
//   name: { value: string; isValid: boolean; error?: string; };
//   email: { value: string; isValid: boolean; error?: string; };
//   age: { value: number | undefined; isValid: boolean; error?: string; };
//   consent: { value: boolean | undefined; isValid: boolean; error?: string; };
// }
```

### 3. Создание типобезопасной системы конфигурации

```typescript
// Утилита для создания иерархической конфигурации
type ConfigSchema<T> = {
  [K in keyof T]: T[K] extends object 
    ? T[K] extends any[] 
      ? T[K] 
      : ConfigSchema<T[K]> & { _default?: T[K] } 
    : { value: T[K]; _default?: T[K] }
};

interface AppConfig {
  server: {
    host: string;
    port: number;
  };
  database: {
    url: string;
    timeout: number;
  };
  features: {
    caching: boolean;
    logging: {
      enabled: boolean;
      level: "debug" | "info" | "warn" | "error";
    };
  };
}

type TypedConfig = ConfigSchema<AppConfig>;
// Создает тип с дополнительной информацией о конфигурации
```

## Продвинутые паттерны

### 1. Утилиты для работы с событиями

```typescript
// Утилита для создания типобезопасной системы событий
type EventPayloadMap = {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  error: { message: string; code: number };
};

type EventName = keyof EventPayloadMap;

// Утилита для извлечения типа полезной нагрузки события
type EventPayload<T extends EventName> = EventPayloadMap[T];

// Утилита для создания типа обработчика события
type EventHandler<T extends EventName> = (payload: EventPayload<T>) => void;

// Утилита для создания типа слушателей событий
type EventListeners = {
  [K in EventName]?: EventHandler<K>[];
};

const listeners: EventListeners = {};
```

### 2. Утилиты для работы с API ответами

```typescript
// Универсальные типы для API ответов
interface ApiResponse<T> {
  data: T;
  status: number;
  message?: string;
}

interface ApiError {
  error: string;
  code: number;
  details?: any;
}

type ApiResult<T> = ApiResponse<T> | ApiError;

// Утилита для извлечения типа данных из API результата
type ApiData<T> = T extends ApiResponse<infer U> ? U : never;

type SuccessResponse = ApiResponse<User>;
type UserData = ApiData<SuccessResponse>; // User

// Утилита для создания типа успешного ответа
type SuccessResponseOf<T> = T extends any ? ApiResponse<T> : never;

type UserSuccessResponse = SuccessResponseOf<User>;
// { data: User; status: number; message?: string; }
```

### 3. Утилиты для создания DSL (Domain Specific Language)

```typescript
// Утилита для создания типобезопасного DSL для валидации
type Validator<T> = (value: T) => boolean;

interface ValidationRule<T> {
  validator: Validator<T>;
  message: string;
}

type ValidationSchema<T> = {
  [K in keyof T]: ValidationRule<T[K]>[];
};

interface UserRegistration {
  email: string;
  password: string;
  age: number;
}

const userValidationSchema: ValidationSchema<UserRegistration> = {
  email: [
    { 
      validator: (email) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email), 
      message: "Неверный формат email" 
    }
  ],
  password: [
    { 
      validator: (password) => password.length >= 8, 
      message: "Пароль должен быть не менее 8 символов" 
    }
  ],
  age: [
    { 
      validator: (age) => age >= 18, 
      message: "Возраст должен быть не менее 18" 
    }
  ]
};
```

## Ошибки и лучшие практики

### 1. Ограничения производительности

```typescript
// Слишком сложные утилиты могут замедлить компиляцию
// type VeryComplex = DeepNestedUtility<ComplexType>; // Может быть медленным

// Лучше разбивать на несколько шагов
type Step1 = FirstTransformation<ComplexType>;
type Step2 = SecondTransformation<Step1>;
type FinalType = ThirdTransformation<Step2>;
```

### 2. Лучшие практики

```typescript
// 1. Документируйте сложные утилиты типов
/**
 * Утилита для создания глубоко неизменяемого типа
 * Преобразует все свойства объекта и его вложенных объектов в readonly
 */
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object 
    ? T[P] extends Function 
      ? T[P] 
      : DeepReadonly<T[P]> 
    : T[P]
};

// 2. Используйте именованные типы для сложных выражений
type UserWithoutSensitiveData = Omit<User, "password" | "ssn" | "creditCard">;
type ValidatedUser = UserWithoutSensitiveData & { isValidated: true };

// 3. Избегайте излишне сложных вложенных утилит
// Сложно читать:
// type Complex = DeepPartial<DeepReadonly<Required<Pick<User, 'name' | 'email'>>>>;

// Лучше:
type RequiredUserFields = Required<Pick<User, 'name' | 'email'>>;
type ProtectedUserFields = DeepReadonly<RequiredUserFields>;
type FinalUserType = DeepPartial<ProtectedUserFields>;
```

## Практические примеры из реальной разработки

### 1. Утилиты для работы с ORM

```typescript
// Утилиты для создания типобезопасных сущностей ORM
interface BaseEntity {
  id: number;
  createdAt: Date;
  updatedAt: Date;
}

// Утилита для создания типа сущности без полей базовой сущности
type EntityWithoutBase<T> = Omit<T, keyof BaseEntity>;

// Утилита для создания типа создания сущности (без id)
type CreateEntity<T> = Omit<T, 'id'>;

// Утилита для создания типа обновления сущности (все поля опциональны)
type UpdateEntity<T> = Partial<Omit<T, 'id' | 'createdAt'>>;

interface UserEntity extends BaseEntity {
  name: string;
  email: string;
  age: number;
}

type CreateUserDto = CreateEntity<UserEntity>;
// { name: string; email: string; age: number; createdAt: Date; updatedAt: Date; }

type UpdateUserDto = UpdateEntity<UserEntity>;
// { name?: string; email?: string; age?: number; updatedAt?: Date; }
```

### 2. Утилиты для работы с Redux

```typescript
// Утилиты для типизации Redux actions
interface Action<T extends string> {
  type: T;
}

interface ActionWithPayload<T extends string, P> extends Action<T> {
  payload: P;
}

// Утилита для создания типа action creator
type ActionCreator<T extends string, P = undefined> = 
  P extends undefined 
    ? () => Action<T>
    : (payload: P) => ActionWithPayload<T, P>;

// Утилита для извлечения типа полезной нагрузки из action
type PayloadType<T extends ActionWithPayload<string, any>> = T['payload'];

// Утилита для создания типа reducer
type Reducer<S, A extends Action<string>> = (state: S, action: A) => S;
```

### 3. Утилиты для работы с GraphQL

```typescript
// Утилиты для работы с GraphQL типами
type GraphQLResponse<T> = {
  data?: T;
  errors?: Array<{ message: string; locations?: Array<{ line: number; column: number }>; path?: string[] }>;
};

// Утилита для извлечения типа данных из GraphQL ответа
type GraphQLData<T> = T extends GraphQLResponse<infer U> ? U : never;

// Утилита для создания типа мутации с аргументами
type MutationWithArgs<MutationName extends string, Args, Return> = {
  [K in MutationName]: (args: Args) => Return;
};

type CreateUserMutation = MutationWithArgs<'createUser', { input: User }, User>;
// { createUser: (args: { input: User }) => User; }
```

## Лучшие практики

1. **Используйте встроенные утилиты когда это возможно** - они оптимизированы и проверены
2. **Документируйте сложные пользовательские утилиты** - особенно когда они используют условные типы
3. **Разбивайте сложные утилиты на несколько шагов** - для лучшей читаемости
4. **Тестируйте утилиты типов** - особенно пользовательские
5. **Избегайте излишне сложных вложенных утилит** - это может замедлить компиляцию
6. **Создавайте переиспользуемые утилиты** - для часто используемых паттернов

## Связи с другими концепциями

- [[ts/advanced-types/conditional-types|Условные типы]] - основа для многих утилит типов
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - используются во многих утилитах
- [[ts/type-system/type-inference|Вывод типов]] - как утилиты влияют на вывод типов
- [[ts/generics/TS Generics|Обобщения]] - утилиты типов часто параметризованы
- [[ts/type-checking/strategies-best-practices-comprehensive|Стратегии проверки типов]] - как использовать утилиты эффективно
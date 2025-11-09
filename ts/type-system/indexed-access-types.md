# Indexed Access Types

Indexed Access Types (типы доступа по индексу) позволяют получить тип значения по заданному ключу объекта или элемента массива. Это делается с помощью синтаксиса `T[K]`, где `T` - тип объекта, а `K` - тип ключа.

## Основное использование

### Базовый доступ к типу
```typescript
interface Person {
  name: string;
  age: number;
  email: string;
}

// Получение типа свойства по известному ключу
type PersonName = Person["name"];  // string
type PersonAge = Person["age"];    // number
type PersonEmail = Person["email"]; // string

// Получение типа по объединению ключей
type PersonNameOrEmail = Person["name" | "email"]; // string
type PersonStringOrNumber = Person["name" | "age"]; // string | number

// Использование с keyof
type AllPersonTypes = Person[keyof Person]; // string | number (объединение всех значений)
```

## Практические примеры

### 1. Получение типа свойства с использованием переменной
```typescript
// Это невозможно сделать напрямую через переменную JavaScript
interface User {
  id: number;
  username: string;
  email: string;
}

// Но можно использовать в обобщенных функциях
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = { id: 123, username: "john_doe", email: "john@example.com" };

// Получение типа по ключу
type UserIdType = User["id"];        // number
type UsernameType = User["username"]; // string

// Использование функции
const userId = getProperty(user, "id");        // number
const username = getProperty(user, "username"); // string
```

### 2. Работа с массивами
```typescript
// Типы элементов массива
type StringArray = string[];
type ArrayElementType = StringArray[number]; // string

type NumberArray = number[];
type NumElementType = NumberArray[number]; // number

// Работа с кортежами
type Tuple = [string, number, boolean];
type FirstElement = Tuple[0];  // string
type SecondElement = Tuple[1]; // number
type ThirdElement = Tuple[2];  // boolean

// Использование с переменными
type ElementByIndex<I extends number> = Tuple[I];
type DynamicFirst = ElementByIndex<0>;  // string
type DynamicSecond = ElementByIndex<1>; // number
```

### 3. Получение типа вложенного свойства
```typescript
interface Address {
  street: string;
  city: string;
  zipCode: string;
}

interface User {
  name: string;
  age: number;
  address: Address;
  contacts: {
    email: string;
    phone: string;
  }[];
}

// Получение типа вложенного свойства
type UserStreet = User["address"]["street"];     // string
type UserContacts = User["contacts"];            // { email: string; phone: string; }[]
type FirstContact = User["contacts"][0];         // { email: string; phone: string; } | undefined
type ContactEmail = User["contacts"][number]["email"]; // string (доступ к элементу массива по индексу number)
```

## Продвинутое использование

### 1. Доступ через вычисленные ключи
```typescript
// Использование с литералами
type Status = "active" | "inactive";
type Role = "admin" | "user" | "moderator";

interface Permissions {
  active_admin: string[];
  inactive_user: string[];
  active_mod: string[];
}

// Получение типа на основе литеральных типов
type ActiveAdminPerms = Permissions[`${Status}_${Role}`]; 
// string[] (объединение типов всех возможных комбинаций)

// Уточнение типа для конкретной комбинации
type SpecificPerm = Permissions["active_admin"]; // string[]
```

### 2. Создание маппинга типов
```typescript
interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  data_update: { field: string; oldValue: any; newValue: any };
}

// Получение типа для конкретного события
type LoginData = EventMap["user_login"]; // { userId: number; timestamp: Date }
type LogoutData = EventMap["user_logout"]; // { userId: number; duration: number }

// Использование в функции обработки событий
function handleEvent<T extends keyof EventMap>(event: T, data: EventMap[T]) {
  switch (event) {
    case "user_login":
      // data: { userId: number; timestamp: Date }
      console.log(`User ${data.userId} logged in at ${data.timestamp}`);
      break;
    case "user_logout":
      // data: { userId: number; duration: number }
      console.log(`User ${data.userId} logged out after ${data.duration}ms`);
      break;
  }
}
```

### 3. Доступ через keyof
```typescript
interface Config {
  apiUrl: string;
  timeout: number;
  debug: boolean;
}

// Получение типа через keyof
type ConfigValue = Config[keyof Config]; // string | number | boolean (все возможные типы значений)

// Это эквивалентно объединению: Config["apiUrl"] | Config["timeout"] | Config["debug"]

// Получение типа для конкретного ключа
function getConfigValue<K extends keyof Config>(key: K): Config[K] {
  const configs: Config = {
    apiUrl: "https://api.example.com",
    timeout: 5000,
    debug: false
  };
  return configs[key];
}

const timeoutValue = getConfigValue("timeout"); // number
const apiUrlValue = getConfigValue("apiUrl");   // string
```

## Сопоставляющие типы с Indexed Access

### 1. Создание типа с преобразованными значениями
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

// Создание типа, где все свойства становятся строками
type StringifiedUser = {
  [K in keyof User]: string;
};

// Это эквивалентно:
// interface StringifiedUser {
//   id: string;
//   name: string;
//   email: string;
//   createdAt: string;
// }

// Создание типа, где все свойства становятся опциональными
type OptionalUser<T> = {
  [K in keyof T]?: T[K];
};

type MaybeUser = OptionalUser<User>;
// Все свойства User становятся необязательными
```

### 2. Создание типа с фильтрацией
```typescript
interface FormData {
  name: string;
  age: number;
  email: string;
  isValid: boolean;
  submitted: boolean;
}

// Создание типа с только булевыми свойствами
type BooleanFields<T> = {
  [K in keyof T as T[K] extends boolean ? K : never]: T[K]
};

type FormBooleans = BooleanFields<FormData>;
// Результат: { isValid: boolean; submitted: boolean; }

// Создание типа с только строковыми свойствами
type StringFields<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K]
};

type FormStrings = StringFields<FormData>;
// Результат: { name: string; email: string; }
```

## Практические паттерны

### 1. Изменение типа свойства
```typescript
// Утилита для изменения типа одного свойства
type ChangeFieldType<T, Prop extends keyof T, NewType> = {
  [K in keyof T]: K extends Prop ? NewType : T[K];
};

interface User {
  id: number;
  name: string;
  age: number;
  createdAt: Date;
}

type UserWithStringDates = ChangeFieldType<User, "createdAt", string>;
// { id: number; name: string; age: number; createdAt: string; }
```

### 2. Работа с API ответами
```typescript
interface ApiResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}

interface UserProfile {
  id: number;
  name: string;
  email: string;
  preferences: {
    theme: "light" | "dark";
    notifications: boolean;
  };
}

// Получение типа данных из ответа API
type ResponseData<T> = T extends ApiResponse<infer D> ? D : never;

type UserProfileResponse = ApiResponse<UserProfile>;
type UserProfileData = ResponseData<UserProfileResponse>; // UserProfile

// Или более конкретный подход
type ApiResponseData<T> = ApiResponse<T>["data"];
type DirectData = ApiResponseData<UserProfileResponse>; // UserProfile
```

### 3. Создание безопасного getter'а
```typescript
class SafeObjectAccessor<T> {
  constructor(private obj: T) {}
  
  get<K extends keyof T>(key: K): T[K] {
    return this.obj[key];
  }
  
  // Получение вложенного свойства
  nestedGet<K1 extends keyof T, K2 extends keyof T[K1]>(
    key1: K1, 
    key2: K2
  ): T[K1][K2] {
    const nestedObj = this.obj[key1] as Record<string, any>;
    return nestedObj[key2];
  }
}

const user = { 
  profile: { 
    name: "Alice", 
    settings: { theme: "dark", lang: "en" } 
  } 
};

const accessor = new SafeObjectAccessor(user);
const name = accessor.nestedGet("profile", "name"); // "Alice"
type NameType = typeof name; // string
```

## Расширенные примеры

### 1. Работа с конфигурацией
```typescript
interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
}

interface AppConfig {
  name: string;
  version: string;
  database: DatabaseConfig;
}

// Получение глубокого типа
type DeepAccessType = AppConfig["database"]["host"]; // string

// Утилита для получения типа вложенного свойства
type NestedType<T, Path> = Path extends `${infer K}.${infer Rest}`
  ? K extends keyof T
    ? NestedType<T[K], Rest>
    : never
  : Path extends keyof T
    ? T[Path]
    : never;

// Использование (упрощенный пример)
type ConfigType = {
  api: {
    url: string;
    timeout: number;
  };
  ui: {
    theme: "light" | "dark";
    lang: string;
  };
};

// Тип для "api.url"
type ApiUrlType = ConfigType["api"]["url"]; // string
```

### 2. Создание системы валидации
```typescript
interface UserForm {
  name: string;
  email: string;
  age: number;
}

// Тип для ошибок валидации
type ValidationError<T> = {
  [K in keyof T]?: string[];
};

const errors: ValidationError<UserForm> = {
  name: ["Name is required"],
  email: ["Invalid email format"],
  // age не указываем, т.к. валидно
};

// Тип для валидаторов
type Validator<T> = {
  [K in keyof T]: (value: T[K]) => boolean;
};

const validators: Validator<UserForm> = {
  name: (value: string) => value.length > 0,
  email: (value: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  age: (value: number) => value >= 0 && value <= 120
};
```

### 3. Создание системы событий
```typescript
interface EventDataMap {
  user_created: { userId: number; timestamp: Date };
  user_updated: { userId: number; fields: string[]; timestamp: Date };
  user_deleted: { userId: number; reason?: string; timestamp: Date };
}

// Тип для обработчика события
type EventHandler<T extends keyof EventDataMap> = (data: EventDataMap[T]) => void;

// Регистрация обработчиков
const handlers: {
  [K in keyof EventDataMap]?: EventHandler<K>[];
} = {};

// Добавление обработчика
function addHandler<T extends keyof EventDataMap>(
  event: T,
  handler: (data: EventDataMap[T]) => void
) {
  if (!handlers[event]) {
    handlers[event] = [];
  }
  (handlers[event] as EventHandler<T>[]).push(handler);
}

// Использование
addHandler("user_created", (data) => {
  // data: { userId: number; timestamp: Date }
  console.log(`User ${data.userId} created at ${data.timestamp}`);
});
```

## Ограничения и подводные камни

### 1. Доступ к несуществующим свойствам
```typescript
interface User {
  name: string;
  age: number;
}

// Следующий код вызовет ошибку на этапе компиляции:
// type InvalidType = User["nonexistent"]; // ОШИБКА: "nonexistent" не является ключом User
```

### 2. Доступ к индексам вне диапазона
```typescript
type Tuple = [string, number];

// Для кортежей TypeScript может вернуть union с undefined
type OutOfBounds = Tuple[5]; // undefined (индекс вне диапазона)

// Для массивов все элементы имеют одинаковый тип
type ArrayType = string[number]; // string (для доступа по индексу)
type ArrayElements = string[string["length"]]; // string (неопределенно)
```

### 3. Динамические ключи
```typescript
// Следующее невозможно на этапе компиляции:
// const key = "name";
// type DynamicType = User[key]; // ОШИБКА: переменные не могут использоваться в типах

// Нужно использовать обобщения:
function getTypeByKey<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

## Лучшие практики

### 1. Использование в утилитах
```typescript
// Создание утилиты для получения типа значения
type ValueOf<T> = T[keyof T];

type UserValues = ValueOf<User>; // string | number

// Утилита для получения типов свойств объекта
type PropertyTypes<T> = {
  [K in keyof T]: T[K];
};

type UserProps = PropertyTypes<User>;
// { name: string; age: number; }
```

### 2. Обеспечение безопасности типов
```typescript
// Использование indexed access для обеспечения согласованности
interface ApiEndpoints {
  getUsers: "/api/users";
  getUser: "/api/users/:id";
  createUser: "/api/users";
}

interface ApiResponseTypes {
  getUsers: { users: { id: number; name: string }[] };
  getUser: { user: { id: number; name: string } };
  createUser: { user: { id: number; name: string } };
}

// Типы ответов связаны с эндпоинтами
function callEndpoint<T extends keyof ApiEndpoints>(
  endpoint: T
): Promise<ApiResponseTypes[T]> {
  // реализация
  return Promise.resolve({} as ApiResponseTypes[T]);
}
```

## Совместимость с другими возможностями

### 1. С conditional types
```typescript
// Условные типы с indexed access
type NonNullableProperties<T> = {
  [K in keyof T]: T[K] extends null | undefined ? never : K;
}[keyof T];

interface Example {
  a: string;
  b: number | null;
  c: boolean | undefined;
  d: string;
}

type NonNullKeys = NonNullableProperties<Example>; // "a" | "d"
```

### 2. С mapped types
```typescript
// Сопоставляющие типы с indexed access
type GetReturnType<T> = {
  [K in keyof T]: T[K] extends (...args: any[]) => infer R ? R : never;
};

interface ApiService {
  getUser: () => { id: number; name: string };
  getPosts: () => { id: number; title: string }[];
}

type ServiceReturnTypes = GetReturnType<ApiService>;
// { getUser: { id: number; name: string }; getPosts: { id: number; title: string }[]; }
```

## Связь с другими концепциями
- [[keyof-operator]] - получение ключей для доступа по индексу
- [[../advanced-types/mapped-types]] - сопоставляющие типы, использующие indexed access
- [[../advanced-types/conditional-types]] - условные типы, использующие indexed access
- [[TS Generics]] - обобщения, использующие indexed access
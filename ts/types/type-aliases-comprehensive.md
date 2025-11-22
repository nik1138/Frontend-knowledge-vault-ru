# Псевдонимы типов в TypeScript

Псевдонимы типов (type aliases) - это способ создания нового имени для существующего типа. Они позволяют давать описательные имена сложным типам, упрощая чтение и поддержку кода. Псевдонимы могут представлять примитивные типы, объединения, кортежи и практически любой другой тип.

## Основной синтаксис

Псевдонимы типов создаются с помощью ключевого слова `type`:

```typescript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;

function getName(n: NameOrResolver): Name {
  if (typeof n === "string") {
    return n;
  } else {
    return n();
  }
}
```

## Простые примеры

### 1. Псевдонимы для примитивов

```typescript
type Age = number;
type Email = string;
type ID = string | number;

let userAge: Age = 25;
let userEmail: Email = "user@example.com";
let userId: ID = 123; // или "abc"
```

### 2. Псевдонимы для объединений

```typescript
type Status = "active" | "inactive" | "pending";
type NumericId = number | string;

interface User {
  id: NumericId;
  status: Status;
  name: string;
}
```

### 3. Псевдонимы для кортежей

```typescript
type Coordinates = [number, number];
type KeyValuePair<T, U> = [T, U];

let position: Coordinates = [10, 20];
let entry: KeyValuePair<string, number> = ["count", 42];
```

## Псевдонимы для сложных типов

### 1. Объектные типы

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  isActive: boolean;
};

// Псевдонимы можно использовать вместо интерфейсов для простых объектных типов
const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  isActive: true
};
```

### 2. Функциональные типы

```typescript
type GreetingFunction = (name: string, age: number) => string;

const greet: GreetingFunction = (name, age) => {
  return `Привет, меня зовут ${name}, мне ${age} лет`;
};

// Псевдонимы для функций с коллбэками
type EventHandler<T> = (event: T) => void;
type AsyncOperation<T> = () => Promise<T>;
```

### 3. Обобщенные псевдонимы

```typescript
type Container<T> = { value: T };
type KeyValueCollection<T> = { [key: string]: T };

let stringContainer: Container<string> = { value: "hello" };
let numberContainer: Container<number> = { value: 42 };

let keyValuePairs: KeyValueCollection<number> = {
  first: 1,
  second: 2
};
```

## Практические применения

### 1. Создание типов для API ответов

```typescript
type ApiResponse<T> = {
  data: T;
  status: number;
  message: string;
};

type UserApiResponse = ApiResponse<User>;
type PostApiResponse = ApiResponse<Post[]>;

// Более сложный пример
type PaginatedResponse<T> = {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
  };
};

type UserListResponse = PaginatedResponse<User>;
```

### 2. Работа с конфигурацией

```typescript
type LogLevel = "debug" | "info" | "warn" | "error";
type Environment = "development" | "staging" | "production";

type AppConfig = {
  apiUrl: string;
  logLevel: LogLevel;
  environment: Environment;
  timeout: number;
};

const config: AppConfig = {
  apiUrl: "https://api.example.com",
  logLevel: "info",
  environment: "development",
  timeout: 5000
};
```

### 3. Создание типов для форм

```typescript
type FormField<T> = {
  value: T;
  isValid: boolean;
  error?: string;
};

type TextField = FormField<string>;
type NumberField = FormField<number>;
type BooleanField = FormField<boolean>;

type UserForm = {
  name: TextField;
  age: NumberField;
  subscribed: BooleanField;
};

const form: UserForm = {
  name: { value: "Alice", isValid: true },
  age: { value: 30, isValid: true },
  subscribed: { value: true, isValid: true }
};
```

## Продвинутые паттерны

### 1. Псевдонимы с условными типами

```typescript
type ElementOf<T> = T extends (infer U)[] ? U : T;

type ArrayElement = ElementOf<string[]>;  // string
type NonArrayElement = ElementOf<number>; // number

// Псевдонимы для извлечения типов из промисов
type Unpromisify<T> = T extends Promise<infer U> ? U : T;

type ResolvedType = Unpromisify<Promise<string>>; // string
```

### 2. Псевдонимы для комбинирования типов

```typescript
type WithTimestamp<T> = T & { createdAt: Date; updatedAt: Date };

interface User {
  id: number;
  name: string;
}

type TimestampedUser = WithTimestamp<User>;
// { id: number; name: string; } & { createdAt: Date; updatedAt: Date; }

// Псевдонимы для создания частичных типов
type PartialWithRequired<T, K extends keyof T> = Partial<T> & Required<Pick<T, K>>;

type UserWithRequiredEmail = PartialWithRequired<User, "name">;
// { id?: number; name: string; }
```

### 3. Псевдонимы для работы с DOM

```typescript
type HTMLElementTagName = keyof HTMLElementTagNameMap;

type ElementCreator<T extends HTMLElementTagName> = 
  (tag: T, props?: Partial<HTMLElementTagNameMap[T]>) => HTMLElementTagNameMap[T];

type InputElement = HTMLElementTagNameMap['input'];
type DivElement = HTMLElementTagNameMap['div'];
```

## Практические примеры из реальной разработки

### 1. Создание типобезопасной системы событий

```typescript
type EventName = string & { readonly brand: unique symbol };
type EventHandler<T> = (data: T) => void;

interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  error: { message: string; code: number };
}

type TypedEventEmitter = {
  [K in keyof EventMap as `on${Capitalize<string & K>}`]: (handler: EventHandler<EventMap[K]>) => void
} & {
  emit<K extends keyof EventMap>(event: K, data: EventMap[K]): void;
};

// Более простой пример
type EventHandlers = {
  [K in keyof EventMap]: EventHandler<EventMap[K]>[]
};

const handlers: EventHandlers = {
  user_login: [],
  user_logout: [],
  error: []
};
```

### 2. Работа с настройками и конфигурацией

```typescript
type Theme = "light" | "dark";
type Language = "en" | "ru" | "es" | "fr";

type UserPreferences = {
  theme: Theme;
  language: Language;
  notifications: boolean;
  autoSave: boolean;
};

type SettingsSection = keyof UserPreferences;
type SettingsValue<T extends SettingsSection> = UserPreferences[T];

// Функция для безопасного обновления настроек
function updateSetting<T extends SettingsSection>(
  settings: UserPreferences,
  section: T,
  value: SettingsValue<T>
): void {
  settings[section] = value;
}

const userSettings: UserPreferences = {
  theme: "light",
  language: "en",
  notifications: true,
  autoSave: false
};

updateSetting(userSettings, "theme", "dark");  // OK
// updateSetting(userSettings, "theme", "blue"); // Ошибка
```

### 3. Создание типов для валидации

```typescript
type ValidationRule<T> = (value: T) => boolean;
type ValidationMessage = string;

type FieldValidator<T> = {
  rules: ValidationRule<T>[];
  messages: ValidationMessage[];
};

type StringValidator = FieldValidator<string>;
type NumberValidator = FieldValidator<number>;

const emailValidator: StringValidator = {
  rules: [
    (value) => value.length > 0,
    (value) => value.includes("@"),
    (value) => value.includes(".")
  ],
  messages: [
    "Email не может быть пустым",
    "Email должен содержать @",
    "Email должен содержать ."
  ]
};

type FormValidators<T> = {
  [K in keyof T]: FieldValidator<T[K]>
};

interface UserForm {
  email: string;
  age: number;
  consent: boolean;
}

type UserFormValidators = FormValidators<UserForm>;
```

## Различия между type и interface

### Когда использовать type:

```typescript
// 1. Для примитивных типов и объединений
type Status = "active" | "inactive";
type ID = string | number;

// 2. Для кортежей
type Coordinates = [number, number];

// 3. Для функциональных типов
type Predicate<T> = (value: T) => boolean;

// 4. Для условных типов
type ElementOf<T> = T extends (infer U)[] ? U : T;
```

### Когда использовать interface:

```typescript
// 1. Для объектных типов, которые могут быть расширены
interface User {
  id: number;
  name: string;
}

interface Admin extends User {
  permissions: string[];
}

// 2. Когда нужна возможность объявления нескольких интерфейсов с одним именем (declaration merging)
interface Config {
  apiUrl: string;
}

interface Config {
  timeout: number;
}

// Результат: Config с обоими полями
const config: Config = {
  apiUrl: "https://api.com",
  timeout: 5000
};
```

## Ограничения и особенности

### 1. Объявление нескольких псевдонимов с одним именем

```typescript
// Это невозможно с type - будет ошибка
// type User = { id: number; name: string; };
// type User = { email: string; }; // Ошибка: нельзя повторно объявить

// Но возможно с interface (declaration merging)
interface Config {
  apiUrl: string;
}
interface Config {
  timeout: number;
}
// Config теперь имеет оба свойства
```

### 2. Рекурсивные псевдонимы

```typescript
// Осторожно с рекурсивными псевдонимами
type Category = {
  name: string;
  subcategories: Category[]; // Это работает
};

// Но следующее может вызвать проблемы:
// type Infinite = { value: Infinite }; // Может вызвать ошибку рекурсии
```

### 3. Совместимость с классами

```typescript
class MyClass {
  value: string = "";
}

type MyType = MyClass; // Это работает
const instance: MyType = new MyClass(); // OK

// Но типы не могут быть расширены так же, как интерфейсы
// type MyType = { base: string };
// type ExtendedType = MyType & { additional: number }; // Это работает, но не так, как extends
```

## Лучшие практики

1. **Используйте описательные имена** - псевдонимы должны ясно указывать на их назначение
2. **Документируйте сложные псевдонимы** - особенно когда они используют обобщения или условные типы
3. **Используйте type для примитивов и объединений, interface для объектных типов** - это общепринятая практика
4. **Создавайте переиспользуемые псевдонимы** - для часто используемых комбинаций типов
5. **Избегайте чрезмерно сложных псевдонимов** - это может затруднить понимание кода
6. **Группируйте связанные псевдонимы** - в одном файле или модуле

## Связи с другими концепциями

- [[ts/advanced-types/conditional-types|Условные типы]] - часто используются в сложных псевдонимах
- [[ts/generics/TS Generics|Обобщения]] - псевдонимы часто параметризованы
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - могут использоваться в псевдонимах
- [[ts/types/union-intersection-types|Объединения и пересечения]] - основа для многих псевдонимов
- [[Утилиты типов TypeScript|Утилиты типов]] - многие утилиты реализованы как псевдонимы
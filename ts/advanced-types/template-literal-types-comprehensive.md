# Типы-шаблонные литералы в TypeScript

Типы-шаблонные литералы (template literal types) - это возможность TypeScript для создания новых строковых типов с использованием синтаксиса шаблонных строк. Они позволяют создавать сложные строковые типы на основе других строковых типов, что особенно полезно для работы с API, путями, именами свойств и другими строковыми значениями.

## Основной синтаксис

Типы-шаблонные литералы используют синтаксис, похожий на шаблонные строки в JavaScript, но на уровне типов:

```typescript
type World = "world";
type Greeting = `hello ${World}`;  // "hello world"

// С объединениями строк
type EmailLocaleIDs = "welcome_email" | "email_heading";
type AllLocaleIDs = `${EmailLocaleIDs}_id`;  
// "welcome_email_id" | "email_heading_id"
```

## Простые примеры

### 1. Базовые шаблонные типы

```typescript
type Color = "red" | "blue";
type Quantity = "one" | "two";

type SeussFish = `${Quantity | Capitalize<Quantity>}${Color}`;
// "oneRed" | "oneBlue" | "twoRed" | "twoBlue" | "OneRed" | "OneBlue" | "TwoRed" | "TwoBlue"
```

### 2. Комбинирование строковых литералов

```typescript
type EventName<T extends string> = `${T}Changed`;

type ButtonEvents = EventName<"click" | "hover" | "focus">;
// "clickChanged" | "hoverChanged" | "focusChanged"
```

## Преобразования регистров

TypeScript предоставляет встроенные утилиты для преобразования регистров строковых типов:

### 1. Uppercase, Lowercase, Capitalize, Uncapitalize

```typescript
type Greeting = "Hello, World";

type Upper = Uppercase<Greeting>;      // "HELLO, WORLD"
type Lower = Lowercase<Greeting>;      // "hello, world"
type Cap = Capitalize<Greeting>;       // "Hello, world"
type Uncap = Uncapitalize<Greeting>;   // "hello, World"
```

### 2. Практическое применение преобразований

```typescript
// Создание геттеров и сеттеров
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number; }

type PersonSetters = Setters<Person>;
// { setName: (value: string) => void; setAge: (value: number) => void; }
```

## Практические применения

### 1. Создание имен свойств с префиксами/суффиксами

```typescript
// Добавление префиксов к ключам объекта
type PrefixedProperties<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K]
};

interface User {
  id: number;
  name: string;
}

type PrefixedUser = PrefixedProperties<User, "api">;
// { apiId: number; apiName: string; }
```

### 2. Работа с API путями

```typescript
// Создание типов для REST API путей
type RequestMethod = "GET" | "POST" | "PUT" | "DELETE";

type ApiPath<T extends string> = `/${T}`;

type UserPaths = `${ApiPath<'users'>}/${number}` | `${ApiPath<'users'>}/${number}/${string}`;
// "/users/123" | "/users/123/profile" | "/users/456/settings" | ...

// Комбинация методов и путей
type ApiEndpoint<M extends RequestMethod, P extends string> = `${M} ${ApiPath<P>}`;

type GetUserEndpoint = ApiEndpoint<"GET", "users">;
// "GET /users"
```

### 3. Создание типов для событий

```typescript
// Типизация событий с пространствами имен
type EventWithNamespace<T extends string, N extends string> = `${N}:${T}`;

type MouseEvent = "click" | "hover" | "scroll";
type KeyboardEvent = "keypress" | "keydown" | "keyup";

type UIEvents = EventWithNamespace<MouseEvent, "ui">;
// "ui:click" | "ui:hover" | "ui:scroll"

type InputEvents = EventWithNamespace<KeyboardEvent, "input">;
// "input:keypress" | "input:keydown" | "input:keyup"
```

## Сложные паттерны

### 1. Разбор строк с помощью шаблонных литералов

```typescript
// Извлечение частей строкового типа
type ExtractRoutePath<T> = T extends `/${infer P}/${infer Rest}` ? P : never;

type Route1 = ExtractRoutePath<"/users/123">;  // "users" (но на самом деле извлечет только первую часть)

// Более сложный пример извлечения параметров из URL
type RouteParam<T extends string> = T extends `${string}:${infer Param}${string}` 
  ? Param 
  : never;

type Param1 = RouteParam<"/users/:id">;  // "id"
type Param2 = RouteParam<"/posts/:postId/comments/:commentId">;  // "postId" (первый найденный)
```

### 2. Рекурсивные шаблонные типы

```typescript
// Преобразование подчеркивания в camelCase
type SnakeToCamel<T extends string> = 
  T extends `${infer Word}_${infer Rest}`
    ? `${Word}${Capitalize<SnakeToCamel<Rest>>}`
    : T;

type CamelCaseProp = SnakeToCamel<"user_name_and_email">;
// "userNameAndEmail"
```

### 3. Комбинирование с сопоставленными типами

```typescript
// Создание событий для каждого свойства объекта
type PropertyEvents<T> = {
  [K in keyof T as 
    `${string & K}Changed` | `${string & K}Validated` | `${string & K}Reset`
  ]: (value: T[K]) => void
};

type UserEvents = PropertyEvents<User>;
// {
//   idChanged: (value: number) => void;
//   idValidated: (value: number) => void;
//   idReset: (value: number) => void;
//   nameChanged: (value: string) => void;
//   nameValidated: (value: string) => void;
//   nameReset: (value: string) => void;
// }
```

## Практические примеры из реальной разработки

### 1. Типизация конфигурации с префиксами переменных окружения

```typescript
interface AppConfig {
  apiUrl: string;
  dbHost: string;
  dbName: string;
  port: number;
}

// Создание типа для переменных окружения
type EnvVarNames<T> = {
  [K in keyof T as Uppercase<`${string & K}`>]: string
};

// На практике, это будет выглядеть как:
// {
//   APIURL: string;
//   DBHOST: string;
//   DBNAME: string;
//   PORT: string;  // переменные окружения обычно строковые
// }
```

### 2. Создание типов для CSS классов

```typescript
type BemElement<Block extends string, Element extends string> = `${Block}__${Element}`;
type BemModifier<Block extends string, Modifier extends string> = `${Block}--${Modifier}`;
type BemElementModifier<Block extends string, Element extends string, Modifier extends string> = 
  `${Block}__${Element}--${Modifier}`;

type ButtonBlock = "button";
type ButtonElements = "text" | "icon" | "loader";
type ButtonModifiers = "primary" | "secondary" | "disabled";

type ButtonElementClasses = BemElement<ButtonBlock, ButtonElements>;
// "button__text" | "button__icon" | "button__loader"

type ButtonModifierClasses = BemModifier<ButtonBlock, ButtonModifiers>;
// "button--primary" | "button--secondary" | "button--disabled"
```

### 3. Работа с именованными конфигурациями

```typescript
// Создание типов для настроек с иерархией
type ConfigPath<Prefix extends string, Suffix extends string> = `${Prefix}.${Suffix}`;

type ServerConfig = {
  host: string;
  port: number;
};

type DatabaseConfig = {
  url: string;
  timeout: number;
};

type AppConfiguration = {
  server: ServerConfig;
  database: DatabaseConfig;
};

// Создание плоского типа с иерархическими ключами
type FlattenConfig<T, Prefix extends string = ""> = {
  [K in keyof T]: T[K] extends object
    ? T[K] extends any[] | Date | RegExp
      ? `${Prefix extends "" ? "" : `${Prefix}.`}${K & string}`
      : FlattenConfig<T[K], `${Prefix extends "" ? "" : `${Prefix}.`}${K & string}`>
    : `${Prefix extends "" ? "" : `${Prefix}.`}${K & string}`
}[keyof T];

// Это создаст объединение всех возможных путей конфигурации
type ConfigKeys = FlattenConfig<AppConfiguration>;
// "server.host" | "server.port" | "database.url" | "database.timeout"
```

## Ограничения и особенности

### 1. Ограничения на сложность

Сложные шаблонные типы могут быть ограничены компилятором по сложности:

```typescript
// Слишком сложные рекурсивные шаблонные типы могут вызвать ошибки компиляции
// type TooComplex<T> = /* сложная рекурсивная структура */; // может вызвать ошибку
```

### 2. Совместимость с более ранними версиями

Шаблонные типы доступны начиная с TypeScript 4.1, поэтому они не будут работать в более ранних версиях.

### 3. Ограничения на infer в шаблонных типах

```typescript
// Вывод типов в шаблонных литералах может быть ограничен
type ParseRoute<T extends string> = 
  T extends `/${infer Suffix}` 
    ? Suffix 
    : never;

type Parsed = ParseRoute<"/users/123/details">;  // "users/123/details"
```

## Практические рекомендации

### 1. Использование для API и конфигураций

Шаблонные типы особенно полезны для типизации API и конфигурационных структур:

```typescript
// Типизация API эндпоинтов
type ApiEndpoints = {
  getUsers: "/api/users";
  getUser: `/api/users/${number}`;
  createUser: "/api/users";
  updateUser: `/api/users/${number}`;
  deleteUser: `/api/users/${number}`;
};
```

### 2. Создание типобезопасных DSL

```typescript
// Создание типобезопасного языка для описания путей
type PathBuilder<Current extends string = ""> = {
  segment<S extends string>(segment: S): PathBuilder<`${Current}/${S}`>;
  param<P extends string>(param: P): PathBuilder<`${Current}/:${P}`>;
  build(): `${Current extends "" ? "/" : Current}`;
};

// Использование:
// const path = new PathBuilder().segment("users").param("id").segment("posts").build();
// Тип результата: "/users/:id/posts"
```

## Лучшие практики

1. **Используйте для создания понятных имен** - шаблонные типы хороши для создания понятных строковых типов
2. **Избегайте излишней сложности** - чрезмерно сложные шаблонные типы могут затруднить понимание кода
3. **Тестируйте с разными входными типами** - убедитесь, что типы работают корректно с разными значениями
4. **Документируйте сложные шаблонные типы** - добавляйте комментарии к сложным конструкциям
5. **Учитывайте производительность** - сложные шаблонные типы могут замедлять компиляцию
6. **Проверяйте совместимость версий** - убедитесь, что используете поддерживаемые возможности TypeScript

## Связи с другими концепциями

- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - часто используются совместно с шаблонными типами
- [[ts/utility-types/built-in-string-utilities|Строковые утилиты]] - Uppercase, Lowercase, Capitalize, Uncapitalize
- [[ts/advanced-types/conditional-types|Условные типы]] - могут использоваться внутри шаблонных типов
- [[ts/type-system/keyof-operator|Оператор keyof]] - для получения строковых ключей
- [[ts/generics/TS Generics|Обобщения]] - шаблонные типы часто параметризованы
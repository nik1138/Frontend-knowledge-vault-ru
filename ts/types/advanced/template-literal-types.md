# Template Literal Types

Шаблонные литеральные типы (Template Literal Types) позволяют создавать строковые типы, объединяя строковые литералы с типами через синтаксис, похожий на шаблонные строки JavaScript. Этот функционал доступен начиная с TypeScript 4.1.

## Основной синтаксис

### Простые шаблонные литералы
```typescript
// Простое объединение литералов
type World = "world";
type Greeting = `hello ${World}`; // "hello world"

// Использование с несколькими вставками
type FirstName = "Jane";
type LastName = "Smith";
type FullName = `${FirstName} ${LastName}`; // "Jane Smith"

// С числовыми литералами
type Version = `v${number}`;
type APIVersion = Version; // string (пока конкретное число неизвестно)

const v1: Version = "v1.2.3"; // OK
const v2: Version = "v42";    // OK
// const v3: Version = "1.0";  // Ошибка: не начинается с 'v'

// С объединениями
type Color = "red" | "blue";
type Size = "small" | "large";
type ClassName = `${Color}-${Size}`; // "red-small" | "red-large" | "blue-small" | "blue-large"
```

### Комбинирование с литералами
```typescript
// Создание имен для CSS классов
type ButtonType = "primary" | "secondary" | "danger";
type ButtonSize = "small" | "medium" | "large";

type ButtonClass = `btn-${ButtonType}` | `btn-${ButtonSize}` | `btn-${ButtonType}-${ButtonSize}`;
// Результат: 
// "btn-primary" | "btn-secondary" | "btn-danger" |
// "btn-small" | "btn-medium" | "btn-large" |
// "btn-primary-small" | "btn-primary-medium" | "btn-primary-large" |
// и т.д. для всех комбинаций

// Имена для API endpoints
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Resource = "users" | "posts" | "comments";
type Endpoint = `${Lowercase<HttpMethod>}/${Resource}`;
// "get/users" | "get/posts" | "get/comments" | ...
// "post/users" | "post/posts" | "post/comments" | ...
// и т.д.
```

## Комбинация с преобразованиями регистров

### Использование встроенных утилит
```typescript
// Capitalize - делает первую букву заглавной
type T0 = Capitalize<"hello">;  // "Hello"

// Uncapitalize - делает первую букву строчной  
type T1 = Uncapitalize<"Hello">;  // "hello"

// Uppercase - все буквы заглавные
type T2 = Uppercase<"hello">;  // "HELLO"

// Lowercase - все буквы строчные
type T3 = Lowercase<"HELLO">;  // "hello"

// Практическое применение
type EventName = "click" | "hover" | "focus";
type EventHandler = `${EventName}Handler`;
// "clickHandler" | "hoverHandler" | "focusHandler"

type Command = "start" | "stop" | "restart";
type ActionMethod = `on${Capitalize<Command>}`;
// "onStart" | "onStop" | "onRestart"
```

## Сопоставляющие типы с шаблонными литералами

### Добавление префиксов и суффиксов
```typescript
// Добавление префиксов к свойствам объекта
type AddPrefix<T> = {
  [K in keyof T as `new_${K & string}`]: T[K]
};

interface User {
  id: number;
  name: string;
  email: string;
}

type PrefixedUser = AddPrefix<User>;
// { new_id: number; new_name: string; new_email: string; }

// Добавление суффиксов
type AddSuffix<T> = {
  [K in keyof T as `${K & string}_computed`]: () => T[K]
};

type ComputedUser = AddSuffix<User>;
// { id_computed: () => number; name_computed: () => string; email_computed: () => string; }
```

### Преобразование методов
```typescript
// Создание методов геттеров
type Getter<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface Config {
  apiEndpoint: string;
  timeout: number;
  retries: number;
}

type ConfigGetters = Getter<Config>;
// {
//   getApiEndpoint: () => string;
//   getTimeout: () => number; 
//   getRetries: () => number;
// }
```

## Продвинутые паттерны

### Разбор и извлечение типов
```typescript
// Извлечение типа из строки
type ParseType<T extends string> = T extends `${infer Name}:${infer Type}` 
  ? { name: Name; type: Type } 
  : never;

type Parsed1 = ParseType<"user:name">;    // { name: "user"; type: "name" }
type Parsed2 = ParseType<"post:id">;      // { name: "post"; type: "id" }
type Parsed3 = ParseType<"invalid">;      // never

// Разбор путей
type ParsePath<T extends string> = T extends `${infer Head}/${infer Tail}`
  ? [Head, ...ParsePath<Tail>]
  : T extends `${infer Last}`
    ? [Last]
    : [];

type Path1 = ParsePath<"users/123/posts">;  // ["users", "123", "posts"]
type Path2 = ParsePath<"root">;             // ["root"]
type Path3 = ParsePath<"a/b/c/d">;          // ["a", "b", "c", "d"]
```

### Условные шаблонные литералы
```typescript
// Условное форматирование строкового типа
type FormatKey<T extends string> = T extends `${infer Prefix}_${infer Suffix}`
  ? `${Lowercase<Prefix>}_${Capitalize<Suffix>}`
  : Capitalize<T>;

type Key1 = FormatKey<"user_id">;     // "user_Id" 
type Key2 = FormatKey<"first_name">;  // "first_Name"
type Key3 = FormatKey<"simple">;      // "Simple"

// Сложное условное преобразование
type NormalizeKey<T extends string> = 
  T extends `${infer First}_${infer Rest}`
    ? `${Lowercase<First>}_${NormalizeKey<Rest>}`
    : T extends `${infer First}${infer Rest}`
      ? `${Lowercase<First>}${NormalizeKey<Rest>}`
      : T;

type Normalized = NormalizeKey<"USER_NAME">;  // "user_name"
```

## Практические примеры

### Событийная система
```typescript
// Типизация системы событий
type EventType = "user" | "system" | "network";
type Action = "login" | "logout" | "connect" | "disconnect";

type EventName = `${EventType}:${Action}`;
// "user:login" | "user:logout" | "system:login" | "system:logout" | 
// "network:connect" | "network:disconnect" | ...

interface EventBus {
  emit<T extends EventName>(event: T, data?: any): void;
  on<T extends EventName>(event: T, handler: (data?: any) => void): void;
}

// Использование
const bus: EventBus = {
  emit: (event, data) => console.log(`Emitting ${event}`, data),
  on: (event, handler) => console.log(`Subscribed to ${event}`)
};

bus.emit("user:login", { userId: 123 });     // OK
bus.emit("system:connect");                  // OK
// bus.emit("invalid:event");               // Ошибка: "invalid:event" не входит в EventName
```

### Типизированные конфигурации
```typescript
// Система конфигурации с типизированными ключами
interface ConfigSchema {
  api: {
    url: string;
    timeout: number;
  };
  ui: {
    theme: "light" | "dark";
    animations: boolean;
  };
}

type ConfigKey<T extends string = any> = 
  T extends `${infer TopLevel}.${infer Rest}`
    ? TopLevel extends keyof ConfigSchema
      ? `${TopLevel}.${ConfigKey<Rest>}`
      : never
    : T extends keyof ConfigSchema
      ? T
      : never;

// Примеры
type ValidKey1 = ConfigKey<"api.url">;  // "api.url"
type ValidKey2 = ConfigKey<"ui.theme">; // "ui.theme"
type ValidKey3 = ConfigKey<"invalid">;  // never
```

### Система валидации
```typescript
// Типизированная система валидации с сообщениями об ошибках
type ValidationRule = "required" | "minLength" | "maxLength" | "pattern";
type FieldName = "username" | "email" | "password";

type ValidationErrorPath = `${FieldName}.${ValidationRule}`;
// "username.required" | "username.minLength" | "username.maxLength" | ...
// "email.required" | "email.minLength" | "email.maxLength" | ...
// и т.д.

interface ValidationErrors {
  [key: ValidationErrorPath]: string;
}

const errors: ValidationErrors = {
  "username.required": "Username is required",
  "email.pattern": "Invalid email format",
  "password.minLength": "Password too short"
};
```

## Продвинутые утилиты

### Deep renaming
```typescript
// Глубокое переименование ключей объекта
type DeepRename<T, Prefix extends string> = T extends object
  ? {
      [K in keyof T as K extends string 
        ? T[K] extends object | object[] 
          ? `${Prefix}_${K & string}` 
          : `${Prefix}_${K & string}` 
        : never
      ]: T[K] extends object[]
        ? DeepRename<T[K][number], `${Prefix}_${K & string}`>[]
        : T[K] extends object
          ? DeepRename<T[K], `${Prefix}_${K & string}`>
          : T[K]
    }
  : T;

interface User {
  id: number;
  profile: {
    name: string;
    contacts: {
      email: string;
      phone: string;
    }[];
  };
}

type RenamedUser = DeepRename<User, "api">;
// Преобразует в структуру с префиксами для всех ключей
```

### Fluent interface
```typescript
// Создание fluent interface для объектов
type FluentBuilder<T> = {
  [K in keyof T as `with${Capitalize<K & string>}`]: (value: T[K]) => FluentBuilder<T>;
} & {
  build(): T;
};

interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
  ssl: boolean;
}

// Это создаст тип с методами:
// .withHost(), .withPort(), .withUsername(), .withPassword(), .withSsl(), .build()
// каждый из которых возвращает новый FluentBuilder
```

## Ограничения и лучшие практики

### Ограничения
```typescript
// Осторожное использование рекурсии
// Следующий тип может привести к переполнению стека компиляции
type BadRecursion<T extends string> = T extends `${infer Head}${infer Tail}`
  ? `${Head}${BadRecursion<Tail>}`  // потенциально бесконечная рекурсия
  : T;

// Избегайте чрезмерно сложных объединений
type TooManyCombinations = `${'a' | 'b' | 'c' | 'd'}${'1' | '2' | '3' | '4'}${'x' | 'y' | 'z'}`;
// 4 * 4 * 3 = 48 комбинаций, что может повлиять на производительность
```

### Лучшие практики
1. **Используйте шаблонные литералы для ограниченных строковых типов** - особенно в enum-подобных сценариях
2. **Комбинируйте с сопоставляющими типами** - для мощных трансформаций объектов
3. **Документируйте сложные шаблонные типы** - особенно с условными выражениями
4. **Тестируйте производительность** - особенно при большом количестве комбинаций
5. **Используйте для создания безопасных API** - особенно для названий событий, путей и т.д.

## Практическое применение в реальных сценариях

### API маршруты
```typescript
// Типизированные API маршруты
type Version = "v1" | "v2";
type Resource = "users" | "posts" | "comments";
type Action = "get" | "create" | "update" | "delete";

type ApiRoute = `/${Version}/${Resource}/${Action}`;
// "/v1/users/get" | "/v1/users/create" | ...
// "/v2/comments/delete" | ...

interface RouteHandler {
  [route: ApiRoute]: (req: any, res: any) => void;
}

// Использование
const handlers: RouteHandler = {
  "/v1/users/get": (req, res) => { /* реализация */ },
  "/v1/users/create": (req, res) => { /* реализация */ },
  // "/v1/users/invalid": () => {}  // Ошибка: не входит в ApiRoute
};
```

### CSS классы
```typescript
// Генерация CSS классов с модификаторами
type BEMBlock = "button" | "card" | "form";
type BEMModifier = "primary" | "secondary" | "disabled" | "large";

type BEMClass = `${BEMBlock}--${BEMModifier}` | BEMBlock;
// "button", "button--primary", "button--disabled", ...
// "card", "card--primary", "card--secondary", ...
// и т.д.

function cn(...classes: (BEMClass | false | null | undefined)[]): string {
  return classes.filter(Boolean).join(' ');
}

// Использование
const buttonClass = cn("button", "button--primary", false && "button--disabled");
```

## Связь с другими концепциями
- [[../advanced-types/mapped-types]] - комбинация с сопоставляющими типами
- [[../advanced-types/conditional-types]] - часто используется с условными типами  
- [[literal-types]] - расширение концепции литеральных типов
- [[../generics/index]] - часто используется с обобщениями
- [[../type-system/keyof-operator]] - часто используется для получения ключей
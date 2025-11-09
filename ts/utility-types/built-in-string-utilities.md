# Встроенные утилиты для работы со строками

TypeScript 4.1+ предоставляет утилиты для преобразования строковых типов-литералов, что особенно полезно при работе с шаблонными литералами.

## Uppercase<S>

Преобразует все символы строки в верхний регистр.

```typescript
type Uppercase<S extends string> = intrinsic;

type Greeting = Uppercase<"hello, world">; // "HELLO, WORLD"

type Status = "pending" | "loading" | "success" | "error";
type UpperStatus = Uppercase<Status>;
// "PENDING" | "LOADING" | "SUCCESS" | "ERROR"

// Практическое применение
type Environment = "development" | "production" | "staging";
type EnvVariables = {
  [K in Uppercase<Environment>]: string;
};
// {
//   DEVELOPMENT: string;
//   PRODUCTION: string;
//   STAGING: string;
// }

// Для констант
const CONFIG_KEYS = {
  API_URL: "apiUrl",
  DB_HOST: "dbHost",
  JWT_SECRET: "jwtSecret"
} as const;

type ConfigKeys = Uppercase<keyof typeof CONFIG_KEYS>;
// "API_URL" | "DB_HOST" | "JWT_SECRET"
```

## Lowercase<S>

Преобразует все символы строки в нижний регистр.

```typescript
type Lowercase<S extends string> = intrinsic;

type Greeting = Lowercase<"HELLO, WORLD">; // "hello, world"

type Status = "PENDING" | "LOADING" | "SUCCESS" | "ERROR";
type LowerStatus = Lowercase<Status>;
// "pending" | "loading" | "success" | "error"

// Практическое применение
type HttpHeaders = {
  [K in Lowercase<string & keyof IncomingHttpHeaders>]: string;
};

// Для CSS классов
type ButtonVariant = "PRIMARY" | "SECONDARY" | "DANGER";
type CssClasses = {
  [K in `btn-${Lowercase<ButtonVariant>}`]: string;
};
// {
//   "btn-primary": string;
//   "btn-secondary": string;
//   "btn-danger": string;
// }
```

## Capitalize<S>

Преобразует первый символ строки в верхний регистр.

```typescript
type Capitalize<S extends string> = intrinsic;

type Greeting = Capitalize<"hello, world">; // "Hello, world"

type EventTypes = "click" | "focus" | "blur";
type EventHandlerNames = `on${Capitalize<EventTypes>}`;
// "onClick" | "onFocus" | "onBlur"

// Практическое применение
type EventHandlers = {
  [K in EventHandlerNames]: (event: Event) => void;
};
// {
//   onClick: (event: Event) => void;
//   onFocus: (event: Event) => void;
//   onBlur: (event: Event) => void;
// }

// Для React компонентов
type ComponentNames = "button" | "input" | "select";
type ComponentTypes = {
  [K in Capitalize<ComponentNames>]: React.ComponentType<any>;
};
```

## Uncapitalize<S>

Преобразует первый символ строки в нижний регистр.

```typescript
type Uncapitalize<S extends string> = intrinsic;

type Greeting = Uncapitalize<"Hello, World">; // "hello, World"

type ComponentNames = "Button" | "Input" | "Select";
type PropNames = `${Uncapitalize<ComponentNames>}Props`;
// "buttonProps" | "inputProps" | "selectProps"

// Практическое применение
interface ComponentProps {
  buttonProps: ButtonProps;
  inputProps: InputProps;
  selectProps: SelectProps;
}

type GetProps<T extends ComponentNames> = 
  ComponentProps[`${Uncapitalize<T>}Props`];

// Для именованных экспорта/импорта
type ModuleNames = "UserModule" | "ProductModule" | "OrderModule";
type FileNames = `${Uncapitalize<ModuleNames>}.ts`;
// "userModule.ts" | "productModule.ts" | "orderModule.ts"
```

## Комбинирование утилит

### 1. Создание консистентных имен

```typescript
type ApiEndpoints = "getUser" | "createUser" | "updateUser" | "deleteUser";

type ApiActions = {
  [K in ApiEndpoints as Uppercase<K>]: () => Promise<any>;
};
// {
//   GETUSER: () => Promise<any>;
//   CREATEUSER: () => Promise<any>;
//   UPDATEUSER: () => Promise<any>;
//   DELETEUSER: () => Promise<any>;
// }

type ApiConstants = {
  [K in ApiEndpoints as `API_${Uppercase<K>}`]: string;
};
// {
//   API_GETUSER: string;
//   API_CREATEUSER: string;
//   API_UPDATEUSER: string;
//   API_DELETEUSER: string;
// }
```

### 2. Преобразование для разных форматов

```typescript
type PropertyNames = "firstName" | "lastName" | "dateOfBirth";

// Для CSS custom properties
type CssVars = {
  [K in PropertyNames as `--${K}`]: string;
};
// {
//   "--firstName": string;
//   "--lastName": string;
//   "--dateOfBirth": string;
// }

// Для data-атрибутов
type DataAttributes = {
  [K in PropertyNames as `data-${K}`]: string;
};
// {
//   "data-firstName": string;
//   "data-lastName": string;
//   "data-dateOfBirth": string;
// }

// Для kebab-case
type KebabCase<S extends string> = S extends `${infer T}${infer U}`
  ? U extends Uncapitalize<U>
    ? `${Lowercase<T>}${KebabCase<U>}`
    : `${Lowercase<T>}-${KebabCase<U>}`
  : S;

type KebabPropertyNames = {
  [K in PropertyNames as KebabCase<K>]: string;
};
// {
//   "first-name": string;
//   "last-name": string;
//   "date-of-birth": string;
// }
```

### 3. Генерация типов для API

```typescript
type HttpMethods = "GET" | "POST" | "PUT" | "DELETE";
type Resources = "users" | "products" | "orders";

type ApiEndpoints = `${Lowercase<HttpMethods>} /api/${Resources}`;
// "get /api/users" | "get /api/products" | "get /api/orders" |
// "post /api/users" | "post /api/products" | "post /api/orders" |
// "put /api/users" | "put /api/products" | "put /api/orders" |
// "delete /api/users" | "delete /api/products" | "delete /api/orders"

type ApiClient = {
  [K in ApiEndpoints]: K extends `${infer Method} /api/${infer Resource}`
    ? Method extends "get"
      ? () => Promise<Array<{ id: string }>>
      : (data: any) => Promise<{ id: string }>
    : never;
};
```

## Пользовательские утилиты

### 1. TitleCase

```typescript
type TitleCase<S extends string> = S extends `${infer First}${infer Rest}`
  ? `${Capitalize<First>}${Rest}`
  : S;

type Title = TitleCase<"hello world">; // "Hello world"
```

### 2. SnakeCase

```typescript
type SnakeCase<S extends string> = S extends `${infer T}${infer U}`
  ? U extends Uncapitalize<U>
    ? `${Lowercase<T>}${SnakeCase<U>}`
    : `${Lowercase<T>}_${SnakeCase<U>}`
  : S;

type SnakePropertyNames = {
  [K in PropertyNames as SnakeCase<K>]: string;
};
// {
//   "first_name": string;
//   "last_name": string;
//   "date_of_birth": string;
// }
```

## Ограничения и особенности

1. Строковые утилиты работают только с типами-литералами
2. Могут замедлять компиляцию при сложных объединениях
3. Ограничения на длину результирующих строк
4. Не поддерживают сложные операции со строками (только базовые преобразования)

## Связанные концепции

- [[ts/advanced-types/template-literal-types|Типы-шаблонные литералы]]
- [[ts/utility-types/built-in-object-transformations|Утилиты для преобразования объектов]]
- [[ts/type-system/string-literal-types|Строковые типы-литералы]]
- [[ts/generics/TS Generics|Дженерики]]
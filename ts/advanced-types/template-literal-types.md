# Типы-шаблонные литералы (Template Literal Types)

Типы-шаблонные литералы в TypeScript позволяют создавать строковые типы-литералы с использованием синтаксиса шаблонных строк JavaScript. Это мощный инструмент для создания сложных строковых типов на основе других типов.

## Основной синтаксис

Типы-шаблонные литералы используют тот же синтаксис, что и шаблонные строки в JavaScript:

```typescript
type Greeting = `Hello, ${string}!`;
const greet: Greeting = "Hello, World!"; // ✅ Допустимо
const greet2: Greeting = "Hello, 123!";  // ✅ Допустимо
```

## Простые примеры

### 1. Объединение строковых литералов

```typescript
type World = "world";
type Greeting = `hello ${World}`; // "hello world"
```

### 2. Работа с объединениями

```typescript
type Color = "red" | "blue";
type Quantity = "one" | "two";

type SeussFish = `${Quantity | Color} fish`;
// "one fish" | "two fish" | "red fish" | "blue fish"
```

## Практическое применение

### 1. Создание ключей объектов

```typescript
type PropEventSource<T> = {
  on(
    eventName: `${string & keyof T}Changed`,
    callback: (newValue: any) => void
  ): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;

const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", () => {}); // ✅ Допустимо
person.on("lastNameChanged", () => {});  // ✅ Допустимо
person.on("ageChanged", () => {});       // ✅ Допустимо
person.on("heightChanged", () => {});    // ❌ Ошибка
```

### 2. Преобразование регистров

```typescript
type cases = '' | 'rest'
type Kelvin = 'kelvin'
type celsius = 'celsius'
type fahrenheit = 'fahrenheit'

type unit = `${cases}${Kelvin | celsius | fahrenheit}`
// "" | "kelvin" | "celsius" | "fahrenheit" | "rest"
```

## Встроенные утилиты для работы с регистрами

TypeScript предоставляет встроенные типы для преобразования регистров:

### 1. Uppercase<StringType>

Преобразует все символы в верхний регистр:

```typescript
type Greeting = Uppercase<"Hello, world!">; // "HELLO, WORLD!"

type SiteURL = Uppercase<"www.example.com">; // "WWW.EXAMPLE.COM"
```

### 2. Lowercase<StringType>

Преобразует все символы в нижний регистр:

```typescript
type Greeting = Lowercase<"Hello, WORLD!">; // "hello, world!"

type DBField = Lowercase<"FIRST_NAME" | "LAST_NAME">; // "first_name" | "last_name"
```

### 3. Capitalize<StringType>

Преобразует первый символ в верхний регистр:

```typescript
type LowercaseGreeting = "hello, world!";
type Greeting = Capitalize<LowercaseGreeting>; // "Hello, world!"
```

### 4. Uncapitalize<StringType>

Преобразует первый символ в нижний регистр:

```typescript
type UppercaseGreeting = "HELLO, WORLD!";
type Greeting = Uncapitalize<UppercaseGreeting>; // "hELLO, WORLD!"
```

## Продвинутые примеры

### 1. Создание типов для CSS-классов

```typescript
type Size = "small" | "medium" | "large";
type Color = "primary" | "secondary" | "danger";

type ButtonClass = `btn-${Size}-${Color}`;
// "btn-small-primary" | "btn-small-secondary" | "btn-small-danger" | 
// "btn-medium-primary" | "btn-medium-secondary" | "btn-medium-danger" | 
// "btn-large-primary" | "btn-large-secondary" | "btn-large-danger"
```

### 2. Генерация типов для API эндпоинтов

```typescript
type Method = "GET" | "POST" | "PUT" | "DELETE";
type Resource = "users" | "posts" | "comments";

type Endpoint = `${Lowercase<Method>} /api/${Resource}`;
// "get /api/users" | "get /api/posts" | "get /api/comments" |
// "post /api/users" | "post /api/posts" | "post /api/comments" |
// "put /api/users" | "put /api/posts" | "put /api/comments" |
// "delete /api/users" | "delete /api/posts" | "delete /api/comments"
```

### 3. Создание типов для событий

```typescript
type EventType = "click" | "focus" | "blur";
type Element = "button" | "input" | "div";

type EventHandlerName = `on${Capitalize<EventType>}${Capitalize<Element>}`;
// "onClickButton" | "onClickInput" | "onClickDiv" |
// "onFocusButton" | "onFocusInput" | "onFocusDiv" |
// "onBlurButton" | "onBlurInput" | "onBlurDiv"
```

## Работа с числовыми и булевыми типами

### 1. Преобразование чисел в строки

```typescript
type NumberToString<T extends number> = `${T}`;

type Age = NumberToString<25>; // "25"
type Year = NumberToString<2023>; // "2023"
```

### 2. Создание типов для диапазонов

```typescript
type Percentage = `${number}%`;

const width: Percentage = "50%"; // ✅ Допустимо
const height: Percentage = "100%"; // ✅ Допустимо
const invalid: Percentage = "50px"; // ❌ Ошибка
```

## Условные типы с шаблонными литералами

Можно использовать шаблонные литералы в условных типах для извлечения частей строк:

```typescript
type GetFileName<T extends string> = 
  T extends `${infer FileName}.${infer Extension}` 
    ? FileName 
    : T;

type File1 = GetFileName<"app.ts">; // "app"
type File2 = GetFileName<"index.html">; // "index"
type File3 = GetFileName<"readme">; // "readme"
```

## Практические примеры использования

### 1. Создание типов для URL-параметров

```typescript
type RouteParam<T extends string> = `:${T}`;

type UserIdParam = RouteParam<"userId">; // ":userId"
type PostIdParam = RouteParam<"postId">; // ":postId"
```

### 2. Генерация типов для стилей

```typescript
type CSSProperty = "color" | "background" | "margin";
type CSSValue = "red" | "blue" | "10px" | "20px";

type CSSRule = `${CSSProperty}: ${CSSValue};`;
// "color: red;" | "color: blue;" | "color: 10px;" | "color: 20px;" |
// "background: red;" | "background: blue;" | "background: 10px;" | "background: 20px;" |
// "margin: red;" | "margin: blue;" | "margin: 10px;" | "margin: 20px;"
```

### 3. Создание типов для сообщений об ошибках

```typescript
type ErrorCode = "NOT_FOUND" | "UNAUTHORIZED" | "FORBIDDEN";
type ErrorMessage<T extends ErrorCode> = `Error ${T}: ${string}`;

type NotFoundError = ErrorMessage<"NOT_FOUND">;
// "Error NOT_FOUND: ${string}"
```

## Ограничения и особенности

1. Шаблонные литералы могут замедлять компиляцию при сложных объединениях
2. Сложные шаблоны могут быть трудны для понимания
3. Ограничения на длину результирующих строк (влияют на производительность)
4. Не поддерживают сложные операции со строками (только базовые преобразования)

## Связанные концепции

- [[ts/type-system/string-literal-types|Строковые типы-литералы]]
- [[ts/advanced-types/conditional-types|Условные типы]]
- [[ts/type-system/type-inference|Вывод типов]]
- [[ts/generics/TS Generics|Дженерики]]
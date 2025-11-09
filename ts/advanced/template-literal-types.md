# Шаблонные литеральные типы в TypeScript

## Введение

Шаблонные литеральные типы (Template Literal Types) - это мощная возможность TypeScript, появившаяся в версии 4.1. Они позволяют создавать новые строковые типы на основе других строковых типов с использованием синтаксиса шаблонных строк JavaScript. Это позволяет создавать выразительные и гибкие типы, особенно полезные для работы с именами свойств, путями, префиксами и другими строковыми паттернами.

## Основы шаблонных литеральных типов

Шаблонные литеральные типы используют синтаксис шаблонных строк, но вместо значений переменных используют типы:

```typescript
// Простой пример шаблонного литерального типа
type World = "world";
type Greeting = `hello ${World}`; // "hello world"

// Шаблонный литеральный тип с объединением
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`; 
// "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

## Продвинутое использование

### Префиксы и суффиксы

Шаблонные литеральные типы часто используются для добавления префиксов или суффиксов к существующим строковым типам:

```typescript
// Добавление префикса к строке
type PropEventName<T> = `on${Capitalize<string & keyof T>}`;

interface MyEvent {
  mouseup: MouseEvent;
  mousedown: MouseEvent;
  keyup: KeyboardEvent;
  keydown: KeyboardEvent;
}

// Создание типа для обработчиков событий
type MyEventHandlers = {
  [K in keyof MyEvent as PropEventName<MyEvent>]: (event: MyEvent[K]) => void;
};
// Результат: {
//   onMouseup: (event: MouseEvent) => void;
//   onMousedown: (event: MouseEvent) => void;
//   onKeyup: (event: KeyboardEvent) => void;
//   onKeydown: (event: KeyboardEvent) => void;
// }
```

### Преобразование регистров

TypeScript предоставляет встроенные утилиты для преобразования регистра строк:

```typescript
// Утилиты для преобразования регистра
type T0 = Capitalize<"hello">;  // "Hello"
type T1 = Uncapitalize<"Hello">; // "hello"
type T2 = Uppercase<"hello">;   // "HELLO"
type T3 = Lowercase<"HELLO">;   // "hello"

// Практический пример
type FieldName<T> = {
  [K in keyof T as `${Lowercase<string & K>}Field`]: T[K]
};

interface User {
  FirstName: string;
  LastName: string;
  Age: number;
}

type NormalizedUser = FieldName<User>;
// {
//   firstnamefield: string;
//   lastnamefield: string;
//   agefield: number;
// }
```

## Практические примеры

### Создание имен свойств на основе других свойств

```typescript
// Создание типа для "dirty" флагов на основе существующего интерфейса
interface UserForm {
  name: string;
  email: string;
  age: number;
}

type DirtyFields<T> = {
  [K in keyof T as `is${Capitalize<string & K>}Dirty`]: boolean;
};

type UserFormDirty = DirtyFields<UserForm>;
// {
//   isNameDirty: boolean;
//   isEmailDirty: boolean;
//   isAgeDirty: boolean;
// }
```

### Генерация типов для API эндпоинтов

```typescript
// Создание типов для REST API эндпоинтов
type HttpMethod = 'get' | 'post' | 'put' | 'delete';
type Endpoint = 'users' | 'posts' | 'comments';

type ApiEndpoint = `${HttpMethod}_${Endpoint}`;
// "get_users" | "post_users" | "put_users" | "delete_users" | 
// "get_posts" | "post_posts" | "put_posts" | "delete_posts" | 
// "get_comments" | "post_comments" | "put_comments" | "delete_comments"

// Создание типа для API вызовов
type ApiCall<T extends ApiEndpoint> = {
  endpoint: T;
  data?: any;
  headers?: Record<string, string>;
};

const getUserCall: ApiCall<'get_users'> = {
  endpoint: 'get_users',
  headers: { 'Content-Type': 'application/json' }
};
```

### Работа с путями к свойствам

```typescript
// Создание типа для представления путей к свойствам
type PropPath<T, Prefix extends string = ''> = 
  T extends Record<string, any>
    ? { [K in keyof T]: K extends string
        ? T[K] extends Record<string, any>
          ? `${Prefix}${K}` | `${Prefix}${K}.${PropPath<T[K], ''>}`
          : `${Prefix}${K}`
        : never
      }[keyof T]
    : never;

interface User {
  id: number;
  profile: {
    name: string;
    contact: {
      email: string;
      phone: string;
    };
  };
  preferences: {
    theme: string;
    notifications: boolean;
  };
}

type UserPaths = PropPath<User>;
// "id" | "profile" | "profile.name" | "profile.contact" | 
// "profile.contact.email" | "profile.contact.phone" | 
// "preferences" | "preferences.theme" | "preferences.notifications"
```

## Продвинутые паттерны

### Утилиты с шаблонными литеральными типами

```typescript
// Утилита для добавления префикса к ключам объекта
type AddPrefix<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K]
};

interface Config {
  apiUrl: string;
  timeout: number;
  retries: number;
}

type PrefixedConfig = AddPrefix<Config, 'app_'>;
// {
//   app_ApiUrl: string;
//   app_Timeout: number;
//   app_Retries: number;
// }
```

### Извлечение значений из шаблонных строк

```typescript
// Утилита для извлечения части строки из шаблонного типа
type ExtractRouteParam<T extends string> = 
  T extends `${string}:${infer Param}${string}` ? Param : never;

type Route1 = "/users/:id";
type Route2 = "/posts/:postId/comments/:commentId";

type Param1 = ExtractRouteParam<Route1>; // "id"
type Param2 = ExtractRouteParam<Route2>; // "postId" (первый найденный параметр)

// Более сложная утилита для извлечения всех параметров
type ExtractRouteParams<T extends string> = 
  T extends `${infer _Start}:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<`/${Rest}`>
    : T extends `${infer _Start}:${infer Param}`
      ? Param
      : never;

type AllParams = ExtractRouteParams<"/users/:userId/posts/:postId">; // "userId" | "postId"
```

### Условные типы с шаблонными литералами

```typescript
// Утилита для преобразования свойств с определенным паттерном
type TransformEventHandlers<T> = {
  [K in keyof T as K extends `on${infer Event}` 
    ? `handle${Event}` 
    : K]: T[K]
};

interface ComponentEvents {
  onClick: () => void;
  onMouseEnter: () => void;
  onSubmit: () => void;
  className: string; // не будет преобразовано
}

type TransformedEvents = TransformEventHandlers<ComponentEvents>;
// {
//   handleClick: () => void;
//   handleMouseEnter: () => void;
//   handleSubmit: () => void;
//   className: string;
// }
```

## Связь с другими концепциями

- [[utility-types/index|Утилиты типов]] - как шаблонные литеральные типы интегрируются с другими утилитами
- [[advanced-types/conditional-types|Условные типы]] - часто используются вместе с шаблонными литералами
- [[advanced-types/mapped-types|Сопоставляющие типы]] - особенно полезны с шаблонными литералами
- [[types/type-aliases-comprehensive|Псевдонимы типов]] - для создания более сложных типов

## Лучшие практики

1. **Используйте шаблонные литеральные типы для создания выразительных API** - особенно полезны для именования свойств, эндпоинтов, событий
2. **Будьте осторожны с производительностью** - сложные шаблонные типы могут замедлить компиляцию
3. **Комбинируйте с сопоставляющими типами** - для создания мощных преобразований объектов
4. **Документируйте сложные шаблонные типы** - они могут быть трудны для понимания другими разработчиками

## Заключение

Шаблонные литеральные типы - мощный инструмент для создания гибких и выразительных типов в TypeScript. Они позволяют создавать сложные преобразования строковых типов, что особенно полезно при работе с API, событиями, путями к свойствам и другими сценариями, где строковые значения имеют структуру.
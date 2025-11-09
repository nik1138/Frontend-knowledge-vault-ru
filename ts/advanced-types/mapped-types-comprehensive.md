# Сопоставленные типы в TypeScript

Сопоставленные типы (mapped types) - это возможность TypeScript для создания новых типов путем перебора и преобразования свойств существующих типов. Они позволяют создавать новые типы, "сопоставляя" каждый член исходного типа с новым членом, потенциально применяя преобразования.

## Основной синтаксис

Сопоставленные типы используют синтаксис `{ [P in K]: T }`, где:
- `P` - переменная типа, представляющая каждый ключ
- `K` - объединение ключей (обычно получаемое через `keyof`)
- `T` - тип значения, который может ссылаться на `P`

```typescript
// Простейший сопоставленный тип
type SimpleMapped<T> = { [P in keyof T]: T[P] };

interface Person {
  name: string;
  age: number;
}

type CopyOfPerson = SimpleMapped<Person>;
// Эквивалентно: { name: string; age: number; }
```

## Простые примеры сопоставленных типов

### 1. Создание необязательных свойств

```typescript
type Partial<T> = { [P in keyof T]?: T[P] };

interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; }
```

### 2. Создание свойств только для чтения

```typescript
type Readonly<T> = { readonly [P in keyof T]: T[P] };

type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string; readonly email: string; }
```

### 3. Выбор определенных свойств

```typescript
type Pick<T, K extends keyof T> = { [P in K]: T[P] };

type UserIdAndName = Pick<User, "id" | "name">;
// { id: number; name: string; }
```

## Продвинутые преобразования

### 1. Изменение модификаторов доступа

```typescript
// Удаление модификатора readonly
type Mutable<T> = { -readonly [P in keyof T]: T[P] };

interface ReadOnlyPoint {
  readonly x: number;
  readonly y: number;
}

type MutablePoint = Mutable<ReadOnlyPoint>;
// { x: number; y: number; } - теперь можно изменять
```

### 2. Удаление опциональности

```typescript
// Делает все свойства обязательными
type Concrete<T> = { [P in keyof T]-?: T[P] };

interface PartialUser {
  id?: number;
  name?: string;
  email?: string;
}

type RequiredUser = Concrete<PartialUser>;
// { id: number; name: string; email: string; } - все обязательные
```

### 3. Преобразование ключей

```typescript
// Использование ключей с преобразованием (требует TypeScript 4.1+)
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number; }
```

## Практические применения

### 1. Создание типов для API

```typescript
// Создание DTO для обновления с опциональными полями
type UpdateDto<T> = Partial<T>;

type UserUpdateDto = UpdateDto<User>;
// { id?: number; name?: string; email?: string; }

// Создание DTO для создания с обязательными полями, кроме ID
type CreateDto<T> = Omit<Required<T>, "id">;

type UserCreateDto = CreateDto<User>;
// { name: string; email: string; } - без id, но все остальные обязательны
```

### 2. Глубокие преобразования

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
    address: {
      street: string;
      city: string;
    };
  };
}

type DeepReadonlyUser = DeepReadonly<NestedUser>;
// Все вложенные объекты тоже будут readonly
```

### 3. Преобразование свойств на основе условий

```typescript
// Создание типа, где числовые поля становятся строками
type NumericToString<T> = {
  [K in keyof T]: T[K] extends number ? string : T[K]
};

interface Product {
  id: number;
  name: string;
  price: number;
  isActive: boolean;
}

type ProductWithStringNumbers = NumericToString<Product>;
// { id: string; name: string; price: string; isActive: boolean; }
```

## Сопоставленные типы с преобразованием ключей

Начиная с TypeScript 4.1, можно преобразовывать ключи с помощью конструкции `as`:

### 1. Изменение регистра ключей

```typescript
// Преобразование ключей в верхний регистр
type UpperKeys<T> = {
  [K in keyof T as Uppercase<string & K>]: T[K]
};

interface User {
  userId: number;
  userName: string;
}

type UpperUser = UpperKeys<User>;
// { USERID: number; USERNAME: string; }
```

### 2. Фильтрация ключей

```typescript
// Оставляем только строковые поля
type StringOnly<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K]
};

type StringUser = StringOnly<User>;
// { userName: string; } - только строковые поля
```

### 3. Создание префиксов/суффиксов

```typescript
// Добавление префикса к ключам
type Prefixed<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K]
};

type PrefixedUser = Prefixed<User, "api">;
// { apiUserId: number; apiUserName: string; }
```

## Комбинация с условными типами

Сопоставленные типы часто комбинируются с условными типами для создания мощных преобразований:

```typescript
// Создание типа, где функции оборачиваются в промис
type Asyncify<T> = {
  [K in keyof T]: T[K] extends (...args: infer A) => infer R
    ? (...args: A) => Promise<R>
    : T[K]
};

interface Api {
  getUser: (id: number) => User;
  validate: () => boolean;
  message: string;
}

type AsyncApi = Asyncify<Api>;
// {
//   getUser: (id: number) => Promise<User>;
//   validate: () => Promise<boolean>;
//   message: string;  // не функция, остается без изменений
// }
```

## Встроенные сопоставленные типы

TypeScript предоставляет несколько встроенных сопоставленных типов:

```typescript
// Partial<T> - делает все свойства опциональными
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type OptionalTodo = Partial<Todo>;
// { title?: string; description?: string; completed?: boolean; }

// Required<T> - делает все свойства обязательными
type RequiredTodo = Required<OptionalTodo>;
// { title: string; description: string; completed: boolean; }

// Readonly<T> - делает все свойства readonly
type ImmutableTodo = Readonly<Todo>;
// { readonly title: string; readonly description: string; readonly completed: boolean; }

// Pick<T, K> - выбирает подмножество свойств
type TodoPreview = Pick<Todo, "title" | "completed">;
// { title: string; completed: boolean; }

// Record<K, T> - создает тип с ключами K и значениями T
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>;
// { prop1: string; prop2: string; prop3: string; }
```

## Продвинутые паттерны

### 1. Условные сопоставленные типы

```typescript
// Преобразование только функций в асинхронные
type AsyncFunctionsOnly<T> = {
  [K in keyof T]: T[K] extends Function ? 
    T[K] extends (...args: infer A) => infer R ?
      (...args: A) => Promise<Awaited<R>> :
      T[K] :
    T[K]
};
```

### 2. Рекурсивные сопоставленные типы

```typescript
// Глубокое преобразование в опциональные поля
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object 
    ? T[P] extends Function 
      ? T[P] 
      : DeepPartial<T[P]> 
    : T[P]
};

type DeepPartialUser = DeepPartial<NestedUser>;
// Все поля, включая вложенные, становятся опциональными
```

### 3. Сопоставленные типы с фильтрацией

```typescript
// Тип, который включает только определенные типы свойств
type FilterProps<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};

type StringPropsOnly = FilterProps<User, string>;
// { userName: string; } - только строковые свойства
```

## Практические примеры из реальной разработки

### 1. Создание типов для форм

```typescript
interface UserForm {
  name: string;
  email: string;
  age: number;
  consent: boolean;
}

// Тип для ошибок валидации формы
type FormErrors<T> = { [K in keyof T]?: string[] };

type UserFormErrors = FormErrors<UserForm>;
// { name?: string[]; email?: string[]; age?: string[]; consent?: string[]; }

// Тип для значений формы с возможностью null
type FormValues<T> = { [K in keyof T]: T[K] | null };

type UserFormValues = FormValues<UserForm>;
// { name: string | null; email: string | null; age: number | null; consent: boolean | null; }
```

### 2. Работа с API ответами

```typescript
// Преобразование API ответа в тип с опциональными полями
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

type OptionalApiResponse<T> = {
  [K in keyof ApiResponse<T>]: K extends 'data' 
    ? T | null 
    : ApiResponse<T>[K]
};

type OptionalUserResponse = OptionalApiResponse<User>;
// { data: User | null; status: number; message: string; }
```

### 3. Создание вспомогательных типов

```typescript
// Тип для определения, какие поля являются необязательными
type OptionalPropertyNames<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never
}[keyof T];

type OptionalUserFields = OptionalPropertyNames<User>;
// Тип, содержащий имена необязательных полей (если бы они были)
```

## Ограничения и лучшие практики

### 1. Производительность

Сложные сопоставленные типы могут замедлять компиляцию:

```typescript
// Избегайте чрезмерно сложных вложенных сопоставлений
type Complex<T> = {
  [K in keyof T as T[K] extends object ? 
    T[K] extends Function ? 
      never : 
      `nested${Capitalize<string & K>}` : 
    K]: T[K] extends object ? 
      T[K] extends Function ? 
        T[K] : 
        DeepPartial<T[K]> : 
      T[K]
};
```

### 2. Читаемость

Разбивайте сложные сопоставленные типы на несколько шагов:

```typescript
// Вместо одного сложного типа
type BetterApproach<T> = TransformFunctions<TransformNested<T>>;

type TransformNested<T> = {
  [K in keyof T]: T[K] extends object ? DeepPartial<T[K]> : T[K]
};

type TransformFunctions<T> = {
  [K in keyof T]: T[K] extends Function ? string : T[K]
};
```

### 3. Ограничения на рекурсию

Будьте осторожны с рекурсивными сопоставленными типами, чтобы избежать бесконечной рекурсии:

```typescript
// Это может вызвать проблемы:
// type BadRecursive<T> = { [K in keyof T]: BadRecursive<T[K]> };
```

## Лучшие практики

1. **Начинайте с простых сопоставленных типов** - постепенно переходите к сложным
2. **Используйте именованные типы** - давайте понятные имена для сопоставленных типов
3. **Документируйте сложные типы** - добавляйте комментарии к сложным сопоставленным типам
4. **Тестируйте с разными входными типами** - убедитесь, что типы работают корректно
5. **Избегайте излишней сложности** - сложные сопоставленные типы могут затруднить понимание кода
6. **Разбивайте на подтипы** - при необходимости разбивайте сложные преобразования на части

## Связи с другими концепциями

- [[ts/advanced-types/conditional-types|Условные типы]] - часто используются совместно с сопоставленными типами
- [[ts/utility-types/index|Утилиты типов]] - многие утилиты реализованы через сопоставленные типы
- [[ts/type-system/keyof-operator|Оператор keyof]] - основа для получения ключей типов
- [[ts/advanced-types/template-literal-types|Типы-шаблонные литералы]] - используются для преобразования ключей
- [[ts/generics/TS Generics|Обобщения]] - сопоставленные типы часто параметризованы
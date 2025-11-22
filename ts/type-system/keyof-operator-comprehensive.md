# Оператор keyof в TypeScript

Оператор `keyof` - это один из фундаментальных операторов системы типов TypeScript, который позволяет получать объединение всех публичных ключей (свойств) типа. Он возвращает строковый литеральный тип, представляющий все возможные ключи объекта, что делает его мощным инструментом для создания типобезопасного кода.

## Основной синтаксис

Оператор `keyof` принимает тип и возвращает объединение строковых литералов всех его ключей:

```typescript
interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonKeys = keyof Person;  // "name" | "age" | "email"

// Это эквивалентно:
// type PersonKeys = "name" | "age" | "email"
```

## Простые примеры

### 1. Базовое использование

```typescript
const person: Person = {
  name: "Alice",
  age: 30,
  email: "alice@example.com"
};

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Использование функции
const name = getProperty(person, "name");    // string
const age = getProperty(person, "age");      // number
const email = getProperty(person, "email");  // string

// Следующая строка вызовет ошибку компиляции:
// const invalid = getProperty(person, "invalid");  // Ошибка: "invalid" не является ключом Person
```

### 2. Использование с встроенными типами

```typescript
type ArrayKeys = keyof string[];     // "length" | "toString" | "pop" | "push" | ...
type ObjectKeys = keyof object;      // "constructor" | "toString" | "hasOwnProperty" | ...
type FunctionKeys = keyof Function;  // "apply" | "bind" | "call" | ...
```

## Продвинутые паттерны

### 1. Использование с дженериками

```typescript
// Функция для получения значений по нескольким ключам
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  keys.forEach(key => {
    result[key] = obj[key];
  });
  return result;
}

const user = { id: 1, name: "John", email: "john@example.com", role: "admin" };
const nameAndEmail = pick(user, ["name", "email"]);
// { name: string; email: string; }
```

### 2. Создание типов на основе существующих

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

// Тип, содержащий только ключи с типом string
type StringKeyOf<T> = {
  [K in keyof T]: T[K] extends string ? K : never
}[keyof T];

type StringUserKeys = StringKeyOf<User>;  // "name" | "email"
```

### 3. Использование с сопоставленными типами

```typescript
// Создание типа с опциональными свойствами только для чтения
type OptionalReadOnly<T> = {
  +readonly [P in keyof T]?: T[P]
};

type OptionalReadOnlyUser = OptionalReadOnly<User>;
// {
//   readonly id?: number;
//   readonly name?: string;
//   readonly email?: string;
//   readonly createdAt?: Date;
// }
```

## Практические применения

### 1. Создание утилит типов

```typescript
// Утилита для создания типа с обязательными полями по ключам
type RequiredBy<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

interface UserProfile {
  id: number;
  name?: string;
  email?: string;
  avatar?: string;
}

type RequiredProfile = RequiredBy<UserProfile, "name" | "email">;
// {
//   id: number;
//   name: string;        // теперь обязательное
//   email: string;       // теперь обязательное
//   avatar?: string;     // остается опциональным
// }
```

### 2. Работа с объектами конфигурации

```typescript
interface ApiConfig {
  baseUrl: string;
  timeout: number;
  retries: number;
  headers: Record<string, string>;
}

// Тип для обновления конфигурации
type UpdateConfig = Partial<Pick<ApiConfig, keyof ApiConfig>>;

// Или более конкретно:
type ConfigUpdateKeys = keyof ApiConfig;  // "baseUrl" | "timeout" | "retries" | "headers"

function updateConfig<T extends ConfigUpdateKeys>(
  config: ApiConfig,
  key: T,
  value: ApiConfig[T]
): void {
  config[key] = value;
}
```

### 3. Создание безопасных обновлений объектов

```typescript
// Тип для безопасного обновления объекта
type Updater<T> = {
  [K in keyof T]: (value: T[K]) => T & { [P in K]: T[K] }
};

interface State {
  count: number;
  name: string;
  active: boolean;
}

// Создание обновляторов для каждого поля
type StateUpdaters = {
  [K in keyof State]: (state: State, value: State[K]) => State
};

const updaters: StateUpdaters = {
  count: (state, value) => ({ ...state, count: value }),
  name: (state, value) => ({ ...state, name: value }),
  active: (state, value) => ({ ...state, active: value })
};
```

## Связь с другими операторами

### 1. Комбинация с typeof

```typescript
const formData = {
  username: "john_doe",
  email: "john@example.com",
  age: 25
};

type FormDataKeys = keyof typeof formData;  // "username" | "email" | "age"

// Это позволяет создавать типобезопасные формы
type FormField = keyof typeof formData;
```

### 2. Использование с индексными типами

```typescript
// Получение типа значения по ключу
type ValueOf<T> = T[keyof T];

type UserValues = ValueOf<User>;  // number | string | Date
```

## Ограничения и особенности

### 1. Приватные и защищенные свойства

Оператор `keyof` включает только публичные свойства:

```typescript
class MyClass {
  public publicProp: string = "";
  private privateProp: number = 0;
  protected protectedProp: boolean = true;
}

type MyClassKeys = keyof MyClass;  // "publicProp" (только публичное свойство)
```

### 2. Индексные сигнатуры

```typescript
interface IndexSignature {
  [key: string]: number;
  id: string;
}

type IndexKeys = keyof IndexSignature;  // string | "id"
// Объединение строкового типа (из индексной сигнатуры) и конкретного ключа
```

### 3. Числовые ключи

```typescript
interface NumericKeys {
  0: string;
  1: number;
  name: boolean;
}

type NumericKeyTypes = keyof NumericKeys;  // 0 | 1 | "name"
// TypeScript сохраняет числовые ключи как числовые типы
```

## Практические примеры из реальной разработки

### 1. Создание типобезопасного доступа к свойствам

```typescript
// Функция для получения вложенного свойства объекта
type NestedKeyOf<T> = T extends object
  ? { [K in keyof T]: K extends string 
      ? T[K] extends object 
        ? `${K}` | `${K}.${NestedKeyOf<T[K]>}` 
        : `${K}`
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
}

type UserNestedKeys = NestedKeyOf<User>;
// "id" | "profile" | "profile.name" | "profile.contact" | "profile.contact.email" | "profile.contact.phone"

function getNestedValue<T, K extends NestedKeyOf<T>>(
  obj: T,
  path: K
): K extends `${infer A}.${infer B}` 
  ? A extends keyof T 
    ? B extends NestedKeyOf<T[A]> 
      ? T[A][B] 
      : unknown 
    : unknown 
  : K extends keyof T 
    ? T[K] 
    : unknown {
  
  const keys = path.split('.') as (keyof T)[];
  let result: any = obj;
  
  for (const key of keys) {
    result = result[key];
  }
  
  return result;
}

const user: User = {
  id: 1,
  profile: {
    name: "John",
    contact: {
      email: "john@example.com",
      phone: "+1234567890"
    }
  }
};

// Типобезопасный доступ к вложенным свойствам
const email = getNestedValue(user, "profile.contact.email");  // string
const name = getNestedValue(user, "profile.name");            // string
```

### 2. Работа с маппингами и словарями

```typescript
// Создание типа для словаря с фиксированными ключами
type Dictionary<T> = {
  [K in keyof T]: T[K]
};

interface HttpStatus {
  OK: 200;
  NOT_FOUND: 404;
  SERVER_ERROR: 500;
}

type StatusCodes = keyof HttpStatus;  // "OK" | "NOT_FOUND" | "SERVER_ERROR"
type StatusValues = HttpStatus[StatusCodes];  // 200 | 404 | 500

const statusMessages: { [K in StatusCodes]: string } = {
  OK: "Success",
  NOT_FOUND: "Not Found",
  SERVER_ERROR: "Internal Server Error"
};
```

### 3. Создание типобезопасных событий

```typescript
interface EventMap {
  user_login: { userId: number; timestamp: Date };
  user_logout: { userId: number; duration: number };
  error: { message: string; code: number };
}

type EventNames = keyof EventMap;  // "user_login" | "user_logout" | "error"

class TypedEventEmitter {
  private listeners: {
    [K in EventNames]?: ((data: EventMap[K]) => void)[];
  } = {};

  on<K extends EventNames>(event: K, listener: (data: EventMap[K]) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }

  emit<K extends EventNames>(event: K, data: EventMap[K]): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(listener => listener(data));
    }
  }
}
```

## Лучшие практики

1. **Используйте keyof для создания типобезопасных функций доступа** - это предотвращает опечатки в именах свойств
2. **Комбинируйте с дженериками** - для создания гибких, переиспользуемых типов
3. **Документируйте сложные конструкции** - особенно когда используете keyof с другими операторами
4. **Тестируйте с разными типами** - убедитесь, что ваши типы работают с различными структурами
5. **Используйте для создания утилит типов** - keyof является основой для многих полезных утилит

## Ограничения производительности

Сложные конструкции с `keyof`, особенно рекурсивные или с большими объектами, могут замедлить компиляцию:

```typescript
// Избегайте чрезмерно сложных конструкций
// type VeryComplex = /* сложная рекурсивная структура с keyof */;
```

## Связи с другими концепциями

- [[ts/advanced-types/indexed-access-types|Индексные типы доступа]] - часто используются совместно с keyof
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - база для многих паттернов с keyof
- [[ts/type-system/type-inference|Вывод типов]] - keyof влияет на вывод типов
- [[ts/generics/TS Generics|Обобщения]] - keyof часто используется в дженериках
- [[Утилиты типов TypeScript|Утилиты типов]] - многие утилиты используют keyof
# Другие встроенные утилиты TypeScript

TypeScript предоставляет дополнительные встроенные утилиты, которые решают специфические задачи типизации.

## Record<K, T>

Создает тип с набором свойств `K` типа `T`.

```typescript
type Record<K extends keyof any, T> = {
  [P in K]: T;
};

// Простой пример
type Scores = Record<string, number>;
// { [key: string]: number }

const scores: Scores = {
  "John": 95,
  "Jane": 87,
  "Bob": 92
};

// С конкретными ключами
type UserRoles = Record<"admin" | "user" | "guest", string>;
// {
//   admin: string;
//   user: string;
//   guest: string;
// }

// Практическое применение
interface User {
  id: string;
  name: string;
}

type UserDictionary = Record<string, User>;
// { [id: string]: User }

function createUsersDictionary(users: User[]): UserDictionary {
  return users.reduce((dict, user) => {
    dict[user.id] = user;
    return dict;
  }, {} as UserDictionary);
}

// Для конфигураций
type Config = Record<string, string | number | boolean>;

const appConfig: Config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  debug: true
};

// Для маппинга значений
type ColorPalette = Record<"primary" | "secondary" | "accent", string>;

const colors: ColorPalette = {
  primary: "#007bff",
  secondary: "#6c757d",
  accent: "#28a745"
};
```

## NonNullable<T>

Исключает `null` и `undefined` из типа `T`.

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type NullableString = string | null | undefined;
type StrictString = NonNullable<NullableString>; // string

// Практическое применение
function processUser(user: User | null) {
  if (user !== null) {
    // TypeScript знает, что user теперь NonNullable<User>
    const strictUser: NonNullable<typeof user> = user;
    console.log(strictUser.name);
  }
}

// С массивами
type NullableUsers = (User | null)[];
type StrictUsers = NonNullable<NullableUsers>[number][]; // User[]

// Для API ответов
interface ApiResponse<T> {
  data: T | null;
  error: string | null;
}

type StrictApiResponse<T> = {
  data: NonNullable<ApiResponse<T>["data"]>;
  error: NonNullable<ApiResponse<T>["error"]>;
};
```

## ThisType<T>

Манипулирует типом `this` в объектных литералах и методах.

```typescript
type ThisType<T> = {};

// Пример использования
interface Helpers {
  log(message: string): void;
}

interface Data {
  value: string;
}

const obj: Data & ThisType<Data & Helpers> = {
  value: "test",
  log(message) {
    // this имеет тип Data & Helpers
    console.log(`${this.value}: ${message}`);
  }
};

// Практическое применение
interface ComponentMethods {
  setState(newState: Partial<this>): void;
  render(): string;
}

interface ComponentState {
  name: string;
  count: number;
}

type Component<T extends ComponentState> = T & ThisType<T & ComponentMethods>;

const counter: Component<ComponentState> = {
  name: "Counter",
  count: 0,
  setState(newState) {
    // this имеет тип ComponentState & ComponentMethods
    Object.assign(this, newState);
  },
  render() {
    return `${this.name}: ${this.count}`;
  }
};

// Для DSL (Domain Specific Languages)
interface CalculatorMethods {
  add(x: number): this;
  multiply(x: number): this;
  result(): number;
}

type Calculator = ThisType<CalculatorMethods>;

const calculator: Calculator = {
  add(x) {
    // this имеет тип CalculatorMethods
    return this;
  },
  multiply(x) {
    return this;
  },
  result() {
    return 42; // имитация вычислений
  }
};
```

## Awaited<T>

Получает тип, который будет доступен после ожидания Promise.

```typescript
type Awaited<T> = T extends null | undefined 
  ? T 
  : T extends object & { then(onfulfilled: infer F): any } 
    ? F extends (value: infer V, ...args: any) => any 
      ? Awaited<V> 
      : never 
    : T;

// Простой пример
type AsyncString = Promise<string>;
type UnwrappedString = Awaited<AsyncString>; // string

// Вложенные Promise
type NestedPromise = Promise<Promise<string>>;
type UnwrappedNested = Awaited<NestedPromise>; // string

// Практическое применение
async function fetchUserData(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

type UserData = Awaited<ReturnType<typeof fetchUserData>>; // User

// Для утилит работы с асинхронными функциями
type AsyncFunction = (...args: any[]) => Promise<any>;
type AsyncFunctionResult<T extends AsyncFunction> = 
  Awaited<ReturnType<T>>;

function createAsyncCache<T extends AsyncFunction>() {
  const cache = new Map<string, AsyncFunctionResult<T>>();
  
  return {
    get: (key: string): AsyncFunctionResult<T> | undefined => cache.get(key),
    set: (key: string, value: AsyncFunctionResult<T>): void => {
      cache.set(key, value);
    }
  };
}

// Для обработки результатов Promise.all
type PromisesTuple = [Promise<string>, Promise<number>, Promise<boolean>];
type ResolvedTuple = { [K in keyof PromisesTuple]: Awaited<PromisesTuple[K]> };
// [string, number, boolean]
```

## PartialRecord<K, T>

Пользовательская утилита для создания записи с опциональными свойствами.

```typescript
type PartialRecord<K extends keyof any, T> = {
  [P in K]?: T;
};

// Альтернатива Record<K, T> с опциональными свойствами
type OptionalConfig = PartialRecord<"debug" | "timeout" | "retries", number>;

const config: OptionalConfig = {
  debug: 1,
  // timeout и retries опциональны
};
```

## RequiredRecord<K, T>

Пользовательская утилита для создания записи с обязательными свойствами.

```typescript
type RequiredRecord<K extends keyof any, T> = {
  [P in K]: T;
};

// Гарантирует, что все свойства присутствуют
type FullConfig = RequiredRecord<"apiUrl" | "apiKey" | "timeout", string>;

const config: FullConfig = {
  apiUrl: "https://api.example.com",
  apiKey: "secret123",
  timeout: "5000" // Все свойства обязательны
};
```

## Nullable<T>

Пользовательская утилита для добавления null и undefined к типу.

```typescript
type Nullable<T> = T | null | undefined;

// Обратная операция к NonNullable<T>
type MaybeString = Nullable<string>; // string | null | undefined

// Практическое применение
interface ApiResponse<T> {
  data: Nullable<T>;
  error: Nullable<string>;
}

function handleResponse<T>(response: ApiResponse<T>) {
  if (response.data !== null && response.data !== undefined) {
    // TypeScript знает, что response.data имеет тип T
    console.log(response.data);
  }
  
  if (response.error) {
    console.error(response.error);
  }
}
```

## Ограничения и особенности

1. `ThisType<T>` работает только в объектных литералах и требует включения `noImplicitThis`
2. `Awaited<T>` доступен начиная с TypeScript 4.5
3. Некоторые утилиты могут быть сложны для понимания без практического применения
4. При использовании с большими объединениями могут замедлять компиляцию

## Связанные концепции

- [[ts/utility-types/built-in-object-transformations|Утилиты для преобразования объектов]]
- [[ts/utility-types/built-in-function-utilities|Утилиты для работы с функциями]]
- [[ts/utility-types/built-in-string-utilities|Утилиты для работы со строками]]
- [[ts/type-system/type-inference|Вывод типов]]
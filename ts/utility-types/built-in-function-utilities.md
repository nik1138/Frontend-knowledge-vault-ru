# Встроенные утилиты для работы с функциями

TypeScript предоставляет утилиты, которые позволяют извлекать информацию о функциях и манипулировать ими на уровне типов.

## ReturnType<T>

Получает тип возвращаемого значения функции `T`.

```typescript
type ReturnType<T extends (...args: any) => any> = 
  T extends (...args: any) => infer R ? R : any;

function getUser() {
  return {
    name: "John",
    age: 30,
    email: "john@example.com"
  };
}

type User = ReturnType<typeof getUser>;
// {
//   name: string;
//   age: number;
//   email: string;
// }

// Практическое применение
function createAsyncAction<T extends (...args: any[]) => any>(
  action: T
): (...args: Parameters<T>) => Promise<ReturnType<T>> {
  return async (...args: Parameters<T>): Promise<ReturnType<T>> => {
    // Выполняем асинхронную операцию
    return action(...args);
  };
}

// Для API клиентов
interface ApiService {
  getUser: () => Promise<User>;
  updateUser: (user: User) => Promise<User>;
}

type ApiResponses = {
  [K in keyof ApiService]: ReturnType<ApiService[K]> extends Promise<infer R> ? R : never;
};
// {
//   getUser: User;
//   updateUser: User;
// }
```

## Parameters<T>

Получает типы параметров функции `T`.

```typescript
type Parameters<T extends (...args: any) => any> = 
  T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number) {
  return `Hello, ${name}! You are ${age} years old.`;
}

type GreetParams = Parameters<typeof greet>;
// [string, number]

// Практическое применение
function createLogger<T extends (...args: any[]) => any>(
  fn: T
): (...args: Parameters<T>) => ReturnType<T> {
  return (...args: Parameters<T>): ReturnType<T> => {
    console.log(`Calling ${fn.name} with args:`, args);
    const result = fn(...args);
    console.log(`Result:`, result);
    return result;
  };
}

// Для оберток над API
interface ApiClient {
  fetchUser: (id: string) => Promise<User>;
  createUser: (user: Omit<User, "id">) => Promise<User>;
}

type ApiParams = {
  [K in keyof ApiClient]: Parameters<ApiClient[K]>;
};
// {
//   fetchUser: [string];
//   createUser: [Omit<User, "id">];
// }
```

## ConstructorParameters<T>

Получает типы параметров конструктора класса `T`.

```typescript
type ConstructorParameters<T extends abstract new (...args: any) => any> = 
  T extends abstract new (...args: infer P) => any ? P : never;

class User {
  constructor(
    public name: string,
    public age: number,
    public email?: string
  ) {}
}

type UserConstructorParams = ConstructorParameters<typeof User>;
// [string, number, (string | undefined)?]

// Практическое применение
function createFactory<T extends new (...args: any[]) => any>(
  Class: T
): (...args: ConstructorParameters<T>) => InstanceType<T> {
  return (...args: ConstructorParameters<T>): InstanceType<T> => {
    return new Class(...args);
  };
}

const userFactory = createFactory(User);
const user = userFactory("John", 30, "john@example.com");
```

## InstanceType<T>

Получает тип экземпляра класса `T`.

```typescript
type InstanceType<T extends abstract new (...args: any) => any> = 
  T extends abstract new (...args: any) => infer R ? R : any;

class User {
  constructor(
    public name: string,
    public age: number
  ) {}
}

type UserInstance = InstanceType<typeof User>;
// User

// Практическое применение
function createService<T extends new (...args: any[]) => any>(
  ServiceClass: T,
  ...args: ConstructorParameters<T>
): InstanceType<T> {
  return new ServiceClass(...args);
}

// Для dependency injection
interface Services {
  userService: typeof UserService;
  productService: typeof ProductService;
}

type ServiceInstances = {
  [K in keyof Services]: InstanceType<Services[K]>;
};
```

## ThisParameterType<T>

Получает тип `this` из сигнатуры функции `T`.

```typescript
type ThisParameterType<T> = 
  T extends (this: infer U, ...args: any[]) => any ? U : unknown;

interface User {
  name: string;
  age: number;
}

function greet(this: User) {
  return `Hello, I'm ${this.name} and I'm ${this.age} years old.`;
}

type UserContext = ThisParameterType<typeof greet>;
// User

// Практическое применение
function bindContext<T extends (this: any, ...args: any[]) => any>(
  fn: T,
  context: ThisParameterType<T>
): (...args: Parameters<T>) => ReturnType<T> {
  return (...args: Parameters<T>): ReturnType<T> => {
    return fn.apply(context, args);
  };
}
```

## OmitThisParameter<T>

Удаляет параметр `this` из сигнатуры функции `T`.

```typescript
type OmitThisParameter<T> = 
  unknown extends ThisParameterType<T> 
    ? T 
    : T extends (...args: infer A) => infer R 
      ? (...args: A) => R 
      : T;

function greet(this: { name: string }, message: string) {
  return `${message}, ${this.name}!`;
}

type GreetWithoutThis = OmitThisParameter<typeof greet>;
// (message: string) => string

// Практическое применение
function createStandaloneFunction<T extends (this: any, ...args: any[]) => any>(
  method: T
): OmitThisParameter<T> {
  return ((...args: any[]) => method(...args)) as OmitThisParameter<T>;
}
```

## Parameter<T, Index>

Пользовательская утилита для получения типа конкретного параметра.

```typescript
type Parameter<T extends (...args: any[]) => any, Index extends number> = 
  Parameters<T>[Index];

function processUser(
  user: User,
  options: { validate?: boolean },
  callback: (result: boolean) => void
) {
  // ...
}

type FirstParam = Parameter<typeof processUser, 0>; // User
type SecondParam = Parameter<typeof processUser, 1>; // { validate?: boolean }
type ThirdParam = Parameter<typeof processUser, 2>; // (result: boolean) => void
```

## AsyncReturnType<T>

Пользовательская утилита для получения типа возвращаемого значения асинхронной функции.

```typescript
type AsyncReturnType<T extends (...args: any) => Promise<any>> = 
  T extends (...args: any) => Promise<infer R> ? R : never;

async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`);
  return response.json() as Promise<User>;
}

type FetchedUser = AsyncReturnType<typeof fetchUser>;
// User

// Практическое применение
function createCache<T extends (...args: any[]) => Promise<any>>() {
  const cache = new Map<string, AsyncReturnType<T>>();
  
  return {
    get: (key: string): AsyncReturnType<T> | undefined => cache.get(key),
    set: (key: string, value: AsyncReturnType<T>): void => {
      cache.set(key, value);
    }
  };
}
```

## Ограничения и особенности

1. Утилиты работают только с функциями и конструкторами
2. Некоторые утилиты могут быть сложны для понимания с сложными сигнатурами
3. При использовании с обобщенными функциями могут потребоваться дополнительные аннотации
4. Не все утилиты поддерживают корректную работу с перегруженными функциями

## Связанные концепции

- [[ts/utility-types/built-in-object-transformations|Утилиты для преобразования объектов]]
- [[ts/utility-types/built-in-property-selection|Утилиты для выбора свойств]]
- [[ts/type-system/type-inference|Вывод типов]]
- [[ts/generics/TS Generics|Дженерики]]
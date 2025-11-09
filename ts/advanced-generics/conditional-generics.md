# Условные дженерики

Условные дженерики - это мощная возможность TypeScript, которая позволяет создавать дженерики, поведение которых зависит от условий, проверяющих отношения между типами. Это открывает возможности для создания гибких и адаптивных обобщенных типов.

## Базовый синтаксис

Условные дженерики используют синтаксис условных типов внутри дженериков:

```typescript
type ConditionalGeneric<T> = T extends U ? X : Y;
```

## Простые примеры

### 1. Дженерик для обработки разных типов

```typescript
type ProcessType<T> = T extends string 
  ? `Processed string: ${T}`
  : T extends number 
    ? `Processed number: ${T}`
    : T extends boolean
      ? `Processed boolean: ${T}`
      : "Unsupported type";

type Result1 = ProcessType<"hello">;     // "Processed string: hello"
type Result2 = ProcessType<42>;          // "Processed number: 42"
type Result3 = ProcessType<true>;        // "Processed boolean: true"
type Result4 = ProcessType<{ a: 1 }>;    // "Unsupported type"
```

### 2. Дженерик для работы с массивами

```typescript
type ArrayElement<T> = T extends (infer U)[] ? U : T;

type Element1 = ArrayElement<string[]>;    // string
type Element2 = ArrayElement<number[]>;    // number
type Element3 = ArrayElement<string>;      // string (не массив)
```

## Продвинутые примеры

### 1. Дженерик для создания API клиентов

```typescript
interface ApiEndpoints {
  getUser: (id: string) => Promise<{ id: string; name: string }>;
  createUser: (data: { name: string }) => Promise<{ id: string; name: string }>;
  deleteUser: (id: string) => Promise<void>;
}

type ApiClient<T extends Record<string, (...args: any[]) => any>> = {
  [K in keyof T]: T[K] extends (...args: infer Args) => Promise<infer Result>
    ? (...args: Args) => Promise<Result>
    : never;
};

type UserApiClient = ApiClient<ApiEndpoints>;

// Создание реализации клиента
function createApiClient<T extends Record<string, (...args: any[]) => any>>(
  endpoints: T
): ApiClient<T> {
  const client = {} as ApiClient<T>;
  
  for (const key in endpoints) {
    client[key] = ((...args: any[]) => {
      // Здесь была бы реализация вызова API
      return endpoints[key](...args);
    }) as any;
  }
  
  return client;
}
```

### 2. Дженерик для работы с промисами

```typescript
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

async function fetchData(): Promise<{ data: string }> {
  return { data: "Hello" };
}

type DataType = UnwrapPromise<ReturnType<typeof fetchData>>; 
// { data: string }

// Дженерик для обработки массивов промисов
type UnwrapPromiseArray<T> = T extends Promise<infer U>[]
  ? U[]
  : T extends Promise<infer U>
    ? U
    : T;

async function fetchMultiple(): Promise<string[]> {
  return ["a", "b", "c"];
}

type MultipleResult = UnwrapPromiseArray<ReturnType<typeof fetchMultiple>>;
// string[]
```

### 3. Дженерик для создания фабрик

```typescript
type Constructor<T> = new (...args: any[]) => T;

type InstanceOf<T> = T extends Constructor<infer U> ? U : never;

class User {
  constructor(public name: string) {}
}

class Product {
  constructor(public title: string, public price: number) {}
}

function createInstance<T extends Constructor<any>>(
  Class: T,
  ...args: ConstructorParameters<T>
): InstanceOf<T> {
  return new Class(...args) as InstanceOf<T>;
}

const user = createInstance(User, "John");        // User
const product = createInstance(Product, "Laptop", 999); // Product
```

## Практические примеры

### 1. Дженерик для валидации форм

```typescript
type ValidationRule<T> = (value: T) => boolean;

interface ValidationSchema {
  [key: string]: ValidationRule<any>;
}

type ValidationResult<T extends ValidationSchema> = {
  [K in keyof T]: T[K] extends ValidationRule<infer U> ? U : never;
};

function validateForm<T extends ValidationSchema>(
  data: Record<string, any>,
  schema: T
): data is ValidationResult<T> {
  for (const key in schema) {
    const rule = schema[key];
    const value = data[key];
    
    if (!rule(value)) {
      return false;
    }
  }
  return true;
}

const userSchema = {
  name: (value: string) => typeof value === "string" && value.length > 0,
  age: (value: number) => typeof value === "number" && value > 0
};

const formData = {
  name: "John",
  age: 30
};

if (validateForm(formData, userSchema)) {
  // TypeScript знает, что formData имеет тип ValidationResult<typeof userSchema>
  console.log(formData.name); // ✅ string
  console.log(formData.age);  // ✅ number
}
```

### 2. Дженерик для работы с событиями

```typescript
interface EventMap {
  "user-login": { userId: string; timestamp: Date };
  "user-logout": { userId: string; timestamp: Date };
  "product-purchase": { productId: string; userId: string; amount: number };
}

type EventData<T extends keyof EventMap> = EventMap[T];

class EventEmitter {
  private listeners: { [K in keyof EventMap]?: ((data: EventMap[K]) => void)[] } = {};
  
  on<T extends keyof EventMap>(
    event: T, 
    listener: (data: EventData<T>) => void
  ): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }
  
  emit<T extends keyof EventMap>(
    event: T, 
    data: EventData<T>
  ): void {
    const listeners = this.listeners[event];
    if (listeners) {
      listeners.forEach(listener => listener(data));
    }
  }
}

const emitter = new EventEmitter();

emitter.on("user-login", (data) => {
  // TypeScript знает, что data имеет тип { userId: string; timestamp: Date }
  console.log(`User ${data.userId} logged in at ${data.timestamp}`);
});

emitter.emit("user-login", {
  userId: "123",
  timestamp: new Date()
}); // ✅ Типобезопасно
```

### 3. Дженерик для создания роутеров

```typescript
interface RouteParams {
  "/users/:id": { id: string };
  "/products/:category/:id": { category: string; id: string };
  "/search": { query: string };
}

type RouteHandler<T extends keyof RouteParams> = (
  params: RouteParams[T]
) => void;

class Router {
  private routes: { [K in keyof RouteParams]?: RouteHandler<K> } = {};
  
  addRoute<T extends keyof RouteParams>(
    path: T,
    handler: RouteHandler<T>
  ): void {
    this.routes[path] = handler;
  }
  
  navigate<T extends keyof RouteParams>(
    path: T,
    params: RouteParams[T]
  ): void {
    const handler = this.routes[path];
    if (handler) {
      handler(params);
    }
  }
}

const router = new Router();

router.addRoute("/users/:id", (params) => {
  // TypeScript знает, что params имеет тип { id: string }
  console.log(`Navigating to user ${params.id}`);
});

router.navigate("/users/:id", { id: "123" }); // ✅ Типобезопасно
// router.navigate("/users/:id", { name: "John" }); // ❌ Ошибка компиляции
```

## Распределительные условные дженерики

Когда условные дженерики применяются к объединению типов, они становятся распределительными:

```typescript
type ToArray<T> = T extends any ? T[] : never;

type StrArrOrNumArr = ToArray<string | number>; 
// string[] | number[] (распределено)

// Без распределения было бы (string | number)[]
```

## Дженерики с несколькими условиями

```typescript
type ComplexGeneric<T, U> = 
  T extends string 
    ? U extends number 
      ? `String: ${T}, Number: ${U}`
      : "First param is string, second is not number"
    : T extends number
      ? U extends string
        ? `Number: ${T}, String: ${U}`
        : "First param is number, second is not string"
      : "First param is neither string nor number";

type Result1 = ComplexGeneric<"hello", 42>;  // "String: hello, Number: 42"
type Result2 = ComplexGeneric<42, "hello">;  // "Number: 42, String: hello"
type Result3 = ComplexGeneric<"hello", "world">; // "First param is string, second is not number"
```

## Ограничения и особенности

1. Условные дженерики могут быть сложны для понимания
2. Сложные условные дженерики могут замедлять компиляцию
3. При использовании с большими объединениями могут потребоваться дополнительные аннотации
4. Не все условные дженерики могут быть корректно выведены TypeScript

## Связанные концепции

- [[ts/advanced-types/conditional-types|Условные типы]]
- [[ts/advanced-generics/generic-constraints|Ограничения дженериков]]
- [[ts/type-system/type-inference|Вывод типов]]
- [[ts/generics/TS Generics|Основы дженериков]]
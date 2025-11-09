# Ограничения дженериков (Generic Constraints)

Ограничения дженериков позволяют накладывать ограничения на типы, которые могут быть использованы в обобщенных функциях и типах. Это делает дженерики более мощными и безопасными.

## Базовый синтаксис

Ограничения накладываются с помощью ключевого слова `extends`:

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

const person = { name: "John", age: 30 };
const name = getProperty(person, "name"); // ✅ string
const age = getProperty(person, "age");   // ✅ number
// const invalid = getProperty(person, "invalid"); // ❌ Ошибка компиляции
```

## Простые ограничения

### 1. Ограничение примитивными типами

```typescript
function compare<T extends string | number>(a: T, b: T): number {
  if (a < b) return -1;
  if (a > b) return 1;
  return 0;
}

compare("a", "b"); // ✅
compare(1, 2);     // ✅
// compare("a", 1);   // ❌ Ошибка: типы не совпадают
```

### 2. Ограничение интерфейсами

```typescript
interface HasName {
  name: string;
}

function greet<T extends HasName>(obj: T): string {
  return `Hello, ${obj.name}!`;
}

const person = { name: "John", age: 30 };
const company = { name: "Acme Corp", address: "123 Main St" };

greet(person);   // ✅
greet(company);  // ✅
// greet({ age: 30 }); // ❌ Ошибка: нет свойства name
```

## Сложные ограничения

### 1. Множественные ограничения

```typescript
interface Serializable {
  serialize(): string;
}

interface Cloneable {
  clone(): this;
}

// Ограничение несколькими интерфейсами через пересечение
function process<T extends Serializable & Cloneable>(obj: T): T {
  console.log(obj.serialize());
  return obj.clone();
}

class User implements Serializable, Cloneable {
  constructor(public name: string) {}
  
  serialize(): string {
    return JSON.stringify(this);
  }
  
  clone(): this {
    return new User(this.name) as this;
  }
}

const user = new User("John");
const processedUser = process(user); // ✅
```

### 2. Ограничения с литеральными типами

```typescript
function createStyle<T extends "primary" | "secondary" | "danger">(type: T) {
  return {
    type,
    color: type === "primary" ? "blue" : 
           type === "secondary" ? "gray" : "red"
  };
}

const primaryStyle = createStyle("primary");   // ✅ { type: "primary", color: "blue" }
const dangerStyle = createStyle("danger");     // ✅ { type: "danger", color: "red" }
// const invalidStyle = createStyle("invalid");   // ❌ Ошибка компиляции
```

### 3. Ограничения с keyof

```typescript
// Получение значения по ключу с проверкой типов
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Установка значения с проверкой типов
function set<T, K extends keyof T>(obj: T, key: K, value: T[K]): void {
  obj[key] = value;
}

const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  debug: true
};

const apiUrl = get(config, "apiUrl"); // ✅ string
const timeout = get(config, "timeout"); // ✅ number
set(config, "debug", false); // ✅
// set(config, "invalid", "value"); // ❌ Ошибка компиляции
```

## Условные ограничения

### 1. Условные дженерики

```typescript
// Дженерик, который работает только с объектами
function processObject<T extends Record<string, any>>(obj: T): T {
  console.log("Processing object:", obj);
  return obj;
}

// Дженерик, который работает только с массивами
function processArray<T extends any[]>(arr: T): T {
  console.log("Processing array with", arr.length, "elements");
  return arr;
}

const obj = { name: "John" };
const arr = [1, 2, 3];

processObject(obj); // ✅
processArray(arr);  // ✅
// processObject(arr); // ❌ Ошибка: массив не является объектом
// processArray(obj);  // ❌ Ошибка: объект не является массивом
```

### 2. Ограничения с условными типами

```typescript
// Дженерик, который добавляет метод только к объектам
type WithToString<T> = T extends object 
  ? T & { toString(): string } 
  : T;

function addToString<T>(value: T): WithToString<T> {
  if (typeof value === "object" && value !== null) {
    return {
      ...value as any,
      toString() {
        return JSON.stringify(this);
      }
    };
  }
  return value as WithToString<T>;
}

const obj = addToString({ name: "John" });
console.log(obj.toString()); // ✅

const str = addToString("hello");
console.log(str.toString()); // ✅ (у строки уже есть toString)
```

## Практические примеры

### 1. ORM-подобный API

```typescript
interface Model {
  id: string;
  createdAt: Date;
  updatedAt: Date;
}

interface User extends Model {
  name: string;
  email: string;
}

interface Product extends Model {
  title: string;
  price: number;
}

// Функция поиска с ограничением на модели
function findById<T extends Model>(collection: T[], id: string): T | undefined {
  return collection.find(item => item.id === id);
}

const users: User[] = [
  { id: "1", name: "John", email: "john@example.com", createdAt: new Date(), updatedAt: new Date() }
];

const products: Product[] = [
  { id: "1", title: "Laptop", price: 999, createdAt: new Date(), updatedAt: new Date() }
];

const user = findById(users, "1");     // ✅ User | undefined
const product = findById(products, "1"); // ✅ Product | undefined
// const invalid = findById(["a", "b"], "1"); // ❌ Ошибка: string[] не extends Model
```

### 2. Конфигурационная система

```typescript
interface ConfigSchema {
  database: {
    host: string;
    port: number;
  };
  api: {
    baseUrl: string;
    timeout: number;
  };
}

// Типобезопасный доступ к конфигурации
function getConfigValue<
  T extends ConfigSchema,
  K extends keyof T,
  S extends keyof T[K]
>(config: T, section: K, key: S): T[K][S] {
  return config[section][key];
}

const config: ConfigSchema = {
  database: {
    host: "localhost",
    port: 5432
  },
  api: {
    baseUrl: "https://api.example.com",
    timeout: 5000
  }
};

const dbHost = getConfigValue(config, "database", "host"); // ✅ string
const apiTimeout = getConfigValue(config, "api", "timeout"); // ✅ number
// const invalid = getConfigValue(config, "invalid", "key"); // ❌ Ошибка компиляции
```

### 3. Валидация данных

```typescript
interface Validator<T> {
  validate(value: unknown): value is T;
  errorMessage: string;
}

// Дженерик для создания валидаторов с ограничениями
function createValidator<T extends Record<string, any>>(
  schema: { [K in keyof T]: Validator<T[K]> }
): Validator<T> {
  return {
    validate(value: unknown): value is T {
      if (typeof value !== "object" || value === null) {
        return false;
      }
      
      for (const key in schema) {
        const validator = schema[key];
        const propValue = (value as any)[key];
        
        if (!validator.validate(propValue)) {
          return false;
        }
      }
      
      return true;
    },
    errorMessage: "Validation failed"
  };
}

const userValidator = createValidator({
  name: { validate: (v): v is string => typeof v === "string", errorMessage: "Name must be string" },
  age: { validate: (v): v is number => typeof v === "number" && v > 0, errorMessage: "Age must be positive number" }
});

// ✅ Типобезопасная валидация
if (userValidator.validate({ name: "John", age: 30 })) {
  console.log(userValidator.name); // ✅ TypeScript знает, что это string
  console.log(userValidator.age);  // ✅ TypeScript знает, что это number
}
```

## Ограничения и особенности

1. Ограничения проверяются на этапе компиляции
2. Сложные ограничения могут замедлять компиляцию
3. Некоторые ограничения могут быть трудны для понимания
4. При использовании с большими объединениями могут потребоваться дополнительные аннотации

## Связанные концепции

- [[ts/generics/TS Generics|Основы дженериков]]
- [[ts/advanced-types/conditional-types|Условные типы]]
- [[ts/type-system/type-inference|Вывод типов]]
- [[ts/interfaces-classes/TS Interfaces and Classes|Интерфейсы и классы]]
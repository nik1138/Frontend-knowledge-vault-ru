# Generic Constraints

Ограничения обобщений (generic constraints) позволяют указывать требования к типам-параметрам в обобщенных типах и функциях, что делает обобщения более мощными и безопасными.

## Основные ограничения

### Базовое ограничение с extends
```typescript
// Ограничение типа параметром
interface Lengthwise {
  length: number;
}

// T должен иметь свойство length
function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // OK: теперь мы знаем, что у arg есть свойство length
  return arg;
}

loggingIdentity({ length: 10, value: 3 }); // OK
// loggingIdentity(3); // Ошибка: number не имеет свойства length
```

### Ограничения с базовыми типами
```typescript
// Ограничение примитивными типами
function getProperty<T, K extends string>(obj: T, key: K): T[K] {
  return obj[key];
}

// Ограничение числовыми значениями
function multiply<T extends number>(value: T, factor: number): number {
  return value * factor;
}
```

## Ограничения ключей объекта

### keyof ограничения
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // OK
// getProperty(x, "m"); // Ошибка: "m" не является ключом x
```

### Примеры использования keyof
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Функция для выбора подмножества свойств
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  for (const key of keys) {
    result[key] = obj[key];
  }
  return result;
}

const user: User = { id: 1, name: "Alice", email: "alice@example.com", age: 30 };
const nameAndEmail = pick(user, ['name', 'email']); 
// Тип: { name: string; email: string }
```

## Ограничения на основе существующих типов

### Использование существующих типов как ограничений
```typescript
class BeeKeeper {
  hasMask: boolean = true;
}

class ZooKeeper {
  nametag: string = "Mikle";
}

class Animal {
  numLegs: number = 4;
}

class Bee extends Animal {
  keeper: BeeKeeper = new BeeKeeper();
}

class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}

createInstance(Lion).keeper.nametag;  // OK
createInstance(Bee).keeper.hasMask;   // OK
```

### Ограничения с интерфейсами
```typescript
interface WithId {
  id: string;
}

interface WithName {
  name: string;
}

// Ограничение с пересечением
function processEntity<T extends WithId & WithName>(entity: T): T {
  console.log(`Processing ${entity.name} with ID ${entity.id}`);
  return entity;
}

const user = { id: "123", name: "Alice", email: "alice@example.com" };
processEntity(user); // OK: имеет и id, и name
```

## Сложные ограничения

### Ограничения с вычисляемыми свойствами
```typescript
function getPropertyByPath<T extends Record<string, any>, K extends keyof T>(
  obj: T,
  key: K
): T[K] {
  return obj[key];
}

// Более сложное ограничение для вложенных свойств
type NestedPath<T> = {
  [K in keyof T]: T[K] extends Record<string, any>
    ? K | `${K & string}.${NestedPath<T[K]> & string}`
    : K;
}[keyof T];

// Псевдо-реализация для демонстрации ограничений
```

### Ограничения с условиями
```typescript
// Условное ограничение
function mergeObjects<T extends Record<string, any>, U extends Record<string, any>>(
  obj1: T,
  obj2: U
): T & U {
  return { ...obj1, ...obj2 };
}

const result = mergeObjects({ a: 1 }, { b: 2 });
// Тип: { a: number; } & { b: number; }
```

## Практические примеры

### Ограничение для объектов с обязательными свойствами
```typescript
interface Configurable {
  enabled: boolean;
}

function configure<T extends Configurable>(item: T): T {
  item.enabled = true;
  return item;
}

interface Service {
  name: string;
  enabled: boolean;
  port: number;
}

const myService: Service = { name: "API", enabled: false, port: 3000 };
const configured = configure(myService); // Service с enabled: true
```

### Ограничения для функций
```typescript
interface Callable {
  (...args: any[]): any;
}

function callWithDebug<T extends Callable>(fn: T): T {
  return function (...args: any[]) {
    console.log("Calling function with args:", args);
    return fn.apply(this, args);
  } as T;
}

const add = (a: number, b: number) => a + b;
const debugAdd = callWithDebug(add);
debugAdd(1, 2); // Выведет лог и вернет 3
```

### Ограничения для конструкторов
```typescript
interface WithValidator {
  validate(): boolean;
}

function withValidation<T extends new (...args: any[]) => WithValidator>(
  Constructor: T
) {
  return class extends Constructor {
    constructor(...args: any[]) {
      super(...args);
      if (!this.validate()) {
        throw new Error("Validation failed");
      }
    }
  };
}

class MyForm implements WithValidator {
  data: string = "";
  
  validate(): boolean {
    return this.data.length > 0;
  }
}

const ValidatedForm = withValidation(MyForm);
// const form = new ValidatedForm(); // Бросит ошибку, если валидация не пройдена
```

## Ограничения с несколькими параметрами

### Взаимозависимые ограничения
```typescript
function getPropertyFromObject<
  T extends Record<string, any>,
  K extends keyof T
>(obj: T, key: K): T[K] {
  return obj[key];
}

// Более сложный пример с двумя зависимыми параметрами
function selectProperties<
  T extends Record<string, any>,
  K extends keyof T
>(obj: T, keys: K[]): T[K][] {
  return keys.map(key => obj[key]);
}

const user = { id: 1, name: "Alice", email: "a@example.com" };
const values = selectProperties(user, ['name', 'email']); // (string | number)[]
```

### Ограничения с обобщенными типами
```typescript
interface Container<T> {
  value: T;
}

function unwrapContainer<U, T extends Container<U>>(container: T): U {
  return container.value;
}

interface MyContainer extends Container<string> {
  id: number;
}

const myContainer: MyContainer = { value: "hello", id: 123 };
const unwrapped = unwrapContainer(myContainer); // string
```

## Продвинутые паттерны

### Ограничения с типами-утилитами
```typescript
// Ограничение объектом с определенными свойствами
function updateEntity<
  T extends Record<string, any>,
  K extends keyof T
>(entity: T, updates: Pick<T, K>): T {
  return { ...entity, ...updates };
}

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

const user: User = { id: 1, name: "Alice", email: "a@example.com", age: 30 };
const updated = updateEntity(user, { name: "Bob", age: 25 });
// Сохраняет все свойства, обновляет только указанные
```

### Ограничения с шаблонными литералами
```typescript
type EventName = `on${string}`;

function subscribe<T extends Record<EventName, Function>>(handlers: T): void {
  // Подписка на события
  Object.entries(handlers).forEach(([eventName, handler]) => {
    console.log(`Subscribed to ${eventName}`);
  });
}

// OK: имена событий начинаются с "on"
subscribe({
  onClick: () => console.log("Clicked"),
  onMouseDown: () => console.log("Mouse down")
});

// Ошибка: имя события не начинается с "on"
// subscribe({ click: () => console.log("Clicked") });
```

## Ошибки и ловушки

### Общие ошибки
```typescript
// ПЛОХО: слишком широкое ограничение
function badExample<T extends any>(arg: T): T {
  // return arg.length; // Ошибка: у T нет гарантии наличия length
  return arg;
}

// ЛУЧШЕ: конкретное ограничение
function goodExample<T extends { length: number }>(arg: T): T {
  console.log(arg.length); // OK: T гарантированно имеет length
  return arg;
}
```

### Проблемы с выводом типов
```typescript
// Иногда TypeScript не может вывести типы автоматически
function problematic<T extends string | number>(a: T, b: T): T {
  return a;
}

// Это может потребовать явного указания типа
// const result = problematic("hello", 42); // Ошибка
const result = problematic<number>("hello" as any, 42); // Нужно избегать
```

## Лучшие практики

1. **Используйте минимально возможные ограничения** - ограничивайте только тем, что действительно нужно
2. **Предпочитайте конкретные ограничения абстрактным** - `T extends { length: number }` лучше, чем `T extends object`
3. **Документируйте ограничения** - особенно сложные
4. **Используйте вспомогательные интерфейсы для сложных ограничений** - для лучшей читаемости
5. **Тестируйте граничные случаи** - убедитесь, что ограничения работают как ожидается

## Связь с другими концепциями
- [[../generics/generic-functions]] - как ограничения применяются к функциям
- [[../generics/generic-interfaces]] - ограничения в обобщенных интерфейсах
- [[../advanced-types/conditional-types]] - условные ограничения
- [[../type-system/type-compatibility]] - как ограничения влияют на совместимость
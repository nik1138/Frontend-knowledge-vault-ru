# Intersection Types

Пересечения типов (Intersection Types) в TypeScript объединяют два или более типа в один новый тип, который имеет все свойства из каждого из объединяемых типов. Это обозначается с помощью оператора `&`.

## Основы пересечений

### Простые пересечения
```typescript
// Пересечение двух простых типов
interface Name {
  name: string;
}

interface Age {
  age: number;
}

type Person = Name & Age;

const person: Person = {
  name: "Alice",
  age: 30
};

// Пересечение с дополнительными полями
type DetailedPerson = Person & {
  email: string;
  isActive: boolean;
};

const detailedPerson: DetailedPerson = {
  name: "Bob",
  age: 25,
  email: "bob@example.com",
  isActive: true
};
```

### Пересечение с примитивами
```typescript
// Хотя редко используется, пересечения могут включать примитивы
type StringAndNumber = string & number; // never (нет значений, которые могут быть одновременно строкой и числом)

// Более полезный пример с литералами
type PositiveNumber = number & { readonly brand: unique symbol };

function createPositive(value: number): PositiveNumber | null {
  if (value > 0) {
    return value as PositiveNumber;
  }
  return null;
}
```

## Практические применения

### Композиция функциональности
```typescript
// Разделение функциональности на отдельные интерфейсы
interface Timestamped {
  createdAt: Date;
  updatedAt: Date;
}

interface SoftDelete {
  deletedAt?: Date;
  isDeleted: boolean;
}

interface Validated {
  isValid: boolean;
  validationErrors?: string[];
}

// Комбинирование различных аспектов
type AuditableEntity<T> = T & Timestamped & SoftDelete & Validated;

interface User {
  id: number;
  name: string;
  email: string;
}

type AuditableUser = AuditableEntity<User>;

const user: AuditableUser = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  createdAt: new Date(),
  updatedAt: new Date(),
  isDeleted: false,
  isValid: true
};
```

### Пересечение функций
```typescript
// Пересечение функциональных типов (редко используется, но возможно)
interface FunctionWithProp {
  (x: number): number;
  displayName: string;
  version: number;
}

const add: FunctionWithProp = (x: number) => x + 1 as any;
add.displayName = "AddOne";
add.version = 1.0;

// Возврат пересечения типов из функции
function mergeObjects<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 } as T & U;
}

const nameObj = { name: "Alice" };
const ageObj = { age: 30 };
const merged = mergeObjects(nameObj, ageObj);
// merged имеет тип: { name: string; } & { age: number; }
// что эквивалентно: { name: string; age: number; }
```

## Продвинутые паттерны

### Пересечение с условными типами
```typescript
// Условное добавление свойств
type WithOptionalTimestamp<T, Timestamp extends boolean> = 
  T & (Timestamp extends true ? Timestamped : {});

type UserWithTimestamp = WithOptionalTimestamp<User, true>;
// { id: number; name: string; email: string; } & { createdAt: Date; updatedAt: Date; }

type UserWithoutTimestamp = WithOptionalTimestamp<User, false>;
// { id: number; name: string; email: string; } & {}
// что упрощается до: { id: number; name: string; email: string; }
```

### Пересечение с обобщениями
```typescript
// Обобщенные пересечения для создания гибких типов
interface Identifiable {
  id: string;
}

interface Timestamped {
  createdAt: Date;
}

function createEntity<T extends object>(
  base: T
): T & Identifiable & Timestamped {
  return {
    ...base,
    id: Math.random().toString(36),
    createdAt: new Date()
  } as T & Identifiable & Timestamped;
}

interface Product {
  name: string;
  price: number;
}

const product = createEntity<Product>({ name: "Laptop", price: 999 });
// Результат: Product & Identifiable & Timestamped
```

## Конфликты в пересечениях

### Обработка конфликтующих свойств
```typescript
// Когда пересекающиеся типы имеют свойства с одинаковыми именами, но разными типами
interface TypeA {
  value: string;
}

interface TypeB {
  value: number;
}

// Это создаст проблему - нет типа, который может быть одновременно string и number
// type Conflicting = TypeA & TypeB; // value: string & number = never

// Решение: убедиться, что типы совместимы
interface CompatibleA {
  value: string | number;
}

interface CompatibleB {
  value: number | boolean;
}

type Compatible = CompatibleA & CompatibleB;
// value: (string | number) & (number | boolean) = number
// (только number может быть одновременно string|number и number|boolean)
```

### Пересечение с одинаковыми структурами
```typescript
interface Common {
  id: number;
  name: string;
}

interface Extra1 {
  id: number;  // такой же тип
  name: string; // такой же тип
  email: string;
}

interface Extra2 {
  id: number;  // такой же тип
  name: string; // такой же тип
  age: number;
}

type Combined = Common & Extra1 & Extra2;
// Результат: { id: number; name: string; email: string; age: number; }
// Общие свойства должны иметь совпадающие типы
```

## Практические примеры

### API клиент с пересечениями
```typescript
// Определение базовых интерфейсов
interface ApiResponse {
  success: boolean;
  timestamp: Date;
}

interface Authenticated {
  userId: number;
  sessionId: string;
}

interface Paged {
  currentPage: number;
  totalPages: number;
  totalItems: number;
}

// Комбинирование для специфических ответов
interface UserListResponse extends ApiResponse, Paged {
  users: { id: number; name: string; email: string }[];
}

// Или через пересечение
type SecureUserListResponse = UserListResponse & Authenticated;

const response: SecureUserListResponse = {
  success: true,
  timestamp: new Date(),
  currentPage: 1,
  totalPages: 10,
  totalItems: 100,
  users: [{ id: 1, name: "Alice", email: "alice@example.com" }],
  userId: 123,
  sessionId: "abc123"
};
```

### Миксины (Mixins) с пересечениями
```typescript
// Базовый класс
class Component {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

// Типы для миксинов
type TimestampedMixin = {
  createdAt: Date;
  logCreation(): void;
};

type ValidatedMixin = {
  isValid(): boolean;
  validate(): void;
};

// Комбинирование типов
type EnhancedComponent = Component & TimestampedMixin & ValidatedMixin;

// Функция для расширения класса (в рантайме)
function withTimestamp<T extends new (...args: any[]) => any>(Base: T) {
  return class extends Base {
    createdAt = new Date();
    logCreation() {
      console.log(`Created at ${this.createdAt}`);
    }
  };
}

function withValidation<T extends new (...args: any[]) => any>(Base: T) {
  return class extends Base {
    isValid() {
      return true; // упрощенный пример
    }
    validate() {
      console.log("Validating...");
    }
  };
}

const EnhancedClass = withValidation(withTimestamp(Component));
const instance = new EnhancedClass("test");
// instance имеет тип, эквивалентный Component & TimestampedMixin & ValidatedMixin
```

## Сложные пересечения

### Пересечения с индексными сигнатурами
```typescript
// Объединение с индексной сигнатурой
interface SpecificProps {
  name: string;
  id: number;
}

type FlexibleObject = SpecificProps & { [key: string]: any };

const obj: FlexibleObject = {
  name: "Test",
  id: 123,
  dynamicProp: "can be anything",
  anotherProp: 456
};
```

### Пересечения с обобщенными типами
```typescript
// Сложный пример с обобщениями
interface WithId<T> {
  id: T;
}

interface WithName {
  name: string;
}

type IdentifiedNamed<T> = WithId<T> & WithName;

const user: IdentifiedNamed<string> = {
  id: "user123",
  name: "Alice"
};

const product: IdentifiedNamed<number> = {
  id: 12345,
  name: "Laptop"
};
```

## Лучшие практики

### Читаемость пересечений
```typescript
// ПЛОХО: слишком сложные вложенные пересечения
type OverlyComplex = 
  (User & Timestamped) & 
  (SoftDelete & Validated) & 
  { permissions: string[] } & 
  { [key: string]: any };

// ЛУЧШЕ: разбиение на понятные составляющие
interface UserPermissions {
  permissions: string[];
}

interface FlexibleEntity {
  [key: string]: any;
}

type ClearEntity = User & Timestamped & SoftDelete & Validated & UserPermissions & FlexibleEntity;
```

### Использование интерфейсов для расширения
```typescript
// Предпочтительно расширение через интерфейсы
interface BaseUser {
  id: number;
  name: string;
}

interface ExtendedUser extends BaseUser {
  email: string;
  createdAt: Date;
}

// Вместо пересечения
type ExtendedUserWithIntersection = BaseUser & {
  email: string;
  createdAt: Date;
};
```

## Сравнение с другими концепциями

### Пересечение vs Объединение
```typescript
interface A { a: string; }
interface B { b: number; }

// Пересечение: требует оба свойства
type Intersection = A & B; // { a: string; b: number; }

// Объединение: может быть любое из двух
type Union = A | B; // { a: string; } | { b: number; }

// Сравнение использования
const intersection: Intersection = { 
  a: "hello", 
  b: 42 
}; // Должно содержать оба свойства

const union: Union = {
  a: "hello"
  // b: 42  // не обязательно
}; // Должно соответствовать одному из типов
```

## Ошибки и ловушки

### Неправильное понимание пересечений
```typescript
// Общая ошибка: думать, что пересечение работает как простое объединение
interface Cat {
  name: string;
  meow(): void;
}

interface Dog {
  name: string;
  bark(): void;
}

// Это НЕ создает тип, который может быть или кошкой или собакой
type CatAndDog = Cat & Dog; // { name: string; meow(): void; bark(): void; }

// Это создает тип, который должен обладать всеми свойствами Cat И Dog
const animal: CatAndDog = {
  name: "Fluffy",
  meow: () => console.log("meow"),
  bark: () => console.log("woof")  // Должен реализовать оба!
};
```

## Связь с другими концепциями
- [[union-types]] - противоположность пересечений
- [[advanced-types/mapped-types]] - часто используется вместе с сопоставляющими типами
- [[interfaces-classes/inheritance]] - альтернатива наследованию
- [[advanced-types/conditional-types]] - можно комбинировать для сложной типизации
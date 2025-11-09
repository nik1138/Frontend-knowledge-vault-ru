# Functional Programming in TypeScript

Функциональное программирование в TypeScript - это парадигма программирования, которая использует функции как основные строительные блоки программ. Она подчеркивает чистые функции, неизменяемость и композицию, что делает код более предсказуемым, тестируемым и надежным.

## Основные концепции

### [Fundamentals](fundamentals.md)
Основы функционального программирования: чистые функции, неизменяемость, функции высшего порядка, функции как объекты первого класса.

### [Advanced Techniques](advanced.md)
Продвинутые техники функционального программирования: функторы, моноиды, линзы, FRP, монады и другие функциональные паттерны.

## Архитектурная схема

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        Functional Programming                           │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │    Core         │               │    Functional Design     │                                    │  Advanced       │
  │  Principles     │               │                          │                                    │   Concepts      │
  │                 │               │ - Pure Functions         │                                    │                 │
  │ - Immutability  │◄──────────────┤ - Higher-Order Functions │───────────────────────────────────►│ - Functors      │
  │ - Pure Functions│               │ - Function Composition   │                                    │ - Monads        │
  │ - Deterministic │               │ - Map/Filter/Reduce      │                                    │ - Category      │
  │ - No Side Effects│              │ - Currying & Partial   │                                    │   Theory        │
  │ - Referential   │               │   Application            │                                    │ - Lenses        │
  │   Transparency  │               │ - Recursion              │                                    │ - FRP           │
  └─────────────────┘               │ - Memoization            │                                    └─────────────────┘
        │                           └─────────────────┬────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │             Data Types            │
        │                           │                                 │
        │                           │ - Immutable Collections         │
        │                           │ - Maybe/Optional                │
        │                           │ - Either/Result                 │
        │                           │ - Future/Promise                │
        │                           │ - Reader/Writer/State           │
        │                           └─────────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                        Real-World Usage                               │
        │                           │                                                                         │
        │                           │ - State Management                                                      │
        │                           │ - Event Handling                                                        │
        │                           │ - Form Processing                                                       │
        │                           │ - API Communication                                                     │
        │                           │ - Error Handling                                                        │
        └───────────────────────────┤ - Data Transformation                                                   │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Основные принципы функционального программирования

### 1. Чистые функции (Pure Functions)
```typescript
// Чистая функция
const add = (a: number, b: number): number => a + b;

// Нечистая функция
let counter = 0;
const increment = (): number => ++counter; // изменяет внешнее состояние
```

### 2. Неизменяемость (Immutability)
```typescript
// Иммутабельные операции
const originalArray = [1, 2, 3];
const newArray = [...originalArray, 4]; // новый массив, старый не изменен

const originalObj = { name: "Alice" };
const newObj = { ...originalObj, age: 30 }; // новый объект
```

### 3. Функции высшего порядка (Higher-Order Functions)
```typescript
// Функция, принимающая другую функцию
const applyOperation = (value: number, operation: (n: number) => number): number => 
  operation(value);

// Использование
const result = applyOperation(5, n => n * 2); // 10
```

### 4. Композиция функций (Function Composition)
```typescript
// Композиция двух функций
const compose = <A, B, C>(f: (a: B) => C, g: (a: A) => B) => (x: A): C => f(g(x));

const addOne = (x: number): number => x + 1;
const multiplyByTwo = (x: number): number => x * 2;

const addOneThenMultiplyByTwo = compose(multiplyByTwo, addOne);
console.log(addOneThenMultiplyByTwo(5)); // (5 + 1) * 2 = 12
```

## Практические применения

### 1. Обработка данных
```typescript
// Функциональный подход к обработке данных
interface User {
  id: number;
  name: string;
  email: string;
  active: boolean;
}

const users: User[] = [
  { id: 1, name: "Alice", email: "alice@example.com", active: true },
  { id: 2, name: "Bob", email: "bob@example.com", active: false },
  { id: 3, name: "Charlie", email: "charlie@example.com", active: true }
];

// Цепочка функциональных преобразований
const activeUserEmails = users
  .filter(user => user.active)           // фильтрация
  .map(user => user.email);              // проекция

// Результат: ["alice@example.com", "charlie@example.com"]
```

### 2. Валидация данных
```typescript
// Функциональная валидация
type ValidationResult<T> = 
  | { success: true; data: T }
  | { success: false; errors: string[] };

type Validator<T> = (value: T) => ValidationResult<T>;

// Создание валидаторов
const required: Validator<string> = (value: string) => 
  value.trim().length > 0 
    ? { success: true, data: value } 
    : { success: false, errors: ["Value is required"] };

const minLength = (min: number): Validator<string> => (value: string) => 
  value.length >= min 
    ? { success: true, data: value } 
    : { success: false, errors: [`Min length is ${min}`] };

// Композиция валидаторов
const validateEmail: Validator<string> = (value: string) => 
  /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
    ? { success: true, data: value }
    : { success: false, errors: ["Invalid email format"] };
```

## Функциональные паттерны

### 1. Maybe/Optional
```typescript
// Тип для безопасной работы с null/undefined
type Maybe<T> = T | null | undefined;

const mapMaybe = <T, U>(value: Maybe<T>, fn: (v: T) => U): Maybe<U> => 
  value != null ? fn(value) : null;

// Использование
const user: Maybe<{ name: string }> = { name: "Alice" };
const upperName = mapMaybe(user, u => u.name.toUpperCase()); // "ALICE"
```

### 2. Either для обработки ошибок
```typescript
// Тип для обработки ошибок
type Either<L, R> = { type: "Left", value: L } | { type: "Right", value: R };

const left = <L, R>(value: L): Either<L, R> => ({ type: "Left", value });
const right = <L, R>(value: R): Either<L, R> => ({ type: "Right", value });

// Использование
const divide = (a: number, b: number): Either<string, number> => 
  b === 0 ? left("Division by zero") : right(a / b);

const result = divide(10, 2); // Right(5)
const error = divide(10, 0);  // Left("Division by zero")
```

## Преимущества функционального подхода

1. **Предсказуемость** - чистые функции всегда дают одинаковый результат
2. **Тестируемость** - легко тестировать в изоляции
3. **Параллелизм** - отсутствие побочных эффектов позволяет легкую параллелизацию
4. **Композиция** - легко соединять простые функции в сложные
5. **Отладка** - проще отлаживать из-за отсутствия скрытого состояния

## Совместимость с другим кодом

Функциональный подход отлично сочетается с другими парадигмами:

- **OOP** - функции как методы, функциональные утилиты в классах
- **Reactive Programming** - потоки данных как функциональные преобразования
- **Promise/Async** - цепочки асинхронных операций

## Связь с другими концепциями
- [[../types/fundamentals]] - функциональные типы
- [[../advanced-types/conditional-types]] - условные типы для функций
- [[../generics/index]] - обобщенные функциональные утилиты
- [[TypeScript Design Patterns Complete Guide]] - функциональные паттерны
- [[TypeScript Modules]] - функциональные утилиты как модули
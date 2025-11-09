# TypeScript Types

Система типов TypeScript - это основа языка, обеспечивающая статическую типизацию JavaScript. Типы позволяют указать, какие значения могут быть использованы в переменных, параметрах функций и других местах.

## Категории типов

### [Базовые типы](types-basics.md)
Основы системы типов: примитивы и литеральные типы.

#### Включает:
- [[basics/primitive-types]] - string, number, boolean, symbol, bigint, null, undefined
- [[basics/literal-types]] - строковые, числовые и булевые литералы

### [Объектные типы](types-objects.md)
Сложные структуры данных: объекты, функции, массивы, объединения и пересечения.

#### Включает:
- [[objects/object-types]] - объекты, интерфейсы, индексные сигнатуры
- [[objects/union-types]] - объединения типов, сужение, дискриминированные объединения
- [[objects/intersection-types]] - пересечения типов, композиция

### [Продвинутые типы](types-advanced.md)
Специальные типы и продвинутые паттерны типизации.

#### Включает:
- [[advanced/special-types]] - any, unknown, never, void
- [[advanced/template-literal-types]] - шаблонные литеральные типы, преобразования
- [[advanced/type-aliases]] - псевдонимы типов, обобщенные псевдонимы

## Структура типов

TypeScript использует следующую иерархию типов:

```
                    ┌─────────────┐
                    │  unknown    │ <- Top type
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  all types  │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐      ┌─────▼─────┐    ┌─────▼─────┐
    │ object  │      │ primitive │    │ function  │
    └─────────┘      └───────────┘    └───────────┘
         │                 │
    ┌────▼────┐      ┌─────▼─────────────┐
    │ array   │      │ string, number,   │
    └─────────┘      │ boolean, etc      │
                     └───────────────────┘
```

## Практические примеры

### Базовая типизация
```typescript
// Примитивы
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;

// Литералы
type Direction = "up" | "down" | "left" | "right";
let direction: Direction = "up";  // Ошибка при "middle"
```

### Объектные типы
```typescript
// Объекты
interface User {
  name: string;
  age: number;
}

// Объединения
let id: string | number;
id = "ABC123";  // OK
id = 123;       // OK

// Пересечения
type Person = User & { email: string };
```

### Продвинутые паттерны
```typescript
// Шаблонные литералы
type EventName = `user:${"login" | "logout" | "update"}`;
// "user:login" | "user:logout" | "user:update"

// Псевдонимы
type ApiResponse<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

// Специальные типы
type Guard<T> = (value: unknown) => value is T;
```

## Связь с другими концепциями
- [[relations-organized]] - связи между всеми категориями типов
- [[../basics/type-annotations]] - основы аннотаций типов
- [[../type-system/type-inference]] - автоматическое определение типов
- [[../type-system/type-compatibility]] - совместимость типов
- [[TypeScript Advanced Types]] - продвинутые возможности систем типов
- [[TS Generics]] - обобщенные типы
- [[TS Interfaces and Classes]] - объектно-ориентированные типы
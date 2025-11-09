# TypeScript Type System

Система типов TypeScript - это мощный статический анализатор типов, работающий до выполнения кода. Она обеспечивает безопасность типов, проверяет совместимость и позволяет создавать выразительные и надежные API.

## Основные принципы

### [Type Inference](type-inference.md)
Автоматическое определение типов компилятором на основе контекста

### [Type Compatibility](type-compatibility.md)
Структурное типирование и совместимость типов в TypeScript

### [Type Assertions](type-assertions.md)
Явное указание типа для значения при отсутствии информации у компилятора

### [Structural Typing](structural-typing.md)
Сравнение типов по структуре, а не по имени (в отличие от номинального типирования)

### [Duck Typing](duck-typing.md)
"Если выглядит как утка и крякает как утка, значит это утка" - принцип структурного типирования

## Расширенные концепции

### [Type Guards](type-guards.md)
Средства для уточнения типа в процессе выполнения

### [Control Flow Analysis](control-flow.md)
Анализ потока управления и сужение типов в зависимости от условий

### [Widening and Narrowing](widening-narrowing.md)
Расширение и сужение типов компилятором

### [Contextual Typing](contextual-typing.md)
Определение типов на основе контекста использования

## Принципы работы системы типов

### Инференция типов
```typescript
let x = 3;        // x имеет тип number
let arr = [1, 2]; // arr имеет тип number[]
```

### Совместимость структур
```typescript
interface Named {
  name: string;
}

class Person {
  name: string;
}

let n: Named;
n = new Person(); // OK, структуры совпадают
```

### Сужение типов
```typescript
function processValue(value: string | number) {
  if (typeof value === "string") {
    // Здесь value имеет тип string
    return value.toUpperCase();
  }
  // Здесь value имеет тип number
  return value.toFixed(2);
}
```

## Система типов в сравнении

| Язык | Типизация | Пример |
|------|-----------|--------|
| TypeScript | Статическая, структурная | Определяется во время компиляции |
| Java | Статическая, номинальная | Определяется классами |
| JavaScript | Динамическая | Определяется во время выполнения |

## Практические применения

### Безопасность типов
```typescript
// TypeScript предотвращает ошибки времени выполнения
function getLength(str: string): number {
  return str.length; // str точно является строкой
}

// getLength(123); // Ошибка компиляции
```

### Улучшенная автодополнение
```typescript
interface User {
  name: string;
  email: string;
  age: number;
}

function greetUser(user: User) {
  // Редактор будет предлагать автодополнение для user
  return `Hello, ${user.name}!`; // user.name точно существует
}
```

## Связь с другими концепциями
- [[types]] - базовые типы системы
- [[TypeScript Advanced Types]] - расширенные возможности
- [[TS Generics]] - обобщенные типы
- [[type-system/type-guards]] - конкретизация типов
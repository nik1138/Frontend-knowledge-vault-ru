# TypeScript Basics

Основы TypeScript включают введение в язык, его возможности и базовые концепции, необходимые для начала работы.

## Основные темы

### [Type Annotations](type-annotations.md)
Синтаксис аннотаций типов и где их использовать

### [Variables and Constants](variables.md)
Различия между var, let, const с точки зрения типизации

### [Functions](functions.md)
Аннотации типов для параметров и возвращаемых значений функций

### [Arrays](arrays.md)
Работа с массивами и типизация их элементов

### [Control Flow](control-flow.md)
Условные операторы, циклы и типизация в зависимости от потока выполнения

## Структура основ

TypeScript основы строятся на следующих взаимосвязанных концепциях:

```
                    ┌─────────────────────────────────────────────────────────┐
                    │               TypeScript Basics                         │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────┐
        ▼                                     ▼                                                   ▼
  ┌─────────────────┐               ┌──────────────────────────┐                            ┌─────────────────┐
  │ Type Annotations│               │      Variables &         │                            │  Control Flow   │
  │                 │               │      Constants           │                            │                 │
  │ - Primitive     │               │ - var, let, const        │                            │ - Conditional   │
  │   Types         │               │ - Type Inference         │                            │   Statements    │
  │ - Object Types  │               │ - Scope Rules            │                            │ - Loops         │
  │ - Function      │               │ - const vs readonly      │                            │ - Type Guards   │
  │   Types         │               │ - any vs unknown         │                            │ - Assertions    │
  └─────────────────┘               │ - Literal Types          │                            └─────────────────┘
        │                           └─────────────────┬────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │            Functions              │
        │                           │                                 │
        │                           │ - Parameters & Return Types     │
        │                           │ - Arrow Functions               │
        │                           │ - Function Types                │
        │                           │ - Generics in Functions         │
        │                           │ - Callbacks                     │
        │                           └─────────────────┬───────────────┘
        │                                             │
        │                           ┌─────────────────▼───────────────┐
        │                           │           Arrays                │
        │                           │                                 │
        │                           │ - Array Types                   │
        │                           │ - Methods (map, filter, etc.)   │
        │                           │ - Multidimensional Arrays       │
        │                           │ - Generics with Arrays          │
        └───────────────────────────┤ - Type Narrowing in Arrays      │
                                    └─────────────────────────────────┘
```

## Простые типы (повторение)

Хотя подробно описаны в [[types]], основы простых типов важны для понимания базы:

- **boolean** - логические значения
- **number** - числовые значения (все числа в JS)
- **string** - строковые значения
- **null** и **undefined** - специальные значения
- **void** - отсутствие значения
- **never** - тип для функций, которые никогда не завершаются
- **any** - динамически типизированное значение
- **unknown** - безопасный аналог any

## TypeScript vs JavaScript

### Преимущества TypeScript:
- Обнаружение ошибок до выполнения
- Лучшая автодополнение и навигация в коде
- Ясное определение интерфейсов API
- Улучшенная документация кода через типы

### Основные отличия:
```typescript
// JavaScript
function greet(name) {
  return "Hello, " + name;
}

// TypeScript
function greet(name: string): string {
  return "Hello, " + name;
}
```

## Песочница для практики

```typescript
// Простой пример для начала
let message: string = "Hello, TypeScript!";
let count: number = 42;

function add(a: number, b: number): number {
  return a + b;
}

// Использование
let result: number = add(count, 8);
console.log(`${message} Result: ${result}`);
```

## Связь с другими концепциями
- [[Связи между концепциями TypeScript Basics]] - связи между всеми основными концепциями
- [[types]] - расширенное рассмотрение типов
- [[../type-system/type-inference]] - автоматическое определение типов
- [[functions]] - более глубокое рассмотрение функций
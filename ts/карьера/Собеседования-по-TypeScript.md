---
aliases: [Собеседования по TypeScript, TypeScript собеседования]
tags: [typescript, interview, career]
---

# Собеседования-по-TypeScript

## Обзор

Собеседования по TypeScript могут включать вопросы о типизации, объектно-ориентированном программировании, функциональном программировании и других аспектах языка. В этой статье мы рассмотрим, как подготовиться к собеседованиям, связанным с TypeScript, и какие темы стоит изучить.

## Типизация

### Базовые типы

- `number`
- `string`
- `boolean`
- `any`
- `void`
- `null`
- `undefined`

### Объектные типы

- `object`
- `array`
- `tuple`
- `enum`

### Интерфейсы и типы

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type UserId = number;
```

### Дженерики

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

### Утилиты типов

- `Partial<T>`
- `Required<T>`
- `Readonly<T>`
- `Pick<T, K>`
- `Omit<T, K>`

## Объектно-ориентированное программирование

### Классы

```typescript
class User {
  id: number;
  name: string;

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }

  getId(): number {
    return this.id;
  }
}
```

### Наследование

```typescript
class Admin extends User {
  permissions: string[];

  constructor(id: number, name: string, permissions: string[]) {
    super(id, name);
    this.permissions = permissions;
  }
}
```

### Абстрактные классы

```typescript
abstract class Animal {
  abstract makeSound(): void;
}

class Dog extends Animal {
  makeSound(): void {
    console.log("Woof!");
  }
}
```

## Функциональное программирование

### Функции

```typescript
const add = (a: number, b: number): number => a + b;
```

### Колбэки и промисы

```typescript
function fetchData(callback: (data: string) => void): void {
  setTimeout(() => {
    callback("Data fetched");
  }, 1000);
}

async function fetchDataAsync(): Promise<string> {
  const response = await fetch("/api/data");
  return response.json();
}
```

## Практические задачи

### Задача 1: Фильтрация массива

```typescript
function filterByType<T>(array: T[], type: string): T[] {
  return array.filter(item => typeof item === type);
}
```

### Задача 2: Генерация уникального ID

```typescript
function generateId(): string {
  return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
}
```

## Вопросы на собеседовании

### Что такое TypeScript?

TypeScript — это язык программирования, который добавляет статическую типизацию к JavaScript. Он компилируется в JavaScript и позволяет избежать многих ошибок на этапе разработки.

### В чем разница между `interface` и `type`?

- `interface` используется для описания структуры объекта и может быть расширен.
- `type` используется для определения типа, включая примитивы, объединения и пересечения.

### Что такое дженерики?

Дженерики позволяют создавать компоненты, которые работают с различными типами данных, не ограничиваясь конкретным типом.

### Что такое утилиты типов?

Утилиты типов — это встроенные типы, которые позволяют манипулировать типами, например, `Partial<T>` делает все свойства типа `T` необязательными.

## Подготовка к собеседованию

### Изучите документацию

- [TypeScript документация](https://www.typescriptlang.org/docs/)

### Практика

- Решайте задачи на [LeetCode](https://leetcode.com/) с использованием TypeScript
- Создавайте проекты на TypeScript

### Ревью кода

- Просматривайте код других разработчиков
- Участвуйте в open-source проектах

## Внутренние ссылки

- [[Резюме-и-TypeScript]]
- [[Портфолио-разработчика]]
- [[Развитие-навыков]]

## Теги

- #typescript
- #interview
- #career
- #frontend
- #backend
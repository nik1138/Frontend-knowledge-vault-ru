---
aliases: ["Типы never и void", "void vs never", "Специальные типы возвращения"]
tags: [typescript, programming, types, void, never]
---

# never и void

Типы `void` и `never` в TypeScript представляют специальные случаи возвращаемых значений функций, но имеют важные различия в семантике и использовании. Оба типа относятся к "пустым" или "отсутствующим" значениям, но каждый из них используется в своих уникальных сценариях.

## Тип void

Тип `void` представляет отсутствие значения. Он используется в основном для функций, которые не возвращают значение.

### Основное использование void

```ts
// Функция с возвращаемым типом void
function warnUser(): void {
  console.log("Это сообщение с предупреждением");
}

// Переменная типа void может содержать только undefined или null (при отключенном strictNullChecks)
let unusable: void;
unusable = undefined; // OK
// unusable = null; // OK при отключенном strictNullChecks
// unusable = 1; // Ошибка: нельзя присвоить число переменной типа void
```

### Практические примеры void

```ts
// Функция, изменяющая DOM
function updateUI(message: string): void {
  const element = document.getElementById("status");
  if (element) {
    element.textContent = message;
  }
}

// Функция, вызывающая коллбэки
function executeCallback(cb: () => void): void {
  console.log("Выполняем действие...");
  cb(); // Вызываем коллбэк
  console.log("Действие выполнено");
}

// Использование
executeCallback(() => {
  console.log("Коллбэк выполнен");
});
```

### void в коллбэках

```ts
// При работе с коллбэками часто используется void
function forEach<T>(items: T[], callback: (item: T) => void): void {
  for (const item of items) {
    callback(item);
  }
}

// Использование
forEach([1, 2, 3], (num) => {
  console.log(num * 2); // Результат не используется
});
```

## Тип never

Тип `never` представляет значения, которые никогда не происходят. Он используется для функций, которые:

1. Всегда выбрасывают исключение
2. Никогда не завершаются (бесконечный цикл)

### Основное использование never

```ts
// Функция, которая всегда выбрасывает исключение
function error(message: string): never {
  throw new Error(message);
}

// Функция, которая никогда не завершается
function infiniteLoop(): never {
  while (true) {}
}

// Функция, которая возвращает never в определенных условиях
function fail(): never {
  return error("Что-то пошло не так");
}
```

### never в проверке типов

Тип `never` полезен для обеспечения полноты проверки типов:

```ts
// Перечисление
enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE"
}

// Функция, которая должна обрабатывать все значения перечисления
function getColorName(color: Color): string {
  switch (color) {
    case Color.Red:
      return "Красный";
    case Color.Green:
      return "Зеленый";
    case Color.Blue:
      return "Синий";
    default:
      // Если все значения перечисления обработаны,
      // то переменная color будет типа never
      const _exhaustiveCheck: never = color;
      return _exhaustiveCheck;
  }
}

// Если добавить новое значение в перечисление,
// TypeScript выдаст ошибку в default-ветке
```

### never в объединениях типов

```ts
// never исчезает из объединения типов
type NeverExample = string | number | never;
// Тип NeverExample эквивалентен string | number

// Практическое применение в утилитах типов
type NonNullable<T> = T extends null | undefined ? never : T;

type Example = NonNullable<string | null>; // string
```

## Практические примеры использования

### void в асинхронных функциях

```ts
// Асинхронная функция, не возвращающая значение
async function logUserData(userId: number): Promise<void> {
  try {
    const userData = await fetchUserData(userId);
    console.log("Данные пользователя:", userData);
  } catch (error) {
    console.error("Ошибка при получении данных пользователя:", error);
  }
}

// Функция для выполнения нескольких асинхронных операций
async function processAllUsers(userIds: number[]): Promise<void> {
  for (const id of userIds) {
    await logUserData(id);
  }
}
```

### never в обработке ошибок

```ts
// Тип для результатов операций
type Success = { success: true; data: string };
type Failure = { success: false; error: string };
type Result = Success | Failure;

function handleResult(result: Result): string {
  if (result.success) {
    return `Успешно: ${result.data}`;
  } else {
    return `Ошибка: ${result.error}`;
  }
  // TypeScript проверяет, что все ветви обработаны,
  // и не требует дополнительного возврата
}

// Функция, которая никогда не возвращает нормальное значение
function handleError(error: string): never {
  console.error("Критическая ошибка:", error);
  process.exit(1); // Завершает процесс
}
```

### never в условных типах

```ts
// Условный тип, который может вернуть never
type FilteredType<T> = T extends string 
  ? T 
  : T extends number 
    ? T 
    : never;

type Test1 = FilteredType<string>;  // string
type Test2 = FilteredType<number>;  // number
type Test3 = FilteredType<boolean>; // never

// Это полезно для создания типов, которые допускают только определенные входные типы
```

## Сравнение void и never

| Характеристика | void | never |
|---|---|---|
| Значения | undefined (и null при отключенном strictNullChecks) | Нет значений |
| Использование | Функции, не возвращающие значения | Функции, которые не завершаются или всегда выбрасывают исключения |
| Возвращаемый тип | Функции, возвращающие undefined | Функции, которые не возвращают ничего |
| Полезность в объединениях | Сохраняется | Исчезает |

### Пример сравнения

```ts
// Функция с типом void - завершается, но ничего не возвращает
function logMessage(message: string): void {
  console.log(message);
  // Явно не возвращается значение, возвращается undefined
}

// Функция с типом never - никогда не завершается нормально
function throwAlways(error: string): never {
  throw new Error(error);
  // Код после этой строки никогда не выполнится
}

// Функция с типом never - бесконечный цикл
function loopForever(): never {
  while (true) {
    // Никогда не выходит из цикла
  }
}
```

## Продвинутые сценарии использования

### never для обеспечения исчерпывающей проверки

```ts
interface Square {
  kind: "square";
  size: number;
}

interface Circle {
  kind: "circle";
  radius: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Square | Circle | Triangle;

// Функция для вычисления площади
function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "square":
      return shape.size * shape.size;
    case "circle":
      return Math.PI * shape.radius * shape.radius;
    case "triangle":
      return 0.5 * shape.base * shape.height;
    default:
      // Если все типы обработаны, shape будет never
      const _exhaustiveCheck: never = shape;
      throw new Error(`Необработанный тип фигуры: ${_exhaustiveCheck}`);
  }
}

// Если добавить новый тип в Shape, TypeScript выдаст ошибку в default-ветке
```

### void в коллбэках и обработчиках событий

```ts
// Обработчик события, который ничего не возвращает
function handleButtonClick(event: MouseEvent): void {
  console.log("Кнопка нажата");
  // Не возвращаем значение
}

// Функция для подписки на события
function subscribe<T>(
  event: string,
  handler: (data: T) => void
): () => void {
  // Логика подписки
  console.log(`Подписан на событие: ${event}`);
  
  // Возвращаем функцию отписки
  return () => {
    console.log(`Отписан от события: ${event}`);
  };
}

// Использование
const unsubscribe = subscribe<string>("user-login", (username) => {
  console.log(`Пользователь вошел: ${username}`);
});
```

## Заключение

Типы `void` и `never` играют важную роль в системе типов TypeScript, обеспечивая точное описание функций, которые не возвращают значения, и функций, которые никогда не завершаются нормально. Понимание этих типов помогает создавать более безопасный и документированный код, особенно при работе с обработкой ошибок и обеспечении полноты проверки типов.

Для более глубокого понимания рекомендуется ознакомиться с [[any-и-unknown]] и [[Контроль-типов]].

## Дополнительные ресурсы

- [[Типы-данных]] - основные типы данных в TypeScript
- [[any-и-unknown]] - типы any и unknown
- [[Контроль-типов]] - управление типами в TypeScript
- [[Утилиты-типов]] - встроенные утилиты для работы с типами
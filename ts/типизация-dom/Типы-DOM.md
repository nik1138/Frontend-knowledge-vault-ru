---
aliases: ["DOM Типизация", "Типы DOM", "TypeScript DOM"]
tags: [typescript, dom, типизация]
---

# Типы DOM в TypeScript

## Обзор

Типизация DOM (Document Object Model) в TypeScript позволяет разработчикам более безопасно работать с элементами, событиями и атрибутами веб-страниц. TypeScript предоставляет богатую систему типов для DOM API, что помогает избежать ошибок во время выполнения и улучшает опыт разработки.

## Основные концепции

### DOM в TypeScript

Когда вы работаете с DOM в TypeScript, вы сталкиваетесь с различными типами, которые представляют элементы, узлы, события и атрибуты. Эти типы определены в глобальной области видимости TypeScript и доступны без импорта.

### Основные типы DOM

- [[Элементы|DOM Элементы]] - базовые типы для элементов HTML
- [[Узлы|DOM Узлы]] - базовые типы для узлов документа
- [[События|DOM События]] - типы для работы с событиями
- [[Атрибуты|DOM Атрибуты]] - типы для работы с атрибутами элементов

## Примеры типизации

### Получение элемента по ID

```typescript
// Безопасное получение элемента с проверкой на null
const element = document.getElementById('myElement');

// TypeScript знает, что element может быть HTMLElement | null
if (element) {
  // Здесь element имеет тип HTMLElement
  console.log(element.tagName);
}
```

### Работа с конкретными типами элементов

```typescript
// Получение элемента input
const inputElement = document.querySelector('input[type="text"]');

// TypeScript определяет тип как HTMLInputElement | null
if (inputElement instanceof HTMLInputElement) {
  // Теперь TypeScript знает точный тип
  inputElement.value = 'Новое значение';
}
```

### Использование строгой типизации

```typescript
// Типизация для конкретного типа элемента
const button = document.querySelector('button') as HTMLButtonElement;

// Или с проверкой
const buttonElement = document.querySelector('button');
if (buttonElement instanceof HTMLButtonElement) {
  buttonElement.disabled = true;
}
```

## Расширенные типы

### HTML элементы

TypeScript предоставляет специфичные типы для различных HTML элементов:

- `HTMLInputElement` - для `<input>`
- `HTMLTextAreaElement` - для `<textarea>`
- `HTMLSelectElement` - для `<select>`
- `HTMLButtonElement` - для `<button>`
- `HTMLImageElement` - для `<img>`
- `HTMLAnchorElement` - для `<a>`
- `HTMLFormElement` - для `<form>`
- и многие другие

### SVG элементы

Также существуют типы для SVG элементов:

- `SVGElement` - базовый тип для SVG элементов
- `SVGSVGElement` - для `<svg>`
- `SVGPathElement` - для `<path>`

## Практические рекомендации

### Использование type guards

```typescript
function isInputElement(element: Element): element is HTMLInputElement {
  return element.tagName === 'INPUT';
}

const element = document.querySelector('input');
if (isInputElement(element)) {
  // element теперь имеет тип HTMLInputElement
  console.log(element.value);
}
```

### Создание пользовательских типов

```typescript
type ValidInputTypes = 'text' | 'email' | 'password' | 'number';

interface InputElement extends HTMLInputElement {
  type: ValidInputTypes;
}
```

### Работа с коллекциями элементов

```typescript
// Получение коллекции элементов
const inputs = document.querySelectorAll('input');

// inputs имеет тип NodeListOf<HTMLInputElement>
inputs.forEach(input => {
  // input имеет тип HTMLInputElement
  console.log(input.value);
});
```

## Заключение

Типизация DOM в TypeScript значительно улучшает надежность и читаемость кода. Понимание основных типов и способов их использования позволяет избежать многих распространенных ошибок при работе с DOM.

См. также:
- [[Элементы]]
- [[Узлы]]
- [[События]]
- [[Атрибуты]]
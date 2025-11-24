---
aliases: [ECMAScript Versions, ES Versions, ECMAScript Compatibility]
tags: [typescript, javascript, ecmascript, compatibility]
---

# Версии ECMAScript

## Обзор

ECMAScript - это спецификация языка программирования, на которой основан JavaScript. TypeScript компилируется в JavaScript, и поэтому важно понимать, как версии ECMAScript влияют на совместимость вашего кода. TypeScript позволяет указать целевую версию ECMAScript при компиляции, что определяет, какие возможности JavaScript будут использоваться в выходном коде.

## История версий ECMAScript

### ES3 (1999)
- Первоначальная версия с минимальными возможностями
- Поддерживается всеми браузерами
- Ограниченный функционал

### ES5 (2009)
- Добавлены важные методы массивов: `map`, `filter`, `reduce`
- JSON поддержка
- Strict mode
- Поддерживается всеми современными браузерами

### ES6/ES2015 (2015)
- Классы
- Модули
- Стрелочные функции
- Promise
- Let/const
- Деструктуризация
- Шаблонные строки

### ES2016
- Оператор возведения в степень (`**`)
- Метод `Array.prototype.includes`

### ES2017
- Async/await
- Shared memory и atomics
- Object.values, Object.entries, Object.getOwnPropertyDescriptors

### ES2018
- Rest/Spread свойства
- Async iteration
- Promise.prototype.finally

### ES2019
- Array.flat, Array.flatMap
- Object.fromEntries
- String.trimStart, String.trimEnd

### ES2020
- Nullish coalescing (`??`)
- Optional chaining (`?.`)
- BigInt
- Promise.allSettled
- globalThis

### ES2021
- Logical assignment operators
- WeakRefs
- String.replaceAll

### ES2022
- Top-level await
- Class fields
- Ergonomic brand checks for private fields

## Настройка в TypeScript

В файле `tsconfig.json` вы можете указать целевую версию ECMAScript:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"]
  }
}
```

## Влияние на транспиляцию

При указании более старой целевой версии TypeScript транспилирует современные возможности в более совместимые конструкции:

```typescript
// TypeScript код
const numbers = [1, 2, 3];
const doubled = numbers.map(n => n * 2);
const [first, ...rest] = numbers;
```

С `target: "ES5"`:
```javascript
// Результат транспиляции
var numbers = [1, 2, 3];
var doubled = numbers.map(function(n) { return n * 2; });
var first = numbers[0], rest = numbers.slice(1);
```

## Рекомендации

- Используйте самую новую версию ECMAScript, которую поддерживают ваши целевые среды
- Для широкой совместимости используйте ES5 или ES2015
- Учитывайте браузеры, которые вы поддерживаете, и их возможности
- Проверяйте поддержку возможностей на [caniuse.com](https://caniuse.com)

## Связанные темы

- [[Транспиляция]]
- [[Совместимость-с-браузерами]]
- [[Целевые-платформы]]
- [[Polyfills]]
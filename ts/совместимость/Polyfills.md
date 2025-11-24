---
aliases: [Polyfill, JavaScript Polyfills, ES6 Polyfills]
tags: [typescript, javascript, polyfills, compatibility]
---

# Polyfills

## Обзор

Polyfill - это код, который предоставляет современные возможности JavaScript в более старых браузерах, которые не поддерживают эти возможности изначально. Polyfills позволяют использовать современные возможности JavaScript в проектах, требующих совместимости со старыми браузерами.

## Зачем нужны Polyfills

Когда TypeScript компилируется в JavaScript для старых браузеров, транспиляция синтаксиса может быть недостаточной. Некоторые возможности, такие как новые методы объектов или глобальные функции, не могут быть транспилированы и требуют полифилов.

## Распространенные Polyfills

### Promise

```typescript
// Пример использования Promise
async function fetchData(): Promise<any> {
  const response = await fetch('/api/data');
  return response.json();
}
```

Для поддержки Promise в IE11:
```bash
npm install es6-promise
```

```javascript
// Включение полифила
import 'es6-promise/auto';
```

### Fetch API

```typescript
// Использование fetch
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data));
```

Для поддержки fetch в старых браузерах:
```bash
npm install whatwg-fetch
```

```javascript
// Включение полифила
import 'whatwg-fetch';
```

### Object.assign

```typescript
// Использование Object.assign
const obj1 = { a: 1 };
const obj2 = { b: 2 };
const merged = Object.assign({}, obj1, obj2);
```

Для поддержки в старых браузерах:
```bash
npm install core-js
```

```javascript
// Включение полифила для Object.assign
import 'core-js/es/object/assign';
```

## Использование core-js

`core-js` - это универсальный набор полифилов для ECMAScript:

```bash
npm install core-js
```

### Полный полифил

```javascript
// Подключение всех необходимых полифилов
import 'core-js/stable';
```

### Селективные полифилы

```javascript
// Подключение только необходимых полифилов
import 'core-js/es/array/includes';
import 'core-js/es/promise';
import 'core-js/es/object/assign';
```

## Modernizr

Modernizr - это библиотека для обнаружения возможностей браузера:

```bash
npm install modernizr
```

```javascript
// Проверка поддержки возможностей
if (Modernizr.promises) {
  // Использование Promise напрямую
} else {
  // Подключение полифила
  require('es6-promise/auto');
}
```

## Автоматическая загрузка полифилов

### Babel с core-js

```json
{
  "presets": [
    ["@babel/preset-env", {
      "useBuiltIns": "usage",
      "corejs": 3
    }]
  ]
}
```

Эта настройка автоматически добавляет полифилы только для возможностей, используемых в коде.

### Polyfill.io

Сервис, который предоставляет полифилы на основе User-Agent браузера:

```html
<script src="https://polyfill.io/v3/polyfill.min.js"></script>
```

## Настройка TypeScript с полифилами

```json
{
  "compilerOptions": {
    "target": "ES5",
    "lib": ["ES2015", "DOM"],
    "types": ["core-js"]
  },
  "include": [
    "node_modules/core-js/stable",
    "src/**/*"
  ]
}
```

## Практические рекомендации

### 1. Минимизация использования полифилов

```typescript
// Вместо использования новых методов
if (Array.prototype.includes) {
  return arr.includes(item);
} else {
  // Резервная реализация
  return arr.indexOf(item) !== -1;
}
```

### 2. Условная загрузка

```typescript
// Загрузка полифилов только при необходимости
if (!window.Promise) {
  import('es6-promise/auto').then(() => {
    // Запуск приложения
    startApp();
  });
} else {
  startApp();
}
```

### 3. Мониторинг размера бандла

```bash
npm install webpack-bundle-analyzer
```

Проверяйте, какие полифилы добавляют вес бандлу.

## Популярные библиотеки полифилов

1. **core-js** - универсальный набор полифилов
2. **es6-shim** - полифилы для ES6 возможностей
3. **babel-polyfill** (устарел, используйте core-js)
4. **regenerator-runtime** - для поддержки async/await

## Современные альтернативы

Для новых проектов, где не требуется поддержка старых браузеров, можно использовать нативные возможности без полифилов:

```typescript
// Современный код
const result = arr.flatMap(item => item.children);
const value = obj ?? defaultValue;
const nestedValue = obj?.property?.subproperty;
```

## Тестирование с полифилами

Убедитесь, что полифилы работают корректно в целевых браузерах:

```typescript
// Тестирование полифилов
describe('Polyfills', () => {
  it('should support Promise', () => {
    expect(window.Promise).toBeDefined();
  });

  it('should support Array.includes', () => {
    expect(Array.prototype.includes).toBeDefined();
  });
});
```

## Связанные темы

- [[Версии-ECMAScript]]
- [[Совместимость-с-браузерами]]
- [[Транспиляция]]
- [[Целевые-платформы]]
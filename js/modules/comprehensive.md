---
aliases: ["Модули JavaScript", "ES6 Модули", "CommonJS"]
tags: 
  - #javascript
  - #modules
  - #es6
  - #import
  - #export
  - #bundling
---

# Модули JavaScript: Полное руководство

Модули в JavaScript позволяют организовать код в отдельные файлы, обеспечивая лучшую структуру, повторное использование и управление зависимостями.

## ES6 модули vs CommonJS

### ES6 Модули (ESM)
- Статический импорт/экспорт (анализируется на этапе компиляции)
- Используются в браузерах и современных Node.js
- Поддерживают tree-shaking
- Синтаксис:
```javascript
// Экспорт
export const myVariable = 'value';
export function myFunction() { /* ... */ }
export default myDefaultExport;

// Импорт
import { myVariable, myFunction } from './module.js';
import myDefaultExport from './module.js';
import * as namespace from './module.js';
```

### CommonJS
- Динамический импорт/экспорт (выполняется во время выполнения)
- Используется в Node.js до внедрения ESM
- Синтаксис:
```javascript
// Экспорт
module.exports = { myVariable, myFunction };
exports.myVariable = 'value';

// Импорт
const { myVariable, myFunction } = require('./module');
```

## Синтаксис Import/Export

### Основные варианты экспорта
```javascript
// Именованный экспорт
export const name = 'value';
export function myFunc() { }

// Экспорт по умолчанию
export default function() { }

// Групповой экспорт
export { name, myFunc };
```

### Варианты импорта
```javascript
// Именованный импорт
import { name, myFunc } from './module.js';

// Импорт по умолчанию
import myDefault from './module.js';

// Импорт всего модуля
import * as module from './module.js';

// Комбинированный импорт
import myDefault, { name, myFunc } from './module.js';
import myDefault, * as module from './module.js';

// Переименование при импорте/экспорте
import { originalName as newName } from './module.js';
export { originalName as newName };
```

## Динамические импорты

Динамические импорты позволяют загружать модули по требованию:

```javascript
// Динамический импорт возвращает Promise
async function loadModule() {
    const module = await import('./module.js');
    return module;
}

// Условная загрузка
if (condition) {
    const module = await import('./moduleA.js');
} else {
    const module = await import('./moduleB.js');
}
```

## Процесс загрузки модулей

1. **Разбор** - браузер/движок разбирает import/export
2. **Поиск** - находит файлы по спецификации
3. **Компиляция** - компилирует модули
4. **Выполнение** - выполняет код модулей в правильном порядке

## Tree Shaking

Tree Shaking - это процесс удаления неиспользуемого кода при сборке:

```javascript
// В модуле есть несколько экспортов
export const utilA = () => 'A';
export const utilB = () => 'B';
export const utilC = () => 'C';

// При импорте только utilA, остальные будут исключены
import { utilA } from './utils.js'; // Только utilA будет включено в сборку
```

[[js/bundling]] подробно описывает процесс сборки и оптимизации.

## Циклические зависимости

JavaScript обрабатывает циклические зависимости следующим образом:

```javascript
// moduleA.js
import { valueB } from './moduleB.js';
export const valueA = 'A';
console.log(valueB); // undefined на момент выполнения

// moduleB.js
import { valueA } from './moduleA.js';
export const valueB = 'B';
```

## Паттерны модулей

### IIFE (Immediately Invoked Function Expression)
```javascript
const myModule = (function() {
    let privateVariable = 'private';
    
    return {
        publicMethod: function() {
            return privateVariable;
        }
    };
})();
```

### AMD (Asynchronous Module Definition)
```javascript
// Определение модуля
define(['dependency'], function(dep) {
    return {
        method: function() { }
    };
});

// Загрузка модуля
require(['myModule'], function(myModule) {
    // Использование модуля
});
```

## Совместимость с браузерами

- Современные браузеры поддерживают ES6 модули
- Для старых браузеров необходима транспиляция и сборка
- Используйте `<script type="module">` для загрузки модулей в браузерах

## Интеграция с инструментами сборки

### Webpack
- Поддерживает ES6, CommonJS, AMD
- Реализует tree-shaking
- Обрабатывает циклические зависимости

### Rollup
- Оптимизирован для библиотек
- Отличная поддержка tree-shaking
- Поддержка ES6 модулей

### Vite
- Использует нативные ES6 модули в разработке
- Быстрая загрузка модулей
- Автоматическая оптимизация

## Разрешение модулей

Node.js использует алгоритм разрешения модулей:
- Проверка относительных и абсолютных путей
- Поиск в node_modules
- Проверка файлов с расширениями (.js, .json, .node)

[[js/module-resolution]] содержит подробную информацию о процессе разрешения.

## Модульная федерация

Модульная федерация позволяет делиться модулями между приложениями:
- Позволяет загружать модули из удаленных источников
- Используется в микросервисных архитектурах
- Реализуется через Webpack 5

## Заключение

Модули JavaScript обеспечивают структурированный подход к организации кода. ES6 модули стали стандартом, но CommonJS все еще используется в Node.js. Понимание процесса загрузки, интеграции с инструментами сборки и оптимизаций позволяет создавать эффективные приложения.

## Связанные темы

- [[js/bundling]] - Подробнее о процессе сборки
- [[js/module-resolution]] - Разрешение модулей
- [[js/import-export-patterns]] - Шаблоны импорта/экспорта
- [[js/dependency-management]] - Управление зависимостями
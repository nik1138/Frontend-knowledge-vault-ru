# Module Compatibility

Совместимость модулей в TypeScript определяет, как различные системы модулей и форматы экспорта/импорта взаимодействуют друг с другом. TypeScript поддерживает несколько систем модулей и обеспечивает совместимость между ними.

## Основные системы модулей

### ES Modules vs CommonJS

#### ES Modules (ESM)
```typescript
// esm-module.ts
export const PI = 3.14159;
export function calculateArea(radius: number): number {
  return PI * radius * radius;
}

export default class Calculator {
  static multiply(a: number, b: number): number {
    return a * b;
  }
}

// Использование ES Modules
import Calculator, { PI, calculateArea } from './esm-module';
import * as MathUtils from './esm-module';

// Динамический импорт
async function loadModule() {
  const module = await import('./esm-module');
  return module;
}
```

#### CommonJS
```javascript
// cjs-module.js
const PI = 3.14159;

function calculateArea(radius) {
  return PI * radius * radius;
}

class Calculator {
  static multiply(a, b) {
    return a * b;
  }
}

// CommonJS экспорт
module.exports = {
  PI,
  calculateArea,
  Calculator
};

// Или по умолчанию
// module.exports = Calculator;

// Использование CommonJS
// const { PI, calculateArea, Calculator } = require('./cjs-module');
// const Calculator = require('./cjs-module');
```

### Совместимость между системами
```typescript
// TypeScript может импортировать CommonJS модули как ES Modules
import * as cjsModule from './cjs-module';
import { PI, calculateArea } from './cjs-module';

// И наоборот - ES Module как CommonJS
// const esmModule = require('./esm-module'); // В Node.js
```

## Совместимость форматов экспорта/импорта

### Default экспорты
```typescript
// default-export.ts
export default function greet(name: string): string {
  return `Hello, ${name}!`;
}

// Импорт default экспорта различными способами
import greet from './default-export';              // ES Module
import greet1 = require('./default-export');       // CommonJS имитация
import { default as greet2 } from './default-export'; // Импорт как именованный
```

### Именованные экспорт и импорт
```typescript
// named-exports.ts
export const MAX_SIZE = 100;
export const MIN_SIZE = 1;
export function validateSize(size: number): boolean {
  return size >= MIN_SIZE && size <= MAX_SIZE;
}

// Импорт именованных экспортов
import { MAX_SIZE, MIN_SIZE, validateSize } from './named-exports';
import { MAX_SIZE as MAX, validateSize as check } from './named-exports';
import * as Validators from './named-exports'; // всё как namespace

// Импорт с миграцией между системами
// CommonJS: const { MAX_SIZE, validateSize } = require('./named-exports');
```

### Re-exports (переэкспорт)
```typescript
// re-exports.ts
export { default as Calculator } from './calculator';
export { PI, calculateArea } from './geometry';
export * from './utils'; // всё из utils
export * as Geometry from './geometry'; // как namespace

// При импорте
import { Calculator, PI, calculateArea, Geometry } from './re-exports';
```

## Совместимость с Ambient Declarations

### Declaration файлы
```typescript
// types.d.ts - ambient declarations
declare module 'legacy-library' {
  export function legacyFunction(): void;
  export const VERSION: string;
}

// Совместимость с существующими JS библиотеками
declare module '*.json' {
  const value: any;
  export default value;
}

declare module '*!text' {
  const text: string;
  export default text;
}

// Использование
import data from './config.json';
import text from './template.txt!text';
```

### Triple-slash directives
```typescript
// reference-to-declaration.ts
/// <reference path="./types.d.ts" />

// Совместимость с глобальными объявлениями
declare global {
  interface Window {
    myCustomProperty: string;
  }
  
  namespace NodeJS {
    interface Global {
      myCustomGlobal: string;
    }
  }
}

export {};
```

## Совместимость TypeScript и JavaScript модулей

### Импорт из JavaScript
```javascript
// plain-js-module.js
function jsFunction() {
  return "Hello from JavaScript";
}

const jsObject = {
  value: 42
};

// В JavaScript нельзя использовать export default
// module.exports = { jsFunction, jsObject }; // CommonJS
export { jsFunction, jsObject }; // ES Module
```

```typescript
// ts-importer.ts
import { jsFunction, jsObject } from './plain-js-module';

// TypeScript будет использовать тип any для импорта из JS
// если нет соответствующего declaration файла
let result = jsFunction(); // тип: any
let value = jsObject.value; // тип: any
```

### Автоматическая генерация типов из JS
```json
// tsconfig.json для совместимости с JS
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": true,
    "declaration": true,
    "emitDeclarationOnly": true,
    "outDir": "./types"
  },
  "include": ["./src/**/*.js"]
}
```

## Совместимость между разными форматами компиляции

### Модульные системы в tsconfig
```json
// Разные настройки для разных систем модулей
{
  // Для ES Module
  "compilerOptions": {
    "module": "ES2020",
    "target": "ES2020",
    "outDir": "./dist/esm",
    "declaration": true,
    "declarationMap": true
  },
  
  // Для CommonJS
  {
    "module": "CommonJS",
    "target": "ES2018",
    "outDir": "./dist/cjs",
    "declaration": true
  }
}
```

### Совместимость форматов
```typescript
// universal-module.ts - совместимость с разными системами
export class UniversalClass {
  static create(): UniversalClass {
    return new UniversalClass();
  }
}

// Это будет работать в обеих системах:
// ES Module: export default UniversalClass;
// CommonJS: module.exports = UniversalClass;

// Универсальный экспорт для разных систем
function UniversalFunction() {
  return "Universal";
}

// Совместимость экспортов
if (typeof module !== 'undefined' && module.exports) {
  // CommonJS
  module.exports = { UniversalFunction, UniversalClass };
} else if (typeof define === 'function' && define.amd) {
  // AMD
  define([], function() { return { UniversalFunction, UniversalClass }; });
} else {
  // Глобальный объект
  (globalThis as any).UniversalModule = { UniversalFunction, UniversalClass };
}

// TypeScript позволяет использовать conditional exports в package.json
```

## Совместимость с Package Exports

### Conditional Exports
```json
// package.json с conditional exports
{
  "name": "my-package",
  "main": "./dist/cjs/index.js",
  "module": "./dist/esm/index.js",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js", 
      "types": "./dist/types/index.d.ts"
    },
    "./utils": {
      "import": "./dist/esm/utils.js",
      "require": "./dist/cjs/utils.js",
      "types": "./dist/types/utils.d.ts"
    }
  }
}
```

### Совместимость с разными средами
```typescript
// environment-specific.ts
declare const __NODE__: boolean;
declare const __BROWSER__: boolean;

// Код, совместимый с разными окружениями
let platformSpecific: string;

if (__NODE__) {
  // Код для Node.js
  const fs = await import('fs');
  platformSpecific = 'Node.js environment';
} else if (__BROWSER__) {
  // Код для браузера
  platformSpecific = 'Browser environment';
} else {
  platformSpecific = 'Unknown environment';
}
```

## Совместимость с разными версиями TypeScript

### Совместимость версий
```typescript
// Код, совместимый с разными версиями TypeScript
// Некоторые возможности могут быть недоступны в старых версиях

// Использование новых возможностей с проверками
type NewFeatureSupport = {
  // Проверка поддержки версии компилятора
  compilerVersion: string;
  features: {
    // Некоторые фичи могут быть опциональными
    optionalChaining?: boolean;
    nullishCoalescing?: boolean;
    topLevelAwait?: boolean;
    moduleBlocks?: boolean;
  };
};

// Для обеспечения обратной совместимости:
// - Используйте флаги компиляции
// - Обеспечьте полифилы для новых возможностей
// - Проверяйте версию TypeScript при необходимости
```

## Практические примеры совместимости

### Миграция с CommonJS на ES Modules
```typescript
// legacy-commonjs.ts - старый формат
function legacyFunction(): string {
  return "Legacy function";
}

const legacyObject = {
  property: "value"
};

// Старый экспорт
// module.exports = { legacyFunction, legacyObject }; // CommonJS

// Совместимый экспорт
export { legacyFunction, legacyObject }; // ES Module
export default { legacyFunction, legacyObject }; // для back-compat

// В новом коде можно использовать оба формата:
// import { legacyFunction } from './legacy-commonjs'; // ES Module стиль
// import legacyModule from './legacy-commonjs'; // default import
```

### Совместимость с UMD
```typescript
// umd-compatible.ts
(function (global, factory) {
  if (typeof module !== "undefined" && module.exports) {
    // CommonJS
    module.exports = factory();
  } else if (typeof define === "function" && define.amd) {
    // AMD
    define(factory);
  } else {
    // Глобальный скрипт
    (global as any).MyLibrary = factory();
  }
})(typeof self !== "undefined" ? self : this, function () {
  // Основной код библиотеки
  class MyLibrary {
    static version = "1.0.0";
    static doSomething() {
      return "UMD Compatible!";
    }
  }
  
  return MyLibrary;
});

// Для TypeScript объявления:
declare module 'my-library' {
  export = MyLibrary;
}

class MyLibrary {
  static version: string;
  static doSomething(): string;
}
```

## Совместимость зависимостей

### Совместимость с DefinitelyTyped
```typescript
// Использование типов из @types/*
import * as express from 'express';
import * as lodash from 'lodash';

// Совместимость с разными версиями типов
// @types/express@^4.0.0 для Express 4
// @types/lodash@^4.0.0 для Lodash 4

// Проверка версий в package.json
{
  "devDependencies": {
    "@types/node": "^16.0.0",
    "@types/express": "^4.17.0"
  }
}
```

### Peer Dependencies совместимость
```json
{
  "name": "my-typescript-package",
  "peerDependencies": {
    "typescript": ">=4.0.0 <5.0.0",
    "ts-node": ">=9.0.0"
  },
  "engines": {
    "node": ">=14.0.0"
  }
}
```

## Лучшие практики совместимости модулей

### 1. Использование современных стандартов
```typescript
// Рекомендуется использовать ES Modules как стандарт
export class RecommendedClass {}

// Но обеспечивайте совместимость с CommonJS при необходимости
export = RecommendedClass; // для CommonJS back-compat
```

### 2. Построение универсальных библиотек
```typescript
// universal-lib.ts - максимальная совместимость
export class UniversalLib {
  static create(): UniversalLib {
    return new UniversalLib();
  }
  
  doWork(): string {
    return "Universal work";
  }
}

// Обеспечиваем экспорт для разных систем:
// ES Module: import { UniversalLib } from './universal-lib';
// CommonJS: const { UniversalLib } = require('./universal-lib');
// AMD: require(['./universal-lib'], function(lib) {})
// Global: window.UniversalLib (если упаковано правильно)
```

### 3. Работа с разными версиями
```typescript
// compatibility-layer.ts - слой совместимости
function createCompatibleFunction() {
  if (supportsTopLevelAwait()) {
    return async () => { /* современная реализация */ };
  } else {
    return () => { /* совместимая реализация */ };
  }
}

function supportsTopLevelAwait(): boolean {
  try {
    // Проверка поддержки возможностей средой выполнения
    // Это может зависеть от версии Node.js или браузера
    return true; // упрощенный пример
  } catch {
    return false;
  }
}
```

## Ошибки и подводные камни

### Частые проблемы совместимости
```typescript
// 1. Смешивание систем модулей
// файл.ts - использует ES Module
export const value = 1;

// другой-файл.js - использует CommonJS
// const { value } = require('./файл'); // Может не работать как ожидается

// 2. Проблемы с default экспортом
// В TypeScript: export default class A {}
// В JS: module.exports = class A {}
// Импорт может отличаться в разных системах

// 3. Dynamic imports в разных средах
async function loadModuleCorrectly() {
  if (typeof require !== 'undefined') {
    // Node.js
    return require('./module');
  } else {
    // Браузер
    return await import('./module');
  }
}
```

## Совместимость с инструментами сборки

### Webpack и Rollup совместимость
```json
// webpack.config.js - настройки для совместимости
{
  "resolve": {
    "extensions": [".ts", ".tsx", ".js", ".jsx"],
    "alias": {
      // Совместимость с разными форматами
    }
  },
  "module": {
    "rules": [
      {
        "test": /\.tsx?$/,
        "use": "ts-loader",
        "exclude": /node_modules/
      }
    ]
  }
}
```

### Совместимость с Babel
```javascript
// babel.config.js - для совместимости TS и JS
module.exports = {
  presets: [
    ['@babel/preset-env', { targets: { node: 'current' } }],
    '@babel/preset-typescript',
  ],
  // Обеспечивает совместимость форматов модулей
  plugins: [
    // Плагины для преобразования модулей
  ],
};
```

## Заключение

Совместимость модулей в TypeScript требует внимательного подхода к:

1. **Выбору формата модулей** - ES Modules как современный стандарт
2. **Обеспечению обратной совместимости** - поддержка разных систем
3. **Работе с зависимостями** - версионирование и peer dependencies
4. **Настройке инструментов сборки** - для корректной обработки модулей
5. **Миграции кода** - постепенный переход между системами

Правильная совместимость модулей обеспечивает стабильность, переносимость и возможность миграции кодовой базы между разными окружениями и версиями инструментов.

## Связь с другими концепциями
- [[../modules/es-modules]] - подробнее о ES Modules
- [[../modules/commonjs]] - подробнее о CommonJS
- [[../modules/module-resolution]] - разрешение модулей
- [[../compilation/bundling]] - сборка и упаковка модулей
- [[../ecosystem/tools]] - инструменты для работы с модулями
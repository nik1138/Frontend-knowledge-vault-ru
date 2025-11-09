# Module Systems in TypeScript

Системы модулей в TypeScript обеспечивают организацию кода в отдельные файлы и управление зависимостями между ними. TypeScript поддерживает несколько систем модулей, каждая из которых имеет свои особенности и сценарии использования.

## История систем модулей

### AMD, CommonJS, UMD
```javascript
// AMD (Asynchronous Module Definition) - для браузеров
define(['dependency'], function(dep) {
  return {
    doSomething: function() { dep.util(); }
  };
});

// CommonJS (Node.js) - синхронная загрузка
const dep = require('dependency');
module.exports = {
  doSomething: function() { dep.util(); }
};

// UMD (Universal Module Definition) - универсальный формат
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define(['dependency'], factory);
  } else if (typeof module === 'object' && module.exports) {
    // Node.js
    module.exports = factory(require('dependency'));
  } else {
    // Браузер
    root.returnExports = factory(root.dependency);
  }
}(typeof self !== 'undefined' ? self : this, function (dep) {
  // код модуля
  return {
    doSomething: function() { dep.util(); }
  };
}));
```

### ES Modules (современный стандарт)
```typescript
// ES Modules (TypeScript)
import { util } from 'dependency';
import type { UtilityType } from 'dependency'; // только тип
import * as dep from 'dependency'; // namespace импорт
import defaultExport from 'dependency'; // default импорт

export { util };
export type { UtilityType }; // только тип
export default defaultExport;
export * from 'dependency'; // re-export
```

## Конфигурация модульных систем

### module в tsconfig.json
```json
{
  "compilerOptions": {
    // Системы модулей
    "module": "ES2020",        // ES Modules (современный стандарт)
    "module": "ESNext",        // Последняя версия ES Modules
    "module": "CommonJS",      // Node.js стандарт
    "module": "AMD",           // Для браузеров с RequireJS
    "module": "UMD",           // Универсальный формат
    "module": "System",        // SystemJS
    "module": "ES2022",        // ES2022 модули
    
    // Система резолюции модулей
    "moduleResolution": "node",     // Node.js стиль резолюции
    "moduleResolution": "classic",  // Классический TypeScript стиль (устаревший)
    
    // Совместимость
    "esModuleInterop": true,        // Совместимость ES/CommonJS
    "allowSyntheticDefaultImports": true, // Синтетические default импорты
    "resolveJsonModule": true,      // Импорт JSON модулей
    "allowUmdGlobalAccess": true    // Доступ к UMD модулям как к глобальным
  }
}
```

## Node.js ES Modules

### package.json для ES Modules
```json
{
  "name": "my-package",
  "version": "1.0.0",
  
  // Указание, что пакет использует ES Modules
  "type": "module",
  
  // Указание точек входа
  "main": "./dist/cjs/index.cjs",      // для CommonJS потребителей
  "module": "./dist/esm/index.js",     // для ES Modules потребителей
  "types": "./dist/types/index.d.ts",  // для типов TypeScript
  
  // Условные экспорты (новый стандарт)
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.cjs",
      "types": "./dist/types/index.d.ts"
    },
    "./utils": {
      "import": "./dist/esm/utils.js",
      "require": "./dist/cjs/utils.cjs",
      "types": "./dist/types/utils.d.ts"
    }
  },
  
  "scripts": {
    "build": "npm run build:esm && npm run build:cjs",
    "build:esm": "tsc --module ES2020 --outDir dist/esm",
    "build:cjs": "tsc --module CommonJS --outDir dist/cjs"
  }
}
```

### Совместимость импортов
```typescript
// В ES Module файле (с .js или type: "module" в package.json)
import { foo } from './foo.js';  // ДОЛЖЕН содержать .js расширение

// В CommonJS
const { foo } = require('./foo');  // РАСШИРЕНИЕ НЕ НУЖНО

// В TypeScript с разными целями
import { Config } from './config';  // TS компилятор будет добавлять/удалять расширения
```

## Современные возможности

### Import Attributes
```typescript
// TypeScript 5.0+ и ES2022+ - import attributes
import jsonData from './data.json' with { type: 'json' };
// или
import jsonData from './data.json' assert { type: 'json' };

// В tsconfig.json:
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "bundler"  // позволяет использовать import attributes
  }
}
```

### Top-Level Await
```typescript
// Модули могут использовать await на верхнем уровне
const config = await import('./config.json');
const database = await connectToDatabase(config.connectionString);

export { database };
```

## Системы резолюции модулей

### Node.js резолюция
```typescript
// node_modules/resolve-algorithm.ts
// 1. Абсолютные пути: /src/utils/helper.ts -> /src/utils/helper.ts
import { helper } from '/src/utils/helper';

// 2. Относительные пути: ./utils/helper.ts
// - ищет ./utils/helper.ts
// - ищет ./utils/helper/index.ts
// - ищет ./utils/helper.tsx, ./utils/helper.json и т.д.
import { helper } from './utils/helper';

// 3. Базовые модули: utils/helper.ts
// - ищет node_modules/utils/helper.ts
// - ищет node_modules/utils/helper/index.ts
// - ищет в baseUrl (если указан)
// - ищет в paths (если указаны)
import { helper } from 'utils/helper';
```

### Path mapping
```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",              // базовый путь для некорректных модулей
    "paths": {
      "@/*": ["src/*"],              // @/component -> src/component
      "@utils/*": ["src/utils/*"],    // @utils/helper -> src/utils/helper
      "@shared": ["src/shared/index"], // @shared -> src/shared/index.ts
      "library/*": ["node_modules/some-library/dist/*"]  // переопределение пути
    }
  }
}
```

```typescript
// Использование aliased импортов
import { Component } from '@/components/Component';
import { formatDate } from '@utils/date';
import { SharedUtils } from '@shared';
import { SpecificHelper } from 'library/helpers';
```

## Практические примеры

### Многоуровневая архитектура модулей
```typescript
// src/core/index.ts
export { Service } from './Service';
export { EventEmitter } from './EventEmitter';
export type { ServiceConfig } from './types';

// src/utils/index.ts
export { Logger } from './Logger';
export { Validator } from './Validator';
export { formatter } from './formatter';

// src/index.ts - главный экспорт
export * from './core';
export * from './utils';
export { default as App } from './App';
```

### Утилиты для работы с модулями
```typescript
// src/module-utils.ts
type ModuleLoader<T> = () => Promise<T>;

class ModuleManager {
  private cache = new Map<string, any>();
  
  async load<T>(modulePath: string, force?: boolean): Promise<T> {
    if (!force && this.cache.has(modulePath)) {
      return this.cache.get(modulePath);
    }
    
    const module = await import(modulePath);
    this.cache.set(modulePath, module);
    return module as T;
  }
  
  async loadOnce<T>(modulePath: string): Promise<T> {
    if (this.cache.has(modulePath)) {
      return this.cache.get(modulePath);
    }
    
    const module = await this.load<T>(modulePath);
    this.cache.set(modulePath, module);
    return module;
  }
  
  clear(path?: string): void {
    if (path) {
      this.cache.delete(path);
    } else {
      this.cache.clear();
    }
  }
}

// Использование
const moduleManager = new ModuleManager();
const { SomeModule } = await moduleManager.load<typeof import('./SomeModule')>('./SomeModule');
```

### Условная загрузка модулей
```typescript
// src/platform-loader.ts
async function loadPlatformModule() {
  if (typeof window !== 'undefined') {
    // Браузерная версия
    return await import('./platform/browser');
  } else if (typeof require !== 'undefined') {
    // Node.js версия
    return await import('./platform/node');
  } else if (typeof WorkerGlobalScope !== 'undefined') {
    // Web Worker версия
    return await import('./platform/worker');
  }
  
  throw new Error('Unknown platform');
}

// Использование
const platformModule = await loadPlatformModule();
const platformService = new platformModule.PlatformService();
```

### Динамический импорт с кешированием
```typescript
// src/dynamic-loader.ts
class CachedImporter {
  private static cache = new Map<string, Promise<any>>();
  
  static async import<T>(modulePath: string): Promise<T> {
    if (this.cache.has(modulePath)) {
      return this.cache.get(modulePath) as Promise<T>;
    }
    
    const promise = import(modulePath);
    this.cache.set(modulePath, promise);
    
    return promise as Promise<T>;
  }
  
  static clearCache(modulePath?: string): void {
    if (modulePath) {
      this.cache.delete(modulePath);
    } else {
      this.cache.clear();
    }
  }
}

// Использование
const { heavyModule } = await CachedImporter.import<{ heavyModule: any }>('./heavy-module');
```

## Совместимость с разными системами

### Node.js CommonJS vs ES Modules
```javascript
// commonjs-compat.js
const { createRequire } = require('module');
const require = createRequire(import.meta.url);
const config = require('./config.json'); // CommonJS импорт в ES Module окружении

// typescript-transpiled.js (из .ts файла)
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.myFunction = void 0;
const dependency_1 = require("dependency");
var __importDefault = (this && this.__importDefault) || function (mod) {
    return (mod && mod.__esModule) ? mod : { "default": mod };
};
const someLib_1 = __importDefault(require("someLib")); // для default импортов
```

### Совместимость с Webpack
```javascript
// webpack.config.js - поддержка разных систем модулей
module.exports = {
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@utils': path.resolve(__dirname, 'src/utils')
    }
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader', // использует tsconfig.json для module настроек
        exclude: /node_modules/
      }
    ]
  }
};
```

## Паттерны организации модулей

### Barrel exports
```typescript
// src/components/index.ts - barrel файл
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
export type { ButtonProps, InputProps, ModalProps } from './types';

// src/services/index.ts
export { ApiService } from './ApiService';
export { StorageService } from './StorageService';
export { WebSocketService } from './WebSocketService';

// src/types/index.ts
export type { User } from './User';
export type { Config } from './Config';
export type { ApiResponse } from './ApiResponse';
```

### Feature-based структура
```typescript
// src/features/user/index.ts
export { UserService } from './service/UserService';
export { UserComponent } from './ui/UserComponent';
export type { User, UserUpdatePayload } from './types';

// src/features/order/index.ts
export { OrderService } from './service/OrderService';
export { OrderListComponent } from './ui/OrderListComponent';
export type { Order, OrderStatus } from './types';

// src/index.ts
export { UserService, UserComponent } from './features/user';
export { OrderService, OrderListComponent } from './features/order';
```

### Lazy loading с модулями
```typescript
// src/lazy-modules.ts
type LazyModule<T> = () => Promise<T>;

export const lazyLoadUserModule = (): LazyModule<typeof import('./features/user')> => 
  () => import('./features/user');

export const lazyLoadOrderModule = (): LazyModule<typeof import('./features/order')> => 
  () => import('./features/order');

// В компоненте
async function loadUserModule() {
  const loadModule = lazyLoadUserModule();
  const userModule = await loadModule();
  return userModule.UserService;
}
```

## Лучшие практики

### 1. Используйте ES Modules как стандарт
```typescript
// ПЛОХО: микс систем модулей в одном проекте
// some-file.ts
import { helper } from './helper'; // ES Module
const utils = require('./utils'); // CommonJS в том же файле

// ХОРОШО: единообразное использование
// some-file.ts
import { helper } from './helper';
import { utils } from './utils';
```

### 2. Организуйте barrel exports разумно
```typescript
// ХОРОШО: логическая группировка
// src/ui/index.ts
export { Button, Input, Modal } from './controls';
export { Header, Footer, Layout } from './layout';

// ПЛОХО: слишком много экспорта в одном barrel
// src/index.ts
export * from './controls/Button';
export * from './controls/Input'; 
export * from './controls/Modal';
// ... 20+ дополнительных экспорта
```

### 3. Используйте явные расширения при необходимости
```typescript
// В ES Module окружениях в Node.js
import { helper } from './helper.js';        // явное .js расширение
import { type Config } from './config.d.ts';   // явное .d.ts расширение

// В CommonJS или с moduleResolution: "node"
import { helper } from './helper';            // без расширения
```

### 4. Правильно настраивайте tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@tests/*": ["__tests__/*"]
    },
    "rootDir": "src",
    "outDir": "dist",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "importHelpers": true,
    "downlevelIteration": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "skipLibCheck": true,
    "resolveJsonModule": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

## Современные тенденции

### Module Federation (для микросервисов)
```typescript
// Вебпак 5 Module Federation позволяет делиться модулями между приложениями
// webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'user_app',
      filename: 'remoteEntry.js',
      exposes: {
        './UserService': './src/remote-services/UserService',
      },
      shared: {
        'typescript': { singleton: true },
        'vue': { singleton: true }
      }
    })
  ]
};
```

### Tree-shaking оптимизации
```typescript
// Убедитесь, что ваши модули позволяют tree-shaking
// ПЛОХО: side effects при импорте
import 'bootstrap/dist/css/bootstrap.css'; // импорт с побочными эффектами

// ХОРОШО: чистые экспорты для tree-shaking
export function utils() { /* только функции */ }
export const constants = { /* только константы */ }
// без побочных эффектов при импорте
```

## Ошибки и решения

### 1. Проблемы с дублированием модулей
```typescript
// Решение: использовать exact versions и npm dedupe
// package.json
{
  "resolutions": {  // для Yarn
    "typescript": "4.8.4"
  },
  "overrides": {    // для npm 8.3+
    "typescript": "4.8.4"
  }
}
```

### 2. Циклические зависимости
```typescript
// ПЛОХО: циклическая зависимость
// A.ts -> B.ts -> C.ts -> A.ts

// РЕШЕНИЕ: извлеките общую зависимость
// common.ts
export interface SharedInterface { /* общий интерфейс */ }

// A.ts и B.ts импортируют common.ts, но не друг друга напрямую
```

### 3. Проблемы с type-only импортами
```typescript
// Использование import type для типов
import type { SomeInterface } from './types';  // только тип
import { SomeClass } from './class';          // только значение

// Или в одном импорте
import { type SomeInterface, SomeClass } from './combined';
```

## Производительность

### Lazy loading модулей
```typescript
// src/router.ts
interface Route {
  path: string;
  component: () => Promise<any>;
}

const routes: Route[] = [
  { 
    path: '/home', 
    component: () => import('./pages/Home')  // загрузка при посещении
  },
  { 
    path: '/profile', 
    component: () => import('./pages/Profile')  // загрузка при посещении  
  },
  { 
    path: '/admin', 
    component: () => import('./pages/Admin')  // загрузка при посещении
  }
];
```

## Заключение

Системы модулей в TypeScript:

1. **Обеспечивают организацию кода** в логические единицы
2. **Управляют зависимостями** между частями приложения  
3. **Позволяют повторное использование** кода между проектами
4. **Поддерживают совместимость** с разными средами выполнения
5. **Улучшают производительность** через ленивую загрузку и tree-shaking

Правильное понимание систем модулей критично для создания масштабируемых и поддерживаемых TypeScript приложений.

## Связь с другими концепциями
- [[build-tools/typescript-compiler]] - как компилятор обрабатывает модули
- [[build-tools/webpack]] - интеграция с вебпаком
- [[build-tools/vite]] - современные билды с ES Modules
- [[../type-system/type-compatibility]] - совместимость типов между модулями
- [[ecosystem/package-management]] - управление модулями как пакетами
- [[deployment/bundling]] - упаковка модулей для деплоя
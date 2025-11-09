# TypeScript Module Systems

Системы модулей в TypeScript определяют, как код организуется в отдельные файлы и как зависимости между ними управляются. Это ключевая часть архитектуры современных TypeScript приложений.

## Основные концепции модулей

### [Module Systems](module-systems.md)
Различные системы модулей: ES Modules, CommonJS, AMD, UMD и их использование в TypeScript.

### [Module Resolution](module-resolution.md)
Как TypeScript находит и разрешает импорты, настройка baseUrl, paths и других опций.

### [Import/Export Syntax](import-export.md)
Подробное руководство по различным способам импорта и экспорта в TypeScript.

### [Module Bundling](bundling.md)
Создание бандлов и оптимизация модулей для production.

### [Dynamic Imports](dynamic-imports.md)
Использование динамических импортов и ленивой загрузки.

### [Module Declaration Files](declaration-files.md)
Создание файлов объявлений для модулей JavaScript.

## Архитектурная схема модульной системы

```
                    ┌─────────────────────────────────────────────────────────┐
                    │        TypeScript Module System                        │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │   ES Modules    │               │    CommonJS System     │                                    │  Module         │
  │  (Modern)       │               │  (Node.js)             │                                    │  Resolution     │
  │                 │               │                        │                                    │  Strategy       │
  │ - import/export │◄──────────────┤ - require/module.exports │─────────────────────────────────────►│ - Node         │
  │ - Static        │               │ - Synchronous loading  │                                    │ - Classic      │
  │   Analysis      │               │ - Runtime resolution   │                                    │ - Bundler      │
  │ - Tree Shaking  │               │ - Callback-style       │                                    │ - Config       │
  │ - Top-level     │               │ - Callback Hell        │                                    │ - Paths        │
  │   await         │               │ - Legacy compatibility │                                    │ - baseUrl      │
  └─────────────────┘               └─────────────────┬────────┘                                    └─────────────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │        AMD System           │
        │                           │                               │
        │                           │ - Asynchronous Module        │
        │                           │   Definition                │
        │                           │ - Browser-focused            │
        │                           │ - RequireJS compatibility    │
        │                           │ - Legacy but still used      │
        │                           └───────────────────────────────┘
        │
        │                           ┌─────────────────────────────────────────────────────────────────────────┐
        │                           │                        Module Formats                             │
        │                           │                                                                 │
        │                           │ - Universal Module Definition (UMD)                             │
        │                           │ - SystemJS                                                      │
        │                           │ - Native Browser Modules                                        │
        │                           │ - Legacy Formats (IIFE, Global)                                 │
        │                           │ - Interoperability                                              │
        │                           │ - Format Detection & Conversion                                 │
        └───────────────────────────┤ - Cross-Platform Compatibility                                  │
                                    └─────────────────────────────────────────────────────────────────────────┘
```

## Типы модулей

### ES Modules (современный стандарт)
```typescript
// Использование современного синтаксиса
import { Component } from './Component';           // именованный импорт
import DefaultExport from './DefaultExport';        // default импорт
import * as Utils from './Utils';                   // namespace импорт
import type { SomeType } from './Types';           // только типы (не экспортируется в JS)

export { Component };                                // именованный экспорт
export default DefaultExport;                        // default экспорт
export * from './OtherModule';                      // re-export
export type { SomeType };                          // только типы
```

### CommonJS (Node.js стандарт)
```javascript
// CommonJS синтаксис (все еще используется в Node.js)
const { Component } = require('./Component');
const DefaultExport = require('./DefaultExport').default;

module.exports = { Component, DefaultExport };
module.exports.DefaultExport = DefaultExport;
```

## Основные особенности системы модулей

### 1. Является модулем
Файл становится модулем, если он содержит `import` или `export`:

```typescript
// global.ts - не модуль (глобальная область видимости)
function globalFunction() { return "I'm global"; }

// module.ts - модуль (локальная область видимости)
function moduleFunction() { return "I'm in a module"; }
export { moduleFunction }; // делает файл модулем
```

### 2. Разрешение модулей
```json
// tsconfig.json
{
  "compilerOptions": {
    "moduleResolution": "node",  // или "classic"
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  }
}
```

### 3. Совместимость с разными целями
```typescript
// Код может быть скомпилирован под разные системы модулей
// tsconfig.node.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "module": "CommonJS",
    "target": "ES2018",
    "outDir": "./dist/node"
  }
}

// tsconfig.browser.json
{
  "extends": "./tsconfig.json", 
  "compilerOptions": {
    "module": "ESNext",
    "target": "ES2020",
    "outDir": "./dist/browser"
  }
}
```

## Практические примеры

### 1. Структура проекта с модулями
```
src/
├── index.ts
├── components/
│   ├── index.ts            // barrel export
│   └── Button.ts
├── services/
│   ├── index.ts
│   └── UserService.ts
├── utils/
│   ├── index.ts
│   ├── validation.ts
│   └── formatting.ts
├── types/
│   ├── index.ts
│   └── models.ts
└── config/
    └── index.ts
```

```typescript
// src/components/index.ts (barrel)
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';

// src/services/index.ts
export { UserService } from './UserService';
export { ApiService } from './ApiService';

// src/utils/index.ts
export { validateEmail } from './validation';
export { formatDate } from './formatting';

// src/index.ts
import { Button, Input } from './components';    // из barrel
import { UserService } from './services';       // из barrel
import * as utils from './utils';               // namespace import
import config from './config';                   // default import
```

### 2. Динамические импорты для ленивой загрузки
```typescript
//Ленивая загрузка модулей
async function loadFeature(feature: string) {
  switch (feature) {
    case 'admin':
      const { AdminPanel } = await import('./features/AdminPanel');
      return AdminPanel;
    case 'dashboard':
      const { Dashboard } = await import('./features/Dashboard');
      return Dashboard;
    default:
      const { DefaultFeature } = await import('./features/DefaultFeature');
      return DefaultFeature;
  }
}
```

### 3. Условная загрузка в зависимости от окружения
```typescript
// Загрузка платформо-специфичных модулей
async function loadPlatformSpecific() {
  if (typeof window !== 'undefined') {
    // Клиентский код
    const { BrowserAPI } = await import('./platform/BrowserAPI');
    return BrowserAPI;
  } else {
    // Серверный код
    const { ServerAPI } = await import('./platform/ServerAPI');
    return ServerAPI;
  }
}
```

## Современные паттерны

### 1. Conditional Exports
```json
{
  "name": "my-package",
  "version": "1.0.0",
  "type": "module",
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
  "main": "./dist/cjs/index.cjs",
  "module": "./dist/esm/index.js",
  "types": "./dist/types/index.d.ts"
}
```

### 2. Barrel optimization
```typescript
// src/index.ts - основной barrel
export * from './components';
export * from './services'; 
export * from './utils';

// Ленивый barrel (для улучшения tree-shaking)
export { Button } from './components/Button';
export { UserService } from './services/UserService';
export { validateEmail } from './utils/validation';
```

### 3. Module Federation (для микросервисов)
```typescript
// webpack.config.js
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host_app',
      remotes: {
        user_app: 'user_app@http://localhost:3001/remoteEntry.js'
      },
      shared: ['typescript']
    })
  ]
};
```

## Лучшие практики

1. **Используйте ES Modules как стандарт** для новых проектов
2. **Избегайте циклических зависимостей** - реорганизуйте код при необходимости
3. **Используйте barrel exports** для упрощения импортов
4. **Настройте baseUrl и paths** для удобства работы с длинными путями
5. **Используйте типо-безопасные алиасы** - особенно в крупных проектах
6. **Ограничьте число re-exports** - для сохранения ясности зависимостей
7. **Используйте динамические импорты** для оптимизации загрузки

## Заключение

Системы модулей TypeScript обеспечивают:

1. **Организацию кода** в логические единицы
2. **Управление зависимостями** между компонентами
3. **Повторное использование кода** через импорты/экспорты  
4. **Оптимизацию сборки** через tree-shaking и lazy loading
5. **Совместимость с разными средами** выполнения

Правильное понимание систем модулей критично для построения масштабируемых и поддерживаемых TypeScript приложений.

## Связь с другими концепциями
- [[../compilation/bundling]] - сборка модулей
- [[../ecosystem/build-tools]] - инструменты для работы с модулями
- [[../type-system/type-compatibility]] - совместимость типов между модулями
- [[../testing/module-testing]] - тестирование модульной архитектуры
- [[../deployment/modular-deployment]] - деплой модульных приложений
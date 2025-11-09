# TypeScript Modules

Система модулей TypeScript позволяет организовать код в отдельные файлы и пространства имен, обеспечивая изоляцию и переиспользуемость компонентов. TypeScript поддерживает стандарт ES Modules и обеспечивает совместимость с CommonJS, AMD и другими системами модулей.

## Основные концепции

### [ES Modules Syntax](es-modules-syntax.md)
Синтаксис import и export в стиле ES2015, статические и динамические импорты, переэкспорт.

### [Exports](exports.md)
Дефолтные и именованные экспорты, их использование и особенности.

### [Module Resolution](resolution.md)
Как TypeScript ищет и разрешает модули, разрешение относительных и абсолютных путей.

## Архитектурная схема модулей

```
                    ┌─────────────────────────────────────────────────────────┐
                    │              TypeScript Modules                         │
                    └─────────────────────────┬───────────────────────────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────────────────────────────┐
        ▼                                     ▼                                                           ▼
  ┌─────────────────┐               ┌──────────────────────────┐                                    ┌─────────────────┐
  │ ES Modules      │               │      Module Resolution   │                                    │  Module         │
  │                 │               │                          │                                    │  Compatibility  │
  │ - Import/Export │◄──────────────┤ - Relative Resolution    │───────────────────────────────────►│                 │
  │ - Dynamic       │               │ - Absolute Resolution    │                                    │ - ES Modules    │
  │   Imports       │               │ - Path Mapping           │                                    │ - CommonJS      │
  │ - Re-exports    │               │ - baseUrl Configuration  │                                    │ - AMD           │
  │ - Type Exports  │               │ - Root Directories       │                                    │ - UMD           │
  │ - Namespace     │               │ - Conditional Resolution │                                    │ - SystemJS      │
  │   Imports       │               │ - Performance            │                                    └─────────────────┘
  └─────────────────┘               └─────────────────┬────────┘
        │                                             │
        │                           ┌─────────────────▼─────────────────┐
        │                           │      Module Declaration         │
        │                           │                                 │
        │                           │ - Ambient Modules               │
        │                           │ - Global Augmentation           │
        │                           │ - Module Augmentation           │
        │                           │ - Declaration Merging           │
        │                           │ - Type Definitions              │
        └───────────────────────────┤ - Module Scoping                │
                                    └─────────────────────────────────┘
```

## Основные принципы модульной системы

### 1. Файловая система как система модулей
Каждый файл TypeScript считается модулем, если он содержит import или export. Файлы без них считаются частью глобального пространства имен.

```typescript
// math.ts - это модуль потому что содержит export
export function add(a: number, b: number): number {
  return a + b;
}

// global.ts - это не модуль (не содержит import/export)
// function globalFunction() { return "I'm global"; } // находится в глобальном пространстве
```

### 2. Статическое разрешение
ES Modules разрешаются статически во время компиляции, что позволяет:

- Обнаруживать ошибки импорта/экспорта до выполнения
- Оптимизировать bundle через tree-shaking
- Создавать эффективные системы сборки

```typescript
// Статический импорт - проверяется компилятором
import { specificFunction } from './some-module'; // Должна существовать

// Динамический импорт - разрешается во время выполнения
const module = await import('./dynamic-module'); // Проверяется в runtime
```

## Типы модулей

### ES Modules (рекомендуемые)
```typescript
// user.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export class UserService {
  private users: User[] = [];
  
  addUser(user: User): void {
    this.users.push(user);
  }
  
  getUserById(id: number): User | undefined {
    return this.users.find(u => u.id === id);
  }
}

// Использование
import { UserService, User } from './user';

const userService = new UserService();
const user: User = { id: 1, name: "Alice", email: "alice@example.com" };
userService.addUser(user);
```

### Совместимость с CommonJS
```javascript
// commonjs-style.js
function calculate(a, b) {
  return a + b;
}

module.exports = {
  calculate: calculate,
  VERSION: '1.0.0'
};

// В TypeScript - можно импортировать через разные синтаксисы
import * as CommonJSModule from './commonjs-style';  // Импорт как namespace
import { calculate, VERSION } from './commonjs-style'; // Деструктурирующий импорт
const { calculate, VERSION } = require('./commonjs-style'); // CommonJS стиль (в JS)
```

## Примеры использования

### Простой модуль с экспортом
```typescript
// utils.ts
export const add = (a: number, b: number): number => a + b;
export const subtract = (a: number, b: number): number => a - b;

export default class Calculator {
  static version = '1.0.0';
  
  static multiply(a: number, b: number): number {
    return a * b;
  }
}

// main.ts
import Calculator, { add, subtract } from './utils';
import * as MathUtils from './utils';

const result1 = add(2, 3); // 5
const calculator = new Calculator();
const result2 = Calculator.multiply(4, 5); // 20
```

### Динамический импорт для ленивой загрузки
```typescript
// heavy-module.ts
export class HeavyClass {
  doHeavyWork() {
    // тяжелые вычисления
    return "Heavy work completed";
  }
}

// main.ts
async function useHeavyModule() {
  // Ленивая загрузка модуля
  const { HeavyClass } = await import('./heavy-module');
  const heavyInstance = new HeavyClass();
  return heavyInstance.doHeavyWork();
}
```

## Модульные системы

### ES Modules vs CommonJS
| Feature | ES Modules | CommonJS |
|---------|------------|----------|
| Import | `import { x } from 'module'` | `const { x } = require('module')` |
| Export | `export { x }` | `module.exports = { x }` |
| Loading | Static, at top level | Dynamic, anywhere |
| Top-level await | ✅ (Node 14.8+) | ❌ |
| Tree shaking | ✅ | ❌ |

### Совместимость между системами
```json
// tsconfig.json для совместимости
{
  "compilerOptions": {
    "module": "ES2020",           // Источник
    "moduleResolution": "node",   // Резолюция как в Node.js
    "esModuleInterop": true,      // Совместимость ES/CommonJS
    "allowSyntheticDefaultImports": true
  }
}
```

## Практические паттерны

### Re-exports для организации API
```typescript
// api/index.ts
export { UserService } from './user-service';
export { AuthService } from './auth-service'; 
export { HttpService } from './http-service';
export type { User } from './types/user';
export type { AuthToken } from './types/auth';

// В другом файле
import { UserService, AuthService, type User } from './api';
```

### Условная загрузка модулей
```typescript
// platform-specific.ts
async function loadPlatformSpecific() {
  if (typeof window !== 'undefined') {
    // Загрузка клиентского кода
    const { BrowserService } = await import('./browser-service');
    return new BrowserService();
  } else {
    // Загрузка серверного кода
    const { ServerService } = await import('./server-service');
    return new ServerService();
  }
}
```

### Ленивая загрузка компонентов
```typescript
// component-loader.ts
async function loadComponent(type: string) {
  switch (type) {
    case 'chart':
      const { ChartComponent } = await import('./components/chart');
      return ChartComponent;
    case 'table':
      const { TableComponent } = await import('./components/table');
      return TableComponent;
    default:
      const { DefaultComponent } = await import('./components/default');
      return DefaultComponent;
  }
}
```

## Лучшие практики

### 1. Структурирование экспорта
```typescript
// features/index.ts - центральный экспорт
export { default as FeatureA } from './feature-a';
export { default as FeatureB } from './feature-b';
export { utilityFunction } from './utility-function';
export type { FeatureAType, FeatureBType } from './types';

// components/index.ts - переэкспорт компонентов
export { Button } from './button/button.component';
export { Input } from './input/input.component';
export { Modal } from './modal/modal.component';
```

### 2. Использование type-only экспорта
```typescript
// Типы и значения по-разному экспортируются
import type { SomeInterface } from './types';  // Только типы
import { SomeValue, SomeClass } from './values'; // Только значения

// Или в одном импорте
import { type SomeInterface, SomeValue } from './combined'; // Разные типы импортов
```

### 3. Использование алиасов для длинных путей
```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    }
  }
}
```

```typescript
// Вместо длинных относительных импортов
// import { Button } from '../../../../components/button/button.component';

// Использовать короткие абсолютные
import { Button } from '@components/button/button.component';
```

## Ошибки и подводные камни

### Частые ошибки
```typescript
// 1. Циклические зависимости
// a.ts -> b.ts -> a.ts (цикл!)

// 2. Неправильные пути импорта
// import { something } from './nonexistent-file'; // Ошибка разрешения

// 3. Смешивание систем модулей без настройки
// import в одном файле, require в другом - требует правильную настройку компиляции
```

## Совместимость с инструментами сборки

### Webpack и module resolution
```javascript
// webpack.config.js
module.exports = {
  resolve: {
    extensions: ['.ts', '.tsx', '.js'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components')
    }
  }
};
```

### TypeScript path mapping
```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@shared/*": ["../shared/*"]
    }
  }
}
```

## Связь с другими концепциями
- [[../type-system/type-compatibility]] - совместимость типов между модулями
- [[../declaration-files/index]] - создание файлов объявлений для модулей
- [[../compilation/configuration]] - настройка компиляции модулей
- [[../ecosystem/tools]] - инструменты для работы с модулями
- [[../runtime/javascript-interop]] - взаимодействие с JavaScript модулями
- [[advanced]] - продвинутые паттерны модулей
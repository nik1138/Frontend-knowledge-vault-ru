# Module Resolution

Разрешение модулей (Module Resolution) в TypeScript - это процесс, по которому компилятор или среда выполнения определяет, какой файл или библиотека соответствует оператору import или require. Понимание этого процесса критично для правильной настройки проектов и предотвращения проблем с зависимостями.

## Базовые концепции

### Типы импортов
```typescript
// Относительные импорты (начинаются с . или ..)
import { Component } from './Component';     // из текущей директории
import { Helper } from '../utils/Helper';    // из родительской директории
import { Utility } from '../../shared/Utility'; // из 2 директорий выше

// Абсолютные импорты (все остальные)
import { NgModule } from '@angular/core';    // из node_modules
import { Config } from 'config';              // из node_modules (если установлен)
import { MyModule } from '@/src/MyModule';    // с использованием baseUrl/paths
import { Service } from 'my-lib/Service';     // из пакета my-lib
```

## Стратегии разрешения модулей

### Node.js стратегия (по умолчанию)
```json
{
  "compilerOptions": {
    "moduleResolution": "node"
  }
}
```

#### Алгоритм разрешения Node.js стиля:

Для `import { thing } from './module'`:

1. Проверить `./module.ts`
2. Проверить `./module.tsx`  
3. Проверить `./module.d.ts`
4. Проверить `./module/index.ts`
5. Проверить `./module/index.tsx`
6. Проверить `./module/index.d.ts`
7. Проверить `./module/package.json` (см. поле "types" или "main")

Для `import { thing } from 'module'`:

1. Проверить в текущей директории: `./node_modules/module.ts`
2. Проверить `./node_modules/module.tsx`
3. Проверить `./node_modules/module.d.ts`
4. Проверить `./node_modules/module/package.json`
5. Проверить `./node_modules/module/index.ts`
6. Продолжить вверх по дереву директорий до корня файловой системы:
   - `../node_modules/module.ts`
   - `../../node_modules/module.ts`
   - и т.д.

### Пример разрешения
```typescript
// src/users/UserService.ts
import { API } from './api';           // ищет src/users/api.ts
import { Config } from '../config';    // ищет src/config.ts
import { Logger } from 'winston';      // ищет node_modules/winston/index.js или package.json
import { Util } from '@utils/helpers'; // разрешается через baseUrl/paths (смотри ниже)
```

### Classic (устаревшая) стратегия
```json
{
  "compilerOptions": {
    "moduleResolution": "classic"
  }
}
```
Использовалась в TypeScript до версии 1.6, сейчас не рекомендуется. Относительные импорты ищутся относительно позиции файла, абсолютные импорты ищутся в подкаталогах от корня проекта.

## Настройка разрешения модулей

### baseUrl
```json
{
  "compilerOptions": {
    "baseUrl": ".",        // базовый путь для некорректных (absolute) модулей
  }
}
```

```typescript
// С baseUrl: "."
import { UserService } from 'src/services/UserService';  // разрешается как ./src/services/UserService
import { Config } from 'config/app';                      // разрешается как ./config/app
```

### Paths mapping
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],                                    // @/component -> src/component
      "@components/*": ["src/components/*"],               // @components/Button -> src/components/Button
      "@utils/*": ["src/utils/*"],                         // @utils/helper -> src/utils/helper
      "@shared": ["src/shared/index"],                     // @shared -> src/shared/index.ts
      "@shared/*": ["src/shared/*"],                       // @shared/types -> src/shared/types
      "my-library": ["node_modules/my-library/dist/index"], // переопределение пути к библиотеке
      "@/*": ["generated/*", "src/*"]                      // множественные пути для одного префикса
    }
  }
}
```

```typescript
// Использование path mappings
import { User } from '@/models/User';           // разрешается в src/models/User
import { Button } from '@components/Button';    // разрешается в src/components/Button
import { formatDate } from '@utils/date';       // разрешается в src/utils/date
import { SharedTypes } from '@shared';          // разрешается в src/shared/index
```

### RootDirs
```json
{
  "compilerOptions": {
    "rootDirs": [
      "src",           // основной исходный код
      "generated",     // сгенерированный код
      "custom-modules" // кастомные модули
    ]
  }
}
```

Это позволяет TypeScript воспринимать файлы из разных директорий как будто они находятся в одной директории для целей разрешения модулей.

```typescript
// src/components/MyComponent.ts
import { Config } from 'config/AppConfig';  // может быть в generated/config/AppConfig

// TypeScript будет искать в:
// - src/config/AppConfig.ts
// - generated/config/AppConfig.ts  
// - custom-modules/config/AppConfig.ts
```

## Виртуальные директории и алиасы

### Вложенные алиасы
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@api/*": ["src/services/api/*"],
      "@api-v1/*": ["src/services/api/v1/*"],
      "@api-v2/*": ["src/services/api/v2/*"],
      
      "@domain/*": ["src/domain/*"],
      "@domain/entities/*": ["src/domain/entities/*"],
      "@domain/usecases/*": ["src/domain/usecases/*"],
      "@domain/repositories/*": ["src/domain/repositories/*"]
    }
  }
}
```

### Условные пути
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@platform/*": [
        "src/platform/web/*",     // для веба
        "src/platform/node/*"     // для сервера
      ]
    }
  }
}
```

## Современные возможности

### Conditional Exports (package.json)
```json
{
  "name": "my-library",
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
    },
    "./legacy": {
      "require": "./dist/legacy.cjs",
      "import": "./dist/legacy.mjs"
    }
  },
  "main": "./dist/cjs/index.cjs",
  "module": "./dist/esm/index.mjs",
  "types": "./dist/types/index.d.ts"
}
```

### Import Maps (браузерные алиасы)
```html
<!-- index.html -->
<script type="importmap">
{
  "imports": {
    "my-library": "/node_modules/my-library/dist/index.js",
    "@utils": "/src/utils/index.js",
    "@components": "/src/components/index.js"
  }
}
</script>

<script type="module">
  import { helper } from 'my-library/utils';
  import { Component } from '@components/Component';
</script>
```

## Практические примеры

### Многоуровневая архитектура
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@root/*": ["./*"],
      "@src/*": ["src/*"],
      "@ui/*": ["src/ui/*"],
      "@api/*": ["src/api/*"],
      "@config/*": ["src/config/*"],
      "@constants/*": ["src/constants/*"],
      "@types/*": ["src/types/*"],
      
      // Feature-based пути
      "@features/*": ["src/features/*"],
      "@features/user/*": ["src/features/user/*"],
      "@features/orders/*": ["src/features/orders/*"],
      
      // Shared путь
      "@shared/*": ["src/shared/*"],
      "@shared/ui/*": ["src/shared/ui/*"],
      "@shared/utils/*": ["src/shared/utils/*"],
      "@shared/types/*": ["src/shared/types/*"]
    }
  }
}
```

```typescript
// src/features/user/UserService.ts
import { API } from '@api/client';          // src/api/client.ts
import { Logger } from '@shared/utils/logger'; // src/shared/utils/logger.ts
import { User, UserUpdate } from '@features/user/types'; // src/features/user/types.ts
import { Config } from '@config/app';      // src/config/app.ts
import { Button } from '@shared/ui/button'; // src/shared/ui/button.ts
```

### Разрешение для разных целей
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext", 
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      // Разные версии для разных целей
      "@environment/*": [
        "src/environments/environment.production.ts",
        "src/environments/environment.development.ts"
      ]
    }
  }
}
```

### Импорт JSON и других файлов
```json
{
  "compilerOptions": {
    "resolveJsonModule": true,      // разрешение JSON модулей
    "allowArbitraryExtensions": true, // произвольные расширения
    "moduleResolution": "node"
  }
}
```

```typescript
// Импорт JSON
import config from './config.json';
import { version, name } from './package.json';

// Использование
console.log(`App: ${name} v${version}`);
console.log(`Environment: ${config.environment}`);
```

## Проблемы и решения

### Циклические зависимости
```typescript
// ПЛОХО: циклическая зависимость
// A.ts
import { B } from './B';
export class A { 
  useB(): void { new B(); }
}

// B.ts
import { A } from './A';
export class B { 
  useA(): void { new A(); }
}

// РЕШЕНИЕ: вынос общего кода
// Common.ts
export interface SharedType { /* */ }

// A.ts
import type { SharedType } from './Common';
export class A { /* */ }

// B.ts
import type { SharedType } from './Common';
import { A } from './A'; // не циклически, так как A не импортирует SharedType
export class B { /* */ }
```

### Проблемы с baseUrl и путями
```json
{
  "compilerOptions": {
    "baseUrl": ".", 
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

```typescript
// Файл в ./test/utils/TestHelper.ts
import { UserService } from '@/services/UserService'; // ОШИБКА: не найдет

// Решение: использовать более конкретные пути
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@src/*": ["src/*"],
      "@test/*": ["test/*"]  // для тестов
    }
  }
}
```

### Совместимость с разными системами сборки
```json
// Для Webpack
{
  "compilerOptions": {
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}

// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    }
  }
};
```

```json
// Для Vite
{
  "compilerOptions": {
    "moduleResolution": "bundler",  // новая опция для Vite
    "allowImportingTsExtensions": true
  }
}

// vite.config.ts
export default {
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    }
  }
};
```

## Лучшие практики

### 1. Логическая структура путей
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      // Плохо: хаотичные алиасы
      // "a": ["src/utils"],
      // "b": ["src/components"],
      
      // Хорошо: логическая структура
      "@/*": ["src/*"],              
      "@assets/*": ["src/assets/*"],     // статические ресурсы
      "@components/*": ["src/components/*"], // UI компоненты
      "@services/*": ["src/services/*"],   // бизнес-логика
      "@models/*": ["src/models/*"],      // доменные модели
      "@utils/*": ["src/utils/*"],        // утилиты
      "@types/*": ["src/types/*"],        // типы
      "@tests/*": ["__tests__/*"]         // тесты
    }
  }
}
```

### 2. Консистентность
```typescript
// Всегда используйте один стиль импортов в проекте
// Консистентные алиасы:
import { UserService } from '@services/UserService';
import { UserComponent } from '@components/UserComponent';
import { User } from '@models/User';
import { userUtils } from '@utils/user';
import { UserType } from '@types/UserType';

// Избегайте смешивания стилей:
// import { UserService } from './services/UserService'; // неconsistently с @services/*
// import { UserService } from '../services/UserService'; // неconsistently с @services/*
```

### 3. Типобезопасные алиасы
```typescript
// Использование declaration files для алиасов
declare module '@api/*' {
  const content: any;
  export default content;
}

// Или более конкретно:
declare module '@api/client' {
  export const apiClient: { get: (url: string) => Promise<any> };
}
```

## Современные тенденции

### Module Path Bundling
```json
// Использование для tree-shaking и оптимизации
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      // Раздельные пути для разных версий
      "@lib": ["./src/index.ts"],
      "@lib/core": ["./src/core/index.ts"],
      "@lib/utils": ["./src/utils/index.ts"]
    }
  }
}
```

### Bundle Analysis
```typescript
// Структура, оптимизированная для анализа бандла
import { UserService } from '@services/user/UserService';
import { OrderService } from '@services/order/OrderService';
// Эти импорты позволяют инструментам сборки определить, что загружать по частям
```

### Import Assertions
```typescript
// Совместимость с современными возможностями
import data from './data.json' assert { type: 'json' };
import wasm from './module.wasm' assert { type: 'wasm' };
```

## Отладка разрешения модулей

### Инструменты отладки
```bash
# Проверка разрешения модулей TypeScript
tsc --traceResolution

# В IDE отображение информации о разрешении
# TypeScript Plugin в VSCode показывает пути разрешения
```

### Пример трассировки
```typescript
// myFile.ts
import { something } from 'someModule';

// При запуске tsc --traceResolution:
// ======== Resolving module 'someModule' from 'myFile.ts'. ========
// Explicitly specified module resolution kind: 'NodeJs'.
// 'moduleResolution' is 'node'.
// Loading module 'someModule' from 'node_modules'.
// File '/node_modules/someModule/package.json' exists according to earlier cached lookups.
// File '/node_modules/someModule/index.js' exists according to earlier cached lookups.
```

## Связь с другими системами

### Node.js ESM
```json
// Совместимость с Node.js ES Modules
{
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src/**/*"]
}
```

```javascript
// package.json
{
  "type": "module",  // указывает, что это ES Module проект
  "exports": "./dist/index.js"
}
```

### Системы сборки
```json
// Совместимость с разными системами
{
  "compilerOptions": {
    // Для Webpack
    "moduleResolution": "node",
    
    // Для Vite (новее)
    // "moduleResolution": "bundler",
    
    // Для Browser (старые системы)
    // "moduleResolution": "classic",
    
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "rootDir": "src",
    "outDir": "dist"
  }
}
```

## Заключение

Разрешение модулей в TypeScript:

1. **Критично для организации кода** - определяет, как компоненты находят друг друга
2. **Влияет на производительность сборки** - неправильная настройка может замедлить компиляцию
3. **Влияет на совместимость** - особенно с разными системами сборки
4. **Требует внимания к деталям** - малейшая ошибка в путях может вызвать проблемы

Правильная настройка разрешения модулей обеспечивает чистую архитектуру, быструю разработку и легкую миграцию между инструментами сборки.

## Связь с другими концепциями
- [[module-systems]] - системы модулей и их особенности
- [[build-tools/typescript-compiler]] - компилятор и разрешение модулей  
- [[build-tools/webpack]] - Webpack и разрешение
- [[build-tools/vite]] - Vite и современное разрешение
- [[compatibility/module-compatibility]] - совместимость разных систем модулей
- [[deployment/bundling]] - влияние на упаковку приложений
# Module Resolution

Разрешение модулей в TypeScript - это процесс, по которому компилятор определяет, какой файл или файлы соответствуют импорту в исходном коде. Это критическая часть системы модулей, которая обеспечивает правильную работу импортов и экспорта.

## Основные концепции разрешения модулей

### Алгоритмы разрешения
TypeScript поддерживает два основных алгоритма разрешения модулей:

1. **Classic** - оригинальный алгоритм TypeScript (до версии 1.6)
2. **Node** - алгоритм, совместимый с Node.js

### Node-style разрешение (по умолчанию для CommonJS/UMD)
```typescript
// Импорт в TypeScript
import { Component } from './component';

// Алгоритм поиска для './component':
// 1. ./component.ts
// 2. ./component.tsx  
// 3. ./component.d.ts
// 4. ./component/index.ts
// 5. ./component/index.tsx
// 6. ./component/index.d.ts

// Импорт файла с расширением
import { utils } from './utils.js';

// Алгоритм поиска:
// 1. ./utils.js.ts
// 2. ./utils.js.tsx
// 3. ./utils.js.d.ts
// 4. ./utils.js/index.ts
// 5. ./utils.js/index.tsx
// 6. ./utils.js/index.d.ts
```

### Absolute vs Relative imports
```typescript
// Относительные импорты (начинаются с ./ или ../)
import { UserService } from './services/user-service';    // Относительно текущего файла
import { constants } from '../config/constants';          // На уровень выше
import { helpers } from '../../utils/helpers';            // На два уровня выше

// Абсолютные импорты (начинаются с корня проекта)
import { ApiService } from 'services/api-service';        // Стартует с baseUrl или paths
import { config } from 'config/app-config';               // Через path mapping
import { AuthModule } from '@modules/auth';               // Через alias в tsconfig
```

## Конфигурация разрешения модулей

### tsconfig.json настройки
```json
{
  "compilerOptions": {
    // Алгоритм разрешения модулей
    "moduleResolution": "node",      // "node" | "classic"
    
    // Корневая директория для абсолютных импортов
    "baseUrl": ".",                  // "." для корня проекта
    
    // Алиасы для директорий
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@assets/*": ["src/assets/*"],
      "@types": ["src/types/index"],  // Для одного файла
      "my-library": ["node_modules/my-library/src/index"] // Переопределение библиотеки
    },
    
    // Корневые директории для модулей
    "rootDirs": [
      "src",
      "generated"
    ],
    
    // Директории для поиска модулей
    "typeRoots": ["./node_modules/@types", "./custom-types"],
    
    // Список библиотек для поиска типов
    "types": ["node", "jest", "@types/node"]
  },
  
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Примеры настройки

### Простая конфигурация
```json
{
  "compilerOptions": {
    "moduleResolution": "node",
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@/utils/*": ["helpers/*"]
    }
  },
  "include": ["src/**/*"]
}
```

### Сложная архитектура с монорепозиторием
```json
{
  "compilerOptions": {
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@shared/*": ["packages/shared/src/*"],
      "@frontend/*": ["packages/frontend/src/*"],
      "@backend/*": ["packages/backend/src/*"],
      "@test/*": ["__tests__/*"],
      "@env": ["environments/environment"]
    }
  },
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/frontend" },
    { "path": "./packages/backend" }
  ]
}
```

## Практические примеры разрешения

### 1. Простое приложение
```
project/
├── tsconfig.json
├── src/
│   ├── main.ts
│   ├── components/
│   │   ├── button/
│   │   │   └── button.ts
│   │   └── index.ts
│   ├── services/
│   │   ├── api.ts
│   │   └── auth.ts
│   └── utils/
│       ├── helpers.ts
│       └── types.ts
```

```typescript
// src/main.ts
import { Button } from './components/button/button';  // относительный
import { Button } from '@components/button/button';    // абсолютный с baseUrl
import { Button } from '@components';                 // через index.ts

import { ApiService } from 'services/api';             // через baseUrl
import { format } from 'utils/helpers';                // через baseUrl
```

### 2. Использование rootDirs
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "rootDirs": [
      "src",
      "build/generated-sources"
    ]
  }
}
```

```typescript
// Файл в src/components.ts
import { GeneratedUtil } from 'utils/generated';  // На самом деле находится в build/generated-sources/utils/generated.ts
```

## Типы разрешения модулей

### File-based modules
```typescript
// component.ts
export class Button {
  constructor(public text: string) {}
  render() { return `<button>${this.text}</button>`; }
}

// В другом файле
import { Button } from './component';  // Разрешается в компонент.ts
```

### Folder-based modules
```typescript
// components/index.ts
export { Button } from './button';
export { Card } from './card';

// В другом файле
import { Button } from './components';  // Разрешается через index.ts
```

## Особенности разрешения

### Типы vs значения
```typescript
// types.ts
export interface User {
  name: string;
  email: string;
}

export function createUser(name: string, email: string): User {
  return { name, email };
}

// Использование
import type { User } from './types';           // Только тип
import { createUser } from './types';          // Только значение
import { type User, createUser } from './types'; // И тип, и значение из одного импорта
```

### Ambient modules
```typescript
// types/ambient.d.ts
declare module 'custom-library' {
  export function customFunction(): string;
}

declare module '*.json' {
  const value: any;
  export default value;
}

declare module '*.svg' {
  const content: string;
  export default content;
}

// Использование
import customLib from 'custom-library';  // Разрешается через ambient declaration
import data from './config.json';       // Разрешается как JSON
import icon from './logo.svg';          // Разрешается как SVG
```

## Алгоритм разрешения пошагово

### Node-style resolution (относительные пути)
Для `import { a } from "./moduleA"` из файла `/root/src/folder/file.ts`:

```
1. /root/src/folder/moduleA.ts
2. /root/src/folder/moduleA.tsx
3. /root/src/folder/moduleA.d.ts
4. /root/src/folder/moduleA/package.json (main field)
5. /root/src/folder/moduleA/index.ts
6. /root/src/folder/moduleA/index.tsx
7. /root/src/folder/moduleA/index.d.ts
```

### Node-style resolution (абсолютные пути)
Для `import { b } from "moduleB"` из файла `/root/src/folder/file.ts`:

```
// Поиск начинается с директории импортирующего файла и идет вверх по иерархии
1. /root/src/folder/node_modules/moduleB.ts
2. /root/src/folder/node_modules/moduleB.tsx
3. /root/src/folder/node_modules/moduleB.d.ts
4. /root/src/folder/node_modules/moduleB/package.json
5. /root/src/folder/node_modules/moduleB/index.ts
6. /root/src/node_modules/moduleB.ts  // проверка в родительской директории
7. /root/node_modules/moduleB.ts     // проверка в корне проекта
8. /node_modules/moduleB.ts          // проверка в корне файловой системы
```

## Path mapping examples

### Использование path mappings
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@shared/*": ["shared/*"],
      "@ui/*": ["src/components/ui/*"],
      "@hooks": ["src/hooks/index"],
      "@hooks/*": ["src/hooks/*"],
      "lib": ["node_modules/some-library/dist/index"],
      "lib/*": ["node_modules/some-library/dist/*"]
    }
  }
}
```

```typescript
// Использование маппингов
import { Button } from '@ui/button';           // преобразуется в src/components/ui/button
import { useAuth } from '@hooks/use-auth';     // преобразуется в src/hooks/use-auth
import { SharedUtil } from '@shared/utils';    // преобразуется в shared/utils
import { AuthProvider } from '@hooks';         // преобразуется в src/hooks/index
import { SomeUtility } from 'lib/utility';     // преобразуется в some-library/dist/utility
```

## Runtime vs Compile-time resolution

### Differences in resolution
```typescript
// Compile-time (TS) и Runtime (JS) могут иметь разные результаты
import { config } from '@env';  // TS разрешает через path mapping

// В runtime это будет:
// const { config } = require('./environments/environment'); // или где бы ни был на самом деле

// Для проверки соответствия используется:
// - tsconfig-paths для Node.js во время разработки
// - соответствующие настройки сборщика (webpack, rollup) для production
```

### Configuration for bundlers
```javascript
// webpack.config.js
const TsconfigPathsPlugin = require('tsconfig-paths-webpack-plugin');

module.exports = {
  resolve: {
    plugins: [new TsconfigPathsPlugin()],
    extensions: ['.ts', '.tsx', '.js', '.jsx']
  }
};
```

## Troubleshooting

### Common resolution issues

#### 1. Cannot find module
```typescript
// Причина: неправильное разрешение пути
import { something } from './non-existent-file';

// Решение: проверить наличие файла или настройки tsconfig
```

#### 2. Different paths at runtime
```typescript
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"]
    }
  }
}

// При использовании с Node.js без ts-node/tsx нужна настройка
// чтобы runtime мог найти файлы по тем же путям
```

#### 3. Module augmentation issues
```typescript
// types/augment.d.ts
declare module 'existing-library' {
  interface ExtendedInterface {
    extraProperty: string;
  }
}

// Если файл с декларацией не включён в компиляцию, augmentation не будет работать
// Решение: добавить в tsconfig.json
{
  "include": ["**/*.d.ts", "types/**/*.ts"]
}
```

## Best Practices

### 1. Consistent import style
```typescript
// Выберите один стиль и придерживайтесь его
// ПЛОХО: смешанные стили
import { A } from './a';
import { B } from 'components/b';
import { C } from '@/utils/c';

// ХОРОШО: последовательность
import { A } from './a';                    // относительные
import { B } from '../components/b';        // относительные для соседних
import { C } from '@/utils/c';              // абсолютные для корня
import { D } from '@shared/d';              // абсолютные для общих
```

### 2. Proper path configuration
```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],                           // для основных импортов
      "@test/*": ["../__tests__/*"],         // для тестов
      "@types": ["types/index"]              // для основного тип-файла
    }
  }
}
```

### 3. Alias naming conventions
```typescript
// ПЛОХО: непоследовательные названия
import { util } from '@utils';
import { helper } from 'helpers';
import { lib } from 'libs';

// ХОРОШО: последовательные и осмысленные
import { util } from '@/utils';
import { helper } from '@/helpers'; 
import { lib } from '@/libraries';
```

## Integration with build tools

### TypeScript + Webpack
```javascript
// webpack.config.js
const path = require('path');
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');

module.exports = {
  resolve: {
    extensions: ['.ts', '.tsx', '.js'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components')
    }
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    new ForkTsCheckerWebpackPlugin()
  ]
};
```

### TypeScript + Jest
```javascript
// jest.config.js
const { pathsToModuleNameMapper } = require('ts-jest');

const { compilerOptions } = require('./tsconfig');

module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  moduleNameMapper: pathsToModuleNameMapper(compilerOptions.paths, { 
    prefix: '<rootDir>/'
  })
};
```

## Advanced patterns

### Conditional resolution
```json
{
  "compilerOptions": {
    "paths": {
      "@environment": ["environments/environment.prod"],
      "@environment": ["environments/environment.dev"]
    }
  }
}
```

### Platform-specific resolution
```json
{
  "compilerOptions": {
    "paths": {
      "@platform/*": ["platforms/web/*"],
      "@platform/*": ["platforms/mobile/*"]
    }
  }
}
```

```typescript
// Платформо-специфичный импорт (настройки сборки определяют, какой файл будет использован)
import { PlatformService } from '@platform/service';
```

## Performance considerations

### Path resolution performance
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      // Меньше путей = быстрее разрешение
      "@/*": ["src/*"],                // 1 правило
      // Вместо:
      // "@/components/*": ["src/components/*"],
      // "@/utils/*": ["src/utils/*"], 
      // "@/services/*": ["src/services/*"]
    }
  }
}
```

## Security considerations

### Path traversal protection
```typescript
// TypeScript защищает от path traversal attacks на этапе компиляции
// Невозможно импортировать файлы за пределами разрешенных путей
// import { secret } from '../../../etc/passwd'; // Ошибка разрешения модуля
```

## Связь с другими концепциями
- [[../compilation/configuration]] - конфигурация компиляции
- [[../ecosystem/tools]] - инструменты сборки
- [[../compatibility/modules]] - совместимость модульных систем
- [[../declaration-files/publishing]] - публикация библиотек с правильным разрешением модулей
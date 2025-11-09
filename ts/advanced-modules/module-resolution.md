# Разрешение модулей в TypeScript

Разрешение модулей (Module Resolution) - это процесс, с помощью которого TypeScript ищет и находит файлы модулей, на которые ссылаются импорты в коде. Понимание этого процесса критически важно для правильной организации проекта и избежания ошибок при импорте.

## Содержание

1. [Основы разрешения модулей](#основы-разрешения-модулей)
2. [Стратегии разрешения](#стратегии-разрешения)
3. [Алгоритмы поиска](#алгоритмы-поиска)
4. [Псевдонимы путей](#псевдонимы-путей)
5. [Практические примеры](#практические-примеры)
6. [Оптимизация разрешения](#оптимизация-разрешения)
7. [Проблемы и решения](#проблемы-и-решения)

## Основы разрешения модулей

Разрешение модулей - это механизм, который определяет, какой файл должен быть загружен при встрече импорта. TypeScript поддерживает два основных режима разрешения модулей: `classic` и `node`.

### Базовый процесс разрешения

Когда TypeScript встречает импорт:

```typescript
import { something } from './module';
```

Он выполняет следующие шаги:
1. Определяет тип модуля (относительный или неотносительный)
2. Применяет выбранную стратегию разрешения
3. Ищет файл модуля по определенным правилам
4. Проверяет наличие файла объявления типов (`.d.ts`)

### Типы модулей

```typescript
// Относительные импорты (начинаются с ./, ../, или /)
import { helper } from './utils/helper';
import { config } from '../config';
import { global } from '/src/global';

// Неотносительные импорты (без ./, ../, /)
import { Component } from 'react';
import { express } from 'express';
import { myLibrary } from 'my-custom-library';
```

## Стратегии разрешения

### 1. Classic (устаревшая)

Classic стратегия использовалась в ранних версиях TypeScript и сейчас устарела:

```json
{
  "compilerOptions": {
    "moduleResolution": "classic"
  }
}
```

При classic стратегии:
- Для относительных импортов ищет файл напрямую
- Для неотносительных импортов ищет в `node_modules` текущей директории

### 2. Node (рекомендуемая)

Node стратегия эмулирует поведение Node.js:

```json
{
  "compilerOptions": {
    "moduleResolution": "node"
  }
}
```

При node стратегии:
- Следует алгоритму разрешения модулей Node.js
- Поддерживает `package.json` и поле `types`/`typings`
- Ищет файлы с расширениями `.ts`, `.tsx`, `.d.ts`

### 3. Node16 и NodeNext

Новые стратегии для поддержки ECMAScript modules:

```json
{
  "compilerOptions": {
    "moduleResolution": "node16" // или "nodenext"
  }
}
```

## Алгоритмы поиска

### Алгоритм Node.js

Для импорта `import { something } from 'module-name'`:

1. **Поиск в node_modules текущей директории**
   - `./node_modules/module-name/package.json` (проверка поля `types`/`typings`)
   - `./node_modules/module-name/index.d.ts`
   - `./node_modules/module-name/index.js`

2. **Поиск в родительских директориях**
   - Повторяется для каждой родительской директории до корня

3. **Поиск в глобальных node_modules**
   - В директории, указанной в `$NODE_PATH`

### Алгоритм для относительных импортов

Для импорта `import { something } from './path/to/module'`:

1. **Прямой поиск файла**
   - `./path/to/module.ts`
   - `./path/to/module.tsx`
   - `./path/to/module.d.ts`

2. **Поиск как директории**
   - `./path/to/module/index.ts`
   - `./path/to/module/index.tsx`
   - `./path/to/module/index.d.ts`

3. **Поиск с расширениями**
   - `./path/to/module.js`
   - `./path/to/module.jsx`

### Пример структуры поиска

```typescript
// Структура проекта:
// project/
//   src/
//     components/
//       Button/
//         index.ts
//         Button.tsx
//         types.d.ts
//     utils/
//       helpers.ts
//     index.ts
//   node_modules/
//     lodash/
//       package.json
//       index.d.ts
//     react/
//       index.d.ts

// Импорты в src/index.ts:
import { Button } from './components/Button'; // -> ./components/Button/index.ts
import { helper } from './utils/helpers'; // -> ./utils/helpers.ts
import _ from 'lodash'; // -> ./node_modules/lodash/index.d.ts
import React from 'react'; // -> ./node_modules/react/index.d.ts
```

## Псевдонимы путей

Псевдонимы путей позволяют создавать короткие имена для часто используемых директорий:

### Базовая конфигурация

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@assets/*": ["src/assets/*"],
      "@types/*": ["src/types/*"]
    }
  }
}
```

### Примеры использования

```typescript
// Без псевдонимов
import { Button } from '../../../components/Button/Button';
import { helper } from '../../../../utils/helpers';
import { User } from '../../../../../types/user';

// С псевдонимами
import { Button } from '@components/Button/Button';
import { helper } from '@utils/helpers';
import { User } from '@types/user';
```

### Расширенные псевдонимы

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      // Множественные пути
      "@shared/*": [
        "src/shared/*",
        "src/common/*",
        "node_modules/shared/*"
      ],
      
      // Абсолютные пути
      "@root/*": ["./*"],
      
      // Конкретные файлы
      "@config": ["src/config/index"],
      "@constants": ["src/constants/index"],
      
      // Группировка
      "@ui/*": ["src/components/ui/*"],
      "@layout/*": ["src/components/layout/*"],
      "@pages/*": ["src/components/pages/*"]
    }
  }
}
```

### Псевдонимы с webpack

Для корректной работы в runtime также нужно настроить сборщик:

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@assets': path.resolve(__dirname, 'src/assets')
    }
  }
};
```

## Практические примеры

### 1. Монорепозиторий с псевдонимами

```typescript
// Структура монорепозитория:
// monorepo/
//   packages/
//     core/
//       src/
//         index.ts
//         utils/
//           helpers.ts
//     ui/
//       src/
//         components/
//           Button/
//             Button.tsx
//             index.ts
//     api/
//       src/
//         client/
//           index.ts
//   tsconfig.json

// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@monorepo/*": ["packages/*"],
      "@monorepo/core/*": ["packages/core/src/*"],
      "@monorepo/ui/*": ["packages/ui/src/*"],
      "@monorepo/api/*": ["packages/api/src/*"]
    }
  }
}

// Использование в packages/ui/src/components/Button/Button.tsx
import { helper } from '@monorepo/core/utils/helpers';
import { ApiClient } from '@monorepo/api/client';
```

### 2. Многоуровневая структура

```typescript
// Структура проекта:
// project/
//   src/
//     app/
//       components/
//         Header/
//           Header.tsx
//         Footer/
//           Footer.tsx
//       pages/
//         Home/
//           Home.tsx
//     shared/
//       components/
//         Button/
//           Button.tsx
//       utils/
//         api.ts
//         validation.ts
//       types/
//         index.ts
//     features/
//       auth/
//         components/
//           LoginForm/
//             LoginForm.tsx
//         services/
//           authService.ts
//   tsconfig.json

// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@app/*": ["app/*"],
      "@app/components/*": ["app/components/*"],
      "@app/pages/*": ["app/pages/*"],
      "@shared/*": ["shared/*"],
      "@shared/components/*": ["shared/components/*"],
      "@shared/utils/*": ["shared/utils/*"],
      "@shared/types": ["shared/types"],
      "@features/*": ["features/*"],
      "@features/auth/*": ["features/auth/*"]
    }
  }
}

// Использование в src/app/pages/Home/Home.tsx
import { Header } from '@app/components/Header/Header';
import { Button } from '@shared/components/Button/Button';
import { api } from '@shared/utils/api';
import { LoginForm } from '@features/auth/components/LoginForm/LoginForm';
```

### 3. Условные импорты

```typescript
// Структура проекта:
// project/
//   src/
//     config/
//       development.ts
//       production.ts
//       index.ts
//     services/
//       api/
//         development.ts
//         production.ts
//         index.ts
//   tsconfig.json

// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@config": ["config/index"],
      "@config/env": [
        "config/development",
        "config/production"
      ],
      "@services/*": ["services/*"]
    }
  }
}

// src/config/index.ts
let config: any;

if (process.env.NODE_ENV === 'development') {
  config = require('@config/env/development').default;
} else {
  config = require('@config/env/production').default;
}

export default config;

// src/services/api/index.ts
import developmentApi from '@services/api/development';
import productionApi from '@services/api/production';

const api = process.env.NODE_ENV === 'development' 
  ? developmentApi 
  : productionApi;

export default api;
```

## Оптимизация разрешения

### 1. Настройка baseUrl и paths

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      // Оптимизированные псевдонимы
      "@/*": ["*"],
      "@components": ["components/index"],
      "@utils": ["utils/index"],
      "@types": ["types/index"]
    }
  }
}
```

### 2. Использование индексных файлов

```typescript
// Вместо множества импортов
// components/Button/Button.tsx
// components/Button/types.ts
// components/Button/styles.ts

// components/Button/index.ts
export { Button } from './Button';
export type { ButtonProps } from './types';
export { buttonStyles } from './styles';

// Теперь можно импортировать всё из одной точки
import { Button, type ButtonProps, buttonStyles } from '@components/Button';
```

### 3. Оптимизация структуры проекта

```typescript
// Хорошая структура
// src/
//   components/
//     ui/
//       Button/
//         index.ts
//         Button.tsx
//         types.ts
//     layout/
//       Header/
//         index.ts
//         Header.tsx
//   features/
//     auth/
//       components/
//         LoginForm/
//           index.ts
//           LoginForm.tsx
//       services/
//         index.ts
//         authService.ts
//   shared/
//     hooks/
//       index.ts
//       useApi.ts
//     utils/
//       index.ts
//       helpers.ts

// Плохая структура
// src/
//   components/
//     Button.tsx
//     Header.tsx
//     LoginForm.tsx
//     useApi.ts
//     helpers.ts
//   services/
//     authService.ts
```

### 4. Кэширование разрешения

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo"
  }
}
```

## Проблемы и решения

### 1. Ошибка "Cannot find module"

```bash
# Проблема: Модуль не найден
error TS2307: Cannot find module 'module-name' or its corresponding type declarations.

# Решения:
# 1. Проверить правильность пути
# 2. Убедиться, что файл существует
# 3. Проверить конфигурацию paths в tsconfig.json
# 4. Установить недостающие типы: npm install @types/module-name
```

### 2. Проблемы с псевдонимами

```json
// Проблема: Псевдонимы не работают
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@components/*": ["src/components/*"]
    }
  }
}

// Решение: Правильный путь
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@components/*": ["components/*"]
    }
  }
}
```

### 3. Конфликты путей

```json
// Проблема: Конфликтующие псевдонимы
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"] // Конфликт!
    }
  }
}

// Решение: Четкая иерархия
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@app/*": ["app/*"],
      "@shared/*": ["shared/*"],
      "@features/*": ["features/*"]
    }
  }
}
```

### 4. Проблемы с монорепозиториями

```json
// Проблема: Модули из других пакетов не находятся
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@myorg/*": ["packages/*"]
    }
  }
}

// Решение: Явные пути к каждому пакету
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@myorg/core": ["packages/core/src/index"],
      "@myorg/ui": ["packages/ui/src/index"],
      "@myorg/api": ["packages/api/src/index"]
    }
  }
}
```

### 5. Оптимизация производительности

```json
// Медленное разрешение модулей
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "*": ["node_modules/*", "src/types/*"] // Слишком общие пути
    }
  }
}

// Оптимизированная конфигурация
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@types/*": ["../types/*"] // Конкретные пути
    }
  }
}
```

## Инструменты для диагностики

### 1. Проверка конфигурации

```bash
# Проверка tsconfig.json
tsc --showConfig

# Проверка разрешения модулей
tsc --traceResolution
```

### 2. Анализ зависимостей

```bash
# Анализ циклических зависимостей
npx madge --circular src/

# Анализ зависимостей
npx dpdm src/

# Визуализация зависимостей
npx dependency-cruiser --output-type dot src/ | dot -T svg > dependencies.svg
```

### 3. Отладка импортов

```typescript
// debug-imports.ts
declare global {
  interface ImportMeta {
    resolve?(specifier: string): string;
  }
}

// Для отладки в runtime
async function debugImport(specifier: string) {
  try {
    const module = await import(specifier);
    console.log(`Successfully imported ${specifier}:`, Object.keys(module));
    return module;
  } catch (error) {
    console.error(`Failed to import ${specifier}:`, error);
    throw error;
  }
}
```

## Связи с другими концепциями

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Основы модульной системы
- [[ts/advanced-modules/ambient-modules|Внешние модули]] - Работа с внешними библиотеками
- [[ts/advanced-modules/bundling|Бандлинг модулей]] - Оптимизация для сборки
- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора
- [[ts/advanced-modules/code-splitting|Разделение кода]] - Разделение приложения на части

## Рекомендации по изучению

1. Практикуйтесь в настройке baseUrl и paths
2. Освойте алгоритмы разрешения модулей
3. Изучите оптимизацию структуры проекта
4. Практикуйтесь в диагностике проблем с импортами
5. Освойте работу с монорепозиториями
6. Изучите инструменты для анализа зависимостей
7. Практикуйтесь в создании масштабируемых архитектур

## Следующие шаги

- [[ts/advanced-modules/bundling|Бандлинг модулей]] - Оптимизация для сборки
- [[ts/advanced-modules/code-splitting|Разделение кода]] - Разделение приложения на части
- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора
- [[ts/advanced-modules/dynamic-imports|Динамические импорты]] - Динамическая загрузка модулей
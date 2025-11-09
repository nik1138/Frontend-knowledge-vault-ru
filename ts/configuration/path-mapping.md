# Сопоставление путей в TypeScript

Сопоставление путей (Path Mapping) - это механизм TypeScript, который позволяет создавать псевдонимы для часто используемых директорий и модулей. Это упрощает импорты, улучшает читаемость кода и делает структуру проекта более гибкой.

## Содержание

1. [Основы сопоставления путей](#основы-сопоставления-путей)
2. [Конфигурация baseUrl и paths](#конфигурация-baseurl-и-paths)
3. [Практические примеры](#практические-примеры)
4. [Интеграция с бандлерами](#интеграция-с-бандлерами)
5. [Расширенные возможности](#расширенные-возможности)
6. [Лучшие практики](#лучшие-практики)
7. [Распространенные ошибки](#распространенные-ошибки)

## Основы сопоставления путей

Сопоставление путей позволяет заменять длинные относительные пути на короткие псевдонимы:

```typescript
// Без сопоставления путей
import { Button } from '../../../components/ui/Button/Button';
import { api } from '../../../../utils/api/client';
import { User } from '../../../../../types/user.interface';

// С сопоставлением путей
import { Button } from '@components/ui/Button/Button';
import { api } from '@utils/api/client';
import { User } from '@types/user.interface';
```

### Как работает сопоставление путей

TypeScript использует две опции в `tsconfig.json`:

1. **baseUrl** - Базовая директория для разрешения неотносительных путей
2. **paths** - Карта сопоставления псевдонимов с реальными путями

## Конфигурация baseUrl и paths

### Базовая конфигурация

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

### Расширенная конфигурация

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"],
      "@assets/*": ["assets/*"],
      "@services/*": ["services/*"],
      "@hooks/*": ["hooks/*"],
      "@store/*": ["store/*"]
    }
  }
}
```

### Понимание синтаксиса

```json
{
  "paths": {
    // Простое сопоставление
    "@utils": ["utils/index"],           // @utils -> utils/index
    "@utils/*": ["utils/*"],             // @utils/path -> utils/path
    
    // Множественные пути
    "@shared/*": [
      "shared/*",
      "common/*",
      "../node_modules/shared/*"
    ],
    
    // Абсолютные пути
    "@root/*": ["./*"],
    
    // Конкретные файлы
    "config": ["config/index"],
    "constants": ["constants/index"]
  }
}
```

## Практические примеры

### 1. Структура веб-приложения

```bash
web-app/
├── src/
│   ├── components/
│   │   ├── ui/
│   │   │   ├── Button/
│   │   │   │   ├── Button.tsx
│   │   │   │   └── index.ts
│   │   │   └── Input/
│   │   │       ├── Input.tsx
│   │   │       └── index.ts
│   │   └── layout/
│   │       ├── Header/
│   │       │   ├── Header.tsx
│   │       │   └── index.ts
│   │       └── Footer/
│   │           ├── Footer.tsx
│   │           └── index.ts
│   ├── utils/
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   └── index.ts
│   │   ├── helpers/
│   │   │   ├── formatting.ts
│   │   │   └── index.ts
│   │   └── index.ts
│   ├── types/
│   │   ├── user.ts
│   │   ├── product.ts
│   │   └── index.ts
│   ├── services/
│   │   ├── authService.ts
│   │   ├── productService.ts
│   │   └── index.ts
│   ├── hooks/
│   │   ├── useApi.ts
│   │   ├── useAuth.ts
│   │   └── index.ts
│   └── index.tsx
├── tsconfig.json
└── package.json
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@ui/*": ["components/ui/*"],
      "@layout/*": ["components/layout/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"],
      "@services/*": ["services/*"],
      "@hooks/*": ["hooks/*"]
    }
  },
  "include": ["src/**/*"]
}
```

Использование:

```typescript
// src/components/layout/Header/Header.tsx
import { Button } from '@ui/Button';
import { useAuth } from '@hooks/useAuth';
import { api } from '@utils/api';
import { User } from '@types/user';

export function Header() {
  const { user, logout } = useAuth();
  
  const handleLogout = async () => {
    await api.logout();
    logout();
  };
  
  return (
    <header>
      <h1>Welcome, {user?.name}</h1>
      <Button onClick={handleLogout}>Logout</Button>
    </header>
  );
}
```

### 2. Монорепозиторий

```bash
monorepo/
├── packages/
│   ├── core/
│   │   ├── src/
│   │   │   ├── utils/
│   │   │   ├── types/
│   │   │   └── index.ts
│   │   └── tsconfig.json
│   ├── ui/
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   └── index.ts
│   │   └── tsconfig.json
│   └── api/
│       ├── src/
│       │   ├── client/
│       │   ├── services/
│       │   └── index.ts
│       └── tsconfig.json
├── tsconfig.json
└── package.json
```

```json
// Корневой tsconfig.json
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
```

```json
// packages/core/tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@utils/*": ["src/utils/*"],
      "@types/*": ["src/types/*"]
    }
  }
}
```

Использование:

```typescript
// packages/ui/src/components/UserCard.tsx
import { User } from '@monorepo/core/types/user';
import { formatDate } from '@monorepo/core/utils/date';

interface UserCardProps {
  user: User;
}

export function UserCard({ user }: UserCardProps) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>Joined: {formatDate(user.createdAt)}</p>
    </div>
  );
}
```

### 3. Условные пути

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@env": ["config/development", "config/production"],
      "@utils": ["utils/browser", "utils/node"],
      "@api": ["services/api-mock", "services/api-real"]
    }
  }
}
```

```typescript
// src/services/userService.ts
import config from '@env';
import { api } from '@api';
import { isBrowser } from '@utils';

export class UserService {
  async getCurrentUser() {
    if (config.useMock) {
      return { id: '1', name: 'Mock User' };
    }
    
    return api.get('/user/me');
  }
}
```

## Интеграция с бандлерами

### 1. Webpack

Сопоставление путей в Webpack требует дополнительной настройки:

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@types': path.resolve(__dirname, 'src/types')
    }
  }
};
```

Или автоматическая синхронизация:

```javascript
// webpack.config.js
const tsconfig = require('./tsconfig.json');
const path = require('path');

const aliases = {};
const baseUrl = path.resolve(__dirname, tsconfig.compilerOptions.baseUrl);

Object.keys(tsconfig.compilerOptions.paths).forEach(alias => {
  const paths = tsconfig.compilerOptions.paths[alias];
  const target = paths[0].replace('/*', '');
  aliases[alias.replace('/*', '')] = path.resolve(baseUrl, target);
});

module.exports = {
  resolve: {
    alias: aliases
  }
};
```

### 2. Rollup

```javascript
// rollup.config.js
import alias from '@rollup/plugin-alias';
import path from 'path';

export default {
  plugins: [
    alias({
      entries: [
        { find: '@', replacement: path.resolve(__dirname, 'src') },
        { find: '@components', replacement: path.resolve(__dirname, 'src/components') },
        { find: '@utils', replacement: path.resolve(__dirname, 'src/utils') }
      ]
    })
  ]
};
```

### 3. Vite

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import path from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@types': path.resolve(__dirname, 'src/types')
    }
  }
});
```

### 4. Jest

```javascript
// jest.config.js
module.exports = {
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1',
    '^@types/(.*)$': '<rootDir>/src/types/$1'
  }
};
```

## Расширенные возможности

### 1. Динамические пути

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      // Группировка по функциональности
      "@features/*": ["src/features/*"],
      "@features/*/components": ["src/features/*/components"],
      "@features/*/services": ["src/features/*/services"],
      "@features/*/types": ["src/features/*/types"],
      
      // Версионирование API
      "@api/v1/*": ["src/api/v1/*"],
      "@api/v2/*": ["src/api/v2/*"]
    }
  }
}
```

### 2. Множественные сопоставления

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@shared/*": [
        "shared/*",
        "common/*",
        "../node_modules/@company/shared/*"
      ],
      "@utils": [
        "utils/browser",
        "utils/node",
        "utils/index"
      ]
    }
  }
}
```

TypeScript будет искать файлы в порядке перечисления:

1. `src/shared/file` 
2. `src/common/file`
3. `../node_modules/@company/shared/file`

### 3. Абсолютные пути

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@root/*": ["./*"],
      "@project/*": ["/absolute/path/to/project/*"],
      "@external/*": ["/external/libraries/*"]
    }
  }
}
```

## Лучшие практики

### 1. Согласованная структура

```bash
# Хорошая структура
src/
├── components/
├── utils/
├── types/
├── services/
├── hooks/
└── store/

# Плохая структура
src/
├── components/
├── utils/
├── helpers/
├── interfaces/
├── api/
├── services/
└── custom-hooks/
```

```json
// Хорошая конфигурация
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"],
      "@services/*": ["services/*"],
      "@hooks/*": ["hooks/*"]
    }
  }
}
```

### 2. Явные импорты

```typescript
// Хорошо - явные импорты
import { Button } from '@components/ui/Button';
import { api } from '@utils/api';
import { User } from '@types/user';

// Плохо - импорт всего модуля
import * as UI from '@components/ui';
import * as Utils from '@utils';
```

### 3. Документирование псевдонимов

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      // Компоненты пользовательского интерфейса
      "@components/*": ["components/*"],
      "@ui/*": ["components/ui/*"],
      "@layout/*": ["components/layout/*"],
      
      // Утилиты и вспомогательные функции
      "@utils/*": ["utils/*"],
      "@helpers/*": ["utils/helpers/*"],
      
      // Типы и интерфейсы
      "@types/*": ["types/*"],
      
      // Сервисы и API клиенты
      "@services/*": ["services/*"],
      "@api/*": ["services/api/*"],
      
      // React хуки
      "@hooks/*": ["hooks/*"],
      
      // Состояние приложения
      "@store/*": ["store/*"]
    }
  }
}
```

### 4. Избегайте глубоких псевдонимов

```json
// Плохо - слишком много уровней
{
  "paths": {
    "@components/ui/buttons/primary": ["components/ui/buttons/PrimaryButton"],
    "@components/ui/buttons/secondary": ["components/ui/buttons/SecondaryButton"]
  }
}

// Хорошо - плоская структура
{
  "paths": {
    "@components/*": ["components/*"],
    "@ui/*": ["components/ui/*"]
  }
}
```

## Распространенные ошибки

### 1. Несоответствие между TypeScript и бандлером

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@components/*": ["components/*"]
    }
  }
}

// webpack.config.js - забыли настроить
module.exports = {
  // Нет alias для @components
};
```

Исправление:

```javascript
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      '@components': path.resolve(__dirname, 'src/components')
    }
  }
};
```

### 2. Неправильные пути

```json
// Ошибка - относительный путь без ./
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@utils/*": ["utils/*"]  // Правильно
    }
  }
}

// Ошибка - неправильный baseUrl
{
  "compilerOptions": {
    "baseUrl": ".",            // Должен указывать на src
    "paths": {
      "@utils/*": ["src/utils/*"]  // Путь от baseUrl
    }
  }
}
```

### 3. Циклические зависимости через псевдонимы

```typescript
// src/utils/helpers.ts
import { UserService } from '@services/user'; // Псевдоним

// src/services/user.ts
import { formatUser } from '@utils/helpers'; // Цикл!

// Исправление - вынести общие зависимости
// src/shared/formatters.ts
export function formatUser(user: User) {
  return `${user.firstName} ${user.lastName}`;
}
```

### 4. Проблемы с индексными файлами

```json
// Проблема - нет индексных файлов
{
  "paths": {
    "@components/Button": ["components/Button/Button"]  // Нет index.ts
  }
}

// Исправление - добавить index.ts
// components/Button/index.ts
export { Button } from './Button';
export type { ButtonProps } from './types';
```

## Инструменты для работы с путями

### 1. Проверка конфигурации

```bash
# Проверка конфигурации TypeScript
tsc --showConfig

# Трассировка разрешения модулей
tsc --traceResolution
```

### 2. Анализ зависимостей

```bash
# Анализ циклических зависимостей
npx madge --circular src/

# Анализ зависимостей с псевдонимами
npx dpdm --alias @components=src/components src/
```

### 3. Автоматическая генерация

```javascript
// generate-paths.js
const fs = require('fs');
const path = require('path');

function generatePaths(srcDir) {
  const paths = {};
  const componentsDir = path.join(srcDir, 'components');
  
  if (fs.existsSync(componentsDir)) {
    const components = fs.readdirSync(componentsDir);
    components.forEach(component => {
      if (fs.statSync(path.join(componentsDir, component)).isDirectory()) {
        paths[`@components/${component}/*`] = [`components/${component}/*`];
      }
    });
  }
  
  return paths;
}

const paths = generatePaths('./src');
console.log(JSON.stringify(paths, null, 2));
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[ts/configuration/basic-setup|Базовая настройка]] - Начальная конфигурация
- [[ts/configuration/compiler-options|Опции компилятора]] - Подробное описание всех опций
- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Алгоритмы поиска модулей
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев

## Рекомендации по изучению

1. Начните с простых псевдонимов для основных директорий
2. Освойте интеграцию с бандлерами
3. Практикуйтесь в создании согласованной структуры проекта
4. Изучите инструменты для анализа зависимостей
5. Освойте работу с монорепозиториями и множественными путями
6. Практикуйтесь в устранении распространенных ошибок

## Следующие шаги

- [[ts/configuration/watch-options|Опции наблюдения]] - Настройка режима разработки
- [[ts/configuration/type-acquisition|Получение типов]] - Автоматическое получение типов
- [[ts/configuration/performance|Оптимизация производительности]] - Настройки для ускорения компиляции
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев
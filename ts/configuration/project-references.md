# Ссылки на проекты в TypeScript

Ссылки на проекты (Project References) - это мощная возможность TypeScript, которая позволяет организовать большие кодовые базы в виде множества взаимосвязанных проектов. Это улучшает производительность сборки, обеспечивает лучшую изоляцию кода и упрощает управление зависимостями.

## Содержание

1. [Основы ссылок на проекты](#основы-ссылок-на-проекты)
2. [Конфигурация проектов](#конфигурация-проектов)
3. [Практические примеры](#практические-примеры)
4. [Сборка с ссылками](#сборка-с-ссылками)
5. [Интеграция с бандлерами](#интеграция-с-бандлерами)
6. [Лучшие практики](#лучшие-практики)
7. [Распространенные ошибки](#распространенные-ошибки)

## Основы ссылок на проекты

Ссылки на проекты позволяют TypeScript понимать зависимости между различными проектами в монорепозитории и оптимизировать процесс сборки.

### Что такое ссылки на проекты

Ссылки на проекты - это способ указать TypeScript, что один проект зависит от другого проекта, а не от уже скомпилированного кода.

```json
// Корневой tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/ui" },
    { "path": "./packages/api" }
  ]
}
```

### Преимущества ссылок на проекты

1. **Инкрементальная сборка** - собираются только измененные проекты
2. **Параллельная сборка** - проекты могут собираться параллельно
3. **Лучшая изоляция** - каждый проект имеет свои настройки
4. **Упрощенное управление зависимостями** - зависимости между проектами явны
5. **Улучшенная производительность** - TypeScript может кэшировать результаты

## Конфигурация проектов

### Структура монорепозитория

```bash
monorepo/
├── packages/
│   ├── core/
│   │   ├── src/
│   │   │   └── index.ts
│   │   ├── dist/
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── ui/
│   │   ├── src/
│   │   │   └── index.ts
│   │   ├── dist/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── api/
│       ├── src/
│       │   └── index.ts
│       ├── dist/
│       ├── package.json
│       └── tsconfig.json
├── package.json
└── tsconfig.json
```

### Конфигурация корневого проекта

```json
// tsconfig.json (корневой)
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/ui" },
    { "path": "./packages/api" }
  ]
}
```

### Конфигурация дочерних проектов

```json
// packages/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

```json
// packages/ui/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../core" }
  ]
}
```

```json
// packages/api/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../core" },
    { "path": "../ui" }
  ]
}
```

### Обязательные опции для ссылок

Для работы ссылок на проекты необходимо включить определенные опции:

```json
{
  "compilerOptions": {
    "composite": true,        // Обязательно для проектов со ссылками
    "declaration": true,      // Генерация .d.ts файлов
    "declarationMap": true    // Генерация .d.ts.map файлов
  }
}
```

## Практические примеры

### 1. Библиотека с несколькими пакетами

```bash
library/
├── packages/
│   ├── core/
│   │   ├── src/
│   │   │   ├── utils.ts
│   │   │   └── index.ts
│   │   ├── dist/
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── react/
│   │   ├── src/
│   │   │   ├── hooks.ts
│   │   │   └── index.ts
│   │   ├── dist/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── node/
│       ├── src/
│       │   ├── server.ts
│       │   └── index.ts
│       ├── dist/
│       ├── package.json
│       └── tsconfig.json
├── package.json
└── tsconfig.json
```

```json
// tsconfig.json (корневой)
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/react" },
    { "path": "./packages/node" }
  ]
}
```

```json
// packages/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "target": "es2020",
    "module": "esnext",
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true
  },
  "include": ["src/**/*"]
}
```

```json
// packages/react/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "target": "es2020",
    "module": "esnext",
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "jsx": "react-jsx"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../core" }
  ]
}
```

```json
// packages/node/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "target": "es2020",
    "module": "commonjs",
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../core" }
  ]
}
```

### 2. Приложение с разделением на слои

```bash
application/
├── src/
│   ├── shared/
│   │   ├── types/
│   │   ├── utils/
│   │   └── tsconfig.json
│   ├── domain/
│   │   ├── models/
│   │   ├── services/
│   │   └── tsconfig.json
│   ├── infrastructure/
│   │   ├── database/
│   │   ├── api/
│   │   └── tsconfig.json
│   └── presentation/
│       ├── components/
│       ├── pages/
│       └── tsconfig.json
├── tsconfig.json
└── package.json
```

```json
// tsconfig.json (корневой)
{
  "files": [],
  "references": [
    { "path": "./src/shared" },
    { "path": "./src/domain" },
    { "path": "./src/infrastructure" },
    { "path": "./src/presentation" }
  ]
}
```

```json
// src/shared/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "../../dist/shared",
    "rootDir": ".",
    "strict": true
  },
  "include": ["**/*"]
}
```

```json
// src/domain/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "../../dist/domain",
    "rootDir": ".",
    "strict": true
  },
  "include": ["**/*"],
  "references": [
    { "path": "../shared" }
  ]
}
```

```json
// src/infrastructure/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "../../dist/infrastructure",
    "rootDir": ".",
    "strict": true
  },
  "include": ["**/*"],
  "references": [
    { "path": "../shared" },
    { "path": "../domain" }
  ]
}
```

```json
// src/presentation/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "../../dist/presentation",
    "rootDir": ".",
    "strict": true,
    "jsx": "react-jsx"
  },
  "include": ["**/*"],
  "references": [
    { "path": "../shared" },
    { "path": "../domain" },
    { "path": "../infrastructure" }
  ]
}
```

## Сборка с ссылками

### Команда сборки

```bash
# Сборка всех проектов с ссылками
tsc --build

# Сборка с очисткой
tsc --build --clean

# Инкрементальная сборка
tsc --build --incremental

# Сборка с подробным выводом
tsc --build --verbose

# Сборка определенных проектов
tsc --build packages/core packages/ui
```

### Конфигурация сборки

```json
// tsconfig.build.json
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/ui" },
    { "path": "./packages/api" }
  ],
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  }
}
```

### Скрипты package.json

```json
{
  "scripts": {
    "build": "tsc --build",
    "build:clean": "tsc --build --clean",
    "build:watch": "tsc --build --watch",
    "typecheck": "tsc --build --dry",
    "typecheck:watch": "tsc --build --watch --dry"
  }
}
```

### Порядок сборки

TypeScript автоматически определяет порядок сборки на основе ссылок:

```json
// Если ui зависит от core, а api зависит от ui и core:
// Порядок сборки: core -> ui -> api
{
  "references": [
    { "path": "./packages/core" },  // Собирается первым
    { "path": "./packages/ui" },    // Зависит от core
    { "path": "./packages/api" }    // Зависит от core и ui
  ]
}
```

## Интеграция с бандлерами

### Webpack с Project References

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              projectReferences: true // Включает поддержку ссылок
            }
          }
        ]
      }
    ]
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js']
  }
};
```

### Rollup с Project References

```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';

export default {
  input: 'src/index.ts',
  output: {
    dir: 'dist',
    format: 'es'
  },
  plugins: [
    typescript({
      tsconfig: './tsconfig.json',
      projectReferences: true
    })
  ]
};
```

### Vite с Project References

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react(),
    {
      name: 'typescript-project-references',
      buildStart() {
        // Предварительная сборка проектов со ссылками
        require('child_process').execSync('tsc --build', { stdio: 'inherit' });
      }
    }
  ]
});
```

## Лучшие практики

### 1. Структура проекта

```bash
# Хорошая структура
project/
├── packages/
│   ├── shared/          # Общие типы и утилиты
│   ├── domain/          # Бизнес-логика
│   ├── infrastructure/  # Внешние зависимости
│   └── presentation/    # Интерфейс
└── tsconfig.json

# Плохая структура
project/
├── packages/
│   ├── utils1/
│   ├── utils2/
│   ├── helpers1/
│   └── helpers2/
└── tsconfig.json
```

### 2. Явные зависимости

```json
// packages/ui/tsconfig.json
{
  "references": [
    { "path": "../core" }  // Явная зависимость
  ]
}

// packages/api/tsconfig.json
{
  "references": [
    { "path": "../core" },  // Зависимость от core
    { "path": "../ui" }     // Зависимость от ui
  ]
}
```

### 3. Консистентная конфигурация

```json
// tsconfig.base.json
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}

// packages/core/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

### 4. Изоляция типов

```typescript
// packages/shared/src/types.ts
export interface User {
  id: string;
  name: string;
  email: string;
}

export interface Product {
  id: string;
  name: string;
  price: number;
}

// packages/domain/src/models/user.model.ts
import { User } from '@shared/types';

export class UserModel implements User {
  constructor(
    public id: string,
    public name: string,
    public email: string
  ) {}
  
  get displayName(): string {
    return `${this.name} (${this.email})`;
  }
}
```

## Распространенные ошибки

### 1. Отсутствие composite

```json
// Ошибка: Project references may not form a circular graph
{
  "references": [
    { "path": "../other-project" }
  ]
  // Отсутствует "composite": true
}

// Исправление:
{
  "compilerOptions": {
    "composite": true,
    "declaration": true
  },
  "references": [
    { "path": "../other-project" }
  ]
}
```

### 2. Циклические зависимости

```json
// packages/project-a/tsconfig.json
{
  "references": [
    { "path": "../project-b" }  // Зависит от B
  ]
}

// packages/project-b/tsconfig.json
{
  "references": [
    { "path": "../project-a" }  // Зависит от A - цикл!
  ]
}

// Исправление - вынести общие зависимости:
// packages/shared/tsconfig.json
{
  // Общие зависимости
}

// packages/project-a/tsconfig.json
{
  "references": [
    { "path": "../shared" }  // Зависит от shared
  ]
}

// packages/project-b/tsconfig.json
{
  "references": [
    { "path": "../shared" }  // Зависит от shared
  ]
}
```

### 3. Неправильные пути

```json
// Ошибка: Can't resolve reference
{
  "references": [
    { "path": "./packages/core" }  // Неправильный путь
  ]
}

// Исправление:
{
  "references": [
    { "path": "../packages/core" }  // Правильный путь
  ]
}
```

### 4. Отсутствие declaration

```json
// Ошибка: Composite projects must have 'declaration' enabled
{
  "compilerOptions": {
    "composite": true
    // Отсутствует "declaration": true
  }
}

// Исправление:
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true
  }
}
```

## Инструменты для работы со ссылками

### 1. Анализ зависимостей

```bash
# Показывает граф зависимостей
tsc --build --dry --verbose

# Проверяет консистентность ссылок
tsc --build --clean
```

### 2. Мониторинг производительности

```bash
# Сборка с метриками
tsc --build --verbose

# Инкрементальная сборка
tsc --build --incremental
```

### 3. Интеграция с IDE

```json
// .vscode/settings.json
{
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "typescript.suggest.autoImports": true,
  "typescript.updateImportsOnFileMove.enabled": "always"
}
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[ts/configuration/basic-setup|Базовая настройка]] - Начальная конфигурация
- [[ts/configuration/compiler-options|Опции компилятора]] - Подробное описание всех опций
- [[ts/advanced-modules/bundling|Бандлинг модулей]] - Интеграция с бандлерами
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев

## Рекомендации по изучению

1. Начните с простой структуры из 2-3 проектов
2. Освойте базовую конфигурацию ссылок
3. Практикуйтесь в сборке проектов с ссылками
4. Изучите интеграцию с бандлерами
5. Освойте лучшие практики организации кода
6. Практикуйтесь в устранении циклических зависимостей

## Следующие шаги

- [[ts/configuration/build-modes|Режимы сборки]] - Инкрементальная и композитная сборка
- [[ts/configuration/path-mapping|Сопоставление путей]] - Псевдонимы и разрешение модулей
- [[ts/configuration/watch-options|Опции наблюдения]] - Настройка режима разработки
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев
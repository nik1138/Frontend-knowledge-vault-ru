# Режимы сборки TypeScript

Режимы сборки TypeScript определяют, как компилятор обрабатывает проекты, управляет зависимостями и оптимизирует процесс компиляции. Понимание различных режимов сборки критически важно для эффективной разработки крупных приложений.

## Содержание

1. [Инкрементальная сборка](#инкрементальная-сборка)
2. [Композитные проекты](#композитные-проекты)
3. [Параллельная сборка](#параллельная-сборка)
4. [Практические примеры](#практические-примеры)
5. [Оптимизация производительности](#оптимизация-производительности)
6. [Интеграция с инструментами](#интеграция-с-инструментами)
7. [Лучшие практики](#лучшие-практики)

## Инкрементальная сборка

Инкрементальная сборка позволяет TypeScript пересобирать только измененные файлы и их зависимости, значительно ускоряя процесс разработки.

### Базовая настройка

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  }
}
```

### Как работает инкрементальная сборка

1. **Анализ изменений** - TypeScript отслеживает изменения в файлах
2. **Кэширование** - Результаты предыдущих сборок сохраняются
3. **Сборка изменений** - Пересобираются только измененные файлы
4. **Обновление кэша** - Кэш обновляется новыми результатами

### Файл .tsbuildinfo

Файл `.tsbuildinfo` содержит метаданные для инкрементальной сборки:

```json
{
  "program": {
    "fileNames": [
      "/project/src/index.ts",
      "/project/src/utils.ts"
    ],
    "fileInfos": {
      "/project/src/index.ts": {
        "version": "1234567890",
        "signature": "abc123"
      }
    }
  },
  "version": "3.9.0"
}
```

### Преимущества инкрементальной сборки

```bash
# Без инкрементальной сборки
Full build: 10.0s

# С инкрементальной сборкой
Full build: 10.0s
Incremental build: 0.5s  # 20x ускорение
```

## Композитные проекты

Композитные проекты (Composite Projects) - это проекты, которые могут быть использованы как зависимости других проектов.

### Настройка композитного проекта

```json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist"
  }
}
```

### Требования к композитным проектам

1. **composite: true** - Обязательная опция
2. **declaration: true** - Генерация файлов объявлений
3. **declarationMap: true** - Генерация source maps для объявлений
4. **outDir** - Явное указание выходной директории

### Пример структуры композитного проекта

```bash
library/
├── packages/
│   ├── core/
│   │   ├── src/
│   │   │   └── index.ts
│   │   ├── dist/
│   │   │   ├── index.js
│   │   │   ├── index.d.ts
│   │   │   └── index.d.ts.map
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── ui/
│       ├── src/
│       │   └── index.ts
│       ├── dist/
│       │   ├── index.js
│       │   ├── index.d.ts
│       │   └── index.d.ts.map
│       ├── package.json
│       └── tsconfig.json
├── tsconfig.json
└── package.json
```

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

## Параллельная сборка

Параллельная сборка позволяет одновременно собирать независимые проекты, ускоряя общий процесс сборки.

### Настройка параллельной сборки

```json
// tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/ui" },
    { "path": "./packages/api" }
  ]
}
```

### Порядок сборки

TypeScript автоматически определяет порядок сборки на основе зависимостей:

```json
// Если ui зависит от core, а api зависит от ui и core:
// Порядок сборки: core -> ui -> api (последовательно)
// Но если ui и api независимы - они могут собираться параллельно
```

### Пример параллельной сборки

```bash
# Последовательная сборка
tsc --build packages/core packages/ui packages/api

# Параллельная сборка (автоматически определяется)
tsc --build --force  # Принудительная пересборка всех проектов
```

## Практические примеры

### 1. Монорепозиторий с инкрементальной сборкой

```bash
monorepo/
├── packages/
│   ├── shared/
│   │   ├── src/
│   │   │   ├── types.ts
│   │   │   └── utils.ts
│   │   ├── dist/
│   │   └── tsconfig.json
│   ├── frontend/
│   │   ├── src/
│   │   │   └── index.tsx
│   │   ├── dist/
│   │   └── tsconfig.json
│   └── backend/
│       ├── src/
│       │   └── index.ts
│       ├── dist/
│       └── tsconfig.json
├── tsconfig.json
└── package.json
```

```json
// tsconfig.json (корневой)
{
  "files": [],
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/frontend" },
    { "path": "./packages/backend" }
  ]
}
```

```json
// packages/shared/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src/**/*"]
}
```

```json
// packages/frontend/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "tsBuildInfoFile": "./dist/.tsbuildinfo",
    "jsx": "react-jsx"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../shared" }
  ]
}
```

```json
// packages/backend/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../shared" }
  ]
}
```

### 2. Скрипты для сборки

```json
// package.json
{
  "scripts": {
    "build": "tsc --build",
    "build:clean": "tsc --build --clean",
    "build:watch": "tsc --build --watch",
    "build:force": "tsc --build --force",
    "typecheck": "tsc --build --dry",
    "typecheck:watch": "tsc --build --watch --dry"
  }
}
```

### 3. CI/CD интеграция

```yaml
# .github/workflows/build.yml
name: Build
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm ci
      - run: npm run build
      - run: npm run typecheck
```

## Оптимизация производительности

### 1. Кэширование сборки

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/cache/.tsbuildinfo",
    "disableSizeLimit": true
  }
}
```

### 2. Исключение ненужных файлов

```json
{
  "compilerOptions": {
    "skipLibCheck": true,        // Пропуск проверки .d.ts файлов
    "skipDefaultLibCheck": true  // Пропуск проверки стандартных библиотек
  },
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/__tests__/**"
  ]
}
```

### 3. Оптимизация для разработки

```json
// tsconfig.dev.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo",
    "noEmitOnError": false,
    "diagnostics": true
  }
}
```

### 4. Оптимизация для продакшена

```json
// tsconfig.prod.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "incremental": false,
    "sourceMap": false,
    "removeComments": true,
    "noEmitOnError": true
  }
}
```

### 5. Метрики производительности

```bash
# Базовая диагностика
tsc --build --verbose

# Расширенная диагностика
tsc --build --verbose --extendedDiagnostics

# Только проверка типов (без эмита)
tsc --build --dry

# Очистка кэша
tsc --build --clean
```

## Интеграция с инструментами

### 1. Webpack

```javascript
// webpack.config.js
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
              projectReferences: true,
              transpileOnly: true, // Для ускорения в режиме разработки
              compilerOptions: {
                module: 'esnext'
              }
            }
          }
        ]
      }
    ]
  }
};
```

### 2. Rollup

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

### 3. Vite

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react(),
    {
      name: 'typescript-build',
      buildStart() {
        // Предварительная сборка с инкрементальной сборкой
        require('child_process').execSync('tsc --build', { stdio: 'inherit' });
      }
    }
  ]
});
```

### 4. ESLint

```javascript
// .eslintrc.js
module.exports = {
  parserOptions: {
    project: './tsconfig.json',
    tsconfigRootDir: __dirname
  },
  extends: [
    '@typescript-eslint/recommended',
    '@typescript-eslint/recommended-requiring-type-checking'
  ]
};
```

## Лучшие практики

### 1. Структура проекта

```bash
# Хорошая структура
project/
├── packages/
│   ├── shared/          # Общие зависимости
│   ├── domain/          # Бизнес-логика
│   ├── infrastructure/  # Внешние зависимости
│   └── presentation/    # Интерфейс
├── tsconfig.json        # Корневая конфигурация
├── tsconfig.build.json  # Конфигурация для сборки
└── package.json

# Плохая структура
project/
├── src/
│   ├── utils1/
│   ├── utils2/
│   ├── helpers1/
│   └── helpers2/
└── tsconfig.json
```

### 2. Конфигурация сборки

```json
// tsconfig.build.json
{
  "files": [],
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/domain" },
    { "path": "./packages/infrastructure" },
    { "path": "./packages/presentation" }
  ],
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  }
}
```

### 3. Скрипты сборки

```json
{
  "scripts": {
    "prebuild": "rimraf dist",
    "build": "tsc --build tsconfig.build.json",
    "build:clean": "tsc --build tsconfig.build.json --clean",
    "build:watch": "tsc --build tsconfig.build.json --watch",
    "postbuild": "echo 'Build completed'",
    "typecheck": "tsc --build tsconfig.build.json --dry",
    "typecheck:watch": "tsc --build tsconfig.build.json --watch --dry"
  }
}
```

### 4. Мониторинг производительности

```bash
# Регулярный мониторинг
npm run build -- --extendedDiagnostics

# Сравнение времени сборки
# До оптимизации: 15.2s
# После оптимизации: 3.8s (4x ускорение)
```

### 5. Кэширование в CI/CD

```yaml
# .github/workflows/build.yml
name: Build
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/cache@v2
        with:
          path: |
            **/dist
            **/.tsbuildinfo
          key: ${{ runner.os }}-build-${{ hashFiles('**/tsconfig.json') }}
      - run: npm ci
      - run: npm run build
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
  }
}
```

### 2. Неправильная конфигурация кэша

```json
// Проблема: Кэш в той же директории, что и исходники
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./src/.tsbuildinfo"  // Плохо!
  }
}

// Исправление:
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"  // Хорошо
  }
}
```

### 3. Циклические зависимости

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

// Исправление - вынести общие зависимости
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[ts/configuration/project-references|Ссылки на проекты]] - Много-проектные конфигурации
- [[ts/configuration/compiler-options|Опции компилятора]] - Подробное описание всех опций
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации

## Рекомендации по изучению

1. Начните с простой инкрементальной сборки
2. Освойте композитные проекты и ссылки
3. Практикуйтесь в параллельной сборке
4. Изучите интеграцию с бандлерами
5. Освойте оптимизацию производительности
6. Практикуйтесь в CI/CD интеграции

## Следующие шаги

- [[ts/configuration/path-mapping|Сопоставление путей]] - Псевдонимы и разрешение модулей
- [[ts/configuration/watch-options|Опции наблюдения]] - Настройка режима разработки
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев
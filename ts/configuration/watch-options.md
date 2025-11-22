# Опции наблюдения в TypeScript

Опции наблюдения (Watch Options) в TypeScript позволяют настраивать поведение компилятора в режиме разработки, когда он отслеживает изменения файлов и автоматически перекомпилирует проект. Правильная настройка этих опций критически важна для эффективной разработки.

## Содержание

1. [Основы режима наблюдения](#основы-режима-наблюдения)
2. [Конфигурация watchOptions](#конфигурация-watchoptions)
3. [Практические примеры](#практические-примеры)
4. [Интеграция с инструментами](#интеграция-с-инструментами)
5. [Оптимизация производительности](#оптимизация-производительности)
6. [Лучшие практики](#лучшие-практики)
7. [Распространенные ошибки](#распространенные-ошибки)

## Основы режима наблюдения

Режим наблюдения позволяет TypeScript автоматически перекомпилировать проект при изменении файлов:

```bash
# Запуск в режиме наблюдения
tsc --watch

# Запуск сборки проектов в режиме наблюдения
tsc --build --watch
```

### Как работает режим наблюдения

1. **Инициализация** - TypeScript сканирует все файлы проекта
2. **Наблюдение** - Система отслеживает изменения в файлах
3. **Перекомпиляция** - При изменении файлов запускается перекомпиляция
4. **Уведомление** - Обновляются выходные файлы и уведомляются инструменты

### Преимущества режима наблюдения

```bash
# Без режима наблюдения
# 1. tsc
# 2. Внести изменения
# 3. tsc (повторная компиляция)
# 4. Повторять...

# С режимом наблюдения
# 1. tsc --watch (однократный запуск)
# 2. Автоматическая перекомпиляция при изменениях
```

## Конфигурация watchOptions

### Базовая конфигурация

```json
{
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents"
  }
}
```

### Все доступные опции

```json
{
  "watchOptions": {
    // Метод наблюдения за файлами
    "watchFile": "useFsEvents", // или "fixedPollingInterval", "priorityPollingInterval", "dynamicPriorityPolling", "useFsEventsOnParentDirectory"
    
    // Метод наблюдения за директориями
    "watchDirectory": "useFsEvents", // или "fixedPollingInterval", "dynamicPriorityPolling"
    
    // Резервный метод опроса
    "fallbackPolling": "dynamicPriority", // или "fixedInterval", "priorityInterval", "dynamicPriority"
    
    // Синхронное наблюдение за директориями
    "synchronousWatchDirectory": true,
    
    // Исключенные директории
    "excludeDirectories": ["**/node_modules", "_build"],
    
    // Исключенные файлы
    "excludeFiles": ["**/*.spec.ts", "**/*.test.ts"]
  }
}
```

### watchFile - Наблюдение за файлами

```json
{
  "watchOptions": {
    "watchFile": "useFsEvents" // Использует события файловой системы (рекомендуется)
  }
}
```

Доступные значения:
- **useFsEvents** - Использует события файловой системы (самый эффективный)
- **fixedPollingInterval** - Постоянный опрос каждые 250мс
- **priorityPollingInterval** - Приоритетный опрос с разными интервалами
- **dynamicPriorityPolling** - Динамический приоритетный опрос
- **useFsEventsOnParentDirectory** - События файловой системы для родительской директории

### watchDirectory - Наблюдение за директориями

```json
{
  "watchOptions": {
    "watchDirectory": "useFsEvents" // Использует события файловой системы
  }
}
```

Доступные значения:
- **useFsEvents** - Использует события файловой системы (рекомендуется)
- **fixedPollingInterval** - Постоянный опрос каждые 250мс
- **dynamicPriorityPolling** - Динамический приоритетный опрос

### fallbackPolling - Резервный опрос

```json
{
  "watchOptions": {
    "fallbackPolling": "dynamicPriority" // Динамический приоритетный опрос
  }
}
```

Доступные значения:
- **fixedInterval** - Фиксированный интервал опроса
- **priorityInterval** - Приоритетный интервал опроса
- **dynamicPriority** - Динамический приоритетный опрос (рекомендуется)

## Практические примеры

### 1. Оптимизированная конфигурация для разработки

```json
// tsconfig.dev.json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "synchronousWatchDirectory": true,
    "excludeDirectories": [
      "**/node_modules",
      "dist",
      "_build",
      ".git"
    ],
    "excludeFiles": [
      "**/*.spec.ts",
      "**/*.test.ts",
      "**/*.d.ts"
    ]
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

### 2. Конфигурация для монорепозитория

```json
// tsconfig.json (корневой)
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/ui" },
    { "path": "./packages/api" }
  ],
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "excludeDirectories": [
      "**/node_modules",
      "**/dist",
      ".git"
    ]
  }
}
```

```json
// packages/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "excludeDirectories": [
      "**/node_modules",
      "dist",
      "node_modules"
    ]
  },
  "include": ["src/**/*"]
}
```

### 3. Скрипты для режима разработки

```json
// package.json
{
  "scripts": {
    "dev": "tsc --watch --project tsconfig.dev.json",
    "dev:build": "tsc --build --watch",
    "dev:verbose": "tsc --watch --project tsconfig.dev.json --extendedDiagnostics",
    "dev:clean": "tsc --build --clean && tsc --build --watch"
  }
}
```

### 4. Интеграция с другими инструментами

```json
// tsconfig.watch.json
{
  "extends": "./tsconfig.json",
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "synchronousWatchDirectory": true,
    "excludeDirectories": [
      "**/node_modules",
      "dist",
      ".git",
      ".cache"
    ]
  }
}
```

## Интеграция с инструментами

### 1. Webpack Dev Server

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  devServer: {
    contentBase: './dist',
    hot: true,
    watchContentBase: true,
    watchOptions: {
      ignored: /node_modules/
    }
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js']
  }
};
```

```json
// tsconfig.json для Webpack
{
  "compilerOptions": {
    "target": "es5",
    "module": "esnext",
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  },
  "watchOptions": {
    "watchFile": "useFsEventsOnParentDirectory",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority"
  }
}
```

### 2. Vite

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    watch: {
      useFsEvents: true,
      ignored: ['**/node_modules/**', '**/dist/**']
    }
  },
  optimizeDeps: {
    include: ['react', 'react-dom']
  }
});
```

```json
// tsconfig.json для Vite
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "lib": ["esnext", "dom"],
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "jsx": "react-jsx"
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority"
  }
}
```

### 3. Node.js с nodemon

```json
// nodemon.json
{
  "watch": ["src"],
  "ext": "ts,json",
  "ignore": ["src/**/*.spec.ts", "src/**/*.test.ts"],
  "exec": "tsc && node dist/index.js"
}
```

```json
// tsconfig.json для Node.js
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "sourceMap": true
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "excludeDirectories": ["**/node_modules", "dist"]
  }
}
```

## Оптимизация производительности

### 1. Исключение ненужных файлов

```json
{
  "watchOptions": {
    "excludeDirectories": [
      "**/node_modules",
      "**/dist",
      "**/.git",
      "**/.cache",
      "**/coverage",
      "**/build"
    ],
    "excludeFiles": [
      "**/*.spec.ts",
      "**/*.test.ts",
      "**/*.d.ts",
      "**/*.map"
    ]
  }
}
```

### 2. Настройка интервалов опроса

```json
// Для высокопроизводительных систем
{
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "fixedInterval"
  }
}

// Для систем с ограниченными ресурсами
{
  "watchOptions": {
    "watchFile": "dynamicPriorityPolling",
    "watchDirectory": "dynamicPriorityPolling",
    "fallbackPolling": "dynamicPriority"
  }
}
```

### 3. Синхронное наблюдение

```json
{
  "watchOptions": {
    "synchronousWatchDirectory": true, // Улучшает производительность на Windows
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents"
  }
}
```

### 4. Мониторинг производительности

```bash
# Запуск с расширенной диагностикой
tsc --watch --extendedDiagnostics

# Пример вывода:
# Files:            1000
# Lines:           50000
# Memory used:     200000K
# Parse time:        2.00s
# Bind time:         1.00s
# Check time:        3.00s
# Emit time:         0.50s
# Watch invoke:      0.10s  # Время на запуск наблюдения
```

## Лучшие практики

### 1. Разделение конфигураций

```json
// tsconfig.json - основная конфигурация
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "strict": true
  }
}

// tsconfig.dev.json - для разработки
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "sourceMap": true,
    "outDir": "./dist/dev"
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents"
  }
}

// tsconfig.prod.json - для продакшена
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "sourceMap": false,
    "removeComments": true,
    "outDir": "./dist/prod"
  }
}
```

### 2. Исключение тестовых файлов

```json
{
  "watchOptions": {
    "excludeFiles": [
      "**/*.spec.ts",
      "**/*.test.ts",
      "**/*.e2e.ts",
      "**/__tests__/**",
      "**/__mocks__/**"
    ]
  }
}
```

### 3. Оптимизация для CI/CD

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
      - run: npm run build -- --watch false # Отключение watch в CI
```

### 4. Мониторинг ресурсов

```bash
# Мониторинг использования памяти
tsc --watch --diagnostics

# Мониторинг времени компиляции
time tsc --watch --dry # Только проверка типов
```

## Распространенные ошибки

### 1. Конфликт с другими инструментами

```json
// Проблема: конфликтующие настройки
{
  "watchOptions": {
    "watchFile": "fixedPollingInterval" // Высокая нагрузка
  }
}

// Исправление:
{
  "watchOptions": {
    "watchFile": "useFsEvents", // Использует события ОС
    "fallbackPolling": "dynamicPriority"
  }
}
```

### 2. Неправильное исключение файлов

```json
// Проблема: исключение нужных файлов
{
  "watchOptions": {
    "excludeFiles": [
      "**/*.ts" // Исключает все TypeScript файлы!
    ]
  }
}

// Исправление:
{
  "watchOptions": {
    "excludeFiles": [
      "**/*.spec.ts",
      "**/*.test.ts"
    ]
  }
}
```

### 3. Проблемы с правами доступа

```json
// Проблема: недостаточные права для наблюдения
{
  "watchOptions": {
    "watchFile": "useFsEvents" // Может не работать без прав
  }
}

// Исправление:
{
  "watchOptions": {
    "watchFile": "dynamicPriorityPolling", // Работает всегда
    "fallbackPolling": "dynamicPriority"
  }
}
```

### 4. Избыточное наблюдение

```json
// Проблема: наблюдение за node_modules
{
  "watchOptions": {
    // Нет excludeDirectories для node_modules
  }
}

// Исправление:
{
  "watchOptions": {
    "excludeDirectories": [
      "**/node_modules",
      "dist",
      ".git"
    ]
  }
}
```

## Инструменты для диагностики

### 1. Мониторинг файловой системы

```bash
# Linux/macOS
inotifywait -m -r -e modify src/

# Windows
powershell Get-EventLog -LogName System -Source Microsoft-Windows-Kernel-File
```

### 2. Профилирование TypeScript

```bash
# Запуск с профилированием
tsc --watch --generateCpuProfile cpu-profile.cpuprofile

# Анализ профиля в Chrome DevTools
# chrome://inspect -> Open dedicated DevTools for Node -> Profiler
```

### 3. Логирование изменений

```bash
# Логирование всех изменений
tsc --watch --listFiles | grep -E "(src|changed)"

# Мониторинг конкретных файлов
tsc --watch --explainFiles
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[Базовая настройка TypeScript|Базовая настройка]] - Начальная конфигурация
- [[ts/configuration/compiler-options|Опции компилятора]] - Подробное описание всех опций
- [[ts/configuration/build-modes|Режимы сборки]] - Инкрементальная и композитная сборка
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации

## Рекомендации по изучению

1. Начните с базовой конфигурации watchOptions
2. Освойте интеграцию с популярными инструментами разработки
3. Практикуйтесь в оптимизации производительности
4. Изучите инструменты для диагностики проблем
5. Освойте лучшие практики для разных сред
6. Практикуйтесь в устранении распространенных ошибок

## Следующие шаги

- [[ts/configuration/type-acquisition|Получение типов]] - Автоматическое получение типов
- [[ts/configuration/performance|Оптимизация производительности]] - Настройки для ускорения компиляции
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев
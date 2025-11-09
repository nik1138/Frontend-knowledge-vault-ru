# Получение типов в TypeScript

Получение типов (Type Acquisition) - это механизм TypeScript, который автоматически находит и устанавливает типы для библиотек JavaScript, не имеющих встроенных типов. Это позволяет использовать сторонние библиотеки с полной поддержкой типизации.

## Содержание

1. [Основы получения типов](#основы-получения-типов)
2. [Конфигурация typeAcquisition](#конфигурация-typeacquisition)
3. [Практические примеры](#практические-примеры)
4. [Интеграция с DefinitelyTyped](#интеграция-с-definitelytyped)
5. [Расширенные возможности](#расширенные-возможности)
6. [Лучшие практики](#лучшие-практики)
7. [Распространенные ошибки](#распространенные-ошибки)

## Основы получения типов

Получение типов позволяет TypeScript автоматически находить и устанавливать типы для библиотек:

```typescript
// Без получения типов
import * as _ from 'lodash'; // any тип

// С получением типов
import * as _ from 'lodash'; // Полная типизация
```

### Как работает получение типов

1. **Анализ зависимостей** - TypeScript анализирует package.json
2. **Поиск типов** - Ищет типы в @types/* и в самих библиотеках
3. **Установка типов** - Автоматически устанавливает недостающие типы
4. **Кэширование** - Сохраняет типы для повторного использования

### Преимущества получения типов

```bash
# Без автоматического получения типов
npm install lodash
npm install @types/lodash  # Нужно устанавливать вручную

# С автоматическим получением типов
npm install lodash
# Типы устанавливаются автоматически
```

## Конфигурация typeAcquisition

### Базовая конфигурация

```json
{
  "typeAcquisition": {
    "enable": true,
    "include": ["lodash", "express"],
    "exclude": ["jquery"]
  }
}
```

### Все доступные опции

```json
{
  "typeAcquisition": {
    // Включение/выключение автоматического получения типов
    "enable": true,
    
    // Явное включение типов
    "include": [
      "lodash",
      "express",
      "react",
      "node"
    ],
    
    // Явное исключение типов
    "exclude": [
      "jquery",
      "bootstrap"
    ],
    
    // Директории для поиска типов
    "typeRoots": [
      "./node_modules/@types",
      "./src/types"
    ],
    
    // Автоматическое включение типов из node_modules
    "autoImportFromNodeModules": true,
    
    // Кэширование типов
    "cache": true,
    
    // Директория для кэша типов
    "cacheDirectory": "./.tscache"
  }
}
```

### enable - Включение получения типов

```json
{
  "typeAcquisition": {
    "enable": true // Включает автоматическое получение типов
  }
}
```

### include - Явное включение типов

```json
{
  "typeAcquisition": {
    "include": [
      "lodash",        // Установит @types/lodash
      "express",       // Установит @types/express
      "react",         // Установит @types/react
      "node"           // Установит @types/node
    ]
  }
}
```

### exclude - Явное исключение типов

```json
{
  "typeAcquisition": {
    "exclude": [
      "jquery",        // Не будет устанавливать @types/jquery
      "bootstrap"      // Не будет устанавливать @types/bootstrap
    ]
  }
}
```

### typeRoots - Директории для типов

```json
{
  "typeAcquisition": {
    "typeRoots": [
      "./node_modules/@types",    // Стандартные типы
      "./src/custom-types",       // Пользовательские типы
      "./typings"                 // Альтернативные типы
    ]
  }
}
```

## Практические примеры

### 1. Конфигурация для веб-приложения

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "lib": ["dom", "es2020"],
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "typeAcquisition": {
    "enable": true,
    "include": [
      "react",
      "react-dom",
      "react-router-dom",
      "axios",
      "lodash",
      "moment",
      "styled-components"
    ],
    "exclude": [
      "jquery"  // Исключаем, если не используем
    ]
  },
  "include": [
    "src/**/*"
  ]
}
```

### 2. Конфигурация для Node.js приложения

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "lib": ["es2020"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "typeAcquisition": {
    "enable": true,
    "include": [
      "express",
      "mongoose",
      "jsonwebtoken",
      "bcrypt",
      "cors",
      "helmet",
      "winston",
      "node"
    ],
    "exclude": [
      "browser-specific-types"
    ]
  },
  "include": [
    "src/**/*"
  ]
}
```

### 3. Конфигурация для монорепозитория

```json
// Корневой tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/frontend" },
    { "path": "./packages/backend" },
    { "path": "./packages/shared" }
  ],
  "typeAcquisition": {
    "enable": true,
    "include": [
      "node"
    ]
  }
}
```

```json
// packages/frontend/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "target": "es2020",
    "module": "esnext",
    "jsx": "react-jsx"
  },
  "typeAcquisition": {
    "enable": true,
    "include": [
      "react",
      "react-dom",
      "react-router-dom",
      "styled-components"
    ]
  }
}
```

```json
// packages/backend/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "target": "es2020",
    "module": "commonjs"
  },
  "typeAcquisition": {
    "enable": true,
    "include": [
      "express",
      "mongoose",
      "jsonwebtoken"
    ]
  }
}
```

### 4. Скрипты для управления типами

```json
// package.json
{
  "scripts": {
    "types:install": "tsc --installTypes",
    "types:check": "tsc --noEmit --skipLibCheck false",
    "types:clean": "rimraf node_modules/@types",
    "types:update": "npm update @types/*",
    "postinstall": "tsc --installTypes"
  }
}
```

## Интеграция с DefinitelyTyped

### Как работает DefinitelyTyped

DefinitelyTyped - это репозиторий с типами для библиотек JavaScript:

```bash
# Установка типов из DefinitelyTyped
npm install @types/lodash
npm install @types/express
npm install @types/react
```

### Автоматическая установка типов

```json
{
  "typeAcquisition": {
    "enable": true
  }
}
```

При включенной опции TypeScript автоматически:

1. Анализирует импорты в коде
2. Ищет соответствующие типы в @types/*
3. Предлагает установить недостающие типы

### Пример автоматической установки

```typescript
// src/index.ts
import * as _ from 'lodash';
import express from 'express';

// При запуске tsc с включенным typeAcquisition
// TypeScript предложит установить:
// npm install @types/lodash @types/express
```

### Конфигурация для DefinitelyTyped

```json
{
  "typeAcquisition": {
    "enable": true,
    "typeRoots": [
      "./node_modules/@types"
    ]
  },
  "compilerOptions": {
    "types": [
      "node",
      "jest"
    ]
  }
}
```

## Расширенные возможности

### 1. Пользовательские типы

```json
{
  "typeAcquisition": {
    "enable": true,
    "typeRoots": [
      "./node_modules/@types",
      "./src/types"
    ]
  }
}
```

```typescript
// src/types/my-custom-library/index.d.ts
declare module 'my-custom-library' {
  export function doSomething(input: string): number;
  export const VERSION: string;
}

// src/index.ts
import { doSomething, VERSION } from 'my-custom-library';
```

### 2. Условное получение типов

```json
{
  "typeAcquisition": {
    "enable": true,
    "include": [
      "react"
    ],
    "exclude": [
      "jquery"
    ]
  },
  "compilerOptions": {
    "types": [
      "node"  // Только определенные типы
    ]
  }
}
```

### 3. Кэширование типов

```json
{
  "typeAcquisition": {
    "enable": true,
    "cache": true,
    "cacheDirectory": "./.tscache/types"
  }
}
```

### 4. Мониторинг типов

```bash
# Проверка установленных типов
tsc --listFilesOnly | grep types

# Анализ зависимостей типов
tsc --showConfig | jq '.typeAcquisition'

# Установка типов без компиляции
tsc --noEmit --installTypes
```

## Лучшие практики

### 1. Явное управление типами

```json
// Хорошо - явное управление
{
  "typeAcquisition": {
    "enable": true,
    "include": [
      "react",
      "express",
      "lodash"
    ],
    "exclude": [
      "jquery"  // Явно исключаем ненужные типы
    ]
  }
}

// Плохо - полагаемся только на автоматику
{
  "typeAcquisition": {
    "enable": true
    // Нет явного управления
  }
}
```

### 2. Организация пользовательских типов

```bash
# Хорошая структура
project/
├── src/
│   ├── types/
│   │   ├── custom-library/
│   │   │   └── index.d.ts
│   │   ├── global/
│   │   │   └── index.d.ts
│   │   └── index.d.ts
├── node_modules/
│   └── @types/
└── tsconfig.json
```

```json
// tsconfig.json
{
  "typeAcquisition": {
    "enable": true,
    "typeRoots": [
      "./node_modules/@types",
      "./src/types"
    ]
  }
}
```

### 3. Версионирование типов

```json
// package.json
{
  "dependencies": {
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "@types/lodash": "^4.14.182"  // Соответствует версии lodash
  }
}
```

### 4. Тестирование типов

```typescript
// src/__tests__/types.test.ts
import * as _ from 'lodash';
import express from 'express';

// Тесты типов
const result: number = _.add(1, 2); // Проверка типов
const app: express.Application = express(); // Проверка типов
```

## Распространенные ошибки

### 1. Конфликт версий типов

```json
// Проблема: конфликт версий
{
  "dependencies": {
    "lodash": "^3.0.0"
  },
  "devDependencies": {
    "@types/lodash": "^4.0.0"  // Несовместимая версия
  }
}

// Исправление:
{
  "dependencies": {
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "@types/lodash": "^4.14.182"  // Совместимая версия
  }
}
```

### 2. Отсутствие typeRoots

```json
// Проблема: пользовательские типы не находятся
{
  "typeAcquisition": {
    "enable": true
    // Нет typeRoots для пользовательских типов
  }
}

// Исправление:
{
  "typeAcquisition": {
    "enable": true,
    "typeRoots": [
      "./node_modules/@types",
      "./src/types"
    ]
  }
}
```

### 3. Неправильное исключение типов

```json
// Проблема: исключение нужных типов
{
  "typeAcquisition": {
    "exclude": [
      "react"  // Исключает нужные типы
    ]
  }
}

// Исправление:
{
  "typeAcquisition": {
    "exclude": [
      "jquery"  // Исключаем только ненужные типы
    ],
    "include": [
      "react"   // Явно включаем нужные типы
    ]
  }
}
```

### 4. Проблемы с кэшированием

```json
// Проблема: устаревший кэш типов
{
  "typeAcquisition": {
    "cache": true,
    "cacheDirectory": "./.tscache"
  }
}

// Решение: очистка кэша
// rimraf .tscache
// Или отключение кэширования при проблемах
{
  "typeAcquisition": {
    "cache": false
  }
}
```

## Инструменты для работы с типами

### 1. Анализ зависимостей типов

```bash
# Показ всех установленных типов
npm list | grep @types

# Анализ зависимостей
npx npm-check

# Поиск типов для библиотеки
npm search @types/lodash
```

### 2. Управление типами

```bash
# Установка типов
npm install @types/lodash

# Обновление типов
npm update @types/*

# Удаление типов
npm uninstall @types/jquery

# Проверка типов без компиляции
tsc --noEmit --skipLibCheck false
```

### 3. Генерация типов

```bash
# Генерация типов из JavaScript
tsc --allowJs --declaration --emitDeclarationOnly

# Генерация типов из JSON схемы
npx json2ts schema.json > types.d.ts
```

### 4. Мониторинг типов

```bash
# Мониторинг изменений типов
tsc --watch --listEmittedFiles | grep types

# Анализ размера типов
npx bundlephobia-cli @types/lodash

# Сравнение версий типов
npm outdated | grep @types
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[ts/configuration/basic-setup|Базовая настройка]] - Начальная конфигурация
- [[ts/advanced-modules/ambient-modules|Внешние модули]] - Объявления типов для библиотек
- [[ts/type-system/interfaces|Интерфейсы]] - Определение форм типов
- [[ts/utility-types/Утилиты_типов|Утилиты типов]] - Встроенные типы для упрощения объявления

## Рекомендации по изучению

1. Начните с базовой конфигурации typeAcquisition
2. Освойте интеграцию с DefinitelyTyped
3. Практикуйтесь в создании пользовательских типов
4. Изучите инструменты для управления типами
5. Освойте лучшие практики версионирования типов
6. Практикуйтесь в устранении конфликтов типов

## Следующие шаги

- [[ts/configuration/performance|Оптимизация производительности]] - Настройки для ускорения компиляции
- [[ts/advanced-modules/ambient-modules|Внешние модули]] - Объявления типов для библиотек
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев
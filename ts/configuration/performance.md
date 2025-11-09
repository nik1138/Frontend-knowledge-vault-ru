# Оптимизация производительности TypeScript

Оптимизация производительности TypeScript включает настройку компилятора, структуры проекта и инструментов разработки для достижения максимальной скорости компиляции и проверки типов.

## Содержание

1. [Основы оптимизации производительности](#основы-оптимизации-производительности)
2. [Конфигурация компилятора](#конфигурация-компилятора)
3. [Инкрементальная сборка](#инкрементальная-сборка)
4. [Практические примеры](#практические-примеры)
5. [Инструменты профилирования](#инструменты-профилирования)
6. [Лучшие практики](#лучшие-практики)
7. [Распространенные ошибки](#распространенные-ошибки)

## Основы оптимизации производительности

Оптимизация производительности TypeScript направлена на ускорение процессов компиляции, проверки типов и сборки проекта.

### Проблемы производительности

```bash
# Медленная компиляция
tsc --build  # 30 секунд

# Быстрая компиляция
tsc --build  # 3 секунды (10x ускорение)
```

### Метрики производительности

```bash
# Базовая диагностика
tsc --diagnostics

# Расширенная диагностика
tsc --extendedDiagnostics

# Профилирование CPU
tsc --generateCpuProfile profile.cpuprofile
```

Пример вывода диагностики:
```
Files:            1000
Lines:           50000
Nodes:          200000
Identifiers:     50000
Symbols:         30000
Types:           15000
Memory used:    200000K
I/O read:         0.50s
I/O write:        0.20s
Parse time:       2.00s
Bind time:        1.00s
Check time:       5.00s
Emit time:        1.00s
Total time:       9.00s
```

## Конфигурация компилятора

### Оптимизированные опции компилятора

```json
{
  "compilerOptions": {
    // Базовые оптимизации
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo",
    
    // Пропуск проверок для ускорения
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    
    // Ограничение проверок типов
    "noUnusedLocals": false,        // Отключить для скорости
    "noUnusedParameters": false,    // Отключить для скорости
    "noImplicitReturns": false,     // Отключить для скорости
    
    // Оптимизация вывода типов
    "assumeChangesOnlyAffectDirectDependencies": true,
    
    // Кэширование
    "disableSizeLimit": false,
    "maxNodeModuleJsDepth": 0
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority"
  }
}
```

### Настройка для разработки

```json
// tsconfig.dev.json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "sourceMap": true,
    "outDir": "./dist/dev",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    
    // Оптимизации для разработки
    "incremental": true,
    "tsBuildInfoFile": "./dist/dev/.tsbuildinfo",
    "skipLibCheck": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noImplicitReturns": false
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "excludeDirectories": [
      "**/node_modules",
      "dist",
      ".git"
    ]
  }
}
```

### Настройка для продакшена

```json
// tsconfig.prod.json
{
  "compilerOptions": {
    "target": "es2018",
    "module": "commonjs",
    "sourceMap": false,
    "removeComments": true,
    "outDir": "./dist/prod",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    
    // Полные проверки для продакшена
    "incremental": false,
    "skipLibCheck": false,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

## Инкрементальная сборка

### Настройка инкрементальной сборки

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  }
}
```

### Преимущества инкрементальной сборки

```bash
# Полная сборка
tsc --build  # 10.0s

# Инкрементальная сборка (после небольших изменений)
tsc --build  # 0.5s (20x ускорение)
```

### Композитные проекты для инкрементальной сборки

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

```json
// packages/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  },
  "include": ["src/**/*"]
}
```

### Скрипты для инкрементальной сборки

```json
{
  "scripts": {
    "build": "tsc --build",
    "build:clean": "tsc --build --clean",
    "build:watch": "tsc --build --watch",
    "build:force": "tsc --build --force",
    "typecheck": "tsc --build --dry",  // Только проверка типов
    "typecheck:watch": "tsc --build --watch --dry"
  }
}
```

## Практические примеры

### 1. Оптимизация большого проекта

```bash
# Структура большого проекта
large-project/
├── packages/
│   ├── shared/
│   ├── frontend/
│   ├── backend/
│   ├── mobile/
│   └── admin/
├── tsconfig.json
└── package.json
```

```json
// tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/frontend" },
    { "path": "./packages/backend" },
    { "path": "./packages/mobile" },
    { "path": "./packages/admin" }
  ],
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo",
    "assumeChangesOnlyAffectDirectDependencies": true
  }
}
```

```json
// packages/shared/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "tsBuildInfoFile": "./dist/.tsbuildinfo",
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": [
    "**/*.test.ts",
    "**/*.spec.ts"
  ]
}
```

### 2. Оптимизация для CI/CD

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
            ~/.npm
          key: ${{ runner.os }}-build-${{ hashFiles('**/package-lock.json') }}
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm ci
      - run: npm run build
      - run: npm run typecheck
```

```json
// package.json для CI/CD
{
  "scripts": {
    "prebuild": "rimraf dist",
    "build": "tsc --build --verbose",
    "typecheck": "tsc --build --dry --verbose",
    "postbuild": "echo 'Build completed successfully'"
  }
}
```

### 3. Оптимизация для разработки

```json
// tsconfig.dev.json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "esnext",
    "sourceMap": true,
    "outDir": "./dist/dev",
    "rootDir": "./src",
    "strict": true,
    
    // Оптимизации для разработки
    "incremental": true,
    "tsBuildInfoFile": "./dist/dev/.tsbuildinfo",
    "skipLibCheck": true,
    "skipDefaultLibCheck": true,
    "assumeChangesOnlyAffectDirectDependencies": true
  },
  "watchOptions": {
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",
    "fallbackPolling": "dynamicPriority",
    "excludeDirectories": [
      "**/node_modules",
      "dist/dev",
      ".git",
      ".cache"
    ]
  },
  "include": ["src/**/*"],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
    "**/*.spec.ts"
  ]
}
```

## Инструменты профилирования

### 1. Встроенные инструменты TypeScript

```bash
# Базовая диагностика
tsc --diagnostics

# Расширенная диагностика
tsc --extendedDiagnostics

# Профилирование CPU
tsc --generateCpuProfile profile.cpuprofile

# Только проверка типов (без эмита)
tsc --noEmit

# Проверка с подробным выводом
tsc --listFiles
```

### 2. Анализ производительности

```bash
# Анализ времени компиляции
time tsc --build

# Мониторинг использования памяти
tsc --diagnostics | grep "Memory used"

# Анализ количества файлов
tsc --listFiles | wc -l
```

### 3. Внешние инструменты

```bash
# Анализ зависимостей
npx madge --circular src/

# Анализ размера бандла
npx bundlephobia-cli my-package

# Анализ производительности
npx clinic doctor -- node dist/index.js
```

### 4. Профилирование в Chrome DevTools

```bash
# Генерация профиля CPU
tsc --generateCpuProfile profile.cpuprofile

# Открытие в Chrome DevTools
# chrome://inspect -> Open dedicated DevTools for Node -> Profiler
```

## Лучшие практики

### 1. Структура проекта

```bash
# Хорошая структура для производительности
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

### 2. Исключение ненужных файлов

```json
{
  "compilerOptions": {
    "skipLibCheck": true,
    "skipDefaultLibCheck": true
  },
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/__tests__/**",
    "**/__mocks__/**",
    "**/*.d.ts"
  ]
}
```

### 3. Оптимизация импортов

```typescript
// Плохо - импорт всего модуля
import * as _ from 'lodash';
const result = _.chunk([1, 2, 3, 4], 2);

// Хорошо - импорт только нужного
import chunk from 'lodash/chunk';
const result = chunk([1, 2, 3, 4], 2);

// Ещё лучше - tree shaking
import { chunk } from 'lodash';
const result = chunk([1, 2, 3, 4], 2);
```

### 4. Кэширование в CI/CD

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
            ~/.npm
          key: ${{ runner.os }}-build-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-
```

## Распространенные ошибки

### 1. Отключение важных проверок

```json
// Проблема: отключение всех проверок
{
  "compilerOptions": {
    "strict": false,
    "skipLibCheck": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false
  }
}

// Лучше: селективная оптимизация
{
  "compilerOptions": {
    "strict": true,           // Оставляем строгую типизацию
    "skipLibCheck": true,     // Отключаем только проверку библиотек
    "noUnusedLocals": false,  // Отключаем для скорости разработки
    "noUnusedParameters": false
  }
}
```

### 2. Неправильная конфигурация инкрементальной сборки

```json
// Проблема: кэш в той же директории, что и исходники
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

### 3. Избыточное включение файлов

```json
// Проблема: включение всех файлов
{
  "include": ["**/*"]  // Включает node_modules, dist и т.д.
}

// Исправление:
{
  "include": ["src/**/*"],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts"
  ]
}
```

### 4. Проблемы с композитными проектами

```json
// Проблема: отсутствие обязательных опций
{
  "compilerOptions": {
    // Нет "composite": true
    // Нет "declaration": true
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

## Мониторинг производительности

### 1. Регулярный мониторинг

```bash
# Ежедневный мониторинг
npm run build -- --extendedDiagnostics

# Сравнение производительности
# До оптимизации: 15.2s
# После оптимизации: 3.8s (4x ускорение)
```

### 2. Метрики для отслеживания

```bash
# Ключевые метрики
tsc --extendedDiagnostics | grep -E "(Files|Memory used|Total time)"
```

### 3. Автоматизированный мониторинг

```javascript
// performance-monitor.js
const { execSync } = require('child_process');

function measureBuildTime() {
  const start = Date.now();
  execSync('tsc --build', { stdio: 'inherit' });
  const end = Date.now();
  
  const duration = (end - start) / 1000;
  console.log(`Build time: ${duration}s`);
  
  // Отправка метрик в систему мониторинга
  // sendMetrics({ buildTime: duration });
}

measureBuildTime();
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[ts/configuration/build-modes|Режимы сборки]] - Инкрементальная и композитная сборка
- [[ts/configuration/watch-options|Опции наблюдения]] - Настройка режима разработки
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев

## Рекомендации по изучению

1. Начните с базовой оптимизации конфигурации компилятора
2. Освойте инкрементальную сборку и композитные проекты
3. Практикуйтесь в использовании инструментов профилирования
4. Изучите оптимизацию для CI/CD
5. Освойте лучшие практики структуры проекта
6. Практикуйтесь в устранении распространенных ошибок

## Следующие шаги

- [[ts/react/React_с_TypeScript|TypeScript с React]] - Типобезопасная разработка React-приложений
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Общие принципы оптимизации
- [[ts/architecture/monorepo|Монорепозитории]] - Архитектура монорепозиториев
- [[ts/testing/index|Тестирование TypeScript]] - Тестирование приложений
# Управление пакетами в TypeScript

Управление пакетами в TypeScript - это процесс организации, установки, обновления и публикации зависимостей в проектах TypeScript. Это важная часть экосистемы TypeScript, обеспечивающая переиспользование кода и модульность.

## Основы управления пакетами

### Пакетные менеджеры

TypeScript проекты могут использовать различные пакетные менеджеры:

- **npm** - самый популярный менеджер пакетов для JavaScript/TypeScript
- **yarn** - быстрый, надежный альтернативный менеджер пакетов
- **pnpm** - эффективный по использованию дискового пространства менеджер
- **Bun** - новый универсальный инструмент, включающий свой менеджер пакетов

### package.json и TypeScript

TypeScript проекты используют стандартный файл package.json для определения зависимостей:

```json
{
  "name": "my-typescript-project",
  "version": "1.0.0",
  "description": "TypeScript проект с управлением пакетами",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "dev": "ts-node src/index.ts",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0",
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "@types/express": "^4.17.13",
    "typescript": "^4.8.0",
    "ts-node": "^10.0.0",
    "jest": "^29.0.0",
    "@types/jest": "^29.0.0"
  }
}
```

## Типизированные зависимости

### @types пакеты

Для JavaScript библиотек, не содержащих встроенных типов, TypeScript использует пакеты @types:

```bash
# Установка типов для внешней библиотеки
npm install --save-dev @types/lodash
```

Эти пакеты содержат файлы объявления типов (.d.ts), обеспечивающие типобезопасность при работе с JavaScript библиотеками.

### Создание типизированных пакетов

При написании пакетов на TypeScript, важно правильно настроить экспорт типов:

```json
{
  "name": "my-typescript-library",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist/**/*"
  ],
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  }
}
```

## Лучшие практики управления пакетами

### Спецификации версий

Использование правильных спецификаторов версий для стабильности:

```json
{
  "dependencies": {
    "express": "^4.18.0",     // Совместимые версии (4.x.x)
    "lodash": "~4.17.21",     // Патч-обновления (4.17.x)
    "typescript": "4.8.0"     // Точная версия
  }
}
```

### Управление зависимостями

Разделение зависимостей на производственные и разработческие:

- **dependencies** - необходимые в продакшене пакеты
- **devDependencies** - используемые только для разработки (тесты, сборка, линтинг)
- **peerDependencies** - ожидаемые в проекте-пользователе пакеты (например, для библиотек)

### Работа с устаревшими пакетами

При работе с пакетами без типов:

```typescript
// Для пакетов без типов можно создать объявление типов
declare module 'some-untyped-package' {
  export function someFunction(): string;
}

// Или использовать import type из js
import someModule = require('some-untyped-package');
```

## Работа с реестрами пакетов

### Публичные реестры

- **npm registry** - основной реестр для npm пакетов
- **GitHub Packages** - реестр, интегрированный с GitHub
- **GitLab Packages** - реестр, интегрированный с GitLab

### Частные реестры

Для корпоративных проектов могут использоваться частные реестры:

```bash
# Настройка частного реестра
npm set @mycompany:registry https://npm.mycompany.com
```

## Современные тенденции

### Monorepos

Монорепозитории меняют подход к управлению пакетами:

```json
{
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

### ESM и типы

Поддержка ES модулей в TypeScript:

```json
{
  "type": "module",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs"
    }
  }
}
```

## Безопасность зависимостей

### Аудит зависимостей

```bash
# Проверка безопасности зависимостей
npm audit
npm audit fix
```

### Управление уязвимостями

Проверка уязвимостей в зависимостях и обновление устаревших пакетов.

## Интеграция с TypeScript

### Файлы объявлений

Пакеты на TypeScript автоматически включают свои .d.ts файлы:

```typescript
// При импорте пакета TypeScript автоматически находит связанные типы
import { SomeType } from 'some-pkg';

// Типы SomeType будут доступны благодаря файлам объявлений
const value: SomeType = { /* ... */ };
```

### Conditional Exports и типы

Современные пакеты используют conditional exports:

```json
{
  "exports": {
    ".": {
      "import": {
        "types": "./dist/index.d.mts",
        "default": "./dist/index.mjs"
      },
      "require": {
        "types": "./dist/index.d.cts",
        "default": "./dist/index.cjs"
      }
    }
  }
}
```

## Связь с другими концепциями

- [[TypeScript Module Systems]] - модульная система TypeScript
- [[configuration/index]] - настройка TypeScript для работы с пакетами
- [[tooling-workflows/index]] - инструменты и рабочие процессы
- [[deployment/targets]] - деплой пакетов
- [[ecosystem/build-tools/typescript-compiler]] - компиляция пакетов с типами
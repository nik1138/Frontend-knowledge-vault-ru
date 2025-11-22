# Базовая настройка TypeScript

Базовая настройка TypeScript включает создание и конфигурацию файла `tsconfig.json`, который определяет основные параметры компилятора и структуру проекта. Правильная базовая настройка обеспечивает эффективную разработку и поддержку кода.

## Содержание

1. [Создание tsconfig.json](#создание-tsconfigjson)
2. [Основные параметры компилятора](#основные-параметры-компилятора)
3. [Структура проекта](#структура-проекта)
4. [Практические примеры](#практические-примеры)
5. [Распространенные ошибки](#распространенные-ошибки)
6. [Лучшие практики](#лучшие-практики)

## Создание tsconfig.json

### Инициализация через CLI

Самый простой способ создать базовый `tsconfig.json`:

```bash
# Создание базового файла конфигурации
tsc --init
```

Эта команда создаст файл с комментариями и рекомендуемыми настройками:

```json
{
  "compilerOptions": {
    /* Visit https://aka.ms/tsconfig.json to read more about this file */

    /* Basic Options */
    // "incremental": true,                   /* Enable incremental compilation */
    "target": "es5",                          /* Specify ECMAScript target version: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', 'ES2018', 'ES2019', 'ES2020', or 'ESNEXT'. */
    "module": "commonjs",                     /* Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', 'es2020', or 'ESNext'. */
    // "lib": [],                             /* Specify library files to be included in the compilation. */
    // "allowJs": true,                       /* Allow javascript files to be compiled. */
    // "checkJs": true,                       /* Report errors in .js files. */
    // "jsx": "preserve",                     /* Specify JSX code generation: 'preserve', 'react-native', or 'react'. */
    // "declaration": true,                   /* Generates corresponding '.d.ts' file. */
    // "declarationMap": true,                /* Generates a sourcemap for each corresponding '.d.ts' file. */
    // "sourceMap": true,                     /* Generates corresponding '.map' file. */
    // "outFile": "./",                       /* Concatenate and emit output to single file. */
    // "outDir": "./",                        /* Redirect output structure to the directory. */
    // "rootDir": "./",                       /* Specify the root directory of input files. Use to control the output directory structure with --outDir. */
    // "composite": true,                     /* Enable project compilation */
    // "tsBuildInfoFile": "./",               /* Specify file to store incremental compilation information */
    // "removeComments": true,                /* Do not emit comments to output. */
    // "noEmit": true,                        /* Do not emit outputs. */
    // "importHelpers": true,                 /* Import emit helpers from 'tslib'. */
    // "downlevelIteration": true,            /* Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'. */
    // "isolatedModules": true,               /* Transpile each file as a separate module (similar to 'ts.transpileModule'). */

    /* Strict Type-Checking Options */
    "strict": true,                           /* Enable all strict type-checking options. */
    // "noImplicitAny": true,                 /* Raise error on expressions and declarations with an implied 'any' type. */
    // "strictNullChecks": true,              /* Enable strict null checks. */
    // "strictFunctionTypes": true,           /* Enable strict checking of function types. */
    // "strictBindCallApply": true,           /* Enable strict 'bind', 'call', and 'apply' methods on functions. */
    // "strictPropertyInitialization": true,  /* Enable strict checking of property initialization in classes. */
    // "noImplicitThis": true,                /* Raise error on 'this' expressions with an implied 'any' type. */
    // "alwaysStrict": true,                  /* Parse in strict mode and emit "use strict" for each source file. */

    /* Additional Checks */
    // "noUnusedLocals": true,                /* Report errors on unused locals. */
    // "noUnusedParameters": true,            /* Report errors on unused parameters. */
    // "noImplicitReturns": true,             /* Report error when not all code paths in function return a value. */
    // "noFallthroughCasesInSwitch": true,    /* Report errors for fallthrough cases in switch statement. */

    /* Module Resolution Options */
    // "moduleResolution": "node",            /* Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6). */
    // "baseUrl": "./",                       /* Base directory to resolve non-absolute module names. */
    // "paths": {},                           /* A series of entries which re-map imports to lookup locations relative to the 'baseUrl'. */
    // "rootDirs": [],                        /* List of root folders whose combined content represents the structure of the project at runtime. */
    // "typeRoots": [],                       /* List of folders to include type definitions from. */
    // "types": [],                           /* Type declaration files to be included in compilation. */
    // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
    "esModuleInterop": true,                  /* Enables emit interoperability between CommonJS and ES Modules via creation of namespace objects for all imports. Implies 'allowSyntheticDefaultImports'. */
    // "preserveSymlinks": true,              /* Do not resolve the real path of symlinks. */
    // "allowUmdGlobalAccess": true,          /* Allow accessing UMD globals from modules. */

    /* Source Map Options */
    // "sourceRoot": "",                      /* Specify the location where debugger should locate TypeScript files instead of source locations. */
    // "mapRoot": "",                         /* Specify the location where debugger should locate map files instead of generated locations. */
    // "inlineSourceMap": true,               /* Emit a single file with source maps instead of having a separate file. */
    // "inlineSources": true,                 /* Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set. */

    /* Experimental Options */
    // "experimentalDecorators": true,        /* Enables experimental support for ES7 decorators. */
    // "emitDecoratorMetadata": true,         /* Enables experimental support for emitting type metadata for decorators. */

    /* Advanced Options */
    "skipLibCheck": true,                     /* Skip type checking of declaration files. */
    "forceConsistentCasingInFileNames": true  /* Disallow inconsistently-cased references to the same file. */
  }
}
```

### Ручное создание

Для большего контроля можно создать файл вручную:

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src"
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

## Основные параметры компилятора

### 1. Целевая версия ECMAScript

```json
{
  "compilerOptions": {
    "target": "es2020" // или "es5", "es2015", "es2018", "esnext"
  }
}
```

### 2. Система модулей

```json
{
  "compilerOptions": {
    "module": "commonjs" // или "esnext", "amd", "umd", "system"
  }
}
```

### 3. Строгая типизация

```json
{
  "compilerOptions": {
    "strict": true // Включает все строгие проверки
  }
}
```

### 4. Совместимость модулей

```json
{
  "compilerOptions": {
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true
  }
}
```

### 5. Директории

```json
{
  "compilerOptions": {
    "outDir": "./dist",    // Директория для скомпилированных файлов
    "rootDir": "./src"     // Корневая директория исходных файлов
  }
}
```

## Структура проекта

### 1. Простая структура

```bash
project/
├── src/
│   ├── index.ts
│   ├── utils/
│   │   └── helpers.ts
│   └── models/
│       └── user.ts
├── dist/
├── package.json
└── tsconfig.json
```

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 2. Структура с тестами

```bash
project/
├── src/
│   ├── index.ts
│   └── utils/
│       └── helpers.ts
├── tests/
│   └── utils/
│       └── helpers.test.ts
├── dist/
├── package.json
└── tsconfig.json
```

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### 3. Много-проектная структура

```bash
monorepo/
├── packages/
│   ├── core/
│   │   ├── src/
│   │   ├── dist/
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── ui/
│   │   ├── src/
│   │   ├── dist/
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── api/
│       ├── src/
│       ├── dist/
│       ├── package.json
│       └── tsconfig.json
├── package.json
└── tsconfig.json
```

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

// packages/core/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "target": "es2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}
```

## Практические примеры

### 1. Конфигурация для Node.js приложения

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "lib": ["es2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true,
    "removeComments": false,
    "importHelpers": true,
    "downlevelIteration": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
    "**/*.spec.ts"
  ]
}
```

### 2. Конфигурация для библиотеки

```json
{
  "compilerOptions": {
    "target": "es2018",
    "module": "esnext",
    "lib": ["es2018"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "importHelpers": true,
    "noEmit": false,
    "noEmitOnError": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts",
    "**/*.spec.ts",
    "**/__tests__/**"
  ]
}
```

### 3. Конфигурация для разработки и продакшена

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

// tsconfig.dev.json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "sourceMap": true,
    "noEmitOnError": false,
    "diagnostics": true
  },
  "include": ["src/**/*"]
}

// tsconfig.prod.json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "target": "es2018",
    "module": "commonjs",
    "sourceMap": false,
    "removeComments": true,
    "noEmitOnError": true
  },
  "include": ["src/**/*"]
}
```

## Распространенные ошибки

### 1. Неправильные пути

```json
// Ошибка: неправильные пути
{
  "compilerOptions": {
    "outDir": "dist",    // Относительный путь без ./
    "rootDir": "src"     // Относительный путь без ./
  }
}

// Исправление:
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

### 2. Конфликтующие настройки

```json
// Ошибка: конфликт module и moduleResolution
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "classic" // Не совместимо с esnext
  }
}

// Исправление:
{
  "compilerOptions": {
    "module": "esnext",
    "moduleResolution": "node" // Совместимо с esnext
  }
}
```

### 3. Отсутствие include/exclude

```json
// Ошибка: все файлы включены в компиляцию
{
  "compilerOptions": {
    "outDir": "./dist"
    // Нет include/exclude - компилируются все .ts файлы
  }
}

// Исправление:
{
  "compilerOptions": {
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Лучшие практики

### 1. Использование строгой типизации

```json
{
  "compilerOptions": {
    "strict": true, // Включает все строгие проверки
    
    // Или по отдельности:
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

### 2. Безопасность кода

```json
{
  "compilerOptions": {
    "noUnusedLocals": true,           // Предупреждение о неиспользуемых переменных
    "noUnusedParameters": true,       // Предупреждение о неиспользуемых параметрах
    "noImplicitReturns": true,        // Все пути кода должны возвращать значение
    "noFallthroughCasesInSwitch": true // Предупреждение о fallthrough в switch
  }
}
```

### 3. Совместимость и производительность

```json
{
  "compilerOptions": {
    "esModuleInterop": true,          // Совместимость ES modules и CommonJS
    "allowSyntheticDefaultImports": true, // Разрешает default imports
    "skipLibCheck": true,             // Пропуск проверки .d.ts файлов для скорости
    "forceConsistentCasingInFileNames": true, // Согласованность регистра имен файлов
    "incremental": true,              // Инкрементальная компиляция
    "tsBuildInfoFile": "./dist/.tsbuildinfo" // Файл для инкрементальной компиляции
  }
}
```

### 4. Отладка и разработка

```json
{
  "compilerOptions": {
    "sourceMap": true,        // Генерация source maps
    "declaration": true,      // Генерация .d.ts файлов
    "declarationMap": true,   // Генерация source maps для .d.ts
    "diagnostics": true,      // Показ диагностики компиляции
    "extendedDiagnostics": true // Расширенная диагностика
  }
}
```

## Инструменты для работы с конфигурацией

### 1. CLI команды

```bash
# Инициализация конфигурации
tsc --init

# Проверка конфигурации
tsc --showConfig

# Компиляция с определенной конфигурацией
tsc -p tsconfig.prod.json

# Компиляция без эмита
tsc --noEmit

# Наблюдение за изменениями
tsc --watch
```

### 2. Анализ производительности

```bash
# Базовая диагностика
tsc --diagnostics

# Расширенная диагностика
tsc --extendedDiagnostics

# Трассировка разрешения модулей
tsc --traceResolution

# Сборка с проектными ссылками
tsc --build
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[ts/configuration/compiler-options|Опции компилятора]] - Подробное описание всех опций
- [[ts/configuration/project-references|Ссылки на проекты]] - Много-проектные конфигурации
- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Настройка путей и разрешения
- [[ts/type-system/interfaces|Интерфейсы]] - Строгая типизация

## Рекомендации по изучению

1. Начните с простой конфигурации для вашего типа проекта
2. Освойте основные параметры компилятора
3. Практикуйтесь в создании конфигураций для разных сред
4. Изучите много-проектные конфигурации
5. Освойте инструменты для анализа и отладки конфигурации
6. Практикуйтесь в оптимизации производительности компиляции

## Следующие шаги

- [[ts/configuration/compiler-options|Опции компилятора]] - Подробное описание всех опций
- [[ts/configuration/project-references|Ссылки на проекты]] - Много-проектные конфигурации
- [[ts/configuration/path-mapping|Сопоставление путей]] - Псевдонимы и разрешение модулей
- [[ts/configuration/build-modes|Режимы сборки]] - Инкрементальная и композитная сборка
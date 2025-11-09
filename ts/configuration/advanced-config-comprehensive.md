# Конфигурация TypeScript: Лучшие практики и расширенные настройки

Конфигурация TypeScript через файл `tsconfig.json` критически важна для обеспечения типобезопасности, производительности компиляции и совместимости с различными средами выполнения. Правильная настройка помогает создавать надежные и поддерживаемые приложения.

## Основные настройки компилятора

### 1. Строгие настройки типизации

```json
{
  "compilerOptions": {
    // Основная настройка строгой типизации
    "strict": true,
    
    // Предотвращение неявного any
    "noImplicitAny": true,
    
    // Проверка null и undefined
    "strictNullChecks": true,
    
    // Проверка типов функций
    "strictFunctionTypes": true,
    
    // Проверка вызовов функций
    "strictBindCallApply": true,
    
    // Проверка инициализации свойств
    "strictPropertyInitialization": true,
    
    // Предотвращение неявного this
    "noImplicitThis": true,
    
    // Всегда использовать strict mode
    "alwaysStrict": true
  }
}
```

### 2. Проверки безопасности

```json
{
  "compilerOptions": {
    // Проверка лишних свойств в объектных литералах
    "noImplicitReturns": true,
    
    // Проверка достижимости кода
    "noFallthroughCasesInSwitch": true,
    
    // Проверка неиспользуемых локальных переменных
    "noUnusedLocals": true,
    
    // Проверка неиспользуемых параметров
    "noUnusedParameters": true,
    
    // Обязательная проверка всех ветвей в операторе switch
    "exactOptionalPropertyTypes": true,
    
    // Проверка совпадения регистра имен файлов
    "forceConsistentCasingInFileNames": true
  }
}
```

## Настройки модулей и импортов

### 1. Конфигурация модульной системы

```json
{
  "compilerOptions": {
    // Тип модульной системы
    "module": "ES2022",
    
    // Как разрешать модули
    "moduleResolution": "node",
    
    // Путь к базовому URL для разрешения неабсолютных модулей
    "baseUrl": ".",
    
    // Псевдонимы для путей
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@types": ["src/types"],
      "@types/*": ["src/types/*"]
    },
    
    // Корни для разрешения модулей
    "rootDirs": ["src", "generated"],
    
    // Разрешение модулей с расширением .json
    "resolveJsonModule": true,
    
    // Разрешение модулей с расширением .js
    "allowJs": false,
    
    // Генерация деклараций для .js файлов
    "checkJs": false
  }
}
```

### 2. Настройки импорта и экспорта

```json
{
  "compilerOptions": {
    // Разрешение синтетических default импортов
    "allowSyntheticDefaultImports": true,
    
    // Импорт интеропа ES
    "esModuleInterop": true,
    
    // Предотвращение эмиттинга при ошибках типов
    "noEmitOnError": true,
    
    // Разрешение циклических зависимостей между файлами
    "allowUmdGlobalAccess": false
  }
}
```

## Настройки вывода и эмиттинга

### 1. Конфигурация вывода

```json
{
  "compilerOptions": {
    // Версия ECMAScript для вывода
    "target": "ES2020",
    
    // Генерировать декларации типов
    "declaration": true,
    
    // Генерировать декларации для .js файлов
    "declarationMap": true,
    
    // Генерировать sourcemaps
    "sourceMap": true,
    
    // Путь для вывода деклараций
    "declarationDir": "./dist/types",
    
    // Путь для вывода sourcemaps
    "mapRoot": "./dist/maps",
    
    // Путь для вывода JavaScript
    "outDir": "./dist",
    
    // Сохранение структуры каталогов
    "preserveConstEnums": true,
    
    // Генерация полной карты исходников
    "inlineSourceMap": false,
    
    // Встраивание исходного кода в sourcemaps
    "inlineSources": false
  }
}
```

### 2. Настройки модулей для вывода

```json
{
  "compilerOptions": {
    // Эмиттинг декораторов метаданных
    "emitDecoratorMetadata": true,
    
    // Эмиттинг декораторов
    "experimentalDecorators": true,
    
    // Удаление комментариев из вывода
    "removeComments": false,
    
    // Создание sourcemaps для JS файлов
    "sourceRoot": "../src",
    
    // Использование строк для именования модулей
    "moduleSuffixes": [".ts", ".js"]
  }
}
```

## Продвинутые настройки

### 1. Оптимизации производительности

```json
{
  "compilerOptions": {
    // Кеширование решений о разрешении модулей
    "moduleResolutionCache": true,
    
    // Использование incremental compilation
    "incremental": true,
    
    // Путь для хранения incremental state
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo",
    
    // Параллельная компиляция (TypeScript 3.4+)
    "composite": true,
    
    // Оптимизация для больших проектов
    "disableSourceOfProjectReferenceRedirect": false
  }
}
```

### 2. Настройки для специфических сред

```json
{
  "compilerOptions": {
    // Для веб-браузеров
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    
    // Для Node.js
    // "lib": ["ES2020"],
    
    // Для веб-воркеров
    // "lib": ["ES2020", "WebWorker"],
    
    // Платформа исполнения
    "types": ["node"], // для Node.js проектов
    
    // Исключение определенных типов
    "typeRoots": ["./node_modules/@types", "./types"]
  }
}
```

## Практические конфигурации для разных типов проектов

### 1. Конфигурация для библиотеки

```json
{
  "compilerOptions": {
    "target": "ES2018",
    "module": "ESNext",
    "lib": ["ES2020"],
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "stripInternal": true,
    "importHelpers": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

### 2. Конфигурация для веб-приложения (React)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "declaration": false,
    "declarationMap": false,
    "sourceMap": true,
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"],
      "@utils/*": ["src/utils/*"],
      "@assets/*": ["src/assets/*"]
    },
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "build", "public"]
}
```

### 3. Конфигурация для Node.js API

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@controllers/*": ["src/controllers/*"],
      "@models/*": ["src/models/*"],
      "@routes/*": ["src/routes/*"],
      "@utils/*": ["src/utils/*"]
    },
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "incremental": true,
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

## Расширенные возможности конфигурации

### 1. Условные компиляции

```json
{
  "compilerOptions": {
    // Для разных целевых платформ
    "types": ["node"],
    
    // Условные пути для разных сред
    "paths": {
      "@platform/*": [
        "src/platform/node/*",
        "src/platform/browser/*"
      ]
    }
  }
}
```

### 2. Конфигурация для монорепозиториев

```json
// tsconfig.base.json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "inlineSources": false,
    "isolatedModules": true,
    "moduleResolution": "node",
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "preserveWatchOutput": true,
    "skipLibCheck": true,
    "strict": true
  },
  "exclude": ["node_modules"]
}
```

```json
// packages/my-package/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "../../dist/packages/my-package",
    "rootDir": "./src"
  },
  "references": [
    { "path": "../shared-package" }
  ],
  "include": ["src/**/*"]
}
```

## Практические примеры настройки

### 1. Настройка для работы с декораторами

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### 2. Настройка для работы с WebAssembly

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "WebWorker"],
    "types": ["emscripten"],
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### 3. Настройка для работы с JSON Schema

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020"],
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true
  },
  "include": ["src/**/*", "schemas/**/*"]
}
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки конфигурации

```json
// ПЛОХО: Слишком много настроек без понимания их смысла
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": false,  // Противоречит strict
    "strictNullChecks": false,  // Противоречит strict
    "allowJs": true,
    "checkJs": true,
    "skipLibCheck": false,
    "forceConsistentCasingInFileNames": false
  }
}

// ХОРОШО: Последовательная конфигурация
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### 2. Лучшие практики настройки

```json
// 1. Используйте extends для наследования базовой конфигурации
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}

// 2. Разделяйте конфигурации для разных целей
// tsconfig.json (базовая)
// tsconfig.node.json (для скриптов сборки)
// tsconfig.app.json (для приложения)
// tsconfig.test.json (для тестов)

// 3. Используйте include/exclude для точного контроля
{
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "**/*.spec.ts",
    "**/*.test.ts"
  ]
}
```

## Проверка и отладка конфигурации

### 1. Инструменты для проверки конфигурации

```bash
# Проверка синтаксиса tsconfig.json
npx tsc --noEmit

# Проверка с отображением подробной информации
npx tsc --noEmit --explainFiles

# Проверка конфигурации с отображением всех настроек
npx tsc --showConfig
```

### 2. Отладка проблем с разрешением модулей

```json
{
  "compilerOptions": {
    "traceResolution": true,  // Включить трассировку разрешения модулей
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

## Примеры реальных проектов

### 1. Конфигурация для Next.js проекта

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["dom", "dom.iterable", "ESNext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": false,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "incremental": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "baseUrl": ".",
    "paths": {
      "@/components/*": ["components/*"],
      "@/pages/*": ["pages/*"],
      "@/utils/*": ["utils/*"],
      "@/types/*": ["types/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

### 2. Конфигурация для Vue.js проекта

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "node",
    "lib": [
      "ESNext",
      "ESNext.AsyncIterable",
      "DOM"
    ],
    "esModuleInterop": true,
    "allowJs": true,
    "sourceMap": true,
    "strict": true,
    "noEmit": true,
    "experimentalDecorators": true,
    "baseUrl": ".",
    "paths": {
      "~/*": ["./*"],
      "@/*": ["src/*"]
    },
    "types": [
      "webpack-env"
    ],
    "resolveJsonModule": true
  },
  "exclude": [
    "dist",
    "node_modules"
  ]
}
```

## Лучшие практики

1. **Используйте строгую типизацию** - включайте `strict` и связанные настройки
2. **Настройте пути к модулям** - используйте `baseUrl` и `paths` для удобства импорта
3. **Используйте наследование конфигураций** - создавайте базовые конфигурации и расширяйте их
4. **Контролируйте эмиттинг** - правильно настраивайте `outDir`, `rootDir`, `include`, `exclude`
5. **Оптимизируйте производительность** - используйте `incremental`, `tsBuildInfoFile`, `composite`
6. **Регулярно обновляйте конфигурацию** - следите за новыми возможностями TypeScript

## Связи с другими концепциями

- [[ts/type-checking/strategies-best-practices-comprehensive|Стратегии проверки типов]] - как настройки влияют на проверку типов
- [[architecture/component-architecture]] - архитектурные аспекты организации кода
- [[js/es6-modules]] - JavaScript модульная система, на которой основаны настройки
- [[ts/modules/organization-comprehensive|Организация модулей]] - как конфигурация влияет на структуру модулей
- [[build-tools/typescript-compiler]] - интеграция с инструментами сборки
---
tags: [typescript, frontend, tooling, linters, build-tools]
aliases: [Инструменты TypeScript, Линтеры TypeScript]
---

# Инструменты и линтеры для TypeScript

## Введение

Эффективная разработка на TypeScript невозможна без правильно настроенных инструментов и линтеров. Они обеспечивают качество кода, соблюдение стандартов, обнаружение ошибок на ранних стадиях и улучшают общую производительность разработки.

## Основные инструменты TypeScript

### TypeScript Compiler (tsc)

TypeScript Compiler - это основной инструмент для компиляции TypeScript в JavaScript.

#### Установка и использование

```bash
# Установка TypeScript
npm install --save-dev typescript

# Инициализация конфигурационного файла
npx tsc --init

# Компиляция проекта
npx tsc

# Компиляция с отслеживанием изменений
npx tsc --watch
```

#### Основные параметры компиляции

```json
{
  "compilerOptions": {
    /* Целевая платформа */
    "target": "ES2020",                    /* Уровень ECMAScript */
    "lib": ["ES2020", "DOM"],             /* Библиотеки для компиляции */
    
    /* Система модулей */
    "module": "ESNext",                   /* Тип модуля */
    "moduleResolution": "node",           /* Стратегия разрешения модулей */
    
    /* Проверки типов */
    "strict": true,                       /* Включить все строгие проверки */
    "noImplicitAny": true,                /* Запретить неявный any */
    "strictNullChecks": true,             /* Строгая проверка null/undefined */
    "strictFunctionTypes": true,          /* Строгая проверка типов функций */
    "noImplicitThis": true,               /* Запретить неявный this */
    "alwaysStrict": true,                 /* Парсить в strict mode */
    
    /* Дополнительные проверки */
    "noUnusedLocals": true,               /* Обнаруживать неиспользуемые локальные переменные */
    "noUnusedParameters": true,           /* Обнаруживать неиспользуемые параметры */
    "noImplicitReturns": true,            /* Обязательный return во всех ветвях */
    "noFallthroughCasesInSwitch": true,   /* Запретить падение сквозь switch */
    
    /* Система модулей */
    "esModuleInterop": true,              /* Позволить интероп ES и CommonJS */
    "allowSyntheticDefaultImports": true, /* Позволить default imports из модулей */
    
    /* Выходные файлы */
    "outDir": "./dist",                   /* Каталог вывода */
    "rootDir": "./src",                   /* Корневой каталог исходников */
    
    /* Дополнительно */
    "forceConsistentCasingInFileNames": true, /* Строгая проверка регистра файлов */
    "skipLibCheck": true,                 /* Пропустить проверку файлов .d.ts */
    "declaration": true,                  /* Генерировать файлы .d.ts */
    "declarationMap": true,               /* Генерировать sourcemaps для .d.ts */
    "sourceMap": true                     /* Генерировать sourcemaps */
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

### TypeScript Language Service

TypeScript Language Service обеспечивает интеллектуальные возможности в редакторах кода.

#### Возможности Language Service

- Автодополнение
- Переход к определению
- Наведение для просмотра типов
- Переименование
- Поиск ссылок
- Быстрые исправления

## Линтеры

### ESLint с TypeScript

ESLint - это мощный линтер, который поддерживает TypeScript через специальные плагины.

#### Установка

```bash
# Установка ESLint и TypeScript плагинов
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

#### Конфигурация .eslintrc.js

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
  },
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    // Дополнительные пресеты
    'plugin:react/recommended',
    'plugin:import/errors',
    'plugin:import/warnings',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 12,
    sourceType: 'module',
    project: './tsconfig.json', // Для проверки типов
  },
  plugins: [
    '@typescript-eslint',
    'react',
    'import',
  ],
  rules: {
    // Правила TypeScript
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'warn',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/consistent-type-definitions': ['error', 'interface'],
    
    // Правила React
    'react/react-in-jsx-scope': 'off', // Не требуется в новых версиях React
    'react/prop-types': 'off', // Не требуется при использовании TypeScript
    
    // Общие правила
    'no-console': 'warn',
    'no-debugger': 'error',
  },
  settings: {
    react: {
      version: 'detect',
    },
    'import/resolver': {
      node: {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
      },
    },
  },
};
```

### Популярные плагины и пресеты

#### @typescript-eslint/eslint-plugin

```js
// .eslintrc.js
module.exports = {
  extends: [
    '@typescript-eslint/recommended',
  ],
  rules: {
    // Строгая типизация
    '@typescript-eslint/no-unused-vars': ['error', { 
      'argsIgnorePattern': '^_', 
      'varsIgnorePattern': '^_' 
    }],
    '@typescript-eslint/strict-boolean-expressions': 'error',
    '@typescript-eslint/prefer-nullish-coalescing': 'error',
    '@typescript-eslint/prefer-optional-chain': 'error',
    
    // Улучшение безопасности
    '@typescript-eslint/no-non-null-assertion': 'error',
    '@typescript-eslint/no-unnecessary-type-assertion': 'error',
  }
};
```

#### eslint-plugin-import

```js
// .eslintrc.js
module.exports = {
  extends: [
    'plugin:import/recommended',
    'plugin:import/typescript',
  ],
  rules: {
    'import/order': [
      'error',
      {
        'groups': [
          'builtin',      // Built-in imports (ex: path)
          'external',     // External imports (ex: react)
          'internal',     // Internal imports (ex: my-component)
          ['sibling', 'parent'], // Sibling and parent imports
          'index',        // Index imports
          'unknown'       // Unknown imports
        ],
        'newlines-between': 'always',
        'alphabetize': {
          'order': 'asc',
          'caseInsensitive': true
        }
      }
    ],
    'import/no-unresolved': 'error',
    'import/no-cycle': 'error',
  }
};
```

## Форматирование кода

### Prettier

Prettier - это инструмент автоматического форматирования кода, который работает с TypeScript.

#### Установка

```bash
npm install --save-dev prettier
```

#### Конфигурация .prettierrc

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid"
}
```

#### Интеграция с ESLint

```bash
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

```js
// .eslintrc.js
module.exports = {
  extends: [
    '@typescript-eslint/recommended',
    'plugin:prettier/recommended', // Добавляет prettier в ESLint
  ],
  rules: {
    // Дополнительные правила
  }
};
```

## Системы сборки и инструменты

### TypeScript с Webpack

```js
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader', // или 'babel-loader' с @babel/preset-typescript
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

### TypeScript с Vite

```ts
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
        },
      },
    },
  },
})
```

## Практические примеры настройки для frontend проектов

### Настройка для React проекта

```bash
# Установка зависимостей
npm install --save-dev typescript @types/react @types/react-dom @types/node
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

```json
// package.json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint src --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "format": "prettier --write src/**/*.{ts,tsx,js,jsx,json,md}"
  }
}
```

```json
// tsconfig.json для React
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Настройка для Node.js API

```json
// tsconfig.json для Node.js
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
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

## Автоматизация и CI/CD

### Husky и lint-staged для пре-коммит хуков

```bash
npm install --save-dev husky lint-staged
```

```json
// package.json
{
  "scripts": {
    "lint-staged": "lint-staged"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{js,jsx,json,md}": [
      "prettier --write"
    ]
  }
}
```

```bash
# Установка husky
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

### GitHub Actions для проверки типов

```yaml
# .github/workflows/type-check.yml
name: Type Check

on: [push, pull_request]

jobs:
  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - name: Install dependencies
        run: npm ci
      - name: Type check
        run: npx tsc --noEmit
```

## Полезные VS Code расширения

- **ESLint** - интеграция ESLint с VS Code
- **Prettier** - форматирование кода
- **TypeScript Importer** - автоматический импорт
- **Path Intellisense** - автодополнение путей
- **Bracket Pair Colorizer** - цветовое выделение скобок

## Заключение

Правильная настройка инструментов и линтеров для TypeScript критически важна для обеспечения качества кода и эффективной разработки. Комбинация TypeScript Compiler, ESLint, Prettier и других инструментов создает мощную систему, которая помогает разработчикам создавать типобезопасный, чистый и поддерживаемый код.

> [!tip] Совет
> Используйте пре-коммит хуки для автоматической проверки и форматирования кода перед каждым коммитом.

> [!warning] Важно
> Не забывайте регулярно обновлять конфигурации инструментов и следить за совместимостью версий.

## Связанные темы

- [[Модули и системы сборки]]
- [[Утилиты типов TypeScript]]
- [[Практические примеры для frontend разработки]]
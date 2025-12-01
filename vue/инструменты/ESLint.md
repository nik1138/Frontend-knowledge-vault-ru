---
aliases: [ESLint, Linter, Code Quality]
tags: [vue, инструменты-разработки, linting, code-quality, best-practices]
---

# ESLint

## Обзор

ESLint — это мощный инструмент статического анализа кода (линтер), который помогает находить и исправлять проблемы в JavaScript и Vue.js коде. Он проверяет код на соответствие заданным правилам и стандартам, обеспечивая консистентность и качество кода в проекте.

В 2025 году ESLint остается незаменимым инструментом в процессе разработки Vue.js приложений, особенно в командной разработке.

## Установка и настройка

Установка ESLint в проект:

```bash
npm install -D eslint
# или
yarn add -D eslint
```

Для Vue.js проектов рекомендуется использовать специальные плагины:

```bash
npm install -D eslint-plugin-vue @vue/eslint-config-standard
# или
yarn add -D eslint-plugin-vue @vue/eslint-config-standard
```

Инициализация конфигурации ESLint:

```bash
npx eslint --init
```

## Базовая конфигурация

Файл `.eslintrc.js` для Vue.js проекта:

```javascript
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-essential', // или vue3-recommended
    '@vue/eslint-config-standard'
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    parser: '@babel/eslint-parser'
  },
  plugins: [
    'vue'
  ],
  rules: {
    // Пользовательские правила
    'vue/multi-word-component-names': 'off',
    'vue/no-unused-vars': 'error',
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off'
  },
  overrides: [
    {
      files: ['*.vue'],
      rules: {
        'vue/script-indent': ['error', 2, { baseIndent: 1 }]
      }
    }
  ]
}
```

## Конфигурация для TypeScript

Для проектов с TypeScript:

```javascript
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    '@vue/typescript/recommended',
    '@vue/eslint-config-standard'
  ],
  parser: 'vue-eslint-parser',
  parserOptions: {
    parser: '@typescript-eslint/parser',
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: [
    'vue',
    '@typescript-eslint'
  ],
  rules: {
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    'vue/multi-word-component-names': 'off'
  }
}
```

## Основные правила для Vue.js

### Правила для компонентов

```javascript
module.exports = {
  rules: {
    // Запрет на использование анонимных компонентов
    'vue/component-definition-name-casing': ['error', 'PascalCase'],
    
    // Требование описания пропсов
    'vue/require-prop-types': 'error',
    
    // Проверка обязательных пропсов
    'vue/require-default-prop': 'error',
    
    // Проверка корректного использования v-for
    'vue/valid-v-for': 'error',
    
    // Проверка корректного использования v-model
    'vue/valid-v-model': 'error'
  }
}
```

### Правила для шаблонов

```javascript
module.exports = {
  rules: {
    // Проверка корректного использования атрибутов
    'vue/attribute-hyphenation': ['error', 'always'],
    
    // Проверка корректного форматирования атрибутов
    'vue/multiline-html-element-content-newline': 'error',
    
    // Проверка корректного использования директив
    'vue/no-unused-vars': 'error',
    
    // Проверка корректного использования v-bind
    'vue/valid-v-bind': 'error'
  }
}
```

## Интеграция с Prettier

Для совместной работы ESLint и Prettier:

```bash
npm install -D eslint-config-prettier eslint-plugin-prettier
# или
yarn add -D eslint-config-prettier eslint-plugin-prettier
```

Конфигурация `.eslintrc.js`:

```javascript
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    '@vue/eslint-config-standard',
    '@vue/eslint-config-prettier'
  ],
  plugins: [
    'vue',
    'prettier'
  ],
  rules: {
    'prettier/prettier': 'error'
  }
}
```

Файл `.prettierrc.js`:

```javascript
module.exports = {
  semi: false,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 80,
  tabWidth: 2,
  useTabs: false
}
```

## Практические рекомендации для российских разработчиков

### Работа в команде

Для обеспечения консистентности кода в командной разработке:

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    // Использование общих конфигураций
    '@vue/standard',
    // Добавление внутренних правил компании
    './eslint-rules/company-standard.js'
  ],
  rules: {
    // Адаптация под российские стандарты разработки
    'camelcase': ['error', { properties: 'always' }],
    'max-len': ['error', { code: 120 }],
    'lines-between-class-members': ['error', 'always', { exceptAfterSingleLine: true }]
  }
}
```

### Интеграция с CI/CD

Для интеграции с системами непрерывной интеграции:

```json
// package.json
{
  "scripts": {
    "lint": "eslint src --ext .js,.vue --fix",
    "lint:check": "eslint src --ext .js,.vue",
    "lint:staged": "lint-staged"
  },
  "lint-staged": {
    "*.{js,vue}": [
      "eslint --fix",
      "git add"
    ]
  }
}
```

### Использование в IDE

Конфигурация для VS Code (файл `.vscode/settings.json`):

```json
{
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "vue"
  ],
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "dbaeumer.vscode-eslint"
}
```

### Работа с российскими стандартами

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    // Использование российского формата комментариев
    'spaced-comment': ['error', 'always', {
      line: {
        markers: ['/'],
        exceptions: ['-', '+', '*']
      },
      block: {
        balanced: true
      }
    }],
    
    // Проверка корректности именования переменных на русском
    'id-match': ['error', '^[a-zA-Z][a-zA-Z0-9]*$']
  }
}
```

## Популярные плагины для Vue.js

### eslint-plugin-vue

Основной плагин для проверки Vue.js компонентов:

```javascript
module.exports = {
  plugins: [
    'vue'
  ],
  extends: [
    'plugin:vue/vue3-essential',  // или vue3-recommended, vue3-strongly-recommended
  ],
  rules: {
    'vue/html-self-closing': ['error', {
      html: {
        void: 'always',
        normal: 'never',
        component: 'always'
      }
    }]
  }
}
```

### @typescript-eslint/eslint-plugin

Для проектов с TypeScript:

```javascript
module.exports = {
  extends: [
    '@vue/typescript/recommended'
  ],
  rules: {
    '@typescript-eslint/consistent-type-imports': 'error'
  }
}
```

## Автоматическое исправление

ESLint может автоматически исправлять многие проблемы:

```bash
# Исправление всех исправляемых проблем
npx eslint src --ext .js,.vue --fix

# Исправление в VS Code при сохранении файла
```

## Интеграция с Git Hooks

Для автоматической проверки кода перед коммитом:

```bash
npm install -D husky lint-staged
```

Конфигурация в `package.json`:

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,vue}": ["eslint --fix", "git add"]
  }
}
```

## Заключение

ESLint является важным инструментом для обеспечения качества кода в Vue.js проектах. Его правильная настройка помогает избежать многих распространенных ошибок и обеспечивает единообразие кода в командной разработке.

## См. также

- [[Vue-CLI]]
- [[Vite]]
- [[Webpack]]
- [[DevTools]]
- [[Vue-проект]]
- [[Vue-архитектура]]
- [[Vue-производительность]]
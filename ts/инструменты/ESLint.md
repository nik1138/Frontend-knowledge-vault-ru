---
aliases: [ESLint для TypeScript, Линтер TypeScript]
tags: [typescript, инструменты, линтер, eslint, качество-кода]
---

# ESLint для TypeScript

ESLint - это мощный инструмент статического анализа кода, который помогает находить и исправлять проблемы в JavaScript и TypeScript коде. Для TypeScript проектов ESLint особенно важен, так как дополняет встроенную систему типизации дополнительными проверками стиля и потенциальных ошибок.

## Установка ESLint в TypeScript проект

Для начала работы с ESLint в TypeScript проекте, необходимо установить необходимые зависимости:

```bash
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

Также установим сам ESLint, если он еще не установлен:

```bash
npm install --save-dev eslint
```

## Базовая конфигурация

Создадим файл `.eslintrc.js` или `.eslintrc.json` в корне проекта:

```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module',
    project: './tsconfig.json', // Указываем путь к tsconfig.json для проверки типов
  },
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
  ],
  rules: {
    // Переопределение или добавление своих правил
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-unused-vars': 'error',
  },
  env: {
    browser: true,
    node: true,
    es6: true,
  },
};
```

## Важные плагины и расширения

### @typescript-eslint/parser

Этот парсер позволяет ESLint понимать TypeScript синтаксис, включая типы, интерфейсы и другие конструкции.

### @typescript-eslint/eslint-plugin

Плагин предоставляет набор правил, специфичных для TypeScript, таких как:

- `@typescript-eslint/no-unused-vars` - обнаруживает неиспользуемые переменные
- `@typescript-eslint/strict-boolean-expressions` - строгая проверка булевых выражений
- `@typescript-eslint/prefer-readonly` - рекомендует использовать readonly для свойств

## Популярные конфигурации

### TypeScript с React

Для проектов, использующих TypeScript и React:

```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 2020,
    sourceType: 'module',
    project: './tsconfig.json',
  },
  plugins: [
    '@typescript-eslint',
    'react',
    'react-hooks',
  ],
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime', // для React 17+
    'plugin:react-hooks/recommended',
  ],
  settings: {
    react: {
      version: 'detect',
    },
  },
  rules: {
    'react/react-in-jsx-scope': 'off', // не требуется в React 17+
    '@typescript-eslint/no-non-null-assertion': 'warn',
  },
};
```

### Airbnb Style Guide

Для использования конфигурации, соответствующей Airbnb Style Guide:

```bash
npm install --save-dev eslint-config-airbnb eslint-config-airbnb-typescript
```

```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json',
    tsconfigRootDir: __dirname,
    ecmaVersion: 2020,
    sourceType: 'module',
  },
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'airbnb',
    'airbnb/hooks',
    'airbnb-typescript',
  ],
  rules: {
    // Пользовательские правила
  },
};
```

## Интеграция с IDE

ESLint легко интегрируется с большинством популярных редакторов:

- **VSCode**: Установите расширение ESLint
- **WebStorm**: Поддержка встроена
- **Vim/Neovim**: Используйте ALE или coc-eslint

## Автоматизация проверки

Добавьте скрипты в `package.json` для автоматизации:

```json
{
  "scripts": {
    "lint": "eslint . --ext .ts,.tsx",
    "lint:fix": "eslint . --ext .ts,.tsx --fix",
    "lint:check": "eslint . --ext .ts,.tsx --max-warnings 0"
  }
}
```

## Практические советы

- Всегда используйте `parserOptions.project` для полной проверки типов
- Настройте `.eslintignore` для исключения файлов из проверки
- Используйте `overrides` для настройки правил под разные типы файлов
- Регулярно обновляйте конфигурации для получения последних улучшений

## Связанные темы

- [[TypeScript-плагины]]
- [[Редакторы-и-IDE]]
- [[TypeScript-Language-Server]]
- [[Prettier]]

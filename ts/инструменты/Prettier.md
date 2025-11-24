---
aliases: [Форматирование TypeScript, Prettier для TypeScript]
tags: [typescript, инструменты, форматирование, prettier, стиль-кода]
---

# Prettier для TypeScript

Prettier - это инструмент автоматического форматирования кода, который помогает поддерживать единообразный стиль во всем проекте. В контексте TypeScript, Prettier особенно полезен для форматирования как самого TypeScript кода, так и связанных файлов (TSX, JSON, Markdown и др.).

## Установка Prettier

Установите Prettier как dev зависимость:

```bash
npm install --save-dev --save-exact prettier
```

## Базовая конфигурация

Создайте файл конфигурации `.prettierrc` в корне проекта:

```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "avoid"
}
```

### Объяснение параметров

- `semi`: Добавляет точку с запятой в конце выражений
- `trailingComma`: Добавляет запятую после последнего элемента в объектах/массивах
- `singleQuote`: Использует одинарные кавычки вместо двойных
- `printWidth`: Максимальная длина строки (Prettier переносит при необходимости)
- `tabWidth`: Размер отступа в пробелах
- `useTabs`: Использовать табы вместо пробелов (false для пробелов)
- `bracketSpacing`: Пробелы между скобками объектов
- `arrowParens`: Оборачивать один параметр стрелочных функций в скобки

## Интеграция с ESLint

Для совместной работы с ESLint, установите дополнительные пакеты:

```bash
npm install --save-dev eslint-config-prettier eslint-plugin-prettier
```

Обновите конфигурацию ESLint (`.eslintrc.js`):

```javascript
module.exports = {
  extends: [
    'eslint:recommended',
    '@typescript-eslint/recommended',
    'plugin:prettier/recommended', // Добавляет Prettier в качестве ESLint правила
  ],
  plugins: ['@typescript-eslint', 'prettier'],
  rules: {
    'prettier/prettier': 'error', // Форматирование считается ошибкой линтера
  },
};
```

## Файл .prettierignore

Создайте файл `.prettierignore`, чтобы исключить файлы из форматирования:

```
# Исключаем скомпилированные файлы
dist/
build/
node_modules/

# Исключаем файлы конфигурации
*.min.js
*.bundle.js

# Временные файлы
*.tmp
*.temp
```

## Интеграция с IDE

### VSCode

Установите расширение Prettier и настройте `settings.json`:

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### WebStorm

В WebStorm поддержка Prettier встроена:
1. Перейдите в Settings → Languages & Frameworks → JavaScript → Prettier
2. Укажите путь к пакету Prettier
3. Включите опцию "On 'Reformat Code' action"

## Скрипты для форматирования

Добавьте скрипты в `package.json`:

```json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "format:ts": "prettier --write \"**/*.ts\" \"**/*.tsx\"",
    "prepare": "husky install" // если используется husky для git hooks
  }
}
```

## Использование с Git hooks

Для автоматического форматирования перед коммитом, используйте husky и lint-staged:

```bash
npm install --save-dev husky lint-staged
```

В `package.json` добавьте:

```json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx,css,scss,md}": [
      "prettier --write",
      "git add"
    ]
  },
  "scripts": {
    "prepare": "husky install"
  }
}
```

Затем настройте хук:

```bash
npx husky add .husky/pre-commit "npx lint-staged"
```

## Особенности работы с TypeScript

### Форматирование типов

Prettier автоматически форматирует определения типов:

```typescript
// До форматирования
type User = { name: string; age: number; email: string; };

// После форматирования
type User = {
  name: string;
  age: number;
  email: string;
};
```

### Форматирование дженериков

```typescript
// До
const result: Promise<Array<Record<string, any>>> = getComplexData();

// После
const result: Promise<Array<Record<string, any>>> = getComplexData();
```

## Практические советы

- Используйте `--check` флаг для проверки форматирования без изменения файлов
- Объединяйте с ESLint для комплексной проверки кода
- Настройте форматирование при сохранении в редакторе
- Используйте `--write` для форматирования всех файлов проекта
- Рассмотрите использование `.prettierignore` для исключения файлов

## Связанные темы

- [[ESLint]]
- [[TypeScript-плагины]]
- [[Редакторы-и-IDE]]
- [[TypeScript-Language-Server]]

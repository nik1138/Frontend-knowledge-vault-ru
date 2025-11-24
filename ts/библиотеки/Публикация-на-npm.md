---
aliases: ["Публикация библиотек", "npm публикация", "Релиз библиотек"]
tags: [typescript, npm, publishing, libraries]
---

# Публикация-на-npm

## Обзор

Публикация TypeScript библиотек на npm позволяет делиться кодом с сообществом и использовать его в других проектах. Этот процесс требует тщательной подготовки, чтобы обеспечить надежность и удобство использования.

## Подготовка к публикации

### Регистрация на npm

Перед публикацией необходимо зарегистрироваться на npm:

```bash
npm adduser
```

Или если у вас уже есть аккаунт:

```bash
npm login
```

### Настройка package.json

Корректный `package.json` критически важен для публикации:

```json
{
  "name": "@your-username/your-library-name",
  "version": "1.0.0",
  "description": "Описание вашей TypeScript библиотеки",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "module": "dist/esm/index.js",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js",
      "types": "./dist/index.d.ts"
    }
  },
  "files": [
    "dist/**/*",
    "README.md",
    "LICENSE"
  ],
  "scripts": {
    "build": "npm run build:commonjs && npm run build:esm",
    "build:commonjs": "tsc --module commonjs --outDir dist/cjs",
    "build:esm": "tsc --module es2020 --outDir dist/esm",
    "prepublishOnly": "npm run build"
  },
  "keywords": [
    "typescript",
    "library",
    "util",
    "helpers"
  ],
  "author": "Ваше имя <ваш.email@example.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/ваш-аккаунт/ваш-репозиторий.git"
  },
  "homepage": "https://github.com/ваш-аккаунт/ваш-репозиторий#readme",
  "bugs": {
    "url": "https://github.com/ваш-аккаунт/ваш-репозиторий/issues"
  },
  "devDependencies": {
    "typescript": "^4.9.0",
    "@types/node": "^18.0.0"
  },
  "engines": {
    "node": ">=14.0.0"
  }
}
```

## Структура файлов

Убедитесь, что ваша библиотека правильно упакована:

```
your-library/
├── dist/
│   ├── index.js
│   ├── index.d.ts
│   ├── index.js.map
│   ├── core/
│   │   ├── user.js
│   │   ├── user.d.ts
│   │   └── user.js.map
│   └── utils/
│       ├── helpers.js
│       ├── helpers.d.ts
│       └── helpers.js.map
├── src/
│   ├── index.ts
│   ├── core/
│   │   └── user.ts
│   └── utils/
│       └── helpers.ts
├── package.json
├── README.md
├── LICENSE
└── tsconfig.json
```

## Проверка перед публикацией

### Тестирование публикации

Перед публикацией протестируйте библиотеку локально:

```bash
# Проверьте, что сборка проходит успешно
npm run build

# Протестируйте локально с помощью npm link
npm link

# В другом проекте
npm link @your-username/your-library-name
```

### Использование npm pack

Проверьте, что будет упаковано:

```bash
npm pack
```

Это создаст `.tgz` файл, который можно протестировать установкой:

```bash
npm install path/to/your-library-1.0.0.tgz
```

## Публикация

### Основная публикация

```bash
npm publish
```

Для публикации приватного пакета:

```bash
npm publish --access restricted
```

### Публикация с тегом

Для публикации с тегом (например, beta версии):

```bash
npm publish --tag beta
```

Пользователи смогут установить библиотеку с тегом:

```bash
npm install @your-username/your-library-name@beta
```

## Версионирование

Следуйте семантическому версионированию (SemVer):

- **MAJOR** версия (X.0.0) - для несовместимых изменений API
- **MINOR** версия (0.Y.0) - для новых функций, совместимых с предыдущими версиями
- **PATCH** версия (0.0.Z) - для исправлений ошибок

### Увеличение версии

```bash
# Увеличить patch версию (0.0.1 -> 0.0.2)
npm version patch

# Увеличить minor версию (0.1.0 -> 0.2.0)
npm version minor

# Увеличить major версию (1.0.0 -> 2.0.0)
npm version major
```

## Автоматизация публикации

### GitHub Actions

Пример GitHub Actions для автоматической публикации:

```yaml
name: Publish to npm
on:
  release:
    types: [published]
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '16.x'
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm run build
      - run: npm test
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Скрипты автоматизации

```json
{
  "scripts": {
    "release:patch": "npm version patch && npm publish",
    "release:minor": "npm version minor && npm publish",
    "release:major": "npm version major && npm publish",
    "prepublishOnly": "npm run build && npm test"
  }
}
```

## Управление версиями зависимостей

### peerDependencies

Для библиотек рекомендуется использовать `peerDependencies` для основных зависимостей:

```json
{
  "peerDependencies": {
    "typescript": ">=4.0.0",
    "react": ">=16.8.0"
  },
  "peerDependenciesMeta": {
    "react": {
      "optional": true
    }
  }
}
```

### devDependencies

Тестирование с различными версиями зависимостей:

```json
{
  "devDependencies": {
    "@types/react": "^18.0.0",
    "typescript": "^4.9.0",
    "jest": "^29.0.0"
  }
}
```

## Приватные пакеты

Для приватных пакетов на npm:

```bash
# Настройка .npmrc для приватных пакетов
echo "@your-username:registry=https://npm.pkg.github.com" >> .npmrc
echo "//npm.pkg.github.com/:_authToken=YOUR_TOKEN" >> .npmrc
```

## Лучшие практики

1. **Проверка кода**: Используйте ESLint и Prettier
2. **Тестирование**: Пишите unit-тесты
3. **Документация**: Поддерживайте актуальный README
4. **Лицензия**: Включайте файл LICENSE
5. **Changelog**: Поддерживайте CHANGELOG.md

## Связанные темы

- [[Создание-библиотек]]
- [[Типизация-библиотек]]
- [[Совместимость-библиотек]]
- [[Документация-библиотек]]
- [[npm]]
- [[TypeScript]]
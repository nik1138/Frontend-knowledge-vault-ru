---
aliases: [@types, типы-пакеты]
tags: [typescript, typing, external-libraries, packages]
---

# @types

**@types** — это пространство имен npm для пакетов определений типов. Все пакеты определений из [[DefinitelyTyped]] публикуются под этим префиксом, что делает их легко идентифицируемыми и доступными для установки через npm.

## Обзор

Пакеты @types содержат файлы определений типов (`.d.ts`) для библиотек JavaScript, которые не имеют встроенной поддержки TypeScript. Они позволяют использовать проверку типов, автодополнение и навигацию по коду при работе со сторонними библиотеками.

## Установка пакетов @types

Для установки определений типов используйте npm или yarn:

```bash
# Установка определений для конкретной библиотеки
npm install --save-dev @types/react
npm install --save-dev @types/node
npm install --save-dev @types/jest

# Установка определений для библиотеки с версией
npm install --save-dev @types/lodash@4.14.182
```

## Автоматическое разрешение типов

TypeScript автоматически находит и использует пакеты @types при следующих условиях:
- Пакет @types установлен локально
- Имя пакета соответствует импортируемой библиотеке
- В `tsconfig.json` включена опция `types` или `typeRoots`

## Настройка в tsconfig.json

```json
{
  "compilerOptions": {
    "typeRoots": [
      "./node_modules/@types",
      "./types"
    ],
    "types": [
      "node",
      "jest"
    ]
  }
}
```

## Структура пакета @types

Типичный пакет @types включает:
- Основной файл `index.d.ts`
- Дополнительные файлы для различных модулей
- Файл `package.json` с метаданными
- Тесты для проверки корректности определений

## Пример пакета @types

```json
{
  "name": "@types/lodash",
  "version": "4.14.182",
  "description": "TypeScript definitions for Lo-Dash",
  "main": "",
  "types": "index.d.ts",
  "repository": {
    "type": "git",
    "url": "https://github.com/DefinitelyTyped/DefinitelyTyped.git",
    "directory": "types/lodash"
  },
  "scripts": {},
  "dependencies": {},
  "typesPublisherContentHash": "hash...",
  "typeScriptVersion": "3.9"
}
```

## Популярные пакеты @types

- `@types/node` — определения для Node.js
- `@types/react` — определения для React
- `@types/jest` — определения для Jest
- `@types/express` — определения для Express.js
- `@types/lodash` — определения для Lodash

## Создание собственных @types пакетов

Хотя большинство определений типов доступны через [[DefinitelyTyped]], иногда может потребоваться создать собственные определения:

- Для проприетарных библиотек
- Для библиотек без определений в DefinitelyTyped
- Для настройки существующих определений под конкретные нужды

## Связанные темы

- [[DefinitelyTyped]]
- [[Создание-определений]]
- [[Ambient-модули]]
- [[Глобальные-переменные]]
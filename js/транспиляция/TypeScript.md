---
aliases: ["Транспиляция TypeScript", "TS компиляция", "Компилятор TypeScript"]
tags: [typescript, javascript, transpilation, frontend-development, development-tools]
---

# TypeScript

**TypeScript** — это строго типизированный надмножество JavaScript, разработанное Microsoft. Он компилируется (транспилируется) в чистый JavaScript и предоставляет разработчикам возможности статической типизации, интерфейсов, классов и других возможностей, отсутствующих в стандартном JavaScript.

## Основные возможности

- Статическая типизация для обнаружения ошибок на этапе компиляции
- Интерфейсы и типы для описания структуры данных
- Классы с поддержкой модификаторов доступа (public, private, protected)
- Обобщения (generics) для создания гибких компонентов
- Декораторы для расширения функциональности
- Автоматическое определение типов (type inference)

## Установка и настройка

Для установки TypeScript выполните:

```bash
npm install --save-dev typescript
```

### Создание конфигурационного файла

Для генерации файла `tsconfig.json` выполните:

```bash
npx tsc --init
```

### Основные параметры tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    "allowJs": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "jsx": "react-jsx"
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

## Процесс транспиляции

TypeScript компилируется в JavaScript с помощью команды:

```bash
npx tsc
```

Или в режиме наблюдения:

```bash
npx tsc --watch
```

### Целевые версии JavaScript

В параметрах `tsconfig.json` можно указать целевую версию JavaScript:

```json
{
  "compilerOptions": {
    "target": "ES5",     // Поддержка старых браузеров
    "target": "ES2015",  // Современные возможности
    "target": "ESNext"   // Экспериментальные возможности
  }
}
```

В российских условиях часто требуется поддержка ES5 из-за использования устаревших браузеров в корпоративных средах.

## Практические рекомендации

### Работа с существующими JavaScript-проектами

TypeScript можно постепенно внедрять в существующие JavaScript-проекты:

1. Установите TypeScript и настройте `tsconfig.json`
2. Измените расширения файлов с `.js` на `.ts`
3. Используйте `// @ts-ignore` для проблемных участков кода
4. Пошагово добавляйте типы к функциям и переменным

### Опции компиляции для продакшена

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "removeComments": true,
    "sourceMap": false,
    "declaration": true,
    "declarationMap": false
  }
}
```

### Опции для разработки

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "sourceMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

## Особенности использования в России

В 2025 году в российской разработке TypeScript занимает все более прочные позиции:

1. **Корпоративная разработка**: Крупные российские компании (Яндекс, Mail.Ru Group, Сбер и др.) активно используют TypeScript для масштабируемых приложений, что делает знание языка важным навыком для разработчиков.

2. **Государственные проекты**: При разработке государственных информационных систем TypeScript помогает обеспечить надежность и поддерживаемость кода.

3. **Образование**: Все больше российских вузов и курсов обучения программированию включают TypeScript в свои программы.

4. **Интеграция с отечественными решениями**: При работе с российскими фреймворками и CMS часто требуется создание определений типов для обеспечения совместимости.

## Совместимость с JavaScript-библиотеками

Для работы с JavaScript-библиотеками используйте DefinitelyTyped:

```bash
npm install --save-dev @types/lodash
```

Или создайте собственные определения типов:

```typescript
declare module 'my-external-library' {
  export function myFunction(param: string): boolean;
}
```

## Интеграция с системами сборки

### Webpack

Для использования TypeScript с Webpack установите `ts-loader`:

```bash
npm install --save-dev ts-loader
```

Конфигурация webpack:

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
};
```

### Rollup

Для использования с Rollup установите `@rollup/plugin-typescript`:

```bash
npm install --save-dev @rollup/plugin-typescript
```

## Связанные темы

- [[Babel]] - для интеграции с Babel
- [[Преобразование-кода]] - общие принципы транспиляции
- [[Полифилы]] - для обеспечения совместимости с устаревшими браузерами
- [[React]] - использование TypeScript с React
- [[Node.js]] - серверная разработка с TypeScript

## Заключение

TypeScript остается ключевым инструментом в современной JavaScript-разработке, особенно в условиях необходимости создания надежных и поддерживаемых приложений. В российских условиях его использование особенно актуально для корпоративных и государственных проектов, где важны надежность и качество кода.
---
aliases: [tsconfig.json, конфигурационный файл TypeScript, настройки компиляции]
tags: [typescript, configuration, compilation]
---

# tsconfig.json - основной конфигурационный файл TypeScript

Файл `tsconfig.json` является основным конфигурационным файлом для проектов TypeScript. Он определяет параметры компиляции, пути к исходным файлам, настройки типизации и другие важные опции, которые влияют на процесс компиляции TypeScript в JavaScript.

## Основная структура файла

```json
{
  "compilerOptions": {
    // Параметры компилятора
  },
  "include": [
    // Список файлов или шаблонов для включения
  ],
  "exclude": [
    // Список файлов или шаблонов для исключения
  ],
  "files": [
    // Список конкретных файлов для включения
  ],
  "references": [
    // Ссылки на другие проекты TypeScript
  ]
}
```

## Базовая конфигурация

Вот пример минимальной конфигурации для нового проекта TypeScript:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
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

## Объяснение ключевых параметров

### `compilerOptions`

- **`target`** - Указывает версию ECMAScript, в которую компилируется код. Обычно `ES2020`, `ES2021` или `ESNext`.
- **`module`** - Определяет систему модулей для генерируемого JavaScript. Часто `commonjs`, `es6` или `umd`.
- **`outDir`** - Каталог, в который выводятся скомпилированные JavaScript-файлы.
- **`rootDir`** - Каталог, содержащий исходные файлы TypeScript.
- **`strict`** - Включает строгие проверки типов, что помогает избежать многих ошибок.
- **`esModuleInterop`** - Позволяет использовать импорт CommonJS модулей в ES модулях.
- **`skipLibCheck`** - Пропускает проверку типов в файлах библиотек, ускоряя компиляцию.
- **`forceConsistentCasingInFileNames`** - Обеспечивает согласованность регистра имен файлов.

### `include` и `exclude`

- **`include`** - Массив шаблонов файлов, которые должны быть включены в компиляцию.
- **`exclude`** - Массив шаблонов файлов, которые должны быть исключены из компиляции.
- **`files`** - Массив конкретных файлов для включения (редко используется, обычно используется `include`).

## Продвинутая конфигурация

Для более сложных проектов может потребоваться расширенная конфигурация:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "outDir": "./dist",
    "rootDir": "./src",
    "allowJs": true,
    "checkJs": false,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,
    "useUnknownInCatchVariables": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "allowUnreachableCode": false,
    "allowUnusedLabels": false,
    "strictBindCallApply": true,
    "noPropertyAccessFromIndexSignature": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "allowArbitraryExtensions": false,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    }
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "dist",
    "tests"
  ]
}
```

## Использование в проектах

Файл `tsconfig.json` должен находиться в корне проекта TypeScript. Компилятор TypeScript автоматически ищет этот файл в текущей директории и всех родительских директориях.

При запуске команды `tsc` (TypeScript compiler) без аргументов, компилятор использует параметры из `tsconfig.json`. Можно также указать конкретный файл конфигурации с помощью флага `--project` или `-p`.

## Связанные темы

- [[Настройка-проекта]] - Как настроить проект TypeScript с нуля
- [[Наследование-конфигураций]] - Использование extends для наследования конфигураций
- [[Пути-к-файлам]] - Настройка путей к файлам и алиасов
- [[Опции-компилятора]] - Подробное описание всех опций компилятора
- [[Модули]] - Работа с модулями в TypeScript
- [[Типизация-сторонних-библиотек]] - Настройка типов для сторонних библиотек
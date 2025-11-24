---
aliases: ["Конфигурационный файл TypeScript", "tsconfig", "Файл настроек TypeScript"]
tags: ["#typescript", "#compilation", "#configuration", "#build-settings"]
---

# tsconfig.json

Файл `tsconfig.json` - это конфигурационный файл TypeScript, который определяет параметры компиляции и настройки проекта. Он позволяет указать, какие файлы должны быть включены в компиляцию, какие флаги компиляции использовать, и другие важные параметры.

## Основная структура

```json
{
  "compilerOptions": {
    // Параметры компилятора
  },
  "include": [
    // Включаемые файлы
  ],
  "exclude": [
    // Исключаемые файлы
  ],
  "files": [
    // Конкретные файлы для включения
  ]
}
```

## Основные параметры компилятора

### Target
Определяет версию ECMAScript, в которую будет компилироваться код:

```json
{
  "compilerOptions": {
    "target": "ES2020"
  }
}
```

### Module
Указывает систему модулей для генерируемого JavaScript:

```json
{
  "compilerOptions": {
    "module": "commonjs"
  }
}
```

### OutDir
Указывает каталог для вывода скомпилированных файлов:

```json
{
  "compilerOptions": {
    "outDir": "./dist"
  }
}
```

### RootDir
Определяет корневой каталог исходных файлов:

```json
{
  "compilerOptions": {
    "rootDir": "./src"
  }
}
```

## Включение и исключение файлов

### Include
Список файлов или шаблонов файлов, которые должны быть включены в компиляцию:

```json
{
  "include": [
    "src/**/*"
  ]
}
```

### Exclude
Список файлов или шаблонов файлов, которые должны быть исключены из компиляции:

```json
{
  "exclude": [
    "node_modules",
    "**/test/**/*",
    "**/tests/**/*"
  ]
}
```

## Пример полного файла tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false,
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
    },
    "allowSyntheticDefaultImports": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
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

## Наследование конфигураций

Файл `tsconfig.json` может наследовать настройки из другого файла с помощью свойства `extends`:

```json
{
  "extends": "./base-config.json",
  "compilerOptions": {
    "outDir": "./custom-dist"
  }
}
```

## Практические рекомендации

- Используйте `"strict": true` для включения всех строгих проверок типов
- Включите `"sourceMap": true` для отладки TypeScript кода в браузере
- Установите `"declaration": true` для генерации файлов `.d.ts`
- Используйте `"paths"` для создания алиасов импортов

## См. также

- [[Флаги-компиляции]]
- [[Компилятор]]
- [[Инкрементальная-компиляция]]
- [[Множественные-конфигурации]]
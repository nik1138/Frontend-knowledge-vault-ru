---
aliases: [Ambient модули, Объявление модулей]
tags: [typescript, typing, external-libraries, ambient, modules]
---

# Ambient модули

**Ambient модули** — это механизм в TypeScript, позволяющий описывать типы для JavaScript-библиотек, которые не имеют собственной поддержки TypeScript. Они позволяют объявлять типы для внешних библиотек, файлов или модулей, которые будут доступны в рантайме.

## Обзор

Ambient модули используются для описания формы существующего JavaScript-кода, чтобы TypeScript мог понимать и проверять типы при его использовании. Они особенно полезны при работе с библиотеками, которые не предоставляют собственные файлы определений типов.

## Синтаксис объявления

### Объявление модуля по имени

```typescript
declare module 'my-module' {
  export function doSomething(): void;
  export const VERSION: string;
}
```

### Объявление модуля с шаблоном

```typescript
declare module '*.json' {
  const content: any;
  export default content;
}

declare module '*.png' {
  const path: string;
  export default path;
}
```

## Примеры использования

### 1. Объявление сторонней библиотеки

```typescript
// types/my-external-lib.d.ts
declare module 'my-external-lib' {
  export interface Options {
    apiKey: string;
    timeout?: number;
  }

  export function initialize(options: Options): void;
  export function process(data: any[]): Promise<any[]>;
  
  // Объявление как значения (если библиотека экспортирует класс)
  const MyExternalLib: {
    new (options: Options): MyExternalLib;
  };
  
  export default MyExternalLib;
}
```

### 2. Объявление модуля с пространством имен

```typescript
declare module 'my-namespace-lib' {
  namespace MyLib {
    export interface Config {
      settingA: string;
      settingB: number;
    }
    
    export class MainClass {
      constructor(config: Config);
      doWork(): void;
    }
  }
  
  export = MyLib;
}
```

### 3. Объявление модуля с UMD-паттерном

```typescript
declare module 'umd-lib' {
  export = myUmdLib;

  namespace myUmdLib {
    interface Options {
      debug?: boolean;
    }

    function init(options?: Options): void;
    function doSomething(): void;
  }
}
```

## Ambient модули для файлов ресурсов

### JSON файлы

```typescript
declare module '*.json' {
  const content: any;
  export default content;
}
```

### Изображения и другие ресурсы

```typescript
declare module '*.jpg' {
  const path: string;
  export default path;
}

declare module '*.svg' {
  import React = require('react');
  export const ReactComponent: React.SFC<React.SVGProps<SVGSVGElement>>;
  const src: string;
  export default src;
}
```

## Расширение существующих объявлений

Можно расширять уже существующие ambient объявления:

```typescript
// Добавление к существующему модулю
declare module 'existing-module' {
  export interface AdditionalInterface {
    newProperty: string;
  }
  
  // Добавление нового метода к существующему классу
  export class ExistingClass {
    additionalMethod(): void;
  }
}
```

## Глобальные объявления

Для библиотек, которые добавляют глобальные переменные:

```typescript
// types/global-lib.d.ts
declare global {
  const GLOBAL_LIB: {
    version: string;
    init(): void;
  };
  
  interface Window {
    myGlobalLib?: any;
  }
}

// Обязательно экспортировать пустой объект для корректной работы глобальных объявлений
export {};
```

## Практические советы

### 1. Расположение файлов объявлений

Рекомендуется хранить файлы ambient объявлений в директории `types/` в корне проекта:

```
project/
├── src/
├── types/
│   ├── my-external-lib.d.ts
│   ├── global-extensions.d.ts
│   └── file-types.d.ts
└── tsconfig.json
```

### 2. Использование в tsconfig.json

Убедитесь, что TypeScript включает файлы объявлений:

```json
{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"]
  },
  "include": [
    "src/**/*",
    "types/**/*"
  ]
}
```

### 3. Проверка корректности объявлений

Создайте тестовый файл для проверки объявлений:

```typescript
// tests/ambient-modules.test.ts
import myModule from 'my-module';

// Проверка типов
const result = myModule.doSomething(); // Должно работать с правильными типами

// Проверка импорта JSON
import config from './config.json';
console.log(config.someProperty); // Должно работать с автодополнением
```

## Частые ошибки

### 1. Забытый экспорт

```typescript
// НЕПРАВИЛЬНО
declare module 'my-module' {
  function doSomething(): void;
  // Забыли export
}

// ПРАВИЛЬНО
declare module 'my-module' {
  export function doSomething(): void;
}
```

### 2. Неправильное объявление глобальных переменных

```typescript
// НЕПРАВИЛЬНО - забыли export {}
declare global {
  const MY_GLOBAL: string;
}

// ПРАВИЛЬНО
declare global {
  const MY_GLOBAL: string;
}
export {};
```

## Связанные темы

- [[DefinitelyTyped]]
- [@types]
- [[Создание-определений]]
- [[Глобальные-переменные]]
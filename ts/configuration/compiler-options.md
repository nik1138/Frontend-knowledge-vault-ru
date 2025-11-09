# Опции компилятора TypeScript

Опции компилятора TypeScript определяют поведение транспилятора, проверки типов и генерации кода. Понимание этих опций критически важно для эффективной настройки и оптимизации TypeScript-проектов.

## Содержание

1. [Основные опции](#основные-опции)
2. [Строгая типизация](#строгая-типизация)
3. [Модули и разрешение](#модули-и-разрешение)
4. [Эмит и вывод](#эмит-и-вывод)
5. [Источники и карты](#источники-и-карты)
6. [Экспериментальные опции](#экспериментальные-опции)
7. [Дополнительные проверки](#дополнительные-проверки)
8. [Производительность](#производительность)

## Основные опции

### target

Определяет целевую версию ECMAScript для генерируемого JavaScript:

```json
{
  "compilerOptions": {
    "target": "es2020" // или "es5", "es2015", "es2018", "esnext"
  }
}
```

Примеры различий:

```typescript
// Исходный код TypeScript
class MyClass {
  async getData() {
    const data = await fetch('/api/data');
    return data.json();
  }
}

// target: "es5"
var MyClass = /** @class */ (function () {
  function MyClass() {}
  MyClass.prototype.getData = function () {
    return __awaiter(this, void 0, void 0, function () {
      var data;
      return __generator(this, function (_a) {
        switch (_a.label) {
          case 0:
            return [4 /*yield*/, fetch('/api/data')];
          case 1:
            data = _a.sent();
            return [4 /*yield*/, data.json()];
          case 2:
            return [2 /*return*/, _a.sent()];
        }
      });
    });
  };
  return MyClass;
}());

// target: "es2017"
class MyClass {
  async getData() {
    const data = await fetch('/api/data');
    return data.json();
  }
}
```

### module

Определяет систему модулей для генерируемого кода:

```json
{
  "compilerOptions": {
    "module": "commonjs" // или "esnext", "amd", "umd", "system"
  }
}
```

Примеры:

```typescript
// Исходный код
import { helper } from './utils';
export const value = 42;

// module: "commonjs"
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.value = void 0;
const utils_1 = require("./utils");
exports.value = 42;

// module: "esnext"
import { helper } from './utils';
export const value = 42;
```

### lib

Определяет библиотеки, которые должны быть включены в компиляцию:

```json
{
  "compilerOptions": {
    "lib": [
      "es2020",
      "dom",
      "dom.iterable"
    ]
  }
}
```

Доступные библиотеки:
- `es5`, `es2015`, `es2016`, `es2017`, `es2018`, `es2019`, `es2020`, `esnext`
- `dom`, `dom.iterable`, `webworker`, `scripthost`
- `es2015.core`, `es2015.collection`, `es2015.generator`, и т.д.

### outDir и rootDir

Определяют структуру выходной директории:

```json
{
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

Структура:
```bash
project/
├── src/
│   ├── utils/
│   │   └── helpers.ts
│   └── index.ts
├── dist/
│   ├── utils/
│   │   └── helpers.js
│   └── index.js
└── tsconfig.json
```

### allowJs и checkJs

Позволяют компилировать и проверять JavaScript файлы:

```json
{
  "compilerOptions": {
    "allowJs": true,    // Разрешает компиляцию .js файлов
    "checkJs": true,    // Проверяет типы в .js файлах
    "outDir": "./dist"
  },
  "include": [
    "src/**/*"
  ]
}
```

Пример JavaScript файла с JSDoc:

```javascript
// utils.js
/**
 * @param {string} name
 * @param {number} age
 * @returns {string}
 */
export function greet(name, age) {
  return `Hello, ${name}! You are ${age} years old.`;
}

// main.js
import { greet } from './utils.js';

// TypeScript проверит типы благодаря checkJs
const message = greet("John", 30); // OK
const error = greet("John", "30"); // Ошибка типов
```

## Строгая типизация

### strict

Включает все строгие проверки типов:

```json
{
  "compilerOptions": {
    "strict": true // Включает все опции ниже
  }
}
```

Эквивалентно:

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

### noImplicitAny

Запрещает неявный тип `any`:

```json
{
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```

```typescript
// Ошибка: Parameter 'param' implicitly has an 'any' type
function processData(param) {
  return param.toString();
}

// Исправление:
function processData(param: unknown) {
  return param.toString();
}

// Или явное указание any (не рекомендуется):
function processData(param: any) {
  return param.toString();
}
```

### strictNullChecks

Строгая проверка на `null` и `undefined`:

```json
{
  "compilerOptions": {
    "strictNullChecks": true
  }
}
```

```typescript
// Без strictNullChecks
let value: string;
console.log(value.length); // undefined.length - runtime error

// С strictNullChecks
let value: string;
console.log(value.length); // Ошибка: Object is possibly 'undefined'

// Исправление:
let value: string | undefined;
if (value !== undefined) {
  console.log(value.length); // OK
}

// Или с оператором утверждения:
let value: string;
console.log(value!.length); // OK, но небезопасно
```

### strictPropertyInitialization

Проверяет инициализацию свойств класса:

```json
{
  "compilerOptions": {
    "strictPropertyInitialization": true
  }
}
```

```typescript
// Ошибка: Property 'name' has no initializer and is not definitely assigned in the constructor
class User {
  name: string; // Не инициализировано
  age: number;  // Не инициализировано
}

// Исправления:
class User {
  // 1. Инициализация при объявлении
  name: string = "Anonymous";
  
  // 2. Инициализация в конструкторе
  constructor(public age: number) {
    this.name = "User";
  }
  
  // 3. Утверждение определения (!)
  email!: string; // Будет инициализировано позже
  
  // 4. Необязательное свойство
  phone?: string;
}
```

## Модули и разрешение

### moduleResolution

Определяет стратегию разрешения модулей:

```json
{
  "compilerOptions": {
    "moduleResolution": "node" // или "classic"
  }
}
```

### baseUrl и paths

Настройка псевдонимов путей:

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"]
    }
  }
}
```

Использование:

```typescript
// Без псевдонимов
import { Button } from '../../../components/ui/Button';
import { api } from '../../../../utils/api';

// С псевдонимами
import { Button } from '@components/ui/Button';
import { api } from '@utils/api';
```

### rootDirs

Объединяет несколько директорий в одну виртуальную:

```json
{
  "compilerOptions": {
    "rootDirs": ["src", "generated"]
  }
}
```

Структура:
```bash
project/
├── src/
│   └── components/
│       └── Button.ts
├── generated/
│   └── types/
│       └── api.d.ts
└── tsconfig.json
```

В коде можно импортировать из обоих мест как из одного:

```typescript
// src/components/Button.ts
import { ApiTypes } from 'types/api'; // Из generated/types/api.d.ts
```

### typeRoots и types

Настройка директорий и типов для включения:

```json
{
  "compilerOptions": {
    "typeRoots": [
      "./node_modules/@types",
      "./src/types"
    ],
    "types": [
      "node",
      "jest"
    ]
  }
}
```

## Эмит и вывод

### declaration и declarationMap

Генерация файлов объявлений:

```json
{
  "compilerOptions": {
    "declaration": true,      // Генерирует .d.ts файлы
    "declarationMap": true    // Генерирует .d.ts.map файлы
  }
}
```

### sourceMap и inlineSourceMap

Генерация source maps:

```json
{
  "compilerOptions": {
    "sourceMap": true,        // Отдельные .map файлы
    "inlineSourceMap": false, // Source map встроена в .js файл
    "inlineSources": false    // Исходный код встроен в source map
  }
}
```

### removeComments

Удаление комментариев из сгенерированного кода:

```json
{
  "compilerOptions": {
    "removeComments": true
  }
}
```

```typescript
// Исходный код
/**
 * Функция сложения
 * @param a Первое число
 * @param b Второе число
 * @returns Сумма чисел
 */
function add(a: number, b: number): number {
  // Выполняем сложение
  return a + b;
}

// removeComments: false
/**
 * Функция сложения
 * @param a Первое число
 * @param b Второе число
 * @returns Сумма чисел
 */
function add(a, b) {
    // Выполняем сложение
    return a + b;
}

// removeComments: true
function add(a, b) {
    return a + b;
}
```

### noEmit и noEmitOnError

Контроль генерации кода:

```json
{
  "compilerOptions": {
    "noEmit": false,        // Генерировать код (по умолчанию)
    "noEmitOnError": true   // Не генерировать код при ошибках
  }
}
```

## Источники и карты

### sourceRoot и mapRoot

Настройка путей для source maps:

```json
{
  "compilerOptions": {
    "sourceRoot": "/src",     // Путь к исходным файлам для дебаггера
    "mapRoot": "/sourcemaps"  // Путь к source map файлам
  }
}
```

### inlineSources

Встраивание исходного кода в source maps:

```json
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSources": true
  }
}
```

## Экспериментальные опции

### experimentalDecorators

Включает поддержку декораторов:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true
  }
}
```

```typescript
// С experimentalDecorators: true
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    return originalMethod.apply(this, args);
  };
  
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

### emitDecoratorMetadata

Генерация метаданных для декораторов:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

```typescript
// С emitDecoratorMetadata: true
import "reflect-metadata";

function Type(type: any) {
  return function(target: any, propertyKey: string) {
    Reflect.defineMetadata("design:type", type, target, propertyKey);
  };
}

class User {
  @Type(String)
  name: string;
  
  @Type(Number)
  age: number;
}

// Получение метаданных
const nameType = Reflect.getMetadata("design:type", User.prototype, "name");
console.log(nameType === String); // true
```

## Дополнительные проверки

### noUnusedLocals

Проверка неиспользуемых локальных переменных:

```json
{
  "compilerOptions": {
    "noUnusedLocals": true
  }
}
```

```typescript
// Ошибка: 'unusedVar' is declared but its value is never read
function processData(data: any[]) {
  const unusedVar = "This is not used";
  const usedVar = "This is used";
  
  return data.map(item => `${usedVar}: ${item}`);
}
```

### noUnusedParameters

Проверка неиспользуемых параметров:

```json
{
  "compilerOptions": {
    "noUnusedParameters": true
  }
}
```

```typescript
// Ошибка: 'unusedParam' is declared but its value is never read
function handler(event: Event, unusedParam: string) {
  console.log('Event handled');
}

// Исправление - префикс подчеркивания
function handler(event: Event, _unusedParam: string) {
  console.log('Event handled');
}
```

### noImplicitReturns

Проверка, что все пути кода возвращают значение:

```json
{
  "compilerOptions": {
    "noImplicitReturns": true
  }
}
```

```typescript
// Ошибка: Not all code paths return a value
function getStatus(code: number): string {
  if (code === 200) {
    return "OK";
  } else if (code === 404) {
    return "Not Found";
  }
  // Что возвращать для других кодов?
}

// Исправление:
function getStatus(code: number): string {
  if (code === 200) {
    return "OK";
  } else if (code === 404) {
    return "Not Found";
  }
  return "Unknown";
}
```

### noFallthroughCasesInSwitch

Проверка fallthrough в switch:

```json
{
  "compilerOptions": {
    "noFallthroughCasesInSwitch": true
  }
}
```

```typescript
// Ошибка: Fallthrough case in switch
function handleAction(action: string) {
  switch (action) {
    case "save":
      saveData();
      // Забыли break?
    case "load":
      loadData();
      break;
    default:
      console.log("Unknown action");
  }
}

// Исправление:
function handleAction(action: string) {
  switch (action) {
    case "save":
      saveData();
      break;
    case "load":
      loadData();
      break;
    default:
      console.log("Unknown action");
  }
}

// Или явный fallthrough:
function handleAction(action: string) {
  switch (action) {
    case "save":
      saveData();
      // falls through
    case "load":
      loadData();
      break;
    default:
      console.log("Unknown action");
  }
}
```

## Производительность

### incremental

Включает инкрементальную компиляцию:

```json
{
  "compilerOptions": {
    "incremental": true,
    "tsBuildInfoFile": "./dist/.tsbuildinfo"
  }
}
```

### composite

Включает композитные проекты:

```json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true
  }
}
```

### skipLibCheck

Пропускает проверку файлов деклараций:

```json
{
  "compilerOptions": {
    "skipLibCheck": true // Ускоряет компиляцию
  }
}
```

### diagnostics и extendedDiagnostics

Показывает диагностику компиляции:

```json
{
  "compilerOptions": {
    "diagnostics": true,           // Базовая диагностика
    "extendedDiagnostics": true    // Расширенная диагностика
  }
}
```

```bash
# Пример вывода diagnostics
Files:            100
Lines:          50000
Nodes:         200000
Identifiers:    50000
Symbols:        30000
Types:          15000
Memory used:   100000K
I/O read:        0.10s
I/O write:       0.05s
Parse time:      0.50s
Bind time:       0.20s
Check time:      1.00s
Emit time:       0.30s
Total time:      2.00s
```

## Связи с другими концепциями

- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Основы конфигурации
- [[ts/configuration/basic-setup|Базовая настройка]] - Начальная конфигурация
- [[ts/configuration/project-references|Ссылки на проекты]] - Много-проектные конфигурации
- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Настройка путей и разрешения
- [[ts/decorators/Декораторы|Декораторы]] - Настройка поддержки декораторов

## Рекомендации по изучению

1. Начните с основных опций (`target`, `module`, `strict`)
2. Освойте настройку путей и разрешения модулей
3. Практикуйтесь в использовании строгой типизации
4. Изучите опции для оптимизации производительности
5. Освойте экспериментальные опции при необходимости
6. Практикуйтесь в настройке диагностики и отладки

## Следующие шаги

- [[ts/configuration/project-references|Ссылки на проекты]] - Много-проектные конфигурации
- [[ts/configuration/build-modes|Режимы сборки]] - Инкрементальная и композитная сборка
- [[ts/configuration/path-mapping|Сопоставление путей]] - Псевдонимы и разрешение модулей
- [[ts/configuration/watch-options|Опции наблюдения]] - Настройка режима разработки
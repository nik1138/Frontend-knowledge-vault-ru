---
aliases: [Tree Shaking, Удаление неиспользуемого кода]
tags: [ts, build, optimization, tree-shaking, performance]
---

# Tree-shaking

Tree-shaking - это техника удаления неиспользуемого кода из JavaScript-бандлов во время сборки. Термин был введен впервые для описания процесса "стряхивания" неиспользуемого кода, как листья с дерева. Эта техника особенно эффективна при использовании ES-модулей и позволяет значительно уменьшить размер итогового бандла.

## Принцип работы

Tree-shaking работает на основе статического анализа ES-модулей (import/export). Сборщик может определить, какие части кода не используются, и исключить их из итогового бандла:

```typescript
// utils.ts
export const util1 = () => console.log('util1');
export const util2 = () => console.log('util2');
export const util3 = () => console.log('util3');

// main.ts
import { util1 } from './utils'; // используем только util1

util1(); // вызываем только util1

// В итоговом бандле будут только util1 и код, который его использует
// util2 и util3 будут исключены
```

## Условия для эффективного Tree-shaking

### 1. Использование ES-модулей

Для эффективного tree-shaking необходимо использовать `import`/`export` вместо `require`:

```typescript
// Хорошо - ES-модуль, поддерживает tree-shaking
import { debounce } from 'lodash-es';

// Плохо - CommonJS, не поддерживает tree-shaking
import { debounce } from 'lodash';
```

### 2. Иммутабельность экспорта

Экспортируемые значения должны быть константами:

```typescript
// Хорошо - иммутабельный экспорт
export const API_URL = 'https://api.example.com';
export const config = { timeout: 5000 };

// Плохо - изменяемый экспорт
export let counter = 0;
counter++; // изменение после экспорта
```

### 3. Чистые функции

Функции не должны иметь побочных эффектов:

```typescript
// Хорошо - чистая функция
export const add = (a: number, b: number) => a + b;

// Плохо - функция с побочным эффектом
export const logAndAdd = (a: number, b: number) => {
  console.log('Adding numbers');
  return a + b;
};
```

## Практические примеры

### Пример 1: Использование конкретных функций из библиотеки

```typescript
// Вместо импорта всей библиотеки
import * as _ from 'lodash';

// Импортируем только необходимые функции
import { debounce, throttle } from 'lodash-es';

// Это позволяет включить в бандл только используемые функции
```

### Пример 2: Создание библиотеки с поддержкой tree-shaking

```typescript
// math-utils.ts
export const add = (a: number, b: number): number => a + b;
export const subtract = (a: number, b: number): number => a - b;
export const multiply = (a: number, b: number): number => a * b;
export const divide = (a: number, b: number): number => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};

// main.ts
import { add } from './math-utils';

const result = add(5, 3);
console.log(result); // В бандл попадет только функция add
```

### Пример 3: Условный экспорт

```typescript
// config.ts
export const API_URL = process.env.NODE_ENV === 'production' 
  ? 'https://api.prod.com' 
  : 'https://api.dev.com';

export const DEBUG_MODE = process.env.NODE_ENV !== 'production';

// main.ts
import { API_URL } from './config';

// DEBUG_MODE не используется, поэтому может быть исключен из бандла
```

## Настройка для различных сборщиков

### Webpack

Webpack поддерживает tree-shaking из коробки в режиме production:

```javascript
// webpack.config.js
module.exports = {
  mode: 'production', // Включает оптимизации, включая tree-shaking
  optimization: {
    usedExports: true, // Определяет используемые экспорт
    sideEffects: false, // Указывает, что модули не имеют побочных эффектов
  },
};
```

Указание побочных эффектов в package.json:

```json
{
  "name": "my-library",
  "sideEffects": [
    "./src/polyfills.ts",
    "./src/styles.css"
  ]
}
```

### Rollup

Rollup изначально создан с поддержкой tree-shaking:

```javascript
// rollup.config.js
export default {
  input: 'src/index.ts',
  output: {
    file: 'dist/bundle.js',
    format: 'es',
  },
  treeshake: {
    moduleSideEffects: false, // Отключает побочные эффекты модулей
  },
};
```

### Vite

Vite использует Rollup под капотом, поэтому также поддерживает tree-shaking:

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      treeshake: true, // Включено по умолчанию
    },
  },
});
```

## Проблемы и ограничения

### 1. Побочные эффекты

Если модуль имеет побочные эффекты, tree-shaking может не сработать:

```typescript
// logger.ts - модуль с побочным эффектом
console.log('Logger module loaded'); // побочный эффект
export const log = (message: string) => console.log(message);

// main.ts
// Даже если мы не используем log, модуль logger все равно будет включен
// из-за побочного эффекта в виде console.log
```

### 2. Динамические импорты

Tree-shaking не работает с динамическими импортами:

```typescript
// Это не поддерживает tree-shaking
const utils = await import('./utils');
```

### 3. Re-exporting

При переэкспортировании модулей tree-shaking может не сработать:

```typescript
// barrel.ts - barrel-файл
export { util1 } from './util1';
export { util2 } from './util2';
export { util3 } from './util3';

// main.ts
import { util1 } from './barrel'; // может привести к включению всех утилит
```

## Лучшие практики

### 1. Используйте ES-модули

```typescript
// Вместо CommonJS
const { someFunction } = require('./utils');

// Используйте ES-модули
import { someFunction } from './utils';
```

### 2. Импортируйте только необходимое

```typescript
// Плохо
import * as utils from './utils';
utils.functionA();

// Хорошо
import { functionA } from './utils';
functionA();
```

### 3. Создавайте чистые модули

```typescript
// utils.ts - чистый модуль
export const add = (a: number, b: number) => a + b;
export const multiply = (a: number, b: number) => a * b;

// Избегайте побочных эффектов
// console.log('Utils module loaded'); // Это побочный эффект
```

### 4. Используйте sideEffects в package.json

```json
{
  "name": "my-package",
  "sideEffects": [
    "./src/polyfills.js",
    "./src/globals.css"
  ]
}
```

### 5. Избегайте barrel-файлов для библиотек

```typescript
// Вместо этого
export { functionA } from './functionA';
export { functionB } from './functionB';
export { functionC } from './functionC';

// Импортируйте напрямую
import { functionA } from './functionA';
```

## Инструменты анализа

### Bundle Analyzer

Анализируйте, какой код включается в бандл:

```bash
# Установка
npm install --save-dev webpack-bundle-analyzer

# Запуск анализа
npx webpack-bundle-analyzer dist/static/js/*.js
```

### Утилиты для проверки tree-shaking

```typescript
// test-tree-shaking.ts
import { heavyFunction, lightFunction } from './utils';

// Используем только lightFunction
const result = lightFunction();

// heavyFunction не используется, проверяем, включен ли он в бандл
```

## Практические советы

1. **Проверяйте бандл регулярно** - используйте инструменты анализа
2. **Используйте библиотеки с поддержкой tree-shaking** - ищите версии с `-es` суффиксом
3. **Избегайте побочных эффектов** в модулях, предназначенных для tree-shaking
4. **Документируйте побочные эффекты** в package.json
5. **Тестируйте эффективность** - сравнивайте размеры бандлов до и после оптимизаций

## Сравнение с другими техниками

| Техника | Описание | Эффективность |
|---------|----------|---------------|
| Tree-shaking | Удаление неиспользуемых ES-модулей | Высокая |
| Dead code elimination | Удаление недостижимого кода | Средняя |
| Minification | Удаление пробелов и сжатие | Средняя |
| Compression | Сжатие бандла (gzip, brotli) | Средняя |

## Связанные темы

- [[Webpack]] - сборщик с поддержкой tree-shaking
- [[Vite]] - современный сборщик с tree-shaking
- [[Rollup]] - сборщик с отличной поддержкой tree-shaking
- [[Оптимизация-бандлов]] - общие техники оптимизации
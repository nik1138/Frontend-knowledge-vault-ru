---
aliases: [Сборка Rollup, Rollup конфигурация]
tags: [ts, build, rollup, bundler, library]
---

# Rollup

Rollup - это модульный сборщик JavaScript, специально разработанный для создания библиотек. Он идеально подходит для TypeScript-проектов, особенно когда целью является создание библиотеки, которая будет использоваться в других проектах. Rollup поддерживает ES-модули из коробки и обеспечивает отличную оптимизацию кода.

## Архитектура Rollup

Rollup работает на основе:

- **Input** - входные точки приложения
- **Plugins** - плагины для обработки различных типов файлов
- **Output** - настройка формата итогового бандла
- **Treeshaking** - встроенная поддержка удаления неиспользуемого кода

## Основная конфигурация для TypeScript

Файл конфигурации Rollup обычно называется `rollup.config.js` или `rollup.config.ts`:

```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

export default {
  input: 'src/index.ts',
  output: [
    {
      file: 'dist/bundle.cjs',
      format: 'cjs',
      sourcemap: true,
    },
    {
      file: 'dist/bundle.esm.js',
      format: 'es',
      sourcemap: true,
    },
  ],
  plugins: [
    resolve(),
    commonjs(),
    typescript({ tsconfig: './tsconfig.json' }),
  ],
};
```

## Поддержка TypeScript

Для работы с TypeScript в Rollup используется плагин `@rollup/plugin-typescript`:

```javascript
import typescript from '@rollup/plugin-typescript';

export default {
  // ... остальная конфигурация
  plugins: [
    typescript({
      tsconfig: './tsconfig.json', // указываем путь к tsconfig
      declaration: true, // генерация .d.ts файлов
      declarationDir: './dist/types', // директория для .d.ts файлов
    }),
  ],
};
```

## Множественные форматы вывода

Rollup позволяет создавать бандлы в различных форматах:

```javascript
export default {
  input: 'src/index.ts',
  output: [
    // CommonJS для Node.js
    {
      file: 'dist/index.cjs',
      format: 'cjs',
      exports: 'named',
    },
    // ES-модули для современных сборщиков
    {
      file: 'dist/index.esm.js',
      format: 'es',
    },
    // IIFE для использования в браузере
    {
      file: 'dist/index.iife.js',
      format: 'iife',
      name: 'MyLibrary',
    },
    // UMD для универсального использования
    {
      file: 'dist/index.umd.js',
      format: 'umd',
      name: 'MyLibrary',
    },
  ],
};
```

## Плагины для TypeScript-проектов

### @rollup/plugin-node-resolve

Позволяет использовать зависимости из node_modules:

```javascript
import resolve from '@rollup/plugin-node-resolve';

export default {
  plugins: [
    resolve({
      extensions: ['.js', '.ts'],
    }),
  ],
};
```

### @rollup/plugin-commonjs

Преобразует CommonJS-модули в ES-модули:

```javascript
import commonjs from '@rollup/plugin-commonjs';

export default {
  plugins: [
    commonjs(),
  ],
};
```

### @rollup/plugin-terser

Минификация кода:

```javascript
import terser from '@rollup/plugin-terser';

export default {
  output: {
    file: 'dist/bundle.min.js',
    format: 'iife',
  },
  plugins: [
    terser(), // минификация
  ],
};
```

## Tree-shaking

Rollup изначально поддерживает tree-shaking, что позволяет удалять неиспользуемый код:

```typescript
// utils.ts
export const util1 = () => 'util1';
export const util2 = () => 'util2';
export const util3 = () => 'util3';

// main.ts
import { util1 } from './utils'; // используем только util1

export const main = () => {
  return util1();
};

// В итоговом бандле будут только util1 и main, util2 и util3 будут исключены
```

## Работа с зависимостями

Для библиотек важно правильно обрабатывать внешние зависимости:

```javascript
export default {
  input: 'src/index.ts',
  external: [
    'react', // не включать в бандл
    'lodash',
    // или использовать функцию для определения внешних зависимостей
    (id) => id.includes('node_modules'),
  ],
  output: {
    // ... настройки вывода
  },
};
```

## Генерация объявлений типов

Для TypeScript-библиотек важно генерировать .d.ts файлы:

```javascript
import typescript from '@rollup/plugin-typescript';

export default {
  plugins: [
    typescript({
      tsconfig: './tsconfig.json',
      declaration: true,
      declarationDir: './dist/types',
    }),
  ],
};
```

## Практические примеры

### Создание библиотеки с несколькими точками входа

```javascript
// rollup.config.js
export default [
  {
    input: 'src/main.ts',
    output: {
      file: 'dist/main.cjs',
      format: 'cjs',
    },
    plugins: [/* плагины */],
  },
  {
    input: 'src/utils.ts',
    output: {
      file: 'dist/utils.cjs',
      format: 'cjs',
    },
    plugins: [/* плагины */],
  },
];
```

### Условная сборка для разных целей

```javascript
// rollup.config.js
const isProduction = process.env.NODE_ENV === 'production';

export default {
  input: 'src/index.ts',
  output: {
    file: isProduction ? 'dist/index.prod.js' : 'dist/index.dev.js',
    format: 'es',
  },
  plugins: [
    isProduction ? terser() : null, // минификация только в продакшене
  ].filter(Boolean),
};
```

## Сравнение с другими сборщиками

| Особенность | Rollup | Webpack | Vite |
|-------------|--------|---------|------|
| Назначение | Библиотеки | Приложения | Приложения |
| Tree-shaking | Отличный | Хороший | Хороший |
| Производительность | Высокая | Средняя | Высокая |
| Конфигурация | Простая | Сложная | Простая |

## Практические советы

1. **Используйте external** для зависимостей, которые не должны быть включены в бандл
2. **Генерируйте .d.ts файлы** для TypeScript-библиотек
3. **Создавайте несколько форматов вывода** для совместимости
4. **Настройте tsconfig.json** оптимально для библиотеки

## Связанные темы

- [[Webpack]] - сборщик приложений
- [[Vite]] - современный сборщик
- [[Оптимизация-бандлов]] - техники оптимизации
- [[Tree-shaking]] - удаление неиспользуемого кода
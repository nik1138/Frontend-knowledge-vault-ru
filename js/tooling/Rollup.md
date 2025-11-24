---
aliases: [Rollup.js, Модульный сборщик Rollup]
tags: [javascript, bundler, build-tool, tooling]
---

# Rollup.js - Модульный сборщик JavaScript

## Общее описание

Rollup - это инструмент сборки JavaScript, который позволяет объединять небольшие модули в более крупные библиотеки или приложения. В отличие от Webpack или Browserify, Rollup специализируется на создании библиотек и оптимизации модулей, используя стандарт ES6 модулей (ESM) в качестве основы. Это делает его идеальным выбором для создания библиотек, которые будут использоваться другими разработчиками или в других проектах.

## Основные особенности

- **Поддержка ES6 модулей**: Rollup изначально разработан с поддержкой ES6 модулей, что позволяет ему эффективно оптимизировать код и удалять неиспользуемые импорты (tree-shaking).
- **Tree-shaking**: Одна из ключевых особенностей Rollup - способность удалять неиспользуемый код (dead code elimination), что приводит к меньшему размеру конечного бандла.
- **Поддержка различных форматов вывода**: Rollup может выводить бандлы в различных форматах: ES, CommonJS, UMD, AMD.
- **Плагинная архитектура**: Широкий выбор плагинов для обработки различных типов файлов, препроцессоров и оптимизаций.

## Установка и настройка

### Установка

```bash
npm install --save-dev rollup
# или
yarn add -D rollup
```

### Базовая конфигурация

Создайте файл `rollup.config.js` в корне проекта:

```javascript
export default {
  input: 'src/main.js', // Входной файл
  output: {
    file: 'dist/bundle.js', // Выходной файл
    format: 'iife', // Формат вывода (iife, es, cjs, umd, amd)
    name: 'MyBundle' // Имя для IIFE/UMD форматов
  }
};
```

## Практическое применение в российских реалиях 2025

### Преимущества для российских разработчиков

- **Оптимизация для библиотек**: В России активно развиваются open-source проекты и внутренние библиотеки. Rollup идеально подходит для создания легковесных библиотек с минимальным размером и высокой производительностью.
- **Поддержка ES6 модулей**: В 2025 году большинство современных браузеров полностью поддерживают ES6 модули, что делает Rollup отличным выбором для создания современных библиотек.
- **Tree-shaking**: В условиях ограниченной пропускной способности интернета в некоторых регионах России оптимизация размера бандла особенно важна.

### Настройка для российских проектов

```javascript
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import { terser } from 'rollup-plugin-terser';

const isProduction = process.env.NODE_ENV === 'production';

export default {
  input: 'src/index.js',
  output: {
    file: 'dist/library.js',
    format: 'umd',
    name: 'MyLibrary',
    globals: {
      // Определение глобальных зависимостей для UMD формата
      'react': 'React',
      'lodash': '_'
    }
  },
  external: ['react', 'lodash'], // Внешние зависимости
  plugins: [
    resolve({
      browser: true
    }),
    commonjs(),
    babel({
      babelHelpers: 'bundled',
      exclude: 'node_modules/**'
    }),
    isProduction && terser() // Минификация в продакшене
  ].filter(Boolean)
};
```

## Плагины и расширения

### Основные плагины

- **@rollup/plugin-node-resolve**: Позволяет импортировать модули из node_modules
- **@rollup/plugin-commonjs**: Преобразует CommonJS модули в ES6 для использования в Rollup
- **@rollup/plugin-babel**: Интеграция с Babel для транспиляции кода
- **rollup-plugin-terser**: Минификация кода
- **@rollup/plugin-replace**: Замена строк в процессе сборки
- **@rollup/plugin-alias**: Алиасы для импортов

### Пример конфигурации с плагинами

```javascript
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import replace from '@rollup/plugin-replace';
import { terser } from 'rollup-plugin-terser';
import alias from '@rollup/plugin-alias';

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.js',
    format: 'es'
  },
  plugins: [
    alias({
      entries: [
        { find: '@', replacement: './src' },
        { find: '@components', replacement: './src/components' }
      ]
    }),
    replace({
      preventAssignment: true,
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development')
    }),
    resolve({
      browser: true
    }),
    commonjs(),
    babel({
      babelHelpers: 'bundled',
      presets: [
        ['@babel/preset-env', {
          targets: '> 0.25%, not dead'
        }]
      ],
      plugins: ['@babel/plugin-proposal-class-properties']
    }),
    process.env.NODE_ENV === 'production' && terser()
  ].filter(Boolean)
};
```

## Сравнение с другими сборщиками

| Особенность | Rollup | Webpack | Parcel |
|-------------|--------|---------|--------|
| Основное применение | Библиотеки | Приложения | Быстрая разработка |
| Tree-shaking | Отличный | Хороший | Ограниченный |
| Конфигурация | Средняя | Сложная | Минимальная |
| Скорость сборки | Быстрая | Средняя | Быстрая |
| Поддержка форматов | ESM, CJS, UMD, AMD, IIFE | ESM, CJS, AMD | ESM, CJS, UMD |

## Рекомендации по использованию

### Когда использовать Rollup

- При создании библиотек для npm
- Когда важна оптимизация размера бандла (tree-shaking)
- При работе с ES6 модулями
- Для проектов, где важна производительность и минимизация кода

### Когда не использовать Rollup

- Для сложных приложений с большим количеством ресурсов
- Когда нужна сложная настройка лоадеров и плагинов
- Для быстрой прототипизации (Parcel может быть проще)

## Практические примеры

### Создание библиотеки с несколькими форматами вывода

```javascript
// rollup.config.js
export default [
  // ES Module
  {
    input: 'src/index.js',
    output: {
      file: 'dist/library.esm.js',
      format: 'es'
    }
  },
  // CommonJS
  {
    input: 'src/index.js',
    output: {
      file: 'dist/library.cjs.js',
      format: 'cjs'
    }
  },
  // Universal Module Definition
  {
    input: 'src/index.js',
    output: {
      file: 'dist/library.umd.js',
      format: 'umd',
      name: 'MyLibrary'
    }
  }
];
```

### Работа с CSS и другими ресурсами

```javascript
// Для работы с CSS нужно использовать дополнительные плагины
import styles from 'rollup-plugin-styles';

export default {
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.js',
    format: 'iife'
  },
  plugins: [
    styles({
      mode: 'extract', // Извлечение CSS в отдельный файл
      sourceMap: true
    })
  ]
};
```

## Заключение

Rollup остается одним из ведущих инструментов сборки JavaScript, особенно подходящим для создания библиотек и оптимизации кода. В российской экосистеме разработки, где важны производительность и эффективность, Rollup предоставляет мощные возможности для создания качественных библиотек с минимальным размером и высокой производительностью.

Для успешного использования Rollup рекомендуется:

- Изучить возможности tree-shaking для оптимизации бандла
- Использовать подходящие плагины для обработки различных типов файлов
- Правильно настроить форматы вывода в зависимости от целевого использования
- Учитывать особенности российских пользователей при оптимизации размера бандла

[[Webpack]] [[Parcel]] [[Vite]] [[ES6-модули]] [[Tree-shaking]] [[JavaScript-библиотеки]]
# Бандлинг модулей в TypeScript

Бандлинг модулей - это процесс объединения множества отдельных модулей в один или несколько файлов для оптимизации загрузки и выполнения в браузере или Node.js. TypeScript интегрируется с различными бандлерами для создания эффективных сборок приложений.

## Содержание

1. [Основы бандлинга](#основы-бандлинга)
2. [Популярные бандлеры](#популярные-бандлеры)
3. [Интеграция с TypeScript](#интеграция-с-typescript)
4. [Оптимизации бандлинга](#оптимизации-бандлинга)
5. [Практические примеры](#практические-примеры)
6. [Продвинутые техники](#продвинутые-техники)
7. [Мониторинг и анализ](#мониторинг-и-анализ)

## Основы бандлинга

Бандлинг - это процесс, при котором бандлер анализирует зависимости между модулями и создает оптимизированные бандлы для развертывания. Это включает в себя разрешение импортов, транспиляцию кода, минификацию и другие оптимизации.

### Зачем нужен бандлинг

1. **Уменьшение количества HTTP-запросов**
2. **Минификация кода**
3. **Удаление неиспользуемого кода (tree shaking)**
4. **Оптимизация загрузки**
5. **Поддержка старых браузеров**

### Базовый процесс бандлинга

```typescript
// Исходные файлы:
// src/utils/helpers.ts
export function formatDate(date: Date): string {
  return date.toISOString();
}

export function capitalize(str: string): string {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

// src/components/Button.tsx
import { capitalize } from '../utils/helpers';

export interface ButtonProps {
  text: string;
  onClick: () => void;
}

export function Button({ text, onClick }: ButtonProps) {
  return (
    <button onClick={onClick}>
      {capitalize(text)}
    </button>
  );
}

// src/index.tsx
import { Button } from './components/Button';

const App = () => (
  <div>
    <Button text="click me" onClick={() => console.log('clicked')} />
  </div>
);

// Результат бандлинга (упрощенный):
// dist/bundle.js
function formatDate(date) {
  return date.toISOString();
}

function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

function Button({ text, onClick }) {
  return React.createElement(
    'button',
    { onClick: onClick },
    capitalize(text)
  );
}

const App = () => React.createElement(
  'div',
  null,
  React.createElement(Button, {
    text: "click me",
    onClick: () => console.log('clicked')
  })
);
```

## Популярные бандлеры

### 1. Webpack

Webpack - самый популярный бандлер в экосистеме JavaScript:

```javascript
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    })
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
};
```

### 2. Rollup

Rollup оптимизирован для библиотек и использует ES-модули:

```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';
import { nodeResolve } from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import { terser } from 'rollup-plugin-terser';

export default {
  input: 'src/index.ts',
  output: [
    {
      file: 'dist/bundle.cjs.js',
      format: 'cjs'
    },
    {
      file: 'dist/bundle.esm.js',
      format: 'es'
    },
    {
      file: 'dist/bundle.umd.js',
      format: 'umd',
      name: 'MyLibrary'
    }
  ],
  plugins: [
    typescript(),
    nodeResolve(),
    commonjs(),
    terser()
  ]
};
```

### 3. Parcel

Parcel - нулевой конфигурации бандлер:

```json
// package.json
{
  "name": "my-app",
  "scripts": {
    "start": "parcel public/index.html",
    "build": "parcel build public/index.html"
  },
  "devDependencies": {
    "parcel": "^2.0.0",
    "typescript": "^4.0.0"
  }
}
```

### 4. Vite

Vite использует ES-модули для быстрой разработки:

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': '/src'
    }
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'moment']
        }
      }
    }
  }
});
```

## Интеграция с TypeScript

### 1. Webpack + TypeScript

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              configFile: path.resolve(__dirname, 'tsconfig.json'),
              transpileOnly: true, // Для ускорения сборки
              compilerOptions: {
                module: 'esnext' // Важно для tree shaking
              }
            }
          }
        ],
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js']
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

### 2. Rollup + TypeScript

```javascript
// rollup.config.js
import typescript from '@rollup/plugin-typescript';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';

export default {
  input: 'src/index.ts',
  output: {
    dir: 'dist',
    format: 'es'
  },
  plugins: [
    typescript({
      tsconfig: './tsconfig.json',
      declaration: true,
      declarationDir: 'dist/types'
    }),
    resolve(),
    commonjs()
  ]
};
```

### 3. Конфигурация TypeScript для бандлинга

```json
// tsconfig.json для бандлинга
{
  "compilerOptions": {
    "target": "es2018",
    "module": "esnext", // Важно для tree shaking
    "moduleResolution": "node",
    "lib": ["es2018", "dom"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true, // Для генерации .d.ts файлов
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false, // Оставляем комментарии для минификации
    "importHelpers": true, // Используем tslib для вспомогательных функций
    "downlevelIteration": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Оптимизации бандлинга

### 1. Tree Shaking

Tree shaking удаляет неиспользуемый код:

```typescript
// math.ts - библиотека с множеством функций
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export function divide(a: number, b: number): number {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

export function power(base: number, exponent: number): number {
  return Math.pow(base, exponent);
}

// main.ts - используем только часть функций
import { add, multiply } from './math';

const result = add(2, multiply(3, 4)); // 14
// Функции subtract, divide, power будут удалены при tree shaking

// Для эффективного tree shaking:
// 1. Используйте именованные экспорты
// 2. Избегайте импорта всего модуля
// 3. Настройте module: "esnext" в tsconfig.json
```

### 2. Code Splitting

Разделение кода на чанки:

```typescript
// Без code splitting - всё в одном бандле
import { heavyFunction } from './heavy-module';
import { lightFunction } from './light-module';

// С code splitting - динамический импорт
async function loadHeavyFeature() {
  const { heavyFunction } = await import('./heavy-module');
  return heavyFunction();
}

// Webpack автоматически создаст отдельный чанк
const result = await loadHeavyFeature();

// Роутинг с code splitting
const routes = [
  {
    path: '/home',
    component: () => import('./pages/Home')
  },
  {
    path: '/profile',
    component: () => import('./pages/Profile')
  },
  {
    path: '/admin',
    component: () => import('./pages/Admin')
  }
];
```

### 3. Минификация

Минификация уменьшает размер бандла:

```javascript
// webpack.config.js с минификацией
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // Удаляем console.log
            drop_debugger: true, // Удаляем debugger
            pure_funcs: ['console.log'] // Удаляем конкретные функции
          },
          mangle: true, // Сокращаем имена переменных
          format: {
            comments: false // Удаляем комментарии
          }
        }
      })
    ]
  }
};
```

### 4. Кэширование

Кэширование улучшает производительность повторных сборок:

```javascript
// webpack.config.js с кэшированием
module.exports = {
  cache: {
    type: 'filesystem', // Используем файловую систему для кэша
    buildDependencies: {
      config: [__filename] // Пересобираем при изменении конфига
    }
  },
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
          maxInitialRequests: 10,
          minSize: 0
        }
      }
    }
  }
};
```

## Практические примеры

### 1. Бандлинг библиотеки

```typescript
// src/index.ts - точка входа библиотеки
export { Button } from './components/Button';
export { Input } from './components/Input';
export { Modal } from './components/Modal';
export { useApi } from './hooks/useApi';
export { formatDate } from './utils/date';
export type { ButtonProps } from './components/Button';
export type { InputProps } from './components/Input';

// rollup.config.js для библиотеки
import typescript from '@rollup/plugin-typescript';
import { nodeResolve } from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import { terser } from 'rollup-plugin-terser';
import dts from 'rollup-plugin-dts';

export default [
  // JavaScript бандлы
  {
    input: 'src/index.ts',
    output: [
      {
        file: 'dist/index.cjs.js',
        format: 'cjs',
        sourcemap: true
      },
      {
        file: 'dist/index.esm.js',
        format: 'es',
        sourcemap: true
      }
    ],
    plugins: [
      typescript(),
      nodeResolve(),
      commonjs(),
      terser()
    ],
    external: ['react', 'react-dom'] // Не включаем внешние зависимости
  },
  // Типы TypeScript
  {
    input: 'src/index.ts',
    output: {
      file: 'dist/index.d.ts',
      format: 'es'
    },
    plugins: [dts()]
  }
];
```

### 2. Бандлинг приложения React

```typescript
// webpack.config.js для React приложения
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
  entry: './src/index.tsx',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
    clean: true
  },
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils')
    }
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader'
        ]
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    new MiniCssExtractPlugin({
      filename: '[name].[contenthash].css'
    })
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        },
        styles: {
          name: 'styles',
          type: 'css/mini-extract',
          chunks: 'all',
          enforce: true
        }
      }
    },
    runtimeChunk: 'single'
  }
};
```

### 3. Микрофронтенды

```typescript
// shared-deps.ts - общие зависимости
export { React, ReactDOM } from 'react';
export { useState, useEffect } from 'react';

// webpack.config.js для микрофронтенда
module.exports = {
  entry: './src/microfrontend.ts',
  output: {
    filename: 'microfrontend.js',
    library: 'MicroFrontend',
    libraryTarget: 'umd'
  },
  externals: {
    // Используем общие зависимости
    'react': 'React',
    'react-dom': 'ReactDOM'
  }
};

// container-app.ts - контейнерное приложение
async function loadMicroFrontend() {
  // Загружаем общие зависимости
  await Promise.all([
    import('https://cdn.example.com/react.js'),
    import('https://cdn.example.com/react-dom.js')
  ]);
  
  // Загружаем микрофронтенд
  const microFrontend = await import('https://cdn.example.com/microfrontend.js');
  return microFrontend;
}
```

## Продвинутые техники

### 1. Пользовательские чанки

```javascript
// webpack.config.js с пользовательскими чанками
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Разделяем зависимости по категориям
        react: {
          test: /[\\/]node_modules[\\/](react|react-dom)[\\/]/,
          name: 'react',
          chunks: 'all'
        },
        ui: {
          test: /[\\/]node_modules[\\/](antd|@material-ui)[\\/]/,
          name: 'ui',
          chunks: 'all'
        },
        utils: {
          test: /[\\/]node_modules[\\/](lodash|moment|axios)[\\/]/,
          name: 'utils',
          chunks: 'all'
        },
        // Разделяем по маршрутам
        home: {
          test: /[\\/]src[\\/]pages[\\/]Home[\\/]/,
          name: 'home',
          chunks: 'all'
        },
        profile: {
          test: /[\\/]src[\\/]pages[\\/]Profile[\\/]/,
          name: 'profile',
          chunks: 'all'
        }
      }
    }
  }
};
```

### 2. Предзагрузка и предзапрос

```typescript
// preload.ts - предзагрузка модулей
// Webpack magic comments
const Home = React.lazy(() => import(
  /* webpackChunkName: "home" */
  /* webpackPreload: true */
  './pages/Home'
));

const Profile = React.lazy(() => import(
  /* webpackChunkName: "profile" */
  /* webpackPrefetch: true */
  './pages/Profile'
));

// Использование
function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}
```

### 3. Аналитика бандла

```javascript
// webpack.config.js с анализом
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html'
    })
  ]
};

// package.json
{
  "scripts": {
    "build:analyze": "webpack --analyze"
  }
}
```

## Мониторинг и анализ

### 1. Размер бандла

```bash
# Анализ размера бандла
npx bundlephobia-cli my-package

# Сравнение размеров
npx size-limit

# webpack-bundle-analyzer
npx webpack-bundle-analyzer dist/bundle.js
```

### 2. Оптимизация импортов

```typescript
// Плохо - импорт всего lodash
import * as _ from 'lodash';
const result = _.chunk([1, 2, 3, 4], 2);

// Хорошо - импорт только нужного
import chunk from 'lodash/chunk';
const result = chunk([1, 2, 3, 4], 2);

// Ещё лучше - tree shaking
import { chunk } from 'lodash';
const result = chunk([1, 2, 3, 4], 2);
```

### 3. Метрики производительности

```javascript
// webpack.config.js с метриками
const WebpackBar = require('webpackbar');

module.exports = {
  plugins: [
    new WebpackBar() // Красивая полоса прогресса
  ]
};

// package.json с метриками
{
  "scripts": {
    "build": "webpack --json=stats.json && npx webpack-bundle-analyzer stats.json"
  }
}
```

### 4. CI/CD интеграция

```yaml
# .github/workflows/build.yml
name: Build and Analyze
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm ci
      - run: npm run build
      - run: npx bundlewatch --config bundlewatch.config.js
```

```javascript
// bundlewatch.config.js
module.exports = {
  files: [
    {
      path: 'dist/**/*.js',
      maxSize: '100kB'
    },
    {
      path: 'dist/**/*.css',
      maxSize: '10kB'
    }
  ]
};
```

## Связи с другими концепциями

- [[ts/advanced-modules/Продвинутые_модули|Продвинутые модули]] - Основы модульной системы
- [[ts/advanced-modules/module-resolution|Разрешение модулей]] - Алгоритмы поиска модулей
- [[ts/advanced-modules/code-splitting|Разделение кода]] - Разделение приложения на части
- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора
- [[ts/advanced-modules/dynamic-imports|Динамические импорты]] - Динамическая загрузка модулей

## Рекомендации по изучению

1. Практикуйтесь в настройке различных бандлеров
2. Освойте оптимизации tree shaking и code splitting
3. Изучите конфигурацию TypeScript для бандлинга
4. Практикуйтесь в анализе размера бандла
5. Освойте работу с микрофронтендами
6. Изучите интеграцию с CI/CD
7. Практикуйтесь в создании оптимизированных библиотек

## Следующие шаги

- [[ts/advanced-modules/code-splitting|Разделение кода]] - Разделение приложения на части
- [[ts/configuration/Конфигурация|Конфигурация TypeScript]] - Настройка компилятора
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация приложений
- [[ts/advanced-modules/dynamic-imports|Динамические импорты]] - Динамическая загрузка модулей
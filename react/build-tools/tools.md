---
aliases: ["React Build Tools", "React Bundlers", "Сборка React"]
tags: 
  - #react
  - #build-tools
  - #bundlers
  - #webpack
  - #vite
  - #frontend
---

# Инструменты сборки React и бандлеры

## Введение

Инструменты сборки (бандлеры) играют ключевую роль в современной разработке на React. Они отвечают за объединение модулей, оптимизацию кода, обработку ресурсов и подготовку приложения к работе в продакшене. В этой статье мы рассмотрим основные инструменты сборки для React-приложений.

## Webpack: Конфигурация для React

Webpack — один из самых популярных бандлеров, обеспечивающий гибкую настройку процесса сборки.

```js
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react'],
          },
        },
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource',
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
    }),
  ],
  resolve: {
    extensions: ['.js', '.jsx'],
  },
  devServer: {
    static: './dist',
    hot: true,
  },
};
```

> [!tip] 
> Webpack требует более сложной настройки по сравнению с другими инструментами, но предоставляет максимальный контроль над процессом сборки.

Для глубокого понимания Webpack см. [[react/build-tools/webpack-configuration]].

## Vite: Преимущества и настройка

Vite — современный инструмент сборки, обеспечивающий быструю загрузку во время разработки за счет использования нативных ES-модулей.

```js
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
        },
      },
    },
  },
});
```

### Преимущества Vite:
- Мгновенный запуск сервера разработки
- Горячая замена модулей (HMR)
- Автоматическая обработка TypeScript и CSS
- Встроенная поддержка современных фич

Для подробного изучения Vite см. [[react/build-tools/vite-setup]].

## Rollup: Для React-библиотек

Rollup идеально подходит для сборки библиотек благодаря отличной поддержке tree-shaking и различных форматов вывода.

```js
// rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';

export default {
  input: 'src/index.js',
  output: [
    { file: 'dist/bundle.cjs.js', format: 'cjs' },
    { file: 'dist/bundle.esm.js', format: 'es' },
  ],
  external: ['react', 'react-dom'],
  plugins: [
    resolve(),
    commonjs(),
    babel({
      babelHelpers: 'bundled',
      presets: ['@babel/preset-env', '@babel/preset-react'],
      exclude: 'node_modules/**',
    }),
  ],
};
```

> [!info] 
> Rollup особенно хорош для создания публикуемых библиотек, где важна оптимизация размера и поддержка разных форматов.

## Parcel: Особенности нулевой конфигурации

Parcel предоставляет "нулевую конфигурацию" — работает сразу после установки без настройки.

```json
// package.json
{
  "name": "my-react-app",
  "scripts": {
    "dev": "parcel public/index.html",
    "build": "parcel build public/index.html"
  },
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  }
}
```

Parcel автоматически:
- Обнаруживает зависимости
- Обрабатывает CSS, изображения и другие ресурсы
- Применяет оптимизации
- Генерирует манифесты

## Create React App

Create React App (CRA) был стандартным способом начать разработку React-приложений. Хотя он скрывает конфигурацию Webpack, он обеспечивает надежную базу.

```bash
npx create-react-app my-react-app
cd my-react-app
npm start
```

> [!warning] 
> CRA больше не рекомендуется для новых проектов, так как развитие прекращено. Рекомендуется использовать Vite или Next.js.

## Next.js: Система сборки

Next.js предоставляет встроенную систему сборки с поддержкой SSR, SSG и других возможностей.

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  webpack: (config, { isServer }) => {
    // Дополнительная настройка Webpack
    return config;
  },
};

module.exports = nextConfig;
```

## Оптимизация сборки

### Tree Shaking

Tree shaking — это процесс удаления неиспользуемого кода из финальной сборки.

```js
// utils.js
export const formatDate = (date) => {
  // код форматирования даты
};

export const formatNumber = (num) => {
  // код форматирования чисел
};

// В другом файле
import { formatDate } from './utils'; // Только formatDate будет включено
```

### Code Splitting

Code splitting позволяет разбивать приложение на более мелкие части.

```js
import { lazy, Suspense } from 'react';

const LazyComponent = lazy(() => import('./LazyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Загрузка...</div>}>
      <LazyComponent />
    </Suspense>
  );
}
```

## Обработка ресурсов

### CSS-процессинг

```js
// В Webpack
{
  test: /\.css$/,
  use: [
    'style-loader', // Вставляет CSS в DOM
    'css-loader',   // Обрабатывает @import и url()
    'postcss-loader' // Автопрефиксы и т.д.
  ]
}
```

### Обработка ассетов

```js
// В Vite
export default {
  assetsInclude: ['**/*.gltf'], // Включить 3D-модели
  build: {
    assetsDir: 'assets', // Папка для ассетов
    rollupOptions: {
      output: {
        assetFileNames: (assetInfo) => {
          // Настройка имен файлов
          if (assetInfo.name.endsWith('.css')) {
            return 'css/[name].[hash][extname]';
          }
          return 'assets/[name].[hash][extname]';
        }
      }
    }
  }
};
```

## CI/CD интеграция

```yaml
# .github/workflows/build.yml
name: Build and Test
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
      - run: npm test
```

## Заключение

Выбор инструмента сборки зависит от конкретных требований проекта. Для новых проектов рекомендуется использовать Vite из-за его производительности и простоты настройки. Для сложных приложений с SSR подойдет Next.js. Rollup остается отличным выбором для библиотек.

## См. также

- [[react/performance/optimization]]
- [[react/typescript/integration]]
- [[react/state-management/comparison]]
- [[react/testing/build-verification]]

## Ключевые теги

#react #build-tools #bundlers #webpack #vite #rollup #parcel #cra #nextjs #optimization #tree-shaking #code-splitting #ci-cd
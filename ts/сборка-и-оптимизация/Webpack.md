---
aliases: [Сборка Webpack, Webpack конфигурация]
tags: [ts, build, webpack, bundler]
---

# Webpack

Webpack - это мощный модульный сборщик для JavaScript-приложений, который позволяет собирать, оптимизировать и управлять зависимостями проекта. Он особенно полезен при работе с TypeScript-приложениями, обеспечивая эффективную сборку и оптимизацию кода.

## Архитектура Webpack

Webpack работает на основе конфигурационного файла (обычно `webpack.config.js` или `webpack.config.ts`), в котором определяются:

- **Entry points** - начальные точки приложения
- **Output** - место и формат вывода сборки
- **Loaders** - обработчики различных типов файлов (TS, CSS, изображения и т.д.)
- **Plugins** - расширения функциональности
- **Modes** - режимы разработки и продакшена

## Основная конфигурация для TypeScript

Для работы с TypeScript в Webpack необходимо настроить соответствующие лоадеры:

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  mode: 'development',
};
```

## Использование ts-loader

`ts-loader` позволяет Webpack напрямую обрабатывать TypeScript-файлы, используя компилятор TypeScript. Это предпочтительный способ при разработке, так как обеспечивает проверку типов во время сборки.

```javascript
// webpack.config.js
module.exports = {
  // ... остальная конфигурация
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              configFile: path.resolve(__dirname, 'tsconfig.json'),
            },
          },
        ],
        exclude: /node_modules/,
      },
    ],
  },
};
```

## Плагины для TypeScript-проектов

### DefinePlugin

Позволяет определять глобальные переменные, которые будут заменены во время сборки:

```javascript
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    }),
  ],
};
```

### HtmlWebpackPlugin

Автоматически генерирует HTML-файл для вставки бандла:

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
};
```

## Оптимизация производительности

### SplitChunksPlugin

Позволяет разделять код на отдельные бандлы для лучшей кешируемости:

```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
  },
};
```

## Режимы разработки и продакшена

Webpack предоставляет встроенные режимы:

- `development` - оптимизирован для быстрой пересборки и отладки
- `production` - оптимизирован для минимального размера бандла

```javascript
module.exports = {
  mode: process.env.NODE_ENV || 'development',
};
```

## DevServer

Для разработки рекомендуется использовать Webpack DevServer:

```javascript
module.exports = {
  devServer: {
    contentBase: path.join(__dirname, 'dist'),
    compress: true,
    port: 9000,
  },
};
```

## Практические советы

1. **Используйте ts-loader с опцией `transpileOnly: true`** для ускорения сборки в режиме разработки
2. **Настройте tsconfig.json** для оптимальной работы с Webpack
3. **Используйте source maps** для отладки TypeScript-кода в браузере
4. **Разделяйте код** с помощью SplitChunksPlugin для лучшей производительности

## Связанные темы

- [[Vite]] - современный инструмент сборки
- [[Rollup]] - сборщик библиотек
- [[Оптимизация-бандлов]] - техники оптимизации
- [[Tree-shaking]] - удаление неиспользуемого кода
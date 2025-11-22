---
aliases: ["Регулярные выражения в сборщиках", "Webpack и регулярки", "Vite и регулярные выражения"]
tags: ["#regexp", "#build-tools", "#webpack", "#vite", "#frontend", "#optimization"]
---

# Использование регулярных выражений в сборщиках (Webpack, Vite)

## Введение

Сборщики модулей, такие как Webpack и Vite, активно используют регулярные выражения для определения файлов, которые нужно обработать определенным образом. Регулярные выражения позволяют гибко настраивать правила обработки файлов, оптимизировать сборку и управлять зависимостями.

## Webpack

### 1. Настройка правил для обработки файлов

В Webpack регулярные выражения используются в свойстве `module.rules` для определения того, какие файлы нужно обрабатывать тем или иным лоадером:

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      // Обработка JavaScript файлов
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      },
      // Обработка TypeScript файлов
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      },
      // Обработка CSS файлов
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      // Обработка изображений
      {
        test: /\.(png|jpe?g|gif|svg)$/i,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: '[name].[hash].[ext]',
              outputPath: 'images/'
            }
          }
        ]
      },
      // Обработка шрифтов
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        use: ['file-loader']
      }
    ]
  }
};
```

### 2. Оптимизация сборки с помощью регулярных выражений

Регулярные выражения позволяют исключать ненужные файлы из процесса сборки:

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        // Исключаем все файлы из node_modules
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader'
        }
      },
      {
        test: /\.js$/,
        // Включаем только определенные библиотеки для обработки
        include: /some-library/,
        use: {
          loader: 'babel-loader'
        }
      }
    ]
  }
};
```

### 3. Настройка externals

Для исключения определенных зависимостей из бандла:

```javascript
// webpack.config.js
module.exports = {
  externals: {
    // Исключить jQuery из бандла
    jquery: 'jQuery',
    // Исключить все модули, начинающиеся с 'lodash/'
    '^lodash/.*': {
      commonjs: 'lodash',
      commonjs2: 'lodash',
      amd: 'lodash',
      root: '_'
    }
  }
};
```

### 4. Настройка alias с использованием регулярных выражений

```javascript
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      // Сокращение для часто используемых директорий
      '^@/(.*)$': path.resolve(__dirname, 'src/$1'),
      '^components/(.*)$': path.resolve(__dirname, 'src/components/$1'),
      '^utils/(.*)$': path.resolve(__dirname, 'src/utils/$1')
    }
  }
};
```

## Vite

### 1. Настройка правил обработки файлов

В Vite регулярные выражения также используются для настройки плагинов и обработки файлов:

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { resolve } from 'path';

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      // Регулярное выражение для сокращения импортов
      '/^@/(.*)$/': resolve(__dirname, 'src/$1'),
      '/^~(.*)$/': resolve(__dirname, 'node_modules/$1')
    }
  },
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      }
    }
  }
});
```

### 2. Настройка сервера разработки

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  server: {
    proxy: {
      // Проксирование API запросов
      '^/api/.*': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      },
      // Проксирование запросов к аутентификации
      '^/auth/.*': {
        target: 'http://auth-server:8080',
        changeOrigin: true
      }
    }
  }
});
```

### 3. Пользовательские плагины с регулярными выражениями

Создание собственного плагина для обработки файлов по определенным паттернам:

```javascript
// vite-plugin-custom-transform.js
export default function customTransform() {
  return {
    name: 'custom-transform',
    transform(code, id) {
      // Обработка файлов с расширением .custom
      if (/\.custom$/.test(id)) {
        // Замена специальных тегов в файлах
        code = code.replace(/{{\s*(\w+)\s*}}/g, (match, varName) => {
          return `process.env.${varName}`;
        });
        
        return {
          code,
          map: null
        };
      }
    }
  };
}
```

## Практические примеры

### 1. Оптимизация сборки с исключением ненужных файлов

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        // Исключаем тестовые файлы из сборки
        exclude: /\.(test|spec)\.js$/,
        use: {
          loader: 'babel-loader'
        }
      }
    ]
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        }
      }
    }
  }
};
```

### 2. Обработка файлов с определенным именованием

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      // Обработка файлов с префиксом vendor
      {
        test: /vendor\.(css|scss)$/,
        use: ['style-loader', 'css-loader']
      },
      // Обработка файлов с суффиксом .min
      {
        test: /\.min\.js$/,
        use: ['uglify-loader']
      }
    ]
  }
};
```

### 3. Динамическая загрузка модулей

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.async\.(js|ts)$/,
        use: {
          loader: 'babel-loader',
          options: {
            plugins: [
              ['@babel/plugin-syntax-dynamic-import'],
              ['@babel/plugin-transform-modules-commonjs']
            ]
          }
        }
      }
    ]
  }
};
```

### 4. Обработка языковых файлов

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      // Обработка языковых файлов
      {
        test: /\.locale\.(json|yml)$/,
        use: [
          {
            loader: 'i18n-loader',
            options: {
              // Регулярное выражение для извлечения языка из имени файла
              lang: /([a-z]{2})\.locale\.(json|yml)$/
            }
          }
        ]
      }
    ]
  }
};
```

## Лучшие практики

### 1. Эффективные регулярные выражения

При создании регулярных выражений для сборщиков важно учитывать производительность:

```javascript
// Плохо - неэффективное регулярное выражение
const badPattern = /.*\.js$/;

// Хорошо - эффективное регулярное выражение
const goodPattern = /\.js$/;
```

### 2. Группировка файлов

Для улучшения производительности объединяйте похожие правила:

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      // Группировка всех файлов ресурсов
      {
        test: /\.(png|jpe?g|gif|svg|webp)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              name: '[name].[hash:7].[ext]',
              outputPath: 'assets/images/'
            }
          }
        ]
      }
    ]
  }
};
```

### 3. Использование исключений

Всегда используйте `exclude` для исключения ненужных файлов:

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: [
          /node_modules/,
          /\.test\.js$/,
          /\.spec\.js$/
        ],
        use: {
          loader: 'babel-loader'
        }
      }
    ]
  }
};
```

## Заключение

Регулярные выражения в сборщиках модулей играют ключевую роль в настройке процесса сборки. Они позволяют точно определять, какие файлы нужно обрабатывать, каким образом и в каком порядке. Правильное использование регулярных выражений в конфигурации сборщиков помогает оптимизировать процесс сборки, ускорить время разработки и улучшить производительность приложения.

При настройке сборщиков важно учитывать производительность регулярных выражений и использовать их разумно, комбинируя с другими методами оптимизации.
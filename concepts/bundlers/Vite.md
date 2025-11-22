---
aliases: [Vite, Современный сборщик Vite]
tags: [bundlers, vite, javascript, build-tools, frontend]
---

# Vite

## Обзор

Vite - это современный инструмент сборки, созданный Эваном Ю (Evan You), автором Vue.js. Vite решает проблемы традиционных сборщиков, таких как Webpack, особенно в контексте разработки. Основные преимущества Vite включают мгновенный запуск сервера разработки, быструю горячую замену модулей (HMR) и оптимизированную сборку для продакшена.

Vite работает на основе двух ключевых принципов:
1. Разделение зависимости и приложения для быстрой разработки
2. Использование нативных ES-модулей в браузере во время разработки

## Архитектура Vite

### Сервер разработки

В отличие от традиционных сборщиков, Vite использует нативные ES-модули браузера во время разработки. Это позволяет:

- Мгновенно запускать сервер разработки (обычно менее 1 секунды)
- Обрабатывать только измененные файлы при HMR
- Избегать полной пересборки при каждом изменении

### Сборка для продакшена

Для продакшена Vite использует Rollup для создания оптимизированных бандлов, обеспечивая:

- Маленький размер бандла благодаря tree-shaking
- Эффективную код-сплиттинг
- Современную оптимизацию кода

## Установка и настройка

### Базовая установка

```bash
# Создание нового проекта с Vite
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

### Конфигурационный файл

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true, // Автоматически открывает браузер
    hmr: {
      overlay: true // Показывает ошибки поверх приложения
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true, // Генерирует source maps для продакшена
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@mui/material', '@emotion/react']
        }
      }
    }
  }
});
```

### Поддержка различных фреймворков

Vite поддерживает множество фреймворков через официальные плагины:

- `@vitejs/plugin-vue` для Vue.js
- `@vitejs/plugin-react` для React
- `@vitejs/plugin-svelte` для Svelte
- `@vitejs/plugin-legacy` для поддержки старых браузеров

## Особенности и возможности

### Горячая замена модулей (HMR)

Vite обеспечивает очень быструю HMR благодаря использованию нативных ES-модулей:

```javascript
// vite.config.js с настройками HMR
export default defineConfig({
  server: {
    hmr: {
      protocol: 'ws',
      host: 'localhost',
      port: 24678,
      clientPort: 24678,
      overlay: true,
      timeout: 30000,
      successTimeout: 100
    }
  }
});
```

### Виртуальные модули

Vite поддерживает виртуальные модули, что позволяет генерировать код на лету:

```javascript
// plugins/virtual-module.js
export default function virtualModule() {
  return {
    name: 'virtual-module',
    resolveId(id) {
      if (id === 'virtual:my-module') {
        return id;
      }
    },
    load(id) {
      if (id === 'virtual:my-module') {
        return `
          export const message = 'Hello from virtual module';
          export const timestamp = ${Date.now()};
        `;
      }
    }
  };
}

// Использование в коде
import { message } from 'virtual:my-module';
console.log(message); // "Hello from virtual module"
```

### Глобальные переменные и переменные окружения

```javascript
// vite.config.js
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
    __API_URL__: JSON.stringify(process.env.VITE_API_URL)
  }
});

// Использование в приложении
console.log(__APP_VERSION__); // Версия из package.json
console.log(__API_URL__);     // URL из .env файла
```

## Конфигурация

### Основные параметры сервера

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  server: {
    host: true,           // Разрешает доступ с других IP
    port: 3000,           // Порт сервера
    strictPort: false,    // Автоматически выбирает другой порт если текущий занят
    open: true,           // Открывает браузер при запуске
    cors: true,           // Включает CORS
    proxy: {              // Прокси для API запросов
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        secure: false
      }
    }
  }
});
```

### Настройки сборки

```javascript
// vite.config.js
export default defineConfig({
  build: {
    target: 'modules',    // Современные браузеры с поддержкой ES modules
    outDir: 'dist',       // Папка вывода
    assetsDir: 'assets',  // Подпапка для ассетов
    sourcemap: false,     // Генерация source maps
    minify: 'esbuild',    // Использовать esbuild для минификации
    manifest: true,       // Генерирует manifest.json
    rollupOptions: {
      output: {
        assetFileNames: (assetInfo) => {
          let extType = assetInfo.name.split('.').at(1);
          if (/png|jpe?g|svg|gif|tiff|bmp|ico/i.test(extType)) {
            extType = 'img';
          }
          return `${extType}/[name]-[hash][extname]`;
        }
      }
    }
  }
});
```

### Плагины

```javascript
// vite.config.js с кастомным плагином
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    // Кастомный плагин для обработки специфичных файлов
    {
      name: 'custom-transform',
      transform(code, id) {
        if (id.endsWith('.special')) {
          // Преобразование специфичных файлов
          return {
            code: `export default ${JSON.stringify(code)}`,
            map: null
          };
        }
      }
    }
  ]
});
```

## Интеграция с различными фреймворками

### React

```javascript
// vite.config.js для React
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  define: {
    global: 'globalThis',
  },
  resolve: {
    alias: {
      '@': '/src',
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom'],
          'router': ['react-router-dom'],
        }
      }
    }
  }
});
```

### Vue.js

```javascript
// vite.config.js для Vue
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': '/src',
    },
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

### TypeScript

```javascript
// vite.config.js с TypeScript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': '/src',
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-typescript': ['typescript']
        }
      }
    }
  }
});
```

## Практические примеры

### Полная конфигурация для продакшена

```javascript
// vite.config.js для продакшена
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}']
      },
      manifest: {
        name: 'My App',
        short_name: 'MyApp',
        theme_color: '#ffffff',
        icons: [
          {
            src: 'pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png'
          }
        ]
      }
    })
  ],
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
  },
  server: {
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        secure: true
      }
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: false,
    minify: 'esbuild',
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-router': ['react-router-dom'],
          'vendor-ui': ['@mui/material', '@emotion/react']
        },
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
        assetFileNames: (assetInfo) => {
          let extType = assetInfo.name.split('.').at(1);
          if (/png|jpe?g|svg|gif|tiff|bmp|ico/i.test(extType)) {
            extType = 'img';
          } else if (/woff|woff2|eot|ttf|otf/i.test(extType)) {
            extType = 'fonts';
          }
          return `${extType}/[name]-[hash][extname]`;
        }
      }
    }
  }
});
```

### Конфигурация для монорепозитория

```javascript
// vite.config.js для монорепозитория
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@shared': resolve(__dirname, '../shared/src'),
      '@utils': resolve(__dirname, '../shared/utils')
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:4000',
        changeOrigin: true
      }
    }
  }
});
```

### Кастомный плагин для обработки данных

```javascript
// plugins/data-loader.js
export default function dataLoader() {
  return {
    name: 'data-loader',
    transform(code, id) {
      if (id.endsWith('.json')) {
        return {
          code: `export default ${JSON.stringify(require(id))}`,
          map: null
        };
      }
    }
  };
}

// vite.config.js
import { defineConfig } from 'vite';
import dataLoader from './plugins/data-loader';

export default defineConfig({
  plugins: [dataLoader()]
});
```

## Производительность и оптимизация

### Предварительная загрузка зависимостей

```javascript
// vite.config.js с предварительной загрузкой
export default defineConfig({
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      '@mui/material',
      '@emotion/react'
    ],
    exclude: ['some-big-package'], // Исключить из предварительной загрузки
    esbuildOptions: {
      target: 'es2020'
    }
  }
});
```

### Код-сплиттинг

```javascript
// Пример динамического импорта для код-сплиттинга
const LazyComponent = React.lazy(() => import('./LazyComponent'));

function App() {
  return (
    <React.Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </React.Suspense>
  );
}
```

### Статический анализ и tree-shaking

Vite автоматически выполняет tree-shaking благодаря использованию Rollup для продакшена:

```javascript
// utils.js
export function utilA() {
  return 'utilA';
}

export function utilB() {
  return 'utilB';
}

// main.js - только utilA будет включен в бандл
import { utilA } from './utils.js';
console.log(utilA());
```

## Сравнение с другими инструментами

### Vite vs Webpack

| Характеристика | Vite | Webpack |
|----------------|------|---------|
| Скорость запуска | Очень быстрая | Медленная |
| HMR | Быстрая | Медленнее |
| Конфигурация | Простая | Сложная |
| Поддержка ES-модулей | Нативная | Через плагины |
| Поддержка legacy | Ограниченная | Полная |

### Vite vs Parcel

| Характеристика | Vite | Parcel |
|----------------|------|--------|
| Конфигурация | Минимальная | Нулевая |
| Производительность | Высокая | Высокая |
| Гибкость | Высокая | Ограниченная |
| Поддержка фреймворков | Хорошая | Отличная |

## Лучшие практики

### 1. Использование alias для импортов

```javascript
// vite.config.js
export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils')
    }
  }
});

// В коде
import Button from '@components/Button';
import { formatDate } from '@utils/date';
```

### 2. Оптимизация для продакшена

```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Разделение библиотек для оптимизации загрузки
          'vendor-react': ['react', 'react-dom'],
          'vendor-ui': ['@mui/material', '@emotion/react'],
          'vendor-utils': ['lodash', 'date-fns']
        }
      }
    }
  }
});
```

### 3. Управление переменными окружения

```bash
# .env
VITE_API_URL=https://api.example.com
VITE_APP_NAME=My Application
VITE_DEBUG_MODE=true
```

```javascript
// В приложении
const apiUrl = import.meta.env.VITE_API_URL;
const appName = import.meta.env.VITE_APP_NAME;
const isDebug = import.meta.env.VITE_DEBUG_MODE === 'true';
```

### 4. Проверка производительности

```javascript
// vite.config.js с плагином для анализа бандла
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    react(),
    visualizer({
      filename: './dist/stats.html',
      open: true,
      gzipSize: true,
      brotliSize: true
    })
  ]
});
```

## Заключение

Vite представляет собой современный подход к сборке JavaScript-приложений, сочетая в себе высокую производительность во время разработки с оптимизированной сборкой для продакшена. Его архитектура, основанная на нативных ES-модулях, позволяет достичь мгновенного запуска сервера и быстрой горячей замены модулей.

Ключевые преимущества Vite:
- Мгновенный запуск сервера разработки
- Быстрая горячая замена модулей
- Современная архитектура с поддержкой ES-модулей
- Интеграция с различными фреймворками
- Оптимизированная сборка для продакшена

Vite особенно хорошо подходит для современных JavaScript-приложений, где важна скорость разработки и производительность. С ростом поддержки ES-модулей в браузерах и развитием экосистемы, Vite становится все более популярным выбором для новых проектов.

[[Build Tools]] [[JavaScript]] [[Module Bundlers]] [[Frontend Development]]
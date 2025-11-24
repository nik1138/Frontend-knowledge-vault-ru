---
aliases: [Vite.js, Молниеносный сборщик Vite, Next Generation Frontend Tooling]
tags: [javascript, bundler, build-tool, tooling, development-server, esm]
---

# Vite - Молниеносный инструмент сборки следующего поколения

## Общее описание

Vite (произносится как "veet" или "вит") - это современный инструмент сборки для фронтенд-разработки, созданный Evan You (автором Vue.js). Vite решает проблемы традиционных сборщиков, таких как Webpack, предоставляя мгновенный запуск разработки и быструю сборку благодаря использованию нативных ES модулей в браузере и разделению зависимостей от исходного кода.

## Основные особенности

- **Мгновенный запуск разработки**: Использует нативные ES модули в браузере, что позволяет избежать полной сборки при запуске
- **Горячая замена модулей (HMR)**: Чрезвычайно быстрая HMR, обновляющая только измененные модули
- **Разделение зависимостей и исходного кода**: Предварительно собирает зависимости, что ускоряет запуск
- **Поддержка современных стандартов**: Полная поддержка ES6+, TypeScript, JSX и других современных возможностей
- **Встроенная поддержка фреймворков**: Поддержка Vue, React, Preact, Svelte и других
- **Нативная поддержка CSS и ассетов**: Встроенные возможности обработки CSS, изображений и других ресурсов

## Установка и настройка

### Установка

```bash
# Создание нового проекта с помощью create-vite
npm create vite@latest my-project -- --template vanilla
# или с фреймворком
npm create vite@latest my-project -- --template vue
npm create vite@latest my-project -- --template react

# Или установка в существующий проект
npm install --save-dev vite
```

### Базовая конфигурация

Создайте файл `vite.config.js` в корне проекта:

```javascript
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue'; // для Vue проектов

export default defineConfig({
  plugins: [vue()],
  server: {
    port: 3000,
    open: true // автоматически открывать браузер
  },
  build: {
    outDir: 'dist'
  }
});
```

### Структура проекта

```
project/
├── public/
│   └── favicon.ico
├── src/
│   ├── assets/
│   ├── components/
│   ├── App.vue (или App.jsx)
│   └── main.js
├── index.html
├── package.json
└── vite.config.js
```

## Практическое применение в российских реалиях 2025

### Преимущества для российских разработчиков

- **Высокая производительность разработки**: В условиях плотного графика разработки и необходимости быстрой итерации Vite обеспечивает мгновенную загрузку и обновление кода
- **Поддержка всех популярных фреймворков**: В России активно используются Vue, React и другие фреймворки, Vite предоставляет оптимальную поддержку для всех них
- **Уменьшение времени ожидания**: Сокращение времени ожидания сборки улучшает продуктивность разработчиков
- **Современные возможности**: Внедрение современных возможностей JavaScript и CSS без сложной настройки

### Типичная конфигурация для российских проектов

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import { VitePWA } from 'vite-plugin-pwa';
import { resolve } from 'path';

export default defineConfig({
  plugins: [
    vue(),
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}']
      }
    })
  ],
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@components': resolve(__dirname, './src/components')
    }
  },
  server: {
    host: true,
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router'],
          ui: ['element-plus', '@unocss/reset']
        }
      }
    }
  }
});
```

## Плагины и расширения

### Основные официальные плагины

- **@vitejs/plugin-vue**: Поддержка Vue 3
- **@vitejs/plugin-vue-jsx**: Поддержка Vue с JSX
- **@vitejs/plugin-react**: Поддержка React с Fast Refresh
- **@vitejs/plugin-legacy**: Поддержка старых браузеров

### Популярные сторонние плагины

- **vite-plugin-pwa**: Поддержка Progressive Web Apps
- **vite-plugin-components**: Автоматический импорт компонентов
- **unplugin-auto-import**: Автоматический импорт API
- **vite-plugin-eslint**: Интеграция с ESLint
- **vite-plugin-svgr**: Поддержка SVG как React компонентов
- **vite-plugin-compression**: Сжатие бандла

### Пример конфигурации с плагинами

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import vueJsx from '@vitejs/plugin-vue-jsx';
import { VitePWA } from 'vite-plugin-pwa';
import Components from 'unplugin-vue-components/vite';
import AutoImport from 'unplugin-auto-import/vite';
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
    AutoImport({
      resolvers: [ElementPlusResolver()],
      imports: ['vue', 'vue-router', '@vueuse/core']
    }),
    Components({
      resolvers: [ElementPlusResolver()]
    }),
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}']
      }
    }),
    visualizer({
      filename: './dist/stats.html',
      gzipSize: true,
      brotliSize: true
    })
  ],
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: '@use "@/styles/variables" as *;'
      }
    }
  }
});
```

## Сравнение с другими сборщиками

| Особенность | Vite | Webpack | Parcel | Rollup |
|-------------|------|---------|--------|--------|
| Скорость запуска | Очень высокая | Низкая | Высокая | Средняя |
| Горячая замена модулей | Очень быстрая | Быстрая | Быстрая | Требует настройки |
| Конфигурация | Минимальная | Сложная | Нулевая | Средняя |
| Поддержка ES модулей | Нативная | Через настройку | Поддерживается | Основа |
| Подходит для библиотек | Ограниченная | Да | Ограниченная | Отлично |
| Подходит для приложений | Отлично | Отлично | Хорошо | Требует настройки |

## Практические примеры

### Простой React проект с Vite

```javascript
// src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

```jsx
// src/App.jsx
import { useState } from 'react';
import './App.css';

function App() {
  const [count, setCount] = useState(0);

  return (
    <div className="App">
      <h1>Привет из Vite!</h1>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}

export default App;
```

### Работа с TypeScript

```typescript
// src/types.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

// src/api.ts
export const fetchUser = async (id: number): Promise<User> => {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error('Failed to fetch user');
  }
  return response.json();
};
```

Vite предоставляет встроенную поддержку TypeScript без дополнительной настройки.

### Импорт ассетов

```javascript
// Импорт изображений
import logo from './assets/logo.svg';
import './styles.css';

// Использование в шаблоне
const App = () => {
  return {
    html: `<img src="${logo}" alt="Logo" />`
  };
};
```

### Работа с CSS и препроцессорами

```scss
// src/styles/variables.scss
$primary-color: #409eff;
$secondary-color: #67c23a;

// src/App.scss
@use './styles/variables' as *;

.app-container {
  background-color: $primary-color;
  padding: 20px;
  
  .header {
    color: white;
    border-bottom: 2px solid $secondary-color;
  }
}
```

## Рекомендации по использованию

### Когда использовать Vite

- При создании современных SPA и MPWA
- Когда важна скорость разработки и итерации
- При использовании современных фреймворков (Vue, React, Svelte)
- Для проектов, где важна поддержка ES модулей
- При необходимости быстрой настройки разработки

### Когда не использовать Vite

- При создании библиотек (лучше использовать Rollup)
- Когда нужна максимальная гибкость настройки сборки
- Для проектов с особыми требованиями к архитектуре бандлов
- В случаях, когда требуется поддержка устаревших систем сборки

## Продвинутые возможности

### Модульные алиасы

```javascript
// vite.config.js
import { resolve } from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': resolve(__dirname, './src'),
      '@utils': resolve(__dirname, './src/utils'),
      '@components': resolve(__dirname, './src/components')
    }
  }
});
```

### Пользовательские плагины

```javascript
// plugins/my-plugin.js
export default function myPlugin() {
  return {
    name: 'my-plugin',
    transform(code, id) {
      if (id.endsWith('.myext')) {
        // Преобразование пользовательских файлов
        return {
          code: `export default ${JSON.stringify(code)}`,
          map: null
        };
      }
    }
  };
}
```

### Оптимизация зависимостей

```javascript
// vite.config.js
export default defineConfig({
  optimizeDeps: {
    include: ['vue', 'vue-router', 'axios'],
    exclude: ['@my-lib/utils']
  },
  build: {
    rollupOptions: {
      external: ['some-big-lib'],
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router'],
          ui: ['element-plus']
        }
      }
    }
  }
});
```

## Заключение

Vite представляет собой революционный подход к фронтенд-разработке, предлагая молниеносную скорость разработки и современные возможности. В российской экосистеме разработки, где важна скорость вывода продуктов на рынок и эффективность разработчиков, Vite становится все более популярным выбором для новых проектов.

Для успешного использования Vite рекомендуется:

- Использовать для подходящих типов проектов (приложения, а не библиотеки)
- Эксплуатировать возможности горячей замены модулей
- Понимать особенности работы с зависимостями
- Использовать подходящие плагины для конкретных задач

[[Rollup]] [[Parcel]] [[Webpack]] [[JavaScript-сборщики]] [[ES-модули]] [[Горячая-замена-модулей]] [[Vue.js]] [[React]] [[TypeScript]]
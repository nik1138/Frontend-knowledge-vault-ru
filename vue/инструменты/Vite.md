---
aliases: [Vite, Next Generation Frontend Tooling]
tags: [vue, инструменты-разработки, сборка-проекта, vite, быстрая-разработка]
---

# Vite

## Обзор

Vite — это современный инструмент сборки для фронтенд-разработки, созданный создателем Vue.js, Evan You. Он обеспечивает молниеносно быструю инициализацию сервера разработки и горячую перезагрузку модулей (HMR), используя нативные ES-модули в браузере.

В 2025 году Vite стал стандартом де-факто для новых проектов Vue.js благодаря своей скорости и гибкости.

## Установка и создание проекта

Создание нового проекта с Vite:

```bash
npm create vue@latest my-vue-app
cd my-vue-app
npm install
npm run dev
```

Или с использованием yarn:

```bash
yarn create vue my-vue-app
cd my-vue-app
yarn
yarn dev
```

## Конфигурация

Конфигурация Vite находится в файле `vite.config.js`:

```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components')
    }
  },
  server: {
    host: '0.0.0.0',
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: false,
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['vue', 'vue-router'],
          ui: ['element-plus', '@unocss/reset']
        }
      }
    }
  }
})
```

## Особенности Vite

### Быстрая загрузка сервера разработки

Vite использует нативные ES-модули в браузере, что позволяет избежать полной сборки проекта при запуске:

```javascript
// Сервер запускается почти мгновенно, независимо от размера проекта
```

### Горячая перезагрузка модулей (HMR)

HMR в Vite работает быстрее и точнее, чем в Webpack:

```javascript
// Изменения в компонентах обновляются без перезагрузки страницы
// Сохраняется состояние приложения
```

### Поддержка TypeScript

Vite из коробки поддерживает TypeScript:

```javascript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment'
  }
})
```

## Плагины Vite

Vite поддерживает богатую экосистему плагинов:

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import unocss from 'unocss/vite'
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
    unocss(),
    VitePWA({
      registerType: 'autoUpdate',
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}']
      }
    })
  ]
})
```

## Работа с CSS

Vite поддерживает различные препроцессоры CSS:

```javascript
// vite.config.js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      }
    },
    postcss: './postcss.config.js'
  }
})
```

## Оптимизация сборки

Vite предоставляет возможности для оптимизации продакшн-сборки:

```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        assetFileNames: (assetInfo) => {
          if (assetInfo.name.endsWith('.css')) {
            return 'css/[name].[hash].[ext]'
          }
          if (assetInfo.name.endsWith('.png')) {
            return 'images/[name].[hash].[ext]'
          }
          return '[name].[hash].[ext]'
        }
      }
    }
  }
})
```

## Практические рекомендации для российских разработчиков

### Работа за "железным занавесом"

При работе в условиях ограничений доступа к международным ресурсам:

```bash
# Использование китайского зеркала npm
npm config set registry https://registry.npmmirror.com
```

### Оптимизация для медленных соединений

Для работы в регионах с ограниченной пропускной способностью интернета:

```javascript
// vite.config.js
export default defineConfig({
  server: {
    // Уменьшение количества одновременных запросов
    hmr: {
      overlay: true
    }
  },
  optimizeDeps: {
    include: [
      'vue',
      'vue-router',
      'pinia'
    ]
  }
})
```

### Интеграция с российскими сервисами

```javascript
// Пример интеграции с Яндекс.Метрика
// plugins/yandex-metrica.js
export default function yandexMetrica() {
  return {
    name: 'yandex-metrica',
    transformIndexHtml(html) {
      return {
        html,
        tags: [
          {
            tag: 'script',
            children: `
              (function(m,e,t,r,i,k,a){
                // код Яндекс.Метрики
              })(window,document,'script','https://mc.yandex.ru/metrika/tag.js','ym');
            `
          }
        ]
      }
    }
  }
}
```

## Сравнение с Vue CLI

| Особенность | Vite | Vue CLI |
|-------------|------|---------|
| Скорость запуска | Молниеносная | ~30-60 секунд |
| HMR | Быстрее и точнее | Традиционный Webpack HMR |
| Современные стандарты | ES-модули | Webpack |
| Гибкость конфигурации | Высокая | Средняя |
| Поддержка legacy | Ограниченная | Полная |

## Заключение

Vite стал важным инструментом в современной разработке Vue.js приложений. Его скорость и современный подход к сборке делают его идеальным выбором для новых проектов.

## См. также

- [[Vue-CLI]]
- [[Webpack]]
- [[ESLint]]
- [[DevTools]]
- [[Vue-проект]]
- [[Vue-архитектура]]
- [[Vue-производительность]]
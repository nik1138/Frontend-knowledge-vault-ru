---
aliases: ["Разделение кода", "Code Splitting Vue"]
tags: ["#vue", "#code-splitting", "#bundle-optimization", "#performance", "#chunking"]
---

# Code-splitting

## Обзор

[[Code splitting]] (разделение кода) - это техника, которая позволяет разделить бандл приложения на более мелкие части, которые могут загружаться по мере необходимости. Это важная стратегия оптимизации для Vue приложений, особенно в условиях российского интернета в 2025 году, где скорость соединения может быть ограничена, а доступ к зарубежным ресурсам - ограничен.

## Основные концепции

### Что такое Code Splitting

Code splitting позволяет разбивать приложение на отдельные "чанки" (chunks), которые загружаются по требованию. Это уменьшает начальный размер бандла и улучшает время загрузки приложения.

### Преимущества

- Уменьшение начального размера бандла
- Улучшение времени загрузки
- Оптимизация использования памяти
- Возможность загрузки только необходимого кода
- Лучший опыт пользователя

## Реализация в Vue приложениях

### 1. Разделение по маршрутам (Route-based splitting)

Самый распространенный способ разделения кода - это разделение по маршрутам с использованием Vue Router:

```js
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('../views/Home.vue') // Асинхронный импорт
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('../views/About.vue') // Отдельный чанк
  },
  {
    path: '/profile',
    name: 'Profile',
    component: () => import('../views/Profile.vue') // Отдельный чанк
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### 2. Разделение по компонентам (Component-based splitting)

Можно также разделять код по компонентам:

```vue
<template>
  <div>
    <header-component />
    <main-content />
    <heavy-component v-if="showHeavyComponent" />
    <footer-component />
  </div>
</template>

<script>
// Ленивая загрузка компонентов
const HeavyComponent = () => import('../components/HeavyComponent.vue')

export default {
  components: {
    HeaderComponent: () => import('../components/HeaderComponent.vue'),
    MainContent: () => import('../components/MainContent.vue'),
    HeavyComponent, // уже асинхронный импорт
    FooterComponent: () => import('../components/FooterComponent.vue')
  },
  data() {
    return {
      showHeavyComponent: false
    }
  }
}
</script>
```

### 3. Использование Composables

Для разделения кода по функциональности можно использовать асинхронные импорты composables:

```js
// composables/useHeavyFeature.js
export async function useHeavyFeature() {
  const { heavyFunction } = await import('../utils/heavyUtils.js')
  return { heavyFunction }
}
```

```vue
<template>
  <button @click="loadFeature">Загрузить тяжелую функцию</button>
  <div v-if="featureLoaded">{{ result }}</div>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const featureLoaded = ref(false)
    const result = ref(null)
    
    const loadFeature = async () => {
      const { useHeavyFeature } = await import('../composables/useHeavyFeature.js')
      const { heavyFunction } = await useHeavyFeature()
      result.value = heavyFunction()
      featureLoaded.value = true
    }
    
    return {
      loadFeature,
      featureLoaded,
      result
    }
  }
}
</script>
```

## Настройка в Vite

### Конфигурация разделения кода

```js
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  build: {
    rollupOptions: {
      output: {
        // Настройка именования чанков
        chunkFileNames: 'chunks/[name].[hash].js',
        entryFileNames: 'assets/[name].[hash].js',
        
        // Настройка разделения кода
        manualChunks: {
          // Группировка основных зависимостей
          vendor: ['vue', 'vue-router'],
          // Группировка UI-библиотек
          ui: ['element-plus', '@vueuse/core'],
          // Группировка функций для работы с датами
          date: ['dayjs'],
          // Группировка функций для работы с массивами
          utils: ['lodash-es']
        }
      }
    }
  }
})
```

### Автоматическое разделение кода

```js
build: {
  rollupOptions: {
    output: {
      manualChunks(id) {
        if (id.includes('node_modules')) {
          if (id.includes('vue')) {
            return 'vue-vendor'
          }
          if (id.includes('element-plus')) {
            return 'ui-vendor'
          }
          if (id.includes('chart.js')) {
            return 'charts-vendor'
          }
          // Создание общего чанка для остальных зависимостей
          return 'vendor'
        }
      }
    }
  }
}
```

## Настройка в Webpack

### SplitChunksPlugin

```js
// webpack.config.js
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
        vue: {
          test: /[\\/]node_modules[\\/](vue|vue-router)[\\/]/,
          name: 'vue',
          chunks: 'all',
        }
      }
    }
  }
}
```

## Практические примеры

### Пример 1: Разделение по функциональности

```vue
<template>
  <div>
    <nav-menu />
    <router-view />
    <lazy-component v-if="showLazy" />
  </div>
</template>

<script>
import NavMenu from '../components/NavMenu.vue'

export default {
  components: {
    NavMenu,
    // Компонент загружается только при необходимости
    LazyComponent: () => import('../components/LazyComponent.vue')
  },
  data() {
    return {
      showLazy: false
    }
  },
  mounted() {
    // Условная загрузка компонента
    if (this.$route.query.showAdvanced) {
      this.showLazy = true
    }
  }
}
</script>
```

### Пример 2: Условная загрузка функций

```js
// utils/featureLoader.js
export class FeatureLoader {
  static async loadChartFeature() {
    const { createChart } = await import('chart.js')
    return { createChart }
  }

  static async loadImageEditor() {
    const { default: ImageEditor } = await import('../components/ImageEditor.vue')
    return { ImageEditor }
  }

  static async loadExportFeature() {
    const { exportToPdf } = await import('../utils/export.js')
    return { exportToPdf }
  }
}
```

## Мониторинг и анализ

### Использование Webpack Bundle Analyzer

Для анализа результатов разделения кода можно использовать `webpack-bundle-analyzer`:

```bash
npm install --save-dev webpack-bundle-analyzer
```

```js
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html'
    })
  ]
}
```

### Инструменты для Vite

Для [[Vite]] можно использовать плагины вроде `rollup-plugin-visualizer`:

```bash
npm install --save-dev rollup-plugin-visualizer
```

```js
// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    vue(),
    visualizer({
      filename: './bundle-analysis.html',
      open: true
    })
  ]
})
```

## Российские особенности

### Работа с ограничениями

В условиях ограничений на доступ к зарубежным ресурсам в 2025 году:

- Использование локальных зеркал npm-репозиториев
- Хранение критически важных зависимостей локально
- Оптимизация количества HTTP-запросов
- Использование российских CDN

### Соответствие требованиям

- Минимизация внешних зависимостей
- Обеспечение работоспособности без доступа к зарубежным сервисам
- Использование отечественных решений для аналитики и мониторинга

## Заключение

Code splitting - это мощный инструмент оптимизации Vue приложений, который позволяет улучшить производительность и пользовательский опыт. В условиях российского интернета в 2025 году правильное разделение кода становится особенно важным для обеспечения быстрой загрузки и стабильной работы приложений.

См. также:
- [[Конфигурация-сборки]]
- [[Оптимизация]]
- [[Lazy-loading]]
- [[Анализ-бандла]]
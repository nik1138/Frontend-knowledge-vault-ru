---
aliases: [Vue Command Line Interface, Vue CLI]
tags: [vue, инструменты-разработки, сборка-проекта, cli]
---

# Vue-CLI

## Обзор

Vue CLI (Command Line Interface) — это инструмент командной строки для быстрого создания и управления проектов на Vue.js. Он предоставляет готовые шаблоны проектов и встроенную систему плагинов для упрощения настройки и разработки.

В 2025 году Vue CLI остается важным инструментом для разработчиков, особенно при работе с традиционными одностраничными приложениями (SPA) и при миграции с более старых версий Vue.

## Установка

Для установки Vue CLI используйте npm или yarn:

```bash
npm install -g @vue/cli
# или
yarn global add @vue/cli
```

## Создание проекта

Создание нового проекта с помощью Vue CLI:

```bash
vue create my-project
```

CLI предложит выбор пресетов или возможность ручной настройки:

- Babel
- TypeScript
- ESLint
- Prettier
- Vuex
- Vue Router
- CSS Pre-processors (Sass, Less, Stylus)
- Unit Testing (Jest, Mocha)
- E2E Testing (Cypress, Nightwatch)

## Конфигурация

Конфигурация Vue CLI хранится в файле `vue.config.js` в корне проекта:

```javascript
module.exports = {
  publicPath: process.env.NODE_ENV === 'production' ? '/my-app/' : '/',
  outputDir: 'dist',
  assetsDir: 'static',
  indexPath: 'index.html',
  filenameHashing: true,
  pages: {
    index: {
      entry: 'src/main.js',
      template: 'public/index.html',
      filename: 'index.html',
      title: 'Index Page',
      chunks: ['chunk-vendors', 'chunk-common', 'index']
    }
  },
  lintOnSave: process.env.NODE_ENV !== 'production',
  devServer: {
    port: 8080,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true
      }
    }
  }
}
```

## Плагины

Vue CLI поддерживает систему плагинов, которые можно устанавливать через npm:

```bash
vue add router
vue add vuex
vue add @vue/cli-plugin-eslint
```

## Работа с переменными окружения

Vue CLI поддерживает переменные окружения через файлы `.env`:

```
.env                # загружается во всех случаях
.env.local          # локальный файл, игнорируется git
.env.[mode]         # загружается только для указанного режима
.env.[mode].local   # локальный файл для указанного режима
```

Переменные доступны через `process.env`:

```javascript
console.log(process.env.VUE_APP_TITLE)
```

## Практические рекомендации для российских разработчиков

### Работа с прокси-серверами

При работе в корпоративных сетях России часто требуется настройка прокси:

```javascript
// vue.config.js
module.exports = {
  devServer: {
    proxy: {
      '/api': {
        target: process.env.VUE_APP_API_URL || 'http://localhost:3000',
        changeOrigin: true,
        secure: false,
        logLevel: 'debug'
      }
    }
  }
}
```

### Использование российских CDN

Для ускорения загрузки зависимостей в российском сегменте интернета можно использовать российские зеркала npm:

```bash
npm config set registry https://registry.npmmirror.com
```

### Локализация и i18n

Vue CLI отлично работает с библиотеками локализации, такими как vue-i18n:

```bash
vue add i18n
```

## Сравнение с Vite

Хотя Vite становится предпочтительным выбором для новых проектов, Vue CLI остается актуальным для:

- Поддержки старых проектов
- Сложных конфигураций сборки
- Интеграции с legacy-системами
- Стабильности в корпоративной среде

## Заключение

Vue CLI остается важным инструментом в экосистеме Vue.js, особенно при работе с существующими проектами и в корпоративной среде. Его понятная архитектура и богатые возможности конфигурации делают его полезным инструментом для разработчиков.

## См. также

- [[Vite]]
- [[Webpack]]
- [[ESLint]]
- [[DevTools]]
- [[Vue-проект]]
- [[Vue-архитектура]]

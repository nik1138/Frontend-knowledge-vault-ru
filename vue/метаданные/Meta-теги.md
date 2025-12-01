---
aliases: ["Метатеги", "HTML метатеги", "Vue Meta Tags"]
tags: [vue, seo, metadata, html]
---

# Meta-теги в Vue.js

Meta-теги играют важнейшую роль в SEO и отображении страниц в социальных сетях. В Vue.js существует несколько подходов для управления метатегами, каждый из которых имеет свои особенности и области применения.

## Основные виды метатегов

### Основные SEO метатеги
- `<title>` - заголовок страницы
- `<meta name="description">` - описание страницы
- `<meta name="keywords">` - ключевые слова (устаревший, но иногда используется)
- `<meta name="robots">` - инструкции для поисковых роботов
- `<meta charset>` - кодировка страницы
- `<meta name="viewport">` - настройки отображения на мобильных устройствах

### Метатеги Open Graph
- `<meta property="og:title">` - заголовок для социальных сетей
- `<meta property="og:description">` - описание для социальных сетей
- `<meta property="og:image">` - изображение для предпросмотра
- `<meta property="og:url">` - URL страницы
- `<meta property="og:type">` - тип контента

### Метатеги Twitter Card
- `<meta name="twitter:card">` - тип карточки
- `<meta name="twitter:title">` - заголовок для Twitter
- `<meta name="twitter:description">` - описание для Twitter
- `<meta name="twitter:image">` - изображение для Twitter

## Управление метатегами в Vue

### 1. Вручную в index.html

Самый простой способ - добавить метатеги непосредственно в файл `public/index.html`:

```html
<!DOCTYPE html>
<html lang="ru">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title>Мой Vue проект</title>
    <meta name="description" content="Описание моего Vue проекта">
  </head>
  <body>
    <noscript>
      <strong>Для работы этого сайта необходим JavaScript</strong>
    </noscript>
    <div id="app"></div>
  </body>
</html>
```

⚠️ **Важно**: Этот способ подходит только для статических метатегов, общих для всего приложения.

### 2. Vue Meta (рекомендуется для Vue 2)

Пакет `vue-meta` предоставляет удобный способ управления метатегами на основе маршрутов и компонентов:

```bash
npm install vue-meta
```

В `main.js`:
```javascript
import Vue from 'vue'
import VueMeta from 'vue-meta'

Vue.use(VueMeta)

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

В компоненте:
```javascript
export default {
  name: 'ArticlePage',
  metaInfo() {
    return {
      title: this.article.title,
      meta: [
        {
          vmid: 'description',
          name: 'description',
          content: this.article.excerpt
        },
        {
          vmid: 'og-title',
          property: 'og:title',
          content: this.article.title
        },
        {
          vmid: 'og-description',
          property: 'og:description',
          content: this.article.excerpt
        }
      ]
    }
  },
  data() {
    return {
      article: {
        title: 'Заголовок статьи',
        excerpt: 'Краткое описание статьи'
      }
    }
  }
}
```

### 3. Vueuse/head (рекомендуется для Vue 3)

Для Vue 3 рекомендуется использовать `@vueuse/head`:

```bash
npm install @vueuse/head
```

В `main.js`:
```javascript
import { createApp } from 'vue'
import { createHead } from '@vueuse/head'
import App from './App.vue'

const app = createApp(App)
const head = createHead()

app.use(head)
app.mount('#app')
```

В компоненте:
```javascript
import { useHead } from '@vueuse/head'

export default {
  setup() {
    const title = ref('Динамический заголовок')
    const description = ref('Динамическое описание')
    
    useHead({
      title,
      meta: [
        {
          name: 'description',
          content: description
        }
      ]
    })
    
    return {
      title,
      description
    }
  }
}
```

## Особенности для российских проектов (2025)

### 1. Поддержка русского языка
При работе с русским контентом важно правильно кодировать метатеги:

```javascript
// Убедитесь, что содержимое метатегов правильно кодируется
const title = encodeURIComponent(название статьи)
```

### 2. Интеграция с Яндекс.Вебмастером
Для российских проектов важно добавить верификационный метатег:

```html
<meta name="yandex-verification" content="ваш-ключ-верификации">
```

### 3. Поддержка Рунета
Для соответствия российному законодательству может потребоваться:

```html
<meta name="RU-CENTER-verification" content="ключ-верификации">
```

## Лучшие практики

1. **Уникальность** - каждый маршрут должен иметь уникальные метатеги
2. **Длина** - заголовок до 60 символов, описание до 160 символов
3. **Релевантность** - содержимое метатегов должно соответствовать содержимому страницы
4. **Динамическое обновление** - при изменении данных в компоненте метатеги должны обновляться
5. **Тестирование** - регулярно проверяйте метатеги через инструменты разработчика

## Заключение

Правильное управление метатегами в Vue.js критически важно для SEO и отображения страниц в социальных сетях. Выбор подхода зависит от версии Vue, сложности приложения и требований к динамичности метатегов.

## См. также

- [[Open-Graph]]
- [[Twitter-Card]]
- [[SEO-в-Nuxt]]
- [[Structured-Data]]
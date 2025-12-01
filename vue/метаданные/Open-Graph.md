---
aliases: ["Open Graph Protocol", "OGP", "Протокол Open Graph"]
tags: [vue, seo, social, metadata, open-graph]
---

# Open Graph

Протокол Open Graph (OGP) позволяет контролировать, как контент веб-страниц отображается при публикации в социальных сетях. В 2025 году, особенно для российских проектов, правильная настройка Open Graph метатегов критически важна для эффективного продвижения в социальных сетях.

## Основы Open Graph

Open Graph протокол был разработан Facebook в 2010 году и позволяет веб-администраторам добавлять метатеги в HTML, чтобы контролировать, как контент отображается при публикации в социальных сетях. Эти метатеги используются Facebook, Twitter, LinkedIn, Pinterest и другими платформами.

### Обязательные метатеги

```html
<meta property="og:title" content="Заголовок страницы">
<meta property="og:type" content="website">
<meta property="og:image" content="https://mysite.ru/images/preview.jpg">
<meta property="og:url" content="https://mysite.ru/page">
```

### Рекомендуемые метатеги

```html
<meta property="og:description" content="Описание страницы">
<meta property="og:site_name" content="Название сайта">
<meta property="og:locale" content="ru_RU">
```

## Реализация в Vue.js

### 1. Вручную в index.html (для статических метатегов)

```html
<head>
  <!-- Основные метатеги Open Graph -->
  <meta property="og:title" content="Мой Vue проект">
  <meta property="og:type" content="website">
  <meta property="og:image" content="https://mysite.ru/og-image.jpg">
  <meta property="og:url" content="https://mysite.ru">
  <meta property="og:description" content="Описание моего Vue проекта">
  <meta property="og:site_name" content="Мой сайт">
  <meta property="og:locale" content="ru_RU">
</head>
```

### 2. С использованием vue-meta (Vue 2)

```javascript
// В компоненте страницы
export default {
  name: 'ArticlePage',
  metaInfo() {
    return {
      meta: [
        {
          vmid: 'og-title',
          property: 'og:title',
          content: this.article.title
        },
        {
          vmid: 'og-type',
          property: 'og:type',
          content: 'article'
        },
        {
          vmid: 'og-image',
          property: 'og:image',
          content: this.article.image
        },
        {
          vmid: 'og-url',
          property: 'og:url',
          content: window.location.href
        },
        {
          vmid: 'og-description',
          property: 'og:description',
          content: this.article.excerpt
        },
        {
          vmid: 'og-site_name',
          property: 'og:site_name',
          content: 'Мой сайт'
        },
        {
          vmid: 'og-locale',
          property: 'og:locale',
          content: 'ru_RU'
        }
      ]
    }
  },
  data() {
    return {
      article: {
        title: 'Заголовок статьи',
        image: 'https://mysite.ru/images/article.jpg',
        excerpt: 'Краткое описание статьи'
      }
    }
  }
}
```

### 3. С использованием @vueuse/head (Vue 3)

```vue
<script setup>
import { useHead } from '@vueuse/head'

const article = ref({
  title: 'Заголовок статьи',
  image: 'https://mysite.ru/images/article.jpg',
  excerpt: 'Краткое описание статьи'
})

useHead({
  meta: [
    { property: 'og:title', content: article.value.title },
    { property: 'og:type', content: 'article' },
    { property: 'og:image', content: article.value.image },
    { property: 'og:url', content: window.location.href },
    { property: 'og:description', content: article.value.excerpt },
    { property: 'og:site_name', content: 'Мой сайт' },
    { property: 'og:locale', content: 'ru_RU' }
  ]
})
</script>
```

## Специфические метатеги для разных типов контента

### Для статьи/новости

```html
<meta property="og:type" content="article">
<meta property="article:published_time" content="2025-01-15T12:00:00+00:00">
<meta property="article:author" content="Имя автора">
<meta property="article:section" content="Раздел статьи">
<meta property="article:tag" content="тег1, тег2">
```

### Для видео

```html
<meta property="og:type" content="video.other">
<meta property="og:video" content="https://mysite.ru/video.mp4">
<meta property="og:video:width" content="1280">
<meta property="og:video:height" content="720">
<meta property="og:video:type" content="video/mp4">
```

### Для музыки

```html
<meta property="og:type" content="music.song">
<meta property="og:music:duration" content="200">
<meta property="og:music:album" content="Название альбома">
<meta property="og:music:album:track" content="1">
```

## Особенности для российских социальных сетей

### Поддержка ВКонтакте

ВКонтакте также использует Open Graph метатеги, но имеет свои особенности:

- Изображение должно быть не менее 200x200 пикселей
- Рекомендуется использовать соотношение сторон 1.91:1 для превью
- Для лучшего отображения добавьте метатег `vk:app_id`, если у вас есть приложение ВКонтакте

```html
<meta property="vk:app_id" content="ваш_app_id">
```

### Поддержка Одноклассники

Одноклассники также поддерживает Open Graph метатеги, но рекомендуется также использовать:

```html
<meta property="og:image:width" content="800">
<meta property="og:image:height" content="600">
```

## Требования к изображениям

### Основные требования

- **Минимальный размер**: 200x200 пикселей
- **Рекомендуемый размер**: 1200x630 пикселей (для Facebook)
- **Соотношение сторон**: 1.91:1 (рекомендуется)
- **Формат**: JPG или PNG
- **Размер файла**: не более 8 МБ

### Оптимизация изображений для Open Graph

```javascript
// В компоненте Vue
export default {
  computed: {
    ogImage() {
      // Генерация изображения с правильными размерами
      const baseImage = this.article.mainImage
      return baseImage.replace('original', 'og-1200x630')
    }
  }
}
```

## Тестирование Open Graph метатегов

### Инструменты для тестирования

1. **Facebook Sharing Debugger**: https://developers.facebook.com/tools/debug/
2. **LinkedIn Post Inspector**: https://www.linkedin.com/post-inspector/
3. **Twitter Card Validator**: https://cards-dev.twitter.com/validator
4. **Pinterest Rich Pins Validator**: https://developers.pinterest.com/tools/url-debugger/

### Автоматическое тестирование

```javascript
// В файле тестов
describe('Open Graph метатеги', () => {
  it('должны содержать обязательные метатеги', () => {
    const wrapper = mount(ArticlePage, {
      props: {
        article: {
          title: 'Тестовая статья',
          image: 'https://mysite.ru/test.jpg',
          excerpt: 'Тестовое описание'
        }
      }
    })

    expect(wrapper.find('meta[property="og:title"]').attributes('content')).toBe('Тестовая статья')
    expect(wrapper.find('meta[property="og:type"]').attributes('content')).toBe('article')
    expect(wrapper.find('meta[property="og:image"]').attributes('content')).toBe('https://mysite.ru/test.jpg')
    expect(wrapper.find('meta[property="og:description"]').attributes('content')).toBe('Тестовое описание')
  })
})
```

## Особенности для российских проектов (2025)

### 1. Поддержка кириллицы

При работе с русским контентом важно правильно кодировать заголовки:

```javascript
// Убедитесь, что содержимое метатегов правильно кодируется
const title = encodeURIComponent(название статьи)
```

### 2. Региональные настройки

Для российских проектов используйте:

```html
<meta property="og:locale" content="ru_RU">
<meta property="og:locale:alternate" content="en_US">
```

### 3. Временные зоны

При указании времени публикации учитывайте:

```javascript
// В компоненте
computed: {
  publishedTime() {
    // Преобразование времени в UTC для Open Graph
    return new Date(this.article.publishedAt).toISOString()
  }
}
```

## Лучшие практики

1. **Уникальные метатеги** - для каждой страницы
2. **Качественные изображения** - 1200x630 пикселей
3. **Актуальные описания** - не более 200 символов
4. **Регулярное тестирование** - через инструменты социальных сетей
5. **Адаптивность** - изображения должны хорошо смотреться на разных устройствах
6. **Загрузка изображений** - используйте оптимизированные форматы (WebP при поддержке)

## Пример комплексной реализации в Nuxt

```vue
<script setup>
const route = useRoute()
const { data: article } = await useAsyncData(`article-${route.params.slug}`, () => 
  $fetch(`/api/articles/${route.params.slug}`)
)

useHead({
  meta: [
    { property: 'og:title', content: article.value.title },
    { property: 'og:type', content: 'article' },
    { property: 'og:image', content: article.value.ogImage },
    { property: 'og:url', content: `https://mysite.ru${route.path}` },
    { property: 'og:description', content: article.value.excerpt },
    { property: 'og:site_name', content: 'Мой сайт' },
    { property: 'og:locale', content: 'ru_RU' },
    { property: 'article:published_time', content: new Date(article.value.publishedAt).toISOString() },
    { property: 'article:author', content: article.value.author.name }
  ]
})
</script>
```

## Заключение

Правильная настройка Open Graph метатегов в Vue.js приложениях критически важна для эффективного распространения контента в социальных сетях. Особенно это актуально для российских проектов, где социальные сети играют важную роль в привлечении трафика.

## См. также

- [[Meta-теги]]
- [[Twitter-Card]]
- [[SEO-в-Nuxt]]
- [[Structured-Data]]
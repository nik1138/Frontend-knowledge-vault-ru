---
aliases: ["Twitter Cards", "Карточки Twitter", "Twitter Card Protocol"]
tags: [vue, social, metadata, twitter, seo]
---

# Twitter-Card

Twitter Card - это спецификация метатегов, разработанная Twitter для улучшения отображения контента при публикации ссылок в Twitter. В 2025 году, особенно для российских проектов, правильная настройка Twitter Cards важна для эффективного продвижения контента в социальной сети.

## Основы Twitter Cards

Twitter Cards позволяют превратить обычную ссылку в богатый медиа-контент с изображениями, видео, описаниями и другими элементами. В отличие от Open Graph, Twitter Cards используют собственные метатеги, начинающиеся с `twitter:`.

### Обязательные метатеги

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Заголовок страницы">
<meta name="twitter:description" content="Описание страницы">
<meta name="twitter:image" content="https://mysite.ru/images/twitter-card.jpg">
```

### Необязательные, но рекомендуемые метатеги

```html
<meta name="twitter:site" content="@username">
<meta name="twitter:creator" content="@creator">
<meta name="twitter:url" content="https://mysite.ru/page">
```

## Типы Twitter Cards

### 1. Summary Card (summary)

Классическая карточка с небольшим изображением слева:

```html
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="Заголовок статьи">
<meta name="twitter:description" content="Краткое описание статьи">
<meta name="twitter:image" content="https://mysite.ru/images/thumb.jpg">
<meta name="twitter:image:alt" content="Описание изображения">
```

### 2. Summary Card with Large Image (summary_large_image)

Карточка с большим изображением, занимает всю ширину твита:

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Заголовок статьи">
<meta name="twitter:description" content="Краткое описание статьи">
<meta name="twitter:image" content="https://mysite.ru/images/large-image.jpg">
<meta name="twitter:image:alt" content="Описание изображения">
```

### 3. App Card

Для продвижения мобильных приложений:

```html
<meta name="twitter:card" content="app">
<meta name="twitter:app:name:iphone" content="Имя приложения">
<meta name="twitter:app:id:iphone" content="id_приложения">
<meta name="twitter:app:url:iphone" content="appstore://...">
<meta name="twitter:app:name:googleplay" content="Имя приложения">
<meta name="twitter:app:id:googleplay" content="id_приложения">
<meta name="twitter:app:url:googleplay" content="market://...">
```

### 4. Player Card

Для встраивания медиа-контента:

```html
<meta name="twitter:card" content="player">
<meta name="twitter:player" content="https://mysite.ru/player">
<meta name="twitter:player:width" content="480">
<meta name="twitter:player:height" content="270">
<meta name="twitter:image" content="https://mysite.ru/thumbnail.jpg">
```

## Реализация в Vue.js

### 1. Вручную в index.html (для статических метатегов)

```html
<head>
  <!-- Основные метатеги Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Мой Vue проект">
  <meta name="twitter:description" content="Описание моего Vue проекта">
  <meta name="twitter:image" content="https://mysite.ru/twitter-card.jpg">
  <meta name="twitter:site" content="@mysite">
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
          vmid: 'twitter-card',
          name: 'twitter:card',
          content: 'summary_large_image'
        },
        {
          vmid: 'twitter-title',
          name: 'twitter:title',
          content: this.article.title
        },
        {
          vmid: 'twitter-description',
          name: 'twitter:description',
          content: this.article.excerpt
        },
        {
          vmid: 'twitter-image',
          name: 'twitter:image',
          content: this.article.twitterImage
        },
        {
          vmid: 'twitter-site',
          name: 'twitter:site',
          content: '@mysite'
        },
        {
          vmid: 'twitter-creator',
          name: 'twitter:creator',
          content: `@${this.article.author.twitter}`
        }
      ]
    }
  },
  data() {
    return {
      article: {
        title: 'Заголовок статьи',
        excerpt: 'Краткое описание статьи',
        twitterImage: 'https://mysite.ru/images/article-twitter.jpg',
        author: {
          twitter: 'author_handle'
        }
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
  excerpt: 'Краткое описание статьи',
  twitterImage: 'https://mysite.ru/images/article-twitter.jpg',
  author: {
    twitter: 'author_handle'
  }
})

useHead({
  meta: [
    { name: 'twitter:card', content: 'summary_large_image' },
    { name: 'twitter:title', content: article.value.title },
    { name: 'twitter:description', content: article.value.excerpt },
    { name: 'twitter:image', content: article.value.twitterImage },
    { name: 'twitter:site', content: '@mysite' },
    { name: 'twitter:creator', content: `@${article.value.author.twitter}` },
    { name: 'twitter:image:alt', content: article.value.title }
  ]
})
</script>
```

## Особенности для российских проектов (2025)

### 1. Регистрация в Twitter Developer

Для российских проектов важно:

- Пройти верификацию Twitter Developer аккаунта
- Получить подтверждение для использования Twitter Cards
- Регулярно тестировать карточки через Twitter Card Validator

### 2. Поддержка русского языка

При работе с русским контентом:

```javascript
// Убедитесь, что содержимое метатегов правильно кодируется
const title = encodeURIComponent(название статьи)
```

### 3. Учет особенностей российской аудитории

- Используйте изображения, подходящие для российской аудитории
- Учитывайте культурные особенности при создании описаний
- Проверяйте отображение карточек с учетом русского языка

## Требования к изображениям

### Основные требования

- **Summary Card**: изображение не менее 144x144 пикселей
- **Summary Card with Large Image**: изображение не менее 280x150 пикселей (рекомендуется 1200x600)
- **Максимальный размер**: 5MB
- **Форматы**: JPG, PNG, WEBP
- **Соотношение сторон**: от 1:1 до 2:1

### Оптимизация изображений для Twitter Cards

```javascript
// В компоненте Vue
export default {
  computed: {
    twitterImage() {
      // Генерация изображения с правильными размерами для Twitter
      const baseImage = this.article.mainImage
      return baseImage.replace('original', 'twitter-1200x600')
    }
  }
}
```

## Тестирование Twitter Cards

### Инструменты для тестирования

1. **Twitter Card Validator**: https://cards-dev.twitter.com/validator
2. **Social Share Preview**: для предварительного просмотра

### Автоматическое тестирование

```javascript
// В файле тестов
describe('Twitter Card метатеги', () => {
  it('должны содержать обязательные метатеги', () => {
    const wrapper = mount(ArticlePage, {
      props: {
        article: {
          title: 'Тестовая статья',
          excerpt: 'Тестовое описание',
          twitterImage: 'https://mysite.ru/test.jpg',
          author: {
            twitter: 'test_author'
          }
        }
      }
    })

    expect(wrapper.find('meta[name="twitter:card"]').attributes('content')).toBe('summary_large_image')
    expect(wrapper.find('meta[name="twitter:title"]').attributes('content')).toBe('Тестовая статья')
    expect(wrapper.find('meta[name="twitter:description"]').attributes('content')).toBe('Тестовое описание')
    expect(wrapper.find('meta[name="twitter:image"]').attributes('content')).toBe('https://mysite.ru/test.jpg')
    expect(wrapper.find('meta[name="twitter:site"]').attributes('content')).toBe('@mysite')
  })
})
```

## Интеграция с другими метатегами

Twitter Cards должны работать в комплексе с Open Graph метатегами:

```vue
<script setup>
import { useHead } from '@vueuse/head'

// Общие данные для всех метатегов
const title = 'Заголовок статьи'
const description = 'Описание статьи'
const image = 'https://mysite.ru/images/social.jpg'

useHead({
  meta: [
    // Open Graph метатеги
    { property: 'og:title', content: title },
    { property: 'og:description', content: description },
    { property: 'og:image', content: image },
    { property: 'og:type', content: 'article' },
    { property: 'og:url', content: window.location.href },
    
    // Twitter Card метатеги
    { name: 'twitter:card', content: 'summary_large_image' },
    { name: 'twitter:title', content: title },
    { name: 'twitter:description', content: description },
    { name: 'twitter:image', content: image },
    { name: 'twitter:site', content: '@mysite' }
  ]
})
</script>
```

## Лучшие практики

1. **Используйте summary_large_image** - для большинства контентных страниц
2. **Качественные изображения** - с правильными размерами и соотношением сторон
3. **Уникальные описания** - для каждой страницы
4. **Проверка отображения** - через Twitter Card Validator
5. **Альтернативный текст** - для изображений с помощью `twitter:image:alt`
6. **Совместимость с Open Graph** - обеспечьте согласованность метатегов

## Пример комплексной реализации в Nuxt

```vue
<script setup>
const route = useRoute()
const { data: article } = await useAsyncData(`article-${route.params.slug}`, () => 
  $fetch(`/api/articles/${route.params.slug}`)
)

useHead({
  meta: [
    // Twitter Card метатеги
    { name: 'twitter:card', content: 'summary_large_image' },
    { name: 'twitter:title', content: article.value.title },
    { name: 'twitter:description', content: article.value.excerpt },
    { name: 'twitter:image', content: article.value.twitterImage },
    { name: 'twitter:image:alt', content: article.value.title },
    { name: 'twitter:site', content: '@mysite' },
    { name: 'twitter:creator', content: `@${article.value.author.twitter}` },
    
    // Open Graph метатеги
    { property: 'og:title', content: article.value.title },
    { property: 'og:description', content: article.value.excerpt },
    { property: 'og:image', content: article.value.ogImage },
    { property: 'og:type', content: 'article' },
    { property: 'og:url', content: `https://mysite.ru${route.path}` }
  ]
})
</script>
```

## Расширенные возможности

### Добавление ценовой информации (для e-commerce)

```html
<meta name="twitter:label1" content="Цена">
<meta name="twitter:data1" content="999 ₽">
<meta name="twitter:label2" content="Доставка">
<meta name="twitter:data2" content="Бесплатно">
```

### Добавление рейтинга

```html
<meta name="twitter:label1" content="Рейтинг">
<meta name="twitter:data1" content="4.5/5">
```

## Заключение

Twitter Cards - важный инструмент для продвижения контента в Twitter. Правильная настройка этих метатегов в Vue.js приложениях, особенно с учетом российских особенностей, может значительно улучшить кликабельность и вовлеченность аудитории.

## См. также

- [[Meta-теги]]
- [[Open-Graph]]
- [[SEO-в-Nuxt]]
- [[Structured-Data]]
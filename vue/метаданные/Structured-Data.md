---
aliases: ["Структурированные данные", "Schema.org", "JSON-LD", "Маркап структурированных данных"]
tags: [vue, seo, structured-data, schema-org, json-ld]
---

# Structured-Data

Структурированные данные (Structured Data) - это стандартизированный формат для предоставления информации о странице, которая помогает поисковым системам лучше понимать контент. В 2025 году, особенно для российских проектов, правильная реализация структурированных данных критически важна для улучшения видимости в поисковой выдаче.

## Основы структурированных данных

Структурированные данные используют схему (schema) для описания содержимого веб-страницы. Наиболее распространенная схема - Schema.org, которая поддерживается Google, Yandex, Bing и другими поисковыми системами.

### Форматы структурированных данных

1. **JSON-LD** (рекомендуемый) - встраивается в тег `<script>`
2. **Microdata** - атрибуты в HTML-тегах
3. **RDFa** - атрибуты в HTML-тегах

## Реализация в Vue.js

### 1. JSON-LD вручную в index.html

```html
<head>
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Organization",
    "name": "Моя компания",
    "url": "https://mysite.ru",
    "logo": "https://mysite.ru/logo.png"
  }
  </script>
</head>
```

### 2. С использованием vue-meta (Vue 2)

```javascript
// В компоненте страницы
export default {
  name: 'ProductPage',
  metaInfo() {
    return {
      script: [
        {
          type: 'application/ld+json',
          json: {
            '@context': 'https://schema.org',
            '@type': 'Product',
            'name': this.product.name,
            'description': this.product.description,
            'offers': {
              '@type': 'Offer',
              'price': this.product.price,
              'priceCurrency': 'RUB',
              'availability': 'https://schema.org/InStock'
            },
            'image': this.product.image
          }
        }
      ],
      __dangerouslyDisableSanitizers: ['script']
    }
  },
  data() {
    return {
      product: {
        name: 'Название товара',
        description: 'Описание товара',
        price: '1000',
        image: 'https://mysite.ru/product.jpg'
      }
    }
  }
}
```

### 3. С использованием @vueuse/head (Vue 3)

```vue
<script setup>
import { useHead } from '@vueuse/head'

const product = ref({
  name: 'Название товара',
  description: 'Описание товара',
  price: '1000',
  image: 'https://mysite.ru/product.jpg',
  sku: 'SKU123456'
})

useHead({
  script: [
    {
      type: 'application/ld+json',
      children: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'Product',
        name: product.value.name,
        description: product.value.description,
        sku: product.value.sku,
        offers: {
          '@type': 'Offer',
          price: product.value.price,
          priceCurrency: 'RUB',
          availability: 'https://schema.org/InStock'
        },
        image: product.value.image
      })
    }
  ]
})
</script>
```

## Типы структурированных данных

### 1. Article (Статья)

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Заголовок статьи",
  "description": "Краткое описание статьи",
  "author": {
    "@type": "Person",
    "name": "Имя автора"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Название издания",
    "logo": {
      "@type": "ImageObject",
      "url": "https://mysite.ru/logo.png"
    }
  },
  "datePublished": "2025-01-15",
  "dateModified": "2025-01-16",
  "image": "https://mysite.ru/article-image.jpg"
}
```

### 2. Product (Товар)

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Название товара",
  "description": "Описание товара",
  "brand": {
    "@type": "Brand",
    "name": "Название бренда"
  },
  "offers": {
    "@type": "Offer",
    "price": "1000",
    "priceCurrency": "RUB",
    "availability": "https://schema.org/InStock",
    "seller": {
      "@type": "Organization",
      "name": "Название магазина"
    }
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "123"
  },
  "image": "https://mysite.ru/product.jpg"
}
```

### 3. BreadcrumbList (Навигационная цепочка)

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Главная",
      "item": "https://mysite.ru/"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Категория",
      "item": "https://mysite.ru/category"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Текущая страница",
      "item": "https://mysite.ru/category/current-page"
    }
  ]
}
```

### 4. LocalBusiness (Местный бизнес)

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Название бизнеса",
  "image": "https://mysite.ru/business.jpg",
  "telephone": "+7 (495) 123-45-67",
  "email": "info@mysite.ru",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "Улица, дом",
    "addressLocality": "Москва",
    "postalCode": "123456",
    "addressCountry": "RU"
  },
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": [
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday"
      ],
      "opens": "09:00",
      "closes": "18:00"
    }
  ],
  "priceRange": "₽₽"
}
```

## Особенности для российских проектов (2025)

### 1. Поддержка российских схем

Для российских проектов важно учитывать:

- Использование рубля как валюты: `"priceCurrency": "RUB"`
- Российские форматы дат: `"datePublished": "2025-01-15"`
- Локализованные названия и адреса

### 2. Интеграция с Яндекс.Маркетом

Для e-commerce проектов:

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Название товара",
  "offers": {
    "@type": "Offer",
    "price": "1000",
    "priceCurrency": "RUB",
    "seller": {
      "@type": "Organization",
      "name": "Название магазина",
      "url": "https://mysite.ru"
    }
  }
}
```

### 3. Соответствие российскому законодательству

При указании цен и условий продажи:

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "offers": {
    "@type": "Offer",
    "price": "1000",
    "priceCurrency": "RUB",
    "priceSpecification": {
      "@type": "PriceSpecification",
      "valueAddedTaxIncluded": "true"
    }
  }
}
```

## Тестирование структурированных данных

### Инструменты для тестирования

1. **Google Rich Results Test**: https://search.google.com/test/rich-results
2. **Yandex Structured Data Validator**: https://webmaster.yandex.ru/tools/microtest/
3. **Schema Markup Validator**: https://validator.schema.org/

### Автоматическое тестирование

```javascript
// В файле тестов
describe('Структурированные данные', () => {
  it('должны содержать корректный JSON-LD для продукта', () => {
    const wrapper = mount(ProductPage, {
      props: {
        product: {
          name: 'Тестовый товар',
          description: 'Описание тестового товара',
          price: '1000',
          image: 'https://mysite.ru/test.jpg',
          sku: 'TEST123'
        }
      }
    })

    const scriptTag = wrapper.find('script[type="application/ld+json"]')
    const jsonData = JSON.parse(scriptTag.text())
    
    expect(jsonData['@type']).toBe('Product')
    expect(jsonData.name).toBe('Тестовый товар')
    expect(jsonData.offers.price).toBe('1000')
    expect(jsonData.offers.priceCurrency).toBe('RUB')
  })
})
```

## Лучшие практики

1. **Используйте JSON-LD** - рекомендуемый формат
2. **Точность данных** - все данные должны быть актуальными
3. **Полнота информации** - включайте все релевантные свойства
4. **Регулярное тестирование** - проверяйте через инструменты поисковых систем
5. **Соответствие контенту** - структурированные данные должны отражать реальный контент страницы
6. **Многоязычность** - используйте локализованные версии для разных языков

## Пример комплексной реализации в Nuxt

```vue
<script setup>
const route = useRoute()
const { data: product } = await useAsyncData(`product-${route.params.slug}`, () => 
  $fetch(`/api/products/${route.params.slug}`)
)

useHead({
  script: [
    {
      type: 'application/ld+json',
      children: JSON.stringify({
        '@context': 'https://schema.org',
        '@type': 'Product',
        name: product.value.name,
        description: product.value.description,
        sku: product.value.sku,
        mpn: product.value.mpn,
        brand: {
          '@type': 'Brand',
          name: product.value.brand
        },
        offers: {
          '@type': 'Offer',
          price: product.value.price,
          priceCurrency: 'RUB',
          availability: product.value.inStock 
            ? 'https://schema.org/InStock' 
            : 'https://schema.org/OutOfStock',
          seller: {
            '@type': 'Organization',
            name: 'Мой магазин'
          }
        },
        aggregateRating: product.value.rating ? {
          '@type': 'AggregateRating',
          ratingValue: product.value.rating,
          reviewCount: product.value.reviewCount
        } : undefined,
        image: product.value.images[0]
      })
    }
  ]
})
</script>
```

## Продвинутые схемы

### Event (Событие)

```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "Название события",
  "startDate": "2025-03-15T19:00",
  "endDate": "2025-03-15T22:00",
  "eventStatus": "https://schema.org/EventScheduled",
  "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
  "location": {
    "@type": "Place",
    "name": "Название места",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "Улица, дом",
      "addressLocality": "Город",
      "postalCode": "123456",
      "addressCountry": "RU"
    }
  },
  "offers": {
    "@type": "Offer",
    "price": "500",
    "priceCurrency": "RUB",
    "availability": "https://schema.org/InStock"
  }
}
```

### Course (Курс)

```json
{
  "@context": "https://schema.org",
  "@type": "Course",
  "name": "Название курса",
  "description": "Описание курса",
  "provider": {
    "@type": "Organization",
    "name": "Название образовательной организации",
    "sameAs": "https://mysite.ru"
  }
}
```

## Ошибки и как их избежать

### 1. Несоответствие данных контенту

```javascript
// ПЛОХО - данные не соответствуют реальному контенту
{
  "@type": "Product",
  "name": "Товар А",
  "offers": {
    "price": "1000"
  }
}
// На странице отображается "Товар Б" с ценой 1500

// ХОРОШО - данные соответствуют реальному контенту
{
  "@type": "Product",
  "name": "Товар Б",
  "offers": {
    "price": "1500"
  }
}
```

### 2. Неполные данные

```javascript
// ПЛОХО - недостаточно информации
{
  "@type": "Product",
  "name": "Товар"
}

// ХОРОШО - полная информация
{
  "@type": "Product",
  "name": "Название товара",
  "description": "Описание товара",
  "offers": {
    "@type": "Offer",
    "price": "1000",
    "priceCurrency": "RUB",
    "availability": "https://schema.org/InStock"
  },
  "image": "https://mysite.ru/product.jpg"
}
```

## Заключение

Структурированные данные играют важную роль в SEO и помогают поисковым системам лучше понимать содержимое веб-страниц. Правильная реализация этих данных в Vue.js приложениях, особенно с учетом российских особенностей, может значительно улучшить видимость сайта в поисковой выдаче.

## См. также

- [[Meta-теги]]
- [[Open-Graph]]
- [[Twitter-Card]]
- [[SEO-в-Nuxt]]
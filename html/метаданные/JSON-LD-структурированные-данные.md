---
aliases: ["JSON-LD", "Structured Data JSON-LD", "JSON-LD структурированные данные"]
tags: [html, metadata, structured-data, seo, json-ld]
---

# JSON-LD-структурированные-данные

JSON-LD (JavaScript Object Notation for Linked Data) - это формат структурированных данных, рекомендованный W3C для добавления семантической разметки веб-страницам. JSON-LD позволяет поисковым системам и другим инструментам лучше понимать содержимое страницы и отображать его в расширенном виде в результатах поиска.

## Основы JSON-LD

### Базовая структура
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Моя компания",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png"
}
</script>
```

### Основные компоненты:
- `@context` - указывает схему (обычно https://schema.org)
- `@type` - тип сущности (Organization, Person, Article и т.д.)
- Свойства - конкретные данные о сущности

## Популярные типы JSON-LD

### Article
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Заголовок статьи",
  "author": {
    "@type": "Person",
    "name": "Имя автора"
  },
  "datePublished": "2025-01-01T12:00:00+07:00",
  "dateModified": "2025-01-01T14:30:00+07:00",
  "image": "https://example.com/article-image.jpg",
  "publisher": {
    "@type": "Organization",
    "name": "Название издания",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  },
  "description": "Краткое описание статьи"
}
```

### Product
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Название товара",
  "image": "https://example.com/product-image.jpg",
  "description": "Описание товара",
  "sku": "ART12345",
  "offers": {
    "@type": "Offer",
    "price": "12999",
    "priceCurrency": "RUB",
    "availability": "https://schema.org/InStock",
    "seller": {
      "@type": "Organization",
      "name": "Магазин"
    }
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "89"
  }
}
```

### LocalBusiness
```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Компания",
  "image": "https://example.com/image.jpg",
  "telephone": "+7 (495) 123-45-67",
  "email": "info@example.com",
  "openingHours": [
    "Mo-Fr 09:00-18:00",
    "Sa 10:00-15:00",
    "Su 11:00-14:00"
  ],
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "Улица Примерная, 1",
    "addressLocality": "Москва",
    "addressRegion": "Москва",
    "postalCode": "123456",
    "addressCountry": "RU"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 55.7558,
    "longitude": 37.6176
  }
}
```

## Российские особенности 2025

В 2025 году в России:

- Яндекс активно использует JSON-LD данные для своих сервисов
- Увеличение значимости локального SEO
- Рост популярности голосового поиска
- Ужесточение требований к достоверности информации

### Яндекс и JSON-LD

Яндекс поддерживает большинство схем Schema.org:
- Поддержка структурированных данных для Rich-контента
- Использование для улучшения понимания контента
- Интеграция с Яндекс.Картами и другими сервисами

### Особенности российского рынка

Для российских сайтов важно учитывать:
- Использование российских форматов телефонов
- Указание российских адресов в нужном формате
- Поддержка кириллических доменов
- Соответствие требованиям российского законодательства

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "ООО 'Российская компания'",
  "telephone": "+7 (495) 123-45-67",
  "vatID": "7701234567",  // ИНН для российских компаний
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "ул. Примерная, д. 1, стр. 1",
    "addressLocality": "Москва",
    "addressRegion": "Москва",
    "postalCode": "123456",
    "addressCountry": "RU"
  }
}
```

## Практические рекомендации

### Местоположение JSON-LD
Размещайте JSON-LD в секции `<head>` или в начале секции `<body>`:
```html
<head>
  <title>Заголовок страницы</title>
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "WebSite",
    "name": "Мой сайт",
    "url": "https://example.com/"
  }
  </script>
</head>
```

### Валидация JSON-LD
Всегда проверяйте корректность JSON-LD:
- Используйте валидаторы JSON
- Проверяйте через инструменты поисковых систем
- Тестируйте на различных страницах сайта

### Пример сложной структуры
```json
{
  "@context": "https://schema.org",
  "@type": "NewsArticle",
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "https://example.com/news/article"
  },
  "headline": "Заголовок новости",
  "alternativeHeadline": "Альтернативный заголовок",
  "image": [
    "https://example.com/photos/1.jpg",
    "https://example.com/photos/2.jpg"
  ],
  "datePublished": "2025-01-01T12:00:00+07:00",
  "dateModified": "2025-01-01T14:30:00+07:00",
  "author": {
    "@type": "Person",
    "name": "Иван Иванов",
    "url": "https://example.com/author/ivan-ivanov"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Издательство",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.jpg",
      "width": 600,
      "height": 60
    }
  },
  "description": "Описание новости",
  "articleSection": "Политика",
  "keywords": "ключевые, слова, новость"
}
```

> [!tip] Совет
> В России в 2025 году особенно важно использовать JSON-LD для улучшения видимости в Яндексе, так как отечественная поисковая система активно использует структурированные данные.

## Популярные схемы для российских сайтов

### Event (События)
```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "Конференция разработчиков",
  "startDate": "2025-03-15T10:00",
  "endDate": "2025-03-15T18:00",
  "eventStatus": "https://schema.org/EventScheduled",
  "eventAttendanceMode": "https://schema.org/OfflineEventAttendanceMode",
  "location": {
    "@type": "Place",
    "name": "Конгресс-холл",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "Улица Примерная, 1",
      "addressLocality": "Москва",
      "addressRegion": "Москва",
      "postalCode": "123456",
      "addressCountry": "RU"
    }
  },
  "organizer": {
    "@type": "Organization",
    "name": "Организатор",
    "url": "https://example.com"
  },
  "offers": {
    "@type": "Offer",
    "price": "5000",
    "priceCurrency": "RUB",
    "availability": "https://schema.org/InStock",
    "url": "https://example.com/buy"
  }
}
```

### Course (Курсы)
```json
{
  "@context": "https://schema.org",
  "@type": "Course",
  "name": "Основы веб-разработки",
  "description": "Полный курс по основам веб-разработки для начинающих",
  "provider": {
    "@type": "Organization",
    "name": "Образовательный центр",
    "sameAs": "https://example.com",
    "areaServed": {
      "@type": "Country",
      "name": "RU"
    }
  }
}
```

## Инструменты для работы с JSON-LD

### Валидаторы
- [Google Rich Results Test](https://search.google.com/test/rich-results)
- [Yandex Structured Data Validator](https://webmaster.yandex.ru/tools/microtest/)
- [Schema Markup Validator](https://validator.schema.org/)
- [JSON-LD Playground](https://json-ld.org/playground/)

### Генераторы
- Schema Markup Generator
- JSON-LD для WooCommerce (для интернет-магазинов)
- JSON-LD для новостных сайтов

## Ошибки при использовании JSON-LD

1. **Некорректный формат JSON** - проверяйте синтаксис
2. **Использование несуществующих свойств** - следуйте официальной документации
3. **Несоответствие данных реальности** - используйте достоверную информацию
4. **Дублирование данных** - избегайте дубликатов в разных форматах
5. **Игнорирование обновлений схемы** - следите за изменениями Schema.org

> [!warning] Важно
> Неправильное использование JSON-LD может привести к снижению видимости в поисковой выдаче. Убедитесь, что все данные достоверны и соответствуют реальному содержимому страницы.

## Совместимость с другими форматами

JSON-LD можно использовать вместе с:
- [[Schema.org]] (в виде JSON-LD)
- [[Open-Graph]] - для социальных сетей
- [[Twitter-Cards]] - для Twitter
- [[Мета-теги]] - общие метатеги

## См. также

- [[Schema.org]]
- [[Open-Graph]]
- [[Twitter-Cards]]
- [[SEO-метаданные]]
- [[Meta-теги]]
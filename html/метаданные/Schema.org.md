---
aliases: ["Schema.org", "Structured Data", "Схема.орг"]
tags: [html, metadata, structured-data, seo]
---

# Schema.org

Schema.org - это совместный проект Google, Microsoft, Yahoo и Yandex, предоставляющий схему для структурированных данных на веб-страницах. Эти данные помогают поисковым системам лучше понимать содержимое страницы и могут улучшить видимость в поисковой выдаче.

## Форматы структурированных данных

### JSON-LD (рекомендуемый)
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Заголовок статьи",
  "author": {
    "@type": "Person",
    "name": "Имя автора"
  },
  "datePublished": "2025-01-01",
  "description": "Описание статьи"
}
</script>
```

### Microdata
```html
<div itemscope itemtype="https://schema.org/Article">
  <h1 itemprop="headline">Заголовок статьи</h1>
  <div itemprop="author" itemscope itemtype="https://schema.org/Person">
    <span itemprop="name">Имя автора</span>
  </div>
</div>
```

### RDFa
```html
<div vocab="https://schema.org/" typeof="Article">
  <h1 property="headline">Заголовок статьи</h1>
  <div property="author" typeof="Person">
    <span property="name">Имя автора</span>
  </div>
</div>
```

## Типы сущностей Schema.org

### Article и NewsArticle
```json
{
  "@context": "https://schema.org",
  "@type": "NewsArticle",
  "headline": "Новость дня",
  "image": [
    "https://example.com/photos/1.jpg",
    "https://example.com/photos/2.jpg"
  ],
  "datePublished": "2025-01-01T12:00:00+07:00",
  "dateModified": "2025-01-01T12:30:00+07:00",
  "author": {
    "@type": "Person",
    "name": "Иван Иванов",
    "url": "https://example.com/ivan"
  }
}
```

### Organization
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Компания",
  "legalName": "ООО 'Компания'",
  "url": "https://example.com",
  "logo": "https://example.com/logo.png",
  "foundingDate": "2020",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "Улица Примерная, 1",
    "addressLocality": "Москва",
    "addressRegion": "Москва",
    "postalCode": "123456",
    "addressCountry": "RU"
  }
}
```

### LocalBusiness
```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Местный бизнес",
  "image": "https://example.com/image.jpg",
  "telephone": "+7 (495) 123-45-67",
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
  }
}
```

## Российские особенности 2025

В 2025 году в России:

- Яндекс активно использует Schema.org данные для своих сервисов
- Важно указывать российские форматы адресов и телефонов
- Используйте русскоязычные схемы для лучшего понимания контента
- Учитывайте требования российского законодательства при указании данных

> [!tip] Совет
> Для российских сайтов рекомендуется использовать Schema.org вместе с Яндекс.Вебмастер и Google Search Console для максимального эффекта.

## Популярные типы Schema.org в России

### BreadcrumbList
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [{
    "@type": "ListItem",
    "position": 1,
    "name": "Главная",
    "item": "https://example.com"
  },{
    "@type": "ListItem",
    "position": 2,
    "name": "Категория",
    "item": "https://example.com/category"
  }]
}
```

### Product
```json
{
  "@context": "https://schema.org/",
  "@type": "Product",
  "name": "Название товара",
  "image": "https://example.com/product.jpg",
  "description": "Описание товара",
  "sku": "PART_NUMBER",
  "offers": {
    "@type": "Offer",
    "price": "5000",
    "priceCurrency": "RUB",
    "availability": "https://schema.org/InStock",
    "seller": {
      "@type": "Organization",
      "name": "Магазин"
    }
  }
}
```

### Event
```json
{
  "@context": "https://schema.org",
  "@type": "Event",
  "name": "Название события",
  "startDate": "2025-01-01T19:00",
  "endDate": "2025-01-01T21:00",
  "location": {
    "@type": "Place",
    "name": "Место проведения",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "Улица Примерная, 1",
      "addressLocality": "Москва",
      "addressRegion": "Москва",
      "postalCode": "123456",
      "addressCountry": "RU"
    }
  }
}
```

## Проверка и валидация

Используйте следующие инструменты для проверки:

- [Google Rich Results Test](https://search.google.com/test/rich-results)
- [Yandex Structured Data Validator](https://webmaster.yandex.ru/tools/microtest/)
- [Schema Markup Validator](https://validator.schema.org/)

## Лучшие практики

1. Используйте JSON-LD вместо Microdata или RDFa
2. Указывайте только достоверную информацию
3. Включайте обязательные свойства для каждого типа
4. Тестируйте структурированные данные перед публикацией
5. Регулярно обновляйте данные при изменении контента

> [!warning] Важно
> Неправильное использование Schema.org может привести к санкциям со стороны поисковых систем. Убедитесь, что все данные достоверны и соответствуют реальному контенту страницы.

## Интеграция с другими метаданными

Schema.org хорошо сочетается с другими метаданными:

- [[Open-Graph]] - для социальных сетей
- [[Twitter-Cards]] - для Twitter
- [[JSON-LD-структурированные-данные]] - альтернативный формат
- [[SEO-метаданные]] - общие SEO практики

## См. также

- [[JSON-LD-структурированные-данные]]
- [[Open-Graph]]
- [[Twitter-Cards]]
- [[SEO-метаданные]]
- [[Мета-теги]]
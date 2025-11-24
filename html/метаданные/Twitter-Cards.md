---
aliases: ["Twitter Card", "Twitter Meta Tags", "Twitter Cards"]
tags: [html, metadata, social-sharing, twitter]
---

# Twitter-Cards

Twitter Cards - это специфические метатеги, которые позволяют показывать расширенный контент при публикации ссылок в Twitter. Эти теги создают богатый медиа-опыт для пользователей Twitter при клике на ссылки.

## Типы Twitter Cards

### Summary Card
```html
<meta name="twitter:card" content="summary">
<meta name="twitter:title" content="Заголовок страницы">
<meta name="twitter:description" content="Краткое описание">
<meta name="twitter:image" content="https://example.com/image.jpg">
```
Показывает изображение, заголовок, описание и имя сайта.

### Summary Card with Large Image
```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Заголовок страницы">
<meta name="twitter:description" content="Краткое описание">
<meta name="twitter:image" content="https://example.com/large-image.jpg">
```
Отображает большое изображение сверху карточки.

### App Card
```html
<meta name="twitter:card" content="app">
<meta name="twitter:app:id:iphone" content="appid">
<meta name="twitter:app:id:ipad" content="appid">
<meta name="twitter:app:id:googleplay" content="appid">
```
Для продвижения мобильных приложений.

### Player Card
```html
<meta name="twitter:card" content="player">
<meta name="twitter:player" content="https://example.com/player">
<meta name="twitter:player:width" content="480">
<meta name="twitter:player:height" content="360">
```
Для встраивания медиаконтента.

## Обязательные теги

Каждая Twitter Card требует следующие теги:

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@username">
```

- `twitter:card` - тип карточки
- `twitter:site` - @username владельца сайта

## Рекомендуемые теги

```html
<meta name="twitter:title" content="Заголовок (до 70 символов)">
<meta name="twitter:description" content="Описание (до 200 символов)">
<meta name="twitter:image" content="URL изображения">
```

## Требования к изображениям

- **Summary Card**: 280x150 пикселей (минимум 144x144)
- **Summary Card Large Image**: 1200x600 пикселей (минимум 300x157)
- **Максимальный размер файла**: 5 МБ
- **Форматы**: JPG, PNG, GIF
- **Соотношение сторон**: 2:1 для large image

## Российские особенности 2025

В 2025 году в России:

- Twitter официально недоступен, но используется через обходные методы
- Важно учитывать, что российские пользователи могут использовать Twitter через VPN
- Альтернативные платформы: Telegram, ВКонтакте, Одноклассники
- Рекомендуется использовать Twitter Cards в дополнение к [[Open-Graph]] тегам

## Проверка и отладка

Используйте Twitter Card Validator для проверки:
- [Twitter Card Validator](https://cards-dev.twitter.com/validator)

> [!tip] Совет
> После изменения тегов Twitter Cards обязательно протестируйте их через валидатор, так как Twitter кэширует метаданные.

## Пример полного набора тегов

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@mysite">
<meta name="twitter:title" content="Интересная статья">
<meta name="twitter:description" content="Узнайте больше об этой теме">
<meta name="twitter:image" content="https://example.com/image.jpg">
<meta name="twitter:image:alt" content="Описание изображения">
```

## Дополнительные теги

### twitter:image:alt
```html
<meta name="twitter:image:alt" content="Описание изображения">
```
Доступность: описание изображения для пользователей с нарушениями зрения.

### twitter:creator
```html
<meta name="twitter:creator" content="@author">
```
@username автора контента.

## Совместимость с Open Graph

Twitter Cards можно использовать вместе с [[Open-Graph]] тегами:

```html
<!-- Open Graph -->
<meta property="og:title" content="Заголовок">
<meta property="og:description" content="Описание">
<meta property="og:image" content="https://example.com/image.jpg">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Заголовок">
<meta name="twitter:description" content="Описание">
<meta name="twitter:image" content="https://example.com/image.jpg">
```

## См. также

- [[Open-Graph]]
- [[Мета-теги]]
- [[SEO-метаданные]]
- [[Мета-теги-для-мобильных]]
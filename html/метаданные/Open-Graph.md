---
aliases: ["Open Graph", "OG Tags", "Open Graph Protocol"]
tags: [html, metadata, social-sharing, seo]
---

# Open-Graph

Open Graph протокол был разработан Facebook для стандартизации способа, которым веб-страницы представлены при ссылке на них в социальных сетях. Эти теги позволяют контролировать, как контент отображается при публикации в социальных сетях.

## Основные Open Graph теги

### og:title
```html
<meta property="og:title" content="Заголовок страницы" />
```
Указывает заголовок контента, когда он будет поделен.

### og:type
```html
<meta property="og:type" content="website" />
```
Определяет тип контента (website, article, video, etc.).

### og:image
```html
<meta property="og:image" content="https://example.com/image.jpg" />
```
URL изображения, которое будет отображаться при публикации ссылки.

> [!tip] Рекомендация
> Используйте изображения размером 1200x630 пикселей для оптимального отображения в социальных сетях.

### og:url
```html
<meta property="og:url" content="https://example.com/page" />
```
Постоянный URL-адрес ресурса.

### og:description
```html
<meta property="og:description" content="Описание страницы" />
```
Описание контента страницы.

### og:site_name
```html
<meta property="og:site_name" content="Название сайта" />
```
Имя сайта, на котором публикуется контент.

## Российские социальные сети

В 2025 году в России популярны следующие социальные платформы, которые поддерживают Open Graph:

- [[ВКонтакте]] - ВКонтакте активно использует Open Graph теги для предварительного просмотра ссылок
- Одноклассники - также поддерживает Open Graph протокол
- Telegram - частично использует OG теги для предварительного просмотра

## Требования к изображениям

- **Рекомендуемый размер**: 1200x630 пикселей
- **Максимальный размер файла**: 8 МБ
- **Форматы**: JPG, PNG, GIF
- **Соотношение сторон**: 1.91:1 (предпочтительно)

## Дополнительные теги

### og:locale
```html
<meta property="og:locale" content="ru_RU" />
```
Язык и локаль контента (для России: ru_RU).

### og:video
```html
<meta property="og:video" content="https://example.com/video.mp4" />
```
Для встраивания видео.

### article:publisher
```html
<meta property="article:publisher" content="https://www.facebook.com/publisher" />
```
Ссылка на публикующую организацию (Facebook).

## Практические рекомендации

1. **Тестирование**: Используйте инструменты:
   - [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/)
   - [VK Share Preview](https://vk.com/dev/Share)

2. **Кэширование**: Социальные сети кэшируют OG теги, поэтому после изменений может потребоваться очистка кэша.

3. **Адаптивность**: Убедитесь, что изображения корректно отображаются на мобильных устройствах.

## Проверка и отладка

Для проверки корректности Open Graph тегов используйте:

- Facebook Sharing Debugger
- VK Share Preview
- Telegram Bot @WebpageBot
- Яндекс.Вебмастер (для Яндекса)

> [!warning] Важно
> В России в 2025 году из-за санкций и ограничений на западные платформы важно тестировать OG теги в отечественных социальных сетях и мессенджерах.

## Пример полного набора тегов

```html
<meta property="og:title" content="Заголовок статьи" />
<meta property="og:type" content="article" />
<meta property="og:image" content="https://example.com/images/article.jpg" />
<meta property="og:url" content="https://example.com/article" />
<meta property="og:description" content="Описание статьи" />
<meta property="og:site_name" content="Мой сайт" />
<meta property="og:locale" content="ru_RU" />
<meta property="article:author" content="Автор статьи" />
<meta property="article:published_time" content="2025-01-01T12:00:00+00:00" />
```

## См. также

- [[Twitter-Cards]]
- [[Мета-теги]]
- [[SEO-метаданные]]
- [[Альтернативные-версии]]
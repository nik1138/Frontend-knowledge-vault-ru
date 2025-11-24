---
aliases: ["Веб-манифест", "Manifest", "Web App Manifest"]
tags: ["#pwa", "#web", "#mobile", "#installation", "#ui"]
---

# Web App Manifest

Web App Manifest - это JSON-файл, который предоставляет браузеру информацию о веб-приложении при установке на мобильное или десктопное устройство. Манифест позволяет определить, как приложение будет отображаться и вести себя вне браузера.

## Основные свойства манифеста

### Обязательные свойства

```json
{
  "name": "Мое приложение",
  "short_name": "Приложение",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

### Описание основных свойств

- **name** - полное название приложения (до 45 символов)
- **short_name** - краткое название приложения для иконки (до 12 символов)
- **start_url** - URL, с которого начинается приложение при запуске
- **display** - способ отображения приложения (fullscreen, standalone, minimal-ui, browser)
- **background_color** - цвет фона экрана-заставки при загрузке
- **theme_color** - цвет темы для окна браузера и UI-элементов
- **icons** - массив иконок приложения различных размеров

## Дополнительные свойства

```json
{
  "name": "Мое приложение",
  "short_name": "Приложение",
  "description": "Описание моего приложения",
  "start_url": "/?utm_source=web_app_manifest",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable"
    },
    {
      "src": "/icons/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "categories": ["productivity", "utilities"],
  "lang": "ru-RU",
  "dir": "ltr",
  "screenshots": [
    {
      "src": "/screenshots/screen1.png",
      "sizes": "1280x720",
      "type": "image/png",
      "label": "Главная страница"
    }
  ],
  "shortcuts": [
    {
      "name": "Каталог товаров",
      "short_name": "Каталог",
      "description": "Просмотр каталога товаров",
      "url": "/catalog",
      "icons": [
        {
          "src": "/icons/catalog-icon.png",
          "sizes": "96x96"
        }
      ]
    }
  ]
}
```

### Подробное описание дополнительных свойств

- **description** - описание приложения
- **scope** - область действия приложения (URL, в пределах которой работает PWA)
- **orientation** - ориентация экрана (any, natural, landscape, portrait)
- **categories** - категории приложения
- **lang** - язык приложения
- **dir** - направление текста (ltr, rtl, auto)
- **screenshots** - скриншоты для магазина приложений
- **shortcuts** - ярлыки для быстрого доступа к функциям приложения

## Типы отображения

### fullscreen
Приложение занимает весь экран, скрывая все элементы ОС.

### standalone
Приложение выглядит как нативное приложение с собственной панелью навигации.

### minimal-ui
Приложение отображается как в браузере, но с минимальными элементами управления.

### browser
Приложение отображается в стандартном окне браузера.

## Рекомендации по иконкам

Для корректного отображения иконок рекомендуется:

- Использовать квадратные изображения (например, 192x192, 512x512)
- Обеспечить прозрачный фон для масштабируемых иконок
- Использовать формат PNG
- Предоставить несколько размеров для разных устройств

### Maskable иконки

Maskable иконки позволяют браузеру обрезать иконку по маске, характерной для платформы:

```json
{
  "src": "/icons/icon-192x192.png",
  "sizes": "192x192",
  "type": "image/png",
  "purpose": "maskable"
}
```

## Подключение манифеста

Для подключения манифеста к HTML-документу используйте следующий тег:

```html
<link rel="manifest" href="/manifest.json">
```

## Примеры манифестов для разных типов приложений

### Для веб-магазина

```json
{
  "name": "Интернет-магазин товаров",
  "short_name": "Магазин",
  "description": "Широкий выбор товаров для дома и офиса",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#f5f5f5",
  "theme_color": "#2196f3",
  "icons": [
    {
      "src": "/icons/shop-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/icons/shop-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/icons/shop-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/icons/shop-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/icons/shop-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/icons/shop-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/shop-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/icons/shop-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "categories": ["shopping", "e-commerce"],
  "lang": "ru-RU",
  "dir": "ltr"
}
```

### Для новостного приложения

```json
{
  "name": "Новости дня",
  "short_name": "Новости",
  "description": "Последние новости и аналитика",
  "start_url": "/?source=pwa",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "background_color": "#ffffff",
  "theme_color": "#ff5722",
  "icons": [
    {
      "src": "/icons/news-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/news-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "categories": ["news", "information"],
  "lang": "ru-RU",
  "dir": "ltr",
  "screenshots": [
    {
      "src": "/screenshots/main-news.png",
      "sizes": "1080x1920",
      "type": "image/png",
      "label": "Главная страница новостей"
    }
  ]
}
```

## Проверка валидности манифеста

Для проверки корректности манифеста можно использовать:

- Инструменты разработчика в Chrome DevTools (Application → Manifest)
- Онлайн-валидаторы манифестов
- Lighthouse (инструмент аудита PWA)

## Совместимость и поддержка

Web App Manifest поддерживается всеми современными браузерами:
- Chrome: с версии 38
- Firefox: с версии 45
- Safari: с версии 11.1 (ограниченная поддержка)
- Edge: с версии 79

> [!tip] Совет
> Всегда указывайте fallback для старых браузеров с помощью тегов `<meta>` и `<link>` для иконок и темы.

## Практические рекомендации для российских разработчиков

В 2025 году в России наблюдается рост популярности PWA-приложений, особенно в связи с ограничениями на традиционные магазины приложений. При создании Web App Manifest для российского рынка рекомендуется:

- Указывать `lang: "ru-RU"` для корректной локализации
- Использовать цвета, соответствующие российским предпочтениям
- Обеспечивать поддержку кириллических символов в названиях
- Учитывать особенности мобильных устройств, популярных в России
- Обеспечивать быструю загрузку и корректное отображение на слабых устройствах

## Рекомендации по SEO и видимости

- Указывайте корректное описание в поле `description`
- Используйте релевантные категории
- Обеспечьте уникальные и информативные названия
- Поддерживайте актуальность манифеста при обновлениях приложения

## Связанные темы

- [[Service Workers]]
- [[Оффлайн-работа]]
- [[Кэширование]]
- [[Современные практики JavaScript]]
- [[Мобильная оптимизация]]
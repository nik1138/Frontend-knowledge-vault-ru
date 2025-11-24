---
aliases: ["Manifest", "Web Manifest", "PWA Manifest"]
tags: [pwa, web-manifest, json, app-install, metadata]
---

# Web App Manifest

Web App Manifest — это JSON-файл, который предоставляет браузеру информацию о веб-приложении при его установке на устройство пользователя. Manifest позволяет определить, как приложение будет выглядеть и вести себя на домашнем экране пользователя.

## Назначение Web App Manifest

Web App Manifest позволяет:

- Определить имя приложения и описание
- Указать иконки для различных разрешений
- Задать начальный URL для запуска
- Определить режим отображения (fullscreen, standalone, minimal-ui, browser)
- Указать тему и цвет фона
- Определить ориентацию экрана
- Задать язык приложения

## Базовая структура manifest.json

```json
{
  "name": "Мое PWA приложение",
  "short_name": "МоеPWA",
  "description": "Описание моего прогрессивного веб-приложения",
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

## Обязательные поля

- `name` — полное имя приложения (длина до 45 символов)
- `short_name` — краткое имя приложения (длина до 12 символов)
- `icons` — массив иконок приложения различных размеров
- `start_url` — URL, с которого начинается приложение при запуске

## Опциональные поля

- `display` — режим отображения приложения
- `orientation` — ориентация экрана
- `background_color` — цвет фона при загрузке
- `theme_color` — цвет темы приложения
- `description` — описание приложения
- `lang` — язык приложения
- `dir` — направление текста (ltr, rtl, auto)
- `scope` — область действия приложения
- `categories` — категории приложения
- `iarc_rating_id` — ID для возрастного рейтинга
- `screenshots` — скриншоты приложения
- `related_applications` — связанные нативные приложения
- `prefer_related_applications` — предпочтение нативных приложений

## Режимы отображения

- `fullscreen` — приложение занимает весь экран
- `standalone` — приложение выглядит как нативное приложение
- `minimal-ui` — минимальный набор элементов управления браузера
- `browser` — обычный веб-браузер

## Иконки

Для корректной работы на различных устройствах рекомендуется использовать следующие размеры иконок:

```json
"icons": [
  {
    "src": "/icons/maskable_icon_x192.png",
    "sizes": "192x192",
    "type": "image/png",
    "purpose": "maskable"
  },
  {
    "src": "/icons/maskable_icon_x512.png",
    "sizes": "512x512",
    "type": "image/png",
    "purpose": "maskable"
  },
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
```

## Подключение Web App Manifest

Для подключения Web App Manifest добавьте в `<head>` HTML-документа:

```html
<link rel="manifest" href="/manifest.json">
```

## Проверка Web App Manifest

Для проверки корректности manifest.json можно использовать:

- Chrome DevTools → Application → Manifest
- Валидаторы JSON
- Lighthouse
- Web App Manifest Validator

## Особенности для российского рынка

При создании Web App Manifest для российского рынка 2025 года необходимо учитывать:

- Использование русского языка в названиях и описаниях
- Поддержка российских браузеров (Яндекс.Браузер, Mail.Ru Browser)
- Учет требований ФЗ-152 к обработке персональных данных
- Оптимизация иконок под российские устройства
- Учет специфики установки приложений в условиях ограничений на Google Play

## Пример продвинутого manifest.json

```json
{
  "name": "Мое PWA приложение для России",
  "short_name": "МоеPWA",
  "description": "Прогрессивное веб-приложение, оптимизированное для российского рынка",
  "start_url": "/?utm_source=homescreen",
  "display": "standalone",
  "background_color": "#f0f0f0",
  "theme_color": "#1a73e8",
  "lang": "ru-RU",
  "dir": "ltr",
  "scope": "/",
  "categories": ["social", "utilities"],
  "icons": [
    {
      "src": "/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
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
  "screenshots": [
    {
      "src": "/screenshots/screen1.webp",
      "sizes": "1280x720",
      "type": "image/webp",
      "label": "Главная страница приложения"
    },
    {
      "src": "/screenshots/screen2.webp",
      "sizes": "1280x720",
      "type": "image/webp",
      "label": "Страница профиля"
    }
  ]
}
```

## Совместимость с браузерами

Web App Manifest поддерживается большинством современных браузеров:

- Chrome (Android и Desktop)
- Firefox (Android и Desktop)
- Safari (iOS и macOS)
- Edge
- Яндекс.Браузер
- Mail.Ru Browser

## Заключение

Web App Manifest является важной частью PWA, которая позволяет веб-приложениям вести себя как нативные. Правильно настроенный manifest улучшает пользовательский опыт и способствует установке приложения на устройства пользователей.

> [!tip] Совет
> Используйте инструменты автоматической генерации иконок для создания всех необходимых размеров из одного исходного изображения.

> [!warning] Важно
> В России могут действовать ограничения на установку веб-приложений в корпоративных сетях, поэтому обязательно тестируйте установку в различных условиях.

Следующие темы: [[Введение в PWA]], [[Service Workers]], [[Кэширование]], [[Оффлайн работа]]
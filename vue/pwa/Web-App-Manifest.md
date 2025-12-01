# Web App Manifest в PWA на Vue.js

**Alias**: [[Веб-манифест]]

**Tags**: #vue #pwa #web-manifest #installation #ui #performance

Web App Manifest - это JSON-файл, который предоставляет браузеру информацию о веб-приложении, чтобы сделать его installable и обеспечить native app-like experience.

## Основные параметры манифеста

```json
{
  "name": "Мое Vue PWA приложение",
  "short_name": "VuePWA",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#42b549",
  "icons": [
    {
      "src": "/img/icons/android-chrome-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    }
  ]
}
```

## Интеграция в Vue.js

При использовании `@vue/cli-plugin-pwa`, манифест автоматически подключается и может быть настроен через `vue.config.js`.

## Рекомендации по иконкам

- Использовать формат PNG
- Предоставлять несколько размеров (192x192, 512x512)
- Учитывать требования различных платформ (Android, iOS, Windows)

## Российские особенности (2025)

- Поддержка отечественных платформ и маркетов
- Учет требований российского законодательства
- Локализация названий и описаний

## Связанные темы

- [[Service-Workers]]
- [[Установка-PWA]]

## Структура Web App Manifest

Web App Manifest - это JSON-файл, который содержит метаданные о веб-приложении. Он позволяет контролировать отображение приложения при установке на различных платформах.

### Обязательные поля

- `name` - полное название приложения
- `short_name` - краткое название для отображения на домашнем экране
- `start_url` - начальный URL приложения при запуске
- `display` - способ отображения приложения (standalone, fullscreen, minimal-ui)
- `icons` - массив иконок различных размеров

### Дополнительные поля

- `description` - описание приложения
- `lang` - основной язык приложения
- `dir` - направление текста (ltr, rtl, auto)
- `orientation` - предпочтительная ориентация экрана
- `theme_color` - цвет темы для UI браузера
- `background_color` - цвет фона при загрузке приложения
- `categories` - категории приложения
- `screenshots` - скриншоты приложения

## Пример расширенного манифеста

```json
{
  "name": "Мое Vue PWA приложение",
  "short_name": "VuePWA",
  "description": "Прогрессивное веб-приложение на Vue.js",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait",
  "theme_color": "#42b549",
  "background_color": "#ffffff",
  "icons": [
    {
      "src": "/img/icons/android-chrome-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/img/icons/android-chrome-96x96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/img/icons/android-chrome-128x128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/img/icons/android-chrome-144x144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/img/icons/android-chrome-152x152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/img/icons/android-chrome-192x192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/img/icons/android-chrome-384x384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "any"
    },
    {
      "src": "/img/icons/android-chrome-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any"
    }
  ],
  "screenshots": [
    {
      "src": "/img/screenshots/phone-home.png",
      "sizes": "640x1136",
      "type": "image/png",
      "platform": "narrow",
      "label": "Главная страница"
    },
    {
      "src": "/img/screenshots/phone-settings.png",
      "sizes": "640x1136",
      "type": "image/png",
      "platform": "narrow",
      "label": "Настройки"
    }
  ],
  "categories": [
    "productivity",
    "utilities"
  ],
  "lang": "ru-RU",
  "dir": "ltr"
}
```

## Локализация манифеста

Для поддержки нескольких языков можно использовать локализованные поля:

```json
{
  "name": "My App",
  "short_name": "MyApp",
  "name": {
    "en-US": "My Application",
    "ru-RU": "Мое Приложение"
  },
  "short_name": {
    "en-US": "MyApp",
    "ru-RU": "Мое Прил"
  }
}
```

## Проверка валидности манифеста

Для проверки корректности Web App Manifest можно использовать инструменты:
- Chrome DevTools - вкладка Application
- Валидаторы онлайн
- Lighthouse

## Интеграция с Vue CLI

При использовании Vue CLI с PWA плагином, манифест автоматически генерируется и подключается. Основные настройки можно изменить в файле `vue.config.js`:

```javascript
module.exports = {
  pwa: {
    name: 'Мое Vue PWA приложение',
    themeColor: '#42b549',
    msTileColor: '#000000',
    appleMobileWebAppCapable: 'yes',
    appleMobileWebAppStatusBarStyle: 'black',
    iconPaths: {
      favicon32: 'img/icons/favicon-32x32.png',
      favicon16: 'img/icons/favicon-16x16.png',
      appleTouchIcon: 'img/icons/apple-touch-icon-152x152.png',
      maskIcon: 'img/icons/safari-pinned-tab.svg',
      msTileImage: 'img/icons/msapplication-icon-144x144.png'
    }
  }
}
```
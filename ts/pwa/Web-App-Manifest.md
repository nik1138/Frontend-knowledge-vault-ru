---
aliases: ["Манифест веб-приложения", "Web App Manifest", "manifest.json"]
tags: ["#pwa", "#typescript", "#web", "#installation", "#metadata"]
---

# Web App Manifest

Web App Manifest - это JSON-файл, который предоставляет браузеру информацию о веб-приложении, когда оно добавляется на домашний экран мобильного устройства или устанавливается на десктоп. Манифест позволяет контролировать внешний вид и поведение приложения, включая иконки, имя, тему и режим отображения.

## Основные поля манифеста

Манифест веб-приложения - это JSON-файл, обычно называемый `manifest.json`, который содержит следующие основные поля:

### `name` и `short_name`

Эти поля определяют имя приложения, которое будет отображаться на домашнем экране или в списке установленных приложений.

```json
{
  "name": "Мое потрясающее Progressive Web App",
  "short_name": "MyPWA"
}
```

### `description`

Описание приложения, которое может использоваться браузером или магазином приложений для предоставления информации пользователю.

```json
{
  "description": "Мое Progressive Web App с офлайн-функциональностью и push-уведомлениями"
}
```

### `start_url`

URL-адрес, который будет открыт при запуске установленного PWA. Обычно это корневой URL или главная страница приложения.

```json
{
  "start_url": "/",
  "scope": "/"
}
```

Поле `scope` определяет область приложения, в пределах которой будет работать манифест.

### `display`

Определяет режим отображения приложения при запуске. Возможные значения:

- `fullscreen` - приложение занимает весь экран
- `standalone` - приложение выглядит как нативное приложение
- `minimal-ui` - минимальный набор элементов управления браузера
- `browser` - обычное окно браузера

```json
{
  "display": "standalone"
}
```

### `theme_color` и `background_color`

Цвет темы и фоновый цвет приложения, которые влияют на внешний вид панели навигации и заставки при загрузке.

```json
{
  "theme_color": "#0000ff",
  "background_color": "#ffffff"
}
```

### `icons`

Массив иконок приложения различных размеров для разных устройств и экранов.

```json
{
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
  ]
}
```

## Пример полного манифеста

```json
{
  "name": "Мое Progressive Web App",
  "short_name": "MyPWA",
  "description": "Пример PWA с использованием TypeScript",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "theme_color": "#0000ff",
  "background_color": "#ffffff",
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
  ]
}
```

## Подключение манифеста к HTML

Для подключения манифеста к веб-странице необходимо добавить соответствующий тег `<link>` в секцию `<head>` HTML-документа:

```html
<link rel="manifest" href="/manifest.json">
```

## Генерация манифеста с помощью TypeScript

Для автоматизации создания манифеста можно использовать TypeScript-скрипты. Например, при сборке проекта можно генерировать манифест на основе конфигурации:

```typescript
// manifest-generator.ts
interface ManifestIcon {
  src: string;
  sizes: string;
  type: string;
}

interface WebAppManifest {
  name: string;
  short_name: string;
  description: string;
  start_url: string;
  scope: string;
  display: 'fullscreen' | 'standalone' | 'minimal-ui' | 'browser';
  theme_color: string;
  background_color: string;
  icons: ManifestIcon[];
}

const generateManifest = (config: WebAppManifest): string => {
  return JSON.stringify(config, null, 2);
};

const manifestConfig: WebAppManifest = {
  name: 'Мое Progressive Web App',
  short_name: 'MyPWA',
  description: 'Пример PWA с использованием TypeScript',
  start_url: '/',
  scope: '/',
  display: 'standalone',
  theme_color: '#0000ff',
  background_color: '#ffffff',
  icons: [
    {
      src: '/icons/icon-72x72.png',
      sizes: '72x72',
      type: 'image/png'
    },
    {
      src: '/icons/icon-96x96.png',
      sizes: '96x96',
      type: 'image/png'
    },
    {
      src: '/icons/icon-128x128.png',
      sizes: '128x128',
      type: 'image/png'
    },
    {
      src: '/icons/icon-144x144.png',
      sizes: '144x144',
      type: 'image/png'
    },
    {
      src: '/icons/icon-152x152.png',
      sizes: '152x152',
      type: 'image/png'
    },
    {
      src: '/icons/icon-192x192.png',
      sizes: '192x192',
      type: 'image/png'
    },
    {
      src: '/icons/icon-384x384.png',
      sizes: '384x384',
      type: 'image/png'
    },
    {
      src: '/icons/icon-512x512.png',
      sizes: '512x512',
      type: 'image/png'
    }
  ]
};

const manifestString = generateManifest(manifestConfig);
console.log(manifestString);
```

## Проверка манифеста

Для проверки корректности манифеста можно использовать инструменты разработчика в браузере Chrome:

1. Откройте DevTools (F12)
2. Перейдите на вкладку "Application"
3. Выберите "Manifest" в левой панели
4. Проверьте, что все поля заполнены корректно и иконки отображаются

## Типизация манифеста в TypeScript

Для работы с манифестом в TypeScript можно определить интерфейсы, соответствующие спецификации Web App Manifest:

```typescript
interface WebAppManifest {
  name?: string;
  short_name?: string;
  description?: string;
  start_url?: string;
  scope?: string;
  display?: 'fullscreen' | 'standalone' | 'minimal-ui' | 'browser';
  orientation?: 'any' | 'natural' | 'landscape' | 'portrait-up' | 'portrait-down' | 'landscape-up' | 'landscape-down';
  theme_color?: string;
  background_color?: string;
  icons?: ManifestImageResource[];
  related_applications?: RelatedApplication[];
  prefer_related_applications?: boolean;
  categories?: string[];
  iarc_rating_id?: string;
  screenshots?: ManifestImageResource[];
  shortcuts?: ShortcutItem[];
}

interface ManifestImageResource {
  src: string;
  sizes?: string;
  type?: string;
  purpose?: 'monochrome' | 'maskable' | 'any';
}

interface RelatedApplication {
  platform: string;
  url?: string;
  id?: string;
}

interface ShortcutItem {
  name: string;
  short_name?: string;
  description?: string;
  url: string;
  icons?: ManifestImageResource[];
}
```

## Полезные ресурсы

- [Официальная спецификация Web App Manifest](https://www.w3.org/TR/appmanifest/)
- [[Service-Workers]] - для понимания полного цикла PWA
- [[Установка-PWA]] - процесс установки приложения
- [[Офлайн-работа]] - стратегии работы без подключения к интернету
- [Google Developers - Web App Manifest](https://developers.google.com/web/fundamentals/web-app-manifest)

## Заключение

Web App Manifest является важной частью Progressive Web App, позволяя определить метаданные приложения и улучшить пользовательский опыт при установке и запуске. Правильная настройка манифеста с использованием TypeScript-интерфейсов помогает избежать ошибок и упрощает поддержку проекта.
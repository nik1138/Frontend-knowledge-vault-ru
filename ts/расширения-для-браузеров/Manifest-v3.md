---
aliases: [Манифест v3, Manifest V3]
tags: [browser-extension, manifest-v3, chrome-extension, web-extension]
---

# Manifest V3: Современная архитектура расширений браузера

## Обзор

Manifest V3 - это третья версия спецификации манифеста для расширений браузеров, введенная Google Chrome в 2021 году. Она представляет собой значительное обновление по сравнению с Manifest V2 и вводит новые возможности безопасности, производительности и приватности.

## Основные изменения в Manifest V3

### 1. Service Worker вместо Background Pages

В Manifest V3 фоновые скрипты теперь выполняются в Service Worker, а не в постоянной фоновой странице.

```json
{
  "manifest_version": 3,
  "name": "Пример расширения",
  "version": "1.0",
  "background": {
    "service_worker": "background.js"
  },
  "permissions": [
    "activeTab",
    "storage"
  ]
}
```

### 2. Улучшенная безопасность с declarativeNetRequest

API `webRequest` был заменен на `declarativeNetRequest`, который обеспечивает более строгую безопасность и производительность.

```json
{
  "permissions": [
    "declarativeNetRequest",
    "declarativeNetRequestWithHostAccess"
  ],
  "host_permissions": [
    "https://example.com/*"
  ]
}
```

### 3. Изменения в Content Security Policy

Manifest V3 требует более строгой Content Security Policy (CSP), которая запрещает встроенные скрипты и eval().

```json
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'"
  }
}
```

## Структура манифеста

### Обязательные поля

- `manifest_version`: должно быть равно 3
- `name`: имя расширения
- `version`: версия расширения

### Часто используемые поля

- `description`: описание расширения
- `icons`: иконки расширения
- `action`: настройки popup-интерфейса
- `permissions`: разрешения, необходимые расширению
- `host_permissions`: доступ к внешним сайтам
- `content_scripts`: настройки content-скриптов
- `background`: настройки фонового скрипта

## Пример полного манифеста

```json
{
  "manifest_version": 3,
  "name": "Пример расширения с TypeScript",
  "version": "1.0.0",
  "description": "Демонстрация использования TypeScript в расширении браузера",
  "icons": {
    "16": "images/icon16.png",
    "32": "images/icon32.png",
    "48": "images/icon48.png",
    "128": "images/icon128.png"
  },
  "action": {
    "default_popup": "popup.html",
    "default_title": "Пример расширения",
    "default_icon": {
      "16": "images/icon16.png",
      "32": "images/icon32.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://*/*", "http://*/*"],
      "js": ["content-script.js"],
      "css": ["content-style.css"]
    }
  ],
  "permissions": [
    "activeTab",
    "storage",
    "tabs"
  ],
  "host_permissions": [
    "https://api.example.com/*",
    "https://*.github.com/*"
  ]
}
```

## Преимущества Manifest V3

1. **Улучшенная безопасность**: более строгие политики безопасности
2. **Лучшая производительность**: Service Worker потребляет меньше ресурсов
3. **Улучшенная приватность**: более точный контроль доступа к данным
4. **Будущая совместимость**: поддержка новых функций браузера

## Миграция с Manifest V2

При миграции с Manifest V2 необходимо:

1. Изменить `manifest_version` на 3
2. Заменить `background.page` на `background.service_worker`
3. Обновить CSP политику
4. Заменить `webRequest` на `declarativeNetRequest` при необходимости
5. Обновить разрешения и хост-разрешения

## Связанные темы

- [[Content-scripts]]
- [[Background-scripts]]
- [[Popup-интерфейс]]
- [[DevTools-расширения]]

## Теги

#browser-extension #manifest-v3 #chrome-extension #web-extension #typescript
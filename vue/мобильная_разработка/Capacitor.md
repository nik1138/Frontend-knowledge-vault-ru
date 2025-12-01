---
aliases: ["Capacitor", "CapacitorJS", "Капацитор"]
tags: ["#vue", "#mobile", "#cordova", "#native-api", "#hybrid-apps"]
---

# Capacitor

## Общее описание

Capacitor - это кроссплатформенный нативный runtime, разработанныный Ionic Team, который позволяет веб-разработчикам создавать нативные мобильные приложения с использованием веб-технологий. Он предоставляет современную альтернативу Apache Cordova и идеально интегрируется с Vue.js приложениями.

## Архитектура и возможности

Capacitor работает как мост между веб-приложением и нативной платформой, позволяя использовать нативные API через JavaScript. В отличие от Cordova, Capacitor предоставляет более современный подход к разработке гибридных приложений.

### Основные особенности

- Современная архитектура с поддержкой нативных плагинов
- Интеграция с популярными фреймворками (Vue, React, Angular)
- Поддержка iOS и Android
- Возможность добавления нативного кода
- Совместимость с существующими Cordova плагинами
- Поддержка PWA и веб-функций

## Установка и настройка

Для добавления Capacitor в Vue.js проект выполните следующие шаги:

```bash
npm install @capacitor/core @capacitor/cli
npx cap init
```

Для интеграции с Vue.js проектом:

```bash
npm install @capacitor/core @capacitor/cli @capacitor/android @capacitor/ios
npx cap init MyApp com.example.myapp
```

## Конфигурация проекта

Создание файла конфигурации `capacitor.config.ts`:

```typescript
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.example.myapp',
  appName: 'MyApp',
  webDir: 'dist',
  bundledWebRuntime: false,
  plugins: {
    SplashScreen: {
      launchShowDuration: 3000,
      launchAutoHide: true,
      launchFadeOutDuration: 3000,
      backgroundColor: "#ffffffff",
      androidSplashResourceName: "splash",
      androidScaleType: "CENTER_CROP",
      showSpinner: true,
      androidSpinnerStyle: "large",
      iosSpinnerStyle: "small",
      spinnerColor: "#999999",
      splashFullScreen: true,
      splashImmersive: true,
      layoutName: "launch_screen",
      useDialog: true,
    }
  }
};

export default config;
```

## Интеграция с Vue.js

Для интеграции с Vue.js приложением необходимо настроить сборку проекта:

```javascript
// vue.config.js
module.exports = {
  publicPath: './',
  outputDir: 'dist',
  assetsDir: 'static',
  indexPath: 'index.html',
  filenameHashing: true
}
```

После сборки Vue приложения выполните:

```bash
npm run build
npx cap add android
npx cap add ios
npx cap sync
npx cap open android
```

## Использование нативных API

Capacitor предоставляет доступ к нативным возможностям устройства через JavaScript API:

```javascript
// Использование камеры
import { Camera, CameraResultType } from '@capacitor/camera';

const takePicture = async () => {
  const image = await Camera.getPhoto({
    quality: 90,
    allowEditing: true,
    resultType: CameraResultType.Uri
  });

  // image.webPath содержит путь к изображению
  console.log(image.webPath);
};

// Использование геолокации
import { Geolocation } from '@capacitor/geolocation';

const getCurrentPosition = async () => {
  const coordinates = await Geolocation.getCurrentPosition();
  console.log('Широта:', coordinates.coords.latitude);
  console.log('Долгота:', coordinates.coords.longitude);
};

// Использование файловой системы
import { Filesystem, Directory } from '@capacitor/filesystem';

const writeFile = async () => {
  const result = await Filesystem.writeFile({
    path: 'secrets/text.txt',
    data: 'Это секретный текст!',
    directory: Directory.Documents,
  });
  console.log('Файл записан:', result.uri);
};
```

## Создание кастомных плагинов

Capacitor позволяет создавать собственные плагины для доступа к специфичным нативным возможностям:

```bash
npx @capacitor/cli plugin:generate
```

Пример структуры кастомного плагина:

```
my-plugin/
├── src/
│   └── web.ts
├── android/
│   ├── src/
│   └── build.gradle
├── ios/
│   ├── Plugin/
│   └── Plugin.podspec
└── MyPlugin.ts
```

## Практические рекомендации

### Оптимизация производительности

- Минимизируйте количество нативных вызовов
- Используйте асинхронные операции для тяжелых задач
- Оптимизируйте размер веб-приложения
- Кэшируйте часто используемые данные

### Обработка ошибок

```javascript
import { Camera, CameraResultType, CameraSource } from '@capacitor/camera';

const takePicture = async () => {
  try {
    const image = await Camera.getPhoto({
      quality: 90,
      allowEditing: true,
      resultType: CameraResultType.Uri,
      source: CameraSource.Camera
    });
    
    return image;
  } catch (error) {
    console.error('Ошибка камеры:', error);
    
    // Обработка ошибки
    if (error.message.includes('permission')) {
      // Запрос разрешения на использование камеры
      console.log('Необходимо разрешение на использование камеры');
    }
    
    return null;
  }
};
```

### Работа с разрешениями

Capacitor предоставляет систему управления разрешениями:

```javascript
import { Permissions } from '@capacitor/core';

const requestCameraPermission = async () => {
  const result = await Permissions.request({
    name: 'camera'
  });
  
  return result.state === 'granted';
};
```

## Российские особенности

В 2025 году в России Capacitor становится все более популярным для разработки мобильных приложений, особенно в условиях необходимости поддержки альтернативных магазинов приложений и ограничений на доступ к некоторым западным сервисам. Capacitor позволяет создавать приложения, которые могут быть опубликованы в различных магазинах.

### Интеграция с российскими сервисами

Для российского рынка важно интегрировать приложения с местными сервисами:

```javascript
// Интеграция с push-уведомлениями через российские сервисы
import { PushNotifications } from '@capacitor/push-notifications';

const registerPushNotifications = async () => {
  try {
    const result = await PushNotifications.requestPermissions();
    
    if (result.receive === 'granted') {
      await PushNotifications.register();
    }
    
    PushNotifications.addListener('registration', token => {
      console.log('Push registration success, token:', token.value);
      // Отправка токена на сервер для интеграции с российскими push-сервисами
    });
    
    PushNotifications.addListener('pushNotificationReceived', notification => {
      console.log('Push received:', notification);
    });
  } catch (error) {
    console.error('Ошибка push-уведомлений:', error);
  }
};
```

### Поддержка российских платежных систем

Для интеграции с российскими платежными системами:

```javascript
// Использование нативных платежных API
import { Plugins } from '@capacitor/core';
const { Device } = Plugins;

const checkPlatform = async () => {
  const info = await Device.getInfo();
  
  if (info.platform === 'android') {
    // Использование Google Pay для Android (или альтернатив для российского рынка)
    console.log('Android платформа');
  } else if (info.platform === 'ios') {
    // Использование Apple Pay для iOS
    console.log('iOS платформа');
  }
};
```

## Публикация приложений

Для публикации Capacitor приложений:

1. Настройте `capacitor.config.ts` с необходимыми параметрами
2. Создайте подписанные APK/IPA файлы
3. Подготовьте маркетинговые материалы
4. Опубликуйте в соответствующих магазинах

### Android

```bash
npx cap build android
# Затем откройте проект в Android Studio для создания релизной сборки
npx cap open android
```

### iOS

```bash
npx cap build ios
# Затем откройте проект в Xcode для создания релизной сборки
npx cap open ios
```

## Миграция с Cordova

Capacitor предоставляет инструменты для миграции с Cordova:

```bash
npm install @capacitor/cli @capacitor/core
npx cap init
npx cap migrate
```

## Заключение

Capacitor - современное решение для создания гибридных мобильных приложений с использованием веб-технологий. Его интеграция с Vue.js позволяет разработчикам создавать высококачественные мобильные приложения с доступом к нативным возможностям устройств.

См. также: [[Vue-Native]], [[Ionic-Vue]], [[Quasar]], [[Мобильные-паттерны]]
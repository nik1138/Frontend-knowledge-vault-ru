---
aliases: ["Установка Progressive Web App", "PWA Installation", "Install PWA"]
tags: ["#pwa", "#typescript", "#web", "#installation", "#user-experience", "#manifest"]
---

# Установка PWA

Установка Progressive Web App (PWA) позволяет пользователям добавлять веб-приложение на домашний экран устройства как нативное приложение. Это улучшает пользовательский опыт, обеспечивает автономную работу и позволяет использовать возможности устройства.

## Условия для установки PWA

Для того чтобы браузер распознал веб-приложение как PWA и предложил его установку, необходимо выполнить следующие требования:

1. **Web App Manifest** - правильно настроенный файл `manifest.json`
2. **Service Worker** - зарегистрированный и работающий сервис-воркер
3. **HTTPS** - приложение должно быть доступно по протоколу HTTPS (кроме localhost)
4. **Иконки** - наличие иконок различных размеров для разных устройств
5. **Определенные поля в манифесте** - `name`, `short_name`, `start_url`, `display`, `icons`

## Подготовка манифеста

Для начала необходимо создать файл `manifest.json` с обязательными полями:

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

Убедитесь, что манифест подключен в HTML-документе:

```html
<link rel="manifest" href="/manifest.json">
```

## Регистрация Service Worker

Для установки PWA необходим зарегистрированный Service Worker:

```typescript
// register-sw.ts
export async function registerServiceWorker(): Promise<void> {
  if ('serviceWorker' in navigator) {
    try {
      const registration = await navigator.serviceWorker.register('/sw.js');
      console.log('ServiceWorker зарегистрирован с scope:', registration.scope);
    } catch (error) {
      console.error('ServiceWorker регистрация не удалась:', error);
    }
  }
}

// Вызов функции при загрузке страницы
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    registerServiceWorker();
  });
}
```

## Определение возможности установки

Для определения, может ли пользователь установить PWA, можно использовать события и свойства браузера:

```typescript
// install-prompt.ts
class PWAInstallManager {
  private deferredPrompt: Event | null = null;
  private installButton: HTMLElement | null = null;
  
  constructor(installButtonId: string) {
    this.installButton = document.getElementById(installButtonId);
    this.init();
  }
  
  private init(): void {
    // Слушаем событие beforeinstallprompt
    window.addEventListener('beforeinstallprompt', (event) => {
      console.log('beforeinstallprompt событие сработало');
      
      // Предотвращаем стандартное всплывающее окно установки
      event.preventDefault();
      
      // Сохраняем событие для дальнейшего использования
      this.deferredPrompt = event;
      
      // Показываем пользовательскую кнопку установки
      if (this.installButton) {
        this.installButton.style.display = 'block';
        this.installButton.addEventListener('click', () => {
          this.promptInstall();
        });
      }
    });
    
    // Слушаем событие appinstalled
    window.addEventListener('appinstalled', () => {
      console.log('PWA успешно установлено');
      this.onAppInstalled();
    });
  }
  
  public async promptInstall(): Promise<void> {
    if (!this.deferredPrompt) {
      console.warn('Нет доступного deferredPrompt');
      return;
    }
    
    // Показываем диалог установки
    (this.deferredPrompt as any).prompt();
    
    // Ждем результата выбора пользователя
    const { outcome } = await (this.deferredPrompt as any).userChoice;
    
    console.log(`Результат установки: ${outcome}`);
    
    // Сбрасываем deferredPrompt
    this.deferredPrompt = null;
    
    // Скрываем кнопку установки
    if (this.installButton) {
      this.installButton.style.display = 'none';
    }
  }
  
  private onAppInstalled(): void {
    // Выполняем действия после установки PWA
    if (this.installButton) {
      this.installButton.style.display = 'none';
    }
    
    // Можем отправить аналитику или выполнить другие действия
    console.log('Логика после установки PWA');
  }
  
  public isInstallable(): boolean {
    return this.deferredPrompt !== null;
  }
  
  public isStandalone(): boolean {
    // Проверяем, запущено ли приложение в автономном режиме
    return window.matchMedia('(display-mode: standalone)').matches || 
           (navigator as any).standalone === true;
  }
}

// Инициализация менеджера установки
const installManager = new PWAInstallManager('install-button');
```

## Создание пользовательской кнопки установки

HTML-разметка для кнопки установки:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Мое PWA</title>
  <link rel="manifest" href="/manifest.json">
  <style>
    #install-button {
      position: fixed;
      bottom: 20px;
      right: 20px;
      background-color: #4CAF50;
      color: white;
      border: none;
      padding: 15px 20px;
      text-align: center;
      text-decoration: none;
      display: none; /* Сначала скрыта */
      font-size: 16px;
      border-radius: 50px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
      cursor: pointer;
      z-index: 1000;
    }
    
    #install-button:hover {
      background-color: #45a049;
    }
  </style>
</head>
<body>
  <h1>Добро пожаловать в мое PWA!</h1>
  <p>Это пример Progressive Web App с возможностью установки.</p>
  
  <button id="install-button">Установить PWA</button>
  
  <script src="/js/install-prompt.js"></script>
</body>
</html>
```

## Проверка статуса установки

Для проверки статуса установки PWA можно использовать различные методы:

```typescript
// installation-status.ts
class InstallationStatusChecker {
  /**
   * Проверяет, запущено ли приложение в автономном режиме (standalone)
   * Это указывает на то, что PWA было установлено
   */
  public static isStandalone(): boolean {
    return window.matchMedia('(display-mode: standalone)').matches || 
           (navigator as any).standalone === true;
  }
  
  /**
   * Проверяет, доступна ли установка PWA
   */
  public static isInstallable(): boolean {
    // Проверяем, есть ли событие beforeinstallprompt
    // Это событие доступно только когда PWA может быть установлено
    return typeof (window as any).deferredInstallPrompt !== 'undefined';
  }
  
  /**
   * Проверяет поддержку PWA в текущем браузере
   */
  public static isPWASupported(): boolean {
    return 'serviceWorker' in navigator && 'PushManager' in window;
  }
  
  /**
   * Получает информацию о статусе установки
   */
  public static getInstallationStatus(): {
    isSupported: boolean;
    isInstallable: boolean;
    isInstalled: boolean;
    displayMode: 'browser' | 'standalone' | 'minimal-ui' | 'fullscreen';
  } {
    const isSupported = this.isPWASupported();
    const isInstalled = this.isStandalone();
    
    // Определяем текущий режим отображения
    let displayMode: 'browser' | 'standalone' | 'minimal-ui' | 'fullscreen' = 'browser';
    
    if (window.matchMedia('(display-mode: standalone)').matches) {
      displayMode = 'standalone';
    } else if (window.matchMedia('(display-mode: minimal-ui)').matches) {
      displayMode = 'minimal-ui';
    } else if (window.matchMedia('(display-mode: fullscreen)').matches) {
      displayMode = 'fullscreen';
    }
    
    return {
      isSupported,
      isInstallable: isSupported && !isInstalled, // Установка возможна только если поддерживается и не установлено
      isInstalled,
      displayMode
    };
  }
}

// Пример использования
const status = InstallationStatusChecker.getInstallationStatus();
console.log('Статус установки PWA:', status);
```

## Автоматическая проверка установки

Для улучшения пользовательского опыта можно автоматически проверять статус установки и адаптировать интерфейс:

```typescript
// auto-install-checker.ts
class AutoInstallChecker {
  private checkInterval: number = 1000; // Проверять каждую секунду
  private maxChecks: number = 30; // Максимум 30 проверок (30 секунд)
  private currentCheck: number = 0;
  private onInstalledCallback: (() => void) | null = null;
  
  constructor(onInstalledCallback?: () => void) {
    this.onInstalledCallback = onInstalledCallback;
  }
  
  public startChecking(): void {
    const check = () => {
      if (InstallationStatusChecker.isStandalone()) {
        console.log('PWA было установлено');
        if (this.onInstalledCallback) {
          this.onInstalledCallback();
        }
        return; // Останавливаем проверку
      }
      
      this.currentCheck++;
      if (this.currentCheck < this.maxChecks) {
        setTimeout(check, this.checkInterval);
      } else {
        console.log('Время ожидания установки истекло');
      }
    };
    
    // Начинаем проверку через небольшую задержку
    setTimeout(check, 2000);
  }
  
  public static showInstallHint(): void {
    // Показываем подсказку пользователю о возможности установки
    if (!InstallationStatusChecker.isStandalone() && InstallationStatusChecker.isPWASupported()) {
      const hint = document.createElement('div');
      hint.id = 'pwa-install-hint';
      hint.innerHTML = `
        <div style="position: fixed; bottom: 0; left: 0; right: 0; background: #333; color: white; padding: 15px; text-align: center; z-index: 10000;">
          <p>Установите наше приложение для лучшего опыта! &nbsp;
            <button id="pwa-install-hint-btn" style="background: #4CAF50; color: white; border: none; padding: 8px 16px; border-radius: 4px; cursor: pointer;">Установить</button>
          </p>
        </div>
      `;
      
      document.body.appendChild(hint);
      
      // Добавляем обработчик для кнопки подсказки
      const hintButton = document.getElementById('pwa-install-hint-btn');
      if (hintButton) {
        hintButton.addEventListener('click', () => {
          // Пытаемся вызвать установку через событие beforeinstallprompt
          const event = (window as any).deferredInstallPrompt;
          if (event) {
            (event as any).prompt();
          } else {
            // Если событие недоступно, показываем инструкцию
            this.showInstallInstructions();
          }
        });
      }
    }
  }
  
  private static showInstallInstructions(): void {
    // Показываем инструкцию по установке для разных платформ
    const userAgent = navigator.userAgent.toLowerCase();
    let instructions = '';
    
    if (userAgent.includes('android')) {
      instructions = 'Для установки нажмите на меню браузера > Добавить на главный экран';
    } else if (userAgent.includes('iphone') || userAgent.includes('ipad')) {
      instructions = 'Для установки нажмите на кнопку "Поделиться" > Добавить на экран "Домой"';
    } else {
      instructions = 'Для установки используйте меню браузера > Установить приложение';
    }
    
    alert(`Как установить PWA:\n\n${instructions}`);
  }
}

// Использование
const autoChecker = new AutoInstallChecker(() => {
  console.log('PWA успешно установлено! Выполняем дополнительную логику...');
  // Выполняем действия после установки
});

autoChecker.startChecking();
AutoInstallChecker.showInstallHint();
```

## Платформо-специфические особенности

### Установка на Android (Chrome)

На Android Chrome автоматически предлагает установку PWA, если приложение соответствует критериям. Пользователь увидит баннер с предложением установки.

### Установка на iOS (Safari)

На iOS установка PWA осуществляется через меню "Поделиться" > "На экран 'Домой'". Для лучшей видимости рекекомендуется добавить инструкцию для пользователей iOS.

```typescript
// ios-install-helper.ts
class IOSInstallHelper {
  public static isIOS(): boolean {
    return /iPad|iPhone|iPod/.test(navigator.userAgent);
  }
  
  public static showIOSInstallGuide(): void {
    if (!this.isIOS()) {
      return;
    }
    
    const guide = document.createElement('div');
    guide.id = 'ios-install-guide';
    guide.innerHTML = `
      <div style="position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.7); z-index: 10001; display: flex; align-items: center; justify-content: center;">
        <div style="background: white; padding: 20px; border-radius: 10px; max-width: 90%; text-align: center;">
          <h3>Установка PWA на iOS</h3>
          <p>Чтобы установить приложение:</p>
          <ol style="text-align: left; margin: 15px 0;">
            <li>Нажмите на значок "Поделиться" в Safari</li>
            <li>Выберите "На экран 'Домой'"</li>
          </ol>
          <button id="close-ios-guide" style="background: #007AFF; color: white; border: none; padding: 10px 20px; border-radius: 5px; cursor: pointer;">Закрыть</button>
        </div>
      </div>
    `;
    
    document.body.appendChild(guide);
    
    const closeButton = document.getElementById('close-ios-guide');
    if (closeButton) {
      closeButton.addEventListener('click', () => {
        document.body.removeChild(guide);
      });
    }
  }
}

// Показываем руководство по установке для iOS-пользователей
if (IOSInstallHelper.isIOS()) {
  // Показываем руководство через несколько секунд после загрузки
  setTimeout(() => {
    if (!InstallationStatusChecker.isStandalone()) {
      IOSInstallHelper.showIOSInstallGuide();
    }
  }, 5000);
}
```

## Аналитика установки

Для отслеживания установки PWA можно использовать веб-аналитику:

```typescript
// install-analytics.ts
class InstallAnalytics {
  public static trackInstallAttempt(): void {
    // Отправляем событие в Google Analytics или другую систему аналитики
    if (typeof gtag !== 'undefined') {
      gtag('event', 'pwa_install_attempt', {
        event_category: 'PWA',
        event_label: 'Install Attempt'
      });
    }
    
    console.log('Отслеживание попытки установки PWA');
  }
  
  public static trackInstallSuccess(): void {
    if (typeof gtag !== 'undefined') {
      gtag('event', 'pwa_install_success', {
        event_category: 'PWA',
        event_label: 'Install Success'
      });
    }
    
    console.log('Отслеживание успешной установки PWA');
  }
  
  public static trackInstallDismissed(): void {
    if (typeof gtag !== 'undefined') {
      gtag('event', 'pwa_install_dismissed', {
        event_category: 'PWA',
        event_label: 'Install Dismissed'
      });
    }
    
    console.log('Отслеживание отклонения установки PWA');
  }
}

// Интеграция с менеджером установки
class EnhancedPWAInstallManager extends PWAInstallManager {
  protected onAppInstalled(): void {
    super.onAppInstalled();
    InstallAnalytics.trackInstallSuccess();
  }
  
  public async promptInstall(): Promise<void> {
    InstallAnalytics.trackInstallAttempt();
    try {
      await super.promptInstall();
    } catch (error) {
      InstallAnalytics.trackInstallDismissed();
      throw error;
    }
  }
}
```

## Лучшие практики

### 1. Правильное время для предложения установки

Не предлагайте установку сразу при посещении сайта. Лучше подождать, пока пользователь проявит интерес к приложению:

```typescript
// delayed-install-prompt.ts
class DelayedInstallPrompt {
  private showAfterSeconds: number = 30; // Показывать через 30 секунд
  private minVisits: number = 2; // Минимум 2 посещения
  private currentVisits: number = 0;
  
  constructor() {
    this.init();
  }
  
  private init(): void {
    // Увеличиваем счетчик посещений
    this.currentVisits = parseInt(localStorage.getItem('pwa_visits') || '0', 10) + 1;
    localStorage.setItem('pwa_visits', this.currentVisits.toString());
    
    // Устанавливаем таймер для показа предложения
    setTimeout(() => {
      if (this.shouldShowPrompt()) {
        this.showInstallPrompt();
      }
    }, this.showAfterSeconds * 1000);
  }
  
  private shouldShowPrompt(): boolean {
    return this.currentVisits >= this.minVisits && 
           !localStorage.getItem('pwa_install_prompted');
  }
  
  private showInstallPrompt(): void {
    // Показываем пользовательскую подсказку о возможности установки
    localStorage.setItem('pwa_install_prompted', 'true');
    
    // Показываем кастомный баннер или модальное окно
    this.displayCustomInstallPrompt();
  }
  
  private displayCustomInstallPrompt(): void {
    // Реализация кастомного интерфейса предложения установки
    console.log('Показываем пользовательское предложение установки PWA');
  }
}
```

### 2. Обработка отказа от установки

```typescript
// install-decision-handler.ts
class InstallDecisionHandler {
  public static handleInstallDecision(outcome: string): void {
    switch (outcome) {
      case 'accepted':
        console.log('Пользователь согласился на установку');
        break;
      case 'dismissed':
        console.log('Пользователь отклонил установку');
        // Можно запланировать повторное предложение через некоторое время
        this.scheduleFuturePrompt();
        break;
      case 'cancelled':
        console.log('Установка была отменена');
        break;
      default:
        console.log('Неизвестный результат установки:', outcome);
    }
  }
  
  private static scheduleFuturePrompt(): void {
    // Планируем повторное предложение через неделю
    const nextPromptTime = Date.now() + (7 * 24 * 60 * 60 * 1000); // 7 дней
    localStorage.setItem('next_install_prompt', nextPromptTime.toString());
  }
}
```

## Полезные ресурсы

- [[Web-App-Manifest]] - для понимания настройки манифеста
- [[Service-Workers]] - для понимания работы сервис-воркеров
- [[Push-уведомления]] - для полной функциональности PWA
- [[Офлайн-работа]] - для автономной работы приложения
- [Google Developers - Make it installable](https://developers.google.com/web/fundamentals/app-install-banners)
- [MDN - Web App Manifest](https://developer.mozilla.org/ru/docs/Web/Manifest)
- [PWA Checklist](https://developers.google.com/web/progressive-web-apps/checklist)

## Заключение

Установка PWA - важная часть пользовательского опыта, которая позволяет превратить веб-приложение в полноценное приложение на устройстве пользователя. Правильная реализация с использованием TypeScript обеспечивает типобезопасность и улучшает поддерживаемость кода. Важно учитывать особенности разных платформ и предлагать установку в подходящее время для максимального эффекта.
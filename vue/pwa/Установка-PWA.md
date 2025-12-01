# Установка PWA на Vue.js

**Alias**: [[Установка веб-приложения]]

**Tags**: #vue #pwa #installation #user-engagement #ux

Установка PWA позволяет пользователям добавлять веб-приложение на главный экран устройства, обеспечивая native app-like experience без необходимости скачивания из app store.

## Условия для установки

- Приложение должно быть доступно по HTTPS
- Наличие валидного Web App Manifest
- Регистрация Service Worker
- Минимальное время взаимодействия с пользователем

## Добавление установочного баннера

```javascript
// Обработка события beforeinstallprompt
let deferredPrompt;

window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  deferredPrompt = e;
  // Показать пользовательский интерфейс для установки
});
```

## Российские особенности (2025)

- Совместимость с отечественными операционными системами
- Учет требований к установке приложений
- Работа с российскими маркетами приложений

## Связанные темы

- [[Web-App-Manifest]]
- [[Service-Workers]]

## Подробная реализация установки PWA

### Проверка возможности установки

Прежде чем показывать пользователю возможность установки PWA, необходимо проверить, выполнены ли все необходимые условия:

```javascript
// Проверка поддержки PWA
function isPWAInstallable() {
  // Проверяем, является ли приложение уже установленным
  if (window.matchMedia('(display-mode: standalone)').matches) {
    return false; // Уже установлено
  }
  
  // Проверяем поддержку событий установки
  if ('BeforeInstallPromptEvent' in window) {
    return true; // Поддерживается
  }
  
  return false; // Не поддерживается
}
```

### Обработка события beforeinstallprompt

```javascript
let deferredPrompt;

window.addEventListener('beforeinstallprompt', (e) => {
  // Предотвращаем стандартное всплывающее окно
  e.preventDefault();
  
  // Сохраняем событие для дальнейшего использования
  deferredPrompt = e;
  
  // Показываем пользовательский интерфейс для установки
  showInstallPromotion();
});

function showInstallPromotion() {
  // Показываем кнопку "Установить" или баннер
  const installButton = document.getElementById('install-button');
  if (installButton) {
    installButton.style.display = 'block';
  }
}
```

### Реализация кнопки установки

```vue
<template>
  <div>
    <!-- Основной контент приложения -->
    <main>
      <slot></slot>
    </main>
    
    <!-- Кнопка установки PWA -->
    <div v-if="showInstallButton" class="install-prompt">
      <div class="install-content">
        <h3>{{ appName }} доступно оффлайн!</h3>
        <p>Добавьте это приложение на главный экран для быстрого доступа и оффлайн-использования.</p>
        <div class="install-buttons">
          <button @click="cancelInstall" class="cancel-btn">Не сейчас</button>
          <button @click="installPWA" class="install-btn">Установить</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PWAInstaller',
  data() {
    return {
      showInstallButton: false,
      deferredPrompt: null,
      appName: 'Мое Vue PWA'
    };
  },
  mounted() {
    this.setupInstallPrompt();
  },
  methods: {
    setupInstallPrompt() {
      window.addEventListener('beforeinstallprompt', (e) => {
        // Предотвращаем стандартный баннер установки
        e.preventDefault();
        
        // Сохраняем событие для дальнейшего использования
        this.deferredPrompt = e;
        
        // Показываем пользовательский баннер установки
        this.showInstallButton = true;
      });
      
      // Также можно отслеживать, установлено ли приложение
      window.addEventListener('appinstalled', () => {
        console.log('PWA установлено успешно');
        this.showInstallButton = false;
        this.deferredPrompt = null;
      });
    },
    
    async installPWA() {
      if (!this.deferredPrompt) {
        return;
      }
      
      // Показываем приглашение к установке
      this.deferredPrompt.prompt();
      
      // Ждем результата
      const { outcome } = await this.deferredPrompt.userChoice;
      
      // Сбрасываем событие
      this.deferredPrompt = null;
      this.showInstallButton = false;
      
      if (outcome === 'accepted') {
        console.log('Пользователь согласился на установку PWA');
      } else {
        console.log('Пользователь отклонил установку PWA');
      }
    },
    
    cancelInstall() {
      this.showInstallButton = false;
      this.deferredPrompt = null;
    }
  }
}
</script>

<style scoped>
.install-prompt {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: white;
  border-top: 1px solid #e0e0e0;
  box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
  z-index: 10000;
  padding: 16px;
}

.install-content {
  max-width: 600px;
  margin: 0 auto;
}

.install-content h3 {
  margin: 0 0 8px 0;
  font-size: 16px;
  font-weight: 600;
}

.install-content p {
  margin: 0 0 16px 0;
  font-size: 14px;
  color: #666;
}

.install-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}

.cancel-btn, .install-btn {
  padding: 8px 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.install-btn {
  background-color: #42b549;
  color: white;
  border-color: #42b549;
}

.cancel-btn {
  background-color: transparent;
  color: #666;
}
</style>
```

### Определение статуса установки

```javascript
// Проверка, запущено ли приложение в режиме PWA
function isStandaloneMode() {
  return window.matchMedia('(display-mode: standalone)').matches || 
         window.navigator.standalone === true;
}

// Использование статуса установки в компоненте
export default {
  data() {
    return {
      isInstalled: false
    };
  },
  mounted() {
    // Проверяем статус установки при загрузке
    this.checkInstallationStatus();
    
    // Следим за изменениями статуса
    window.addEventListener('appinstalled', () => {
      this.isInstalled = true;
    });
  },
  methods: {
    checkInstallationStatus() {
      this.isInstalled = isStandaloneMode();
    }
  }
}
```

### Улучшение UX для установки

#### 1. Правильное время для предложения установки

```javascript
// Показываем предложение установки после определенных действий пользователя
let userInteractionCount = 0;

function trackUserInteraction() {
  userInteractionCount++;
  
  // Показываем предложение после 3-х взаимодействий
  if (userInteractionCount === 3 && deferredPrompt) {
    showInstallPromotion();
  }
}

// Отслеживаем клики и другие взаимодействия
document.addEventListener('click', trackUserInteraction);
document.addEventListener('scroll', trackUserInteraction);
```

#### 2. Обработка отказа от установки

```javascript
// Обработка отказа пользователя от установки
window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  deferredPrompt = e;
  
  // Показываем кнопку установки
  showInstallPromotion();
  
  // Сохраняем факт показа предложения
  localStorage.setItem('installPromptShown', 'true');
});

// Проверяем, показывалось ли уже предложение
if (!localStorage.getItem('installPromptShown')) {
  // Показываем предложение
}
```

### Тестирование установки PWA

Для тестирования установки PWA можно использовать:

1. Chrome DevTools - вкладка Application - раздел "Installability"
2. Lighthouse - проверка аудита PWA
3. Тестирование на реальных устройствах

### Российские особенности установки PWA

#### Совместимость с отечественными ОС

- Учет особенностей установки в отечественных операционных системах (например, Аврора ОС, РОСА, Astra Linux)
- Проверка работы PWA в браузерах, разработанных в России (Яндекс.Браузер, Спутник)

#### Требования к установке

- Соответствие требованиям российского законодательства
- Учет особенностей магазинов приложений, используемых в России
- Возможность работы без Google Play Services (для устройств с отечественными ОС)

#### Работа с российскими маркетами

- Поддержка альтернативных способов установки приложений
- Интеграция с отечественными маркетами приложений
- Учет требований к идентификации приложений

## Лучшие практики установки PWA

- Обеспечьте качественный Web App Manifest с правильными иконками и названиями
- Проверьте, что Service Worker корректно зарегистрирован и работает
- Предлагайте установку в подходящий момент, не сразу при посещении
- Обеспечьте понятный пользовательский интерфейс для установки
- Обрабатывайте все возможные сценарии установки (успех, отказ, ошибка)
- Проверяйте установку на различных устройствах и браузерах
# Push-уведомления в PWA на Vue.js

**Alias**: [[Пуш-нотификации]]

**Tags**: #vue #pwa #push-notifications #messaging #engagement

Push-уведомления позволяют отправлять сообщения пользователям даже тогда, когда приложение не активно, что способствует повышению вовлеченности.

## Техническая реализация

Для реализации push-уведомлений в PWA используются:
- Service Workers
- Push API
- Notification API
- Backend-сервер для отправки уведомлений

## Настройка в Vue.js

```javascript
// Регистрация Service Worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(registration => {
      return registration.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
      });
    });
}
```

## Российские особенности (2025)

- Совместимость с отечественными мессенджерами
- Учет требований законодательства о персональных данных
- Работа с российскими push-сервисами

## Связанные темы

- [[Service-Workers]]
- [[Оффлайн-функционал]]

## Подробная реализация push-уведомлений

### Подготовка к реализации

Для работы с push-уведомлениями в PWA необходимо:

1. Иметь действительный SSL-сертификат (HTTPS)
2. Зарегистрировать Service Worker
3. Получить VAPID-ключи (Voluntary Application Server Identification)
4. Настроить backend-сервер для отправки уведомлений

### Генерация VAPID-ключей

VAPID (Voluntary Application Server Identification) - это стандарт, который позволяет серверу идентифицировать себя перед push-сервисом. Для генерации ключей можно использовать библиотеку `web-push`:

```bash
npm install web-push
```

```javascript
const webpush = require('web-push');

// Генерация VAPID-ключей
const vapidKeys = webpush.generateVAPIDKeys();

console.log('Public Key:', vapidKeys.publicKey);
console.log('Private Key:', vapidKeys.privateKey);
```

### Подписка пользователя на уведомления

```javascript
// Файл main.js или компонент, где происходит подписка
async function subscribeUserToPush() {
  try {
    const registration = await navigator.serviceWorker.ready;
    
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true,
      applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
    });
    
    // Отправка подписки на сервер
    await sendSubscriptionToServer(subscription);
    
    console.log('Пользователь подписан на push-уведомления', subscription);
  } catch (error) {
    console.error('Ошибка при подписке на push-уведомления:', error);
  }
}

function urlBase64ToUint8Array(base64String) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');
  
  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);
  
  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}

async function sendSubscriptionToServer(subscription) {
  const response = await fetch('/api/subscribe', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(subscription)
  });
  
  if (!response.ok) {
    throw new Error('Не удалось отправить подписку на сервер');
  }
}
```

### Обработка push-уведомлений в Service Worker

```javascript
// Файл sw.js
self.addEventListener('push', function(event) {
  let payload = {};

  if (event.data) {
    payload = event.data.json();
  }

  const options = {
    body: payload.body || 'Новое сообщение',
    icon: payload.icon || '/img/icons/android-chrome-192x192.png',
    badge: payload.badge || '/img/icons/badge-72x72.png',
    tag: payload.tag || 'default-tag',
    data: payload.data || {},
    actions: payload.actions || []
  };

  event.waitUntil(
    self.registration.showNotification(payload.title || 'Уведомление', options)
  );
});

// Обработка клика по уведомлению
self.addEventListener('notificationclick', function(event) {
  event.notification.close();

  if (event.action === 'open_url' && event.notification.data.url) {
    event.waitUntil(
      clients.openWindow(event.notification.data.url)
    );
  } else if (event.notification.data.url) {
    event.waitUntil(
      clients.openWindow(event.notification.data.url)
    );
  }
});
```

### Реализация на стороне Vue-компонента

```vue
<template>
  <div>
    <button @click="requestNotificationPermission" :disabled="notificationPermission !== 'default'">
      {{ notificationPermission === 'default' ? 'Разрешить уведомления' : 'Уведомления разрешены' }}
    </button>
    
    <div v-if="subscriptionStatus" class="subscription-status">
      {{ subscriptionStatus }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'PushNotifications',
  data() {
    return {
      notificationPermission: Notification.permission,
      subscriptionStatus: null
    };
  },
  mounted() {
    this.checkServiceWorkerAndSubscribe();
  },
  methods: {
    async requestNotificationPermission() {
      try {
        const permission = await Notification.requestPermission();
        this.notificationPermission = permission;
        
        if (permission === 'granted') {
          await this.subscribeToPushNotifications();
        }
      } catch (error) {
        console.error('Ошибка при запросе разрешения на уведомления:', error);
      }
    },
    
    async checkServiceWorkerAndSubscribe() {
      if ('serviceWorker' in navigator && 'PushManager' in window) {
        try {
          const registration = await navigator.serviceWorker.register('/sw.js');
          const subscription = await registration.pushManager.getSubscription();
          
          if (subscription) {
            this.subscriptionStatus = 'Уже подписаны на уведомления';
          }
        } catch (error) {
          console.error('Ошибка при регистрации Service Worker:', error);
        }
      } else {
        this.subscriptionStatus = 'Push-уведомления не поддерживаются в этом браузере';
      }
    },
    
    async subscribeToPushNotifications() {
      try {
        const registration = await navigator.serviceWorker.ready;
        
        const subscription = await registration.pushManager.subscribe({
          userVisibleOnly: true,
          applicationServerKey: this.urlBase64ToUint8Array(process.env.VUE_APP_VAPID_PUBLIC_KEY)
        });
        
        // Отправка подписки на сервер
        await this.sendSubscriptionToServer(subscription);
        this.subscriptionStatus = 'Успешно подписаны на уведомления';
        
        console.log('Подписка на push-уведомления:', subscription);
      } catch (error) {
        console.error('Ошибка при подписке на push-уведомления:', error);
        this.subscriptionStatus = 'Ошибка при подписке на уведомления';
      }
    },
    
    urlBase64ToUint8Array(base64String) {
      const padding = '='.repeat((4 - base64String.length % 4) % 4);
      const base64 = (base64String + padding)
        .replace(/\-/g, '+')
        .replace(/_/g, '/');
      
      const rawData = window.atob(base64);
      const outputArray = new Uint8Array(rawData.length);
      
      for (let i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
      }
      return outputArray;
    },
    
    async sendSubscriptionToServer(subscription) {
      try {
        const response = await fetch('/api/subscribe', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(subscription)
        });
        
        if (!response.ok) {
          throw new Error('Не удалось отправить подписку на сервер');
        }
      } catch (error) {
        console.error('Ошибка при отправке подписки на сервер:', error);
        throw error;
      }
    }
  }
}
</script>

<style scoped>
.subscription-status {
  margin-top: 10px;
  padding: 8px;
  border-radius: 4px;
  background-color: #f0f0f0;
}
</style>
```

### Отправка уведомлений с backend-сервера

Пример на Node.js с использованием `web-push`:

```javascript
const express = require('express');
const webpush = require('web-push');
const app = express();

// Настройка VAPID-ключей
webpush.setVapidDetails(
  'mailto:your-email@example.com', // Ваш email
  process.env.VAPID_PUBLIC_KEY,    // Публичный ключ
  process.env.VAPID_PRIVATE_KEY    // Приватный ключ
);

app.use(express.json());

// Эндпоинт для получения подписки от клиента
app.post('/api/subscribe', (req, res) => {
  // Сохраняем подписку в базе данных
  saveSubscriptionToDatabase(req.body);
  res.status(201).json({ message: 'Подписка сохранена' });
});

// Функция для отправки уведомления
function sendNotification(subscription, payload) {
  webpush.sendNotification(
    subscription,
    JSON.stringify(payload)
  ).catch(err => {
    console.error('Ошибка при отправке уведомления:', err);
    // Обработка ошибки, возможно удаление невалидной подписки
  });
}

// Пример отправки уведомления
app.post('/api/send-notification', async (req, res) => {
  const subscriptions = await getSubscriptionsFromDatabase(); // Получаем все подписки
  
  const payload = {
    title: 'Новое уведомление',
    body: 'Сообщение уведомления',
    icon: '/img/icons/android-chrome-192x192.png',
    data: {
      url: '/notification-url'
    }
  };
  
  // Отправляем уведомление всем подписчикам
  subscriptions.forEach(subscription => {
    sendNotification(subscription, payload);
  });
  
  res.status(200).json({ message: 'Уведомления отправлены' });
});
```

## Российские особенности и требования

### Соответствие законодательству

- Соблюдение требований ФЗ-152 "О персональных данных"
- Получение согласия пользователя на получение уведомлений
- Предоставление возможности отписки от уведомлений

### Совместимость с отечественными платформами

- Учет особенностей работы push-уведомлений в отечественных браузерах
- Альтернативные способы доставки уведомлений при блокировке Google Firebase Cloud Messaging

## Лучшие практики

- Запрашивать разрешение на уведомления в подходящий момент
- Обеспечивать полезный и релевантный контент в уведомлениях
- Учитывать время суток при отправке уведомлений
- Предоставлять настройки уведомлений для пользователей
- Обрабатывать отказы от подписки и ошибки доставки
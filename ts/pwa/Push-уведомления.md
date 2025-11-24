---
aliases: ["Push-нотификации", "Веб-уведомления", "Web Push Notifications"]
tags: ["#pwa", "#typescript", "#web", "#notifications", "#push", "#service-workers"]
---

# Push-уведомления

Push-уведомления - это мощная функция Progressive Web Apps, позволяющая отправлять сообщения пользователям даже тогда, когда веб-приложение не открыто. Это улучшает вовлеченность пользователей и позволяет информировать их о важных событиях в реальном времени.

## Основные компоненты Push-уведомлений

Для реализации Push-уведомлений в PWA необходимы следующие компоненты:

1. **Service Worker** - получает и отображает уведомления
2. **Push-сервер** - отправляет уведомления через Push-сервис (например, Firebase Cloud Messaging)
3. **Web Push API** - интерфейс для подписки на уведомления
4. **Notifications API** - интерфейс для отображения уведомлений

## Подписка на Push-уведомления

### Проверка поддержки

Перед началом работы с Push-уведомлениями необходимо проверить поддержку в браузере:

```typescript
// notification-manager.ts
class NotificationManager {
  private isSupported: boolean;
  
  constructor() {
    this.isSupported = this.checkSupport();
  }
  
  private checkSupport(): boolean {
    return 'serviceWorker' in navigator && 
           'PushManager' in window && 
           'Notification' in window;
  }
  
  public canShowNotifications(): boolean {
    return this.isSupported && 
           Notification.permission === 'granted';
  }
  
  public canSubscribeToPush(): boolean {
    return this.canShowNotifications() && 
           'PushManager' in window;
  }
}
```

### Запрос разрешения на показ уведомлений

```typescript
// notification-manager.ts (продолжение)
class NotificationManager {
  // ... предыдущий код
  
  public async requestPermission(): Promise<NotificationPermission> {
    if (!this.isSupported) {
      throw new Error('Push-уведомления не поддерживаются в этом браузере');
    }
    
    const permission = await Notification.requestPermission();
    console.log('Разрешение на уведомления:', permission);
    return permission;
  }
  
  public async subscribeToPush(): Promise<PushSubscription | null> {
    if (!this.canSubscribeToPush()) {
      console.warn('Подписка на Push-уведомления невозможна');
      return null;
    }
    
    try {
      // Получаем зарегистрированный Service Worker
      const registration = await navigator.serviceWorker.ready;
      
      // Подписываемся на Push-уведомления
      const subscription = await registration.pushManager.subscribe({
        userVisibleOnly: true, // Обязательный параметр для Web Push
        applicationServerKey: this.urlB64ToUint8Array(VAPID_PUBLIC_KEY)
      });
      
      console.log('Успешно подписались на Push-уведомления:', subscription);
      
      // Отправляем подписку на сервер
      await this.sendSubscriptionToServer(subscription);
      
      return subscription;
    } catch (error) {
      console.error('Ошибка подписки на Push-уведомления:', error);
      return null;
    }
  }
  
  private urlB64ToUint8Array(base64String: string): Uint8Array {
    const padding = '='.repeat((4 - (base64String.length % 4)) % 4);
    const base64 = (base64String + padding)
      .replace(/\-/g, '+')
      .replace(/_/g, '/');
    
    const rawData = atob(base64);
    const outputArray = new Uint8Array(rawData.length);
    
    for (let i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
  }
  
  private async sendSubscriptionToServer(subscription: PushSubscription): Promise<void> {
    try {
      const response = await fetch('/api/subscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(subscription)
      });
      
      if (!response.ok) {
        throw new Error('Ошибка отправки подписки на сервер');
      }
      
      console.log('Подписка успешно отправлена на сервер');
    } catch (error) {
      console.error('Ошибка отправки подписки:', error);
    }
  }
}
```

## Service Worker для Push-уведомлений

Service Worker должен обрабатывать события `push` и `notificationclick`:

```typescript
// sw.ts
declare const self: ServiceWorkerGlobalScope;

// Обработка получения Push-уведомления
self.addEventListener('push', (event: PushEvent) => {
  let payload: any = {};
  
  // Получаем данные из уведомления (если есть)
  if (event.data) {
    try {
      payload = event.data.json();
    } catch (error) {
      console.error('Ошибка парсинга данных уведомления:', error);
      payload = { title: 'Новое уведомление', body: 'У вас новое сообщение' };
    }
  } else {
    payload = { title: 'Новое уведомление', body: 'У вас новое сообщение' };
  }
  
  const title = payload.title || 'Новое уведомление';
  const options = {
    body: payload.body || 'У вас новое сообщение',
    icon: payload.icon || '/icons/icon-192x192.png',
    badge: payload.badge || '/icons/badge-72x72.png',
    tag: payload.tag || 'default',
    data: payload.data || {},
    actions: payload.actions || [],
    vibrate: payload.vibrate || [200, 100, 200],
    renotify: payload.renotify || false,
    requireInteraction: payload.requireInteraction || false,
    silent: payload.silent || false
  };
  
  // Отображаем уведомление
  event.waitUntil(
    self.registration.showNotification(title, options)
  );
});

// Обработка клика по уведомлению
self.addEventListener('notificationclick', (event: NotificationEvent) => {
  event.notification.close(); // Закрываем уведомление
  
  // Определяем действие при клике
  const action = event.action;
  const notificationData = event.notification.data;
  
  if (action === 'close') {
    // Пользователь закрыл уведомление
    console.log('Пользователь закрыл уведомление');
  } else {
    // Открываем вкладку с определенным URL
    const urlToOpen = notificationData.url || self.location.origin;
    
    event.waitUntil(
      self.clients.openWindow(urlToOpen)
    );
  }
});

// Обработка исчезновения уведомления
self.addEventListener('notificationclose', (event: NotificationEvent) => {
  console.log('Уведомление закрыто пользователем');
  // Здесь можно выполнить дополнительную логику при закрытии уведомления
});
```

## Отправка Push-уведомлений с сервера

Для отправки Push-уведомлений с сервера можно использовать библиотеку `web-push` (Node.js):

```typescript
// server/push-service.ts
import * as webpush from 'web-push';

// Настройка VAPID-ключей (замените на ваши реальные ключи)
const vapidKeys = {
  publicKey: process.env.VAPID_PUBLIC_KEY || 'your_public_vapid_key',
  privateKey: process.env.VAPID_PRIVATE_KEY || 'your_private_vapid_key'
};

webpush.setVapidDetails(
  'mailto:your-email@example.com', // Ваш email для идентификации
  vapidKeys.publicKey,
  vapidKeys.privateKey
);

interface PushSubscription {
  endpoint: string;
  keys: {
    p256dh: string;
    auth: string;
  };
}

class PushService {
  public async sendNotification(
    subscription: PushSubscription, 
    payload: any
  ): Promise<boolean> {
    try {
      const response = await webpush.sendNotification(
        subscription,
        JSON.stringify(payload),
        {
          TTL: 60 * 60 * 24 // Время жизни сообщения - 24 часа
        }
      );
      
      console.log('Уведомление успешно отправлено:', response.statusCode);
      return true;
    } catch (error) {
      console.error('Ошибка отправки уведомления:', error);
      return false;
    }
  }
  
  public async sendBulkNotifications(
    subscriptions: PushSubscription[], 
    payload: any
  ): Promise<number> {
    const results = await Promise.all(
      subscriptions.map(sub => this.sendNotification(sub, payload))
    );
    
    // Возвращаем количество успешно отправленных уведомлений
    return results.filter(result => result).length;
  }
}

// Пример использования
const pushService = new PushService();

// Пример отправки уведомления
const exampleSubscription: PushSubscription = {
  endpoint: 'https://fcm.googleapis.com/fcm/send/...',
  keys: {
    p256dh: '...',
    auth: '...'
  }
};

const notificationPayload = {
  title: 'Новое сообщение',
  body: 'У вас новое сообщение от пользователя John',
  icon: '/icons/icon-192x192.png',
  data: {
    url: '/messages/123',
    userId: '123'
  },
  actions: [
    { action: 'open', title: 'Открыть' },
    { action: 'close', title: 'Закрыть' }
  ]
};

// pushService.sendNotification(exampleSubscription, notificationPayload);
```

## Управление Push-уведомлениями в приложении

### Отписка от Push-уведомлений

```typescript
// notification-manager.ts (продолжение)
class NotificationManager {
  // ... предыдущий код
  
  public async unsubscribeFromPush(): Promise<boolean> {
    try {
      const registration = await navigator.serviceWorker.ready;
      const subscription = await registration.pushManager.getSubscription();
      
      if (subscription) {
        const success = await subscription.unsubscribe();
        
        if (success) {
          console.log('Успешно отписались от Push-уведомлений');
          
          // Удаляем подписку с сервера
          await this.removeSubscriptionFromServer(subscription.endpoint);
          return true;
        }
      }
      
      return false;
    } catch (error) {
      console.error('Ошибка отписки от Push-уведомлений:', error);
      return false;
    }
  }
  
  private async removeSubscriptionFromServer(endpoint: string): Promise<void> {
    try {
      const response = await fetch('/api/unsubscribe', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ endpoint })
      });
      
      if (!response.ok) {
        throw new Error('Ошибка удаления подписки с сервера');
      }
      
      console.log('Подписка успешно удалена с сервера');
    } catch (error) {
      console.error('Ошибка удаления подписки:', error);
    }
  }
}
```

### Проверка статуса подписки

```typescript
// notification-manager.ts (продолжение)
class NotificationManager {
  // ... предыдущий код
  
  public async getSubscriptionStatus(): Promise<{
    isSubscribed: boolean;
    permission: NotificationPermission;
  }> {
    const permission = Notification.permission;
    
    if (permission !== 'granted') {
      return { isSubscribed: false, permission };
    }
    
    try {
      const registration = await navigator.serviceWorker.ready;
      const subscription = await registration.pushManager.getSubscription();
      
      return {
        isSubscribed: !!subscription,
        permission
      };
    } catch (error) {
      console.error('Ошибка проверки статуса подписки:', error);
      return { isSubscribed: false, permission };
    }
  }
}
```

## Практические примеры использования

### Уведомления о новых сообщениях

```typescript
// message-notifications.ts
class MessageNotificationService {
  private notificationManager: NotificationManager;
  
  constructor(notificationManager: NotificationManager) {
    this.notificationManager = notificationManager;
  }
  
  public async showNewMessageNotification(
    sender: string, 
    message: string, 
    conversationId: string
  ): Promise<void> {
    if (!this.notificationManager.canShowNotifications()) {
      return;
    }
    
    const title = `Новое сообщение от ${sender}`;
    const options = {
      body: message,
      icon: '/icons/message-icon.png',
      tag: `message-${conversationId}`,
      data: {
        url: `/messages/${conversationId}`,
        conversationId
      },
      actions: [
        { action: 'reply', title: 'Ответить' },
        { action: 'view', title: 'Просмотреть' }
      ]
    };
    
    // Отображаем уведомление
    new Notification(title, options);
  }
}
```

### Уведомления о важных событиях

```typescript
// event-notifications.ts
interface EventNotification {
  eventId: string;
  title: string;
  body: string;
  type: 'reminder' | 'alert' | 'info' | 'warning';
  data?: any;
}

class EventNotificationService {
  public showEventNotification(notification: EventNotification): void {
    if (!('Notification' in window) || Notification.permission !== 'granted') {
      return;
    }
    
    const options = {
      body: notification.body,
      icon: this.getIconByType(notification.type),
      tag: `event-${notification.eventId}`,
      data: notification.data,
      vibrate: this.getVibrationPattern(notification.type)
    };
    
    new Notification(notification.title, options);
  }
  
  private getIconByType(type: string): string {
    const icons: Record<string, string> = {
      'reminder': '/icons/reminder-icon.png',
      'alert': '/icons/alert-icon.png',
      'info': '/icons/info-icon.png',
      'warning': '/icons/warning-icon.png'
    };
    
    return icons[type] || icons['info'];
  }
  
  private getVibrationPattern(type: string): number[] {
    const patterns: Record<string, number[]> = {
      'reminder': [200, 100, 200],
      'alert': [100, 50, 100, 50, 100],
      'info': [],
      'warning': [150, 100, 150]
    };
    
    return patterns[type] || [];
  }
}
```

## Обработка различных типов уведомлений

### Уведомления с действиями

```typescript
// sw.ts (дополнение)
self.addEventListener('notificationclick', (event: NotificationEvent) => {
  event.notification.close();
  
  const action = event.action;
  const notificationData = event.notification.data;
  
  switch (action) {
    case 'reply':
      // Открываем окно для ответа
      event.waitUntil(
        self.clients.openWindow(`/messages/${notificationData.conversationId}?reply=true`)
      );
      break;
      
    case 'view':
      // Открываем соответствующую страницу
      event.waitUntil(
        self.clients.openWindow(notificationData.url)
      );
      break;
      
    case 'close':
      // Просто закрываем уведомление
      break;
      
    default:
      // Действие по умолчанию
      event.waitUntil(
        self.clients.openWindow(notificationData.url || self.location.origin)
      );
      break;
  }
});
```

## Полезные ресурсы

- [[Service-Workers]] - для понимания основ работы с уведомлениями
- [[Web-App-Manifest]] - для полной настройки PWA
- [[Офлайн-работа]] - для понимания полного цикла PWA
- [[Установка-PWA]] - для понимания процесса установки
- [Web Push Protocol](https://tools.ietf.org/html/rfc8030)
- [Google Developers - Web Push Notifications](https://developers.google.com/web/fundamentals/push-notifications)
- [MDN - Push API](https://developer.mozilla.org/ru/docs/Web/API/Push_API)

## Заключение

Push-уведомления значительно улучшают пользовательский опыт в Progressive Web Apps, позволяя информировать пользователей о важных событиях даже при закрытом приложении. Правильная реализация с использованием TypeScript обеспечивает типобезопасность и улучшает поддерживаемость кода. Важно учитывать приватность пользователей и предоставлять им контроль над уведомлениями.
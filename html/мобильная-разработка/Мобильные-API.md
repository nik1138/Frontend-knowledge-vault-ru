---
aliases: ["Мобильные API", "Mobile APIs", "Сенсоры и API"]
tags: [javascript, mobile-development, web-api]
---

# Мобильные API в веб-разработке

## Обзор мобильных API

Мобильные API позволяют веб-приложениям использовать специфические функции мобильных устройств, такие как геолокация, камера, акселерометр и другие сенсоры. В 2025 году эти API становятся все более важными для создания полноценных мобильных веб-приложений.

## Геолокация (Geolocation API)

Позволяет получить текущее местоположение пользователя:

```javascript
if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(
        (position) => {
            const latitude = position.coords.latitude;
            const longitude = position.coords.longitude;
            const accuracy = position.coords.accuracy;
            
            console.log(`Широта: ${latitude}, Долгота: ${longitude}, Точность: ${accuracy} м`);
        },
        (error) => {
            console.error('Ошибка получения геолокации:', error.message);
        },
        {
            enableHighAccuracy: true,
            timeout: 5000,
            maximumAge: 600000 // 10 минут
        }
    );
} else {
    console.log('Геолокация не поддерживается');
}
```

### Наблюдение за изменением местоположения

```javascript
const watchId = navigator.geolocation.watchPosition(
    (position) => {
        updateLocation(position.coords.latitude, position.coords.longitude);
    },
    (error) => {
        console.error('Ошибка отслеживания местоположения:', error);
    },
    {
        enableHighAccuracy: true,
        maximumAge: 30000,
        timeout: 27000
    }
);

// Остановка отслеживания
// navigator.geolocation.clearWatch(watchId);
```

## Доступ к камере и микрофону (MediaDevices API)

### Захват видео с камеры

```javascript
async function startCamera() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({
            video: { 
                facingMode: 'user', // передняя камера
                width: { ideal: 1280 },
                height: { ideal: 720 }
            },
            audio: true
        });
        
        const video = document.getElementById('video-element');
        video.srcObject = stream;
        
        return stream;
    } catch (error) {
        console.error('Ошибка доступа к камере:', error);
    }
}

// Захват изображения
function capturePhoto() {
    const video = document.getElementById('video-element');
    const canvas = document.createElement('canvas');
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;
    
    const ctx = canvas.getContext('2d');
    ctx.drawImage(video, 0, 0);
    
    // Получение изображения в формате data URL
    const imageData = canvas.toDataURL('image/png');
    return imageData;
}
```

## Акселерометр и гироскоп

### DeviceOrientation Event

```javascript
// Проверка поддержки
if (typeof DeviceOrientationEvent.requestPermission === 'function') {
    // iOS 13+ требует явного разрешения
    DeviceOrientationEvent.requestPermission()
        .then((permissionState) => {
            if (permissionState === 'granted') {
                window.addEventListener('deviceorientation', handleOrientation);
            }
        })
        .catch(console.error);
} else {
    // Android и старые iOS
    window.addEventListener('deviceorientation', handleOrientation);
}

function handleOrientation(event) {
    const alpha = event.alpha; // вращение вокруг оси Z
    const beta = event.beta;   // вращение вокруг оси X
    const gamma = event.gamma; // вращение вокруг оси Y
    
    console.log(`Альфа: ${alpha}, Бета: ${beta}, Гамма: ${gamma}`);
    
    // Пример использования: поворот элемента
    const element = document.getElementById('rotatable');
    element.style.transform = `rotateX(${beta}deg) rotateY(${gamma}deg) rotateZ(${alpha}deg)`;
}
```

### DeviceMotion Event

```javascript
window.addEventListener('devicemotion', (event) => {
    const acceleration = event.acceleration;
    const accelerationIncludingGravity = event.accelerationIncludingGravity;
    const rotationRate = event.rotationRate;
    const interval = event.interval;
    
    console.log({
        acceleration: acceleration,
        accelerationIncludingGravity: accelerationIncludingGravity,
        rotationRate: rotationRate,
        interval: interval
    });
    
    // Пример: детекция встряхивания
    if (acceleration.x > 15 || acceleration.y > 15 || acceleration.z > 15) {
        console.log('Обнаружено встряхивание устройства!');
    }
});
```

## Уведомления (Notifications API)

```javascript
// Проверка поддержки и запрос разрешения
if ('Notification' in window) {
    Notification.requestPermission().then((permission) => {
        if (permission === 'granted') {
            // Отправка уведомления
            const notification = new Notification('Заголовок уведомления', {
                body: 'Текст уведомления',
                icon: '/path/to/icon.png',
                badge: '/path/to/badge.png'
            });
            
            // Обработка событий уведомления
            notification.onclick = () => {
                console.log('Пользователь кликнул по уведомлению');
                window.focus();
            };
            
            notification.onerror = () => {
                console.log('Ошибка отображения уведомления');
            };
        }
    });
}
```

## Push-уведомления

```javascript
// Регистрация сервис-воркера
if ('serviceWorker' in navigator && 'PushManager' in window) {
    navigator.serviceWorker.register('/sw.js')
        .then((registration) => {
            return registration.pushManager.subscribe({
                userVisibleOnly: true,
                applicationServerKey: urlB64ToUint8Array('YOUR_PUBLIC_VAPID_KEY')
            });
        })
        .then((subscription) => {
            console.log('Подписка на push-уведомления:', subscription);
            // Отправка подписки на сервер
            sendSubscriptionToServer(subscription);
        })
        .catch((error) => {
            console.error('Ошибка подписки на push:', error);
        });
}

// Вспомогательная функция для VAPID ключа
function urlB64ToUint8Array(base64String) {
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
```

## Доступ к контактам

```javascript
// Contact Picker API (экспериментальный)
if ('contacts' in navigator && 'ContactsManager' in window) {
    async function selectContacts() {
        const props = ['name', 'email', 'tel'];
        const options = { multiple: true };
        
        try {
            const contacts = await navigator.contacts.select(props, options);
            console.log('Выбранные контакты:', contacts);
        } catch (error) {
            console.error('Ошибка доступа к контактам:', error);
        }
    }
}
```

## Доступ к файлам и камере (File API)

```javascript
// Выбор файлов с устройства
function setupFileInput() {
    const fileInput = document.getElementById('file-input');
    
    fileInput.addEventListener('change', (event) => {
        const files = event.target.files;
        
        for (let i = 0; i < files.length; i++) {
            const file = files[i];
            
            // Создание превью изображения
            if (file.type.startsWith('image/')) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    const img = document.createElement('img');
                    img.src = e.target.result;
                    document.body.appendChild(img);
                };
                reader.readAsDataURL(file);
            }
        }
    });
}
```

## Хранение данных (IndexedDB)

```javascript
// Создание базы данных для автономного хранения
class MobileDB {
    constructor() {
        this.dbName = 'MobileAppDB';
        this.version = 1;
        this.db = null;
    }
    
    async init() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);
            
            request.onerror = () => reject(request.error);
            request.onsuccess = () => {
                this.db = request.result;
                resolve(this.db);
            };
            
            request.onupgradeneeded = (event) => {
                const db = event.target.result;
                
                // Создание объектного хранилища
                if (!db.objectStoreNames.contains('cache')) {
                    const objectStore = db.createObjectStore('cache', { keyPath: 'id' });
                    objectStore.createIndex('timestamp', 'timestamp', { unique: false });
                }
            };
        });
    }
    
    async addData(data) {
        const transaction = this.db.transaction(['cache'], 'readwrite');
        const store = transaction.objectStore('cache');
        
        return new Promise((resolve, reject) => {
            const request = store.add({
                id: Date.now().toString(),
                data: data,
                timestamp: new Date().toISOString()
            });
            
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }
    
    async getData() {
        const transaction = this.db.transaction(['cache'], 'readonly');
        const store = transaction.objectStore('cache');
        
        return new Promise((resolve, reject) => {
            const request = store.getAll();
            
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }
}
```

## Вибрация (Vibration API)

```javascript
// Проверка поддержки вибрации
if ('vibrate' in navigator) {
    // Простая вибрация на 1000 мс
    navigator.vibrate(1000);
    
    // Сложный паттерн вибрации (вкл-выкл в мс)
    navigator.vibrate([100, 300, 100, 300, 200, 1000]);
    
    // Отмена вибрации
    // navigator.vibrate(0);
} else {
    console.log('Vibration API не поддерживается');
}
```

## Батарея (Battery Status API)

```javascript
// Получение информации о батарее (устаревший API)
if ('getBattery' in navigator) {
    navigator.getBattery().then((battery) => {
        console.log('Уровень заряда:', battery.level * 100 + '%');
        console.log('Заряжается:', battery.charging);
        console.log('Время до полной зарядки:', battery.chargingTime);
        console.log('Время до разрядки:', battery.dischargingTime);
        
        battery.addEventListener('levelchange', () => {
            console.log('Уровень заряда изменился:', battery.level * 100 + '%');
        });
        
        battery.addEventListener('chargingchange', () => {
            console.log('Статус зарядки изменился:', battery.charging);
        });
    });
}
```

## Связанные API: Web Share API

```javascript
// API для шаринга данных
if ('share' in navigator) {
    async function shareData(title, text, url) {
        try {
            await navigator.share({
                title: title,
                text: text,
                url: url
            });
            console.log('Успешно поделились');
        } catch (error) {
            console.error('Ошибка шаринга:', error);
        }
    }
}
```

## Практические рекомендации

### 1. Проверка поддержки API

Всегда проверяйте поддержку API перед использованием:

```javascript
function checkAPISupport() {
    const apis = {
        geolocation: 'geolocation' in navigator,
        mediaDevices: 'mediaDevices' in navigator,
        serviceWorker: 'serviceWorker' in navigator,
        pushManager: 'PushManager' in window,
        notifications: 'Notification' in window,
        indexedDB: 'indexedDB' in window,
        vibration: 'vibrate' in navigator,
        deviceOrientation: 'DeviceOrientationEvent' in window,
        deviceMotion: 'DeviceMotionEvent' in window
    };
    
    console.log('Поддержка мобильных API:', apis);
    return apis;
}
```

### 2. Обработка ошибок

Обязательно обрабатывайте ошибки и отказы пользователей:

```javascript
async function safeAPIAccess(apiFunction) {
    try {
        return await apiFunction();
    } catch (error) {
        console.error('Ошибка доступа к API:', error.message);
        
        // Показать пользователю понятное сообщение
        showUserMessage('Для использования этой функции необходимы разрешения. Пожалуйста, проверьте настройки браузера.');
    }
}
```

### 3. Учет особенностей российского рынка

В 2025 году в России важно:
- Учитывать разнообразие мобильных устройств и их возможностей
- Обеспечивать работу приложений на устройствах с разными версиями ОС
- Учитывать возможные ограничения интернет-соединения
- Обеспечивать конфиденциальность данных при использовании API

## Связанные темы

- [[Touch-события]]
- [[Адаптивность]]
- [[Оптимизация-для-мобильных]]
- [[Гибридные-приложения]]
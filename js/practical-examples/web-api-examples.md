---
aliases: ["Работа с Web API", "Web API примеры JavaScript"]
tags: [js, practical-examples, web-api, frontend]
---

# Работа с Web API (Geolocation, Notification, и другие)

Практические примеры использования различных Web API в JavaScript, включая геолокацию, уведомления, локальное хранилище и другие возможности браузера.

## 1. Geolocation API

```javascript
// Получение текущей геолокации пользователя
function getCurrentLocation() {
    return new Promise((resolve, reject) => {
        if (!navigator.geolocation) {
            reject(new Error('Geolocation is not supported by this browser'));
            return;
        }
        
        navigator.geolocation.getCurrentPosition(
            position => {
                const { latitude, longitude, accuracy } = position.coords;
                resolve({
                    latitude,
                    longitude,
                    accuracy,
                    timestamp: position.timestamp
                });
            },
            error => {
                reject(getGeolocationErrorMessage(error));
            },
            {
                enableHighAccuracy: true,
                timeout: 10000,
                maximumAge: 60000
            }
        );
    });
}

// Получение сообщения об ошибке геолокации
function getGeolocationErrorMessage(error) {
    switch(error.code) {
        case error.PERMISSION_DENIED:
            return "Пользователь отклонил запрос на геолокацию";
        case error.POSITION_UNAVAILABLE:
            return "Информация о местоположении недоступна";
        case error.TIMEOUT:
            return "Истекло время ожидания запроса на геолокацию";
        default:
            return "Произошла неизвестная ошибка";
    }
}

// Отслеживание изменения местоположения
function watchLocation() {
    if (!navigator.geolocation) {
        console.error('Geolocation is not supported');
        return null;
    }
    
    const watchId = navigator.geolocation.watchPosition(
        position => {
            console.log('Текущее местоположение:', {
                latitude: position.coords.latitude,
                longitude: position.coords.longitude,
                accuracy: position.coords.accuracy
            });
        },
        error => {
            console.error('Ошибка геолокации:', getGeolocationErrorMessage(error));
        },
        {
            enableHighAccuracy: true,
            maximumAge: 30000, // 30 секунд
            timeout: 5000
        }
    );
    
    return watchId;
}

// Пример использования
async function showUserLocation() {
    try {
        const location = await getCurrentLocation();
        console.log(`Широта: ${location.latitude}, Долгота: ${location.longitude}`);
        
        // Пример использования с картами (Google Maps API)
        // const mapUrl = `https://www.google.com/maps?q=${location.latitude},${location.longitude}`;
        // window.open(mapUrl, '_blank');
    } catch (error) {
        console.error('Не удалось получить местоположение:', error.message);
    }
}

// showUserLocation();
```

## 2. Notifications API

```javascript
// Проверка поддержки уведомлений и запрос разрешения
function requestNotificationPermission() {
    if (!('Notification' in window)) {
        console.log('Уведомления не поддерживаются этим браузером');
        return Promise.resolve('not_supported');
    }
    
    if (Notification.permission === 'granted') {
        return Promise.resolve('granted');
    }
    
    if (Notification.permission !== 'denied') {
        return Notification.requestPermission().then(permission => {
            return permission;
        });
    }
    
    return Promise.resolve('denied');
}

// Показ уведомления
function showNotification(title, options = {}) {
    return requestNotificationPermission().then(permission => {
        if (permission === 'granted') {
            const notification = new Notification(title, options);
            
            // Обработчики событий уведомления
            notification.onclick = function(event) {
                event.preventDefault();
                window.focus();
                notification.close();
                console.log('Уведомление кликнуто');
            };
            
            notification.onerror = function() {
                console.log('Ошибка уведомления');
            };
            
            notification.onshow = function() {
                console.log('Уведомление показано');
            };
            
            notification.onclose = function() {
                console.log('Уведомление закрыто');
            };
            
            return notification;
        } else {
            console.log('Разрешение на уведомления не получено');
            return null;
        }
    });
}

// Уведомление с дополнительными опциями
function showAdvancedNotification() {
    const options = {
        body: 'Это пример расширенного уведомления',
        icon: 'https://placehold.co/100', // Замените на реальный URL
        badge: 'https://placehold.co/72', // Замените на реальный URL
        tag: 'notification-tag',
        requireInteraction: false, // Уведомление автоматически закроется
        silent: false,
        vibrate: [200, 100, 200], // Вибрация: 200мс, пауза 100мс, 200мс
        actions: [
            { action: 'read', title: 'Прочитать' },
            { action: 'close', title: 'Закрыть' }
        ]
    };
    
    return showNotification('Заголовок уведомления', options);
}

// Пример использования
requestNotificationPermission().then(permission => {
    if (permission === 'granted') {
        showNotification('Привет!', {
            body: 'Это тестовое уведомление',
            icon: 'https://placehold.co/100'
        });
    }
});
```

## 3. Local Storage и Session Storage

```javascript
// Утилиты для работы с Local Storage
const StorageUtils = {
    // Сохранение объекта в localStorage
    setItem(key, value) {
        try {
            const serializedValue = JSON.stringify(value);
            localStorage.setItem(key, serializedValue);
            return true;
        } catch (error) {
            console.error('Ошибка при сохранении в localStorage:', error);
            return false;
        }
    },
    
    // Получение объекта из localStorage
    getItem(key, defaultValue = null) {
        try {
            const serializedValue = localStorage.getItem(key);
            return serializedValue === null ? defaultValue : JSON.parse(serializedValue);
        } catch (error) {
            console.error('Ошибка при получении из localStorage:', error);
            return defaultValue;
        }
    },
    
    // Удаление элемента из localStorage
    removeItem(key) {
        try {
            localStorage.removeItem(key);
            return true;
        } catch (error) {
            console.error('Ошибка при удалении из localStorage:', error);
            return false;
        }
    },
    
    // Очистка localStorage
    clear() {
        try {
            localStorage.clear();
            return true;
        } catch (error) {
            console.error('Ошибка при очистке localStorage:', error);
            return false;
        }
    },
    
    // Проверка доступности localStorage
    isAvailable() {
        try {
            const test = '__storage_test__';
            localStorage.setItem(test, test);
            localStorage.removeItem(test);
            return true;
        } catch (error) {
            return false;
        }
    }
};

// Управление пользовательскими настройками
class UserPreferences {
    constructor() {
        this.storageKey = 'userPreferences';
        this.defaults = {
            theme: 'light',
            language: 'ru',
            notifications: true,
            fontSize: 16
        };
        
        this.load();
    }
    
    load() {
        this.preferences = StorageUtils.getItem(this.storageKey, this.defaults);
    }
    
    save() {
        StorageUtils.setItem(this.storageKey, this.preferences);
    }
    
    get(key) {
        return this.preferences[key] !== undefined ? this.preferences[key] : this.defaults[key];
    }
    
    set(key, value) {
        this.preferences[key] = value;
        this.save();
    }
    
    reset() {
        this.preferences = { ...this.defaults };
        this.save();
    }
}

// Пример использования
const userPrefs = new UserPreferences();
console.log('Тема:', userPrefs.get('theme'));
userPrefs.set('theme', 'dark');
console.log('Новая тема:', userPrefs.get('theme'));
```

## 4. Clipboard API

```javascript
// Копирование текста в буфер обмена
async function copyToClipboard(text) {
    if (navigator.clipboard && window.isSecureContext) {
        // Современный способ (работает только в безопасных контекстах)
        try {
            await navigator.clipboard.writeText(text);
            console.log('Текст скопирован в буфер обмена');
            return true;
        } catch (error) {
            console.error('Не удалось скопировать текст:', error);
            return false;
        }
    } else {
        // Резервный метод для старых браузеров или небезопасных контекстов
        return fallbackCopyTextToClipboard(text);
    }
}

// Резервный метод для копирования
function fallbackCopyTextToClipboard(text) {
    const textArea = document.createElement("textarea");
    textArea.value = text;
    
    // Необходимо для Firefox
    textArea.style.top = "0";
    textArea.style.left = "0";
    textArea.style.position = "fixed";
    textArea.style.opacity = "0";
    
    document.body.appendChild(textArea);
    textArea.focus();
    textArea.select();
    
    try {
        const successful = document.execCommand('copy');
        document.body.removeChild(textArea);
        console.log(successful ? 'Текст скопирован' : 'Не удалось скопировать текст');
        return successful;
    } catch (error) {
        console.error('Ошибка при копировании текста:', error);
        document.body.removeChild(textArea);
        return false;
    }
}

// Чтение текста из буфера обмена
async function readFromClipboard() {
    if (navigator.clipboard && window.isSecureContext) {
        try {
            const text = await navigator.clipboard.readText();
            console.log('Текст из буфера:', text);
            return text;
        } catch (error) {
            console.error('Не удалось прочитать текст из буфера:', error);
            return null;
        }
    } else {
        console.log('Clipboard API не поддерживается или контекст небезопасен');
        return null;
    }
}

// Пример использования
async function exampleClipboard() {
    const textToCopy = "Пример текста для копирования";
    
    // Копируем текст
    const copySuccess = await copyToClipboard(textToCopy);
    if (copySuccess) {
        // Читаем обратно
        setTimeout(async () => {
            const pastedText = await readFromClipboard();
            console.log('Прочитанный текст:', pastedText);
        }, 1000);
    }
}

// exampleClipboard();
```

## 5. Fullscreen API

```javascript
// Утилиты для работы с полноэкранным режимом
const FullscreenUtils = {
    // Проверка поддержки Fullscreen API
    isSupported() {
        return !!(document.fullscreenEnabled || 
                  document.webkitFullscreenEnabled || 
                  document.mozFullScreenEnabled || 
                  document.msFullscreenEnabled);
    },
    
    // Вход в полноэкранный режим
    async enterFullscreen(element = document.documentElement) {
        if (!this.isSupported()) {
            throw new Error('Fullscreen API не поддерживается');
        }
        
        try {
            if (element.requestFullscreen) {
                return await element.requestFullscreen();
            } else if (element.webkitRequestFullscreen) {
                return await element.webkitRequestFullscreen();
            } else if (element.mozRequestFullScreen) {
                return await element.mozRequestFullScreen();
            } else if (element.msRequestFullscreen) {
                return await element.msRequestFullscreen();
            }
        } catch (error) {
            console.error('Ошибка при входе в полноэкранный режим:', error);
            throw error;
        }
    },
    
    // Выход из полноэкранного режима
    async exitFullscreen() {
        try {
            if (document.exitFullscreen) {
                return await document.exitFullscreen();
            } else if (document.webkitExitFullscreen) {
                return await document.webkitExitFullscreen();
            } else if (document.mozCancelFullScreen) {
                return await document.mozCancelFullScreen();
            } else if (document.msExitFullscreen) {
                return await document.msExitFullscreen();
            }
        } catch (error) {
            console.error('Ошибка при выходе из полноэкранного режима:', error);
            throw error;
        }
    },
    
    // Проверка, находится ли документ в полноэкранном режиме
    isFullscreen() {
        return !!(document.fullscreenElement || 
                  document.webkitFullscreenElement || 
                  document.mozFullScreenElement || 
                  document.msFullscreenElement);
    },
    
    // Переключение полноэкранного режима
    async toggleFullscreen(element = document.documentElement) {
        if (this.isFullscreen()) {
            await this.exitFullscreen();
        } else {
            await this.enterFullscreen(element);
        }
    }
};

// Пример использования
document.addEventListener('keydown', (event) => {
    if (event.key === 'F11') {
        event.preventDefault();
        FullscreenUtils.toggleFullscreen();
    }
});
```

## 6. History API

```javascript
// Утилиты для работы с History API
const HistoryUtils = {
    // Добавление записи в историю
    pushState(state, title, url) {
        try {
            history.pushState(state, title, url);
            console.log(`Состояние добавлено: ${url}`);
        } catch (error) {
            console.error('Ошибка при добавлении состояния:', error);
        }
    },
    
    // Замена текущей записи в истории
    replaceState(state, title, url) {
        try {
            history.replaceState(state, title, url);
            console.log(`Состояние заменено: ${url}`);
        } catch (error) {
            console.error('Ошибка при замене состояния:', error);
        }
    },
    
    // Навигация назад
    goBack() {
        history.back();
    },
    
    // Навигация вперед
    goForward() {
        history.forward();
    },
    
    // Переход на n шагов в истории
    go(n) {
        history.go(n);
    }
};

// Обработка события popstate для навигации
window.addEventListener('popstate', (event) => {
    console.log('Изменение состояния истории:', event.state);
    
    // Здесь можно обновить содержимое страницы в зависимости от состояния
    if (event.state) {
        console.log('Состояние:', event.state);
        // Обновление UI в зависимости от состояния
    }
});

// Пример использования
function navigateToPage(page, title, url) {
    HistoryUtils.pushState({ page, timestamp: Date.now() }, title, url);
    // Обновляем содержимое страницы
    document.title = title;
    updatePageContent(page);
}

function updatePageContent(page) {
    // Обновление содержимого страницы в зависимости от страницы
    console.log(`Обновление содержимого для страницы: ${page}`);
}
```

## 7. Intersection Observer API

```javascript
// Утилиты для ленивой загрузки изображений
class LazyLoader {
    constructor() {
        this.imageObserver = null;
        this.setupObserver();
    }
    
    setupObserver() {
        this.imageObserver = new IntersectionObserver((entries, observer) => {
            entries.forEach(entry => {
                if (entry.isIntersecting) {
                    this.loadImage(entry.target);
                    observer.unobserve(entry.target);
                }
            });
        }, {
            rootMargin: '50px 0px', // Начинать загрузку за 50px до появления
            threshold: 0.01
        });
    }
    
    loadImage(img) {
        const src = img.dataset.src;
        if (src) {
            img.src = src;
            img.classList.remove('lazy');
            img.classList.add('loaded');
            
            // Убираем data-атрибут
            delete img.dataset.src;
        }
    }
    
    observeImages(container = document) {
        const lazyImages = container.querySelectorAll('img[data-src]');
        lazyImages.forEach(img => {
            this.imageObserver.observe(img);
        });
    }
    
    disconnect() {
        if (this.imageObserver) {
            this.imageObserver.disconnect();
        }
    }
}

// Утилиты для определения видимости элементов
class VisibilityDetector {
    constructor(callback) {
        this.callback = callback;
        this.elementObserver = new IntersectionObserver((entries) => {
            entries.forEach(entry => {
                this.callback(entry.target, entry.isIntersecting, entry.intersectionRatio);
            });
        }, {
            threshold: [0, 0.25, 0.5, 0.75, 1] // Уведомления при 0%, 25%, 50%, 75% и 100% видимости
        });
    }
    
    observe(element) {
        this.elementObserver.observe(element);
    }
    
    unobserve(element) {
        this.elementObserver.unobserve(element);
    }
    
    disconnect() {
        this.elementObserver.disconnect();
    }
}

// Пример использования
const lazyLoader = new LazyLoader();
lazyLoader.observeImages();

// Добавляем обработчик для анимации при скролле
const visibilityDetector = new VisibilityDetector((element, isVisible, ratio) => {
    if (isVisible && ratio > 0.5) {
        element.classList.add('visible');
    } else {
        element.classList.remove('visible');
    }
});

// Пример HTML:
// <img data-src="image.jpg" class="lazy" alt="Lazy image">
```

## 8. Battery Status API (доступно не во всех браузерах)

```javascript
// Утилиты для получения информации о батарее
class BatteryMonitor {
    constructor() {
        this.battery = null;
        this.callbacks = {
            level: [],
            charging: [],
            time: []
        };
    }
    
    async init() {
        if ('getBattery' in navigator) {
            try {
                this.battery = await navigator.getBattery();
                this.setupEvents();
                return true;
            } catch (error) {
                console.error('Не удалось получить информацию о батарее:', error);
                return false;
            }
        } else {
            console.log('Battery Status API не поддерживается');
            return false;
        }
    }
    
    setupEvents() {
        this.battery.addEventListener('chargingchange', () => {
            this.callbacks.charging.forEach(callback => callback(this.battery.charging));
        });
        
        this.battery.addEventListener('levelchange', () => {
            this.callbacks.level.forEach(callback => callback(this.battery.level));
        });
        
        this.battery.addEventListener('chargingtimechange', () => {
            this.callbacks.time.forEach(callback => callback(this.battery.chargingTime));
        });
        
        this.battery.addEventListener('dischargingtimechange', () => {
            this.callbacks.time.forEach(callback => callback(this.battery.dischargingTime));
        });
    }
    
    // Подписка на изменения
    onChargingChange(callback) {
        this.callbacks.charging.push(callback);
    }
    
    onLevelChange(callback) {
        this.callbacks.level.push(callback);
    }
    
    onTimeChange(callback) {
        this.callbacks.time.push(callback);
    }
    
    // Получение текущего состояния
    getBatteryInfo() {
        if (!this.battery) return null;
        
        return {
            level: this.battery.level * 100,
            charging: this.battery.charging,
            chargingTime: this.battery.chargingTime,
            dischargingTime: this.battery.dischargingTime
        };
    }
}

// Пример использования (где поддерживается)
// const batteryMonitor = new BatteryMonitor();
// batteryMonitor.init().then(success => {
//     if (success) {
//         batteryMonitor.onLevelChange(level => {
//             console.log(`Уровень заряда: ${Math.round(level * 100)}%`);
//         });
//         
//         batteryMonitor.onChargingChange(charging => {
//             console.log(`Заряжается: ${charging}`);
//         });
//     }
// });
```

## 9. Page Visibility API

```javascript
// Утилиты для определения видимости страницы
class PageVisibilityManager {
    constructor() {
        this.callbacks = {
            hidden: [],
            visible: []
        };
        
        this.setupVisibilityListener();
    }
    
    setupVisibilityListener() {
        document.addEventListener('visibilitychange', () => {
            if (document.hidden) {
                this.callbacks.hidden.forEach(callback => callback());
            } else {
                this.callbacks.visible.forEach(callback => callback());
            }
        });
    }
    
    onHidden(callback) {
        this.callbacks.hidden.push(callback);
    }
    
    onVisible(callback) {
        this.callbacks.visible.push(callback);
    }
    
    isHidden() {
        return document.hidden;
    }
    
    isVisible() {
        return !document.hidden;
    }
    
    getVisibilityState() {
        return document.visibilityState;
    }
}

// Пример использования
const visibilityManager = new PageVisibilityManager();

visibilityManager.onHidden(() => {
    console.log('Страница скрыта');
    // Можно приостановить анимации, таймеры и т.д.
});

visibilityManager.onVisible(() => {
    console.log('Страница видима');
    // Можно возобновить анимации, обновить данные и т.д.
});

// Пример приостановки анимации при скрытии страницы
let animationFrameId;
let isAnimating = false;

function animate() {
    if (!visibilityManager.isHidden()) {
        // Выполняем анимацию
        console.log('Анимация работает');
    }
    animationFrameId = requestAnimationFrame(animate);
}

visibilityManager.onHidden(() => {
    cancelAnimationFrame(animationFrameId);
    isAnimating = false;
});

visibilityManager.onVisible(() => {
    if (!isAnimating) {
        animationFrameId = requestAnimationFrame(animate);
        isAnimating = true;
    }
});
```

Эти примеры демонстрируют использование различных Web API для расширения возможностей веб-приложений и улучшения пользовательского опыта.

См. также:
- [[js/practical-examples/canvas-graphics-examples]] - Работа с canvas и графикой
- [[js/practical-examples/web-workers-examples]] - Примеры использования Web Workers
- [[js/practical-examples/fetch-websocket-examples]] - Примеры с Fetch API и WebSocket
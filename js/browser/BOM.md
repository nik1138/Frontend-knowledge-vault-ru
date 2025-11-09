# BOM (Browser Object Model)

BOM (Browser Object Model) - это набор объектов, предоставляемых браузером для взаимодействия с окном браузера и другими компонентами. В отличие от DOM, который работает с содержимым страницы, BOM предоставляет интерфейсы для взаимодействия с самим браузером.

## Структура BOM

```
window (глобальный объект)
├── navigator - информация о браузере и системе
├── screen - информация об экране
├── location - информация об URL текущей страницы
├── history - история посещенных страниц
├── document - объект DOM (часть BOM)
└── frames - доступ к фреймам
```

## Объект Window

### Основные свойства и методы

```javascript
// Глобальные методы
alert('Сообщение'); // Показать сообщение
confirm('Подтвердить действие?'); // Подтвердить действие
prompt('Введите значение:'); // Запросить ввод

// Работа с размерами окна
console.log(window.innerWidth);  // Ширина окна
console.log(window.innerHeight); // Высота окна

// Работа с прокруткой
console.log(window.pageXOffset); // Горизонтальная прокрутка
console.log(window.pageYOffset); // Вертикальная прокрутка

// Открытие и закрытие окон
const newWindow = window.open('https://example.com', '_blank', 'width=600,height=400');
newWindow.close();

// Таймеры
const timeoutId = setTimeout(() => console.log('Выполнено через 2 сек'), 2000);
const intervalId = setInterval(() => console.log('Выполняется каждые 1 сек'), 1000);

// Отмена таймеров
clearTimeout(timeoutId);
clearInterval(intervalId);
```

### Прокрутка окна

```javascript
// Плавная прокрутка
window.scrollTo({
    top: 100,
    left: 0,
    behavior: 'smooth'
});

// Прокрутка к элементу
const element = document.getElementById('target');
element.scrollIntoView({
    behavior: 'smooth',
    block: 'start'
});

// Получение текущей позиции прокрутки
console.log(window.scrollX, window.scrollY);
```

## Объект Navigator

### Информация о браузере и системе

```javascript
// Основная информация
console.log(navigator.userAgent);     // Строка User Agent
console.log(navigator.appName);       // Имя браузера
console.log(navigator.appVersion);    // Версия браузера
console.log(navigator.platform);      // Платформа

// Поддержка возможностей
console.log(navigator.onLine);        // Статус подключения
console.log(navigator.cookieEnabled); // Поддержка cookies
console.log(navigator.language);      // Язык браузера

// Современные API
if ('geolocation' in navigator) {
    // Поддержка геолокации
}

if ('serviceWorker' in navigator) {
    // Поддержка Service Workers
}

if ('mediaDevices' in navigator && 'getUserMedia' in navigator.mediaDevices) {
    // Поддержка доступа к камере/микрофону
}
```

### Современные возможности Navigator

```javascript
// Доступ к медиаустройствам
navigator.mediaDevices.enumerateDevices()
    .then(devices => {
        devices.forEach(device => {
            console.log(`${device.kind}: ${device.label} id = ${device.deviceId}`);
        });
    });

// Поддержка современных API
const supports = {
    clipboard: 'clipboard' in navigator,
    credentials: 'credentials' in navigator,
    locks: 'locks' in navigator,
    storage: 'storage' in navigator,
    wakeLock: 'wakeLock' in navigator
};
```

## Объект Screen

### Информация об экране

```javascript
// Основные свойства
console.log(screen.width);        // Ширина экрана
console.log(screen.height);       // Высота экрана
console.log(screen.availWidth);   // Доступная ширина (без панелей)
console.log(screen.availHeight);  // Доступная высота (без панелей)
console.log(screen.colorDepth);   // Глубина цвета
console.log(screen.pixelDepth);   // Глубина пикселей
```

## Объект Location

### Работа с URL

```javascript
// Текущий URL
console.log(location.href);       // Полный URL
console.log(location.protocol);   // Протокол (http:, https:)
console.log(location.host);       // Хост (домен:порт)
console.log(location.hostname);   // Домен
console.log(location.port);       // Порт
console.log(location.pathname);   // Путь
console.log(location.search);     // Параметры запроса
console.log(location.hash);       // Якорь

// Изменение URL
location.href = 'https://example.com';  // Переход по новому URL
location.assign('https://example.com'); // Аналогично
location.replace('https://example.com'); // Заменить текущую страницу (без истории)

// Перезагрузка страницы
location.reload(); // Перезагрузить страницу
```

### Работа с параметрами URL

```javascript
// Получение параметров URL
const urlParams = new URLSearchParams(window.location.search);
const name = urlParams.get('name');
const age = urlParams.get('age');

// Проверка наличия параметра
if (urlParams.has('debug')) {
    console.log('Режим отладки включен');
}

// Итерация по параметрам
for (const [key, value] of urlParams) {
    console.log(`${key}: ${value}`);
}
```

## Объект History

### Навигация по истории

```javascript
// Навигация
history.back();    // Назад
history.forward(); // Вперед
history.go(-2);    // На 2 страницы назад
history.go(2);     // На 2 страницы вперед

// Длина истории
console.log(history.length);

// Современные методы (HTML5 History API)
history.pushState({page: 1}, 'Page 1', '/page1'); // Добавить новую запись
history.replaceState({page: 2}, 'Page 2', '/page2'); // Заменить текущую запись

// Событие popstate
window.addEventListener('popstate', function(event) {
    console.log('Состояние истории изменено:', event.state);
});
```

### Работа с состоянием истории

```javascript
// Пример использования History API
class Router {
    constructor() {
        this.routes = {};
        this.currentRoute = null;
        
        window.addEventListener('popstate', this.handlePopState.bind(this));
    }
    
    addRoute(path, handler) {
        this.routes[path] = handler;
    }
    
    navigate(path) {
        history.pushState({path}, '', path);
        this.handleRoute(path);
    }
    
    handlePopState(event) {
        const path = event.state ? event.state.path : window.location.pathname;
        this.handleRoute(path);
    }
    
    handleRoute(path) {
        if (this.routes[path]) {
            this.routes[path]();
            this.currentRoute = path;
        }
    }
}

// Использование
const router = new Router();
router.addRoute('/home', () => console.log('Главная страница'));
router.addRoute('/about', () => console.log('О нас'));
```

## Объект Frames

### Работа с фреймами

```javascript
// Доступ к фреймам
console.log(frames.length); // Количество фреймов
console.log(frames[0]);     // Первый фрейм
console.log(frames['name']); // Фрейм по имени

// Доступ к родительскому окну
if (window.parent !== window) {
    // Мы в фрейме
    console.log('Родительское окно:', window.parent);
}

// Доступ к верхнему окну
console.log('Верхнее окно:', window.top);
```

## Современные возможности BOM

### Page Visibility API

```javascript
// Проверка видимости страницы
console.log(document.hidden); // true если страница скрыта
console.log(document.visibilityState); // 'visible', 'hidden', 'prerender', 'unloaded'

// Событие изменения видимости
document.addEventListener('visibilitychange', function() {
    if (document.hidden) {
        console.log('Страница скрыта');
        // Приостановить таймеры, анимации и т.д.
    } else {
        console.log('Страница видима');
        // Возобновить таймеры, анимации и т.д.
    }
});
```

### Fullscreen API

```javascript
// Проверка поддержки
if (document.documentElement.requestFullscreen) {
    // Вход в полноэкранный режим
    document.documentElement.requestFullscreen()
        .catch(err => console.error('Ошибка полноэкранного режима:', err));
    
    // Выход из полноэкранного режима
    document.exitFullscreen();
    
    // Событие изменения полноэкранного режима
    document.addEventListener('fullscreenchange', function() {
        if (document.fullscreenElement) {
            console.log('Полноэкранный режим включен');
        } else {
            console.log('Полноэкранный режим выключен');
        }
    });
}
```

### Network Information API

```javascript
// Получение информации о соединении
if ('connection' in navigator) {
    const connection = navigator.connection;
    console.log('Тип соединения:', connection.effectiveType);
    console.log('Пропускная способность:', connection.downlink);
    console.log('Задержка:', connection.rtt);
}
```

## Практические примеры

### Обнаружение мобильного устройства

```javascript
function isMobileDevice() {
    return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
}

function isMobileScreen() {
    return window.innerWidth <= 768;
}

// Комбинированный подход
function isMobile() {
    return isMobileDevice() || isMobileScreen();
}
```

### Обнаружение онлайн/оффлайн статуса

```javascript
class NetworkMonitor {
    constructor() {
        this.isOnline = navigator.onLine;
        this.callbacks = {
            online: [],
            offline: []
        };
        
        window.addEventListener('online', this.handleOnline.bind(this));
        window.addEventListener('offline', this.handleOffline.bind(this));
    }
    
    onOnline(callback) {
        this.callbacks.online.push(callback);
    }
    
    onOffline(callback) {
        this.callbacks.offline.push(callback);
    }
    
    handleOnline() {
        this.isOnline = true;
        this.callbacks.online.forEach(callback => callback());
    }
    
    handleOffline() {
        this.isOnline = false;
        this.callbacks.offline.forEach(callback => callback());
    }
}

// Использование
const networkMonitor = new NetworkMonitor();
networkMonitor.onOnline(() => console.log('Соединение восстановлено'));
networkMonitor.onOffline(() => console.log('Соединение потеряно'));
```

### Определение производительности устройства

```javascript
class DevicePerformance {
    static async estimate() {
        const start = performance.now();
        
        // Выполнение вычислительной задачи
        let result = 0;
        for (let i = 0; i < 1000000; i++) {
            result += Math.sqrt(i);
        }
        
        const end = performance.now();
        const executionTime = end - start;
        
        return {
            executionTime: executionTime,
            performance: executionTime < 50 ? 'high' : executionTime < 100 ? 'medium' : 'low'
        };
    }
}

// Использование
DevicePerformance.estimate().then(performance => {
    console.log('Производительность:', performance);
});
```

## Лучшие практики

### 1. Проверка поддержки API

```javascript
// Всегда проверяйте поддержку API перед использованием
if ('serviceWorker' in navigator) {
    // Использовать Service Worker
} else {
    // Альтернативная реализация
}
```

### 2. Обработка ошибок

```javascript
// Обработка ошибок при работе с BOM
try {
    const connection = navigator.connection;
    if (connection) {
        console.log('Скорость:', connection.downlink);
    }
} catch (error) {
    console.warn('Не удалось получить информацию о соединении:', error);
}
```

### 3. Использование современных API

```javascript
// Предпочтение современным API
// Вместо window.location.hash используйте History API
history.pushState({page: 1}, 'Page 1', '/page1');
```

## Безопасность и ограничения

### Ограничения кросс-домена

```javascript
// Ограничения при работе с фреймами из других доменов
try {
    const frameWindow = frames[0];
    // Это может вызвать ошибку безопасности
    console.log(frameWindow.location.href);
} catch (error) {
    console.error('Доступ к фрейму запрещен:', error);
}
```

### Правила использования

1. Не полагайтесь на user agent для определения возможностей
2. Используйте feature detection вместо browser detection
3. Учитывайте приватность при использовании информации о пользователе
4. Соблюдайте ограничения кросс-доменных запросов

#javascript #bom #frontend #web-development #browser-api
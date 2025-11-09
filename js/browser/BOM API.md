# BOM API

## Введение

BOM (Browser Object Model) - это набор объектов, предоставляемых браузером для взаимодействия с самим браузером и его компонентами. В отличие от DOM, который предоставляет интерфейс для работы с содержимым документа, BOM позволяет управлять окном браузера, историей, информацией о браузере и устройстве.

## Структура BOM

BOM включает в себя несколько основных объектов:

- `window` - корневой объект, представляющий окно браузера
- `navigator` - информация о браузере и системе пользователя
- `location` - информация об URL текущей страницы
- `history` - история посещенных страниц
- `screen` - информация об экране пользователя
- `document` - часть DOM, но также часть BOM

## Основные объекты BOM

### Window Object

`window` - это глобальный объект в браузере, который представляет окно браузера.

```javascript
// Основные свойства window
console.log(window.innerWidth);  // Ширина области просмотра
console.log(window.innerHeight); // Высота области просмотра
console.log(window.location);    // Объект Location
console.log(window.navigator);   // Объект Navigator

// Основные методы window
window.alert('Привет!');         // Показать алерт
window.confirm('Вы уверены?');   // Подтверждение
window.prompt('Введите имя:');   // Ввод текста

// Открытие и закрытие окон
const newWindow = window.open('https://example.com', '_blank', 'width=600,height=400');
newWindow.close();

// Таймеры
const timeoutId = window.setTimeout(() => {
    console.log('Выполнено через 1 секунду');
}, 1000);

const intervalId = window.setInterval(() => {
    console.log('Выполняется каждую секунду');
}, 1000);

// Остановка таймеров
window.clearTimeout(timeoutId);
window.clearInterval(intervalId);
```

### Navigator Object

`navigator` предоставляет информацию о браузере и системе пользователя.

```javascript
// Информация о браузере
console.log(navigator.userAgent);     // Строка User Agent
console.log(navigator.appName);       // Имя приложения
console.log(navigator.appVersion);    // Версия приложения
console.log(navigator.platform);      // Платформа

// Возможности браузера
console.log(navigator.onLine);        // Статус подключения
console.log(navigator.cookieEnabled); // Поддержка cookies
console.log(navigator.javaEnabled()); // Поддержка Java

// Современные API через navigator
navigator.geolocation.getCurrentPosition(position => {
    console.log('Широта:', position.coords.latitude);
    console.log('Долгота:', position.coords.longitude);
});

// Проверка поддержки API
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js');
}

if ('permissions' in navigator) {
    navigator.permissions.query({name: 'geolocation'}).then(result => {
        console.log('Разрешение на геолокацию:', result.state);
    });
}
```

### Location Object

`location` предоставляет информацию об URL текущей страницы и позволяет изменять его.

```javascript
// Свойства location
console.log(location.href);        // Полный URL
console.log(location.protocol);    // Протокол (http:, https:)
console.log(location.host);        // Хост (домен + порт)
console.log(location.hostname);    // Имя хоста
console.log(location.port);        // Порт
console.log(location.pathname);    // Путь
console.log(location.search);      // Строка запроса
console.log(location.hash);        // Хэш

// Изменение URL
location.href = 'https://example.com';  // Полная перезагрузка страницы
location.assign('https://example.com'); // Альтернатива href
location.replace('https://example.com'); // Замена текущей страницы (без истории)

// Работа с параметрами URL
const urlParams = new URLSearchParams(location.search);
const name = urlParams.get('name');
const age = urlParams.get('age');

// Создание нового URL
const newUrl = new URL('https://example.com/api');
newUrl.searchParams.append('query', 'javascript');
newUrl.searchParams.append('limit', '10');
console.log(newUrl.toString()); // https://example.com/api?query=javascript&limit=10
```

### History Object

`history` позволяет управлять историей сессии браузера.

```javascript
// Навигация по истории
history.back();     // Назад
history.forward();  // Вперед
history.go(-2);     // На 2 страницы назад

// Современное управление историей (HTML5 History API)
history.pushState({page: 1}, 'Title', '/page1'); // Добавить в историю
history.replaceState({page: 2}, 'Title', '/page2'); // Заменить текущую запись

// Событие popstate для отслеживания изменений истории
window.addEventListener('popstate', (event) => {
    console.log('Состояние истории изменено:', event.state);
    // Обновление содержимого страницы без перезагрузки
});
```

### Screen Object

`screen` предоставляет информацию об экране пользователя.

```javascript
// Свойства экрана
console.log(screen.width);         // Ширина экрана
console.log(screen.height);        // Высота экрана
console.log(screen.availWidth);    // Доступная ширина
console.log(screen.availHeight);   // Доступная высота
console.log(screen.colorDepth);    // Глубина цвета
console.log(screen.pixelDepth);    // Глубина пикселей
```

## Примеры из промышленной разработки

### Определение мобильного устройства

```javascript
function isMobileDevice() {
    return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
}

// Или более точное определение
function getDeviceInfo() {
    const isMobile = window.innerWidth <= 768;
    const isTablet = window.innerWidth > 768 && window.innerWidth <= 1024;
    const isDesktop = window.innerWidth > 1024;
    
    return {
        isMobile,
        isTablet,
        isDesktop,
        userAgent: navigator.userAgent,
        platform: navigator.platform
    };
}

// Использование
const deviceInfo = getDeviceInfo();
if (deviceInfo.isMobile) {
    // Загрузка мобильной версии интерфейса
    loadMobileUI();
}
```

### Управление сессией и редиректами

```javascript
// Класс для управления сессией
class SessionManager {
    constructor() {
        this.checkSession();
        this.setupEventListeners();
    }
    
    checkSession() {
        // Проверка срока действия сессии
        const sessionExpiry = localStorage.getItem('sessionExpiry');
        if (sessionExpiry && new Date() > new Date(sessionExpiry)) {
            this.logout();
        }
    }
    
    logout() {
        localStorage.removeItem('sessionToken');
        localStorage.removeItem('sessionExpiry');
        window.location.href = '/login';
    }
    
    setupEventListeners() {
        // Обработка события beforeunload
        window.addEventListener('beforeunload', () => {
            // Сохранение состояния приложения
            localStorage.setItem('appState', JSON.stringify(this.getState()));
        });
    }
    
    getState() {
        return {
            currentPage: window.location.pathname,
            scrollPosition: window.pageYOffset
        };
    }
}

const sessionManager = new SessionManager();
```

### Работа с URL и параметрами

```javascript
// Класс для управления URL параметрами
class URLManager {
    constructor() {
        this.params = new URLSearchParams(window.location.search);
    }
    
    getParam(name) {
        return this.params.get(name);
    }
    
    setParam(name, value) {
        this.params.set(name, value);
        this.updateURL();
    }
    
    removeParam(name) {
        this.params.delete(name);
        this.updateURL();
    }
    
    updateURL() {
        const newURL = `${window.location.pathname}?${this.params.toString()}${window.location.hash}`;
        window.history.replaceState({}, '', newURL);
    }
    
    getAllParams() {
        const params = {};
        for (const [key, value] of this.params) {
            params[key] = value;
        }
        return params;
    }
}

// Использование
const urlManager = new URLManager();
const searchTerm = urlManager.getParam('search');
const page = urlManager.getParam('page') || 1;

// Обновление параметров при поиске
function performSearch(query) {
    urlManager.setParam('search', query);
    urlManager.removeParam('page'); // Сброс страницы при новом поиске
    searchAPI(query);
}
```

## Лучшие практики

### 1. Проверка поддержки API

```javascript
// Всегда проверяйте поддержку API перед использованием
if ('geolocation' in navigator) {
    navigator.geolocation.getCurrentPosition(
        position => console.log(position),
        error => console.error(error)
    );
} else {
    console.log('Geolocation не поддерживается');
}
```

### 2. Обработка ошибок

```javascript
// Обработка ошибок при работе с BOM API
try {
    const result = await navigator.permissions.query({name: 'camera'});
    console.log('Разрешение камеры:', result.state);
} catch (error) {
    console.error('Ошибка при запросе разрешения камеры:', error);
}
```

### 3. Оптимизация производительности

```javascript
// Использование debounce для событий, вызываемых часто
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

// Пример использования с событием resize
const debouncedResize = debounce(() => {
    console.log('Размер окна изменен');
    updateLayout();
}, 250);

window.addEventListener('resize', debouncedResize);
```

## Современные API

### Page Visibility API

```javascript
// Для определения видимости страницы
document.addEventListener('visibilitychange', () => {
    if (document.hidden) {
        console.log('Страница скрыта');
        // Приостановить анимации, таймеры и т.д.
    } else {
        console.log('Страница видима');
        // Возобновить анимации, таймеры и т.д.
    }
});
```

### Network Information API

```javascript
// Для получения информации о подключении
if ('connection' in navigator) {
    const connection = navigator.connection;
    console.log('Тип подключения:', connection.effectiveType);
    console.log('Пропускная способность:', connection.downlink);
    
    connection.addEventListener('change', () => {
        console.log('Подключение изменено:', connection.effectiveType);
    });
}
```

## Безопасность

При работе с BOM API важно учитывать безопасность:

```javascript
// При работе с URL всегда проверяйте домен
function safeRedirect(url) {
    const allowedDomains = ['example.com', 'sub.example.com'];
    const urlObj = new URL(url);
    
    if (allowedDomains.includes(urlObj.hostname)) {
        window.location.href = url;
    } else {
        console.warn('Небезопасный редирект:', url);
    }
}

// При работе с localStorage учитывайте конфиденциальность
function storeSensitiveData(data) {
    // Не храните чувствительные данные в localStorage
    // Используйте sessionStorage или передавайте через безопасные каналы
    console.warn('Чувствительные данные не должны храниться в localStorage');
}
```

## Теги

#javascript #bom #api #frontend #security #browser
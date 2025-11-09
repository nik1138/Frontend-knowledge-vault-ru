# localStorage и sessionStorage

localStorage и sessionStorage - это веб-хранилища, предоставляемые браузером для хранения данных на стороне клиента. Они позволяют сохранять пары ключ-значение в браузере без ограничения по времени (localStorage) или до закрытия вкладки (sessionStorage).

## Основные различия

| Характеристика | localStorage | sessionStorage |
|----------------|--------------|----------------|
| Срок хранения | Постоянный (до удаления пользователем) | До закрытия вкладки/окна |
| Доступность | Во всех вкладках одного домена | Только в текущей вкладке |
| Использование | Долгосрочные данные, настройки | Временные данные, формы |
| Ограничение | Обычно 5-10 МБ | Обычно 5-10 МБ |

## Основные методы

### localStorage

```javascript
// Сохранение данных
localStorage.setItem('key', 'value');

// Получение данных
const value = localStorage.getItem('key');

// Удаление данных
localStorage.removeItem('key');

// Очистка всего хранилища
localStorage.clear();

// Получение ключа по индексу
const key = localStorage.key(0);

// Получение количества элементов
const length = localStorage.length;
```

### sessionStorage

```javascript
// Аналогичные методы для sessionStorage
sessionStorage.setItem('key', 'value');
const value = sessionStorage.getItem('key');
sessionStorage.removeItem('key');
sessionStorage.clear();
```

## Работа с различными типами данных

### Хранение объектов и массивов

```javascript
// localStorage хранит только строки, поэтому используем JSON
const userData = {
    name: 'Иван',
    age: 30,
    preferences: ['dark-mode', 'notifications']
};

// Сохранение объекта
localStorage.setItem('userData', JSON.stringify(userData));

// Получение объекта
const storedUserData = JSON.parse(localStorage.getItem('userData'));
console.log(storedUserData.name); // 'Иван'

// Сохранение и получение массива
const favorites = ['item1', 'item2', 'item3'];
localStorage.setItem('favorites', JSON.stringify(favorites));
const storedFavorites = JSON.parse(localStorage.getItem('favorites'));
```

### Хранение сложных структур данных

```javascript
// Пример хранения сложной структуры
const appState = {
    user: {
        id: 1,
        name: 'Иван',
        email: 'ivan@example.com'
    },
    settings: {
        theme: 'dark',
        language: 'ru',
        notifications: true
    },
    recentItems: [
        { id: 1, title: 'Элемент 1', timestamp: Date.now() },
        { id: 2, title: 'Элемент 2', timestamp: Date.now() }
    ]
};

localStorage.setItem('appState', JSON.stringify(appState));

// Функция для безопасного получения данных
function getStoredData(key, defaultValue = null) {
    try {
        const item = localStorage.getItem(key);
        return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
        console.error('Ошибка при чтении из localStorage:', error);
        return defaultValue;
    }
}

const state = getStoredData('appState', {});
```

## Обработка событий хранения

### Событие storage

```javascript
// Событие срабатывает при изменении данных в другом окне/вкладке
window.addEventListener('storage', function(event) {
    console.log('Хранилище изменено:');
    console.log('Ключ:', event.key);
    console.log('Старое значение:', event.oldValue);
    console.log('Новое значение:', event.newValue);
    console.log('URL источника:', event.url);
    
    // Обновление интерфейса при изменении данных
    if (event.key === 'theme') {
        document.body.className = event.newValue || 'light';
    }
});
```

## Управление хранилищем

### Класс для работы с localStorage

```javascript
class LocalStorageManager {
    constructor(namespace = '') {
        this.namespace = namespace ? `${namespace}_` : '';
    }
    
    // Добавление префикса к ключу
    getKey(key) {
        return this.namespace + key;
    }
    
    // Сохранение данных с меткой времени
    set(key, value, ttl = null) {
        const item = {
            value: JSON.stringify(value),
            timestamp: Date.now(),
            ttl: ttl // время жизни в миллисекундах
        };
        
        try {
            localStorage.setItem(this.getKey(key), JSON.stringify(item));
        } catch (error) {
            if (error.name === 'QuotaExceededError') {
                console.error('Превышено ограничение хранилища');
                this.clearExpired(); // Очистка устаревших данных
            } else {
                throw error;
            }
        }
    }
    
    // Получение данных с проверкой срока годности
    get(key) {
        const itemStr = localStorage.getItem(this.getKey(key));
        
        if (!itemStr) {
            return null;
        }
        
        try {
            const item = JSON.parse(itemStr);
            const now = Date.now();
            
            // Проверка срока годности
            if (item.ttl && (now - item.timestamp) > item.ttl) {
                this.remove(key); // Удаление устаревшего элемента
                return null;
            }
            
            return JSON.parse(item.value);
        } catch (error) {
            console.error('Ошибка при чтении из хранилища:', error);
            return null;
        }
    }
    
    // Удаление данных
    remove(key) {
        localStorage.removeItem(this.getKey(key));
    }
    
    // Очистка устаревших данных
    clearExpired() {
        const keysToRemove = [];
        
        for (let i = 0; i < localStorage.length; i++) {
            const key = localStorage.key(i);
            if (key.startsWith(this.namespace)) {
                const itemStr = localStorage.getItem(key);
                
                try {
                    const item = JSON.parse(itemStr);
                    if (item.ttl && (Date.now() - item.timestamp) > item.ttl) {
                        keysToRemove.push(key);
                    }
                } catch (error) {
                    // Удаление поврежденных записей
                    keysToRemove.push(key);
                }
            }
        }
        
        keysToRemove.forEach(key => localStorage.removeItem(key));
    }
    
    // Очистка всего пространства имен
    clear() {
        const keysToRemove = [];
        
        for (let i = 0; i < localStorage.length; i++) {
            const key = localStorage.key(i);
            if (key.startsWith(this.namespace)) {
                keysToRemove.push(key);
            }
        }
        
        keysToRemove.forEach(key => localStorage.removeItem(key));
    }
    
    // Получение всех ключей в пространстве имен
    getKeys() {
        const keys = [];
        
        for (let i = 0; i < localStorage.length; i++) {
            const key = localStorage.key(i);
            if (key.startsWith(this.namespace)) {
                keys.push(key.substring(this.namespace.length));
            }
        }
        
        return keys;
    }
}

// Использование
const storage = new LocalStorageManager('myApp');

// Сохранение данных с TTL (1 час)
storage.set('userPreferences', { theme: 'dark', lang: 'ru' }, 60 * 60 * 1000);

// Получение данных
const prefs = storage.get('userPreferences');
console.log(prefs); // { theme: 'dark', lang: 'ru' } или null если устарело
```

## Практические примеры

### Сохранение настроек приложения

```javascript
class AppSettings {
    constructor() {
        this.storage = new LocalStorageManager('appSettings');
    }
    
    // Загрузка настроек
    load() {
        const defaults = {
            theme: 'light',
            language: 'en',
            notifications: true,
            fontSize: 'medium'
        };
        
        const settings = this.storage.get('settings');
        return settings ? { ...defaults, ...settings } : defaults;
    }
    
    // Сохранение настроек
    save(settings) {
        this.storage.set('settings', settings);
        // Оповещение об изменении настроек
        window.dispatchEvent(new CustomEvent('settingsChanged', { detail: settings }));
    }
    
    // Обновление отдельного параметра
    update(key, value) {
        const currentSettings = this.load();
        currentSettings[key] = value;
        this.save(currentSettings);
    }
}

// Использование
const appSettings = new AppSettings();
const settings = appSettings.load();

// Применение настроек
document.body.className = settings.theme;
document.documentElement.lang = settings.language;

// Обновление настроек
appSettings.update('theme', 'dark');
```

### Кэширование данных

```javascript
class DataCache {
    constructor(ttl = 5 * 60 * 1000) { // 5 минут по умолчанию
        this.storage = new LocalStorageManager('dataCache');
        this.ttl = ttl;
    }
    
    async getOrFetch(key, fetchFunction) {
        // Попытка получить данные из кэша
        const cached = this.storage.get(key);
        
        if (cached) {
            console.log('Данные получены из кэша');
            return cached;
        }
        
        // Если данных нет в кэше, получаем их
        try {
            console.log('Загрузка данных из API');
            const data = await fetchFunction();
            
            // Сохраняем в кэш
            this.storage.set(key, data, this.ttl);
            
            return data;
        } catch (error) {
            console.error('Ошибка при загрузке данных:', error);
            throw error;
        }
    }
    
    invalidate(key) {
        this.storage.remove(key);
    }
    
    clear() {
        this.storage.clear();
    }
}

// Использование
const cache = new DataCache(10 * 60 * 1000); // 10 минут

const userData = await cache.getOrFetch('userProfile', async () => {
    const response = await fetch('/api/user/profile');
    return response.json();
});
```

## Ограничения и особенности

### Обработка ошибок и ограничений

```javascript
class SafeStorage {
    static isSupported(type = 'localStorage') {
        try {
            const storage = window[type];
            const x = '__storage_test__';
            storage.setItem(x, x);
            storage.removeItem(x);
            return true;
        } catch (e) {
            return false;
        }
    }
    
    static getStorage(type = 'localStorage') {
        if (this.isSupported(type)) {
            return window[type];
        }
        return null;
    }
    
    static set(key, value, type = 'localStorage') {
        const storage = this.getStorage(type);
        if (!storage) {
            console.warn(`${type} не поддерживается`);
            return false;
        }
        
        try {
            storage.setItem(key, JSON.stringify(value));
            return true;
        } catch (error) {
            if (error.name === 'QuotaExceededError') {
                console.error('Превышено ограничение хранилища');
                this.clearOldEntries(storage);
            } else {
                console.error('Ошибка при записи в хранилище:', error);
            }
            return false;
        }
    }
    
    static get(key, type = 'localStorage') {
        const storage = this.getStorage(type);
        if (!storage) {
            return null;
        }
        
        try {
            const item = storage.getItem(key);
            return item ? JSON.parse(item) : null;
        } catch (error) {
            console.error('Ошибка при чтении из хранилища:', error);
            return null;
        }
    }
    
    static clearOldEntries(storage) {
        // Удаление самых старых записей до освобождения места
        const keys = [];
        for (let i = 0; i < storage.length; i++) {
            keys.push(storage.key(i));
        }
        
        // Сортировка по времени (предполагаем, что используем формат с меткой времени)
        keys.sort((a, b) => {
            try {
                const aTime = JSON.parse(storage.getItem(a)).timestamp || 0;
                const bTime = JSON.parse(storage.getItem(b)).timestamp || 0;
                return aTime - bTime;
            } catch (e) {
                return 0;
            }
        });
        
        // Удаление старых записей
        for (const key of keys) {
            try {
                storage.removeItem(key);
                // Попробовать записать снова
                break;
            } catch (e) {
                continue;
            }
        }
    }
}

// Проверка поддержки
if (!SafeStorage.isSupported('localStorage')) {
    console.warn('localStorage не поддерживается, используем альтернативное хранилище');
}
```

## Совместимость и полифилы

### Полифил для старых браузеров

```javascript
// Полифил для localStorage/sessionStorage
(function(window) {
    if (typeof Storage !== 'undefined') {
        return; // Хранилище уже поддерживается
    }
    
    // Использование userData для IE
    if (document.documentElement.addBehavior) {
        document.documentElement.addBehavior('#default#userData');
        
        const userData = document.documentElement;
        
        window.localStorage = {
            getItem: function(key) {
                userData.load('localStorage');
                return userData.getAttribute(key);
            },
            setItem: function(key, value) {
                userData.setAttribute(key, value);
                userData.save('localStorage');
            },
            removeItem: function(key) {
                userData.removeAttribute(key);
                userData.save('localStorage');
            },
            clear: function() {
                userData.load('localStorage');
                const attrs = userData.XMLDocument.documentElement.attributes;
                for (let i = attrs.length - 1; i >= 0; i--) {
                    userData.removeAttribute(attrs[i].name);
                }
                userData.save('localStorage');
            },
            key: function(i) {
                userData.load('localStorage');
                return userData.XMLDocument.documentElement.attributes[i] ? 
                    userData.XMLDocument.documentElement.attributes[i].name : null;
            },
            get length() {
                userData.load('localStorage');
                return userData.XMLDocument.documentElement.attributes.length;
            }
        };
    }
})(window);
```

## Лучшие практики

### 1. Организация данных

```javascript
// Использование пространств имен
const StorageKeys = {
    USER: {
        PROFILE: 'user_profile',
        PREFERENCES: 'user_preferences',
        TOKEN: 'auth_token'
    },
    APP: {
        SETTINGS: 'app_settings',
        CACHE: 'app_cache',
        TEMP: 'temp_data'
    }
};

// Централизованное управление
class AppStorage {
    static set(key, value) {
        localStorage.setItem(key, JSON.stringify(value));
    }
    
    static get(key) {
        const item = localStorage.getItem(key);
        return item ? JSON.parse(item) : null;
    }
    
    static remove(key) {
        localStorage.removeItem(key);
    }
    
    static clearUser() {
        localStorage.removeItem(StorageKeys.USER.PROFILE);
        localStorage.removeItem(StorageKeys.USER.PREFERENCES);
        localStorage.removeItem(StorageKeys.USER.TOKEN);
    }
}
```

### 2. Обработка синхронизации

```javascript
// Синхронизация данных между вкладками
class StorageSync {
    constructor() {
        this.callbacks = new Map();
        window.addEventListener('storage', this.handleStorageChange.bind(this));
    }
    
    subscribe(key, callback) {
        if (!this.callbacks.has(key)) {
            this.callbacks.set(key, []);
        }
        this.callbacks.get(key).push(callback);
    }
    
    unsubscribe(key, callback) {
        if (this.callbacks.has(key)) {
            const index = this.callbacks.get(key).indexOf(callback);
            if (index > -1) {
                this.callbacks.get(key).splice(index, 1);
            }
        }
    }
    
    handleStorageChange(event) {
        if (this.callbacks.has(event.key)) {
            const callbacks = this.callbacks.get(event.key);
            callbacks.forEach(callback => callback(event.newValue, event.oldValue));
        }
    }
}

// Использование
const sync = new StorageSync();
sync.subscribe('theme', (newValue) => {
    document.body.className = newValue || 'light';
});
```

localStorage и sessionStorage предоставляют простой способ хранения данных на стороне клиента. При правильном использовании они могут значительно улучшить пользовательский опыт за счет сохранения настроек, кэширования данных и обеспечения автономной работы.

#javascript #storage #localstorage #sessionstorage #frontend #web-development
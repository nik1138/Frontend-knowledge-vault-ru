---
aliases: ["LocalStorage", "SessionStorage", "Web Storage API"]
tags: [js, storage, browser, practical-examples]
---

# Работа с LocalStorage и SessionStorage

## Введение в Web Storage API

Web Storage API предоставляет возможность хранить данные в браузере без их передачи на сервер. Существует два типа хранилища:

- **LocalStorage** - данные сохраняются до тех пор, пока пользователь не очистит их вручную
- **SessionStorage** - данные сохраняются только в течение сессии вкладки

## Основные операции

### Проверка поддержки

```javascript
function isStorageAvailable(type) {
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

// Проверка доступности
if (isStorageAvailable('localStorage')) {
  console.log('LocalStorage доступен');
} else {
  console.log('LocalStorage недоступен');
}

if (isStorageAvailable('sessionStorage')) {
  console.log('SessionStorage доступен');
} else {
  console.log('SessionStorage недоступен');
}
```

### Базовые операции

```javascript
// Сохранение данных
localStorage.setItem('username', 'john_doe');
sessionStorage.setItem('sessionId', 'abc123');

// Получение данных
const username = localStorage.getItem('username');
const sessionId = sessionStorage.getItem('sessionId');

// Удаление данных
localStorage.removeItem('username');
sessionStorage.removeItem('sessionId');

// Очистка всего хранилища
localStorage.clear();
sessionStorage.clear();

// Получение ключа по индексу
const key = localStorage.key(0); // первый ключ в localStorage
const length = localStorage.length; // количество элементов
```

## Расширенный класс для работы с хранилищем

```javascript
class StorageManager {
  constructor(type = 'localStorage') {
    this.storage = window[type];
    this.type = type;
    
    if (!this.storage) {
      throw new Error(`${type} не поддерживается в этом браузере`);
    }
  }
  
  // Сохранение данных с автоматической сериализацией
  set(key, value) {
    try {
      const serializedValue = JSON.stringify(value);
      this.storage.setItem(key, serializedValue);
      return true;
    } catch (error) {
      console.error(`Ошибка сохранения в ${this.type}:`, error);
      this.handleQuotaExceeded(error);
      return false;
    }
  }
  
  // Получение данных с автоматической десериализацией
  get(key, defaultValue = null) {
    try {
      const item = this.storage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error(`Ошибка получения из ${this.type}:`, error);
      return defaultValue;
    }
  }
  
  // Удаление данных
  remove(key) {
    this.storage.removeItem(key);
  }
  
  // Проверка наличия ключа
  has(key) {
    return this.storage.getItem(key) !== null;
  }
  
  // Получение всех ключей
  getKeys() {
    const keys = [];
    for (let i = 0; i < this.storage.length; i++) {
      keys.push(this.storage.key(i));
    }
    return keys;
  }
  
  // Получение размера хранилища
  getSize() {
    let total = 0;
    for (let i = 0; i < this.storage.length; i++) {
      const key = this.storage.key(i);
      const value = this.storage.getItem(key);
      total += key.length + value.length;
    }
    return total; // в байтах
  }
  
  // Получение свободного места
  getFreeSpace() {
    const maxSize = 5 * 1024 * 1024; // 5MB для большинства браузеров
    return maxSize - this.getSize();
  }
  
  // Обработка превышения квоты
  handleQuotaExceeded(error) {
    if (error instanceof DOMException) {
      if (error.name === 'QuotaExceededError') {
        console.warn(`Хранилище ${this.type} переполнено`);
        // Можно реализовать стратегию очистки старых данных
        this.cleanupOldEntries();
      }
    }
  }
  
  // Очистка устаревших записей
  cleanupOldEntries() {
    const now = Date.now();
    const keysToRemove = [];
    
    for (let i = 0; i < this.storage.length; i++) {
      const key = this.storage.key(i);
      if (key && key.startsWith('exp_')) { // Предполагаем, что устаревшие ключи начинаются с 'exp_'
        const item = this.get(key);
        if (item && item.expires && item.expires < now) {
          keysToRemove.push(key);
        }
      }
    }
    
    keysToRemove.forEach(key => this.remove(key));
  }
  
  // Сохранение с TTL (временем жизни)
  setWithExpiry(key, value, ttlMs) {
    const item = {
      value: value,
      expiry: Date.now() + ttlMs
    };
    return this.set(key, item);
  }
  
  // Получение с проверкой TTL
  getWithExpiry(key) {
    const item = this.get(key);
    if (!item) return null;
    
    const now = Date.now();
    if (item.expiry && item.expiry < now) {
      this.remove(key);
      return null;
    }
    
    return item.value;
  }
}

// Создание менеджеров для обоих типов хранилища
const localStorageManager = new StorageManager('localStorage');
const sessionStorageManager = new StorageManager('sessionStorage');
```

## Практические примеры использования

### Хранение настроек пользователя

```javascript
class UserPreferences {
  constructor() {
    this.storage = new StorageManager('localStorage');
    this.key = 'userPreferences';
    this.defaults = {
      theme: 'light',
      language: 'ru',
      notifications: true,
      fontSize: 14,
      lastVisit: null
    };
  }
  
  // Загрузка настроек
  load() {
    const saved = this.storage.get(this.key);
    this.preferences = { ...this.defaults, ...saved };
    return this.preferences;
  }
  
  // Сохранение настроек
  save() {
    this.storage.set(this.key, this.preferences);
  }
  
  // Получение конкретной настройки
  get(key) {
    this.load(); // Убедиться, что настройки загружены
    return this.preferences[key];
  }
  
  // Установка настройки
  set(key, value) {
    this.load();
    this.preferences[key] = value;
    this.save();
  }
  
  // Сброс настроек
  reset() {
    this.preferences = { ...this.defaults };
    this.save();
  }
  
  // Обновление нескольких настроек
  update(updates) {
    this.load();
    Object.assign(this.preferences, updates);
    this.save();
  }
}

// Использование
const userPrefs = new UserPreferences();

// Установка темы
userPrefs.set('theme', 'dark');

// Получение языка
const language = userPrefs.get('language');

// Обновление нескольких настроек
userPrefs.update({
  fontSize: 16,
  notifications: false
});
```

### Хранение данных формы

```javascript
class FormPersistence {
  constructor(formId, storageType = 'sessionStorage') {
    this.formId = formId;
    this.storage = new StorageManager(storageType);
    this.storageKey = `form_${formId}`;
    this.form = document.getElementById(formId);
    
    if (this.form) {
      this.loadFormData();
      this.setupAutoSave();
    }
  }
  
  // Загрузка данных формы
  loadFormData() {
    const formData = this.storage.get(this.storageKey, {});
    
    Object.keys(formData).forEach(fieldName => {
      const field = this.form.querySelector(`[name="${fieldName}"]`);
      if (field) {
        if (field.type === 'checkbox' || field.type === 'radio') {
          field.checked = formData[fieldName];
        } else {
          field.value = formData[fieldName];
        }
      }
    });
  }
  
  // Сохранение данных формы
  saveFormData() {
    const formData = {};
    const fields = this.form.querySelectorAll('input, textarea, select');
    
    fields.forEach(field => {
      if (field.type === 'checkbox' || field.type === 'radio') {
        formData[field.name] = field.checked;
      } else {
        formData[field.name] = field.value;
      }
    });
    
    this.storage.set(this.storageKey, formData);
  }
  
  // Настройка автосохранения
  setupAutoSave() {
    const fields = this.form.querySelectorAll('input, textarea, select');
    
    fields.forEach(field => {
      field.addEventListener('input', () => {
        this.saveFormData();
      });
      
      field.addEventListener('change', () => {
        this.saveFormData();
      });
    });
  }
  
  // Очистка сохраненных данных
  clear() {
    this.storage.remove(this.storageKey);
  }
  
  // Восстановление формы из сохраненных данных
  restore() {
    this.loadFormData();
  }
}

// Использование
const contactFormPersistence = new FormPersistence('contactForm');

// Ручное сохранение
document.getElementById('saveDraft').addEventListener('click', () => {
  contactFormPersistence.saveFormData();
  alert('Черновик сохранен!');
});
```

### Кэширование API данных

```javascript
class ApiCache {
  constructor(ttlMinutes = 10) {
    this.storage = new StorageManager('localStorage');
    this.ttlMs = ttlMinutes * 60 * 1000;
  }
  
  // Генерация ключа для кэша
  generateKey(url, params = {}) {
    const paramString = Object.keys(params)
      .sort()
      .map(key => `${key}=${params[key]}`)
      .join('&');
    
    return `api_cache_${url}_${btoa(paramString)}`;
  }
  
  // Получение данных из кэша
  get(url, params = {}) {
    const key = this.generateKey(url, params);
    return this.storage.getWithExpiry(key);
  }
  
  // Сохранение данных в кэш
  set(url, params = {}, data) {
    const key = this.generateKey(url, params);
    return this.storage.setWithExpiry(key, data, this.ttlMs);
  }
  
  // Удаление из кэша
  remove(url, params = {}) {
    const key = this.generateKey(url, params);
    this.storage.remove(key);
  }
  
  // Очистка устаревшего кэша
  cleanup() {
    const keys = this.storage.getKeys();
    const expiredKeys = keys.filter(key => 
      key.startsWith('api_cache_') && !this.storage.getWithExpiry(key)
    );
    
    expiredKeys.forEach(key => this.storage.remove(key));
  }
}

// Класс для работы с API с кэшированием
class CachedApiClient {
  constructor() {
    this.cache = new ApiCache(5); // 5 минут TTL
  }
  
  async get(url, params = {}) {
    // Проверяем кэш
    const cachedData = this.cache.get(url, params);
    if (cachedData) {
      console.log('Данные взяты из кэша:', url);
      return cachedData;
    }
    
    // Делаем запрос
    try {
      const response = await fetch(url, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ошибка! статус: ${response.status}`);
      }
      
      const data = await response.json();
      
      // Сохраняем в кэш
      this.cache.set(url, params, data);
      
      return data;
    } catch (error) {
      console.error('Ошибка API запроса:', error);
      throw error;
    }
  }
  
  // Метод для очистки кэша
  clearCache() {
    this.cache.cleanup();
  }
}

// Использование
const apiClient = new CachedApiClient();

async function loadUserData(userId) {
  const userData = await apiClient.get(`/api/users/${userId}`);
  return userData;
}
```

## События изменения хранилища

```javascript
// Обработка событий изменения хранилища
class StorageWatcher {
  constructor() {
    this.listeners = [];
    this.setupListener();
  }
  
  setupListener() {
    window.addEventListener('storage', (event) => {
      this.listeners.forEach(callback => {
        callback({
          key: event.key,
          oldValue: event.oldValue,
          newValue: event.newValue,
          url: event.url,
          storageArea: event.storageArea
        });
      });
    });
  }
  
  // Подписка на изменения
  onChange(callback) {
    this.listeners.push(callback);
    return () => {
      this.listeners = this.listeners.filter(listener => listener !== callback);
    };
  }
  
  // Подписка на изменения конкретного ключа
  onChangeForKey(key, callback) {
    return this.onChange((event) => {
      if (event.key === key) {
        callback(event);
      }
    });
  }
}

// Использование
const storageWatcher = new StorageWatcher();

// Общая подписка
const unsubscribe = storageWatcher.onChange((event) => {
  console.log('Хранилище изменено:', event);
});

// Подписка на конкретный ключ
storageWatcher.onChangeForKey('userPreferences', (event) => {
  console.log('Настройки пользователя изменены');
  // Можно обновить UI или выполнить другие действия
});
```

## Безопасность и защита данных

### Шифрование данных

```javascript
class SecureStorage {
  constructor(encryptionKey) {
    this.storage = new StorageManager('localStorage');
    this.encryptionKey = encryptionKey;
  }
  
  // Простое шифрование (в реальных приложениях используйте криптографически стойкие методы)
  encrypt(data) {
    try {
      const jsonString = JSON.stringify(data);
      // Простое XOR шифрование (только для демонстрации)
      let encrypted = '';
      for (let i = 0; i < jsonString.length; i++) {
        encrypted += String.fromCharCode(
          jsonString.charCodeAt(i) ^ this.encryptionKey.charCodeAt(i % this.encryptionKey.length)
        );
      }
      return btoa(encrypted); // кодируем в base64
    } catch (error) {
      console.error('Ошибка шифрования:', error);
      return null;
    }
  }
  
  decrypt(encryptedData) {
    try {
      const decoded = atob(encryptedData); // декодируем из base64
      let decrypted = '';
      for (let i = 0; i < decoded.length; i++) {
        decrypted += String.fromCharCode(
          decoded.charCodeAt(i) ^ this.encryptionKey.charCodeAt(i % this.encryptionKey.length)
        );
      }
      return JSON.parse(decrypted);
    } catch (error) {
      console.error('Ошибка расшифровки:', error);
      return null;
    }
  }
  
  set(key, value) {
    const encrypted = this.encrypt(value);
    if (encrypted) {
      return this.storage.set(`encrypted_${key}`, encrypted);
    }
    return false;
  }
  
  get(key, defaultValue = null) {
    const encrypted = this.storage.get(`encrypted_${key}`);
    if (encrypted) {
      return this.decrypt(encrypted) || defaultValue;
    }
    return defaultValue;
  }
  
  remove(key) {
    this.storage.remove(`encrypted_${key}`);
  }
}

// Использование (в реальных приложениях используйте криптографически стойкие методы!)
const secureStorage = new SecureStorage('mySecretKey');
secureStorage.set('sensitiveData', { token: 'secret-token', userId: 123 });
const data = secureStorage.get('sensitiveData');
```

### Валидация данных

```javascript
class ValidatedStorage {
  constructor() {
    this.storage = new StorageManager('localStorage');
    this.validators = new Map();
  }
  
  // Добавление валидатора для ключа
  addValidator(key, validatorFn, errorMessage) {
    this.validators.set(key, {
      validate: validatorFn,
      message: errorMessage
    });
  }
  
  // Проверка данных перед сохранением
  set(key, value) {
    const validator = this.validators.get(key);
    if (validator) {
      if (!validator.validate(value)) {
        console.error(`Валидация не пройдена для ключа ${key}: ${validator.message}`);
        return false;
      }
    }
    
    return this.storage.set(key, value);
  }
  
  // Получение данных с валидацией
  get(key, defaultValue = null) {
    const value = this.storage.get(key, defaultValue);
    const validator = this.validators.get(key);
    
    if (validator && value !== defaultValue) {
      if (!validator.validate(value)) {
        console.error(`Сохраненные данные не прошли валидацию: ${validator.message}`);
        return defaultValue;
      }
    }
    
    return value;
  }
}

// Пример использования
const validatedStorage = new ValidatedStorage();

// Валидация email
validatedStorage.addValidator(
  'userEmail',
  value => typeof value === 'string' && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  'Некорректный email'
);

// Валидация массива
validatedStorage.addValidator(
  'userFavorites',
  value => Array.isArray(value) && value.length <= 100,
  'Массив избранных элементов должен содержать не более 100 элементов'
);

validatedStorage.set('userEmail', 'user@example.com'); // OK
validatedStorage.set('userEmail', 'invalid-email'); // Ошибка валидации
```

## Практические паттерны

### Менеджер сессии

```javascript
class SessionManager {
  constructor() {
    this.storage = new StorageManager('sessionStorage');
    this.sessionKey = 'userSession';
  }
  
  // Создание сессии
  create(userData) {
    const session = {
      ...userData,
      createdAt: Date.now(),
      expiresAt: Date.now() + (30 * 60 * 1000) // 30 минут
    };
    
    this.storage.set(this.sessionKey, session);
    return session;
  }
  
  // Получение сессии
  get() {
    const session = this.storage.get(this.sessionKey);
    
    if (!session) {
      return null;
    }
    
    // Проверка срока действия
    if (session.expiresAt < Date.now()) {
      this.destroy();
      return null;
    }
    
    return session;
  }
  
  // Обновление сессии (продление)
  refresh() {
    const session = this.get();
    if (session) {
      session.expiresAt = Date.now() + (30 * 60 * 1000); // Продлеваем на 30 минут
      this.storage.set(this.sessionKey, session);
    }
    return session;
  }
  
  // Уничтожение сессии
  destroy() {
    this.storage.remove(this.sessionKey);
  }
  
  // Проверка активности сессии
  isActive() {
    return this.get() !== null;
  }
  
  // Получение данных пользователя из сессии
  getUserData() {
    const session = this.get();
    return session ? { id: session.id, name: session.name } : null;
  }
}

// Использование
const sessionManager = new SessionManager();

// Создание сессии
sessionManager.create({
  id: 123,
  name: 'John Doe',
  role: 'user'
});

// Проверка сессии
if (sessionManager.isActive()) {
  const userData = sessionManager.getUserData();
  console.log('Пользователь:', userData);
}
```

### Очередь событий

```javascript
class EventQueue {
  constructor(maxSize = 100) {
    this.storage = new StorageManager('localStorage');
    this.queueKey = 'eventQueue';
    this.maxSize = maxSize;
  }
  
  // Добавление события в очередь
  add(event) {
    const queue = this.getQueue();
    queue.push({
      ...event,
      timestamp: Date.now(),
      id: Date.now() + Math.random()
    });
    
    // Ограничение размера очереди
    if (queue.length > this.maxSize) {
      queue.splice(0, queue.length - this.maxSize);
    }
    
    return this.storage.set(this.queueKey, queue);
  }
  
  // Получение очереди
  getQueue() {
    return this.storage.get(this.queueKey, []);
  }
  
  // Получение и удаление всех событий
  flush() {
    const queue = this.getQueue();
    this.storage.remove(this.queueKey);
    return queue;
  }
  
  // Получение и удаление одного события
  pop() {
    const queue = this.getQueue();
    if (queue.length === 0) {
      return null;
    }
    
    const event = queue.shift();
    this.storage.set(this.queueKey, queue);
    return event;
  }
  
  // Очистка очереди
  clear() {
    this.storage.remove(this.queueKey);
  }
  
  // Размер очереди
  size() {
    return this.getQueue().length;
  }
}

// Использование
const eventQueue = new EventQueue();

// Добавление событий
eventQueue.add({ type: 'USER_LOGIN', userId: 123 });
eventQueue.add({ type: 'PAGE_VIEW', url: '/dashboard' });

// Обработка очереди
function processEvents() {
  const events = eventQueue.flush();
  events.forEach(event => {
    console.log('Обработка события:', event);
    // Отправка событий на сервер аналитики
  });
}
```

## Лучшие практики

### Обработка ошибок и резервное хранилище

```javascript
class ResilientStorage {
  constructor() {
    this.primary = new StorageManager('localStorage');
    this.secondary = new StorageManager('sessionStorage');
    this.fallback = new Map(); // In-memory fallback
  }
  
  set(key, value) {
    // Пробуем сохранить в primary
    if (!this.primary.set(key, value)) {
      // Если не удалось, пробуем secondary
      if (!this.secondary.set(key, value)) {
        // Если и secondary не работает, используем in-memory
        this.fallback.set(key, JSON.stringify(value));
      }
    }
  }
  
  get(key, defaultValue = null) {
    // Пробуем получить из primary
    let value = this.primary.get(key);
    if (value !== null) return value;
    
    // Если нет в primary, пробуем secondary
    value = this.secondary.get(key);
    if (value !== null) return value;
    
    // Если нет в secondary, пробуем in-memory
    const fallbackValue = this.fallback.get(key);
    if (fallbackValue) {
      try {
        return JSON.parse(fallbackValue);
      } catch (e) {
        console.error('Ошибка парсинга in-memory значения:', e);
      }
    }
    
    return defaultValue;
  }
  
  remove(key) {
    this.primary.remove(key);
    this.secondary.remove(key);
    this.fallback.delete(key);
  }
}

// Использование
const resilientStorage = new ResilientStorage();
resilientStorage.set('importantData', { status: 'active', timestamp: Date.now() });
const data = resilientStorage.get('importantData');
```

## Ссылки по теме

- [[js/storage/localstorage-api]] - LocalStorage API
- [[js/storage/sessionstorage-api]] - SessionStorage API
- [[js/security/storage-security]] - Безопасность Web Storage
- [[js/patterns/cache-strategy]] - Стратегии кэширования
---
aliases: ["Заготовки кода", "JS Snippets", "Code Templates"]
tags: [js, snippets, templates, utilities]
---

# Заготовки кода для частых задач

## Введение

Этот раздел содержит готовые заготовки кода для решения частых задач в JavaScript. Эти сниппеты можно использовать как есть или адаптировать под конкретные нужды.

## Универсальные функции

### 1. Работа с массивами

```javascript
// Уникальные значения из массива
const getUnique = (arr) => [...new Set(arr)];

// Группировка массива по свойству
const groupBy = (array, key) => {
  return array.reduce((result, item) => {
    const group = item[key];
    if (!result[group]) {
      result[group] = [];
    }
    result[group].push(item);
    return result;
  }, {});
};

// Разделение массива на чанки
const chunk = (array, size) => {
  return array.reduce((chunks, item, index) => {
    const chunkIndex = Math.floor(index / size);
    if (!chunks[chunkIndex]) {
      chunks[chunkIndex] = [];
    }
    chunks[chunkIndex].push(item);
    return chunks;
  }, []);
};

// Перемешивание массива (алгоритм Фишера-Йетса)
const shuffle = (array) => {
  const result = [...array];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
};

// Удаление дубликатов объектов по ключу
const uniqueBy = (array, key) => {
  const seen = new Set();
  return array.filter(item => {
    const value = item[key];
    if (seen.has(value)) {
      return false;
    }
    seen.add(value);
    return true;
  });
};

// Поиск элемента в массиве объектов
const findInArray = (array, searchParams) => {
  return array.find(item => 
    Object.entries(searchParams).every(([key, value]) => item[key] === value)
  );
};
```

### 2. Работа с объектами

```javascript
// Глубокое объединение объектов
const deepMerge = (target, source) => {
  const result = { ...target };
  
  for (const key in source) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      result[key] = deepMerge(result[key] || {}, source[key]);
    } else {
      result[key] = source[key];
    }
  }
  
  return result;
};

// Выбор подмножества свойств объекта
const pick = (obj, keys) => {
  return keys.reduce((result, key) => {
    if (key in obj) {
      result[key] = obj[key];
    }
    return result;
  }, {});
};

// Исключение свойств из объекта
const omit = (obj, keys) => {
  const result = { ...obj };
  keys.forEach(key => delete result[key]);
  return result;
};

// Преобразование ключей объекта
const mapKeys = (obj, mapper) => {
  return Object.keys(obj).reduce((result, key) => {
    const newKey = mapper(key, obj[key]);
    result[newKey] = obj[key];
    return result;
  }, {});
};

// Преобразование значений объекта
const mapValues = (obj, mapper) => {
  return Object.keys(obj).reduce((result, key) => {
    result[key] = mapper(obj[key], key);
    return result;
  }, {});
};

// Глубокое копирование объекта
const deepClone = (obj) => {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(item => deepClone(item));
  if (typeof obj === 'object') {
    const cloned = {};
    for (let key in obj) {
      if (obj.hasOwnProperty(key)) {
        cloned[key] = deepClone(obj[key]);
      }
    }
    return cloned;
  }
};
```

### 3. Валидация данных

```javascript
// Валидация email
const isValidEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// Валидация телефона
const isValidPhone = (phone) => {
  const phoneRegex = /^(\+7|7|8)?[\s-]?\(?[0-9]{3}\)?[\s-]?[0-9]{3}[\s-]?[0-9]{2}[\s-]?[0-9]{2}$/;
  return phoneRegex.test(phone);
};

// Валидация пароля
const isValidPassword = (password) => {
  const checks = {
    length: password.length >= 8,
    hasUpper: /[A-Z]/.test(password),
    hasLower: /[a-z]/.test(password),
    hasNumber: /\d/.test(password),
    hasSpecial: /[!@#$%^&*(),.?":{}|<>]/.test(password)
  };
  
  return Object.values(checks).every(check => check);
};

// Проверка на пустой объект/массив
const isEmpty = (value) => {
  if (Array.isArray(value)) return value.length === 0;
  if (typeof value === 'object' && value !== null) return Object.keys(value).length === 0;
  if (typeof value === 'string') return value.trim() === '';
  return !value;
};

// Проверка на простой объект
const isPlainObject = (obj) => {
  return obj != null && 
         typeof obj === 'object' && 
         obj.constructor === Object &&
         Object.getPrototypeOf(obj) === Object.prototype;
};
```

## DOM манипуляции

### 4. Создание и управление DOM элементами

```javascript
// Быстрое создание элемента
const createElement = (tag, props = {}, children = []) => {
  const element = document.createElement(tag);
  
  // Установка атрибутов
  Object.entries(props).forEach(([key, value]) => {
    if (key.startsWith('on')) {
      // Обработчики событий
      element.addEventListener(key.slice(2).toLowerCase(), value);
    } else if (key === 'className') {
      element.className = value;
    } else if (key === 'textContent') {
      element.textContent = value;
    } else if (key === 'style' && typeof value === 'object') {
      Object.assign(element.style, value);
    } else {
      element.setAttribute(key, value);
    }
  });
  
  // Добавление детей
  children.forEach(child => {
    element.appendChild(
      typeof child === 'string' ? document.createTextNode(child) : child
    );
  });
  
  return element;
};

// Поиск элементов с проверкой
const findElement = (selector, parent = document) => {
  const element = parent.querySelector(selector);
  if (!element) {
    throw new Error(`Элемент не найден: ${selector}`);
  }
  return element;
};

// Проверка видимости элемента
const isElementVisible = (element) => {
  const rect = element.getBoundingClientRect();
  const windowHeight = window.innerHeight || document.documentElement.clientHeight;
  const windowWidth = window.innerWidth || document.documentElement.clientWidth;
  
  return (
    rect.top >= 0 &&
    rect.left >= 0 &&
    rect.bottom <= windowHeight &&
    rect.right <= windowWidth
  );
};

// Добавление класса с задержкой
const addClassWithDelay = (element, className, delay = 0) => {
  setTimeout(() => {
    element.classList.add(className);
  }, delay);
};

// Удаление класса с задержкой
const removeClassWithDelay = (element, className, delay = 0) => {
  setTimeout(() => {
    element.classList.remove(className);
  }, delay);
};
```

### 5. Работа с формами

```javascript
// Сбор данных формы
const getFormData = (formElement) => {
  const formData = new FormData(formElement);
  const data = {};
  
  for (let [key, value] of formData.entries()) {
    // Обработка множественных значений
    if (data[key]) {
      if (Array.isArray(data[key])) {
        data[key].push(value);
      } else {
        data[key] = [data[key], value];
      }
    } else {
      data[key] = value;
    }
  }
  
  return data;
};

// Установка данных формы
const setFormData = (formElement, data) => {
  Object.entries(data).forEach(([name, value]) => {
    const field = formElement.querySelector(`[name="${name}"]`);
    if (field) {
      if (field.type === 'checkbox' || field.type === 'radio') {
        field.checked = Array.isArray(value) 
          ? value.includes(field.value)
          : field.value === value;
      } else {
        field.value = value;
      }
    }
  });
};

// Валидация формы
const validateForm = (formElement, rules) => {
  const errors = {};
  
  Object.entries(rules).forEach(([fieldName, validators]) => {
    const field = formElement.querySelector(`[name="${fieldName}"]`);
    if (field) {
      const value = field.value;
      
      for (const validator of validators) {
        if (!validator.validate(value)) {
          if (!errors[fieldName]) {
            errors[fieldName] = [];
          }
          errors[fieldName].push(validator.message);
          break;
        }
      }
    }
  });
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
};
```

## Асинхронные операции

### 6. Утилиты для работы с Promise

```javascript
// Ожидание с таймаутом
const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Таймаут для асинхронной операции
const withTimeout = (promise, timeoutMs) => {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Таймаут операции')), timeoutMs)
    )
  ]);
};

// Повторение операции с экспоненциальной задержкой
const retryAsync = async (asyncFn, maxRetries = 3, baseDelay = 1000) => {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await asyncFn();
    } catch (error) {
      lastError = error;
      
      if (attempt === maxRetries) break;
      
      const delay = baseDelay * Math.pow(2, attempt - 1);
      await wait(delay);
    }
  }
  
  throw lastError;
};

// Параллельное выполнение с ограничением количества одновременных операций
const asyncPool = async (poolLimit, array, iteratorFn) => {
  const ret = [];
  const executing = [];
  
  for (const item of array) {
    const p = Promise.resolve().then(() => iteratorFn(item, array));
    ret.push(p);

    if (ret.length >= poolLimit) {
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e);
      if (executing.length >= poolLimit) {
        await Promise.race(executing);
      }
    }
  }
  
  return Promise.all(ret);
};

// Сериализация асинхронных операций
const serializeAsync = async (operations) => {
  const results = [];
  
  for (const operation of operations) {
    try {
      const result = await operation();
      results.push({ success: true, data: result });
    } catch (error) {
      results.push({ success: false, error });
    }
  }
  
  return results;
};
```

### 7. Работа с API

```javascript
// Универсальный API клиент
class ApiClient {
  constructor(baseURL, defaultOptions = {}) {
    this.baseURL = baseURL;
    this.defaultOptions = defaultOptions;
  }
  
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      ...this.defaultOptions,
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...this.defaultOptions.headers,
        ...options.headers
      }
    };
    
    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка! статус: ${response.status}`);
    }
    
    return await response.json();
  }
  
  get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;
    return this.request(url, { method: 'GET' });
  }
  
  post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
  
  put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
  
  delete(endpoint) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

// Использование
const api = new ApiClient('https://api.example.com');
// const users = await api.get('/users');
// const newUser = await api.post('/users', { name: 'Иван' });
```

## Работа с датами и временем

### 8. Утилиты для работы с датами

```javascript
// Форматирование даты
const formatDate = (date, format = 'DD.MM.YYYY') => {
  const d = new Date(date);
  const day = String(d.getDate()).padStart(2, '0');
  const month = String(d.getMonth() + 1).padStart(2, '0');
  const year = d.getFullYear();
  
  return format
    .replace('DD', day)
    .replace('MM', month)
    .replace('YYYY', year);
};

// Разница между датами
const dateDiff = (date1, date2) => {
  const d1 = new Date(date1);
  const d2 = new Date(date2);
  const diffTime = Math.abs(d2 - d1);
  const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
  
  return {
    milliseconds: diffTime,
    seconds: Math.floor(diffTime / 1000),
    minutes: Math.floor(diffTime / (1000 * 60)),
    hours: Math.floor(diffTime / (1000 * 60 * 60)),
    days: diffDays
  };
};

// Проверка, является ли дата сегодняшней
const isToday = (date) => {
  const today = new Date();
  const checkDate = new Date(date);
  
  return checkDate.getDate() === today.getDate() &&
         checkDate.getMonth() === today.getMonth() &&
         checkDate.getFullYear() === today.getFullYear();
};

// Проверка, является ли дата вчерашней
const isYesterday = (date) => {
  const yesterday = new Date();
  yesterday.setDate(yesterday.getDate() - 1);
  
  const checkDate = new Date(date);
  
  return checkDate.getDate() === yesterday.getDate() &&
         checkDate.getMonth() === yesterday.getMonth() &&
         checkDate.getFullYear() === yesterday.getFullYear();
};

// Получение начала/конца дня
const startOfDay = (date) => {
  const d = new Date(date);
  d.setHours(0, 0, 0, 0);
  return d;
};

const endOfDay = (date) => {
  const d = new Date(date);
  d.setHours(23, 59, 59, 999);
  return d;
};
```

## Утилиты для строк

### 9. Работа со строками

```javascript
// Обрезка строки с добавлением многоточия
const truncateString = (str, maxLength, suffix = '...') => {
  if (str.length <= maxLength) return str;
  return str.substring(0, maxLength - suffix.length) + suffix;
};

// Камель кейс
const toCamelCase = (str) => {
  return str.replace(/(?:^\w|[A-Z]|\b\w)/g, (word, index) => {
    return index === 0 ? word.toLowerCase() : word.toUpperCase();
  }).replace(/\s+/g, '');
};

// Снейк кейс
const toSnakeCase = (str) => {
  return str.replace(/\W+/g, ' ')
    .split(/ |\B(?=[A-Z])/)
    .map(word => word.toLowerCase())
    .join('_');
};

// Кебаб кейс
const toKebabCase = (str) => {
  return str.replace(/\W+/g, ' ')
    .split(/ |\B(?=[A-Z])/)
    .map(word => word.toLowerCase())
    .join('-');
};

// Удаление HTML тегов
const stripHtml = (html) => {
  const tmp = document.createElement('div');
  tmp.textContent = html;
  return tmp.innerHTML;
};

// Экранирование HTML
const escapeHtml = (str) => {
  const escapeMap = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;'
  };
  
  return str.replace(/[&<>"'\/]/g, (match) => escapeMap[match]);
};

// Генерация случайной строки
const generateRandomString = (length = 10) => {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  let result = '';
  
  for (let i = 0; i < length; i++) {
    result += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  
  return result;
};
```

## Хранилище данных

### 10. Работа с LocalStorage

```javascript
// Универсальный менеджер хранилища
class StorageManager {
  constructor(type = 'localStorage') {
    this.storage = window[type];
  }
  
  set(key, value) {
    try {
      const serializedValue = JSON.stringify(value);
      this.storage.setItem(key, serializedValue);
      return true;
    } catch (error) {
      console.error('Ошибка сохранения в хранилище:', error);
      return false;
    }
  }
  
  get(key, defaultValue = null) {
    try {
      const item = this.storage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error('Ошибка получения из хранилища:', error);
      return defaultValue;
    }
  }
  
  remove(key) {
    this.storage.removeItem(key);
  }
  
  has(key) {
    return this.storage.getItem(key) !== null;
  }
  
  clear() {
    this.storage.clear();
  }
  
  // Сохранение с TTL
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

// Использование
const storage = new StorageManager();
// storage.set('user', { name: 'Иван', id: 123 });
// const user = storage.get('user');
```

## Функциональные утилиты

### 11. Функциональные паттерны

```javascript
// Дебаунс
const debounce = (func, delay) => {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
};

// Троттлинг
const throttle = (func, limit) => {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
};

// Memoization
const memoize = (fn) => {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
};

// Композиция функций
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);

// Пайп (цепочка вызовов)
const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);

// Пример использования
const add = (x) => (y) => x + y;
const multiply = (x) => (y) => x * y;
const square = (x) => x * x;

const result = pipe(
  add(5),      // 10 + 5 = 15
  multiply(2), // 15 * 2 = 30
  square       // 30^2 = 900
)(10);
```

## Практические шаблоны

### 12. Шаблон для модального окна

```javascript
class Modal {
  constructor(options = {}) {
    this.options = {
      title: '',
      content: '',
      closable: true,
      onClose: () => {},
      ...options
    };
    
    this.element = this.createModal();
    this.isOpen = false;
  }
  
  createModal() {
    const modal = document.createElement('div');
    modal.className = 'modal';
    modal.innerHTML = `
      <div class="modal-overlay"></div>
      <div class="modal-content">
        <div class="modal-header">
          <h3>${this.options.title}</h3>
          ${this.options.closable ? '<button class="modal-close">&times;</button>' : ''}
        </div>
        <div class="modal-body">${this.options.content}</div>
        <div class="modal-footer">
          ${this.options.footer || ''}
        </div>
      </div>
    `;
    
    // Добавляем обработчики событий
    modal.querySelector('.modal-overlay')?.addEventListener('click', () => this.close());
    modal.querySelector('.modal-close')?.addEventListener('click', () => this.close());
    
    return modal;
  }
  
  open() {
    document.body.appendChild(this.element);
    document.body.style.overflow = 'hidden';
    this.isOpen = true;
  }
  
  close() {
    if (this.element.parentNode) {
      this.element.parentNode.removeChild(this.element);
    }
    document.body.style.overflow = '';
    this.isOpen = false;
    this.options.onClose();
  }
}

// Использование
// const modal = new Modal({
//   title: 'Заголовок модального окна',
//   content: '<p>Содержимое модального окна</p>',
//   footer: '<button onclick="modal.close()">Закрыть</button>'
// });
// modal.open();
```

### 13. Шаблон для табов

```javascript
class Tabs {
  constructor(container, options = {}) {
    this.container = typeof container === 'string' ? document.querySelector(container) : container;
    this.options = {
      activeTab: 0,
      onChange: () => {},
      ...options
    };
    
    this.init();
  }
  
  init() {
    this.tabs = Array.from(this.container.querySelectorAll('[data-tab]'));
    this.panels = Array.from(this.container.querySelectorAll('[data-tab-panel]'));
    
    this.tabs.forEach((tab, index) => {
      tab.addEventListener('click', () => this.setActiveTab(index));
    });
    
    this.setActiveTab(this.options.activeTab);
  }
  
  setActiveTab(index) {
    this.tabs.forEach((tab, i) => {
      tab.classList.toggle('active', i === index);
    });
    
    this.panels.forEach((panel, i) => {
      panel.classList.toggle('active', i === index);
    });
    
    this.options.onChange(index);
  }
}

// HTML структура:
// <div class="tabs-container">
//   <div class="tab-header" data-tab>Вкладка 1</div>
//   <div class="tab-header" data-tab>Вкладка 2</div>
//   <div class="tab-panel" data-tab-panel>Содержимое 1</div>
//   <div class="tab-panel" data-tab-panel>Содержимое 2</div>
// </div>
```

## Ссылки по теме

- [[js/utils]] - Утилиты JavaScript
- [[js/patterns]] - Паттерны JavaScript
- [[js/dom/manipulation]] - DOM манипуляции
- [[js/async]] - Асинхронные операции
- [[js/storage]] - Хранилище данных
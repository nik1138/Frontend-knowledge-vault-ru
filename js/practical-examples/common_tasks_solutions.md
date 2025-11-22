---
tags: [javascript, frontend, practical-examples, common-tasks]
aliases: [распространенные задачи js для фронтенда, решения повседневных задач]
---

# Решение распространенных задач в JavaScript для Frontend разработки

## Введение

В этой статье рассмотрены решения распространенных задач, с которыми сталкиваются фронтенд разработчики при создании веб-приложений. Мы рассмотрим практические примеры и готовые решения для повседневных задач.

## 1. Работа с формами и валидацией

### Динамическая валидация формы в реальном времени

```javascript
// Класс для управления формой с валидацией
class DynamicFormValidator {
  constructor(formSelector) {
    this.form = document.querySelector(formSelector);
    this.fields = {};
    this.validationRules = {};
    this.errorDisplays = {};
    
    if (this.form) {
      this.init();
    }
  }
  
  init() {
    // Инициализация полей формы
    const inputs = this.form.querySelectorAll('input, textarea, select');
    
    inputs.forEach(input => {
      const fieldName = input.name || input.id;
      if (fieldName) {
        this.fields[fieldName] = input;
        
        // Устанавливаем обработчики событий
        input.addEventListener('blur', () => this.validateField(fieldName));
        input.addEventListener('input', () => this.clearFieldError(fieldName));
      }
    });
    
    // Обработчик отправки формы
    this.form.addEventListener('submit', this.handleSubmit.bind(this));
  }
  
  // Установка правил валидации
  setValidationRules(rules) {
    this.validationRules = { ...this.validationRules, ...rules };
  }
  
  // Валидация отдельного поля
  async validateField(fieldName) {
    const field = this.fields[fieldName];
    if (!field) return true;
    
    const value = field.value;
    const rules = this.validationRules[fieldName];
    
    if (!rules || rules.length === 0) return true;
    
    // Проверяем все правила
    for (const rule of rules) {
      const result = await this.executeValidationRule(rule, value, fieldName);
      
      if (!result.isValid) {
        this.showFieldError(fieldName, result.message);
        return false;
      }
    }
    
    this.clearFieldError(fieldName);
    return true;
  }
  
  // Выполнение правила валидации
  async executeValidationRule(rule, value, fieldName) {
    if (typeof rule === 'function') {
      try {
        const result = await Promise.resolve(rule(value, this.getFormData()));
        return typeof result === 'object' ? result : { isValid: Boolean(result), message: 'Некорректное значение' };
      } catch (error) {
        return { isValid: false, message: error.message || 'Ошибка валидации' };
      }
    } else if (rule.test && rule.message) {
      // Объект с регулярным выражением или функцией проверки
      const isValid = typeof rule.test === 'function' ? rule.test(value) : rule.test.test(value);
      return { isValid, message: rule.message };
    }
    
    return { isValid: true, message: '' };
  }
  
  // Показ ошибки поля
  showFieldError(fieldName, message) {
    const field = this.fields[fieldName];
    let errorDisplay = this.errorDisplays[fieldName];
    
    if (!errorDisplay) {
      errorDisplay = document.createElement('div');
      errorDisplay.className = 'field-error';
      errorDisplay.style.color = 'red';
      errorDisplay.style.fontSize = '12px';
      errorDisplay.style.marginTop = '4px';
      
      this.errorDisplays[fieldName] = errorDisplay;
      field.parentNode.insertBefore(errorDisplay, field.nextSibling);
    }
    
    errorDisplay.textContent = message;
    field.classList.add('invalid');
  }
  
  // Очистка ошибки поля
  clearFieldError(fieldName) {
    const field = this.fields[fieldName];
    const errorDisplay = this.errorDisplays[fieldName];
    
    if (errorDisplay) {
      errorDisplay.textContent = '';
    }
    
    field.classList.remove('invalid');
  }
  
  // Получение данных формы
  getFormData() {
    const formData = new FormData(this.form);
    return Object.fromEntries(formData.entries());
  }
  
  // Валидация всей формы
  async validateForm() {
    const fieldNames = Object.keys(this.fields);
    const results = await Promise.all(
      fieldNames.map(fieldName => this.validateField(fieldName))
    );
    
    return results.every(result => result);
  }
  
  // Обработка отправки формы
  async handleSubmit(event) {
    event.preventDefault();
    
    const isValid = await this.validateForm();
    
    if (isValid) {
      // Отправка формы
      this.submitForm();
    } else {
      console.log('Форма содержит ошибки');
      // Можно добавить прокрутку к первому полю с ошибкой
      this.scrollToFirstError();
    }
  }
  
  // Прокрутка к первому полю с ошибкой
  scrollToFirstError() {
    const invalidField = this.form.querySelector('.invalid');
    if (invalidField) {
      invalidField.scrollIntoView({ behavior: 'smooth', block: 'center' });
      invalidField.focus();
    }
  }
  
  // Отправка формы
  async submitForm() {
    // Показываем индикатор загрузки
    this.setSubmitting(true);
    
    try {
      const formData = new FormData(this.form);
      const response = await fetch(this.form.action, {
        method: this.form.method,
        body: formData
      });
      
      if (response.ok) {
        this.showSuccessMessage('Форма успешно отправлена!');
        this.form.reset();
      } else {
        throw new Error('Ошибка сервера');
      }
    } catch (error) {
      this.showErrorMessage('Произошла ошибка при отправке формы');
    } finally {
      this.setSubmitting(false);
    }
  }
  
  // Управление состоянием отправки
  setSubmitting(isSubmitting) {
    const submitButton = this.form.querySelector('button[type="submit"]');
    if (submitButton) {
      submitButton.disabled = isSubmitting;
      submitButton.textContent = isSubmitting ? 'Отправка...' : 'Отправить';
    }
  }
  
  // Показ сообщения об успехе
  showSuccessMessage(message) {
    this.showMessage(message, 'success');
  }
  
  // Показ сообщения об ошибке
  showErrorMessage(message) {
    this.showMessage(message, 'error');
  }
  
  // Универсальный метод показа сообщений
  showMessage(message, type) {
    let messageElement = document.getElementById('form-message');
    
    if (!messageElement) {
      messageElement = document.createElement('div');
      messageElement.id = 'form-message';
      messageElement.style.padding = '10px';
      messageElement.style.margin = '10px 0';
      messageElement.style.borderRadius = '4px';
      messageElement.style.display = 'none';
      
      this.form.parentNode.insertBefore(messageElement, this.form);
    }
    
    messageElement.textContent = message;
    messageElement.className = `form-message ${type}`;
    messageElement.style.display = 'block';
    messageElement.style.backgroundColor = type === 'success' ? '#d4edda' : '#f8d7da';
    messageElement.style.color = type === 'success' ? '#155724' : '#721c24';
    messageElement.style.border = `1px solid ${type === 'success' ? '#c3e6cb' : '#f5c6cb'}`;
    
    // Автоматическое скрытие сообщения
    setTimeout(() => {
      messageElement.style.display = 'none';
    }, 5000);
  }
}

// Пример использования
document.addEventListener('DOMContentLoaded', () => {
  const formValidator = new DynamicFormValidator('#contact-form');
  
  // Установка правил валидации
  formValidator.setValidationRules({
    name: [
      value => value && value.trim().length > 0 ? { isValid: true } : { isValid: false, message: 'Имя обязательно' },
      value => value && value.trim().length >= 2 ? { isValid: true } : { isValid: false, message: 'Имя должно быть не менее 2 символов' }
    ],
    email: [
      value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? { isValid: true } : { isValid: false, message: 'Некорректный email' }
    ],
    message: [
      value => value && value.trim().length > 0 ? { isValid: true } : { isValid: false, message: 'Сообщение обязательно' },
      value => value && value.trim().length <= 1000 ? { isValid: true } : { isValid: false, message: 'Сообщение слишком длинное' }
    ]
  });
});
```

## 2. Работа с API и асинхронными операциями

### Универсальный API клиент с кешированием и повторными попытками

```javascript
// Класс для работы с API
class APIClient {
  constructor(baseURL, defaultOptions = {}) {
    this.baseURL = baseURL;
    this.defaultOptions = {
      headers: {
        'Content-Type': 'application/json',
      },
      ...defaultOptions
    };
    this.cache = new Map();
    this.pendingRequests = new Map();
  }
  
  // Основной метод для выполнения запросов
  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;
    const config = {
      ...this.defaultOptions,
      ...options,
      headers: {
        ...this.defaultOptions.headers,
        ...options.headers
      }
    };
    
    // Генерация ключа для кеширования
    const cacheKey = this.generateCacheKey(url, config);
    
    // Проверка кеша
    if (config.cache && this.cache.has(cacheKey)) {
      const cached = this.cache.get(cacheKey);
      if (Date.now() - cached.timestamp < (config.cacheTTL || 5 * 60 * 1000)) { // 5 минут по умолчанию
        return cached.data;
      } else {
        this.cache.delete(cacheKey);
      }
    }
    
    // Проверка на ожидающий запрос
    if (this.pendingRequests.has(cacheKey)) {
      return this.pendingRequests.get(cacheKey);
    }
    
    // Выполнение запроса
    const promise = this.executeRequest(url, config, cacheKey);
    this.pendingRequests.set(cacheKey, promise);
    
    try {
      const result = await promise;
      return result;
    } finally {
      this.pendingRequests.delete(cacheKey);
    }
  }
  
  // Выполнение HTTP запроса
  async executeRequest(url, config, cacheKey) {
    let lastError;
    
    // Повторные попытки
    for (let attempt = 0; attempt <= (config.retries || 0); attempt++) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), config.timeout || 10000);
        
        const response = await fetch(url, {
          ...config,
          signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        
        // Кеширование результата
        if (config.cache) {
          this.cache.set(cacheKey, {
            data,
            timestamp: Date.now()
          });
        }
        
        return data;
      } catch (error) {
        lastError = error;
        
        if (attempt < config.retries) {
          // Задержка перед повторной попыткой
          await this.delay((config.retryDelay || 1000) * (attempt + 1));
        }
      }
    }
    
    throw lastError;
  }
  
  // Генерация ключа для кеширования
  generateCacheKey(url, config) {
    return `${url}_${JSON.stringify({
      method: config.method,
      body: config.body
    })}`;
  }
  
  // Задержка
  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  // Методы для разных HTTP методов
  get(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'GET' });
  }
  
  post(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
  
  put(endpoint, data, options = {}) {
    return this.request(endpoint, {
      ...options,
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }
  
  delete(endpoint, options = {}) {
    return this.request(endpoint, { ...options, method: 'DELETE' });
  }
  
  // Очистка кеша
  clearCache(pattern = null) {
    if (pattern) {
      // Очистка по паттерну
      for (const key of this.cache.keys()) {
        if (key.includes(pattern)) {
          this.cache.delete(key);
        }
      }
    } else {
      this.cache.clear();
    }
  }
}

// Использование API клиента
const apiClient = new APIClient('https://api.example.com', {
  retries: 3,
  retryDelay: 1000,
  timeout: 10000
});

// Примеры использования
async function loadUserProfile(userId) {
  try {
    const profile = await apiClient.get(`/users/${userId}`, {
      cache: true,
      cacheTTL: 10 * 60 * 1000 // 10 минут
    });
    return profile;
  } catch (error) {
    console.error('Ошибка загрузки профиля:', error);
    throw error;
  }
}

async function updateUserProfile(userId, profileData) {
  try {
    const updatedProfile = await apiClient.put(`/users/${userId}`, profileData);
    // Очистка кеша для этого пользователя
    apiClient.clearCache(`/users/${userId}`);
    return updatedProfile;
  } catch (error) {
    console.error('Ошибка обновления профиля:', error);
    throw error;
  }
}
```

## 3. Управление состоянием приложения

### Простой менеджер состояния с поддержкой middleware

```javascript
// Менеджер состояния
class StateManager {
  constructor(initialState = {}) {
    this.state = this.deepFreeze({ ...initialState });
    this.listeners = [];
    this.middleware = [];
    this.history = [this.state];
    this.historyIndex = 0;
  }
  
  // Получение текущего состояния
  getState() {
    return this.state;
  }
  
  // Установка нового состояния
  setState(newState, description = 'Update state') {
    const prevState = this.state;
    
    // Применение middleware
    const processedState = this.applyMiddleware(prevState, newState);
    
    // Создание нового состояния (иммутабельность)
    this.state = this.deepFreeze({ ...prevState, ...processedState });
    
    // Уведомление слушателей
    this.notifyListeners(prevState, this.state);
    
    // Добавление в историю (для отладки/undo)
    this.addToHistory(this.state, description);
  }
  
  // Обновление вложенного состояния
  updateNestedState(path, value, description = 'Update nested state') {
    const pathParts = path.split('.');
    const newState = this.updateNestedValue({ ...this.state }, pathParts, value);
    this.setState(newState, description);
  }
  
  // Внутренний метод для обновления вложенного значения
  updateNestedValue(obj, pathParts, value) {
    if (pathParts.length === 1) {
      return { ...obj, [pathParts[0]]: value };
    }
    
    const [head, ...tail] = pathParts;
    return {
      ...obj,
      [head]: this.updateNestedValue({ ...(obj[head] || {}) }, tail, value)
    };
  }
  
  // Подписка на изменения
  subscribe(listener) {
    this.listeners.push(listener);
    
    // Возвращаем функцию для отписки
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
  
  // Уведомление слушателей
  notifyListeners(prevState, newState) {
    this.listeners.forEach(listener => {
      try {
        listener(newState, prevState);
      } catch (error) {
        console.error('Ошибка в слушателе состояния:', error);
      }
    });
  }
  
  // Добавление middleware
  use(middleware) {
    this.middleware.push(middleware);
  }
  
  // Применение middleware
  applyMiddleware(prevState, newState) {
    return this.middleware.reduce((state, middleware) => {
      return middleware(state, newState) || state;
    }, newState);
  }
  
  // Добавление в историю
  addToHistory(state, description) {
    // Ограничение размера истории
    if (this.history.length > 50) {
      this.history = this.history.slice(-20); // Сохраняем последние 20 состояний
    }
    
    this.history.push({
      state: this.deepFreeze({ ...state }),
      timestamp: Date.now(),
      description
    });
    
    this.historyIndex = this.history.length - 1;
  }
  
  // Получение истории
  getHistory() {
    return this.history;
  }
  
  // Глубокое замораживание объекта
  deepFreeze(obj) {
    if (obj === null || typeof obj !== 'object') return obj;
    
    Object.getOwnPropertyNames(obj).forEach(prop => {
      if (obj[prop] !== null && typeof obj[prop] === 'object') {
        this.deepFreeze(obj[prop]);
      }
    });
    
    return Object.freeze(obj);
  }
  
  // Middleware для логирования
  static createLoggingMiddleware() {
    return (prevState, newState) => {
      console.group('State Update');
      console.log('Previous state:', prevState);
      console.log('New state:', newState);
      console.groupEnd();
      return newState;
    };
  }
  
  // Middleware для валидации
  static createValidationMiddleware(validations) {
    return (prevState, newState) => {
      const errors = {};
      
      Object.keys(validations).forEach(key => {
        if (key in newState) {
          const validator = validations[key];
          const isValid = typeof validator === 'function' 
            ? validator(newState[key], newState) 
            : validator.test(newState[key]);
          
          if (!isValid) {
            errors[key] = validator.message || 'Invalid value';
          }
        }
      });
      
      if (Object.keys(errors).length > 0) {
        console.warn('Validation errors:', errors);
        // Можно выбросить ошибку или изменить состояние
      }
      
      return newState;
    };
  }
}

// Использование StateManager
const store = new StateManager({
  user: { id: null, name: '', email: '' },
  theme: 'light',
  notifications: []
});

// Добавление middleware
store.use(StateManager.createLoggingMiddleware());
store.use(StateManager.createValidationMiddleware({
  'user.email': {
    test: (value) => !value || /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    message: 'Некорректный email'
  }
}));

// Подписка на изменения
const unsubscribe = store.subscribe((newState, prevState) => {
  console.log('Состояние изменилось');
  
  // Обновление UI
  updateUI(newState, prevState);
});

// Функция обновления UI
function updateUI(newState, prevState) {
  // Обновление имени пользователя
  if (newState.user.name !== prevState.user.name) {
    const nameElement = document.getElementById('user-name');
    if (nameElement) {
      nameElement.textContent = newState.user.name || 'Гость';
    }
  }
  
  // Обновление темы
  if (newState.theme !== prevState.theme) {
    document.body.className = newState.theme;
  }
  
  // Обновление уведомлений
  if (newState.notifications.length !== prevState.notifications.length) {
    renderNotifications(newState.notifications);
  }
}

// Пример использования
store.setState({
  user: { id: 1, name: 'John Doe', email: 'john@example.com' },
  theme: 'dark'
});
```

## 4. Работа с DOM и пользовательским интерфейсом

### Управление динамическими списками с виртуализацией

```javascript
// Класс для виртуализации списков (эффективный рендеринг больших списков)
class VirtualList {
  constructor(container, itemHeight, data = []) {
    this.container = container;
    this.itemHeight = itemHeight;
    this.data = data;
    this.visibleItems = [];
    this.startIndex = 0;
    this.endIndex = 0;
    this.scrollHeight = 0;
    this.scrollTop = 0;
    
    this.init();
  }
  
  init() {
    // Создание скролл контейнера
    this.scrollContainer = document.createElement('div');
    this.scrollContainer.style.height = '400px';
    this.scrollContainer.style.overflow = 'auto';
    this.scrollContainer.style.position = 'relative';
    
    // Создание внутреннего контейнера для элементов
    this.itemsContainer = document.createElement('div');
    this.itemsContainer.style.position = 'relative';
    
    this.scrollContainer.appendChild(this.itemsContainer);
    this.container.appendChild(this.scrollContainer);
    
    // Обработчик скролла
    this.scrollContainer.addEventListener('scroll', this.handleScroll.bind(this));
    
    // Инициализация
    this.updateList();
  }
  
  // Обработчик скролла
  handleScroll() {
    this.scrollTop = this.scrollContainer.scrollTop;
    this.updateVisibleItems();
  }
  
  // Обновление видимых элементов
  updateVisibleItems() {
    const containerHeight = this.scrollContainer.clientHeight;
    const visibleStart = this.scrollTop;
    const visibleEnd = this.scrollTop + containerHeight;
    
    // Рассчитываем индексы видимых элементов
    this.startIndex = Math.floor(visibleStart / this.itemHeight);
    this.endIndex = Math.min(
      this.data.length - 1,
      Math.ceil(visibleEnd / this.itemHeight)
    );
    
    // Ограничиваем индексы
    this.startIndex = Math.max(0, this.startIndex - 2); // Добавляем небольшой запас
    this.endIndex = Math.min(this.data.length - 1, this.endIndex + 2);
    
    this.renderVisibleItems();
  }
  
  // Рендеринг видимых элементов
  renderVisibleItems() {
    // Очищаем текущие элементы
    this.itemsContainer.innerHTML = '';
    
    // Добавляем пустое пространство сверху
    const topSpacer = document.createElement('div');
    topSpacer.style.height = `${this.startIndex * this.itemHeight}px`;
    this.itemsContainer.appendChild(topSpacer);
    
    // Рендерим видимые элементы
    for (let i = this.startIndex; i <= this.endIndex; i++) {
      if (i < this.data.length) {
        const itemElement = this.createItemElement(this.data[i], i);
        this.itemsContainer.appendChild(itemElement);
      }
    }
    
    // Добавляем пустое пространство снизу
    const bottomSpacer = document.createElement('div');
    bottomSpacer.style.height = `${(this.data.length - this.endIndex - 1) * this.itemHeight}px`;
    this.itemsContainer.appendChild(bottomSpacer);
  }
  
  // Создание элемента списка
  createItemElement(item, index) {
    const element = document.createElement('div');
    element.style.height = `${this.itemHeight}px`;
    element.style.display = 'flex';
    element.style.alignItems = 'center';
    element.style.padding = '0 10px';
    element.style.borderBottom = '1px solid #eee';
    element.style.position = 'absolute';
    element.style.left = '0';
    element.style.right = '0';
    element.style.top = `${index * this.itemHeight}px`;
    
    element.innerHTML = `
      <span style="flex: 1;">${item.id}. ${item.name}</span>
      <button onclick="console.log('Item ${index} clicked')" style="background: #007bff; color: white; border: none; padding: 4px 8px; border-radius: 4px; cursor: pointer;">
        Действие
      </button>
    `;
    
    return element;
  }
  
  // Обновление данных
  updateData(newData) {
    this.data = newData;
    this.updateList();
  }
  
  // Обновление списка
  updateList() {
    this.scrollHeight = this.data.length * this.itemHeight;
    this.scrollContainer.style.height = Math.min(400, this.scrollHeight) + 'px';
    this.updateVisibleItems();
  }
  
  // Прокрутка к элементу
  scrollToItem(index) {
    const scrollTop = index * this.itemHeight;
    this.scrollContainer.scrollTop = scrollTop;
  }
}

// Использование VirtualList
function initializeVirtualList() {
  // Генерация тестовых данных
  const testData = Array.from({ length: 10000 }, (_, i) => ({
    id: i + 1,
    name: `Элемент ${i + 1}`,
    description: `Описание элемента ${i + 1}`
  }));
  
  const container = document.getElementById('virtual-list-container');
  if (container) {
    const virtualList = new VirtualList(container, 50, testData);
  }
}
```

## 5. Работа с событиями и пользовательским вводом

### Система горячих клавиш (keyboard shortcuts)

```javascript
// Менеджер горячих клавиш
class KeyboardManager {
  constructor() {
    this.shortcuts = new Map();
    this.activeScope = 'global';
    this.scopes = { global: {} };
    
    this.init();
  }
  
  init() {
    document.addEventListener('keydown', this.handleKeydown.bind(this));
  }
  
  // Регистрация горячей клавиши
  register(shortcut, callback, scope = 'global') {
    const normalizedShortcut = this.normalizeShortcut(shortcut);
    
    if (!this.scopes[scope]) {
      this.scopes[scope] = {};
    }
    
    this.scopes[scope][normalizedShortcut] = callback;
  }
  
  // Удаление горячей клавиши
  unregister(shortcut, scope = 'global') {
    const normalizedShortcut = this.normalizeShortcut(shortcut);
    delete this.scopes[scope][normalizedShortcut];
  }
  
  // Установка активной области
  setScope(scope) {
    this.activeScope = scope;
  }
  
  // Обработчик нажатия клавиши
  handleKeydown(event) {
    const shortcut = this.getShortcutFromEvent(event);
    
    // Проверяем в активной области
    if (this.scopes[this.activeScope] && this.scopes[this.activeScope][shortcut]) {
      event.preventDefault();
      this.scopes[this.activeScope][shortcut](event);
      return;
    }
    
    // Проверяем в глобальной области
    if (this.scopes.global[shortcut]) {
      event.preventDefault();
      this.scopes.global[shortcut](event);
    }
  }
  
  // Получение комбинации клавиш из события
  getShortcutFromEvent(event) {
    const keys = [];
    
    if (event.ctrlKey) keys.push('Ctrl');
    if (event.shiftKey) keys.push('Shift');
    if (event.altKey) keys.push('Alt');
    if (event.metaKey) keys.push('Meta');
    
    keys.push(event.key);
    
    return keys.join('+').toLowerCase();
  }
  
  // Нормализация комбинации клавиш
  normalizeShortcut(shortcut) {
    return shortcut.toLowerCase()
      .replace(/\s+/g, '')
      .split('+')
      .sort()
      .join('+');
  }
  
  // Проверка, является ли элемент полем ввода
  isInputElement(element) {
    return ['INPUT', 'TEXTAREA', 'SELECT'].includes(element.tagName);
  }
}

// Использование KeyboardManager
const keyboardManager = new KeyboardManager();

// Регистрация горячих клавиш
keyboardManager.register('Ctrl+S', (event) => {
  event.preventDefault();
  console.log('Сохранение документа...');
  // saveDocument();
});

keyboardManager.register('Ctrl+Z', (event) => {
  event.preventDefault();
  console.log('Отмена действия...');
  // undoAction();
});

keyboardManager.register('Ctrl+Shift+Z', (event) => {
  event.preventDefault();
  console.log('Повтор действия...');
  // redoAction();
});

keyboardManager.register('Escape', (event) => {
  console.log('Закрытие модального окна...');
  // closeModal();
});
```

## 6. Оптимизация производительности

### Дебаунс и троттлинг

```javascript
// Утилиты для оптимизации производительности
class PerformanceUtils {
  // Дебаунс функции
  static debounce(func, delay) {
    let timeoutId;
    return function (...args) {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
  }
  
  // Троттлинг функции
  static throttle(func, limit) {
    let inThrottle;
    return function (...args) {
      if (!inThrottle) {
        func.apply(this, args);
        inThrottle = true;
        setTimeout(() => inThrottle = false, limit);
      }
    };
  }
  
  // Оптимизированный обработчик скролла
  static createOptimizedScrollHandler(callback, options = {}) {
    const { throttleMs = 16, debounceMs = 100 } = options;
    
    if (throttleMs > 0) {
      callback = this.throttle(callback, throttleMs);
    }
    
    if (debounceMs > 0) {
      callback = this.debounce(callback, debounceMs);
    }
    
    return callback;
  }
  
  // Ленивая загрузка изображений
  static lazyLoadImages(container = document) {
    const images = container.querySelectorAll('img[data-src]');
    
    const imageObserver = new IntersectionObserver((entries, observer) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          img.src = img.dataset.src;
          img.removeAttribute('data-src');
          observer.unobserve(img);
        }
      });
    });
    
    images.forEach(img => imageObserver.observe(img));
    
    return imageObserver;
  }
  
  // Виртуальный скролл для таблиц
  static createVirtualTable(container, rows, rowHeight) {
    const tableContainer = document.createElement('div');
    tableContainer.style.height = '400px';
    tableContainer.style.overflow = 'auto';
    tableContainer.style.position = 'relative';
    
    const tableContent = document.createElement('div');
    tableContent.style.position = 'relative';
    
    // Рассчитываем видимые строки
    const updateVisibleRows = PerformanceUtils.throttle(() => {
      const start = Math.floor(tableContainer.scrollTop / rowHeight);
      const visibleCount = Math.ceil(tableContainer.clientHeight / rowHeight);
      const end = Math.min(rows.length, start + visibleCount + 5); // Добавляем буфер
      
      // Очищаем контент
      tableContent.innerHTML = '';
      
      // Добавляем верхний отступ
      const topSpacer = document.createElement('div');
      topSpacer.style.height = `${start * rowHeight}px`;
      tableContent.appendChild(topSpacer);
      
      // Добавляем видимые строки
      for (let i = start; i < end; i++) {
        if (i < rows.length) {
          const row = document.createElement('div');
          row.style.height = `${rowHeight}px`;
          row.style.display = 'flex';
          row.style.alignItems = 'center';
          row.style.padding = '0 10px';
          row.style.borderBottom = '1px solid #ddd';
          row.innerHTML = rows[i].join(' | ');
          tableContent.appendChild(row);
        }
      }
      
      // Добавляем нижний отступ
      const bottomSpacer = document.createElement('div');
      bottomSpacer.style.height = `${(rows.length - end) * rowHeight}px`;
      tableContent.appendChild(bottomSpacer);
    }, 16); // ~60 FPS
    
    tableContainer.addEventListener('scroll', updateVisibleRows);
    tableContainer.appendChild(tableContent);
    container.appendChild(tableContainer);
    
    // Инициализация
    updateVisibleRows();
    
    return { container: tableContainer, update: updateVisibleRows };
  }
}

// Пример использования дебаунса для поиска
const searchInput = document.getElementById('search-input');
if (searchInput) {
  const debouncedSearch = PerformanceUtils.debounce((query) => {
    console.log('Поиск:', query);
    // Выполнение поиска
  }, 300);
  
  searchInput.addEventListener('input', (e) => {
    debouncedSearch(e.target.value);
  });
}

// Пример использования троттлинга для скролла
const throttledScrollHandler = PerformanceUtils.throttle(() => {
  console.log('Скролл', window.scrollY);
  // Обновление UI при скролле
}, 16); // ~60 FPS

window.addEventListener('scroll', throttledScrollHandler);
```

## Заключение

Решение распространенных задач в JavaScript для фронтенд разработки требует понимания как основ языка, так и специфики работы с браузерным API. Рассмотренные примеры помогут вам эффективно решать повседневные задачи и создавать более качественные веб-приложения.

## См. также

- [[Функции в JavaScript - Продвинутые концепции]]
- [[closure_practical_examples.md]]
- [[Прототипное наследование в JavaScript]]
- [[Метапрограммирование и прокси в JavaScript]]
- [[Функциональное программирование в JS]]
- [[DOM манипуляции в JavaScript]]
- [[promise_practical_examples.md]]
- [[frontend_patterns_examples.md]]
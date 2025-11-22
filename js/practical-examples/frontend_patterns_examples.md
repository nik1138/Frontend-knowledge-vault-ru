---
tags: [javascript, frontend, practical-examples, patterns]
aliases: [практические примеры js для фронтенда, паттерны фронтенд разработки]
---

# Практические примеры JavaScript для Frontend разработки

## Введение

В этой статье собраны практические примеры использования JavaScript в фронтенд разработке, охватывающие распространенные задачи, паттерны и решения, с которыми сталкиваются разработчики при создании веб-приложений.

## Практический пример: Управление формами с валидацией

```javascript
// Класс для управления формами с валидацией
class FormManager {
  constructor(formElement, validationRules = {}) {
    this.form = formElement;
    this.validationRules = validationRules;
    this.errors = {};
    this.fieldValidators = new Map();
    
    this.init();
  }
  
  init() {
    // Устанавливаем обработчики событий
    this.form.addEventListener('submit', this.handleSubmit.bind(this));
    
    // Устанавливаем валидацию для каждого поля
    Object.keys(this.validationRules).forEach(fieldName => {
      const field = this.form.querySelector(`[name="${fieldName}"]`);
      if (field) {
        field.addEventListener('blur', () => this.validateField(fieldName));
        field.addEventListener('input', () => this.clearFieldError(fieldName));
      }
    });
  }
  
  async validateField(fieldName) {
    const field = this.form.querySelector(`[name="${fieldName}"]`);
    const value = field.value;
    const rules = this.validationRules[fieldName];
    
    if (!rules) return true;
    
    const fieldErrors = [];
    
    for (const rule of rules) {
      let isValid = false;
      let errorMessage = '';
      
      if (typeof rule === 'function') {
        const result = await Promise.resolve(rule(value));
        isValid = result.isValid !== undefined ? result.isValid : result;
        errorMessage = result.error || (isValid ? '' : 'Поле не прошло валидацию');
      } else if (rule.test && rule.message) {
        // Объект с регулярным выражением и сообщением
        isValid = rule.test.test ? rule.test.test(value) : rule.test(value);
        errorMessage = rule.message;
      } else {
        // Простое логическое выражение
        isValid = Boolean(rule(value));
        errorMessage = 'Некорректное значение';
      }
      
      if (!isValid) {
        fieldErrors.push(errorMessage);
        break; // Останавливаемся на первой ошибке
      }
    }
    
    if (fieldErrors.length > 0) {
      this.setFieldError(fieldName, fieldErrors[0]);
      return false;
    } else {
      this.clearFieldError(fieldName);
      return true;
    }
  }
  
  setFieldError(fieldName, message) {
    this.errors[fieldName] = message;
    
    const field = this.form.querySelector(`[name="${fieldName}"]`);
    const errorElement = field.parentNode.querySelector('.error-message') || 
                        this.createErrorElement();
    
    errorElement.textContent = message;
    errorElement.style.display = 'block';
    
    field.classList.add('error');
  }
  
  clearFieldError(fieldName) {
    delete this.errors[fieldName];
    
    const field = this.form.querySelector(`[name="${fieldName}"]`);
    const errorElement = field.parentNode.querySelector('.error-message');
    
    if (errorElement) {
      errorElement.style.display = 'none';
    }
    
    field.classList.remove('error');
  }
  
  createErrorElement() {
    const errorElement = document.createElement('div');
    errorElement.className = 'error-message';
    errorElement.style.color = 'red';
    errorElement.style.fontSize = '12px';
    errorElement.style.display = 'none';
    
    return errorElement;
  }
  
  async validateForm() {
    const fieldNames = Object.keys(this.validationRules);
    const results = await Promise.all(
      fieldNames.map(fieldName => this.validateField(fieldName))
    );
    
    return results.every(result => result);
  }
  
  async handleSubmit(event) {
    event.preventDefault();
    
    const isValid = await this.validateForm();
    
    if (isValid) {
      // Отправка формы
      this.submitForm();
    } else {
      console.log('Форма содержит ошибки:', this.errors);
    }
  }
  
  async submitForm() {
    const formData = new FormData(this.form);
    const data = Object.fromEntries(formData.entries());
    
    try {
      const response = await fetch(this.form.action, {
        method: this.form.method,
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      });
      
      if (response.ok) {
        this.showSuccessMessage('Форма успешно отправлена!');
        this.form.reset();
      } else {
        throw new Error('Ошибка отправки формы');
      }
    } catch (error) {
      this.showErrorMessage('Произошла ошибка при отправке формы');
    }
  }
  
  showSuccessMessage(message) {
    this.showMessage(message, 'success');
  }
  
  showErrorMessage(message) {
    this.showMessage(message, 'error');
  }
  
  showMessage(message, type) {
    const messageElement = document.getElementById('form-message') || 
                          this.createMessageElement();
    
    messageElement.textContent = message;
    messageElement.className = `form-message ${type}`;
    
    // Автоматическое скрытие сообщения
    setTimeout(() => {
      messageElement.style.display = 'none';
    }, 5000);
  }
  
  createMessageElement() {
    const messageElement = document.createElement('div');
    messageElement.id = 'form-message';
    messageElement.style.padding = '10px';
    messageElement.style.margin = '10px 0';
    messageElement.style.borderRadius = '4px';
    
    document.querySelector('.form-container').prepend(messageElement);
    return messageElement;
  }
}

// Пример использования
const validationRules = {
  email: [
    value => value && value.length > 0 ? { isValid: true } : { isValid: false, error: 'Email обязателен' },
    value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? { isValid: true } : { isValid: false, error: 'Некорректный email' },
    async value => {
      // Асинхронная проверка уникальности email
      try {
        const response = await fetch(`/api/check-email?email=${encodeURIComponent(value)}`);
        const result = await response.json();
        return result.isUnique ? { isValid: true } : { isValid: false, error: 'Email уже используется' };
      } catch (error) {
        return { isValid: true }; // Не блокируем валидацию при ошибке API
      }
    }
  ],
  password: [
    value => value && value.length >= 8 ? { isValid: true } : { isValid: false, error: 'Пароль должен быть не менее 8 символов' },
    value => /(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value) ? { isValid: true } : { isValid: false, error: 'Пароль должен содержать заглавную букву, строчную букву и цифру' }
  ],
  confirmPassword: [
    value => value && value.length > 0 ? { isValid: true } : { isValid: false, error: 'Подтверждение пароля обязательно' },
    (value, formData) => value === formData.password ? { isValid: true } : { isValid: false, error: 'Пароли не совпадают' }
  ]
};

// Инициализация формы
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('registration-form');
  if (form) {
    new FormManager(form, validationRules);
  }
});
```

## Практический пример: Управление состоянием приложения

```javascript
// Простой менеджер состояния с использованием паттерна Observer
class StateManager {
  constructor(initialState = {}) {
    this.state = { ...initialState };
    this.listeners = [];
  }
  
  getState() {
    return { ...this.state };
  }
  
  setState(newState) {
    const prevState = { ...this.state };
    this.state = { ...this.state, ...newState };
    
    // Уведомляем слушателей об изменении состояния
    this.notifyListeners(prevState, this.state);
  }
  
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
  
  notifyListeners(prevState, newState) {
    this.listeners.forEach(listener => {
      listener(newState, prevState);
    });
  }
  
  // Метод для обновления вложенных свойств
  updateNestedState(path, value) {
    const pathParts = path.split('.');
    const lastKey = pathParts.pop();
    const nestedState = pathParts.reduce((obj, key) => obj[key] || {}, this.state);
    
    if (nestedState[lastKey] !== value) {
      const newState = { ...this.state };
      let current = newState;
      
      for (let i = 0; i < pathParts.length; i++) {
        current[pathParts[i]] = { ...current[pathParts[i]] };
        current = current[pathParts[i]];
      }
      
      current[lastKey] = value;
      this.setState(newState);
    }
  }
}

// Менеджер состояния с middleware
class AdvancedStateManager extends StateManager {
  constructor(initialState = {}) {
    super(initialState);
    this.middleware = [];
  }
  
  use(middleware) {
    this.middleware.push(middleware);
  }
  
  setState(newState) {
    // Применяем middleware
    const processedState = this.middleware.reduce((state, middleware) => {
      return middleware(state, newState);
    }, this.state);
    
    super.setState(processedState);
  }
}

// Пример middleware для логирования
const loggingMiddleware = (prevState, newState) => {
  console.log('Предыдущее состояние:', prevState);
  console.log('Новое состояние:', newState);
  return newState;
};

// Пример middleware для валидации
const validationMiddleware = (prevState, newState) => {
  // Пример простой валидации
  if (newState.user && newState.user.age && newState.user.age < 0) {
    console.warn('Возраст не может быть отрицательным');
    return { ...newState, user: { ...newState.user, age: 0 } };
  }
  return newState;
};

// Использование продвинутого менеджера состояния
const store = new AdvancedStateManager({
  user: { name: '', email: '', age: 0 },
  theme: 'light',
  notifications: []
});

store.use(loggingMiddleware);
store.use(validationMiddleware);

// Подписка на изменения состояния
const unsubscribe = store.subscribe((newState, prevState) => {
  console.log('Состояние изменилось');
  
  // Обновление UI при изменении состояния
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
  user: { name: 'John Doe', email: 'john@example.com', age: 30 },
  theme: 'dark'
});
```

## Практический пример: Кастомные хуки (для React-подобных приложений)

```javascript
// Кастомные хуки для функциональных компонентов
class HooksManager {
  constructor() {
    this.hooks = [];
    this.index = 0;
  }
  
  reset() {
    this.index = 0;
  }
  
  useState(initialValue) {
    const hookIndex = this.index;
    this.index++;
    
    if (!this.hooks[hookIndex]) {
      this.hooks[hookIndex] = {
        state: typeof initialValue === 'function' ? initialValue() : initialValue,
        setState: (newState) => {
          const hook = this.hooks[hookIndex];
          const newStateValue = typeof newState === 'function' ? newState(hook.state) : newState;
          hook.state = newStateValue;
          
          // Вызов обновления компонента
          if (hook.updateCallback) {
            hook.updateCallback();
          }
        }
      };
    }
    
    return [this.hooks[hookIndex].state, this.hooks[hookIndex].setState];
  }
  
  useEffect(callback, dependencies) {
    const hookIndex = this.index;
    this.index++;
    
    if (!this.hooks[hookIndex]) {
      this.hooks[hookIndex] = {
        callback,
        dependencies,
        cleanup: null
      };
      
      // Выполняем эффект при первом рендере
      this.hooks[hookIndex].cleanup = callback();
    } else {
      const hook = this.hooks[hookIndex];
      
      // Проверяем, изменились ли зависимости
      if (!dependencies || this.dependenciesChanged(hook.dependencies, dependencies)) {
        // Выполняем cleanup предыдущего эффекта
        if (hook.cleanup && typeof hook.cleanup === 'function') {
          hook.cleanup();
        }
        
        // Выполняем новый эффект
        hook.cleanup = callback();
        hook.dependencies = dependencies;
      }
    }
  }
  
  dependenciesChanged(prevDeps, nextDeps) {
    if (!prevDeps || !nextDeps || prevDeps.length !== nextDeps.length) {
      return true;
    }
    
    for (let i = 0; i < prevDeps.length; i++) {
      if (prevDeps[i] !== nextDeps[i]) {
        return true;
      }
    }
    
    return false;
  }
  
  useRef(initialValue) {
    const hookIndex = this.index;
    this.index++;
    
    if (!this.hooks[hookIndex]) {
      this.hooks[hookIndex] = { current: initialValue };
    }
    
    return this.hooks[hookIndex];
  }
}

// Глобальный менеджер хуков
const globalHooksManager = new HooksManager();

// Функция для создания компонента с хуками
function createComponent(renderFunction) {
  return function(props) {
    globalHooksManager.reset();
    
    // Устанавливаем callback обновления
    const updateCallback = () => {
      // Здесь должна быть логика перерендера компонента
      console.log('Компонент обновлен');
    };
    
    // Устанавливаем callback для всех useState хуков
    for (const hook of globalHooksManager.hooks) {
      if (hook && hook.setState) {
        hook.updateCallback = updateCallback;
      }
    }
    
    return renderFunction(props);
  };
}

// Пример использования хуков
function CounterComponent() {
  const [count, setCount] = globalHooksManager.useState(0);
  const [name, setName] = globalHooksManager.useState('Guest');
  
  globalHooksManager.useEffect(() => {
    document.title = `Счетчик: ${count}`;
    console.log('Заголовок обновлен');
    
    return () => {
      console.log('Очистка эффекта');
    };
  }, [count]);
  
  const countRef = globalHooksManager.useRef(count);
  
  // Возвращаем виртуальный DOM элемент
  return {
    type: 'div',
    props: {
      children: [
        { type: 'h2', props: { children: `Привет, ${name}!` } },
        { type: 'p', props: { children: `Счетчик: ${count}` } },
        {
          type: 'button',
          props: {
            children: 'Увеличить',
            onClick: () => setCount(prev => prev + 1)
          }
        },
        {
          type: 'button',
          props: {
            children: 'Сброс',
            onClick: () => setCount(0)
          }
        }
      ]
    }
  };
}
```

## Практический пример: Управление асинхронными операциями

```javascript
// Менеджер асинхронных операций
class AsyncManager {
  constructor() {
    this.pendingRequests = new Map();
    this.cache = new Map();
  }
  
  // Выполнение асинхронной операции с кешированием
  async execute(key, asyncFunction, options = {}) {
    const { cache = true, ttl = 5 * 60 * 1000 } = options; // 5 минут по умолчанию
    
    // Проверяем кеш
    if (cache && this.cache.has(key)) {
      const cached = this.cache.get(key);
      if (Date.now() - cached.timestamp < ttl) {
        return cached.data;
      } else {
        // Удаляем просроченный кеш
        this.cache.delete(key);
      }
    }
    
    // Проверяем, есть ли уже выполняющийся запрос
    if (this.pendingRequests.has(key)) {
      return this.pendingRequests.get(key);
    }
    
    // Выполняем асинхронную операцию
    const promise = asyncFunction()
      .then(result => {
        if (cache) {
          this.cache.set(key, {
            data: result,
            timestamp: Date.now()
          });
        }
        this.pendingRequests.delete(key);
        return result;
      })
      .catch(error => {
        this.pendingRequests.delete(key);
        throw error;
      });
    
    this.pendingRequests.set(key, promise);
    return promise;
  }
  
  // Отмена асинхронной операции
  cancel(key) {
    if (this.pendingRequests.has(key)) {
      this.pendingRequests.delete(key);
    }
  }
  
  // Очистка кеша
  clearCache(key) {
    if (key) {
      this.cache.delete(key);
    } else {
      this.cache.clear();
    }
  }
  
  // Выполнение с таймаутом
  async executeWithTimeout(key, asyncFunction, timeoutMs, options = {}) {
    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => reject(new Error('Таймаут операции')), timeoutMs);
    });
    
    const operationPromise = this.execute(key, asyncFunction, options);
    
    return Promise.race([operationPromise, timeoutPromise]);
  }
  
  // Выполнение с повторными попытками
  async executeWithRetry(key, asyncFunction, maxRetries = 3, delay = 1000, options = {}) {
    let lastError;
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        return await this.execute(key, asyncFunction, options);
      } catch (error) {
        lastError = error;
        
        if (attempt < maxRetries) {
          await new Promise(resolve => setTimeout(resolve, delay * (attempt + 1)));
        }
      }
    }
    
    throw lastError;
  }
}

// Использование AsyncManager
const asyncManager = new AsyncManager();

// Функция для загрузки данных пользователя
async function loadUserData(userId) {
  const response = await fetch(`/api/users/${userId}`);
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  return response.json();
}

// Загрузка с кешированием
async function loadUserWithCache(userId) {
  return asyncManager.execute(
    `user-${userId}`,
    () => loadUserData(userId),
    { cache: true, ttl: 10 * 60 * 1000 } // 10 минут
  );
}

// Загрузка с таймаутом
async function loadUserWithTimeout(userId) {
  return asyncManager.executeWithTimeout(
    `user-${userId}`,
    () => loadUserData(userId),
    5000, // 5 секунд
    { cache: true }
  );
}

// Загрузка с повторными попытками
async function loadUserWithRetry(userId) {
  return asyncManager.executeWithRetry(
    `user-${userId}`,
    () => loadUserData(userId),
    3, // 3 попытки
    1000 // 1 секунда задержка
  );
}
```

## Практический пример: Работа с DOM элементами

```javascript
// Утилиты для работы с DOM
class DOMUtils {
  // Создание элемента с атрибутами и дочерними элементами
  static createElement(tag, attributes = {}, ...children) {
    const element = document.createElement(tag);
    
    // Установка атрибутов
    Object.entries(attributes).forEach(([key, value]) => {
      if (key.startsWith('on')) {
        // Обработчики событий
        const eventName = key.slice(2).toLowerCase();
        element.addEventListener(eventName, value);
      } else if (key === 'className') {
        element.className = value;
      } else if (key === 'style' && typeof value === 'object') {
        Object.assign(element.style, value);
      } else if (key === 'dataset' && typeof value === 'object') {
        Object.entries(value).forEach(([dataKey, dataValue]) => {
          element.dataset[dataKey] = dataValue;
        });
      } else {
        element.setAttribute(key, value);
      }
    });
    
    // Добавление дочерних элементов
    children.flat().forEach(child => {
      if (typeof child === 'string' || typeof child === 'number') {
        element.appendChild(document.createTextNode(child));
      } else if (child instanceof Element) {
        element.appendChild(child);
      } else if (child && typeof child === 'object' && child.nodeType) {
        element.appendChild(child);
      }
    });
    
    return element;
  }
  
  // Поиск элементов с кэшированием
  static createCachedSelector(container = document) {
    const cache = new Map();
    
    return (selector) => {
      if (!cache.has(selector)) {
        cache.set(selector, container.querySelector(selector));
      }
      return cache.get(selector);
    };
  }
  
  // Анимация появления/исчезновения
  static fadeToggle(element, duration = 300) {
    return new Promise(resolve => {
      const isShowing = element.style.opacity !== '0';
      
      let start = null;
      const initialOpacity = isShowing ? 1 : 0;
      const targetOpacity = isShowing ? 0 : 1;
      
      function animate(timestamp) {
        if (!start) start = timestamp;
        const progress = Math.min((timestamp - start) / duration, 1);
        const currentOpacity = isShowing 
          ? initialOpacity - (progress * initialOpacity)
          : progress * targetOpacity;
        
        element.style.opacity = currentOpacity;
        
        if (progress < 1) {
          requestAnimationFrame(animate);
        } else {
          if (isShowing) {
            element.style.display = 'none';
          } else {
            element.style.display = '';
          }
          resolve();
        }
      }
      
      if (!isShowing) {
        element.style.display = 'block';
        element.style.opacity = '0';
      }
      
      requestAnimationFrame(animate);
    });
  }
  
  // Плавный скролл к элементу
  static smoothScrollTo(element, offset = 0) {
    const targetPosition = element.getBoundingClientRect().top + window.pageYOffset + offset;
    window.scrollTo({
      top: targetPosition,
      behavior: 'smooth'
    });
  }
  
  // Добавление класса с возможностью отмены
  static async addClassWithTimeout(element, className, duration) {
    element.classList.add(className);
    
    await new Promise(resolve => setTimeout(resolve, duration));
    element.classList.remove(className);
  }
  
  // Проверка видимости элемента в области просмотра
  static isElementInViewport(element) {
    const rect = element.getBoundingClientRect();
    return (
      rect.top >= 0 &&
      rect.left >= 0 &&
      rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
      rect.right <= (window.innerWidth || document.documentElement.clientWidth)
    );
  }
  
  // Отслеживание появления элемента в области просмотра
  static observeVisibility(element, callback, options = {}) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        callback(entry.isIntersecting, entry);
      });
    }, {
      threshold: options.threshold || 0.1,
      rootMargin: options.rootMargin || '0px'
    });
    
    observer.observe(element);
    return observer;
  }
}

// Пример использования DOMUtils
function initializeUI() {
  // Создание элемента
  const button = DOMUtils.createElement('button', {
    className: 'btn btn-primary',
    onClick: () => alert('Кнопка нажата!')
  }, 'Нажми меня');
  
  document.body.appendChild(button);
  
  // Анимация
  const modal = document.getElementById('modal');
  if (modal) {
    document.getElementById('open-modal').addEventListener('click', () => {
      DOMUtils.fadeToggle(modal, 300);
    });
    
    document.getElementById('close-modal').addEventListener('click', () => {
      DOMUtils.fadeToggle(modal, 300);
    });
  }
  
  // Отслеживание видимости
  const section = document.getElementById('features-section');
  if (section) {
    DOMUtils.observeVisibility(section, (isVisible) => {
      if (isVisible) {
        console.log('Секция "Возможности" появилась в области просмотра');
        // Запуск анимаций или загрузка контента
      }
    });
  }
}

// Инициализация после загрузки DOM
document.addEventListener('DOMContentLoaded', initializeUI);
```

## Практический пример: Паттерны и антипаттерны

```javascript
// Хорошие паттерны (Best Practices)

// 1. Паттерн Module для изоляции кода
const UserModule = (() => {
  let users = [];
  const listeners = [];
  
  const notifyListeners = (event, data) => {
    listeners.forEach(callback => callback(event, data));
  };
  
  return {
    addListener: (callback) => {
      listeners.push(callback);
    },
    
    addUser: (userData) => {
      const user = { ...userData, id: Date.now() };
      users.push(user);
      notifyListeners('userAdded', user);
      return user;
    },
    
    getUser: (id) => {
      return users.find(user => user.id === id);
    },
    
    updateUser: (id, updates) => {
      const index = users.findIndex(user => user.id === id);
      if (index !== -1) {
        users[index] = { ...users[index], ...updates };
        notifyListeners('userUpdated', users[index]);
        return users[index];
      }
      return null;
    },
    
    deleteUser: (id) => {
      const index = users.findIndex(user => user.id === id);
      if (index !== -1) {
        const deletedUser = users.splice(index, 1)[0];
        notifyListeners('userDeleted', deletedUser);
        return deletedUser;
      }
      return null;
    }
  };
})();

// 2. Паттерн Factory для создания элементов
const ComponentFactory = {
  createButton: (options = {}) => {
    return DOMUtils.createElement('button', {
      className: `btn ${options.type || 'btn-default'} ${options.size || ''}`,
      onClick: options.onClick,
      disabled: options.disabled
    }, options.text || 'Кнопка');
  },
  
  createInput: (options = {}) => {
    return DOMUtils.createElement('input', {
      type: options.type || 'text',
      className: `form-control ${options.className || ''}`,
      placeholder: options.placeholder,
      value: options.value,
      required: options.required
    });
  },
  
  createModal: (options = {}) => {
    const modal = DOMUtils.createElement('div', {
      className: 'modal',
      style: { display: 'none' }
    }, [
      DOMUtils.createElement('div', { className: 'modal-content' }, [
        DOMUtils.createElement('span', {
          className: 'close',
          onClick: options.onClose
        }, '×'),
        DOMUtils.createElement('h2', {}, options.title || ''),
        DOMUtils.createElement('div', {}, options.content || '')
      ])
    ]);
    
    return modal;
  }
};

// Антипаттерны (Anti-patterns) - Чего следует избегать

// 1. Глобальные переменные
// Плохо:
// var currentUserData = {};
// var appState = {};
// var config = {};

// Хорошо:
const App = {
  state: {
    currentUser: null,
    theme: 'light'
  },
  
  config: {
    apiEndpoint: 'https://api.example.com',
    version: '1.0.0'
  }
};

// 2. "Божественные" функции, делающие слишком много
// Плохо:
function processUserDataAndUI(data) {
  // Много кода, делающего разные вещи
  // 1. Валидация данных
  // 2. Сохранение в localStorage
  // 3. Обновление DOM
  // 4. Отправка на сервер
  // 5. Уведомления пользователю
}

// Хорошо:
function validateUserData(data) { /* только валидация */ }
function saveUserData(data) { /* только сохранение */ }
function updateUIWithUserData(data) { /* только обновление UI */ }
function notifyUser(message) { /* только уведомления */ }

// 3. Избыточные проверки и дублирование кода
// Плохо:
if (user && user.profile && user.profile.settings && user.profile.settings.theme) {
  document.body.className = user.profile.settings.theme;
}
if (user && user.profile && user.profile.settings && user.profile.settings.language) {
  document.documentElement.lang = user.profile.settings.language;
}

// Хорошо:
function updateUserSettings(user) {
  const settings = user?.profile?.settings || {};
  
  if (settings.theme) {
    document.body.className = settings.theme;
  }
  
  if (settings.language) {
    document.documentElement.lang = settings.language;
  }
}
```

## Заключение

Практические примеры JavaScript для фронтенд разработки показывают, как можно эффективно решать повседневные задачи, используя различные паттерны и подходы. Важно помнить о принципах чистого кода, избегать антипаттернов и стремиться к созданию поддерживаемого и тестируемого кода.

## См. также

- [[Функции в JavaScript - Продвинутые концепции]]
- [[closure_practical_examples.md]]
- [[Прототипное наследование в JavaScript]]
- [[Метапрограммирование и прокси в JavaScript]]
- [[Функциональное программирование в JS]]
- [[DOM манипуляции в JavaScript]]
- [[События в JavaScript для фронтенд разработки]]
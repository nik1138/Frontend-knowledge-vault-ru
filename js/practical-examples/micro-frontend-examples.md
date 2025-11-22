---
aliases: ["Примеры микрофронтендов", "JS Micro Frontends"]
tags: [javascript, micro-frontend, architecture, frontend]
---

# Примеры микрофронтендов

Микрофронтенды — это архитектурный подход к разработке фронтенд-приложений, при котором интерфейс разбивается на независимые части, каждая из которых может быть разработана, протестирована и развернута отдельно.

## Основы микрофронтендов

### 1. Архитектура микрофронтендов

```javascript
// Основная структура микрофронтенд приложения
class MicroFrontendApp {
  constructor() {
    this.modules = new Map();
    this.container = document.getElementById('app-container');
    this.loadOrder = [];
  }
  
  /**
   * Регистрирует модуль микрофронтенда
   * @param {string} name - имя модуля
   * @param {object} module - объект модуля с методами mount/unmount
   */
  registerModule(name, module) {
    this.modules.set(name, module);
    this.loadOrder.push(name);
  }
  
  /**
   * Монтирует модуль в DOM
   * @param {string} name - имя модуля
   * @param {HTMLElement} container - контейнер для монтирования
   */
  async mountModule(name, container) {
    const module = this.modules.get(name);
    if (!module) {
      throw new Error(`Модуль ${name} не найден`);
    }
    
    try {
      // Загрузка зависимостей модуля
      if (module.dependencies) {
        await this.loadDependencies(module.dependencies);
      }
      
      // Монтирование модуля
      await module.mount(container);
      
      console.log(`Модуль ${name} успешно смонтирован`);
    } catch (error) {
      console.error(`Ошибка при монтировании модуля ${name}:`, error);
    }
  }
  
  /**
   * Загружает зависимости модуля
   * @param {Array} dependencies - массив зависимостей
   */
  async loadDependencies(dependencies) {
    const loadPromises = dependencies.map(dep => this.loadScript(dep));
    await Promise.all(loadPromises);
  }
  
  /**
   * Загружает JavaScript скрипт динамически
   * @param {string} src - URL скрипта
   * @returns {Promise} - промис загрузки
   */
  loadScript(src) {
    return new Promise((resolve, reject) => {
      // Проверяем, не загружен ли скрипт уже
      if (document.querySelector(`script[src="${src}"]`)) {
        resolve();
        return;
      }
      
      const script = document.createElement('script');
      script.src = src;
      script.onload = resolve;
      script.onerror = reject;
      document.head.appendChild(script);
    });
  }
  
  /**
   * Загружает CSS стили динамически
   * @param {string} href - URL стилей
   * @returns {Promise} - промис загрузки
   */
  loadStyles(href) {
    return new Promise((resolve, reject) => {
      // Проверяем, не загружен ли стиль уже
      if (document.querySelector(`link[href="${href}"]`)) {
        resolve();
        return;
      }
      
      const link = document.createElement('link');
      link.rel = 'stylesheet';
      link.href = href;
      link.onload = resolve;
      link.onerror = reject;
      document.head.appendChild(link);
    });
  }
  
  /**
   * Инициализирует приложение
   */
  async initialize() {
    console.log('Инициализация микрофронтенд приложения...');
    
    // Загружаем общие зависимости
    await this.loadCommonDependencies();
    
    // Монтируем модули в порядке загрузки
    for (const moduleName of this.loadOrder) {
      const moduleContainer = document.getElementById(`${moduleName}-container`);
      if (moduleContainer) {
        await this.mountModule(moduleName, moduleContainer);
      }
    }
    
    console.log('Микрофронтенд приложение инициализировано');
  }
  
  /**
   * Загружает общие зависимости приложения
   */
  async loadCommonDependencies() {
    // Загружаем общую библиотеку для общения между модулями
    await this.loadScript('/common/micro-frontend-bridge.js');
  }
}

// Инициализация приложения
const app = new MicroFrontendApp();
```

### 2. Пример модуля микрофронтенда

```javascript
// Пример модуля "Профиль пользователя"
class UserProfileModule {
  constructor() {
    this.name = 'user-profile';
    this.version = '1.0.0';
    this.dependencies = [
      'https://cdn.example.com/react/17.0.2/react.production.min.js',
      'https://cdn.example.com/react/17.0.2/react-dom.production.min.js'
    ];
    this.styles = [
      '/modules/user-profile/styles.css'
    ];
  }
  
  /**
   * Монтирует модуль в DOM
   * @param {HTMLElement} container - контейнер для монтирования
   */
  async mount(container) {
    // Загружаем стили модуля
    for (const style of this.styles) {
      await app.loadStyles(style);
    }
    
    // Создаем DOM элементы
    container.innerHTML = `
      <div class="user-profile-module">
        <div class="profile-header">
          <h2>Профиль пользователя</h2>
        </div>
        <div class="profile-content" id="profile-content-${this.name}">
          <div class="loading">Загрузка...</div>
        </div>
      </div>
    `;
    
    // Загружаем данные пользователя
    await this.loadUserData(container);
    
    // Подписываемся на события
    this.subscribeToEvents();
  }
  
  /**
   * Загружает данные пользователя
   * @param {HTMLElement} container - контейнер модуля
   */
  async loadUserData(container) {
    try {
      const contentDiv = container.querySelector(`#profile-content-${this.name}`);
      
      // Имитация загрузки данных
      const userData = await this.fetchUserData();
      
      contentDiv.innerHTML = `
        <div class="user-info">
          <div class="avatar">
            <img src="${userData.avatar}" alt="Аватар" />
          </div>
          <div class="details">
            <h3>${userData.name}</h3>
            <p>Email: ${userData.email}</p>
            <p>Регистрация: ${userData.registrationDate}</p>
          </div>
        </div>
        <button class="edit-profile-btn">Редактировать профиль</button>
      `;
      
      // Добавляем обработчики событий
      const editBtn = contentDiv.querySelector('.edit-profile-btn');
      editBtn.addEventListener('click', () => this.editProfile());
      
    } catch (error) {
      console.error('Ошибка загрузки данных пользователя:', error);
      const contentDiv = container.querySelector(`#profile-content-${this.name}`);
      contentDiv.innerHTML = '<div class="error">Ошибка загрузки данных</div>';
    }
  }
  
  /**
   * Имитация загрузки данных пользователя
   * @returns {Promise} - промис с данными пользователя
   */
  async fetchUserData() {
    // В реальном приложении это будет API вызов
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          name: 'Иван Петров',
          email: 'ivan@example.com',
          avatar: '/images/avatar.jpg',
          registrationDate: '2023-01-15'
        });
      }, 1000);
    });
  }
  
  /**
   * Редактирование профиля
   */
  editProfile() {
    // Отправляем событие другим модулям
    window.microFrontendBridge?.emit('profile-edit-requested', {
      moduleName: this.name,
      timestamp: Date.now()
    });
  }
  
  /**
   * Подписывается на события от других модулей
   */
  subscribeToEvents() {
    window.microFrontendBridge?.on('user-updated', (data) => {
      console.log('Данные пользователя обновлены:', data);
      // Перезагружаем данные профиля
      const container = document.querySelector('.user-profile-module')?.parentElement;
      if (container) {
        this.loadUserData(container);
      }
    });
  }
}

// Регистрация модуля
const userProfileModule = new UserProfileModule();
app.registerModule('user-profile', userProfileModule);
```

### 3. Модуль навигации

```javascript
// Модуль навигации
class NavigationModule {
  constructor() {
    this.name = 'navigation';
    this.version = '1.0.0';
    this.dependencies = [];
    this.styles = ['/modules/navigation/styles.css'];
    this.menuItems = [
      { id: 'home', label: 'Главная', route: '/' },
      { id: 'profile', label: 'Профиль', route: '/profile' },
      { id: 'settings', label: 'Настройки', route: '/settings' },
      { id: 'about', label: 'О нас', route: '/about' }
    ];
  }
  
  async mount(container) {
    // Загружаем стили
    for (const style of this.styles) {
      await app.loadStyles(style);
    }
    
    // Создаем навигацию
    container.innerHTML = `
      <nav class="micro-frontend-nav">
        <div class="nav-brand">Микрофронтенд</div>
        <ul class="nav-menu">
          ${this.menuItems.map(item => `
            <li class="nav-item">
              <a href="${item.route}" class="nav-link" data-route="${item.id}">
                ${item.label}
              </a>
            </li>
          `).join('')}
        </ul>
      </nav>
    `;
    
    // Добавляем обработчики кликов
    this.addNavigationListeners();
    
    // Подписываемся на события
    this.subscribeToEvents();
  }
  
  addNavigationListeners() {
    const navLinks = document.querySelectorAll('.nav-link');
    navLinks.forEach(link => {
      link.addEventListener('click', (e) => {
        e.preventDefault();
        
        const routeId = link.getAttribute('data-route');
        const route = this.menuItems.find(item => item.id === routeId)?.route;
        
        if (route) {
          // Обновляем URL
          history.pushState({ route: routeId }, '', route);
          
          // Отправляем событие о смене маршрута
          window.microFrontendBridge?.emit('route-changed', {
            routeId,
            route,
            timestamp: Date.now()
          });
        }
      });
    });
  }
  
  subscribeToEvents() {
    // Обработка события смены маршрута
    window.microFrontendBridge?.on('route-changed', (data) => {
      this.highlightActiveRoute(data.routeId);
    });
  }
  
  highlightActiveRoute(activeRouteId) {
    const navLinks = document.querySelectorAll('.nav-link');
    navLinks.forEach(link => {
      const routeId = link.getAttribute('data-route');
      if (routeId === activeRouteId) {
        link.classList.add('active');
      } else {
        link.classList.remove('active');
      }
    });
  }
}

// Регистрация модуля навигации
const navigationModule = new NavigationModule();
app.registerModule('navigation', navigationModule);
```

## Коммуникация между модулями

### 1. Шина событий для микрофронтендов

```javascript
// Шина событий для коммуникации между модулями
class MicroFrontendBridge {
  constructor() {
    this.events = {};
    this.modules = new Set();
  }
  
  /**
   * Подписка на событие
   * @param {string} eventName - имя события
   * @param {function} callback - функция обратного вызова
   */
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
  }
  
  /**
   * Отписка от события
   * @param {string} eventName - имя события
   * @param {function} callback - функция обратного вызова
   */
  off(eventName, callback) {
    if (this.events[eventName]) {
      this.events[eventName] = this.events[eventName].filter(cb => cb !== callback);
    }
  }
  
  /**
   * Вызов события
   * @param {string} eventName - имя события
   * @param {*} data - данные события
   */
  emit(eventName, data) {
    if (this.events[eventName]) {
      this.events[eventName].forEach(callback => {
        try {
          callback(data);
        } catch (error) {
          console.error(`Ошибка при обработке события ${eventName}:`, error);
        }
      });
    }
  }
  
  /**
   * Регистрация модуля
   * @param {string} moduleName - имя модуля
   */
  registerModule(moduleName) {
    this.modules.add(moduleName);
    console.log(`Модуль ${moduleName} зарегистрирован`);
  }
  
  /**
   * Получение списка зарегистрированных модулей
   * @returns {Array} - массив имен модулей
   */
  getRegisteredModules() {
    return Array.from(this.modules);
  }
  
  /**
   * Проверка, зарегистрирован ли модуль
   * @param {string} moduleName - имя модуля
   * @returns {boolean} - true если модуль зарегистрирован
   */
  isModuleRegistered(moduleName) {
    return this.modules.has(moduleName);
  }
}

// Глобальная шина событий
window.microFrontendBridge = new MicroFrontendBridge();
```

### 2. Модуль обмена данными

```javascript
// Модуль управления общими данными
class SharedDataManager {
  constructor() {
    this.dataStore = new Map();
    this.subscribers = new Map(); // подписчики на изменения данных
    this.bridge = window.microFrontendBridge;
    
    // Подписываемся на события обновления данных
    this.bridge?.on('data-updated', (data) => {
      this.handleDataUpdate(data);
    });
  }
  
  /**
   * Устанавливает значение в хранилище
   * @param {string} key - ключ
   * @param {*} value - значение
   * @param {string} sourceModule - имя модуля-источника
   */
  set(key, value, sourceModule) {
    const oldValue = this.dataStore.get(key);
    this.dataStore.set(key, value);
    
    // Отправляем событие об изменении данных
    this.bridge?.emit('data-updated', {
      key,
      value,
      oldValue,
      sourceModule,
      timestamp: Date.now()
    });
  }
  
  /**
   * Получает значение из хранилища
   * @param {string} key - ключ
   * @returns {*} - значение
   */
  get(key) {
    return this.dataStore.get(key);
  }
  
  /**
   * Удаляет значение из хранилища
   * @param {string} key - ключ
   * @param {string} sourceModule - имя модуля-источника
   */
  delete(key, sourceModule) {
    const value = this.dataStore.get(key);
    const deleted = this.dataStore.delete(key);
    
    if (deleted) {
      this.bridge?.emit('data-deleted', {
        key,
        value,
        sourceModule,
        timestamp: Date.now()
      });
    }
  }
  
  /**
   * Подписывается на изменения конкретного ключа
   * @param {string} key - ключ для отслеживания
   * @param {function} callback - функция обратного вызова
   * @param {string} subscriberModule - имя модуля-подписчика
   */
  subscribe(key, callback, subscriberModule) {
    if (!this.subscribers.has(key)) {
      this.subscribers.set(key, []);
    }
    
    this.subscribers.get(key).push({
      callback,
      module: subscriberModule
    });
  }
  
  /**
   * Отписывается от изменений ключа
   * @param {string} key - ключ
   * @param {function} callback - функция обратного вызова
   * @param {string} subscriberModule - имя модуля-подписчика
   */
  unsubscribe(key, callback, subscriberModule) {
    if (this.subscribers.has(key)) {
      const subscribers = this.subscribers.get(key);
      this.subscribers.set(key, subscribers.filter(sub => 
        !(sub.callback === callback && sub.module === subscriberModule)
      ));
    }
  }
  
  /**
   * Обрабатывает обновление данных
   * @param {object} data - данные обновления
   */
  handleDataUpdate(data) {
    const { key } = data;
    
    if (this.subscribers.has(key)) {
      this.subscribers.get(key).forEach(subscriber => {
        try {
          subscriber.callback(data);
        } catch (error) {
          console.error(`Ошибка при уведомлении подписчика ${subscriber.module}:`, error);
        }
      });
    }
  }
  
  /**
   * Получает все ключи в хранилище
   * @returns {Array} - массив ключей
   */
  getKeys() {
    return Array.from(this.dataStore.keys());
  }
  
  /**
   * Очищает хранилище
   */
  clear() {
    this.dataStore.clear();
  }
}

// Глобальный менеджер общих данных
window.sharedDataManager = new SharedDataManager();
```

## Загрузка модулей по требованию

### 1. Lazy loading модулей

```javascript
// Менеджер ленивой загрузки модулей
class LazyModuleLoader {
  constructor() {
    this.loadedModules = new Map();
    this.loadingPromises = new Map();
  }
  
  /**
   * Загружает модуль по требованию
   * @param {string} moduleName - имя модуля
   * @param {string} modulePath - путь к модулю
   * @returns {Promise} - промис с модулем
   */
  async loadModule(moduleName, modulePath) {
    // Проверяем, не загружен ли модуль уже
    if (this.loadedModules.has(moduleName)) {
      return this.loadedModules.get(moduleName);
    }
    
    // Проверяем, не загружается ли модуль уже
    if (this.loadingPromises.has(moduleName)) {
      return this.loadingPromises.get(moduleName);
    }
    
    // Создаем промис загрузки
    const loadPromise = this.importModule(modulePath)
      .then(module => {
        this.loadedModules.set(moduleName, module);
        this.loadingPromises.delete(moduleName);
        return module;
      })
      .catch(error => {
        console.error(`Ошибка загрузки модуля ${moduleName}:`, error);
        this.loadingPromises.delete(moduleName);
        throw error;
      });
    
    this.loadingPromises.set(moduleName, loadPromise);
    return loadPromise;
  }
  
  /**
   * Импортирует модуль динамически
   * @param {string} path - путь к модулю
   * @returns {Promise} - промис с модулем
   */
  async importModule(path) {
    try {
      // Используем динамический import для загрузки модуля
      const module = await import(path);
      return module.default || module;
    } catch (error) {
      console.error(`Ошибка импорта модуля ${path}:`, error);
      throw error;
    }
  }
  
  /**
   * Проверяет, загружен ли модуль
   * @param {string} moduleName - имя модуля
   * @returns {boolean} - true если модуль загружен
   */
  isModuleLoaded(moduleName) {
    return this.loadedModules.has(moduleName);
  }
  
  /**
   * Выгружает модуль из памяти
   * @param {string} moduleName - имя модуля
   */
  unloadModule(moduleName) {
    if (this.loadedModules.has(moduleName)) {
      // Удаляем модуль из кэша
      this.loadedModules.delete(moduleName);
      
      // Если модуль был в DOM, удаляем его
      const moduleElement = document.querySelector(`[data-module="${moduleName}"]`);
      if (moduleElement) {
        moduleElement.remove();
      }
    }
  }
}

// Глобальный загрузчик модулей
window.lazyModuleLoader = new LazyModuleLoader();

// Пример использования lazy loading
async function loadUserProfileModule() {
  const module = await window.lazyModuleLoader.loadModule(
    'user-profile', 
    '/modules/user-profile/index.js'
  );
  
  // Монтируем модуль
  const container = document.getElementById('user-profile-container');
  if (container) {
    await module.mount(container);
  }
}
```

### 2. Управление жизненным циклом модулей

```javascript
// Менеджер жизненного цикла модулей
class ModuleLifecycleManager {
  constructor() {
    this.modules = new Map();
    this.lifecycleHooks = new Map(); // хуки жизненного цикла
  }
  
  /**
   * Регистрирует модуль
   * @param {string} name - имя модуля
   * @param {object} module - объект модуля
   */
  register(name, module) {
    this.modules.set(name, {
      instance: module,
      state: 'registered',
      createdAt: Date.now()
    });
    
    // Регистрируем хуки жизненного цикла
    this.registerLifecycleHooks(name, module);
  }
  
  /**
   * Регистрирует хуки жизненного цикла модуля
   * @param {string} name - имя модуля
   * @param {object} module - объект модуля
   */
  registerLifecycleHooks(name, module) {
    const hooks = {
      beforeMount: module.beforeMount?.bind(module) || (() => Promise.resolve()),
      afterMount: module.afterMount?.bind(module) || (() => Promise.resolve()),
      beforeUnmount: module.beforeUnmount?.bind(module) || (() => Promise.resolve()),
      afterUnmount: module.afterUnmount?.bind(module) || (() => Promise.resolve())
    };
    
    this.lifecycleHooks.set(name, hooks);
  }
  
  /**
   * Монтирует модуль
   * @param {string} name - имя модуля
   * @param {HTMLElement} container - контейнер для монтирования
   */
  async mount(name, container) {
    const moduleInfo = this.modules.get(name);
    if (!moduleInfo) {
      throw new Error(`Модуль ${name} не зарегистрирован`);
    }
    
    if (moduleInfo.state !== 'registered' && moduleInfo.state !== 'unmounted') {
      throw new Error(`Модуль ${name} уже смонтирован`);
    }
    
    const hooks = this.lifecycleHooks.get(name);
    
    try {
      // Вызываем хук перед монтированием
      await hooks.beforeMount();
      
      // Монтируем модуль
      await moduleInfo.instance.mount(container);
      
      // Обновляем состояние
      moduleInfo.state = 'mounted';
      moduleInfo.mountedAt = Date.now();
      moduleInfo.container = container;
      
      // Вызываем хук после монтирования
      await hooks.afterMount();
      
      console.log(`Модуль ${name} успешно смонтирован`);
    } catch (error) {
      console.error(`Ошибка монтирования модуля ${name}:`, error);
      throw error;
    }
  }
  
  /**
   * Отмонтирование модуля
   * @param {string} name - имя модуля
   */
  async unmount(name) {
    const moduleInfo = this.modules.get(name);
    if (!moduleInfo || moduleInfo.state !== 'mounted') {
      console.warn(`Модуль ${name} не смонтирован или не существует`);
      return;
    }
    
    const hooks = this.lifecycleHooks.get(name);
    
    try {
      // Вызываем хук перед отмонтированием
      await hooks.beforeUnmount();
      
      // Вызываем метод unmount модуля, если он существует
      if (moduleInfo.instance.unmount) {
        await moduleInfo.instance.unmount();
      }
      
      // Удаляем модуль из DOM
      if (moduleInfo.container) {
        moduleInfo.container.innerHTML = '';
      }
      
      // Обновляем состояние
      moduleInfo.state = 'unmounted';
      moduleInfo.unmountedAt = Date.now();
      
      // Вызываем хук после отмонтирования
      await hooks.afterUnmount();
      
      console.log(`Модуль ${name} успешно отмонтирован`);
    } catch (error) {
      console.error(`Ошибка отмонтирования модуля ${name}:`, error);
      throw error;
    }
  }
  
  /**
   * Получает информацию о модуле
   * @param {string} name - имя модуля
   * @returns {object} - информация о модуле
   */
  getModuleInfo(name) {
    return this.modules.get(name);
  }
  
  /**
   * Получает список всех модулей
   * @returns {Array} - массив имен модулей
   */
  getAllModuleNames() {
    return Array.from(this.modules.keys());
  }
  
  /**
   * Проверяет, смонтирован ли модуль
   * @param {string} name - имя модуля
   * @returns {boolean} - true если модуль смонтирован
   */
  isMounted(name) {
    const moduleInfo = this.modules.get(name);
    return moduleInfo && moduleInfo.state === 'mounted';
  }
}

// Глобальный менеджер жизненного цикла
window.moduleLifecycleManager = new ModuleLifecycleManager();
```

## Практические примеры использования

### 1. Пример полного приложения

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Микрофронтенд Приложение</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
        }
        
        #app-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .module-container {
            margin-bottom: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
        }
        
        .micro-frontend-nav {
            background-color: #333;
            padding: 1rem;
            margin-bottom: 20px;
        }
        
        .micro-frontend-nav .nav-brand {
            color: white;
            font-size: 1.5rem;
            margin-bottom: 10px;
        }
        
        .nav-menu {
            list-style: none;
            padding: 0;
            margin: 0;
            display: flex;
            gap: 20px;
        }
        
        .nav-link {
            color: white;
            text-decoration: none;
            padding: 8px 16px;
            border-radius: 4px;
            transition: background-color 0.3s;
        }
        
        .nav-link:hover, .nav-link.active {
            background-color: #555;
        }
        
        .user-profile-module {
            padding: 20px;
        }
        
        .user-info {
            display: flex;
            align-items: center;
            gap: 15px;
        }
        
        .avatar img {
            width: 80px;
            height: 80px;
            border-radius: 50%;
        }
        
        .loading {
            text-align: center;
            padding: 20px;
            color: #666;
        }
        
        .error {
            color: red;
            text-align: center;
            padding: 20px;
        }
    </style>
</head>
<body>
    <div id="app-container">
        <div id="navigation-container" class="module-container"></div>
        <div id="user-profile-container" class="module-container"></div>
        <div id="settings-container" class="module-container"></div>
    </div>

    <script>
        // Инициализация приложения при загрузке DOM
        document.addEventListener('DOMContentLoaded', () => {
            app.initialize().catch(error => {
                console.error('Ошибка инициализации приложения:', error);
            });
        });
    </script>
</body>
</html>
```

### 2. Интеграция с современными фреймворками

```javascript
// Пример интеграции с React
class ReactMicroFrontendModule {
  constructor(componentFactory, dependencies = []) {
    this.componentFactory = componentFactory;
    this.dependencies = [
      'https://unpkg.com/react@18/umd/react.production.min.js',
      'https://unpkg.com/react-dom@18/umd/react-dom.production.min.js',
      ...dependencies
    ];
  }
  
  async mount(container) {
    // Загружаем зависимости React
    for (const dep of this.dependencies) {
      await app.loadScript(dep);
    }
    
    // Создаем контейнер для React компонента
    const reactContainer = document.createElement('div');
    container.appendChild(reactContainer);
    
    // Монтируем React компонент
    const React = window.React;
    const ReactDOM = window.ReactDOM;
    
    if (React && ReactDOM && typeof this.componentFactory === 'function') {
      const component = this.componentFactory();
      ReactDOM.render(component, reactContainer);
    }
  }
  
  async unmount() {
    // Очистка React компонента
    if (window.ReactDOM) {
      window.ReactDOM.unmountComponentAtNode(this.container);
    }
  }
}

// Пример использования с React
const UserProfileReactComponent = () => {
  const [user, setUser] = window.React.useState(null);
  
  window.React.useEffect(() => {
    // Загрузка данных пользователя
    const fetchUser = async () => {
      // Имитация загрузки данных
      const userData = await new Promise(resolve => 
        setTimeout(() => resolve({ name: 'Иван', email: 'ivan@example.com' }), 1000)
      );
      setUser(userData);
    };
    
    fetchUser();
  }, []);
  
  if (!user) {
    return window.React.createElement('div', { className: 'loading' }, 'Загрузка...');
  }
  
  return window.React.createElement('div', { className: 'user-profile' },
    window.React.createElement('h2', null, user.name),
    window.React.createElement('p', null, `Email: ${user.email}`)
  );
};

// Регистрация React модуля
const reactModule = new ReactMicroFrontendModule(() => UserProfileReactComponent);
app.registerModule('react-user-profile', reactModule);
```

## Практические советы

- Разделяйте ответственность между модулями по бизнес-функциям
- Используйте контракты API для коммуникации между модулями
- Обеспечьте изоляцию стилей между модулями
- Реализуйте надежную систему обработки ошибок
- Используйте систему логирования для отладки
- Планируйте стратегию деплоя и версионирования модулей
- Обеспечьте совместимость между версиями модулей

## Связанные темы

- [[frontend-architecture-patterns]]
- [[module-federation-guide]]
- [[frontend-security-basics]]
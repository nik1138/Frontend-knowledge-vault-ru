# Комплексное упражнение: Создание мини-фреймворка

## Создание мини-фреймворка, объединяющего все изученные концепции

```javascript
// mini-framework.js - мини-фреймворк
class MiniFramework {
  constructor() {
    this.components = new Map();
    this.services = new Map();
    this.middlewares = [];
    this.routes = new Map();
  }
  
  // Регистрация компонента
  component(name, factory) {
    if (typeof factory !== 'function') {
      throw new Error('Фабрика компонента должна быть функцией');
    }
    
    this.components.set(name, factory);
    return this;
  }
  
  // Получение компонента
  getComponent(name, ...args) {
    const factory = this.components.get(name);
    if (!factory) {
      throw new Error(`Компонент ${name} не найден`);
    }
    
    return factory(...args);
  }
  
  // Регистрация сервиса
  service(name, factory) {
    if (typeof factory !== 'function') {
      throw new Error('Фабрика сервиса должна быть функцией');
    }
    
    this.services.set(name, {
      factory,
      instance: null,
      singleton: false
    });
    
    return this;
  }
  
  // Регистрация синглтон-сервиса
  singleton(name, factory) {
    this.service(name, factory);
    this.services.get(name).singleton = true;
    return this;
  }
  
  // Получение сервиса
  getService(name) {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Сервис ${name} не найден`);
    }
    
    if (service.singleton) {
      if (!service.instance) {
        service.instance = service.factory();
      }
      return service.instance;
    }
    
    return service.factory();
  }
  
  // Добавление middleware
  use(middleware) {
    if (typeof middleware !== 'function') {
      throw new Error('Middleware должен быть функцией');
    }
    
    this.middlewares.push(middleware);
    return this;
  }
  
  // Регистрация маршрута
  route(path, handler) {
    if (typeof handler !== 'function') {
      throw new Error('Обработчик маршрута должен быть функцией');
    }
    
    this.routes.set(path, handler);
    return this;
  }
  
  // Обработка запроса
  async handleRequest(path, context = {}) {
    // Применяем middleware
    let currentContext = { ...context, path };
    
    for (const middleware of this.middlewares) {
      try {
        currentContext = await middleware(currentContext);
        if (!currentContext) {
          throw new Error('Middleware должен возвращать контекст');
        }
      } catch (error) {
        console.error('Ошибка в middleware:', error);
        throw error;
      }
    }
    
    // Находим обработчик маршрута
    const handler = this.routes.get(path);
    if (!handler) {
      throw new Error(`Маршрут ${path} не найден`);
    }
    
    // Выполняем обработчик
    try {
      const result = await handler(currentContext);
      return result;
    } catch (error) {
      console.error('Ошибка в обработчике маршрута:', error);
      throw error;
    }
  }
}

// logger-service.js - сервис логирования
class LoggerService {
  constructor(level = 'info') {
    this.level = level;
    this.levels = {
      error: 0,
      warn: 1,
      info: 2,
      debug: 3
    };
  }
  
  _shouldLog(messageLevel) {
    return this.levels[messageLevel] <= this.levels[this.level];
  }
  
  error(message, ...args) {
    if (this._shouldLog('error')) {
      console.error(`[ERROR] ${new Date().toISOString()}: ${message}`, ...args);
    }
  }
  
  warn(message, ...args) {
    if (this._shouldLog('warn')) {
      console.warn(`[WARN] ${new Date().toISOString()}: ${message}`, ...args);
    }
  }
  
  info(message, ...args) {
    if (this._shouldLog('info')) {
      console.info(`[INFO] ${new Date().toISOString()}: ${message}`, ...args);
    }
  }
  
  debug(message, ...args) {
    if (this._shouldLog('debug')) {
      console.debug(`[DEBUG] ${new Date().toISOString()}: ${message}`, ...args);
    }
  }
}

// http-client.js - HTTP клиент
class HttpClient {
  constructor(baseURL = '', defaultOptions = {}) {
    this.baseURL = baseURL.replace(/\/$/, '');
    this.defaultOptions = {
      timeout: 5000,
      headers: {
        'Content-Type': 'application/json'
      },
      ...defaultOptions
    };
  }
  
  async request(url, options = {}) {
    const fullURL = url.startsWith('http') ? url : `${this.baseURL}${url}`;
    const config = {
      ...this.defaultOptions,
      ...options
    };
    
    // Преобразуем тело запроса
    if (config.body && typeof config.body === 'object') {
      config.body = JSON.stringify(config.body);
    }
    
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), config.timeout);
      
      const response = await fetch(fullURL, {
        ...config,
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      
      const contentType = response.headers.get('content-type');
      let data;
      
      if (contentType && contentType.includes('application/json')) {
        data = await response.json();
      } else {
        data = await response.text();
      }
      
      return {
        status: response.status,
        statusText: response.statusText,
        headers: response.headers,
        data
      };
    } catch (error) {
      if (error.name === 'AbortError') {
        throw new Error('Таймаут запроса');
      }
      throw error;
    }
  }
  
  get(url, options = {}) {
    return this.request(url, { ...options, method: 'GET' });
  }
  
  post(url, data, options = {}) {
    return this.request(url, { 
      ...options, 
      method: 'POST', 
      body: data 
    });
  }
  
  put(url, data, options = {}) {
    return this.request(url, { 
      ...options, 
      method: 'PUT', 
      body: data 
    });
  }
  
  delete(url, options = {}) {
    return this.request(url, { ...options, method: 'DELETE' });
  }
}

// data-service.js - сервис данных
class DataService {
  constructor(httpClient) {
    this.http = httpClient;
  }
  
  async getUsers() {
    const response = await this.http.get('/users');
    return response.data;
  }
  
  async getUser(id) {
    const response = await this.http.get(`/users/${id}`);
    return response.data;
  }
  
  async createUser(userData) {
    const response = await this.http.post('/users', userData);
    return response.data;
  }
  
  async updateUser(id, userData) {
    const response = await this.http.put(`/users/${id}`, userData);
    return response.data;
  }
  
  async deleteUser(id) {
    await this.http.delete(`/users/${id}`);
    return true;
  }
}

// Тестирование мини-фреймворка
async function testMiniFramework() {
  console.log('=== Тестирование мини-фреймворка ===');
  
  // Создание фреймворка
  const app = new MiniFramework();
  
  // Регистрация сервисов
  app
    .singleton('logger', () => new LoggerService('debug'))
    .singleton('httpClient', () => new HttpClient('https://jsonplaceholder.typicode.com'))
    .service('dataService', () => {
      const http = app.getService('httpClient');
      return new DataService(http);
    });
  
  // Регистрация компонентов
  app
    .component('userCard', (user) => {
      return {
        render: () => `
          <div class="user-card">
            <h3>${user.name}</h3>
            <p>Email: ${user.email}</p>
            <p>Телефон: ${user.phone}</p>
          </div>
        `
      };
    })
    .component('userList', (users) => {
      return {
        render: () => {
          const logger = app.getService('logger');
          logger.debug('Рендер списка пользователей', { count: users.length });
          
          return `
            <div class="user-list">
              ${users.map(user => {
                const card = app.getComponent('userCard', user);
                return card.render();
              }).join('')}
            </div>
          `;
        }
      };
    });
  
  // Добавление middleware
  app
    .use(async (context) => {
      const logger = app.getService('logger');
      logger.info(`Запрос к ${context.path}`);
      return { ...context, startTime: Date.now() };
    })
    .use(async (context) => {
      // Имитация аутентификации
      return { ...context, user: { id: 1, name: 'Тестовый пользователь' } };
    });
  
  // Регистрация маршрутов
  app
    .route('/users', async (context) => {
      const logger = app.getService('logger');
      const dataService = app.getService('dataService');
      
      try {
        logger.info('Получение списка пользователей');
        const users = await dataService.getUsers();
        logger.info(`Получено ${users.length} пользователей`);
        
        const userList = app.getComponent('userList', users.slice(0, 3)); // Только первые 3 для теста
        return userList.render();
      } catch (error) {
        logger.error('Ошибка при получении пользователей', error);
        throw error;
      }
    })
    .route('/users/:id', async (context) => {
      const logger = app.getService('logger');
      const dataService = app.getService('dataService');
      const userId = context.path.split('/').pop();
      
      try {
        logger.info(`Получение пользователя ${userId}`);
        const user = await dataService.getUser(userId);
        
        const userCard = app.getComponent('userCard', user);
        return userCard.render();
      } catch (error) {
        logger.error(`Ошибка при получении пользователя ${userId}`, error);
        throw error;
      }
    });
  
  // Тестирование
  try {
    console.log('Тестирование маршрута /users:');
    const result = await app.handleRequest('/users');
    console.log('Результат рендеринга (первые 200 символов):');
    console.log(result.substring(0, 200) + '...');
  } catch (error) {
    console.error('Ошибка в тесте:', error.message);
  }
}

// Раскомментируйте для запуска тестов
// testMiniFramework();
```

#javascript #framework #advanced #components #services
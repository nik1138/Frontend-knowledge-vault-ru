# Упражнения по асинхронному программированию

## Упражнение 3: Создание асинхронного API клиента
Создайте клиент для работы с REST API:

```javascript
// api-client.js - асинхронный клиент API
class ApiClient {
  constructor(baseUrl, options = {}) {
    this.baseUrl = baseUrl.replace(/\/$/, ''); // Убираем завершающий слеш
    this.defaultOptions = {
      timeout: 5000,
      retries: 3,
      ...options
    };
  }
  
  // Вспомогательный метод для создания запроса
  async _makeRequest(url, options = {}) {
    const fullUrl = url.startsWith('http') ? url : `${this.baseUrl}${url}`;
    const config = {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...this.defaultOptions,
      ...options
    };
    
    // Добавляем тело запроса для POST/PUT
    if (config.body && typeof config.body === 'object') {
      config.body = JSON.stringify(config.body);
    }
    
    let lastError;
    
    // Повторные попытки
    for (let i = 0; i <= this.defaultOptions.retries; i++) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), config.timeout);
        
        const response = await fetch(fullUrl, {
          ...config,
          signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const contentType = response.headers.get('content-type');
        if (contentType && contentType.includes('application/json')) {
          return await response.json();
        }
        
        return await response.text();
      } catch (error) {
        lastError = error;
        
        // Если это последняя попытка или ошибка не подлежит повтору
        if (i === this.defaultOptions.retries || 
            error.name === 'AbortError' || 
            (error.message && error.message.includes('HTTP'))) {
          throw new Error(`Запрос не удался после ${i + 1} попыток: ${error.message}`);
        }
        
        // Ждем перед следующей попыткой
        await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
      }
    }
    
    throw lastError;
  }
  
  // Методы HTTP запросов
  async get(url, options = {}) {
    return this._makeRequest(url, { ...options, method: 'GET' });
  }
  
  async post(url, data, options = {}) {
    return this._makeRequest(url, { 
      ...options, 
      method: 'POST', 
      body: data 
    });
  }
  
  async put(url, data, options = {}) {
    return this._makeRequest(url, { 
      ...options, 
      method: 'PUT', 
      body: data 
    });
  }
  
  async delete(url, options = {}) {
    return this._makeRequest(url, { ...options, method: 'DELETE' });
  }
  
  // Пакетные запросы
  async batch(requests) {
    const results = [];
    const errors = [];
    
    // Параллельное выполнение запросов
    const promises = requests.map(async (request, index) => {
      try {
        const { method = 'GET', url, data, options = {} } = request;
        let result;
        
        switch (method.toUpperCase()) {
          case 'GET':
            result = await this.get(url, options);
            break;
          case 'POST':
            result = await this.post(url, data, options);
            break;
          case 'PUT':
            result = await this.put(url, data, options);
            break;
          case 'DELETE':
            result = await this.delete(url, options);
            break;
          default:
            throw new Error(`Неподдерживаемый метод: ${method}`);
        }
        
        results[index] = { success: true, data: result };
      } catch (error) {
        errors[index] = { success: false, error: error.message };
        results[index] = { success: false, error: error.message };
      }
    });
    
    await Promise.allSettled(promises);
    return results;
  }
  
  // Последовательное выполнение запросов
  async sequence(requests) {
    const results = [];
    
    for (const request of requests) {
      try {
        const { method = 'GET', url, data, options = {} } = request;
        let result;
        
        switch (method.toUpperCase()) {
          case 'GET':
            result = await this.get(url, options);
            break;
          case 'POST':
            result = await this.post(url, data, options);
            break;
          case 'PUT':
            result = await this.put(url, data, options);
            break;
          case 'DELETE':
            result = await this.delete(url, options);
            break;
          default:
            throw new Error(`Неподдерживаемый метод: ${method}`);
        }
        
        results.push({ success: true, data: result });
      } catch (error) {
        results.push({ success: false, error: error.message });
        // Можно остановить выполнение при ошибке
        // throw error;
      }
    }
    
    return results;
  }
}

// data-service.js - сервис для работы с данными
class DataService {
  constructor(apiClient) {
    this.api = apiClient;
  }
  
  // Работа с пользователями
  async getUsers() {
    try {
      const users = await this.api.get('/users');
      return users;
    } catch (error) {
      console.error('Ошибка при получении пользователей:', error.message);
      throw error;
    }
  }
  
  async getUser(id) {
    try {
      const user = await this.api.get(`/users/${id}`);
      return user;
    } catch (error) {
      console.error(`Ошибка при получении пользователя ${id}:`, error.message);
      throw error;
    }
  }
  
  async createUser(userData) {
    try {
      const user = await this.api.post('/users', userData);
      return user;
    } catch (error) {
      console.error('Ошибка при создании пользователя:', error.message);
      throw error;
    }
  }
  
  async updateUser(id, userData) {
    try {
      const user = await this.api.put(`/users/${id}`, userData);
      return user;
    } catch (error) {
      console.error(`Ошибка при обновлении пользователя ${id}:`, error.message);
      throw error;
    }
  }
  
  async deleteUser(id) {
    try {
      await this.api.delete(`/users/${id}`);
      return true;
    } catch (error) {
      console.error(`Ошибка при удалении пользователя ${id}:`, error.message);
      throw error;
    }
  }
  
  // Работа с постами
  async getUserPosts(userId) {
    try {
      const posts = await this.api.get(`/users/${userId}/posts`);
      return posts;
    } catch (error) {
      console.error(`Ошибка при получении постов пользователя ${userId}:`, error.message);
      throw error;
    }
  }
  
  // Пакетные операции
  async batchGetUsers(userIds) {
    const requests = userIds.map(id => ({
      method: 'GET',
      url: `/users/${id}`
    }));
    
    const results = await this.api.batch(requests);
    return results;
  }
  
  // Поиск с пагинацией
  async searchUsers(query, page = 1, limit = 10) {
    try {
      const users = await this.api.get(`/users?search=${encodeURIComponent(query)}&page=${page}&limit=${limit}`);
      return users;
    } catch (error) {
      console.error('Ошибка при поиске пользователей:', error.message);
      throw error;
    }
  }
}

// cache-service.js - сервис кэширования
class CacheService {
  constructor(ttl = 60000) { // TTL по умолчанию 1 минута
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  // Получение данных из кэша
  get(key) {
    const item = this.cache.get(key);
    if (!item) return null;
    
    if (Date.now() - item.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
  
  // Сохранение данных в кэш
  set(key, value) {
    this.cache.set(key, {
      value,
      timestamp: Date.now()
    });
  }
  
  // Удаление данных из кэша
  delete(key) {
    this.cache.delete(key);
  }
  
  // Очистка устаревших записей
  cleanup() {
    const now = Date.now();
    for (const [key, item] of this.cache.entries()) {
      if (now - item.timestamp > this.ttl) {
        this.cache.delete(key);
      }
    }
  }
  
  // Очистка всего кэша
  clear() {
    this.cache.clear();
  }
  
  // Получение статистики
  getStats() {
    return {
      size: this.cache.size,
      ttl: this.ttl
    };
  }
}

// Тестирование асинхронных сервисов
async function testAsyncServices() {
  console.log('=== Тестирование асинхронных сервисов ===');
  
  // Создание клиента API
  const apiClient = new ApiClient('https://jsonplaceholder.typicode.com');
  
  // Создание сервисов данных и кэширования
  const dataService = new DataService(apiClient);
  const cacheService = new CacheService(30000); // 30 секунд TTL
  
  try {
    // Тестирование получения пользователей
    console.log('Получение списка пользователей...');
    const users = await dataService.getUsers();
    console.log(`Получено ${users.length} пользователей`);
    
    // Кэширование первого пользователя
    const firstUser = users[0];
    cacheService.set(`user_${firstUser.id}`, firstUser);
    console.log('Первый пользователь закэширован');
    
    // Получение пользователя из кэша
    const cachedUser = cacheService.get(`user_${firstUser.id}`);
    if (cachedUser) {
      console.log('Пользователь получен из кэша:', cachedUser.name);
    }
    
    // Получение постов пользователя
    console.log('Получение постов пользователя...');
    const posts = await dataService.getUserPosts(firstUser.id);
    console.log(`Получено ${posts.length} постов`);
    
    // Пакетное получение нескольких пользователей
    console.log('Пакетное получение пользователей...');
    const userIds = [1, 2, 3];
    const batchResults = await dataService.batchGetUsers(userIds);
    const successfulRequests = batchResults.filter(r => r.success).length;
    console.log(`Успешно получено ${successfulRequests} из ${userIds.length} пользователей`);
    
    // Тестирование создания пользователя
    console.log('Создание нового пользователя...');
    const newUser = {
      name: 'Тестовый Пользователь',
      username: 'testuser',
      email: 'test@example.com'
    };
    
    // Имитация создания (на самом деле jsonplaceholder не сохраняет данные)
    try {
      const createdUser = await dataService.createUser(newUser);
      console.log('Пользователь создан:', createdUser.id);
    } catch (error) {
      console.log('Создание пользователя (имитация):', error.message);
    }
    
    // Статистика кэша
    console.log('Статистика кэша:', cacheService.getStats());
    
  } catch (error) {
    console.error('Ошибка в тестах:', error.message);
  }
}

// Раскомментируйте для запуска тестов
// testAsyncServices();
```

#javascript #async #promise #api #advanced
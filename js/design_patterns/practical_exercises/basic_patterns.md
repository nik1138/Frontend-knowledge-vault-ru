# Практические упражнения: Основные паттерны проектирования

## Упражнение 1: Модульный паттерн

### Задача
Создайте модуль для управления корзиной покупок интернет-магазина, используя модульный паттерн для инкапсуляции данных и методов.

### Требования
1. Инкапсуляция внутреннего состояния корзины
2. Публичные методы для добавления/удаления товаров
3. Методы для получения информации о корзине
4. Защита от прямого доступа к внутренним данным

### Решение
```javascript
const ShoppingCart = (function() {
  // Приватные переменные
  let items = [];
  let totalPrice = 0;
  
  // Приватные методы
  function calculateTotal() {
    totalPrice = items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
    return totalPrice;
  }
  
  function findItemById(id) {
    return items.findIndex(item => item.id === id);
  }
  
  // Публичный API
  return {
    // Добавить товар в корзину
    addItem(item) {
      const existingItemIndex = findItemById(item.id);
      
      if (existingItemIndex >= 0) {
        items[existingItemIndex].quantity += item.quantity;
      } else {
        items.push({ ...item });
      }
      
      calculateTotal();
      return this;
    },
    
    // Удалить товар из корзины
    removeItem(id) {
      const itemIndex = findItemById(id);
      
      if (itemIndex >= 0) {
        items.splice(itemIndex, 1);
        calculateTotal();
      }
      
      return this;
    },
    
    // Обновить количество товара
    updateQuantity(id, quantity) {
      const itemIndex = findItemById(id);
      
      if (itemIndex >= 0 && quantity > 0) {
        items[itemIndex].quantity = quantity;
        calculateTotal();
      } else if (itemIndex >= 0 && quantity <= 0) {
        this.removeItem(id);
      }
      
      return this;
    },
    
    // Получить все товары в корзине
    getItems() {
      return [...items]; // Возвращаем копию массива
    },
    
    // Получить общую стоимость
    getTotalPrice() {
      return totalPrice;
    },
    
    // Получить количество товаров
    getItemCount() {
      return items.reduce((count, item) => count + item.quantity, 0);
    },
    
    // Очистить корзину
    clear() {
      items = [];
      totalPrice = 0;
      return this;
    },
    
    // Получить информацию о корзине
    getInfo() {
      return {
        items: this.getItems(),
        totalItems: this.getItemCount(),
        totalPrice: this.getTotalPrice()
      };
    }
  };
})();

// Пример использования
ShoppingCart
  .addItem({ id: 1, name: "Ноутбук", price: 1000, quantity: 1 })
  .addItem({ id: 2, name: "Мышь", price: 25, quantity: 2 })
  .addItem({ id: 3, name: "Клавиатура", price: 75, quantity: 1 });

console.log(ShoppingCart.getInfo());
// {
//   items: [
//     { id: 1, name: "Ноутбук", price: 1000, quantity: 1 },
//     { id: 2, name: "Мышь", price: 25, quantity: 2 },
//     { id: 3, name: "Клавиатура", price: 75, quantity: 1 }
//   ],
//   totalItems: 4,
//   totalPrice: 1125
// }

ShoppingCart.updateQuantity(2, 1);
console.log(ShoppingCart.getTotalPrice()); // 1100

ShoppingCart.removeItem(3);
console.log(ShoppingCart.getItemCount()); // 2
```

## Упражнение 2: Паттерн наблюдатель

### Задача
Создайте систему уведомлений для веб-приложения, используя паттерн наблюдатель для подписки на события и рассылки уведомлений.

### Требования
1. Возможность подписки на события
2. Возможность отписки от событий
3. Рассылка уведомлений всем подписчикам
4. Различные типы уведомлений

### Решение
```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  // Подписка на событие
  subscribe(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    
    this.events[event].push(callback);
    
    // Возвращаем функцию для отписки
    return () => {
      this.unsubscribe(event, callback);
    };
  }
  
  // Отписка от события
  unsubscribe(event, callback) {
    if (this.events[event]) {
      const index = this.events[event].indexOf(callback);
      if (index > -1) {
        this.events[event].splice(index, 1);
      }
    }
  }
  
  // Отправка события всем подписчикам
  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => {
        try {
          callback(data);
        } catch (error) {
          console.error('Ошибка в обработчике события:', error);
        }
      });
    }
  }
}

// Система уведомлений
class NotificationSystem extends EventEmitter {
  constructor() {
    super();
    this.notifications = [];
  }
  
  // Отправить уведомление
  sendNotification(type, message, data = {}) {
    const notification = {
      id: Date.now(),
      type,
      message,
      data,
      timestamp: new Date()
    };
    
    this.notifications.push(notification);
    
    // Отправляем событие всем подписчикам
    this.emit(type, notification);
    this.emit('notification', notification);
    
    return notification;
  }
  
  // Получить последние уведомления
  getRecentNotifications(count = 10) {
    return this.notifications.slice(-count);
  }
  
  // Очистить уведомления старше определенного времени
  clearOldNotifications(hours = 24) {
    const cutoffTime = Date.now() - (hours * 60 * 60 * 1000);
    this.notifications = this.notifications.filter(
      notification => notification.timestamp.getTime() > cutoffTime
    );
  }
}

// Подписчики уведомлений
class NotificationSubscriber {
  constructor(name) {
    this.name = name;
  }
  
  handleNotification(notification) {
    console.log(`[${this.name}] Получено уведомление: ${notification.message}`);
    console.log(`Тип: ${notification.type}, Время: ${notification.timestamp}`);
  }
  
  handleError(errorNotification) {
    console.error(`[${this.name}] Ошибка: ${errorNotification.message}`);
  }
  
  handleInfo(infoNotification) {
    console.info(`[${this.name}] Информация: ${infoNotification.message}`);
  }
}

// Пример использования
const notificationSystem = new NotificationSystem();

// Создаем подписчиков
const userSubscriber = new NotificationSubscriber('Пользователь');
const adminSubscriber = new NotificationSubscriber('Администратор');
const errorSubscriber = new NotificationSubscriber('Система ошибок');

// Подписываемся на уведомления
const unsubscribeUser = notificationSystem.subscribe('notification', 
  userSubscriber.handleNotification.bind(userSubscriber)
);

const unsubscribeAdmin = notificationSystem.subscribe('info', 
  adminSubscriber.handleInfo.bind(adminSubscriber)
);

const unsubscribeError = notificationSystem.subscribe('error', 
  errorSubscriber.handleError.bind(errorSubscriber)
);

// Отправляем уведомления
notificationSystem.sendNotification('info', 'Пользователь вошел в систему', {
  userId: 123,
  username: 'john_doe'
});

notificationSystem.sendNotification('error', 'Ошибка базы данных', {
  errorCode: 500,
  errorMessage: 'Connection timeout'
});

notificationSystem.sendNotification('warning', 'Низкий уровень заряда батареи', {
  level: 15
});

// Отписываемся от уведомлений
unsubscribeUser();

// Отправляем еще одно уведомление (пользователь уже отписан)
notificationSystem.sendNotification('info', 'Новое сообщение', {
  messageId: 456
});

// Показываем последние уведомления
console.log('Последние уведомления:', notificationSystem.getRecentNotifications());
```

## Упражнение 3: Фабричный паттерн

### Задача
Создайте систему создания различных типов пользователей (администратор, модератор, обычный пользователь) с использованием фабричного паттерна.

### Требования
1. Создание пользователей разных типов с общим интерфейсом
2. Уникальные свойства и методы для каждого типа пользователя
3. Централизованная фабрика для создания пользователей
4. Валидация входных данных

### Решение
```javascript
// Базовый класс пользователя
class User {
  constructor(id, name, email) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.createdAt = new Date();
    this.permissions = [];
  }
  
  // Общие методы
  getInfo() {
    return {
      id: this.id,
      name: this.name,
      email: this.email,
      type: this.constructor.name,
      createdAt: this.createdAt
    };
  }
  
  hasPermission(permission) {
    return this.permissions.includes(permission);
  }
  
  addPermission(permission) {
    if (!this.permissions.includes(permission)) {
      this.permissions.push(permission);
    }
  }
  
  removePermission(permission) {
    const index = this.permissions.indexOf(permission);
    if (index > -1) {
      this.permissions.splice(index, 1);
    }
  }
}

// Администратор
class Admin extends User {
  constructor(id, name, email) {
    super(id, name, email);
    this.permissions = ['read', 'write', 'delete', 'manage_users', 'manage_system'];
    this.role = 'admin';
  }
  
  manageUsers() {
    console.log(`${this.name} управляет пользователями`);
  }
  
  manageSystem() {
    console.log(`${this.name} управляет системой`);
  }
}

// Модератор
class Moderator extends User {
  constructor(id, name, email) {
    super(id, name, email);
    this.permissions = ['read', 'write', 'moderate_content'];
    this.role = 'moderator';
  }
  
  moderateContent() {
    console.log(`${this.name} модерирует контент`);
  }
  
  banUser(userId) {
    console.log(`${this.name} заблокировал пользователя ${userId}`);
  }
}

// Обычный пользователь
class RegularUser extends User {
  constructor(id, name, email) {
    super(id, name, email);
    this.permissions = ['read', 'write_own'];
    this.role = 'user';
  }
  
  createPost(content) {
    console.log(`${this.name} создал пост: ${content}`);
  }
  
  editOwnPost(postId, newContent) {
    console.log(`${this.name} отредактировал пост ${postId}`);
  }
}

// Фабрика пользователей
class UserFactory {
  static createUser(type, id, name, email) {
    // Валидация входных данных
    if (!id || !name || !email) {
      throw new Error('Необходимо указать id, имя и email');
    }
    
    if (!this.isValidEmail(email)) {
      throw new Error('Неверный формат email');
    }
    
    // Создание пользователя в зависимости от типа
    switch (type.toLowerCase()) {
      case 'admin':
        return new Admin(id, name, email);
      case 'moderator':
        return new Moderator(id, name, email);
      case 'user':
      case 'regular':
        return new RegularUser(id, name, email);
      default:
        throw new Error(`Неизвестный тип пользователя: ${type}`);
    }
  }
  
  static isValidEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  // Создание пользователей из конфигурационного объекта
  static createUsersFromConfig(config) {
    const users = [];
    const errors = [];
    
    config.forEach((userConfig, index) => {
      try {
        const user = this.createUser(
          userConfig.type,
          userConfig.id,
          userConfig.name,
          userConfig.email
        );
        users.push(user);
      } catch (error) {
        errors.push({
          index,
          config: userConfig,
          error: error.message
        });
      }
    });
    
    return { users, errors };
  }
}

// Пример использования
try {
  // Создание отдельных пользователей
  const admin = UserFactory.createUser('admin', 1, 'Alice Admin', 'alice@example.com');
  const moderator = UserFactory.createUser('moderator', 2, 'Bob Moderator', 'bob@example.com');
  const user = UserFactory.createUser('user', 3, 'Charlie User', 'charlie@example.com');
  
  console.log(admin.getInfo());
  console.log(moderator.getInfo());
  console.log(user.getInfo());
  
  // Администратор может управлять системой
  admin.manageSystem();
  admin.manageUsers();
  
  // Модератор может модерировать контент
  moderator.moderateContent();
  moderator.banUser(123);
  
  // Пользователь может создавать посты
  user.createPost('Мой первый пост');
  user.editOwnPost(1, 'Отредактированный пост');
  
  // Создание пользователей из конфигурации
  const userConfigs = [
    { type: 'admin', id: 4, name: 'David Admin', email: 'david@example.com' },
    { type: 'moderator', id: 5, name: 'Eve Moderator', email: 'eve@example.com' },
    { type: 'user', id: 6, name: 'Frank User', email: 'frank@example.com' },
    { type: 'user', id: 7, name: 'Grace User', email: 'invalid-email' }, // Неверный email
    { type: 'unknown', id: 8, name: 'Henry Unknown', email: 'henry@example.com' } // Неизвестный тип
  ];
  
  const { users, errors } = UserFactory.createUsersFromConfig(userConfigs);
  
  console.log('Созданные пользователи:', users.map(u => u.getInfo()));
  console.log('Ошибки создания:', errors);
  
} catch (error) {
  console.error('Ошибка создания пользователя:', error.message);
}
```

## Упражнение 4: Синглтон

### Задача
Создайте систему логирования для приложения, используя паттерн синглтон для обеспечения единственного экземпляра логгера.

### Требования
1. Единственный экземпляр логгера в приложении
2. Различные уровни логирования (debug, info, warn, error)
3. Возможность настройки формата логов
4. Сохранение логов в памяти и вывод в консоль

### Решение
```javascript
class Logger {
  constructor() {
    // Проверяем, существует ли уже экземпляр
    if (Logger.instance) {
      return Logger.instance;
    }
    
    // Инициализация свойств
    this.logs = [];
    this.level = 'info'; // По умолчанию уровень info
    this.format = 'simple'; // Формат по умолчанию
    this.maxLogs = 1000; // Максимальное количество хранимых логов
    
    // Сохраняем ссылку на экземпляр
    Logger.instance = this;
    
    return this;
  }
  
  // Установка уровня логирования
  setLevel(level) {
    const validLevels = ['debug', 'info', 'warn', 'error'];
    if (validLevels.includes(level)) {
      this.level = level;
    } else {
      throw new Error(`Неверный уровень логирования: ${level}`);
    }
  }
  
  // Установка формата логов
  setFormat(format) {
    const validFormats = ['simple', 'detailed', 'json'];
    if (validFormats.includes(format)) {
      this.format = format;
    } else {
      throw new Error(`Неверный формат логов: ${format}`);
    }
  }
  
  // Получение числового значения уровня для сравнения
  getLevelValue(level) {
    const levels = {
      'debug': 0,
      'info': 1,
      'warn': 2,
      'error': 3
    };
    return levels[level] || 1;
  }
  
  // Форматирование сообщения
  formatMessage(level, message, data) {
    const timestamp = new Date().toISOString();
    
    switch (this.format) {
      case 'detailed':
        return `[${timestamp}] ${level.toUpperCase()}: ${message} ${data ? JSON.stringify(data) : ''}`;
      case 'json':
        return JSON.stringify({
          timestamp,
          level,
          message,
          data
        });
      case 'simple':
      default:
        return `${level.toUpperCase()}: ${message}`;
    }
  }
  
  // Добавление лога
  addLog(level, message, data = null) {
    // Проверяем уровень логирования
    if (this.getLevelValue(level) < this.getLevelValue(this.level)) {
      return;
    }
    
    const logEntry = {
      timestamp: new Date(),
      level,
      message,
      data
    };
    
    // Добавляем лог в хранилище
    this.logs.push(logEntry);
    
    // Ограничиваем количество хранимых логов
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }
    
    // Выводим в консоль
    const formattedMessage = this.formatMessage(level, message, data);
    switch (level) {
      case 'error':
        console.error(formattedMessage);
        break;
      case 'warn':
        console.warn(formattedMessage);
        break;
      case 'info':
        console.info(formattedMessage);
        break;
      case 'debug':
        console.debug(formattedMessage);
        break;
      default:
        console.log(formattedMessage);
    }
  }
  
  // Методы для разных уровней логирования
  debug(message, data) {
    this.addLog('debug', message, data);
  }
  
  info(message, data) {
    this.addLog('info', message, data);
  }
  
  warn(message, data) {
    this.addLog('warn', message, data);
  }
  
  error(message, data) {
    this.addLog('error', message, data);
  }
  
  // Получение логов
  getLogs(level = null) {
    if (level) {
      return this.logs.filter(log => log.level === level);
    }
    return [...this.logs]; // Возвращаем копию массива
  }
  
  // Получение последних логов
  getLastLogs(count = 10) {
    return this.logs.slice(-count);
  }
  
  // Очистка логов
  clearLogs() {
    this.logs = [];
  }
  
  // Получение статистики по логам
  getStats() {
    const stats = {
      total: this.logs.length,
      byLevel: {}
    };
    
    this.logs.forEach(log => {
      if (!stats.byLevel[log.level]) {
        stats.byLevel[log.level] = 0;
      }
      stats.byLevel[log.level]++;
    });
    
    return stats;
  }
  
  // Экспорт логов в файл (имитация)
  exportLogs() {
    return this.logs.map(log => ({
      timestamp: log.timestamp.toISOString(),
      level: log.level,
      message: log.message,
      data: log.data
    }));
  }
}

// Пример использования
const logger1 = new Logger();
const logger2 = new Logger();

// Проверяем, что это один и тот же экземпляр
console.log(logger1 === logger2); // true

// Используем логгер
logger1.info('Приложение запущено');
logger1.debug('Отладочная информация', { userId: 123 });
logger1.warn('Предупреждение о низком уровне заряда');
logger1.error('Ошибка подключения к базе данных', { 
  errorCode: 500, 
  errorMessage: 'Connection timeout' 
});

// Меняем уровень логирования
logger1.setLevel('debug');
logger1.debug('Теперь это сообщение будет отображено');

// Меняем формат логов
logger1.setFormat('detailed');
logger1.info('Сообщение с детальным форматом');

// Получаем статистику
console.log('Статистика логов:', logger1.getStats());

// Получаем последние логи
console.log('Последние логи:', logger1.getLastLogs(3));
```

## Упражнение 5: Декоратор

### Задача
Создайте систему декораторов для добавления дополнительной функциональности к функциям (логирование, кэширование, ограничение частоты вызовов).

### Требования
1. Декоратор для логирования вызовов функций
2. Декоратор для кэширования результатов
3. Декоратор для ограничения частоты вызовов
4. Возможность комбинирования декораторов

### Решение
```javascript
// Базовый декоратор
class Decorator {
  constructor(fn) {
    this.fn = fn;
  }
  
  execute(...args) {
    return this.fn.apply(this, args);
  }
}

// Декоратор логирования
class LoggingDecorator extends Decorator {
  constructor(fn, logger = console) {
    super(fn);
    this.logger = logger;
    this.fnName = fn.name || 'anonymous';
  }
  
  execute(...args) {
    const startTime = Date.now();
    this.logger.log(`[${this.fnName}] Вызов функции с аргументами:`, args);
    
    try {
      const result = super.execute(...args);
      const endTime = Date.now();
      this.logger.log(`[${this.fnName}] Функция выполнена за ${endTime - startTime}мс, результат:`, result);
      return result;
    } catch (error) {
      const endTime = Date.now();
      this.logger.error(`[${this.fnName}] Ошибка при выполнении функции за ${endTime - startTime}мс:`, error.message);
      throw error;
    }
  }
}

// Декоратор кэширования
class CachingDecorator extends Decorator {
  constructor(fn, ttl = 60000) { // ttl в миллисекундах (по умолчанию 1 минута)
    super(fn);
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  execute(...args) {
    const key = this.generateKey(args);
    
    // Проверяем кэш
    if (this.cache.has(key)) {
      const cached = this.cache.get(key);
      const now = Date.now();
      
      // Проверяем время жизни кэша
      if (now - cached.timestamp < this.ttl) {
        console.log(`[Кэш] Возвращено закэшированное значение для ключа: ${key}`);
        return cached.value;
      } else {
        // Удаляем устаревший кэш
        this.cache.delete(key);
      }
    }
    
    // Выполняем функцию и кэшируем результат
    const result = super.execute(...args);
    this.cache.set(key, {
      value: result,
      timestamp: Date.now()
    });
    
    console.log(`[Кэш] Значение закэшировано для ключа: ${key}`);
    return result;
  }
  
  generateKey(args) {
    return args.map(arg => {
      if (typeof arg === 'object' && arg !== null) {
        return JSON.stringify(arg);
      }
      return String(arg);
    }).join('|');
  }
  
  clearCache() {
    this.cache.clear();
  }
  
  getCacheSize() {
    return this.cache.size;
  }
}

// Декоратор ограничения частоты вызовов (throttling)
class ThrottlingDecorator extends Decorator {
  constructor(fn, delay = 1000) {
    super(fn);
    this.delay = delay;
    this.lastCall = 0;
    this.lastResult = null;
  }
  
  execute(...args) {
    const now = Date.now();
    
    if (now - this.lastCall >= this.delay) {
      this.lastResult = super.execute(...args);
      this.lastCall = now;
    } else {
      console.log(`[Throttle] Вызов функции ограничен, возвращено предыдущее значение`);
    }
    
    return this.lastResult;
  }
}

// Декоратор повторных попыток
class RetryDecorator extends Decorator {
  constructor(fn, maxRetries = 3, delay = 1000) {
    super(fn);
    this.maxRetries = maxRetries;
    this.delay = delay;
  }
  
  async execute(...args) {
    let lastError;
    
    for (let i = 0; i <= this.maxRetries; i++) {
      try {
        const result = await super.execute(...args);
        if (i > 0) {
          console.log(`[Retry] Функция успешно выполнена после ${i} попыток`);
        }
        return result;
      } catch (error) {
        lastError = error;
        if (i < this.maxRetries) {
          console.log(`[Retry] Попытка ${i + 1} не удалась, повтор через ${this.delay}мс`);
          await this.wait(this.delay);
        }
      }
    }
    
    throw new Error(`[Retry] Функция не выполнена после ${this.maxRetries + 1} попыток. Последняя ошибка: ${lastError.message}`);
  }
  
  wait(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Фабрика декораторов
class DecoratorFactory {
  static log(fn, logger) {
    return new LoggingDecorator(fn, logger);
  }
  
  static cache(fn, ttl) {
    return new CachingDecorator(fn, ttl);
  }
  
  static throttle(fn, delay) {
    return new ThrottlingDecorator(fn, delay);
  }
  
  static retry(fn, maxRetries, delay) {
    return new RetryDecorator(fn, maxRetries, delay);
  }
  
  // Комбинирование декораторов
  static combine(fn, decorators) {
    let decoratedFn = fn;
    
    // Применяем декораторы в обратном порядке (снизу вверх)
    for (let i = decorators.length - 1; i >= 0; i--) {
      const decorator = decorators[i];
      decoratedFn = decorator(decoratedFn);
    }
    
    return decoratedFn;
  }
}

// Примеры функций для декорирования
function expensiveCalculation(x, y) {
  // Имитация тяжелых вычислений
  console.log(`Выполнение тяжелых вычислений для ${x} и ${y}...`);
  // Имитация задержки
  const start = Date.now();
  while (Date.now() - start < 100) {} // 100мс задержка
  return x * y + (x + y);
}

async function fetchUserData(userId) {
  console.log(`Запрос данных пользователя ${userId}...`);
  
  // Имитация случайных ошибок
  if (Math.random() < 0.3) {
    throw new Error(`Ошибка загрузки данных пользователя ${userId}`);
  }
  
  // Имитация задержки
  await new Promise(resolve => setTimeout(resolve, 500));
  
  return {
    id: userId,
    name: `User ${userId}`,
    email: `user${userId}@example.com`
  };
}

function apiCall(endpoint, data) {
  console.log(`API вызов к ${endpoint} с данными:`, data);
  
  // Имитация случайных ошибок
  if (Math.random() < 0.2) {
    throw new Error(`Ошибка API: ${endpoint}`);
  }
  
  return { 
    success: true, 
    data: `Результат для ${endpoint}`,
    timestamp: new Date().toISOString()
  };
}

// Пример использования декораторов
console.log('=== Декоратор логирования ===');
const loggedCalculation = DecoratorFactory.log(expensiveCalculation);
console.log(loggedCalculation.execute(5, 3));
console.log(loggedCalculation.execute(10, 2));

console.log('\n=== Декоратор кэширования ===');
const cachedCalculation = DecoratorFactory.cache(expensiveCalculation, 5000); // 5 секунд TTL
console.log(cachedCalculation.execute(5, 3)); // Первый вызов
console.log(cachedCalculation.execute(5, 3)); // Из кэша
console.log(cachedCalculation.execute(10, 2)); // Новый вызов

console.log('\n=== Декоратор ограничения частоты ===');
const throttledApiCall = DecoratorFactory.throttle(apiCall, 2000); // 2 секунды
console.log(throttledApiCall.execute('/users', { page: 1 }));
console.log(throttledApiCall.execute('/users', { page: 2 })); // Ограничен
setTimeout(() => {
  console.log(throttledApiCall.execute('/users', { page: 3 })); // После задержки
}, 2100);

console.log('\n=== Декоратор повторных попыток ===');
const retryFetchUser = DecoratorFactory.retry(fetchUserData, 3, 500);
retryFetchUser.execute(123).then(result => {
  console.log('Пользователь загружен:', result);
}).catch(error => {
  console.error('Ошибка после всех попыток:', error.message);
});

console.log('\n=== Комбинирование декораторов ===');
const combinedFunction = DecoratorFactory.combine(
  expensiveCalculation,
  [
    fn => DecoratorFactory.log(fn),
    fn => DecoratorFactory.cache(fn, 10000),
    fn => DecoratorFactory.throttle(fn, 1000)
  ]
);

console.log(combinedFunction.execute(7, 8));
console.log(combinedFunction.execute(7, 8)); // Из кэша
console.log(combinedFunction.execute(9, 4));
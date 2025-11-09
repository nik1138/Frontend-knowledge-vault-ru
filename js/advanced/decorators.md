# Декораторы в JavaScript

## Что такое декораторы

Декораторы - это специальный синтаксис в JavaScript, который позволяет добавлять метаданные и модифицировать классы, методы, свойства и параметры функций. Это мощный инструмент метапрограммирования, который упрощает создание повторно используемой логики и паттернов проектирования.

## Статус декораторов

Декораторы находятся на стадии предложения (Stage 3) в процессе стандартизации ECMAScript. Это означает, что спецификация почти завершена, но может измениться до финального утверждения. Для использования декораторов сегодня требуется транспилятор, такой как Babel.

## Основы декораторов

### Декораторы классов

```javascript
// Базовый декоратор класса
function logClass(target) {
  console.log(`Создан класс: ${target.name}`);
  return target;
}

@logClass
class MyClass {
  constructor() {
    console.log('Конструктор MyClass вызван');
  }
}

const instance = new MyClass();
// Вывод:
// Создан класс: MyClass
// Конструктор MyClass вызван
```

### Декораторы методов

```javascript
// Декоратор для логирования вызовов методов
function logMethod(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args) {
    console.log(`Вызов метода ${propertyKey} с аргументами:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Метод ${propertyKey} вернул:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @logMethod
  add(a, b) {
    return a + b;
  }
  
  @logMethod
  multiply(a, b) {
    return a * b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// Вывод:
// Вызов метода add с аргументами: [2, 3]
// Метод add вернул: 5

calc.multiply(4, 5);
// Вывод:
// Вызов метода multiply с аргументами: [4, 5]
// Метод multiply вернул: 20
```

### Декораторы свойств

```javascript
// Декоратор для валидации свойств
function validateString(target, propertyKey) {
  let value = target[propertyKey];
  
  const getter = function() {
    return value;
  };
  
  const setter = function(newVal) {
    if (typeof newVal !== 'string') {
      throw new TypeError(`Свойство ${propertyKey} должно быть строкой`);
    }
    if (newVal.length === 0) {
      throw new Error(`Свойство ${propertyKey} не может быть пустой строкой`);
    }
    value = newVal;
  };
  
  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter,
    enumerable: true,
    configurable: true
  });
}

class User {
  @validateString
  name;
  
  @validateString
  email;
  
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

const user = new User("Иван", "ivan@example.com");
console.log(user.name); // "Иван"

// user.name = 123; // TypeError: Свойство name должно быть строкой
// user.email = ""; // Error: Свойство email не может быть пустой строкой
```

## Продвинутые декораторы

### Фабрика декораторов

```javascript
// Фабрика декораторов для установки значений по умолчанию
function defaultValue(defaultValue) {
  return function(target, propertyKey) {
    let value = target[propertyKey];
    
    const getter = function() {
      return value !== undefined ? value : defaultValue;
    };
    
    const setter = function(newVal) {
      value = newVal;
    };
    
    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

// Фабрика декораторов для ограничения диапазона значений
function range(min, max) {
  return function(target, propertyKey) {
    let value = target[propertyKey];
    
    const getter = function() {
      return value;
    };
    
    const setter = function(newVal) {
      if (typeof newVal !== 'number') {
        throw new TypeError(`Свойство ${propertyKey} должно быть числом`);
      }
      if (newVal < min || newVal > max) {
        throw new RangeError(`Свойство ${propertyKey} должно быть в диапазоне от ${min} до ${max}`);
      }
      value = newVal;
    };
    
    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

class Product {
  @defaultValue("Без названия")
  name;
  
  @range(0, 1000000)
  price;
  
  @defaultValue(0)
  @range(0, 100)
  discount;
  
  constructor(name, price, discount) {
    this.name = name;
    this.price = price;
    this.discount = discount;
  }
}

const product = new Product();
console.log(product.name); // "Без названия"
console.log(product.discount); // 0

product.price = 1000;
product.discount = 15;

console.log(product.price); // 1000
console.log(product.discount); // 15

// product.price = -100; // RangeError: Свойство price должно быть в диапазоне от 0 до 1000000
```

### Декораторы с параметрами для методов

```javascript
// Декоратор для ограничения частоты вызовов метода
function throttle(delay) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;
    let lastCall = 0;
    
    descriptor.value = function(...args) {
      const now = Date.now();
      if (now - lastCall >= delay) {
        lastCall = now;
        return originalMethod.apply(this, args);
      } else {
        console.log(`Метод ${propertyKey} вызывается слишком часто, пропущен вызов`);
      }
    };
    
    return descriptor;
  };
}

// Декоратор для кэширования результатов метода
function memoize(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  const cache = new Map();
  
  descriptor.value = function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log(`Возвращено значение из кэша для метода ${propertyKey}`);
      return cache.get(key);
    }
    
    console.log(`Вычисление значения для метода ${propertyKey}`);
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}

class ExpensiveOperations {
  @throttle(1000) // Ограничение 1 вызов в секунду
  logMessage(message) {
    console.log(`Сообщение: ${message}`);
  }
  
  @memoize
  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
}

const operations = new ExpensiveOperations();

// Тест throttle
operations.logMessage("Первое сообщение");
operations.logMessage("Второе сообщение"); // Будет пропущено
setTimeout(() => {
  operations.logMessage("Третье сообщение"); // Будет выполнено
}, 1100);

// Тест memoize
console.log(operations.fibonacci(10)); // Вычисление
console.log(operations.fibonacci(10)); // Из кэша
```

## Практические примеры

### Система валидации

```javascript
// Декораторы для валидации данных
const validators = new Map();

function required(target, propertyKey) {
  if (!validators.has(target.constructor)) {
    validators.set(target.constructor, new Map());
  }
  
  const classValidators = validators.get(target.constructor);
  if (!classValidators.has(propertyKey)) {
    classValidators.set(propertyKey, []);
  }
  
  classValidators.get(propertyKey).push({
    validate: (value) => value !== undefined && value !== null && value !== '',
    message: `Свойство ${propertyKey} обязательно`
  });
}

function minLength(min) {
  return function(target, propertyKey) {
    if (!validators.has(target.constructor)) {
      validators.set(target.constructor, new Map());
    }
    
    const classValidators = validators.get(target.constructor);
    if (!classValidators.has(propertyKey)) {
      classValidators.set(propertyKey, []);
    }
    
    classValidators.get(propertyKey).push({
      validate: (value) => !value || value.length >= min,
      message: `Свойство ${propertyKey} должно содержать минимум ${min} символов`
    });
  };
}

function email(target, propertyKey) {
  if (!validators.has(target.constructor)) {
    validators.set(target.constructor, new Map());
  }
  
  const classValidators = validators.get(target.constructor);
  if (!classValidators.has(propertyKey)) {
    classValidators.set(propertyKey, []);
  }
  
  classValidators.get(propertyKey).push({
    validate: (value) => !value || /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    message: `Свойство ${propertyKey} должно быть корректным email адресом`
  });
}

function validate(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args) {
    const instance = args[0];
    const classValidators = validators.get(instance.constructor);
    
    if (classValidators) {
      const errors = [];
      
      for (const [property, propertyValidators] of classValidators) {
        const value = instance[property];
        for (const validator of propertyValidators) {
          if (!validator.validate(value)) {
            errors.push(validator.message);
          }
        }
      }
      
      if (errors.length > 0) {
        throw new Error(`Валидация не пройдена:\n${errors.join('\n')}`);
      }
    }
    
    return originalMethod.apply(this, args);
  };
  
  return descriptor;
}

// Использование
class User {
  @required
  @minLength(2)
  name;
  
  @required
  @email
  email;
  
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  @validate
  save() {
    console.log('Пользователь сохранен:', this);
  }
}

try {
  const user = new User("И", "invalid-email");
  user.save(); // Ошибка валидации
} catch (error) {
  console.error(error.message);
}

try {
  const user2 = new User("Иван", "ivan@example.com");
  user2.save(); // Успешное сохранение
} catch (error) {
  console.error(error.message);
}
```

### Система событий

```javascript
// Декораторы для создания системы событий
const eventHandlers = new Map();

function on(event) {
  return function(target, propertyKey, descriptor) {
    if (!eventHandlers.has(target.constructor)) {
      eventHandlers.set(target.constructor, new Map());
    }
    
    const classEvents = eventHandlers.get(target.constructor);
    if (!classEvents.has(event)) {
      classEvents.set(event, []);
    }
    
    classEvents.get(event).push({
      method: propertyKey,
      descriptor: descriptor
    });
    
    return descriptor;
  };
}

function emit(event) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args) {
      const result = originalMethod.apply(this, args);
      
      // Вызываем обработчики событий
      const classEvents = eventHandlers.get(this.constructor);
      if (classEvents && classEvents.has(event)) {
        const handlers = classEvents.get(event);
        for (const handler of handlers) {
          this[handler.method](...args);
        }
      }
      
      return result;
    };
    
    return descriptor;
  };
}

class EventEmitter {
  emit(event, ...args) {
    const classEvents = eventHandlers.get(this.constructor);
    if (classEvents && classEvents.has(event)) {
      const handlers = classEvents.get(event);
      for (const handler of handlers) {
        this[handler.method](...args);
      }
    }
  }
}

class UserService extends EventEmitter {
  users = [];
  
  @on('userCreated')
  sendWelcomeEmail(user) {
    console.log(`Отправка приветственного email пользователю ${user.name}`);
  }
  
  @on('userCreated')
  logUserCreation(user) {
    console.log(`Пользователь ${user.name} создан в ${new Date().toISOString()}`);
  }
  
  @emit('userCreated')
  createUser(name, email) {
    const user = { id: Date.now(), name, email };
    this.users.push(user);
    return user;
  }
}

const userService = new UserService();
const user = userService.createUser("Иван", "ivan@example.com");
// Вывод:
// Отправка приветственного email пользователю Иван
// Пользователь Иван создан в 2023-...
```

### Система кэширования

```javascript
// Декораторы для кэширования
const cacheStorage = new Map();

function cache(ttl = 60000) { // ttl в миллисекундах (по умолчанию 1 минута)
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args) {
      const cacheKey = `${propertyKey}_${JSON.stringify(args)}`;
      
      // Проверяем наличие кэша
      if (cacheStorage.has(cacheKey)) {
        const cached = cacheStorage.get(cacheKey);
        if (Date.now() - cached.timestamp < ttl) {
          console.log(`Возвращено значение из кэша для ${propertyKey}`);
          return cached.value;
        } else {
          // Удаляем устаревший кэш
          cacheStorage.delete(cacheKey);
        }
      }
      
      // Выполняем оригинальный метод
      console.log(`Вычисление значения для ${propertyKey}`);
      const result = originalMethod.apply(this, args);
      
      // Сохраняем в кэш
      cacheStorage.set(cacheKey, {
        value: result,
        timestamp: Date.now()
      });
      
      return result;
    };
    
    return descriptor;
  };
}

function clearCache(prefix = '') {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(...args) {
      const result = originalMethod.apply(this, args);
      
      // Очищаем кэш
      for (const key of cacheStorage.keys()) {
        if (key.startsWith(prefix)) {
          cacheStorage.delete(key);
        }
      }
      
      console.log(`Кэш очищен для префикса: ${prefix}`);
      return result;
    };
    
    return descriptor;
  };
}

class DataService {
  @cache(30000) // Кэш на 30 секунд
  async fetchUserData(userId) {
    // Имитация асинхронного запроса
    await new Promise(resolve => setTimeout(resolve, 1000));
    return { id: userId, name: `Пользователь ${userId}`, timestamp: Date.now() };
  }
  
  @cache(60000) // Кэш на 1 минуту
  async fetchUserPosts(userId) {
    // Имитация асинхронного запроса
    await new Promise(resolve => setTimeout(resolve, 1500));
    return [
      { id: 1, title: `Пост 1 пользователя ${userId}`, userId },
      { id: 2, title: `Пост 2 пользователя ${userId}`, userId }
    ];
  }
  
  @clearCache('fetchUser')
  async updateUser(userId, data) {
    // Имитация обновления данных
    await new Promise(resolve => setTimeout(resolve, 500));
    console.log(`Пользователь ${userId} обновлен`);
    return { success: true };
  }
}

// Использование
async function demonstrateCaching() {
  const service = new DataService();
  
  console.log('Первый вызов:');
  const user1 = await service.fetchUserData(1);
  console.log(user1);
  
  console.log('Повторный вызов (из кэша):');
  const user2 = await service.fetchUserData(1);
  console.log(user2);
  
  console.log('Обновление пользователя (очистка кэша):');
  await service.updateUser(1, { name: "Новое имя" });
  
  console.log('Вызов после очистки кэша:');
  const user3 = await service.fetchUserData(1);
  console.log(user3);
}

demonstrateCaching();
```

## Современные возможности

### Декораторы в TypeScript

```typescript
// Пример использования декораторов в TypeScript
function autobind(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  return {
    configurable: true,
    get() {
      const boundFn = originalMethod.bind(this);
      Object.defineProperty(this, propertyKey, {
        value: boundFn,
        configurable: true,
        writable: true
      });
      return boundFn;
    }
  };
}

class Button {
  label: string;
  
  constructor(label: string) {
    this.label = label;
  }
  
  @autobind
  handleClick() {
    console.log(`Кнопка "${this.label}" нажата`);
  }
}

const button = new Button("Отправить");
const handler = button.handleClick;
handler(); // "Кнопка "Отправить" нажата" (this корректно привязан)
```

### Декораторы параметров

```javascript
// Декораторы параметров (экспериментальная функция)
const parameterMetadata = new Map();

function logParameter(target, propertyKey, parameterIndex) {
  const existingParameters = parameterMetadata.get(propertyKey) || [];
  existingParameters.push(parameterIndex);
  parameterMetadata.set(propertyKey, existingParameters);
}

function logMethodWithParameters(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  const parameterIndexes = parameterMetadata.get(propertyKey) || [];
  
  descriptor.value = function(...args) {
    parameterIndexes.forEach(index => {
      console.log(`Параметр ${index}:`, args[index]);
    });
    
    return originalMethod.apply(this, args);
  };
  
  return descriptor;
}

class Example {
  @logMethodWithParameters
  greet(@logParameter name, @logParameter greeting = "Привет") {
    return `${greeting}, ${name}!`;
  }
}

const example = new Example();
example.greet("Иван", "Здравствуйте");
// Вывод:
// Параметр 0: Иван
// Параметр 1: Здравствуйте
// Здравствуйте, Иван!
```

## Лучшие практики

### 1. Используйте декораторы для сквозной функциональности

```javascript
// Хорошо: декораторы для логирования, кэширования, валидации
class ApiClient {
  @logMethod
  @cache(60000)
  async fetchUser(id) {
    // реализация
  }
  
  @validate
  @emit('userUpdated')
  updateUser(id, data) {
    // реализация
  }
}
```

### 2. Делайте декораторы предсказуемыми

```javascript
// Хорошо: декораторы с четким поведением
function readonly(target, propertyKey, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

// Плохо: декораторы с неожиданным поведением
function confusingDecorator(target, propertyKey, descriptor) {
  // Меняет поведение в зависимости от внешних факторов
  if (Math.random() > 0.5) {
    descriptor.value = function() { return "A"; };
  } else {
    descriptor.value = function() { return "B"; };
  }
  return descriptor;
}
```

### 3. Обрабатывайте ошибки в декораторах

```javascript
function safeDecorator(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args) {
    try {
      return originalMethod.apply(this, args);
    } catch (error) {
      console.error(`Ошибка в методе ${propertyKey}:`, error);
      // Можно реализовать логику восстановления
      return null;
    }
  };
  
  return descriptor;
}
```

## Настройка Babel для декораторов

### Конфигурация Babel

```json
// .babelrc
{
  "presets": ["@babel/preset-env"],
  "plugins": [
    ["@babel/plugin-proposal-decorators", { "legacy": true }],
    "@babel/plugin-proposal-class-properties"
  ]
}
```

### Использование с Webpack

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env'],
            plugins: [
              ['@babel/plugin-proposal-decorators', { 'legacy': true }],
              '@babel/plugin-proposal-class-properties'
            ]
          }
        }
      }
    ]
  }
};
```

## Заключение

Декораторы - это мощный инструмент метапрограммирования в JavaScript, который позволяет добавлять функциональность к классам, методам и свойствам декларативным способом. Хотя они еще не являются частью официального стандарта, они уже активно используются в современных фреймворках и библиотеках.

Правильное использование декораторов позволяет создавать чистый, повторно используемый код для сквозной функциональности, такой как логирование, кэширование, валидация и управление доступом. Однако их следует использовать с осторожностью, так как чрезмерное использование может сделать код сложным для понимания.

Понимание декораторов критически важно для разработчиков, работающих с современными фреймворками, такими как Angular, и для тех, кто создает библиотеки и инструменты для разработки. По мере продвижения спецификации к финальному утверждению, декораторы станут стандартной частью языка JavaScript.

#javascript #programming #decorators #metaprogramming #typescript #web-development
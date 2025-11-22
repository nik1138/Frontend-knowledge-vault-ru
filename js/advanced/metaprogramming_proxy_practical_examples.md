---
tags: [javascript, frontend, advanced, metaprogramming, proxy]
aliases: [метапрограммирование в js для фронтенда, прокси в javascript, практические примеры метапрограммирования]
---

# Метапрограммирование и прокси в JavaScript - Практические примеры для Frontend разработки

## Введение

Метапрограммирование в JavaScript позволяет программам анализировать и изменять собственную структуру и поведение во время выполнения. Прокси - это мощный инструмент метапрограммирования, который позволяет перехватывать и переопределять основные операции с объектами. В этой статье рассмотрим практические примеры использования прокси с акцентом на задачи фронтенд разработки.

## Что такое Proxy?

Proxy (прокси) - это объект, который позволяет перехватывать и переопределять операции с другим объектом (целью). Прокси может "перехватывать" такие операции, как получение свойства, присваивание, перечисление, функции вызова и другие.

## Практический пример: Отслеживание изменений состояния

```javascript
// Создание реактивного объекта состояния с помощью Proxy
function createReactiveState(initialState, onChangeCallback) {
  const handler = {
    get(target, property, receiver) {
      // Перехватываем получение свойства
      const value = Reflect.get(target, property, receiver);
      
      // Если значение - объект, оборачиваем его в прокси рекурсивно
      if (value && typeof value === 'object' && !Array.isArray(value)) {
        return createReactiveState(value, onChangeCallback);
      }
      
      return value;
    },
    
    set(target, property, value, receiver) {
      const oldValue = target[property];
      
      // Устанавливаем новое значение
      const result = Reflect.set(target, property, value, receiver);
      
      // Вызываем callback при изменении
      if (oldValue !== value) {
        onChangeCallback(property, value, oldValue, target);
      }
      
      return result;
    }
  };
  
  return new Proxy(initialState, handler);
}

// Использование реактивного состояния
const state = createReactiveState({
  user: {
    name: 'John',
    email: 'john@example.com'
  },
  theme: 'light',
  notifications: true
}, (property, newValue, oldValue, target) => {
  console.log(`Свойство ${property} изменилось с ${oldValue} на ${newValue}`);
  
  // Обновление UI при изменении состояния
  updateUIBasedOnState(property, newValue, target);
});

// При изменении свойств будет вызван callback
state.user.name = 'Jane'; // Выведет: "Свойство name изменилось с John на Jane"
state.theme = 'dark';     // Выведет: "Свойство theme изменилось с light на dark"

function updateUIBasedOnState(property, newValue, state) {
  // Обновление UI в зависимости от измененного свойства
  switch(property) {
    case 'theme':
      document.body.className = newValue;
      break;
    case 'notifications':
      const notificationToggle = document.getElementById('notification-toggle');
      if (notificationToggle) {
        notificationToggle.checked = newValue;
      }
      break;
  }
}
```

## Практический пример: Валидация данных с помощью Proxy

```javascript
// Создание валидируемого объекта
function createValidatedObject(schema, initialData = {}) {
  const handler = {
    set(target, property, value) {
      // Проверяем, существует ли схема для этого свойства
      if (schema[property]) {
        const validation = schema[property];
        
        // Проверка типа
        if (validation.type && typeof value !== validation.type) {
          throw new Error(`Свойство ${property} должно быть типа ${validation.type}`);
        }
        
        // Проверка на валидатор
        if (validation.validator && !validation.validator(value)) {
          throw new Error(`Свойство ${property} не прошло валидацию`);
        }
        
        // Проверка на обязательность
        if (validation.required && (value === undefined || value === null)) {
          throw new Error(`Свойство ${property} обязательно для заполнения`);
        }
      }
      
      return Reflect.set(target, property, value);
    },
    
    get(target, property) {
      return Reflect.get(target, property);
    }
  };
  
  return new Proxy(initialData, handler);
}

// Использование валидируемого объекта
const userSchema = {
  name: {
    type: 'string',
    required: true,
    validator: (value) => value.length >= 2
  },
  email: {
    type: 'string',
    required: true,
    validator: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
  },
  age: {
    type: 'number',
    validator: (value) => value >= 0 && value <= 120
  }
};

const userData = createValidatedObject(userSchema, {
  name: 'John',
  email: 'john@example.com'
});

// Попытка установить невалидные данные вызовет ошибку
try {
  userData.email = 'invalid-email'; // Вызовет ошибку
} catch (error) {
  console.error('Ошибка валидации:', error.message);
}

// Попытка установить число в поле email вызовет ошибку
try {
  userData.age = 'not a number'; // Вызовет ошибку
} catch (error) {
  console.error('Ошибка валидации:', error.message);
}
```

## Практический пример: Логирование операций с Proxy

```javascript
// Создание логируемого объекта
function createLoggedObject(target, logger = console.log) {
  const handler = {
    get(obj, prop) {
      logger(`GET: ${prop}`);
      return Reflect.get(obj, prop);
    },
    
    set(obj, prop, value) {
      logger(`SET: ${prop} = ${value}`);
      return Reflect.set(obj, prop, value);
    },
    
    has(obj, prop) {
      logger(`HAS: ${prop}`);
      return Reflect.has(obj, prop);
    },
    
    deleteProperty(obj, prop) {
      logger(`DELETE: ${prop}`);
      return Reflect.deleteProperty(obj, prop);
    },
    
    apply(target, thisArg, argumentsList) {
      logger(`CALL: Function with args: ${JSON.stringify(argumentsList)}`);
      return Reflect.apply(target, thisArg, argumentsList);
    }
  };
  
  return new Proxy(target, handler);
}

// Использование логируемого объекта для отладки
const apiClient = createLoggedObject({
  getUsers: () => [{ id: 1, name: 'John' }],
  getUser: (id) => ({ id, name: 'John' }),
  data: { count: 0 }
});

apiClient.data.count = 5; // Лог: "SET: count = 5"
const users = apiClient.getUsers(); // Лог: "CALL: Function with args: []"
const hasData = 'data' in apiClient; // Лог: "HAS: data"
```

## Практический пример: Безопасный доступ к DOM элементам

```javascript
// Создание безопасного DOM прокси
function createSafeDOMProxy() {
  const handler = {
    get(target, property) {
      try {
        if (property.startsWith('#')) {
          // Поиск по ID
          return document.getElementById(property.slice(1));
        } else if (property.startsWith('.')) {
          // Поиск по классу
          return document.querySelector(property);
        } else if (property.startsWith('[')) {
          // Поиск по атрибуту
          return document.querySelector(property);
        } else {
          // Обычный доступ к свойствам
          return Reflect.get(target, property);
        }
      } catch (error) {
        console.warn(`Ошибка доступа к DOM элементу: ${property}`, error);
        return null;
      }
    }
  };
  
  return new Proxy({}, handler);
}

// Использование безопасного DOM прокси
const $ = createSafeDOMProxy();

// Безопасный доступ к элементам
const header = $.header; // Попытается получить document.querySelector('header')
const mainButton = $('#main-button'); // Попытается получить document.getElementById('main-button')
const activeLink = $('.active-link'); // Попытается получить document.querySelector('.active-link')

// Использование в функциях
function initializeComponents() {
  const loginButton = $('#login-btn');
  const logoutButton = $('#logout-btn');
  const profileSection = $('.profile-section');
  
  if (loginButton) {
    loginButton.addEventListener('click', handleLogin);
  }
  
  if (logoutButton) {
    logoutButton.addEventListener('click', handleLogout);
  }
  
  if (profileSection) {
    profileSection.style.display = 'block';
  }
}
```

## Практический пример: Кеширование результатов вычислений

```javascript
// Создание объекта с кешированием вычислений
function createCachedCalculator() {
  const cache = new Map();
  
  const handler = {
    get(target, property) {
      if (typeof target[property] === 'function') {
        return function(...args) {
          const cacheKey = `${property}_${JSON.stringify(args)}`;
          
          if (cache.has(cacheKey)) {
            console.log(`Кеш попадание для ${property}`);
            return cache.get(cacheKey);
          }
          
          const result = target[property].apply(target, args);
          cache.set(cacheKey, result);
          return result;
        };
      }
      
      return Reflect.get(target, property);
    }
  };
  
  const calculator = {
    fibonacci(n) {
      if (n <= 1) return n;
      return this.fibonacci(n - 1) + this.fibonacci(n - 2);
    },
    
    factorial(n) {
      if (n <= 1) return 1;
      return n * this.factorial(n - 1);
    },
    
    expensiveOperation(data) {
      // Симуляция дорогой операции
      return data.map(item => item * 2).reduce((a, b) => a + b, 0);
    }
  };
  
  return new Proxy(calculator, handler);
}

// Использование кешированного калькулятора
const cachedCalc = createCachedCalculator();

console.time('Первый вызов');
console.log(cachedCalc.fibonacci(30)); // Выполняется полностью
console.timeEnd('Первый вызов');

console.time('Второй вызов');
console.log(cachedCalc.fibonacci(30)); // Из кеша
console.timeEnd('Второй вызов');
```

## Практический пример: Создание ORM-подобного интерфейса

```javascript
// Создание простой ORM с помощью Proxy
class SimpleORM {
  constructor(tableName) {
    this.tableName = tableName;
    this.data = [];
  }
  
  async find(query = {}) {
    // Симуляция поиска в "базе данных"
    return this.data.filter(item => {
      return Object.entries(query).every(([key, value]) => item[key] === value);
    });
  }
  
  async findOne(query = {}) {
    const results = await this.find(query);
    return results[0] || null;
  }
  
  async save(data) {
    // Симуляция сохранения
    const id = data.id || (this.data.length + 1);
    const record = { ...data, id };
    
    const existingIndex = this.data.findIndex(item => item.id === id);
    if (existingIndex !== -1) {
      this.data[existingIndex] = record;
    } else {
      this.data.push(record);
    }
    
    return record;
  }
}

function createORMProxy(tableName) {
  const orm = new SimpleORM(tableName);
  
  const handler = {
    get(target, property) {
      if (property in orm) {
        return orm[property].bind(orm);
      }
      
      // Префиксные методы типа findByName, findByEmail
      if (property.startsWith('findBy')) {
        const fieldName = property.slice(6).toLowerCase();
        
        return async (value) => {
          return orm.find({ [fieldName]: value });
        };
      }
      
      if (property.startsWith('findOneBy')) {
        const fieldName = property.slice(9).toLowerCase();
        
        return async (value) => {
          return orm.findOne({ [fieldName]: value });
        };
      }
      
      return undefined;
    }
  };
  
  return new Proxy({}, handler);
}

// Использование ORM прокси
const User = createORMProxy('users');

// Сохранение данных
User.save({ name: 'John', email: 'john@example.com', age: 30 });
User.save({ name: 'Jane', email: 'jane@example.com', age: 25 });

// Использование динамических методов
User.findByName('John').then(users => {
  console.log('Найденные пользователи:', users);
});

User.findOneByEmail('jane@example.com').then(user => {
  console.log('Найденный пользователь:', user);
});
```

## Практический пример: Обертка для API с логированием и повторными попытками

```javascript
// Создание API клиента с прокси для логирования и повторных попыток
function createAPIClient(baseURL) {
  const handler = {
    get(target, property) {
      if (typeof target[property] === 'function') {
        return async function(...args) {
          const startTime = Date.now();
          const endpoint = property;
          
          console.log(`Вызов API: ${baseURL}/${endpoint}`, args);
          
          let attempts = 0;
          const maxAttempts = 3;
          
          while (attempts < maxAttempts) {
            try {
              const result = await target[property].apply(target, args);
              
              console.log(`Успешный вызов API: ${endpoint}, время: ${Date.now() - startTime}ms`);
              return result;
            } catch (error) {
              attempts++;
              console.warn(`Попытка ${attempts} не удалась для ${endpoint}:`, error.message);
              
              if (attempts >= maxAttempts) {
                console.error(`Все попытки не удалась для ${endpoint}`);
                throw error;
              }
              
              // Ждем перед повторной попыткой
              await new Promise(resolve => setTimeout(resolve, 1000 * attempts));
            }
          }
        };
      }
      
      return Reflect.get(target, property);
    }
  };
  
  const api = {
    async getUsers() {
      const response = await fetch(`${baseURL}/users`);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response.json();
    },
    
    async getUser(id) {
      const response = await fetch(`${baseURL}/users/${id}`);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response.json();
    },
    
    async createUser(userData) {
      const response = await fetch(`${baseURL}/users`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      return response.json();
    }
  };
  
  return new Proxy(api, handler);
}

// Использование API клиента
const apiClient = createAPIClient('/api');

// Вызовы будут автоматически логироваться и с повторными попытками
apiClient.getUsers().then(users => {
  console.log('Полученные пользователи:', users);
}).catch(error => {
  console.error('Ошибка получения пользователей:', error);
});
```

## Лучшие практики использования Proxy в фронтенд разработке

### 1. Избегайте чрезмерного использования

```javascript
// Плохо - избыточное использование Proxy
function createOverProxy(obj) {
  return new Proxy(obj, {
    get(target, prop) {
      console.log(`Доступ к ${prop}`);
      return new Proxy(Reflect.get(target, prop), {
        get(nestedTarget, nestedProp) {
          console.log(`Доступ к ${prop}.${nestedProp}`);
          return Reflect.get(nestedTarget, nestedProp);
        }
      });
    }
  });
}

// Хорошо - целенаправленное использование
function createReactiveObject(obj, onChange) {
  return new Proxy(obj, {
    set(target, prop, value) {
      const oldValue = target[prop];
      const result = Reflect.set(target, prop, value);
      
      if (oldValue !== value) {
        onChange(prop, value, oldValue);
      }
      
      return result;
    }
  });
}
```

### 2. Правильная обработка ошибок

```javascript
// Важно правильно обрабатывать ошибки в прокси
function createSafeProxy(target) {
  const handler = {
    get(obj, prop) {
      try {
        const value = Reflect.get(obj, prop);
        
        // Если значение - функция, оборачиваем в безопасный вызов
        if (typeof value === 'function') {
          return function(...args) {
            try {
              return value.apply(obj, args);
            } catch (error) {
              console.error(`Ошибка вызова метода ${prop}:`, error);
              throw error;
            }
          };
        }
        
        return value;
      } catch (error) {
        console.error(`Ошибка доступа к свойству ${prop}:`, error);
        return undefined;
      }
    }
  };
  
  return new Proxy(target, handler);
}
```

## Заключение

Метапрограммирование и прокси в JavaScript фронтенд разработке позволяют создавать мощные, гибкие и интеллектуальные системы. Они особенно полезны для создания реактивных систем, валидации данных, логирования операций и создания удобных API. Однако важно использовать их осознанно, учитывая производительность и читаемость кода.

## См. также

- [[closure_practical_examples.md]]
- [[Прототипное наследование в JavaScript]]
- [[Функциональное программирование в JS]]
- [[Генераторы в JavaScript]]
- [[DOM манипуляции в JavaScript]]
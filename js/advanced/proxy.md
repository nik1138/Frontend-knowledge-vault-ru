# Proxy и Reflect в JavaScript

## Что такое Proxy

Proxy - это специальный объект в JavaScript, который позволяет перехватывать и модифицировать операции, выполняемые над другим объектом (называемым target). Это мощный инструмент метапрограммирования, который открывает возможности для создания сложных паттернов и библиотек.

## Основы Proxy

### Создание Proxy

```javascript
// Создание простого Proxy
const target = {
  name: "Иван",
  age: 30
};

const handler = {
  get(target, property) {
    console.log(`Доступ к свойству: ${property}`);
    return target[property];
  },
  
  set(target, property, value) {
    console.log(`Установка свойства ${property} в значение ${value}`);
    target[property] = value;
    return true; // Указывает, что операция прошла успешно
  }
};

const proxy = new Proxy(target, handler);

console.log(proxy.name); // "Доступ к свойству: name" и "Иван"
proxy.age = 31; // "Установка свойства age в значение 31"
```

### Перехват различных операций

```javascript
const target = {
  name: "Иван",
  privateData: "секрет"
};

const handler = {
  // Перехват доступа к свойству
  get(target, property) {
    if (property.startsWith('_')) {
      throw new Error(`Доступ к приватному свойству ${property} запрещен`);
    }
    return target[property];
  },
  
  // Перехват установки свойства
  set(target, property, value) {
    if (property.startsWith('_')) {
      throw new Error(`Установка приватного свойства ${property} запрещена`);
    }
    target[property] = value;
    return true;
  },
  
  // Перехват проверки наличия свойства
  has(target, property) {
    if (property.startsWith('_')) {
      return false; // Скрываем приватные свойства
    }
    return property in target;
  },
  
  // Перехват удаления свойства
  deleteProperty(target, property) {
    if (property.startsWith('_')) {
      throw new Error(`Удаление приватного свойства ${property} запрещено`);
    }
    delete target[property];
    return true;
  }
};

const proxy = new Proxy(target, handler);

console.log(proxy.name); // "Иван"
console.log('name' in proxy); // true
console.log('_privateData' in proxy); // false

// proxy._privateData; // Ошибка: Доступ к приватному свойству _privateData запрещен
```

## Продвинутые возможности Proxy

### Валидация данных

```javascript
function createValidatedObject(schema) {
  const target = {};
  
  const handler = {
    set(target, property, value) {
      // Проверка наличия схемы для свойства
      if (schema[property]) {
        const validator = schema[property];
        
        // Проверка типа
        if (validator.type && typeof value !== validator.type) {
          throw new TypeError(`Свойство ${property} должно быть типа ${validator.type}`);
        }
        
        // Проверка минимального значения
        if (validator.min !== undefined && value < validator.min) {
          throw new RangeError(`Свойство ${property} должно быть не меньше ${validator.min}`);
        }
        
        // Проверка максимального значения
        if (validator.max !== undefined && value > validator.max) {
          throw new RangeError(`Свойство ${property} должно быть не больше ${validator.max}`);
        }
        
        // Проверка длины строки
        if (validator.maxLength !== undefined && value.length > validator.maxLength) {
          throw new RangeError(`Свойство ${property} должно содержать не более ${validator.maxLength} символов`);
        }
      }
      
      target[property] = value;
      return true;
    }
  };
  
  return new Proxy(target, handler);
}

// Использование
const userSchema = {
  name: { type: 'string', maxLength: 50 },
  age: { type: 'number', min: 0, max: 150 },
  email: { type: 'string' }
};

const user = createValidatedObject(userSchema);

user.name = "Иван"; // OK
user.age = 30; // OK
// user.age = -5; // Ошибка: RangeError
// user.email = 123; // Ошибка: TypeError
```

### Создание "умных" объектов

```javascript
// Объект с автоматическим вычислением свойств
function createComputedObject(target) {
  const computed = new Map();
  
  const handler = {
    get(target, property) {
      // Если есть вычисляемое свойство
      if (computed.has(property)) {
        return computed.get(property).call(target);
      }
      
      return target[property];
    },
    
    set(target, property, value) {
      // Если устанавливаем функцию, делаем её вычисляемым свойством
      if (typeof value === 'function') {
        computed.set(property, value);
        return true;
      }
      
      target[property] = value;
      return true;
    }
  };
  
  return new Proxy(target, handler);
}

// Использование
const obj = createComputedObject({
  firstName: "Иван",
  lastName: "Иванов"
});

// Добавление вычисляемого свойства
obj.fullName = function() {
  return `${this.firstName} ${this.lastName}`;
};

obj.age = 30; // Обычное свойство
obj.isAdult = function() {
  return this.age >= 18;
};

console.log(obj.fullName()); // "Иван Иванов"
console.log(obj.isAdult()); // true
console.log(obj.age); // 30
```

## Reflect API

### Основы Reflect

Reflect - это встроенный объект, который предоставляет методы для перехвата JavaScript-операций. Эти методы аналогичны методам Proxy, но работают напрямую с объектами.

```javascript
// Использование Reflect вместо Object
const obj = { name: "Иван" };

// Вместо obj.hasOwnProperty('name')
console.log(Reflect.has(obj, 'name')); // true

// Вместо delete obj.name
Reflect.deleteProperty(obj, 'name');

// Вместо Object.defineProperty
Reflect.defineProperty(obj, 'age', {
  value: 30,
  writable: true,
  enumerable: true,
  configurable: true
});

console.log(Reflect.get(obj, 'age')); // 30
```

### Использование Reflect с Proxy

```javascript
const target = {
  name: "Иван",
  age: 30
};

const handler = {
  get(target, property, receiver) {
    console.log(`Получение свойства: ${property}`);
    // Использование Reflect.get для правильной передачи receiver
    return Reflect.get(target, property, receiver);
  },
  
  set(target, property, value, receiver) {
    console.log(`Установка свойства ${property} в ${value}`);
    // Использование Reflect.set для правильной передачи receiver
    return Reflect.set(target, property, value, receiver);
  },
  
  has(target, property) {
    console.log(`Проверка наличия свойства: ${property}`);
    return Reflect.has(target, property);
  }
};

const proxy = new Proxy(target, handler);

console.log(proxy.name); // "Получение свойства: name" и "Иван"
proxy.age = 31; // "Установка свойства age в 31"
console.log('name' in proxy); // "Проверка наличия свойства: name" и true
```

## Практические примеры

### Создание observable объекта

```javascript
// Система наблюдения за изменениями объекта
function createObservable(target, observer) {
  const handler = {
    set(target, property, value, receiver) {
      const oldValue = target[property];
      const result = Reflect.set(target, property, value, receiver);
      
      // Уведомляем наблюдателя об изменении
      if (oldValue !== value) {
        observer({
          type: 'set',
          property,
          value,
          oldValue,
          target
        });
      }
      
      return result;
    },
    
    deleteProperty(target, property) {
      const oldValue = target[property];
      const result = Reflect.deleteProperty(target, property);
      
      if (result) {
        observer({
          type: 'delete',
          property,
          oldValue,
          target
        });
      }
      
      return result;
    }
  };
  
  return new Proxy(target, handler);
}

// Использование
const data = { name: "Иван", age: 30 };

const observable = createObservable(data, (change) => {
  console.log('Изменение:', change);
});

observable.name = "Мария"; 
// Изменение: { type: 'set', property: 'name', value: 'Мария', oldValue: 'Иван', target: ... }

delete observable.age; 
// Изменение: { type: 'delete', property: 'age', oldValue: 30, target: ... }
```

### Создание immutable объекта

```javascript
// Неизменяемый объект
function createImmutable(target) {
  const handler = {
    set() {
      throw new Error('Невозможно изменить неизменяемый объект');
    },
    
    deleteProperty() {
      throw new Error('Невозможно удалить свойство из неизменяемого объекта');
    },
    
    defineProperty() {
      throw new Error('Невозможно определить новое свойство в неизменяемом объекте');
    }
  };
  
  return new Proxy(target, handler);
}

// Использование
const config = createImmutable({
  apiUrl: 'https://api.example.com',
  timeout: 5000
});

console.log(config.apiUrl); // 'https://api.example.com'
// config.timeout = 10000; // Ошибка: Невозможно изменить неизменяемый объект
```

### Создание объекта с отложенной инициализацией

```javascript
// Объект с отложенной инициализацией свойств
function createLazyObject(factory) {
  const cache = new Map();
  const target = {};
  
  const handler = {
    get(target, property) {
      // Если значение уже в кэше, возвращаем его
      if (cache.has(property)) {
        return cache.get(property);
      }
      
      // Если есть фабричная функция для свойства, вызываем её
      if (factory[property] && typeof factory[property] === 'function') {
        const value = factory[property]();
        cache.set(property, value);
        return value;
      }
      
      return target[property];
    }
  };
  
  return new Proxy(target, handler);
}

// Использование
const lazyObj = createLazyObject({
  expensiveData() {
    console.log('Вычисление дорогих данных...');
    // Имитация дорогой операции
    return Array.from({ length: 1000000 }, (_, i) => i);
  },
  
  currentUser() {
    console.log('Получение текущего пользователя...');
    // Имитация запроса к API
    return { name: "Иван", role: "admin" };
  }
});

console.log('Первый доступ:');
console.log(lazyObj.expensiveData.length); // "Вычисление дорогих данных..." и 1000000

console.log('Повторный доступ:');
console.log(lazyObj.expensiveData.length); // 1000000 (без повторного вычисления)
```

### Создание объекта с логированием

```javascript
// Объект с автоматическим логированием всех операций
function createLoggedObject(target, logger = console) {
  const handler = {
    get(target, property, receiver) {
      logger.log(`[GET] ${String(property)}`);
      return Reflect.get(target, property, receiver);
    },
    
    set(target, property, value, receiver) {
      logger.log(`[SET] ${String(property)} = ${JSON.stringify(value)}`);
      return Reflect.set(target, property, value, receiver);
    },
    
    has(target, property) {
      logger.log(`[HAS] ${String(property)}`);
      return Reflect.has(target, property);
    },
    
    deleteProperty(target, property) {
      logger.log(`[DELETE] ${String(property)}`);
      return Reflect.deleteProperty(target, property);
    },
    
    apply(target, thisArg, argumentsList) {
      logger.log(`[APPLY] ${target.name}(${argumentsList.map(JSON.stringify).join(', ')})`);
      return Reflect.apply(target, thisArg, argumentsList);
    }
  };
  
  return new Proxy(target, handler);
}

// Использование
const user = createLoggedObject({
  name: "Иван",
  greet(greeting = "Привет") {
    return `${greeting}, меня зовут ${this.name}`;
  }
});

console.log(user.name); // [GET] name и "Иван"
user.name = "Мария"; // [SET] name = "Мария"
console.log('age' in user); // [HAS] age и false
user.greet("Здравствуйте"); // [GET] greet и [APPLY] greet("Здравствуйте, меня зовут Мария")
```

## Продвинутые паттерны

### Создание ORM-подобного интерфейса

```javascript
// Простая ORM с использованием Proxy
class Model {
  constructor(data = {}) {
    this._data = data;
    this._changes = new Set();
    
    return new Proxy(this, {
      get(target, property) {
        // Если свойство существует в объекте, возвращаем его
        if (property in target) {
          return target[property];
        }
        
        // Иначе возвращаем данные из _data
        return target._data[property];
      },
      
      set(target, property, value) {
        // Устанавливаем свойства объекта напрямую
        if (property.startsWith('_')) {
          target[property] = value;
          return true;
        }
        
        // Для остальных свойств сохраняем в _data и отслеживаем изменения
        if (target._data[property] !== value) {
          target._data[property] = value;
          target._changes.add(property);
        }
        
        return true;
      }
    });
  }
  
  save() {
    if (this._changes.size > 0) {
      console.log('Сохранение изменений:', Array.from(this._changes));
      this._changes.clear();
      return true;
    }
    console.log('Нет изменений для сохранения');
    return false;
  }
  
  toJSON() {
    return { ...this._data };
  }
}

// Использование
class User extends Model {
  constructor(data) {
    super(data);
  }
  
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

const user = new User({ firstName: "Иван", lastName: "Иванов", age: 30 });

console.log(user.firstName); // "Иван"
console.log(user.fullName); // "Иван Иванов"

user.age = 31; // Изменение отслеживается
user.email = "ivan@example.com"; // Добавление нового свойства

user.save(); // "Сохранение изменений: ['age', 'email']"
```

### Создание системы доступа к API

```javascript
// Proxy для автоматического создания API клиентов
function createApiClient(baseURL) {
  const api = {};
  
  return new Proxy(api, {
    get(target, endpoint) {
      // Если метод уже существует, возвращаем его
      if (target[endpoint]) {
        return target[endpoint];
      }
      
      // Создаем метод для работы с endpoint
      const method = async function(data, options = {}) {
        const url = `${baseURL}/${endpoint}`;
        const config = {
          method: data ? 'POST' : 'GET',
          headers: {
            'Content-Type': 'application/json'
          },
          ...options
        };
        
        if (data) {
          config.body = JSON.stringify(data);
        }
        
        try {
          const response = await fetch(url, config);
          if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
          }
          return await response.json();
        } catch (error) {
          console.error(`Ошибка при запросе к ${url}:`, error.message);
          throw error;
        }
      };
      
      // Кэшируем метод
      target[endpoint] = method;
      return method;
    }
  });
}

// Использование
const api = createApiClient('https://jsonplaceholder.typicode.com');

// Автоматически созданные методы
api.users().then(users => console.log('Пользователи:', users));
api.posts({ title: 'Новый пост', body: 'Содержание поста' })
  .then(post => console.log('Создан пост:', post));
```

## Лучшие практики

### 1. Используйте Reflect для правильной передачи контекста

```javascript
// Плохо: неправильная передача контекста
const handler = {
  get(target, property) {
    return target[property]; // Не передает receiver
  }
};

// Хорошо: правильная передача контекста
const handler = {
  get(target, property, receiver) {
    return Reflect.get(target, property, receiver);
  }
};
```

### 2. Обрабатывайте все необходимые ловушки

```javascript
// Хорошо: обработка всех важных операций
const handler = {
  get(target, property, receiver) {
    return Reflect.get(target, property, receiver);
  },
  
  set(target, property, value, receiver) {
    return Reflect.set(target, property, value, receiver);
  },
  
  has(target, property) {
    return Reflect.has(target, property);
  },
  
  deleteProperty(target, property) {
    return Reflect.deleteProperty(target, property);
  }
};
```

### 3. Избегайте избыточного использования Proxy

```javascript
// Плохо: использование Proxy для простых случаев
const simpleObj = new Proxy({}, {
  get(target, property) {
    return target[property];
  }
});

// Хорошо: использование обычных объектов для простых случаев
const simpleObj = {};
```

## Производительность

### Оптимизация Proxy

```javascript
// Кэширование часто используемых свойств
function createOptimizedProxy(target) {
  const cache = new Map();
  const frequentlyAccessed = new Set(['id', 'name', 'type']);
  
  const handler = {
    get(target, property, receiver) {
      // Для часто используемых свойств используем кэш
      if (frequentlyAccessed.has(property) && cache.has(property)) {
        return cache.get(property);
      }
      
      const value = Reflect.get(target, property, receiver);
      
      // Кэшируем часто используемые свойства
      if (frequentlyAccessed.has(property)) {
        cache.set(property, value);
      }
      
      return value;
    },
    
    set(target, property, value, receiver) {
      const result = Reflect.set(target, property, value, receiver);
      
      // Обновляем кэш при изменении
      if (frequentlyAccessed.has(property)) {
        cache.set(property, value);
      }
      
      return result;
    }
  };
  
  return new Proxy(target, handler);
}
```

## Совместимость и ограничения

### Поддержка браузерами

Proxy поддерживается всеми современными браузерами, но не поддерживается в Internet Explorer. Для старых браузеров можно использовать полифилы или альтернативные решения.

### Ограничения Proxy

```javascript
// Некоторые операции не могут быть перехвачены
const obj = {};
const proxy = new Proxy(obj, {
  get() {
    console.log('Перехват');
    return 'значение';
  }
});

// Эти операции не могут быть перехвачены:
console.log(Object.isExtensible(obj)); // Не вызывает ловушку
Object.preventExtensions(obj); // Не вызывает ловушку
Object.getOwnPropertyNames(obj); // Не вызывает ловушку
```

## Заключение

Proxy и Reflect - мощные инструменты метапрограммирования в JavaScript, которые позволяют перехватывать и модифицировать операции над объектами. Они открывают возможности для создания сложных паттернов, библиотек и фреймворков. Однако их следует использовать с осторожностью, так как они могут повлиять на производительность и читаемость кода.

Правильное использование Proxy и Reflect позволяет создавать гибкие и расширяемые решения, такие как системы наблюдения, валидации, ORM и другие. Понимание этих механизмов критически важно для разработчиков, создающих библиотеки и фреймворки, а также для тех, кто хочет глубже понять внутреннее устройство JavaScript.

#javascript #programming #proxy #reflect #metaprogramming #web-development
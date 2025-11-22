---
aliases: ["Совместимость с браузерами", "JS Browser Compatibility"]
tags: [javascript, browser, compatibility, frontend]
---

# Работа с различиями в браузерах

Совместимость с различными браузерами — важная задача при разработке веб-приложений. В этом документе собраны техники и подходы для обеспечения корректной работы JavaScript кода в разных браузерах.

## Определение браузера и версии

### 1. Определение браузера по User Agent

```javascript
// Базовое определение браузера
function getBrowserInfo() {
  const userAgent = navigator.userAgent;
  
  // Определение браузера
  const browsers = {
    chrome: /Chrome/i.test(userAgent) && !/Edg/i.test(userAgent),
    firefox: /Firefox/i.test(userAgent),
    safari: /Safari/i.test(userAgent) && !/Chrome/i.test(userAgent),
    edge: /Edg/i.test(userAgent),
    ie: /MSIE|Trident/i.test(userAgent)
  };
  
  // Определение версии
  let version = 'Unknown';
  const match = userAgent.match(/(chrome|firefox|safari|edge|msie|trident)\/?\s*(\.?\d+)/i);
  if (match) {
    version = match[2];
  }
  
  return { browsers, version, userAgent };
}

console.log(getBrowserInfo());
```

> [!warning] Важно
> Определение браузера по User Agent не всегда надежно. Рекомендуется использовать feature detection вместо browser detection.

### 2. Feature Detection (обнаружение возможностей)

```javascript
// Проверка поддержки возможностей браузера
const featureSupport = {
  // Проверка поддержки ES6 возможностей
  es6: {
    arrowFunctions: (() => true)() || false,
    templateLiterals: typeof '' === 'string',
    letConst: (function() {
      try {
        eval('let a = 1; const b = 2;');
        return true;
      } catch (e) {
        return false;
      }
    })(),
    promises: typeof Promise !== 'undefined',
    fetch: typeof fetch !== 'undefined',
    asyncAwait: (function() {
      try {
        eval('async function test() {}');
        return true;
      } catch (e) {
        return false;
      }
    })()
  },
  
  // Проверка поддержки DOM API
  dom: {
    querySelector: typeof document.querySelector !== 'undefined',
    addEventListener: typeof document.addEventListener !== 'undefined',
    classList: typeof document.documentElement.classList !== 'undefined',
    dataset: typeof document.documentElement.dataset !== 'undefined',
    localStorage: typeof Storage !== 'undefined',
    historyAPI: typeof history.pushState !== 'undefined'
  },
  
  // Проверка поддержки Canvas
  canvas: (function() {
    const canvas = document.createElement('canvas');
    return !!(canvas.getContext && canvas.getContext('2d'));
  })(),
  
  // Проверка поддержки WebGL
  webgl: (function() {
    try {
      const canvas = document.createElement('canvas');
      return !!(window.WebGLRenderingContext && 
                (canvas.getContext('webgl') || canvas.getContext('experimental-webgl')));
    } catch (e) {
      return false;
    }
  })()
};

console.log('Поддержка возможностей:', featureSupport);
```

## Совместимость с различными версиями JavaScript

### 1. Polyfills для старых браузеров

```javascript
// Polyfill для Object.assign
if (typeof Object.assign !== 'function') {
  Object.assign = function(target) {
    if (target === null || target === undefined) {
      throw new TypeError('Cannot convert undefined or null to object');
    }
    
    const to = Object(target);
    
    for (let index = 1; index < arguments.length; index++) {
      const nextSource = arguments[index];
      
      if (nextSource !== null && nextSource !== undefined) {
        for (const nextKey in nextSource) {
          if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
            to[nextKey] = nextSource[nextKey];
          }
        }
      }
    }
    
    return to;
  };
}

// Polyfill для Array.includes
if (!Array.prototype.includes) {
  Array.prototype.includes = function(searchElement, fromIndex) {
    return this.indexOf(searchElement, fromIndex) !== -1;
  };
}

// Polyfill для String.startsWith
if (!String.prototype.startsWith) {
  String.prototype.startsWith = function(searchString, position) {
    position = position || 0;
    return this.indexOf(searchString, position) === position;
  };
}
```

### 2. Совместимость с ES6+ возможностями

```javascript
// Использование Babel для транспиляции ES6+ кода
// Но также можно реализовать собственные решения:

// Замена стрелочных функций
// ES6: const add = (a, b) => a + b;
// ES5: var add = function(a, b) { return a + b; };

// Замена let/const для старых браузеров
// ES6: let variable = 'value';
// ES5: var variable = 'value';

// Замена деструктуризации
function processUser_ES5(user) {
  var name = user.name;
  var age = user.age;
  var email = user.email;
  
  // обработка данных
}

function processUser_ES6({ name, age, email }) {
  // обработка данных
}

// Замена spread оператора
function sum_ES5() {
  var numbers = Array.prototype.slice.call(arguments);
  return numbers.reduce(function(acc, num) {
    return acc + num;
  }, 0);
}

function sum_ES6(...numbers) {
  return numbers.reduce((acc, num) => acc + num, 0);
}
```

## Решение проблем с различными браузерами

### 1. Проблемы с CSSOM и стилями

```javascript
// Получение вычисленных стилей с учетом браузерных различий
function getComputedStyles(element, property) {
  if (window.getComputedStyle) {
    return window.getComputedStyle(element)[property];
  } else if (element.currentStyle) {
    // Internet Explorer
    return element.currentStyle[property];
  }
  return element.style[property];
}

// Применение стилей с префиксами для разных браузеров
function setStyleWithPrefixes(element, property, value) {
  const prefixes = ['', 'webkit', 'moz', 'ms', 'o'];
  
  prefixes.forEach(prefix => {
    const prefixedProperty = prefix ? 
      prefix + property.charAt(0).toUpperCase() + property.slice(1) : 
      property;
    
    if (element.style[prefixedProperty] !== undefined) {
      element.style[prefixedProperty] = value;
    }
  });
}

// Пример использования
const element = document.querySelector('.my-element');
setStyleWithPrefixes(element, 'transform', 'translateX(100px)');
```

### 2. Работа с событиями

```javascript
// Кросс-браузерная работа с событиями
const EventUtil = {
  addHandler: function(element, type, handler) {
    if (element.addEventListener) {
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent) {
      element.attachEvent('on' + type, handler);
    } else {
      element['on' + type] = handler;
    }
  },
  
  removeHandler: function(element, type, handler) {
    if (element.removeEventListener) {
      element.removeEventListener(type, handler, false);
    } else if (element.detachEvent) {
      element.detachEvent('on' + type, handler);
    } else {
      element['on' + type] = null;
    }
  },
  
  getEvent: function(event) {
    return event ? event : window.event;
  },
  
  getTarget: function(event) {
    return event.target || event.srcElement;
  },
  
  preventDefault: function(event) {
    if (event.preventDefault) {
      event.preventDefault();
    } else {
      event.returnValue = false;
    }
  },
  
  stopPropagation: function(event) {
    if (event.stopPropagation) {
      event.stopPropagation();
    } else {
      event.cancelBubble = true;
    }
  }
};

// Использование
const button = document.getElementById('my-button');
EventUtil.addHandler(button, 'click', function(event) {
  event = EventUtil.getEvent(event);
  const target = EventUtil.getTarget(event);
  
  EventUtil.preventDefault(event);
  EventUtil.stopPropagation(event);
  
  console.log('Кнопка нажата');
});
```

### 3. Совместимость с localStorage и sessionStorage

```javascript
// Безопасная работа с localStorage
const StorageUtil = {
  setItem: function(key, value) {
    try {
      if (typeof(Storage) !== 'undefined') {
        localStorage.setItem(key, JSON.stringify(value));
        return true;
      }
    } catch (e) {
      console.warn('localStorage недоступен:', e.message);
      // Резервное хранилище в памяти
      if (!this.memoryStorage) {
        this.memoryStorage = {};
      }
      this.memoryStorage[key] = value;
    }
    return false;
  },
  
  getItem: function(key) {
    try {
      if (typeof(Storage) !== 'undefined') {
        const item = localStorage.getItem(key);
        return item ? JSON.parse(item) : null;
      }
    } catch (e) {
      console.warn('localStorage недоступен:', e.message);
      // Проверка резервного хранилища
      return this.memoryStorage && this.memoryStorage[key] || null;
    }
    return null;
  },
  
  removeItem: function(key) {
    try {
      if (typeof(Storage) !== 'undefined') {
        localStorage.removeItem(key);
        return true;
      }
    } catch (e) {
      console.warn('localStorage недоступен:', e.message);
      if (this.memoryStorage) {
        delete this.memoryStorage[key];
      }
    }
    return false;
  }
};
```

## Совместимость с различными API

### 1. Совместимость с Fetch API

```javascript
// Polyfill для Fetch API или использование XMLHttpRequest
function fetchWithFallback(url, options = {}) {
  // Проверяем поддержку Fetch API
  if (typeof fetch !== 'undefined') {
    return fetch(url, options);
  }
  
  // Резервный вариант с XMLHttpRequest
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open(options.method || 'GET', url, true);
    
    // Установка заголовков
    if (options.headers) {
      Object.keys(options.headers).forEach(key => {
        xhr.setRequestHeader(key, options.headers[key]);
      });
    }
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve({
          ok: true,
          status: xhr.status,
          statusText: xhr.statusText,
          text: () => Promise.resolve(xhr.responseText),
          json: () => Promise.resolve(JSON.parse(xhr.responseText)),
          headers: {}
        });
      } else {
        reject(new Error(`HTTP Error: ${xhr.status} ${xhr.statusText}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Network Error'));
    };
    
    xhr.send(options.body || null);
  });
}

// Использование
fetchWithFallback('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Ошибка:', error));
```

### 2. Совместимость с Promise

```javascript
// Проверка поддержки Promise и создание резервного варианта
const PromiseUtil = (function() {
  if (typeof Promise !== 'undefined') {
    return Promise;
  }
  
  // Простая реализация Promise для очень старых браузеров
  function SimplePromise(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.handlers = [];
    
    const resolve = (value) => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.handlers.forEach(handleFulfillment);
      }
    };
    
    const reject = (reason) => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.value = reason;
        this.handlers.forEach(handleRejection);
      }
    };
    
    try {
      executor(resolve, reject);
    } catch (error) {
      reject(error);
    }
  }
  
  SimplePromise.prototype.then = function(onFulfilled, onRejected) {
    return new SimplePromise((resolve, reject) => {
      const handler = {
        onFulfilled: typeof onFulfilled === 'function' ? onFulfilled : null,
        onRejected: typeof onRejected === 'function' ? onRejected : null,
        resolve: resolve,
        reject: reject
      };
      
      this.handlers.push(handler);
      
      if (this.state !== 'pending') {
        const processQueue = this.state === 'fulfilled' ? handleFulfillment : handleRejection;
        processQueue.call(this, handler);
      }
    });
  };
  
  function handleFulfillment(handler) {
    if (!handler.onFulfilled) {
      handler.resolve(this.value);
      return;
    }
    
    try {
      const result = handler.onFulfilled(this.value);
      handler.resolve(result);
    } catch (error) {
      handler.reject(error);
    }
  }
  
  function handleRejection(handler) {
    if (!handler.onRejected) {
      handler.reject(this.value);
      return;
    }
    
    try {
      const result = handler.onRejected(this.value);
      handler.resolve(result);
    } catch (error) {
      handler.reject(error);
    }
  }
  
  return SimplePromise;
})();

// Использование
const promise = new PromiseUtil((resolve, reject) => {
  setTimeout(() => resolve('Успешно!'), 1000);
});

promise.then(result => console.log(result));
```

## Практические рекомендации

### 1. Тестирование в разных браузерах

```javascript
// Утилита для тестирования совместимости
const CompatibilityTester = {
  tests: [],
  
  addTest: function(name, testFunction) {
    this.tests.push({ name, testFunction });
  },
  
  runAllTests: function() {
    const results = {};
    
    this.tests.forEach(test => {
      try {
        results[test.name] = test.testFunction();
      } catch (e) {
        results[test.name] = { error: e.message, supported: false };
      }
    });
    
    return results;
  }
};

// Пример добавления тестов
CompatibilityTester.addTest('Promise Support', () => ({
  supported: typeof Promise !== 'undefined',
  version: typeof Promise !== 'undefined' ? 'ES6' : 'Not supported'
}));

CompatibilityTester.addTest('Fetch API Support', () => ({
  supported: typeof fetch !== 'undefined',
  version: typeof fetch !== 'undefined' ? 'Modern' : 'Not supported'
}));

// Запуск тестов
const compatibilityResults = CompatibilityTester.runAllTests();
console.table(compatibilityResults);
```

### 2. Управление совместимостью в проекте

```javascript
// Конфигурация совместимости
const CompatibilityConfig = {
  // Минимальные версии браузеров
  minVersions: {
    chrome: 49,
    firefox: 45,
    safari: 10,
    edge: 14,
    ie: 11
  },
  
  // Необходимые polyfills
  requiredPolyfills: [
    'Promise',
    'fetch',
    'Array.prototype.includes',
    'Object.assign'
  ],
  
  // Функции для проверки поддержки
  checkFunctions: {
    es6: () => typeof Symbol !== 'undefined',
    modules: () => typeof import !== 'undefined',
    async: () => {
      try {
        eval('async function test() {}');
        return true;
      } catch (e) {
        return false;
      }
    }
  }
};

// Проверка совместимости при загрузке
function checkCompatibility() {
  const results = {};
  
  for (const [feature, checkFn] of Object.entries(CompatibilityConfig.checkFunctions)) {
    results[feature] = checkFn();
  }
  
  // Если есть проблемы совместимости
  if (!results.es6) {
    console.warn('Браузер не поддерживает ES6. Загрузка полифилов...');
    // Логика загрузки полифилов
  }
  
  return results;
}

// Вызов проверки
const compatibility = checkCompatibility();
```

## Связанные темы

- [[javascript-polyfills-guide]]
- [[cross-browser-css-techniques]]
- [[feature-detection-vs-browser-detection]]
---
aliases: ["Техники отладки JavaScript", "JS Debugging"]
tags: [javascript, debugging, frontend, development]
---

# Техники отладки JavaScript кода

Эффективная отладка — ключевой навык для разработчика. В этом документе собраны полезные техники и инструменты для отладки JavaScript кода в браузере и в Node.js.

## Встроенные инструменты отладки

### 1. Использование console.log()

Самый простой способ отладки, но не всегда самый эффективный:

```javascript
function calculateTotal(items) {
  console.log('Полученные items:', items); // Логируем входные данные
  
  let total = 0;
  for (let i = 0; i < items.length; i++) {
    const item = items[i];
    console.log(`Обрабатываем элемент ${i}:`, item); // Логируем каждый элемент
    
    total += item.price * item.quantity;
    console.log(`Промежуточный итог: ${total}`); // Логируем промежуточный результат
  }
  
  console.log('Итоговый результат:', total); // Логируем финальный результат
  return total;
}
```

> [!tip] Совет
> Используйте `console.log()` с метками для лучшей читаемости: `console.log('DEBUG: ', variable)`

### 2. Другие методы console

```javascript
// console.table() для табличного отображения массивов/объектов
const users = [
  { id: 1, name: 'John', age: 25 },
  { id: 2, name: 'Jane', age: 30 }
];
console.table(users);

// console.group() для группировки логов
console.group('Расчеты');
console.log('Шаг 1: Инициализация');
console.log('Шаг 2: Вычисление');
console.groupEnd();

// console.time() для измерения времени выполнения
console.time('Выполнение операции');
// ... код ...
console.timeEnd('Выполнение операции');

// console.trace() для отображения стека вызовов
function a() { b(); }
function b() { c(); }
function c() { console.trace('Стек вызовов:'); }
a();
```

### 3. Точки останова (breakpoints)

Использование `debugger` в коде:

```javascript
function processData(data) {
  if (data.length > 100) {
    debugger; // Точка останова
    // Код выполнится до этой точки, затем остановится в отладчике
  }
  
  return data.map(item => item.value * 2);
}
```

## Продвинутые техники отладки

### 1. Условные точки останова

В отладчике браузера можно установить условные точки останова:

```javascript
// Вместо этого:
for (let i = 0; i < 1000; i++) {
  if (i === 500) { // Условие для отладки
    debugger;
  }
  // обработка данных
}

// Лучше использовать условную точку останова в отладчике
// Установите точку останова и добавьте условие: i === 500
```

### 2. Отладка асинхронного кода

```javascript
// Отладка Promise
async function fetchUserData(userId) {
  try {
    console.log('Запрашиваем данные пользователя:', userId);
    const response = await fetch(`/api/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`Ошибка HTTP: ${response.status}`);
    }
    
    const userData = await response.json();
    console.log('Полученные данные:', userData);
    
    return userData;
  } catch (error) {
    console.error('Ошибка при получении данных пользователя:', error);
    throw error;
  }
}

// Отладка с использованием Promise.catch()
fetchUserData(123)
  .then(data => console.log('Данные:', data))
  .catch(error => console.error('Ошибка:', error));
```

### 3. Отслеживание изменений переменных

```javascript
// Использование Object.defineProperty для отслеживания изменений
function trackProperty(obj, prop, callback) {
  let value = obj[prop];
  
  Object.defineProperty(obj, prop, {
    get: function() {
      return value;
    },
    set: function(newValue) {
      console.log(`Свойство ${prop} изменено с ${value} на ${newValue}`);
      if (callback) callback(newValue);
      value = newValue;
    }
  });
}

const user = { name: 'John' };
trackProperty(user, 'name', (newName) => {
  console.log('Имя пользователя изменено на:', newName);
});

user.name = 'Jane'; // Вызовет отладочное сообщение
```

### 4. Отладка производительности

```javascript
// Измерение производительности функций
function performanceDecorator(fn, fnName) {
  return function(...args) {
    console.time(`${fnName} выполнение`);
    const result = fn.apply(this, args);
    console.timeEnd(`${fnName} выполнение`);
    return result;
  };
}

// Пример использования
const slowFunction = performanceDecorator(function(n) {
  let sum = 0;
  for (let i = 0; i < n; i++) {
    sum += i;
  }
  return sum;
}, 'slowFunction');

slowFunction(1000000);
```

## Инструменты отладки браузера

### 1. Network панель

Для отладки сетевых запросов:

```javascript
// Логирование сетевых ошибок
async function apiCall(endpoint, options = {}) {
  try {
    console.log('Отправка запроса:', endpoint);
    const response = await fetch(endpoint, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      }
    });
    
    console.log('Ответ получен:', response.status, response.statusText);
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка! статус: ${response.status}`);
    }
    
    const data = await response.json();
    console.log('Данные:', data);
    return data;
  } catch (error) {
    console.error('Ошибка запроса:', error);
    throw error;
  }
}
```

### 2. Memory панель

Для отслеживания утечек памяти:

```javascript
// Отслеживание утечек памяти
class Component {
  constructor() {
    this.data = new Array(1000000).fill(0);
    this.eventHandlers = [];
    
    // Регистрация обработчиков событий
    this.addHandler();
  }
  
  addHandler() {
    const handler = () => console.log('Событие');
    document.addEventListener('click', handler);
    this.eventHandlers.push(handler);
  }
  
  destroy() {
    // Очистка обработчиков событий для предотвращения утечек
    this.eventHandlers.forEach((handler, index) => {
      document.removeEventListener('click', this.eventHandlers[index]);
    });
    this.eventHandlers = [];
    this.data = null;
  }
}
```

### 3. Sources панель

Для установки точек останова и пошаговой отладки:

```javascript
// Использование console.assert() для проверки условий
function divide(a, b) {
  console.assert(b !== 0, 'Деление на ноль!', a, b);
  return a / b;
}

// Использование console.count() для подсчета вызовов
function renderComponent() {
  console.count('Количество рендеров');
  // логика рендера
}

renderComponent(); // "Количество рендеров: 1"
renderComponent(); // "Количество рендеров: 2"
```

## Отладка в продакшене

### 1. Удаленная отладка

```javascript
// Логирование ошибок на удаленный сервер
class RemoteLogger {
  static log(level, message, data = {}) {
    // В реальном приложении отправлять на специальный сервис
    const logData = {
      level,
      message,
      data,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      url: window.location.href
    };
    
    // Отправка лога (в реальном приложении через fetch или WebSocket)
    console.log('Отправка лога:', logData);
  }
  
  static error(message, error) {
    this.log('error', message, {
      errorMessage: error.message,
      errorStack: error.stack
    });
  }
}

// Использование
try {
  // какой-то код
} catch (error) {
  RemoteLogger.error('Произошла ошибка в компоненте', error);
}
```

### 2. Условная отладка

```javascript
// Управление отладкой через конфигурацию
const DEBUG_CONFIG = {
  enabled: process.env.NODE_ENV !== 'production',
  logLevel: 'debug', // debug, info, warn, error
  modules: {
    auth: true,
    api: true,
    ui: false
  }
};

function debugLog(module, message, data) {
  if (DEBUG_CONFIG.enabled && DEBUG_CONFIG.modules[module]) {
    console.log(`[${module.toUpperCase()}] ${message}`, data);
  }
}

// Использование
debugLog('auth', 'Пользователь вошел', { userId: 123 });
```

## Практические советы

- Используйте именованные точки останова для лучшей организации
- Не забывайте удалять временные `console.log()` перед деплоем
- Используйте conditional breakpoints для отладки циклов
- Применяйте `console.profile()` и `console.profileEnd()` для профилирования
- Используйте `monitorEvents()` в консоли для отслеживания событий DOM элементов

## Связанные темы

- [[javascript-error-handling]]
- [[performance-optimization-techniques]]
- [[browser-development-tools]]
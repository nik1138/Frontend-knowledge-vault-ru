# Отладка в JavaScript

## Введение

Отладка - это процесс обнаружения, локализации и устранения ошибок в коде. Эффективная отладка критически важна для разработки качественных веб-приложений. JavaScript предоставляет множество инструментов и техник для отладки, от простых console.log до продвинутых профилировщиков производительности.

## Основные инструменты отладки

### 1. Консоль разработчика

Консоль разработчика - основной инструмент отладки в браузере. Она предоставляет доступ к различным панелям для анализа кода, сетевых запросов, производительности и других аспектов приложения.

```javascript
// Основные методы console
console.log('Обычное сообщение');
console.info('Информационное сообщение');
console.warn('Предупреждение');
console.error('Ошибка');

// Форматирование вывода
console.log('Пользователь: %s, Возраст: %d', 'Иван', 25);
console.log('Объект:', { name: 'Иван', age: 25 });

// Группировка сообщений
console.group('Группа сообщений');
console.log('Сообщение 1');
console.log('Сообщение 2');
console.groupEnd();

// Таблицы для массивов объектов
const users = [
  { name: 'Иван', age: 25 },
  { name: 'Мария', age: 30 },
  { name: 'Алексей', age: 35 }
];
console.table(users);

// Трассировка стека вызовов
function functionA() {
  functionB();
}

function functionB() {
  functionC();
}

function functionC() {
  console.trace('Трассировка стека вызовов');
}

functionA();

// Условный вывод
const debugMode = true;
console.debug('Отладочное сообщение', debugMode ? 'включен' : 'выключен');

// Вывод времени выполнения
console.time('Тест производительности');
for (let i = 0; i < 1000000; i++) {
  // Некоторая операция
}
console.timeEnd('Тест производительности');

// Утверждения
const userAge = -5;
console.assert(userAge >= 0, 'Возраст не может быть отрицательным:', userAge);
```

### 2. Точки останова (Breakpoints)

Точки останова позволяют приостановить выполнение кода в определенном месте для анализа состояния приложения.

```javascript
function calculateTotal(items) {
  let total = 0;
  
  // Точка останова в коде
  debugger;
  
  for (let i = 0; i < items.length; i++) {
    total += items[i].price * items[i].quantity;
  }
  
  return total;
}

// Условные точки останова
function processItems(items) {
  for (let i = 0; i < items.length; i++) {
    // Остановка только при определенном условии
    if (i === 5) {
      debugger; // Остановка на 6-м элементе
    }
    
    // Обработка элемента
    items[i].processed = true;
  }
}

// Логические точки останова
function findUser(users, targetId) {
  for (let i = 0; i < users.length; i++) {
    // Логирование без остановки
    console.log('Проверка пользователя:', users[i].id);
    
    if (users[i].id === targetId) {
      // Остановка при нахождении нужного пользователя
      debugger;
      return users[i];
    }
  }
  
  return null;
}
```

### 3. Инспектор объектов

Инспектор объектов позволяет детально изучить структуру и свойства объектов во время отладки.

```javascript
// Раскрытие объектов в консоли
const complexObject = {
  name: 'Продукт',
  price: 100,
  category: {
    id: 1,
    name: 'Электроника'
  },
  tags: ['новинка', 'популярный'],
  metadata: {
    createdAt: new Date(),
    updatedAt: new Date(),
    author: {
      id: 1,
      name: 'Автор'
    }
  }
};

console.log(complexObject); // Сворачиваемый объект
console.dir(complexObject); // Детальный просмотр свойств

// Инспекция DOM элементов
const element = document.querySelector('.product-card');
console.log(element); // HTML представление
console.dir(element); // Свойства элемента

// Инспекция функций
function sampleFunction(a, b) {
  return a + b;
}

console.log(sampleFunction); // Функция как объект
console.dir(sampleFunction); // Детали функции (параметры, тело)
```

## Продвинутые техники отладки

### 1. Отладка асинхронного кода

Отладка асинхронного кода требует специальных подходов из-за его неблокирующей природы.

```javascript
// Отладка Promise
async function fetchUserData(userId) {
  try {
    console.log('Начало загрузки данных пользователя:', userId);
    
    // Точка останова перед асинхронной операцией
    debugger;
    
    const response = await fetch(`/api/users/${userId}`);
    
    // Точка останова после получения ответа
    debugger;
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const userData = await response.json();
    
    // Точка останова после парсинга данных
    debugger;
    
    console.log('Данные пользователя загружены:', userData);
    return userData;
  } catch (error) {
    console.error('Ошибка загрузки данных пользователя:', error);
    debugger; // Точка останова при ошибке
    throw error;
  }
}

// Отладка цепочек Promise
function processUserData(users) {
  return Promise.resolve(users)
    .then(users => {
      console.log('Фильтрация пользователей');
      debugger;
      return users.filter(user => user.active);
    })
    .then(activeUsers => {
      console.log('Сортировка пользователей');
      debugger;
      return activeUsers.sort((a, b) => a.name.localeCompare(b.name));
    })
    .then(sortedUsers => {
      console.log('Форматирование пользователей');
      debugger;
      return sortedUsers.map(user => ({
        id: user.id,
        displayName: `${user.firstName} ${user.lastName}`,
        email: user.email
      }));
    })
    .catch(error => {
      console.error('Ошибка обработки пользователей:', error);
      debugger;
      throw error;
    });
}

// Отладка async/await с обработкой ошибок
async function complexAsyncOperation() {
  try {
    console.log('Шаг 1: Загрузка данных');
    debugger;
    
    const data1 = await fetchData1();
    console.log('Данные 1 получены:', data1);
    debugger;
    
    console.log('Шаг 2: Обработка данных');
    const processedData = await processData(data1);
    console.log('Данные обработаны:', processedData);
    debugger;
    
    console.log('Шаг 3: Сохранение результатов');
    const result = await saveData(processedData);
    console.log('Результаты сохранены:', result);
    debugger;
    
    return result;
  } catch (error) {
    console.error('Ошибка в асинхронной операции:', error);
    debugger;
    
    // Повторная попытка
    console.log('Повторная попытка...');
    return await retryOperation();
  }
}
```

### 2. Отладка событий

Отладка событий помогает понять, как пользователь взаимодействует с приложением.

```javascript
// Мониторинг всех событий
class EventDebugger {
  constructor() {
    this.eventLog = [];
    this.maxLogSize = 100;
  }
  
  // Логирование событий
  logEvent(event) {
    const eventInfo = {
      type: event.type,
      target: event.target.tagName,
      timestamp: new Date().toISOString(),
      details: {
        clientX: event.clientX,
        clientY: event.clientY,
        keyCode: event.keyCode,
        key: event.key
      }
    };
    
    this.eventLog.push(eventInfo);
    
    // Ограничение размера лога
    if (this.eventLog.length > this.maxLogSize) {
      this.eventLog.shift();
    }
    
    console.log('Событие:', eventInfo);
  }
  
  // Начало мониторинга событий
  startMonitoring() {
    // Мониторинг всех событий (осторожно - может быть много!)
    document.addEventListener('click', (e) => this.logEvent(e), true);
    document.addEventListener('keydown', (e) => this.logEvent(e), true);
    document.addEventListener('keyup', (e) => this.logEvent(e), true);
    document.addEventListener('focus', (e) => this.logEvent(e), true);
    document.addEventListener('blur', (e) => this.logEvent(e), true);
  }
  
  // Мониторинг конкретных событий
  monitorElement(element, eventTypes) {
    eventTypes.forEach(eventType => {
      element.addEventListener(eventType, (e) => {
        console.log(`Событие ${eventType} на элементе:`, e.target);
        debugger; // Точка останова при событии
      });
    });
  }
}

// Использование
const eventDebugger = new EventDebugger();
// eventDebugger.startMonitoring(); // Раскомментируйте для мониторинга

// Мониторинг конкретных событий
const button = document.querySelector('#submit-button');
if (button) {
  eventDebugger.monitorElement(button, ['click', 'focus', 'blur']);
}

// Отладка обработчиков событий
function debugEventHandler(event) {
  console.group(`Обработка события: ${event.type}`);
  console.log('Цель события:', event.target);
  console.log('Текущая цель:', event.currentTarget);
  console.log('Фаза события:', event.eventPhase);
  console.log('Свойства события:', event);
  console.groupEnd();
  
  debugger; // Точка останова для анализа
}

// Добавление отладочного обработчика
document.addEventListener('click', debugEventHandler);
```

### 3. Профилирование производительности

Профилирование помогает выявить узкие места в производительности приложения.

```javascript
// Простое профилирование функций
class PerformanceProfiler {
  constructor() {
    this.measurements = new Map();
  }
  
  // Измерение времени выполнения функции
  timeFunction(fn, functionName) {
    return function(...args) {
      const start = performance.now();
      const result = fn.apply(this, args);
      const end = performance.now();
      const duration = end - start;
      
      console.log(`${functionName} выполнена за ${duration.toFixed(2)}ms`);
      
      // Сохранение измерений
      if (!this.measurements.has(functionName)) {
        this.measurements.set(functionName, []);
      }
      this.measurements.get(functionName).push(duration);
      
      return result;
    }.bind(this);
  }
  
  // Получение статистики по функции
  getFunctionStats(functionName) {
    const measurements = this.measurements.get(functionName);
    if (!measurements || measurements.length === 0) {
      return null;
    }
    
    const sum = measurements.reduce((a, b) => a + b, 0);
    const avg = sum / measurements.length;
    const min = Math.min(...measurements);
    const max = Math.max(...measurements);
    
    return {
      count: measurements.length,
      average: avg.toFixed(2),
      min: min.toFixed(2),
      max: max.toFixed(2),
      total: sum.toFixed(2)
    };
  }
  
  // Профилирование с использованием Performance API
  profileBlock(blockName, callback) {
    performance.mark(`${blockName}-start`);
    
    const result = callback();
    
    performance.mark(`${blockName}-end`);
    performance.measure(blockName, `${blockName}-start`, `${blockName}-end`);
    
    const measures = performance.getEntriesByName(blockName);
    if (measures.length > 0) {
      console.log(`${blockName}: ${measures[0].duration.toFixed(2)}ms`);
    }
    
    // Очистка меток
    performance.clearMarks(`${blockName}-start`);
    performance.clearMarks(`${blockName}-end`);
    performance.clearMeasures(blockName);
    
    return result;
  }
}

// Использование профайлера
const profiler = new PerformanceProfiler();

// Пример функции для профилирования
function slowFunction() {
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}

// Обернутая функция с профилированием
const profiledSlowFunction = profiler.timeFunction(slowFunction, 'slowFunction');

// Вызов функции
profiledSlowFunction();
profiledSlowFunction();
profiledSlowFunction();

// Получение статистики
console.log('Статистика slowFunction:', profiler.getFunctionStats('slowFunction'));

// Профилирование блока кода
profiler.profileBlock('dataProcessing', () => {
  const data = Array.from({ length: 1000 }, (_, i) => i);
  return data.map(x => x * 2).filter(x => x > 500);
});
```

### 4. Отладка памяти

Отладка памяти помогает выявить утечки памяти и оптимизировать использование ресурсов.

```javascript
// Мониторинг использования памяти
class MemoryDebugger {
  // Получение информации о памяти (если доступно)
  static getMemoryInfo() {
    if (performance.memory) {
      return {
        used: Math.round(performance.memory.usedJSHeapSize / 1048576 * 100) / 100,
        total: Math.round(performance.memory.totalJSHeapSize / 1048576 * 100) / 100,
        limit: Math.round(performance.memory.jsHeapSizeLimit / 1048576 * 100) / 100
      };
    }
    return null;
  }
  
  // Логирование использования памяти
  static logMemoryUsage(label = 'Memory usage') {
    const memoryInfo = this.getMemoryInfo();
    if (memoryInfo) {
      console.log(`${label}:`, memoryInfo);
    } else {
      console.log(`${label}: Memory API not available`);
    }
  }
  
  // Мониторинг утечек памяти
  static monitorLeaks() {
    // Проверка на утечки каждые 5 секунд
    setInterval(() => {
      this.logMemoryUsage('Memory leak check');
      
      // Проверка на рост использования памяти
      if (this.lastMemoryUsage) {
        const currentUsage = performance.memory?.usedJSHeapSize;
        if (currentUsage && currentUsage > this.lastMemoryUsage * 1.1) {
          console.warn('Возможная утечка памяти обнаружена!');
          debugger;
        }
      }
      
      this.lastMemoryUsage = performance.memory?.usedJSHeapSize;
    }, 5000);
  }
  
  // Создание объектов для тестирования утечек
  static createTestObjects(count) {
    const objects = [];
    for (let i = 0; i < count; i++) {
      objects.push({
        id: i,
        data: new Array(1000).fill('test'),
        timestamp: Date.now()
      });
    }
    return objects;
  }
}

// Использование
MemoryDebugger.logMemoryUsage('Начало');

// Создание объектов для тестирования
let testObjects = MemoryDebugger.createTestObjects(1000);
MemoryDebugger.logMemoryUsage('После создания объектов');

// Очистка ссылок
testObjects = null;
// Принудительная сборка мусора (в реальном приложении не используйте)
if (window.gc) {
  window.gc();
}
MemoryDebugger.logMemoryUsage('После очистки объектов');

// Мониторинг утечек (раскомментируйте для использования)
// MemoryDebugger.monitorLeaks();
```

## Инструменты браузера для отладки

### 1. Панель Sources

Панель Sources в инструментах разработчика позволяет:
- Устанавливать точки останова
- Просматривать и редактировать исходный код
- Отслеживать стек вызовов
- Просматривать области видимости переменных

```javascript
// Пример кода для отладки в Sources
function complexCalculation(data) {
  let result = 0;
  
  // Точка останова для анализа входных данных
  debugger;
  
  for (let i = 0; i < data.length; i++) {
    // Точка останова внутри цикла
    if (i % 100 === 0) {
      debugger;
    }
    
    const item = data[i];
    result += processItem(item);
  }
  
  return result;
}

function processItem(item) {
  // Точка останова для анализа обработки элемента
  debugger;
  
  if (item.value > 100) {
    return item.value * 2;
  } else {
    return item.value;
  }
}

// Вызов функции для отладки
const testData = Array.from({ length: 1000 }, (_, i) => ({ value: i }));
const result = complexCalculation(testData);
console.log('Результат:', result);
```

### 2. Панель Network

Панель Network помогает отладить сетевые запросы и проблемы с загрузкой ресурсов.

```javascript
// Отладка сетевых запросов
class NetworkDebugger {
  // Мониторинг fetch запросов
  static monitorFetch() {
    const originalFetch = window.fetch;
    
    window.fetch = function(...args) {
      console.log('Fetch request:', args);
      debugger;
      
      return originalFetch.apply(this, args)
        .then(response => {
          console.log('Fetch response:', response);
          debugger;
          return response;
        })
        .catch(error => {
          console.error('Fetch error:', error);
          debugger;
          throw error;
        });
    };
  }
  
  // Мониторинг XMLHttpRequest
  static monitorXHR() {
    const originalOpen = XMLHttpRequest.prototype.open;
    const originalSend = XMLHttpRequest.prototype.send;
    
    XMLHttpRequest.prototype.open = function(method, url, ...args) {
      this._method = method;
      this._url = url;
      console.log('XHR open:', method, url);
      debugger;
      return originalOpen.apply(this, [method, url, ...args]);
    };
    
    XMLHttpRequest.prototype.send = function(body) {
      console.log('XHR send:', this._method, this._url, body);
      debugger;
      
      this.addEventListener('load', function() {
        console.log('XHR load:', this.status, this.responseText);
        debugger;
      });
      
      this.addEventListener('error', function() {
        console.error('XHR error:', this._method, this._url);
        debugger;
      });
      
      return originalSend.apply(this, [body]);
    };
  }
  
  // Симуляция медленного запроса
  static simulateSlowRequest(url, delay = 2000) {
    return new Promise((resolve) => {
      setTimeout(() => {
        fetch(url)
          .then(resolve)
          .catch(resolve); // Игнорируем ошибки для симуляции
      }, delay);
    });
  }
}

// Использование
// NetworkDebugger.monitorFetch(); // Раскомментируйте для мониторинга fetch
// NetworkDebugger.monitorXHR();   // Раскомментируйте для мониторинга XHR

// Тестовый запрос
fetch('/api/test')
  .then(response => response.json())
  .then(data => console.log('Данные получены:', data))
  .catch(error => console.error('Ошибка запроса:', error));
```

### 3. Панель Performance

Панель Performance позволяет анализировать производительность приложения и выявлять узкие места.

```javascript
// Маркировка производительности для анализа в Performance панели
class PerformanceMarker {
  // Начало измерения
  static start(label) {
    performance.mark(`${label}-start`);
  }
  
  // Конец измерения
  static end(label) {
    performance.mark(`${label}-end`);
    performance.measure(label, `${label}-start`, `${label}-end`);
  }
  
  // Измерение функции
  static measureFunction(fn, label) {
    return function(...args) {
      PerformanceMarker.start(label);
      const result = fn.apply(this, args);
      PerformanceMarker.end(label);
      return result;
    };
  }
  
  // Асинхронное измерение
  static async measureAsyncFunction(asyncFn, label) {
    return async function(...args) {
      PerformanceMarker.start(label);
      const result = await asyncFn.apply(this, args);
      PerformanceMarker.end(label);
      return result;
    };
  }
}

// Пример использования маркировки
function expensiveOperation() {
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += Math.sin(i);
  }
  return result;
}

async function asyncOperation() {
  // Симуляция асинхронной операции
  await new Promise(resolve => setTimeout(resolve, 100));
  return 'Operation completed';
}

// Маркированные версии функций
const markedExpensiveOperation = PerformanceMarker.measureFunction(
  expensiveOperation, 
  'expensive-operation'
);

const markedAsyncOperation = PerformanceMarker.measureAsyncFunction(
  asyncOperation, 
  'async-operation'
);

// Вызов функций
markedExpensiveOperation();
markedAsyncOperation().then(result => console.log(result));
```

## Отладка в разных средах

### 1. Отладка Node.js приложений

Для отладки Node.js приложений можно использовать встроенные инструменты:

```javascript
// Отладка Node.js приложений
// Запуск с инспектором: node --inspect app.js

// Использование debugger в Node.js
function nodeDebugExample() {
  const data = { message: 'Hello from Node.js' };
  
  debugger; // Точка останова для инспектора
  
  console.log(data);
  
  return data;
}

// Использование встроенных отладочных утилит
const util = require('util');

// Глубокое инспектирование объектов
const complexObject = {
  name: 'Test',
  nested: {
    value: 42,
    array: [1, 2, 3, { deep: 'value' }]
  }
};

console.log(util.inspect(complexObject, { depth: null, colors: true }));

// Отладка событий в Node.js
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

// Мониторинг всех событий
myEmitter.on('newListener', (event, listener) => {
  console.log('Добавлен новый слушатель для события:', event);
  debugger;
});

myEmitter.on('data', (data) => {
  console.log('Получены данные:', data);
  debugger;
});

// Вызов события
myEmitter.emit('data', { id: 1, value: 'test' });
```

### 2. Отладка в мобильных браузерах

Для отладки в мобильных браузерах можно использовать удаленную отладку:

```javascript
// Отладка в мобильных браузерах
class MobileDebugger {
  // Проверка мобильного устройства
  static isMobile() {
    return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
  }
  
  // Расширенное логирование для мобильных устройств
  static mobileLog(...args) {
    if (this.isMobile()) {
      // Добавление временных меток для мобильной отладки
      const timestamp = new Date().toISOString();
      console.log(`[MOBILE] ${timestamp}`, ...args);
      
      // Сохранение логов в массив для последующего анализа
      if (!window.mobileLogs) {
        window.mobileLogs = [];
      }
      window.mobileLogs.push({
        timestamp,
        args: args.map(arg => 
          typeof arg === 'object' ? JSON.stringify(arg) : arg
        )
      });
    } else {
      console.log(...args);
    }
  }
  
  // Отображение логов на экране для мобильных устройств
  static showMobileLogs() {
    if (!this.isMobile() || !window.mobileLogs) return;
    
    const logContainer = document.createElement('div');
    logContainer.style.cssText = `
      position: fixed;
      top: 0;
      right: 0;
      width: 300px;
      height: 200px;
      background: rgba(0,0,0,0.8);
      color: white;
      font-size: 12px;
      overflow-y: auto;
      z-index: 9999;
      padding: 10px;
    `;
    
    const logs = window.mobileLogs.slice(-20); // Последние 20 записей
    logContainer.innerHTML = logs
      .map(log => `${log.timestamp}: ${log.args.join(' ')}`)
      .join('<br>');
    
    document.body.appendChild(logContainer);
  }
}

// Использование
MobileDebugger.mobileLog('Тестовое сообщение для мобильного устройства');
MobileDebugger.mobileLog('Объект:', { test: 'value' });

// Для отображения логов на мобильном устройстве:
// MobileDebugger.showMobileLogs();
```

## Лучшие практики отладки

### 1. Систематический подход к отладке

```javascript
// Систематический подход к отладке
class SystematicDebugger {
  // Метод деления пополам для поиска ошибок
  static binarySearchDebug(data, testFunction) {
    console.log('Начало бинарного поиска ошибки');
    
    let left = 0;
    let right = data.length - 1;
    
    while (left <= right) {
      const mid = Math.floor((left + right) / 2);
      console.log(`Проверка элемента ${mid}`);
      debugger;
      
      try {
        const result = testFunction(data[mid]);
        console.log(`Элемент ${mid} прошел тест:`, result);
        
        // Если элемент прошел тест, ищем в правой половине
        left = mid + 1;
      } catch (error) {
        console.error(`Элемент ${mid} не прошел тест:`, error);
        debugger;
        
        // Если элемент не прошел тест, ищем в левой половине
        right = mid - 1;
      }
    }
    
    console.log('Поиск завершен. Первая ошибка находится после элемента:', right);
  }
  
  // Метод исключения для поиска ошибок
  static eliminationDebug(components, testFunction) {
    console.log('Начало отладки методом исключения');
    
    // Тестирование всех компонентов
    const results = components.map((component, index) => {
      console.log(`Тестирование компонента ${index}:`, component);
      debugger;
      
      try {
        const result = testFunction(component);
        console.log(`Компонент ${index} работает корректно`);
        return { index, component, success: true, result };
      } catch (error) {
        console.error(`Компонент ${index} вызвал ошибку:`, error);
        debugger;
        return { index, component, success: false, error };
      }
    });
    
    // Анализ результатов
    const failedComponents = results.filter(r => !r.success);
    const successfulComponents = results.filter(r => r.success);
    
    console.log('Рабочие компоненты:', successfulComponents.length);
    console.log('Сломанные компоненты:', failedComponents.length);
    
    if (failedComponents.length > 0) {
      console.warn('Проблемные компоненты:', failedComponents);
      debugger;
    }
    
    return { successfulComponents, failedComponents };
  }
  
  // Создание минимального воспроизводимого примера
  static createMinimalExample(problemFunction, testData) {
    console.log('Создание минимального воспроизводимого примера');
    debugger;
    
    // Начинаем с минимальных данных
    let minimalData = [];
    
    // Постепенно увеличиваем данные до воспроизведения ошибки
    for (let i = 0; i <= testData.length; i++) {
      minimalData = testData.slice(0, i);
      
      try {
        console.log(`Тестирование с ${i} элементами`);
        problemFunction(minimalData);
        console.log(`Тест с ${i} элементами прошел успешно`);
      } catch (error) {
        console.error(`Ошибка воспроизведена с ${i} элементами:`, error);
        debugger;
        break;
      }
    }
    
    console.log('Минимальный набор данных для воспроизведения ошибки:', minimalData);
    return minimalData;
  }
}

// Пример использования систематической отладки
function problematicFunction(data) {
  if (data.length > 5) {
    throw new Error('Ошибка при обработке более 5 элементов');
  }
  
  return data.map(item => item * 2);
}

const testData = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Поиск ошибки методом деления пополам
SystematicDebugger.binarySearchDebug(testData, (item) => {
  return problematicFunction([item]);
});

// Поиск ошибки методом исключения
SystematicDebugger.eliminationDebug(testData, (item) => {
  return problematicFunction([item]);
});

// Создание минимального примера
SystematicDebugger.createMinimalExample(problematicFunction, testData);
```

### 2. Отладка с использованием логирования

```javascript
// Расширенная система логирования для отладки
class DebugLogger {
  constructor(options = {}) {
    this.level = options.level || 'debug';
    this.prefix = options.prefix || '';
    this.enabled = options.enabled !== false;
    this.logs = [];
  }
  
  // Уровни логирования
  static levels = {
    error: 0,
    warn: 1,
    info: 2,
    debug: 3
  };
  
  // Проверка уровня логирования
  shouldLog(level) {
    return this.enabled && DebugLogger.levels[level] <= DebugLogger.levels[this.level];
  }
  
  // Логирование ошибок
  error(message, ...args) {
    if (this.shouldLog('error')) {
      const logEntry = {
        timestamp: new Date().toISOString(),
        level: 'error',
        message,
        args,
        prefix: this.prefix
      };
      
      this.logs.push(logEntry);
      console.error(`[ERROR] ${this.prefix} ${message}`, ...args);
      debugger;
    }
  }
  
  // Логирование предупреждений
  warn(message, ...args) {
    if (this.shouldLog('warn')) {
      const logEntry = {
        timestamp: new Date().toISOString(),
        level: 'warn',
        message,
        args,
        prefix: this.prefix
      };
      
      this.logs.push(logEntry);
      console.warn(`[WARN] ${this.prefix} ${message}`, ...args);
    }
  }
  
  // Логирование информации
  info(message, ...args) {
    if (this.shouldLog('info')) {
      const logEntry = {
        timestamp: new Date().toISOString(),
        level: 'info',
        message,
        args,
        prefix: this.prefix
      };
      
      this.logs.push(logEntry);
      console.info(`[INFO] ${this.prefix} ${message}`, ...args);
    }
  }
  
  // Логирование отладочной информации
  debug(message, ...args) {
    if (this.shouldLog('debug')) {
      const logEntry = {
        timestamp: new Date().toISOString(),
        level: 'debug',
        message,
        args,
        prefix: this.prefix
      };
      
      this.logs.push(logEntry);
      console.log(`[DEBUG] ${this.prefix} ${message}`, ...args);
    }
  }
  
  // Логирование с контекстом
  withContext(context) {
    return new DebugLogger({
      level: this.level,
      prefix: `${this.prefix}[${context}]`,
      enabled: this.enabled
    });
  }
  
  // Получение логов
  getLogs(level) {
    if (!level) {
      return this.logs;
    }
    return this.logs.filter(log => log.level === level);
  }
  
  // Очистка логов
  clearLogs() {
    this.logs = [];
  }
  
  // Экспорт логов
  exportLogs() {
    return JSON.stringify(this.logs, null, 2);
  }
}

// Использование расширенного логирования
const logger = new DebugLogger({ level: 'debug', prefix: 'App' });

// Логирование с контекстом
const userLogger = logger.withContext('User');
const authLogger = logger.withContext('Auth');

function authenticateUser(username, password) {
  authLogger.debug('Начало аутентификации', { username });
  
  if (!username) {
    authLogger.error('Имя пользователя не указано');
    return false;
  }
  
  if (!password) {
    authLogger.error('Пароль не указан');
    return false;
  }
  
  authLogger.info('Попытка входа', { username });
  
  // Симуляция аутентификации
  if (username === 'admin' && password === 'password') {
    authLogger.info('Аутентификация успешна', { username });
    return true;
  } else {
    authLogger.warn('Неверные учетные данные', { username });
    return false;
  }
}

// Тестирование
authenticateUser('admin', 'password');
authenticateUser('user', 'wrong');
authenticateUser('', 'password');
authenticateUser('user', '');

// Просмотр логов
console.log('Все логи:', logger.getLogs());
console.log('Ошибки:', logger.getLogs('error'));
```

### 3. Отладка с использованием тестов

```javascript
// Отладка с использованием unit тестов
class TestDebugger {
  constructor() {
    this.tests = [];
    this.results = [];
  }
  
  // Добавление теста
  addTest(name, testFunction) {
    this.tests.push({ name, testFunction });
  }
  
  // Выполнение тестов с отладкой
  async runTests(debug = true) {
    console.log(`Запуск ${this.tests.length} тестов`);
    
    for (const [index, test] of this.tests.entries()) {
      console.log(`Тест ${index + 1}/${this.tests.length}: ${test.name}`);
      
      if (debug) {
        debugger;
      }
      
      try {
        const startTime = performance.now();
        const result = await test.testFunction();
        const endTime = performance.now();
        
        const testResult = {
          name: test.name,
          status: 'passed',
          duration: endTime - startTime,
          result
        };
        
        this.results.push(testResult);
        console.log(`✓ Тест пройден: ${test.name} (${testResult.duration.toFixed(2)}ms)`);
        
      } catch (error) {
        const testResult = {
          name: test.name,
          status: 'failed',
          error: error.message,
          stack: error.stack
        };
        
        this.results.push(testResult);
        console.error(`✗ Тест провален: ${test.name}`, error);
        
        if (debug) {
          debugger;
        }
      }
    }
    
    // Сводка результатов
    const passed = this.results.filter(r => r.status === 'passed').length;
    const failed = this.results.filter(r => r.status === 'failed').length;
    
    console.log(`Результаты: ${passed} пройдено, ${failed} провалено`);
    
    if (failed > 0) {
      console.warn('Проваленные тесты:', this.results.filter(r => r.status === 'failed'));
    }
    
    return this.results;
  }
  
  // Создание тестовых данных
  static createTestData(type, options = {}) {
    switch (type) {
      case 'users':
        return Array.from({ length: options.count || 5 }, (_, i) => ({
          id: i + 1,
          name: `User ${i + 1}`,
          email: `user${i + 1}@example.com`,
          active: i % 2 === 0
        }));
      
      case 'products':
        return Array.from({ length: options.count || 5 }, (_, i) => ({
          id: i + 1,
          name: `Product ${i + 1}`,
          price: (i + 1) * 10,
          category: ['Electronics', 'Books', 'Clothing'][i % 3]
        }));
      
      default:
        return [];
    }
  }
}

// Пример использования тестовой отладки
const testDebugger = new TestDebugger();

// Функции для тестирования
function calculateTotal(items) {
  return items.reduce((sum, item) => sum + item.price, 0);
}

function filterActiveUsers(users) {
  return users.filter(user => user.active);
}

function findUserById(users, id) {
  return users.find(user => user.id === id);
}

// Добавление тестов
testDebugger.addTest('Тест расчета общей суммы', async () => {
  const products = TestDebugger.createTestData('products', { count: 3 });
  const total = calculateTotal(products);
  console.assert(total === 60, 'Общая сумма должна быть 60');
  return total;
});

testDebugger.addTest('Тест фильтрации активных пользователей', async () => {
  const users = TestDebugger.createTestData('users', { count: 5 });
  const activeUsers = filterActiveUsers(users);
  console.assert(activeUsers.length === 3, 'Должно быть 3 активных пользователя');
  return activeUsers;
});

testDebugger.addTest('Тест поиска пользователя по ID', async () => {
  const users = TestDebugger.createTestData('users');
  const user = findUserById(users, 3);
  console.assert(user && user.id === 3, 'Должен найти пользователя с ID 3');
  return user;
});

testDebugger.addTest('Тест проваленного сценария', async () => {
  const users = TestDebugger.createTestData('users');
  const user = findUserById(users, 999);
  console.assert(user === undefined, 'Должен вернуть undefined для несуществующего ID');
  // Специально провальный тест для демонстрации
  console.assert(false, 'Специально провальный тест');
  return user;
});

// Запуск тестов с отладкой
// testDebugger.runTests(true);
```

## Заключение

Эффективная отладка - это критически важный навык для любого разработчика JavaScript. Основные принципы успешной отладки:

1. **Используйте правильные инструменты**:
   - Консоль разработчика для быстрой отладки
   - Точки останова для детального анализа
   - Профилировщики для анализа производительности
   - Мониторы сетевых запросов для отладки API

2. **Применяйте систематический подход**:
   - Метод деления пополам для поиска ошибок
   - Метод исключения для изоляции проблем
   - Создание минимальных воспроизводимых примеров

3. **Используйте логирование эффективно**:
   - Различные уровни логирования
   - Контекстуальное логирование
   - Структурированное логирование для анализа

4. **Отлаживайте в разных средах**:
   - Браузерные инструменты разработчика
   - Node.js инспектор
   - Мобильная отладка

5. **Интегрируйте отладку в процесс разработки**:
   - Unit тесты с отладочными точками
   - Автоматизированное логирование
   - Профилирование как часть CI/CD

Помните, что отладка - это не только поиск ошибок, но и понимание того, как работает ваш код. Регулярная практика отладки поможет вам стать более эффективным разработчиком и создавать более качественные приложения.

#javascript #debugging #devtools #performance #testing #frontend-development #web-development
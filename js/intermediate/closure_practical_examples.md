---
tags: [javascript, frontend, intermediate, closures]
aliases: [замыкания в js для фронтенда, практические примеры замыканий]
---

# Замыкания в JavaScript - Практические примеры для Frontend разработки

## Введение

Замыкания - это мощная концепция в JavaScript, которая особенно полезна при разработке фронтенд приложений. В этой статье рассмотрим практические примеры использования замыканий с акцентом на задачи фронтенд разработки.

## Что такое замыкания?

Замыкание - это комбинация функции и лексического окружения, в котором эта функция была объявлена. Это позволяет функции обращаться к переменным из внешней функции даже после того, как внешняя функция завершила выполнение.

## Практический пример: Каррирование для UI компонентов

```javascript
// Функция для создания каррированного обработчика событий
function createEventHandler(componentName) {
  return function (eventName) {
    return function (callback) {
      document.addEventListener(eventName, function (event) {
        console.log(`Компонент ${componentName} обработал событие ${eventName}`);
        callback(event);
      });
    };
  };
}

// Использование
const buttonHandler = createEventHandler('Button');
const clickHandler = buttonHandler('click');
clickHandler(() => {
  console.log('Кнопка была нажата!');
});
```

## Практический пример: Приватные переменные в компонентах

```javascript
// Создание компонента с приватным состоянием
function createCounter(initialValue = 0) {
  let count = initialValue;
  
  return {
    increment: function() {
      count++;
      this.updateDisplay();
    },
    decrement: function() {
      count--;
      this.updateDisplay();
    },
    reset: function() {
      count = initialValue;
      this.updateDisplay();
    },
    getValue: function() {
      return count;
    },
    updateDisplay: function() {
      const displayElement = document.getElementById('counter-display');
      if (displayElement) {
        displayElement.textContent = `Счетчик: ${count}`;
      }
    }
  };
}

// Использование счетчика
const counter = createCounter(10);
const incrementBtn = document.getElementById('increment');
const decrementBtn = document.getElementById('decrement');
const resetBtn = document.getElementById('reset');

if (incrementBtn) {
  incrementBtn.addEventListener('click', counter.increment.bind(counter));
}
if (decrementBtn) {
  decrementBtn.addEventListener('click', counter.decrement.bind(counter));
}
if (resetBtn) {
  resetBtn.addEventListener('click', counter.reset.bind(counter));
}
```

## Практический пример: Модульный паттерн для управления состоянием

```javascript
// Модуль для управления пользовательскими настройками
const userPreferences = (function() {
  // Приватные переменные
  let preferences = {
    theme: 'light',
    language: 'ru',
    notifications: true
  };
  
  // Приватные методы
  function saveToStorage() {
    localStorage.setItem('userPreferences', JSON.stringify(preferences));
  }
  
  // Публичный API
  return {
    get: function(key) {
      return preferences[key];
    },
    
    set: function(key, value) {
      preferences[key] = value;
      saveToStorage();
      
      // Уведомление об изменениях
      const event = new CustomEvent('preferencesChanged', {
        detail: { key, value }
      });
      document.dispatchEvent(event);
    },
    
    getAll: function() {
      return { ...preferences };
    },
    
    reset: function() {
      preferences = {
        theme: 'light',
        language: 'ru',
        notifications: true
      };
      saveToStorage();
    }
  };
})();

// Использование
userPreferences.set('theme', 'dark');
console.log(userPreferences.get('theme')); // 'dark'

// Подписка на изменения
document.addEventListener('preferencesChanged', (e) => {
  console.log(`Настройка ${e.detail.key} изменена на ${e.detail.value}`);
});
```

## Практический пример: Создание оберток с кешированием

```javascript
// Функция для создания кешированного API клиента
function createCachedApiClient(baseURL) {
  const cache = new Map();
  
  return {
    async get(endpoint) {
      const cacheKey = `${baseURL}${endpoint}`;
      
      if (cache.has(cacheKey)) {
        console.log('Данные взяты из кеша');
        return cache.get(cacheKey);
      }
      
      try {
        const response = await fetch(`${baseURL}${endpoint}`);
        const data = await response.json();
        
        // Кеширование на 5 минут
        cache.set(cacheKey, data);
        
        setTimeout(() => {
          cache.delete(cacheKey);
        }, 5 * 60 * 1000); // 5 минут
        
        return data;
      } catch (error) {
        console.error('Ошибка при получении данных:', error);
        throw error;
      }
    },
    
    clearCache() {
      cache.clear();
    }
  };
}

// Использование
const apiClient = createCachedApiClient('https://api.example.com');
```

## Практический пример: Фабрика компонентов

```javascript
// Фабрика для создания компонентов с настраиваемым поведением
function createComponentFactory(defaultOptions) {
  return function createComponent(type, options = {}) {
    const config = { ...defaultOptions, ...options };
    
    switch (type) {
      case 'button':
        return createButton(config);
      case 'input':
        return createInput(config);
      case 'modal':
        return createModal(config);
      default:
        throw new Error(`Неизвестный тип компонента: ${type}`);
    }
  };
}

function createButton({ text, onClick, className = 'btn' }) {
  const button = document.createElement('button');
  button.textContent = text;
  button.className = className;
  button.addEventListener('click', onClick);
  return button;
}

function createInput({ placeholder, onChange, type = 'text' }) {
  const input = document.createElement('input');
  input.type = type;
  input.placeholder = placeholder;
  input.addEventListener('input', onChange);
  return input;
}

function createModal({ title, content, onClose }) {
  const modal = document.createElement('div');
  modal.className = 'modal';
  
  const header = document.createElement('div');
  header.className = 'modal-header';
  header.textContent = title;
  
  const closeBtn = document.createElement('span');
  closeBtn.textContent = '×';
  closeBtn.className = 'modal-close';
  closeBtn.addEventListener('click', onClose);
  
  const body = document.createElement('div');
  body.className = 'modal-body';
  body.innerHTML = content;
  
  header.appendChild(closeBtn);
  modal.appendChild(header);
  modal.appendChild(body);
  
  return modal;
}

// Использование фабрики
const defaultOptions = {
  theme: 'default',
  size: 'medium'
};

const createCustomComponent = createComponentFactory(defaultOptions);

// Создание компонентов
const customButton = createCustomComponent('button', {
  text: 'Нажми меня',
  onClick: () => alert('Кнопка нажата!')
});
```

## Практический пример: Управление асинхронными операциями

```javascript
// Создание обработчика асинхронных операций с отслеживанием состояния
function createAsyncHandler() {
  const pendingOperations = new Map();
  
  return {
    async execute(id, asyncFunction) {
      // Отмена предыдущей операции с тем же ID
      if (pendingOperations.has(id)) {
        pendingOperations.get(id).cancel();
      }
      
      // Создание отменяемой операции
      const controller = new AbortController();
      pendingOperations.set(id, { cancel: () => controller.abort() });
      
      try {
        const result = await asyncFunction(controller.signal);
        pendingOperations.delete(id);
        return result;
      } catch (error) {
        if (error.name === 'AbortError') {
          console.log(`Операция ${id} была отменена`);
        } else {
          throw error;
        }
      } finally {
        pendingOperations.delete(id);
      }
    },
    
    cancel(id) {
      if (pendingOperations.has(id)) {
        pendingOperations.get(id).cancel();
        pendingOperations.delete(id);
      }
    }
  };
}

// Использование
const asyncHandler = createAsyncHandler();

async function searchUsers(query, signal) {
  const response = await fetch(`/api/users?q=${query}`, { signal });
  return await response.json();
}

// Вызов асинхронной операции
asyncHandler.execute('user-search', () => searchUsers('john'));
```

## Лучшие практики при использовании замыканий в фронтенд разработке

### 1. Избегайте утечек памяти

```javascript
// Плохо - возможна утечка памяти
function createComponent() {
  const element = document.getElementById('my-element');
  let largeData = new Array(1000000).fill('data');
  
  // Обработчик держит ссылку на largeData
  element.addEventListener('click', function() {
    console.log('Клик! Данные:', largeData.length);
  });
  
  // largeData не будет освобождена сборщиком мусора
}

// Хорошо - удаляем обработчики событий
function createComponent() {
  const element = document.getElementById('my-element');
  let largeData = new Array(1000000).fill('data');
  
  function handleClick() {
    console.log('Клик! Данные:', largeData.length);
  }
  
  element.addEventListener('click', handleClick);
  
  // Возвращаем функцию для очистки
  return function cleanup() {
    element.removeEventListener('click', handleClick);
    largeData = null; // Удаляем ссылку
  };
}
```

### 2. Используйте замыкания для изоляции состояния

```javascript
// Плохо - глобальное состояние
let currentUserId = null;
function setCurrentUser(id) {
  currentUserId = id;
}

// Хорошо - изолированное состояние через замыкание
function createUserContext() {
  let currentUserId = null;
  
  return {
    setCurrentUser: function(id) {
      currentUserId = id;
    },
    getCurrentUser: function() {
      return currentUserId;
    }
  };
}

const userContext = createUserContext();
userContext.setCurrentUser(123);
```

## Заключение

Замыкания в JavaScript фронтенд разработке позволяют создавать более гибкие, безопасные и переиспользуемые компоненты. Они особенно полезны для инкапсуляции состояния, создания приватных переменных и управления асинхронными операциями.

## См. также

- [[functions_practical_examples.md]]
- [[Прототипное наследование в JavaScript]]
- [[Метапрограммирование и прокси в JavaScript]]
- [[Функциональное программирование в JS]]
- [[DOM манипуляции в JavaScript]]
---
aliases: ["Универсальные функции", "JS Utils", "Universal Functions"]
tags: [js, utilities, functions, helpers]
---

# Универсальные функции

## Введение

Универсальные функции - это многократно используемые функции, которые решают типичные задачи в JavaScript. Эти функции можно использовать в различных проектах и контекстах, что повышает продуктивность и снижает количество дублирующегося кода.

## Функции для работы с массивами

### 1. Основные операции с массивами

```javascript
/**
 * Возвращает уникальные значения из массива
 * @param {Array} arr - Входной массив
 * @returns {Array} Массив с уникальными значениями
 */
const getUnique = (arr) => [...new Set(arr)];

/**
 * Группировка массива по значению свойства
 * @param {Array} array - Массив объектов
 * @param {string} key - Ключ для группировки
 * @returns {Object} Объект с группами
 */
const groupBy = (array, key) => {
  return array.reduce((result, item) => {
    const group = item[key];
    if (!result[group]) {
      result[group] = [];
    }
    result[group].push(item);
    return result;
  }, {});
};

/**
 * Разделение массива на чанки
 * @param {Array} array - Входной массив
 * @param {number} size - Размер чанка
 * @returns {Array} Массив чанков
 */
const chunk = (array, size) => {
  return array.reduce((chunks, item, index) => {
    const chunkIndex = Math.floor(index / size);
    if (!chunks[chunkIndex]) {
      chunks[chunkIndex] = [];
    }
    chunks[chunkIndex].push(item);
    return chunks;
  }, []);
};

/**
 * Перемешивание массива (алгоритм Фишера-Йетса)
 * @param {Array} array - Массив для перемешивания
 * @returns {Array} Новый перемешанный массив
 */
const shuffle = (array) => {
  const result = [...array];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
};

/**
 * Удаление дубликатов из массива объектов по ключу
 * @param {Array} array - Массив объектов
 * @param {string} key - Ключ для проверки уникальности
 * @returns {Array} Массив без дубликатов
 */
const uniqueBy = (array, key) => {
  const seen = new Set();
  return array.filter(item => {
    const value = item[key];
    if (seen.has(value)) {
      return false;
    }
    seen.add(value);
    return true;
  });
};

/**
 * Плоское объединение вложенных массивов
 * @param {Array} array - Массив массивов
 * @param {number} depth - Глубина объединения
 * @returns {Array} Плоский массив
 */
const flat = (array, depth = 1) => {
  return array.reduce((acc, val) => 
    Array.isArray(val) && depth > 0
      ? acc.concat(flat(val, depth - 1))
      : acc.concat(val), 
  []);
};
```

### 2. Поиск и фильтрация в массивах

```javascript
/**
 * Поиск элемента в массиве по нескольким критериям
 * @param {Array} array - Массив для поиска
 * @param {Object} searchParams - Объект с критериями поиска
 * @returns {Object|undefined} Найденный элемент или undefined
 */
const findInArray = (array, searchParams) => {
  return array.find(item => 
    Object.entries(searchParams).every(([key, value]) => item[key] === value)
  );
};

/**
 * Фильтрация массива по нескольким критериям
 * @param {Array} array - Массив для фильтрации
 * @param {Object} filterParams - Объект с критериями фильтрации
 * @returns {Array} Отфильтрованный массив
 */
const filterBy = (array, filterParams) => {
  return array.filter(item => 
    Object.entries(filterParams).every(([key, value]) => {
      if (typeof value === 'function') {
        return value(item[key]);
      }
      return item[key] === value;
    })
  );
};

/**
 * Сортировка массива по нескольким полям
 * @param {Array} array - Массив для сортировки
 * @param {Array} sortFields - Массив объектов {field: 'name', direction: 'asc|desc'}
 * @returns {Array} Отсортированный массив
 */
const multiSort = (array, sortFields) => {
  return [...array].sort((a, b) => {
    for (const { field, direction = 'asc' } of sortFields) {
      const aVal = a[field];
      const bVal = b[field];
      
      let comparison = 0;
      if (aVal > bVal) comparison = 1;
      else if (aVal < bVal) comparison = -1;
      
      if (comparison !== 0) {
        return direction === 'desc' ? -comparison : comparison;
      }
    }
    return 0;
  });
};

/**
 * Преобразование массива в объект по ключу
 * @param {Array} array - Массив объектов
 * @param {string} key - Ключ для использования в качестве ключа объекта
 * @returns {Object} Объект с элементами массива в качестве значений
 */
const arrayToObject = (array, key) => {
  return array.reduce((obj, item) => {
    obj[item[key]] = item;
    return obj;
  }, {});
};
```

## Функции для работы с объектами

### 3. Основные операции с объектами

```javascript
/**
 * Глубокое объединение объектов
 * @param {...Object} objects - Объекты для объединения
 * @returns {Object} Объединенный объект
 */
const deepMerge = (...objects) => {
  return objects.reduce((acc, obj) => {
    Object.keys(obj).forEach(key => {
      if (obj[key] && typeof obj[key] === 'object' && !Array.isArray(obj[key])) {
        acc[key] = deepMerge(acc[key] || {}, obj[key]);
      } else {
        acc[key] = obj[key];
      }
    });
    return acc;
  }, {});
};

/**
 * Выбор подмножества свойств объекта
 * @param {Object} obj - Исходный объект
 * @param {Array} keys - Массив ключей для выбора
 * @returns {Object} Новый объект с выбранными свойствами
 */
const pick = (obj, keys) => {
  return keys.reduce((result, key) => {
    if (key in obj) {
      result[key] = obj[key];
    }
    return result;
  }, {});
};

/**
 * Исключение свойств из объекта
 * @param {Object} obj - Исходный объект
 * @param {Array} keys - Массив ключей для исключения
 * @returns {Object} Новый объект без указанных свойств
 */
const omit = (obj, keys) => {
  const result = { ...obj };
  keys.forEach(key => delete result[key]);
  return result;
};

/**
 * Глубокое копирование объекта
 * @param {Object} obj - Объект для копирования
 * @returns {Object} Глубокая копия объекта
 */
const deepClone = (obj) => {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(item => deepClone(item));
  if (obj instanceof Object) {
    const cloned = {};
    for (let key in obj) {
      if (obj.hasOwnProperty(key)) {
        cloned[key] = deepClone(obj[key]);
      }
    }
    return cloned;
  }
};

/**
 * Преобразование ключей объекта
 * @param {Object} obj - Исходный объект
 * @param {Function} mapper - Функция для преобразования ключей
 * @returns {Object} Новый объект с преобразованными ключами
 */
const mapKeys = (obj, mapper) => {
  return Object.keys(obj).reduce((result, key) => {
    const newKey = mapper(key, obj[key]);
    result[newKey] = obj[key];
    return result;
  }, {});
};

/**
 * Преобразование значений объекта
 * @param {Object} obj - Исходный объект
 * @param {Function} mapper - Функция для преобразования значений
 * @returns {Object} Новый объект с преобразованными значениями
 */
const mapValues = (obj, mapper) => {
  return Object.keys(obj).reduce((result, key) => {
    result[key] = mapper(obj[key], key);
    return result;
  }, {});
};
```

### 4. Поиск и манипуляции с объектами

```javascript
/**
 * Получение значения из объекта по пути (dot notation)
 * @param {Object} obj - Объект для поиска
 * @param {string} path - Путь к значению (например, 'user.profile.name')
 * @param {*} defaultValue - Значение по умолчанию
 * @returns {*} Найденное значение или значение по умолчанию
 */
const get = (obj, path, defaultValue = undefined) => {
  const keys = path.split('.');
  let result = obj;
  
  for (const key of keys) {
    if (result == null || typeof result !== 'object') {
      return defaultValue;
    }
    result = result[key];
  }
  
  return result !== undefined ? result : defaultValue;
};

/**
 * Установка значения в объекте по пути (dot notation)
 * @param {Object} obj - Объект для изменения
 * @param {string} path - Путь к значению
 * @param {*} value - Значение для установки
 * @returns {Object} Модифицированный объект
 */
const set = (obj, path, value) => {
  const keys = path.split('.');
  let current = obj;
  
  for (let i = 0; i < keys.length - 1; i++) {
    const key = keys[i];
    if (!(key in current) || typeof current[key] !== 'object') {
      current[key] = {};
    }
    current = current[key];
  }
  
  current[keys[keys.length - 1]] = value;
  return obj;
};

/**
 * Проверка, является ли объект пустым
 * @param {Object} obj - Объект для проверки
 * @returns {boolean} true, если объект пустой
 */
const isEmptyObject = (obj) => {
  return obj && Object.keys(obj).length === 0 && obj.constructor === Object;
};

/**
 * Преобразование объекта в URL параметры
 * @param {Object} obj - Объект для преобразования
 * @returns {string} Строка URL параметров
 */
const objectToParams = (obj) => {
  return Object.keys(obj)
    .map(key => `${encodeURIComponent(key)}=${encodeURIComponent(obj[key])}`)
    .join('&');
};
```

## Функции для работы с типами и валидацией

### 5. Валидация данных

```javascript
/**
 * Проверка, является ли значение массивом
 * @param {*} value - Значение для проверки
 * @returns {boolean} true, если значение - массив
 */
const isArray = (value) => Array.isArray(value);

/**
 * Проверка, является ли значение простым объектом
 * @param {*} value - Значение для проверки
 * @returns {boolean} true, если значение - простой объект
 */
const isPlainObject = (value) => {
  return value !== null && 
         typeof value === 'object' && 
         value.constructor === Object &&
         Object.getPrototypeOf(value) === Object.prototype;
};

/**
 * Валидация email
 * @param {string} email - Email для валидации
 * @returns {boolean} true, если email валиден
 */
const isValidEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

/**
 * Валидация телефона
 * @param {string} phone - Телефон для валидации
 * @returns {boolean} true, если телефон валиден
 */
const isValidPhone = (phone) => {
  const phoneRegex = /^(\+7|7|8)?[\s-]?\(?[0-9]{3}\)?[\s-]?[0-9]{3}[\s-]?[0-9]{2}[\s-]?[0-9]{2}$/;
  return phoneRegex.test(phone);
};

/**
 * Валидация URL
 * @param {string} url - URL для валидации
 * @returns {boolean} true, если URL валиден
 */
const isValidUrl = (url) => {
  try {
    new URL(url);
    return true;
  } catch {
    return false;
  }
};

/**
 * Проверка на пустое значение
 * @param {*} value - Значение для проверки
 * @returns {boolean} true, если значение пустое
 */
const isEmpty = (value) => {
  if (Array.isArray(value)) return value.length === 0;
  if (isPlainObject(value)) return Object.keys(value).length === 0;
  if (typeof value === 'string') return value.trim() === '';
  return value == null;
};

/**
 * Валидация сложных данных по схеме
 * @param {Object} data - Данные для валидации
 * @param {Object} schema - Схема валидации
 * @returns {Object} Результат валидации
 */
const validateSchema = (data, schema) => {
  const errors = {};
  
  for (const [field, rules] of Object.entries(schema)) {
    const value = data[field];
    
    if (rules.required && (value === undefined || value === null)) {
      errors[field] = 'Обязательное поле';
      continue;
    }
    
    if (value !== undefined && value !== null) {
      if (rules.type && typeof value !== rules.type) {
        errors[field] = `Должно быть типа ${rules.type}`;
      }
      
      if (rules.validator && !rules.validator(value)) {
        errors[field] = rules.message || 'Невалидное значение';
      }
    }
  }
  
  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
};
```

## Функции для работы с DOM

### 6. Универсальные DOM функции

```javascript
/**
 * Создание DOM элемента с атрибутами
 * @param {string} tag - Тег элемента
 * @param {Object} props - Атрибуты и свойства элемента
 * @param {Array} children - Дочерние элементы
 * @returns {HTMLElement} Созданный элемент
 */
const createElement = (tag, props = {}, children = []) => {
  const element = document.createElement(tag);
  
  Object.entries(props).forEach(([key, value]) => {
    if (key.startsWith('on')) {
      element.addEventListener(key.slice(2).toLowerCase(), value);
    } else if (key === 'className') {
      element.className = value;
    } else if (key === 'style' && typeof value === 'object') {
      Object.assign(element.style, value);
    } else if (key === 'dataset') {
      Object.assign(element.dataset, value);
    } else {
      element.setAttribute(key, value);
    }
  });
  
  children.forEach(child => {
    element.appendChild(
      typeof child === 'string' ? document.createTextNode(child) : child
    );
  });
  
  return element;
};

/**
 * Поиск элемента с выбрасыванием ошибки при отсутствии
 * @param {string} selector - CSS селектор
 * @param {HTMLElement} parent - Родительский элемент (по умолчанию document)
 * @returns {HTMLElement} Найденный элемент
 */
const findElement = (selector, parent = document) => {
  const element = parent.querySelector(selector);
  if (!element) {
    throw new Error(`Элемент не найден: ${selector}`);
  }
  return element;
};

/**
 * Проверка, находится ли элемент в области видимости
 * @param {HTMLElement} element - Элемент для проверки
 * @returns {boolean} true, если элемент видим
 */
const isElementVisible = (element) => {
  const rect = element.getBoundingClientRect();
  return (
    rect.top >= 0 &&
    rect.left >= 0 &&
    rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
    rect.right <= (window.innerWidth || document.documentElement.clientWidth)
  );
};

/**
 * Анимированное прокручивание к элементу
 * @param {HTMLElement} element - Элемент для прокрутки
 * @param {Object} options - Опции прокрутки
 */
const smoothScrollTo = (element, options = {}) => {
  const {
    behavior = 'smooth',
    block = 'start',
    inline = 'nearest'
  } = options;
  
  element.scrollIntoView({ behavior, block, inline });
};

/**
 * Добавление класса с задержкой
 * @param {HTMLElement} element - Элемент для изменения
 * @param {string} className - Имя класса
 * @param {number} delay - Задержка в миллисекундах
 */
const addClassWithDelay = (element, className, delay = 0) => {
  setTimeout(() => {
    element.classList.add(className);
  }, delay);
};
```

## Функции для работы с асинхронными операциями

### 7. Утилиты для Promise

```javascript
/**
 * Ожидание указанного времени
 * @param {number} ms - Время ожидания в миллисекундах
 * @returns {Promise} Promise, который разрешится через указанное время
 */
const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

/**
 * Выполнение асинхронной операции с таймаутом
 * @param {Promise} promise - Асинхронная операция
 * @param {number} timeoutMs - Время таймаута в миллисекундах
 * @returns {Promise} Promise с таймаутом
 */
const withTimeout = (promise, timeoutMs) => {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Таймаут операции')), timeoutMs)
    )
  ]);
};

/**
 * Повторение асинхронной операции с экспоненциальной задержкой
 * @param {Function} asyncFn - Асинхронная функция для повторения
 * @param {number} maxRetries - Максимальное количество попыток
 * @param {number} baseDelay - Базовая задержка в миллисекундах
 * @returns {Promise} Результат выполнения функции
 */
const retryAsync = async (asyncFn, maxRetries = 3, baseDelay = 1000) => {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await asyncFn();
    } catch (error) {
      lastError = error;
      
      if (attempt === maxRetries) break;
      
      const delay = baseDelay * Math.pow(2, attempt - 1);
      await wait(delay);
    }
  }
  
  throw lastError;
};

/**
 * Ограничение количества одновременных асинхронных операций
 * @param {number} poolLimit - Максимальное количество одновременных операций
 * @param {Array} array - Массив данных для обработки
 * @param {Function} iteratorFn - Функция для обработки каждого элемента
 * @returns {Promise} Promise с результатами
 */
const asyncPool = async (poolLimit, array, iteratorFn) => {
  const ret = [];
  const executing = [];
  
  for (const item of array) {
    const p = Promise.resolve().then(() => iteratorFn(item, array));
    ret.push(p);

    if (ret.length >= poolLimit) {
      const e = p.then(() => executing.splice(executing.indexOf(e), 1));
      executing.push(e);
      if (executing.length >= poolLimit) {
        await Promise.race(executing);
      }
    }
  }
  
  return Promise.all(ret);
};

/**
 * Выполнение асинхронных операций последовательно
 * @param {Array} operations - Массив асинхронных функций
 * @returns {Promise} Promise с массивом результатов
 */
const runSequentially = async (operations) => {
  const results = [];
  
  for (const operation of operations) {
    try {
      const result = await operation();
      results.push({ success: true, data: result });
    } catch (error) {
      results.push({ success: false, error });
    }
  }
  
  return results;
};
```

## Функции для работы с датами

### 8. Утилиты для работы с датами

```javascript
/**
 * Форматирование даты
 * @param {Date|string} date - Дата для форматирования
 * @param {string} format - Формат (DD.MM.YYYY, YYYY-MM-DD и т.д.)
 * @returns {string} Отформатированная дата
 */
const formatDate = (date, format = 'DD.MM.YYYY') => {
  const d = new Date(date);
  const day = String(d.getDate()).padStart(2, '0');
  const month = String(d.getMonth() + 1).padStart(2, '0');
  const year = d.getFullYear();
  
  return format
    .replace('DD', day)
    .replace('MM', month)
    .replace('YYYY', year)
    .replace('HH', String(d.getHours()).padStart(2, '0'))
    .replace('mm', String(d.getMinutes()).padStart(2, '0'))
    .replace('ss', String(d.getSeconds()).padStart(2, '0'));
};

/**
 * Получение разницы между датами
 * @param {Date|string} date1 - Первая дата
 * @param {Date|string} date2 - Вторая дата
 * @returns {Object} Объект с разницей в различных единицах
 */
const dateDiff = (date1, date2) => {
  const d1 = new Date(date1);
  const d2 = new Date(date2);
  const diffTime = Math.abs(d2 - d1);
  
  return {
    milliseconds: diffTime,
    seconds: Math.floor(diffTime / 1000),
    minutes: Math.floor(diffTime / (1000 * 60)),
    hours: Math.floor(diffTime / (1000 * 60 * 60)),
    days: Math.floor(diffTime / (1000 * 60 * 60 * 24)),
    weeks: Math.floor(diffTime / (1000 * 60 * 60 * 24 * 7)),
    months: Math.floor(diffTime / (1000 * 60 * 60 * 24 * 30.44)),
    years: Math.floor(diffTime / (1000 * 60 * 60 * 24 * 365.25))
  };
};

/**
 * Проверка, является ли дата сегодняшней
 * @param {Date|string} date - Дата для проверки
 * @returns {boolean} true, если дата - сегодня
 */
const isToday = (date) => {
  const today = new Date();
  const checkDate = new Date(date);
  
  return checkDate.getDate() === today.getDate() &&
         checkDate.getMonth() === today.getMonth() &&
         checkDate.getFullYear() === today.getFullYear();
};

/**
 * Проверка, является ли дата вчерашней
 * @param {Date|string} date - Дата для проверки
 * @returns {boolean} true, если дата - вчера
 */
const isYesterday = (date) => {
  const yesterday = new Date();
  yesterday.setDate(yesterday.getDate() - 1);
  
  const checkDate = new Date(date);
  
  return checkDate.getDate() === yesterday.getDate() &&
         checkDate.getMonth() === yesterday.getMonth() &&
         checkDate.getFullYear() === yesterday.getFullYear();
};

/**
 * Получение начала дня
 * @param {Date|string} date - Дата
 * @returns {Date} Дата с 00:00:00
 */
const startOfDay = (date) => {
  const d = new Date(date);
  d.setHours(0, 0, 0, 0);
  return d;
};

/**
 * Получение конца дня
 * @param {Date|string} date - Дата
 * @returns {Date} Дата с 23:59:59
 */
const endOfDay = (date) => {
  const d = new Date(date);
  d.setHours(23, 59, 59, 999);
  return d;
};
```

## Функции для работы со строками

### 9. Утилиты для работы со строками

```javascript
/**
 * Обрезка строки с добавлением многоточия
 * @param {string} str - Строка для обрезки
 * @param {number} maxLength - Максимальная длина
 * @param {string} suffix - Суффикс для добавления
 * @returns {string} Обрезанная строка
 */
const truncateString = (str, maxLength, suffix = '...') => {
  if (str.length <= maxLength) return str;
  return str.substring(0, maxLength - suffix.length) + suffix;
};

/**
 * Преобразование строки в camelCase
 * @param {string} str - Строка для преобразования
 * @returns {string} Строка в camelCase
 */
const toCamelCase = (str) => {
  return str.replace(/(?:^\w|[A-Z]|\b\w)/g, (word, index) => {
    return index === 0 ? word.toLowerCase() : word.toUpperCase();
  }).replace(/\s+/g, '');
};

/**
 * Преобразование строки в snake_case
 * @param {string} str - Строка для преобразования
 * @returns {string} Строка в snake_case
 */
const toSnakeCase = (str) => {
  return str.replace(/\W+/g, ' ')
    .split(/ |\B(?=[A-Z])/)
    .map(word => word.toLowerCase())
    .join('_');
};

/**
 * Преобразование строки в kebab-case
 * @param {string} str - Строка для преобразования
 * @returns {string} Строка в kebab-case
 */
const toKebabCase = (str) => {
  return str.replace(/\W+/g, ' ')
    .split(/ |\B(?=[A-Z])/)
    .map(word => word.toLowerCase())
    .join('-');
};

/**
 * Удаление HTML тегов из строки
 * @param {string} html - HTML строка
 * @returns {string} Строка без тегов
 */
const stripHtml = (html) => {
  const tmp = document.createElement('div');
  tmp.innerHTML = html;
  return tmp.textContent || tmp.innerText || '';
};

/**
 * Экранирование HTML специальных символов
 * @param {string} str - Строка для экранирования
 * @returns {string} Экранированная строка
 */
const escapeHtml = (str) => {
  const escapeMap = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#x27;',
    '/': '&#x2F;'
  };
  
  return str.replace(/[&<>"'\/]/g, (match) => escapeMap[match]);
};

/**
 * Генерация случайной строки
 * @param {number} length - Длина строки
 * @param {string} chars - Символы для генерации
 * @returns {string} Случайная строка
 */
const generateRandomString = (length = 10, chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789') => {
  let result = '';
  for (let i = 0; i < length; i++) {
    result += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return result;
};
```

## Функции для работы с хранилищем

### 10. Универсальные функции для LocalStorage

```javascript
/**
 * Универсальный менеджер хранилища
 */
class StorageManager {
  constructor(type = 'localStorage') {
    this.storage = window[type];
  }
  
  /**
   * Сохранение данных в хранилище
   * @param {string} key - Ключ
   * @param {*} value - Значение
   * @returns {boolean} true, если сохранение успешно
   */
  set(key, value) {
    try {
      const serializedValue = JSON.stringify(value);
      this.storage.setItem(key, serializedValue);
      return true;
    } catch (error) {
      console.error('Ошибка сохранения в хранилище:', error);
      return false;
    }
  }
  
  /**
   * Получение данных из хранилища
   * @param {string} key - Ключ
   * @param {*} defaultValue - Значение по умолчанию
   * @returns {*} Сохраненные данные или значение по умолчанию
   */
  get(key, defaultValue = null) {
    try {
      const item = this.storage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error('Ошибка получения из хранилища:', error);
      return defaultValue;
    }
  }
  
  /**
   * Удаление данных из хранилища
   * @param {string} key - Ключ
   */
  remove(key) {
    this.storage.removeItem(key);
  }
  
  /**
   * Проверка наличия ключа в хранилище
   * @param {string} key - Ключ
   * @returns {boolean} true, если ключ существует
   */
  has(key) {
    return this.storage.getItem(key) !== null;
  }
  
  /**
   * Очистка хранилища
   */
  clear() {
    this.storage.clear();
  }
  
  /**
   * Сохранение с временем жизни
   * @param {string} key - Ключ
   * @param {*} value - Значение
   * @param {number} ttlMs - Время жизни в миллисекундах
   * @returns {boolean} true, если сохранение успешно
   */
  setWithExpiry(key, value, ttlMs) {
    const item = {
      value: value,
      expiry: Date.now() + ttlMs
    };
    return this.set(key, item);
  }
  
  /**
   * Получение данных с проверкой времени жизни
   * @param {string} key - Ключ
   * @returns {*} Значение или null, если срок жизни истек
   */
  getWithExpiry(key) {
    const item = this.get(key);
    if (!item) return null;
    
    const now = Date.now();
    if (item.expiry && item.expiry < now) {
      this.remove(key);
      return null;
    }
    
    return item.value;
  }
}

// Экземпляр по умолчанию
const storage = new StorageManager();
```

## Функции для функционального программирования

### 11. Функциональные утилиты

```javascript
/**
 * Дебаунс функции
 * @param {Function} func - Функция для дебаунса
 * @param {number} delay - Задержка в миллисекундах
 * @returns {Function} Дебаунсированная функция
 */
const debounce = (func, delay) => {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
};

/**
 * Троттлинг функции
 * @param {Function} func - Функция для троттлинга
 * @param {number} limit - Ограничение в миллисекундах
 * @returns {Function} Троттлинг функция
 */
const throttle = (func, limit) => {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
};

/**
 * Мемоизация функции
 * @param {Function} fn - Функция для мемоизации
 * @returns {Function} Мемоизированная функция
 */
const memoize = (fn) => {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
};

/**
 * Композиция функций (справа налево)
 * @param {...Function} fns - Функции для композиции
 * @returns {Function} Композиция функций
 */
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);

/**
 * Цепочка вызовов (слева направо)
 * @param {...Function} fns - Функции для цепочки
 * @returns {Function} Цепочка функций
 */
const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);

/**
 * Каррирование функции
 * @param {Function} fn - Функция для каррирования
 * @returns {Function} Каррированная функция
 */
const curry = (fn) => {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...moreArgs) {
        return curried.apply(this, args.concat(moreArgs));
      };
    }
  };
};

/**
 * Повторение выполнения функции
 * @param {number} times - Количество повторений
 * @param {Function} fn - Функция для повторения
 * @returns {Array} Массив результатов
 */
const times = (times, fn) => {
  const results = [];
  for (let i = 0; i < times; i++) {
    results.push(fn(i));
  }
  return results;
};
```

## Ссылки по теме

- [[js/utils]] - Утилиты JavaScript
- [[js/snippets]] - Заготовки кода
- [[js/patterns]] - Паттерны JavaScript
- [[js/functional]] - Функциональное программирование
- [[js/dom/utils]] - DOM утилиты
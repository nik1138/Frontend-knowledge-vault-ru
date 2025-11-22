---
aliases: ["Хитрости JavaScript", "JS Tricks", "JavaScript Hacks"]
tags: [js, tricks, hacks, performance]
---

# Хитрости и лайфхаки JavaScript

## Введение

JavaScript полон интересных возможностей и хитростей, которые могут сделать ваш код более кратким, эффективным и читаемым. В этом разделе собраны полезные приемы, которые стоит знать каждому разработчику.

## Распространенные хитрости

### 1. Сокращенные условия

```javascript
// Вместо длинного условия
if (condition1 && condition2 && condition3) {
  doSomething();
}

// Используем сокращенную форму
condition1 && condition2 && condition3 && doSomething();

// Условный рендеринг (как в React)
const element = showElement && <div>Контент</div>;

// Условное выполнение с проверкой на существование
user && user.name && console.log(user.name);
```

### 2. Операторы объединения с null

```javascript
// Объединение с null (??) vs логическое ИЛИ (||)
const user = {
  name: 'Иван',
  age: 0,  // 0 - это ложное значение, но может быть валидным
  city: null
};

// Логическое ИЛИ вернет 'Неизвестно' для age = 0
const age1 = user.age || 'Неизвестно';  // 'Неизвестно'
const city1 = user.city || 'Москва';    // 'Москва'

// Объединение с null вернет 0 для age, но 'Москва' для null
const age2 = user.age ?? 'Неизвестно';  // 0
const city2 = user.city ?? 'Москва';    // 'Москва'
```

### 3. Опциональная цепочка (Optional Chaining)

```javascript
// Без опциональной цепочки
const userName = user && user.profile && user.profile.name;

// С опциональной цепочкой
const userName = user?.profile?.name;

// Вызов метода с проверкой
const result = obj?.method?.();

// Доступ к элементам массива
const firstItem = arr?.[0]?.property;

// Динамический доступ к свойствам
const value = obj?.[dynamicKey];
```

### 4. Сокращенное присваивание свойств объекта

```javascript
// Длинная запись
const name = 'Иван';
const age = 30;
const user = {
  name: name,
  age: age,
  getName: function() { return this.name; }
};

// Сокращенная запись
const name = 'Иван';
const age = 30;
const user = {
  name,  // name: name
  age,   // age: age
  getName() { return this.name; }  // Методы без function
};
```

### 5. Деструктуризация с переименованием и значениями по умолчанию

```javascript
const user = {
  name: 'Иван',
  age: 30,
  address: {
    city: 'Москва',
    street: 'Ленина'
  }
};

// Деструктуризация с переименованием
const { name: userName, age: userAge } = user;

// Деструктуризация со значениями по умолчанию
const { name = 'Аноним', email = 'no-email@example.com' } = user;

// Вложенная деструктуризация
const { address: { city, street } = {} } = user;

// Деструктуризация массива с пропуском элементов
const [first, , third, ...rest] = [1, 2, 3, 4, 5, 6];
// first = 1, third = 3, rest = [4, 5, 6]
```

## Расширенные хитрости

### 6. Слияние объектов с помощью spread-оператора

```javascript
// Объединение объектов
const defaults = { theme: 'light', lang: 'ru' };
const userPrefs = { theme: 'dark', notifications: true };
const settings = { ...defaults, ...userPrefs };  // { theme: 'dark', lang: 'ru', notifications: true }

// Глубокое копирование объекта
const original = { a: 1, b: { c: 2 } };
const copy = { ...original, b: { ...original.b } };

// Обновление конкретного свойства
const updatedUser = { ...user, profile: { ...user.profile, lastLogin: new Date() } };
```

### 7. Уникальные значения из массива

```javascript
// Получение уникальных значений
const numbers = [1, 2, 2, 3, 4, 4, 5];
const uniqueNumbers = [...new Set(numbers)];  // [1, 2, 3, 4, 5]

// Уникальные объекты по свойству
const users = [
  { id: 1, name: 'Иван' },
  { id: 2, name: 'Мария' },
  { id: 1, name: 'Иван-2' }
];

// Уникальные по id
const uniqueUsers = users.filter((user, index, self) =>
  index === self.findIndex(u => u.id === user.id)
);
```

### 8. Проверка на пустой объект/массив

```javascript
// Проверка пустого объекта
const isEmptyObject = (obj) => Object.keys(obj).length === 0;

// Проверка пустого массива
const isEmptyArray = (arr) => Array.isArray(arr) && arr.length === 0;

// Универсальная проверка
const isEmpty = (value) => {
  if (Array.isArray(value)) return value.length === 0;
  if (typeof value === 'object' && value !== null) return Object.keys(value).length === 0;
  if (typeof value === 'string') return value.trim() === '';
  return !value;
};
```

### 9. Форматирование чисел и строк

```javascript
// Форматирование чисел
const number = 1234567.89;
const formatted = number.toLocaleString('ru-RU');  // "1 234 567,89"

// Форматирование с фиксированным количеством знаков после запятой
const fixed = number.toFixed(2);  // "1234567.89"

// Форматирование с разделением разрядов
const formattedNumber = number.toLocaleString('ru-RU', {
  minimumFractionDigits: 2,
  maximumFractionDigits: 2
});

// Повторение строки
const repeated = 'Hello '.repeat(3);  // "Hello Hello Hello "

// Заполнение строки
const padded = '42'.padStart(5, '0');  // "00042"
const paddedEnd = '42'.padEnd(5, '0');  // "42000"
```

### 10. Работа с URL и параметрами

```javascript
// Работа с URL параметрами
const urlParams = new URLSearchParams(window.location.search);
const id = urlParams.get('id');
const name = urlParams.get('name');

// Создание параметров из объекта
const params = new URLSearchParams({
  search: 'javascript',
  page: 1,
  limit: 10
}).toString();  // "search=javascript&page=1&limit=10"

// Добавление параметров к URL
const baseUrl = 'https://api.example.com/users';
const fullUrl = `${baseUrl}?${params}`;
```

## Хитрости производительности

### 11. Memoization (кеширование результатов функций)

```javascript
// Простая реализация memoization
function memoize(fn) {
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
}

// Использование
const expensiveFunction = memoize((n) => {
  console.log(`Вычисляем для ${n}`);
  return n * n;
});

expensiveFunction(5);  // Вычисляем для 5, возвращает 25
expensiveFunction(5);  // Возвращает 25 из кэша
```

### 12. Debounce и Throttle

```javascript
// Debounce - выполнение функции только после окончания вызовов
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// Throttle - выполнение функции с заданной частотой
function throttle(func, limit) {
  let inThrottle;
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// Использование
const debouncedSearch = debounce((query) => {
  console.log('Поиск:', query);
}, 300);

const throttledScroll = throttle(() => {
  console.log('Скролл');
}, 100);
```

### 13. Эффективная фильтрация и поиск

```javascript
// Быстрый поиск с использованием Set
const items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const searchSet = new Set(items);

// O(1) вместо O(n) для поиска
const exists = searchSet.has(5);  // true

// Фильтрация с использованием объекта для поиска
const filterIds = [2, 4, 6];
const filterMap = Object.fromEntries(filterIds.map(id => [id, true]));

const filtered = items.filter(item => filterMap[item]);
```

## Неочевидные особенности языка

### 14. Особенности сравнения

```javascript
// Сравнение объектов
const obj1 = { a: 1 };
const obj2 = { a: 1 };
console.log(obj1 == obj2);  // false
console.log(obj1 === obj2); // false

// Сравнение массивов
const arr1 = [1, 2, 3];
const arr2 = [1, 2, 3];
console.log(arr1 == arr2);  // false
console.log(arr1 === arr2); // false

// Но:
console.log([1, 2, 3] == [1, 2, 3]); // false
console.log(JSON.stringify([1, 2, 3]) === JSON.stringify([1, 2, 3])); // true

// Особенности сортировки
const numbers = [10, 2, 1, 20];
console.log(numbers.sort());  // [1, 10, 2, 20] - сортировка как строк!

// Правильная числовая сортировка
console.log(numbers.sort((a, b) => a - b));  // [1, 2, 10, 20]
```

### 15. Особенности работы с this

```javascript
const obj = {
  name: 'Объект',
  regularFunction: function() {
    console.log(this.name);  // 'Объект'
  },
  arrowFunction: () => {
    console.log(this.name);  // undefined (в браузере this = Window)
  }
};

// Привязка контекста
const boundFunction = obj.regularFunction.bind({ name: 'Привязанный' });
boundFunction();  // 'Привязанный'
```

### 16. Особенности работы с замыканиями

```javascript
// Проблема с циклом
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);  // 3, 3, 3
}

// Решение 1: let вместо var
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);  // 0, 1, 2
}

// Решение 2: IIFE
for (var i = 0; i < 3; i++) {
  (function(index) {
    setTimeout(() => console.log(index), 100);  // 0, 1, 2
  })(i);
}
```

## Практические хитрости

### 17. Работа с датами

```javascript
// Быстрое создание диапазона дат
const getDates = (startDate, endDate) => {
  const dates = [];
  const currentDate = new Date(startDate);
  const end = new Date(endDate);
  
  while (currentDate <= end) {
    dates.push(new Date(currentDate));
    currentDate.setDate(currentDate.getDate() + 1);
  }
  
  return dates;
};

// Форматирование даты
const formatDate = (date) => {
  return new Date(date).toLocaleDateString('ru-RU', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    weekday: 'long'
  });
};

// Проверка, является ли дата сегодняшней
const isToday = (someDate) => {
  const today = new Date();
  return someDate.getDate() == today.getDate() &&
    someDate.getMonth() == today.getMonth() &&
    someDate.getFullYear() == today.getFullYear();
};
```

### 18. Работа с DOM

```javascript
// Быстрое создание элемента
const createElement = (tag, props = {}, children = []) => {
  const element = document.createElement(tag);
  
  // Установка атрибутов
  Object.entries(props).forEach(([key, value]) => {
    if (key.startsWith('on')) {
      // Обработчики событий
      element.addEventListener(key.slice(2).toLowerCase(), value);
    } else if (key === 'className') {
      element.className = value;
    } else if (key === 'style' && typeof value === 'object') {
      Object.assign(element.style, value);
    } else {
      element.setAttribute(key, value);
    }
  });
  
  // Добавление детей
  children.forEach(child => {
    element.appendChild(
      typeof child === 'string' ? document.createTextNode(child) : child
    );
  });
  
  return element;
};

// Использование
const button = createElement('button', {
  className: 'btn btn-primary',
  onClick: () => console.log('Клик!')
}, ['Нажми меня']);
```

### 19. Проверка типов

```javascript
// Улучшенная проверка типов
const getType = (value) => {
  if (value === null) return 'null';
  if (value === undefined) return 'undefined';
  if (Array.isArray(value)) return 'array';
  return Object.prototype.toString.call(value).slice(8, -1).toLowerCase();
};

// Проверки
console.log(getType([]));        // 'array'
console.log(getType({}));        // 'object'
console.log(getType(new Date())); // 'date'
console.log(getType(null));      // 'null'
console.log(getType(/regex/));   // 'regexp'

// Проверка на простой объект
const isPlainObject = (obj) => {
  return obj != null && 
         typeof obj === 'object' && 
         obj.constructor === Object &&
         Object.getPrototypeOf(obj) === Object.prototype;
};
```

### 20. Работа с асинхронностью

```javascript
// Ожидание с таймаутом
const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Таймаут для асинхронной операции
const withTimeout = (promise, timeoutMs) => {
  return Promise.race([
    promise,
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Таймаут')), timeoutMs)
    )
  ]);
};

// Использование
async function example() {
  try {
    const result = await withTimeout(
      fetch('/api/data'),
      5000  // 5 секунд таймаут
    );
    console.log(result);
  } catch (error) {
    console.error('Ошибка:', error.message);
  }
}

// Параллельное выполнение с обработкой ошибок
const parallelSafe = async (promises) => {
  return await Promise.allSettled(promises)
    .then(results => {
      const successful = results
        .filter(r => r.status === 'fulfilled')
        .map(r => r.value);
      
      const failed = results
        .filter(r => r.status === 'rejected')
        .map(r => r.reason);
      
      return { successful, failed };
    });
};
```

## Полезные сокращения

### 21. Условные операции

```javascript
// Множественный выбор
const status = code => ({
  200: 'OK',
  404: 'Not Found',
  500: 'Server Error'
}[code] || 'Unknown');

// Сокращенное присваивание с проверкой
const assignIfValid = (obj, key, value) => {
  if (value != null) obj[key] = value;
  return obj;
};

// Быстрая проверка вхождения
const includes = (arr, value) => arr.includes(value);
const oneOf = (value, ...validValues) => validValues.includes(value);

// Проверка на вхождение в список
const isValidStatus = (status) => ['active', 'inactive', 'pending'].includes(status);
```

### 22. Работа с массивами

```javascript
// Группировка массива по свойству
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

// Разделение массива на чанки
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

// Перемешивание массива (алгоритм Фишера-Йетса)
const shuffle = (array) => {
  const result = [...array];
  for (let i = result.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
};
```

### 23. Работа с объектами

```javascript
// Глубокое объединение объектов
const deepMerge = (target, source) => {
  const result = { ...target };
  
  for (const key in source) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      result[key] = deepMerge(result[key] || {}, source[key]);
    } else {
      result[key] = source[key];
    }
  }
  
  return result;
};

// Выбор подмножества свойств объекта
const pick = (obj, keys) => {
  return keys.reduce((result, key) => {
    if (key in obj) {
      result[key] = obj[key];
    }
    return result;
  }, {});
};

// Исключение свойств из объекта
const omit = (obj, keys) => {
  const result = { ...obj };
  keys.forEach(key => delete result[key]);
  return result;
};
```

## Ссылки по теме

- [[js/performance/optimization]] - Оптимизация производительности
- [[js/es6+/features]] - Современные возможности JavaScript
- [[js/best-practices]] - Лучшие практики
- [[js/troubleshooting/common-issues]] - Распространенные проблемы
---
tags: [javascript, frontend, advanced, functional-programming, pure-functions]
aliases: [функциональное программирование в js для фронтенда, чистые функции, иммутабельность]
---

# Функциональное программирование в JavaScript - Практические примеры для Frontend разработки

## Введение

Функциональное программирование - это парадигма программирования, в которой вычисления рассматриваются как вычисление математических функций без изменения состояния и без данных с побочными эффектами. В этой статье рассмотрим практические примеры функционального программирования с акцентом на задачи фронтенд разработки.

## Основные принципы функционального программирования

### 1. Чистые функции (Pure Functions)

Чистая функция всегда возвращает одинаковый результат для одних и тех же входных данных и не имеет побочных эффектов.

```javascript
// Чистая функция
function calculateTotalPrice(items) {
  return items.reduce((total, item) => total + (item.price * item.quantity), 0);
}

// Не чистая функция (зависит от внешнего состояния)
let taxRate = 0.1;
function calculateTotalWithTax(items) {
  return calculateTotalPrice(items) * (1 + taxRate); // Зависит от внешней переменной
}

// Чистая альтернатива
function calculateTotalWithTax(items, taxRate) {
  return calculateTotalPrice(items) * (1 + taxRate);
}
```

### 2. Иммутабельность (Immutability)

Данные не изменяются, вместо этого создаются новые версии данных.

```javascript
// Нарушение иммутабельности
function addItemToCart_bad(cart, newItem) {
  cart.items.push(newItem); // Изменяет исходный массив
  cart.total += newItem.price;
  return cart;
}

// Соблюдение иммутабельности
function addItemToCart(cart, newItem) {
  return {
    ...cart,
    items: [...cart.items, { ...newItem }],
    total: cart.total + newItem.price
  };
}

// Пример использования
const initialCart = {
  items: [{ id: 1, name: 'Товар 1', price: 100, quantity: 1 }],
  total: 100
};

const updatedCart = addItemToCart(initialCart, { id: 2, name: 'Товар 2', price: 200, quantity: 1 });

console.log(initialCart.total); // 100 - не изменилось
console.log(updatedCart.total); // 300 - новое значение
```

## Практический пример: Управление состоянием с функциональным подходом

```javascript
// Функции для работы с состоянием корзины покупок
const cartInitialState = {
  items: [],
  total: 0,
  discount: 0
};

// Чистые функции-редьюсеры
function addItem(state, item) {
  const existingItem = state.items.find(i => i.id === item.id);
  
  if (existingItem) {
    return {
      ...state,
      items: state.items.map(i => 
        i.id === item.id 
          ? { ...i, quantity: i.quantity + 1 } 
          : i
      )
    };
  }
  
  return {
    ...state,
    items: [...state.items, { ...item, quantity: 1 }]
  };
}

function removeItem(state, itemId) {
  const itemToRemove = state.items.find(i => i.id === itemId);
  
  if (!itemToRemove) return state;
  
  return {
    ...state,
    items: state.items.filter(i => i.id !== itemId),
    total: state.total - (itemToRemove.price * itemToRemove.quantity)
  };
}

function updateQuantity(state, itemId, newQuantity) {
  if (newQuantity <= 0) {
    return removeItem(state, itemId);
  }
  
  return {
    ...state,
    items: state.items.map(item => 
      item.id === itemId 
        ? { ...item, quantity: newQuantity } 
        : item
    )
  };
}

function applyDiscount(state, discountPercent) {
  return {
    ...state,
    discount: Math.min(100, Math.max(0, discountPercent))
  };
}

// Комбинирование редьюсеров
function cartReducer(state = cartInitialState, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return addItem(state, action.payload);
    case 'REMOVE_ITEM':
      return removeItem(state, action.payload);
    case 'UPDATE_QUANTITY':
      return updateQuantity(state, action.payload.itemId, action.payload.quantity);
    case 'APPLY_DISCOUNT':
      return applyDiscount(state, action.payload);
    case 'RESET_CART':
      return cartInitialState;
    default:
      return state;
  }
}

// Использование в компоненте
function useCart(initialState = cartInitialState) {
  const [state, dispatch] = React.useReducer(cartReducer, initialState);
  
  return {
    state,
    addItem: (item) => dispatch({ type: 'ADD_ITEM', payload: item }),
    removeItem: (itemId) => dispatch({ type: 'REMOVE_ITEM', payload: itemId }),
    updateQuantity: (itemId, quantity) => dispatch({ 
      type: 'UPDATE_QUANTITY', 
      payload: { itemId, quantity } 
    }),
    applyDiscount: (discount) => dispatch({ type: 'APPLY_DISCOUNT', payload: discount })
  };
}
```

## Практический пример: Композиция функций для обработки данных

```javascript
// Утилиты для композиции функций
const compose = (...fns) => (value) => fns.reduceRight((acc, fn) => fn(acc), value);
const pipe = (...fns) => (value) => fns.reduce((acc, fn) => fn(acc), value);

// Функции для обработки пользовательских данных
const trim = (str) => str.trim();
const toLowerCase = (str) => str.toLowerCase();
const normalizeEmail = (email) => pipe(trim, toLowerCase)(email);

const validateEmail = (email) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

const maskEmail = (email) => {
  const [local, domain] = email.split('@');
  const maskedLocal = local.substring(0, 1) + '***' + local.substring(local.length - 1);
  return `${maskedLocal}@${domain}`;
};

// Композиция функций для обработки email
const processEmail = compose(
  maskEmail,
  normalizeEmail
);

// Использование
const rawEmail = "  John.Doe@Example.Com  ";
const processedEmail = processEmail(rawEmail); // "j***e@example.com"

// Проверка и обработка
const validateAndProcessEmail = (email) => {
  const normalized = normalizeEmail(email);
  return validateEmail(normalized) ? processEmail(email) : null;
};
```

## Практический пример: Функции высшего порядка для обработки списков

```javascript
// Функции высшего порядка для работы с коллекциями данных
const map = (fn) => (array) => array.map(fn);
const filter = (predicate) => (array) => array.filter(predicate);
const reduce = (reducer, initial) => (array) => array.reduce(reducer, initial);

// Функции для работы с продуктами
const products = [
  { id: 1, name: 'Ноутбук', price: 50000, category: 'электроника', inStock: true },
  { id: 2, name: 'Книга', price: 500, category: 'книги', inStock: true },
  { id: 3, name: 'Телефон', price: 30000, category: 'электроника', inStock: false },
  { id: 4, name: 'Чай', price: 200, category: 'продукты', inStock: true }
];

// Композиция операций над продуктами
const getAvailableElectronics = pipe(
  filter(product => product.inStock),
  filter(product => product.category === 'электроника'),
  map(product => ({ ...product, discountedPrice: product.price * 0.9 }))
);

const availableElectronics = getAvailableElectronics(products);
console.log(availableElectronics);

// Функция для группировки продуктов по категории
const groupBy = (key) => (array) => 
  array.reduce((groups, item) => {
    const groupKey = item[key];
    groups[groupKey] = groups[groupKey] || [];
    groups[groupKey].push(item);
    return groups;
  }, {});

const groupProductsByCategory = pipe(
  groupBy('category')
);

const productsByCategory = groupProductsByCategory(products);
console.log(productsByCategory);
```

## Практический пример: Функциональные компоненты с чистыми функциями

```javascript
// Функциональные компоненты с чистыми функциями
const ProductCard = ({ product, onAddToCart }) => {
  return React.createElement('div', { className: 'product-card' }, [
    React.createElement('h3', { key: 'title' }, product.name),
    React.createElement('p', { key: 'price' }, `Цена: ${product.price} руб.`),
    React.createElement('button', {
      key: 'add-btn',
      onClick: () => onAddToCart(product),
      disabled: !product.inStock
    }, product.inStock ? 'Добавить в корзину' : 'Нет в наличии')
  ]);
};

// Функция для фильтрации продуктов
const createProductFilter = (filters) => (products) => {
  return products.filter(product => {
    const matchesCategory = !filters.category || product.category === filters.category;
    const matchesInStock = !filters.inStock || product.inStock;
    const matchesMinPrice = !filters.minPrice || product.price >= filters.minPrice;
    const matchesMaxPrice = !filters.maxPrice || product.price <= filters.maxPrice;
    
    return matchesCategory && matchesInStock && matchesMinPrice && matchesMaxPrice;
  });
};

// Использование фильтра
const filterProducts = createProductFilter({
  category: 'электроника',
  inStock: true,
  minPrice: 10000,
  maxPrice: 100000
});

const filteredProducts = filterProducts(products);
```

## Практический пример: Функции для работы с DOM (без мутаций)

```javascript
// Функции для функциональной работы с DOM
const createElement = (tag, props = {}, ...children) => {
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
  
  // Добавление дочерних элементов
  children.flat().forEach(child => {
    if (typeof child === 'string' || typeof child === 'number') {
      element.appendChild(document.createTextNode(child));
    } else if (child instanceof Element) {
      element.appendChild(child);
    }
  });
  
  return element;
};

// Функция для создания списка элементов
const createList = (items, renderItem) => {
  return createElement('ul', { className: 'item-list' }, 
    items.map((item, index) => 
      createElement('li', { key: index }, renderItem(item))
    )
  );
};

// Пример использования
const todoItems = [
  { id: 1, text: 'Купить продукты', completed: false },
  { id: 2, text: 'Сходить в спортзал', completed: true },
  { id: 3, text: 'Прочитать книгу', completed: false }
];

const renderTodoItem = (todo) => 
  createElement('div', { 
    className: `todo-item ${todo.completed ? 'completed' : ''}` 
  }, [
    createElement('input', {
      type: 'checkbox',
      checked: todo.completed,
      onChange: (e) => toggleTodo(todo.id, e.target.checked)
    }),
    createElement('span', {}, todo.text)
  ]);

const todoList = createList(todoItems, renderTodoItem);
document.getElementById('app').appendChild(todoList);

// Функция для обновления состояния (без мутаций)
let todos = [...todoItems]; // иммутабельная копия

function toggleTodo(id, completed) {
  todos = todos.map(todo => 
    todo.id === id ? { ...todo, completed } : todo
  );
  
  // Пересоздаем список (в реальном приложении использовался бы Virtual DOM)
  const newTodoList = createList(todos, renderTodoItem);
  const container = document.getElementById('app');
  container.replaceChild(newTodoList, container.firstChild);
}
```

## Практический пример: Функции для валидации форм

```javascript
// Функции для функциональной валидации форм
const validators = {
  required: (value) => value && value.trim().length > 0,
  email: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  minLength: (min) => (value) => value && value.length >= min,
  maxLength: (max) => (value) => value && value.length <= max,
  matches: (pattern) => (value) => pattern.test(value)
};

// Композиция валидаторов
const combineValidators = (...validatorFns) => (value) => {
  return validatorFns.reduce((result, validator) => {
    if (!result.isValid) return result; // Если уже невалидно, не продолжаем
    
    const currentResult = validator(value);
    return typeof currentResult === 'boolean' 
      ? { isValid: currentResult, error: currentResult ? null : 'Поле не прошло валидацию' }
      : currentResult;
  }, { isValid: true, error: null });
};

// Создание валидатора для конкретного поля
const createFieldValidator = (rules) => (value) => {
  const validatorsToApply = rules.map(rule => {
    if (typeof rule === 'function') return rule;
    if (typeof rule === 'string' && validators[rule]) return validators[rule];
    return rule;
  });
  
  return combineValidators(...validatorsToApply)(value);
};

// Определение правил валидации
const validationRules = {
  email: [
    validators.required,
    validators.email
  ],
  password: [
    validators.required,
    validators.minLength(8),
    value => validators.matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)(value)
      ? { isValid: true }
      : { isValid: false, error: 'Пароль должен содержать заглавную букву, строчную букву и цифру' }
  ],
  confirmPassword: [
    validators.required,
    (value, formData) => value === formData.password
      ? { isValid: true }
      : { isValid: false, error: 'Пароли не совпадают' }
  ]
};

// Валидация всей формы
const validateForm = (formData, rules) => {
  const results = {};
  let isFormValid = true;
  
  Object.keys(rules).forEach(fieldName => {
    const fieldValidator = createFieldValidator(rules[fieldName]);
    const result = fieldValidator(formData[fieldName], formData);
    
    results[fieldName] = result;
    if (!result.isValid) {
      isFormValid = false;
    }
  });
  
  return { isValid: isFormValid, results };
};

// Использование валидации формы
const formData = {
  email: 'user@example.com',
  password: 'MySecurePassword123',
  confirmPassword: 'MySecurePassword123'
};

const formValidation = validateForm(formData, validationRules);
console.log('Результат валидации:', formValidation);
```

## Практический пример: Мемоизация для оптимизации вычислений

```javascript
// Функция для мемоизации
const memoize = (fn) => {
  const cache = new Map();
  
  return (...args) => {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Взято из кеша');
      return cache.get(key);
    }
    
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
};

// Мемоизированная функция для вычисления сложных выражений
const expensiveCalculation = memoize((a, b, operation) => {
  console.log('Выполняется дорогое вычисление...');
  
  switch(operation) {
    case 'add':
      return a + b;
    case 'multiply':
      return a * b;
    case 'complex':
      // Симуляция сложного вычисления
      let result = 0;
      for (let i = 0; i < 1000000; i++) {
        result += Math.sqrt(i) * a * b;
      }
      return result;
    default:
      return 0;
  }
});

// Первый вызов выполнит вычисление
console.log(expensiveCalculation(5, 3, 'multiply')); // Выполняется вычисление

// Повторный вызов с теми же аргументами использует кеш
console.log(expensiveCalculation(5, 3, 'multiply')); // Взято из кеша
```

## Практический пример: Функции для работы с API

```javascript
// Функции для функциональной работы с API
const createApiService = (baseUrl) => {
  // Чистая функция для создания HTTP запроса
  const makeRequest = async (endpoint, options = {}) => {
    const url = `${baseUrl}${endpoint}`;
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    };
    
    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return response.json();
  };
  
  // Функции для конкретных операций
  const api = {
    getUsers: () => makeRequest('/users'),
    getUser: (id) => makeRequest(`/users/${id}`),
    createUser: (userData) => makeRequest('/users', {
      method: 'POST',
      body: JSON.stringify(userData)
    }),
    updateUser: (id, userData) => makeRequest(`/users/${id}`, {
      method: 'PUT',
      body: JSON.stringify(userData)
    }),
    deleteUser: (id) => makeRequest(`/users/${id}`, {
      method: 'DELETE'
    })
  };
  
  return api;
};

// Использование API сервиса
const apiService = createApiService('https://api.example.com');

// Функция для обработки пользовательских данных
const processUserData = (users) => {
  return users
    .filter(user => user.active)
    .map(user => ({
      ...user,
      displayName: `${user.firstName} ${user.lastName}`,
      avatar: user.avatar || '/default-avatar.png'
    }))
    .sort((a, b) => a.displayName.localeCompare(b.displayName));
};

// Комбинирование API вызова и обработки данных
const loadAndProcessUsers = async () => {
  try {
    const users = await apiService.getUsers();
    return processUserData(users);
  } catch (error) {
    console.error('Ошибка загрузки пользователей:', error);
    throw error;
  }
};

// Использование
loadAndProcessUsers()
  .then(processedUsers => {
    // Обновление UI с обработанными данными
    renderUserList(processedUsers);
  })
  .catch(error => {
    showErrorMessage('Не удалось загрузить пользователей');
  });
```

## Лучшие практики функционального программирования в фронтенд разработке

### 1. Используйте неизменяемые структуры данных

```javascript
// Хорошо - использование неизменяемых операций
const addItemToCart = (cart, newItem) => ({
  ...cart,
  items: [...cart.items, { ...newItem, id: generateId() }]
});

// Плохо - мутация исходных данных
const addItemToCart_bad = (cart, newItem) => {
  cart.items.push({ ...newItem, id: generateId() }); // Мутирует cart
  return cart;
};
```

### 2. Создавайте чистые функции для бизнес-логики

```javascript
// Хорошо - чистая функция для вычислений
const calculateDiscount = (total, discountPercent) => {
  return total - (total * (discountPercent / 100));
};

// Плохо - функция с побочными эффектами
const calculateDiscount_bad = (total, discountPercent) => {
  const discount = total * (discountPercent / 100);
  localStorage.setItem('lastDiscount', discount); // Побочный эффект
  return total - discount;
};
```

### 3. Используйте композицию для создания сложной логики

```javascript
// Композиция для сложных преобразований
const processUserInput = pipe(
  trim,
  normalizeWhitespace,
  sanitizeHTML,
  validateInput
);
```

## Заключение

Функциональное программирование в JavaScript фронтенд разработке помогает создавать более предсказуемые, тестируемые и надежные приложения. Использование чистых функций, иммутабельности и композиции позволяет избежать многих распространенных ошибок и упрощает отладку приложений.

## См. также

- [[closure_practical_examples.md]]
- [[Прототипное наследование в JavaScript]]
- [[Метапрограммирование и прокси в JavaScript]]
- [[Генераторы в JavaScript]]
- [[DOM манипуляции в JavaScript]]
- [[События в JavaScript для фронтенд разработки]]
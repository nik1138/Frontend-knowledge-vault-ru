---
aliases: ["Модули ES6", "ECMAScript 6 Модули"]
tags: ["#javascript", "#es6", "#modules", "#frontend", "#architecture"]
---

# ES6-модули

ES6-модули (также известные как ECMAScript 2015 модули) представляют собой стандартный способ организации и структурирования JavaScript кода, позволяющий экспортировать и импортировать функции, объекты или примитивы между файлами. Это важный элемент современной модульной архитектуры фронтенд-разработки.

## Обзор

ES6-модули были введены в спецификацию ECMAScript 2015 как стандартный способ работы с модулями в JavaScript. До этого разработчики полагались на сторонние решения, такие как CommonJS, AMD или IIFE (Immediately Invoked Function Expressions). ES6-модули позволяют создавать более предсказуемую и понятную архитектуру кода, обеспечивая явную зависимость между файлами и поддерживая статический анализ.

## Основные концепции

### Экспорт (Export)

Существует два типа экспорта:

- **Именованный экспорт (Named Export)**: позволяет экспортировать несколько значений из модуля
- **Дефолтный экспорт (Default Export)**: позволяет экспортировать одно значение из модуля

#### Именованный экспорт

```javascript
// mathUtils.js
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

// Также можно экспортировать переменные
const subtract = (a, b) => a - b;
export { subtract };
```

#### Дефолтный экспорт

```javascript
// calculator.js
const Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  divide: (a, b) => a / b
};

export default Calculator;
```

Можно также экспортировать функцию напрямую:

```javascript
// greeting.js
export default function greet(name) {
  return `Hello, ${name}!`;
}
```

### Импорт (Import)

Соответственно, для импорта используются следующие синтаксисы:

#### Импорт именованных экспортов

```javascript
// main.js
import { add, multiply, PI } from './mathUtils.js';

console.log(PI); // 3.14159
console.log(add(2, 3)); // 5
console.log(multiply(4, 5)); // 20
```

#### Импорт дефолтного экспорта

```javascript
// main.js
import Calculator from './calculator.js';

console.log(Calculator.add(2, 3)); // 5
```

#### Комбинированный импорт

```javascript
// main.js
import Calculator, { add, multiply, PI } from './mathUtils.js';

console.log(PI);
console.log(add(2, 3));
console.log(Calculator.add(1, 2));
```

#### Импорт всех экспортированных значений

```javascript
// main.js
import * as mathUtils from './mathUtils.js';

console.log(mathUtils.PI); // 3.14159
console.log(mathUtils.add(2, 3)); // 5
console.log(mathUtils.multiply(4, 5)); // 20
```

#### Импорт с переименованием

```javascript
// main.js
import { add as sum, multiply as product } from './mathUtils.js';

console.log(sum(2, 3)); // 5
console.log(product(4, 5)); // 20
```

## Практические примеры

### Пример 1: Модуль для работы с API

```javascript
// apiService.js
const API_BASE_URL = 'https://api.example.com';

export async function fetchUserData(userId) {
  try {
    const response = await fetch(`${API_BASE_URL}/users/${userId}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Error fetching user data:', error);
    throw error;
  }
}

export async function updateUserProfile(userId, profileData) {
  try {
    const response = await fetch(`${API_BASE_URL}/users/${userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(profileData),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Error updating user profile:', error);
    throw error;
  }
}

export default {
  fetchUserData,
  updateUserProfile
};
```

### Пример 2: Модуль утилит

```javascript
// utils.js
// Именованные экспорты
export const formatDate = (date) => {
  return new Date(date).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });
};

export const formatCurrency = (amount, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: currency
  }).format(amount);
};

// Дефолтный экспорт
export default function validateEmail(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}
```

```javascript
// main.js
import validateEmail, { formatDate, formatCurrency } from './utils.js';

console.log(validateEmail('test@example.com')); // true
console.log(formatDate(new Date())); // November 23, 2025
console.log(formatCurrency(1234.56)); // $1,234.56
```

## Преимущества ES6-модулей

1. **Статический анализ**: Импорты и экспорты определяются на этапе компиляции, что позволяет инструментам проводить анализ зависимостей и оптимизацию кода.

2. **Ленивая загрузка**: Современные сборщики могут использовать информацию о модулях для реализации ленивой загрузки и разделения кода.

3. **Явные зависимости**: Каждый модуль явно указывает, какие зависимости он имеет, что улучшает читаемость и обслуживаемость кода.

4. **Изоляция области видимости**: Каждый модуль имеет свою область видимости, что предотвращает загрязнение глобального пространства имен.

5. **Поддержка деревьев (Tree Shaking)**: Позволяет удалять неиспользуемый код при сборке, уменьшая размер итогового бандла.

## Совместимость и использование

ES6-модули поддерживаются всеми современными браузерами. Для использования в браузере файл должен быть подключен с атрибутом `type="module"`:

```html
<script type="module" src="main.js"></script>
```

При использовании модулей в браузере важно учитывать, что пути к модулям должны быть указаны с расширением файла (например, `.js`), а также использовать абсолютные или относительные пути.

## Связанные концепции

- [[Модульные-паттерны]] - другие паттерны организации модулей
- [[Зависимости-модулей]] - управление зависимостями между модулями
- [[Ленивая-загрузка]] - техники динамической загрузки модулей
- [[Code-splitting]] - разделение кода на отдельные бандлы
- [[CommonJS]] - предыдущий стандарт модулей в Node.js
- [[AMD]] - асинхронный модульный паттерн

## Заключение

ES6-модули являются фундаментальным элементом современной архитектуры JavaScript-приложений. Они обеспечивают чистую структуру кода, явные зависимости и возможность эффективной оптимизации. Понимание и правильное использование ES6-модулей критически важно для разработчиков, стремящихся создавать масштабируемые и поддерживаемые фронтенд-приложения.
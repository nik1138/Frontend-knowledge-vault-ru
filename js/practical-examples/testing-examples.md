---
aliases: ["Примеры тестирования JavaScript", "JS Testing Examples"]
tags: [javascript, testing, frontend, unit-test]
---

# Примеры тестирования JavaScript кода

Этот документ содержит практические примеры тестирования JavaScript кода, включая модульное тестирование, интеграционное тестирование и тестирование пользовательского интерфейса.

## Основы тестирования

### 1. Простой тестовый фреймворк

```javascript
// simple-test-framework.js - простой тестовый фреймворк
class SimpleTestFramework {
  constructor() {
    this.tests = [];
    this.results = {
      passed: 0,
      failed: 0,
      total: 0
    };
    this.currentSuite = null;
  }
  
  /**
   * Добавляет тест
   * @param {string} name - имя теста
   * @param {function} testFn - функция теста
   */
  test(name, testFn) {
    this.tests.push({
      name,
      testFn,
      suite: this.currentSuite
    });
  }
  
  /**
   * Добавляет тест в группу
   * @param {string} name - имя группы
   * @param {function} suiteFn - функция группы тестов
   */
  describe(name, suiteFn) {
    const previousSuite = this.currentSuite;
    this.currentSuite = name;
    
    suiteFn();
    
    this.currentSuite = previousSuite;
  }
  
  /**
   * Выполняет все тесты
   */
  async run() {
    console.log('=== Запуск тестов ===');
    
    for (const test of this.tests) {
      this.results.total++;
      
      try {
        await Promise.resolve(test.testFn());
        this.results.passed++;
        console.log(`✓ ${test.suite ? `[${test.suite}] ` : ''}${test.name}`);
      } catch (error) {
        this.results.failed++;
        console.log(`✗ ${test.suite ? `[${test.suite}] ` : ''}${test.name}`);
        console.log(`  Ошибка: ${error.message}`);
      }
    }
    
    console.log('=== Результаты ===');
    console.log(`Всего: ${this.results.total}`);
    console.log(`Пройдено: ${this.results.passed}`);
    console.log(`Провалено: ${this.results.failed}`);
    console.log(`Процент успеха: ${((this.results.passed / this.results.total) * 100).toFixed(2)}%`);
    
    return {
      ...this.results,
      success: this.results.failed === 0
    };
  }
  
  /**
   * Проверяет равенство значений
   * @param {*} actual - фактическое значение
   * @param {*} expected - ожидаемое значение
   * @param {string} message - сообщение об ошибке
   */
  assertEqual(actual, expected, message = 'Значения должны быть равны') {
    if (actual !== expected) {
      throw new Error(`${message}: ожидается ${expected}, получено ${actual}`);
    }
  }
  
  /**
   * Проверяет, что значение истинно
   * @param {*} value - значение для проверки
   * @param {string} message - сообщение об ошибке
   */
  assertTrue(value, message = 'Значение должно быть истинным') {
    if (!value) {
      throw new Error(message);
    }
  }
  
  /**
   * Проверяет, что значение ложно
   * @param {*} value - значение для проверки
   * @param {string} message - сообщение об ошибке
   */
  assertFalse(value, message = 'Значение должно быть ложным') {
    if (value) {
      throw new Error(message);
    }
  }
  
  /**
   * Проверяет, что функция выбрасывает ошибку
   * @param {function} fn - функция для проверки
   * @param {string} message - сообщение об ошибке
   */
  assertThrows(fn, message = 'Функция должна выбросить ошибку') {
    try {
      fn();
      throw new Error(message);
    } catch (error) {
      if (error.message === message) {
        throw error; // Это наша собственная ошибка, а не выброшенная функцией
      }
      // Функция действительно выбросила ошибку
    }
  }
  
  /**
   * Проверяет, что массив содержит элемент
   * @param {Array} array - массив для проверки
   * @param {*} element - элемент для поиска
   * @param {string} message - сообщение об ошибке
   */
  assertArrayContains(array, element, message = 'Массив должен содержать элемент') {
    if (!array.includes(element)) {
      throw new Error(message);
    }
  }
}

// Глобальные функции для удобства
const testFramework = new SimpleTestFramework();
const test = testFramework.test.bind(testFramework);
const describe = testFramework.describe.bind(testFramework);
const assert = testFramework;
```

### 2. Примеры модульных тестов

```javascript
// math-utils.js - утилиты для математических операций
class MathUtils {
  /**
   * Сложение двух чисел
   * @param {number} a - первое число
   * @param {number} b - второе число
   * @returns {number} - результат сложения
   */
  static add(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Оба аргумента должны быть числами');
    }
    return a + b;
  }
  
  /**
   * Вычитание двух чисел
   * @param {number} a - уменьшаемое
   * @param {number} b - вычитаемое
   * @returns {number} - результат вычитания
   */
  static subtract(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Оба аргумента должны быть числами');
    }
    return a - b;
  }
  
  /**
   * Умножение двух чисел
   * @param {number} a - первый множитель
   * @param {number} b - второй множитель
   * @returns {number} - результат умножения
   */
  static multiply(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Оба аргумента должны быть числами');
    }
    return a * b;
  }
  
  /**
   * Деление двух чисел
   * @param {number} a - делимое
   * @param {number} b - делитель
   * @returns {number} - результат деления
   */
  static divide(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
      throw new Error('Оба аргумента должны быть числами');
    }
    if (b === 0) {
      throw new Error('Деление на ноль');
    }
    return a / b;
  }
  
  /**
   * Проверяет, является ли число четным
   * @param {number} num - число для проверки
   * @returns {boolean} - true если число четное
   */
  static isEven(num) {
    if (typeof num !== 'number') {
      throw new Error('Аргумент должен быть числом');
    }
    return num % 2 === 0;
  }
  
  /**
   * Находит максимальное значение в массиве
   * @param {Array} arr - массив чисел
   * @returns {number} - максимальное значение
   */
  static max(arr) {
    if (!Array.isArray(arr) || arr.length === 0) {
      throw new Error('Аргумент должен быть непустым массивом');
    }
    
    return Math.max(...arr);
  }
  
  /**
   * Находит минимальное значение в массиве
   * @param {Array} arr - массив чисел
   * @returns {number} - минимальное значение
   */
  static min(arr) {
    if (!Array.isArray(arr) || arr.length === 0) {
      throw new Error('Аргумент должен быть непустым массивом');
    }
    
    return Math.min(...arr);
  }
}

// Тесты для MathUtils
describe('MathUtils', () => {
  test('add - корректное сложение', () => {
    assert.assertEqual(MathUtils.add(2, 3), 5);
    assert.assertEqual(MathUtils.add(-1, 1), 0);
    assert.assertEqual(MathUtils.add(0, 0), 0);
  });
  
  test('add - проверка ошибок', () => {
    assert.assertThrows(() => MathUtils.add('a', 2));
    assert.assertThrows(() => MathUtils.add(2, 'b'));
    assert.assertThrows(() => MathUtils.add(null, 2));
  });
  
  test('subtract - корректное вычитание', () => {
    assert.assertEqual(MathUtils.subtract(5, 3), 2);
    assert.assertEqual(MathUtils.subtract(0, 5), -5);
    assert.assertEqual(MathUtils.subtract(-2, -3), 1);
  });
  
  test('multiply - корректное умножение', () => {
    assert.assertEqual(MathUtils.multiply(3, 4), 12);
    assert.assertEqual(MathUtils.multiply(-2, 3), -6);
    assert.assertEqual(MathUtils.multiply(0, 5), 0);
  });
  
  test('divide - корректное деление', () => {
    assert.assertEqual(MathUtils.divide(10, 2), 5);
    assert.assertEqual(MathUtils.divide(7, 2), 3.5);
    assert.assertEqual(MathUtils.divide(-6, 2), -3);
  });
  
  test('divide - проверка деления на ноль', () => {
    assert.assertThrows(() => MathUtils.divide(5, 0));
  });
  
  test('isEven - проверка четности', () => {
    assert.assertTrue(MathUtils.isEven(4));
    assert.assertFalse(MathUtils.isEven(3));
    assert.assertTrue(MathUtils.isEven(0));
    assert.assertTrue(MathUtils.isEven(-2));
    assert.assertFalse(MathUtils.isEven(-3));
  });
  
  test('max - нахождение максимального значения', () => {
    assert.assertEqual(MathUtils.max([1, 5, 3, 9, 2]), 9);
    assert.assertEqual(MathUtils.max([-1, -5, -3]), -1);
    assert.assertEqual(MathUtils.max([42]), 42);
  });
  
  test('max - проверка ошибок', () => {
    assert.assertThrows(() => MathUtils.max([]));
    assert.assertThrows(() => MathUtils.max(null));
    assert.assertThrows(() => MathUtils.max('not array'));
  });
  
  test('min - нахождение минимального значения', () => {
    assert.assertEqual(MathUtils.min([1, 5, 3, 9, 2]), 1);
    assert.assertEqual(MathUtils.min([-1, -5, -3]), -5);
    assert.assertEqual(MathUtils.min([42]), 42);
  });
});
```

## Тестирование асинхронного кода

### 1. Тестирование Promise

```javascript
// async-utils.js - утилиты для асинхронных операций
class AsyncUtils {
  /**
   * Задержка выполнения
   * @param {number} ms - время задержки в миллисекундах
   * @returns {Promise} - промис, который разрешится через указанное время
   */
  static delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  /**
   * Таймаут для асинхронной операции
   * @param {Promise} promise - промис для таймаута
   * @param {number} timeout - время таймаута в миллисекундах
   * @returns {Promise} - промис с таймаутом
   */
  static timeout(promise, timeout) {
    return Promise.race([
      promise,
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Таймаут операции')), timeout)
      )
    ]);
  }
  
  /**
   * Последовательное выполнение асинхронных операций
   * @param {Array} promises - массив промисов
   * @returns {Promise} - промис с результатами
   */
  static async sequential(promises) {
    const results = [];
    for (const promise of promises) {
      results.push(await promise);
    }
    return results;
  }
  
  /**
   * Параллельное выполнение асинхронных операций с ограничением
   * @param {Array} promises - массив функций, возвращающих промисы
   * @param {number} concurrency - максимальное количество параллельных операций
   * @returns {Promise} - промис с результатами
   */
  static async parallelLimited(promises, concurrency) {
    const results = [];
    
    for (let i = 0; i < promises.length; i += concurrency) {
      const batch = promises.slice(i, i + concurrency);
      const batchResults = await Promise.all(batch.map(fn => fn()));
      results.push(...batchResults);
    }
    
    return results;
  }
  
  /**
   * Повторная попытка выполнения асинхронной операции
   * @param {function} asyncFn - асинхронная функция
   * @param {number} maxRetries - максимальное количество попыток
   * @param {number} delay - задержка между попытками
   * @returns {*} - результат выполнения
   */
  static async retry(asyncFn, maxRetries = 3, delay = 1000) {
    let lastError;
    
    for (let i = 0; i <= maxRetries; i++) {
      try {
        return await asyncFn();
      } catch (error) {
        lastError = error;
        if (i < maxRetries) {
          await this.delay(delay);
        }
      }
    }
    
    throw lastError;
  }
}

// Тесты для асинхронных утилит
describe('AsyncUtils', () => {
  test('delay - корректная задержка', async () => {
    const start = Date.now();
    await AsyncUtils.delay(100);
    const end = Date.now();
    
    // Проверяем, что задержка была не менее 100мс
    assert.assertTrue(end - start >= 95, `Задержка должна быть не менее 100мс, получено ${end - start}мс`);
  });
  
  test('timeout - таймаут операции', async () => {
    const slowPromise = AsyncUtils.delay(200).then(() => 'result');
    
    try {
      await AsyncUtils.timeout(slowPromise, 100);
      assert.assertTrue(false, 'Операция не должна была завершиться успешно');
    } catch (error) {
      assert.assertEqual(error.message, 'Таймаут операции');
    }
  });
  
  test('timeout - успешная операция до таймаута', async () => {
    const fastPromise = Promise.resolve('fast result');
    const result = await AsyncUtils.timeout(fastPromise, 100);
    assert.assertEqual(result, 'fast result');
  });
  
  test('sequential - последовательное выполнение', async () => {
    const promises = [
      AsyncUtils.delay(10).then(() => 1),
      AsyncUtils.delay(5).then(() => 2),
      AsyncUtils.delay(1).then(() => 3)
    ];
    
    const start = Date.now();
    const results = await AsyncUtils.sequential(promises);
    const end = Date.now();
    
    assert.assertEqual(results, [1, 2, 3]);
    // Проверяем, что операции выполнялись последовательно (суммарное время > 16мс)
    assert.assertTrue(end - start >= 15, 'Операции должны выполняться последовательно');
  });
  
  test('retry - успешная операция с повторением', async () => {
    let callCount = 0;
    const failingThenSucceeding = async () => {
      callCount++;
      if (callCount < 3) {
        throw new Error('Попытка не удалась');
      }
      return 'успех';
    };
    
    const result = await AsyncUtils.retry(failingThenSucceeding, 5, 10);
    assert.assertEqual(result, 'успех');
    assert.assertEqual(callCount, 3);
  });
  
  test('retry - превышение количества попыток', async () => {
    const alwaysFailing = async () => {
      throw new Error('Всегда неудача');
    };
    
    try {
      await AsyncUtils.retry(alwaysFailing, 2, 10);
      assert.assertTrue(false, 'Операция не должна была завершиться успешно');
    } catch (error) {
      assert.assertEqual(error.message, 'Всегда неудача');
    }
  });
});
```

### 2. Тестирование с использованием моков

```javascript
// api-service.js - сервис для работы с API
class ApiService {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }
  
  /**
   * Получает данные пользователя
   * @param {number} userId - ID пользователя
   * @returns {Promise} - промис с данными пользователя
   */
  async getUser(userId) {
    if (!userId) {
      throw new Error('ID пользователя обязателен');
    }
    
    const response = await fetch(`${this.baseURL}/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }
    
    return await response.json();
  }
  
  /**
   * Создает нового пользователя
   * @param {object} userData - данные пользователя
   * @returns {Promise} - промис с созданным пользователем
   */
  async createUser(userData) {
    if (!userData || !userData.name || !userData.email) {
      throw new Error('Данные пользователя обязательны и должны содержать name и email');
    }
    
    const response = await fetch(`${this.baseURL}/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }
    
    return await response.json();
  }
  
  /**
   * Обновляет данные пользователя
   * @param {number} userId - ID пользователя
   * @param {object} userData - новые данные пользователя
   * @returns {Promise} - промис с обновленным пользователем
   */
  async updateUser(userId, userData) {
    if (!userId) {
      throw new Error('ID пользователя обязателен');
    }
    
    if (!userData) {
      throw new Error('Данные пользователя обязательны');
    }
    
    const response = await fetch(`${this.baseURL}/users/${userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }
    
    return await response.json();
  }
  
  /**
   * Удаляет пользователя
   * @param {number} userId - ID пользователя
   * @returns {Promise} - промис подтверждения удаления
   */
  async deleteUser(userId) {
    if (!userId) {
      throw new Error('ID пользователя обязателен');
    }
    
    const response = await fetch(`${this.baseURL}/users/${userId}`, {
      method: 'DELETE'
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }
    
    return { success: true, userId };
  }
}

// Мок для fetch
class MockFetch {
  constructor() {
    this.responses = new Map();
    this.calls = [];
  }
  
  /**
   * Устанавливает ответ для конкретного URL
   * @param {string} url - URL для мока
   * @param {object} response - объект ответа
   */
  mockResponse(url, response) {
    this.responses.set(url, response);
  }
  
  /**
   * Мокирует fetch
   */
  mock() {
    // Сохраняем оригинальный fetch
    this.originalFetch = window.fetch;
    
    // Заменяем fetch на мок
    window.fetch = async (url, options = {}) => {
      this.calls.push({ url, options });
      
      // Ищем мок-ответ
      const response = this.responses.get(url);
      
      if (response) {
        return {
          ok: response.ok !== false,
          status: response.status || 200,
          json: async () => response.data || {},
          text: async () => JSON.stringify(response.data || {})
        };
      }
      
      // Если мок не найден, возвращаем ошибку
      return {
        ok: false,
        status: 404,
        json: async () => ({ error: 'Mock response not found' }),
        text: async () => 'Mock response not found'
      };
    };
  }
  
  /**
   * Восстанавливает оригинальный fetch
   */
  restore() {
    if (this.originalFetch) {
      window.fetch = this.originalFetch;
    }
  }
  
  /**
   * Получает список вызовов fetch
   * @returns {Array} - массив вызовов
   */
  getCalls() {
    return [...this.calls];
  }
  
  /**
   * Очищает моки и вызовы
   */
  clear() {
    this.responses.clear();
    this.calls = [];
  }
}

// Тесты для ApiService с моками
describe('ApiService', () => {
  let mockFetch;
  let apiService;
  
  beforeEach(() => {
    mockFetch = new MockFetch();
    mockFetch.mock();
    apiService = new ApiService('https://api.example.com');
  });
  
  afterEach(() => {
    mockFetch.restore();
    mockFetch.clear();
  });
  
  test('getUser - успешное получение пользователя', async () => {
    mockFetch.mockResponse('https://api.example.com/users/123', {
      data: { id: 123, name: 'John Doe', email: 'john@example.com' }
    });
    
    const user = await apiService.getUser(123);
    
    assert.assertEqual(user.id, 123);
    assert.assertEqual(user.name, 'John Doe');
    assert.assertEqual(user.email, 'john@example.com');
    
    // Проверяем, что был сделан правильный вызов
    const calls = mockFetch.getCalls();
    assert.assertEqual(calls.length, 1);
    assert.assertEqual(calls[0].url, 'https://api.example.com/users/123');
    assert.assertEqual(calls[0].options.method, 'GET');
  });
  
  test('getUser - ошибка при отсутствии ID', async () => {
    try {
      await apiService.getUser();
      assert.assertTrue(false, 'Должна быть выброшена ошибка');
    } catch (error) {
      assert.assertEqual(error.message, 'ID пользователя обязателен');
    }
  });
  
  test('getUser - ошибка HTTP', async () => {
    mockFetch.mockResponse('https://api.example.com/users/999', {
      ok: false,
      status: 404
    });
    
    try {
      await apiService.getUser(999);
      assert.assertTrue(false, 'Должна быть выброшена ошибка');
    } catch (error) {
      assert.assertEqual(error.message, 'HTTP ошибка: 404');
    }
  });
  
  test('createUser - успешное создание пользователя', async () => {
    const userData = { name: 'Jane Doe', email: 'jane@example.com' };
    mockFetch.mockResponse('https://api.example.com/users', {
      data: { id: 456, ...userData, createdAt: '2023-01-01' }
    });
    
    const user = await apiService.createUser(userData);
    
    assert.assertEqual(user.name, 'Jane Doe');
    assert.assertEqual(user.email, 'jane@example.com');
    assert.assertTrue(user.id === 456);
    
    // Проверяем вызов
    const calls = mockFetch.getCalls();
    assert.assertEqual(calls.length, 1);
    assert.assertEqual(calls[0].url, 'https://api.example.com/users');
    assert.assertEqual(calls[0].options.method, 'POST');
    assert.assertEqual(JSON.parse(calls[0].options.body), userData);
  });
  
  test('createUser - валидация данных', async () => {
    try {
      await apiService.createUser({});
      assert.assertTrue(false, 'Должна быть выброшена ошибка');
    } catch (error) {
      assert.assertEqual(error.message, 'Данные пользователя обязательны и должны содержать name и email');
    }
    
    try {
      await apiService.createUser({ name: 'John' });
      assert.assertTrue(false, 'Должна быть выброшена ошибка');
    } catch (error) {
      assert.assertEqual(error.message, 'Данные пользователя обязательны и должны содержать name и email');
    }
  });
});
```

## Тестирование DOM элементов

### 1. Тестирование пользовательского интерфейса

```javascript
// ui-components.js - компоненты пользовательского интерфейса
class UIComponents {
  /**
   * Создает кнопку
   * @param {object} options - опции кнопки
   * @returns {HTMLElement} - элемент кнопки
   */
  static createButton(options = {}) {
    const button = document.createElement('button');
    button.className = options.className || 'btn';
    button.textContent = options.text || 'Кнопка';
    button.disabled = !!options.disabled;
    
    if (options.onClick) {
      button.addEventListener('click', options.onClick);
    }
    
    return button;
  }
  
  /**
   * Создает форму
   * @param {object} options - опции формы
   * @returns {HTMLElement} - элемент формы
   */
  static createForm(options = {}) {
    const form = document.createElement('form');
    form.className = options.className || 'form';
    
    if (options.onSubmit) {
      form.addEventListener('submit', options.onSubmit);
    }
    
    // Добавляем поля формы
    if (options.fields) {
      options.fields.forEach(field => {
        const fieldElement = document.createElement('div');
        fieldElement.className = 'form-field';
        
        const label = document.createElement('label');
        label.textContent = field.label || field.name;
        label.setAttribute('for', field.name);
        
        let input;
        switch (field.type) {
          case 'textarea':
            input = document.createElement('textarea');
            break;
          case 'select':
            input = document.createElement('select');
            if (field.options) {
              field.options.forEach(option => {
                const optionElement = document.createElement('option');
                optionElement.value = option.value;
                optionElement.textContent = option.text;
                input.appendChild(optionElement);
              });
            }
            break;
          default:
            input = document.createElement('input');
            input.type = field.type || 'text';
        }
        
        input.id = field.name;
        input.name = field.name;
        input.required = !!field.required;
        input.placeholder = field.placeholder;
        
        if (field.value) {
          input.value = field.value;
        }
        
        fieldElement.appendChild(label);
        fieldElement.appendChild(input);
        form.appendChild(fieldElement);
      });
    }
    
    return form;
  }
  
  /**
   * Создает модальное окно
   * @param {object} options - опции модального окна
   * @returns {object} - объект с элементами модального окна
   */
  static createModal(options = {}) {
    const modal = document.createElement('div');
    modal.className = 'modal';
    modal.style.display = 'none';
    
    const modalContent = document.createElement('div');
    modalContent.className = 'modal-content';
    
    const modalHeader = document.createElement('div');
    modalHeader.className = 'modal-header';
    modalHeader.innerHTML = `
      <h2>${options.title || 'Модальное окно'}</h2>
      <span class="close">&times;</span>
    `;
    
    const modalBody = document.createElement('div');
    modalBody.className = 'modal-body';
    modalBody.innerHTML = options.content || '<p>Содержимое модального окна</p>';
    
    const modalFooter = document.createElement('div');
    modalFooter.className = 'modal-footer';
    
    // Добавляем кнопки
    if (options.buttons) {
      options.buttons.forEach(buttonOptions => {
        const button = UIComponents.createButton(buttonOptions);
        modalFooter.appendChild(button);
      });
    }
    
    modalContent.appendChild(modalHeader);
    modalContent.appendChild(modalBody);
    modalContent.appendChild(modalFooter);
    modal.appendChild(modalContent);
    
    // Обработчик закрытия
    const closeBtn = modalHeader.querySelector('.close');
    closeBtn.addEventListener('click', () => {
      modal.style.display = 'none';
      if (options.onClose) {
        options.onClose();
      }
    });
    
    // Закрытие при клике вне модального окна
    window.addEventListener('click', (event) => {
      if (event.target === modal) {
        modal.style.display = 'none';
        if (options.onClose) {
          options.onClose();
        }
      }
    });
    
    return {
      modal,
      show: () => {
        modal.style.display = 'block';
        if (options.onShow) {
          options.onShow();
        }
      },
      hide: () => {
        modal.style.display = 'none';
        if (options.onHide) {
          options.onHide();
        }
      }
    };
  }
}

// Мок DOM для тестирования
class MockDOM {
  constructor() {
    this.elements = new Map();
    this.eventListeners = new Map();
  }
  
  /**
   * Создает мок-элемент
   * @param {string} tagName - тег элемента
   * @param {object} attributes - атрибуты элемента
   * @returns {object} - мок-элемент
   */
  createElement(tagName, attributes = {}) {
    const element = {
      tagName: tagName.toUpperCase(),
      attributes: { ...attributes },
      children: [],
      textContent: '',
      innerHTML: '',
      style: {},
      className: '',
      disabled: false,
      value: '',
      required: false,
      type: attributes.type || 'div',
      
      // Методы
      addEventListener: (event, handler) => {
        if (!this.eventListeners.has(element)) {
          this.eventListeners.set(element, new Map());
        }
        const handlers = this.eventListeners.get(element);
        if (!handlers.has(event)) {
          handlers.set(event, []);
        }
        handlers.get(event).push(handler);
      },
      
      removeEventListener: (event, handler) => {
        if (this.eventListeners.has(element)) {
          const handlers = this.eventListeners.get(element);
          if (handlers.has(event)) {
            const eventHandlers = handlers.get(event);
            const index = eventHandlers.indexOf(handler);
            if (index > -1) {
              eventHandlers.splice(index, 1);
            }
          }
        }
      },
      
      dispatchEvent: (event) => {
        if (this.eventListeners.has(element)) {
          const handlers = this.eventListeners.get(element);
          const eventHandlers = handlers.get(event.type);
          if (eventHandlers) {
            eventHandlers.forEach(handler => handler(event));
          }
        }
      },
      
      appendChild: (child) => {
        element.children.push(child);
      },
      
      setAttribute: (name, value) => {
        element.attributes[name] = value;
      },
      
      getAttribute: (name) => {
        return element.attributes[name];
      }
    };
    
    return element;
  }
  
  /**
   * Мокирует DOM API
   */
  mock() {
    // Сохраняем оригинальные методы
    this.originalDocument = {
      createElement: document.createElement,
      addEventListener: document.addEventListener
    };
    
    // Заменяем методы на моки
    document.createElement = this.createElement.bind(this);
    document.addEventListener = (event, handler) => {
      if (!this.eventListeners.has(document)) {
        this.eventListeners.set(document, new Map());
      }
      const handlers = this.eventListeners.get(document);
      if (!handlers.has(event)) {
        handlers.set(event, []);
      }
      handlers.get(event).push(handler);
    };
  }
  
  /**
   * Восстанавливает оригинальные DOM методы
   */
  restore() {
    if (this.originalDocument) {
      document.createElement = this.originalDocument.createElement;
      document.addEventListener = this.originalDocument.addEventListener;
    }
  }
}

// Тесты для UI компонентов
describe('UIComponents', () => {
  let mockDOM;
  
  beforeEach(() => {
    mockDOM = new MockDOM();
    mockDOM.mock();
  });
  
  afterEach(() => {
    mockDOM.restore();
  });
  
  test('createButton - создание кнопки с опциями', () => {
    let clicked = false;
    const onClick = () => { clicked = true; };
    
    const button = UIComponents.createButton({
      text: 'Нажми меня',
      className: 'my-button',
      disabled: true,
      onClick: onClick
    });
    
    assert.assertEqual(button.textContent, 'Нажми меня');
    assert.assertEqual(button.className, 'my-button');
    assert.assertTrue(button.disabled);
    
    // Симулируем клик
    button.dispatchEvent({ type: 'click' });
    assert.assertTrue(clicked);
  });
  
  test('createForm - создание формы с полями', () => {
    const submitHandler = jest.fn(); // Предполагаем, что jest доступен для mock-функций
    
    const form = UIComponents.createForm({
      className: 'my-form',
      onSubmit: submitHandler,
      fields: [
        { name: 'name', label: 'Имя', type: 'text', required: true },
        { name: 'email', label: 'Email', type: 'email', required: true },
        { name: 'message', label: 'Сообщение', type: 'textarea' }
      ]
    });
    
    assert.assertEqual(form.className, 'my-form');
    
    // Проверяем, что поля были созданы
    const fieldDivs = form.children.filter(child => child.className === 'form-field');
    assert.assertEqual(fieldDivs.length, 3);
    
    // Проверяем первое поле
    const firstField = fieldDivs[0];
    const firstInput = firstField.children.find(child => child.tagName === 'INPUT');
    assert.assertEqual(firstInput.name, 'name');
    assert.assertTrue(firstInput.required);
  });
  
  test('createModal - создание модального окна', () => {
    let shown = false;
    let hidden = false;
    
    const modalObj = UIComponents.createModal({
      title: 'Тестовое модальное окно',
      content: '<p>Тестовый контент</p>',
      onShow: () => { shown = true; },
      onHide: () => { hidden = true; },
      buttons: [
        { text: 'ОК', onClick: () => modalObj.hide() },
        { text: 'Отмена', className: 'btn-cancel' }
      ]
    });
    
    // Проверяем, что элементы созданы
    assert.assertTrue(modalObj.modal.tagName === 'DIV');
    assert.assertTrue(modalObj.modal.className.includes('modal'));
    
    // Проверяем отображение
    modalObj.show();
    assert.assertTrue(shown);
    assert.assertEqual(modalObj.modal.style.display, 'block');
    
    // Проверяем скрытие
    modalObj.hide();
    assert.assertTrue(hidden);
    assert.assertEqual(modalObj.modal.style.display, 'none');
  });
});
```

## Интеграционное тестирование

### 1. Тестирование взаимодействия компонентов

```javascript
// user-profile-manager.js - менеджер профиля пользователя
class UserProfileManager {
  constructor(apiService) {
    this.apiService = apiService;
    this.currentUser = null;
    this.listeners = [];
  }
  
  /**
   * Аутентифицирует пользователя
   * @param {string} email - email пользователя
   * @param {string} password - пароль пользователя
   * @returns {Promise} - промис с результатом аутентификации
   */
  async authenticate(email, password) {
    if (!email || !password) {
      throw new Error('Email и пароль обязательны');
    }
    
    // В реальном приложении здесь будет вызов API аутентификации
    // Для тестирования используем мок
    this.currentUser = {
      id: 123,
      email: email,
      name: 'Test User',
      role: 'user'
    };
    
    this.notifyListeners('userAuthenticated', this.currentUser);
    return this.currentUser;
  }
  
  /**
   * Получает профиль текущего пользователя
   * @returns {object} - профиль пользователя
   */
  getCurrentUser() {
    return this.currentUser;
  }
  
  /**
   * Обновляет профиль пользователя
   * @param {object} profileData - новые данные профиля
   * @returns {Promise} - промис с обновленным профилем
   */
  async updateProfile(profileData) {
    if (!this.currentUser) {
      throw new Error('Пользователь не аутентифицирован');
    }
    
    if (!profileData) {
      throw new Error('Данные профиля обязательны');
    }
    
    // Обновляем локальные данные
    Object.assign(this.currentUser, profileData);
    
    // В реальном приложении отправляем на сервер
    // const updatedUser = await this.apiService.updateUser(this.currentUser.id, profileData);
    // this.currentUser = updatedUser;
    
    this.notifyListeners('profileUpdated', this.currentUser);
    return this.currentUser;
  }
  
  /**
   * Выход из системы
   */
  logout() {
    this.currentUser = null;
    this.notifyListeners('userLoggedOut');
  }
  
  /**
   * Подписывается на события
   * @param {string} event - тип события
   * @param {function} callback - функция обратного вызова
   */
  on(event, callback) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(callback);
  }
  
  /**
   * Отписывается от событий
   * @param {string} event - тип события
   * @param {function} callback - функция обратного вызова
   */
  off(event, callback) {
    if (this.listeners[event]) {
      this.listeners[event] = this.listeners[event].filter(cb => cb !== callback);
    }
  }
  
  /**
   * Уведомляет подписчиков о событии
   * @param {string} event - тип события
   * @param {*} data - данные события
   */
  notifyListeners(event, data) {
    if (this.listeners[event]) {
      this.listeners[event].forEach(callback => {
        try {
          callback(data);
        } catch (error) {
          console.error(`Ошибка в обработчике события ${event}:`, error);
        }
      });
    }
  }
}

// user-profile-component.js - компонент профиля пользователя
class UserProfileComponent {
  constructor(container, profileManager) {
    this.container = container;
    this.profileManager = profileManager;
    this.isRendered = false;
  }
  
  /**
   * Рендерит компонент
   */
  render() {
    const user = this.profileManager.getCurrentUser();
    
    if (user) {
      this.container.innerHTML = `
        <div class="user-profile">
          <h2>Профиль пользователя</h2>
          <div class="user-info">
            <p><strong>Имя:</strong> <span id="user-name">${user.name}</span></p>
            <p><strong>Email:</strong> <span id="user-email">${user.email}</span></p>
            <p><strong>Роль:</strong> <span id="user-role">${user.role}</span></p>
          </div>
          <button id="edit-profile-btn">Редактировать профиль</button>
          <button id="logout-btn">Выйти</button>
        </div>
      `;
      
      this.attachEventListeners();
    } else {
      this.container.innerHTML = `
        <div class="login-form">
          <h2>Вход в систему</h2>
          <form id="login-form">
            <div class="form-group">
              <label for="email">Email:</label>
              <input type="email" id="email" name="email" required>
            </div>
            <div class="form-group">
              <label for="password">Пароль:</label>
              <input type="password" id="password" name="password" required>
            </div>
            <button type="submit">Войти</button>
          </form>
        </div>
      `;
      
      this.attachLoginEventListeners();
    }
    
    this.isRendered = true;
  }
  
  /**
   * Привязывает обработчики событий
   */
  attachEventListeners() {
    document.getElementById('edit-profile-btn').addEventListener('click', () => {
      this.showEditForm();
    });
    
    document.getElementById('logout-btn').addEventListener('click', () => {
      this.profileManager.logout();
    });
    
    // Подписываемся на события менеджера
    this.profileManager.on('profileUpdated', () => {
      this.render();
    });
    
    this.profileManager.on('userLoggedOut', () => {
      this.render();
    });
  }
  
  /**
   * Привязывает обработчики событий формы входа
   */
  attachLoginEventListeners() {
    document.getElementById('login-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const email = document.getElementById('email').value;
      const password = document.getElementById('password').value;
      
      try {
        await this.profileManager.authenticate(email, password);
        this.render();
      } catch (error) {
        alert('Ошибка аутентификации: ' + error.message);
      }
    });
  }
  
  /**
   * Показывает форму редактирования профиля
   */
  showEditForm() {
    const user = this.profileManager.getCurrentUser();
    
    this.container.innerHTML = `
      <div class="edit-profile-form">
        <h2>Редактировать профиль</h2>
        <form id="edit-profile-form">
          <div class="form-group">
            <label for="edit-name">Имя:</label>
            <input type="text" id="edit-name" name="name" value="${user.name}" required>
          </div>
          <div class="form-group">
            <label for="edit-email">Email:</label>
            <input type="email" id="edit-email" name="email" value="${user.email}" required>
          </div>
          <button type="submit">Сохранить</button>
          <button type="button" id="cancel-edit">Отмена</button>
        </form>
      </div>
    `;
    
    this.attachEditEventListeners();
  }
  
  /**
   * Привязывает обработчики событий формы редактирования
   */
  attachEditEventListeners() {
    document.getElementById('edit-profile-form').addEventListener('submit', async (e) => {
      e.preventDefault();
      
      const name = document.getElementById('edit-name').value;
      const email = document.getElementById('edit-email').value;
      
      try {
        await this.profileManager.updateProfile({ name, email });
      } catch (error) {
        alert('Ошибка обновления профиля: ' + error.message);
      }
    });
    
    document.getElementById('cancel-edit').addEventListener('click', () => {
      this.render();
    });
  }
}

// Интеграционные тесты для UserProfileManager и UserProfileComponent
describe('UserProfile Integration', () => {
  let mockApiService;
  let profileManager;
  let container;
  let component;
  
  beforeEach(() => {
    // Создаем мок API сервиса
    mockApiService = {
      getUser: jest.fn(),
      updateUser: jest.fn(),
      createUser: jest.fn()
    };
    
    profileManager = new UserProfileManager(mockApiService);
    container = document.createElement('div');
    component = new UserProfileComponent(container, profileManager);
  });
  
  test('аутентификация пользователя и отображение профиля', async () => {
    // Рендерим компонент (должна отобразиться форма входа)
    component.render();
    assert.assertTrue(container.innerHTML.includes('Вход в систему'));
    
    // Аутентифицируем пользователя
    await profileManager.authenticate('test@example.com', 'password123');
    
    // Обновляем компонент
    component.render();
    
    // Проверяем, что отображается профиль пользователя
    assert.assertTrue(container.innerHTML.includes('Профиль пользователя'));
    assert.assertTrue(container.innerHTML.includes('Test User'));
    assert.assertTrue(container.innerHTML.includes('test@example.com'));
  });
  
  test('редактирование профиля пользователя', async () => {
    // Аутентифицируем пользователя
    await profileManager.authenticate('test@example.com', 'password123');
    
    // Рендерим компонент
    component.render();
    
    // Нажимаем кнопку редактирования
    const editButton = container.querySelector('#edit-profile-btn');
    editButton.click();
    
    // Проверяем, что отобразилась форма редактирования
    assert.assertTrue(container.innerHTML.includes('Редактировать профиль'));
    
    // Обновляем профиль
    await profileManager.updateProfile({ name: 'Updated Name', email: 'updated@example.com' });
    
    // Рендерим компонент снова
    component.render();
    
    // Проверяем, что профиль обновлен
    assert.assertTrue(container.innerHTML.includes('Updated Name'));
    assert.assertTrue(container.innerHTML.includes('updated@example.com'));
  });
  
  test('выход из системы', async () => {
    // Аутентифицируем пользователя
    await profileManager.authenticate('test@example.com', 'password123');
    
    // Рендерим компонент
    component.render();
    
    // Проверяем, что пользователь аутентифицирован
    assert.assertTrue(container.innerHTML.includes('Профиль пользователя'));
    
    // Выходим из системы
    profileManager.logout();
    
    // Рендерим компонент
    component.render();
    
    // Проверяем, что отобразилась форма входа
    assert.assertTrue(container.innerHTML.includes('Вход в систему'));
  });
  
  test('обработка ошибок при аутентификации', async () => {
    // Пытаемся аутентифицироваться без данных
    try {
      await profileManager.authenticate();
      assert.assertTrue(false, 'Должна быть выброшена ошибка');
    } catch (error) {
      assert.assertEqual(error.message, 'Email и пароль обязательны');
    }
    
    // Рендерим компонент
    component.render();
    
    // Проверяем, что все равно отображается форма входа
    assert.assertTrue(container.innerHTML.includes('Вход в систему'));
  });
});
```

## Тестирование производительности

### 1. Измерение производительности

```javascript
// performance-tester.js - инструмент для тестирования производительности
class PerformanceTester {
  constructor() {
    this.metrics = new Map();
  }
  
  /**
   * Измеряет время выполнения функции
   * @param {function} fn - функция для измерения
   * @param {string} name - имя метрики
   * @param {number} iterations - количество итераций
   * @returns {object} - результаты измерения
   */
  measureFunction(fn, name, iterations = 1) {
    const times = [];
    
    for (let i = 0; i < iterations; i++) {
      const start = performance.now();
      fn();
      const end = performance.now();
      times.push(end - start);
    }
    
    const avgTime = times.reduce((sum, time) => sum + time, 0) / times.length;
    const minTime = Math.min(...times);
    const maxTime = Math.max(...times);
    
    const metric = {
      name,
      iterations,
      times,
      avgTime,
      minTime,
      maxTime,
      timestamp: Date.now()
    };
    
    this.metrics.set(name, metric);
    return metric;
  }
  
  /**
   * Измеряет производительность асинхронной функции
   * @param {function} asyncFn - асинхронная функция для измерения
   * @param {string} name - имя метрики
   * @param {number} iterations - количество итераций
   * @returns {object} - результаты измерения
   */
  async measureAsyncFunction(asyncFn, name, iterations = 1) {
    const times = [];
    
    for (let i = 0; i < iterations; i++) {
      const start = performance.now();
      await asyncFn();
      const end = performance.now();
      times.push(end - start);
    }
    
    const avgTime = times.reduce((sum, time) => sum + time, 0) / times.length;
    const minTime = Math.min(...times);
    const maxTime = Math.max(...times);
    
    const metric = {
      name,
      iterations,
      times,
      avgTime,
      minTime,
      maxTime,
      timestamp: Date.now()
    };
    
    this.metrics.set(name, metric);
    return metric;
  }
  
  /**
   * Сравнивает производительность двух функций
   * @param {function} fn1 - первая функция
   * @param {function} fn2 - вторая функция
   * @param {string} name1 - имя первой функции
   * @param {string} name2 - имя второй функции
   * @param {number} iterations - количество итераций
   * @returns {object} - результаты сравнения
   */
  compareFunctions(fn1, fn2, name1, name2, iterations = 100) {
    const metric1 = this.measureFunction(fn1, name1, iterations);
    const metric2 = this.measureFunction(fn2, name2, iterations);
    
    const comparison = {
      [name1]: metric1,
      [name2]: metric2,
      winner: metric1.avgTime < metric2.avgTime ? name1 : name2,
      performanceRatio: Math.min(metric1.avgTime, metric2.avgTime) / 
                       Math.max(metric1.avgTime, metric2.avgTime)
    };
    
    return comparison;
  }
  
  /**
   * Проверяет, укладывается ли функция в заданный лимит времени
   * @param {function} fn - функция для проверки
   * @param {number} timeLimit - лимит времени в миллисекундах
   * @param {string} name - имя метрики
   * @returns {boolean} - true если укладывается в лимит
   */
  checkTimeLimit(fn, timeLimit, name) {
    const metric = this.measureFunction(fn, name, 1);
    return metric.avgTime <= timeLimit;
  }
  
  /**
   * Получает все метрики
   * @returns {Map} - карта метрик
   */
  getMetrics() {
    return new Map(this.metrics);
  }
  
  /**
   * Очищает метрики
   */
  clearMetrics() {
    this.metrics.clear();
  }
  
  /**
   * Выводит отчет о производительности
   */
  printReport() {
    console.log('=== Отчет о производительности ===');
    
    for (const [name, metric] of this.metrics) {
      console.log(`\n${name}:`);
      console.log(`  Среднее время: ${metric.avgTime.toFixed(4)}ms`);
      console.log(`  Минимальное время: ${metric.minTime.toFixed(4)}ms`);
      console.log(`  Максимальное время: ${metric.maxTime.toFixed(4)}ms`);
      console.log(`  Итераций: ${metric.iterations}`);
    }
  }
}

// Примеры функций для тестирования производительности
function slowFunction() {
  let sum = 0;
  for (let i = 0; i < 1000000; i++) {
    sum += i;
  }
  return sum;
}

function fastFunction() {
  return (1000000 * 999999) / 2; // Формула суммы арифметической прогрессии
}

function arrayMapVsFor() {
  const arr = new Array(10000).fill(0).map((_, i) => i);
  
  // Использование map
  const result1 = arr.map(x => x * 2);
  
  // Использование for loop
  const result2 = new Array(arr.length);
  for (let i = 0; i < arr.length; i++) {
    result2[i] = arr[i] * 2;
  }
  
  return { result1, result2 };
}

// Тестирование производительности
describe('Performance Tests', () => {
  let performanceTester;
  
  beforeEach(() => {
    performanceTester = new PerformanceTester();
  });
  
  test('сравнение медленной и быстрой функций', () => {
    const comparison = performanceTester.compareFunctions(
      slowFunction,
      fastFunction,
      'slowFunction',
      'fastFunction',
      100
    );
    
    console.log('Сравнение функций:');
    console.log(`Победитель: ${comparison.winner}`);
    console.log(`Соотношение производительности: ${(comparison.performanceRatio * 100).toFixed(2)}%`);
    
    // Проверяем, что быстрая функция действительно быстрее
    assert.assertTrue(
      comparison.performanceRatio < 0.5,
      'Быстрая функция должна быть значительно быстрее'
    );
  });
  
  test('проверка лимита времени', () => {
    const withinLimit = performanceTester.checkTimeLimit(
      slowFunction,
      100, // 100ms лимит
      'slowFunctionTimeCheck'
    );
    
    // Для этой функции лимит в 100ms может быть превышен
    console.log(`Функция укладывается в 100ms лимит: ${withinLimit}`);
  });
  
  test('сравнение map vs for loop', () => {
    const comparison = performanceTester.compareFunctions(
      () => arrayMapVsFor().result1,
      () => arrayMapVsFor().result2,
      'Array.map',
      'For loop',
      10
    );
    
    console.log('Сравнение map vs for loop:');
    console.log(`Победитель: ${comparison.winner}`);
    console.log(`Соотношение: ${(comparison.performanceRatio * 100).toFixed(2)}%`);
  });
  
  afterEach(() => {
    performanceTester.printReport();
    performanceTester.clearMetrics();
  });
});
```

## Практические советы

- Пишите тесты до реализации функционала (TDD)
- Тестируйте как положительные, так и отрицательные сценарии
- Используйте моки для изоляции тестируемых компонентов
- Покрывайте тестами критические пути приложения
- Регулярно запускайте тесты в CI/CD процессе
- Используйте различные уровни тестирования (модульные, интеграционные, сквозные)
- Поддерживайте чистоту тестовой среды между запусками

## Связанные темы

- [[test-driven-development]]
- [[testing-strategies]]
- [[mocking-techniques]]
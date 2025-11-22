---
aliases: ["Сниппеты валидации", "JS Validation Snippets"]
tags: [javascript, validation, frontend, snippets]
---

# Сниппеты для валидации разных типов данных

Набор готовых сниппетов для валидации различных типов данных в JavaScript. Эти сниппеты можно использовать в формах, API валидациях и других сценариях проверки данных.

## Валидация строк

### 1. Проверка на пустоту

```javascript
/**
 * Проверяет, является ли строка пустой или содержит только пробелы
 * @param {string} str - строка для проверки
 * @returns {boolean} - true если строка пуста или содержит только пробелы
 */
function isEmpty(str) {
  return !str || str.trim().length === 0;
}

// Пример использования
console.log(isEmpty('')); // true
console.log(isEmpty('   ')); // true
console.log(isEmpty('hello')); // false
```

### 2. Проверка длины строки

```javascript
/**
 * Проверяет длину строки в заданном диапазоне
 * @param {string} str - строка для проверки
 * @param {number} minLength - минимальная длина
 * @param {number} maxLength - максимальная длина
 * @returns {boolean} - true если длина в диапазоне
 */
function validateStringLength(str, minLength = 0, maxLength = Infinity) {
  const length = str ? str.length : 0;
  return length >= minLength && length <= maxLength;
}

// Пример использования
console.log(validateStringLength('hello', 3, 10)); // true
console.log(validateStringLength('hi', 3, 10)); // false
```

### 3. Проверка формата email

```javascript
/**
 * Проверяет формат email
 * @param {string} email - email для проверки
 * @returns {boolean} - true если формат корректный
 */
function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

// Более строгая проверка email
function validateEmailStrict(email) {
  const emailRegex = /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;
  return emailRegex.test(email);
}

// Пример использования
console.log(validateEmail('user@example.com')); // true
console.log(validateEmail('invalid.email')); // false
```

### 4. Проверка формата телефона

```javascript
/**
 * Проверяет формат телефона (международный)
 * @param {string} phone - телефон для проверки
 * @returns {boolean} - true если формат корректный
 */
function validatePhone(phone) {
  // Удаляем все нецифровые символы
  const cleanPhone = phone.replace(/\D/g, '');
  
  // Проверяем длину (обычно от 10 до 15 цифр)
  return cleanPhone.length >= 10 && cleanPhone.length <= 15;
}

/**
 * Проверяет формат телефона с конкретным шаблоном
 * @param {string} phone - телефон для проверки
 * @param {string} pattern - шаблон (например, '+7 (XXX) XXX-XXXX')
 * @returns {boolean} - true если формат соответствует шаблону
 */
function validatePhoneWithPattern(phone, pattern = '+X (XXX) XXX-XXXX') {
  // Для примера, проверяем российский формат
  const ruPhoneRegex = /^\+7\s\(\d{3}\)\s\d{3}-\d{4}$/;
  return ruPhoneRegex.test(phone);
}

// Пример использования
console.log(validatePhone('+79123456789')); // true
console.log(validatePhoneWithPattern('+7 (912) 345-6789')); // true
```

## Валидация чисел

### 1. Проверка на число

```javascript
/**
 * Проверяет, является ли значение числом
 * @param {*} value - значение для проверки
 * @returns {boolean} - true если значение является числом
 */
function isNumber(value) {
  return !isNaN(parseFloat(value)) && isFinite(value);
}

/**
 * Проверяет, является ли значение целым числом
 * @param {*} value - значение для проверки
 * @returns {boolean} - true если значение является целым числом
 */
function isInteger(value) {
  return Number.isInteger(Number(value));
}

/**
 * Проверяет, является ли значение положительным числом
 * @param {*} value - значение для проверки
 * @returns {boolean} - true если значение является положительным числом
 */
function isPositiveNumber(value) {
  return isNumber(value) && parseFloat(value) > 0;
}

// Пример использования
console.log(isNumber('123')); // true
console.log(isNumber('123.45')); // true
console.log(isNumber('abc')); // false
console.log(isInteger('123')); // true
console.log(isInteger('123.45')); // false
```

### 2. Проверка диапазона чисел

```javascript
/**
 * Проверяет, находится ли число в заданном диапазоне
 * @param {number} value - число для проверки
 * @param {number} min - минимальное значение
 * @param {number} max - максимальное значение
 * @returns {boolean} - true если число в диапазоне
 */
function isInRange(value, min, max) {
  return isNumber(value) && value >= min && value <= max;
}

// Пример использования
console.log(isInRange(5, 1, 10)); // true
console.log(isInRange(15, 1, 10)); // false
```

## Валидация дат

### 1. Проверка формата даты

```javascript
/**
 * Проверяет, является ли строка корректной датой
 * @param {string} dateString - строка даты для проверки
 * @returns {boolean} - true если дата корректна
 */
function isValidDate(dateString) {
  const date = new Date(dateString);
  return date instanceof Date && !isNaN(date.getTime());
}

/**
 * Проверяет дату в формате YYYY-MM-DD
 * @param {string} dateString - строка даты для проверки
 * @returns {boolean} - true если дата в корректном формате
 */
function isValidISODate(dateString) {
  const isoDateRegex = /^\d{4}-\d{2}-\d{2}$/;
  if (!isoDateRegex.test(dateString)) return false;
  
  const date = new Date(dateString);
  return date instanceof Date && !isNaN(date) && 
         date.toISOString().slice(0, 10) === dateString;
}

// Пример использования
console.log(isValidDate('2023-11-13')); // true
console.log(isValidDate('invalid-date')); // false
console.log(isValidISODate('2023-11-13')); // true
console.log(isValidISODate('13-11-2023')); // false
```

### 2. Проверка диапазона дат

```javascript
/**
 * Проверяет, находится ли дата в заданном диапазоне
 * @param {string|Date} date - дата для проверки
 * @param {string|Date} startDate - начальная дата диапазона
 * @param {string|Date} endDate - конечная дата диапазона
 * @returns {boolean} - true если дата в диапазоне
 */
function isDateInRange(date, startDate, endDate) {
  const dateObj = new Date(date);
  const startObj = new Date(startDate);
  const endObj = new Date(endDate);
  
  return dateObj >= startObj && dateObj <= endObj;
}

// Пример использования
console.log(isDateInRange('2023-11-10', '2023-11-01', '2023-11-30')); // true
console.log(isDateInRange('2023-12-10', '2023-11-01', '2023-11-30')); // false
```

## Валидация объектов

### 1. Проверка обязательных полей

```javascript
/**
 * Проверяет, содержит ли объект все обязательные поля
 * @param {object} obj - объект для проверки
 * @param {string[]} requiredFields - массив обязательных полей
 * @returns {boolean} - true если все обязательные поля присутствуют
 */
function hasRequiredFields(obj, requiredFields) {
  if (!obj || typeof obj !== 'object') return false;
  
  return requiredFields.every(field => {
    const value = getNestedValue(obj, field);
    return value !== undefined && value !== null && value !== '';
  });
}

/**
 * Получает значение вложенного свойства объекта по строковому пути
 * @param {object} obj - объект
 * @param {string} path - путь к свойству (например, 'user.profile.name')
 * @returns {*} - значение свойства или undefined
 */
function getNestedValue(obj, path) {
  return path.split('.').reduce((current, key) => {
    return current && current[key] !== undefined ? current[key] : undefined;
  }, obj);
}

// Пример использования
const user = {
  name: 'John',
  email: 'john@example.com',
  profile: {
    age: 30
  }
};

console.log(hasRequiredFields(user, ['name', 'email'])); // true
console.log(hasRequiredFields(user, ['name', 'phone'])); // false
```

### 2. Валидация структуры объекта

```javascript
/**
 * Валидирует объект по заданной схеме
 * @param {object} obj - объект для валидации
 * @param {object} schema - схема валидации
 * @returns {object} - объект с результатами валидации { isValid, errors }
 */
function validateObjectSchema(obj, schema) {
  const errors = [];
  
  for (const [field, rules] of Object.entries(schema)) {
    const value = getNestedValue(obj, field);
    
    // Проверка обязательности
    if (rules.required && (value === undefined || value === null || value === '')) {
      errors.push(`Поле ${field} обязательно для заполнения`);
      continue;
    }
    
    // Пропускаем проверку, если поле не обязательно и пустое
    if (value === undefined || value === null || value === '') {
      continue;
    }
    
    // Проверка типа
    if (rules.type && typeof value !== rules.type) {
      errors.push(`Поле ${field} должно быть типа ${rules.type}`);
    }
    
    // Проверка кастомной функции валидации
    if (rules.validator && typeof rules.validator === 'function') {
      const result = rules.validator(value);
      if (result !== true && typeof result === 'string') {
        errors.push(`${field}: ${result}`);
      }
    }
    
    // Проверка длины строки
    if (rules.minLength && typeof value === 'string' && value.length < rules.minLength) {
      errors.push(`Поле ${field} должно содержать не менее ${rules.minLength} символов`);
    }
    
    if (rules.maxLength && typeof value === 'string' && value.length > rules.maxLength) {
      errors.push(`Поле ${field} должно содержать не более ${rules.maxLength} символов`);
    }
    
    // Проверка числового диапазона
    if (rules.min !== undefined && typeof value === 'number' && value < rules.min) {
      errors.push(`Поле ${field} должно быть не менее ${rules.min}`);
    }
    
    if (rules.max !== undefined && typeof value === 'number' && value > rules.max) {
      errors.push(`Поле ${field} должно быть не более ${rules.max}`);
    }
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
}

// Пример использования
const userSchema = {
  'name': { type: 'string', required: true, minLength: 2, maxLength: 50 },
  'email': { type: 'string', required: true, validator: validateEmail },
  'age': { type: 'number', min: 0, max: 150 }
};

const userData = {
  name: 'John',
  email: 'john@example.com',
  age: 30
};

const validation = validateObjectSchema(userData, userSchema);
console.log(validation); // { isValid: true, errors: [] }
```

## Валидация массивов

### 1. Проверка массива

```javascript
/**
 * Проверяет, является ли значение массивом
 * @param {*} value - значение для проверки
 * @returns {boolean} - true если значение является массивом
 */
function isArray(value) {
  return Array.isArray(value);
}

/**
 * Проверяет массив на соответствие критериям
 * @param {Array} arr - массив для проверки
 * @param {object} options - опции валидации
 * @returns {boolean} - true если массив соответствует критериям
 */
function validateArray(arr, options = {}) {
  if (!isArray(arr)) return false;
  
  // Проверка минимальной длины
  if (options.minLength !== undefined && arr.length < options.minLength) {
    return false;
  }
  
  // Проверка максимальной длины
  if (options.maxLength !== undefined && arr.length > options.maxLength) {
    return false;
  }
  
  // Проверка уникальности элементов
  if (options.unique) {
    const seen = new Set();
    for (const item of arr) {
      if (seen.has(item)) return false;
      seen.add(item);
    }
  }
  
  // Проверка типа элементов
  if (options.elementType) {
    return arr.every(item => typeof item === options.elementType);
  }
  
  return true;
}

// Пример использования
console.log(validateArray([1, 2, 3], { minLength: 2, maxLength: 5 })); // true
console.log(validateArray([1, 2, 2], { unique: true })); // false
console.log(validateArray([1, 2, 3], { elementType: 'number' })); // true
```

## Комплексная валидация формы

### 1. Валидация формы с использованием всех техник

```javascript
/**
 * Комплексная валидация формы
 */
class FormValidator {
  constructor() {
    this.rules = new Map();
    this.customValidators = new Map();
  }
  
  /**
   * Добавляет правило валидации для поля
   * @param {string} field - имя поля
   * @param {object} rules - правила валидации
   */
  addRule(field, rules) {
    this.rules.set(field, rules);
    return this;
  }
  
  /**
   * Добавляет кастомный валидатор
   * @param {string} name - имя валидатора
   * @param {function} validator - функция валидации
   */
  addCustomValidator(name, validator) {
    this.customValidators.set(name, validator);
    return this;
  }
  
  /**
   * Валидирует данные формы
   * @param {object} data - данные формы
   * @returns {object} - результат валидации
   */
  validate(data) {
    const errors = {};
    
    for (const [field, rules] of this.rules) {
      const value = getNestedValue(data, field);
      const fieldErrors = [];
      
      // Проверка обязательности
      if (rules.required && (value === undefined || value === null || value === '')) {
        fieldErrors.push('Это поле обязательно для заполнения');
      }
      
      // Пропускаем другие проверки, если поле пустое и не обязательное
      if ((value === undefined || value === null || value === '') && !rules.required) {
        continue;
      }
      
      // Применение правил валидации
      for (const [ruleName, ruleValue] of Object.entries(rules)) {
        switch (ruleName) {
          case 'required':
            break; // Уже проверили
          
          case 'type':
            if (typeof value !== ruleValue) {
              fieldErrors.push(`Значение должно быть типа ${ruleValue}`);
            }
            break;
            
          case 'minLength':
            if (typeof value === 'string' && value.length < ruleValue) {
              fieldErrors.push(`Минимальная длина: ${ruleValue} символов`);
            }
            break;
            
          case 'maxLength':
            if (typeof value === 'string' && value.length > ruleValue) {
              fieldErrors.push(`Максимальная длина: ${ruleValue} символов`);
            }
            break;
            
          case 'min':
            if (typeof value === 'number' && value < ruleValue) {
              fieldErrors.push(`Минимальное значение: ${ruleValue}`);
            }
            break;
            
          case 'max':
            if (typeof value === 'number' && value > ruleValue) {
              fieldErrors.push(`Максимальное значение: ${ruleValue}`);
            }
            break;
            
          case 'pattern':
            if (typeof value === 'string' && !ruleValue.test(value)) {
              fieldErrors.push('Неверный формат значения');
            }
            break;
            
          case 'validator':
            if (typeof ruleValue === 'string' && this.customValidators.has(ruleValue)) {
              const customValidator = this.customValidators.get(ruleValue);
              const result = customValidator(value);
              if (result !== true) {
                fieldErrors.push(result || 'Неверное значение');
              }
            } else if (typeof ruleValue === 'function') {
              const result = ruleValue(value);
              if (result !== true) {
                fieldErrors.push(result || 'Неверное значение');
              }
            }
            break;
        }
      }
      
      if (fieldErrors.length > 0) {
        errors[field] = fieldErrors;
      }
    }
    
    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }
}

// Пример использования
const validator = new FormValidator();

// Добавляем кастомные валидаторы
validator.addCustomValidator('email', validateEmail);
validator.addCustomValidator('strongPassword', (password) => {
  if (password.length < 8) return 'Пароль должен содержать не менее 8 символов';
  if (!/[A-Z]/.test(password)) return 'Пароль должен содержать заглавную букву';
  if (!/[a-z]/.test(password)) return 'Пароль должен содержать строчную букву';
  if (!/\d/.test(password)) return 'Пароль должен содержать цифру';
  return true;
});

// Добавляем правила валидации
validator
  .addRule('name', { required: true, minLength: 2, maxLength: 50 })
  .addRule('email', { required: true, validator: 'email' })
  .addRule('password', { required: true, validator: 'strongPassword' })
  .addRule('age', { type: 'number', min: 18, max: 100 });

// Валидация данных
const formData = {
  name: 'John',
  email: 'john@example.com',
  password: 'MySecurePassword123',
  age: 30
};

const result = validator.validate(formData);
console.log(result); // { isValid: true, errors: {} }
```

## Практические советы

- Используйте валидацию как на клиенте, так и на сервере
- Предоставляйте пользователю понятные сообщения об ошибках
- Валидируйте данные как можно ближе к источнику
- Используйте регулярные выражения аккуратно и тестируйте их тщательно
- Для сложных форм используйте специализированные библиотеки валидации

## Связанные темы

- [[javascript-error-handling]]
- [[form-handling-best-practices]]
- [[security-validation-patterns]]
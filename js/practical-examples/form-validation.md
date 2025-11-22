---
aliases: ["Валидация форм", "Работа с формами", "Form Validation"]
tags: [js, forms, validation, frontend, practical-examples]
---

# Работа с формами и валидация

## Создание и управление формами

### Базовая структура формы

```javascript
// Класс для управления формой
class FormHandler {
  constructor(formSelector) {
    this.form = document.querySelector(formSelector);
    this.fields = {};
    this.errors = {};
    this.validators = {};
    this.init();
  }
  
  init() {
    // Сбор всех полей формы
    const inputs = this.form.querySelectorAll('input, textarea, select');
    inputs.forEach(input => {
      this.fields[input.name] = input;
      this.setupFieldValidation(input);
    });
    
    // Добавление обработчика отправки формы
    this.form.addEventListener('submit', this.handleSubmit.bind(this));
  }
  
  setupFieldValidation(input) {
    // Валидация при потере фокуса
    input.addEventListener('blur', () => {
      this.validateField(input.name);
    });
    
    // Валидация в реальном времени
    input.addEventListener('input', () => {
      if (this.errors[input.name]) {
        this.clearFieldError(input.name);
      }
    });
  }
  
  addValidator(fieldName, validatorFn, errorMessage) {
    if (!this.validators[fieldName]) {
      this.validators[fieldName] = [];
    }
    this.validators[fieldName].push({
      validate: validatorFn,
      message: errorMessage
    });
  }
  
  validateField(fieldName) {
    const field = this.fields[fieldName];
    const value = field.value;
    const validators = this.validators[fieldName] || [];
    
    for (const validator of validators) {
      if (!validator.validate(value)) {
        this.setFieldError(fieldName, validator.message);
        return false;
      }
    }
    
    this.clearFieldError(fieldName);
    return true;
  }
  
  setFieldError(fieldName, message) {
    this.errors[fieldName] = message;
    const field = this.fields[fieldName];
    field.classList.add('error');
    
    // Добавление сообщения об ошибке
    let errorElement = field.parentNode.querySelector('.error-message');
    if (!errorElement) {
      errorElement = document.createElement('div');
      errorElement.className = 'error-message';
      field.parentNode.appendChild(errorElement);
    }
    errorElement.textContent = message;
  }
  
  clearFieldError(fieldName) {
    delete this.errors[fieldName];
    const field = this.fields[fieldName];
    field.classList.remove('error');
    
    const errorElement = field.parentNode.querySelector('.error-message');
    if (errorElement) {
      errorElement.remove();
    }
  }
  
  validateForm() {
    let isValid = true;
    Object.keys(this.fields).forEach(fieldName => {
      if (!this.validateField(fieldName)) {
        isValid = false;
      }
    });
    return isValid;
  }
  
  handleSubmit(event) {
    event.preventDefault();
    
    if (this.validateForm()) {
      // Форма валидна, отправка данных
      const formData = this.getFormData();
      this.onSubmit(formData);
    } else {
      console.log('Форма содержит ошибки:', this.errors);
    }
  }
  
  getFormData() {
    const data = {};
    Object.keys(this.fields).forEach(fieldName => {
      data[fieldName] = this.fields[fieldName].value;
    });
    return data;
  }
  
  onSubmit(formData) {
    // Переопределяется в наследниках
    console.log('Данные формы:', formData);
  }
}

// Пример использования
const contactForm = new FormHandler('#contactForm');

// Добавление валидаторов
contactForm.addValidator('name', value => value.length >= 2, 'Имя должно содержать не менее 2 символов');
contactForm.addValidator('email', value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value), 'Введите корректный email');
contactForm.addValidator('message', value => value.length >= 10, 'Сообщение должно содержать не менее 10 символов');

contactForm.onSubmit = (formData) => {
  console.log('Форма отправлена:', formData);
  // Отправка данных на сервер
};
```

## Валидация различных типов полей

### Валидация email

```javascript
// Расширенная валидация email
function validateEmail(email) {
  // Базовая проверка формата
  const basicRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!basicRegex.test(email)) {
    return false;
  }
  
  // Проверка длины домена
  const domain = email.split('@')[1];
  if (domain.length > 253) {
    return false;
  }
  
  // Проверка на наличие проблемных символов
  if (email.includes('..')) {
    return false;
  }
  
  return true;
}

// Валидация с проверкой домена
async function validateEmailWithDomain(email) {
  if (!validateEmail(email)) {
    return false;
  }
  
  const domain = email.split('@')[1];
  try {
    // Проверка MX записи домена (требует серверного API)
    const response = await fetch(`/api/validate-email-domain?domain=${domain}`);
    return response.ok;
  } catch (error) {
    console.warn('Не удалось проверить домен email:', error);
    return true; // Позволяем продолжить, если проверка домена недоступна
  }
}
```

### Валидация паролей

```javascript
function validatePassword(password) {
  const checks = {
    length: password.length >= 8,
    hasUpper: /[A-Z]/.test(password),
    hasLower: /[a-z]/.test(password),
    hasNumber: /\d/.test(password),
    hasSpecial: /[!@#$%^&*(),.?":{}|<>]/.test(password)
  };
  
  return Object.values(checks).every(check => check);
}

function getPasswordStrength(password) {
  let score = 0;
  const checks = [
    { test: p => p.length >= 8, value: 1 },
    { test: p => /[A-Z]/.test(p), value: 1 },
    { test: p => /[a-z]/.test(p), value: 1 },
    { test: p => /\d/.test(p), value: 1 },
    { test: p => /[!@#$%^&*(),.?":{}|<>]/.test(p), value: 2 },
    { test: p => p.length >= 12, value: 2 }
  ];
  
  checks.forEach(check => {
    if (check.test(password)) {
      score += check.value;
    }
  });
  
  if (score < 3) return 'слабый';
  if (score < 5) return 'средний';
  if (score < 7) return 'сильный';
  return 'очень сильный';
}

// Показатель сложности пароля
function showPasswordStrength(password) {
  const strength = getPasswordStrength(password);
  const strengthElement = document.querySelector('#password-strength');
  
  if (strengthElement) {
    strengthElement.textContent = `Сложность: ${strength}`;
    strengthElement.className = `password-strength ${strength}`;
  }
}
```

### Валидация телефонов

```javascript
function validatePhone(phone) {
  // Удаление всех нецифровых символов
  const cleanPhone = phone.replace(/\D/g, '');
  
  // Проверка длины (обычно 10-15 цифр)
  if (cleanPhone.length < 10 || cleanPhone.length > 15) {
    return false;
  }
  
  // Проверка формата (начинается с + или 0)
  return /^(\+|0)[1-9]\d{9,14}$/.test(phone);
}

function formatPhone(phone) {
  // Форматирование телефонного номера
  const cleanPhone = phone.replace(/\D/g, '');
  
  // Пример форматирования: +7 (XXX) XXX-XX-XX
  if (cleanPhone.length === 11 && cleanPhone.startsWith('7')) {
    return `+${cleanPhone.slice(0, 1)} (${cleanPhone.slice(1, 4)}) ${cleanPhone.slice(4, 7)}-${cleanPhone.slice(7, 9)}-${cleanPhone.slice(9, 11)}`;
  }
  
  return phone;
}
```

## Асинхронная валидация

```javascript
class AsyncFormHandler extends FormHandler {
  constructor(formSelector) {
    super(formSelector);
    this.asyncValidators = {};
  }
  
  addAsyncValidator(fieldName, validatorFn, errorMessage) {
    if (!this.asyncValidators[fieldName]) {
      this.asyncValidators[fieldName] = [];
    }
    this.asyncValidators[fieldName].push({
      validate: validatorFn,
      message: errorMessage
    });
  }
  
  async validateFieldAsync(fieldName) {
    const field = this.fields[fieldName];
    const value = field.value;
    
    // Сначала синхронная валидация
    if (!this.validateField(fieldName)) {
      return false;
    }
    
    // Асинхронная валидация
    const asyncValidators = this.asyncValidators[fieldName] || [];
    for (const validator of asyncValidators) {
      try {
        const isValid = await validator.validate(value);
        if (!isValid) {
          this.setFieldError(fieldName, validator.message);
          return false;
        }
      } catch (error) {
        console.error('Ошибка асинхронной валидации:', error);
        this.setFieldError(fieldName, 'Ошибка проверки поля');
        return false;
      }
    }
    
    return true;
  }
  
  async validateFormAsync() {
    let isValid = true;
    const fieldNames = Object.keys(this.fields);
    
    // Параллельная валидация всех полей
    const results = await Promise.all(
      fieldNames.map(fieldName => this.validateFieldAsync(fieldName))
    );
    
    return results.every(result => result);
  }
  
  async handleSubmit(event) {
    event.preventDefault();
    
    // Показать индикатор загрузки
    this.setLoading(true);
    
    try {
      const isValid = await this.validateFormAsync();
      if (isValid) {
        const formData = this.getFormData();
        await this.onSubmit(formData);
      } else {
        console.log('Форма содержит ошибки:', this.errors);
      }
    } finally {
      this.setLoading(false);
    }
  }
  
  setLoading(loading) {
    const submitButton = this.form.querySelector('button[type="submit"]');
    if (submitButton) {
      submitButton.disabled = loading;
      submitButton.textContent = loading ? 'Отправка...' : 'Отправить';
    }
  }
}

// Пример использования с асинхронной валидацией
const registerForm = new AsyncFormHandler('#registerForm');

// Добавление синхронных валидаторов
registerForm.addValidator('username', value => value.length >= 3, 'Имя пользователя должно содержать не менее 3 символов');
registerForm.addValidator('email', validateEmail, 'Введите корректный email');

// Добавление асинхронных валидаторов
registerForm.addAsyncValidator('username', async (username) => {
  const response = await fetch(`/api/check-username?username=${encodeURIComponent(username)}`);
  const result = await response.json();
  return result.available;
}, 'Имя пользователя уже занято');

registerForm.addAsyncValidator('email', async (email) => {
  const response = await fetch(`/api/check-email?email=${encodeURIComponent(email)}`);
  const result = await response.json();
  return result.available;
}, 'Email уже используется');
```

## Валидация сложных форм

### Валидация вложенных данных

```javascript
class ComplexFormHandler extends AsyncFormHandler {
  constructor(formSelector) {
    super(formSelector);
    this.fieldGroups = {};
  }
  
  addFieldGroup(groupName, fieldNames) {
    this.fieldGroups[groupName] = fieldNames;
  }
  
  validateFieldGroup(groupName) {
    const fieldNames = this.fieldGroups[groupName] || [];
    let isValid = true;
    
    fieldNames.forEach(fieldName => {
      if (!this.validateField(fieldName)) {
        isValid = false;
      }
    });
    
    return isValid;
  }
  
  // Валидация условных полей
  addConditionalValidator(fieldToValidate, conditionField, conditionFn, validatorFn, errorMessage) {
    const conditionFieldElement = this.fields[conditionField];
    if (conditionFieldElement) {
      conditionFieldElement.addEventListener('change', () => {
        if (conditionFn(conditionFieldElement.value)) {
          // Делаем поле обязательным
          this.addValidator(fieldToValidate, validatorFn, errorMessage);
          this.validateField(fieldToValidate);
        } else {
          // Убираем валидатор
          this.validators[fieldToValidate] = this.validators[fieldToValidate]
            .filter(v => v.validate !== validatorFn);
          this.clearFieldError(fieldToValidate);
        }
      });
    }
  }
}

// Пример формы с условной валидацией
const profileForm = new ComplexFormHandler('#profileForm');

// Основные поля
profileForm.addValidator('firstName', value => value.length >= 2, 'Имя должно содержать не менее 2 символов');
profileForm.addValidator('lastName', value => value.length >= 2, 'Фамилия должна содержать не менее 2 символов');

// Условная валидация - если выбран тип "бизнес", требовать ИНН
profileForm.addConditionalValidator(
  'taxId', // поле для валидации
  'accountType', // условное поле
  value => value === 'business', // условие
  value => /^\d{10,12}$/.test(value), // валидатор
  'ИНН должен содержать 10-12 цифр' // сообщение об ошибке
);
```

## Пользовательский интерфейс валидации

### Индикаторы валидации

```javascript
class VisualFormHandler extends ComplexFormHandler {
  constructor(formSelector) {
    super(formSelector);
    this.validationIcons = {};
  }
  
  setupFieldValidation(input) {
    super.setupFieldValidation(input);
    this.addValidationIcon(input);
  }
  
  addValidationIcon(input) {
    const container = input.parentNode;
    const icon = document.createElement('span');
    icon.className = 'validation-icon';
    container.style.position = 'relative';
    container.appendChild(icon);
    this.validationIcons[input.name] = icon;
  }
  
  setFieldError(fieldName, message) {
    super.setFieldError(fieldName, message);
    const icon = this.validationIcons[fieldName];
    if (icon) {
      icon.textContent = '❌';
      icon.title = message;
    }
  }
  
  clearFieldError(fieldName) {
    super.clearFieldError(fieldName);
    const icon = this.validationIcons[fieldName];
    if (icon) {
      icon.textContent = '✅';
      icon.title = 'Поле валидно';
      setTimeout(() => {
        icon.textContent = '';
        icon.title = '';
      }, 2000);
    }
  }
}
```

### Валидация с подсказками

```javascript
function createValidationTooltip(field, message) {
  const tooltip = document.createElement('div');
  tooltip.className = 'validation-tooltip';
  tooltip.textContent = message;
  
  // Позиционирование подсказки
  const rect = field.getBoundingClientRect();
  tooltip.style.position = 'absolute';
  tooltip.style.left = `${rect.left + window.scrollX}px`;
  tooltip.style.top = `${rect.bottom + window.scrollY + 5}px`;
  
  document.body.appendChild(tooltip);
  
  // Удаление через время
  setTimeout(() => {
    if (tooltip.parentNode) {
      tooltip.parentNode.removeChild(tooltip);
    }
  }, 3000);
  
  return tooltip;
}

// Проверка в реальном времени с подсказками
function setupRealTimeValidation(input, validators) {
  input.addEventListener('input', () => {
    const value = input.value;
    
    for (const validator of validators) {
      if (!validator.validate(value)) {
        // Показать подсказку
        createValidationTooltip(input, validator.message);
        input.classList.add('invalid');
        input.classList.remove('valid');
        return;
      }
    }
    
    // Если все валидаторы пройдены
    input.classList.remove('invalid');
    input.classList.add('valid');
  });
}
```

## Массовая валидация

```javascript
// Валидация массива данных
function validateDataArray(data, schema) {
  const results = [];
  
  for (let i = 0; i < data.length; i++) {
    const item = data[i];
    const itemErrors = {};
    
    for (const [field, validators] of Object.entries(schema)) {
      for (const validator of validators) {
        if (!validator.validate(item[field])) {
          if (!itemErrors[field]) {
            itemErrors[field] = [];
          }
          itemErrors[field].push(validator.message);
        }
      }
    }
    
    results.push({
      index: i,
      data: item,
      errors: itemErrors,
      isValid: Object.keys(itemErrors).length === 0
    });
  }
  
  return results;
}

// Пример использования
const usersData = [
  { name: 'John', email: 'john@example.com', age: 25 },
  { name: 'Jane', email: 'invalid-email', age: 17 },
  { name: '', email: 'joe@example.com', age: 30 }
];

const userSchema = {
  name: [
    { validate: value => value && value.length >= 2, message: 'Имя обязательно и должно быть не менее 2 символов' }
  ],
  email: [
    { validate: validateEmail, message: 'Введите корректный email' }
  ],
  age: [
    { validate: value => value >= 18, message: 'Возраст должен быть не менее 18 лет' }
  ]
};

const validationResults = validateDataArray(usersData, userSchema);
console.log('Результаты валидации:', validationResults);
```

## Лучшие практики

### Оптимизация производительности

```javascript
// Дебаунсинг валидации
function debounceValidator(validator, delay = 300) {
  let timeoutId;
  return function(value) {
    clearTimeout(timeoutId);
    return new Promise(resolve => {
      timeoutId = setTimeout(() => {
        resolve(validator(value));
      }, delay);
    });
  };
}

// Батчевая валидация
class BatchValidator {
  constructor() {
    this.queue = [];
    this.timer = null;
  }
  
  add(validationFn) {
    this.queue.push(validationFn);
    
    if (!this.timer) {
      this.timer = setTimeout(() => {
        this.processBatch();
      }, 100); // Обработка каждые 100мс
    }
  }
  
  async processBatch() {
    const results = await Promise.all(
      this.queue.map(fn => fn())
    );
    
    this.queue = [];
    this.timer = null;
    
    return results;
  }
}
```

### Обработка ошибок

```javascript
// Централизованная обработка ошибок валидации
class ValidationError extends Error {
  constructor(field, message, code = 'VALIDATION_ERROR') {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
    this.code = code;
  }
}

// Обработчик ошибок формы
function handleFormError(error) {
  if (error instanceof ValidationError) {
    console.error(`Ошибка валидации поля ${error.field}: ${error.message}`);
    // Можем показать пользователю специфичную ошибку
    showFieldError(error.field, error.message);
  } else {
    console.error('Неизвестная ошибка формы:', error);
    // Показать общее сообщение об ошибке
    showGeneralError('Произошла ошибка при обработке формы');
  }
}
```

## Ссылки по теме

- [[js/forms/html5-validation]] - HTML5 валидация
- [[js/patterns/form-builder]] - Паттерн Form Builder
- [[js/async/validation]] - Асинхронная валидация
- [[js/ui/validation-ui]] - UI компоненты валидации
# Продвинутые HTML формы: валидация, доступность и пользовательский опыт

## Введение

HTML-формы являются ключевым элементом взаимодействия между пользователем и веб-приложением. Современные формы должны быть не только функциональными, но и доступными, безопасными и удобными в использовании. В этой статье мы рассмотрим продвинутые техники работы с HTML-формами.

## Продвинутые элементы форм

### Новые элементы и атрибуты HTML5

```html
<!-- Пользовательские элементы форм -->
<form class="advanced-form" novalidate>
  <!-- Поле для выбора даты с ограничениями -->
  <div class="form-group">
    <label for="birth-date">Дата рождения</label>
    <input 
      type="date" 
      id="birth-date" 
      name="birth-date"
      min="1900-01-01"
      max="2023-12-31"
      required>
    <span class="form-hint">Выберите вашу дату рождения</span>
  </div>

  <!-- Поле для выбора времени -->
  <div class="form-group">
    <label for="appointment-time">Время встречи</label>
    <input 
      type="time" 
      id="appointment-time" 
      name="appointment-time"
      min="09:00"
      max="18:00"
      value="10:30">
  </div>

  <!-- Диапазон значений -->
  <div class="form-group">
    <label for="satisfaction">Удовлетворенность (1-10)</label>
    <input 
      type="range" 
      id="satisfaction" 
      name="satisfaction"
      min="1" 
      max="10" 
      value="5"
      step="1">
    <output for="satisfaction" id="satisfaction-value">5</output>
  </div>

  <!-- Цвет -->
  <div class="form-group">
    <label for="favorite-color">Любимый цвет</label>
    <input 
      type="color" 
      id="favorite-color" 
      name="favorite-color"
      value="#3498db">
  </div>

  <!-- Телефон с маской -->
  <div class="form-group">
    <label for="phone">Телефон</label>
    <input 
      type="tel" 
      id="phone" 
      name="phone"
      pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
      placeholder="123-456-7890"
      autocomplete="tel">
    <span class="form-hint">Формат: 123-456-7890</span>
  </div>

  <!-- Email с множественным выбором -->
  <div class="form-group">
    <label for="email">Основной email</label>
    <input 
      type="email" 
      id="email" 
      name="email"
      required
      autocomplete="email">
    
    <label for="additional-emails">Дополнительные email (через запятую)</label>
    <input 
      type="email" 
      id="additional-emails" 
      name="additional-emails"
      multiple
      autocomplete="email">
  </div>

  <!-- Числовое поле с шагом -->
  <div class="form-group">
    <label for="quantity">Количество</label>
    <input 
      type="number" 
      id="quantity" 
      name="quantity"
      min="1" 
      max="100" 
      step="1"
      value="1"
      required>
  </div>

  <!-- Поле с автодополнением -->
  <div class="form-group">
    <label for="country">Страна</label>
    <input 
      type="text" 
      id="country" 
      name="country"
      list="countries"
      autocomplete="country-name"
      required>
    <datalist id="countries">
      <option value="Россия">
      <option value="США">
      <option value="Канада">
      <option value="Германия">
      <option value="Франция">
    </datalist>
  </div>

  <!-- Файл с ограничениями -->
  <div class="form-group">
    <label for="avatar">Фото профиля</label>
    <input 
      type="file" 
      id="avatar" 
      name="avatar"
      accept="image/*"
      capture="environment"
      required>
    <span class="form-hint">Только изображения (JPG, PNG), не более 5MB</span>
  </div>

  <button type="submit">Отправить</button>
</form>
```

### Современные атрибуты валидации

```html
<form class="validation-form">
  <!-- Встроенная валидация -->
  <div class="form-group">
    <label for="username">Имя пользователя</label>
    <input 
      type="text" 
      id="username" 
      name="username"
      minlength="3"
      maxlength="20"
      pattern="[a-zA-Z0-9_]+"
      required
      title="Только буквы, цифры и подчеркивание, от 3 до 20 символов">
    <div class="validation-message" data-field="username"></div>
  </div>

  <!-- Пароль с требованиями -->
  <div class="form-group">
    <label for="password">Пароль</label>
    <input 
      type="password" 
      id="password" 
      name="password"
      minlength="8"
      required>
    <div class="password-requirements">
      <ul>
        <li data-req="length">Минимум 8 символов</li>
        <li data-req="uppercase">Хотя бы одна заглавная буква</li>
        <li data-req="lowercase">Хотя бы одна строчная буква</li>
        <li data-req="number">Хотя бы одна цифра</li>
        <li data-req="special">Хотя бы один специальный символ</li>
      </ul>
    </div>
  </div>

  <!-- Подтверждение пароля -->
  <div class="form-group">
    <label for="confirm-password">Подтвердите пароль</label>
    <input 
      type="password" 
      id="confirm-password" 
      name="confirm-password"
      required>
    <div class="validation-message" data-field="confirm-password"></div>
  </div>

  <!-- URL с проверкой -->
  <div class="form-group">
    <label for="website">Веб-сайт</label>
    <input 
      type="url" 
      id="website" 
      name="website"
      placeholder="https://example.com">
  </div>

  <!-- Поиск с результатами -->
  <div class="form-group">
    <label for="search">Поиск</label>
    <input 
      type="search" 
      id="search" 
      name="search"
      results="5"
      autosave="search-queries">
  </div>

  <button type="submit">Зарегистрироваться</button>
</form>
```

## Современные методы валидации

### Клиентская валидация с использованием Constraint Validation API

```html
<form id="registration-form" class="validation-form" novalidate>
  <div class="form-group">
    <label for="email">Email</label>
    <input 
      type="email" 
      id="email" 
      name="email"
      required>
    <div class="validation-message"></div>
  </div>

  <div class="form-group">
    <label for="age">Возраст</label>
    <input 
      type="number" 
      id="age" 
      name="age"
      min="18" 
      max="120"
      required>
    <div class="validation-message"></div>
  </div>

  <div class="form-group">
    <label for="phone">Телефон</label>
    <input 
      type="tel" 
      id="phone" 
      name="phone"
      required>
    <div class="validation-message"></div>
  </div>

  <button type="submit">Отправить</button>
</form>
```

```javascript
// Продвинутая валидация формы
class FormValidator {
  constructor(formElement) {
    this.form = formElement;
    this.fields = formElement.querySelectorAll('input, select, textarea');
    this.init();
  }

  init() {
    this.form.addEventListener('submit', this.handleSubmit.bind(this));
    
    this.fields.forEach(field => {
      field.addEventListener('blur', () => this.validateField(field));
      field.addEventListener('input', () => this.clearFieldError(field));
    });
  }

  handleSubmit(event) {
    event.preventDefault();
    
    let isValid = true;
    
    this.fields.forEach(field => {
      if (!this.validateField(field)) {
        isValid = false;
      }
    });

    if (isValid) {
      this.submitForm();
    }
  }

  validateField(field) {
    // Проверяем встроенную валидацию
    let isValid = field.checkValidity();
    
    // Дополнительные проверки
    if (field.type === 'email' && field.value) {
      isValid = this.validateEmail(field.value);
    }
    
    if (field.type === 'tel' && field.value) {
      isValid = this.validatePhone(field.value);
    }
    
    if (field.name === 'password') {
      isValid = this.validatePassword(field.value);
    }
    
    if (!isValid) {
      this.showFieldError(field, field.validationMessage || this.getCustomErrorMessage(field));
    } else {
      this.clearFieldError(field);
    }
    
    return isValid;
  }

  validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }

  validatePhone(phone) {
    // Простая проверка телефонного номера
    const phoneRegex = /^[\+]?[1-9][\d]{0,15}$/;
    return phoneRegex.test(phone.replace(/[\s\-\(\)]/g, ''));
  }

  validatePassword(password) {
    // Проверка сложности пароля
    const minLength = 8;
    const hasUpperCase = /[A-Z]/.test(password);
    const hasLowerCase = /[a-z]/.test(password);
    const hasNumbers = /\d/.test(password);
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(password);
    
    return password.length >= minLength && 
           hasUpperCase && 
           hasLowerCase && 
           hasNumbers && 
           hasSpecialChar;
  }

  getCustomErrorMessage(field) {
    if (field.type === 'email') return 'Пожалуйста, введите корректный email';
    if (field.type === 'tel') return 'Пожалуйста, введите корректный номер телефона';
    if (field.name === 'password') return 'Пароль не соответствует требованиям безопасности';
    return 'Поле заполнено некорректно';
  }

  showFieldError(field, message) {
    const errorContainer = field.parentNode.querySelector('.validation-message');
    if (errorContainer) {
      errorContainer.textContent = message;
      errorContainer.className = 'validation-message error';
    }
    field.classList.add('invalid');
  }

  clearFieldError(field) {
    const errorContainer = field.parentNode.querySelector('.validation-message');
    if (errorContainer) {
      errorContainer.textContent = '';
      errorContainer.className = 'validation-message';
    }
    field.classList.remove('invalid');
  }

  async submitForm() {
    const formData = new FormData(this.form);
    
    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        this.showSuccess('Форма успешно отправлена!');
      } else {
        const error = await response.json();
        this.showGlobalError(error.message);
      }
    } catch (error) {
      this.showGlobalError('Произошла ошибка при отправке формы');
    }
  }

  showSuccess(message) {
    // Показать сообщение об успехе
    console.log('Успех:', message);
  }

  showGlobalError(message) {
    // Показать глобальную ошибку формы
    console.error('Ошибка:', message);
  }
}

// Инициализация валидатора
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('registration-form');
  new FormValidator(form);
});
```

### Валидация с реальными примерами

```html
<form id="advanced-validation-form" class="validation-form" novalidate>
  <!-- Поле с мгновенной валидацией -->
  <div class="form-group">
    <label for="username">Имя пользователя</label>
    <input 
      type="text" 
      id="username" 
      name="username"
      required
      minlength="3"
      maxlength="30"
      pattern="[a-zA-Z0-9_]+">
    <div class="validation-message"></div>
    <div class="feedback"></div>
  </div>

  <!-- Поле с проверкой уникальности -->
  <div class="form-group">
    <label for="email">Email</label>
    <input 
      type="email" 
      id="email" 
      name="email"
      required>
    <div class="validation-message"></div>
    <div class="feedback"></div>
  </div>

  <!-- Сложное поле с маской -->
  <div class="form-group">
    <label for="credit-card">Номер карты</label>
    <input 
      type="text" 
      id="credit-card" 
      name="credit-card"
      required
      placeholder="0000 0000 0000 0000">
    <div class="validation-message"></div>
    <div class="card-type"></div>
  </div>

  <button type="submit">Отправить</button>
</form>
```

```javascript
// Продвинутая валидация с асинхронными проверками
class AdvancedFormValidator extends FormValidator {
  constructor(formElement) {
    super(formElement);
    this.debounceTimers = new Map();
  }

  validateField(field) {
    // Применяем debounce для асинхронных проверок
    if (field.name === 'username' || field.name === 'email') {
      this.debounceAsyncValidation(field);
      return true; // Пока не завершена асинхронная проверка, считаем поле валидным
    }
    
    return super.validateField(field);
  }

  debounceAsyncValidation(field) {
    // Отменяем предыдущий таймер
    if (this.debounceTimers.has(field.name)) {
      clearTimeout(this.debounceTimers.get(field.name));
    }

    // Устанавливаем новый таймер
    const timer = setTimeout(async () => {
      await this.asyncValidateField(field);
    }, 500); // 500ms задержка

    this.debounceTimers.set(field.name, timer);
  }

  async asyncValidateField(field) {
    if (field.name === 'username') {
      await this.validateUsernameUniqueness(field);
    } else if (field.name === 'email') {
      await this.validateEmailUniqueness(field);
    }
  }

  async validateUsernameUniqueness(field) {
    if (field.value.length < 3) return true;

    try {
      const response = await fetch(`/api/check-username?username=${encodeURIComponent(field.value)}`);
      const result = await response.json();
      
      if (!result.isAvailable) {
        this.showFieldError(field, 'Имя пользователя уже занято');
        return false;
      } else {
        this.clearFieldError(field);
        this.showFieldSuccess(field, 'Имя пользователя доступно');
        return true;
      }
    } catch (error) {
      this.showFieldError(field, 'Ошибка проверки имени пользователя');
      return false;
    }
  }

  async validateEmailUniqueness(field) {
    if (!this.validateEmail(field.value)) return false;

    try {
      const response = await fetch(`/api/check-email?email=${encodeURIComponent(field.value)}`);
      const result = await response.json();
      
      if (!result.isAvailable) {
        this.showFieldError(field, 'Email уже используется');
        return false;
      } else {
        this.clearFieldError(field);
        this.showFieldSuccess(field, 'Email доступен');
        return true;
      }
    } catch (error) {
      this.showFieldError(field, 'Ошибка проверки email');
      return false;
    }
  }

  showFieldSuccess(field, message) {
    const feedback = field.parentNode.querySelector('.feedback');
    if (feedback) {
      feedback.textContent = message;
      feedback.className = 'feedback success';
    }
    field.classList.remove('invalid');
    field.classList.add('valid');
  }

  validateCreditCard(cardNumber) {
    // Проверка номера карты по алгоритму Луна
    cardNumber = cardNumber.replace(/\s/g, '');
    
    if (!/^\d{16}$/.test(cardNumber)) {
      return { valid: false, type: 'invalid' };
    }

    // Алгоритм Луна
    let sum = 0;
    let isEven = false;
    
    for (let i = cardNumber.length - 1; i >= 0; i--) {
      let digit = parseInt(cardNumber.charAt(i));
      
      if (isEven) {
        digit *= 2;
        if (digit > 9) {
          digit -= 9;
        }
      }
      
      sum += digit;
      isEven = !isEven;
    }
    
    const isValid = (sum % 10) === 0;
    
    // Определение типа карты
    let type = 'unknown';
    if (/^4/.test(cardNumber)) type = 'visa';
    else if (/^5[1-5]/.test(cardNumber)) type = 'mastercard';
    else if (/^3[47]/.test(cardNumber)) type = 'amex';
    else if (/^6(?:011|5)/.test(cardNumber)) type = 'discover';

    return { valid: isValid, type };
  }
}

// Инициализация продвинутого валидатора
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('advanced-validation-form');
  new AdvancedFormValidator(form);
});
```

## Доступность HTML-форм

### Семантическая разметка и ARIA

```html
<!-- Доступная форма с правильной семантикой -->
<form 
  id="accessible-form" 
  class="accessible-form" 
  aria-labelledby="form-title"
  aria-describedby="form-description"
  novalidate>

  <h2 id="form-title">Форма регистрации</h2>
  <p id="form-description">Пожалуйста, заполните все обязательные поля</p>

  <!-- Группировка полей -->
  <fieldset aria-labelledby="personal-info">
    <legend id="personal-info">Личная информация</legend>
    
    <div class="form-group">
      <label for="first-name">Имя <span class="required" aria-label="обязательное поле">*</span></label>
      <input 
        type="text" 
        id="first-name" 
        name="first-name"
        required
        aria-describedby="first-name-help">
      <span id="first-name-help" class="form-hint">Введите ваше имя</span>
      <span id="first-name-error" class="error-message" role="alert" aria-live="polite"></span>
    </div>

    <div class="form-group">
      <label for="last-name">Фамилия <span class="required" aria-label="обязательное поле">*</span></label>
      <input 
        type="text" 
        id="last-name" 
        name="last-name"
        required
        aria-describedby="last-name-help last-name-error">
      <span id="last-name-help" class="form-hint">Введите вашу фамилию</span>
      <span id="last-name-error" class="error-message" role="alert" aria-live="polite"></span>
    </div>
  </fieldset>

  <!-- Контактная информация -->
  <fieldset aria-labelledby="contact-info">
    <legend id="contact-info">Контактная информация</legend>
    
    <div class="form-group">
      <label for="email">Email <span class="required" aria-label="обязательное поле">*</span></label>
      <input 
        type="email" 
        id="email" 
        name="email"
        required
        aria-describedby="email-help email-error"
        autocomplete="email">
      <span id="email-help" class="form-hint">Введите действующий email</span>
      <span id="email-error" class="error-message" role="alert" aria-live="polite"></span>
    </div>

    <div class="form-group">
      <label for="phone">Телефон</label>
      <input 
        type="tel" 
        id="phone" 
        name="phone"
        aria-describedby="phone-help phone-error"
        autocomplete="tel">
      <span id="phone-help" class="form-hint">Введите номер телефона</span>
      <span id="phone-error" class="error-message" role="alert" aria-live="polite"></span>
    </div>
  </fieldset>

  <!-- Выбор опций -->
  <fieldset aria-labelledby="preferences-title">
    <legend id="preferences-title">Предпочтения</legend>
    
    <div class="checkbox-group">
      <label class="checkbox-label">
        <input 
          type="checkbox" 
          name="newsletter" 
          value="yes"
          aria-describedby="newsletter-help">
        <span class="checkbox-custom"></span>
        Подписка на рассылку
      </label>
      <span id="newsletter-help" class="form-hint">Получать новости и обновления</span>
    </div>

    <div class="radio-group">
      <p id="contact-method-label">Предпочтительный способ связи:</p>
      <label class="radio-label">
        <input 
          type="radio" 
          name="contact-method" 
          value="email"
          aria-describedby="contact-method-label">
        <span class="radio-custom"></span>
        Email
      </label>
      <label class="radio-label">
        <input 
          type="radio" 
          name="contact-method" 
          value="phone"
          aria-describedby="contact-method-label">
        <span class="radio-custom"></span>
        Телефон
      </label>
      <label class="radio-label">
        <input 
          type="radio" 
          name="contact-method" 
          value="sms"
          aria-describedby="contact-method-label">
        <span class="radio-custom"></span>
        SMS
      </label>
    </div>
  </fieldset>

  <div class="form-actions">
    <button type="submit" id="submit-btn" aria-describedby="submit-help">
      Зарегистрироваться
    </button>
    <span id="submit-help" class="form-hint">Нажмите для отправки формы</span>
  </div>
</form>
```

### Клавиатурная навигация и фокус

```css
/* Стили для доступности фокуса */
.accessible-form {
  max-width: 600px;
  margin: 0 auto;
  padding: 2rem;
}

/* Визуальная индикация фокуса */
.accessible-form input:focus,
.accessible-form select:focus,
.accessible-form textarea:focus,
.accessible-form button:focus {
  outline: 2px solid #3498db;
  outline-offset: 2px;
}

/* Скрытие нативного фокуса только при использовании мыши */
@media (prefers-reduced-motion: no-preference) {
  .accessible-form input,
  .accessible-form select,
  .accessible-form textarea,
  .accessible-form button {
    transition: outline 0.2s ease;
  }
}

/* Стили для кастомных чекбоксов и радио-кнопок */
.checkbox-label,
.radio-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  margin-bottom: 0.5rem;
  position: relative;
  padding-left: 2rem;
}

.checkbox-custom,
.radio-custom {
  position: absolute;
  left: 0;
  top: 0.25rem;
  width: 1.25rem;
  height: 1.25rem;
  border: 2px solid #bdc3c7;
  background: white;
}

.checkbox-custom {
  border-radius: 0.25rem;
}

.radio-custom {
  border-radius: 50%;
}

/* Состояния для кастомных элементов */
input[type="checkbox"]:checked + .checkbox-custom {
  background: #3498db;
  border-color: #3498db;
}

input[type="radio"]:checked + .radio-custom {
  background: #3498db;
  border-color: #3498db;
}

/* Псевдоэлементы для галочки и точки */
input[type="checkbox"]:checked + .checkbox-custom::after {
  content: '';
  position: absolute;
  left: 0.3rem;
  top: 0.1rem;
  width: 0.3rem;
  height: 0.6rem;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

input[type="radio"]:checked + .radio-custom::after {
  content: '';
  position: absolute;
  left: 0.3rem;
  top: 0.3rem;
  width: 0.5rem;
  height: 0.5rem;
  border-radius: 50%;
  background: white;
}

/* Стили для сообщений об ошибках */
.error-message {
  color: #e74c3c;
  font-size: 0.875rem;
  margin-top: 0.25rem;
  display: block;
}

/* Состояния полей при ошибках */
input:invalid,
input.error {
  border-color: #e74c3c;
  background-color: #fdeded;
}

input:valid:not(:placeholder-shown) {
  border-color: #2ecc71;
  background-color: #f0fdf4;
}

/* Скрытие элементов для скринридеров */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* Индикация обязательных полей */
.required {
  color: #e74c3c;
}
```

### JavaScript для доступности

```javascript
// Улучшенная доступность формы
class AccessibleForm {
  constructor(formElement) {
    this.form = formElement;
    this.currentFieldIndex = 0;
    this.fields = Array.from(formElement.querySelectorAll(
      'input:not([type="hidden"]), select, textarea, button'
    ));
    this.init();
  }

  init() {
    this.setupKeyboardNavigation();
    this.setupErrorHandling();
    this.setupAnnouncements();
  }

  setupKeyboardNavigation() {
    // Обработка клавиш для навигации
    this.form.addEventListener('keydown', (event) => {
      if (event.key === 'Enter' && event.target.tagName === 'SELECT') {
        event.preventDefault();
        // Дополнительная логика для select элементов
      }
      
      if (event.key === 'Escape') {
        this.resetForm();
      }
    });
  }

  setupErrorHandling() {
    // Обработка ошибок с доступностью
    this.form.addEventListener('invalid', (event) => {
      event.preventDefault();
      this.showAccessibleError(event.target);
    }, true);
  }

  showAccessibleError(field) {
    const errorId = `error-${field.id || field.name}`;
    const errorMessage = field.validationMessage || 'Поле заполнено некорректно';
    
    // Создаем элемент сообщения об ошибке
    let errorElement = document.getElementById(errorId);
    if (!errorElement) {
      errorElement = document.createElement('div');
      errorElement.id = errorId;
      errorElement.className = 'error-message';
      errorElement.setAttribute('role', 'alert');
      errorElement.setAttribute('aria-live', 'assertive');
      field.parentNode.appendChild(errorElement);
    }
    
    errorElement.textContent = errorMessage;
    field.setAttribute('aria-describedby', `${field.getAttribute('aria-describedby') || ''} ${errorId}`.trim());
    
    // Фокус на поле с ошибкой
    field.focus();
  }

  setupAnnouncements() {
    // Создаем live region для анонсировок
    this.announcementDiv = document.createElement('div');
    this.announcementDiv.setAttribute('aria-live', 'polite');
    this.announcementDiv.setAttribute('aria-atomic', 'true');
    this.announcementDiv.className = 'sr-only';
    document.body.appendChild(this.announcementDiv);
  }

  announce(message) {
    this.announcementDiv.textContent = message;
    
    // Очистка через 1 секунду
    setTimeout(() => {
      this.announcementDiv.textContent = '';
    }, 1000);
  }

  resetForm() {
    this.form.reset();
    this.announce('Форма сброшена');
    
    // Возвращаем фокус на первое поле
    if (this.fields.length > 0) {
      this.fields[0].focus();
    }
  }
}

// Инициализация доступной формы
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('accessible-form');
  new AccessibleForm(form);
});
```

## Современные паттерны форм

### Форма с пошаговым заполнением

```html
<form id="multi-step-form" class="multi-step-form">
  <!-- Прогресс-бар -->
  <div class="progress-bar">
    <div class="progress" id="progress"></div>
    <div class="progress-steps">
      <div class="step active" data-step="1">1. Профиль</div>
      <div class="step" data-step="2">2. Контакты</div>
      <div class="step" data-step="3">3. Подтверждение</div>
    </div>
  </div>

  <!-- Шаг 1: Профиль -->
  <div class="form-step active" data-step="1">
    <h3>Информация профиля</h3>
    
    <div class="form-group">
      <label for="profile-first-name">Имя</label>
      <input type="text" id="profile-first-name" name="profile-first-name" required>
    </div>
    
    <div class="form-group">
      <label for="profile-last-name">Фамилия</label>
      <input type="text" id="profile-last-name" name="profile-last-name" required>
    </div>
    
    <div class="form-group">
      <label for="profile-birth-date">Дата рождения</label>
      <input type="date" id="profile-birth-date" name="profile-birth-date" required>
    </div>
    
    <div class="form-actions">
      <button type="button" class="btn-next" data-next-step="2">Далее</button>
    </div>
  </div>

  <!-- Шаг 2: Контакты -->
  <div class="form-step" data-step="2">
    <h3>Контактная информация</h3>
    
    <div class="form-group">
      <label for="contact-email">Email</label>
      <input type="email" id="contact-email" name="contact-email" required>
    </div>
    
    <div class="form-group">
      <label for="contact-phone">Телефон</label>
      <input type="tel" id="contact-phone" name="contact-phone" required>
    </div>
    
    <div class="form-group">
      <label for="contact-address">Адрес</label>
      <textarea id="contact-address" name="contact-address" rows="3"></textarea>
    </div>
    
    <div class="form-actions">
      <button type="button" class="btn-prev" data-prev-step="1">Назад</button>
      <button type="button" class="btn-next" data-next-step="3">Далее</button>
    </div>
  </div>

  <!-- Шаг 3: Подтверждение -->
  <div class="form-step" data-step="3">
    <h3>Подтверждение</h3>
    <div id="summary"></div>
    
    <div class="form-actions">
      <button type="button" class="btn-prev" data-prev-step="2">Назад</button>
      <button type="submit" class="btn-submit">Отправить</button>
    </div>
  </div>
</form>
```

```javascript
// Многошаговая форма
class MultiStepForm {
  constructor(formElement) {
    this.form = formElement;
    this.currentStep = 1;
    this.totalSteps = this.form.querySelectorAll('.form-step').length;
    this.init();
  }

  init() {
    this.updateProgress();
    this.bindEvents();
  }

  bindEvents() {
    // Обработчики для кнопок "Далее"
    this.form.querySelectorAll('.btn-next').forEach(button => {
      button.addEventListener('click', (e) => {
        e.preventDefault();
        this.nextStep(parseInt(e.target.dataset.nextStep));
      });
    });

    // Обработчики для кнопок "Назад"
    this.form.querySelectorAll('.btn-prev').forEach(button => {
      button.addEventListener('click', (e) => {
        e.preventDefault();
        this.prevStep(parseInt(e.target.dataset.prevStep));
      });
    });

    // Валидация перед переходом
    this.form.addEventListener('click', (e) => {
      if (e.target.matches('.btn-next')) {
        if (!this.validateCurrentStep()) {
          e.preventDefault();
        }
      }
    });
  }

  nextStep(step) {
    if (this.validateCurrentStep()) {
      this.goToStep(step);
    }
  }

  prevStep(step) {
    this.goToStep(step);
  }

  goToStep(step) {
    // Скрываем текущий шаг
    this.form.querySelector(`[data-step="${this.currentStep}"]`).classList.remove('active');
    this.form.querySelector(`[data-step="${this.currentStep}"]`).setAttribute('aria-hidden', 'true');
    
    // Показываем новый шаг
    this.form.querySelector(`[data-step="${step}"]`).classList.add('active');
    this.form.querySelector(`[data-step="${step}"]`).setAttribute('aria-hidden', 'false');
    
    // Обновляем текущий шаг
    this.currentStep = step;
    this.updateProgress();
    this.updateSummary();
    
    // Фокус на первый элемент нового шага
    const firstInput = this.form.querySelector(`[data-step="${step}"] input, [data-step="${step}"] textarea, [data-step="${step}"] select`);
    if (firstInput) {
      firstInput.focus();
    }
  }

  validateCurrentStep() {
    const currentStepElement = this.form.querySelector(`[data-step="${this.currentStep}"]`);
    const requiredFields = currentStepElement.querySelectorAll('[required]');
    let isValid = true;

    requiredFields.forEach(field => {
      if (!field.value.trim()) {
        field.classList.add('error');
        isValid = false;
      } else {
        field.classList.remove('error');
      }
    });

    return isValid;
  }

  updateProgress() {
    const progress = ((this.currentStep - 1) / (this.totalSteps - 1)) * 100;
    this.form.querySelector('#progress').style.width = `${progress}%`;
    
    // Обновляем статусы шагов
    this.form.querySelectorAll('.step').forEach(step => {
      const stepNum = parseInt(step.dataset.step);
      if (stepNum < this.currentStep) {
        step.classList.add('completed');
        step.classList.remove('active');
      } else if (stepNum === this.currentStep) {
        step.classList.add('active');
        step.classList.remove('completed');
      } else {
        step.classList.remove('active', 'completed');
      }
    });
  }

  updateSummary() {
    if (this.currentStep === 3) {
      const summaryDiv = this.form.querySelector('#summary');
      const formData = new FormData(this.form);
      
      let summaryHTML = '<h4>Ваши данные:</h4><ul>';
      
      for (let [key, value] of formData.entries()) {
        if (value) {
          summaryHTML += `<li><strong>${this.getFieldLabel(key)}:</strong> ${value}</li>`;
        }
      }
      
      summaryHTML += '</ul>';
      summaryDiv.innerHTML = summaryHTML;
    }
  }

  getFieldLabel(fieldName) {
    // Возвращает дружелюбную метку для поля
    const labels = {
      'profile-first-name': 'Имя',
      'profile-last-name': 'Фамилия',
      'profile-birth-date': 'Дата рождения',
      'contact-email': 'Email',
      'contact-phone': 'Телефон',
      'contact-address': 'Адрес'
    };
    return labels[fieldName] || fieldName;
  }
}

// Инициализация многошаговой формы
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('multi-step-form');
  new MultiStepForm(form);
});
```

## Примеры из промышленной разработки

### Форма заказа с валидацией и доступностью

```html
<form id="order-form" class="order-form" novalidate>
  <div class="form-header">
    <h2>Оформление заказа</h2>
    <p>Пожалуйста, заполните все обязательные поля</p>
  </div>

  <!-- Информация о получателе -->
  <fieldset>
    <legend>Информация о получателе</legend>
    
    <div class="form-row">
      <div class="form-group">
        <label for="recipient-first-name">Имя <span class="required">*</span></label>
        <input 
          type="text" 
          id="recipient-first-name" 
          name="recipient-first-name"
          required
          aria-describedby="recipient-first-name-error"
          autocomplete="given-name">
        <div id="recipient-first-name-error" class="error-message" role="alert" aria-live="assertive"></div>
      </div>
      
      <div class="form-group">
        <label for="recipient-last-name">Фамилия <span class="required">*</span></label>
        <input 
          type="text" 
          id="recipient-last-name" 
          name="recipient-last-name"
          required
          aria-describedby="recipient-last-name-error"
          autocomplete="family-name">
        <div id="recipient-last-name-error" class="error-message" role="alert" aria-live="assertive"></div>
      </div>
    </div>
    
    <div class="form-group">
      <label for="recipient-phone">Телефон <span class="required">*</span></label>
      <input 
        type="tel" 
        id="recipient-phone" 
        name="recipient-phone"
        required
        aria-describedby="recipient-phone-error"
        autocomplete="tel">
      <div id="recipient-phone-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
  </fieldset>

  <!-- Адрес доставки -->
  <fieldset>
    <legend>Адрес доставки</legend>
    
    <div class="form-group">
      <label for="delivery-country">Страна <span class="required">*</span></label>
      <select 
        id="delivery-country" 
        name="delivery-country"
        required
        aria-describedby="delivery-country-error"
        autocomplete="country">
        <option value="">Выберите страну</option>
        <option value="ru">Россия</option>
        <option value="ua">Украина</option>
        <option value="by">Беларусь</option>
      </select>
      <div id="delivery-country-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
    
    <div class="form-row">
      <div class="form-group">
        <label for="delivery-city">Город <span class="required">*</span></label>
        <input 
          type="text" 
          id="delivery-city" 
          name="delivery-city"
          required
          aria-describedby="delivery-city-error"
          autocomplete="address-level2">
        <div id="delivery-city-error" class="error-message" role="alert" aria-live="assertive"></div>
      </div>
      
      <div class="form-group">
        <label for="delivery-zip">Индекс <span class="required">*</span></label>
        <input 
          type="text" 
          id="delivery-zip" 
          name="delivery-zip"
          required
          aria-describedby="delivery-zip-error"
          autocomplete="postal-code">
        <div id="delivery-zip-error" class="error-message" role="alert" aria-live="assertive"></div>
      </div>
    </div>
    
    <div class="form-group">
      <label for="delivery-address">Адрес <span class="required">*</span></label>
      <input 
        type="text" 
        id="delivery-address" 
        name="delivery-address"
        required
        aria-describedby="delivery-address-error"
        autocomplete="street-address">
      <div id="delivery-address-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
  </fieldset>

  <!-- Платежная информация -->
  <fieldset>
    <legend>Платежная информация</legend>
    
    <div class="form-group">
      <label for="card-number">Номер карты <span class="required">*</span></label>
      <input 
        type="text" 
        id="card-number" 
        name="card-number"
        required
        aria-describedby="card-number-error"
        placeholder="0000 0000 0000 0000"
        maxlength="19">
      <div id="card-number-error" class="error-message" role="alert" aria-live="assertive"></div>
      <div id="card-type" class="card-type-info"></div>
    </div>
    
    <div class="form-row">
      <div class="form-group">
        <label for="card-expiry">Срок действия <span class="required">*</span></label>
        <input 
          type="text" 
          id="card-expiry" 
          name="card-expiry"
          required
          aria-describedby="card-expiry-error"
          placeholder="MM/YY"
          maxlength="5">
        <div id="card-expiry-error" class="error-message" role="alert" aria-live="assertive"></div>
      </div>
      
      <div class="form-group">
        <label for="card-cvc">CVC <span class="required">*</span></label>
        <input 
          type="text" 
          id="card-cvc" 
          name="card-cvc"
          required
          aria-describedby="card-cvc-error"
          placeholder="123"
          maxlength="3">
        <div id="card-cvc-error" class="error-message" role="alert" aria-live="assertive"></div>
      </div>
    </div>
  </fieldset>

  <!-- Кнопки формы -->
  <div class="form-actions">
    <button type="button" class="btn btn-secondary" onclick="window.history.back()">Назад</button>
    <button type="submit" class="btn btn-primary" id="submit-order">
      <span class="btn-text">Оформить заказ</span>
      <span class="btn-loading" aria-hidden="true" style="display: none;">Загрузка...</span>
    </button>
  </div>
</form>
```

```javascript
// Промышленная реализация формы заказа
class OrderForm {
  constructor(formElement) {
    this.form = formElement;
    this.isLoading = false;
    this.init();
  }

  init() {
    this.setupEventListeners();
    this.setupInputMasks();
    this.setupRealTimeValidation();
  }

  setupEventListeners() {
    this.form.addEventListener('submit', this.handleSubmit.bind(this));
    
    // Обработчики для мгновенной валидации
    const fields = this.form.querySelectorAll('input, select, textarea');
    fields.forEach(field => {
      field.addEventListener('blur', () => this.validateField(field));
      field.addEventListener('input', () => {
        this.clearFieldError(field);
        this.liveValidation(field);
      });
    });
  }

  setupInputMasks() {
    // Маска для номера карты
    const cardNumberField = this.form.querySelector('#card-number');
    if (cardNumberField) {
      cardNumberField.addEventListener('input', this.formatCardNumber.bind(this));
    }
    
    // Маска для срока действия
    const expiryField = this.form.querySelector('#card-expiry');
    if (expiryField) {
      expiryField.addEventListener('input', this.formatExpiryDate.bind(this));
    }
  }

  formatCardNumber(event) {
    let input = event.target;
    let value = input.value.replace(/\D/g, ''); // Убираем все нецифры
    let formattedValue = '';
    
    for (let i = 0; i < value.length; i++) {
      if (i > 0 && i % 4 === 0) {
        formattedValue += ' ';
      }
      formattedValue += value[i];
    }
    
    input.value = formattedValue.substring(0, 19); // Ограничиваем длину
    
    // Определяем тип карты
    this.detectCardType(value);
  }

  formatExpiryDate(event) {
    let input = event.target;
    let value = input.value.replace(/\D/g, ''); // Убираем все нецифры
    
    if (value.length >= 2) {
      value = value.substring(0, 2) + '/' + value.substring(2, 4);
    }
    
    input.value = value.substring(0, 5); // MM/YY
  }

  detectCardType(cardNumber) {
    const cardTypeElement = this.form.querySelector('#card-type');
    let cardType = 'Неизвестно';
    
    if (cardNumber.startsWith('4')) {
      cardType = 'Visa';
    } else if (['51', '52', '53', '54', '55'].some(prefix => cardNumber.startsWith(prefix))) {
      cardType = 'Mastercard';
    } else if (['34', '37'].some(prefix => cardNumber.startsWith(prefix))) {
      cardType = 'American Express';
    } else if (cardNumber.startsWith('6')) {
      cardType = 'Discover';
    }
    
    if (cardTypeElement) {
      cardTypeElement.textContent = `Тип карты: ${cardType}`;
      cardTypeElement.className = `card-type-info card-${cardType.toLowerCase().replace(' ', '-')}`;
    }
  }

  setupRealTimeValidation() {
    // Валидация email в реальном времени
    const emailField = this.form.querySelector('input[type="email"]');
    if (emailField) {
      emailField.addEventListener('input', this.debounce(() => {
        this.validateEmailRealTime(emailField);
      }, 500));
    }
  }

  validateEmailRealTime(emailField) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    const isValid = emailRegex.test(emailField.value);
    
    if (emailField.value && !isValid) {
      this.showFieldError(emailField, 'Пожалуйста, введите корректный email');
    } else if (isValid) {
      this.clearFieldError(emailField);
    }
  }

  liveValidation(field) {
    // Мгновенная валидация определенных полей
    switch (field.name) {
      case 'recipient-phone':
        this.validatePhoneField(field);
        break;
      case 'delivery-zip':
        this.validateZipField(field);
        break;
      case 'card-cvc':
        this.validateCvcField(field);
        break;
    }
  }

  validatePhoneField(phoneField) {
    const phoneRegex = /^\+?[\d\s\-\(\)]{10,15}$/;
    if (phoneField.value && !phoneRegex.test(phoneField.value.replace(/\s/g, ''))) {
      this.showFieldError(phoneField, 'Введите корректный номер телефона');
    } else {
      this.clearFieldError(phoneField);
    }
  }

  validateZipField(zipField) {
    const zipRegex = /^\d{6}$/; // Простая проверка для российского формата
    if (zipField.value && !zipRegex.test(zipField.value)) {
      this.showFieldError(zipField, 'Индекс должен содержать 6 цифр');
    } else {
      this.clearFieldError(zipField);
    }
  }

  validateCvcField(cvcField) {
    const cvcRegex = /^\d{3,4}$/;
    if (cvcField.value && !cvcRegex.test(cvcField.value)) {
      this.showFieldError(cvcField, 'CVC должен содержать 3-4 цифры');
    } else {
      this.clearFieldError(cvcField);
    }
  }

  validateField(field) {
    let isValid = field.checkValidity();
    
    if (isValid) {
      // Дополнительные проверки
      switch (field.type) {
        case 'email':
          isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(field.value);
          break;
        case 'tel':
          isValid = /^[\+]?[1-9][\d]{0,15}$/.test(field.value.replace(/[\s\-\(\)]/g, ''));
          break;
      }
    }
    
    if (!isValid) {
      this.showFieldError(field, field.validationMessage);
    } else {
      this.clearFieldError(field);
    }
    
    return isValid;
  }

  showFieldError(field, message) {
    const errorElement = this.form.querySelector(`#${field.id}-error`);
    if (errorElement) {
      errorElement.textContent = message;
      field.setAttribute('aria-invalid', 'true');
    }
  }

  clearFieldError(field) {
    const errorElement = this.form.querySelector(`#${field.id}-error`);
    if (errorElement) {
      errorElement.textContent = '';
      field.setAttribute('aria-invalid', 'false');
    }
  }

  async handleSubmit(event) {
    event.preventDefault();
    
    if (this.isLoading) return;
    
    // Валидация всех полей
    let isFormValid = true;
    const fields = this.form.querySelectorAll('input[required], select[required], textarea[required]');
    
    fields.forEach(field => {
      if (!this.validateField(field)) {
        isFormValid = false;
      }
    });
    
    if (!isFormValid) {
      this.announceError('Пожалуйста, исправьте ошибки в форме');
      return;
    }
    
    this.setLoadingState(true);
    
    try {
      const formData = new FormData(this.form);
      const response = await fetch('/api/order', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        const result = await response.json();
        this.handleSuccess(result);
      } else {
        const error = await response.json();
        this.handleError(error);
      }
    } catch (error) {
      this.handleError({ message: 'Произошла ошибка при отправке заказа' });
    } finally {
      this.setLoadingState(false);
    }
  }

  setLoadingState(loading) {
    this.isLoading = loading;
    const submitBtn = this.form.querySelector('#submit-order');
    const btnText = submitBtn.querySelector('.btn-text');
    const btnLoading = submitBtn.querySelector('.btn-loading');
    
    if (loading) {
      submitBtn.disabled = true;
      btnText.style.display = 'none';
      btnLoading.style.display = 'inline';
    } else {
      submitBtn.disabled = false;
      btnText.style.display = 'inline';
      btnLoading.style.display = 'none';
    }
  }

  handleSuccess(result) {
    // Показать сообщение об успехе
    this.announceSuccess('Заказ успешно оформлен!');
    
    // Редирект или показ подтверждающего экрана
    setTimeout(() => {
      window.location.href = `/order-success/${result.orderId}`;
    }, 2000);
  }

  handleError(error) {
    this.announceError(error.message || 'Произошла ошибка при оформлении заказа');
  }

  announceSuccess(message) {
    // Создаем временный элемент для скринридеров
    const announcement = document.createElement('div');
    announcement.setAttribute('aria-live', 'polite');
    announcement.setAttribute('aria-atomic', 'true');
    announcement.className = 'sr-only';
    announcement.textContent = message;
    document.body.appendChild(announcement);
    
    setTimeout(() => {
      document.body.removeChild(announcement);
    }, 3000);
  }

  announceError(message) {
    // Создаем временный элемент для ошибок
    const announcement = document.createElement('div');
    announcement.setAttribute('role', 'alert');
    announcement.setAttribute('aria-atomic', 'true');
    announcement.className = 'sr-only';
    announcement.textContent = message;
    document.body.appendChild(announcement);
    
    setTimeout(() => {
      document.body.removeChild(announcement);
    }, 5000);
  }

  // Утилита для debounce
  debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
      const later = () => {
        clearTimeout(timeout);
        func(...args);
      };
      clearTimeout(timeout);
      timeout = setTimeout(later, wait);
    };
  }
}

// Инициализация формы заказа
document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('order-form');
  new OrderForm(form);
});
```

## Ссылки на связанные темы
- [[html/accessibility]] - Доступность HTML
- [[js/dom/forms]] - Работа с формами в JavaScript
- [[css/styling/forms]] - Стилизация форм
- [[security/validation]] - Валидация на стороне сервера

#programming #html #forms #validation #accessibility #best-practices
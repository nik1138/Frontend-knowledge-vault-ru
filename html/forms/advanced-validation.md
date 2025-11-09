# Продвинутые техники валидации HTML форм

## Введение

Продвинутая валидация HTML форм включает в себя комплексный подход к проверке пользовательских данных, объединяющий клиентскую и серверную валидацию, обеспечение доступности и улучшение пользовательского опыта. Современные веб-приложения требуют надежных систем проверки данных, которые не только защищают от некорректного ввода, но и помогают пользователям легко и интуитивно заполнять формы.

## Современные подходы к валидации

### Constraint Validation API

Constraint Validation API предоставляет программный интерфейс для проверки валидности элементов формы:

```html
<form id="constraint-form">
  <div class="form-group">
    <label for="email">Email:</label>
    <input type="email" 
           id="email" 
           name="email" 
           required 
           aria-describedby="email-error">
    <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <div class="form-group">
    <label for="password">Пароль:</label>
    <input type="password" 
           id="password" 
           name="password" 
           required 
           minlength="8" 
           aria-describedby="password-requirements password-error">
    <ul id="password-requirements" class="requirements-list">
      <li id="req-length">Не менее 8 символов</li>
      <li id="req-upper">С заглавной буквой</li>
      <li id="req-lower">Со строчной буквой</li>
      <li id="req-number">С цифрой</li>
      <li id="req-special">С специальным символом</li>
    </ul>
    <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <button type="submit">Отправить</button>
</form>

<script>
class ConstraintValidator {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.setupValidation();
  }

  setupValidation() {
    // Валидация при потере фокуса
    this.form.addEventListener('blur', (e) => {
      if (e.target.matches('input[required]')) {
        this.validateField(e.target);
      }
    }, true);

    // Валидация пароля в реальном времени
    document.getElementById('password').addEventListener('input', () => {
      this.validatePasswordRequirements();
    });

    // Отправка формы
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.validateForm();
    });
  }

  validateField(field) {
    const fieldId = field.id;
    const errorElement = document.getElementById(`${fieldId}-error`);

    if (field.validity.valid) {
      field.classList.remove('invalid');
      field.classList.add('valid');
      field.setAttribute('aria-invalid', 'false');
      errorElement.textContent = '';
      return true;
    } else {
      field.classList.remove('valid');
      field.classList.add('invalid');
      field.setAttribute('aria-invalid', 'true');

      errorElement.textContent = this.getErrorMessage(field);
      return false;
    }
  }

  validatePasswordRequirements() {
    const password = document.getElementById('password').value;
    const requirements = {
      length: password.length >= 8,
      upper: /[A-Z]/.test(password),
      lower: /[a-z]/.test(password),
      number: /[0-9]/.test(password),
      special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
    };

    // Обновляем требования
    document.getElementById('req-length').className = requirements.length ? 'met' : '';
    document.getElementById('req-upper').className = requirements.upper ? 'met' : '';
    document.getElementById('req-lower').className = requirements.lower ? 'met' : '';
    document.getElementById('req-number').className = requirements.number ? 'met' : '';
    document.getElementById('req-special').className = requirements.special ? 'met' : '';

    return Object.values(requirements).every(req => req);
  }

  validateForm() {
    let isValid = true;
    const fields = this.form.querySelectorAll('input[required]');

    fields.forEach(field => {
      if (!this.validateField(field)) {
        isValid = false;
      }
    });

    if (this.validatePasswordRequirements()) {
      isValid = isValid && true;
    } else {
      isValid = false;
    }

    if (isValid) {
      alert('Форма успешно валидирована!');
    } else {
      // Сообщение для скринридера о наличии ошибок
      const errorAnnouncement = document.createElement('div');
      errorAnnouncement.setAttribute('aria-live', 'assertive');
      errorAnnouncement.setAttribute('aria-atomic', 'true');
      errorAnnouncement.className = 'sr-only';
      errorAnnouncement.textContent = 'Форма содержит ошибки. Пожалуйста, исправьте их перед отправкой.';
      document.body.appendChild(errorAnnouncement);

      setTimeout(() => {
        if (document.body.contains(errorAnnouncement)) {
          document.body.removeChild(errorAnnouncement);
        }
      }, 5000);
    }
  }

  getErrorMessage(field) {
    if (field.validity.valueMissing) {
      return 'Это поле обязательно для заполнения';
    } else if (field.validity.typeMismatch && field.type === 'email') {
      return 'Введите действительный email адрес';
    } else if (field.validity.tooShort) {
      return `Введите не менее ${field.minLength} символов`;
    }
    return 'Неверный формат данных';
  }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
  new ConstraintValidator('constraint-form');
});
</script>
```

### Валидация с использованием регулярных выражений

```html
<form id="regex-validation-form">
  <div class="form-group">
    <label for="username">Имя пользователя:</label>
    <input type="text"
           id="username"
           name="username"
           required
           pattern="[A-Za-z0-9_]{3,20}"
           aria-describedby="username-help username-error"
           aria-invalid="false">
    <div id="username-help" class="help-text">Только буквы, цифры и подчеркивание, 3-20 символов</div>
    <div id="username-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <div class="form-group">
    <label for="phone">Телефон:</label>
    <input type="tel"
           id="phone"
           name="phone"
           pattern="^(\+7|7|8)?[\s\-]?\(?[489][0-9]{2}\)?[\s\-]?[0-9]{3}[\s\-]?[0-9]{2}[\s\-]?[0-9]{2}$"
           aria-describedby="phone-help phone-error"
           aria-invalid="false">
    <div id="phone-help" class="help-text">Формат: +7 (XXX) XXX-XX-XX</div>
    <div id="phone-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <div class="form-group">
    <label for="zip">Почтовый индекс:</label>
    <input type="text"
           id="zip"
           name="zip"
           pattern="[0-9]{6}"
           aria-describedby="zip-help zip-error"
           aria-invalid="false">
    <div id="zip-help" class="help-text">6 цифр</div>
    <div id="zip-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <button type="submit">Отправить</button>
</form>

<script>
class RegexValidator {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.patterns = {
      username: /^[A-Za-z0-9_]{3,20}$/,
      phone: /^(\+7|7|8)?[\s\-]?\(?[489][0-9]{2}\)?[\s\-]?[0-9]{3}[\s\-]?[0-9]{2}[\s\-]?[0-9]{2}$/,
      zip: /^[0-9]{6}$/
    };
    this.setupValidation();
  }

  setupValidation() {
    // Валидация при потере фокуса
    this.form.addEventListener('blur', (e) => {
      if (e.target.matches('input[pattern]')) {
        this.validateRegexField(e.target);
      }
    }, true);

    // Отправка формы
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.validateForm();
    });
  }

  validateRegexField(field) {
    const fieldId = field.id;
    const errorElement = document.getElementById(`${fieldId}-error`);
    const pattern = this.patterns[fieldId];

    if (!field.value) {
      field.classList.remove('invalid', 'valid');
      field.setAttribute('aria-invalid', 'false');
      errorElement.textContent = '';
      return true;
    }

    if (pattern && pattern.test(field.value)) {
      field.classList.remove('invalid');
      field.classList.add('valid');
      field.setAttribute('aria-invalid', 'false');
      errorElement.textContent = '';
      return true;
    } else {
      field.classList.remove('valid');
      field.classList.add('invalid');
      field.setAttribute('aria-invalid', 'true');
      errorElement.textContent = this.getPatternErrorMessage(field);
      return false;
    }
  }

  getPatternErrorMessage(field) {
    const fieldId = field.id;
    switch(fieldId) {
      case 'username':
        return 'Имя пользователя должно содержать 3-20 символов, только буквы, цифры и подчеркивание';
      case 'phone':
        return 'Введите номер в формате +7 (XXX) XXX-XX-XX';
      case 'zip':
        return 'Почтовый индекс должен содержать 6 цифр';
      default:
        return 'Неверный формат данных';
    }
  }

  validateForm() {
    let isValid = true;
    const fields = this.form.querySelectorAll('input[pattern]');

    fields.forEach(field => {
      if (!this.validateRegexField(field)) {
        isValid = false;
      }
    });

    if (isValid) {
      alert('Форма успешно валидирована!');
    } else {
      alert('Пожалуйста, исправьте ошибки в форме');
    }
  }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
  new RegexValidator('regex-validation-form');
});
</script>
```

## Асинхронная валидация

### Валидация уникальности данных

```html
<form id="async-validation-form">
  <div class="form-group">
    <label for="async-email">Email:</label>
    <input type="email"
           id="async-email"
           name="email"
           required
           aria-describedby="async-email-status async-email-error"
           aria-invalid="false">
    <div id="async-email-status" class="status" aria-live="polite"></div>
    <div id="async-email-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <div class="form-group">
    <label for="async-username">Имя пользователя:</label>
    <input type="text"
           id="async-username"
           name="username"
           required
           minlength="3"
           maxlength="20"
           pattern="[A-Za-z0-9_]+"
           aria-describedby="async-username-status async-username-error"
           aria-invalid="false">
    <div id="async-username-status" class="status" aria-live="polite"></div>
    <div id="async-username-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <button type="submit" id="submit-btn">Отправить</button>
</form>

<script>
class AsyncValidator {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.debounceTimers = {};
    this.setupValidation();
  }

  setupValidation() {
    // Асинхронная валидация email
    document.getElementById('async-email').addEventListener('input', this.debounce((e) => {
      this.validateEmailUniqueness(e.target);
    }, 500));

    // Асинхронная валидация имени пользователя
    document.getElementById('async-username').addEventListener('input', this.debounce((e) => {
      this.validateUsernameUniqueness(e.target);
    }, 500));

    // Отправка формы
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.validateForm();
    });
  }

  debounce(func, wait) {
    return (...args) => {
      const later = () => {
        clearTimeout(this.debounceTimers[func.name]);
        func.apply(this, args);
      };
      clearTimeout(this.debounceTimers[func.name]);
      this.debounceTimers[func.name] = setTimeout(later, wait);
    };
  }

  async validateEmailUniqueness(field) {
    const email = field.value.trim();
    const statusElement = document.getElementById('async-email-status');
    const errorElement = document.getElementById('async-email-error');

    if (!email) {
      statusElement.textContent = '';
      errorElement.textContent = '';
      field.classList.remove('invalid', 'valid');
      field.setAttribute('aria-invalid', 'false');
      return true;
    }

    if (!this.isValidEmail(email)) {
      errorElement.textContent = 'Введите действительный email';
      field.classList.add('invalid');
      field.classList.remove('valid');
      field.setAttribute('aria-invalid', 'true');
      statusElement.textContent = '';
      return false;
    }

    // Показываем индикатор проверки
    statusElement.textContent = 'Проверка...';
    field.classList.remove('invalid', 'valid');
    field.setAttribute('aria-invalid', 'false');

    try {
      // Симуляция API вызова
      const isUnique = await this.checkEmailUniqueness(email);
      
      if (isUnique) {
        field.classList.remove('invalid');
        field.classList.add('valid');
        field.setAttribute('aria-invalid', 'false');
        errorElement.textContent = '';
        statusElement.textContent = 'Email доступен';
      } else {
        field.classList.add('invalid');
        field.classList.remove('valid');
        field.setAttribute('aria-invalid', 'true');
        errorElement.textContent = 'Email уже используется';
        statusElement.textContent = '';
      }
    } catch (error) {
      field.classList.add('invalid');
      field.setAttribute('aria-invalid', 'true');
      errorElement.textContent = 'Ошибка проверки email';
      statusElement.textContent = '';
    }
  }

  async validateUsernameUniqueness(field) {
    const username = field.value.trim();
    const statusElement = document.getElementById('async-username-status');
    const errorElement = document.getElementById('async-username-error');

    if (!username) {
      statusElement.textContent = '';
      errorElement.textContent = '';
      field.classList.remove('invalid', 'valid');
      field.setAttribute('aria-invalid', 'false');
      return true;
    }

    if (username.length < 3) {
      errorElement.textContent = 'Имя пользователя должно быть не менее 3 символов';
      field.classList.add('invalid');
      field.classList.remove('valid');
      field.setAttribute('aria-invalid', 'true');
      statusElement.textContent = '';
      return false;
    }

    if (!/^[A-Za-z0-9_]+$/.test(username)) {
      errorElement.textContent = 'Только буквы, цифры и подчеркивание';
      field.classList.add('invalid');
      field.classList.remove('valid');
      field.setAttribute('aria-invalid', 'true');
      statusElement.textContent = '';
      return false;
    }

    // Показываем индикатор проверки
    statusElement.textContent = 'Проверка...';
    field.classList.remove('invalid', 'valid');
    field.setAttribute('aria-invalid', 'false');

    try {
      // Симуляция API вызова
      const isUnique = await this.checkUsernameUniqueness(username);
      
      if (isUnique) {
        field.classList.remove('invalid');
        field.classList.add('valid');
        field.setAttribute('aria-invalid', 'false');
        errorElement.textContent = '';
        statusElement.textContent = 'Имя пользователя доступно';
      } else {
        field.classList.add('invalid');
        field.classList.remove('valid');
        field.setAttribute('aria-invalid', 'true');
        errorElement.textContent = 'Имя пользователя уже занято';
        statusElement.textContent = '';
      }
    } catch (error) {
      field.classList.add('invalid');
      field.setAttribute('aria-invalid', 'true');
      errorElement.textContent = 'Ошибка проверки имени пользователя';
      statusElement.textContent = '';
    }
  }

  async checkEmailUniqueness(email) {
    // Симуляция проверки на сервере
    return new Promise(resolve => {
      setTimeout(() => {
        // Список занятых email для демонстрации
        resolve(!['admin@example.com', 'test@example.com', 'user@example.com'].includes(email));
      }, 800);
    });
  }

  async checkUsernameUniqueness(username) {
    // Симуляция проверки на сервере
    return new Promise(resolve => {
      setTimeout(() => {
        // Список занятых имен для демонстрации
        resolve(!['admin', 'user', 'test', 'root', 'guest'].includes(username.toLowerCase()));
      }, 800);
    });
  }

  isValidEmail(email) {
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return re.test(email);
  }

  async validateForm() {
    const submitBtn = document.getElementById('submit-btn');
    const originalText = submitBtn.textContent;
    
    submitBtn.textContent = 'Проверка...';
    submitBtn.disabled = true;

    try {
      // Проверяем валидность всех полей
      const emailField = document.getElementById('async-email');
      const usernameField = document.getElementById('async-username');
      
      // Дожидаемся завершения асинхронных проверок
      await new Promise(resolve => setTimeout(resolve, 100));
      
      const isEmailValid = emailField.classList.contains('valid');
      const isUsernameValid = usernameField.classList.contains('valid');
      
      if (isEmailValid && isUsernameValid) {
        alert('Форма успешно валидирована и готова к отправке!');
      } else {
        alert('Пожалуйста, исправьте ошибки в форме');
      }
    } finally {
      submitBtn.textContent = originalText;
      submitBtn.disabled = false;
    }
  }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
  new AsyncValidator('async-validation-form');
});
</script>
```

## Комплексная валидация с пользовательским опытом

### Валидация с индикаторами сложности пароля

```html
<form id="password-strength-form">
  <div class="form-group">
    <label for="password-strength">Пароль:</label>
    <input type="password"
           id="password-strength"
           name="password"
           required
           aria-describedby="password-strength-meter password-requirements password-error"
           aria-invalid="false">
    <div id="password-strength-meter" class="strength-meter">
      <div class="strength-bar">
        <div class="strength-level" id="strength-level"></div>
      </div>
      <div class="strength-text" id="strength-text">Слабый</div>
    </div>
    <ul id="password-requirements" class="requirements-list">
      <li id="req-min-length">Минимум 8 символов</li>
      <li id="req-upper">Заглавная буква</li>
      <li id="req-lower">Строчная буква</li>
      <li id="req-number">Цифра</li>
      <li id="req-special">Специальный символ</li>
    </ul>
    <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <div class="form-group">
    <label for="confirm-password-strength">Подтверждение пароля:</label>
    <input type="password"
           id="confirm-password-strength"
           name="confirm_password"
           required
           aria-describedby="confirm-password-error"
           aria-invalid="false">
    <div id="confirm-password-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <button type="submit">Зарегистрироваться</button>
</form>

<script>
class PasswordStrengthValidator {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.setupValidation();
  }

  setupValidation() {
    // Валидация сложности пароля в реальном времени
    document.getElementById('password-strength').addEventListener('input', () => {
      this.validatePasswordStrength();
    });

    // Валидация совпадения паролей
    document.getElementById('confirm-password-strength').addEventListener('input', () => {
      this.validatePasswordMatch();
    });

    // Отправка формы
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.validateForm();
    });
  }

  validatePasswordStrength() {
    const password = document.getElementById('password-strength').value;
    const strengthLevel = document.getElementById('strength-level');
    const strengthText = document.getElementById('strength-text');
    const errorElement = document.getElementById('password-error');

    if (!password) {
      strengthLevel.style.width = '0%';
      strengthLevel.className = 'strength-level';
      strengthText.textContent = '';
      errorElement.textContent = '';
      this.updateRequirements({});
      return 0;
    }

    const score = this.calculatePasswordStrength(password);
    const strength = this.getStrengthLevel(score);

    // Обновляем визуальный индикатор
    strengthLevel.style.width = `${score * 20}%`;
    strengthLevel.className = `strength-level ${strength.level}`;
    strengthText.textContent = strength.text;

    // Обновляем требования
    this.updateRequirements(this.getPasswordRequirements(password));

    // Проверяем, достаточно ли сложный пароль
    if (score < 3) {
      errorElement.textContent = 'Пароль слишком слабый. Используйте более сложный пароль.';
      return score;
    } else {
      errorElement.textContent = '';
      return score;
    }
  }

  calculatePasswordStrength(password) {
    let score = 0;

    // Длина пароля
    if (password.length >= 8) score += 1;
    if (password.length >= 12) score += 1;

    // Наличие заглавных букв
    if (/[A-Z]/.test(password)) score += 1;

    // Наличие строчных букв
    if (/[a-z]/.test(password)) score += 1;

    // Наличие цифр
    if (/[0-9]/.test(password)) score += 1;

    // Наличие специальных символов
    if (/[!@#$%^&*(),.?":{}|<>]/.test(password)) score += 1;

    // Разнообразие символов
    const uniqueChars = new Set(password).size;
    if (uniqueChars > password.length * 0.7) score += 1;

    return Math.min(score, 5);
  }

  getStrengthLevel(score) {
    if (score < 2) return { level: 'weak', text: 'Слабый' };
    if (score < 3) return { level: 'fair', text: 'Удовлетворительный' };
    if (score < 4) return { level: 'good', text: 'Хороший' };
    return { level: 'strong', text: 'Надежный' };
  }

  getPasswordRequirements(password) {
    return {
      minLength: password.length >= 8,
      upper: /[A-Z]/.test(password),
      lower: /[a-z]/.test(password),
      number: /[0-9]/.test(password),
      special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
    };
  }

  updateRequirements(requirements) {
    document.getElementById('req-min-length').className = requirements.minLength ? 'met' : '';
    document.getElementById('req-upper').className = requirements.upper ? 'met' : '';
    document.getElementById('req-lower').className = requirements.lower ? 'met' : '';
    document.getElementById('req-number').className = requirements.number ? 'met' : '';
    document.getElementById('req-special').className = requirements.special ? 'met' : '';
  }

  validatePasswordMatch() {
    const password = document.getElementById('password-strength').value;
    const confirmPassword = document.getElementById('confirm-password-strength').value;
    const errorElement = document.getElementById('confirm-password-error');

    if (!confirmPassword) {
      errorElement.textContent = '';
      document.getElementById('confirm-password-strength').classList.remove('invalid', 'valid');
      document.getElementById('confirm-password-strength').setAttribute('aria-invalid', 'false');
      return true;
    }

    if (password !== confirmPassword) {
      errorElement.textContent = 'Пароли не совпадают';
      document.getElementById('confirm-password-strength').classList.add('invalid');
      document.getElementById('confirm-password-strength').classList.remove('valid');
      document.getElementById('confirm-password-strength').setAttribute('aria-invalid', 'true');
      return false;
    } else {
      errorElement.textContent = '';
      document.getElementById('confirm-password-strength').classList.remove('invalid');
      document.getElementById('confirm-password-strength').classList.add('valid');
      document.getElementById('confirm-password-strength').setAttribute('aria-invalid', 'false');
      return true;
    }
  }

  validateForm() {
    const passwordField = document.getElementById('password-strength');
    const passwordScore = this.calculatePasswordStrength(passwordField.value);
    const isPasswordMatch = this.validatePasswordMatch();

    if (passwordScore >= 3 && isPasswordMatch) {
      alert('Пароль удовлетворяет требованиям безопасности!');
    } else {
      if (passwordScore < 3) {
        alert('Пожалуйста, используйте более надежный пароль');
      }
      if (!isPasswordMatch) {
        alert('Пароли не совпадают');
      }
    }
  }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
  new PasswordStrengthValidator('password-strength-form');
});
</script>

<style>
.strength-meter {
  margin: 10px 0;
}

.strength-bar {
  width: 100%;
  height: 10px;
  background-color: #e0e0e0;
  border-radius: 5px;
  overflow: hidden;
}

.strength-level {
  height: 100%;
  width: 0%;
  transition: width 0.3s ease, background-color 0.3s ease;
}

.strength-level.weak { background-color: #f44336; }
.strength-level.fair { background-color: #ff9800; }
.strength-level.good { background-color: #ffc107; }
.strength-level.strong { background-color: #4caf50; }

.strength-text {
  font-size: 0.8em;
  margin-top: 5px;
  font-weight: bold;
}

.requirements-list {
  list-style: none;
  padding: 0;
  margin: 10px 0;
}

.requirements-list li {
  padding: 2px 0;
  font-size: 0.9em;
  color: #666;
}

.requirements-list li.met {
  color: #4caf50;
  font-weight: bold;
}
</style>
```

## Валидация с машинным обучением (симуляция)

### Интеллектуальная валидация

```html
<form id="ml-validation-form">
  <div class="form-group">
    <label for="ml-email">Email:</label>
    <input type="email"
           id="ml-email"
           name="email"
           required
           aria-describedby="ml-email-confidence ml-email-error"
           aria-invalid="false">
    <div id="ml-email-confidence" class="confidence-indicator" aria-live="polite"></div>
    <div id="ml-email-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <div class="form-group">
    <label for="ml-password">Пароль:</label>
    <input type="password"
           id="ml-password"
           name="password"
           required
           aria-describedby="ml-password-confidence ml-password-error"
           aria-invalid="false">
    <div id="ml-password-confidence" class="confidence-indicator" aria-live="polite"></div>
    <div id="ml-password-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <button type="submit">Отправить</button>
</form>

<script>
class MLValidator {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.setupValidation();
  }

  setupValidation() {
    // ML-валидация email
    document.getElementById('ml-email').addEventListener('input', (e) => {
      this.mlValidateEmail(e.target);
    });

    // ML-валидация пароля
    document.getElementById('ml-password').addEventListener('input', (e) => {
      this.mlValidatePassword(e.target);
    });

    // Отправка формы
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.validateForm();
    });
  }

  async mlValidateEmail(field) {
    const email = field.value;
    const confidenceElement = document.getElementById('ml-email-confidence');
    const errorElement = document.getElementById('ml-email-error');

    if (!email) {
      confidenceElement.textContent = '';
      errorElement.textContent = '';
      field.classList.remove('invalid', 'valid');
      field.setAttribute('aria-invalid', 'false');
      return true;
    }

    // Показываем индикатор "анализа"
    confidenceElement.textContent = 'Анализ...';
    field.classList.remove('invalid', 'valid');
    field.setAttribute('aria-invalid', 'false');

    // Симуляция ML-анализа
    const result = await this.simulateEmailAnalysis(email);

    confidenceElement.innerHTML = `
      <span class="${result.valid ? 'valid-confidence' : 'invalid-confidence'}">
        Доверие: ${(result.confidence * 100).toFixed(0)}%
      </span>
      <span class="analysis-result">${result.message}</span>
    `;

    if (result.valid) {
      field.classList.remove('invalid');
      field.classList.add('valid');
      field.setAttribute('aria-invalid', 'false');
      errorElement.textContent = '';
    } else {
      field.classList.add('invalid');
      field.classList.remove('valid');
      field.setAttribute('aria-invalid', 'true');
      errorElement.textContent = result.message;
    }

    return result.valid;
  }

  async mlValidatePassword(field) {
    const password = field.value;
    const confidenceElement = document.getElementById('ml-password-confidence');
    const errorElement = document.getElementById('ml-password-error');

    if (!password) {
      confidenceElement.textContent = '';
      errorElement.textContent = '';
      field.classList.remove('invalid', 'valid');
      field.setAttribute('aria-invalid', 'false');
      return true;
    }

    // Показываем индикатор "анализа"
    confidenceElement.textContent = 'Анализ...';
    field.classList.remove('invalid', 'valid');
    field.setAttribute('aria-invalid', 'false');

    // Симуляция ML-анализа
    const result = await this.simulatePasswordAnalysis(password);

    confidenceElement.innerHTML = `
      <span class="${result.valid ? 'valid-confidence' : 'invalid-confidence'}">
        Надежность: ${(result.confidence * 100).toFixed(0)}%
      </span>
      <span class="analysis-result">${result.message}</span>
    `;

    if (result.valid) {
      field.classList.remove('invalid');
      field.classList.add('valid');
      field.setAttribute('aria-invalid', 'false');
      errorElement.textContent = '';
    } else {
      field.classList.add('invalid');
      field.classList.remove('valid');
      field.setAttribute('aria-invalid', 'true');
      errorElement.textContent = result.message;
    }

    return result.valid;
  }

  simulateEmailAnalysis(email) {
    return new Promise(resolve => {
      setTimeout(() => {
        // Симуляция ML-модели для email
        const isDisposable = /temp|mailinator|guerrillamail|10minutemail/i.test(email);
        const hasValidDomain = /\.[a-zA-Z]{2,}$/.test(email);
        const isCorporate = /\.(com|org|net|edu|gov|mil|int)$/.test(email);
        
        if (isDisposable) {
          resolve({
            valid: false,
            confidence: 0.95,
            message: 'Временные email-адреса не разрешены'
          });
        } else if (!hasValidDomain) {
          resolve({
            valid: false,
            confidence: 0.85,
            message: 'Недействительный домен'
          });
        } else {
          resolve({
            valid: true,
            confidence: isCorporate ? 0.95 : 0.88,
            message: isCorporate ? 'Корпоративный email, высокое доверие' : 'Действительный email'
          });
        }
      }, 600);
    });
  }

  simulatePasswordAnalysis(password) {
    return new Promise(resolve => {
      setTimeout(() => {
        // Симуляция ML-модели для пароля
        const commonPatterns = [
          /123456/, /password/, /qwerty/, /abc/, /admin/, 
          /000000/, /111111/, /999999/, /123123/, /7777777/
        ];
        
        const hasCommonPattern = commonPatterns.some(pattern => pattern.test(password.toLowerCase()));
        const entropy = this.calculatePasswordEntropy(password);
        
        if (hasCommonPattern) {
          resolve({
            valid: false,
            confidence: 0.98,
            message: 'Пароль содержит распространенные шаблоны'
          });
        } else if (entropy < 2.0) {
          resolve({
            valid: false,
            confidence: 0.88,
            message: 'Слишком предсказуемый пароль'
          });
        } else if (entropy < 3.0) {
          resolve({
            valid: true,
            confidence: 0.75,
            message: 'Пароль удовлетворительный'
          });
        } else {
          resolve({
            valid: true,
            confidence: entropy > 4.0 ? 0.95 : 0.85,
            message: entropy > 4.0 ? 'Очень надежный пароль' : 'Надежный пароль'
          });
        }
      }, 600);
    });
  }

  calculatePasswordEntropy(password) {
    // Упрощенный расчет энтропии пароля
    const charsetSize = this.getPasswordCharsetSize(password);
    const entropy = password.length * Math.log2(charsetSize);
    return entropy / password.length; // Нормализуем на длину пароля
  }

  getPasswordCharsetSize(password) {
    let size = 0;
    if (/[a-z]/.test(password)) size += 26; // строчные буквы
    if (/[A-Z]/.test(password)) size += 26; // заглавные буквы
    if (/[0-9]/.test(password)) size += 10; // цифры
    if (/[^a-zA-Z0-9]/.test(password)) size += 32; // специальные символы
    return size;
  }

  async validateForm() {
    const emailField = document.getElementById('ml-email');
    const passwordField = document.getElementById('ml-password');
    
    // Асинхронно валидируем оба поля
    const [emailValid, passwordValid] = await Promise.all([
      this.mlValidateEmail(emailField),
      this.mlValidatePassword(passwordField)
    ]);
    
    if (emailValid && passwordValid) {
      alert('Форма успешно валидирована с использованием ML-анализа!');
    } else {
      alert('Пожалуйста, исправьте ошибки в форме');
    }
  }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
  new MLValidator('ml-validation-form');
});
</script>

<style>
.confidence-indicator {
  font-size: 0.85em;
  margin-top: 5px;
}

.valid-confidence {
  color: #4caf50;
  font-weight: bold;
}

.invalid-confidence {
  color: #f44336;
  font-weight: bold;
}

.analysis-result {
  margin-left: 10px;
  color: #666;
}
</style>
```

## Лучшие практики продвинутой валидации

### 1. Комбинированная валидация (клиент + сервер)

```html
<form id="combined-validation-form">
  <div class="form-group">
    <label for="combined-email">Email:</label>
    <input type="email"
           id="combined-email"
           name="email"
           required
           aria-describedby="combined-email-error"
           aria-invalid="false">
    <div id="combined-email-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <div class="form-group">
    <label for="combined-captcha">Капча:</label>
    <div class="captcha-container">
      <img id="captcha-image" src="data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTIwIiBoZWlnaHQ9IjQwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjxyZWN0IHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiIGZpbGw9IiNmZmYiLz48Y2lyY2xlIGN4PSIzMCIgY3k9IjIwIiByPSIxMCIgZmlsbD0iI2NjYyIvPjxjaXJjbGUgY3g9IjkwIiBjeT0iMjAiIHI9IjEwIiBmaWxsPSIjY2NjIi8+PHRleHQgeD0iNTAiIHk9IjI1IiBmb250LWZhbWlseT0iQXJpYWwiIGZvbnQtc2l6ZT0iMTgiIGZpbGw9IiM2NjYiIHRleHQtYW5jaG9yPSJtaWRkbGUiPjEyMzQ1PC90ZXh0Pjwvc3ZnPg==" alt="Капча">
      <button type="button" onclick="refreshCaptcha()">🔄</button>
    </div>
    <input type="text"
           id="combined-captcha"
           name="captcha"
           required
           maxlength="5"
           aria-describedby="combined-captcha-error"
           aria-invalid="false">
    <div id="combined-captcha-error" class="error-message" role="alert" aria-live="assertive"></div>
  </div>

  <button type="submit">Отправить</button>
</form>

<script>
class CombinedValidator {
  constructor(formId) {
    this.form = document.getElementById(formId);
    this.setupValidation();
  }

  setupValidation() {
    // Клиентская валидация
    document.getElementById('combined-email').addEventListener('blur', (e) => {
      this.validateEmailClient(e.target);
    });

    document.getElementById('combined-captcha').addEventListener('input', (e) => {
      if (e.target.value.length === 5) {
        this.validateCaptcha(e.target);
      }
    });

    // Отправка формы с серверной валидацией
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.validateFormCombined();
    });
  }

  validateEmailClient(field) {
    const email = field.value.trim();
    const errorElement = document.getElementById('combined-email-error');

    if (!email) {
      this.showError(field, errorElement, 'Email обязателен');
      return false;
    }

    if (!this.isValidEmail(email)) {
      this.showError(field, errorElement, 'Введите действительный email');
      return false;
    }

    this.clearError(field, errorElement);
    return true;
  }

  validateCaptcha(field) {
    const captcha = field.value.trim();
    const errorElement = document.getElementById('combined-captcha-error');

    if (!captcha) {
      this.clearError(field, errorElement);
      return true;
    }

    if (captcha.length !== 5) {
      this.showError(field, errorElement, 'Введите 5 символов капчи');
      return false;
    }

    // Симуляция проверки капчи
    if (captcha.toUpperCase() !== '12345') { // В реальном приложении - серверная проверка
      this.showError(field, errorElement, 'Неверная капча. Попробуйте снова.');
      refreshCaptcha();
      return false;
    }

    this.clearError(field, errorElement);
    return true;
  }

  showError(field, errorElement, message) {
    field.classList.add('invalid');
    field.setAttribute('aria-invalid', 'true');
    errorElement.textContent = message;
  }

  clearError(field, errorElement) {
    field.classList.remove('invalid');
    field.setAttribute('aria-invalid', 'false');
    errorElement.textContent = '';
  }

  isValidEmail(email) {
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return re.test(email);
  }

  async validateFormCombined() {
    const emailField = document.getElementById('combined-email');
    const captchaField = document.getElementById('combined-captcha');
    
    const isEmailValid = this.validateEmailClient(emailField);
    const isCaptchaValid = this.validateCaptcha(captchaField);

    if (isEmailValid && isCaptchaValid) {
      // Серверная валидация
      try {
        const response = await fetch('/api/validate-form', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            email: emailField.value,
            captcha: captchaField.value
          })
        });

        const result = await response.json();

        if (result.valid) {
          alert('Форма успешно валидирована и отправлена!');
        } else {
          if (result.errors && result.errors.email) {
            this.showError(emailField, 
                         document.getElementById('combined-email-error'), 
                         result.errors.email);
          }
          if (result.errors && result.errors.captcha) {
            this.showError(captchaField, 
                         document.getElementById('combined-captcha-error'), 
                         result.errors.captcha);
            refreshCaptcha();
          }
        }
      } catch (error) {
        alert('Ошибка при проверке данных на сервере');
      }
    }
  }
}

function refreshCaptcha() {
  const captchaImage = document.getElementById('captcha-image');
  // В реальном приложении - новый URL с серверной генерацией
  const svg = `data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTIwIiBoZWlnaHQ9IjQwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjxyZWN0IHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiIGZpbGw9IiNmZmYiLz48Y2lyY2xlIGN4PSIzMCIgY3k9IjIwIiByPSIxMCIgZmlsbD0iI2NjYyIvPjxjaXJjbGUgY3g9IjkwIiBjeT0iMjAiIHI9IjEwIiBmaWxsPSIjY2NjIi8+PHRleHQgeD0iNTAiIHk9IjI1IiBmb250LWZhbWlseT0iQXJpYWwiIGZvbnQtc2l6ZT0iMTgiIGZpbGw9IiM2NjYiIHRleHQtYW5jaG9yPSJtaWRkbGUiPj${Math.floor(Math.random()*90000)+10000}</text></svg>`;
  captchaImage.src = svg;
  document.getElementById('combined-captcha').value = '';
  document.getElementById('combined-captcha').classList.remove('valid', 'invalid');
  document.getElementById('combined-captcha-error').textContent = '';
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
  new CombinedValidator('combined-validation-form');
});
</script>
```

## Заключение

Продвинутая валидация HTML форм включает в себя:

1. Использование Constraint Validation API для стандартной валидации
2. Применение регулярных выражений для сложных проверок
3. Асинхронную валидацию для проверки уникальности данных
4. Индикаторы сложности пароля и пользовательский опыт
5. Комбинированную валидацию (клиент + сервер)
6. Обеспечение доступности для всех пользователей
7. Понятные и информативные сообщения об ошибках

Эти техники позволяют создавать формы, которые не только эффективно проверяют данные, но и обеспечивают положительный опыт взаимодействия для пользователей.

## Следующие темы
- [[HTML/Формы/Доступность]]
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Формы/Обработка данных]]

## Теги
#html #forms #validation #advanced-validation #web-development #frontend #user-experience #accessibility #javascript #security #constraint-validation-api #regex-validation #async-validation #password-strength #machine-learning-validation
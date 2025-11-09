# HTML Формы: Валидация

Валидация HTML форм - это процесс проверки корректности введенных пользователем данных перед отправкой формы. HTML предоставляет встроенные средства валидации, которые помогают обеспечить целостность данных и улучшить пользовательский опыт.

## Встроенные атрибуты валидации

HTML5 предоставляет несколько атрибутов для валидации данных на стороне клиента:

### Обязательные поля

Атрибут `required` указывает, что поле должно быть заполнено:

```html
<form>
    <label for="name">Имя:</label>
    <input type="text" id="name" name="name" required>
    
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <button type="submit">Отправить</button>
</form>
```

### Ограничения длины

Атрибуты `minlength` и `maxlength` ограничивают количество символов:

```html
<input type="text" 
       name="username" 
       required 
       minlength="3" 
       maxlength="20" 
       placeholder="3-20 символов">
```

### Числовые ограничения

Для числовых полей используются атрибуты `min`, `max` и `step`:

```html
<input type="number" 
       name="age" 
       min="1" 
       max="120" 
       value="18">

<input type="range" 
       name="rating" 
       min="1" 
       max="10" 
       step="1" 
       value="5">
```

### Проверка по регулярному выражению

Атрибут `pattern` позволяет задать регулярное выражение для проверки:

```html
<input type="text" 
       name="phone" 
       pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}" 
       placeholder="123-456-7890"
       title="Формат: 123-456-7890">

<input type="text" 
       name="username" 
       pattern="[A-Za-z0-9_]{3,16}" 
       placeholder="Только буквы, цифры и подчеркивание"
       title="3-16 символов, только буквы, цифры и подчеркивание">
```

### Уникальные проверки для типов полей

Различные типы полей имеют встроенную валидацию:

```html
<input type="email" name="email" required placeholder="user@example.com">
<!-- Автоматически проверяет формат email -->

<input type="url" name="website" placeholder="https://example.com">
<!-- Автоматически проверяет формат URL -->

<input type="date" name="birthdate" min="1900-01-01" max="2023-12-31">
<!-- Ограничивает диапазон дат -->
```

## Визуальная обратная связь

### CSS псевдоклассы для валидации

HTML предоставляет псевдоклассы для стилизации полей в зависимости от их состояния:

```css
/* Стили для действительных полей */
input:valid {
    border: 2px solid #28a745;
    background-color: #f8fff9;
}

/* Стили для недействительных полей */
input:invalid {
    border: 2px solid #dc3545;
    background-color: #fff8f8;
}

/* Стили для полей, когда они сфокусированы */
input:focus:invalid {
    outline: 2px solid #dc3545;
    outline-offset: 2px;
}

/* Стили для обязательных полей */
input:required {
    border-left: 4px solid #007bff;
}

/* Стили для необязательных полей */
input:not(:required) {
    border-left: 4px solid #6c757d;
}
```

### Псевдоклассы для различных состояний

```css
/* Поле в фокусе */
input:focus {
    box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
}

/* Поле было сфокусировано и потеряло фокус */
input:focus:valid {
    border-color: #28a745;
}

input:focus:invalid {
    border-color: #dc3545;
}

/* После отправки формы */
form:invalid input {
    animation: shake 0.5s;
}

@keyframes shake {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-5px); }
    75% { transform: translateX(5px); }
}
```

## Кастомные сообщения об ошибках

### Атрибут `title`

Атрибут `title` предоставляет подсказку при ошибке валидации:

```html
<input type="text" 
       name="phone" 
       pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}" 
       title="Пожалуйста, введите номер в формате: 123-456-7890">
```

### JavaScript для кастомных сообщений

```html
<form id="registration-form">
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    <div class="error-message" id="email-error"></div>
    
    <button type="submit">Отправить</button>
</form>

<script>
document.getElementById('registration-form').addEventListener('submit', function(e) {
    const emailInput = document.getElementById('email');
    const errorDiv = document.getElementById('email-error');
    
    // Очистить предыдущие ошибки
    errorDiv.textContent = '';
    
    // Проверить email
    if (!emailInput.validity.valid) {
        e.preventDefault(); // Остановить отправку формы
        
        if (emailInput.validity.valueMissing) {
            errorDiv.textContent = 'Пожалуйста, введите email';
        } else if (emailInput.validity.typeMismatch) {
            errorDiv.textContent = 'Пожалуйста, введите действительный email';
        }
        
        // Стилизовать поле как ошибочное
        emailInput.style.borderColor = '#dc3545';
    }
});
</script>
```

## События валидации

HTML предоставляет события, связанные с валидацией:

```javascript
const form = document.getElementById('myForm');
const input = document.getElementById('myInput');

// Событие при потере фокуса (проверка валидности)
input.addEventListener('blur', function() {
    if (!this.validity.valid) {
        this.style.borderColor = '#dc3545';
    }
});

// Событие перед отправкой формы
form.addEventListener('submit', function(e) {
    if (!this.checkValidity()) {
        e.preventDefault(); // Остановить отправку
        alert('Пожалуйста, исправьте ошибки в форме');
    }
});

// Проверить валидность в реальном времени
input.addEventListener('input', function() {
    if (this.validity.valid) {
        this.style.borderColor = '#28a745';
    } else {
        this.style.borderColor = '#dc3545';
    }
});
```

## Пример формы с комплексной валидацией

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Форма с валидацией</title>
    <style>
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 8px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        
        input:valid {
            border-color: #28a745;
        }
        
        input:invalid {
            border-color: #dc3545;
        }
        
        input:focus {
            outline: none;
            box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
        }
        
        .error-message {
            color: #dc3545;
            font-size: 14px;
            margin-top: 5px;
        }
        
        .success-message {
            color: #28a745;
            font-size: 14px;
            margin-top: 5px;
        }
        
        button {
            background-color: #007bff;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button:hover {
            background-color: #0056b3;
        }
        
        button:disabled {
            background-color: #6c757d;
            cursor: not-allowed;
        }
    </style>
</head>
<body>
    <h1>Регистрация пользователя</h1>
    
    <form id="registrationForm">
        <div class="form-group">
            <label for="fullName">Полное имя:</label>
            <input type="text" 
                   id="fullName" 
                   name="fullName" 
                   required 
                   minlength="2" 
                   maxlength="50"
                   placeholder="Иван Иванов">
        </div>
        
        <div class="form-group">
            <label for="email">Email:</label>
            <input type="email" 
                   id="email" 
                   name="email" 
                   required
                   placeholder="user@example.com">
            <div class="error-message" id="email-error"></div>
        </div>
        
        <div class="form-group">
            <label for="phone">Телефон:</label>
            <input type="tel" 
                   id="phone" 
                   name="phone" 
                   required
                   pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
                   placeholder="123-456-7890"
                   title="Введите телефон в формате: 123-456-7890">
            <div class="error-message" id="phone-error"></div>
        </div>
        
        <div class="form-group">
            <label for="password">Пароль:</label>
            <input type="password" 
                   id="password" 
                   name="password" 
                   required
                   minlength="8"
                   placeholder="Минимум 8 символов">
        </div>
        
        <div class="form-group">
            <label for="confirmPassword">Подтверждение пароля:</label>
            <input type="password" 
                   id="confirmPassword" 
                   name="confirmPassword" 
                   required
                   placeholder="Повторите пароль">
            <div class="error-message" id="password-error"></div>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox" 
                       id="terms" 
                       name="terms" 
                       required>
                Я согласен с условиями использования
            </label>
        </div>
        
        <button type="submit">Зарегистрироваться</button>
    </form>

    <script>
        document.getElementById('registrationForm').addEventListener('submit', function(e) {
            e.preventDefault();
            
            // Сброс ошибок
            document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
            document.querySelectorAll('input').forEach(el => el.style.borderColor = '#ddd');
            
            let isValid = true;
            
            // Проверка подтверждения пароля
            const password = document.getElementById('password').value;
            const confirmPassword = document.getElementById('confirmPassword').value;
            
            if (password !== confirmPassword) {
                document.getElementById('password-error').textContent = 'Пароли не совпадают';
                document.getElementById('confirmPassword').style.borderColor = '#dc3545';
                isValid = false;
            }
            
            // Проверка валидности всей формы
            if (this.checkValidity() && isValid) {
                alert('Форма успешно заполнена!');
                // Здесь можно отправить данные на сервер
            } else {
                alert('Пожалуйста, исправьте ошибки в форме');
            }
        });
        
        // Реальная проверка подтверждения пароля
        document.getElementById('confirmPassword').addEventListener('input', function() {
            const password = document.getElementById('password').value;
            const confirmPassword = this.value;
            
            if (password && confirmPassword && password !== confirmPassword) {
                document.getElementById('password-error').textContent = 'Пароли не совпадают';
                this.style.borderColor = '#dc3545';
            } else if (password === confirmPassword && password !== '') {
                document.getElementById('password-error').textContent = '';
                this.style.borderColor = '#28a745';
            } else {
                document.getElementById('password-error').textContent = '';
            }
        });
    </script>
</body>
</html>
```

## Отключение валидации

Валидацию можно отключить с помощью атрибута `novalidate`:

```html
<form action="/submit" method="post" novalidate>
    <!-- Валидация отключена для всей формы -->
    <input type="email" name="email" required>
    <button type="submit">Отправить</button>
</form>

<!-- Или для отдельных кнопок -->
<form action="/submit" method="post">
    <input type="email" name="email" required>
    <button type="submit" formnovalidate>Отправить без проверки</button>
    <button type="submit">Отправить с проверкой</button>
</form>
```

## Заключение

Валидация HTML форм важна для обеспечения корректности пользовательских данных и улучшения пользовательского опыта. Использование встроенных атрибутов валидации в сочетании с CSS и JavaScript позволяет создавать формы, которые помогают пользователям вводить правильные данные с первого раза.

## Современные подходы к валидации форм

### Клиентская валидация с использованием Constraint Validation API
```html
<form id="advanced-validation-form">
    <div class="form-group">
        <label for="user-email">Email:</label>
        <input type="email" 
               id="user-email" 
               name="email" 
               required
               aria-describedby="email-error">
        <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <div class="form-group">
        <label for="user-password">Пароль:</label>
        <input type="password" 
               id="user-password" 
               name="password" 
               required 
               minlength="8"
               aria-describedby="password-requirements">
        <ul id="password-requirements" class="requirements-list">
            <li id="req-length">Не менее 8 символов</li>
            <li id="req-upper">Содержит заглавную букву</li>
            <li id="req-lower">Содержит строчную букву</li>
            <li id="req-number">Содержит цифру</li>
        </ul>
        <div id="password-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <div class="form-group">
        <label for="confirm-password">Подтверждение пароля:</label>
        <input type="password" 
               id="confirm-password" 
               name="confirm_password" 
               required
               aria-describedby="confirm-error">
        <div id="confirm-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <button type="submit">Зарегистрироваться</button>
</form>

<script>
class FormValidator {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.setupEventListeners();
    }

    setupEventListeners() {
        // Валидация при потере фокуса
        this.form.addEventListener('blur', (e) => {
            if (e.target.type !== 'submit') {
                this.validateField(e.target);
            }
        }, true);

        // Валидация в реальном времени для пароля
        document.getElementById('user-password').addEventListener('input', () => {
            this.validatePasswordRequirements();
        });

        // Валидация при вводе подтверждения пароля
        document.getElementById('confirm-password').addEventListener('input', () => {
            this.validatePasswordMatch();
        });

        // Валидация всей формы при отправке
        this.form.addEventListener('submit', (e) => {
            e.preventDefault();
            if (this.validateForm()) {
                this.submitForm();
            }
        });
    }

    validateField(field) {
        const fieldId = field.id;
        const errorElement = document.getElementById(`${fieldId}-error`);
        
        if (field.validity.valid) {
            field.classList.remove('invalid');
            field.classList.add('valid');
            errorElement.textContent = '';
            return true;
        } else {
            field.classList.remove('valid');
            field.classList.add('invalid');
            
            // Показываем соответствующее сообщение об ошибке
            errorElement.textContent = this.getErrorMessage(field);
            return false;
        }
    }

    validatePasswordRequirements() {
        const password = document.getElementById('user-password').value;
        const requirements = {
            length: password.length >= 8,
            upper: /[A-Z]/.test(password),
            lower: /[a-z]/.test(password),
            number: /[0-9]/.test(password)
        };

        // Обновляем статус требований
        document.getElementById('req-length').className = requirements.length ? 'met' : '';
        document.getElementById('req-upper').className = requirements.upper ? 'met' : '';
        document.getElementById('req-lower').className = requirements.lower ? 'met' : '';
        document.getElementById('req-number').className = requirements.number ? 'met' : '';

        return Object.values(requirements).every(req => req);
    }

    validatePasswordMatch() {
        const password = document.getElementById('user-password').value;
        const confirmPassword = document.getElementById('confirm-password').value;
        const errorElement = document.getElementById('confirm-error');

        if (confirmPassword && password !== confirmPassword) {
            errorElement.textContent = 'Пароли не совпадают';
            document.getElementById('confirm-password').classList.add('invalid');
            return false;
        } else {
            errorElement.textContent = '';
            document.getElementById('confirm-password').classList.remove('invalid');
            return true;
        }
    }

    validateForm() {
        let isValid = true;
        const fields = this.form.querySelectorAll('input[required]');

        fields.forEach(field => {
            if (!this.validateField(field)) {
                isValid = false;
            }
        });

        // Дополнительная проверка сложных требований
        if (!this.validatePasswordRequirements()) {
            isValid = false;
        }

        if (!this.validatePasswordMatch()) {
            isValid = false;
        }

        return isValid;
    }

    getErrorMessage(field) {
        if (field.validity.valueMissing) {
            return 'Это поле обязательно для заполнения';
        } else if (field.validity.typeMismatch && field.type === 'email') {
            return 'Введите действительный email адрес';
        } else if (field.validity.tooShort) {
            return `Минимальная длина: ${field.minLength} символов`;
        } else if (field.validity.patternMismatch) {
            return field.title || 'Неверный формат данных';
        } else if (field.validity.rangeUnderflow) {
            return `Значение должно быть не менее ${field.min}`;
        } else if (field.validity.rangeOverflow) {
            return `Значение должно быть не более ${field.max}`;
        }
        return 'Неверный формат данных';
    }

    async submitForm() {
        // Показываем индикатор загрузки
        const submitBtn = this.form.querySelector('button[type="submit"]');
        const originalText = submitBtn.textContent;
        submitBtn.textContent = 'Отправка...';
        submitBtn.disabled = true;

        try {
            // Здесь будет реальная отправка данных
            await this.sendData();
            alert('Форма успешно отправлена!');
        } catch (error) {
            alert('Ошибка при отправке формы: ' + error.message);
        } finally {
            submitBtn.textContent = originalText;
            submitBtn.disabled = false;
        }
    }

    async sendData() {
        // Симуляция отправки данных
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                Math.random() > 0.2 ? resolve() : reject(new Error('Сервер недоступен'));
            }, 2000);
        });
    }
}

// Инициализация валидатора
document.addEventListener('DOMContentLoaded', () => {
    new FormValidator('advanced-validation-form');
});
</script>
```

### Валидация с кастомными правилами
```html
<form id="custom-validation-form">
    <div class="form-group">
        <label for="username">Имя пользователя:</label>
        <input type="text" 
               id="username" 
               name="username" 
               required
               aria-describedby="username-error">
        <div id="username-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <div class="form-group">
        <label for="phone">Телефон:</label>
        <input type="tel" 
               id="phone" 
               name="phone" 
               required
               aria-describedby="phone-error">
        <div id="phone-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <button type="submit">Отправить</button>
</form>

<script>
class CustomValidator {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.setupValidation();
    }

    setupValidation() {
        // Валидация уникальности имени пользователя
        document.getElementById('username').addEventListener('blur', async (e) => {
            await this.validateUsername(e.target);
        });

        // Валидация формата телефона
        document.getElementById('phone').addEventListener('input', (e) => {
            this.formatPhone(e.target);
        });

        document.getElementById('phone').addEventListener('blur', (e) => {
            this.validatePhone(e.target);
        });

        this.form.addEventListener('submit', (e) => {
            e.preventDefault();
            this.validateForm();
        });
    }

    async validateUsername(field) {
        const username = field.value.trim();
        const errorElement = document.getElementById('username-error');

        if (!username) {
            errorElement.textContent = 'Имя пользователя обязательно';
            field.classList.add('invalid');
            return false;
        }

        if (username.length < 3) {
            errorElement.textContent = 'Имя пользователя должно быть не менее 3 символов';
            field.classList.add('invalid');
            return false;
        }

        if (!/^[A-Za-z0-9_]+$/.test(username)) {
            errorElement.textContent = 'Имя пользователя может содержать только буквы, цифры и подчеркивание';
            field.classList.add('invalid');
            return false;
        }

        // Проверка уникальности (симуляция)
        field.classList.add('validating');
        errorElement.textContent = 'Проверка...';

        try {
            const isUnique = await this.checkUsernameUniqueness(username);
            if (!isUnique) {
                errorElement.textContent = 'Имя пользователя уже занято';
                field.classList.add('invalid');
                field.classList.remove('validating');
                return false;
            } else {
                errorElement.textContent = '';
                field.classList.add('valid');
                field.classList.remove('validating');
                return true;
            }
        } catch (error) {
            errorElement.textContent = 'Ошибка проверки имени пользователя';
            field.classList.add('invalid');
            field.classList.remove('validating');
            return false;
        }
    }

    async checkUsernameUniqueness(username) {
        // Симуляция проверки на сервере
        return new Promise(resolve => {
            setTimeout(() => {
                resolve(!['admin', 'user', 'test'].includes(username.toLowerCase()));
            }, 500);
        });
    }

    formatPhone(field) {
        let value = field.value.replace(/\D/g, ''); // Удаляем все нецифры
        
        if (value.length >= 11) {
            // Формат: +7 (XXX) XXX-XX-XX
            value = value.substring(0, 11);
            const formatted = `+7 (${value.substring(1, 4)}) ${value.substring(4, 7)}-${value.substring(7, 9)}-${value.substring(9, 11)}`;
            field.value = formatted;
        } else if (value.length > 1) {
            const prefix = value.substring(0, 1);
            const rest = value.substring(1);
            let formatted = `+${prefix}`;
            
            if (rest.length > 0) formatted += ` (${rest.substring(0, 3)}`;
            if (rest.length > 3) formatted += `) ${rest.substring(3, 6)}`;
            if (rest.length > 6) formatted += `-${rest.substring(6, 8)}`;
            if (rest.length > 8) formatted += `-${rest.substring(8, 10)}`;
            
            field.value = formatted;
        }
    }

    validatePhone(field) {
        const phone = field.value.replace(/\D/g, '');
        const errorElement = document.getElementById('phone-error');

        if (!phone) {
            errorElement.textContent = 'Телефон обязателен';
            field.classList.add('invalid');
            return false;
        }

        if (phone.length !== 11 || phone[0] !== '7' && phone[0] !== '8') {
            errorElement.textContent = 'Введите действительный российский номер телефона';
            field.classList.add('invalid');
            return false;
        }

        errorElement.textContent = '';
        field.classList.add('valid');
        return true;
    }

    validateForm() {
        let isValid = true;

        // Валидация всех полей
        isValid = this.validateUsername(document.getElementById('username')) && isValid;
        isValid = this.validatePhone(document.getElementById('phone')) && isValid;

        if (isValid) {
            alert('Форма успешно валидирована!');
        }
    }
}

// Инициализация кастомного валидатора
document.addEventListener('DOMContentLoaded', () => {
    new CustomValidator('custom-validation-form');
});
</script>
```

## Валидация с использованием Web API

### Валидация с использованием Fetch API
```html
<form id="api-validation-form">
    <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" 
               id="email" 
               name="email" 
               required
               aria-describedby="email-error">
        <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <div class="form-group">
        <label for="captcha">Капча:</label>
        <div class="captcha-container">
            <img id="captcha-image" src="/captcha" alt="Капча">
            <button type="button" onclick="refreshCaptcha()">🔄</button>
        </div>
        <input type="text" 
               id="captcha-input" 
               name="captcha" 
               required 
               maxlength="6"
               aria-describedby="captcha-error">
        <div id="captcha-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>

    <button type="submit">Отправить</button>
</form>

<script>
class APIValidator {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.setupValidation();
    }

    setupValidation() {
        document.getElementById('email').addEventListener('blur', async (e) => {
            await this.validateEmailWithAPI(e.target);
        });

        document.getElementById('captcha-input').addEventListener('input', (e) => {
            if (e.target.value.length === 6) {
                this.validateCaptcha(e.target);
            }
        });

        this.form.addEventListener('submit', async (e) => {
            e.preventDefault();
            await this.validateFormWithAPI();
        });
    }

    async validateEmailWithAPI(field) {
        const email = field.value.trim();
        const errorElement = document.getElementById('email-error');

        if (!email) {
            errorElement.textContent = 'Email обязателен';
            field.classList.add('invalid');
            return false;
        }

        if (!field.validity.valid) {
            errorElement.textContent = 'Введите действительный email';
            field.classList.add('invalid');
            return false;
        }

        field.classList.add('validating');
        errorElement.textContent = 'Проверка...';

        try {
            const response = await fetch('/api/validate-email', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ email: email })
            });

            const result = await response.json();

            if (result.valid) {
                errorElement.textContent = '';
                field.classList.add('valid');
            } else {
                errorElement.textContent = result.message || 'Email недействителен или уже используется';
                field.classList.add('invalid');
            }
        } catch (error) {
            errorElement.textContent = 'Ошибка проверки email';
            field.classList.add('invalid');
        } finally {
            field.classList.remove('validating');
        }
    }

    async validateCaptcha(field) {
        const captcha = field.value.trim();
        const errorElement = document.getElementById('captcha-error');

        if (!captcha) {
            errorElement.textContent = 'Введите капчу';
            field.classList.add('invalid');
            return false;
        }

        field.classList.add('validating');
        errorElement.textContent = 'Проверка...';

        try {
            const response = await fetch('/api/validate-captcha', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ captcha: captcha })
            });

            const result = await response.json();

            if (result.valid) {
                errorElement.textContent = '';
                field.classList.add('valid');
            } else {
                errorElement.textContent = 'Неправильная капча. Попробуйте снова.';
                field.classList.add('invalid');
                refreshCaptcha(); // Обновляем капчу при ошибке
            }
        } catch (error) {
            errorElement.textContent = 'Ошибка проверки капчи';
            field.classList.add('invalid');
        } finally {
            field.classList.remove('validating');
        }
    }

    async validateFormWithAPI() {
        const email = document.getElementById('email');
        const captcha = document.getElementById('captcha-input');

        // Валидация всех полей
        const emailValid = await this.validateEmailWithAPI(email);
        const captchaValid = await this.validateCaptcha(captcha);

        if (emailValid && captchaValid) {
            // Отправка формы
            const formData = new FormData(this.form);
            try {
                const response = await fetch('/api/submit-form', {
                    method: 'POST',
                    body: formData
                });

                if (response.ok) {
                    alert('Форма успешно отправлена!');
                } else {
                    alert('Ошибка при отправке формы');
                }
            } catch (error) {
                alert('Ошибка сети при отправке формы');
            }
        }
    }
}

function refreshCaptcha() {
    document.getElementById('captcha-image').src = '/captcha?' + new Date().getTime();
    document.getElementById('captcha-input').value = '';
    document.getElementById('captcha-input').classList.remove('valid', 'invalid');
    document.getElementById('captcha-error').textContent = '';
}

// Инициализация API валидатора
document.addEventListener('DOMContentLoaded', () => {
    new APIValidator('api-validation-form');
});
</script>
```

## Современные паттерны валидации

### Валидация с использованием Web Components
```html
<template id="validation-input-template">
    <style>
        .input-container {
            position: relative;
            margin-bottom: 20px;
        }

        input {
            width: 100%;
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }

        input.valid {
            border-color: #4caf50;
        }

        input.invalid {
            border-color: #f44336;
        }

        input.validating {
            border-color: #ff9800;
        }

        .error-message {
            color: #f44336;
            font-size: 0.875em;
            margin-top: 5px;
        }

        .status-icon {
            position: absolute;
            right: 10px;
            top: 50%;
            transform: translateY(-50%);
        }
    </style>
    <div class="input-container">
        <input type="text" id="input-field">
        <span class="status-icon" id="status-icon">🔍</span>
        <div class="error-message" id="error-message"></div>
    </div>
</template>

<script>
class ValidationInput extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        const template = document.getElementById('validation-input-template');
        this.shadowRoot.appendChild(template.content.cloneNode(true));
        
        this.inputField = this.shadowRoot.getElementById('input-field');
        this.statusIcon = this.shadowRoot.getElementById('status-icon');
        this.errorMessage = this.shadowRoot.getElementById('error-message');
        
        this.setupValidation();
    }

    static get observedAttributes() {
        return ['type', 'required', 'pattern', 'minlength', 'maxlength', 'validation-url'];
    }

    attributeChangedCallback(name, oldValue, newValue) {
        if (oldValue !== newValue) {
            switch(name) {
                case 'type':
                    this.inputField.type = newValue;
                    break;
                case 'required':
                    this.inputField.required = newValue !== null;
                    break;
                case 'pattern':
                    this.inputField.pattern = newValue;
                    break;
                case 'minlength':
                    this.inputField.minLength = newValue;
                    break;
                case 'maxlength':
                    this.inputField.maxLength = newValue;
                    break;
                case 'validation-url':
                    this.validationUrl = newValue;
                    break;
            }
        }
    }

    setupValidation() {
        this.inputField.addEventListener('blur', async () => {
            await this.validate();
        });

        this.inputField.addEventListener('input', () => {
            this.clearValidation();
        });
    }

    async validate() {
        const value = this.inputField.value.trim();
        
        // Валидация на стороне клиента
        if (this.inputField.required && !value) {
            this.showError('Это поле обязательно для заполнения');
            return false;
        }

        if (this.inputField.pattern && !new RegExp(this.inputField.pattern).test(value)) {
            this.showError('Неверный формат данных');
            return false;
        }

        if (this.inputField.minLength && value.length < this.inputField.minLength) {
            this.showError(`Минимум ${this.inputField.minLength} символов`);
            return false;
        }

        if (this.inputField.maxLength && value.length > this.inputField.maxLength) {
            this.showError(`Максимум ${this.inputField.maxLength} символов`);
            return false;
        }

        // Валидация на стороне сервера (если указан URL)
        if (this.validationUrl) {
            return await this.validateWithServer(value);
        }

        this.showSuccess();
        return true;
    }

    async validateWithServer(value) {
        this.inputField.classList.add('validating');
        this.statusIcon.textContent = '⏳';

        try {
            const response = await fetch(this.validationUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ [this.inputField.name]: value })
            });

            const result = await response.json();

            if (result.valid) {
                this.showSuccess();
                return true;
            } else {
                this.showError(result.message || 'Значение недействительно');
                return false;
            }
        } catch (error) {
            this.showError('Ошибка проверки значения');
            return false;
        } finally {
            this.inputField.classList.remove('validating');
        }
    }

    showSuccess() {
        this.inputField.classList.remove('invalid');
        this.inputField.classList.add('valid');
        this.statusIcon.textContent = '✓';
        this.errorMessage.textContent = '';
    }

    showError(message) {
        this.inputField.classList.remove('valid');
        this.inputField.classList.add('invalid');
        this.statusIcon.textContent = '✗';
        this.errorMessage.textContent = message;
    }

    clearValidation() {
        this.inputField.classList.remove('valid', 'invalid', 'validating');
        this.statusIcon.textContent = '🔍';
        this.errorMessage.textContent = '';
    }

    getValue() {
        return this.inputField.value;
    }

    setValue(value) {
        this.inputField.value = value;
    }
}

customElements.define('validation-input', ValidationInput);
</script>

<!-- Использование кастомного элемента -->
<form id="web-component-form">
    <validation-input 
        name="email" 
        type="email" 
        required 
        validation-url="/api/validate-email">
    </validation-input>

    <validation-input 
        name="username" 
        required 
        pattern="[A-Za-z0-9_]{3,20}" 
        validation-url="/api/validate-username">
    </validation-input>

    <button type="submit">Отправить</button>
</form>

<script>
document.getElementById('web-component-form').addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const inputs = document.querySelectorAll('validation-input');
    let allValid = true;
    
    for (const input of inputs) {
        const isValid = await input.validate();
        allValid = allValid && isValid;
    }
    
    if (allValid) {
        alert('Форма успешно валидирована!');
    }
});
</script>
```

## Доступная валидация форм

### Валидация с улучшенной доступностью
```html
<form id="accessible-validation-form" novalidate>
    <div class="form-group">
        <label for="acc-email">Email:</label>
        <input type="email" 
               id="acc-email" 
               name="email" 
               required
               aria-describedby="email-error email-help"
               aria-invalid="false">
        <div id="email-help" class="help-text">Введите ваш email для связи</div>
        <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <div class="form-group">
        <label for="acc-password">Пароль:</label>
        <input type="password" 
               id="acc-password" 
               name="password" 
               required 
               minlength="8"
               aria-describedby="password-requirements password-error"
               aria-invalid="false">
        <ul id="password-requirements" class="requirements-list">
            <li id="acc-req-length">Не менее 8 символов</li>
            <li id="acc-req-upper">Содержит заглавную букву</li>
            <li id="acc-req-lower">Содержит строчную букву</li>
            <li id="acc-req-number">Содержит цифру</li>
        </ul>
        <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <div class="form-group">
        <label for="acc-confirm">Подтверждение пароля:</label>
        <input type="password" 
               id="acc-confirm" 
               name="confirm_password" 
               required
               aria-describedby="confirm-error"
               aria-invalid="false">
        <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <div class="form-group">
        <label>
            <input type="checkbox" 
                   name="terms" 
                   required
                   aria-describedby="terms-error">
            Я согласен с условиями использования
        </label>
        <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <button type="submit">Зарегистрироваться</button>
</form>

<script>
class AccessibleValidator {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.setupAccessibleValidation();
    }

    setupAccessibleValidation() {
        // Валидация при потере фокуса
        this.form.addEventListener('blur', (e) => {
            if (e.target.type !== 'submit') {
                this.validateField(e.target);
            }
        }, true);

        // Валидация пароля в реальном времени
        document.getElementById('acc-password').addEventListener('input', () => {
            this.updatePasswordRequirements();
        });

        // Валидация при вводе подтверждения пароля
        document.getElementById('acc-confirm').addEventListener('input', () => {
            this.validatePasswordMatch();
        });

        // Валидация всей формы при отправке
        this.form.addEventListener('submit', (e) => {
            e.preventDefault();
            this.validateFormAccessible();
        });
    }

    validateField(field) {
        const fieldId = field.id;
        const errorElement = document.getElementById(`${fieldId.replace('-', '')}-error`);
        
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
            
            // Показываем соответствующее сообщение об ошибке
            errorElement.textContent = this.getAccessibleErrorMessage(field);
            
            // Фокус на поле с ошибкой
            field.focus();
            return false;
        }
    }

    updatePasswordRequirements() {
        const password = document.getElementById('acc-password').value;
        const requirements = {
            length: password.length >= 8,
            upper: /[A-Z]/.test(password),
            lower: /[a-z]/.test(password),
            number: /[0-9]/.test(password)
        };

        // Обновляем статус требований и делаем доступным для скринридеров
        document.getElementById('acc-req-length').className = requirements.length ? 'met' : '';
        document.getElementById('acc-req-upper').className = requirements.upper ? 'met' : '';
        document.getElementById('acc-req-lower').className = requirements.lower ? 'met' : '';
        document.getElementById('acc-req-number').className = requirements.number ? 'met' : '';
    }

    validatePasswordMatch() {
        const password = document.getElementById('acc-password').value;
        const confirmPassword = document.getElementById('acc-confirm').value;
        const errorElement = document.getElementById('confirm-error');

        if (confirmPassword && password !== confirmPassword) {
            errorElement.textContent = 'Пароли не совпадают';
            document.getElementById('acc-confirm').classList.add('invalid');
            document.getElementById('acc-confirm').setAttribute('aria-invalid', 'true');
            return false;
        } else {
            errorElement.textContent = '';
            document.getElementById('acc-confirm').classList.remove('invalid');
            document.getElementById('acc-confirm').setAttribute('aria-invalid', 'false');
            return true;
        }
    }

    validateFormAccessible() {
        let isValid = true;
        const fields = this.form.querySelectorAll('input[required], checkbox[required]');

        fields.forEach(field => {
            if (!this.validateField(field)) {
                isValid = false;
            }
        });

        // Проверка сложных требований
        if (!this.validatePasswordRequirements()) {
            isValid = false;
        }

        if (!this.validatePasswordMatch()) {
            isValid = false;
        }

        if (isValid) {
            alert('Форма успешно валидирована и готова к отправке!');
        } else {
            // Сообщение для пользователей скринридеров о количестве ошибок
            const errorCount = this.form.querySelectorAll('.error-message:not(:empty)').length;
            if (errorCount > 0) {
                const errorSummary = document.createElement('div');
                errorSummary.setAttribute('aria-live', 'assertive');
                errorSummary.setAttribute('role', 'alert');
                errorSummary.style.position = 'absolute';
                errorSummary.style.left = '-9999px';
                errorSummary.textContent = `Обнаружено ${errorCount} ошибок. Пожалуйста, исправьте их перед отправкой.`;
                document.body.appendChild(errorSummary);
                
                // Удаляем сообщение через 5 секунд
                setTimeout(() => {
                    document.body.removeChild(errorSummary);
                }, 5000);
            }
        }
    }

    validatePasswordRequirements() {
        const password = document.getElementById('acc-password').value;
        const requirements = {
            length: password.length >= 8,
            upper: /[A-Z]/.test(password),
            lower: /[a-z]/.test(password),
            number: /[0-9]/.test(password)
        };

        return Object.values(requirements).every(req => req);
    }

    getAccessibleErrorMessage(field) {
        if (field.validity.valueMissing) {
            return 'Это поле обязательно для заполнения';
        } else if (field.validity.typeMismatch && field.type === 'email') {
            return 'Введите действительный email адрес';
        } else if (field.validity.tooShort) {
            return `Минимальная длина: ${field.minLength} символов`;
        } else if (field.validity.patternMismatch) {
            return field.title || 'Неверный формат данных';
        } else if (field.validity.rangeUnderflow) {
            return `Значение должно быть не менее ${field.min}`;
        } else if (field.validity.rangeOverflow) {
            return `Значение должно быть не более ${field.max}`;
        }
        return 'Неверный формат данных';
    }
}

// Инициализация доступного валидатора
document.addEventListener('DOMContentLoaded', () => {
    new AccessibleValidator('accessible-validation-form');
});
</script>
```

## Лучшие практики валидации форм

### 1. Комбинированная валидация (клиент + сервер)
```html
<form id="combined-validation-form">
    <div class="form-group">
        <label for="combined-email">Email:</label>
        <input type="email" 
               id="combined-email" 
               name="email" 
               required
               aria-describedby="combined-email-error">
        <div id="combined-email-error" class="error-message" role="alert" aria-live="assertive"></div>
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
        // Клиентская валидация при потере фокуса
        document.getElementById('combined-email').addEventListener('blur', (e) => {
            this.validateEmailClient(e.target);
        });

        // Валидация при отправке формы
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

        if (!field.validity.valid) {
            this.showError(field, errorElement, 'Введите действительный email');
            return false;
        }

        // Очистка ошибки, если поле валидно
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

    async validateFormCombined() {
        // Сначала клиентская валидация
        const emailField = document.getElementById('combined-email');
        const isClientValid = this.validateEmailClient(emailField);

        if (!isClientValid) {
            return; // Не продолжаем, если клиентская валидация не пройдена
        }

        // Затем серверная валидация
        try {
            const response = await fetch('/api/validate-form', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    email: emailField.value
                })
            });

            const result = await response.json();

            if (result.valid) {
                alert('Форма успешно валидирована и отправлена!');
                // Здесь можно выполнить отправку формы
            } else {
                // Показать ошибки серверной валидации
                if (result.errors && result.errors.email) {
                    this.showError(emailField, 
                                 document.getElementById('combined-email-error'), 
                                 result.errors.email);
                }
            }
        } catch (error) {
            alert('Ошибка при проверке данных на сервере');
        }
    }
}

// Инициализация комбинированного валидатора
document.addEventListener('DOMContentLoaded', () => {
    new CombinedValidator('combined-validation-form');
});
</script>
```

### 2. Валидация с индикаторами прогресса
```html
<form id="progress-validation-form">
    <div class="progress-bar">
        <div class="progress" id="form-progress"></div>
        <div class="progress-label" id="progress-label">0% заполнено</div>
    </div>

    <div class="form-step active" id="step1">
        <h3>Шаг 1: Контактная информация</h3>
        
        <div class="form-group">
            <label for="prog-email">Email:</label>
            <input type="email" id="prog-email" name="email" required>
            <div class="error-message" id="prog-email-error" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="prog-phone">Телефон:</label>
            <input type="tel" id="prog-phone" name="phone" required>
            <div class="error-message" id="prog-phone-error" role="alert" aria-live="assertive"></div>
        </div>
        
        <button type="button" onclick="nextStep(1)">Далее</button>
    </div>
    
    <div class="form-step" id="step2">
        <h3>Шаг 2: Дополнительная информация</h3>
        
        <div class="form-group">
            <label for="prog-name">Имя:</label>
            <input type="text" id="prog-name" name="name" required>
            <div class="error-message" id="prog-name-error" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="prog-age">Возраст:</label>
            <input type="number" id="prog-age" name="age" min="18" max="100">
            <div class="error-message" id="prog-age-error" role="alert" aria-live="assertive"></div>
        </div>
        
        <button type="button" onclick="prevStep(2)">Назад</button>
        <button type="button" onclick="nextStep(2)">Далее</button>
    </div>
    
    <div class="form-step" id="step3">
        <h3>Шаг 3: Подтверждение</h3>
        <div id="summary"></div>
        
        <div class="form-group">
            <label>
                <input type="checkbox" name="terms" required>
                Я согласен с условиями
            </label>
            <div class="error-message" id="prog-terms-error" role="alert" aria-live="assertive"></div>
        </div>
        
        <button type="button" onclick="prevStep(3)">Назад</button>
        <button type="submit">Отправить</button>
    </div>
</form>

<script>
let currentStep = 1;
const totalSteps = 3;
const validator = {
    validateStep: function(step) {
        const stepElement = document.getElementById(`step${step}`);
        const requiredFields = stepElement.querySelectorAll('[required]');
        let isValid = true;

        requiredFields.forEach(field => {
            const errorElement = document.getElementById(`${field.id}-error`);
            
            if (!field.value.trim()) {
                errorElement.textContent = 'Это поле обязательно';
                field.classList.add('invalid');
                isValid = false;
            } else {
                errorElement.textContent = '';
                field.classList.remove('invalid');
            }
        });

        return isValid;
    },

    updateProgress: function() {
        const validSteps = Array.from({length: totalSteps}, (_, i) => i + 1)
            .filter(step => this.isStepValid(step));
        
        const progress = (validSteps.length / totalSteps) * 100;
        document.getElementById('form-progress').style.width = `${progress}%`;
        document.getElementById('progress-label').textContent = `${Math.round(progress)}% заполнено`;
    },

    isStepValid: function(step) {
        const stepElement = document.getElementById(`step${step}`);
        const requiredFields = stepElement.querySelectorAll('[required]');
        
        return Array.from(requiredFields).every(field => field.value.trim() !== '');
    }
};

function nextStep(current) {
    if (validator.validateStep(current)) {
        document.getElementById(`step${current}`).classList.remove('active');
        currentStep = current + 1;
        document.getElementById(`step${currentStep}`).classList.add('active');
        validator.updateProgress();
        
        if (currentStep === 3) {
            updateSummary();
        }
    }
}

function prevStep(current) {
    document.getElementById(`step${current}`).classList.remove('active');
    currentStep = current - 1;
    document.getElementById(`step${currentStep}`).classList.add('active');
    validator.updateProgress();
}

function updateSummary() {
    const formData = new FormData(document.getElementById('progress-validation-form'));
    const summary = document.getElementById('summary');
    
    summary.innerHTML = `
        <h4>Проверьте вашу информацию:</h4>
        <p><strong>Email:</strong> ${formData.get('email')}</p>
        <p><strong>Телефон:</strong> ${formData.get('phone')}</p>
        <p><strong>Имя:</strong> ${formData.get('name')}</p>
        <p><strong>Возраст:</strong> ${formData.get('age') || 'Не указан'}</p>
    `;
}

document.getElementById('progress-validation-form').addEventListener('submit', function(e) {
    e.preventDefault();
    validator.validateStep(3); // Проверяем последний шаг
    
    if (validator.isStepValid(3)) {
        alert('Форма успешно отправлена!');
    }
});

// Инициализация прогресса
validator.updateProgress();
</script>
```

## Пример комплексной формы с современной валидацией

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Комплексная валидация формы</title>
    <meta name="description" content="Пример формы с современной валидацией">
    
    <style>
        .form-container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            font-family: Arial, sans-serif;
        }
        
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
            transition: border-color 0.3s;
        }
        
        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
        }
        
        input.valid {
            border-color: #4caf50;
        }
        
        input.invalid {
            border-color: #f44336;
        }
        
        input.validating {
            border-color: #ff9800;
        }
        
        .error-message {
            color: #f44336;
            font-size: 0.875em;
            margin-top: 5px;
        }
        
        .help-text {
            color: #666;
            font-size: 0.875em;
            margin-top: 5px;
        }
        
        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 5px 0;
        }
        
        .requirements-list li {
            font-size: 0.875em;
            color: #666;
            margin-bottom: 2px;
        }
        
        .requirements-list li.met {
            color: #4caf50;
        }
        
        .progress-bar {
            height: 20px;
            background-color: #f0f0f0;
            border-radius: 10px;
            margin-bottom: 20px;
            overflow: hidden;
        }
        
        .progress {
            height: 100%;
            background-color: #007acc;
            transition: width 0.3s;
        }
        
        .progress-label {
            text-align: center;
            font-size: 0.875em;
            color: #666;
            margin-bottom: 15px;
        }
        
        .form-step {
            display: none;
        }
        
        .form-step.active {
            display: block;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin-right: 10px;
        }
        
        button:hover {
            background-color: #005a9e;
        }
        
        button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        
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
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Регистрация нового пользователя</h1>
        
        <div class="progress-bar">
            <div class="progress" id="form-progress"></div>
            <div class="progress-label" id="progress-label">0% заполнено</div>
        </div>
        
        <form id="comprehensive-validation-form" novalidate>
            <!-- Шаг 1: Контактная информация -->
            <div class="form-step active" id="step1">
                <h2>Шаг 1: Контактная информация</h2>
                
                <div class="form-group">
                    <label for="contact-email">Email:</label>
                    <input type="email" 
                           id="contact-email" 
                           name="email" 
                           required
                           aria-describedby="email-error email-help"
                           aria-invalid="false">
                    <div id="email-help" class="help-text">Будет использоваться для входа в аккаунт</div>
                    <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="contact-phone">Телефон:</label>
                    <input type="tel" 
                           id="contact-phone" 
                           name="phone" 
                           required
                           aria-describedby="phone-error phone-help"
                           aria-invalid="false">
                    <div id="phone-help" class="help-text">Формат: +7 (XXX) XXX-XX-XX</div>
                    <div id="phone-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <button type="button" onclick="nextStep(1)">Далее</button>
            </div>
            
            <!-- Шаг 2: Аккаунт -->
            <div class="form-step" id="step2">
                <h2>Шаг 2: Аккаунт</h2>
                
                <div class="form-group">
                    <label for="account-username">Имя пользователя:</label>
                    <input type="text" 
                           id="account-username" 
                           name="username" 
                           required
                           minlength="3"
                           maxlength="20"
                           pattern="[A-Za-z0-9_]+"
                           aria-describedby="username-error username-help"
                           aria-invalid="false">
                    <div id="username-help" class="help-text">Только буквы, цифры и подчеркивание, 3-20 символов</div>
                    <div id="username-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="account-password">Пароль:</label>
                    <input type="password" 
                           id="account-password" 
                           name="password" 
                           required 
                           minlength="8"
                           aria-describedby="password-requirements password-error"
                           aria-invalid="false">
                    <ul id="password-requirements" class="requirements-list">
                        <li id="req-length">Не менее 8 символов</li>
                        <li id="req-upper">Содержит заглавную букву</li>
                        <li id="req-lower">Содержит строчную букву</li>
                        <li id="req-number">Содержит цифру</li>
                    </ul>
                    <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="account-confirm">Подтверждение пароля:</label>
                    <input type="password" 
                           id="account-confirm" 
                           name="confirm_password" 
                           required
                           aria-describedby="confirm-error"
                           aria-invalid="false">
                    <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <button type="button" onclick="prevStep(2)">Назад</button>
                <button type="button" onclick="nextStep(2)">Далее</button>
            </div>
            
            <!-- Шаг 3: Подтверждение -->
            <div class="form-step" id="step3">
                <h2>Шаг 3: Подтверждение</h2>
                
                <div id="summary"></div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox" 
                               name="newsletter" 
                               value="yes">
                        Подписаться на рассылку
                    </label>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox" 
                               name="terms" 
                               required
                               aria-describedby="terms-error">
                        Я согласен с условиями использования
                    </label>
                    <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <button type="button" onclick="prevStep(3)">Назад</button>
                <button type="submit">Зарегистрироваться</button>
            </div>
        </form>
    </div>

    <script>
        let currentStep = 1;
        const totalSteps = 3;
        
        // Класс для комплексной валидации
        class ComprehensiveValidator {
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
                document.getElementById('account-password').addEventListener('input', () => {
                    this.updatePasswordRequirements();
                });

                // Валидация подтверждения пароля в реальном времени
                document.getElementById('account-confirm').addEventListener('input', () => {
                    this.validatePasswordMatch();
                });

                // Валидация при вводе телефона
                document.getElementById('contact-phone').addEventListener('input', (e) => {
                    this.formatPhone(e.target);
                });

                // Отправка формы
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateForm();
                });
            }

            validateField(field) {
                const fieldId = field.id;
                const errorElement = document.getElementById(`${fieldId.replace(/-/g, '')}-error`);
                
                if (field.validity.valid) {
                    field.classList.remove('invalid');
                    field.classList.add('valid');
                    field.setAttribute('aria-invalid', 'false');
                    errorElement.textContent = '';
                    this.updateProgress();
                    return true;
                } else {
                    field.classList.remove('valid');
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    
                    errorElement.textContent = this.getErrorMessage(field);
                    this.updateProgress();
                    return false;
                }
            }

            validatePasswordRequirements() {
                const password = document.getElementById('account-password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password)
                };

                // Обновляем требования
                document.getElementById('req-length').className = requirements.length ? 'met' : '';
                document.getElementById('req-upper').className = requirements.upper ? 'met' : '';
                document.getElementById('req-lower').className = requirements.lower ? 'met' : '';
                document.getElementById('req-number').className = requirements.number ? 'met' : '';

                return Object.values(requirements).every(req => req);
            }

            updatePasswordRequirements() {
                this.validatePasswordRequirements();
                this.updateProgress();
            }

            validatePasswordMatch() {
                const password = document.getElementById('account-password').value;
                const confirmPassword = document.getElementById('account-confirm').value;
                const errorElement = document.getElementById('confirm-error');

                if (confirmPassword && password !== confirmPassword) {
                    errorElement.textContent = 'Пароли не совпадают';
                    document.getElementById('account-confirm').classList.add('invalid');
                    document.getElementById('account-confirm').setAttribute('aria-invalid', 'true');
                    this.updateProgress();
                    return false;
                } else {
                    errorElement.textContent = '';
                    document.getElementById('account-confirm').classList.remove('invalid');
                    document.getElementById('account-confirm').setAttribute('aria-invalid', 'false');
                    this.updateProgress();
                    return true;
                }
            }

            formatPhone(field) {
                let value = field.value.replace(/\D/g, '');
                
                if (value.length >= 11) {
                    value = value.substring(0, 11);
                    const formatted = `+7 (${value.substring(1, 4)}) ${value.substring(4, 7)}-${value.substring(7, 9)}-${value.substring(9, 11)}`;
                    field.value = formatted;
                } else if (value.length > 1) {
                    const prefix = value.substring(0, 1);
                    const rest = value.substring(1);
                    let formatted = `+${prefix}`;
                    
                    if (rest.length > 0) formatted += ` (${rest.substring(0, 3)}`;
                    if (rest.length > 3) formatted += `) ${rest.substring(3, 6)}`;
                    if (rest.length > 6) formatted += `-${rest.substring(6, 8)}`;
                    if (rest.length > 8) formatted += `-${rest.substring(8, 10)}`;
                    
                    field.value = formatted;
                }
            }

            validateForm() {
                let isValid = true;
                
                // Проверяем все обязательные поля
                const requiredFields = this.form.querySelectorAll('input[required]');
                requiredFields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                });

                // Проверяем сложные требования
                if (!this.validatePasswordRequirements()) {
                    isValid = false;
                }

                if (!this.validatePasswordMatch()) {
                    isValid = false;
                }

                if (isValid) {
                    alert('Форма успешно валидирована и готова к отправке!');
                    // Здесь могла бы быть отправка формы
                } else {
                    // Сообщение для скринридеров
                    const errorSummary = document.createElement('div');
                    errorSummary.setAttribute('aria-live', 'assertive');
                    errorSummary.setAttribute('role', 'alert');
                    errorSummary.style.position = 'absolute';
                    errorSummary.style.left = '-9999px';
                    errorSummary.textContent = 'Обнаружены ошибки валидации. Пожалуйста, исправьте их перед отправкой.';
                    document.body.appendChild(errorSummary);
                    
                    setTimeout(() => {
                        if (document.body.contains(errorSummary)) {
                            document.body.removeChild(errorSummary);
                        }
                    }, 5000);
                }
            }

            updateProgress() {
                const validFields = this.form.querySelectorAll('input.valid').length;
                const totalRequired = this.form.querySelectorAll('input[required]').length;
                
                const progress = totalRequired > 0 ? (validFields / totalRequired) * 100 : 0;
                document.getElementById('form-progress').style.width = `${progress}%`;
                document.getElementById('progress-label').textContent = `${Math.round(progress)}% заполнено`;
            }

            getErrorMessage(field) {
                if (field.validity.valueMissing) {
                    return 'Это поле обязательно для заполнения';
                } else if (field.validity.typeMismatch && field.type === 'email') {
                    return 'Введите действительный email адрес';
                } else if (field.validity.tooShort) {
                    return `Минимум ${field.minLength} символов`;
                } else if (field.validity.patternMismatch) {
                    return 'Неверный формат данных';
                } else if (field.validity.tooLong) {
                    return `Максимум ${field.maxLength} символов`;
                }
                return 'Неверный формат данных';
            }
        }

        // Навигация по шагам
        function nextStep(current) {
            const stepElement = document.getElementById(`step${current}`);
            const requiredFields = stepElement.querySelectorAll('[required]');
            let stepValid = true;

            requiredFields.forEach(field => {
                if (!field.value.trim()) {
                    stepValid = false;
                    document.getElementById(`${field.id.replace(/-/g, '')}-error`).textContent = 'Это поле обязательно';
                    field.classList.add('invalid');
                }
            });

            if (stepValid) {
                document.getElementById(`step${current}`).classList.remove('active');
                currentStep = current + 1;
                document.getElementById(`step${currentStep}`).classList.add('active');
                
                if (currentStep === 3) {
                    updateSummary();
                }
            }
        }

        function prevStep(current) {
            document.getElementById(`step${current}`).classList.remove('active');
            currentStep = current - 1;
            document.getElementById(`step${currentStep}`).classList.add('active');
        }

        function updateSummary() {
            const formData = new FormData(document.getElementById('comprehensive-validation-form'));
            const summary = document.getElementById('summary');
            
            summary.innerHTML = `
                <h3>Проверьте вашу информацию:</h3>
                <p><strong>Email:</strong> ${formData.get('email')}</p>
                <p><strong>Телефон:</strong> ${formData.get('phone')}</p>
                <p><strong>Имя пользователя:</strong> ${formData.get('username')}</p>
            `;
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new ComprehensiveValidator('comprehensive-validation-form');
        });
    </script>
</body>
</html>
```

## Современные возможности валидации форм

### Валидация с использованием Web Workers
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация с Web Workers</title>
</head>
<body>
    <form id="worker-validation-form">
        <div class="form-group">
            <label for="large-data">Большие данные для проверки:</label>
            <textarea id="large-data" name="data" rows="10" cols="50"></textarea>
            <button type="button" onclick="validateWithWorker()">Проверить данные</button>
        </div>
        <div id="validation-result"></div>
    </form>

    <script>
        // Создаем Web Worker для сложной валидации
        const workerScript = `
            self.onmessage = function(e) {
                const data = e.data;
                // Симуляция сложной валидации
                setTimeout(() => {
                    const result = complexValidation(data);
                    self.postMessage(result);
                }, 1000); // Имитация длительной операции
            };

            function complexValidation(data) {
                // Сложная логика валидации
                const lines = data.split('\\n');
                const errors = [];
                
                lines.forEach((line, index) => {
                    if (line.length > 100) {
                        errors.push({
                            line: index + 1,
                            message: 'Слишком длинная строка (максимум 100 символов)'
                        });
                    }
                    
                    if (!line.trim()) {
                        errors.push({
                            line: index + 1,
                            message: 'Пустая строка недопустима'
                        });
                    }
                });
                
                return {
                    valid: errors.length === 0,
                    errors: errors,
                    totalLines: lines.length
                };
            }
        `;

        // Создаем Blob и Worker
        const blob = new Blob([workerScript], { type: 'application/javascript' });
        const worker = new Worker(URL.createObjectURL(blob));

        worker.onmessage = function(e) {
            const result = e.data;
            const resultDiv = document.getElementById('validation-result');
            
            if (result.valid) {
                resultDiv.innerHTML = `
                    <div style="color: green; padding: 10px; background: #e8f5e8; border-radius: 4px;">
                        Данные валидны! Всего строк: ${result.totalLines}
                    </div>
                `;
            } else {
                resultDiv.innerHTML = `
                    <div style="color: red; padding: 10px; background: #ffebee; border-radius: 4px;">
                        <h4>Найдены ошибки (${result.errors.length}):</h4>
                        <ul>
                            ${result.errors.map(error => 
                                '<li>Строка ' + error.line + ': ' + error.message + '</li>'
                            ).join('')}
                        </ul>
                    </div>
                `;
            }
        };

        function validateWithWorker() {
            const data = document.getElementById('large-data').value;
            worker.postMessage(data);
        }
    </script>
</body>
</html>
```

### Валидация с использованием WebAssembly
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация с WebAssembly</title>
</head>
<body>
    <h1>Валидация с WebAssembly</h1>
    
    <form id="wasm-validation-form">
        <div class="form-group">
            <label for="credit-card">Номер кредитной карты:</label>
            <input type="text" id="credit-card" name="credit_card" maxlength="19" placeholder="0000-0000-0000-0000">
            <div id="wasm-result"></div>
        </div>
    </form>

    <script>
        // Простой WebAssembly модуль для проверки номера кредитной карты (алгоритм Луна)
        // В реальном приложении это будет скомпилированный WASM файл
        async function validateCreditCardWASM(cardNumber) {
            // В реальном приложении: 
            // const wasmModule = await WebAssembly.instantiateStreaming(fetch('validation.wasm'));
            // return wasmModule.instance.exports.validateCard(cardNumber);
            
            // Симуляция WASM валидации
            return new Promise(resolve => {
                setTimeout(() => {
                    // Простая проверка алгоритма Луна
                    const cleaned = cardNumber.replace(/[\s-]/g, '');
                    let sum = 0;
                    let isEven = false;
                    
                    for (let i = cleaned.length - 1; i >= 0; i--) {
                        let digit = parseInt(cleaned[i]);
                        
                        if (isEven) {
                            digit *= 2;
                            if (digit > 9) digit -= 9;
                        }
                        
                        sum += digit;
                        isEven = !isEven;
                    }
                    
                    const isValid = (sum % 10) === 0 && cleaned.length === 16;
                    resolve(isValid);
                }, 50);
            });
        }

        document.getElementById('credit-card').addEventListener('input', async function(e) {
            const cardNumber = e.target.value.replace(/[^\d\s-]/g, '');
            
            // Форматирование ввода
            const cleaned = cardNumber.replace(/[\s-]/g, '');
            let formatted = '';
            for (let i = 0; i < cleaned.length; i++) {
                if (i > 0 && i % 4 === 0) formatted += '-';
                formatted += cleaned[i];
            }
            e.target.value = formatted;
            
            // Валидация
            if (cleaned.length === 16) {
                const isValid = await validateCreditCardWASM(cleaned);
                const resultDiv = document.getElementById('wasm-result');
                
                if (isValid) {
                    resultDiv.innerHTML = '<span style="color: green;">✓ Номер карты действителен</span>';
                    e.target.style.borderColor = '#4caf50';
                } else {
                    resultDiv.innerHTML = '<span style="color: red;">✗ Неверный номер карты</span>';
                    e.target.style.borderColor = '#f44336';
                }
            }
        });
    </script>
</body>
</html>
```

### Валидация с использованием сервис-воркеров
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация с сервис-воркерами</title>
</head>
<body>
    <h1>Валидация с сервис-воркерами</h1>
    
    <form id="sw-validation-form">
        <div class="form-group">
            <label for="email-offline">Email (проверка в оффлайн):</label>
            <input type="email" id="email-offline" name="email" required>
            <div id="sw-result"></div>
        </div>
    </form>

    <script>
        // Регистрация сервис-воркера для оффлайн валидации
        if ('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/sw-validation.js').then(registration => {
                console.log('Сервис-воркер зарегистрирован:', registration.scope);
            }).catch(error => {
                console.log('Ошибка регистрации сервис-воркера:', error);
            });
        }

        // Оффлайн валидация email
        document.getElementById('email-offline').addEventListener('blur', async function(e) {
            const email = e.target.value;
            const resultDiv = document.getElementById('sw-result');
            
            if (!email) return;
            
            // Проверка формата email
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!emailRegex.test(email)) {
                resultDiv.innerHTML = '<span style="color: red;">✗ Неверный формат email</span>';
                e.target.style.borderColor = '#f44336';
                return;
            }
            
            // Проверка с использованием сервис-воркера
            try {
                const response = await fetch('/api/validate-email', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({ email: email })
                });
                
                const data = await response.json();
                
                if (data.valid) {
                    resultDiv.innerHTML = '<span style="color: green;">✓ Email доступен</span>';
                    e.target.style.borderColor = '#4caf50';
                } else {
                    resultDiv.innerHTML = `<span style="color: red;">✗ ${data.message}</span>`;
                    e.target.style.borderColor = '#f44336';
                }
            } catch (error) {
                // В оффлайн режиме показываем предупреждение
                resultDiv.innerHTML = '<span style="color: orange;">⚠ Проверка недоступна (оффлайн)</span>';
                e.target.style.borderColor = '#ff9800';
            }
        });
    </script>
    
    <!-- Содержимое sw-validation.js -->
    <script type="text/plain" id="sw-validation-code">
        // sw-validation.js
        const CACHE_NAME = 'validation-cache-v1';
        const urlsToCache = [
            '/api/validate-email'
        ];

        self.addEventListener('install', event => {
            event.waitUntil(
                caches.open(CACHE_NAME)
                    .then(cache => {
                        return cache.addAll(urlsToCache);
                    })
            );
        });

        self.addEventListener('fetch', event => {
            if (event.request.url.includes('/api/validate-email')) {
                event.respondWith(
                    caches.match(event.request)
                        .then(response => {
                            if (response) {
                                return response;
                            }
                            return fetch(event.request);
                        })
                        .catch(() => {
                            // Возвращаем оффлайн ответ
                            return new Response(JSON.stringify({
                                valid: true,
                                message: 'Проверка недоступна, данные сохранены для последующей проверки'
                            }), {
                                headers: { 'Content-Type': 'application/json' }
                            });
                        })
                );
            }
        });
    </script>
</body>
</html>
```

## Практические примеры и шаблоны

### Валидация с кешированием результатов
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация с кешированием</title>
    <style>
        .form-container {
            max-width: 500px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .cache-info {
            font-size: 0.8em;
            color: #666;
            margin-top: 5px;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Валидация с кешированием</h1>
        
        <form id="cached-validation-form">
            <div class="form-group">
                <label for="cached-email">Email:</label>
                <input type="email" id="cached-email" name="email" required>
                <div class="cache-info" id="email-cache-info"></div>
                <div class="error-message" id="email-error"></div>
            </div>
            
            <div class="form-group">
                <label for="cached-username">Имя пользователя:</label>
                <input type="text" id="cached-username" name="username" required>
                <div class="cache-info" id="username-cache-info"></div>
                <div class="error-message" id="username-error"></div>
            </div>
            
            <button type="submit">Отправить</button>
        </form>
    </div>

    <script>
        class CachedValidator {
            constructor() {
                this.cache = new Map();
                this.cacheTimeout = 5 * 60 * 1000; // 5 минут
                this.setupValidation();
            }
            
            setupValidation() {
                document.getElementById('cached-email').addEventListener('blur', async (e) => {
                    await this.validateWithCache(e.target, 'email');
                });
                
                document.getElementById('cached-username').addEventListener('blur', async (e) => {
                    await this.validateWithCache(e.target, 'username');
                });
            }
            
            async validateWithCache(field, type) {
                const value = field.value.trim();
                const fieldId = field.id;
                const errorElement = document.getElementById(`${fieldId}-error`);
                const cacheInfoElement = document.getElementById(`${fieldId}-cache-info`);
                
                if (!value) {
                    errorElement.textContent = '';
                    cacheInfoElement.textContent = '';
                    return true;
                }
                
                // Проверяем кеш
                const cacheKey = `${type}:${value}`;
                const cachedResult = this.cache.get(cacheKey);
                
                if (cachedResult && (Date.now() - cachedResult.timestamp) < this.cacheTimeout) {
                    // Используем кешированный результат
                    if (cachedResult.valid) {
                        field.classList.add('valid');
                        field.classList.remove('invalid');
                        errorElement.textContent = '';
                        cacheInfoElement.textContent = 'Проверено (кешировано)';
                    } else {
                        field.classList.add('invalid');
                        field.classList.remove('valid');
                        errorElement.textContent = cachedResult.message;
                        cacheInfoElement.textContent = 'Проверено (кешировано)';
                    }
                    return cachedResult.valid;
                }
                
                // Выполняем валидацию
                field.classList.add('validating');
                errorElement.textContent = 'Проверка...';
                cacheInfoElement.textContent = '';
                
                try {
                    const result = await this.validateOnServer(value, type);
                    
                    // Кешируем результат
                    this.cache.set(cacheKey, {
                        valid: result.valid,
                        message: result.message,
                        timestamp: Date.now()
                    });
                    
                    if (result.valid) {
                        field.classList.add('valid');
                        field.classList.remove('invalid', 'validating');
                        errorElement.textContent = '';
                        cacheInfoElement.textContent = 'Проверено и закешировано';
                    } else {
                        field.classList.add('invalid');
                        field.classList.remove('valid', 'validating');
                        errorElement.textContent = result.message;
                        cacheInfoElement.textContent = 'Проверено и закешировано';
                    }
                    
                    return result.valid;
                } catch (error) {
                    field.classList.add('invalid');
                    field.classList.remove('valid', 'validating');
                    errorElement.textContent = 'Ошибка проверки';
                    cacheInfoElement.textContent = '';
                    return false;
                }
            }
            
            async validateOnServer(value, type) {
                // Симуляция серверной валидации
                return new Promise((resolve) => {
                    setTimeout(() => {
                        if (type === 'email') {
                            if (value === 'existing@example.com') {
                                resolve({ valid: false, message: 'Email уже используется' });
                            } else {
                                resolve({ valid: true, message: '' });
                            }
                        } else if (type === 'username') {
                            if (['admin', 'user', 'test'].includes(value)) {
                                resolve({ valid: false, message: 'Имя пользователя занято' });
                            } else {
                                resolve({ valid: true, message: '' });
                            }
                        }
                    }, 800);
                });
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new CachedValidator();
        });
    </script>
</body>
</html>
```

### Валидация с машинным обучением (симуляция)
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидация с ML (симуляция)</title>
    <style>
        .ml-validation {
            position: relative;
        }
        
        .ml-confidence {
            font-size: 0.8em;
            color: #666;
            margin-top: 2px;
        }
        
        .ml-indicator {
            position: absolute;
            right: 10px;
            top: 50%;
            transform: translateY(-50%);
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Валидация с ML (симуляция)</h1>
        
        <form id="ml-validation-form">
            <div class="form-group ml-validation">
                <label for="ml-email">Email:</label>
                <input type="email" id="ml-email" name="email" required>
                <div class="ml-confidence" id="email-confidence"></div>
                <div class="error-message" id="ml-email-error"></div>
                <div class="ml-indicator" id="ml-email-indicator">🤖</div>
            </div>
            
            <div class="form-group ml-validation">
                <label for="ml-password">Пароль:</label>
                <input type="password" id="ml-password" name="password" required>
                <div class="ml-confidence" id="password-confidence"></div>
                <div class="error-message" id="ml-password-error"></div>
                <div class="ml-indicator" id="ml-password-indicator">🤖</div>
            </div>
            
            <button type="submit">Отправить</button>
        </form>
    </div>

    <script>
        class MLValidator {
            constructor() {
                this.setupValidation();
            }
            
            setupValidation() {
                document.getElementById('ml-email').addEventListener('input', (e) => {
                    this.mlValidateEmail(e.target);
                });
                
                document.getElementById('ml-password').addEventListener('input', (e) => {
                    this.mlValidatePassword(e.target);
                });
            }
            
            async mlValidateEmail(field) {
                const email = field.value;
                const errorElement = document.getElementById('ml-email-error');
                const confidenceElement = document.getElementById('email-confidence');
                const indicatorElement = document.getElementById('ml-email-indicator');
                
                if (!email) {
                    errorElement.textContent = '';
                    confidenceElement.textContent = '';
                    indicatorElement.textContent = '🤖';
                    return true;
                }
                
                // Симуляция ML-валидации
                indicatorElement.textContent = '⏳';
                
                const result = await this.simulateMLValidation('email', email);
                
                indicatorElement.textContent = result.confidence > 0.8 ? '✅' : '⚠️';
                
                if (result.valid) {
                    field.classList.add('valid');
                    field.classList.remove('invalid');
                    errorElement.textContent = '';
                    confidenceElement.textContent = `Доверие: ${(result.confidence * 100).toFixed(0)}%`;
                } else {
                    field.classList.add('invalid');
                    field.classList.remove('valid');
                    errorElement.textContent = result.message;
                    confidenceElement.textContent = `Доверие: ${(result.confidence * 100).toFixed(0)}%`;
                }
                
                return result.valid;
            }
            
            async mlValidatePassword(field) {
                const password = field.value;
                const errorElement = document.getElementById('ml-password-error');
                const confidenceElement = document.getElementById('password-confidence');
                const indicatorElement = document.getElementById('ml-password-indicator');
                
                if (!password) {
                    errorElement.textContent = '';
                    confidenceElement.textContent = '';
                    indicatorElement.textContent = '🤖';
                    return true;
                }
                
                // Симуляция ML-валидации
                indicatorElement.textContent = '⏳';
                
                const result = await this.simulateMLValidation('password', password);
                
                indicatorElement.textContent = result.confidence > 0.8 ? '✅' : '⚠️';
                
                if (result.valid) {
                    field.classList.add('valid');
                    field.classList.remove('invalid');
                    errorElement.textContent = '';
                    confidenceElement.textContent = `Доверие: ${(result.confidence * 100).toFixed(0)}%`;
                } else {
                    field.classList.add('invalid');
                    field.classList.remove('valid');
                    errorElement.textContent = result.message;
                    confidenceElement.textContent = `Доверие: ${(result.confidence * 100).toFixed(0)}%`;
                }
                
                return result.valid;
            }
            
            simulateMLValidation(type, value) {
                return new Promise(resolve => {
                    setTimeout(() => {
                        if (type === 'email') {
                            // Симуляция ML-модели для email
                            const isDisposable = /temp|mailinator|guerrillamail/.test(value);
                            const hasValidDomain = /\.[a-zA-Z]{2,}$/.test(value);
                            
                            if (isDisposable) {
                                resolve({
                                    valid: false,
                                    message: 'Временные email-адреса не разрешены',
                                    confidence: 0.95
                                });
                            } else if (!hasValidDomain) {
                                resolve({
                                    valid: false,
                                    message: 'Недействительный домен',
                                    confidence: 0.85
                                });
                            } else {
                                resolve({
                                    valid: true,
                                    message: '',
                                    confidence: 0.92
                                });
                            }
                        } else if (type === 'password') {
                            // Симуляция ML-модели для пароля
                            const hasCommonPattern = /(123456|password|qwerty|admin)/i.test(value);
                            const entropy = this.calculateEntropy(value);
                            
                            if (hasCommonPattern) {
                                resolve({
                                    valid: false,
                                    message: 'Пароль слишком простой',
                                    confidence: 0.98
                                });
                            } else if (entropy < 2.0) {
                                resolve({
                                    valid: false,
                                    message: 'Слишком предсказуемый пароль',
                                    confidence: 0.88
                                });
                            } else {
                                resolve({
                                    valid: true,
                                    message: '',
                                    confidence: 0.94
                                });
                            }
                        }
                    }, 300);
                });
            }
            
            calculateEntropy(password) {
                const chars = {};
                for (let i = 0; i < password.length; i++) {
                    const char = password[i];
                    chars[char] = (chars[char] || 0) + 1;
                }
                
                let entropy = 0;
                for (const char in chars) {
                    const prob = chars[char] / password.length;
                    entropy -= prob * Math.log2(prob);
                }
                
                return entropy;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new MLValidator();
        });
    </script>
</body>
</html>
```

## Проверка и тестирование валидации

### Инструменты для тестирования валидации:
1. **Jest** - для тестирования JavaScript валидации
2. **Cypress** - для интеграционного тестирования форм
3. **Puppeteer** - для автоматизированного тестирования
4. **Lighthouse** - для проверки доступности валидации
5. **axe-core** - для проверки доступности сообщений об ошибках

### Проверка доступности валидации:
1. Убедитесь, что сообщения об ошибках доступны для скринридеров
2. Проверьте, что фокус устанавливается на поле с ошибкой
3. Убедитесь, что ошибки описываются понятным языком
4. Проверьте, что валидация работает с клавиатуры

## Лучшие практики валидации форм

### 1. Безопасность валидации
```html
<!-- Безопасная валидация с CSRF защитой -->
<form id="secure-validation-form">
    <input type="hidden" name="csrf_token" value="generated_csrf_token">
    
    <div class="form-group">
        <label for="secure-email">Email:</label>
        <input type="email" id="secure-email" name="email" required>
        <div class="error-message" id="secure-email-error"></div>
    </div>
    
    <button type="submit">Отправить</button>
</form>

<script>
class SecureValidator {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.csrfToken = document.querySelector('input[name="csrf_token"]').value;
        this.setupValidation();
    }
    
    setupValidation() {
        this.form.addEventListener('submit', async (e) => {
            e.preventDefault();
            
            // Проверка всех полей
            const isValid = await this.validateForm();
            if (isValid) {
                await this.submitSecurely();
            }
        });
    }
    
    async validateForm() {
        // Клиентская валидация
        const emailField = document.getElementById('secure-email');
        let isValid = true;
        
        if (!emailField.value || !this.isValidEmail(emailField.value)) {
            document.getElementById('secure-email-error').textContent = 'Введите действительный email';
            emailField.classList.add('invalid');
            isValid = false;
        }
        
        return isValid;
    }
    
    isValidEmail(email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }
    
    async submitSecurely() {
        const formData = new FormData(this.form);
        
        try {
            const response = await fetch('/api/secure-submit', {
                method: 'POST',
                headers: {
                    'X-CSRF-Token': this.csrfToken,
                    'X-Requested-With': 'XMLHttpRequest'
                },
                body: formData
            });
            
            if (response.ok) {
                alert('Форма успешно отправлена!');
            } else {
                const error = await response.json();
                alert('Ошибка: ' + error.message);
            }
        } catch (error) {
            alert('Ошибка сети при отправке формы');
        }
    }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
    new SecureValidator('secure-validation-form');
});
</script>
```

### 2. Производительность валидации
```html
<!-- Оптимизированная валидация с debounce -->
<form id="optimized-validation-form">
    <div class="form-group">
        <label for="optimized-search">Поиск:</label>
        <input type="text" id="optimized-search" name="search" placeholder="Введите для поиска...">
        <div class="error-message" id="search-error"></div>
    </div>
</form>

<script>
class OptimizedValidator {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.debounceTimers = {};
        this.setupValidation();
    }
    
    setupValidation() {
        // Используем debounce для полей, которые требуют частой валидации
        const searchField = document.getElementById('optimized-search');
        searchField.addEventListener('input', this.debounce((e) => {
            this.validateSearch(e.target);
        }, 500));
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
    
    async validateSearch(field) {
        const value = field.value.trim();
        const errorElement = document.getElementById('search-error');
        
        if (value.length < 3) {
            errorElement.textContent = 'Введите минимум 3 символа';
            field.classList.add('invalid');
            field.classList.remove('valid');
            return false;
        }
        
        // Симуляция API вызова
        field.classList.add('validating');
        errorElement.textContent = 'Проверка...';
        
        try {
            await new Promise(resolve => setTimeout(resolve, 300));
            field.classList.remove('validating');
            field.classList.add('valid');
            field.classList.remove('invalid');
            errorElement.textContent = '';
            return true;
        } catch (error) {
            field.classList.remove('validating');
            field.classList.add('invalid');
            field.classList.remove('valid');
            errorElement.textContent = 'Ошибка проверки';
            return false;
        }
    }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
    new OptimizedValidator('optimized-validation-form');
});
</script>
```

## Заключение

Современная валидация HTML форм включает в себя комплексный подход, объединяющий клиентскую и серверную валидацию, обеспечение доступности, улучшение пользовательского опыта и использование современных веб-технологий. Правильная реализация валидации помогает пользователям корректно заполнять формы и предотвращает отправку некорректных данных на сервер.

Ключевые аспекты современной валидации:
1. Сочетание клиентской и серверной валидации
2. Обеспечение доступности для всех пользователей
3. Понятные и информативные сообщения об ошибках
4. Производительность и оптимизация
5. Безопасность и защита от атак
6. Адаптивность и отзывчивость интерфейса
7. Совместимость с современными веб-стандартами

Эти практики позволяют создавать формы, которые не только эффективно проверяют данные, но и обеспечивают положительный опыт взаимодействия для пользователей.

## Следующие темы
- [[HTML/Формы/Доступность]]
- [[HTML/Доступность]]
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Формы/Безопасность]]

## Теги
#html #forms #validation #web-development #frontend #user-experience #data-validation #accessibility #javascript #security #web-workers #webassembly #service-workers #ml-validation #performance
# Современные элементы HTML форм

## Введение

Современные HTML формы включают в себя расширенный набор элементов и атрибутов, появившихся в HTML5 и более поздних спецификациях. Эти элементы обеспечивают лучший пользовательский опыт, встроенную валидацию и улучшенную семантику. В этой статье мы рассмотрим современные элементы форм, их возможности и лучшие практики использования.

## Расширенные типы полей ввода

### Типы для специфических данных

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Современные типы полей ввода</title>
</head>
<body>
    <form id="modern-inputs-form">
        <h2>Расширенные типы полей ввода</h2>
        
        <!-- Email поле с валидацией -->
        <div class="form-group">
            <label for="email">Email:</label>
            <input type="email"
                   id="email"
                   name="email"
                   required
                   placeholder="user@example.com"
                   autocomplete="email"
                   aria-describedby="email-help email-error">
            <div id="email-help" class="help-text">Введите действительный email адрес</div>
            <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <!-- URL поле -->
        <div class="form-group">
            <label for="website">Веб-сайт:</label>
            <input type="url"
                   id="website"
                   name="website"
                   placeholder="https://example.com"
                   autocomplete="url"
                   aria-describedby="website-help website-error">
            <div id="website-help" class="help-text">Введите полный URL (включая https://)</div>
            <div id="website-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <!-- Телефонное поле -->
        <div class="form-group">
            <label for="phone">Телефон:</label>
            <input type="tel"
                   id="phone"
                   name="phone"
                   placeholder="+7 (XXX) XXX-XX-XX"
                   autocomplete="tel"
                   aria-describedby="phone-help phone-error">
            <div id="phone-help" class="help-text">Введите номер телефона в международном формате</div>
            <div id="phone-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <!-- Числовое поле -->
        <div class="form-group">
            <label for="age">Возраст:</label>
            <input type="number"
                   id="age"
                   name="age"
                   min="1"
                   max="120"
                   placeholder="18"
                   aria-describedby="age-help age-error">
            <div id="age-help" class="help-text">Введите ваш возраст (1-120)</div>
            <div id="age-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <!-- Поле поиска -->
        <div class="form-group">
            <label for="search">Поиск:</label>
            <input type="search"
                   id="search"
                   name="search"
                   placeholder="Введите поисковый запрос..."
                   aria-describedby="search-help"
                   results="5">
            <div id="search-help" class="help-text">Поиск по сайту</div>
        </div>
        
        <!-- Цвет -->
        <div class="form-group">
            <label for="favorite-color">Любимый цвет:</label>
            <input type="color"
                   id="favorite-color"
                   name="favorite_color"
                   value="#3498db"
                   aria-describedby="color-help">
            <div id="color-help" class="help-text">Выберите ваш любимый цвет</div>
        </div>
        
        <!-- Дата -->
        <div class="form-group">
            <label for="birth-date">Дата рождения:</label>
            <input type="date"
                   id="birth-date"
                   name="birth_date"
                   max="2010-01-01"
                   aria-describedby="date-help date-error">
            <div id="date-help" class="help-text">Выберите вашу дату рождения</div>
            <div id="date-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <!-- Дата и время -->
        <div class="form-group">
            <label for="event-datetime">Дата и время события:</label>
            <input type="datetime-local"
                   id="event-datetime"
                   name="event_datetime"
                   aria-describedby="datetime-help">
            <div id="datetime-help" class="help-text">Выберите дату и время события</div>
        </div>
        
        <!-- Время -->
        <div class="form-group">
            <label for="appointment-time">Время встречи:</label>
            <input type="time"
                   id="appointment-time"
                   name="appointment_time"
                   min="09:00"
                   max="18:00"
                   aria-describedby="time-help">
            <div id="time-help" class="help-text">Выберите время встречи (09:00-18:00)</div>
        </div>
        
        <!-- Месяц -->
        <div class="form-group">
            <label for="birth-month">Месяц рождения:</label>
            <input type="month"
                   id="birth-month"
                   name="birth_month"
                   aria-describedby="month-help">
            <div id="month-help" class="help-text">Выберите месяц рождения</div>
        </div>
        
        <!-- Неделя -->
        <div class="form-group">
            <label for="work-week">Рабочая неделя:</label>
            <input type="week"
                   id="work-week"
                   name="work_week"
                   aria-describedby="week-help">
            <div id="week-help" class="help-text">Выберите рабочую неделю</div>
        </div>
        
        <!-- Диапазон -->
        <div class="form-group">
            <label for="satisfaction">Уровень удовлетворенности (1-10):</label>
            <input type="range"
                   id="satisfaction"
                   name="satisfaction"
                   min="1"
                   max="10"
                   value="5"
                   step="1"
                   aria-describedby="range-help">
            <output for="satisfaction" id="satisfaction-value">5</output>
            <div id="range-help" class="help-text">Оцените уровень удовлетворенности</div>
        </div>
        
        <button type="submit">Отправить</button>
    </form>

    <script>
        // Обработка изменения значения диапазона
        document.getElementById('satisfaction').addEventListener('input', function() {
            document.getElementById('satisfaction-value').textContent = this.value;
        });
        
        // Валидация формы
        document.getElementById('modern-inputs-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            // Сброс ошибок
            document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
            document.querySelectorAll('input').forEach(el => el.classList.remove('invalid'));
            
            let isValid = true;
            
            // Валидация email
            const email = document.getElementById('email');
            if (email.value && !isValidEmail(email.value)) {
                showError(email, 'Введите действительный email адрес');
                isValid = false;
            }
            
            // Валидация URL
            const website = document.getElementById('website');
            if (website.value && !isValidUrl(website.value)) {
                showError(website, 'Введите действительный URL');
                isValid = false;
            }
            
            // Валидация возраста
            const age = document.getElementById('age');
            if (age.value && (age.value < 1 || age.value > 120)) {
                showError(age, 'Возраст должен быть от 1 до 120');
                isValid = false;
            }
            
            if (isValid) {
                alert('Форма успешно валидирована!');
            }
        });
        
        function isValidEmail(email) {
            const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            return re.test(email);
        }
        
        function isValidUrl(url) {
            try {
                new URL(url);
                return true;
            } catch (e) {
                return false;
            }
        }
        
        function showError(field, message) {
            field.classList.add('invalid');
            const errorElement = document.getElementById(`${field.id}-error`);
            if (errorElement) {
                errorElement.textContent = message;
            }
        }
    </script>
    
    <style>
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
            box-sizing: border-box;
        }
        
        input:focus {
            outline: none;
            border-color: #007bff;
            box-shadow: 0 0 5px rgba(0, 123, 255, 0.3);
        }
        
        input.invalid {
            border-color: #dc3545;
            background-color: #fff5f5;
        }
        
        .help-text {
            font-size: 0.875em;
            color: #666;
            margin-top: 5px;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875em;
            margin-top: 5px;
        }
        
        #satisfaction-value {
            display: inline-block;
            margin-left: 10px;
            font-weight: bold;
        }
    </style>
</body>
</html>
```

## Современные атрибуты форм

### Атрибуты автозаполнения (autocomplete)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Атрибуты автозаполнения</title>
</head>
<body>
    <form id="autocomplete-form">
        <h2>Атрибуты автозаполнения</h2>
        
        <!-- Персональная информация -->
        <fieldset>
            <legend>Персональная информация</legend>
            
            <div class="form-group">
                <label for="given-name">Имя:</label>
                <input type="text"
                       id="given-name"
                       name="given_name"
                       autocomplete="given-name"
                       required>
            </div>
            
            <div class="form-group">
                <label for="family-name">Фамилия:</label>
                <input type="text"
                       id="family-name"
                       name="family_name"
                       autocomplete="family-name"
                       required>
            </div>
            
            <div class="form-group">
                <label for="email-autocomplete">Email:</label>
                <input type="email"
                       id="email-autocomplete"
                       name="email"
                       autocomplete="email"
                       required>
            </div>
            
            <div class="form-group">
                <label for="tel-autocomplete">Телефон:</label>
                <input type="tel"
                       id="tel-autocomplete"
                       name="tel"
                       autocomplete="tel"
                       required>
            </div>
        </fieldset>
        
        <!-- Адрес -->
        <fieldset>
            <legend>Адрес доставки</legend>
            
            <div class="form-group">
                <label for="country">Страна:</label>
                <input type="text"
                       id="country"
                       name="country"
                       autocomplete="country"
                       required>
            </div>
            
            <div class="form-group">
                <label for="address-level1">Область/Штат:</label>
                <input type="text"
                       id="address-level1"
                       name="address_level1"
                       autocomplete="address-level1"
                       required>
            </div>
            
            <div class="form-group">
                <label for="address-level2">Город:</label>
                <input type="text"
                       id="address-level2"
                       name="address_level2"
                       autocomplete="address-level2"
                       required>
            </div>
            
            <div class="form-group">
                <label for="address-line1">Улица и номер дома:</label>
                <input type="text"
                       id="address-line1"
                       name="address_line1"
                       autocomplete="address-line1"
                       required>
            </div>
            
            <div class="form-group">
                <label for="postal-code">Почтовый индекс:</label>
                <input type="text"
                       id="postal-code"
                       name="postal_code"
                       autocomplete="postal-code"
                       required>
            </div>
        </fieldset>
        
        <!-- Платежная информация -->
        <fieldset>
            <legend>Платежная информация</legend>
            
            <div class="form-group">
                <label for="cc-name">Имя на карте:</label>
                <input type="text"
                       id="cc-name"
                       name="cc_name"
                       autocomplete="cc-name"
                       required>
            </div>
            
            <div class="form-group">
                <label for="cc-number">Номер карты:</label>
                <input type="text"
                       id="cc-number"
                       name="cc_number"
                       autocomplete="cc-number"
                       required>
            </div>
            
            <div class="form-group">
                <label for="cc-exp">Срок действия:</label>
                <input type="text"
                       id="cc-exp"
                       name="cc_exp"
                       autocomplete="cc-exp"
                       required>
            </div>
            
            <div class="form-group">
                <label for="cc-csc">CVC:</label>
                <input type="text"
                       id="cc-csc"
                       name="cc_csc"
                       autocomplete="cc-csc"
                       required>
            </div>
        </fieldset>
        
        <button type="submit">Отправить</button>
    </form>
    
    <style>
        fieldset {
            margin-bottom: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        legend {
            font-weight: bold;
            padding: 0 10px;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
            box-sizing: border-box;
        }
        
        input:focus {
            outline: none;
            border-color: #007bff;
            box-shadow: 0 0 5px rgba(0, 123, 255, 0.3);
        }
    </style>
</body>
</html>
```

### Атрибуты валидации

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Атрибуты валидации</title>
</head>
<body>
    <form id="validation-attributes-form">
        <h2>Атрибуты валидации</h2>
        
        <!-- Обязательные поля -->
        <div class="form-group">
            <label for="required-field">Обязательное поле:</label>
            <input type="text"
                   id="required-field"
                   name="required_field"
                   required
                   aria-describedby="required-help">
            <div id="required-help" class="help-text">Это поле обязательно для заполнения</div>
        </div>
        
        <!-- Ограничения длины -->
        <div class="form-group">
            <label for="min-max-length">Длина текста (3-20):</label>
            <input type="text"
                   id="min-max-length"
                   name="min_max_length"
                   required
                   minlength="3"
                   maxlength="20"
                   aria-describedby="length-help">
            <div id="length-help" class="help-text">Введите от 3 до 20 символов</div>
        </div>
        
        <!-- Числовые ограничения -->
        <div class="form-group">
            <label for="number-range">Число (10-100):</label>
            <input type="number"
                   id="number-range"
                   name="number_range"
                   required
                   min="10"
                   max="100"
                   step="5"
                   aria-describedby="number-help">
            <div id="number-help" class="help-text">Введите число от 10 до 100 с шагом 5</div>
        </div>
        
        <!-- Проверка по регулярному выражению -->
        <div class="form-group">
            <label for="pattern-field">Имя пользователя (только буквы и цифры):</label>
            <input type="text"
                   id="pattern-field"
                   name="pattern_field"
                   required
                   pattern="[A-Za-z0-9]+"
                   title="Только буквы и цифры"
                   aria-describedby="pattern-help">
            <div id="pattern-help" class="help-text">Только латинские буквы и цифры</div>
        </div>
        
        <!-- Пользовательское сообщение об ошибке -->
        <div class="form-group">
            <label for="custom-validation">Пароль (8+ символов):</label>
            <input type="password"
                   id="custom-validation"
                   name="custom_validation"
                   required
                   minlength="8"
                   aria-describedby="password-help password-error">
            <div id="password-help" class="help-text">Минимум 8 символов</div>
            <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <!-- Только для чтения -->
        <div class="form-group">
            <label for="readonly-field">ID пользователя:</label>
            <input type="text"
                   id="readonly-field"
                   name="readonly_field"
                   value="user_12345"
                   readonly
                   aria-describedby="readonly-help">
            <div id="readonly-help" class="help-text">Это поле нельзя редактировать</div>
        </div>
        
        <!-- Отключенное поле -->
        <div class="form-group">
            <label for="disabled-field">Старый email:</label>
            <input type="email"
                   id="disabled-field"
                   name="disabled_field"
                   value="old@example.com"
                   disabled
                   aria-describedby="disabled-help">
            <div id="disabled-help" class="help-text">Это поле отключено</div>
        </div>
        
        <!-- Автофокус -->
        <div class="form-group">
            <label for="autofocus-field">Имя:</label>
            <input type="text"
                   id="autofocus-field"
                   name="autofocus_field"
                   autofocus
                   required
                   aria-describedby="autofocus-help">
            <div id="autofocus-help" class="help-text">Это поле получит фокус при загрузке</div>
        </div>
        
        <!-- Плейсхолдер -->
        <div class="form-group">
            <label for="placeholder-field">Поиск:</label>
            <input type="search"
                   id="placeholder-field"
                   name="placeholder_field"
                   placeholder="Введите поисковый запрос..."
                   aria-describedby="placeholder-help">
            <div id="placeholder-help" class="help-text">Введите поисковый запрос</div>
        </div>
        
        <button type="submit">Отправить</button>
    </form>
    
    <script>
        // Обработка отправки формы с кастомной валидацией
        document.getElementById('validation-attributes-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            // Сброс ошибок
            document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
            document.querySelectorAll('input').forEach(el => el.classList.remove('invalid'));
            
            let isValid = true;
            
            // Валидация пароля
            const password = document.getElementById('custom-validation');
            if (password.value.length < 8) {
                password.classList.add('invalid');
                document.getElementById('password-error').textContent = 'Пароль должен содержать минимум 8 символов';
                isValid = false;
            }
            
            if (isValid) {
                alert('Форма успешно валидирована!');
            }
        });
    </script>
    
    <style>
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
            box-sizing: border-box;
        }
        
        input:focus {
            outline: none;
            border-color: #007bff;
            box-shadow: 0 0 5px rgba(0, 123, 255, 0.3);
        }
        
        input:invalid {
            border-color: #dc3545;
        }
        
        input:valid:not(:placeholder-shown) {
            border-color: #28a745;
        }
        
        input.invalid {
            border-color: #dc3545;
            background-color: #fff5f5;
        }
        
        .help-text {
            font-size: 0.875em;
            color: #666;
            margin-top: 5px;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875em;
            margin-top: 5px;
        }
    </style>
</body>
</html>
```

## Современные элементы формы

### Элементы <datalist> и <output>

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Современные элементы формы</title>
</head>
<body>
    <form id="modern-elements-form">
        <h2>Современные элементы формы</h2>
        
        <!-- Использование datalist для автодополнения -->
        <div class="form-group">
            <label for="country-select">Страна:</label>
            <input type="text"
                   id="country-select"
                   name="country"
                   list="countries"
                   placeholder="Начните вводить название страны..."
                   aria-describedby="country-help">
            <datalist id="countries">
                <option value="Россия">
                <option value="США">
                <option value="Канада">
                <option value="Германия">
                <option value="Франция">
                <option value="Япония">
                <option value="Китай">
                <option value="Индия">
                <option value="Бразилия">
                <option value="Великобритания">
            </datalist>
            <div id="country-help" class="help-text">Выберите или введите страну</div>
        </div>
        
        <!-- Использование datalist для числовых значений -->
        <div class="form-group">
            <label for="browser-select">Веб-браузер:</label>
            <input type="text"
                   id="browser-select"
                   name="browser"
                   list="browsers"
                   placeholder="Выберите браузер...">
            <datalist id="browsers">
                <option value="Chrome">
                <option value="Firefox">
                <option value="Safari">
                <option value="Edge">
                <option value="Opera">
            </datalist>
        </div>
        
        <!-- Использование output для отображения вычислений -->
        <div class="form-group">
            <label for="quantity">Количество:</label>
            <input type="number"
                   id="quantity"
                   name="quantity"
                   min="1"
                   max="100"
                   value="1"
                   oninput="calculateTotal()">
            
            <label for="price">Цена за единицу:</label>
            <input type="number"
                   id="price"
                   name="price"
                   min="0.01"
                   step="0.01"
                   value="10.00"
                   oninput="calculateTotal()">
            
            <div>Общая стоимость: <output name="total" id="total">10.00</output> руб.</div>
        </div>
        
        <!-- Расчет скидки -->
        <div class="form-group">
            <label for="discount">Скидка (%):</label>
            <input type="number"
                   id="discount"
                   name="discount"
                   min="0"
                   max="100"
                   value="0"
                   oninput="calculateTotal()">
            
            <div>Итоговая стоимость: <output name="final_total" id="final_total">10.00</output> руб.</div>
        </div>
        
        <!-- Пример сложного расчета -->
        <div class="form-group">
            <label for="loan-amount">Сумма кредита (руб.):</label>
            <input type="number"
                   id="loan-amount"
                   name="loan_amount"
                   min="10000"
                   max="10000000"
                   value="1000000"
                   oninput="calculateLoan()">
            
            <label for="loan-term">Срок кредита (месяцы):</label>
            <input type="number"
                   id="loan-term"
                   name="loan_term"
                   min="6"
                   max="360"
                   value="120"
                   oninput="calculateLoan()">
            
            <label for="interest-rate">Процентная ставка (%):</label>
            <input type="number"
                   id="interest-rate"
                   name="interest_rate"
                   min="1"
                   max="50"
                   step="0.1"
                   value="10"
                   oninput="calculateLoan()">
            
            <div>Ежемесячный платеж: <output name="monthly_payment" id="monthly_payment">0</output> руб.</div>
            <div>Общая переплата: <output name="overpayment" id="overpayment">0</output> руб.</div>
        </div>
        
        <button type="submit">Отправить</button>
    </form>
    
    <script>
        function calculateTotal() {
            const quantity = parseFloat(document.getElementById('quantity').value) || 0;
            const price = parseFloat(document.getElementById('price').value) || 0;
            const discount = parseFloat(document.getElementById('discount').value) || 0;
            
            const subtotal = quantity * price;
            const discountAmount = subtotal * (discount / 100);
            const total = subtotal - discountAmount;
            
            document.getElementById('total').value = subtotal.toFixed(2);
            document.getElementById('final_total').value = total.toFixed(2);
        }
        
        function calculateLoan() {
            const amount = parseFloat(document.getElementById('loan-amount').value) || 0;
            const term = parseFloat(document.getElementById('loan-term').value) || 0;
            const rate = parseFloat(document.getElementById('interest-rate').value) || 0;
            
            if (amount > 0 && term > 0 && rate > 0) {
                // Простой расчет аннуитетного платежа
                const monthlyRate = (rate / 100) / 12;
                const payment = amount * monthlyRate * Math.pow(1 + monthlyRate, term) / 
                               (Math.pow(1 + monthlyRate, term) - 1);
                
                const totalPayment = payment * term;
                const overpayment = totalPayment - amount;
                
                document.getElementById('monthly_payment').value = payment.toFixed(2);
                document.getElementById('overpayment').value = overpayment.toFixed(2);
            } else {
                document.getElementById('monthly_payment').value = '0';
                document.getElementById('overpayment').value = '0';
            }
        }
        
        // Инициализация расчетов
        calculateTotal();
        calculateLoan();
    </script>
    
    <style>
        .form-group {
            margin-bottom: 20px;
            padding: 15px;
            border: 1px solid #eee;
            border-radius: 4px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
            box-sizing: border-box;
            margin-bottom: 10px;
        }
        
        output {
            font-weight: bold;
            color: #007bff;
        }
    </style>
</body>
</html>
```

## Современные шаблоны форм

### Форма с динамическими полями

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Форма с динамическими полями</title>
</head>
<body>
    <form id="dynamic-fields-form">
        <h2>Форма с динамическими полями</h2>
        
        <!-- Основная информация -->
        <fieldset>
            <legend>Основная информация</legend>
            
            <div class="form-group">
                <label for="main-name">Имя:</label>
                <input type="text" id="main-name" name="main_name" required>
            </div>
            
            <div class="form-group">
                <label for="main-email">Email:</label>
                <input type="email" id="main-email" name="main_email" required>
            </div>
        </fieldset>
        
        <!-- Динамические контактные данные -->
        <fieldset id="contacts-container">
            <legend>Контактные данные</legend>
            
            <div class="contact-group" data-index="0">
                <h4>Контакт 1</h4>
                
                <div class="form-row">
                    <div class="form-group">
                        <label for="contact-type-0">Тип контакта:</label>
                        <select id="contact-type-0" name="contact_type_0" class="contact-type">
                            <option value="phone">Телефон</option>
                            <option value="email">Email</option>
                            <option value="social">Социальная сеть</option>
                            <option value="other">Другое</option>
                        </select>
                    </div>
                    
                    <div class="form-group">
                        <label for="contact-value-0">Значение:</label>
                        <input type="text" 
                               id="contact-value-0" 
                               name="contact_value_0" 
                               class="contact-value"
                               required>
                    </div>
                </div>
                
                <button type="button" 
                        class="remove-contact" 
                        data-index="0"
                        onclick="removeContact(0)"
                        aria-label="Удалить контакт 1">
                    Удалить
                </button>
            </div>
        </fieldset>
        
        <button type="button" onclick="addContact()" id="add-contact-btn">
            + Добавить контакт
        </button>
        
        <!-- Динамические навыки -->
        <fieldset id="skills-container">
            <legend>Навыки</legend>
            
            <div class="skill-group" data-index="0">
                <h4>Навык 1</h4>
                
                <div class="form-group">
                    <label for="skill-name-0">Название навыка:</label>
                    <input type="text" 
                           id="skill-name-0" 
                           name="skill_name_0" 
                           class="skill-name"
                           required>
                </div>
                
                <div class="form-group">
                    <label for="skill-level-0">Уровень (1-5):</label>
                    <input type="range" 
                           id="skill-level-0" 
                           name="skill_level_0" 
                           class="skill-level"
                           min="1" 
                           max="5" 
                           value="3">
                    <output for="skill-level-0" class="skill-level-value">3</output>
                </div>
                
                <button type="button" 
                        class="remove-skill" 
                        data-index="0"
                        onclick="removeSkill(0)"
                        aria-label="Удалить навык 1">
                    Удалить
                </button>
            </div>
        </fieldset>
        
        <button type="button" onclick="addSkill()" id="add-skill-btn">
            + Добавить навык
        </button>
        
        <button type="submit">Отправить</button>
    </form>
    
    <script>
        let contactIndex = 1;
        let skillIndex = 1;
        
        // Добавление нового контакта
        function addContact() {
            const container = document.getElementById('contacts-container');
            const newContact = document.createElement('div');
            newContact.className = 'contact-group';
            newContact.setAttribute('data-index', contactIndex);
            
            newContact.innerHTML = `
                <h4>Контакт ${contactIndex + 1}</h4>
                
                <div class="form-row">
                    <div class="form-group">
                        <label for="contact-type-${contactIndex}">Тип контакта:</label>
                        <select id="contact-type-${contactIndex}" name="contact_type_${contactIndex}" class="contact-type">
                            <option value="phone">Телефон</option>
                            <option value="email">Email</option>
                            <option value="social">Социальная сеть</option>
                            <option value="other">Другое</option>
                        </select>
                    </div>
                    
                    <div class="form-group">
                        <label for="contact-value-${contactIndex}">Значение:</label>
                        <input type="text" 
                               id="contact-value-${contactIndex}" 
                               name="contact_value_${contactIndex}" 
                               class="contact-value"
                               required>
                    </div>
                </div>
                
                <button type="button" 
                        class="remove-contact" 
                        data-index="${contactIndex}"
                        onclick="removeContact(${contactIndex})"
                        aria-label="Удалить контакт ${contactIndex + 1}">
                    Удалить
                </button>
            `;
            
            container.appendChild(newContact);
            contactIndex++;
        }
        
        // Удаление контакта
        function removeContact(index) {
            const contactGroup = document.querySelector(`.contact-group[data-index="${index}"]`);
            if (contactGroup) {
                contactGroup.remove();
            }
        }
        
        // Добавление нового навыка
        function addSkill() {
            const container = document.getElementById('skills-container');
            const newSkill = document.createElement('div');
            newSkill.className = 'skill-group';
            newSkill.setAttribute('data-index', skillIndex);
            
            newSkill.innerHTML = `
                <h4>Навык ${skillIndex + 1}</h4>
                
                <div class="form-group">
                    <label for="skill-name-${skillIndex}">Название навыка:</label>
                    <input type="text" 
                           id="skill-name-${skillIndex}" 
                           name="skill_name_${skillIndex}" 
                           class="skill-name"
                           required>
                </div>
                
                <div class="form-group">
                    <label for="skill-level-${skillIndex}">Уровень (1-5):</label>
                    <input type="range" 
                           id="skill-level-${skillIndex}" 
                           name="skill_level_${skillIndex}" 
                           class="skill-level"
                           min="1" 
                           max="5" 
                           value="3"
                           oninput="updateSkillLevel(${skillIndex})">
                    <output for="skill-level-${skillIndex}" class="skill-level-value">3</output>
                </div>
                
                <button type="button" 
                        class="remove-skill" 
                        data-index="${skillIndex}"
                        onclick="removeSkill(${skillIndex})"
                        aria-label="Удалить навык ${skillIndex + 1}">
                    Удалить
                </button>
            `;
            
            container.appendChild(newSkill);
            skillIndex++;
        }
        
        // Удаление навыка
        function removeSkill(index) {
            const skillGroup = document.querySelector(`.skill-group[data-index="${index}"]`);
            if (skillGroup) {
                skillGroup.remove();
            }
        }
        
        // Обновление значения уровня навыка
        function updateSkillLevel(index) {
            const slider = document.getElementById(`skill-level-${index}`);
            const output = slider.parentNode.querySelector('.skill-level-value');
            output.textContent = slider.value;
        }
        
        // Валидация формы
        document.getElementById('dynamic-fields-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            let isValid = true;
            
            // Проверка основных полей
            const mainName = document.getElementById('main-name');
            const mainEmail = document.getElementById('main-email');
            
            if (!mainName.value.trim()) {
                alert('Пожалуйста, введите имя');
                isValid = false;
            }
            
            if (!mainEmail.value.trim()) {
                alert('Пожалуйста, введите email');
                isValid = false;
            } else if (!isValidEmail(mainEmail.value)) {
                alert('Пожалуйста, введите действительный email');
                isValid = false;
            }
            
            // Проверка динамических контактов
            const contactValues = document.querySelectorAll('.contact-value');
            contactValues.forEach((contact, index) => {
                if (!contact.value.trim()) {
                    alert(`Пожалуйста, заполните значение для контакта ${index + 1}`);
                    isValid = false;
                }
            });
            
            // Проверка динамических навыков
            const skillNames = document.querySelectorAll('.skill-name');
            skillNames.forEach((skill, index) => {
                if (!skill.value.trim()) {
                    alert(`Пожалуйста, введите название для навыка ${index + 1}`);
                    isValid = false;
                }
            });
            
            if (isValid) {
                alert('Форма успешно валидирована!');
                // Здесь могла бы быть отправка формы
            }
        });
        
        function isValidEmail(email) {
            const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            return re.test(email);
        }
    </script>
    
    <style>
        fieldset {
            margin-bottom: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        legend {
            font-weight: bold;
            padding: 0 10px;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        .form-row {
            display: flex;
            gap: 15px;
        }
        
        .form-row .form-group {
            flex: 1;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
            box-sizing: border-box;
        }
        
        button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 4px;
            cursor: pointer;
            margin-top: 5px;
        }
        
        button:hover {
            background-color: #0056b3;
        }
        
        .remove-contact, .remove-skill {
            background-color: #dc3545;
        }
        
        .remove-contact:hover, .remove-skill:hover {
            background-color: #c82333;
        }
        
        .contact-group, .skill-group {
            border: 1px solid #eee;
            padding: 15px;
            margin-bottom: 15px;
            border-radius: 4px;
            position: relative;
        }
        
        .skill-level-value {
            margin-left: 10px;
            font-weight: bold;
        }
    </style>
</body>
</html>
```

## Современные возможности файловых загрузок

### Расширенные возможности загрузки файлов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Современные возможности загрузки файлов</title>
</head>
<body>
    <form id="file-upload-form">
        <h2>Современные возможности загрузки файлов</h2>
        
        <!-- Простая загрузка файла -->
        <div class="form-group">
            <label for="simple-file">Загрузить файл:</label>
            <input type="file" 
                   id="simple-file" 
                   name="simple_file"
                   accept=".pdf,.doc,.docx,.txt"
                   aria-describedby="simple-file-help">
            <div id="simple-file-help" class="help-text">Допустимые форматы: PDF, DOC, DOCX, TXT</div>
        </div>
        
        <!-- Загрузка нескольких файлов -->
        <div class="form-group">
            <label for="multiple-files">Загрузить несколько файлов:</label>
            <input type="file" 
                   id="multiple-files" 
                   name="multiple_files"
                   multiple
                   accept="image/*"
                   aria-describedby="multiple-files-help">
            <div id="multiple-files-help" class="help-text">Загрузите несколько изображений (JPEG, PNG, GIF)</div>
        </div>
        
        <!-- Загрузка с ограничением размера -->
        <div class="form-group">
            <label for="size-limited-file">Файл до 5MB:</label>
            <input type="file" 
                   id="size-limited-file" 
                   name="size_limited_file"
                   accept=".jpg,.jpeg,.png"
                   aria-describedby="size-help">
            <div id="size-help" class="help-text">Изображение до 5MB, форматы JPG, JPEG, PNG</div>
        </div>
        
        <!-- Загрузка с использованием камеры -->
        <div class="form-group">
            <label for="camera-upload">Фото с камеры:</label>
            <input type="file" 
                   id="camera-upload" 
                   name="camera_upload"
                   accept="image/*"
                   capture="environment">
            <div class="help-text">Сделайте фото на мобильном устройстве</div>
        </div>
        
        <!-- Зона перетаскивания файлов -->
        <div class="form-group">
            <label>Перетащите файлы сюда:</label>
            <div id="drop-zone" class="drop-zone" ondrop="dropHandler(event);" ondragover="dragOverHandler(event);">
                <p>Перетащите файлы сюда или нажмите для выбора</p>
                <input type="file" 
                       id="drop-file-input" 
                       name="drop_files"
                       multiple
                       style="display: none;"
                       accept="image/*,.pdf">
            </div>
            <button type="button" onclick="document.getElementById('drop-file-input').click()">
                Выбрать файлы
            </button>
        </div>
        
        <!-- Прогресс-бар загрузки -->
        <div class="form-group">
            <label for="upload-progress">Загрузка файла:</label>
            <input type="file" 
                   id="upload-progress" 
                   name="upload_progress"
                   accept=".pdf">
            <div class="progress-container">
                <div class="progress-bar" id="progress-bar" style="width: 0%;"></div>
                <div class="progress-text" id="progress-text">0%</div>
            </div>
        </div>
        
        <button type="submit">Отправить</button>
    </form>
    
    <script>
        // Обработчик зоны перетаскивания
        function dropHandler(ev) {
            ev.preventDefault();
            
            // Получаем файлы из события
            const files = ev.dataTransfer.files;
            
            // Отображаем информацию о файлах
            displayDroppedFiles(files);
        }
        
        function dragOverHandler(ev) {
            ev.preventDefault();
        }
        
        function displayDroppedFiles(files) {
            const dropZone = document.getElementById('drop-zone');
            dropZone.innerHTML = '<p>Выбранные файлы:</p>';
            
            for (let i = 0; i < files.length; i++) {
                const file = files[i];
                const fileDiv = document.createElement('div');
                fileDiv.className = 'file-item';
                fileDiv.innerHTML = `
                    <div class="file-name">${file.name}</div>
                    <div class="file-size">${formatFileSize(file.size)}</div>
                    <div class="file-type">${file.type || 'Неизвестный'}</div>
                `;
                dropZone.appendChild(fileDiv);
            }
        }
        
        function formatFileSize(bytes) {
            if (bytes === 0) return '0 Bytes';
            const k = 1024;
            const sizes = ['Bytes', 'KB', 'MB', 'GB'];
            const i = Math.floor(Math.log(bytes) / Math.log(k));
            return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
        }
        
        // Симуляция прогресса загрузки
        document.getElementById('upload-progress').addEventListener('change', function() {
            simulateUploadProgress();
        });
        
        function simulateUploadProgress() {
            const progressBar = document.getElementById('progress-bar');
            const progressText = document.getElementById('progress-text');
            
            let progress = 0;
            const interval = setInterval(() => {
                progress += Math.random() * 10; // Случайный прогресс
                if (progress >= 100) {
                    progress = 100;
                    clearInterval(interval);
                }
                
                progressBar.style.width = `${progress}%`;
                progressText.textContent = `${Math.round(progress)}%`;
            }, 200);
        }
        
        // Валидация файлов перед отправкой
        document.getElementById('file-upload-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            let isValid = true;
            
            // Проверка простого файла
            const simpleFile = document.getElementById('simple-file');
            if (simpleFile.files.length > 0) {
                const file = simpleFile.files[0];
                if (!validateFile(file, ['.pdf', '.doc', '.docx', '.txt'])) {
                    alert('Неверный формат файла для поля "Загрузить файл"');
                    isValid = false;
                }
            }
            
            // Проверка файлов с ограничением размера
            const sizeLimitedFile = document.getElementById('size-limited-file');
            if (sizeLimitedFile.files.length > 0) {
                const file = sizeLimitedFile.files[0];
                if (file.size > 5 * 1024 * 1024) { // 5MB
                    alert('Файл превышает 5MB');
                    isValid = false;
                }
                if (!validateFile(file, ['.jpg', '.jpeg', '.png'])) {
                    alert('Неверный формат изображения');
                    isValid = false;
                }
            }
            
            if (isValid) {
                alert('Форма успешно валидирована!');
            }
        });
        
        function validateFile(file, allowedExtensions) {
            const fileExtension = '.' + file.name.split('.').pop().toLowerCase();
            return allowedExtensions.includes(fileExtension);
        }
    </script>
    
    <style>
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input[type="file"] {
            margin-bottom: 10px;
        }
        
        .help-text {
            font-size: 0.875em;
            color: #666;
            margin-top: 5px;
        }
        
        .drop-zone {
            border: 2px dashed #ccc;
            border-radius: 4px;
            padding: 20px;
            text-align: center;
            margin-bottom: 10px;
            cursor: pointer;
        }
        
        .drop-zone:hover {
            border-color: #007bff;
        }
        
        .file-item {
            border: 1px solid #eee;
            padding: 10px;
            margin: 5px 0;
            border-radius: 4px;
        }
        
        .file-name {
            font-weight: bold;
            margin-bottom: 5px;
        }
        
        .file-size, .file-type {
            font-size: 0.875em;
            color: #666;
        }
        
        .progress-container {
            margin-top: 10px;
        }
        
        .progress-bar {
            height: 20px;
            background-color: #007bff;
            border-radius: 10px;
            transition: width 0.3s ease;
        }
        
        .progress-text {
            text-align: center;
            margin-top: 5px;
            font-weight: bold;
        }
        
        button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 8px 15px;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button:hover {
            background-color: #0056b3;
        }
    </style>
</body>
</html>
```

## Лучшие практики современных форм

### Комплексный пример современной формы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Комплексный пример современной формы</title>
    <meta name="description" content="Пример современной HTML формы с лучшими практиками">
</head>
<body>
    <main>
        <form id="modern-complex-form" novalidate>
            <header>
                <h1>Регистрация нового пользователя</h1>
                <p>Пожалуйста, заполните все обязательные поля, отмеченные звездочкой *</p>
            </header>
            
            <!-- Прогресс-бар -->
            <div class="progress-container" 
                 role="progressbar" 
                 aria-valuenow="0" 
                 aria-valuemin="0" 
                 aria-valuemax="100" 
                 aria-valuetext="Шаг 1 из 4: Персональная информация">
                <div class="progress-bar">
                    <div class="progress" id="form-progress" style="width: 0%;"></div>
                </div>
                <div class="progress-text" id="progress-text">Шаг 1 из 4: Персональная информация</div>
            </div>
            
            <!-- Шаг 1: Персональная информация -->
            <section class="form-step active" id="step1" role="region" aria-labelledby="step1-title">
                <h2 id="step1-title">Персональная информация</h2>
                
                <div class="form-row">
                    <div class="form-group">
                        <label for="first-name">Имя <span class="required" aria-label="обязательное поле">*</span></label>
                        <input type="text"
                               id="first-name"
                               name="first_name"
                               required
                               minlength="2"
                               maxlength="50"
                               autocomplete="given-name"
                               aria-describedby="first-name-error first-name-help"
                               aria-invalid="false">
                        <div id="first-name-help" class="help-text">Введите ваше имя</div>
                        <div id="first-name-error" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                    
                    <div class="form-group">
                        <label for="last-name">Фамилия <span class="required" aria-label="обязательное поле">*</span></label>
                        <input type="text"
                               id="last-name"
                               name="last_name"
                               required
                               minlength="2"
                               maxlength="50"
                               autocomplete="family-name"
                               aria-describedby="last-name-error last-name-help"
                               aria-invalid="false">
                        <div id="last-name-help" class="help-text">Введите вашу фамилию</div>
                        <div id="last-name-error" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                </div>
                
                <div class="form-group">
                    <label for="birth-date">Дата рождения</label>
                    <input type="date"
                           id="birth-date"
                           name="birth_date"
                           max="2010-01-01"
                           autocomplete="bday"
                           aria-describedby="birth-date-help birth-date-error"
                           aria-invalid="false">
                    <div id="birth-date-help" class="help-text">Введите вашу дату рождения</div>
                    <div id="birth-date-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label>Пол</label>
                    <div class="radio-group" role="radiogroup" aria-labelledby="gender-label">
                        <div id="gender-label" class="sr-only">Выберите ваш пол</div>
                        <label class="radio-label">
                            <input type="radio"
                                   name="gender"
                                   value="male"
                                   id="gender-male"
                                   autocomplete="sex">
                            <span class="radio-custom"></span>
                            Мужской
                        </label>
                        <label class="radio-label">
                            <input type="radio"
                                   name="gender"
                                   value="female"
                                   id="gender-female"
                                   autocomplete="sex">
                            <span class="radio-custom"></span>
                            Женский
                        </label>
                        <label class="radio-label">
                            <input type="radio"
                                   name="gender"
                                   value="other"
                                   id="gender-other">
                            <span class="radio-custom"></span>
                            Другой
                        </label>
                    </div>
                </div>
                
                <div class="form-actions">
                    <button type="button" class="btn btn-primary" onclick="nextStep(1)">Далее</button>
                </div>
            </section>
            
            <!-- Шаг 2: Контактная информация -->
            <section class="form-step" id="step2" role="region" aria-labelledby="step2-title" hidden>
                <h2 id="step2-title">Контактная информация</h2>
                
                <div class="form-group">
                    <label for="email">Email <span class="required" aria-label="обязательное поле">*</span></label>
                    <input type="email"
                           id="email"
                           name="email"
                           required
                           autocomplete="email"
                           aria-describedby="email-error email-help"
                           aria-invalid="false">
                    <div id="email-help" class="help-text">Для подтверждения регистрации и связи</div>
                    <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="phone">Телефон</label>
                    <input type="tel"
                           id="phone"
                           name="phone"
                           autocomplete="tel"
                           aria-describedby="phone-help phone-error"
                           aria-invalid="false">
                    <div id="phone-help" class="help-text">Формат: +7 (XXX) XXX-XX-XX</div>
                    <div id="phone-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="country">Страна</label>
                    <input type="text"
                           id="country"
                           name="country"
                           list="countries-list"
                           autocomplete="country-name"
                           aria-describedby="country-help"
                           aria-invalid="false">
                    <datalist id="countries-list">
                        <option value="Россия">
                        <option value="США">
                        <option value="Канада">
                        <option value="Германия">
                        <option value="Франция">
                        <option value="Япония">
                    </datalist>
                    <div id="country-help" class="help-text">Начните вводить название страны</div>
                </div>
                
                <div class="form-row">
                    <div class="form-group">
                        <label for="city">Город</label>
                        <input type="text"
                               id="city"
                               name="city"
                               autocomplete="address-level2"
                               aria-describedby="city-help"
                               aria-invalid="false">
                        <div id="city-help" class="help-text">Введите название города</div>
                    </div>
                    
                    <div class="form-group">
                        <label for="zip-code">Почтовый индекс</label>
                        <input type="text"
                               id="zip-code"
                               name="zip_code"
                               autocomplete="postal-code"
                               aria-describedby="zip-help"
                               aria-invalid="false">
                        <div id="zip-help" class="help-text">6 цифр</div>
                    </div>
                </div>
                
                <div class="form-group">
                    <label for="address">Адрес</label>
                    <textarea id="address"
                              name="address"
                              rows="3"
                              autocomplete="street-address"
                              aria-describedby="address-help"
                              aria-invalid="false"></textarea>
                    <div id="address-help" class="help-text">Полный почтовый адрес</div>
                </div>
                
                <div class="form-actions">
                    <button type="button" class="btn btn-secondary" onclick="prevStep(2)">Назад</button>
                    <button type="button" class="btn btn-primary" onclick="nextStep(2)">Далее</button>
                </div>
            </section>
            
            <!-- Шаг 3: Аккаунт -->
            <section class="form-step" id="step3" role="region" aria-labelledby="step3-title" hidden>
                <h2 id="step3-title">Информация аккаунта</h2>
                
                <div class="form-group">
                    <label for="username">Имя пользователя <span class="required" aria-label="обязательное поле">*</span></label>
                    <input type="text"
                           id="username"
                           name="username"
                           required
                           minlength="3"
                           maxlength="20"
                           pattern="[A-Za-z0-9_]+"
                           autocomplete="username"
                           aria-describedby="username-help username-error"
                           aria-invalid="false">
                    <div id="username-help" class="help-text">Только буквы, цифры и подчеркивание, 3-20 символов</div>
                    <div id="username-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="password">Пароль <span class="required" aria-label="обязательное поле">*</span></label>
                    <input type="password"
                           id="password"
                           name="password"
                           required
                           minlength="8"
                           aria-describedby="password-requirements password-error"
                           aria-invalid="false">
                    <div id="password-requirements" class="requirements-list">
                        <p>Требования к паролю:</p>
                        <ul>
                            <li id="req-length">Минимум 8 символов</li>
                            <li id="req-upper">С заглавной буквой</li>
                            <li id="req-lower">Со строчной буквой</li>
                            <li id="req-number">С цифрой</li>
                            <li id="req-special">С специальным символом</li>
                        </ul>
                    </div>
                    <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="confirm-password">Подтверждение пароля <span class="required" aria-label="обязательное поле">*</span></label>
                    <input type="password"
                           id="confirm-password"
                           name="confirm_password"
                           required
                           aria-describedby="confirm-password-error"
                           aria-invalid="false">
                    <div id="confirm-password-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox"
                               name="newsletter"
                               value="yes"
                               id="newsletter">
                        <span class="checkbox-custom"></span>
                        Подписаться на рассылку новостей
                    </label>
                </div>
                
                <div class="form-actions">
                    <button type="button" class="btn btn-secondary" onclick="prevStep(3)">Назад</button>
                    <button type="button" class="btn btn-primary" onclick="nextStep(3)">Далее</button>
                </div>
            </section>
            
            <!-- Шаг 4: Подтверждение -->
            <section class="form-step" id="step4" role="region" aria-labelledby="step4-title" hidden>
                <h2 id="step4-title">Подтверждение регистрации</h2>
                
                <div class="summary" role="region" aria-labelledby="summary-title">
                    <h3 id="summary-title">Ваша информация:</h3>
                    <div id="summary-content"></div>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox"
                               name="terms"
                               required
                               id="terms-agreement"
                               aria-describedby="terms-error">
                        <span class="checkbox-custom"></span>
                        Я согласен с <a href="/terms" target="_blank">условиями использования</a> и
                        <a href="/privacy" target="_blank">политикой конфиденциальности</a> <span class="required" aria-label="обязательное поле">*</span>
                    </label>
                    <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox"
                               name="marketing"
                               value="yes"
                               id="marketing-consent">
                        <span class="checkbox-custom"></span>
                        Я согласен на получение маркетинговых материалов
                    </label>
                </div>
                
                <div class="form-actions">
                    <button type="button" class="btn btn-secondary" onclick="prevStep(4)">Назад</button>
                    <button type="submit" class="btn btn-primary" id="submit-btn">
                        <span class="btn-text">Зарегистрироваться</span>
                        <span class="btn-loading" style="display: none;">Загрузка...</span>
                    </button>
                </div>
            </section>
        </form>
    </main>
    
    <script>
        let currentStep = 1;
        const totalSteps = 4;
        
        class ModernFormHandler {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupForm();
            }
            
            setupForm() {
                // Валидация в реальном времени
                this.setupRealTimeValidation();
                
                // Обработка отправки формы
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.submitForm();
                });
            }
            
            setupRealTimeValidation() {
                // Валидация пароля в реальном времени
                document.getElementById('password').addEventListener('input', () => {
                    this.validatePasswordRequirements();
                });
                
                // Валидация подтверждения пароля
                document.getElementById('confirm-password').addEventListener('input', () => {
                    this.validatePasswordMatch();
                });
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
            
            validatePasswordMatch() {
                const password = document.getElementById('password').value;
                const confirmPassword = document.getElementById('confirm-password').value;
                const errorElement = document.getElementById('confirm-password-error');
                
                if (confirmPassword && password !== confirmPassword) {
                    errorElement.textContent = 'Пароли не совпадают';
                    document.getElementById('confirm-password').classList.add('invalid');
                    document.getElementById('confirm-password').setAttribute('aria-invalid', 'true');
                    return false;
                } else {
                    errorElement.textContent = '';
                    document.getElementById('confirm-password').classList.remove('invalid');
                    document.getElementById('confirm-password').setAttribute('aria-invalid', 'false');
                    return true;
                }
            }
            
            async submitForm() {
                // Проверяем последний шаг
                if (!this.validateCurrentStep()) {
                    return;
                }
                
                // Проверяем согласие с условиями
                const termsCheckbox = document.getElementById('terms-agreement');
                if (!termsCheckbox.checked) {
                    const errorElement = document.getElementById('terms-error');
                    errorElement.textContent = 'Необходимо согласие с условиями использования';
                    termsCheckbox.setAttribute('aria-invalid', 'true');
                    termsCheckbox.focus();
                    return;
                }
                
                // Показываем индикатор загрузки
                const submitBtn = document.getElementById('submit-btn');
                const btnText = submitBtn.querySelector('.btn-text');
                const btnLoading = submitBtn.querySelector('.btn-loading');
                
                btnText.style.display = 'none';
                btnLoading.style.display = 'inline';
                submitBtn.disabled = true;
                
                try {
                    // Симуляция отправки формы
                    await new Promise(resolve => setTimeout(resolve, 2000));
                    
                    // Успешная регистрация
                    alert('Вы успешно зарегистрированы!');
                    this.form.reset();
                    goToStep(1);
                } catch (error) {
                    alert('Произошла ошибка при регистрации. Пожалуйста, попробуйте еще раз.');
                } finally {
                    btnText.style.display = 'inline';
                    btnLoading.style.display = 'none';
                    submitBtn.disabled = false;
                }
            }
            
            validateCurrentStep() {
                const stepElement = document.getElementById(`step${currentStep}`);
                const requiredFields = stepElement.querySelectorAll('[required]');
                let isValid = true;
                
                // Сбрасываем ошибки
                stepElement.querySelectorAll('.error-message').forEach(el => el.textContent = '');
                stepElement.querySelectorAll('[aria-invalid]').forEach(el => el.setAttribute('aria-invalid', 'false'));
                
                requiredFields.forEach(field => {
                    if (!field.value.trim()) {
                        const errorElement = document.getElementById(`${field.id}-error`);
                        if (errorElement) {
                            errorElement.textContent = `Поле "${field.labels[0]?.textContent || field.name}" обязательно для заполнения`;
                        }
                        field.setAttribute('aria-invalid', 'true');
                        field.classList.add('invalid');
                        isValid = false;
                        
                        // Фокус на первом невалидном поле
                        if (isValid === false) {
                            field.focus();
                        }
                    } else {
                        const errorElement = document.getElementById(`${field.id}-error`);
                        if (errorElement) {
                            errorElement.textContent = '';
                        }
                        field.setAttribute('aria-invalid', 'false');
                        field.classList.remove('invalid');
                    }
                });
                
                return isValid;
            }
        }
        
        function nextStep(current) {
            const formHandler = new ModernFormHandler('modern-complex-form');
            
            // Валидируем текущий шаг
            if (formHandler.validateCurrentStep()) {
                goToStep(current + 1);
            }
        }
        
        function prevStep(current) {
            goToStep(current - 1);
        }
        
        function goToStep(step) {
            // Скрываем текущий шаг
            document.getElementById(`step${currentStep}`).hidden = true;
            document.getElementById(`step${currentStep}`).classList.remove('active');
            
            // Показываем новый шаг
            currentStep = step;
            document.getElementById(`step${currentStep}`).hidden = false;
            document.getElementById(`step${currentStep}`).classList.add('active');
            
            // Обновляем прогресс
            updateProgress();
            
            // Фокус на заголовок шага
            const heading = document.getElementById(`step${currentStep}-title`);
            if (heading) {
                heading.setAttribute('tabindex', '-1');
                heading.focus();
                setTimeout(() => heading.removeAttribute('tabindex'), 100);
            }
            
            // Обновляем сводку на последнем шаге
            if (step === 4) {
                updateSummary();
            }
        }
        
        function updateProgress() {
            const progress = ((currentStep - 1) / (totalSteps - 1)) * 100;
            document.getElementById('form-progress').style.width = `${progress}%`;
            
            const stepTitles = [
                'Шаг 1 из 4: Персональная информация',
                'Шаг 2 из 4: Контактная информация', 
                'Шаг 3 из 4: Информация аккаунта',
                'Шаг 4 из 4: Подтверждение регистрации'
            ];
            
            document.getElementById('progress-text').textContent = stepTitles[currentStep - 1];
        }
        
        function updateSummary() {
            const formData = new FormData(document.getElementById('modern-complex-form'));
            const summaryContent = document.getElementById('summary-content');
            
            const firstName = formData.get('first_name') || 'Не указано';
            const lastName = formData.get('last_name') || 'Не указано';
            const email = formData.get('email') || 'Не указан';
            const phone = formData.get('phone') || 'Не указан';
            const username = formData.get('username') || 'Не указано';
            const birthDate = formData.get('birth_date') || 'Не указана';
            const gender = getGenderText(formData.get('gender'));
            const newsletter = formData.get('newsletter') ? 'Да' : 'Нет';
            const marketing = formData.get('marketing') ? 'Да' : 'Нет';
            
            summaryContent.innerHTML = `
                <ul>
                    <li><strong>Имя:</strong> ${firstName} ${lastName}</li>
                    <li><strong>Дата рождения:</strong> ${birthDate}</li>
                    <li><strong>Пол:</strong> ${gender}</li>
                    <li><strong>Email:</strong> ${email}</li>
                    <li><strong>Телефон:</strong> ${phone}</li>
                    <li><strong>Имя пользователя:</strong> ${username}</li>
                    <li><strong>Подписка на новости:</strong> ${newsletter}</li>
                    <li><strong>Маркетинговое согласие:</strong> ${marketing}</li>
                </ul>
            `;
        }
        
        function getGenderText(genderValue) {
            const genderMap = {
                'male': 'Мужской',
                'female': 'Женский',
                'other': 'Другой'
            };
            return genderMap[genderValue] || 'Не указан';
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new ModernFormHandler('modern-complex-form');
            updateProgress();
        });
    </script>
    
    <style>
        :root {
            --primary-color: #007bff;
            --secondary-color: #6c757d;
            --success-color: #28a745;
            --danger-color: #dc3545;
            --light-color: #f8f9fa;
            --dark-color: #343a40;
        }
        
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            color: #333;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        header {
            text-align: center;
            margin-bottom: 30px;
        }
        
        h1 {
            color: var(--dark-color);
            margin-bottom: 10px;
        }
        
        .progress-container {
            margin-bottom: 30px;
        }
        
        .progress-bar {
            height: 20px;
            background-color: #e9ecef;
            border-radius: 10px;
            overflow: hidden;
            margin-bottom: 10px;
        }
        
        .progress {
            height: 100%;
            background-color: var(--primary-color);
            transition: width 0.3s ease;
        }
        
        .progress-text {
            text-align: center;
            font-weight: bold;
            color: #666;
        }
        
        .form-step {
            display: none;
        }
        
        .form-step.active {
            display: block;
            animation: fadeIn 0.5s ease-in;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .form-row {
            display: flex;
            gap: 20px;
            margin-bottom: 20px;
        }
        
        .form-row .form-group {
            flex: 1;
        }
        
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        .required {
            color: var(--danger-color);
        }
        
        input, select, textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
            box-sizing: border-box;
        }
        
        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: var(--primary-color);
            box-shadow: 0 0 5px rgba(0, 123, 255, 0.3);
        }
        
        input:invalid:not(:focus):not(:placeholder-shown) {
            border-color: var(--danger-color);
        }
        
        input:valid:not(:placeholder-shown) {
            border-color: var(--success-color);
        }
        
        input.invalid {
            border-color: var(--danger-color);
            background-color: #fff5f5;
        }
        
        textarea {
            resize: vertical;
            min-height: 100px;
        }
        
        .radio-group {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
        }
        
        .radio-label {
            display: flex;
            align-items: center;
            cursor: pointer;
            font-weight: normal;
        }
        
        .radio-custom {
            position: relative;
            height: 20px;
            width: 20px;
            background-color: white;
            border: 2px solid #ccc;
            border-radius: 50%;
            margin-right: 8px;
            transition: all 0.3s ease;
        }
        
        input[type="radio"]:checked + .radio-custom {
            background-color: var(--primary-color);
            border-color: var(--primary-color);
        }
        
        input[type="radio"]:checked + .radio-custom::after {
            content: "";
            position: absolute;
            display: block;
            top: 50%;
            left: 50%;
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: white;
            transform: translate(-50%, -50%);
        }
        
        .checkbox-custom {
            position: relative;
            height: 20px;
            width: 20px;
            background-color: white;
            border: 2px solid #ccc;
            border-radius: 4px;
            margin-right: 8px;
            transition: all 0.3s ease;
        }
        
        input[type="checkbox"]:checked + .checkbox-custom {
            background-color: var(--primary-color);
            border-color: var(--primary-color);
        }
        
        input[type="checkbox"]:checked + .checkbox-custom::after {
            content: "";
            position: absolute;
            display: block;
            left: 6px;
            top: 2px;
            width: 5px;
            height: 10px;
            border: solid white;
            border-width: 0 2px 2px 0;
            transform: rotate(45deg);
        }
        
        .help-text {
            font-size: 0.875em;
            color: #666;
            margin-top: 5px;
        }
        
        .error-message {
            color: var(--danger-color);
            font-size: 0.875em;
            margin-top: 5px;
            font-weight: bold;
        }
        
        .requirements-list {
            margin-top: 10px;
        }
        
        .requirements-list ul {
            list-style: none;
            padding: 0;
        }
        
        .requirements-list li {
            padding: 2px 0;
            font-size: 0.9em;
            color: #666;
        }
        
        .requirements-list li.met {
            color: var(--success-color);
            font-weight: bold;
        }
        
        .form-actions {
            margin-top: 30px;
            display: flex;
            justify-content: space-between;
        }
        
        .btn {
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            text-decoration: none;
            display: inline-block;
            text-align: center;
        }
        
        .btn-primary {
            background-color: var(--primary-color);
            color: white;
        }
        
        .btn-primary:hover {
            background-color: #0056b3;
        }
        
        .btn-secondary {
            background-color: var(--secondary-color);
            color: white;
        }
        
        .btn-secondary:hover {
            background-color: #545b62;
        }
        
        .btn:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        
        .summary {
            background-color: var(--light-color);
            padding: 20px;
            border-radius: 4px;
            margin-bottom: 20px;
        }
        
        .summary ul {
            list-style: none;
            padding: 0;
        }
        
        .summary li {
            padding: 5px 0;
            border-bottom: 1px solid #ddd;
        }
        
        .summary li:last-child {
            border-bottom: none;
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
        
        @media (max-width: 768px) {
            .form-row {
                flex-direction: column;
                gap: 0;
            }
            
            .radio-group {
                flex-direction: column;
                gap: 10px;
            }
            
            .form-actions {
                flex-direction: column;
                gap: 10px;
            }
            
            .form-actions button {
                width: 100%;
            }
        }
    </style>
</body>
</html>
```

## Заключение

Современные HTML формы включают в себя множество улучшений по сравнению с базовыми элементами. Ключевые улучшения:

1. **Расширенные типы полей** - обеспечивают лучший пользовательский опыт на мобильных устройствах
2. **Атрибуты автозаполнения** - упрощают заполнение форм для пользователей
3. **Встроенные атрибуты валидации** - обеспечивают базовую проверку данных
4. **Элементы datalist и output** - добавляют интерактивности и вычислений
5. **Современные шаблоны форм** - улучшают пользовательский опыт
6. **Расширенные возможности файловых загрузок** - упрощают работу с файлами
7. **Доступность** - обеспечивает использование форм всеми пользователями

Эти элементы и техники позволяют создавать формы, которые не только функциональны, но и обеспечивают отличный пользовательский опыт.

## Следующие темы
- [[HTML/Формы/Валидация]]
- [[HTML/Формы/Доступность]]
- [[HTML/Интеграция с JavaScript]]

## Теги
#html #forms #modern-forms #web-development #frontend #user-experience #input-types #validation #accessibility #file-upload #datalist #output #best-practices #form-design
# HTML Формы: Доступность

Доступность HTML форм - это важный аспект веб-разработки, который обеспечивает возможность использования форм всеми пользователями, включая людей с ограниченными возможностями. Правильная разметка форм делает их удобными для пользователей скринридеров, клавиатурной навигации и других вспомогательных технологий.

## Основы доступности форм

### Правильная разметка форм

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные HTML формы</title>
</head>
<body>
    <main>
        <h1>Регистрация нового пользователя</h1>
        
        <form action="/register" method="post" novalidate>
            <!-- Использование fieldset для группировки связанных полей -->
            <fieldset>
                <legend>Личная информация</legend>
                
                <div class="form-group">
                    <label for="first-name">Имя <span class="required">*</span></label>
                    <input type="text" 
                           id="first-name" 
                           name="first-name" 
                           required 
                           aria-describedby="first-name-help">
                    <div id="first-name-help" class="help-text">
                        Введите ваше имя
                    </div>
                </div>
                
                <div class="form-group">
                    <label for="last-name">Фамилия <span class="required">*</span></label>
                    <input type="text" 
                           id="last-name" 
                           name="last-name" 
                           required>
                </div>
                
                <div class="form-group">
                    <label for="email">Email <span class="required">*</span></label>
                    <input type="email" 
                           id="email" 
                           name="email" 
                           required 
                           aria-describedby="email-error email-help">
                    <div id="email-help" class="help-text">
                        Введите действительный email адрес
                    </div>
                    <div id="email-error" class="error-message" role="alert" aria-live="polite">
                        <!-- Ошибки будут отображаться здесь -->
                    </div>
                </div>
            </fieldset>
            
            <fieldset>
                <legend>Настройки аккаунта</legend>
                
                <div class="form-group">
                    <label for="username">Имя пользователя <span class="required">*</span></label>
                    <input type="text" 
                           id="username" 
                           name="username" 
                           required 
                           minlength="3" 
                           maxlength="20">
                </div>
                
                <div class="form-group">
                    <label for="password">Пароль <span class="required">*</span></label>
                    <input type="password" 
                           id="password" 
                           name="password" 
                           required 
                           minlength="8">
                </div>
                
                <div class="form-group">
                    <label for="confirm-password">Подтверждение пароля <span class="required">*</span></label>
                    <input type="password" 
                           id="confirm-password" 
                           name="confirm-password" 
                           required>
                </div>
            </fieldset>
            
            <fieldset>
                <legend>Дополнительные настройки</legend>
                
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
                               value="accepted" 
                               required 
                               aria-describedby="terms-description">
                        Я согласен с условиями использования
                    </label>
                    <div id="terms-description" class="help-text">
                        Обязательное поле для регистрации
                    </div>
                </div>
            </fieldset>
            
            <div class="form-actions">
                <button type="submit">Зарегистрироваться</button>
                <button type="reset">Очистить форму</button>
            </div>
        </form>
    </main>
    
    <style>
        .form-group {
            margin-bottom: 1rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.25rem;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 0.5rem;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        .help-text {
            font-size: 0.875rem;
            color: #666;
            margin-top: 0.25rem;
        }
        
        .error-message {
            color: #d32f2f;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        
        .required {
            color: #d32f2f;
        }
        
        fieldset {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 1rem;
            margin-bottom: 1rem;
        }
        
        legend {
            font-weight: bold;
            padding: 0 0.5rem;
        }
        
        .form-actions {
            margin-top: 1rem;
        }
        
        button {
            padding: 0.5rem 1rem;
            margin-right: 0.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button[type="submit"] {
            background-color: #1976d2;
            color: white;
        }
        
        button[type="reset"] {
            background-color: #757575;
            color: white;
        }
    </style>
</body>
</html>
```

## Основные принципы доступности форм

### 1. Правильное использование меток (labels)

```html
<!-- ПРАВИЛЬНО - явная метка -->
<label for="email">Email:</label>
<input type="email" id="email" name="email">

<!-- ПРАВИЛЬНО - вложенная метка -->
<label>
    Email:
    <input type="email" name="email">
</label>

<!-- НЕПРАВИЛЬНО - без метки -->
<input type="email" name="email" placeholder="Email">
```

### 2. Использование aria-describedby для дополнительной информации

```html
<label for="phone">Телефон:</label>
<input type="tel" 
       id="phone" 
       name="phone" 
       aria-describedby="phone-format phone-required">
       
<div id="phone-format">Формат: +7 (XXX) XXX-XX-XX</div>
<div id="phone-required">Обязательное поле</div>
```

### 3. Группировка связанных элементов

```html
<!-- Использование fieldset и legend -->
<fieldset>
    <legend>Выберите ваш пол:</legend>
    <label>
        <input type="radio" name="gender" value="male" id="male">
        Мужской
    </label>
    <label>
        <input type="radio" name="gender" value="female" id="female">
        Женский
    </label>
    <label>
        <input type="radio" name="gender" value="other" id="other">
        Другое
    </label>
</fieldset>

<!-- Группировка чекбоксов -->
<fieldset>
    <legend>Интересы:</legend>
    <label>
        <input type="checkbox" name="interests" value="tech">
        Технологии
    </label>
    <label>
        <input type="checkbox" name="interests" value="sports">
        Спорт
    </label>
    <label>
        <input type="checkbox" name="interests" value="music">
        Музыка
    </label>
</fieldset>
```

## Типы полей и атрибуты доступности

### Текстовые поля

```html
<!-- Email поле -->
<label for="email">Email:</label>
<input type="email" 
       id="email" 
       name="email" 
       required 
       autocomplete="email"
       aria-describedby="email-help">

<!-- Телефонное поле -->
<label for="phone">Телефон:</label>
<input type="tel" 
       id="phone" 
       name="phone" 
       autocomplete="tel"
       aria-describedby="phone-format">

<!-- Поле поиска -->
<label for="search">Поиск:</label>
<input type="search" 
       id="search" 
       name="search" 
       aria-label="Поле поиска"
       autocomplete="off">
```

### Числовые поля

```html
<!-- Поле возраста -->
<label for="age">Возраст:</label>
<input type="number" 
       id="age" 
       name="age" 
       min="1" 
       max="120"
       aria-describedby="age-range">

<div id="age-range">Введите возраст от 1 до 120 лет</div>

<!-- Поле рейтинга -->
<label for="rating">Рейтинг (1-5):</label>
<input type="range" 
       id="rating" 
       name="rating" 
       min="1" 
       max="5" 
       value="3"
       aria-valuetext="Средний рейтинг">
```

### Дата и время

```html
<!-- Поле даты -->
<label for="birthdate">Дата рождения:</label>
<input type="date" 
       id="birthdate" 
       name="birthdate"
       aria-describedby="date-format">

<div id="date-format">Выберите дату рождения</div>

<!-- Поле времени -->
<label for="appointment-time">Время встречи:</label>
<input type="time" 
       id="appointment-time" 
       name="appointment-time">
```

## Валидация и сообщения об ошибках

### Встроенные HTML валидации

```html
<form id="validation-form">
    <div class="form-group">
        <label for="username">Имя пользователя:</label>
        <input type="text" 
               id="username" 
               name="username" 
               required 
               minlength="3" 
               maxlength="20"
               pattern="[A-Za-z0-9_]+"
               aria-describedby="username-help username-error">
        <div id="username-help" class="help-text">
            Только буквы, цифры и подчеркивание. 3-20 символов.
        </div>
        <div id="username-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>
    
    <div class="form-group">
        <label for="email">Email:</label>
        <input type="email" 
               id="email" 
               name="email" 
               required
               aria-describedby="email-error">
        <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>
    
    <button type="submit">Отправить</button>
</form>

<script>
document.getElementById('validation-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    // Сброс ошибок
    document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
    document.querySelectorAll('input').forEach(el => el.classList.remove('error'));
    
    let isValid = true;
    
    // Валидация имени пользователя
    const username = document.getElementById('username');
    if (!username.value.trim()) {
        showError(username, 'Имя пользователя обязательно');
        isValid = false;
    } else if (username.value.length < 3) {
        showError(username, 'Имя пользователя должно быть не менее 3 символов');
        isValid = false;
    } else if (username.value.length > 20) {
        showError(username, 'Имя пользователя должно быть не более 20 символов');
        isValid = false;
    } else if (!/^[A-Za-z0-9_]+$/.test(username.value)) {
        showError(username, 'Имя пользователя может содержать только буквы, цифры и подчеркивание');
        isValid = false;
    }
    
    // Валидация email
    const email = document.getElementById('email');
    if (!email.value.trim()) {
        showError(email, 'Email обязателен');
        isValid = false;
    } else if (!isValidEmail(email.value)) {
        showError(email, 'Введите действительный email адрес');
        isValid = false;
    }
    
    if (isValid) {
        alert('Форма успешно валидирована!');
    }
});

function showError(input, message) {
    input.classList.add('error');
    const errorElement = document.getElementById(input.id + '-error');
    errorElement.textContent = message;
    
    // Фокус на поле с ошибкой
    input.focus();
}

function isValidEmail(email) {
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return re.test(email);
}

// Добавление стилей для ошибок
const style = document.createElement('style');
style.textContent = `
    input.error {
        border-color: #d32f2f;
        background-color: #ffebee;
    }
    
    .error-message {
        color: #d32f2f;
        font-size: 0.875rem;
        margin-top: 0.25rem;
    }
    
    .help-text {
        color: #666;
        font-size: 0.875rem;
        margin-top: 0.25rem;
    }
`;
document.head.appendChild(style);
</script>

<style>
    input.error {
        border-color: #d32f2f;
        background-color: #ffebee;
    }
    
    .error-message {
        color: #d32f2f;
        font-size: 0.875rem;
        margin-top: 0.25rem;
    }
    
    .help-text {
        color: #666;
        font-size: 0.875rem;
        margin-top: 0.25rem;
    }
</style>
```

### ARIA атрибуты для валидации

```html
<form>
    <div class="form-group">
        <label for="password">Пароль:</label>
        <input type="password" 
               id="password" 
               name="password" 
               required
               aria-describedby="password-requirements password-error"
               aria-invalid="false">
        <div id="password-requirements" class="help-text">
            Пароль должен содержать не менее 8 символов, включая заглавную букву и цифру
        </div>
        <div id="password-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>
    
    <div class="form-group">
        <label for="confirm-password">Подтверждение пароля:</label>
        <input type="password" 
               id="confirm-password" 
               name="confirm-password" 
               required
               aria-describedby="confirm-password-error"
               aria-invalid="false">
        <div id="confirm-password-error" class="error-message" role="alert" aria-live="polite"></div>
    </div>
    
    <button type="submit" aria-describedby="submit-help">Отправить</button>
    <div id="submit-help" class="help-text">Нажмите для отправки формы</div>
</form>
```

## Доступность сложных компонентов

### Выпадающие списки

```html
<label for="country">Страна:</label>
<select id="country" 
        name="country" 
        required
        aria-describedby="country-help">
    <option value="">Выберите страну</option>
    <option value="ru">Россия</option>
    <option value="us">США</option>
    <option value="de">Германия</option>
    <option value="fr">Франция</option>
</select>
<div id="country-help" class="help-text">
    Выберите вашу страну проживания
</div>

<!-- Группировка опций -->
<label for="car">Выберите автомобиль:</label>
<select id="car" name="car">
    <optgroup label="Европейские">
        <option value="bmw">BMW</option>
        <option value="mercedes">Mercedes</option>
        <option value="audi">Audi</option>
    </optgroup>
    <optgroup label="Японские">
        <option value="toyota">Toyota</option>
        <option value="honda">Honda</option>
        <option value="nissan">Nissan</option>
    </optgroup>
</select>
```

### Переключатели (Switches)

```html
<div class="switch-container">
    <label for="notifications" class="switch-label">
        Уведомления
    </label>
    <div class="switch" role="switch" 
         aria-checked="false" 
         tabindex="0" 
         onclick="toggleSwitch(this)"
         onkeydown="handleSwitchKeydown(event)">
        <div class="switch-thumb"></div>
    </div>
    <span class="switch-status" aria-live="polite">Выключено</span>
</div>

<style>
    .switch-container {
        display: flex;
        align-items: center;
        gap: 10px;
        margin: 10px 0;
    }
    
    .switch {
        width: 50px;
        height: 24px;
        background-color: #ccc;
        border-radius: 12px;
        position: relative;
        cursor: pointer;
        transition: background-color 0.3s ease;
    }
    
    .switch[aria-checked="true"] {
        background-color: #1976d2;
    }
    
    .switch-thumb {
        width: 20px;
        height: 20px;
        background-color: white;
        border-radius: 50%;
        position: absolute;
        top: 2px;
        left: 2px;
        transition: transform 0.3s ease;
    }
    
    .switch[aria-checked="true"] .switch-thumb {
        transform: translateX(26px);
    }
    
    .switch:focus {
        outline: 2px solid #1976d2;
        outline-offset: 2px;
    }
</style>

<script>
function toggleSwitch(element) {
    const isCurrentlyChecked = element.getAttribute('aria-checked') === 'true';
    const newCheckedState = !isCurrentlyChecked;
    
    element.setAttribute('aria-checked', newCheckedState);
    
    // Обновить визуальное состояние
    element.classList.toggle('checked', newCheckedState);
    
    // Обновить статус
    const statusElement = element.nextElementSibling;
    statusElement.textContent = newCheckedState ? 'Включено' : 'Выключено';
}

function handleSwitchKeydown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        toggleSwitch(event.target);
    }
}
</script>
```

### Автодополнение и поисковые подсказки

```html
<div class="autocomplete-container">
    <label for="search-input">Поиск:</label>
    <input type="text" 
           id="search-input" 
           name="search" 
           role="combobox"
           aria-expanded="false"
           aria-owns="suggestions-list"
           aria-autocomplete="list"
           aria-describedby="search-help">
    <div id="search-help" class="help-text">
        Начните вводить текст для поиска
    </div>
    
    <ul id="suggestions-list" 
        role="listbox" 
        aria-label="Поисковые подсказки"
        style="display: none; border: 1px solid #ccc; background: white; margin: 0; padding: 0;">
        <!-- Подсказки будут добавляться сюда -->
    </ul>
</div>

<script>
const searchInput = document.getElementById('search-input');
const suggestionsList = document.getElementById('suggestions-list');
const searchContainer = document.querySelector('.autocomplete-container');

// Пример данных для автодополнения
const suggestions = [
    'HTML',
    'CSS',
    'JavaScript',
    'React',
    'Vue.js',
    'Angular',
    'Node.js',
    'Python',
    'Java',
    'C++'
];

searchInput.addEventListener('input', function() {
    const query = this.value.toLowerCase();
    
    // Фильтрация подсказок
    const filteredSuggestions = query ? 
        suggestions.filter(item => item.toLowerCase().includes(query)) : 
        [];
    
    // Обновление списка подсказок
    updateSuggestionsList(filteredSuggestions);
});

function updateSuggestionsList(items) {
    suggestionsList.innerHTML = '';
    
    if (items.length > 0) {
        items.forEach((item, index) => {
            const li = document.createElement('li');
            li.textContent = item;
            li.setAttribute('role', 'option');
            li.setAttribute('id', `suggestion-${index}`);
            li.setAttribute('tabindex', '-1');
            
            li.addEventListener('click', () => {
                searchInput.value = item;
                suggestionsList.style.display = 'none';
                searchInput.setAttribute('aria-expanded', 'false');
            });
            
            suggestionsList.appendChild(li);
        });
        
        suggestionsList.style.display = 'block';
        searchInput.setAttribute('aria-expanded', 'true');
    } else {
        suggestionsList.style.display = 'none';
        searchInput.setAttribute('aria-expanded', 'false');
    }
}

// Обработка клавиатурной навигации
let currentFocus = -1;

searchInput.addEventListener('keydown', function(e) {
    const items = suggestionsList.querySelectorAll('li');
    
    if (e.key === 'ArrowDown') {
        e.preventDefault();
        currentFocus = (currentFocus + 1) % items.length;
        setActive(items, currentFocus);
    } else if (e.key === 'ArrowUp') {
        e.preventDefault();
        currentFocus = currentFocus > 0 ? currentFocus - 1 : items.length - 1;
        setActive(items, currentFocus);
    } else if (e.key === 'Enter') {
        e.preventDefault();
        if (currentFocus > -1 && items[currentFocus]) {
            searchInput.value = items[currentFocus].textContent;
            suggestionsList.style.display = 'none';
            searchInput.setAttribute('aria-expanded', 'false');
            currentFocus = -1;
        }
    } else if (e.key === 'Escape') {
        suggestionsList.style.display = 'none';
        searchInput.setAttribute('aria-expanded', 'false');
        currentFocus = -1;
    }
});

function setActive(items, index) {
    items.forEach((item, i) => {
        item.classList.toggle('active', i === index);
    });
    
    if (items[index]) {
        items[index].scrollIntoView({ block: 'nearest' });
    }
}
</script>

<style>
    .autocomplete-container {
        position: relative;
        max-width: 400px;
    }
    
    #suggestions-list {
        position: absolute;
        top: 100%;
        left: 0;
        right: 0;
        z-index: 1000;
        list-style: none;
    }
    
    #suggestions-list li {
        padding: 10px;
        cursor: pointer;
        border-bottom: 1px solid #eee;
    }
    
    #suggestions-list li:hover,
    #suggestions-list li.active {
        background-color: #f0f0f0;
    }
</style>
```

## Клавиатурная навигация

### Порядок табуляции

```html
<form>
    <input type="text" name="field1" tabindex="1">
    <input type="text" name="field2" tabindex="2">
    <button type="button" tabindex="3">Кнопка 1</button>
    <input type="text" name="field3" tabindex="4">
    <button type="submit" tabindex="5">Отправить</button>
</form>
```

### Пропуск навигации для форм

```html
<a class="skip-link" href="#main-form">Перейти к основной форме</a>

<!-- Контент страницы -->

<form id="main-form">
    <h2>Основная форма</h2>
    <!-- Поля формы -->
</form>

<style>
    .skip-link {
        position: absolute;
        top: -40px;
        left: 6px;
        color: #fff;
        background: #000;
        padding: 8px;
        text-decoration: none;
        z-index: 1000;
    }
    
    .skip-link:focus {
        top: 6px;
    }
</style>
```

## Современные паттерны доступности

### Динамические формы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Динамическая доступная форма</title>
</head>
<body>
    <main>
        <h1>Динамическая форма</h1>
        
        <form id="dynamic-form">
            <div class="form-section">
                <h2>Основная информация</h2>
                
                <div class="form-group">
                    <label for="name">Имя:</label>
                    <input type="text" id="name" name="name" required>
                </div>
                
                <div class="form-group">
                    <label for="contact-method">Способ связи:</label>
                    <select id="contact-method" name="contact-method" required>
                        <option value="">Выберите способ</option>
                        <option value="email">Email</option>
                        <option value="phone">Телефон</option>
                        <option value="both">Оба</option>
                    </select>
                </div>
            </div>
            
            <!-- Динамические поля будут добавляться сюда -->
            <div id="dynamic-fields" aria-live="polite"></div>
            
            <div class="form-actions">
                <button type="button" onclick="addField()">Добавить поле</button>
                <button type="submit">Отправить</button>
            </div>
        </form>
    </main>
    
    <script>
        const contactMethod = document.getElementById('contact-method');
        const dynamicFields = document.getElementById('dynamic-fields');
        let fieldCount = 0;
        
        contactMethod.addEventListener('change', updateDynamicFields);
        
        function updateDynamicFields() {
            const selectedMethod = contactMethod.value;
            dynamicFields.innerHTML = '';
            
            if (selectedMethod === 'email' || selectedMethod === 'both') {
                addEmailField();
            }
            
            if (selectedMethod === 'phone' || selectedMethod === 'both') {
                addPhoneField();
            }
        }
        
        function addEmailField() {
            fieldCount++;
            const fieldId = `email-${fieldCount}`;
            
            const fieldDiv = document.createElement('div');
            fieldDiv.className = 'form-group';
            fieldDiv.innerHTML = `
                <label for="${fieldId}">Email ${fieldCount}:</label>
                <input type="email" 
                       id="${fieldId}" 
                       name="email-${fieldCount}" 
                       required
                       aria-describedby="${fieldId}-help">
                <div id="${fieldId}-help" class="help-text">Введите действительный email</div>
            `;
            
            dynamicFields.appendChild(fieldDiv);
        }
        
        function addPhoneField() {
            fieldCount++;
            const fieldId = `phone-${fieldCount}`;
            
            const fieldDiv = document.createElement('div');
            fieldDiv.className = 'form-group';
            fieldDiv.innerHTML = `
                <label for="${fieldId}">Телефон ${fieldCount}:</label>
                <input type="tel" 
                       id="${fieldId}" 
                       name="phone-${fieldCount}" 
                       required
                       aria-describedby="${fieldId}-help">
                <div id="${fieldId}-help" class="help-text">Введите номер телефона</div>
            `;
            
            dynamicFields.appendChild(fieldDiv);
        }
        
        function addField() {
            fieldCount++;
            const fieldId = `dynamic-${fieldCount}`;
            
            const fieldDiv = document.createElement('div');
            fieldDiv.className = 'form-group';
            fieldDiv.innerHTML = `
                <label for="${fieldId}">Дополнительное поле ${fieldCount}:</label>
                <input type="text" 
                       id="${fieldId}" 
                       name="dynamic-${fieldCount}"
                       aria-describedby="${fieldId}-help">
                <div id="${fieldId}-help" class="help-text">Дополнительная информация</div>
                <button type="button" onclick="this.parentElement.remove()" 
                        aria-label="Удалить поле ${fieldCount}">
                    Удалить
                </button>
            `;
            
            dynamicFields.appendChild(fieldDiv);
        }
        
        document.getElementById('dynamic-form').addEventListener('submit', function(e) {
            e.preventDefault();
            alert('Форма отправлена!');
        });
    </script>
    
    <style>
        .form-group {
            margin-bottom: 1rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.25rem;
            font-weight: bold;
        }
        
        input, select {
            width: 100%;
            padding: 0.5rem;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        .help-text {
            font-size: 0.875rem;
            color: #666;
            margin-top: 0.25rem;
        }
        
        .form-actions {
            margin-top: 1rem;
        }
        
        button {
            padding: 0.5rem 1rem;
            margin-right: 0.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            background-color: #1976d2;
            color: white;
        }
        
        button[aria-label*="Удалить"] {
            background-color: #d32f2f;
            font-size: 0.8rem;
            padding: 0.25rem 0.5rem;
        }
    </style>
</body>
</html>
```

## Проверка доступности форм

### Инструменты для проверки

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Проверка доступности форм</title>
    <script src="https://cdn.jsdelivr.net/npm/axe-core@4.7.2/axe.min.js"></script>
</head>
<body>
    <main>
        <h1>Инструменты проверки доступности</h1>
        
        <form id="test-form">
            <div class="form-group">
                <label for="name">Имя:</label>
                <input type="text" id="name" name="name" required>
            </div>
            
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
            </div>
            
            <button type="submit">Отправить</button>
        </form>
    </main>
    
    <button onclick="runAccessibilityCheck()">Проверить доступность</button>
    <div id="a11y-results" aria-live="polite"></div>
    
    <script>
        async function runAccessibilityCheck() {
            const results = await axe.run('#test-form', {
                runOnly: {
                    type: 'tag',
                    values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa']
                }
            });
            
            const resultsDiv = document.getElementById('a11y-results');
            
            if (results.violations.length === 0) {
                resultsDiv.innerHTML = '<div style="color: green;">✅ Форма соответствует стандартам доступности!</div>';
            } else {
                resultsDiv.innerHTML = '<div style="color: red;"><strong>❌ Найдены нарушения доступности:</strong></div>';
                
                const violationsList = document.createElement('ul');
                
                results.violations.forEach(violation => {
                    const li = document.createElement('li');
                    li.innerHTML = `
                        <strong>${violation.help}:</strong><br>
                        <span style="font-size: 0.9em; color: #666;">
                            ${violation.nodes.map(node => node.html).join(', ')}
                        </span>
                    `;
                    violationsList.appendChild(li);
                });
                
                resultsDiv.appendChild(violationsList);
            }
        }
    </script>
</body>
</html>
```

## Заключение

Доступность HTML форм - это комплексный подход, который включает:

1. **Правильную семантику** - использование соответствующих тегов и атрибутов
2. **Явные метки** - для всех интерактивных элементов
3. **Группировку** - связанных элементов с помощью fieldset и legend
4. **ARIA атрибуты** - для дополнительной информации и состояний
5. **Клавиатурную навигацию** - обеспечение логичного порядка табуляции
6. **Ясные сообщения об ошибках** - с правильной разметкой
7. **Адаптивность** - для пользователей с различными возможностями

Создание доступных форм не только помогает пользователям с ограниченными возможностями, но и улучшает общий пользовательский опыт для всех.

## Современные подходы к доступности форм

### Расширенное использование ARIA для форм
```html
<form action="/register" method="post">
    <div class="form-group">
        <label for="adv-name">Имя:</label>
        <input type="text" 
               id="adv-name" 
               name="name" 
               required
               aria-describedby="name-help name-error"
               aria-invalid="false"
               aria-errormessage="name-error">
        <div id="name-help" class="help-text">Введите ваше полное имя</div>
        <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <div class="form-group">
        <label for="adv-email">Email:</label>
        <input type="email" 
               id="adv-email" 
               name="email" 
               required
               aria-describedby="email-help email-error"
               aria-invalid="false"
               aria-errormessage="email-error">
        <div id="email-help" class="help-text">Адрес для подтверждения регистрации</div>
        <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <!-- Группировка связанных полей -->
    <fieldset aria-describedby="dob-help">
        <legend>Дата рождения</legend>
        <div id="dob-help" class="help-text">Введите дату в формате ДД.ММ.ГГГГ</div>
        <div class="date-group">
            <label for="day">День:</label>
            <input type="number" 
                   id="day" 
                   name="day" 
                   min="1" 
                   max="31"
                   aria-label="День рождения">
            
            <label for="month">Месяц:</label>
            <input type="number" 
                   id="month" 
                   name="month" 
                   min="1" 
                   max="12"
                   aria-label="Месяц рождения">
            
            <label for="year">Год:</label>
            <input type="number" 
                   id="year" 
                   name="year" 
                   min="1900" 
                   max="2023"
                   aria-label="Год рождения">
        </div>
    </fieldset>

    <!-- Группировка чекбоксов -->
    <fieldset role="group" aria-labelledby="interests-label">
        <legend id="interests-label">Интересы:</legend>
        <div class="checkbox-group">
            <label>
                <input type="checkbox" 
                       name="interests" 
                       value="tech"
                       aria-describedby="tech-desc">
                Технологии
            </label>
            <div id="tech-desc" class="sr-only">Выберите, если интересуетесь технологиями</div>
            
            <label>
                <input type="checkbox" 
                       name="interests" 
                       value="sports"
                       aria-describedby="sports-desc">
                Спорт
            </label>
            <div id="sports-desc" class="sr-only">Выберите, если интересуетесь спортом</div>
            
            <label>
                <input type="checkbox" 
                       name="interests" 
                       value="music"
                       aria-describedby="music-desc">
                Музыка
            </label>
            <div id="music-desc" class="sr-only">Выберите, если интересуетесь музыкой</div>
        </div>
    </fieldset>

    <button type="submit" aria-describedby="submit-help">Зарегистрироваться</button>
    <div id="submit-help" class="sr-only">Нажмите для отправки формы регистрации</div>
</form>
```

### Доступная валидация форм
```html
<form id="accessible-validation-form">
    <div class="form-group">
        <label for="accv-email">Email:</label>
        <input type="email" 
               id="accv-email" 
               name="email" 
               required
               aria-describedby="accv-email-help accv-email-error"
               aria-invalid="false">
        <div id="accv-email-help" class="help-text">Введите действительный email адрес</div>
        <div id="accv-email-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <div class="form-group">
        <label for="accv-password">Пароль:</label>
        <input type="password" 
               id="accv-password" 
               name="password" 
               required 
               minlength="8"
               aria-describedby="password-requirements accv-password-error"
               aria-invalid="false">
        <ul id="password-requirements" class="requirements-list">
            <li id="req-len">Не менее 8 символов</li>
            <li id="req-upper">С заглавной буквой</li>
            <li id="req-lower">Со строчной буквой</li>
            <li id="req-num">С цифрой</li>
        </ul>
        <div id="accv-password-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <div class="form-group">
        <label for="accv-confirm">Подтверждение пароля:</label>
        <input type="password" 
               id="accv-confirm" 
               name="confirm_password" 
               required
               aria-describedby="accv-confirm-error"
               aria-invalid="false">
        <div id="accv-confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>

    <button type="submit">Зарегистрироваться</button>
</form>

<script>
class AccessibleFormValidator {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.setupAccessibleValidation();
    }

    setupAccessibleValidation() {
        // Валидация при потере фокуса
        this.form.addEventListener('blur', (e) => {
            if (e.target.matches('input[required]')) {
                this.validateFieldAccessible(e.target);
            }
        }, true);

        // Валидация пароля в реальном времени
        document.getElementById('accv-password').addEventListener('input', () => {
            this.updatePasswordRequirements();
        });

        // Валидация подтверждения пароля
        document.getElementById('accv-confirm').addEventListener('input', () => {
            this.validatePasswordMatch();
        });

        // Отправка формы
        this.form.addEventListener('submit', (e) => {
            e.preventDefault();
            this.validateFormAccessible();
        });
    }

    validateFieldAccessible(field) {
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
            
            const errorMessage = this.getAccessibleErrorMessage(field);
            errorElement.textContent = errorMessage;
            
            // Сообщение для скринридера о фокусе на ошибке
            this.announceError(errorMessage);
            return false;
        }
    }

    updatePasswordRequirements() {
        const password = document.getElementById('accv-password').value;
        const requirements = {
            length: password.length >= 8,
            upper: /[A-Z]/.test(password),
            lower: /[a-z]/.test(password),
            number: /[0-9]/.test(password)
        };

        document.getElementById('req-len').className = requirements.length ? 'met' : '';
        document.getElementById('req-upper').className = requirements.upper ? 'met' : '';
        document.getElementById('req-lower').className = requirements.lower ? 'met' : '';
        document.getElementById('req-num').className = requirements.number ? 'met' : '';
    }

    validatePasswordMatch() {
        const password = document.getElementById('accv-password').value;
        const confirmPassword = document.getElementById('accv-confirm').value;
        const errorElement = document.getElementById('accv-confirm-error');

        if (confirmPassword && password !== confirmPassword) {
            errorElement.textContent = 'Пароли не совпадают';
            document.getElementById('accv-confirm').classList.add('invalid');
            document.getElementById('accv-confirm').setAttribute('aria-invalid', 'true');
            this.announceError('Пароли не совпадают');
            return false;
        } else {
            errorElement.textContent = '';
            document.getElementById('accv-confirm').classList.remove('invalid');
            document.getElementById('accv-confirm').setAttribute('aria-invalid', 'false');
            return true;
        }
    }

    validateFormAccessible() {
        let isValid = true;
        const fields = this.form.querySelectorAll('input[required]');

        fields.forEach(field => {
            if (!this.validateFieldAccessible(field)) {
                isValid = false;
            }
        });

        if (this.validatePasswordMatch() && this.validatePasswordRequirements()) {
            isValid = isValid && true;
        } else {
            isValid = false;
        }

        if (isValid) {
            // Сообщение об успешной валидации
            this.announceSuccess('Форма успешно валидирована. Вы можете отправить её.');
        } else {
            // Сообщение о наличии ошибок
            const errorCount = this.form.querySelectorAll('.error-message:not(:empty)').length;
            this.announceError(`Обнаружено ${errorCount} ошибок валидации. Пожалуйста, исправьте их.`);
        }
    }

    validatePasswordRequirements() {
        const password = document.getElementById('accv-password').value;
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
            return `Введите не менее ${field.minLength} символов`;
        }
        return 'Неверный формат данных';
    }

    announceError(message) {
        this.announce(message, 'assertive');
    }

    announceSuccess(message) {
        this.announce(message, 'polite');
    }

    announce(message, priority) {
        const announcement = document.createElement('div');
        announcement.setAttribute('aria-live', priority);
        announcement.setAttribute('aria-atomic', 'true');
        announcement.className = 'sr-only';
        announcement.textContent = message;

        document.body.appendChild(announcement);

        // Удаляем сообщение после оглашения
        setTimeout(() => {
            if (document.body.contains(announcement)) {
                document.body.removeChild(announcement);
            }
        }, 1000);
    }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
    new AccessibleFormValidator('accessible-validation-form');
});
</script>
```

## Доступные сложные формы

### Форма с автодополнением и подсказками
```html
<div class="autocomplete-container">
    <label for="city-input">Город:</label>
    <input type="text" 
           id="city-input" 
           name="city" 
           aria-autocomplete="list"
           aria-controls="suggestions-list"
           aria-expanded="false"
           aria-describedby="city-help"
           role="combobox"
           autocomplete="off">
    <div id="city-help" class="help-text">Начните вводить название города для поиска</div>
    
    <ul id="suggestions-list" 
        role="listbox" 
        aria-label="Результаты поиска городов"
        style="display: none; border: 1px solid #ccc; background: white; margin: 0; padding: 0;">
        <!-- Подсказки будут добавляться сюда -->
    </ul>
</div>

<script>
class AccessibleAutocomplete {
    constructor(inputId) {
        this.input = document.getElementById(inputId);
        this.suggestionsList = document.getElementById('suggestions-list');
        this.isExpanded = false;
        
        this.cities = [
            'Москва', 'Санкт-Петербург', 'Новосибирск', 'Екатеринбург', 
            'Казань', 'Нижний Новгород', 'Челябинск', 'Самара'
        ];
        
        this.setupEvents();
    }

    setupEvents() {
        this.input.addEventListener('input', (e) => {
            this.handleInput(e.target.value);
        });

        this.input.addEventListener('keydown', (e) => {
            this.handleKeydown(e);
        });

        this.input.addEventListener('blur', () => {
            // Задержка для возможности клика по подсказке
            setTimeout(() => {
                this.hideSuggestions();
            }, 200);
        });
    }

    handleInput(query) {
        if (query.length < 2) {
            this.hideSuggestions();
            return;
        }

        const matches = this.cities.filter(city => 
            city.toLowerCase().includes(query.toLowerCase())
        );

        this.showSuggestions(matches);
    }

    showSuggestions(matches) {
        this.suggestionsList.innerHTML = '';

        if (matches.length === 0) {
            this.hideSuggestions();
            return;
        }

        matches.forEach((city, index) => {
            const li = document.createElement('li');
            li.textContent = city;
            li.setAttribute('role', 'option');
            li.setAttribute('id', `city-option-${index}`);
            li.setAttribute('tabindex', '-1');
            
            li.addEventListener('click', () => {
                this.selectOption(city);
            });

            li.addEventListener('mousedown', (e) => {
                // Предотвращаем потерю фокуса с input при клике на опцию
                e.preventDefault();
            });

            this.suggestionsList.appendChild(li);
        });

        this.suggestionsList.style.display = 'block';
        this.input.setAttribute('aria-expanded', 'true');
        this.isExpanded = true;
    }

    hideSuggestions() {
        this.suggestionsList.style.display = 'none';
        this.input.setAttribute('aria-expanded', 'false');
        this.isExpanded = false;
    }

    selectOption(city) {
        this.input.value = city;
        this.hideSuggestions();
        this.input.focus();
        
        // Сообщение для скринридера
        this.announce(`Выбран город ${city}`);
    }

    handleKeydown(e) {
        const options = this.suggestionsList.querySelectorAll('li');
        if (options.length === 0) return;

        let currentFocus = -1;
        const currentOption = document.activeElement;

        if (currentOption && currentOption.parentElement === this.suggestionsList) {
            currentFocus = Array.from(options).indexOf(currentOption);
        }

        switch(e.key) {
            case 'ArrowDown':
                e.preventDefault();
                currentFocus = (currentFocus + 1) % options.length;
                options[currentFocus].focus();
                break;
                
            case 'ArrowUp':
                e.preventDefault();
                currentFocus = currentFocus > 0 ? currentFocus - 1 : options.length - 1;
                options[currentFocus].focus();
                break;
                
            case 'Enter':
                e.preventDefault();
                if (currentFocus >= 0) {
                    this.selectOption(options[currentFocus].textContent);
                }
                break;
                
            case 'Escape':
                this.hideSuggestions();
                this.input.focus();
                break;
        }
    }

    announce(message) {
        const announcement = document.createElement('div');
        announcement.setAttribute('aria-live', 'polite');
        announcement.setAttribute('aria-atomic', 'true');
        announcement.className = 'sr-only';
        announcement.textContent = message;

        document.body.appendChild(announcement);

        setTimeout(() => {
            if (document.body.contains(announcement)) {
                document.body.removeChild(announcement);
            }
        }, 1000);
    }
}

// Инициализация
document.addEventListener('DOMContentLoaded', () => {
    new AccessibleAutocomplete('city-input');
});
</script>
```

### Доступная многошаговая форма
```html
<form id="accessible-multi-step-form">
    <!-- Индикатор прогресса -->
    <div class="progress-container" role="progressbar" aria-valuenow="1" aria-valuemin="1" aria-valuemax="3" aria-valuetext="Шаг 1 из 3">
        <div class="progress-bar">
            <div class="progress" style="width: 33%;"></div>
        </div>
        <div class="progress-text">Шаг 1 из 3</div>
    </div>

    <!-- Шаг 1: Контактная информация -->
    <div class="form-step active" id="step1" aria-label="Шаг 1: Контактная информация" role="region">
        <h2>Контактная информация</h2>
        
        <div class="form-group">
            <label for="step1-email">Email:</label>
            <input type="email" 
                   id="step1-email" 
                   name="email" 
                   required
                   aria-describedby="step1-email-error step1-email-help"
                   aria-invalid="false">
            <div id="step1-email-help" class="help-text">Для подтверждения регистрации</div>
            <div id="step1-email-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="step1-phone">Телефон:</label>
            <input type="tel" 
                   id="step1-phone" 
                   name="phone" 
                   required
                   aria-describedby="step1-phone-error step1-phone-help"
                   aria-invalid="false">
            <div id="step1-phone-help" class="help-text">Формат: +7 (XXX) XXX-XX-XX</div>
            <div id="step1-phone-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <button type="button" onclick="nextStep(1)" aria-describedby="next1-desc">Далее</button>
        <div id="next1-desc" class="sr-only">Перейти к следующему шагу формы</div>
    </div>

    <!-- Шаг 2: Личная информация -->
    <div class="form-step" id="step2" aria-label="Шаг 2: Личная информация" role="region" hidden>
        <h2>Личная информация</h2>
        
        <div class="form-group">
            <label for="step2-name">Имя:</label>
            <input type="text" 
                   id="step2-name" 
                   name="name" 
                   required
                   aria-describedby="step2-name-error"
                   aria-invalid="false">
            <div id="step2-name-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="step2-birthdate">Дата рождения:</label>
            <input type="date" 
                   id="step2-birthdate" 
                   name="birthdate" 
                   max="2010-01-01"
                   aria-describedby="step2-birthdate-error"
                   aria-invalid="false">
            <div id="step2-birthdate-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="step2-gender">Пол:</label>
            <select id="step2-gender" name="gender" aria-describedby="step2-gender-error" aria-invalid="false">
                <option value="">Выберите пол</option>
                <option value="male">Мужской</option>
                <option value="female">Женский</option>
                <option value="other">Другой</option>
            </select>
            <div id="step2-gender-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <button type="button" onclick="prevStep(2)" aria-describedby="prev2-desc">Назад</button>
        <button type="button" onclick="nextStep(2)" aria-describedby="next2-desc">Далее</button>
        <div id="prev2-desc" class="sr-only">Вернуться к предыдущему шагу</div>
        <div id="next2-desc" class="sr-only">Перейти к следующему шагу</div>
    </div>

    <!-- Шаг 3: Подтверждение -->
    <div class="form-step" id="step3" aria-label="Шаг 3: Подтверждение" role="region" hidden>
        <h2>Подтверждение</h2>
        
        <div id="step3-summary" class="summary" role="region" aria-labelledby="summary-title">
            <h3 id="summary-title">Проверьте вашу информацию:</h3>
            <div id="summary-content"></div>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox" 
                       name="newsletter" 
                       value="yes"
                       aria-describedby="newsletter-desc">
                Подписаться на рассылку
            </label>
            <div id="newsletter-desc" class="sr-only">Получать новости и обновления по email</div>
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
        
        <button type="button" onclick="prevStep(3)" aria-describedby="prev3-desc">Назад</button>
        <button type="submit" aria-describedby="submit-desc">Зарегистрироваться</button>
        <div id="prev3-desc" class="sr-only">Вернуться к предыдущему шагу</div>
        <div id="submit-desc" class="sr-only">Отправить форму регистрации</div>
    </div>
</form>

<script>
let currentStep = 1;
const totalSteps = 3;

class AccessibleMultiStepForm {
    constructor(formId) {
        this.form = document.getElementById(formId);
        this.currentStep = 1;
        this.setupForm();
    }

    setupForm() {
        // Добавляем горячие клавиши
        this.form.addEventListener('keydown', (e) => {
            if (e.altKey) {
                switch(e.key) {
                    case 'ArrowLeft':
                        e.preventDefault();
                        if (this.currentStep > 1) prevStep(this.currentStep);
                        break;
                    case 'ArrowRight':
                        e.preventDefault();
                        if (this.currentStep < totalSteps) nextStep(this.currentStep);
                        break;
                }
            }
        });

        // Отправка формы
        this.form.addEventListener('submit', (e) => {
            e.preventDefault();
            this.submitForm();
        });
    }

    updateProgress() {
        const progressPercent = (this.currentStep / totalSteps) * 100;
        const progressBar = this.form.querySelector('.progress');
        const progressText = this.form.querySelector('.progress-text');
        
        if (progressBar) {
            progressBar.style.width = `${progressPercent}%`;
        }
        
        if (progressText) {
            progressText.textContent = `Шаг ${this.currentStep} из ${totalSteps}`;
        }
        
        // Обновляем ARIA атрибуты прогресса
        this.form.querySelector('.progress-container').setAttribute('aria-valuenow', this.currentStep);
        this.form.querySelector('.progress-container').setAttribute('aria-valuetext', `Шаг ${this.currentStep} из ${totalSteps}`);
    }

    async submitForm() {
        // Валидация перед отправкой
        if (await this.validateCurrentStep()) {
            // Симуляция отправки
            alert('Форма успешно отправлена!');
        }
    }

    async validateCurrentStep() {
        const stepElement = document.getElementById(`step${this.currentStep}`);
        const requiredFields = stepElement.querySelectorAll('[required]');
        let isValid = true;

        for (const field of requiredFields) {
            if (!field.value.trim()) {
                const errorElement = document.getElementById(`${field.id}-error`);
                errorElement.textContent = 'Это поле обязательно для заполнения';
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                isValid = false;
                
                // Фокус на первом невалидном поле
                if (isValid === false) {
                    field.focus();
                    break;
                }
            } else {
                const errorElement = document.getElementById(`${field.id}-error`);
                errorElement.textContent = '';
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
            }
        }

        return isValid;
    }
}

function nextStep(current) {
    const form = new AccessibleMultiStepForm('accessible-multi-step-form');
    
    form.validateCurrentStep().then(isValid => {
        if (isValid) {
            document.getElementById(`step${current}`).hidden = true;
            document.getElementById(`step${current}`).classList.remove('active');
            
            currentStep = current + 1;
            
            const nextStepElement = document.getElementById(`step${currentStep}`);
            nextStepElement.hidden = false;
            nextStepElement.classList.add('active');
            
            form.currentStep = currentStep;
            form.updateProgress();
            
            // Фокус на заголовок следующего шага для скринридеров
            const heading = nextStepElement.querySelector('h2');
            if (heading) {
                heading.setAttribute('tabindex', '-1');
                heading.focus();
            }
            
            // Обновляем сводку на последнем шаге
            if (currentStep === 3) {
                updateSummary();
            }
        }
    });
}

function prevStep(current) {
    document.getElementById(`step${current}`).hidden = true;
    document.getElementById(`step${current}`).classList.remove('active');
    
    currentStep = current - 1;
    
    const prevStepElement = document.getElementById(`step${currentStep}`);
    prevStepElement.hidden = false;
    prevStepElement.classList.add('active');
    
    const form = new AccessibleMultiStepForm('accessible-multi-step-form');
    form.currentStep = currentStep;
    form.updateProgress();
    
    // Фокус на заголовок предыдущего шага
    const heading = prevStepElement.querySelector('h2');
    if (heading) {
        heading.setAttribute('tabindex', '-1');
        heading.focus();
    }
}

function updateSummary() {
    const formData = new FormData(document.getElementById('accessible-multi-step-form'));
    const summaryContent = document.getElementById('summary-content');
    
    summaryContent.innerHTML = `
        <p><strong>Email:</strong> ${formData.get('email') || 'Не указан'}</p>
        <p><strong>Телефон:</strong> ${formData.get('phone') || 'Не указан'}</p>
        <p><strong>Имя:</strong> ${formData.get('name') || 'Не указано'}</p>
        <p><strong>Дата рождения:</strong> ${formData.get('birthdate') || 'Не указана'}</p>
        <p><strong>Пол:</strong> ${getGenderText(formData.get('gender'))}</p>
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

// Инициализация формы
document.addEventListener('DOMContentLoaded', () => {
    new AccessibleMultiStepForm('accessible-multi-step-form');
});
</script>
```

## Доступные интерактивные элементы форм

### Доступный переключатель (toggle switch)
```html
<div class="switch-container">
    <label id="notifications-label">Уведомления</label>
    <div class="switch" 
         role="switch"
         aria-checked="false"
         aria-labelledby="notifications-label"
         tabindex="0"
         onclick="toggleSwitch(this)"
         onkeydown="handleSwitchKeydown(event)">
        <div class="switch-thumb"></div>
    </div>
    <span class="switch-status" aria-live="polite">Выключено</span>
</div>

<div class="switch-container">
    <label id="dark-mode-label">Темная тема</label>
    <div class="switch" 
         role="switch"
         aria-checked="false"
         aria-labelledby="dark-mode-label"
         tabindex="0"
         onclick="toggleSwitch(this)"
         onkeydown="handleSwitchKeydown(event)">
        <div class="switch-thumb"></div>
    </div>
    <span class="switch-status" aria-live="polite">Выключено</span>
</div>

<style>
    .switch-container {
        display: flex;
        align-items: center;
        gap: 10px;
        margin: 15px 0;
    }

    .switch {
        width: 50px;
        height: 24px;
        background-color: #ccc;
        border-radius: 12px;
        position: relative;
        cursor: pointer;
        transition: background-color 0.3s ease;
    }

    .switch[aria-checked="true"] {
        background-color: #1976d2;
    }

    .switch-thumb {
        width: 20px;
        height: 20px;
        background-color: white;
        border-radius: 50%;
        position: absolute;
        top: 2px;
        left: 2px;
        transition: transform 0.3s ease;
    }

    .switch[aria-checked="true"] .switch-thumb {
        transform: translateX(26px);
    }

    .switch:focus {
        outline: 2px solid #1976d2;
        outline-offset: 2px;
    }

    .switch-status {
        font-size: 0.9em;
        color: #666;
    }
</style>

<script>
function toggleSwitch(element) {
    const isCurrentlyChecked = element.getAttribute('aria-checked') === 'true';
    const newCheckedState = !isCurrentlyChecked;

    element.setAttribute('aria-checked', newCheckedState);

    // Обновить визуальное состояние
    element.classList.toggle('checked', newCheckedState);

    // Обновить статус
    const statusElement = element.nextElementSibling;
    statusElement.textContent = newCheckedState ? 'Включено' : 'Выключено';
    
    // Сообщение для скринридера
    announceToScreenReader(`Переключатель ${newCheckedState ? 'включен' : 'выключен'}`);
}

function handleSwitchKeydown(event) {
    if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        toggleSwitch(event.target);
    }
}

function announceToScreenReader(message) {
    const announcement = document.createElement('div');
    announcement.setAttribute('aria-live', 'polite');
    announcement.setAttribute('aria-atomic', 'true');
    announcement.className = 'sr-only';
    announcement.textContent = message;

    document.body.appendChild(announcement);

    setTimeout(() => {
        if (document.body.contains(announcement)) {
            document.body.removeChild(announcement);
        }
    }, 1000);
}
</script>
```

### Доступный слайдер (range input)
```html
<div class="slider-container">
    <label for="volume-slider">Громкость: <span id="volume-value">50</span>%</label>
    <input type="range" 
           id="volume-slider" 
           name="volume" 
           min="0" 
           max="100" 
           value="50"
           aria-valuetext="50%"
           onchange="updateVolume(this)"
           oninput="updateVolume(this)">
</div>

<div class="slider-container">
    <label for="brightness-slider">Яркость: <span id="brightness-value">75</span>%</label>
    <input type="range" 
           id="brightness-slider" 
           name="brightness" 
           min="0" 
           max="100" 
           value="75"
           aria-valuetext="75%"
           onchange="updateBrightness(this)"
           oninput="updateBrightness(this)">
</div>

<style>
    .slider-container {
        margin: 20px 0;
    }

    .slider-container label {
        display: block;
        margin-bottom: 10px;
        font-weight: bold;
    }

    #volume-slider, #brightness-slider {
        width: 100%;
    }
</style>

<script>
function updateVolume(slider) {
    const value = slider.value;
    document.getElementById('volume-value').textContent = value;
    slider.setAttribute('aria-valuetext', `${value}%`);
    
    // Сообщение для скринридера
    announceToScreenReader(`Громкость установлена на ${value}%`);
}

function updateBrightness(slider) {
    const value = slider.value;
    document.getElementById('brightness-value').textContent = value;
    slider.setAttribute('aria-valuetext', `${value}%`);
    
    // Сообщение для скринридера
    announceToScreenReader(`Яркость установлена на ${value}%`);
}

function announceToScreenReader(message) {
    const announcement = document.createElement('div');
    announcement.setAttribute('aria-live', 'polite');
    announcement.setAttribute('aria-atomic', 'true');
    announcement.className = 'sr-only';
    announcement.textContent = message;

    document.body.appendChild(announcement);

    setTimeout(() => {
        if (document.body.contains(announcement)) {
            document.body.removeChild(announcement);
        }
    }, 1000);
}
</script>
```

## Лучшие практики доступности форм

### 1. Правильная структура и семантика
```html
<!-- Хорошо: семантически правильная форма -->
<form role="form" aria-label="Форма обратной связи">
    <h2>Обратная связь</h2>
    
    <div class="form-group">
        <label for="good-name">Имя:</label>
        <input type="text" 
               id="good-name" 
               name="name" 
               required
               aria-describedby="name-help">
        <div id="name-help">Введите ваше имя</div>
    </div>
    
    <fieldset>
        <legend>Тип обращения:</legend>
        <label>
            <input type="radio" name="type" value="question" required>
            Вопрос
        </label>
        <label>
            <input type="radio" name="type" value="complaint">
            Жалоба
        </label>
        <label>
            <input type="radio" name="type" value="feedback">
            Отзыв
        </label>
    </fieldset>
    
    <button type="submit">Отправить</button>
</form>

<!-- Плохо: отсутствие семантики -->
<form>
    <div>Имя: <input type="text" name="name" required></div>
    <div>
        <input type="radio" name="type" value="question" required> Вопрос
        <input type="radio" name="type" value="complaint"> Жалоба
        <input type="radio" name="type" value="feedback"> Отзыв
    </div>
    <input type="submit" value="Отправить">
</form>
```

### 2. Ясные и последовательные метки
```html
<!-- Хорошо: ясные метки и вспомогательный текст -->
<div class="form-group">
    <label for="clear-email">Email для связи:</label>
    <input type="email" 
           id="clear-email" 
           name="email" 
           required
           aria-describedby="email-purpose email-format">
    <div id="email-purpose" class="help-text">Мы будем использовать этот email для ответа на ваш запрос</div>
    <div id="email-format" class="help-text">Формат: example@domain.com</div>
</div>

<!-- Плохо: неясные метки -->
<div class="form-group">
    <label for="unclear-email">Email:</label>
    <input type="email" id="unclear-email" name="email" required>
    <small>example@domain.com</small>
</div>
```

### 3. Доступные сообщения об ошибках
```html
<!-- Хорошо: структурированные и доступные ошибки -->
<form id="error-form">
    <div class="form-group">
        <label for="error-email">Email:</label>
        <input type="email" 
               id="error-email" 
               name="email" 
               required
               aria-describedby="error-email-help error-email-error"
               aria-invalid="false">
        <div id="error-email-help" class="help-text">Введите действительный email</div>
        <div id="error-email-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
    
    <div class="form-group">
        <label>
            <input type="checkbox" 
                   name="terms" 
                   required
                   aria-describedby="terms-error">
            Я согласен с условиями
        </label>
        <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
    
    <button type="submit">Отправить</button>
</form>

<script>
document.getElementById('error-form').addEventListener('submit', function(e) {
    e.preventDefault();
    
    // Сброс ошибок
    document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
    document.querySelectorAll('[aria-invalid]').forEach(el => el.setAttribute('aria-invalid', 'false'));
    
    let hasErrors = false;
    
    // Проверка email
    const email = document.getElementById('error-email');
    if (!email.value.trim()) {
        document.getElementById('error-email-error').textContent = 'Email обязателен для заполнения';
        email.setAttribute('aria-invalid', 'true');
        hasErrors = true;
    } else if (!email.validity.valid) {
        document.getElementById('error-email-error').textContent = 'Введите действительный email адрес';
        email.setAttribute('aria-invalid', 'true');
        hasErrors = true;
    }
    
    // Проверка условий
    const terms = this.querySelector('input[name="terms"]');
    if (!terms.checked) {
        document.getElementById('terms-error').textContent = 'Необходимо согласие с условиями';
        hasErrors = true;
    }
    
    if (hasErrors) {
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
    } else {
        alert('Форма успешно отправлена!');
    }
});
</script>
```

## Пример комплексной доступной формы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Доступная регистрационная форма</title>
    <meta name="description" content="Пример полностью доступной HTML формы">
    
    <style>
        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .form-container {
            background: #f9f9f9;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        .form-group {
            margin-bottom: 25px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
            transition: border-color 0.3s;
        }
        
        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 5px rgba(0, 122, 204, 0.3);
        }
        
        input.valid {
            border-color: #4caf50;
        }
        
        input.invalid {
            border-color: #f44336;
        }
        
        .help-text {
            color: #666;
            font-size: 0.9em;
            margin-top: 5px;
        }
        
        .error-message {
            color: #d32f2f;
            font-size: 0.9em;
            margin-top: 5px;
            font-weight: bold;
        }
        
        fieldset {
            border: 2px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 25px;
        }
        
        legend {
            font-weight: bold;
            padding: 0 10px;
            color: #333;
        }
        
        .checkbox-group, .radio-group {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        
        .checkbox-group label, .radio-group label {
            display: flex;
            align-items: center;
            gap: 8px;
            font-weight: normal;
            margin-bottom: 0;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 12px 30px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
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
        
        .required {
            color: #d32f2f;
        }
        
        .form-header {
            text-align: center;
            margin-bottom: 30px;
        }
        
        .progress-container {
            margin-bottom: 20px;
            text-align: center;
        }
        
        .progress-bar {
            height: 20px;
            background-color: #e0e0e0;
            border-radius: 10px;
            margin-bottom: 10px;
            overflow: hidden;
        }
        
        .progress {
            height: 100%;
            background-color: #007acc;
            transition: width 0.3s;
        }
    </style>
</head>
<body>
    <a href="#main-form" class="sr-only">Перейти к основной форме</a>
    
    <div class="form-container">
        <header class="form-header">
            <h1>Регистрация нового пользователя</h1>
            <p>Пожалуйста, заполните все обязательные поля, отмеченные <span class="required">*</span></p>
        </header>
        
        <div class="progress-container">
            <div class="progress-bar">
                <div class="progress" id="registration-progress" style="width: 0%;"></div>
            </div>
            <div id="progress-text">Шаг 1 из 3</div>
        </div>
        
        <form id="main-form" action="/register" method="post" novalidate>
            <!-- Шаг 1: Контактная информация -->
            <section id="step1" class="form-step" aria-labelledby="step1-title" role="region">
                <h2 id="step1-title">Контактная информация</h2>
                
                <div class="form-group">
                    <label for="email">Email: <span class="required">*</span></label>
                    <input type="email" 
                           id="email" 
                           name="email" 
                           required
                           aria-describedby="email-help email-error"
                           aria-invalid="false"
                           aria-errormessage="email-error">
                    <div id="email-help" class="help-text">Адрес для подтверждения регистрации и связи</div>
                    <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="phone">Телефон:</label>
                    <input type="tel" 
                           id="phone" 
                           name="phone"
                           aria-describedby="phone-help phone-error"
                           aria-invalid="false"
                           aria-errormessage="phone-error">
                    <div id="phone-help" class="help-text">Формат: +7 (XXX) XXX-XX-XX</div>
                    <div id="phone-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <button type="button" onclick="nextStep(1)" id="next1-btn">Далее</button>
            </section>
            
            <!-- Шаг 2: Личная информация -->
            <section id="step2" class="form-step" aria-labelledby="step2-title" role="region" hidden>
                <h2 id="step2-title">Личная информация</h2>
                
                <div class="form-group">
                    <label for="first-name">Имя: <span class="required">*</span></label>
                    <input type="text" 
                           id="first-name" 
                           name="first_name" 
                           required
                           minlength="2"
                           maxlength="50"
                           aria-describedby="first-name-error"
                           aria-invalid="false"
                           aria-errormessage="first-name-error">
                    <div id="first-name-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="last-name">Фамилия: <span class="required">*</span></label>
                    <input type="text" 
                           id="last-name" 
                           name="last_name" 
                           required
                           minlength="2"
                           maxlength="50"
                           aria-describedby="last-name-error"
                           aria-invalid="false"
                           aria-errormessage="last-name-error">
                    <div id="last-name-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="birthdate">Дата рождения:</label>
                    <input type="date" 
                           id="birthdate" 
                           name="birthdate" 
                           max="2010-01-01"
                           aria-describedby="birthdate-error"
                           aria-invalid="false"
                           aria-errormessage="birthdate-error">
                    <div id="birthdate-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <fieldset aria-describedby="gender-help">
                    <legend>Пол</legend>
                    <div id="gender-help" class="help-text">Выберите ваш пол</div>
                    <div class="radio-group">
                        <label>
                            <input type="radio" name="gender" value="male" id="gender-male">
                            Мужской
                        </label>
                        <label>
                            <input type="radio" name="gender" value="female" id="gender-female">
                            Женский
                        </label>
                        <label>
                            <input type="radio" name="gender" value="other" id="gender-other">
                            Другой
                        </label>
                    </div>
                </fieldset>
                
                <button type="button" onclick="prevStep(2)" id="prev2-btn">Назад</button>
                <button type="button" onclick="nextStep(2)" id="next2-btn">Далее</button>
            </section>
            
            <!-- Шаг 3: Аккаунт и подтверждение -->
            <section id="step3" class="form-step" aria-labelledby="step3-title" role="region" hidden>
                <h2 id="step3-title">Аккаунт и подтверждение</h2>
                
                <div class="form-group">
                    <label for="username">Имя пользователя: <span class="required">*</span></label>
                    <input type="text" 
                           id="username" 
                           name="username" 
                           required
                           minlength="3"
                           maxlength="20"
                           pattern="[A-Za-z0-9_]+"
                           aria-describedby="username-help username-error"
                           aria-invalid="false"
                           aria-errormessage="username-error">
                    <div id="username-help" class="help-text">Только буквы, цифры и подчеркивание, 3-20 символов</div>
                    <div id="username-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="password">Пароль: <span class="required">*</span></label>
                    <input type="password" 
                           id="password" 
                           name="password" 
                           required 
                           minlength="8"
                           aria-describedby="password-requirements password-error"
                           aria-invalid="false"
                           aria-errormessage="password-error">
                    <ul id="password-requirements" class="help-text">
                        <li>Минимум 8 символов</li>
                        <li>Содержит заглавную букву</li>
                        <li>Содержит строчную букву</li>
                        <li>Содержит цифру</li>
                    </ul>
                    <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="confirm-password">Подтверждение пароля: <span class="required">*</span></label>
                    <input type="password" 
                           id="confirm-password" 
                           name="confirm_password" 
                           required
                           aria-describedby="confirm-password-error"
                           aria-invalid="false"
                           aria-errormessage="confirm-password-error">
                    <div id="confirm-password-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox" 
                               name="newsletter" 
                               value="yes"
                               id="newsletter">
                        Подписаться на рассылку полезных материалов
                    </label>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox" 
                               name="terms" 
                               required
                               id="terms"
                               aria-describedby="terms-error">
                        Я согласен с <a href="/terms" target="_blank">условиями использования</a> и 
                        <a href="/privacy" target="_blank">политикой конфиденциальности</a> <span class="required">*</span>
                    </label>
                    <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div id="summary" class="summary" role="region" aria-labelledby="summary-title">
                    <h3 id="summary-title">Проверьте вашу информацию:</h3>
                    <div id="summary-content"></div>
                </div>
                
                <button type="button" onclick="prevStep(3)" id="prev3-btn">Назад</button>
                <button type="submit" id="submit-btn">Зарегистрироваться</button>
            </section>
        </form>
    </div>

    <script>
        let currentStep = 1;
        const totalSteps = 3;
        
        class AccessibleRegistrationForm {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.currentStep = 1;
                this.setupForm();
            }

            setupForm() {
                // Валидация при потере фокуса
                this.form.addEventListener('blur', (e) => {
                    if (e.target.matches('input[required]')) {
                        this.validateField(e.target);
                    }
                }, true);

                // Валидация пароля в реальном времени
                document.getElementById('password').addEventListener('input', () => {
                    this.updatePasswordRequirements();
                });

                // Валидация подтверждения пароля
                document.getElementById('confirm-password').addEventListener('input', () => {
                    this.validatePasswordMatch();
                });

                // Отправка формы
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
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

            updatePasswordRequirements() {
                const password = document.getElementById('password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password)
                };

                const reqElements = document.querySelectorAll('#password-requirements li');
                reqElements[0].style.color = requirements.length ? '#4caf50' : '#666';
                reqElements[1].style.color = requirements.upper ? '#4caf50' : '#666';
                reqElements[2].style.color = requirements.lower ? '#4caf50' : '#666';
                reqElements[3].style.color = requirements.number ? '#4caf50' : '#666';
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

            validateAndSubmit() {
                // Валидация всех полей на текущем шаге
                const stepElement = document.getElementById(`step${this.currentStep}`);
                const requiredFields = stepElement.querySelectorAll('[required]');
                let allValid = true;

                requiredFields.forEach(field => {
                    if (!this.validateField(field)) {
                        allValid = false;
                    }
                });

                if (this.currentStep === 3) {
                    // Дополнительная проверка для последнего шага
                    if (!this.validatePasswordMatch()) {
                        allValid = false;
                    }
                }

                if (allValid) {
                    if (this.currentStep < totalSteps) {
                        nextStep(this.currentStep);
                    } else {
                        // Отправка формы
                        this.submitForm();
                    }
                } else {
                    // Сообщение о наличии ошибок
                    const errorCount = this.form.querySelectorAll('.error-message:not(:empty)').length;
                    this.announceError(`Найдено ${errorCount} ошибок. Пожалуйста, исправьте их перед продолжением.`);
                }
            }

            submitForm() {
                alert('Форма успешно зарегистрирована!');
                // Здесь могла бы быть реальная отправка данных
            }

            updateProgress() {
                const filledFields = this.form.querySelectorAll('input:not([type="checkbox"]):not([type="radio"])').length;
                const completedFields = Array.from(this.form.querySelectorAll('input:not([type="checkbox"]):not([type="radio"])'))
                    .filter(input => input.value.trim() !== '').length;
                
                const progress = (completedFields / filledFields) * 100;
                document.getElementById('registration-progress').style.width = `${Math.min(progress, 100)}%`;
            }

            getErrorMessage(field) {
                if (field.validity.valueMissing) {
                    return 'Это поле обязательно для заполнения';
                } else if (field.validity.typeMismatch && field.type === 'email') {
                    return 'Введите действительный email адрес';
                } else if (field.validity.tooShort) {
                    return `Введите не менее ${field.minLength} символов`;
                } else if (field.validity.patternMismatch) {
                    return 'Неверный формат данных';
                }
                return 'Неверный формат данных';
            }

            announceError(message) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'assertive');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = message;

                document.body.appendChild(announcement);

                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 5000);
            }
        }

        function nextStep(current) {
            const form = new AccessibleRegistrationForm('main-form');
            
            // Валидация текущего шага
            const stepElement = document.getElementById(`step${current}`);
            const requiredFields = stepElement.querySelectorAll('[required]');
            let stepValid = true;

            requiredFields.forEach(field => {
                if (field.hasAttribute('required') && !field.value.trim()) {
                    const errorElement = document.getElementById(`${field.id}-error`);
                    errorElement.textContent = 'Это поле обязательно для заполнения';
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    stepValid = false;
                }
            });

            if (stepValid) {
                document.getElementById(`step${current}`).hidden = true;
                
                currentStep = current + 1;
                
                const nextStepElement = document.getElementById(`step${currentStep}`);
                nextStepElement.hidden = false;
                
                // Обновляем прогресс
                const progressPercent = (currentStep / totalSteps) * 100;
                document.getElementById('registration-progress').style.width = `${progressPercent}%`;
                document.getElementById('progress-text').textContent = `Шаг ${currentStep} из ${totalSteps}`;
                
                // Фокус на заголовок следующего шага
                const heading = nextStepElement.querySelector('h2');
                if (heading) {
                    heading.setAttribute('tabindex', '-1');
                    heading.focus();
                }
                
                // Обновляем сводку на последнем шаге
                if (currentStep === 3) {
                    updateSummary();
                }
            } else {
                form.announceError('Пожалуйста, заполните все обязательные поля перед продолжением.');
            }
        }

        function prevStep(current) {
            document.getElementById(`step${current}`).hidden = true;
            
            currentStep = current - 1;
            
            const prevStepElement = document.getElementById(`step${currentStep}`);
            prevStepElement.hidden = false;
            
            // Обновляем прогресс
            const progressPercent = (currentStep / totalSteps) * 100;
            document.getElementById('registration-progress').style.width = `${progressPercent}%`;
            document.getElementById('progress-text').textContent = `Шаг ${currentStep} из ${totalSteps}`;
            
            // Фокус на заголовок предыдущего шага
            const heading = prevStepElement.querySelector('h2');
            if (heading) {
                heading.setAttribute('tabindex', '-1');
                heading.focus();
            }
        }

        function updateSummary() {
            const formData = new FormData(document.getElementById('main-form'));
            const summaryContent = document.getElementById('summary-content');
            
            summaryContent.innerHTML = `
                <p><strong>Email:</strong> ${formData.get('email') || 'Не указан'}</p>
                <p><strong>Имя:</strong> ${formData.get('first_name') || 'Не указано'} ${formData.get('last_name') || ''}</p>
                <p><strong>Телефон:</strong> ${formData.get('phone') || 'Не указан'}</p>
                <p><strong>Имя пользователя:</strong> ${formData.get('username') || 'Не указано'}</p>
            `;
        }

        // Инициализация формы
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleRegistrationForm('main-form');
        });
    </script>
</body>
</html>
```

## Современные возможности доступности форм

### Валидация с доступностью в реальном времени
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная валидация в реальном времени</title>
    <style>
        .form-container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            font-family: Arial, sans-serif;
        }
        
        .form-group {
            margin-bottom: 25px;
            position: relative;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        
        input:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 5px rgba(0, 122, 204, 0.3);
        }
        
        input.valid {
            border-color: #4caf50;
        }
        
        input.invalid {
            border-color: #f44336;
        }
        
        .status-icon {
            position: absolute;
            right: 15px;
            top: 40px;
            font-size: 1.2em;
        }
        
        .help-text {
            color: #666;
            font-size: 0.9em;
            margin-top: 5px;
        }
        
        .error-message {
            color: #d32f2f;
            font-size: 0.9em;
            margin-top: 5px;
            font-weight: bold;
        }
        
        .success-message {
            color: #4caf50;
            font-size: 0.9em;
            margin-top: 5px;
        }
        
        .requirements-list {
            list-style: none;
            padding: 0;
            margin-top: 5px;
        }
        
        .requirements-list li {
            font-size: 0.9em;
            margin-bottom: 3px;
        }
        
        .requirements-list li.fulfilled {
            color: #4caf50;
        }
        
        .requirements-list li.unfulfilled {
            color: #d32f2f;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Форма с доступной валидацией в реальном времени</h1>
        
        <form id="realtime-validation-form">
            <div class="form-group">
                <label for="rtv-email">Email: <span class="required">*</span></label>
                <input type="email"
                       id="rtv-email"
                       name="email"
                       required
                       aria-describedby="rtv-email-help rtv-email-status"
                       aria-invalid="false">
                <div id="rtv-email-help" class="help-text">Введите действительный email для связи</div>
                <div id="rtv-email-status" class="status-icon" aria-live="polite">⏳</div>
                <div id="rtv-email-error" class="error-message" role="alert" aria-live="assertive"></div>
                <div id="rtv-email-success" class="success-message" role="status" aria-live="polite"></div>
            </div>
            
            <div class="form-group">
                <label for="rtv-password">Пароль: <span class="required">*</span></label>
                <input type="password"
                       id="rtv-password"
                       name="password"
                       required
                       minlength="8"
                       aria-describedby="password-requirements rtv-password-status"
                       aria-invalid="false">
                <ul id="password-requirements" class="requirements-list">
                    <li id="req-length">Не менее 8 символов</li>
                    <li id="req-upper">С заглавной буквой</li>
                    <li id="req-lower">Со строчной буквой</li>
                    <li id="req-number">С цифрой</li>
                    <li id="req-special">С специальным символом</li>
                </ul>
                <div id="rtv-password-status" class="status-icon" aria-live="polite">⏳</div>
                <div id="rtv-password-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="rtv-confirm-password">Подтверждение пароля: <span class="required">*</span></label>
                <input type="password"
                       id="rtv-confirm-password"
                       name="confirm_password"
                       required
                       aria-describedby="rtv-confirm-status"
                       aria-invalid="false">
                <div id="rtv-confirm-status" class="status-icon" aria-live="polite">⏳</div>
                <div id="rtv-confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <button type="submit">Зарегистрироваться</button>
        </form>
    </div>

    <script>
        class RealTimeAccessibleValidator {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupValidation();
            }
            
            setupValidation() {
                // Валидация email в реальном времени
                document.getElementById('rtv-email').addEventListener('input', (e) => {
                    this.validateEmailRealtime(e.target);
                });
                
                // Валидация пароля в реальном времени
                document.getElementById('rtv-password').addEventListener('input', (e) => {
                    this.validatePasswordRealtime(e.target);
                });
                
                // Валидация подтверждения пароля в реальном времени
                document.getElementById('rtv-confirm-password').addEventListener('input', (e) => {
                    this.validatePasswordMatchRealtime(e.target);
                });
                
                // Отправка формы
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateFormComplete();
                });
            }
            
            validateEmailRealtime(field) {
                const email = field.value.trim();
                const statusIcon = document.getElementById('rtv-email-status');
                const errorElement = document.getElementById('rtv-email-error');
                const successElement = document.getElementById('rtv-email-success');
                
                // Очищаем предыдущие состояния
                field.classList.remove('valid', 'invalid');
                statusIcon.textContent = '⏳';
                statusIcon.style.color = '#666';
                errorElement.textContent = '';
                successElement.textContent = '';
                
                if (!email) {
                    statusIcon.textContent = '';
                    field.setAttribute('aria-invalid', 'false');
                    return true;
                }
                
                if (!this.isValidEmail(email)) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    statusIcon.textContent = '✗';
                    statusIcon.style.color = '#f44336';
                    errorElement.textContent = 'Введите действительный email адрес';
                    return false;
                }
                
                // Симуляция проверки уникальности
                setTimeout(() => {
                    if (field.value === email) { // Проверяем, что значение не изменилось
                        field.classList.add('valid');
                        field.setAttribute('aria-invalid', 'false');
                        statusIcon.textContent = '✓';
                        statusIcon.style.color = '#4caf50';
                        successElement.textContent = 'Email доступен';
                    }
                }, 800);
                
                return true;
            }
            
            validatePasswordRealtime(field) {
                const password = field.value;
                const statusIcon = document.getElementById('rtv-password-status');
                const errorElement = document.getElementById('rtv-password-error');
                
                // Очищаем предыдущие состояния
                field.classList.remove('valid', 'invalid');
                statusIcon.textContent = '⏳';
                statusIcon.style.color = '#666';
                errorElement.textContent = '';
                
                if (!password) {
                    statusIcon.textContent = '';
                    this.clearPasswordRequirements();
                    field.setAttribute('aria-invalid', 'false');
                    return true;
                }
                
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password),
                    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
                };
                
                this.updatePasswordRequirements(requirements);
                
                if (Object.values(requirements).every(req => req)) {
                    field.classList.add('valid');
                    field.setAttribute('aria-invalid', 'false');
                    statusIcon.textContent = '✓';
                    statusIcon.style.color = '#4caf50';
                    return true;
                } else {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    statusIcon.textContent = '✗';
                    statusIcon.style.color = '#f44336';
                    return false;
                }
            }
            
            validatePasswordMatchRealtime(field) {
                const password = document.getElementById('rtv-password').value;
                const confirmPassword = field.value;
                const statusIcon = document.getElementById('rtv-confirm-status');
                const errorElement = document.getElementById('rtv-confirm-error');
                
                // Очищаем предыдущие состояния
                field.classList.remove('valid', 'invalid');
                statusIcon.textContent = '⏳';
                statusIcon.style.color = '#666';
                errorElement.textContent = '';
                
                if (!confirmPassword) {
                    statusIcon.textContent = '';
                    field.setAttribute('aria-invalid', 'false');
                    return true;
                }
                
                if (password !== confirmPassword) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    statusIcon.textContent = '✗';
                    statusIcon.style.color = '#f44336';
                    errorElement.textContent = 'Пароли не совпадают';
                    return false;
                } else {
                    field.classList.add('valid');
                    field.setAttribute('aria-invalid', 'false');
                    statusIcon.textContent = '✓';
                    statusIcon.style.color = '#4caf50';
                    return true;
                }
            }
            
            validateFormComplete() {
                const emailField = document.getElementById('rtv-email');
                const passwordField = document.getElementById('rtv-password');
                const confirmField = document.getElementById('rtv-confirm-password');
                
                const isEmailValid = this.isValidEmail(emailField.value) && emailField.value.trim() !== '';
                const isPasswordValid = this.isPasswordValid(passwordField.value);
                const isMatchValid = passwordField.value === confirmField.value && confirmField.value.trim() !== '';
                
                if (isEmailValid && isPasswordValid && isMatchValid) {
                    alert('Форма успешно валидирована и готова к отправке!');
                    // Здесь могла бы быть отправка формы
                } else {
                    const errorAnnouncement = document.createElement('div');
                    errorAnnouncement.setAttribute('aria-live', 'assertive');
                    errorAnnouncement.setAttribute('aria-atomic', 'true');
                    errorAnnouncement.className = 'sr-only';
                    errorAnnouncement.textContent = 'Форма содержит ошибки. Пожалуйста, исправьте все поля перед отправкой.';
                    
                    document.body.appendChild(errorAnnouncement);
                    
                    setTimeout(() => {
                        if (document.body.contains(errorAnnouncement)) {
                            document.body.removeChild(errorAnnouncement);
                        }
                    }, 5000);
                }
            }
            
            isValidEmail(email) {
                const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return emailRegex.test(email);
            }
            
            isPasswordValid(password) {
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password),
                    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
                };
                
                return Object.values(requirements).every(req => req);
            }
            
            updatePasswordRequirements(requirements) {
                document.getElementById('req-length').className = requirements.length ? 'fulfilled' : 'unfulfilled';
                document.getElementById('req-upper').className = requirements.upper ? 'fulfilled' : 'unfulfilled';
                document.getElementById('req-lower').className = requirements.lower ? 'fulfilled' : 'unfulfilled';
                document.getElementById('req-number').className = requirements.number ? 'fulfilled' : 'unfulfilled';
                document.getElementById('req-special').className = requirements.special ? 'fulfilled' : 'unfulfilled';
            }
            
            clearPasswordRequirements() {
                const reqElements = document.querySelectorAll('#password-requirements li');
                reqElements.forEach(el => el.className = 'unfulfilled');
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new RealTimeAccessibleValidator('realtime-validation-form');
        });
    </script>
</body>
</html>
```

### Доступные формы с Web Components
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные формы с Web Components</title>
    <style>
        .form-container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            font-family: Arial, sans-serif;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Форма с доступными Web Components</h1>
        
        <form id="web-component-form">
            <accessible-input
                type="email"
                name="email"
                label="Email"
                required
                help-text="Введите действительный email для связи">
            </accessible-input>
            
            <accessible-input
                type="password"
                name="password"
                label="Пароль"
                required
                help-text="Введите надежный пароль">
            </accessible-input>
            
            <accessible-input
                type="tel"
                name="phone"
                label="Телефон"
                help-text="Формат: +7 (XXX) XXX-XX-XX">
            </accessible-input>
            
            <accessible-select
                name="country"
                label="Страна"
                required>
                <option value="">Выберите страну</option>
                <option value="ru">Россия</option>
                <option value="us">США</option>
                <option value="de">Германия</option>
            </accessible-select>
            
            <button type="submit">Отправить</button>
        </form>
    </div>

    <script>
        // Кастомный элемент доступного поля ввода
        class AccessibleInput extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                const label = this.getAttribute('label') || '';
                const type = this.getAttribute('type') || 'text';
                const name = this.getAttribute('name') || '';
                const required = this.hasAttribute('required');
                const helpText = this.getAttribute('help-text') || '';
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .input-container {
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
                            border: 2px solid #ddd;
                            border-radius: 4px;
                            font-size: 16px;
                        }
                        
                        input:focus {
                            outline: none;
                            border-color: #007acc;
                            box-shadow: 0 0 5px rgba(0, 122, 204, 0.3);
                        }
                        
                        input.valid {
                            border-color: #4caf50;
                        }
                        
                        input.invalid {
                            border-color: #f44336;
                        }
                        
                        .help-text {
                            color: #666;
                            font-size: 0.9em;
                            margin-top: 5px;
                        }
                        
                        .error-message {
                            color: #d32f2f;
                            font-size: 0.9em;
                            margin-top: 5px;
                        }
                    </style>
                    
                    <div class="input-container">
                        <label for="input-${name}">${label}${required ? ' <span class="required">*</span>' : ''}</label>
                        <input 
                            type="${type}" 
                            id="input-${name}" 
                            name="${name}"
                            ${required ? 'required' : ''}
                            aria-describedby="help-${name} error-${name}"
                            aria-invalid="false">
                        ${helpText ? `<div id="help-${name}" class="help-text">${helpText}</div>` : ''}
                        <div id="error-${name}" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                `;
                
                this.input = this.shadowRoot.querySelector('input');
                this.errorElement = this.shadowRoot.querySelector(`#error-${name}`);
                
                // Добавляем обработчики событий
                this.input.addEventListener('blur', () => this.validate());
                this.input.addEventListener('input', () => this.clearError());
            }
            
            validate() {
                if (!this.input.value.trim() && this.input.hasAttribute('required')) {
                    this.showError('Это поле обязательно для заполнения');
                    return false;
                }
                
                if (this.input.type === 'email' && this.input.value && !this.isValidEmail(this.input.value)) {
                    this.showError('Введите действительный email адрес');
                    return false;
                }
                
                this.clearError();
                return true;
            }
            
            showError(message) {
                this.input.classList.add('invalid');
                this.input.classList.remove('valid');
                this.input.setAttribute('aria-invalid', 'true');
                this.errorElement.textContent = message;
            }
            
            clearError() {
                this.input.classList.remove('invalid');
                this.input.classList.add('valid');
                this.input.setAttribute('aria-invalid', 'false');
                this.errorElement.textContent = '';
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            getValue() {
                return this.input.value;
            }
            
            setValue(value) {
                this.input.value = value;
            }
        }
        
        // Кастомный элемент доступного выпадающего списка
        class AccessibleSelect extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                const label = this.getAttribute('label') || '';
                const name = this.getAttribute('name') || '';
                const required = this.hasAttribute('required');
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .select-container {
                            margin-bottom: 20px;
                        }
                        
                        label {
                            display: block;
                            margin-bottom: 5px;
                            font-weight: bold;
                        }
                        
                        select {
                            width: 100%;
                            padding: 10px;
                            border: 2px solid #ddd;
                            border-radius: 4px;
                            font-size: 16px;
                        }
                        
                        select:focus {
                            outline: none;
                            border-color: #007acc;
                            box-shadow: 0 0 5px rgba(0, 122, 204, 0.3);
                        }
                        
                        select.valid {
                            border-color: #4caf50;
                        }
                        
                        select.invalid {
                            border-color: #f44336;
                        }
                    </style>
                    
                    <div class="select-container">
                        <label for="select-${name}">${label}${required ? ' <span class="required">*</span>' : ''}</label>
                        <select 
                            id="select-${name}" 
                            name="${name}"
                            ${required ? 'required' : ''}
                            aria-describedby="error-select-${name}"
                            aria-invalid="false">
                            <slot></slot>
                        </select>
                        <div id="error-select-${name}" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                `;
                
                this.select = this.shadowRoot.querySelector('select');
                this.errorElement = this.shadowRoot.querySelector(`#error-select-${name}`);
                
                // Добавляем обработчик события
                this.select.addEventListener('change', () => this.validate());
            }
            
            validate() {
                if (this.select.hasAttribute('required') && !this.select.value) {
                    this.select.classList.add('invalid');
                    this.select.classList.remove('valid');
                    this.select.setAttribute('aria-invalid', 'true');
                    this.errorElement.textContent = 'Пожалуйста, выберите значение';
                    return false;
                }
                
                this.select.classList.remove('invalid');
                this.select.classList.add('valid');
                this.select.setAttribute('aria-invalid', 'false');
                this.errorElement.textContent = '';
                return true;
            }
            
            getValue() {
                return this.select.value;
            }
            
            setValue(value) {
                this.select.value = value;
            }
        }
        
        // Регистрация кастомных элементов
        customElements.define('accessible-input', AccessibleInput);
        customElements.define('accessible-select', AccessibleSelect);
        
        // Обработка формы
        document.getElementById('web-component-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            // Получаем все кастомные элементы и валидируем их
            const inputs = this.querySelectorAll('accessible-input, accessible-select');
            let allValid = true;
            
            inputs.forEach(input => {
                if (!input.validate) return;
                
                if (!input.validate()) {
                    allValid = false;
                }
            });
            
            if (allValid) {
                alert('Форма успешно валидирована!');
                // Здесь могла бы быть отправка данных
            } else {
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
        });
    </script>
</body>
</html>
```

## Практические примеры и шаблоны

### Доступная многошаговая форма с индикатором прогресса
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная многошаговая форма</title>
    <style>
        .form-container {
            max-width: 700px;
            margin: 0 auto;
            padding: 20px;
            font-family: Arial, sans-serif;
        }
        
        .progress-container {
            margin-bottom: 30px;
        }
        
        .progress-bar {
            height: 25px;
            background-color: #e0e0e0;
            border-radius: 12px;
            overflow: hidden;
            position: relative;
        }
        
        .progress {
            height: 100%;
            background-color: #007acc;
            transition: width 0.3s ease;
        }
        
        .progress-text {
            text-align: center;
            margin-top: 5px;
            font-weight: bold;
        }
        
        .step-indicators {
            display: flex;
            justify-content: space-between;
            margin-top: 10px;
        }
        
        .step-indicator {
            text-align: center;
            flex: 1;
        }
        
        .step-number {
            display: inline-block;
            width: 30px;
            height: 30px;
            line-height: 30px;
            border-radius: 50%;
            background-color: #ccc;
            color: white;
            margin-bottom: 5px;
        }
        
        .step-indicator.active .step-number {
            background-color: #007acc;
        }
        
        .step-indicator.completed .step-number {
            background-color: #4caf50;
        }
        
        .step-title {
            font-size: 0.8em;
        }
        
        .form-step {
            display: none;
        }
        
        .form-step.active {
            display: block;
            animation: fadeIn 0.5s;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        
        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 5px rgba(0, 122, 204, 0.3);
        }
        
        .button-group {
            margin-top: 30px;
            display: flex;
            justify-content: space-between;
        }
        
        button {
            padding: 12px 24px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        
        button.primary {
            background-color: #007acc;
            color: white;
        }
        
        button.secondary {
            background-color: #f5f5f5;
            color: #333;
        }
        
        button:disabled {
            background-color: #cccccc;
            cursor: not-allowed;
        }
        
        .summary {
            background-color: #f9f9f9;
            padding: 15px;
            border-radius: 8px;
            margin-top: 20px;
        }
        
        .summary h3 {
            margin-top: 0;
        }
        
        .required {
            color: #d32f2f;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Регистрация участника мероприятия</h1>
        
        <form id="accessible-multi-step">
            <!-- Индикатор прогресса -->
            <div class="progress-container">
                <div class="progress-bar">
                    <div class="progress" id="multi-step-progress" style="width: 25%;"></div>
                </div>
                <div class="progress-text" id="progress-text">Шаг 1 из 4: Персональная информация</div>
                
                <div class="step-indicators">
                    <div class="step-indicator active" id="indicator-1">
                        <div class="step-number">1</div>
                        <div class="step-title">Информация</div>
                    </div>
                    <div class="step-indicator" id="indicator-2">
                        <div class="step-number">2</div>
                        <div class="step-title">Детали</div>
                    </div>
                    <div class="step-indicator" id="indicator-3">
                        <div class="step-number">3</div>
                        <div class="step-title">Предпочтения</div>
                    </div>
                    <div class="step-indicator" id="indicator-4">
                        <div class="step-number">4</div>
                        <div class="step-title">Подтверждение</div>
                    </div>
                </div>
            </div>
            
            <!-- Шаг 1: Персональная информация -->
            <div class="form-step active" id="step-1" role="region" aria-labelledby="step1-title">
                <h2 id="step1-title">Персональная информация</h2>
                
                <div class="form-group">
                    <label for="first-name">Имя: <span class="required">*</span></label>
                    <input type="text"
                           id="first-name"
                           name="first_name"
                           required
                           aria-describedby="first-name-error"
                           aria-invalid="false">
                    <div id="first-name-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="last-name">Фамилия: <span class="required">*</span></label>
                    <input type="text"
                           id="last-name"
                           name="last_name"
                           required
                           aria-describedby="last-name-error"
                           aria-invalid="false">
                    <div id="last-name-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="email">Email: <span class="required">*</span></label>
                    <input type="email"
                           id="email"
                           name="email"
                           required
                           aria-describedby="email-error"
                           aria-invalid="false">
                    <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="button-group">
                    <div></div> <!-- Пустой div для выравнивания -->
                    <button type="button" class="primary" onclick="nextStep(1)">Далее</button>
                </div>
            </div>
            
            <!-- Шаг 2: Контактная информация -->
            <div class="form-step" id="step-2" role="region" aria-labelledby="step2-title" hidden>
                <h2 id="step2-title">Контактная информация</h2>
                
                <div class="form-group">
                    <label for="phone">Телефон:</label>
                    <input type="tel"
                           id="phone"
                           name="phone"
                           aria-describedby="phone-error"
                           aria-invalid="false">
                    <div id="phone-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="company">Компания:</label>
                    <input type="text"
                           id="company"
                           name="company"
                           aria-describedby="company-error"
                           aria-invalid="false">
                    <div id="company-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="form-group">
                    <label for="position">Должность:</label>
                    <input type="text"
                           id="position"
                           name="position"
                           aria-describedby="position-error"
                           aria-invalid="false">
                    <div id="position-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="button-group">
                    <button type="button" class="secondary" onclick="prevStep(2)">Назад</button>
                    <button type="button" class="primary" onclick="nextStep(2)">Далее</button>
                </div>
            </div>
            
            <!-- Шаг 3: Предпочтения -->
            <div class="form-step" id="step-3" role="region" aria-labelledby="step3-title" hidden>
                <h2 id="step3-title">Предпочтения участия</h2>
                
                <fieldset aria-describedby="dietary-help">
                    <legend>Диетические предпочтения</legend>
                    <div id="dietary-help" class="help-text">Выберите ваши диетические предпочтения для кофе-брейков и обедов</div>
                    <div class="checkbox-group">
                        <label>
                            <input type="checkbox" name="dietary" value="vegetarian">
                            Вегетарианская диета
                        </label>
                        <label>
                            <input type="checkbox" name="dietary" value="vegan">
                            Веганская диета
                        </label>
                        <label>
                            <input type="checkbox" name="dietary" value="gluten-free">
                            Без глютена
                        </label>
                    </div>
                </fieldset>
                
                <fieldset aria-describedby="accessibility-help">
                    <legend>Специальные требования</legend>
                    <div id="accessibility-help" class="help-text">Укажите любые специальные требования для участия в мероприятии</div>
                    <div class="form-group">
                        <label for="accessibility-req">Требования:</label>
                        <textarea id="accessibility-req" name="accessibility_req" rows="3" aria-describedby="accessibility-req-error"></textarea>
                        <div id="accessibility-req-error" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                </fieldset>
                
                <div class="button-group">
                    <button type="button" class="secondary" onclick="prevStep(3)">Назад</button>
                    <button type="button" class="primary" onclick="nextStep(3)">Далее</button>
                </div>
            </div>
            
            <!-- Шаг 4: Подтверждение -->
            <div class="form-step" id="step-4" role="region" aria-labelledby="step4-title" hidden>
                <h2 id="step4-title">Подтверждение регистрации</h2>
                
                <div class="summary" role="region" aria-labelledby="summary-title">
                    <h3 id="summary-title">Ваша информация:</h3>
                    <div id="summary-content"></div>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox"
                               name="newsletter"
                               value="yes"
                               id="newsletter">
                        Получать новости и обновления мероприятия
                    </label>
                </div>
                
                <div class="form-group">
                    <label>
                        <input type="checkbox"
                               name="terms"
                               required
                               id="terms"
                               aria-describedby="terms-error">
                        Я согласен с <a href="/terms" target="_blank">условиями участия</a> <span class="required">*</span>
                    </label>
                    <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
                
                <div class="button-group">
                    <button type="button" class="secondary" onclick="prevStep(4)">Назад</button>
                    <button type="submit" class="primary">Зарегистрироваться</button>
                </div>
            </div>
        </form>
    </div>

    <script>
        let currentStep = 1;
        const totalSteps = 4;
        
        class AccessibleMultiStepForm {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.currentStep = 1;
                this.setupForm();
            }
            
            setupForm() {
                // Отправка формы
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.submitForm();
                });
                
                // Клавиатурная навигация
                this.form.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter' && e.altKey) {
                        if (this.currentStep < totalSteps) {
                            e.preventDefault();
                            this.goToStep(this.currentStep + 1);
                        }
                    }
                });
            }
            
            goToStep(step) {
                // Сначала валидируем текущий шаг
                if (this.currentStep < step && !this.validateCurrentStep()) {
                    return false;
                }
                
                // Скрываем текущий шаг
                document.getElementById(`step-${this.currentStep}`).hidden = true;
                document.getElementById(`indicator-${this.currentStep}`).classList.remove('active');
                
                // Показываем новый шаг
                this.currentStep = step;
                document.getElementById(`step-${this.currentStep}`).hidden = false;
                document.getElementById(`indicator-${this.currentStep}`).classList.add('active');
                
                // Обновляем индикаторы
                for (let i = 1; i <= totalSteps; i++) {
                    const indicator = document.getElementById(`indicator-${i}`);
                    if (i < step) {
                        indicator.classList.add('completed');
                    } else if (i > step) {
                        indicator.classList.remove('completed');
                    }
                }
                
                // Обновляем прогресс
                this.updateProgress();
                
                // Фокус на заголовок шага для скринридеров
                const heading = document.getElementById(`step${this.currentStep}-title`);
                if (heading) {
                    heading.setAttribute('tabindex', '-1');
                    heading.focus();
                    // Убираем tabindex после фокуса
                    setTimeout(() => heading.removeAttribute('tabindex'), 100);
                }
                
                // Обновляем сводку на последнем шаге
                if (step === 4) {
                    this.updateSummary();
                }
                
                return true;
            }
            
            validateCurrentStep() {
                const stepElement = document.getElementById(`step-${this.currentStep}`);
                const requiredFields = stepElement.querySelectorAll('[required]');
                let isValid = true;
                
                // Сбрасываем ошибки
                stepElement.querySelectorAll('.error-message').forEach(el => el.textContent = '');
                stepElement.querySelectorAll('[aria-invalid]').forEach(el => el.setAttribute('aria-invalid', 'false'));
                
                requiredFields.forEach(field => {
                    if (!field.value.trim()) {
                        const errorElement = document.getElementById(`${field.id}-error`);
                        errorElement.textContent = 'Это поле обязательно для заполнения';
                        field.setAttribute('aria-invalid', 'true');
                        field.classList.add('invalid');
                        isValid = false;
                        
                        // Фокус на первом невалидном поле
                        if (isValid === false) {
                            field.focus();
                        }
                    }
                });
                
                return isValid;
            }
            
            updateProgress() {
                const progressPercent = (this.currentStep / totalSteps) * 100;
                document.getElementById('multi-step-progress').style.width = `${progressPercent}%`;
                
                const stepTitles = [
                    'Шаг 1 из 4: Персональная информация',
                    'Шаг 2 из 4: Контактная информация',
                    'Шаг 3 из 4: Предпочтения участия',
                    'Шаг 4 из 4: Подтверждение регистрации'
                ];
                
                document.getElementById('progress-text').textContent = stepTitles[this.currentStep - 1];
                
                // Обновляем ARIA атрибуты прогресса
                this.form.querySelector('.progress-container').setAttribute('aria-valuenow', this.currentStep);
                this.form.querySelector('.progress-container').setAttribute('aria-valuemax', totalSteps);
                this.form.querySelector('.progress-container').setAttribute('aria-valuetext', stepTitles[this.currentStep - 1]);
            }
            
            updateSummary() {
                const formData = new FormData(this.form);
                const summaryContent = document.getElementById('summary-content');
                
                const firstName = formData.get('first_name') || 'Не указано';
                const lastName = formData.get('last_name') || 'Не указано';
                const email = formData.get('email') || 'Не указан';
                const phone = formData.get('phone') || 'Не указан';
                const company = formData.get('company') || 'Не указана';
                const position = formData.get('position') || 'Не указана';
                
                const dietary = Array.from(formData.getAll('dietary')).join(', ') || 'Не указаны';
                const accessibilityReq = formData.get('accessibility_req') || 'Не указаны';
                
                summaryContent.innerHTML = `
                    <p><strong>Имя:</strong> ${firstName} ${lastName}</p>
                    <p><strong>Email:</strong> ${email}</p>
                    <p><strong>Телефон:</strong> ${phone}</p>
                    <p><strong>Компания:</strong> ${company}</p>
                    <p><strong>Должность:</strong> ${position}</p>
                    <p><strong>Диетические предпочтения:</strong> ${dietary}</p>
                    <p><strong>Специальные требования:</strong> ${accessibilityReq || 'Не указаны'}</p>
                `;
            }
            
            async submitForm() {
                // Валидация последнего шага
                if (!this.validateCurrentStep()) {
                    return;
                }
                
                // Симуляция отправки
                const submitBtn = this.form.querySelector('button[type="submit"]');
                const originalText = submitBtn.textContent;
                
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    // Симуляция API вызова
                    await new Promise(resolve => setTimeout(resolve, 2000));
                    
                    // Успешная регистрация
                    alert('Вы успешно зарегистрированы на мероприятие!');
                    
                    // Сброс формы
                    this.form.reset();
                    this.goToStep(1);
                } catch (error) {
                    alert('Ошибка регистрации. Пожалуйста, попробуйте еще раз.');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
        }
        
        function nextStep(current) {
            const form = new AccessibleMultiStepForm('accessible-multi-step');
            form.goToStep(current + 1);
        }
        
        function prevStep(current) {
            const form = new AccessibleMultiStepForm('accessible-multi-step');
            form.goToStep(current - 1);
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleMultiStepForm('accessible-multi-step');
        });
    </script>
</body>
</html>
```

## Проверка и тестирование доступности форм

### Инструменты для проверки доступности:
1. **WAVE** - веб-инструмент для оценки доступности
2. **axe-core** - библиотека для автоматизированного тестирования
3. **Lighthouse** - встроенная проверка в DevTools
4. **WebAIM WCAG Contrast Checker** - проверка контрастности
5. **Keyboard testing** - ручное тестирование клавиатурной навигации
6. **Screen reader testing** - тестирование с помощью скринридеров (NVDA, JAWS, VoiceOver)

### Проверка ключевых аспектов:
1. Все интерактивные элементы доступны с клавиатуры
2. Порядок фокуса логичен и последователен
3. Все поля имеют соответствующие метки
4. Сообщения об ошибках доступны для скринридеров
5. Используются подходящие ARIA-атрибуты
6. Цвет не является единственным средством передачи информации
7. Все элементы имеют достаточный размер для взаимодействия

## Лучшие практики доступности форм

### 1. Семантическая разметка
```html
<!-- Хорошо: семантически правильная разметка -->
<form role="form" aria-label="Форма обратной связи">
    <h2>Обратная связь</h2>
    
    <fieldset>
        <legend>Тип обращения</legend>
        <label>
            <input type="radio" name="type" value="question" required>
            Вопрос
        </label>
        <label>
            <input type="radio" name="type" value="complaint">
            Жалоба
        </label>
    </fieldset>
    
    <div class="form-group">
        <label for="contact-email">Email:</label>
        <input type="email"
               id="contact-email"
               name="email"
               required
               aria-describedby="email-help email-error">
        <div id="email-help">Введите email для обратной связи</div>
        <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
    </div>
</form>
```

### 2. Безопасность и доступность
```html
<!-- Форма с CSRF защитой и доступностью -->
<form action="/secure-submit" method="post">
    <input type="hidden" name="csrf_token" value="token_here">
    
    <div class="form-group">
        <label for="secure-email">Email:</label>
        <input type="email"
               id="secure-email"
               name="email"
               required
               autocomplete="email"
               aria-describedby="secure-email-help">
        <div id="secure-email-help">Ваш email для связи</div>
    </div>
    
    <button type="submit">Отправить</button>
</form>
```

## Заключение

Доступные HTML формы являются важной частью инклюзивного веб-дизайна. Они обеспечивают равный доступ к интерактивным элементам для всех пользователей, включая тех, кто использует вспомогательные технологии. Ключевые аспекты создания доступных форм включают:

1. Правильную семантическую разметку
2. Явные и описательные метки
3. Подробные инструкции и подсказки
4. Понятные сообщения об ошибках
5. Логичный порядок табуляции
6. Совместимость с клавиатурной навигацией
7. Правильное использование ARIA-атрибутов
8. Поддержка вспомогательных технологий
9. Адаптивность под различные пользовательские нужды
10. Соответствие стандартам WCAG

Реализация этих принципов улучшает опыт всех пользователей, не только тех, кто имеет ограниченные возможности. Современные подходы к доступности форм включают использование Web Components, валидацию в реальном времени, многошаговые формы с индикаторами прогресса и другие передовые методы, которые делают веб-формы более удобными и инклюзивными для всех пользователей.

## Следующие темы
- [[HTML/Доступность/Основы]]
- [[HTML/Формы/Валидация]]
- [[HTML/Семантика]]
- [[HTML/Доступность/ARIA]]

## Теги
#html #forms #accessibility #a11y #web-development #frontend #inclusive-design #aria #validation #user-experience #wcag #web-components #real-time-validation #multi-step-forms #screen-readers #keyboard-navigation
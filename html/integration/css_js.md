# HTML Интеграция: С CSS и JavaScript

HTML, CSS и JavaScript работают вместе как три кита современной веб-разработки. Правильная интеграция этих технологий позволяет создавать интерактивные, визуально привлекательные и функциональные веб-страницы.

## Интеграция HTML с CSS

### Варианты подключения CSS

#### 1. Встроенные стили (Inline styles)

```html
<div style="color: blue; font-size: 16px; padding: 10px;">
    Элемент с встроенными стилями
</div>

<p style="background-color: yellow; margin: 10px 0;">
    Абзац с встроенными стилями
</p>
```

> **Примечание**: Использование встроенных стилей не рекомендуется, так как они имеют высокий приоритет и затрудняют поддержку кода.

#### 2. Внутренние стили (Internal styles)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Внутренние стили</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
        }
        
        .header {
            background-color: #333;
            color: white;
            padding: 10px;
            text-align: center;
        }
        
        .content {
            margin: 20px 0;
            line-height: 1.6;
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>Заголовок страницы</h1>
    </div>
    
    <div class="content">
        <p>Содержимое страницы с внутренними стилями</p>
    </div>
</body>
</html>
```

#### 3. Внешние стили (External styles)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Внешние стили</title>
    <link rel="stylesheet" href="styles/main.css">
    <link rel="stylesheet" href="styles/layout.css">
    <link rel="stylesheet" href="styles/components.css">
</head>
<body>
    <header class="main-header">
        <h1>Сайт с внешними стилями</h1>
    </header>
    
    <main class="main-content">
        <p>Содержимое страницы</p>
    </main>
</body>
</html>
```

### CSS селекторы и HTML

#### Селекторы по тегам

```css
/* Стили для всех заголовков h1 */
h1 {
    color: #333;
    font-size: 2em;
    margin-bottom: 0.5em;
}

/* Стили для всех абзацев */
p {
    margin: 0 0 1em 0;
    line-height: 1.6;
}
```

#### Селекторы по классам

```html
<div class="card">
    <h2 class="card-title">Заголовок карточки</h2>
    <p class="card-content">Содержимое карточки</p>
</div>
```

```css
.card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
    margin: 10px 0;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card-title {
    margin: 0 0 10px 0;
    color: #333;
}

.card-content {
    margin: 0;
    color: #666;
}
```

#### Селекторы по ID

```html
<div id="main-navigation">
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
    </ul>
</div>
```

```css
#main-navigation {
    background-color: #333;
    padding: 10px;
}

#main-navigation ul {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
}

#main-navigation li {
    margin-right: 20px;
}

#main-navigation a {
    color: white;
    text-decoration: none;
}
```

### Атрибуты доступности и CSS

```html
<button aria-pressed="false" class="toggle-button">
    Переключатель
</button>
```

```css
.toggle-button[aria-pressed="true"] {
    background-color: #007acc;
    color: white;
}

.toggle-button[aria-pressed="false"] {
    background-color: #f0f0f0;
    color: #333;
}
```

## Интеграция HTML с JavaScript

### Способы подключения JavaScript

#### 1. Встроенный JavaScript

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Встроенный JavaScript</title>
</head>
<body>
    <button onclick="showMessage()">Нажми меня</button>
    
    <script>
        function showMessage() {
            alert('Привет из JavaScript!');
        }
        
        // Использование DOM API
        document.addEventListener('DOMContentLoaded', function() {
            console.log('Страница загружена');
        });
    </script>
</body>
</html>
```

#### 2. Внешние скрипты

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Внешние скрипты</title>
</head>
<body>
    <button id="myButton">Кнопка</button>
    <div id="output"></div>
    
    <!-- Скрипты внизу body для лучшей производительности -->
    <script src="js/utils.js"></script>
    <script src="js/main.js"></script>
    <script src="js/components.js"></script>
</body>
</html>
```

#### 3. Модули JavaScript

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>JavaScript модули</title>
</head>
<body>
    <button id="counter-btn">Счетчик: <span id="count">0</span></button>
    
    <script type="module">
        import { initializeCounter } from './js/counter.js';
        
        initializeCounter();
    </script>
</body>
</html>
```

### Обработка событий

#### Встроенные обработчики

```html
<button onclick="handleClick()">Кнопка</button>
<input type="text" oninput="handleInput(event)" onfocus="handleFocus()" onblur="handleBlur()">
<div onmouseover="handleMouseOver()" onmouseout="handleMouseOut()">Область</div>
```

#### Обработчики через JavaScript

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Обработка событий</title>
</head>
<body>
    <button id="action-btn">Действие</button>
    <input type="text" id="search-input" placeholder="Поиск...">
    <ul id="results"></ul>
    
    <script>
        // Получение элементов
        const button = document.getElementById('action-btn');
        const input = document.getElementById('search-input');
        const results = document.getElementById('results');
        
        // Добавление обработчиков событий
        button.addEventListener('click', function() {
            alert('Кнопка нажата!');
        });
        
        input.addEventListener('input', function(event) {
            const query = event.target.value;
            if (query.length > 2) {
                performSearch(query);
            } else {
                results.innerHTML = '';
            }
        });
        
        function performSearch(query) {
            // Имитация поиска
            results.innerHTML = `
                <li>Результат 1 для "${query}"</li>
                <li>Результат 2 для "${query}"</li>
                <li>Результат 3 для "${query}"</li>
            `;
        }
    </script>
</body>
</html>
```

### DOM манипуляции

#### Создание и добавление элементов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>DOM манипуляции</title>
</head>
<body>
    <div id="container"></div>
    <button id="add-item">Добавить элемент</button>
    
    <script>
        const container = document.getElementById('container');
        const addButton = document.getElementById('add-item');
        let itemCount = 0;
        
        addButton.addEventListener('click', function() {
            itemCount++;
            
            // Создание нового элемента
            const newItem = document.createElement('div');
            newItem.className = 'item';
            newItem.innerHTML = `
                <h3>Элемент ${itemCount}</h3>
                <p>Создан динамически</p>
                <button class="delete-btn" data-id="${itemCount}">Удалить</button>
            `;
            
            // Добавление обработчика для кнопки удаления
            newItem.querySelector('.delete-btn').addEventListener('click', function() {
                const id = this.getAttribute('data-id');
                if (confirm(`Удалить элемент ${id}?`)) {
                    newItem.remove();
                }
            });
            
            // Добавление элемента в контейнер
            container.appendChild(newItem);
        });
    </script>
    
    <style>
        .item {
            border: 1px solid #ddd;
            margin: 10px 0;
            padding: 10px;
            border-radius: 4px;
        }
        
        .delete-btn {
            background-color: #dc3545;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 3px;
            cursor: pointer;
        }
    </style>
</body>
</html>
```

#### Изменение атрибутов и свойств

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Изменение атрибутов</title>
</head>
<body>
    <img id="dynamic-image" src="default.jpg" alt="Изображение">
    <button id="change-image">Сменить изображение</button>
    
    <script>
        const image = document.getElementById('dynamic-image');
        const button = document.getElementById('change-image');
        
        const images = [
            { src: 'image1.jpg', alt: 'Первое изображение' },
            { src: 'image2.jpg', alt: 'Второе изображение' },
            { src: 'image3.jpg', alt: 'Третье изображение' }
        ];
        
        let currentIndex = 0;
        
        button.addEventListener('click', function() {
            currentIndex = (currentIndex + 1) % images.length;
            const imgData = images[currentIndex];
            
            // Изменение атрибутов
            image.setAttribute('src', imgData.src);
            image.setAttribute('alt', imgData.alt);
            
            // Альтернативный способ через свойства
            // image.src = imgData.src;
            // image.alt = imgData.alt;
        });
    </script>
</body>
</html>
```

## Совместное использование HTML, CSS и JavaScript

### Создание интерактивного компонента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивный компонент</title>
    <style>
        .accordion {
            border: 1px solid #ddd;
            margin-bottom: 10px;
            border-radius: 4px;
            overflow: hidden;
        }
        
        .accordion-header {
            background-color: #f5f5f5;
            padding: 15px;
            cursor: pointer;
            user-select: none;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .accordion-header:hover {
            background-color: #e9e9e9;
        }
        
        .accordion-content {
            padding: 0;
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.3s ease-out;
        }
        
        .accordion-content.open {
            padding: 15px;
            max-height: 500px; /* Должно быть больше, чем высота содержимого */
        }
        
        .accordion-icon {
            transition: transform 0.3s ease;
        }
        
        .accordion-header.active .accordion-icon {
            transform: rotate(180deg);
        }
    </style>
</head>
<body>
    <div class="accordion">
        <div class="accordion-header" onclick="toggleAccordion(this)">
            <span>Заголовок аккордеона</span>
            <span class="accordion-icon">▼</span>
        </div>
        <div class="accordion-content">
            <p>Содержимое аккордеона. Этот текст будет скрыт или показан при клике на заголовок.</p>
            <p>Можно добавить любой HTML контент внутрь.</p>
        </div>
    </div>
    
    <div class="accordion">
        <div class="accordion-header" onclick="toggleAccordion(this)">
            <span>Второй аккордеон</span>
            <span class="accordion-icon">▼</span>
        </div>
        <div class="accordion-content">
            <p>Еще один аккордеон с другим содержимым.</p>
        </div>
    </div>
    
    <script>
        function toggleAccordion(header) {
            const content = header.nextElementSibling;
            const isActive = header.classList.contains('active');
            
            // Закрытие всех аккордеонов
            document.querySelectorAll('.accordion-header').forEach(h => {
                h.classList.remove('active');
                h.nextElementSibling.classList.remove('open');
            });
            
            // Открытие текущего аккордеона, если он был закрыт
            if (!isActive) {
                header.classList.add('active');
                content.classList.add('open');
            }
        }
    </script>
</body>
</html>
```

### Динамическая форма с валидацией

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Динамическая форма</title>
    <style>
        .form-container {
            max-width: 500px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        
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
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }
        
        .error {
            border-color: #dc3545 !important;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875em;
            margin-top: 5px;
        }
        
        .success-message {
            color: #28a745;
            font-size: 0.875em;
            margin-top: 5px;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button:hover {
            background-color: #005a9e;
        }
        
        button:disabled {
            background-color: #6c757d;
            cursor: not-allowed;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h2>Регистрация пользователя</h2>
        
        <form id="registration-form">
            <div class="form-group">
                <label for="name">Имя:</label>
                <input type="text" id="name" name="name" required>
                <div class="error-message" id="name-error"></div>
            </div>
            
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
                <div class="error-message" id="email-error"></div>
            </div>
            
            <div class="form-group">
                <label for="password">Пароль:</label>
                <input type="password" id="password" name="password" required minlength="8">
                <div class="error-message" id="password-error"></div>
            </div>
            
            <div class="form-group">
                <label for="confirm-password">Подтверждение пароля:</label>
                <input type="password" id="confirm-password" name="confirm-password" required>
                <div class="error-message" id="confirm-password-error"></div>
            </div>
            
            <button type="submit" id="submit-btn">Зарегистрироваться</button>
            <div id="form-message"></div>
        </form>
    </div>
    
    <script>
        document.getElementById('registration-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            // Сброс ошибок
            document.querySelectorAll('.error-message').forEach(el => el.textContent = '');
            document.querySelectorAll('input').forEach(el => el.classList.remove('error'));
            
            let isValid = true;
            
            // Валидация имени
            const name = document.getElementById('name');
            if (!name.value.trim()) {
                showError(name, 'Имя обязательно для заполнения');
                isValid = false;
            } else if (name.value.trim().length < 2) {
                showError(name, 'Имя должно содержать не менее 2 символов');
                isValid = false;
            }
            
            // Валидация email
            const email = document.getElementById('email');
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            if (!email.value.trim()) {
                showError(email, 'Email обязателен для заполнения');
                isValid = false;
            } else if (!emailRegex.test(email.value)) {
                showError(email, 'Введите действительный email');
                isValid = false;
            }
            
            // Валидация пароля
            const password = document.getElementById('password');
            if (!password.value) {
                showError(password, 'Пароль обязателен для заполнения');
                isValid = false;
            } else if (password.value.length < 8) {
                showError(password, 'Пароль должен содержать не менее 8 символов');
                isValid = false;
            }
            
            // Валидация подтверждения пароля
            const confirmPassword = document.getElementById('confirm-password');
            if (!confirmPassword.value) {
                showError(confirmPassword, 'Подтвердите пароль');
                isValid = false;
            } else if (password.value !== confirmPassword.value) {
                showError(confirmPassword, 'Пароли не совпадают');
                isValid = false;
            }
            
            if (isValid) {
                // Здесь можно отправить данные на сервер
                showMessage('Форма успешно отправлена!', 'success');
                
                // Сброс формы
                this.reset();
            }
        });
        
        // Реальная проверка подтверждения пароля
        document.getElementById('confirm-password').addEventListener('input', function() {
            const password = document.getElementById('password').value;
            const confirmPassword = this.value;
            
            if (password && confirmPassword && password !== confirmPassword) {
                showError(this, 'Пароли не совпадают');
            } else if (password === confirmPassword && password !== '') {
                hideError(this);
            }
        });
        
        function showError(input, message) {
            input.classList.add('error');
            const errorElement = document.getElementById(input.id + '-error');
            errorElement.textContent = message;
        }
        
        function hideError(input) {
            input.classList.remove('error');
            const errorElement = document.getElementById(input.id + '-error');
            errorElement.textContent = '';
        }
        
        function showMessage(message, type) {
            const messageElement = document.getElementById('form-message');
            messageElement.textContent = message;
            messageElement.className = type === 'success' ? 'success-message' : 'error-message';
            
            // Очистка сообщения через 5 секунд
            setTimeout(() => {
                messageElement.textContent = '';
                messageElement.className = '';
            }, 5000);
        }
    </script>
</body>
</html>
```

## Современные подходы интеграции

### Использование Web Components

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Web Components</title>
</head>
<body>
    <h1>Компонент таймера</h1>
    <custom-timer duration="10"></custom-timer>
    <custom-timer duration="30"></custom-timer>
    
    <script>
        class CustomTimer extends HTMLElement {
            constructor() {
                super();
                
                this.shadow = this.attachShadow({mode: 'open'});
                
                this.duration = parseInt(this.getAttribute('duration')) || 10;
                this.remaining = this.duration;
                this.intervalId = null;
                
                this.render();
            }
            
            static get observedAttributes() {
                return ['duration'];
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (name === 'duration') {
                    this.duration = parseInt(newValue) || 10;
                    this.resetTimer();
                }
            }
            
            render() {
                this.shadow.innerHTML = `
                    <style>
                        .timer {
                            display: inline-block;
                            padding: 10px 20px;
                            background-color: #f0f0f0;
                            border-radius: 5px;
                            font-family: monospace;
                            font-size: 1.2em;
                        }
                        
                        .timer.warning {
                            background-color: #ffeb3b;
                        }
                        
                        .timer.danger {
                            background-color: #f44336;
                            color: white;
                        }
                    </style>
                    <div class="timer" id="timer-display">${this.remaining}s</div>
                `;
            }
            
            connectedCallback() {
                this.startTimer();
            }
            
            disconnectedCallback() {
                this.stopTimer();
            }
            
            startTimer() {
                this.stopTimer(); // Остановить предыдущий таймер
                this.intervalId = setInterval(() => {
                    this.remaining--;
                    this.updateDisplay();
                    
                    if (this.remaining <= 0) {
                        this.stopTimer();
                        this.dispatchEvent(new CustomEvent('timer-end', {
                            detail: { duration: this.duration }
                        }));
                    }
                }, 1000);
            }
            
            stopTimer() {
                if (this.intervalId) {
                    clearInterval(this.intervalId);
                    this.intervalId = null;
                }
            }
            
            resetTimer() {
                this.stopTimer();
                this.remaining = this.duration;
                this.updateDisplay();
                this.startTimer();
            }
            
            updateDisplay() {
                const display = this.shadow.getElementById('timer-display');
                display.textContent = `${this.remaining}s`;
                
                // Обновление стилей в зависимости от оставшегося времени
                display.classList.remove('warning', 'danger');
                if (this.remaining <= 5) {
                    display.classList.add(this.remaining <= 2 ? 'danger' : 'warning');
                }
            }
        }
        
        customElements.define('custom-timer', CustomTimer);
        
        // Обработка событий таймера
        document.querySelectorAll('custom-timer').forEach(timer => {
            timer.addEventListener('timer-end', (e) => {
                console.log(`Таймер на ${e.detail.duration} секунд завершен!`);
            });
        });
    </script>
</body>
</html>
```

## Заключение

Интеграция HTML, CSS и JavaScript является фундаментом современной веб-разработки. Правильное использование этих технологий вместе позволяет создавать интерактивные, визуально привлекательные и функциональные веб-приложения. Ключевые аспекты успешной интеграции включают:

1. **Разделение ответственности**: HTML для структуры, CSS для стиля, JavaScript для поведения
2. **Семантическая разметка**: использование правильных HTML элементов
3. **Доступность**: обеспечение работы с клавиатуры и скринридерами
4. **Производительность**: оптимизация загрузки и выполнения скриптов
5. **Поддерживаемость**: чистый, понятный код с правильной архитектурой

## Следующие темы
- [[HTML/Интеграция с JavaScript/События и DOM]]
- [[HTML/Интеграция с CSS/Адаптивная верстка]]
- [[HTML/Продвинутые темы/Шаблоны и веб-компоненты]]

## Теги
#html #css #javascript #integration #web-development #frontend #dom #styling
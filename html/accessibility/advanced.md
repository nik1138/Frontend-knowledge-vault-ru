# Доступность HTML: создание инклюзивных веб-приложений

## Введение

Доступность HTML (Web Accessibility) - это практика создания веб-сайтов и приложений, которые могут использоваться всеми людьми, включая тех, кто имеет ограниченные возможности. Это включает людей с нарушениями зрения, слуха, двигательными ограничениями, когнитивными расстройствами и другими инвалидностями. В этой статье мы рассмотрим ключевые аспекты создания доступных HTML-документов.

## Основы доступности

### 1. Семантическая разметка

Семантическая разметка помогает вспомогательным технологиям (скринридерам, голосовым интерфейсам) понимать структуру и содержание страницы.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Доступный веб-сайт</title>
</head>
<body>
    <!-- Используем семантические элементы -->
    <header>
        <h1>Название сайта</h1>
        <nav aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/services">Услуги</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <h2>Основной контент</h2>
        
        <!-- Структура статьи -->
        <article>
            <header>
                <h3>Заголовок статьи</h3>
                <time datetime="2023-10-15">15 октября 2023</time>
            </header>
            
            <section>
                <h4>Раздел статьи</h4>
                <p>Содержимое раздела...</p>
            </section>
            
            <section>
                <h4>Еще один раздел</h4>
                <p>Более содержимого...</p>
            </section>
            
            <footer>
                <p>Автор: <address>Иван Иванов</address></p>
            </footer>
        </article>
    </main>

    <aside aria-label="Дополнительная информация">
        <h3>Рекомендуем</h3>
        <ul>
            <li><a href="/related1">Связанная статья 1</a></li>
            <li><a href="/related2">Связанная статья 2</a></li>
        </ul>
    </aside>

    <footer>
        <p>&copy; 2023 Название сайта. Все права защищены.</p>
        <nav aria-label="Дополнительная навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/terms">Условия использования</a></li>
            </ul>
        </nav>
    </footer>
</body>
</html>
```

### 2. Правильная иерархия заголовков

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Правильная иерархия заголовков</title>
</head>
<body>
    <!-- Правильная последовательность заголовков -->
    <h1>Основной заголовок страницы</h1>
    
    <h2>Раздел 1</h2>
    <p>Содержимое раздела 1...</p>
    
    <h3>Подраздел 1.1</h3>
    <p>Содержимое подраздела 1.1...</p>
    
    <h3>Подраздел 1.2</h3>
    <p>Содержимое подраздела 1.2...</p>
    
    <h2>Раздел 2</h2>
    <p>Содержимое раздела 2...</p>
    
    <h3>Подраздел 2.1</h3>
    <p>Содержимое подраздела 2.1...</p>
    
    <!-- Плохо: пропуск уровней -->
    <!--
    <h1>Основной заголовок</h1>
    <h4>Подзаголовок</h4> Пропущены h2 и h3
    -->
</body>
</html>
```

### 3. Альтернативный текст для изображений

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Доступные изображения</title>
</head>
<body>
    <!-- Изображения с альтернативным текстом -->
    <img src="logo.png" alt="Логотип компании Acme Inc" />
    
    <!-- Декоративные изображения -->
    <img src="decoration.jpg" alt="" role="presentation" />
    
    <!-- Комплексные изображения с подробным описанием -->
    <img src="chart.png" alt="Диаграмма продаж за 2023 год, показывающая рост на 15% по сравнению с 2022 годом" />
    
    <!-- Изображения с длинным описанием -->
    <img src="infographic.png" alt="Инфографика о здоровом питании" longdesc="description.html" />
    
    <!-- Фигуры с подписями -->
    <figure>
        <img src="photo.jpg" alt="Группа людей на совещании" />
        <figcaption>Команда работает над новым проектом</figcaption>
    </figure>
</body>
</html>
```

## ARIA (Accessible Rich Internet Applications)

### 1. Основы ARIA

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>ARIA атрибуты</title>
</head>
<body>
    <!-- Роли -->
    <div role="banner">Шапка сайта</div>
    <div role="main">Основное содержимое</div>
    <div role="contentinfo">Подвал</div>
    
    <!-- Навигация -->
    <nav role="navigation" aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
        </ul>
    </nav>
    
    <!-- Виджеты -->
    <div role="button" tabindex="0" aria-pressed="false" onclick="toggleButton(this)">
        Переключатель
    </div>
    
    <!-- Лейблы -->
    <label for="search">Поиск:</label>
    <input type="text" id="search" aria-label="Поле поиска" />
    
    <!-- Описание -->
    <div id="help-text">Введите ваш email</div>
    <input type="email" aria-describedby="help-text" />
    
    <!-- Заголовки -->
    <div id="form-header">Регистрация</div>
    <form aria-labelledby="form-header">
        <input type="text" placeholder="Имя" />
    </form>
</body>
</html>
```

### 2. Состояния и свойства ARIA

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>ARIA состояния и свойства</title>
</head>
<body>
    <!-- Состояния -->
    <button 
        aria-expanded="false" 
        aria-controls="collapsible-content"
        onclick="toggleContent(this)">
        Показать детали
    </button>
    
    <div id="collapsible-content" role="region" aria-labelledby="toggle-button" hidden>
        <p>Скрытый контент</p>
    </div>
    
    <!-- Индикаторы прогресса -->
    <div 
        role="progressbar" 
        aria-valuenow="50" 
        aria-valuemin="0" 
        aria-valuemax="100"
        aria-valuetext="50% завершено">
        <span>50%</span>
    </div>
    
    <!-- Списки с ARIA -->
    <ul role="listbox" aria-label="Список задач">
        <li role="option" aria-selected="true">Задача 1</li>
        <li role="option" aria-selected="false">Задача 2</li>
    </ul>
    
    <!-- Вкладки -->
    <div role="tablist" aria-label="Настройки">
        <button 
            role="tab" 
            aria-selected="true" 
            aria-controls="panel1" 
            id="tab1">
            Общие
        </button>
        <button 
            role="tab" 
            aria-selected="false" 
            aria-controls="panel2" 
            id="tab2" 
            tabindex="-1">
            Дополнительно
        </button>
        
        <div 
            role="tabpanel" 
            id="panel1" 
            aria-labelledby="tab1">
            <p>Содержимое вкладки 1</p>
        </div>
        <div 
            role="tabpanel" 
            id="panel2" 
            aria-labelledby="tab2" 
            hidden>
            <p>Содержимое вкладки 2</p>
        </div>
    </div>
</body>
</html>
```

### 3. Живые области (Live Regions)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Живые области ARIA</title>
</head>
<body>
    <!-- Полит для уведомлений -->
    <div 
        aria-live="polite" 
        aria-atomic="true" 
        id="status-updates"
        class="sr-only">
        <!-- Обновления будут объявлены скринридером -->
    </div>
    
    <!-- Агрессивное объявление -->
    <div 
        aria-live="assertive" 
        aria-atomic="true" 
        id="alert-messages"
        class="alert">
        <!-- Критические уведомления -->
    </div>
    
    <!-- Регион с частичным обновлением -->
    <div 
        aria-live="polite" 
        aria-relevant="additions text"
        aria-atomic="false"
        id="chat-messages">
        <!-- Сообщения чата -->
    </div>
    
    <button onclick="addStatusMessage()">Добавить статус</button>
    <button onclick="addAlertMessage()">Добавить предупреждение</button>
    
    <script>
        function addStatusMessage() {
            const statusDiv = document.getElementById('status-updates');
            statusDiv.textContent = 'Статус обновлен: ' + new Date().toLocaleTimeString();
        }
        
        function addAlertMessage() {
            const alertDiv = document.getElementById('alert-messages');
            alertDiv.innerHTML = '<p>Важное уведомление!</p>';
        }
    </script>
</body>
</html>
```

## Формы и доступность

### 1. Доступные формы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Доступные формы</title>
</head>
<body>
    <form action="/submit" method="POST" novalidate>
        <!-- Группировка полей -->
        <fieldset>
            <legend>Личная информация</legend>
            
            <div class="form-group">
                <label for="first-name">Имя <span class="required" aria-label="обязательное поле">*</span></label>
                <input 
                    type="text" 
                    id="first-name" 
                    name="first_name" 
                    required
                    autocomplete="given-name"
                    aria-describedby="first-name-help first-name-error"
                    aria-invalid="false">
                <div id="first-name-help" class="form-hint">Введите ваше имя</div>
                <div id="first-name-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="last-name">Фамилия <span class="required" aria-label="обязательное поле">*</span></label>
                <input 
                    type="text" 
                    id="last-name" 
                    name="last_name" 
                    required
                    autocomplete="family-name"
                    aria-describedby="last-name-error"
                    aria-invalid="false">
                <div id="last-name-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
        </fieldset>
        
        <fieldset>
            <legend>Контактная информация</legend>
            
            <div class="form-group">
                <label for="email">Email <span class="required" aria-label="обязательное поле">*</span></label>
                <input 
                    type="email" 
                    id="email" 
                    name="email" 
                    required
                    autocomplete="email"
                    aria-describedby="email-error"
                    aria-invalid="false">
                <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            
            <div class="form-group">
                <label for="phone">Телефон</label>
                <input 
                    type="tel" 
                    id="phone" 
                    name="phone"
                    autocomplete="tel"
                    aria-describedby="phone-help"
                    pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
                    placeholder="123-456-7890">
                <div id="phone-help" class="form-hint">Формат: 123-456-7890</div>
            </div>
        </fieldset>
        
        <!-- Выбор опций -->
        <fieldset>
            <legend>Предпочтения</legend>
            
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
                <div id="newsletter-help" class="form-hint">Получать новости и обновления</div>
            </div>
            
            <div class="radio-group">
                <p id="contact-method-label">Предпочтительный способ связи:</p>
                <label class="radio-label">
                    <input 
                        type="radio" 
                        name="contact_method" 
                        value="email"
                        aria-describedby="contact-method-label">
                    <span class="radio-custom"></span>
                    Email
                </label>
                <label class="radio-label">
                    <input 
                        type="radio" 
                        name="contact_method" 
                        value="phone"
                        aria-describedby="contact-method-label">
                    <span class="radio-custom"></span>
                    Телефон
                </label>
            </div>
        </fieldset>
        
        <button type="submit">Отправить</button>
    </form>
</body>
</html>
```

### 2. Обработка ошибок форм

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Обработка ошибок форм</title>
    <style>
        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }
        
        input:invalid {
            border-color: #dc3545;
        }
        
        input:valid {
            border-color: #28a745;
        }
    </style>
</head>
<body>
    <form id="validation-form" novalidate>
        <div class="form-group">
            <label for="email">Email <span class="required" aria-label="обязательное поле">*</span></label>
            <input 
                type="email" 
                id="email" 
                name="email" 
                required
                aria-describedby="email-error"
                aria-invalid="false">
            <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="password">Пароль <span class="required" aria-label="обязательное поле">*</span></label>
            <input 
                type="password" 
                id="password" 
                name="password" 
                required
                minlength="8"
                aria-describedby="password-requirements password-error"
                aria-invalid="false">
            <div id="password-requirements" class="form-hint">
                <ul>
                    <li>Минимум 8 символов</li>
                    <li>Хотя бы одна заглавная буква</li>
                    <li>Хотя бы одна строчная буква</li>
                    <li>Хотя бы одна цифра</li>
                </ul>
            </div>
            <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <button type="submit">Войти</button>
    </form>
    
    <script>
        document.getElementById('validation-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            let isValid = true;
            const fields = this.querySelectorAll('[required]');
            
            fields.forEach(field => {
                const errorElement = document.getElementById(field.name + '-error');
                
                if (!field.checkValidity()) {
                    field.setAttribute('aria-invalid', 'true');
                    if (errorElement) {
                        errorElement.textContent = getErrorMessage(field);
                    }
                    isValid = false;
                } else {
                    field.setAttribute('aria-invalid', 'false');
                    if (errorElement) {
                        errorElement.textContent = '';
                    }
                }
            });
            
            if (isValid) {
                // Форма валидна, отправляем данные
                announceSuccess('Форма успешно отправлена!');
            } else {
                announceError('Пожалуйста, исправьте ошибки в форме');
            }
        });
        
        function getErrorMessage(field) {
            if (field.validity.valueMissing) {
                return 'Это поле обязательно для заполнения';
            } else if (field.validity.typeMismatch) {
                return 'Пожалуйста, введите корректный email';
            } else if (field.validity.tooShort) {
                return `Минимум ${field.minLength} символов`;
            }
            return 'Некорректное значение';
        }
        
        function announceSuccess(message) {
            createAnnouncement(message, 'success');
        }
        
        function announceError(message) {
            createAnnouncement(message, 'error');
        }
        
        function createAnnouncement(message, type) {
            const announcement = document.createElement('div');
            announcement.setAttribute('role', type === 'error' ? 'alert' : 'status');
            announcement.setAttribute('aria-live', 'polite');
            announcement.setAttribute('aria-atomic', 'true');
            announcement.className = `sr-only announcement-${type}`;
            announcement.textContent = message;
            
            document.body.appendChild(announcement);
            
            setTimeout(() => {
                document.body.removeChild(announcement);
            }, 5000);
        }
    </script>
</body>
</html>
```

## Клавиатурная навигация

### 1. Порядок фокуса

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Клавиатурная навигация</title>
    <style>
        /* Визуальная индикация фокуса */
        a:focus,
        button:focus,
        input:focus,
        select:focus,
        textarea:focus,
        [tabindex]:focus {
            outline: 2px solid #007bff;
            outline-offset: 2px;
        }
        
        /* Альтернативная индикация фокуса */
        .focus-indicator:focus {
            box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5);
        }
        
        /* Скрытие нативного фокуса только при мыши (не рекомендуется) */
        a:focus:not(:focus-visible),
        button:focus:not(:focus-visible) {
            outline: none;
        }
        
        a:focus-visible,
        button:focus-visible {
            outline: 2px solid #007bff;
            outline-offset: 2px;
        }
    </style>
</head>
<body>
    <!-- Логический порядок табуляции -->
    <nav>
        <a href="/">Главная</a>
        <a href="/about">О нас</a>
        <a href="/services">Услуги</a>
        <a href="/contact">Контакты</a>
    </nav>
    
    <main>
        <h1>Основной контент</h1>
        <p>Текст страницы...</p>
        
        <!-- Ссылка для пропуска навигации -->
        <a href="#main-content" class="skip-link">Перейти к основному содержимому</a>
        
        <div id="main-content">
            <h2>Заголовок основного содержимого</h2>
            <p>Содержимое...</p>
        </div>
        
        <!-- Интерактивные элементы -->
        <button type="button">Кнопка 1</button>
        <button type="button" tabindex="0">Кнопка 2 (явный tabindex)</button>
        <div role="button" tabindex="0" class="custom-button">Кастомная кнопка</div>
    </main>
    
    <!-- Вспомогательные стили для пропуска навигации -->
    <style>
        .skip-link {
            position: absolute;
            top: -40px;
            left: 6px;
            background: #000;
            color: #fff;
            padding: 8px;
            text-decoration: none;
            border-radius: 4px;
            z-index: 1000;
        }
        
        .skip-link:focus {
            top: 6px;
        }
    </style>
</body>
</html>
```

### 2. Интерактивные компоненты

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Интерактивные компоненты</title>
    <style>
        .custom-button {
            display: inline-block;
            padding: 8px 16px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            user-select: none;
        }
        
        .custom-button:focus {
            outline: 2px solid #007bff;
            outline-offset: 2px;
        }
        
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 1000;
        }
        
        .modal.active {
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        .modal-content {
            background: white;
            padding: 20px;
            border-radius: 8px;
            max-width: 500px;
            width: 90%;
        }
    </style>
</head>
<body>
    <button onclick="openModal()">Открыть модальное окно</button>
    
    <div id="modal" class="modal" role="dialog" aria-modal="true" aria-labelledby="modal-title">
        <div class="modal-content">
            <h2 id="modal-title">Модальное окно</h2>
            <p>Содержимое модального окна</p>
            <button onclick="closeModal()">Закрыть</button>
        </div>
    </div>
    
    <script>
        function openModal() {
            const modal = document.getElementById('modal');
            modal.classList.add('active');
            modal.setAttribute('aria-hidden', 'false');
            
            // Сохраняем текущий элемент фокуса
            const currentFocus = document.activeElement;
            modal.dataset.returnFocus = currentFocus.tagName;
            
            // Устанавливаем фокус на модальное окно
            modal.focus();
        }
        
        function closeModal() {
            const modal = document.getElementById('modal');
            modal.classList.remove('active');
            modal.setAttribute('aria-hidden', 'true');
            
            // Возвращаем фокус на предыдущий элемент
            const returnFocus = document.querySelector(modal.dataset.returnFocus);
            if (returnFocus) {
                returnFocus.focus();
            }
        }
        
        // Обработка клавиатурных событий для модального окна
        document.getElementById('modal').addEventListener('keydown', function(e) {
            if (e.key === 'Escape') {
                closeModal();
            }
            
            // Ограничение фокуса внутри модального окна
            if (e.key === 'Tab') {
                const focusableElements = this.querySelectorAll(
                    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
                );
                const firstElement = focusableElements[0];
                const lastElement = focusableElements[focusableElements.length - 1];
                
                if (e.shiftKey) {
                    if (document.activeElement === firstElement) {
                        lastElement.focus();
                        e.preventDefault();
                    }
                } else {
                    if (document.activeElement === lastElement) {
                        firstElement.focus();
                        e.preventDefault();
                    }
                }
            }
        });
    </script>
</body>
</html>
```

## Скрытый контент и вспомогательные технологии

### 1. Скрытие контента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Скрытие контента</title>
    <style>
        /* Класс для скрытия элементов только визуально */
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
        
        /* Класс для скрытия элементов, но с доступом для скринридеров */
        .visually-hidden {
            position: absolute !important;
            height: 1px;
            width: 1px;
            overflow: hidden;
            clip: rect(1px, 1px, 1px, 1px);
        }
        
        /* Класс для полного скрытия */
        .hidden {
            display: none !important;
        }
        
        /* Временное скрытие */
        .visually-hidden.focusable:focus,
        .visually-hidden.focusable:active {
            position: static !important;
            height: auto;
            width: auto;
            overflow: visible;
            clip: auto;
            white-space: normal;
        }
    </style>
</head>
<body>
    <!-- Контент, скрытый визуально но доступный для скринридеров -->
    <h2 class="sr-only">Список товаров</h2>
    
    <!-- Вспомогательная информация -->
    <span class="sr-only" aria-live="polite" id="item-count">
        Найдено 24 товара
    </span>
    
    <!-- Кнопка с расширенным описанием -->
    <button aria-label="Увеличить шрифт">
        <svg aria-hidden="true" width="24" height="24" viewBox="0 0 24 24">
            <!-- SVG иконка -->
        </svg>
        <span class="sr-only">Увеличить размер шрифта</span>
    </button>
    
    <!-- Ссылка для пропуска навигации -->
    <a href="#main-content" class="sr-only sr-only-focusable">
        Перейти к основному содержимому
    </a>
    
    <!-- Скрытие при загрузке, показ при готовности -->
    <div id="loading-content" aria-hidden="true" class="hidden">
        <p>Загрузка контента...</p>
    </div>
    
    <div id="main-content" aria-live="polite">
        <h1>Основной контент</h1>
        <p>Содержимое страницы...</p>
    </div>
    
    <script>
        // Пример показа/скрытия контента
        function showContent() {
            const loading = document.getElementById('loading-content');
            const main = document.getElementById('main-content');
            
            loading.setAttribute('aria-hidden', 'true');
            loading.classList.add('hidden');
            
            main.setAttribute('aria-busy', 'false');
        }
        
        function hideContent() {
            const loading = document.getElementById('loading-content');
            const main = document.getElementById('main-content');
            
            loading.setAttribute('aria-hidden', 'false');
            loading.classList.remove('hidden');
            
            main.setAttribute('aria-busy', 'true');
        }
    </script>
</body>
</html>
```

### 2. Альтернативные интерфейсы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Альтернативные интерфейсы</title>
</head>
<body>
    <!-- Альтернативная навигация для клавиатурных пользователей -->
    <nav class="keyboard-nav" aria-label="Навигация для клавиатуры">
        <ul>
            <li><a href="#main">Перейти к основному содержимому</a></li>
            <li><a href="#search">Перейти к поиску</a></li>
            <li><a href="#footer">Перейти к подвалу</a></li>
        </ul>
    </nav>
    
    <!-- Контент с альтернативными описаниями -->
    <div class="content-with-alternatives">
        <h2>Инфографика: Статистика продаж</h2>
        
        <!-- Альтернативное текстовое описание -->
        <details>
            <summary>Текстовое описание диаграммы</summary>
            <p>Диаграмма показывает рост продаж с января по декабрь 2023 года. 
               Январь: 1000 единиц, Февраль: 1200 единиц, Март: 1500 единиц и так далее. 
               Максимальный рост наблюдается в ноябре и декабре.</p>
        </details>
        
        <!-- Сама инфографика -->
        <div aria-hidden="true">
            <!-- Визуальная инфографика -->
            <img src="sales-chart.png" alt="" />
        </div>
    </div>
    
    <!-- Аудио/видео контент с альтернативами -->
    <figure>
        <video 
            controls 
            aria-describedby="video-description"
            poster="video-poster.jpg">
            <source src="presentation.mp4" type="video/mp4">
            <track kind="captions" src="captions.vtt" srclang="ru" label="Русские субтитры">
            <track kind="descriptions" src="descriptions.vtt" srclang="ru" label="Описание для слабовидящих">
            Ваш браузер не поддерживает видео.
        </video>
        <figcaption id="video-description">
            Презентация о новых возможностях продукта. 
            Длительность: 5 минут 30 секунд. 
            Включает субтитры и аудиоописание.
        </figcaption>
    </figure>
</body>
</html>
```

## Лучшие практики и проверки

### 1. Автоматические проверки

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Проверки доступности</title>
</head>
<body>
    <!-- Структура для автоматических проверок -->
    <header>
        <h1>Название сайта</h1>
        <nav aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>
    
    <main id="main-content">
        <h2>Основной контент</h2>
        
        <!-- Проверка: все изображения имеют alt -->
        <img src="image1.jpg" alt="Описание изображения">
        <img src="decoration.png" alt="" role="presentation">
        
        <!-- Проверка: формы имеют лейблы -->
        <form>
            <div class="form-group">
                <label for="name">Имя</label>
                <input type="text" id="name" name="name" required>
            </div>
        </form>
        
        <!-- Проверка: ссылки имеют осмысленный текст -->
        <a href="/more">Подробнее о продукте</a> <!-- Хорошо -->
        <a href="/more">кликните здесь</a> <!-- Плохо -->
        
        <!-- Проверка: цвет не является единственным средством передачи информации -->
        <div class="status-indicator">
            <span class="status-dot" style="background-color: #28a745;"></span>
            <span class="status-text">Активен</span>
        </div>
    </main>
    
    <footer>
        <p>&copy; 2023 Название сайта</p>
    </footer>
    
    <!-- Инструменты для проверки доступности -->
    <script>
        // Простая проверка доступности
        class AccessibilityChecker {
            static runChecks() {
                const issues = [];
                
                // Проверка изображений без alt
                const images = document.querySelectorAll('img');
                images.forEach(img => {
                    if (!img.hasAttribute('alt')) {
                        issues.push({
                            type: 'error',
                            element: img,
                            message: 'Изображение не имеет атрибута alt'
                        });
                    }
                });
                
                // Проверка ссылок с бессмысленным текстом
                const links = document.querySelectorAll('a');
                links.forEach(link => {
                    const text = link.textContent.trim();
                    if (['кликните здесь', 'читать далее', 'подробнее'].includes(text)) {
                        issues.push({
                            type: 'warning',
                            element: link,
                            message: 'Ссылка использует бессмысленный текст'
                        });
                    }
                });
                
                // Проверка заголовков
                const headers = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
                if (headers.length > 0 && headers[0].tagName !== 'H1') {
                    issues.push({
                        type: 'warning',
                        element: headers[0],
                        message: 'Первый заголовок должен быть H1'
                    });
                }
                
                return issues;
            }
            
            static reportIssues() {
                const issues = this.runChecks();
                
                if (issues.length === 0) {
                    console.log('✅ Доступность: Все проверки пройдены');
                } else {
                    console.log(`⚠️ Доступность: Найдено ${issues.length} проблем`);
                    issues.forEach(issue => {
                        console.log(`${issue.type.toUpperCase()}: ${issue.message}`);
                        console.log('Элемент:', issue.element);
                    });
                }
            }
        }
        
        // Запуск проверок при загрузке
        document.addEventListener('DOMContentLoaded', () => {
            AccessibilityChecker.reportIssues();
        });
    </script>
</body>
</html>
```

### 2. Тестирование с вспомогательными технологиями

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Тестирование доступности</title>
</head>
<body>
    <!-- Страница для тестирования с разными вспомогательными технологиями -->
    
    <!-- Тест для скринридеров -->
    <header role="banner">
        <h1>Тестовая страница доступности</h1>
        <nav aria-label="Основная навигация">
            <ul>
                <li><a href="#section1">Раздел 1</a></li>
                <li><a href="#section2">Раздел 2</a></li>
                <li><a href="#section3">Раздел 3</a></li>
            </ul>
        </nav>
    </header>
    
    <main>
        <section id="section1" aria-labelledby="section1-title">
            <h2 id="section1-title">Раздел 1: Формы</h2>
            <form aria-describedby="form-instructions">
                <div id="form-instructions" class="sr-only">
                    Пожалуйста, заполните все обязательные поля, помеченные звездочкой
                </div>
                
                <div class="form-group">
                    <label for="test-input">Тестовое поле *</label>
                    <input 
                        type="text" 
                        id="test-input" 
                        name="test-input"
                        required
                        aria-describedby="test-input-help test-input-error"
                        aria-invalid="false">
                    <div id="test-input-help" class="form-hint">Введите тестовое значение</div>
                    <div id="test-input-error" class="error-message" role="alert" aria-live="assertive"></div>
                </div>
            </form>
        </section>
        
        <section id="section2" aria-labelledby="section2-title">
            <h2 id="section2-title">Раздел 2: Интерактивные элементы</h2>
            
            <!-- Тест для клавиатурной навигации -->
            <div class="interactive-test">
                <button type="button" id="test-button">Тестовая кнопка</button>
                <div 
                    role="button" 
                    tabindex="0" 
                    class="custom-button"
                    onclick="alert('Кастомная кнопка')"
                    onkeydown="handleKeyDown(event, 'alert(\'Кастомная кнопка\')')">
                    Кастомная кнопка
                </div>
            </div>
        </section>
        
        <section id="section3" aria-labelledby="section3-title">
            <h2 id="section3-title">Раздел 3: Сложные компоненты</h2>
            
            <!-- Аккордеон для тестирования -->
            <div class="accordion" role="tablist">
                <div class="accordion-item">
                    <h3>
                        <button 
                            role="tab" 
                            aria-selected="true" 
                            aria-expanded="true"
                            aria-controls="panel1"
                            id="tab1">
                            Вкладка 1
                        </button>
                    </h3>
                    <div 
                        role="tabpanel" 
                        id="panel1" 
                        aria-labelledby="tab1"
                        hidden>
                        <p>Содержимое первой вкладки</p>
                    </div>
                </div>
            </div>
        </section>
    </main>
    
    <script>
        function handleKeyDown(event, action) {
            if (event.key === 'Enter' || event.key === ' ') {
                event.preventDefault();
                eval(action); // В реальном коде используйте более безопасный способ
            }
        }
        
        // Тестирование фокуса
        document.addEventListener('focusin', function(e) {
            console.log('Фокус на элементе:', e.target);
        });
    </script>
</body>
</html>
```

## Примеры из промышленной разработки

### 1. Доступная таблица данных

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Доступная таблица данных</title>
    <style>
        table {
            width: 100%;
            border-collapse: collapse;
        }
        
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        
        th {
            background-color: #f2f2f2;
            position: sticky;
            top: 0;
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
    <h1>Отчет по продажам</h1>
    
    <table aria-label="Ежемесячный отчет по продажам за 2023 год">
        <caption>Ежемесячный отчет по продажам за 2023 год</caption>
        
        <thead>
            <tr>
                <th scope="col">Месяц</th>
                <th scope="col" sort="ascending" aria-sort="ascending">
                    Продажи
                    <button 
                        aria-label="Сортировать по продажам"
                        onclick="sortTable('sales')">
                        ▲▼
                    </button>
                </th>
                <th scope="col">Цель</th>
                <th scope="col">Выполнение</th>
            </tr>
        </thead>
        
        <tbody>
            <tr>
                <th scope="row">Январь</th>
                <td data-label="Продажи">1,200,000</td>
                <td data-label="Цель">1,000,000</td>
                <td data-label="Выполнение" aria-describedby="jan-performance">
                    <span aria-hidden="true">120%</span>
                    <span class="sr-only">Превышение цели на 20%</span>
                </td>
            </tr>
            <tr>
                <th scope="row">Февраль</th>
                <td data-label="Продажи">1,350,000</td>
                <td data-label="Цель">1,100,000</td>
                <td data-label="Выполнение" aria-describedby="feb-performance">
                    <span aria-hidden="true">123%</span>
                    <span class="sr-only">Превышение цели на 23%</span>
                </td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

### 2. Доступный чат-интерфейс

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <title>Доступный чат-интерфейс</title>
    <style>
        .chat-container {
            display: flex;
            flex-direction: column;
            height: 500px;
            border: 1px solid #ccc;
        }
        
        .chat-messages {
            flex: 1;
            overflow-y: auto;
            padding: 10px;
        }
        
        .chat-input {
            display: flex;
            padding: 10px;
            border-top: 1px solid #ccc;
        }
        
        .chat-input input {
            flex: 1;
            padding: 8px;
        }
        
        .chat-message {
            margin-bottom: 10px;
            padding: 8px;
            border-radius: 4px;
        }
        
        .chat-message.own {
            background-color: #e3f2fd;
            margin-left: 20%;
        }
        
        .chat-message.other {
            background-color: #f5f5f5;
            margin-right: 20%;
        }
    </style>
</head>
<body>
    <div class="chat-container" role="main" aria-label="Чат">
        <div 
            class="chat-messages" 
            role="log" 
            aria-live="polite" 
            aria-atomic="false"
            aria-relevant="additions">
            
            <div class="chat-message other" role="logitem">
                <strong>Иван:</strong> 
                <span>Привет! Как дела?</span>
                <time datetime="2023-10-15T10:30:00">10:30</time>
            </div>
            
            <div class="chat-message own" role="logitem">
                <strong>Вы:</strong> 
                <span>Привет! Все хорошо, спасибо!</span>
                <time datetime="2023-10-15T10:31:00">10:31</time>
            </div>
        </div>
        
        <div class="chat-input">
            <input 
                type="text" 
                id="message-input" 
                placeholder="Введите сообщение..."
                aria-label="Поле ввода сообщения"
                aria-describedby="chat-instructions">
            <button 
                type="button" 
                onclick="sendMessage()"
                aria-label="Отправить сообщение">
                Отправить
            </button>
        </div>
        
        <div id="chat-instructions" class="sr-only">
            Нажмите Enter для отправки сообщения, используйте стрелки для навигации по истории
        </div>
    </div>
    
    <script>
        function sendMessage() {
            const input = document.getElementById('message-input');
            const message = input.value.trim();
            
            if (message) {
                const messagesContainer = document.querySelector('.chat-messages');
                const newMessage = document.createElement('div');
                newMessage.className = 'chat-message own';
                newMessage.setAttribute('role', 'logitem');
                newMessage.innerHTML = `
                    <strong>Вы:</strong> 
                    <span>${message}</span>
                    <time datetime="${new Date().toISOString()}">${new Date().toLocaleTimeString()}</time>
                `;
                
                messagesContainer.appendChild(newMessage);
                input.value = '';
                
                // Прокрутка к новому сообщению
                messagesContainer.scrollTop = messagesContainer.scrollHeight;
            }
        }
        
        document.getElementById('message-input').addEventListener('keydown', function(e) {
            if (e.key === 'Enter') {
                e.preventDefault();
                sendMessage();
            }
        });
    </script>
</body>
</html>
```

## Ссылки на связанные темы
- [[html/best-practices]] - Лучшие практики HTML
- [[css/accessibility]] - Доступность CSS
- [[js/accessibility]] - Доступность JavaScript
- [[web-standards]] - Веб-стандарты

#programming #html #accessibility #a11y #inclusive-design #best-practices
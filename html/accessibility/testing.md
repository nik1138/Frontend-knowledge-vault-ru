# Тестирование доступности веб-сайтов

Тестирование доступности - это процесс проверки того, насколько веб-сайт или веб-приложение доступно для пользователей с различными ограничениями. Эффективное тестирование доступности включает в себя как автоматизированные, так и ручные методы проверки.

## Типы тестирования доступности

### 1. Автоматизированное тестирование
Автоматизированные инструменты могут обнаружить большую часть проблем доступности (около 30-40%), особенно технические ошибки в разметке.

**Преимущества:**
- Быстрое сканирование больших объемов контента
- Повторяемость и согласованность
- Интеграция в процесс разработки
- Обнаружение очевидных ошибок

**Ограничения:**
- Не могут оценить контекст и смысл
- Не проверяют функциональность
- Не обнаруживают все проблемы

**Пример использования AXE-core:**
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Тестирование доступности с AXE</title>
    <script src="https://cdn.jsdelivr.net/npm/axe-core@4.7.2/axe.min.js"></script>
</head>
<body>
    <main id="main-content">
        <h1>Тестовая страница</h1>
        <button onclick="runAccessibilityTest()">Проверить доступность</button>
        <div id="results"></div>
    </main>

    <script>
        function runAccessibilityTest() {
            axe.run('#main-content', {
                runOnly: {
                    type: 'tag',
                    values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa']
                }
            }, function(err, results) {
                if (err) {
                    console.error('Ошибка проверки:', err);
                    return;
                }

                const resultsDiv = document.getElementById('results');
                
                if (results.violations.length === 0) {
                    resultsDiv.innerHTML = '<div style="color: green;">✓ Нет нарушений доступности!</div>';
                } else {
                    resultsDiv.innerHTML = `
                        <div style="color: red;">
                            <h3>Найдено ${results.violations.length} нарушений:</h3>
                            <ul>
                                ${results.violations.map(violation => `
                                    <li>
                                        <strong>${violation.help}:</strong><br>
                                        <code>${violation.nodes[0].html.substring(0, 100)}...</code>
                                    </li>
                                `).join('')}
                            </ul>
                        </div>
                    `;
                }
            });
        }
    </script>
</body>
</html>
```

### 2. Ручное тестирование
Ручное тестирование включает в себя проверку контента с использованием различных методов и инструментов.

**Методы ручного тестирования:**
- Тестирование клавиатурной навигации
- Тестирование со скринридерами
- Проверка контрастности цветов
- Тестирование на разных устройствах
- Использование инструментов разработчика

**Пример ручного тестирования клавиатурной навигации:**
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Тестирование клавиатурной навигации</title>
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

        .keyboard-navigation :focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }

        .interactive-element {
            padding: 10px;
            margin: 5px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
        }

        .interactive-element:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
    </style>
</head>
<body>
    <a href="#main-content" class="skip-link">Перейти к основному содержимому</a>
    
    <nav role="navigation" aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
            <li><a href="/services">Услуги</a></li>
            <li><a href="/contact">Контакты</a></li>
        </ul>
    </nav>

    <main id="main-content">
        <h1>Тестирование клавиатурной навигации</h1>
        
        <div class="interactive-element" tabindex="0" role="button" onclick="handleClick(this)">
            Интерактивный элемент 1
        </div>
        <div class="interactive-element" tabindex="0" role="button" onclick="handleClick(this)">
            Интерактивный элемент 2
        </div>
        <div class="interactive-element" tabindex="0" role="button" onclick="handleClick(this)">
            Интерактивный элемент 3
        </div>
        
        <form>
            <label for="input1">Поле ввода 1:</label>
            <input type="text" id="input1" name="input1">
            
            <label for="input2">Поле ввода 2:</label>
            <input type="text" id="input2" name="input2">
            
            <button type="submit">Отправить</button>
        </form>
    </main>

    <script>
        // Добавляем класс для показа клавиатурной навигации
        document.addEventListener('keydown', function(e) {
            if (e.key === 'Tab') {
                document.body.classList.add('keyboard-navigation');
            }
        });

        document.addEventListener('mousedown', function() {
            document.body.classList.remove('keyboard-navigation');
        });

        function handleClick(element) {
            alert('Элемент "' + element.textContent.trim() + '" активирован');
        }
    </script>
</body>
</html>
```

### 3. Тестирование со скринридерами
Тестирование с использованием программ чтения с экрана, таких как NVDA, JAWS или VoiceOver.

**Популярные скринридеры:**
- **NVDA** - бесплатный скринридер для Windows
- **JAWS** - коммерческий скринридер для Windows
- **VoiceOver** - встроенный скринридер в macOS и iOS
- **TalkBack** - скринридер для Android

**Пример тестирования с ARIA-атрибутами:**
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Тестирование со скринридером</title>
</head>
<body>
    <header role="banner">
        <h1>Сайт с хорошей доступностью</h1>
    </header>

    <nav role="navigation" aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
            <li><a href="/services">Услуги</a></li>
            <li><a href="/contact">Контакты</a></li>
        </ul>
    </nav>

    <main role="main" id="main-content">
        <h2>Основное содержимое</h2>
        
        <section aria-labelledby="section1-title">
            <h3 id="section1-title">Раздел 1</h3>
            <p>Этот контент будет читаться скринридером.</p>
        </section>

        <section aria-labelledby="section2-title">
            <h3 id="section2-title">Раздел 2</h3>
            <p>Еще один раздел с контентом.</p>
        </section>

        <div role="alert" aria-live="assertive" id="alert-container">
            <!-- Сюда будут добавляться алерты -->
        </div>

        <button onclick="showAlert()">Показать алерт</button>
    </main>

    <script>
        function showAlert() {
            const alertContainer = document.getElementById('alert-container');
            alertContainer.textContent = 'Это важное уведомление!';
            
            // Очистка алерта через 5 секунд
            setTimeout(() => {
                alertContainer.textContent = '';
            }, 5000);
        }
    </script>
</body>
</html>
```

## Инструменты тестирования доступности

### 1. Встроенные инструменты браузеров
- **Chrome DevTools** - вкладка Accessibility
- **Firefox Accessibility Inspector** - инспектор дерева доступности
- **Safari Accessibility Inspector** - для проверки на macOS

**Использование Chrome DevTools:**
1. Откройте DevTools (F12 или Ctrl+Shift+I)
2. Перейдите на вкладку "Elements"
3. Выберите элемент
4. Перейдите на вкладку "Accessibility"
5. Просмотрите дерево доступности и атрибуты

### 2. Расширения браузеров
- **axe DevTools** - расширение для Chrome и Firefox
- **WAVE** - веб-инструмент с расширением
- **Accessibility Insights** - от Microsoft
- **Lighthouse** - встроенное расширение Chrome

### 3. Командные инструменты
- **Pa11y** - командная строка для тестирования
- **axe-core CLI** - командная строка для AXE
- **Lighthouse CLI** - командная строка для Lighthouse

**Пример использования Pa11y:**
```bash
# Установка
npm install -g pa11y

# Проверка URL
pa11y https://example.com

# Проверка с определенными стандартами
pa11y https://example.com --standard WCAG2AA

# Проверка с игнорированием определенных ошибок
pa11y https://example.com --ignore color-contrast
```

### 4. Онлайн-инструменты
- **WAVE** - webaim.org/wave
- **Accessibility Insights for Web** - accessibilityinsights.io
- **WebAIM WAVE** - wave.webaim.org
- **Contrast Checker** - webaim.org/resources/contrastchecker

## Проверка контрастности цветов

Контрастность текста должна соответствовать требованиям WCAG:
- Уровень AA: 4.5:1 для нормального текста, 3:1 для крупного текста
- Уровень AAA: 7:1 для нормального текста, 4.5:1 для крупного текста

**Пример проверки контрастности:**
```css
/* Хорошая контрастность */
.good-contrast {
    color: #000000; /* Черный */
    background-color: #ffffff; /* Белый */
    /* Соотношение: 21:1 - превышает требования AAA */
}

.acceptable-contrast {
    color: #333333; /* Темно-серый */
    background-color: #ffffff; /* Белый */
    /* Соотношение: 12.6:1 - превышает требования AA */
}

.borderline-contrast {
    color: #767676; /* Светло-серый */
    background-color: #ffffff; /* Белый */
    /* Соотношение: 4.58:1 - минимальное для AA */
}

.bad-contrast {
    color: #aaaaaa; /* Светло-серый */
    background-color: #eeeeee; /* Очень светло-серый */
    /* Соотношение: 1.4:1 - не соответствует требованиям */
}
```

## Тестирование форм

Формы должны быть доступны для всех пользователей, включая тех, кто использует клавиатуру или скринридеры.

**Пример доступной формы:**
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступная форма</title>
</head>
<body>
    <main>
        <h1>Регистрация пользователя</h1>

        <form id="accessible-form" novalidate>
            <div class="form-group">
                <label for="full-name">Полное имя: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="text"
                       id="full-name"
                       name="full_name"
                       required
                       minlength="2"
                       maxlength="50"
                       autocomplete="name"
                       aria-describedby="name-help name-error"
                       aria-invalid="false">
                <div id="name-help" class="help-text">Введите ваше полное имя</div>
                <div id="name-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>

            <div class="form-group">
                <label for="email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="email"
                       id="email"
                       name="email"
                       required
                       autocomplete="email"
                       aria-describedby="email-help email-error"
                       aria-invalid="false">
                <div id="email-help" class="help-text">Введите действительный email для связи</div>
                <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>

            <div class="form-group">
                <label for="password">Пароль: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="password"
                       id="password"
                       name="password"
                       required
                       minlength="8"
                       aria-describedby="password-requirements password-error"
                       aria-invalid="false">
                <ul id="password-requirements" class="requirements-list">
                    <li>Минимум 8 символов</li>
                    <li>С заглавной буквой</li>
                    <li>С цифрой</li>
                </ul>
                <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>

            <div class="form-group">
                <label for="confirm-password">Подтверждение пароля: <span class="required" aria-label="обязательное поле">*</span></label>
                <input type="password"
                       id="confirm-password"
                       name="confirm_password"
                       required
                       aria-describedby="confirm-error"
                       aria-invalid="false">
                <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>

            <div class="form-group">
                <fieldset aria-describedby="preferences-help">
                    <legend>Предпочтения</legend>
                    <div id="preferences-help" class="help-text">Выберите ваши предпочтения</div>
                    
                    <label>
                        <input type="checkbox" name="newsletter" value="yes">
                        Подписка на рассылку
                    </label>
                    
                    <label>
                        <input type="checkbox" name="terms" value="yes" required>
                        Согласие с условиями <span class="required" aria-label="обязательное поле">*</span>
                    </label>
                </fieldset>
            </div>

            <button type="submit">Зарегистрироваться</button>
        </form>
    </main>

    <script>
        class AccessibleFormValidator {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupValidation();
            }

            setupValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateForm();
                });

                // Валидация в реальном времени
                this.form.addEventListener('blur', (e) => {
                    if (e.target.matches('input[required], textarea[required]')) {
                        this.validateField(e.target);
                    }
                }, true);

                this.form.addEventListener('input', (e) => {
                    if (e.target.matches('input[name="confirm_password"]')) {
                        this.validatePasswordMatch();
                    }
                });
            }

            validateForm() {
                let isValid = true;
                const requiredFields = this.form.querySelectorAll('input[required], textarea[required]');
                
                requiredFields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                });

                if (this.form.querySelector('input[name="password"]').value !== 
                    this.form.querySelector('input[name="confirm_password"]').value) {
                    this.showFieldError(
                        this.form.querySelector('input[name="confirm_password"]'), 
                        'Пароли не совпадают'
                    );
                    isValid = false;
                }

                if (isValid) {
                    alert('Форма успешно валидирована!');
                    // Тут будет отправка формы
                } else {
                    // Сообщение для скринридеров
                    const errorCount = this.form.querySelectorAll('.error-message:not(:empty)').length;
                    this.announceToScreenReader(`Форма содержит ${errorCount} ошибок. Пожалуйста, исправьте их.`);
                }
            }

            validateField(field) {
                const fieldName = field.name.replace(/_/g, '-');
                const errorElement = document.getElementById(`${fieldName}-error`);

                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                if (errorElement) errorElement.textContent = '';

                // Проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    this.showFieldError(field, 'Это поле обязательно для заполнения');
                    return false;
                }

                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    this.showFieldError(field, 'Введите действительный email адрес');
                    return false;
                }

                if (field.minLength && field.value.length < field.minLength) {
                    this.showFieldError(field, `Минимум ${field.minLength} символов`);
                    return false;
                }

                field.classList.add('valid');
                return true;
            }

            validatePasswordMatch() {
                const password = this.form.querySelector('input[name="password"]').value;
                const confirmPassword = this.form.querySelector('input[name="confirm_password"]').value;
                const errorElement = document.getElementById('confirm-error');

                if (confirmPassword && password !== confirmPassword) {
                    this.showFieldError(
                        this.form.querySelector('input[name="confirm_password"]'), 
                        'Пароли не совпадают'
                    );
                    return false;
                }

                if (errorElement) errorElement.textContent = '';
                return true;
            }

            showFieldError(field, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');

                const fieldName = field.name.replace(/_/g, '-');
                const errorElement = document.getElementById(`${fieldName}-error`);
                
                if (errorElement) {
                    errorElement.textContent = message;
                }
            }

            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }

            announceToScreenReader(message) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.style.position = 'absolute';
                announcement.style.left = '-9999px';
                announcement.textContent = message;

                document.body.appendChild(announcement);

                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 3000);
            }
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleFormValidator('accessible-form');
        });
    </script>

    <style>
        .form-group {
            margin-bottom: 1.5rem;
        }

        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }

        fieldset {
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 1rem;
            margin-bottom: 1rem;
        }

        legend {
            font-weight: bold;
            padding: 0 0.5rem;
        }

        input, select, textarea {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }

        input:focus, select:focus, textarea:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }

        input.valid {
            border-color: #28a745;
        }

        input.invalid {
            border-color: #dc3545;
        }

        .help-text {
            color: #666;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }

        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }

        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 0.5rem 0 0;
        }

        .requirements-list li {
            padding: 0.25rem 0;
            font-size: 0.875rem;
        }

        .requirements-list li::before {
            content: "• ";
            color: #28a745;
        }

        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }

        .required {
            color: #d32f2f;
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
</body>
</html>
```

## Тестирование медиа-контента

### Видео с субтитрами и описаниями
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступное видео</title>
</head>
<body>
    <main>
        <h1>Обучающее видео по HTML</h1>

        <figure>
            <video controls
                   width="640"
                   height="360"
                   preload="metadata"
                   aria-describedby="video-description">
                <source src="html-tutorial.mp4" type="video/mp4">
                <source src="html-tutorial.webm" type="video/webm">

                <!-- Субтитры -->
                <track kind="subtitles"
                       src="html-tutorial-ru.vtt"
                       srclang="ru"
                       label="Русские">
                <track kind="subtitles"
                       src="html-tutorial-en.vtt"
                       srclang="en"
                       label="English">

                <!-- Описания для слабовидящих -->
                <track kind="descriptions"
                       src="html-tutorial-desc.vtt"
                       srclang="ru"
                       label="Описания для слабовидящих">

                <!-- Титры -->
                <track kind="captions"
                       src="html-tutorial-captions.vtt"
                       srclang="ru"
                       label="Титры (включая звуки)">

                Ваш браузер не поддерживает видео элемент.
            </video>

            <figcaption id="video-description">
                Обучающее видео по основам HTML. Продолжительность: 15 минут.
                Видео охватывает основные теги HTML, семантическую разметку и лучшие практики.
            </figcaption>
        </figure>

        <!-- Транскрипция -->
        <details aria-labelledby="transcription-title">
            <summary id="transcription-title">Транскрипция видео</summary>
            <div class="transcription">
                <h3>Вступление</h3>
                <p>Добро пожаловать в наше обучающее видео по основам HTML...</p>
                
                <h3>Основная часть</h3>
                <p>HTML (HyperText Markup Language) - это язык разметки...</p>
                
                <h3>Заключение</h3>
                <p>Спасибо за просмотр! Практикуйтесь и создавайте свои веб-страницы.</p>
            </div>
        </details>
    </main>
</body>
</html>
```

### Аудио с транскрипцией
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступное аудио</title>
</head>
<body>
    <main>
        <h1>Подкаст: Современные веб-технологии</h1>

        <figure>
            <audio controls aria-labelledby="audio-title">
                <source src="web-tech-podcast.mp3" type="audio/mpeg">
                <source src="web-tech-podcast.ogg" type="audio/ogg">
                Ваш браузер не поддерживает аудио элемент.
            </audio>

            <figcaption>
                <h2 id="audio-title">Подкаст: Современные веб-технологии</h2>
                <p>Обсуждение последних тенденций в веб-разработке.</p>
            </figcaption>
        </figure>

        <!-- Транскрипция аудио -->
        <section aria-labelledby="transcription-header">
            <h2 id="transcription-header">Транскрипция подкаста</h2>
            <div class="podcast-transcription">
                <h3>Ведущий: Иван Иванов</h3>
                <p>Добро пожаловать на наш подкаст о современных веб-технологиях. Сегодня мы поговорим о последних тенденциях...</p>

                <h3>Гость: Мария Петрова</h3>
                <p>Современные веб-технологии развивается очень быстро. Важно следить за новыми возможностями и лучшими практиками...</p>

                <h3>Ведущий: Иван Иванов</h3>
                <p>Какие технологии вы считаете наиболее перспективными?</p>

                <h3>Гость: Мария Петрова</h3>
                <p>Я думаю, что веб-компоненты, доступность и производительность будут ключевыми направлениями в ближайшие годы...</p>
            </div>
        </section>
    </main>
</body>
</html>
```

## Тестирование сложных компонентов

### Доступные вкладки
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные вкладки</title>
    <style>
        .tabs {
            display: flex;
            border-bottom: 1px solid #ccc;
        }

        .tab {
            padding: 10px 20px;
            cursor: pointer;
            border: 1px solid transparent;
            border-bottom: none;
            background: #f0f0f0;
            margin-right: 2px;
            border-radius: 4px 4px 0 0;
        }

        .tab[aria-selected="true"] {
            background: white;
            border-color: #ccc;
        }

        .tab:focus {
            outline: 2px solid #007acc;
            outline-offset: -2px;
        }

        .tab-panel {
            padding: 20px;
            border: 1px solid #ccc;
            border-top: none;
            display: none;
        }

        .tab-panel[aria-hidden="false"] {
            display: block;
        }
    </style>
</head>
<body>
    <main>
        <h1>Доступные вкладки</h1>

        <div role="tablist" aria-label="Пример вкладок">
            <div role="tab"
                 aria-selected="true"
                 aria-controls="panel-1"
                 class="tab"
                 tabindex="0"
                 onclick="activateTab(event.target)"
                 onkeydown="handleTabKeydown(event)">
                Вкладка 1
            </div>
            <div role="tab"
                 aria-selected="false"
                 aria-controls="panel-2"
                 class="tab"
                 tabindex="-1"
                 onclick="activateTab(event.target)"
                 onkeydown="handleTabKeydown(event)">
                Вкладка 2
            </div>
            <div role="tab"
                 aria-selected="false"
                 aria-controls="panel-3"
                 class="tab"
                 tabindex="-1"
                 onclick="activateTab(event.target)"
                 onkeydown="handleTabKeydown(event)">
                Вкладка 3
            </div>
        </div>

        <div id="panel-1"
             role="tabpanel"
             aria-labelledby="tab-1"
             class="tab-panel"
             aria-hidden="false">
            <h2>Содержимое первой вкладки</h2>
            <p>Это содержимое первой вкладки. Оно будет отображаться, когда активна первая вкладка.</p>
        </div>
        <div id="panel-2"
             role="tabpanel"
             aria-labelledby="tab-2"
             class="tab-panel"
             aria-hidden="true">
            <h2>Содержимое второй вкладки</h2>
            <p>Это содержимое второй вкладки. Оно будет отображаться, когда активна вторая вкладка.</p>
        </div>
        <div id="panel-3"
             role="tabpanel"
             aria-labelledby="tab-3"
             class="tab-panel"
             aria-hidden="true">
            <h2>Содержимое третьей вкладки</h2>
            <p>Это содержимое третьей вкладки. Оно будет отображаться, когда активна третья вкладка.</p>
        </div>
    </main>

    <script>
        function activateTab(tabElement) {
            // Снять выделение с текущей вкладки
            document.querySelectorAll('[role="tab"]').forEach(tab => {
                tab.setAttribute('aria-selected', 'false');
                tab.setAttribute('tabindex', '-1');
            });

            // Выделить выбранную вкладку
            tabElement.setAttribute('aria-selected', 'true');
            tabElement.setAttribute('tabindex', '0');
            tabElement.focus();

            // Скрыть все панели
            document.querySelectorAll('[role="tabpanel"]').forEach(panel => {
                panel.setAttribute('aria-hidden', 'true');
            });

            // Показать соответствующую панель
            const panelId = tabElement.getAttribute('aria-controls');
            document.getElementById(panelId).setAttribute('aria-hidden', 'false');
        }

        function handleTabKeydown(event) {
            const currentTab = event.target;
            let newTab;

            switch(event.key) {
                case 'ArrowRight':
                    newTab = currentTab.nextElementSibling || document.querySelector('[role="tab"]:first-child');
                    break;
                case 'ArrowLeft':
                    newTab = currentTab.previousElementSibling || document.querySelector('[role="tab"]:last-child');
                    break;
                case 'Home':
                    newTab = document.querySelector('[role="tab"]:first-child');
                    break;
                case 'End':
                    newTab = document.querySelector('[role="tab"]:last-child');
                    break;
                case 'Enter':
                case ' ':
                    event.preventDefault();
                    activateTab(currentTab);
                    return;
                default:
                    return;
            }

            event.preventDefault();
            activateTab(newTab);
        }
    </script>
</body>
</html>
```

### Доступное модальное окно
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступное модальное окно</title>
    <style>
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
        }

        .modal-content {
            background: white;
            padding: 20px;
            border-radius: 8px;
            max-width: 500px;
            width: 90%;
            position: relative;
        }

        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .close-button {
            background: none;
            border: none;
            font-size: 1.5em;
            cursor: pointer;
            padding: 0;
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .close-button:focus {
            outline: 2px solid #007acc;
        }
    </style>
</head>
<body>
    <main>
        <h1>Доступное модальное окно</h1>
        <p>Это основное содержимое страницы.</p>
        <button onclick="openModal()">Открыть модальное окно</button>
    </main>

    <div id="modal-overlay" class="modal-overlay" style="display: none;" role="dialog" aria-modal="true" aria-labelledby="modal-title">
        <div class="modal-content" role="document">
            <div class="modal-header">
                <h2 id="modal-title">Заголовок модального окна</h2>
                <button class="close-button" onclick="closeModal()" aria-label="Закрыть модальное окно">&times;</button>
            </div>
            <div class="modal-body">
                <p>Содержимое модального окна. Это важная информация, которую пользователь должен увидеть.</p>
                <button onclick="closeModal()">Закрыть</button>
            </div>
        </div>
    </div>

    <script>
        let lastFocusedElement = null;

        function openModal() {
            // Сохраняем последний сфокусированный элемент
            lastFocusedElement = document.activeElement;

            // Показываем модальное окно
            const modal = document.getElementById('modal-overlay');
            modal.style.display = 'flex';

            // Устанавливаем фокус на модальное окно
            modal.focus();

            // Ограничиваем фокус внутри модального окна
            trapFocus(modal);
        }

        function closeModal() {
            // Скрываем модальное окно
            document.getElementById('modal-overlay').style.display = 'none';

            // Возвращаем фокус на предыдущий элемент
            if (lastFocusedElement && lastFocusedElement.focus) {
                lastFocusedElement.focus();
            }
        }

        function trapFocus(modal) {
            const focusableElements = modal.querySelectorAll(
                'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
            );

            if (focusableElements.length === 0) return;

            const firstElement = focusableElements[0];
            const lastElement = focusableElements[focusableElements.length - 1];

            modal.addEventListener('keydown', function(event) {
                if (event.key === 'Tab') {
                    if (event.shiftKey) {
                        if (document.activeElement === firstElement) {
                            lastElement.focus();
                            event.preventDefault();
                        }
                    } else {
                        if (document.activeElement === lastElement) {
                            firstElement.focus();
                            event.preventDefault();
                        }
                    }
                } else if (event.key === 'Escape') {
                    closeModal();
                }
            });

            // Устанавливаем фокус на первый элемент
            firstElement.focus();
        }

        // Закрытие по клику вне модального окна
        document.getElementById('modal-overlay').addEventListener('click', function(event) {
            if (event.target === this) {
                closeModal();
            }
        });
    </script>
</body>
</html>
```

## Практические проверки доступности

### 1. Проверка структуры документа
```html
<!-- Правильная структура -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Правильная структура документа</title>
</head>
<body>
    <header role="banner">
        <h1>Название сайта</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
            </ul>
        </nav>
    </header>

    <main role="main">
        <h2>Основной заголовок страницы</h2>
        
        <section>
            <h3>Раздел 1</h3>
            <p>Содержимое раздела 1</p>
        </section>
        
        <section>
            <h3>Раздел 2</h3>
            <p>Содержимое раздела 2</p>
        </section>
    </main>

    <aside role="complementary" aria-label="Боковая панель">
        <h3>Полезная информация</h3>
        <p>Дополнительный контент</p>
    </aside>

    <footer role="contentinfo">
        <p>&copy; 2023 Все права защищены</p>
    </footer>
</body>
</html>
```

### 2. Проверка альтернативного текста
```html
<!-- Правильное использование alt -->
<img src="logo.png" alt="Логотип компании Acme Inc">
<img src="decorative.png" alt="" role="presentation">
<img src="chart.png" alt="График роста продаж за последние 5 лет, показывающий увеличение на 25% ежегодно">
<img src="photo.jpg" alt="Фотография команды на корпоративном мероприятии, 2023 год">
```

### 3. Проверка клавиатурной навигации
```css
/* Видимые индикаторы фокуса */
a:focus,
button:focus,
input:focus,
select:focus,
textarea:focus,
[tabindex]:focus {
    outline: 2px solid #007acc;
    outline-offset: 2px;
}

/* Никогда не удаляйте outline без замены */
/* ПЛОХО: a:focus { outline: none; } */

/* ХОРОШО: Замените outline на другой видимый индикатор */
.button:focus {
    outline: 2px solid #007acc;
    outline-offset: 2px;
    box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.2);
}
```

## Отчеты о тестировании доступности

### Шаблон отчета
```
Отчет о тестировании доступности
=================================

Дата: [Дата]
Страница: [URL]
Тестировал: [Имя]

1. Автоматизированное тестирование:
   - Инструмент: AXE-core
   - Найдено нарушений: [Количество]
   - Уровень соответствия: [A/AA/AAA]

2. Ручное тестирование:
   - Клавиатурная навигация: [Пройдена/Не пройдена]
   - Скринридер: [Пройден/Не пройден]
   - Контрастность: [Пройдена/Не пройдена]

3. Найденные проблемы:
   - [Проблема 1]
   - [Проблема 2]

4. Рекомендации:
   - [Рекомендация 1]
   - [Рекомендация 2]

5. Статус исправления:
   - [Открыт/В процессе/Исправлен]
```

## Заключение

Тестирование доступности - это критический этап в разработке веб-сайтов и приложений. Эффективное тестирование включает в себя сочетание автоматизированных инструментов, ручных проверок и тестирования с использованием вспомогательных технологий. Регулярное тестирование позволяет выявлять и устранять проблемы доступности на ранних стадиях, обеспечивая инклюзивный опыт для всех пользователей.

Ключевые аспекты эффективного тестирования:
1. Использование различных методов и инструментов
2. Регулярное тестирование на протяжении всего цикла разработки
3. Вовлечение пользователей с ограниченными возможностями
4. Обучение команды принципам доступности
5. Интеграция проверок доступности в процесс разработки

Эти практики помогают создавать действительно доступные веб-ресурсы, которые работают для всех пользователей, независимо от их возможностей или используемых технологий.

## Следующие темы
- [[HTML/Доступность/Стандарты WCAG]]
- [[HTML/Доступность/ARIA]]
- [[HTML/Доступность/Лучшие практики]]

## Теги
#html #accessibility #accessibility-testing #web-development #frontend #a11y #screen-readers #keyboard-navigation #aria #wcag #accessibility-audit #accessibility-tools #axe-core #wave #lighthouse #accessibility-insights #pa11y #contrast-checker #form-accessibility #media-accessibility #tab-accessibility #modal-accessibility #document-structure #alternative-text #focus-indicators #automated-testing #manual-testing #inclusivity #universal-design #user-experience #cognitive-accessibility #motor-accessibility #visual-accessibility #auditory-accessibility
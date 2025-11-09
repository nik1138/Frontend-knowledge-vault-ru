# Лучшие практики HTML: семантика, доступность и поддержка

## Введение

HTML (HyperText Markup Language) является основой веб-разработки. Правильное использование HTML-элементов и атрибутов критически важно для создания доступных, семантически правильных и легко поддерживаемых веб-страниц. В этой статье мы рассмотрим лучшие практики написания HTML-кода.

## Семантическая разметка

### 1. Использование правильных элементов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Семантическая разметка</title>
</head>
<body>
    <!-- Шапка сайта -->
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

    <!-- Основное содержимое -->
    <main>
        <!-- Статья -->
        <article>
            <header>
                <h2>Заголовок статьи</h2>
                <time datetime="2023-10-15">15 октября 2023</time>
                <p>Автор: <address>Иван Иванов</address></p>
            </header>
            
            <section>
                <h3>Раздел статьи</h3>
                <p>Содержимое раздела...</p>
            </section>
            
            <section>
                <h3>Еще один раздел</h3>
                <p>Более содержимого...</p>
            </section>
            
            <footer>
                <p>Опубликовано: <time datetime="2023-10-15T10:30">15 октября 2023, 10:30</time></p>
            </footer>
        </article>

        <!-- Боковая панель -->
        <aside aria-label="Дополнительная информация">
            <h3>Рекомендуем</h3>
            <ul>
                <li><a href="/related1">Связанная статья 1</a></li>
                <li><a href="/related2">Связанная статья 2</a></li>
            </ul>
        </aside>
    </main>

    <!-- Подвал сайта -->
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
<html>
<head>
    <title>Правильная иерархия заголовков</title>
</head>
<body>
    <!-- Плохо: пропуск уровней -->
    <h1>Основной заголовок</h1>
    <h4>Подзаголовок</h4> <!-- Пропущены h2 и h3 -->
    <h5>Дочерний заголовок</h5>

    <!-- Хорошо: последовательная иерархия -->
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
</body>
</html>
```

### 3. Использование семантических элементов

```html
<!DOCTYPE html>
<html>
<head>
    <title>Семантические элементы</title>
</head>
<body>
    <!-- Контентная статья -->
    <article>
        <h2>Новость дня</h2>
        <p>Содержимое новости...</p>
        
        <!-- Комментарии к статье -->
        <section aria-label="Комментарии">
            <h3>Комментарии</h3>
            <article>
                <header>
                    <h4>Комментарий пользователя</h4>
                    <time datetime="2023-10-15T14:30">2 часа назад</time>
                </header>
                <p>Текст комментария...</p>
            </article>
        </section>
    </article>

    <!-- Определения -->
    <dl>
        <dt>HTML</dt>
        <dd>HyperText Markup Language - язык разметки гипертекста</dd>
        
        <dt>CSS</dt>
        <dd>Cascading Style Sheets - каскадные таблицы стилей</dd>
    </dl>

    <!-- Форматирование кода -->
    <pre><code>
function greet(name) {
    return `Привет, ${name}!`;
}
    </code></pre>

    <!-- Цитаты -->
    <blockquote cite="https://example.com">
        <p>Цитируемый текст</p>
        <footer>— <cite>Источник цитаты</cite></footer>
    </blockquote>

    <!-- Абзацы и форматирование -->
    <p>Это <strong>важный</strong> текст, а это <em>выделенное</em> слово.</p>
    <p>Цена: <s>1000 руб.</s> <mark>800 руб.</mark></p>
    <p>Версия: <code>v1.0.0</code>, пользователь: <kbd>Ctrl+C</kbd></p>
</body>
</html>
```

## Доступность (Accessibility)

### 1. ARIA атрибуты

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступность с ARIA</title>
</head>
<body>
    <!-- Роли и свойства ARIA -->
    <div role="banner">
        <h1>Шапка сайта</h1>
    </div>

    <nav role="navigation" aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
            <li><a href="/services">Услуги</a></li>
            <li><a href="/contact">Контакты</a></li>
        </ul>
    </nav>

    <main role="main" id="main-content">
        <h1>Основной контент</h1>
    </main>

    <!-- Интерактивные элементы -->
    <button 
        aria-expanded="false" 
        aria-controls="collapsible-content"
        onclick="toggleContent(this)">
        Показать больше
    </button>

    <div 
        id="collapsible-content" 
        role="region" 
        aria-labelledby="toggle-button" 
        hidden>
        <p>Дополнительный контент...</p>
    </div>

    <!-- Формы с доступностью -->
    <form>
        <div class="form-group">
            <label for="email">Email <span class="required" aria-label="обязательное поле">*</span></label>
            <input 
                type="email" 
                id="email" 
                name="email" 
                required
                aria-describedby="email-help email-error"
                autocomplete="email">
            <span id="email-help" class="form-hint">Введите действительный email</span>
            <span id="email-error" class="error-message" role="alert" aria-live="polite"></span>
        </div>
    </form>

    <!-- Живые области -->
    <div 
        aria-live="polite" 
        aria-atomic="true" 
        id="status-updates"
        class="sr-only">
        <!-- Обновления будут объявлены скринридером -->
    </div>

    <!-- Прогресс бар -->
    <div 
        role="progressbar" 
        aria-valuenow="50" 
        aria-valuemin="0" 
        aria-valuemax="100" 
        aria-valuetext="50% завершено"
        style="width: 50%;">
        <span>50%</span>
    </div>
</body>
</html>
```

### 2. Клавиатурная навигация

```html
<!DOCTYPE html>
<html>
<head>
    <title>Клавиатурная навигация</title>
    <style>
        /* Визуальная индикация фокуса */
        a:focus,
        button:focus,
        input:focus,
        select:focus,
        textarea:focus {
            outline: 2px solid #007bff;
            outline-offset: 2px;
        }
        
        /* Скрытие нативного фокуса только при использовании мыши */
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
        <a href="/contact">Контакты</a>
    </nav>

    <main>
        <h1>Основной контент</h1>
        <p>Текст страницы...</p>
        
        <!-- Пропуск навигации для клавиатурных пользователей -->
        <a href="#main-content" class="skip-link">Перейти к основному содержимому</a>
        
        <div id="main-content">
            <h2>Заголовок основного содержимого</h2>
            <p>Содержимое...</p>
        </div>
    </main>

    <!-- Кнопки с доступностью -->
    <button type="button" aria-pressed="false" onclick="toggleButton(this)">
        Переключатель
    </button>

    <!-- Сложные интерфейсы -->
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
            <p>Содержимое вкладки "Общие"</p>
        </div>
        <div 
            role="tabpanel" 
            id="panel2" 
            aria-labelledby="tab2" 
            hidden>
            <p>Содержимое вкладки "Дополнительно"</p>
        </div>
    </div>
</body>
</html>
```

### 3. Скрытый контент для скринридеров

```html
<!DOCTYPE html>
<html>
<head>
    <title>Скрытый контент для скринридеров</title>
    <style>
        /* Класс для скрытия элементов только для визуального отображения */
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
        
        /* Класс для скрытия элементов для всех, кроме скринридеров */
        .visually-hidden {
            position: absolute !important;
            height: 1px;
            width: 1px;
            overflow: hidden;
            clip: rect(1px, 1px, 1px, 1px);
        }
    </style>
</head>
<body>
    <!-- Визуально скрытый, но доступный для скринридеров -->
    <h2 class="sr-only">Список товаров</h2>
    
    <!-- Счетчик товаров для скринридеров -->
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
    
    <main id="main-content">
        <h1>Основной контент</h1>
        <p>Содержимое страницы...</p>
    </main>
</body>
</html>
```

## Оптимизация и производительность

### 1. Структура документа

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <!-- Обязательные мета-теги -->
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- Критические стили встроенные -->
    <style>
        /* Критические стили для выше-видимой части */
        body { margin: 0; font-family: system-ui, sans-serif; }
        .above-fold { display: block; }
    </style>
    
    <!-- Предзагрузка критических ресурсов -->
    <link rel="preload" href="critical.css" as="style">
    <link rel="preload" href="main.js" as="script">
    <link rel="preload" href="hero-image.jpg" as="image">
    
    <!-- Асинхронная загрузка основных стилей -->
    <link rel="stylesheet" href="styles.css" media="print" onload="this.media='all'">
    <noscript><link rel="stylesheet" href="styles.css"></noscript>
    
    <title>Оптимизированная страница</title>
    <meta name="description" content="Краткое описание страницы для SEO">
</head>
<body>
    <!-- Содержимое страницы -->
    <main>
        <h1>Заголовок страницы</h1>
        <p>Содержимое...</p>
    </main>
    
    <!-- Скрипты в конце body -->
    <script src="main.js" defer></script>
</body>
</html>
```

### 2. Оптимизация изображений

```html
<!DOCTYPE html>
<html>
<head>
    <title>Оптимизация изображений</title>
</head>
<body>
    <!-- Изображения с правильной семантикой -->
    <figure>
        <img 
            src="image.jpg" 
            srcset="image-small.jpg 480w, 
                    image-medium.jpg 800w, 
                    image-large.jpg 1200w"
            sizes="(max-width: 480px) 100vw, 
                   (max-width: 800px) 50vw, 
                   33vw"
            alt="Описание изображения">
        <figcaption>Подпись к изображению</figcaption>
    </figure>

    <!-- Современные форматы изображений -->
    <picture>
        <source srcset="image.webp" type="image/webp">
        <source srcset="image.avif" type="image/avif">
        <img 
            src="image.jpg" 
            alt="Альтернативное изображение"
            loading="lazy"
            decoding="async">
    </picture>

    <!-- Иконки -->
    <svg aria-hidden="true" width="24" height="24" focusable="false">
        <use href="#icon-name"></use>
    </svg>

    <!-- Изображения с lazy loading -->
    <img 
        src="placeholder.jpg" 
        data-src="actual-image.jpg" 
        alt="Описание"
        loading="lazy"
        class="lazy-image">
</body>
</html>
```

### 3. Оптимизация форм

```html
<!DOCTYPE html>
<html>
<head>
    <title>Оптимизированные формы</title>
</head>
<body>
    <form 
        action="/submit" 
        method="POST" 
        id="optimized-form"
        novalidate>
        
        <!-- Группировка связанных полей -->
        <fieldset>
            <legend>Контактная информация</legend>
            
            <div class="form-group">
                <label for="full-name">Полное имя</label>
                <input 
                    type="text" 
                    id="full-name" 
                    name="full-name" 
                    required
                    autocomplete="name"
                    aria-describedby="name-help">
                <div id="name-help" class="form-hint">Введите ваше полное имя</div>
            </div>
            
            <div class="form-row">
                <div class="form-group">
                    <label for="first-name">Имя</label>
                    <input 
                        type="text" 
                        id="first-name" 
                        name="first-name" 
                        required
                        autocomplete="given-name">
                </div>
                
                <div class="form-group">
                    <label for="last-name">Фамилия</label>
                    <input 
                        type="text" 
                        id="last-name" 
                        name="last-name" 
                        required
                        autocomplete="family-name">
                </div>
            </div>
            
            <div class="form-group">
                <label for="email">Email</label>
                <input 
                    type="email" 
                    id="email" 
                    name="email" 
                    required
                    autocomplete="email"
                    pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$">
            </div>
            
            <div class="form-group">
                <label for="phone">Телефон</label>
                <input 
                    type="tel" 
                    id="phone" 
                    name="phone" 
                    required
                    autocomplete="tel"
                    pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
                    placeholder="123-456-7890">
            </div>
        </fieldset>
        
        <!-- Выбор опций -->
        <fieldset>
            <legend>Предпочтения</legend>
            
            <div class="checkbox-group">
                <label>
                    <input type="checkbox" name="newsletter" value="yes">
                    <span>Подписка на рассылку</span>
                </label>
            </div>
            
            <div class="radio-group">
                <p>Предпочтительный способ связи:</p>
                <label>
                    <input type="radio" name="contact-method" value="email" checked>
                    <span>Email</span>
                </label>
                <label>
                    <input type="radio" name="contact-method" value="phone">
                    <span>Телефон</span>
                </label>
            </div>
        </fieldset>
        
        <button type="submit">Отправить</button>
    </form>
</body>
</html>
```

## Безопасность HTML

### 1. Защита от XSS

```html
<!DOCTYPE html>
<html>
<head>
    <title>Безопасность HTML</title>
    <!-- Content Security Policy -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self'; 
                   script-src 'self' 'unsafe-inline'; 
                   style-src 'self' 'unsafe-inline'; 
                   img-src 'self' data: https:; 
                   font-src 'self'; 
                   connect-src 'self';">
    
    <!-- Дополнительные заголовки безопасности -->
    <meta http-equiv="X-Content-Type-Options" content="nosniff">
    <meta http-equiv="X-Frame-Options" content="DENY">
    <meta http-equiv="X-XSS-Protection" content="1; mode=block">
</head>
<body>
    <!-- Правильная обработка пользовательского ввода на сервере -->
    <!-- Ниже показаны безопасные паттерны -->
    
    <!-- Безопасные формы -->
    <form method="POST" action="/submit">
        <!-- CSRF токен -->
        <input type="hidden" name="csrf_token" value="generated_token_here">
        
        <label for="comment">Комментарий:</label>
        <textarea 
            id="comment" 
            name="comment" 
            maxlength="1000"
            required></textarea>
        
        <button type="submit">Отправить</button>
    </form>
    
    <!-- Безопасные ссылки -->
    <a href="https://trusted-site.com" rel="noopener noreferrer">Доверенный сайт</a>
    
    <!-- Безопасные iframe -->
    <iframe 
        src="https://trusted-video.com" 
        sandbox="allow-scripts allow-same-origin"
        referrerpolicy="no-referrer"
        loading="lazy">
    </iframe>
</body>
</html>
```

### 2. Защита данных

```html
<!DOCTYPE html>
<html>
<head>
    <title>Защита данных</title>
</head>
<body>
    <!-- Правильное использование autocomplete -->
    <form>
        <!-- Личная информация -->
        <input type="text" name="full-name" autocomplete="name" required>
        <input type="email" name="email" autocomplete="email" required>
        <input type="tel" name="phone" autocomplete="tel" required>
        
        <!-- Адрес -->
        <input type="text" name="street" autocomplete="street-address" required>
        <input type="text" name="city" autocomplete="address-level2" required>
        <input type="text" name="state" autocomplete="address-level1" required>
        <input type="text" name="zip" autocomplete="postal-code" required>
        <input type="text" name="country" autocomplete="country" required>
        
        <!-- Платежная информация -->
        <input type="text" name="card-number" autocomplete="cc-number" required>
        <input type="text" name="card-name" autocomplete="cc-name" required>
        <input type="text" name="card-expiry" autocomplete="cc-exp" required>
        <input type="text" name="card-csc" autocomplete="cc-csc" required>
        
        <!-- Конфиденциальная информация -->
        <input type="password" name="password" autocomplete="new-password" required>
        <input type="password" name="confirm-password" autocomplete="new-password" required>
    </form>
    
    <!-- Защита от автозаполнения чувствительных полей -->
    <input type="text" name="security-answer" autocomplete="off" spellcheck="false">
</body>
</html>
```

## Лучшие практики для командной разработки

### 1. Соглашения по кодированию

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Соглашения по кодированию</title>
    <!-- Комментарии для сложных частей -->
    <!-- 
        Блок навигации
        Используется на всех страницах
        Обновляется через CMS
    -->
</head>
<body>
    <!-- Структура с понятной иерархией -->
    <header class="site-header" role="banner">
        <div class="container">
            <h1 class="site-title">
                <a href="/" class="site-title__link" rel="home">Название сайта</a>
            </h1>
            
            <nav class="site-navigation" role="navigation" aria-label="Основная навигация">
                <ul class="site-navigation__list">
                    <li class="site-navigation__item">
                        <a href="/" class="site-navigation__link">Главная</a>
                    </li>
                    <li class="site-navigation__item">
                        <a href="/about" class="site-navigation__link">О нас</a>
                    </li>
                </ul>
            </nav>
        </div>
    </header>
    
    <main class="site-main" id="main-content">
        <article class="content-article">
            <header class="content-article__header">
                <h1 class="content-article__title">Заголовок статьи</h1>
                <time class="content-article__date" datetime="2023-10-15">15 октября 2023</time>
            </header>
            
            <div class="content-article__body">
                <p>Содержимое статьи...</p>
            </div>
        </article>
    </main>
    
    <footer class="site-footer" role="contentinfo">
        <div class="container">
            <p class="site-footer__copyright">&copy; 2023 Название сайта. Все права защищены.</p>
        </div>
    </footer>
</body>
</html>
```

### 2. Тестирование и валидация

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Тестирование и валидация</title>
</head>
<body>
    <!-- Структура для тестирования доступности -->
    <div id="test-area">
        <!-- Тестирование форм -->
        <form id="test-form" novalidate>
            <div class="form-group">
                <label for="test-input">Тестовое поле</label>
                <input 
                    type="text" 
                    id="test-input" 
                    name="test-input"
                    required
                    aria-describedby="test-error">
                <div id="test-error" class="error-message" role="alert" aria-live="assertive"></div>
            </div>
            <button type="submit">Отправить</button>
        </form>
        
        <!-- Тестирование интерактивных элементов -->
        <div class="test-accordion" role="tablist">
            <div class="test-accordion__item">
                <button 
                    role="tab" 
                    aria-selected="true" 
                    aria-controls="panel-1" 
                    id="tab-1">
                    Вкладка 1
                </button>
                <div 
                    role="tabpanel" 
                    id="panel-1" 
                    aria-labelledby="tab-1">
                    <p>Содержимое вкладки 1</p>
                </div>
            </div>
        </div>
    </div>
    
    <script>
        // Тестирование функциональности
        document.getElementById('test-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const input = document.getElementById('test-input');
            const error = document.getElementById('test-error');
            
            if (!input.value.trim()) {
                error.textContent = 'Поле обязательно для заполнения';
                input.setAttribute('aria-invalid', 'true');
            } else {
                error.textContent = '';
                input.setAttribute('aria-invalid', 'false');
                console.log('Форма валидна');
            }
        });
    </script>
</body>
</html>
```

## Примеры из промышленной разработки

### 1. Адаптивная навигация

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная навигация</title>
    <style>
        .nav-toggle {
            display: none;
            background: none;
            border: none;
            font-size: 1.5rem;
            cursor: pointer;
        }
        
        .nav-menu {
            list-style: none;
            margin: 0;
            padding: 0;
        }
        
        @media (max-width: 767px) {
            .nav-toggle {
                display: block;
            }
            
            .nav-menu {
                position: fixed;
                top: 0;
                left: -100%;
                width: 100%;
                height: 100vh;
                background: white;
                transition: left 0.3s ease;
            }
            
            .nav-menu.active {
                left: 0;
            }
        }
        
        @media (min-width: 768px) {
            .nav-menu {
                display: flex;
                justify-content: space-between;
            }
        }
    </style>
</head>
<body>
    <nav class="main-navigation" aria-label="Основная навигация">
        <div class="container">
            <div class="nav-brand">
                <a href="/" aria-label="Перейти на главную">
                    <span class="logo">Логотип</span>
                </a>
            </div>
            
            <button 
                class="nav-toggle" 
                aria-expanded="false" 
                aria-controls="nav-menu"
                aria-label="Открыть/закрыть меню">
                ☰
            </button>
            
            <ul id="nav-menu" class="nav-menu">
                <li><a href="/" aria-current="page">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/services">Услуги</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </div>
    </nav>
    
    <script>
        document.querySelector('.nav-toggle').addEventListener('click', function() {
            const menu = document.getElementById('nav-menu');
            const isExpanded = this.getAttribute('aria-expanded') === 'true';
            
            this.setAttribute('aria-expanded', !isExpanded);
            menu.classList.toggle('active');
        });
    </script>
</body>
</html>
```

### 2. Карточка продукта

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Карточка продукта</title>
    <style>
        .product-card {
            border: 1px solid #e0e0e0;
            border-radius: 8px;
            overflow: hidden;
            transition: transform 0.2s ease, box-shadow 0.2s ease;
        }
        
        .product-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        
        .product-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
        }
        
        .product-info {
            padding: 1rem;
        }
        
        .product-title {
            font-size: 1.1rem;
            margin: 0 0 0.5rem 0;
        }
        
        .product-price {
            font-size: 1.25rem;
            font-weight: bold;
            color: #e53e3e;
            margin: 0.5rem 0;
        }
        
        .product-rating {
            color: #e6b400;
            margin: 0.5rem 0;
        }
        
        .product-actions {
            margin-top: 1rem;
        }
    </style>
</head>
<body>
    <article class="product-card" itemscope itemtype="http://schema.org/Product">
        <figure class="product-image-container">
            <img 
                src="product-image.jpg" 
                alt="Описание продукта" 
                class="product-image"
                itemprop="image"
                loading="lazy">
        </figure>
        
        <div class="product-info">
            <h3 class="product-title" itemprop="name">Название продукта</h3>
            
            <div class="product-rating" itemprop="aggregateRating" itemscope itemtype="http://schema.org/AggregateRating">
                <span itemprop="ratingValue">4.5</span>
                <span>из</span>
                <span itemprop="bestRating">5</span>
                <span>(</span>
                <span itemprop="reviewCount">128</span>
                <span>отзывов)</span>
            </div>
            
            <div class="product-price" itemprop="offers" itemscope itemtype="http://schema.org/Offer">
                <meta itemprop="priceCurrency" content="RUB">
                <span itemprop="price">2 990</span> ₽
            </div>
            
            <div class="product-description" itemprop="description">
                <p>Краткое описание продукта...</p>
            </div>
            
            <div class="product-actions">
                <button class="btn btn-primary" type="button" aria-label="Добавить в корзину">
                    В корзину
                </button>
                <button class="btn btn-secondary" type="button" aria-label="Добавить в избранное">
                    ❤
                </button>
            </div>
        </div>
    </article>
</body>
</html>
```

### 3. Комплексная форма регистрации

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Форма регистрации</title>
</head>
<body>
    <main class="container">
        <article aria-labelledby="form-title">
            <header>
                <h1 id="form-title">Регистрация нового пользователя</h1>
                <p>Создайте аккаунт, чтобы получить доступ ко всем возможностям</p>
            </header>
            
            <form 
                id="registration-form" 
                action="/register" 
                method="POST" 
                novalidate
                aria-describedby="form-description">
                
                <div id="form-description" class="sr-only">
                    Заполните все обязательные поля для регистрации
                </div>
                
                <!-- CSRF токен -->
                <input type="hidden" name="csrf_token" value="token_here">
                
                <fieldset aria-labelledby="personal-info">
                    <legend id="personal-info">Личная информация</legend>
                    
                    <div class="form-row">
                        <div class="form-group">
                            <label for="first-name">Имя <abbr title="обязательное поле" aria-label="обязательное поле">*</abbr></label>
                            <input 
                                type="text" 
                                id="first-name" 
                                name="first_name" 
                                required
                                autocomplete="given-name"
                                aria-describedby="first-name-error"
                                aria-invalid="false">
                            <div id="first-name-error" class="error-message" role="alert" aria-live="assertive"></div>
                        </div>
                        
                        <div class="form-group">
                            <label for="last-name">Фамилия <abbr title="обязательное поле" aria-label="обязательное поле">*</abbr></label>
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
                    </div>
                    
                    <div class="form-group">
                        <label for="birth-date">Дата рождения</label>
                        <input 
                            type="date" 
                            id="birth-date" 
                            name="birth_date"
                            min="1900-01-01"
                            max="2023-12-31"
                            autocomplete="bday">
                    </div>
                </fieldset>
                
                <fieldset aria-labelledby="contact-info">
                    <legend id="contact-info">Контактная информация</legend>
                    
                    <div class="form-group">
                        <label for="email">Email <abbr title="обязательное поле" aria-label="обязательное поле">*</abbr></label>
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
                            pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
                            placeholder="123-456-7890">
                    </div>
                </fieldset>
                
                <fieldset aria-labelledby="account-info">
                    <legend id="account-info">Информация аккаунта</legend>
                    
                    <div class="form-group">
                        <label for="username">Имя пользователя <abbr title="обязательное поле" aria-label="обязательное поле">*</abbr></label>
                        <input 
                            type="text" 
                            id="username" 
                            name="username" 
                            required
                            minlength="3"
                            maxlength="30"
                            pattern="[a-zA-Z0-9_]+"
                            autocomplete="username"
                            aria-describedby="username-help username-error"
                            aria-invalid="false">
                        <div id="username-help" class="form-hint">Только буквы, цифры и подчеркивание, от 3 до 30 символов</div>
                        <div id="username-error" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                    
                    <div class="form-group">
                        <label for="password">Пароль <abbr title="обязательное поле" aria-label="обязательное поле">*</abbr></label>
                        <input 
                            type="password" 
                            id="password" 
                            name="password" 
                            required
                            minlength="8"
                            autocomplete="new-password"
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
                    
                    <div class="form-group">
                        <label for="confirm-password">Подтвердите пароль <abbr title="обязательное поле" aria-label="обязательное поле">*</abbr></label>
                        <input 
                            type="password" 
                            id="confirm-password" 
                            name="confirm_password" 
                            required
                            autocomplete="new-password"
                            aria-describedby="confirm-password-error"
                            aria-invalid="false">
                        <div id="confirm-password-error" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                </fieldset>
                
                <fieldset>
                    <legend>Дополнительные настройки</legend>
                    
                    <div class="checkbox-group">
                        <label>
                            <input type="checkbox" name="newsletter" value="yes">
                            <span>Подписка на рассылку</span>
                        </label>
                    </div>
                    
                    <div class="checkbox-group">
                        <label>
                            <input type="checkbox" name="terms" value="accepted" required>
                            <span>Я согласен с <a href="/terms">условиями использования</a> и <a href="/privacy">политикой конфиденциальности</a> <abbr title="обязательное поле" aria-label="обязательное поле">*</abbr></span>
                        </label>
                        <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
                    </div>
                </fieldset>
                
                <div class="form-actions">
                    <button type="submit" class="btn btn-primary">Зарегистрироваться</button>
                    <button type="reset" class="btn btn-secondary">Очистить</button>
                </div>
            </form>
        </article>
    </main>
    
    <script>
        // Пример клиентской валидации
        document.getElementById('registration-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            let isValid = true;
            const requiredFields = this.querySelectorAll('[required]');
            
            requiredFields.forEach(field => {
                const errorElement = document.getElementById(field.name + '-error');
                
                if (!field.value.trim()) {
                    field.setAttribute('aria-invalid', 'true');
                    if (errorElement) {
                        errorElement.textContent = 'Это поле обязательно для заполнения';
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
                console.log('Форма валидна, отправляем данные...');
                // Тут будет отправка формы
            }
        });
    </script>
</body>
</html>
```

## Ссылки на связанные темы
- [[html/accessibility]] - Доступность HTML
- [[html/semantics]] - Семантическая верстка
- [[html/security]] - Безопасность HTML
- [[css/best-practices]] - Лучшие практики CSS

#programming #html #best-practices #accessibility #semantics #validation
# Продвинутые возможности HTML: современные элементы и атрибуты

## Введение

Современный HTML предоставляет богатый набор элементов и атрибутов, которые позволяют создавать семантически правильные, доступные и функциональные веб-страницы. В этой статье мы рассмотрим продвинутые возможности HTML, включая новые элементы, атрибуты и возможности, которые значительно улучшают пользовательский опыт и доступность.

## Новые элементы HTML5 и более поздних версий

### Семантические элементы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Семантическая структура</title>
</head>
<body>
    <!-- Шапка сайта -->
    <header>
        <h1>Название сайта</h1>
        <nav>
            <ul>
                <li><a href="#home">Главная</a></li>
                <li><a href="#about">О нас</a></li>
                <li><a href="#contact">Контакты</a></li>
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
            </header>
            <section>
                <h3>Раздел статьи</h3>
                <p>Содержимое раздела...</p>
            </section>
            <footer>
                <p>Автор: <address>Иван Иванов</address></p>
            </footer>
        </article>

        <!-- Боковая панель -->
        <aside>
            <h3>Реклама</h3>
            <p>Спонсорский контент</p>
        </aside>
    </main>

    <!-- Подвал сайта -->
    <footer>
        <p>&copy; 2023 Название сайта. Все права защищены.</p>
        <nav>
            <ul>
                <li><a href="#privacy">Политика конфиденциальности</a></li>
                <li><a href="#terms">Условия использования</a></li>
            </ul>
        </nav>
    </footer>
</body>
</html>
```

### Интерактивные элементы

```html
<!-- Детали и сворачиваемый контент -->
<details>
    <summary>Часто задаваемые вопросы</summary>
    <p>Ответ на вопрос 1...</p>
    <p>Ответ на вопрос 2...</p>
</details>

<!-- Диалоговое окно -->
<dialog id="modal-dialog">
    <h3>Заголовок модального окна</h3>
    <p>Содержимое модального окна</p>
    <button onclick="document.getElementById('modal-dialog').close()">Закрыть</button>
</dialog>

<button onclick="document.getElementById('modal-dialog').showModal()">
    Открыть модальное окно
</button>

<!-- Прогресс и метрики -->
<div>
    <label for="progress">Прогресс загрузки:</label>
    <progress id="progress" value="70" max="100">70%</progress>
</div>

<div>
    <label for="meter">Уровень заполнения:</label>
    <meter id="meter" value="0.6" min="0" max="1" low="0.3" high="0.8" optimum="0.5">
        60%
    </meter>
</div>
```

### Мультимедийные элементы

```html
<!-- Аудио -->
<audio controls preload="metadata">
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    Ваш браузер не поддерживает аудио элемент.
</audio>

<!-- Видео -->
<video controls width="640" height="360" preload="metadata">
    <source src="video.mp4" type="video/mp4">
    <source src="video.webm" type="video/webm">
    <track kind="subtitles" src="subtitles.vtt" srclang="ru" label="Русский">
    <track kind="captions" src="captions.vtt" srclang="en" label="English">
    Ваш браузер не поддерживает видео элемент.
</video>

<!-- Канвас -->
<canvas id="drawing-canvas" width="400" height="300">
    Ваш браузер не поддерживает элемент canvas.
</canvas>

<!-- Изображение с поддержкой разных форматов -->
<picture>
    <source media="(min-width: 800px)" srcset="large.webp" type="image/webp">
    <source media="(min-width: 800px)" srcset="large.jpg" type="image/jpeg">
    <source srcset="small.webp" type="image/webp">
    <img src="small.jpg" alt="Описание изображения" loading="lazy">
</picture>
```

## Продвинутые атрибуты и возможности

### Атрибуты доступности (ARIA)

```html
<!-- Роли и свойства ARIA -->
<div role="banner">
    <h1>Шапка сайта</h1>
</div>

<nav role="navigation" aria-label="Основная навигация">
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
    </ul>
</nav>

<main role="main" id="main-content">
    <h2>Основной контент</h2>
</main>

<!-- Интерактивные элементы с ARIA -->
<button 
    aria-expanded="false" 
    aria-controls="collapsible-content"
    onclick="toggleContent()">
    Показать больше
</button>

<div id="collapsible-content" role="region" aria-labelledby="toggle-button" hidden>
    <p>Дополнительный контент...</p>
</div>

<!-- Состояния и свойства -->
<div 
    role="progressbar" 
    aria-valuenow="50" 
    aria-valuemin="0" 
    aria-valuemax="100" 
    aria-label="Прогресс загрузки">
    <div style="width: 50%;"></div>
</div>

<!-- Живые области -->
<div 
    aria-live="polite" 
    aria-atomic="true" 
    id="status-updates">
    <!-- Обновления будут объявлены скринридером -->
</div>
```

### Атрибуты форм

```html
<form>
    <!-- Автозаполнение -->
    <input type="text" name="full-name" autocomplete="name" required>
    <input type="email" name="email" autocomplete="email" required>
    <input type="tel" name="phone" autocomplete="tel" required>
    
    <!-- Адрес -->
    <input type="text" name="street" autocomplete="street-address" required>
    <input type="text" name="city" autocomplete="address-level2" required>
    <input type="text" name="zip" autocomplete="postal-code" required>
    
    <!-- Платежная информация -->
    <input type="text" name="card-number" autocomplete="cc-number" required>
    <input type="text" name="card-name" autocomplete="cc-name" required>
    <input type="text" name="card-expiry" autocomplete="cc-exp" required>
    <input type="text" name="card-csc" autocomplete="cc-csc" required>
    
    <!-- Даты и время -->
    <input type="date" name="birth-date" min="1900-01-01" max="2023-12-31">
    <input type="time" name="appointment-time" min="09:00" max="18:00">
    <input type="datetime-local" name="event-date">
    
    <!-- Числовые значения -->
    <input type="number" name="age" min="18" max="120" step="1">
    <input type="range" name="satisfaction" min="1" max="10" value="5">
    
    <!-- Файлы -->
    <input type="file" name="avatar" accept="image/*" capture="environment">
    <input type="file" name="documents" accept=".pdf,.doc,.docx" multiple>
    
    <!-- Поиск -->
    <input type="search" name="query" results="5" autosave="search-queries">
    
    <!-- Телефон -->
    <input type="tel" name="phone" pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}" placeholder="123-456-7890">
    
    <!-- Цвет -->
    <input type="color" name="favorite-color" value="#3498db">
    
    <!-- Списки автодополнения -->
    <input type="text" name="country" list="countries" autocomplete="country-name">
    <datalist id="countries">
        <option value="Россия">
        <option value="США">
        <option value="Канада">
        <option value="Германия">
        <option value="Франция">
    </datalist>
    
    <button type="submit">Отправить</button>
</form>
```

### Контентные атрибуты

```html
<!-- Редактируемый контент -->
<div contenteditable="true" spellcheck="true">
    Этот текст можно редактировать
</div>

<!-- Перетаскивание -->
<div draggable="true" ondragstart="drag(event)">
    Перетаскиваемый элемент
</div>

<!-- Контекстное меню -->
<div contextmenu="custom-menu">
    Правый клик вызовет кастомное меню
</div>

<menu type="context" id="custom-menu">
    <menuitem label="Копировать" onclick="copyText()"></menuitem>
    <menuitem label="Вставить" onclick="pasteText()"></menuitem>
    <menuitem label="Удалить" onclick="deleteText()"></menuitem>
</menu>

<!-- Скрытие/показ элементов -->
<div hidden>Скрытый элемент</div>
<div aria-hidden="true">Скрытый для скринридеров</div>
```

## Современные возможности HTML

### Web Components

```html
<!-- Определение кастомного элемента -->
<script>
class UserCard extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
    }
    
    connectedCallback() {
        const name = this.getAttribute('name');
        const email = this.getAttribute('email');
        
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    border: 1px solid #ccc;
                    border-radius: 8px;
                    padding: 16px;
                    margin: 8px;
                    font-family: Arial, sans-serif;
                }
                
                .card-header {
                    font-weight: bold;
                    margin-bottom: 8px;
                }
                
                .card-email {
                    color: #666;
                }
            </style>
            <div class="card-header">${name}</div>
            <div class="card-email">${email}</div>
        `;
    }
    
    static get observedAttributes() {
        return ['name', 'email'];
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        // Обновление при изменении атрибутов
        this.connectedCallback();
    }
}

customElements.define('user-card', UserCard);
</script>

<!-- Использование кастомного элемента -->
<user-card name="Иван Иванов" email="ivan@example.com"></user-card>
```

### Template элемент

```html
<!-- Шаблон для динамического контента -->
<template id="user-template">
    <div class="user-card">
        <h3 class="user-name"></h3>
        <p class="user-email"></p>
        <button class="user-button">Подробнее</button>
    </div>
</template>

<div id="users-container"></div>

<script>
function createUserCard(user) {
    const template = document.getElementById('user-template');
    const clone = template.content.cloneNode(true);
    
    clone.querySelector('.user-name').textContent = user.name;
    clone.querySelector('.user-email').textContent = user.email;
    clone.querySelector('.user-button').addEventListener('click', () => {
        console.log('Пользователь:', user);
    });
    
    document.getElementById('users-container').appendChild(clone);
}

// Использование
createUserCard({ name: 'Иван Иванов', email: 'ivan@example.com' });
</script>
```

### Custom Elements с Shadow DOM

```html
<script>
class ImageGallery extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        // Стили галереи
        const style = document.createElement('style');
        style.textContent = `
            :host {
                display: block;
                max-width: 800px;
                margin: 0 auto;
            }
            
            .gallery {
                display: grid;
                grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
                gap: 16px;
            }
            
            .gallery img {
                width: 100%;
                height: 200px;
                object-fit: cover;
                border-radius: 8px;
                cursor: pointer;
                transition: transform 0.2s;
            }
            
            .gallery img:hover {
                transform: scale(1.05);
            }
        `;
        
        this.shadowRoot.appendChild(style);
    }
    
    connectedCallback() {
        const images = this.getAttribute('images')?.split(',') || [];
        this.render(images);
    }
    
    render(images) {
        const gallery = document.createElement('div');
        gallery.className = 'gallery';
        
        images.forEach(src => {
            const img = document.createElement('img');
            img.src = src.trim();
            img.alt = 'Галерея изображений';
            img.addEventListener('click', () => this.showLightbox(src.trim()));
            gallery.appendChild(img);
        });
        
        this.shadowRoot.appendChild(gallery);
    }
    
    showLightbox(src) {
        // Логика показа лайтбокса
        console.log('Показываем изображение:', src);
    }
}

customElements.define('image-gallery', ImageGallery);
</script>

<!-- Использование -->
<image-gallery images="image1.jpg, image2.jpg, image3.jpg"></image-gallery>
```

## Продвинутые формы

### Форма с валидацией и доступностью

```html
<form id="advanced-form" novalidate>
    <fieldset>
        <legend>Информация пользователя</legend>
        
        <div class="form-group">
            <label for="first-name">Имя <span class="required" aria-label="обязательное поле">*</span></label>
            <input 
                type="text" 
                id="first-name" 
                name="first-name"
                required
                aria-describedby="first-name-help first-name-error"
                autocomplete="given-name">
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
                aria-describedby="last-name-help last-name-error"
                autocomplete="family-name">
            <span id="last-name-help" class="form-hint">Введите вашу фамилию</span>
            <span id="last-name-error" class="error-message" role="alert" aria-live="polite"></span>
        </div>
        
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
    </fieldset>
    
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
        </div>
    </fieldset>
    
    <button type="submit">Отправить</button>
</form>
```

### Многошаговая форма

```html
<form id="multi-step-form">
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

## Современные атрибуты безопасности

### Content Security Policy через meta-тег

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' 'unsafe-eval'; 
               style-src 'self' 'unsafe-inline'; 
               img-src 'self' data: https:; 
               font-src 'self'; 
               connect-src 'self'; 
               frame-ancestors 'none'; 
               base-uri 'self';">
```

### Безопасные атрибуты iframe

```html
<!-- Безопасное встраивание внешнего контента -->
<iframe 
    src="https://example.com" 
    sandbox="allow-scripts allow-same-origin allow-forms"
    referrerpolicy="no-referrer"
    loading="lazy"
    width="600" 
    height="400">
</iframe>
```

### Referrer Policy

```html
<!-- Контроль реферера -->
<meta name="referrer" content="no-referrer">
<!-- Или для конкретных ссылок -->
<a href="https://example.com" referrerpolicy="no-referrer">Ссылка без реферера</a>
```

## Примеры из промышленной разработки

### Интерактивная карта сайта

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Интерактивная карта сайта</title>
    <style>
        .sitemap {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 2rem;
            padding: 2rem;
        }
        
        .sitemap-section {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 1rem;
        }
        
        .sitemap-section h3 {
            margin-top: 0;
            cursor: pointer;
            user-select: none;
        }
        
        .sitemap-section[aria-expanded="false"] .sitemap-links {
            display: none;
        }
        
        .sitemap-links {
            list-style: none;
            padding: 0;
        }
        
        .sitemap-links li {
            margin-bottom: 0.5rem;
        }
    </style>
</head>
<body>
    <header>
        <h1>Карта сайта</h1>
        <p>Навигация по разделам сайта</p>
    </header>
    
    <nav role="navigation" aria-label="Карта сайта">
        <div class="sitemap">
            <section class="sitemap-section" aria-expanded="true">
                <h3 onclick="toggleSection(this)" role="button" tabindex="0" aria-expanded="true">
                    Главная
                </h3>
                <ul class="sitemap-links">
                    <li><a href="/">Главная страница</a></li>
                    <li><a href="/about">О компании</a></li>
                    <li><a href="/news">Новости</a></li>
                </ul>
            </section>
            
            <section class="sitemap-section" aria-expanded="true">
                <h3 onclick="toggleSection(this)" role="button" tabindex="0" aria-expanded="true">
                    Услуги
                </h3>
                <ul class="sitemap-links">
                    <li><a href="/services/web">Веб-разработка</a></li>
                    <li><a href="/services/mobile">Мобильные приложения</a></li>
                    <li><a href="/services/consulting">Консалтинг</a></li>
                </ul>
            </section>
            
            <section class="sitemap-section" aria-expanded="true">
                <h3 onclick="toggleSection(this)" role="button" tabindex="0" aria-expanded="true">
                    Поддержка
                </h3>
                <ul class="sitemap-links">
                    <li><a href="/help">Помощь</a></li>
                    <li><a href="/faq">Частые вопросы</a></li>
                    <li><a href="/contact">Контакты</a></li>
                </ul>
            </section>
        </div>
    </nav>
    
    <script>
        function toggleSection(element) {
            const section = element.parentElement;
            const isExpanded = section.getAttribute('aria-expanded') === 'true';
            
            section.setAttribute('aria-expanded', !isExpanded);
            element.setAttribute('aria-expanded', !isExpanded);
        }
        
        // Добавляем обработчик клавиатуры
        document.querySelectorAll('.sitemap-section h3').forEach(header => {
            header.addEventListener('keydown', (e) => {
                if (e.key === 'Enter' || e.key === ' ') {
                    e.preventDefault();
                    toggleSection(header);
                }
            });
        });
    </script>
</body>
</html>
```

### Продвинутый фильтр контента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Продвинутый фильтр контента</title>
    <style>
        .filter-container {
            display: flex;
            flex-wrap: wrap;
            gap: 1rem;
            padding: 1rem;
            background: #f5f5f5;
            border-radius: 8px;
            margin-bottom: 2rem;
        }
        
        .filter-group {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
        }
        
        .filter-group label {
            font-weight: bold;
        }
        
        .content-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 1.5rem;
        }
        
        .content-item {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 1rem;
            transition: transform 0.2s;
        }
        
        .content-item.hidden {
            display: none;
        }
        
        .content-item:hover {
            transform: translateY(-4px);
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <h1>Контент с фильтрацией</h1>
    
    <div class="filter-container">
        <div class="filter-group">
            <label for="category-filter">Категория:</label>
            <select id="category-filter" multiple size="3">
                <option value="web">Веб-разработка</option>
                <option value="mobile">Мобильные приложения</option>
                <option value="design">Дизайн</option>
                <option value="marketing">Маркетинг</option>
            </select>
        </div>
        
        <div class="filter-group">
            <label for="level-filter">Уровень сложности:</label>
            <select id="level-filter">
                <option value="">Все уровни</option>
                <option value="beginner">Начинающий</option>
                <option value="intermediate">Средний</option>
                <option value="advanced">Продвинутый</option>
            </select>
        </div>
        
        <div class="filter-group">
            <label for="search-filter">Поиск:</label>
            <input type="search" id="search-filter" placeholder="Поиск по названию...">
        </div>
        
        <div class="filter-group">
            <label for="date-filter">Дата:</label>
            <input type="date" id="date-filter">
        </div>
    </div>
    
    <div class="content-grid" id="content-grid">
        <article class="content-item" data-category="web" data-level="beginner" data-date="2023-10-01">
            <h3>Основы HTML</h3>
            <p>Изучите основы HTML для создания веб-страниц</p>
            <time datetime="2023-10-01">1 октября 2023</time>
        </article>
        
        <article class="content-item" data-category="web" data-level="intermediate" data-date="2023-10-05">
            <h3>Продвинутый CSS</h3>
            <p>Изучите современные возможности CSS</p>
            <time datetime="2023-10-05">5 октября 2023</time>
        </article>
        
        <article class="content-item" data-category="mobile" data-level="advanced" data-date="2023-10-10">
            <h3>React Native</h3>
            <p>Создание мобильных приложений с React Native</p>
            <time datetime="2023-10-10">10 октября 2023</time>
        </article>
        
        <article class="content-item" data-category="design" data-level="intermediate" data-date="2023-10-15">
            <h3>UX/UI дизайн</h3>
            <p>Принципы пользовательского опыта</p>
            <time datetime="2023-10-15">15 октября 2023</time>
        </article>
    </div>
    
    <script>
        class ContentFilter {
            constructor() {
                this.items = document.querySelectorAll('.content-item');
                this.init();
            }
            
            init() {
                document.getElementById('category-filter').addEventListener('change', () => this.filterContent());
                document.getElementById('level-filter').addEventListener('change', () => this.filterContent());
                document.getElementById('search-filter').addEventListener('input', () => this.filterContent());
                document.getElementById('date-filter').addEventListener('change', () => this.filterContent());
            }
            
            filterContent() {
                const categoryFilter = Array.from(document.querySelectorAll('#category-filter option:checked')).map(option => option.value);
                const levelFilter = document.getElementById('level-filter').value;
                const searchFilter = document.getElementById('search-filter').value.toLowerCase();
                const dateFilter = document.getElementById('date-filter').value;
                
                this.items.forEach(item => {
                    const category = item.dataset.category;
                    const level = item.dataset.level;
                    const title = item.querySelector('h3').textContent.toLowerCase();
                    const date = item.dataset.date;
                    
                    const matchesCategory = categoryFilter.length === 0 || categoryFilter.includes(category);
                    const matchesLevel = !levelFilter || level === levelFilter;
                    const matchesSearch = !searchFilter || title.includes(searchFilter);
                    const matchesDate = !dateFilter || date >= dateFilter;
                    
                    if (matchesCategory && matchesLevel && matchesSearch && matchesDate) {
                        item.classList.remove('hidden');
                    } else {
                        item.classList.add('hidden');
                    }
                });
            }
        }
        
        // Инициализация фильтра
        document.addEventListener('DOMContentLoaded', () => {
            new ContentFilter();
        });
    </script>
</body>
</html>
```

## Ссылки на связанные темы
- [[html/semantics]] - Семантическая верстка
- [[html/accessibility]] - Доступность HTML
- [[html/forms]] - HTML формы
- [[css/styling/components]] - Компонентная стилизация

#programming #html #advanced #web-components #accessibility #forms #modern-html
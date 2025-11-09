# CSS Адаптивный дизайн

Адаптивный дизайн (Responsive Design) — это подход к веб-дизайну, который позволяет веб-страницам адаптироваться к различным размерам экранов и устройствам. Он обеспечивает оптимальный пользовательский опыт на всех устройствах: от мобильных телефонов до настольных компьютеров.

## Основы адаптивного дизайна

### Viewport meta tag

Первый и самый важный шаг — установка мета-тега viewport:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивный дизайн</title>
    <style>
        /* Базовые стили */
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
            line-height: 1.6;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 15px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Адаптивный веб-сайт</h1>
        <p>Этот сайт адаптируется под различные размеры экранов.</p>
    </div>
</body>
</html>
```

### Основные принципы адаптивности:

1. **Гибкие сетки** - использование относительных единиц измерения
2. **Гибкие изображения** - изображения адаптируются к контейнеру
3. **Медиа-запросы** - изменение стилей в зависимости от характеристик устройства

## Гибкие сетки

### Использование процентов

```css
/* Простая гибкая сетка */
.container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
}

.grid-row {
    display: flex;
    flex-wrap: wrap;
    margin: 0 -10px;
}

.grid-col {
    padding: 0 10px;
    margin-bottom: 20px;
}

.col-1 { width: 8.333333%; }
.col-2 { width: 16.666667%; }
.col-3 { width: 25%; }
.col-4 { width: 33.333333%; }
.col-5 { width: 41.666667%; }
.col-6 { width: 50%; }
.col-7 { width: 58.333333%; }
.col-8 { width: 66.666667%; }
.col-9 { width: 75%; }
.col-10 { width: 83.333333%; }
.col-11 { width: 91.666667%; }
.col-12 { width: 100%; }
```

### Использование Flexbox

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox сетка</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
        }

        .flex-container {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
        }

        .flex-item {
            flex: 1 1 300px; /* grow, shrink, basis */
            padding: 20px;
            background-color: #f0f0f0;
            border-radius: 8px;
        }

        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .flex-container {
                flex-direction: column;
            }

            .flex-item {
                flex: 1 1 auto;
            }
        }
    </style>
</head>
<body>
    <div class="flex-container">
        <div class="flex-item">Колонка 1</div>
        <div class="flex-item">Колонка 2</div>
        <div class="flex-item">Колонка 3</div>
    </div>
</body>
</html>
```

### Использование CSS Grid

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Grid адаптивность</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
        }

        .grid-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
        }

        .grid-item {
            padding: 20px;
            background-color: #e0e0e0;
            border-radius: 8px;
        }

        /* Адаптация для планшетов */
        @media (max-width: 1024px) {
            .grid-container {
                grid-template-columns: repeat(2, 1fr);
            }
        }

        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .grid-container {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="grid-container">
        <div class="grid-item">Элемент 1</div>
        <div class="grid-item">Элемент 2</div>
        <div class="grid-item">Элемент 3</div>
        <div class="grid-item">Элемент 4</div>
    </div>
</body>
</html>
```

## Гибкие изображения

### Основные техники

```css
/* Основные стили для адаптивных изображений */
img {
    max-width: 100%;
    height: auto;
    display: block;
}

/* Для изображений с фиксированным соотношением сторон */
.aspect-ratio-box {
    position: relative;
    width: 100%;
    height: 0;
    padding-bottom: 56.25%; /* 16:9 соотношение сторон */
}

.aspect-ratio-content {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}

/* Изображения с различными размерами для разных устройств */
.adaptive-image {
    width: 100%;
    height: auto;
    object-fit: cover; /* Сохраняет пропорции изображения */
}
```

### Использование picture элемента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивные изображения</title>
</head>
<body>
    <h1>Адаптивные изображения</h1>

    <!-- Использование picture для разных форматов -->
    <picture>
        <source media="(min-width: 1200px)" srcset="large.avif" type="image/avif">
        <source media="(min-width: 768px)" srcset="medium.avif" type="image/avif">
        <source media="(min-width: 1200px)" srcset="large.webp" type="image/webp">
        <source media="(min-width: 768px)" srcset="medium.webp" type="image/webp">
        <source srcset="small.jpg" type="image/jpeg">
        <img src="small.jpg" 
             alt="Описание изображения" 
             loading="lazy"
             width="800" 
             height="600">
    </picture>

    <!-- Использование srcset для разных разрешений -->
    <img src="image-800.jpg"
         srcset="image-400.jpg 400w,
                 image-800.jpg 800w,
                 image-1200.jpg 1200w"
         sizes="(max-width: 600px) 100vw,
                (max-width: 1000px) 50vw,
                33vw"
         alt="Адаптивное изображение"
         loading="lazy">

    <!-- Фоновое изображение с адаптацией -->
    <div class="hero-section" style="background-image: url('hero-mobile.jpg');">
        <h2>Заголовок</h2>
        <p>Содержимое</p>
    </div>

    <style>
        .hero-section {
            background-size: cover;
            background-position: center;
            padding: 60px 20px;
            text-align: center;
            color: white;
        }

        /* Адаптация фона для разных размеров экрана */
        @media (min-width: 768px) {
            .hero-section {
                background-image: url('hero-tablet.jpg');
            }
        }

        @media (min-width: 1200px) {
            .hero-section {
                background-image: url('hero-desktop.jpg');
            }
        }
    </style>
</body>
</html>
```

## Медиа-запросы

### Основные медиа-типы и условия

```css
/* Основные медиа-типы */
@media screen { ... } /* Экран */
@media print { ... }  /* Печать */
@media speech { ... } /* Голосовой синтез */

/* Ширина экрана */
@media (max-width: 768px) { ... }   /* Мобильные устройства */
@media (min-width: 769px) { ... }   /* Планшеты и выше */
@media (min-width: 1024px) { ... }  /* Планшеты в альбомной ориентации */
@media (min-width: 1200px) { ... }  /* Настольные компьютеры */

/* Высота экрана */
@media (max-height: 600px) { ... }  /* Короткие экраны */

/* Ориентация */
@media (orientation: portrait) { ... }  /* Портретная ориентация */
@media (orientation: landscape) { ... } /* Альбомная ориентация */

/* Разрешение экрана */
@media (min-resolution: 2dppx) { ... } /* Ретина-дисплеи */

/* Тип устройства */
@media (hover: hover) { ... }     /* Устройства с поддержкой наведения */
@media (any-hover: hover) { ... } /* Любое устройство с поддержкой наведения */
@media (pointer: fine) { ... }    /* Точное указание (мышь, трекпад) */
@media (pointer: coarse) { ... }  /* Грубое указание (сенсорный экран) */
```

### Пример сложного медиа-запроса

```css
/* Комплексный медиа-запрос */
@media screen and (min-width: 768px) and (max-width: 1024px) and (orientation: landscape) {
    .container {
        max-width: 900px;
    }
    
    .sidebar {
        width: 25%;
    }
    
    .main-content {
        width: 75%;
    }
}

/* Условия для принтера */
@media print {
    .no-print {
        display: none;
    }
    
    body {
        font-size: 12pt;
        color: black;
        background: white;
    }
    
    a:after {
        content: " (" attr(href) ")";
        font-size: 0.8em;
        color: #666;
    }
}

/* Условия для высококонтрастного режима */
@media (prefers-contrast: high) {
    body {
        background-color: white;
        color: black;
    }
    
    .button {
        border: 2px solid black;
    }
}

/* Условия для уменьшенной анимации */
@media (prefers-reduced-motion: reduce) {
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

/* Условия для темного режима */
@media (prefers-color-scheme: dark) {
    body {
        background-color: #121212;
        color: #e0e0e0;
    }
    
    .card {
        background-color: #1e1e1e;
        border: 1px solid #333;
    }
}
```

## Адаптивные навигационные меню

### Мобильное меню

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивное меню</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: Arial, sans-serif;
        }

        .header {
            background-color: #333;
            padding: 1rem;
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .nav-container {
            display: flex;
            justify-content: space-between;
            align-items: center;
            max-width: 1200px;
            margin: 0 auto;
        }

        .logo {
            color: white;
            font-size: 1.5rem;
            text-decoration: none;
        }

        .nav-menu {
            display: flex;
            list-style: none;
        }

        .nav-menu li {
            margin-left: 1rem;
        }

        .nav-menu a {
            color: white;
            text-decoration: none;
            padding: 0.5rem 1rem;
            border-radius: 4px;
            transition: background-color 0.3s;
        }

        .nav-menu a:hover {
            background-color: #555;
        }

        .hamburger {
            display: none;
            flex-direction: column;
            cursor: pointer;
        }

        .hamburger span {
            width: 25px;
            height: 3px;
            background-color: white;
            margin: 3px 0;
            transition: 0.3s;
        }

        /* Мобильное меню */
        @media (max-width: 768px) {
            .nav-menu {
                position: fixed;
                left: -100%;
                top: 70px;
                flex-direction: column;
                background-color: #333;
                width: 100%;
                text-align: center;
                transition: 0.3s;
                box-shadow: 0 10px 27px rgba(0,0,0,0.05);
            }

            .nav-menu.active {
                left: 0;
            }

            .nav-menu li {
                margin: 1rem 0;
            }

            .hamburger {
                display: flex;
            }

            .hamburger.active span:nth-child(2) {
                opacity: 0;
            }

            .hamburger.active span:nth-child(1) {
                transform: translateY(8px) rotate(45deg);
            }

            .hamburger.active span:nth-child(3) {
                transform: translateY(-8px) rotate(-45deg);
            }
        }
    </style>
</head>
<body>
    <header class="header">
        <div class="nav-container">
            <a href="#" class="logo">Logo</a>
            
            <ul class="nav-menu" id="nav-menu">
                <li><a href="#">Главная</a></li>
                <li><a href="#">О нас</a></li>
                <li><a href="#">Услуги</a></li>
                <li><a href="#">Контакты</a></li>
            </ul>
            
            <div class="hamburger" id="hamburger">
                <span></span>
                <span></span>
                <span></span>
            </div>
        </div>
    </header>

    <script>
        const hamburger = document.getElementById('hamburger');
        const navMenu = document.getElementById('nav-menu');

        hamburger.addEventListener('click', () => {
            hamburger.classList.toggle('active');
            navMenu.classList.toggle('active');
        });

        // Закрытие меню при клике на ссылку
        document.querySelectorAll('.nav-menu a').forEach(link => {
            link.addEventListener('click', () => {
                hamburger.classList.remove('active');
                navMenu.classList.remove('active');
            });
        });
    </script>
</body>
</html>
```

## Адаптивные таблицы

### Техники адаптации таблиц

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивные таблицы</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 20px;
            font-family: Arial, sans-serif;
        }

        /* Базовая таблица */
        .table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }

        .table th,
        .table td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }

        .table th {
            background-color: #f2f2f2;
            font-weight: bold;
        }

        /* Таблица с прокруткой на мобильных */
        @media (max-width: 768px) {
            .table-container {
                overflow-x: auto;
                -webkit-overflow-scrolling: touch;
            }

            .table {
                min-width: 600px;
            }
        }

        /* Преобразование таблицы в карточки на мобильных */
        @media (max-width: 768px) {
            .card-table {
                display: block;
            }

            .card-table thead {
                display: none;
            }

            .card-table tbody tr {
                display: block;
                margin-bottom: 15px;
                border: 1px solid #ddd;
                border-radius: 8px;
                padding: 15px;
            }

            .card-table tbody tr:last-child {
                margin-bottom: 0;
            }

            .card-table tbody td {
                display: block;
                text-align: right;
                padding: 8px 0;
                border: none;
            }

            .card-table tbody td:before {
                content: attr(data-label) ": ";
                float: left;
                font-weight: bold;
                color: #555;
            }
        }
    </style>
</head>
<body>
    <h1>Адаптивные таблицы</h1>

    <!-- Таблица с прокруткой -->
    <div class="table-container">
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Имя</th>
                    <th>Email</th>
                    <th>Телефон</th>
                    <th>Город</th>
                    <th>Статус</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>1</td>
                    <td>Иван Иванов</td>
                    <td>ivan@example.com</td>
                    <td>+7 (999) 123-45-67</td>
                    <td>Москва</td>
                    <td>Активен</td>
                </tr>
                <tr>
                    <td>2</td>
                    <td>Мария Петрова</td>
                    <td>maria@example.com</td>
                    <td>+7 (999) 234-56-78</td>
                    <td>Санкт-Петербург</td>
                    <td>Неактивен</td>
                </tr>
            </tbody>
        </table>
    </div>

    <!-- Таблица, преобразуемая в карточки -->
    <table class="table card-table">
        <thead>
            <tr>
                <th>Имя</th>
                <th>Email</th>
                <th>Телефон</th>
                <th>Город</th>
                <th>Статус</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td data-label="Имя">Иван Иванов</td>
                <td data-label="Email">ivan@example.com</td>
                <td data-label="Телефон">+7 (999) 123-45-67</td>
                <td data-label="Город">Москва</td>
                <td data-label="Статус">Активен</td>
            </tr>
            <tr>
                <td data-label="Имя">Мария Петрова</td>
                <td data-label="Email">maria@example.com</td>
                <td data-label="Телефон">+7 (999) 234-56-78</td>
                <td data-label="Город">Санкт-Петербург</td>
                <td data-label="Статус">Неактивен</td>
            </tr>
        </tbody>
    </table>
</body>
</html>
```

## Адаптивные формы

### Адаптивные формы с различными подходами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивные формы</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 20px;
            font-family: Arial, sans-serif;
            background-color: #f5f5f5;
        }

        .form-container {
            max-width: 600px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        .form-row {
            display: flex;
            gap: 20px;
            margin-bottom: 20px;
        }

        .form-group {
            flex: 1;
            margin-bottom: 20px;
        }

        .form-group.full-width {
            flex: 100%;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #333;
        }

        input, select, textarea {
            width: 100%;
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }

        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 5px rgba(0, 122, 204, 0.3);
        }

        .form-actions {
            display: flex;
            gap: 10px;
            margin-top: 30px;
        }

        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }

        .btn-primary {
            background-color: #007acc;
            color: white;
        }

        .btn-secondary {
            background-color: #6c757d;
            color: white;
        }

        /* Адаптация формы для планшетов */
        @media (max-width: 1024px) {
            .form-row {
                flex-direction: column;
                gap: 0;
            }

            .form-group {
                margin-bottom: 20px;
            }
        }

        /* Адаптация формы для мобильных устройств */
        @media (max-width: 768px) {
            .form-container {
                margin: 10px;
                padding: 20px;
            }

            .form-actions {
                flex-direction: column;
            }

            .btn {
                width: 100%;
            }
        }

        /* Адаптация для маленьких экранов */
        @media (max-width: 480px) {
            body {
                margin: 10px;
            }

            .form-container {
                padding: 15px;
            }

            input, select, textarea {
                font-size: 16px; /* Для лучшей доступности на сенсорных устройствах */
            }
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Контактная форма</h1>

        <form>
            <div class="form-row">
                <div class="form-group">
                    <label for="first-name">Имя:</label>
                    <input type="text" id="first-name" name="first-name" required>
                </div>
                <div class="form-group">
                    <label for="last-name">Фамилия:</label>
                    <input type="text" id="last-name" name="last-name" required>
                </div>
            </div>

            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
            </div>

            <div class="form-group">
                <label for="phone">Телефон:</label>
                <input type="tel" id="phone" name="phone">
            </div>

            <div class="form-group">
                <label for="subject">Тема:</label>
                <select id="subject" name="subject" required>
                    <option value="">Выберите тему</option>
                    <option value="general">Общий вопрос</option>
                    <option value="support">Техническая поддержка</option>
                    <option value="billing">Биллинг</option>
                </select>
            </div>

            <div class="form-group full-width">
                <label for="message">Сообщение:</label>
                <textarea id="message" name="message" rows="5" required></textarea>
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Отправить</button>
                <button type="reset" class="btn btn-secondary">Очистить</button>
            </div>
        </form>
    </div>
</body>
</html>
```

## Современные подходы к адаптивности

### Container Queries (экспериментальная функция)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Container Queries</title>
    <style>
        /* Резервная поддержка для браузеров без container queries */
        .card {
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .card-content {
            display: flex;
            flex-direction: column;
        }

        .card-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
            margin-bottom: 15px;
        }

        .card-title {
            font-size: 1.2em;
            margin-bottom: 10px;
        }

        /* Container queries (работает в поддерживаемых браузерах) */
        @container (min-width: 400px) {
            .card-content {
                flex-direction: row;
                align-items: center;
            }

            .card-image {
                width: 150px;
                height: 150px;
                margin-bottom: 0;
                margin-right: 15px;
            }

            .card-text {
                flex: 1;
            }
        }

        @container (min-width: 600px) {
            .card {
                padding: 30px;
            }

            .card-title {
                font-size: 1.5em;
            }
        }
    </style>
</head>
<body>
    <h1>Container Queries пример</h1>

    <!-- Контейнер с определением для container queries -->
    <div class="card-container" style="container-type: inline-size;">
        <div class="card">
            <div class="card-content">
                <img src="https://via.placeholder.com/300x200" alt="Изображение" class="card-image">
                <div class="card-text">
                    <h3 class="card-title">Заголовок карточки</h3>
                    <p>Содержимое карточки. При достаточной ширине карточка будет отображаться горизонтально.</p>
                </div>
            </div>
        </div>
    </div>

    <!-- Разные размеры контейнеров для демонстрации -->
    <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; margin-top: 20px;">
        <div class="card-container" style="container-type: inline-size; border: 1px solid #ccc; padding: 10px;">
            <div class="card">
                <div class="card-content">
                    <img src="https://via.placeholder.com/150x100" alt="Маленькое изображение" class="card-image">
                    <div class="card-text">
                        <h3>Маленькая карточка</h3>
                        <p>Текст будет подстраиваться под размер контейнера.</p>
                    </div>
                </div>
            </div>
        </div>

        <div class="card-container" style="container-type: inline-size; border: 1px solid #ccc; padding: 10px;">
            <div class="card">
                <div class="card-content">
                    <img src="https://via.placeholder.com/200x150" alt="Среднее изображение" class="card-image">
                    <div class="card-text">
                        <h3>Средняя карточка</h3>
                        <p>При достаточной ширине отобразится горизонтально.</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

### CSS Custom Properties для адаптивности

```css
:root {
    /* Переменные для адаптивности */
    --container-max-width: 1200px;
    --grid-gap: 20px;
    --font-size-base: 16px;
    --breakpoint-mobile: 768px;
    --breakpoint-tablet: 1024px;
    --breakpoint-desktop: 1200px;
}

/* Использование переменных в медиа-запросах */
.main-content {
    padding: 2rem;
    max-width: var(--container-max-width);
    margin: 0 auto;
}

@supports (width: clamp(20px, 5vw, 100px)) {
    /* Использование clamp() для адаптивных размеров */
    .responsive-heading {
        font-size: clamp(1.5rem, 4vw, 3rem);
    }

    .responsive-padding {
        padding: clamp(1rem, 3vw, 2rem);
    }
}

/* Адаптивные отступы */
.responsive-card {
    padding: clamp(1rem, 2.5vw, 2rem);
    margin: clamp(0.5rem, 1.5vw, 1rem);
}

/* Использование calc() для адаптивных размеров */
.adaptive-width {
    width: calc(100vw - 2rem);
}

@media (min-width: var(--breakpoint-mobile)) {
    .adaptive-width {
        width: calc(100% - 4rem);
    }
}
```

## Практические примеры адаптивных компонентов

### Адаптивная галерея

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная галерея</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
        }

        .gallery {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 20px;
            max-width: 1200px;
            margin: 0 auto;
        }

        .gallery-item {
            position: relative;
            overflow: hidden;
            border-radius: 8px;
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            aspect-ratio: 1 / 1;
        }

        .gallery-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: transform 0.3s ease;
        }

        .gallery-item:hover .gallery-image {
            transform: scale(1.05);
        }

        .gallery-caption {
            position: absolute;
            bottom: 0;
            left: 0;
            right: 0;
            background: linear-gradient(to top, rgba(0,0,0,0.8), transparent);
            color: white;
            padding: 20px 10px 10px;
            transform: translateY(100%);
            transition: transform 0.3s ease;
        }

        .gallery-item:hover .gallery-caption {
            transform: translateY(0);
        }

        /* Адаптация для планшетов */
        @media (max-width: 1024px) {
            .gallery {
                grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
                gap: 15px;
            }
        }

        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .gallery {
                grid-template-columns: repeat(2, 1fr);
                gap: 10px;
            }
        }

        /* Адаптация для маленьких экранов */
        @media (max-width: 480px) {
            .gallery {
                grid-template-columns: 1fr;
                gap: 10px;
            }

            body {
                padding: 10px;
            }
        }
    </style>
</head>
<body>
    <h1>Адаптивная галерея</h1>

    <div class="gallery">
        <div class="gallery-item">
            <img src="https://via.placeholder.com/300x300/FF6B6B/FFFFFF?text=1" alt="Изображение 1" class="gallery-image">
            <div class="gallery-caption">Описание 1</div>
        </div>
        <div class="gallery-item">
            <img src="https://via.placeholder.com/300x300/4ECDC4/FFFFFF?text=2" alt="Изображение 2" class="gallery-image">
            <div class="gallery-caption">Описание 2</div>
        </div>
        <div class="gallery-item">
            <img src="https://via.placeholder.com/300x300/45B7D1/FFFFFF?text=3" alt="Изображение 3" class="gallery-image">
            <div class="gallery-caption">Описание 3</div>
        </div>
        <div class="gallery-item">
            <img src="https://via.placeholder.com/300x300/96CEB4/FFFFFF?text=4" alt="Изображение 4" class="gallery-image">
            <div class="gallery-caption">Описание 4</div>
        </div>
        <div class="gallery-item">
            <img src="https://via.placeholder.com/300x300/FFEAA7/333333?text=5" alt="Изображение 5" class="gallery-image">
            <div class="gallery-caption">Описание 5</div>
        </div>
        <div class="gallery-item">
            <img src="https://via.placeholder.com/300x300/DDA0DD/FFFFFF?text=6" alt="Изображение 6" class="gallery-image">
            <div class="gallery-caption">Описание 6</div>
        </div>
    </div>
</body>
</html>
```

### Адаптивная карточка продукта

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная карточка продукта</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 20px;
            font-family: Arial, sans-serif;
            background-color: #f5f5f5;
        }

        .product-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 30px;
            max-width: 1200px;
            margin: 0 auto;
        }

        .product-card {
            background: white;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .product-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0,0,0,0.15);
        }

        .product-image-container {
            position: relative;
            height: 200px;
            overflow: hidden;
        }

        .product-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: transform 0.3s ease;
        }

        .product-card:hover .product-image {
            transform: scale(1.05);
        }

        .product-badge {
            position: absolute;
            top: 10px;
            left: 10px;
            background: #ff5722;
            color: white;
            padding: 5px 10px;
            border-radius: 20px;
            font-size: 0.8em;
            font-weight: bold;
        }

        .product-info {
            padding: 20px;
        }

        .product-category {
            color: #666;
            font-size: 0.9em;
            margin-bottom: 5px;
        }

        .product-title {
            margin: 0 0 10px 0;
            font-size: 1.2em;
            color: #333;
        }

        .product-description {
            color: #666;
            margin-bottom: 15px;
            line-height: 1.5;
        }

        .product-rating {
            display: flex;
            align-items: center;
            margin-bottom: 15px;
        }

        .stars {
            color: #ffc107;
            margin-right: 10px;
        }

        .rating-count {
            color: #999;
            font-size: 0.9em;
        }

        .product-price-container {
            display: flex;
            align-items: center;
            margin-bottom: 15px;
        }

        .current-price {
            font-size: 1.5em;
            font-weight: bold;
            color: #007acc;
        }

        .original-price {
            text-decoration: line-through;
            color: #999;
            margin-left: 10px;
            font-size: 0.9em;
        }

        .discount-percent {
            background: #f44336;
            color: white;
            padding: 2px 8px;
            border-radius: 10px;
            font-size: 0.8em;
            margin-left: 10px;
        }

        .product-actions {
            display: flex;
            gap: 10px;
        }

        .btn {
            flex: 1;
            padding: 12px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 1em;
            transition: background-color 0.3s;
        }

        .btn-primary {
            background-color: #007acc;
            color: white;
        }

        .btn-primary:hover {
            background-color: #005a9e;
        }

        .btn-outline {
            background-color: transparent;
            color: #007acc;
            border: 2px solid #007acc;
        }

        .btn-outline:hover {
            background-color: #007acc;
            color: white;
        }

        /* Адаптация для планшетов */
        @media (max-width: 1024px) {
            .product-grid {
                grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
                gap: 20px;
            }
        }

        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .product-grid {
                grid-template-columns: 1fr;
                gap: 20px;
            }

            .product-actions {
                flex-direction: column;
            }
        }

        /* Адаптация для маленьких экранов */
        @media (max-width: 480px) {
            body {
                margin: 10px;
            }

            .product-info {
                padding: 15px;
            }

            .product-title {
                font-size: 1.1em;
            }

            .current-price {
                font-size: 1.3em;
            }
        }
    </style>
</head>
<body>
    <h1>Адаптивные карточки продуктов</h1>

    <div class="product-grid">
        <div class="product-card">
            <div class="product-image-container">
                <img src="https://via.placeholder.com/300x200/FF6B6B/FFFFFF?text=Товар" alt="Наушники" class="product-image">
                <div class="product-badge">Хит продаж</div>
            </div>
            <div class="product-info">
                <div class="product-category">Электроника</div>
                <h3 class="product-title">Беспроводные наушники</h3>
                <p class="product-description">Высококачественные беспроводные наушники с шумоподавлением и длительным временем работы.</p>
                
                <div class="product-rating">
                    <div class="stars">★★★★☆</div>
                    <div class="rating-count">(124 отзыва)</div>
                </div>
                
                <div class="product-price-container">
                    <span class="current-price">12 990 ₽</span>
                    <span class="original-price">15 990 ₽</span>
                    <span class="discount-percent">19% скидка</span>
                </div>
                
                <div class="product-actions">
                    <button class="btn btn-primary">В корзину</button>
                    <button class="btn btn-outline">В избранное</button>
                </div>
            </div>
        </div>

        <div class="product-card">
            <div class="product-image-container">
                <img src="https://via.placeholder.com/300x200/4ECDC4/FFFFFF?text=Товар" alt="Смартфон" class="product-image">
                <div class="product-badge">Новинка</div>
            </div>
            <div class="product-info">
                <div class="product-category">Телефоны</div>
                <h3 class="product-title">Смартфон iPhone 15</h3>
                <p class="product-description">Новейший iPhone с передовыми технологиями и улучшенной камерой.</p>
                
                <div class="product-rating">
                    <div class="stars">★★★★★</div>
                    <div class="rating-count">(256 отзывов)</div>
                </div>
                
                <div class="product-price-container">
                    <span class="current-price">99 990 ₽</span>
                </div>
                
                <div class="product-actions">
                    <button class="btn btn-primary">В корзину</button>
                    <button class="btn btn-outline">Сравнить</button>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

## Проверка адаптивности

### Инструменты для проверки:
1. **Browser DevTools** - встроенные инструменты для эмуляции устройств
2. **Responsinator** - онлайн инструмент для проверки адаптивности
3. **BrowserStack** - тестирование на реальных устройствах
4. **Lighthouse** - проверка адаптивности и доступности
5. **Am I Responsive** - визуализация сайта на разных устройствах

### Проверка на реальных устройствах:
- Мобильные телефоны (iOS и Android)
- Планшеты (iPad, Android планшеты)
- Ноутбуки и настольные компьютеры
- Различные размеры экранов и разрешения

### Тестирование доступности:
- Использование клавиатуры на мобильных устройствах
- Поддержка вспомогательных технологий
- Читаемость текста на маленьких экранах
- Размер сенсорных зон

## Лучшие практики адаптивного дизайна

### 1. Mobile First подход
```css
/* Начинаем с мобильных стилей */
.container {
    width: 100%;
    padding: 10px;
}

.grid {
    display: flex;
    flex-direction: column;
}

/* Затем добавляем стили для больших экранов */
@media (min-width: 768px) {
    .container {
        max-width: 750px;
        padding: 20px;
    }
    
    .grid {
        flex-direction: row;
    }
}

@media (min-width: 1024px) {
    .container {
        max-width: 1000px;
    }
}
```

### 2. Использование относительных единиц
```css
/* Хорошо: использование относительных единиц */
.element {
    width: 100%;           /* Проценты */
    font-size: 1.2rem;     /* rem для типографики */
    padding: 2em;          /* em для пропорциональных отступов */
    margin: 5vw;           /* viewport width */
    height: 30vh;          /* viewport height */
}

/* Плохо: жесткие фиксированные размеры */
.bad-element {
    width: 960px;          /* Не подходит для мобильных */
    font-size: 16px;       /* Не масштабируется с настройками пользователя */
}
```

### 3. Оптимизация производительности
```css
/* Оптимизированные медиа-запросы */
@media screen and (min-width: 768px) {
    /* Стили только для экранов, не для печати */
    .component {
        /* Оптимизированные стили */
    }
}

/* Использование contain для производительности */
.optimized-component {
    contain: layout style paint;
}
```

### 4. Проверка контрастности
```css
/* Обеспечение хорошего контрастного соотношения */
.text-on-dark {
    color: #ffffff;
    background-color: #333333;
    /* Контрастное соотношение 7:1 - AAA рейтинг */
}

.text-on-light {
    color: #333333;
    background-color: #ffffff;
    /* Контрастное соотношение 12:1 - AAA рейтинг */
}
```

## Современные возможности адаптивности

### 1. CSS Grid Layout
```css
/* Адаптивная сетка с автоматическим размещением */
.responsive-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
}

/* Адаптивная сетка с именованными областями */
.layout-grid {
    display: grid;
    grid-template-areas: 
        "header header"
        "sidebar main"
        "footer footer";
    grid-template-columns: 200px 1fr;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }

@media (max-width: 768px) {
    .layout-grid {
        grid-template-areas: 
            "header"
            "main"
            "sidebar"
            "footer";
        grid-template-columns: 1fr;
    }
}
```

### 2. Flexbox для адаптивных компонентов
```css
/* Адаптивная навигация */
.nav-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
}

.nav-links {
    display: flex;
    gap: 20px;
    flex-wrap: wrap;
}

@media (max-width: 768px) {
    .nav-container {
        flex-direction: column;
        gap: 15px;
    }
    
    .nav-links {
        width: 100%;
        justify-content: center;
    }
}
```

### 3. Векторные изображения и масштабирование
```css
/* Использование SVG для масштабируемых изображений */
.icon {
    width: 24px;
    height: 24px;
    /* SVG автоматически масштабируются без потери качества */
}

/* Адаптивные изображения с правильным соотношением сторон */
.aspect-ratio-container {
    position: relative;
    width: 100%;
    height: 0;
    padding-bottom: 56.25%; /* 16:9 */
}

.aspect-ratio-content {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
```

### 4. Современные функции CSS
```css
/* Использование clamp() для адаптивных размеров */
.responsive-heading {
    font-size: clamp(1.5rem, 4vw, 3rem);
}

.responsive-padding {
    padding: clamp(1rem, 5vw, 3rem);
}

/* Использование min(), max() и calc() */
.adaptive-element {
    width: min(100%, 600px); /* Не больше 600px, но не больше 100% ширины */
    font-size: calc(1rem + 0.5vw); /* Адаптивный размер шрифта */
}
```

## Заключение

Адаптивный дизайн - это не просто тренд, а необходимость современной веб-разработки. С ростом количества мобильных устройств и различных размеров экранов, создание адаптивных сайтов стало критически важным для обеспечения хорошего пользовательского опыта на всех устройствах.

Ключевые аспекты адаптивного дизайна:
1. Использование гибких сеток и макетов
2. Адаптация изображений и медиа-контента
3. Эффективное использование медиа-запросов
4. Оптимизация производительности
5. Обеспечение доступности
6. Тестирование на различных устройствах
7. Следование современным веб-стандартам

Современные возможности CSS, такие как Grid, Flexbox, Container Queries и новые функции, открывают новые возможности для создания сложных адаптивных макетов с минимальными затратами на поддержку.

## Следующие темы
- [[CSS/Анимации]]
- [[CSS/Продвинутые возможности]]
- [[CSS/Производительность]]

## Теги
#css #responsive-design #web-development #frontend #mobile-first #adaptive-layout #css-grid #flexbox #media-queries #accessibility #performance #modern-css #viewport #scalable-design #cross-device
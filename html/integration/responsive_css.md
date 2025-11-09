# HTML Интеграция: Адаптивная верстка с CSS

Адаптивная верстка - это подход к веб-дизайну, при котором веб-страницы автоматически адаптируются к различным размерам экранов и устройствам. HTML и CSS тесно интегрируются для создания адаптивных интерфейсов.

## Основы адаптивной верстки

### Viewport метатег

Первый шаг в создании адаптивной верстки - правильная настройка viewport:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная верстка</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
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
        <h1>Адаптивная веб-страница</h1>
        <p>Этот контент адаптируется к размеру экрана.</p>
    </div>
</body>
</html>
```

### Основные принципы адаптивности

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Принципы адаптивности</title>
    <style>
        /* 1. Гибкие сетки */
        .grid-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            padding: 20px;
        }
        
        .grid-item {
            background-color: #f0f0f0;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #ddd;
        }
        
        /* 2. Гибкие изображения */
        img {
            max-width: 100%;
            height: auto;
        }
        
        /* 3. Медиа-запросы */
        @media (max-width: 768px) {
            .grid-container {
                grid-template-columns: 1fr;
            }
            
            body {
                padding: 10px;
            }
        }
    </style>
</head>
<body>
    <div class="grid-container">
        <div class="grid-item">
            <h3>Карточка 1</h3>
            <p>Содержимое первой карточки</p>
            <img src="https://via.placeholder.com/300x200" alt="Пример изображения">
        </div>
        <div class="grid-item">
            <h3>Карточка 2</h3>
            <p>Содержимое второй карточки</p>
            <img src="https://via.placeholder.com/300x200" alt="Пример изображения">
        </div>
        <div class="grid-item">
            <h3>Карточка 3</h3>
            <p>Содержимое третьей карточки</p>
            <img src="https://via.placeholder.com/300x200" alt="Пример изображения">
        </div>
    </div>
</body>
</html>
```

## Flexbox для адаптивности

### Основы Flexbox

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flexbox адаптивность</title>
    <style>
        .flex-container {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            padding: 20px;
        }
        
        .flex-item {
            flex: 1 1 300px; /* grow, shrink, basis */
            background-color: #e3f2fd;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #bbdefb;
        }
        
        /* Адаптация на мобильных устройствах */
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
        <div class="flex-item">
            <h3>Элемент 1</h3>
            <p>Первый гибкий элемент</p>
        </div>
        <div class="flex-item">
            <h3>Элемент 2</h3>
            <p>Второй гибкий элемент</p>
        </div>
        <div class="flex-item">
            <h3>Элемент 3</h3>
            <p>Третий гибкий элемент</p>
        </div>
    </div>
</body>
</html>
```

### Расширенные Flexbox примеры

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Расширенный Flexbox</title>
    <style>
        .advanced-flex {
            display: flex;
            min-height: 100vh;
        }
        
        .sidebar {
            flex: 0 0 250px;
            background-color: #333;
            color: white;
            padding: 20px;
        }
        
        .main-content {
            flex: 1;
            padding: 20px;
            background-color: #f5f5f5;
        }
        
        .footer {
            flex: 0 0 auto;
            background-color: #666;
            color: white;
            text-align: center;
            padding: 10px;
        }
        
        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .advanced-flex {
                flex-direction: column;
            }
            
            .sidebar {
                flex: 0 0 auto;
            }
        }
    </style>
</head>
<body>
    <div class="advanced-flex">
        <aside class="sidebar">
            <h2>Навигация</h2>
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/services">Услуги</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </aside>
        
        <main class="main-content">
            <h1>Основной контент</h1>
            <p>Это основное содержимое страницы.</p>
        </main>
        
        <footer class="footer">
            <p>&copy; 2023 Все права защищены</p>
        </footer>
    </div>
</body>
</html>
```

## CSS Grid для адаптивности

### Основы CSS Grid

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Grid адаптивность</title>
    <style>
        .grid-layout {
            display: grid;
            grid-template-areas: 
                "header header header"
                "sidebar main aside"
                "footer footer footer";
            grid-template-columns: 200px 1fr 200px;
            grid-template-rows: auto 1fr auto;
            gap: 20px;
            min-height: 100vh;
            padding: 20px;
        }
        
        .header { grid-area: header; background-color: #4CAF50; color: white; padding: 20px; }
        .sidebar { grid-area: sidebar; background-color: #2196F3; color: white; padding: 20px; }
        .main { grid-area: main; background-color: #f5f5f5; padding: 20px; }
        .aside { grid-area: aside; background-color: #FF9800; color: white; padding: 20px; }
        .footer { grid-area: footer; background-color: #795548; color: white; padding: 20px; }
        
        /* Адаптация для планшетов */
        @media (max-width: 1024px) {
            .grid-layout {
                grid-template-areas: 
                    "header"
                    "main"
                    "sidebar"
                    "aside"
                    "footer";
                grid-template-columns: 1fr;
            }
        }
        
        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .grid-layout {
                grid-template-areas: 
                    "header"
                    "main"
                    "footer";
                grid-template-columns: 1fr;
            }
            
            .sidebar, .aside {
                display: none;
            }
        }
    </style>
</head>
<body>
    <div class="grid-layout">
        <header class="header">
            <h1>Заголовок сайта</h1>
        </header>
        
        <aside class="sidebar">
            <h3>Боковая панель</h3>
            <p>Содержимое боковой панели</p>
        </aside>
        
        <main class="main">
            <h2>Основной контент</h2>
            <p>Это основное содержимое страницы.</p>
        </main>
        
        <aside class="aside">
            <h3>Дополнительная информация</h3>
            <p>Дополнительный контент</p>
        </aside>
        
        <footer class="footer">
            <p>&copy; 2023 Все права защищены</p>
        </footer>
    </div>
</body>
</html>
```

## Адаптивные изображения

### Responsive images с srcset

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивные изображения</title>
    <style>
        .image-container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        img {
            width: 100%;
            height: auto;
            display: block;
        }
    </style>
</head>
<body>
    <div class="image-container">
        <!-- Адаптивное изображение с разными размерами -->
        <img 
            srcset="
                small-image.jpg 400w,
                medium-image.jpg 800w,
                large-image.jpg 1200w
            "
            sizes="(max-width: 600px) 400px, (max-width: 1000px) 800px, 1200px"
            src="medium-image.jpg" 
            alt="Адаптивное изображение">
        
        <p>Изображение автоматически подбирается в зависимости от размера экрана.</p>
    </div>
</body>
</html>
```

### Picture элемент для разных форматов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Picture элемент</title>
    <style>
        .picture-container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
    </style>
</head>
<body>
    <div class="picture-container">
        <picture>
            <!-- Изображение для больших экранов -->
            <source 
                media="(min-width: 800px)" 
                srcset="large-image.webp" 
                type="image/webp">
            <source 
                media="(min-width: 800px)" 
                srcset="large-image.jpg" 
                type="image/jpeg">
            
            <!-- Изображение для планшетов -->
            <source 
                media="(min-width: 600px)" 
                srcset="medium-image.webp" 
                type="image/webp">
            <source 
                media="(min-width: 600px)" 
                srcset="medium-image.jpg" 
                type="image/jpeg">
            
            <!-- Изображение по умолчанию -->
            <source srcset="small-image.webp" type="image/webp">
            <source srcset="small-image.jpg" type="image/jpeg">
            
            <!-- Fallback для браузеров без поддержки picture -->
            <img src="small-image.jpg" alt="Адаптивное изображение с picture">
        </picture>
        
        <p>Picture элемент позволяет показывать разные изображения для разных условий.</p>
    </div>
</body>
</html>
```

## Адаптивные формы

### Адаптивная форма с CSS Grid

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная форма</title>
    <style>
        .form-container {
            max-width: 600px;
            margin: 20px auto;
            padding: 20px;
            background-color: #f9f9f9;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        .form-grid {
            display: grid;
            grid-template-columns: 1fr;
            gap: 15px;
        }
        
        .form-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
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
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 12px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        
        button:hover {
            background-color: #005a9e;
        }
        
        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .form-row {
                grid-template-columns: 1fr;
            }
            
            .form-container {
                margin: 10px;
                padding: 15px;
            }
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h2>Адаптивная регистрационная форма</h2>
        
        <form class="form-grid">
            <div class="form-row">
                <div class="form-group">
                    <label for="firstName">Имя:</label>
                    <input type="text" id="firstName" name="firstName" required>
                </div>
                
                <div class="form-group">
                    <label for="lastName">Фамилия:</label>
                    <input type="text" id="lastName" name="lastName" required>
                </div>
            </div>
            
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
            </div>
            
            <div class="form-row">
                <div class="form-group">
                    <label for="phone">Телефон:</label>
                    <input type="tel" id="phone" name="phone">
                </div>
                
                <div class="form-group">
                    <label for="birthDate">Дата рождения:</label>
                    <input type="date" id="birthDate" name="birthDate">
                </div>
            </div>
            
            <div class="form-group">
                <label for="country">Страна:</label>
                <select id="country" name="country">
                    <option value="">Выберите страну</option>
                    <option value="ru">Россия</option>
                    <option value="us">США</option>
                    <option value="de">Германия</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="message">Сообщение:</label>
                <textarea id="message" name="message" rows="4"></textarea>
            </div>
            
            <div class="form-group">
                <button type="submit">Отправить</button>
            </div>
        </form>
    </div>
</body>
</html>
```

## Адаптивная навигация

### Мобильное меню

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная навигация</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        .navbar {
            background-color: #333;
            padding: 1rem;
            position: relative;
        }
        
        .nav-container {
            max-width: 1200px;
            margin: 0 auto;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .logo {
            color: white;
            font-size: 1.5rem;
            font-weight: bold;
        }
        
        .nav-menu {
            display: flex;
            list-style: none;
        }
        
        .nav-item {
            margin-left: 2rem;
        }
        
        .nav-link {
            color: white;
            text-decoration: none;
            transition: color 0.3s ease;
        }
        
        .nav-link:hover {
            color: #ddd;
        }
        
        .hamburger {
            display: none;
            flex-direction: column;
            cursor: pointer;
        }
        
        .bar {
            width: 25px;
            height: 3px;
            background-color: white;
            margin: 3px 0;
            transition: 0.3s;
        }
        
        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .hamburger {
                display: flex;
            }
            
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
            
            .nav-item {
                margin: 2.5rem 0;
            }
            
            .nav-link {
                font-size: 1.2rem;
            }
            
            .hamburger.active .bar:nth-child(2) {
                opacity: 0;
            }
            
            .hamburger.active .bar:nth-child(1) {
                transform: translateY(8px) rotate(45deg);
            }
            
            .hamburger.active .bar:nth-child(3) {
                transform: translateY(-8px) rotate(-45deg);
            }
        }
    </style>
</head>
<body>
    <nav class="navbar">
        <div class="nav-container">
            <a href="/" class="logo">Логотип</a>
            
            <ul class="nav-menu" id="nav-menu">
                <li class="nav-item">
                    <a href="/" class="nav-link">Главная</a>
                </li>
                <li class="nav-item">
                    <a href="/about" class="nav-link">О нас</a>
                </li>
                <li class="nav-item">
                    <a href="/services" class="nav-link">Услуги</a>
                </li>
                <li class="nav-item">
                    <a href="/contact" class="nav-link">Контакты</a>
                </li>
            </ul>
            
            <div class="hamburger" id="hamburger">
                <span class="bar"></span>
                <span class="bar"></span>
                <span class="bar"></span>
            </div>
        </div>
    </nav>
    
    <main style="padding: 2rem; max-width: 1200px; margin: 0 auto;">
        <h1>Адаптивная навигация</h1>
        <p>Эта навигация адаптируется к размеру экрана.</p>
    </main>
    
    <script>
        const hamburger = document.getElementById('hamburger');
        const navMenu = document.getElementById('nav-menu');
        
        hamburger.addEventListener('click', function() {
            hamburger.classList.toggle('active');
            navMenu.classList.toggle('active');
        });
        
        // Закрытие меню при клике на ссылку
        document.querySelectorAll('.nav-link').forEach(link => {
            link.addEventListener('click', () => {
                hamburger.classList.remove('active');
                navMenu.classList.remove('active');
            });
        });
    </script>
</body>
</html>
</body>
</html>
```

## Адаптивные медиа-запросы

### Расширенные медиа-запросы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Расширенные медиа-запросы</title>
    <style>
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .card-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
        }
        
        .card {
            background: white;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            transition: transform 0.3s ease;
        }
        
        .card:hover {
            transform: translateY(-5px);
        }
        
        /* Мобильные устройства */
        @media (max-width: 480px) {
            .container {
                padding: 10px;
            }
            
            .card-grid {
                grid-template-columns: 1fr;
                gap: 15px;
            }
        }
        
        /* Планшеты */
        @media (min-width: 481px) and (max-width: 768px) {
            .card-grid {
                grid-template-columns: repeat(2, 1fr);
                gap: 15px;
            }
        }
        
        /* Настольные компьютеры */
        @media (min-width: 769px) and (max-width: 1024px) {
            .card-grid {
                grid-template-columns: repeat(3, 1fr);
            }
        }
        
        /* Большие экраны */
        @media (min-width: 1025px) {
            .card-grid {
                grid-template-columns: repeat(4, 1fr);
            }
        }
        
        /* Портретная ориентация */
        @media (orientation: portrait) {
            .card {
                min-height: 200px;
            }
        }
        
        /* Альбомная ориентация */
        @media (orientation: landscape) and (max-height: 500px) {
            .card-grid {
                grid-auto-flow: column;
                grid-auto-columns: minmax(300px, 1fr);
                overflow-x: auto;
            }
        }
        
        /* Высококонтрастный режим */
        @media (prefers-contrast: high) {
            .card {
                border: 2px solid;
            }
        }
        
        /* Темная тема */
        @media (prefers-color-scheme: dark) {
            body {
                background-color: #1a1a1a;
                color: #fff;
            }
            
            .card {
                background-color: #2d2d2d;
                color: #fff;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Расширенные медиа-запросы</h1>
        <p>Этот контент адаптируется к различным условиям просмотра.</p>
        
        <div class="card-grid">
            <div class="card">
                <h3>Карточка 1</h3>
                <p>Содержимое карточки 1</p>
            </div>
            <div class="card">
                <h3>Карточка 2</h3>
                <p>Содержимое карточки 2</p>
            </div>
            <div class="card">
                <h3>Карточка 3</h3>
                <p>Содержимое карточки 3</p>
            </div>
            <div class="card">
                <h3>Карточка 4</h3>
                <p>Содержимое карточки 4</p>
            </div>
        </div>
    </div>
</body>
</html>
```

## Адаптивные таблицы

### Адаптивные таблицы с различными подходами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивные таблицы</title>
    <style>
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        /* Базовая таблица */
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        
        th {
            background-color: #f2f2f2;
            font-weight: bold;
        }
        
        /* Подход 1: Горизонтальный скролл */
        .scrollable-table {
            overflow-x: auto;
            white-space: nowrap;
        }
        
        /* Подход 2: Преобразование в карточки на мобильных */
        .responsive-table {
            display: block;
        }
        
        .responsive-table thead {
            display: none;
        }
        
        .responsive-table tr {
            display: block;
            margin-bottom: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 10px;
        }
        
        .responsive-table td {
            display: block;
            text-align: right;
            border: none;
            position: relative;
            padding-right: 50%;
        }
        
        .responsive-table td:before {
            content: attr(data-label) ": ";
            position: absolute;
            left: 6px;
            width: 45%;
            text-align: left;
            font-weight: bold;
        }
        
        /* Подход 3: Скрытие колонок */
        .conditional-table .hide-mobile {
            display: table-cell;
        }
        
        @media (max-width: 768px) {
            .conditional-table .hide-mobile {
                display: none;
            }
        }
        
        /* Подход 4: Стили для мобильных */
        @media (max-width: 768px) {
            .responsive-table {
                display: table;
            }
            
            .responsive-table thead {
                display: table-header-group;
            }
            
            .responsive-table tr {
                display: table-row;
                margin-bottom: 0;
                border: none;
                padding: 0;
            }
            
            .responsive-table td {
                display: table-cell;
                text-align: left;
                border-bottom: 1px solid #ddd;
                position: static;
                padding: 12px;
            }
            
            .responsive-table td:before {
                display: none;
            }
            
            /* Альтернативный подход для маленьких экранов */
            .mobile-card-table tr {
                display: block;
                margin-bottom: 15px;
                border: 1px solid #ddd;
                border-radius: 8px;
                padding: 10px;
            }
            
            .mobile-card-table td {
                display: block;
                text-align: right;
                border: none;
                position: relative;
                padding-right: 50%;
            }
            
            .mobile-card-table td:before {
                content: attr(data-label) ": ";
                position: absolute;
                left: 6px;
                width: 45%;
                text-align: left;
                font-weight: bold;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Адаптивные таблицы</h1>
        
        <!-- Таблица с горизонтальным скроллом -->
        <h2>Подход 1: Горизонтальный скролл</h2>
        <div class="scrollable-table">
            <table>
                <thead>
                    <tr>
                        <th>Имя</th>
                        <th>Email</th>
                        <th>Телефон</th>
                        <th>Должность</th>
                        <th>Отдел</th>
                        <th>Зарплата</th>
                        <th>Дата найма</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>Иван Иванов</td>
                        <td>ivan@example.com</td>
                        <td>+7 (123) 456-78-90</td>
                        <td>Разработчик</td>
                        <td>IT</td>
                        <td>100,000 руб.</td>
                        <td>2020-01-15</td>
                    </tr>
                    <tr>
                        <td>Мария Петрова</td>
                        <td>maria@example.com</td>
                        <td>+7 (123) 456-78-91</td>
                        <td>Дизайнер</td>
                        <td>Дизайн</td>
                        <td>80,000 руб.</td>
                        <td>2020-03-20</td>
                    </tr>
                </tbody>
            </table>
        </div>
        
        <!-- Таблица как карточки на мобильных -->
        <h2>Подход 2: Карточки на мобильных</h2>
        <table class="responsive-table mobile-card-table">
            <thead>
                <tr>
                    <th>Имя</th>
                    <th>Email</th>
                    <th>Телефон</th>
                    <th>Должность</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td data-label="Имя">Иван Иванов</td>
                    <td data-label="Email">ivan@example.com</td>
                    <td data-label="Телефон">+7 (123) 456-78-90</td>
                    <td data-label="Должность">Разработчик</td>
                </tr>
                <tr>
                    <td data-label="Имя">Мария Петрова</td>
                    <td data-label="Email">maria@example.com</td>
                    <td data-label="Телефон">+7 (123) 456-78-91</td>
                    <td data-label="Должность">Дизайнер</td>
                </tr>
            </tbody>
        </table>
        
        <!-- Таблица с условным отображением колонок -->
        <h2>Подход 3: Скрытие колонок</h2>
        <table class="conditional-table">
            <thead>
                <tr>
                    <th>Имя</th>
                    <th>Email</th>
                    <th>Телефон</th>
                    <th class="hide-mobile">Должность</th>
                    <th class="hide-mobile">Отдел</th>
                    <th>Зарплата</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Иван Иванов</td>
                    <td>ivan@example.com</td>
                    <td>+7 (123) 456-78-90</td>
                    <td class="hide-mobile">Разработчик</td>
                    <td class="hide-mobile">IT</td>
                    <td>100,000 руб.</td>
                </tr>
                <tr>
                    <td>Мария Петрова</td>
                    <td>maria@example.com</td>
                    <td>+7 (123) 456-78-91</td>
                    <td class="hide-mobile">Дизайнер</td>
                    <td class="hide-mobile">Дизайн</td>
                    <td>80,000 руб.</td>
                </tr>
            </tbody>
        </table>
    </div>
</body>
</html>
</body>
</html>
```

## Заключение

Адаптивная верстка с HTML и CSS - это ключевой навык современного веб-разработчика. Основные принципы включают:

1. **Гибкие сетки** - использование процентов, em, rem, fr единиц
2. **Гибкие изображения** - max-width: 100%, srcset, picture элемент
3. **Медиа-запросы** - адаптация под разные размеры экранов
4. **Flexbox и Grid** - современные методы раскладки
5. **Адаптивные таблицы** - различные подходы для разных ситуаций
6. **Адаптивная навигация** - мобильные меню и структура

Правильная интеграция HTML и CSS позволяет создавать веб-сайты, которые отлично выглядят и работают на всех устройствах.

## Следующие темы
- [[HTML/Интеграция с JavaScript/События и DOM]]
- [[HTML/Производительность]]
- [[HTML/Доступность]]

## Теги
#html #css #responsive-design #web-development #frontend #flexbox #grid #media-queries
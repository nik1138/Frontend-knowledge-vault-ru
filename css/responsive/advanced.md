# Адаптивный дизайн CSS: медиа-запросы, флексбокс, грид и современные подходы

## Введение

Адаптивный дизайн (Responsive Design) - это подход к веб-дизайну, который позволяет веб-страницам корректно отображаться на различных устройствах с разными размерами экрана. В этой статье мы рассмотрим ключевые техники и современные подходы к созданию адаптивных веб-интерфейсов.

## Основы адаптивного дизайна

### 1. Viewport метатег

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <!-- Критический метатег для адаптивного дизайна -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная страница</title>
</head>
<body>
    <!-- Содержимое страницы -->
</body>
</html>
```

### 2. Мобильный первый подход

```css
/* Стили по умолчанию для мобильных устройств */
.container {
    width: 100%;
    padding: 1rem;
}

.card {
    margin-bottom: 1rem;
    padding: 1rem;
}

/* Адаптация для планшетов */
@media (min-width: 768px) {
    .container {
        max-width: 750px;
        margin: 0 auto;
        padding: 2rem;
    }
    
    .card {
        margin-bottom: 1.5rem;
        padding: 1.5rem;
    }
}

/* Адаптация для десктопов */
@media (min-width: 1024px) {
    .container {
        max-width: 1000px;
        padding: 3rem;
    }
    
    .card {
        margin-bottom: 2rem;
        padding: 2rem;
    }
}
```

### 3. Десктопный первый подход

```css
/* Стили по умолчанию для десктопов */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 3rem;
}

.card {
    margin-bottom: 2rem;
    padding: 2rem;
}

/* Адаптация для планшетов */
@media (max-width: 1023px) {
    .container {
        max-width: 960px;
        padding: 2rem;
    }
    
    .card {
        margin-bottom: 1.5rem;
        padding: 1.5rem;
    }
}

/* Адаптация для мобильных устройств */
@media (max-width: 767px) {
    .container {
        max-width: 100%;
        padding: 1rem;
    }
    
    .card {
        margin-bottom: 1rem;
        padding: 1rem;
    }
}
```

## Медиа-запросы

### 1. Основные брейкпоинты

```css
/* Общие брейкпоинты */
/* Мобильные устройства */
@media (max-width: 767px) {
    .mobile-only { display: block; }
    .desktop-only { display: none; }
}

/* Планшеты */
@media (min-width: 768px) and (max-width: 1023px) {
    .tablet-only { display: block; }
}

/* Десктопы */
@media (min-width: 1024px) {
    .desktop-only { display: block; }
    .mobile-only { display: none; }
}

/* Большие экраны */
@media (min-width: 1200px) {
    .large-screen { display: block; }
}
```

### 2. Современные медиа-запросы

```css
/* Медиа-запросы на основе характеристик устройства */
@media (hover: hover) {
    /* Только для устройств с мышью */
    .button:hover {
        background-color: #007bff;
    }
}

@media (hover: none) {
    /* Только для сенсорных устройств */
    .button:active {
        background-color: #007bff;
    }
}

/* Ориентация экрана */
@media (orientation: landscape) {
    .landscape-only { display: block; }
}

@media (orientation: portrait) {
    .portrait-only { display: block; }
}

/* Плотность пикселей */
@media (min-resolution: 2dppx) {
    /* Для высококачественных экранов */
    .logo {
        background-image: url('logo@2x.png');
        background-size: 100px 50px;
    }
}

/* Предпочтения пользователя */
@media (prefers-reduced-motion: reduce) {
    /* Для пользователей, предпочитающих минимальные анимации */
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

@media (prefers-color-scheme: dark) {
    /* Для темной темы */
    body {
        background-color: #1a1a1a;
        color: #ffffff;
    }
}
```

### 3. Контейнерные запросы (экспериментально)

```css
/* Контейнерные запросы (требуется полифил или поддержка браузера) */
.card-container {
    container-type: inline-size;
    container-name: card;
}

@container card (min-width: 300px) {
    .card-content {
        display: grid;
        grid-template-columns: 1fr 2fr;
        gap: 1rem;
    }
}

@container card (max-width: 299px) {
    .card-content {
        display: block;
    }
}

/* Контейнерные запросы на основе стиля */
@container card (width > 400px) {
    .card-title {
        font-size: 1.5rem;
    }
}
```

## Flexbox для адаптивного дизайна

### 1. Основы Flexbox

```css
/* Простой адаптивный макет */
.flex-container {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.flex-header {
    flex: 0 0 auto;
    background: #333;
    color: white;
    padding: 1rem;
}

.flex-main {
    flex: 1 1 auto;
    padding: 1rem;
}

.flex-footer {
    flex: 0 0 auto;
    background: #f8f9fa;
    padding: 1rem;
}

/* Адаптивная навигация */
.nav {
    display: flex;
    flex-direction: column;
}

.nav-item {
    flex: 1;
    margin-bottom: 0.5rem;
}

/* Для больших экранов */
@media (min-width: 768px) {
    .nav {
        flex-direction: row;
    }
    
    .nav-item {
        margin-bottom: 0;
        margin-right: 1rem;
    }
}
```

### 2. Адаптивные карточки

```css
/* Адаптивная сетка карточек */
.card-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}

.card {
    flex: 1 1 calc(100% - 2rem); /* 1 карточка на мобильном */
    min-width: 280px;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    padding: 1rem;
}

/* 2 карточки в строке */
@media (min-width: 576px) {
    .card {
        flex: 1 1 calc(50% - 1rem);
    }
}

/* 3 карточки в строке */
@media (min-width: 768px) {
    .card {
        flex: 1 1 calc(33.333% - 1rem);
    }
}

/* 4 карточки в строке */
@media (min-width: 1024px) {
    .card {
        flex: 1 1 calc(25% - 1rem);
    }
}
```

### 3. Выравнивание и распределение

```css
/* Адаптивные формы */
.form-container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.form-row {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

/* Для больших экранов - горизонтальное расположение */
@media (min-width: 768px) {
    .form-row {
        flex-direction: row;
        align-items: flex-end;
    }
    
    .form-group {
        flex: 1;
        margin-right: 1rem;
    }
    
    .form-group:last-child {
        margin-right: 0;
    }
}

/* Адаптивные кнопки */
.button-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

/* Для больших экранов - горизонтальное расположение */
@media (min-width: 768px) {
    .button-group {
        flex-direction: row;
        justify-content: flex-end;
    }
    
    .button {
        margin-left: 0.5rem;
    }
    
    .button:first-child {
        margin-left: 0;
    }
}
```

## CSS Grid для адаптивного дизайна

### 1. Основы Grid

```css
/* Адаптивная сетка с Grid */
.grid-container {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1rem;
    padding: 1rem;
}

.grid-item {
    background: white;
    padding: 1rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* 2 колонки */
@media (min-width: 768px) {
    .grid-container {
        grid-template-columns: repeat(2, 1fr);
    }
}

/* 3 колонки */
@media (min-width: 1024px) {
    .grid-container {
        grid-template-columns: repeat(3, 1fr);
    }
}

/* 4 колонки */
@media (min-width: 1200px) {
    .grid-container {
        grid-template-columns: repeat(4, 1fr);
    }
}
```

### 2. Адаптивный макет страницы

```css
/* Сложный адаптивный макет */
.page-layout {
    display: grid;
    grid-template-areas: 
        "header"
        "nav"
        "main"
        "sidebar"
        "footer";
    grid-template-columns: 1fr;
    grid-template-rows: auto auto 1fr auto auto;
    min-height: 100vh;
    gap: 1rem;
    padding: 1rem;
}

.page-header {
    grid-area: header;
    background: #333;
    color: white;
    padding: 1rem;
}

.page-nav {
    grid-area: nav;
    background: #f8f9fa;
    padding: 1rem;
}

.page-main {
    grid-area: main;
    background: white;
    padding: 1rem;
}

.page-sidebar {
    grid-area: sidebar;
    background: #f8f9fa;
    padding: 1rem;
}

.page-footer {
    grid-area: footer;
    background: #333;
    color: white;
    padding: 1rem;
}

/* Для больших экранов */
@media (min-width: 1024px) {
    .page-layout {
        grid-template-areas: 
            "header header"
            "nav nav"
            "sidebar main"
            "footer footer";
        grid-template-columns: 250px 1fr;
        grid-template-rows: auto auto 1fr auto;
    }
}
```

### 3. Адаптивная сетка с автоматическими колонками

```css
/* Адаптивная сетка с auto-fit */
.auto-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1rem;
    padding: 1rem;
}

.auto-grid-item {
    background: white;
    padding: 1rem;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Адаптивная сетка с auto-fill */
.auto-fill-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 1rem;
    padding: 1rem;
}

/* Адаптивная сетка с max-content */
.content-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(max-content, 300px));
    gap: 1rem;
    justify-content: center;
}
```

## Современные подходы к адаптивности

### 1. Fluid Typography

```css
/* Адаптивный размер шрифта */
.fluid-title {
    font-size: clamp(1.5rem, 4vw, 3rem);
    /* min, preferred, max */
}

.fluid-text {
    font-size: clamp(1rem, 2.5vw, 1.5rem);
    line-height: 1.6;
}

/* Адаптивные отступы */
.fluid-container {
    padding: clamp(1rem, 5vw, 3rem);
    max-width: clamp(300px, 90%, 1200px);
    margin: 0 auto;
}

/* Адаптивные размеры элементов */
.fluid-card {
    padding: clamp(1rem, 3vw, 2rem);
    margin: clamp(0.5rem, 2vw, 1rem);
}
```

### 2. CSS Logical Properties

```css
/* Адаптивные отступы с логическими свойствами */
.text-container {
    margin-inline-start: 1rem;    /* Вместо margin-left */
    margin-inline-end: 1rem;      /* Вместо margin-right */
    padding-inline: 1rem;         /* Вместо padding-left и padding-right */
    text-align: start;            /* Вместо left/right */
}

/* Адаптивные размеры */
.layout-box {
    inline-size: 100%;            /* Вместо width */
    block-size: 200px;            /* Вместо height */
    min-inline-size: 300px;       /* Вместо min-width */
    max-block-size: 400px;        /* Вместо max-height */
}

/* Адаптивные границы */
.border-container {
    border-inline-start: 2px solid #ccc;  /* Вместо border-left */
    border-inline-end: 2px solid #ccc;    /* Вместо border-right */
}
```

### 3. Container Queries (экспериментально)

```css
/* Пример использования Container Queries */
.card-container {
    container-type: inline-size;
    container-name: card-container;
    width: 100%;
    max-width: 400px;
    margin: 0 auto;
}

.card {
    padding: 1rem;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card-header {
    font-size: 1.2rem;
    font-weight: bold;
    margin-bottom: 0.5rem;
}

/* Адаптация на основе размера контейнера */
@container card-container (min-width: 300px) {
    .card {
        padding: 1.5rem;
    }
    
    .card-header {
        font-size: 1.5rem;
    }
}

@container card-container (min-width: 350px) {
    .card {
        display: grid;
        grid-template-columns: 80px 1fr;
        gap: 1rem;
        align-items: start;
    }
    
    .card-image {
        grid-row: 1 / span 2;
    }
}
```

## Адаптивные изображения

### 1. Responsive Images

```html
<!-- Адаптивные изображения с srcset -->
<img 
    src="small.jpg" 
    srcset="small.jpg 480w, 
            medium.jpg 800w, 
            large.jpg 1200w, 
            extra-large.jpg 1600w"
    sizes="(max-width: 480px) 100vw, 
           (max-width: 800px) 50vw, 
           (max-width: 1200px) 33vw, 
           25vw"
    alt="Адаптивное изображение"
    loading="lazy">

<!-- Использование picture для разных форматов -->
<picture>
    <source srcset="image.webp" type="image/webp">
    <source srcset="image.avif" type="image/avif">
    <img src="image.jpg" alt="Изображение" loading="lazy">
</picture>
```

### 2. Адаптивные фоны

```css
/* Адаптивные фоны */
.hero-section {
    background-image: url('hero-mobile.jpg');
    background-size: cover;
    background-position: center;
    height: 400px;
    display: flex;
    align-items: center;
    justify-content: center;
}

/* Для планшетов */
@media (min-width: 768px) {
    .hero-section {
        background-image: url('hero-tablet.jpg');
        height: 500px;
    }
}

/* Для десктопов */
@media (min-width: 1024px) {
    .hero-section {
        background-image: url('hero-desktop.jpg');
        height: 600px;
    }
}

/* Адаптивный фон с object-fit */
.adaptive-image {
    width: 100%;
    height: 300px;
    object-fit: cover;
    object-position: center;
}

/* Адаптивный фон с градиентом */
.gradient-bg {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    background-size: 400% 400%;
    animation: gradientShift 4s ease infinite;
}

@keyframes gradientShift {
    0% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
    100% { background-position: 0% 50%; }
}
```

## Адаптивные формы

### 1. Адаптивные формы

```html
<form class="responsive-form">
    <div class="form-grid">
        <div class="form-group">
            <label for="first-name">Имя</label>
            <input type="text" id="first-name" name="first-name" required>
        </div>
        
        <div class="form-group">
            <label for="last-name">Фамилия</label>
            <input type="text" id="last-name" name="last-name" required>
        </div>
    </div>
    
    <div class="form-group">
        <label for="email">Email</label>
        <input type="email" id="email" name="email" required>
    </div>
    
    <div class="form-group">
        <label for="message">Сообщение</label>
        <textarea id="message" name="message" rows="4"></textarea>
    </div>
    
    <button type="submit" class="btn btn-primary">Отправить</button>
</form>
```

```css
/* Стили для адаптивной формы */
.responsive-form {
    max-width: 600px;
    margin: 0 auto;
    padding: 1rem;
}

.form-grid {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.form-group label {
    font-weight: 500;
    color: #333;
}

.form-group input,
.form-group textarea {
    padding: 0.75rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1rem;
}

.form-group input:focus,
.form-group textarea:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

/* Для планшетов и выше */
@media (min-width: 768px) {
    .form-grid {
        flex-direction: row;
    }
    
    .form-group {
        flex: 1;
    }
}

/* Адаптивные кнопки */
.btn {
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.btn-primary {
    background-color: #007bff;
    color: white;
}

.btn-primary:hover {
    background-color: #0056b3;
}
```

### 2. Адаптивные чекбоксы и радио-кнопки

```css
/* Адаптивные кастомные чекбоксы */
.custom-checkbox {
    position: relative;
    display: inline-flex;
    align-items: center;
    cursor: pointer;
    margin-bottom: 1rem;
}

.custom-checkbox input {
    opacity: 0;
    position: absolute;
    width: 0;
    height: 0;
}

.checkmark {
    width: 20px;
    height: 20px;
    border: 2px solid #ddd;
    border-radius: 4px;
    margin-right: 0.5rem;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
}

.custom-checkbox input:checked + .checkmark {
    background-color: #007bff;
    border-color: #007bff;
}

.custom-checkbox input:checked + .checkmark::after {
    content: '✓';
    color: white;
    font-size: 12px;
}

/* Для мобильных устройств */
@media (max-width: 767px) {
    .custom-checkbox {
        margin-bottom: 0.75rem;
    }
    
    .checkmark {
        width: 18px;
        height: 18px;
    }
}
```

## Адаптивные таблицы

### 1. Адаптивные таблицы

```html
<div class="table-container">
    <table class="responsive-table">
        <thead>
            <tr>
                <th>Имя</th>
                <th>Email</th>
                <th>Возраст</th>
                <th>Город</th>
                <th>Действия</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Иван Иванов</td>
                <td>ivan@example.com</td>
                <td>30</td>
                <td>Москва</td>
                <td>
                    <button class="btn btn-sm">Изменить</button>
                    <button class="btn btn-sm btn-danger">Удалить</button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
```

```css
/* Адаптивные таблицы */
.table-container {
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
}

.responsive-table {
    width: 100%;
    border-collapse: collapse;
    background: white;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.responsive-table th,
.responsive-table td {
    padding: 0.75rem;
    text-align: left;
    border-bottom: 1px solid #eee;
}

.responsive-table th {
    background-color: #f8f9fa;
    font-weight: 600;
    color: #333;
}

/* Для мобильных устройств - превращаем таблицу в карточки */
@media (max-width: 767px) {
    .table-container {
        overflow-x: visible;
    }
    
    .responsive-table,
    .responsive-table thead,
    .responsive-table tbody,
    .responsive-table th,
    .responsive-table td,
    .responsive-table tr {
        display: block;
    }
    
    .responsive-table thead tr {
        position: absolute;
        top: -9999px;
        left: -9999px;
    }
    
    .responsive-table tr {
        border: 1px solid #ccc;
        margin-bottom: 1rem;
        border-radius: 8px;
        overflow: hidden;
    }
    
    .responsive-table tr:last-child {
        margin-bottom: 0;
    }
    
    .responsive-table td {
        border: none;
        position: relative;
        padding-left: 50%;
        text-align: right;
    }
    
    .responsive-table td:before {
        content: attr(data-label) ": ";
        position: absolute;
        left: 0.75rem;
        width: 45%;
        text-align: left;
        font-weight: 600;
        color: #333;
    }
}
```

## Продвинутые техники

### 1. Адаптивная навигация

```html
<nav class="responsive-nav">
    <div class="nav-brand">
        <a href="/">Логотип</a>
    </div>
    
    <button class="nav-toggle" aria-label="Переключить навигацию">
        <span class="hamburger"></span>
    </button>
    
    <ul class="nav-menu">
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
        <li><a href="/services">Услуги</a></li>
        <li><a href="/contact">Контакты</a></li>
    </ul>
</nav>
```

```css
/* Адаптивная навигация */
.responsive-nav {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    background: white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    position: relative;
}

.nav-brand a {
    font-size: 1.5rem;
    font-weight: bold;
    text-decoration: none;
    color: #333;
}

.nav-toggle {
    display: none;
    flex-direction: column;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0.5rem;
}

.hamburger {
    width: 25px;
    height: 3px;
    background: #333;
    margin: 3px 0;
    transition: 0.3s;
    display: block;
}

/* Для мобильных устройств */
@media (max-width: 767px) {
    .nav-toggle {
        display: flex;
    }
    
    .nav-menu {
        position: fixed;
        left: -100%;
        top: 70px;
        flex-direction: column;
        background-color: white;
        width: 100%;
        text-align: center;
        transition: 0.3s;
        box-shadow: 0 10px 27px rgba(0,0,0,0.05);
        height: calc(100vh - 70px);
        overflow-y: auto;
    }
    
    .nav-menu.active {
        left: 0;
    }
    
    .nav-menu li {
        margin: 0;
    }
    
    .nav-menu a {
        padding: 1.5rem;
        display: block;
        border-bottom: 1px solid #eee;
    }
}

/* Для планшетов и выше */
@media (min-width: 768px) {
    .nav-menu {
        display: flex;
        list-style: none;
        margin: 0;
        padding: 0;
    }
    
    .nav-menu li {
        margin-left: 2rem;
    }
    
    .nav-menu li:first-child {
        margin-left: 0;
    }
    
    .nav-menu a {
        text-decoration: none;
        color: #333;
        font-weight: 500;
        transition: color 0.2s ease;
    }
    
    .nav-menu a:hover {
        color: #007bff;
    }
}
```

### 2. Адаптивная галерея

```html
<div class="gallery">
    <div class="gallery-item">
        <img src="image1.jpg" alt="Изображение 1">
    </div>
    <div class="gallery-item">
        <img src="image2.jpg" alt="Изображение 2">
    </div>
    <div class="gallery-item">
        <img src="image3.jpg" alt="Изображение 3">
    </div>
</div>
```

```css
/* Адаптивная галерея */
.gallery {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1rem;
    padding: 1rem;
}

.gallery-item {
    position: relative;
    overflow: hidden;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    aspect-ratio: 1/1;
}

.gallery-item img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.3s ease;
}

.gallery-item:hover img {
    transform: scale(1.05);
}

/* Для очень маленьких экранов */
@media (max-width: 480px) {
    .gallery {
        grid-template-columns: 1fr;
        gap: 0.5rem;
    }
}

/* Для больших экранов */
@media (min-width: 1200px) {
    .gallery {
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 1.5rem;
    }
}
```

## Лучшие практики адаптивного дизайна

### 1. Структура проекта

```css
/* Пример структуры CSS для адаптивного дизайна */
/* variables.css */
:root {
    --breakpoint-sm: 576px;
    --breakpoint-md: 768px;
    --breakpoint-lg: 992px;
    --breakpoint-xl: 1200px;
    --breakpoint-xxl: 1400px;
    
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.25rem;
    --font-size-xl: 1.5rem;
    
    --spacing-xs: 0.25rem;
    --spacing-sm: 0.5rem;
    --spacing-md: 1rem;
    --spacing-lg: 1.5rem;
    --spacing-xl: 2rem;
}

/* mixins.css или в препроцессоре */
/* Миксин для медиа-запросов */
@custom-media --viewport-sm (min-width: 576px);
@custom-media --viewport-md (min-width: 768px);
@custom-media --viewport-lg (min-width: 992px);
@custom-media --viewport-xl (min-width: 1200px);

/* Использование */
.card {
    padding: var(--spacing-md);
    
    @media (--viewport-md) {
        padding: var(--spacing-lg);
    }
}
```

### 2. Тестирование адаптивности

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Тест адаптивности</title>
    <style>
        /* Основные стили */
        body {
            margin: 0;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            line-height: 1.6;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 1rem;
        }
        
        /* Индикатор размера экрана */
        .screen-size-indicator {
            position: fixed;
            top: 10px;
            right: 10px;
            background: rgba(0,0,0,0.8);
            color: white;
            padding: 0.5rem;
            border-radius: 4px;
            font-size: 0.8rem;
            z-index: 1000;
        }
        
        /* Скрытие индикатора при печати */
        @media print {
            .screen-size-indicator { display: none; }
        }
    </style>
</head>
<body>
    <div class="screen-size-indicator" id="screenSize"></div>
    
    <div class="container">
        <h1>Тест адаптивности</h1>
        <p>Эта страница используется для тестирования адаптивности дизайна.</p>
    </div>
    
    <script>
        function updateScreenSize() {
            const width = window.innerWidth;
            const height = window.innerHeight;
            const indicator = document.getElementById('screenSize');
            
            let deviceType = 'Неизвестно';
            if (width < 768) deviceType = 'Мобильный';
            else if (width < 1024) deviceType = 'Планшет';
            else deviceType = 'Десктоп';
            
            indicator.textContent = `${width}×${height} (${deviceType})`;
        }
        
        // Обновляем при загрузке и изменении размера
        window.addEventListener('load', updateScreenSize);
        window.addEventListener('resize', updateScreenSize);
    </script>
</body>
</html>
```

## Примеры из промышленной разработки

### 1. Адаптивный макет электронной коммерции

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивный интернет-магазин</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            line-height: 1.6;
        }
        
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 1rem;
            background: #333;
            color: white;
        }
        
        .logo {
            font-size: 1.5rem;
            font-weight: bold;
        }
        
        .nav {
            display: none;
        }
        
        .mobile-menu-btn {
            background: none;
            border: none;
            color: white;
            font-size: 1.5rem;
            cursor: pointer;
        }
        
        .hero {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            text-align: center;
            padding: 3rem 1rem;
        }
        
        .hero h1 {
            font-size: clamp(1.5rem, 5vw, 2.5rem);
            margin-bottom: 1rem;
        }
        
        .hero p {
            font-size: clamp(1rem, 3vw, 1.2rem);
            margin-bottom: 2rem;
            opacity: 0.9;
        }
        
        .btn {
            display: inline-block;
            padding: 0.75rem 1.5rem;
            background: white;
            color: #667eea;
            text-decoration: none;
            border-radius: 4px;
            font-weight: 600;
            transition: all 0.2s ease;
        }
        
        .btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        
        .products-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 2rem;
            padding: 2rem 1rem;
            max-width: 1200px;
            margin: 0 auto;
        }
        
        .product-card {
            background: white;
            border-radius: 8px;
            overflow: hidden;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: transform 0.2s ease;
        }
        
        .product-card:hover {
            transform: translateY(-4px);
        }
        
        .product-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
        }
        
        .product-info {
            padding: 1.5rem;
        }
        
        .product-title {
            font-size: 1.25rem;
            margin-bottom: 0.5rem;
        }
        
        .product-price {
            font-size: 1.5rem;
            font-weight: bold;
            color: #e74c3c;
            margin-bottom: 1rem;
        }
        
        .footer {
            background: #f8f9fa;
            padding: 2rem 1rem;
            text-align: center;
        }
        
        /* Адаптация для планшетов и десктопов */
        @media (min-width: 768px) {
            .header {
                padding: 1rem 2rem;
            }
            
            .nav {
                display: flex;
                list-style: none;
            }
            
            .nav li {
                margin-left: 2rem;
            }
            
            .nav a {
                color: white;
                text-decoration: none;
            }
            
            .mobile-menu-btn {
                display: none;
            }
            
            .hero {
                padding: 5rem 2rem;
            }
            
            .products-grid {
                padding: 3rem 2rem;
                gap: 3rem;
            }
        }
        
        @media (min-width: 1024px) {
            .products-grid {
                grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            }
        }
    </style>
</head>
<body>
    <header class="header">
        <div class="logo">Магазин</div>
        <nav class="nav">
            <ul>
                <li><a href="#">Главная</a></li>
                <li><a href="#">Каталог</a></li>
                <li><a href="#">О нас</a></li>
                <li><a href="#">Контакты</a></li>
            </ul>
        </nav>
        <button class="mobile-menu-btn">☰</button>
    </header>
    
    <section class="hero">
        <h1>Сезонные скидки до 50%</h1>
        <p>Только до конца месяца</p>
        <a href="#" class="btn">Посмотреть товары</a>
    </section>
    
    <section class="products">
        <div class="products-grid">
            <div class="product-card">
                <img src="https://placehold.co/300x200" alt="Товар 1" class="product-image">
                <div class="product-info">
                    <h3 class="product-title">Товар 1</h3>
                    <div class="product-price">1 990 ₽</div>
                    <a href="#" class="btn">В корзину</a>
                </div>
            </div>
            
            <div class="product-card">
                <img src="https://placehold.co/300x200" alt="Товар 2" class="product-image">
                <div class="product-info">
                    <h3 class="product-title">Товар 2</h3>
                    <div class="product-price">2 490 ₽</div>
                    <a href="#" class="btn">В корзину</a>
                </div>
            </div>
            
            <div class="product-card">
                <img src="https://placehold.co/300x200" alt="Товар 3" class="product-image">
                <div class="product-info">
                    <h3 class="product-title">Товар 3</h3>
                    <div class="product-price">1 290 ₽</div>
                    <a href="#" class="btn">В корзину</a>
                </div>
            </div>
        </div>
    </section>
    
    <footer class="footer">
        <p>&copy; 2023 Интернет-магазин. Все права защищены.</p>
    </footer>
</body>
</html>
```

## Ссылки на связанные темы
- [[best-practices/best-practices]] - Лучшие практики CSS
- [[layout/flexbox-comprehensive]] - Flexbox
- [[layout/grid-comprehensive]] - CSS Grid
- [[html/responsive]] - Адаптивный HTML

#programming #css #responsive #mobile-first #flexbox #grid #media-queries #best-practices
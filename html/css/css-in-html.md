# CSS в HTML: Обзор

CSS (Cascading Style Sheets) — это язык таблиц стилей, который описывает внешний вид документов, написанных на HTML. CSS позволяет отделить содержимое документа от его представления, что делает веб-разработку более гибкой и управляемой.

## Что такое CSS?

CSS — это технология, которая используется для описания внешнего вида HTML-документа. Она управляет:

- **Цветами** — текста, фона, границ
- **Шрифтами** — типографикой, размерами, начертанием
- **Разметкой** — расположением элементов на странице
- **Анимациями** — переходами и анимациями элементов
- **Адаптивностью** — адаптацией под разные устройства

## Подключение CSS к HTML

### 1. Встроенные стили (Inline Styles)

```html
<p style="color: blue; font-size: 16px;">Этот текст синий и 16 пикселей</p>
```

### 2. Внутренние стили (Internal Stylesheet)

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
        }
        
        h1 {
            color: #333;
            text-align: center;
        }
        
        p {
            font-size: 16px;
            line-height: 1.5;
        }
    </style>
</head>
<body>
    <h1>Заголовок</h1>
    <p>Параграф текста</p>
</body>
</html>
```

### 3. Внешние стили (External Stylesheet)

Создание файла `styles.css`:

```css
/* styles.css */
body {
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
    margin: 0;
    padding: 20px;
}

h1 {
    color: #333;
    text-align: center;
    border-bottom: 2px solid #007acc;
}

p {
    font-size: 16px;
    line-height: 1.5;
    color: #666;
}

.container {
    max-width: 800px;
    margin: 0 auto;
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}
```

Подключение в HTML:

```html
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="styles.css">
    <title>Внешние стили</title>
</head>
<body>
    <div class="container">
        <h1>Заголовок</h1>
        <p>Параграф текста</p>
    </div>
</body>
</html>
```

## Приоритеты CSS (Каскад)

CSS следует принципу каскада, где стили применяются в следующем порядке приоритета (от наименьшего к наибольшему):

1. **Браузерные стили** (user agent styles)
2. **Пользовательские стили** (user styles)
3. **Авторские стили** (author styles)
4. **!important** объявления
5. **Встроенные стили** (inline styles)

```css
/* Пример приоритетов */
p {
    color: blue; /* Низкий приоритет */
}

p {
    color: red !important; /* Наивысший приоритет */
}

/* В HTML: <p style="color: green;">Текст</p> - будет зеленым */
```

## Селекторы CSS

Селекторы определяют, к каким элементам будут применяться стили:

```css
/* Селектор элемента */
p {
    color: #333;
}

/* Селектор класса */
.warning {
    color: #d32f2f;
    background-color: #ffebee;
    padding: 10px;
    border-radius: 4px;
}

/* Селектор ID */
#main-header {
    font-size: 2em;
    color: #007acc;
}

/* Селектор потомка */
.nav ul li {
    display: inline-block;
    margin-right: 15px;
}

/* Селектор псевдокласса */
a:hover {
    text-decoration: underline;
    color: #005a9e;
}

/* Селектор атрибута */
input[type="email"] {
    border: 2px solid #007acc;
    padding: 8px;
}
```

## Свойства CSS

### Цвета и фоны

```css
.element {
    color: #333; /* Цвет текста */
    background-color: #f5f5f5; /* Цвет фона */
    background-image: url('image.jpg'); /* Фоновое изображение */
    background-repeat: no-repeat; /* Не повторять фон */
    background-position: center; /* Позиционирование фона */
    background-size: cover; /* Размер фона */
}
```

### Шрифты и текст

```css
.text {
    font-family: Arial, sans-serif; /* Семейство шрифта */
    font-size: 16px; /* Размер шрифта */
    font-weight: bold; /* Начертание (жирность) */
    font-style: italic; /* Курсив */
    text-align: center; /* Выравнивание текста */
    text-decoration: underline; /* Подчеркивание */
    line-height: 1.5; /* Высота строки */
    letter-spacing: 1px; /* Расстояние между буквами */
}
```

### Блочная модель

```css
.box {
    width: 200px; /* Ширина */
    height: 100px; /* Высота */
    padding: 20px; /* Внутренние отступы */
    border: 2px solid #333; /* Граница */
    margin: 10px; /* Внешние отступы */
    box-sizing: border-box; /* Включать padding и border в ширину */
}
```

### Позиционирование

```css
.positioned {
    position: relative; /* Относительное позиционирование */
    top: 10px;
    left: 10px;
}

.absolute-position {
    position: absolute; /* Абсолютное позиционирование */
    top: 0;
    right: 0;
}

.fixed-position {
    position: fixed; /* Фиксированное позиционирование */
    bottom: 20px;
    right: 20px;
}

.sticky-position {
    position: sticky; /* Липкое позиционирование */
    top: 0;
}
```

## Современные возможности CSS

### Flexbox

```css
.flex-container {
    display: flex;
    justify-content: space-between; /* Распределение по основной оси */
    align-items: center; /* Распределение по поперечной оси */
    flex-wrap: wrap; /* Перенос строк */
}

.flex-item {
    flex: 1; /* Растяжение элемента */
    flex-grow: 1; /* Коэффициент роста */
    flex-shrink: 1; /* Коэффициент сжатия */
    flex-basis: auto; /* Базовый размер */
}
```

### Grid

```css
.grid-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 3 равные колонки */
    grid-template-rows: auto 1fr auto; /* Шаблон строк */
    grid-gap: 20px; /* Отступы между ячейками */
    grid-template-areas: 
        "header header header"
        "main sidebar aside"
        "footer footer footer"; /* Именованные области */
}

.header { grid-area: header; }
.main { grid-area: main; }
.sidebar { grid-area: sidebar; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

### Media Queries

```css
/* Адаптивный дизайн */
.container {
    width: 100%;
    padding: 20px;
}

/* Стили для планшетов */
@media (max-width: 768px) {
    .container {
        padding: 15px;
    }
    
    .grid-container {
        grid-template-columns: 1fr;
    }
}

/* Стили для мобильных устройств */
@media (max-width: 480px) {
    .container {
        padding: 10px;
    }
    
    h1 {
        font-size: 1.5em;
    }
}

/* Стили для печати */
@media print {
    .no-print {
        display: none;
    }
    
    body {
        font-size: 12pt;
    }
}
```

## Анимации и переходы

### Transition

```css
.button {
    background-color: #007acc;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
}

.button:hover {
    background-color: #005a9e;
    transform: translateY(-2px);
}
```

### Animation

```css
.loading {
    width: 40px;
    height: 40px;
    border: 4px solid #f3f3f3;
    border-top: 4px solid #007acc;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
```

## Практические примеры

### Адаптивный макет

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивный макет</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            line-height: 1.6;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 20px;
        }

        header {
            background-color: #333;
            color: white;
            padding: 1rem 0;
        }

        nav ul {
            display: flex;
            list-style: none;
            justify-content: center;
        }

        nav li {
            margin: 0 15px;
        }

        nav a {
            color: white;
            text-decoration: none;
        }

        .main-content {
            display: grid;
            grid-template-columns: 2fr 1fr;
            gap: 30px;
            margin: 30px 0;
        }

        .article {
            background-color: #f9f9f9;
            padding: 20px;
            border-radius: 8px;
        }

        .sidebar {
            background-color: #e9e9e9;
            padding: 20px;
            border-radius: 8px;
        }

        footer {
            background-color: #333;
            color: white;
            text-align: center;
            padding: 1rem 0;
            margin-top: 30px;
        }

        @media (max-width: 768px) {
            .main-content {
                grid-template-columns: 1fr;
            }

            nav ul {
                flex-direction: column;
                align-items: center;
            }

            nav li {
                margin: 5px 0;
            }
        }
    </style>
</head>
<body>
    <header>
        <div class="container">
            <nav>
                <ul>
                    <li><a href="#">Главная</a></li>
                    <li><a href="#">О нас</a></li>
                    <li><a href="#">Услуги</a></li>
                    <li><a href="#">Контакты</a></li>
                </ul>
            </nav>
        </div>
    </header>

    <div class="container">
        <div class="main-content">
            <main>
                <article class="article">
                    <h2>Основная статья</h2>
                    <p>Содержимое основной статьи...</p>
                </article>
            </main>
            <aside class="sidebar">
                <h3>Боковая панель</h3>
                <p>Содержимое боковой панели...</p>
            </aside>
        </div>
    </div>

    <footer>
        <div class="container">
            <p>&copy; 2023 Все права защищены</p>
        </div>
    </footer>
</body>
</html>
```

## Лучшие практики

### 1. Организация CSS

```css
/* 1. Сброс и нормализация */
*, *::before, *::after {
    box-sizing: border-box;
}

/* 2. Переменные */
:root {
    --primary-color: #007acc;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --font-size-base: 16px;
    --border-radius: 4px;
    --box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* 3. Базовые стили */
body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    font-size: var(--font-size-base);
    line-height: 1.6;
    color: #333;
}

/* 4. Компоненты */
.btn {
    display: inline-block;
    padding: 8px 16px;
    border: none;
    border-radius: var(--border-radius);
    cursor: pointer;
    text-decoration: none;
    font-size: 1rem;
}

.btn-primary {
    background-color: var(--primary-color);
    color: white;
}

.btn-primary:hover {
    background-color: #005a9e;
}

/* 5. Утилиты */
.text-center { text-align: center; }
.mt-1 { margin-top: 0.25rem; }
.mb-2 { margin-bottom: 0.5rem; }
.d-none { display: none; }
.visually-hidden {
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
```

### 2. Методологии CSS

#### БЭМ (Блок-Элемент-Модификатор)

```css
/* Блок */
.menu { }

/* Элемент */
.menu__item { }

/* Модификатор */
.menu__item--active { }

.menu__item--disabled { }
```

### 3. Доступность

```css
/* Фокус индикаторы */
a:focus,
button:focus,
input:focus,
select:focus,
textarea:focus {
    outline: 2px solid var(--primary-color);
    outline-offset: 2px;
}

/* Пользовательские фокус индикаторы */
.custom-focus:focus-visible {
    outline: 2px solid var(--primary-color);
    outline-offset: 2px;
    box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.3);
}

/* Уменьшенная анимация для пользователей */
@media (prefers-reduced-motion: reduce) {
    *,
    *::before,
    *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

/* Высокий контраст для пользователей */
@media (prefers-contrast: high) {
    body {
        background-color: white;
        color: black;
    }
    
    .btn {
        border: 2px solid black;
    }
}
```

## Современные возможности CSS

### CSS Custom Properties (Переменные)
```css
:root {
    /* Определение переменных */
    --primary-color: #007acc;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --warning-color: #ffc107;
    --info-color: #17a2b8;
    
    --font-size-base: 16px;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.25rem;
    --font-size-xl: 1.5rem;
    
    --border-radius: 0.25rem;
    --border-radius-lg: 0.3rem;
    --border-radius-sm: 0.2rem;
    
    --shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    --shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
    --shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);
    
    --transition-base: all 0.2s ease-in-out;
    --transition-fast: all 0.1s ease-in-out;
}

/* Использование переменных */
.button {
    background-color: var(--primary-color);
    color: white;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: var(--border-radius);
    transition: var(--transition-base);
}

.button:hover {
    background-color: #005a9e; /* Темнее основного цвета */
    transform: translateY(-2px);
    box-shadow: var(--shadow);
}

/* Темы с переменными */
.light-theme {
    --background-color: #ffffff;
    --text-color: #333333;
    --border-color: #dddddd;
}

.dark-theme {
    --background-color: #2d2d2d;
    --text-color: #e0e0e0;
    --border-color: #555555;
}

.component {
    background-color: var(--background-color);
    color: var(--text-color);
    border: 1px solid var(--border-color);
}
```

### CSS Functions (calc, clamp, min, max)
```css
/* calc() - вычисления в CSS */
.responsive-container {
    width: calc(100% - 40px); /* 100% ширины минус 40px отступов */
    margin: calc(10vh - 20px) auto; /* Вертикальный отступ */
    font-size: calc(16px + 0.5vw); /* Адаптивный размер шрифта */
}

/* clamp() - ограничение значений */
.adaptive-heading {
    /* clamp(минимум, предпочтительное, максимум) */
    font-size: clamp(1.2rem, 2.5vw, 2.5rem);
}

.adaptive-padding {
    padding: clamp(1rem, 3vw, 3rem);
}

/* min() и max() */
.adaptive-width {
    width: min(100%, 600px); /* Не больше 600px, но не больше 100% ширины */
    height: max(200px, 30vh); /* Не меньше 200px или 30% высоты окна */
}

/* Комбинация функций */
.complex-calculation {
    width: calc(clamp(300px, 50%, 800px) - 2rem);
    padding: calc(max(1rem, 2vw) / 2);
}
```

### CSS Grid Layout
```css
/* Основы Grid */
.grid-container {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 3 равные колонки */
    grid-template-rows: auto 1fr auto; /* Шапка, контент, подвал */
    grid-gap: 20px; /* Отступы между ячейками */
    height: 100vh; /* Высота на весь экран */
}

/* Именованные области */
.layout-grid {
    display: grid;
    grid-template-areas: 
        "header header header"
        "sidebar main aside" 
        "footer footer footer";
    grid-template-columns: 200px 1fr 200px;
    grid-template-rows: auto 1fr auto;
    min-height: 100vh;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }

/* Адаптивная Grid */
.responsive-grid {
    display: grid;
    /* Автоматическое количество колонок в зависимости от доступного места */
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
}

/* Grid с автоматическим размещением */
.auto-grid {
    display: grid;
    grid-auto-flow: dense; /* Плотное размещение элементов */
    grid-template-columns: repeat(4, 200px);
    grid-auto-rows: minmax(100px, auto);
}

.item-span-2 {
    grid-column: span 2; /* Занимает 2 колонки */
}

.item-area {
    grid-area: 1 / 1 / 3 / 3; /* row-start / col-start / row-end / col-end */
}
```

### Flexbox
```css
/* Основы Flexbox */
.flex-container {
    display: flex;
    flex-direction: row; /* или column */
    justify-content: space-between; /* Распределение по основной оси */
    align-items: center; /* Распределение по поперечной оси */
    flex-wrap: wrap; /* Перенос строк */
    gap: 10px; /* Отступы между элементами */
}

.flex-item {
    flex: 1; /* flex-grow: 1, flex-shrink: 1, flex-basis: 0% */
    flex-grow: 1; /* Коэффициент роста */
    flex-shrink: 0; /* Коэффициент сжатия */
    flex-basis: auto; /* Базовый размер */
}

/* Адаптивный навбар */
.navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
}

.nav-links {
    display: flex;
    gap: 2rem;
    list-style: none;
}

.nav-item {
    flex: 0 0 auto; /* Не растягивается, не сжимается, базовый размер по контенту */
}

/* Центрирование элемента */
.centered-content {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}
```

### Container Queries (современная возможность)
```css
/* Определение контейнера */
.card-container {
    container-type: inline-size;
    container-name: card-container;
}

/* Стили, применяемые когда контейнер достигает определенного размера */
@container card-container (min-width: 400px) {
    .card {
        display: flex;
        flex-direction: row;
    }
    
    .card-image {
        flex: 0 0 150px;
    }
    
    .card-content {
        flex: 1;
    }
}

@container card-container (min-width: 600px) {
    .card {
        flex-direction: row-reverse;
    }
}

/* Вложенные container queries */
@container (min-width: 300px) {
    @container (orientation: landscape) {
        .adaptive-component {
            flex-direction: row;
        }
    }
}
```

### Media Queries и условия
```css
/* Традиционные медиа-запросы */
@media (max-width: 768px) {
    .responsive-element {
        font-size: 0.9rem;
    }
}

/* Современные медиа-условия */
@media (hover: hover) {
    /* Только для устройств с мышью */
    .hover-effect:hover {
        transform: scale(1.05);
    }
}

@media (pointer: fine) {
    /* Для точного указания (мышь, стилус) */
    .precise-controls {
        padding: 4px;
    }
}

@media (pointer: coarse) {
    /* Для грубого указания (сенсорный экран) */
    .touch-controls {
        padding: 12px;
    }
}

@media (prefers-reduced-motion: reduce) {
    /* Для пользователей, предпочитающих уменьшенную анимацию */
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
}

@media (prefers-color-scheme: dark) {
    /* Для темного режима */
    body {
        background-color: #121212;
        color: #e0e0e0;
    }
}

@media (prefers-contrast: high) {
    /* Для высокого контраста */
    body {
        background-color: white;
        color: black;
    }
    
    .button {
        border: 2px solid black;
    }
}

@media print {
    /* Для печати */
    .no-print {
        display: none;
    }
    
    body {
        font-size: 12pt;
        color: black;
        background: white;
    }
}
```

## Практические примеры современных возможностей

### Адаптивная сетка карточек
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Современная адаптивная сетка</title>
    <style>
        :root {
            --primary-color: #007acc;
            --secondary-color: #6c757d;
            --card-bg: #ffffff;
            --card-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            --border-radius: 8px;
            --gap: 20px;
        }

        body {
            margin: 0;
            padding: var(--gap);
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background-color: #f5f5f5;
        }

        .cards-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
            gap: var(--gap);
            max-width: 1400px;
            margin: 0 auto;
        }

        .card {
            background: var(--card-bg);
            border-radius: var(--border-radius);
            box-shadow: var(--card-shadow);
            overflow: hidden;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
        }

        .card-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
        }

        .card-content {
            padding: 1.5rem;
        }

        .card-title {
            margin: 0 0 0.5rem 0;
            font-size: clamp(1.1rem, 2.5vw, 1.5rem);
            color: #333;
        }

        .card-description {
            color: #666;
            line-height: 1.6;
            margin-bottom: 1rem;
        }

        .card-meta {
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 0.9rem;
            color: #999;
        }

        .card-button {
            display: inline-block;
            padding: 0.5rem 1rem;
            background-color: var(--primary-color);
            color: white;
            text-decoration: none;
            border-radius: 4px;
            transition: background-color 0.3s;
        }

        .card-button:hover {
            background-color: #005a9e;
        }

        /* Адаптация для разных размеров экрана */
        @media (min-width: 1200px) {
            .cards-grid {
                grid-template-columns: repeat(4, 1fr);
            }
        }

        @media (min-width: 992px) and (max-width: 1199px) {
            .cards-grid {
                grid-template-columns: repeat(3, 1fr);
            }
        }

        @media (min-width: 768px) and (max-width: 991px) {
            .cards-grid {
                grid-template-columns: repeat(2, 1fr);
            }
        }

        @container (min-width: 350px) {
            .card-content {
                display: flex;
                flex-direction: column;
            }
        }
    </style>
</head>
<body>
    <h1>Современная адаптивная сетка</h1>
    
    <div class="cards-grid">
        <div class="card">
            <img src="https://via.placeholder.com/300x200/FF6B6B/FFFFFF?text=1" alt="Карточка 1" class="card-image">
            <div class="card-content">
                <h3 class="card-title">Заголовок карточки 1</h3>
                <p class="card-description">Описание содержимого первой карточки с краткой информацией о представленном элементе.</p>
                <div class="card-meta">
                    <span>20.11.2023</span>
                    <a href="#" class="card-button">Подробнее</a>
                </div>
            </div>
        </div>
        
        <div class="card">
            <img src="https://via.placeholder.com/300x200/4ECDC4/FFFFFF?text=2" alt="Карточка 2" class="card-image">
            <div class="card-content">
                <h3 class="card-title">Заголовок карточки 2</h3>
                <p class="card-description">Описание содержимого второй карточки с краткой информацией о представленном элементе.</p>
                <div class="card-meta">
                    <span>19.11.2023</span>
                    <a href="#" class="card-button">Подробнее</a>
                </div>
            </div>
        </div>
        
        <div class="card">
            <img src="https://via.placeholder.com/300x200/45B7D1/FFFFFF?text=3" alt="Карточка 3" class="card-image">
            <div class="card-content">
                <h3 class="card-title">Заголовок карточки 3</h3>
                <p class="card-description">Описание содержимого третьей карточки с краткой информацией о представленном элементе.</p>
                <div class="card-meta">
                    <span>18.11.2023</span>
                    <a href="#" class="card-button">Подробнее</a>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

### Адаптивная навигация с современными возможностями
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Современная адаптивная навигация</title>
    <style>
        :root {
            --nav-bg: #333;
            --nav-text: #fff;
            --nav-hover: #555;
            --nav-active: #007acc;
            --nav-height: 60px;
        }

        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Arial, sans-serif;
            padding-top: var(--nav-height);
        }

        .navbar {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            height: var(--nav-height);
            background-color: var(--nav-bg);
            display: flex;
            align-items: center;
            padding: 0 1rem;
            z-index: 1000;
        }

        .nav-brand {
            color: var(--nav-text);
            text-decoration: none;
            font-size: 1.2rem;
            font-weight: bold;
        }

        .nav-menu {
            display: flex;
            list-style: none;
            margin: 0;
            padding: 0;
            margin-left: auto;
        }

        .nav-item {
            margin-left: 1.5rem;
        }

        .nav-link {
            color: var(--nav-text);
            text-decoration: none;
            padding: 0.5rem 1rem;
            border-radius: 4px;
            transition: background-color 0.3s;
        }

        .nav-link:hover {
            background-color: var(--nav-hover);
        }

        .nav-link.active {
            background-color: var(--nav-active);
        }

        /* Мобильное меню */
        .nav-toggle {
            display: none;
            flex-direction: column;
            cursor: pointer;
            padding: 0.5rem;
        }

        .nav-toggle span {
            width: 25px;
            height: 3px;
            background-color: var(--nav-text);
            margin: 3px 0;
            transition: 0.3s;
        }

        @media (max-width: 768px) {
            .nav-menu {
                position: fixed;
                left: -100%;
                top: var(--nav-height);
                flex-direction: column;
                background-color: var(--nav-bg);
                width: 100%;
                text-align: center;
                transition: 0.3s;
                box-shadow: 0 10px 27px rgba(0,0,0,0.05);
            }

            .nav-menu.active {
                left: 0;
            }

            .nav-item {
                margin: 0;
            }

            .nav-link {
                display: block;
                padding: 1rem;
                border-bottom: 1px solid rgba(255,255,255,0.1);
            }

            .nav-toggle {
                display: flex;
            }

            .nav-toggle.active span:nth-child(1) {
                transform: rotate(-45deg) translate(-5px, 6px);
            }

            .nav-toggle.active span:nth-child(2) {
                opacity: 0;
            }

            .nav-toggle.active span:nth-child(3) {
                transform: rotate(45deg) translate(-5px, -6px);
            }
        }

        /* Контент */
        .content {
            padding: 2rem;
        }

        .content-section {
            margin-bottom: 3rem;
        }
    </style>
</head>
<body>
    <nav class="navbar">
        <a href="#" class="nav-brand">Логотип</a>
        
        <ul class="nav-menu" id="nav-menu">
            <li class="nav-item"><a href="#" class="nav-link active">Главная</a></li>
            <li class="nav-item"><a href="#" class="nav-link">О нас</a></li>
            <li class="nav-item"><a href="#" class="nav-link">Услуги</a></li>
            <li class="nav-item"><a href="#" class="nav-link">Контакты</a></li>
        </ul>
        
        <div class="nav-toggle" id="nav-toggle">
            <span></span>
            <span></span>
            <span></span>
        </div>
    </nav>

    <div class="content">
        <div class="content-section">
            <h1>Основной контент</h1>
            <p>Это основной контент страницы. Навигация адаптируется под размер экрана.</p>
        </div>
        
        <div class="content-section">
            <h2>Секция 2</h2>
            <p>Дополнительный контент для демонстрации адаптивности.</p>
        </div>
    </div>

    <script>
        const navToggle = document.getElementById('nav-toggle');
        const navMenu = document.getElementById('nav-menu');

        navToggle.addEventListener('click', () => {
            navToggle.classList.toggle('active');
            navMenu.classList.toggle('active');
        });

        // Закрытие меню при клике на ссылку
        document.querySelectorAll('.nav-link').forEach(link => {
            link.addEventListener('click', () => {
                navToggle.classList.remove('active');
                navMenu.classList.remove('active');
            });
        });
    </script>
</body>
</html>
```

### Адаптивная таблица с современными возможностями
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная таблица</title>
    <style>
        :root {
            --table-bg: #fff;
            --table-border: #ddd;
            --table-header-bg: #f8f9fa;
            --table-hover: #f5f5f5;
        }

        body {
            margin: 0;
            padding: 1rem;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        .table-container {
            overflow-x: auto;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }

        .data-table {
            width: 100%;
            border-collapse: collapse;
            background-color: var(--table-bg);
        }

        .data-table th,
        .data-table td {
            padding: 0.75rem;
            text-align: left;
            border-bottom: 1px solid var(--table-border);
        }

        .data-table th {
            background-color: var(--table-header-bg);
            font-weight: 600;
            position: sticky;
            top: 0;
        }

        .data-table tbody tr:hover {
            background-color: var(--table-hover);
        }

        /* Адаптация для мобильных устройств */
        @media (max-width: 768px) {
            .table-container {
                overflow-x: scroll;
            }

            .data-table {
                min-width: 600px; /* Позволяет таблице быть шире экрана */
            }

            /* Альтернативный подход: преобразование в карточки */
            .table-as-cards {
                display: block;
            }

            .table-as-cards thead {
                display: none;
            }

            .table-as-cards tbody tr {
                display: block;
                margin-bottom: 1rem;
                border: 1px solid var(--table-border);
                border-radius: 8px;
                padding: 1rem;
            }

            .table-as-cards tbody tr:last-child {
                margin-bottom: 0;
            }

            .table-as-cards tbody td {
                display: block;
                text-align: right;
                padding: 0.5rem 0;
                border: none;
            }

            .table-as-cards tbody td:before {
                content: attr(data-label) ": ";
                float: left;
                font-weight: 600;
                color: #666;
            }
        }

        /* Для очень маленьких экранов */
        @media (max-width: 480px) {
            .data-table {
                font-size: 0.875rem;
            }

            .data-table th,
            .data-table td {
                padding: 0.5rem;
            }
        }
    </style>
</head>
<body>
    <h1>Адаптивная таблица</h1>

    <!-- Обычная таблица -->
    <div class="table-container">
        <table class="data-table">
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
                    <td data-label="ID">1</td>
                    <td data-label="Имя">Иван Иванов</td>
                    <td data-label="Email">ivan@example.com</td>
                    <td data-label="Телефон">+7 (999) 123-45-67</td>
                    <td data-label="Город">Москва</td>
                    <td data-label="Статус">Активен</td>
                </tr>
                <tr>
                    <td data-label="ID">2</td>
                    <td data-label="Имя">Мария Петрова</td>
                    <td data-label="Email">maria@example.com</td>
                    <td data-label="Телефон">+7 (999) 234-56-78</td>
                    <td data-label="Город">Санкт-Петербург</td>
                    <td data-label="Статус">Неактивен</td>
                </tr>
                <tr>
                    <td data-label="ID">3</td>
                    <td data-label="Имя">Алексей Сидоров</td>
                    <td data-label="Email">alexey@example.com</td>
                    <td data-label="Телефон">+7 (999) 345-67-89</td>
                    <td data-label="Город">Новосибирск</td>
                    <td data-label="Статус">Активен</td>
                </tr>
            </tbody>
        </table>
    </div>

    <!-- Таблица, преобразуемая в карточки на мобильных -->
    <div class="table-container">
        <table class="data-table table-as-cards">
            <thead>
                <tr>
                    <th>Пользователь</th>
                    <th>Контакт</th>
                    <th>Дополнительно</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td data-label="Имя">Иван Иванов</td>
                    <td data-label="Email">ivan@example.com</td>
                    <td data-label="Статус">Активен</td>
                </tr>
                <tr>
                    <td data-label="Имя">Мария Петрова</td>
                    <td data-label="Email">maria@example.com</td>
                    <td data-label="Статус">Неактивен</td>
                </tr>
            </tbody>
        </table>
    </div>
</body>
</html>
```

## Современные методологии и практики

### БЭМ (Блок-Элемент-Модификатор) с современными возможностями
```css
/* Блок карточки пользователя */
.user-card {
    --card-bg: #ffffff;
    --card-border: #e0e0e0;
    --card-shadow: 0 2px 10px rgba(0,0,0,0.1);
    --primary-color: #007acc;
    
    background-color: var(--card-bg);
    border: 1px solid var(--card-border);
    border-radius: 8px;
    padding: 1.5rem;
    box-shadow: var(--card-shadow);
    max-width: 300px;
}

/* Элемент аватара */
.user-card__avatar {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    object-fit: cover;
    margin-bottom: 1rem;
}

/* Элемент имени */
.user-card__name {
    margin: 0 0 0.5rem 0;
    font-size: 1.2rem;
    color: #333;
}

/* Элемент должности */
.user-card__title {
    margin: 0 0 1rem 0;
    color: #666;
    font-size: 0.9rem;
}

/* Элемент кнопки */
.user-card__button {
    background-color: var(--primary-color);
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s;
}

.user-card__button:hover {
    background-color: #005a9e;
}

/* Модификатор - карточка администратора */
.user-card--admin {
    --primary-color: #d32f2f;
    border-left: 4px solid var(--primary-color);
}

/* Модификатор - карточка онлайн */
.user-card--online::after {
    content: '';
    position: absolute;
    top: 10px;
    right: 10px;
    width: 10px;
    height: 10px;
    background-color: #4caf50;
    border-radius: 50%;
}
```

### Использование CSS-контейнеров для компонентов
```css
/* Определение контейнера для карточки */
.product-card {
    container-type: inline-size;
    container-name: product-card;
    display: flex;
    flex-direction: column;
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    max-width: 300px;
}

.product-card__image {
    width: 100%;
    height: 200px;
    object-fit: cover;
}

.product-card__content {
    padding: 1rem;
    flex: 1;
    display: flex;
    flex-direction: column;
}

/* Адаптация карточки внутри контейнера */
@container product-card (min-width: 250px) {
    .product-card {
        flex-direction: row;
    }
    
    .product-card__image {
        width: 120px;
        height: 120px;
        flex-shrink: 0;
    }
    
    .product-card__content {
        padding: 0.75rem;
    }
}

@container product-card (min-width: 350px) {
    .product-card {
        max-width: 400px;
    }
    
    .product-card__image {
        width: 150px;
        height: 150px;
    }
}
```

### CSS-переменные для темизации
```css
/* Светлая тема по умолчанию */
:root {
    --color-primary: #007acc;
    --color-primary-dark: #005a9e;
    --color-secondary: #6c757d;
    --color-success: #28a745;
    --color-danger: #dc3545;
    --color-warning: #ffc107;
    --color-info: #17a2b8;
    
    --bg-primary: #ffffff;
    --bg-secondary: #f8f9fa;
    --bg-tertiary: #e9ecef;
    
    --text-primary: #212529;
    --text-secondary: #6c757d;
    --text-muted: #868e96;
    
    --border-color: #dee2e6;
    --shadow: 0 2px 4px rgba(0,0,0,0.1);
    --radius: 0.25rem;
}

/* Темная тема */
[data-theme="dark"], .dark-theme {
    --color-primary: #0d6efd;
    --color-primary-dark: #0b5ed7;
    --color-secondary: #6c757d;
    --color-success: #198754;
    --color-danger: #dc3545;
    --color-warning: #ffc107;
    --color-info: #0dcaf0;
    
    --bg-primary: #212529;
    --bg-secondary: #343a40;
    --bg-tertiary: #495057;
    
    --text-primary: #f8f9fa;
    --text-secondary: #e9ecef;
    --text-muted: #ced4da;
    
    --border-color: #6c757d;
    --shadow: 0 2px 4px rgba(0,0,0,0.3);
}

/* Использование переменных в компонентах */
.button {
    background-color: var(--color-primary);
    color: var(--bg-primary);
    border: 1px solid var(--border-color);
    padding: 0.5rem 1rem;
    border-radius: var(--radius);
    cursor: pointer;
    transition: background-color 0.3s;
}

.button:hover {
    background-color: var(--color-primary-dark);
}

.card {
    background-color: var(--bg-primary);
    color: var(--text-primary);
    border: 1px solid var(--border-color);
    border-radius: var(--radius);
    box-shadow: var(--shadow);
}
```

## Производительность и оптимизация CSS

### Оптимизация рендеринга
```css
/* Использование transform вместо изменения layout свойств */
.element-move {
    transition: transform 0.3s ease; /* Лучше для производительности */
}

.element-move:hover {
    transform: translateX(10px);
}

/* Плохо: изменение layout свойств */
.element-move-bad {
    transition: left 0.3s ease; /* Вызывает перерисовку */
}

/* Использование will-change для оптимизации */
.interactive-element {
    will-change: transform; /* Подсказка браузеру о будущих изменениях */
}

/* Использование contain для изоляции изменений */
.component-container {
    contain: layout style paint; /* Изолирует изменения в компоненте */
}

/* Оптимизация шрифтов */
@font-face {
    font-family: 'CustomFont';
    src: url('font.woff2') format('woff2');
    font-display: swap; /* Улучшает отображение текста при загрузке шрифта */
}
```

### Использование современных селекторов
```css
/* Современные селекторы для улучшения производительности */
/* :is() - позволяет объединять селекторы */
.card:is(.featured, .new, .popular) {
    border: 2px solid var(--color-primary);
}

/* :where() - с нулевой специфичностью */
.card:where(.featured, .new, .popular) {
    border: 2px solid var(--color-primary);
}

/* :has() - родительский селектор (ограниченная поддержка) */
.article:has(h1.special) {
    background-color: #ffffcc;
}

/* :not() с несколькими условиями */
.button:not(:disabled):not(.inactive) {
    cursor: pointer;
}

/* Соседние селекторы для стилизации */
.step:not(:last-child)::after {
    content: '';
    position: absolute;
    left: 50%;
    top: 100%;
    width: 2px;
    height: 20px;
    background-color: #ccc;
    transform: translateX(-50%);
}
```

## Заключение

Современные возможности CSS предоставляют мощные инструменты для создания адаптивных, доступных и производительных веб-сайтов. Использование CSS Custom Properties, современных функций (calc, clamp, min, max), Grid и Flexbox, Container Queries и других возможностей позволяет создавать сложные адаптивные макеты с минимальными затратами на поддержку.

Ключевые аспекты современного CSS:
1. Использование переменных для поддержки тем и консистентности
2. Применение современных методов раскладки (Grid, Flexbox)
3. Использование Container Queries для компонентной адаптивности
4. Следование современным методологиям (БЭМ, компонентный подход)
5. Оптимизация производительности и доступности
6. Использование современных селекторов для улучшения читаемости кода
7. Поддержка различных пользовательских предпочтений (темный режим, уменьшенная анимация)

Эти возможности позволяют создавать современные, гибкие и масштабируемые веб-приложения, которые хорошо работают на различных устройствах и соответствуют современным веб-стандартам.

## Следующие темы
- [[CSS/Основы]]
- [[CSS/Фреймворки]]
- [[CSS/Адаптивный дизайн]]
- [[CSS/Анимации]]

## Теги
#css #html #web-development #frontend #styling #cascading-stylesheets #web-design #responsive-design #accessibility #bem #css-frameworks #css-variables #css-grid #flexbox #container-queries #modern-css #css-functions #clamp #calc #custom-properties #themes #performance
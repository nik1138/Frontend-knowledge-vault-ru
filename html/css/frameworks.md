# CSS Фреймворки

CSS фреймворки — это библиотеки стилей, которые предоставляют готовые компоненты, сетки и шаблоны для быстрой разработки веб-сайтов. Они значительно ускоряют процесс разработки и обеспечивают согласованный внешний вид.

## Популярные CSS фреймворки

### 1. Bootstrap

Bootstrap — один из самых популярных CSS фреймворков, разработанный Twitter.

#### Основные особенности:
- Адаптивная сетка 12 колонок
- Готовые компоненты (кнопки, формы, навигация)
- JavaScript плагины
- Большое сообщество и документация

#### Пример использования:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bootstrap пример</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container">
        <div class="row">
            <div class="col-md-8">
                <h1 class="text-primary">Основной контент</h1>
                <p class="lead">Это основной контент страницы.</p>
                
                <div class="card mb-3">
                    <div class="card-body">
                        <h5 class="card-title">Карточка</h5>
                        <p class="card-text">Содержимое карточки с Bootstrap.</p>
                        <a href="#" class="btn btn-primary">Подробнее</a>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-header">
                        Боковая панель
                    </div>
                    <div class="card-body">
                        <h5 class="card-title">Информация</h5>
                        <p class="card-text">Дополнительная информация на боковой панели.</p>
                        <a href="#" class="btn btn-outline-primary">Действие</a>
                    </div>
                </div>
            </div>
        </div>
        
        <nav aria-label="Page navigation">
            <ul class="pagination justify-content-center">
                <li class="page-item disabled"><a class="page-link" href="#">Предыдущая</a></li>
                <li class="page-item active"><a class="page-link" href="#">1</a></li>
                <li class="page-item"><a class="page-link" href="#">2</a></li>
                <li class="page-item"><a class="page-link" href="#">3</a></li>
                <li class="page-item"><a class="page-link" href="#">Следующая</a></li>
            </ul>
        </nav>
    </div>

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 2. Tailwind CSS

Tailwind CSS — utility-first CSS framework, который предоставляет низкоуровневые утилиты для создания пользовательских компонентов.

#### Основные особенности:
- Утилитарный подход (utility-first)
- Высокая гибкость
- Возможность настройки
- Маленький размер при правильной оптимизации

#### Пример использования:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tailwind CSS пример</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center">
    <div class="max-w-4xl w-full bg-white rounded-lg shadow-xl overflow-hidden">
        <div class="md:flex">
            <div class="md:flex-shrink-0 p-8 bg-blue-600 text-white">
                <h1 class="text-3xl font-bold mb-4">Заголовок</h1>
                <p class="text-blue-100">Описание компонента</p>
            </div>
            <div class="p-8">
                <div class="uppercase tracking-wide text-sm text-blue-500 font-semibold">Категория</div>
                <h2 class="mt-2 text-xl leading-tight font-semibold text-gray-900">Подзаголовок</h2>
                <p class="mt-2 text-gray-600">Основное содержимое компонента с описанием.</p>
                
                <div class="mt-6 flex space-x-4">
                    <button class="px-6 py-3 bg-blue-600 text-white font-medium rounded-lg hover:bg-blue-700 transition-colors duration-200">
                        Действие 1
                    </button>
                    <button class="px-6 py-3 border border-gray-300 text-gray-700 font-medium rounded-lg hover:bg-gray-50 transition-colors duration-200">
                        Действие 2
                    </button>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

### 3. Foundation

Foundation — адаптивный CSS фреймворк от ZURB.

#### Основные особенности:
- Адаптивная сетка
- Много компонентов
- Хорошая документация
- Поддержка доступности

#### Пример использования:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Foundation пример</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/foundation-sites@6.7.5/dist/css/foundation.min.css">
</head>
<body>
    <div class="grid-container">
        <div class="grid-x grid-padding-x">
            <div class="cell large-8">
                <h1 class="margin-bottom-2">Основной контент</h1>
                <div class="callout">
                    <h5>Заголовок</h5>
                    <p>Содержимое в callout блоке.</p>
                    <a href="#" class="button">Кнопка</a>
                </div>
            </div>
            <div class="cell large-4">
                <div class="card">
                    <div class="card-divider">
                        Боковая панель
                    </div>
                    <div class="card-section">
                        <h4>Заголовок</h4>
                        <p>Содержимое карточки.</p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

### 4. Bulma

Bulma — современный CSS фреймворк на основе Flexbox.

#### Основные особенности:
- Основан на Flexbox
- Чистый CSS (без JavaScript)
- Адаптивный дизайн
- Модульная структура

#### Пример использования:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bulma пример</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.9.4/css/bulma.min.css">
</head>
<body>
    <section class="section">
        <div class="container">
            <div class="columns">
                <div class="column is-two-thirds">
                    <div class="box">
                        <h1 class="title">Основной контент</h1>
                        <p class="subtitle">Описание основного контента</p>
                        
                        <article class="media">
                            <div class="media-content">
                                <div class="content">
                                    <p>
                                        <strong>Иван Иванов</strong> <small>@ivan</small> <small>31м</small>
                                        <br>
                                        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean efficitur sit amet massa fringilla egestas. Nullam condimentum luctus turpis.
                                    </p>
                                </div>
                            </div>
                        </article>
                    </div>
                </div>
                <div class="column">
                    <div class="box">
                        <h3 class="title is-4">Боковая панель</h3>
                        <aside class="menu">
                            <p class="menu-label">
                                General
                            </p>
                            <ul class="menu-list">
                                <li><a>Dashboard</a></li>
                                <li><a>Customers</a></li>
                            </ul>
                            <p class="menu-label">
                                Administration
                            </p>
                            <ul class="menu-list">
                                <li><a>Team Settings</a></li>
                                <li>
                                    <a class="is-active">Manage Your Team</a>
                                    <ul>
                                        <li><a>Members</a></li>
                                        <li><a>Plugins</a></li>
                                        <li><a>Add a member</a></li>
                                    </ul>
                                </li>
                            </ul>
                        </aside>
                    </div>
                </div>
            </div>
        </div>
    </section>
</body>
</html>
```

## Создание собственного CSS фреймворка

### 1. Структура проекта

```
my-framework/
├── src/
│   ├── base/
│   │   ├── reset.css
│   │   ├── typography.css
│   │   └── utilities.css
│   ├── layout/
│   │   ├── grid.css
│   │   ├── containers.css
│   │   └── flex.css
│   ├── components/
│   │   ├── buttons.css
│   │   ├── cards.css
│   │   ├── forms.css
│   │   └── navigation.css
│   └── themes/
│       ├── light.css
│       └── dark.css
├── dist/
│   ├── my-framework.css
│   └── my-framework.min.css
└── package.json
```

### 2. Базовая сетка

```css
/* src/layout/grid.css */
.container {
    width: 100%;
    margin-right: auto;
    margin-left: auto;
    padding-right: 15px;
    padding-left: 15px;
}

@media (min-width: 576px) {
    .container {
        max-width: 540px;
    }
}

@media (min-width: 768px) {
    .container {
        max-width: 720px;
    }
}

@media (min-width: 992px) {
    .container {
        max-width: 960px;
    }
}

@media (min-width: 1200px) {
    .container {
        max-width: 1140px;
    }
}

.row {
    display: flex;
    flex-wrap: wrap;
    margin-right: -15px;
    margin-left: -15px;
}

.col {
    flex-basis: 0;
    flex-grow: 1;
    max-width: 100%;
    padding-right: 15px;
    padding-left: 15px;
}

.col-1 { flex: 0 0 8.333333%; max-width: 8.333333%; }
.col-2 { flex: 0 0 16.666667%; max-width: 16.666667%; }
.col-3 { flex: 0 0 25%; max-width: 25%; }
.col-4 { flex: 0 0 33.333333%; max-width: 33.333333%; }
.col-5 { flex: 0 0 41.666667%; max-width: 41.666667%; }
.col-6 { flex: 0 0 50%; max-width: 50%; }
.col-7 { flex: 0 0 58.333333%; max-width: 58.333333%; }
.col-8 { flex: 0 0 66.666667%; max-width: 66.666667%; }
.col-9 { flex: 0 0 75%; max-width: 75%; }
.col-10 { flex: 0 0 83.333333%; max-width: 83.333333%; }
.col-11 { flex: 0 0 91.666667%; max-width: 91.666667%; }
.col-12 { flex: 0 0 100%; max-width: 100%; }

/* Адаптивные колонки */
@media (min-width: 576px) {
    .col-sm-1 { flex: 0 0 8.333333%; max-width: 8.333333%; }
    .col-sm-2 { flex: 0 0 16.666667%; max-width: 16.666667%; }
    .col-sm-3 { flex: 0 0 25%; max-width: 25%; }
    .col-sm-4 { flex: 0 0 33.333333%; max-width: 33.333333%; }
    .col-sm-5 { flex: 0 0 41.666667%; max-width: 41.666667%; }
    .col-sm-6 { flex: 0 0 50%; max-width: 50%; }
    .col-sm-7 { flex: 0 0 58.333333%; max-width: 58.333333%; }
    .col-sm-8 { flex: 0 0 66.666667%; max-width: 66.666667%; }
    .col-sm-9 { flex: 0 0 75%; max-width: 75%; }
    .col-sm-10 { flex: 0 0 83.333333%; max-width: 83.333333%; }
    .col-sm-11 { flex: 0 0 91.666667%; max-width: 91.666667%; }
    .col-sm-12 { flex: 0 0 100%; max-width: 100%; }
}

@media (min-width: 768px) {
    .col-md-1 { flex: 0 0 8.333333%; max-width: 8.333333%; }
    .col-md-2 { flex: 0 0 16.666667%; max-width: 16.666667%; }
    .col-md-3 { flex: 0 0 25%; max-width: 25%; }
    .col-md-4 { flex: 0 0 33.333333%; max-width: 33.333333%; }
    .col-md-5 { flex: 0 0 41.666667%; max-width: 41.666667%; }
    .col-md-6 { flex: 0 0 50%; max-width: 50%; }
    .col-md-7 { flex: 0 0 58.333333%; max-width: 58.333333%; }
    .col-md-8 { flex: 0 0 66.666667%; max-width: 66.666667%; }
    .col-md-9 { flex: 0 0 75%; max-width: 75%; }
    .col-md-10 { flex: 0 0 83.333333%; max-width: 83.333333%; }
    .col-md-11 { flex: 0 0 91.666667%; max-width: 91.666667%; }
    .col-md-12 { flex: 0 0 100%; max-width: 100%; }
}
```

### 3. Компоненты

```css
/* src/components/buttons.css */
.btn {
    display: inline-block;
    font-weight: 400;
    text-align: center;
    vertical-align: middle;
    user-select: none;
    border: 1px solid transparent;
    padding: 0.375rem 0.75rem;
    font-size: 1rem;
    line-height: 1.5;
    border-radius: 0.25rem;
    transition: color 0.15s ease-in-out, background-color 0.15s ease-in-out, 
                border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.btn:focus, .btn.focus {
    outline: 0;
    box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.btn:disabled, .btn.disabled {
    opacity: 0.65;
    pointer-events: none;
}

.btn-primary {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}

.btn-primary:hover {
    color: #fff;
    background-color: #0069d9;
    border-color: #0062cc;
}

.btn-secondary {
    color: #fff;
    background-color: #6c757d;
    border-color: #6c757d;
}

.btn-success {
    color: #fff;
    background-color: #28a745;
    border-color: #28a745;
}

.btn-danger {
    color: #fff;
    background-color: #dc3545;
    border-color: #dc3545;
}

.btn-outline-primary {
    color: #007bff;
    border-color: #007bff;
}

.btn-outline-primary:hover {
    color: #fff;
    background-color: #007bff;
    border-color: #007bff;
}
```

### 4. Утилиты

```css
/* src/base/utilities.css */
/* Отступы */
.m-0 { margin: 0 !important; }
.m-1 { margin: 0.25rem !important; }
.m-2 { margin: 0.5rem !important; }
.m-3 { margin: 1rem !important; }
.m-4 { margin: 1.5rem !important; }
.m-5 { margin: 3rem !important; }

.mt-0 { margin-top: 0 !important; }
.mt-1 { margin-top: 0.25rem !important; }
.mt-2 { margin-top: 0.5rem !important; }
.mt-3 { margin-top: 1rem !important; }
.mt-4 { margin-top: 1.5rem !important; }
.mt-5 { margin-top: 3rem !important; }

/* Паддинги */
.p-0 { padding: 0 !important; }
.p-1 { padding: 0.25rem !important; }
.p-2 { padding: 0.5rem !important; }
.p-3 { padding: 1rem !important; }
.p-4 { padding: 1.5rem !important; }
.p-5 { padding: 3rem !important; }

/* Текстовые утилиты */
.text-left { text-align: left !important; }
.text-center { text-align: center !important; }
.text-right { text-align: right !important; }
.text-justify { text-align: justify !important; }

.text-lowercase { text-transform: lowercase !important; }
.text-uppercase { text-transform: uppercase !important; }
.text-capitalize { text-transform: capitalize !important; }

.font-weight-normal { font-weight: 400 !important; }
.font-weight-bold { font-weight: 700 !important; }
.font-italic { font-style: italic !important; }

/* Отображение */
.d-none { display: none !important; }
.d-inline { display: inline !important; }
.d-inline-block { display: inline-block !important; }
.d-block { display: block !important; }
.d-flex { display: flex !important; }
.d-inline-flex { display: inline-flex !important; }

/* Визуальные хитрости */
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

.clearfix::after {
    display: block;
    clear: both;
    content: "";
}
```

## Адаптивный дизайн в фреймворках

### Медиа-запросы

```css
/* src/base/responsive.css */
/* Мобильные устройства */
@media (max-width: 575.98px) {
    .container {
        padding-right: 10px;
        padding-left: 10px;
    }
    
    .text-responsive {
        font-size: 0.9rem;
        line-height: 1.4;
    }
}

/* Планшеты */
@media (min-width: 576px) and (max-width: 767.98px) {
    .container {
        max-width: 540px;
    }
}

/* Настольные компьютеры */
@media (min-width: 768px) and (max-width: 991.98px) {
    .container {
        max-width: 720px;
    }
}

/* Большие экраны */
@media (min-width: 992px) and (max-width: 1199.98px) {
    .container {
        max-width: 960px;
    }
}

/* Очень большие экраны */
@media (min-width: 1200px) {
    .container {
        max-width: 1140px;
    }
}

/* Портретная ориентация */
@media (orientation: portrait) {
    .landscape-only {
        display: none;
    }
}

/* Альбомная ориентация */
@media (orientation: landscape) {
    .portrait-only {
        display: none;
    }
}
```

## Темизация и настройка

### CSS переменные для темизации

```css
/* src/themes/variables.css */
:root {
    /* Цвета */
    --primary-color: #007bff;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --warning-color: #ffc107;
    --info-color: #17a2b8;
    --light-color: #f8f9fa;
    --dark-color: #343a40;

    /* Типографика */
    --font-family-base: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    --font-size-base: 1rem;
    --font-weight-normal: 400;
    --font-weight-bold: 700;
    --line-height-base: 1.5;

    /* Размеры */
    --border-radius: 0.25rem;
    --border-radius-lg: 0.3rem;
    --border-radius-sm: 0.2rem;

    /* Тени */
    --box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
    --box-shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
    --box-shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);

    /* Анимации */
    --transition-base: all 0.2s ease-in-out;
    --transition-fade: opacity 0.15s linear;
    --transition-collapse: height 0.35s ease;
}
```

### Тема для темного режима

```css
/* src/themes/dark.css */
[data-theme="dark"], .dark-theme {
    --primary-color: #0d6efd;
    --secondary-color: #6c757d;
    --success-color: #198754;
    --danger-color: #dc3545;
    --warning-color: #ffc107;
    --info-color: #0dcaf0;
    --light-color: #212529;
    --dark-color: #f8f9fa;

    --body-bg: #121212;
    --body-color: #e9ecef;
    --border-color: #495057;
}

[data-theme="dark"] body,
.dark-theme body {
    background-color: var(--body-bg);
    color: var(--body-color);
}

[data-theme="dark"] .card,
.dark-theme .card {
    background-color: #1e1e1e;
    border-color: var(--border-color);
}

[data-theme="dark"] .btn {
    border-color: var(--border-color);
}

[data-theme="dark"] .form-control,
.dark-theme .form-control {
    background-color: #2d2d2d;
    border-color: var(--border-color);
    color: var(--body-color);
}

[data-theme="dark"] .form-control:focus,
.dark-theme .form-control:focus {
    border-color: var(--primary-color);
    box-shadow: 0 0 0 0.2rem rgba(13, 110, 253, 0.25);
}
```

## Современные возможности фреймворков

### Использование CSS Grid

```css
/* src/layout/grid-modern.css */
.grid-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 1rem;
    padding: 1rem;
}

.grid-item {
    background: var(--light-color);
    padding: 1rem;
    border-radius: 0.5rem;
    box-shadow: var(--box-shadow);
}

/* Сетка с именованными областями */
.layout-grid {
    display: grid;
    grid-template-areas: 
        "header header header"
        "nav main aside"
        "footer footer footer";
    grid-template-rows: auto 1fr auto;
    grid-template-columns: 200px 1fr 200px;
    min-height: 100vh;
}

.header { grid-area: header; }
.nav { grid-area: nav; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

### Анимации и переходы

```css
/* src/components/animations.css */
/* Появление */
.fade-in {
    animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}

.slide-in-left {
    animation: slideInLeft 0.3s ease-out;
}

@keyframes slideInLeft {
    from {
        transform: translateX(-100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

/* Пульсация */
.pulse {
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0% { box-shadow: 0 0 0 0 rgba(0, 123, 255, 0.7); }
    70% { box-shadow: 0 0 0 10px rgba(0, 123, 255, 0); }
    100% { box-shadow: 0 0 0 0 rgba(0, 123, 255, 0); }
}

/* Загрузка */
.spinner {
    display: inline-block;
    width: 20px;
    height: 20px;
    border: 3px solid rgba(0, 0, 0, 0.1);
    border-left-color: var(--primary-color);
    border-radius: 50%;
    animation: spinner 1.5s linear infinite;
}

@keyframes spinner {
    to { transform: rotate(360deg); }
}

/* Переходы */
.transition-all {
    transition: all 0.3s ease;
}

.transition-opacity {
    transition: opacity 0.3s ease;
}

.transition-transform {
    transition: transform 0.3s ease;
}
```

## Практические примеры

### Адаптивная карточка товара

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Адаптивная карточка товара</title>
    <style>
        /* Базовые стили */
        :root {
            --primary-color: #007acc;
            --secondary-color: #6c757d;
            --success-color: #28a745;
            --danger-color: #dc3545;
            --border-radius: 8px;
            --box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        }

        * {
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f5f5f5;
        }

        .products-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 20px;
            max-width: 1200px;
            margin: 0 auto;
        }

        .product-card {
            background: white;
            border-radius: var(--border-radius);
            box-shadow: var(--box-shadow);
            overflow: hidden;
            transition: transform 0.3s ease, box-shadow 0.3s ease;
        }

        .product-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 15px rgba(0, 0, 0, 0.15);
        }

        .product-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
        }

        .product-info {
            padding: 15px;
        }

        .product-title {
            margin: 0 0 10px 0;
            font-size: 1.1em;
            color: #333;
        }

        .product-price {
            font-size: 1.3em;
            font-weight: bold;
            color: var(--primary-color);
            margin-bottom: 10px;
        }

        .product-old-price {
            text-decoration: line-through;
            color: #999;
            margin-left: 8px;
            font-size: 0.9em;
        }

        .product-rating {
            display: flex;
            align-items: center;
            margin-bottom: 10px;
        }

        .stars {
            color: #ffc107;
            margin-right: 5px;
        }

        .rating-count {
            color: #666;
            font-size: 0.9em;
        }

        .product-actions {
            display: flex;
            gap: 10px;
            margin-top: 15px;
        }

        .btn {
            flex: 1;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            text-align: center;
            text-decoration: none;
            font-size: 0.9em;
            transition: background-color 0.3s;
        }

        .btn-add-to-cart {
            background-color: var(--primary-color);
            color: white;
        }

        .btn-add-to-cart:hover {
            background-color: #005a9e;
        }

        .btn-wishlist {
            background-color: #f8f9fa;
            color: #333;
            border: 1px solid #ddd;
        }

        .btn-wishlist:hover {
            background-color: #e9ecef;
        }

        .product-badge {
            position: absolute;
            top: 10px;
            left: 10px;
            background-color: var(--danger-color);
            color: white;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.8em;
            font-weight: bold;
            z-index: 1;
        }

        /* Адаптивность */
        @media (max-width: 768px) {
            .products-grid {
                grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
                gap: 15px;
            }

            .product-actions {
                flex-direction: column;
            }
        }

        @media (max-width: 480px) {
            .products-grid {
                grid-template-columns: 1fr;
                gap: 15px;
            }

            body {
                padding: 10px;
            }
        }
    </style>
</head>
<body>
    <h1>Каталог товаров</h1>

    <div class="products-grid">
        <div class="product-card">
            <div style="position: relative;">
                <img src="https://via.placeholder.com/300x200" alt="Смартфон" class="product-image">
                <div class="product-badge">Хит продаж</div>
            </div>
            <div class="product-info">
                <h3 class="product-title">Смартфон iPhone 15</h3>
                <div class="product-price">
                    129 990₽
                    <span class="product-old-price">139 990₽</span>
                </div>
                <div class="product-rating">
                    <div class="stars">★★★★★</div>
                    <div class="rating-count">(124 отзыва)</div>
                </div>
                <div class="product-actions">
                    <button class="btn btn-add-to-cart">В корзину</button>
                    <button class="btn btn-wishlist">В избранное</button>
                </div>
            </div>
        </div>

        <div class="product-card">
            <img src="https://via.placeholder.com/300x200" alt="Ноутбук" class="product-image">
            <div class="product-info">
                <h3 class="product-title">Ноутбук MacBook Air M2</h3>
                <div class="product-price">99 990₽</div>
                <div class="product-rating">
                    <div class="stars">★★★★☆</div>
                    <div class="rating-count">(89 отзывов)</div>
                </div>
                <div class="product-actions">
                    <button class="btn btn-add-to-cart">В корзину</button>
                    <button class="btn btn-wishlist">В избранное</button>
                </div>
            </div>
        </div>

        <div class="product-card">
            <img src="https://via.placeholder.com/300x200" alt="Наушники" class="product-image">
            <div class="product-info">
                <h3 class="product-title">Наушники AirPods Pro</h3>
                <div class="product-price">29 990₽</div>
                <div class="product-rating">
                    <div class="stars">★★★★★</div>
                    <div class="rating-count">(256 отзывов)</div>
                </div>
                <div class="product-actions">
                    <button class="btn btn-add-to-cart">В корзину</button>
                    <button class="btn btn-wishlist">В избранное</button>
                </div>
            </div>
        </div>
    </div>
</body>
</html>
```

### Форма с валидацией и фреймворком

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Форма с валидацией</title>
    <style>
        :root {
            --primary-color: #007acc;
            --success-color: #28a745;
            --danger-color: #dc3545;
            --warning-color: #ffc107;
            --border-radius: 4px;
            --box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }

        * {
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f8f9fa;
        }

        .form-container {
            max-width: 600px;
            margin: 0 auto;
            background: white;
            padding: 30px;
            border-radius: 8px;
            box-shadow: var(--box-shadow);
        }

        .form-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
            color: #333;
        }

        .input-field {
            width: 100%;
            padding: 12px 15px;
            border: 2px solid #ddd;
            border-radius: var(--border-radius);
            font-size: 16px;
            transition: border-color 0.3s;
        }

        .input-field:focus {
            outline: none;
            border-color: var(--primary-color);
            box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.2);
        }

        .input-field.valid {
            border-color: var(--success-color);
        }

        .input-field.invalid {
            border-color: var(--danger-color);
        }

        .help-text {
            font-size: 0.875em;
            color: #666;
            margin-top: 5px;
        }

        .error-message {
            color: var(--danger-color);
            font-size: 0.875em;
            margin-top: 5px;
            display: block;
        }

        .success-message {
            color: var(--success-color);
            font-size: 0.875em;
            margin-top: 5px;
            display: block;
        }

        .requirements-list {
            list-style: none;
            padding: 0;
            margin-top: 5px;
        }

        .requirement {
            font-size: 0.875em;
            margin-bottom: 2px;
        }

        .requirement.met {
            color: var(--success-color);
        }

        .requirement.unmet {
            color: var(--danger-color);
        }

        .form-actions {
            display: flex;
            gap: 10px;
            margin-top: 30px;
        }

        .btn {
            padding: 12px 24px;
            border: none;
            border-radius: var(--border-radius);
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }

        .btn-primary {
            background-color: var(--primary-color);
            color: white;
        }

        .btn-primary:hover {
            background-color: #005a9e;
        }

        .btn-secondary {
            background-color: #6c757d;
            color: white;
        }

        .btn-secondary:hover {
            background-color: #545b62;
        }

        .btn:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }

        .form-status {
            padding: 10px;
            border-radius: 4px;
            margin-bottom: 15px;
            display: none;
        }

        .form-status.success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }

        .form-status.error {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h1>Регистрация пользователя</h1>

        <div class="form-status" id="form-status"></div>

        <form id="validation-form" novalidate>
            <div class="form-group">
                <label for="name">Имя:</label>
                <input type="text"
                       id="name"
                       name="name"
                       class="input-field"
                       required
                       minlength="2"
                       maxlength="50"
                       aria-describedby="name-help name-error">
                <div id="name-help" class="help-text">Введите ваше имя</div>
                <div id="name-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>

            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email"
                       id="email"
                       name="email"
                       class="input-field"
                       required
                       aria-describedby="email-help email-error">
                <div id="email-help" class="help-text">Введите действительный email</div>
                <div id="email-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>

            <div class="form-group">
                <label for="password">Пароль:</label>
                <input type="password"
                       id="password"
                       name="password"
                       class="input-field"
                       required
                       minlength="8"
                       aria-describedby="password-requirements password-error">
                <ul id="password-requirements" class="requirements-list">
                    <li id="req-length" class="requirement unmet">Не менее 8 символов</li>
                    <li id="req-upper" class="requirement unmet">С заглавной буквой</li>
                    <li id="req-lower" class="requirement unmet">Со строчной буквой</li>
                    <li id="req-number" class="requirement unmet">С цифрой</li>
                </ul>
                <div id="password-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>

            <div class="form-group">
                <label for="confirm-password">Подтверждение пароля:</label>
                <input type="password"
                       id="confirm-password"
                       name="confirm-password"
                       class="input-field"
                       required
                       aria-describedby="confirm-password-error">
                <div id="confirm-password-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>

            <div class="form-group">
                <label>
                    <input type="checkbox"
                           name="terms"
                           required
                           aria-describedby="terms-error">
                    Я согласен с условиями использования
                </label>
                <div id="terms-error" class="error-message" role="alert" aria-live="polite"></div>
            </div>

            <div class="form-actions">
                <button type="submit" class="btn btn-primary">Зарегистрироваться</button>
                <button type="reset" class="btn btn-secondary">Очистить</button>
            </div>
        </form>
    </div>

    <script>
        class FormValidator {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupValidation();
            }

            setupValidation() {
                // Валидация при потере фокуса
                this.form.addEventListener('blur', (e) => {
                    if (e.target.matches('input[required]')) {
                        this.validateField(e.target);
                    }
                }, true);

                // Валидация пароля в реальном времени
                document.getElementById('password').addEventListener('input', () => {
                    this.validatePasswordRequirements();
                });

                // Валидация подтверждения пароля
                document.getElementById('confirm-password').addEventListener('input', () => {
                    this.validatePasswordMatch();
                });

                // Отправка формы
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateForm();
                });
            }

            validateField(field) {
                const fieldId = field.id;
                const errorElement = document.getElementById(`${fieldId}-error`);

                if (field.validity.valid) {
                    field.classList.remove('invalid');
                    field.classList.add('valid');
                    errorElement.textContent = '';
                    return true;
                } else {
                    field.classList.remove('valid');
                    field.classList.add('invalid');

                    const errorMessage = this.getFieldErrorMessage(field);
                    errorElement.textContent = errorMessage;
                    return false;
                }
            }

            validatePasswordRequirements() {
                const password = document.getElementById('password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password)
                };

                // Обновление требований
                document.getElementById('req-length').className = `requirement ${requirements.length ? 'met' : 'unmet'}`;
                document.getElementById('req-upper').className = `requirement ${requirements.upper ? 'met' : 'unmet'}`;
                document.getElementById('req-lower').className = `requirement ${requirements.lower ? 'met' : 'unmet'}`;
                document.getElementById('req-number').className = `requirement ${requirements.number ? 'met' : 'unmet'}`;

                return Object.values(requirements).every(req => req);
            }

            validatePasswordMatch() {
                const password = document.getElementById('password').value;
                const confirmPassword = document.getElementById('confirm-password').value;
                const errorElement = document.getElementById('confirm-password-error');

                if (confirmPassword && password !== confirmPassword) {
                    errorElement.textContent = 'Пароли не совпадают';
                    document.getElementById('confirm-password').classList.add('invalid');
                    return false;
                } else {
                    errorElement.textContent = '';
                    document.getElementById('confirm-password').classList.remove('invalid');
                    return true;
                }
            }

            validateForm() {
                let isValid = true;

                // Проверка всех обязательных полей
                const requiredFields = this.form.querySelectorAll('input[required]');
                requiredFields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                });

                // Проверка требований к паролю
                if (!this.validatePasswordRequirements()) {
                    isValid = false;
                }

                // Проверка совпадения паролей
                if (!this.validatePasswordMatch()) {
                    isValid = false;
                }

                // Проверка согласия с условиями
                const termsCheckbox = this.form.querySelector('input[name="terms"]');
                const termsError = document.getElementById('terms-error');
                
                if (!termsCheckbox.checked) {
                    termsError.textContent = 'Необходимо согласие с условиями';
                    isValid = false;
                } else {
                    termsError.textContent = '';
                }

                if (isValid) {
                    this.showStatus('Форма успешно валидирована!', 'success');
                    // Здесь можно отправить форму
                    console.log('Форма готова к отправке');
                } else {
                    this.showStatus('Пожалуйста, исправьте ошибки в форме', 'error');
                }
            }

            getFieldErrorMessage(field) {
                if (field.validity.valueMissing) {
                    return 'Это поле обязательно для заполнения';
                } else if (field.validity.typeMismatch && field.type === 'email') {
                    return 'Введите действительный email адрес';
                } else if (field.validity.tooShort) {
                    return `Введите не менее ${field.minLength} символов`;
                } else if (field.validity.tooLong) {
                    return `Введите не более ${field.maxLength} символов`;
                }
                return 'Неверный формат данных';
            }

            showStatus(message, type) {
                const statusElement = document.getElementById('form-status');
                statusElement.textContent = message;
                statusElement.className = `form-status ${type}`;
                statusElement.style.display = 'block';

                // Скрыть сообщение через 5 секунд
                setTimeout(() => {
                    statusElement.style.display = 'none';
                }, 5000);
            }
        }

        // Инициализация валидатора
        document.addEventListener('DOMContentLoaded', () => {
            new FormValidator('validation-form');
        });
    </script>
</body>
</html>
```

## Заключение

CSS фреймворки предоставляют мощные инструменты для быстрой и эффективной разработки веб-сайтов. Они предлагают готовые решения для типичных задач разработки, такие как сетки, компоненты и стилизация. Однако при использовании фреймворков важно помнить о:

1. **Размере фреймворка** - используйте только необходимые компоненты
2. **Настройке** - адаптируйте фреймворк под свои нужды
3. **Доступности** - убедитесь, что компоненты фреймворка доступны
4. **Производительности** - оптимизируйте загрузку и использование
5. **Поддержке** - выбирайте фреймворки с активной поддержкой
6. **Совместимости** - проверяйте совместимость с различными браузерами

Современные CSS фреймворки также поддерживают современные веб-стандарты, включая Flexbox, Grid, CSS Custom Properties и адаптивный дизайн. Правильное использование этих возможностей позволяет создавать современные, красивые и функциональные веб-сайты.

## Следующие темы
- [[CSS/Адаптивный дизайн]]
- [[CSS/Анимации]]
- [[CSS/Продвинутые возможности]]

## Теги
#css #frameworks #bootstrap #tailwind #foundation #bulma #web-development #frontend #styling #grid #components #responsive-design #web-components #accessibility #modern-css #css-grid #flexbox #custom-properties #theming #validation
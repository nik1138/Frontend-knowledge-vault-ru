# Структура HTML документа

Структура HTML документа является основой любого веб-сайта. Понимание этой структуры критически важно для создания правильно организованных веб-страниц.

## Базовая структура HTML документа

Каждый HTML документ начинается с определения типа документа (DOCTYPE), за которым следуют основные элементы:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Заголовок страницы</title>
</head>
<body>
    <!-- Содержимое страницы -->
</body>
</html>
```

## Объяснение элементов

### `<!DOCTYPE html>`
Этот тег сообщает браузеру, что документ следует стандарту HTML5. Это важно для правильного отображения страницы.

### `<html>` элемент
Это корневой элемент всего HTML документа. Атрибут `lang` указывает язык содержимого страницы, что полезно для доступности и SEO.

### `<head>` элемент
Содержит метаданные о документе, которые не отображаются на странице, но важны для браузера и поисковых систем:
- `<title>` - заголовок страницы, отображаемый во вкладке браузера
- `<meta>` - метаданные, такие как кодировка, описание, ключевые слова
- `<link>` - подключение внешних файлов (CSS, иконки)
- `<script>` - встраивание или подключение JavaScript

### `<body>` элемент
Содержит видимое содержимое страницы, которое отображается в окне браузера.

## Важные теги в `<head>`

### Мета-теги
```html
<meta charset="UTF-8">  <!-- Определяет кодировку документа -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">  <!-- Адаптивность для мобильных устройств -->
<meta name="description" content="Описание страницы">  <!-- Описание для поисковых систем -->
```

### Подключение внешних ресурсов
```html
<link rel="stylesheet" href="styles.css">  <!-- Подключение CSS файла -->
<link rel="icon" href="favicon.ico">  <!-- Иконка сайта -->
```

## Правила хорошей структуры

1. **Использование семантических элементов** - помогает браузерам и поисковым системам понимать структуру документа
2. **Последовательность заголовков** - использовать заголовки в правильной иерархии (h1, h2, h3 и т.д.)
3. **Правильная вложенность элементов** - закрывать теги в правильном порядке
4. **Использование атрибута `lang`** - указывает язык содержимого для доступности и SEO

## Пример полной структуры

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Пример структуры HTML документа">
    <title>Пример HTML документа</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>Основной заголовок страницы</h1>
        <nav>
            <ul>
                <li><a href="#home">Главная</a></li>
                <li><a href="#about">О нас</a></li>
                <li><a href="#contact">Контакты</a></li>
            </ul>
        </nav>
    </header>
    
    <main>
        <section id="home">
            <h2>Домашняя страница</h2>
            <p>Содержимое домашней страницы.</p>
        </section>
    </main>
    
    <footer>
        <p>&copy; 2023 Все права защищены</p>
    </footer>
</body>
</html>
```

## Проверка структуры

Для проверки правильности структуры HTML документа можно использовать онлайн-валидаторы, такие как [W3C Markup Validator](https://validator.w3.org/), который проверяет документ на соответствие стандартам HTML.

## Заключение

Правильная структура HTML документа является фундаментом для создания доступных, SEO-оптимизированных и легко поддерживаемых веб-страниц. Следование этим принципам обеспечивает лучший опыт для пользователей и поисковых систем.

## Современные возможности HTML5

### Элементы структуры
HTML5 предоставляет семантические элементы для лучшей структуризации документа:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Современная структура HTML</title>
</head>
<body>
    <header>
        <!-- Заголовок сайта или раздела -->
        <h1>Название сайта</h1>
        <nav>
            <!-- Навигационные ссылки -->
        </nav>
    </header>

    <main>
        <article>
            <!-- Самостоятельный контент -->
            <header>
                <h2>Заголовок статьи</h2>
            </header>
            <section>
                <!-- Раздел статьи -->
                <h3>Подзаголовок</h3>
                <p>Содержимое раздела...</p>
            </section>
            <footer>
                <!-- Информация об авторе, дата и т.д. -->
            </footer>
        </article>

        <aside>
            <!-- Сопутствующий контент -->
            <h3>Реклама или связанные ссылки</h3>
        </aside>
    </main>

    <footer>
        <!-- Подвал сайта -->
        <p>&copy; 2023 Все права защищены</p>
    </footer>
</body>
</html>
```

### Мета-теги для социальных сетей
```html
<!-- Open Graph теги для Facebook -->
<meta property="og:title" content="Заголовок страницы">
<meta property="og:description" content="Описание страницы">
<meta property="og:image" content="https://example.com/image.jpg">
<meta property="og:url" content="https://example.com/page">
<meta property="og:type" content="website">

<!-- Twitter Cards -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Заголовок страницы">
<meta name="twitter:description" content="Описание страницы">
<meta name="twitter:image" content="https://example.com/image.jpg">
```

### Производительность и оптимизация
```html
<head>
    <!-- Предварительная загрузка критических ресурсов -->
    <link rel="preload" href="critical-font.woff2" as="font" type="font/woff2" crossorigin>
    <link rel="preload" href="hero-image.jpg" as="image">
    
    <!-- Предварительная выборка -->
    <link rel="prefetch" href="next-page.html">
    
    <!-- Подключение к доменам -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="dns-prefetch" href="//analytics.google.com">
</head>
```

### Доступность
```html
<body>
    <!-- Ссылка для пропуска навигации -->
    <a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
    
    <header role="banner">
        <!-- Заголовок сайта -->
    </header>
    
    <nav role="navigation" aria-label="Основная навигация">
        <!-- Навигационные ссылки -->
    </nav>
    
    <main id="main-content" role="main">
        <!-- Основное содержимое -->
    </main>
    
    <footer role="contentinfo">
        <!-- Подвал сайта -->
    </footer>
</body>
```

## Лучшие практики современной структуры HTML

1. **Использование семантических элементов** - правильно использовать элементы `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`
2. **Правильная иерархия заголовков** - следить за логическим порядком h1, h2, h3 и т.д.
3. **Оптимизация для SEO** - использование мета-тегов, канонических ссылок, Open Graph тегов
4. **Оптимизация производительности** - использование preload, prefetch, preconnect
5. **Доступность** - добавление ARIA-ролей, атрибутов доступности, ссылок для пропуска навигации
6. **Международизация** - использование атрибутов `lang`, `dir` для поддержки разных языков и направлений текста

## Проверка структуры и доступности

### Инструменты для проверки:
- W3C Markup Validator - проверка соответствия стандартам HTML
- axe-core - инструмент автоматизированной проверки доступности
- Lighthouse - комплексная проверка производительности, доступности и SEO
- WAVE - веб-инструмент для оценки доступности
- Инструменты разработчика браузера - проверка структуры DOM

## Пример комплексной структуры с современными практиками

```html
<!DOCTYPE html>
<html lang="ru" dir="ltr" prefix="og: https://ogp.me/ns#">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    
    <!-- SEO теги -->
    <title>Заголовок страницы - Название сайта</title>
    <meta name="description" content="Описание страницы для поисковых систем">
    
    <!-- Open Graph теги -->
    <meta property="og:type" content="website">
    <meta property="og:url" content="https://example.com/page">
    <meta property="og:title" content="Заголовок страницы">
    <meta property="og:description" content="Описание страницы">
    <meta property="og:image" content="https://example.com/image.jpg">
    
    <!-- Twitter Cards -->
    <meta property="twitter:card" content="summary_large_image">
    
    <!-- Подключение ресурсов -->
    <link rel="icon" href="/favicon.ico" sizes="any">
    <link rel="icon" href="/icon.svg" type="image/svg+xml">
    <link rel="apple-touch-icon" href="/apple-touch-icon.png">
    <link rel="manifest" href="/site.webmanifest">
    
    <!-- Оптимизация производительности -->
    <link rel="preload" href="critical-font.woff2" as="font" type="font/woff2" crossorigin>
    <link rel="preconnect" href="https://fonts.googleapis.com" crossorigin>
    
    <!-- Подключение стилей -->
    <link rel="stylesheet" href="styles.css" media="screen">
</head>
<body class="no-js">
    <!-- Ссылка для пропуска навигации -->
    <a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
    
    <header role="banner">
        <h1>Название сайта</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/" aria-current="page">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/services">Услуги</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>

    <main id="main-content" role="main">
        <article>
            <header>
                <h2>Заголовок статьи</h2>
                <p>Опубликовано: <time datetime="2023-11-08">8 ноября 2023</time></p>
            </header>

            <section>
                <h3>Введение</h3>
                <p>Содержимое введения статьи...</p>
            </section>

            <section>
                <h3>Основная часть</h3>
                <p>Содержимое основной части статьи...</p>
            </section>

            <footer>
                <p>Автор: <span>Имя автора</span></p>
            </footer>
        </article>

        <aside role="complementary" aria-label="Сопутствующий контент">
            <h3>Похожие статьи</h3>
            <ul>
                <li><a href="/article1">Статья 1</a></li>
                <li><a href="/article2">Статья 2</a></li>
            </ul>
        </aside>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Название компании. Все права защищены.</p>
        <nav aria-label="Нижняя навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/terms">Условия использования</a></li>
            </ul>
        </nav>
    </footer>

    <!-- Подключение JavaScript -->
    <script src="script.js" defer></script>
    
    <script>
        // Определение поддержки JavaScript
        document.body.classList.remove('no-js');
        document.body.classList.add('js');
    </script>
</body>
</html>
```

## Заключение

Современная структура HTML документа включает в себя не только базовую семантическую разметку, но и элементы оптимизации для SEO, социальных сетей, производительности и доступности. Использование всех этих аспектов создает высококачественные веб-страницы, которые хорошо работают для всех пользователей.

## Следующие темы
- [[HTML/Теги и атрибуты]]
- [[HTML/Семантические элементы]]
- [[HTML/Доступность]]

## Теги
#html #structure #document-structure #web-development #frontend #basics #seo #accessibility #html5 #performance
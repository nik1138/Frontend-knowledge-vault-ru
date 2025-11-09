# Семантическая разметка HTML

Семантическая разметка — это использование HTML тегов по их прямому назначению, что делает код более понятным для браузеров, поисковых систем и разработчиков. Правильная семантика улучшает доступность, SEO и поддерживаемость кода.

## Что такое семантическая разметка?

Семантическая разметка — это подход к написанию HTML, при котором каждый элемент используется в соответствии с его смысловым значением. Это помогает браузерам, скринридерам и поисковым системам лучше понимать структуру и содержание веб-страницы.

### Преимущества семантической разметки

1. **Улучшенная доступность** - помогает пользователям с ограниченными возможностями навигировать по сайту
2. **Лучшая SEO оптимизация** - поисковые системы лучше понимают содержание страницы
3. **Улучшенная поддерживаемость** - код становится более понятным для других разработчиков
4. **Будущая совместимость** - семантически правильный код будет работать с новыми технологиями

## Основные семантические элементы HTML5

### `<header>` - Заголовок
```html
<header>
    <h1>Название сайта</h1>
    <nav>
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
        </ul>
    </nav>
</header>
```

Используется для представления вступительного содержимого или набора навигационных ссылок. Может встречаться как в документе целиком, так и в отдельных разделах.

### `<nav>` - Навигация
```html
<nav>
    <ul>
        <li><a href="#home">Главная</a></li>
        <li><a href="#services">Услуги</a></li>
        <li><a href="#contact">Контакты</a></li>
    </ul>
</nav>
```

Представляет собой раздел, содержащий навигационные ссылки. Не все группы ссылок должны быть обернуты в `<nav>` — только основные блоки навигации.

### `<main>` - Основное содержимое
```html
<main>
    <article>
        <h2>Заголовок статьи</h2>
        <p>Содержимое статьи...</p>
    </article>
</main>
```

Представляет основное содержимое документа или приложения. Должен содержать контент, уникальный для этой страницы, и не повторяться на других страницах сайта.

### `<section>` - Секция
```html
<section>
    <h2>О компании</h2>
    <p>Информация о компании...</p>
</section>
```

Группирует тематически связанные элементы. Обычно содержит заголовок. Используется для логического разделения содержимого.

### `<article>` - Независимый контент
```html
<article>
    <h2>Заголовок статьи</h2>
    <p>Текст статьи...</p>
    <footer>
        <p>Опубликовано: 15.03.2023</p>
    </footer>
</article>
```

Представляет собой независимый, автономный фрагмент содержимого, который может быть распространён, переиздан или использован независимо от остального сайта.

### `<aside>` - Сопутствующий контент
```html
<aside>
    <h3>Похожие статьи</h3>
    <ul>
        <li><a href="#">Статья 1</a></li>
        <li><a href="#">Статья 2</a></li>
    </ul>
</aside>
```

Содержит контент, косвенно связанный с основным содержимым. Может быть использован для боковой панели, рекламы, биографии автора и т.д.

### `<footer>` - Подвал
```html
<footer>
    <p>&copy; 2023 Все права защищены</p>
    <nav>
        <ul>
            <li><a href="/privacy">Политика конфиденциальности</a></li>
            <li><a href="/terms">Условия использования</a></li>
        </ul>
    </nav>
</footer>
```

Представляет собой нижнюю часть раздела или документа. Содержит информацию об авторе, информацию о копирайте, ссылки на связанные документы и т.д.

## Правила использования семантических элементов

### 1. Правильная иерархия заголовков
```html
<!-- Правильно -->
<main>
    <h1>Основной заголовок страницы</h1>
    <section>
        <h2>Подраздел 1</h2>
        <article>
            <h3>Статья в подразделе</h3>
        </article>
    </section>
    <section>
        <h2>Подраздел 2</h2>
    </section>
</main>

<!-- Неправильно -->
<h3>Основной заголовок</h3>  <!-- Пропущен h1 -->
<h1>Подзаголовок</h1>        <!-- Нарушена иерархия -->
```

### 2. Вложенность элементов
```html
<!-- Правильно -->
<article>
    <header>
        <h2>Заголовок статьи</h2>
    </header>
    <section>
        <h3>Подраздел статьи</h3>
        <p>Содержимое...</p>
    </section>
    <footer>
        <p>Опубликовано: 2023</p>
    </footer>
</article>

<!-- Неправильно -->
<header>
    <article>  <!-- article не должен быть внутри header -->
        <h2>Заголовок</h2>
    </article>
</header>
```

### 3. Использование вместе с классами
```html
<!-- Хорошо: семантика + стилизация -->
<article class="blog-post">
    <header class="post-header">
        <h2 class="post-title">Заголовок статьи</h2>
    </header>
    <div class="post-content">
        <p>Содержимое статьи...</p>
    </div>
    <footer class="post-footer">
        <time datetime="2023-03-15">15 марта 2023</time>
    </footer>
</article>
```

## Пример полной семантической структуры

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Пример семантической разметки</title>
</head>
<body>
    <header>
        <h1>Название сайта</h1>
        <nav>
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/blog">Блог</a></li>
                <li><a href="/about">О нас</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <header>
                <h1>Заголовок статьи</h1>
                <p>Автор: Иван Иванов</p>
            </header>
            
            <section>
                <h2>Введение</h2>
                <p>Вступительный абзац статьи...</p>
            </section>
            
            <section>
                <h2>Основная часть</h2>
                <p>Основное содержимое статьи...</p>
            </section>
            
            <aside>
                <h3>Связанные темы</h3>
                <ul>
                    <li><a href="#">Ссылка 1</a></li>
                    <li><a href="#">Ссылка 2</a></li>
                </ul>
            </aside>
            
            <footer>
                <p>Опубликовано: <time datetime="2023-03-15">15 марта 2023</time></p>
            </footer>
        </article>
    </main>

    <aside>
        <h2>Боковая панель</h2>
        <nav>
            <h3>Категории</h3>
            <ul>
                <li><a href="#">Технологии</a></li>
                <li><a href="#">Дизайн</a></li>
                <li><a href="#">Разработка</a></li>
            </ul>
        </nav>
    </aside>

    <footer>
        <p>&copy; 2023 Все права защищены</p>
        <nav>
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </footer>
</body>
</html>
```

## Проверка семантической разметки

Для проверки семантической разметки можно использовать:

1. **Инструменты разработчика браузера** - для визуального анализа структуры
2. **W3C Markup Validator** - для проверки соответствия стандартам
3. **Accessibility Developer Tools** - для проверки доступности

## Заключение

Семантическая разметка — это фундамент хорошего HTML кода. Она делает веб-страницы более доступными, понятными для поисковых систем и удобными для поддержки. Использование правильных элементов в правильном контексте улучшает общее качество веб-сайта.

## Расширенные семантические паттерны

### Вложенные структуры
```html
<!-- Правильное вложение семантических элементов -->
<article>
    <header>
        <h1>Заголовок статьи</h1>
        <p>Автор: <a href="/author/john">Иван Петров</a></p>
        <p>Опубликовано: <time datetime="2023-11-08">8 ноября 2023</time></p>
    </header>

    <nav aria-label="Навигация по статье">
        <ul>
            <li><a href="#introduction">Введение</a></li>
            <li><a href="#main-content">Основное содержимое</a></li>
            <li><a href="#conclusion">Заключение</a></li>
        </ul>
    </nav>

    <section id="introduction">
        <h2>Введение</h2>
        <p>Вступительный текст статьи...</p>
    </section>

    <section id="main-content">
        <h2>Основное содержимое</h2>
        
        <section>
            <h3>Подраздел основного содержимого</h3>
            <p>Содержимое подраздела...</p>
            
            <aside>
                <h4>Интересный факт</h4>
                <p>Дополнительная информация, связанная с темой...</p>
            </aside>
        </section>

        <section>
            <h3>Еще один подраздел</h3>
            <p>Содержимое другого подраздела...</p>
        </section>
    </section>

    <section id="conclusion">
        <h2>Заключение</h2>
        <p>Заключительный текст статьи...</p>
    </section>

    <footer>
        <p>Теги: <a href="/tag/web-development">Веб-разработка</a>, <a href="/tag/html">HTML</a></p>
        <p>Просмотров: <span>1234</span></p>
    </footer>
</article>
```

### Семантика для списков и данных
```html
<!-- Описание данных -->
<dl>
    <dt>Имя</dt>
    <dd>Иван Иванов</dd>
    <dt>Email</dt>
    <dd>ivan@example.com</dd>
    <dt>Должность</dt>
    <dd>Веб-разработчик</dd>
</dl>

<!-- Список определений с семантикой -->
<dl>
    <dt>HTML</dt>
    <dd>Язык гипертекстовой разметки</dd>
    <dt>CSS</dt>
    <dd>Каскадные таблицы стилей</dd>
    <dt>JavaScript</dt>
    <dd>Язык программирования для веба</dd>
</dl>

<!-- Нумерованный список с семантическим значением -->
<ol itemscope itemtype="http://schema.org/HowTo">
    <li itemprop="step" itemscope itemtype="http://schema.org/HowToStep">
        <span itemprop="name">Первый шаг</span>
        <div itemprop="itemListElement">Описание первого шага</div>
    </li>
    <li itemprop="step" itemscope itemtype="http://schema.org/HowToStep">
        <span itemprop="name">Второй шаг</span>
        <div itemprop="itemListElement">Описание второго шага</div>
    </li>
</ol>
```

### Семантика для мультимедиа
```html
<!-- Аудио с описанием -->
<figure>
    <audio controls>
        <source src="podcast.mp3" type="audio/mpeg">
        <source src="podcast.ogg" type="audio/ogg">
        Ваш браузер не поддерживает аудио элемент.
    </audio>
    <figcaption>
        Подкаст: "Современные веб-технологии" - 
        <time datetime="PT45M30S">45:30</time>
    </figcaption>
</figure>

<!-- Видео с описанием и субтитрами -->
<figure>
    <video controls width="640" height="360">
        <source src="tutorial.mp4" type="video/mp4">
        <source src="tutorial.webm" type="video/webm">
        <track kind="subtitles" src="subtitles.vtt" srclang="ru" label="Русские">
        <track kind="captions" src="captions.vtt" srclang="ru" label="Русские (для слабослышащих)">
        Ваш браузер не поддерживает видео элемент.
    </video>
    <figcaption>
        Обучающее видео: "Основы HTML" - 
        <time datetime="PT15M20S">15:20</time>
    </figcaption>
</figure>
```

## Семантические паттерны для разных типов контента

### Блог/статьи
```html
<main>
    <article itemscope itemtype="http://schema.org/BlogPosting">
        <header>
            <h1 itemprop="headline">Заголовок статьи</h1>
            <p>Автор: <span itemprop="author">Имя автора</span></p>
            <p>Опубликовано: 
                <time datetime="2023-11-08T10:00:00+00:00" itemprop="datePublished">
                    8 ноября 2023
                </time>
            </p>
            <p>Обновлено: 
                <time datetime="2023-11-09T14:30:00+00:00" itemprop="dateModified">
                    9 ноября 2023
                </time>
            </p>
        </header>

        <div itemprop="articleBody">
            <p>Содержимое статьи...</p>
        </div>

        <footer>
            <p>Теги: 
                <span itemprop="keywords">
                    <a href="/tag/html">HTML</a>, 
                    <a href="/tag/semantics">Семантика</a>
                </span>
            </p>
            <p>Категория: <span itemprop="articleSection">Веб-разработка</span></p>
        </footer>
    </article>
</main>
```

### Комментарии
```html
<section aria-labelledby="comments-heading">
    <h2 id="comments-heading">Комментарии</h2>
    
    <article class="comment" itemscope itemtype="http://schema.org/Comment">
        <header>
            <h3 itemprop="author">Алексей Сидоров</h3>
            <p>Опубликовано: 
                <time datetime="2023-11-08T12:30:00+00:00" itemprop="dateCreated">
                    8 ноября 2023, 12:30
                </time>
            </p>
        </header>
        <div itemprop="text">
            <p>Отличная статья! Особенно понравилось объяснение семантики.</p>
        </div>
        <footer>
            <button>Ответить</button>
        </footer>
    </article>
    
    <article class="comment" itemscope itemtype="http://schema.org/Comment">
        <header>
            <h3 itemprop="author">Мария Козлова</h3>
            <p>Опубликовано: 
                <time datetime="2023-11-08T15:45:00+00:00" itemprop="dateCreated">
                    8 ноября 2023, 15:45
                </time>
            </p>
        </header>
        <div itemprop="text">
            <p>Есть вопрос по поводу использования тега section...</p>
        </div>
        <footer>
            <button>Ответить</button>
        </footer>
    </article>
</section>
```

### Товары/продукты
```html
<article itemscope itemtype="http://schema.org/Product">
    <header>
        <h1 itemprop="name">Смартфон XYZ Pro</h1>
        <img src="phone.jpg" alt="Смартфон XYZ Pro" itemprop="image">
    </header>
    
    <div>
        <p itemprop="description">Современный смартфон с отличной камерой и производительным процессором.</p>
        <p>Цена: <span itemprop="offers" itemscope itemtype="http://schema.org/Offer">
            <meta itemprop="priceCurrency" content="RUB">
            <span itemprop="price">59990</span> ₽
        </span></p>
        <p>Рейтинг: <span itemprop="aggregateRating" itemscope itemtype="http://schema.org/AggregateRating">
            <span itemprop="ratingValue">4.5</span> из 
            <span itemprop="bestRating">5</span> (
            <span itemprop="reviewCount">128</span> отзывов)
        </span></p>
    </div>
    
    <footer>
        <button itemprop="potentialAction" itemscope itemtype="http://schema.org/BuyAction">
            Купить
        </button>
    </footer>
</article>
```

## Современные семантические практики

### Использование микроформатов
```html
<!-- hCard - контактная информация -->
<div class="vcard">
    <h3 class="fn">Иван Иванов</h3>
    <div class="org">Компания ABC</div>
    <div class="adr">
        <span class="locality">Москва</span>, 
        <span class="country-name">Россия</span>
    </div>
    <div class="tel">+7 (495) 123-45-67</div>
    <a class="email" href="mailto:ivan@example.com">ivan@example.com</a>
</div>

<!-- hCalendar - календарное событие -->
<div class="vevent">
    <h3 class="summary">Вебинар по HTML5</h3>
    <div class="dtstart">2023-11-15T15:00:00+03:00</div>
    <div class="dtend">2023-11-15T16:00:00+03:00</div>
    <div class="description">Обзор новых возможностей HTML5</div>
    <div class="location">Онлайн</div>
</div>
```

### Семантика для навигации
```html
<!-- Хлебные крошки -->
<nav aria-label="Навигационная цепочка">
    <ol class="breadcrumb">
        <li><a href="/">Главная</a></li>
        <li><a href="/blog">Блог</a></li>
        <li><a href="/blog/html">HTML</a></li>
        <li aria-current="page">Семантическая разметка</li>
    </ol>
</nav>

<!-- Пагинация -->
<nav aria-label="Навигация по страницам">
    <ul class="pagination">
        <li><a href="/page/1" aria-label="Предыдущая страница">‹</a></li>
        <li><a href="/page/1">1</a></li>
        <li aria-current="page">2</li>
        <li><a href="/page/3">3</a></li>
        <li><a href="/page/3" aria-label="Следующая страница">›</a></li>
    </ul>
</nav>
```

## Проверка семантической разметки

### Инструменты для проверки:
- W3C Markup Validator - проверка соответствия стандартам HTML
- HTML5 Outliner - проверка иерархии заголовков
- axe-core - проверка доступности и семантики
- WAVE - веб-инструмент для оценки доступности
- Инструменты разработчика браузера - проверка структуры DOM
- Google Rich Results Test - проверка структурированных данных

### Проверка иерархии заголовков
```html
<!-- Правильная иерархия -->
<h1>Основной заголовок страницы</h1>
<h2>Подраздел 1</h2>
<h3>Подподраздел 1.1</h3>
<h4>Еще более глубокий уровень</h4>
<h2>Подраздел 2</h2>
<h3>Подподраздел 2.1</h3>
```

## Лучшие практики семантической разметки

### 1. Использование правильных элементов
```html
<!-- Правильно: использование article для самостоятельного контента -->
<article>
    <h2>Новость дня</h2>
    <p>Содержимое новости...</p>
</article>

<!-- Неправильно: использование div для всего -->
<div>
    <div>Новость дня</div>
    <div>Содержимое новости...</div>
</div>
```

### 2. Семантика + доступность
```html
<!-- Семантически и доступно -->
<nav aria-label="Основная навигация">
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
    </ul>
</nav>

<!-- Только семантически -->
<nav>
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
    </ul>
</nav>
```

### 3. Семантика + SEO
```html
<!-- Оптимизированная семантическая разметка -->
<main>
    <article itemscope itemtype="http://schema.org/Article">
        <header>
            <h1 itemprop="headline">Заголовок статьи для SEO</h1>
            <p>Опубликовано: <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time></p>
        </header>
        <div itemprop="articleBody">
            <p>Ключевое содержимое с важными ключевыми словами...</p>
        </div>
    </article>
</main>
```

## Пример комплексной семантической разметки

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Семантическая разметка в действии</title>
    <meta name="description" content="Пример современной семантической HTML разметки">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Семантическая разметка в действии">
    <meta property="og:description" content="Пример современной семантической HTML разметки">
    <meta property="og:type" content="article">
    
    <!-- JSON-LD структурированные данные -->
    <script type="application/ld+json">
    {
        "@context": "http://schema.org",
        "@type": "Article",
        "headline": "Семантическая разметка в действии",
        "author": {
            "@type": "Person",
            "name": "Имя автора"
        },
        "datePublished": "2023-11-08",
        "dateModified": "2023-11-08",
        "description": "Пример современной семантической HTML разметки"
    }
    </script>
</head>
<body>
    <!-- Ссылка для пропуска навигации -->
    <a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
    
    <header role="banner">
        <h1>Название сайта</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/" aria-current="page">Главная</a></li>
                <li><a href="/blog">Блог</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>

    <main id="main-content" role="main">
        <article itemscope itemtype="http://schema.org/Article">
            <header>
                <h1 itemprop="headline">Современная семантическая разметка HTML</h1>
                <p>Автор: <span itemprop="author" itemscope itemtype="http://schema.org/Person">
                    <span itemprop="name">Имя автора</span>
                </span></p>
                <p>Опубликовано: 
                    <time datetime="2023-11-08T10:00:00+00:00" itemprop="datePublished">
                        8 ноября 2023
                    </time>
                </p>
            </header>

            <nav aria-label="Навигация по статье">
                <ul>
                    <li><a href="#introduction">Введение</a></li>
                    <li><a href="#basics">Основы семантики</a></li>
                    <li><a href="#advanced">Расширенные паттерны</a></li>
                    <li><a href="#best-practices">Лучшие практики</a></li>
                </ul>
            </nav>

            <section id="introduction">
                <h2>Введение в семантическую разметку</h2>
                <p itemprop="description">Семантическая разметка - это основа современного веб-дизайна...</p>
            </section>

            <section id="basics">
                <h2>Основы семантической разметки</h2>
                <p>Семантические элементы HTML5...</p>
                
                <figure>
                    <img src="semantic-elements.png" alt="Диаграмма семантических элементов HTML5" itemprop="image">
                    <figcaption>Семантические элементы HTML5</figcaption>
                </figure>
            </section>

            <section id="advanced">
                <h2>Расширенные семантические паттерны</h2>
                <p>Современные подходы к семантической разметке...</p>
            </section>

            <section id="best-practices">
                <h2>Лучшие практики</h2>
                <p>Рекомендации по использованию семантической разметки...</p>
            </section>

            <footer>
                <p>Теги: 
                    <span itemprop="keywords">
                        <a href="/tag/html" rel="tag">HTML</a>, 
                        <a href="/tag/semantics" rel="tag">Семантика</a>, 
                        <a href="/tag/accessibility" rel="tag">Доступность</a>
                    </span>
                </p>
                <p>Категория: <span itemprop="articleSection">Веб-разработка</span></p>
            </footer>
        </article>

        <aside role="complementary" aria-label="Сопутствующий контент">
            <section>
                <h2>Похожие статьи</h2>
                <ul>
                    <li><a href="/article1">Статья 1</a></li>
                    <li><a href="/article2">Статья 2</a></li>
                    <li><a href="/article3">Статья 3</a></li>
                </ul>
            </section>
            
            <section>
                <h2>Популярные теги</h2>
                <ul>
                    <li><a href="/tag/html">HTML</a></li>
                    <li><a href="/tag/css">CSS</a></li>
                    <li><a href="/tag/javascript">JavaScript</a></li>
                </ul>
            </section>
        </aside>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Название компании. Все права защищены.</p>
        <nav aria-label="Нижняя навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/terms">Условия использования</a></li>
                <li><a href="/sitemap">Карта сайта</a></li>
            </ul>
        </nav>
    </footer>
</body>
</html>
```

## Заключение

Современная семантическая разметка - это комплексный подход, который включает в себя правильное использование HTML элементов, структурированные данные, доступность и SEO оптимизацию. Понимание и применение семантических паттернов делает веб-контент более понятным как для пользователей, так и для поисковых систем и вспомогательных технологий.

## Следующие темы
- [[HTML/Семантика/Заголовки и навигация]]
- [[HTML/Семантика/Статьи и разделы]]
- [[HTML/Доступность]]
- [[HTML/SEO/Структурированные данные]]

## Теги
#html #semantics #semantic-markup #web-development #accessibility #seo #structure #html5 #structured-data #microdata
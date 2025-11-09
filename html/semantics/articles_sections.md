# HTML Семантика: Статьи и разделы

Элементы `<article>` и `<section>` являются важными компонентами семантической разметки HTML. Они помогают структурировать содержимое веб-страницы логически и делают его более понятным для браузеров, поисковых систем и пользователей с ограниченными возможностями.

## Элемент `<article>`

Элемент `<article>` представляет собой независимый, автономный фрагмент содержимого, который может быть распространён, переиздан или использован независимо от остального сайта.

### Характеристики `<article>`

- Содержимое должно быть **самодостаточным** и понятным вне контекста страницы
- Может быть **распространено** независимо от остального сайта
- Часто содержит заголовки, авторов, даты публикации и т.д.
- Может быть вложенным в другие элементы, включая `<section>`

### Примеры использования `<article>`

```html
<!-- Новостная статья -->
<article>
    <header>
        <h2>Новый релиз фреймворка</h2>
        <p>Опубликовано: <time datetime="2023-03-15">15 марта 2023</time></p>
        <p>Автор: Иван Иванов</p>
    </header>
    
    <p>Содержимое статьи о новом релизе фреймворка...</p>
    
    <footer>
        <p>Метки: <span>новости</span>, <span>технологии</span></p>
    </footer>
</article>

<!-- Комментарий пользователя -->
<article>
    <header>
        <h3>Комментарий от Алексея</h3>
        <time datetime="2023-03-16T10:30">16 марта 2023, 10:30</time>
    </header>
    <p>Отличная статья! Особенно понравилось объяснение концепции...</p>
</article>

<!-- Вложенные статьи -->
<article>
    <h2>Обзор смартфонов 2023</h2>
    <p>Обзор топовых смартфонов этого года...</p>
    
    <section>
        <h3>Сравнение камер</h3>
        <p>Сравнительный анализ камер устройств...</p>
    </section>
    
    <!-- Вложенные статьи с отдельными обзорами -->
    <article>
        <h4>Обзор iPhone 15</h4>
        <p>Подробный обзор iPhone 15...</p>
    </article>
    
    <article>
        <h4>Обзор Samsung Galaxy S23</h4>
        <p>Подробный обзор Samsung Galaxy S23...</p>
    </article>
</article>
```

## Элемент `<section>`

Элемент `<section>` представляет собой тематически связанную группу содержимого, обычно с заголовком. Он используется для логического разделения документа на части.

### Характеристики `<section>`

- Обычно содержит **заголовок** (`<h1>`-`<h6>`)
- Содержимое в пределах секции **тематически связано**
- Используется для **логического разделения** документа
- Не должен использоваться только для стилизации (для этого есть `<div>`)

### Примеры использования `<section>`

```html
<!-- Структура документа -->
<section>
    <h2>Введение</h2>
    <p>Вступительный текст документа...</p>
</section>

<section>
    <h2>Основные принципы</h2>
    <p>Описание основных принципов...</p>
    
    <section>
        <h3>Принцип первый</h3>
        <p>Подробное объяснение первого принципа...</p>
    </section>
    
    <section>
        <h3>Принцип второй</h3>
        <p>Подробное объяснение второго принципа...</p>
    </section>
</section>

<section>
    <h2>Заключение</h2>
    <p>Заключительные мысли...</p>
</section>
```

## Отличия `<article>` от `<section>`

| Критерий | `<article>` | `<section>` |
|----------|-------------|-------------|
| Независимость | Может существовать отдельно | Часть большего документа |
| Повторное использование | Может быть переиздано отдельно | Часть контекста документа |
| Заголовок | Может, но не обязательно | Обычно содержит заголовок |
| Вложение | Может содержать `<section>` | Может содержать `<article>` |

```html
<!-- Пример правильного использования обоих элементов -->
<article>
    <header>
        <h1>Статья о веб-разработке</h1>
        <p>Автор: Джон Дэвис</p>
    </header>
    
    <!-- Разделы внутри статьи -->
    <section>
        <h2>История веб-разработки</h2>
        <p>Исторический обзор развития веб-технологий...</p>
    </section>
    
    <section>
        <h2>Современные технологии</h2>
        <p>Обзор современных фреймворков и инструментов...</p>
        
        <!-- Вложенная статья с конкретным примером -->
        <article>
            <h3>Кейс: Рефакторинг старого приложения</h3>
            <p>Описание конкретного случая рефакторинга...</p>
        </article>
    </section>
    
    <footer>
        <p>Опубликовано: <time datetime="2023-03-15">15 марта 2023</time></p>
    </footer>
</article>
```

## Использование с другими семантическими элементами

### Комбинация с `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`

```html
<main>
    <article>
        <header>
            <h1>Заголовок статьи</h1>
            <p>Опубликовано: <time datetime="2023-03-15">15 марта 2023</time></p>
        </header>
        
        <section>
            <h2>Введение</h2>
            <p>Вступительная часть статьи...</p>
        </section>
        
        <section>
            <h2>Основная часть</h2>
            <p>Основное содержимое статьи...</p>
            
            <aside>
                <h3>Интересный факт</h3>
                <p>Дополнительная информация, связанная с темой...</p>
            </aside>
        </section>
        
        <footer>
            <p>Автор: Иван Иванов</p>
            <nav>
                <ul>
                    <li><a href="#comments">Комментарии</a></li>
                    <li><a href="#related">Похожие статьи</a></li>
                </ul>
            </nav>
        </footer>
    </article>
</main>
```

### Использование в блоге

```html
<main>
    <article>
        <header>
            <h1>Как улучшить производительность веб-сайта</h1>
            <p>Автор: <a href="/author/john">Иван Петров</a></p>
            <p>Опубликовано: <time datetime="2023-03-10">10 марта 2023</time></p>
        </header>
        
        <section>
            <h2>Оптимизация изображений</h2>
            <p>Методы оптимизации изображений для улучшения скорости загрузки...</p>
        </section>
        
        <section>
            <h2>Минификация CSS и JavaScript</h2>
            <p>Как уменьшить размер файлов CSS и JavaScript...</p>
        </section>
        
        <section>
            <h2>Использование CDN</h2>
            <p>Преимущества использования CDN для ускорения загрузки...</p>
        </section>
        
        <footer>
            <p>Категории: <a href="/category/performance">Производительность</a>, <a href="/category/optimization">Оптимизация</a></p>
            <p>Метки: <span>оптимизация</span>, <span>производительность</span>, <span>заголовок</span></p>
        </footer>
    </article>
    
    <!-- Комментарии как отдельные статьи -->
    <section>
        <h2>Комментарии</h2>
        <article>
            <header>
                <h3>Комментарий от Александра</h3>
                <time datetime="2023-03-11T09:15">11 марта 2023, 09:15</time>
            </header>
            <p>Отличная статья! Особенно помогла оптимизация изображений.</p>
        </article>
        
        <article>
            <header>
                <h3>Комментарий от Марии</h3>
                <time datetime="2023-03-12T14:30">12 марта 2023, 14:30</time>
            </header>
            <p>Спасибо за полезные советы. Вопрос: как быть с SVG?</p>
        </article>
    </section>
</main>
```

## Распространенные ошибки

### 1. Использование `<div>` вместо семантических элементов

```html
<!-- Неправильно -->
<div class="article">
    <h2>Заголовок статьи</h2>
    <p>Содержимое статьи...</p>
</div>

<!-- Правильно -->
<article>
    <h2>Заголовок статьи</h2>
    <p>Содержимое статьи...</p>
</article>
```

### 2. Неправильное вложение

```html
<!-- Неправильно - section без заголовка -->
<section>
    <p>Содержимое без заголовка</p>
</section>

<!-- Правильно -->
<section>
    <h2>Заголовок секции</h2>
    <p>Содержимое секции</p>
</section>
```

### 3. Использование для стилизации

```html
<!-- Неправильно - только для стилизации -->
<section class="blue-background">
    <p>Текст в синем фоне</p>
</section>

<!-- Правильно - для логического разделения -->
<div class="blue-background">
    <p>Текст в синем фоне</p>
</div>
```

## Доступность и SEO

Использование `<article>` и `<section>` улучшает:

1. **Доступность** - скринридеры могут лучше ориентироваться в структуре документа
2. **SEO** - поисковые системы лучше понимают структуру и иерархию содержимого
3. **Поддерживаемость** - код становится более понятным для других разработчиков

```html
<!-- Пример с ARIA-ролями для дополнительной доступности -->
<article role="article" aria-labelledby="article-title">
    <header>
        <h1 id="article-title">Заголовок статьи</h1>
    </header>
    
    <section role="region" aria-labelledby="section1-title">
        <h2 id="section1-title">Первый раздел</h2>
        <p>Содержимое первого раздела...</p>
    </section>
    
    <section role="region" aria-labelledby="section2-title">
        <h2 id="section2-title">Второй раздел</h2>
        <p>Содержимое второго раздела...</p>
    </section>
</article>
```

## Заключение

Правильное использование элементов `<article>` и `<section>` является ключевым аспектом семантической разметки. Они помогают создать логическую структуру документа, улучшают доступность и SEO. Важно понимать различия между этими элементами и использовать их по назначению, а не просто для стилизации.

## Современные семантические паттерны с article и section

### Вложенные статьи и разделы
```html
<!-- Статья с вложенными разделами и подстатьями -->
<article itemscope itemtype="http://schema.org/Article">
    <header>
        <h1 itemprop="headline">Комплексное руководство по HTML5</h1>
        <p>Автор: <span itemprop="author">Иван Петров</span></p>
        <p>Опубликовано: 
            <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time>
        </p>
    </header>

    <section itemprop="articleBody">
        <h2>Введение в HTML5</h2>
        <p itemprop="description">HTML5 представляет собой значительное обновление предыдущих версий...</p>
    </section>

    <section itemprop="articleBody">
        <h2>Семантические элементы</h2>
        
        <section>
            <h3>Элемент article</h3>
            <p>Элемент article используется для...</p>
        </section>
        
        <section>
            <h3>Элемент section</h3>
            <p>Элемент section используется для...</p>
            
            <!-- Вложенная статья с примером кода -->
            <article>
                <h4>Пример использования section</h4>
                <pre><code>&lt;section&gt;
    &lt;h3&gt;Заголовок раздела&lt;/h3&gt;
    &lt;p&gt;Содержимое раздела...&lt;/p&gt;
&lt;/section&gt;</code></pre>
            </article>
        </section>
    </section>

    <section itemprop="articleBody">
        <h2>Практические примеры</h2>
        <p>Рассмотрим, как использовать эти элементы на практике...</p>
    </section>

    <footer>
        <p>Категория: <span itemprop="articleSection">Веб-разработка</span></p>
        <p>Теги: 
            <span itemprop="keywords">
                <a href="/tag/html">HTML</a>, 
                <a href="/tag/semantics">Семантика</a>
            </span>
        </p>
    </footer>
</article>
```

### Структура новостного сайта
```html
<main>
    <!-- Главная статья дня -->
    <article class="featured-article" itemscope itemtype="http://schema.org/NewsArticle">
        <header>
            <h1 itemprop="headline">Главные новости дня</h1>
            <p>Опубликовано: 
                <time datetime="2023-11-08T08:00:00+00:00" itemprop="datePublished">8 ноября 2023, 11:00</time>
            </p>
            <p>Автор: <span itemprop="author">Анна Сидорова</span></p>
        </header>

        <div itemprop="image" itemscope itemtype="http://schema.org/ImageObject">
            <img src="news-image.jpg" alt="Описание изображения" itemprop="url">
        </div>

        <div itemprop="articleBody">
            <p itemprop="description">Полный текст главной новости дня...</p>
        </div>

        <footer>
            <p>Рейтинг: 
                <span itemprop="aggregateRating" itemscope itemtype="http://schema.org/AggregateRating">
                    <span itemprop="ratingValue">4.7</span> из 5
                </span>
            </p>
        </footer>
    </article>

    <!-- Раздел с другими новостями -->
    <section aria-labelledby="other-news">
        <h2 id="other-news">Другие новости</h2>
        
        <article itemscope itemtype="http://schema.org/NewsArticle">
            <header>
                <h3 itemprop="headline">
                    <a href="/news/politics" itemprop="url">Политические новости</a>
                </h3>
                <p>Категория: <span itemprop="articleSection">Политика</span></p>
            </header>
            <p itemprop="description">Краткое описание политической новости...</p>
        </article>

        <article itemscope itemtype="http://schema.org/NewsArticle">
            <header>
                <h3 itemprop="headline">
                    <a href="/news/technology" itemprop="url">Технологические новшества</a>
                </h3>
                <p>Категория: <span itemprop="articleSection">Технологии</span></p>
            </header>
            <p itemprop="description">Краткое описание технологической новости...</p>
        </article>
    </section>

    <!-- Комментарии как отдельные статьи -->
    <section aria-labelledby="comments-section">
        <h2 id="comments-section">Комментарии</h2>
        
        <article class="comment" itemscope itemtype="http://schema.org/Comment">
            <header>
                <h3 itemprop="author">Алексей Иванов</h3>
                <p>Опубликовано: 
                    <time datetime="2023-11-08T10:30:00+00:00" itemprop="dateCreated">13:30</time>
                </p>
            </header>
            <div itemprop="text">
                <p>Очень интересная статья! Особенно понравилось...</p>
            </div>
        </article>
        
        <article class="comment" itemscope itemtype="http://schema.org/Comment">
            <header>
                <h3 itemprop="author">Мария Козлова</h3>
                <p>Опубликовано: 
                    <time datetime="2023-11-08T11:15:00+00:00" itemprop="dateCreated">14:15</time>
                </p>
            </header>
            <div itemprop="text">
                <p>Спасибо за развернутый обзор...</p>
            </div>
        </article>
    </section>
</main>
```

## Расширенные примеры использования article и section

### Блог с тегами и категориями
```html
<main>
    <article itemscope itemtype="http://schema.org/BlogPosting">
        <header>
            <h1 itemprop="headline">Руководство по современным веб-технологиям</h1>
            <p>Автор: <span itemprop="author">Иван Иванов</span></p>
            <p>Опубликовано: 
                <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time>
            </p>
        </header>

        <section itemprop="articleBody">
            <h2>Введение</h2>
            <p itemprop="description">Современные веб-технологии развиваются стремительными темпами...</p>
        </section>

        <section itemprop="articleBody">
            <h2>HTML5 и семантика</h2>
            <p>HTML5 предоставляет множество новых семантических элементов...</p>
            
            <section>
                <h3>Элемент article</h3>
                <p>Элемент article используется для...</p>
            </section>
            
            <section>
                <h3>Элемент section</h3>
                <p>Элемент section используется для...</p>
            </section>
        </section>

        <section itemprop="articleBody">
            <h2>Практические примеры</h2>
            <p>Рассмотрим несколько практических примеров...</p>
        </section>

        <footer>
            <p>Категория: <span itemprop="articleSection">Веб-разработка</span></p>
            <p>Теги: 
                <span itemprop="keywords">
                    <a href="/tag/html" rel="tag">HTML</a>, 
                    <a href="/tag/css" rel="tag">CSS</a>, 
                    <a href="/tag/javascript" rel="tag">JavaScript</a>
                </span>
            </p>
        </footer>
    </article>

    <!-- Похожие статьи -->
    <section aria-labelledby="related-articles">
        <h2 id="related-articles">Похожие статьи</h2>
        <div class="related-articles-grid">
            <article itemscope itemtype="http://schema.org/BlogPosting">
                <h3 itemprop="headline">
                    <a href="/post/advanced-css" itemprop="url">Продвинутый CSS</a>
                </h3>
                <p itemprop="description">Изучаем продвинутые возможности CSS...</p>
            </article>
            
            <article itemscope itemtype="http://schema.org/BlogPosting">
                <h3 itemprop="headline">
                    <a href="/post/javascript-features" itemprop="url">Нововведения JavaScript</a>
                </h3>
                <p itemprop="description">Что нового в современном JavaScript...</p>
            </article>
        </div>
    </section>
</main>
```

### Документация с разделами и подразделами
```html
<main>
    <article itemscope itemtype="http://schema.org/TechArticle">
        <header>
            <h1 itemprop="headline">Документация по API</h1>
            <p>Версия: <span itemprop="version">v2.1.0</span></p>
            <p>Последнее обновление: 
                <time datetime="2023-11-08" itemprop="dateModified">8 ноября 2023</time>
            </p>
        </header>

        <nav aria-labelledby="api-nav">
            <h2 id="api-nav">Навигация по документации</h2>
            <ul>
                <li><a href="#introduction">Введение</a></li>
                <li><a href="#authentication">Аутентификация</a></li>
                <li><a href="#endpoints">Эндпоинты</a></li>
                <li><a href="#examples">Примеры</a></li>
            </ul>
        </nav>

        <section id="introduction" itemprop="articleBody">
            <h2>Введение</h2>
            <p itemprop="description">Наше API предоставляет доступ к различным функциям...</p>
        </section>

        <section id="authentication" itemprop="articleBody">
            <h2>Аутентификация</h2>
            <p>Для доступа к API требуется аутентификация с помощью токена...</p>
            
            <section>
                <h3>Получение токена</h3>
                <p>Для получения токена выполните следующий запрос...</p>
                <pre><code>POST /api/auth/token</code></pre>
            </section>
            
            <section>
                <h3>Использование токена</h3>
                <p>Токен должен быть передан в заголовке Authorization...</p>
            </section>
        </section>

        <section id="endpoints" itemprop="articleBody">
            <h2>Эндпоинты</h2>
            
            <section>
                <h3>Получение данных пользователя</h3>
                <p>Эндпоинт <code>/api/user</code> возвращает информацию о пользователе...</p>
            </section>
            
            <section>
                <h3>Обновление данных</h3>
                <p>Эндпоинт <code>/api/user/update</code> позволяет обновить данные...</p>
            </section>
        </section>

        <section id="examples" itemprop="articleBody">
            <h2>Примеры использования</h2>
            
            <article>
                <h3>Пример запроса на JavaScript</h3>
                <pre><code>fetch('/api/user', {
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN'
  }
})</code></pre>
            </article>
            
            <article>
                <h3>Пример запроса на Python</h3>
                <pre><code>import requests

response = requests.get('/api/user', headers={
  'Authorization': 'Bearer YOUR_TOKEN'
})</code></pre>
            </article>
        </section>

        <footer>
            <p>Поддержка: <a href="mailto:support@example.com">support@example.com</a></p>
        </footer>
    </article>
</main>
```

## Доступность с article и section

### Использование ARIA с семантическими элементами
```html
<!-- Статья с дополнительной ARIA семантикой -->
<article role="article" aria-labelledby="article-title" itemscope itemtype="http://schema.org/Article">
    <header>
        <h1 id="article-title" itemprop="headline">Заголовок статьи</h1>
    </header>

    <section role="region" aria-labelledby="intro-title" itemprop="articleBody">
        <h2 id="intro-title">Введение</h2>
        <p itemprop="description">Вступительный текст статьи...</p>
    </section>

    <section role="region" aria-labelledby="main-title" itemprop="articleBody">
        <h2 id="main-title">Основная часть</h2>
        <p>Основное содержимое статьи...</p>
        
        <section role="region" aria-labelledby="subsection-title">
            <h3 id="subsection-title">Подраздел</h3>
            <p>Содержимое подраздела...</p>
        </section>
    </section>

    <footer role="contentinfo">
        <p>Опубликовано: <time datetime="2023-11-08">8 ноября 2023</time></p>
    </footer>
</article>

<!-- Раздел с новостями -->
<section role="feed" aria-label="Лента новостей">
    <h2>Последние новости</h2>
    
    <article role="article" aria-labelledby="news1-title">
        <header>
            <h3 id="news1-title">Заголовок новости 1</h3>
        </header>
        <p>Краткое содержание новости...</p>
    </article>
    
    <article role="article" aria-labelledby="news2-title">
        <header>
            <h3 id="news2-title">Заголовок новости 2</h3>
        </header>
        <p>Краткое содержание новости...</p>
    </article>
</section>
```

### Структура для скринридеров
```html
<!-- Четкая иерархия заголовков -->
<main>
    <article>
        <h1>Заголовок статьи</h1>
        
        <section>
            <h2>Первый раздел</h2>
            <p>Содержимое первого раздела...</p>
            
            <section>
                <h3>Подраздел первого раздела</h3>
                <p>Содержимое подраздела...</p>
            </section>
        </section>
        
        <section>
            <h2>Второй раздел</h2>
            <p>Содержимое второго раздела...</p>
            
            <article>
                <h3>Вложенная статья</h3>
                <p>Содержимое вложенной статьи...</p>
            </article>
        </section>
    </article>
</main>

<!-- Альтернативная структура с пояснениями для вспомогательных технологий -->
<main>
    <article aria-label="Основная статья о веб-разработке">
        <header>
            <h1>Современные подходы к веб-разработке</h1>
        </header>

        <section aria-labelledby="background-title">
            <h2 id="background-title">Исторический контекст</h2>
            <p>Развитие веб-технологий началось с...</p>
        </section>

        <section aria-labelledby="current-state-title">
            <h2 id="current-state-title">Современное состояние</h2>
            <p>Сегодня веб-разработка включает в себя...</p>
        </section>

        <section aria-labelledby="future-directions-title">
            <h2 id="future-directions-title">Будущие направления</h2>
            <p>Перспективные технологии включают в себя...</p>
        </section>

        <footer aria-label="Информация об авторе">
            <p>Автор: Иван Иванов</p>
            <p>Опубликовано: 8 ноября 2023</p>
        </footer>
    </article>
</main>
```

## Современные паттерны использования

### Статьи в социальной сети
```html
<main>
    <!-- Пост пользователя -->
    <article class="user-post" itemscope itemtype="http://schema.org/SocialMediaPosting">
        <header>
            <div class="post-author">
                <img src="avatar.jpg" alt="Аватар пользователя" itemprop="author" itemscope itemtype="http://schema.org/Person">
                <div>
                    <h3 itemprop="name">Имя пользователя</h3>
                    <p>Опубликовано: 
                        <time datetime="2023-11-08T15:30:00+00:00" itemprop="datePublished">30 минут назад</time>
                    </p>
                </div>
            </div>
        </header>

        <section class="post-content" itemprop="articleBody">
            <p itemprop="articleBody">Текст поста пользователя...</p>
            
            <!-- Вложенные статьи с комментариями -->
            <article class="comment" itemscope itemtype="http://schema.org/Comment">
                <header>
                    <h4 itemprop="author">Комментатор 1</h4>
                    <time datetime="2023-11-08T15:35:00+00:00" itemprop="dateCreated">5 минут назад</time>
                </header>
                <p itemprop="text">Интересный комментарий к посту...</p>
            </article>
        </section>

        <footer class="post-actions">
            <button>Нравится</button>
            <button>Комментировать</button>
            <button>Поделиться</button>
        </footer>
    </article>
</main>
```

### Структура электронной книги
```html
<main>
    <article itemscope itemtype="http://schema.org/Book">
        <header>
            <h1 itemprop="headline">Заголовок книги</h1>
            <p>Автор: <span itemprop="author">Имя автора</span></p>
            <p>Издательство: <span itemprop="publisher">Название издательства</span></p>
        </header>

        <nav aria-labelledby="toc-title">
            <h2 id="toc-title">Содержание</h2>
            <ol>
                <li><a href="#chapter1">Глава 1</a></li>
                <li><a href="#chapter2">Глава 2</a></li>
                <li><a href="#chapter3">Глава 3</a></li>
            </ol>
        </nav>

        <section id="chapter1" itemprop="hasPart" itemscope itemtype="http://schema.org/Chapter">
            <h2 itemprop="headline">Первая глава</h2>
            <p itemprop="description">Содержимое первой главы...</p>
            
            <section itemprop="hasPart" itemscope itemtype="http://schema.org/Section">
                <h3 itemprop="headline">Подраздел главы</h3>
                <p>Содержимое подраздела...</p>
            </section>
        </section>

        <section id="chapter2" itemprop="hasPart" itemscope itemtype="http://schema.org/Chapter">
            <h2 itemprop="headline">Вторая глава</h2>
            <p itemprop="description">Содержимое второй главы...</p>
        </section>

        <section id="chapter3" itemprop="hasPart" itemscope itemtype="http://schema.org/Chapter">
            <h2 itemprop="headline">Третья глава</h2>
            <p itemprop="description">Содержимое третьей главы...</p>
        </section>

        <footer>
            <p>ISBN: <span itemprop="isbn">123-456-789</span></p>
            <p>Дата публикации: <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time></p>
        </footer>
    </article>
</main>
```

## Лучшие практики использования article и section

### 1. Правильное определение независимости контента
```html
<!-- Хорошо: статья может существовать отдельно -->
<article itemscope itemtype="http://schema.org/Article">
    <h2 itemprop="headline">Как использовать элемент article</h2>
    <p itemprop="description">Полное руководство по использованию элемента article...</p>
</article>

<!-- Хорошо: раздел части более крупного документа -->
<section>
    <h2>Преимущества семантической разметки</h2>
    <p>Семантическая разметка улучшает доступность...</p>
</section>

<!-- Плохо: неправильное использование article для подраздела -->
<article>
    <h3>Подраздел основной статьи</h3>
    <p>Это не отдельная статья, а часть большего документа</p>
</article>

<!-- Правильно: использование section -->
<section>
    <h3>Подраздел основной статьи</h3>
    <p>Это часть большего документа</p>
</section>
```

### 2. Правильная иерархия заголовков
```html
<!-- Правильная иерархия -->
<article>
    <h1>Заголовок статьи</h1>
    
    <section>
        <h2>Раздел 1</h2>
        <p>Содержимое раздела 1...</p>
        
        <section>
            <h3>Подраздел 1.1</h3>
            <p>Содержимое подраздела...</p>
        </section>
    </section>
    
    <section>
        <h2>Раздел 2</h2>
        <p>Содержимое раздела 2...</p>
    </section>
</article>

<!-- Неправильная иерархия -->
<article>
    <h1>Заголовок статьи</h1>
    
    <section>
        <h4>Раздел 1</h4> <!-- Пропущены h2 и h3 -->
        <p>Содержимое...</p>
    </section>
</article>
```

### 3. Использование с микроформатами и структурированными данными
```html
<article itemscope itemtype="http://schema.org/BlogPosting">
    <header>
        <h1 itemprop="headline">Современные веб-технологии</h1>
        <p>Автор: <span itemprop="author" itemscope itemtype="http://schema.org/Person">
            <span itemprop="name">Имя автора</span>
        </span></p>
        <p>Опубликовано: 
            <time datetime="2023-11-08T10:00:00+00:00" itemprop="datePublished">8 ноября 2023</time>
        </p>
    </header>

    <section itemprop="articleBody">
        <h2>Введение</h2>
        <p itemprop="description">Современные веб-технологии включают в себя...</p>
    </section>

    <section itemprop="articleBody">
        <h2>Основные технологии</h2>
        <p>Ключевые технологии современного веба...</p>
    </section>

    <footer>
        <p>Категория: <span itemprop="articleSection">Веб-разработка</span></p>
        <p>Теги: 
            <span itemprop="keywords">
                <a href="/tag/html" rel="tag">HTML</a>, 
                <a href="/tag/css" rel="tag">CSS</a>, 
                <a href="/tag/javascript" rel="tag">JavaScript</a>
            </span>
        </p>
    </footer>
</article>
```

## Пример комплексной страницы с article и section

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Семантические элементы article и section</title>
    <meta name="description" content="Руководство по использованию семантических элементов article и section">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Семантические элементы article и section">
    <meta property="og:description" content="Руководство по использованию семантических элементов article и section">
    <meta property="og:type" content="article">
    
    <!-- Структурированные данные -->
    <script type="application/ld+json">
    {
        "@context": "http://schema.org",
        "@type": "Article",
        "headline": "Семантические элементы article и section",
        "author": {
            "@type": "Person",
            "name": "Имя автора"
        },
        "datePublished": "2023-11-08",
        "dateModified": "2023-11-08",
        "description": "Руководство по использованию семантических элементов article и section",
        "articleSection": "Веб-разработка",
        "keywords": "HTML, семантика, article, section, веб-разработка"
    }
    </script>
</head>
<body>
    <header role="banner">
        <h1>Веб-разработка</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/blog">Блог</a></li>
                <li><a href="/tutorials">Обучение</a></li>
                <li><a href="/about">О нас</a></li>
            </ul>
        </nav>
    </header>

    <main role="main">
        <article itemscope itemtype="http://schema.org/Article">
            <header>
                <h1 itemprop="headline">Понимание элементов article и section в HTML5</h1>
                <p>Автор: <span itemprop="author">Иван Иванов</span></p>
                <p>Опубликовано: 
                    <time datetime="2023-11-08T10:00:00+00:00" itemprop="datePublished">8 ноября 2023</time>
                </p>
            </header>

            <section itemprop="articleBody">
                <h2>Введение</h2>
                <p itemprop="description">HTML5 ввел множество новых семантических элементов, 
                которые помогают лучше структурировать веб-страницы. Два из них — 
                <code>&lt;article&gt;</code> и <code>&lt;section&gt;</code> — 
                часто вызывают вопросы у разработчиков.</p>
            </section>

            <section itemprop="articleBody">
                <h2>Элемент article</h2>
                <p>Элемент <code>&lt;article&gt;</code> представляет собой независимый, 
                автономный фрагмент содержимого, который может быть распространён, 
                переиздан или использован независимо от остального сайта.</p>
                
                <section>
                    <h3>Когда использовать article</h3>
                    <ul>
                        <li>Статьи в блоге</li>
                        <li>Новостные статьи</li>
                        <li>Комментарии пользователей</li>
                        <li>Форумные посты</li>
                        <li>Виджеты социальных сетей</li>
                    </ul>
                </section>
            </section>

            <section itemprop="articleBody">
                <h2>Элемент section</h2>
                <p>Элемент <code>&lt;section&gt;</code> представляет собой тематически 
                связанную группу содержимого, обычно с заголовком. Он используется для 
                логического разделения документа на части.</p>
                
                <section>
                    <h3>Когда использовать section</h3>
                    <ul>
                        <li>Главы в документе</li>
                        <li>Разделы с разными темами</li>
                        <li>Группы связанных элементов</li>
                        <li>Части формы</li>
                    </ul>
                </section>
                
                <section>
                    <h3>Примеры использования</h3>
                    <p>Рассмотрим несколько практических примеров...</p>
                    
                    <article>
                        <h4>Пример статьи внутри раздела</h4>
                        <p>Иногда бывает нужно вложить статью внутрь раздела...</p>
                    </article>
                </section>
            </section>

            <section itemprop="articleBody">
                <h2>Совместное использование</h2>
                <p>Элементы <code>&lt;article&gt;</code> и <code>&lt;section&gt;</code> 
                могут использоваться вместе для создания сложных структур.</p>
                
                <pre><code>&lt;article&gt;
    &lt;header&gt;
        &lt;h1&gt;Заголовок статьи&lt;/h1&gt;
    &lt;/header&gt;
    
    &lt;section&gt;
        &lt;h2&gt;Раздел 1&lt;/h2&gt;
        &lt;p&gt;Содержимое первого раздела...&lt;/p&gt;
    &lt;/section&gt;
    
    &lt;section&gt;
        &lt;h2&gt;Раздел 2&lt;/h2&gt;
        &lt;p&gt;Содержимое второго раздела...&lt;/p&gt;
    &lt;/section&gt;
&lt;/article&gt;</code></pre>
            </section>

            <section itemprop="articleBody">
                <h2>Лучшие практики</h2>
                <p>При использовании этих элементов следуйте следующим рекомендациям:</p>
                
                <ol>
                    <li>Используйте <code>&lt;article&gt;</code> для контента, 
                    который может существовать отдельно</li>
                    <li>Используйте <code>&lt;section&gt;</code> для логических 
                    разделов документа</li>
                    <li>Не используйте их только для стилизации — для этого есть <code>&lt;div&gt;</code></li>
                    <li>Следите за иерархией заголовков</li>
                    <li>Добавляйте структурированные данные при необходимости</li>
                </ol>
            </section>

            <footer>
                <p>Категория: <span itemprop="articleSection">HTML</span></p>
                <p>Теги: 
                    <span itemprop="keywords">
                        <a href="/tag/html" rel="tag">HTML</a>, 
                        <a href="/tag/semantics" rel="tag">Семантика</a>, 
                        <a href="/tag/structure" rel="tag">Структура</a>
                    </span>
                </p>
                <p>Последнее обновление: 
                    <time datetime="2023-11-08T14:30:00+00:00" itemprop="dateModified">8 ноября 2023, 17:30</time>
                </p>
            </footer>
        </article>

        <aside role="complementary" aria-labelledby="related-content">
            <h2 id="related-content">Связанный контент</h2>
            
            <section>
                <h3>Похожие статьи</h3>
                <ul>
                    <li><a href="/post/html5-semantic-elements">Другие семантические элементы HTML5</a></li>
                    <li><a href="/post/web-accessibility">Доступность веб-контента</a></li>
                    <li><a href="/post/css-architecture">Архитектура CSS</a></li>
                </ul>
            </section>
            
            <section>
                <h3>Популярные теги</h3>
                <ul>
                    <li><a href="/tag/html">HTML</a></li>
                    <li><a href="/tag/css">CSS</a></li>
                    <li><a href="/tag/javascript">JavaScript</a></li>
                </ul>
            </section>
        </aside>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Веб-разработка. Все права защищены.</p>
        <nav aria-label="Нижняя навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/terms">Условия использования</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </footer>
</body>
</html>
```

## Современные возможности article и section

### Использование с веб-компонентами
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Article и Section с веб-компонентами</title>
</head>
<body>
    <main>
        <blog-article title="Современные веб-компоненты">
            <article-content>
                <section>
                    <h2>Введение в веб-компоненты</h2>
                    <p>Веб-компоненты позволяют создавать переиспользуемые элементы...</p>
                </section>
                
                <section>
                    <h2>Практическое применение</h2>
                    <p>Рассмотрим, как использовать веб-компоненты на практике...</p>
                    
                    <blog-article title="Вложенный компонент">
                        <article-content>
                            <section>
                                <h3>Вложенные компоненты</h3>
                                <p>Веб-компоненты могут быть вложены друг в друга...</p>
                            </section>
                        </article-content>
                    </blog-article>
                </section>
            </article-content>
        </blog-article>
    </main>

    <script>
        // Кастомный элемент для статьи в блоге
        class BlogArticle extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                const title = this.getAttribute('title') || 'Без заголовка';
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .article-header {
                            background-color: #f5f5f5;
                            padding: 1rem;
                            border-left: 4px solid #007acc;
                            margin-bottom: 1rem;
                        }
                        
                        .article-title {
                            margin: 0;
                            color: #333;
                        }
                        
                        .article-content {
                            padding: 1rem;
                        }
                    </style>
                    
                    <div class="article-header">
                        <h2 class="article-title">${title}</h2>
                    </div>
                    <div class="article-content">
                        <slot name="content"></slot>
                    </div>
                `;
            }
        }
        
        // Регистрация кастомного элемента
        customElements.define('blog-article', BlogArticle);
    </script>
</body>
</html>
```

### Использование с WebSockets и динамическим контентом
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Динамические статьи с WebSocket</title>
</head>
<body>
    <main>
        <article id="live-blog-post" data-post-id="123">
            <header>
                <h1>Живой блог: Новости в реальном времени</h1>
                <p>Автор: <span id="author">Иван Иванов</span></p>
            </header>
            
            <section id="updates-container">
                <h2>Обновления</h2>
                <!-- Сюда будут добавляться новые обновления -->
            </section>
            
            <footer>
                <p>Последнее обновление: <span id="last-update">-</span></p>
            </footer>
        </article>
    </main>

    <script>
        // Подключение к WebSocket для получения обновлений
        const ws = new WebSocket('ws://localhost:8080/live-updates');
        
        ws.onmessage = function(event) {
            const update = JSON.parse(event.data);
            
            // Создание новой секции для обновления
            const section = document.createElement('section');
            section.className = 'live-update';
            section.setAttribute('aria-label', `Обновление от ${update.timestamp}`);
            
            section.innerHTML = `
                <h3>${update.title}</h3>
                <p>${update.content}</p>
                <time datetime="${update.timestamp}">${formatDate(update.timestamp)}</time>
            `;
            
            document.getElementById('updates-container').appendChild(section);
            
            // Обновление времени последнего обновления
            document.getElementById('last-update').textContent = formatDate(update.timestamp);
        };
        
        function formatDate(dateString) {
            return new Date(dateString).toLocaleString('ru-RU');
        }
    </script>
</body>
</html>
```

### Использование с IndexedDB для оффлайн-чтения
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оффлайн статьи с IndexedDB</title>
</head>
<body>
    <main>
        <article id="offline-article" data-article-id="456">
            <header>
                <h1>Статья для оффлайн-чтения</h1>
                <button id="save-offline-btn">Сохранить для оффлайн-чтения</button>
            </header>
            
            <section>
                <h2>Введение</h2>
                <p>Содержимое статьи, которое будет доступно в оффлайн-режиме...</p>
            </section>
            
            <section>
                <h2>Основная часть</h2>
                <p>Основное содержимое статьи...</p>
            </section>
        </article>
    </main>

    <script>
        // Открытие базы данных IndexedDB
        const request = indexedDB.open('OfflineArticles', 1);
        
        let db;
        
        request.onsuccess = function(event) {
            db = event.target.result;
        };
        
        request.onerror = function(event) {
            console.error('Ошибка при открытии IndexedDB:', event.target.error);
        };
        
        request.onupgradeneeded = function(event) {
            db = event.target.result;
            
            // Создание объектного хранилища для статей
            if (!db.objectStoreNames.contains('articles')) {
                const store = db.createObjectStore('articles', { keyPath: 'id' });
                store.createIndex('title', 'title', { unique: false });
                store.createIndex('date', 'date', { unique: false });
            }
        };
        
        // Сохранение статьи в оффлайн-режиме
        document.getElementById('save-offline-btn').addEventListener('click', function() {
            const articleElement = document.getElementById('offline-article');
            const articleId = articleElement.getAttribute('data-article-id');
            
            // Сбор содержимого статьи
            const articleContent = {
                id: articleId,
                title: document.querySelector('h1').textContent,
                content: articleElement.innerHTML,
                date: new Date().toISOString()
            };
            
            const transaction = db.transaction(['articles'], 'readwrite');
            const store = transaction.objectStore('articles');
            
            store.put(articleContent);
            
            transaction.oncomplete = function() {
                alert('Статья сохранена для оффлайн-чтения');
            };
            
            transaction.onerror = function() {
                alert('Ошибка при сохранении статьи');
            };
        });
    </script>
</body>
</html>
```

## Практические примеры и шаблоны

### Шаблон для новостного сайта
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Новостной портал</title>
    <style>
        .news-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .featured-article {
            border: 2px solid #007acc;
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 30px;
        }
        
        .news-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
        }
        
        .news-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            transition: box-shadow 0.3s;
        }
        
        .news-card:hover {
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
        }
        
        .news-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
            border-radius: 4px;
            margin-bottom: 10px;
        }
    </style>
</head>
<body>
    <div class="news-container">
        <header>
            <h1>Новостной портал</h1>
            <nav aria-label="Навигация по категориям">
                <ul>
                    <li><a href="/politics">Политика</a></li>
                    <li><a href="/technology">Технологии</a></li>
                    <li><a href="/sports">Спорт</a></li>
                    <li><a href="/culture">Культура</a></li>
                </ul>
            </nav>
        </header>
        
        <main>
            <!-- Главная новость дня -->
            <article class="featured-article" itemscope itemtype="http://schema.org/NewsArticle">
                <header>
                    <h1 itemprop="headline">Главная новость дня</h1>
                    <p>Автор: <span itemprop="author">Анна Сидорова</span></p>
                    <p>Опубликовано:
                        <time datetime="2023-11-08T08:00:00+00:00" itemprop="datePublished">8 ноября 2023, 11:00</time>
                    </p>
                </header>
                
                <div itemprop="image" itemscope itemtype="http://schema.org/ImageObject">
                    <img src="main-news.jpg" alt="Изображение к главной новости" class="news-image" itemprop="url">
                </div>
                
                <div itemprop="articleBody">
                    <p itemprop="description">Полный текст главной новости дня с подробным анализом ситуации...</p>
                </div>
                
                <footer>
                    <p>Категория: <span itemprop="articleSection">Политика</span></p>
                    <p>Теги:
                        <span itemprop="keywords">
                            <a href="/tag/elections" rel="tag">Выборы</a>,
                            <a href="/tag/government" rel="tag">Правительство</a>
                        </span>
                    </p>
                </footer>
            </article>
            
            <!-- Лента новостей -->
            <section aria-labelledby="news-feed-title">
                <h2 id="news-feed-title">Лента новостей</h2>
                
                <div class="news-grid">
                    <article class="news-card" itemscope itemtype="http://schema.org/NewsArticle">
                        <header>
                            <h3 itemprop="headline">
                                <a href="/news/tech-breakthrough" itemprop="url">Прорыв в технологиях</a>
                            </h3>
                            <p>Категория: <span itemprop="articleSection">Технологии</span></p>
                        </header>
                        <p itemprop="description">Ученые объявили о значительном прорыве в области квантовых вычислений...</p>
                    </article>
                    
                    <article class="news-card" itemscope itemtype="http://schema.org/NewsArticle">
                        <header>
                            <h3 itemprop="headline">
                                <a href="/news/sports-championship" itemprop="url">Чемпионат мира по футболу</a>
                            </h3>
                            <p>Категория: <span itemprop="articleSection">Спорт</span></p>
                        </header>
                        <p itemprop="description">Финал чемпионата мира завершился сенсационной победой...</p>
                    </article>
                    
                    <article class="news-card" itemscope itemtype="http://schema.org/NewsArticle">
                        <header>
                            <h3 itemprop="headline">
                                <a href="/news/cultural-festival" itemprop="url">Международный фестиваль искусств</a>
                            </h3>
                            <p>Категория: <span itemprop="articleSection">Культура</span></p>
                        </header>
                        <p itemprop="description">В Париже стартовал международный фестиваль современного искусства...</p>
                    </article>
                </div>
            </section>
        </main>
    </div>
</body>
</html>
```

### Шаблон для технической документации
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Документация API</title>
    <style>
        .docs-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            display: grid;
            grid-template-columns: 250px 1fr;
            gap: 30px;
        }
        
        .sidebar {
            position: sticky;
            top: 20px;
            height: fit-content;
        }
        
        .api-reference {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
        }
        
        .endpoint {
            margin-bottom: 30px;
            padding-bottom: 20px;
            border-bottom: 1px solid #eee;
        }
        
        .code-block {
            background-color: #f5f5f5;
            padding: 15px;
            border-radius: 4px;
            overflow-x: auto;
            font-family: 'Courier New', monospace;
        }
    </style>
</head>
<body>
    <div class="docs-container">
        <aside class="sidebar">
            <nav aria-label="Навигация по документации">
                <h2>Содержание</h2>
                <ul>
                    <li><a href="#introduction">Введение</a></li>
                    <li><a href="#authentication">Аутентификация</a></li>
                    <li><a href="#endpoints">Эндпоинты</a>
                        <ul>
                            <li><a href="#users">Пользователи</a></li>
                            <li><a href="#posts">Посты</a></li>
                            <li><a href="#comments">Комментарии</a></li>
                        </ul>
                    </li>
                    <li><a href="#errors">Обработка ошибок</a></li>
                    <li><a href="#examples">Примеры</a></li>
                </ul>
            </nav>
        </aside>
        
        <main>
            <article class="api-reference" itemscope itemtype="http://schema.org/TechArticle">
                <header>
                    <h1 itemprop="headline">Документация REST API</h1>
                    <p>Версия: <span itemprop="version">v2.1.0</span></p>
                    <p>Последнее обновление:
                        <time datetime="2023-11-08" itemprop="dateModified">8 ноября 2023</time>
                    </p>
                </header>
                
                <section id="introduction" itemprop="articleBody">
                    <h2>Введение</h2>
                    <p itemprop="description">Наше API предоставляет программный доступ к функциям платформы...</p>
                </section>
                
                <section id="authentication" itemprop="articleBody">
                    <h2>Аутентификация</h2>
                    <p>Для доступа к API требуется аутентификация с помощью токена:</p>
                    <div class="code-block">
                        <pre>Authorization: Bearer YOUR_API_TOKEN</pre>
                    </div>
                </section>
                
                <section id="endpoints" itemprop="articleBody">
                    <h2>Эндпоинты</h2>
                    
                    <article class="endpoint" id="users" itemscope itemtype="http://schema.org/APIReference">
                        <h3 itemprop="name">Пользователи</h3>
                        <p itemprop="description">Операции с пользовательскими данными</p>
                        
                        <section>
                            <h4>Получить информацию о пользователе</h4>
                            <div class="code-block">
                                <pre>GET /api/users/{id}</pre>
                            </div>
                            <p>Возвращает информацию о пользователе с указанным ID.</p>
                        </section>
                        
                        <section>
                            <h4>Обновить пользователя</h4>
                            <div class="code-block">
                                <pre>PUT /api/users/{id}</pre>
                            </div>
                            <p>Обновляет информацию о пользователе.</p>
                        </section>
                    </article>
                    
                    <article class="endpoint" id="posts" itemscope itemtype="http://schema.org/APIReference">
                        <h3 itemprop="name">Посты</h3>
                        <p itemprop="description">Операции с постами</p>
                        
                        <section>
                            <h4>Создать пост</h4>
                            <div class="code-block">
                                <pre>POST /api/posts</pre>
                            </div>
                            <p>Создает новый пост.</p>
                        </section>
                    </article>
                </section>
                
                <section id="errors" itemprop="articleBody">
                    <h2>Обработка ошибок</h2>
                    <p>API возвращает стандартные HTTP коды ошибок:</p>
                    <ul>
                        <li>200 - Успешно</li>
                        <li>400 - Неверный запрос</li>
                        <li>401 - Неавторизованный доступ</li>
                        <li>404 - Ресурс не найден</li>
                        <li>500 - Внутренняя ошибка сервера</li>
                    </ul>
                </section>
                
                <section id="examples" itemprop="articleBody">
                    <h2>Примеры использования</h2>
                    
                    <article>
                        <h3>Пример запроса на JavaScript</h3>
                        <div class="code-block">
                            <pre>fetch('https://api.example.com/users/123', {
  method: 'GET',
  headers: {
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));</pre>
                        </div>
                    </article>
                    
                    <article>
                        <h3>Пример запроса на Python</h3>
                        <div class="code-block">
                            <pre>import requests

headers = {
    'Authorization': 'Bearer YOUR_TOKEN',
    'Content-Type': 'application/json'
}

response = requests.get('https://api.example.com/users/123', headers=headers)
data = response.json()
print(data)</pre>
                        </div>
                    </article>
                </section>
                
                <footer>
                    <p>Поддержка: <a href="mailto:api-support@example.com">api-support@example.com</a></p>
                    <p>Документация версии 2.1.0 &copy; 2023</p>
                </footer>
            </article>
        </main>
    </div>
</body>
</html>
```

## Проверка и тестирование семантики

### Инструменты для проверки:
1. **HTML5 Outliner** - проверка иерархии заголовков и структуры
2. **W3C Markup Validator** - проверка соответствия стандартам HTML
3. **axe-core** - проверка доступности, включая семантику
4. **Lighthouse** - комплексная проверка, включая структуру страницы
5. **WAVE** - веб-инструмент для оценки доступности и структуры
6. **Browser developer tools** - проверка DOM структуры

### Проверка правильности использования article и section:
1. Убедитесь, что `<article>` содержит самодостаточный контент
2. Проверьте, что `<section>` используется для тематически связанных разделов
3. Убедитесь, что у каждого `<section>` есть заголовок
4. Проверьте иерархию заголовков
5. Убедитесь, что элементы используются по их прямому назначению, а не для стилизации

## Лучшие практики использования article и section

### 1. Контекстно-зависимое использование
```html
<!-- Хорошо: article для независимого контента -->
<article itemscope itemtype="http://schema.org/BlogPosting">
    <h2 itemprop="headline">Как использовать семантические элементы</h2>
    <p>Полная статья о семантических элементах...</p>
</article>

<!-- Хорошо: section для разделов внутри статьи -->
<article itemscope itemtype="http://schema.org/Article">
    <h1 itemprop="headline">Полное руководство по HTML</h1>
    
    <section>
        <h2>Основы HTML</h2>
        <p>Введение в HTML...</p>
    </section>
    
    <section>
        <h2>Семантические элементы</h2>
        <p>Описание семантических элементов...</p>
    </section>
</article>
```

### 2. Семантика + доступность
```html
<!-- Статья с улучшенной доступностью -->
<article role="article" aria-labelledby="main-article-title">
    <header>
        <h1 id="main-article-title">Заголовок основной статьи</h1>
    </header>
    
    <section role="region" aria-labelledby="intro-section-title">
        <h2 id="intro-section-title">Введение</h2>
        <p>Вступительная часть статьи...</p>
    </section>
    
    <section role="region" aria-labelledby="main-section-title">
        <h2 id="main-section-title">Основная часть</h2>
        <p>Основное содержимое статьи...</p>
    </section>
</article>
```

### 3. SEO оптимизация
```html
<!-- SEO-оптимизированная статья -->
<article itemscope itemtype="http://schema.org/Article">
    <header>
        <h1 itemprop="headline">Полное руководство по семантической разметке HTML</h1>
        <p>Автор: <span itemprop="author">Имя автора</span></p>
        <p>Опубликовано: 
            <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time>
        </p>
    </header>
    
    <section itemprop="articleBody">
        <h2>Что такое семантическая разметка</h2>
        <p itemprop="description">Семантическая разметка - это использование HTML элементов...</p>
    </section>
    
    <section itemprop="articleBody">
        <h2>Преимущества семантической разметки</h2>
        <p>Семантическая разметка улучшает доступность, SEO и поддерживаемость...</p>
    </section>
    
    <footer>
        <p>Категория: <span itemprop="articleSection">Веб-разработка</span></p>
        <p>Теги: 
            <span itemprop="keywords">
                <a href="/tag/html" rel="tag">HTML</a>,
                <a href="/tag/semantics" rel="tag">Семантика</a>,
                <a href="/tag/accessibility" rel="tag">Доступность</a>
            </span>
        </p>
    </footer>
</article>
```

## Заключение

Элементы `<article>` и `<section>` являются важными компонентами современной семантической разметки HTML. Правильное использование этих элементов в сочетании с другими семантическими элементами, структурированными данными и принципами доступности создает веб-страницы, которые лучше понимаются как пользователями, так и поисковыми системами. Понимание различий между этими элементами и их правильное применение является ключевым навыком современного веб-разработчика.

Ключевые моменты для запоминания:
1. Используйте `<article>` для независимого, самодостаточного контента
2. Используйте `<section>` для тематически связанных разделов документа
3. Следите за правильной иерархией заголовков
4. Добавляйте структурированные данные при необходимости
5. Обеспечьте доступность для всех пользователей
6. Используйте элементы по их прямому назначению
7. Сочетайте семантику с современными веб-технологиями
8. Проверяйте правильность разметки с помощью инструментов

Эти практики помогут создавать веб-страницы, которые не только выглядят современно, но и имеют правильную структуру, доступны для всех пользователей и оптимизированы для поисковых систем.

## Следующие темы
- [[HTML/Семантика/Заголовки и навигация]]
- [[HTML/Доступность]]
- [[HTML/SEO/Структура страницы]]
- [[HTML/Семантика/Микроформаты]]

## Теги
#html #semantics #article #section #web-development #accessibility #seo #structure #html5 #structured-data #microdata #web-components #indexeddb #websockets #best-practices
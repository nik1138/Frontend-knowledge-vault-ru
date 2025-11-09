# HTML Элементы: Блочные элементы

Блочные элементы в HTML занимают всю доступную ширину родительского элемента и всегда начинаются с новой строки. Они формируют основу структуры веб-страницы и играют важную роль в семантической разметке.

## Основные характеристики блочных элементов

1. **Занимают всю ширину** - блочные элементы расширяются до ширины родительского контейнера
2. **Начинаются с новой строки** - каждый блочный элемент начинается на новой строке
3. **Могут содержать другие блочные элементы** - большинство блочных элементов могут быть вложены друг в друга
4. **Могут содержать строчные элементы** - блочные элементы могут содержать строчные элементы внутри себя

## Основные блочные элементы

### `<div>` - Универсальный блочный контейнер

Элемент `<div>` является универсальным контейнером без специального значения. Используется для группировки элементов для стилизации или других целей.

```html
<div class="container">
    <h2>Заголовок внутри контейнера</h2>
    <p>Текст внутри контейнера</p>
</div>
```

### `<p>` - Абзац

Элемент `<p>` представляет собой абзац текста. Это один из самых часто используемых элементов в HTML.

```html
<p>Это первый абзац текста. Он содержит основную информацию, которую нужно передать пользователю.</p>
<p>Это второй абзац. Каждый абзац автоматически начинается с новой строки.</p>
```

### Заголовки (`<h1>` - `<h6>`)

Заголовки используются для структурирования содержимого документа. У них есть важное семантическое значение.

```html
<h1>Основной заголовок страницы</h1>
<h2>Подзаголовок первого уровня</h2>
<p>Содержимое подзаголовка...</p>
<h3>Подзаголовок второго уровня</h3>
<p>Еще более конкретное содержимое...</p>
```

### `<section>` - Секция документа

Элемент `<section>` представляет собой тематически связанную группу содержимого, обычно с заголовком.

```html
<section>
    <h2>О компании</h2>
    <p>Информация о компании и её деятельности...</p>
</section>

<section>
    <h2>Наши услуги</h2>
    <p>Описание предоставляемых услуг...</p>
</section>
```

### `<article>` - Независимый контент

Элемент `<article>` представляет собой независимый, автономный фрагмент содержимого, который может быть распространён независимо.

```html
<article>
    <h2>Новость дня</h2>
    <p>Содержимое новости...</p>
    <footer>
        <p>Опубликовано: 15.03.2023</p>
    </footer>
</article>
```

### `<header>` - Заголовок документа или раздела

Элемент `<header>` содержит вступительное содержимое или набор навигационных элементов.

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

### `<footer>` - Подвал документа или раздела

Элемент `<footer>` содержит информацию о нижнем колонтитуле раздела или документа.

```html
<footer>
    <p>&copy; 2023 Все права защищены</p>
    <address>
        Контактная информация: <a href="mailto:info@example.com">info@example.com</a>
    </address>
</footer>
```

### `<nav>` - Навигационный блок

Элемент `<nav>` представляет собой раздел, содержащий навигационные ссылки.

```html
<nav>
    <ul>
        <li><a href="/">Главная</a></li>
        <li><a href="/products">Продукты</a></li>
        <li><a href="/services">Услуги</a></li>
        <li><a href="/contact">Контакты</a></li>
    </ul>
</nav>
```

### `<main>` - Основное содержимое

Элемент `<main>` представляет основное содержимое документа или приложения.

```html
<main>
    <article>
        <h1>Основная статья</h1>
        <p>Содержимое основной статьи...</p>
    </article>
</main>
```

### `<aside>` - Сопутствующий контент

Элемент `<aside>` содержит контент, косвенно связанный с основным содержимым.

```html
<article>
    <h2>Основная статья</h2>
    <p>Основное содержимое статьи...</p>
    
    <aside>
        <h3>Похожие статьи</h3>
        <ul>
            <li><a href="#">Статья 1</a></li>
            <li><a href="#">Статья 2</a></li>
        </ul>
    </aside>
</article>
```

## Семантические блочные элементы

### `<address>` - Контактная информация

Элемент `<address>` используется для указания контактной информации.

```html
<address>
    Автор: <a href="mailto:john@example.com">Иван Иванов</a><br>
    Телефон: +7 (123) 456-78-90<br>
    Адрес: г. Москва, ул. Примерная, д. 1
</address>
```

### `<blockquote>` - Цитата

Элемент `<blockquote>` представляет собой цитату из другого источника.

```html
<blockquote cite="https://example.com/source">
    <p>Это пример цитаты из другого источника.</p>
    <footer>— <cite>Автор цитаты</cite></footer>
</blockquote>
```

### `<figure>` и `<figcaption>` - Иллюстрация с подписью

Эти элементы используются для представления иллюстраций с подписями.

```html
<figure>
    <img src="chart.png" alt="Диаграмма продаж">
    <figcaption>Диаграмма продаж за последний квартал</figcaption>
</figure>
```

## Стилизация блочных элементов

Блочные элементы могут быть стилизованы с помощью CSS:

```html
<style>
.highlight-section {
    background-color: #f0f8ff;
    padding: 20px;
    border-left: 4px solid #007acc;
    margin: 20px 0;
}

.article-preview {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 15px;
    margin-bottom: 20px;
}
</style>

<section class="highlight-section">
    <h2>Важная информация</h2>
    <p>Этот раздел выделен стилями для привлечения внимания.</p>
</section>

<article class="article-preview">
    <h3>Предварительный просмотр статьи</h3>
    <p>Краткое содержание статьи...</p>
</article>
```

## Распространенные ошибки

### 1. Неправильное вложение строчных элементов

```html
<!-- Неправильно -->
<span>
    <div>Блочный элемент внутри строчного</div>
</span>

<!-- Правильно -->
<div>
    <span>Строчный элемент внутри блочного</span>
</div>
```

### 2. Использование `<div>` вместо семантических элементов

```html
<!-- Не рекомендуется -->
<div class="header">Заголовок</div>
<div class="nav">Навигация</div>
<div class="article">Статья</div>

<!-- Рекомендуется -->
<header>Заголовок</header>
<nav>Навигация</nav>
<article>Статья</article>
```

## Заключение

Блочные элементы являются основой структуры HTML документа. Понимание их использования и семантики помогает создавать более доступные, SEO-оптимизированные и поддерживаемые веб-страницы. Использование семантически правильных элементов вместо универсального `<div>` делает код более понятным и функциональным.

## Современные семантические блочные элементы HTML5

### Дополнительные семантические элементы
```html
<!-- Основные контейнеры -->
<main>Основное содержимое страницы</main>
<section>Тематически связанная группа содержимого</section>
<article>Независимый, автономный фрагмент содержимого</article>
<aside>Содержимое, косвенно связанное с основным содержимым</aside>

<!-- Заголовки и навигация -->
<header>Заголовок документа или раздела</header>
<nav>Раздел, содержащий навигационные ссылки</nav>
<footer>Подвал документа или раздела</footer>

<!-- Группировка содержимого -->
<figure>Иллюстрация с подписью</figure>
<figcaption>Подпись к иллюстрации</figcaption>
<blockquote>Цитата из другого источника</blockquote>
<details>Раскрывающийся раздел</details>
<summary>Заголовок для details</summary>
```

### Семантические элементы для группировки
```html
<!-- Адрес -->
<address>
    Контактная информация:<br>
    Email: <a href="mailto:info@example.com">info@example.com</a><br>
    Телефон: +7 (495) 123-45-67
</address>

<!-- Диалог -->
<dialog id="modal-dialog">
    <h2>Заголовок диалога</h2>
    <p>Содержимое диалогового окна</p>
    <button onclick="document.getElementById('modal-dialog').close()">Закрыть</button>
</dialog>

<!-- Списки -->
<ul>Немаркированный список</ul>
<ol>Маркированный список</ol>
<dl>Список определений</dl>
```

## Расширенные примеры использования блочных элементов

### Комплексная структура статьи
```html
<article itemscope itemtype="http://schema.org/Article">
    <header>
        <h1 itemprop="headline">Заголовок статьи</h1>
        <p>Автор: <span itemprop="author">Имя автора</span></p>
        <p>Опубликовано: 
            <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time>
        </p>
    </header>

    <section itemprop="articleBody">
        <h2>Введение</h2>
        <p>Вступительный абзац статьи...</p>
    </section>

    <section itemprop="articleBody">
        <h2>Основная часть</h2>
        
        <section>
            <h3>Подраздел основной части</h3>
            <p>Содержимое подраздела...</p>
            
            <figure>
                <img src="image.jpg" alt="Описание изображения" itemprop="image">
                <figcaption>Подпись к изображению</figcaption>
            </figure>
        </section>

        <section>
            <h3>Другой подраздел</h3>
            <p>Содержимое другого подраздела...</p>
            
            <blockquote cite="https://example.com/source" itemprop="comment">
                <p>Цитата из другого источника...</p>
                <footer>— <cite>Автор цитаты</cite></footer>
            </blockquote>
        </section>
    </section>

    <aside role="complementary" aria-label="Сопутствующая информация">
        <h3>Связанные темы</h3>
        <ul>
            <li><a href="/related-topic1">Связанная тема 1</a></li>
            <li><a href="/related-topic2">Связанная тема 2</a></li>
        </ul>
    </aside>

    <footer>
        <p>Теги: 
            <span itemprop="keywords">
                <a href="/tag/html">HTML</a>, 
                <a href="/tag/semantics">Семантика</a>
            </span>
        </p>
        <p>Обновлено: 
            <time datetime="2023-11-09" itemprop="dateModified">9 ноября 2023</time>
        </p>
    </footer>
</article>
```

### Структура веб-сайта
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Структура веб-сайта</title>
</head>
<body>
    <header role="banner">
        <h1>Название сайта</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/services">Услуги</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>

    <main role="main">
        <section>
            <h2>Основной контент</h2>
            <article>
                <h3>Статья 1</h3>
                <p>Содержимое статьи...</p>
            </article>
            <article>
                <h3>Статья 2</h3>
                <p>Содержимое второй статьи...</p>
            </article>
        </section>

        <aside role="complementary" aria-label="Боковая панель">
            <section>
                <h3>Популярные статьи</h3>
                <ul>
                    <li><a href="/popular1">Популярная статья 1</a></li>
                    <li><a href="/popular2">Популярная статья 2</a></li>
                </ul>
            </section>
            <section>
                <h3>Реклама</h3>
                <p>Рекламный контент</p>
            </section>
        </aside>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Все права защищены</p>
        <nav aria-label="Нижняя навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/terms">Условия использования</a></li>
            </ul>
        </nav>
    </footer>
</body>
</html>
```

## Доступность блочных элементов

### Использование ARIA-ролей с блочными элементами
```html
<!-- Основные разделы с ARIA ролями -->
<header role="banner">Шапка сайта</header>
<nav role="navigation" aria-label="Основная навигация">Навигация</nav>
<main role="main" id="main-content">Основное содержимое</main>
<aside role="complementary">Боковая панель</aside>
<footer role="contentinfo">Подвал сайта</footer>

<!-- Структура с дополнительной семантикой -->
<section role="region" aria-labelledby="section-title">
    <h2 id="section-title">Название раздела</h2>
    <p>Содержимое раздела с дополнительной семантикой</p>
</section>
```

### Логические группы с fieldset и legend
```html
<form>
    <fieldset>
        <legend>Личная информация</legend>
        <label for="name">Имя:</label>
        <input type="text" id="name" name="name">
        
        <label for="email">Email:</label>
        <input type="email" id="email" name="email">
    </fieldset>

    <fieldset>
        <legend>Настройки</legend>
        <label>
            <input type="checkbox" name="newsletter"> Подписка на рассылку
        </label>
        <label>
            <input type="checkbox" name="notifications"> Уведомления
        </label>
    </fieldset>
</form>
```

## Стилизация блочных элементов

### CSS Grid с блочными элементами
```html
<style>
.grid-container {
    display: grid;
    grid-template-columns: 2fr 1fr;
    gap: 20px;
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

.main-content {
    grid-column: 1;
}

.sidebar {
    grid-column: 2;
}

.article-card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 20px;
    margin-bottom: 20px;
}
</style>

<div class="grid-container">
    <main class="main-content">
        <article class="article-card">
            <h2>Заголовок статьи</h2>
            <p>Содержимое статьи...</p>
        </article>
        <article class="article-card">
            <h2>Заголовок второй статьи</h2>
            <p>Содержимое второй статьи...</p>
        </article>
    </main>
    <aside class="sidebar">
        <section>
            <h3>Боковая панель</h3>
            <p>Дополнительная информация</p>
        </section>
    </aside>
</div>
```

### Flexbox с блочными элементами
```html
<style>
.flex-container {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
}

.header {
    flex: 0 0 auto;
    background-color: #333;
    color: white;
    padding: 1rem;
}

.content {
    flex: 1 0 auto;
    padding: 2rem;
}

.footer {
    flex: 0 0 auto;
    background-color: #f0f0f0;
    padding: 1rem;
    text-align: center;
}
</style>

<div class="flex-container">
    <header class="header">
        <h1>Заголовок сайта</h1>
    </header>
    
    <main class="content">
        <section>
            <h2>Основное содержимое</h2>
            <p>Текст основного содержимого...</p>
        </section>
    </main>
    
    <footer class="footer">
        <p>&copy; 2023 Все права защищены</p>
    </footer>
</div>
```

## Современные паттерны использования блочных элементов

### Карточки с article
```html
<main>
    <article class="card">
        <header class="card-header">
            <h2>Заголовок карточки</h2>
        </header>
        <div class="card-content">
            <p>Содержимое карточки...</p>
        </div>
        <footer class="card-footer">
            <time datetime="2023-11-08">8 ноября 2023</time>
        </footer>
    </article>

    <article class="card">
        <header class="card-header">
            <h2>Вторая карточка</h2>
        </header>
        <div class="card-content">
            <p>Содержимое второй карточки...</p>
        </div>
        <footer class="card-footer">
            <time datetime="2023-11-07">7 ноября 2023</time>
        </footer>
    </article>
</main>
```

### Аккордеон с details и summary
```html
<details>
    <summary>Часто задаваемые вопросы</summary>
    <div class="faq-content">
        <details>
            <summary>Как создать аккаунт?</summary>
            <p>Чтобы создать аккаунт, перейдите на страницу регистрации...</p>
        </details>
        <details>
            <summary>Как сбросить пароль?</summary>
            <p>Для сброса пароля нажмите на ссылку "Забыли пароль"...</p>
        </details>
    </div>
</details>
```

### Структурированные данные с блочными элементами
```html
<article itemscope itemtype="http://schema.org/BlogPosting">
    <header>
        <h1 itemprop="headline">Современные блочные элементы HTML</h1>
        <div itemprop="author" itemscope itemtype="http://schema.org/Person">
            <span itemprop="name">Имя автора</span>
        </div>
        <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time>
    </header>

    <div itemprop="articleBody">
        <section>
            <h2>Введение</h2>
            <p>HTML5 предоставляет множество семантических блочных элементов...</p>
        </section>

        <section>
            <h2>Практическое применение</h2>
            <p>Рассмотрим, как использовать эти элементы на практике...</p>
        </section>
    </div>

    <footer>
        <div itemprop="publisher" itemscope itemtype="http://schema.org/Organization">
            <span itemprop="name">Название организации</span>
        </div>
    </footer>
</article>
```

## Лучшие практики использования блочных элементов

### 1. Правильная семантика
```html
<!-- Хорошо: семантически правильное использование -->
<article>
    <h2>Независимая статья</h2>
    <p>Содержимое статьи...</p>
</article>

<section>
    <h2>Связанная группа контента</h2>
    <p>Содержимое раздела...</p>
</section>

<!-- Плохо: использование div для всего -->
<div>
    <div>Статья</div>
    <div>Содержимое статьи...</div>
</div>

<div>
    <div>Раздел</div>
    <div>Содержимое раздела...</div>
</div>
```

### 2. Доступность и SEO
```html
<!-- Семантически, доступно и SEO-оптимизировано -->
<main role="main">
    <article itemscope itemtype="http://schema.org/Article">
        <header>
            <h1 itemprop="headline">Заголовок статьи</h1>
        </header>
        <section aria-labelledby="intro-title">
            <h2 id="intro-title">Введение</h2>
            <p itemprop="description">Вступительный текст...</p>
        </section>
    </article>
</main>
```

### 3. Структура документа
```html
<!DOCTYPE html>
<html lang="ru">
<head>...</head>
<body>
    <header>...</header>
    
    <nav>...</nav>
    
    <main>
        <section>
            <h1>Основной заголовок</h1>
            <article>...</article>
            <article>...</article>
        </section>
        
        <aside>...</aside>
    </main>
    
    <footer>...</footer>
</body>
</html>
```

## Пример комплексной страницы с блочными элементами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Комплексное использование блочных элементов</title>
    <meta name="description" content="Пример использования различных блочных элементов HTML">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Комплексное использование блочных элементов">
    <meta property="og:description" content="Пример использования различных блочных элементов HTML">
    <meta property="og:type" content="article">
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
        <section>
            <h2>Популярные статьи</h2>
            
            <article itemscope itemtype="http://schema.org/Article">
                <header>
                    <h3 itemprop="headline">
                        <a href="/article1" itemprop="url">Современные блочные элементы HTML</a>
                    </h3>
                    <p>Автор: <span itemprop="author">Имя автора</span></p>
                    <time datetime="2023-11-08" itemprop="datePublished">8 ноября 2023</time>
                </header>
                
                <div itemprop="articleBody">
                    <p itemprop="description">В этой статье мы рассмотрим современные блочные элементы HTML5...</p>
                </div>
                
                <footer>
                    <p>Теги: 
                        <span itemprop="keywords">
                            <a href="/tag/html" rel="tag">HTML</a>, 
                            <a href="/tag/semantics" rel="tag">Семантика</a>
                        </span>
                    </p>
                </footer>
            </article>

            <article itemscope itemtype="http://schema.org/Article">
                <header>
                    <h3 itemprop="headline">
                        <a href="/article2" itemprop="url">Практическое применение семантики</a>
                    </h3>
                    <p>Автор: <span itemprop="author">Другой автор</span></p>
                    <time datetime="2023-11-07" itemprop="datePublished">7 ноября 2023</time>
                </header>
                
                <div itemprop="articleBody">
                    <p itemprop="description">Практические примеры использования семантических элементов...</p>
                </div>
                
                <footer>
                    <p>Теги: 
                        <span itemprop="keywords">
                            <a href="/tag/semantics" rel="tag">Семантика</a>, 
                            <a href="/tag/accessibility" rel="tag">Доступность</a>
                        </span>
                    </p>
                </footer>
            </article>
        </section>

        <aside role="complementary" aria-label="Сопутствующий контент">
            <section>
                <h3>Категории</h3>
                <nav aria-label="Навигация по категориям">
                    <ul>
                        <li><a href="/category/html">HTML</a></li>
                        <li><a href="/category/css">CSS</a></li>
                        <li><a href="/category/javascript">JavaScript</a></li>
                    </ul>
                </nav>
            </section>
            
            <section>
                <h3>Архив</h3>
                <nav aria-label="Навигация по архиву">
                    <ul>
                        <li><a href="/archive/2023/11">Ноябрь 2023</a></li>
                        <li><a href="/archive/2023/10">Октябрь 2023</a></li>
                        <li><a href="/archive/2023/09">Сентябрь 2023</a></li>
                    </ul>
                </nav>
            </section>
        </aside>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Название компании. Все права защищены.</p>
        
        <address>
            Контактная информация: 
            <a href="mailto:info@example.com">info@example.com</a>
        </address>
        
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

Современные блочные элементы HTML предоставляют мощные возможности для создания семантически правильной, доступной и SEO-оптимизированной разметки. Понимание различий между элементами и правильное их использование позволяет создавать веб-страницы, которые лучше воспринимаются как пользователями, так и поисковыми системами. Важно использовать элементы по их прямому назначению, учитывать доступность и современные веб-стандарты.

## Следующие темы
- [[HTML/Элементы/Строчные элементы]]
- [[HTML/Семантика]]
- [[HTML/Доступность]]
- [[HTML/Формы]]

## Теги
#html #elements #block-elements #semantics #web-development #frontend #structure #html5 #accessibility #seo
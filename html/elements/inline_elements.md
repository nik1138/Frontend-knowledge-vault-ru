# HTML Элементы: Строчные элементы

Строчные элементы в HTML занимают только необходимое пространство и не начинаются с новой строки. Они используются для стилизации части текста или представления элементов внутри потока текста.

## Основные характеристики строчных элементов

1. **Занимают только необходимую ширину** - строчные элементы расширяются только настолько, насколько необходимо для отображения их содержимого
2. **Не начинаются с новой строки** - строчные элементы размещаются на той же строке, что и соседние элементы
3. **Могут содержать другие строчные элементы** - строчные элементы могут быть вложены друг в друга
4. **Не могут содержать блочные элементы** - строчные элементы не могут содержать блочные элементы внутри себя

## Основные строчные элементы

### `<span>` - Универсальный строчный контейнер

Элемент `<span>` является универсальным строчным контейнером без специального значения. Используется для группировки элементов для стилизации или других целей.

```html
<p>Это <span class="highlight">выделенное</span> слово в предложении.</p>
<p>Цена: <span class="price">$99.99</span></p>
```

### `<a>` - Ссылка

Элемент `<a>` создает гиперссылки на другие страницы или ресурсы.

```html
<a href="https://example.com">Ссылка на внешний сайт</a>
<a href="#section1">Ссылка на раздел на этой странице</a>
<a href="mailto:info@example.com">Отправить email</a>
<a href="tel:+71234567890">Позвонить</a>
```

### `<strong>` и `<b>` - Жирный текст

Элемент `<strong>` указывает на важность содержимого, а `<b>` просто делает текст жирным без семантического значения.

```html
<p>Это <strong>очень важно</strong> для понимания контекста.</p>
<p>Это просто <b>жирный текст</b> для выделения.</p>
```

### `<em>` и `<i>` - Курсивный текст

Элемент `<em>` указывает на акцентированное произношение, а `<i>` используется для выделения, таких как названия книг или термины.

```html
<p>Я <em>действительно</em> уверен в этом решении.</p>
<p>Прочитайте книгу <i>Война и мир</i> для лучшего понимания.</p>
```

### `<img>` - Изображение

Элемент `<img>` вставляет изображение в документ.

```html
<img src="photo.jpg" alt="Описание изображения" width="300" height="200">
<img src="logo.png" alt="Логотип компании" class="logo">
```

### `<code>` - Фрагмент кода

Элемент `<code>` представляет собой фрагмент компьютерного кода.

```html
<p>Используйте команду <code>npm install</code> для установки зависимостей.</p>
<code>function hello() { return "Привет, мир!"; }</code>
```

### `<time>` - Время и дата

Элемент `<time>` представляет собой время и/или дату.

```html
<p>Событие состоится <time datetime="2023-03-15">15 марта 2023 года</time>.</p>
<p>Встреча в <time datetime="14:30">14:30</time>.</p>
```

### `<small>` - Второстепенный текст

Элемент `<small>` представляет собой второстепенный текст, такой как авторские права или юридические уведомления.

```html
<p>Основной текст статьи.</p>
<small>© 2023 Все права защищены</small>
```

### `<sub>` и `<sup>` - Подстрочный и надстрочный текст

Элементы `<sub>` и `<sup>` используются для подстрочного и надстрочного текста соответственно.

```html
<p>Формула воды: H<sub>2</sub>O</p>
<p>Возвести в квадрат: x<sup>2</sup></p>
```

## Интерактивные строчные элементы

### `<button>` - Кнопка

Элемент `<button>` создает интерактивную кнопку.

```html
<button type="submit">Отправить форму</button>
<button onclick="alert('Кнопка нажата!')">Нажми меня</button>
<button disabled>Неактивная кнопка</button>
```

### `<input>` - Поле ввода

Элемент `<input>` создает поле ввода различных типов.

```html
<input type="text" placeholder="Введите имя">
<input type="email" placeholder="email@example.com">
<input type="checkbox" id="agree">
<label for="agree">Согласен с условиями</label>
```

### `<select>` и `<option>` - Выпадающий список

Эти элементы создают выпадающий список выбора.

```html
<label for="country">Страна:</label>
<select id="country" name="country">
    <option value="ru">Россия</option>
    <option value="us">США</option>
    <option value="de">Германия</option>
</select>
```

## Текстовые строчные элементы

### `<mark>` - Выделенный текст

Элемент `<mark>` выделяет текст для дальнейшего обращения.

```html
<p>В поисковом запросе "HTML" найдено: это <mark>HTML</mark> тег.</p>
```

### `<kbd>` - Ввод с клавиатуры

Элемент `<kbd>` представляет собой ввод с клавиатуры или другой текст ввода.

```html
<p>Для сохранения используйте комбинацию <kbd>Ctrl</kbd> + <kbd>S</kbd>.</p>
```

### `<samp>` - Вывод программы

Элемент `<samp>` представляет собой вывод программы или компьютерной системы.

```html
<p>Ошибка: <samp>File not found</samp></p>
```

### `<var>` - Переменная

Элемент `<var>` представляет собой переменную в математическом выражении или программе.

```html
<p>В уравнении y = mx + b, <var>x</var> и <var>y</var> являются переменными.</p>
```

## Встраивание медиа

### `<audio>` и `<video>` - Аудио и видео

Эти элементы используются для встраивания аудио и видео контента.

```html
<audio controls>
    <source src="audio.mp3" type="audio/mpeg">
    <source src="audio.ogg" type="audio/ogg">
    Ваш браузер не поддерживает аудио элемент.
</audio>

<video width="320" height="240" controls>
    <source src="movie.mp4" type="video/mp4">
    <source src="movie.ogg" type="video/ogg">
    Ваш браузер не поддерживает видео элемент.
</video>
```

### `<iframe>` - Встроенная рамка

Элемент `<iframe>` встраивает другую HTML страницу.

```html
<iframe src="https://example.com" width="600" height="400" title="Внешний контент"></iframe>
```

## Стилизация строчных элементов

Строчные элементы можно стилизовать с помощью CSS:

```html
<style>
.highlight {
    background-color: yellow;
    padding: 2px 4px;
}

.price {
    color: #007acc;
    font-weight: bold;
}

.logo {
    vertical-align: middle;
    margin-right: 10px;
}
</style>

<p>Это <span class="highlight">выделенное</span> слово и цена <span class="price">$99.99</span>.</p>
<img src="logo.png" alt="Логотип" class="logo">Компания
```

## Распространенные ошибки

### 1. Вложение блочных элементов в строчные

```html
<!-- Неправильно -->
<a href="#">
    <div>Блочный элемент внутри ссылки</div>
</a>

<!-- Правильно -->
<div>
    <a href="#">Ссылка внутри блока</a>
</div>
```

### 2. Неправильное использование семантики

```html
<!-- Не рекомендуется - потеря семантики -->
<span class="strong">Важный текст</span>
<span class="em">Акцентированный текст</span>

<!-- Рекомендуется - правильная семантика -->
<strong>Важный текст</strong>
<em>Акцентированный текст</em>
```

## Взаимодействие со строчными и блочными элементами

Строчные элементы могут свободно перемещаться внутри блочных:

```html
<div class="container">
    <p>Это абзац с <strong>жирным</strong> и <em>курсивным</em> текстом.</p>
    <p>Цена: <span class="price">$99.99</span> <small>включая налог</small>.</p>
</div>
```

## Заключение

Строчные элементы играют важную роль в HTML, позволяя форматировать и структурировать текст внутри блочных элементов. Правильное использование строчных элементов с учетом их семантики делает HTML код более понятным, доступным и SEO-оптимизированным. Понимание различий между строчными и блочными элементами помогает создавать правильно структурированные веб-страницы.

## Современные строчные элементы HTML5

### Текстовые элементы форматирования
```html
<!-- Семантическое выделение -->
<strong>Сильное выделение (важность)</strong>
<em>Акцентированное выделение</em>
<mark>Выделенный текст для дальнейшего обращения</mark>

<!-- Специфические типы текста -->
<code>Фрагмент кода</code>
<kbd>Ввод с клавиатуры</kbd>
<samp>Вывод программы</samp>
<var>Переменная в математическом выражении</var>
<s>Устаревший или неверный текст</s>
<del>Удаленный текст</del>
<ins>Вставленный текст</ins>

<!-- Тонкое форматирование -->
<small>Второстепенный текст (например, авторские права)</small>
<sub>Подстрочный текст (например, H<sub>2</sub>O)</sub>
<sup>Надстрочный текст (например, x<sup>2</sup>)</sup>
<cite>Название произведения (книги, статьи и т.д.)</cite>
<dfn>Определение термина</dfn>
<abbr title="Аббревиатура">Абр.</abbr>
<time datetime="2023-11-08T10:00:00+00:00">8 ноября 2023</time>
```

### Элементы для представления данных
```html
<!-- Код и компьютерный текст -->
<code>inline code</code>
<pre><code>Блок кода
    с сохранением форматирования</code></pre>
<samp>Вывод программы</samp>
<kbd>Ctrl + C</kbd>
<var>переменная</var>

<!-- Цитирование и ссылки -->
<q>Короткая цитата</q>
<blockquote cite="https://example.com/source">
    <p>Длинная цитата, которая занимает несколько строк и может содержать абзацы.</p>
</blockquote>
<cite>Название источника</cite>

<!-- Библиографические элементы -->
<address>
    Автор: <a href="mailto:author@example.com">Имя автора</a>
</address>
```

## Расширенные примеры использования строчных элементов

### Комплексное форматирование текста
```html
<p>В современном веб-дизайне <strong>семантическая разметка</strong> играет ключевую роль. 
Как объясняет специалист в области доступности <cite>Джон Смит</cite>: 
<q>Правильное использование HTML элементов делает веб-контент доступным для всех пользователей</q>. 
Это особенно важно при разработке <mark>доступных интерфейсов</mark>, 
где каждый элемент должен выполнять свою <em>семантическую функцию</em>.</p>

<p>В математическом выражении <var>x</var> = <var>y</var> + <var>z</var>, 
переменные обозначаются тегом <code>&lt;var&gt;</code>, а при вычислениях 
используется комбинация <kbd>Ctrl</kbd> + <kbd>C</kbd> для копирования значений.</p>

<p>Формула воды: H<sub>2</sub>O, а площадь круга: A = πr<sup>2</sup>.</p>
```

### Использование с микроформатами и структурированными данными
```html
<article>
    <h1>Рецензия на книгу</h1>
    
    <p>Автор: <span class="author">Имя автора</span></p>
    <p>Опубликовано: <time datetime="2023-11-08" class="dt-published">8 ноября 2023</time></p>
    
    <p>В своей новой книге <cite class="p-name">"Название книги"</cite> автор 
    <span class="p-author">Имя автора</span> рассматривает тему 
    <span class="p-category">веб-доступности</span> с новой перспективы.</p>
    
    <p>Ключевая цитата: <q class="e-content">Семантическая разметка — это фундамент доступного веба</q></p>
</article>

<!-- Структурированные данные JSON-LD -->
<script type="application/ld+json">
{
    "@context": "http://schema.org",
    "@type": "Review",
    "itemReviewed": {
        "@type": "Book",
        "name": "Название книги"
    },
    "author": {
        "@type": "Person",
        "name": "Имя автора"
    },
    "datePublished": "2023-11-08",
    "reviewBody": "Семантическая разметка — это фундамент доступного веба"
}
</script>
```

### Использование в формах и интерактивных элементах
```html
<form>
    <label for="username">Имя пользователя:</label>
    <input type="text" id="username" name="username" required>
    <small class="help-text">Имя пользователя должно содержать от <strong>3 до 20</strong> символов</small>
    
    <label for="password">Пароль:</label>
    <input type="password" id="password" name="password" required>
    <small class="help-text">Пароль должен содержать хотя бы одну <mark>заглавную букву</mark> и <mark>цифру</mark></small>
    
    <label>
        <input type="checkbox" name="terms" required> 
        Я согласен с <a href="/terms">условиями использования</a> и 
        <a href="/privacy">политикой конфиденциальности</a>
    </label>
    
    <button type="submit">Зарегистрироваться</button>
</form>
```

## Доступность строчных элементов

### Правильное использование семантических элементов для доступности
```html
<!-- Правильное выделение важности -->
<p>Вам нужно <strong>срочно</strong> заполнить эту форму до 23:59.</p>

<!-- Правильное акцентирование -->
<p>Я <em>настаиваю</em> на том, чтобы мы рассмотрели этот вариант.</p>

<!-- Описание изображений -->
<figure>
    <img src="chart.png" alt="График роста продаж за последние 6 месяцев">
    <figcaption>Рост продаж: <mark>наибольший прирост</mark> зафиксирован в октябре</figcaption>
</figure>

<!-- Код и технические термины -->
<p>Для решения проблемы используйте команду <code>npm install package-name</code>, 
где <var>package-name</var> — это имя устанавливаемого пакета.</p>
```

### ARIA-атрибуты с строчными элементов
```html
<!-- Дополнительная информация для вспомогательных технологий -->
<span aria-label="Кнопка закрытия">✕</span>
<span aria-describedby="help-text">?</span>
<div id="help-text" class="sr-only">Нажмите для получения справки</div>

<!-- Состояния и роли -->
<span role="button" tabindex="0" aria-pressed="false">Переключатель</span>
<time datetime="2023-11-08T10:00:00+00:00" aria-label="8 ноября 2023 года, 10:00">сегодня</time>
```

## Стилизация строчных элементов

### CSS для строчных элементов
```html
<style>
/* Стилизация семантических элементов */
strong {
    font-weight: bold;
    color: #d32f2f;
}

em {
    font-style: italic;
    color: #1976d2;
}

mark {
    background-color: #ffeb3b;
    padding: 2px 4px;
    border-radius: 2px;
}

code {
    font-family: 'Courier New', monospace;
    background-color: #f5f5f5;
    padding: 2px 4px;
    border-radius: 3px;
    font-size: 0.9em;
}

kbd {
    display: inline-block;
    padding: 3px 6px;
    font-family: Arial, sans-serif;
    font-size: 0.8em;
    background-color: #f0f0f0;
    border: 1px solid #ccc;
    border-radius: 3px;
    box-shadow: 0 1px 0 rgba(0,0,0,0.2);
}

/* Стилизация для доступности */
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

/* Стили для цитат */
q {
    font-style: italic;
    quotes: "«" "»";
}

q:before {
    content: open-quote;
}

q:after {
    content: close-quote;
}
</style>

<p>Вот пример <strong>сильного выделения</strong>, <em>акцентированного текста</em>, 
и <mark>выделенного фрагмента</mark> в абзаце. Также можно использовать 
<code>инлайн-код</code> и комбинацию клавиш <kbd>Ctrl + C</kbd>.</p>
```

### Интерактивные строчные элементы
```html
<style>
.clickable-span {
    cursor: pointer;
    border-bottom: 1px dotted #1976d2;
    color: #1976d2;
    text-decoration: none;
}

.clickable-span:hover {
    color: #1565c0;
    border-bottom-style: solid;
}

.tooltip {
    position: relative;
    display: inline-block;
    border-bottom: 1px dotted #666;
    cursor: help;
}

.tooltip .tooltip-text {
    visibility: hidden;
    width: 200px;
    background-color: #333;
    color: #fff;
    text-align: center;
    border-radius: 6px;
    padding: 5px;
    position: absolute;
    z-index: 1;
    bottom: 125%;
    left: 50%;
    margin-left: -100px;
    opacity: 0;
    transition: opacity 0.3s;
}

.tooltip:hover .tooltip-text {
    visibility: visible;
    opacity: 1;
}
</style>

<p>Наведите на <span class="tooltip">этот текст<span class="tooltip-text">Всплывающая подсказка</span></span> 
для просмотра подсказки. Также можно <span class="clickable-span" onclick="alert('Клик!')">кликнуть</span> 
на этот фрагмент.</p>
```

## Современные паттерны использования строчных элементов

### Микроразметка с строчными элементами
```html
<div itemscope itemtype="http://schema.org/Person">
    <h2 itemprop="name">Иван Иванов</h2>
    <p>Профессия: <span itemprop="jobTitle">Веб-разработчик</span></p>
    <p>Email: <a href="mailto:ivan@example.com" itemprop="email">ivan@example.com</a></p>
    <p>Телефон: <a href="tel:+74951234567" itemprop="telephone">+7 (495) 123-45-67</a></p>
    <p>Адрес: <span itemprop="address" itemscope itemtype="http://schema.org/PostalAddress">
        <span itemprop="addressLocality">Москва</span>, 
        <span itemprop="addressCountry">Россия</span>
    </span></p>
</div>
```

### Использование в многоязычных текстах
```html
<p>Оригинальное название: <span lang="en">Web Development Guide</span></p>
<p>В японском языке это будет <span lang="ja">ウェブ開発ガイド</span></p>
<p>Перевод на французский: <span lang="fr">Guide de développement web</span></p>

<blockquote lang="de" cite="https://example.com/german-quote">
    <p>Die Zugänglichkeit ist die Grundlage für eine inklusive Gesellschaft</p>
    <footer>— <cite>Немецкий активист</cite></footer>
</blockquote>
```

### Использование с веб-компонентами
```html
<script>
class CodeInline extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        const code = this.getAttribute('code');
        const lang = this.getAttribute('lang') || 'javascript';
        
        this.shadowRoot.innerHTML = `
            <style>
                code {
                    font-family: 'Courier New', monospace;
                    background-color: #f5f5f5;
                    padding: 2px 6px;
                    border-radius: 4px;
                    font-size: 0.9em;
                    color: #d32f2f;
                }
            </style>
            <code>${code}</code>
        `;
    }
}

customElements.define('code-inline', CodeInline);
</script>

<p>Используйте тег <code-inline code="&lt;div&gt;"></code-inline> для создания блоков.</p>
```

## Лучшие практики использования строчных элементов

### 1. Семантическая правильность
```html
<!-- Хорошо: правильное использование семантики -->
<p>Важно использовать <strong>правильные теги</strong> для структурирования контента.</p>
<p>Я <em>настоятельно рекомендую</em> следовать современным стандартам.</p>
<p>Формула воды: H<sub>2</sub>O</p>

<!-- Плохо: неправильное использование -->
<p>Важно использовать <b>правильные теги</b> для структурирования контента.</p>
<p>Я <i>настоятельно рекомендую</i> следовать современным стандартам.</p>
<p>Формула воды: H<em>2</em>O</p>
```

### 2. Доступность
```html
<!-- Доступно: использование семантически правильных элементов -->
<p>Пожалуйста, <strong>срочно</strong> выполните следующие действия:</p>
<ol>
    <li><a href="#step1">Первый шаг</a></li>
    <li><a href="#step2">Второй шаг</a></li>
</ol>

<!-- Менее доступно: использование только визуального форматирования -->
<p><span style="font-weight: bold;">Пожалуйста, срочно</span> выполните следующие действия:</p>
```

### 3. SEO оптимизация
```html
<!-- SEO-оптимизировано -->
<h1>Руководство по HTML</h1>
<p>В этой статье мы рассмотрим <strong>основы HTML разметки</strong>, 
включая <code>семантические элементы</code> и <mark>лучшие практики</mark>.</p>

<!-- Менее SEO-оптимизировано -->
<h1>Руководство по HTML</h1>
<p>В этой статье мы рассмотрим <span style="font-weight: bold;">основы HTML разметки</span>, 
включая <span style="font-family: monospace;">семантические элементы</span> и 
<span style="background-color: yellow;">лучшие практики</span>.</p>
```

## Пример комплексного использования строчных элементов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Комплексное использование строчных элементов</title>
    <meta name="description" content="Пример использования различных строчных элементов HTML">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Комплексное использование строчных элементов">
    <meta property="og:description" content="Пример использования различных строчных элементов HTML">
    <meta property="og:type" content="article">
</head>
<body>
    <header>
        <h1>Современное использование строчных элементов</h1>
    </header>

    <main>
        <article itemscope itemtype="http://schema.org/Article">
            <header>
                <h2 itemprop="headline">Руководство по строчным элементам HTML</h2>
                <p>Автор: <span itemprop="author">Имя автора</span></p>
                <p>Опубликовано: 
                    <time datetime="2023-11-08T10:00:00+00:00" itemprop="datePublished">
                        8 ноября 2023
                    </time>
                </p>
            </header>

            <div itemprop="articleBody">
                <section>
                    <h3>Введение в строчные элементы</h3>
                    <p>Строчные элементы в HTML играют важную роль в <em>семантической разметке</em>. 
                    Они позволяют точно определить значение и структуру текстового содержимого.</p>
                    
                    <p>Как отмечает специалист по доступности <cite>Джон Смит</cite>: 
                    <q>Правильное использование строчных элементов — ключ к созданию 
                    <strong>доступного веб-контента</strong></q>.</p>
                </section>

                <section>
                    <h3>Практические примеры</h3>
                    <p>В математическом выражении <var>x</var> = <var>y</var> + <var>z</var> 
                    переменные обозначаются тегом <code>&lt;var&gt;</code>. 
                    При работе с системой используйте комбинацию клавиш 
                    <kbd>Ctrl</kbd> + <kbd>C</kbd> для копирования.</p>
                    
                    <p>Формула воды: H<sub>2</sub>O, а площадь круга: A = πr<sup>2</sup>. 
                    В программе <mark>наиболее важные функции</mark> выделены цветом.</p>
                    
                    <p>Старая версия спецификации: <s>не рекомендуется использовать</s>, 
                    используйте <ins>новую версию</ins> вместо этого.</p>
                </section>

                <section>
                    <h3>Лучшие практики</h3>
                    <p>При использовании строчных элементов помните:</p>
                    <ul>
                        <li>Используйте <code>&lt;strong&gt;</code> для обозначения <strong>важности</strong></li>
                        <li>Используйте <code>&lt;em&gt;</code> для обозначения <em>акцента</em></li>
                        <li>Используйте <code>&lt;mark&gt;</code> для выделения <mark>релевантного текста</mark></li>
                    </ul>
                    
                    <p>Как сказано в спецификации: 
                    <blockquote cite="https://html.spec.whatwg.org/">
                        <p>Строчные элементы не начинают новую строку и обычно отображаются в той же строке, 
                        что и соседние элементы.</p>
                    </blockquote></p>
                </section>
            </div>

            <footer>
                <p>Теги: 
                    <span itemprop="keywords">
                        <a href="/tag/html" rel="tag">HTML</a>, 
                        <a href="/tag/inline-elements" rel="tag">Строчные элементы</a>, 
                        <a href="/tag/semantics" rel="tag">Семантика</a>
                    </span>
                </p>
                <p>Обновлено: 
                    <time datetime="2023-11-08T14:30:00+00:00" itemprop="dateModified">
                        8 ноября 2023, 14:30
                    </time>
                </p>
            </footer>
        </article>
    </main>

    <footer>
        <p>&copy; 2023 Название компании. Все права защищены.</p>
    </footer>
</body>
</html>
```

## Заключение

Строчные элементы HTML предоставляют мощные возможности для семантического и структурного форматирования текста. Современное использование этих элементов должно учитывать не только визуальное представление, но и доступность, SEO и семантическую правильность. Правильный выбор и использование строчных элементов делает веб-контент более понятным для пользователей, вспомогательных технологий и поисковых систем.

## Следующие темы
- [[HTML/Элементы/Интерактивные элементы]]
- [[HTML/Формы]]
- [[HTML/Семантика]]
- [[HTML/Доступность]]

## Теги
#html #elements #inline-elements #semantics #web-development #frontend #text-formatting #accessibility #seo #html5
# HTML Теги и атрибуты

HTML теги и атрибуты являются основными строительными блоками веб-страниц. Понимание их использования критически важно для создания структурированного и функционального HTML кода.

## Что такое HTML теги?

HTML теги — это ключевые слова, заключенные в угловые скобки `< >`. Они определяют структуру и семантику содержимого веб-страницы.

### Основные теги

#### Блочные элементы
Блочные элементы занимают всю доступную ширину и начинаются с новой строки:

```html
<div>Блочный контейнер</div>
<p>Абзац текста</p>
<h1>Заголовок первого уровня</h1>
<h2>Заголовок второго уровня</h2>
<h3>Заголовок третьего уровня</h3>
<h4>Заголовок четвертого уровня</h4>
<h5>Заголовок пятого уровня</h5>
<h6>Заголовок шестого уровня</h6>
<section>Секция документа</section>
<article>Независимый контент</article>
<nav>Навигационный блок</nav>
<header>Заголовок документа или раздела</header>
<footer>Подвал документа или раздела</footer>
<main>Основное содержимое</main>
```

#### Строчные элементы
Строчные элементы занимают только необходимое пространство и не начинаются с новой строки:

```html
<span>Текстовый контейнер</span>
<a href="https://example.com">Ссылка</a>
<strong>Жирный текст</strong>
<em>Курсивный текст</em>
<img src="image.jpg" alt="Описание изображения">
<br>Разрыв строки
<small>Мелкий текст</small>
<code>Фрагмент кода</code>
```

## Атрибуты HTML элементов

Атрибуты предоставляют дополнительную информацию об элементах и всегда указываются в открывающем теге.

### Общие атрибуты

#### Универсальные атрибуты
Эти атрибуты могут использоваться с любыми HTML элементами:

```html
<!-- id - уникальный идентификатор элемента -->
<div id="main-content">Содержимое</div>

<!-- class - класс для стилизации и JavaScript -->
<p class="highlight">Выделенный абзац</p>

<!-- style - встроенные стили CSS -->
<span style="color: red;">Красный текст</span>

<!-- title - всплывающая подсказка -->
<button title="Нажмите для выполнения действия">Кнопка</button>

<!-- data-* - пользовательские атрибуты данных -->
<div data-user-id="123" data-role="admin">Пользователь</div>
```

#### Атрибуты доступности
```html
<!-- role - роль элемента для скринридеров -->
<div role="alert">Сообщение об ошибке</div>

<!-- aria-* - атрибуты доступности -->
<button aria-label="Закрыть">X</button>
<input aria-describedby="help-text" type="text">
<div id="help-text">Помощь по полю ввода</div>
```

### Атрибуты конкретных элементов

#### Атрибуты ссылок
```html
<a href="https://example.com" 
   target="_blank" 
   rel="noopener noreferrer"
   title="Перейти на внешний сайт">Ссылка</a>
```

#### Атрибуты изображений
```html
<img src="image.jpg" 
     alt="Описание изображения" 
     width="300" 
     height="200" 
     loading="lazy">
```

#### Атрибуты форм
```html
<form action="/submit" method="post" novalidate>
    <input type="email" 
           name="email" 
           required 
           placeholder="Введите email" 
           autocomplete="email">
    <textarea name="message" 
              rows="5" 
              cols="40" 
              maxlength="500"></textarea>
    <button type="submit">Отправить</button>
</form>
```

## Правила написания тегов и атрибутов

### Правильная вложенность
```html
<!-- Правильно -->
<p>Текст <strong>жирный текст</strong> продолжение</p>

<!-- Неправильно -->
<p>Текст <strong>жирный текст</p> продолжение</strong>
```

### Закрытие тегов
```html
<!-- Правильно -->
<div>
    <p>Абзац</p>
</div>

<!-- Неправильно -->
<div>
    <p>Абзац>
</div>
```

### Кавычки для значений атрибутов
```html
<!-- Рекомендуется -->
<img src="image.jpg" alt="Описание">

<!-- Также допустимо, если значение не содержит пробелов -->
<img src=image.jpg alt=описание>

<!-- Не рекомендуется -->
<img src=image.jpg alt=описание>
```

## Специальные символы и сущности

Некоторые символы имеют специальное значение в HTML и должны быть экранированы:

```html
<!-- Символы, которые нужно экранировать -->
&amp;  <!-- & (амперсанд) -->
&lt;   <!-- < (меньше) -->
&gt;   <!-- > (больше) -->
&quot;  <!-- " (двойная кавычка) -->
&apos;  <!-- ' (одинарная кавычка) -->

<!-- Пример использования -->
<p>Используйте &lt;div&gt; для создания блока</p>
```

## Пользовательские атрибуты данных

HTML5 позволяет использовать пользовательские атрибуты данных с префиксом `data-`:

```html
<div data-product-id="12345" 
     data-category="electronics" 
     data-price="299.99">
    Товар
</div>

<!-- Доступ к этим данным через JavaScript -->
<script>
const element = document.querySelector('div');
console.log(element.dataset.productId); // 12345
console.log(element.dataset.category);  // electronics
</script>
```

## Заключение

Правильное использование HTML тегов и атрибутов является ключом к созданию семантически правильных, доступных и SEO-оптимизированных веб-страниц. Понимание различий между блочными и строчными элементами, а также знание универсальных и специфических атрибутов помогает создавать более качественный код.

## Современные HTML5 теги и их применение

HTML5 ввел множество новых семантических элементов, которые делают разметку более понятной:

### Структурные элементы
```html
<header>     <!-- Заголовок документа или раздела -->
<nav>        <!-- Навигационные ссылки -->
<main>       <!-- Основное содержимое документа -->
<article>    <!-- Самостоятельный контент -->
<section>    <!-- Тематически связанная группа содержимого -->
<aside>      <!-- Сопутствующий контент -->
<footer>     <!-- Подвал документа или раздела -->
<figure>     <!-- Иллюстрация с подписью -->
<figcaption> <!-- Подпись к иллюстрации -->
```

### Медиа элементы
```html
<audio>      <!-- Аудио контент -->
<video>      <!-- Видео контент -->
<source>     <!-- Источник медиа контента -->
<track>      <!-- Текстовые дорожки для видео/аудио -->
```

### Интерактивные элементы
```html
<details>    <!-- Раскрывающийся раздел -->
<summary>    <!-- Заголовок для details -->
<dialog>     <!-- Диалоговое окно -->
<menu>       <!-- Меню команд -->
```

## Расширенные атрибуты доступности

### ARIA-атрибуты
```html
<!-- Роли элементов -->
<div role="button">Кнопка-имитация</div>
<div role="navigation">Навигация</div>
<div role="main">Основное содержимое</div>

<!-- Состояния -->
<button aria-pressed="false">Переключатель</button>
<input type="text" aria-invalid="true" aria-errormessage="error-id">

<!-- Свойства -->
<button aria-describedby="help-text">Помощь</button>
<div id="help-text">Эта кнопка выполняет важное действие</div>

<!-- Структура -->
<ul role="listbox" aria-activedescendant="option1">
    <li id="option1" role="option">Опция 1</li>
    <li id="option2" role="option">Опция 2</li>
</ul>
```

### Глобальные ARIA атрибуты
```html
<!-- Описание содержимого -->
<div aria-label="Описание элемента">Содержимое</div>
<div aria-labelledby="label-id">Содержимое</div>

<!-- Живые области -->
<div aria-live="polite">Обновляемое содержимое</div>
<div aria-live="assertive">Критические обновления</div>

<!-- Атомарность обновлений -->
<div aria-atomic="true">Обновляется целиком</div>

<!-- Взаимодействие -->
<div aria-disabled="true">Неактивный элемент</div>
<div aria-hidden="true">Скрыт для вспомогательных технологий</div>
```

## Современные атрибуты форм

### Валидация
```html
<form>
    <!-- Обязательные поля -->
    <input type="email" required>
    
    <!-- Ограничения длины -->
    <input type="text" minlength="3" maxlength="20">
    
    <!-- Регулярные выражения -->
    <input type="text" pattern="[A-Za-z]{3,20}" title="Только буквы, 3-20 символов">
    
    <!-- Числовые ограничения -->
    <input type="number" min="1" max="100" step="5">
    
    <!-- Автозаполнение -->
    <input type="text" name="name" autocomplete="name">
    <input type="email" name="email" autocomplete="email">
    <input type="tel" name="tel" autocomplete="tel">
</form>
```

### Дата и время
```html
<!-- Типы полей -->
<input type="date" min="2023-01-01" max="2024-12-31">
<input type="time" min="09:00" max="18:00">
<input type="datetime-local">
<input type="month">
<input type="week">
<input type="color">
<input type="range" min="0" max="100" value="50">
```

## Атрибуты производительности

### Изображения
```html
<!-- Ленивая загрузка -->
<img src="image.jpg" alt="Описание" loading="lazy">

<!-- Декларативное захватывание -->
<img src="image.jpg" alt="Описание" fetchpriority="high">

<!-- Адаптивные изображения -->
<picture>
    <source media="(max-width: 799px)" srcset="small.webp" type="image/webp">
    <source media="(max-width: 799px)" srcset="small.jpg">
    <source media="(min-width: 800px)" srcset="large.webp" type="image/webp">
    <source media="(min-width: 800px)" srcset="large.jpg">
    <img src="fallback.jpg" alt="Описание">
</picture>
```

### Ссылки
```html
<!-- Отложенная загрузка -->
<link rel="preload" href="critical.css" as="style">
<link rel="prefetch" href="next-page.html">

<!-- Подключение к ресурсам -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="dns-prefetch" href="//analytics.google.com">

<!-- Безопасность -->
<a href="https://example.com" rel="noopener noreferrer" target="_blank">Ссылка</a>
```

## Специфические атрибуты элементов

### Таблицы
```html
<table>
    <caption>Описание таблицы</caption>
    <thead>
        <tr>
            <th scope="col">Заголовок столбца</th>
            <th scope="row">Заголовок строки</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td headers="col1 row1">Ячейка</td>
        </tr>
    </tbody>
</table>
```

### Формы
```html
<form>
    <!-- Группировка -->
    <fieldset>
        <legend>Описание группы</legend>
        <input type="radio" name="option" value="1">
    </fieldset>
    
    <!-- Выбор -->
    <select name="country" size="5" multiple>
        <optgroup label="Европа">
            <option value="de">Германия</option>
        </optgroup>
    </select>
    
    <!-- Многострочный ввод -->
    <textarea name="comment" rows="5" cols="50" wrap="soft"></textarea>
</form>
```

## Пользовательские атрибуты данных

HTML5 позволяет использовать пользовательские атрибуты данных с префиксом `data-`:

```html
<!-- Простые данные -->
<div data-product-id="12345" 
     data-category="electronics" 
     data-price="299.99"
     data-in-stock>
    Товар
</div>

<!-- Сложные данные (JSON) -->
<div data-config='{"theme": "dark", "lang": "ru", "features": ["a", "b"]}'>
    Элемент с настройками
</div>

<!-- Использование в JavaScript -->
<script>
const element = document.querySelector('div');
console.log(element.dataset.productId); // 12345
console.log(element.dataset.category);  // electronics
console.log(element.dataset.inStock);   // true (т.к. атрибут присутствует)
console.log(element.dataset.config);    // {"theme": "dark", ...}
</script>
```

## Лучшие практики использования тегов и атрибутов

### 1. Семантическая разметка
```html
<!-- Хорошо: семантически правильная разметка -->
<article>
    <header>
        <h2>Заголовок статьи</h2>
    </header>
    <section>
        <h3>Подраздел</h3>
        <p>Содержимое...</p>
    </section>
    <footer>
        <p>Опубликовано: <time datetime="2023-11-08">8 ноября 2023</time></p>
    </footer>
</article>

<!-- Плохо: использование div для всего -->
<div>
    <div>
        <div>Заголовок статьи</div>
    </div>
    <div>
        <div>Подраздел</div>
        <div>Содержимое...</div>
    </div>
    <div>
        <div>Опубликовано: 8 ноября 2023</div>
    </div>
</div>
```

### 2. Доступность
```html
<!-- С использованием доступности -->
<label for="email">Email:</label>
<input type="email" id="email" name="email" required aria-describedby="email-help">
<div id="email-help">Введите действительный email адрес</div>

<!-- Без доступности -->
<input type="email" name="email" placeholder="Email">
```

### 3. SEO оптимизация
```html
<!-- Оптимизированная разметка -->
<h1>Основной заголовок страницы</h1>
<h2>Подзаголовок</h2>
<p>Ключевое содержимое с важными ключевыми словами</p>

<!-- Использование микроразметки -->
<div itemscope itemtype="http://schema.org/Article">
    <h2 itemprop="headline">Заголовок статьи</h2>
    <p itemprop="author">Автор: <span itemprop="name">Имя автора</span></p>
    <time itemprop="datePublished" datetime="2023-11-08">8 ноября 2023</time>
</div>
```

## Проверка корректности разметки

### Инструменты проверки:
- W3C Markup Validator - проверка соответствия стандартам HTML
- HTMLHint - инструмент статического анализа HTML
- axe-core - проверка доступности
- Lighthouse - комплексная проверка
- Инструменты разработчика браузера

## Пример комплексной разметки с современными практиками

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Комплексная HTML разметка</title>
    <meta name="description" content="Пример современной HTML разметки">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Комплексная HTML разметка">
    <meta property="og:description" content="Пример современной HTML разметки">
    <meta property="og:type" content="article">
    
    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image">
</head>
<body>
    <!-- Ссылка для пропуска навигации -->
    <a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
    
    <header role="banner">
        <h1>Название сайта</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/blog">Блог</a></li>
            </ul>
        </nav>
    </header>

    <main id="main-content" role="main">
        <article itemscope itemtype="http://schema.org/Article">
            <header>
                <h2 itemprop="headline">Заголовок статьи</h2>
                <p>Автор: <span itemprop="author">Имя автора</span></p>
                <p>Опубликовано: 
                    <time datetime="2023-11-08T10:00:00+00:00" itemprop="datePublished">
                        8 ноября 2023
                    </time>
                </p>
            </header>

            <section>
                <h3>Введение</h3>
                <p itemprop="description">Вступительный абзац статьи...</p>
            </section>

            <section>
                <h3>Основная часть</h3>
                <p>Основное содержимое статьи...</p>
                
                <!-- Иллюстрация с подписью -->
                <figure>
                    <img src="image.jpg" alt="Описание изображения" loading="lazy">
                    <figcaption>Подпись к изображению</figcaption>
                </figure>
            </section>

            <footer>
                <p>Теги: 
                    <span itemprop="keywords">
                        <a href="/tag/html">HTML</a>, 
                        <a href="/tag/semantics">Семантика</a>
                    </span>
                </p>
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
    </footer>
</body>
</html>
```

## Заключение

Современное использование HTML тегов и атрибутов требует понимания не только базовых принципов семантики, но и аспектов доступности, SEO, производительности и пользовательского опыта. Комплексный подход к разметке делает веб-страницы более доступными, понятными и эффективными для всех пользователей.

## Следующие темы
- [[HTML/Семантика]]
- [[HTML/Элементы/Блочные элементы]]
- [[HTML/Элементы/Строчные элементы]]
- [[HTML/Доступность]]

## Теги
#html #tags #attributes #markup #web-development #frontend #basics #semantics #accessibility #seo #html5 #performance
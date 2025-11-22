# HTML Доступность: Основы

Доступность веб-контента (web accessibility) - это практика создания веб-сайтов и приложений, которые могут использоваться всеми людьми, включая тех, кто имеет ограничения по здоровью. HTML предоставляет множество инструментов для обеспечения доступности.

## Что такое доступность веб-контента?

Доступность веб-контента означает, что веб-сайты, инструменты и технологии разработаны так, чтобы люди с ограниченными возможностями могли использовать их. Это включает в себя людей с:
- Визуальными ограничениями (слепота, слабовидение)
- Слуховыми ограничениями (глухота, слабослышание)
- Ограничениями опорно-двигательного аппарата
- Когнитивными ограничениями
- Неврологическими расстройствами

## Основные принципы доступности (WCAG)

### POUR - основные принципы:
1. **Perceivable (Воспринимаемость)** - информация должна быть представлена способами, которые пользователи могут воспринять
2. **Operable (Управляемость)** - интерфейс должен быть удобен в использовании
3. **Understandable (Понятность)** - информация и работа интерфейса должны быть понятны
4. **Robust (Надежность)** - контент должен быть надежным, чтобы различные вспомогательные технологии могли его использовать

## Структура документа и доступность

### Правильная иерархия заголовков

```html
<!-- Правильная иерархия заголовков -->
<main>
    <h1>Основной заголовок страницы</h1>
    
    <section>
        <h2>Первый раздел</h2>
        <p>Содержимое первого раздела...</p>
        
        <section>
            <h3>Подраздел</h3>
            <p>Содержимое подраздела...</p>
        </section>
    </section>
    
    <section>
        <h2>Второй раздел</h2>
        <p>Содержимое второго раздела...</p>
    </section>
</main>
```

### Семантические элементы

```html
<!-- Использование семантических элементов -->
<header>
    <h1>Название сайта</h1>
    <nav aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
        </ul>
    </nav>
</header>

<main>
    <article>
        <h2>Заголовок статьи</h2>
        <p>Содержимое статьи...</p>
    </article>
</main>

<aside aria-label="Боковая панель">
    <h3>Реклама</h3>
    <p>Рекламный контент</p>
</aside>

<footer>
    <p>&copy; 2023 Все права защищены</p>
</footer>
```

## Альтернативный текст

### Изображения

```html
<!-- Хороший альтернативный текст -->
<img src="chart.png" alt="График роста продаж за последний квартал, показывающий увеличение на 25%">

<!-- Декоративное изображение -->
<img src="decoration.png" alt="">

<!-- Иконка с поясняющим текстом -->
<div>
    <img src="warning-icon.png" alt="Предупреждение: ">
    <p>Важное уведомление для пользователя</p>
</div>
```

### Медиа-элементы

```html
<!-- Видео с субтитрами -->
<video controls width="640" height="360">
    <source src="video.mp4" type="video/mp4">
    <track kind="subtitles" src="subtitles.vtt" srclang="ru" label="Русские">
    Ваш браузер не поддерживает видео элемент.
</video>

<!-- Аудио с транскрипцией -->
<audio controls>
    <source src="podcast.mp3" type="audio/mpeg">
    <p>Транскрипция подкаста доступна ниже.</p>
</audio>
```

## ARIA (Accessible Rich Internet Applications)

ARIA - это набор атрибутов, которые позволяют разработчикам улучшить доступность веб-контента.

### Основные ARIA атрибуты

```html
<!-- Роль элемента -->
<div role="button" tabindex="0" onclick="doSomething()">Кнопка-имитация</div>

<!-- Описание элемента -->
<button aria-label="Закрыть">✕</button>

<button aria-describedby="help-text">Помощь</button>
<div id="help-text">Нажмите для получения справки</div>

<!-- Состояния -->
<input type="checkbox" id="terms" aria-checked="false">
<label for="terms">Согласен с условиями</label>

<!-- Структура списка -->
<ul role="list">
    <li role="listitem">Элемент 1</li>
    <li role="listitem">Элемент 2</li>
</ul>
```

### ARIA для сложных компонентов

```html
<!-- Раскрывающийся список -->
<div role="combobox" aria-haspopup="listbox" aria-expanded="false">
    <input type="text" role="combobox" aria-autocomplete="list" aria-controls="suggestions">
    <ul id="suggestions" role="listbox">
        <li role="option">Вариант 1</li>
        <li role="option">Вариант 2</li>
    </ul>
</div>

<!-- Вкладки -->
<div role="tablist">
    <button role="tab" aria-selected="true" aria-controls="panel1">Вкладка 1</button>
    <button role="tab" aria-selected="false" aria-controls="panel2">Вкладка 2</button>
</div>

<div id="panel1" role="tabpanel" aria-labelledby="tab1">
    Содержимое первой вкладки
</div>
<div id="panel2" role="tabpanel" aria-labelledby="tab2" hidden>
    Содержимое второй вкладки
</div>
```

## Клавиатурная навигация

### Атрибут `tabindex`

```html
<!-- Элемент в порядке табуляции -->
<input type="text" tabindex="0">

<!-- Элемент в специфическом порядке табуляции -->
<input type="text" tabindex="1">
<button tabindex="2">Кнопка</button>

<!-- Элемент исключенный из табуляции -->
<div tabindex="-1">Не получает фокус при табуляции</div>
```

### Пропуск навигации

```html
<!-- Ссылка для пропуска навигации -->
<a class="skip-link" href="#main-content">Перейти к основному содержимому</a>

<header>
    <nav>
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
        </ul>
    </nav>
</header>

<main id="main-content">
    <h1>Основное содержимое</h1>
    <p>Текст страницы...</p>
</main>
```

## Формы и доступность

### Правильная разметка форм

```html
<!-- Использование label -->
<label for="email">Email:</label>
<input type="email" id="email" name="email">

<!-- Вложенная метка -->
<label>
    <input type="checkbox" name="subscribe"> Подписаться на рассылку
</label>

<!-- Группировка элементов -->
<fieldset>
    <legend>Выберите ваш пол:</legend>
    <label><input type="radio" name="gender" value="male"> Мужской</label>
    <label><input type="radio" name="gender" value="female"> Женский</label>
</fieldset>

<!-- Ошибки формы -->
<label for="password">Пароль:</label>
<input type="password" id="password" name="password" aria-describedby="password-error">
<div id="password-error" class="error" aria-live="polite">Пароль должен содержать не менее 8 символов</div>
```

### Состояния формы

```html
<!-- Фокус на элементе формы -->
<input type="text" id="search" name="search" aria-label="Поиск" autofocus>

<!-- Неактивный элемент -->
<button disabled aria-disabled="true">Неактивная кнопка</button>

<!-- Обязательное поле -->
<label for="name">Имя (обязательно):</label>
<input type="text" id="name" name="name" required aria-required="true">
```

## Цвет и контрастность

### Контрастность текста

```css
/* Достаточная контрастность для нормального текста (3:1) */
.high-contrast-text {
    color: #000000;
    background-color: #ffffff;
}

/* Достаточная контрастность для крупного текста (4.5:1) */
.normal-contrast-text {
    color: #333333;
    background-color: #ffffff;
}

/* Недостаточная контрастность (избегать) */
.low-contrast-text {
    color: #aaaaaa;
    background-color: #eeeeee;
}
```

## Стили фокуса

```css
/* Видимый фокус для клавиатурной навигации */
a:focus,
button:focus,
input:focus,
select:focus,
textarea:focus {
    outline: 2px solid #007acc;
    outline-offset: 2px;
}

/* Никогда не удалять outline без замены */
/* ПЛОХО: a:focus { outline: none; } */

/* ХОРОШО: Заменить outline на другой видимый индикатор */
button:focus {
    outline: 2px solid #007acc;
    background-color: #e6f0ff;
}
```

## Семантические таблицы

```html
<!-- Правильная разметка таблицы -->
<table>
    <caption>Ежемесячные продажи за 2023 год</caption>
    <thead>
        <tr>
            <th scope="col">Месяц</th>
            <th scope="col">Продажи</th>
            <th scope="col">Рост</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th scope="row">Январь</th>
            <td>100,000</td>
            <td>+5%</td>
        </tr>
        <tr>
            <th scope="row">Февраль</th>
            <td>110,000</td>
            <td>+10%</td>
        </tr>
    </tbody>
</table>
```

## Динамическое содержимое

### ARIA live области

```html
<!-- Область с динамическим содержимым -->
<div aria-live="polite" aria-atomic="false" id="notifications">
    <!-- Сообщения будут добавляться сюда -->
</div>

<!-- Важные сообщения -->
<div aria-live="assertive" id="alerts">
    <!-- Критические уведомления -->
</div>
```

### Пример с JavaScript

```html
<button onclick="showNotification()">Показать уведомление</button>
<div id="notification-area" aria-live="polite"></div>

<script>
function showNotification() {
    const area = document.getElementById('notification-area');
    area.textContent = 'Уведомление: Действие выполнено успешно!';
}
</script>
```

## Проверка доступности

### Инструменты для проверки

1. **Встроенные инструменты браузера** (инспектор DOM, проверка контраста)
2. **Расширения браузера**: axe, WAVE, Lighthouse
3. **Командная строка**: pa11y, axe-core
4. **Онлайн инструменты**: WebAIM WAVE, Accessibility Insights

### Пример проверки с axe-core

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Страница с проверкой доступности</title>
    <script src="https://cdn.jsdelivr.net/npm/axe-core@4.7.2/axe.min.js"></script>
</head>
<body>
    <div id="main-content">
        <h1>Основной заголовок</h1>
        <p>Содержимое страницы</p>
    </div>

    <script>
        axe.run('#main-content', { 
            runOnly: {
                type: 'tag',
                values: ['wcag2a', 'wcag2aa']
            }
        }, function(err, results) {
            if (err) {
                console.error(err);
                return;
            }
            
            if (results.violations.length) {
                console.log('Найдены проблемы доступности:', results.violations);
            } else {
                console.log('Страница соответствует стандартам доступности');
            }
        });
    </script>
</body>
</html>
```

## Заключение

Доступность - это не опциональная функция, а важная часть разработки веб-сайтов. Использование семантической разметки, правильных атрибутов ARIA, обеспечение клавиатурной навигации и контрастности текста помогает создать инклюзивный веб-опыт для всех пользователей. Регулярная проверка доступности в процессе разработки позволяет выявлять и устранять проблемы на ранних стадиях.

## Современные принципы доступности

### Инклюзивный дизайн
Инклюзивный дизайн — это подход к созданию продуктов и услуг, которые могут использовать все люди, включая тех, у кого есть ограниченные возможности. В контексте веб-доступности это означает:

```html
<!-- Пример инклюзивного дизайна -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Инклюзивный веб-сайт</title>
    <style>
        /* Гибкая вёрстка для адаптации под разные устройства */
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        /* Увеличенные сенсорные зоны для удобства использования */
        .button, .link {
            min-height: 44px;
            min-width: 44px;
            padding: 12px 16px;
        }
        
        /* Достаточный контраст текста */
        body {
            color: #333;
            background-color: #fff;
        }
        
        /* Увеличенный размер шрифта по умолчанию */
        body {
            font-size: 18px; /* или 1.125rem */
            line-height: 1.5;
        }
        
        /* Фокусные индикаторы */
        a:focus, button:focus, input:focus, select:focus, textarea:focus {
            outline: 2px solid #005fcc;
            outline-offset: 2px;
        }
    </style>
</head>
<body>
    <!-- Структура с семантическими элементами -->
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
        <article>
            <header>
                <h2>Заголовок статьи</h2>
            </header>
            
            <section>
                <h3>Подраздел статьи</h3>
                <p>Содержимое статьи...</p>
            </section>
        </article>
    </main>

    <footer role="contentinfo">
        <p>&copy; 2023 Все права защищены</p>
    </footer>
</body>
</html>
```

### Адаптивность и отзывчивость
Веб-сайт должен адаптироваться к различным условиям использования:

```html
<!-- Адаптивная навигация -->
<nav role="navigation" aria-label="Мобильная навигация">
    <button id="menu-toggle" aria-expanded="false" aria-controls="mobile-menu">
        <span class="sr-only">Открыть/закрыть меню</span>
        <span class="menu-icon" aria-hidden="true">☰</span>
    </button>
    
    <ul id="mobile-menu" hidden>
        <li><a href="/">Главная</a></li>
        <li><a href="/about">О нас</a></li>
        <li><a href="/services">Услуги</a></li>
        <li><a href="/contact">Контакты</a></li>
    </ul>
</nav>

<script>
document.getElementById('menu-toggle').addEventListener('click', function() {
    const menu = document.getElementById('mobile-menu');
    const isExpanded = this.getAttribute('aria-expanded') === 'true';
    
    this.setAttribute('aria-expanded', !isExpanded);
    menu.hidden = isExpanded;
});
</script>

<style>
/* Стили для адаптивной навигации */
@media (max-width: 768px) {
    nav ul {
        position: absolute;
        top: 100%;
        left: 0;
        width: 100%;
        background: white;
        box-shadow: 0 2px 5px rgba(0,0,0,0.2);
    }
    
    nav li {
        display: block;
    }
}
</style>
```

## Расширенные техники доступности

### Когнитивная доступность
Создание контента, который легко понимать и использовать людям с когнитивными особенностями:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Когнитивная доступность</title>
    <style>
        /* Простая и понятная визуальная структура */
        .page-content {
            max-width: 800px;
            margin: 0 auto;
            font-family: Arial, sans-serif; /* Избегаем декоративных шрифтов */
        }
        
        /* Четкая иерархия заголовков */
        h1, h2, h3, h4 {
            margin-top: 1.5em;
            margin-bottom: 0.5em;
        }
        
        /* Достаточный интервал между элементами */
        p, ul, ol, div {
            margin-bottom: 1em;
        }
        
        /* Высококонтрастные цвета для важных элементов */
        .important {
            background-color: #fff9c4; /* мягкий желтый фон */
            border-left: 4px solid #f57f17; /* контрастная граница */
            padding: 15px;
        }
    </style>
</head>
<body>
    <main>
        <h1>Как использовать наш сервис</h1>
        
        <div class="important">
            <p><strong>Важно:</strong> Прежде чем начать, убедитесь, что у вас есть все необходимые документы.</p>
        </div>
        
        <ol>
            <li>
                <h3>Шаг 1: Регистрация</h3>
                <p>Перейдите на страницу регистрации и заполните форму.</p>
            </li>
            <li>
                <h3>Шаг 2: Подтверждение</h3>
                <p>Проверьте ваш email и подтвердите регистрацию по ссылке.</p>
            </li>
            <li>
                <h3>Шаг 3: Использование сервиса</h3>
                <p>После подтверждения вы можете начать использовать сервис.</p>
            </li>
        </ol>
        
        <section aria-labelledby="faq-title">
            <h2 id="faq-title">Часто задаваемые вопросы</h2>
            <details>
                <summary>Как сбросить пароль?</summary>
                <p>Нажмите на ссылку "Забыли пароль" на странице входа.</p>
            </details>
            <details>
                <summary>Как связаться с поддержкой?</summary>
                <p>Используйте форму обратной связи или напишите на support@example.com</p>
            </details>
        </section>
    </main>
</body>
</html>
```

### Сенсорная доступность
Обеспечение удобства использования для людей с сенсорными ограничениями:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Сенсорная доступность</title>
    <style>
        /* Увеличенные сенсорные зоны */
        .clickable-area {
            min-height: 44px;
            min-width: 44px;
            padding: 12px;
            margin: 8px;
            display: inline-block;
        }
        
        /* Предотупреждение об анимации */
        .reduced-motion {
            animation-duration: 0.01ms !important;
            animation-iteration-count: 1 !important;
            transition-duration: 0.01ms !important;
        }
        
        /* Стили для режима высокой контрастности */
        @media (prefers-contrast: high) {
            body {
                background-color: black;
                color: white;
            }
            
            .button {
                border: 2px solid white;
                background-color: black;
                color: white;
            }
        }
        
        /* Стили для пользователей, предпочитающих уменьшенную анимацию */
        @media (prefers-reduced-motion: reduce) {
            * {
                animation-duration: 0.01ms !important;
                animation-iteration-count: 1 !important;
                transition-duration: 0.01ms !important;
            }
        }
    </style>
</head>
<body>
    <button class="clickable-area">Большая кнопка</button>
    <a href="/page" class="clickable-area">Большая ссылка</a>
    
    <div class="animated-element" aria-label="Анимированный элемент">
        <!-- Содержимое с анимацией -->
    </div>
</body>
</html>
```

## Доступность мультимедиа

### Видео и аудио контент
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступность мультимедиа</title>
</head>
<body>
    <main>
        <h1>Обучающее видео по HTML</h1>
        
        <figure>
            <video controls 
                   width="640" 
                   height="360" 
                   preload="metadata"
                   aria-describedby="video-description">
                <source src="html-tutorial.mp4" type="video/mp4">
                <source src="html-tutorial.webm" type="video/webm">
                
                <!-- Треки субтитров и описаний -->
                <track kind="subtitles" 
                       src="html-tutorial-ru.vtt" 
                       srclang="ru" 
                       label="Русские">
                <track kind="subtitles" 
                       src="html-tutorial-en.vtt" 
                       srclang="en" 
                       label="English">
                <track kind="descriptions" 
                       src="html-tutorial-desc.vtt" 
                       srclang="ru" 
                       label="Описания для слабовидящих">
                <track kind="captions" 
                       src="html-tutorial-captions.vtt" 
                       srclang="ru" 
                       label="Титры (включая звуки)">
                
                Ваш браузер не поддерживает видео элемент.
            </video>
            
            <figcaption id="video-description">
                Обучающее видео по основам HTML. Продолжительность: 15 минут.
            </figcaption>
        </figure>
        
        <!-- Аудио с транскрипцией -->
        <figure>
            <audio controls aria-describedby="audio-description">
                <source src="podcast.mp3" type="audio/mpeg">
                <source src="podcast.ogg" type="audio/ogg">
                Ваш браузер не поддерживает аудио элемент.
            </audio>
            
            <figcaption id="audio-description">
                Подкаст о современных веб-технологиях
            </figcaption>
        </figure>
        
        <!-- Транскрипция аудио -->
        <section aria-labelledby="transcription-title">
            <h2 id="transcription-title">Транскрипция подкаста</h2>
            <div class="transcription">
                <p><strong>Ведущий:</strong> Добро пожаловать на наш подкаст о веб-технологиях...</p>
                <p><strong>Гость:</strong> Сегодня мы поговорим о последних тенденциях в веб-разработке...</p>
                <!-- Полная транскрипция -->
            </div>
        </section>
    </main>
</body>
</html>
```

### Изображения и визуальный контент
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступность изображений</title>
</head>
<body>
    <main>
        <h1>Доступные изображения</h1>
        
        <!-- Декоративное изображение -->
        <img src="decorative-border.png" alt="" role="presentation">
        
        <!-- Контентное изображение -->
        <img src="chart.png" 
             alt="Диаграмма роста пользователей за последние 12 месяцев. Наблюдается стабильный рост с 1000 до 5000 пользователей."
             longdesc="chart-description.html">
        
        <!-- Информативное изображение с подробным описанием -->
        <figure>
            <img src="infographic.png" 
                 alt="Инфографика о преимуществах доступного веб-дизайна">
            <figcaption>
                Инфографика показывает, как доступный дизайн улучшает опыт всех пользователей.
            </figcaption>
        </figure>
        
        <!-- Изображение с функцией -->
        <button aria-label="Закрыть модальное окно">
            <img src="close-icon.png" alt="">
        </button>
        
        <!-- Комплексное изображение с детализированным описанием -->
        <div class="image-with-description">
            <img src="complex-diagram.png" 
                 alt="Сложная диаграмма системных взаимодействий"
                 aria-describedby="diagram-details">
            <div id="diagram-details">
                <h3>Описание диаграммы</h3>
                <p>Диаграмма показывает взаимодействие между различными компонентами системы...</p>
                <ul>
                    <li>Компонент A взаимодействует с Компонентом B через API</li>
                    <li>Компонент B обрабатывает данные и передает их Компоненту C</li>
                </ul>
            </div>
        </div>
    </main>
</body>
</html>
```

## Современные инструменты доступности

### Инструменты разработчика
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Инструменты доступности</title>
</head>
<body>
    <main>
        <h1>Проверка доступности</h1>
        
        <!-- Пример использования axe-core для автоматизированной проверки -->
        <div id="content-to-test">
            <h2>Содержимое для проверки</h2>
            <p>Этот контент будет проверен на доступность.</p>
        </div>
        
        <button id="run-a11y-check">Проверить доступность</button>
    </main>

    <script src="https://cdn.jsdelivr.net/npm/axe-core@4.7.2/axe.min.js"></script>
    <script>
        document.getElementById('run-a11y-check').addEventListener('click', function() {
            axe.run('#content-to-test', {
                runOnly: {
                    type: 'tag',
                    values: ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa']
                }
            }, function(err, results) {
                if (err) {
                    console.error('Ошибка проверки:', err);
                    return;
                }

                if (results.violations.length) {
                    console.log('Найдены нарушения доступности:');
                    results.violations.forEach(violation => {
                        console.log(`- ${violation.help}: ${violation.nodes.map(node => node.html).join(', ')}`);
                    });
                    
                    // Отображение результатов пользователю
                    const violationsList = document.createElement('ul');
                    results.violations.forEach(violation => {
                        const li = document.createElement('li');
                        li.innerHTML = `<strong>${violation.help}:</strong> ${violation.nodes.map(node => node.target.join(' ')).join(', ')}`;
                        violationsList.appendChild(li);
                    });
                    
                    document.body.appendChild(violationsList);
                } else {
                    console.log('Страница соответствует стандартам доступности');
                    alert('Страница соответствует стандартам доступности!');
                }
            });
        });
    </script>
</body>
</html>
```

### Расширения браузера для проверки
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Расширения для доступности</title>
</head>
<body>
    <main>
        <h1>Полезные расширения браузера</h1>
        
        <section aria-labelledby="accessibility-tools">
            <h2 id="accessibility-tools">Инструменты для проверки доступности</h2>
            
            <ul>
                <li>
                    <h3>axe DevTools</h3>
                    <p>Автоматизированная проверка доступности в инструментах разработчика</p>
                </li>
                <li>
                    <h3>WAVE</h3>
                    <p>Расширение для визуальной проверки доступности</p>
                </li>
                <li>
                    <h3>Lighthouse</h3>
                    <p>Комплексная проверка производительности, доступности и SEO</p>
                </li>
            </ul>
        </section>
    </main>
</body>
</html>
```

## Лучшие практики современной доступности

### 1. Инклюзивная разработка с самого начала
```html
<!-- Планирование доступности на этапе проектирования -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Инклюзивный подход</title>
    <style>
        /* Раннее планирование доступности */
        :root {
            --focus-color: #005fcc;
            --text-color: #333;
            --bg-color: #fff;
            --contrast-ratio: 4.5; /* Минимальный контраст для нормального текста */
        }
        
        body {
            color: var(--text-color);
            background-color: var(--bg-color);
            font-size: 18px; /* Минимум 16px для удобства чтения */
            line-height: 1.5; /* Улучшает читаемость */
        }
        
        /* Стандартные фокусные стили */
        *:focus {
            outline: 2px solid var(--focus-color);
            outline-offset: 2px;
        }
    </style>
</head>
<body>
    <header>
        <h1>Проект с доступностью в приоритете</h1>
    </header>
    
    <nav aria-label="Основная навигация">
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/features">Возможности</a></li>
            <li><a href="/accessibility">Доступность</a></li>
        </ul>
    </nav>
    
    <main>
        <h2>Наши ценности</h2>
        <p>Мы верим, что веб должен быть доступен для всех пользователей, 
        независимо от их возможностей и используемых технологий.</p>
    </main>
</body>
</html>
```

### 2. Тестирование с реальными пользователями
```html
<!-- Пример формы обратной связи по доступности -->
<form action="/accessibility-feedback" method="post">
    <fieldset>
        <legend>Отзыв о доступности</legend>
        
        <div class="form-group">
            <label for="user-experience">Какой у вас опыт использования веб-сайтов?</label>
            <select id="user-experience" name="experience">
                <option value="none">Нет опыта</option>
                <option value="beginner">Начинающий</option>
                <option value="intermediate">Средний</option>
                <option value="advanced">Продвинутый</option>
            </select>
        </div>
        
        <div class="form-group">
            <label for="assistive-tech">Какие вспомогательные технологии вы используете?</label>
            <div class="checkbox-group">
                <label>
                    <input type="checkbox" name="assistive_tech" value="screen_reader">
                    Скринридер
                </label>
                <label>
                    <input type="checkbox" name="assistive_tech" value="voice_control">
                    Голосовое управление
                </label>
                <label>
                    <input type="checkbox" name="assistive_tech" value="magnification">
                    Увеличение экрана
                </label>
                <label>
                    <input type="checkbox" name="assistive_tech" value="other">
                    Другое
                </label>
            </div>
        </div>
        
        <div class="form-group">
            <label for="feedback">Ваши комментарии и предложения по улучшению доступности:</label>
            <textarea id="feedback" name="feedback" rows="6" cols="50"></textarea>
        </div>
        
        <button type="submit">Отправить отзыв</button>
    </fieldset>
</form>
```

### 3. Постоянное обучение и улучшение
```html
<!-- Страница ресурсов по доступности -->
<main>
    <h1>Ресурсы по веб-доступности</h1>
    
    <section aria-labelledby="guidelines">
        <h2 id="guidelines">Руководства и стандарты</h2>
        <ul>
            <li><a href="https://www.w3.org/WAI/WCAG21/quickref/">WCAG 2.1 Quick Reference</a></li>
            <li><a href="https://www.w3.org/WAI/ARIA/apg/">ARIA Authoring Practices Guide</a></li>
            <li><a href="https://webaim.org/">WebAIM Resources</a></li>
        </ul>
    </section>
    
    <section aria-labelledby="tools">
        <h2 id="tools">Инструменты проверки</h2>
        <ul>
            <li><a href="https://www.deque.com/axe/">axe Accessibility Testing</a></li>
            <li><a href="https://webaim.org/projects/wave/">WAVE Evaluation Tool</a></li>
            <li><a href="https://developers.google.com/web/tools/lighthouse">Lighthouse</a></li>
        </ul>
    </section>
    
    <section aria-labelledby="training">
        <h2 id="training">Обучение и курсы</h2>
        <ul>
            <li><a href="https://www.w3.org/WAI/fundamentals/">W3C WAI Fundamentals</a></li>
            <li><a href="https://web.dev/accessible/">Web.dev Accessibility</a></li>
            <li><a href="https://a11yproject.com/">The A11Y Project</a></li>
        </ul>
    </section>
</main>
```

## Пример комплексной страницы с доступностью

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Доступный веб-сайт - Пример реализации</title>
    <meta name="description" content="Пример веб-сайта с полной реализацией принципов доступности">
    
    <!-- Open Graph -->
    <meta property="og:title" content="Доступный веб-сайт - Пример реализации">
    <meta property="og:description" content="Пример веб-сайта с полной реализацией принципов доступности">
    <meta property="og:type" content="website">
    
    <style>
        /* Сброс стилей для доступности */
        * {
            box-sizing: border-box;
        }
        
        /* Базовые стили доступности */
        body {
            font-family: Arial, sans-serif;
            font-size: 18px;
            line-height: 1.5;
            color: #333;
            background-color: #fff;
            margin: 0;
            padding: 0;
        }
        
        /* Стили для пропуска навигации */
        .skip-link {
            position: absolute;
            top: -40px;
            left: 6px;
            background: #000;
            color: #fff;
            padding: 8px;
            text-decoration: none;
            border-radius: 4px;
            z-index: 1000;
        }
        
        .skip-link:focus {
            top: 6px;
        }
        
        /* Стили для фокуса */
        a:focus, button:focus, input:focus, select:focus, textarea:focus, [tabindex]:focus {
            outline: 2px solid #005fcc;
            outline-offset: 2px;
        }
        
        /* Контейнеры */
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        /* Навигация */
        nav ul {
            list-style: none;
            padding: 0;
            margin: 0;
            display: flex;
            flex-wrap: wrap;
        }
        
        nav li {
            margin-right: 20px;
        }
        
        nav a {
            display: block;
            padding: 10px 15px;
            text-decoration: none;
            color: #007acc;
        }
        
        nav a:hover, nav a:focus {
            background-color: #f0f0f0;
            color: #005a9e;
        }
        
        /* Кнопки */
        .button {
            display: inline-block;
            padding: 12px 24px;
            background-color: #007acc;
            color: white;
            text-decoration: none;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        
        .button:hover, .button:focus {
            background-color: #005a9e;
        }
        
        /* Заголовки */
        h1, h2, h3, h4, h5, h6 {
            margin-top: 1.5em;
            margin-bottom: 0.5em;
        }
        
        /* Изображения */
        img {
            max-width: 100%;
            height: auto;
        }
        
        /* Формы */
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 10px;
            border: 2px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }
        
        input:focus, select:focus, textarea:focus {
            border-color: #007acc;
        }
        
        /* Таблицы */
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        
        th, td {
            border: 1px solid #ddd;
            padding: 12px;
            text-align: left;
        }
        
        th {
            background-color: #f2f2f2;
        }
        
        /* Скрытие элементов для скринридеров */
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
        
        /* Адаптивность */
        @media (max-width: 768px) {
            nav ul {
                flex-direction: column;
            }
            
            nav li {
                margin: 5px 0;
            }
        }
        
        /* Уменьшенная анимация */
        @media (prefers-reduced-motion: reduce) {
            * {
                animation-duration: 0.01ms !important;
                animation-iteration-count: 1 !important;
                transition-duration: 0.01ms !important;
            }
        }
    </style>
</head>
<body>
    <!-- Ссылка для пропуска навигации -->
    <a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
    <a class="skip-link" href="#search">Перейти к поиску</a>
    
    <header role="banner">
        <div class="container">
            <h1>Доступный веб-сайт</h1>
            
            <nav role="navigation" aria-label="Основная навигация">
                <ul>
                    <li><a href="/" aria-current="page">Главная</a></li>
                    <li><a href="/about">О нас</a></li>
                    <li><a href="/services">Услуги</a></li>
                    <li><a href="/accessibility">Доступность</a></li>
                    <li><a href="/contact">Контакты</a></li>
                </ul>
            </nav>
            
            <div role="search" id="search">
                <form action="/search" method="get">
                    <div class="form-group">
                        <label for="search-input">Поиск по сайту:</label>
                        <input type="search" id="search-input" name="q" placeholder="Введите поисковый запрос...">
                        <button type="submit" class="button">Найти</button>
                    </div>
                </form>
            </div>
        </div>
    </header>

    <main id="main-content" role="main">
        <div class="container">
            <h2>Добро пожаловать на наш доступный сайт</h2>
            
            <p>Мы стремимся сделать наш веб-сайт максимально доступным для всех пользователей, 
            включая тех, кто использует вспомогательные технологии или имеет ограниченные возможности.</p>
            
            <section aria-labelledby="features-title">
                <h3 id="features-title">Наши возможности</h3>
                
                <ul>
                    <li>Семантическая разметка для лучшего понимания структуры</li>
                    <li>Правильные атрибуты ARIA для вспомогательных технологий</li>
                    <li>Контрастность текста в соответствии с WCAG</li>
                    <li>Клавиатурная навигация по всем интерактивным элементам</li>
                    <li>Альтернативный текст для всех изображений</li>
                </ul>
            </section>
            
            <section aria-labelledby="accessibility-commitment">
                <h3 id="accessibility-commitment">Наше обязательство по доступности</h3>
                
                <p>Мы постоянно работаем над улучшением доступности нашего веб-сайта. 
                Если вы обнаружите какие-либо проблемы с доступностью, 
                пожалуйста, <a href="/contact">свяжитесь с нами</a>.</p>
                
                <div class="important-note" aria-label="Важное уведомление">
                    <h4>Важно знать</h4>
                    <p>Мы регулярно тестируем наш сайт с помощью автоматизированных инструментов 
                    и приглашаем пользователей с ограниченными возможностями 
                    к участию в тестировании.</p>
                </div>
            </section>
            
            <section aria-labelledby="contact-form">
                <h3 id="contact-form">Свяжитесь с нами</h3>
                
                <form action="/contact" method="post">
                    <div class="form-group">
                        <label for="name">Имя: <span class="required" aria-label="обязательное поле">*</span></label>
                        <input type="text" id="name" name="name" required>
                    </div>
                    
                    <div class="form-group">
                        <label for="email">Email: <span class="required" aria-label="обязательное поле">*</span></label>
                        <input type="email" id="email" name="email" required aria-describedby="email-help">
                        <div id="email-help" class="sr-only">Введите действительный email адрес</div>
                    </div>
                    
                    <div class="form-group">
                        <label for="subject">Тема:</label>
                        <select id="subject" name="subject">
                            <option value="">Выберите тему</option>
                            <option value="accessibility">Доступность</option>
                            <option value="general">Общий вопрос</option>
                            <option value="feedback">Обратная связь</option>
                        </select>
                    </div>
                    
                    <div class="form-group">
                        <label for="message">Сообщение: <span class="required" aria-label="обязательное поле">*</span></label>
                        <textarea id="message" name="message" rows="5" required></textarea>
                    </div>
                    
                    <div class="form-group">
                        <label>
                            <input type="checkbox" name="response" value="yes">
                            Я хочу получить ответ
                        </label>
                    </div>
                    
                    <button type="submit" class="button">Отправить сообщение</button>
                </form>
            </section>
            
            <section aria-labelledby="accessibility-statement">
                <h3 id="accessibility-statement">Заявление о доступности</h3>
                
                <p>Мы стремимся обеспечить доступность контента для всех пользователей. 
                Этот сайт соответствует стандартам WCAG 2.1 уровня AA.</p>
                
                <h4>Меры по обеспечению доступности:</h4>
                <ul>
                    <li>Использование семантически правильных HTML элементов</li>
                    <li>Обеспечение достаточной контрастности текста</li>
                    <li>Предоставление альтернативного текста для изображений</li>
                    <li>Реализация клавиатурной навигации</li>
                    <li>Использование ARIA-атрибутов там, где это необходимо</li>
                </ul>
            </section>
        </div>
    </main>

    <aside role="complementary" aria-labelledby="additional-info">
        <div class="container">
            <h2 id="additional-info">Дополнительная информация</h2>
            
            <section aria-labelledby="quick-links">
                <h3 id="quick-links">Быстрые ссылки</h3>
                <ul>
                    <li><a href="/accessibility-statement">Заявление о доступности</a></li>
                    <li><a href="/keyboard-navigation">Клавиатурная навигация</a></li>
                    <li><a href="/screen-readers">Совместимость со скринридерами</a></li>
                    <li><a href="/text-size">Изменение размера текста</a></li>
                </ul>
            </section>
            
            <section aria-labelledby="resources">
                <h3 id="resources">Полезные ресурсы</h3>
                <ul>
                    <li><a href="https://www.w3.org/WAI/" target="_blank" rel="noopener">W3C WAI</a></li>
                    <li><a href="https://webaim.org/" target="_blank" rel="noopener">WebAIM</a></li>
                    <li><a href="https://www.a11yproject.com/" target="_blank" rel="noopener">The A11Y Project</a></li>
                </ul>
            </section>
        </div>
    </aside>

    <footer role="contentinfo">
        <div class="container">
            <p>&copy; 2023 Доступный веб-сайт. Все права защищены.</p>
            
            <nav aria-label="Нижняя навигация">
                <ul>
                    <li><a href="/privacy">Политика конфиденциальности</a></li>
                    <li><a href="/terms">Условия использования</a></li>
                    <li><a href="/sitemap">Карта сайта</a></li>
                    <li><a href="/accessibility">Обязательства по доступности</a></li>
                </ul>
            </nav>
            
            <p>Сделано с заботой о доступности</p>
        </div>
    </footer>
</body>
</html>
```

## Современные возможности доступности

### Доступность с использованием Web Components
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступные Web Components</title>
</head>
<body>
    <main>
        <!-- Использование доступного кастомного элемента -->
        <accessible-accordion title="Часто задаваемые вопросы">
            <accordion-item header="Как использовать этот сайт?">
                <p>Для навигации по сайту используйте клавиатуру или мышь...</p>
            </accordion-item>
            <accordion-item header="Как изменить размер текста?">
                <p>Большинство браузеров позволяют изменять размер текста...</p>
            </accordion-item>
        </accessible-accordion>
    </main>

    <script>
        // Кастомный элемент доступной аккордеон-панели
        class AccessibleAccordion extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                const title = this.getAttribute('title') || 'Аккордеон';
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .accordion {
                            border: 1px solid #ddd;
                            border-radius: 4px;
                            margin-bottom: 10px;
                        }
                        
                        .accordion-header {
                            background-color: #f5f5f5;
                            padding: 10px 15px;
                            cursor: pointer;
                            display: flex;
                            justify-content: space-between;
                            align-items: center;
                        }
                        
                        .accordion-header:focus {
                            outline: 2px solid #007acc;
                            outline-offset: 2px;
                        }
                        
                        .accordion-content {
                            padding: 15px;
                            display: none;
                        }
                        
                        .accordion-content.expanded {
                            display: block;
                        }
                        
                        .toggle-icon {
                            transition: transform 0.3s;
                        }
                        
                        .toggle-icon.expanded {
                            transform: rotate(180deg);
                        }
                    </style>
                    
                    <div class="accordion" role="region" aria-labelledby="accordion-title">
                        <div class="accordion-header" 
                             role="button" 
                             tabindex="0" 
                             aria-expanded="false"
                             aria-controls="accordion-content"
                             id="accordion-title">
                            <span>${title}</span>
                            <span class="toggle-icon" aria-hidden="true">▼</span>
                        </div>
                        <div class="accordion-content" id="accordion-content" role="region" aria-labelledby="accordion-title">
                            <slot name="items"></slot>
                        </div>
                    </div>
                `;
                
                this.header = this.shadowRoot.querySelector('.accordion-header');
                this.content = this.shadowRoot.querySelector('.accordion-content');
                this.toggleIcon = this.shadowRoot.querySelector('.toggle-icon');
                
                this.header.addEventListener('click', () => this.toggle());
                this.header.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter' || e.key === ' ') {
                        e.preventDefault();
                        this.toggle();
                    }
                });
            }
            
            toggle() {
                const isExpanded = this.header.getAttribute('aria-expanded') === 'true';
                
                this.header.setAttribute('aria-expanded', !isExpanded);
                this.content.classList.toggle('expanded', !isExpanded);
                this.toggleIcon.classList.toggle('expanded', !isExpanded);
            }
        }
        
        // Кастомный элемент для элемента аккордеона
        class AccordionItem extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                const header = this.getAttribute('header') || 'Заголовок';
                
                this.shadowRoot.innerHTML = `
                    <style>
                        .item-header {
                            background-color: #e9ecef;
                            padding: 8px 12px;
                            cursor: pointer;
                            display: flex;
                            justify-content: space-between;
                            align-items: center;
                            border-bottom: 1px solid #ddd;
                        }
                        
                        .item-header:focus {
                            outline: 2px solid #007acc;
                            outline-offset: 2px;
                        }
                        
                        .item-content {
                            padding: 12px;
                            display: none;
                        }
                        
                        .item-content.expanded {
                            display: block;
                        }
                        
                        .item-toggle {
                            transition: transform 0.3s;
                        }
                        
                        .item-toggle.expanded {
                            transform: rotate(180deg);
                        }
                    </style>
                    
                    <div class="item">
                        <div class="item-header" 
                             role="button" 
                             tabindex="0" 
                             aria-expanded="false"
                             aria-controls="item-content">
                            <span>${header}</span>
                            <span class="item-toggle" aria-hidden="true">▼</span>
                        </div>
                        <div class="item-content" id="item-content" role="region" aria-labelledby="item-header">
                            <slot></slot>
                        </div>
                    </div>
                `;
                
                this.header = this.shadowRoot.querySelector('.item-header');
                this.content = this.shadowRoot.querySelector('.item-content');
                this.toggle = this.shadowRoot.querySelector('.item-toggle');
                
                this.header.addEventListener('click', () => this.toggleItem());
                this.header.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter' || e.key === ' ') {
                        e.preventDefault();
                        this.toggleItem();
                    }
                });
            }
            
            toggleItem() {
                const isExpanded = this.header.getAttribute('aria-expanded') === 'true';
                
                this.header.setAttribute('aria-expanded', !isExpanded);
                this.content.classList.toggle('expanded', !isExpanded);
                this.toggle.classList.toggle('expanded', !isExpanded);
            }
        }
        
        // Регистрация кастомных элементов
        customElements.define('accessible-accordion', AccessibleAccordion);
        customElements.define('accordion-item', AccordionItem);
    </script>
</body>
</html>
```

### Доступность с WebSockets
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступность с WebSockets</title>
</head>
<body>
    <main>
        <h1>Чат в реальном времени</h1>
        
        <div id="chat-container" aria-live="polite" aria-atomic="false">
            <div id="messages" aria-label="Сообщения чата"></div>
            
            <form id="message-form" role="form" aria-label="Форма отправки сообщения">
                <div class="form-group">
                    <label for="message-input">Ваше сообщение:</label>
                    <input type="text" 
                           id="message-input" 
                           name="message" 
                           aria-describedby="message-help"
                           required>
                    <div id="message-help" class="help-text">Введите сообщение и нажмите Enter для отправки</div>
                </div>
                
                <button type="submit" aria-describedby="send-help">Отправить</button>
                <div id="send-help" class="sr-only">Отправить сообщение в чат</div>
            </form>
        </div>
        
        <div id="status" aria-live="polite" aria-atomic="true"></div>
    </main>

    <script>
        // Подключение к WebSocket серверу
        const ws = new WebSocket('ws://localhost:8080/chat');
        const messagesDiv = document.getElementById('messages');
        const statusDiv = document.getElementById('status');
        const messageForm = document.getElementById('message-form');
        const messageInput = document.getElementById('message-input');
        
        ws.onopen = function(event) {
            statusDiv.textContent = 'Соединение с чатом установлено';
            announceToScreenReader('Соединение с чатом установлено');
        };
        
        ws.onclose = function(event) {
            statusDiv.textContent = 'Соединение с чатом закрыто';
            announceToScreenReader('Соединение с чатом закрыто');
        };
        
        ws.onmessage = function(event) {
            const messageData = JSON.parse(event.data);
            addMessage(messageData);
        };
        
        messageForm.addEventListener('submit', function(e) {
            e.preventDefault();
            
            if (messageInput.value.trim()) {
                const message = {
                    text: messageInput.value,
                    timestamp: new Date().toISOString(),
                    user: 'current-user'
                };
                
                ws.send(JSON.stringify(message));
                messageInput.value = '';
            }
        });
        
        function addMessage(messageData) {
            const messageElement = document.createElement('div');
            messageElement.className = 'message';
            messageElement.setAttribute('role', 'log');
            messageElement.setAttribute('aria-label', `Сообщение от ${messageData.user}`);
            
            const time = new Date(messageData.timestamp).toLocaleTimeString();
            messageElement.innerHTML = `
                <strong>${messageData.user}</strong> 
                <small>(${time})</small>
                <p>${escapeHtml(messageData.text)}</p>
            `;
            
            messagesDiv.appendChild(messageElement);
            
            // Прокрутка к последнему сообщению
            messageElement.scrollIntoView({ behavior: 'smooth' });
            
            // Объявление нового сообщения для скринридера
            announceToScreenReader(`Новое сообщение от ${messageData.user}: ${messageData.text}`);
        }
        
        function announceToScreenReader(message) {
            const announcement = document.createElement('div');
            announcement.setAttribute('aria-live', 'assertive');
            announcement.setAttribute('aria-atomic', 'true');
            announcement.className = 'sr-only';
            announcement.textContent = message;
            
            document.body.appendChild(announcement);
            
            // Удаляем элемент через 10 секунд
            setTimeout(() => {
                if (document.body.contains(announcement)) {
                    document.body.removeChild(announcement);
                }
            }, 10000);
        }
        
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
    </script>
    
    <style>
        #chat-container {
            max-width: 600px;
            margin: 0 auto;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
        }
        
        .message {
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 4px;
            background-color: #f9f9f9;
        }
        
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
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input[type="text"] {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }
        
        input[type="text"]:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
        
        button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
    </style>
</body>
</html>
```

### Доступность с IndexedDB
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Доступность с IndexedDB</title>
</head>
<body>
    <main>
        <h1>Заметки (работают оффлайн)</h1>
        
        <div id="app-status" aria-live="polite" aria-atomic="true"></div>
        
        <form id="note-form" role="form" aria-label="Форма создания заметки">
            <div class="form-group">
                <label for="note-title">Заголовок заметки:</label>
                <input type="text" 
                       id="note-title" 
                       name="title" 
                       required 
                       aria-describedby="title-help">
                <div id="title-help" class="help-text">Введите заголовок для заметки</div>
            </div>
            
            <div class="form-group">
                <label for="note-content">Содержимое заметки:</label>
                <textarea id="note-content" 
                          name="content" 
                          rows="5" 
                          required 
                          aria-describedby="content-help"></textarea>
                <div id="content-help" class="help-text">Введите содержимое заметки</div>
            </div>
            
            <button type="submit" id="save-note">Сохранить заметку</button>
        </form>
        
        <section id="notes-list" aria-labelledby="notes-title">
            <h2 id="notes-title">Сохраненные заметки</h2>
            <div id="notes-container" role="list" aria-label="Список заметок"></div>
        </section>
    </main>

    <script>
        // Открытие базы данных IndexedDB
        const request = indexedDB.open('AccessibleNotesDB', 1);
        let db;
        
        request.onerror = function(event) {
            console.error('Ошибка при открытии базы данных:', event.target.error);
            updateStatus('Ошибка при открытии базы данных. Функциональность оффлайн-заметок недоступна.', 'error');
        };
        
        request.onsuccess = function(event) {
            db = event.target.result;
            updateStatus('База данных открыта. Заметки сохраняются локально.', 'success');
            loadNotes();
        };
        
        request.onupgradeneeded = function(event) {
            db = event.target.result;
            
            // Создание хранилища для заметок
            if (!db.objectStoreNames.contains('notes')) {
                const store = db.createObjectStore('notes', { keyPath: 'id', autoIncrement: true });
                store.createIndex('title', 'title', { unique: false });
                store.createIndex('date', 'date', { unique: false });
            }
        };
        
        // Форма для создания заметки
        document.getElementById('note-form').addEventListener('submit', function(e) {
            e.preventDefault();
            
            const title = document.getElementById('note-title').value.trim();
            const content = document.getElementById('note-content').value.trim();
            
            if (title && content) {
                const note = {
                    title: title,
                    content: content,
                    date: new Date().toISOString()
                };
                
                addNote(note);
                this.reset();
            }
        });
        
        function addNote(note) {
            const transaction = db.transaction(['notes'], 'readwrite');
            const store = transaction.objectStore('notes');
            const request = store.add(note);
            
            request.onsuccess = function() {
                updateStatus('Заметка успешно сохранена.', 'success');
                loadNotes();
            };
            
            request.onerror = function() {
                updateStatus('Ошибка при сохранении заметки.', 'error');
            };
        }
        
        function loadNotes() {
            const transaction = db.transaction(['notes'], 'readonly');
            const store = transaction.objectStore('notes');
            const request = store.getAll();
            
            request.onsuccess = function() {
                displayNotes(request.result);
            };
            
            request.onerror = function() {
                updateStatus('Ошибка при загрузке заметок.', 'error');
            };
        }
        
        function displayNotes(notes) {
            const container = document.getElementById('notes-container');
            container.innerHTML = '';
            
            if (notes.length === 0) {
                container.innerHTML = '<p>Нет сохраненных заметок.</p>';
                return;
            }
            
            // Сортировка по дате (новые первыми)
            notes.sort((a, b) => new Date(b.date) - new Date(a.date));
            
            notes.forEach(note => {
                const noteElement = document.createElement('article');
                noteElement.className = 'note';
                noteElement.setAttribute('role', 'article');
                noteElement.setAttribute('aria-label', `Заметка: ${note.title}`);
                
                const date = new Date(note.date).toLocaleString('ru-RU');
                
                noteElement.innerHTML = `
                    <header>
                        <h3>${escapeHtml(note.title)}</h3>
                        <time datetime="${note.date}">${date}</time>
                    </header>
                    <div class="note-content">${escapeHtml(note.content)}</div>
                    <div class="note-actions">
                        <button type="button" onclick="deleteNote(${note.id})" aria-label="Удалить заметку ${escapeHtml(note.title)}">
                            Удалить
                        </button>
                    </div>
                `;
                
                container.appendChild(noteElement);
            });
        }
        
        function deleteNote(id) {
            if (confirm('Вы уверены, что хотите удалить эту заметку?')) {
                const transaction = db.transaction(['notes'], 'readwrite');
                const store = transaction.objectStore('notes');
                const request = store.delete(id);
                
                request.onsuccess = function() {
                    updateStatus('Заметка успешно удалена.', 'success');
                    loadNotes();
                };
                
                request.onerror = function() {
                    updateStatus('Ошибка при удалении заметки.', 'error');
                };
            }
        }
        
        function updateStatus(message, type = 'info') {
            const statusDiv = document.getElementById('app-status');
            statusDiv.textContent = message;
            statusDiv.className = `status status-${type}`;
            
            // Объявление статуса для скринридера
            announceToScreenReader(message);
            
            // Удаление сообщения через 5 секунд
            setTimeout(() => {
                if (statusDiv.textContent === message) {
                    statusDiv.textContent = '';
                    statusDiv.className = 'status';
                }
            }, 5000);
        }
        
        function announceToScreenReader(message) {
            const announcement = document.createElement('div');
            announcement.setAttribute('aria-live', 'polite');
            announcement.setAttribute('aria-atomic', 'true');
            announcement.className = 'sr-only';
            announcement.textContent = message;
            
            document.body.appendChild(announcement);
            
            setTimeout(() => {
                if (document.body.contains(announcement)) {
                    document.body.removeChild(announcement);
                }
            }, 1000);
        }
        
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
    </script>
    
    <style>
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input[type="text"], textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }
        
        input:focus, textarea:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
        
        .note {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
            background-color: #fafafa;
        }
        
        .note header {
            border-bottom: 1px solid #eee;
            padding-bottom: 10px;
            margin-bottom: 10px;
        }
        
        .note h3 {
            margin: 0 0 5px 0;
        }
        
        .note time {
            color: #666;
            font-size: 0.9em;
        }
        
        .note-actions {
            margin-top: 10px;
            text-align: right;
        }
        
        .note-actions button {
            background-color: #dc3545;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .note-actions button:hover {
            background-color: #c82333;
        }
        
        .status {
            padding: 10px;
            margin-bottom: 15px;
            border-radius: 4px;
            font-weight: bold;
        }
        
        .status-success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .status-error {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
        .status-info {
            background-color: #d1ecf1;
            color: #0c5460;
            border: 1px solid #bee5eb;
        }
        
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
        
        button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        button:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }
    </style>
</body>
</html>
```

## Практические примеры и шаблоны

### Шаблон доступной админ-панели
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Доступная админ-панель</title>
    <style>
        :root {
            --primary-color: #007acc;
            --secondary-color: #6c757d;
            --success-color: #28a745;
            --danger-color: #dc3545;
            --light-bg: #f8f9fa;
            --dark-text: #333;
            --border-color: #dee2e6;
        }
        
        body {
            font-family: Arial, sans-serif;
            font-size: 16px;
            line-height: 1.5;
            color: var(--dark-text);
            margin: 0;
            padding: 0;
            background-color: #fff;
        }
        
        .dashboard {
            display: grid;
            grid-template-columns: 250px 1fr;
            min-height: 100vh;
        }
        
        .sidebar {
            background-color: var(--light-bg);
            padding: 20px;
            border-right: 1px solid var(--border-color);
        }
        
        .main-content {
            padding: 20px;
        }
        
        .skip-link {
            position: absolute;
            top: -40px;
            left: 6px;
            background: #000;
            color: #fff;
            padding: 8px;
            text-decoration: none;
            border-radius: 4px;
            z-index: 1000;
        }
        
        .skip-link:focus {
            top: 6px;
        }
        
        .nav-list {
            list-style: none;
            padding: 0;
            margin: 0;
        }
        
        .nav-item {
            margin-bottom: 5px;
        }
        
        .nav-link {
            display: block;
            padding: 10px 15px;
            text-decoration: none;
            color: var(--dark-text);
            border-radius: 4px;
            transition: background-color 0.2s;
        }
        
        .nav-link:hover, .nav-link:focus {
            background-color: rgba(0, 122, 204, 0.1);
            outline: 2px solid var(--primary-color);
            outline-offset: -2px;
        }
        
        .nav-link[aria-current="page"] {
            background-color: var(--primary-color);
            color: white;
        }
        
        .card {
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 20px;
            background-color: white;
        }
        
        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .stat-card {
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 15px;
            text-align: center;
        }
        
        .stat-number {
            font-size: 2em;
            font-weight: bold;
            color: var(--primary-color);
            margin: 10px 0;
        }
        
        .table-container {
            overflow-x: auto;
        }
        
        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid var(--border-color);
        }
        
        th {
            background-color: var(--light-bg);
            font-weight: bold;
        }
        
        .action-buttons {
            display: flex;
            gap: 5px;
        }
        
        .btn {
            padding: 8px 12px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            background-color: white;
            cursor: pointer;
            text-decoration: none;
            display: inline-block;
            color: var(--dark-text);
        }
        
        .btn:hover, .btn:focus {
            outline: 2px solid var(--primary-color);
            outline-offset: 2px;
        }
        
        .btn-primary {
            background-color: var(--primary-color);
            color: white;
            border-color: var(--primary-color);
        }
        
        .btn-danger {
            background-color: var(--danger-color);
            color: white;
            border-color: var(--danger-color);
        }
        
        .form-group {
            margin-bottom: 20px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            font-size: 16px;
        }
        
        input:focus, select:focus, textarea:focus {
            outline: 2px solid var(--primary-color);
            outline-offset: 2px;
        }
        
        .alert {
            padding: 15px;
            border-radius: 4px;
            margin-bottom: 20px;
        }
        
        .alert-success {
            background-color: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .alert-error {
            background-color: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
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
        
        @media (max-width: 768px) {
            .dashboard {
                grid-template-columns: 1fr;
            }
            
            .sidebar {
                border-right: none;
                border-bottom: 1px solid var(--border-color);
            }
        }
    </style>
</head>
<body>
    <a class="skip-link" href="#main-content">Перейти к основному содержимому</a>
    
    <div class="dashboard">
        <aside class="sidebar" role="banner">
            <h1>Админ-панель</h1>
            
            <nav role="navigation" aria-label="Основная навигация админ-панели">
                <ul class="nav-list">
                    <li class="nav-item">
                        <a href="/admin/dashboard" class="nav-link" aria-current="page">Панель управления</a>
                    </li>
                    <li class="nav-item">
                        <a href="/admin/users" class="nav-link">Пользователи</a>
                    </li>
                    <li class="nav-item">
                        <a href="/admin/content" class="nav-link">Контент</a>
                    </li>
                    <li class="nav-item">
                        <a href="/admin/settings" class="nav-link">Настройки</a>
                    </li>
                    <li class="nav-item">
                        <a href="/admin/analytics" class="nav-link">Аналитика</a>
                    </li>
                </ul>
            </nav>
        </aside>
        
        <main id="main-content" role="main">
            <h2>Панель управления</h2>
            
            <div class="stats-grid">
                <div class="stat-card">
                    <h3>Пользователи</h3>
                    <div class="stat-number" aria-label="Количество пользователей">1,248</div>
                    <p>+12% за последний месяц</p>
                </div>
                
                <div class="stat-card">
                    <h3>Сессии</h3>
                    <div class="stat-number" aria-label="Количество сессий">5,632</div>
                    <p>+5% за последний месяц</p>
                </div>
                
                <div class="stat-card">
                    <h3>Конверсия</h3>
                    <div class="stat-number" aria-label="Процент конверсии">3.2%</div>
                    <p>+0.4% за последний месяц</p>
                </div>
            </div>
            
            <div class="card">
                <h3>Последние пользователи</h3>
                
                <div class="table-container">
                    <table aria-label="Таблица последних зарегистрированных пользователей">
                        <thead>
                            <tr>
                                <th scope="col">Имя</th>
                                <th scope="col">Email</th>
                                <th scope="col">Дата регистрации</th>
                                <th scope="col">Статус</th>
                                <th scope="col">Действия</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr>
                                <td scope="row">Иван Иванов</td>
                                <td>ivan@example.com</td>
                                <td><time datetime="2023-11-07">07.11.2023</time></td>
                                <td><span class="badge" aria-label="Активный">Активный</span></td>
                                <td class="action-buttons">
                                    <a href="/admin/users/1" class="btn" aria-label="Просмотреть пользователя Иван Иванов">Просмотр</a>
                                    <a href="/admin/users/1/edit" class="btn btn-primary" aria-label="Редактировать пользователя Иван Иванов">Редактировать</a>
                                </td>
                            </tr>
                            <tr>
                                <td scope="row">Мария Петрова</td>
                                <td>maria@example.com</td>
                                <td><time datetime="2023-11-06">06.11.2023</time></td>
                                <td><span class="badge" aria-label="Активный">Активный</span></td>
                                <td class="action-buttons">
                                    <a href="/admin/users/2" class="btn" aria-label="Просмотреть пользователя Мария Петрова">Просмотр</a>
                                    <a href="/admin/users/2/edit" class="btn btn-primary" aria-label="Редактировать пользователя Мария Петрова">Редактировать</a>
                                </td>
                            </tr>
                            <tr>
                                <td scope="row">Алексей Сидоров</td>
                                <td>alexey@example.com</td>
                                <td><time datetime="2023-11-05">05.11.2023</time></td>
                                <td><span class="badge" aria-label="Неактивный">Неактивный</span></td>
                                <td class="action-buttons">
                                    <a href="/admin/users/3" class="btn" aria-label="Просмотреть пользователя Алексей Сидоров">Просмотр</a>
                                    <a href="/admin/users/3/edit" class="btn btn-primary" aria-label="Редактировать пользователя Алексей Сидоров">Редактировать</a>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
            
            <div class="card">
                <h3>Добавить нового пользователя</h3>
                
                <form action="/admin/users/create" method="post" aria-label="Форма создания нового пользователя">
                    <div class="form-group">
                        <label for="new-user-name">Полное имя:</label>
                        <input type="text" id="new-user-name" name="name" required aria-describedby="name-help">
                        <div id="name-help" class="sr-only">Введите полное имя нового пользователя</div>
                    </div>
                    
                    <div class="form-group">
                        <label for="new-user-email">Email:</label>
                        <input type="email" id="new-user-email" name="email" required aria-describedby="email-help">
                        <div id="email-help" class="sr-only">Введите email нового пользователя</div>
                    </div>
                    
                    <div class="form-group">
                        <label for="new-user-role">Роль:</label>
                        <select id="new-user-role" name="role" required>
                            <option value="">Выберите роль</option>
                            <option value="user">Пользователь</option>
                            <option value="moderator">Модератор</option>
                            <option value="admin">Администратор</option>
                        </select>
                    </div>
                    
                    <button type="submit" class="btn btn-primary">Создать пользователя</button>
                </form>
            </div>
        </main>
    </div>
</body>
</html>
```

## Проверка и тестирование доступности

### Инструменты для автоматизированного тестирования:
1. **axe-core** - библиотека для тестирования доступности
2. **Pa11y** - инструмент командной строки для тестирования
3. **Lighthouse** - встроенный инструмент в Chrome DevTools
4. **WAVE** - веб-инструмент для оценки доступности
5. **Accessibility Insights** - расширение для браузеров

### Ручное тестирование:
1. **Тестирование с клавиатуры** - навигация только с помощью клавиатуры
2. **Тестирование со скринридером** - использование NVDA, JAWS или VoiceOver
3. **Проверка контрастности** - использование инструментов для проверки контрастности
4. **Тестирование на разных устройствах** - проверка адаптивности
5. **Тестирование с уменьшенной моторикой** - проверка размера сенсорных зон

## Лучшие практики доступности

### 1. Проектирование с учетом доступности
```html
<!-- Пример доступного дизайна с самого начала -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Доступный дизайн с проекта</title>
    <style>
        /* Палитра цветов с учетом контрастности */
        :root {
            --text-primary: #212529; /* Высокая контрастность */
            --text-secondary: #495057; /* Средняя контрастность */
            --bg-primary: #ffffff;
            --bg-secondary: #f8f9fa;
            --accent-primary: #007acc;
            --accent-secondary: #005a9e;
            --success: #28a745;
            --warning: #ffc107;
            --danger: #dc3545;
        }
        
        /* Увеличенные сенсорные зоны */
        .touch-target {
            min-height: 44px;
            min-width: 44px;
            padding: 12px;
        }
        
        /* Доступные формы */
        .accessible-form {
            max-width: 600px;
            margin: 0 auto;
        }
        
        .form-field {
            margin-bottom: 20px;
        }
        
        .form-label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: var(--text-primary);
        }
        
        .form-input {
            width: 100%;
            padding: 12px;
            border: 2px solid var(--text-secondary);
            border-radius: 4px;
            font-size: 16px;
            transition: border-color 0.2s;
        }
        
        .form-input:focus {
            outline: none;
            border-color: var(--accent-primary);
            box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.2);
        }
        
        /* Видимые индикаторы фокуса */
        .focusable {
            outline: 2px solid transparent;
            outline-offset: 2px;
            transition: outline-color 0.2s;
        }
        
        .focusable:focus {
            outline-color: var(--accent-primary);
        }
        
        /* Адаптивный типографика */
        body {
            font-family: system-ui, -apple-system, sans-serif;
            font-size: 18px; /* Минимум 16px */
            line-height: 1.5;
            color: var(--text-primary);
            background-color: var(--bg-primary);
        }
    </style>
</head>
<body>
    <header>
        <h1>Проект с доступностью в приоритете</h1>
    </header>
    
    <main>
        <section class="accessible-form">
            <h2>Контактная форма</h2>
            
            <form>
                <div class="form-field">
                    <label for="contact-name" class="form-label">Ваше имя:</label>
                    <input type="text" 
                           id="contact-name" 
                           name="name" 
                           class="form-input focusable touch-target"
                           required
                           aria-describedby="name-help">
                    <div id="name-help" class="form-help">Введите ваше полное имя</div>
                </div>
                
                <div class="form-field">
                    <label for="contact-email" class="form-label">Email:</label>
                    <input type="email" 
                           id="contact-email" 
                           name="email" 
                           class="form-input focusable touch-target"
                           required
                           aria-describedby="email-help">
                    <div id="email-help" class="form-help">Введите действительный email</div>
                </div>
                
                <div class="form-field">
                    <label for="contact-message" class="form-label">Сообщение:</label>
                    <textarea id="contact-message" 
                              name="message" 
                              rows="6"
                              class="form-input focusable touch-target"
                              required
                              aria-describedby="message-help"></textarea>
                    <div id="message-help" class="form-help">Введите ваше сообщение</div>
                </div>
                
                <button type="submit" class="focusable touch-target" style="background: var(--accent-primary); color: white; border: none; padding: 12px 24px; border-radius: 4px; cursor: pointer;">
                    Отправить
                </button>
            </form>
        </section>
    </main>
</body>
</html>
```

### 2. Постоянное улучшение доступности
```html
<!-- Страница с отчетом о доступности и планами по улучшению -->
<main>
    <h1>Отчет о доступности и планы по улучшению</h1>
    
    <section aria-labelledby="accessibility-status">
        <h2 id="accessibility-status">Текущий статус доступности</h2>
        
        <div class="status-summary">
            <h3>WCAG 2.1 Соответствие</h3>
            <ul>
                <li><strong>Уровень A:</strong> <span class="status status-passed">Соответствует</span></li>
                <li><strong>Уровень AA:</strong> <span class="status status-partial">Частично соответствует</span> (85%)</li>
                <li><strong>Уровень AAA:</strong> <span class="status status-in-progress">В процессе</span></li>
            </ul>
        </div>
        
        <div class="recent-audits">
            <h3>Недавние аудиты</h3>
            <ul>
                <li>
                    <strong>Автоматизированный аудит (axe-core):</strong> 
                    <span class="audit-result">92% соответствия</span>
                    <time datetime="2023-11-01">1 ноября 2023</time>
                </li>
                <li>
                    <strong>Ручной аудит:</strong> 
                    <span class="audit-result">88% соответствия</span>
                    <time datetime="2023-10-28">28 октября 2023</time>
                </li>
                <li>
                    <strong>Тестирование со скринридером:</strong> 
                    <span class="audit-result">90% соответствия</span>
                    <time datetime="2023-10-25">25 октября 2023</time>
                </li>
            </ul>
        </div>
    </section>
    
    <section aria-labelledby="improvement-plan">
        <h2 id="improvement-plan">План по улучшению доступности</h2>
        
        <div class="improvement-priorities">
            <h3>Приоритеты на ближайший квартал</h3>
            <ol>
                <li>
                    <strong>Клавиатурная навигация</strong>
                    <p>Улучшить порядок фокуса и добавить пропуск навигации</p>
                    <span class="priority high">Высокий приоритет</span>
                </li>
                <li>
                    <strong>Альтернативный текст</strong>
                    <p>Добавить альтернативный текст ко всем изображениям</p>
                    <span class="priority high">Высокий приоритет</span>
                </li>
                <li>
                    <strong>Контрастность</strong>
                    <p>Проверить и улучшить контрастность всех текстовых элементов</p>
                    <span class="priority medium">Средний приоритет</span>
                </li>
                <li>
                    <strong>Формы</strong>
                    <p>Улучшить разметку форм и сообщения об ошибках</p>
                    <span class="priority medium">Средний приоритет</span>
                </li>
                <li>
                    <strong>Анимации</strong>
                    <p>Добавить поддержку предпочтения уменьшенной анимации</p>
                    <span class="priority low">Низкий приоритет</span>
                </li>
            </ol>
        </div>
        
        <div class="user-feedback">
            <h3>Обратная связь пользователей</h3>
            <div class="feedback-item">
                <p>"Форма контакта не работает с VoiceOver. Трудно понять, какие поля обязательны."</p>
                <time datetime="2023-10-30">30 октября 2023</time>
                <span class="status-tag resolved">Решено</span>
            </div>
            <div class="feedback-item">
                <p>"Маленькие кнопки трудно нажимать на мобильных устройствах."</p>
                <time datetime="2023-10-28">28 октября 2023</time>
                <span class="status-tag in-progress">В работе</span>
            </div>
        </div>
    </section>
    
    <section aria-labelledby="resources">
        <h2 id="resources">Ресурсы для команды</h2>
        <ul>
            <li><a href="https://www.w3.org/WAI/WCAG21/quickref/">WCAG 2.1 Quick Reference</a></li>
            <li><a href="https://webaim.org/articles/">WebAIM Articles</a></li>
            <li><a href="https://a11yproject.com/checklist/">A11Y Project Checklist</a></li>
            <li><a href="https://www.accessibility-developer-guide.com/">Accessibility Developer Guide</a></li>
        </ul>
    </section>
</main>

<style>
    .status {
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 0.8em;
        font-weight: bold;
    }
    
    .status-passed {
        background-color: #d4edda;
        color: #155724;
    }
    
    .status-partial {
        background-color: #fff3cd;
        color: #856404;
    }
    
    .status-in-progress {
        background-color: #cce5ff;
        color: #004085;
    }
    
    .audit-result {
        display: inline-block;
        padding: 2px 6px;
        background-color: #e9ecef;
        border-radius: 3px;
    }
    
    .priority {
        display: inline-block;
        padding: 2px 6px;
        border-radius: 3px;
        font-size: 0.8em;
        color: white;
    }
    
    .priority.high { background-color: #dc3545; }
    .priority.medium { background-color: #ffc107; color: #212529; }
    .priority.low { background-color: #28a745; }
    
    .status-tag {
        display: inline-block;
        padding: 2px 6px;
        border-radius: 3px;
        font-size: 0.8em;
        color: white;
    }
    
    .status-tag.resolved { background-color: #28a745; }
    .status-tag.in-progress { background-color: #ffc107; color: #212529; }
    
    .feedback-item {
        border-left: 4px solid #007acc;
        padding: 10px;
        margin-bottom: 15px;
        background-color: #f8f9fa;
    }
</style>
```

## Заключение

Современная веб-доступность — это комплексный подход, который включает в себя не только технические аспекты, но и понимание потребностей различных групп пользователей. Создание доступных веб-сайтов и приложений требует системного подхода, начиная с этапа проектирования и заканчивая тестированием с реальными пользователями. Регулярное обучение, использование современных инструментов и ориентация на международные стандарты позволяют создавать по-настоящему инклюзивный веб-опыт для всех пользователей.

Ключевые аспекты современной доступности:
1. Использование семантически правильной разметки
2. Обеспечение клавиатурной навигации
3. Достаточная контрастность текста
4. Альтернативный текст для изображений
5. Правильное использование ARIA-атрибутов
6. Поддержка вспомогательных технологий
7. Адаптивность под различные пользовательские предпочтения
8. Регулярное тестирование и улучшение
9. Обратная связь от пользователей
10. Обучение и осведомленность команды
11. Интеграция в процесс разработки (CI/CD)
12. Мониторинг доступности в продакшене
13. Следование стандартам WCAG 2.1 и 2.2
14. Учет когнитивной доступности
15. Поддержка пользовательских настроек (prefers-reduced-motion, prefers-color-scheme)

Эти практики помогают создавать веб-ресурсы, которые доступны для всех пользователей, независимо от их возможностей или используемых технологий.

## Современные тенденции в доступности

### 1. Инклюзивный дизайн с самого начала
Современные подходы к разработке включают доступность как неотъемлемую часть процесса проектирования и разработки, а не как дополнительную функцию. Это включает:

- Раннее вовлечение пользователей с ограниченными возможностями в процесс тестирования
- Использование паттернов, которые работают для всех пользователей
- Создание компонентов, которые по умолчанию доступны
- Обучение всей команды принципам доступности

### 2. Автоматизация проверок доступности
- Интеграция инструментов доступности в CI/CD пайплайны
- Использование linter'ов для проверки разметки
- Автоматические сканирования продакшен-сайтов
- Мониторинг регрессий доступности

### 3. Персонализированный опыт
Современные веб-сайты адаптируются под индивидуальные потребности пользователей:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Персонализированный опыт</title>
    <style>
        /* Уважение к пользовательским предпочтениям */
        @media (prefers-reduced-motion: reduce) {
            .animated-element {
                animation-duration: 0.01ms !important;
                animation-iteration-count: 1 !important;
                transition-duration: 0.01ms !important;
            }
        }

        @media (prefers-contrast: high) {
            body {
                background-color: #fff;
                color: #000;
            }
            
            .button {
                border: 2px solid #000;
                background-color: #fff;
                color: #000;
            }
        }

        @media (prefers-color-scheme: dark) {
            body {
                background-color: #121212;
                color: #e0e0e0;
            }
            
            .button {
                background-color: #1e88e5;
            }
        }

        /* Доступные анимации */
        .smooth-animation {
            animation: slideIn 0.5s ease-out;
            transition: all 0.3s ease;
        }

        @keyframes slideIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        /* Увеличенные сенсорные зоны */
        .touch-target {
            min-height: 44px;
            min-width: 44px;
            padding: 12px;
        }

        /* Доступные формы */
        .form-field {
            margin-bottom: 20px;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }

        input, select, textarea {
            width: 100%;
            padding: 12px;
            border: 2px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }

        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.2);
        }

        /* Видимые индикаторы фокуса */
        .focusable {
            outline: 2px solid transparent;
            outline-offset: 2px;
        }

        .focusable:focus {
            outline-color: #007acc;
        }

        /* Адаптивный типографика */
        body {
            font-family: system-ui, -apple-system, sans-serif;
            font-size: 18px; /* Минимум 16px */
            line-height: 1.5;
        }
    </style>
</head>
<body>
    <header>
        <h1>Персонализированный веб-сайт</h1>
    </header>

    <main>
        <section aria-labelledby="preferences-title">
            <h2 id="preferences-title">Настройки доступности</h2>
            
            <form id="accessibility-preferences">
                <div class="form-field">
                    <label for="font-size">Размер шрифта:</label>
                    <select id="font-size" name="font-size" class="focusable">
                        <option value="small">Маленький</option>
                        <option value="medium" selected>Средний</option>
                        <option value="large">Большой</option>
                        <option value="x-large">Очень большой</option>
                    </select>
                </div>
                
                <div class="form-field">
                    <label>
                        <input type="checkbox" name="high-contrast" value="on" class="focusable">
                        Высокая контрастность
                    </label>
                </div>
                
                <div class="form-field">
                    <label>
                        <input type="checkbox" name="reduce-motion" value="on" class="focusable">
                        Уменьшить анимацию
                    </label>
                </div>
                
                <div class="form-field">
                    <label for="reading-speed">Скорость чтения (для вспомогательных технологий):</label>
                    <input type="range" 
                           id="reading-speed" 
                           name="reading-speed" 
                           min="0.5" 
                           max="2" 
                           step="0.1" 
                           value="1" 
                           class="focusable"
                           aria-describedby="speed-help">
                    <div id="speed-help" class="sr-only">Изменение скорости произношения текста для вспомогательных технологий</div>
                </div>
                
                <button type="submit" class="button touch-target focusable">Сохранить настройки</button>
            </form>
        </section>
    </main>

    <script>
        class PersonalizationManager {
            constructor() {
                this.preferencesForm = document.getElementById('accessibility-preferences');
                this.loadUserPreferences();
                this.setupEventListeners();
            }

            setupEventListeners() {
                this.preferencesForm.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.saveUserPreferences();
                });
                
                // Отслеживание изменений предпочтений пользователя
                window.matchMedia('(prefers-reduced-motion: reduce)').addListener(() => {
                    this.applyReduceMotionPreference();
                });
                
                window.matchMedia('(prefers-contrast: high)').addListener(() => {
                    this.applyHighContrastPreference();
                });
                
                window.matchMedia('(prefers-color-scheme: dark)').addListener(() => {
                    this.applyDarkModePreference();
                });
            }

            loadUserPreferences() {
                const savedPreferences = JSON.parse(localStorage.getItem('accessibilityPreferences')) || {};
                
                // Восстановление сохраненных настроек
                if (savedPreferences.fontSize) {
                    document.getElementById('font-size').value = savedPreferences.fontSize;
                    this.applyFontSize(savedPreferences.fontSize);
                }
                
                if (savedPreferences.highContrast) {
                    document.querySelector('input[name="high-contrast"]').checked = savedPreferences.highContrast;
                    this.applyHighContrast(savedPreferences.highContrast);
                }
                
                if (savedPreferences.reduceMotion) {
                    document.querySelector('input[name="reduce-motion"]').checked = savedPreferences.reduceMotion;
                    this.applyReduceMotion(savedPreferences.reduceMotion);
                }
                
                if (savedPreferences.readingSpeed) {
                    document.getElementById('reading-speed').value = savedPreferences.readingSpeed;
                    this.applyReadingSpeed(savedPreferences.readingSpeed);
                }
            }

            saveUserPreferences() {
                const formData = new FormData(this.preferencesForm);
                const preferences = {
                    fontSize: formData.get('font-size'),
                    highContrast: formData.get('high-contrast') === 'on',
                    reduceMotion: formData.get('reduce-motion') === 'on',
                    readingSpeed: parseFloat(formData.get('reading-speed'))
                };
                
                localStorage.setItem('accessibilityPreferences', JSON.stringify(preferences));
                
                // Применение настроек
                this.applyFontSize(preferences.fontSize);
                this.applyHighContrast(preferences.highContrast);
                this.applyReduceMotion(preferences.reduceMotion);
                this.applyReadingSpeed(preferences.readingSpeed);
                
                // Уведомление для скринридеров
                this.announceToScreenReader('Настройки доступности сохранены');
            }

            applyFontSize(size) {
                const sizeMap = {
                    'small': '16px',
                    'medium': '18px',
                    'large': '22px',
                    'x-large': '26px'
                };
                
                document.body.style.fontSize = sizeMap[size] || '18px';
            }

            applyHighContrast(enabled) {
                if (enabled) {
                    document.body.classList.add('high-contrast-mode');
                    document.body.style.filter = 'contrast(1.5)';
                } else {
                    document.body.classList.remove('high-contrast-mode');
                    document.body.style.filter = '';
                }
            }

            applyReduceMotion(enabled) {
                if (enabled) {
                    document.body.classList.add('reduce-motion-mode');
                    document.documentElement.style.setProperty('--animation-duration', '0.01ms');
                } else {
                    document.body.classList.remove('reduce-motion-mode');
                    document.documentElement.style.setProperty('--animation-duration', '');
                }
            }

            applyReadingSpeed(speed) {
                // В реальных приложениях это может влиять на скорость синтеза речи
                document.documentElement.style.setProperty('--reading-speed', speed);
            }

            applyReduceMotionPreference() {
                const systemPrefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
                const userPrefersReducedMotion = document.querySelector('input[name="reduce-motion"]').checked;
                
                // Если пользователь не указал свои предпочтения, использовать системные
                if (!localStorage.getItem('accessibilityPreferences')) {
                    this.applyReduceMotion(systemPrefersReducedMotion);
                }
            }

            applyHighContrastPreference() {
                const systemPrefersHighContrast = window.matchMedia('(prefers-contrast: high)').matches;
                const userPrefersHighContrast = document.querySelector('input[name="high-contrast"]').checked;
                
                if (!localStorage.getItem('accessibilityPreferences')) {
                    this.applyHighContrast(systemPrefersHighContrast);
                }
            }

            applyDarkModePreference() {
                const systemPrefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
                
                if (systemPrefersDark) {
                    document.body.classList.add('dark-mode');
                } else {
                    document.body.classList.remove('dark-mode');
                }
            }

            announceToScreenReader(message) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = message;

                document.body.appendChild(announcement);

                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 3000);
            }
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new PersonalizationManager();
        });
    </script>

    <style>
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

        .button {
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }

        .button:hover, .button:focus {
            background-color: #005a9e;
        }
    </style>
</body>
</html>
```

### 4. Современные веб-технологии и доступность
Современные веб-технологии, такие как Web Components, WebSockets, IndexedDB и другие, также должны быть доступны:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Современные технологии и доступность</title>
</head>
<body>
    <main>
        <h1>Современные веб-технологии с доступностью</h1>

        <!-- Доступный веб-компонент -->
        <accessible-data-table 
            data-source="api/users" 
            caption="Таблица пользователей системы">
        </accessible-data-table>

        <!-- Доступный чат с реальным временем -->
        <div id="realtime-chat" 
             role="log" 
             aria-label="Чат в реальном времени"
             aria-live="polite">
            <div id="messages-container"></div>
        </div>

        <form id="message-form">
            <label for="message-input">Сообщение:</label>
            <input type="text" 
                   id="message-input" 
                   name="message" 
                   aria-describedby="message-help">
            <div id="message-help" class="sr-only">Введите сообщение и нажмите Enter для отправки</div>
            <button type="submit">Отправить</button>
        </form>
    </main>

    <script>
        // Доступный веб-компонент таблицы данных
        class AccessibleDataTable extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                this.render();
                this.loadData();
            }

            static get observedAttributes() {
                return ['data-source', 'caption'];
            }

            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                        }

                        table {
                            width: 100%;
                            border-collapse: collapse;
                            margin: 20px 0;
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

                        th:focus, td:focus {
                            outline: 2px solid #007acc;
                            outline-offset: -2px;
                        }

                        .loading {
                            text-align: center;
                            padding: 20px;
                        }

                        .error {
                            color: #d32f2f;
                            padding: 20px;
                        }
                    </style>

                    <table role="table" aria-label="${this.getAttribute('caption') || 'Таблица данных'}">
                        <caption class="sr-only">${this.getAttribute('caption') || 'Таблица данных'}</caption>
                        <thead role="rowgroup">
                            <tr role="row">
                                <th role="columnheader" scope="col" tabindex="0">ID</th>
                                <th role="columnheader" scope="col" tabindex="0">Имя</th>
                                <th role="columnheader" scope="col" tabindex="0">Email</th>
                                <th role="columnheader" scope="col" tabindex="0">Статус</th>
                            </tr>
                        </thead>
                        <tbody role="rowgroup" id="table-body">
                            <tr role="row">
                                <td role="cell" colspan="4" class="loading">Загрузка данных...</td>
                            </tr>
                        </tbody>
                    </table>
                `;
            }

            async loadData() {
                try {
                    const dataSource = this.getAttribute('data-source');
                    // В реальном приложении здесь будет запрос к API
                    const data = await this.fetchData(dataSource);
                    this.renderData(data);
                } catch (error) {
                    this.renderError(error);
                }
            }

            async fetchData(url) {
                // Симуляция API запроса
                return new Promise(resolve => {
                    setTimeout(() => {
                        resolve([
                            { id: 1, name: 'Иван Иванов', email: 'ivan@example.com', status: 'Активный' },
                            { id: 2, name: 'Мария Петрова', email: 'maria@example.com', status: 'Активный' },
                            { id: 3, name: 'Алексей Сидоров', email: 'alexey@example.com', status: 'Неактивный' }
                        ]);
                    }, 1000);
                });
            }

            renderData(data) {
                const tbody = this.shadowRoot.getElementById('table-body');
                tbody.innerHTML = '';

                data.forEach(item => {
                    const row = document.createElement('tr');
                    row.setAttribute('role', 'row');

                    const idCell = document.createElement('td');
                    idCell.setAttribute('role', 'cell');
                    idCell.textContent = item.id;

                    const nameCell = document.createElement('td');
                    nameCell.setAttribute('role', 'cell');
                    nameCell.textContent = item.name;

                    const emailCell = document.createElement('td');
                    emailCell.setAttribute('role', 'cell');
                    emailCell.textContent = item.email;

                    const statusCell = document.createElement('td');
                    statusCell.setAttribute('role', 'cell');
                    statusCell.textContent = item.status;

                    row.appendChild(idCell);
                    row.appendChild(nameCell);
                    row.appendChild(emailCell);
                    row.appendChild(statusCell);

                    tbody.appendChild(row);
                });
            }

            renderError(error) {
                const tbody = this.shadowRoot.getElementById('table-body');
                tbody.innerHTML = `
                    <tr role="row">
                        <td role="cell" colspan="4" class="error">
                            Ошибка загрузки данных: ${error.message}
                        </td>
                    </tr>
                `;
            }

            attributeChangedCallback(name, oldValue, newValue) {
                if (name === 'data-source' && oldValue !== newValue) {
                    this.loadData();
                }
            }
        }

        // Регистрация компонента
        customElements.define('accessible-data-table', AccessibleDataTable);

        // Доступный чат в реальном времени
        class AccessibleChat {
            constructor() {
                this.messagesContainer = document.getElementById('messages-container');
                this.messageForm = document.getElementById('message-form');
                this.messageInput = document.getElementById('message-input');
                
                this.setupEventListeners();
                this.connectToChat();
            }

            setupEventListeners() {
                this.messageForm.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.sendMessage();
                });

                this.messageInput.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter' && !e.shiftKey) {
                        e.preventDefault();
                        this.sendMessage();
                    }
                });
            }

            connectToChat() {
                // В реальном приложении здесь будет WebSocket соединение
                // или Server-Sent Events
                console.log('Подключение к чату...');
            }

            sendMessage() {
                const message = this.messageInput.value.trim();
                
                if (message) {
                    this.addMessage({
                        id: Date.now(),
                        text: message,
                        timestamp: new Date(),
                        sender: 'current-user'
                    });
                    
                    this.messageInput.value = '';
                    
                    // Объявление для скринридеров
                    this.announceToScreenReader(`Сообщение отправлено: ${message}`);
                }
            }

            addMessage(messageData) {
                const messageElement = document.createElement('div');
                messageElement.className = 'message';
                messageElement.setAttribute('role', 'logitem');
                messageElement.setAttribute('aria-label', `Сообщение от ${messageData.sender}: ${messageData.text}`);
                
                const time = messageData.timestamp.toLocaleTimeString();
                messageElement.innerHTML = `
                    <div class="message-header">
                        <strong>${messageData.sender === 'current-user' ? 'Вы' : messageData.sender}:</strong>
                        <time datetime="${messageData.timestamp.toISOString()}">${time}</time>
                    </div>
                    <div class="message-content">${this.escapeHtml(messageData.text)}</div>
                `;
                
                this.messagesContainer.appendChild(messageElement);
                
                // Прокрутка к последнему сообщению
                messageElement.scrollIntoView({ behavior: 'smooth' });
            }

            escapeHtml(text) {
                const div = document.createElement('div');
                div.textContent = text;
                return div.innerHTML;
            }

            announceToScreenReader(message) {
                const announcement = document.createElement('div');
                announcement.setAttribute('aria-live', 'polite');
                announcement.setAttribute('aria-atomic', 'true');
                announcement.className = 'sr-only';
                announcement.textContent = message;

                document.body.appendChild(announcement);

                setTimeout(() => {
                    if (document.body.contains(announcement)) {
                        document.body.removeChild(announcement);
                    }
                }, 3000);
            }
        }

        // Инициализация чата
        document.addEventListener('DOMContentLoaded', () => {
            new AccessibleChat();
        });
    </script>

    <style>
        #realtime-chat {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            height: 400px;
            overflow-y: auto;
            margin-bottom: 20px;
        }

        .message {
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 8px;
            background-color: #f9f9f9;
        }

        .message-header {
            display: flex;
            justify-content: space-between;
            margin-bottom: 5px;
            font-size: 0.9em;
            color: #666;
        }

        #message-form {
            display: flex;
            gap: 10px;
        }

        #message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }

        #message-input:focus {
            outline: 2px solid #007acc;
            outline-offset: 2px;
        }

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
    </style>
</body>
</html>
```

## Следующие темы
- [[HTML/Доступность/ARIA]]
- [[HTML/Формы/Доступность]]
- [[HTML/Семантика]]
- [[HTML/Доступность/Тестирование]]
- [[HTML/Доступность/Стандарты WCAG]]
- [[HTML/Доступность/Промышленные примеры]]

## Теги
#html #accessibility #web-development #aria #inclusive-design #wcag #frontend #a11y #universal-design #cognitive-accessibility #sensory-accessibility #web-components #websockets #indexeddb #best-practices #accessibility-testing #inclusive-design #modern-web #accessibility-api #web-api-integration #user-personalization #prefers-reduced-motion #prefers-contrast #prefers-color-scheme #adaptive-design #responsive-accessibility #accessibility-monitoring #ci-cd-integration #automated-accessibility #accessibility-regression #accessibility-tools #axe-core #wave #lighthouse #accessibility-insights #user-experience #cognitive-accessibility #motor-accessibility #visual-accessibility #auditory-accessibility #accessibility-compliance #wcag2.1 #wcag2.2 #wcag3.0 #accessibility-ux #accessibility-pa11y #accessibility-audit #accessibility-research #inclusive-technology #universal-design-principles
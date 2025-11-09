# HTML Валидация: Стандарты и проверка

HTML валидация - это процесс проверки HTML кода на соответствие официальным стандартам. Валидный HTML код обеспечивает совместимость с различными браузерами и улучшает доступность веб-сайтов.

## Что такое HTML валидация?

HTML валидация - это процесс проверки HTML документа на соответствие спецификации HTML. Валидный HTML код:
- Правильно отображается в различных браузерах
- Лучше воспринимается вспомогательными технологиями
- Легче поддерживается и развивается
- Повышает доверие к сайту

## Стандарты HTML

### HTML5 Спецификация

HTML5 - это последняя версия HTML, которая включает:
- Улучшенные семантические элементы
- Встроенные возможности для мультимедиа
- Улучшенные формы
- Поддержку графики и анимации

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Валидный HTML5 документ</title>
</head>
<body>
    <header>
        <h1>Основной заголовок</h1>
        <nav>
            <ul>
                <li><a href="/">Главная</a></li>
                <li><a href="/about">О нас</a></li>
            </ul>
        </nav>
    </header>
    
    <main>
        <article>
            <h2>Статья</h2>
            <p>Содержимое статьи</p>
        </article>
    </main>
    
    <footer>
        <p>&copy; 2023 Все права защищены</p>
    </footer>
</body>
</html>
```

### DOCTYPE

DOCTYPE должен быть первым элементом в HTML документе:

```html
<!-- HTML5 DOCTYPE -->
<!DOCTYPE html>
<html>
<!-- содержимое -->
</html>

<!-- НЕПРАВИЛЬНО -->
<html>
<!DOCTYPE html>  <!-- Неправильное расположение -->
<!-- содержимое -->
</html>
```

## Правила валидного HTML

### Правильная вложенность элементов

```html
<!-- ПРАВИЛЬНО -->
<p>Текст <strong>жирный текст</strong> продолжение</p>

<!-- НЕПРАВИЛЬНО -->
<p>Текст <strong>жирный текст</p> продолжение</strong>
```

### Закрытие тегов

```html
<!-- ПРАВИЛЬНО -->
<div>
    <p>Абзац текста</p>
    <img src="image.jpg" alt="Изображение">
</div>

<!-- НЕПРАВИЛЬНО -->
<div>
    <p>Абзац текста  <!-- Не закрыт тег p -->
    <img src="image.jpg" alt="Изображение">  <!-- Не закрыт тег img -->
</div>
```

### Кавычки для значений атрибутов

```html
<!-- ПРАВИЛЬНО -->
<img src="image.jpg" alt="Описание">
<div class="container" id="main">

<!-- ПРАВИЛЬНО, но не рекомендуется -->
<img src=image.jpg alt=описание>

<!-- НЕПРАВИЛЬНО -->
<img src="image.jpg alt="Описание">  <!-- Неправильное экранирование -->
```

### Уникальные ID

```html
<!-- ПРАВИЛЬНО -->
<h1 id="main-title">Заголовок</h1>
<h2 id="section-1">Раздел 1</h2>

<!-- НЕПРАВИЛЬНО -->
<h1 id="title">Заголовок 1</h1>
<h2 id="title">Заголовок 2</h2>  <!-- Повторяющийся ID -->
```

## Общие ошибки валидации

### 1. Незакрытые теги

```html
<!-- НЕПРАВИЛЬНО -->
<div>
    <p>Текст
</div>

<!-- ПРАВИЛЬНО -->
<div>
    <p>Текст</p>
</div>
```

### 2. Неправильная вложенность

```html
<!-- НЕПРАВИЛЬНО -->
<div>
    <p>Текст</div>
</p>

<!-- ПРАВИЛЬНО -->
<div>
    <p>Текст</p>
</div>
```

### 3. Неправильные атрибуты

```html
<!-- НЕПРАВИЛЬНО -->
<div class="container" class="another-class">  <!-- Повторяющийся атрибут -->
<img src="image.jpg" alt="Описание с "кавычками"">  <!-- Неправильное экранирование -->

<!-- ПРАВИЛЬНО -->
<div class="container another-class">
<img src="image.jpg" alt="Описание с &quot;кавычками&quot;">
```

### 4. Использование устаревших элементов

```html
<!-- НЕПРАВИЛЬНО (устаревшие элементы) -->
<font color="red">Красный текст</font>
<center>Центрированный текст</center>
<marquee>Бегущая строка</marquee>

<!-- ПРАВИЛЬНО -->
<span style="color: red;">Красный текст</span>
<div style="text-align: center;">Центрированный текст</div>
```

## Инструменты валидации

### W3C Markup Validator

Официальный инструмент для проверки HTML валидности:

```html
<!-- Пример валидного HTML документа для проверки -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Валидный документ</title>
</head>
<body>
    <header>
        <h1>Валидный HTML документ</h1>
    </header>
    
    <main>
        <p>Этот документ соответствует стандартам HTML5.</p>
        
        <figure>
            <img src="example.jpg" alt="Пример изображения">
            <figcaption>Подпись к изображению</figcaption>
        </figure>
    </main>
    
    <footer>
        <p>Подвал документа</p>
    </footer>
</body>
</html>
```

### HTMLHint

Инструмент для проверки HTML кода с настраиваемыми правилами:

```json
{
  "tagname-lowercase": true,
  "attr-lowercase": true,
  "attr-value-double-quotes": true,
  "id-class-value": "dash",
  "src-require": true,
  "alt-require": true
}
```

## Автоматическая проверка

### Встроенные инструменты редакторов

Большинство современных редакторов кода поддерживают проверку HTML:

```html
<!-- VS Code с расширением для HTML -->
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Пример</title>
    <!-- Редактор подсветит ошибки в реальном времени -->
</head>
<body>
    <h1>Заголовок</h1>
    <p>Текст абзаца</p>
</body>
</html>
```

### Проверка через командную строку

Использование HTML Tidy для проверки и форматирования:

```bash
# Установка HTML Tidy
npm install -g html-tidy

# Проверка HTML файла
tidy -e index.html

# Проверка с выводом в формате JSON
tidy -f /dev/stdout -asxhtml index.html
```

## Проверка доступности

Валидный HTML не всегда означает доступный HTML:

```html
<!-- Валидно, но не доступно -->
<img src="chart.png" alt="">

<!-- Валидно и доступно -->
<img src="chart.png" alt="График роста продаж за последний квартал">
```

## Пример валидного документа

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Пример валидного HTML документа">
    <title>Валидный HTML документ</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header role="banner">
        <h1>Название сайта</h1>
        <nav role="navigation" aria-label="Основная навигация">
            <ul>
                <li><a href="/" aria-current="page">Главная</a></li>
                <li><a href="/about">О нас</a></li>
                <li><a href="/contact">Контакты</a></li>
            </ul>
        </nav>
    </header>

    <main>
        <article>
            <header>
                <h2>Заголовок статьи</h2>
                <p>Опубликовано: <time datetime="2023-03-15">15 марта 2023</time></p>
            </header>
            
            <section>
                <h3>Введение</h3>
                <p>Вступительный текст статьи...</p>
            </section>
            
            <section>
                <h3>Основная часть</h3>
                <p>Основное содержимое статьи...</p>
                
                <figure>
                    <img src="diagram.png" alt="Диаграмма процесса" width="400" height="300">
                    <figcaption>Диаграмма иллюстрирует основной процесс</figcaption>
                </figure>
            </section>
            
            <footer>
                <p>Автор: <a href="/author">Иван Иванов</a></p>
            </footer>
        </article>
    </main>

    <aside aria-label="Боковая панель">
        <h2>Похожие статьи</h2>
        <ul>
            <li><a href="/article1">Статья 1</a></li>
            <li><a href="/article2">Статья 2</a></li>
        </ul>
    </aside>

    <footer role="contentinfo">
        <p>&copy; 2023 Все права защищены</p>
        <nav aria-label="Нижняя навигация">
            <ul>
                <li><a href="/privacy">Политика конфиденциальности</a></li>
                <li><a href="/sitemap">Карта сайта</a></li>
            </ul>
        </nav>
    </footer>

    <script src="script.js"></script>
</body>
</html>
```

## Интеграция валидации в процесс разработки

### CI/CD интеграция

```yaml
# .github/workflows/html-validation.yml
name: HTML Validation
on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - name: Install validator
        run: npm install -g html-validator
      - name: Validate HTML
        run: html-validator --file=index.html
```

### Pre-commit хуки

```javascript
// .lintstagedrc.js
module.exports = {
  '*.html': ['html-validate', 'git add'],
};
```

## Заключение

HTML валидация является важной частью веб-разработки, обеспечивающей совместимость, доступность и поддерживаемость веб-сайтов. Регулярная проверка HTML кода на соответствие стандартам помогает избежать многих проблем и улучшает качество веб-сайтов. Использование автоматических инструментов проверки в процессе разработки позволяет выявлять и устранять ошибки на ранних стадиях.

## Следующие темы
- [[HTML/Валидация/Инструменты проверки]]
- [[HTML/Валидация/Соответствие стандартам]]
- [[HTML/Доступность]]

## Теги
#html #validation #web-development #standards #w3c #quality-assurance #frontend
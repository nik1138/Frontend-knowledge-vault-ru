# HTML (HyperText Markup Language)

HTML (HyperText Markup Language) — это стандартный язык разметки для создания веб-страниц и веб-приложений. HTML описывает структуру веб-страниц с помощью разметки, где элементы являются строительными блоками HTML-страниц.

## Основные характеристики

### Декларативная природа
HTML является декларативным языком, который описывает *что* должно быть отображено, а не *как* это сделать.

```html
<!-- Пример HTML разметки -->
<!DOCTYPE html>
<html>
<head>
    <title>Моя страница</title>
</head>
<body>
    <h1>Заголовок</h1>
    <p>Это абзац текста.</p>
    <ul>
        <li>Первый элемент списка</li>
        <li>Второй элемент списка</li>
    </ul>
</body>
</html>
```

### Семантическая разметка
Современный HTML поддерживает семантические элементы, которые описывают смысл содержимого.

```html
<!-- Семантические элементы -->
<header>
    <h1>Заголовок сайта</h1>
    <nav>
        <ul>
            <li><a href="/">Главная</a></li>
            <li><a href="/about">О нас</a></li>
        </ul>
    </nav>
</header>

<main>
    <article>
        <h2>Заголовок статьи</h2>
        <p>Содержание статьи...</p>
    </article>
</main>

<footer>
    <p>&copy; 2023 Мой сайт</p>
</footer>
```

### Атрибуты и данные
HTML поддерживает атрибуты для предоставления дополнительной информации об элементах.

```html
<!-- Атрибуты элементов -->
<img src="image.jpg" alt="Описание изображения" width="300" height="200">
<a href="https://example.com" target="_blank" rel="noopener">Ссылка</a>
<input type="email" required placeholder="Введите email">
```

## Структура документа

### Основные элементы
Каждый HTML-документ должен иметь определенную структуру:

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

### Метаданные
Элементы `<head>` содержат метаданные о документе:

```html
<head>
    <meta charset="UTF-8">
    <meta name="description" content="Описание страницы">
    <meta name="keywords" content="ключевые, слова">
    <meta name="author" content="Автор">
    <title>Заголовок страницы</title>
</head>
```

## Преимущества HTML

1. **Универсальность** — Стандарт для веб-контента
2. **Простота** — Легко изучить и использовать
3. **Семантика** — Поддержка семантической разметки
4. **Доступность** — Встроенная поддержка доступности
5. **Совместимость** — Поддерживается всеми браузерами

## Связанные концепции

- [[Декларативное программирование]] - HTML является декларативным языком разметки
- [[Чистый код]] - Семантическая разметка способствует написанию чистого кода
- [[KISS (Keep It Simple, Stupid)]] - HTML поощряет простоту в структурировании контента
- [[DRY (Don't Repeat Yourself)]] - Семантические элементы помогают избежать дублирования структур

## В других технологиях

### В [[css]]
CSS используется для стилизации HTML-элементов:

```html
<!-- HTML -->
<div class="container">
    <h1 class="title">Заголовок</h1>
    <p class="text">Текст абзаца</p>
</div>
```

```css
/* CSS */
.container {
    max-width: 800px;
    margin: 0 auto;
}

.title {
    color: #333;
    font-size: 2rem;
}

.text {
    color: #666;
    line-height: 1.6;
}
```

### В [[js]]
JavaScript используется для добавления интерактивности к HTML:

```html
<!-- HTML -->
<button id="myButton">Нажми меня</button>
<p id="message"></p>
```

```javascript
// JavaScript
document.getElementById('myButton').addEventListener('click', function() {
    document.getElementById('message').textContent = 'Кнопка была нажата!';
});
```

### В [[react]]
React использует JSX, который комбинирует HTML и JavaScript:

```jsx
function MyComponent() {
    const [count, setCount] = useState(0);
    
    return (
        <div>
            <h1>Счетчик: {count}</h1>
            <button onClick={() => setCount(count + 1)}>
                Увеличить
            </button>
        </div>
    );
}
```

## Теги
#html #web-development #markup-language #frontend #semantics
---
aliases: [Модульный HTML, Компонентный подход]
tags: [html, architecture, modules, components]
---

# Модульность HTML

## Введение

Модульность HTML - это подход к созданию веб-страниц, при котором структура разбивается на независимые, повторно используемые компоненты. В российской веб-разработке 2025 года модульный подход становится стандартом для создания масштабируемых и поддерживаемых проектов.

## Принципы модульности

### Независимость компонентов
Каждый HTML-компонент должен быть:
- Самодостаточным
- Повторно используемым
- Независимым от других компонентов
- Тестируемым отдельно

### Единообразие
Все компоненты должны следовать единым принципам:
- Стандартизированные имена классов
- Единая структура файлов
- Последовательная архитектура

## Типичные HTML-компоненты

### Карточка (Card)
```html
<article class="card">
    <div class="card__image">
        <img src="image.jpg" alt="Описание изображения" class="card__img">
    </div>
    <div class="card__content">
        <h3 class="card__title">Заголовок карточки</h3>
        <p class="card__description">Краткое описание</p>
        <a href="#" class="card__link">Подробнее</a>
    </div>
</article>
```

### Навигационное меню
```html
<nav class="nav" aria-label="Основная навигация">
    <ul class="nav__list">
        <li class="nav__item">
            <a href="/" class="nav__link">Главная</a>
        </li>
        <li class="nav__item">
            <a href="/about" class="nav__link">О нас</a>
        </li>
        <li class="nav__item">
            <a href="/services" class="nav__link">Услуги</a>
        </li>
    </ul>
</nav>
```

### Форма
```html
<form class="form form--contact" action="/submit" method="post">
    <div class="form__group">
        <label for="name" class="form__label">Имя:</label>
        <input type="text" id="name" name="name" class="form__input" required>
    </div>
    <div class="form__group">
        <label for="email" class="form__label">Email:</label>
        <input type="email" id="email" name="email" class="form__input" required>
    </div>
    <button type="submit" class="form__submit">Отправить</button>
</form>
```

### Модальное окно
```html
<div class="modal" id="modal-example" aria-hidden="true">
    <div class="modal__overlay" tabindex="-1">
        <div class="modal__content" role="dialog" aria-modal="true">
            <button class="modal__close" aria-label="Закрыть">×</button>
            <h2 class="modal__title">Заголовок модального окна</h2>
            <div class="modal__body">
                <p>Содержимое модального окна</p>
            </div>
            <div class="modal__footer">
                <button class="btn btn--primary">Подтвердить</button>
                <button class="btn btn--secondary">Отмена</button>
            </div>
        </div>
    </div>
</div>
```

## Структура файлов компонентов

Для модульной архитектуры HTML рекомендуется следующая структура файлов:

```
components/
├── card/
│   ├── card.html
│   ├── card.css
│   └── card.js
├── navigation/
│   ├── navigation.html
│   ├── navigation.css
│   └── navigation.js
├── form/
│   ├── form.html
│   ├── form.css
│   └── form.js
└── modal/
    ├── modal.html
    ├── modal.css
    └── modal.js
```

## БЭМ-методология

Блок-Элемент-Модификатор (БЭМ) - популярная методология в российской разработке:

```html
<!-- Блок -->
<div class="button">
    <!-- Элемент блока -->
    <span class="button__text">Текст кнопки</span>
    <!-- Модификатор блока -->
    <span class="button button--primary button__text">Основная кнопка</span>
    <!-- Модифицированный элемент -->
    <span class="button__icon button__icon--left">◄</span>
</div>
```

## Повторное использование компонентов

Пример переиспользования карточки продукта:

```html
<!-- Карточка продукта - базовая -->
<article class="product-card">
    <img src="product1.jpg" alt="Название продукта" class="product-card__image">
    <div class="product-card__info">
        <h3 class="product-card__title">Название продукта</h3>
        <p class="product-card__price">1000 ₽</p>
        <button class="product-card__button">В корзину</button>
    </div>
</article>

<!-- Карточка продукта - с модификатором -->
<article class="product-card product-card--featured">
    <img src="featured-product.jpg" alt="Рекомендуемый продукт" class="product-card__image">
    <div class="product-card__info">
        <h3 class="product-card__title">Рекомендуемый продукт</h3>
        <p class="product-card__price">1500 ₽</p>
        <button class="product-card__button product-card__button--featured">Купить</button>
    </div>
</article>
```

## Системы шаблонов

Для реализации модульности часто используются системы шаблонов:

### Пример с использованием веб-компонентов:
```html
<template id="product-card-template">
    <style>
        .card { border: 1px solid #ddd; padding: 1rem; }
        .card__title { font-size: 1.2rem; margin: 0; }
    </style>
    <article class="card">
        <h3 class="card__title"></h3>
        <p class="card__price"></p>
        <button class="card__button">В корзину</button>
    </article>
</template>

<script>
class ProductCard extends HTMLElement {
    constructor() {
        super();
        const template = document.getElementById('product-card-template');
        const templateContent = template.content;
        this.appendChild(templateContent.cloneNode(true));
    }
    
    connectedCallback() {
        // Логика инициализации компонента
        this.querySelector('.card__title').textContent = this.getAttribute('title');
        this.querySelector('.card__price').textContent = this.getAttribute('price');
    }
}
customElements.define('product-card', ProductCard);
</script>

<!-- Использование компонента -->
<product-card title="Товар 1" price="1000 ₽"></product-card>
```

## Паттерны модульности

### Паттерн "Контейнер-компонент"
Разделение логики и представления:

```html
<!-- Контейнер (определяет структуру) -->
<div class="product-list">
    <h2>Список товаров</h2>
    <div class="product-list__items">
        <!-- Компоненты будут вставлены сюда -->
    </div>
</div>

<!-- Компонент (определяет отображение) -->
<article class="product-item">
    <img src="" alt="" class="product-item__image">
    <h3 class="product-item__title"></h3>
    <p class="product-item__price"></p>
</article>
```

### Паттерн "Слоты"
Для гибкости компонентов:

```html
<!-- Шаблон карточки с слотами -->
<article class="card">
    <header class="card__header">
        <slot name="header">Заголовок по умолчанию</slot>
    </header>
    <div class="card__body">
        <slot name="body">Содержимое по умолчанию</slot>
    </div>
    <footer class="card__footer">
        <slot name="footer">Футер по умолчанию</slot>
    </footer>
</article>

<!-- Использование -->
<card-component>
    <h3 slot="header">Пользовательская карточка</h3>
    <p slot="body">Произвольное содержимое</p>
    <button slot="footer">Действие</button>
</card-component>
```

## Автоматизация и сборка

Современные инструменты для работы с модульным HTML:

### Пример с Webpack и HTML-шаблонами:
```javascript
// webpack.config.js
module.exports = {
    entry: './src/index.js',
    module: {
        rules: [
            {
                test: /\.html$/,
                use: 'html-loader'
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html'
        })
    ]
};
```

### Использование препроцессоров:
```pug
// Пример с Pug
mixin card(title, image, description)
    article.card
        .card__image
            img.card__img(src=image alt=title)
        .card__content
            h3.card__title= title
            p.card__description= description
            a.card__link(href="#") Подробнее

// Использование миксина
+card("Заголовок", "image.jpg", "Описание")
```

## Тестирование модульных компонентов

Для обеспечения качества модульных компонентов:

```html
<!-- HTML-компонент с атрибутами для тестирования -->
<button 
    class="btn btn--primary" 
    data-testid="submit-button"
    aria-label="Кнопка отправки формы">
    Отправить
</button>
```

## Масштабируемость модульной архитектуры

Для крупных проектов рекомендуется:

1. Использовать систему именования компонентов
2. Внедрять систему документирования компонентов
3. Создавать библиотеку компонентов
4. Использовать системы автоматического тестирования

## Заключение

Модульность HTML позволяет создавать гибкие, масштабируемые и поддерживаемые веб-проекты. В условиях российского рынка 2025 года, когда важна скорость разработки и поддержка, модульный подход становится не просто удобным, а необходимым элементом архитектуры проекта.

Следующие темы для изучения: [[Организация-HTML-документов]], [[Структура-страницы]], [[Архитектурные-паттерны]].
---
aliases: ["Псевдоклассы структурного соответствия", "nth-child", "first-child", "last-child", "only-child"]
tags: [css, pseudo-classes, structural, selectors]
---

# Псевдоклассы структурного соответствия (nth-child, first-child, last-child, only-child и др.)

## Введение

Псевдоклассы структурного соответствия позволяют выбирать элементы на основе их позиции в дереве DOM относительно родительского элемента. Эти псевдоклассы особенно полезны для стилизации списков, таблиц и других структурированных данных без необходимости добавления специфичных классов.

## Псевдоклассы первого и последнего элемента

### :first-child

Выбирает элемент, который является первым дочерним элементом своего родителя:

```css
.list-item:first-child {
  margin-top: 0;
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
}

.table-row:first-child {
  font-weight: bold;
  background-color: #f8f9fa;
}
```

### :last-child

Выбирает элемент, который является последним дочерним элементом своего родителя:

```css
.list-item:last-child {
  margin-bottom: 0;
  border-bottom-left-radius: 8px;
  border-bottom-right-radius: 8px;
}

.menu-item:last-child {
  border-bottom: none;
}
```

### :first-of-type

Выбирает первый элемент определенного типа среди дочерних элементов родителя:

```css
p:first-of-type {
  margin-top: 0;
  font-size: 1.2em;
  font-weight: bold;
}

img:first-of-type {
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
}
```

### :last-of-type

Выбирает последний элемент определенного типа среди дочерних элементов родителя:

```css
p:last-of-type {
  margin-bottom: 0;
}

img:last-of-type {
  border-bottom-left-radius: 8px;
  border-bottom-right-radius: 8px;
}
```

## Псевдоклассы единственного элемента

### :only-child

Выбирает элемент, который является единственным дочерним элементом своего родителя:

```css
.icon:only-child {
  margin: 0 auto;
  display: block;
}

.message:only-child {
  border-radius: 8px;
}
```

### :only-of-type

Выбирает элемент, который является единственным дочерним элементом своего типа среди дочерних элементов родителя:

```css
p:only-of-type {
  text-align: center;
  font-style: italic;
}

img:only-of-type {
  display: block;
  margin: 0 auto;
}
```

## Псевдоклассы nth-семейства

### :nth-child()

Выбирает элементы на основе их позиции среди дочерних элементов родителя. Принимает аргумент в формате `an+b`:

```css
/* Выбирает каждый второй элемент, начиная с первого */
.list-item:nth-child(2n+1) {
  background-color: #f8f9fa;
}

/* То же самое, что и выше */
.list-item:nth-child(odd) {
  background-color: #f8f9fa;
}

/* Выбирает каждый второй элемент, начиная со второго */
.list-item:nth-child(2n) {
  background-color: white;
}

/* То же самое, что и выше */
.list-item:nth-child(even) {
  background-color: white;
}

/* Выбирает третий элемент */
.list-item:nth-child(3) {
  font-weight: bold;
}

/* Выбирает каждый третий элемент, начиная с первого */
.list-item:nth-child(3n) {
  border-left: 3px solid #007bff;
}

/* Выбирает первые три элемента */
.list-item:nth-child(-n+3) {
  color: #007bff;
}
```

### :nth-last-child()

То же, что и `:nth-child()`, но отсчет ведется с конца:

```css
/* Выбирает третий элемент с конца */
.list-item:nth-last-child(3) {
  font-weight: bold;
}

/* Выбирает последние три элемента */
.list-item:nth-last-child(-n+3) {
  color: #dc3545;
}
```

### :nth-of-type()

Выбирает элементы на основе их позиции среди элементов одного типа:

```css
/* Выбирает каждую вторую статью */
article:nth-of-type(2n) {
  background-color: #f8f9fa;
  padding: 20px;
  margin: 10px 0;
}

/* Выбирает первую картинку в контейнере */
img:nth-of-type(1) {
  width: 100%;
  height: auto;
}

/* Выбирает каждую третью таблицу */
table:nth-of-type(3n) {
  border: 2px solid #007bff;
}
```

### :nth-last-of-type()

То же, что и `:nth-of-type()`, но отсчет ведется с конца:

```css
/* Выбирает третью статью с конца */
article:nth-last-of-type(3) {
  border-bottom: 3px solid #28a745;
}

/* Выбирает последние две картинки */
img:nth-last-of-type(-n+2) {
  opacity: 0.7;
}
```

## Практические примеры

### Зебра-таблица

```html
<table class="zebra-table">
  <thead>
    <tr>
      <th>Имя</th>
      <th>Email</th>
      <th>Статус</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Иван Иванов</td>
      <td>ivan@example.com</td>
      <td>Активен</td>
    </tr>
    <tr>
      <td>Мария Петрова</td>
      <td>maria@example.com</td>
      <td>Неактивен</td>
    </tr>
    <!-- Другие строки -->
  </tbody>
</table>
```

```css
.zebra-table {
  width: 100%;
  border-collapse: collapse;
}

.zebra-table th,
.zebra-table td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.zebra-table th {
  background-color: #f8f9fa;
  font-weight: bold;
}

.zebra-table tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

.zebra-table tbody tr:hover {
  background-color: #f5f5f5;
}

.zebra-table tbody tr:first-child {
  border-top: 2px solid #007bff;
}

.zebra-table tbody tr:last-child {
  border-bottom: 2px solid #007bff;
}
```

### Адаптивная сетка карточек

```html
<div class="card-grid">
  <div class="card">Карточка 1</div>
  <div class="card">Карточка 2</div>
  <div class="card">Карточка 3</div>
  <div class="card">Карточка 4</div>
  <div class="card">Карточка 5</div>
  <div class="card">Карточка 6</div>
</div>
```

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
}

.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  padding: 20px;
  transition: transform 0.3s ease;
}

/* Выделяем каждую третью карточку */
.card:nth-child(3n) {
  border-top: 4px solid #007bff;
}

/* Увеличиваем первую карточку */
.card:first-child {
  grid-column: span 2;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

/* Уменьшаем последнюю карточку */
.card:last-child {
  opacity: 0.9;
  transform: scale(0.95);
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 20px rgba(0,0,0,0.15);
}
```

### Навигация с выделением активного элемента

```html
<nav class="nav-menu">
  <a href="#" class="nav-link">Главная</a>
  <a href="#" class="nav-link">О нас</a>
  <a href="#" class="nav-link">Услуги</a>
  <a href="#" class="nav-link">Контакты</a>
</nav>
```

```css
.nav-menu {
  display: flex;
  gap: 2rem;
  padding: 1rem;
  background-color: #f8f9fa;
}

.nav-link {
  text-decoration: none;
  color: #333;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.nav-link:hover {
  background-color: #e9ecef;
}

/* Выделяем первый элемент */
.nav-link:first-child {
  color: #007bff;
  font-weight: bold;
}

/* Выделяем последний элемент */
.nav-link:last-child {
  color: #dc3545;
  font-weight: bold;
}

/* Выделяем каждый второй элемент */
.nav-link:nth-child(even) {
  color: #28a745;
}

.nav-link.active {
  background-color: #007bff;
  color: white;
}
```

### Блог с различной стилизацией статей

```html
<div class="blog-container">
  <article class="blog-post">
    <h2>Заголовок статьи 1</h2>
    <p>Содержимое статьи...</p>
  </article>
  <article class="blog-post">
    <h2>Заголовок статьи 2</h2>
    <p>Содержимое статьи...</p>
  </article>
  <article class="blog-post">
    <h2>Заголовок статьи 3</h2>
    <p>Содержимое статьи...</p>
  </article>
</div>
```

```css
.blog-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.blog-post {
  margin-bottom: 30px;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  transition: transform 0.3s ease;
}

.blog-post:hover {
  transform: translateY(-3px);
}

/* Первая статья - выделяем */
.blog-post:first-of-type {
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  border-left: 4px solid #007bff;
}

/* Последняя статья - другая стилизация */
.blog-post:last-of-type {
  background-color: #fff3cd;
  border-left: 4px solid #ffc107;
}

/* Каждая четная статья - другая тема */
.blog-post:nth-of-type(even) {
  background-color: #f8f9fa;
  border-left: 4px solid #6c757d;
}

/* Третья статья - особая стилизация */
.blog-post:nth-of-type(3) {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}
```

### Сложная нумерация списков

```html
<ol class="numbered-list">
  <li>Элемент списка 1</li>
  <li>Элемент списка 2</li>
  <li>Элемент списка 3</li>
  <li>Элемент списка 4</li>
  <li>Элемент списка 5</li>
</ol>
```

```css
.numbered-list {
  counter-reset: item;
  list-style: none;
  padding-left: 0;
}

.numbered-list li {
  counter-increment: item;
  padding: 10px 10px 10px 40px;
  margin-bottom: 5px;
  position: relative;
  border-radius: 4px;
}

.numbered-list li::before {
  content: counter(item);
  position: absolute;
  left: 10px;
  top: 10px;
  background-color: #007bff;
  color: white;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8em;
}

/* Выделяем первый элемент */
.numbered-list li:first-child {
  background-color: #d4edda;
  border-left: 4px solid #28a745;
}

/* Выделяем последний элемент */
.numbered-list li:last-child {
  background-color: #f8d7da;
  border-left: 4px solid #dc3545;
}

/* Выделяем каждый третий элемент */
.numbered-list li:nth-child(3n) {
  background-color: #fff3cd;
  border-left: 4px solid #ffc107;
}
```

## Расширенные примеры

### Календарь с выделением выходных

```html
<div class="calendar-week">
  <div class="day">Пн</div>
  <div class="day">Вт</div>
  <div class="day">Ср</div>
  <div class="day">Чт</div>
  <div class="day">Пт</div>
  <div class="day weekend">Сб</div>
  <div class="day weekend">Вс</div>
</div>
```

```css
.calendar-week {
  display: flex;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.day {
  flex: 1;
  padding: 15px;
  text-align: center;
  border-right: 1px solid #ddd;
  background-color: white;
  transition: background-color 0.3s;
}

.day:last-child {
  border-right: none;
}

/* Выделяем выходные дни */
.day:nth-child(6),
.day:nth-child(7) {
  background-color: #fff3cd;
  font-weight: bold;
}

.day:hover {
  background-color: #e9ecef;
}
```

### Галерея с различной стилизацией элементов

```html
<div class="gallery">
  <div class="gallery-item">Изображение 1</div>
  <div class="gallery-item">Изображение 2</div>
  <div class="gallery-item">Изображение 3</div>
  <div class="gallery-item">Изображение 4</div>
  <div class="gallery-item">Изображение 5</div>
  <div class="gallery-item">Изображение 6</div>
</div>
```

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  padding: 20px;
}

.gallery-item {
  aspect-ratio: 1;
  background-color: #e9ecef;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  transition: transform 0.3s ease;
}

.gallery-item:hover {
  transform: scale(1.05);
}

/* Первая и последняя колонки */
.gallery-item:nth-child(3n+1) {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.gallery-item:nth-child(3n) {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
}

/* Каждый второй ряд */
.gallery-item:nth-child(n+4):nth-child(-n+6) {
  transform: rotate(5deg);
}
```

## Заключение

Псевдоклассы структурного соответствия предоставляют мощные возможности для стилизации элементов на основе их позиции в DOM. Их использование позволяет создавать сложные и адаптивные макеты без необходимости добавления дополнительных классов в HTML-разметку.

Для изучения других категорий псевдоклассов рекомендуется ознакомиться с [[Псевдоклассы форм и состояний (focus, hover, active, focus-within и др.)]] и [[Псевдоклассы пользовательских действий (target, lang, dir и др.)]].

## Ссылки

- [MDN: Псевдоклассы структурного соответствия](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_selectors#%D0%BF%D1%81%D0%B5%D0%B2%D0%B4%D0%BE%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D1%8B_%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%BD%D0%BE%D0%B3%D0%BE_%D1%81%D0%BE%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D1%81%D1%82%D0%B2%D0%B8%D1%8F)
- [CSS-Tricks: nth-child() Guide](https://css-tricks.com/how-nth-child-works/)
- [CSS-Tricks: :first-child, :last-child, and friends](https://css-tricks.com/almanac/selectors/f/first-child/)
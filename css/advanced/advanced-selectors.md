# Продвинутые возможности - Продвинутые селекторы

Продвинутые CSS-селекторы предоставляют мощные возможности для точного выбора элементов на основе сложных условий, атрибутов, позиции и состояния. Эти селекторы позволяют создавать более точные и гибкие стили без необходимости добавления дополнительных классов в HTML.

## Атрибутные селекторы

Атрибутные селекторы позволяют выбирать элементы на основе наличия или значения атрибутов:

### Базовые атрибутные селекторы

```css
/* Элементы с определенным атрибутом */
[href] {
  color: #007bff;
  text-decoration: underline;
}

/* Элементы с определенным атрибутом и значением */
[type="text"] {
  border: 1px solid #ccc;
  padding: 8px;
  border-radius: 4px;
}

/* Элементы с атрибутом, начинающимся с определенного значения */
[href^="https://"] {
  background: url(lock-icon.png) left center no-repeat;
  padding-left: 20px;
}

/* Элементы с атрибутом, заканчивающимся определенным значением */
[href$=".pdf"] {
  background: url(pdf-icon.png) left center no-repeat;
  padding-left: 20px;
}

/* Элементы с атрибутом, содержащим определенное значение */
[title*="important"] {
  font-weight: bold;
  color: #e74c3c;
}
```

### Продвинутые атрибутные селекторы

```css
/* Точный или начинающийся с определенного значения (разделенного пробелами) */
[lang~="en"] {
  quotes: "«" "»";
}

/* Точный или начинающийся с определенного значения (разделенного дефисами) */
[lang|="en"] {
  /* Подходит для языковых кодов типа en-US, en-GB */
  font-family: Georgia, serif;
}

/* Нечувствительность к регистру */
[title^="warning" i] {
  color: #e74c3c;
}

/* Сравнение с использованием регулярных выражений (экспериментально) */
[href|=^"(?i)https://"] {
  /* Поддержка ограничена */
}
```

## Псевдоклассы позиции

Псевдоклассы позиции позволяют выбирать элементы на основе их положения среди родительских элементов:

### Основные псевдоклассы позиции

```css
/* Первый и последний дочерние элементы */
li:first-child {
  font-weight: bold;
}

li:last-child {
  border-bottom: none;
}

/* Первый и последний элементы определенного типа */
p:first-of-type {
  margin-top: 0;
}

p:last-of-type {
  margin-bottom: 0;
}

/* Второй, третий и т.д. элементы */
li:nth-child(2) {
  color: #e74c3c;
}

/* Каждый третий элемент */
li:nth-child(3n) {
  background-color: #f8f9fa;
}

/* Каждый четный элемент */
tr:nth-child(even) {
  background-color: #f2f2f2;
}

/* Каждый нечетный элемент */
tr:nth-child(odd) {
  background-color: #ffffff;
}

/* Сложные выражения */
li:nth-child(3n+1) {
  /* Первый, четвертый, седьмой и т.д. элементы */
  font-weight: bold;
}

li:nth-child(n+4):nth-child(-n+6) {
  /* Только 4-й, 5-й и 6-й элементы */
  color: #3498db;
}
```

### Нестандартные псевдоклассы позиции

```css
/* Только один дочерний элемент */
p:only-child {
  text-align: center;
}

/* Только один элемент определенного типа */
p:only-of-type {
  font-style: italic;
}

/* Второй последний элемент */
li:nth-last-child(2) {
  font-weight: bold;
}

/* Второй последний элемент определенного типа */
p:nth-last-of-type(2) {
  margin-bottom: 2em;
}
```

## Псевдоклассы состояния

Псевдоклассы состояния позволяют выбирать элементы на основе их текущего состояния:

### Интерактивные псевдоклассы

```css
/* Состояния ссылок */
a:link {
  color: #007bff;
}

a:visited {
  color: #6f42c1;
}

a:hover {
  color: #0056b3;
  text-decoration: underline;
}

a:active {
  color: #004085;
}

/* Состояния фокуса */
input:focus {
  border-color: #80bdff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

/* Состояния наведения */
button:hover {
  background-color: #e9ecef;
}

/* Состояния активности */
button:active {
  transform: translateY(1px);
}
```

### Псевдоклассы форм

```css
/* Состояния формы */
input:focus {
  outline: 2px solid #007bff;
  outline-offset: 2px;
}

input:required {
  border-left: 4px solid #dc3545;
}

input:optional {
  border-left: 4px solid #28a745;
}

input:valid {
  border-color: #28a745;
}

input:invalid {
  border-color: #dc3545;
}

input:placeholder-shown {
  font-style: italic;
}

input:read-only {
  background-color: #e9ecef;
}

input:read-write {
  background-color: #ffffff;
}

/* Состояния чекбоксов и радио-кнопок */
input[type="checkbox"]:checked {
  background-color: #28a745;
}

input[type="checkbox"]:indeterminate {
  background-color: #ffc107;
}

/* Состояния селектов */
select:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

select:enabled {
  cursor: pointer;
}
```

### Псевдоклассы дерева

```css
/* Состояния дерева */
details:open {
  background-color: #f8f9fa;
}

/* Состояния рута */
:root {
  --main-color: #007bff;
}

/* Состояния пустых элементов */
p:empty {
  display: none;
}

/* Состояния цели */
:target {
  background-color: #fff3cd;
  border: 1px solid #ffeaa7;
  padding: 10px;
  border-radius: 4px;
}
```

## Псевдоклассы пользователя

```css
/* Предпочтения пользователя */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Тема пользователя */
@media (prefers-color-scheme: dark) {
  .adaptive-theme {
    background-color: #2d3748;
    color: #e2e8f0;
  }
}

/* Контрастность */
@media (prefers-contrast: high) {
  .high-contrast {
    border: 2px solid #000000;
  }
}
```

## Псевдоэлементы

Псевдоэлементы позволяют стилизовать определенные части элемента:

### Основные псевдоэлементы

```css
/* До и после содержимого элемента */
.quote::before {
  content: """;
  font-size: 1.5em;
  color: #6c757d;
}

.quote::after {
  content: """;
  font-size: 1.5em;
  color: #6c757d;
}

/* Первые буква и строка */
p::first-letter {
  font-size: 2em;
  font-weight: bold;
  float: left;
  margin-right: 5px;
}

p::first-line {
  font-weight: bold;
  color: #495057;
}

/* Выделение */
::selection {
  background-color: #007bff;
  color: white;
}
```

### Продвинутые псевдоэлементы

```css
/* Слоты веб-компонентов */
::slotted(*) {
  margin: 10px;
}

/* Ввод в полях форм */
input::placeholder {
  color: #6c757d;
  font-style: italic;
}

/* Счетчик списков */
li::marker {
  color: #e74c3c;
  font-weight: bold;
}

/* Валидационные сообщения */
input::value {
  font-weight: bold;
}

/* Рейтинговые элементы */
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  background: #007bff;
  width: 20px;
  height: 20px;
  border-radius: 50%;
}
```

## Комбинированные селекторы

### Сложные комбинации

```css
/* Соседние селекторы с условиями */
h2 + p:not([class]) {
  margin-top: 0.5em;
}

/* Дочерние селекторы с атрибутами */
ul > li[data-status="active"] {
  background-color: #d4edda;
  color: #155724;
}

/* Сложные псевдоклассы */
tr:nth-child(even):not(.excluded) {
  background-color: #f8f9fa;
}

/* Комбинация атрибутов и позиции */
div[class*="card"]:nth-of-type(odd) {
  margin-right: 10px;
}

div[class*="card"]:nth-of-type(even) {
  margin-left: 10px;
}
```

## Практические примеры

### Пример 1: Адаптивная таблица

```css
/* Стилизация таблицы с учетом позиции */
.data-table {
  width: 100%;
  border-collapse: collapse;
}

.data-table th,
.data-table td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #dee2e6;
}

/* Первая колонка */
.data-table th:first-child,
.data-table td:first-child {
  font-weight: bold;
  background-color: #f8f9fa;
}

/* Последняя колонка */
.data-table th:last-child,
.data-table td:last-child {
  text-align: right;
}

/* Заголовки */
.data-table th {
  background-color: #e9ecef;
  position: sticky;
  top: 0;
  z-index: 10;
}

/* Четные строки */
.data-table tbody tr:nth-child(even) {
  background-color: #f8f9fa;
}

/* При наведении */
.data-table tbody tr:hover {
  background-color: #e9ecef;
}

/* Последняя строка без границы */
.data-table tbody tr:last-child td {
  border-bottom: none;
}
```

### Пример 2: Навигация с активным элементом

```css
.nav-menu {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
}

.nav-item {
  position: relative;
}

.nav-link {
  display: block;
  padding: 15px 20px;
  text-decoration: none;
  color: #495057;
  transition: color 0.3s ease;
}

/* Первый элемент */
.nav-item:first-child .nav-link {
  padding-left: 0;
}

/* Последний элемент */
.nav-item:last-child .nav-link {
  padding-right: 0;
}

/* Активный элемент */
.nav-link[aria-current="page"],
.nav-link.active {
  color: #007bff;
  font-weight: bold;
}

/* Активный элемент с подчеркиванием */
.nav-link[aria-current="page"]::after,
.nav-link.active::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 20px;
  right: 20px;
  height: 3px;
  background-color: #007bff;
}

/* При наведении */
.nav-link:hover {
  color: #0056b3;
}

/* Все, кроме последнего, с правым бордером */
.nav-item:not(:last-child) {
  border-right: 1px solid #dee2e6;
}
```

### Пример 3: Галерея с фильтрами

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
}

.gallery-item {
  background-color: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.gallery-item:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
}

/* Элементы с определенными тегами */
.gallery-item[data-category="nature"] {
  border-top: 4px solid #27ae60;
}

.gallery-item[data-category="urban"] {
  border-top: 4px solid #3498db;
}

.gallery-item[data-category="portrait"] {
  border-top: 4px solid #e74c3c;
}

/* Первые 3 элемента с особым стилем */
.gallery-item:nth-child(-n+3) {
  transform: scale(1.05);
  z-index: 1;
}

/* Каждый 4-й элемент */
.gallery-item:nth-child(4n) {
  grid-column: span 2;
}

/* Пустые элементы */
.gallery-item:empty {
  display: none;
}
```

### Пример 4: Форма с валидацией

```css
.form-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.form-label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.form-input {
  width: 100%;
  padding: 10px 15px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
  box-sizing: border-box;
}

/* При фокусе */
.form-input:focus {
  outline: none;
  border-color: #80bdff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

/* Обязательные поля */
.form-input:required {
  border-left: 4px solid #dc3545;
}

/* Валидные поля */
.form-input:valid {
  border-color: #28a745;
}

/* Невалидные поля */
.form-input:invalid:not(:placeholder-shown) {
  border-color: #dc3545;
  box-shadow: 0 0 0 0.2rem rgba(220, 53, 69, 0.25);
}

/* Поля с плейсхолдером */
.form-input:placeholder-shown {
  font-style: italic;
  color: #6c757d;
}

/* Текст ошибки */
.form-input:invalid:not(:placeholder-shown):not(:focus) + .error-message {
  display: block;
  color: #dc3545;
  font-size: 0.875em;
  margin-top: 5px;
}

.error-message {
  display: none;
}

/* Кнопка отправки */
.form-submit {
  width: 100%;
  padding: 12px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.form-submit:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}

.form-submit:not(:disabled):hover {
  background-color: #0056b3;
}
```

## Лучшие практики

### 1. Оптимизация производительности

```css
/* Избегайте чрезмерной специфичности */
/* Плохо */
div.container ul.navigation li.item a.link:hover { }

/* Лучше */
.nav-link:hover { }

/* Используйте простые селекторы */
.nav-item:nth-child(odd) { }
```

### 2. Читаемость и поддержка

```css
/* Комментируйте сложные селекторы */
/* Выбор каждого четвертого элемента, начиная с третьего */
.collection-item:nth-child(4n+3) {
  background-color: #f8f9fa;
}

/* Используйте осмысленные имена классов с продвинутыми селекторами */
.card-grid .card:nth-child(3n+1) {
  clear: left;
}
```

### 3. Совместимость

```css
/* Обеспечьте fallback для старых браузеров */
.featured-item {
  /* Fallback для старых браузеров */
  background-color: #f8f9fa;
  
  /* Современный селектор */
  background-color: color-mod(var(--primary-color) a(10%));
}
```

## Современные возможности

### CSS Selectors 4

```css
/* :is() псевдокласс для упрощения сложных селекторов */
:is(h1, h2, h3):not(.excluded) {
  margin-bottom: 0.5em;
}

/* :where() псевдокласс с нулевой специфичностью */
:where(.card, .panel, .box) {
  border: 1px solid #dee2e6;
  border-radius: 4px;
  padding: 1rem;
}

/* :has() псевдокласс (экспериментально) */
article:has(h2.warning) {
  border-left: 4px solid #ffc107;
}

/* Псевдоклассы для списков */
li:defined {
  /* Применяется к элементам, которые определены как веб-компоненты */
}
```

## Заключение

Продвинутые CSS-селекторы предоставляют мощные возможности для точного выбора и стилизации элементов. Правильное использование этих селекторов позволяет создавать более гибкие и поддерживаемые стили, уменьшая необходимость в добавлении дополнительных классов в HTML. Важно понимать специфичность селекторов и учитывать производительность при их использовании.

#programming #css #selectors #advanced #web-development #frontend
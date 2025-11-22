---
aliases: ["Селекторы атрибутов", "Синтаксис селекторов атрибутов", "Применение селекторов атрибутов"]
tags: [css, selectors, attributes, advanced]
---

# Селекторы атрибутов: полное руководство по синтаксису и применению

## Введение

Селекторы атрибутов позволяют выбирать HTML-элементы на основе наличия или значений их атрибутов. Это мощный инструмент для точной настройки стилей без необходимости добавления дополнительных классов или идентификаторов в HTML-разметку.

## Основной синтаксис

Селекторы атрибутов записываются в квадратных скобках `[]` после селектора элемента:

```css
/* Выбирает все элементы с атрибутом title */
[title] {
  color: #007bff;
}

/* Выбирает все элементы с атрибутом href */
[href] {
  text-decoration: underline;
}

/* Выбирает все элементы img с атрибутом alt */
img[alt] {
  border: 1px solid #ddd;
}
```

## Типы селекторов атрибутов

### 1. Простое наличие атрибута `[attr]`

Выбирает элементы, у которых есть определенный атрибут, независимо от его значения:

```css
/* Выбирает все элементы с атрибутом title */
[title] {
  border-bottom: 1px dotted #007bff;
}

/* Выбирает все ссылки с атрибутом href */
a[href] {
  color: #007bff;
}

/* Выбирает все изображения с атрибутом alt */
img[alt] {
  margin: 5px;
  padding: 5px;
  background-color: #f8f9fa;
}
```

### 2. Точное совпадение `[attr="value"]`

Выбирает элементы, у которых атрибут имеет точное значение:

```css
/* Выбирает ссылки с конкретным значением href */
a[href="https://example.com"] {
  color: #28a745;
  font-weight: bold;
}

/* Выбирает элементы с определенным классом */
div[class="container"] {
  max-width: 1200px;
  margin: 0 auto;
}

/* Выбирает кнопки с определенным значением data-type */
button[data-type="primary"] {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
}
```

### 3. Совпадение с началом строки `[attr^="value"]`

Выбирает элементы, у которых атрибут начинается с определенного значения:

```css
/* Выбирает ссылки, начинающиеся с "https://" */
a[href^="https://"] {
  background-image: url("lock-icon.svg");
  background-repeat: no-repeat;
  background-position: right 5px center;
  padding-right: 25px;
}

/* Выбирает ссылки, начинающиеся с "#" (якорные ссылки) */
a[href^="#"] {
  color: #6f42c1;
}

/* Выбирает изображения, путь к которым начинается с "/images/" */
img[src^="/images/"] {
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
```

### 4. Совпадение с окончанием строки `[attr$="value"]`

Выбирает элементы, у которых атрибут заканчивается определенным значением:

```css
/* Выбирает ссылки на PDF-файлы */
a[href$=".pdf"] {
  background-image: url("pdf-icon.svg");
  background-repeat: no-repeat;
  background-position: left center;
  padding-left: 25px;
}

/* Выбирает ссылки на файлы изображений */
a[href$=".jpg"],
a[href$=".png"],
a[href$=".gif"] {
  border: 1px solid #ddd;
  padding: 5px;
}

/* Выбирает email-адреса */
a[href^="mailto:"] {
  color: #dc3545;
}
```

### 5. Совпадение с подстрокой `[attr*="value"]`

Выбирает элементы, у которых атрибут содержит определенное значение:

```css
/* Выбирает ссылки, содержащие "example.com" */
a[href*="example.com"] {
  color: #28a745;
}

/* Выбирает элементы с атрибутом class, содержащим "btn" */
[class*="btn"] {
  display: inline-block;
  padding: 8px 16px;
  border-radius: 4px;
  text-decoration: none;
}

/* Выбирает ссылки на внешние ресурсы */
a[href*="//"]:not([href*="yoursite.com"]) {
  background-image: url("external-link-icon.svg");
  background-repeat: no-repeat;
  background-position: right 5px center;
  padding-right: 20px;
}
```

### 6. Совпадение со словом `[attr~="value"]`

Выбирает элементы, у которых атрибут содержит определенное слово (разделенное пробелами):

```css
/* Выбирает элементы с классом "featured" среди других классов */
div[class~="featured"] {
  border: 2px solid #ffc107;
  background-color: #fff3cd;
}

/* Элемент с class="main featured sidebar" будет выбран */
/* Элемент с class="featured-item" НЕ будет выбран */
```

### 7. Совпадение с префиксом `[attr|="value"]`

Выбирает элементы, у которых атрибут начинается с определенного значения, за которым следует дефис или ничего:

```css
/* Выбирает элементы с языком "en" (например, "en", "en-US", "en-GB") */
[lang|="en"] {
  font-family: "Open Sans", Arial, sans-serif;
}

/* Выбирает элементы с кодом страны */
[lang|="ru"] {
  font-family: "PT Sans", Arial, sans-serif;
}
```

## Совмещение селекторов атрибутов

Можно комбинировать несколько селекторов атрибутов:

```css
/* Элементы с несколькими атрибутами */
input[type="text"][name="username"][required] {
  border: 2px solid #28a745;
}

/* Ссылки с определенным протоколом и доменом */
a[href^="https://"][href*="secure-site.com"] {
  background-color: #d4edda;
  padding: 5px;
}

/* Комбинация с другими селекторами */
button[type="submit"][disabled] {
  opacity: 0.6;
  cursor: not-allowed;
}

form input[type="email"]:required {
  border-left: 4px solid #007bff;
}
```

## Практические примеры

### 1. Стилизация ссылок по типу

```css
/* Внешние ссылки */
a[href^="http://"]:not([href*="yoursite.com"]),
a[href^="https://"]:not([href*="yoursite.com"]) {
  background-image: url("external-icon.svg");
  background-repeat: no-repeat;
  background-position: right 5px center;
  padding-right: 20px;
  color: #6c757d;
}

/* Ссылки на электронную почту */
a[href^="mailto:"] {
  background-image: url("email-icon.svg");
  background-repeat: no-repeat;
  background-position: left center;
  padding-left: 20px;
  color: #dc3545;
}

/* Ссылки на телефон */
a[href^="tel:"] {
  background-image: url("phone-icon.svg");
  background-repeat: no-repeat;
  background-position: left center;
  padding-left: 20px;
  color: #28a745;
}

/* Ссылки на файлы */
a[href$=".pdf"] {
  background-image: url("pdf-icon.svg");
  background-repeat: no-repeat;
  background-position: left center;
  padding-left: 20px;
  color: #6f42c1;
}

a[href$=".doc"],
a[href$=".docx"] {
  background-image: url("doc-icon.svg");
  background-repeat: no-repeat;
  background-position: left center;
  padding-left: 20px;
  color: #28a745;
}
```

### 2. Стилизация форм на основе атрибутов

```css
/* Общие стили для всех input-элементов */
input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
}

/* Стили для обязательных полей */
input[required] {
  border-left: 4px solid #007bff;
}

/* Стили для разных типов input */
input[type="email"] {
  border-color: #28a745;
}

input[type="password"] {
  border-color: #dc3545;
}

input[type="url"] {
  border-color: #ffc107;
}

/* Стили для полей с определенными атрибутами */
input[readonly] {
  background-color: #e9ecef;
  cursor: not-allowed;
}

input[disabled] {
  background-color: #f8f9fa;
  opacity: 0.6;
  cursor: not-allowed;
}

/* Стили для полей с минимальной длиной */
input[minlength] {
  position: relative;
}

input[minlength]::after {
  content: attr(minlength) " символов минимум";
  font-size: 0.8em;
  color: #6c757d;
  display: block;
  margin-top: 2px;
}
```

### 3. Стилизация таблиц на основе атрибутов

```css
/* Стили для таблиц с определенными атрибутами */
table[data-sortable="true"] th {
  cursor: pointer;
  position: relative;
  padding-right: 20px;
}

table[data-sortable="true"] th::after {
  content: "↕";
  position: absolute;
  right: 5px;
  opacity: 0.5;
}

table[data-sortable="true"] th:hover::after {
  opacity: 1;
}

/* Стили для ячеек с определенными данными */
td[data-status="active"] {
  background-color: #d4edda;
  color: #155724;
}

td[data-status="pending"] {
  background-color: #fff3cd;
  color: #856404;
}

td[data-status="inactive"] {
  background-color: #f8d7da;
  color: #721c24;
}
```

### 4. Адаптивные изображения по атрибутам

```css
/* Стили для изображений с атрибутами */
img[data-fullscreen="true"] {
  cursor: zoom-in;
  transition: transform 0.3s ease;
}

img[data-fullscreen="true"]:hover {
  transform: scale(1.05);
}

/* Стили для изображений с определенными расширениями */
img[src$=".jpg"],
img[src$=".jpeg"] {
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

img[src$=".gif"] {
  border: 2px solid #ffc107;
}
```

## Совместимость и особенности

### Совместимость с браузерами

Селекторы атрибутов поддерживаются во всех современных браузерах. Поддержка в старых браузерах:

- IE7+: Полная поддержка
- IE8+: Полная поддержка
- Современные браузеры: Полная поддержка

### Особенности использования

```css
/* Регистрозависимые атрибуты */
/* HTML: <img src="image.jpg" alt="Пример"> */
img[alt="Пример"] { /* Совпадет */ }
img[alt="пример"] { /* Не совпадет */ }

/* Регистронезависимые атрибуты (с модификатором i) */
img[alt="пример" i] { /* Совпадет с "Пример", "ПРИМЕР", "пример" и т.д. */ }

/* Использование кавычек */
/* Все эти селекторы эквивалентны: */
a[href="https://example.com"]
a[href='https://example.com']
a[href=https://example.com] /* Только если значение не содержит пробелов */
```

## Производительность

Селекторы атрибутов могут быть медленнее других селекторов, особенно при сложных комбинациях. Для оптимизации:

```css
/* Неэффективно - слишком общий */
[title] { /* влияет на все элементы с title */ }

/* Эффективнее - более конкретный */
img[title] { /* влияет только на изображения с title */ }

/* Еще эффективнее - с использованием классов */
.icon[title] { /* влияет только на элементы с классом icon и атрибутом title */ }
```

## Продвинутые примеры

### 1. Интерактивная таблица с фильтрами

```html
<table class="data-table" data-filterable="true">
  <thead>
    <tr>
      <th data-type="text">Имя</th>
      <th data-type="number">Возраст</th>
      <th data-type="date">Дата регистрации</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td data-value="Иван">Иван Иванов</td>
      <td data-value="30">30 лет</td>
      <td data-value="2023-01-15">15.01.2023</td>
    </tr>
  </tbody>
</table>
```

```css
.data-table[data-filterable="true"] th {
  position: relative;
  cursor: pointer;
  user-select: none;
}

.data-table[data-filterable="true"] th::after {
  content: "↕";
  position: absolute;
  right: 8px;
  opacity: 0.5;
}

.data-table[data-filterable="true"] th:hover::after {
  opacity: 1;
}

/* Стили для разных типов данных */
th[data-type="number"]::before {
  content: "#";
  margin-right: 5px;
  color: #007bff;
}

th[data-type="date"]::before {
  content: "📅";
  margin-right: 5px;
}

/* Выделение строк на основе данных */
tr[data-status="active"] {
  background-color: #d4edda;
}

tr[data-status="pending"] {
  background-color: #fff3cd;
}

tr[data-status="inactive"] {
  background-color: #f8d7da;
}
```

### 2. Система уведомлений с атрибутами

```html
<div class="notification" data-type="success" data-persistent="false">
  Операция выполнена успешно!
</div>
<div class="notification" data-type="warning" data-persistent="true">
  Внимание: действие требует подтверждения
</div>
```

```css
.notification {
  padding: 15px;
  margin: 10px 0;
  border-radius: 4px;
  position: relative;
  padding-left: 40px;
}

.notification::before {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  font-weight: bold;
}

.notification[data-type="success"] {
  background-color: #d4edda;
  border: 1px solid #c3e6cb;
  color: #155724;
}

.notification[data-type="success"]::before {
  content: "✓";
}

.notification[data-type="warning"] {
  background-color: #fff3cd;
  border: 1px solid #ffeaa7;
  color: #856404;
}

.notification[data-type="warning"]::before {
  content: "⚠";
}

.notification[data-type="error"] {
  background-color: #f8d7da;
  border: 1px solid #f5c6cb;
  color: #721c24;
}

.notification[data-type="error"]::before {
  content: "✗";
}

.notification[data-persistent="true"] {
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
```

## Заключение

Селекторы атрибутов предоставляют мощный способ выбора и стилизации элементов на основе их атрибутов и значений. Они позволяют создавать гибкие и семантически правильные стили без необходимости добавления дополнительных классов в HTML.

Для изучения других типов селекторов рекомендуется ознакомиться с [[Комбинаторы CSS: подробный анализ и производительность]] и [[Сложные селекторы и оптимизация производительности]].

## Ссылки

- [MDN: Селекторы атрибутов](https://developer.mozilla.org/ru/docs/Web/CSS/Attribute_selectors)
- [CSS-Tricks: Attribute Selectors](https://css-tricks.com/attribute-selectors/)
- [Can I use: Attribute selectors](https://caniuse.com/css-sel3)
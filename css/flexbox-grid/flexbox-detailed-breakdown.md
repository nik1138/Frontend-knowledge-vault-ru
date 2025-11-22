---
aliases: ["Flexbox детальный разбор", "Свойства Flexbox", "Комбинации Flexbox"]
tags: [css, flexbox, layout, properties]
---

# Flexbox: детальный разбор всех свойств и их комбинаций

## Введение

Flexbox (Flexible Box Layout) - это мощная система макета CSS, предназначенная для одномерных макетов. Она позволяет эффективно распределять пространство между элементами и выравнивать их в контейнере, даже когда размеры этих элементов неизвестны или динамичны.

## Свойства контейнера (flex-container)

### display

Свойство `display` создает flex-контейнер:

```css
.flex-container {
  display: flex; /* Блочный flex-контейнер */
  /* или */
  display: inline-flex; /* Строчно-блочный flex-контейнер */
}
```

### flex-direction

Определяет направление основной оси, по которой располагаются flex-элементы:

```css
.container {
  flex-direction: row | row-reverse | column | column-reverse;
}
```

- `row` (по умолчанию): элементы располагаются слева направо
- `row-reverse`: элементы располагаются справа налево
- `column`: элементы располагаются сверху вниз
- `column-reverse`: элементы располагаются снизу вверх

### flex-wrap

Определяет, будут ли элементы переноситься на новые строки:

```css
.container {
  flex-wrap: nowrap | wrap | wrap-reverse;
}
```

- `nowrap` (по умолчанию): все элементы на одной строке
- `wrap`: элементы переносятся на новые строки сверху вниз
- `wrap-reverse`: элементы переносятся на новые строки снизу вверх

### flex-flow

Сокращенное свойство для `flex-direction` и `flex-wrap`:

```css
.container {
  flex-flow: <flex-direction> || <flex-wrap>;
  /* Примеры */
  flex-flow: row wrap;
  flex-flow: column;
}
```

### justify-content

Определяет выравнивание элементов по основной оси:

```css
.container {
  justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
}
```

- `flex-start` (по умолчанию): элементы прижаты к началу контейнера
- `flex-end`: элементы прижаты к концу контейнера
- `center`: элементы центрированы
- `space-between`: элементы равномерно распределены, первый и последний прижаты к краям
- `space-around`: элементы равномерно распределены с равными отступами
- `space-evenly`: элементы равномерно распределены с равными пространствами между ними

### align-items

Определяет выравнивание элементов по поперечной оси:

```css
.container {
  align-items: stretch | flex-start | flex-end | center | baseline;
}
```

- `stretch` (по умолчанию): элементы растягиваются по поперечной оси
- `flex-start`: элементы прижаты к началу поперечной оси
- `flex-end`: элементы прижаты к концу поперечной оси
- `center`: элементы центрированы по поперечной оси
- `baseline`: элементы выравниваются по базовой линии текста

### align-content

Определяет выравнивание строк при наличии свободного места (работает при нескольких строках):

```css
.container {
  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
}
```

## Свойства элементов (flex-items)

### order

Определяет порядок элемента внутри контейнера:

```css
.item {
  order: <integer>; /* по умолчанию 0 */
}
```

### flex-grow

Определяет, насколько элемент может расти относительно других элементов:

```css
.item {
  flex-grow: <number>; /* по умолчанию 0 */
}
```

### flex-shrink

Определяет, насколько элемент может сжиматься относительно других элементов:

```css
.item {
  flex-shrink: <number>; /* по умолчанию 1 */
}
```

### flex-basis

Определяет начальный размер элемента перед распределением свободного места:

```css
.item {
  flex-basis: <length> | auto; /* по умолчанию auto */
}
```

### flex

Сокращенное свойство для `flex-grow`, `flex-shrink` и `flex-basis`:

```css
.item {
  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ];
  /* Примеры */
  flex: 1; /* эквивалентно flex: 1 1 0% */
  flex: 0 1 auto; /* по умолчанию */
  flex: initial; /* эквивалентно flex: 0 1 auto */
  flex: auto; /* эквивалентно flex: 1 1 auto */
  flex: none; /* эквивалентно flex: 0 0 auto */
}
```

### align-self

Позволяет переопределить выравнивание для отдельного элемента:

```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

## Практические комбинации свойств

### Центрирование элемента

```css
.center-container {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### Карточки с равной высотой

```css
.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.card {
  flex: 1 1 300px; /* Растягивается, сжимается, минимальная ширина 300px */
  display: flex;
  flex-direction: column;
}

.card-content {
  flex: 1; /* Занимает все доступное пространство */
}
```

### Навигационное меню

```css
.nav-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.nav-links {
  display: flex;
  gap: 20px;
}

.nav-item {
  flex: 0 0 auto; /* Не растягивается и не сжимается */
}
```

### Сложный макет

```css
.layout-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.header {
  flex: 0 0 auto; /* Фиксированная высота */
}

.main-content {
  display: flex;
  flex: 1 1 auto; /* Занимает оставшееся пространство */
}

.sidebar {
  flex: 0 0 250px; /* Фиксированная ширина */
}

.content {
  flex: 1 1 auto; /* Занимает оставшееся пространство */
}

.footer {
  flex: 0 0 auto; /* Фиксированная высота */
}
```

## Распространенные паттерны использования

### 1. "Холодец" (Holy Grail Layout)

```css
.holy-grail {
  display: flex;
  min-height: 100vh;
  flex-direction: column;
}

.holy-grail-header {
  flex: 0 0 auto;
}

.holy-grail-body {
  display: flex;
  flex: 1;
}

.holy-grail-nav {
  flex: 0 0 200px;
  order: -1; /* Перемещаем навигацию перед содержимым */
}

.holy-grail-content {
  flex: 1;
}

.holy-grail-aside {
  flex: 0 0 200px;
}

.holy-grail-footer {
  flex: 0 0 auto;
}
```

### 2. Сетка с гибкими колонками

```css
.grid-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.grid-item {
  flex: 1 1 calc(33.333% - 14px); /* Три колонки с учетом gap */
  min-width: 250px; /* Минимальная ширина для адаптивности */
}

@media (max-width: 768px) {
  .grid-item {
    flex: 1 1 calc(50% - 10px); /* Две колонки на планшетах */
  }
}

@media (max-width: 480px) {
  .grid-item {
    flex: 1 1 100%; /* Одна колонка на мобильных */
  }
}
```

### 3. Выравнивание элементов разного размера

```css
.align-container {
  display: flex;
  align-items: center;
  height: 200px;
  border: 1px solid #ccc;
}

.item-top {
  align-self: flex-start;
}

.item-center {
  align-self: center;
}

.item-bottom {
  align-self: flex-end;
}

.item-stretch {
  align-self: stretch;
  width: 50px;
}
```

## Совместимость и поддержка

Flexbox поддерживается всеми современными браузерами. Для поддержки старых версий браузеров (IE 10-11) могут потребоваться префиксы:

```css
.flex-container {
  display: -webkit-box;      /* Старый синтаксис Safari */
  display: -webkit-flex;     /* Новый синтаксис Safari */
  display: -ms-flexbox;      /* Синтаксис IE 10 */
  display: flex;             /* Современный синтаксис */
}
```

## Заключение

Flexbox предоставляет мощные возможности для создания гибких и адаптивных макетов. Понимание всех свойств и их комбинаций позволяет создавать сложные структуры с минимальным количеством кода.

Для изучения комбинированных макетов с Grid рекомендуется ознакомиться с [[Grid: детальный разбор всех свойств и их комбинаций]] и [[Практические примеры сложных макетов с сочетанием Flex и Grid]].

## Ссылки

- [MDN: Flexbox](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_Flexible_Box_Layout)
- [CSS-Tricks: A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Can I use: Flexbox](https://caniuse.com/flexbox)
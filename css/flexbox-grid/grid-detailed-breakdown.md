---
aliases: ["Grid детальный разбор", "Свойства Grid", "Комбинации Grid"]
tags: [css, grid, layout, properties]
---

# Grid: детальный разбор всех свойств и их комбинаций

## Введение

CSS Grid Layout - это мощная двумерная система макета, позволяющая создавать сложные структуры с точным контролем над расположением элементов по вертикали и горизонтали. В отличие от Flexbox, который идеален для одномерных макетов, Grid предназначен для двумерных структур.

## Свойства контейнера (grid-container)

### display

Создает grid-контейнер:

```css
.grid-container {
  display: grid; /* Блочный grid-контейнер */
  /* или */
  display: inline-grid; /* Строчно-блочный grid-контейнер */
}
```

### grid-template-columns и grid-template-rows

Определяют размеры и количество колонок и строк:

```css
.grid-container {
  grid-template-columns: 100px 200px 1fr 2fr;
  grid-template-rows: auto 100px minmax(100px, auto) 1fr;
}
```

#### Функции и ключевые слова

- `fr`: доля свободного пространства
- `minmax(min, max)`: диапазон значений
- `auto`: размер содержимого
- `fit-content(length)`: размер содержимого, ограниченный заданным значением

### grid-template-areas

Определяет именованные области сетки:

```css
.grid-container {
  grid-template-areas: 
    "header header header"
    "sidebar main aside"
    "footer footer footer";
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

### grid-column-gap, grid-row-gap и gap

Определяют промежутки между колонками и строками:

```css
.grid-container {
  grid-column-gap: 20px;
  grid-row-gap: 10px;
  /* или */
  gap: 10px 20px; /* row-gap column-gap */
  /* или */
  gap: 15px; /* одинаковые промежутки */
}
```

### grid-auto-columns и grid-auto-rows

Определяют размеры неявных (автоматически создаваемых) колонок и строк:

```css
.grid-container {
  grid-auto-columns: 100px; /* Ширина неявных колонок */
  grid-auto-rows: minmax(100px, auto); /* Высота неявных строк */
}
```

### grid-auto-flow

Определяет, как автоматически размещаются элементы:

```css
.grid-container {
  grid-auto-flow: row | column | row dense | column dense;
}
```

- `row` (по умолчанию): элементы заполняют сетку по строкам
- `column`: элементы заполняют сетку по колонкам
- `dense`: позволяет заполнять пустые ячейки предыдущих элементов

### grid-template

Сокращенное свойство для `grid-template-columns`, `grid-template-rows` и `grid-template-areas`:

```css
.grid-container {
  grid-template: 
    [row1-start] "header header header" 80px [row1-end]
    [row2-start] "sidebar main aside" 1fr [row2-end]
    [row3-start] "footer footer footer" 50px [row3-end]
    / 150px 1fr 150px;
}
```

### grid

Сокращенное свойство для всех grid-свойств:

```css
.grid-container {
  grid: 
    "header header" 80px
    "sidebar content" 1fr
    / 200px 1fr;
}
```

## Свойства элементов (grid-items)

### grid-column-start, grid-column-end, grid-row-start, grid-row-end

Определяют линии сетки, между которыми размещается элемент:

```css
.grid-item {
  grid-column-start: 2;
  grid-column-end: 4;
  /* или */
  grid-column: 2 / 4;
  
  grid-row-start: 1;
  grid-row-end: 3;
  /* или */
  grid-row: 1 / 3;
}
```

### grid-column и grid-row

Сокращенные свойства для определения положения элемента:

```css
.grid-item {
  grid-column: 1 / 3; /* С колонки 1 до колонки 3 */
  grid-row: 2 / span 2; /* Начиная с строки 2, занимает 2 строки */
}
```

### grid-area

Сокращенное свойство для всех позиционирующих свойств:

```css
.grid-item {
  grid-area: row-start / column-start / row-end / column-end;
  /* Пример */
  grid-area: 1 / 2 / 3 / 4;
}
```

### justify-self

Выравнивание элемента по горизонтали в пределах его ячейки:

```css
.grid-item {
  justify-self: start | end | center | stretch;
}
```

### align-self

Выравнивание элемента по вертикали в пределах его ячейки:

```css
.grid-item {
  align-self: start | end | center | stretch;
}
```

### place-self

Сокращенное свойство для `align-self` и `justify-self`:

```css
.grid-item {
  place-self: center; /* align-self: center; justify-self: center; */
  place-self: center stretch; /* align-self: center; justify-self: stretch; */
}
```

## Расширенные возможности Grid

### Повторяющиеся шаблоны

```css
.grid-container {
  grid-template-columns: repeat(3, 1fr); /* 3 колонки по 1fr */
  grid-template-columns: repeat(2, 100px 1fr); /* 100px 1fr 100px 1fr */
  grid-template-columns: 100px repeat(3, 1fr) 100px; /* С фиксированными колонками по краям */
}
```

### Именованные линии

```css
.grid-container {
  grid-template-columns: [start] 1fr [middle] 1fr [end];
  grid-template-rows: [top] auto [middle] 1fr [bottom];
}

.grid-item {
  grid-column: start / end;
  grid-row: top / middle;
}
```

### Функция minmax()

```css
.grid-container {
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  /* Автоматически подстраивается под доступное пространство */
}
```

### Функция fit-content()

```css
.grid-container {
  grid-template-columns: fit-content(200px) 1fr 1fr;
  /* Первая колонка - по содержимому, но не больше 200px */
}
```

## Практические примеры

### 1. Адаптивная сетка карточек

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
}

.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  padding: 20px;
}
```

### 2. Макет с фиксированной боковой панелью

```css
.layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: 80px 1fr 60px;
  grid-template-areas:
    "sidebar header"
    "sidebar main"
    "sidebar footer";
  height: 100vh;
}

.sidebar { grid-area: sidebar; }
.header { grid-area: header; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

### 3. Сложный макет с перекрывающимися элементами

```css
.complex-layout {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(3, 200px);
  gap: 10px;
}

.item-a {
  grid-column: 1 / 3;
  grid-row: 1 / 3;
}

.item-b {
  grid-column: 3 / 5;
  grid-row: 1 / 2;
}

.item-c {
  grid-column: 3 / 4;
  grid-row: 2 / 4;
}

.item-d {
  grid-column: 4 / 5;
  grid-row: 2 / 4;
}
```

### 4. Журнальный макет

```css
.magazine-layout {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  grid-template-rows: repeat(8, 100px);
  gap: 10px;
}

.article-main {
  grid-column: 1 / 5;
  grid-row: 1 / 5;
}

.article-sidebar {
  grid-column: 5 / 7;
  grid-row: 1 / 4;
}

.article-quote {
  grid-column: 5 / 7;
  grid-row: 4 / 6;
}

.article-footer {
  grid-column: 1 / 7;
  grid-row: 6 / 9;
}
```

## Адаптивность и медиа-запросы

```css
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 20px;
}

@media (min-width: 768px) {
  .responsive-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (min-width: 1024px) {
  .responsive-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

## Выравнивание содержимого

```css
.grid-alignment {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 200px);
  gap: 20px;
  
  /* Выравнивание ячеек */
  justify-items: center; /* По горизонтали */
  align-items: center;   /* По вертикали */
  
  /* Выравнивание всей сетки */
  justify-content: center; /* По горизонтали */
  align-content: center;   /* По вертикали */
}

.item-left {
  justify-self: start;
  align-self: start;
}

.item-right {
  justify-self: end;
  align-self: end;
}

.item-center {
  justify-self: center;
  align-self: center;
}
```

## Совместимость и поддержка

CSS Grid поддерживается всеми современными браузерами. Для поддержки старых версий браузеров (IE 10-11) может потребоваться префиксы и альтернативные подходы:

```css
.grid-container {
  display: -ms-grid; /* Префикс для IE */
  display: grid;
  
  -ms-grid-columns: 1fr 1fr 1fr;
  -ms-grid-rows: auto auto;
}

.grid-item:nth-child(1) { -ms-grid-column: 1; -ms-grid-row: 1; }
.grid-item:nth-child(2) { -ms-grid-column: 2; -ms-grid-row: 1; }
.grid-item:nth-child(3) { -ms-grid-column: 3; -ms-grid-row: 1; }
```

## Заключение

CSS Grid предоставляет мощные возможности для создания сложных двумерных макетов. Его гибкость и контроль позволяют реализовать даже самые сложные дизайнерские решения с минимальным количеством кода.

Для изучения комбинированных макетов с Flexbox рекомендуется ознакомиться с [[Flexbox: детальный разбор всех свойств и их комбинаций]] и [[Практические примеры сложных макетов с сочетанием Flex и Grid]].

## Ссылки

- [MDN: CSS Grid Layout](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_Grid_Layout)
- [CSS-Tricks: A Complete Guide to Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
- [Can I use: Grid](https://caniuse.com/css-grid)
- [Grid Garden: Интерактивный тьюториал](https://cssgridgarden.com/)
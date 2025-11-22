---
aliases: [CSS Grid Specification, CSS Grid Layout, CSS Grid Levels, CSS Grid System]
tags: [css, grid, layout, specification, css-grid]
---

# CSS Grid Layout Module: Уровни 1, 2, 3 - Полное руководство

## Введение

CSS Grid Layout Module - это мощная система компоновки, позволяющая создавать двумерные макеты с элементами, расположенными в строках и столбцах. Спецификация развивалась в трех уровнях, каждый из которых добавляет новые возможности.

## CSS Grid Layout Module Level 1

Level 1 - основа системы CSS Grid, включающая базовые возможности:

### Основные свойства Grid

#### Свойства контейнера сетки

```css
.grid-container {
  display: grid;                    /* Создает блочный контекст сетки */
  display: inline-grid;             /* Создает встроенный контекст сетки */
  
  /* Определение строк и столбцов */
  grid-template-columns: 100px 1fr 2fr;    /* Ширина столбцов */
  grid-template-rows: auto 100px 1fr;      /* Высота строк */
  
  /* Явное определение сетки */
  grid-template-areas: 
    "header header header"
    "sidebar main main"
    "footer footer footer";
  
  /* Краткая запись */
  grid-template: 
    [row1-start] "header header header" 100px [row1-end]
    [row2-start] "sidebar main main" 1fr [row2-end]
    [row3-start] "footer footer footer" 50px [row3-end]
    / 150px 1fr 1fr;
  
  /* Отступы между ячейками */
  gap: 10px;                        /* Равные отступы */
  gap: 10px 20px;                   /* gap: row-gap column-gap */
  row-gap: 10px;                    /* Отступы между строками */
  column-gap: 20px;                 /* Отступы между столбцами */
  
  /* Выравнивание сетки */
  justify-items: start | end | center | stretch;  /* Выравнивание по оси X */
  align-items: start | end | center | stretch;    /* Выравнивание по оси Y */
  place-items: center;              /* Краткая запись justify-items и align-items */
  
  /* Выравнивание содержимого сетки */
  justify-content: start | end | center | stretch | space-around | space-between | space-evenly;
  align-content: start | end | center | stretch | space-around | space-between | space-evenly;
  place-content: center;            /* Краткая запись justify-content и align-content */
}
```

#### Свойства элементов сетки

```css
.grid-item {
  /* Позиционирование в сетке */
  grid-column-start: 1;             /* Начало в колонке 1 */
  grid-column-end: 3;               /* Заканчивается в колонке 3 */
  grid-row-start: 2;                /* Начало в строке 2 */
  grid-row-end: 4;                  /* Заканчивается в строке 4 */
  
  /* Краткая запись */
  grid-column: 1 / 3;               /* grid-column-start / grid-column-end */
  grid-row: 2 / 4;                  /* grid-row-start / grid-row-end */
  grid-area: 2 / 1 / 4 / 3;         /* grid-row-start / grid-column-start / grid-row-end / grid-column-end */
  
  /* Использование именованных областей */
  grid-area: header;                /* Использует именованную область */
  
  /* Выравнивание элемента в своей ячейке */
  justify-self: start | end | center | stretch;  /* Выравнивание по оси X */
  align-self: start | end | center | stretch;    /* Выравнивание по оси Y */
  place-self: center;               /* Краткая запись justify-self и align-self */
}
```

### Примеры использования Level 1

```css
/* Простой макет */
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: 80px 1fr 60px;
  grid-template-areas:
    "sidebar header aside"
    "sidebar main aside"
    "sidebar footer aside";
  gap: 10px;
  height: 100vh;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

## CSS Grid Layout Module Level 2

Level 2 добавляет поддержку Grid Areas (именованных областей), что делает макеты более читаемыми и удобными для разработки.

### Grid Areas (Именованные области)

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header header"
    "nav main aside"
    "footer footer footer";
  gap: 10px;
}

.header { grid-area: header; }
.nav { grid-area: nav; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

### Функции и ключевые слова

- `repeat()` - повторение шаблонов
- `minmax()` - определение минимальных и максимальных размеров
- `fr` - доля доступного пространства
- `auto` - размер содержимого
- `max-content`, `min-content` - максимальный/минимальный размер содержимого

```css
.responsive-grid {
  display: grid;
  grid-template-columns: 
    repeat(auto-fit, minmax(250px, 1fr));  /* Адаптивная сетка */
  grid-auto-rows: minmax(100px, auto);
  gap: 1rem;
}
```

## CSS Grid Layout Module Level 3

Level 3 продолжает развитие спецификации, добавляя новые возможности:

### Поддержка суб-сеток

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(3, 200px);
}

.subgrid-item {
  display: grid;
  grid-row: span 2;                 /* Занимает 2 строки */
  grid-column: span 3;              /* Занимает 3 колонки */
  grid-template-rows: subgrid;      /* Наследует строки родителя */
  grid-template-columns: subgrid;   /* Наследует колонки родителя */
}
```

### Новые функции и свойства

- `subgrid` - наследование сетки от родителя
- `masonry` - кирпичная раскладка
- Улучшенные возможности для многомерных сеток

```css
/* Пример использования subgrid */
.parent-grid {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  grid-template-rows: 100px 1fr 100px;
}

.child-grid {
  grid-column: 2;
  grid-row: 2;
  display: grid;
  grid-template-columns: subgrid;   /* Использует колонки родителя */
  grid-template-rows: repeat(3, 1fr);
  gap: 10px;
}
```

## Практические применения

### Адаптивные макеты

```css
.adaptive-layout {
  display: grid;
  grid-template-columns: 
    [full-start] minmax(1rem, 1fr)
    [content-start] 
      repeat(4, [col-start] minmax(min-content, 14rem) [col-end]) 
    [content-end] 
    minmax(1rem, 1fr) 
    [full-end];
  grid-auto-flow: dense;
  gap: 1rem;
}

.item {
  grid-column: span 2;              /* Занимает 2 колонки */
}

.full-width {
  grid-column: full-start / full-end;  /* Занимает всю ширину */
}
```

### Макеты для печати

```css
.print-layout {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  grid-template-rows: 
    [header] 80px 
    [content] 1fr 
    [footer] 40px;
  height: 297mm;                    /* A4 высота */
  width: 210mm;                     /* A4 ширина */
}
```

## Поддержка браузерами

- Level 1: Поддерживается во всех современных браузерах
- Level 2: Хорошая поддержка, особенно в Chrome, Firefox и Edge
- Level 3: Поддержка в процессе внедрения в браузерах

## Связанные темы

- [[CSS Flexbox Module]] - гибкая система компоновки
- [[CSS Box Model]] - модель коробки
- [[CSS Display Module]] - типы отображения элементов

## Заключение

CSS Grid Layout Module предоставляет мощный инструмент для создания сложных двумерных макетов. Понимание всех уровней спецификации позволяет эффективно использовать возможности сетки для решения различных задач компоновки.
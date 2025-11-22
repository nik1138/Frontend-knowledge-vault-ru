---
aliases: [CSS Overflow Specification, CSS Overflow Handling, CSS Overflow Properties, CSS Overflow Module]
tags: [css, overflow, layout, specification, css-overflow]
---

# CSS Overflow Module: Обработка переполнения содержимого - Полное руководство

## Введение

CSS Overflow Module определяет способы обработки содержимого, которое больше, чем доступное пространство контейнера. Модуль включает свойства, которые контролируют, как обрабатывается переполнение по осям X и Y, а также новые возможности для обработки переполнения в современных спецификациях.

## CSS Overflow Module Level 3

Level 3 определяет основные свойства обработки переполнения.

### Основные свойства overflow

```css
.overflow-container {
  /* Базовое свойство overflow */
  overflow: visible;                /* Содержимое видно за пределами контейнера */
  overflow: hidden;                 /* Скрывает переполнение */
  overflow: scroll;                 /* Показывает полосы прокрутки всегда */
  overflow: auto;                   /* Показывает полосы прокрутки при необходимости */
  
  /* Раздельные свойства для осей */
  overflow-x: visible;              /* Горизонтальное переполнение */
  overflow-y: hidden;               /* Вертикальное переполнение */
  
  /* Краткая запись */
  overflow: visible hidden;         /* overflow-x overflow-y */
  overflow: auto scroll;            /* X: auto, Y: scroll */
}
```

### Значения свойства overflow

#### visible

```css
.visible-overflow {
  width: 200px;
  height: 100px;
  overflow: visible;                /* Содержимое выходит за границы */
  border: 1px solid #ccc;
}

.visible-content {
  width: 300px;                     /* Шире контейнера */
  height: 150px;                    /* Выше контейнера */
  background-color: #f0f0f0;
}
```

#### hidden

```css
.hidden-overflow {
  width: 200px;
  height: 100px;
  overflow: hidden;                 /* Обрезает переполняющее содержимое */
  border: 1px solid #ccc;
  position: relative;
}

.hidden-content {
  width: 300px;
  height: 150px;
  background-color: #e0e0e0;
  position: absolute;
  top: 0;
  left: 0;
}
```

#### scroll

```css
.scroll-overflow {
  width: 200px;
  height: 100px;
  overflow: scroll;                 /* Всегда показывает полосы прокрутки */
  border: 1px solid #ccc;
}

.scroll-content {
  width: 400px;                     /* Значительно шире контейнера */
  height: 200px;                    /* Значительно выше контейнера */
  background: linear-gradient(to bottom, #ff0000, #0000ff);
}
```

#### auto

```css
.auto-overflow {
  width: 200px;
  height: 100px;
  overflow: auto;                   /* Показывает прокрутку при необходимости */
  border: 1px solid #ccc;
}

.auto-content {
  width: 250px;                     /* Немного шире контейнера */
  height: 120px;                    /* Немного выше контейнера */
  background-color: #d0d0d0;
}
```

### Раздельное управление осями

```css
.axis-control {
  width: 200px;
  height: 100px;
  overflow-x: hidden;               /* Скрывает горизонтальное переполнение */
  overflow-y: auto;                 /* Автоматическая вертикальная прокрутка */
  border: 1px solid #ccc;
}

.axis-content {
  width: 300px;                     /* Шире - скрыто */
  height: 150px;                    /* Выше - с прокруткой */
  background-color: #c0c0c0;
}
```

## CSS Overflow Module Level 4

Level 4 добавляет новые возможности для тонкой настройки поведения переполнения.

### Свойство overflow-block и overflow-inline

```css
.logical-overflow {
  width: 200px;
  height: 100px;
  overflow-block: auto;             /* Прокрутка в направлении блока */
  overflow-inline: hidden;          /* Скрытие в направлении строки */
  border: 1px solid #ccc;
}
```

### Свойство text-overflow

```css
.text-overflow-example {
  width: 200px;
  white-space: nowrap;              /* Не переносить строки */
  overflow: hidden;                 /* Скрывать переполнение */
  text-overflow: clip;              /* Обрезать текст */
  text-overflow: ellipsis;          /* Показывать многоточие */
  text-overflow: "…";               /* Кастомный символ */
  
  /* Для направления строки */
  text-overflow: ellipsis ellipsis; /* inline-end и block-end */
  text-overflow: clip ellipsis;     /* inline-end: clip, block-end: ellipsis */
}
```

### Свойство block-overflow (экспериментальное)

```css
.block-overflow-example {
  width: 200px;
  height: 100px;
  overflow: hidden;
  block-overflow: ellipsis;         /* Обработка переполнения в блочном направлении */
}
```

## Практические применения

### Карточки с ограниченной высотой

```css
.card {
  width: 300px;
  height: 200px;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.card-content {
  padding: 16px;
  height: 100%;
  overflow-y: auto;                 /* Прокрутка если контент высокий */
}

.card-description {
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 3;            /* Показывать только 3 строки */
  -webkit-box-orient: vertical;
  text-overflow: ellipsis;
}
```

### Навигационные меню

```css
.nav-menu {
  width: 250px;
  height: 400px;
  overflow-y: auto;                 /* Вертикальная прокрутка для длинных меню */
  background-color: #f8f9fa;
  border-right: 1px solid #dee2e6;
}

.nav-item {
  padding: 12px 16px;
  border-bottom: 1px solid #dee2e6;
}

/* Горизонтальное меню с прокруткой */
.horizontal-scroll-menu {
  display: flex;
  overflow-x: auto;                 /* Горизонтальная прокрутка */
  overflow-y: hidden;
  white-space: nowrap;
  padding: 10px 0;
}

.horizontal-scroll-menu::-webkit-scrollbar {
  height: 8px;
}

.horizontal-scroll-menu::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.horizontal-scroll-menu::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}
```

### Таблицы с фиксированной высотой

```css
.scrollable-table-container {
  height: 300px;
  overflow-y: auto;                 /* Прокрутка для длинных таблиц */
  border: 1px solid #ddd;
}

.scrollable-table {
  width: 100%;
  border-collapse: collapse;
}

.scrollable-table th {
  position: sticky;                 /* Липкие заголовки */
  top: 0;
  background-color: #f8f9fa;
  z-index: 10;
}

.scrollable-table td {
  padding: 8px;
  border-bottom: 1px solid #ddd;
}
```

### Модальные окна

```css
.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 80%;
  max-width: 600px;
  max-height: 80vh;                 /* Ограничивает высоту */
  overflow: hidden;                 /* Скрывает внешний скролл */
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.3);
}

.modal-content {
  max-height: calc(80vh - 100px);   /* Учитывает заголовок и футер */
  overflow-y: auto;                 /* Прокрутка содержимого */
  padding: 20px;
}
```

### Текстовые области

```css
.text-area {
  width: 100%;
  height: 150px;
  resize: vertical;                 /* Только вертикальное изменение размера */
  overflow: auto;                   /* Прокрутка при необходимости */
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.truncated-text {
  width: 200px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  padding: 8px;
  border: 1px solid #ddd;
}
```

## Современные возможности

### CSS Scroll Snap

```css
.snapping-container {
  overflow-x: auto;
  display: flex;
  scroll-snap-type: x mandatory;    /* Обязательные точки привязки */
}

.snapping-item {
  scroll-snap-align: start;         /* Выравнивание к началу */
  flex: 0 0 300px;
  height: 200px;
}
```

### Logical Overflow Properties

```css
.logical-directions {
  overflow-block: auto;             /* Прокрутка в логическом блочном направлении */
  overflow-inline: hidden;          /* Скрытие в логическом строчном направлении */
}
```

### Overflow в Flexbox и Grid

```css
.flex-overflow {
  display: flex;
  overflow: hidden;                 /* Контейнер скрывает переполнение */
}

.flex-item {
  flex: 1;
  min-width: 0;                     /* Позволяет элементу сжиматься */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.grid-overflow {
  display: grid;
  grid-template-columns: 1fr 2fr;
  overflow: auto;                   /* Сетка с прокруткой */
}
```

## Поддержка браузерами

- `overflow: visible, hidden, scroll, auto`: Поддерживается во всех браузерах
- `overflow-x`, `overflow-y`: Хорошая поддержка, с префиксами для старых IE
- `text-overflow`: Хорошая поддержка во всех современных браузерах
- `block-overflow`: Ограниченная поддержка, экспериментальное свойство
- `overflow-block`, `overflow-inline`: Поддержка в процессе внедрения

## Связанные темы

- [[CSS Box Model]] - модель коробки
- [[CSS Layout Modules]] - модули компоновки
- [[CSS Scroll Snap Module]] - привязка прокрутки

## Заключение

CSS Overflow Module предоставляет гибкие инструменты для управления переполнением содержимого. Понимание различных значений и свойств позволяет создавать адаптивные интерфейсы, которые корректно отображают содержимое независимо от его размера.
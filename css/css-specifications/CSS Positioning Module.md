---
aliases: [CSS Positioning Specification, CSS Position, CSS Positioning System, CSS Positioning Levels]
tags: [css, positioning, layout, specification, css-positioning]
---

# CSS Positioning Module: Absolute, Relative, Fixed, Sticky - Полное руководство

## Введение

CSS Positioning Module определяет способы позиционирования элементов в веб-документе. Модуль включает в себя различные методы позиционирования, каждый из которых имеет свои особенности и области применения.

## Статическое позиционирование (static)

Статическое позиционирование - это значение по умолчанию для всех элементов. Элементы с `position: static` следуют нормальному потоку документа.

```css
.static-element {
  position: static;                 /* Значение по умолчанию */
  /* top, right, bottom, left и z-index не работают */
}
```

## Относительное позиционирование (relative)

Относительное позиционирование смещает элемент относительно его нормального положения в потоке документа, не влияя на другие элементы.

```css
.relative-element {
  position: relative;
  top: 10px;                        /* Смещение вниз */
  left: 20px;                       /* Смещение вправо */
  z-index: 1;                       /* Уровень наложения */
}
```

### Примеры использования relative

```css
.relative-container {
  position: relative;
  width: 300px;
  height: 200px;
  border: 1px solid #ccc;
}

.offset-element {
  position: relative;
  top: 20px;
  left: 30px;
  background-color: #f0f0f0;
}
```

## Абсолютное позиционирование (absolute)

Абсолютное позиционирование извлекает элемент из нормального потока документа и позиционирует его относительно ближайшего позиционированного предка (элемента с position, отличным от static).

```css
.absolute-element {
  position: absolute;
  top: 10px;                        /* Расстояние от верхнего края контейнера */
  right: 15px;                      /* Расстояние от правого края контейнера */
  bottom: 10px;                     /* Расстояние от нижнего края контейнера */
  left: 15px;                       /* Расстояние от левого края контейнера */
  z-index: 2;                       /* Уровень наложения */
}
```

### Примеры использования absolute

```css
.absolute-container {
  position: relative;               /* Создает контекст позиционирования */
  width: 400px;
  height: 300px;
  border: 2px solid #333;
}

.absolute-child {
  position: absolute;
  top: 50px;
  left: 100px;
  width: 200px;
  height: 100px;
  background-color: #e0e0e0;
}

/* Центрирование элемента */
.centered-absolute {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%); /* Центрирование */
  width: 150px;
  height: 100px;
}
```

## Фиксированное позиционирование (fixed)

Фиксированное позиционирование извлекает элемент из нормального потока и позиционирует его относительно области просмотра (viewport), оставаясь на месте при прокрутке страницы.

```css
.fixed-element {
  position: fixed;
  top: 0;
  right: 0;
  width: 200px;
  height: 50px;
  background-color: #007bff;
  z-index: 1000;                    /* Высокий z-index для отображения поверх */
}
```

### Примеры использования fixed

```css
/* Фиксированное меню навигации */
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 60px;
  background-color: #333;
  z-index: 999;
}

/* Фиксированная боковая панель */
.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: 250px;
  background-color: #f8f9fa;
  overflow-y: auto;
}

/* Плавающая кнопка */
.floating-button {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: #007bff;
  z-index: 1000;
}
```

## Липкое позиционирование (sticky)

Липкое позиционирование - это гибрид относительного и фиксированного позиционирования. Элемент ведет себя как относительно позиционированный до тех пор, пока не будет достигнут определенный порог прокрутки, после чего ведет себя как фиксированный.

```css
.sticky-element {
  position: sticky;
  top: 10px;                        /* Отступ от верха области просмотра */
  /* Поведение зависит от контекста прокрутки */
}
```

### Примеры использования sticky

```css
/* Липкий заголовок таблицы */
.table-container {
  height: 400px;
  overflow-y: auto;
}

.sticky-header {
  position: sticky;
  top: 0;
  background-color: #f8f9fa;
  z-index: 10;
}

/* Липкие элементы навигации */
.sticky-nav {
  position: sticky;
  top: 0;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Липкие заголовки разделов */
.section-header {
  position: sticky;
  top: 0;
  background-color: #fff;
  padding: 10px 0;
}
```

## Взаимодействие между различными типами позиционирования

### Контекст позиционирования

Абсолютно позиционированные элементы позиционируются относительно ближайшего позиционированного предка:

```css
.container {
  position: relative;               /* Создает контекст позиционирования */
  width: 500px;
  height: 400px;
}

.absolute-child {
  position: absolute;
  top: 20px;                        /* Отсчитывается от .container */
  left: 30px;
}

.relative-grandchild {
  position: relative;
  top: 10px;                        /* Смещение внутри .absolute-child */
}
```

### Z-index и слои наложения

Свойство z-index определяет порядок наложения элементов:

```css
.back-layer {
  position: relative;
  z-index: 1;
}

.middle-layer {
  position: relative;
  z-index: 2;
}

.front-layer {
  position: relative;
  z-index: 3;
}

/* Создание нового контекста наложения */
.context {
  position: relative;
  z-index: 10;                      /* Создает новый контекст */
}

.context-child {
  z-index: 5;                       /* Относительно контекста .context */
}
```

## Практические применения

### Модальные окна

```css
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  position: relative;
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
}
```

### Dropdown меню

```css
.dropdown {
  position: relative;
  display: inline-block;
}

.dropdown-content {
  position: absolute;
  top: 100%;
  left: 0;
  background-color: white;
  min-width: 160px;
  box-shadow: 0px 8px 16px rgba(0,0,0,0.2);
  z-index: 1;
  display: none;
}

.dropdown:hover .dropdown-content {
  display: block;
}
```

### Плавающие элементы управления

```css
.editor-toolbar {
  position: sticky;
  top: 0;
  background-color: #f8f9fa;
  padding: 10px;
  z-index: 100;
}

.editor-content {
  position: relative;
  min-height: 500px;
}

.tooltip {
  position: absolute;
  background-color: #333;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  z-index: 200;
}
```

## Совместимость и поддержка

- `position: static` и `position: relative` - поддерживается во всех браузерах
- `position: absolute` - поддерживается во всех браузерах
- `position: fixed` - поддерживается во всех современных браузерах (ограничения в старых IE)
- `position: sticky` - хорошая поддержка в современных браузерах (ограничения в IE)

## Связанные темы

- [[CSS Box Model]] - модель коробки
- [[CSS Display Module]] - типы отображения элементов
- [[CSS Transform Module]] - трансформации элементов

## Заключение

CSS Positioning Module предоставляет гибкие инструменты для точного контроля над расположением элементов на веб-странице. Понимание различий между типами позиционирования позволяет создавать сложные и интерактивные интерфейсы, адаптирующиеся к различным условиям просмотра.
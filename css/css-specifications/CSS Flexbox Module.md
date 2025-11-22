---
aliases: [CSS Flexbox Specification, CSS Flexible Box, CSS Flex Layout, CSS Flexbox Levels]
tags: [css, flexbox, layout, specification, css-flexbox]
---

# CSS Flexbox Module: Уровни 1, 2 - Полное руководство

## Введение

CSS Flexible Box Layout Module (Flexbox) - это одномерная система компоновки, предназначенная для распределения пространства между элементами в контейнере и выравнивания этих элементов. Спецификация включает два уровня: Level 1 и Level 2.

## CSS Flexible Box Layout Module Level 1

Level 1 - основа системы Flexbox, включающая базовые возможности для создания гибких макетов.

### Основные свойства Flexbox

#### Свойства контейнера флекса

```css
.flex-container {
  display: flex;                    /* Блочный флекс-контейнер */
  display: inline-flex;             /* Встроенный флекс-контейнер */
  
  /* Направление основной оси */
  flex-direction: row;              /* Слева направо */
  flex-direction: row-reverse;      /* Справа налево */
  flex-direction: column;           /* Сверху вниз */
  flex-direction: column-reverse;   /* Снизу вверх */
  
  /* Перенос элементов */
  flex-wrap: nowrap;                /* Нет переноса */
  flex-wrap: wrap;                  /* Перенос сверху вниз */
  flex-wrap: wrap-reverse;          /* Перенос снизу вверх */
  
  /* Краткая запись flex-direction и flex-wrap */
  flex-flow: row wrap;              /* direction wrap */
  
  /* Выравнивание элементов по основной оси */
  justify-content: flex-start;      /* Начало оси */
  justify-content: flex-end;        /* Конец оси */
  justify-content: center;          /* По центру */
  justify-content: space-between;   /* Равномерное распределение */
  justify-content: space-around;    /* С отступами вокруг */
  justify-content: space-evenly;    /* Равные отступы */
  
  /* Выравнивание элементов по поперечной оси */
  align-items: stretch;             /* Растягивание по высоте */
  align-items: flex-start;          /* Прижатие к началу */
  align-items: flex-end;            /* Прижатие к концу */
  align-items: center;              /* По центру */
  align-items: baseline;            /* По базовой линии */
  
  /* Выравнивание строк флекса */
  align-content: flex-start;        /* Прижатие к началу */
  align-content: flex-end;          /* Прижатие к концу */
  align-content: center;            /* По центру */
  align-content: space-between;     /* Равномерное распределение */
  align-content: space-around;      /* С отступами вокруг */
  align-content: stretch;           /* Растягивание */
}
```

#### Свойства элементов флекса

```css
.flex-item {
  /* Растяжение элемента */
  flex-grow: 0;                     /* Не растягивается */
  flex-grow: 1;                     /* Растягивается пропорционально */
  
  /* Сжатие элемента */
  flex-shrink: 1;                   /* Сжимается при необходимости */
  flex-shrink: 0;                   /* Не сжимается */
  
  /* Базовый размер элемента */
  flex-basis: auto;                 /* Размер содержимого */
  flex-basis: 100px;                /* Фиксированный размер */
  flex-basis: 50%;                  /* Процентный размер */
  
  /* Краткая запись flex-grow, flex-shrink, flex-basis */
  flex: 1;                          /* flex: 1 1 0% */
  flex: 0 1 auto;                   /* grow shrink basis */
  
  /* Выравнивание отдельного элемента */
  align-self: auto;                 /* Следует align-items */
  align-self: flex-start;           /* Прижатие к началу */
  align-self: flex-end;             /* Прижатие к концу */
  align-self: center;               /* По центру */
  align-self: baseline;             /* По базовой линии */
  align-self: stretch;              /* Растягивание */
  
  /* Порядок элемента */
  order: 0;                         /* Нормальный порядок */
  order: -1;                        /* Раньше нормального */
  order: 1;                         /* Позже нормального */
}
```

### Примеры использования Level 1

```css
/* Центрирование элемента */
.center-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.centered-item {
  /* Элемент будет центрирован */
}

/* Навигационное меню */
.nav-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
}

.nav-item {
  flex: 0 0 auto;                   /* Не растягивается, не сжимается */
}

/* Карточки с равной высотой */
.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.card {
  flex: 1 1 300px;                  /* Минимальная ширина 300px */
  display: flex;
  flex-direction: column;
}
```

## CSS Flexible Box Layout Module Level 2

Level 2 добавляет поддержку `display: contents`, что позволяет элементам участвовать в родительском флекс-контейнере без создания визуального контейнера.

### Новые возможности Level 2

```css
.flex-container {
  display: flex;
  gap: 10px;                        /* Отступы между элементами */
}

.item-container {
  display: contents;                /* Элемент не создает визуального контейнера */
}

.item {
  /* Эти элементы будут напрямую участвовать в флекс-раскладке */
  flex: 1;
}
```

### Улучшенная поддержка gap

```css
.flex-container {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;                        /* Равные отступы */
  column-gap: 1.5rem;               /* Отступы между колонками */
  row-gap: 1rem;                    /* Отступы между строками */
}
```

## Практические применения

### Адаптивные макеты

```css
.adaptive-layout {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.adaptive-item {
  flex: 1 1 300px;                  /* Минимум 300px, растягивается */
  min-width: 0;                     /* Позволяет сжиматься */
}

@media (max-width: 768px) {
  .adaptive-layout {
    flex-direction: column;
  }
}
```

### Компоненты с фиксированными и гибкими частями

```css
.component {
  display: flex;
  height: 100vh;
}

.sidebar {
  flex: 0 0 250px;                  /* Фиксированная ширина 250px */
}

.main-content {
  flex: 1;                          /* Занимает оставшееся пространство */
  overflow: auto;                   /* Прокрутка при необходимости */
}

.footer {
  flex: 0 0 auto;                   /* Высота по содержимому */
}
```

### Компоновка с разными размерами элементов

```css
.mixed-sizes {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.small { flex: 0 0 150px; }
.medium { flex: 0 0 300px; }
.large { flex: 0 0 450px; }
.flexible { flex: 1 1 200px; }      /* Гибкий элемент */
```

## Распространенные паттерны

### "Holy Grail" Layout

```css
.holy-grail {
  display: flex;
  min-height: 100vh;
  flex-direction: column;
}

.header {
  flex: 0 0 auto;                   /* Фиксированная высота */
}

.main-content {
  display: flex;
  flex: 1;                          /* Занимает оставшееся пространство */
}

.sidebar {
  flex: 0 0 200px;                  /* Фиксированная ширина */
}

.content {
  flex: 1;                          /* Основной контент */
}
```

### "Sticky Footer"

```css
.sticky-footer {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.content {
  flex: 1;                          /* Растягивается */
}

.footer {
  flex: 0 0 auto;                   /* Фиксированная высота */
}
```

## Поддержка браузерами

- Level 1: Поддерживается во всех современных браузерах (с префиксами для старых версий)
- Level 2: Хорошая поддержка, особенно в Chrome, Firefox и Edge

## Связанные темы

- [[CSS Grid Layout Module]] - двумерная система компоновки
- [[CSS Box Model]] - модель коробки
- [[CSS Display Module]] - типы отображения элементов

## Заключение

CSS Flexbox Module предоставляет мощный инструмент для создания гибких одномерных макетов. Понимание свойств контейнера и элементов позволяет создавать адаптивные и удобные компоновки, которые автоматически подстраиваются под доступное пространство.
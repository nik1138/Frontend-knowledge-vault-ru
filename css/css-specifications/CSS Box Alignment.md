---
aliases: [CSS Box Alignment Specification, CSS Alignment Properties, CSS Box Alignment Module, CSS Flexbox and Grid Alignment]
tags: [css, alignment, layout, specification, css-alignment]
---

# CSS Box Alignment: Выравнивание в CSS - Полное руководство

## Введение

CSS Box Alignment Module определяет свойства для выравнивания коробок внутри их контейнеров. Модуль предоставляет согласованный набор свойств для выравнивания в разных системах компоновки, включая Flexbox и Grid.

## CSS Box Alignment Module Level 3

Level 3 определяет основные свойства выравнивания для различных систем компоновки.

### Свойства выравнивания для контейнера

```css
.container-alignment {
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
  
  /* Выравнивание строк флекса/сетки */
  align-content: flex-start;        /* Прижатие к началу */
  align-content: flex-end;          /* Прижатие к концу */
  align-content: center;            /* По центру */
  align-content: space-between;     /* Равномерное распределение */
  align-content: space-around;      /* С отступами вокруг */
  align-content: stretch;           /* Растягивание */
  
  /* Выравнивание строк сетки */
  justify-items: stretch;           /* Растягивание элементов */
  justify-items: start;             /* Прижатие к началу строки */
  justify-items: end;               /* Прижатие к концу строки */
  justify-items: center;            /* По центру строки */
}
```

### Свойства выравнивания для отдельных элементов

```css
.item-alignment {
  /* Выравнивание отдельного элемента */
  align-self: auto;                 /* Следует align-items родителя */
  align-self: flex-start;           /* Прижатие к началу */
  align-self: flex-end;             /* Прижатие к концу */
  align-self: center;               /* По центру */
  align-self: baseline;             /* По базовой линии */
  align-self: stretch;              /* Растягивание */
  
  /* Выравнивание по оси строки (для Grid) */
  justify-self: auto;               /* Следует justify-items родителя */
  justify-self: start;              /* Прижатие к началу строки */
  justify-self: end;                /* Прижатие к концу строки */
  justify-self: center;             /* По центру строки */
  justify-self: stretch;            /* Растягивание */
}
```

## CSS Box Alignment Module Level 4

Level 4 добавляет расширенные возможности выравнивания.

### Логические свойства выравнивания

```css
.logical-alignment {
  /* Логические свойства для разных направлений письма */
  justify-items: start;             /* Логическое начало */
  justify-items: end;               /* Логический конец */
  justify-content: start;           /* Логическое начало основной оси */
  justify-content: end;             /* Логический конец основной оси */
  
  /* Выравнивание с учетом направления письма */
  text-align: start;                /* Начало направления чтения */
  text-align: end;                  /* Конец направления чтения */
}
```

### Расширенные свойства выравнивания

```css
.extended-alignment {
  /* Позиционирование с выравниванием */
  place-content: center;            /* Краткая запись align-content и justify-content */
  place-content: start end;         /* align-content: start, justify-content: end */
  
  place-items: center;              /* Краткая запись align-items и justify-items */
  place-items: start stretch;       /* align-items: start, justify-items: stretch */
  
  place-self: center;               /* Краткая запись align-self и justify-self */
}
```

### Свойства распределения пространства

```css
.space-distribution {
  /* Распределение пространства */
  justify-content: space-around;    /* Пространство вокруг элементов */
  justify-content: space-evenly;    /* Равномерное пространство */
  justify-content: space-between;   /* Пространство между элементами */
  
  /* Новое значение для равномерного распределения */
  justify-content: space-evenly;
}
```

## Практические применения

### Flexbox выравнивание

```css
.flex-alignment {
  display: flex;
  flex-direction: row;
  
  /* Выравнивание элементов по основной оси */
  justify-content: center;          /* Центрирование по горизонтали */
  
  /* Выравнивание элементов по поперечной оси */
  align-items: center;              /* Центрирование по вертикали */
  
  /* Выравнивание строк при переполнении */
  flex-wrap: wrap;
  align-content: center;            /* Центрирование строк */
  gap: 10px;                        /* Отступы между элементами */
}

.flex-item {
  padding: 15px;
  background-color: #e9ecef;
  border-radius: 4px;
}

.flex-item-special {
  /* Отдельное выравнивание элемента */
  align-self: flex-start;           /* Прижатие к верху */
  justify-self: center;             /* Центрирование в строке */
}
```

### Grid выравнивание

```css
.grid-alignment {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 200px);
  
  /* Выравнивание сетки в контейнере */
  justify-content: center;          /* Центрирование сетки по горизонтали */
  align-content: center;            /* Центрирование сетки по вертикали */
  
  /* Выравнивание элементов сетки */
  justify-items: stretch;           /* Растягивание элементов по ширине */
  align-items: stretch;             /* Растягивание элементов по высоте */
  
  gap: 15px;
  height: 500px;
  width: 800px;
  border: 1px solid #ddd;
}

.grid-item {
  background-color: #f8f9fa;
  border: 1px solid #dee2e6;
  padding: 20px;
  text-align: center;
}

.grid-item-centered {
  /* Отдельное выравнивание элемента */
  justify-self: center;             /* Центрирование в ячейке по ширине */
  align-self: center;               /* Центрирование в ячейке по высоте */
  width: 100px;                     /* Ограниченный размер */
  height: 100px;
}

.grid-item-stretched {
  /* Растягивание элемента */
  justify-self: stretch;            /* Растягивание по ширине */
  align-self: stretch;              /* Растягивание по высоте */
  grid-column: span 2;              /* Занимает 2 колонки */
}
```

### Центрирование элементов

```css
.centering-examples {
  /* Центрирование в Flexbox */
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.centered-content {
  text-align: center;
  padding: 20px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

/* Центрирование в Grid */
.grid-centering {
  display: grid;
  place-items: center;              /* Краткая запись для центрирования */
  height: 100vh;
}

/* Абсолютное центрирование */
.absolute-centering {
  position: relative;
  height: 100vh;
}

.absolute-centered {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  /* Или использовать Grid */
  display: grid;
  place-self: center;
}
```

### Адаптивное выравнивание

```css
.responsive-alignment {
  display: flex;
  flex-direction: column;
  
  /* Начальное выравнивание */
  align-items: stretch;
  justify-content: flex-start;
}

@media (min-width: 768px) {
  .responsive-alignment {
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }
}

.responsive-item {
  flex: 1;
  margin: 5px;
}

.responsive-item:first-child {
  align-self: flex-start;           /* Прижатие первого элемента к верху */
}

.responsive-item:last-child {
  align-self: flex-end;             /* Прижатие последнего элемента к низу */
}
```

### Сложные схемы выравнивания

```css
.complex-alignment {
  display: grid;
  grid-template-areas: 
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  grid-template-columns: 200px 1fr 150px;
  grid-template-rows: 80px 1fr 60px;
  
  /* Выравнивание всей сетки */
  justify-content: stretch;
  align-content: start;
  
  gap: 10px;
  height: 100vh;
}

.complex-header {
  grid-area: header;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
}

.complex-sidebar {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  padding: 20px;
}

.complex-main {
  grid-area: main;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
  align-content: start;
}

.complex-aside {
  grid-area: aside;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.complex-footer {
  grid-area: footer;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0 20px;
}
```

## Современные возможности

### Логические свойства и выравнивание

```css
.logical-properties-alignment {
  display: flex;
  flex-direction: column;
  
  /* Логическое выравнивание с учетом направления письма */
  align-items: start;               /* Прижатие к логическому началу */
  justify-content: start;           /* Прижатие к началу основной оси */
}

.rtl-alignment {
  direction: rtl;                   /* Справа налево */
  display: flex;
  justify-content: start;           /* Будет справа из-за RTL */
  align-items: start;               /* Будет сверху */
}
```

### Выравнивание с CSS Custom Properties

```css
.dynamic-alignment {
  --alignment-main: center;
  --alignment-cross: center;
  
  display: flex;
  justify-content: var(--alignment-main);
  align-items: var(--alignment-cross);
  
  transition: all 0.3s ease;
}

.dynamic-alignment.left-aligned {
  --alignment-main: flex-start;
}

.dynamic-alignment.right-aligned {
  --alignment-main: flex-end;
}

.dynamic-alignment.top-aligned {
  --alignment-cross: flex-start;
}

.dynamic-alignment.bottom-aligned {
  --alignment-cross: flex-end;
}
```

### Выравнивание в сложных компонентах

```css
.component-alignment {
  display: grid;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.component-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
}

.component-body {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 300px;
  gap: 2rem;
  padding: 1rem;
}

.component-main-content {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.component-sidebar {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.component-footer {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  border-top: 1px solid #e2e8f0;
}
```

## Поддержка браузерами

- `justify-content`, `align-items`, `align-content`: Хорошая поддержка в современных браузерах
- `place-content`, `place-items`, `place-self`: Хорошая поддержка в современных браузерах
- `align-self`, `justify-self`: Хорошая поддержка в современных браузерах
- Логические свойства: Поддержка в процессе внедрения

## Лучшие практики

### Последовательное выравнивание

```css
.consistent-alignment {
  /* Использование согласованных значений выравнивания */
  display: flex;
  justify-content: flex-start;      /* Ясное намерение */
  align-items: center;              /* Согласованное выравнивание */
  gap: 1rem;                        /* Согласованные отступы */
}

/* Для сеток */
.consistent-grid {
  display: grid;
  place-items: stretch;             /* Ясное растягивание элементов */
  gap: 1rem;
}
```

### Адаптивные паттерны выравнивания

```css
.responsive-patterns {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: flex-start;
}

@media (min-width: 768px) {
  .responsive-patterns {
    flex-direction: row;
    align-items: flex-start;
    justify-content: space-between;
  }
}

.responsive-patterns > * {
  flex: 1;
  min-width: 0;                   /* Позволяет сжиматься */
}
```

## Связанные темы

- [[CSS Flexbox Module]] - флекс-модуль
- [[CSS Grid Layout Module]] - модуль сетки
- [[CSS Layout Modules]] - модули компоновки

## Заключение

CSS Box Alignment Module предоставляет мощные и согласованные инструменты для выравнивания элементов в различных системах компоновки. Понимание свойств выравнивания позволяет создавать гибкие и адаптивные интерфейсы с точным контролем над расположением элементов.
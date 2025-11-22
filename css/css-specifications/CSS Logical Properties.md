---
aliases: [CSS Logical Properties Specification, CSS Logical Values, CSS Logical Box Model, CSS Internationalization Properties]
tags: [css, logical-properties, internationalization, specification, css-logical]
---

# CSS Logical Properties: Логические свойства для международного дизайна - Полное руководство

## Введение

CSS Logical Properties and Values Module определяет свойства и значения, основанные на логических направлениях, а не на физических. Это позволяет создавать интерфейсы, которые корректно адаптируются к различным направлениям письма (LTR/RTL) и режимам письма (горизонтальные/вертикальные), что особенно важно для интернационализации.

## CSS Logical Properties and Values Level 1

Level 1 вводит основные логические свойства для замены физических свойств.

### Логические свойства для размеров

```css
.logical-sizes {
  /* Логические размеры вместо width/height */
  block-size: 100px;                /* Размер в блочном направлении */
  inline-size: 200px;               /* Размер в строчном направлении */
  
  /* Минимальные и максимальные логические размеры */
  min-block-size: 50px;             /* Минимальный размер в блочном направлении */
  max-block-size: 200px;            /* Максимальный размер в блочном направлении */
  min-inline-size: 100px;           /* Минимальный размер в строчном направлении */
  max-inline-size: 300px;           /* Максимальный размер в строчном направлении */
}

/* Пример соответствия в разных режимах письма */
.ltr-example {
  /* В LTR горизонтальном режиме: */
  inline-size: 200px;              /* = width: 200px */
  block-size: 100px;               /* = height: 100px */
}

.vertical-rl-example {
  /* В вертикальном режиме (writing-mode: vertical-rl): */
  inline-size: 200px;              /* = height: 200px */
  block-size: 100px;               /* = width: 100px */
}
```

### Логические свойства для отступов

```css
.logical-padding {
  /* Логические отступы вместо padding-top/right/bottom/left */
  padding-block-start: 10px;        /* Отступ в начале блочного направления */
  padding-block-end: 15px;          /* Отступ в конце блочного направления */
  padding-inline-start: 20px;       /* Отступ в начале строчного направления */
  padding-inline-end: 25px;         /* Отступ в конце строчного направления */
  
  /* Краткая запись для блочного направления */
  padding-block: 10px 15px;         /* padding-block-start: 10px; padding-block-end: 15px; */
  
  /* Краткая запись для строчного направления */
  padding-inline: 12px 18px;        /* padding-inline-start: 12px; padding-inline-end: 18px; */
}

/* Соответствие физическим свойствам в LTR горизонтальном режиме */
.ltr-padding {
  padding-block-start: 10px;        /* = padding-top: 10px */
  padding-block-end: 15px;          /* = padding-bottom: 15px */
  padding-inline-start: 20px;       /* = padding-left: 20px */
  padding-inline-end: 25px;         /* = padding-right: 25px */
}

/* Соответствие физическим свойствам в RTL горизонтальном режиме */
.rtl-padding {
  padding-block-start: 10px;        /* = padding-top: 10px */
  padding-block-end: 15px;          /* = padding-bottom: 15px */
  padding-inline-start: 20px;       /* = padding-right: 20px */
  padding-inline-end: 25px;         /* = padding-left: 25px */
}

/* Соответствие физическим свойствам в вертикальном режиме */
.vertical-padding {
  padding-block-start: 10px;        /* = padding-top: 10px */
  padding-block-end: 15px;          /* = padding-bottom: 15px */
  padding-inline-start: 20px;       /* = padding-bottom: 20px (в vertical-rl) */
  padding-inline-end: 25px;         /* = padding-top: 25px (в vertical-rl) */
}
```

### Логические свойства для границ

```css
.logical-borders {
  /* Логические границы */
  border-block-start: 1px solid #000;   /* Граница в начале блочного направления */
  border-block-end: 2px dashed #666;    /* Граница в конце блочного направления */
  border-inline-start: 3px dotted #999; /* Граница в начале строчного направления */
  border-inline-end: 4px solid #ccc;    /* Граница в конце строчного направления */
  
  /* Краткая запись для блочного направления */
  border-block: 1px solid #000;         /* Устанавливает обе блочные границы */
  
  /* Краткая запись для строчного направления */
  border-inline: 2px solid #666;        /* Устанавливает обе строчные границы */
  
  /* Толщина логических границ */
  border-block-start-width: 2px;
  border-block-end-width: 1px;
  border-inline-start-width: 3px;
  border-inline-end-width: 4px;
  
  /* Стиль логических границ */
  border-block-start-style: solid;
  border-block-end-style: dashed;
  border-inline-start-style: dotted;
  border-inline-end-style: double;
  
  /* Цвет логических границ */
  border-block-start-color: #ff0000;
  border-block-end-color: #00ff00;
  border-inline-start-color: #0000ff;
  border-inline-end-color: #ffff00;
}
```

### Логические свойства для внешних отступов

```css
.logical-margins {
  /* Логические внешние отступы */
  margin-block-start: 10px;         /* Внешний отступ в начале блочного направления */
  margin-block-end: 15px;           /* Внешний отступ в конце блочного направления */
  margin-inline-start: 20px;        /* Внешний отступ в начале строчного направления */
  margin-inline-end: 25px;          /* Внешний отступ в конце строчного направления */
  
  /* Краткая запись для блочного направления */
  margin-block: 10px 15px;          /* margin-block-start: 10px; margin-block-end: 15px; */
  
  /* Краткая запись для строчного направления */
  margin-inline: 12px 18px;         /* margin-inline-start: 12px; margin-inline-end: 18px; */
}
```

## CSS Logical Properties and Values Level 2

Level 2 добавляет дополнительные логические свойства и значения.

### Логические свойства для позиционирования

```css
.logical-positioning {
  position: absolute;
  
  /* Логические свойства для позиционирования */
  inset-block-start: 10px;          /* Позиционирование в начале блочного направления */
  inset-block-end: 15px;            /* Позиционирование в конце блочного направления */
  inset-inline-start: 20px;         /* Позиционирование в начале строчного направления */
  inset-inline-end: 25px;           /* Позиционирование в конце строчного направления */
  
  /* Краткая запись для блочного направления */
  inset-block: 10px 15px;           /* inset-block-start: 10px; inset-block-end: 15px; */
  
  /* Краткая запись для строчного направления */
  inset-inline: 20px 25px;          /* inset-inline-start: 20px; inset-inline-end: 25px; */
  
  /* Универсальное позиционирование */
  inset: 10px 20px 15px 25px;       /* top right bottom left в LTR горизонтальном режиме */
  /* или */
  inset: 10px 20px 15px 25px;       /* block-start inline-end block-end inline-start */
}

/* Соответствие в разных режимах */
.ltr-position {
  inset-block-start: 10px;          /* = top: 10px */
  inset-block-end: 15px;            /* = bottom: 15px */
  inset-inline-start: 20px;         /* = left: 20px */
  inset-inline-end: 25px;           /* = right: 25px */
}

.rtl-position {
  inset-block-start: 10px;          /* = top: 10px */
  inset-block-end: 15px;            /* = bottom: 15px */
  inset-inline-start: 20px;         /* = right: 20px */
  inset-inline-end: 25px;           /* = left: 25px */
}
```

### Логические свойства для выравнивания

```css
.logical-alignment {
  display: flex;
  
  /* Логические свойства для выравнивания в Flexbox */
  justify-content: start;           /* Прижимает элементы к началу строчного направления */
  align-items: start;               /* Прижимает элементы к началу блочного направления */
  
  /* В Grid */
  display: grid;
  justify-items: start;             /* Выравнивание элементов сетки по строчному направлению */
  align-items: start;               /* Выравнивание элементов сетки по блочному направлению */
}
```

### Логические свойства для границ (расширенные)

```css
.extended-logical-borders {
  /* Радиусы логических границ */
  border-start-start-radius: 10px;      /* Радиус в начале блока и начале строки */
  border-start-end-radius: 10px;        /* Радиус в начале блока и конце строки */
  border-end-start-radius: 10px;        /* Радиус в конце блока и начале строки */
  border-end-end-radius: 10px;          /* Радиус в конце блока и конце строки */
  
  /* Соответствие в LTR горизонтальном режиме */
  /* border-start-start-radius = border-top-left-radius */
  /* border-start-end-radius = border-top-right-radius */
  /* border-end-start-radius = border-bottom-left-radius */
  /* border-end-end-radius = border-bottom-right-radius */
  
  /* Соответствие в RTL горизонтальном режиме */
  /* border-start-start-radius = border-top-right-radius */
  /* border-start-end-radius = border-top-left-radius */
  /* border-end-start-radius = border-bottom-right-radius */
  /* border-end-end-radius = border-bottom-left-radius */
}
```

## Практические применения

### Адаптация к различным направлениям письма

```css
.bidirectional-layout {
  /* Установка направления письма */
  direction: rtl;                   /* Справа налево */
  /* или */
  direction: ltr;                   /* Слева направо */
  
  /* Использование логических свойств */
  padding-inline-start: 20px;       /* Отступ всегда "в начале направления чтения" */
  padding-inline-end: 10px;         /* Отступ всегда "в конце направления чтения" */
  margin-inline-start: 15px;        /* Внешний отступ в начале направления */
  border-inline-start: 2px solid #000; /* Граница в начале направления */
}

/* Пример для вертикального письма */
.vertical-layout {
  writing-mode: vertical-rl;        /* Вертикальное письмо справа налево */
  
  /* Логические свойства автоматически адаптируются */
  padding-block-start: 10px;        /* Верхний отступ */
  padding-inline-start: 20px;       /* Правый отступ в vertical-rl */
  border-block-end: 1px solid #ccc; /* Нижняя граница */
  margin-inline-end: 15px;          /* Левый внешний отступ в vertical-rl */
}
```

### Интернационализированные компоненты

```css
.internationalized-component {
  display: flex;
  flex-direction: column;
  
  /* Логические отступы для контента */
  padding-block: 20px;
  padding-inline: 15px;
  
  /* Логические границы */
  border-inline-start: 4px solid var(--accent-color);
  
  /* Логические размеры */
  min-block-size: 200px;
  max-inline-size: 500px;
  
  /* Логические внешние отступы */
  margin-block-end: 25px;
}

.component-header {
  /* Логические отступы для заголовка */
  padding-block-end: 10px;
  padding-inline-start: 5px;
  
  /* Логическая граница */
  border-block-end: 1px solid #eee;
  
  /* Выравнивание с учетом направления */
  text-align: start;                /* Прижимает текст к началу направления чтения */
}

.component-content {
  /* Логические отступы для контента */
  padding-block: 15px;
  
  /* Растягивание в блочном направлении */
  flex: 1;
}

.component-footer {
  /* Логические отступы для футера */
  padding-block-start: 10px;
  padding-inline-end: 5px;
  
  /* Выравнивание элементов */
  display: flex;
  justify-content: end;             /* Прижимает элементы к концу строчного направления */
}
```

### Адаптивные сетки с логическими свойствами

```css
.logical-grid {
  display: grid;
  grid-template-columns: 
    minmax(0, 1fr) 
    minmax(280px, 300px) 
    minmax(0, 1fr);
  gap: 20px;
  
  /* Логические отступы для всей сетки */
  padding-block: 25px;
  padding-inline: clamp(1rem, 5vw, 3rem);
  
  /* Выравнивание содержимого сетки */
  align-content: start;
  justify-content: center;
  
  /* Максимальная ширина с центрированием */
  max-inline-size: 1400px;
  margin-inline: auto;
}

.sidebar-left {
  /* Использует первый трек сетки */
  grid-column: 1;
  /* Логические отступы */
  padding-inline-end: 15px;
  border-inline-end: 1px solid #eee;
}

.main-content {
  /* Использует второй трек сетки */
  grid-column: 2;
  /* Логические отступы */
  padding-inline: 20px;
}

.sidebar-right {
  /* Использует третий трек сетки */
  grid-column: 3;
  /* Логические отступы */
  padding-inline-start: 15px;
  border-inline-start: 1px solid #eee;
}
```

### Компоненты с различными режимами письма

```css
.multidirectional-component {
  /* Возможность переключения режимов письма */
  writing-mode: horizontal-tb;      /* Горизонтальное письмо сверху вниз */
  /* или */
  writing-mode: vertical-rl;        /* Вертикальное письмо справа налево */
  /* или */
  writing-mode: vertical-lr;        /* Вертикальное письмо слева направо */
  
  /* Логические свойства автоматически адаптируются */
  padding-block: 15px;
  padding-inline: 20px;
  border-block: 1px solid #ddd;
  border-inline: 2px solid #666;
  
  /* Размеры */
  block-size: 300px;
  inline-size: 200px;
  
  /* Позиционирование */
  position: relative;
  inset-block-start: 10px;
  inset-inline-end: 10px;
}

/* Текст в компоненте */
.multidirectional-text {
  /* Выравнивание текста с учетом направления */
  text-align: start;                /* Прижимает к началу направления чтения */
  
  /* Логические отступы текста */
  text-indent: 1em;                 /* Отступ первой строки */
  
  /* Направление текста */
  direction: inherit;               /* Наследует направление от родителя */
}
```

## Современные возможности

### Логические свойства с CSS Custom Properties

```css
.dynamic-logical-properties {
  /* Использование переменных с логическими свойствами */
  --spacing-unit: 1rem;
  --border-width: 2px;
  --accent-color: #007bff;
  
  padding-block: var(--spacing-unit);
  padding-inline: calc(var(--spacing-unit) * 1.5);
  
  border-inline-start: var(--border-width) solid var(--accent-color);
  margin-block-end: var(--spacing-unit);
  
  /* Адаптивные логические размеры */
  --min-width: 250px;
  --max-width: 500px;
  
  min-inline-size: var(--min-width);
  max-inline-size: var(--max-width);
}
```

### Адаптивные логические свойства

```css
.responsive-logical {
  /* Адаптивные отступы */
  padding-inline: clamp(1rem, 5vw, 3rem);
  
  /* Адаптивные размеры */
  min-block-size: min(300px, 40vh);
  max-inline-size: 100%;            /* Всегда адаптируется к ширине родителя */
  
  /* Адаптивные внешние отступы */
  margin-block: max(1rem, 2vh);
  margin-inline: 0;                 /* Нет боковых внешних отступов */
}

@media (min-width: 768px) {
  .responsive-logical {
    /* Разные отступы на больших экранах */
    padding-block: 2rem;
    padding-inline: 4rem;
    
    /* Центрирование на больших экранах */
    margin-inline: auto;
    max-inline-size: 1200px;
  }
}
```

### Логические свойства в анимациях

```css
.animated-logical {
  /* Анимация с логическими свойствами */
  padding-inline-start: 20px;
  transition: padding-inline-start 0.3s ease;
}

.animated-logical:hover {
  padding-inline-start: 40px;       /* Анимация в направлении чтения */
}

/* Анимация позиционирования */
.positioned-logical {
  position: relative;
  inset-inline-start: 0;
  transition: inset-inline-start 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.positioned-logical.active {
  inset-inline-start: 250px;        /* Сдвиг в направлении чтения */
}
```

## Поддержка браузерами

- Логические свойства размеров: Хорошая поддержка в современных браузерах
- Логические свойства отступов: Хорошая поддержка в современных браузерах
- Логические свойства границ: Хорошая поддержка в современных браузерах
- Логические свойства внешних отступов: Хорошая поддержка в современных браузерах
- Логические свойства позиционирования: Поддержка в процессе внедрения

## Лучшие практики

### Последовательное использование логических свойств

```css
.consistent-logical-approach {
  /* Последовательное использование логических свойств */
  padding-block: 1rem;
  padding-inline: 1.5rem;
  
  margin-block-end: 1rem;
  margin-inline-start: 0;
  
  border-inline-start: 3px solid var(--primary-color);
  
  block-size: auto;
  inline-size: 100%;
  
  min-block-size: 200px;
  max-inline-size: 1200px;
}

/* Для компонентов, которые должны поддерживать разные направления */
.multidirectional-component {
  /* Используйте логические свойства для всех геометрических значений */
  padding-inline: 1rem;
  margin-block: 0.5rem;
  border-inline-start: 2px solid currentColor;
  
  /* Избегайте физических свойств для геометрии */
  /* padding-left: 1rem; - не рекомендуется */
  /* margin-right: 0.5rem; - не рекомендуется */
}
```

### Комбинирование с физическими свойствами

```css
.hybrid-approach {
  /* Используйте логические свойства для основной геометрии */
  padding-inline: 1rem;
  padding-block: 0.75rem;
  
  /* Используйте физические свойства для декоративных элементов */
  box-shadow: 2px 2px 4px rgba(0,0,0,0.1);  /* Тень всегда справа и снизу */
  
  /* Для специфичных эффектов */
  border-radius: 8px 0 8px 0;       /* Специфичные скругления углов */
}
```

## Связанные темы

- [[CSS Writing Modes]] - режимы письма
- [[CSS Internationalization]] - интернационализация
- [[CSS Box Model]] - модель коробки

## Заключение

CSS Logical Properties Module предоставляет мощные инструменты для создания интернационализированных интерфейсов, которые корректно отображаются в различных системах письма. Понимание и использование логических свойств позволяет создавать более гибкие и адаптивные веб-сайты, поддерживающие различные языки и культурные особенности.
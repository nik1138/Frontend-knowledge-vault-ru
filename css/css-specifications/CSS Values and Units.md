---
aliases: [CSS Values and Units Specification, CSS Units, CSS Value Types, CSS Measurement Units]
tags: [css, values, units, specification, css-units]
---

# CSS Values and Units: Все значения и единицы измерения - Полное руководство

## Введение

CSS Values and Units Module определяет все типы значений и единицы измерения, используемые в CSS. Модуль включает в себя числовые значения, размеры, углы, частоты, временные значения, проценты и другие типы данных, используемые для определения стилевых свойств.

## CSS Values and Units Module Level 3

Level 3 определяет основные типы значений и единицы измерения.

### Числовые значения

```css
.numeric-values {
  /* Целые числа */
  z-index: 1;
  order: 2;
  flex-grow: 1;
  
  /* Дробные числа */
  opacity: 0.5;
  flex-shrink: 0.8;
  line-height: 1.4;
  
  /* Безразмерные числа */
  line-height: 1.5;               /* Относительно размера шрифта */
  flex-grow: 2;                   /* Без единиц измерения */
}
```

### Абсолютные единицы измерения

```css
.absolute-units {
  /* Физические единицы */
  width: 25.4mm;                  /* 25.4 миллиметра = 1 дюйм */
  height: 1in;                    /* 1 дюйм = 96px */
  margin: 2.54cm;                 /* 2.54 сантиметра = 1 дюйм */
  padding: 72pt;                  /* 72 пункта = 1 дюйм */
  border-width: 6pc;              /* 6 пика = 1 дюйм */
  font-size: 8.5in;               /* Очень большой шрифт */
}
```

### Относительные единицы измерения

```css
.relative-units {
  /* Относительно шрифта родителя */
  font-size: 1.2em;               /* 1.2 размера шрифта родителя */
  margin: 1.5em;                  /* 1.5 ширины "M" родительского шрифта */
  
  /* Относительно собственного шрифта */
  padding: 1.5ex;                 /* 1.5 высоты буквы "x" */
  line-height: 2ch;               /* 2 ширины символа "0" */
  width: 30rem;                   /* 30 ширины "M" корневого шрифта */
  
  /* Относительно области просмотра */
  height: 50vh;                   /* 50% высоты области просмотра */
  width: 80vw;                    /* 80% ширины области просмотра */
  min-height: 100vmin;            /* 100% от меньшего из высоты/ширины */
  max-width: 50vmax;              /* 50% от большего из высоты/ширины */
  
  /* Относительно единицы */
  flex-basis: 1fr;                /* Доступное пространство */
}
```

### Процентные значения

```css
.percentage-values {
  width: 50%;                     /* 50% от ширины родительского элемента */
  height: 75%;                    /* 75% от высоты родительского элемента */
  font-size: 120%;                /* 120% от размера шрифта родителя */
  line-height: 150%;              /* 1.5 от размера шрифта */
  margin: 5%;                     /* 5% от ширины родительского блока */
  padding: 2.5%;                  /* 2.5% от ширины родительского блока */
}
```

### Угловые единицы

```css
.angle-units {
  transform: rotate(45deg);        /* Градусы */
  transform: rotate(0.5turn);      /* Обороты (1turn = 360deg) */
  transform: rotate(1.57rad);      /* Радианы (π/2 ≈ 1.57rad) */
  transform: rotate(50grad);       /* Грады (400grad = 360deg) */
}
```

### Временные единицы

```css
.time-units {
  transition-duration: 300ms;      /* Миллисекунды */
  animation-duration: 2s;          /* Секунды */
  transition-delay: 0.5s;          /* 500 миллисекунд */
  animation-delay: 1500ms;         /* 1.5 секунды */
}
```

### Частотные единицы

```css
.frequency-units {
  /* Редко используется, в основном в aural CSS */
  pitch: 120Hz;                   /* Герцы */
  src: url('sound.wav') format('wav') speech(100Hz, 200Hz);
}
```

## CSS Values and Units Module Level 4

Level 4 добавляет расширенные возможности и новые единицы измерения.

### Динамические единицы области просмотра

```css
.dynamic-viewport-units {
  /* Единицы, которые изменяются при появлении/исчезновении адресной строки */
  height: 100dvh;                 /* 100% динамической высоты области просмотра */
  width: 100dvw;                  /* 100% динамической ширины области просмотра */
  min-height: 100dvmin;           /* Меньшее из dvh/dvw */
  max-width: 100dvmax;            /* Большее из dvh/dvw */
}

/* Статические единицы (не изменяются при скролле) */
.static-viewport-units {
  height: 100svh;                 /* 100% статической высоты области просмотра */
  width: 100svw;                  /* 100% статической ширины области просмотра */
}

/* Минимальные/максимальные единицы области просмотра */
.viewport-logical-units {
  height: 100lvh;                 /* 100% большой высоты области просмотра */
  width: 100svw;                  /* 100% малой ширины области просмотра */
}
```

### Функции вычисления

```css
.calculation-functions {
  /* Функция calc() для вычислений */
  width: calc(100% - 20px);        /* 100% минус 20px */
  height: calc(50vh - 60px);       /* 50% высоты области просмотра минус 60px */
  margin: calc(2rem + 4px);        /* 2 ширины "M" плюс 4px */
  padding: calc(5% + 10px);        /* 5% плюс 10px */
  
  /* Вложенные вычисления */
  font-size: calc(1rem * calc(1 + 0.2)); /* 1.2rem */
  
  /* С использованием переменных */
  --base-width: 300px;
  --margin: 20px;
  width: calc(var(--base-width) - var(--margin) * 2);
}

/* Функции min(), max(), clamp() */
.min-max-functions {
  /* min() - выбирает наименьшее значение */
  width: min(300px, 100%);         /* Меньшее из 300px и 100% */
  
  /* max() - выбирает наибольшее значение */
  min-width: max(200px, 20vw);     /* Большее из 200px и 20% ширины области просмотра */
  
  /* clamp() - ограничивает значение между минимумом и максимумом */
  font-size: clamp(1rem, 2.5vw, 2rem); /* От 1rem до 2rem, с основой 2.5vw */
}
```

### Новые логические единицы

```css
.logical-units {
  /* Логические единицы для размеров */
  margin-block-start: 1rem;       /* Отступ в начале блока */
  margin-block-end: 1rem;         /* Отступ в конце блока */
  margin-inline-start: 20px;      /* Отступ в начале строки */
  margin-inline-end: 20px;        /* Отступ в конце строки */
  
  padding-block-start: 0.5rem;
  padding-block-end: 0.5rem;
  padding-inline-start: 1rem;
  padding-inline-end: 1rem;
}
```

## Практические применения

### Адаптивные размеры шрифта

```css
.responsive-typography {
  /* Адаптивный заголовок */
  font-size: clamp(1.5rem, 4vw, 3rem);
  /* От 1.5rem до 3rem, с основой 4vw */
  
  /* Адаптивный основной текст */
  font-size: clamp(1rem, 2.5vw, 1.2rem);
  line-height: 1.5;
}

/* Использование в сетке */
.grid-responsive {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(250px, 100%), 1fr));
  gap: 1rem;
}
```

### Адаптивные размеры элементов

```css
.responsive-elements {
  /* Адаптивная ширина карточки */
  width: clamp(280px, 80%, 500px);
  
  /* Адаптивная высота с сохранением соотношения сторон */
  aspect-ratio: 16 / 9;
  height: min(400px, 50vh);
  
  /* Адаптивные отступы */
  padding: clamp(1rem, 3vw, 2rem);
  
  /* Адаптивные границы */
  border-width: clamp(1px, 0.5vw, 3px);
}
```

### Вычисления с использованием разных единиц

```css
.complex-calculations {
  /* Смешивание разных единиц */
  width: calc(50% - 1rem - 2px);   /* Процент минус em минус px */
  margin: calc(2vw + 10px);        /* vw плюс px */
  
  /* Вычисления для сеток */
  grid-template-columns: 
    calc(33.333% - 10px) 
    calc(33.333% - 10px) 
    calc(33.333% - 10px);
  gap: 15px;
  
  /* Вычисления для позиционирования */
  left: calc(50% - 200px);         /* Центрирование фиксированного элемента */
  top: calc(100vh - 100px);        /* Позиционирование от нижнего края */
}
```

### Адаптивные отступы и размеры

```css
.responsive-spacing {
  /* Использование системных отступов */
  --space-unit: clamp(0.5rem, 2vw, 1rem);
  
  margin: var(--space-unit);
  padding: calc(var(--space-unit) * 2) var(--space-unit);
  
  /* Адаптивные размеры границ */
  border-width: max(1px, 0.1em);
  
  /* Адаптивные тени */
  box-shadow: 0 max(2px, 0.1rem) max(4px, 0.2rem) rgba(0,0,0,0.1);
}
```

### Использование с кастомными свойствами

```css
.custom-property-calculations {
  /* Базовые значения */
  --base-font-size: 16px;
  --scale-ratio: 1.2;
  --container-padding: clamp(1rem, 4vw, 3rem);
  
  /* Вычисления на основе переменных */
  --font-size-sm: calc(var(--base-font-size) / var(--scale-ratio));
  --font-size-md: var(--base-font-size);
  --font-size-lg: calc(var(--base-font-size) * var(--scale-ratio));
  --font-size-xl: calc(var(--base-font-size) * var(--scale-ratio) * var(--scale-ratio));
  
  font-size: var(--font-size-md);
  padding: var(--container-padding);
}
```

## Современные возможности

### Единицы для доступности

```css
.accessibility-units {
  /* Единицы, которые учитывают настройки пользователя */
  font-size: 1rem;                /* Относительно настроек шрифта пользователя */
  
  /* Использование системных шрифтов */
  font: caption;                  /* Шрифт для подписей */
  font: icon;                     /* Шрифт для иконок */
  font: menu;                     /* Шрифт для меню */
  font: message-box;              /* Шрифт для сообщений */
  font: small-caption;            /* Шрифт для маленьких подписей */
  font: status-bar;               /* Шрифт для строки состояния */
}
```

### Адаптивные единицы для разных устройств

```css
.device-adaptive-units {
  /* Использование разных единиц для разных устройств */
  font-size: clamp(1rem, 2.5vw, 1.5rem);
  
  /* Адаптация к плотности пикселей */
  border-width: max(1px, 0.02rem);
  
  /* Адаптация к размеру экрана */
  padding: min(5vw, 3rem);
  
  /* Адаптация к ориентации */
  height: 100svh;                 /* Работает везде, включая мобильные устройства */
}
```

### Функции для сложных вычислений

```css
.complex-functions {
  /* Использование нескольких функций вместе */
  width: calc(
    (100% - (var(--gutter) * (var(--columns) - 1))) / var(--columns)
  );
  
  /* Сложные адаптивные значения */
  font-size: clamp(
    var(--min-font-size), 
    var(--fluid-factor) * 1vw, 
    var(--max-font-size)
  );
  
  /* Условные значения */
  margin: max(1em, 4vw);
  padding: min(2rem, 8vh);
}
```

## Поддержка браузерами

- Базовые единицы (px, em, rem, %): Поддерживается во всех браузерах
- `calc()`: Хорошая поддержка в современных браузерах
- `min()`, `max()`, `clamp()`: Хорошая поддержка в современных браузерах
- Динамические единицы области просмотра: Поддержка в процессе внедрения
- Логические единицы: Хорошая поддержка в современных браузерах

## Лучшие практики

### Выбор правильных единиц

```css
.best-practices {
  /* Для размеров шрифта используйте rem для корня, em для внутренних элементов */
  font-size: 1.2rem;              /* Для заголовков */
  line-height: 1.4em;             /* Для внутренних элементов */
  
  /* Для отступов и размеров используйте адаптивные подходы */
  padding: clamp(1rem, 3vw, 2rem);
  
  /* Для сеток используйте fr, % и calc() */
  grid-template-columns: 1fr 2fr 1fr;
  
  /* Для границ используйте px или em */
  border: max(1px, 0.05em) solid #ccc;
  
  /* Для позиционирования используйте %, vw, vh, или px */
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

### Системы отступов

```css
.spacing-system {
  /* Создание системы отступов на основе rem */
  --space-1: 0.25rem;             /* 4px при 16px базе */
  --space-2: 0.5rem;              /* 8px */
  --space-3: 1rem;                /* 16px */
  --space-4: 1.5rem;              /* 24px */
  --space-5: 2rem;                /* 32px */
  --space-6: 3rem;                /* 48px */
  
  margin: var(--space-3);
  padding: var(--space-2) var(--space-4);
}
```

## Связанные темы

- [[CSS Custom Properties Module]] - кастомные свойства
- [[CSS Layout Modules]] - модули компоновки
- [[CSS Typography]] - типографика

## Заключение

CSS Values and Units Module предоставляет богатый набор типов значений и единиц измерения, позволяющих создавать гибкие и адаптивные интерфейсы. Понимание различных единиц и их правильное применение критически важно для создания качественных веб-сайтов и приложений.
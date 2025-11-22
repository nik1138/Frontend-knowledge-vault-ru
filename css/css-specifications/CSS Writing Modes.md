---
aliases: [CSS Writing Modes Specification, CSS Writing Systems, CSS Text Direction, CSS Writing Direction]
tags: [css, writing-modes, typography, specification, css-writing]
---

# CSS Writing Modes: Горизонтальные и вертикальные режимы письма - Полное руководство

## Введение

CSS Writing Modes Module определяет системы письма и способы расположения текста в различных направлениях. Модуль позволяет создавать интерфейсы, поддерживающие различные системы письма, включая горизонтальные (слева направо и справа налево) и вертикальные (сверху вниз) направления.

## CSS Writing Modes Module Level 3

Level 3 определяет основные свойства режимов письма.

### Основные свойства writing-mode

```css
.writing-mode-examples {
  /* Горизонтальное письмо сверху вниз */
  writing-mode: horizontal-tb;      /* Значение по умолчанию для большинства языков */
  
  /* Вертикальное письмо справа налево */
  writing-mode: vertical-rl;        /* Используется в восточноазиатских языках */
  
  /* Вертикальное письмо слева направо */
  writing-mode: vertical-lr;        /* Альтернативный вертикальный режим */
}
```

### Направление текста (direction)

```css
.direction-examples {
  /* Направление потока текста */
  direction: ltr;                   /* Слева направо (по умолчанию) */
  direction: rtl;                   /* Справа налево (для арабского, иврита и др.) */
}
```

### Упорядочивание столбцов (text-orientation)

```css
.text-orientation-examples {
  /* Ориентация символов в вертикальном режиме */
  text-orientation: mixed;          /* Смешанная ориентация (по умолчанию) */
  text-orientation: upright;        /* Символы остаются в обычной ориентации */
  text-orientation: sideways;       /* Символы повернуты на 90 градусов */
  text-orientation: sideways-right; /* Символы повернуты на 90 градусов вправо */
}
```

### Объединение символов (text-combine-upright)

```css
.text-combine-examples {
  /* Объединение символов в вертикальном режиме */
  text-combine-upright: none;       /* Без объединения */
  text-combine-upright: all;        /* Объединить все символы в одну ячейку */
  text-combine-upright: digits 2;   /* Объединить 2 цифры в одну ячейку */
  text-combine-upright: digits 4;   /* Объединить 4 цифры в одну ячейку */
}
```

## Практические примеры использования

### Горизонтальные режимы

```css
.ltr-content {
  writing-mode: horizontal-tb;
  direction: ltr;
  text-align: left;
  /* Текст идет слева направо, строки сверху вниз */
}

.rtl-content {
  writing-mode: horizontal-tb;
  direction: rtl;
  text-align: right;
  /* Текст идет справа налево, строки сверху вниз */
}
```

### Вертикальные режимы

```css
.vertical-rl-content {
  writing-mode: vertical-rl;
  text-orientation: upright;
  /* Текст идет сверху вниз, столбцы справа налево */
}

.vertical-lr-content {
  writing-mode: vertical-lr;
  text-orientation: upright;
  /* Текст идет сверху вниз, столбцы слева направо */
}
```

### Примеры вертикального текста

```css
.vertical-header {
  writing-mode: vertical-rl;
  text-orientation: upright;
  letter-spacing: 0.5em;
  padding: 10px;
  border: 1px solid #ccc;
}

.vertical-paragraph {
  writing-mode: vertical-rl;
  text-orientation: mixed;
  line-height: 2;
  padding: 20px;
  border: 1px solid #ddd;
}

.calendar-date {
  writing-mode: vertical-lr;
  text-combine-upright: digits 2;   /* Объединить 2 цифры даты */
  text-align: center;
  font-size: 1.5em;
}
```

## Влияние на другие CSS свойства

### Изменение поведения margin, padding, border

```css
.logical-properties {
  /* В writing-mode: horizontal-tb */
  margin-top: 10px;                 /* Верхний отступ */
  margin-right: 15px;               /* Правый отступ */
  margin-bottom: 10px;              /* Нижний отступ */
  margin-left: 15px;                /* Левый отступ */
  
  /* В writing-mode: vertical-rl */
  margin-block-start: 10px;         /* Начальный блочный отступ */
  margin-inline-end: 15px;          /* Конечный строчный отступ */
  margin-block-end: 10px;           /* Конечный блочный отступ */
  margin-inline-start: 15px;        /* Начальный строчный отступ */
}
```

### Изменение поведения float, clear

```css
.float-behavior {
  writing-mode: vertical-rl;
  /* Теперь float: left означает "в начало строки" */
  /* float: right означает "в конец строки" */
}

.float-element {
  float: left;                      /* В вертикальном режиме - в начало */
  float: right;                     /* В вертикальном режиме - в конец */
}
```

### Изменение поведения text-align

```css
.text-align-impact {
  writing-mode: vertical-rl;
  /* text-align: left теперь означает "в начало строки" */
  /* text-align: right теперь означает "в конец строки" */
  text-align: start;                /* Начало направления текста */
  text-align: end;                  /* Конец направления текста */
}
```

## CSS Writing Modes Module Level 4

Level 4 добавляет дополнительные возможности и улучшения.

### Улучшенные свойства ориентации

```css
.enhanced-orientation {
  /* Улучшенная ориентация текста */
  text-orientation: mixed;
  text-orientation: upright;
  text-orientation: sideways;
  text-orientation: sideways-right;
  
  /* Новое значение */
  text-orientation: isolate;        /* Изолированная ориентация */
  text-orientation: isolate-override; /* Переопределение изолированной ориентации */
}
```

### Свойства логических направлений

```css
.logical-directions {
  /* Логические свойства учитывают writing-mode */
  margin-block-start: 10px;         /* Отступ в начале блока */
  margin-block-end: 10px;           /* Отступ в конце блока */
  margin-inline-start: 15px;        /* Отступ в начале строки */
  margin-inline-end: 15px;          /* Отступ в конце строки */
  
  padding-block-start: 5px;
  padding-block-end: 5px;
  padding-inline-start: 10px;
  padding-inline-end: 10px;
  
  border-block-start: 1px solid #000;
  border-block-end: 1px solid #000;
  border-inline-start: 1px solid #000;
  border-inline-end: 1px solid #000;
}
```

## Практические применения

### Интерфейсы для восточноазиатских языков

```css
.chinese-vertical {
  writing-mode: vertical-rl;
  text-orientation: upright;
  font-family: "Noto Sans CJK SC", "Source Han Sans SC", serif;
  line-height: 2;
  letter-spacing: 0.1em;
}

.japanese-text {
  writing-mode: vertical-rl;
  text-orientation: mixed;
  font-family: "Noto Sans CJK JP", "Source Han Sans JP", serif;
  text-combine-upright: digits 2;   /* Для чисел в японском тексте */
}
```

### Дизайн с вертикальным текстом

```css
.book-binding {
  writing-mode: vertical-rl;
  text-align: justify;
  line-height: 2;
  letter-spacing: 0.2em;
  padding: 20px 10px;
  border-left: 2px solid #8B4513;
  font-family: serif;
}

.calendar-display {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  writing-mode: vertical-lr;
  text-align: center;
  font-size: 0.8em;
}

.calendar-day {
  padding: 5px;
  border: 1px solid #ccc;
  text-combine-upright: all;        /* Объединить день недели в одну ячейку */
}
```

### Интернационализация интерфейсов

```css
.rtl-interface {
  writing-mode: horizontal-tb;
  direction: rtl;
  text-align: right;
}

.rtl-layout {
  display: flex;
  flex-direction: row-reverse;      /* Реверсивный порядок элементов */
}

.ltr-override {
  direction: ltr;
  unicode-bidi: embed;
}

.bidi-isolate {
  unicode-bidi: isolate;            /* Изолировать двунаправленный текст */
}
```

### Адаптивные режимы письма

```css
.adaptive-writing-mode {
  /* Адаптация к языку пользователя */
  writing-mode: horizontal-tb;
}

@media (orientation: landscape) {
  .adaptive-writing-mode {
    writing-mode: vertical-rl;
  }
}

.vertical-on-mobile {
  writing-mode: horizontal-tb;
}

@media (max-width: 768px) {
  .vertical-on-mobile {
    writing-mode: vertical-lr;
    text-orientation: upright;
  }
}
```

## Совместимость с другими CSS модулями

### Flexbox в разных режимах письма

```css
.flex-writing-modes {
  writing-mode: vertical-rl;
  display: flex;
  flex-direction: column;           /* В вертикальном режиме - строки */
  /* flex-direction: row - в вертикальном режиме это столбцы */
}

.flex-item {
  flex: 1;
  /* Поведение flex изменяется в зависимости от writing-mode */
}
```

### Grid в разных режимах письма

```css
.grid-writing-modes {
  writing-mode: vertical-rl;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* В вертикальном режиме столбцы становятся строками */
}
```

### Позиционирование в разных режимах

```css
.positioned-writing-modes {
  writing-mode: vertical-rl;
  position: relative;
}

.positioned-child {
  position: absolute;
  top: 10px;                        /* В вертикальном режиме - сдвиг вниз */
  right: 20px;                      /* В вертикальном режиме - сдвиг влево */
  /* Использование логических свойств для независимости от режима */
  inset-block-start: 10px;          /* Логическое позиционирование */
  inset-inline-end: 20px;
}
```

## Поддержка браузерами

- `writing-mode: horizontal-tb`: Поддерживается во всех браузерах
- `writing-mode: vertical-rl`, `writing-mode: vertical-lr`: Хорошая поддержка в современных браузерах
- `text-orientation`: Хорошая поддержка, с ограничениями в некоторых браузерах
- `text-combine-upright`: Поддержка в процессе внедрения
- `direction` и `unicode-bidi`: Поддерживается во всех браузерах

## Связанные темы

- [[CSS Text Module]] - модуль текста
- [[CSS Logical Properties]] - логические свойства
- [[CSS Typography]] - типографика

## Заключение

CSS Writing Modes Module предоставляет мощные инструменты для создания интерфейсов, поддерживающих различные системы письма. Понимание режимов письма позволяет создавать интернационализированные интерфейсы, адекватно отображающиеся для пользователей с различными языковыми и культурными особенностями.
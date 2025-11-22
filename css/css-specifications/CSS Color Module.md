---
aliases: [CSS Color Specification, CSS Colors, CSS Color Levels, CSS Color Formats]
tags: [css, color, specification, css3, css4, css5]
---

# CSS Color Module: Уровни 3, 4, 5 - Полное руководство

## Введение

CSS Color Module - это спецификация, определяющая способы определения и использования цветов в CSS. Спецификация развивалась в нескольких уровнях: Level 3, Level 4 и Level 5, каждый из которых добавляет новые возможности и форматы цветов.

## CSS Color Module Level 3

Level 3 - это первый широко поддерживаемый уровень спецификации, включающий основные форматы цветов:

### Форматы цветов в Level 3

- `rgb()` и `rgba()` - цвет в RGB модели с альфа-каналом
- `hsl()` и `hsla()` - цвет в HSL модели с альфа-каналом
- Шестнадцатеричные значения (`#rrggbb`, `#rgb`)
- Ключевые слова цветов (например, `red`, `blue`, `transparent`)

```css
.element {
  color: rgb(255, 0, 0);           /* Красный */
  background-color: rgba(0, 0, 255, 0.5); /* Полупрозрачный синий */
  border-color: #ff0000;           /* Шестнадцатеричный красный */
  background: hsl(120, 100%, 50%); /* Зеленый в HSL */
}
```

## CSS Color Module Level 4

Level 4 добавляет более современные и гибкие форматы цветов:

### Новые форматы цветов в Level 4

- `hwb()` - модель Hue-Whiteness-Blackness
- `lab()` и `lch()` - цветовые модели CIELAB и CIELCH
- `oklab()` и `oklch()` - улучшенные цветовые модели
- Прозрачный цвет `transparent`
- Формат `rgb()` и `hsl()` без `a` параметра с альфа-каналом

```css
.element {
  /* HWB (Hue-Whiteness-Blackness) */
  background: hwb(120 40% 40%);
  
  /* LAB и LCH цвета */
  color: lab(62.25% 23.3 -45.6);
  border-color: lch(62.25% 51.2 300);
  
  /* OKLAB и OKLCH - улучшенные цветовые пространства */
  background: oklab(0.696 0.024 0.104);
  color: oklch(0.696 0.107 256.9);
  
  /* Новый формат RGB с альфа-каналом */
  background: rgb(255 0 0 / 50%);
  
  /* Новый формат HSL с альфа-каналом */
  border: hsl(0 100% 50% / 0.5);
}
```

## CSS Color Module Level 5

Level 5 продолжает развитие спецификации, добавляя новые возможности:

### Новые возможности в Level 5

- `color()` - функция для работы с цветовыми пространствами
- Поддержка новых цветовых пространств
- Функции для манипуляции цветами (`color-mix()`, `color-contrast()`)

```css
.element {
  /* Использование различных цветовых пространств */
  background: color(display-p3 0.4 0.8 0.2);
  color: color(rec2020 0.7 0.2 0.9);
  
  /* Манипуляции с цветами */
  border-color: color-mix(in srgb, red 20%, blue 80%);
  background: color-contrast(red vs blue, yellow, green);
}
```

## Градиенты и функции цветов

### Цветовые градиенты

CSS также поддерживает градиенты, которые могут использовать различные цветовые форматы:

```css
.gradient-box {
  background: linear-gradient(
    to right,
    rgb(255, 0, 0),
    hsl(120, 100%, 50%),
    oklch(0.696 0.107 256.9)
  );
}

.radial-gradient {
  background: radial-gradient(
    circle at center,
    lab(62.25% 23.3 -45.6),
    lch(62.25% 51.2 300)
  );
}
```

## Поддержка браузерами

- Level 3: Поддерживается во всех современных браузерах
- Level 4: Хорошая поддержка, особенно в Chrome, Firefox и Edge
- Level 5: Поддержка в процессе внедрения в браузерах

## Практические применения

### Цветовые палитры

Создание согласованных цветовых палитр:

```css
:root {
  --primary: oklch(0.696 0.107 256.9);
  --secondary: oklch(0.8 0.05 300);
  --accent: color-mix(in oklab, var(--primary) 70%, var(--secondary) 30%);
}
```

### Темы и адаптивные цвета

Использование цветов для создания тем:

```css
/* Светлая тема */
:root {
  --bg-color: oklab(0.95 0.001 0.818);
  --text-color: oklab(0.2 0.03 -2.32);
}

/* Темная тема */
[data-theme="dark"] {
  --bg-color: oklab(0.2 0.03 -2.32);
  --text-color: oklab(0.95 0.001 0.818);
}
```

## Связанные темы

- [[CSS Color Adjust]] - корректировка цвета для темной темы
- [[CSS Values and Units]] - значения и единицы измерения
- [[CSS Custom Properties Module]] - кастомные свойства (переменные)

## Заключение

CSS Color Module продолжает развиваться, предоставляя разработчикам более гибкие и точные инструменты для работы с цветом. Понимание различных уровней спецификации позволяет использовать наиболее подходящие форматы цветов для конкретных задач.
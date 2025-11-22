---
aliases: [CSS Color Functions, Color Functions, Цветовые функции]
tags: [css, advanced, colors]
---

# CSS Color Functions - Полное руководство

## Введение

CSS Color Functions - это набор современных функций, которые позволяют динамически определять и манипулировать цветами в CSS. Эти функции расширяют возможности традиционных цветовых форматов, позволяя создавать более гибкие и адаптивные цветовые схемы. Современные цветовые функции включают `color-mix()`, `color-contrast()`, а также расширенные возможности работы с цветовыми пространствами.

## Современные цветовые функции

### color-mix()

Функция `color-mix()` позволяет смешивать два цвета в заданной пропорции. Это особенно полезно для создания цветовых вариантов, темизации и адаптивных цветов.

#### Синтаксис

```css
color-mix(in <color-space>, <color> <percentage>, <color> <percentage>)
```

- `in <color-space>` - цветовое пространство для смешивания (например, srgb, hsl, hwb, lab, oklab, oklch)
- `<color>` - цвета для смешивания
- `<percentage>` - процентное соотношение каждого цвета

#### Примеры использования

```css
/* Простое смешивание цветов */
.element {
  /* 80% синего и 20% белого */
  background-color: color-mix(in srgb, blue 80%, white 20%);
}

/* Использование с CSS переменными */
:root {
  --primary: #3498db;
  --secondary: #2ecc71;
}

.highlight {
  background-color: color-mix(in srgb, var(--primary) 60%, var(--secondary) 40%);
}

/* Смешивание с прозрачностью */
.transparent-overlay {
  background-color: color-mix(in srgb, rgba(0, 0, 0, 0.5) 50%, #ffffff 50%);
}

/* Работа с цветовым пространством OKLCH */
.gradient-color {
  background-color: color-mix(in oklch, #ff0000, #0000ff);
}
```

#### Практическое применение

Цветовая миксика особенно полезна при создании систем цветов:

```css
:root {
  --primary: #3498db;
  --surface: #ffffff;
  --on-surface: #000000;
}

/* Создание вариантов основного цвета */
.card {
  background-color: color-mix(in srgb, var(--primary) 10%, white);
  border-color: color-mix(in srgb, var(--primary) 50%, white);
}

/* Адаптация текста к фону */
.text-on-primary {
  color: color-mix(in srgb, var(--on-surface) 80%, transparent);
}
```

### color-contrast()

Функция `color-contrast()` выбирает наиболее контрастный цвет из предоставленного списка, основываясь на фоновом цвете. Это позволяет автоматически выбирать правильный цвет текста в зависимости от фона.

#### Синтаксис

```css
color-contrast(<background-color> vs <color> [, <color>]#)
```

#### Примеры использования

```css
/* Выбор наиболее контрастного цвета между черным и белым */
.text {
  color: color-contrast(v #3498db vs white, black);
}

/* С выбором из нескольких цветов */
.accessible-text {
  color: color-contrast(v #ff6b6b vs #333333, #666666, white);
}

/* Использование с переменными */
.card {
  background-color: var(--card-bg, #f0f0f0);
  color: color-contrast(v var(--card-bg) vs #000000, #ffffff);
}
```

## Современные цветовые пространства

### OKLCH и OKLAB

Современные цветовые пространства OKLCH и OKLAB обеспечивают более perceptually uniform (воспринимаемо однородное) распределение цветов, что делает градиенты и цветовые переходы более естественными.

#### OKLCH (Oklab Lightness Chroma Hue)

OKLCH использует:
- L (lightness) - светлота от 0 до 1
- C (chroma) - насыщенность
- H (hue) - оттенок в градусах

```css
.oklch-example {
  /* Ярко-красный цвет в пространстве OKLCH */
  background-color: oklch(0.7 0.25 28.3);
}

.oklch-gradient {
  background: linear-gradient(
    to right,
    oklch(0.5 0.1 0),    /* Темный оттенок */
    oklch(0.9 0.1 0)     /* Светлый оттенок */
  );
}
```

#### OKLAB

OKLAB использует три координаты, основанные на восприятии:
- L - светлота
- a - зеленый/красный спектр
- b - синий/желтый спектр

```css
.oklab-example {
  background-color: oklab(0.5 0.2 0.1);
}
```

### LAB и LCH

LAB и LCH - более ранние perceptually uniform цветовые пространства, которые также поддерживаются в современном CSS.

```css
.lab-example {
  background-color: lab(53.24 -4 25);  /* Более зеленоватый цвет */
}

.lch-example {
  background-color: lch(52.2345% 72.2 56.2);  /* Желтовато-зеленый */
}
```

## Практические примеры применения

### Создание адаптивной дизайн-системы

```css
:root {
  --primary: oklch(0.7 0.25 28.3);        /* Основной цвет */
  --primary-light: color-mix(in oklch, var(--primary), white 20%);
  --primary-dark: color-mix(in oklch, var(--primary), black 20%);
  --surface: oklch(0.95 0.01 28.3);       /* Поверхностный цвет */
  --on-surface: oklch(0.2 0.01 28.3);     /* Цвет текста на поверхности */
}

/* Автоматическая адаптация цвета текста */
.card {
  background-color: var(--surface);
  color: color-contrast(v var(--surface) vs var(--on-surface), white);
}

.button {
  background-color: var(--primary);
  color: color-contrast(v var(--primary) vs white, black);
}

.button:hover {
  background-color: var(--primary-light);
  color: color-contrast(v var(--primary-light) vs white, black);
}
```

### Темизация с помощью цветовых функций

```css
/* Светлая тема (по умолчанию) */
:root {
  --bg-lightness: 0.95;
  --text-lightness: 0.2;
  --accent-hue: 28.3;
}

/* Темная тема */
[data-theme="dark"] {
  --bg-lightness: 0.15;
  --text-lightness: 0.9;
}

/* Использование переменных для динамического определения цветов */
body {
  background-color: oklch(from var(--bg-lightness) l c var(--accent-hue));
  color: oklch(from var(--text-lightness) l c var(--accent-hue));
}

.accent {
  color: oklch(0.7 0.25 var(--accent-hue));
}
```

### Адаптация цветов под доступность

```css
/* Создание контрастных цветовых пар */
.accessible {
  /* Выбор цвета с наивысшим контрастом к фону */
  color: color-contrast(v oklch(0.8 0.05 28.3) vs #000000, #333333, #666666);
}

/* Генерация шкалы цветов для уведомлений */
.notification {
  --error-base: oklch(0.6 0.25 0);
  --error-light: color-mix(in oklch, var(--error-base), oklch(0.95 0.01 28.3) 30%);
  --error-dark: color-mix(in oklch, var(--error-base), black 20%);
  
  background-color: var(--error-light);
  border-color: var(--error-dark);
}
```

## Совместимость с существующими подходами

### Использование вместе с CSS переменными

Цветовые функции отлично сочетаются с CSS переменными:

```css
:root {
  /* Базовые цвета */
  --primary-hsl: 210 70% 50%;
  --secondary-hsl: 120 50% 40%;
  
  /* Производные цвета с использованием цветовых функций */
  --primary: hsl(var(--primary-hsl));
  --primary-light: color-mix(in hsl, hsl(var(--primary-hsl)), white 30%);
  --primary-dark: color-mix(in hsl, hsl(var(--primary-hsl)), black 20%);
}
```

### Резервные значения для старых браузеров

```css
.element {
  /* Резервный цвет для браузеров без поддержки цветовых функций */
  background-color: #3498db;
  /* Современный цветовой подход */
  background-color: color-mix(in srgb, #3498db 80%, white 20%);
}
```

## Лучшие практики

1. **Используйте perceptually uniform цветовые пространства** (OKLCH, OKLAB) для создания естественных градиентов и цветовых шкал.

2. **Создавайте цветовые варианты с помощью `color-mix()`** вместо жестко заданных цветовых кодов.

3. **Автоматизируйте контрастность с помощью `color-contrast()`** для улучшения доступности.

4. **Разделяйте определение цветов и их использование** с помощью CSS переменных.

5. **Предоставляйте резервные значения** для браузеров без поддержки новых функций.

## Поддержка браузерами

На момент 2023 года:

- `color-mix()` поддерживается в Chrome 111+, Firefox 113+, Safari 15+
- `color-contrast()` поддерживается в Chrome 111+
- Цветовые пространства OKLCH, OKLAB поддерживаются в Chrome 111+, Firefox 113+, Safari 15.4+

Для браузеров без поддержки рекомендуется использовать резервные стили или JavaScript-полифиллы.

## Заключение

CSS Color Functions представляют собой важный шаг вперед в развитии веб-дизайна, позволяя создавать более гибкие, доступные и воспринимаемо однородные цветовые решения. Эти функции особенно ценны при создании дизайн-систем, темизации интерфейсов и обеспечении доступности. Правильное использование цветовых функций позволяет значительно упростить управление цветами и создать более качественные визуальные решения.

## Смежные темы

- [[colors]]
- [[color-mix]]
- [[css-architecture]]
- [[accessibility]]

#css #color #color-functions #frontend #design-system
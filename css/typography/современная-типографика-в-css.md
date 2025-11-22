---
aliases: ["Современная CSS типографика", "CSS font-variation", "CSS font-palette", "Переменные шрифты"]
tags: ["css", "typography", "font-variation", "font-palette", "variable-fonts", "fonts"]
---

# Современная типографика в CSS: font-variation, font-palette

## Введение

Современная типографика в CSS включает в себя передовые возможности, такие как переменные шрифты (variable fonts) и палитры шрифтов (font palettes). Эти технологии позволяют создавать более гибкие и адаптивные текстовые интерфейсы с меньшими затратами на загрузку ресурсов.

## Переменные шрифты (Variable Fonts)

### Что такое переменные шрифты

Переменные шрифты - это формат шрифта, который содержит множество вариантов одного шрифта в одном файле. Вместо отдельных файлов для каждого начертания (жирный, курсив, разные размеры), переменный шрифт позволяет плавно изменять характеристики шрифта с помощью CSS.

### Основы использования

```css
/* Определение переменного шрифта */
@font-face {
  font-family: 'MyVariableFont';
  src: url('my-variable-font.woff2') format('woff2-variations');
  font-weight: 100 900; /* Диапазон толщины */
  font-stretch: 25% 151%; /* Диапазон ширины */
  font-style: normal;
}

/* Использование переменного шрифта */
.variable-text {
  font-family: 'MyVariableFont', sans-serif;
  font-weight: 450; /* Промежуточное значение между 400 и 500 */
  font-stretch: 110%; /* Растянутый вариант */
}
```

### Стандартные оси переменных шрифтов

#### Ось толщины (weight) - `wght`

```css
.weight-variations {
  /* Плавное изменение толщины */
  font-weight: 350; /* Промежуточное значение */
}

.weight-animation {
  font-weight: 400;
  transition: font-weight 0.3s ease;
}

.weight-animation:hover {
  font-weight: 700;
}
```

#### Ось ширины (width) - `wdth`

```css
.width-variations {
  /* Изменение ширины начертания */
  font-stretch: 75%; /* Узкое начертание */
}

.width-responsive {
  /* Адаптивная ширина */
  font-stretch: clamp(75%, 5vw, 125%);
}
```

#### Ось курсива (slant) - `slnt`

```css
.slant-variations {
  /* Угол наклона */
  font-style: oblique 12deg; /* Наклон 12 градусов */
}

.slant-animation {
  font-style: oblique 0deg;
  transition: font-style 0.3s ease;
}

.slant-animation:hover {
  font-style: oblique 15deg;
}
```

### Пользовательские оси

Многие переменные шрифты включают пользовательские оси:

```css
/* Использование пользовательской оси */
@font-face {
  font-family: 'CustomVariableFont';
  src: url('custom-variable-font.woff2') format('woff2-variations');
  /* Определение пользовательских осей */
  font-variation-settings: 'XTRA' 400, 'YOPQ' 100;
}

.custom-axis {
  font-family: 'CustomVariableFont';
  /* Изменение пользовательских осей */
  font-variation-settings: 'XTRA' 500, 'YOPQ' 80, 'GRAD' 88;
}

/* Комбинация стандартных и пользовательских осей */
.combined-axes {
  font-family: 'CustomVariableFont';
  font-weight: 500;
  font-variation-settings: 'opsz' 14, 'GRAD' 88;
}
```

### Практические примеры использования

#### Адаптивная типографика

```css
/* Адаптивный заголовок */
.responsive-heading {
  font-family: 'VariableHeadline', sans-serif;
  font-weight: 750; /* Очень жирный для заголовков */
  font-stretch: 105%; /* Легко растянутый */
  font-size: clamp(1.5rem, 4vw, 3rem);
  /* Плавная адаптация толщины под размер */
  font-variation-settings: 'opsz' calc(10 + (1rem - 10) * 2);
}

/* Адаптивный основной текст */
.responsive-body {
  font-family: 'VariableBody', serif;
  font-weight: 380; /* Промежуточное значение для удобства чтения */
  font-size: clamp(1rem, 2.5vw, 1.2rem);
  line-height: 1.6;
}
```

#### Интерактивные эффекты

```css
/* Интерактивное изменение начертания */
.interactive-text {
  font-family: 'InteractiveFont';
  font-weight: 400;
  font-variation-settings: 'slnt' 0;
  transition: font-weight 0.3s, font-variation-settings 0.3s;
}

.interactive-text:hover {
  font-weight: 600;
  font-variation-settings: 'slnt' 10;
}

/* Анимация осей */
.animated-text {
  font-family: 'AnimatedFont';
  font-weight: 400;
  animation: axisVariation 3s infinite alternate ease-in-out;
}

@keyframes axisVariation {
  0% { 
    font-weight: 300; 
    font-variation-settings: 'wdth' 75; 
  }
  100% { 
    font-weight: 700; 
    font-variation-settings: 'wdth' 125; 
  }
}
```

## Палитры шрифтов (Font Palettes)

### Что такое палитры шрифтов

Палитры шрифтов позволяют изменять цвета глифов в цветных шрифтах без необходимости создания отдельных версий шрифта. Это особенно полезно для эмотиконов и других цветных символов.

### Использование font-palette

```css
/* Определение цветного переменного шрифта с палитрами */
@font-face {
  font-family: 'ColorFont';
  src: url('color-font.woff2') format('woff2');
  font-palette: light, dark;
}

/* Использование стандартных палитр */
.color-text-light {
  font-family: 'ColorFont';
  font-palette: light;
}

.color-text-dark {
  font-family: 'ColorFont';
  font-palette: dark;
}

/* Создание пользовательских палитр */
@font-palette-values --my-palette {
  font-family: 'ColorFont';
  base-palette: 0; /* Использовать базовую палитру как основу */
  override-colors: 
    0 red,
    1 blue,
    2 green;
}

.custom-palette-text {
  font-family: 'ColorFont';
  font-palette: --my-palette;
}
```

### Практические примеры палитр

```css
/* Темизированные эмотиконы */
.emoji-container {
  font-family: 'ColorEmoji';
  font-size: 2rem;
}

.light-theme .emoji-container {
  font-palette: light;
}

.dark-theme .emoji-container {
  font-palette: dark;
}

/* Создание тематических палитр */
@font-palette-values --summer-palette {
  font-family: 'ColorFont';
  base-palette: 0;
  override-colors: 
    0 #ff6b6b,    /* Красный */
    1 #4ecdc4,    /* Бирюзовый */
    2 #45b7d1,    /* Синий */
    3 #96ceb4;    /* Мятный */
}

@font-palette-values --winter-palette {
  font-family: 'ColorFont';
  base-palette: 0;
  override-colors: 
    0 #3498db,    /* Синий */
    1 #2c3e50,    /* Темно-синий */
    2 #ecf0f1,    /* Светло-серый */
    3 #7f8c8d;    /* Серый */
}

.summer-text {
  font-family: 'ColorFont';
  font-palette: --summer-palette;
}

.winter-text {
  font-family: 'ColorFont';
  font-palette: --winter-palette;
}
```

## Совмещение возможностей

### Переменные шрифты с палитрами

```css
/* Переменный цветной шрифт */
@font-face {
  font-family: 'VarColorFont';
  src: url('variable-color-font.woff2') format('woff2-variations');
  font-weight: 100 900;
  font-stretch: 75% 125%;
}

@font-palette-values --brand-palette {
  font-family: 'VarColorFont';
  base-palette: 0;
  override-colors: 
    0 var(--primary-color),
    1 var(--secondary-color),
    2 var(--accent-color);
}

.var-color-text {
  font-family: 'VarColorFont';
  font-weight: var(--dynamic-weight, 400);
  font-palette: --brand-palette;
}
```

## Адаптивные типографские системы

### Использование CSS-переменных с переменными шрифтами

```css
:root {
  /* Базовые значения для типографской системы */
  --heading-weight: 750;
  --body-weight: 380;
  --text-stretch: 100%;
  --font-size-multiplier: 1;
}

/* Адаптивная типографская система */
.adaptive-typography {
  font-family: 'SystemVariableFont';
  font-weight: var(--heading-weight);
  font-stretch: var(--text-stretch);
  font-size: calc(1rem * var(--font-size-multiplier));
}

/* Адаптация под размер экрана */
@media (max-width: 768px) {
  :root {
    --font-size-multiplier: 0.95;
    --heading-weight: 700;
  }
}

@media (prefers-contrast: high) {
  :root {
    --heading-weight: 800;
    --body-weight: 450;
  }
}
```

### Создание типографских шкал

```css
/* Типографская шкала с переменными шрифтами */
.typographic-scale {
  --step: 0;
  --base-size: 1rem;
  --scale-factor: 1.2;
  --calculated-size: calc(var(--base-size) * pow(var(--scale-factor), var(--step)));
  
  font-family: 'VariableFont';
  font-weight: calc(300 + (var(--step) * 100));
  font-size: calc(var(--base-size) * pow(var(--scale-factor), var(--step)));
}

/* Использование с конкретными классами */
.h1-typography {
  --step: 3;
  font-weight: 750;
}

.h2-typography {
  --step: 2;
  font-weight: 700;
}

.body-typography {
  --step: 0;
  font-weight: 380;
}

.small-typography {
  --step: -1;
  font-weight: 300;
}
```

## Совместимость и резервные значения

```css
/* Резервные значения для старых браузеров */
.modern-typography {
  /* Резервный шрифт */
  font-family: 'Arial', sans-serif;
  
  /* Современный переменный шрифт */
  font-family: 'ModernVariableFont', 'Arial', sans-serif;
  
  /* Резервные значения для осей */
  font-weight: 400;
  
  /* Современные значения */
  font-weight: 450;
  font-stretch: 105%;
  font-variation-settings: 'opsz' 14;
}

/* Условия поддержки */
@supports (font-variation-settings: 'wght' 400) {
  .variable-font-enhanced {
    font-weight: 450;
    font-variation-settings: 'wght' 450, 'wdth' 105;
  }
}

@supports (font-palette: light) {
  .palette-enhanced {
    font-palette: light;
  }
}
```

## Оптимизация производительности

### Загрузка переменных шрифтов

```css
/* Оптимизированная загрузка переменных шрифтов */
@font-face {
  font-family: 'OptimizedVariableFont';
  src: url('optimized-font.woff2') format('woff2-variations');
  font-weight: 100 900;
  font-display: swap; /* Для лучшей производительности */
}

/* Предзагрузка критических значений */
.critical-typography {
  font-family: 'OptimizedVariableFont';
  /* Использование наиболее часто используемых значений */
  font-weight: 400;
  font-variation-settings: 'opsz' 16;
}
```

### Анимация осей с оптимизацией

```css
/* Оптимизированная анимация осей */
.optimized-animation {
  font-family: 'AnimatedVariableFont';
  font-weight: 400;
  /* Использование transform для анимации, когда возможно */
  will-change: font-weight;
  transition: font-weight 0.3s ease;
}

.optimized-animation:hover {
  font-weight: 600;
}
```

## Практические примеры

### Адаптивный логотип

```css
.logo-text {
  font-family: 'LogoVariableFont';
  font-weight: 750;
  font-stretch: 110%;
  font-size: clamp(1.5rem, 4vw, 3rem);
  letter-spacing: calc(0.01em * (1 - (1rem - 16) * 0.1));
  transition: font-variation-settings 0.3s ease;
}

.logo-text:hover {
  font-variation-settings: 'wght' 800, 'wdth' 115;
}
```

### Интерактивная статья с изменяющейся типографикой

```css
.article-container {
  --reading-comfort: 380;
  --focus-level: 100%;
}

.article-text {
  font-family: 'ArticleVariableFont';
  font-weight: var(--reading-comfort);
  font-stretch: var(--focus-level);
  font-size: clamp(1rem, 2.5vw, 1.1rem);
  line-height: 1.6;
  transition: font-variation-settings 0.2s ease;
}

/* Увеличение удобства чтения при фокусе */
.article-text:focus {
  --reading-comfort: 400;
  --focus-level: 102%;
}
```

## Заключение

Современная типографика в CSS с использованием переменных шрифтов и палитр шрифтов открывает новые возможности для создания адаптивных, интерактивных и визуально привлекательных текстовых интерфейсов. Эти технологии позволяют значительно сократить объем загружаемых ресурсов при этом обеспечивая большую гибкость в оформлении текста.

При использовании этих возможностей важно учитывать совместимость с браузерами, обеспечивать резервные значения и оптимизировать производительность. Также необходимо тестировать типографские решения на разных устройствах и с учетом потребностей пользователей с особыми требованиями.

## См. также

- [[адаптивная-типографика]]
- [[подстановка-шрифтов-и-производительность]]
- [[CSS-типографика]]
- [[CSS-шрифты]]
- [[цветовые-пространства-и-форматы]]
- [[цветовые-функции-и-их-применение]]
- [[работа-с-цветовыми-темами-и-контрастностью]]
- [[обзор-css-функций]]
- [[продвинутые-функции-css]]
- [[практическое-применение-функций-в-адаптивном-дизайне]]
- [[2d-и-3d-трансформации]]
- [[перспектива-и-3d-пространство-в-css]]
- [[трансформации-и-производительность]]
- [[css-фильтры-применение-и-оптимизация]]
- [[css-формы-и-обтекание]]
- [[комбинирование-фильтров-и-создание-визуальных-эффектов]]
---
aliases: ["CSS фильтры", "CSS filter", "Визуальные эффекты CSS"]
tags: ["css", "filters", "effects", "blur", "brightness", "contrast"]
---

# CSS-фильтры: применение и оптимизация

## Введение

CSS-фильтры предоставляют возможность применения визуальных эффектов к элементам без использования JavaScript или изображений. Они позволяют создавать размытие, изменять яркость, контрастность, насыщенность и другие визуальные эффекты, что делает их мощным инструментом для улучшения пользовательского интерфейса.

## Основы CSS-фильтров

### Свойство filter

Свойство `filter` позволяет применять графические эффекты к элементу, изменяя его визуальное представление. Оно принимает одну или несколько функций фильтра, которые можно комбинировать.

```css
.basic-filter {
  filter: blur(5px);
  filter: brightness(1.2);
  filter: contrast(1.5);
}
```

## Функции фильтров

### blur()

Функция `blur()` применяет размытие по Гауссу к элементу. Значение определяет радиус размытия.

```css
/* Разное количество размытия */
.blur-light {
  filter: blur(2px);
}

.blur-medium {
  filter: blur(5px);
}

.blur-heavy {
  filter: blur(10px);
}

/* Адаптивное размытие */
.blur-responsive {
  filter: blur(calc(1px + 1vw));
}
```

### brightness()

Функция `brightness()` изменяет яркость элемента. Значение 1 оставляет изображение без изменений, значения больше 1 увеличивают яркость, меньше 1 - уменьшают.

```css
/* Изменение яркости */
.brightness-normal {
  filter: brightness(1); /* Без изменений */
}

.brightness-high {
  filter: brightness(1.5); /* Увеличение яркости */
}

.brightness-low {
  filter: brightness(0.7); /* Уменьшение яркости */
}

.brightness-zero {
  filter: brightness(0); /* Полностью черное */
}

.brightness-high-value {
  filter: brightness(3); /* Сильно увеличенная яркость */
}
```

### contrast()

Функция `contrast()` изменяет контрастность элемента. Значение 1 оставляет изображение без изменений, значения больше 1 увеличивают контраст, меньше 1 - уменьшают.

```css
/* Изменение контрастности */
.contrast-normal {
  filter: contrast(1); /* Без изменений */
}

.contrast-high {
  filter: contrast(1.8); /* Увеличение контраста */
}

.contrast-low {
  filter: contrast(0.5); /* Уменьшение контраста */
}

.contrast-zero {
  filter: contrast(0); /* Оттенки серого */
}
```

### grayscale()

Функция `grayscale()` преобразует изображение в оттенки серого. Значение 0 оставляет цвета без изменений, 1 - полностью преобразует в оттенки серого.

```css
/* Преобразование в оттенки серого */
.grayscale-none {
  filter: grayscale(0); /* Без изменений */
}

.grayscale-partial {
  filter: grayscale(0.5); /* 50% оттенки серого */
}

.grayscale-full {
  filter: grayscale(1); /* Полностью оттенки серого */
}

/* Адаптивное преобразование */
.grayscale-responsive {
  filter: grayscale(calc(var(--grayscale-level, 0) * 1%));
}
```

### hue-rotate()

Функция `hue-rotate()` поворачивает цвета в элементе на заданный угол в цветовом круге.

```css
/* Поворот оттенка */
.hue-normal {
  filter: hue-rotate(0deg); /* Без изменений */
}

.hue-90 {
  filter: hue-rotate(90deg); /* Поворот на 90 градусов */
}

.hue-180 {
  filter: hue-rotate(180deg); /* Поворот на 180 градусов */
}

.hue-270 {
  filter: hue-rotate(270deg); /* Поворот на 270 градусов */
}

.hue-full {
  filter: hue-rotate(360deg); /* Полный оборот */
}
```

### invert()

Функция `invert()` инвертирует цвета элемента. Значение 0 оставляет цвета без изменений, 1 - полностью инвертирует.

```css
/* Инверсия цветов */
.invert-none {
  filter: invert(0); /* Без изменений */
}

.invert-partial {
  filter: invert(0.3); /* Частичная инверсия */
}

.invert-full {
  filter: invert(1); /* Полная инверсия */
}
```

### opacity()

Функция `opacity()` изменяет прозрачность элемента. Работает аналогично свойству `opacity`, но как часть фильтра.

```css
/* Изменение прозрачности через фильтр */
.opacity-normal {
  filter: opacity(1); /* Полностью непрозрачный */
}

.opacity-transparent {
  filter: opacity(0.5); /* 50% прозрачности */
}

.opacity-hidden {
  filter: opacity(0); /* Полностью прозрачный */
}
```

### saturate()

Функция `saturate()` изменяет насыщенность цветов. Значение 1 оставляет насыщенность без изменений, значения больше 1 увеличивают, меньше 1 - уменьшают.

```css
/* Изменение насыщенности */
.saturate-normal {
  filter: saturate(1); /* Без изменений */
}

.saturate-high {
  filter: saturate(2); /* Удвоенная насыщенность */
}

.saturate-low {
  filter: saturate(0.5); /* Половинная насыщенность */
}

.saturate-zero {
  filter: saturate(0); /* Оттенки серого */
}

.saturate-high-value {
  filter: saturate(5); /* Сильно насыщенные цвета */
}
```

### sepia()

Функция `sepia()` применяет сепию к элементу. Значение 0 оставляет цвета без изменений, 1 - полностью преобразует в сепию.

```css
/* Применение сепии */
.sepia-none {
  filter: sepia(0); /* Без изменений */
}

.sepia-partial {
  filter: sepia(0.5); /* Частичная сепия */
}

.sepia-full {
  filter: sepia(1); /* Полная сепия */
}
```

## Комбинирование фильтров

Фильтры можно комбинировать в одном свойстве `filter`:

```css
/* Комбинация нескольких фильтров */
.combined-filters {
  filter: 
    blur(2px) 
    brightness(1.1) 
    contrast(1.2) 
    saturate(1.3);
}

/* Более сложная комбинация */
.complex-filters {
  filter: 
    grayscale(0.3) 
    hue-rotate(15deg) 
    sepia(0.2) 
    opacity(0.9);
}

/* Адаптивные комбинированные фильтры */
.adaptive-filters {
  filter: 
    blur(calc(var(--blur-amount, 0) * 1px)) 
    brightness(calc(1 + var(--brightness-amount, 0) * 0.1)) 
    contrast(calc(1 + var(--contrast-amount, 0) * 0.1));
}
```

## Практические примеры использования

### Эффект размытия фона

```css
/* Эффект размытия фона при наведении */
.blur-on-hover {
  transition: filter 0.3s ease;
}

.blur-on-hover:hover {
  filter: blur(3px);
}

/* Эффект размытия фона для модальных окон */
.modal-backdrop {
  filter: blur(5px);
  transition: filter 0.3s ease;
}

.modal-open .modal-backdrop {
  filter: blur(0);
}
```

### Темизация изображений

```css
/* Темизация изображений для темной темы */
[data-theme="dark"] .image {
  filter: 
    brightness(0.9) 
    contrast(1.1) 
    saturate(0.9);
}

/* Адаптация изображений под цветовую слепоту */
.protanopia-mode .image {
  filter: 
    hue-rotate(30deg) 
    saturate(1.5);
}
```

### Создание визуальных эффектов

```css
/* Эффект "холодного" света */
.cool-light-effect {
  filter: 
    hue-rotate(-30deg) 
    saturate(1.2) 
    brightness(1.1);
}

/* Эффект "теплого" света */
.warm-light-effect {
  filter: 
    hue-rotate(20deg) 
    saturate(1.3) 
    brightness(1.05);
}

/* Эффект старой фотографии */
.vintage-effect {
  filter: 
    sepia(0.5) 
    contrast(1.2) 
    saturate(0.8) 
    hue-rotate(-10deg);
}
```

### Анимация фильтров

Фильтры можно анимировать с помощью CSS-анимаций и переходов:

```css
/* Анимация размытия */
.blur-animation {
  filter: blur(0);
  transition: filter 0.5s ease-in-out;
}

.blur-animation:hover {
  filter: blur(5px);
}

/* Пульсация яркости */
.brightness-pulse {
  animation: pulseBrightness 2s infinite alternate;
}

@keyframes pulseBrightness {
  0% { filter: brightness(1); }
  100% { filter: brightness(1.3); }
}

/* Анимация оттенка */
.hue-rotation {
  animation: rotateHue 4s infinite linear;
}

@keyframes rotateHue {
  from { filter: hue-rotate(0deg); }
  to { filter: hue-rotate(360deg); }
}
```

## Продвинутые эффекты с CSS-переменными

```css
/* Адаптивные фильтры с CSS-переменными */
.adaptive-filter {
  --blur-amount: 0;
  --brightness-amount: 1;
  --contrast-amount: 1;
  --saturate-amount: 1;
  
  filter: 
    blur(calc(var(--blur-amount) * 1px)) 
    brightness(var(--brightness-amount)) 
    contrast(var(--contrast-amount)) 
    saturate(var(--saturate-amount));
}

/* Пример изменения переменных */
.adaptive-filter.effect-1 {
  --blur-amount: 3;
  --brightness-amount: 1.2;
}

.adaptive-filter.effect-2 {
  --contrast-amount: 1.5;
  --saturate-amount: 0.7;
}
```

## Применение к различным элементам

### Фильтры для изображений

```css
.image-filter {
  filter: 
    contrast(1.1) 
    saturate(1.2);
  transition: filter 0.3s ease;
}

.image-filter:hover {
  filter: 
    contrast(1.3) 
    saturate(1.4) 
    brightness(1.1);
}
```

### Фильтры для текста

```css
.text-filter {
  filter: drop-shadow(2px 2px 4px rgba(0,0,0,0.3));
  font-size: 2rem;
}

/* Комбинация с text-shadow */
.text-with-filter {
  text-shadow: 0 0 5px rgba(255,255,255,0.5);
  filter: contrast(1.2) saturate(1.1);
}
```

### Фильтры для SVG

```css
.svg-filter {
  filter: 
    drop-shadow(0 0 3px rgba(0,0,0,0.3)) 
    hue-rotate(180deg);
}

.svg-filter:hover {
  filter: 
    drop-shadow(0 0 6px rgba(0,0,0,0.5)) 
    hue-rotate(0deg);
}
```

## Совместимость и резервные значения

```css
/* Резервные значения для старых браузеров */
.filter-fallback {
  /* Резервный стиль */
  opacity: 0.8;
  
  /* Современный фильтр */
  filter: 
    opacity(0.8) 
    contrast(1.2) 
    saturate(1.1);
}

/* Условия для поддержки фильтров */
@supports (filter: blur(1px)) {
  .filter-enhanced {
    filter: blur(2px) brightness(1.1);
  }
}
```

## Оптимизация производительности

### Создание отдельных слоев для фильтров

```css
/* Активация аппаратного ускорения для фильтров */
.optimized-filter {
  filter: blur(3px);
  /* Создание отдельного слоя для оптимизации */
  will-change: filter;
  /* Альтернативный способ активации GPU */
  transform: translateZ(0);
}

/* Оптимизация анимации фильтров */
.animated-filter {
  filter: blur(0);
  transition: filter 0.3s ease;
  will-change: filter;
}

.animated-filter:hover {
  filter: blur(5px);
}
```

### Ограничение сложности фильтров

```css
/* Упрощенные фильтры для мобильных устройств */
@media (max-width: 768px) {
  .mobile-optimized {
    filter: brightness(1.1); /* Простой фильтр вместо сложного */
  }
}

/* Учет предпочтений пользователя */
@media (prefers-reduced-motion: reduce) {
  .motion-reduced {
    filter: none;
    transition: none;
  }
}
```

## Практические примеры

### Интерактивная галерея с фильтрами

```css
.gallery-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

.gallery-item {
  transition: filter 0.3s ease;
  filter: 
    contrast(1.1) 
    saturate(1.1) 
    brightness(1.05);
}

.gallery-item:hover {
  filter: 
    contrast(1.2) 
    saturate(1.3) 
    brightness(1.1) 
    drop-shadow(0 10px 20px rgba(0,0,0,0.2));
  transform: scale(1.03);
}
```

### Темизация интерфейса с помощью фильтров

```css
.interface-theme {
  --theme-filter: none;
  filter: var(--theme-filter);
  transition: filter 0.3s ease;
}

.theme-cool {
  --theme-filter: 
    hue-rotate(-10deg) 
    contrast(1.1) 
    saturate(0.9);
}

.theme-warm {
  --theme-filter: 
    hue-rotate(15deg) 
    contrast(1.05) 
    saturate(1.2);
}

.theme-contrast {
  --theme-filter: 
    contrast(1.3) 
    brightness(0.95);
}
```

## Заключение

CSS-фильтры предоставляют мощный инструмент для создания визуальных эффектов без использования JavaScript или изображений. Они позволяют изменять яркость, контрастность, насыщенность, применять размытие и другие эффекты к любым элементам на странице.

При использовании фильтров важно учитывать производительность, особенно при анимации. Сложные комбинации фильтров могут замедлить рендеринг, особенно на мобильных устройствах. Также важно обеспечивать резервные значения для браузеров без поддержки фильтров и учитывать предпочтения пользователей.

## См. также

- [[css-фильтры-и-производительность]]
- [[комбинирование-фильтров-и-создание-визуальных-эффектов]]
- [[CSS-эффекты]]
- [[CSS-анимации]]
- [[цветовые-пространства-и-форматы]]
- [[цветовые-функции-и-их-применение]]
- [[работа-с-цветовыми-темами-и-контрастностью]]
- [[обзор-css-функций]]
- [[продвинутые-функции-css]]
- [[практическое-применение-функций-в-адаптивном-дизайне]]
- [[2d-и-3d-трансформации]]
- [[перспектива-и-3d-пространство-в-css]]
- [[трансформации-и-производительность]]
- [[css-формы-и-обтекание]]
- [[современная-типографика-в-css]]
- [[адаптивная-типографика]]
- [[подстановка-шрифтов-и-производительность]]
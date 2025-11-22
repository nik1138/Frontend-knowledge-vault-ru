---
aliases: [CSS Filters Specification, CSS Filter Effects, CSS Image Filters, CSS Filter Functions]
tags: [css, filters, effects, specification, css-filters]
---

# CSS Filters Module: Фильтры изображений и элементов - Полное руководство

## Введение

CSS Filters Module определяет эффекты, которые могут быть применены к элементам для изменения их визуального представления. Фильтры позволяют применять графические эффекты, такие как размытие, изменение яркости, контраста, насыщенности и другие, без использования JavaScript или внешних библиотек.

## CSS Filter Effects Module Level 1

Level 1 определяет основные фильтры для изображений и элементов.

### Основные фильтры

```css
.filter-examples {
  /* Отключение фильтров */
  filter: none;
  
  /* Размытие */
  filter: blur(5px);                /* Гауссово размытие */
  
  /* Яркость */
  filter: brightness(1.5);          /* 150% яркости */
  filter: brightness(50%);          /* 50% яркости */
  
  /* Контраст */
  filter: contrast(200%);           /* 200% контраста */
  filter: contrast(0.5);            /* 50% контраста */
  
  /* Насыщенность */
  filter: saturate(200%);           /* Удвоенная насыщенность */
  filter: saturate(0%);             /* Отключение насыщенности (оттенки серого) */
  
  /* Негатив */
  filter: invert(100%);             /* Полный инверсия цветов */
  filter: invert(50%);              /* Частичная инверсия */
  
  /* Непрозрачность */
  filter: opacity(50%);             /* 50% непрозрачности */
  filter: opacity(0.75);            /* 75% непрозрачности */
  
  /* Тень */
  filter: drop-shadow(10px 10px 5px rgba(0,0,0,0.5)); /* Тень как у box-shadow */
  
  /* Оттенок */
  filter: hue-rotate(90deg);        /* Поворот оттенка на 90 градусов */
  
  /* Градиент серого */
  filter: grayscale(100%);          /* Полное отображение в оттенках серого */
  filter: grayscale(50%);           /* Частичное отображение в оттенках серого */
  
  /* Сепия */
  filter: sepia(100%);              /* Полный эффект сепии */
  filter: sepia(30%);               /* Частичный эффект сепии */
  
  /* Размытие по Гауссу */
  filter: blur(8px);                /* Радиус размытия 8px */
}
```

### Комбинирование фильтров

```css
.combined-filters {
  /* Несколько фильтров в одной записи */
  filter: 
    contrast(175%) 
    saturate(200%) 
    brightness(110%);
  
  /* Более сложное сочетание */
  filter: 
    brightness(0.75) 
    contrast(150%) 
    saturate(120%) 
    sepia(20%);
  
  /* Сочетание с тенями */
  filter: 
    blur(1px) 
    drop-shadow(4px 4px 6px rgba(0,0,0,0.5));
}
```

## CSS Filter Effects Module Level 2

Level 2 добавляет дополнительные возможности и свойства.

### Функция url() для SVG фильтров

```css
.svg-filters {
  /* Использование SVG фильтров */
  filter: url(#svg-filter);
  /* Ссылка на SVG фильтр, определенный в SVG документе */
}
```

### Расширенные возможности

```css
.advanced-filters {
  /* Фильтры с переменными */
  filter: 
    brightness(var(--brightness, 1)) 
    contrast(var(--contrast, 1)) 
    saturate(var(--saturate, 1));
}
```

## Практические применения

### Эффекты для изображений

```css
.image-effects {
  transition: filter 0.3s ease;
}

.image-effects:hover {
  filter: 
    brightness(110%) 
    contrast(120%) 
    saturate(150%);
}

/* Эффект "стеклянной морфологии" */
.glass-effect {
  filter: blur(10px) saturate(120%) contrast(110%);
  backdrop-filter: blur(10px);     /* Отдельное свойство для фона */
}

/* Темный режим для изображений */
.dark-mode-image {
  filter: brightness(0.8) contrast(120%) saturate(110%) hue-rotate(5deg);
}

/* Эффект "антивозраст" для изображений */
.age-reduction {
  filter: 
    contrast(110%) 
    saturate(105%) 
    brightness(105%) 
    blur(0.5px);
}
```

### Интерактивные элементы

```css
.filter-buttons {
  padding: 10px 20px;
  border: none;
  background-color: #007bff;
  color: white;
  cursor: pointer;
  transition: filter 0.2s ease;
}

.filter-buttons:hover {
  filter: brightness(120%) saturate(150%);
}

.filter-buttons:active {
  filter: brightness(80%) contrast(120%);
}

/* Кнопка с эффектом размытия */
.blur-button {
  filter: blur(0.5px);
  transition: filter 0.3s ease;
}

.blur-button:hover {
  filter: blur(0px);
}
```

### Анимированные фильтры

```css
.animated-filter {
  filter: hue-rotate(0deg);
  animation: hueRotate 10s infinite linear;
}

@keyframes hueRotate {
  to {
    filter: hue-rotate(360deg);
  }
}

/* Пульсирующий эффект */
.pulsing-filter {
  animation: pulseFilter 2s infinite alternate;
}

@keyframes pulseFilter {
  0% {
    filter: brightness(1) contrast(100%);
  }
  100% {
    filter: brightness(1.2) contrast(120%) saturate(150%);
  }
}

/* Эффект "шума" */
.noise-filter {
  animation: noiseFilter 0.1s infinite;
}

@keyframes noiseFilter {
  0%, 100% { filter: contrast(100%); }
  50% { filter: contrast(110%) brightness(105%); }
}
```

### Текстовые эффекты

```css
.text-filters {
  font-size: 3rem;
  font-weight: bold;
  color: white;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  
  /* Текст с эффектом размытия */
  filter: blur(0.5px);
}

/* Неоновый текст */
.neon-text {
  color: #fff;
  text-shadow: 
    0 0 5px #fff, 
    0 0 10px #fff, 
    0 0 15px #007bff, 
    0 0 20px #007bff;
  filter: brightness(150%) contrast(200%);
}

/* Голографический текст */
.holographic-text {
  color: #00ffff;
  text-shadow: 
    0 0 5px #00ffff, 
    0 0 10px #00ffff, 
    0 0 20px #ff00de, 
    0 0 30px #ff00de;
  filter: hue-rotate(90deg) saturate(200%);
}
```

### Эффекты для видео и медиа

```css
.video-filters {
  /* Эффект "кино" для видео */
  filter: 
    contrast(110%) 
    saturate(120%) 
    brightness(95%) 
    sepia(10%);
}

.video-grayscale {
  filter: grayscale(100%);
  transition: filter 0.5s ease;
}

.video-grayscale:hover {
  filter: grayscale(0%);
}

/* Винтажный эффект */
.vintage-video {
  filter: 
    sepia(40%) 
    contrast(120%) 
    saturate(80%) 
    brightness(90%);
}
```

### Эффекты для фона

```css
.background-filters {
  background: url('background.jpg') center/cover;
  filter: 
    brightness(80%) 
    contrast(120%) 
    saturate(110%);
}

/* Фильтр для полупрозрачного фона */
.translucent-filter {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: 
    blur(10px) 
    brightness(90%) 
    contrast(110%);
}
```

## Современные возможности

### CSS Custom Properties с фильтрами

```css
.dynamic-filters {
  --brightness: 1;
  --contrast: 1;
  --saturate: 1;
  
  filter: 
    brightness(var(--brightness)) 
    contrast(var(--contrast)) 
    saturate(var(--saturate));
  
  transition: filter 0.3s ease;
}

.dynamic-filters:hover {
  --brightness: 1.2;
  --contrast: 1.1;
  --saturate: 1.3;
}
```

### Фильтры с анимацией на основе пользовательского взаимодействия

```css
.interactive-filters {
  filter: 
    brightness(1) 
    contrast(100%) 
    saturate(100%);
  transition: filter 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.interactive-filters[data-state="active"] {
  filter: 
    brightness(1.1) 
    contrast(110%) 
    saturate(120%);
}

.interactive-filters[data-state="warning"] {
  filter: 
    brightness(0.9) 
    contrast(130%) 
    saturate(150%) 
    hue-rotate(180deg);
}
```

## Производительность и оптимизация

```css
.optimized-filters {
  /* Активация GPU ускорения */
  transform: translateZ(0);
  
  /* Использование will-change для подготовки к анимации */
  will-change: filter;
  
  /* Оптимизация сложных фильтров */
  filter: 
    brightness(1.1) 
    contrast(110%);
  /* Избегать чрезмерно сложных комбинаций фильтров */
}

/* Упрощенные фильтры для мобильных устройств */
@media (max-width: 768px) {
  .optimized-filters {
    filter: brightness(1.05); /* Упрощенный фильтр */
  }
}
```

## Поддержка браузерами

- `filter` свойства: Хорошая поддержка в современных браузерах
- `backdrop-filter`: Хорошая поддержка, с префиксами для старых WebKit браузеров
- `url()` для SVG фильтров: Поддержка в современных браузерах
- Некоторые фильтры требуют префиксов в старых браузерах: `-webkit-filter`

## Связанные темы

- [[CSS Blend Modes]] - режимы наложения
- [[CSS Masking Module]] - маскировка элементов
- [[CSS Color Module]] - модуль цветов

## Заключение

CSS Filters Module предоставляет мощные инструменты для визуального изменения элементов без использования JavaScript. Понимание различных фильтров и их комбинаций позволяет создавать визуально привлекательные и интерактивные интерфейсы с высокой производительностью.
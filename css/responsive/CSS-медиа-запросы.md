---
aliases: ["Медиа-запросы CSS", "CSS Media Queries", "Адаптивный дизайн CSS"]
tags: 
  - css
  - responsive
  - media-queries
  - frontend
---

# CSS Медиа-запросы: Полное руководство

## Обзор

CSS медиа-запросы — это мощный инструмент для создания адаптивного дизайна, позволяющий применять различные стили в зависимости от характеристик устройства, таких как ширина экрана, ориентация, разрешение и другие параметры. Они являются ключевым компонентом современной веб-разработки и обеспечивают оптимальный пользовательский опыт на различных устройствах.

Медиа-запросы позволяют создавать гибкие и адаптивные интерфейсы, которые корректно отображаются как на мобильных устройствах, так и на настольных компьютерах, планшетах и других устройствах с различными характеристиками экрана.

## Основы медиа-запросов

### Синтаксис

Базовый синтаксис медиа-запросов включает ключевое слово `@media`, за которым следует медиа-тип и медиа-условия:

```css
@media media-type and (media-feature) {
  /* CSS правила */
}
```

### Медиа-типы

Медиа-типы определяют тип устройства, для которого применяются стили:

- `all` — для всех устройств (по умолчанию)
- `print` — для печатных устройств
- `screen` — для экранов
- `speech` — для речевых синтезаторов

Пример использования:

```css
@media print {
  body {
    font-size: 9pt;
    color: black;
  }
}

@media screen {
  body {
    font-size: 16px;
    background-color: white;
  }
}
```

## Медиа-характеристики

Медиа-характеристики позволяют задавать более точные условия для применения стилей:

### Характеристики размера

- `width` — ширина области просмотра
- `height` — высота области просмотра
- `min-width` — минимальная ширина
- `max-width` — максимальная ширина
- `min-height` — минимальная высота
- `max-height` — максимальная высота

```css
/* Для экранов шириной от 768px */
@media screen and (min-width: 768px) {
  .container {
    width: 750px;
  }
}

/* Для экранов шириной до 480px */
@media screen and (max-width: 480px) {
  .navigation {
    display: none;
  }
}
```

### Характеристики ориентации

- `orientation: portrait` — портретная ориентация
- `orientation: landscape` — альбомная ориентация

```css
@media screen and (orientation: landscape) {
  .mobile-menu {
    display: none;
  }
}
```

### Характеристики разрешения

- `resolution` — плотность пикселей устройства

```css
/* Для устройств с высоким разрешением */
@media screen and (-webkit-min-device-pixel-ratio: 2),
       screen and (min-resolution: 192dpi) {
  .icon {
    background-image: url('icon@2x.png');
    background-size: 24px 24px;
  }
}
```

## Логические операторы

Для создания более сложных условий используются логические операторы:

### and

Оператор `and` позволяет комбинировать несколько условий:

```css
@media screen and (min-width: 768px) and (max-width: 1024px) {
  .sidebar {
    width: 25%;
  }
}
```

### comma (or)

Запятая действует как логическое ИЛИ, позволяя применять стили при выполнении любого из условий:

```css
@media screen and (max-width: 480px), screen and (max-width: 768px) {
  .header {
    padding: 10px;
  }
}
```

### not

Оператор `not` инвертирует результат медиа-запроса:

```css
@media not screen and (color) {
  .image {
    filter: grayscale(100%);
  }
}
```

## Практические примеры использования

### Адаптивная сетка

```css
.grid {
  display: flex;
  flex-wrap: wrap;
}

/* Мобильная версия - одна колонка */
.grid-item {
  flex: 1 1 100%;
}

/* Планшет - две колонки */
@media screen and (min-width: 768px) {
  .grid-item {
    flex: 1 1 50%;
  }
}

/* Десктоп - три колонки */
@media screen and (min-width: 1024px) {
  .grid-item {
    flex: 1 1 calc(100% / 3);
  }
}
```

### Адаптивный типографика

```css
/* Базовый размер шрифта */
h1 {
  font-size: 1.5rem;
}

/* Увеличиваем шрифт для планшетов */
@media screen and (min-width: 768px) {
  h1 {
    font-size: 2rem;
  }
}

/* Увеличиваем шрифт для десктопа */
@media screen and (min-width: 1024px) {
  h1 {
    font-size: 2.5rem;
  }
}

/* Использование динамического изменения размера */
@media screen and (min-width: 320px) and (max-width: 1200px) {
  h1 {
    font-size: calc(1.5rem + ((1vw - 3.2px) * 0.6579));
  }
}
```

### Адаптивные изображения

```css
.image-container {
  width: 100%;
  height: auto;
}

.image-container img {
  width: 100%;
  height: auto;
  object-fit: cover;
}

/* Для больших экранов */
@media screen and (min-width: 1200px) {
  .image-container {
    max-width: 800px;
  }
}
```

## Наиболее распространенные точки останова (breakpoints)

Точки останова (breakpoints) — это ключевые значения размеров экрана, при которых меняется макет. Вот общепринятые точки останова:

| Тип устройства | Ширина экрана | Пример использования |
|----------------|---------------|----------------------|
| Мобильные устройства | до 480px | Базовая мобильная версия |
| Мобильные устройства (Landscape) | 481px - 768px | Альбомная ориентация мобильных устройств |
| Планшеты | 768px - 1024px | Планшетные устройства |
| Десктоп | 1025px - 1200px | Небольшие десктопные экраны |
| Большие десктопы | 1201px и выше | Большие экраны |

```css
/* Мобильная версия по умолчанию */
.navigation {
  display: flex;
  flex-direction: column;
}

/* Планшетная версия */
@media screen and (min-width: 768px) {
  .navigation {
    flex-direction: row;
  }
}

/* Десктопная версия */
@media screen and (min-width: 1024px) {
  .navigation {
    justify-content: space-between;
  }
}
```

## Современные подходы к медиа-запросам

### Mobile First подход

Mobile First — это стратегия, при которой сначала создается дизайн для мобильных устройств, а затем адаптируется для более крупных экранов с использованием `min-width`:

```css
/* Мобильная версия по умолчанию */
.card {
  padding: 10px;
  margin: 5px;
}

/* Адаптация для планшетов */
@media screen and (min-width: 768px) {
  .card {
    padding: 15px;
    margin: 10px;
  }
}

/* Адаптация для десктопа */
@media screen and (min-width: 1024px) {
  .card {
    padding: 20px;
    margin: 15px;
    max-width: 300px;
  }
}
```

### Desktop First подход

Desktop First — это противоположная стратегия, при которой сначала создается дизайн для больших экранов, а затем адаптируется для меньших с использованием `max-width`:

```css
/* Десктопная версия по умолчанию */
.sidebar {
  width: 300px;
  float: left;
}

.content {
  width: calc(100% - 300px);
  float: right;
}

/* Адаптация для планшетов */
@media screen and (max-width: 1024px) {
  .sidebar {
    width: 200px;
  }
  .content {
    width: calc(100% - 200px);
  }
}

/* Адаптация для мобильных устройств */
@media screen and (max-width: 768px) {
  .sidebar,
  .content {
    float: none;
    width: 100%;
  }
}
```

## Расширенные медиа-характеристики

### Характеристики цвета

- `color` — количество битов на цветовой компонент
- `color-index` — количество цветов в цветовой палитре
- `monochrome` — количество битов на пиксель в монохромном буфере

```css
/* Для устройств с ограниченной цветопередачей */
@media screen and (max-color: 8) {
  .gradient {
    background: linear-gradient(to bottom, #ffffff, #000000);
  }
}
```

### Характеристики функций

- `grid` — поддержка сетки (0 или 1)
- `hover` — поддержка наведения мыши
- `any-hover` — поддержка наведения на любом устройстве ввода
- `pointer` — тип указателя
- `any-pointer` — тип указателя на любом устройстве ввода

```css
/* Для устройств с поддержкой наведения */
@media (hover: hover) and (pointer: fine) {
  .button:hover {
    background-color: #0056b3;
  }
}

/* Для сенсорных устройств */
@media (hover: none) and (pointer: coarse) {
  .button {
    padding: 15px; /* Увеличенная область для нажатия */
  }
}
```

### Характеристики производительности

- `prefers-reduced-motion` — предпочтение уменьшения анимации
- `prefers-color-scheme` — предпочтение цветовой схемы
- `prefers-contrast` — предпочтение контраста

```css
/* Уменьшение анимации для пользователей, предпочитающих это */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Темная тема */
@media (prefers-color-scheme: dark) {
  body {
    background-color: #121212;
    color: #ffffff;
  }
}
```

## Лучшие практики

### 1. Используйте относительные единицы

Всегда используйте относительные единицы измерения (rem, em, %, vw, vh) вместо фиксированных значений (px) для создания гибких интерфейсов:

```css
.container {
  width: 90%;
  max-width: 75rem; /* 1200px при 16px базовом размере */
  margin: 0 auto;
}
```

### 2. Оптимизация для производительности

Избегайте чрезмерного использования медиа-запросов и старайтесь группировать их логически:

```css
/* Хорошо: сгруппировано по компонентам */
@media screen and (max-width: 768px) {
  .header {
    height: 60px;
  }
  
  .navigation {
    display: none;
  }
  
  .mobile-menu {
    display: block;
  }
}

/* Избегайте: разбросано по файлу */
@media screen and (max-width: 768px) {
  .header {
    height: 60px;
  }
}

@media screen and (max-width: 768px) {
  .navigation {
    display: none;
  }
}

@media screen and (max-width: 768px) {
  .mobile-menu {
    display: block;
  }
}
```

### 3. Тестирование на реальных устройствах

Хотя эмуляторы полезны, всегда тестируйте на реальных устройствах, особенно для критических пользовательских сценариев.

### 4. Использование современных методов

Рассмотрите использование современных методов, таких как Container Queries (контейнерные запросы), которые позволяют создавать компоненты, адаптирующиеся к размеру их родительского контейнера, а не к размеру окна:

```css
.card {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card-content {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

## Распространенные ошибки

### 1. Использование только фиксированных точек останова

Не создавайте точки останова только для конкретных устройств. Вместо этого определяйте их на основе содержимого:

```css
/* Плохо: точки останова для конкретных устройств */
@media screen and (max-width: 375px) { } /* iPhone X */
@media screen and (max-width: 360px) { } /* Samsung Galaxy S7 */

/* Хорошо: точки останова на основе содержимого */
@media screen and (max-width: 480px) { } /* Мобильные устройства */
@media screen and (max-width: 768px) { } /* Планшеты */
```

### 2. Неправильный порядок медиа-запросов

Располагайте медиа-запросы в правильном порядке, особенно при использовании Mobile First подхода:

```css
/* Правильно: от меньшего к большему */
.component {
  width: 100%;
}

@media screen and (min-width: 768px) {
  .component {
    width: 50%;
  }
}

@media screen and (min-width: 1024px) {
  .component {
    width: 33.33%;
  }
}
```

### 3. Избыточные медиа-запросы

Не дублируйте медиа-запросы, если можно объединить стили:

```css
/* Плохо: дублирование медиа-запросов */
@media screen and (max-width: 768px) {
  .element {
    padding: 10px;
  }
}

@media screen and (max-width: 768px) {
  .element {
    margin: 5px;
  }
}

/* Хорошо: объединенные стили */
@media screen and (max-width: 768px) {
  .element {
    padding: 10px;
    margin: 5px;
  }
}
```

## Инструменты и ресурсы

### Встроенные инструменты браузера

Современные браузеры предоставляют встроенные инструменты для тестирования адаптивного дизайна:

- Chrome DevTools Device Toolbar
- Firefox Responsive Design Mode
- Safari Responsive Design Mode

### Полезные ресурсы

- [MDN Web Docs: Using Media Queries](https://developer.mozilla.org/ru/docs/Web/CSS/Media_Queries/Using_media_queries)
- [CSS-Tricks: A Complete Guide to CSS Media Queries](https://css-tricks.com/a-complete-guide-to-css-media-queries/)
- [Can I Use: CSS Media Queries](https://caniuse.com/css-mediaqueries)

## Заключение

CSS медиа-запросы — это фундаментальный инструмент для создания адаптивных и отзывчивых веб-сайтов. Понимание их работы и правильное применение позволяет создавать интерфейсы, которые отлично выглядят и функционируют на всех устройствах.

Следуя лучшим практикам и избегая распространенных ошибок, вы можете создавать эффективные и поддерживаемые медиа-запросы, которые обеспечивают отличный пользовательский опыт на всех устройствах.

Для более глубокого понимания рекомендуется изучить связь медиа-запросов с другими аспектами адаптивного дизайна, такими как [[CSS Flexbox]], [[CSS Grid Layout]], [[Адаптивные изображения]] и [[Mobile First подход]].

Также важно понимать, как медиа-запросы интегрируются с современными фреймворками и библиотеками, такими как [[Bootstrap]] и [[Tailwind CSS]], которые предоставляют предопределенные точки останова и утилиты для адаптивного дизайна.

При разработке адаптивных интерфейсов также следует учитывать [[Доступность веб-контента]] и [[Оптимизация производительности]], чтобы обеспечить наилучший пользовательский опыт на всех устройствах.
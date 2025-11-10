---
aliases: ["Современные CSS функции", "CSS3", "CSS Features"]
tags: [css, modern-css, web-development, frontend]
---

# Современные CSS функции

## Введение

Современный CSS предоставляет мощные возможности для создания адаптивных, гибких и выразительных веб-интерфейсов. В этом руководстве рассмотрим ключевые современные функции CSS, которые значительно улучшают разработку пользовательского интерфейса.

## CSS Custom Properties (Переменные)

CSS Custom Properties (переменные) позволяют хранить и повторно использовать значения в CSS:

```css
:root {
  --primary-color: #3498db;
  --font-size: 16px;
}

.button {
  background-color: var(--primary-color);
  font-size: var(--font-size);
}
```

### Расширенное использование

Можно использовать сallback-значения и вложенность:

```css
:root {
  --base-color: #ff6b6b;
  --primary-color: color-mix(in srgb, var(--base-color) 80%, white);
}

.card {
  --card-padding: clamp(1rem, 2.5vw, 2rem);
  padding: var(--card-padding);
}
```

См. также: [[css/variables/advanced]] и [[css/variables/basics]]

## CSS Grid Level 2 и 3

CSS Grid Level 2 и 3 добавили поддержку фреймворков макета, улучшенное выравнивание и поддержку субгридов.

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}
```

## Subgrid

Subgrid позволяет дочерним элементам наследовать сетку родительского элемента:

```css
.parent {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto;
}

.child {
  display: subgrid;
  grid-column: span 3;
}
```

См. также: [[css/grid/subgrid]]

## Container Queries

Container Queries позволяют применять стили на основе размеров родительского контейнера:

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card-content {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

См. также: [[css/container-queries/basics]]

## Cascade Layers (@layer)

Cascade Layers позволяют контролировать приоритетность стилей:

```css
@layer defaults, theme, layout, components;

@layer defaults {
  button {
    padding: 0.5rem;
  }
}

@layer theme {
  button {
    background-color: var(--primary-color);
  }
}
```

## CSS Anchor Positioning

CSS Anchor Positioning позволяет позиционировать элементы относительно других элементов:

```css
.tooltip {
  position: absolute;
  anchor-name: --tooltip-anchor;
}

.trigger {
  anchor-name: --tooltip-anchor;
}

.tooltip {
  position-anchor: --tooltip-anchor;
  position-try: flip-block;
}
```

## Nesting (будущее)

CSS Nesting позволяет вкладывать селекторы:

```css
.card {
  background: white;
  
  &__header {
    font-weight: bold;
  }
  
  &__content {
    padding: 1rem;
    
    &--featured {
      border: 2px solid gold;
    }
  }
}
```

## Custom Highlight API

Custom Highlight API позволяет программно выделять текст:

```javascript
const customHighlight = new Highlight(
  new Range(), // определенный диапазон
);

CSS.highlights.set('custom-highlight', customHighlight);
```

```css
::highlight(custom-highlight) {
  background-color: yellow;
  font-weight: bold;
}
```

## View Transitions API

View Transitions API обеспечивает плавные переходы между состояниями:

```css
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to { opacity: 0; }
}

::view-transition-old(root) {
  animation: 90ms cubic-bezier(0.4, 0, 1, 1) both fadeOut;
}

::view-transition-new(root) {
  animation: 210ms cubic-bezier(0, 0, 0.2, 1) both fadeIn;
}
```

## Scroll-Driven Animations

Scroll-Driven Animations связывают анимации с прокруткой:

```css
.scrolling-element {
  animation-timeline: scroll();
}

@keyframes slide-in {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}
```

## Новые псевдоклассы и псевдоэлементы

Новые псевдоклассы:

- `:has()` - стилизация элемента на основе его потомков
- `:is()` и `:where()` - группировка селекторов
- `:not()` - расширенная функциональность
- `:focus-visible` - стили для клавиатурной навигации

```css
/* Стилизация ссылок, содержащих изображения */
a:has(img) {
  border: none;
  text-decoration: none;
}

/* Стилизация заголовков с уменьшенной специфичностью */
h1:where(.content > *) {
  margin-top: 0;
}
```

## CSS функции: min(), max(), clamp()

Функции `min()`, `max()` и `clamp()` позволяют создавать адаптивные размеры:

```css
.responsive-text {
  font-size: clamp(1rem, 2.5vw, 2rem);
}

.container {
  width: min(100%, 1200px);
  padding: max(1rem, 2vw);
}
```

## Цветовые функции: color(), lab(), lch()

Современные цветовые пространства:

```css
.element {
  /* CIELAB цветовое пространство */
  color: lab(62.25% 23.3 10.7);
  
  /* CIELCh цветовое пространство */
  color: lch(62.25% 25.6 23.5);
  
  /* Указание пространства и преобразование */
  color: color(display-p3 0.4 0.6 0.8);
}
```

См. также: [[css/colors/advanced]]

## Логические свойства

Логические свойства обеспечивают адаптацию к направлению письма:

```css
.component {
  margin-inline-start: 1rem;
  margin-inline-end: 1rem;
  padding-block-start: 0.5rem;
  padding-block-end: 0.5rem;
}
```

## Новые единицы измерения

- `ex` - высота строчной буквы x
- `cap` - высота заглавной буквы
- `ic` - ширина идеограммы (например, иероглифа)
- `lh` - высота строки
- `rlh` - высота строки корневого элемента

```css
.text {
  line-height: 1.5lh; /* 1.5 высоты строки */
  margin-block-start: 2cap; /* 2 высоты заглавной буквы */
}
```

## Улучшения Media Queries

Улучшенные медиазапросы:

```css
/* Условия с использованием and/or */
@media (width > 600px) and (width < 1000px) {
  .container {
    display: grid;
  }
}

/* Интервалы */
@media (300px <= width <= 600px) {
  .mobile-layout {
    flex-direction: column;
  }
}
```

## Заключение

Современные CSS функции значительно расширяют возможности веб-разработки, позволяя создавать более гибкие, адаптивные и выразительные интерфейсы. Важно использовать их осознанно, учитывая поддержку браузерами и доступность.

См. также: [[css/best-practices]], [[css/responsive-design]], [[css/animation-principles]]
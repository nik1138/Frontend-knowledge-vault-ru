---
aliases: [CSS Grid, Grid Layout, Сеточные системы, Сетка CSS]
tags: [frontend, architecture, css, grid, layout, responsive]
---

# Grid-системы

Grid-системы - это фундаментальный инструмент современной веб-архитектуры, позволяющий создавать сложные и адаптивные макеты с минимальными усилиями. CSS Grid Layout (CSS Grid) стал стандартом для создания двумерных макетов, превосходя традиционные методы на основе float и inline-block.

## Основные понятия Grid

### Grid Container и Grid Items

```css
.grid-container {
  display: grid;
  /* Свойства контейнера */
}

.grid-item {
  /* Свойства элементов сетки */
}
```

### Основные свойства Grid Container

```css
.grid-container {
  display: grid;
  
  /* Определение строк и столбцов */
  grid-template-columns: 1fr 2fr 1fr; /* 3 столбца */
  grid-template-rows: auto 100px auto; /* 3 строки */
  
  /* Отступы между ячейками */
  gap: 1rem; /* или отдельно: row-gap и column-gap */
  
  /* Выравнивание элементов */
  justify-items: center; /* по горизонтали */
  align-items: center;   /* по вертикали */
}
```

### Основные свойства Grid Items

```css
.grid-item {
  /* Позиционирование в сетке */
  grid-column: 1 / 3;    /* занимает с 1 по 3 столбец */
  grid-row: 1 / 2;       /* занимает с 1 по 2 строку */
  
  /* Альтернативный синтаксис */
  grid-column-start: 1;
  grid-column-end: 3;
  
  /* Выравнивание конкретного элемента */
  justify-self: start;
  align-self: end;
}
```

## Практические примеры Grid-систем

### 1. Базовая сетка с адаптивными колонками

```css
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
  padding: 1rem;
}

/* Для мобильных устройств */
@media (max-width: 768px) {
  .responsive-grid {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
}
```

### 2. Макет страницы с Grid

```css
.page-layout {
  display: grid;
  grid-template-areas: 
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  grid-template-rows: auto 1fr auto;
  grid-template-columns: 200px 1fr 200px;
  min-height: 100vh;
  gap: 1rem;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }

/* Адаптация для мобильных */
@media (max-width: 768px) {
  .page-layout {
    grid-template-areas: 
      "header"
      "main"
      "sidebar"
      "aside"
      "footer";
    grid-template-columns: 1fr;
    grid-template-rows: auto 1fr auto auto auto;
  }
}
```

### 3. Карточки товаров с Grid

```css
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 1.5rem;
  padding: 1rem;
}

.product-card {
  display: grid;
  grid-template-rows: auto 1fr auto;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.2s, box-shadow 0.2s;
}

.product-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.product-info {
  padding: 1rem;
  display: grid;
  grid-template-rows: 1fr auto;
}

.product-title {
  margin: 0 0 0.5rem 0;
}

.product-price {
  font-weight: bold;
  color: #e74c3c;
  align-self: end;
}

.product-actions {
  padding: 1rem;
  border-top: 1px solid #eee;
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 0.5rem;
}
```

## Расширенные возможности Grid

### 1. Именованные линии

```css
.named-grid {
  display: grid;
  grid-template-columns: [start] 1fr [middle] 2fr [end];
  grid-template-rows: [top] auto [content] 1fr [bottom];
}

.item {
  grid-column: start / middle; /* от линии start до линии middle */
  grid-row: top / content;
}
```

### 2. Grid с использованием функции repeat()

```css
.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr); /* 7 равных столбцов для дней недели */
  grid-template-rows: auto repeat(5, 1fr); /* заголовок + 5 строк дней */
}

/* Повтор с разными значениями */
.complex-grid {
  grid-template-columns: 
    [sidebar] 200px 
    [content-start] repeat(12, [col-start] 1fr [col-end]) 
    [content-end] 200px [aside];
}
```

### 3. Grid Areas для сложных макетов

```css
.dashboard-layout {
  display: grid;
  grid-template-areas: 
    "header header header"
    "nav main sidebar"
    "nav footer footer";
  grid-template-columns: 250px 1fr 200px;
  grid-template-rows: 60px 1fr 80px;
  height: 100vh;
  gap: 1rem;
}

.dashboard-header { grid-area: header; }
.dashboard-nav { grid-area: nav; }
.dashboard-main { grid-area: main; }
.dashboard-sidebar { grid-area: sidebar; }
.dashboard-footer { grid-area: footer; }
```

## Адаптивные Grid-системы

### 1. Использование minmax() и fr единиц

```css
.adaptive-grid {
  display: grid;
  /* Минимум 250px, максимум 1fr (равные колонки) */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

/* Адаптация под разные размеры экрана */
@media (max-width: 480px) {
  .adaptive-grid {
    grid-template-columns: 1fr; /* одна колонка на мобильных */
  }
}

@media (min-width: 1200px) {
  .adaptive-grid {
    grid-template-columns: repeat(4, 1fr); /* 4 колонки на больших экранах */
  }
}
```

### 2. Grid с динамическим количеством колонок

```css
.dynamic-grid {
  display: grid;
  /* Автоматическое определение количества колонок */
  grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
  gap: 1rem;
}

/* Использование clamp() для плавного масштабирования */
.responsive-clamp-grid {
  display: grid;
  grid-template-columns: 
    repeat(auto-fit, 
           minmax(clamp(250px, 25vw, 400px), 1fr));
  gap: clamp(1rem, 2vw, 2rem);
}
```

## Современные подходы (2025)

### 1. Subgrid (экспериментальная возможность)

```css
/* Позволяет вложенным элементам участвовать в родительской сетке */
.parent-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto;
}

.child-grid {
  display: subgrid;
  grid-column: span 3; /* занимает все 3 колонки родителя */
}
```

### 2. Grid с Container Queries

```css
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
}

@container (min-width: 600px) {
  .card-grid {
    grid-template-columns: 1fr 1fr 1fr;
  }
}
```

## Практические рекомендации

### 1. Использование Grid в реальных проектах

```css
/* Базовая сетка для контента */
.content-grid {
  display: grid;
  grid-template-columns: 
    [full-start] 1fr 
    [main-start] minmax(300px, 1200px) [main-end] 
    1fr [full-end];
  gap: 1rem;
}

.content-area {
  grid-column: main-start / main-end;
}

.full-width-element {
  grid-column: full-start / full-end;
}
```

### 2. Сочетание Grid и Flexbox

```css
.grid-with-flex {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1.5rem;
}

.grid-item {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.item-header {
  flex-shrink: 0;
}

.item-content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.item-footer {
  flex-shrink: 0;
}
```

### 3. Grid для форм

```css
.form-grid {
  display: grid;
  grid-template-columns: 
    [labels] 150px 
    [inputs] 1fr 
    [actions] 200px;
  grid-template-rows: auto;
  gap: 1rem 2rem;
  align-items: start;
}

.form-label {
  grid-column: labels;
  justify-self: end;
}

.form-input {
  grid-column: inputs;
}

.form-actions {
  grid-column: actions;
  grid-row: 1 / -1; /* Занимает все строки */
}
```

## Российские особенности (2025)

### 1. Поддержка браузеров

CSS Grid поддерживается всеми современными браузерами, включая:

- Chrome 57+
- Firefox 52+
- Safari 10.1+
- Edge 16+

Для поддержки старых браузеров (IE11 и ниже) можно использовать polyfill:

```html
<script src="https://cdn.jsdelivr.net/npm/css-grid-polyfill@1.0.0/dist/css-grid-polyfill.min.js"></script>
```

### 2. Производительность

Grid-системы в 2025 году оптимизированы для высокой производительности:

- Использование hardware acceleration
- Эффективное перераспределение элементов
- Минимизация relayout операций

## Заключение

Grid-системы стали неотъемлемой частью современной веб-архитектуры, позволяя создавать сложные адаптивные макеты с минимальными усилиями. Их сочетание с [[Responsive-дизайн]], [[Mobile-first]] подходом и современными возможностями CSS делает их мощным инструментом для разработки пользовательских интерфейсов.

См. также:
- [[Responsive-дизайн]]
- [[Mobile-first]]
- [[Adaptive-дизайн]]
- [[Тестирование]]
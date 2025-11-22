---
aliases: ["CSS Subgrid", "Subgrid", "Подсетка CSS", "CSS Nested Grid"]
tags: [css, grid, layout, subgrid, modern-css, web-development, frontend]
---

# CSS Subgrid

## Введение

CSS Subgrid — это мощная функция CSS Grid Layout, которая позволяет дочерним элементам наследовать линии сетки от родительского элемента. Это особенно полезно при создании сложных вложенных макетов, где необходимо согласовать структуру сетки между родительскими и дочерними элементами.

## Основные концепции

### Что такое Subgrid

Subgrid позволяет дочерним элементам внутри CSS Grid контейнера наследовать линии сетки от родительского элемента, а не создавать свою собственную независимую сетку. Это позволяет создавать более согласованные и сложные макеты.

### Когда использовать Subgrid

Subgrid особенно полезен в следующих сценариях:
- Создание сложных форм с выравниванием по сетке
- Макеты с несколькими уровнями вложенности
- Согласование структуры между родительскими и дочерними компонентами
- Создание сложных таблиц с вложенными элементами

## Основы Subgrid

### Синтаксис

```css
.parent-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 20px;
}

.child-subgrid {
  /* Использование subgrid для наследования сетки родителя */
  display: subgrid;
  grid-column: span 2; /* Занимает 2 колонки родительской сетки */
  grid-row: span 2;    /* Занимает 2 строки родительской сетки */
}
```

### grid-template-columns и grid-template-rows с subgrid

```css
/* Пример базового использования */
.main-container {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}

.form-section {
  display: subgrid;
  grid-column: 1 / -1; /* Занимает все колонки */
  grid-template-rows: subgrid; /* Наследует строки родителя */
}

/* Важно: при использовании subgrid, 
   grid-template-columns/rows должны быть равны subgrid */
.nested-subgrid {
  display: subgrid;
  grid-column: span 2;
  grid-template-columns: subgrid; /* Наследует колонки родителя */
  grid-template-rows: subgrid;    /* Наследует строки родителя */
}
```

## Практические примеры

### 1. Форма с выравниванием по сетке

```css
.form-container {
  display: grid;
  grid-template-columns: 
    [start] 1fr 
    [label-start] 200px 
    [input-start] 1fr 
    [button-start] 150px 
    [end];
  grid-template-rows: auto;
  gap: 1rem;
  padding: 2rem;
}

.form-section {
  display: subgrid;
  grid-column: start / end; /* Занимает всю ширину */
  grid-template-columns: subgrid; /* Наследует колонки родителя */
  gap: inherit;
  margin-bottom: 1.5rem;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.form-label {
  grid-column: label-start / input-start;
  align-self: center;
  font-weight: bold;
}

.form-input {
  grid-column: input-start / button-start;
}

.form-button {
  grid-column: button-start / end;
  justify-self: end;
}
```

### 2. Карточки с согласованным макетом

```css
.card-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
  padding: 1rem;
}

.card {
  display: subgrid;
  grid-column: span 1;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
}

.card-header {
  grid-row: 1;
  grid-column: 1 / -1;
  background-color: #f5f5f5;
  padding: 1rem;
}

.card-content {
  grid-row: 2;
  grid-column: 1 / -1;
  padding: 1rem;
}

.card-footer {
  grid-row: 3;
  grid-column: 1 / -1;
  padding: 1rem;
  background-color: #fafafa;
}
```

### 3. Сложный макет с вложенными секциями

```css
.main-layout {
  display: grid;
  grid-template-columns: 
    [sidebar-start] 250px 
    [content-start] 1fr 
    [aside-start] 200px 
    [end];
  grid-template-rows: 
    [header-start] auto 
    [main-start] 1fr 
    [footer-start] auto 
    [footer-end];
  height: 100vh;
}

.header {
  display: subgrid;
  grid-column: sidebar-start / end;
  grid-row: header-start / main-start;
  grid-template-columns: subgrid;
  border-bottom: 1px solid #ccc;
}

.main-content {
  display: subgrid;
  grid-column: content-start / aside-start;
  grid-row: main-start / footer-start;
  grid-template-columns: subgrid;
  padding: 1rem;
}

.sidebar {
  display: subgrid;
  grid-column: sidebar-start / content-start;
  grid-row: main-start / footer-start;
  grid-template-columns: subgrid;
  background-color: #f9f9f9;
}

.aside {
  display: subgrid;
  grid-column: aside-start / end;
  grid-row: main-start / footer-start;
  grid-template-columns: subgrid;
  background-color: #f0f0f0;
}

.footer {
  display: subgrid;
  grid-column: sidebar-start / end;
  grid-row: footer-start / footer-end;
  grid-template-columns: subgrid;
  border-top: 1px solid #ccc;
}
```

## Продвинутые техники

### Вложенные subgrid-ы

```css
.level-1-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(3, 200px);
  gap: 1rem;
}

.level-1-item {
  display: subgrid;
  grid-column: span 2;
  grid-row: span 2;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}

.level-2-container {
  display: subgrid;
  grid-column: 1 / -1;
  grid-row: 1 / -1;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(2, 1fr);
  gap: 0.5rem;
}

.level-2-item {
  /* Наследует сетку от level-1-container */
}
```

### Subgrid с именованными линиями

```css
.named-grid {
  display: grid;
  grid-template-columns: 
    [full-start] 1fr 
    [main-start] 2fr 
    [sidebar-start] 1fr 
    [full-end];
  grid-template-rows: 
    [top] auto 
    [middle] 1fr 
    [bottom] auto;
  gap: 1rem;
}

.named-subgrid {
  display: subgrid;
  grid-column: main-start / sidebar-start;
  grid-row: middle;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}
```

### Subgrid с функциями grid

```css
.flexible-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  grid-template-rows: repeat(3, minmax(150px, auto));
  gap: 1rem;
}

.flexible-subgrid {
  display: subgrid;
  grid-column: span 2;
  grid-row: span 1;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}
```

## Совместимость и резервные варианты

### Проверка поддержки

```css
@supports (display: subgrid) {
  .subgrid-supported {
    display: subgrid;
    grid-column: span 2;
    grid-template-columns: subgrid;
  }
}

@supports not (display: subgrid) {
  .subgrid-fallback {
    display: grid;
    grid-template-columns: 1fr 1fr; /* Альтернативная структура */
    grid-column: span 2;
  }
}
```

### Постепенное улучшение

```css
/* Базовая структура */
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

/* Современная структура с subgrid */
@supports (display: subgrid) {
  .card-section {
    display: subgrid;
    grid-column: span 2;
    grid-template-columns: subgrid;
  }
}
```

## Производительность и оптимизация

### Оптимизация subgrid-макетов

```css
/* Оптимизированный subgrid-контейнер */
.optimized-subgrid-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(3, 200px);
  contain: layout style;
}

.optimized-subgrid {
  display: subgrid;
  grid-column: span 2;
  grid-row: span 2;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
  contain: layout style;
}
```

### Избегайте чрезмерной вложенности

```css
/* Хорошо: разумное количество уровней */
.good-nesting {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

.good-subgrid {
  display: subgrid;
  grid-column: span 2;
  grid-template-columns: subgrid;
}

/* Избегайте: слишком глубокая вложенность */
.avoid-deep-nesting {
  display: grid;
  /* Много уровней вложенности subgrid может повлиять на производительность */
}
```

## Примеры из промышленной разработки

### Дашборд с согласованным макетом

```css
.dashboard-grid {
  display: grid;
  grid-template-columns: 
    [sidebar-start] 250px 
    [main-start] 1fr 
    [widgets-start] 300px 
    [end];
  grid-template-rows: 
    [header-start] 60px 
    [content-start] 1fr 
    [footer-start] 40px;
  gap: 1rem;
  height: 100vh;
}

.dashboard-header {
  display: subgrid;
  grid-column: sidebar-start / end;
  grid-row: header-start / content-start;
  grid-template-columns: subgrid;
  background-color: #333;
  color: white;
  align-items: center;
}

.dashboard-sidebar {
  display: subgrid;
  grid-column: sidebar-start / main-start;
  grid-row: content-start / footer-start;
  grid-template-columns: subgrid;
  background-color: #f8f9fa;
}

.dashboard-main {
  display: subgrid;
  grid-column: main-start / widgets-start;
  grid-row: content-start / footer-start;
  grid-template-columns: subgrid;
  padding: 1rem;
}

.dashboard-widgets {
  display: subgrid;
  grid-column: widgets-start / end;
  grid-row: content-start / footer-start;
  grid-template-columns: subgrid;
  background-color: #f0f0f0;
}

.dashboard-footer {
  display: subgrid;
  grid-column: sidebar-start / end;
  grid-row: footer-start / span 1;
  grid-template-columns: subgrid;
  background-color: #e9ecef;
}
```

### Сложная форма с несколькими секциями

```css
.complex-form-grid {
  display: grid;
  grid-template-columns: 
    [label-start] 150px 
    [input-start] 1fr 
    [validation-start] 200px 
    [end];
  grid-template-rows: auto;
  gap: 1rem;
  padding: 2rem;
}

.form-section {
  display: subgrid;
  grid-column: label-start / end;
  grid-template-columns: subgrid;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 1.5rem;
}

.section-title {
  grid-column: label-start / end;
  font-weight: bold;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #eee;
}

.form-label {
  grid-column: label-start / input-start;
  align-self: center;
  font-weight: 500;
}

.form-input {
  grid-column: input-start / validation-start;
}

.validation-message {
  grid-column: validation-start / end;
  align-self: center;
  color: #e74c3c;
  font-size: 0.9em;
}

.form-actions {
  grid-column: input-start / end;
  justify-self: end;
  margin-top: 1rem;
}
```

## Лучшие практики

### 1. Использование именованных линий

```css
/* Хорошо: использование именованных линий для лучшей читаемости */
.named-lines-grid {
  display: grid;
  grid-template-columns: 
    [content-start] 1fr 
    [sidebar-start] 300px 
    [content-end];
}

.named-lines-subgrid {
  display: subgrid;
  grid-column: content-start / sidebar-start;
  grid-template-columns: subgrid;
}
```

### 2. Учет доступности

```css
/* Убедитесь, что структура DOM логична независимо от визуального порядка */
.subgrid-accessible {
  display: subgrid;
  /* Порядок визуального отображения может отличаться от DOM */
  /* Убедитесь, что логика навигации клавиатуры остается логичной */
}
```

### 3. Тестирование на разных размерах экрана

```css
/* Адаптивные subgrid-макеты */
@media (max-width: 768px) {
  .responsive-grid {
    grid-template-columns: 1fr;
  }

  .responsive-subgrid {
    grid-column: span 1;
  }
}
```

## Ограничения и подводные камни

### 1. Ограничения в браузерах

Subgrid не поддерживается в Internet Explorer и некоторых более старых браузерах. Всегда предоставляйте резервные варианты.

### 2. Сложность отладки

Вложенные subgrid-структуры могут быть сложны для отладки. Используйте инструменты разработчика браузера для визуализации сеток.

### 3. Ограничения в вычислениях

Subgrid не может изменять размеры линий сетки родителя. Он только наследует существующую структуру.

## Заключение

CSS Subgrid — это мощный инструмент для создания сложных и согласованных макетов. Он особенно полезен при создании форм, дашбордов и других структур, где необходимо согласовать структуру между родительскими и дочерними элементами.

При правильном использовании subgrid позволяет создавать более гибкие и согласованные макеты, упрощая процесс создания сложных визуальных структур. Важно учитывать совместимость с браузерами и всегда предоставлять резервные варианты для более старых браузеров.

## См. также

- [[CSS Grid Layout]]
- [[Grid Patterns]]
- [[Modern CSS Layout]]
- [[CSS Layout Techniques]]
- [[CSS Flexbox]]
- [[CSS Container Queries]]
- [[CSS Custom Properties]]
- [[CSS Containment]]
- [[Responsive Grid Design]]
- [[Advanced CSS Grid]]
- [[CSS Layout Modules]]
- [[CSS Architecture Patterns]]
- [[Component-Based Architecture]]
- [[CSS Methodologies]]
- [[CSS Architecture]]

## Дополнительные ресурсы

- [MDN Web Docs: CSS Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout/Subgrid)
- [CSS Working Group: CSS Grid Layout Module Level 2](https://www.w3.org/TR/css-grid-2/)
- [Web.dev: Learn Grid](https://web.dev/learn/css/grid/)
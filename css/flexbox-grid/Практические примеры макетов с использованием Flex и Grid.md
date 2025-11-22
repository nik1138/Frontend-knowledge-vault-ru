---
aliases: ["Flex и Grid примеры", "Практические макеты", "Примеры CSS раскладок"]
tags: [css, flexbox, grid, layout, examples, practical]
---

# Практические примеры макетов с использованием Flex и Grid

## Введение

На практике Flexbox и Grid часто используются совместно для создания сложных и адаптивных интерфейсов. В этом руководстве мы рассмотрим реальные примеры макетов, где обе системы раскладки применяются эффективно и гармонично.

## Базовые комбинации Flex и Grid

### Простой макет страницы

```css
.page-layout {
  display: grid;
  grid-template-areas: 
    "header"
    "main"
    "footer";
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.page-layout .header {
  grid-area: header;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  background: #343a40;
  color: white;
}

.page-layout .main {
  grid-area: main;
  padding: 2rem;
}

.page-layout .footer {
  grid-area: footer;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  background: #e9ecef;
}
```

### Адаптивная сетка карточек

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
  padding: 1rem;
}

.card {
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
}

.card .card-image {
  width: 100%;
  height: 200px;
  background: #e9ecef;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card .card-content {
  flex-grow: 1;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
}

.card .card-footer {
  padding: 1rem 1.5rem;
  background: #f8f9fa;
  display: flex;
  justify-content: flex-end;
}
```

## Сложные макеты

### Панель управления (dashboard)

```css
.dashboard {
  display: grid;
  grid-template-areas: 
    "sidebar header"
    "sidebar main";
  grid-template-columns: 250px 1fr;
  grid-template-rows: 60px 1fr;
  height: 100vh;
}

.dashboard .sidebar {
  grid-area: sidebar;
  background: #2c3e50;
  color: white;
  display: flex;
  flex-direction: column;
}

.dashboard .sidebar .logo {
  padding: 1rem;
  border-bottom: 1px solid #34495e;
}

.dashboard .sidebar .nav {
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}

.dashboard .sidebar .nav-item {
  padding: 0.75rem 1rem;
  cursor: pointer;
  transition: background 0.2s ease;
}

.dashboard .sidebar .nav-item:hover {
  background: #34495e;
}

.dashboard .header {
  grid-area: header;
  background: white;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 1.5rem;
  border-bottom: 1px solid #e9ecef;
}

.dashboard .main {
  grid-area: main;
  display: grid;
  grid-template-columns: 2fr 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas: 
    "stats stats"
    "content sidebar"
    "footer footer";
  gap: 1.5rem;
  padding: 1.5rem;
  overflow: auto;
}

.dashboard .stats {
  grid-area: stats;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.dashboard .content {
  grid-area: content;
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.dashboard .content .toolbar {
  padding: 1rem;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.dashboard .content .content-area {
  flex-grow: 1;
  padding: 1.5rem;
  overflow: auto;
}

.dashboard .sidebar-widgets {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.dashboard .footer {
  grid-area: footer;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 8px;
}
```

### Страница продукта

```css
.product-page {
  display: grid;
  grid-template-areas: 
    "gallery info"
    "gallery info"
    "description description"
    "reviews reviews";
  grid-template-columns: 1fr 1fr;
  grid-template-rows: auto auto auto 1fr;
  gap: 2rem;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

.product-page .gallery {
  grid-area: gallery;
  display: grid;
  grid-template-columns: 80px 1fr;
  grid-template-rows: repeat(4, 80px) 1fr;
  gap: 0.5rem;
}

.product-page .gallery .main-image {
  grid-column: 2;
  grid-row: 1 / span 4;
  background: #f8f9fa;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.product-page .gallery .thumbs {
  grid-column: 1;
  grid-row: 1 / span 4;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.product-page .gallery .thumb {
  height: 75px;
  background: #e9ecef;
  border-radius: 4px;
  cursor: pointer;
}

.product-page .info {
  grid-area: info;
  display: flex;
  flex-direction: column;
}

.product-page .info .title {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.product-page .info .price {
  font-size: 1.25rem;
  color: #28a745;
  font-weight: bold;
  margin-bottom: 1rem;
}

.product-page .info .rating {
  display: flex;
  align-items: center;
  margin-bottom: 1rem;
}

.product-page .info .actions {
  display: flex;
  gap: 1rem;
  margin-top: auto;
}

.product-page .info .actions .btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

.product-page .info .actions .primary-btn {
  background: #007bff;
  color: white;
  flex-grow: 1;
}

.product-page .info .actions .secondary-btn {
  background: #6c757d;
  color: white;
}

.product-page .description {
  grid-area: description;
  padding: 1.5rem;
  background: #f8f9fa;
  border-radius: 8px;
}

.product-page .reviews {
  grid-area: reviews;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
```

## Адаптивные макеты

### Адаптивный макет без медиа-запросов

```css
.adaptive-layout {
  display: grid;
  grid-template-columns: 1fr;
  grid-template-areas: 
    "header"
    "sidebar"
    "main"
    "footer";
}

@media (min-width: 768px) {
  .adaptive-layout {
    grid-template-columns: 200px 1fr;
    grid-template-areas: 
      "header header"
      "sidebar main"
      "footer footer";
  }
}

@media (min-width: 1024px) {
  .adaptive-layout {
    grid-template-columns: 250px 1fr 200px;
    grid-template-areas: 
      "header header header"
      "sidebar main aside"
      "footer footer footer";
  }
}

.adaptive-layout .header {
  grid-area: header;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  background: #343a40;
  color: white;
}

.adaptive-layout .sidebar {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
  padding: 1rem;
  background: #e9ecef;
}

.adaptive-layout .main {
  grid-area: main;
  padding: 1rem;
}

.adaptive-layout .aside {
  grid-area: aside;
  padding: 1rem;
  background: #f8f9fa;
}

.adaptive-layout .footer {
  grid-area: footer;
  padding: 1rem;
  background: #6c757d;
  color: white;
  display: flex;
  justify-content: center;
}
```

### Карточки с адаптивным поведением

```css
.adaptive-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
  padding: 1rem;
}

.adaptive-card {
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.adaptive-card .card-header {
  padding: 1.5rem;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.adaptive-card .card-body {
  flex-grow: 1;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
}

.adaptive-card .card-footer {
  padding: 1.5rem;
  background: #f8f9fa;
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
}

/* Адаптация для маленьких экранов */
@media (max-width: 480px) {
  .adaptive-card {
    flex-direction: row;
  }
  
  .adaptive-card .card-body {
    flex-grow: 1;
  }
  
  .adaptive-card .card-footer {
    flex-direction: column;
    justify-content: flex-start;
    gap: 0.5rem;
  }
}
```

## Специфические паттерны

### "Holy Grail" макет

```css
.holy-grail {
  display: grid;
  grid-template-areas: 
    "header header header"
    "nav main aside"
    "footer footer footer";
  grid-template-columns: 200px 1fr 150px;
  grid-template-rows: 60px 1fr 40px;
  min-height: 100vh;
}

.holy-grail .header {
  grid-area: header;
  background: #343a40;
  color: white;
  display: flex;
  align-items: center;
  padding: 0 1.5rem;
}

.holy-grail .nav {
  grid-area: nav;
  background: #e9ecef;
  padding: 1rem 0;
}

.holy-grail .nav ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.holy-grail .nav li {
  padding: 0.5rem 1rem;
}

.holy-grail .main {
  grid-area: main;
  padding: 2rem;
  display: flex;
  flex-direction: column;
}

.holy-grail .main .content {
  flex-grow: 1;
}

.holy-grail .aside {
  grid-area: aside;
  background: #f8f9fa;
  padding: 1rem;
}

.holy-grail .footer {
  grid-area: footer;
  background: #6c757d;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

### Чат-интерфейс

```css
.chat-interface {
  display: grid;
  grid-template-areas: 
    "sidebar header"
    "sidebar main"
    "sidebar footer";
  grid-template-columns: 250px 1fr;
  grid-template-rows: 60px 1fr 60px;
  height: 100vh;
  background: #f0f2f5;
}

.chat-interface .sidebar {
  grid-area: sidebar;
  background: white;
  border-right: 1px solid #e9ecef;
  display: flex;
  flex-direction: column;
}

.chat-interface .sidebar .search {
  padding: 0.75rem;
  border-bottom: 1px solid #e9ecef;
}

.chat-interface .sidebar .contacts {
  flex-grow: 1;
  overflow-y: auto;
}

.chat-interface .sidebar .contact {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  cursor: pointer;
  transition: background 0.2s ease;
}

.chat-interface .sidebar .contact:hover {
  background: #f8f9fa;
}

.chat-interface .header {
  grid-area: header;
  background: white;
  border-bottom: 1px solid #e9ecef;
  display: flex;
  align-items: center;
  padding: 0 1rem;
}

.chat-interface .main {
  grid-area: main;
  display: flex;
  flex-direction: column;
}

.chat-interface .messages {
  flex-grow: 1;
  padding: 1rem;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.chat-interface .message {
  max-width: 70%;
  padding: 0.75rem 1rem;
  border-radius: 18px;
  position: relative;
}

.chat-interface .message.sent {
  align-self: flex-end;
  background: #007bff;
  color: white;
  border-bottom-right-radius: 4px;
}

.chat-interface .message.received {
  align-self: flex-start;
  background: #e9ecef;
  color: #333;
  border-bottom-left-radius: 4px;
}

.chat-interface .footer {
  grid-area: footer;
  background: white;
  border-top: 1px solid #e9ecef;
  display: flex;
  align-items: center;
  padding: 0 1rem;
}

.chat-interface .message-input {
  flex-grow: 1;
  padding: 0.75rem;
  border: 1px solid #e9ecef;
  border-radius: 20px;
  margin-right: 1rem;
}

.chat-interface .send-button {
  background: #007bff;
  color: white;
  border: none;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

## Продвинутые паттерны

### Макет с плавающими элементами

```css
.floating-layout {
  display: grid;
  grid-template-columns: 1fr 300px;
  grid-template-rows: auto 1fr;
  grid-template-areas: 
    "main sidebar"
    "main sidebar";
  min-height: 100vh;
}

.floating-layout .main {
  grid-area: main;
  position: relative;
}

.floating-layout .floating-widget {
  position: absolute;
  top: 2rem;
  right: 2rem;
  width: 250px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  z-index: 100;
}

.floating-layout .sidebar {
  grid-area: sidebar;
  background: #f8f9fa;
  padding: 1.5rem;
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
}
```

### Галерея с деталями

```css
.gallery-with-details {
  display: grid;
  grid-template-columns: 1fr 300px;
  grid-template-areas: 
    "gallery details";
  gap: 2rem;
  padding: 2rem;
  height: 100vh;
}

.gallery-with-details .gallery {
  grid-area: gallery;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  grid-auto-rows: 150px;
  gap: 1rem;
  overflow-y: auto;
  padding: 1rem;
}

.gallery-with-details .gallery .item {
  background: #e9ecef;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.2s ease;
  position: relative;
  overflow: hidden;
}

.gallery-with-details .gallery .item:hover {
  transform: scale(1.05);
}

.gallery-with-details .gallery .item.active {
  border: 3px solid #007bff;
}

.gallery-with-details .details {
  grid-area: details;
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.gallery-with-details .details .preview {
  height: 300px;
  background: #e9ecef;
  display: flex;
  align-items: center;
  justify-content: center;
}

.gallery-with-details .details .info {
  flex-grow: 1;
  padding: 1.5rem;
  overflow-y: auto;
}

.gallery-with-details .details .actions {
  padding: 1.5rem;
  background: #f8f9fa;
  display: flex;
  gap: 1rem;
}
```

## Оптимизация производительности

### Эффективные паттерны

```css
/* Использование contain для оптимизации */
.optimized-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
  contain: layout style paint;
}

.optimized-grid .item {
  contain: layout style paint;
}

/* Избегайте чрезмерной вложенности */
/* ПЛОХО */
.deeply-nested {
  display: grid;
}

.deeply-nested .level1 {
  display: flex;
}

.deeply-nested .level1 .level2 {
  display: grid;
}

/* ЛУЧШЕ: плоская структура */
.flat-structure {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

.flat-structure .item {
  /* Стили элемента */
}
```

## Практические рекомендации

### Когда использовать Grid vs Flexbox

```css
/* Используйте Grid для: */
.two-dimensional-layout {
  /* Двумерные макеты, где важна как колонка, так и строка */
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(4, 200px);
}

/* Используйте Flexbox для: */
.one-dimensional-layout {
  /* Одномерные макеты, где важна только одна ось */
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

/* Используйте оба для: */
.combined-approach {
  /* Сложные макеты, где Grid для основной структуры, Flexbox для внутренних элементов */
  display: grid;
  grid-template-columns: 200px 1fr;
}

.combined-approach .sidebar {
  display: flex;
  flex-direction: column;
}

.combined-approach .main {
  display: flex;
  flex-direction: column;
}
```

## Заключение

Комбинирование Flexbox и Grid позволяет создавать мощные и гибкие макеты. Ключ к успеху - понимание сильных сторон каждой системы и их правильное применение в зависимости от задачи. Grid идеально подходит для двумерных макетов, а Flexbox - для одномерных раскладок и выравнивания элементов.

## Связанные темы

- [[Расширенные возможности Flexbox: глубокое погружение]]
- [[Расширенные возможности Grid: глубокое погружение]]
- [[Сравнение возможностей Flexbox и Grid]]
- [[CSS Layout Systems]]
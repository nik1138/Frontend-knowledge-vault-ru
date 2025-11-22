---
aliases: ["Комбинации Flex и Grid", "Сложные макеты Flex и Grid", "Практические примеры макетов"]
tags: [css, flexbox, grid, layout, advanced]
---

# Практические примеры сложных макетов с сочетанием Flex и Grid

## Введение

Хотя Flexbox и Grid являются мощными системами макета по отдельности, их комбинация позволяет создавать еще более сложные и гибкие структуры. Flexbox идеально подходит для одномерных макетов (одна ось), а Grid - для двумерных структур. Использование обоих подходов в одном проекте дает максимальную гибкость и контроль над макетом.

## Основные принципы комбинации

### Когда использовать Grid внутри Flex

Grid лучше использовать для основной структуры страницы, а Flex - для выравнивания элементов внутри отдельных секций:

```css
.page-container {
  display: grid;
  grid-template-areas: 
    "header"
    "main"
    "sidebar"
    "footer";
  grid-template-rows: auto 1fr auto auto;
  min-height: 100vh;
}

.header {
  grid-area: header;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
}

.main-content {
  grid-area: main;
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 20px;
  padding: 20px;
}

.article {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.sidebar {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 20px;
}

.footer {
  grid-area: footer;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 1rem;
}
```

### Когда использовать Flex внутри Grid

Flex идеально подходит для выравнивания элементов внутри отдельных ячеек Grid:

```css
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  padding: 20px;
}

.product-card {
  display: flex;
  flex-direction: column;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  overflow: hidden;
}

.product-image {
  flex: 0 0 auto; /* Фиксированная высота изображения */
  height: 200px;
  background-color: #f5f5f5;
}

.product-info {
  flex: 1; /* Занимает оставшееся пространство */
  display: flex;
  flex-direction: column;
  padding: 15px;
}

.product-actions {
  flex: 0 0 auto; /* Фиксированная высота для действий */
  display: flex;
  justify-content: space-between;
  padding: 10px 15px;
  background-color: #f9f9f9;
}
```

## Практические примеры

### 1. Адаптивная панель навигации

```html
<nav class="navbar">
  <div class="nav-brand">Логотип</div>
  <ul class="nav-menu">
    <li><a href="#">Главная</a></li>
    <li><a href="#">О нас</a></li>
    <li><a href="#">Услуги</a></li>
    <li><a href="#">Контакты</a></li>
  </ul>
  <div class="nav-toggle">Меню</div>
</nav>
```

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background-color: #333;
  color: white;
}

.nav-brand {
  font-size: 1.5rem;
  font-weight: bold;
}

.nav-menu {
  display: flex;
  gap: 2rem;
  list-style: none;
  margin: 0;
  padding: 0;
}

.nav-menu a {
  color: white;
  text-decoration: none;
  transition: color 0.3s;
}

.nav-menu a:hover {
  color: #ddd;
}

.nav-toggle {
  display: none;
  cursor: pointer;
}

/* Адаптивность */
@media (max-width: 768px) {
  .navbar {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .nav-menu {
    display: none;
    width: 100%;
    flex-direction: column;
    gap: 1rem;
    margin-top: 1rem;
  }
  
  .nav-menu.active {
    display: flex;
  }
  
  .nav-toggle {
    display: block;
    margin-top: 1rem;
  }
}
```

### 2. Сложный макет дашборда

```html
<div class="dashboard">
  <header class="dashboard-header">Шапка дашборда</header>
  <aside class="dashboard-sidebar">Боковая панель</aside>
  <main class="dashboard-main">
    <section class="dashboard-widgets">
      <div class="widget">Виджет 1</div>
      <div class="widget">Виджет 2</div>
      <div class="widget">Виджет 3</div>
    </section>
    <section class="dashboard-content">Основной контент</section>
  </main>
  <footer class="dashboard-footer">Футер</footer>
</div>
```

```css
.dashboard {
  display: grid;
  grid-template-areas: 
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 250px 1fr;
  grid-template-rows: 60px 1fr 40px;
  height: 100vh;
  gap: 1px;
  background-color: #e0e0e0;
}

.dashboard-header {
  grid-area: header;
  background-color: #2c3e50;
  color: white;
  display: flex;
  align-items: center;
  padding: 0 20px;
}

.dashboard-sidebar {
  grid-area: sidebar;
  background-color: #34495e;
  color: white;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.dashboard-main {
  grid-area: main;
  background-color: white;
  display: grid;
  grid-template-rows: auto 1fr;
  overflow: auto;
}

.dashboard-widgets {
  display: flex;
  gap: 20px;
  padding: 20px;
  flex-wrap: wrap;
}

.widget {
  flex: 1;
  min-width: 200px;
  padding: 15px;
  background-color: #ecf0f1;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.dashboard-content {
  padding: 20px;
  overflow: auto;
}

.dashboard-footer {
  grid-area: footer;
  background-color: #bdc3c7;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

### 3. Карточки с адаптивной сеткой

```html
<div class="card-container">
  <div class="card">
    <div class="card-header">Заголовок карточки</div>
    <div class="card-body">Содержимое карточки</div>
    <div class="card-footer">
      <button class="btn-primary">Действие</button>
      <button class="btn-secondary">Отмена</button>
    </div>
  </div>
  <!-- Другие карточки -->
</div>
```

```css
.card-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  padding: 20px;
}

.card {
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  transition: transform 0.2s, box-shadow 0.2s;
}

.card:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.15);
}

.card-header {
  flex: 0 0 auto;
  padding: 15px;
  background-color: #f8f9fa;
  font-weight: bold;
  border-bottom: 1px solid #eee;
}

.card-body {
  flex: 1;
  padding: 15px;
}

.card-footer {
  flex: 0 0 auto;
  padding: 15px;
  background-color: #f8f9fa;
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  border-top: 1px solid #eee;
}

.btn-primary, .btn-secondary {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}
```

### 4. Галерея с адаптивным макетом

```html
<div class="gallery">
  <div class="gallery-item">
    <img src="image1.jpg" alt="Изображение 1">
    <div class="gallery-caption">Описание 1</div>
  </div>
  <div class="gallery-item">
    <img src="image2.jpg" alt="Изображение 2">
    <div class="gallery-caption">Описание 2</div>
  </div>
  <!-- Другие элементы галереи -->
</div>
```

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
}

.gallery-item {
  display: flex;
  flex-direction: column;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  transition: transform 0.3s ease;
}

.gallery-item:hover {
  transform: scale(1.03);
}

.gallery-item img {
  width: 100%;
  height: 200px;
  object-fit: cover;
  display: block;
}

.gallery-caption {
  flex: 1;
  padding: 15px;
  background-color: white;
  display: flex;
  align-items: center;
}
```

### 5. Форма с адаптивным макетом

```html
<form class="responsive-form">
  <div class="form-row">
    <div class="form-group">
      <label for="firstName">Имя</label>
      <input type="text" id="firstName" name="firstName">
    </div>
    <div class="form-group">
      <label for="lastName">Фамилия</label>
      <input type="text" id="lastName" name="lastName">
    </div>
  </div>
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" id="email" name="email">
  </div>
  <div class="form-actions">
    <button type="submit">Отправить</button>
    <button type="reset">Сбросить</button>
  </div>
</form>
```

```css
.responsive-form {
  display: grid;
  gap: 20px;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.form-row {
  display: flex;
  gap: 20px;
}

.form-group {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.form-group label {
  margin-bottom: 5px;
  font-weight: 500;
}

.form-group input {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
  margin-top: 10px;
}

.form-actions button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form-actions button[type="submit"] {
  background-color: #28a745;
  color: white;
}

.form-actions button[type="reset"] {
  background-color: #6c757d;
  color: white;
}

/* Адаптивность */
@media (max-width: 600px) {
  .form-row {
    flex-direction: column;
    gap: 15px;
  }
  
  .form-actions {
    flex-direction: column;
  }
}
```

## Лучшие практики

### 1. Иерархия макетов

Используйте Grid для основной структуры страницы и Flex для выравнивания элементов внутри секций:

```css
.page {
  display: grid;
  grid-template-areas: "header" "nav" "main" "footer";
  grid-template-rows: auto auto 1fr auto;
  min-height: 100vh;
}

.main-content {
  grid-area: main;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.content-section {
  flex: 1;
  width: 100%;
  max-width: 1200px;
}
```

### 2. Адаптивность

Комбинируйте Grid и Flex для создания гибких адаптивных макетов:

```css
.adaptive-container {
  display: grid;
  grid-template-columns: 1fr;
  gap: 20px;
}

@media (min-width: 768px) {
  .adaptive-container {
    grid-template-columns: 1fr 300px;
  }
}

.sidebar {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.main-area {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}
```

### 3. Производительность

Используйте Grid для структуры, чтобы избежать лишних элементов-оберток, и Flex для внутреннего выравнивания:

```css
.product-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
}

.product-card {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.product-image {
  flex: 0 0 auto;
}

.product-details {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.product-actions {
  flex: 0 0 auto;
  display: flex;
  justify-content: space-between;
}
```

## Заключение

Комбинация Flexbox и Grid открывает широкие возможности для создания сложных, гибких и адаптивных макетов. Понимание сильных сторон каждой системы позволяет использовать их наиболее эффективно: Grid для двумерных структур и Flex для одномерного выравнивания.

Для более глубокого изучения отдельных систем рекомендуется ознакомиться с [[Flexbox: детальный разбор всех свойств и их комбинаций]] и [[Grid: детальный разбор всех свойств и их комбинаций]].

## Ссылки

- [MDN: Combining CSS Grid and Flexbox](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_Grid_Layout/Relationship_of_Grid_Layout)
- [CSS-Tricks: When to Use Grid vs When to Use Flexbox](https://css-tricks.com/css-grid-replace-flexbox/)
- [A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [A Complete Guide to Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
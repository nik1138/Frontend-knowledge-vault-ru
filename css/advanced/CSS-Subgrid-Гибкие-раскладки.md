---
aliases: [Subgrid, CSS Subgrid, Вложенная сетка]
tags: [css, advanced, layout, grid]
---

# CSS Subgrid - Гибкие раскладки внутри Grid-контейнеров

## Введение

CSS Subgrid - это мощная функция, которая позволяет дочерним элементам сетки (grid items) наследовать линии сетки от родительского сетки (grid container), создавая согласованные и сложные макеты. Subgrid является частью спецификации CSS Grid Level 2 и позволяет создавать более сложные и согласованные раскладки без дублирования определений сетки.

## Понимание Subgrid

### Проблема, которую решает Subgrid

В традиционной модели CSS Grid, каждый grid-контейнер создает свою собственную сетку, независимую от других сеток. Это означает, что если у вас есть вложенные компоненты, они не могут легко согласовать свои линии сетки с родительской сеткой. Subgrid решает эту проблему, позволяя дочернему контейнеру использовать те же линии сетки, что и его родитель.

### Основные применения

Subgrid особенно полезен в следующих сценариях:

- Сложные формы и макеты форм с согласованным выравниванием
- Карточки с различной структурой, которые должны выравниваться по общей сетке
- Таблицы и списки с расширенной информацией
- Макеты с заголовками, которые охватывают несколько столбцов или строк

## Синтаксис Subgrid

### Определение Subgrid

Чтобы создать subgrid, используйте значение `subgrid` для одного или обоих свойств `grid-template-columns` и `grid-template-rows`:

```css
.child-container {
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
  grid-column: 1 / -1; /* Занимает всю ширину родительской сетки */
}
```

### Поддержка направлений

Subgrid можно применять только в одном направлении:

```css
/* Только в горизонтальном направлении */
.grid-item {
  grid-template-columns: subgrid;
  grid-template-rows: 1fr 2fr 1fr; /* Собственное определение строк */
}

/* Только в вертикальном направлении */
.grid-item {
  grid-template-rows: subgrid;
  grid-template-columns: 1fr 1fr; /* Собственное определение столбцов */
}
```

## Практические примеры

### Пример 1: Согласованная форма

Представьте форму с заголовками и полями ввода, где все элементы должны быть выровнены по общей сетке:

```html
<form class="main-form">
  <div class="form-section">
    <h2>Персональная информация</h2>
    <div class="form-grid">
      <label for="name">Имя:</label>
      <input type="text" id="name" name="name">
      
      <label for="email">Email:</label>
      <input type="email" id="email" name="email">
    </div>
  </div>
  
  <div class="form-section">
    <h2>Контактная информация</h2>
    <div class="form-grid">
      <label for="phone">Телефон:</label>
      <input type="tel" id="phone" name="phone">
      
      <label for="address">Адрес:</label>
      <textarea id="address" name="address"></textarea>
    </div>
  </div>
</form>
```

```css
.main-form {
  display: grid;
  grid-template-columns: [label] 150px [input] 1fr [actions] 200px;
  grid-template-rows: auto 1fr;
  gap: 1rem;
  padding: 1rem;
}

.form-section {
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: auto 1fr;
  grid-column: 1 / -1; /* Занимает всю ширину */
  margin-bottom: 1.5rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-grid {
  display: grid;
  grid-template-columns: subgrid;
  grid-column: 1 / -1; /* Занимает всю ширину */
  gap: 0.5rem;
}

.form-grid label {
  grid-column: label;
  align-self: center;
}

.form-grid input,
.form-grid textarea {
  grid-column: input;
}

.form-grid button {
  grid-column: actions;
  justify-self: start;
}
```

### Пример 2: Карточки с разной структурой

Вот пример использования subgrid для карточек с разной внутренней структурой, которые всё равно выравниваются по общей сетке:

```html
<div class="card-grid">
  <div class="card">
    <img src="image1.jpg" alt="Product" class="card-image">
    <div class="card-content">
      <h3>Продукт 1</h3>
      <p>Описание продукта 1</p>
      <div class="card-footer">
        <span class="price">1000 руб.</span>
        <button>Купить</button>
      </div>
    </div>
  </div>
  
  <div class="card">
    <div class="dual-content">
      <img src="image2.jpg" alt="Product" class="card-image">
      <div class="card-text-content">
        <h3>Продукт 2</h3>
        <p>Более длинное описание продукта 2</p>
      </div>
    </div>
    <div class="card-footer">
      <span class="price">1500 руб.</span>
      <button>Купить</button>
    </div>
  </div>
</div>
```

```css
.card-grid {
  display: grid;
  grid-template-columns: [image] 200px [text] 1fr [actions] 120px;
  grid-auto-rows: minmax(150px, auto);
  gap: 1rem;
  padding: 1rem;
}

.card {
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
  grid-column: 1 / -1; /* Занимает всю ширину сетки */
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.card-image {
  grid-column: image;
  grid-row: 1 / span 2; /* Для первой карточки */
  width: 100%;
  height: auto;
}

.card-content {
  display: grid;
  grid-template-columns: subgrid;
  grid-column: text / actions;
  padding: 1rem;
}

.dual-content {
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
  grid-column: image / actions;
  grid-row: 1;
}

.card-text-content {
  grid-column: text / actions;
  padding: 1rem;
}

.card-footer {
  display: grid;
  grid-template-columns: subgrid;
  grid-column: text / -1;
  padding: 1rem;
  border-top: 1px solid #eee;
  align-items: center;
}

.card-footer .price {
  grid-column: text;
  font-weight: bold;
}

.card-footer button {
  grid-column: actions;
  justify-self: end;
}
```

## Совместимость с Grid Areas

Subgrid прекрасно работает с именованными областями сетки:

```css
.layout {
  display: grid;
  grid-template-areas: 
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  grid-template-columns: [left] 200px [center] 1fr [right] 200px [end];
  grid-template-rows: auto 1fr auto;
  height: 100vh;
}

.main {
  grid-area: main;
  display: grid;
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
}

.main-content {
  grid-column: center / right; /* Использует те же линии, что и родитель */
  grid-row: 2; /* Вторая строка родительской сетки */
  padding: 1rem;
}
```

## Поддержка браузерами

На момент 2023 года:

- Chrome 110+ поддерживает Subgrid
- Firefox поддерживает Subgrid с версии 71
- Safari поддерживает Subgrid с версии 16

Для браузеров без поддержки Subgrid рекомендуется использовать резервные стили.

## Лучшие практики

1. **Используйте Subgrid для согласования вложенных компонентов**: Это основное применение Subgrid - обеспечение согласованности между родительской сеткой и вложенными компонентами.

2. **Создавайте именованные линии сетки**: Использование именованных линий делает код более читаемым и управляемым:
   ```css
   .parent {
     grid-template-columns: [start] 1fr [content] 2fr [end];
   }
   ```

3. **Ограничивайте область применения**: Не используйте Subgrid везде - только там, где действительно нужно согласование сетки.

4. **Обеспечьте резервные стили**: Для браузеров без поддержки Subgrid обеспечьте адекватные резервные стили.

## Сравнение с альтернативами

### Subgrid против вложенных Grid-контейнеров

Вложенные grid-контейнеры создают новую независимую сетку, в то время как subgrid наследует сетку родителя. Это означает, что с subgrid элементы вложенной сетки могут совмещаться с элементами родительской сетки по одним и тем же линиям сетки.

### Subgrid против Flexbox

Flexbox хорош для одномерных раскладок, в то время как subgrid используется для согласования двумерных раскладок с родительскими сетками.

## Ограничения и соображения

1. **Сложность понимания**: Subgrid может быть сложным для понимания, особенно в сложных макетах.

2. **Ограниченная поддержка**: Хотя поддержка растет, некоторые браузеры всё ещё не поддерживают subgrid.

3. **Производительность**: В сложных случаях использование subgrid может повлиять на производительность рендеринга.

## Заключение

CSS Subgrid - это мощный инструмент для создания сложных и согласованных макетов, особенно когда вложенные компоненты должны соответствовать структуре родительской сетки. Его правильное использование может значительно упростить создание сложных UI-компонентов и обеспечить их консистентность. При этом важно понимать, когда и как использовать subgrid, чтобы избежать излишней сложности кода.

## Смежные темы

- [[css-grid-layout]]
- [[layout]]
- [[advanced-layouts]]
- [[grid-comprehensive]]

#css #grid #subgrid #layout #frontend
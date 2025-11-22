---
aliases: ["Современные CSS-селекторы", "Псевдоселекторы :has :is :where :not", "Новые возможности селекторов"]
tags: [css/selectors, modern-css, advanced-selectors, css-features]
---

# Современные селекторы: :has(), :is(), :where(), :not()

## Введение

Современные CSS-селекторы `:has()`, `:is()`, `:where()` и обновленный `:not()` предоставляют новые возможности для написания более краткого, гибкого и мощного CSS-кода. Эти селекторы решают многие проблемы, связанные со сложностью и специфичностью традиционных селекторов.

## :is() - функция для объединения селекторов

### Основы :is()

`:is()` позволяет объединять несколько селекторов в один, упрощая написание CSS и позволяя избежать дублирования:

```css
/* Вместо повторения стилей для разных элементов */
h1, h2, h3, h4, h5, h6 {
  font-family: Arial, sans-serif;
  font-weight: bold;
}

/* Используем :is() */
:is(h1, h2, h3, h4, h5, h6) {
  font-family: Arial, sans-serif;
  font-weight: bold;
}
```

### Специфичность :is()

`:is()` использует **максимальную специфичность** среди переданных селекторов:

```css
/* Специфичность: 0,0,1,1 (берется максимальная из .featured и .highlight) */
.card:is(.featured, .highlight) {
  border: 2px solid #007bff;
}

/* Это эквивалентно: */
.card.featured, .card.highlight {
  border: 2px solid #007bff;
}
```

### Практические примеры :is()

```css
/* Стили для разных типов заголовков */
:is(.page-title, .section-title, .card-title) {
  color: #333;
  margin-bottom: 1rem;
}

/* Стили для элементов с разными состояниями */
.button:is(:hover, :focus, :active) {
  background-color: #0056b3;
  transform: translateY(-2px);
}

/* Сложные комбинации */
:is(header, footer) :is(nav, .menu) :is(.item, .link) {
  padding: 0.5rem 1rem;
}
```

## :where() - нулевая специфичность

### Основы :where()

`:where()` работает аналогично `:is()`, но **не влияет на специфичность**:

```css
/* :is() влияет на специфичность */
.card:is(.featured, .highlight) { /* Специфичность: 0,0,2,1 */
  background-color: #f8f9fa;
}

/* :where() не влияет на специфичность */
.card:where(.featured, .highlight) { /* Специфичность: 0,0,1,1 */
  background-color: #f8f9fa;
}
```

### Практические примеры :where()

```css
/* Сброс стилей с низкой специфичностью */
:where(.button, .link, .nav-item):focus {
  outline: 2px solid #007bff;
  outline-offset: 2px;
}

/* Утилиты с низкой специфичностью */
:where(.text-bold, .text-italic, .text-underline) {
  color: #333;
}

/* Стили для разных типов форм */
:where(input, textarea, select):invalid {
  border-color: #dc3545;
}
```

### Сравнение :is() и :where()

```css
/* :is() - увеличивает специфичность */
.form-control:is(:valid, :invalid) {
  /* Специфичность: 0,0,2,1 */
  transition: border-color 0.3s;
}

/* :where() - не увеличивает специфичность */
.form-control:where(:valid, :invalid) {
  /* Специфичность: 0,0,1,1 */
  transition: border-color 0.3s;
}
```

## :not() - расширенные возможности

### Современный :not()

Современная версия `:not()` позволяет передавать **несколько селекторов**:

```css
/* Традиционный :not() - только один селектор */
.item:not(.disabled) { opacity: 1; }

/* Современный :not() - несколько селекторов */
.item:not(.disabled, .hidden, .archived) {
  opacity: 1;
}

/* Это эквивалентно: */
.item:not(.disabled):not(.hidden):not(.archived) {
  opacity: 1;
}
```

### Практические примеры :not()

```css
/* Все элементы, кроме определенных классов */
.content > *:not(.ad, .sponsored, .promoted) {
  margin-bottom: 1rem;
}

/* Интерактивные элементы, кроме определенных */
.interactive:not(button, .disabled, [disabled]) {
  cursor: pointer;
}

/* Сложные комбинации */
.card:not(.featured, .premium):hover {
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}
```

## :has() - родительский селектор

### Основы :has()

`:has()` позволяет выбирать элементы, **содержащие** другие элементы или соответствующие другим условиям:

```css
/* Выбирает заголовок, за которым следует .content */
h2:has(+ .content) {
  border-bottom: 2px solid #007bff;
  padding-bottom: 0.5rem;
}

/* Выбирает контейнер, содержащий активный элемент */
.container:has(.active) {
  border: 2px solid #007bff;
}

/* Выбирает форму с ошибками */
.form:has(.error) {
  background-color: #fff5f5;
}
```

### Практические примеры :has()

```css
/* Навигация с активным элементом */
.nav:has(.active) {
  background-color: #f8f9fa;
  border-radius: 8px;
}

/* Заголовок перед активным разделом */
h2:has(+ .active-section) {
  color: #007bff;
  text-shadow: 0 0 8px rgba(0, 123, 255, 0.5);
}

/* Таблица с отфильтрованными строками */
.table:has(tbody tr.filtered) {
  opacity: 0.7;
}

/* Карточка с изображением */
.card:has(img) {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 1rem;
}

/* Форма с незаполненными обязательными полями */
.form:has(input:required:invalid) {
  border: 2px solid #dc3545;
}
```

### Сложные примеры :has()

```css
/* Контейнер, содержащий хотя бы один элемент с определенным атрибутом */
.content:has([data-featured="true"]) {
  background: linear-gradient(135deg, #f5f7fa, #e4edf5);
}

/* Навигация, содержащая активную ссылку */
.nav:has(a[aria-current="page"]) {
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

/* Секция с заголовком определенного уровня */
section:has(h1, h2) {
  margin-top: 2rem;
}
```

## Комбинация современных селекторов

### Сложные комбинации

```css
/* Комбинация всех современных селекторов */
.container:has(.active):is(.featured, .premium):not(.disabled) {
  background-color: #fff;
  border: 2px solid #007bff;
  border-radius: 8px;
}

/* Более читаемая версия с :where() */
.container:has(.active):where(.featured, .premium):not(.disabled, .archived) {
  background-color: #fff;
  border: 2px solid #007bff;
  border-radius: 8px;
}
```

### Практический пример: адаптивная сетка

```html
<div class="grid-container">
  <div class="grid-item">Элемент 1</div>
  <div class="grid-item featured">Особый элемент</div>
  <div class="grid-item">Элемент 3</div>
  <div class="grid-item">Элемент 4</div>
</div>
```

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

/* Особые элементы */
.grid-item:is(.featured, .highlight) {
  grid-column: span 2;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
}

/* Обычные элементы, кроме особых */
.grid-item:not(.featured, .highlight):hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
}

/* Контейнер с особыми элементами */
.grid-container:has(.featured) {
  border: 2px solid #667eea;
  border-radius: 12px;
  padding: 1rem;
}

/* Элементы с низкой специфичностью */
.grid-item:where(:first-child, :last-child) {
  border-radius: 8px;
}
```

### Пример: интеллектуальная форма

```html
<form class="smart-form">
  <div class="form-group">
    <input type="text" class="form-input" required>
    <span class="error-message">Ошибка</span>
  </div>
  <div class="form-group">
    <input type="email" class="form-input" required>
  </div>
</form>
```

```css
.smart-form {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  border-radius: 8px;
  background-color: #f8f9fa;
  transition: all 0.3s ease;
}

/* Форма с ошибками */
.smart-form:has(.error-message:empty) {
  background-color: #fff;
}

.smart-form:has(.error-message:not(:empty)) {
  background-color: #fff5f5;
  border: 2px solid #dc3545;
}

/* Группы полей */
.form-group {
  margin-bottom: 1rem;
  padding: 1rem;
  border-radius: 4px;
  transition: all 0.3s ease;
}

/* Группа с ошибкой */
.form-group:has(.error-message:not(:empty)) {
  background-color: #fff5f5;
  border-left: 4px solid #dc3545;
}

/* Группа с валидным полем */
.form-group:has(.form-input:valid) {
  background-color: #f0fff4;
  border-left: 4px solid #28a745;
}

/* Все поля формы, кроме особых */
.form-input:where(:not(.special, .custom)) {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
}

/* Состояния полей */
.form-input:is(:focus, :active) {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
}

.form-input:is(:valid, :focus:valid) {
  border-color: #28a745;
}

.form-input:is(:invalid:not(:placeholder-shown), :focus:invalid) {
  border-color: #dc3545;
}
```

## Производительность и ограничения

### Производительность :has()

`:has()` может быть **вычислительно затратным**, особенно с глубокими селекторами:

```css
/* Может быть медленным */
.container:has(div > span + p ~ ul li a[href^="https://"]) {
  /* стили */
}

/* Более производительная альтернатива */
.container.has-external-links {
  /* стили */
}
/* Управление классом через JavaScript */
```

### Оптимизация использования

```css
/* Хорошо: простые условия */
.card:has(.featured) { }

/* Лучше: использовать классы для сложных условий */
.card.has-featured-content { }

/* Избегать глубоких вложенностей в :has() */
.container:has(.deeply .nested .structure) { }
```

## Совместимость и полифиллы

### Поддержка браузерами

| Селектор | IE | Chrome | Firefox | Safari | Edge |
|----------|----|--------|---------|--------|------|
| :is() | Нет | 88+ | 78+ | 15.4+ | 88+ |
| :where() | Нет | 88+ | 78+ | 15.4+ | 88+ |
| :not() (множественный) | Нет | 86+ | 51+ | 15.4+ | 86+ |
| :has() | Нет | 105+ | 121+ | 15.4+ | 105+ |

### Резервные решения

```css
/* Резервное решение для :has() */
@supports not selector(:has(*)) {
  .form.error { /* Класс добавляется через JavaScript */
    background-color: #fff5f5;
  }
  
  .nav.active { /* Класс добавляется через JavaScript */
    background-color: #f8f9fa;
  }
}

/* Резервное решение для :is() и :where() */
@supports not selector(:is(*)) {
  .title, .subtitle, .heading {
    font-weight: bold;
  }
}
```

### JavaScript полифиллы

```javascript
// Простой полифилл для :has() (упрощенный пример)
function polyfillHas() {
  // Находим элементы, содержащие определенные селекторы
  const elements = document.querySelectorAll('*');
  
  elements.forEach(el => {
    // Проверяем, содержит ли элемент дочерние элементы, соответствующие селектору
    if (el.querySelector('.active')) {
      el.classList.add('has-active');
    }
  });
}

// Вызываем при загрузке и при изменениях DOM
document.addEventListener('DOMContentLoaded', polyfillHas);
```

## Лучшие практики

### 1. Использование :where() для утилит

```css
/* Утилиты с низкой специфичностью */
.text-center:where(.mobile, .tablet, .desktop) { text-align: center; }
.text-bold:where(.title, .subtitle) { font-weight: bold; }
```

### 2. Комбинация :is() и :where()

```css
/* :is() для логических групп, :where() для утилит */
.card:is(.featured, .premium):where(.active, .highlighted) {
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
}
```

### 3. Избегание чрезмерной сложности

```css
/* Слишком сложно */
.container:has(.nested:is(.item, .element):not(.disabled)):where(.active, .featured) { }

/* Лучше использовать классы */
.container.has-special-content.active { }
```

### 4. Документирование сложных селекторов

```css
/* 
 * Селектор: .container:has(.featured):is(.active, .premium)
 * Описание: Стилизует контейнер, содержащий особый элемент и находящийся в активном/премиум состоянии
 * Применение: Используется для выделения важных секций с особым контентом
 */
.container:has(.featured):is(.active, .premium) {
  border: 2px solid #007bff;
  border-radius: 8px;
  padding: 1rem;
}
```

## Заключение

Современные CSS-селекторы `:has()`, `:is()`, `:where()` и расширенный `:not()` значительно расширяют возможности CSS:

1. `:is()` - объединяет селекторы с учетом специфичности
2. `:where()` - объединяет селекторы без влияния на специфичность
3. `:not()` - теперь поддерживает несколько селекторов
4. `:has()` - позволяет выбирать элементы на основе их содержимого

Эти селекторы позволяют писать более краткий, читаемый и гибкий CSS-код, хотя и требуют внимательного подхода к производительности и совместимости.

## См. также

- [[Псевдоселекторы: углубленное руководство]]
- [[Селекторы и специфичность: продвинутые аспекты]]
- [[Оптимизация селекторов для лучшей производительности]]
- [[CSS-архитектура: методологии и лучшие практики]]
- [[Продвинутые комбинаторы и их производительность]]
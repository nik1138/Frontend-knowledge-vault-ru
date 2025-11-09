# Продвинутые возможности - Псевдоэлементы

Псевдоэлементы в CSS позволяют стилизовать определенные части элемента, которые не существуют в HTML-разметке. Они создаются с помощью двоеточия (`::`) и позволяют добавлять визуальные элементы или стилизовать специфические части элемента без изменения HTML-кода.

## Основные псевдоэлементы

### ::before и ::after

Псевдоэлементы `::before` и `::after` позволяют добавлять контент перед или после содержимого элемента:

```css
/* Базовое использование */
.quote::before {
  content: """;
  font-size: 2em;
  color: #6c757d;
  font-weight: bold;
}

.quote::after {
  content: """;
  font-size: 2em;
  color: #6c757d;
  font-weight: bold;
}

/* Добавление иконок */
.button::before {
  content: "★";
  margin-right: 5px;
}

/* Стилизация декоративных элементов */
.card {
  position: relative;
  padding: 20px;
}

.card::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(to right, #ff0000, #0000ff);
}

/* Счетчики и номера */
.numbered-list li {
  counter-increment: item;
  position: relative;
  padding-left: 30px;
}

.numbered-list li::before {
  content: counter(item);
  position: absolute;
  left: 0;
  top: 0;
  background: #007bff;
  color: white;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8em;
  font-weight: bold;
}
```

### ::first-letter и ::first-line

Эти псевдоэлементы позволяют стилизовать первую букву или первую строку текста:

```css
/* Стилизация первой буквы (буквица) */
.drop-cap::first-letter {
  font-size: 3em;
  font-weight: bold;
  float: left;
  line-height: 1;
  margin-right: 5px;
  color: #e74c3c;
  font-family: Georgia, serif;
}

/* Стилизация первой строки */
.article-intro::first-line {
  font-weight: bold;
  color: #495057;
  font-variant: small-caps;
}

/* Комбинация с другими псевдоэлементами */
.quote {
  font-style: italic;
  position: relative;
  padding: 20px 20px 20px 60px;
}

.quote::first-letter {
  font-size: 2.5em;
  font-weight: bold;
  float: left;
  margin-right: 5px;
  color: #007bff;
}

.quote::before {
  content: """;
  position: absolute;
  left: 10px;
  top: 5px;
  font-size: 3em;
  color: #dee2e6;
  line-height: 1;
}
```

### ::selection

Псевдоэлемент `::selection` позволяет стилизовать выделенный пользователем текст:

```css
/* Базовое выделение */
::selection {
  background-color: #007bff;
  color: white;
}

/* Выделение в определенных элементах */
.quote::selection {
  background-color: rgba(231, 76, 60, 0.8);
  color: white;
}

/* Разные стили для разных элементов */
h1::selection {
  background-color: #6f42c1;
  color: white;
}

p::selection {
  background-color: #28a745;
  color: white;
}

/* Обработка разных префиксов для лучшей совместимости */
::-moz-selection {
  background-color: #007bff;
  color: white;
}

::selection {
  background-color: #007bff;
  color: white;
}
```

## Продвинутые псевдоэлементы

### ::placeholder

Псевдоэлемент для стилизации placeholder текста в полях ввода:

```css
/* Базовая стилизация placeholder */
input::placeholder {
  color: #6c757d;
  font-style: italic;
  opacity: 0.7;
}

/* Стилизация placeholder для разных типов полей */
input[type="email"]::placeholder {
  color: #28a745;
}

input[type="password"]::placeholder {
  color: #dc3545;
}

/* Анимация placeholder */
input:focus::placeholder {
  opacity: 0.5;
  transition: opacity 0.3s ease;
}

/* Сложная стилизация placeholder */
.search-input::placeholder {
  color: #495057;
  background: url(search-icon.svg) left center no-repeat;
  padding-left: 25px;
  font-style: italic;
}
```

### ::marker

Псевдоэлемент для стилизации маркеров списков:

```css
/* Стилизация маркеров неупорядоченных списков */
.custom-list {
  list-style: none;
  padding-left: 0;
}

.custom-list li {
  position: relative;
  padding-left: 30px;
  margin-bottom: 10px;
}

.custom-list li::marker {
  color: #e74c3c;
  font-size: 1.2em;
  font-weight: bold;
}

/* Альтернативный способ стилизации маркеров */
.fancy-list li {
  position: relative;
  padding-left: 30px;
}

.fancy-list li::before {
  content: "→";
  position: absolute;
  left: 0;
  color: #3498db;
  font-weight: bold;
}

/* Стилизация маркеров упорядоченных списков */
.numbered-list {
  counter-reset: item;
}

.numbered-list li {
  counter-increment: item;
  position: relative;
  padding-left: 40px;
  margin-bottom: 10px;
}

.numbered-list li::before {
  content: counter(item) ".";
  position: absolute;
  left: 0;
  top: 0;
  background: #007bff;
  color: white;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.9em;
  font-weight: bold;
}
```

### ::backdrop

Псевдоэлемент для стилизации фона модальных окон (используется с `<dialog>`):

```css
/* Стилизация backdrop для модальных окон */
dialog::backdrop {
  background-color: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(5px);
}

/* Анимация backdrop */
dialog[open]::backdrop {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

## Практические примеры

### Пример 1: Декоративные карточки

```css
.decorative-card {
  position: relative;
  background: white;
  border: 1px solid #dee2e6;
  border-radius: 8px;
  padding: 25px;
  margin: 20px 0;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

/* Угловые декорации */
.decorative-card::before,
.decorative-card::after {
  content: "";
  position: absolute;
  width: 20px;
  height: 20px;
  border: 2px solid #007bff;
}

.decorative-card::before {
  top: 10px;
  left: 10px;
  border-right: none;
  border-bottom: none;
}

.decorative-card::after {
  bottom: 10px;
  right: 10px;
  border-left: none;
  border-top: none;
}

/* Виньетка */
.decorative-card::backdrop {
  /* Не применимо к обычным элементам */
}

/* Декоративная линия */
.decorative-card .card-title {
  position: relative;
  padding-bottom: 10px;
  margin-bottom: 15px;
}

.decorative-card .card-title::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 50px;
  height: 2px;
  background: linear-gradient(to right, #007bff, #6f42c1);
}
```

### Пример 2: Интерактивные кнопки

```css
.interactive-button {
  position: relative;
  padding: 12px 24px;
  background: linear-gradient(45deg, #007bff, #6f42c1);
  color: white;
  border: none;
  border-radius: 50px;
  cursor: pointer;
  overflow: hidden;
  transition: all 0.3s ease;
  font-weight: bold;
}

/* Эффект волны при клике */
.interactive-button::before {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
  transition: width 0.6s, height 0.6s;
  transform: translate(-50%, -50%);
}

.interactive-button:active::before {
  width: 300px;
  height: 300px;
}

/* Текстовый эффект */
.interactive-button::after {
  content: attr(data-hover-text);
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, #6f42c1, #007bff);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.interactive-button:hover::after {
  opacity: 1;
}

.interactive-button:hover span {
  opacity: 0;
}
```

### Пример 3: Прогресс-бар с анимацией

```css
.progress-container {
  width: 100%;
  height: 20px;
  background-color: #e9ecef;
  border-radius: 10px;
  overflow: hidden;
  position: relative;
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #007bff, #0056b3);
  border-radius: 10px;
  width: 0%;
  transition: width 0.3s ease;
  position: relative;
}

/* Анимация загрузки */
.progress-bar.loading::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.2),
    transparent
  );
  animation: loading 1.5s infinite;
}

@keyframes loading {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

/* Отображение процентов */
.progress-bar::before {
  content: attr(data-percent);
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: white;
  font-size: 0.7em;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
}
```

### Пример 4: Кастомные чекбоксы

```css
.custom-checkbox-container {
  display: flex;
  align-items: center;
  cursor: pointer;
  user-select: none;
  margin-bottom: 10px;
}

.custom-checkbox-input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
}

.custom-checkbox-mark {
  position: relative;
  height: 20px;
  width: 20px;
  background-color: #eee;
  border: 2px solid #ccc;
  border-radius: 4px;
  margin-right: 10px;
  transition: all 0.3s ease;
}

/* Псевдоэлемент для галочки */
.custom-checkbox-mark::after {
  content: "";
  position: absolute;
  display: none;
  left: 6px;
  top: 2px;
  width: 5px;
  height: 10px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

/* Стили при наведении */
.custom-checkbox-container:hover .custom-checkbox-mark {
  background-color: #f8f9fa;
  border-color: #007bff;
}

/* Стили для отмеченного чекбокса */
.custom-checkbox-input:checked ~ .custom-checkbox-mark {
  background-color: #007bff;
  border-color: #007bff;
}

.custom-checkbox-input:checked ~ .custom-checkbox-mark::after {
  display: block;
}

/* Анимация при переключении */
.custom-checkbox-mark {
  transition: all 0.3s ease;
}

.custom-checkbox-input:checked ~ .custom-checkbox-mark {
  animation: checkmark 0.3s ease;
}

@keyframes checkmark {
  0% { transform: scale(0.8); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}
```

## Современные возможности

### CSS-переменные с псевдоэлементами

```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --accent-color: #28a745;
}

.themed-element::before {
  content: "";
  position: absolute;
  width: 20px;
  height: 20px;
  background-color: var(--primary-color);
  border-radius: 50%;
}

.themed-element:hover::before {
  background-color: var(--accent-color);
  transform: scale(1.2);
}
```

### Псевдоэлементы с CSS Grid и Flexbox

```css
.grid-with-decoration {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  position: relative;
}

/* Декоративная сетка */
.grid-with-decoration::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: 
    linear-gradient(rgba(0, 123, 255, 0.1) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0, 123, 255, 0.1) 1px, transparent 1px);
  background-size: 50px 50px;
  z-index: -1;
  pointer-events: none;
}
```

## Лучшие практики

### 1. Использование content свойства

```css
/* Всегда указывайте content для ::before и ::after */
.element::before {
  content: ""; /* Даже если пустой */
  display: block;
  width: 10px;
  height: 10px;
  background: red;
}

/* Используйте осмысленные значения content */
.icon::before {
  content: "★"; /* Символ */
  content: url(icon.png); /* Изображение */
  content: counter(item); /* Счетчик */
  content: attr(data-label); /* Атрибут */
}
```

### 2. Позиционирование

```css
/* Убедитесь, что родительский элемент имеет position: relative */
.decoration {
  position: relative;
}

.decoration::after {
  content: "";
  position: absolute;
  top: 0;
  right: 0;
  width: 10px;
  height: 100%;
  background: linear-gradient(to bottom, red, blue);
}
```

### 3. Доступность

```css
/* Убедитесь, что псевдоэлементы не нарушают доступность */
.button-with-icon::before {
  content: "★";
  display: inline-block;
  width: 16px;
  height: 16px;
  /* Добавьте aria-label для скринридеров */
}

/* Используйте CSS для визуальных эффектов, не для контента */
/* Не добавляйте важную информацию через псевдоэлементы */
```

## Совместимость

Большинство псевдоэлементов имеют отличную поддержку в современных браузерах. Некоторые старые браузеры требуют одинарное двоеточие (`:before`, `:after`), но современные стандарты используют двойное двоеточие (`::before`, `::after`).

## Заключение

Псевдоэлементы - мощный инструмент для добавления визуальных элементов и стилизации частей элементов без изменения HTML-разметки. Они позволяют создавать сложные визуальные эффекты, кастомные компоненты и улучшать пользовательский интерфейс. Правильное использование псевдоэлементов может значительно улучшить внешний вид и функциональность веб-страниц.

#programming #css #pseudo-elements #advanced #web-development #frontend
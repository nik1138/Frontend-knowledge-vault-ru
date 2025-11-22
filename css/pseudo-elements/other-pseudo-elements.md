---
aliases: ["Другие псевдоэлементы", "selection", "placeholder", "marker", "CSS псевдоэлементы"]
tags: [css, pseudo-elements, selection, placeholder, marker, advanced]
---

# Другие псевдоэлементы (selection, placeholder, marker и др.)

## Введение

Помимо уже рассмотренных псевдоэлементов, CSS предоставляет множество других, которые позволяют стилизовать специфические части элементов. Эти псевдоэлементы дают возможность контролировать внешний вид выделенного текста, плейсхолдеров, маркеров списков и других интерактивных элементов формы.

## Псевдоэлемент ::selection

Псевдоэлемент `::selection` применяет стили к части документа, которая была выделена пользователем. Он позволяет кастомизировать цвет выделения текста.

### Основное использование

```css
/* Общее выделение для всего документа */
::selection {
  background-color: #007bff;
  color: white;
}

/* Выделение для конкретного элемента */
.highlighted-text::selection {
  background-color: #ffc107;
  color: #212529;
}

/* Разное выделение для разных элементов */
.quote::selection {
  background-color: rgba(255, 193, 7, 0.7);
  color: #212529;
}

.code-block::selection {
  background-color: #6f42c1;
  color: white;
}
```

### Продвинутые примеры

```css
/* Анимированное выделение */
.animated-selection::selection {
  animation: selectionGlow 1s infinite alternate;
}

@keyframes selectionGlow {
  0% { background-color: #007bff; }
  100% { background-color: #6610f2; }
}

/* Градиентное выделение */
.gradient-selection::selection {
  background: linear-gradient(45deg, #007bff, #6610f2);
  color: white;
}
```

### Совместимость с префиксами

```css
/* Совместимая версия с префиксами для старых браузеров */
::selection {
  background-color: #007bff;
  color: white;
}

::-moz-selection {
  background-color: #007bff;
  color: white;
}
```

## Псевдоэлементы форм

### ::placeholder

Псевдоэлемент `::placeholder` применяет стили к тексту-заполнителю (placeholder) в полях ввода.

```css
.input-field::placeholder {
  color: #6c757d;
  font-style: italic;
  opacity: 0.7;
}

/* Разные стили для разных типов полей */
.email-input::placeholder {
  color: #007bff;
}

.password-input::placeholder {
  color: #dc3545;
}

.search-input::placeholder {
  text-align: center;
  font-weight: 300;
}

/* Анимация плейсхолдера */
.animated-placeholder::placeholder {
  animation: placeholderPulse 2s infinite;
  transition: opacity 0.3s ease;
}

@keyframes placeholderPulse {
  0% { opacity: 0.5; }
  50% { opacity: 1; }
  100% { opacity: 0.5; }
}
```

### ::input-placeholder (устаревший, для совместимости)

```css
/* Префиксы для старых браузеров */
.input-field::-webkit-input-placeholder { /* Chrome/Opera/Safari */
  color: #6c757d;
  opacity: 0.7;
}
.input-field::-moz-placeholder { /* Firefox 19+ */
  color: #6c757d;
  opacity: 0.7;
}
.input-field:-ms-input-placeholder { /* IE 10+ */
  color: #6c757d;
  opacity: 0.7;
}
.input-field:-moz-placeholder { /* Firefox 18- */
  color: #6c757d;
  opacity: 0.7;
}
```

## Псевдоэлементы списков

### ::marker

Псевдоэлемент `::marker` позволяет стилизовать маркеры списков (например, точки, цифры, кастомные символы).

```css
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
  content: "• ";
  color: #007bff;
  font-size: 1.5em;
}

/* Альтернативный способ стилизации маркеров */
.numbered-list {
  list-style: none;
  counter-reset: item-counter;
}

.numbered-list li {
  counter-increment: item-counter;
  position: relative;
  padding-left: 40px;
  margin-bottom: 15px;
}

.numbered-list li::before {
  content: counter(item-counter) ".";
  position: absolute;
  left: 0;
  top: 0;
  background-color: #007bff;
  color: white;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}

/* Стилизация маркеров с иконками */
.icon-list {
  list-style: none;
  padding-left: 0;
}

.icon-list li {
  position: relative;
  padding-left: 30px;
  margin-bottom: 10px;
}

.icon-list li::before {
  content: "✓";
  position: absolute;
  left: 0;
  top: 0;
  color: #28a745;
  font-weight: bold;
}
```

### Примеры продвинутой стилизации маркеров

```css
/* Маркеры с градиентом */
.gradient-marker li::marker {
  color: #007bff;
  font-weight: bold;
}

.gradient-marker li::before {
  content: "";
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 10px;
  height: 10px;
  background: linear-gradient(45deg, #007bff, #6610f2);
  border-radius: 50%;
}

/* Анимированные маркеры */
.animated-marker li {
  position: relative;
  padding-left: 30px;
  margin-bottom: 15px;
  transition: transform 0.3s ease;
}

.animated-marker li:hover {
  transform: translateX(10px);
}

.animated-marker li::before {
  content: "▶";
  position: absolute;
  left: 0;
  top: 0;
  transition: transform 0.3s ease;
}

.animated-marker li:hover::before {
  transform: translateX(5px);
}
```

## Псевдоэлементы для элементов формы

### ::-webkit-slider-thumb и ::-moz-range-thumb

Для стилизации ползунков input[type="range"]:

```css
.range-slider {
  width: 100%;
  height: 8px;
  border-radius: 4px;
  background: #dee2e6;
  outline: none;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #007bff;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

.range-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #007bff;
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
}

/* Анимация при наведении */
.range-slider::-webkit-slider-thumb:hover {
  background: #0056b3;
  transform: scale(1.2);
}
```

### ::-webkit-progress-bar и ::-webkit-progress-value

Для стилизации элемента progress:

```css
.progress-bar {
  width: 100%;
  height: 20px;
  border-radius: 10px;
  overflow: hidden;
}

.progress-bar::-webkit-progress-bar {
  background-color: #e9ecef;
  border-radius: 10px;
}

.progress-bar::-webkit-progress-value {
  background: linear-gradient(90deg, #007bff, #0056b3);
  border-radius: 10px;
  transition: width 0.3s ease;
}

.progress-bar::-moz-progress-bar {
  background: linear-gradient(90deg, #007bff, #0056b3);
  border-radius: 10px;
}
```

## Псевдоэлементы для элементов с атрибутами

### ::cue (для элементов track)

Для стилизации субтитров в элементах video:

```css
video::cue {
  background-color: rgba(0, 0, 0, 0.8);
  color: white;
  font-size: 16px;
  padding: 5px 10px;
  border-radius: 4px;
}

video::cue(b) {
  font-weight: bold;
  color: yellow;
}
```

## Практические примеры

### Интерактивная форма с улучшенной стилизацией

```html
<form class="styled-form">
  <div class="form-group">
    <input type="text" class="input-field" placeholder="Введите ваше имя">
  </div>
  <div class="form-group">
    <input type="email" class="input-field" placeholder="Введите ваш email">
  </div>
  <div class="form-group">
    <textarea class="textarea-field" placeholder="Ваше сообщение"></textarea>
  </div>
  <button type="submit" class="submit-btn">Отправить</button>
</form>
```

```css
.styled-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.input-field,
.textarea-field {
  width: 100%;
  padding: 12px 15px;
  border: 2px solid #dee2e6;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s, box-shadow 0.3s;
  box-sizing: border-box;
}

.input-field:focus,
.textarea-field:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.input-field::placeholder,
.textarea-field::placeholder {
  color: #6c757d;
  opacity: 0.7;
  transition: opacity 0.3s ease;
}

.input-field:focus::placeholder,
.textarea-field:focus::placeholder {
  opacity: 0.5;
}

/* Стили для выделения текста в полях формы */
.input-field::selection,
.textarea-field::selection {
  background-color: #007bff;
  color: white;
}

.submit-btn {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s;
}

.submit-btn:hover {
  background-color: #0056b3;
}
```

### Список с кастомными маркерами

```html
<ul class="custom-markers-list">
  <li>Первый элемент списка</li>
  <li>Второй элемент списка</li>
  <li>Третий элемент списка</li>
  <li>Четвертый элемент списка</li>
</ul>
```

```css
.custom-markers-list {
  list-style: none;
  padding-left: 0;
  max-width: 400px;
  margin: 0 auto;
}

.custom-markers-list li {
  position: relative;
  padding: 10px 10px 10px 40px;
  margin-bottom: 10px;
  background-color: #f8f9fa;
  border-radius: 4px;
  border-left: 4px solid #007bff;
  transition: transform 0.3s ease, background-color 0.3s ease;
}

.custom-markers-list li:hover {
  background-color: #e9ecef;
  transform: translateX(5px);
}

.custom-markers-list li::before {
  content: "✓";
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  background-color: #007bff;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8em;
  font-weight: bold;
}

/* Анимация маркера при наведении */
.custom-markers-list li:hover::before {
  animation: pulse 0.5s ease;
}

@keyframes pulse {
  0% { transform: translateY(-50%) scale(1); }
  50% { transform: translateY(-50%) scale(1.2); }
  100% { transform: translateY(-50%) scale(1); }
}
```

### Текст с кастомным выделением

```html
<div class="content-with-selection">
  <p>Этот текст можно выделить, и вы увидите кастомный стиль выделения. Попробуйте выделить часть этого предложения, чтобы увидеть, как работает псевдоэлемент ::selection.</p>
  <p>Второй абзац также поддерживает кастомное выделение, но с другим стилем.</p>
</div>
```

```css
.content-with-selection p {
  margin-bottom: 20px;
  line-height: 1.6;
}

.content-with-selection p:first-of-type::selection {
  background: linear-gradient(45deg, #007bff, #6610f2);
  color: white;
  text-shadow: 0 0 2px rgba(0,0,0,0.5);
}

.content-with-selection p:last-of-type::selection {
  background-color: #ffc107;
  color: #212529;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='10' height='10'%3E%3Crect width='10' height='10' fill='%23ffc107'/%3E%3C/svg%3E");
}
```

## Расширенные примеры

### Интерактивный прогресс-бар с деталями

```html
<div class="progress-container">
  <label for="progress">Загрузка файла</label>
  <progress id="progress" class="advanced-progress" value="65" max="100">65%</progress>
  <div class="progress-details">
    <span class="progress-text">Загружено: <span class="progress-value">65%</span></span>
    <span class="progress-speed">Скорость: 2.4 MB/s</span>
  </div>
</div>
```

```css
.progress-container {
  max-width: 500px;
  margin: 20px auto;
  padding: 20px;
}

.advanced-progress {
  width: 100%;
  height: 24px;
  border-radius: 12px;
  overflow: hidden;
  border: none;
  box-shadow: inset 0 1px 3px rgba(0,0,0,0.2);
}

.advanced-progress::-webkit-progress-bar {
  background-color: #e9ecef;
  border-radius: 12px;
}

.advanced-progress::-webkit-progress-value {
  background: linear-gradient(90deg, #007bff, #0056b3);
  border-radius: 12px;
  position: relative;
}

.advanced-progress::-webkit-progress-value::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent
  );
  animation: progressShine 2s infinite;
}

.advanced-progress::-moz-progress-bar {
  background: linear-gradient(90deg, #007bff, #0056b3);
  border-radius: 12px;
}

.progress-details {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
  font-size: 0.9em;
  color: #6c757d;
}

.progress-value {
  font-weight: bold;
  color: #007bff;
}

@keyframes progressShine {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}
```

### Стилизованный диапазонный слайдер

```html
<div class="slider-container">
  <label for="range-slider">Яркость: <span class="value-display">50</span>%</label>
  <input type="range" id="range-slider" class="range-slider" min="0" max="100" value="50">
</div>
```

```css
.slider-container {
  max-width: 400px;
  margin: 20px auto;
  padding: 20px;
}

.range-slider {
  width: 100%;
  height: 8px;
  border-radius: 4px;
  background: linear-gradient(to right, #e9ecef, #007bff 50%, #e9ecef);
  outline: none;
  margin: 20px 0;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: white;
  cursor: pointer;
  border: 2px solid #007bff;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
  transition: all 0.2s ease;
}

.range-slider::-webkit-slider-thumb:hover {
  transform: scale(1.2);
  background: #007bff;
}

.range-slider::-moz-range-thumb {
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: white;
  cursor: pointer;
  border: 2px solid #007bff;
  box-shadow: 0 2px 6px rgba(0,0,0,0.3);
}

.range-slider::-moz-range-thumb:hover {
  background: #007bff;
}

.value-display {
  font-weight: bold;
  color: #007bff;
}
```

## Заключение

Другие псевдоэлементы CSS предоставляют мощные возможности для тонкой настройки внешнего вида различных элементов интерфейса. Их правильное использование позволяет создавать более привлекательные и интерактивные пользовательские интерфейсы, улучшая общий пользовательский опыт.

Для изучения других псевдоэлементов рекомендуется ознакомиться с [[Псевдоэлементы ::before и ::after: продвинутые техники]] и [[Псевдоэлементы ::first-line и ::first-letter: стилизация текста]].

## Ссылки

- [MDN: ::selection](https://developer.mozilla.org/ru/docs/Web/CSS/::selection)
- [MDN: ::placeholder](https://developer.mozilla.org/ru/docs/Web/CSS/::placeholder)
- [MDN: ::marker](https://developer.mozilla.org/ru/docs/Web/CSS/::marker)
- [CSS-Tricks: ::selection](https://css-tricks.com/almanac/selectors/s/selection/)
- [CSS-Tricks: ::placeholder](https://css-tricks.com/almanac/selectors/p/placeholder/)
- [CSS-Tricks: ::marker](https://css-tricks.com/almanac/selectors/m/marker/)
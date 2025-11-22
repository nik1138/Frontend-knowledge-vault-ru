---
aliases: ["Псевдоэлементы before и after", "::before ::after", "Продвинутые техники псевдоэлементов"]
tags: [css, pseudo-elements, before, after, advanced]
---

# Псевдоэлементы ::before и ::after: продвинутые техники

## Введение

Псевдоэлементы `::before` и `::after` позволяют вставлять дополнительное содержимое в документ без изменения HTML-разметки. Эти псевдоэлементы создаются с помощью CSS и могут быть стилизованы как обычные элементы, что открывает широкие возможности для креативного дизайна и улучшения пользовательского интерфейса.

## Основы псевдоэлементов ::before и ::after

### Синтаксис и базовое использование

```css
.element::before {
  content: ""; /* Обязательное свойство для отображения псевдоэлемента */
  display: block; /* Тип отображения */
  /* Другие стили */
}

.element::after {
  content: "";
  display: inline-block;
  /* Другие стили */
}
```

### Обязательное свойство content

Свойство `content` обязательно для отображения псевдоэлементов:

```css
.quote::before {
  content: """; /* Текст перед элементом */
  font-size: 1.5em;
  color: #666;
}

.quote::after {
  content: """; /* Текст после элементом */
  font-size: 1.5em;
  color: #666;
}

.icon::before {
  content: "★"; /* Символ перед элементом */
  color: gold;
  margin-right: 5px;
}

.separator::after {
  content: " | "; /* Разделитель после элемента */
  color: #ccc;
}

.separator:last-child::after {
  content: ""; /* Убираем разделитель у последнего элемента */
}
```

## Продвинутые техники

### 1. Декоративные элементы

#### Стилизация заголовков с декоративными линиями

```css
.fancy-header {
  position: relative;
  text-align: center;
  margin: 40px 0;
}

.fancy-header::before,
.fancy-header::after {
  content: "";
  position: absolute;
  top: 50%;
  width: 30%;
  height: 1px;
  background-color: #007bff;
}

.fancy-header::before {
  left: 0;
  transform: translateY(-50%);
}

.fancy-header::after {
  right: 0;
  transform: translateY(-50%);
}

/* Альтернативный вариант с градиентом */
.gradient-header {
  position: relative;
  text-align: center;
  margin: 40px 0;
}

.gradient-header::before,
.gradient-header::after {
  content: "";
  position: absolute;
  top: 50%;
  width: 30%;
  height: 2px;
  background: linear-gradient(to right, transparent, #007bff, transparent);
}

.gradient-header::before {
  left: 0;
}

.gradient-header::after {
  right: 0;
}
```

#### Стилизация ссылок с анимированными подчеркиваниями

```css
.animated-link {
  position: relative;
  text-decoration: none;
  color: #007bff;
  padding-bottom: 5px;
}

.animated-link::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 2px;
  background-color: #007bff;
  transition: width 0.3s ease;
}

.animated-link:hover::after {
  width: 100%;
}

/* Альтернативный вариант с волнистой линией */
.wavy-link {
  position: relative;
  text-decoration: none;
  color: #28a745;
  padding-bottom: 5px;
}

.wavy-link::after {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: 
    linear-gradient(90deg, 
      transparent 0%, 
      transparent 33%, 
      #28a745 33%, 
      #28a745 66%, 
      transparent 66%, 
      transparent 100%);
  background-size: 6px 2px;
  background-repeat: repeat-x;
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.3s ease;
}

.wavy-link:hover::after {
  transform: scaleX(1);
}
```

### 2. Геометрические фигуры

#### Создание треугольников

```css
.triangle-up {
  position: relative;
  width: 50px;
  height: 50px;
}

.triangle-up::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 0;
  height: 0;
  border-left: 25px solid transparent;
  border-right: 25px solid transparent;
  border-bottom: 50px solid #007bff;
}

.triangle-down {
  position: relative;
  width: 50px;
  height: 50px;
}

.triangle-down::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 0;
  height: 0;
  border-left: 25px solid transparent;
  border-right: 25px solid transparent;
  border-top: 50px solid #dc3545;
}

.triangle-left {
  position: relative;
  width: 50px;
  height: 50px;
}

.triangle-left::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 0;
  height: 0;
  border-top: 25px solid transparent;
  border-bottom: 25px solid transparent;
  border-right: 50px solid #28a745;
}

.triangle-right {
  position: relative;
  width: 50px;
  height: 50px;
}

.triangle-right::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 0;
  height: 0;
  border-top: 25px solid transparent;
  border-bottom: 25px solid transparent;
  border-left: 50px solid #ffc107;
}
```

#### Создание значков и иконок

```css
.checkbox-icon {
  position: relative;
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 2px solid #007bff;
  border-radius: 3px;
}

.checkbox-icon.checked::before {
  content: "";
  position: absolute;
  top: 2px;
  left: 6px;
  width: 6px;
  height: 12px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
  background: transparent;
}

.star-rating {
  position: relative;
  display: inline-block;
  font-size: 0;
}

.star {
  position: relative;
  display: inline-block;
  width: 20px;
  height: 20px;
  margin-right: 2px;
}

.star::before {
  content: "☆";
  position: absolute;
  left: 0;
  top: 0;
  width: 20px;
  height: 20px;
  line-height: 20px;
  font-size: 20px;
  color: #ddd;
}

.star.filled::before {
  content: "★";
  color: gold;
}
```

### 3. Эффекты наведения

#### Карточки с подъемом при наведении

```css
.card {
  position: relative;
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  transition: transform 0.3s ease;
}

.card::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 8px;
  box-shadow: 0 8px 15px rgba(0,0,0,0.2);
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: -1;
}

.card:hover {
  transform: translateY(-5px);
}

.card:hover::before {
  opacity: 1;
}
```

#### Кнопки с эффектом "волны"

```css
.wave-button {
  position: relative;
  overflow: hidden;
  background: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}

.wave-button::before {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255,255,255,0.3);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

.wave-button:active::before {
  width: 300px;
  height: 300px;
}
```

### 4. Креативные эффекты

#### Текст с эффектом "галерея"

```css
.gallery-text {
  position: relative;
  font-size: 2em;
  font-weight: bold;
  color: transparent;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4, #feca57);
  background-clip: text;
  -webkit-background-clip: text;
  background-size: 300% 300%;
  animation: gradientShift 4s ease infinite;
}

.gallery-text::before,
.gallery-text::after {
  content: attr(data-text);
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4, #feca57);
  background-clip: text;
  -webkit-background-clip: text;
  background-size: 300% 300%;
  animation: gradientShift 4s ease infinite;
  opacity: 0.1;
}

.gallery-text::before {
  transform: translate(-2px, -2px);
  z-index: -1;
}

.gallery-text::after {
  transform: translate(2px, 2px);
  z-index: -2;
}

@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}
```

#### Кнопка с анимированным бордером

```css
.border-animation {
  position: relative;
  background: transparent;
  border: none;
  padding: 15px 30px;
  color: #007bff;
  font-size: 16px;
  overflow: hidden;
}

.border-animation::before {
  content: "";
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, transparent, #007bff, transparent);
  transition: 0.5s;
}

.border-animation:hover::before {
  left: 100%;
}

.border-animation::after {
  content: "";
  position: absolute;
  bottom: 0;
  right: -100%;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, transparent, #007bff, transparent);
  transition: 0.5s;
  transition-delay: 0.5s;
}

.border-animation:hover::after {
  right: 100%;
}
```

## Практические примеры

### 1. Календарь с выделением сегодняшней даты

```html
<div class="calendar-date">15</div>
```

```css
.calendar-date {
  position: relative;
  display: inline-block;
  width: 40px;
  height: 40px;
  line-height: 40px;
  text-align: center;
  border-radius: 50%;
  background-color: #e9ecef;
  cursor: pointer;
  transition: background-color 0.3s;
}

.calendar-date.today {
  background-color: #007bff;
  color: white;
}

.calendar-date.today::before {
  content: "";
  position: absolute;
  top: -5px;
  right: -5px;
  width: 12px;
  height: 12px;
  background-color: #dc3545;
  border-radius: 50%;
  box-shadow: 0 0 0 2px white;
}
```

### 2. Комментарии с пузырьками

```html
<div class="comment">
  <div class="comment-text">Текст комментария</div>
</div>
```

```css
.comment {
  position: relative;
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  max-width: 80%;
}

.comment::before {
  content: "";
  position: absolute;
  left: -10px;
  top: 15px;
  width: 0;
  height: 0;
  border-top: 10px solid transparent;
  border-bottom: 10px solid transparent;
  border-right: 10px solid #f8f9fa;
}
```

### 3. Прогресс-бар с анимацией

```html
<div class="progress-container">
  <div class="progress-bar" style="width: 75%;"></div>
</div>
```

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
  position: relative;
  transition: width 0.3s ease;
}

.progress-bar::after {
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
  animation: progressShine 2s infinite;
}

@keyframes progressShine {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}
```

## Расширенные техники

### 1. Множественные псевдоэлементы

```css
.multi-effect {
  position: relative;
  padding: 20px;
  background: white;
  border: 1px solid #ddd;
}

.multi-effect::before,
.multi-effect::after {
  content: "";
  position: absolute;
  width: 10px;
  height: 10px;
  background: #007bff;
}

.multi-effect::before {
  top: -5px;
  left: -5px;
  border-radius: 50% 0 50% 50%;
}

.multi-effect::after {
  bottom: -5px;
  right: -5px;
  border-radius: 50% 50% 0 50%;
}
```

### 2. Псевдоэлементы с CSS-переменными

```css
.dynamic-element {
  --color: #007bff;
  --size: 20px;
  position: relative;
  padding: 10px;
}

.dynamic-element::before {
  content: "";
  position: absolute;
  width: var(--size);
  height: var(--size);
  background-color: var(--color);
  border-radius: 50%;
}
```

## Заключение

Псевдоэлементы `::before` и `::after` предоставляют мощные возможности для создания сложных визуальных эффектов без изменения HTML-разметки. Их правильное использование позволяет создавать креативные и интерактивные интерфейсы, улучшая пользовательский опыт.

Для изучения других псевдоэлементов рекомендуется ознакомиться с [[Псевдоэлементы ::first-line и ::first-letter: стилизация текста]] и [[Другие псевдоэлементы (selection, placeholder, marker и др.)]].

## Ссылки

- [MDN: ::before](https://developer.mozilla.org/ru/docs/Web/CSS/::before)
- [MDN: ::after](https://developer.mozilla.org/ru/docs/Web/CSS/::after)
- [CSS-Tricks: Before and After](https://css-tricks.com/almanac/selectors/b/beforeafter/)
- [CSS-Tricks: The Content Property](https://css-tricks.com/almanac/properties/c/content/)
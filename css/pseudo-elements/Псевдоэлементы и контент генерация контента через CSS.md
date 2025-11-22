---
aliases: ["Генерация контента через CSS", "CSS Generated Content", "Псевдоэлементы и контент"]
tags: [css/pseudo-elements, css/content, generated-content, css-before-after]
---

# Псевдоэлементы и контент: генерация контента через CSS

## Введение

Псевдоэлементы `::before` и `::after` в CSS позволяют добавлять генерируемый контент к элементам без изменения HTML-разметки. Это мощный инструмент для создания декоративных элементов, иконок, подсказок и других визуальных компонентов исключительно с помощью CSS.

## Основы генерации контента

### Синтаксис ::before и ::after

Псевдоэлементы `::before` и `::after` создаются визуальные элементы, которые вставляются в DOM виртуально:

```css
.element::before {
  content: "Текст до элемента";
  display: inline;
}

.element::after {
  content: "Текст после элемента";
  display: inline;
}
```

```html
<!-- Визуально будет отображаться как: -->
<!-- "Текст до элемента" Элемент "Текст после элемента" -->
<div class="element">Элемент</div>
```

### Обязательное свойство content

Свойство `content` обязательно для отображения псевдоэлемента:

```css
/* Псевдоэлемент не будет отображаться без content */
.invalid-pseudo {
  background-color: red;
  /* ::before или ::after не появится без content */
}

/* Правильное использование */
.valid-pseudo::before {
  content: ""; /* Пустой контент */
  display: block;
  width: 20px;
  height: 20px;
  background-color: red;
}
```

## Типы генерируемого контента

### Текстовый контент

```css
.quote::before {
  content: """;
  font-size: 2rem;
  color: #666;
  font-weight: bold;
}

.quote::after {
  content: """;
  font-size: 2rem;
  color: #666;
  font-weight: bold;
}

/* Добавление текста с кавычками */
.author::before {
  content: "— ";
}

.author::after {
  content: " (автор)";
}
```

### Специальные символы и иконки

```css
/* Использование Unicode символов */
.checkmark::before {
  content: "✓";
  color: green;
  font-weight: bold;
  margin-right: 5px;
}

.cross::before {
  content: "✗";
  color: red;
  font-weight: bold;
  margin-right: 5px;
}

/* Стрелки и другие символы */
.arrow-right::after {
  content: " →";
}

.heart::before {
  content: "♥";
  color: red;
  margin-right: 5px;
}

/* Использование эмодзи */
.star-rating::before {
  content: "⭐⭐⭐⭐⭐";
  color: gold;
}
```

### Счетчики и нумерация

```css
/* Создание собственной нумерации */
.custom-counter {
  counter-increment: item;
  position: relative;
}

.custom-counter::before {
  content: counter(item) ". ";
  position: absolute;
  left: -30px;
  color: #007bff;
  font-weight: bold;
}

/* Нумерация с префиксом */
.chapter {
  counter-increment: chapter;
}

.chapter::before {
  content: "Глава " counter(chapter) ": ";
  font-weight: bold;
  color: #333;
}
```

## Продвинутые техники генерации контента

### Генерация контента на основе атрибутов

```html
<a href="https://example.com" class="external-link" title="Внешний ресурс">Ссылка</a>
```

```css
/* Добавление иконки внешней ссылки */
.external-link::after {
  content: " ↗";
  font-size: 0.8em;
  opacity: 0.7;
}

/* Отображение title атрибута */
[title]::after {
  content: " (" attr(title) ")";
  font-size: 0.8em;
  color: #666;
  font-style: italic;
}

/* Отображение URL для ссылок */
.link-url[href]::after {
  content: " (" attr(href) ")";
  font-size: 0.8em;
  color: #666;
  display: block;
}
```

### Генерация декоративных элементов

```css
/* Декоративная линия под заголовком */
.decorative-header {
  position: relative;
  margin-bottom: 2rem;
}

.decorative-header::after {
  content: "";
  position: absolute;
  bottom: -10px;
  left: 0;
  width: 50px;
  height: 3px;
  background: linear-gradient(90deg, #007bff, #0056b3);
  border-radius: 2px;
}

/* Декоративные углы */
.corner-decor::before,
.corner-decor::after {
  content: "";
  position: absolute;
  width: 20px;
  height: 20px;
  border: 2px solid #007bff;
}

.corner-decor::before {
  top: -10px;
  left: -10px;
  border-right: none;
  border-bottom: none;
}

.corner-decor::after {
  top: -10px;
  right: -10px;
  border-left: none;
  border-bottom: none;
}
```

## Практические примеры генерации контента

### Пример 1: Кастомные чекбоксы

```html
<label class="custom-checkbox">
  <input type="checkbox" class="checkbox-input">
  <span class="checkbox-label">Принять условия</span>
</label>
```

```css
.custom-checkbox {
  display: flex;
  align-items: center;
  cursor: pointer;
  position: relative;
  padding-left: 35px;
  margin-bottom: 12px;
}

.checkbox-input {
  opacity: 0;
  position: absolute;
  width: 0;
  height: 0;
}

.checkbox-input + .checkbox-label::before {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 24px;
  height: 24px;
  background-color: #fff;
  border: 2px solid #007bff;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.checkbox-input:checked + .checkbox-label::after {
  content: "✓";
  position: absolute;
  left: 6px;
  top: 2px;
  color: white;
  font-size: 14px;
  font-weight: bold;
}

.checkbox-input:checked + .checkbox-label::before {
  background-color: #007bff;
  border-color: #007bff;
}

.checkbox-input:focus + .checkbox-label::before {
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.25);
}
```

### Пример 2: Кавычки для цитат

```html
<blockquote class="styled-quote">
  <p>Жизнь — это то, что с тобой происходит, пока ты строишь планы.</p>
  <cite>Джон Леннон</cite>
</blockquote>
```

```css
.styled-quote {
  position: relative;
  padding: 2rem 2rem 2rem 4rem;
  margin: 2rem 0;
  background-color: #f8f9fa;
  border-left: 4px solid #007bff;
  font-style: italic;
  border-radius: 0 8px 8px 0;
}

.styled-quote::before {
  content: """;
  position: absolute;
  top: 10px;
  left: 10px;
  font-size: 4rem;
  color: rgba(0, 123, 255, 0.2);
  font-family: Georgia, serif;
  line-height: 1;
}

.styled-quote::after {
  content: "";
  position: absolute;
  bottom: 10px;
  right: 20px;
  font-size: 4rem;
  color: rgba(0, 123, 255, 0.2);
  font-family: Georgia, serif;
  line-height: 1;
  content: """;
}

.styled-quote cite {
  display: block;
  text-align: right;
  margin-top: 1rem;
  font-style: normal;
  font-weight: bold;
  color: #666;
}

.styled-quote cite::before {
  content: "— ";
}
```

### Пример 3: Тултипы с псевдоэлементами

```html
<span class="tooltip" data-tooltip="Это всплывающая подсказка">Наведи на меня</span>
```

```css
.tooltip {
  position: relative;
  display: inline-block;
  border-bottom: 1px dotted #007bff;
  cursor: help;
}

.tooltip::before,
.tooltip::after {
  position: absolute;
  left: 50%;
  opacity: 0;
  pointer-events: none;
  transition: all 0.3s ease;
}

.tooltip::before {
  content: attr(data-tooltip);
  bottom: 150%;
  transform: translateX(-50%) translateY(-10px);
  padding: 8px 12px;
  min-width: 150px;
  background-color: #333;
  color: white;
  text-align: center;
  border-radius: 4px;
  font-size: 14px;
  white-space: nowrap;
  z-index: 1000;
}

.tooltip::after {
  content: "";
  bottom: 140%;
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top-color: #333;
  z-index: 1001;
}

.tooltip:hover::before,
.tooltip:hover::after,
.tooltip:focus::before,
.tooltip:focus::after {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}
```

## Сложные примеры с генерацией контента

### Пример 4: Прогресс-бар с индикаторами

```html
<div class="progress-container" data-progress="75">
  <div class="progress-bar">
    <div class="progress-fill"></div>
  </div>
  <div class="progress-text">75%</div>
</div>
```

```css
.progress-container {
  position: relative;
  width: 100%;
  margin: 2rem 0;
}

.progress-bar {
  height: 20px;
  background-color: #e9ecef;
  border-radius: 10px;
  overflow: hidden;
  position: relative;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #007bff, #0056b3);
  width: 0;
  transition: width 1s ease-in-out;
  position: relative;
}

/* Индикаторы на шкале прогресса */
.progress-bar::before {
  content: "";
  position: absolute;
  top: -10px;
  left: 0;
  height: 5px;
  width: 100%;
  background: repeating-linear-gradient(
    to right,
    #ccc 0px,
    #ccc 2px,
    transparent 2px,
    transparent 20px
  );
}

/* Текстовые метки на шкале */
.progress-container::after {
  content: "0% 25% 50% 75% 100%";
  position: absolute;
  top: -30px;
  left: 0;
  width: 100%;
  font-size: 0.7rem;
  color: #666;
  display: flex;
  justify-content: space-between;
  pointer-events: none;
}
```

### Пример 5: Кастомные табы с генерацией контента

```html
<div class="tab-container">
  <div class="tab active" data-tab="1">Вкладка 1</div>
  <div class="tab" data-tab="2">Вкладка 2</div>
  <div class="tab" data-tab="3">Вкладка 3</div>
</div>
```

```css
.tab-container {
  display: flex;
  border-bottom: 2px solid #dee2e6;
  position: relative;
}

.tab {
  padding: 1rem 2rem;
  cursor: pointer;
  position: relative;
  transition: color 0.3s ease;
}

.tab::after {
  content: attr(data-tab);
  position: absolute;
  top: -10px;
  right: 5px;
  background-color: #007bff;
  color: white;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.7rem;
  opacity: 0;
  transform: scale(0);
  transition: all 0.3s ease;
}

.tab:hover::after {
  opacity: 1;
  transform: scale(1);
}

.tab.active {
  color: #007bff;
  font-weight: bold;
}

.tab.active::before {
  content: "";
  position: absolute;
  bottom: -2px;
  left: 0;
  width: 100%;
  height: 3px;
  background-color: #007bff;
}
```

## Современные возможности и ограничения

### Совместимость и ограничения

```css
/* Ограничения для ::first-letter и ::first-line */
.paragraph::first-letter {
  font-size: 2em;
  font-weight: bold;
  float: left;
  margin-right: 5px;
}

.paragraph::first-line {
  font-weight: bold;
  color: #007bff;
}

/* Эти псевдоэлементы имеют особенности использования */
```

### Современные подходы

```css
/* Использование CSS-переменных с генерируемым контентом */
.dynamic-content {
  --icon: "★";
  --color: gold;
}

.dynamic-content::before {
  content: var(--icon);
  color: var(--color);
  margin-right: 5px;
}

/* JavaScript может изменять переменные */
/* element.style.setProperty('--icon', '❤'); */
```

## Лучшие практики

### 1. Семантическая корректность

```css
/* Хорошо: декоративный контент */
.quote::before,
.quote::after {
  content: """;
  /* Декоративные кавычки для цитаты */
}

/* Избегайте: смысловой контент */
.bad-example::before {
  content: "Важно: "; /* Лучше добавить в HTML */
}
```

### 2. Доступность

```css
/* Обеспечиваем доступность генерируемого контента */
.sr-only::before {
  content: attr(data-label);
  /* Только для скринридеров */
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### 3. Производительность

```css
/* Избегаем избыточного использования псевдоэлементов */
.optimized-element {
  /* Используем ::before и ::after только когда необходимо */
  position: relative;
}

.optimized-element::before {
  /* Только если действительно нужен генерируемый контент */
  content: "";
}
```

## Заключение

Генерация контента через CSS-псевдоэлементы - мощный инструмент для создания визуально привлекательных и функциональных интерфейсов. Ключевые моменты:

1. Свойство `content` обязательно для отображения псевдоэлемента
2. Используйте `::before` и `::after` для декоративных элементов
3. Генерируемый контент не должен содержать важную смысловую информацию
4. Обеспечивайте доступность при использовании псевдоэлементов
5. Учитывайте производительность при сложных анимациях
6. Тестируйте на разных устройствах и браузерах

Правильное использование псевдоэлементов позволяет создавать чистую HTML-разметку и богатые визуальные эффекты.

## См. также

- [[Псевдоэлементы и стили: продвинутые эффекты]]
- [[Псевдоэлементы и доступность: ограничения и особенности]]
- [[CSS ::before и ::after: полное руководство]]
- [[Современные CSS-техники: продвинутые псевдоэлементы]]
- [[CSS-анимации и переходы: продвинутые техники]]
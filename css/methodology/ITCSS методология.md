---
aliases: [Методология ITCSS, ITCSS Architecture, Scalable CSS Architecture]
tags: [css, methodology, itcss, frontend]
---

# ITCSS (Inverted Triangle CSS) - Методология для масштабируемого CSS

ITCSS (Inverted Triangle CSS) - это методология организации CSS-кода, разработанная Стивом Шоу (Steve Schoger), которая помогает создавать масштабируемые, поддерживаемые и предсказуемые таблицы стилей. Методология основана на принципе "обратного треугольника", где более общие стили находятся в начале, а более специфичные - в конце.

## Общее понимание ITCSS

ITCSS решает проблему роста CSS-кода в больших проектах, где становится сложно управлять специфичностью, каскадом и переиспользованием стилей. Методология разделяет CSS на слои, каждый из которых имеет свою роль и уровень специфичности.

> [!info] Основная идея
> ITCSS структурирует CSS в виде перевернутого треугольника, где общие стили находятся вверху, а более специфичные - внизу, что упрощает управление специфичностью и предотвращает конфликты стилей.

## Слои ITCSS

### 1. Settings (Настройки)

Слой Settings содержит переменные CSS-препроцессоров (Sass, Less, Stylus), такие как цвета, размеры шрифтов, отступы, медиа-запросы и другие глобальные настройки.

```scss
// settings/_colors.scss
$primary-color: #007bff;
$secondary-color: #6c757d;
$success-color: #28a745;
$danger-color: #dc3545;

// settings/_spacing.scss
$spacer: 1rem;
$spacer-sm: $spacer * 0.5;
$spacer-lg: $spacer * 2;
```

### 2. Tools (Инструменты)

Слой Tools содержит функции, миксины и другие инструменты, которые помогают в создании CSS. Эти элементы не генерируют CSS напрямую, но используются в других слоях.

```scss
// tools/_mixins.scss
@mixin respond-to($breakpoint) {
  @if $breakpoint == mobile {
    @media (max-width: 768px) { @content; }
  }
  @else if $breakpoint == tablet {
    @media (min-width: 769px) and (max-width: 1024px) { @content; }
  }
}

@mixin font-size($size) {
  font-size: #{$size}rem;
  line-height: #{$size * 1.5}rem;
}
```

### 3. Generic (Общие)

Слой Generic содержит сбросы стилей, нормализацию и другие глобальные сбросы. Это первый слой, который действительно генерирует CSS-код.

```css
/* generic/_reset.scss */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* generic/_normalize.scss */
html {
  line-height: 1.15;
  -webkit-text-size-adjust: 100%;
}
```

### 4. Elements (Элементы)

Слой Elements содержит стили для голых HTML-элементов, таких как `h1`, `h2`, `a`, `p`, `ul`, `ol` и т.д. Это базовые стили, применяемые ко всем элементам по умолчанию.

```css
/* elements/_headings.scss */
h1 { font-size: 2.5rem; font-weight: bold; }
h2 { font-size: 2rem; font-weight: bold; }
h3 { font-size: 1.75rem; font-weight: bold; }

/* elements/_links.scss */
a {
  color: $primary-color;
  text-decoration: none;
  
  &:hover {
    color: darken($primary-color, 10%);
  }
}
```

### 5. Objects (Объекты)

Слой Objects содержит стили для абстрактных паттернов, которые не имеют визуального оформления. Это структурные классы, которые определяют макеты, такие как `.o-layout`, `.o-media`, `.o-grid`.

```css
/* objects/_layout.scss */
.o-layout {
  display: flex;
  flex-direction: column;
  
  &--row {
    flex-direction: row;
  }
}

/* objects/_grid.scss */
.o-grid {
  display: grid;
  gap: $spacer;
  
  &--2col { grid-template-columns: repeat(2, 1fr); }
  &--3col { grid-template-columns: repeat(3, 1fr); }
}
```

### 6. Components (Компоненты)

Слой Components содержит стили для конкретных компонентов пользовательского интерфейса, таких как кнопки, карточки, формы и т.д. Это первый слой с визуальным оформлением.

```css
/* components/_button.scss */
.c-btn {
  display: inline-block;
  padding: $spacer-sm $spacer;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &--primary {
    background-color: $primary-color;
    color: white;
  }
  
  &--secondary {
    background-color: $secondary-color;
    color: white;
  }
}

/* components/_card.scss */
.c-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  padding: $spacer;
  
  &__header {
    font-weight: bold;
    margin-bottom: $spacer-sm;
  }
  
  &__body {
    margin-bottom: $spacer-sm;
  }
  
  &__footer {
    border-top: 1px solid #eee;
    padding-top: $spacer-sm;
  }
}
```

### 7. Trumps (Трубы/Переопределения)

Последний слой Trumps содержит утилиты и переопределения, которые имеют высокую специфичность и используются для решения специфических проблем. Эти классы обычно имеют префикс `u-`.

```css
/* trumps/_utilities.scss */
.u-hidden { display: none !important; }
.u-show { display: block !important; }

.u-text-center { text-align: center !important; }
.u-text-left { text-align: left !important; }
.u-text-right { text-align: right !important; }

.u-margin-0 { margin: 0 !important; }
.u-padding-0 { padding: 0 !important; }

/* trumps/_spacing.scss */
.u-margin-top-sm { margin-top: $spacer-sm !important; }
.u-margin-bottom-lg { margin-bottom: $spacer-lg !important; }
```

## Преимущества ITCSS

### 1. Управление специфичностью

ITCSS помогает управлять специфичностью CSS-селекторов, так как каждый слой имеет определенный уровень специфичности. Более специфичные селекторы находятся в нижних слоях, что предотвращает необходимость в использовании `!important`.

### 2. Масштабируемость

Структура ITCSS позволяет легко добавлять новые компоненты и функции без нарушения существующего CSS-кода. Это делает методологию идеальной для больших проектов.

### 3. Поддерживаемость

Разделение CSS на логические слои облегчает понимание кода и упрощает внесение изменений. Разработчики могут легко находить нужные стили и понимать их назначение.

### 4. Предсказуемость

Благодаря четкой структуре, поведение CSS становится более предсказуемым. Это уменьшает количество ошибок и конфликтов стилей.

## Практические рекомендации

### 1. Именование классов

Рекомендуется использовать систему именования, которая соответствует слою:

- `.o-` для объектов (objects)
- `.c-` для компонентов (components)
- `.u-` для утилит (trumps)
- `.js-` для JavaScript-селекторов

```css
/* Пример именования */
.o-layout {}          /* Объект */
.c-button {}          /* Компонент */
.u-hidden {}          /* Утилита */
.js-modal-trigger {}  /* JavaScript-селектор */
```

### 2. Организация файлов

Каждый слой ITCSS должен быть представлен в отдельной директории с соответствующими файлами. Используйте файл `_main.scss` для импорта всех слоев в правильном порядке:

```scss
// main.scss
@import 'settings/colors';
@import 'settings/spacing';
@import 'tools/mixins';
@import 'tools/functions';
@import 'generic/reset';
@import 'generic/normalize';
@import 'elements/headings';
@import 'elements/links';
@import 'objects/layout';
@import 'objects/grid';
@import 'components/button';
@import 'components/card';
@import 'trumps/utilities';
@import 'trumps/spacing';
```

### 3. Согласование специфичности

При создании новых стилей всегда учитывайте уровень специфичности. Новые стили должны соответствовать слою, в котором они находятся. Избегайте использования селекторов с высокой специфичностью в верхних слоях.

### 4. Документирование компонентов

Каждый компонент должен быть должным образом задокументирован. Используйте комментарии или отдельные файлы документации для описания назначения компонента, его вариантов и использования.

```css
/**
 * Кнопка компонента
 * Используется для основных действий на странице
 * 
 * @modifier --primary - Основная кнопка действия
 * @modifier --secondary - Второстепенная кнопка
 * @modifier --disabled - Неактивное состояние
 */
.c-btn {
  // стили кнопки
}
```

## Интеграция с другими методологиями

ITCSS может использоваться в сочетании с другими методологиями CSS, такими как [[БЭМ (Блок-Элемент-Модификатор)]] или [[OOCSS (Object-Oriented CSS)]]. Это позволяет комбинировать преимущества различных подходов:

- Используйте [[БЭМ (Блок-Элемент-Модификатор)]] для именования классов внутри слоев ITCSS
- Применяйте принципы [[OOCSS (Object-Oriented CSS)]] в слоях Objects и Components
- Используйте [[SMACSS (Scalable and Modular Architecture for CSS)]] для организации стилей в слоях Components и Trumps

## Пример структуры проекта

```
styles/
├── settings/
│   ├── _colors.scss
│   ├── _spacing.scss
│   └── _breakpoints.scss
├── tools/
│   ├── _functions.scss
│   └── _mixins.scss
├── generic/
│   ├── _reset.scss
│   └── _typography.scss
├── elements/
│   ├── _headings.scss
│   └── _forms.scss
├── objects/
│   ├── _layout.scss
│   └── _grid.scss
├── components/
│   ├── _button.scss
│   ├── _card.scss
│   └── _navigation.scss
└── trumps/
    ├── _utilities.scss
    └── _helpers.scss
```

## Заключение

ITCSS - это мощная методология для организации CSS-кода в больших проектах. Она помогает управлять сложностью, улучшает масштабируемость и поддерживаемость кода. Правильное применение ITCSS требует дисциплины и понимания принципов методологии, но результатом является более предсказуемый и управляемый CSS-код.

> [!tip] 
> При внедрении ITCSS в существующий проект начинайте с малого: определите структуру слоев и постепенно переносите существующие стили в соответствующие слои, избегая резких изменений, которые могут повлиять на производство.

## Связанные темы

- [[CSS - Каскадные таблицы стилей]]
- [[БЭМ (Блок-Элемент-Модификатор)]]
- [[OOCSS (Object-Oriented CSS)]]
- [[SMACSS (Scalable and Modular Architecture for CSS)]]
- [[Архитектура CSS]]
- [[CSS-препроцессоры]]
- [[CSS Grid и Flexbox]]
- [[Адаптивный дизайн]]
- [[CSS-переменные]]
- [[CSS-модули]]
- [[Стилистические рекомендации]]
- [[Организация файлов CSS]]
- [[CSS-фреймворки]]
- [[CSS-производительность]]
- [[CSS-документация]]
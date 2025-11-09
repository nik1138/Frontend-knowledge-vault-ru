# Методологии CSS: БЭМ, OOCSS, SMACSS, Atomic CSS и ITCSS

## Введение

CSS-методологии представляют собой систематизированные подходы к организации и структурированию CSS-кода. Они помогают создавать более предсказуемый, масштабируемый и поддерживаемый код. В этой статье мы рассмотрим пять популярных методологий: БЭМ, OOCSS, SMACSS, Atomic CSS и ITCSS, их особенности, преимущества и недостатки.

## БЭМ (Блок-Элемент-Модификатор)

### Основные концепции

БЭМ (Block Element Modifier) — это методология, разработанная Яндексом, которая предлагает строгую именовательную конвенцию для CSS-классов. Основные принципы:

- **Блок** — независимый компонент интерфейса (например, `header`, `menu`, `button`)
- **Элемент** — составная часть блока, которая не может использоваться в отрыве от него (например, `menu__item`, `button__text`)
- **Модификатор** — сущность, определяющая внешний вид, состояние или поведение блока/элемента (например, `button--disabled`, `menu__item--active`)

### Именование по БЭМ

```css
/* Блок */
.menu { }

/* Элемент блока */
.menu__item { }

/* Модификатор блока */
.menu--horizontal { }

/* Модификатор элемента */
.menu__item--active { }

/* Модификатор со значением */
.button--size-large { }
```

### Практический пример

```html
<!-- HTML структура -->
<nav class="menu menu--horizontal">
  <ul class="menu__list">
    <li class="menu__item menu__item--active">
      <a class="menu__link">Главная</a>
    </li>
    <li class="menu__item">
      <a class="menu__link">О нас</a>
    </li>
    <li class="menu__item">
      <a class="menu__link menu__link--disabled">Контакты</a>
    </li>
  </ul>
</nav>
```

```css
/* CSS по БЭМ */
.menu {
  display: flex;
  background-color: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 4px;
}

.menu--horizontal {
  flex-direction: row;
}

.menu--vertical {
  flex-direction: column;
}

.menu__list {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
  flex: 1;
}

.menu--vertical .menu__list {
  flex-direction: column;
}

.menu__item {
  padding: 0.5rem 1rem;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.menu__item--active {
  background-color: #007bff;
  color: white;
}

.menu__item--disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.menu__link {
  text-decoration: none;
  color: inherit;
  display: block;
}

.menu__link--disabled {
  color: #6c757d;
  pointer-events: none;
}
```

### Преимущества БЭМ

- **Предсказуемость**: четкая структура именования
- **Масштабируемость**: легко добавлять новые компоненты
- **Читаемость**: понятная связь между элементами
- **Поддерживаемость**: легче рефакторить и обновлять

### Недостатки БЭМ

- **Длинные имена классов**: может быть громоздко
- **Жесткая структура**: требует строгого следования правилам
- **Кривая обучения**: требует времени для освоения

## OOCSS (Object-Oriented CSS)

### Основные принципы

OOCSS основывается на двух ключевых принципах:

1. **Структура и скин** — разделение структуры элемента от его визуального оформления
2. **Контейнер и содержимое** — независимость стилей содержимого от контейнера

### Практический пример

```css
/* Структурные классы (Skeleton) */
.media { display: flex; align-items: flex-start; }
.media-object { flex-shrink: 0; }
.media-body { flex: 1; }

/* Скины (Skin) */
.btn { 
  display: inline-block; 
  padding: 0.5rem 1rem; 
  border: none; 
  border-radius: 4px; 
  cursor: pointer; 
  text-decoration: none;
  transition: all 0.2s ease;
}

.btn--primary { 
  background-color: #007bff; 
  color: white; 
}

.btn--secondary { 
  background-color: #6c757d; 
  color: white; 
}

.btn--large { 
  padding: 0.75rem 1.5rem; 
  font-size: 1.25rem; 
}

.btn--small { 
  padding: 0.25rem 0.5rem; 
  font-size: 0.875rem; 
}

.btn--primary:hover { 
  background-color: #0056b3; 
}

.btn--secondary:hover { 
  background-color: #545b62; 
}
```

```html
<!-- Использование OOCSS -->
<a href="#" class="btn btn--primary btn--large">Primary Button</a>
<a href="#" class="btn btn--secondary btn--small">Secondary Button</a>

<div class="media">
  <img src="avatar.jpg" alt="Avatar" class="media-object" width="64" height="64">
  <div class="media-body">
    <h3>Заголовок</h3>
    <p>Описание</p>
  </div>
</div>
```

### Преимущества OOCSS

- **Повторное использование**: высокий уровень переиспользования кода
- **Гибкость**: легко комбинировать классы
- **Масштабируемость**: легко расширять функциональность

### Недостатки OOCSS

- **Требует дисциплины**: легко нарушить принципы
- **Сложность в обучении**: требует понимания объектно-ориентированного подхода
- **Меньше семантики**: классы могут быть менее описательными

## SMACSS (Scalable and Modular Architecture for CSS)

### Пять категорий правил

SMACSS делит CSS на пять категорий:

#### 1. Base (Базовые стили)
```css
/* Base: сброс стилей и базовые стили */
* { box-sizing: border-box; }
html { font-size: 16px; }
body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; }
h1, h2, h3, h4, h5, h6 { margin: 0; font-weight: 600; }
p { margin: 0 0 1rem 0; line-height: 1.6; }
a { color: #007bff; text-decoration: none; }
a:hover { text-decoration: underline; }
```

#### 2. Layout (Слой макета)
```css
/* Layout: структура страницы */
.l-header { height: 60px; background: #343a40; }
.l-sidebar { width: 250px; float: left; }
.l-main { margin-left: 250px; }
.l-footer { clear: both; }

/* Модульные лейауты */
.l-box { float: left; width: 50%; }
.l-box--third { float: left; width: 33.333%; }
.l-box--quarter { float: left; width: 25%; }
```

#### 3. Module (Модули)
```css
/* Module: переиспользуемые компоненты */
.nav { list-style: none; margin: 0; padding: 0; }
.nav__item { display: inline-block; }
.nav__link { display: block; padding: 1rem; }

/* Более сложный модуль */
.search-form { position: relative; }
.search-form__input { 
  width: 100%; 
  padding: 0.5rem 2rem 0.5rem 1rem; 
  border: 1px solid #ced4da; 
  border-radius: 4px;
}
.search-form__button { 
  position: absolute; 
  right: 4px; 
  top: 4px; 
  bottom: 4px; 
  padding: 0 1rem; 
  border: none; 
  background: #007bff; 
  color: white; 
  border-radius: 2px; 
}
```

#### 4. State (Состояния)
```css
/* State: временные изменения */
.is-hidden { display: none !important; }
.is-visible { display: block !important; }
.is-active { background: #007bff; color: white; }
.is-disabled { opacity: 0.5; pointer-events: none; }
.is-loading { position: relative; }
.is-loading::after { 
  content: ''; 
  position: absolute; 
  top: 50%; 
  left: 50%; 
  width: 20px; 
  height: 20px; 
  margin: -10px 0 0 -10px; 
  border: 2px solid #f3f3f3; 
  border-top: 2px solid #3498db; 
  border-radius: 50%; 
  animation: spin 1s linear infinite; 
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
```

#### 5. Theme (Темы)
```css
/* Theme: визуальное оформление */
.theme-light { 
  --bg-color: #ffffff;
  --text-color: #212529;
  --border-color: #dee2e6;
}

.theme-dark { 
  --bg-color: #212529;
  --text-color: #ffffff;
  --border-color: #495057;
}

.card { 
  background: var(--bg-color); 
  color: var(--text-color); 
  border: 1px solid var(--border-color); 
}
```

### Преимущества SMACSS

- **Хорошая организация**: логическое разделение кода
- **Масштабируемость**: легко масштабировать проекты
- **Гибкость**: можно адаптировать под нужды проекта

### Недостатки SMACSS

- **Сложность для начинающих**: требует времени для понимания
- **Более сложная структура**: чем другие методологии

## Atomic CSS (Functional CSS)

### Основные идеи

Atomic CSS использует маленькие, однозначные классы, каждый из которых отвечает за одно CSS-свойство. Это позволяет собирать компоненты из атомарных классов.

### Практический пример

```html
<!-- Пример Atomic CSS -->
<div class="flex flex-col items-center justify-center p-6 bg-white rounded-lg shadow-md">
  <h2 class="text-xl font-bold text-gray-900 mb-2">Заголовок</h2>
  <p class="text-gray-600 mb-4">Описание</p>
  <button class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition-colors">
    Кнопка
  </button>
</div>
```

### Преимущества Atomic CSS

- **Высокая гибкость**: можно быстро создавать уникальные компоненты
- **Отсутствие дубликатов**: каждый класс уникален
- **Быстрая разработка**: меньше писать CSS

### Недостатки Atomic CSS

- **Грязный HTML**: много классов в разметке
- **Крутая кривая обучения**: требует изменения подхода
- **Сложность поддержки**: трудно поддерживать согласованность

## ITCSS (Inverted Triangle CSS)

### Концепция перевернутого треугольника

ITCSS организует CSS в виде перевернутого треугольника по степени специфичности и влияния:

```
┌─────────────────┐  ← Наименее специфичные (настройки, инструменты)
│   Settings      │
├─────────────────┤
│    Tools        │
├─────────────────┤
│   Generic       │
├─────────────────┤
│   Elements      │
├─────────────────┤
│   Objects       │
├─────────────────┤
│  Components     │
├─────────────────┤
│   Utilities     │  ← Наиболее специфичные (утилиты)
└─────────────────┘
```

### Практический пример структуры

#### 1. Settings (настройки)
```css
/* settings.css - переменные и настройки */
:root {
  --color-primary: #007bff;
  --color-secondary: #6c757d;
  --color-success: #28a745;
  --color-danger: #dc3545;
  
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.25rem;
  
  --border-radius: 0.25rem;
  --box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

#### 2. Tools (инструменты)
```css
/* tools.css - миксины и функции */
@custom-media --tablet (min-width: 768px);
@custom-media --desktop (min-width: 1024px);

@define-mixin card-shadow {
  box-shadow: var(--box-shadow);
  border: 1px solid #e9ecef;
}
```

#### 3. Generic (общие стили)
```css
/* generic.css - reset и нормализация */
* { box-sizing: border-box; }
html { font-size: 16px; }
body { margin: 0; font-family: system-ui, sans-serif; }
```

#### 4. Elements (элементы)
```css
/* elements.css - стили для HTML элементов */
h1, h2, h3, h4, h5, h6 {
  margin: 0 0 0.5rem 0;
  font-weight: 600;
  line-height: 1.2;
}

h1 { font-size: var(--font-size-lg); }
h2 { font-size: 1.5rem; }
h3 { font-size: 1.25rem; }

p { margin: 0 0 1rem 0; line-height: 1.6; }
```

#### 5. Objects (объекты OOCSS)
```css
/* objects.css - OOCSS объекты */
.o-layout { display: flex; }
.o-layout__item { flex: 1; }

.o-media { display: flex; align-items: flex-start; }
.o-media__img { margin-right: 1rem; }
.o-media__body { flex: 1; }
```

#### 6. Components (компоненты)
```css
/* components.css - конкретные компоненты */
.c-button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: var(--border-radius);
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s ease;
}

.c-button--primary {
  background-color: var(--color-primary);
  color: white;
}

.c-button--secondary {
  background-color: var(--color-secondary);
  color: white;
}

.c-card {
  @mixin card-shadow;
  border-radius: var(--border-radius);
  overflow: hidden;
  background: white;
}
```

#### 7. Utilities (утилиты)
```css
/* utilities.css - вспомогательные классы */
.u-hidden { display: none !important; }
.u-screen-reader-only { position: absolute; left: -9999px; }
.u-text-center { text-align: center; }
.u-float-left { float: left; }
.u-clearfix::after { content: ""; display: table; clear: both; }
```

### Преимущества ITCSS

- **Контроль специфичности**: предотвращает проблемы с переопределением
- **Хорошая архитектура**: логическая организация кода
- **Масштабируемость**: легко масштабировать большие проекты

### Недостатки ITCSS

- **Сложность для начинающих**: требует понимания архитектуры
- **Более сложная структура**: чем другие методологии

## Сравнение методологий

### Таблица сравнения

| Методология | Преимущества | Недостатки | Лучше всего подходит |
|-------------|--------------|------------|---------------------|
| БЭМ | Ясная структура, легко читается | Длинные имена классов | Команды с разным уровнем опыта |
| OOCSS | Повторное использование, гибкость | Требует дисциплины | Библиотеки компонентов |
| SMACSS | Хорошая организация, масштабируемость | Более сложная структура | Крупные проекты |
| Atomic CSS | Высокая гибкость, отсутствие дубликатов | Грязный HTML, крутая кривая обучения | Быстрая разработка прототипов |
| ITCSS | Хорошая архитектура, контроль специфичности | Сложность для начинающих | Крупные, долгосрочные проекты |

### Практические примеры сравнения

```html
<!-- БЭМ -->
<div class="product-card product-card--featured">
  <img class="product-card__image" src="image.jpg" alt="Product">
  <div class="product-card__content">
    <h3 class="product-card__title">Product Name</h3>
    <button class="product-card__button product-card__button--primary">Buy Now</button>
  </div>
</div>

<!-- OOCSS -->
<div class="card card--featured">
  <img class="card__img" src="image.jpg" alt="Product">
  <div class="card__content">
    <h3 class="card__title">Product Name</h3>
    <button class="btn btn--primary">Buy Now</button>
  </div>
</div>

<!-- Atomic CSS -->
<div class="border rounded-lg shadow-md bg-white overflow-hidden">
  <img class="w-full" src="image.jpg" alt="Product">
  <div class="p-4">
    <h3 class="text-lg font-semibold mb-2">Product Name</h3>
    <button class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">Buy Now</button>
  </div>
</div>

<!-- SMACSS -->
<div class="c-product c-product--featured">
  <img class="c-product__image" src="image.jpg" alt="Product">
  <div class="c-product__content">
    <h3 class="c-product__title">Product Name</h3>
    <button class="c-button c-button--primary">Buy Now</button>
  </div>
</div>
```

## Современные подходы и гибридные решения

### CSS-in-JS
```javascript
// Пример с styled-components
import styled from 'styled-components';

const Button = styled.button`
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: ${props => props.primary ? '#007bff' : '#6c757d'};
  color: white;
  cursor: pointer;
  transition: background-color 0.2s ease;
  
  &:hover {
    background-color: ${props => props.primary ? '#0056b3' : '#545b62'};
  }
`;

// Использование
<Button primary>Primary Button</Button>
<Button>Secondary Button</Button>
```

### CSS Modules
```css
/* Button.module.css */
.button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.primary {
  composes: button;
  background-color: #007bff;
  color: white;
}

.secondary {
  composes: button;
  background-color: #6c757d;
  color: white;
}

.primary:hover {
  background-color: #0056b3;
}
```

```javascript
// Component.js
import styles from './Button.module.css';

function Component() {
  return (
    <div>
      <button className={styles.primary}>Primary</button>
      <button className={styles.secondary}>Secondary</button>
    </div>
  );
}
```

## Лучшие практики применения методологий

### 1. Выбор подходящей методологии

```css
/* Пример гибридного подхода */
/* Используем БЭМ для компонентов */
.product-card { /* стили компонента */ }
.product-card__image { /* стили элемента */ }
.product-card--featured { /* модификатор */ }

/* Используем OOCSS для общих структур */
.media { display: flex; align-items: flex-start; }
.media__img { flex-shrink: 0; margin-right: 1rem; }
.media__body { flex: 1; }

/* Используем utility classes для мелких изменений */
.text-center { text-align: center; }
.mb-4 { margin-bottom: 1rem; }
```

### 2. Документирование и стандарты

```css
/**
 * Карточка продукта
 * 
 * @class product-card
 * @classdesc Основной компонент карточки продукта
 * 
 * @state product-card--featured - выделенная карточка
 * @state product-card--loading - карточка в состоянии загрузки
 * 
 * @example
 * <div class="product-card product-card--featured">
 *   <img class="product-card__image" src="image.jpg" alt="Product">
 *   <div class="product-card__content">
 *     <h3 class="product-card__title">Product Name</h3>
 *     <button class="product-card__button">Buy Now</button>
 *   </div>
 * </div>
 */

.product-card {
  /* Стили компонента */
}
```

### 3. Автоматизация и инструменты

```json
// .stylelintrc.json
{
  "extends": ["stylelint-config-standard"],
  "rules": {
    "selector-class-pattern": [
      "^([a-z][a-z0-9]*)(-[a-z0-9]+)*(__[a-z0-9]+(-[a-z0-9]+)*)?(--[a-z0-9]+(-[a-z0-9]+)*)?$",
      {
        "message": "Expected class selector to be BEM formatted"
      }
    ]
  }
}
```

## Примеры из промышленной разработки

### Система компонентов на основе БЭМ

```css
/* components.css */
/* Базовые стили для всех компонентов */
.component {
  box-sizing: border-box;
  font-family: inherit;
}

/* Кнопка */
.button {
  display: inline-block;
  padding: 0.5rem 1rem;
  border: 1px solid transparent;
  border-radius: 4px;
  font-size: 1rem;
  font-weight: 500;
  text-align: center;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.2s ease;
  user-select: none;
  outline: none;
}

.button:focus {
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.button--primary {
  background-color: #007bff;
  color: white;
  border-color: #007bff;
}

.button--primary:hover {
  background-color: #0069d9;
  border-color: #0062cc;
}

.button--secondary {
  background-color: #6c757d;
  color: white;
  border-color: #6c757d;
}

.button--secondary:hover {
  background-color: #5a6268;
  border-color: #545b62;
}

.button--large {
  padding: 0.75rem 1.5rem;
  font-size: 1.25rem;
}

.button--small {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}

.button--disabled {
  opacity: 0.65;
  pointer-events: none;
}

/* Карточка */
.card {
  position: relative;
  display: flex;
  flex-direction: column;
  min-width: 0;
  word-wrap: break-word;
  background-color: #fff;
  background-clip: border-box;
  border: 1px solid rgba(0, 0, 0, 0.125);
  border-radius: 0.25rem;
}

.card__body {
  flex: 1 1 auto;
  padding: 1.25rem;
}

.card__header {
  padding: 0.75rem 1.25rem;
  margin-bottom: 0;
  background-color: rgba(0, 0, 0, 0.03);
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
}

.card__footer {
  padding: 0.75rem 1.25rem;
  background-color: rgba(0, 0, 0, 0.03);
  border-top: 1px solid rgba(0, 0, 0, 0.125);
}

.card--primary {
  border-color: #b8daff;
}

.card--primary .card__header {
  background-color: #cce5ff;
  border-color: #b8daff;
}
```

## Ссылки на связанные темы
- [[best-practices/best-practices]] - Лучшие практики CSS
- [[best-practices/css-architecture]] - Архитектура CSS
- [[html/semantics]] - Семантическая верстка
- [[js/dom/styling]] - Стилизация через JavaScript

#programming #css #methodology #bem #oocss #smacss #atomic-css #itcss #best-practices
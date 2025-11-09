# CSS (Cascading Style Sheets)

CSS (Cascading Style Sheets) — это язык таблиц стилей, используемый для описания представления документа, написанного на языке разметки. CSS является основной технологией для стилизации веб-страниц и веб-приложений.

## Основные характеристики

### Декларативная природа
CSS является декларативным языком, который описывает *как* элементы должны выглядеть, а не *как* этого достичь.

```css
/* Пример CSS правил */
body {
    font-family: Arial, sans-serif;
    background-color: #f0f0f0;
    margin: 0;
    padding: 20px;
}

h1 {
    color: #333;
    font-size: 2rem;
    text-align: center;
}
```

### Каскадирование и специфичность
CSS следует принципу каскадирования, где стили применяются в определенном порядке с учетом специфичности.

```css
/* Специфичность: 0,0,0,1 */
p {
    color: blue;
}

/* Специфичность: 0,0,1,0 */
.class {
    color: red;
}

/* Специфичность: 0,1,0,0 */
#id {
    color: green;
}

/* Специфичность: 1,0,0,0 */
!important {
    color: black !important;
}
```

### Селекторы
CSS поддерживает различные типы селекторов для выбора элементов:

```css
/* Селекторы элементов */
h1 { color: blue; }

/* Селекторы классов */
.header { font-size: 2rem; }

/* Селекторы ID */
#main { width: 100%; }

/* Псевдоклассы */
a:hover { color: red; }
li:nth-child(odd) { background-color: #f0f0f0; }

/* Псевдоэлементы */
p::first-line { font-weight: bold; }
```

## Основные концепции

### Блочная модель
Каждый элемент в CSS имеет блочную модель, состоящую из контента, padding, border и margin.

```css
.box {
    width: 200px;           /* Ширина контента */
    height: 100px;          /* Высота контента */
    padding: 20px;          /* Внутренний отступ */
    border: 5px solid black; /* Граница */
    margin: 10px;           /* Внешний отступ */
    /* Общая ширина: 200 + 20*2 + 5*2 + 10*2 = 270px */
}
```

### Flexbox
Flexbox — это одномерная система компоновки для распределения пространства между элементами.

```css
.flex-container {
    display: flex;
    justify-content: space-between; /* Распределение по главной оси */
    align-items: center;            /* Выравнивание по поперечной оси */
}

.flex-item {
    flex: 1; /* Элементы будут расти равномерно */
}
```

### Grid
CSS Grid — это двумерная система компоновки для создания сложных макетов.

```css
.grid-container {
    display: grid;
    grid-template-columns: 1fr 2fr 1fr; /* Три колонки */
    grid-template-rows: auto;           /* Автоматическая высота строк */
    gap: 20px;                          /* Промежутки между элементами */
}

.grid-item {
    grid-column: span 2; /* Элемент занимает 2 колонки */
}
```

## Преимущества CSS

1. **Разделение содержимого и представления** — HTML для структуры, CSS для стиля
2. **Гибкость** — Мощные возможности для создания различных визуальных эффектов
3. **Адаптивность** — Поддержка адаптивного дизайна через медиазапросы
4. **Производительность** — Эффективная отрисовка стилей браузером
5. **Поддержка сообщества** — Богатая экосистема методологий и фреймворков

## Связанные концепции

- [[Декларативное программирование]] - CSS является декларативным языком стилей
- [[Чистый код]] - Методологии CSS (БЭМ, OOCSS, SMACSS) способствуют написанию чистого кода
- [[KISS (Keep It Simple, Stupid)]] - CSS поощряет простоту в стилизации элементов
- [[DRY (Don't Repeat Yourself)]] - CSS позволяет избегать дублирования стилей через классы и переменные

## Методологии CSS

### БЭМ (Блок, Элемент, Модификатор)
```css
/* Блок */
.button { }

/* Элемент */
.button__icon { }

/* Модификатор */
.button--primary { }
```

### OOCSS (Объектно-ориентированный CSS)
```css
/* Структура и оформление разделены */
.media { overflow: hidden; }
.media-object { float: left; }
.media-body { margin-left: 20px; }
```

## В других технологиях

### В [[html]]
CSS применяется к HTML-элементам для стилизации:

```html
<!-- HTML -->
<div class="card">
    <h2 class="card-title">Заголовок</h2>
    <p class="card-text">Текст карточки</p>
</div>
```

```css
/* CSS */
.card {
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    padding: 20px;
}

.card-title {
    color: #333;
    margin-bottom: 10px;
}

.card-text {
    color: #666;
    line-height: 1.6;
}
```

### В [[js]]
JavaScript может динамически изменять CSS-стили:

```javascript
// Изменение стилей через JavaScript
const element = document.querySelector('.my-element');
element.style.backgroundColor = 'red';
element.classList.add('active');
```

### В [[react]]
React может использовать CSS через различные подходы:

```jsx
// Inline стили
const styles = {
    container: {
        backgroundColor: 'blue',
        padding: '20px'
    }
};

function MyComponent() {
    return <div style={styles.container}>Контент</div>;
}

// CSS Modules
import styles from './MyComponent.module.css';

function MyComponent() {
    return <div className={styles.container}>Контент</div>;
}
```

## Теги
#css #web-development #styling #frontend #design
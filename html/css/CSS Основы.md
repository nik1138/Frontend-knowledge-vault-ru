# CSS Основы

CSS (Cascading Style Sheets) — это язык таблиц стилей, который используется для описания внешнего вида документов, написанных на HTML. Основы CSS включают в себя базовые концепции, синтаксис и возможности стилизации веб-страниц.

## Структура CSS

CSS состоит из **правил**, которые в свою очередь состоят из **селекторов** и **объявлений**:

```css
селектор {
    свойство: значение;
    другое-свойство: другое-значение;
}
```

### Пример простого CSS правила:

```css
h1 {
    color: blue;
    font-size: 24px;
    text-align: center;
}
```

В этом примере:
- `h1` — **селектор** (определяет, к какому элементу применяются стили)
- `color`, `font-size`, `text-align` — **свойства** (характеристики, которые мы хотим изменить)
- `blue`, `24px`, `center` — **значения** (конкретные значения свойств)

## Типы селекторов

### 1. Селекторы элементов

Применяются к конкретным HTML элементам:

```css
p {
    color: #333;
    font-size: 16px;
}

h1 {
    font-size: 2em;
    margin-bottom: 1em;
}

div {
    padding: 20px;
    border: 1px solid #ccc;
}
```

### 2. Селекторы классов

Применяются к элементам с определенным классом (указывается через точку):

```css
/* В HTML: <p class="highlight">Текст</p> */
.highlight {
    background-color: yellow;
    font-weight: bold;
}

/* В HTML: <div class="container">Содержимое</div> */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}
```

### 3. Селекторы ID

Применяются к элементу с конкретным ID (указывается через решетку):

```css
/* В HTML: <div id="main-header">Содержимое</div> */
#main-header {
    background-color: #333;
    color: white;
    padding: 20px;
}

#navigation {
    display: flex;
    justify-content: space-around;
}
```

### 4. Селекторы атрибутов

Применяются к элементам с определенными атрибутами:

```css
/* Все input элементы с type="email" */
input[type="email"] {
    border: 2px solid #007acc;
    padding: 8px;
}

/* Все ссылки, которые начинаются с "https://" */
a[href^="https://"] {
    color: #28a745;
}

/* Все input элементы с атрибутом placeholder */
input[placeholder] {
    font-style: italic;
}
```

## Каскад и наследование

### Каскад

Каскад определяет, какие стили применяются, когда есть конфликты:

```css
/* Эти стили будут переопределены */
p {
    color: red;
}

/* Эти стили будут применены */
p {
    color: blue;
}

/* Встроенные стили имеют более высокий приоритет */
/* В HTML: <p style="color: green;">Текст</p> - будет зеленым */
```

### Наследование

Некоторые свойства наследуются дочерними элементами:

```css
/* Шрифт и цвет текста будут унаследованы */
body {
    font-family: Arial, sans-serif;
    color: #333;
}

/* Свойства, которые обычно наследуются: */
/* color, font-family, font-size, line-height, text-align и др. */

/* Свойства, которые не наследуются: */
/* margin, padding, border, width, height и др. */
```

## Блочная модель CSS

Все элементы HTML рассматриваются как прямоугольные блоки. Блочная модель включает:

- **Content** — само содержимое элемента
- **Padding** — внутренние отступы
- **Border** — граница
- **Margin** — внешние отступы

```css
.box {
    width: 200px;        /* Ширина содержимого */
    height: 100px;       /* Высота содержимого */
    padding: 20px;       /* Внутренние отступы */
    border: 5px solid #333; /* Граница */
    margin: 10px;        /* Внешние отступы */
}

/* Общая ширина элемента: 200 + 20 + 20 + 5 + 5 + 10 + 10 = 270px */
/* Общая высота элемента: 100 + 20 + 20 + 5 + 5 + 10 + 10 = 170px */
```

### `box-sizing`

Свойство `box-sizing` изменяет расчет размеров элемента:

```css
/* border-box: ширина и высота включают padding и border */
.element {
    box-sizing: border-box;
    width: 200px;
    padding: 20px;
    border: 5px solid #333;
    /* Фактическая ширина содержимого: 200 - 40 - 10 = 150px */
}

/* content-box: ширина и высота только для содержимого (по умолчанию) */
.other-element {
    box-sizing: content-box;
    width: 200px;
    padding: 20px;
    border: 5px solid #333;
    /* Общая ширина: 200 + 40 + 10 = 250px */
}
```

## Типы отображения (display)

### `block` — блочные элементы

```css
.block-element {
    display: block;
    width: 100%; /* Занимают всю доступную ширину */
    margin: 10px 0; /* Вертикальные отступы работают */
    padding: 10px;
}
```

Примеры блочных элементов: `div`, `p`, `h1-h6`, `section`, `article`, `header`, `footer`

### `inline` — строчные элементы

```css
.inline-element {
    display: inline;
    /* width и height игнорируются */
    margin: 0; /* Только горизонтальные margin работают */
    padding: 10px; /* Все отступы работают */
}
```

Примеры строчных элементов: `span`, `a`, `strong`, `em`, `img`

### `inline-block` — строчно-блочные элементы

```css
.inline-block-element {
    display: inline-block;
    width: 150px; /* Работает ширина */
    height: 50px; /* Работает высота */
    margin: 5px; /* Все отступы работают */
}
```

### `none` — скрытие элемента

```css
.hidden-element {
    display: none; /* Элемент полностью скрывается */
    /* Он не занимает места в документе */
}

/* visibility: hidden; скрывает элемент, но он занимает место */
.invisible-element {
    visibility: hidden;
}
```

## Цвета в CSS

### Именованные цвета

```css
.named-colors {
    color: red;
    background-color: blue;
    border-color: green;
}
```

### HEX значения

```css
.hex-colors {
    color: #ff0000;      /* Красный */
    background-color: #00ff00; /* Зеленый */
    border-color: #0000ff; /* Синий */
    
    /* Сокращенный формат для одинаковых пар: */
    shorthand: #f0f; /* тоже самое что #ff00ff */
}
```

### RGB и RGBA

```css
.rgb-colors {
    color: rgb(255, 0, 0);        /* Красный */
    background-color: rgb(0, 255, 0); /* Зеленый */
    border-color: rgba(0, 0, 255, 0.5); /* Синий с 50% прозрачностью */
}
```

### HSL и HSLA

```css
.hsl-colors {
    color: hsl(0, 100%, 50%);      /* Красный */
    background-color: hsl(120, 100%, 50%); /* Зеленый */
    border-color: hsla(240, 100%, 50%, 0.5); /* Синий с 50% прозрачностью */
}
```

## Размеры и единицы измерения

### Относительные единицы

```css
.relative-units {
    font-size: 16px;     /* Базовый размер */
    
    margin: 1em;         /* 1em = 16px (относительно родительского font-size) */
    padding: 1.5em;      /* 1.5em = 24px */
    
    width: 50%;          /* 50% от родительского элемента */
    
    height: 2rem;        /* 2rem = 32px (относительно корневого font-size) */
    
    line-height: 1.5;    /* 1.5 * font-size */
    
    margin-top: 2ex;     /* 2ex = приблизительно 2 высоты 'x' */
    
    width: 10ch;         /* 10ch = ширина 10 символов '0' */
    
    viewport-width: 50vw;  /* 50% ширины окна просмотра */
    viewport-height: 30vh; /* 30% высоты окна просмотра */
    
    vmin: 5vmin;         /* 5% от меньшего из vw или vh */
    vmax: 8vmax;         /* 8% от большего из vw или vh */
}
```

### Абсолютные единицы

```css
.absolute-units {
    font-size: 16px;     /* Пиксели (наиболее часто используемая) */
    width: 100px;        /* Пиксели */
    
    margin: 1cm;         /* Сантиметры */
    padding: 10mm;       /* Миллиметры */
    border-width: 1in;   /* Дюймы */
    height: 72pt;        /* Пункты */
    letter-spacing: 1pc; /* Пики */
}
```

## Шрифты и текст

### Семейства шрифтов

```css
.font-families {
    /* Специфические шрифты */
    font-family: "Times New Roman", Times, serif;
    
    /* Общие семейства */
    generic-family: Arial, sans-serif; /* Без засечек */
    serif-family: Georgia, serif;      /* С засечками */
    monospace-family: "Courier New", monospace; /* Моноширинный */
    cursive-family: "Comic Sans MS", cursive; /* Курсивный */
    fantasy-family: "Impact", fantasy; /* Декоративный */
}
```

### Свойства шрифта

```css
.font-properties {
    font-size: 18px;           /* Размер шрифта */
    font-weight: bold;         /* Жирность: normal, bold, 100-900 */
    font-style: italic;        /* Начертание: normal, italic, oblique */
    font-variant: small-caps;  /* Вариант: normal, small-caps */
    font-stretch: condensed;   /* Растяжение: normal, condensed, expanded */
    
    /* Сокращенная запись font */
    font: italic small-caps bold 18px/1.5 Arial, sans-serif;
    /* стиль вариант жирность размер/высота-линии семейство */
}
```

### Свойства текста

```css
.text-properties {
    color: #333;                    /* Цвет текста */
    text-align: center;             /* Выравнивание: left, right, center, justify */
    text-decoration: underline;     /* Оформление: none, underline, overline, line-through */
    text-transform: uppercase;      /* Преобразование: none, uppercase, lowercase, capitalize */
    text-indent: 20px;              /* Отступ первой строки */
    letter-spacing: 2px;            /* Расстояние между буквами */
    word-spacing: 5px;              /* Расстояние между словами */
    line-height: 1.6;               /* Высота строки */
    white-space: nowrap;            /* Обработка пробелов: normal, nowrap, pre, pre-line, pre-wrap */
    word-wrap: break-word;          /* Перенос слов: normal, break-word */
    text-shadow: 2px 2px 4px rgba(0,0,0,0.5); /* Тень текста: x y размытие цвет */
}
```

## Позиционирование

### Static (по умолчанию)

```css
.static {
    position: static; /* По умолчанию, элемент находится в обычном потоке */
    /* top, right, bottom, left игнорируются */
}
```

### Relative

```css
.relative {
    position: relative; /* Относительное позиционирование */
    top: 10px;          /* Сдвиг вниз от исходного положения */
    left: 20px;         /* Сдвиг вправо от исходного положения */
    /* Элемент остается в обычном потоке, но смещен */
}
```

### Absolute

```css
.absolute {
    position: absolute; /* Абсолютное позиционирование */
    top: 0;             /* Отступ от верхнего края ближайшего позиционированного родителя */
    right: 0;           /* Отступ от правого края */
    width: 200px;
    height: 100px;
    /* Элемент удаляется из обычного потока */
}
```

### Fixed

```css
.fixed {
    position: fixed;    /* Фиксированное позиционирование */
    top: 0;             /* Отступ от верха окна просмотра */
    right: 0;           /* Отступ от правого края окна просмотра */
    /* Элемент остается на месте при прокрутке */
}
```

### Sticky

```css
.sticky {
    position: sticky;   /* Липкое позиционирование */
    top: 10px;          /* Срабатывает при прокрутке до 10px от верха */
    /* Ведет себя как relative до определенного порога, затем как fixed */
}
```

## Практические примеры

### Простая стилизация формы

```css
.form-container {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ddd;
    border-radius: 8px;
    background-color: #f9f9f9;
}

.form-group {
    margin-bottom: 15px;
}

.form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
    color: #333;
}

.form-group input,
.form-group textarea,
.form-group select {
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
    box-sizing: border-box;
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
    outline: none;
    border-color: #007acc;
    box-shadow: 0 0 5px rgba(0, 122, 204, 0.3);
}

.form-group button {
    background-color: #007acc;
    color: white;
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
}

.form-group button:hover {
    background-color: #005a9e;
}
```

### Навигационное меню

```css
.navbar {
    background-color: #333;
    padding: 15px 0;
}

.nav-list {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
}

.nav-item {
    margin: 0 10px;
}

.nav-link {
    color: white;
    text-decoration: none;
    padding: 10px 15px;
    display: block;
    border-radius: 4px;
    transition: background-color 0.3s;
}

.nav-link:hover {
    background-color: #555;
}

.nav-link.active {
    background-color: #007acc;
}
```

### Карточка продукта

```css
.product-card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 15px;
    margin: 10px;
    background-color: white;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    transition: transform 0.3s, box-shadow 0.3s;
}

.product-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.product-image {
    width: 100%;
    height: 200px;
    object-fit: cover;
    border-radius: 4px;
    margin-bottom: 10px;
}

.product-title {
    font-size: 1.2em;
    margin: 0 0 10px 0;
    color: #333;
}

.product-price {
    font-size: 1.3em;
    font-weight: bold;
    color: #007acc;
    margin-bottom: 10px;
}

.product-description {
    color: #666;
    margin-bottom: 15px;
    line-height: 1.5;
}

.add-to-cart {
    background-color: #28a745;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 4px;
    cursor: pointer;
    width: 100%;
    font-size: 16px;
}

.add-to-cart:hover {
    background-color: #218838;
}
```

## Современные возможности CSS

### Переменные CSS

```css
:root {
    --primary-color: #007acc;
    --secondary-color: #6c757d;
    --success-color: #28a745;
    --danger-color: #dc3545;
    --font-size-base: 16px;
    --border-radius: 4px;
    --box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.button {
    background-color: var(--primary-color);
    color: white;
    padding: 8px 16px;
    border: none;
    border-radius: var(--border-radius);
    cursor: pointer;
}

.button:hover {
    background-color: #005a9e;
}
```

### Псевдоклассы и псевдоэлементы

```css
/* Псевдоклассы */
a:link { color: blue; }          /* Непосещенная ссылка */
a:visited { color: purple; }     /* Посещенная ссылка */
a:hover { color: red; }          /* При наведении */
a:active { color: orange; }      /* При клике */

input:focus { border-color: #007acc; } /* При фокусе */

/* Псевдоэлементы */
p:first-line { font-weight: bold; }     /* Первая строка параграфа */
p:first-letter { font-size: 2em; }      /* Первая буква параграфа */

.element::before { content: "> "; }      /* Содержимое перед элементом */
.element::after { content: " <"; }       /* Содержимое после элемента */

.list-item:nth-child(odd) { background-color: #f0f0f0; } /* Нечетные элементы */
.list-item:nth-child(even) { background-color: white; } /* Четные элементы */
```

## Проверка и отладка CSS

### Инструменты разработчика

1. **Inspect Element** — для просмотра и изменения стилей
2. **Computed Styles** — для просмотра итоговых стилей элемента
3. **Box Model** — для визуализации блочной модели
4. **Selectors** — для проверки соответствия селекторов элементам

### Популярные ошибки и как их избежать

```css
/* 1. Не забывайте про ; после каждого объявления */
.property { color: red } /* НЕТ ТОЧКИ С ЗАПЯТОЙ */

/* 2. Проверяйте синтаксис */
.invalid { color:: red; } /* ДВА ДВОЕТОЧИЯ */

/* 3. Убедитесь, что селекторы существуют в HTML */
.nonexistent-class { color: blue; } /* Если класса нет в HTML, стиль не применится */

/* 4. Помните о приоритете */
.low-priority { color: red; }
.high-priority { color: blue; } /* Если оба селектора применяются к одному элементу,
                                   последний в коде будет иметь приоритет */
```

## Лучшие практики

### 1. Организация кода

```css
/* Хорошая организация: группы связанных стилей */
/* 1. Сброс/нормализация */
* { box-sizing: border-box; }

/* 2. Переменные */
:root { --primary-color: #007acc; }

/* 3. Базовые стили */
body { font-family: Arial, sans-serif; }

/* 4. Компоненты */
.button { padding: 8px 16px; }

/* 5. Утилиты */
.text-center { text-align: center; }
```

### 2. Читаемость и поддержка

```css
/* Используйте понятные имена классов */
.user-profile-card { } /* Хорошо */
.upc { } /* Плохо */

/* Комментируйте сложные стили */
/* Временное решение для IE11 */
@media screen and (-ms-high-contrast: active), (-ms-high-contrast: none) {
    .flex-fix { display: block; }
}
```

## Заключение

Основы CSS включают в себя понимание селекторов, свойств, значений и принципов каскада и наследования. Знание этих основ позволяет создавать правильно структурированные и поддерживаемые таблицы стилей. CSS является мощным инструментом для визуального оформления веб-страниц, и его правильное использование критически важно для создания качественных пользовательских интерфейсов.

## Следующие темы
- [[CSS/Фреймворки]]
- [[CSS/Адаптивный дизайн]]
- [[CSS/Анимации]]
- [[CSS/Продвинутые возможности]]

## Теги
#css #basics #web-development #frontend #styling #selectors #properties #cascading #inheritance #box-model #display #positioning #colors #fonts #text #layout
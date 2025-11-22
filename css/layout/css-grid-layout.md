---
aliases: [CSS Grid, Grid Layout, Сетка CSS]
tags: [css, grid, layout, frontend]
---

# CSS Grid Layout: Полное руководство

CSS Grid Layout — это мощная система макета, которая позволяет создавать сложные двумерные макеты с легкостью. В отличие от Flexbox, который работает в одном измерении, Grid оперирует как по горизонтали, так и по вертикали, что делает его идеальным выбором для создания сеточных структур.

## Содержание

- [[#Основы CSS Grid]]
- [[#Свойства контейнера Grid]]
- [[#Свойства элементов Grid]]
- [[#Функции и значения Grid]]
- [[#Практические примеры]]
- [[#Лучшие практики]]
- [[#Распространенные ошибки]]

## Основы CSS Grid

CSS Grid — это двумерная система макета, которая позволяет разработчикам создавать сложные сетки с минимальным количеством кода. Она идеально подходит для создания макетов страниц, где важны как строки, так и столбцы.

Чтобы создать Grid-контейнер, необходимо установить свойство `display` в значение `grid` или `inline-grid`:

```css
.grid-container {
  display: grid;
  /* или */
  display: inline-grid;
}
```

Как только контейнер становится Grid-контейнером, все его прямые потомки становятся Grid-элементами.

## Свойства контейнера Grid

### display

Свойство `display` определяет, что элемент является Grid-контейнером:

```css
.container {
  display: grid; /* Блочный Grid-контейнер */
  /* или */
  display: inline-grid; /* Встроенный Grid-контейнер */
}
```

### grid-template-columns и grid-template-rows

Эти свойства определяют размеры колонок и строк сетки:

```css
.container {
  grid-template-columns: 100px 200px 100px;
  grid-template-rows: 50px 100px;
}
```

В этом примере мы создаем сетку с 3 колонками (100px, 200px, 100px) и 2 строками (50px, 100px).

### grid-template-areas

Позволяет задать именованные области сетки:

```css
.container {
  grid-template-areas: 
    "header header header"
    "sidebar main main"
    "footer footer footer";
}
```

Затем элементы могут быть привязаны к этим областям с помощью `grid-area`.

### grid-column-gap и grid-row-gap

Определяют промежутки между колонками и строками:

```css
.container {
  grid-column-gap: 20px;
  grid-row-gap: 10px;
}
```

Или сокращенная версия:

```css
.container {
  gap: 10px 20px; /* row gap, column gap */
  /* или */
  gap: 10px; /* одинаковый промежуток для строк и колонок */
}
```

### grid-auto-columns и grid-auto-rows

Определяют размеры неявно созданных колонок и строк:

```css
.container {
  grid-auto-columns: 100px;
  grid-auto-rows: 50px;
}
```

### grid-auto-flow

Определяет, как Grid автоматически размещает элементы:

```css
.container {
  grid-auto-flow: row; /* по умолчанию */
  /* или */
  grid-auto-flow: column;
  /* или */
  grid-auto-flow: dense; /* уплотнение */
}
```

## Свойства элементов Grid

### grid-column-start, grid-column-end, grid-row-start, grid-row-end

Эти свойства определяют, где начинается и заканчивается элемент в сетке:

```css
.item {
  grid-column-start: 2;
  grid-column-end: 4;
  grid-row-start: 1;
  grid-row-end: 3;
}
```

Или сокращенная версия:

```css
.item {
  grid-column: 2 / 4;
  grid-row: 1 / 3;
}
```

### grid-column и grid-row

Сокращенные свойства для определения положения элемента:

```css
.item {
  grid-column: 1 / 3; /* от колонки 1 до колонки 3 */
  grid-row: 1 / 3;    /* от строки 1 до строки 3 */
}
```

### grid-area

Свойство, которое объединяет `grid-row-start`, `grid-column-start`, `grid-row-end`, и `grid-column-end`:

```css
.item {
  grid-area: 1 / 2 / 3 / 4; /* row-start / column-start / row-end / column-end */
}
```

Также может использоваться с именованными областями:

```css
.item {
  grid-area: header; /* привязка к области "header" */
}
```

### justify-self

Выравнивание элемента в пределах его ячейки по горизонтали:

```css
.item {
  justify-self: start | end | center | stretch;
}
```

### align-self

Выравнивание элемента в пределах его ячейки по вертикали:

```css
.item {
  align-self: start | end | center | stretch;
}
```

### place-self

Сокращенное свойство для `justify-self` и `align-self`:

```css
.item {
  place-self: center; /* и justify и align */
  /* или */
  place-self: center stretch; /* justify align */
}
```

## Функции и значения Grid

### fr (fractional unit)

Единица дроби, представляющая долю доступного пространства:

```css
.container {
  grid-template-columns: 1fr 2fr 1fr; /* первая и последняя колонки по 1 части, средняя - 2 части */
}
```

### repeat()

Функция для повторения шаблона:

```css
.container {
  grid-template-columns: repeat(3, 1fr); /* эквивалентно 1fr 1fr 1fr */
  grid-template-columns: repeat(2, 100px 1fr); /* эквивалентно 100px 1fr 100px 1fr */
}
```

### minmax()

Определяет диапазон размеров:

```css
.container {
  grid-template-columns: repeat(3, minmax(100px, 1fr)); /* минимально 100px, максимально 1fr */
}
```

### auto-fit и auto-fill

Используются с `repeat()` для автоматического создания колонок:

```css
.container {
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  /* автоматически подстраивается под доступное пространство */
}
```

### fit-content()

Ограничивает размер по содержимому:

```css
.container {
  grid-template-columns: fit-content(200px) 1fr;
  /* первая колонка будет не больше 200px в зависимости от содержимого */
}
```

## Практические примеры

### Простой макет страницы

```css
.page {
  display: grid;
  grid-template-areas: 
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-rows: 80px 1fr 40px;
  grid-template-columns: 200px 1fr;
  height: 100vh;
  gap: 10px;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

### Галерея с автоматическим расположением

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.gallery-item {
  height: 200px;
  background-color: #eee;
  border-radius: 8px;
}
```

### Сложная сетка с перекрывающимися элементами

```css
.complex-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(3, 100px);
  gap: 10px;
}

.item-a {
  grid-column: 1 / 3; /* занимает 1 и 2 колонки */
  grid-row: 1 / 2;    /* занимает 1 строку */
}

.item-b {
  grid-column: 2 / 5; /* занимает 2, 3 и 4 колонки */
  grid-row: 1 / 3;    /* занимает 1 и 2 строки */
}
```

## Лучшие практики

### 1. Используйте именованные области для сложных макетов

Именованные области делают код более читаемым и поддерживаемым:

```css
.layout {
  display: grid;
  grid-template-areas: 
    "header header header"
    "sidebar content aside"
    "footer footer footer";
  grid-template-rows: 60px 1fr 40px;
  grid-template-columns: 200px 1fr 150px;
}
```

### 2. Комбинируйте Grid с Flexbox

Grid и Flexbox отлично работают вместе. Используйте Grid для основного макета, а Flexbox для выравнивания элементов внутри отдельных ячеек:

```css
.grid-item {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### 3. Используйте responsive значения

Создавайте адаптивные сетки с помощью `minmax()` и `auto-fit`:

```css
.responsive-grid {
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
}
```

### 4. Обеспечьте доступность

Всегда учитывайте порядок элементов при работе с Grid, особенно когда визуальный порядок отличается от порядка в DOM:

```css
.grid-container {
  grid-auto-flow: column; /* может повлиять на восприятие скринридерами */
}
```

### 5. Тестируйте на разных размерах экрана

Grid отлично поддерживает адаптивность, но важно протестировать макеты на разных размерах экрана:

```css
@media (max-width: 768px) {
  .layout {
    grid-template-columns: 1fr;
    grid-template-areas: 
      "header"
      "sidebar"
      "content"
      "aside"
      "footer";
  }
}
```

## Распространенные ошибки

### 1. Неправильное понимание единиц измерения

Не путайте `fr` с процентами. Единица `fr` распределяет оставшееся свободное пространство, а проценты — от общего размера родителя.

### 2. Игнорирование семантики HTML

Grid не должен нарушать логическую структуру HTML. Визуальное расположение элементов не должно мешать их логическому порядку.

### 3. Избыточное использование Grid

Не все макеты требуют использования Grid. Иногда Flexbox или обычные методы позиционирования будут более подходящими.

### 4. Неправильное использование gap

Свойство `gap` не работает с элементами, которые перекрываются. В таких случаях используйте `margin` для внутренних элементов.

### 5. Отсутствие резервных решений

Хотя поддержка Grid хороша, в некоторых случаях стоит предусмотреть резервные решения для старых браузеров:

```css
.container {
  display: flex; /* резервное решение */
  display: grid; /* основное решение */
}
```

## Заключение

CSS Grid — это мощный инструмент для создания сложных макетов. Его двумерная природа делает его идеальным для создания сеток, где важны как строки, так и колонки. При правильном использовании Grid может значительно упростить разработку сложных интерфейсов.

Ключ к успешному использованию Grid — понимание его основ, практика и следование лучшим практикам. Экспериментируйте с различными комбинациями свойств, чтобы найти оптимальные решения для ваших задач.

> [!tip] 
> Начинайте с простых сеток и постепенно переходите к более сложным. Практика — ключ к освоению Grid Layout.

> [!warning] 
> Не забывайте о доступности и семантике при использовании Grid. Визуальный порядок не должен нарушать логическую структуру документа.

> [!note] 
> CSS Grid отлично сочетается с другими методами CSS, особенно с Flexbox. Используйте каждый инструмент по его прямому назначению.

## Связанные темы

- [[CSS Flexbox]]
- [[CSS Layout Fundamentals]]
- [[Responsive Web Design]]
- [[CSS Positioning]]
- [[CSS Units]]
- [[CSS Media Queries]]
- [[HTML Semantic Elements]]
- [[CSS Transforms]]
- [[CSS Animations]]
- [[CSS Variables]]
- [[CSS Pseudo-elements]]
- [[CSS Selectors]]
- [[CSS Box Model]]
- [[CSS Display Property]]
- [[CSS Overflow]]

# Верстка - Grid

CSS Grid Layout (или просто Grid) - это мощная двухмерная система макета, которая позволяет создавать сложные макеты с точным контролем над расположением элементов по строкам и столбцам.

## Table of Contents

- [Основные концепции Grid](#основные-концепции-grid)
- [Свойства grid-контейнера](#свойства-grid-контейнера)
- [Свойства grid-элементов](#свойства-grid-элементов)
- [Функции Grid](#функции-grid)
- [Практические примеры](#практические-примеры)
- [Продвинутые техники Grid](#продвинутые-техники-grid)
- [Совместимость и поддержка](#совместимость-и-поддержка)
- [Производительность и оптимизация](#производительность-и-оптимизация)
- [Лучшие практики](#лучшие-практики)

## Основные концепции Grid

Grid работает с двумя типами элементов:
- **Grid-контейнер** - родительский элемент, к которому применяется `display: grid`
- **Grid-элементы** - непосредственные дочерние элементы grid-контейнера

```css
.grid-container {
  display: grid;  /* Создает grid-контейнер */
}

.grid-item {
  /* Автоматически становится grid-элементом */
}
```

### Оси Grid

Grid работает с двумя осями:
- **Inline axis (ось строк)**: горизонтальная ось
- **Block axis (ось колонок)**: вертикальная ось

### Терминология Grid

- **Grid line**: Линии сетки, разделяющие колонки и строки
- **Grid track**: Колонка или строка сетки
- **Grid cell**: Одна ячейка сетки
- **Grid area**: Прямоугольная область, состоящая из нескольких ячеек
- **Grid gap**: Промежутки между ячейками

## Свойства grid-контейнера

### display

Создает grid-контейнер:

```css
.container {
  display: grid;          /* Блочный grid-контейнер */
  display: inline-grid;   /* Встроенный grid-контейнер */
}
```

### grid-template-columns и grid-template-rows

Определяют структуру сетки, указывая размеры колонок и строк:

```css
.container {
  /* Задание фиксированных размеров */
  grid-template-columns: 100px 200px 150px;
  grid-template-rows: 50px 100px;

  /* Задание с использованием различных единиц */
  grid-template-columns: 200px 1fr 2fr;  /* 1fr = 1 доля свободного места */
  grid-template-rows: auto 1fr auto;

  /* Повторение шаблона с repeat() */
  grid-template-columns: repeat(3, 1fr);  /* Три равные колонки */
  grid-template-columns: repeat(2, 100px 1fr);  /* Повторение шаблона */

  /* Минимальные и максимальные размеры */
  grid-template-columns: repeat(3, minmax(100px, 1fr));
}
```

### grid-template-areas

Определяет именованные области сетки:

```css
.container {
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

### column-gap и row-gap

Определяют промежутки между колонками и строками:

```css
.container {
  column-gap: 20px;    /* Промежуток между колонками */
  row-gap: 15px;       /* Промежуток между строками */

  /* Или сокращенное свойство */
  gap: 15px 20px;      /* row-gap column-gap */
  gap: 20px;           /* Одинаковый промежуток для строк и колонок */
}
```

### grid-auto-columns и grid-auto-rows

Определяют размер автоматически создаваемых колонок и строк:

```css
.container {
  grid-template-columns: repeat(3, 100px);
  grid-auto-columns: 150px;  /* Автоматически созданные колонки будут 150px */
  grid-auto-rows: 100px;     /* Автоматически созданные строки будут 100px */
}
```

### grid-auto-flow

Определяет, как автоматически размещаются элементы:

```css
.container {
  /* Значения: row | column | row dense | column dense */
  grid-auto-flow: row;        /* Заполнение по строкам (по умолчанию) */
  grid-auto-flow: column;     /* Заполнение по колонкам */
  grid-auto-flow: row dense;  /* Заполнение по строкам с плотной упаковкой */
}
```

### justify-items

Выравнивает элементы вдоль оси колонок:

```css
.container {
  /* Значения: start | end | center | stretch */
  justify-items: start;    /* Прижатие к началу ячейки */
  justify-items: end;      /* Прижатие к концу ячейки */
  justify-items: center;   /* Центрирование в ячейке */
  justify-items: stretch;  /* Растяжение на всю ширину ячейки (по умолчанию) */
}
```

### align-items

Выравнивает элементы вдоль оси строк:

```css
.container {
  /* Значения: start | end | center | stretch */
  align-items: start;      /* Прижатие к началу ячейки */
  align-items: end;        /* Прижатие к концу ячейки */
  align-items: center;     /* Центрирование в ячейке */
  align-items: stretch;    /* Растяжение на всю высоту ячейки (по умолчанию) */
}
```

### justify-content

Выравнивает сетку внутри контейнера по оси колонок:

```css
.container {
  /* Значения: start | end | center | stretch | space-around | space-between | space-evenly */
  justify-content: start;        /* Прижатие к началу контейнера */
  justify-content: center;       /* Центрирование сетки */
  justify-content: space-between; /* Равномерное распределение */
}
```

### align-content

Выравнивает сетку внутри контейнера по оси строк:

```css
.container {
  /* Значения: start | end | center | stretch | space-around | space-between | space-evenly */
  align-content: start;      /* Прижатие к началу контейнера */
  align-content: center;     /* Центрирование сетки */
  align-content: space-around; /* Равномерное распределение с отступами */
}
```

## Свойства grid-элементов

### grid-column-start и grid-column-end

Определяют, с какой колонки начинается элемент и на какой заканчивается:

```css
.item {
  grid-column-start: 2;    /* Начинается с 2-й колонки */
  grid-column-end: 4;      /* Заканчивается перед 4-й колонкой */

  /* Или сокращенное свойство */
  grid-column: 2 / 4;      /* Эквивалентно двум предыдущим свойствам */
  grid-column: 1 / -1;     /* Занимает всю ширину сетки */
}
```

### grid-row-start и grid-row-end

Определяют, с какой строки начинается элемент и на какой заканчивается:

```css
.item {
  grid-row-start: 1;       /* Начинается с 1-й строки */
  grid-row-end: 3;         /* Заканчивается перед 3-й строкой */

  /* Или сокращенное свойство */
  grid-row: 1 / 3;         /* Эквивалентно двум предыдущим свойствам */
}
```

### grid-column и grid-row

Сокращенные свойства для определения положения элемента:

```css
.item {
  grid-column: 1 / 3;      /* Колонки с 1 по 3 */
  grid-row: 2 / span 2;    /* Начинается с 2-й строки и занимает 2 строки */
}
```

### grid-area

Сокращенное свойство для всех четырех значений:

```css
.item {
  /* grid-row-start / grid-column-start / grid-row-end / grid-column-end */
  grid-area: 1 / 2 / 3 / 4;

  /* Или имя области (если используется grid-template-areas) */
  grid-area: header;
}
```

### justify-self

Выравнивает отдельный элемент по оси колонок:

```css
.item {
  /* Значения: auto | start | end | center | stretch */
  justify-self: start;     /* Прижатие к началу ячейки */
  justify-self: center;    /* Центрирование в ячейке */
  justify-self: end;       /* Прижатие к концу ячейки */
  justify-self: stretch;   /* Растяжение на всю ширину ячейки */
}
```

### align-self

Выравнивает отдельный элемент по оси строк:

```css
.item {
  /* Значения: auto | start | end | center | stretch */
  align-self: start;       /* Прижатие к началу ячейки */
  align-self: center;      /* Центрирование в ячейке */
  align-self: end;         /* Прижатие к концу ячейки */
  align-self: stretch;     /* Растяжение на всю высоту ячейки */
}
```

## Функции Grid

Grid предоставляет несколько полезных функций:

### minmax()

Определяет минимальный и максимальный размер:

```css
.grid-container {
  grid-template-columns: repeat(3, minmax(200px, 1fr));
  /* Каждая колонка будет минимум 200px, но может расти до 1fr */
}
```

### fit-content()

Ограничивает размер содержимым, но не превышает указанное значение:

```css
.grid-container {
  grid-template-columns: fit-content(300px) 1fr;
  /* Первая колонка будет размером с содержимое, но не больше 300px */
}
```

### repeat()

Повторяет шаблон колонок или строк:

```css
.grid-container {
  grid-template-columns: repeat(5, 1fr);  /* 5 равных колонок */
  grid-template-columns: repeat(2, 100px 1fr);  /* Повторение шаблона */
}
```

### clamp()

Ограничивает значение между минимумом и максимумом:

```css
.grid-container {
  grid-template-columns: repeat(auto-fit, minmax(clamp(200px, 4vw, 400px), 1fr));
  /* Колонки будут адаптироваться в пределах 200px-400px */
}
```

## Практические примеры

### Пример 1: Основной макет страницы

```css
.page-layout {
  display: grid;
  grid-template-areas:
    "header"
    "main"
    "sidebar"
    "footer";
  grid-template-rows: 80px 1fr 200px 60px;
  grid-template-columns: 1fr;
  height: 100vh;
  gap: 10px;
}

.header { grid-area: header; }
.main { grid-area: main; }
.sidebar { grid-area: sidebar; }
.footer { grid-area: footer; }
```

### Пример 2: Адаптивная сетка карточек

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
}

.card {
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
```

### Пример 3: Сложный макет с перекрывающимися элементами

```css
.complex-layout {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: repeat(3, 100px);
  gap: 10px;
  height: 400px;
}

.item1 {
  grid-column: 1 / 3;    /* Занимает колонки 1-2 */
  grid-row: 1 / 2;       /* Занимает строку 1 */
}

.item2 {
  grid-column: 2 / 5;    /* Занимает колонки 2-4 */
  grid-row: 1 / 3;       /* Занимает строки 1-2 */
}

.item3 {
  grid-column: 1 / 2;    /* Занимает колонку 1 */
  grid-row: 2 / 4;       /* Занимает строки 2-3 */
}
```

### Пример 4: Макет галереи

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  grid-auto-rows: minmax(200px, auto);
  gap: 15px;
}

.gallery-item {
  background-color: #f0f0f0;
  border-radius: 8px;
  overflow: hidden;
}

.gallery-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

## Продвинутые техники Grid

### Создание сложных макетов с именованными областями

```css
.complex-page {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  grid-template-columns: 200px 1fr 150px;
  grid-template-rows: 80px 1fr 60px;
  height: 100vh;
  gap: 10px;
}

.header { 
  grid-area: header;
  background: #333;
  color: white;
  display: flex;
  align-items: center;
  padding: 0 20px;
}

.sidebar { 
  grid-area: sidebar;
  background: #f8f9fa;
  padding: 15px;
}

.main { 
  grid-area: main;
  background: white;
  padding: 20px;
  overflow-y: auto;
}

.aside { 
  grid-area: aside;
  background: #e9ecef;
  padding: 15px;
}

.footer { 
  grid-area: footer;
  background: #6c757d;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

### Использование grid-auto-flow для динамического размещения

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 100px;
  grid-auto-flow: row; /* Элементы заполняют сетку по строкам */
  gap: 15px;
  padding: 20px;
}

/* Элементы будут автоматически размещаться в сетке */
.widget {
  background: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 8px;
  padding: 15px;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

### Использование grid-template-areas с дублированием

```css
.form-layout {
  display: grid;
  grid-template-areas:
    "label input input"
    "label input input"
    "error error error"
    "actions actions actions";
  grid-template-columns: max-content 1fr 1fr;
  grid-template-rows: auto auto auto auto;
  gap: 10px;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.form-label { grid-area: label; }
.form-input { grid-area: input; }
.form-error { grid-area: error; }
.form-actions { grid-area: actions; }
```

### Создание адаптивных макетов с repeat() и auto-fit

```css
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  padding: 20px;
}

.grid-item {
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

/* Дополнительная адаптация для мобильных устройств */
@media (max-width: 600px) {
  .responsive-grid {
    grid-template-columns: 1fr; /* Одна колонка на мобильных */
    gap: 15px;
  }
}
```

### Использование grid с CSS-переменными для гибкости

```css
:root {
  --grid-columns: repeat(auto-fit, minmax(300px, 1fr));
  --grid-gap: 20px;
  --grid-areas: "header header" "sidebar main" "footer footer";
  --grid-rows: auto 1fr auto;
}

.flexible-grid {
  display: grid;
  grid-template-columns: var(--grid-columns);
  grid-template-rows: var(--grid-rows);
  grid-template-areas: var(--grid-areas);
  gap: var(--grid-gap);
}

/* Можно легко изменить переменные для разных состояний */
@media (min-width: 768px) {
  :root {
    --grid-columns: repeat(3, 1fr);
    --grid-areas: "header header header" "sidebar main aside" "footer footer footer";
    --grid-rows: auto 1fr auto;
  }
}
```

### Создание макетов с дроблением содержимого

```css
.article-layout {
  display: grid;
  grid-template-columns: 1fr 300px;
  grid-template-areas:
    "content sidebar"
    "content sidebar";
  gap: 30px;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.article-content {
  grid-area: content;
  line-height: 1.6;
}

.article-sidebar {
  grid-area: sidebar;
  background: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  align-self: start; /* Прижимаем к верху */
}
```

## Совместимость и поддержка

Grid имеет отличную поддержку во всех современных браузерах. Для поддержки старых версий IE можно использовать префиксы:

```css
.grid-container {
  display: -ms-grid;  /* Для IE */
  display: grid;
}

/* Для IE10/IE11 может потребоваться более подробное определение */
.ie-grid {
  display: -ms-grid;
  -ms-grid-columns: 1fr 20px 1fr 20px 1fr; /* Определение колонок */
  -ms-grid-rows: auto 10px auto; /* Определение строк */
}

.ie-grid .item1 {
  -ms-grid-column: 1; /* Номер колонки */
  -ms-grid-row: 1;    /* Номер строки */
}
```

### Проверка поддержки через CSS

```css
/* Проверка поддержки grid */
@supports (display: grid) {
  .container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 20px;
  }
}

/* Резервный вариант для старых браузеров */
@supports not (display: grid) {
  .container {
    display: flex;
    flex-wrap: wrap;
  }
  
  .item {
    flex: 0 0 calc(33.333% - 20px);
    margin: 10px;
  }
}
```

## Производительность и оптимизация

### Оптимизация производительности Grid

Grid может влиять на производительность в следующих аспектах:

1. **Сложные вычисления макета**: Сложные grid-структуры требуют больше вычислений
2. **Перерасчет макета**: Изменение размеров элементов может вызывать перерасчет макета

```css
/* Оптимизированный grid для анимаций */
.optimized-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* Используйте transform и opacity для анимаций */
  will-change: transform;
}

.grid-item {
  transition: transform 0.3s ease;
}

.grid-item:hover {
  transform: scale(1.05); /* Использует GPU */
}
```

### Использование contain для изоляции

```css
/* Использование contain для изоляции grid-контейнера */
.isolated-grid-container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  contain: layout style paint;
  /* Ограничивает влияние на окружающий макет */
}
```

### Оптимизация при работе с большим количеством элементов

```css
/* Для большого количества элементов используйте виртуализацию или постраничную загрузку */
.performance-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  /* Ограничьте количество отображаемых элементов */
}

/* Используйте position: sticky для фиксированных элементов в сетке */
.sticky-header {
  position: sticky;
  top: 0;
  background: white;
  z-index: 10;
}
```

## Лучшие практики

### 1. Используйте grid для двумерных макетов
Когда важен порядок элементов по двум осям:

```css
/* Хорошо: двумерный макет */
.dashboard {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 200px);
}

/* Не так хорошо: для одномерных макетов лучше использовать Flexbox */
.navigation {
  /* Используйте Flexbox вместо Grid */
}
```

### 2. Комбинируйте с flexbox
Используйте grid для основного макета и flexbox для выравнивания внутри элементов:

```css
.page-layout {
  display: grid;
  grid-template-areas: "header main sidebar";
  grid-template-columns: 1fr 2fr 1fr;
  min-height: 100vh;
}

.main-content {
  grid-area: main;
  display: flex; /* Flexbox внутри grid */
  flex-direction: column;
}

.card {
  display: flex; /* Flexbox для внутреннего выравнивания */
  flex-direction: column;
  height: 100%;
}
```

### 3. Используйте именованные области для сложных макетов
Это улучшает читаемость кода:

```css
/* Хорошо: именованные области для сложного макета */
.complex-layout {
  display: grid;
  grid-template-areas:
    "header header header"
    "nav main aside"
    "nav footer footer";
  grid-template-columns: 200px 1fr 250px;
  grid-template-rows: 80px 1fr 60px;
}
```

### 4. Используйте minmax() для создания адаптивных колонок
```css
.adaptive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}
```

### 5. Тестируйте на разных размерах экрана
Убедитесь, что макет адаптируется должным образом:

```css
/* Адаптивный grid */
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
}

/* Адаптация для мобильных */
@media (max-width: 768px) {
  .responsive-grid {
    grid-template-columns: 1fr; /* Одна колонка на мобильных */
    gap: 10px;
  }
}
```

### 6. Используйте семантические имена классов
Это улучшает читаемость и поддержку кода:

```css
/* Хорошие имена классов */
.main-grid { }
.article-grid { }
.product-grid { }

/* Плохие имена классов */
.g1 { }
.grid123 { }
.box-grid { }
```

### 7. Используйте CSS-переменные для согласованности
```css
:root {
  --grid-gap: 20px;
  --grid-columns: repeat(auto-fit, minmax(300px, 1fr));
  --grid-areas: "header" "main" "footer";
}

.main-grid {
  display: grid;
  grid-template-columns: var(--grid-columns);
  grid-template-areas: var(--grid-areas);
  gap: var(--grid-gap);
}
```

### 8. Обрабатывайте крайние случаи
```css
/* Обработка пустых элементов */
.grid-container:empty {
  display: none;
}

/* Обработка одного элемента */
.grid-container:only-child {
  justify-content: center;
}

/* Обработка элементов с длинным содержимым */
.grid-item {
  overflow-wrap: break-word;
  min-width: 0; /* Позволяет элементу сжиматься */
}
```

### 9. Используйте логические свойства для международных проектов
```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* Вместо gap используйте логические свойства */
  column-gap: 20px;
  row-gap: 15px;
}
```

## Заключение

CSS Grid - это мощный инструмент для создания сложных двумерных макетов. Он предоставляет точный контроль над расположением элементов и позволяет создавать макеты, которые ранее требовали сложных хаков или вспомогательных библиотек. Современные возможности Grid, такие как именованные области, функции minmax() и repeat(), делают его еще более гибким и мощным. Понимание свойств grid позволяет создавать гибкие и адаптивные веб-интерфейсы, а знание лучших практик помогает избежать распространенных ошибок и оптимизировать производительность.

#programming #css #grid #layout #web-development #frontend #responsive-design
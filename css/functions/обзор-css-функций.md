---
aliases: ["CSS функции", "Функции CSS", "CSS-функции обзор"]
tags: ["css", "functions", "calc", "clamp", "min", "max", "attr", "counter"]
---

# Полный обзор CSS-функций

## Введение

CSS-функции - это мощные инструменты, которые позволяют выполнять вычисления, манипулировать значениями и создавать динамические стили. Они значительно расширяют возможности каскадных таблиц стилей, позволяя создавать более гибкие и адаптивные интерфейсы.

## Основные математические функции

### calc()

Функция `calc()` позволяет выполнять математические вычисления внутри CSS-свойств. Она поддерживает сложение, вычитание, умножение и деление.

#### Синтаксис

```css
property: calc(expression);
```

#### Примеры использования

```css
/* Смешивание разных единиц измерения */
.calc-example {
  width: calc(100% - 80px); /* 100% минус 80 пикселей */
  height: calc(50vh - 2rem); /* 50% высоты вьюпорта минус 2 рема */
  margin: calc(2em + 10px); /* 2 эма плюс 10 пикселей */
}

/* Использование с CSS-переменными */
:root {
  --sidebar-width: 250px;
  --header-height: 60px;
}

.main-content {
  width: calc(100% - var(--sidebar-width));
  height: calc(100vh - var(--header-height));
}

/* Сложные вычисления */
.complex-calc {
  padding: calc((100% - 600px) / 2); /* Центрирование при максимальной ширине 600px */
  font-size: calc(1rem + 0.5vw); /* Адаптивный размер шрифта */
}
```

> [!tip] Совет
> Функция `calc()` позволяет использовать разные единицы измерения в одном выражении, что особенно полезно для создания адаптивных макетов.

### min() и max()

Функции `min()` и `max()` позволяют выбирать минимальное или максимальное значение из списка, что идеально подходит для создания адаптивных размеров.

#### Синтаксис

```css
property: min(value1, value2, ...);
property: max(value1, value2, ...);
```

#### Примеры использования

```css
/* min() - выбирает наименьшее значение */
.min-example {
  width: min(300px, 100%); /* Не превышает 300px или 100% */
  font-size: min(4vw, 24px); /* Выбирает меньшее из двух значений */
}

/* max() - выбирает наибольшее значение */
.max-example {
  width: max(300px, 50%); /* Не меньше 300px или 50% от родителя */
  min-width: max(15em, 200px); /* Наименьшая ширина - максимум из двух значений */
}

/* clamp() - комбинация min и max */
.clamp-example {
  font-size: clamp(1rem, 2.5vw, 2rem); /* Минимум 1rem, идеал 2.5vw, максимум 2rem */
  padding: clamp(10px, 3vw, 30px);
}
```

### clamp()

Функция `clamp()` позволяет задать минимальное, предпочтительное и максимальное значение. Это идеальный инструмент для создания адаптивных размеров.

#### Синтаксис

```css
property: clamp(minimum, preferred, maximum);
```

#### Примеры использования

```css
/* Адаптивный размер шрифта */
.responsive-text {
  font-size: clamp(1rem, 2.5vw, 2rem);
}

/* Адаптивные размеры элементов */
.responsive-box {
  width: clamp(200px, 50%, 500px);
  height: clamp(100px, 25vh, 300px);
  padding: clamp(8px, 2vw, 20px);
}

/* Адаптивные отступы */
.responsive-spacing {
  margin: clamp(1rem, 5vw, 4rem);
  gap: clamp(0.5rem, 2vw, 2rem);
}
```

## Функции получения значений

### attr()

Функция `attr()` позволяет получать значение атрибута HTML-элемента и использовать его в CSS. Это особенно полезно для создания динамических стилей на основе данных из HTML.

#### Синтаксис

```css
property: attr(attribute-name unit, fallback-value);
```

#### Примеры использования

```css
/* Использование атрибута data-* */
.data-attr-example::before {
  content: attr(data-label);
  background-color: attr(data-color, red); /* Красный как резервное значение */
}

/* Использование стандартных атрибутов */
.title::after {
  content: " (" attr(title) ")";
}

/* Использование с единицами измерения */
.width-from-attr {
  width: attr(data-width px, 100px); /* Значение в пикселях */
  height: attr(data-height px, auto);
}

/* Практический пример: прогресс-бар */
.progress-bar {
  width: attr(data-progress %, 0%);
  background: linear-gradient(to right, #4CAF50 0%, #4CAF50 attr(data-progress %), #e0e0e0 attr(data-progress %), #e0e0e0 100%);
}
```

### counter()

Функция `counter()` используется для создания автоматических счетчиков в CSS. Она работает в сочетании с свойствами `counter-reset` и `counter-increment`.

#### Синтаксис

```css
counter(name, style);
counters(name, string, style);
```

#### Примеры использования

```css
/* Простой счетчик */
ol {
  counter-reset: item; /* Сброс счетчика */
}

li {
  counter-increment: item; /* Увеличение счетчика */
}

li::before {
  content: counters(item, ".") ". "; /* Отображение значения счетчика */
}

/* Нумерация заголовков */
h1, h2, h3, h4, h5, h6 {
  counter-increment: heading;
}

h1::before {
  content: counter(heading) ". ";
}

/* Сложная нумерация */
.document {
  counter-reset: chapter;
}

.chapter {
  counter-increment: chapter;
  counter-reset: section;
}

.chapter::before {
  content: counter(chapter) ". ";
}

.section {
  counter-increment: section;
  counter-reset: subsection;
}

.section::before {
  content: counter(chapter) "." counter(section) " ";
}

.subsection {
  counter-increment: subsection;
}

.subsection::before {
  content: counter(chapter) "." counter(section) "." counter(subsection) " ";
}

/* Счетчики с пользовательским стилем */
.custom-counter {
  counter-reset: custom-counter;
}

.custom-counter li {
  counter-increment: custom-counter;
}

.custom-counter li::before {
  content: "(" counter(custom-counter, upper-roman) ") "; /* Римские цифры в верхнем регистре */
}
```

## Функции цвета

### color() и color-mix()

Функции для работы с цветами в различных цветовых пространствах.

```css
.color-functions {
  /* Функция color() для указания цвета в конкретном цветовом пространстве */
  background-color: color(srgb 0.5 0.5 1); /* sRGB */
  border-color: color(display-p3 1 0.5 0.5); /* Display P3 */
  
  /* Функция color-mix() для смешивания цветов */
  color: color-mix(in srgb, red 70%, blue 30%);
  background: color-mix(in oklch, oklch(0.7 0.15 240) 60%, white 40%);
}
```

## Функции трансформации

### translate(), rotate(), scale()

Функции для преобразования элементов в 2D и 3D пространстве.

```css
.transform-functions {
  transform: translate(10px, 20px); /* Перемещение */
  transform: rotate(45deg); /* Вращение */
  transform: scale(1.2); /* Масштабирование */
  transform: translateX(30px) rotate(30deg) scale(0.8); /* Комбинация */
}
```

## Функции градиента

### linear-gradient(), radial-gradient(), conic-gradient()

Функции для создания градиентов.

```css
.gradient-functions {
  /* Линейный градиент */
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  
  /* Радиальный градиент */
  background: radial-gradient(circle, #ff6b6b, #4ecdc4);
  
  /* Конический градиент */
  background: conic-gradient(from 0deg, #ff6b6b, #4ecdc4, #45b7d1, #ff6b6b);
}
```

## Функции фильтрации

### filter() и related functions

Функции для применения визуальных эффектов.

```css
.filter-functions {
  filter: blur(5px); /* Размытие */
  filter: brightness(1.2); /* Яркость */
  filter: contrast(1.5); /* Контраст */
  filter: grayscale(0.5); /* Оттенки серого */
  filter: hue-rotate(90deg); /* Поворот оттенка */
  filter: invert(0.3); /* Инверсия */
  filter: opacity(0.8); /* Прозрачность */
  filter: saturate(2); /* Насыщенность */
  filter: sepia(0.6); /* Сепия */
  
  /* Комбинация фильтров */
  filter: blur(2px) brightness(1.1) contrast(1.2);
}
```

## Функции формы

### path(), polygon()

Функции для определения сложных форм.

```css
.shape-functions {
  /* Определение формы с помощью пути */
  clip-path: path('M 20 20 H 80 V 80 H 20 Z');
  
  /* Полигональная форма */
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
}
```

## Практические примеры комбинации функций

### Адаптивная сетка с calc() и min()

```css
.responsive-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
  gap: calc(1rem + 2vw);
}
```

### Динамический размер шрифта с clamp() и CSS-переменными

```css
:root {
  --min-font-size: 1rem;
  --max-font-size: 2.5rem;
  --responsive-ratio: 0.5vw;
}

.dynamic-text {
  font-size: clamp(
    var(--min-font-size), 
    calc(var(--min-font-size) + var(--responsive-ratio)), 
    var(--max-font-size)
  );
}
```

### Счетчик с attr() и counter()

```css
.task-list {
  counter-reset: task-counter;
}

.task-item {
  counter-increment: task-counter;
}

.task-item::before {
  content: counter(task-counter) ". " attr(data-task-name);
}
```

## Совместимость и резервные значения

```css
.compatibility-example {
  /* Резервное значение для calc() */
  width: 50%;
  width: calc(50% - 20px);
  
  /* Резервные значения для новых функций */
  font-size: 1.2rem;
  font-size: clamp(1rem, 2.5vw, 1.5rem);
  
  /* Резервное значение для attr() */
  color: blue;
  color: attr(data-color, blue);
}
```

## Заключение

CSS-функции предоставляют мощные возможности для создания динамических и адаптивных стилей. Они позволяют выполнять вычисления, манипулировать значениями и создавать сложные визуальные эффекты без использования JavaScript. Понимание и правильное использование этих функций значительно расширяет возможности веб-разработки.

## См. также

- [[продвинутые-функции-css]]
- [[практическое-применение-функций-в-адаптивном-дизайне]]
- [[цветовые-функции-и-их-применение]]
- [[CSS-математические-функции]]
- [[цветовые-пространства-и-форматы]]
- [[работа-с-цветовыми-темами-и-контрастностью]]
- [[2d-и-3d-трансформации]]
- [[перспектива-и-3d-пространство-в-css]]
- [[трансформации-и-производительность]]
- [[css-фильтры-применение-и-оптимизация]]
- [[css-формы-и-обтекание]]
- [[комбинирование-фильтров-и-создание-визуальных-эффектов]]
- [[современная-типографика-в-css]]
- [[адаптивная-типографика]]
- [[подстановка-шрифтов-и-производительность]]
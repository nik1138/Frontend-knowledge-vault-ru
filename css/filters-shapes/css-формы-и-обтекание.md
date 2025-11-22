---
aliases: ["CSS формы", "Обтекание CSS", "shape-outside", "shape-margin", "CSS clipping"]
tags: ["css", "shapes", "layout", "shape-outside", "shape-margin", "clipping", "wrapping"]
---

# CSS-формы и обтекание: shape-outside, shape-margin

## Введение

CSS-формы и свойства обтекания позволяют создавать сложные макеты, где текст и другие элементы обтекают нестандартные формы, а не только прямоугольные блоки. Свойства `shape-outside` и `shape-margin` открывают возможности для создания макетов, схожих с профессиональным полиграфическим дизайном.

## Основы обтекания

### Принципы работы обтекания

Обтекание в CSS позволяет элементам определять форму, вокруг которой будет происходить обтекание содержимого. Это особенно полезно для создания макетов с изображениями нестандартной формы, где текст должен обтекать изображение по его контуру.

```css
/* Базовое обтекание */
.basic-wrap {
  float: left;
  width: 200px;
  height: 200px;
  shape-outside: circle(50% at center);
  shape-margin: 10px;
}
```

## Свойство shape-outside

Свойство `shape-outside` определяет форму, вокруг которой будет происходить обтекание текста или других элементов. Оно работает только с элементами, у которых `display` равен `block`, `inline-block` или `table-caption`.

### Формы на основе CSS-функций

#### circle()

Функция `circle()` создает круговую форму обтекания.

```css
.circle-wrap {
  float: left;
  width: 200px;
  height: 200px;
  background: #ff6b6b;
  shape-outside: circle();
}

/* Круг с определенным радиусом */
.circle-custom-radius {
  float: left;
  width: 200px;
  height: 200px;
  background: #4ecdc4;
  shape-outside: circle(40% at 30% 30%);
}

/* Круг с конкретным радиусом */
.circle-specific-radius {
  float: left;
  width: 200px;
  height: 200px;
  background: #45b7d1;
  shape-outside: circle(80px at center);
}
```

#### ellipse()

Функция `ellipse()` создает эллиптическую форму обтекания.

```css
.ellipse-wrap {
  float: left;
  width: 200px;
  height: 150px;
  background: #96ceb4;
  shape-outside: ellipse();
}

/* Эллипс с разными радиусами */
.ellipse-custom {
  float: left;
  width: 200px;
  height: 150px;
  background: #feca57;
  shape-outside: ellipse(60% 40% at 30% 40%);
}
```

#### inset()

Функция `inset()` создает прямоугольную форму с возможностью скругления углов.

```css
.inset-wrap {
  float: left;
  width: 200px;
  height: 150px;
  background: #ff9ff3;
  shape-outside: inset(10px 20px 15px 25px round 20px);
}

/* Простой прямоугольник с закругленными углами */
.inset-rounded {
  float: left;
  width: 200px;
  height: 150px;
  background: #54a0ff;
  shape-outside: inset(0 0 0 0 round 30px);
}
```

#### polygon()

Функция `polygon()` позволяет создавать произвольные многоугольные формы обтекания.

```css
.triangle-wrap {
  float: left;
  width: 200px;
  height: 200px;
  background: #5f27cd;
  shape-outside: polygon(50% 0%, 0% 100%, 100% 100%);
}

/* Произвольный многоугольник */
.polygon-wrap {
  float: left;
  width: 200px;
  height: 200px;
  background: #00d2d3;
  shape-outside: polygon(0% 0%, 100% 0%, 75% 50%, 100% 100%, 0% 100%);
}

/* Многоугольник с относительными координатами */
.polygon-relative {
  float: left;
  width: 200px;
  height: 200px;
  background: #ff6b6b;
  shape-outside: polygon(10% 10%, 90% 10%, 90% 90%, 10% 90%);
}
```

### Использование изображений для формы обтекания

```css
/* Использование альфа-канала изображения для формы обтекания */
.image-wrap {
  float: left;
  width: 250px;
  height: 250px;
  background: url('example-image.png') no-repeat center;
  background-size: cover;
  shape-outside: url('example-image.png');
}

/* С использованием маски */
.masked-wrap {
  float: left;
  width: 250px;
  height: 250px;
  background: url('example-image.jpg') no-repeat center;
  background-size: cover;
  shape-outside: url('shape-mask.svg');
}
```

### Использование градиентов для формы обтекания

```css
/* Использование градиента для определения формы */
.gradient-wrap {
  float: left;
  width: 200px;
  height: 200px;
  background: radial-gradient(circle, #ff6b6b 50%, transparent 50%);
  shape-outside: radial-gradient(circle, #ff6b6b 50%, transparent 50%);
}
```

## Свойство shape-margin

Свойство `shape-margin` определяет дополнительное пространство вокруг формы обтекания, увеличивая расстояние между элементом и обтекающим его содержимым.

```css
.shape-with-margin {
  float: left;
  width: 200px;
  height: 200px;
  background: #4ecdc4;
  shape-outside: circle();
  shape-margin: 20px; /* Добавляет 20px пространства вокруг формы */
}

/* Адаптивный отступ */
.responsive-margin {
  float: left;
  width: 200px;
  height: 200px;
  background: #45b7d1;
  shape-outside: circle();
  shape-margin: calc(10px + 2vw); /* Адаптивный отступ */
}
```

## Практические примеры использования

### Обтекание изображения с нестандартной формой

```css
.article-image {
  float: left;
  width: 300px;
  height: 300px;
  background: url('circular-image.jpg') no-repeat center;
  background-size: cover;
  shape-outside: circle(45% at center);
  shape-margin: 15px;
  margin: 10px 20px 10px 0;
  border-radius: 50%;
}
```

### Комплексное обтекание с использованием многоугольника

```css
.complex-wrap {
  float: left;
  width: 250px;
  height: 300px;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  shape-outside: polygon(
    0% 0%, 
    100% 0%, 
    80% 25%, 
    100% 50%, 
    80% 75%, 
    100% 100%, 
    0% 100%
  );
  shape-margin: 10px;
}
```

### Адаптивные формы обтекания

```css
.adaptive-shape {
  float: left;
  width: min(300px, 40vw);
  height: min(300px, 40vw);
  background: #96ceb4;
  shape-outside: circle(
    calc(50% - 10px) at 
    calc(50% + 20px) 
    calc(50% - 10px)
  );
  shape-margin: calc(5px + 1vw);
}
```

## Свойство clip-path в контексте форм

Хотя `clip-path` не влияет на обтекание, оно связано с формированием нестандартных форм:

```css
.clip-and-wrap {
  float: left;
  width: 200px;
  height: 200px;
  background: #5f27cd;
  /* Форма отображения */
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  /* Форма обтекания */
  shape-outside: polygon(50% 0%, 0% 100%, 100% 100%);
  shape-margin: 10px;
}
```

## Совмещение с другими CSS-свойствами

### Обтекание с flexbox

```css
.flex-shape-container {
  display: flex;
  align-items: flex-start;
}

.flex-shape-item {
  width: 200px;
  height: 200px;
  background: #00d2d3;
  shape-outside: circle();
  shape-margin: 15px;
  margin-right: 20px;
}

.flex-shape-content {
  flex: 1;
  /* Содержимое обтекает shape-outside элемент */
}
```

### Обтекание с CSS Grid

```css
.grid-shape-container {
  display: grid;
  grid-template-columns: 250px 1fr;
  gap: 20px;
}

.grid-shape-item {
  width: 100%;
  height: 250px;
  background: #ff9ff3;
  shape-outside: ellipse();
  shape-margin: 10px;
}

.grid-shape-content {
  /* Содержимое обтекает shape-outside элемент */
}
```

## Продвинутые техники

### Анимация форм обтекания

```css
.animated-shape {
  float: left;
  width: 200px;
  height: 200px;
  background: #54a0ff;
  shape-outside: circle(30%);
  shape-margin: 10px;
  transition: shape-outside 0.5s ease;
}

.animated-shape:hover {
  shape-outside: circle(45%);
}
```

### Использование CSS-переменных для динамических форм

```css
.dynamic-shape {
  --shape-radius: 40%;
  --shape-margin: 10px;
  
  float: left;
  width: 200px;
  height: 200px;
  background: #ff6b6b;
  shape-outside: circle(var(--shape-radius));
  shape-margin: var(--shape-margin);
}

/* Пример изменения формы */
.dynamic-shape.variant-1 {
  --shape-radius: 30%;
  --shape-margin: 15px;
}

.dynamic-shape.variant-2 {
  --shape-radius: 50%;
  --shape-margin: 5px;
}
```

## Практические применения

### Статья с обтекаемыми изображениями

```css
.article-content {
  font-size: 1.1rem;
  line-height: 1.6;
}

.article-image {
  float: left;
  width: 300px;
  height: 300px;
  background: url('feature-image.jpg') no-repeat center;
  background-size: cover;
  border-radius: 10px;
  shape-outside: circle(45% at center);
  shape-margin: 20px;
  margin: 0 25px 20px 0;
}

/* Адаптивное обтекание */
@media (max-width: 768px) {
  .article-image {
    float: none;
    width: 100%;
    height: 200px;
    margin: 0 0 15px 0;
    shape-outside: none;
  }
}
```

### Креативный макет с обтеканием

```css
.creative-layout {
  position: relative;
  padding: 20px;
}

.creative-element {
  position: absolute;
  top: 20px;
  left: 20px;
  width: 200px;
  height: 200px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  shape-outside: polygon(
    0% 0%, 
    100% 0%, 
    80% 50%, 
    100% 100%, 
    0% 100%, 
    20% 50%
  );
  shape-margin: 15px;
  clip-path: polygon(
    0% 0%, 
    100% 0%, 
    80% 50%, 
    100% 100%, 
    0% 100%, 
    20% 50%
  );
}

.creative-content {
  margin-left: 250px; /* Учитываем ширину обтекаемого элемента */
  padding: 20px;
}
```

## Совместимость и резервные значения

```css
.shape-fallback {
  /* Резервное значение для старых браузеров */
  float: left;
  width: 200px;
  height: 200px;
  margin: 0 20px 20px 0;
  
  /* Современное значение */
  shape-outside: circle();
  shape-margin: 15px;
}

/* Условия поддержки */
@supports (shape-outside: circle()) {
  .shape-enhanced {
    shape-outside: circle(45% at center);
    shape-margin: 10px;
  }
}
```

## Ограничения и особенности

### Ограничения shape-outside

1. Работает только с `float` элементами или элементами в `inline-flow`
2. Не работает с `position: absolute` или `position: fixed`
3. Требует определенных значений `display`

```css
/* Эти значения display поддерживают shape-outside */
.supported-display {
  display: block;        /* Поддерживается */
  display: inline-block; /* Поддерживается */
  display: table-caption; /* Поддерживается */
}

/* Эти значения display не поддерживают shape-outside */
.not-supported-display {
  display: flex;    /* Не поддерживается напрямую */
  display: grid;    /* Не поддерживается напрямую */
}
```

## Оптимизация производительности

### Оптимизация сложных форм

```css
/* Упрощение сложных многоугольников для лучшей производительности */
.optimized-polygon {
  shape-outside: polygon(
    0% 0%, 
    100% 0%, 
    100% 100%, 
    0% 100%
  ); /* Ближе к прямоугольнику для лучшей производительности */
  
  /* Вместо сложного многоугольника с множеством точек */
}

/* Использование простых форм когда возможно */
.simple-shapes-preferred {
  shape-outside: circle(); /* Быстрее, чем сложный многоугольник */
  shape-margin: 10px;
}
```

## Практические примеры

### Слайдер с обтекаемыми элементами

```css
.carousel-shape-container {
  display: flex;
  overflow-x: auto;
  padding: 20px 0;
}

.carousel-shape-item {
  flex: 0 0 auto;
  width: 250px;
  height: 200px;
  margin-right: 20px;
  background: #4ecdc4;
  shape-outside: polygon(10% 0%, 90% 0%, 100% 50%, 90% 100%, 10% 100%, 0% 50%);
  shape-margin: 10px;
}

.carousel-content {
  flex: 1;
  /* Содержимое обтекает слайдер */
}
```

### Интерактивные элементы с обтеканием

```css
.interactive-shape {
  float: left;
  width: 200px;
  height: 200px;
  background: #45b7d1;
  shape-outside: circle(40%);
  shape-margin: 15px;
  transition: shape-outside 0.3s ease;
  cursor: pointer;
}

.interactive-shape:hover {
  shape-outside: circle(45%);
}

.interactive-shape:active {
  shape-outside: circle(35%);
}
```

## Заключение

CSS-формы и свойства обтекания предоставляют мощные возможности для создания сложных макетов, где текст и другие элементы обтекают нестандартные формы. Свойства `shape-outside` и `shape-margin` позволяют создавать макеты, схожие с профессиональным полиграфическим дизайном, где элементы не ограничены прямоугольными рамками.

При использовании этих свойств важно учитывать совместимость с браузерами, обеспечивать резервные значения и тестировать макеты на разных устройствах. Также важно помнить об ограничениях этих свойств и оптимизировать сложные формы для лучшей производительности.

## См. также

- [[css-фильтры-применение-и-оптимизация]]
- [[комбинирование-фильтров-и-создание-визуальных-эффектов]]
- [[CSS-макеты]]
- [[CSS-формы]]
- [[цветовые-пространства-и-форматы]]
- [[цветовые-функции-и-их-применение]]
- [[работа-с-цветовыми-темами-и-контрастностью]]
- [[обзор-css-функций]]
- [[продвинутые-функции-css]]
- [[практическое-применение-функций-в-адаптивном-дизайне]]
- [[2d-и-3d-трансформации]]
- [[перспектива-и-3d-пространство-в-css]]
- [[трансформации-и-производительность]]
- [[современная-типографика-в-css]]
- [[адаптивная-типографика]]
- [[подстановка-шрифтов-и-производительность]]
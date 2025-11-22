---
aliases: [CSS Shapes Specification, CSS Shape Properties, CSS Clipping and Shapes, CSS Shape Functions]
tags: [css, shapes, layout, specification, css-shapes]
---

# CSS Shapes: Обтекание фигур, clip-path, shape-outside - Полное руководство

## Введение

CSS Shapes Module позволяет создавать нестандартные формы для обтекания текстом и других элементов, а также определять области обрезки элементов. Модуль предоставляет инструменты для создания сложных и визуально интересных макетов, выходящих за рамки прямоугольных форм.

## CSS Shapes Module Level 1

Level 1 определяет основные свойства для создания фигур и управления обтеканием.

### Свойство shape-outside

Свойство `shape-outside` определяет форму, вокруг которой обтекает текст. Оно применяется к плавающим элементам, элементам с `display: inline-block`, или элементам с `position: absolute/fixed`.

```css
.shape-outside-examples {
  /* Круг */
  shape-outside: circle(50% at center);
  
  /* Эллипс */
  shape-outside: ellipse(50% 30% at 25% 25%);
  
  /* Полигон */
  shape-outside: polygon(0 0, 100% 0, 50% 100%);
  
  /* Вписанное изображение */
  shape-outside: inset(10px 20px 10px 20px round 5px);
  
  /* Изображение с альфа-каналом */
  shape-outside: url(image.png);
  
  /* Градиент как фигура */
  shape-outside: radial-gradient(ellipse at center, transparent 60%, red 60%);
  
  /* Использование margin-box */
  shape-outside: circle(50%) margin-box;
  
  /* Значения для определения области */
  shape-outside: circle(50%) content-box;
  shape-outside: circle(50%) padding-box;
  shape-outside: circle(50%) border-box;
  shape-outside: circle(50%) margin-box;
}
```

### Свойство shape-image-threshold

Свойство `shape-image-threshold` определяет порог прозрачности для определения формы из изображения:

```css
.shape-threshold {
  shape-image-threshold: 0;         /* Все непрозрачные пиксели */
  shape-image-threshold: 0.5;       /* Пиксели с прозрачностью > 50% */
  shape-image-threshold: 1;         /* Только полностью непрозрачные */
}
```

### Свойство shape-margin

Свойство `shape-margin` добавляет отступ вокруг фигуры обтекания:

```css
.shape-margin {
  shape-margin: 10px;               /* Фиксированный отступ */
  shape-margin: 5%;                 /* Процентный отступ от размера элемента */
}
```

## Типы фигур

### Круг (circle)

```css
.circle-shape {
  shape-outside: circle(50% at center);
  /* или */
  shape-outside: circle(50px at 100px 100px);
  /* радиус и позиция центра */
}

/* Пример использования */
.circle-float {
  float: left;
  width: 200px;
  height: 200px;
  shape-outside: circle(50% at center);
  background: radial-gradient(circle, #ff0000, #0000ff);
}
```

### Эллипс (ellipse)

```css
.ellipse-shape {
  shape-outside: ellipse(60% 40% at center);
  /* или */
  shape-outside: ellipse(100px 50px at 150px 100px);
  /* rx ry и позиция центра */
}

/* Пример использования */
.ellipse-float {
  float: right;
  width: 300px;
  height: 200px;
  shape-outside: ellipse(45% 45% at center);
  background: radial-gradient(ellipse, #00ff00, #ffff00);
}
```

### Полигон (polygon)

```css
.polygon-shape {
  /* Простой треугольник */
  shape-outside: polygon(0 0, 100% 0, 50% 100%);
  
  /* Параллелограмм */
  shape-outside: polygon(20% 0%, 100% 0%, 80% 100%, 0% 100%);
  
  /* Сложный многоугольник */
  shape-outside: polygon(
    0% 0%, 
    100% 0%, 
    100% 20%, 
    80% 20%, 
    80% 40%, 
    60% 40%, 
    60% 60%, 
    40% 60%, 
    40% 80%, 
    20% 80%, 
    20% 100%, 
    0% 100%
  );
}

/* Пример использования */
.polygon-float {
  float: left;
  width: 200px;
  height: 200px;
  shape-outside: polygon(0 0, 100% 0, 50% 100%);
  background: linear-gradient(45deg, #ff0000, #0000ff);
}
```

### Вписанные прямоугольники (inset)

```css
.inset-shape {
  /* Простой прямоугольник с закругленными углами */
  shape-outside: inset(10px 20px 10px 20px round 5px);
  
  /* Параметры: top right bottom left / border-radius */
}
```

## CSS Shapes Module Level 2

Level 2 добавляет дополнительные возможности для работы с фигурами.

### Свойство shape-inside (экспериментальное)

```css
.shape-inside {
  /* Определяет форму внутри элемента */
  shape-inside: circle(50% at center);
  /* Управляет размещением содержимого внутри фигуры */
}
```

### Улучшенные возможности clip-path

```css
.enhanced-clip-path {
  /* Сложные формы обрезки */
  clip-path: path('M 0,0 L 100,0 L 100,50 Q 50,100 0,50 Z');
  /* Использование SVG path синтаксиса */
}
```

## Практические применения

### Обтекание изображений нестандартной формы

```css
.article-content {
  display: flex;
  gap: 20px;
}

.image-container {
  float: left;
  width: 200px;
  height: 200px;
  shape-outside: circle(50% at center);
  shape-margin: 10px;
  background: url('image.jpg') center/cover;
}

.article-text {
  flex: 1;
  /* Текст будет обтекать круглое изображение */
}
```

### Создание газетного макета

```css
.newspaper-layout {
  column-count: 3;
  column-gap: 30px;
  column-rule: 1px solid #ddd;
}

.shape-element {
  float: left;
  width: 150px;
  height: 200px;
  shape-outside: polygon(0 0, 100% 0, 80% 100%, 0 100%);
  shape-margin: 10px;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
}
```

### Интерактивные элементы с нестандартной формой

```css
.interactive-shape {
  width: 200px;
  height: 200px;
  clip-path: polygon(50% 0%, 100% 38%, 82% 100%, 18% 100%, 0% 38%);
  /* Форма в виде звезды */
  background: linear-gradient(45deg, #ff9a9e, #fecfef);
  transition: clip-path 0.3s ease;
}

.interactive-shape:hover {
  clip-path: circle(50% at center);
}
```

### Креативные формы обтекания

```css
.wave-shape {
  float: left;
  width: 300px;
  height: 150px;
  shape-outside: path('M0,75 Q75,0 150,75 T300,75 L300,150 L0,150 Z');
  shape-margin: 10px;
  background: linear-gradient(to bottom, #a8edea, #fed6e3);
}

.spiral-shape {
  float: right;
  width: 200px;
  height: 200px;
  shape-outside: path('M100,100 m-75,0 a75,75 0 1,0 150,0 a75,75 0 1,0 -150,0');
  shape-margin: 15px;
  background: conic-gradient(from 0deg, red, yellow, lime, aqua, blue, magenta, red);
}
```

## Свойство clip-path

Хотя `clip-path` формально принадлежит другому модулю, он тесно связан с CSS Shapes и часто используется совместно.

```css
.clip-path-examples {
  /* Основные фигуры */
  clip-path: circle(50% at center);
  clip-path: ellipse(50% 30% at center);
  clip-path: polygon(0 0, 100% 0, 50% 100%);
  clip-path: inset(10px 20px 10px 20px round 5px);
  
  /* Использование URL для SVG фигур */
  clip-path: url(#myClipPath);
  
  /* Использование path() для сложных фигур */
  clip-path: path('M 0,0 L 100,0 L 100,50 L 50,100 L 0,50 Z');
  
  /* Комбинации */
  clip-path: circle(50% at 25% 25%) border-box;
}
```

## Совместимость с макетами

### Flexbox и Shapes

```css
.flex-shapes {
  display: flex;
  align-items: center;
}

.flex-shape-item {
  flex: 0 0 200px;
  height: 200px;
  shape-outside: circle(50% at center);
  shape-margin: 10px;
  background: #3498db;
}
```

### Grid и Shapes

```css
.grid-shapes {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}

.grid-shape-item {
  height: 150px;
  clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
  /* Треугольная форма */
  background: #e74c3c;
}
```

## Производительность и оптимизация

```css
.optimized-shapes {
  /* Использование GPU ускорения */
  transform: translateZ(0);
  
  /* Упрощение сложных фигур при анимации */
  transition: clip-path 0.3s ease;
}

.optimized-shapes:hover {
  /* Более простая фигура при наведении */
  clip-path: circle(50% at center);
}
```

## Поддержка браузерами

- `shape-outside` с базовыми фигурами: Хорошая поддержка в современных браузерах
- `clip-path` с фигурами: Хорошая поддержка, с префиксами для старых браузеров
- `shape-image-threshold` и `shape-margin`: Хорошая поддержка в современных браузерах
- `path()` в `clip-path`: Поддержка в процессе внедрения

## Связанные темы

- [[CSS Masking Module]] - маскировка элементов
- [[CSS Clip Path]] - обрезка элементов
- [[CSS Layout Modules]] - модули компоновки

## Заключение

CSS Shapes Module предоставляет мощные инструменты для создания нестандартных форм обтекания и обрезки элементов. Понимание свойств фигур позволяет создавать визуально интересные и креативные макеты, выходящие за рамки традиционных прямоугольных компоновок.
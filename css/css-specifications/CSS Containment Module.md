---
aliases: [CSS Containment Specification, CSS Containment Properties, CSS Performance Optimization, CSS Containment Module]
tags: [css, containment, performance, specification, css-containment]
---

# CSS Containment Module: Производительность и изоляция - Полное руководство

## Введение

CSS Containment Module определяет способы изоляции элементов и их поддеревьев для оптимизации производительности рендеринга. Модуль позволяет браузеру применять оптимизации при вычислениях макета, стилей, рисовании и размерах элементов.

## CSS Containment Module Level 1

Level 1 определяет основные свойства контейнерности.

### Свойство contain

Свойство `contain` указывает, что элемент и его поддерево изолированы от остальной части документа в различных аспектах.

```css
.containment-examples {
  /* Основные значения */
  contain: none;                    /* Без контейнерности */
  contain: strict;                  /* Максимальная контейнерность */
  contain: content;                 /* Контейнерность содержимого */
  
  /* Отдельные аспекты контейнерности */
  contain: size;                    /* Контейнерность размеров */
  contain: layout;                  /* Контейнерность макета */
  contain: style;                   /* Контейнерность стилей */
  contain: paint;                   /* Контейнерность рисования */
  
  /* Комбинации аспектов */
  contain: layout paint;            /* Макет и рисование */
  contain: layout paint size;       /* Макет, рисование и размеры */
  contain: style paint;             /* Стили и рисование */
}
```

### Контейнерность размеров (size)

```css
.size-containment {
  contain: size;                    /* Браузер знает размер без изучения содержимого */
  width: 300px;                     /* Явно заданный размер */
  height: 200px;                    /* Или использовать aspect-ratio */
  aspect-ratio: 16/9;              /* Соотношение сторон */
  
  /* Элемент будет иметь фиксированный размер независимо от содержимого */
}

.size-containment-auto {
  contain: size;
  /* Браузер вычислит размер заранее, даже если он не фиксирован */
}
```

### Контейнерность макета (layout)

```css
.layout-containment {
  contain: layout;                  /* Изолирует макет внутри элемента */
  
  /* Браузер может игнорировать внешние влияния на макет */
  /* и внутренние влияния на внешний макет */
  
  /* Пример: float не влияет на элементы за пределами */
  float: left;
  width: 200px;
}

.layout-containment-complex {
  contain: layout;
  /* Содержимое не влияет на внешний макет */
  /* Внешние стили не влияют на внутренний макет */
}
```

### Контейнерность рисования (paint)

```css
.paint-containment {
  contain: paint;                   /* Изолирует рисование внутри элемента */
  
  /* Элемент и его содержимое не будут видны за пределами границы */
  /* даже если содержимое выходит за границы */
  
  /* Полезно для скрытия overflow без использования overflow: hidden */
  position: relative;
  z-index: 1;                      /* Создает новый контекст наложения */
}

.paint-containment-visual {
  contain: paint;
  /* Элементы за пределами границы не будут нарисованы */
  /* даже если они должны быть видны по CSS */
}
```

### Контейнерность стилей (style)

```css
.style-containment {
  contain: style;                   /* Изолирует стили внутри элемента */
  
  /* Счетчики, имена анимаций и другие стилевые особенности */
  /* не влияют на внешние элементы и не зависят от них */
  
  counter-reset: item-counter;      /* Счетчик изолирован */
}

.style-containment-counters {
  contain: style;
  /* counter-increment и другие стилевые особенности изолированы */
}
```

## CSS Containment Module Level 2

Level 2 добавляет дополнительные возможности и улучшения.

### Новые возможности контейнерности

```css
.enhanced-containment {
  /* Улучшенная контейнерность для специфических случаев */
  contain: size layout paint style;
  
  /* Более точный контроль над контейнерностью */
  contain-intrinsic-size: 300px 200px; /* Внутренний размер для контейнерности */
  contain-intrinsic-width: 300px;      /* Внутренняя ширина */
  contain-intrinsic-height: 200px;     /* Внутренняя высота */
}
```

### Свойства внутреннего размера

```css
.intrinsic-size {
  contain: layout;
  contain-intrinsic-size: 300px;        /* Ширина и высота одинаковы */
  contain-intrinsic-size: 300px 200px; /* Ширина и высота отдельно */
  
  /* Эти свойства помогают браузеру вычислить размеры заранее */
  /* даже если реальные размеры определяются динамически */
}

.intrinsic-width-height {
  contain: layout paint;
  contain-intrinsic-width: 400px;
  contain-intrinsic-height: auto;       /* Автоматическая высота */
  
  /* Используется для элементов с динамическим содержимым */
}
```

## Практические применения

### Карточки с контейнерностью

```css
.card-container {
  contain: layout paint style;
  width: 300px;
  margin: 10px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.card-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  contain: layout paint;
}

.card-content {
  contain: layout paint;
  margin-top: 10px;
}
```

### Списки с высокой производительностью

```css
.optimized-list {
  contain: layout paint;
  list-style: none;
  padding: 0;
  margin: 0;
}

.list-item {
  contain: layout paint style;
  padding: 10px;
  border-bottom: 1px solid #eee;
  
  /* Каждый элемент изолирован для оптимизации производительности */
}

.list-item:last-child {
  border-bottom: none;
}
```

### Компоненты с динамическим содержимым

```css
.dynamic-component {
  contain: layout paint size;
  contain-intrinsic-size: 300px 200px;
  /* Браузер заранее знает размеры, даже если содержимое меняется */
  
  transition: contain 0.3s ease;   /* Может быть анимировано при необходимости */
}

.component-content {
  /* Содержимое может динамически меняться */
  /* без влияния на внешний макет */
}
```

### Анимации с контейнерностью

```css
.animated-container {
  contain: layout paint;
  overflow: hidden;                /* Дополнительная оптимизация */
}

.animated-element {
  contain: paint;
  /* Изолируем анимацию внутри элемента */
  animation: slideIn 0.5s ease-out;
}

@keyframes slideIn {
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(0);
  }
}
```

### Галерея изображений

```css
.image-gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 15px;
  contain: layout paint;
}

.gallery-item {
  contain: layout paint size;
  contain-intrinsic-size: 250px 200px;
  position: relative;
  overflow: hidden;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.gallery-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
  contain: paint;
}

.gallery-image:hover {
  transform: scale(1.05);
}
```

## Современные возможности

### Контейнерность с CSS Custom Properties

```css
.dynamic-containment {
  --contain-value: layout paint;
  
  contain: var(--contain-value);
  /* Позволяет динамически изменять уровень контейнерности */
}

.dynamic-containment.full {
  --contain-value: strict;
}

.dynamic-containment.minimal {
  --contain-value: size;
}
```

### Контейнерность с анимациями

```css
.containment-transitions {
  contain: layout paint;
  transition: 
    contain 0.2s ease,
    opacity 0.3s ease;
}

.containment-transitions.minimal {
  contain: paint;                   /* Менее строгая контейнерность */
}
```

### Адаптивная контейнерность

```css
.responsive-containment {
  contain: layout paint;
}

@media (max-width: 768px) {
  .responsive-containment {
    contain: size layout paint;     /* Более строгая контейнерность на мобильных */
  }
}

@media (prefers-reduced-motion: reduce) {
  .responsive-containment {
    contain: strict;                /* Максимальная оптимизация при отключенных анимациях */
  }
}
```

## Производительность и оптимизация

### Измерение влияния контейнерности

```css
.performance-optimized {
  contain: layout paint;
  /* Оптимизация макета и рисования */
  
  /* Избегать чрезмерной контейнерности */
  /* которая может замедлить рендеринг в некоторых случаях */
}

/* Использование контейнерности для конкретных сценариев */
.animation-container {
  contain: paint;
  /* Подходит для анимаций, где содержимое не влияет на макет */
}

.layout-container {
  contain: layout;
  /* Подходит для статичных компонентов с фиксированным макетом */
}
```

### Контейнерность и виртуализация списков

```css
.virtualized-list {
  contain: layout paint;
  height: 400px;
  overflow: auto;
  position: relative;
}

.virtual-item {
  contain: layout paint size;
  contain-intrinsic-height: 50px;
  position: absolute;
  width: 100%;
  /* Только видимые элементы рендерятся для оптимизации производительности */
}
```

## Поддержка браузерами

- `contain: layout, paint, size`: Хорошая поддержка в современных браузерах
- `contain: style`: Поддержка в процессе внедрения
- `contain-intrinsic-size`: Хорошая поддержка в современных браузерах
- Некоторые значения могут требовать префиксов в старых браузерах

## Связанные темы

- [[CSS Performance Optimization]] - оптимизация производительности
- [[CSS Layout Modules]] - модули компоновки
- [[CSS Custom Properties Module]] - кастомные свойства

## Заключение

CSS Containment Module предоставляет мощные инструменты для оптимизации производительности веб-страниц. Понимание различных аспектов контейнерности позволяет создавать более быстрые и отзывчивые интерфейсы за счет изоляции элементов и их поддеревьев.
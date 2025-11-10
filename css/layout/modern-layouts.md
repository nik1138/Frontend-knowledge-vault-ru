---
aliases: ["Современные CSS-раскладки", "CSS Layout", "Модерные раскладки"]
tags: 
  - #css
  - #layout
  - #flexbox
  - #grid
  - #responsive-design
  - #css-modern
---

# Современные подходы к CSS-раскладке

## Введение

Современные CSS-раскладки предоставляют мощные инструменты для создания сложных и адаптивных интерфейсов. В этом руководстве мы рассмотрим основные концепции и методы, которые позволяют эффективно создавать современные веб-дизайны.

## Flexbox: Гибкая раскладка

Flexbox (Flexible Box Layout) - это одномерная система раскладки, позволяющая элементам внутри контейнера распределяться по главной и поперечной осям. Flexbox идеально подходит для создания компонентов и небольших раскладок.

```css
.container {
  display: flex;
  flex-direction: row; /* или column */
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
}

.item {
  flex: 1 1 auto; /* grow shrink basis */
  flex-grow: 1;
  flex-shrink: 0;
  flex-basis: 100px;
}
```

### Основные свойства контейнера

- `display: flex` - включает flex-раскладку
- `flex-direction` - направление главной оси
- `justify-content` - выравнивание по главной оси
- `align-items` - выравнивание по поперечной оси
- `flex-wrap` - перенос строк/колонок
- `align-content` - выравнивание строк при переносе

### Основные свойства элементов

- `flex-grow` - насколько элемент может расти
- `flex-shrink` - насколько элемент может сжиматься
- `flex-basis` - базовый размер элемента
- `order` - порядок элемента
- `align-self` - выравнивание конкретного элемента

> [!tip] 
> Используйте `flex: 1` как сокращение для `flex: 1 1 0%`, чтобы элемент занимал равное пространство.

## CSS Grid: Двумерная раскладка

CSS Grid - это двумерная система раскладки, позволяющая создавать сложные сетки с контролем по строкам и колонкам.

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  grid-gap: 20px;
  grid-template-areas: 
    "header header header"
    "sidebar main aside"
    "footer footer footer";
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

### Основные свойства контейнера

- `display: grid` - включает grid-раскладку
- `grid-template-columns/rows` - определяет размеры колонок и строк
- `grid-template-areas` - именованные области
- `grid-gap` - промежутки между элементами
- `grid-auto-flow` - направление автоматического размещения

### Основные свойства элементов

- `grid-column`/`grid-row` - позиционирование в сетке
- `grid-area` - объединяет row/column start/end
- `justify-self`/`align-self` - выравнивание элемента

> [!tip] 
> Используйте `minmax()` для создания адаптивных колонок: `grid-template-columns: repeat(auto-fit, minmax(250px, 1fr))`.

## Multi-column Layout

Многоколоночный макет позволяет создавать текстовые блоки, распределенные по нескольким колонкам, как в газетах.

```css
.newspaper {
  column-count: 3;
  column-gap: 2em;
  column-rule: 1px solid #ddd;
  column-width: 15em;
}
```

### Основные свойства

- `column-count` - количество колонок
- `column-width` - ширина колонки
- `column-gap` - промежуток между колонками
- `column-rule` - линия между колонками
- `column-span` - объединение колонок для элемента

## Display: Contents

Свойство `display: contents` позволяет элементу вести себя так, как будто его собственный контейнер не существует, но его потомки участвуют в раскладке родительского элемента.

```css
.card {
  display: contents;
}

.card > .header,
.card > .content,
.card > .footer {
  /* Эти элементы теперь прямые потомки .card */
  /* и могут участвовать в раскладке родителя */
}
```

## Intrinsic and Extrinsic Sizing

CSS теперь поддерживает интринсические (внутренние) и экстринсические (внешние) размеры:

- **Intrinsic sizes**: `min-content`, `max-content`, `fit-content()`
- **Extrinsic sizes**: значения, зависящие от внешнего контекста

```css
.element {
  width: max-content; /* Ширина содержимого */
  height: min-content; /* Минимальная высота */
  inline-size: fit-content; /* Адаптивный размер */
}
```

## Container Queries

Container queries позволяют применять стили на основе размеров родительского контейнера, а не окна просмотра.

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
  }
  
  .card img {
    width: 50%;
  }
}
```

> [!warning] 
> Container queries пока не поддерживаются в старых браузерах. Проверяйте совместимость перед использованием.

## Logical Properties

Логические свойства позволяют определять стили в зависимости от потока письма (LTR/RTL) и направления текста.

```css
.element {
  margin-inline-start: 10px; /* Вместо left/right */
  margin-inline-end: 10px;
  padding-block-start: 5px; /* Вместо top/bottom */
  padding-block-end: 5px;
  
  border-inline-start: 1px solid black;
  inset-block-start: 0; /* Вместо top */
  inset-inline-start: 0; /* Вместо left */
}
```

## Responsive Design Patterns

### Mobile-first подход

```css
.grid {
  display: grid;
  grid-template-columns: 1fr;
}

@media (min-width: 768px) {
  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### Container-based подход

```css
.card {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card-content {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}
```

## Modern Units

### Min(), Max(), Clamp()

```css
/* Адаптивный шрифт */
.title {
  font-size: clamp(1.2rem, 2.5vw, 2.5rem);
}

/* Адаптивная ширина */
.container {
  width: min(100%, 1200px); /* Не больше 1200px */
  max-width: max(300px, 50%); /* Не меньше 300px и не больше 50% */
}
```

## Aspect Ratio

Свойство `aspect-ratio` позволяет сохранять пропорции элемента:

```css
.image-container {
  aspect-ratio: 16 / 9;
  background-color: #f0f0f0;
}

.embed-responsive {
  position: relative;
  aspect-ratio: 16 / 9;
}

.embed-responsive iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
```

## Subgrid

Subgrid позволяет вложенным элементам участвовать в родительской grid-раскладке:

```css
.parent {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, auto);
}

.child {
  display: subgrid;
  grid-column: span 2;
  grid-row: span 2;
}
```

> [!warning] 
> Subgrid пока не полностью поддерживается во всех браузерах.

## Nesting (будущее)

CSS Nesting позволит использовать вложенные правила без препроцессоров:

```css
/* Будущий синтаксис */
.card {
  background: white;
  border-radius: 8px;
  
  &:hover {
    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
  }
  
  @media (min-width: 768px) {
    padding: 2rem;
  }
  
  & .title {
    font-size: 1.5rem;
  }
}
```

## Best Practices

1. **Выбирайте подходящий инструмент**: 
   - Flexbox для одномерных раскладок
   - Grid для двумерных раскладок
   - Multi-column для текстовых блоков

2. **Используйте семантическую разметку** независимо от визуального порядка

3. **Создавайте адаптивные раскладки** с помощью modern units и container queries

4. **Оптимизируйте производительность** - избегайте сложных раскладок при анимации

## Performance Considerations

- Избегайте частых перерасчетов раскладки
- Используйте `transform` и `opacity` для анимаций
- Минимизируйте использование `position: absolute` в сложных раскладках
- Тестируйте производительность на слабых устройствах

## Связь с другими файлами

- [[css/flexbox]] - подробное руководство по Flexbox
- [[css/grid]] - подробное руководство по CSS Grid
- [[css/responsive-design]] - адаптивный дизайн
- [[css/units]] - единицы измерения в CSS
- [[css/positioning]] - позиционирование элементов
- [[html/semantic-html]] - семантическая разметка

## Заключение

Современные CSS-раскладки предоставляют мощные инструменты для создания сложных и адаптивных интерфейсов. Понимание различий между Flexbox, Grid и другими методами позволяет выбирать наиболее подходящий инструмент для каждой задачи.
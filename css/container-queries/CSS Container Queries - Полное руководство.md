---
aliases: ["Container Queries", "Контейнерные запросы", "CSS Container Queries Guide", "Адаптивные контейнеры"]
tags: [css, responsive, container-queries, web-development, frontend]
---

# CSS Container Queries - Полное руководство

## Введение

CSS Container Queries (контейнерные запросы) - это революционная функция в веб-дизайне, которая позволяет создавать действительно независимые и адаптивные компоненты. В отличие от традиционных [[CSS Media Queries]], которые реагируют на размер окна браузера, Container Queries позволяют элементам адаптироваться к размеру их непосредственного родительского контейнера.

> [!info] Информация
> Container Queries особенно полезны в системах компонентов, где один и тот же элемент может использоваться в разных контекстах с разными размерами.

## Основы Container Queries

### Что такое Container Queries

Container Queries позволяют применять стили к элементу на основе характеристик его родительского контейнера, а не характеристик всего окна браузера. Это дает разработчикам возможность создавать по-настоящему независимые и переиспользуемые компоненты, которые адаптируются к доступному пространству независимо от их расположения на странице.

### Проблема, которую решают Container Queries

Традиционные [[CSS Media Queries]] имеют ограничения:

- Они зависят от размера области просмотра (viewport), а не от размера родительского контейнера
- Один и тот же компонент может выглядеть по-разному в зависимости от его расположения на странице
- Сложно создавать универсальные компоненты, которые адаптируются к своему окружению

Container Queries решают эти проблемы, позволяя компонентам адаптироваться к своему непосредственному окружению.

## Синтаксис и основные свойства

### Определение контейнера

Для создания контейнера используются следующие свойства:

- `container-type`: определяет тип контейнера
- `container-name`: задает имя контейнера (опционально)

```css
.card-container {
  container-type: inline-size;
  container-name: card-container; /* опционально */
}
```

### Типы контейнеров

| Тип | Описание | Использование |
|-----|----------|---------------|
| `inline-size` | Реагирует на размер вдоль строки (ширина в LTR) | Наиболее распространенный тип |
| `block-size` | Реагирует на размер поперек строки (высота в LTR) | Для адаптации по высоте |
| `size` | Реагирует на оба размера | Когда нужно учитывать ширину и высоту |
| `normal` | Отменяет поведение контейнера | Для отключения контейнера |

### Использование запросов

```css
@container card-container (min-width: 300px) {
  .card-content {
    font-size: 16px;
  }
}

@container (max-height: 200px) {
  .card-image {
    display: none;
  }
}
```

## Сравнение с Media Queries

| Критерий | Media Queries | Container Queries |
|----------|---------------|-------------------|
| Основа | Размер области просмотра | Размер родительского контейнера |
| Область применения | Вся страница | Конкретный элемент и его потомки |
| Гибкость | Ограниченная | Высокая |
| Повторное использование компонентов | Сложное | Простое и эффективное |
| Контекст зависимость | Зависит от viewport | Зависит от родительского контейнера |

## Практические примеры использования

### Пример 1: Адаптивная карточка продукта

```css
.product-card-container {
  container-type: inline-size;
  width: 100%;
  max-width: 400px;
  margin: 1rem;
}

.product-card {
  display: grid;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: white;
}

/* Начальный мобильный вид */
.product-card {
  grid-template-columns: 1fr;
  text-align: center;
}

/* Промежуточный вид - изображение и текст */
@container (min-width: 250px) {
  .product-card {
    grid-template-columns: auto 1fr;
    text-align: left;
  }
  
  .product-image {
    grid-row: span 2;
  }
}

/* Полноценный вид с кнопками */
@container (min-width: 350px) {
  .product-card {
    grid-template-columns: auto 1fr auto;
  }
  
  .product-actions {
    grid-column: 2 / -1;
    display: flex;
    justify-content: flex-end;
    gap: 0.5rem;
  }
}

/* Вид с увеличенными элементами */
@container (min-width: 450px) {
  .product-card {
    padding: 1.5rem;
    gap: 1.5rem;
  }
  
  .product-title {
    font-size: 1.25rem;
  }
  
  .product-price {
    font-size: 1.1rem;
  }
}
```

### Пример 2: Адаптивное меню навигации

```css
.nav-container {
  container-type: inline-size;
  background: #f5f5f5;
  padding: 0.5rem;
  border-radius: 4px;
}

.nav-list {
  display: flex;
  flex-direction: column;
  list-style: none;
  padding: 0;
  margin: 0;
  gap: 0.25rem;
}

.nav-item {
  padding: 0.5rem;
  border-radius: 4px;
  background: white;
}

.nav-link {
  text-decoration: none;
  color: #333;
}

/* Начальный мобильный вид */
@container (min-width: 200px) {
  .nav-list {
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    gap: 0.5rem;
  }
}

/* Средний размер - более равномерное распределение */
@container (min-width: 350px) {
  .nav-list {
    justify-content: space-between;
  }
  
  .nav-item {
    flex: 1;
    text-align: center;
  }
}

/* Большой размер - горизонтальное меню */
@container (min-width: 500px) {
  .nav-list {
    flex-direction: row;
    justify-content: flex-start;
  }
  
  .nav-item {
    flex: none;
    margin-right: 1rem;
  }
  
  .nav-item:last-child {
    margin-right: 0;
  }
}
```

### Пример 3: Адаптивная галерея изображений

```css
.gallery-container {
  container-type: inline-size;
  container-name: gallery;
}

.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  padding: 1rem;
}

.gallery-item {
  aspect-ratio: 1;
  border-radius: 8px;
  overflow: hidden;
  background: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.gallery-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Увеличение размера элементов галереи */
@container gallery (min-width: 400px) {
  .gallery-grid {
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  }
}

@container gallery (min-width: 600px) {
  .gallery-grid {
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  }
}

/* Дополнительные стили для больших контейнеров */
@container gallery (min-width: 800px) {
  .gallery-grid {
    grid-template-columns: repeat(4, 1fr);
  }
  
  .gallery-item {
    aspect-ratio: 4/3;
  }
}
```

## Продвинутые техники

### Вложенные контейнеры

```css
.outer-container {
  container-type: inline-size;
  container-name: outer;
}

.inner-container {
  container-type: inline-size;
  container-name: inner;
}

/* Стили для внешнего контейнера */
@container outer (min-width: 500px) {
  .outer-content {
    display: flex;
    gap: 2rem;
  }
}

/* Стили для внутреннего контейнера */
@container inner (min-width: 300px) {
  .inner-content {
    flex: 1;
  }
  
  .sidebar {
    flex: 0 0 200px;
  }
}
```

### Использование с CSS Grid и Flexbox

```css
.responsive-grid {
  container-type: inline-size;
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

@container (min-width: 400px) {
  .responsive-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@container (min-width: 700px) {
  .responsive-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@container (min-width: 1000px) {
  .responsive-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

.grid-item {
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* Адаптация содержимого элемента */
@container (min-width: 250px) {
  .grid-item {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  
  .item-content {
    text-align: center;
  }
}
```

### Использование с CSS Custom Properties

```css
.card-container {
  container-type: inline-size;
  container-name: card;
  --card-padding: 1rem;
  --font-size: 1rem;
  --border-radius: 4px;
}

.adaptive-card {
  padding: var(--card-padding);
  font-size: var(--font-size);
  border-radius: var(--border-radius);
  border: 1px solid #ddd;
  background: white;
}

@container card (min-width: 300px) {
  .card-container {
    --card-padding: 1.5rem;
    --font-size: 1.1rem;
    --border-radius: 6px;
  }
}

@container card (min-width: 500px) {
  .card-container {
    --card-padding: 2rem;
    --font-size: 1.2rem;
    --border-radius: 8px;
  }
}
```

## Поддержка в браузерах

Container Queries имеют хорошую поддержку в современных браузерах:

- Chrome 105+
- Firefox 110+
- Safari 16+
- Edge 105+

Для проверки поддержки можно использовать:

```css
@supports (container-type: inline-size) {
  .container {
    container-type: inline-size;
  }
}

@supports not (container-type: inline-size) {
  /* Резервные стили для старых браузеров */
  .container {
    display: flex;
    flex-direction: column;
  }
  
  @media (min-width: 600px) {
    .container {
      flex-direction: row;
    }
  }
}
```

## Лучшие практики

### 1. Используйте именованные контейнеры

Именованные контейнеры улучшают читаемость кода и упрощают отладку:

```css
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
}

@container sidebar (min-width: 300px) {
  .sidebar-content {
    display: flex;
  }
}
```

### 2. Оптимизация производительности

- Избегайте чрезмерного вложения контейнеров
- Используйте разумные точки останова
- Минимизируйте перерисовку при изменении размера
- Используйте `contain` для изоляции изменений

```css
.optimized-container {
  container-type: inline-size;
  contain: layout style paint;
}
```

### 3. Комбинирование с современными системами раскладки

Container Queries особенно эффективны при использовании с Grid и Flexbox:

```css
.modern-layout {
  container-type: inline-size;
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

@container (min-width: 600px) {
  .modern-layout {
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  }
}
```

### 4. Обработка граничных случаев

```css
.fallback-container {
  /* Резервные стили */
  display: block;
}

@supports (container-type: inline-size) {
  .fallback-container {
    container-type: inline-size;
  }
  
  @container (min-width: 400px) {
    .fallback-container {
      display: flex;
    }
  }
}
```

## Совместное использование с другими CSS функциями

### Container Queries и CSS Grid

```css
.grid-container {
  container-type: inline-size;
  display: grid;
  grid-template-columns: 1fr;
  grid-auto-rows: minmax(100px, auto);
  gap: 1rem;
}

@container (min-width: 400px) {
  .grid-container {
    grid-template-columns: repeat(2, 1fr);
  }
}

@container (min-width: 700px) {
  .grid-container {
    grid-template-columns: repeat(3, 1fr);
  }
}

@container (min-width: 1000px) {
  .grid-container {
    grid-template-columns: repeat(4, 1fr);
  }
}

.grid-item {
  background: #f9f9f9;
  padding: 1rem;
  border-radius: 4px;
  border: 1px solid #eee;
}
```

### Container Queries и Flexbox

```css
.flex-container {
  container-type: inline-size;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.flex-item {
  flex: 1;
  padding: 1rem;
  background: #f0f0f0;
  border-radius: 4px;
}

@container (min-width: 400px) {
  .flex-container {
    flex-direction: row;
  }
  
  .flex-item {
    flex: 1;
  }
}

@container (min-width: 700px) {
  .flex-container {
    flex-wrap: wrap;
  }
  
  .flex-item {
    flex: 0 0 calc(50% - 0.5rem);
  }
}
```

## Проблемы и ограничения

### 1. Циклические зависимости

Избегайте ситуаций, когда размер контейнера зависит от содержимого, которое изменяется на основе размера контейнера:

```css
/* ПЛОХО - может вызвать циклическую зависимость */
.bad-container {
  container-type: inline-size;
  width: min-content; /* Размер зависит от содержимого */
}

@container (min-width: 500px) {
  .bad-content {
    width: 600px; /* Содержимое влияет на размер контейнера */
  }
}
```

### 2. Производительность

Частые изменения размеров контейнера могут повлиять на производительность. Используйте оптимизацию:

```css
.performance-container {
  container-type: inline-size;
  contain: layout style paint;
  container-name: perf-container;
}
```

### 3. Поддержка в старых браузерах

Всегда предоставляйте резервные стили для браузеров без поддержки Container Queries:

```css
.responsive-component {
  /* Резервные стили */
  display: block;
  padding: 1rem;
}

@supports (container-type: inline-size) {
  .responsive-component-container {
    container-type: inline-size;
  }
  
  @container (min-width: 400px) {
    .responsive-component {
      display: flex;
      align-items: center;
    }
  }
}
```

## Примеры из реальных проектов

### Адаптивный виджет погоды

```css
.weather-widget-container {
  container-type: inline-size;
  container-name: weather-widget;
}

.weather-widget {
  background: linear-gradient(135deg, #74b9ff, #0984e3);
  color: white;
  border-radius: 12px;
  overflow: hidden;
  padding: 1rem;
}

.weather-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.weather-main {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.weather-icon {
  font-size: 3rem;
  margin-bottom: 0.5rem;
}

.weather-temp {
  font-size: 2rem;
  font-weight: bold;
}

.weather-details {
  display: flex;
  justify-content: space-around;
  width: 100%;
  margin-top: 1rem;
}

@container weather-widget (min-width: 300px) {
  .weather-main {
    flex-direction: row;
    align-items: center;
  }
  
  .weather-icon {
    margin-right: 1rem;
    margin-bottom: 0;
  }
  
  .weather-details {
    justify-content: flex-start;
    gap: 1rem;
  }
}

@container weather-widget (min-width: 400px) {
  .weather-temp {
    font-size: 2.5rem;
  }
  
  .weather-details {
    flex-direction: column;
    align-items: flex-start;
  }
}
```

### Адаптивная карточка статьи

```css
.article-card-container {
  container-type: inline-size;
  container-name: article-card;
}

.article-card {
  background: white;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.article-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.15);
}

.article-image {
  width: 100%;
  aspect-ratio: 16/9;
  object-fit: cover;
}

.article-content {
  padding: 1rem;
}

.article-title {
  font-size: 1.2rem;
  margin: 0 0 0.5rem 0;
  color: #333;
}

.article-excerpt {
  color: #666;
  margin-bottom: 1rem;
}

.article-meta {
  display: flex;
  justify-content: space-between;
  color: #999;
  font-size: 0.9rem;
}

@container article-card (min-width: 350px) {
  .article-card {
    display: flex;
    flex-direction: column;
  }
  
  .article-image {
    aspect-ratio: 4/3;
  }
}

@container article-card (min-width: 500px) {
  .article-card {
    flex-direction: row;
    max-height: 200px;
  }
  
  .article-image {
    width: 40%;
    aspect-ratio: auto;
  }
  
  .article-content {
    width: 60%;
  }
  
  .article-title {
    font-size: 1.3rem;
  }
}
```

## Будущие развития и экспериментальные возможности

### Container Units (экспериментальные)

В будущем планируется реализация специальных единиц измерения, связанных с контейнерами:

- `cqw`: 1% ширины контейнера
- `cqh`: 1% высоты контейнера
- `cqi`: 1% размера вдоль оси элемента
- `cqb`: 1% размера поперек оси элемента
- `cqmin`: минимальное из `cqw` и `cqh`
- `cqmax`: максимальное из `cqw` и `cqh`

```css
/* Пример использования (когда будет реализовано) */
.responsive-element {
  width: 50cqw; /* 50% ширины контейнера */
  height: 30cqh; /* 30% высоты контейнера */
}
```

### Динамические имена контейнеров

В будущем планируется поддержка динамических имен контейнеров через CSS-переменные, что позволит создавать более гибкие системы компонентов.

## Отладка Container Queries

### Инструменты разработчика

Большинство современных браузеров предоставляют инструменты для отладки Container Queries:

1. В инструментах разработчика можно увидеть, какие контейнеры определены
2. Можно проверить, срабатывают ли запросы при изменении размера
3. Доступна информация о размерах контейнеров

### Тестирование производительности

Для тестирования производительности используйте инструменты профилирования:

```css
.performance-test {
  container-type: inline-size;
  contain: layout style paint;
  /* Добавьте дополнительные стили для тестирования */
}
```

## Сравнение с другими подходами

### Container Queries vs Media Queries

| Особенность | Media Queries | Container Queries |
|-------------|---------------|-------------------|
| Контекст | Вся страница | Конкретный контейнер |
| Гибкость | Ограниченная | Высокая |
| Повторное использование | Сложное | Простое |
| Зависимость от viewport | Да | Нет |
| Использование в компонентах | Ограниченное | Идеальное |

### Container Queries vs JavaScript решения

| Критерий | JavaScript | Container Queries |
|----------|------------|-------------------|
| Производительность | Ниже | Выше |
| Сложность | Выше | Ниже |
| Поддержка | Универсальная | Современные браузеры |
| Масштабируемость | Ограниченная | Высокая |

## Заключение

CSS Container Queries открывают новые возможности для создания гибких и переиспользуемых компонентов. Они позволяют разработчикам создавать действительно адаптивные элементы, которые реагируют на размеры их родительского контейнера, а не на размеры области просмотра. Это особенно полезно в системах компонентов, где один и тот же компонент может использоваться в разных контекстах с разными размерами.

Container Queries представляют собой важный шаг вперед в веб-дизайне, позволяя создавать более гибкие и независимые компоненты. Хотя поддержка браузерами все еще развивается, использование Container Queries уже сейчас позволяет создавать более современные и адаптивные интерфейсы.

## Связанные темы

- [[CSS Media Queries]] - традиционный подход к адаптивности
- [[CSS Grid]] - современная система раскладки
- [[CSS Flexbox]] - гибкая система раскладки
- [[CSS Responsive Design]] - общие принципы адаптивного дизайна
- [[CSS Layout]] - системы раскладки, используемые с Container Queries
- [[CSS Custom Properties]] - переменные CSS для динамических стилей
- [[CSS Containment]] - свойство contain для оптимизации производительности
- [[CSS Aspect Ratio]] - соотношение сторон элементов
- [[CSS Clamp Function]] - динамические значения размеров
- [[CSS Viewport Units]] - единицы измерения, основанные на размере окна

#css #container-queries #responsive-design #web-development #frontend #modern-css
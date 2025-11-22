---
aliases: [Контейнерные запросы CSS, Контейнерные условия CSS]
tags: [css, container-queries, modern-features, frontend]
---

# CSS Container Queries - Современный подход к адаптивному дизайну

## Введение в CSS Container Queries

CSS Container Queries (контейнерные запросы) - это современная функция CSS, которая позволяет создавать компоненты, адаптирующиеся к своим родительским контейнерам, а не к размеру вьюпорта. Эта технология открывает новые возможности для создания гибких и переиспользуемых компонентов, которые могут адаптироваться к различным размерам контейнеров независимо от общего размера страницы.

Контейнерные запросы решают проблему традиционных медиа-запросов, которые зависят от размера окна браузера, а не от реального пространства, доступного для компонента. Это особенно важно в современном веб-дизайне, где компоненты могут появляться в разных контекстах и размерах.

## Основы Container Queries

### Синтаксис и структура

Контейнерные запросы используют специальный синтаксис, аналогичный медиа-запросам, но вместо `@media` используют `@container`:

```css
@container [name] (min-width: 300px) {
  /* Стили применяются, когда контейнер имеет ширину не менее 300px */
  .component {
    font-size: 1.2rem;
  }
}
```

В отличие от медиа-запросов, контейнерные запросы проверяют размеры родительского контейнера, а не вьюпорта. Это позволяет создавать компоненты, которые адаптируются к доступному пространству независимо от того, где они находятся на странице.

### Определение контейнера

Чтобы использовать контейнерные запросы, необходимо сначала определить элемент как контейнер, используя свойство `container`:

```css
.container {
  container-type: inline-size;
  container-name: sidebar;
}
```

Или в сокращённой форме:

```css
.container {
  container: sidebar / inline-size;
}
```

### Типы контейнеров

Существует несколько типов контейнеров:

- `inline-size` - проверяет размер в направлении текста (ширина в горизонтальных письмах)
- `block-size` - проверяет размер в перпендикулярном направлении (высота в горизонтальных письмах)
- `size` - проверяет оба измерения (эквивалентно `inline-size block-size`)
- `normal` - отключает контейнер

## Практическое применение Container Queries

### Простой пример адаптивного компонента

Рассмотрим карточку продукта, которая должна адаптироваться к размеру контейнера:

```css
.card {
  display: flex;
  flex-direction: column;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.card img {
  width: 100%;
  height: auto;
}

.card h3 {
  margin: 0.5rem 0;
}

.card p {
  margin: 0.5rem 0;
  color: #666;
}

/* Контейнерная проверка */
@container (min-width: 300px) {
  .card {
    flex-direction: row;
    align-items: center;
  }
  
  .card img {
    width: 50%;
    flex-shrink: 0;
  }
  
  .card-content {
    padding-left: 1rem;
    width: 50%;
  }
}

@container (min-width: 500px) {
  .card {
    padding: 1.5rem;
  }
  
  .card h3 {
    font-size: 1.25rem;
  }
}
```

Этот подход позволяет карточке адаптироваться к размеру контейнера, в котором она находится, независимо от размера экрана или вьюпорта.

### Использование именованных контейнеров

Именованные контейнеры позволяют создавать более точные запросы:

```css
.sidebar {
  container: sidebar / inline-size;
  width: 250px;
}

.main-content {
  container: main / inline-size;
  width: 700px;
}

@container sidebar (min-width: 300px) {
  .sidebar-component {
    font-size: 1rem;
  }
}

@container main (min-width: 600px) {
  .main-component {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
}
```

### Комбинация с другими CSS-функциями

Контейнерные запросы могут быть эффективно объединены с другими современными CSS-функциями:

```css
.card-container {
  container-type: inline-size;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

@container (min-width: 400px) {
  .card {
    display: flex;
    flex-direction: row;
  }
}

@container (min-width: 600px) {
  .card {
    flex-direction: column;
  }
  
  .card-image {
    aspect-ratio: 16/9;
  }
}
```

## Продвинутые техники

### Вложенные контейнерные запросы

Контейнерные запросы могут быть вложенными, что позволяет создавать сложные адаптивные структуры:

```css
.outer-container {
  container: outer / inline-size;
  display: flex;
  flex-wrap: wrap;
}

.inner-container {
  container: inner / inline-size;
  flex: 1;
  min-width: 200px;
}

@container outer (min-width: 600px) {
  .outer-container {
    flex-direction: row;
  }
  
  @container inner (min-width: 300px) {
    .inner-component {
      padding: 2rem;
    }
  }
}
```

### Условия для разных измерений

Можно создавать запросы как для ширины, так и для высоты контейнера:

```css
.grid-container {
  container-type: size;
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-auto-rows: 100px;
}

@container (min-width: 400px) {
  .grid-container {
    grid-template-columns: repeat(3, 1fr);
  }
}

@container (min-height: 300px) {
  .grid-container {
    grid-auto-rows: 150px;
  }
}
```

## Лучшие практики использования Container Queries

### 1. Определение подходящих компонентов

Не все компоненты подходят для использования контейнерных запросов. Идеальными кандидатами являются:

- Карточки товаров
- Новостные блоки
- Галереи изображений
- Компоненты навигации
- Блоки с контентом переменной ширины

### 2. Использование семантических имен

При определении именованных контейнеров используйте семантически значимые имена:

```css
/* Хорошо */
.product-grid {
  container: products / inline-size;
}

.article-sidebar {
  container: sidebar / inline-size;
}

/* Не рекомендуется */
.container-1 {
  container: c1 / inline-size;
}
```

### 3. Постепенное улучшение

Всегда начинайте с базового стиля, который работает без контейнерных запросов, а затем добавляйте улучшения:

```css
.card {
  /* Базовый стиль */
  padding: 1rem;
  display: flex;
  flex-direction: column;
}

/* Улучшения с контейнерными запросами */
@container (min-width: 300px) {
  .card {
    flex-direction: row;
  }
}
```

### 4. Оптимизация производительности

- Избегайте чрезмерного вложения контейнерных запросов
- Используйте разумные пороговые значения
- Рассмотрите использование `contain: layout style` для улучшения производительности

## Совместимость и поддержка

На момент 2023 года контейнерные запросы поддерживаются в современных браузерах:

- Chrome 105+
- Edge 105+
- Firefox 110+
- Safari 16+

Для обеспечения совместимости с более старыми браузерами рекомендуется использовать прогрессивное улучшение.

## Практические примеры

### Адаптивная галерея

```css
.gallery {
  container-type: inline-size;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 1rem;
  padding: 1rem;
}

.gallery-item {
  aspect-ratio: 1;
  overflow: hidden;
  border-radius: 8px;
}

@container (min-width: 500px) {
  .gallery {
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  }
}

@container (min-width: 800px) {
  .gallery {
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  }
}
```

### Адаптивная карточка профиля

```css
.profile-card {
  container-type: inline-size;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 1.5rem;
  border: 1px solid #ddd;
  border-radius: 12px;
  text-align: center;
}

.profile-avatar {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  margin-bottom: 1rem;
}

@container (min-width: 350px) {
  .profile-card {
    flex-direction: row;
    text-align: left;
    align-items: flex-start;
  }
  
  .profile-avatar {
    margin-right: 1.5rem;
    margin-bottom: 0;
  }
  
  .profile-info {
    flex: 1;
  }
}
```

## Сравнение с традиционными медиа-запросами

| Характеристика | Медиа-запросы | Контейнерные запросы |
|----------------|---------------|----------------------|
| Основа проверки | Размер вьюпорта | Размер родительского контейнера |
| Гибкость | Ограниченная | Высокая |
| Повторное использование компонентов | Сложное | Простое |
| Зависимость от контекста | Высокая | Низкая |
| Поддержка браузерами | Широкая | Современные браузеры |

## Потенциальные проблемы и решения

### 1. Циклические зависимости

Избегайте ситуаций, когда размер контейнера зависит от размера его содержимого, а размер содержимого зависит от размера контейнера:

```css
/* Избегайте этого */
.container {
  width: fit-content;
  container-type: inline-size;
}

@container (min-width: 300px) {
  .container {
    width: 400px; /* Это может создать циклическую зависимость */
  }
}
```

### 2. Производительность

Контейнерные запросы могут вызывать перерисовку при изменении размеров контейнеров, что может повлиять на производительность. Для минимизации эффекта:

- Используйте `contain` свойства для изоляции изменений
- Избегайте частых изменений размеров контейнеров
- Оптимизируйте сложные стили внутри контейнерных запросов

## Рекомендации по архитектуре

### Структура файлов

При использовании контейнерных запросов рекомендуется организовать CSS следующим образом:

```
styles/
├── components/
│   ├── card.css
│   ├── gallery.css
│   └── profile.css
├── containers/
│   ├── named-containers.css
│   └── utilities.css
└── main.css
```

### Модульность

Каждый компонент должен быть самодостаточным и включать свои контейнерные запросы:

```css
/* components/card.css */
.card {
  container-type: inline-size;
  padding: 1rem;
  border: 1px solid #ccc;
}

@container (min-width: 300px) {
  .card {
    display: flex;
    padding: 1.5rem;
  }
}
```

## Будущее Container Queries

Контейнерные запросы - это часть более широкой инициативы CSS Working Group по созданию более гибких и компонентных подходов к веб-дизайну. В будущем можно ожидать:

- Расширения функциональности (например, запросы по другим свойствам)
- Лучшей поддержки инструментов разработки
- Интеграции с фреймворками и библиотеками компонентов

## Заключение

CSS Container Queries представляют собой важный шаг вперед в создании адаптивных веб-компонентов. Они позволяют разработчикам создавать по-настоящему гибкие компоненты, которые могут адаптироваться к различным контекстам без жесткой зависимости от размеров вьюпорта.

Понимание и правильное использование контейнерных запросов открывает новые возможности для создания современных, гибких и переиспользуемых интерфейсов. При правильной реализации они могут значительно упростить создание адаптивных компонентов и улучшить пользовательский опыт.

Для успешного внедрения контейнерных запросов важно понимать не только синтаксис, но и лучшие практики, ограничения и потенциальные проблемы. С правильным подходом контейнерные запросы становятся мощным инструментом в арсенале современного фронтенд-разработчика.

## См. также

- [[CSS Media Queries]]
- [[CSS Grid Layout]]
- [[CSS Flexbox]]
- [[CSS Containment]]
- [[Responsive Web Design]]
- [[CSS Custom Properties]]
- [[Modern CSS Methodologies]]
- [[CSS Architecture Patterns]]
- [[Frontend Performance Optimization]]
- [[Component-Based Architecture]]
- [[CSS Preprocessors Comparison]]
- [[CSS Frameworks]]
- [[CSS Methodologies]]
- [[CSS Selectors]]
- [[CSS Specificity]]

## Дополнительные ресурсы

- [MDN Web Docs: Container Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Container_Queries)
- [CSS Working Group: Container Queries Specification](https://www.w3.org/TR/css-contain-3/)
- [Web.dev: Learn Container Queries](https://web.dev/learn/css/container-queries/)

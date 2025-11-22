---
aliases: [Container Queries, CSS Container Queries, Контейнерные Запросы]
tags: [css, advanced, responsive, layout]
---

# CSS Container Queries - Полное руководство

## Введение

Container Queries - это новая функция CSS, которая позволяет создавать компоненты, адаптирующиеся к размеру их собственного контейнера, а не к размеру области просмотра (viewport). Эта функция решает многие проблемы, с которыми сталкиваются разработчики при создании действительно гибких и переиспользуемых компонентов.

## Проблема, которую решают Container Queries

Традиционные медиазапросы (Media Queries) зависят от глобальных значений, таких как размер области просмотра, что создает ограничения при разработке независимых компонентов. Например, карточка может выглядеть идеально на широком экране, но стать нечитаемой, если будет размещена в узком сайдбаре. Container Queries позволяют компоненту адаптироваться к доступному ему пространству, независимо от общего размера окна браузера.

## Синтаксис Container Queries

### Container Type (Тип контейнера)

Чтобы компонент мог отвечать на изменения размера своего родительского контейнера, необходимо сначала определить, какой элемент будет контейнером, используя свойство `container`:

```css
.card-container {
  container-type: inline-size;
  container-name: card;
}
```

- `container-type: inline-size` - определяет, что контейнер будет реагировать только на изменения вдоль направления строк (ширина в горизонтальных письмах)
- `container-type: size` - определяет, что контейнер будет реагировать на изменения как ширины, так и высоты
- `container-name: card` - опциональное имя, которое можно использовать для более точной настройки

### Querying Containers

После определения контейнера, можно использовать `@container` для написания стилей, которые применяются в зависимости от размера контейнера:

```css
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }
}
```

## Практические примеры

### Адаптивная карточка товара

Рассмотрим пример карточки товара, которая изменяет макет в зависимости от доступного пространства:

```html
<div class="card-container">
  <div class="card">
    <img src="product.jpg" alt="Product" class="card-image">
    <div class="card-content">
      <h3>Название товара</h3>
      <p>Описание товара</p>
      <button class="buy-btn">Купить</button>
    </div>
  </div>
</div>
```

```css
.card-container {
  container-type: inline-size;
  width: 100%;
  max-width: 400px; /* или динамическая ширина */
}

/* Стили по умолчанию для узкого вида */
.card {
  display: flex;
  flex-direction: column;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 1rem;
  gap: 0.5rem;
}

.card-image {
  width: 100%;
  height: auto;
  border-radius: 4px;
}

/* Изменение макета при достаточном пространстве */
@container (min-width: 350px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
    align-items: center;
    gap: 1rem;
  }
  
  .card-image {
    grid-row: span 2;
  }
}
```

### Адаптивный список компонентов

Вот пример более сложного сценария, где контейнер может вместить разное количество элементов:

```css
.component-grid {
  container-type: inline-size;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  padding: 1rem;
}

@container (min-width: 500px) {
  .component-grid {
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  }
}

@container (min-width: 800px) {
  .component-grid {
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  }
}

.component-card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 1rem;
}

@container (min-width: 200px) {
  .component-card .card-title {
    font-size: 1.2rem;
  }
}

@container (min-width: 300px) {
  .component-card .card-title {
    font-size: 1.5rem;
  }
  
  .component-card .card-content {
    display: block;
  }
}
```

## Использование именованных контейнеров

Можно использовать имена контейнеров для более точного управления стилями:

```css
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
  width: 20%;
}

.main-content {
  container-type: inline-size;
  container-name: main;
  width: 80%;
}

/* Применяется только к элементам внутри контейнера с именем 'sidebar' */
@container sidebar (min-width: 200px) {
  .navigation {
    flex-direction: column;
  }
}

/* Применяется только к элементам внутри контейнера с именем 'main' */
@container main (min-width: 600px) {
  .article {
    display: grid;
    grid-template-columns: 3fr 1fr;
  }
}
```

## Свойства Container Queries

### container-type

- `inline-size` - реагирует на изменения вдоль основного направления текста
- `block-size` - реагирует на изменения в перпендикулярном направлении
- `size` - реагирует на изменения в обоих направлениях
- `normal` - отключает контейнер

### container-name

- Определяет имя контейнера для последующего использования в `@container`
- Можно задать несколько имен: `container-name: sidebar, navigation;`

### Сокращенное свойство container

Можно использовать сокращенное свойство `container`:

```css
.card-container {
  container: card / inline-size;
}
```

## Ограничения и особенности

### Вложенные Container Queries

Container Queries можно вкладывать друг в друга:

```css
@container card (min-width: 300px) {
  @container (min-height: 200px) {
    .card-content {
      display: grid;
      grid-template-rows: auto 1fr auto;
    }
  }
}
```

### Взаимодействие с Media Queries

Container Queries и Media Queries могут использоваться вместе:

```css
/* Сначала медиазапрос по области просмотра */
@media (min-width: 768px) {
  /* Затем контейнерный запрос */
  @container card (min-width: 400px) {
    .card {
      font-size: 1.1rem;
    }
  }
}
```

## Производительность и рекомендации

### Оптимизация производительности

- Избегайте чрезмерно вложенных Container Queries, так как они могут замедлить рендеринг
- Используйте их разумно и только там, где традиционные Media Queries неэффективны
- Используйте `content-visibility` для оптимизации рендеринга больших списков

### Лучшие практики

1. Используйте Container Queries для создания действительно независимых компонентов
2. Сочетайте с Media Queries для полного контроля над адаптивностью
3. Тестируйте компоненты в различных контекстах и размерах контейнеров
4. Используйте именованные контейнеры для лучшей читаемости кода
5. Обеспечьте адекватные стили по умолчанию, когда контейнер слишком мал для применения запроса

## Поддержка браузерами

На момент 2023 года Container Queries поддерживаются в современных браузерах:
- Chrome 105+
- Firefox 110+
- Safari 16+

Для поддержки старых браузеров можно использовать JavaScript-полифиллы или резервные стили.

## Заключение

Container Queries - это мощная функция CSS, которая позволяет создавать более гибкие и адаптивные компоненты. Они особенно полезны при создании дизайн-систем и библиотек компонентов, где элементы могут размещаться в контекстах с различным доступным пространством. Используя их правильно, можно создавать более устойчивый и масштабируемый код.

## Смежные темы

- [[CSS-медиа-запросы]]
- [[responsive-design]]
- [[layout]]
- [[flexbox]]
- [[css-grid-layout]]

#css #container-queries #responsive-design #layout #frontend
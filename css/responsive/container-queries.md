# CSS Container Queries

## Что такое Container Queries и зачем они нужны

Container Queries (контейнерные запросы) — это мощная функция CSS, позволяющая применять стили к элементу на основе характеристик его родительского контейнера, а не характеристик всего окна браузера. Это позволяет создавать по-настоящему независимые и переиспользуемые компоненты, которые адаптируются к доступному пространству, независимо от их расположения на странице.

Традиционные [[CSS Media Queries]] ограничены размерами области просмотра (viewport), что затрудняет создание универсальных компонентов. Container Queries решают эту проблему, позволяя компонентам адаптироваться к своему непосредственному окружению.

## Отличие от Media Queries

| Критерий | Media Queries | Container Queries |
|----------|---------------|-------------------|
| Основа | Размер области просмотра | Размер родительского контейнера |
| Область применения | Вся страница | Конкретный элемент и его потомки |
| Гибкость | Ограниченная | Высокая |
| Повторное использование компонентов | Сложное | Простое и эффективное |

```css
/* Media Query */
@media (min-width: 600px) {
  .card {
    display: flex;
  }
}

/* Container Query */
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    display: flex;
  }
}
```

## Синтаксис и основные свойства

### Определение контейнера

Для создания контейнера используются следующие свойства:

- `container-type`: определяет тип контейнера
- `container-name`: задает имя контейнера (опционально)

```css
.card-container {
  container-type: inline-size; /* или size, normal */
  container-name: card-container; /* опционально */
}
```

### Типы контейнеров

- `inline-size`: реагирует на размер вдоль строки (ширина в LTR)
- `block-size`: реагирует на размер поперек строки (высота в LTR)
- `size`: реагирует на оба размера
- `normal`: отменяет поведение контейнера

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

## Практические примеры использования

### Пример 1: Адаптивная карточка

```css
.product-card-container {
  container-type: inline-size;
}

.product-card {
  display: grid;
  gap: 1rem;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 8px;
}

@container (min-width: 300px) {
  .product-card {
    grid-template-columns: auto 1fr;
  }
}

@container (min-width: 500px) {
  .product-card {
    grid-template-columns: auto 1fr auto;
  }
}
```

### Пример 2: Адаптивный список

```css
.nav-container {
  container-type: inline-size;
}

.nav-list {
  display: flex;
  flex-direction: column;
  list-style: none;
  padding: 0;
  margin: 0;
}

@container (min-width: 400px) {
  .nav-list {
    flex-direction: row;
  }
}

@container (min-width: 600px) {
  .nav-list {
    justify-content: space-around;
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
```

## Паттерны и лучшие практики

### 1. Используйте именованные контейнеры

```css
.sidebar {
  container-type: inline-size;
  container-name: sidebar;
}

@container sidebar (min-width: 300px) {
  /* Стили для боковой панели */
}
```

### 2. Оптимизация производительности

- Избегайте вложенных контейнеров без необходимости
- Используйте разумные точки останова
- Минимизируйте перерисовку при изменении размера

### 3. Комбинирование с Grid и Flexbox

Container Queries особенно эффективны при использовании с современными системами раскладки:

```css
.grid-container {
  container-type: inline-size;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

@container (min-width: 800px) {
  .grid-container {
    grid-template-columns: repeat(4, 1fr);
  }
}
```

## Совместное использование с Media Queries

Container Queries и Media Queries могут использоваться вместе для создания сложных адаптивных решений:

```css
/* Общие стили для мобильных устройств */
@media (max-width: 768px) {
  .main-content {
    padding: 1rem;
  }
}

.article-card-container {
  container-type: inline-size;
}

/* Стили на основе размера контейнера */
@container (min-width: 300px) {
  .article-card {
    display: flex;
    flex-direction: row;
  }
  
  .article-image {
    flex: 0 0 150px;
  }
}

@container (min-width: 500px) {
  .article-card {
    flex-direction: row-reverse;
  }
}
```

## Проблемы и ограничения

### 1. Циклические зависимости

Избегайте ситуаций, когда размер контейнера зависит от содержимого, которое изменяется на основе размера контейнера.

### 2. Поддержка в старых браузерах

Для поддержки старых браузеров используйте резервные стили:

```css
.card {
  /* Резервные стили для старых браузеров */
  display: block;
}

@supports (container-type: inline-size) {
  .card-container {
    container-type: inline-size;
  }
  
  @container (min-width: 400px) {
    .card {
      display: flex;
    }
  }
}
```

### 3. Производительность

Частые изменения размеров контейнера могут повлиять на производительность. Используйте `contain` для оптимизации:

```css
.optimized-container {
  container-type: inline-size;
  contain: layout style paint;
}
```

## Будущие развития и экспериментальные возможности

### Container Units

В будущем планируется реализация специальных единиц измерения, связанных с контейнерами:

- `cqw`: 1% ширины контейнера
- `cqh`: 1% высоты контейнера
- `cqi`: 1% размера вдоль оси элемента
- `cqb`: 1% размера поперек оси элемента

### Динамические имена контейнеров

Планируется поддержка динамических имен контейнеров через CSS-переменные.

## Связь с другими файлами в проекте

- [[CSS Media Queries]] - традиционный подход к адаптивности
- [[CSS Layout]] - системы раскладки, используемые с Container Queries
- [[CSS Responsive Design]] - общие принципы адаптивного дизайна
- [[CSS Grid]] - современная система раскладки
- [[CSS Flexbox]] - гибкая система раскладки

## Заключение

Container Queries открывают новые возможности для создания гибких и переиспользуемых компонентов. Они позволяют разработчикам создавать действительно адаптивные элементы, которые реагируют на размеры их родительского контейнера, а не на размеры области просмотра. Это особенно полезно в системах компонентов, где один и тот же компонент может использоваться в разных контекстах с разными размерами.

#css #container-queries #responsive #web-development #frontend
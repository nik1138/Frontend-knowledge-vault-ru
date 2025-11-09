# OOCSS

Этот файл описывает методологию OOCSS (Object-Oriented CSS).

OOCSS (Object-Oriented CSS) — это методология CSS, которая основывается на принципах объектно-ориентированного программирования. Она фокусируется на разделении структуры и оформления, а также на создании переиспользуемых компонентов.

## Принципы OOCSS:

1. **Разделение структуры и оформления**:
   - Структурные свойства (размеры, позиционирование)
   - Визуальные свойства (цвета, шрифты, границы)

2. **Переиспользуемые компоненты**:
   - Создание независимых объектов
   - Комбинирование объектов для создания более сложных элементов

## Пример:

```css
/* Структурный класс */
.media { display: flex; }
.media-object { flex-shrink: 0; }
.media-body { flex: 1; }

/* Визуальные классы */
.bg-blue { background-color: blue; }
.pad-20 { padding: 20px; }
```

```html
<div class="media bg-blue pad-20">
  <img class="media-object" src="image.jpg" alt="Image">
  <div class="media-body">Content</div>
</div>
```

## Преимущества:

- Повышенная переиспользуемость
- Меньший объем CSS
- Более быстрая разработка

## См. также:
- [[methodology]]
- [[SMACSS]]
- [[BEM]]

#oocss #methodology #object-oriented #web-development
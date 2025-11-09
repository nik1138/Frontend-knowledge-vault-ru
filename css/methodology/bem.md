# БЭМ

Этот файл описывает методологию БЭМ (Блок-Элемент-Модификатор).

БЭМ (Блок-Элемент-Модификатор) — это методология CSS, разработанная Яндексом. Она позволяет создавать переиспользуемые компоненты и обеспечивает гибкость в изменении структуры HTML без необходимости переписывать CSS.

## Принципы БЭМ:

1. **Блок**: Независимый компонент интерфейса (например, header, menu, button)
2. **Элемент**: Часть блока, не имеющая самостоятельного смысла (например, menu-item, header-title)
3. **Модификатор**: Сущность, определяющая внешний вид, состояние или поведение блока/элемента (например, button_size_large, menu_type_horizontal)

## Пример:

```html
<div class="card card_theme_dark">
  <h2 class="card__title">Заголовок</h2>
  <p class="card__text card__text_size_small">Текст</p>
  <button class="card__button card__button_disabled">Кнопка</button>
</div>
```

```css
.card { }
.card_theme_dark { }
.card__title { }
.card__text { }
.card__text_size_small { }
.card__button { }
.card__button_disabled { }
```

## Преимущества:

- Повышенная переиспользуемость
- Лучшая читаемость
- Лучшая поддерживаемость
- Ясная структура

## См. также:
- [[methodology]]
- [[bem]]
- [[methodology/bem]]

#bem #methodology #yandex #web-development
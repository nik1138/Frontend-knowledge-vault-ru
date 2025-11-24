---
aliases: [Flex-контейнеры, Контейнеры Flexbox]
tags: [css, flexbox, layout, frontend]
---

# Контейнеры Flex

## Общее описание

Flex-контейнер - это элемент, к которому применено свойство `display: flex`. Все непосредственные дочерние элементы этого контейнера становятся flex-элементами и подчиняются правилам flex-макета. Контейнер управляет расположением, выравниванием и размерами своих дочерних элементов.

## Основные свойства контейнера

### display

Свойство `display: flex` превращает элемент в flex-контейнер. Это основное свойство, с которого начинается использование flexbox.

```css
.container {
  display: flex;
}
```

### flex-direction

Определяет направление главной оси, по которой будут размещаться flex-элементы.

- `row` - элементы располагаются по горизонтали слева направо (значение по умолчанию)
- `row-reverse` - элементы располагаются по горизонтали справа налево
- `column` - элементы располагаются по вертикали сверху вниз
- `column-reverse` - элементы располагаются по вертикали снизу вверх

```css
.container {
  display: flex;
  flex-direction: row; /* или row-reverse, column, column-reverse */
}
```

### justify-content

Определяет выравнивание flex-элементов по главной оси.

- `flex-start` - элементы прижаты к началу контейнера
- `flex-end` - элементы прижаты к концу контейнера
- `center` - элементы выравниваются по центру
- `space-between` - элементы равномерно распределены; первый элемент прижат к началу, последний к концу
- `space-around` - элементы равномерно распределены с равными отступами вокруг
- `space-evenly` - элементы равномерно распределены с равными отступами между ними

```css
.container {
  display: flex;
  justify-content: center; /* или flex-start, flex-end, space-between, и т.д. */
}
```

### align-items

Определяет выравнивание flex-элементов по поперечной оси.

- `stretch` - элементы растягиваются по высоте контейнера (по умолчанию)
- `flex-start` - элементы прижаты к началу поперечной оси
- `flex-end` - элементы прижаты к концу поперечной оси
- `center` - элементы выравниваются по центру
- `baseline` - элементы выравниваются по базовой линии текста

```css
.container {
  display: flex;
  align-items: center; /* или stretch, flex-start, flex-end, baseline */
}
```

### flex-wrap

Определяет, должны ли flex-элементы переноситься на новые строки.

- `nowrap` - элементы не переносятся (значение по умолчанию)
- `wrap` - элементы переносятся сверху вниз
- `wrap-reverse` - элементы переносятся снизу вверх

```css
.container {
  display: flex;
  flex-wrap: wrap; /* или nowrap, wrap-reverse */
}
```

### align-content

Определяет выравнивание строк при нескольких строках flex-элементов (работает только при наличии нескольких строк).

- `flex-start` - строки прижаты к началу контейнера
- `flex-end` - строки прижаты к концу контейнера
- `center` - строки выравниваются по центру
- `space-between` - строки равномерно распределены
- `space-around` - строки равномерно распределены с отступами
- `stretch` - строки растягиваются равномерно (по умолчанию)

```css
.container {
  display: flex;
  flex-wrap: wrap;
  align-content: center; /* или flex-start, flex-end, space-between, и т.д. */
}
```

## Практические рекомендации

1. Используйте `flex-direction: column` для создания вертикальных списков, особенно актуально для мобильных интерфейсов в российском сегменте
2. При работе с адаптивным дизайном часто комбинируйте `flex-wrap: wrap` с `justify-content: space-around` для создания сеток карточек
3. Свойство `align-items: center` идеально подходит для центрирования элементов, что особенно важно в современном UI/UX дизайне
4. В российских реалиях 2025 года учитывайте особенности отображения на устройствах с разными плотностями пикселей, особенно на Android-устройствах
5. Используйте `justify-content: space-between` для создания навигационных панелей с равномерным распределением элементов
6. Для сложных макетов можно комбинировать несколько flex-контейнеров друг в друге

## Примеры использования

### Простой горизонтальный список
```css
.nav-container {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
}
```

### Адаптивная сетка карточек
```css
.card-grid {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  align-content: flex-start;
}
```

### Вертикальный список с центрированием
```css
.vertical-list {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
```

## Связанные темы

- [[Основы-Flexbox]]
- [[Элементы-Flex]]
- [[Выравнивание]]
- [[Практические-примеры]]
- [[CSS Grid]]
- [[Адаптивный дизайн]]

## Источники и дополнительные материалы

- [MDN Web Docs: Flexbox](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_Flexible_Box_Layout)
- [CSS-Tricks Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [Современные практики верстки в России](https://habr.com/ru/companies/ruvds/articles/516444/)

## Ключевые теги

#css #flexbox #frontend #layout #flex-container #web-development #responsive-design #frontend-development #web-design #html #frontend-tips #flexible-box
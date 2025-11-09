---
aliases: ["CSS Filters", "Визуальные эффекты CSS", "CSS Effects"]
tags: [css, filters, visual-effects, styling]
---

# CSS Filters и визуальные эффекты

CSS предоставляет мощные инструменты для создания визуальных эффектов без использования изображений или JavaScript. Основные свойства `filter` и `backdrop-filter` позволяют применять графические эффекты к элементам HTML, изменяя их внешний вид.

## Свойство filter

Свойство `filter` позволяет применять графические эффекты к элементу. Оно может принимать одно или несколько значений, каждое из которых представляет собой фильтр.

```css
.element {
  filter: blur(5px);
}
```

### Основные функции фильтров

#### blur()
Применяет размытие Гаусса к элементу. Значение определяет радиус размытия.

```css
.blur-element {
  filter: blur(4px);
}
```

#### brightness()
Изменяет яркость изображения. Значение 100% оставляет изображение без изменений, значения выше 100% делают его ярче, а ниже - темнее.

```css
.brighter {
  filter: brightness(150%);
}

.darker {
  filter: brightness(50%);
}
```

#### contrast()
Изменяет контрастность изображения. Значение 100% оставляет изображение без изменений.

```css
.high-contrast {
  filter: contrast(200%);
}

.low-contrast {
  filter: contrast(50%);
}
```

#### grayscale()
Преобразует изображение в оттенки серого. Значение 100% полностью преобразует изображение, 0% - оставляет без изменений.

```css
.grayscale {
  filter: grayscale(100%);
}
```

#### hue-rotate()
Поворачивает оттенки изображения. Значение указывается в градусах.

```css
.hue-rotated {
  filter: hue-rotate(90deg);
}
```

#### invert()
Инвертирует цвета изображения. Значение 100% полностью инвертирует, 0% - оставляет без изменений.

```css
.inverted {
  filter: invert(100%);
}
```

#### opacity()
Изменяет прозрачность изображения. 100% оставляет без изменений, 0% делает полностью прозрачным.

```css
.semi-transparent {
  filter: opacity(50%);
}
```

#### saturate()
Изменяет насыщенность цветов. 100% оставляет без изменений, больше - увеличивает насыщенность, меньше - уменьшает.

```css
.highly-saturated {
  filter: saturate(200%);
}

.desaturated {
  filter: saturate(50%);
}
```

#### sepia()
Применяет сепию к изображению. 100% полностью применяет эффект, 0% - оставляет без изменений.

```css
.sepia-effect {
  filter: sepia(100%);
}
```

## Свойство backdrop-filter

Свойство `backdrop-filter` применяет фильтры к области под элементом, а не к самому элементу. Это особенно полезно для создания эффектов размытия фона, как в iOS.

```css
.modal {
  backdrop-filter: blur(10px);
  background-color: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}
```

## Комбинация фильтров

Фильтры можно комбинировать, перечисляя их через пробел:

```css
.combined-filters {
  filter: blur(2px) brightness(110%) contrast(120%);
}
```

## Практические примеры

### Эффект размытого фона (Glassmorphism)

```css
.glass-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 16px;
  padding: 20px;
}
```

### Эффект при наведении

```css
.image-hover {
  transition: filter 0.3s ease;
}

.image-hover:hover {
  filter: brightness(110%) contrast(110%);
}
```

### Темная тема с помощью invert

```css
.dark-mode {
  filter: invert(1) hue-rotate(180deg);
}
```

## Производительность

При использовании фильтров важно учитывать производительность:

- **Аппаратное ускорение**: Фильтры часто используют GPU, что делает их быстрыми, но может потреблять больше памяти
- **Избегайте фильтров на больших элементах**: Это может вызвать задержки при рендеринге
- **Используйте `will-change`**: Для элементов с анимированными фильтрами

```css
.animated-filter {
  will-change: filter;
  transition: filter 0.3s ease;
}
```

## Поддержка браузерами

| Свойство | Chrome | Firefox | Safari | Edge |
|----------|--------|---------|--------|------|
| filter | 18+ | 35+ | 6+ | 13+ |
| backdrop-filter | 76+ | 99+ | 9+ | 79+ |

> [!warning] 
> `backdrop-filter` имеет более ограниченную поддержку, особенно в старых версиях браузеров.

## Примеры использования

### Создание эффекта размытия для модальных окон

```css
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
}
```

### Эффект при скролле

```css
.parallax-element {
  filter: blur(0px);
}

.parallax-element.scrolled {
  filter: blur(2px);
  transition: filter 0.3s ease;
}
```

## Связанные темы

- [[css/positioning/positioning]] - для понимания позиционирования элементов с фильтрами
- [[css/animation/transitions]] - для анимации фильтров
- [[css/backgrounds/borders]] - для создания комплексных визуальных эффектов
- [[css/transforms/transform]] - для комбинирования с трансформациями

## Заключение

CSS фильтры предоставляют мощный способ визуального изменения элементов без использования изображений или JavaScript. Они особенно полезны для создания современных эффектов, таких как glassmorphism, и могут значительно улучшить пользовательский интерфейс при правильном использовании с учетом производительности.

> [!tip]
> Всегда тестируйте фильтры на разных устройствах, особенно при использовании `backdrop-filter`, так как его поддержка может отличаться.
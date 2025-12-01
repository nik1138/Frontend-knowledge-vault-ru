---
aliases: ["Vue Scoped Styles", "Локальные стили в Vue"]
tags: ["vue", "css", "styling", "best-practices"]
---

# Scoped-стили в Vue.js

## Введение

Scoped-стили в Vue.js позволяют применять CSS-правила только к компоненту, в котором они определены. Это предотвращает влияние стилей на другие компоненты и упрощает локальное управление стилями.

## Основы использования

Для применения scoped-стилей используется атрибут `scoped` в теге `<style>`:

```vue
<template>
  <div class="container">
    <h1 class="title">Заголовок компонента</h1>
    <p class="description">Описание компонента</p>
  </div>
</template>

<script>
export default {
  name: 'MyComponent'
}
</script>

<style scoped>
.container {
  padding: 20px;
  border: 1px solid #ccc;
}

.title {
  color: #333;
  font-size: 24px;
}

.description {
  color: #666;
  font-size: 16px;
}
</style>
```

## Как работает scoped-стилизация

При использовании `scoped` Vue автоматически добавляет уникальные атрибуты к элементам и модифицирует селекторы CSS, чтобы они применялись только к текущему компоненту. Например, код выше будет преобразован в:

```html
<div class="container" data-v-f3f3eg9>
  <h1 class="title" data-v-f3f3eg9>Заголовок компонента</h1>
  <p class="description" data-v-f3f3eg9>Описание компонента</p>
</div>
```

```css
.container[data-v-f3f3eg9] {
  padding: 20px;
  border: 1px solid #ccc;
}

.title[data-v-f3f3eg9] {
  color: #333;
  font-size: 24px;
}

.description[data-v-f3f3eg9] {
  color: #666;
  font-size: 16px;
}
```

## Глубокие селекторы

Иногда необходимо применить стили к дочерним компонентам. Для этого используются глубокие селекторы:

### Вариант 1: Селектор `>>>`

```vue
<style scoped>
.parent >>> .child {
  color: red;
}
</style>
```

### Вариант 2: Селектор `::v-deep`

```vue
<style scoped>
.parent ::v-deep .child {
  color: red;
}
</style>
```

### Вариант 3: Селектор `:deep()`

```vue
<style scoped>
.parent :deep(.child) {
  color: red;
}
</style>
```

> [!warning] Важно
> Глубокие селекторы влияют на все дочерние компоненты, что может нарушить изоляцию стилей. Используйте их с осторожностью.

## Миксование scoped и глобальных стилей

В одном компоненте можно использовать как scoped, так и глобальные стили:

```vue
<style>
/* Глобальные стили */
.global-class {
  color: blue;
}
</style>

<style scoped>
/* Локальные стили */
.local-class {
  color: red;
}
</style>
```

## Практические рекомендации

### 1. Избегайте чрезмерного использования глубоких селекторов

> [!tip] Совет
> Стараться использовать scoped-стили без глубоких селекторов для поддержания изоляции компонентов

### 2. Комбинирование с CSS-переменными

```vue
<template>
  <div class="card">
    <h3 class="card-title">{{ title }}</h3>
    <p class="card-content">{{ content }}</p>
  </div>
</template>

<script>
export default {
  name: 'CardComponent',
  props: {
    title: String,
    content: String
  }
}
</script>

<style scoped>
.card {
  border: 1px solid var(--border-color, #ddd);
  border-radius: 8px;
  padding: 16px;
  background-color: var(--bg-color, #fff);
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.card-title {
  color: var(--title-color, #333);
  font-size: 18px;
  margin-bottom: 8px;
}

.card-content {
  color: var(--text-color, #666);
  font-size: 14px;
}
</style>
```

### 3. Работа с динамическими классами

```vue
<template>
  <div 
    class="button" 
    :class="{ 'button--active': isActive, 'button--disabled': isDisabled }"
  >
    Кнопка
  </div>
</template>

<style scoped>
.button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

.button--active {
  background-color: #0056b3;
}

.button--disabled {
  background-color: #6c757d;
  cursor: not-allowed;
  opacity: 0.6;
}
</style>
```

## Совместимость с препроцессорами

Scoped-стили работают с CSS-препроцессорами:

```vue
<style lang="scss" scoped>
.container {
  padding: 20px;
  
  .title {
    color: #333;
    font-size: 24px;
    
    &.highlighted {
      color: #ff6b6b;
    }
  }
}
</style>
```

## Рекомендации по производительности

1. Избегайте чрезмерного использования глубоких селекторов, так как они могут замедлить рендеринг
2. Минимизируйте количество scoped-стилей в больших приложениях
3. Используйте CSS-переменные для кастомизации компонентов вместо глубоких селекторов

## Альтернативы scoped-стилям

- [[CSS-модули]] - альтернативный подход к изоляции стилей
- [[Препроцессоры]] - использование препроцессоров с дополнительными возможностями
- [[PostCSS]] - инструмент для трансформации CSS

## Заключение

Scoped-стили в Vue.js предоставляют простой способ изолировать стили компонентов от остальной части приложения. Это помогает избежать конфликтов имен классов и упрощает поддержку кода. Однако при использовании глубоких селекторов следует быть осторожным, чтобы не нарушить принципы изоляции компонентов.

Для современных приложений в России в 2025 году рекомендуется использовать scoped-стили в сочетании с CSS-переменными и современными возможностями CSS, такими как flexbox и grid.

## См. также

- [[Глобальные-стили]]
- [[CSS-модули]]
- [[Vue компоненты]]
- [[CSS]]
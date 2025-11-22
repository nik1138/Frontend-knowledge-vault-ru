---
aliases: []
tags: [vue, performance, optimization]
---

# Vue Performance Optimization

Оптимизация производительности в Vue.js - это комплекс мер, направленных на улучшение скорости загрузки, отзывчивости и общего пользовательского опыта приложения. Производительность Vue-приложений может быть оптимизирована на нескольких уровнях: компонентном, архитектурном и уровне сборки.

## Основные аспекты производительности

### Время загрузки приложения
- Размер бандла JavaScript/CSS
- Количество HTTP-запросов
- Эффективность асинхронной загрузки

### Время отклика
- Быстродействие рендеринга
- Время обновления DOM
- Частота обновлений компонентов

### Потребление памяти
- Утечки памяти
- Эффективное управление состоянием
- Очистка ресурсов

## Оптимизация компонентов

### Использование v-memo (Vue 3.2+)

```vue
<template>
  <div v-for="item in list" :key="item.id">
    <ListItem 
      :item="item" 
      :key="item.id"
      v-memo="[item.id, item.selected]"
    />
  </div>
</template>
```

### Ленивая загрузка компонентов

```javascript
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() =>
  import('./MyComponent.vue')
)

export default {
  components: {
    AsyncComponent
  }
}
```

### Использование keep-alive для кэширования

```vue
<template>
  <keep-alive>
    <component :is="currentView"></component>
  </keep-alive>
</template>
```

## Оптимизация рендеринга

### Использование v-show vs v-if

- Используйте `v-show`, когда элемент часто переключается
- Используйте `v-if`, когда условие редко изменяется

### Правильное использование ключей в списках

```vue
<!-- Плохо -->
<li v-for="item in items">{{ item.name }}</li>

<!-- Хорошо -->
<li v-for="item in items" :key="item.id">{{ item.name }}</li>
```

### Избегание ненужных вычислений

```javascript
// Используем computed для кэширования
computed: {
  expensiveValue() {
    // Вычисления будут выполнены только при изменении зависимостей
    return this.items.filter(item => item.active).map(item => item.name)
  }
}
```

## Оптимизация состояния

### Использование реактивных ссылок эффективно

```javascript
// Избегаем ненужной реактивности для больших объектов
import { shallowRef } from 'vue'

const largeList = shallowRef([])
// Только верхний уровень реактивен
```

### Правильное управление глобальным состоянием

- Используйте Pinia или Vuex для централизованного управления состоянием
- Избегайте избыточного обновления компонентов
- Используйте геттеры для вычисляемых значений

## Tree Shaking и код-сплиттинг

### Tree Shaking

```javascript
// Импортируем только необходимые функции
import { ref, computed } from 'vue'
// Вместо импорта всего Vue
```

### Код-сплиттинг с помощью динамических импортов

```javascript
const AdminComponent = defineAsyncComponent(() =>
  import('./AdminPanel.vue')
)
```

## Оптимизация архитектуры

### Минимизация обновлений DOM

- Использование виртуального скроллинга для больших списков
- Оптимизация сложных деревьев компонентов
- Использование событийной шины для минимизации пропсов

### Виртуальный скроллинг

```vue
<template>
  <RecycleScroller
    class="scroller"
    :items="items"
    :item-size="54"
    key-field="id"
    v-slot="{ item }"
  >
    <div class="user">
      {{ item.name }}
    </div>
  </RecycleScroller>
</template>
```

## Использование профилирования

### Vue DevTools

- Профилирование производительности компонентов
- Отслеживание изменений состояния
- Анализ времени обновления

### Browser DevTools

- Time to Interactive (TTI)
- First Contentful Paint (FCP)
- Largest Contentful Paint (LCP)

## Лучшие практики

1. Использовать `v-memo` для оптимизации рендеринга списков
2. Избегать чрезмерной вложенности компонентов
3. Минимизировать количество реактивных свойств
4. Использовать асинхронные компоненты для нечасто используемых элементов
5. Оптимизировать изображения и статические ресурсы
6. Использовать CDN для статических ресурсов
7. Применять lazy loading для контента вне области видимости
8. Использовать веб-воркеры для тяжелых вычислений

## Мониторинг и метрики

### Ключевые метрики

- First Paint (FP) и First Contentful Paint (FCP)
- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)

### Инструменты для измерения

- Lighthouse
- Web Vitals
- Custom performance markers

См. также: [[Vue компоненты]], [[Composition API]], [[Vue Testing]]
---
aliases: [Компонент, Vue Component]
tags: [vue, component, frontend, ru-2025]
---

# Component

Специальный элемент `<component>` в Vue.js позволяет динамически переключаться между различными компонентами на основе определенных условий или значений. Это один из ключевых элементов в арсенале разработчика Vue, особенно при создании адаптивных интерфейсов и сложных приложений.

## Основы использования

Элемент `<component>` используется с директивой `:is` для динамического определения, какой компонент должен быть отображен:

```vue
<template>
  <div>
    <component :is="currentComponent" />
  </div>
</template>

<script>
import ComponentA from './ComponentA.vue'
import ComponentB from './ComponentB.vue'

export default {
  components: {
    ComponentA,
    ComponentB
  },
  data() {
    return {
      currentComponent: 'ComponentA' // или 'ComponentB'
    }
  }
}
</script>
```

## Практическое применение

В российской практике 2025 года динамические компоненты особенно востребованы при:
- Создании универсальных карточек товаров в e-commerce
- Реализации адаптивных интерфейсов под разные устройства
- Построении динамических форм и валидации
- Интеграции с внешними CMS-системами

## Пример использования в российском контексте

```vue
<template>
  <div class="product-card">
    <component 
      :is="getProductComponent(product.type)" 
      :product="product" 
      :locale="'ru-RU'"
      @purchase="handlePurchase"
    />
  </div>
</template>

<script>
import ElectronicsCard from './product-cards/ElectronicsCard.vue'
import ClothingCard from './product-cards/ClothingCard.vue'
import FoodCard from './product-cards/FoodCard.vue'

export default {
  name: 'DynamicProductCard',
  props: {
    product: {
      type: Object,
      required: true
    }
  },
  components: {
    ElectronicsCard,
    ClothingCard,
    FoodCard
  },
  methods: {
    getProductComponent(type) {
      const componentMap = {
        electronics: 'ElectronicsCard',
        clothing: 'ClothingCard',
        food: 'FoodCard'
      }
      return componentMap[type] || 'ElectronicsCard'
    },
    handlePurchase(product) {
      // Логика оформления заказа с учетом российских реалий
      console.log(`Заказ оформлен на ${product.title}`)
    }
  }
}
</script>
```

## Совместимость и особенности

- В Vue 3 элемент `<component>` поддерживает все возможности Composition API
- Работает с асинхронной загрузкой компонентов через `defineAsyncComponent`
- При использовании в SSR (Server-Side Rendering) требует особого внимания к гидратации

## Связанные темы

- [[Slot]] - для передачи контента в динамические компоненты
- [[Teleport]] - для вывода компонентов за пределы DOM-дерева
- [[Keep-alive]] - для кэширования состояния компонентов
- [[Suspense]] - для обработки асинхронной загрузки
- [[Vue компоненты]]

## Лучшие практики

1. Используйте стабильные ключи для корректной работы виртуального DOM
2. Избегайте частой перерисовки компонентов для производительности
3. Учитывайте особенности локализации при динамическом выборе компонентов
---
aliases: [Слот, Vue Slot]
tags: [vue, slot, frontend, ru-2025]
---

# Slot

Слоты в Vue.js - это мощный механизм для композиции компонентов, позволяющий передавать контент от родительского компонента к дочернему. Это один из ключевых элементов для создания гибких и переиспользуемых компонентов в современных приложениях.

## Основы использования

Слоты позволяют определять "дыры" в компонентах, которые могут быть заполнены контентом из родительского компонента:

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="card">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent>
    <template #header>
      <h2>Заголовок карточки</h2>
    </template>
    
    <p>Основной контент карточки</p>
    
    <template #footer>
      <button>Действие</button>
    </template>
  </ChildComponent>
</template>
```

## Типы слотов

### 1. Анонимные слоты (default slots)
```vue
<template>
  <div>
    <slot></slot>
  </div>
</template>
```

### 2. Именованные слоты (named slots)
```vue
<template>
  <div>
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot name="main"></slot>
    </main>
  </div>
</template>
```

### 3. Слоты с ограниченной областью видимости (scoped slots)
```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <slot :user="userData" :status="userStatus"></slot>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userData: { name: 'Иван', role: 'администратор' },
      userStatus: 'активен'
    }
  }
}
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent v-slot="slotProps">
    <p>Пользователь: {{ slotProps.user.name }}</p>
    <p>Статус: {{ slotProps.user.status }}</p>
  </ChildComponent>
</template>
```

## Практическое применение в российских реалиях 2025

Слоты особенно востребованы при создании:
- UI-китов для крупных российских компаний
- Адаптивных интерфейсов под различные устройства
- Компонентов для электронной коммерции с разными вариантами отображения
- Форм с динамическим содержимым

## Пример использования в российском контексте

```vue
<!-- RussianCard.vue -->
<template>
  <div class="russian-card">
    <div class="card-header">
      <slot name="header">
        <!-- Резервный контент -->
        <h3>Стандартный заголовок</h3>
      </slot>
    </div>
    
    <div class="card-body">
      <slot :locale="'ru-RU'" :currency="'RUB'"></slot>
    </div>
    
    <div class="card-footer">
      <slot name="actions">
        <button @click="handleDefaultAction">Действие</button>
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'RussianCard',
  methods: {
    handleDefaultAction() {
      console.log('Выполнено стандартное действие')
    }
  }
}
</script>
```

```vue
<!-- Использование в родительском компоненте -->
<template>
  <RussianCard>
    <template #header>
      <h2>Карточка продукта</h2>
    </template>
    
    <template #default="{ locale, currency }">
      <div class="product-info">
        <p>Цена: {{ formatPrice(product.price, currency) }}</p>
        <p>Локализация: {{ locale }}</p>
      </div>
    </template>
    
    <template #actions>
      <button @click="addToCart">Добавить в корзину</button>
      <button @click="addToFavorites">В избранное</button>
    </template>
  </RussianCard>
</template>

<script>
export default {
  data() {
    return {
      product: {
        price: 5990
      }
    }
  },
  methods: {
    formatPrice(price, currency) {
      return new Intl.NumberFormat('ru-RU', {
        style: 'currency',
        currency: currency
      }).format(price)
    },
    addToCart() {
      // Логика добавления в корзину с учетом российских реалий
    },
    addToFavorites() {
      // Логика добавления в избранное
    }
  }
}
</script>
```

## Совместимость и особенности

- В Vue 3 слоты работают через компиляцию в render-функции
- Поддерживают фрагменты (множественные корневые элементы)
- Совместимы с Teleport и Keep-alive
- Работают с SSR без особых ограничений

## Связанные темы

- [[Component]] - для понимания динамических компонентов
- [[Teleport]] - для вывода содержимого слотов за пределы компонента
- [[Keep-alive]] - для кэширования компонентов с содержимым слотов
- [[Vue компоненты]]
- [[Vue композиция]]

## Лучшие практики

1. Используйте именованные слоты для ясности структуры компонента
2. Предоставляйте резервный контент для слотов
3. Используйте scoped slots для передачи данных из дочернего компонента
4. Избегайте чрезмерной вложенности слотов для поддерживаемости кода
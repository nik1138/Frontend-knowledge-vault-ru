# Vue Components

Components - это основные строительные блоки Vue приложений. Компонент - это переиспользуемый экземпляр с определённой функцией и поведением.

## Основные концепции

### [Single File Components (SFC)](sfc.md)
Структура и особенности компонентов во Vue (файлы .vue)

### [Props](props.md)
Передача данных от родительского компонента к дочернему

### [Events](events.md)
Эмиттинг событий от дочернего компонента к родительскому

### [Slots](slots.md)
Паттерн для композиции компонентов с передачей содержимого

### [Component Registration](registration.md)
Глобальная и локальная регистрация компонентов

## Компонентный жизненный цикл

См. раздел [[lifecycle/Lifecycle Hooks]] для подробностей о жизненном цикле компонентов

## Практические паттерны

### Асинхронные компоненты
```javascript
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() => import('./MyComponent.vue'))
```

### Динамические компоненты
```vue
<template>
  <component :is="currentComponent"></component>
</template>
```

## Связь с другими концепциями
- [[reactivity/Reactivity System]] - реактивность внутри компонентов
- [[composables/Composables]] - логика, вынесенная из компонентов
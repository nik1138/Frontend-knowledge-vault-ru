---
aliases: [CSS Animations in Vue, Анимации CSS в Vue]
tags: [vue, css, animation, frontend]
---

# CSS-анимации в Vue.js

## Обзор

CSS-анимации в Vue.js позволяют создавать плавные переходы и анимации элементов без написания сложного JavaScript кода. Vue предоставляет встроенные возможности для работы с CSS-анимациями, особенно при использовании компонентов `<transition>` и `<transition-group>`.

## Основы CSS-анимаций

Vue автоматически применяет CSS-классы во время переходов. Когда элемент оборачивается в компонент `<transition>`, Vue добавляет и удаляет соответствующие CSS-классы в зависимости от состояния перехода.

### CSS-классы переходов

- `v-enter-from` - начальное состояние для появления
- `v-enter-active` - активное состояние для появления
- `v-enter-to` - конечное состояние для появления
- `v-leave-from` - начальное состояние для исчезновения
- `v-leave-active` - активное состояние для исчезновения
- `v-leave-to` - конечное состояние для исчезновения

### Пример базовой CSS-анимации

```vue
<template>
  <div>
    <button @click="show = !show">Переключить</button>
    <transition name="fade">
      <p v-if="show">Привет, Vue анимации!</p>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: true
    }
  }
}
</script>

<style>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
```

## Расширенные CSS-анимации

### Анимации с использованием ключевых кадров

```css
.bounce-enter-active {
  animation: bounce-in 0.8s;
}
.bounce-leave-active {
  animation: bounce-in 0.8s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.25);
  }
  100% {
    transform: scale(1);
  }
}
```

### Сложные переходы

Для более сложных анимаций можно комбинировать несколько CSS-свойств:

```css
.slide-fade-enter-active {
  transition: all 0.3s ease-out;
}
.slide-fade-leave-active {
  transition: all 0.8s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter-from {
  translate: calc(-100vw - 5px);
  opacity: 0;
}
.slide-fade-leave-to {
  translate: calc(100vw + 5px);
  opacity: 0;
}
```

## Практические примеры

### Анимация списка

```vue
<template>
  <div>
    <button @click="addItem">Добавить элемент</button>
    <transition-group name="list" tag="ul" class="list-container">
      <li v-for="item in items" :key="item.id" class="list-item">
        {{ item.text }}
        <button @click="removeItem(item.id)">Удалить</button>
      </li>
    </transition-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, text: 'Элемент 1' },
        { id: 2, text: 'Элемент 2' },
        { id: 3, text: 'Элемент 3' }
      ],
      nextId: 4
    }
  },
  methods: {
    addItem() {
      this.items.push({ id: this.nextId++, text: `Элемент ${this.nextId - 1}` })
    },
    removeItem(id) {
      this.items = this.items.filter(item => item.id !== id)
    }
  }
}
</script>

<style>
.list-container {
  position: relative;
  overflow: hidden;
}

.list-item {
  display: block;
  margin-right: 10px;
}

.list-enter-active, .list-leave-active {
  transition: all 1s ease;
}
.list-enter-from {
  opacity: 0;
  transform: translateX(30px);
}
.list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
  position: absolute;
}
</style>
```

## Совместимость с российскими реалиями 2025

В 2025 году в российском веб-дизайне и фронтенд-разработке особенно важна адаптация к ограничениям по производительности и требованиям доступности. CSS-анимации в Vue позволяют создавать эффективные интерфейсы, которые работают даже на менее мощных устройствах, что актуально для российского рынка.

### Оптимизация производительности

- Используйте `transform` и `opacity` для анимаций, так как они не вызывают перерисовки
- Избегайте анимации свойств, которые вызывают layout или paint
- Используйте `will-change` для подготовки браузера к анимации

```css
.optimized-animation {
  will-change: transform, opacity;
  transform: translateZ(0); /* Включение аппаратного ускорения */
}
```

## Лучшие практики

1. Используйте CSS-анимации для простых переходов
2. Комбинируйте с JavaScript для сложных анимационных последовательностей
3. Обеспечьте доступность, добавляя опцию отключения анимаций
4. Тестируйте на разных устройствах и браузерах

## Связанные темы

- [[JavaScript-анимации]]
- [[Встроенные-переходы]]
- [[Библиотеки-анимаций]]
- [[Пользовательские-анимации]]

## Заключение

CSS-анимации в Vue.js - мощный инструмент для создания интерактивных и привлекательных пользовательских интерфейсов. Правильное использование CSS-анимаций позволяет создавать плавные переходы, улучшая пользовательский опыт без значительного влияния на производительность.
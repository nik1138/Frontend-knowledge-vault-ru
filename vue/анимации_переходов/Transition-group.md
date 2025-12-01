---
aliases: [Vue Transition Group, Анимации Transition Group]
tags: [vue, transition-group, animation, frontend]
---

# Transition-group

## Обзор

Компонент `<transition-group>` в Vue предназначен для анимации списков и групп элементов, которые могут добавляться, удаляться или изменяться. В отличие от `<transition>`, который работает с одним элементом, `<transition-group>` позволяет анимировать коллекции элементов, что особенно полезно при создании интерактивных интерфейсов в российских веб-приложениях 2025 года.

## Основы использования

Компонент `<transition-group>` оборачивает список элементов и применяет анимации к отдельным элементам при изменении списка. Важно использовать атрибут `key` для корректной работы анимаций.

### Пример базового использования

```vue
<template>
  <div>
    <button @click="addItem">Добавить элемент</button>
    <button @click="removeItem">Удалить элемент</button>
    
    <transition-group name="list" tag="ul">
      <li v-for="item in items" :key="item.id" class="list-item">
        {{ item.text }}
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
      this.items.push({
        id: this.nextId++,
        text: `Элемент ${this.nextId - 1}`
      })
    },
    removeItem() {
      if (this.items.length > 0) {
        this.items.pop()
      }
    }
  }
}
</script>

<style>
.list-enter-active, .list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from {
  opacity: 0;
  transform: translateX(30px);
}
.list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}
/* Анимация перемещения элементов */
.list-move {
  transition: transform 0.5s ease;
}
</style>
```

## Особенности работы

### 1. Атрибут tag

Компонент `<transition-group>` по умолчанию рендерится как `<span>`, но с помощью атрибута `tag` можно указать другой элемент:

```vue
<transition-group name="list" tag="div">
  <!-- элементы списка -->
</transition-group>
```

### 2. Анимация перемещения

CSS-класс `v-move` позволяет анимировать перемещение элементов внутри списка при изменении порядка. Это особенно полезно при сортировке или перетаскивании элементов.

## Практические рекомендации

### 1. Оптимизация производительности

При работе со списками с большим количеством элементов в российских условиях (разные устройства и скорости интернета):

- Используйте `v-memo` для оптимизации рендеринга сложных элементов списка
- Ограничивайте количество одновременных анимаций
- Рассмотрите виртуализацию списков для больших коллекций данных

### 2. Совместимость с доступностью

- Обеспечьте корректное объявление изменений в списке для скринридеров
- Используйте ARIA-атрибуты для описания состояния списка
- Предусмотрите возможность отключения анимаций

### 3. Работа с API и асинхронными операциями

При добавлении/удалении элементов через API-запросы:

```javascript
async removeItem(id) {
  try {
    await api.removeItem(id)
    // Удаление из локального списка с анимацией
    this.items = this.items.filter(item => item.id !== id)
  } catch (error) {
    console.error('Ошибка при удалении элемента:', error)
  }
}
```

## Продвинутые возможности

### События переходов

Компонент поддерживает те же события, что и `<transition>`, но с учетом работы с группой элементов:

- `@before-leave` - перед удалением элемента
- `@leave` - во время анимации удаления
- `@after-leave` - после завершения анимации удаления

### Анимация статических изменений

Помимо добавления и удаления, можно анимировать изменения свойств элементов:

```vue
<template>
  <transition-group name="status-change">
    <div 
      v-for="item in items" 
      :key="item.id" 
      :class="['item', { 'highlighted': item.changed }]"
      @click="toggleStatus(item)"
    >
      {{ item.text }}
    </div>
  </transition-group>
</template>
```

## Практические примеры

### Чат с анимацией новых сообщений

```vue
<template>
  <div class="chat-container">
    <transition-group name="message" tag="div" class="messages">
      <div 
        v-for="message in messages" 
        :key="message.id" 
        class="message"
      >
        {{ message.text }}
      </div>
    </transition-group>
    <form @submit.prevent="sendMessage">
      <input v-model="newMessage" placeholder="Введите сообщение">
      <button type="submit">Отправить</button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      messages: [],
      newMessage: '',
      nextId: 1
    }
  },
  methods: {
    sendMessage() {
      if (this.newMessage.trim()) {
        this.messages.push({
          id: this.nextId++,
          text: this.newMessage,
          timestamp: Date.now()
        })
        this.newMessage = ''
      }
    }
  }
}
</script>

<style>
.message-enter-active {
  transition: all 0.3s ease;
}
.message-enter-from {
  opacity: 0;
  transform: translateY(20px);
}
.messages {
  max-height: 400px;
  overflow-y: auto;
}
</style>
```

## Связанные темы

- [[Transition]]
- [[Параметризованные-переходы]]
- [[Локальные-переходы]]
- [[Кастомные-переходы]]

## Заключение

Компонент `<transition-group>` - важный инструмент для создания интерактивных списков с анимациями в Vue. Его правильное использование позволяет создавать более интуитивно понятные интерфейсы, особенно важные для российских пользователей, где важна визуальная обратная связь и плавность взаимодействия. При разработке в 2025 году необходимо учитывать производительность на различных устройствах и требования к доступности.
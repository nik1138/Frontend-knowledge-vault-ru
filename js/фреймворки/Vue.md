---
aliases: [VueJS, Vue.js, Вью]
tags: [javascript, фреймворк, vue, frontend]
---

# Vue.js

## Обзор

Vue.js - это прогрессивный фреймворк JavaScript для создания пользовательских интерфейсов. В 2025 году Vue остается одним из популярных решений, особенно благодаря своей простоте и гибкости.

## Основные особенности

- **Прогрессивный фреймворк** - Можно использовать как библиотеку или как полноценный фреймворк
- **Двустороннее связывание данных** - Упрощение синхронизации данных между моделью и представлением
- **Компонентная архитектура** - Повторное использование компонентов
- **Легкий вес** - Меньший размер по сравнению с другими фреймворками
- **Встроенная система маршрутизации** - [[Vue Router]]
- **Управление состоянием** - [[Vuex]] и [[Pinia]]

## Установка и настройка

### Создание нового проекта

```bash
npm create vue@latest my-vue-app
cd my-vue-app
npm install
npm run dev
```

### Использование CDN

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```

## Основные концепции

### Реактивность

Vue обеспечивает реактивность данных автоматически:

```javascript
const { createApp, ref } = Vue;

createApp({
  setup() {
    const count = ref(0);
    
    return {
      count
    };
  }
}).mount('#app');
```

### Компоненты

Компоненты в Vue могут быть определены как в одном файле (.vue), так и в JavaScript:

```vue
<template>
  <div class="counter">
    <p>Счетчик: {{ count }}</p>
    <button @click="increment">Увеличить</button>
  </div>
</template>

<script>
import { ref } from 'vue';

export default {
  setup() {
    const count = ref(0);
    
    const increment = () => {
      count.value++;
    };
    
    return {
      count,
      increment
    };
  }
};
</script>

<style scoped>
.counter {
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
</style>
```

### Жизненный цикл компонента

- `onMounted` - Вызывается после монтирования компонента
- `onUpdated` - Вызывается после обновления компонента
- `onUnmounted` - Вызывается при размонтировании компонента

## Composition API vs Options API

### Composition API

```javascript
import { ref, onMounted } from 'vue';

export default {
  setup() {
    const count = ref(0);
    
    onMounted(() => {
      console.log('Компонент смонтирован');
    });
    
    const increment = () => {
      count.value++;
    };
    
    return {
      count,
      increment
    };
  }
};
```

### Options API

```javascript
export default {
  data() {
    return {
      count: 0
    };
  },
  methods: {
    increment() {
      this.count++;
    }
  },
  mounted() {
    console.log('Компонент смонтирован');
  }
};
```

## Практические примеры

### Простое приложение "Список задач"

```vue
<template>
  <div class="todo-app">
    <h1>Список задач</h1>
    <div class="input-section">
      <input 
        v-model="newTodo" 
        @keyup.enter="addTodo" 
        placeholder="Добавить задачу"
      />
      <button @click="addTodo">Добавить</button>
    </div>
    <ul class="todo-list">
      <li 
        v-for="todo in todos" 
        :key="todo.id" 
        :class="{ completed: todo.completed }"
        @click="toggleTodo(todo.id)"
      >
        {{ todo.text }}
      </li>
    </ul>
  </div>
</template>

<script>
import { ref, reactive } from 'vue';

export default {
  setup() {
    const newTodo = ref('');
    const todos = ref([]);
    
    const addTodo = () => {
      if (newTodo.value.trim() !== '') {
        todos.value.push({
          id: Date.now(),
          text: newTodo.value,
          completed: false
        });
        newTodo.value = '';
      }
    };
    
    const toggleTodo = (id) => {
      const todo = todos.value.find(t => t.id === id);
      if (todo) {
        todo.completed = !todo.completed;
      }
    };
    
    return {
      newTodo,
      todos,
      addTodo,
      toggleTodo
    };
  }
};
</script>

<style scoped>
.todo-app {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
}

.input-section {
  display: flex;
  margin-bottom: 20px;
}

.input-section input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px 0 0 4px;
}

.input-section button {
  padding: 8px 16px;
  border: 1px solid #ccc;
  border-left: none;
  border-radius: 0 4px 4px 0;
  background-color: #f5f5f5;
}

.todo-list {
  list-style: none;
  padding: 0;
}

.todo-list li {
  padding: 10px;
  border: 1px solid #eee;
  margin-bottom: 5px;
  cursor: pointer;
  border-radius: 4px;
}

.todo-list li.completed {
  text-decoration: line-through;
  color: #999;
}
</style>
```

## Экосистема Vue

- [[Vue Router]] - Маршрутизация
- [[Vuex]] - Управление состоянием
- [[Pinia]] - Современная альтернатива Vuex
- [[Nuxt.js]] - Фреймворк для универсальных приложений
- [[Vue CLI]] - Инструмент командной строки
- [[Vite]] - Быстрая сборка

## Российские реалии и особенности

### Применение в российских компаниях

Vue.js используется в различных российских компаниях:

- [[Яндекс]] - для некоторых внутренних инструментов
- [[Сбер]] - в цифровых продуктах
- [[Мегафон]] - в клиентских интерфейсах
- [[Ростелеком]] - в веб-приложениях
- Многие стартапы и агентства

### Особенности разработки в России

- **Простота изучения** - Vue идеально подходит для команд с разным уровнем подготовки
- **Документация** - Хорошо документированный фреймворк с переводами
- **Производительность** - Важно для пользователей с медленным интернетом в регионах
- **Санкционные ограничения** - Меньше зависимости от заблокированных сервисов по сравнению с другими фреймворками

## Лучшие практики

- Использование TypeScript с Vue 3
- Правильная организация компонентов
- Тестирование с помощью [[Vue Test Utils]] и [[Jest]]
- Использование ESLint и Prettier
- Оптимизация производительности с помощью v-memo

## Современные тенденции (2025)

- **Vue 3** - Полное использование Composition API
- **Vue Macros** - Дополнительные возможности для метапрограммирования
- **Server-Side Rendering** - Улучшенная поддержка SSR
- **Static Site Generation** - Развитие Nuxt и других решений
- **Vue DevTools** - Постоянное улучшение инструментов разработки

## Ресурсы для изучения

- [[Официальная документация Vue]]
- [[Vue School]]
- [[Vue Mastery]]
- [[Vue.js Best Practices]]
- [[Vue Component Patterns]]

## Заключение

Vue.js остается отличным выбором для разработки пользовательских интерфейсов в 2025 году, особенно для команд, ценящих простоту и гибкость. Его постепенная сложность позволяет использовать его как для небольших проектов, так и для крупных приложений, особенно с учетом российских реалий.

> [!tip] Совет
> Используйте Composition API для новых проектов, так как он обеспечивает лучшую логическую организацию кода и переиспользуемость логики.

> [!warning] Важно
> В условиях санкционных ограничений в России, Vue может быть хорошим выбором благодаря своей меньшей зависимости от западных экосистем по сравнению с другими фреймворками.
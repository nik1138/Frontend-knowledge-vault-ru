# Vue.js

Vue.js — это прогрессивный JavaScript-фреймворк для создания пользовательских интерфейсов. Vue разработан так, чтобы быть постепенно внедряемым, и может использоваться как для простых одностраничных приложений, так и для сложных корпоративных решений.

## Основные характеристики

### Прогрессивная архитектура
Vue спроектирован как постепенно внедряемый фреймворк, что позволяет использовать его на разных уровнях сложности.

```javascript
// Простое использование
const app = Vue.createApp({
  data() {
    return {
      message: 'Hello Vue!'
    }
  }
});

app.mount('#app');
```

### Реактивность
Vue предоставляет мощную систему реактивности, которая автоматически отслеживает изменения данных и обновляет DOM.

```javascript
// Реактивные данные
const app = Vue.createApp({
  data() {
    return {
      name: 'John',
      age: 30,
      isVisible: true
    }
  },
  computed: {
    // Вычисляемые свойства
    description() {
      return `${this.name} is ${this.age} years old`;
    }
  }
});
```

### Декларативный рендеринг
Vue использует декларативный синтаксис для связывания данных с DOM.

```html
<!-- Декларативное связывание -->
<div id="app">
  <p>{{ message }}</p>
  <p v-if="isVisible">This is visible</p>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.text }}
    </li>
  </ul>
</div>
```

## Основные концепции

### Компоненты
Vue компоненты — это переиспользуемые экземпляры с именем, которые могут быть расширены с помощью опций.

```javascript
// Определение компонента
const TodoItem = {
  template: `
    <li>
      {{ todo.text }}
      <button @click="$emit('remove')">Remove</button>
    </li>
  `,
  props: ['todo']
};

// Использование компонента
const app = Vue.createApp({
  components: {
    TodoItem
  },
  data() {
    return {
      todos: [
        { id: 1, text: 'Learn Vue' },
        { id: 2, text: 'Build something awesome' }
      ]
    }
  }
});
```

### Директивы
Vue предоставляет набор директив для декларативного управления DOM.

```html
<!-- Условная отрисовка -->
<p v-if="seen">Now you see me</p>
<p v-show="isVisible">Always rendered, but toggled</p>

<!-- Циклы -->
<ul>
  <li v-for="item in items" :key="item.id">
    {{ item.text }}
  </li>
</ul>

<!-- События -->
<button v-on:click="doSomething">Click me</button>
<button @click="doSomething">Click me (сокращение)</button>

<!-- Двустороннее связывание -->
<input v-model="message" placeholder="Edit me">
```

### Composition API
Composition API — это новый способ организации компонентов в Vue 3, который предоставляет большую гибкость.

```javascript
import { ref, computed, watch } from 'vue';

export default {
  setup() {
    // Реактивные данные
    const count = ref(0);
    const name = ref('John');
    
    // Вычисляемые свойства
    const doubled = computed(() => count.value * 2);
    
    // Методы
    const increment = () => {
      count.value++;
    };
    
    // Наблюдатели
    watch(count, (newVal, oldVal) => {
      console.log(`Count changed from ${oldVal} to ${newVal}`);
    });
    
    // Возвращаемые значения доступны в шаблоне
    return {
      count,
      name,
      doubled,
      increment
    };
  }
};
```

## Преимущества Vue

1. **Низкий порог входа** — Простота изучения и использования
2. **Гибкость** — Может использоваться как для простых, так и для сложных приложений
3. **Производительность** — Оптимизированный рендеринг и малый размер
4. **Богатая экосистема** — Vue Router, Vuex, Vue CLI и другие инструменты
5. **Отличная документация** — Понятная и подробная документация

## Связанные концепции

- [[Композиция]] - Vue поощряет композицию компонентов
- [[Разделение ответственности]] - Vue компоненты способствуют разделению ответственности
- [[Чистый код]] - Vue поощряет написание чистого, декларативного кода
- [[DRY (Don't Repeat Yourself)]] - Компоненты Vue способствуют избеганию дублирования
- [[KISS (Keep It Simple, Stupid)]] - Vue поощряет простоту через декларативный подход
- [[Интерфейсы]] - Vue компоненты определяют интерфейсы через props

## В других технологиях

### В [[ts]]
Vue 3 имеет отличную поддержку TypeScript:

```typescript
import { defineComponent, ref, computed } from 'vue';

interface User {
  id: number;
  name: string;
  email: string;
}

export default defineComponent({
  name: 'UserCard',
  props: {
    user: {
      type: Object as () => User,
      required: true
    }
  },
  setup(props) {
    const isActive = ref(false);
    
    const fullName = computed(() => {
      return props.user.name.toUpperCase();
    });
    
    return {
      isActive,
      fullName
    };
  }
});
```

### В [[js]]
Vue может использоваться с чистым JavaScript:

```javascript
const { createApp } = Vue;

createApp({
  data() {
    return {
      message: 'Hello Vue!',
      todos: [],
      newTodo: ''
    }
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim()) {
        this.todos.push({
          id: Date.now(),
          text: this.newTodo
        });
        this.newTodo = '';
      }
    },
    removeTodo(id) {
      this.todos = this.todos.filter(todo => todo.id !== id);
    }
  }
}).mount('#app');
```

### В [[html]]
Vue может быть интегрирован в существующие HTML-страницы:

```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
</head>
<body>
    <div id="app">
        <h1>{{ message }}</h1>
        <button @click="count++">Count is: {{ count }}</button>
    </div>
    
    <script>
        const { createApp, ref } = Vue;
        
        createApp({
            setup() {
                const message = ref('Hello Vue!');
                const count = ref(0);
                
                return {
                    message,
                    count
                };
            }
        }).mount('#app');
    </script>
</body>
</html>
```

## Теги
#vue #javascript #frontend #framework #components
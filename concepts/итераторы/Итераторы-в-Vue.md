---
aliases: ["Итераторы в Vue", "Vue и итераторы", "Работа с итераторами в Vue"]
tags: [vue, javascript, frontend-development, iterators, vuejs]
---

# Итераторы в Vue

## Обзор

В Vue.js итераторы реализуются через директивы шаблона, такие как `v-for`, которые позволяют отображать элементы коллекций данных. Vue предоставляет мощные и интуитивно понятные инструменты для работы с массивами и объектами, обеспечивая эффективное обновление DOM при изменении данных.

## Директива v-for

Основной способ итерации в Vue — это директива `v-for`, которая позволяет перебирать элементы массивов, объектов и числовые диапазоны.

### Итерация по массиву

```vue
<template>
  <div>
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Элемент 1' },
        { id: 2, name: 'Элемент 2' },
        { id: 3, name: 'Элемент 3' }
      ]
    }
  }
}
</script>
```

### Итерация с индексом

```vue
<template>
  <div>
    <ul>
      <li v-for="(item, index) in items" :key="item.id">
        {{ index + 1 }}. {{ item.name }}
      </li>
    </ul>
  </div>
</template>
```

### Итерация по объекту

```vue
<template>
  <div>
    <ul>
      <li v-for="(value, key) in user" :key="key">
        {{ key }}: {{ value }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: 'Иван',
        age: 30,
        email: 'ivan@example.com'
      }
    }
  }
}
</script>
```

### Итерация по числовому диапазону

```vue
<template>
  <div>
    <span v-for="n in 10" :key="n">{{ n }} </span>
  </div>
</template>
```

## Использование ключей (keys)

Как и в React, в Vue важно использовать атрибут `key` для элементов, созданных с помощью `v-for`. Это помогает Vue эффективно обновлять DOM, отслеживая изменения в списке.

```vue
<template>
  <ul>
    <!-- Хорошо: использование уникального ID в качестве ключа -->
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>
</template>
```

## Условный рендеринг в итераторах

Часто возникает необходимость условно отображать элементы списка:

```vue
<template>
  <div>
    <ul>
      <li 
        v-for="product in products" 
        :key="product.id"
        v-if="product.inStock"
      >
        {{ product.name }} - {{ product.price }}$
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      products: [
        { id: 1, name: 'Товар 1', price: 10, inStock: true },
        { id: 2, name: 'Товар 2', price: 20, inStock: false },
        { id: 3, name: 'Товар 3', price: 15, inStock: true }
      ]
    }
  }
}
</script>
```

Альтернативный подход с использованием фильтрации в JavaScript:

```vue
<template>
  <div>
    <ul>
      <li v-for="product in availableProducts" :key="product.id">
        {{ product.name }} - {{ product.price }}$
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      products: [
        { id: 1, name: 'Товар 1', price: 10, inStock: true },
        { id: 2, name: 'Товар 2', price: 20, inStock: false },
        { id: 3, name: 'Товар 3', price: 15, inStock: true }
      ]
    }
  },
  computed: {
    availableProducts() {
      return this.products.filter(product => product.inStock);
    }
  }
}
</script>
```

## Использование методов итераторов

Vue позволяет использовать методы итераторов JavaScript в шаблонах и computed свойствах:

```vue
<template>
  <div>
    <h3>Фильтрованные пользователи</h3>
    <ul>
      <li v-for="user in filteredUsers" :key="user.id">
        {{ user.name }} ({{ user.age }} лет)
      </li>
    </ul>
    
    <h3>Статистика</h3>
    <p>Всего пользователей: {{ userCount }}</p>
    <p>Средний возраст: {{ averageAge }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      users: [
        { id: 1, name: 'Иван', age: 25 },
        { id: 2, name: 'Мария', age: 30 },
        { id: 3, name: 'Алексей', age: 22 },
        { id: 4, name: 'Елена', age: 28 }
      ],
      ageFilter: 25
    }
  },
  computed: {
    filteredUsers() {
      return this.users.filter(user => user.age >= this.ageFilter);
    },
    userCount() {
      return this.filteredUsers.length;
    },
    averageAge() {
      if (this.filteredUsers.length === 0) return 0;
      const sum = this.filteredUsers.reduce((acc, user) => acc + user.age, 0);
      return Math.round(sum / this.filteredUsers.length);
    }
  }
}
</script>
```

## Итераторы в Composition API

В Composition API Vue 3 итераторы работают аналогично, но с использованием `ref` и `computed`:

```vue
<template>
  <div>
    <div v-for="item in processedItems" :key="item.id" class="item">
      <h4>{{ item.name }}</h4>
      <p>Цена: {{ item.formattedPrice }}</p>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    const items = ref([
      { id: 1, name: 'Продукт 1', price: 12.99 },
      { id: 2, name: 'Продукт 2', price: 24.50 },
      { id: 3, name: 'Продукт 3', price: 8.75 }
    ]);

    const processedItems = computed(() => {
      return items.value.map(item => ({
        ...item,
        formattedPrice: `\$${item.price.toFixed(2)}`
      }));
    });

    return {
      processedItems
    };
  }
}
</script>
```

## Итераторы и обработка событий

При работе с итераторами часто требуется обработка событий для элементов списка:

```vue
<template>
  <div>
    <ul>
      <li 
        v-for="item in items" 
        :key="item.id"
        @click="selectItem(item)"
        :class="{ selected: item.selected }"
        class="list-item"
      >
        {{ item.name }}
        <button @click.stop="removeItem(item.id)">Удалить</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      items: [
        { id: 1, name: 'Элемент 1', selected: false },
        { id: 2, name: 'Элемент 2', selected: false },
        { id: 3, name: 'Элемент 3', selected: false }
      ]
    }
  },
  methods: {
    selectItem(item) {
      // Снимаем выделение со всех элементов
      this.items.forEach(i => i.selected = false);
      // Выделяем выбранный элемент
      item.selected = true;
    },
    removeItem(id) {
      this.items = this.items.filter(item => item.id !== id);
    }
  }
}
</script>

<style scoped>
.list-item {
  cursor: pointer;
  padding: 8px;
  border: 1px solid #ccc;
  margin: 4px 0;
}

.list-item.selected {
  background-color: #e0f7fa;
  border-color: #00bcd4;
}
</style>
```

## Использование шаблонов для сложных итераций

Vue позволяет использовать элемент `<template>` с `v-for` для отображения нескольких элементов на каждой итерации:

```vue
<template>
  <div>
    <template v-for="category in categories" :key="category.id">
      <h3>{{ category.name }}</h3>
      <ul>
        <li v-for="product in category.products" :key="product.id">
          {{ product.name }} - {{ product.price }}$
        </li>
      </ul>
    </template>
  </div>
</template>

<script>
export default {
  data() {
    return {
      categories: [
        {
          id: 1,
          name: 'Электроника',
          products: [
            { id: 1, name: 'Смартфон', price: 599 },
            { id: 2, name: 'Ноутбук', price: 1299 }
          ]
        },
        {
          id: 2,
          name: 'Одежда',
          products: [
            { id: 3, name: 'Футболка', price: 19.99 },
            { id: 4, name: 'Джинсы', price: 49.99 }
          ]
        }
      ]
    }
  }
}
</script>
```

## Итераторы и асинхронные данные

При работе с асинхронными данными итераторы могут использоваться для отображения загрузки, ошибок и данных:

```vue
<template>
  <div>
    <div v-if="loading">Загрузка...</div>
    <div v-else-if="error">Ошибка: {{ error }}</div>
    <div v-else>
      <ul>
        <li v-for="user in users" :key="user.id">
          {{ user.name }} - {{ user.email }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      users: [],
      loading: true,
      error: null
    }
  },
  async mounted() {
    try {
      const response = await fetch('/api/users');
      this.users = await response.json();
    } catch (err) {
      this.error = err.message;
    } finally {
      this.loading = false;
    }
  }
}
</script>
```

## Итераторы и производительность

Для оптимизации производительности при работе с большими списками в Vue:

```vue
<template>
  <div>
    <!-- Использование v-memo для оптимизации в Vue 3.2+ -->
    <div 
      v-for="item in items" 
      :key="item.id"
      v-memo="[item.id, item.selected]"
      class="memoized-item"
    >
      <ItemComponent :item="item" />
    </div>
  </div>
</template>

<script>
import ItemComponent from './ItemComponent.vue';

export default {
  components: {
    ItemComponent
  },
  data() {
    return {
      items: [] // Большой массив элементов
    }
  }
}
</script>
```

## Практические рекомендации

- Всегда используйте уникальные ключи в `v-for`
- Используйте computed свойства для сложных преобразований данных
- Предпочитайте фильтрацию и сортировку в computed свойствах, а не в шаблонах
- Используйте `v-memo` (Vue 3.2+) для оптимизации рендеринга больших списков
- Избегайте выполнения тяжелых вычислений в шаблонах

## Связь с другими концепциями

- [[Директивы]] — специальные атрибуты для манипуляции DOM
- [[Жизненный-цикл-компонента]] — фазы существования компонента Vue
- [[Состояние]] — управление данными в приложении Vue
- [[События]] — взаимодействие между компонентами
- [[Компонентная-архитектура]] — организация приложения в виде компонентов

## Заключение

Итераторы в Vue предоставляют мощный и интуитивно понятный способ работы с коллекциями данных. Директива `v-for` в сочетании с вычисляемыми свойствами и методами позволяет создавать динамические и интерактивные интерфейсы. Понимание эффективного использования итераторов в Vue помогает создавать производительные и поддерживаемые приложения.

## См. также

- [[Итераторы-в-JavaScript]]
- [[Итераторы-в-React]]
- [[Итераторы-и-производительность]]
- [[Паттерн-итератор]]
- [[Директивы]]
- [[Жизненный-цикл-компонента]]

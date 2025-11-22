---
aliases: ["Полиморфизм в Vue", "Vue Полиморфизм"]
tags: ["#vue", "#polymorphism", "#frontend", "#components"]
---

# Полиморфизм-в-Vue

## Введение

Полиморфизм в Vue.js проявляется через гибкие компоненты, которые могут адаптироваться к различным сценариям использования, принимая разные типы данных или отображая разное содержимое при сохранении одинакового интерфейса. Vue предоставляет несколько мощных механизмов для реализации полиморфного поведения: слоты (slots), динамические компоненты, компоненты высшего порядка и директивы.

## Типы полиморфизма в Vue

### 1. Слоты (Slots) как основа полиморфизма

Слоты в Vue являются основным механизмом для создания полиморфных компонентов. Они позволяют передавать разное содержимое в один и тот же компонент.

```vue
<!-- BaseModal.vue -->
<template>
  <div class="modal-overlay" v-if="isOpen" @click="closeModal">
    <div class="modal-content" @click.stop>
      <div class="modal-header">
        <slot name="header">
          <h2>{{ title }}</h2>
        </slot>
        <button @click="closeModal">×</button>
      </div>
      <div class="modal-body">
        <slot name="body">
          <!-- Содержимое по умолчанию -->
        </slot>
      </div>
      <div class="modal-footer">
        <slot name="footer">
          <button @click="closeModal">Закрыть</button>
        </slot>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'BaseModal',
  props: {
    isOpen: Boolean,
    title: String
  },
  methods: {
    closeModal() {
      this.$emit('close');
    }
  }
}
</script>
```

```vue
<!-- Использование с разными содержимым -->
<template>
  <div>
    <!-- Модальное окно для пользователя -->
    <BaseModal :is-open="showUserModal" @close="showUserModal = false" title="Детали пользователя">
      <template #body>
        <UserProfile :user="selectedUser" />
      </template>
      <template #footer>
        <button @click="updateUser">Сохранить</button>
        <button @click="deleteUser">Удалить</button>
      </template>
    </BaseModal>

    <!-- Модальное окно для продукта -->
    <BaseModal :is-open="showProductModal" @close="showProductModal = false" title="Детали продукта">
      <template #body>
        <ProductDetails :product="selectedProduct" />
      </template>
      <template #footer>
        <button @click="updateProduct">Обновить</button>
        <button @click="showProductModal = false">Отмена</button>
      </template>
    </BaseModal>
  </div>
</template>
```

### 2. Слоты с ограниченной областью видимости (Scoped Slots)

Scoped slots позволяют передавать данные из компонента в слот, обеспечивая полиморфное поведение с доступом к внутреннему состоянию компонента.

```vue
<!-- ListRenderer.vue -->
<template>
  <div>
    <div v-for="(item, index) in items" :key="index" class="list-item">
      <slot :item="item" :index="index" :isSelected="isSelected(item)">
        {{ item.name }}
      </slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ListRenderer',
  props: {
    items: Array,
    selectedId: [String, Number]
  },
  methods: {
    isSelected(item) {
      return item.id === this.selectedId;
    }
  }
}
</script>
```

```vue
<!-- Использование с разными представлениями -->
<template>
  <div>
    <!-- Список пользователей -->
    <ListRenderer :items="users" :selected-id="selectedUserId">
      <template #default="{ item, index, isSelected }">
        <div :class="{ 'selected': isSelected, 'user-item': true }">
          <img :src="item.avatar" :alt="item.name" />
          <span>{{ item.name }} ({{ item.role }})</span>
        </div>
      </template>
    </ListRenderer>

    <!-- Список задач -->
    <ListRenderer :items="tasks" :selected-id="selectedTaskId">
      <template #default="{ item, index, isSelected }">
        <div :class="{ 'selected': isSelected, 'task-item': true }">
          <input type="checkbox" :checked="item.completed" @change="toggleTask(item)" />
          <span :class="{ 'completed': item.completed }">{{ item.title }}</span>
          <span class="due-date">{{ item.dueDate }}</span>
        </div>
      </template>
    </ListRenderer>
  </div>
</template>
```

### 3. Динамические компоненты

Vue позволяет использовать динамические компоненты через элемент `<component>`, что является формой полиморфизма на уровне компонентов.

```vue
<template>
  <div>
    <!-- Рендеринг разных компонентов в зависимости от типа -->
    <component 
      :is="currentComponent" 
      v-bind="componentProps"
      @action="handleAction"
    />
  </div>
</template>

<script>
import UserProfile from './UserProfile.vue'
import ProductCard from './ProductCard.vue'
import ArticlePreview from './ArticlePreview.vue'

export default {
  name: 'DynamicRenderer',
  components: {
    UserProfile,
    ProductCard,
    ArticlePreview
  },
  data() {
    return {
      currentView: 'user', // или 'product', 'article'
      userData: { name: 'Иван', email: 'ivan@example.com' },
      productData: { name: 'Телефон', price: 29999 },
      articleData: { title: 'Новости', content: 'Содержимое статьи...' }
    }
  },
  computed: {
    currentComponent() {
      const componentMap = {
        user: 'UserProfile',
        product: 'ProductCard',
        article: 'ArticlePreview'
      }
      return componentMap[this.currentView] || 'UserProfile'
    },
    componentProps() {
      const propsMap = {
        user: { user: this.userData },
        product: { product: this.productData },
        article: { article: this.articleData }
      }
      return propsMap[this.currentView] || {}
    }
  },
  methods: {
    handleAction(action) {
      console.log('Действие:', action)
    }
  }
}
</script>
```

### 4. Компоненты высшего порядка (HOC) в Vue

Хотя HOC менее распространены в Vue, чем в React, их можно реализовать для создания полиморфного поведения.

```javascript
// withDataFetching.js - HOC для загрузки данных
import { defineAsyncComponent } from 'vue'

export function withDataFetching(WrappedComponent, fetchData) {
  return {
    name: `With${WrappedComponent.name}Data`,
    data() {
      return {
        data: null,
        loading: true,
        error: null
      }
    },
    async created() {
      try {
        this.data = await fetchData()
      } catch (err) {
        this.error = err
      } finally {
        this.loading = false
      }
    },
    render() {
      return h(WrappedComponent, {
        ...this.$attrs,
        data: this.data,
        loading: this.loading,
        error: this.error
      })
    }
  }
}

// Использование
const UserListWithData = withDataFetching(UserList, () => fetch('/api/users').then(r => r.json()))
const ProductListWithData = withDataFetching(ProductList, () => fetch('/api/products').then(r => r.json()))
```

### 5. Полиморфизм через директивы

Пользовательские директивы в Vue также могут демонстрировать полиморфное поведение.

```javascript
// v-focus-if.js - директива для условного фокуса
export default {
  mounted(el, binding) {
    if (binding.value) {
      el.focus()
    }
  },
  updated(el, binding) {
    if (binding.value && !binding.oldValue) {
      el.focus()
    }
  }
}
```

```vue
<template>
  <div>
    <!-- Фокус в зависимости от состояния -->
    <input v-focus-if="shouldFocus" v-model="inputValue" />
    <textarea v-focus-if="editMode" v-model="textValue" />
  </div>
</template>
```

## Практические примеры полиморфных компонентов

### 1. Универсальный карточный компонент

```vue
<!-- Card.vue -->
<template>
  <div class="card" :class="cardClasses">
    <div v-if="$slots.header || title" class="card-header">
      <slot name="header">
        <h3>{{ title }}</h3>
      </slot>
    </div>
    
    <div class="card-body">
      <slot />
    </div>
    
    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer" />
    </div>
  </div>
</template>

<script>
export default {
  name: 'Card',
  props: {
    title: String,
    type: {
      type: String,
      default: 'default',
      validator: value => ['default', 'primary', 'success', 'warning', 'danger'].includes(value)
    },
    shadow: {
      type: Boolean,
      default: true
    }
  },
  computed: {
    cardClasses() {
      return [
        `card--${this.type}`,
        { 'card--shadow': this.shadow }
      ]
    }
  }
}
</script>
```

```vue
<!-- Использование универсальной карточки -->
<template>
  <div>
    <!-- Карточка пользователя -->
    <Card title="Профиль пользователя" type="primary">
      <UserProfile :user="currentUser" />
      <template #footer>
        <button @click="editProfile">Редактировать</button>
      </template>
    </Card>

    <!-- Карточка продукта -->
    <Card type="success">
      <template #header>
        <h3>{{ product.name }}</h3>
        <span class="price">{{ product.price }} руб.</span>
      </template>
      
      <ProductImage :src="product.image" />
      <p>{{ product.description }}</p>
      
      <template #footer>
        <button @click="addToCart">Добавить в корзину</button>
      </template>
    </Card>
  </div>
</template>
```

### 2. Полиморфный список с сортировкой

```vue
<!-- SortableList.vue -->
<template>
  <div class="sortable-list">
    <div class="list-controls">
      <select v-model="sortBy" @change="sortItems">
        <option value="">Сортировать по...</option>
        <option v-for="field in sortableFields" :key="field" :value="field">
          {{ field }}
        </option>
      </select>
      <button @click="reverseOrder = !reverseOrder">
        {{ reverseOrder ? 'По убыванию' : 'По возрастанию' }}
      </button>
    </div>
    
    <div class="list-items">
      <slot 
        :items="sortedItems" 
        :sort-by="sortBy"
        :reverse-order="reverseOrder"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'SortableList',
  props: {
    items: {
      type: Array,
      required: true
    },
    sortableFields: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      sortBy: '',
      reverseOrder: false
    }
  },
  computed: {
    sortedItems() {
      if (!this.sortBy) return this.items
      
      return [...this.items].sort((a, b) => {
        const aValue = this.getNestedValue(a, this.sortBy)
        const bValue = this.getNestedValue(b, this.sortBy)
        
        if (aValue < bValue) return this.reverseOrder ? 1 : -1
        if (aValue > bValue) return this.reverseOrder ? -1 : 1
        return 0
      })
    }
  },
  methods: {
    getNestedValue(obj, path) {
      return path.split('.').reduce((current, key) => current?.[key], obj)
    },
    sortItems() {
      // Сортировка уже происходит через computed
    }
  }
}
</script>
```

```vue
<!-- Использование с разными типами данных -->
<template>
  <div>
    <!-- Список пользователей -->
    <SortableList 
      :items="users" 
      :sortable-fields="['name', 'email', 'createdAt']"
    >
      <template #default="{ items }">
        <div v-for="user in items" :key="user.id" class="user-item">
          <h4>{{ user.name }}</h4>
          <p>{{ user.email }}</p>
        </div>
      </template>
    </SortableList>

    <!-- Список продуктов -->
    <SortableList 
      :items="products" 
      :sortable-fields="['name', 'price', 'category']"
    >
      <template #default="{ items }">
        <div v-for="product in items" :key="product.id" class="product-item">
          <h4>{{ product.name }}</h4>
          <p>{{ product.price }} руб.</p>
          <span class="category">{{ product.category }}</span>
        </div>
      </template>
    </SortableList>
  </div>
</template>
```

### 3. Полиморфный компонент формы

```vue
<!-- FormField.vue -->
<template>
  <div class="form-field" :class="fieldClasses">
    <label v-if="label" :for="id">{{ label }}</label>
    
    <component 
      :is="componentType" 
      :id="id"
      v-bind="fieldProps"
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
      @change="$emit('change', $event)"
    >
      <option 
        v-for="option in options" 
        :key="option.value" 
        :value="option.value"
        :selected="option.value === modelValue"
      >
        {{ option.label }}
      </option>
    </component>
    
    <div v-if="error" class="error-message">{{ error }}</div>
    <div v-if="helperText" class="helper-text">{{ helperText }}</div>
  </div>
</template>

<script>
export default {
  name: 'FormField',
  props: {
    type: {
      type: String,
      default: 'text',
      validator: value => [
        'text', 'email', 'password', 'number', 'textarea', 'select', 'checkbox', 'radio'
      ].includes(value)
    },
    label: String,
    modelValue: [String, Number, Boolean, Array],
    placeholder: String,
    required: Boolean,
    error: String,
    helperText: String,
    options: {
      type: Array,
      default: () => []
    },
    id: {
      type: String,
      default: () => `field-${Math.random().toString(36).substr(2, 9)}`
    }
  },
  emits: ['update:modelValue', 'change'],
  computed: {
    componentType() {
      switch (this.type) {
        case 'textarea': return 'textarea'
        case 'select': return 'select'
        case 'checkbox': return 'input'
        case 'radio': return 'input'
        default: return 'input'
      }
    },
    fieldProps() {
      const props = {
        type: this.type !== 'select' && this.type !== 'textarea' ? this.type : undefined,
        placeholder: this.placeholder,
        required: this.required,
        name: this.id
      }
      
      if (this.type === 'checkbox' || this.type === 'radio') {
        props.checked = this.modelValue
      }
      
      return props
    },
    fieldClasses() {
      return {
        'form-field--error': !!this.error,
        'form-field--required': this.required
      }
    }
  }
}
</script>
```

## Полиморфизм в Composition API

Composition API также позволяет реализовывать полиморфное поведение через хуки и реактивные данные:

```vue
<script setup>
import { ref, computed, onMounted } from 'vue'

// Универсальный хук для загрузки данных
function useDataFetcher(fetchFunction, initialData = null) {
  const data = ref(initialData)
  const loading = ref(true)
  const error = ref(null)

  const fetchData = async () => {
    try {
      loading.value = true
      error.value = null
      data.value = await fetchFunction()
    } catch (err) {
      error.value = err
    } finally {
      loading.value = false
    }
  }

  onMounted(fetchData)

  return {
    data,
    loading,
    error,
    refetch: fetchData
  }
}

// Использование с разными API
const { data: users, loading: usersLoading, refetch: refetchUsers } = 
  useDataFetcher(() => fetch('/api/users').then(r => r.json()))

const { data: products, loading: productsLoading, refetch: refetchProducts } = 
  useDataFetcher(() => fetch('/api/products').then(r => r.json()))
</script>
```

## Преимущества полиморфизма в Vue

1. **Повторное использование компонентов** - один компонент может использоваться в разных контекстах
2. **Гибкость** - возможность адаптировать компоненты под разные сценарии использования
3. **Снижение дублирования кода** - уменьшение необходимости создания множества похожих компонентов
4. **Улучшенная тестируемость** - возможность тестировать компоненты с разными типами данных
5. **Лучшая поддерживаемость** - изменения в базовом компоненте автоматически применяются ко всем экземплярам

## Заключение

Полиморфизм в Vue.js позволяет создавать гибкие и переиспользуемые компоненты, которые могут адаптироваться к различным сценариям использования. Через слоты, динамические компоненты, scoped slots и другие механизмы Vue предоставляет мощные инструменты для реализации полиморфного поведения.

Это делает код более чистым, понятным и поддерживаемым, позволяя создавать мощные и гибкие UI-библиотеки.

## См. также

- [[Компонентный подход]]
- [[Слоты]]
- [[Директивы]]
- [[Composition API]]
- [[Дизайн-системы]]
---
aliases: ["Композиция в Vue", "Vue композиция компонентов", "Композиция в Vue приложениях"]
tags: ["#frontend", "#vue", "#composition", "#components", "#architecture", "#javascript"]
---

# Композиция в Vue

## Введение

**Композиция в Vue** — это подход к построению компонентов, который позволяет объединять более простые компоненты в более сложные структуры. Vue поддерживает несколько подходов к композиции: через традиционный Options API и через более современный Composition API, представленный в Vue 3.

## Принципы композиции в Vue

### 1. Компоненты как строительные блоки
Vue позволяет создавать компоненты, которые могут быть использованы как строительные блоки для более сложных интерфейсов. Каждый компонент инкапсулирует свою логику, разметку и стили.

### 2. Props для передачи данных
Props — это механизм передачи данных от родительского компонента к дочернему, что позволяет создавать гибкие и переиспользуемые компоненты.

### 3. Слоты для композиции
Слоты (slots) — мощный механизм композиции, который позволяет передавать содержимое из родительского компонента в дочерний.

## Практические примеры композиции в Vue

### Пример 1: Простая композиция через props

```vue
<!-- Button.vue -->
<template>
  <button 
    :class="['btn', `btn-${variant}`, { 'btn-disabled': disabled }]"
    :disabled="disabled"
    @click="handleClick"
  >
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: 'Button',
  props: {
    variant: {
      type: String,
      default: 'primary'
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  emits: ['click'],
  methods: {
    handleClick(event) {
      if (!this.disabled) {
        this.$emit('click', event);
      }
    }
  }
};
</script>

<style scoped>
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary { background-color: #007bff; color: white; }
.btn-secondary { background-color: #6c757d; color: white; }
.btn-disabled { opacity: 0.6; cursor: not-allowed; }
</style>
```

```vue
<!-- ButtonGroup.vue -->
<template>
  <div class="btn-group">
    <Button
      v-for="(action, index) in actions"
      :key="index"
      :variant="action.variant"
      :disabled="action.disabled"
      @click="action.handler"
    >
      {{ action.label }}
    </Button>
  </div>
</template>

<script>
import Button from './Button.vue';

export default {
  name: 'ButtonGroup',
  components: {
    Button
  },
  props: {
    actions: {
      type: Array,
      required: true
    }
  }
};
</script>

<style scoped>
.btn-group {
  display: flex;
  gap: 8px;
}
</style>
```

### Пример 2: Использование слотов для композиции

```vue
<!-- Card.vue -->
<template>
  <div class="card">
    <div v-if="$slots.header" class="card-header">
      <slot name="header"></slot>
    </div>
    <div class="card-body">
      <slot></slot>
    </div>
    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Card'
};
</script>

<style scoped>
.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.card-header, .card-footer {
  padding: 12px;
  background-color: #f8f9fa;
}

.card-body {
  padding: 16px;
}
</style>
```

```vue
<!-- UserProfileCard.vue -->
<template>
  <Card>
    <template #header>
      <h2>Профиль пользователя</h2>
    </template>
    
    <div>
      <p>Имя: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
      <p>Роль: {{ user.role }}</p>
    </div>
    
    <template #footer>
      <Button variant="secondary" @click="editProfile">Редактировать</Button>
      <Button variant="primary" @click="saveProfile">Сохранить</Button>
    </template>
  </Card>
</template>

<script>
import Card from './Card.vue';
import Button from './Button.vue';

export default {
  name: 'UserProfileCard',
  components: {
    Card,
    Button
  },
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  methods: {
    editProfile() {
      // Логика редактирования профиля
    },
    saveProfile() {
      // Логика сохранения профиля
    }
  }
};
</script>
```

### Пример 3: Композиция с Composition API

```vue
<!-- UserList.vue -->
<template>
  <div class="user-list">
    <div v-if="loading" class="loading">Загрузка...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <ul v-else>
      <li v-for="user in users" :key="user.id" class="user-item">
        {{ user.name }} - {{ user.email }}
      </li>
    </ul>
    <button @click="loadUsers" :disabled="loading">Обновить</button>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';

export default {
  name: 'UserList',
  setup() {
    const users = ref([]);
    const loading = ref(false);
    const error = ref(null);

    const fetchUsers = async () => {
      try {
        loading.value = true;
        error.value = null;
        
        // Имитация API вызова
        const response = await fetch('/api/users');
        if (!response.ok) throw new Error('Ошибка загрузки пользователей');
        
        users.value = await response.json();
      } catch (err) {
        error.value = err.message;
      } finally {
        loading.value = false;
      }
    };

    onMounted(() => {
      fetchUsers();
    });

    return {
      users,
      loading,
      error,
      loadUsers: fetchUsers
    };
  }
};
</script>
```

### Пример 4: Композиция с Provide/Inject

```vue
<!-- ThemeProvider.vue -->
<template>
  <div class="theme-provider">
    <slot></slot>
  </div>
</template>

<script>
import { provide, reactive } from 'vue';

export default {
  name: 'ThemeProvider',
  props: {
    initialTheme: {
      type: String,
      default: 'light'
    }
  },
  setup(props) {
    const theme = reactive({
      current: props.initialTheme,
      toggle: () => {
        theme.current = theme.current === 'light' ? 'dark' : 'light';
      }
    });

    provide('theme', theme);
  }
};
</script>
```

```vue
<!-- ThemedButton.vue -->
<template>
  <button 
    :class="['themed-button', `theme-${theme.current}`]"
    @click="handleClick"
  >
    <slot></slot>
  </button>
</template>

<script>
import { inject } from 'vue';

export default {
  name: 'ThemedButton',
  setup() {
    const theme = inject('theme');
    
    const handleClick = () => {
      // Логика кнопки
    };
    
    return {
      theme,
      handleClick
    };
  }
};
</script>

<style scoped>
.themed-button {
  padding: 8px 16px;
  border: 1px solid;
  border-radius: 4px;
}

.theme-light {
  background-color: white;
  color: black;
  border-color: #ccc;
}

.theme-dark {
  background-color: #333;
  color: white;
  border-color: #666;
}
</style>
```

## Composition API и композиция

Composition API предоставляет более гибкий способ организации логики компонентов и их композиции:

### 1. Composables (Composition Functions)
Функции, которые инкапсулируют и повторно используют логику состояния:

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  
  const increment = () => {
    count.value++;
  };
  
  const decrement = () => {
    count.value--;
  };
  
  const reset = () => {
    count.value = initialValue;
  };
  
  const isPositive = computed(() => count.value > 0);
  
  return {
    count,
    increment,
    decrement,
    reset,
    isPositive
  };
}
```

```vue
<!-- CounterComponent.vue -->
<template>
  <div class="counter">
    <p>Счетчик: {{ count }}</p>
    <p>Положительный: {{ isPositive ? 'Да' : 'Нет' }}</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
    <button @click="reset">Сброс</button>
  </div>
</template>

<script>
import { useCounter } from '@/composables/useCounter';

export default {
  name: 'CounterComponent',
  setup() {
    const { count, increment, decrement, reset, isPositive } = useCounter(0);
    
    return {
      count,
      increment,
      decrement,
      reset,
      isPositive
    };
  }
};
</script>
```

### 2. Композиция компонентов с Composables

```vue
<!-- SearchableUserList.vue -->
<template>
  <div class="searchable-user-list">
    <input 
      v-model="searchQuery" 
      placeholder="Поиск пользователей..." 
      class="search-input"
    />
    <UserList 
      :users="filteredUsers" 
      :loading="loading" 
      :error="error" 
    />
  </div>
</template>

<script>
import { computed } from 'vue';
import { useUsers } from '@/composables/useUsers';
import { useSearch } from '@/composables/useSearch';
import UserList from './UserList.vue';

export default {
  name: 'SearchableUserList',
  components: {
    UserList
  },
  setup() {
    // Используем композицию composables
    const { users, loading, error, loadUsers } = useUsers();
    const { query: searchQuery, filteredItems: filteredUsers } = useSearch(users, 'name');
    
    return {
      searchQuery,
      filteredUsers,
      loading,
      error
    };
  }
};
</script>
```

## Паттерны композиции в Vue

### 1. Container/Presenter Pattern
Разделение компонентов на контейнерные (управляющие данными) и презентационные (отвечающие за отображение):

```vue
<!-- Presentational Component - UserCard.vue -->
<template>
  <div class="user-card">
    <h3>{{ user.name }}</h3>
    <p>{{ user.email }}</p>
    <div class="user-actions">
      <button @click="$emit('edit', user)">Редактировать</button>
      <button @click="$emit('delete', user.id)">Удалить</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserCard',
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  emits: ['edit', 'delete']
};
</script>
```

```vue
<!-- Container Component - UserCardContainer.vue -->
<template>
  <UserCard 
    v-for="user in users" 
    :key="user.id" 
    :user="user" 
    @edit="handleEdit" 
    @delete="handleDelete" 
  />
</template>

<script>
import { ref, onMounted } from 'vue';
import UserCard from './UserCard.vue';

export default {
  name: 'UserCardContainer',
  components: {
    UserCard
  },
  setup() {
    const users = ref([]);
    
    const loadUsers = async () => {
      // Загрузка пользователей
    };
    
    const handleEdit = (user) => {
      // Обработка редактирования
    };
    
    const handleDelete = (userId) => {
      // Обработка удаления
    };
    
    onMounted(() => {
      loadUsers();
    });
    
    return {
      users,
      handleEdit,
      handleDelete
    };
  }
};
</script>
```

### 2. Higher-Order Components (HOC) в Vue
Хотя Vue не имеет встроенного HOC, можно создавать функции, возвращающие компоненты:

```javascript
// hoc/withLoading.js
import { h } from 'vue';

export function withLoading(WrappedComponent) {
  return {
    name: `WithLoading${WrappedComponent.name}`,
    props: {
      loading: {
        type: Boolean,
        default: false
      }
    },
    render() {
      if (this.loading) {
        return h('div', { class: 'loading' }, 'Загрузка...');
      }
      return h(WrappedComponent, {
        ...this.$props,
        ...this.$attrs
      });
    }
  };
}
```

### 3. Renderless Components
Компоненты, которые не рендерят DOM, а предоставляют логику:

```vue
<!-- DataFetcher.vue -->
<template>
  <slot 
    :data="data" 
    :loading="loading" 
    :error="error" 
    :refetch="refetch" 
  />
</template>

<script>
import { ref } from 'vue';

export default {
  name: 'DataFetcher',
  props: {
    url: {
      type: String,
      required: true
    }
  },
  setup(props) {
    const data = ref(null);
    const loading = ref(false);
    const error = ref(null);

    const fetchData = async () => {
      loading.value = true;
      error.value = null;
      
      try {
        const response = await fetch(props.url);
        if (!response.ok) throw new Error('Ошибка запроса');
        data.value = await response.json();
      } catch (err) {
        error.value = err.message;
      } finally {
        loading.value = false;
      }
    };

    const refetch = () => {
      fetchData();
    };

    fetchData();

    return {
      data,
      loading,
      error,
      refetch
    };
  }
};
</script>
```

```vue
<!-- Использование renderless компонента -->
<template>
  <DataFetcher url="/api/users">
    <template #default="{ data: users, loading, error, refetch }">
      <div v-if="loading">Загрузка...</div>
      <div v-else-if="error">{{ error }}</div>
      <ul v-else>
        <li v-for="user in users" :key="user.id">
          {{ user.name }}
        </li>
      </ul>
      <button @click="refetch">Обновить</button>
    </template>
  </DataFetcher>
</template>

<script>
import DataFetcher from './DataFetcher.vue';

export default {
  components: {
    DataFetcher
  }
};
</script>
```

## Рекомендации по композиции в Vue

### 1. Используйте слоты для гибкой композиции
Слоты позволяют создавать очень гибкие компоненты:

```vue
<!-- Modal.vue -->
<template>
  <div v-if="isOpen" class="modal-overlay" @click="closeOnOverlayClick">
    <div class="modal" @click.stop>
      <div class="modal-header">
        <slot name="header"></slot>
        <button @click="close" class="close-btn">×</button>
      </div>
      <div class="modal-body">
        <slot></slot>
      </div>
      <div class="modal-footer">
        <slot name="footer"></slot>
      </div>
    </div>
  </div>
</template>
```

### 2. Организуйте компоненты по функциональности
Создавайте логические группы компонентов:

```
components/
├── ui/                 # Базовые UI компоненты
│   ├── Button.vue
│   ├── Input.vue
│   └── Card.vue
├── layout/             # Компоненты макета
│   ├── Header.vue
│   ├── Sidebar.vue
│   └── Footer.vue
└── features/           # Компоненты с бизнес-логикой
    ├── UserCard.vue
    └── SearchForm.vue
```

### 3. Используйте TypeScript для лучшей типизации
```vue
<script setup lang="ts">
interface User {
  id: number;
  name: string;
  email: string;
}

interface Props {
  user: User;
  editable?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  editable: false
});

const emit = defineEmits<{
  updateUser: [user: User];
  deleteUser: [id: number];
}>();
</script>
```

## Контрольные вопросы

1. В чем разница между Options API и Composition API в контексте композиции?
2. Как использовать слоты для создания гибких компонентов?
3. Что такое Composables и как они улучшают композицию?
4. Как обеспечить типобезопасность в композиции компонентов?
5. Какие паттерны композиции вы знаете и когда их использовать?

## См. также

- [[Композиция компонентов]]
- [[Агрегация компонентов]]
- [[Vue компоненты]]
- [[Composition API в Vue]]
- [[Слоты в Vue]]

## Внешние ресурсы

- [Vue.js Guide - Component Composition](https://vuejs.org/guide/components/composition.html)
- [Vue 3 Composition API Guide](https://vuejs.org/api/composition-api.html)
- [Vue Style Guide - Component Composition](https://vuejs.org/style-guide/rules-recommended.html#component-composition)
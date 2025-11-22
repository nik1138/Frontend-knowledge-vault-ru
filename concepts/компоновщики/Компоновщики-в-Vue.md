---
aliases: ["Компоновщики в Vue", "Composite Pattern Vue", "Vue Composite"]
tags: ["#vue", "#javascript", "#design-patterns", "#frontend", "#component-architecture"]
---

# Компоновщики в Vue

Паттерн **Компоновщик** в Vue.js реализуется через систему компонентов, которая позволяет создавать сложные интерфейсы из простых и повторно используемых элементов. Vue предоставляет мощные инструменты для композиции компонентов, что делает его идеальной реализацией паттерна Компоновщик в веб-разработке.

## Основы компоновщика в Vue

Vue.js построен на концепции компонентов, которые могут включать другие компоненты, создавая древовидную структуру интерфейса. Это позволяет:

- Создавать иерархические структуры UI
- Повторно использовать компоненты
- Управлять сложными взаимодействиями между элементами
- Разделять ответственность между компонентами

## Типы компонентов в Vue

### Компоненты с использованием Options API

```vue
<!-- Button.vue -->
<template>
  <button 
    :class="buttonClass"
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
    }
  },
  computed: {
    buttonClass() {
      return `btn btn-${this.variant}`;
    }
  },
  methods: {
    handleClick(event) {
      this.$emit('click', event);
    }
  }
};
</script>
```

```vue
<!-- ButtonGroup.vue -->
<template>
  <div class="button-group">
    <slot></slot>
  </div>
</template>

<script>
export default {
  name: 'ButtonGroup'
};
</script>
```

### Компоненты с использованием Composition API

```vue
<!-- Card.vue -->
<template>
  <div class="card" :class="{ 'card--expanded': isExpanded }">
    <div class="card__header" @click="toggleExpand">
      <h3>{{ title }}</h3>
      <span class="card__toggle">{{ isExpanded ? '−' : '+' }}</span>
    </div>
    <div v-show="isExpanded" class="card__content">
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const props = defineProps({
  title: String
});

const isExpanded = ref(false);

const toggleExpand = () => {
  isExpanded.value = !isExpanded.value;
};
</script>
```

## Использование слотов (Slots)

Слоты в Vue - это мощный механизм, поддерживающий паттерн Компоновщик:

```vue
<!-- Layout.vue -->
<template>
  <div class="layout">
    <header class="layout__header">
      <slot name="header"></slot>
    </header>
    <main class="layout__main">
      <slot></slot> <!-- Слот по умолчанию -->
    </main>
    <footer class="layout__footer">
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

```vue
<!-- App.vue -->
<template>
  <Layout>
    <template #header>
      <h1>Заголовок приложения</h1>
    </template>
    
    <div>Основное содержимое</div>
    <Button>Нажми меня</Button>
    
    <template #footer>
      <p>© 2023 Все права защищены</p>
    </template>
  </Layout>
</template>

<script setup>
import Layout from './Layout.vue';
import Button from './Button.vue';
</script>
```

## Композиция компонентов

Vue поощряет композицию компонентов через их регистрацию и использование:

```vue
<!-- Dashboard.vue -->
<template>
  <div class="dashboard">
    <Widget 
      v-for="widget in widgets" 
      :key="widget.id"
      :title="widget.title"
      :type="widget.type"
    >
      <component 
        :is="getWidgetComponent(widget.type)" 
        :data="widget.data" 
      />
    </Widget>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import Widget from './Widget.vue';
import ChartWidget from './ChartWidget.vue';
import TableWidget from './TableWidget.vue';

const widgets = ref([
  { id: 1, title: 'График продаж', type: 'chart', data: {} },
  { id: 2, title: 'Таблица данных', type: 'table', data: {} }
]);

const widgetComponents = {
  chart: ChartWidget,
  table: TableWidget
};

const getWidgetComponent = (type) => widgetComponents[type] || 'div';
</script>
```

## Пример сложной композиции

```vue
<!-- TreeView.vue -->
<template>
  <ul class="tree-view">
    <TreeNode 
      v-for="node in treeData" 
      :key="node.id"
      :node="node"
    />
  </ul>
</template>

<script setup>
import { defineProps } from 'vue';
import TreeNode from './TreeNode.vue';

defineProps({
  treeData: {
    type: Array,
    required: true
  }
});
</script>
```

```vue
<!-- TreeNode.vue -->
<template>
  <li class="tree-node">
    <div class="tree-node__header" @click="toggleExpanded">
      <span class="tree-node__icon">{{ expanded ? '▼' : '▶' }}</span>
      <span class="tree-node__label">{{ node.label }}</span>
    </div>
    <ul v-show="expanded" class="tree-node__children">
      <TreeNode 
        v-for="child in node.children" 
        :key="child.id"
        :node="child"
      />
    </ul>
  </li>
</template>

<script setup>
import { ref, defineProps } from 'vue';

const props = defineProps({
  node: {
    type: Object,
    required: true
  }
});

const expanded = ref(false);

const toggleExpanded = () => {
  expanded.value = !expanded.value;
};
</script>
```

## Паттерны композиции в Vue

### Scoped Slots

```vue
<!-- List.vue -->
<template>
  <ul class="list">
    <li 
      v-for="(item, index) in items" 
      :key="item.id"
      class="list-item"
    >
      <slot 
        :item="item" 
        :index="index"
        :isEven="index % 2 === 0"
      >
        {{ item.name }}
      </slot>
    </li>
  </ul>
</template>

<script setup>
defineProps({
  items: Array
});
</script>
```

```vue
<!-- Использование scoped slots -->
<template>
  <List :items="users">
    <template #default="{ item, index, isEven }">
      <div 
        class="user-item" 
        :class="{ 'user-item--even': isEven }"
      >
        <span class="user-index">{{ index + 1 }}.</span>
        <strong>{{ item.name }}</strong>
        <span class="user-email">{{ item.email }}</span>
      </div>
    </template>
  </List>
</template>
```

### Mixins и Composables

```javascript
// useApi.js
import { ref } from 'vue';

export function useApi() {
  const data = ref(null);
  const loading = ref(false);
  const error = ref(null);

  const fetchData = async (url) => {
    loading.value = true;
    error.value = null;
    
    try {
      const response = await fetch(url);
      data.value = await response.json();
    } catch (err) {
      error.value = err;
    } finally {
      loading.value = false;
    }
  };

  return {
    data,
    loading,
    error,
    fetchData
  };
}
```

```vue
<!-- UserList.vue -->
<template>
  <div class="user-list">
    <div v-if="loading">Загрузка...</div>
    <div v-else-if="error">Ошибка: {{ error.message }}</div>
    <ul v-else>
      <li v-for="user in data" :key="user.id">
        {{ user.name }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { onMounted } from 'vue';
import { useApi } from './composables/useApi';

const { data, loading, error, fetchData } = useApi();

onMounted(() => {
  fetchData('/api/users');
});
</script>
```

## Provide/Inject

Механизм provide/inject позволяет передавать данные через несколько уровней компонентов:

```vue
<!-- Parent.vue -->
<template>
  <div>
    <ChildComponent />
  </div>
</template>

<script setup>
import { provide, ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const theme = ref('dark');
const updateTheme = (newTheme) => {
  theme.value = newTheme;
};

provide('theme', theme);
provide('updateTheme', updateTheme);
</script>
```

```vue
<!-- ChildComponent.vue -->
<template>
  <div :class="`component--${theme}`">
    <button @click="updateTheme(theme === 'dark' ? 'light' : 'dark')">
      Переключить тему
    </button>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const theme = inject('theme');
const updateTheme = inject('updateTheme');
</script>
```

## Преимущества использования паттерна Компоновщик в Vue

- **Повторное использование**: компоненты можно использовать в разных частях приложения
- **Изоляция**: каждый компонент изолирован и имеет свою область видимости
- **Гибкость**: через слоты можно создавать гибкие компоненты
- **Управление состоянием**: каждая часть может управлять своим состоянием

## Практические рекомендации

1. **Используйте семантические названия компонентов** - они должны ясно описывать назначение
2. **Ограничьте глубину вложенности** - слишком глубокие иерархии сложны в отладке
3. **Используйте TypeScript** для лучшей типизации компонентов
4. **Разделяйте ответственность** - компоненты должны иметь одну зону ответственности

## Связь с другими концепциями

- [[Компонентная-архитектура]] - архитектурный подход
- [[Принципы-программирования]] - основы проектирования
- [[Жизненный-цикл-компонента]] - управление жизненным циклом
- [[Хуки]] - для управления логикой (аналог в React)
- [[Слоты]] - механизм композиции в Vue
- [[Паттерн-компоновщик]] - теоретическая основа

## Заключение

Паттерн Компоновщик в Vue реализуется естественным образом через систему компонентов, слотов и механизмов композиции. Это делает Vue мощным инструментом для создания сложных и гибких пользовательских интерфейсов, где каждый компонент может быть как простым элементом, так и контейнером для других компонентов.

> [!tip]
> Используйте слоты для создания гибких и переиспользуемых компонентов в Vue.

> [!info]
> Composition API в Vue 3 предоставляет более гибкие возможности для композиции логики между компонентами.

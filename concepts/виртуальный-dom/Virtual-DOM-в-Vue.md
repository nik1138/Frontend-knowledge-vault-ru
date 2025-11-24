---
aliases: ["Virtual DOM в Vue", "Vue Virtual DOM", "Виртуальный DOM Vue"]
tags: [frontend, vue, virtual-dom, javascript, performance]
---

# Virtual DOM в Vue

## Введение

Virtual DOM (виртуальный DOM) в Vue.js - это концепция, аналогичная той, что используется в React, но с некоторыми особенностями, характерными для архитектуры Vue. Vue использует Virtual DOM для оптимизации обновлений пользовательского интерфейса, минимизируя количество дорогостоящих операций с реальным DOM.

## Архитектура Virtual DOM в Vue

Vue 3 представил новую реализацию Virtual DOM, которая отличается от более ранних версий. Основные компоненты:

### VNode (Virtual Node)

В Vue VNode представлен следующей структурой:

```javascript
// Упрощенная структура VNode в Vue
const VNode = {
  type: 'div',           // Тип элемента или компонента
  props: { id: 'app' },  // Свойства элемента
  children: [],          // Дочерние элементы
  key: 'unique-key',     // Ключ для идентификации
  ref: null,             // Ссылка на элемент
  shapeFlag: 0,          // Флаги для определения типа узла
  patchFlag: 0,          // Флаги для оптимизации патчинга
  dynamicProps: null,    // Динамические свойства
  dynamicChildren: null, // Динамические дочерние элементы
  appContext: null       // Контекст приложения
};
```

### Shape Flags

Vue использует shape flags для определения типа VNode:

```javascript
// Основные флаги в Vue
const ShapeFlags = {
  ELEMENT: 1,                    // HTML элемент
  FUNCTIONAL_COMPONENT: 1 << 1,  // Функциональный компонент
  STATEFUL_COMPONENT: 1 << 2,    // Компонент с состоянием
  TEXT_CHILDREN: 1 << 3,         // Текстовые дочерние элементы
  ARRAY_CHILDREN: 1 << 4,        // Массив дочерних элементов
  SLOTS_CHILDREN: 1 << 5,        // Дочерние элементы в виде слотов
  TELEPORT: 1 << 6,              // Teleport компонент
  SUSPENSE: 1 << 7,              // Suspense компонент
  COMPONENT_SHOULD_KEEP_ALIVE: 1 << 8,  // Компонент с keep-alive
  COMPONENT_KEPT_ALIVE: 1 << 9          // Компонент, оставшийся в памяти
};
```

### Patch Flags

Vue 3 ввел patch flags для оптимизации обновлений:

```javascript
// Основные флаги патчинга
const PatchFlags = {
  TEXT: 1,                    // Текстовое содержимое
  CLASS: 2,                   // Классы элемента
  STYLE: 4,                   // Стили элемента
  PROPS: 8,                   // Свойства элемента
  FULL_PROPS: 16,             // Все свойства
  HYDRATE_EVENTS: 32,         // События для гидратации
  STABLE_FRAGMENT: 64,        // Стабильный фрагмент
  KEYED_FRAGMENT: 128,        // Фрагмент с ключами
  UNKEYED_FRAGMENT: 256,      // Фрагмент без ключей
  NEED_PATCH: 512,            // Требуется патчинг
  DYNAMIC_SLOTS: 1024,        // Динамические слоты
  DEV_ROOT_FRAGMENT: 2048,    // Фрагмент корневого уровня (для разработки)
  HOISTED: -1,                // Поднятый статический элемент
  BAIL: -2                    // Прекращение оптимизации
};
```

## Рендеринг в Vue

### Создание VNode

```javascript
import { h } from 'vue';

// Создание VNode с помощью h-функции
const vnode = h('div', { id: 'app', class: 'container' }, [
  h('h1', { class: 'title' }, 'Заголовок'),
  h('p', 'Текст параграфа'),
  h('button', { onClick: () => console.log('Кнопка нажата') }, 'Нажми меня')
]);
```

### Рендер-функции

```javascript
import { h, ref } from 'vue';

// Рендер-функция компонента
function MyComponent() {
  const count = ref(0);
  
  return {
    render() {
      return h('div', [
        h('p', `Счетчик: ${count.value}`),
        h('button', { 
          onClick: () => count.value++ 
        }, 'Увеличить')
      ]);
    }
  };
}
```

## Сравнение с React

| Особенность | Vue | React |
|-------------|-----|-------|
| Компиляция шаблонов | Да, во время сборки | Нет, только JSX |
| Статическая оптимизация | Да, через patch flags | Ограниченная |
| Поддержка слотов | Да, через scoped slots | Да, через render props |
| Система реактивности | На основе прокси/геттеров | На основе setState/useState |

## Алгоритм согласования (Reconciliation)

### Основной алгоритм

```javascript
// Упрощенная реализация алгоритма согласования в Vue
function patch(oldVNode, newVNode, container, anchor = null) {
  if (oldVNode === newVNode) {
    return;
  }
  
  // Проверка, являются ли узлы одинаковыми
  if (oldVNode && !isSameVNodeType(oldVNode, newVNode)) {
    unmount(oldVNode, container);
    oldVNode = null;
  }
  
  const { type, shapeFlag } = newVNode;
  
  switch (type) {
    case Text:
      processText(oldVNode, newVNode, container, anchor);
      break;
    case Comment:
      processCommentNode(oldVNode, newVNode, container, anchor);
      break;
    case Fragment:
      processFragment(oldVNode, newVNode, container, anchor);
      break;
    default:
      if (shapeFlag & ShapeFlags.ELEMENT) {
        processElement(oldVNode, newVNode, container, anchor);
      } else if (shapeFlag & ShapeFlags.COMPONENT) {
        processComponent(oldVNode, newVNode, container, anchor);
      }
  }
}

function isSameVNodeType(n1, n2) {
  return n1.type === n2.type && n1.key === n2.key;
}
```

### Обработка элементов

```javascript
function processElement(n1, n2, container, anchor) {
  if (n1 == null) {
    // Монтирование нового элемента
    mountElement(n2, container, anchor);
  } else {
    // Обновление существующего элемента
    patchElement(n1, n2);
  }
}

function mountElement(vnode, container, anchor) {
  let el;
  const { type, props, shapeFlag } = vnode;
  
  el = vnode.el = document.createElement(type);
  
  if (shapeFlag & ShapeFlags.TEXT_CHILDREN) {
    el.textContent = vnode.children;
  } else if (shapeFlag & ShapeFlags.ARRAY_CHILDREN) {
    mountChildren(vnode.children, el, anchor);
  }
  
  if (props) {
    for (const key in props) {
      if (!isReservedProp(key)) {
        hostPatchProp(el, key, null, props[key]);
      }
    }
  }
  
  container.insertBefore(el, anchor);
}

function patchElement(n1, n2) {
  const el = (n2.el = n1.el);
  const oldProps = (n1 && n1.props) || {};
  const newProps = n2.props || {};
  
  const { patchFlag } = n2;
  
  // Оптимизация через patch flags
  if (patchFlag > 0) {
    if (patchFlag & PatchFlags.FULL_PROPS) {
      // Обновление всех свойств
      patchProps(el, oldProps, newProps);
    } else {
      // Частичное обновление
      if (patchFlag & PatchFlags.CLASS) {
        if (newProps.class !== oldProps.class) {
          el.className = newProps.class || '';
        }
      }
      if (patchFlag & PatchFlags.STYLE) {
        patchStyle(el, oldProps.style, newProps.style);
      }
      if (patchFlag & PatchFlags.PROPS) {
        const propsToUpdate = n2.dynamicProps;
        for (let i = 0; i < propsToUpdate.length; i++) {
          const key = propsToUpdate[i];
          const prev = oldProps[key];
          const next = newProps[key];
          if (next !== prev) {
            hostPatchProp(el, key, prev, next);
          }
        }
      }
    }
  } else if (oldProps !== newProps) {
    // Обновление всех свойств
    patchProps(el, oldProps, newProps);
  }
  
  // Обновление дочерних элементов
  patchChildren(n1, n2, el);
}
```

## Статическая оптимизация

Vue 3 компилирует шаблоны с оптимизациями:

```html
<!-- Шаблон до компиляции -->
<template>
  <div>
    <h1>Заголовок</h1>
    <p>{{ dynamicText }}</p>
    <ul>
      <li v-for="item in items" :key="item.id">{{ item.name }}</li>
    </ul>
  </div>
</template>
```

```javascript
// Результат компиляции (упрощенно)
import { createVNode, toDisplayString, Fragment, openBlock, createBlock } from 'vue';

const _hoisted_1 = /*#__PURE__*/createVNode("h1", null, "Заголовок", -1 /* HOISTED */);

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (openBlock(), createBlock("div", null, [
    _hoisted_1,
    createVNode("p", null, toDisplayString(_ctx.dynamicText), 1 /* TEXT */),
    (openBlock(), createBlock(Fragment, null, 
      _ctx.items.map((item) => {
        return createVNode("li", { key: item.id }, toDisplayString(item.name), 1 /* TEXT */)
      }), 
      128 /* KEYED_FRAGMENT */
    ))
  ]))
}
```

## Слоты и компоненты

Vue имеет продвинутую систему слотов:

```javascript
// Компонент с использованием слотов
function MyComponent(props, { slots }) {
  return h('div', [
    h('header', slots.header ? slots.header() : 'Заголовок по умолчанию'),
    h('main', slots.default ? slots.default() : 'Контент по умолчанию'),
    h('footer', slots.footer ? slots.footer() : 'Футер по умолчанию')
  ]);
}

// Использование компонента со слотами
function ParentComponent() {
  return h(MyComponent, null, {
    header: () => h('h1', 'Специальный заголовок'),
    default: () => h('p', 'Специальный контент'),
    footer: () => h('button', 'Специальная кнопка')
  });
}
```

## Практические примеры

### Пример 1: Список с фильтрацией

```vue
<template>
  <div>
    <input v-model="searchTerm" placeholder="Поиск..." />
    <ul>
      <li 
        v-for="item in filteredItems" 
        :key="item.id"
        :class="{ active: item.active }"
      >
        {{ item.name }}
      </li>
    </ul>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    const searchTerm = ref('');
    const items = ref([
      { id: 1, name: 'Элемент 1', active: true },
      { id: 2, name: 'Элемент 2', active: false },
      { id: 3, name: 'Элемент 3', active: true }
    ]);
    
    const filteredItems = computed(() => {
      return items.value.filter(item => 
        item.name.toLowerCase().includes(searchTerm.value.toLowerCase())
      );
    });
    
    return {
      searchTerm,
      filteredItems
    };
  }
};
</script>
```

### Пример 2: Динамический компонент

```vue
<template>
  <div>
    <button @click="currentView = 'Home'">Главная</button>
    <button @click="currentView = 'About'">О нас</button>
    
    <component :is="currentView" />
  </div>
</template>

<script>
import { ref } from 'vue';
import Home from './Home.vue';
import About from './About.vue';

export default {
  components: { Home, About },
  setup() {
    const currentView = ref('Home');
    
    return {
      currentView
    };
  }
};
</script>
```

## Оптимизации производительности

### v-memo (Vue 3.2+)

```vue
<template>
  <div v-for="item in list" :key="item.id">
    <div v-memo="[item.id, item.selected]">
      <ExpensiveComponent :item="item" />
    </div>
  </div>
</template>
```

### Suspense

```vue
<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <div>Загрузка...</div>
    </template>
  </Suspense>
</template>
```

## Заключение

Virtual DOM в Vue предоставляет мощный механизм для оптимизации обновлений UI с рядом уникальных особенностей, таких как статическая оптимизация через компиляцию шаблонов, система слотов и продвинутые флаги патчинга. Эти особенности позволяют Vue эффективно управлять обновлениями DOM и создавать высокопроизводительные приложения.

См. также:
- [[Реализация-Virtual-DOM]]
- [[Diffing-алгоритмы]]
- [[Virtual-DOM-в-React]]
- [[Виртуальный-DOM]]
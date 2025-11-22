---
aliases: [Vue Slots, Слоты в Vue.js]
tags: [vue, frontend, components, slots]
---

# Слоты в Vue.js

Слоты в Vue.js - это мощный механизм для композиции компонентов, позволяющий передавать контент из родительского компонента в дочерний. Это особенно полезно при создании переиспользуемых компонентов, которые должны содержать переменный контент.

## Основы слотов

В простейшем случае слот позволяет вставить произвольный контент внутрь компонента:

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="card">
    <h3>Заголовок карточки</h3>
    <slot></slot>
  </div>
</template>

<style>
.card {
  border: 1px solid #ccc;
  padding: 16px;
  margin: 8px;
  border-radius: 4px;
}
</style>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent>
    <p>Это содержимое будет отображено внутри слота</p>
  </ChildComponent>
</template>
```

В этом примере текст "Это содержимое будет отображено внутри слота" будет отображаться внутри `ChildComponent` в месте, где определен `<slot></slot>`.

## Именованные слоты

Vue поддерживает именованные слоты, которые позволяют определять несколько областей для вставки контента:

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="layout">
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent>
    <template #header>
      <h1>Заголовок страницы</h1>
    </template>
    
    <p>Основной контент страницы</p>
    
    <template #footer>
      <p>&copy; 2023 Все права защищены</p>
    </template>
  </ChildComponent>
</template>
```

## Скоуп-слоты (Scoped Slots)

Скоуп-слоты позволяют передавать данные из дочернего компонента в родительский, где они могут быть использованы в шаблоне:

```vue
<!-- ChildComponent.vue -->
<template>
  <div>
    <slot :user="userData" :isLoggedIn="checkAuth()"></slot>
  </div>
</template>

<script>
export default {
  data() {
    return {
      userData: {
        name: 'Иван',
        email: 'ivan@example.com'
      }
    }
  },
  methods: {
    checkAuth() {
      return true; // условная проверка
    }
  }
}
</script>
```

```vue
<!-- ParentComponent.vue -->
<template>
  <ChildComponent v-slot="slotProps">
    <div v-if="slotProps.isLoggedIn">
      <h2>Привет, {{ slotProps.user.name }}!</h2>
      <p>Ваш email: {{ slotProps.user.email }}</p>
    </div>
    <div v-else>
      <p>Пожалуйста, войдите в систему</p>
    </div>
  </ChildComponent>
</template>
```

## Сокращения для слотов

Vue предоставляет сокращенные формы записи для слотов:

- `v-slot:` можно сократить до `#`
- `<template v-slot:name>` можно сократить до `#name`
- `<template v-slot:name="slotProps">` можно сократить до `#name="slotProps"`

```vue
<!-- Сокращенный синтаксис -->
<ChildComponent #default="slotProps">
  <p>Содержимое слота с доступом к данным: {{ slotProps.data }}</p>
</ChildComponent>
```

## Пример использования в реальной ситуации

Представим, что мы создаем компонент списка пользователей, где каждый элемент может иметь разный вид:

```vue
<!-- UserList.vue -->
<template>
  <ul class="user-list">
    <li v-for="user in users" :key="user.id" class="user-item">
      <slot :user="user" :index="index">
        <!-- Резервный контент по умолчанию -->
        <span>{{ user.name }}</span>
      </slot>
    </li>
  </ul>
</template>

<script>
export default {
  name: 'UserList',
  props: {
    users: {
      type: Array,
      required: true
    }
  }
}
</script>
```

```vue
<!-- Использование компонента -->
<template>
  <UserList :users="userList">
    <template #default="{ user, index }">
      <div class="user-card">
        <img :src="user.avatar" :alt="user.name" class="avatar">
        <div class="user-info">
          <h3>{{ user.name }}</h3>
          <p>{{ user.email }}</p>
          <button @click="selectUser(user)">Выбрать</button>
        </div>
      </div>
    </template>
  </UserList>
</template>

<script>
import UserList from './UserList.vue'

export default {
  components: {
    UserList
  },
  data() {
    return {
      userList: [
        { id: 1, name: 'Иван Иванов', email: 'ivan@example.com', avatar: '/avatars/ivan.jpg' },
        { id: 2, name: 'Мария Смирнова', email: 'maria@example.com', avatar: '/avatars/maria.jpg' }
      ]
    }
  },
  methods: {
    selectUser(user) {
      console.log('Выбран пользователь:', user.name);
    }
  }
}
</script>
```

## Лучшие практики

1. **Используйте резервный контент** для слотов, чтобы компоненты были более устойчивыми к отсутствию переданного контента.
2. **Документируйте скоуп-слоты** - четко указывайте, какие данные передаются в слот.
3. **Именуйте слоты понятно** - используйте осмысленные имена, чтобы облегчить понимание компонента.
4. **Избегайте чрезмерного вложения** слотов, чтобы не усложнять структуру компонента.

## Связанные темы

- [[Именованные-слоты]]
- [[Скоуп-слоты]]
- [[Композиция-компонентов-в-Vue]]
- [[Vue-компоненты]]

## Теги

#vue #frontend #components #slots #javascript #web-development
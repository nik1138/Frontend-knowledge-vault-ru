---
aliases: [Svelte Slots, Слоты в Svelte]
tags: [svelte, frontend, components, slots]
---

# Слоты в Svelte

Слоты в Svelte - это механизм для передачи содержимого из родительского компонента в дочерний, аналогичный слотам в Vue.js. Svelte предоставляет простой и интуитивно понятный способ работы с контентом, передаваемым в компонент.

## Основы слотов

В Svelte слоты реализованы через тег `<slot>`. Любой контент, переданный в компонент из родительского компонента, будет отображаться в месте, где определен тег `<slot>`:

```svelte
<!-- ChildComponent.svelte -->
<div class="card">
  <h3>Заголовок карточки</h3>
  <slot></slot>
</div>

<style>
.card {
  border: 1px solid #ccc;
  padding: 16px;
  margin: 8px;
  border-radius: 4px;
}
</style>
```

```svelte
<!-- ParentComponent.svelte -->
<script>
  import ChildComponent from './ChildComponent.svelte';
</script>

<ChildComponent>
  <p>Это содержимое будет отображено внутри слота</p>
</ChildComponent>
```

В этом примере текст "Это содержимое будет отображено внутри слота" будет отображаться внутри `ChildComponent` в месте, где определен `<slot></slot>`.

## Именованные слоты

Svelte также поддерживает именованные слоты, которые позволяют определять несколько областей для вставки контента:

```svelte
<!-- ChildComponent.svelte -->
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
```

```svelte
<!-- ParentComponent.svelte -->
<script>
  import ChildComponent from './ChildComponent.svelte';
</script>

<ChildComponent>
  <h1 slot="header">Заголовок страницы</h1>
  <p>Основной контент страницы</p>
  <p slot="footer">&copy; 2023 Все права защищены</p>
</ChildComponent>
```

В отличие от Vue, в Svelte именованные слоты указываются с помощью атрибута `slot` непосредственно на элементе, а не через специальный тег `<template>`.

## Слоты с пропсами (Slot Props)

В Svelte можно передавать данные из дочернего компонента в родительский через слоты, используя функцию `let`:

```svelte
<!-- ChildComponent.svelte -->
<script>
  export let userData = {
    name: 'Иван',
    email: 'ivan@example.com'
  };

  function checkAuth() {
    return true; // условная проверка
  }
</script>

<div>
  <slot {userData} {isLoggedIn=checkAuth()}></slot>
</div>
```

```svelte
<!-- ParentComponent.svelte -->
<script>
  import ChildComponent from './ChildComponent.svelte';
</script>

<ChildComponent let:userData let:isLoggedIn>
  {#if isLoggedIn}
    <h2>Привет, {userData.name}!</h2>
    <p>Ваш email: {userData.email}</p>
  {/if}
</ChildComponent>
```

## Резервный контент

Можно определить резервный контент, который будет отображаться, если в слот не передано никакого содержимого:

```svelte
<!-- ChildComponent.svelte -->
<div class="card">
  <h3>Заголовок карточки</h3>
  <slot>
    <p>Содержимое по умолчанию, если ничего не передано</p>
  </slot>
</div>
```

## Пример использования в реальной ситуации

Рассмотрим пример компонента списка пользователей с возможностью настройки отображения каждого элемента:

```svelte
<!-- UserList.svelte -->
<script>
  export let users = [];
</script>

<ul class="user-list">
  {#each users as user, index}
    <li class="user-item">
      <slot {user} {index}>
        <!-- Резервный контент по умолчанию -->
        <span>{user.name}</span>
      </slot>
    </li>
  {/each}
</ul>

<style>
.user-list {
  list-style: none;
  padding: 0;
}

.user-item {
  padding: 8px;
  border-bottom: 1px solid #eee;
}
</style>
```

```svelte
<!-- Использование компонента -->
<script>
  import UserList from './UserList.svelte';

  let userList = [
    { id: 1, name: 'Иван Иванов', email: 'ivan@example.com', avatar: '/avatars/ivan.jpg' },
    { id: 2, name: 'Мария Смирнова', email: 'maria@example.com', avatar: '/avatars/maria.jpg' }
  ];

  function selectUser(user) {
    console.log('Выбран пользователь:', user.name);
  }
</script>

<UserList {users=userList}>
  <div slot="default" let:user let:index>
    <div class="user-card">
      <img src={user.avatar} alt={user.name} class="avatar">
      <div class="user-info">
        <h3>{user.name}</h3>
        <p>{user.email}</p>
        <button on:click={() => selectUser(user)}>Выбрать</button>
      </div>
    </div>
  </div>
</UserList>
```

## Лучшие практики

1. **Используйте резервный контент** для слотов, чтобы компоненты были более устойчивыми к отсутствию переданного контента.
2. **Документируйте слоты с пропсами** - четко указывайте, какие данные передаются в слот.
3. **Именуйте слоты понятно** - используйте осмысленные имена, чтобы облегчить понимание компонента.
4. **Используйте именованные слоты** для сложных компонентов с несколькими областями для контента.
5. **Избегайте чрезмерного вложения** слотов, чтобы не усложнять структуру компонента.

## Связанные темы

- [[Именованные-слоты]]
- [[Скоуп-слоты]]
- [[Компоненты-в-Svelte]]
- [[Svelte-реактивность]]

## Теги

#svelte #frontend #components #slots #javascript #web-development
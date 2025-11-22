---
aliases: ["Полиморфизм в Svelte", "Svelte Полиморфизм"]
tags: ["#svelte", "#polymorphism", "#frontend", "#components"]
---

# Полиморфизм-в-Svelte

## Введение

Полиморфизм в Svelte проявляется через гибкие компоненты, которые могут адаптироваться к различным сценариям использования, принимая разные типы данных или отображая разное содержимое при сохранении одинакового интерфейса. Svelte предоставляет уникальные возможности для реализации полиморфного поведения через слоты, динамические компоненты, композицию и реактивные концепции.

## Типы полиморфизма в Svelte

### 1. Слоты (Slots) как основа полиморфизма

Слоты в Svelte являются основным механизмом для создания полиморфных компонентов, позволяя передавать разное содержимое в один и тот же компонент.

```svelte
<!-- BaseModal.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  export let isOpen = false;
  export let title = '';
  
  const dispatch = createEventDispatcher();
  
  function closeModal() {
    dispatch('close');
  }
  
  function handleOverlayClick(event) {
    if (event.target === event.currentTarget) {
      closeModal();
    }
  }
</script>

{#if isOpen}
  <div class="modal-overlay" on:click={handleOverlayClick}>
    <div class="modal-content">
      <div class="modal-header">
        <slot name="header">
          <h2>{title}</h2>
        </slot>
        <button on:click={closeModal} class="close-button">×</button>
      </div>
      <div class="modal-body">
        <slot name="body" />
      </div>
      <div class="modal-footer">
        <slot name="footer">
          <button on:click={closeModal}>Закрыть</button>
        </slot>
      </div>
    </div>
  </div>
{/if}

<style>
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }
  
  .modal-content {
    background: white;
    border-radius: 8px;
    padding: 0;
    max-width: 500px;
    width: 90%;
    max-height: 90vh;
    overflow-y: auto;
  }
  
  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    border-bottom: 1px solid #eee;
  }
  
  .modal-body {
    padding: 1rem;
  }
  
  .modal-footer {
    padding: 1rem;
    border-top: 1px solid #eee;
    display: flex;
    justify-content: flex-end;
    gap: 0.5rem;
  }
  
  .close-button {
    background: none;
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
    padding: 0;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
</style>
```

```svelte
<!-- Использование с разными содержимым -->
<script>
  import BaseModal from './BaseModal.svelte';
  import UserProfile from './UserProfile.svelte';
  import ProductDetails from './ProductDetails.svelte';
  
  let showUserModal = false;
  let showProductModal = false;
  
  let selectedUser = null;
  let selectedProduct = null;
  
  function updateUser() {
    // Логика обновления пользователя
  }
  
  function deleteProduct() {
    // Логика удаления продукта
  }
</script>

<div>
  <button on:click={() => showUserModal = true}>Показать пользователя</button>
  <button on:click={() => showProductModal = true}>Показать продукт</button>

  <!-- Модальное окно для пользователя -->
  <BaseModal 
    {isOpen: showUserModal} 
    title="Детали пользователя"
    on:close={() => showUserModal = false}
  >
    <svelte:fragment slot="body">
      {#if selectedUser}
        <UserProfile {user: selectedUser} />
      {/if}
    </svelte:fragment>
    <svelte:fragment slot="footer">
      <button on:click={updateUser}>Сохранить</button>
      <button on:click={() => showUserModal = false}>Отмена</button>
    </svelte:fragment>
  </BaseModal>

  <!-- Модальное окно для продукта -->
  <BaseModal 
    {isOpen: showProductModal} 
    title="Детали продукта"
    on:close={() => showProductModal = false}
  >
    <svelte:fragment slot="body">
      {#if selectedProduct}
        <ProductDetails {product: selectedProduct} />
      {/if}
    </svelte:fragment>
    <svelte:fragment slot="footer">
      <button on:click={deleteProduct}>Удалить</button>
      <button on:click={() => showProductModal = false}>Закрыть</button>
    </svelte:fragment>
  </BaseModal>
</div>
```

### 2. Слоты с доступом к данным (Contextual Slots)

Svelte позволяет передавать данные из компонента в слот, обеспечивая полиморфное поведение с доступом к внутреннему состоянию компонента.

```svelte
<!-- ListRenderer.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  export let items = [];
  export let selectedId = null;
  
  const dispatch = createEventDispatcher();
  
  function isSelected(item) {
    return item.id === selectedId;
  }
  
  function handleItemClick(item, index) {
    dispatch('itemClick', { item, index });
  }
</script>

<div class="list-container">
  {#each items as item, index}
    <div 
      class="list-item" 
      class:selected={isSelected(item)}
      on:click={() => handleItemClick(item, index)}
    >
      <slot {item} {index} {isSelected: isSelected(item)} />
    </div>
  {/each}
</div>

<style>
  .list-container {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .list-item {
    padding: 0.75rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s;
  }
  
  .list-item:hover {
    background-color: #f5f5f5;
  }
  
  .list-item.selected {
    background-color: #e3f2fd;
    border-color: #2196f3;
  }
</style>
```

```svelte
<script>
  import ListRenderer from './ListRenderer.svelte';
  
  let users = [
    { id: 1, name: 'Иван', email: 'ivan@example.com', role: 'admin' },
    { id: 2, name: 'Мария', email: 'maria@example.com', role: 'user' }
  ];
  
  let tasks = [
    { id: 1, title: 'Задача 1', completed: false, dueDate: '2023-12-01' },
    { id: 2, title: 'Задача 2', completed: true, dueDate: '2023-11-30' }
  ];
  
  let selectedUserId = null;
  let selectedTaskId = null;
  
  function handleUserClick(event) {
    const { item } = event.detail;
    selectedUserId = item.id;
  }
  
  function handleTaskClick(event) {
    const { item } = event.detail;
    selectedTaskId = item.id;
  }
</script>

<div>
  <!-- Список пользователей -->
  <h3>Пользователи</h3>
  <ListRenderer 
    {items: users} 
    {selectedId: selectedUserId}
    on:itemClick={handleUserClick}
  >
    {#snippet default({ item, index, isSelected })}
      <div class="user-item">
        <img src={item.avatar || '/default-avatar.png'} alt={item.name} />
        <div>
          <span class="name">{item.name}</span>
          <span class="email">{item.email}</span>
          <span class="role">{item.role}</span>
        </div>
      </div>
    {/snippet}
  </ListRenderer>

  <!-- Список задач -->
  <h3>Задачи</h3>
  <ListRenderer 
    {items: tasks} 
    {selectedId: selectedTaskId}
    on:itemClick={handleTaskClick}
  >
    {#snippet default({ item, index, isSelected })}
      <div class="task-item">
        <input 
          type="checkbox" 
          bind:checked={item.completed} 
          on:change={() => console.log('Task status changed')}
        />
        <span class="title">{item.title}</span>
        <span class="due-date">{item.dueDate}</span>
      </div>
    {/snippet}
  </ListRenderer>
</div>
```

### 3. Динамические компоненты

Svelte позволяет использовать динамические компоненты через специальный элемент `svelte:component`, что является формой полиморфизма на уровне компонентов.

```svelte
<!-- DynamicRenderer.svelte -->
<script>
  import UserProfile from './UserProfile.svelte';
  import ProductCard from './ProductCard.svelte';
  import ArticlePreview from './ArticlePreview.svelte';
  
  export let currentView = 'user';
  export let data = null;
  
  // Карта компонентов
  const componentMap = {
    user: UserProfile,
    product: ProductCard,
    article: ArticlePreview
  };
  
  // Получаем текущий компонент
  $: CurrentComponent = componentMap[currentView] || UserProfile;
  
  // Получаем пропсы для компонента
  $: componentProps = {
    user: currentView === 'user' ? data : undefined,
    product: currentView === 'product' ? data : undefined,
    article: currentView === 'article' ? data : undefined
  };
  
  function handleAction(action) {
    console.log('Действие:', action);
  }
</script>

<svelte:component 
  this={CurrentComponent} 
  {...componentProps}
  on:action={handleAction}
/>
```

```svelte
<!-- Использование динамического рендерера -->
<script>
  import DynamicRenderer from './DynamicRenderer.svelte';
  
  let currentView = 'user';
  let userData = { name: 'Иван', email: 'ivan@example.com', avatar: '/avatar.jpg' };
  let productData = { name: 'Смартфон', price: 29999, image: '/phone.jpg' };
  let articleData = { title: 'Новости', content: 'Содержимое статьи...', author: 'Автор' };
  
  function switchView(view) {
    currentView = view;
  }
</script>

<div>
  <nav>
    <button class:selected={currentView === 'user'} on:click={() => switchView('user')}>
      Пользователь
    </button>
    <button class:selected={currentView === 'product'} on:click={() => switchView('product')}>
      Продукт
    </button>
    <button class:selected={currentView === 'article'} on:click={() => switchView('article')}>
      Статья
    </button>
  </nav>
  
  <DynamicRenderer 
    {currentView}
    data={currentView === 'user' ? userData : 
          currentView === 'product' ? productData : 
          articleData}
  />
</div>

<style>
  nav {
    margin-bottom: 1rem;
  }
  
  button {
    margin-right: 0.5rem;
    padding: 0.5rem 1rem;
    border: 1px solid #ccc;
    background: white;
    cursor: pointer;
  }
  
  button.selected {
    background: #007bff;
    color: white;
  }
</style>
```

### 4. Полиморфизм через композицию

Svelte поощряет композицию над наследованием, что естественным образом приводит к полиморфному поведению:

```svelte
<!-- BaseCard.svelte -->
<script>
  export let title = '';
  export let type = 'default'; // default, primary, success, warning, danger
  export let shadow = true;
</script>

<div class="card" class:shadow={shadow} class:card--{type}>
  {#if title}
    <div class="card-header">
      <slot name="header">
        <h3>{title}</h3>
      </slot>
    </div>
  {/if}
  
  <div class="card-body">
    <slot />
  </div>
  
  {#if $$slots.footer}
    <div class="card-footer">
      <slot name="footer" />
    </div>
  {/if}
</div>

<style>
  .card {
    border-radius: 8px;
    overflow: hidden;
    background: white;
    border: 1px solid #e0e0e0;
  }
  
  .card.shadow {
    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  }
  
  .card--primary {
    border-left: 4px solid #007bff;
  }
  
  .card--success {
    border-left: 4px solid #28a745;
  }
  
  .card--warning {
    border-left: 4px solid #ffc107;
  }
  
  .card--danger {
    border-left: 4px solid #dc3545;
  }
  
  .card-header {
    padding: 1rem;
    border-bottom: 1px solid #eee;
  }
  
  .card-body {
    padding: 1rem;
  }
  
  .card-footer {
    padding: 1rem;
    border-top: 1px solid #eee;
    background-color: #f8f9fa;
  }
</style>
```

```svelte
<!-- UserProfileCard.svelte -->
<script>
  import BaseCard from './BaseCard.svelte';
  import { onMount } from 'svelte';
  
  export let user = null;
  
  // Дополнительная логика для профиля пользователя
  $: displayName = user ? `${user.firstName} ${user.lastName}` : '';
  
  function editProfile() {
    console.log('Редактировать профиль');
  }
  
  function deleteProfile() {
    console.log('Удалить профиль');
  }
</script>

<BaseCard title="Профиль пользователя" type="primary">
  <svelte:fragment slot="header">
    <div class="user-header">
      <img src={user?.avatar} alt={displayName} class="avatar" />
      <h3>{displayName}</h3>
    </div>
  </svelte:fragment>
  
  <div class="user-info">
    <p><strong>Email:</strong> {user?.email}</p>
    <p><strong>Роль:</strong> {user?.role}</p>
    <p><strong>Дата регистрации:</strong> {user?.createdAt}</p>
  </div>
  
  <svelte:fragment slot="footer">
    <button on:click={editProfile}>Редактировать</button>
    <button on:click={deleteProfile} class="danger">Удалить</button>
  </svelte:fragment>
</BaseCard>

<style>
  .user-header {
    display: flex;
    align-items: center;
    gap: 1rem;
  }
  
  .avatar {
    width: 50px;
    height: 50px;
    border-radius: 50%;
  }
  
  .user-info {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .danger {
    background-color: #dc3545;
    color: white;
  }
</style>
```

### 5. Полиморфизм через реактивные концепции

Svelte позволяет создавать полиморфные компоненты через реактивные вычисления:

```svelte
<!-- SmartList.svelte -->
<script>
  import { onMount } from 'svelte';
  
  export let items = [];
  export let itemType = 'user'; // user, product, task
  export let sortBy = '';
  export let reverse = false;
  
  // Определяем доступные поля для сортировки в зависимости от типа
  $: sortableFields = itemType === 'user' ? ['name', 'email', 'role', 'createdAt'] :
                     itemType === 'product' ? ['name', 'price', 'category', 'createdAt'] :
                     itemType === 'task' ? ['title', 'status', 'dueDate', 'priority'] :
                     [];
  
  // Реактивно сортируем элементы
  $: sortedItems = [...items].sort((a, b) => {
    if (!sortBy) return 0;
    
    const aValue = getNestedValue(a, sortBy);
    const bValue = getNestedValue(b, sortBy);
    
    if (aValue < bValue) return reverse ? 1 : -1;
    if (aValue > bValue) return reverse ? -1 : 1;
    return 0;
  });
  
  function getNestedValue(obj, path) {
    return path.split('.').reduce((current, key) => current?.[key], obj);
  }
  
  function setSort(field) {
    if (sortBy === field) {
      reverse = !reverse;
    } else {
      sortBy = field;
      reverse = false;
    }
  }
  
  // Функция для отображения значения в зависимости от типа
  $: renderValue = (item, field) => {
    if (field === 'price' && itemType === 'product') {
      return `${item[field]} руб.`;
    }
    if (field === 'dueDate' && itemType === 'task') {
      return new Date(item[field]).toLocaleDateString();
    }
    return item[field];
  };
</script>

<div class="smart-list">
  <div class="controls">
    <select bind:value={sortBy} on:change={() => setSort(sortBy)}>
      <option value="">Сортировать по...</option>
      {#each sortableFields as field}
        <option value={field}>{field}</option>
      {/each}
    </select>
    <button on:click={() => reverse = !reverse}>
      {reverse ? 'По убыванию' : 'По возрастанию'}
    </button>
  </div>
  
  <div class="items">
    {#each sortedItems as item, index}
      <div class="item">
        <slot {item} {index} {renderValue} />
      </div>
    {/each}
  </div>
</div>

<style>
  .smart-list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  
  .controls {
    display: flex;
    gap: 1rem;
    align-items: center;
  }
  
  .items {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .item {
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
  }
</style>
```

## Практические примеры полиморфных компонентов

### 1. Универсальный форм-филд

```svelte
<!-- FormField.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  export let type = 'text';
  export let label = '';
  export let placeholder = '';
  export let required = false;
  export let error = '';
  export let helperText = '';
  export let value = '';
  export let options = [];
  export let id = `field-${Math.random().toString(36).substr(2, 9)}`;
  
  const dispatch = createEventDispatcher();
  
  function handleChange(event) {
    dispatch('change', event);
    dispatch('input', event);
  }
  
  // Определяем тип элемента
  $: elementType = type === 'textarea' ? 'textarea' : 
                  type === 'select' ? 'select' : 
                  type === 'checkbox' || type === 'radio' ? 'input' : 'input';
</script>

<div class="form-field" class:error={!!error}>
  {#if label}
    <label for={id}>{label} {#if required}<span class="required">*</span>{/if}</label>
  {/if}
  
  {#if elementType === 'select'}
    <select 
      id={id} 
      class:has-error={!!error}
      on:change={handleChange}
      bind:value
    >
      {#each options as option}
        <option value={option.value} selected={option.value === value}>
          {option.label}
        </option>
      {/each}
    </select>
  {:else if elementType === 'textarea'}
    <textarea 
      id={id} 
      placeholder={placeholder}
      class:has-error={!!error}
      on:input={handleChange}
      bind:value
    />
  {:else}
    <input 
      id={id}
      type={type}
      placeholder={placeholder}
      required={required}
      class:has-error={!!error}
      on:input={handleChange}
      bind:value
    />
  {/if}
  
  {#if error}
    <div class="error-message">{error}</div>
  {/if}
  
  {#if helperText}
    <div class="helper-text">{helperText}</div>
  {/if}
</div>

<style>
  .form-field {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
    margin-bottom: 1rem;
  }
  
  .form-field.error {
    color: #dc3545;
  }
  
  label {
    font-weight: bold;
    margin-bottom: 0.25rem;
  }
  
  .required {
    color: #dc3545;
  }
  
  input, select, textarea {
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 1rem;
  }
  
  input:focus, select:focus, textarea:focus {
    outline: none;
    border-color: #007bff;
    box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
  }
  
  input.has-error, select.has-error, textarea.has-error {
    border-color: #dc3545;
  }
  
  .error-message {
    color: #dc3545;
    font-size: 0.875rem;
  }
  
  .helper-text {
    color: #6c757d;
    font-size: 0.875rem;
  }
</style>
```

```svelte
<!-- Использование универсального FormField -->
<script>
  import FormField from './FormField.svelte';
  
  let formData = {
    name: '',
    email: '',
    message: '',
    subscribe: false,
    category: ''
  };
  
  const categories = [
    { value: 'tech', label: 'Технологии' },
    { value: 'business', label: 'Бизнес' },
    { value: 'lifestyle', label: 'Образ жизни' }
  ];
</script>

<form>
  <FormField 
    type="text" 
    label="Имя" 
    bind:value={formData.name}
    required={true}
    placeholder="Введите ваше имя"
  />
  
  <FormField 
    type="email" 
    label="Email" 
    bind:value={formData.email}
    required={true}
    placeholder="Введите ваш email"
  />
  
  <FormField 
    type="select" 
    label="Категория" 
    bind:value={formData.category}
    {options: categories}
  />
  
  <FormField 
    type="textarea" 
    label="Сообщение" 
    bind:value={formData.message}
    placeholder="Введите ваше сообщение"
  />
  
  <FormField 
    type="checkbox" 
    label="Подписаться на рассылку" 
    bind:value={formData.subscribe}
  />
</form>
```

### 2. Полиморфный компонент уведомлений

```svelte
<!-- Notification.svelte -->
<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import { fade, slide } from 'svelte/transition';
  
  export let show = false;
  export let type = 'info'; // info, success, warning, error
  export let message = '';
  export let duration = 0; // 0 = не авто-скрытие
  export let closable = true;
  
  const dispatch = createEventDispatcher();
  
  let timeoutId = null;
  
  // Автоматическое скрытие
  $: if (show && duration > 0) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      show = false;
      dispatch('close');
    }, duration);
  }
  
  function handleClose() {
    show = false;
    dispatch('close');
  }
  
  // Очищаем таймер при разрушении компонента
  onMount(() => {
    return () => {
      if (timeoutId) {
        clearTimeout(timeoutId);
      }
    };
  });
</script>

{#if show}
  <div 
    class="notification" 
    class:notification--{type}
    in:fade={{ duration: 300 }}
    out:slide={{ duration: 300 }}
  >
    <div class="notification-content">
      <span class="notification-message">{message}</span>
      {#if closable}
        <button 
          class="close-button" 
          on:click={handleClose}
          aria-label="Закрыть уведомление"
        >
          ×
        </button>
      {/if}
    </div>
  </div>
{/if}

<style>
  .notification {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 10000;
    min-width: 300px;
    max-width: 500px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
  
  .notification-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
  }
  
  .notification-message {
    flex: 1;
  }
  
  .close-button {
    background: none;
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
    padding: 0;
    width: 30px;
    height: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .notification--info {
    background-color: #d1ecf1;
    border: 1px solid #bee5eb;
    color: #0c5460;
  }
  
  .notification--success {
    background-color: #d4edda;
    border: 1px solid #c3e6cb;
    color: #155724;
  }
  
  .notification--warning {
    background-color: #fff3cd;
    border: 1px solid #ffeaa7;
    color: #856404;
  }
  
  .notification--error {
    background-color: #f8d7da;
    border: 1px solid #f5c6cb;
    color: #721c24;
  }
</style>
```

## Полиморфизм в SvelteKit

В SvelteKit полиморфизм может быть реализован через универсальные страницы и лейауты:

```svelte
<!-- src/routes/[...path]/+page.svelte -->
<script>
  import { page } from '$app/stores';
  import UniversalRenderer from '$lib/components/UniversalRenderer.svelte';
  
  // Получаем данные из лоадера
  export let data;
  
  // Определяем тип страницы на основе данных
  $: pageType = data?.type || 'default';
  $: pageData = data?.content || {};
</script>

<UniversalRenderer {pageType} {pageData} />
```

```javascript
// src/routes/[...path]/+page.js
/** @type {import('./$types').PageLoad} */
export async function load({ params, fetch }) {
  const path = params.path;
  
  // Запрашиваем данные для конкретного типа страницы
  const response = await fetch(`/api/page/${path}`);
  const pageData = await response.json();
  
  return {
    type: pageData.type,
    content: pageData.content
  };
}
```

## Преимущества полиморфизма в Svelte

1. **Повторное использование компонентов** - один компонент может использоваться в разных контекстах
2. **Гибкость** - возможность адаптировать компоненты под разные сценарии использования
3. **Снижение дублирования кода** - уменьшение необходимости создания множества похожих компонентов
4. **Улучшенная тестируемость** - возможность тестировать компоненты с разными типами данных
5. **Лучшая поддерживаемость** - изменения в базовом компоненте автоматически применяются ко всем экземплярам
6. **Производительность** - Svelte компилирует компоненты в эффективный JavaScript без runtime overhead

## Заключение

Полиморфизм в Svelte позволяет создавать гибкие и переиспользуемые компоненты, которые могут адаптироваться к различным сценариям использования. Через слоты, динамические компоненты, композицию и реактивные концепции Svelte предоставляет мощные инструменты для реализации полиморфного поведения.

Это делает код более чистым, понятным и поддерживаемым, позволяя создавать мощные и гибкие UI-библиотеки с минимальным runtime overhead благодаря компиляции Svelte.

## См. также

- [[Компонентный подход]]
- [[Слоты]]
- [[Composition API]]
- [[Дизайн-системы]]
- [[Композиция и агрегация]]
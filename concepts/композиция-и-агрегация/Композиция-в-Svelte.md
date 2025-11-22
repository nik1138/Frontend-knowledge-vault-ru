---
aliases: ["Композиция в Svelte", "Svelte композиция компонентов", "Композиция в Svelte приложениях"]
tags: ["#frontend", "#svelte", "#composition", "#components", "#architecture", "#javascript"]
---

# Композиция в Svelte

## Введение

**Композиция в Svelte** — это подход к построению пользовательских интерфейсов, при котором сложные компоненты создаются путем объединения более простых. Svelte предоставляет мощные и интуитивно понятные механизмы композиции, включая слоты, события компонентов и импорт других компонентов. Архитектура Svelte делает композицию естественной частью разработки компонентов.

## Принципы композиции в Svelte

### 1. Простота и прозрачность
Svelte делает композицию максимально простой и понятной. Компоненты легко комбинируются друг с другом без сложных паттернов.

### 2. Слоты для композиции
Слоты в Svelte работают аналогично слотам в Vue, позволяя передавать содержимое из родительского компонента в дочерний.

### 3. Импорт и использование компонентов
Компоненты в Svelte импортируются как обычные модули JavaScript, что делает композицию интуитивно понятной.

## Практические примеры композиции в Svelte

### Пример 1: Простая композиция через слоты

```svelte
<!-- Button.svelte -->
<script>
  export let variant = 'primary';
  export let disabled = false;
  export let type = 'button';

  function handleClick(event) {
    if (!disabled) {
      dispatch('click', event);
    }
  }

  const dispatch = createEventDispatcher();
  import { createEventDispatcher } from 'svelte';
</script>

<button 
  class="btn btn-{variant} {disabled ? 'btn-disabled' : ''}"
  disabled={disabled}
  type={type}
  on:click={handleClick}
>
  <slot />
</button>

<style>
  .btn {
    padding: 8px 16px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
  }

  .btn-primary { 
    background-color: #007bff; 
    color: white; 
  }

  .btn-secondary { 
    background-color: #6c757d; 
    color: white; 
  }

  .btn-disabled { 
    opacity: 0.6; 
    cursor: not-allowed; 
  }
</style>
```

```svelte
<!-- ButtonGroup.svelte -->
<script>
  import Button from './Button.svelte';
  
  export let actions = [];
</script>

<div class="btn-group">
  {#each actions as action, index}
    <Button
      variant={action.variant}
      disabled={action.disabled}
      on:click={action.handler}
    >
      {action.label}
    </Button>
  {/each}
</div>

<style>
  .btn-group {
    display: flex;
    gap: 8px;
  }
</style>
```

### Пример 2: Использование именованных слотов

```svelte
<!-- Card.svelte -->
<script>
  export let title = '';
  export let showHeader = true;
  export let showFooter = true;
</script>

<div class="card">
  {#if showHeader && (title || $$slots.header)}
    <div class="card-header">
      {#if title}<h3>{title}</h3>{/if}
      <slot name="header" />
    </div>
  {/if}
  
  <div class="card-body">
    <slot />
  </div>
  
  {#if showFooter && $$slots.footer}
    <div class="card-footer">
      <slot name="footer" />
    </div>
  {/if}
</div>

<style>
  .card {
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .card-header, .card-footer {
    padding: 12px;
    background-color: #f8f9fa;
    border-bottom: 1px solid #ddd;
  }

  .card-body {
    padding: 16px;
  }
</style>
```

```svelte
<!-- UserProfileCard.svelte -->
<script>
  import Card from './Card.svelte';
  import Button from './Button.svelte';
  
  export let user = {};
  
  function editProfile() {
    // Логика редактирования профиля
    console.log('Редактирование профиля', user);
  }
  
  function saveProfile() {
    // Логика сохранения профиля
    console.log('Сохранение профиля', user);
  }
</script>

<Card title="Профиль пользователя">
  <div>
    <p>Имя: {user.name}</p>
    <p>Email: {user.email}</p>
    <p>Роль: {user.role}</p>
  </div>
  
  <div slot="footer">
    <Button variant="secondary" on:click={editProfile}>Редактировать</Button>
    <Button variant="primary" on:click={saveProfile}>Сохранить</Button>
  </div>
</Card>
```

### Пример 3: Композиция с использованием контекста

```svelte
<!-- ThemeContext.svelte -->
<script>
  import { setContext, getContext } from 'svelte';
  
  // Ключ контекста
  export const THEME_KEY = Symbol('theme');
  
  // Установка контекста
  export function setTheme(theme) {
    setContext(THEME_KEY, theme);
  }
  
  // Получение контекста
  export function getTheme() {
    return getContext(THEME_KEY);
  }
</script>
```

```svelte
<!-- ThemeProvider.svelte -->
<script>
  import { setTheme } from './ThemeContext.svelte';
  
  export let initialTheme = 'light';
  
  let theme = $state({
    current: initialTheme,
    toggle: () => {
      theme.current = theme.current === 'light' ? 'dark' : 'light';
    }
  });
  
  // Устанавливаем контекст
  setTheme(theme);
</script>

<div class="theme-provider theme-{theme.current}">
  <slot />
</div>

<style>
  .theme-provider {
    min-height: 100vh;
  }
  
  .theme-light {
    background-color: white;
    color: black;
  }
  
  .theme-dark {
    background-color: #333;
    color: white;
  }
</style>
```

```svelte
<!-- ThemedButton.svelte -->
<script>
  import { getTheme } from './ThemeContext.svelte';
  
  export let variant = 'primary';
  export let disabled = false;
  
  const theme = getTheme();
  
  function handleClick(event) {
    if (!disabled) {
      dispatch('click', event);
    }
  }
  
  const dispatch = createEventDispatcher();
  import { createEventDispatcher } from 'svelte';
</script>

<button 
  class="themed-button button-{variant} theme-{theme.current} {disabled ? 'disabled' : ''}"
  disabled={disabled}
  on:click={handleClick}
>
  <slot />
</button>

<style>
  .themed-button {
    padding: 8px 16px;
    border: 1px solid;
    border-radius: 4px;
    cursor: pointer;
  }
  
  .theme-light .button-primary {
    background-color: #007bff;
    color: white;
    border-color: #007bff;
  }
  
  .theme-light .button-secondary {
    background-color: #6c757d;
    color: white;
    border-color: #6c757d;
  }
  
  .theme-dark .button-primary {
    background-color: #0d6efd;
    color: white;
    border-color: #0d6efd;
  }
  
  .theme-dark .button-secondary {
    background-color: #5a6268;
    color: white;
    border-color: #5a6268;
  }
</style>
```

### Пример 4: Композиция с использованием модификаторов и делегирования событий

```svelte
<!-- ClickOutside.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  export let enabled = true;
  const dispatch = createEventDispatcher();
  
  function handleClickOutside(e) {
    if (enabled) {
      dispatch('clickOutside', e);
    }
  }
</script>

<div class="click-outside" on:click|stopPropagation on:click_outside={handleClickOutside}>
  <slot />
</div>
```

```svelte
<!-- Dropdown.svelte -->
<script>
  import ClickOutside from './ClickOutside.svelte';
  
  let isOpen = $state(false);
  
  function toggle() {
    isOpen = !isOpen;
  }
  
  function handleClickOutside() {
    isOpen = false;
  }
  
  function selectItem(item) {
    // Выбор элемента
    console.log('Выбран элемент:', item);
    isOpen = false;
  }
</script>

<div class="dropdown">
  <button class="dropdown-toggle" on:click={toggle}>
    Меню {isOpen ? '▲' : '▼'}
  </button>
  
  {#if isOpen}
    <ClickOutside on:clickOutside={handleClickOutside}>
      <ul class="dropdown-menu">
        <li on:click={() => selectItem('option1')}>Опция 1</li>
        <li on:click={() => selectItem('option2')}>Опция 2</li>
        <li on:click={() => selectItem('option3')}>Опция 3</li>
      </ul>
    </ClickOutside>
  {/if}
</div>

<style>
  .dropdown {
    position: relative;
    display: inline-block;
  }
  
  .dropdown-toggle {
    padding: 8px 12px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  .dropdown-menu {
    position: absolute;
    top: 100%;
    left: 0;
    background-color: white;
    border: 1px solid #ddd;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    list-style: none;
    padding: 0;
    margin: 4px 0 0 0;
    min-width: 120px;
    z-index: 1000;
  }
  
  .dropdown-menu li {
    padding: 8px 12px;
    cursor: pointer;
  }
  
  .dropdown-menu li:hover {
    background-color: #f8f9fa;
  }
</style>
```

## Паттерны композиции в Svelte

### 1. Container/Presenter Pattern
Разделение компонентов на контейнерные (управляющие данными) и презентационные (отвечающие за отображение):

```svelte
<!-- Presentational Component - UserCard.svelte -->
<script>
  export let user = {};
  
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();
</script>

<div class="user-card">
  <h3>{user.name}</h3>
  <p>{user.email}</p>
  <div class="user-actions">
    <button on:click={() => dispatch('edit', user)}>Редактировать</button>
    <button on:click={() => dispatch('delete', user.id)}>Удалить</button>
  </div>
</div>

<style>
  .user-card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
    margin: 8px 0;
  }
  
  .user-actions {
    margin-top: 12px;
    display: flex;
    gap: 8px;
  }
</style>
```

```svelte
<!-- Container Component - UserCardContainer.svelte -->
<script>
  import UserCard from './UserCard.svelte';
  
  let users = $state([]);
  let loading = $state(true);
  
  async function loadUsers() {
    try {
      // Загрузка пользователей
      const response = await fetch('/api/users');
      users = await response.json();
    } catch (error) {
      console.error('Ошибка загрузки пользователей:', error);
    } finally {
      loading = false;
    }
  }
  
  function handleEdit(user) {
    // Обработка редактирования
    console.log('Редактирование пользователя:', user);
  }
  
  function handleDelete(userId) {
    // Обработка удаления
    users = users.filter(user => user.id !== userId);
  }
  
  // Загружаем пользователей при монтировании
  loadUsers();
</script>

{#if loading}
  <div>Загрузка пользователей...</div>
{:else}
  {#each users as user}
    <UserCard 
      {user} 
      on:edit={handleEdit} 
      on:delete={handleDelete} 
    />
  {/each}
{/if}
```

### 2. Renderless Components (Store-based композиция)
Компоненты, которые не рендерят DOM, а предоставляют логику через stores:

```svelte
<!-- UserStore.svelte -->
<script>
  import { writable, derived } from 'svelte/store';
  
  // Создаем store для пользователей
  const usersStore = writable([]);
  const loadingStore = writable(false);
  const errorStore = writable(null);
  
  // Производный store для фильтрации
  const searchTerm = writable('');
  const filteredUsers = derived(
    [usersStore, searchTerm],
    ([$users, $searchTerm]) => {
      if (!$searchTerm) return $users;
      return $users.filter(user => 
        user.name.toLowerCase().includes($searchTerm.toLowerCase()) ||
        user.email.toLowerCase().includes($searchTerm.toLowerCase())
      );
    }
  );
  
  // Экспортируем stores и действия
  export { usersStore, loadingStore, errorStore, searchTerm, filteredUsers };
  
  // Функции для работы с пользователями
  export async function loadUsers() {
    loadingStore.set(true);
    errorStore.set(null);
    
    try {
      const response = await fetch('/api/users');
      if (!response.ok) throw new Error('Ошибка загрузки пользователей');
      const users = await response.json();
      usersStore.set(users);
    } catch (error) {
      errorStore.set(error.message);
    } finally {
      loadingStore.set(false);
    }
  }
  
  export function updateUser(updatedUser) {
    usersStore.update(users => 
      users.map(user => user.id === updatedUser.id ? updatedUser : user)
    );
  }
  
  export function deleteUser(userId) {
    usersStore.update(users => users.filter(user => user.id !== userId));
  }
</script>
```

```svelte
<!-- UserList.svelte -->
<script>
  import { 
    usersStore, 
    loadingStore, 
    errorStore, 
    searchTerm, 
    filteredUsers,
    loadUsers 
  } from './UserStore.svelte';
  
  // Подписываемся на stores
  let users = $derived($usersStore);
  let loading = $derived($loadingStore);
  let error = $derived($errorStore);
  let filtered = $derived($filteredUsers);
  
  // Загружаем пользователей при монтировании
  loadUsers();
</script>

<div class="user-list">
  <input 
    bind:value={$searchTerm} 
    placeholder="Поиск пользователей..." 
    class="search-input"
  />
  
  {#if loading}
    <div>Загрузка...</div>
  {:else if error}
    <div class="error">Ошибка: {error}</div>
  {:else}
    <ul>
      {#each filtered as user}
        <li class="user-item">
          {user.name} - {user.email}
        </li>
      {/each}
    </ul>
  {/if}
</div>

<style>
  .user-list {
    padding: 16px;
  }
  
  .search-input {
    width: 100%;
    padding: 8px;
    margin-bottom: 16px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }
  
  .user-item {
    padding: 8px;
    border-bottom: 1px solid #eee;
  }
  
  .error {
    color: red;
    padding: 8px;
  }
</style>
```

### 3. Compound Components
Компоненты, которые работают вместе как единое целое:

```svelte
<!-- Tabs.svelte -->
<script>
  import { getContext, setContext } from 'svelte';
  
  export let activeTab = '';
  
  const TABS_KEY = Symbol('tabs');
  let tabs = $state([]);
  
  // Устанавливаем контекст для дочерних компонентов
  setContext(TABS_KEY, {
    tabs,
    activeTab: $derived(activeTab),
    registerTab: (tabId) => {
      if (!tabs.includes(tabId)) {
        tabs = [...tabs, tabId];
      }
    },
    unregisterTab: (tabId) => {
      tabs = tabs.filter(id => id !== tabId);
    },
    setActiveTab: (tabId) => {
      activeTab = tabId;
    }
  });
</script>

<div class="tabs">
  <slot />
</div>

<style>
  .tabs {
    display: flex;
    flex-direction: column;
  }
</style>
```

```svelte
<!-- TabList.svelte -->
<script>
  import { getContext } from 'svelte';
  
  const TABS_KEY = Symbol('tabs');
  const tabsContext = getContext(TABS_KEY);
</script>

<div class="tab-list">
  <slot />
</div>

<style>
  .tab-list {
    display: flex;
    border-bottom: 1px solid #ddd;
  }
</style>
```

```svelte
<!-- Tab.svelte -->
<script>
  import { getContext } from 'svelte';
  
  export let id;
  export let disabled = false;
  
  const TABS_KEY = Symbol('tabs');
  const tabsContext = getContext(TABS_KEY);
  
  // Регистрируем таб при монтировании
  tabsContext.registerTab(id);
  
  // Отменяем регистрацию при размонтировании
  $: if (!tabsContext.tabs.includes(id) && id) {
    tabsContext.registerTab(id);
  }
  
  function handleClick() {
    if (!disabled) {
      tabsContext.setActiveTab(id);
    }
  }
  
  let isActive = $derived(tabsContext.activeTab === id);
</script>

<button 
  class="tab {isActive ? 'active' : ''} {disabled ? 'disabled' : ''}"
  on:click={handleClick}
  disabled={disabled}
>
  <slot />
</button>

<style>
  .tab {
    padding: 8px 16px;
    border: none;
    background: none;
    cursor: pointer;
    border-bottom: 2px solid transparent;
  }
  
  .tab.active {
    border-bottom-color: #007bff;
    font-weight: bold;
  }
  
  .tab.disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
```

```svelte
<!-- TabPanels.svelte -->
<script>
  import { getContext } from 'svelte';
  
  const TABS_KEY = Symbol('tabs');
  const tabsContext = getContext(TABS_KEY);
</script>

<div class="tab-panels">
  <slot />
</div>

<style>
  .tab-panels {
    margin-top: 16px;
  }
</style>
```

```svelte
<!-- TabPanel.svelte -->
<script>
  import { getContext } from 'svelte';
  
  export let id;
  
  const TABS_KEY = Symbol('tabs');
  const tabsContext = getContext(TABS_KEY);
  
  let isVisible = $derived(tabsContext.activeTab === id);
</script>

{#if isVisible}
  <div class="tab-panel">
    <slot />
  </div>
{/if}

<style>
  .tab-panel {
    padding: 16px 0;
  }
</style>
```

## Рекомендации по композиции в Svelte

### 1. Используйте слоты для максимальной гибкости
Слоты позволяют создавать очень гибкие компоненты:

```svelte
<!-- FlexibleCard.svelte -->
<script>
  export let title = '';
  export let showHeader = true;
  export let showFooter = true;
</script>

<div class="flexible-card">
  {#if showHeader}
    <header class="card-header">
      <h2>{title}</h2>
      <slot name="header-extra" />
    </header>
  {/if}
  
  <main class="card-body">
    <slot />
  </main>
  
  {#if showFooter}
    <footer class="card-footer">
      <slot name="footer" />
    </footer>
  {/if}
</div>
```

### 2. Используйте runes для управления состоянием
В Svelte 5 с runes управление состоянием становится еще более мощным:

```svelte
<!-- Counter.svelte -->
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);
  
  function increment() {
    count += 1;
  }
  
  function reset() {
    count = 0;
  }
</script>

<div class="counter">
  <p>Счетчик: {count}</p>
  <p>Удвоенное значение: {doubled}</p>
  <button on:click={increment}>+</button>
  <button on:click={reset}>Сброс</button>
</div>
```

### 3. Организуйте компоненты по функциональности
Создавайте логические группы компонентов:

```
components/
├── ui/                 # Базовые UI компоненты
│   ├── Button.svelte
│   ├── Input.svelte
│   └── Card.svelte
├── layout/             # Компоненты макета
│   ├── Header.svelte
│   ├── Sidebar.svelte
│   └── Footer.svelte
└── features/           # Компоненты с бизнес-логикой
    ├── UserCard.svelte
    └── SearchForm.svelte
```

### 4. Используйте TypeScript для лучшей типизации
```svelte
<script lang="ts">
  import type { User } from '../types';
  
  export let user: User;
  export let editable: boolean = false;
  
  const dispatch = createEventDispatcher<{
    updateUser: User;
    deleteUser: number;
  }>();
  
  import { createEventDispatcher } from 'svelte';
</script>
```

## Контрольные вопросы

1. Какие механизмы композиции доступны в Svelte?
2. В чем особенность использования слотов в Svelte?
3. Как работает контекст в Svelte и когда его использовать?
4. Какие паттерны композиции вы знаете и когда их использовать?
5. Как обеспечить типобезопасность в композиции компонентов Svelte?

## См. также

- [[Композиция компонентов]]
- [[Агрегация компонентов]]
- [[Svelte компоненты]]
- [[Слоты в Svelte]]
- [[Контекст в Svelte]]

## Внешние ресурсы

- [Svelte Tutorial - Component Composition](https://svelte.dev/tutorial/component-composition)
- [Svelte Documentation - Slots](https://svelte.dev/docs#template-syntax-slot)
- [Svelte 5 Runes Guide](https://svelte-5-preview.vercel.app/docs/runes)
---
aliases: [Инкапсуляция в Svelte, Приватность в Svelte-компонентах]
tags: [programming, svelte, encapsulation, components]
---

# Инкапсуляция в Svelte

Инкапсуляция в Svelte - это практика скрытия внутренней реализации компонентов и предоставления четкого публичного интерфейса. Svelte, как компилируемый фреймворк, обеспечивает инкапсуляцию через уникальные особенности своей архитектуры и синтаксиса.

## Обзор инкапсуляции в Svelte

Svelte обеспечивает инкапсуляцию через:
- Стилевую инкапсуляцию (scoped styles)
- Приватные переменные и функции
- Экспортируемые пропсы как публичный интерфейс
- Реактивные объявления
- Компонентную архитектуру

## Примеры инкапсуляции в компонентах

### Базовая инкапсуляция в Svelte

```svelte
<script>
  // Приватные переменные (недоступны извне)
  let privateCounter = 0;
  let privateState = 'initial';
  
  // Экспортируемые пропсы (публичный интерфейс)
  export let initialValue = 0;
  export let step = 1;
  export let label = 'Счетчик';
  
  // Приватные функции
  function privateIncrement() {
    privateCounter += step;
    updateState();
  }
  
  function privateDecrement() {
    privateCounter -= step;
    updateState();
  }
  
  function updateState() {
    if (privateCounter > 0) {
      privateState = 'positive';
    } else if (privateCounter < 0) {
      privateState = 'negative';
    } else {
      privateState = 'zero';
    }
  }
  
  // Реактивные объявления
  $: formattedCounter = `Значение: ${privateCounter}`;
  $: isPositive = privateCounter > 0;
  
  // Публичные методы (через createEventDispatcher)
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();
  
  function notifyChange() {
    dispatch('counterChange', { value: privateCounter, state: privateState });
  }
</script>

<div class="counter-container">
  <h3>{label}</h3>
  <p class:positive={isPositive} class:zero={privateCounter === 0}>
    {formattedCounter}
  </p>
  <p>Состояние: {privateState}</p>
  <button on:click={() => { privateIncrement(); notifyChange(); }}>+</button>
  <button on:click={() => { privateDecrement(); notifyChange(); }}>-</button>
  <button on:click={() => { privateCounter = initialValue; updateState(); notifyChange(); }}>Сброс</button>
</div>

<style>
  .counter-container {
    border: 1px solid #ccc;
    padding: 1rem;
    margin: 1rem 0;
    border-radius: 4px;
  }
  
  .positive {
    color: green;
    font-weight: bold;
  }
  
  .zero {
    color: orange;
    font-style: italic;
  }
</style>
```

### Инкапсуляция с использованием модулей скрипта

```svelte
<script context="module">
  // Код в context="module" выполняется один раз при импорте компонента
  // и инкапсулирует общую логику для всех экземпляров
  
  // Приватные данные на уровне модуля
  let componentInstances = 0;
  
  // Приватная функция на уровне модуля
  function generateInstanceId() {
    return ++componentInstances;
  }
  
  // Публичная функция, доступная при импорте компонента
  export function getComponentCount() {
    return componentInstances;
  }
</script>

<script>
  // Код в основном скрипте выполняется для каждого экземпляра компонента
  
  // Приватные данные экземпляра
  const instanceId = generateInstanceId();
  let count = 0;
  let history = [];
  
  // Пропсы (публичный интерфейс)
  export let maxCount = 10;
  export let minCount = 0;
  
  // Приватные функции
  function updateHistory(operation) {
    history = [...history, {
      operation,
      value: count,
      timestamp: new Date()
    }];
  }
  
  function increment() {
    if (count < maxCount) {
      count++;
      updateHistory('increment');
    }
  }
  
  function decrement() {
    if (count > minCount) {
      count--;
      updateHistory('decrement');
    }
  }
  
  // Реактивные объявления
  $: countText = `Значение: ${count} (макс: ${maxCount})`;
  $: canIncrement = count < maxCount;
  $: canDecrement = count > minCount;
  
  // Жизненный цикл
  onMount(() => {
    console.log(`Компонент ${instanceId} смонтирован`);
  });
  
  import { onMount } from 'svelte';
</script>

<div class="advanced-counter">
  <h3>Продвинутый счетчик #{instanceId}</h3>
  <p>{countText}</p>
  <div class="controls">
    <button disabled={!canDecrement} on:click={decrement}>-</button>
    <span class="count-display">{count}</span>
    <button disabled={!canIncrement} on:click={increment}>+</button>
  </div>
  <details>
    <summary>История изменений</summary>
    <ul>
      {#each history as entry, i}
        <li>{entry.operation} на {entry.value} в {entry.timestamp.toLocaleTimeString()}</li>
      {/each}
    </ul>
  </details>
</div>

<style>
  .advanced-counter {
    border: 2px solid #4CAF50;
    padding: 1rem;
    margin: 1rem 0;
    border-radius: 8px;
  }
  
  .controls {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }
  
  .count-display {
    font-size: 1.5rem;
    font-weight: bold;
    min-width: 3rem;
    text-align: center;
  }
</style>
```

## Инкапсуляция с пользовательскими встроенными функциями

```svelte
<script>
  // user-utils.js - инкапсулированная логика работы с пользователем
  const API_BASE = 'https://api.example.com';
  
  // Приватная функция для форматирования данных пользователя
  function formatUserData(userData) {
    return {
      ...userData,
      fullName: `${userData.firstName} ${userData.lastName}`,
      email: userData.email.toLowerCase(),
      joinDate: new Date(userData.createdAt).toLocaleDateString('ru-RU')
    };
  }
  
  // Приватная функция для валидации данных
  function validateUserData(data) {
    return data && data.email && data.email.includes('@');
  }
  
  // Публичный интерфейс
  export async function fetchUser(userId) {
    try {
      const response = await fetch(`${API_BASE}/users/${userId}`);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const userData = await response.json();
      
      if (!validateUserData(userData)) {
        throw new Error('Invalid user data');
      }
      
      return formatUserData(userData);
    } catch (error) {
      console.error('Error fetching user:', error);
      throw error;
    }
  }
  
  // Приватное состояние компонента
  let user = null;
  let loading = false;
  let error = null;
  
  // Пропсы
  export let userId;
  
  // Асинхронная загрузка данных
  $: if (userId) {
    loading = true;
    error = null;
    
    fetchUser(userId)
      .then(userData => {
        user = userData;
      })
      .catch(err => {
        error = err.message;
      })
      .finally(() => {
        loading = false;
      });
  }
</script>

<div class="user-profile">
  {#if loading}
    <p>Загрузка профиля...</p>
  {:else if error}
    <p class="error">Ошибка: {error}</p>
  {:else if user}
    <h2>{user.fullName}</h2>
    <p>Email: {user.email}</p>
    <p>Дата регистрации: {user.joinDate}</p>
  {:else}
    <p>Пользователь не найден</p>
  {/if}
</div>

<style>
  .user-profile {
    border: 1px solid #ddd;
    padding: 1rem;
    border-radius: 4px;
  }
  
  .error {
    color: red;
  }
</style>
```

## Инкапсуляция с использованием хранилищ (stores)

```javascript
// stores/userStore.js
import { writable, derived } from 'svelte/store';

// Приватное состояние
const privateUsers = new Map();
let currentUser = null;

// Приватные функции
function validateUser(user) {
  return user && user.id && user.email;
}

function updateUserInCache(user) {
  if (validateUser(user)) {
    privateUsers.set(user.id, { ...user, lastUpdated: Date.now() });
  }
}

// Создание хранилища с инкапсулированным состоянием
function createUserStore() {
  const { subscribe, set, update } = writable({
    user: null,
    isAuthenticated: false,
    permissions: []
  });
  
  return {
    subscribe,
    
    // Публичные методы
    login: async (credentials) => {
      try {
        const response = await fetch('/api/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(credentials)
        });
        
        const data = await response.json();
        
        if (data.success && data.user) {
          const userData = {
            ...data.user,
            permissions: data.permissions || []
          };
          
          // Инкапсулированное обновление состояния
          set({
            user: userData,
            isAuthenticated: true,
            permissions: userData.permissions
          });
          
          // Обновляем кэш
          updateUserInCache(userData);
          currentUser = userData;
          
          return { success: true };
        }
        
        return { success: false, error: 'Login failed' };
      } catch (error) {
        return { success: false, error: error.message };
      }
    },
    
    logout: () => {
      set({
        user: null,
        isAuthenticated: false,
        permissions: []
      });
      currentUser = null;
    },
    
    // Приватные методы не экспортируются
  };
}

// Публичное хранилище
export const userStore = createUserStore();

// Производные хранилища (инкапсулированная логика на основе основного хранилища)
export const isAdmin = derived(
  userStore,
  $userStore => $userStore.permissions.includes('admin')
);

export const canEdit = derived(
  userStore,
  $userStore => $userStore.permissions.includes('edit')
);
```

### Использование хранилища в компоненте

```svelte
<script>
  import { userStore, isAdmin, canEdit } from './stores/userStore';
  
  // Приватное состояние компонента
  let showAdminPanel = false;
  
  // Приватные функции
  function toggleAdminPanel() {
    $: showAdminPanel = !showAdminPanel && $isAdmin;
  }
  
  // Приватная реактивная логика
  $: if ($userStore.isAuthenticated) {
    console.log('Пользователь аутентифицирован:', $userStore.user.name);
  }
</script>

<div class="app-layout">
  <header>
    <h1>Приложение</h1>
    {#if $userStore.isAuthenticated}
      <p>Привет, {$userStore.user.name}!</p>
      <button on:click={() => userStore.logout()}>Выйти</button>
    {/if}
  </header>
  
  <main>
    {#if $isAdmin}
      <button on:click={toggleAdminPanel}>
        {showAdminPanel ? 'Скрыть' : 'Показать'} админ-панель
      </button>
    {/if}
    
    {#if showAdminPanel}
      <div class="admin-panel">
        <h2>Админ-панель</h2>
        <p>Доступно только администраторам</p>
      </div>
    {/if}
  </main>
</div>

<style>
  .app-layout {
    padding: 1rem;
  }
  
  .admin-panel {
    border: 2px solid red;
    padding: 1rem;
    margin-top: 1rem;
  }
</style>
```

## Практические рекомендации

1. **Используйте `context="module"`** для инкапсуляции логики, общей для всех экземпляров компонента
2. **Экспортируйте только необходимые переменные** как пропсы - остальные остаются приватными
3. **Используйте реактивные объявления** (`$:`) для автоматических вычислений
4. **Создавайте пользовательские хранилища** для инкапсуляции глобального состояния
5. **Используйте scoped стили** для изоляции CSS
6. **Ограничивайте передачу данных** через слоты только необходимым минимумом

## Заключение

Инкапсуляция в Svelte достигается через уникальные особенности фреймворка, включая приватные переменные в скриптах, реактивные объявления и контекст модуля. Архитектура Svelte поощряет чистое разделение между приватной логикой и публичным интерфейсом компонентов.

## См. также
- [[Инкапсуляция-в-JavaScript]]
- [[Инкапсуляция-в-React]]
- [[Инкапсуляция-в-Vue]]
- [[Приватные-поля-и-методы]]
- [[Svelte-компоненты]]

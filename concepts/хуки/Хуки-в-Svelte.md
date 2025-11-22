---
aliases: [Svelte Hooks, Хуки Svelte, Свельт Хуки]
tags: [svelte, hooks, frontend, javascript]
---

# Хуки в Svelte

## Введение

В отличие от React и Vue, Svelte не использует термин "хуки" в традиционном смысле. Однако в Svelte есть концепции и функции, которые выполняют схожие задачи - управление состоянием, жизненным циклом компонентов и побочными эффектами. Эти функции обеспечивают аналогичную функциональность, но с более интуитивным и минимальным синтаксисом.

## Функции жизненного цикла

В Svelte доступны специальные функции жизненного цикла, которые позволяют выполнять код на разных этапах существования компонента.

### onMount

`onMount` вызывается один раз, когда компонент впервые вставлен в DOM. Это эквивалент `componentDidMount` в React или `onMounted` в Vue.

```svelte
<script>
  import { onMount } from 'svelte';

  let data = null;

  onMount(async () => {
    const response = await fetch('/api/data');
    data = await response.json();
  });
</script>

{#if data}
  <div>Данные: {JSON.stringify(data)}</div>
{:else}
  <div>Загрузка...</div>
{/if}
```

### onDestroy

`onDestroy` вызывается непосредственно перед тем, как компонент будет уничтожен. Это эквивалент `componentWillUnmount` в React.

```svelte
<script>
  import { onMount, onDestroy } from 'svelte';

  let intervalId;

  onMount(() => {
    intervalId = setInterval(() => {
      console.log('Тик');
    }, 1000);
  });

  onDestroy(() => {
    clearInterval(intervalId);
  });
</script>

<p>Таймер запущен в консоли</p>
```

### beforeUpdate и afterUpdate

Эти функции позволяют выполнять код до и после обновления компонента.

```svelte
<script>
  import { beforeUpdate, afterUpdate } from 'svelte';

  let count = 0;

  beforeUpdate(() => {
    console.log(`Перед обновлением: ${count}`);
  });

  afterUpdate(() => {
    console.log(`После обновления: ${count}`);
  });
</script>

<button on:click={() => count++}>Счетчик: {count}</button>
```

### tick

`tick` - это специальная функция, которая возвращает промис, разрешающийся после следующего обновления DOM.

```svelte
<script>
  import { tick } from 'svelte';

  let div;
  let text = 'Привет';

  async function handleUpdate() {
    text = 'Обновлено';
    
    // Ожидаем, пока DOM обновится
    await tick();
    
    // Теперь можем получить актуальные размеры элемента
    console.log(div.offsetHeight);
  }
</script>

<div bind:this={div}>{text}</div>
<button on:click={handleUpdate}>Обновить</button>
```

## Реактивные объявления

Svelte использует реактивные объявления для автоматического обновления значений при изменении зависимостей.

```svelte
<script>
  let count = 0;
  
  // Реактивное объявление
  $: doubled = count * 2;
  
  // Реактивный блок
  $: {
    if (count > 5) {
      alert('Счетчик больше 5!');
    }
  }
  
  function increment() {
    count += 1;
  }
</script>

<p>Счетчик: {count}</p>
<p>Удвоенное значение: {doubled}</p>
<button on:click={increment}>+</button>
```

## Сторы (Stores)

Stores в Svelte - это реактивные контейнеры для состояния, аналогичные контексту в React или хранилищам в Vue.

### writable

`writable` создает изменяемое хранилище.

```javascript
// stores.js
import { writable } from 'svelte/store';

export const count = writable(0);
export const user = writable({ name: 'Аноним', age: 0 });
```

```svelte
<!-- Component.svelte -->
<script>
  import { count, user } from './stores.js';
  
  // Подписка на изменение значения
  $: currentCount = $count;
  $: currentUser = $user;
  
  function increment() {
    count.update(n => n + 1);
  }
  
  function updateUserName(newName) {
    user.update(u => ({ ...u, name: newName }));
  }
</script>

<p>Счетчик: {$count}</p>
<p>Пользователь: {$user.name}, {$user.age} лет</p>
<button on:click={increment}>+</button>
<button on:click={() => updateUserName('Иван')}>Изменить имя</button>
```

### readable

`readable` создает хранилище только для чтения.

```javascript
// timeStore.js
import { readable } from 'svelte/store';

export const time = readable(new Date(), function start(set) {
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);

  return function stop() {
    clearInterval(interval);
  };
});
```

```svelte
<script>
  import { time } from './timeStore.js';
</script>

<p>Текущее время: {$time.toLocaleTimeString()}</p>
```

### derived

`derived` создает хранилище, производное от одного или нескольких других хранилищ.

```javascript
// stores.js
import { writable, derived } from 'svelte/store';

export const count = writable(0);

export const doubled = derived(
  count,
  $count => $count * 2
);

export const tripleDoubled = derived(
  [count, doubled],
  ([$count, $doubled]) => $count * $doubled
);
```

## Автоматическая реактивность

Одним из ключевых отличий Svelte является автоматическая реактивность. Вам не нужно явно указывать зависимости, как в `useEffect` в React.

```svelte
<script>
  let name = 'Мир';
  let greeting = 'Привет';
  
  // Это автоматически обновится при изменении name или greeting
  $: message = `${greeting}, ${name}!`;
  
  function updateName() {
    name = 'Svelte';
  }
</script>

<p>{message}</p>
<button on:click={updateName}>Обновить имя</button>
```

## Привязки (Bindings)

Привязки позволяют создавать двустороннюю реактивность между значениями и DOM.

```svelte
<script>
  let name = 'Svelte';
  let visible = true;
  let count = 0;
</script>

<input bind:value={name} placeholder="Введите имя">
{#if visible}
  <p>Привет, {name}!</p>
{/if}

<input type="checkbox" bind:checked={visible}> Показать приветствие

<div bind:this={element} class="box">
  Размеры: {element?.offsetWidth} x {element?.offsetHeight}px
</div>

<script>
  let element;
</script>
```

## Контекст

Контекст в Svelte позволяет передавать данные между компонентами без пропсов.

```svelte
<!-- Parent.svelte -->
<script>
  import { setContext } from 'svelte';
  import Child from './Child.svelte';
  
  const api = {
    getData: () => Promise.resolve('данные'),
    baseUrl: 'https://api.example.com'
  };
  
  setContext('api', api);
</script>

<Child />
```

```svelte
<!-- Child.svelte -->
<script>
  import { getContext } from 'svelte';
  
  const api = getContext('api');
  
  let data = null;
  
  $: if (api) {
    api.getData().then(result => {
      data = result;
    });
  }
</script>

<p>Данные из контекста: {data}</p>
```

## Заключение

Хотя Svelte не использует термин "хуки", он предоставляет мощные и интуитивные инструменты для управления состоянием, жизненным циклом и побочными эффектами. Концепции Svelte часто проще и требуют меньше кода по сравнению с традиционными хуками в других фреймворках.

См. также:
- [[Пользовательские-хуки]]
- [[Лучшие-практики-хуков]]
- [[Хуки-в-React]]
- [[Хуки-в-Vue]]
---
aliases: [Svelte Effects, Svelte реактивность, Эффекты в Свельте]
tags: [svelte, frontend, effects, reactivity]
---

# Эффекты-в-Svelte

## Введение

В Svelte эффекты реализованы через уникальную систему реактивности, которая отличается от подходов в React и Vue. Вместо хуков или watch-функций, Svelte использует автоматическое отслеживание зависимостей и специальные функции жизненного цикла.

## Система реактивности Svelte

В Svelte реактивность встроена в язык. Компоненты автоматически реагируют на изменения переменных:

```svelte
<script>
  let count = 0;

  // Это автоматически обновится при изменении count
  $: doubled = count * 2;

  function handleClick() {
    count += 1;
  }
</script>

<p>Счетчик: {count}</p>
<p>Удвоенное значение: {doubled}</p>
<button on:click={handleClick}>+</button>
```

## Типы эффектов в Svelte

### 1. Реактивные объявления ($:)

Реактивные объявления запускаются автоматически при изменении зависимостей:

```svelte
<script>
  let name = 'мир';
  let greeting;

  // Реактивное объявление - выполнится при изменении name
  $: greeting = name ? `Привет, ${name}!` : '';

  // Блок кода - выполнится при изменении любых зависимостей в блоке
  $: {
    console.log(`Имя изменилось на: ${name}`);
    if (name.length > 10) {
      alert('Слишком длинное имя!');
    }
  }

  function updateName(event) {
    name = event.target.value;
  }
</script>

<input value={name} on:input={updateName} />
<p>{greeting}</p>
```

### 2. Функции жизненного цикла

Svelte предоставляет функции жизненного цикла для выполнения эффектов на определенных этапах:

```svelte
<script>
  import { onMount, beforeUpdate, afterUpdate, onDestroy } from 'svelte';

  let element;

  onMount(() => {
    console.log('Компонент смонтирован');
    
    // Возврат функции для очистки
    return () => {
      console.log('Очистка при размонтировании');
    };
  });

  beforeUpdate(() => {
    console.log('Компонент будет обновлен');
  });

  afterUpdate(() => {
    console.log('Компонент был обновлен');
  });

  onDestroy(() => {
    console.log('Компонент будет уничтожен');
  });
</script>

<div bind:this={element}>
  Содержимое компонента
</div>
```

## onMount - Эквивалент componentDidMount

Используется для выполнения эффектов при монтировании компонента:

```svelte
<script>
  import { onMount } from 'svelte';

  let data = null;
  let loading = true;

  onMount(async () => {
    try {
      const response = await fetch('/api/data');
      data = await response.json();
    } catch (error) {
      console.error('Ошибка загрузки данных:', error);
    } finally {
      loading = false;
    }
  });
</script>

{#if loading}
  <p>Загрузка...</p>
{:else}
  <pre>{JSON.stringify(data, null, 2)}</pre>
{/if}
```

## $: эффекты с условиями

Можно создавать эффекты, которые выполняются только при определенных условиях:

```svelte
<script>
  let count = 0;
  let previousCount = 0;

  // Эффект, который запускается только при увеличении счетчика
  $: if (count > previousCount) {
    console.log(`Счетчик увеличился с ${previousCount} до ${count}`);
    previousCount = count;
  }

  function increment() {
    count += 1;
  }
</script>

<button on:click={increment}>Счетчик: {count}</button>
```

## Работа с DOM в эффектах

```svelte
<script>
  import { onMount } from 'svelte';

  let canvasElement;
  let animationFrame;

  onMount(() => {
    const canvas = canvasElement;
    const ctx = canvas.getContext('2d');

    function draw() {
      // Очистка холста
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      
      // Рисование
      ctx.fillStyle = 'red';
      ctx.fillRect(Math.random() * canvas.width, Math.random() * canvas.height, 50, 50);
      
      animationFrame = requestAnimationFrame(draw);
    }

    draw();

    // Очистка при размонтировании
    return () => {
      cancelAnimationFrame(animationFrame);
    };
  });
</script>

<canvas bind:this={canvasElement} width="400" height="300"></canvas>
```

## Подписка на внешние события

```svelte
<script>
  import { onMount, onDestroy } from 'svelte';

  let windowWidth = window.innerWidth;

  const handleResize = () => {
    windowWidth = window.innerWidth;
  };

  onMount(() => {
    window.addEventListener('resize', handleResize);
  });

  onDestroy(() => {
    window.removeEventListener('resize', handleResize);
  });
</script>

<p>Ширина окна: {windowWidth}px</p>
```

## Практические примеры

### Синхронизация с localStorage

```svelte
<script>
  import { onMount } from 'svelte';

  let name = '';

  // Синхронизация с localStorage при изменении name
  $: {
    localStorage.setItem('name', name);
  }

  onMount(() => {
    // Загрузка значения из localStorage при монтировании
    const savedName = localStorage.getItem('name');
    if (savedName) {
      name = savedName;
    }
  });
</script>

<input bind:value={name} placeholder="Введите имя" />
<p>Привет, {name || 'незнакомец'}!</p>
```

### Таймеры и интервалы

```svelte
<script>
  import { onMount, onDestroy } from 'svelte';

  let seconds = 0;
  let intervalId;

  onMount(() => {
    intervalId = setInterval(() => {
      seconds += 1;
    }, 1000);
  });

  onDestroy(() => {
    if (intervalId) {
      clearInterval(intervalId);
    }
  });

  function resetTimer() {
    seconds = 0;
  }
</script>

<p>Прошло секунд: {seconds}</p>
<button on:click={resetTimer}>Сбросить</button>
```

### Асинхронные эффекты

```svelte
<script>
  import { onMount } from 'svelte';

  let searchTerm = '';
  let results = [];
  let loading = false;

  // Реактивный эффект для поиска
  $: if (searchTerm) {
    loading = true;
    // Отмена предыдущего запроса
    clearTimeout(this.searchTimeout);
    
    this.searchTimeout = setTimeout(async () => {
      try {
        const response = await fetch(`/api/search?q=${encodeURIComponent(searchTerm)}`);
        results = await response.json();
      } catch (error) {
        console.error('Ошибка поиска:', error);
        results = [];
      } finally {
        loading = false;
      }
    }, 300); // Дебаунс 300мс
  }

  function handleInput(event) {
    searchTerm = event.target.value;
  }
</script>

<input on:input={handleInput} placeholder="Поиск..." />
{#if loading}
  <p>Поиск...</p>
{/if}
<ul>
  {#each results as result}
    <li>{result.title}</li>
  {/each}
</ul>
```

## Распространенные ошибки

### 1. Бесконечные циклы

```svelte
<script>
  // ПЛОХО - может вызвать бесконечный цикл
  $: {
    if (count < 10) {
      count++; // Изменение зависимости внутри эффекта
    }
  }

  // ХОРОШО - проверка условия
  $: {
    if (count < 10 && previousCount !== count) {
      previousCount = count;
      count++;
    }
  }
</script>
```

### 2. Забытая очистка ресурсов

```svelte
<script>
  // ПЛОХО - не очищаем подписки
  onMount(() => {
    window.addEventListener('resize', handleResize);
    // Никогда не удаляем событие!
  });

  // ХОРОШО - очищаем ресурсы
  onMount(() => {
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });
</script>
```

## Оптимизация эффектов

### Избегание ненужных перезапусков

```svelte
<script>
  let user = { name: '', email: '' };
  let displayName = '';

  // ПЛОХО - перезапуск при любом изменении объекта user
  $: displayName = user.name.toUpperCase();

  // ХОРОШО - деструктуризация для точного отслеживания
  $: ({ name } = user, displayName = name.toUpperCase());
</script>
```

### Комбинирование эффектов

```svelte
<script>
  let count = 0;
  let multiplier = 1;

  // Комбинированный эффект
  $: {
    const result = count * multiplier;
    console.log(`Результат: ${result}`);
    
    if (result > 100) {
      console.log('Результат больше 100!');
    }
  }
</script>
```

## Сравнение с другими фреймворками

| Особенность | Svelte | React | Vue |
|-------------|--------|-------|-----|
| Реактивность | Автоматическая | Хуки (useEffect) | Composition API (watch) |
| Синтаксис эффектов | `$:` объявления | `useEffect()` | `watch()`, `watchEffect()` |
| Отслеживание зависимостей | Автоматическое | Явное указание | Автоматическое или явное |
| Функции жизненного цикла | `onMount`, `onDestroy` и др. | `useEffect` с зависимостями | `onMounted`, `onUnmounted` и др. |

## Заключение

Эффекты в Svelte обеспечивают мощную и интуитивную систему реактивности. Автоматическое отслеживание зависимостей и встроенные функции жизненного цикла делают работу с побочными эффектами более простой и понятной по сравнению с другими фреймворками.

См. также:
- [[Побочные-эффекты]]
- [[Управление-эффектами]]
- [[Эффекты-в-React]]
- [[Эффекты-в-Vue]]
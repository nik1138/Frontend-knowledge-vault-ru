---
aliases: [Svelte директивы, Svelte Directives, Директивы Svelte]
tags: [svelte, frontend, directives, javascript]
---

# Директивы в Svelte

## Обзор

Директивы в Svelte - это особые инструкции, которые вы добавляете к элементам DOM для управления их поведением, условиями отображения, обработкой событий и другими аспектами. В отличие от Vue, где директивы имеют префикс `v-`, в Svelte директивы имеют префикс `svelte:` или не имеют префикса вовсе, интегрируясь в HTML-синтаксис.

## Встроенные директивы

### Условная отрисовка

#### `#if`, `#else`, `#else if`
Условная отрисовка элементов на основе выражений.

```svelte
<script>
  let user = { loggedIn: false };
  
  function toggleUserState() {
    user.loggedIn = !user.loggedIn;
  }
</script>

{#if user.loggedIn}
  <button on:click={toggleUserState}>
    Выйти
  </button>
{:else}
  <button on:click={toggleUserState}>
    Войти
  </button>
{/if}

<!-- Использование else if -->
{#if x > 10}
  <p>Больше 10</p>
{:else if x > 5}
  <p>Больше 5</p>
{:else}
  <p>5 или меньше</p>
{/if}
```

#### `#each`
Отрисовка списков элементов с возможностью доступа к индексу и ключу.

```svelte
<script>
  let items = [
    { id: 1, name: 'Яблоко' },
    { id: 2, name: 'Банан' },
    { id: 3, name: 'Апельсин' }
  ];
</script>

<ul>
  {#each items as item (item.id)}
    <li>{item.name}</li>
  {/each}
</ul>

<!-- С доступом к индексу -->
{#each items as item, index}
  <p>{index}: {item.name}</p>
{/each}

<!-- С деструктуризацией -->
{#each items as { id, name }}
  <p>{id}: {name}</p>
{/each}
```

### Обработка событий

#### `on:`
Привязка обработчиков событий к элементам.

```svelte
<script>
  let count = 0;
  
  function handleClick() {
    count += 1;
  }
  
  function handleMouseOver(event) {
    console.log('Мышь над элементом');
  }
</script>

<button on:click={handleClick} on:mouseover={handleMouseOver}>
  Нажатий: {count}
</button>

<!-- Модификаторы событий -->
<form on:submit|preventDefault={handleSubmit}>
  <input on:keydown|stopPropagation on:click|once={handleClickOnce} />
</form>
```

### Привязка данных

#### `bind:`
Двустороннее связывание данных между компонентами и переменными.

```svelte
<script>
  let name = '';
  let checked = false;
  let selectedValue = 'option1';
  let textValue = '';
</script>

<!-- Привязка к input -->
<input bind:value={name} placeholder="Введите имя" />
<p>Привет, {name}!</p>

<!-- Привязка к checkbox -->
<input type="checkbox" bind:checked={checked} />
<p>Флажок: {checked ? 'отмечен' : 'не отмечен'}</p>

<!-- Привязка к select -->
<select bind:value={selectedValue}>
  <option value="option1">Опция 1</option>
  <option value="option2">Опция 2</option>
</select>

<!-- Привязка к textarea -->
<textarea bind:value={textValue}></textarea>

<!-- Привязка к атрибутам элементов -->
<input bind:files={fileList} type="file" multiple />
<video bind:currentTime={time} bind:duration bind:paused />
```

### Слоты

#### `slot`
Определение мест для вставки контента в компонентах.

```svelte
<!-- Child.svelte -->
<div class="card">
  <header>
    <slot name="header">Заголовок по умолчанию</slot>
  </header>
  <main>
    <slot>Содержимое по умолчанию</slot>
  </main>
  <footer>
    <slot name="footer" />
  </footer>
</div>

<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';
</script>

<Child>
  <h1 slot="header">Мой заголовок</h1>
  <p>Основное содержимое</p>
  <p slot="footer">Мой футер</p>
</Child>
```

### Специальные директивы

#### `svelte:self`
Позволяет компоненту рекурсивно включать самого себя.

```svelte
<script>
  export let item;
</script>

{#if item.isParent}
  <ul>
    <li>{item.name}</li>
    {#each item.children as child}
      <svelte:self item={child} />
    {/each}
  </ul>
{:else}
  <li>{item.name}</li>
{/if}
```

#### `svelte:component`
Отображение динамического компонента.

```svelte
<script>
  import BlueComponent from './Blue.svelte';
  import RedComponent from './Red.svelte';
  
  let component = BlueComponent;
  let name = 'blue';
  
  $: component = name === 'blue' ? BlueComponent : RedComponent;
</script>

<svelte:component this={component} />
```

#### `svelte:element`
Отображение динамического элемента.

```svelte
<script>
  let currentElement = 'div';
</script>

<svelte:element this={currentElement} class="dynamic-element">
  Это динамический элемент
</svelte:element>
```

### Мета-директивы

#### `svelte:window`
Привязка к событиям окна.

```svelte
<script>
  import { onMount } from 'svelte';
  
  let width = 0;
  let height = 0;
  
  function handleResize() {
    width = window.innerWidth;
    height = window.innerHeight;
  }
  
  onMount(() => {
    handleResize();
  });
</script>

<svelte:window on:resize={handleResize} />

<p>Размер окна: {width} x {height}</p>
```

#### `svelte:body`
Привязка к событиям тела документа.

```svelte
<script>
  function handleClick() {
    console.log('Клик по body');
  }
</script>

<svelte:body on:click={handleClick} />
```

#### `svelte:head`
Добавление содержимого в тег `<head>`.

```svelte
<svelte:head>
  <title>Моя страница</title>
  <meta name="description" content="Описание страницы" />
  <link rel="stylesheet" href="/custom.css" />
</svelte:head>
```

#### `svelte:options`
Настройка опций компонента.

```svelte
<svelte:options
  tag="my-custom-element"
  namespace="svg"
  accessors={true}
  immutable={false}
  data={{ custom: 'data' }}
/>
```

#### `svelte:fragment`
Используется внутри компонентов для определения содержимого слота без дополнительного элемента-обертки.

```svelte
<!-- Parent.svelte -->
<Child>
  <div slot="content">
    <h2>Заголовок</h2>
    <p>Параграф</p>
  </div>
</Child>

<!-- Child.svelte -->
<div class="wrapper">
  <svelte:fragment slot="content">
    <!-- Содержимое будет вставлено без дополнительной обертки -->
  </svelte:fragment>
</div>
```

### Анимация и переходы

#### `animate:`
Директива для анимации при изменении позиции элементов в списке.

```svelte
<script>
  import { flip } from 'svelte/animate';
  import { quintOut } from 'svelte/easing';
  
  let items = [1, 2, 3];
  
  function shuffle() {
    items = [...items].sort(() => Math.random() - 0.5);
  }
</script>

<button on:click={shuffle}>Перемешать</button>

{#each items as n (n)}
  <div animate:flip={{ 
    delay: 250, 
    duration: 250, 
    easing: quintOut 
  }}>
    {n}
  </div>
{/each}
```

#### `transition:`, `in:`, `out:`, `transition:`
Директивы для анимации появления и исчезновения элементов.

```svelte
<script>
  import { fade, fly, slide } from 'svelte/transition';
  import { cubicOut } from 'svelte/easing';
  
  let visible = true;
</script>

<label>
  <input type="checkbox" bind:checked={visible}>
  Показать/скрыть
</label>

{#if visible}
  <div 
    in:fly={{ x: 200, duration: 1000 }}
    out:fade={{ duration: 500 }}
  >
    Летающий элемент
  </div>
  
  <div transition:slide>
    Слайд-элемент
  </div>
{/if}
```

## Модификаторы событий

Svelte поддерживает различные модификаторы событий:

- `|stopPropagation` - останавливает всплытие события
- `|preventDefault` - предотвращает стандартное поведение
- `|passive` - указывает, что обработчик не вызывает `preventDefault`
- `|nonpassive` - указывает, что обработчик может вызвать `preventDefault`
- `|capture` - обработчик вызывается на этапе захвата
- `|once` - обработчик срабатывает только один раз
- `|self` - обработчик срабатывает только если событие произошло на самом элементе

```svelte
<form on:submit|preventDefault={handleSubmit}>
  <input 
    on:click|stopPropagation 
    on:keydown|once={handleOnce}
    on:focus|self={handleSelfFocus}
  />
</form>
```

## Продвинутые возможности

### Контекстные директивы

#### `bind:this`
Привязка к самому DOM-элементу или компоненту.

```svelte
<script>
  let canvasElement;
  
  function draw() {
    if (canvasElement) {
      const ctx = canvasElement.getContext('2d');
      // Рисование на canvas
    }
  }
</script>

<canvas bind:this={canvasElement} width="200" height="200" />
<button on:click={draw}>Нарисовать</button>
```

### Деструктуризация в директивах

Возможно использовать деструктуризацию в директивах `each`:

```svelte
{#each items as { id, name, description = 'Без описания' }}
  <div>
    <h3>{name}</h3>
    <p>{description}</p>
  </div>
{/each}
```

## Сравнение с другими фреймворками

- В отличие от Vue, Svelte интегрирует директивы в HTML-синтаксис
- По сравнению с React, Svelte предоставляет встроенные решения для распространенных задач
- В отличие от Angular, Svelte имеет более простой синтаксис и меньше шаблонного кода

> [!tip] 
> Для более глубокого понимания директив в Svelte рекомендуется ознакомиться с [[Встроенные-директивы]] и [[Пользовательские-директивы]]

> [!note] 
> Директивы в Svelte транспилируются в эффективный JavaScript во время сборки, что делает приложения быстрее.

## Лучшие практики

1. **Используйте `#each` с ключом** для оптимизации производительности при работе со списками
2. **Предпочитайте `bind:` для двустороннего связывания данных** вместо ручного обновления переменных
3. **Используйте модификаторы событий** для упрощения обработки событий
4. **Избегайте сложных выражений в директивах** - выносите логику в функции
5. **Используйте `svelte:window` и `svelte:body`** для глобальных обработчиков событий

## См. также

- [[Директивы-в-Vue]]
- [[Директивы-в-Angular]]
- [[Пользовательские-директивы]]
- [[Встроенные-директивы]]
---
aliases: [Svelte Animations, Анимации в Svelte, Svelte Animation]
tags: [svelte, animation, frontend, web-development, transitions, motion]
---

# Анимации-в-Svelte

Svelte предоставляет встроенные возможности для создания анимаций, отличающиеся своей простотой и эффективностью. В отличие от других фреймворков, Svelte компилирует анимации в эффективный JavaScript, что делает их быстрыми и легкими.

## Встроенные переходы и анимации

### Базовые переходы
Svelte предоставляет встроенные функции переходов для анимации элементов при монтировании и размонтировании.

```svelte
<script>
  import { fade, fly, slide } from 'svelte/transition';
  
  let visible = true;
</script>

<div>
  <button on:click={() => visible = !visible}>
    Переключить видимость
  </button>
  
  {#if visible}
    <div 
      transition:fade 
      class="content"
    >
      Появляющийся и исчезающий контент
    </div>
  {/if}
</div>

<style>
  .content {
    padding: 20px;
    background-color: #f0f0f0;
    border-radius: 4px;
    margin-top: 10px;
  }
</style>
```

### Параметризованные переходы
Переходы могут принимать параметры для настройки анимации.

```svelte
<script>
  import { fly, fade } from 'svelte/transition';
  
  let visible = true;
</script>

<div>
  <button on:click={() => visible = !visible}>
    Переключить
  </button>
  
  {#if visible}
    <div 
      transition:fly={{ 
        x: 100, 
        y: 25, 
        duration: 1000,
        easing: cubicOut 
      }}
      class="content"
    >
      Летящий элемент
    </div>
  {/if}
</div>

<script>
  import { cubicOut } from 'svelte/easing';
</script>

<style>
  .content {
    padding: 20px;
    background-color: #42b983;
    color: white;
    border-radius: 4px;
    margin-top: 10px;
  }
</style>
```

## Встроенные функции переходов

### fade
Плавное появление/исчезновение элемента.

```svelte
<script>
  import { fade } from 'svelte/transition';
  let show = true;
</script>

<button on:click={() => show = !show}>Переключить</button>

{#if show}
  <div 
    transition:fade={{ duration: 1000 }}
    class="fade-element"
  >
    Плавно появляющийся элемент
  </div>
{/if}
```

### fly
Перемещение элемента с анимацией.

```svelte
<script>
  import { fly } from 'svelte/transition';
  let visible = true;
</script>

<button on:click={() => visible = !visible}>Переключить</button>

{#if visible}
  <div 
    transition:fly={{
      x: 100,
      y: 25,
      duration: 1500,
      easing: cubicOut
    }}
    class="fly-element"
  >
    Летящий элемент
  </div>
{/if}
```

### slide
Скольжение элемента с расширением/сжатием.

```svelte
<script>
  import { slide } from 'svelte/transition';
  let visible = true;
</script>

<button on:click={() => visible = !visible}>Переключить</button>

{#if visible}
  <div 
    transition:slide={{
      duration: 500,
      easing: cubicOut
    }}
    class="slide-element"
  >
    Скользящий элемент
  </div>
{/if}
```

### scale
Изменение масштаба элемента.

```svelte
<script>
  import { scale } from 'svelte/transition';
  let visible = true;
</script>

<button on:click={() => visible = !visible}>Переключить</button>

{#if visible}
  <div 
    transition:scale={{
      duration: 500,
      easing: cubicOut,
      start: 0.5
    }}
    class="scale-element"
  >
    Изменяющийся в масштабе элемент
  </div>
{/if}
```

## Анимации с помощью motion

### crossfade
Специальная функция для анимации перехода между элементами.

```svelte
<script>
  import { crossfade } from 'svelte/transition';
  
  let items = [
    { id: 1, name: 'Элемент 1' },
    { id: 2, name: 'Элемент 2' },
    { id: 3, name: 'Элемент 3' }
  ];
  
  let selected = null;
  
  const [send, receive] = crossfade({
    duration: 400,
    easing: cubicOut
  });
</script>

{#each items as item (item.id)}
  {#if item.id !== selected}
    <div
      class="source"
      on:click={() => selected = item.id}
      on:mouseenter={() => selected = item.id}
      in:receive="{{key: item.id}}"
      out:send="{{key: item.id}}"
    >
      {item.name}
    </div>
  {/if}
{/each}

{#if selected}
  <div
    class="target"
    on:click={() => selected = null}
    in:receive="{{key: selected}}"
    out:send="{{key: selected}}"
  >
    {#each items as item (item.id)}
      {#if item.id === selected}
        {item.name} (выбран)
      {/if}
    {/each}
  </div>
{/if}

<style>
  .source, .target {
    padding: 10px;
    margin: 5px;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s;
  }
  
  .source {
    background-color: #f0f0f0;
  }
  
  .target {
    background-color: #42b983;
    color: white;
  }
  
  .source:hover {
    background-color: #e0e0e0;
  }
</style>
```

## Пользовательские переходы

Создание собственных функций переходов для специфических анимаций.

```svelte
<script>
  import { quintOut } from 'svelte/easing';
  
  // Пользовательская функция перехода
  function pop(node, params) {
    const { delay = 0, duration = 400, easing = quintOut, start = 0, opacity = 0 } = params;
    
    const o = +getComputedStyle(node).opacity;
    
    return {
      delay,
      duration,
      easing,
      css: t => {
        const ts = 1 - t;
        return `transform: scale(${1 - (1 - start) * ts}); opacity: ${t * o}`;
      }
    };
  }
  
  let visible = true;
</script>

<button on:click={() => visible = !visible}>Переключить</button>

{#if visible}
  <div 
    transition:pop={{
      duration: 800,
      start: 0.5
    }}
    class="pop-element"
  >
    Пользовательская анимация
  </div>
{/if}

<style>
  .pop-element {
    padding: 20px;
    background-color: #ff6b6b;
    color: white;
    border-radius: 4px;
    margin-top: 10px;
  }
</style>
```

## Анимации с помощью Svelte Motion

Для более сложных анимаций можно использовать библиотеку `svelte-motion`.

```bash
npm install svelte-motion
```

```svelte
<script>
  import { motion } from 'svelte-motion';
  import { useState } from 'svelte';
  
  let [x, setX] = useState(0);
</script>

<div class="container">
  <button on:click={() => setX(x === 0 ? 100 : 0)}>
    Переместить
  </button>
  
  <motion.div
    animate={{ x }}
    transition={{ type: "spring", stiffness: 300, damping: 20 }}
    class="motion-element"
  >
    Анимированный элемент
  </motion.div>
</div>

<style>
  .container {
    position: relative;
    height: 200px;
  }
  
  .motion-element {
    width: 100px;
    height: 100px;
    background-color: #42b983;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
  }
</style>
```

## Анимации списков

Анимация добавления, удаления и перемещения элементов в списках.

```svelte
<script>
  import { flip } from 'svelte/animate';
  
  let items = [
    { id: 1, name: 'Элемент 1', color: '#FF6B6B' },
    { id: 2, name: 'Элемент 2', color: '#4ECDC4' },
    { id: 3, name: 'Элемент 3', color: '#45B7D1' }
  ];
  
  let nextId = 4;
  
  function addItem() {
    const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7'];
    items = [
      ...items,
      { 
        id: nextId++, 
        name: `Элемент ${nextId - 1}`, 
        color: colors[Math.floor(Math.random() * colors.length)] 
      }
    ];
  }
  
  function removeItem(id) {
    items = items.filter(item => item.id !== id);
  }
  
  function shuffle() {
    items = [...items].sort(() => Math.random() - 0.5);
  }
</script>

<div class="list-container">
  <div class="controls">
    <button on:click={addItem}>Добавить элемент</button>
    <button on:click={shuffle}>Перемешать</button>
  </div>
  
  <div class="list">
    {#each items as item (item.id)}
      <div
        animate:flip
        class="list-item"
        style="background-color: {item.color}"
        on:click={() => removeItem(item.id)}
      >
        {item.name}
      </div>
    {/each}
  </div>
</div>

<style>
  .list-container {
    max-width: 400px;
    margin: 0 auto;
  }
  
  .controls {
    margin-bottom: 20px;
    text-align: center;
  }
  
  .controls button {
    margin: 0 5px;
    padding: 8px 16px;
    background-color: #42b983;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  .controls button:hover {
    background-color: #35495e;
  }
  
  .list {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }
  
  .list-item {
    padding: 15px;
    border-radius: 8px;
    color: white;
    cursor: pointer;
    transition: transform 0.2s, box-shadow 0.2s;
    text-align: center;
  }
  
  .list-item:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  }
</style>
```

## Анимации с использованием store

Использование Svelte store для управления анимационными состояниями.

```svelte
<!-- stores/animations.js -->
<script context="module">
  import { writable } from 'svelte/store';
  
  export const animationProgress = writable(0);
  export const isAnimating = writable(false);
</script>

<!-- AnimatedComponent.svelte -->
<script>
  import { animationProgress, isAnimating } from './stores/animations.js';
  import { tweened } from 'svelte/motion';
  
  const animatedValue = tweened(0, {
    duration: 1000,
    easing: t => t // линейная анимация
  });
  
  let targetValue = 100;
  
  function animateToTarget() {
    isAnimating.set(true);
    animatedValue.set(targetValue, {
      duration: 1500,
      easing: t => t * t // квадратичная анимация
    }).then(() => {
      isAnimating.set(false);
    });
  }
  
  $: console.log('Текущее значение:', $animatedValue);
</script>

<div class="animator">
  <h3>Текущее значение: {$animatedValue.toFixed(0)}</h3>
  <button on:click={animateToTarget}>Анимировать до {targetValue}</button>
  
  <div class="progress-bar">
    <div 
      class="progress-fill"
      style="width: {$animatedValue}%"
    ></div>
  </div>
</div>

<style>
  .animator {
    padding: 20px;
    max-width: 400px;
    margin: 0 auto;
  }
  
  .progress-bar {
    height: 20px;
    background-color: #e9ecef;
    border-radius: 10px;
    overflow: hidden;
    margin-top: 15px;
  }
  
  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, #42b983, #35495e);
    transition: width 0.3s ease;
  }
  
  button {
    padding: 10px 20px;
    background-color: #42b983;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  button:hover {
    background-color: #35495e;
  }
</style>
```

## Практические примеры

### Анимированный аккордеон
```svelte
<script>
  import { slide } from 'svelte/transition';
  
  let activeIndex = null;
  
  const items = [
    { title: 'Заголовок 1', content: 'Содержимое первого элемента аккордеона' },
    { title: 'Заголовок 2', content: 'Содержимое второго элемента аккордеона' },
    { title: 'Заголовок 3', content: 'Содержимое третьего элемента аккордеона' }
  ];
  
  function toggleItem(index) {
    activeIndex = activeIndex === index ? null : index;
  }
</script>

<div class="accordion">
  {#each items as item, i}
    <div class="accordion-item">
      <div 
        class="accordion-header" 
        class:active={activeIndex === i}
        on:click={() => toggleItem(i)}
      >
        <span>{item.title}</span>
        <span class="arrow">▼</span>
      </div>
      
      {#if activeIndex === i}
        <div 
          transition:slide={{ 
            duration: 300,
            easing: cubicOut
          }}
          class="accordion-content"
        >
          {item.content}
        </div>
      {/if}
    </div>
  {/each}
</div>

<style>
  .accordion {
    max-width: 600px;
    margin: 0 auto;
  }
  
  .accordion-item {
    border: 1px solid #ddd;
    margin-bottom: 5px;
    border-radius: 4px;
    overflow: hidden;
  }
  
  .accordion-header {
    padding: 15px;
    background-color: #f5f5f5;
    cursor: pointer;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: background-color 0.2s;
  }
  
  .accordion-header:hover {
    background-color: #e9ecef;
  }
  
  .accordion-header.active {
    background-color: #42b983;
    color: white;
  }
  
  .arrow {
    transition: transform 0.3s ease;
  }
  
  .accordion-header.active .arrow {
    transform: rotate(180deg);
  }
  
  .accordion-content {
    padding: 15px;
    background-color: white;
  }
</style>
```

### Анимированный табы
```svelte
<script>
  import { fade } from 'svelte/transition';
  
  let activeTab = 0;
  
  const tabs = [
    { id: 0, title: 'Вкладка 1', content: 'Содержимое первой вкладки' },
    { id: 1, title: 'Вкладка 2', content: 'Содержимое второй вкладки' },
    { id: 2, title: 'Вкладка 3', content: 'Содержимое третьей вкладки' }
  ];
</script>

<div class="tabs">
  <div class="tab-headers">
    {#each tabs as tab}
      <button
        class:active={activeTab === tab.id}
        on:click={() => activeTab = tab.id}
      >
        {tab.title}
      </button>
    {/each}
  </div>
  
  <div class="tab-content">
    {#each tabs as tab}
      {#if activeTab === tab.id}
        <div 
          in:fade={{ duration: 300 }}
          out:fade={{ duration: 150 }}
          class="tab-panel"
        >
          {tab.content}
        </div>
      {/if}
    {/each}
  </div>
</div>

<style>
  .tabs {
    max-width: 600px;
    margin: 0 auto;
  }
  
  .tab-headers {
    display: flex;
    border-bottom: 1px solid #ddd;
  }
  
  .tab-headers button {
    padding: 12px 24px;
    background: none;
    border: none;
    cursor: pointer;
    border-bottom: 2px solid transparent;
  }
  
  .tab-headers button.active {
    border-bottom-color: #42b983;
    color: #42b983;
    font-weight: bold;
  }
  
  .tab-content {
    min-height: 200px;
    padding: 20px 0;
  }
  
  .tab-panel {
    padding: 0 20px;
  }
</style>
```

## Лучшие практики

### 1. Оптимизация производительности
- Используйте `transform` и `opacity` для анимаций
- Избегайте анимации свойств, вызывающих перерисовку
- Используйте `will-change` для элементов, которые будут анимироваться

```svelte
<style>
  .optimized-animation {
    will-change: transform, opacity;
    transform: translateZ(0); /* Включает аппаратное ускорение */
  }
</style>
```

### 2. Учет предпочтений пользователя
```svelte
<script>
  import { onMount } from 'svelte';
  
  let prefersReducedMotion = false;
  let showAnimation = true;
  
  onMount(() => {
    if (window.matchMedia) {
      const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
      prefersReducedMotion = mediaQuery.matches;
      showAnimation = !prefersReducedMotion;
      
      const handler = (e) => {
        prefersReducedMotion = e.matches;
        showAnimation = !e.matches;
      };
      
      mediaQuery.addEventListener('change', handler);
      
      return () => {
        mediaQuery.removeEventListener('change', handler);
      };
    }
  });
</script>

{#if showAnimation}
  <div 
    transition:fade={{ duration: 300 }}
    class="content"
  >
    Анимированный контент
  </div>
{:else}
  <div class="content">
    Обычный контент
  </div>
{/if}
```

> [!tip]
> Всегда учитывайте предпочтения пользователей, отключающих анимации, для обеспечения доступности интерфейса.

### 3. Правильное использование easing-функций
```svelte
<script>
  import { 
    linear, 
    cubicOut, 
    elasticOut, 
    backOut 
  } from 'svelte/easing';
  
  // Разные easing-функции для разных типов анимаций
  // cubicOut - для большинства анимаций
  // elasticOut - для забавных пружинных эффектов
  // backOut - для анимаций с небольшим отскоком
</script>
```

## Заключение

Svelte предоставляет мощные и интуитивные инструменты для создания анимаций:

- Встроенные функции переходов (`fade`, `fly`, `slide`, `scale`)
- Анимации с помощью `svelte/animate` (`flip` для списков)
- Возможность создания пользовательских переходов
- Поддержка библиотек анимаций, таких как `svelte-motion`
- Интеграция с Svelte store для управления анимационными состояниями

Особенности Svelte позволяют создавать эффективные анимации с минимальным объемом кода, при этом обеспечивая отличную производительность благодаря компиляции.

## См. также
- [[CSS-анимации]]
- [[JavaScript-анимации]]
- [[Анимации-в-React]]
- [[Анимации-в-Vue]]
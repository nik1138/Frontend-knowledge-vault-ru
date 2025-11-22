---
aliases: [События в Svelte, Svelte Events, Обработка событий в Svelte]
tags: [svelte, javascript, events, frontend, programming]
---

# События-в-Svelte

## Введение

Svelte предоставляет мощную и интуитивно понятную систему обработки событий, которая сочетает простоту использования с гибкостью. В отличие от других фреймворков, Svelte обрабатывает события на этапе компиляции, что делает его особенно эффективным.

## Базовая обработка событий

### Синтаксис обработки событий

В Svelte для обработки событий используется простой синтаксис `on:имя_события`:

```svelte
<script>
  let count = 0;
  
  function handleClick() {
    count += 1;
  }
  
  function handleMouseOver() {
    console.log('Курсор над элементом');
  }
</script>

<button on:click={handleClick}>
  Кликнуто {count} раз
</button>

<div on:mouseover={handleMouseOver} class="hover-area">
  Наведи на меня курсор
</div>
```

### Inline-обработчики

```svelte
<script>
  let message = 'Привет, Svelte!';
</script>

<button on:click={() => message = 'Сообщение изменено!'}>
  {message}
</button>

<!-- Передача аргументов -->
<button on:click={() => console.log('Клик с аргументами')}>
  Клик с логом
</button>
```

## Модификаторы событий

Svelte поддерживает модификаторы событий, которые изменяют поведение обработчиков:

```svelte
<script>
  function handleSubmit(event) {
    event.preventDefault();
    console.log('Форма отправлена');
  }
  
  function handleStopPropagation(event) {
    console.log('Событие на дочернем элементе');
    event.stopPropagation();
  }
  
  function handleOnce() {
    console.log('Это сообщение появится только один раз');
  }
  
  function handleSelf(event) {
    if (event.target === this) {
      console.log('Клик на самом элементе');
    }
  }
</script>

<!-- .preventDefault -->
<form on:submit|preventDefault={handleSubmit}>
  <input type="text" placeholder="Введите что-то">
  <button type="submit">Отправить</button>
</form>

<!-- .stopPropagation -->
<div on:click={handleStopPropagation}>
  <button on:click={handleStopPropagation}>
    Кнопка (всплытие остановлено)
  </button>
</div>

<!-- .once - обработчик сработает только один раз -->
<button on:click|once={handleOnce}>
  Клик один раз
</button>

<!-- .self - проверка, что событие произошло на этом элементе -->
<div 
  on:click={handleSelf}
  style="padding: 20px; border: 1px solid #ccc;"
>
  Клик на этом div (не на дочерних элементах)
  <button>Кнопка внутри</button>
</div>
```

## Специальные модификаторы

### Модификаторы клавиатуры

```svelte
<script>
  let inputValue = '';
  
  function handleEnter() {
    console.log('Нажат Enter:', inputValue);
  }
  
  function handleEscape() {
    console.log('Нажат Escape');
    inputValue = '';
  }
  
  function handleArrowUp() {
    console.log('Стрелка вверх');
  }
  
  function handleCtrlS(event) {
    if (event.ctrlKey) {
      console.log('Ctrl+S нажат');
      event.preventDefault();
    }
  }
</script>

<!-- Enter -->
<input 
  bind:value={inputValue}
  on:keydown|enter={handleEnter}
  placeholder="Нажмите Enter"
>

<!-- Escape -->
<input 
  bind:value={inputValue}
  on:keydown|escape={handleEscape}
  placeholder="Нажмите Escape"
>

<!-- Стрелки -->
<input 
  on:keydown|arrow-up={handleArrowUp}
  placeholder="Стрелка вверх"
>

<!-- Комбинации клавиш -->
<input 
  on:keydown={handleCtrlS}
  placeholder="Ctrl+S для сохранения"
>
```

### Модификаторы мыши

```svelte
<script>
  function handleLeftClick() {
    console.log('Левая кнопка мыши');
  }
  
  function handleRightClick(event) {
    event.preventDefault(); // Предотвращаем контекстное меню
    console.log('Правая кнопка мыши');
  }
  
  function handleMiddleClick() {
    console.log('Средняя кнопка мыши');
  }
  
  function handleDoubleClick() {
    console.log('Двойной клик');
  }
</script>

<!-- Левая кнопка мыши (по умолчанию) -->
<button on:click={handleLeftClick}>
  Левый клик
</button>

<!-- Правая кнопка мыши -->
<div on:contextmenu|preventDefault={handleRightClick}>
  Правый клик (контекстное меню заблокировано)
</div>

<!-- Средняя кнопка мыши -->
<button on:auxclick={handleMiddleClick}>
  Средний клик
</button>

<!-- Двойной клик -->
<div on:dblclick={handleDoubleClick}>
  Дважды кликни меня
</div>
```

## Работа с формами

### Обработка событий формы

```svelte
<script>
  let formData = {
    name: '',
    email: '',
    message: ''
  };
  
  let errors = {};
  
  function validateForm() {
    errors = {};
    
    if (!formData.name.trim()) {
      errors.name = 'Имя обязательно';
    }
    
    if (!formData.email.trim()) {
      errors.email = 'Email обязателен';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
      errors.email = 'Некорректный email';
    }
    
    return Object.keys(errors).length === 0;
  }
  
  function handleSubmit(event) {
    event.preventDefault();
    
    if (validateForm()) {
      console.log('Форма отправлена:', formData);
      // Здесь будет отправка данных
    } else {
      console.log('Форма содержит ошибки:', errors);
    }
  }
  
  function handleInput(event, field) {
    formData[field] = event.target.value;
    // Очищаем ошибку при вводе
    if (errors[field]) {
      delete errors[field];
    }
  }
</script>

<form on:submit|preventDefault={handleSubmit}>
  <div class="form-group">
    <label for="name">Имя:</label>
    <input 
      id="name"
      type="text" 
      value={formData.name}
      on:input={(e) => handleInput(e, 'name')}
      class:invalid={!!errors.name}
    >
    {#if errors.name}
      <span class="error">{errors.name}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <label for="email">Email:</label>
    <input 
      id="email"
      type="email" 
      value={formData.email}
      on:input={(e) => handleInput(e, 'email')}
      class:invalid={!!errors.email}
    >
    {#if errors.email}
      <span class="error">{errors.email}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <label for="message">Сообщение:</label>
    <textarea 
      id="message"
      value={formData.message}
      on:input={(e) => {
        formData.message = e.target.value;
      }}
    ></textarea>
  </div>
  
  <button type="submit">Отправить</button>
</form>

<style>
  .form-group {
    margin-bottom: 1rem;
  }
  
  .form-group label {
    display: block;
    margin-bottom: 0.5rem;
  }
  
  .form-group input,
  .form-group textarea {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  .form-group input.invalid {
    border-color: #ff0000;
  }
  
  .error {
    color: #ff0000;
    font-size: 0.875rem;
    margin-top: 0.25rem;
    display: block;
  }
</style>
```

## Компонентные события

### Создание и использование пользовательских событий

```svelte
<!-- ChildComponent.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  const dispatch = createEventDispatcher();
  
  let count = 0;
  
  function handleClick() {
    count++;
    // Генерация пользовательского события
    dispatch('count-incremented', { 
      count, 
      message: `Счётчик увеличен до ${count}` 
    });
  }
  
  function notifyParent() {
    dispatch('notification', {
      type: 'info',
      message: 'Сообщение от дочернего компонента'
    });
  }
</script>

<div class="child-component">
  <h3>Дочерний компонент</h3>
  <p>Счётчик: {count}</p>
  <button on:click={handleClick}>Увеличить счётчик</button>
  <button on:click={notifyParent}>Уведомить родителя</button>
</div>
```

```svelte
<!-- ParentComponent.svelte -->
<script>
  import ChildComponent from './ChildComponent.svelte';
  
  function handleCountIncremented(event) {
    console.log('Счётчик увеличен:', event.detail);
  }
  
  function handleNotification(event) {
    console.log('Получено уведомление:', event.detail);
  }
</script>

<div class="parent-component">
  <h2>Родительский компонент</h2>
  <ChildComponent 
    on:count-incremented={handleCountIncremented}
    on:notification={handleNotification}
  />
</div>
```

## Использование bind:this для прямого доступа к элементам

```svelte
<script>
  import { onMount } from 'svelte';
  
  let canvasElement;
  let animationFrame;
  
  onMount(() => {
    if (canvasElement) {
      const ctx = canvasElement.getContext('2d');
      
      function draw() {
        // Очищаем canvas
        ctx.clearRect(0, 0, canvasElement.width, canvasElement.height);
        
        // Рисуем анимированный круг
        const time = Date.now() / 1000;
        const x = 50 + Math.sin(time) * 40;
        const y = 50 + Math.cos(time) * 40;
        
        ctx.beginPath();
        ctx.arc(x, y, 10, 0, Math.PI * 2);
        ctx.fillStyle = '#ff6b6b';
        ctx.fill();
        
        animationFrame = requestAnimationFrame(draw);
      }
      
      draw();
    }
    
    return () => {
      if (animationFrame) {
        cancelAnimationFrame(animationFrame);
      }
    };
  });
</script>

<canvas 
  bind:this={canvasElement}
  width="200" 
  height="100"
  style="border: 1px solid #ccc;"
></canvas>
```

## Работа с событиями вне компонента

### Управление событиями window/document

```svelte
<script>
  import { onMount, onDestroy } from 'svelte';
  
  let windowWidth = 0;
  let windowHeight = 0;
  let scrollPosition = 0;
  
  function handleResize() {
    windowWidth = window.innerWidth;
    windowHeight = window.innerHeight;
  }
  
  function handleScroll() {
    scrollPosition = window.scrollY;
  }
  
  onMount(() => {
    // Инициализация значений
    handleResize();
    handleScroll();
    
    // Добавление обработчиков
    window.addEventListener('resize', handleResize);
    window.addEventListener('scroll', handleScroll);
    
    // Возврат функции очистки
    return () => {
      window.removeEventListener('resize', handleResize);
      window.removeEventListener('scroll', handleScroll);
    };
  });
</script>

<div class="window-info">
  <p>Ширина окна: {windowWidth}px</p>
  <p>Высота окна: {windowHeight}px</p>
  <p>Позиция прокрутки: {scrollPosition}px</p>
</div>
```

## Делегирование событий

### Использование делегирования для эффективности

```svelte
<script>
  let items = [
    { id: 1, name: 'Элемент 1', active: false },
    { id: 2, name: 'Элемент 2', active: false },
    { id: 3, name: 'Элемент 3', active: false }
  ];
  
  function handleItemClick(event) {
    const button = event.target.closest('button');
    if (button) {
      const id = parseInt(button.dataset.id);
      toggleItem(id);
    }
  }
  
  function toggleItem(id) {
    items = items.map(item => 
      item.id === id 
        ? { ...item, active: !item.active } 
        : item
    );
  }
</script>

<div class="item-list" on:click={handleItemClick}>
  {#each items as item (item.id)}
    <div class="item" class:active={item.active}>
      <span>{item.name}</span>
      <button data-id={item.id}>
        {item.active ? 'Деактивировать' : 'Активировать'}
      </button>
    </div>
  {/each}
</div>

<style>
  .item-list {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  .item.active {
    background-color: #e3f2fd;
  }
  
  button {
    padding: 0.25rem 0.5rem;
    background-color: #2196f3;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  button:hover {
    background-color: #0b7dda;
  }
</style>
```

## Пользовательские события с деталями

### Расширенное использование пользовательских событий

```svelte
<!-- EventGenerator.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  const dispatch = createEventDispatcher();
  
  let clicks = 0;
  let lastClickTime = null;
  
  function handleClick() {
    clicks++;
    const now = Date.now();
    const timeSinceLast = lastClickTime ? now - lastClickTime : 0;
    lastClickTime = now;
    
    dispatch('click-statistics', {
      clicks,
      timeSinceLast,
      timestamp: now,
      isDouble: timeSinceLast < 300 // двойной клик если менее 300мс
    });
  }
</script>

<button on:click={handleClick}>
  Кликай для статистики
</button>
```

```svelte
<!-- EventConsumer.svelte -->
<script>
  import EventGenerator from './EventGenerator.svelte';
  
  let statistics = [];
  
  function handleStatistics(event) {
    statistics = [event.detail, ...statistics.slice(0, 9)]; // последние 10
  }
</script>

<div>
  <EventGenerator on:click-statistics={handleStatistics} />
  
  <h3>Статистика кликов:</h3>
  <ul>
    {#each statistics as stat (stat.timestamp)}
      <li>
        Клик #{stat.clicks}, 
        с предыдущего: {stat.timeSinceLast}мс, 
        {stat.isDouble ? 'Двойной' : 'Одинарный'}
      </li>
    {/each}
  </ul>
</div>
```

## Работа с событиями в компонентах с проксированием

### Проксирование событий

```svelte
<!-- ButtonWrapper.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  export let disabled = false;
  export let text = 'Кнопка';
  
  const dispatch = createEventDispatcher();
  
  function handleClick(event) {
    // Выполняем логику в обертке
    console.log('Кнопка была нажата в обертке');
    
    // Проксируем событие дальше
    dispatch('click', event);
  }
</script>

<button 
  on:click={handleClick}
  disabled={disabled}
>
  {text}
</button>
```

```svelte
<!-- Usage.svelte -->
<script>
  import ButtonWrapper from './ButtonWrapper.svelte';
  
  function handleButtonClick() {
    console.log('Кнопка была нажата в родительском компоненте');
  }
</script>

<ButtonWrapper 
  text="Проксируемая кнопка"
  on:click={handleButtonClick}
/>
```

## Современные подходы и паттерны

### Использование Actions для обработки событий

```svelte
<!-- actions.js -->
export function clickOutside(node, callback) {
  const handleClick = (event) => {
    if (!node.contains(event.target)) {
      callback();
    }
  };

  document.addEventListener('click', handleClick, true);

  return {
    destroy() {
      document.removeEventListener('click', handleClick, true);
    }
  };
}

export function longpress(node, callback) {
  let timer;
  
  const handleMousedown = () => {
    timer = setTimeout(callback, 500);
  };
  
  const handleMouseup = () => {
    if (timer) {
      clearTimeout(timer);
      timer = null;
    }
  };
  
  node.addEventListener('mousedown', handleMousedown);
  node.addEventListener('mouseup', handleMouseup);
  node.addEventListener('mouseleave', handleMouseup);
  
  return {
    destroy() {
      node.removeEventListener('mousedown', handleMousedown);
      node.removeEventListener('mouseup', handleMouseup);
      node.removeEventListener('mouseleave', handleMouseup);
      
      if (timer) {
        clearTimeout(timer);
      }
    }
  };
}
```

```svelte
<script>
  import { clickOutside, longpress } from './actions.js';
  
  let showMenu = false;
  let longPressed = false;
  
  function handleOutsideClick() {
    showMenu = false;
  }
  
  function handleLongPress() {
    longPressed = true;
    setTimeout(() => longPressed = false, 1000);
  }
</script>

<div class="container">
  <button on:click={() => showMenu = !showMenu}>
    Показать меню
  </button>
  
  {#if longPressed}
    <div class="long-press-indicator">Долгое нажатие!</div>
  {/if}
  
  {#if showMenu}
    <div 
      use:clickOutside={handleOutsideClick}
      class="menu"
    >
      <div>Пункт 1</div>
      <div>Пункт 2</div>
      <div>Пункт 3</div>
    </div>
  {/if}
  
  <button 
    use:longpress={handleLongPress}
    class="long-press-button"
  >
    Долгое нажатие
  </button>
</div>

<style>
  .container {
    position: relative;
    padding: 20px;
  }
  
  .menu {
    position: absolute;
    top: 100%;
    left: 0;
    background: white;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    z-index: 100;
  }
  
  .menu div {
    padding: 8px 12px;
    cursor: pointer;
  }
  
  .menu div:hover {
    background-color: #f5f5f5;
  }
  
  .long-press-button {
    margin-top: 10px;
    padding: 10px;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 4px;
  }
  
  .long-press-indicator {
    position: fixed;
    top: 20px;
    right: 20px;
    background-color: #ff9800;
    color: white;
    padding: 10px;
    border-radius: 4px;
    z-index: 1000;
  }
</style>
```

## Заключение

Система обработки событий в Svelte предоставляет мощные и гибкие возможности при сохранении простоты использования. Модификаторы событий, пользовательские события, делегирование и Actions делают Svelte отличным выбором для создания интерактивных приложений.

## Смежные темы

- [[Обработка-событий]]
- [[Всплытие-и-перехват-событий]]
- [[События-в-React]]
- [[События-в-Vue]]
- [[Компонентная-архитектура]]
- [[Фреймворки]]
- [[Хуки]]
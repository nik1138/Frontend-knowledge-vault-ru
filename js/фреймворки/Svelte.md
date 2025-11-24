---
aliases: [SvelteJS, Svelte 3+, Свельт]
tags: [javascript, фреймворк, svelte, frontend]
---

# Svelte

## Обзор

Svelte - это современный фреймворк JavaScript для создания пользовательских интерфейсов, который отличается от других фреймворков тем, что переносит большую часть работы в фазу сборки, а не в браузер. В 2025 году Svelte становится все более популярным благодаря своей производительности и простоте.

## Основные особенности

- **Компиляция в чистый JavaScript** - Нет виртуального DOM
- **Малый размер** - Меньше кода в браузере
- **Простота синтаксиса** - Минимальная кривая обучения
- **Реактивность по умолчанию** - Автоматическое обновление при изменении переменных
- **Встроенные функции** - Маршрутизация, анимация и т.д.
- **Легкий вес** - Меньше бандл приложения

## Установка и настройка

### Создание нового проекта

```bash
# Используя degit
npx degit sveltejs/template my-svelte-app
cd my-svelte-app
npm install
npm run dev

# Или с помощью create-vite
npm create vite@latest my-svelte-app -- --template svelte
cd my-svelte-app
npm install
npm run dev
```

### Основная структура файла компонента

```svelte
<script>
  // Логика компонента
  let count = 0;
  
  function increment() {
    count += 1;
  }
</script>

<style>
  /* Стили компонента */
  .counter {
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
</style>

<!-- Шаблон компонента -->
<div class="counter">
  <p>Счетчик: {count}</p>
  <button on:click={increment}>Увеличить</button>
</div>
```

## Основные концепции

### Реактивность

В Svelte реактивность встроена по умолчанию:

```svelte
<script>
  let name = 'мир';
  $: greeting = `Привет, ${name}!`;
  
  function updateName() {
    name = 'Svelte';
  }
</script>

<h1>{greeting}</h1>
<button on:click={updateName}>Изменить имя</button>
```

### Привязка данных

Двусторонняя привязка данных с использованием `bind`:

```svelte
<script>
  let userInput = '';
</script>

<input bind:value={userInput} placeholder="Введите текст" />
<p>Вы ввели: {userInput}</p>
```

### Жизненный цикл компонента

```svelte
<script>
  import { onMount, beforeUpdate, onDestroy } from 'svelte';

  onMount(() => {
    console.log('Компонент смонтирован');
  });

  beforeUpdate(() => {
    console.log('Компонент будет обновлен');
  });

  onDestroy(() => {
    console.log('Компонент будет уничтожен');
  });
</script>
```

## Практические примеры

### Простое приложение "Список задач"

```svelte
<script>
  import { onMount } from 'svelte';
  
  let todos = [];
  let newTodo = '';
  
  function addTodo() {
    if (newTodo.trim() !== '') {
      todos = [...todos, { id: Date.now(), text: newTodo, completed: false }];
      newTodo = '';
    }
  }
  
  function toggleTodo(id) {
    todos = todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
  }
  
  // Загрузка данных при монтировании
  onMount(async () => {
    // Здесь можно загрузить данные из API
    // todos = await fetchTodos();
  });
</script>

<style>
  .todo-app {
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
  }

  .input-section {
    display: flex;
    margin-bottom: 20px;
  }

  .input-section input {
    flex: 1;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px 0 0 4px;
  }

  .input-section button {
    padding: 8px 16px;
    border: 1px solid #ccc;
    border-left: none;
    border-radius: 0 4px 4px 0;
    background-color: #f5f5f5;
  }

  .todo-list {
    list-style: none;
    padding: 0;
  }

  .todo-list li {
    padding: 10px;
    border: 1px solid #eee;
    margin-bottom: 5px;
    cursor: pointer;
    border-radius: 4px;
  }

  .todo-list li.completed {
    text-decoration: line-through;
    color: #999;
  }
</style>

<div class="todo-app">
  <h1>Список задач</h1>
  <div class="input-section">
    <input 
      bind:value={newTodo} 
      on:keydown={(e) => e.key === 'Enter' && addTodo()} 
      placeholder="Добавить задачу"
    />
    <button on:click={addTodo}>Добавить</button>
  </div>
  <ul class="todo-list">
    {#each todos as todo (todo.id)}
      <li 
        class:completed={todo.completed}
        on:click={() => toggleTodo(todo.id)}
      >
        {todo.text}
      </li>
    {/each}
  </ul>
</div>
```

## Экосистема Svelte

- [[SvelteKit]] - Фреймворк для создания полнофункциональных приложений
- [[Svelte Store]] - Управление состоянием
- [[Svelte Material UI]] - Компонентная библиотека
- [[Svelte Testing Library]] - Инструменты тестирования
- [[Svelte Preprocessor]] - Обработка препроцессоров (TypeScript, SCSS и т.д.)

## Российские реалии и особенности

### Применение в российских компаниях

Svelte начинает использоваться в российских компаниях:

- [[Яндекс]] - для экспериментальных проектов
- [[VK]] - в некоторых внутренних инструментах
- [[Сбер] - в стартап-проектах
- Многие стартапы и digital-агентства

### Особенности разработки в России

- **Производительность** - Важно для пользователей с медленным интернетом в регионах
- **Малый размер бандла** - Критично для мобильных пользователей
- **Простота изучения** - Подходит для небольших команд
- **Скорость разработки** - Быстрое создание прототипов

## Лучшие практики

- Использование TypeScript с Svelte
- Правильная организация компонентов
- Тестирование с помощью [[Jest]] и [[Testing Library]]
- Использование SvelteKit для продвинутых приложений
- Оптимизация производительности с помощью компиляции

## Современные тенденции (2025)

- **Svelte 5** - Революционные изменения с runes API
- **SvelteKit** - Развитие как полноценного фреймворка
- **Server-Side Rendering** - Улучшенная поддержка SSR
- **Static Site Generation** - Развитие возможностей генерации статики
- **Svelte DevTools** - Постоянное улучшение инструментов разработки

## Ресурсы для изучения

- [[Официальная документация Svelte]]
- [[Svelte Tutorial]]
- [[Svelte Society]]
- [[Svelte Best Practices]]
- [[Svelte Component Patterns]]

## Заключение

Svelte представляет собой интересный подход к созданию веб-приложений, особенно в 2025 году, когда важна производительность и малый размер бандла. Его простота и эффективность делают его отличным выбором для многих проектов, особенно с учетом российских реалий.

> [!tip] Совет
> Используйте Svelte для проектов, где важна производительность и малый размер бандла, особенно для мобильных пользователей.

> [!warning] Важно
> Svelte имеет меньшее сообщество по сравнению с другими фреймворками, поэтому может быть сложнее найти готовые решения для сложных задач.
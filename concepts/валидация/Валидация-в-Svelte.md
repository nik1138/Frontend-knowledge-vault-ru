---
aliases: ["Валидация форм в Svelte", "Svelte Form Validation"]
tags: ["#svelte", "#validation", "#forms", "#frontend"]
---

# Валидация в Svelte

Валидация в Svelte-приложениях обеспечивает корректность пользовательского ввода и улучшает пользовательский опыт. Благодаря реактивной природе Svelte, валидация может быть реализована элегантно и эффективно как с использованием встроенных возможностей, так и с помощью сторонних библиотек.

## Основные подходы

### Встроенные проверки с реактивностью
```svelte
<script>
  let email = '';
  let password = '';
  let errors = {};

  $: errors = {
    email: !email ? 'Email обязателен' : !/\S+@\S+\.\S+/.test(email) ? 'Некорректный email' : '',
    password: !password ? 'Пароль обязателен' : password.length < 6 ? 'Пароль должен быть не менее 6 символов' : ''
  };

  const isValid = !errors.email && !errors.password && email && password;

  function handleSubmit() {
    if (isValid) {
      console.log('Форма валидна', { email, password });
      // Отправка данных
    }
  }
</script>

<form on:submit|preventDefault={handleSubmit}>
  <div>
    <input
      bind:value={email}
      type="email"
      placeholder="Email"
      class:error={errors.email}
    />
    {#if errors.email}
      <span class="error">{errors.email}</span>
    {/if}
  </div>
  
  <div>
    <input
      bind:value={password}
      type="password"
      placeholder="Password"
      class:error={errors.password}
    />
    {#if errors.password}
      <span class="error">{errors.password}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={!isValid}>Зарегистрироваться</button>
</form>

<style>
  .error {
    border-color: red;
  }
  
  span.error {
    color: red;
    font-size: 0.8em;
  }
</style>
```

## Использование библиотек

### Yup с Svelte
```svelte
<script>
  import { useForm } from 'svelte-forms';
  import * as yup from 'yup';

  const schema = yup.object({
    email: yup.string().email('Некорректный email').required('Email обязателен'),
    password: yup.string().min(6, 'Пароль должен быть не менее 6 символов').required('Пароль обязателен')
  });

  const { form, errors, submitting, submit } = useForm({
    initialValues: {
      email: '',
      password: ''
    },
    validationSchema: schema,
    onSubmit: (values) => {
      console.log(values);
    }
  });
</script>

<form use:submit>
  <div>
    <input
      bind:value={form.email}
      type="email"
      placeholder="Email"
    />
    {#if errors.email}
      <span class="error">{errors.email}</span>
    {/if}
  </div>
  
  <div>
    <input
      bind:value={form.password}
      type="password"
      placeholder="Password"
    />
    {#if errors.password}
      <span class="error">{errors.password}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={submitting}>Войти</button>
</form>
```

### Zod с SvelteKit
```svelte
<script>
  import { superValidate } from 'sveltekit-superforms';
  import { zod } from 'sveltekit-superforms/adapters';
  import { schema } from './schema';

  const form = await superValidate(zod(schema));
</script>

<form method="POST" use:enhance={() => {}}>
  {#if form.errors?.email}
    <p class="error">{form.errors.email}</p>
  {/if}
  
  <input
    name="email"
    placeholder="Email"
    value={form.data.email}
  />
  
  {#if form.errors?.password}
    <p class="error">{form.errors.password}</p>
  {/if}
  
  <input
    type="password"
    name="password"
    placeholder="Password"
    value={form.data.password}
  />
  
  <button type="submit">Войти</button>
</form>
```

## Паттерны валидации

### Валидация при потере фокуса
```svelte
<script>
  let email = '';
  let emailError = '';

  function validateEmail() {
    if (!email) {
      emailError = 'Email обязателен';
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      emailError = 'Некорректный email';
    } else {
      emailError = '';
    }
  }
</script>

<input
  bind:value={email}
  on:blur={validateEmail}
  type="email"
  placeholder="Email"
  class:error={emailError}
/>
{#if emailError}
  <span class="error">{emailError}</span>
{/if}
```

### Валидация в реальном времени
```svelte
<script>
  let password = '';
  let passwordError = '';

  $: {
    if (password.length > 0) {
      if (password.length < 6) {
        passwordError = 'Пароль должен быть не менее 6 символов';
      } else {
        passwordError = '';
      }
    } else {
      passwordError = '';
    }
  }
</script>

<input
  bind:value={password}
  type="password"
  placeholder="Password"
  class:error={passwordError}
/>
{#if passwordError}
  <span class="error">{passwordError}</span>
{/if}
```

## Асинхронная валидация
```svelte>
<script>
  let username = '';
  let usernameError = '';
  let checking = false;

  async function validateUsername() {
    if (username.length < 3) {
      usernameError = 'Имя пользователя должно быть не менее 3 символов';
      return;
    }

    checking = true;
    usernameError = '';

    try {
      const response = await fetch(`/api/check-username/${username}`);
      const result = await response.json();

      if (!result.available) {
        usernameError = 'Имя пользователя уже занято';
      }
    } catch (error) {
      usernameError = 'Ошибка проверки имени пользователя';
    } finally {
      checking = false;
    }
  }
</script>

<input
  bind:value={username}
  on:input={validateUsername}
  type="text"
  placeholder="Username"
  class:error={usernameError}
/>
{#if checking}
  <span class="checking">Проверка...</span>
{/if}
{#if usernameError}
  <span class="error">{usernameError}</span>
{/if}
```

## Лучшие практики

- **Используй реактивность**: Воспользуйся встроенной реактивностью Svelte для автоматической валидации
- **Библиотеки для сложных форм**: Для сложных форм используй специализированные библиотеки
- **Показывай ошибки сразу**: Не жди отправки формы для показа ошибок
- **Удобные сообщения об ошибках**: Делай сообщения понятными и полезными
- **Состояние загрузки**: Показывай индикатор загрузки при асинхронной валидации
- **Тестирование**: Покрой валидацию тестами

## Связанные темы
- [[Валидация-на-сервере]]
- [[Валидация-в-React]]
- [[Валидация-в-Vue]]
- [[Svelte-Form-Management]]
- [[UX-дизайн-форм]]

## Ключевые выводы
- Валидация в Svelte может быть реализована с использованием встроенной реактивности
- Библиотеки упрощают управление сложными формами
- SvelteKit предоставляет специализированные возможности для валидации
- Пользовательский опыт улучшается при своевременном показе ошибок
---
aliases: ["Svelte Forms", "Формы в Svelte", "Form Handling Svelte"]
tags: ["#frontend", "#svelte", "#forms", "#state-management", "#validation"]
---

# Формы в Svelte

Работа с формами в Svelte отличается своей простотой и прямолинейностью. Svelte предоставляет встроенные возможности для управления формами, а также поддерживает сторонние библиотеки для более сложных сценариев. В отличие от React и Vue, Svelte не требует специальных директив для двусторонней привязки данных.

## Введение в формы в Svelte

В Svelte для двусторонней привязки данных используется директива `bind:value` или сокращённая форма `bind:this` для элементов формы. Это позволяет легко синхронизировать состояние компонента с вводом пользователя.

### Простая форма с двусторонней привязкой

```svelte
<script>
  let formData = {
    name: '',
    email: '',
    message: ''
  };

  function handleSubmit(e) {
    e.preventDefault();
    console.log('Отправленные данные:', formData);
  }
</script>

<form on:submit={handleSubmit}>
  <div>
    <label for="name">Имя:</label>
    <input 
      id="name" 
      bind:value={formData.name} 
      type="text" 
      placeholder="Введите ваше имя"
    />
  </div>
  
  <div>
    <label for="email">Email:</label>
    <input 
      id="email" 
      bind:value={formData.email} 
      type="email" 
      placeholder="Введите ваш email"
    />
  </div>
  
  <div>
    <label for="message">Сообщение:</label>
    <textarea 
      id="message" 
      bind:value={formData.message} 
      placeholder="Введите ваше сообщение"
    ></textarea>
  </div>
  
  <button type="submit">Отправить</button>
</form>
```

## Управление состоянием формы

### Локальное состояние
Для простых форм достаточно использовать локальное состояние компонента:

```svelte
<script>
  let username = '';
  let password = '';

  function handleSubmit(e) {
    e.preventDefault();
    console.log({ username, password });
    // Обработка отправки формы
  }
</script>

<form on:submit={handleSubmit}>
  <input bind:value={username} type="text" placeholder="Имя пользователя" />
  <input bind:value={password} type="password" placeholder="Пароль" />
  <button type="submit">Войти</button>
</form>
```

### Использование объектов для управления формой

```svelte
<script>
  let formData = {
    username: '',
    email: ''
  };

  function handleSubmit(e) {
    e.preventDefault();
    console.log('Данные формы:', formData);
  }
</script>

<form on:submit={handleSubmit}>
  <input bind:value={formData.username} type="text" placeholder="Имя пользователя" />
  <input bind:value={formData.email} type="email" placeholder="Email" />
  <button type="submit">Отправить</button>
</form>
```

## Валидация форм

### Простая валидация на клиенте

```svelte
<script>
  let formData = {
    email: '',
    password: ''
  };
  
  let errors = {
    email: null,
    password: null
  };
  
  let isValid = false;

  function validateEmail() {
    if (!formData.email) {
      errors.email = 'Email обязателен';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      errors.email = 'Email некорректен';
    } else {
      errors.email = null;
    }
    updateFormValidity();
  }

  function validatePassword() {
    if (!formData.password) {
      errors.password = 'Пароль обязателен';
    } else if (formData.password.length < 6) {
      errors.password = 'Пароль должен быть не менее 6 символов';
    } else {
      errors.password = null;
    }
    updateFormValidity();
  }

  function updateFormValidity() {
    isValid = formData.email && formData.password && !errors.email && !errors.password;
  }

  function handleSubmit(e) {
    e.preventDefault();
    validateEmail();
    validatePassword();
    
    if (isValid) {
      console.log('Форма валидна, отправляем:', formData);
      // Отправка данных
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div class="form-group">
    <label for="email">Email:</label>
    <input 
      id="email" 
      bind:value={formData.email} 
      type="email" 
      class:invalid={errors.email}
      on:blur={validateEmail}
      placeholder="Email"
    />
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <label for="password">Пароль:</label>
    <input 
      id="password" 
      bind:value={formData.password} 
      type="password" 
      class:invalid={errors.password}
      on:blur={validatePassword}
      placeholder="Пароль"
    />
    {#if errors.password}
      <span class="error-message">{errors.password}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={!isValid}>Войти</button>
</form>

<style>
  .invalid {
    border: 1px solid red;
  }

  .error-message {
    color: red;
    font-size: 0.8em;
    display: block;
    margin-top: 5px;
  }
  
  .form-group {
    margin-bottom: 15px;
  }
  
  button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
</style>
```

### Использование сторонних библиотек валидации

Svelte поддерживает различные библиотеки валидации, такие как:
- Svelte Use - коллекция хуков для Svelte
- Svelte Forms - библиотека для управления формами
- Yup - для валидации схем

Пример с использованием Yup:

```svelte
<script>
  import { onDestroy } from 'svelte';
  import * as yup from 'yup';

  let formData = {
    email: '',
    password: ''
  };
  
  let errors = {};
  let isValid = false;
  let isSubmitting = false;
  
  const schema = yup.object({
    email: yup
      .string()
      .required('Email обязателен')
      .email('Некорректный email'),
    password: yup
      .string()
      .required('Пароль обязателен')
      .min(6, 'Пароль должен быть не менее 6 символов')
  });

  async function validateField(fieldName, value) {
    try {
      await schema.validateAt(fieldName, { [fieldName]: value });
      $set({ errors: { ...errors, [fieldName]: null } });
    } catch (error) {
      $set({ errors: { ...errors, [fieldName]: error.message } });
    }
    
    // Проверяем валидность всей формы
    try {
      await schema.validate(formData);
      isValid = true;
    } catch (error) {
      isValid = false;
    }
  }

  function handleEmailChange(e) {
    formData.email = e.target.value;
    validateField('email', e.target.value);
  }

  function handlePasswordChange(e) {
    formData.password = e.target.value;
    validateField('password', e.target.value);
  }

  async function handleSubmit(e) {
    e.preventDefault();
    isSubmitting = true;
    
    try {
      await schema.validate(formData, { abortEarly: false });
      console.log('Форма валидна, отправляем:', formData);
      // Отправка данных на сервер
    } catch (error) {
      if (error.inner) {
        const newErrors = {};
        error.inner.forEach(err => {
          newErrors[err.path] = err.message;
        });
        errors = newErrors;
      }
    } finally {
      isSubmitting = false;
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div class="form-group">
    <label for="email">Email:</label>
    <input 
      id="email" 
      type="email" 
      value={formData.email}
      on:input={handleEmailChange}
      class:invalid={!!errors.email}
      placeholder="Email"
    />
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <label for="password">Пароль:</label>
    <input 
      id="password" 
      type="password" 
      value={formData.password}
      on:input={handlePasswordChange}
      class:invalid={!!errors.password}
      placeholder="Пароль"
    />
    {#if errors.password}
      <span class="error-message">{errors.password}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={!isValid || isSubmitting}>
    {#if isSubmitting}
      Отправка...
    {:else}
      Войти
    {/if}
  </button>
</form>
```

## Работа с различными типами полей

### Чекбоксы и радиокнопки

```svelte
<script>
  let formData = {
    agreement: false,
    selectedInterests: [],
    gender: ''
  };
  
  const interests = [
    { value: 'sports', label: 'Спорт' },
    { value: 'music', label: 'Музыка' },
    { value: 'travel', label: 'Путешествия' }
  ];
  
  const genders = [
    { value: 'male', label: 'Мужской' },
    { value: 'female', label: 'Женский' },
    { value: 'other', label: 'Другой' }
  ];

  function handleSubmit(e) {
    e.preventDefault();
    console.log('Данные формы:', formData);
  }
</script>

<form on:submit={handleSubmit}>
  <!-- Одиночный чекбокс -->
  <div>
    <label>
      <input 
        type="checkbox" 
        bind:checked={formData.agreement}
      />
      Согласен с условиями
    </label>
  </div>
  
  <!-- Множественные чекбоксы -->
  <div>
    <h3>Интересы:</h3>
    {#each interests as interest}
      <label key={interest.value}>
        <input
          type="checkbox"
          bind:group={formData.selectedInterests}
          value={interest.value}
        />
        {interest.label}
      </label>
    {/each}
  </div>
  
  <!-- Радиокнопки -->
  <div>
    <h3>Пол:</h3>
    {#each genders as gender}
      <label key={gender.value}>
        <input
          type="radio"
          bind:group={formData.gender}
          name="gender"
          value={gender.value}
        />
        {gender.label}
      </label>
    {/each}
  </div>
  
  <button type="submit">Отправить</button>
</form>
```

### Выпадающие списки

```svelte
<script>
  let formData = {
    country: '',
    city: ''
  };
  
  const countries = [
    { value: 'ru', label: 'Россия' },
    { value: 'us', label: 'США' },
    { value: 'de', label: 'Германия' }
  ];
  
  const cities = {
    ru: [
      { value: 'moscow', label: 'Москва' },
      { value: 'spb', label: 'Санкт-Петербург' }
    ],
    us: [
      { value: 'nyc', label: 'Нью-Йорк' },
      { value: 'la', label: 'Лос-Анджелес' }
    ],
    de: [
      { value: 'berlin', label: 'Берлин' },
      { value: 'munich', label: 'Мюнхен' }
    ]
  };
  
  function getCitiesByCountry(country) {
    return cities[country] || [];
  }

  function handleSubmit(e) {
    e.preventDefault();
    console.log('Данные формы:', formData);
  }
</script>

<form on:submit={handleSubmit}>
  <div>
    <label for="country">Страна:</label>
    <select id="country" bind:value={formData.country}>
      <option value="">Выберите страну</option>
      {#each countries as country}
        <option value={country.value}>
          {country.label}
        </option>
      {/each}
    </select>
  </div>
  
  <div>
    <label for="city">Город:</label>
    <select 
      id="city" 
      bind:value={formData.city} 
      disabled={!formData.country}
    >
      <option value="">Выберите город</option>
      {#each getCitiesByCountry(formData.country) as city}
        <option value={city.value}>
          {city.label}
        </option>
      {/each}
    </select>
  </div>
  
  <button type="submit">Отправить</button>
</form>
```

## Практические примеры

### Форма регистрации с валидацией

```svelte
<script>
  import { onMount } from 'svelte';

  let formData = {
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    agreement: false
  };
  
  let errors = {
    username: null,
    email: null,
    password: null,
    confirmPassword: null,
    agreement: null
  };
  
  let isSubmitting = false;
  let isValid = false;

  function validateUsername() {
    if (!formData.username) {
      errors.username = 'Имя пользователя обязательно';
    } else if (formData.username.length < 3) {
      errors.username = 'Имя пользователя должно быть не менее 3 символов';
    } else {
      errors.username = null;
    }
  }

  function validateEmail() {
    if (!formData.email) {
      errors.email = 'Email обязателен';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      errors.email = 'Email некорректен';
    } else {
      errors.email = null;
    }
  }

  function validatePassword() {
    if (!formData.password) {
      errors.password = 'Пароль обязателен';
    } else if (formData.password.length < 6) {
      errors.password = 'Пароль должен быть не менее 6 символов';
    } else {
      errors.password = null;
    }
  }

  function validateConfirmPassword() {
    if (formData.password !== formData.confirmPassword) {
      errors.confirmPassword = 'Пароли не совпадают';
    } else {
      errors.confirmPassword = null;
    }
  }

  function validateAgreement() {
    if (!formData.agreement) {
      errors.agreement = 'Необходимо согласие с условиями';
    } else {
      errors.agreement = null;
    }
  }

  function validateForm() {
    validateUsername();
    validateEmail();
    validatePassword();
    validateConfirmPassword();
    validateAgreement();
    
    isValid = !Object.values(errors).some(error => error !== null) && 
              formData.username && 
              formData.email && 
              formData.password && 
              formData.confirmPassword && 
              formData.agreement;
    
    return isValid;
  }

  async function register(e) {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    isSubmitting = true;
    
    try {
      // Имитация API вызова
      await new Promise(resolve => setTimeout(resolve, 1000));
      console.log('Регистрация успешна:', formData);
      
      // Сброс формы после успешной регистрации
      formData = {
        username: '',
        email: '',
        password: '',
        confirmPassword: '',
        agreement: false
      };
      errors = {};
    } catch (error) {
      console.error('Ошибка регистрации:', error);
    } finally {
      isSubmitting = false;
    }
  }

  // Валидация в реальном времени
  $: validateUsername();
  $: validateEmail();
  $: validatePassword();
  $: validateConfirmPassword();
  $: validateAgreement();
  $: validateForm();
</script>

<div class="registration-form">
  <h2>Регистрация</h2>
  <form on:submit={register}>
    <div class="form-group">
      <label for="username">Имя пользователя:</label>
      <input
        id="username"
        bind:value={formData.username}
        type="text"
        class:invalid={!!errors.username}
        placeholder="Введите имя пользователя"
      />
      {#if errors.username}
        <span class="error-message">{errors.username}</span>
      {/if}
    </div>
    
    <div class="form-group">
      <label for="email">Email:</label>
      <input
        id="email"
        bind:value={formData.email}
        type="email"
        class:invalid={!!errors.email}
        placeholder="Введите email"
      />
      {#if errors.email}
        <span class="error-message">{errors.email}</span>
      {/if}
    </div>
    
    <div class="form-group">
      <label for="password">Пароль:</label>
      <input
        id="password"
        bind:value={formData.password}
        type="password"
        class:invalid={!!errors.password}
        placeholder="Введите пароль"
      />
      {#if errors.password}
        <span class="error-message">{errors.password}</span>
      {/if}
    </div>
    
    <div class="form-group">
      <label for="confirmPassword">Подтверждение пароля:</label>
      <input
        id="confirmPassword"
        bind:value={formData.confirmPassword}
        type="password"
        class:invalid={!!errors.confirmPassword}
        placeholder="Подтвердите пароль"
      />
      {#if errors.confirmPassword}
        <span class="error-message">{errors.confirmPassword}</span>
      {/if}
    </div>
    
    <div class="form-group">
      <label>
        <input
          bind:checked={formData.agreement}
          type="checkbox"
        />
        Я согласен с условиями использования
      </label>
      {#if errors.agreement}
        <span class="error-message">{errors.agreement}</span>
      {/if}
    </div>
    
    <button type="submit" disabled={!isValid || isSubmitting}>
      {#if isSubmitting}
        Регистрация...
      {:else}
        Зарегистрироваться
      {/if}
    </button>
  </form>
</div>

<style>
  .registration-form {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
  }

  .form-group {
    margin-bottom: 15px;
  }

  .form-group label {
    display: block;
    margin-bottom: 5px;
  }

  .form-group input,
  .form-group select,
  .form-group textarea {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
  }

  .form-group input.invalid {
    border-color: red;
  }

  .error-message {
    color: red;
    font-size: 0.8em;
    margin-top: 5px;
    display: block;
  }

  button {
    width: 100%;
    padding: 10px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  button:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
</style>
```

## Использование Svelte Store для управления формой

Для сложных форм или форм, которые используются в нескольких компонентах, можно использовать Svelte Store:

```svelte
<!-- stores/form.js -->
<script context="module">
  import { writable } from 'svelte/store';
  
  function createFormStore(initialData = {}) {
    const { subscribe, set, update } = writable(initialData);
    
    return {
      subscribe,
      set,
      update,
      reset: () => set(initialData),
      setValue: (field, value) => update(data => ({ ...data, [field]: value }))
    };
  }
  
  export const registrationForm = createFormStore({
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  });
  
  export const formErrors = writable({
    username: null,
    email: null,
    password: null,
    confirmPassword: null
  });
</script>

<!-- RegistrationForm.svelte -->
<script>
  import { registrationForm, formErrors } from './stores/form.js';
  
  let errors = $formErrors;
  
  function validateAndSubmit(e) {
    e.preventDefault();
    
    // Валидация
    const currentForm = $registrationForm;
    const newErrors = {};
    
    if (!currentForm.username) newErrors.username = 'Имя пользователя обязательно';
    if (!currentForm.email) newErrors.email = 'Email обязателен';
    if (!currentForm.password) newErrors.password = 'Пароль обязателен';
    if (currentForm.password !== currentForm.confirmPassword) newErrors.confirmPassword = 'Пароли не совпадают';
    
    formErrors.set(newErrors);
    
    if (Object.keys(newErrors).length === 0) {
      console.log('Форма валидна, отправляем:', currentForm);
      // Отправка данных
    }
  }
</script>

<form on:submit={validateAndSubmit}>
  <div class="form-group">
    <label for="username">Имя пользователя:</label>
    <input
      id="username"
      type="text"
      value={$registrationForm.username}
      on:input={(e) => registrationForm.setValue('username', e.target.value)}
    />
    {#if errors.username}
      <span class="error-message">{errors.username}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <label for="email">Email:</label>
    <input
      id="email"
      type="email"
      value={$registrationForm.email}
      on:input={(e) => registrationForm.setValue('email', e.target.value)}
    />
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
  </div>
  
  <!-- Остальные поля формы -->
  
  <button type="submit">Отправить</button>
</form>
```

## Лучшие практики

1. **Используйте `bind:value`** для двусторонней привязки данных
2. **Предотвращайте стандартное поведение формы** с помощью `on:submit|preventDefault`
3. **Валидируйте данные на клиенте** для улучшения UX
4. **Очищайте ошибки при изменении полей** для улучшения UX
5. **Используйте TypeScript** для строгой типизации форм (при использовании Svelte с TS)
6. **Разделяйте сложные формы** на несколько компонентов
7. **Обеспечивайте доступность** с помощью правильных меток и ARIA-атрибутов
8. **Показывайте состояние загрузки** при отправке формы
9. **Обрабатывайте ошибки сервера** и отображайте их пользователю
10. **Используйте reactive declarations** для автоматической валидации

## Заключение

Формы в Svelte отличаются своей простотой и прямолинейностью. Встроенная система двусторонней привязки данных делает работу с формами интуитивно понятной. Использование `bind:value`, реактивных объявлений и сторов позволяет создавать надежные и удобные формы. Правильная реализация форм улучшает пользовательский опыт и предотвращает ошибки ввода данных.

См. также: [[Управление-состоянием-форм]], [[Формы-в-React]], [[Формы-в-Vue]]
---
aliases: ["Svelte Form Validation", "Валидация форм в Svelte", "Svelte Validation"]
tags: ["#svelte", "#validation", "#frontend", "#forms"]
---

# Валидация в Svelte

**Валидация в Svelte** - это процесс проверки данных форм в приложениях, построенных с использованием фреймворка Svelte. Svelte предлагает уникальный подход к управлению состоянием и валидации, используя реактивность на основе присвоения переменных.

## Общие сведения

Svelte отличается от других фреймворков тем, что:
- Не использует виртуальный DOM
- Компилируется в эффективный vanilla JavaScript
- Предоставляет встроенную реактивность через присвоение переменных
- Имеет минимальный размер бандла

Эти особенности влияют на подход к валидации форм в Svelte.

## Встроенные возможности Svelte

### Простая валидация с реактивностью

```svelte
<script>
  let email = '';
  let password = '';
  let errors = {};
  let isSubmitting = false;
  
  // Реактивное вычисление валидности формы
  $: isFormValid = !errors.email && !errors.password && email && password;
  
  // Функция валидации поля
  function validateField(name, value) {
    switch (name) {
      case 'email':
        if (!value) return 'Email обязателен';
        if (!/^\S+@\S+\.\S+$/.test(value)) return 'Email некорректен';
        break;
      case 'password':
        if (!value) return 'Пароль обязателен';
        if (value.length < 8) return 'Пароль должен быть не менее 8 символов';
        if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          return 'Пароль должен содержать заглавную, строчную буквы и цифру';
        }
        break;
      default:
        return '';
    }
    return '';
  }
  
  // Обработка изменения поля
  function handleInput(event, field) {
    const value = event.target.value;
    if (field === 'email') email = value;
    if (field === 'password') password = value;
    
    // Валидация при вводе
    const error = validateField(field, value);
    errors = { ...errors, [field]: error };
  }
  
  // Обработка отправки формы
  async function handleSubmit(event) {
    event.preventDefault();
    
    // Валидация всех полей
    const newErrors = {};
    newErrors.email = validateField('email', email);
    newErrors.password = validateField('password', password);
    
    errors = newErrors;
    
    if (Object.values(newErrors).every(error => !error)) {
      isSubmitting = true;
      
      try {
        console.log('Данные формы:', { email, password });
        // await api.submitForm({ email, password });
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        isSubmitting = false;
      }
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div class="form-group">
    <input
      type="email"
      placeholder="Email"
      value={email}
      on:input={(e) => handleInput(e, 'email')}
      class:error={errors.email}
    />
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <input
      type="password"
      placeholder="Password"
      value={password}
      on:input={(e) => handleInput(e, 'password')}
      class:error={errors.password}
    />
    {#if errors.password}
      <span class="error-message">{errors.password}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={isSubmitting || !isFormValid}>
    {isSubmitting ? 'Отправка...' : 'Отправить'}
  </button>
</form>

<style>
  .form-group {
    margin-bottom: 1rem;
  }
  
  input {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  input.error {
    border-color: #ff4757;
  }
  
  .error-message {
    color: #ff4757;
    font-size: 0.8rem;
    margin-top: 0.25rem;
    display: block;
  }
</style>
```

### Использование модификаторов и реактивных вычислений

```svelte
<script>
  import { onMount } from 'svelte';
  
  // Состояние формы
  let formData = {
    email: '',
    password: '',
    confirmPassword: ''
  };
  
  // Ошибки валидации
  let errors = {};
  
  // Состояние отправки
  let isSubmitting = false;
  
  // Реактивные вычисления
  $: isEmailValid = !errors.email && formData.email.length > 0;
  $: isPasswordValid = !errors.password && formData.password.length >= 8;
  $: passwordsMatch = formData.password === formData.confirmPassword;
  $: isFormValid = isEmailValid && isPasswordValid && passwordsMatch && !Object.keys(errors).length;
  
  // Функция валидации
  function validateField(name, value) {
    switch (name) {
      case 'email':
        if (!value) return 'Email обязателен';
        if (!/^\S+@\S+\.\S+$/.test(value)) return 'Email некорректен';
        break;
      case 'password':
        if (!value) return 'Пароль обязателен';
        if (value.length < 8) return 'Пароль должен быть не менее 8 символов';
        if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          return 'Пароль должен содержать заглавную, строчную буквы и цифру';
        }
        break;
      case 'confirmPassword':
        if (!value) return 'Подтверждение пароля обязательно';
        if (value !== formData.password) return 'Пароли не совпадают';
        break;
      default:
        return '';
    }
    return '';
  }
  
  // Обработчик изменения поля
  function handleFieldChange(field, value) {
    formData = { ...formData, [field]: value };
    
    // Валидация при изменении
    const error = validateField(field, value);
    errors = { ...errors, [field]: error };
  }
  
  // Обработчик отправки формы
  async function handleSubmit(event) {
    event.preventDefault();
    
    // Полная валидация перед отправкой
    const newErrors = {};
    for (const [field, value] of Object.entries(formData)) {
      const error = validateField(field, value);
      if (error) newErrors[field] = error;
    }
    
    errors = newErrors;
    
    if (Object.keys(newErrors).length === 0) {
      isSubmitting = true;
      
      try {
        console.log('Данные формы:', formData);
        // await api.submitForm(formData);
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        isSubmitting = false;
      }
    }
  }
  
  // Живая валидация (валидация при вводе)
  $: if (formData.confirmPassword) {
    const confirmError = validateField('confirmPassword', formData.confirmPassword);
    if (confirmError) {
      errors = { ...errors, confirmPassword: confirmError };
    } else if (errors.confirmPassword) {
      const { confirmPassword, ...rest } = errors;
      errors = rest;
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div class="form-group">
    <input
      type="email"
      placeholder="Email"
      value={formData.email}
      on:input={(e) => handleFieldChange('email', e.target.value)}
      class:error={errors.email}
    />
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
    <div class="validation-indicator" class:valid={isEmailValid} class:invalid={!isEmailValid && formData.email}>
      {#if isEmailValid}
        ✅
      {:else if formData.email}
        ❌
      {/if}
    </div>
  </div>
  
  <div class="form-group">
    <input
      type="password"
      placeholder="Password"
      value={formData.password}
      on:input={(e) => handleFieldChange('password', e.target.value)}
      class:error={errors.password}
    />
    {#if errors.password}
      <span class="error-message">{errors.password}</span>
    {/if}
    <div class="validation-indicator" class:valid={isPasswordValid} class:invalid={!isPasswordValid && formData.password}>
      {#if isPasswordValid}
        ✅
      {:else if formData.password}
        ❌
      {/if}
    </div>
  </div>
  
  <div class="form-group">
    <input
      type="password"
      placeholder="Confirm Password"
      value={formData.confirmPassword}
      on:input={(e) => handleFieldChange('confirmPassword', e.target.value)}
      class:error={errors.confirmPassword}
    />
    {#if errors.confirmPassword}
      <span class="error-message">{errors.confirmPassword}</span>
    {/if}
    <div class="validation-indicator" class:valid={passwordsMatch && formData.confirmPassword} class:invalid={!passwordsMatch && formData.confirmPassword}>
      {#if passwordsMatch && formData.confirmPassword}
        ✅
      {:else if formData.confirmPassword}
        ❌
      {/if}
    </div>
  </div>
  
  <button type="submit" disabled={isSubmitting || !isFormValid}>
    {isSubmitting ? 'Отправка...' : 'Отправить'} (Валидно: {isFormValid ? 'Да' : 'Нет'})
  </button>
</form>

<style>
  .form-group {
    position: relative;
    margin-bottom: 1rem;
  }
  
  input {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  input.error {
    border-color: #ff4757;
  }
  
  .error-message {
    color: #ff4757;
    font-size: 0.8rem;
    margin-top: 0.25rem;
    display: block;
  }
  
  .validation-indicator {
    position: absolute;
    right: 10px;
    top: 50%;
    transform: translateY(-50%);
    font-size: 1.2rem;
  }
  
  .validation-indicator.valid {
    color: #2ed573;
  }
  
  .validation-indicator.invalid {
    color: #ff4757;
  }
</style>
```

## Валидация с использованием внешних библиотек

### Yup с Svelte

```svelte
<script>
  import * as yup from 'yup';
  
  // Состояние формы
  let formData = {
    email: '',
    password: '',
    age: null
  };
  
  let errors = {};
  let isSubmitting = false;
  
  // Схема валидации
  const schema = yup.object().shape({
    email: yup
      .string()
      .email('Некорректный email')
      .required('Email обязателен'),
    password: yup
      .string()
      .min(8, 'Пароль должен быть не менее 8 символов')
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, 
               'Пароль должен содержать заглавную, строчную буквы и цифру')
      .required('Пароль обязателен'),
    age: yup
      .number()
      .positive('Возраст должен быть положительным')
      .integer('Возраст должен быть целым числом')
      .min(18, 'Возраст должен быть не менее 18')
      .max(120, 'Возраст должен быть не более 120')
      .required('Возраст обязателен')
  });
  
  // Функция валидации поля
  async function validateField(fieldName, value) {
    try {
      await schema.validateAt(fieldName, { [fieldName]: value });
      // Если валидация прошла успешно, удаляем ошибку
      const { [fieldName]: removed, ...remainingErrors } = errors;
      errors = remainingErrors;
      return '';
    } catch (error) {
      errors = { ...errors, [fieldName]: error.message };
      return error.message;
    }
  }
  
  // Валидация всей формы
  async function validateForm() {
    try {
      await schema.validate(formData, { abortEarly: false });
      errors = {};
      return true;
    } catch (error) {
      const newErrors = {};
      if (error.inner) {
        error.inner.forEach(err => {
          newErrors[err.path] = err.message;
        });
      }
      errors = newErrors;
      return false;
    }
  }
  
  // Обработчик изменения поля
  async function handleFieldChange(field, value) {
    // Обновляем значение поля
    formData = { ...formData, [field]: value };
    
    // Валидируем поле
    await validateField(field, value);
  }
  
  // Обработчик отправки формы
  async function handleSubmit(event) {
    event.preventDefault();
    
    const isValid = await validateForm();
    
    if (isValid) {
      isSubmitting = true;
      
      try {
        console.log('Данные формы:', formData);
        // await api.submitForm(formData);
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        isSubmitting = false;
      }
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div class="form-group">
    <input
      type="email"
      placeholder="Email"
      value={formData.email}
      on:input={(e) => handleFieldChange('email', e.target.value)}
      class:error={errors.email}
    />
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <input
      type="password"
      placeholder="Password"
      value={formData.password}
      on:input={(e) => handleFieldChange('password', e.target.value)}
      class:error={errors.password}
    />
    {#if errors.password}
      <span class="error-message">{errors.password}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <input
      type="number"
      placeholder="Age"
      value={formData.age}
      on:input={(e) => handleFieldChange('age', Number(e.target.value))}
      class:error={errors.age}
    />
    {#if errors.age}
      <span class="error-message">{errors.age}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={isSubmitting}>
    {isSubmitting ? 'Отправка...' : 'Отправить'}
  </button>
</form>
```

### Кастомный хук валидации для Svelte

```svelte
<!-- validation.svelte -->
<script>
  import { createEventDispatcher } from 'svelte';
  
  export let schema;
  export let initialValues = {};
  
  const dispatch = createEventDispatcher();
  
  let values = { ...initialValues };
  let errors = {};
  let isSubmitting = false;
  let isValidating = false;
  
  // Реактивное вычисление валидности формы
  $: formIsValid = Object.keys(errors).length === 0;
  
  // Функция валидации
  async function validate(valuesToValidate = values) {
    if (!schema) return {};
    
    isValidating = true;
    
    try {
      await schema.validate(valuesToValidate, { abortEarly: false });
      isValidating = false;
      return {};
    } catch (validationError) {
      isValidating = false;
      
      const newErrors = {};
      if (validationError.inner) {
        validationError.inner.forEach(error => {
          newErrors[error.path] = error.message;
        });
      }
      
      return newErrors;
    }
  }
  
  // Валидация конкретного поля
  async function validateField(fieldName, value) {
    if (!schema) return '';
    
    try {
      await schema.validateAt(fieldName, { [fieldName]: value });
      return '';
    } catch (error) {
      return error.message;
    }
  }
  
  // Обработчик изменения значения
  export async function handleChange(field, value) {
    values = { ...values, [field]: value };
    
    // Валидация поля
    const fieldError = await validateField(field, value);
    
    if (fieldError) {
      errors = { ...errors, [field]: fieldError };
    } else {
      const { [field]: removed, ...remainingErrors } = errors;
      errors = remainingErrors;
    }
    
    dispatch('change', { field, value, errors });
  }
  
  // Обработчик отправки формы
  export async function handleSubmit(submitHandler) {
    const newErrors = await validate();
    errors = newErrors;
    
    if (Object.keys(newErrors).length === 0) {
      isSubmitting = true;
      
      try {
        await submitHandler(values);
        dispatch('success', values);
      } catch (error) {
        dispatch('error', error);
      } finally {
        isSubmitting = false;
      }
    } else {
      dispatch('invalid', newErrors);
    }
  }
  
  // Метод для получения значения поля
  export function getValue(field) {
    return values[field];
  }
  
  // Метод для получения ошибки поля
  export function getError(field) {
    return errors[field];
  }
  
  // Метод для установки ошибки поля
  export function setError(field, error) {
    errors = { ...errors, [field]: error };
  }
  
  // Метод для очистки ошибки поля
  export function clearError(field) {
    const { [field]: removed, ...remainingErrors } = errors;
    errors = remainingErrors;
  }
  
  // Экспортируемые значения
  export { values, errors, isSubmitting, isValidating, formIsValid };
</script>

<!-- Использование в компоненте -->
<script>
  import ValidationForm from './validation.svelte';
  import * as yup from 'yup';
  
  const schema = yup.object().shape({
    email: yup
      .string()
      .email('Некорректный email')
      .required('Email обязателен'),
    password: yup
      .string()
      .min(8, 'Пароль должен быть не менее 8 символов')
      .required('Пароль обязателен')
  });
  
  let initialValues = {
    email: '',
    password: ''
  };
  
  // Функция отправки формы
  async function onSubmit(formData) {
    console.log('Данные формы:', formData);
    // await api.submitForm(formData);
  }
  
  // Обработчик событий валидации
  function handleValidationEvent(event) {
    console.log('Событие валидации:', event);
  }
</script>

<ValidationForm
  {schema}
  {initialValues}
  on:change={handleValidationEvent}
  on:success={handleValidationEvent}
  on:error={handleValidationEvent}
  let:values
  let:errors
  let:isSubmitting
  let:formIsValid
  let:handleChange
  let:handleSubmit
>
  <form on:submit={(e) => {
    e.preventDefault();
    handleSubmit(onSubmit);
  }}>
    <div class="form-group">
      <input
        type="email"
        placeholder="Email"
        value={values.email}
        on:input={(e) => handleChange('email', e.target.value)}
        class:error={errors.email}
      />
      {#if errors.email}
        <span class="error-message">{errors.email}</span>
      {/if}
    </div>
    
    <div class="form-group">
      <input
        type="password"
        placeholder="Password"
        value={values.password}
        on:input={(e) => handleChange('password', e.target.value)}
        class:error={errors.password}
      />
      {#if errors.password}
        <span class="error-message">{errors.password}</span>
      {/if}
    </div>
    
    <button type="submit" disabled={isSubmitting || !formIsValid}>
      {isSubmitting ? 'Отправка...' : 'Отправить'}
    </button>
  </form>
</ValidationForm>
```

## Условная валидация

```svelte
<script>
  import * as yup from 'yup';
  
  let formData = {
    accountType: '',
    email: '',
    password: '',
    companyName: '',
    taxId: ''
  };
  
  let errors = {};
  let isSubmitting = false;
  
  // Реактивное вычисление показа бизнес-полей
  $: showBusinessFields = formData.accountType === 'business';
  
  // Схема валидации
  const getSchema = () => yup.object().shape({
    accountType: yup.string().required('Тип аккаунта обязателен'),
    email: yup
      .string()
      .email('Некорректный email')
      .required('Email обязателен'),
    password: yup
      .string()
      .min(8, 'Пароль должен быть не менее 8 символов')
      .required('Пароль обязателен'),
    ...(showBusinessFields && {
      companyName: yup.string().required('Название компании обязательно'),
      taxId: yup
        .string()
        .matches(/^\d{10}$/, 'ИНН должен содержать 10 цифр')
        .required('ИНН обязателен')
    })
  });
  
  // Функция валидации
  async function validateForm() {
    try {
      const schema = getSchema();
      await schema.validate(formData, { abortEarly: false });
      errors = {};
      return true;
    } catch (validationError) {
      const newErrors = {};
      if (validationError.inner) {
        validationError.inner.forEach(error => {
          newErrors[error.path] = error.message;
        });
      }
      errors = newErrors;
      return false;
    }
  }
  
  // Обработчик изменения поля
  function handleFieldChange(field, value) {
    formData = { ...formData, [field]: value };
    
    // Если изменили тип аккаунта, нужно пересчитать валидацию
    if (field === 'accountType') {
      // Очищаем ошибки для условных полей
      if (!showBusinessFields) {
        const { companyName, taxId, ...rest } = errors;
        errors = rest;
      }
    }
  }
  
  // Обработчик отправки формы
  async function handleSubmit(event) {
    event.preventDefault();
    
    const isValid = await validateForm();
    
    if (isValid) {
      isSubmitting = true;
      
      try {
        console.log('Данные формы:', formData);
        // await api.submitForm(formData);
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        isSubmitting = false;
      }
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div class="form-group">
    <select
      value={formData.accountType}
      on:change={(e) => handleFieldChange('accountType', e.target.value)}
      class:error={errors.accountType}
    >
      <option value="">Выберите тип аккаунта</option>
      <option value="personal">Личный</option>
      <option value="business">Бизнес</option>
    </select>
    {#if errors.accountType}
      <span class="error-message">{errors.accountType}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <input
      type="email"
      placeholder="Email"
      value={formData.email}
      on:input={(e) => handleFieldChange('email', e.target.value)}
      class:error={errors.email}
    />
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <input
      type="password"
      placeholder="Password"
      value={formData.password}
      on:input={(e) => handleFieldChange('password', e.target.value)}
      class:error={errors.password}
    />
    {#if errors.password}
      <span class="error-message">{errors.password}</span>
    {/if}
  </div>
  
  {#if showBusinessFields}
    <div class="form-group">
      <input
        type="text"
        placeholder="Название компании"
        value={formData.companyName}
        on:input={(e) => handleFieldChange('companyName', e.target.value)}
        class:error={errors.companyName}
      />
      {#if errors.companyName}
        <span class="error-message">{errors.companyName}</span>
      {/if}
    </div>
    
    <div class="form-group">
      <input
        type="text"
        placeholder="ИНН"
        value={formData.taxId}
        on:input={(e) => handleFieldChange('taxId', e.target.value)}
        class:error={errors.taxId}
      />
      {#if errors.taxId}
        <span class="error-message">{errors.taxId}</span>
      {/if}
    </div>
  {/if}
  
  <button type="submit" disabled={isSubmitting}>
    {isSubmitting ? 'Отправка...' : 'Отправить'}
  </button>
</form>
```

## Асинхронная валидация

```svelte
<script>
  import * as yup from 'yup';
  
  let formData = {
    email: '',
    username: '',
    password: ''
  };
  
  let errors = {};
  let isSubmitting = false;
  
  // Асинхронная проверка уникальности email
  async function checkEmailUniqueness(email) {
    if (!email) return true;
    
    // Имитация API-запроса
    return new Promise(resolve => {
      setTimeout(() => {
        const isUnique = email !== 'existing@example.com';
        resolve(isUnique);
      }, 1000);
    });
  }
  
  // Асинхронная проверка доступности username
  async function checkUsernameAvailability(username) {
    if (!username) return true;
    
    // Имитация API-запроса
    return new Promise(resolve => {
      setTimeout(() => {
        const isAvailable = username.length >= 3 && !['admin', 'root', 'user'].includes(username);
        resolve(isAvailable);
      }, 800);
    });
  }
  
  // Схема валидации с асинхронными правилами
  const schema = yup.object().shape({
    email: yup
      .string()
      .email('Некорректный email')
      .required('Email обязателен')
      .test('unique-email', 'Email уже используется', async (value) => {
        if (!value) return true;
        return await checkEmailUniqueness(value);
      }),
    username: yup
      .string()
      .min(3, 'Имя пользователя должно быть не менее 3 символов')
      .required('Имя пользователя обязательно')
      .test('available-username', 'Имя пользователя недоступно', async (value) => {
        if (!value) return true;
        return await checkUsernameAvailability(value);
      }),
    password: yup
      .string()
      .min(8, 'Пароль должен быть не менее 8 символов')
      .required('Пароль обязателен')
  });
  
  // Состояния загрузки для асинхронной валидации
  let emailValidationLoading = false;
  let usernameValidationLoading = false;
  
  // Функция валидации поля с асинхронной проверкой
  async function validateField(fieldName, value) {
    if (fieldName === 'email') {
      emailValidationLoading = true;
    } else if (fieldName === 'username') {
      usernameValidationLoading = true;
    }
    
    try {
      // Валидация схемы для конкретного поля
      await schema.validateAt(fieldName, { [fieldName]: value });
      
      // Удаляем ошибку поля
      const { [fieldName]: removed, ...remainingErrors } = errors;
      errors = remainingErrors;
    } catch (error) {
      errors = { ...errors, [fieldName]: error.message };
    } finally {
      if (fieldName === 'email') {
        emailValidationLoading = false;
      } else if (fieldName === 'username') {
        usernameValidationLoading = false;
      }
    }
  }
  
  // Обработчик изменения поля
  async function handleFieldChange(field, value) {
    formData = { ...formData, [field]: value };
    
    // Для email и username запускаем асинхронную валидацию
    if (['email', 'username'].includes(field)) {
      await validateField(field, value);
    } else {
      // Для остальных полей - синхронная валидация
      try {
        await schema.validateAt(field, { [field]: value });
        const { [field]: removed, ...remainingErrors } = errors;
        errors = remainingErrors;
      } catch (error) {
        errors = { ...errors, [field]: error.message };
      }
    }
  }
  
  // Валидация всей формы
  async function validateForm() {
    try {
      await schema.validate(formData, { abortEarly: false });
      errors = {};
      return true;
    } catch (validationError) {
      const newErrors = {};
      if (validationError.inner) {
        validationError.inner.forEach(error => {
          newErrors[error.path] = error.message;
        });
      }
      errors = newErrors;
      return false;
    }
  }
  
  // Обработчик отправки формы
  async function handleSubmit(event) {
    event.preventDefault();
    
    // Проверяем, идет ли еще асинхронная валидация
    if (emailValidationLoading || usernameValidationLoading) {
      // Ждем завершения асинхронной валидации
      await new Promise(resolve => setTimeout(resolve, 300));
    }
    
    const isValid = await validateForm();
    
    if (isValid) {
      isSubmitting = true;
      
      try {
        console.log('Данные формы:', formData);
        // await api.submitForm(formData);
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        isSubmitting = false;
      }
    }
  }
</script>

<form on:submit={handleSubmit}>
  <div class="form-group">
    <input
      type="email"
      placeholder="Email"
      value={formData.email}
      on:input={(e) => handleFieldChange('email', e.target.value)}
      class:error={errors.email}
      disabled={emailValidationLoading}
    />
    {#if emailValidationLoading}
      <span class="loading-indicator">Проверка...</span>
    {/if}
    {#if errors.email}
      <span class="error-message">{errors.email}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <input
      type="text"
      placeholder="Username"
      value={formData.username}
      on:input={(e) => handleFieldChange('username', e.target.value)}
      class:error={errors.username}
      disabled={usernameValidationLoading}
    />
    {#if usernameValidationLoading}
      <span class="loading-indicator">Проверка...</span>
    {/if}
    {#if errors.username}
      <span class="error-message">{errors.username}</span>
    {/if}
  </div>
  
  <div class="form-group">
    <input
      type="password"
      placeholder="Password"
      value={formData.password}
      on:input={(e) => handleFieldChange('password', e.target.value)}
      class:error={errors.password}
    />
    {#if errors.password}
      <span class="error-message">{errors.password}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={isSubmitting || emailValidationLoading || usernameValidationLoading}>
    {isSubmitting ? 'Отправка...' : 'Отправить'}
  </button>
</form>
```

## Лучшие практики валидации в Svelte

- **Используйте реактивность Svelte** для автоматического обновления UI при изменении состояния валидации
- **Разделяйте синхронную и асинхронную валидацию** для лучшей производительности
- **Предоставляйте мгновенную обратную связь** при изменении полей
- **Используйте внешние библиотеки** для сложных схем валидации (Yup, Joi и т.д.)
- **Обеспечьте доступность** с помощью ARIA-атрибутов и семантической разметки
- **Валидируйте на сервере** - клиентская валидация не заменяет серверную
- **Оптимизируйте асинхронные проверки** с помощью debounce для избежания частых запросов

## Связанные темы

- [[Валидация-на-клиенте]]
- [[Валидация-на-сервере]]
- [[Валидация-в-React]]
- [[Валидация-в-Vue]]
- [[Svelte-Form-Management]]
- [[UX-валидация-форм]]

## Теги

#svelte #validation #frontend #forms #javascript #web-development #reactivity #yup
---
aliases: ["Валидация форм в Vue", "Vue Form Validation"]
tags: ["#vue", "#validation", "#forms", "#frontend"]
---

# Валидация в Vue

Валидация в Vue.js приложениях обеспечивает корректность пользовательского ввода и улучшает взаимодействие с пользователем. Vue предоставляет гибкие возможности для реализации валидации как с использованием встроенных возможностей, так и с помощью сторонних библиотек.

## Основные подходы

### Встроенные проверки с Composition API
```vue
<template>
  <form @submit.prevent="submitForm">
    <div>
      <input
        v-model="email"
        type="email"
        placeholder="Email"
        :class="{ error: errors.email }"
      />
      <span v-if="errors.email" class="error">{{ errors.email }}</span>
    </div>
    
    <div>
      <input
        v-model="password"
        type="password"
        placeholder="Password"
        :class="{ error: errors.password }"
      />
      <span v-if="errors.password" class="error">{{ errors.password }}</span>
    </div>
    
    <button type="submit" :disabled="isLoading">Зарегистрироваться</button>
  </form>
</template>

<script setup>
import { ref, reactive } from 'vue';

const email = ref('');
const password = ref('');
const isLoading = ref(false);
const errors = reactive({});

const validate = () => {
  const newErrors = {};
  
  if (!email.value) {
    newErrors.email = 'Email обязателен';
  } else if (!/\S+@\S+\.\S+/.test(email.value)) {
    newErrors.email = 'Email некорректен';
  }
  
  if (!password.value) {
    newErrors.password = 'Пароль обязателен';
  } else if (password.value.length < 6) {
    newErrors.password = 'Пароль должен быть не менее 6 символов';
  }
  
  return newErrors;
};

const submitForm = () => {
  const newErrors = validate();
  Object.assign(errors, newErrors);
  
  if (Object.keys(newErrors).length === 0) {
    isLoading.value = true;
    // Отправка данных
    console.log('Форма валидна', { email: email.value, password: password.value });
    // Сброс формы
    email.value = '';
    password.value = '';
    isLoading.value = false;
  }
};
</script>

<style scoped>
.error {
  color: red;
}
</style>
```

## Использование библиотек

### VeeValidate
```vue
<template>
  <Form @submit="onSubmit" :validation-schema="schema">
    <div>
      <Field name="email" type="email" placeholder="Email" />
      <ErrorMessage name="email" />
    </div>
    
    <div>
      <Field name="password" type="password" placeholder="Password" />
      <ErrorMessage name="password" />
    </div>
    
    <button type="submit">Войти</button>
  </Form>
</template>

<script setup>
import { Form, Field, ErrorMessage } from 'vee-validate';
import * as yup from 'yup';

const schema = yup.object({
  email: yup.string().email('Некорректный email').required('Email обязателен'),
  password: yup.string().min(6, 'Пароль должен быть не менее 6 символов').required('Пароль обязателен'),
});

const onSubmit = (values) => {
  console.log(values);
};
</script>
```

### VueUse для валидации
```vue
<template>
  <div>
    <input
      v-model="email"
      type="email"
      placeholder="Email"
      :class="{ error: !emailValid && email.length > 0 }"
    />
    <span v-if="!emailValid && email.length > 0" class="error">Некорректный email</span>
    
    <input
      v-model="password"
      type="password"
      placeholder="Password"
      :class="{ error: !passwordValid && password.length > 0 }"
    />
    <span v-if="!passwordValid && password.length > 0" class="error">Пароль слишком короткий</span>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const email = ref('');
const password = ref('');

const emailValid = computed(() => {
  return /\S+@\S+\.\S+/.test(email.value);
});

const passwordValid = computed(() => {
  return password.value.length >= 6;
});
</script>
```

## Паттерны валидации

### Валидация с помощью валидаторов
```vue
<template>
  <div>
    <input
      v-model="email"
      @blur="validateEmail"
      @input="clearEmailError"
      :class="{ error: emailError }"
      placeholder="Email"
    />
    <span v-if="emailError" class="error">{{ emailError }}</span>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const email = ref('');
const emailError = ref('');

const validateEmail = () => {
  if (!email.value) {
    emailError.value = 'Email обязателен';
  } else if (!/\S+@\S+\.\S+/.test(email.value)) {
    emailError.value = 'Некорректный email';
  }
};

const clearEmailError = () => {
  if (emailError.value) emailError.value = '';
};
</script>
```

### Асинхронная валидация
```vue
<template>
  <div>
    <input
      v-model="username"
      @input="validateUsername"
      :class="{ error: usernameError }"
      placeholder="Username"
    />
    <span v-if="usernameError" class="error">{{ usernameError }}</span>
    <span v-if="checking" class="checking">Проверка...</span>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const username = ref('');
const usernameError = ref('');
const checking = ref(false);

const validateUsername = async () => {
  if (username.value.length < 3) {
    usernameError.value = 'Имя пользователя должно быть не менее 3 символов';
    return;
  }
  
  checking.value = true;
  usernameError.value = '';
  
  try {
    // Асинхронная проверка доступности имени пользователя
    const response = await fetch(`/api/check-username/${username.value}`);
    const result = await response.json();
    
    if (!result.available) {
      usernameError.value = 'Имя пользователя уже занято';
    }
  } catch (error) {
    usernameError.value = 'Ошибка проверки имени пользователя';
  } finally {
    checking.value = false;
  }
};
</script>
```

## Лучшие практики

- **Используй библиотеки**: Для сложных форм используй VeeValidate или другие специализированные библиотеки
- **Показывай ошибки сразу**: Не жди отправки формы для показа ошибок
- **Состояние загрузки**: Показывай индикатор загрузки при асинхронной валидации
- **Тестирование**: Покрой валидацию тестами
- **Удобные сообщения об ошибках**: Делай сообщения понятными и полезными

## Связанные темы
- [[Валидация-на-сервере]]
- [[Валидация-в-React]]
- [[Валидация-в-Svelte]]
- [[Vue-Form-Management]]
- [[UX-дизайн-форм]]

## Ключевые выводы
- Валидация в Vue может быть реализована разными способами в зависимости от сложности
- Библиотеки упрощают управление сложными формами
- Composition API позволяет удобно управлять состоянием валидации
# Form Validation

## Определение
Валидация форм - это процесс проверки данных, вводимых пользователем, на соответствие определенным правилам и ограничениям до отправки формы. Это важный аспект пользовательского опыта и безопасности приложения.

## Встроенные возможности Vue

### Простая валидация
```vue
<template>
  <form @submit.prevent="submitForm">
    <div class="form-group">
      <label for="email">Email:</label>
      <input 
        id="email"
        v-model="form.email" 
        type="email"
        :class="{ 'error': errors.email }"
      >
      <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
    </div>
    
    <div class="form-group">
      <label for="password">Пароль:</label>
      <input 
        id="password"
        v-model="form.password" 
        type="password"
        :class="{ 'error': errors.password }"
      >
      <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
    </div>
    
    <button type="submit" :disabled="!formIsValid">Отправить</button>
  </form>
</template>

<script setup>
import { ref, computed } from 'vue'

const form = ref({
  email: '',
  password: ''
})

const errors = ref({})

const validate = () => {
  errors.value = {}
  
  if (!form.value.email) {
    errors.value.email = 'Email обязателен'
  } else if (!/\S+@\S+\.\S+/.test(form.value.email)) {
    errors.value.email = 'Email недействителен'
  }
  
  if (!form.value.password) {
    errors.value.password = 'Пароль обязателен'
  } else if (form.value.password.length < 6) {
    errors.value.password = 'Пароль должен быть не менее 6 символов'
  }
  
  return Object.keys(errors.value).length === 0
}

const formIsValid = computed(() => {
  return Object.keys(errors.value).length === 0 && 
         form.value.email && 
         form.value.password
})

const submitForm = () => {
  if (validate()) {
    console.log('Форма валидна', form.value)
  }
}
</script>
```

## Популярные библиотеки валидации

### VeeValidate
```vue
<template>
  <Form @submit="onSubmit">
    <Field name="email" type="email" v-slot="{ field, errorMessage }">
      <input v-bind="field" :class="{ 'error': errorMessage }">
      <ErrorMessage name="email" />
    </Field>
    
    <Field name="password" type="password" rules="required|min:6" v-slot="{ field, errorMessage }">
      <input v-bind="field" :class="{ 'error': errorMessage }">
      <ErrorMessage name="password" />
    </Field>
    
    <button type="submit">Отправить</button>
  </Form>
</template>

<script setup>
import { Form, Field, ErrorMessage } from 'vee-validate'

const onSubmit = (values) => {
  console.log('Форма отправлена', values)
}
</script>
```

## Создание собственной системы валидации

### Композабл для валидации
```javascript
// useValidation.js
import { reactive, computed } from 'vue'

export function useValidation(initialState, rules) {
  const state = reactive({ ...initialState })
  const errors = reactive({})
  
  const validateField = (field, value) => {
    if (rules[field]) {
      const ruleResult = rules[field](value, state)
      errors[field] = ruleResult || null
    }
  }
  
  const validateAll = () => {
    let isValid = true
    
    for (const field in rules) {
      const ruleResult = rules[field](state[field], state)
      errors[field] = ruleResult || null
      if (ruleResult) isValid = false
    }
    
    return isValid
  }
  
  const clearErrors = () => {
    for (const field in errors) {
      errors[field] = null
    }
  }
  
  const hasErrors = computed(() => {
    return Object.values(errors).some(error => error !== null)
  })
  
  return {
    state,
    errors,
    validateField,
    validateAll,
    clearErrors,
    hasErrors
  }
}

// Использование
const rules = {
  email: (value) => {
    if (!value) return 'Email обязателен'
    if (!/\S+@\S+\.\S+/.test(value)) return 'Email недействителен'
    return null
  },
  password: (value) => {
    if (!value) return 'Пароль обязателен'
    if (value.length < 6) return 'Пароль должен быть не менее 6 символов'
    return null
  }
}

const { state, errors, validateAll } = useValidation(
  { email: '', password: '' },
  rules
)
```

## Асинхронная валидация

### Проверка доступности имени пользователя
```vue
<template>
  <div class="form-group">
    <label for="username">Имя пользователя:</label>
    <input 
      id="username"
      v-model="form.username" 
      @blur="validateUsername"
      :class="{ 'error': errors.username, 'validating': validating.username }"
      :disabled="validating.username"
    >
    <span v-if="errors.username" class="error-message">{{ errors.username }}</span>
    <span v-if="validating.username" class="validating-message">Проверка...</span>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({ username: '' })
const errors = ref({})
const validating = ref({ username: false })

const validateUsername = async () => {
  if (!form.value.username) {
    errors.value.username = 'Имя пользователя обязательно'
    return
  }
  
  validating.value.username = true
  errors.value.username = null
  
  try {
    const response = await fetch(`/api/check-username/${form.value.username}`)
    const result = await response.json()
    
    if (!result.available) {
      errors.value.username = 'Имя пользователя уже занято'
    }
  } catch (error) {
    errors.value.username = 'Ошибка проверки имени пользователя'
  } finally {
    validating.value.username = false
  }
}
</script>
```

## Реактивная валидация

### Валидация в реальном времени
```vue
<template>
  <input 
    v-model="form.email" 
    @input="validateField('email')"
    :class="{ 'error': errors.email }"
  >
  <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
</template>

<script setup>
import { ref, watch } from 'vue'

const form = ref({ email: '' })
const errors = ref({})

// Валидация при изменении
watch(
  () => form.value.email,
  (newEmail) => {
    errors.value.email = validateEmail(newEmail)
  }
)

const validateEmail = (email) => {
  if (!email) return 'Email обязателен'
  if (!/\S+@\S+\.\S+/.test(email)) return 'Email недействителен'
  return null
}
</script>
```

## Схемы валидации

### Определение правил валидации
```javascript
// validationSchemas.js
export const userSchema = {
  email: {
    required: true,
    pattern: /\S+@\S+\.\S+/,
    message: 'Пожалуйста, введите действительный email'
  },
  password: {
    required: true,
    minLength: 6,
    message: 'Пароль должен быть не менее 6 символов'
  },
  confirmPassword: {
    required: true,
    custom: (value, state) => value === state.password,
    message: 'Пароли не совпадают'
  }
}

export const validate = (schema, data) => {
  const errors = {}
  
  for (const field in schema) {
    const rule = schema[field]
    const value = data[field]
    
    if (rule.required && !value) {
      errors[field] = rule.message
      continue
    }
    
    if (rule.pattern && !rule.pattern.test(value)) {
      errors[field] = rule.message
      continue
    }
    
    if (rule.minLength && value.length < rule.minLength) {
      errors[field] = rule.message
      continue
    }
    
    if (rule.custom && !rule.custom(value, data)) {
      errors[field] = rule.message
      continue
    }
  }
  
  return errors
}
```

## Валидация с состоянием загрузки

```vue
<template>
  <form @submit.prevent="submitForm">
    <input v-model="form.email" type="email">
    
    <button type="submit" :disabled="loading">
      {{ loading ? 'Отправка...' : 'Отправить' }}
    </button>
    
    <div v-if="submitError" class="error-message">
      {{ submitError }}
    </div>
  </form>
</template>

<script setup>
import { ref } from 'vue'

const form = ref({ email: '' })
const loading = ref(false)
const submitError = ref(null)

const submitForm = async () => {
  loading.value = true
  submitError.value = null
  
  try {
    await submitToServer(form.value)
    console.log('Форма успешно отправлена')
  } catch (error) {
    submitError.value = error.message || 'Ошибка отправки формы'
  } finally {
    loading.value = false
  }
}
</script>
```

## Лучшие практики

### Отображение ошибок
- Показывайте ошибки сразу после поля
- Используйте понятные сообщения об ошибках
- Не перегружайте пользователя ошибками

### Производительность
- Избегайте частой асинхронной валидации
- Используйте debounce для валидации ввода

## Тестирование валидации
```javascript
import { mount } from '@vue/test-utils'
import FormComponent from '@/components/FormComponent.vue'

test('показывает ошибку при недействительном email', async () => {
  const wrapper = mount(FormComponent)
  
  await wrapper.find('input[type="email"]').setValue('invalid-email')
  await wrapper.find('input[type="email"]').trigger('blur')
  
  expect(wrapper.find('.error-message').text()).toContain('Email недействителен')
})
```

## Связь с другими концепциями
- [[forms/v-model]] - двусторонняя привязка данных
- [[reactivity/Reactivity System]] - реактивность валидации
- [[components/Components]] - передача ошибок в компоненты
- [[composables/Composables]] - логика валидации как композабл
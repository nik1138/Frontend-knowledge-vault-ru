---
aliases: ["Vue Form Validation", "Валидация форм в Vue", "Vue Validation"]
tags: ["#vue", "#validation", "#frontend", "#forms"]
---

# Валидация в Vue

**Валидация в Vue** - это процесс проверки данных форм в приложениях, построенных с использованием фреймворка Vue.js. Vue предоставляет мощные возможности для реактивного управления состоянием форм и отображения ошибок валидации.

## Общие сведения

Vue.js предлагает несколько подходов к валидации форм:
- Встроенные возможности с использованием реактивного состояния
- Композиционный API с функциями валидации
- Сторонние библиотеки, такие как VeeValidate
- Валидация на основе схем (Yup, Joi и т.д.)

## Встроенные возможности Vue 3

### Простая валидация с Options API

```vue
<template>
  <form @submit.prevent="submitForm">
    <div class="form-group">
      <input
        v-model="form.email"
        type="email"
        placeholder="Email"
        :class="{ 'error': errors.email }"
      />
      <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
    </div>
    
    <div class="form-group">
      <input
        v-model="form.password"
        type="password"
        placeholder="Password"
        :class="{ 'error': errors.password }"
      />
      <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </form>
</template>

<script>
export default {
  name: 'SimpleForm',
  data() {
    return {
      form: {
        email: '',
        password: ''
      },
      errors: {},
      isSubmitting: false
    };
  },
  methods: {
    validateField(name, value) {
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
    },
    
    validateForm() {
      const newErrors = {};
      
      Object.keys(this.form).forEach(key => {
        const error = this.validateField(key, this.form[key]);
        if (error) newErrors[key] = error;
      });
      
      this.errors = newErrors;
      return Object.keys(newErrors).length === 0;
    },
    
    async submitForm() {
      if (!this.validateForm()) return;
      
      this.isSubmitting = true;
      
      try {
        // Отправка данных
        console.log('Данные формы:', this.form);
        // await api.submitForm(this.form);
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        this.isSubmitting = false;
      }
    }
  },
  
  watch: {
    'form.email'(newVal) {
      if (newVal) {
        const error = this.validateField('email', newVal);
        this.$set(this.errors, 'email', error);
      } else {
        this.$delete(this.errors, 'email');
      }
    },
    
    'form.password'(newVal) {
      if (newVal) {
        const error = this.validateField('password', newVal);
        this.$set(this.errors, 'password', error);
      } else {
        this.$delete(this.errors, 'password');
      }
    }
  }
};
</script>

<style scoped>
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

### Валидация с Composition API

```vue
<template>
  <form @submit.prevent="submitForm">
    <div class="form-group">
      <input
        v-model="email"
        type="email"
        placeholder="Email"
        :class="{ 'error': errors.email }"
        @blur="validateField('email', email)"
      />
      <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
    </div>
    
    <div class="form-group">
      <input
        v-model="password"
        type="password"
        placeholder="Password"
        :class="{ 'error': errors.password }"
        @blur="validateField('password', password)"
      />
      <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
    </div>
    
    <div class="form-group">
      <input
        v-model="confirmPassword"
        type="password"
        placeholder="Confirm Password"
        :class="{ 'error': errors.confirmPassword }"
        @blur="validateField('confirmPassword', confirmPassword)"
      />
      <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </form>
</template>

<script>
import { ref, reactive } from 'vue';

export default {
  name: 'CompositionForm',
  setup() {
    // Реактивные переменные
    const email = ref('');
    const password = ref('');
    const confirmPassword = ref('');
    const errors = reactive({});
    const isSubmitting = ref(false);
    
    // Функция валидации поля
    const validateField = (name, value) => {
      let error = '';
      
      switch (name) {
        case 'email':
          if (!value) {
            error = 'Email обязателен';
          } else if (!/^\S+@\S+\.\S+$/.test(value)) {
            error = 'Email некорректен';
          }
          break;
          
        case 'password':
          if (!value) {
            error = 'Пароль обязателен';
          } else if (value.length < 8) {
            error = 'Пароль должен быть не менее 8 символов';
          } else if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
            error = 'Пароль должен содержать заглавную, строчную буквы и цифру';
          }
          break;
          
        case 'confirmPassword':
          if (!value) {
            error = 'Подтверждение пароля обязательно';
          } else if (value !== password.value) {
            error = 'Пароли не совпадают';
          }
          break;
      }
      
      // Обновляем ошибки
      if (error) {
        errors[name] = error;
      } else {
        delete errors[name];
      }
      
      return error;
    };
    
    // Валидация всей формы
    const validateForm = () => {
      validateField('email', email.value);
      validateField('password', password.value);
      validateField('confirmPassword', confirmPassword.value);
      
      return Object.keys(errors).length === 0;
    };
    
    // Отправка формы
    const submitForm = async () => {
      if (!validateForm()) return;
      
      isSubmitting.value = true;
      
      try {
        console.log('Данные формы:', { email: email.value, password: password.value });
        // await api.submitForm({ email: email.value, password: password.value });
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        isSubmitting.value = false;
      }
    };
    
    return {
      email,
      password,
      confirmPassword,
      errors,
      isSubmitting,
      validateField,
      submitForm
    };
  }
};
</script>
```

## Валидация с VeeValidate

VeeValidate - мощная библиотека для валидации форм в Vue:

```vue
<template>
  <Form @submit="submitForm" :validation-schema="schema">
    <div class="form-group">
      <Field
        name="email"
        type="email"
        placeholder="Email"
        class="form-control"
        :class="{ 'error': errors.email }"
      />
      <ErrorMessage name="email" class="error-message" />
    </div>
    
    <div class="form-group">
      <Field
        name="password"
        type="password"
        placeholder="Password"
        class="form-control"
        :class="{ 'error': errors.password }"
      />
      <ErrorMessage name="password" class="error-message" />
    </div>
    
    <div class="form-group">
      <Field
        name="confirmPassword"
        type="password"
        placeholder="Confirm Password"
        class="form-control"
        :class="{ 'error': errors.confirmPassword }"
      />
      <ErrorMessage name="confirmPassword" class="error-message" />
    </div>
    
    <div class="form-group">
      <Field
        name="age"
        type="number"
        placeholder="Age"
        class="form-control"
        :class="{ 'error': errors.age }"
      />
      <ErrorMessage name="age" class="error-message" />
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </Form>
</template>

<script>
import { Form, Field, ErrorMessage } from 'vee-validate';
import * as yup from 'yup';

// Схема валидации
const schema = yup.object({
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
  confirmPassword: yup
    .string()
    .oneOf([yup.ref('password')], 'Пароли должны совпадать')
    .required('Подтверждение пароля обязательно'),
  age: yup
    .number()
    .positive('Возраст должен быть положительным')
    .integer('Возраст должен быть целым числом')
    .min(18, 'Возраст должен быть не менее 18')
    .max(120, 'Возраст должен быть не более 120')
    .required('Возраст обязателен')
});

export default {
  name: 'VeeValidateForm',
  components: {
    Form,
    Field,
    ErrorMessage
  },
  data() {
    return {
      schema,
      isSubmitting: false
    };
  },
  methods: {
    async submitForm(values) {
      this.isSubmitting = true;
      
      try {
        console.log('Данные формы:', values);
        // await api.submitForm(values);
      } catch (error) {
        console.error('Ошибка при отправке:', error);
      } finally {
        this.isSubmitting = false;
      }
    }
  }
};
</script>
```

## Кастомные правила валидации

```javascript
// Кастомные правила для VeeValidate
import { configure, defineRule } from 'vee-validate';
import { localize } from '@vee-validate/i18n';
import { ru } from '@vee-validate/i18n/dist/locale/ru.json';

// Настройка локализации
configure({
  generateMessage: localize('ru', {
    messages: {
      ...ru.messages,
      // Добавление кастомных сообщений
      phone: 'Поле {_field_} должно быть корректным номером телефона'
    }
  })
});

// Кастомное правило для валидации телефона
defineRule('phone', (value) => {
  if (!value) return true; // Пропускаем, если поле необязательное
  
  // Проверяем формат телефона
  const phoneRegex = /^[\+]?[1-9][\d]{0,15}$/;
  return phoneRegex.test(value.replace(/[\s\-\(\)]/g, '')) || 'Некорректный номер телефона';
});

// Кастомное правило для сложности пароля
defineRule('strongPassword', (value) => {
  if (!value) return true;
  
  const minLength = 8;
  const hasUpperCase = /[A-Z]/.test(value);
  const hasLowerCase = /[a-z]/.test(value);
  const hasNumbers = /\d/.test(value);
  const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(value);
  
  if (value.length < minLength) {
    return 'Пароль должен быть не менее 8 символов';
  }
  
  if (!hasUpperCase) {
    return 'Пароль должен содержать заглавную букву';
  }
  
  if (!hasLowerCase) {
    return 'Пароль должен содержать строчную букву';
  }
  
  if (!hasNumbers) {
    return 'Пароль должен содержать цифру';
  }
  
  if (!hasSpecialChar) {
    return 'Пароль должен содержать специальный символ';
  }
  
  return true;
});
```

## Валидация с использованием Composition API и VeeValidate

```vue
<template>
  <form @submit="handleSubmit(submitForm)">
    <div class="form-group">
      <Field
        name="email"
        type="email"
        placeholder="Email"
        v-slot="{ field, meta }"
      >
        <input
          v-bind="field"
          :class="{ 'error': !meta.valid && meta.touched }"
          placeholder="Email"
        />
      </Field>
      <ErrorMessage name="email" class="error-message" />
    </div>
    
    <div class="form-group">
      <Field
        name="password"
        v-slot="{ field, meta }"
        rules="required|min:8|strongPassword"
      >
        <input
          v-bind="field"
          type="password"
          :class="{ 'error': !meta.valid && meta.touched }"
          placeholder="Password"
        />
      </Field>
      <ErrorMessage name="password" class="error-message" />
    </div>
    
    <div class="form-group">
      <Field
        name="terms"
        type="checkbox"
        v-slot="{ field, meta }"
        rules="required"
      >
        <label>
          <input
            v-bind="field"
            type="checkbox"
            :class="{ 'error': !meta.valid && meta.touched }"
          />
          Я согласен с условиями
        </label>
      </Field>
      <ErrorMessage name="terms" class="error-message" />
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </form>
</template>

<script>
import { Field, ErrorMessage, useForm } from 'vee-validate';
import { ref } from 'vue';

export default {
  name: 'CompositionVeeValidateForm',
  components: {
    Field,
    ErrorMessage
  },
  setup() {
    const { handleSubmit, isSubmitting } = useForm();
    
    const submitForm = async (values) => {
      console.log('Данные формы:', values);
      // await api.submitForm(values);
    };
    
    return {
      handleSubmit,
      submitForm,
      isSubmitting
    };
  }
};
</script>
```

## Условная валидация

```vue
<template>
  <Form @submit="submitForm" :validation-schema="schema">
    <div class="form-group">
      <Field
        name="accountType"
        as="select"
        @change="onAccountTypeChange"
      >
        <option value="">Выберите тип аккаунта</option>
        <option value="personal">Личный</option>
        <option value="business">Бизнес</option>
      </Field>
    </div>
    
    <div class="form-group">
      <Field
        name="email"
        type="email"
        placeholder="Email"
      />
      <ErrorMessage name="email" class="error-message" />
    </div>
    
    <div class="form-group">
      <Field
        name="password"
        type="password"
        placeholder="Password"
      />
      <ErrorMessage name="password" class="error-message" />
    </div>
    
    <!-- Условные поля для бизнес-аккаунта -->
    <div v-if="showBusinessFields" class="form-group">
      <Field
        name="companyName"
        type="text"
        placeholder="Название компании"
      />
      <ErrorMessage name="companyName" class="error-message" />
    </div>
    
    <div v-if="showBusinessFields" class="form-group">
      <Field
        name="taxId"
        type="text"
        placeholder="ИНН"
      />
      <ErrorMessage name="taxId" class="error-message" />
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </Form>
</template>

<script>
import { Form, Field, ErrorMessage } from 'vee-validate';
import * as yup from 'yup';
import { ref, computed } from 'vue';

export default {
  name: 'ConditionalValidationForm',
  components: {
    Form,
    Field,
    ErrorMessage
  },
  setup() {
    const showBusinessFields = ref(false);
    
    const schema = computed(() => {
      return yup.object({
        accountType: yup.string().required('Тип аккаунта обязателен'),
        email: yup.string().email('Некорректный email').required('Email обязателен'),
        password: yup.string().min(8).required('Пароль обязателен'),
        ...(showBusinessFields.value && {
          companyName: yup.string().required('Название компании обязательно'),
          taxId: yup.string().matches(/^\d{10}$/, 'ИНН должен содержать 10 цифр').required('ИНН обязателен')
        })
      });
    });
    
    const onAccountTypeChange = (event) => {
      showBusinessFields.value = event.target.value === 'business';
    };
    
    return {
      schema,
      showBusinessFields,
      onAccountTypeChange
    };
  },
  methods: {
    async submitForm(values) {
      console.log('Данные формы:', values);
      // await api.submitForm(values);
    }
  }
};
</script>
```

## Асинхронная валидация

```javascript
// Асинхронная валидация с VeeValidate
import { defineRule } from 'vee-validate';

// Асинхронное правило для проверки уникальности email
defineRule('uniqueEmail', async (value) => {
  if (!value) return true; // Пропускаем, если поле пустое
  
  try {
    // Имитация API-запроса
    const response = await fetch(`/api/check-email?email=${encodeURIComponent(value)}`);
    const result = await response.json();
    
    if (result.exists) {
      return 'Email уже используется';
    }
    
    return true;
  } catch (error) {
    return 'Ошибка при проверке email';
  }
});

// Асинхронное правило для проверки сложности пароля
defineRule('passwordStrength', async (value) => {
  if (!value) return true;
  
  try {
    // Проверка сложности пароля через API
    const response = await fetch('/api/check-password-strength', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ password: value })
    });
    
    const result = await response.json();
    
    if (!result.isStrong) {
      return result.message || 'Пароль недостаточно сложный';
    }
    
    return true;
  } catch (error) {
    return 'Ошибка при проверке сложности пароля';
  }
});
```

## Лучшие практики валидации в Vue

- **Используйте Composition API** для сложных форм с логикой валидации
- **Применяйте VeeValidate** для сложных сценариев валидации
- **Предоставляйте мгновенную обратную связь** при потере фокуса поля
- **Обеспечьте доступность** с помощью ARIA-атрибутов
- **Валидируйте на сервере** - клиентская валидация не заменяет серверную
- **Используйте локализацию** для сообщений об ошибках
- **Оптимизируйте производительность** при валидации больших форм

## Связанные темы

- [[Валидация-на-клиенте]]
- [[Валидация-на-сервере]]
- [[Валидация-в-React]]
- [[Валидация-в-Svelte]]
- [[Vue-Form-Management]]
- [[UX-валидация-форм]]

## Теги

#vue #validation #frontend #forms #javascript #web-development #vee-validate #composition-api
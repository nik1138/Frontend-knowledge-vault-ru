---
aliases: ["Работа с формами в Vue с TypeScript", "Forms in Vue with TypeScript"]
tags: ["#typescript", "#vue", "#forms", "#state-management"]
---

# Формы-в-Vue

Работа с формами в Vue.js с использованием TypeScript включает в себя несколько подходов и паттернов, которые позволяют эффективно управлять состоянием формы, валидацией и обработкой пользовательского ввода. Vue предоставляет мощные инструменты для работы с формами, особенно в сочетании с TypeScript.

## Основы работы с формами в Vue

### Управляемые компоненты в Vue

В Vue управление формами осуществляется через директивы `v-model`, которые связывают значение элемента формы с состоянием компонента.

```typescript
<template>
  <form @submit.prevent="onSubmit">
    <div>
      <label for="name">Имя:</label>
      <input
        id="name"
        v-model="formData.name"
        type="text"
        :class="{ error: errors.name }"
      />
      <span v-if="errors.name" class="error">{{ errors.name }}</span>
    </div>
    
    <div>
      <label for="email">Email:</label>
      <input
        id="email"
        v-model="formData.email"
        type="email"
        :class="{ error: errors.email }"
      />
      <span v-if="errors.email" class="error">{{ errors.email }}</span>
    </div>
    
    <div>
      <label for="message">Сообщение:</label>
      <textarea
        id="message"
        v-model="formData.message"
        :class="{ error: errors.message }"
      ></textarea>
      <span v-if="errors.message" class="error">{{ errors.message }}</span>
    </div>
    
    <div>
      <label>
        <input
          v-model="formData.subscribe"
          type="checkbox"
        />
        Подписаться на рассылку
      </label>
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </form>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
  subscribe: boolean;
}

const initialFormData: ContactFormValues = {
  name: '',
  email: '',
  message: '',
  subscribe: false
};

const formData = reactive<ContactFormValues>({ ...initialFormData });
const errors = reactive<Partial<ContactFormValues>>({});
const isSubmitting = ref(false);

const validate = (): Partial<ContactFormValues> => {
  const newErrors: Partial<ContactFormValues> = {};
  
  if (!formData.name.trim()) {
    newErrors.name = 'Имя обязательно';
  }
  
  if (!formData.email.trim()) {
    newErrors.email = 'Email обязателен';
  } else if (!/^\S+@\S+\.\S+$/.test(formData.email)) {
    newErrors.email = 'Email некорректен';
  }
  
  if (!formData.message.trim()) {
    newErrors.message = 'Сообщение обязательно';
  } else if (formData.message.length < 10) {
    newErrors.message = 'Сообщение должно содержать не менее 10 символов';
  }
  
  return newErrors;
};

const onSubmit = async () => {
  const validationErrors = validate();
  Object.assign(errors, validationErrors);
  
  if (Object.keys(validationErrors).length === 0) {
    isSubmitting.value = true;
    
    try {
      // Имитация отправки данных
      await new Promise(resolve => setTimeout(resolve, 1000));
      console.log('Данные формы:', formData);
      
      // Сброс формы после успешной отправки
      Object.assign(formData, initialFormData);
      Object.keys(errors).forEach(key => delete errors[key as keyof ContactFormValues]);
    } catch (error) {
      console.error('Ошибка при отправке формы:', error);
    } finally {
      isSubmitting.value = false;
    }
  }
};
</script>

<style scoped>
.error {
  color: red;
}
</style>
```

## Популярные библиотеки для работы с формами в Vue

### VeeValidate

VeeValidate — это мощная библиотека для валидации форм в Vue, которая поддерживает TypeScript и предоставляет декларативный подход к валидации.

```typescript
<template>
  <Form @submit="onSubmit" :validation-schema="contactSchema">
    <div>
      <label for="name">Имя:</label>
      <Field
        id="name"
        name="name"
        type="text"
        :class="{ error: errors.name }"
      />
      <ErrorMessage name="name" class="error" />
    </div>
    
    <div>
      <label for="email">Email:</label>
      <Field
        id="email"
        name="email"
        type="email"
        :class="{ error: errors.email }"
      />
      <ErrorMessage name="email" class="error" />
    </div>
    
    <div>
      <label for="message">Сообщение:</label>
      <Field
        id="message"
        name="message"
        as="textarea"
        :class="{ error: errors.message }"
      />
      <ErrorMessage name="message" class="error" />
    </div>
    
    <div>
      <label>
        <Field
          name="subscribe"
          type="checkbox"
        />
        Подписаться на рассылку
      </label>
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </Form>
</template>

<script setup lang="ts">
import { Form, Field, ErrorMessage } from 'vee-validate';
import { object, string, boolean } from 'yup';
import { ref } from 'vue';

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
  subscribe: boolean;
}

const contactSchema = object({
  name: string().required('Имя обязательно').min(2, 'Имя должно содержать не менее 2 символов'),
  email: string().required('Email обязателен').email('Некорректный email'),
  message: string().required('Сообщение обязательно').min(10, 'Сообщение должно содержать не менее 10 символов'),
  subscribe: boolean()
});

const isSubmitting = ref(false);

const onSubmit = async (values: ContactFormValues) => {
  isSubmitting.value = true;
  
  try {
    // Имитация отправки данных
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('Данные формы:', values);
  } catch (error) {
    console.error('Ошибка при отправке формы:', error);
  } finally {
    isSubmitting.value = false;
  }
};
</script>
```

### Vue UseForm

Vue UseForm — это хук для управления формами в Vue, вдохновленный React Hook Form.

```typescript
<template>
  <form @submit="handleSubmit(onSubmit)">
    <div>
      <label for="name">Имя:</label>
      <input
        id="name"
        v-model="fields.name.value"
        type="text"
        :class="{ error: !!fields.name.error }"
      />
      <span v-if="fields.name.error" class="error">{{ fields.name.error }}</span>
    </div>
    
    <div>
      <label for="email">Email:</label>
      <input
        id="email"
        v-model="fields.email.value"
        type="email"
        :class="{ error: !!fields.email.error }"
      />
      <span v-if="fields.email.error" class="error">{{ fields.email.error }}</span>
    </div>
    
    <div>
      <label for="message">Сообщение:</label>
      <textarea
        id="message"
        v-model="fields.message.value"
        :class="{ error: !!fields.message.error }"
      ></textarea>
      <span v-if="fields.message.error" class="error">{{ fields.message.error }}</span>
    </div>
    
    <div>
      <label>
        <input
          v-model="fields.subscribe.value"
          type="checkbox"
        />
        Подписаться на рассылку
      </label>
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </form>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue';
import { useForm, createField } from '@vueuse/forms';

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
  subscribe: boolean;
}

const initialFormData: ContactFormValues = {
  name: '',
  email: '',
  message: '',
  subscribe: false
};

const { fields, handleSubmit, isSubmitting } = useForm<ContactFormValues>({
  initialValues: initialFormData,
  onSubmit: async (values) => {
    // Имитация отправки данных
    await new Promise(resolve => setTimeout(resolve, 1000));
    console.log('Данные формы:', values);
  },
  validate: (values) => {
    const errors: Partial<ContactFormValues> = {};
    
    if (!values.name.trim()) {
      errors.name = 'Имя обязательно';
    } else if (values.name.length < 2) {
      errors.name = 'Имя должно содержать не менее 2 символов';
    }
    
    if (!values.email.trim()) {
      errors.email = 'Email обязателен';
    } else if (!/^\S+@\S+\.\S+$/.test(values.email)) {
      errors.email = 'Email некорректен';
    }
    
    if (!values.message.trim()) {
      errors.message = 'Сообщение обязательно';
    } else if (values.message.length < 10) {
      errors.message = 'Сообщение должно содержать не менее 10 символов';
    }
    
    return errors;
  }
});

// Создание полей с типизацией
fields.name = createField('name', '');
fields.email = createField('email', '');
fields.message = createField('message', '');
fields.subscribe = createField('subscribe', false);
</script>
```

## Работа с композитными компонентами

Для повторного использования компонентов форм с правильной типизацией можно использовать композитные компоненты:

```typescript
<!-- InputField.vue -->
<template>
  <div class="input-field">
    <label v-if="label" :for="id">{{ label }}</label>
    <input
      :id="id"
      v-bind="$attrs"
      :value="modelValue"
      @input="handleInput"
      :class="{ error: !!error }"
    />
    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>

<script setup lang="ts">
interface Props {
  modelValue: string | number;
  id: string;
  label?: string;
  error?: string;
}

interface Emits {
  (e: 'update:modelValue', value: string | number): void;
}

const props = withDefaults(defineProps<Props>(), {
  label: undefined,
  error: undefined
});

const emit = defineEmits<Emits>();

const handleInput = (e: Event) => {
  const target = e.target as HTMLInputElement;
  emit('update:modelValue', target.value);
};
</script>

<style scoped>
.input-field {
  margin-bottom: 1rem;
}

.error {
  color: red;
  font-size: 0.875rem;
}
</style>
```

```typescript
<!-- Использование композитного компонента -->
<template>
  <form @submit.prevent="onSubmit">
    <InputField
      id="name"
      v-model="formData.name"
      label="Имя"
      type="text"
      :error="errors.name"
    />
    
    <InputField
      id="email"
      v-model="formData.email"
      label="Email"
      type="email"
      :error="errors.email"
    />
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? 'Отправка...' : 'Отправить' }}
    </button>
  </form>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import InputField from './InputField.vue';

interface ContactFormValues {
  name: string;
  email: string;
}

const formData = reactive<ContactFormValues>({
  name: '',
  email: ''
});

const errors = reactive<Partial<ContactFormValues>>({});
const isSubmitting = ref(false);

const validate = (): Partial<ContactFormValues> => {
  const newErrors: Partial<ContactFormValues> = {};
  
  if (!formData.name.trim()) {
    newErrors.name = 'Имя обязательно';
  }
  
  if (!formData.email.trim()) {
    newErrors.email = 'Email обязателен';
  } else if (!/^\S+@\S+\.\S+$/.test(formData.email)) {
    newErrors.email = 'Email некорректен';
  }
  
  return newErrors;
};

const onSubmit = async () => {
  const validationErrors = validate();
  Object.assign(errors, validationErrors);
  
  if (Object.keys(validationErrors).length === 0) {
    isSubmitting.value = true;
    
    try {
      await new Promise(resolve => setTimeout(resolve, 1000));
      console.log('Данные формы:', formData);
    } catch (error) {
      console.error('Ошибка при отправке формы:', error);
    } finally {
      isSubmitting.value = false;
    }
  }
};
</script>
```

## Лучшие практики

- Используйте `v-model` для двусторонней привязки данных
- Применяйте строгую типизацию для значений форм
- Используйте валидацию на стороне клиента для улучшения UX
- Используйте библиотеки для сложных сценариев работы с формами
- Создавайте переиспользуемые компоненты для полей форм
- Обеспечьте доступность форм (accessibility)
- Обрабатывайте различные состояния формы (загрузка, ошибка)

## Связанные темы

- [[Управление-формами]]
- [[Валидация-форм]]
- [[Формы-в-React]]
- [[Формы-в-Angular]]
- [[Vue-с-typescript]]
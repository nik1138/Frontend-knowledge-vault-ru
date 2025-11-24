---
aliases: ["Vue Forms", "Формы в Vue", "Form Handling Vue"]
tags: ["#frontend", "#vue", "#forms", "#state-management", "#validation"]
---

# Формы в Vue

Работа с формами в Vue.js предоставляет разработчикам гибкие и мощные инструменты для создания интерактивных пользовательских интерфейсов. Vue предлагает встроенные возможности для управления формами, а также поддерживает сторонние библиотеки для сложных сценариев.

## Введение в формы в Vue

Vue.js использует двустороннюю привязку данных (v-model) для упрощения работы с формами. Это позволяет легко синхронизировать состояние компонента с вводом пользователя.

### Простая форма с v-model

```vue
<template>
  <form @submit.prevent="submitForm">
    <div>
      <label for="name">Имя:</label>
      <input 
        id="name" 
        v-model="form.name" 
        type="text" 
        placeholder="Введите ваше имя"
      />
    </div>
    
    <div>
      <label for="email">Email:</label>
      <input 
        id="email" 
        v-model="form.email" 
        type="email" 
        placeholder="Введите ваш email"
      />
    </div>
    
    <div>
      <label for="message">Сообщение:</label>
      <textarea 
        id="message" 
        v-model="form.message" 
        placeholder="Введите ваше сообщение"
      ></textarea>
    </div>
    
    <button type="submit">Отправить</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        name: '',
        email: '',
        message: ''
      }
    }
  },
  methods: {
    submitForm() {
      console.log('Отправленные данные:', this.form);
      // Здесь можно добавить логику отправки данных на сервер
    }
  }
}
</script>
```

## Управление состоянием формы

### Локальное состояние
Для простых форм достаточно использовать локальное состояние компонента:

```vue
<template>
  <form @submit.prevent="onSubmit">
    <input v-model="username" type="text" placeholder="Имя пользователя" />
    <input v-model="password" type="password" placeholder="Пароль" />
    <button type="submit">Войти</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      username: '',
      password: ''
    }
  },
  methods: {
    onSubmit() {
      // Обработка отправки формы
      console.log({ username: this.username, password: this.password });
    }
  }
}
</script>
```

### Использование Composition API (Vue 3)

```vue
<template>
  <form @submit.prevent="submitForm">
    <input v-model="formData.username" type="text" placeholder="Имя пользователя" />
    <input v-model="formData.email" type="email" placeholder="Email" />
    <button type="submit">Отправить</button>
  </form>
</template>

<script>
import { reactive } from 'vue';

export default {
  setup() {
    const formData = reactive({
      username: '',
      email: ''
    });
    
    const submitForm = () => {
      console.log('Данные формы:', formData);
    };
    
    return {
      formData,
      submitForm
    };
  }
}
</script>
```

## Валидация форм

### Простая валидация на клиенте

```vue
<template>
  <form @submit.prevent="validateAndSubmit">
    <div class="form-group">
      <label for="email">Email:</label>
      <input 
        id="email" 
        v-model="form.email" 
        type="email" 
        :class="{ 'error': errors.email }"
        @blur="validateEmail"
      />
      <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
    </div>
    
    <div class="form-group">
      <label for="password">Пароль:</label>
      <input 
        id="password" 
        v-model="form.password" 
        type="password" 
        :class="{ 'error': errors.password }"
        @blur="validatePassword"
      />
      <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
    </div>
    
    <button type="submit" :disabled="!isFormValid">Войти</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        email: '',
        password: ''
      },
      errors: {
        email: null,
        password: null
      }
    }
  },
  computed: {
    isFormValid() {
      return this.form.email && this.form.password && !this.errors.email && !this.errors.password;
    }
  },
  methods: {
    validateEmail() {
      if (!this.form.email) {
        this.errors.email = 'Email обязателен';
      } else if (!/\S+@\S+\.\S+/.test(this.form.email)) {
        this.errors.email = 'Email некорректен';
      } else {
        this.errors.email = null;
      }
    },
    validatePassword() {
      if (!this.form.password) {
        this.errors.password = 'Пароль обязателен';
      } else if (this.form.password.length < 6) {
        this.errors.password = 'Пароль должен быть не менее 6 символов';
      } else {
        this.errors.password = null;
      }
    },
    validateAndSubmit() {
      this.validateEmail();
      this.validatePassword();
      
      if (this.isFormValid) {
        console.log('Форма валидна, отправляем:', this.form);
        // Отправка данных
      }
    }
  }
}
</script>

<style>
.error {
  border: 1px solid red;
}

.error-message {
  color: red;
  font-size: 0.8em;
}
</style>
```

### Использование сторонних библиотек валидации

Vue поддерживает различные библиотеки валидации, такие как:
- [[VeeValidate]] - мощная библиотека для валидации форм
- Vue Formulate - гибкая система для создания форм

Пример с VeeValidate:

```vue
<template>
  <Form @submit="onSubmit" :validation-schema="schema">
    <div>
      <label for="email">Email:</label>
      <Field id="email" name="email" type="email" class="form-control" />
      <ErrorMessage name="email" class="error-message" />
    </div>
    
    <div>
      <label for="password">Пароль:</label>
      <Field id="password" name="password" type="password" class="form-control" />
      <ErrorMessage name="password" class="error-message" />
    </div>
    
    <button type="submit">Войти</button>
  </Form>
</template>

<script>
import { Form, Field, ErrorMessage } from 'vee-validate';
import * as yup from 'yup';

export default {
  components: {
    Form,
    Field,
    ErrorMessage
  },
  data() {
    return {
      schema: yup.object({
        email: yup
          .string()
          .required('Email обязателен')
          .email('Некорректный email'),
        password: yup
          .string()
          .required('Пароль обязателен')
          .min(6, 'Пароль должен быть не менее 6 символов')
      })
    }
  },
  methods: {
    onSubmit(values) {
      console.log('Форма отправлена:', values);
    }
  }
}
</script>
```

## Работа с различными типами полей

### Чекбоксы и радиокнопки

```vue
<template>
  <form @submit.prevent="submitForm">
    <!-- Одиночный чекбокс -->
    <div>
      <input 
        id="agreement" 
        v-model="form.agreement" 
        type="checkbox" 
      />
      <label for="agreement">Согласен с условиями</label>
    </div>
    
    <!-- Множественные чекбоксы -->
    <div>
      <h3>Интересы:</h3>
      <label v-for="interest in interests" :key="interest.value">
        <input
          v-model="form.selectedInterests"
          type="checkbox"
          :value="interest.value"
        />
        {{ interest.label }}
      </label>
    </div>
    
    <!-- Радиокнопки -->
    <div>
      <h3>Пол:</h3>
      <label v-for="gender in genders" :key="gender.value">
        <input
          v-model="form.gender"
          type="radio"
          :value="gender.value"
        />
        {{ gender.label }}
      </label>
    </div>
    
    <button type="submit">Отправить</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        agreement: false,
        selectedInterests: [],
        gender: ''
      },
      interests: [
        { value: 'sports', label: 'Спорт' },
        { value: 'music', label: 'Музыка' },
        { value: 'travel', label: 'Путешествия' }
      ],
      genders: [
        { value: 'male', label: 'Мужской' },
        { value: 'female', label: 'Женский' },
        { value: 'other', label: 'Другой' }
      ]
    }
  },
  methods: {
    submitForm() {
      console.log('Данные формы:', this.form);
    }
  }
}
</script>
```

### Выпадающие списки

```vue
<template>
  <form @submit.prevent="submitForm">
    <div>
      <label for="country">Страна:</label>
      <select id="country" v-model="form.country">
        <option value="">Выберите страну</option>
        <option 
          v-for="country in countries" 
          :key="country.value" 
          :value="country.value"
        >
          {{ country.label }}
        </option>
      </select>
    </div>
    
    <div>
      <label for="city">Город:</label>
      <select id="city" v-model="form.city" :disabled="!form.country">
        <option value="">Выберите город</option>
        <option 
          v-for="city in getCitiesByCountry(form.country)" 
          :key="city.value" 
          :value="city.value"
        >
          {{ city.label }}
        </option>
      </select>
    </div>
    
    <button type="submit">Отправить</button>
  </form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        country: '',
        city: ''
      },
      countries: [
        { value: 'ru', label: 'Россия' },
        { value: 'us', label: 'США' },
        { value: 'de', label: 'Германия' }
      ],
      cities: {
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
      }
    }
  },
  methods: {
    getCitiesByCountry(country) {
      return this.cities[country] || [];
    },
    submitForm() {
      console.log('Данные формы:', this.form);
    }
  }
}
</script>
```

## Практические примеры

### Форма регистрации с валидацией

```vue
<template>
  <div class="registration-form">
    <h2>Регистрация</h2>
    <form @submit.prevent="register">
      <div class="form-group">
        <label for="username">Имя пользователя:</label>
        <input
          id="username"
          v-model="form.username"
          type="text"
          :class="{ 'error': errors.username }"
          @blur="validateUsername"
          placeholder="Введите имя пользователя"
        />
        <span v-if="errors.username" class="error-message">{{ errors.username }}</span>
      </div>
      
      <div class="form-group">
        <label for="email">Email:</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          :class="{ 'error': errors.email }"
          @blur="validateEmail"
          placeholder="Введите email"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>
      
      <div class="form-group">
        <label for="password">Пароль:</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          :class="{ 'error': errors.password }"
          @blur="validatePassword"
          placeholder="Введите пароль"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
      </div>
      
      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля:</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          :class="{ 'error': errors.confirmPassword }"
          @blur="validateConfirmPassword"
          placeholder="Подтвердите пароль"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
      </div>
      
      <div class="form-group">
        <label>
          <input
            v-model="form.agreement"
            type="checkbox"
            @change="validateAgreement"
          />
          Я согласен с условиями использования
        </label>
        <span v-if="errors.agreement" class="error-message">{{ errors.agreement }}</span>
      </div>
      
      <button type="submit" :disabled="isSubmitting">
        {{ isSubmitting ? 'Регистрация...' : 'Зарегистрироваться' }}
      </button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        username: '',
        email: '',
        password: '',
        confirmPassword: '',
        agreement: false
      },
      errors: {
        username: null,
        email: null,
        password: null,
        confirmPassword: null,
        agreement: null
      },
      isSubmitting: false
    }
  },
  methods: {
    validateUsername() {
      if (!this.form.username) {
        this.errors.username = 'Имя пользователя обязательно';
      } else if (this.form.username.length < 3) {
        this.errors.username = 'Имя пользователя должно быть не менее 3 символов';
      } else {
        this.errors.username = null;
      }
    },
    
    validateEmail() {
      if (!this.form.email) {
        this.errors.email = 'Email обязателен';
      } else if (!/\S+@\S+\.\S+/.test(this.form.email)) {
        this.errors.email = 'Email некорректен';
      } else {
        this.errors.email = null;
      }
    },
    
    validatePassword() {
      if (!this.form.password) {
        this.errors.password = 'Пароль обязателен';
      } else if (this.form.password.length < 6) {
        this.errors.password = 'Пароль должен быть не менее 6 символов';
      } else {
        this.errors.password = null;
      }
    },
    
    validateConfirmPassword() {
      if (this.form.password !== this.form.confirmPassword) {
        this.errors.confirmPassword = 'Пароли не совпадают';
      } else {
        this.errors.confirmPassword = null;
      }
    },
    
    validateAgreement() {
      if (!this.form.agreement) {
        this.errors.agreement = 'Необходимо согласие с условиями';
      } else {
        this.errors.agreement = null;
      }
    },
    
    validateForm() {
      this.validateUsername();
      this.validateEmail();
      this.validatePassword();
      this.validateConfirmPassword();
      this.validateAgreement();
      
      return !Object.values(this.errors).some(error => error !== null);
    },
    
    async register() {
      if (!this.validateForm()) {
        return;
      }
      
      this.isSubmitting = true;
      
      try {
        // Имитация API вызова
        await new Promise(resolve => setTimeout(resolve, 1000));
        console.log('Регистрация успешна:', this.form);
        
        // Сброс формы после успешной регистрации
        this.form = {
          username: '',
          email: '',
          password: '',
          confirmPassword: '',
          agreement: false
        };
        this.errors = {};
      } catch (error) {
        console.error('Ошибка регистрации:', error);
      } finally {
        this.isSubmitting = false;
      }
    }
  }
}
</script>

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

.form-group input.error {
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

## Лучшие практики

1. **Используйте v-model** для двусторонней привязки данных
2. **Предотвращайте стандартное поведение формы** с помощью `.prevent`
3. **Валидируйте данные на клиенте** для улучшения UX
4. **Очищайте ошибки при изменении полей** для улучшения UX
5. **Используйте TypeScript** для строгой типизации форм
6. **Разделяйте сложные формы** на несколько компонентов
7. **Обеспечивайте доступность** с помощью правильных меток и ARIA-атрибутов
8. **Показывайте состояние загрузки** при отправке формы
9. **Обрабатывайте ошибки сервера** и отображайте их пользователю
10. **Используйте Composition API** для более сложной логики управления формой

## Заключение

Формы в Vue.js предоставляют мощные и гибкие инструменты для создания интерактивных пользовательских интерфейсов. Использование v-model, встроенных возможностей валидации и сторонних библиотек позволяет создавать надежные и удобные формы. Правильная реализация форм улучшает пользовательский опыт и предотвращает ошибки ввода данных.

См. также: [[Управление-состоянием-форм]], [[Формы-в-React]], [[Формы-в-Svelte]]
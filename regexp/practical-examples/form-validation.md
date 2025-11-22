---
aliases: ["Регулярные выражения в валидации форм", "Формы и регулярки", "Валидация форм"]
tags: ["#regexp", "#frontend", "#react", "#vue", "#validation", "#forms"]
---

# Регулярные выражения в валидации форм

## Введение

Валидация форм - один из ключевых аспектов разработки пользовательского интерфейса. Регулярные выражения позволяют эффективно проверять корректность вводимых данных, обеспечивая целостность и безопасность приложения.

## Основные паттерны валидации

### Email адреса

```javascript
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
```

**Пример использования в React компоненте:**

```jsx
import React, { useState } from 'react';

const EmailValidation = () => {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  
  const validateEmail = (email) => {
    if (!email) {
      setError('Email обязателен');
      return false;
    }
    
    if (!emailRegex.test(email)) {
      setError('Некорректный email адрес');
      return false;
    }
    
    setError('');
    return true;
  };
  
  const handleChange = (e) => {
    const value = e.target.value;
    setEmail(value);
    validateEmail(value);
  };
  
  return (
    <div>
      <input 
        type="email" 
        value={email} 
        onChange={handleChange} 
        placeholder="Введите email"
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
};

export default EmailValidation;
```

### Номер телефона

```javascript
// Паттерн для российского номера телефона
const phoneRegex = /^(\+7|7|8)?[\s-]?\(?[0-9]{3}\)?[\s-]?[0-9]{3}[\s-]?[0-9]{2}[\s-]?[0-9]{2}$/;
```

**Пример валидации в Vue компоненте:**

```vue
<template>
  <div>
    <input 
      v-model="phone" 
      @input="validatePhone"
      placeholder="Введите номер телефона"
    />
    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>

<script>
export default {
  data() {
    return {
      phone: '',
      error: ''
    };
  },
  methods: {
    validatePhone() {
      const phoneRegex = /^(\+7|7|8)?[\s-]?\(?[0-9]{3}\)?[\s-]?[0-9]{3}[\s-]?[0-9]{2}[\s-]?[0-9]{2}$/;
      
      if (!this.phone) {
        this.error = 'Телефон обязателен';
        return false;
      }
      
      if (!phoneRegex.test(this.phone)) {
        this.error = 'Некорректный номер телефона';
        return false;
      }
      
      this.error = '';
      return true;
    }
  }
};
</script>
```

### Пароль

```javascript
// Пароль должен содержать минимум 8 символов, заглавную букву, цифру и специальный символ
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
```

## Продвинутые паттерны

### URL валидация

```javascript
const urlRegex = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/;
```

### Валидация даты (DD/MM/YYYY)

```javascript
const dateRegex = /^(0[1-9]|[12][0-9]|3[01])\/(0[1-9]|1[0-2])\/\d{4}$/;
```

### Валидация ИНН (российский)

```javascript
const innRegex = /^\d{10}$|^\d{12}$/;
```

## Практические примеры

### Валидация формы регистрации в React

```jsx
import React, { useState } from 'react';

const RegistrationForm = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    phone: '',
    name: ''
  });
  
  const [errors, setErrors] = useState({});
  
  // Регулярные выражения для валидации
  const patterns = {
    email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
    password: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
    phone: /^(\+7|7|8)?[\s-]?\(?[0-9]{3}\)?[\s-]?[0-9]{3}[\s-]?[0-9]{2}[\s-]?[0-9]{2}$/,
    name: /^[a-zA-Zа-яА-ЯёЁ\s]{2,30}$/
  };
  
  const validateField = (name, value) => {
    if (!value) {
      return `${getFieldName(name)} обязателен`;
    }
    
    if (name === 'email' && !patterns.email.test(value)) {
      return 'Некорректный email адрес';
    }
    
    if (name === 'password' && !patterns.password.test(value)) {
      return 'Пароль должен содержать минимум 8 символов, заглавную букву, цифру и специальный символ';
    }
    
    if (name === 'phone' && !patterns.phone.test(value)) {
      return 'Некорректный номер телефона';
    }
    
    if (name === 'name' && !patterns.name.test(value)) {
      return 'Имя должно содержать только буквы и быть длиной от 2 до 30 символов';
    }
    
    return '';
  };
  
  const getFieldName = (name) => {
    const names = {
      email: 'Email',
      password: 'Пароль',
      phone: 'Телефон',
      name: 'Имя'
    };
    return names[name] || name;
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData({
      ...formData,
      [name]: value
    });
    
    const error = validateField(name, value);
    setErrors({
      ...errors,
      [name]: error
    });
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const newErrors = {};
    Object.keys(formData).forEach(field => {
      const error = validateField(field, formData[field]);
      if (error) {
        newErrors[field] = error;
      }
    });
    
    setErrors(newErrors);
    
    if (Object.keys(newErrors).length === 0) {
      console.log('Форма валидна!', formData);
      // Отправка данных на сервер
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input 
          type="text" 
          name="name" 
          value={formData.name} 
          onChange={handleChange} 
          placeholder="Имя"
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <input 
          type="email" 
          name="email" 
          value={formData.email} 
          onChange={handleChange} 
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <input 
          type="password" 
          name="password" 
          value={formData.password} 
          onChange={handleChange} 
          placeholder="Пароль"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      
      <div>
        <input 
          type="text" 
          name="phone" 
          value={formData.phone} 
          onChange={handleChange} 
          placeholder="Телефон"
        />
        {errors.phone && <span className="error">{errors.phone}</span>}
      </div>
      
      <button type="submit">Зарегистрироваться</button>
    </form>
  );
};

export default RegistrationForm;
```

### Валидация формы в Vue с Composition API

```vue
<template>
  <form @submit.prevent="submitForm">
    <div>
      <input 
        v-model="form.email" 
        @blur="validateField('email')"
        placeholder="Email"
      />
      <span v-if="errors.email" class="error">{{ errors.email }}</span>
    </div>
    
    <div>
      <input 
        v-model="form.password" 
        type="password"
        @blur="validateField('password')"
        placeholder="Пароль"
      />
      <span v-if="errors.password" class="error">{{ errors.password }}</span>
    </div>
    
    <button type="submit" :disabled="!isFormValid">Отправить</button>
  </form>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  setup() {
    const form = ref({
      email: '',
      password: ''
    });
    
    const errors = ref({});
    
    const patterns = {
      email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
      password: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/
    };
    
    const validateField = (fieldName) => {
      const value = form.value[fieldName];
      const pattern = patterns[fieldName];
      
      if (!value) {
        errors.value[fieldName] = `${getFieldName(fieldName)} обязателен`;
        return false;
      }
      
      if (!pattern.test(value)) {
        errors.value[fieldName] = `Некорректный формат ${getFieldName(fieldName).toLowerCase()}`;
        return false;
      }
      
      errors.value[fieldName] = '';
      return true;
    };
    
    const getFieldName = (name) => {
      const names = {
        email: 'Email',
        password: 'Пароль'
      };
      return names[name] || name;
    };
    
    const isFormValid = computed(() => {
      return Object.keys(form.value).every(field => 
        validateField(field)
      );
    });
    
    const submitForm = () => {
      if (isFormValid.value) {
        console.log('Форма валидна!', form.value);
        // Отправка данных на сервер
      }
    };
    
    return {
      form,
      errors,
      validateField,
      isFormValid,
      submitForm
    };
  }
};
</script>
```

## Лучшие практики

### 1. Кэширование регулярных выражений

Для производительности лучше создавать регулярные выражения заранее:

```javascript
// Плохо - создание нового экземпляра при каждой проверке
const validateEmail = (email) => {
  return /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(email);
};

// Хорошо - кэширование регулярного выражения
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
const validateEmail = (email) => {
  return emailRegex.test(email);
};
```

### 2. Комбинирование с другими методами валидации

Регулярные выражения - не единственный способ валидации:

```javascript
const validatePassword = (password) => {
  // Проверка длины
  if (password.length < 8) {
    return 'Пароль должен быть не менее 8 символов';
  }
  
  // Проверка с помощью регулярного выражения
  const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
  if (!passwordRegex.test(password)) {
    return 'Пароль должен содержать заглавную букву, цифру и специальный символ';
  }
  
  return '';
};
```

### 3. Использование библиотек валидации

Для сложных форм рекомендуется использовать специализированные библиотеки, которые поддерживают регулярные выражения:

- [[Yup]] - для React
- [[VeeValidate]] - для Vue
- [[Joi]] - для Node.js

## Заключение

Регулярные выражения в валидации форм позволяют эффективно проверять корректность вводимых данных на клиентской стороне, улучшая пользовательский опыт и обеспечивая целостность данных. Важно использовать их разумно, комбинируя с другими методами валидации и учитывая производительность.

Для более сложных сценариев рекомендуется использовать специализированные библиотеки валидации, которые предоставляют более удобный интерфейс и встроенные паттерны для часто используемых проверок.
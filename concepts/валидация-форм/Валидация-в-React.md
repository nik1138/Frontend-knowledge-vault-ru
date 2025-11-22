---
aliases: ["React Form Validation", "Валидация форм в React", "React Validation"]
tags: ["#react", "#validation", "#frontend", "#forms"]
---

# Валидация в React

**Валидация в React** - это процесс проверки данных форм в приложениях, построенных с использованием фреймворка React. В отличие от традиционного подхода с чистым JavaScript, React предоставляет уникальные возможности для управления состоянием и обработки событий, что делает валидацию более предсказуемой и управляемой.

## Общие сведения

Валидация форм в React обычно включает в себя:
- Управление состоянием формы
- Обработку событий ввода
- Отображение сообщений об ошибках
- Проверку данных перед отправкой

React предлагает несколько подходов к валидации форм, от простых встроенных решений до мощных библиотек.

## Встроенные возможности React

### Простая валидация с useState

```jsx
import React, { useState } from 'react';

const SimpleForm = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  
  const [errors, setErrors] = useState({});
  
  const validateField = (name, value) => {
    switch (name) {
      case 'email':
        if (!value) return 'Email обязателен';
        if (!/\S+@\S+\.\S+/.test(value)) return 'Email некорректен';
        break;
      case 'password':
        if (!value) return 'Пароль обязателен';
        if (value.length < 8) return 'Пароль должен быть не менее 8 символов';
        break;
      default:
        break;
    }
    return '';
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Валидация при изменении поля
    const error = validateField(name, value);
    setErrors(prev => ({
      ...prev,
      [name]: error
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Валидация всех полей при отправке
    const newErrors = {};
    Object.keys(formData).forEach(key => {
      const error = validateField(key, formData[key]);
      if (error) newErrors[key] = error;
    });
    
    setErrors(newErrors);
    
    if (Object.keys(newErrors).length === 0) {
      console.log('Форма валидна:', formData);
      // Отправка данных
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
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
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      
      <button type="submit">Отправить</button>
    </form>
  );
};

export default SimpleForm;
```

## Использование хука для валидации

```jsx
import { useState, useCallback } from 'react';

// Пользовательский хук для валидации
const useFormValidation = (initialState, validationRules) => {
  const [values, setValues] = useState(initialState);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validate = useCallback((fieldName, value) => {
    if (validationRules[fieldName]) {
      return validationRules[fieldName](value);
    }
    return '';
  }, [validationRules]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: value
    }));

    // Валидация при изменении
    const error = validate(name, value);
    setErrors(prev => ({
      ...prev,
      [name]: error
    }));
  };

  const handleBlur = (e) => {
    const { name, value } = e.target;
    const error = validate(name, value);
    setErrors(prev => ({
      ...prev,
      [name]: error
    }));
  };

  const handleSubmit = async (onSubmit) => {
    setIsSubmitting(true);
    
    // Валидация всех полей
    const newErrors = {};
    Object.keys(values).forEach(key => {
      const error = validate(key, values[key]);
      if (error) newErrors[key] = error;
    });
    
    setErrors(newErrors);
    
    if (Object.keys(newErrors).length === 0) {
      await onSubmit(values);
    }
    
    setIsSubmitting(false);
  };

  return {
    values,
    errors,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit
  };
};

// Использование хука
const validationRules = {
  email: (value) => {
    if (!value) return 'Email обязателен';
    if (!/\S+@\S+\.\S+/.test(value)) return 'Email некорректен';
    return '';
  },
  password: (value) => {
    if (!value) return 'Пароль обязателен';
    if (value.length < 8) return 'Пароль должен быть не менее 8 символов';
    if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
      return 'Пароль должен содержать заглавную, строчную буквы и цифру';
    }
    return '';
  }
};

const MyForm = () => {
  const { values, errors, isSubmitting, handleChange, handleBlur, handleSubmit } = 
    useFormValidation(
      { email: '', password: '' },
      validationRules
    );

  const onSubmit = async (formData) => {
    console.log('Отправка данных:', formData);
    // Отправка на сервер
  };

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      handleSubmit(onSubmit);
    }}>
      <div>
        <input
          type="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Отправка...' : 'Отправить'}
      </button>
    </form>
  );
};
```

## Валидация с использованием Yup и Formik

Formik - популярная библиотека для управления формами в React:

```jsx
import React from 'react';
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

// Схема валидации с Yup
const validationSchema = Yup.object({
  email: Yup.string()
    .email('Некорректный email')
    .required('Email обязателен'),
  password: Yup.string()
    .min(8, 'Пароль должен быть не менее 8 символов')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, 
             'Пароль должен содержать заглавную, строчную буквы и цифру')
    .required('Пароль обязателен'),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password'), null], 'Пароли должны совпадать')
    .required('Подтверждение пароля обязательно')
});

const FormikForm = () => {
  const initialValues = {
    email: '',
    password: '',
    confirmPassword: ''
  };

  const onSubmit = async (values, { setSubmitting }) => {
    try {
      // Отправка данных
      console.log('Данные формы:', values);
      // Здесь может быть вызов API
    } catch (error) {
      console.error('Ошибка при отправке:', error);
    } finally {
      setSubmitting(false);
    }
  };

  return (
    <Formik
      initialValues={initialValues}
      validationSchema={validationSchema}
      onSubmit={onSubmit}
    >
      {({ isSubmitting }) => (
        <Form>
          <div>
            <Field 
              type="email" 
              name="email" 
              placeholder="Email" 
            />
            <ErrorMessage name="email" component="div" className="error" />
          </div>
          
          <div>
            <Field 
              type="password" 
              name="password" 
              placeholder="Password" 
            />
            <ErrorMessage name="password" component="div" className="error" />
          </div>
          
          <div>
            <Field 
              type="password" 
              name="confirmPassword" 
              placeholder="Confirm Password" 
            />
            <ErrorMessage name="confirmPassword" component="div" className="error" />
          </div>
          
          <button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Отправка...' : 'Отправить'}
          </button>
        </Form>
      )}
    </Formik>
  );
};

export default FormikForm;
```

## Валидация с React Hook Form

React Hook Form - современная библиотека для управления формами:

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
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
  age: yup
    .number()
    .positive('Возраст должен быть положительным')
    .integer('Возраст должен быть целым числом')
    .min(18, 'Возраст должен быть не менее 18')
    .max(120, 'Возраст должен быть не более 120')
    .required('Возраст обязателен')
});

const HookForm = () => {
  const { 
    register, 
    handleSubmit, 
    formState: { errors, isSubmitting },
    watch
  } = useForm({
    resolver: yupResolver(schema)
  });

  const onSubmit = async (data) => {
    console.log('Данные формы:', data);
    // Отправка данных
  };

  // Мониторинг значения поля для условной валидации
  const password = watch('password');

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input 
          {...register('email')} 
          placeholder="Email" 
        />
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>
      
      <div>
        <input 
          {...register('password')} 
          type="password" 
          placeholder="Password" 
        />
        {errors.password && <span className="error">{errors.password.message}</span>}
      </div>
      
      <div>
        <input 
          {...register('age')} 
          type="number" 
          placeholder="Age" 
        />
        {errors.age && <span className="error">{errors.age.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Отправка...' : 'Отправить'}
      </button>
    </form>
  );
};
```

## Условная валидация

```jsx
import React, { useState, useEffect } from 'react';
import { useForm } from 'react-hook-form';

const ConditionalValidationForm = () => {
  const { 
    register, 
    handleSubmit, 
    watch, 
    formState: { errors },
    trigger
  } = useForm();
  
  const [showAdditional, setShowAdditional] = useState(false);
  const accountType = watch('accountType');
  
  useEffect(() => {
    if (accountType === 'business') {
      setShowAdditional(true);
      // Запуск валидации для новых полей
      setTimeout(() => {
        trigger(['companyName', 'taxId']);
      }, 0);
    } else {
      setShowAdditional(false);
    }
  }, [accountType, trigger]);

  const onSubmit = (data) => {
    console.log('Данные формы:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <select {...register('accountType', { required: 'Тип аккаунта обязателен' })}>
          <option value="">Выберите тип аккаунта</option>
          <option value="personal">Личный</option>
          <option value="business">Бизнес</option>
        </select>
        {errors.accountType && <span>{errors.accountType.message}</span>}
      </div>
      
      {showAdditional && (
        <>
          <div>
            <input 
              {...register('companyName', { 
                required: showAdditional ? 'Название компании обязательно' : false 
              })} 
              placeholder="Название компании" 
            />
            {errors.companyName && <span>{errors.companyName.message}</span>}
          </div>
          
          <div>
            <input 
              {...register('taxId', { 
                required: showAdditional ? 'ИНН обязателен' : false,
                pattern: {
                  value: /^\d{10}$/,
                  message: 'ИНН должен содержать 10 цифр'
                }
              })} 
              placeholder="ИНН" 
            />
            {errors.taxId && <span>{errors.taxId.message}</span>}
          </div>
        </>
      )}
      
      <button type="submit">Отправить</button>
    </form>
  );
};
```

## Асинхронная валидация

```jsx
import React from 'react';
import { useForm } from 'react-hook-form';

// Асинхронная проверка уникальности email
const checkEmailUniqueness = async (email) => {
  // Имитация API-вызова
  return new Promise((resolve) => {
    setTimeout(() => {
      const isUnique = email !== 'existing@example.com';
      resolve(isUnique);
    }, 1000);
  });
};

const AsyncValidationForm = () => {
  const { 
    register, 
    handleSubmit, 
    formState: { errors, isSubmitting },
    setError,
    clearErrors
  } = useForm();
  
  const validateEmail = async (email) => {
    if (!email) return 'Email обязателен';
    if (!/\S+@\S+\.\S+/.test(email)) return 'Email некорректен';
    
    // Асинхронная проверка
    const isUnique = await checkEmailUniqueness(email);
    if (!isUnique) {
      return 'Email уже используется';
    }
    
    return '';
  };

  const onSubmit = async (data) => {
    console.log('Данные формы:', data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input 
          {...register('email', {
            validate: async (value) => await validateEmail(value)
          })} 
          placeholder="Email" 
        />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Проверка...' : 'Отправить'}
      </button>
    </form>
  );
};
```

## Лучшие практики валидации в React

- **Используйте подходящую библиотеку**: Formik для сложных форм, React Hook Form для производительности
- **Показывайте ошибки своевременно**: После потери фокуса или при отправке
- **Обеспечьте доступность**: Используйте ARIA-атрибуты для сообщений об ошибках
- **Предоставляйте понятные сообщения**: Пользователь должен понимать, как исправить ошибку
- **Валидируйте на сервере**: Клиентская валидация не заменяет серверную
- **Оптимизируйте производительность**: Избегайте лишних перерисовок при валидации

## Связанные темы

- [[Валидация-на-клиенте]]
- [[Валидация-на-сервере]]
- [[Валидация-в-Vue]]
- [[Валидация-в-Svelte]]
- [[React-Form-Management]]
- [[UX-валидация-форм]]

## Теги

#react #validation #frontend #forms #javascript #web-development #formik #react-hook-form
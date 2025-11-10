---
aliases: ["React Form Validation", "Валидация форм React"]
tags: ["react", "validation", "forms", "javascript", "frontend"]
---

# Валидация форм в React

## Обзор

Валидация форм в React — это важный аспект разработки пользовательского интерфейса, обеспечивающий корректность данных, вводимых пользователями. Правильная валидация улучшает пользовательский опыт и защищает приложение от некорректных данных.

## Клиентская против серверной валидации

**Клиентская валидация** выполняется на стороне пользователя и обеспечивает мгновенную обратную связь. Она не заменяет серверную валидацию, но улучшает UX и снижает нагрузку на сервер.

```jsx
const validateEmail = (email) => {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
};
```

**Серверная валидация** необходима для безопасности и надежности. Даже при наличии клиентской валидации все данные должны проверяться на сервере.

## Встроенные HTML валидации

HTML5 предоставляет встроенные атрибуты валидации:

- `required` — поле обязательно
- `min`, `max` — ограничения для чисел
- `pattern` — регулярные выражения
- `minLength`, `maxLength` — ограничения длины

```jsx
<input 
  type="email" 
  required 
  pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$" 
  placeholder="Email" 
/>
```

## Пользовательские функции валидации

Создание пользовательских функций позволяет реализовать сложную логику валидации:

```jsx
const validatePassword = (password) => {
  if (password.length < 8) {
    return "Пароль должен содержать не менее 8 символов";
  }
  if (!/[A-Z]/.test(password)) {
    return "Пароль должен содержать хотя бы одну заглавную букву";
  }
  if (!/[0-9]/.test(password)) {
    return "Пароль должен содержать хотя бы одну цифру";
  }
  return null;
};
```

## Библиотеки валидации

### Yup
Yup — популярная библиотека для валидации схем:

```jsx
import * as yup from 'yup';

const schema = yup.object({
  email: yup.string().email('Некорректный email').required('Email обязателен'),
  password: yup.string().min(8, 'Слишком короткий').required('Пароль обязателен'),
});
```

### Joi
Joi — мощная библиотека для валидации, разработанная hapi.js:

```jsx
import Joi from 'joi';

const schema = Joi.object({
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required(),
});
```

### Zod
Zod — современная библиотека валидации с поддержкой TypeScript:

```jsx
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Некорректный email'),
  password: z.string().min(8, 'Пароль должен быть не менее 8 символов'),
});
```

## Схемы валидации

Схемы позволяют определить структуру и правила валидации для всей формы:

```jsx
const userSchema = yup.object({
  name: yup.string().required().min(2),
  email: yup.string().email().required(),
  age: yup.number().positive().integer().min(18),
});
```

## Валидация в реальном времени

Валидация в реальном времени улучшает UX, но требует осторожного подхода:

```jsx
const [errors, setErrors] = useState({});

const handleInputChange = (e) => {
  const { name, value } = e.target;
  const error = validateField(name, value);
  setErrors(prev => ({ ...prev, [name]: error }));
};
```

## Валидация при отправке

Валидация при отправке формы — это основной контрольный момент:

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  
  try {
    await schema.validate(formData, { abortEarly: false });
    // Отправка данных
  } catch (err) {
    const validationErrors = err.inner.reduce((acc, error) => {
      acc[error.path] = error.message;
      return acc;
    }, {});
    setErrors(validationErrors);
  }
};
```

## Валидация с хуками

### Хук useFormValidation

Создание пользовательского хука для управления валидацией:

```jsx
import { useState } from 'react';

const useFormValidation = (initialValues, validationSchema) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Валидация только текущего поля
    try {
      validationSchema.validateSyncAt(name, { [name]: value });
      setErrors(prev => ({ ...prev, [name]: '' }));
    } catch (err) {
      setErrors(prev => ({ ...prev, [name]: err.message }));
    }
  };

  const handleSubmit = async (onSubmit) => {
    setIsSubmitting(true);
    try {
      await validationSchema.validate(values, { abortEarly: false });
      await onSubmit(values);
    } catch (err) {
      const newErrors = err.inner.reduce((acc, error) => {
        acc[error.path] = error.message;
        return acc;
      }, {});
      setErrors(newErrors);
    } finally {
      setIsSubmitting(false);
    }
  };

  return { values, errors, handleChange, handleSubmit, isSubmitting };
};
```

## Стратегии сообщений об ошибках

### Inline ошибки
Показывать ошибки сразу под соответствующим полем:

```jsx
{errors.email && <span className="error">{errors.email}</span>}
```

### Summary ошибки
Отображать все ошибки в верхней части формы:

```jsx
{Object.keys(errors).length > 0 && (
  <div className="error-summary">
    <h4>Пожалуйста, исправьте следующие ошибки:</h4>
    <ul>
      {Object.values(errors).map((error, i) => (
        <li key={i}>{error}</li>
      ))}
    </ul>
  </div>
)}
```

## Доступность в валидации

Для обеспечения доступности:

- Используйте `aria-describedby` для связи ошибок с полями
- Обеспечьте контрастность цветов
- Добавьте `role="alert"` для важных сообщений об ошибках
- Фокусируйтесь на первом невалидном поле при отправке

```jsx
<input 
  id="email"
  aria-describedby="email-error"
  aria-invalid={!!errors.email}
/>
{errors.email && (
  <span id="email-error" role="alert" className="error">
    {errors.email}
  </span>
)}
```

## Паттерны валидации для разных типов инпутов

### Email
```jsx
<input type="email" pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$" />
```

### Телефон
```jsx
<input 
  type="tel" 
  pattern="[0-9]{3}-[0-9]{3}-[0-9]{4}"
  placeholder="123-456-7890"
/>
```

### Пароль
```jsx
<input 
  type="password" 
  minLength="8"
  pattern="(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{8,}"
  title="Пароль должен содержать не менее 8 символов, одну заглавную букву и одну цифру"
/>
```

## Обработка ошибок форм

Комплексная обработка ошибок включает:

- Валидацию на клиенте и сервере
- Обработку сетевых ошибок
- Обработку ошибок бизнес-логики
- Уведомление пользователя об ошибках

## Производительность валидации

- Избегайте частой валидации больших форм
- Используйте `useCallback` для функций валидации
- Оптимизируйте сложные валидации
- Используйте debounce для валидации в реальном времени

## Интернационализация сообщений валидации

Для поддержки нескольких языков:

```jsx
const translations = {
  en: {
    required: 'This field is required',
    email: 'Invalid email format'
  },
  ru: {
    required: 'Это поле обязательно',
    email: 'Некорректный формат email'
  }
};

const getErrorMessage = (key, lang = 'ru') => translations[lang][key];
```

## Связи с другими файлами

- [[react/forms/basics]] — основы работы с формами в React
- [[react/hooks/useForm]] — пользовательский хук для управления формами
- [[react/accessibility]] — доступность в React приложениях
- [[javascript/validation]] — общие принципы валидации данных
- [[typescript/forms]] — использование TypeScript с формами
- [[css/forms]] — стилизация форм и сообщений об ошибках

## Заключение

Валидация форм в React требует сбалансированного подхода между UX и безопасностью. Правильное сочетание клиентской и серверной валидации, использование подходящих библиотек и соблюдение принципов доступности создает надежные и удобные формы.
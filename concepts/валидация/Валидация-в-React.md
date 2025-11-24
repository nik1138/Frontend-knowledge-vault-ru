---
aliases: ["Валидация форм в React", "React Form Validation"]
tags: ["#react", "#validation", "#forms", "#frontend"]
---

# Валидация в React

Валидация в React-приложениях обеспечивает корректность пользовательского ввода и улучшает пользовательский опыт. В React доступно несколько подходов к валидации: от простых встроенных проверок до сложных библиотек с продвинутыми возможностями.

## Основные подходы

### Встроенные проверки с хуками
```jsx
import { useState } from 'react';

function RegistrationForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  
  const validate = () => {
    const newErrors = {};
    
    if (!email) {
      newErrors.email = 'Email обязателен';
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      newErrors.email = 'Email некорректен';
    }
    
    if (!password) {
      newErrors.password = 'Пароль обязателен';
    } else if (password.length < 6) {
      newErrors.password = 'Пароль должен быть не менее 6 символов';
    }
    
    return newErrors;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }
    
    // Отправка данных
    console.log('Форма валидна', { email, password });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      
      <button type="submit">Зарегистрироваться</button>
    </form>
  );
}
```

## Использование библиотек

### React Hook Form
```jsx
import { useForm } from 'react-hook-form';
import { yupResolver } from '@hookform/resolvers/yup';
import * as yup from 'yup';

const schema = yup.object({
  email: yup.string().email('Некорректный email').required('Email обязателен'),
  password: yup.string().min(6, 'Пароль должен быть не менее 6 символов').required('Пароль обязателен'),
});

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: yupResolver(schema)
  });
  
  const onSubmit = (data) => console.log(data);
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} placeholder="Email" />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <input {...register('password')} type="password" placeholder="Password" />
        {errors.password && <span>{errors.password.message}</span>}
      </div>
      
      <button type="submit">Войти</button>
    </form>
  );
}
```

### Formik
```jsx
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const validationSchema = Yup.object({
  email: Yup.string()
    .email('Некорректный email')
    .required('Email обязателен'),
  password: Yup.string()
    .min(6, 'Пароль должен быть не менее 6 символов')
    .required('Пароль обязателен'),
});

function LoginForm() {
  return (
    <Formik
      initialValues={{ email: '', password: '' }}
      validationSchema={validationSchema}
      onSubmit={(values) => console.log(values)}
    >
      <Form>
        <Field name="email" type="email" placeholder="Email" />
        <ErrorMessage name="email" component="div" />
        
        <Field name="password" type="password" placeholder="Password" />
        <ErrorMessage name="password" component="div" />
        
        <button type="submit">Войти</button>
      </Form>
    </Formik>
  );
}
```

## Паттерны валидации

### Валидация в реальном времени
```jsx
function RealTimeValidation() {
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');
  
  useEffect(() => {
    if (email) {
      const isValid = /\S+@\S+\.\S+/.test(email);
      setEmailError(isValid ? '' : 'Некорректный email');
    }
  }, [email]);
  
  return (
    <input
      type="email"
      value={email}
      onChange={(e) => setEmail(e.target.value)}
      style={{ borderColor: emailError ? 'red' : 'initial' }}
    />
  );
}
```

### Валидация при потере фокуса
```jsx
function BlurValidation() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  
  const handleBlur = () => {
    if (email && !/\S+@\S+\.\S+/.test(email)) {
      setError('Некорректный email');
    }
  };
  
  return (
    <input
      type="email"
      value={email}
      onChange={(e) => setEmail(e.target.value)}
      onBlur={handleBlur}
      style={{ borderColor: error ? 'red' : 'initial' }}
    />
  );
}
```

## Лучшие практики

- **Используй библиотеки**: Для сложных форм используй специализированные библиотеки
- **Показывай ошибки сразу**: Не жди отправки формы для показа ошибок
- **Удобные сообщения об ошибках**: Делай сообщения понятными и полезными
- **Состояние загрузки**: Показывай индикатор загрузки при асинхронной валидации
- **Тестирование**: Покрой валидацию тестами

## Связанные темы
- [[Валидация-на-сервере]]
- [[Валидация-в-Vue]]
- [[Валидация-в-Svelte]]
- [[React-Form-Management]]
- [[UX-дизайн-форм]]

## Ключевые выводы
- Валидация в React может быть реализована разными способами в зависимости от сложности
- Библиотеки упрощают управление сложными формами
- Пользовательский опыт улучшается при своевременном показе ошибок
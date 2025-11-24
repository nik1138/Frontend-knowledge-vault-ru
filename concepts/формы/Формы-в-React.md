---
aliases: ["React Forms", "Формы в React", "Form Handling React"]
tags: ["#frontend", "#react", "#forms", "#state-management", "#validation"]
---

# Формы в React

Работа с формами в React имеет свои особенности и подходы, отличающиеся от традиционной работы с формами в чистом JavaScript. React предлагает несколько способов управления формами, каждый из которых имеет свои преимущества и недостатки.

## Контролируемые и неконтролируемые компоненты

### Контролируемые компоненты (Controlled Components)
В React состояние формы обычно управляется компонентом, а не DOM. Это означает, что значения полей формы определяются React-состоянием.

```jsx
import React, { useState } from 'react';

function MyForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Отправленные данные:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Имя:</label>
        <input
          type="text"
          id="name"
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
      </div>
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
      </div>
      <div>
        <label htmlFor="message">Сообщение:</label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
        />
      </div>
      <button type="submit">Отправить</button>
    </form>
  );
}
```

### Неконтролируемые компоненты (Uncontrolled Components)
Используют ref для получения значений из DOM напрямую, без управления состоянием через React.

```jsx
import React, { useRef } from 'react';

function MyUncontrolledForm() {
  const nameRef = useRef();
  const emailRef = useRef();

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Имя:', nameRef.current.value);
    console.log('Email:', emailRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} type="text" name="name" />
      <input ref={emailRef} type="email" name="email" />
      <button type="submit">Отправить</button>
    </form>
  );
}
```

## Управление состоянием формы

### Локальное состояние
Для простых форм можно использовать локальное состояние компонента:

```jsx
import React, { useState } from 'react';

function SimpleForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    // Обработка данных
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="text" 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <input 
        type="email" 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
      />
      <button type="submit">Отправить</button>
    </form>
  );
}
```

### Хуки для управления формами
React предоставляет встроенные хуки, но существуют и сторонние библиотеки для более сложного управления:

- [[React Hook Form]] - популярная библиотека для управления формами
- Formik - мощная библиотека для работы с формами
- Final Form - легковесная библиотека

## Валидация форм

### Встроенные возможности HTML5
React позволяет использовать встроенные атрибуты валидации HTML5:

```jsx
function ValidatedForm() {
  const [formData, setFormData] = useState({
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});

  const validate = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email обязателен';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email некорректен';
    }
    
    if (!formData.password) {
      newErrors.password = 'Пароль обязателен';
    } else if (formData.password.length < 6) {
      newErrors.password = 'Пароль должен быть не менее 6 символов';
    }
    
    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();
    if (Object.keys(newErrors).length === 0) {
      // Форма валидна, отправляем данные
      console.log('Форма отправлена:', formData);
    } else {
      setErrors(newErrors);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={(e) => setFormData({...formData, email: e.target.value})}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      <div>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={(e) => setFormData({...formData, password: e.target.value})}
          placeholder="Пароль"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      <button type="submit">Войти</button>
    </form>
  );
}
```

### Сторонние библиотеки валидации
- [[Yup]] - для валидации схем
- Joi - мощная библиотека валидации
- Validator.js - коллекция валидаторов

## Практические примеры

### Форма регистрации с валидацией

```jsx
import React, { useState } from 'react';

function RegistrationForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  });
  
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validate = () => {
    const newErrors = {};
    
    if (!formData.username) newErrors.username = 'Имя пользователя обязательно';
    if (!formData.email) newErrors.email = 'Email обязателен';
    else if (!/\S+@\S+\.\S+/.test(formData.email)) newErrors.email = 'Email некорректен';
    
    if (!formData.password) newErrors.password = 'Пароль обязателен';
    else if (formData.password.length < 6) newErrors.password = 'Пароль должен быть не менее 6 символов';
    
    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'Пароли не совпадают';
    }
    
    return newErrors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Очищаем ошибку при изменении поля
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: undefined
      }));
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    const newErrors = validate();
    
    if (Object.keys(newErrors).length === 0) {
      setIsSubmitting(true);
      try {
        // Отправка данных на сервер
        await new Promise(resolve => setTimeout(resolve, 1000)); // Имитация API вызова
        console.log('Регистрация успешна:', formData);
        // Сброс формы после успешной регистрации
        setFormData({
          username: '',
          email: '',
          password: '',
          confirmPassword: ''
        });
      } catch (error) {
        console.error('Ошибка регистрации:', error);
      } finally {
        setIsSubmitting(false);
      }
    } else {
      setErrors(newErrors);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="text"
          name="username"
          value={formData.username}
          onChange={handleChange}
          placeholder="Имя пользователя"
        />
        {errors.username && <span className="error">{errors.username}</span>}
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
          type="password"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleChange}
          placeholder="Подтверждение пароля"
        />
        {errors.confirmPassword && <span className="error">{errors.confirmPassword}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Регистрация...' : 'Зарегистрироваться'}
      </button>
    </form>
  );
}
```

## Лучшие практики

1. **Используйте контролируемые компоненты** для лучшего контроля над состоянием
2. **Обрабатывайте все события** изменения полей для актуальности состояния
3. **Валидируйте данные на клиенте** для улучшения UX
4. **Используйте TypeScript** для строгой типизации форм
5. **Разделяйте сложные формы** на несколько компонентов
6. **Обеспечивайте доступность** с помощью правильных меток и ARIA-атрибутов
7. **Предоставляйте пользовательские сообщения об ошибках**

## Заключение

Формы в React требуют тщательного подхода к управлению состоянием и валидации. Правильная реализация форм улучшает пользовательский опыт и предотвращает ошибки ввода данных. Выбор между контролируемыми и неконтролируемыми компонентами зависит от сложности формы и требований приложения.

См. также: [[Управление-состоянием-форм]], [[Формы-в-Vue]], [[Формы-в-Svelte]]
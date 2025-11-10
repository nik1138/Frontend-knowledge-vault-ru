---
aliases: ["React формы", "Формы в React", "Управляемые компоненты", "Валидация форм"]
tags: 
  - react
  - forms
  - javascript
  - frontend
  - validation
  - accessibility
---

# Формы в React: Полное руководство

Формы являются ключевым элементом взаимодействия пользователей с веб-приложениями. В React существует несколько подходов к созданию и управлению формами, каждый из которых имеет свои особенности и области применения.

## Управляемые и неуправляемые компоненты

### Управляемые компоненты

Управляемые компоненты — это компоненты, значения которых контролируются React. Состояние формы хранится в `state` компонента, и обновляется через обработчики событий.

```jsx
import React, { useState } from 'react';

const ControlledForm = () => {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ name, email });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Имя"
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <button type="submit">Отправить</button>
    </form>
  );
};
```

### Неуправляемые компоненты

Неуправляемые компоненты полагаются на DOM для хранения значения. Для доступа к значениям используется `ref`.

```jsx
import React, { useRef } from 'react';

const UncontrolledForm = () => {
  const nameRef = useRef();
  const emailRef = useRef();

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({
      name: nameRef.current.value,
      email: emailRef.current.value
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} type="text" placeholder="Имя" />
      <input ref={emailRef} type="email" placeholder="Email" />
      <button type="submit">Отправить</button>
    </form>
  );
};
```

## Обработка отправки формы

Обработка отправки формы в React требует предотвращения стандартного поведения браузера с помощью `e.preventDefault()`.

```jsx
const FormSubmission = () => {
  const [formData, setFormData] = useState({ name: '', email: '' });

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      // Отправка данных на сервер
      const response = await fetch('/api/submit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      if (response.ok) {
        console.log('Форма успешно отправлена');
        setFormData({ name: '', email: '' }); // Сброс формы
      } else {
        console.error('Ошибка при отправке формы');
      }
    } catch (error) {
      console.error('Ошибка сети:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Поля формы */}
    </form>
  );
};
```

## Валидация форм

Валидация может происходить как на стороне клиента, так и на сервере. Рассмотрим клиентскую валидацию:

```jsx
const validateForm = (data) => {
  const errors = {};
  
  if (!data.name.trim()) {
    errors.name = 'Имя обязательно';
  }
  
  if (!data.email) {
    errors.email = 'Email обязателен';
  } else if (!/\S+@\S+\.\S+/.test(data.email)) {
    errors.email = 'Email недействителен';
  }
  
  return errors;
};

const ValidatedForm = () => {
  const [formData, setFormData] = useState({ name: '', email: '' });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Валидация в реальном времени
    const newErrors = validateForm({ ...formData, [name]: value });
    setErrors(newErrors);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const validationErrors = validateForm(formData);
    setErrors(validationErrors);
    
    if (Object.keys(validationErrors).length === 0) {
      // Отправка формы
      console.log('Форма валидна:', formData);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Имя"
      />
      {errors.name && <span className="error">{errors.name}</span>}
      
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      {errors.email && <span className="error">{errors.email}</span>}
      
      <button type="submit">Отправить</button>
    </form>
  );
};
```

## Общие паттерны форм

### Паттерн "изменяемая форма"

```jsx
const EditableForm = ({ initialData, onSave }) => {
  const [formData, setFormData] = useState(initialData);
  const [isEditing, setIsEditing] = useState(false);

  const handleSave = () => {
    onSave(formData);
    setIsEditing(false);
  };

  if (!isEditing) {
    return (
      <div>
        <p>Имя: {initialData.name}</p>
        <p>Email: {initialData.email}</p>
        <button onClick={() => setIsEditing(true)}>Редактировать</button>
      </div>
    );
  }

  return (
    <form onSubmit={(e) => { e.preventDefault(); handleSave(); }}>
      <input
        type="text"
        value={formData.name}
        onChange={(e) => setFormData({...formData, name: e.target.value})}
      />
      <input
        type="email"
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
      />
      <button type="submit">Сохранить</button>
      <button type="button" onClick={() => setIsEditing(false)}>Отмена</button>
    </form>
  );
};
```

## Обработка пользовательского ввода

Для обработки ввода в различных типах полей можно использовать универсальный обработчик:

```jsx
const UniversalForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    age: 0,
    bio: '',
    subscribed: false
  });

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  return (
    <form>
      <input name="name" value={formData.name} onChange={handleChange} />
      <input name="email" type="email" value={formData.email} onChange={handleChange} />
      <input name="age" type="number" value={formData.age} onChange={handleChange} />
      <textarea name="bio" value={formData.bio} onChange={handleChange} />
      <input name="subscribed" type="checkbox" checked={formData.subscribed} onChange={handleChange} />
    </form>
  );
};
```

## Библиотеки для работы с формами

### Formik

Formik — популярная библиотека для управления формами в React:

```jsx
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

const validationSchema = Yup.object({
  email: Yup.string().email('Неверный email').required('Обязательное поле'),
  password: Yup.string().min(8, 'Минимум 8 символов').required('Обязательное поле')
});

const FormikForm = () => {
  return (
    <Formik
      initialValues={{ email: '', password: '' }}
      validationSchema={validationSchema}
      onSubmit={(values) => console.log(values)}
    >
      <Form>
        <Field name="email" type="email" placeholder="Email" />
        <ErrorMessage name="email" component="div" />
        
        <Field name="password" type="password" placeholder="Пароль" />
        <ErrorMessage name="password" component="div" />
        
        <button type="submit">Войти</button>
      </Form>
    </Formik>
  );
};
```

### React Hook Form

React Hook Form предоставляет легковесное решение для управления формами:

```jsx
import { useForm } from 'react-hook-form';

const HookForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input 
        {...register("email", { 
          required: "Email обязателен",
          pattern: {
            value: /\S+@\S+\.\S+/,
            message: "Неверный email"
          }
        })} 
        placeholder="Email" 
      />
      {errors.email && <span>{errors.email.message}</span>}
      
      <button type="submit">Отправить</button>
    </form>
  );
};
```

## Доступность (accessibility)

При создании форм необходимо учитывать требования доступности:

- Использовать семантические теги (`<form>`, `<label>`, `<fieldset>`)
- Привязывать `<label>` к полям ввода
- Обеспечивать визуальную индикацию фокуса
- Правильно обрабатывать ошибки

```jsx
const AccessibleForm = () => {
  const [formData, setFormData] = useState({ name: '', email: '' });
  const [errors, setErrors] = useState({});

  return (
    <form>
      <fieldset>
        <legend>Контактная информация</legend>
        
        <div>
          <label htmlFor="name">Имя:</label>
          <input 
            id="name"
            name="name"
            value={formData.name}
            onChange={(e) => setFormData({...formData, name: e.target.value})}
            aria-invalid={!!errors.name}
            aria-describedby={errors.name ? "name-error" : undefined}
          />
          {errors.name && (
            <span id="name-error" role="alert">{errors.name}</span>
          )}
        </div>
        
        <div>
          <label htmlFor="email">Email:</label>
          <input 
            id="email"
            name="email"
            type="email"
            value={formData.email}
            onChange={(e) => setFormData({...formData, email: e.target.value})}
            aria-invalid={!!errors.email}
            aria-describedby={errors.email ? "email-error" : undefined}
          />
          {errors.email && (
            <span id="email-error" role="alert">{errors.email}</span>
          )}
        </div>
      </fieldset>
    </form>
  );
};
```

## Безопасность

При работе с формами важно учитывать следующие аспекты безопасности:

- Валидация на стороне сервера
- Защита от XSS-атак
- Ограничение длины полей
- Санитизация ввода

## Управление состоянием формы

Для сложных форм можно использовать пользовательские хуки:

```jsx
const useFormState = (initialValues) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
    
    if (touched[name]) {
      // Повторная валидация при изменении
      const validationError = validateField(name, value);
      setErrors(prev => ({ ...prev, [name]: validationError }));
    }
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    const validationError = validateField(name, values[name]);
    setErrors(prev => ({ ...prev, [name]: validationError }));
  };

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur
  };
};
```

## Загрузка файлов

Для загрузки файлов можно использовать `FileReader` или отправку через FormData:

```jsx
const FileUploadForm = () => {
  const [file, setFile] = useState(null);
  const [preview, setPreview] = useState('');

  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    setFile(selectedFile);

    if (selectedFile) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setPreview(reader.result);
      };
      reader.readAsDataURL(selectedFile);
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData
      });
      
      if (response.ok) {
        console.log('Файл загружен');
      }
    } catch (error) {
      console.error('Ошибка загрузки:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" onChange={handleFileChange} />
      {preview && <img src={preview} alt="Превью" />}
      <button type="submit">Загрузить</button>
    </form>
  );
};
```

## Условные поля

Для динамического отображения полей:

```jsx
const ConditionalForm = () => {
  const [userType, setUserType] = useState('individual');
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    company: '',
    taxId: ''
  });

  return (
    <form>
      <select value={userType} onChange={(e) => setUserType(e.target.value)}>
        <option value="individual">Физическое лицо</option>
        <option value="company">Компания</option>
      </select>
      
      <input
        name="name"
        value={formData.name}
        onChange={(e) => setFormData({...formData, name: e.target.value})}
        placeholder="Имя"
      />
      
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={(e) => setFormData({...formData, email: e.target.value})}
        placeholder="Email"
      />
      
      {userType === 'company' && (
        <>
          <input
            name="company"
            value={formData.company}
            onChange={(e) => setFormData({...formData, company: e.target.value})}
            placeholder="Компания"
          />
          <input
            name="taxId"
            value={formData.taxId}
            onChange={(e) => setFormData({...formData, taxId: e.target.value})}
            placeholder="ИНН"
          />
        </>
      )}
    </form>
  );
};
```

## Тестирование форм

При тестировании форм нужно проверять:

- Ввод данных
- Валидацию
- Отправку
- Обработку ошибок

```jsx
// Пример с React Testing Library
import { render, screen, fireEvent, waitFor } from '@testing-library/react';

test('отправка формы с валидными данными', async () => {
  render(<ValidatedForm />);
  
  fireEvent.change(screen.getByPlaceholderText('Имя'), { target: { value: 'Иван' } });
  fireEvent.change(screen.getByPlaceholderText('Email'), { target: { value: 'test@example.com' } });
  
  fireEvent.click(screen.getByRole('button', { name: /отправить/i }));
  
  await waitFor(() => {
    expect(console.log).toHaveBeenCalledWith('Форма валидна:', { name: 'Иван', email: 'test@example.com' });
  });
});
```

## Производительность

Для оптимизации производительности форм:

- Использовать `React.memo` для компонентов полей
- Избегать ненужных перерендеров
- Использовать `useCallback` для обработчиков
- Оптимизировать валидацию

## Связи с другими файлами

- [[react/state-management]] - для понимания управления состоянием в React
- [[react/hooks]] - для работы с хуками, используемых в формах
- [[react/components]] - для понимания архитектуры компонентов
- [[javascript/validation]] - для глубокого понимания валидации данных
- [[css/styling]] - для стилизации форм
- [[accessibility/aria]] - для обеспечения доступности форм

> [!tip] Совет
> Используйте библиотеки форм для сложных сценариев, а встроенные возможности React для простых форм.

> [!warning] Важно
> Всегда проводите валидацию как на клиенте, так и на сервере для обеспечения безопасности.
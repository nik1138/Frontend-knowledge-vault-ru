---
aliases: ["Работа с формами в React с TypeScript", "Forms in React with TypeScript"]
tags: ["#typescript", "#react", "#forms", "#state-management"]
---

# Формы-в-React

Работа с формами в React с использованием TypeScript требует особого внимания к типизации и управлению состоянием. React предоставляет несколько подходов для работы с формами, каждый из которых имеет свои особенности и преимущества.

## Подходы к управлению формами в React

### Управляемые компоненты (Controlled Components)

В управляемых компонентах состояние формы хранится в состоянии компонента, а значения элементов управления синхронизируются с этим состоянием через обработчики событий.

```typescript
import React, { useState } from 'react';

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
  subscribe: boolean;
}

const ContactForm: React.FC = () => {
  const [formData, setFormData] = useState<ContactFormValues>({
    name: '',
    email: '',
    message: '',
    subscribe: false
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value, type } = e.target;
    const val = type === 'checkbox' ? (e.target as HTMLInputElement).checked : value;
    
    setFormData(prev => ({
      ...prev,
      [name]: val
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Данные формы:', formData);
    // Отправка данных на сервер
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
      
      <div>
        <label>
          <input
            type="checkbox"
            name="subscribe"
            checked={formData.subscribe}
            onChange={handleChange}
          />
          Подписаться на рассылку
        </label>
      </div>
      
      <button type="submit">Отправить</button>
    </form>
  );
};
```

### Неуправляемые компоненты (Uncontrolled Components)

В неуправляемых компонентах состояние формы хранится в DOM, а доступ к значениям осуществляется через рефы.

```typescript
import React, { useRef } from 'react';

const UncontrolledContactForm: React.FC = () => {
  const nameRef = useRef<HTMLInputElement>(null);
  const emailRef = useRef<HTMLInputElement>(null);
  const messageRef = useRef<HTMLTextAreaElement>(null);
  const subscribeRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (nameRef.current && emailRef.current && messageRef.current && subscribeRef.current) {
      const formData = {
        name: nameRef.current.value,
        email: emailRef.current.value,
        message: messageRef.current.value,
        subscribe: subscribeRef.current.checked
      };
      
      console.log('Данные формы:', formData);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Имя:</label>
        <input type="text" id="name" ref={nameRef} />
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input type="email" id="email" ref={emailRef} />
      </div>
      
      <div>
        <label htmlFor="message">Сообщение:</label>
        <textarea id="message" ref={messageRef} />
      </div>
      
      <div>
        <label>
          <input type="checkbox" ref={subscribeRef} />
          Подписаться на рассылку
        </label>
      </div>
      
      <button type="submit">Отправить</button>
    </form>
  );
};
```

## Популярные библиотеки для работы с формами

### React Hook Form

React Hook Form — это мощная библиотека для работы с формами, которая обеспечивает легкую валидацию и минимальный ре-рендеринг.

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const contactSchema = z.object({
  name: z.string().min(1, 'Имя обязательно'),
  email: z.string().email('Некорректный email'),
  message: z.string().min(10, 'Сообщение должно содержать не менее 10 символов'),
  subscribe: z.boolean()
});

type ContactFormData = z.infer<typeof contactSchema>;

const ReactHookFormExample: React.FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    reset
  } = useForm<ContactFormData>({
    resolver: zodResolver(contactSchema),
    defaultValues: {
      name: '',
      email: '',
      message: '',
      subscribe: false
    }
  });

  const onSubmit = async (data: ContactFormData) => {
    console.log('Данные формы:', data);
    // Отправка данных на сервер
    await new Promise(resolve => setTimeout(resolve, 1000)); // Имитация отправки
    reset(); // Сброс формы после успешной отправки
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="name">Имя:</label>
        <input
          id="name"
          {...register('name')}
          className={errors.name ? 'error' : ''}
        />
        {errors.name && <span className="error">{errors.name.message}</span>}
      </div>
      
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          {...register('email')}
          className={errors.email ? 'error' : ''}
        />
        {errors.email && <span className="error">{errors.email.message}</span>}
      </div>
      
      <div>
        <label htmlFor="message">Сообщение:</label>
        <textarea
          id="message"
          {...register('message')}
          className={errors.message ? 'error' : ''}
        />
        {errors.message && <span className="error">{errors.message.message}</span>}
      </div>
      
      <div>
        <label>
          <input type="checkbox" {...register('subscribe')} />
          Подписаться на рассылку
        </label>
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Отправка...' : 'Отправить'}
      </button>
    </form>
  );
};
```

### Formik

Formik — еще одна популярная библиотека для управления формами в React.

```typescript
import { Formik, Form, Field, ErrorMessage } from 'formik';
import * as Yup from 'yup';

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
  subscribe: boolean;
}

const contactValidationSchema = Yup.object({
  name: Yup.string().required('Имя обязательно'),
  email: Yup.string().email('Некорректный email').required('Email обязателен'),
  message: Yup.string().min(10, 'Сообщение должно содержать не менее 10 символов').required('Сообщение обязательно'),
  subscribe: Yup.boolean()
});

const FormikExample: React.FC = () => {
  const initialValues: ContactFormValues = {
    name: '',
    email: '',
    message: '',
    subscribe: false
  };

  const onSubmit = async (values: ContactFormValues, { setSubmitting, resetForm }: any) => {
    console.log('Данные формы:', values);
    // Отправка данных на сервер
    await new Promise(resolve => setTimeout(resolve, 1000)); // Имитация отправки
    setSubmitting(false);
    resetForm();
  };

  return (
    <Formik
      initialValues={initialValues}
      validationSchema={contactValidationSchema}
      onSubmit={onSubmit}
    >
      {({ isSubmitting }) => (
        <Form>
          <div>
            <label htmlFor="name">Имя:</label>
            <Field type="text" id="name" name="name" />
            <ErrorMessage name="name" component="div" className="error" />
          </div>
          
          <div>
            <label htmlFor="email">Email:</label>
            <Field type="email" id="email" name="email" />
            <ErrorMessage name="email" component="div" className="error" />
          </div>
          
          <div>
            <label htmlFor="message">Сообщение:</label>
            <Field as="textarea" id="message" name="message" />
            <ErrorMessage name="message" component="div" className="error" />
          </div>
          
          <div>
            <label>
              <Field type="checkbox" name="subscribe" />
              Подписаться на рассылку
            </label>
          </div>
          
          <button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Отправка...' : 'Отправить'}
          </button>
        </Form>
      )}
    </Formik>
  );
};
```

## Пользовательские хуки для форм

Для повторного использования логики форм можно создать пользовательские хуки:

```typescript
import { useState, useCallback } from 'react';

interface UseFormReturn<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  handleChange: (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => void;
  handleSubmit: (callback: (values: T) => void) => (e: React.FormEvent) => void;
  setFieldValue: (name: keyof T, value: any) => void;
  setFieldError: (name: keyof T, error: string) => void;
}

export const useForm = <T extends Record<string, any>>(initialValues: T): UseFormReturn<T> => {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});

  const handleChange = useCallback((e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => {
    const { name, value, type } = e.target;
    const val = type === 'checkbox' ? (e.target as HTMLInputElement).checked : value;
    
    setValues(prev => ({
      ...prev,
      [name]: val
    }));
    
    // Очистка ошибки при изменении поля
    if (errors[name as keyof T]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name as keyof T];
        return newErrors;
      });
    }
  }, [errors]);

  const setFieldValue = useCallback((name: keyof T, value: any) => {
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
    
    // Очистка ошибки при изменении поля
    if (errors[name]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
    }
  }, [errors]);

  const setFieldError = useCallback((name: keyof T, error: string) => {
    setErrors(prev => ({
      ...prev,
      [name]: error
    }));
  }, []);

  const handleSubmit = useCallback((callback: (values: T) => void) => {
    return (e: React.FormEvent) => {
      e.preventDefault();
      callback(values);
    };
  }, [values]);

  return {
    values,
    errors,
    handleChange,
    handleSubmit,
    setFieldValue,
    setFieldError
  };
};
```

## Лучшие практики

- Используйте управляемые компоненты для сложных форм
- Применяйте строгую типизацию для значений форм
- Используйте валидацию на стороне клиента для улучшения UX
- Обрабатывайте различные состояния формы (загрузка, ошибка)
- Используйте библиотеки для сложных сценариев работы с формами
- Обеспечьте доступность форм (accessibility)

## Связанные темы

- [[Управление-формами]]
- [[Валидация-форм]]
- [[Формы-в-Vue]]
- [[Формы-в-Angular]]
- [[React-с-typescript]]
---
aliases: [Медиатор React, Паттерн медиатор в React, Реакт медиатор]
tags: [react, паттерны-проектирования, медиатор, фронтенд]
---

# Реализация паттерна Медиатор в React

## Обзор

В React паттерн Медиатор можно реализовать несколькими способами: через контекст, пользовательские хуки, или централизованный обработчик событий. Это позволяет компонентам взаимодействовать друг с другом без прямой зависимости, что особенно полезно в сложных приложениях с множеством взаимодействующих компонентов.

## Реализация через Context API

### Простой медиатор с использованием Context

```jsx
import React, { createContext, useContext, useReducer, useCallback } from 'react';

// Создание контекста медиатора
const MediatorContext = createContext();

// Редьюсер для управления состоянием
const mediatorReducer = (state, action) => {
  switch (action.type) {
    case 'USER_LOGIN':
      return {
        ...state,
        user: action.payload,
        isAuthenticated: true
      };
    case 'USER_LOGOUT':
      return {
        ...state,
        user: null,
        isAuthenticated: false
      };
    case 'SHOW_MODAL':
      return {
        ...state,
        modal: {
          ...state.modal,
          [action.payload.id]: action.payload
        }
      };
    case 'HIDE_MODAL':
      return {
        ...state,
        modal: {
          ...state.modal,
          [action.payload.id]: null
        }
      };
    default:
      return state;
  }
};

// Провайдер медиатора
export const MediatorProvider = ({ children }) => {
  const [state, dispatch] = useReducer(mediatorReducer, {
    user: null,
    isAuthenticated: false,
    modal: {}
  });

  // Методы для взаимодействия с медиатором
  const actions = {
    login: useCallback((userData) => {
      dispatch({ type: 'USER_LOGIN', payload: userData });
    }, []),

    logout: useCallback(() => {
      dispatch({ type: 'USER_LOGOUT' });
    }, []),

    showModal: useCallback((modalData) => {
      dispatch({ type: 'SHOW_MODAL', payload: modalData });
    }, []),

    hideModal: useCallback((modalId) => {
      dispatch({ type: 'HIDE_MODAL', payload: { id: modalId } });
    }, [])
  };

  return (
    <MediatorContext.Provider value={{ state, actions }}>
      {children}
    </MediatorContext.Provider>
  );
};

// Хук для доступа к медиатору
export const useMediator = () => {
  const context = useContext(MediatorContext);
  if (!context) {
    throw new Error('useMediator must be used within a MediatorProvider');
  }
  return context;
};

// Компоненты, использующие медиатор
const LoginForm = () => {
  const { actions } = useMediator();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const userData = {
      username: formData.get('username'),
      email: formData.get('email')
    };
    
    actions.login(userData);
    actions.showModal({
      id: 'welcome',
      title: 'Добро пожаловать!',
      content: `Здравствуйте, ${userData.username}!`
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="username" placeholder="Имя пользователя" required />
      <input name="email" placeholder="Email" required />
      <button type="submit">Войти</button>
    </form>
  );
};

const UserProfile = () => {
  const { state } = useMediator();
  
  if (!state.isAuthenticated) {
    return <div>Пожалуйста, войдите в систему</div>;
  }

  return (
    <div>
      <h2>Профиль пользователя</h2>
      <p>Имя: {state.user.username}</p>
      <p>Email: {state.user.email}</p>
    </div>
  );
};

const Modal = ({ id }) => {
  const { state, actions } = useMediator();
  const modalData = state.modal[id];

  if (!modalData) return null;

  return (
    <div className="modal">
      <div className="modal-content">
        <h3>{modalData.title}</h3>
        <p>{modalData.content}</p>
        <button onClick={() => actions.hideModal(id)}>Закрыть</button>
      </div>
    </div>
  );
};

// Использование в приложении
const App = () => {
  return (
    <MediatorProvider>
      <div>
        <LoginForm />
        <UserProfile />
        <Modal id="welcome" />
      </div>
    </MediatorProvider>
  );
};

export default App;
```

## Реализация через пользовательский хук

### Медиатор как пользовательский хук

```jsx
import { useState, useCallback, useEffect } from 'react';

// Пользовательский хук медиатора
export const useMediator = () => {
  const [state, setState] = useState({
    events: {},
    subscribers: {}
  });

  // Подписка на события
  const subscribe = useCallback((eventName, callback) => {
    setState(prev => ({
      ...prev,
      subscribers: {
        ...prev.subscribers,
        [eventName]: [...(prev.subscribers[eventName] || []), callback]
      }
    }));

    // Возврат функции отписки
    return () => {
      setState(prev => ({
        ...prev,
        subscribers: {
          ...prev.subscribers,
          [eventName]: (prev.subscribers[eventName] || []).filter(cb => cb !== callback)
        }
      }));
    };
  }, []);

  // Публикация события
  const publish = useCallback((eventName, data) => {
    const subscribers = state.subscribers[eventName] || [];
    subscribers.forEach(callback => callback(data));
  }, [state.subscribers]);

  // Очистка при размонтировании
  useEffect(() => {
    return () => {
      // Очистка подписок при размонтировании компонента
    };
  }, []);

  return { subscribe, publish };
};

// Пример использования в компонентах
const ComponentA = () => {
  const { publish } = useMediator();

  const handleClick = () => {
    publish('button:click', { source: 'ComponentA', timestamp: Date.now() });
  };

  return (
    <div>
      <button onClick={handleClick}>Кнопка из ComponentA</button>
    </div>
  );
};

const ComponentB = () => {
  const { subscribe } = useMediator();

  useEffect(() => {
    const unsubscribe = subscribe('button:click', (data) => {
      console.log('ComponentB получил событие:', data);
    });

    // Отписка при размонтировании
    return unsubscribe;
  }, [subscribe]);

  return <div>ComponentB слушает события</div>;
};

const ComponentC = () => {
  const { subscribe } = useMediator();

  useEffect(() => {
    const unsubscribe = subscribe('button:click', (data) => {
      console.log('ComponentC получил событие:', data);
    });

    // Отписка при размонтировании
    return unsubscribe;
  }, [subscribe]);

  return <div>ComponentC тоже слушает события</div>;
};
```

## Пример сложного взаимодействия компонентов

### Управление формой с валидацией через медиатор

```jsx
import React, { createContext, useContext, useReducer, useCallback } from 'react';

// Контекст формы
const FormContext = createContext();

// Редьюсер формы
const formReducer = (state, action) => {
  switch (action.type) {
    case 'FIELD_UPDATE':
      return {
        ...state,
        fields: {
          ...state.fields,
          [action.payload.name]: {
            ...state.fields[action.payload.name],
            value: action.payload.value,
            isValid: action.payload.isValid,
            error: action.payload.error
          }
        },
        isValid: Object.values({
          ...state.fields,
          [action.payload.name]: {
            ...state.fields[action.payload.name],
            value: action.payload.value,
            isValid: action.payload.isValid,
            error: action.payload.error
          }
        }).every(field => field.isValid)
      };
    case 'FORM_SUBMIT':
      return {
        ...state,
        isSubmitting: true
      };
    case 'FORM_SUBMIT_SUCCESS':
      return {
        ...state,
        isSubmitting: false,
        isSubmitted: true
      };
    case 'FORM_SUBMIT_ERROR':
      return {
        ...state,
        isSubmitting: false,
        error: action.payload
      };
    default:
      return state;
  }
};

// Провайдер формы
export const FormProvider = ({ children, initialFields = {} }) => {
  const [state, dispatch] = useReducer(formReducer, {
    fields: initialFields,
    isValid: false,
    isSubmitting: false,
    isSubmitted: false,
    error: null
  });

  const updateField = useCallback((name, value, validator) => {
    const isValid = validator ? validator(value) : true;
    const error = isValid ? null : 'Поле заполнено неправильно';
    
    dispatch({
      type: 'FIELD_UPDATE',
      payload: { name, value, isValid, error }
    });
  }, []);

  const submitForm = useCallback(async (onSubmit) => {
    dispatch({ type: 'FORM_SUBMIT' });
    
    try {
      const formData = Object.keys(state.fields).reduce((acc, key) => {
        acc[key] = state.fields[key].value;
        return acc;
      }, {});
      
      await onSubmit(formData);
      dispatch({ type: 'FORM_SUBMIT_SUCCESS' });
    } catch (error) {
      dispatch({ type: 'FORM_SUBMIT_ERROR', payload: error.message });
    }
  }, [state.fields]);

  return (
    <FormContext.Provider value={{ state, updateField, submitForm }}>
      {children}
    </FormContext.Provider>
  );
};

// Хук для работы с формой
export const useForm = () => {
  const context = useContext(FormContext);
  if (!context) {
    throw new Error('useForm must be used within a FormProvider');
  }
  return context;
};

// Компонент поля ввода
const InputField = ({ name, label, validator, type = 'text' }) => {
  const { state, updateField } = useForm();
  const field = state.fields[name] || { value: '', isValid: true, error: null };

  const handleChange = (e) => {
    const value = e.target.value;
    updateField(name, value, validator);
  };

  return (
    <div className="form-field">
      <label htmlFor={name}>{label}</label>
      <input
        id={name}
        name={name}
        type={type}
        value={field.value}
        onChange={handleChange}
        className={field.isValid ? '' : 'error'}
      />
      {field.error && <span className="error-message">{field.error}</span>}
    </div>
  );
};

// Кнопка отправки формы
const SubmitButton = ({ onSubmit, children }) => {
  const { state, submitForm } = useForm();
  
  const handleClick = () => {
    submitForm(onSubmit);
  };

  return (
    <button 
      onClick={handleClick} 
      disabled={!state.isValid || state.isSubmitting}
    >
      {state.isSubmitting ? 'Отправка...' : children}
    </button>
  );
};

// Статус формы
const FormStatus = () => {
  const { state } = useForm();
  
  if (state.isSubmitted) {
    return <div className="form-success">Форма успешно отправлена!</div>;
  }
  
  if (state.error) {
    return <div className="form-error">Ошибка: {state.error}</div>;
  }
  
  return null;
};

// Использование
const MyForm = () => {
  const initialFields = {
    name: { value: '', isValid: false, error: null },
    email: { value: '', isValid: false, error: null },
    password: { value: '', isValid: false, error: null }
  };

  const handleSubmit = async (data) => {
    // Отправка данных формы
    console.log('Отправка данных:', data);
    // Симуляция асинхронной операции
    await new Promise(resolve => setTimeout(resolve, 1000));
  };

  const validateEmail = (email) => {
    const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return re.test(email);
  };

  const validatePassword = (password) => {
    return password.length >= 6;
  };

  return (
    <FormProvider initialFields={initialFields}>
      <form>
        <InputField 
          name="name" 
          label="Имя" 
          validator={(value) => value.length > 0} 
        />
        <InputField 
          name="email" 
          label="Email" 
          validator={validateEmail} 
        />
        <InputField 
          name="password" 
          label="Пароль" 
          type="password" 
          validator={validatePassword} 
        />
        <SubmitButton onSubmit={handleSubmit}>
          Отправить
        </SubmitButton>
        <FormStatus />
      </form>
    </FormProvider>
  );
};
```

## Преимущества использования медиатора в React

- **Снижение связанности**: Компоненты не зависят напрямую друг от друга.
- **Централизованное управление состоянием**: Логика взаимодействия сосредоточена в одном месте.
- **Упрощение тестирования**: Компоненты легче тестировать изолированно.

## Возможные проблемы

- **Сложность архитектуры**: При неправильном использовании может усложнить понимание потока данных.
- **Переиспользование**: Компоненты, зависящие от медиатора, могут быть менее переиспользуемыми.

## Заключение

Паттерн Медиатор в React позволяет создать гибкую архитектуру с централизованным управлением взаимодействиями между компонентами. Он особенно полезен в сложных приложениях с множеством взаимодействующих компонентов.

Для изучения реализации в других технологиях см.:
- [[Реализация-в-JavaScript]]
- [[Реализация-в-Vue]]

Для практических примеров см. [[Примеры-использования]].

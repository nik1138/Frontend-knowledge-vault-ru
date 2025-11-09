# Типизация форм в React с TypeScript

Типизация форм в React с TypeScript позволяет создавать строго типизированные и безопасные формы с валидацией, управлением состоянием и обработкой событий. Эта секция охватывает все аспекты типизации форм в React-приложениях.

## Содержание

1. [Базовая типизация форм](#базовая-типизация-форм)
2. [Управление состоянием форм](#управление-состоянием-форм)
3. [Валидация форм](#валидация-форм)
4. [Типизация событий форм](#типизация-событий-форм)
5. [Работа с различными типами полей](#работа-с-различными-типами-полей)
6. [Интеграция с библиотеками форм](#интеграция-с-библиотеками-форм)
7. [Пользовательские хуки для форм](#пользовательские-хуки-для-форм)
8. [Лучшие практики](#лучшие-практики)
9. [Распространенные ошибки](#распространенные-ошибки)

## Базовая типизация форм

### Интерфейсы для форм

```tsx
// Базовый интерфейс для формы
interface LoginFormValues {
  email: string;
  password: string;
  rememberMe: boolean;
}

// Интерфейс с опциональными полями
interface RegistrationFormValues {
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  confirmPassword: string;
  age?: number;
  termsAccepted: boolean;
}

// Интерфейс с вложенными объектами
interface UserProfileFormValues {
  personalInfo: {
    firstName: string;
    lastName: string;
    dateOfBirth: string;
  };
  contactInfo: {
    email: string;
    phone?: string;
    address: {
      street: string;
      city: string;
      country: string;
    };
  };
  preferences: {
    newsletter: boolean;
    notifications: boolean;
  };
}
```

### Простая форма с типизацией

```tsx
// Интерфейс для значений формы
interface ContactFormValues {
  name: string;
  email: string;
  subject: string;
  message: string;
}

// Компонент формы
const ContactForm: React.FC = () => {
  const [values, setValues] = useState<ContactFormValues>({
    name: '',
    email: '',
    subject: '',
    message: ''
  });

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Form submitted:', values);
    // Логика отправки формы
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name</label>
        <input
          type="text"
          id="name"
          name="name"
          value={values.name}
          onChange={handleChange}
          required
        />
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          required
        />
      </div>

      <div>
        <label htmlFor="subject">Subject</label>
        <input
          type="text"
          id="subject"
          name="subject"
          value={values.subject}
          onChange={handleChange}
          required
        />
      </div>

      <div>
        <label htmlFor="message">Message</label>
        <textarea
          id="message"
          name="message"
          value={values.message}
          onChange={handleChange}
          rows={5}
          required
        />
      </div>

      <button type="submit">Send Message</button>
    </form>
  );
};
```

## Управление состоянием форм

### Состояние с ошибками

```tsx
// Интерфейсы для состояния формы
interface FormState<T> {
  values: T;
  errors: Record<keyof T, string>;
  touched: Record<keyof T, boolean>;
  isSubmitting: boolean;
}

// Интерфейс для формы регистрации
interface RegistrationFormValues {
  username: string;
  email: string;
  password: string;
  confirmPassword: string;
}

// Начальное состояние формы
const initialRegistrationState: FormState<RegistrationFormValues> = {
  values: {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  },
  errors: {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  },
  touched: {
    username: false,
    email: false,
    password: false,
    confirmPassword: false
  },
  isSubmitting: false
};

// Компонент формы регистрации
const RegistrationForm: React.FC = () => {
  const [state, setState] = useState<FormState<RegistrationFormValues>>(initialRegistrationState);

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value } = e.target;
    
    setState(prev => ({
      ...prev,
      values: {
        ...prev.values,
        [name]: value
      },
      errors: {
        ...prev.errors,
        [name]: '' // Очищаем ошибку при изменении значения
      }
    }));
  };

  const handleBlur = (
    e: React.FocusEvent<HTMLInputElement>
  ) => {
    const { name } = e.target;
    
    setState(prev => ({
      ...prev,
      touched: {
        ...prev.touched,
        [name]: true
      }
    }));
  };

  const validateField = (name: keyof RegistrationFormValues, value: string): string => {
    switch (name) {
      case 'username':
        if (!value) return 'Username is required';
        if (value.length < 3) return 'Username must be at least 3 characters';
        return '';
      
      case 'email':
        if (!value) return 'Email is required';
        if (!/\S+@\S+\.\S+/.test(value)) return 'Email is invalid';
        return '';
      
      case 'password':
        if (!value) return 'Password is required';
        if (value.length < 8) return 'Password must be at least 8 characters';
        return '';
      
      case 'confirmPassword':
        if (!value) return 'Please confirm your password';
        if (value !== state.values.password) return 'Passwords do not match';
        return '';
      
      default:
        return '';
    }
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    // Валидация всех полей
    const newErrors: Record<keyof RegistrationFormValues, string> = {
      username: validateField('username', state.values.username),
      email: validateField('email', state.values.email),
      password: validateField('password', state.values.password),
      confirmPassword: validateField('confirmPassword', state.values.confirmPassword)
    };
    
    setState(prev => ({
      ...prev,
      errors: newErrors
    }));
    
    // Проверка наличия ошибок
    const hasErrors = Object.values(newErrors).some(error => error !== '');
    if (!hasErrors) {
      setState(prev => ({ ...prev, isSubmitting: true }));
      // Логика отправки формы
      console.log('Form submitted:', state.values);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">Username</label>
        <input
          type="text"
          id="username"
          name="username"
          value={state.values.username}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {state.touched.username && state.errors.username && (
          <span className="error">{state.errors.username}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={state.values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {state.touched.email && state.errors.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          type="password"
          id="password"
          name="password"
          value={state.values.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {state.touched.password && state.errors.password && (
          <span className="error">{state.errors.password}</span>
        )}
      </div>

      <div>
        <label htmlFor="confirmPassword">Confirm Password</label>
        <input
          type="password"
          id="confirmPassword"
          name="confirmPassword"
          value={state.values.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {state.touched.confirmPassword && state.errors.confirmPassword && (
          <span className="error">{state.errors.confirmPassword}</span>
        )}
      </div>

      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
};
```

### Асинхронное состояние форм

```tsx
// Интерфейсы для асинхронного состояния
interface AsyncFormState<T> {
  values: T;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
  submitError: string | null;
  submitSuccess: boolean;
}

// Интерфейс для формы профиля
interface ProfileFormValues {
  firstName: string;
  lastName: string;
  email: string;
  bio: string;
}

// Начальное состояние
const initialProfileState: AsyncFormState<ProfileFormValues> = {
  values: {
    firstName: '',
    lastName: '',
    email: '',
    bio: ''
  },
  errors: {},
  touched: {},
  isSubmitting: false,
  submitError: null,
  submitSuccess: false
};

// Компонент формы профиля
const ProfileForm: React.FC = () => {
  const [state, setState] = useState<AsyncFormState<ProfileFormValues>>(initialProfileState);

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;
    
    setState(prev => ({
      ...prev,
      values: {
        ...prev.values,
        [name]: value
      },
      errors: {
        ...prev.errors,
        [name]: ''
      },
      submitError: null,
      submitSuccess: false
    }));
  };

  const handleBlur = (
    e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name } = e.target;
    
    setState(prev => ({
      ...prev,
      touched: {
        ...prev.touched,
        [name]: true
      }
    }));
  };

  const validateForm = (): boolean => {
    const errors: Record<string, string> = {};
    
    if (!state.values.firstName.trim()) {
      errors.firstName = 'First name is required';
    }
    
    if (!state.values.lastName.trim()) {
      errors.lastName = 'Last name is required';
    }
    
    if (!state.values.email.trim()) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(state.values.email)) {
      errors.email = 'Email is invalid';
    }
    
    setState(prev => ({ ...prev, errors }));
    return Object.keys(errors).length === 0;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!validateForm()) return;
    
    setState(prev => ({ ...prev, isSubmitting: true, submitError: null }));
    
    try {
      // Симуляция API вызова
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      // Успешная отправка
      setState(prev => ({
        ...prev,
        isSubmitting: false,
        submitSuccess: true,
        submitError: null
      }));
      
      // Сброс формы через 3 секунды
      setTimeout(() => {
        setState(initialProfileState);
      }, 3000);
    } catch (error) {
      setState(prev => ({
        ...prev,
        isSubmitting: false,
        submitError: error instanceof Error ? error.message : 'Failed to update profile',
        submitSuccess: false
      }));
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="firstName">First Name</label>
        <input
          type="text"
          id="firstName"
          name="firstName"
          value={state.values.firstName}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {state.errors.firstName && (
          <span className="error">{state.errors.firstName}</span>
        )}
      </div>

      <div>
        <label htmlFor="lastName">Last Name</label>
        <input
          type="text"
          id="lastName"
          name="lastName"
          value={state.values.lastName}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {state.errors.lastName && (
          <span className="error">{state.errors.lastName}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={state.values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {state.errors.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>

      <div>
        <label htmlFor="bio">Bio</label>
        <textarea
          id="bio"
          name="bio"
          value={state.values.bio}
          onChange={handleChange}
          onBlur={handleBlur}
          rows={4}
        />
      </div>

      {state.submitError && (
        <div className="error-message">{state.submitError}</div>
      )}
      
      {state.submitSuccess && (
        <div className="success-message">Profile updated successfully!</div>
      )}

      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Updating...' : 'Update Profile'}
      </button>
    </form>
  );
};
```

## Валидация форм

### Базовая валидация

```tsx
// Интерфейсы для валидации
interface ValidationRule<T> {
  validator: (value: T) => boolean;
  message: string;
}

interface ValidationRules<T> {
  [key: string]: ValidationRule<T>[];
}

// Типы для ошибок валидации
type ValidationErrors<T> = Partial<Record<keyof T, string>>;

// Интерфейс для формы с валидацией
interface LoginFormValues {
  email: string;
  password: string;
  rememberMe: boolean;
}

// Правила валидации
const loginValidationRules: ValidationRules<string> = {
  email: [
    {
      validator: (value: string) => value.length > 0,
      message: 'Email is required'
    },
    {
      validator: (value: string) => /\S+@\S+\.\S+/.test(value),
      message: 'Email is invalid'
    }
  ],
  password: [
    {
      validator: (value: string) => value.length > 0,
      message: 'Password is required'
    },
    {
      validator: (value: string) => value.length >= 8,
      message: 'Password must be at least 8 characters'
    }
  ]
};

// Функция валидации
const validateForm = <T extends Record<string, any>>(
  values: T,
  rules: ValidationRules<any>
): ValidationErrors<T> => {
  const errors: ValidationErrors<T> = {};

  Object.keys(values).forEach(key => {
    const value = values[key];
    const fieldRules = rules[key];
    
    if (fieldRules) {
      for (const rule of fieldRules) {
        if (!rule.validator(value)) {
          errors[key as keyof T] = rule.message;
          break;
        }
      }
    }
  });

  return errors;
};

// Компонент формы входа
const LoginForm: React.FC = () => {
  const [values, setValues] = useState<LoginFormValues>({
    email: '',
    password: '',
    rememberMe: false
  });
  
  const [errors, setErrors] = useState<ValidationErrors<LoginFormValues>>({});
  const [touched, setTouched] = useState<Record<string, boolean>>({});

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value, type, checked } = e.target;
    const newValue = type === 'checkbox' ? checked : value;
    
    setValues(prev => ({
      ...prev,
      [name]: newValue
    }));
    
    // Очищаем ошибку при изменении значения
    if (errors[name as keyof LoginFormValues]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name as keyof LoginFormValues];
        return newErrors;
      });
    }
  };

  const handleBlur = (
    e: React.FocusEvent<HTMLInputElement>
  ) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    const validationErrors = validateForm(values, loginValidationRules);
    setErrors(validationErrors);
    
    if (Object.keys(validationErrors).length === 0) {
      console.log('Form submitted:', values);
      // Логика отправки формы
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          type="password"
          id="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.password && errors.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="rememberMe"
            checked={values.rememberMe}
            onChange={handleChange}
          />
          Remember me
        </label>
      </div>

      <button type="submit">Login</button>
    </form>
  );
};
```

### Расширенная валидация

```tsx
// Интерфейсы для расширенной валидации
interface ExtendedValidationRule<T> {
  validator: (value: T, allValues?: Record<string, any>) => boolean | Promise<boolean>;
  message: string;
  async?: boolean;
}

interface ExtendedValidationRules<T> {
  [key: string]: ExtendedValidationRule<T>[];
}

// Асинхронная функция проверки уникальности email
const checkEmailUnique = async (email: string): Promise<boolean> => {
  // Симуляция API вызова
  return new Promise(resolve => {
    setTimeout(() => {
      // Предположим, что email "test@example.com" уже существует
      resolve(email !== 'test@example.com');
    }, 1000);
  });
};

// Расширенные правила валидации
const registrationValidationRules: ExtendedValidationRules<any> = {
  username: [
    {
      validator: (value: string) => value.length > 0,
      message: 'Username is required'
    },
    {
      validator: (value: string) => value.length >= 3,
      message: 'Username must be at least 3 characters'
    }
  ],
  email: [
    {
      validator: (value: string) => value.length > 0,
      message: 'Email is required'
    },
    {
      validator: (value: string) => /\S+@\S+\.\S+/.test(value),
      message: 'Email is invalid'
    },
    {
      validator: async (value: string) => {
        if (value) {
          const isUnique = await checkEmailUnique(value);
          return isUnique;
        }
        return true;
      },
      message: 'Email is already taken',
      async: true
    }
  ],
  password: [
    {
      validator: (value: string) => value.length > 0,
      message: 'Password is required'
    },
    {
      validator: (value: string) => value.length >= 8,
      message: 'Password must be at least 8 characters'
    },
    {
      validator: (value: string) => /[A-Z]/.test(value),
      message: 'Password must contain at least one uppercase letter'
    },
    {
      validator: (value: string) => /[0-9]/.test(value),
      message: 'Password must contain at least one number'
    }
  ],
  confirmPassword: [
    {
      validator: (value: string) => value.length > 0,
      message: 'Please confirm your password'
    },
    {
      validator: (value: string, allValues?: Record<string, any>) => {
        return allValues?.password === value;
      },
      message: 'Passwords do not match'
    }
  ]
};

// Интерфейс для состояния валидации
interface ValidationState {
  errors: Record<string, string>;
  asyncValidating: Record<string, boolean>;
}

// Компонент формы регистрации с расширенной валидацией
const ExtendedRegistrationForm: React.FC = () => {
  const [values, setValues] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  });
  
  const [validationState, setValidationState] = useState<ValidationState>({
    errors: {},
    asyncValidating: {}
  });
  
  const [touched, setTouched] = useState<Record<string, boolean>>({});

  const validateField = async (name: string, value: string) => {
    const rules = registrationValidationRules[name];
    if (!rules) return;

    setValidationState(prev => ({
      ...prev,
      asyncValidating: { ...prev.asyncValidating, [name]: true }
    }));

    for (const rule of rules) {
      let isValid: boolean;
      
      if (rule.async) {
        isValid = await rule.validator(value, values);
      } else {
        isValid = rule.validator(value, values);
      }
      
      if (!isValid) {
        setValidationState(prev => ({
          errors: { ...prev.errors, [name]: rule.message },
          asyncValidating: { ...prev.asyncValidating, [name]: false }
        }));
        return;
      }
    }

    setValidationState(prev => ({
      errors: { ...prev.errors, [name]: '' },
      asyncValidating: { ...prev.asyncValidating, [name]: false }
    }));
  };

  const handleChange = async (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value } = e.target;
    
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Очищаем ошибку при изменении значения
    if (validationState.errors[name]) {
      setValidationState(prev => {
        const newErrors = { ...prev.errors };
        delete newErrors[name];
        return { ...prev, errors: newErrors };
      });
    }
    
    // Валидация поля при изменении
    if (touched[name]) {
      await validateField(name, value);
    }
  };

  const handleBlur = async (
    e: React.FocusEvent<HTMLInputElement>
  ) => {
    const { name, value } = e.target;
    
    setTouched(prev => ({ ...prev, [name]: true }));
    
    // Валидация поля при потере фокуса
    await validateField(name, value);
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Валидация всех полей
    const fieldNames = Object.keys(values) as Array<keyof typeof values>;
    
    for (const fieldName of fieldNames) {
      await validateField(fieldName, values[fieldName]);
    }
    
    // Проверка наличия ошибок
    const hasErrors = Object.values(validationState.errors).some(error => error !== '');
    const isAsyncValidating = Object.values(validationState.asyncValidating).some(validating => validating);
    
    if (!hasErrors && !isAsyncValidating) {
      console.log('Form submitted:', values);
      // Логика отправки формы
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">Username</label>
        <input
          type="text"
          id="username"
          name="username"
          value={values.username}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.username && validationState.errors.username && (
          <span className="error">{validationState.errors.username}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && (
          <>
            {validationState.asyncValidating.email && (
              <span className="validating">Checking...</span>
            )}
            {validationState.errors.email && (
              <span className="error">{validationState.errors.email}</span>
            )}
          </>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          type="password"
          id="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.password && validationState.errors.password && (
          <span className="error">{validationState.errors.password}</span>
        )}
      </div>

      <div>
        <label htmlFor="confirmPassword">Confirm Password</label>
        <input
          type="password"
          id="confirmPassword"
          name="confirmPassword"
          value={values.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.confirmPassword && validationState.errors.confirmPassword && (
          <span className="error">{validationState.errors.confirmPassword}</span>
        )}
      </div>

      <button type="submit">Register</button>
    </form>
  );
};
```

## Типизация событий форм

### События ввода

```tsx
// Типизация различных событий ввода
interface FormEventHandlers {
  // События изменения текста
  onInputChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  
  // События изменения текстовой области
  onTextAreaChange: (e: React.ChangeEvent<HTMLTextAreaElement>) => void;
  
  // События изменения select
  onSelectChange: (e: React.ChangeEvent<HTMLSelectElement>) => void;
  
  // События изменения checkbox/radio
  onCheckboxChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  
  // События фокуса
  onFocus: (e: React.FocusEvent<HTMLInputElement>) => void;
  onBlur: (e: React.FocusEvent<HTMLInputElement>) => void;
  
  // События формы
  onSubmit: (e: React.FormEvent<HTMLFormElement>) => void;
  onReset: (e: React.FormEvent<HTMLFormElement>) => void;
}

// Компонент с различными типами полей
const ComplexForm: React.FC = () => {
  const [values, setValues] = useState({
    text: '',
    email: '',
    password: '',
    textarea: '',
    select: '',
    checkbox: false,
    radio: ''
  });

  const handleInputChange = (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleTextAreaChange = (
    e: React.ChangeEvent<HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleSelectChange = (
    e: React.ChangeEvent<HTMLSelectElement>
  ) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleFocus = (
    e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>
  ) => {
    console.log('Field focused:', e.target.name);
  };

  const handleBlur = (
    e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>
  ) => {
    console.log('Field blurred:', e.target.name);
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Form submitted:', values);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Text input */}
      <div>
        <label htmlFor="text">Text</label>
        <input
          type="text"
          id="text"
          name="text"
          value={values.text}
          onChange={handleInputChange}
          onFocus={handleFocus}
          onBlur={handleBlur}
        />
      </div>

      {/* Email input */}
      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={values.email}
          onChange={handleInputChange}
          onFocus={handleFocus}
          onBlur={handleBlur}
        />
      </div>

      {/* Password input */}
      <div>
        <label htmlFor="password">Password</label>
        <input
          type="password"
          id="password"
          name="password"
          value={values.password}
          onChange={handleInputChange}
          onFocus={handleFocus}
          onBlur={handleBlur}
        />
      </div>

      {/* Textarea */}
      <div>
        <label htmlFor="textarea">Textarea</label>
        <textarea
          id="textarea"
          name="textarea"
          value={values.textarea}
          onChange={handleTextAreaChange}
          onFocus={handleFocus}
          onBlur={handleBlur}
          rows={4}
        />
      </div>

      {/* Select */}
      <div>
        <label htmlFor="select">Select</label>
        <select
          id="select"
          name="select"
          value={values.select}
          onChange={handleSelectChange}
          onFocus={handleFocus}
          onBlur={handleBlur}
        >
          <option value="">Choose an option</option>
          <option value="option1">Option 1</option>
          <option value="option2">Option 2</option>
          <option value="option3">Option 3</option>
        </select>
      </div>

      {/* Checkbox */}
      <div>
        <label>
          <input
            type="checkbox"
            name="checkbox"
            checked={values.checkbox}
            onChange={handleInputChange}
            onFocus={handleFocus}
            onBlur={handleBlur}
          />
          Checkbox
        </label>
      </div>

      {/* Radio buttons */}
      <div>
        <label>
          <input
            type="radio"
            name="radio"
            value="option1"
            checked={values.radio === 'option1'}
            onChange={handleInputChange}
            onFocus={handleFocus}
            onBlur={handleBlur}
          />
          Radio Option 1
        </label>
        <label>
          <input
            type="radio"
            name="radio"
            value="option2"
            checked={values.radio === 'option2'}
            onChange={handleInputChange}
            onFocus={handleFocus}
            onBlur={handleBlur}
          />
          Radio Option 2
        </label>
      </div>

      <button type="submit">Submit</button>
    </form>
  );
};
```

### События клавиатуры

```tsx
// Типизация событий клавиатуры
interface KeyboardEventHandlers {
  onKeyPress: (e: React.KeyboardEvent<HTMLInputElement>) => void;
  onKeyDown: (e: React.KeyboardEvent<HTMLInputElement>) => void;
  onKeyUp: (e: React.KeyboardEvent<HTMLInputElement>) => void;
}

// Компонент с обработкой клавиатуры
const KeyboardForm: React.FC = () => {
  const [values, setValues] = useState({
    search: '',
    code: ''
  });

  const handleInputChange = (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleKeyPress = (
    e: React.KeyboardEvent<HTMLInputElement>
  ) => {
    if (e.key === 'Enter') {
      console.log('Enter pressed in', e.currentTarget.name);
      // Логика обработки Enter
    }
  };

  const handleKeyDown = (
    e: React.KeyboardEvent<HTMLInputElement>
  ) => {
    if (e.key === 'Escape') {
      console.log('Escape pressed in', e.currentTarget.name);
      // Логика обработки Escape
      setValues(prev => ({ ...prev, [e.currentTarget.name]: '' }));
    }
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Form submitted:', values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="search">Search</label>
        <input
          type="text"
          id="search"
          name="search"
          value={values.search}
          onChange={handleInputChange}
          onKeyPress={handleKeyPress}
          onKeyDown={handleKeyDown}
          placeholder="Press Enter to search, Escape to clear"
        />
      </div>

      <div>
        <label htmlFor="code">Code</label>
        <input
          type="text"
          id="code"
          name="code"
          value={values.code}
          onChange={handleInputChange}
          onKeyPress={handleKeyPress}
          onKeyDown={handleKeyDown}
          placeholder="Enter code"
        />
      </div>

      <button type="submit">Submit</button>
    </form>
  );
};
```

## Работа с различными типами полей

### Поля ввода с масками

```tsx
// Интерфейсы для полей с масками
interface MaskedFormValues {
  phone: string;
  creditCard: string;
  date: string;
  ssn: string;
}

// Компонент поля с маской
interface MaskedInputProps {
  name: string;
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  mask: string;
  placeholder?: string;
}

const MaskedInput: React.FC<MaskedInputProps> = ({ 
  name, 
  value, 
  onChange, 
  mask, 
  placeholder 
}) => {
  const applyMask = (value: string, mask: string): string => {
    const numericValue = value.replace(/\D/g, '');
    let maskedValue = '';
    let valueIndex = 0;

    for (let i = 0; i < mask.length && valueIndex < numericValue.length; i++) {
      if (mask[i] === 'X') {
        maskedValue += numericValue[valueIndex];
        valueIndex++;
      } else {
        maskedValue += mask[i];
      }
    }

    return maskedValue;
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const maskedValue = applyMask(e.target.value, mask);
    const fakeEvent = {
      ...e,
      target: {
        ...e.target,
        value: maskedValue
      }
    } as React.ChangeEvent<HTMLInputElement>;
    
    onChange(fakeEvent);
  };

  return (
    <input
      type="text"
      name={name}
      value={value}
      onChange={handleChange}
      placeholder={placeholder}
    />
  );
};

// Компонент формы с масками
const MaskedForm: React.FC = () => {
  const [values, setValues] = useState<MaskedFormValues>({
    phone: '',
    creditCard: '',
    date: '',
    ssn: ''
  });

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Form submitted:', values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="phone">Phone</label>
        <MaskedInput
          name="phone"
          value={values.phone}
          onChange={handleChange}
          mask="(XXX) XXX-XXXX"
          placeholder="(123) 456-7890"
        />
      </div>

      <div>
        <label htmlFor="creditCard">Credit Card</label>
        <MaskedInput
          name="creditCard"
          value={values.creditCard}
          onChange={handleChange}
          mask="XXXX XXXX XXXX XXXX"
          placeholder="1234 5678 9012 3456"
        />
      </div>

      <div>
        <label htmlFor="date">Date (MM/DD/YYYY)</label>
        <MaskedInput
          name="date"
          value={values.date}
          onChange={handleChange}
          mask="XX/XX/XXXX"
          placeholder="MM/DD/YYYY"
        />
      </div>

      <div>
        <label htmlFor="ssn">SSN</label>
        <MaskedInput
          name="ssn"
          value={values.ssn}
          onChange={handleChange}
          mask="XXX-XX-XXXX"
          placeholder="123-45-6789"
        />
      </div>

      <button type="submit">Submit</button>
    </form>
  );
};
```

### Поля с автозаполнением

```tsx
// Интерфейсы для автозаполнения
interface AutocompleteFormValues {
  country: string;
  city: string;
  language: string;
}

interface Suggestion {
  id: string;
  name: string;
}

// Компонент автозаполнения
interface AutocompleteProps {
  name: string;
  value: string;
  onChange: (value: string) => void;
  suggestions: Suggestion[];
  placeholder?: string;
}

const Autocomplete: React.FC<AutocompleteProps> = ({ 
  name, 
  value, 
  onChange, 
  suggestions, 
  placeholder 
}) => {
  const [isOpen, setIsOpen] = useState(false);
  const [filteredSuggestions, setFilteredSuggestions] = useState<Suggestion[]>([]);

  useEffect(() => {
    if (value) {
      const filtered = suggestions.filter(suggestion =>
        suggestion.name.toLowerCase().includes(value.toLowerCase())
      );
      setFilteredSuggestions(filtered);
      setIsOpen(filtered.length > 0);
    } else {
      setFilteredSuggestions([]);
      setIsOpen(false);
    }
  }, [value, suggestions]);

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.value);
  };

  const handleSuggestionClick = (suggestion: Suggestion) => {
    onChange(suggestion.name);
    setIsOpen(false);
  };

  const handleBlur = () => {
    // Задержка для возможности клика по suggestion
    setTimeout(() => setIsOpen(false), 150);
  };

  return (
    <div className="autocomplete">
      <input
        type="text"
        name={name}
        value={value}
        onChange={handleInputChange}
        onFocus={() => value && setIsOpen(true)}
        onBlur={handleBlur}
        placeholder={placeholder}
      />
      {isOpen && (
        <ul className="suggestions">
          {filteredSuggestions.map(suggestion => (
            <li
              key={suggestion.id}
              onMouseDown={() => handleSuggestionClick(suggestion)}
            >
              {suggestion.name}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};

// Компонент формы с автозаполнением
const AutocompleteForm: React.FC = () => {
  const [values, setValues] = useState<AutocompleteFormValues>({
    country: '',
    city: '',
    language: ''
  });

  // Данные для автозаполнения
  const countries: Suggestion[] = [
    { id: 'us', name: 'United States' },
    { id: 'ca', name: 'Canada' },
    { id: 'uk', name: 'United Kingdom' },
    { id: 'de', name: 'Germany' },
    { id: 'fr', name: 'France' }
  ];

  const cities: Suggestion[] = [
    { id: 'ny', name: 'New York' },
    { id: 'la', name: 'Los Angeles' },
    { id: 'ld', name: 'London' },
    { id: 'br', name: 'Berlin' },
    { id: 'pa', name: 'Paris' }
  ];

  const languages: Suggestion[] = [
    { id: 'en', name: 'English' },
    { id: 'fr', name: 'French' },
    { id: 'de', name: 'German' },
    { id: 'es', name: 'Spanish' },
    { id: 'it', name: 'Italian' }
  ];

  const handleChange = (name: keyof AutocompleteFormValues, value: string) => {
    setValues(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Form submitted:', values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="country">Country</label>
        <Autocomplete
          name="country"
          value={values.country}
          onChange={(value) => handleChange('country', value)}
          suggestions={countries}
          placeholder="Select country"
        />
      </div>

      <div>
        <label htmlFor="city">City</label>
        <Autocomplete
          name="city"
          value={values.city}
          onChange={(value) => handleChange('city', value)}
          suggestions={cities}
          placeholder="Select city"
        />
      </div>

      <div>
        <label htmlFor="language">Language</label>
        <Autocomplete
          name="language"
          value={values.language}
          onChange={(value) => handleChange('language', value)}
          suggestions={languages}
          placeholder="Select language"
        />
      </div>

      <button type="submit">Submit</button>
    </form>
  );
};
```

## Интеграция с библиотеками форм

### React Hook Form

```tsx
// Установка библиотеки
// npm install react-hook-form

// Интерфейсы для React Hook Form
interface ReactHookFormValues {
  firstName: string;
  lastName: string;
  email: string;
  age: number;
  website: string;
  terms: boolean;
}

// Компонент с React Hook Form
const ReactHookFormExample: React.FC = () => {
  const {
    register,
    handleSubmit,
    watch,
    formState: { errors, isSubmitting }
  } = useForm<ReactHookFormValues>({
    defaultValues: {
      firstName: '',
      lastName: '',
      email: '',
      age: 18,
      website: '',
      terms: false
    }
  });

  const onSubmit: SubmitHandler<ReactHookFormValues> = async (data) => {
    console.log('Form submitted:', data);
    // Логика отправки формы
    await new Promise(resolve => setTimeout(resolve, 2000));
  };

  // Наблюдение за значениями
  const watchFirstName = watch('firstName');
  const watchAllFields = watch();

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label htmlFor="firstName">First Name</label>
        <input
          id="firstName"
          {...register('firstName', {
            required: 'First name is required',
            minLength: {
              value: 2,
              message: 'First name must be at least 2 characters'
            }
          })}
        />
        {errors.firstName && (
          <span className="error">{errors.firstName.message}</span>
        )}
        {watchFirstName && (
          <p>Hello, {watchFirstName}!</p>
        )}
      </div>

      <div>
        <label htmlFor="lastName">Last Name</label>
        <input
          id="lastName"
          {...register('lastName', {
            required: 'Last name is required',
            minLength: {
              value: 2,
              message: 'Last name must be at least 2 characters'
            }
          })}
        />
        {errors.lastName && (
          <span className="error">{errors.lastName.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          {...register('email', {
            required: 'Email is required',
            pattern: {
              value: /\S+@\S+\.\S+/,
              message: 'Email is invalid'
            }
          })}
        />
        {errors.email && (
          <span className="error">{errors.email.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="age">Age</label>
        <input
          id="age"
          type="number"
          {...register('age', {
            required: 'Age is required',
            min: {
              value: 18,
              message: 'You must be at least 18 years old'
            },
            max: {
              value: 120,
              message: 'Age must be less than 120'
            }
          })}
        />
        {errors.age && (
          <span className="error">{errors.age.message}</span>
        )}
      </div>

      <div>
        <label htmlFor="website">Website</label>
        <input
          id="website"
          type="url"
          {...register('website', {
            pattern: {
              value: /^https?:\/\/.+/,
              message: 'Website must be a valid URL'
            }
          })}
        />
        {errors.website && (
          <span className="error">{errors.website.message}</span>
        )}
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            {...register('terms', {
              required: 'You must accept the terms and conditions'
            })}
          />
          I accept the terms and conditions
        </label>
        {errors.terms && (
          <span className="error">{errors.terms.message}</span>
        )}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>

      <div>
        <h3>Form Values:</h3>
        <pre>{JSON.stringify(watchAllFields, null, 2)}</pre>
      </div>
    </form>
  );
};
```

### Formik

```tsx
// Установка библиотеки
// npm install formik yup

// Интерфейсы для Formik
interface FormikFormValues {
  email: string;
  password: string;
  confirmPassword: string;
}

// Схема валидации с Yup
const validationSchema = Yup.object({
  email: Yup.string()
    .email('Invalid email address')
    .required('Email is required'),
  password: Yup.string()
    .min(8, 'Password must be at least 8 characters')
    .required('Password is required'),
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password')], 'Passwords must match')
    .required('Confirm password is required')
});

// Компонент с Formik
const FormikExample: React.FC = () => {
  return (
    <Formik
      initialValues={{
        email: '',
        password: '',
        confirmPassword: ''
      }}
      validationSchema={validationSchema}
      onSubmit={async (values, { setSubmitting, resetForm }) => {
        console.log('Form submitted:', values);
        // Логика отправки формы
        await new Promise(resolve => setTimeout(resolve, 2000));
        setSubmitting(false);
        resetForm();
      }}
    >
      {({ isSubmitting, errors, touched }) => (
        <Form>
          <div>
            <label htmlFor="email">Email</label>
            <Field
              type="email"
              id="email"
              name="email"
              as="input"
            />
            {touched.email && errors.email && (
              <span className="error">{errors.email}</span>
            )}
          </div>

          <div>
            <label htmlFor="password">Password</label>
            <Field
              type="password"
              id="password"
              name="password"
              as="input"
            />
            {touched.password && errors.password && (
              <span className="error">{errors.password}</span>
            )}
          </div>

          <div>
            <label htmlFor="confirmPassword">Confirm Password</label>
            <Field
              type="password"
              id="confirmPassword"
              name="confirmPassword"
              as="input"
            />
            {touched.confirmPassword && errors.confirmPassword && (
              <span className="error">{errors.confirmPassword}</span>
            )}
          </div>

          <button type="submit" disabled={isSubmitting}>
            {isSubmitting ? 'Submitting...' : 'Submit'}
          </button>
        </Form>
      )}
    </Formik>
  );
};
```

## Пользовательские хуки для форм

### Базовый хук для форм

```tsx
// Интерфейсы для пользовательского хука
interface UseFormOptions<T> {
  initialValues: T;
  onSubmit: (values: T) => void | Promise<void>;
  validate?: (values: T) => Record<string, string>;
}

interface UseFormResult<T> {
  values: T;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
  handleChange: (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => void;
  handleBlur: (e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => void;
  handleSubmit: (e: React.FormEvent) => void;
  setValues: React.Dispatch<React.SetStateAction<T>>;
  setErrors: React.Dispatch<React.SetStateAction<Record<string, string>>>;
  reset: () => void;
}

// Пользовательский хук для форм
function useForm<T extends Record<string, any>>({
  initialValues,
  onSubmit,
  validate
}: UseFormOptions<T>): UseFormResult<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [touched, setTouched] = useState<Record<string, boolean>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>
  ) => {
    const { name, value, type } = e.target;
    const target = e.target as HTMLInputElement;
    const checked = target.checked;
    
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
    
    // Очищаем ошибку при изменении значения
    if (errors[name]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
    }
  };

  const handleBlur = (
    e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>
  ) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Валидация
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }
    
    setIsSubmitting(true);
    
    try {
      await onSubmit(values);
    } catch (error) {
      console.error('Form submission error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    setValues,
    setErrors,
    reset
  };
}

// Использование пользовательского хука
interface ContactFormValues {
  name: string;
  email: string;
  subject: string;
  message: string;
}

const ContactFormWithHook: React.FC = () => {
  const {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  } = useForm<ContactFormValues>({
    initialValues: {
      name: '',
      email: '',
      subject: '',
      message: ''
    },
    onSubmit: async (values) => {
      console.log('Form submitted:', values);
      // Логика отправки формы
      await new Promise(resolve => setTimeout(resolve, 2000));
    },
    validate: (values) => {
      const errors: Record<string, string> = {};
      
      if (!values.name.trim()) {
        errors.name = 'Name is required';
      }
      
      if (!values.email.trim()) {
        errors.email = 'Email is required';
      } else if (!/\S+@\S+\.\S+/.test(values.email)) {
        errors.email = 'Email is invalid';
      }
      
      if (!values.subject.trim()) {
        errors.subject = 'Subject is required';
      }
      
      if (!values.message.trim()) {
        errors.message = 'Message is required';
      }
      
      return errors;
    }
  });

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="name">Name</label>
        <input
          type="text"
          id="name"
          name="name"
          value={values.name}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.name && errors.name && (
          <span className="error">{errors.name}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>

      <div>
        <label htmlFor="subject">Subject</label>
        <input
          type="text"
          id="subject"
          name="subject"
          value={values.subject}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.subject && errors.subject && (
          <span className="error">{errors.subject}</span>
        )}
      </div>

      <div>
        <label htmlFor="message">Message</label>
        <textarea
          id="message"
          name="message"
          value={values.message}
          onChange={handleChange}
          onBlur={handleBlur}
          rows={5}
        />
        {touched.message && errors.message && (
          <span className="error">{errors.message}</span>
        )}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Sending...' : 'Send Message'}
      </button>
      
      <button type="button" onClick={reset}>
        Reset
      </button>
    </form>
  );
};
```

### Расширенный хук для форм

```tsx
// Интерфейсы для расширенного хука
interface ExtendedUseFormOptions<T> {
  initialValues: T;
  onSubmit: (values: T) => void | Promise<void>;
  validate?: (values: T) => Record<string, string>;
  validateField?: (name: string, value: any, allValues: T) => string | null;
  onSubmitSuccess?: () => void;
  onSubmitError?: (error: any) => void;
}

interface ExtendedUseFormResult<T> {
  values: T;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
  isValidating: boolean;
  isDirty: boolean;
  isValid: boolean;
  handleChange: (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => void;
  handleBlur: (e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>) => void;
  handleSubmit: (e: React.FormEvent) => void;
  setFieldValue: (name: string, value: any) => void;
  setFieldError: (name: string, error: string) => void;
  setFieldTouched: (name: string, touched: boolean) => void;
  reset: () => void;
  resetField: (name: string) => void;
}

// Расширенный пользовательский хук для форм
function useExtendedForm<T extends Record<string, any>>({
  initialValues,
  onSubmit,
  validate,
  validateField,
  onSubmitSuccess,
  onSubmitError
}: ExtendedUseFormOptions<T>): ExtendedUseFormResult<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [touched, setTouched] = useState<Record<string, boolean>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isValidating, setIsValidating] = useState(false);

  // Проверка, были ли изменения в форме
  const isDirty = useMemo(() => {
    return JSON.stringify(values) !== JSON.stringify(initialValues);
  }, [values, initialValues]);

  // Проверка валидности формы
  const isValid = useMemo(() => {
    return Object.keys(errors).length === 0 && 
           Object.values(touched).some(t => t);
  }, [errors, touched]);

  const setFieldValue = (name: string, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Очищаем ошибку при изменении значения
    if (errors[name]) {
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
    }
  };

  const setFieldError = (name: string, error: string) => {
    setErrors(prev => ({ ...prev, [name]: error }));
  };

  const setFieldTouched = (name: string, touched: boolean) => {
    setTouched(prev => ({ ...prev, [name]: touched }));
  };

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>
  ) => {
    const { name, value, type } = e.target;
    const target = e.target as HTMLInputElement;
    const checked = target.checked;
    
    setFieldValue(name, type === 'checkbox' ? checked : value);
  };

  const handleBlur = async (
    e: React.FocusEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>
  ) => {
    const { name, value } = e.target;
    setFieldTouched(name, true);
    
    // Валидация поля при потере фокуса
    if (validateField) {
      setIsValidating(true);
      const error = validateField(name, value, values);
      if (error) {
        setFieldError(name, error);
      }
      setIsValidating(false);
    }
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Помечаем все поля как touched
    const allTouched: Record<string, boolean> = {};
    Object.keys(values).forEach(key => {
      allTouched[key] = true;
    });
    setTouched(allTouched);
    
    // Валидация всей формы
    let validationErrors: Record<string, string> = {};
    
    if (validate) {
      validationErrors = validate(values);
    } else if (validateField) {
      setIsValidating(true);
      Object.keys(values).forEach(key => {
        const error = validateField(key, values[key], values);
        if (error) {
          validationErrors[key] = error;
        }
      });
      setIsValidating(false);
    }
    
    setErrors(validationErrors);
    
    if (Object.keys(validationErrors).length > 0) {
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      await onSubmit(values);
      onSubmitSuccess?.();
    } catch (error) {
      onSubmitError?.(error);
    } finally {
      setIsSubmitting(false);
    }
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
    setIsValidating(false);
  };

  const resetField = (name: string) => {
    setValues(prev => ({ ...prev, [name]: initialValues[name] }));
    setErrors(prev => {
      const newErrors = { ...prev };
      delete newErrors[name];
      return newErrors;
    });
    setTouched(prev => {
      const newTouched = { ...prev };
      delete newTouched[name];
      return newTouched;
    });
  };

  return {
    values,
    errors,
    touched,
    isSubmitting,
    isValidating,
    isDirty,
    isValid,
    handleChange,
    handleBlur,
    handleSubmit,
    setFieldValue,
    setFieldError,
    setFieldTouched,
    reset,
    resetField
  };
}

// Использование расширенного хука
interface UserProfileFormValues {
  firstName: string;
  lastName: string;
  email: string;
  bio: string;
  newsletter: boolean;
}

const UserProfileFormWithExtendedHook: React.FC = () => {
  const {
    values,
    errors,
    touched,
    isSubmitting,
    isValidating,
    isDirty,
    isValid,
    handleChange,
    handleBlur,
    handleSubmit,
    setFieldValue,
    reset,
    resetField
  } = useExtendedForm<UserProfileFormValues>({
    initialValues: {
      firstName: '',
      lastName: '',
      email: '',
      bio: '',
      newsletter: false
    },
    onSubmit: async (values) => {
      console.log('Form submitted:', values);
      // Логика отправки формы
      await new Promise(resolve => setTimeout(resolve, 2000));
    },
    validateField: (name, value, allValues) => {
      switch (name) {
        case 'firstName':
          if (!value.trim()) return 'First name is required';
          if (value.length < 2) return 'First name must be at least 2 characters';
          return null;
        
        case 'lastName':
          if (!value.trim()) return 'Last name is required';
          if (value.length < 2) return 'Last name must be at least 2 characters';
          return null;
        
        case 'email':
          if (!value.trim()) return 'Email is required';
          if (!/\S+@\S+\.\S+/.test(value)) return 'Email is invalid';
          return null;
        
        default:
          return null;
      }
    },
    onSubmitSuccess: () => {
      console.log('Form submitted successfully!');
    },
    onSubmitError: (error) => {
      console.error('Form submission error:', error);
    }
  });

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="firstName">First Name</label>
        <input
          type="text"
          id="firstName"
          name="firstName"
          value={values.firstName}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.firstName && errors.firstName && (
          <span className="error">{errors.firstName}</span>
        )}
      </div>

      <div>
        <label htmlFor="lastName">Last Name</label>
        <input
          type="text"
          id="lastName"
          name="lastName"
          value={values.lastName}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.lastName && errors.lastName && (
          <span className="error">{errors.lastName}</span>
        )}
      </div>

      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>

      <div>
        <label htmlFor="bio">Bio</label>
        <textarea
          id="bio"
          name="bio"
          value={values.bio}
          onChange={handleChange}
          onBlur={handleBlur}
          rows={4}
        />
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="newsletter"
            checked={values.newsletter}
            onChange={handleChange}
          />
          Subscribe to newsletter
        </label>
      </div>

      <div>
        <button type="submit" disabled={isSubmitting || isValidating}>
          {isSubmitting ? 'Saving...' : 'Save Profile'}
        </button>
        
        <button type="button" onClick={reset} disabled={!isDirty}>
          Reset
        </button>
        
        <button type="button" onClick={() => resetField('bio')}>
          Reset Bio
        </button>
      </div>

      <div>
        <p>Form status:</p>
        <ul>
          <li>Is dirty: {isDirty ? 'Yes' : 'No'}</li>
          <li>Is valid: {isValid ? 'Yes' : 'No'}</li>
          <li>Is submitting: {isSubmitting ? 'Yes' : 'No'}</li>
          <li>Is validating: {isValidating ? 'Yes' : 'No'}</li>
        </ul>
      </div>
    </form>
  );
};
```

## Лучшие практики

### 1. Явная типизация всех форм

```tsx
// Хорошо: явная типизация
interface LoginFormValues {
  email: string;
  password: string;
  rememberMe: boolean;
}

const LoginForm: React.FC = () => {
  const [values, setValues] = useState<LoginFormValues>({
    email: '',
    password: '',
    rememberMe: false
  });
  // Реализация
};

// Плохо: неявная типизация
const LoginForm: React.FC = () => {
  const [values, setValues] = useState({
    email: '',
    password: '',
    rememberMe: false
  }); // any типы
  // Реализация
};
```

### 2. Использование utility types

```tsx
interface User {
  id: number;
  firstName: string;
  lastName: string;
  email: string;
  password: string;
  createdAt: Date;
  updatedAt: Date;
}

// Partial для форм редактирования
interface UserEditFormValues extends Partial<User> {
  firstName: string; // Обязательное поле
  lastName: string;  // Обязательное поле
  email: string;     // Обязательное поле
}

// Omit для исключения полей
interface UserPublicFormValues extends Omit<User, 'password' | 'id' | 'createdAt' | 'updatedAt'> {}

// Pick для выбора полей
interface UserBasicFormValues extends Pick<User, 'firstName' | 'lastName' | 'email'> {}
```

### 3. Типизация обработчиков событий

```tsx
// Хорошо: точная типизация событий
interface FormEventHandlers {
  onInputChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  onTextAreaChange: (e: React.ChangeEvent<HTMLTextAreaElement>) => void;
  onSelectChange: (e: React.ChangeEvent<HTMLSelectElement>) => void;
  onSubmit: (e: React.FormEvent<HTMLFormElement>) => void;
}

// Плохо: общая типизация
interface FormEventHandlers {
  onInputChange: (e: React.SyntheticEvent) => void; // Слишком общий тип
  onSubmit: (e: any) => void; // any тип
}
```

### 4. Типизация пользовательских хуков

```tsx
// Хорошо: четкая типизация возвращаемых значений
interface UseFormResult<T> {
  values: T;
  errors: Record<string, string>;
  handleChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  handleSubmit: (e: React.FormEvent) => void;
}

function useForm<T>(options: UseFormOptions<T>): UseFormResult<T> {
  // Реализация
}

// Плохо: неструктурированный возврат
function useForm<T>(options: UseFormOptions<T>) {
  // Реализация
  return [
    values,
    errors,
    handleChange,
    handleSubmit
  ]; // [T, Record<string, string>, Function, Function] - неочевидно
}
```

## Распространенные ошибки

### 1. Неявная типизация any

```tsx
// Проблема: неявный any
function BadForm({ data }) {  // data: any
  const [values, setValues] = useState(data);  // any типы
  return <form>{/* Реализация */}</form>;
}

// Исправление: явная типизация
interface FormData {
  name: string;
  email: string;
}

interface Props {
  data: FormData;
}

function GoodForm({ data }: Props) {
  const [values, setValues] = useState<FormData>(data);
  return <form>{/* Реализация */}</form>;
}
```

### 2. Неправильная типизация событий

```tsx
// Проблема: неправильный тип события
const handleChange = (e) => {  // e: any
  console.log(e.target.value);  // Нет автозаполнения
};

// Исправление: правильная типизация
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);  // Полная автозаполнение и проверка типов
};
```

### 3. Проблемы с useState

```tsx
// Проблема: неопределенное состояние
const [user, setUser] = useState();  // user: undefined

// Исправление: явная типизация
interface User {
  id: number;
  name: string;
}

const [user, setUser] = useState<User | null>(null);
```

### 4. Неправильная типизация пользовательских хуков

```tsx
// Проблема: неправильная типизация параметров
function useForm(initialValues: any) {  // any тип
  const [values, setValues] = useState(initialValues);
  // Реализация
}

// Исправление: дженерик типизация
function useForm<T>(initialValues: T) {
  const [values, setValues] = useState<T>(initialValues);
  // Реализация
}
```

## Связи с другими концепциями

- [[ts/react/React_с_TypeScript|TypeScript с React]] - Основы использования TypeScript в React
- [[ts/react/hooks|Хуки]] - Типизация хуков в React
- [[ts/react/components|Компоненты]] - Типизация компонентов с формами
- [[ts/advanced/generics|Дженерики]] - Использование дженериков в формах
- [[ts/advanced/utility-types|Вспомогательные типы]] - Utility types для типизации форм

## Рекомендации по изучению

1. Начните с базовой типизации форм и управления состоянием
2. Освойте валидацию форм с TypeScript
3. Практикуйтесь в типизации событий форм
4. Изучите работу с различными типами полей
5. Освойте интеграцию с библиотеками форм (React Hook Form, Formik)
6. Практикуйтесь в создании пользовательских хуков для форм
7. Изучите лучшие практики типизации форм
8. Освойте обработку распространенных ошибок

## Следующие шаги

- [[ts/react/context|Контекст]] - Типизация контекста и провайдеров
- [[ts/react/testing|Тестирование]] - Тестирование React-компонентов с TypeScript
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация React-приложений с TypeScript
- [[ts/testing/index|Тестирование]] - Общие принципы тестирования TypeScript-приложений
---
aliases: ["Утилиты для валидации с типами", "TS Validation Utilities"]
tags: [typescript, validation, utilities]
---

# Утилиты для валидации с типами

## Введение

Валидация данных - важная часть любого приложения. TypeScript позволяет создавать типобезопасные утилиты для валидации, которые обеспечивают как проверку во время выполнения, так и строгую типизацию на этапе компиляции.

## Основы валидации с типами

### Простая утилита для валидации

```typescript
interface ValidationResult<T> {
  isValid: boolean;
  errors: string[];
  value?: T;
}

// Базовая утилита валидации
function validate<T>(
  value: unknown,
  validator: (v: unknown) => v is T,
  errorMessage: string = 'Invalid value'
): ValidationResult<T> {
  if (validator(value)) {
    return { isValid: true, errors: [], value };
  }
  return { isValid: false, errors: [errorMessage] };
}

// Проверка типа для строки
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

// Пример использования
const stringValidation = validate('hello', isString, 'Value must be a string');
console.log(stringValidation); // { isValid: true, errors: [], value: 'hello' }
```

### Утилита для валидации чисел

```typescript
function isNumber(value: unknown): value is number {
  return typeof value === 'number' && !isNaN(value);
}

function isPositive(value: number): value is number {
  return value > 0;
}

function validatePositiveNumber(value: unknown): ValidationResult<number> {
  const stringValidation = validate(value, isNumber, 'Value must be a number');
  if (!stringValidation.isValid || stringValidation.value === undefined) {
    return stringValidation;
  }
  
  if (isPositive(stringValidation.value)) {
    return { isValid: true, errors: [], value: stringValidation.value };
  }
  
  return { isValid: false, errors: ['Number must be positive'] };
}

const positiveValidation = validatePositiveNumber(42);
console.log(positiveValidation); // { isValid: true, errors: [], value: 42 }
```

## Продвинутые утилиты валидации

### Утилита для валидации объектов

```typescript
interface UserSchema {
  id: number;
  name: string;
  email: string;
  age?: number;
}

function isUser(value: unknown): value is UserSchema {
  if (typeof value !== 'object' || value === null) {
    return false;
  }
  
  const user = value as Record<string, unknown>;
  
  return (
    typeof user.id === 'number' &&
    typeof user.name === 'string' &&
    typeof user.email === 'string' &&
    (user.age === undefined || typeof user.age === 'number')
  );
}

function validateUser(value: unknown): ValidationResult<UserSchema> {
  return validate(value, isUser, 'Value must be a valid User object');
}

const userData = { id: 1, name: 'John', email: 'john@example.com' };
const userValidation = validateUser(userData);
console.log(userValidation); // { isValid: true, errors: [], value: { id: 1, name: 'John', email: 'john@example.com' } }
```

### Универсальный валидатор с цепочкой проверок

```typescript
class Validator<T> {
  constructor(private value: unknown) {}
  
  static create<T>(value: unknown): Validator<T> {
    return new Validator<T>(value);
  }
  
  test(testFn: (value: unknown) => value is T, message: string): ValidationResult<T> {
    if (testFn(this.value)) {
      return { isValid: true, errors: [], value: this.value as T };
    }
    return { isValid: false, errors: [message] };
  }
}

// Пример использования
const validation = Validator.create('test@example.com')
  .test(isString, 'Value must be a string');

if (validation.isValid && validation.value) {
  console.log(`Valid string: ${validation.value}`);
}
```

### Валидация с использованием схемы

```typescript
// Определяем типы для схемы валидации
type Schema<T> = {
  [K in keyof T]: (value: unknown) => value is T[K];
};

function validateWithSchema<T>(value: unknown, schema: Schema<T>): ValidationResult<T> {
  if (typeof value !== 'object' || value === null) {
    return { isValid: false, errors: ['Value must be an object'] };
  }
  
  const obj = value as Record<string, unknown>;
  const result = {} as T;
  const errors: string[] = [];
  
  for (const key in schema) {
    if (Object.prototype.hasOwnProperty.call(schema, key)) {
      const validator = schema[key];
      const propValue = obj[key];
      
      if (validator(propValue)) {
        (result as any)[key] = propValue;
      } else {
        errors.push(`Invalid value for property '${key}'`);
      }
    }
  }
  
  if (errors.length > 0) {
    return { isValid: false, errors };
  }
  
  return { isValid: true, errors: [], value: result };
}

// Определяем схему для пользователя
const userSchema: Schema<UserSchema> = {
  id: (v: unknown): v is number => typeof v === 'number',
  name: (v: unknown): v is string => typeof v === 'string' && v.length > 0,
  email: (v: unknown): v is string => typeof v === 'string' && v.includes('@'),
  age: (v: unknown): v is number | undefined => v === undefined || typeof v === 'number'
};

const userData2 = { id: 1, name: 'John', email: 'john@example.com', age: 30 };
const schemaValidation = validateWithSchema(userData2, userSchema);
console.log(schemaValidation);
```

## Практические примеры

### Валидация форм

```typescript
interface LoginForm {
  email: string;
  password: string;
}

const loginFormSchema: Schema<LoginForm> = {
  email: (v: unknown): v is string => 
    typeof v === 'string' && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
  password: (v: unknown): v is string => 
    typeof v === 'string' && v.length >= 8
};

function validateLoginForm(data: unknown): ValidationResult<LoginForm> {
  return validateWithSchema(data, loginFormSchema);
}

const formData = { email: 'test@example.com', password: 'password123' };
const loginValidation = validateLoginForm(formData);
console.log(loginValidation);
```

### Валидация API ответов

```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

function isValidApiResponse<T>(value: unknown): value is ApiResponse<T> {
  if (typeof value !== 'object' || value === null) {
    return false;
  }
  
  const resp = value as Record<string, unknown>;
  return (
    typeof resp.success === 'boolean' &&
    (resp.success ? 'data' in resp : 'error' in resp)
  );
}

function validateApiResponse<T>(value: unknown): ValidationResult<ApiResponse<T>> {
  return validate(value, isValidApiResponse, 'Invalid API response format');
}

const apiResponse = { success: true, data: { id: 1, name: 'John' } };
const responseValidation = validateApiResponse(apiResponse);
console.log(responseValidation);
```

### Валидация с кастомными ошибками

```typescript
interface ValidationError {
  field: string;
  message: string;
  value?: unknown;
}

class TypedValidator<T> {
  private errors: ValidationError[] = [];
  
  constructor(private value: unknown) {}
  
  static create<T>(value: unknown): TypedValidator<T> {
    return new TypedValidator<T>(value);
  }
  
  required<K extends keyof T>(field: K, message: string = `${String(field)} is required`): TypedValidator<T> {
    const obj = this.value as Record<string, unknown>;
    if (obj[field] === undefined || obj[field] === null) {
      this.errors.push({ field: String(field), message, value: obj[field] });
    }
    return this;
  }
  
  string<K extends keyof T>(field: K, message: string = `${String(field)} must be a string`): TypedValidator<T> {
    const obj = this.value as Record<string, unknown>;
    if (obj[field] !== undefined && typeof obj[field] !== 'string') {
      this.errors.push({ field: String(field), message, value: obj[field] });
    }
    return this;
  }
  
  email<K extends keyof T>(field: K, message: string = `${String(field)} must be a valid email`): TypedValidator<T> {
    const obj = this.value as Record<string, unknown>;
    if (obj[field] !== undefined && typeof obj[field] === 'string') {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(obj[field] as string)) {
        this.errors.push({ field: String(field), message, value: obj[field] });
      }
    }
    return this;
  }
  
  min<K extends keyof T>(field: K, minLength: number, message: string = `${String(field)} is too short`): TypedValidator<T> {
    const obj = this.value as Record<string, unknown>;
    if (obj[field] !== undefined && typeof obj[field] === 'string') {
      if ((obj[field] as string).length < minLength) {
        this.errors.push({ field: String(field), message, value: obj[field] });
      }
    }
    return this;
  }
  
  validate(): ValidationResult<T> {
    if (this.errors.length > 0) {
      return { isValid: false, errors: this.errors.map(e => e.message) };
    }
    
    // Если ошибок нет, возвращаем значение с типом T
    return { 
      isValid: true, 
      errors: [], 
      value: this.value as T 
    };
  }
}

// Пример использования
interface ContactForm {
  name: string;
  email: string;
  message: string;
}

const contactData = { name: 'John', email: 'invalid-email', message: 'Hi' };
const contactValidation = TypedValidator
  .create<ContactForm>(contactData)
  .required('name')
  .required('email')
  .required('message')
  .string('name')
  .email('email')
  .min('message', 10)
  .validate();

console.log(contactValidation);
```

### Валидация с использованием Zod (альтернативный подход)

```typescript
// Хотя Zod - внешняя библиотека, можно создать похожий подход
interface ZodLikeSchema<T> {
  parse: (data: unknown) => T;
  safeParse: (data: unknown) => { success: true; data: T } | { success: false; error: string };
}

// Простая реализация подобия Zod
class StringType {
  email(): ZodLikeSchema<string> {
    return {
      parse: (data) => {
        if (typeof data !== 'string' || !data.includes('@')) {
          throw new Error('Invalid email format');
        }
        return data;
      },
      safeParse: (data) => {
        try {
          return { success: true, data: this.parse(data) };
        } catch (error) {
          return { success: false, error: (error as Error).message };
        }
      }
    };
  }
  
  min(length: number): ZodLikeSchema<string> {
    return {
      parse: (data) => {
        if (typeof data !== 'string' || data.length < length) {
          throw new Error(`String must be at least ${length} characters long`);
        }
        return data;
      },
      safeParse: (data) => {
        try {
          return { success: true, data: this.parse(data) };
        } catch (error) {
          return { success: false, error: (error as Error).message };
        }
      }
    };
  }
}

const emailSchema = new StringType().email();
const result = emailSchema.safeParse('test@example.com');
console.log(result); // { success: true, data: 'test@example.com' }
```

## Заключение

Утилиты для валидации с типами обеспечивают безопасность и надежность при работе с данными. Использование TypeScript позволяет создавать мощные валидационные утилиты, которые проверяют данные как во время выполнения, так и на этапе компиляции.

См. также: [[api-type-definitions]] | [[react-component-types]] | [[type-guards-advanced]]
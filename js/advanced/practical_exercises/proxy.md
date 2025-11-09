# Упражнения по Proxy и Reflect

## Упражнение 5: Создание системы валидации с Proxy
Создайте систему валидации данных с использованием Proxy:

```javascript
// validation-proxy.js - система валидации с Proxy
class ValidationProxy {
  constructor(target, schema) {
    this.target = target;
    this.schema = schema;
    this.errors = {};
    
    return new Proxy(target, {
      set: (obj, prop, value) => {
        return this._validateAndSet(obj, prop, value);
      },
      get: (obj, prop) => {
        // Возвращаем ошибки валидации
        if (prop === 'errors') {
          return this.errors;
        }
        
        // Возвращаем метод для проверки валидности
        if (prop === 'isValid') {
          return () => Object.keys(this.errors).length === 0;
        }
        
        // Возвращаем метод для очистки ошибок
        if (prop === 'clearErrors') {
          return () => {
            this.errors = {};
          };
        }
        
        return obj[prop];
      }
    });
  }
  
  _validateAndSet(obj, prop, value) {
    // Если нет схемы для этого свойства, просто устанавливаем значение
    if (!this.schema[prop]) {
      obj[prop] = value;
      return true;
    }
    
    const rules = this.schema[prop];
    const errors = [];
    
    // Проверка required
    if (rules.required && (value === undefined || value === null || value === '')) {
      errors.push('Поле обязательно для заполнения');
    }
    
    // Проверка типа
    if (value !== undefined && value !== null && rules.type) {
      if (!this._checkType(value, rules.type)) {
        errors.push(`Ожидается тип ${rules.type}`);
      }
    }
    
    // Проверка минимальной длины
    if (rules.minLength !== undefined && value !== undefined && value !== null) {
      if (typeof value === 'string' || Array.isArray(value)) {
        if (value.length < rules.minLength) {
          errors.push(`Минимальная длина ${rules.minLength} символов`);
        }
      }
    }
    
    // Проверка максимальной длины
    if (rules.maxLength !== undefined && value !== undefined && value !== null) {
      if (typeof value === 'string' || Array.isArray(value)) {
        if (value.length > rules.maxLength) {
          errors.push(`Максимальная длина ${rules.maxLength} символов`);
        }
      }
    }
    
    // Проверка минимального значения
    if (rules.min !== undefined && value !== undefined && value !== null) {
      if (typeof value === 'number') {
        if (value < rules.min) {
          errors.push(`Минимальное значение ${rules.min}`);
        }
      }
    }
    
    // Проверка максимального значения
    if (rules.max !== undefined && value !== undefined && value !== null) {
      if (typeof value === 'number') {
        if (value > rules.max) {
          errors.push(`Максимальное значение ${rules.max}`);
        }
      }
    }
    
    // Проверка регулярного выражения
    if (rules.pattern && value !== undefined && value !== null) {
      if (typeof value === 'string') {
        if (!rules.pattern.test(value)) {
          errors.push(rules.patternMessage || 'Неверный формат');
        }
      }
    }
    
    // Проверка пользовательской функции
    if (rules.validator && typeof rules.validator === 'function') {
      const result = rules.validator(value);
      if (result !== true) {
        errors.push(typeof result === 'string' ? result : 'Неверное значение');
      }
    }
    
    // Сохраняем ошибки или очищаем их
    if (errors.length > 0) {
      this.errors[prop] = errors;
    } else {
      delete this.errors[prop];
      obj[prop] = value;
    }
    
    return true;
  }
  
  _checkType(value, expectedType) {
    switch (expectedType) {
      case 'string':
        return typeof value === 'string';
      case 'number':
        return typeof value === 'number' && !isNaN(value);
      case 'boolean':
        return typeof value === 'boolean';
      case 'array':
        return Array.isArray(value);
      case 'object':
        return typeof value === 'object' && value !== null && !Array.isArray(value);
      case 'email':
        return typeof value === 'string' && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
      default:
        return true;
    }
  }
}

// observable-proxy.js - наблюдаемый объект с Proxy
class ObservableProxy {
  constructor(target, observers = []) {
    this.target = target;
    this.observers = observers;
    
    return new Proxy(target, {
      set: (obj, prop, value) => {
        const oldValue = obj[prop];
        obj[prop] = value;
        
        // Уведомляем наблюдателей об изменении
        this._notifyObservers(prop, value, oldValue);
        
        return true;
      },
      deleteProperty: (obj, prop) => {
        const oldValue = obj[prop];
        delete obj[prop];
        
        // Уведомляем наблюдателей об удалении
        this._notifyObservers(prop, undefined, oldValue, 'delete');
        
        return true;
      }
    });
  }
  
  // Добавление наблюдателя
  subscribe(observer) {
    if (typeof observer === 'function') {
      this.observers.push(observer);
    }
  }
  
  // Удаление наблюдателя
  unsubscribe(observer) {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }
  
  // Уведомление наблюдателей
  _notifyObservers(prop, newValue, oldValue, operation = 'set') {
    for (const observer of this.observers) {
      try {
        observer({
          property: prop,
          newValue,
          oldValue,
          operation,
          timestamp: Date.now()
        });
      } catch (error) {
        console.error('Ошибка в наблюдателе:', error);
      }
    }
  }
}

// Тестирование Proxy
function testProxy() {
  console.log('=== Тестирование Proxy ===');
  
  // Тестирование валидации
  console.log('Тестирование валидации:');
  const userSchema = {
    name: {
      required: true,
      type: 'string',
      minLength: 2,
      maxLength: 50
    },
    email: {
      required: true,
      type: 'email'
    },
    age: {
      type: 'number',
      min: 0,
      max: 150
    },
    password: {
      required: true,
      minLength: 8,
      validator: (value) => {
        if (!/[A-Z]/.test(value)) {
          return 'Пароль должен содержать заглавную букву';
        }
        if (!/[0-9]/.test(value)) {
          return 'Пароль должен содержать цифру';
        }
        return true;
      }
    }
  };
  
  const user = new ValidationProxy({}, userSchema);
  
  // Попытка установки невалидных данных
  user.name = 'И'; // Слишком короткое имя
  user.email = 'invalid-email'; // Неверный формат email
  user.age = -5; // Отрицательный возраст
  user.password = '123'; // Слишком короткий пароль
  
  console.log('Ошибки валидации:', user.errors);
  console.log('Объект валиден:', user.isValid());
  
  // Очистка ошибок и установка валидных данных
  user.clearErrors();
  user.name = 'Иван Иванов';
  user.email = 'ivan@example.com';
  user.age = 30;
  user.password = 'Password123';
  
  console.log('После установки валидных данных:');
  console.log('Ошибки:', user.errors);
  console.log('Объект валиден:', user.isValid());
  console.log('Данные пользователя:', {
    name: user.name,
    email: user.email,
    age: user.age
  });
  
  // Тестирование наблюдаемого объекта
  console.log('\nТестирование наблюдаемого объекта:');
  const observable = new ObservableProxy({
    counter: 0,
    message: 'Привет'
  });
  
  // Добавление наблюдателей
  observable.subscribe((change) => {
    console.log(`Свойство ${change.property} изменилось с ${change.oldValue} на ${change.newValue}`);
  });
  
  observable.subscribe((change) => {
    if (change.property === 'counter') {
      console.log(`Счетчик: ${change.newValue}`);
    }
  });
  
  // Изменение свойств
  observable.counter = 1;
  observable.counter = 2;
  observable.message = 'Пока';
  delete observable.message;
}

// Раскомментируйте для запуска тестов
// testProxy();
```

#javascript #proxy #reflect #validation #advanced
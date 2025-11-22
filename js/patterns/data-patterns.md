---
aliases: ["Паттерны работы с данными", "Data Patterns", "JS Data"]
tags: [js, patterns, data, architecture]
---

# Практические паттерны для работы с данными

## Введение в паттерны работы с данными

Паттерны работы с данными помогают эффективно управлять, преобразовывать и обрабатывать данные в JavaScript приложениях. Они обеспечивают чистоту кода, производительность и поддерживаемость.

## Паттерны преобразования данных

### Цепочка преобразований (Chaining Pattern)

```javascript
// Класс для цепочечных операций с данными
class DataChain {
  constructor(data) {
    this.data = data;
  }
  
  // Фильтрация
  filter(predicate) {
    this.data = this.data.filter(predicate);
    return this;
  }
  
  // Преобразование
  map(transformer) {
    this.data = this.data.map(transformer);
    return this;
  }
  
  // Сортировка
  sort(comparator) {
    this.data.sort(comparator);
    return this;
  }
  
  // Уникальные значения
  unique(keyFn) {
    const seen = new Set();
    this.data = this.data.filter(item => {
      const key = keyFn ? keyFn(item) : item;
      if (seen.has(key)) {
        return false;
      }
      seen.add(key);
      return true;
    });
    return this;
  }
  
  // Пагинация
  paginate(page, pageSize) {
    const start = (page - 1) * pageSize;
    const end = start + pageSize;
    this.data = this.data.slice(start, end);
    return this;
  }
  
  // Получение результата
  value() {
    return this.data;
  }
  
  // Получение статистики
  stats() {
    return {
      count: this.data.length,
      min: this.data.length > 0 ? Math.min(...this.data.map(item => typeof item === 'object' ? item.value : item)) : null,
      max: this.data.length > 0 ? Math.max(...this.data.map(item => typeof item === 'object' ? item.value : item)) : null,
      avg: this.data.length > 0 ? this.data.reduce((sum, item) => sum + (typeof item === 'object' ? item.value : item), 0) / this.data.length : null
    };
  }
}

// Пример использования
const users = [
  { id: 1, name: 'Алексей', age: 25, city: 'Москва', active: true },
  { id: 2, name: 'Мария', age: 30, city: 'СПб', active: false },
  { id: 3, name: 'Иван', age: 35, city: 'Москва', active: true },
  { id: 4, name: 'Елена', age: 28, city: 'Новосибирск', active: true },
  { id: 5, name: 'Сергей', age: 32, city: 'СПб', active: false }
];

const result = new DataChain(users)
  .filter(user => user.active)
  .filter(user => user.age > 25)
  .map(user => ({ ...user, ageGroup: user.age < 30 ? 'молодой' : 'зрелый' }))
  .sort((a, b) => b.age - a.age)
  .value();

console.log(result);
```

### Паттерн "Фасад" для сложных преобразований

```javascript
// Фасад для сложных операций с данными
class DataProcessor {
  constructor(data) {
    this.data = data;
  }
  
  // Комплексная обработка пользовательских данных
  processUserAnalytics(options = {}) {
    const defaults = {
      minAge: 18,
      maxAge: 65,
      includeInactive: false,
      groupByCity: true,
      calculateStats: true
    };
    
    const config = { ...defaults, ...options };
    
    let processedData = [...this.data];
    
    // Фильтрация по возрасту
    if (config.minAge || config.maxAge) {
      processedData = processedData.filter(user => {
        const age = user.age || 0;
        return (!config.minAge || age >= config.minAge) && 
               (!config.maxAge || age <= config.maxAge);
      });
    }
    
    // Фильтрация по активности
    if (!config.includeInactive) {
      processedData = processedData.filter(user => user.active !== false);
    }
    
    // Группировка по городам
    if (config.groupByCity) {
      processedData = this.groupByCity(processedData);
    }
    
    // Расчет статистики
    if (config.calculateStats) {
      processedData = this.calculateStatistics(processedData);
    }
    
    return processedData;
  }
  
  groupByCity(users) {
    const grouped = {};
    users.forEach(user => {
      const city = user.city || 'Неизвестно';
      if (!grouped[city]) {
        grouped[city] = [];
      }
      grouped[city].push(user);
    });
    
    return Object.entries(grouped).map(([city, users]) => ({
      city,
      users,
      count: users.length,
      avgAge: users.reduce((sum, user) => sum + (user.age || 0), 0) / users.length
    }));
  }
  
  calculateStatistics(groupedData) {
    return groupedData.map(group => ({
      ...group,
      ageStats: {
        min: Math.min(...group.users.map(u => u.age)),
        max: Math.max(...group.users.map(u => u.age)),
        avg: group.avgAge
      }
    }));
  }
  
  // Преобразование плоских данных в иерархические
  flatToHierarchical(idField = 'id', parentField = 'parentId', childrenField = 'children') {
    const map = new Map();
    const roots = [];
    
    // Создаем карту всех элементов
    this.data.forEach(item => {
      map.set(item[idField], { ...item, [childrenField]: [] });
    });
    
    // Строим иерархию
    this.data.forEach(item => {
      const currentItem = map.get(item[idField]);
      const parentId = item[parentField];
      
      if (parentId && map.has(parentId)) {
        const parent = map.get(parentId);
        parent[childrenField].push(currentItem);
      } else {
        roots.push(currentItem);
      }
    });
    
    return roots;
  }
}

// Использование
const userData = [
  { id: 1, name: 'Алексей', age: 25, city: 'Москва', active: true },
  { id: 2, name: 'Мария', age: 30, city: 'СПб', active: false },
  { id: 3, name: 'Иван', age: 35, city: 'Москва', active: true }
];

const processor = new DataProcessor(userData);
const analytics = processor.processUserAnalytics({
  minAge: 20,
  groupByCity: true,
  calculateStats: true
});

console.log(analytics);
```

## Паттерны валидации данных

### Валидатор с цепочкой правил

```javascript
class Validator {
  constructor() {
    this.rules = [];
  }
  
  // Добавление правила валидации
  addRule(fieldName, validatorFn, errorMessage) {
    this.rules.push({
      field: fieldName,
      validate: validatorFn,
      message: errorMessage
    });
    return this;
  }
  
  // Валидация объекта
  validate(obj) {
    const errors = {};
    
    for (const rule of this.rules) {
      const value = this.getNestedProperty(obj, rule.field);
      if (!rule.validate(value)) {
        if (!errors[rule.field]) {
          errors[rule.field] = [];
        }
        errors[rule.field].push(rule.message);
      }
    }
    
    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }
  
  // Валидация с условиями
  addConditionalRule(fieldName, conditionField, conditionFn, validatorFn, errorMessage) {
    this.rules.push({
      field: fieldName,
      conditionField,
      condition: conditionFn,
      validate: validatorFn,
      message: errorMessage
    });
    return this;
  }
  
  // Валидация с учетом условий
  validateConditional(obj) {
    const errors = {};
    
    for (const rule of this.rules) {
      // Если есть условие, проверяем его
      if (rule.conditionField) {
        const conditionValue = this.getNestedProperty(obj, rule.conditionField);
        if (!rule.condition(conditionValue)) {
          continue; // Пропускаем правило, если условие не выполнено
        }
      }
      
      const value = this.getNestedProperty(obj, rule.field);
      if (!rule.validate(value)) {
        if (!errors[rule.field]) {
          errors[rule.field] = [];
        }
        errors[rule.field].push(rule.message);
      }
    }
    
    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }
  
  // Вспомогательный метод для получения вложенных свойств
  getNestedProperty(obj, path) {
    return path.split('.').reduce((current, key) => current && current[key], obj);
  }
}

// Создание валидатора для пользователя
const userValidator = new Validator()
  .addRule('name', value => typeof value === 'string' && value.length >= 2, 'Имя должно содержать не менее 2 символов')
  .addRule('email', value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value), 'Введите корректный email')
  .addRule('age', value => typeof value === 'number' && value >= 0 && value <= 120, 'Возраст должен быть от 0 до 120')
  .addRule('phone', value => /^(\+7|8)[\d\s\-\(\)]{10,15}$/.test(value), 'Введите корректный номер телефона')
  .addConditionalRule('emergencyContact', 'age', age => age < 18, value => !!value, 'Для несовершеннолетних требуется контакт экстренной связи');

// Проверка данных
const userData = {
  name: 'Иван',
  email: 'ivan@example.com',
  age: 16,
  phone: '+79991234567'
  // emergencyContact отсутствует, что вызовет ошибку для несовершеннолетнего
};

const validation = userValidator.validateConditional(userData);
console.log('Валидация:', validation);
```

### Схема валидации

```javascript
class Schema {
  constructor(definition) {
    this.definition = definition;
  }
  
  validate(data) {
    const errors = this.validateField(data, this.definition, '');
    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }
  
  validateField(data, schema, path) {
    const errors = {};
    
    for (const [key, fieldSchema] of Object.entries(schema)) {
      const currentValue = data[key];
      const currentPath = path ? `${path}.${key}` : key;
      
      // Проверка на обязательность
      if (fieldSchema.required && (currentValue === undefined || currentValue === null)) {
        errors[currentPath] = fieldSchema.requiredMessage || `Поле ${currentPath} обязательно`;
        continue;
      }
      
      // Пропускаем проверку, если значение не задано и не обязательно
      if (currentValue === undefined || currentValue === null) {
        continue;
      }
      
      // Проверка типа
      if (fieldSchema.type) {
        const typeError = this.validateType(currentValue, fieldSchema.type, currentPath);
        if (typeError) {
          errors[currentPath] = typeError;
          continue;
        }
      }
      
      // Проверка пользовательской функцией
      if (fieldSchema.validator) {
        const customError = fieldSchema.validator(currentValue);
        if (customError) {
          errors[currentPath] = customError;
          continue;
        }
      }
      
      // Проверка вложенной схемы
      if (fieldSchema.schema) {
        const nestedErrors = this.validateField(currentValue, fieldSchema.schema, currentPath);
        Object.assign(errors, nestedErrors);
      }
      
      // Проверка массива объектов
      if (fieldSchema.arrayOf && Array.isArray(currentValue)) {
        currentValue.forEach((item, index) => {
          const itemPath = `${currentPath}[${index}]`;
          const itemErrors = this.validateField(item, fieldSchema.arrayOf, itemPath);
          Object.assign(errors, itemErrors);
        });
      }
    }
    
    return errors;
  }
  
  validateType(value, expectedType, path) {
    switch (expectedType) {
      case 'string':
        return typeof value !== 'string' ? `Поле ${path} должно быть строкой` : null;
      case 'number':
        return typeof value !== 'number' || isNaN(value) ? `Поле ${path} должно быть числом` : null;
      case 'boolean':
        return typeof value !== 'boolean' ? `Поле ${path} должно быть булевым значением` : null;
      case 'object':
        return typeof value !== 'object' || value === null || Array.isArray(value) ? `Поле ${path} должно быть объектом` : null;
      case 'array':
        return !Array.isArray(value) ? `Поле ${path} должно быть массивом` : null;
      default:
        return null;
    }
  }
}

// Определение схемы
const userSchema = new Schema({
  name: {
    type: 'string',
    required: true,
    requiredMessage: 'Имя пользователя обязательно'
  },
  age: {
    type: 'number',
    required: true,
    validator: (value) => {
      if (value < 0 || value > 120) return 'Возраст должен быть от 0 до 120';
      return null;
    }
  },
  email: {
    type: 'string',
    required: true,
    validator: (value) => {
      if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) return 'Некорректный email';
      return null;
    }
  },
  address: {
    schema: {
      street: { type: 'string', required: true },
      city: { type: 'string', required: true },
      zipCode: {
        type: 'string',
        validator: (value) => {
          if (!/^\d{6}$/.test(value)) return 'Индекс должен содержать 6 цифр';
          return null;
        }
      }
    }
  },
  hobbies: {
    type: 'array',
    arrayOf: {
      name: { type: 'string', required: true },
      level: { 
        type: 'string', 
        validator: (value) => {
          if (!['начинающий', 'средний', 'продвинутый'].includes(value)) return 'Некорректный уровень';
          return null;
        }
      }
    }
  }
});

// Проверка данных
const userData = {
  name: 'Иван Иванов',
  age: 30,
  email: 'ivan@example.com',
  address: {
    street: 'Ленина 123',
    city: 'Москва',
    zipCode: '123456'
  },
  hobbies: [
    { name: 'Теннис', level: 'средний' },
    { name: 'Плавание', level: 'начинающий' }
  ]
};

const result = userSchema.validate(userData);
console.log('Результат валидации:', result);
```

## Паттерны нормализации данных

### Нормализатор данных

```javascript
class DataNormalizer {
  constructor() {
    this.normalizers = new Map();
  }
  
  // Регистрация нормализатора для типа данных
  register(type, normalizerFn) {
    this.normalizers.set(type, normalizerFn);
    return this;
  }
  
  // Нормализация данных по схеме
  normalize(data, schema) {
    if (Array.isArray(data)) {
      return data.map(item => this.normalizeObject(item, schema));
    }
    return this.normalizeObject(data, schema);
  }
  
  normalizeObject(obj, schema) {
    const normalized = {};
    
    for (const [key, fieldSchema] of Object.entries(schema)) {
      const value = obj[key];
      
      if (value === undefined || value === null) {
        // Установка значения по умолчанию
        if (fieldSchema.default !== undefined) {
          normalized[key] = fieldSchema.default;
        }
        continue;
      }
      
      // Применение нормализатора
      if (fieldSchema.type && this.normalizers.has(fieldSchema.type)) {
        normalized[key] = this.normalizers.get(fieldSchema.type)(value);
      } else if (fieldSchema.schema) {
        // Рекурсивная нормализация вложенных объектов
        normalized[key] = this.normalizeObject(value, fieldSchema.schema);
      } else {
        normalized[key] = value;
      }
    }
    
    return normalized;
  }
}

// Создание нормализатора
const normalizer = new DataNormalizer()
  .register('string', (value) => String(value).trim())
  .register('number', (value) => {
    const num = Number(value);
    return isNaN(num) ? 0 : num;
  })
  .register('boolean', (value) => Boolean(value))
  .register('date', (value) => new Date(value).toISOString())
  .register('email', (value) => String(value).toLowerCase().trim());

// Схема нормализации
const userSchema = {
  id: { type: 'number', default: 0 },
  name: { type: 'string', default: 'Аноним' },
  email: { type: 'email' },
  age: { type: 'number' },
  active: { type: 'boolean', default: false },
  createdAt: { type: 'date' },
  profile: {
    schema: {
      firstName: { type: 'string' },
      lastName: { type: 'string' },
      bio: { type: 'string', default: '' }
    }
  }
};

// Нормализация данных
const rawData = {
  id: '123',
  name: '  Иван Иванов  ',
  email: ' IVAN@EXAMPLE.COM  ',
  age: '30',
  active: 'true',
  createdAt: '2023-01-01',
  profile: {
    firstName: '  Иван  ',
    lastName: '  Иванов  ',
    bio: null
  }
};

const normalizedData = normalizer.normalize(rawData, userSchema);
console.log('Нормализованные данные:', normalizedData);
```

## Паттерны кэширования данных

### Кэш с TTL

```javascript
class TTLCache {
  constructor(defaultTTL = 5 * 60 * 1000) { // 5 минут по умолчанию
    this.cache = new Map();
    this.defaultTTL = defaultTTL;
    this.cleanupInterval = null;
  }
  
  // Установка значения с TTL
  set(key, value, ttl = this.defaultTTL) {
    const expiry = Date.now() + ttl;
    this.cache.set(key, { value, expiry });
    return this;
  }
  
  // Получение значения
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) {
      return undefined;
    }
    
    // Проверка срока действия
    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return undefined;
    }
    
    return item.value;
  }
  
  // Проверка наличия ключа
  has(key) {
    const item = this.cache.get(key);
    if (!item) return false;
    
    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return false;
    }
    
    return true;
  }
  
  // Удаление ключа
  delete(key) {
    return this.cache.delete(key);
  }
  
  // Очистка кэша
  clear() {
    this.cache.clear();
  }
  
  // Запуск автоматической очистки
  startCleanup(interval = 60 * 1000) { // раз в минуту
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval);
    }
    
    this.cleanupInterval = setInterval(() => {
      this.cleanup();
    }, interval);
  }
  
  // Остановка автоматической очистки
  stopCleanup() {
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval);
      this.cleanupInterval = null;
    }
  }
  
  // Ручная очистка устаревших записей
  cleanup() {
    const now = Date.now();
    for (const [key, item] of this.cache.entries()) {
      if (now > item.expiry) {
        this.cache.delete(key);
      }
    }
  }
  
  // Получение статистики
  stats() {
    const now = Date.now();
    let validCount = 0;
    let expiredCount = 0;
    let totalTTL = 0;
    
    for (const item of this.cache.values()) {
      if (now > item.expiry) {
        expiredCount++;
      } else {
        validCount++;
        totalTTL += item.expiry - now;
      }
    }
    
    return {
      total: this.cache.size,
      valid: validCount,
      expired: expiredCount,
      avgTTL: validCount > 0 ? totalTTL / validCount : 0
    };
  }
}

// Использование кэша
const cache = new TTLCache();

// Сохранение данных с разными TTL
cache.set('user_123', { id: 123, name: 'Иван' }, 10000); // 10 секунд
cache.set('config', { theme: 'dark', lang: 'ru' }, 60000); // 1 минута

// Получение данных
const user = cache.get('user_123');
console.log('Пользователь:', user);

// Запуск автоматической очистки
cache.startCleanup();
```

### Кэш с зависимостями

```javascript
class DependencyCache {
  constructor() {
    this.cache = new Map();
    this.dependencies = new Map(); // карта зависимостей
    this.dependents = new Map();   // карта обратных зависимостей
  }
  
  // Установка значения с зависимостями
  set(key, value, dependencies = []) {
    this.cache.set(key, value);
    
    // Удаление старых зависимостей
    if (this.dependents.has(key)) {
      for (const dep of this.dependents.get(key)) {
        const dependents = this.dependencies.get(dep);
        if (dependents) {
          dependents.delete(key);
        }
      }
    }
    
    // Установка новых зависимостей
    if (dependencies.length > 0) {
      this.dependents.set(key, new Set(dependencies));
      
      for (const dep of dependencies) {
        if (!this.dependencies.has(dep)) {
          this.dependencies.set(dep, new Set());
        }
        this.dependencies.get(dep).add(key);
      }
    }
    
    return this;
  }
  
  // Получение значения
  get(key) {
    return this.cache.get(key);
  }
  
  // Инвалидация по зависимости
  invalidate(key) {
    // Инвалидация самого ключа
    this.cache.delete(key);
    
    // Инвалидация всех зависимых ключей
    if (this.dependencies.has(key)) {
      const dependents = this.dependencies.get(key);
      for (const dependent of dependents) {
        this.invalidate(dependent);
      }
    }
    
    // Удаление зависимостей
    if (this.dependents.has(key)) {
      for (const dep of this.dependents.get(key)) {
        const dependents = this.dependencies.get(dep);
        if (dependents) {
          dependents.delete(key);
        }
      }
      this.dependents.delete(key);
    }
  }
  
  // Очистка всего кэша
  clear() {
    this.cache.clear();
    this.dependencies.clear();
    this.dependents.clear();
  }
}

// Использование кэша с зависимостями
const depCache = new DependencyCache();

// Установка данных с зависимостями
depCache.set('user_123', { id: 123, name: 'Иван' });
depCache.set('user_123_profile', { bio: 'Программист', skills: ['JS', 'React'] }, ['user_123']);
depCache.set('user_123_dashboard', { widgets: ['stats', 'tasks'] }, ['user_123', 'user_123_profile']);

// При изменении user_123 будут инвалидированы profile и dashboard
depCache.invalidate('user_123');
```

## Паттерны агрегации данных

### Агрегатор данных

```javascript
class DataAggregator {
  constructor(data) {
    this.data = data;
  }
  
  // Группировка по полю
  groupBy(field) {
    const groups = {};
    
    this.data.forEach(item => {
      const key = this.getNestedProperty(item, field);
      if (!groups[key]) {
        groups[key] = [];
      }
      groups[key].push(item);
    });
    
    return groups;
  }
  
  // Агрегация с пользовательской функцией
  aggregate(groupField, aggregateFn) {
    const groups = this.groupBy(groupField);
    const result = {};
    
    for (const [key, items] of Object.entries(groups)) {
      result[key] = aggregateFn(items, key);
    }
    
    return result;
  }
  
  // Стандартные агрегации
  countBy(field) {
    return this.aggregate(field, items => items.length);
  }
  
  sumBy(groupField, valueField) {
    return this.aggregate(groupField, items => 
      items.reduce((sum, item) => sum + (this.getNestedProperty(item, valueField) || 0), 0)
    );
  }
  
  avgBy(groupField, valueField) {
    return this.aggregate(groupField, items => {
      const sum = items.reduce((sum, item) => sum + (this.getNestedProperty(item, valueField) || 0), 0);
      return sum / items.length;
    });
  }
  
  maxBy(groupField, valueField) {
    return this.aggregate(groupField, items => {
      return Math.max(...items.map(item => this.getNestedProperty(item, valueField)));
    });
  }
  
  minBy(groupField, valueField) {
    return this.aggregate(groupField, items => {
      return Math.min(...items.map(item => this.getNestedProperty(item, valueField)));
    });
  }
  
  // Агрегация с фильтрацией
  aggregateWithFilter(groupField, filterFn, aggregateFn) {
    const filteredData = this.data.filter(filterFn);
    const aggregator = new DataAggregator(filteredData);
    return aggregator.aggregate(groupField, aggregateFn);
  }
  
  // Вспомогательный метод для получения вложенных свойств
  getNestedProperty(obj, path) {
    return path.split('.').reduce((current, key) => current && current[key], obj);
  }
}

// Пример использования
const salesData = [
  { product: 'Ноутбук', category: 'Электроника', price: 50000, quantity: 2, date: '2023-01-15' },
  { product: 'Мышь', category: 'Электроника', price: 1000, quantity: 5, date: '2023-01-16' },
  { product: 'Книга', category: 'Книги', price: 500, quantity: 3, date: '2023-01-17' },
  { product: 'Ноутбук', category: 'Электроника', price: 55000, quantity: 1, date: '2023-01-18' },
  { product: 'Книга', category: 'Книги', price: 700, quantity: 2, date: '2023-01-19' }
];

const aggregator = new DataAggregator(salesData);

// Агрегации
console.log('Количество по категориям:', aggregator.countBy('category'));
console.log('Сумма продаж по категориям:', aggregator.sumBy('category', 'price'));
console.log('Средняя цена по категориям:', aggregator.avgBy('category', 'price'));
console.log('Общее количество по продуктам:', aggregator.sumBy('product', 'quantity'));
```

## Практические примеры

### Паттерн "Репозиторий" для работы с данными

```javascript
class Repository {
  constructor(adapter) {
    this.adapter = adapter;
  }
  
  async find(query = {}) {
    return await this.adapter.find(query);
  }
  
  async findOne(query = {}) {
    const results = await this.find(query);
    return results[0] || null;
  }
  
  async findById(id) {
    return await this.findOne({ id });
  }
  
  async create(data) {
    return await this.adapter.create(data);
  }
  
  async update(id, data) {
    return await this.adapter.update(id, data);
  }
  
  async delete(id) {
    return await this.adapter.delete(id);
  }
  
  async count(query = {}) {
    return await this.adapter.count(query);
  }
  
  async findWithPagination(query = {}, page = 1, limit = 10) {
    const offset = (page - 1) * limit;
    return await this.adapter.findWithPagination(query, offset, limit);
  }
}

// Адаптер для работы с разными источниками данных
class InMemoryAdapter {
  constructor() {
    this.data = [];
    this.nextId = 1;
  }
  
  async find(query = {}) {
    let results = [...this.data];
    
    // Простая фильтрация
    for (const [key, value] of Object.entries(query)) {
      results = results.filter(item => item[key] === value);
    }
    
    return results;
  }
  
  async create(data) {
    const item = { ...data, id: this.nextId++ };
    this.data.push(item);
    return item;
  }
  
  async update(id, data) {
    const index = this.data.findIndex(item => item.id === id);
    if (index !== -1) {
      this.data[index] = { ...this.data[index], ...data };
      return this.data[index];
    }
    return null;
  }
  
  async delete(id) {
    const index = this.data.findIndex(item => item.id === id);
    if (index !== -1) {
      return this.data.splice(index, 1)[0];
    }
    return null;
  }
  
  async count(query = {}) {
    return (await this.find(query)).length;
  }
  
  async findWithPagination(query = {}, offset = 0, limit = 10) {
    const all = await this.find(query);
    return all.slice(offset, offset + limit);
  }
}

// Использование репозитория
const userAdapter = new InMemoryAdapter();
const userRepository = new Repository(userAdapter);

// Создание пользователей
await userRepository.create({ name: 'Иван', email: 'ivan@example.com', age: 30 });
await userRepository.create({ name: 'Мария', email: 'maria@example.com', age: 25 });

// Поиск пользователей
const users = await userRepository.find({ age: { $gte: 25 } });
const user = await userRepository.findById(1);
```

### Паттерн "Спецификация" для сложных запросов

```javascript
// Базовый класс спецификации
class Specification {
  constructor() {
    this.criteria = [];
  }
  
  // Добавление критерия
  addCriteria(predicate) {
    this.criteria.push(predicate);
    return this;
  }
  
  // Проверка соответствия объекта спецификации
  isSatisfiedBy(obj) {
    return this.criteria.every(criterion => criterion(obj));
  }
  
  // Комбинация спецификаций
  and(otherSpec) {
    const combinedSpec = new Specification();
    combinedSpec.criteria = [...this.criteria, ...otherSpec.criteria];
    return combinedSpec;
  }
  
  or(otherSpec) {
    return new OrSpecification(this, otherSpec);
  }
}

// Спецификация для ИЛИ условий
class OrSpecification {
  constructor(spec1, spec2) {
    this.spec1 = spec1;
    this.spec2 = spec2;
  }
  
  isSatisfiedBy(obj) {
    return this.spec1.isSatisfiedBy(obj) || this.spec2.isSatisfiedBy(obj);
  }
}

// Предопределенные спецификации
class UserSpecifications {
  static adult() {
    return new Specification().addCriteria(user => user.age >= 18);
  }
  
  static active() {
    return new Specification().addCriteria(user => user.active === true);
  }
  
  static fromCity(city) {
    return new Specification().addCriteria(user => user.city === city);
  }
  
  static ageBetween(min, max) {
    return new Specification()
      .addCriteria(user => user.age >= min)
      .addCriteria(user => user.age <= max);
  }
  
  static hasSkill(skill) {
    return new Specification().addCriteria(user => 
      Array.isArray(user.skills) && user.skills.includes(skill)
    );
  }
}

// Использование спецификаций
const users = [
  { id: 1, name: 'Иван', age: 25, city: 'Москва', active: true, skills: ['JS', 'React'] },
  { id: 2, name: 'Мария', age: 17, city: 'СПб', active: true, skills: ['Python'] },
  { id: 3, name: 'Сергей', age: 35, city: 'Москва', active: false, skills: ['JS', 'Node'] }
];

// Комбинирование спецификаций
const adultActiveMoscowSpec = UserSpecifications.adult()
  .and(UserSpecifications.active())
  .and(UserSpecifications.fromCity('Москва'));

const filteredUsers = users.filter(user => adultActiveMoscowSpec.isSatisfiedBy(user));
console.log('Взрослые активные пользователи из Москвы:', filteredUsers);

// ИЛИ спецификация
const youngOrPythonSpec = UserSpecifications.ageBetween(15, 20)
  .or(UserSpecifications.hasSkill('Python'));

const specialUsers = users.filter(user => youngOrPythonSpec.isSatisfiedBy(user));
console.log('Молодые или Python-разработчики:', specialUsers);
```

## Лучшие практики

### Паттерн "Компоновщик" для сложных преобразований

```javascript
// Интерфейс для операций с данными
class DataOperation {
  execute(data) {
    throw new Error('Метод execute должен быть реализован');
  }
}

// Конкретные операции
class FilterOperation extends DataOperation {
  constructor(predicate) {
    super();
    this.predicate = predicate;
  }
  
  execute(data) {
    return data.filter(this.predicate);
  }
}

class MapOperation extends DataOperation {
  constructor(transformer) {
    super();
    this.transformer = transformer;
  }
  
  execute(data) {
    return data.map(this.transformer);
  }
}

class SortOperation extends DataOperation {
  constructor(comparator) {
    super();
    this.comparator = comparator;
  }
  
  execute(data) {
    return [...data].sort(this.comparator);
  }
}

class CompositeOperation extends DataOperation {
  constructor() {
    super();
    this.operations = [];
  }
  
  add(operation) {
    this.operations.push(operation);
    return this;
  }
  
  execute(data) {
    let result = data;
    for (const operation of this.operations) {
      result = operation.execute(result);
    }
    return result;
  }
}

// Использование компоновщика
const dataProcessor = new CompositeOperation()
  .add(new FilterOperation(user => user.age > 18))
  .add(new MapOperation(user => ({ ...user, ageGroup: user.age < 30 ? 'молодой' : 'зрелый' })))
  .add(new SortOperation((a, b) => a.name.localeCompare(b.name)));

const processedUsers = dataProcessor.execute(users);
console.log('Обработанные пользователи:', processedUsers);
```

## Ссылки по теме

- [[js/patterns/repository]] - Паттерн Репозиторий
- [[js/patterns/specification]] - Паттерн Спецификация
- [[js/data/normalization]] - Нормализация данных
- [[js/data/validation]] - Валидация данных
- [[js/patterns/chain-of-responsibility]] - Цепочка обязанностей
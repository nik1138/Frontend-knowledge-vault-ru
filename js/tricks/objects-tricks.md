---
aliases: ["Хитрости работы с объектами в JavaScript", "Манипуляции с объектами JS"]
tags: [js, objects, tricks, frontend]
---

# Объекты: хитрости при работе с объектами и свойствами

Работа с объектами в JavaScript - ключевой навык для любого разработчика. Ниже приведены полезные хитрости и методы для эффективной работы с объектами и их свойствами.

## 1. Объединение объектов

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const obj3 = { b: 5, e: 6 }; // b будет перезаписан

// Использование spread-оператора
const combined = { ...obj1, ...obj2, ...obj3 };
console.log(combined); // { a: 1, b: 5, c: 3, d: 4, e: 6 }

// Использование Object.assign
const combined2 = Object.assign({}, obj1, obj2, obj3);
console.log(combined2); // { a: 1, b: 5, c: 3, d: 4, e: 6 }

// Глубокое объединение (упрощенная версия)
function deepMerge(target, source) {
  const result = { ...target };
  
  for (const key in source) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      result[key] = deepMerge(result[key] || {}, source[key]);
    } else {
      result[key] = source[key];
    }
  }
  
  return result;
}

const deepObj1 = { a: 1, nested: { x: 1, y: 2 } };
const deepObj2 = { b: 2, nested: { y: 3, z: 4 } };
console.log(deepMerge(deepObj1, deepObj2)); 
// { a: 1, b: 2, nested: { x: 1, y: 3, z: 4 } }
```

## 2. Проверка свойств объекта

```javascript
const obj = { name: 'Alice', age: 25, city: 'New York' };

// Проверка существования свойства
console.log('name' in obj); // true
console.log(obj.hasOwnProperty('name')); // true

// Проверка с использованием undefined
console.log(obj.name !== undefined); // true
console.log(obj.missing !== undefined); // false

// Проверка всех свойств (включая унаследованные)
console.log(Object.keys(obj)); // ['name', 'age', 'city']
console.log(Object.getOwnPropertyNames(obj)); // ['name', 'age', 'city']

// Получение всех перечисляемых свойств
for (const key in obj) {
  if (obj.hasOwnProperty(key)) {
    console.log(key, obj[key]);
  }
}
```

## 3. Деструктуризация объектов

```javascript
const user = {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com',
  profile: {
    age: 25,
    country: 'USA'
  }
};

// Простая деструктуризация
const { name, email } = user;
console.log(name, email); // 'Alice', 'alice@example.com'

// Деструктуризация с переименованием
const { name: userName, email: userEmail } = user;
console.log(userName, userEmail); // 'Alice', 'alice@example.com'

// Деструктуризация с значениями по умолчанию
const { name: defaultName = 'Unknown', age = 0 } = user;
console.log(defaultName, age); // 'Alice', 0

// Глубокая деструктуризация
const { profile: { age, country } } = user;
console.log(age, country); // 25, 'USA'

// Деструктуризация с оставшимися свойствами
const { id, ...rest } = user;
console.log(id); // 1
console.log(rest); // { name: 'Alice', email: 'alice@example.com', profile: { ... } }

// Деструктуризация в параметрах функции
function greetUser({ name, email = 'no-email@example.com' }) {
  console.log(`Привет, ${name}! Твой email: ${email}`);
}

greetUser(user); // Привет, Alice! Твой email: alice@example.com
```

## 4. Создание объектов с динамическими ключами

```javascript
// Динамические ключи
const keyName = 'dynamicKey';
const obj = {
  [keyName]: 'Значение динамического ключа',
  [keyName + '2']: 'Еще одно значение',
  [`computed_${Date.now()}`]: 'Временное значение'
};

console.log(obj);
// { dynamicKey: 'Значение динамического ключа', dynamicKey2: 'Еще одно значение', computed_1234567890: 'Временное значение' }

// Создание объекта из массива пар [ключ, значение]
const pairs = [['name', 'Alice'], ['age', 25], ['city', 'New York']];
const objFromPairs = Object.fromEntries(pairs);
console.log(objFromPairs); // { name: 'Alice', age: 25, city: 'New York' }

// Создание объекта из массива с вычисляемыми ключами
const prefixes = ['user', 'admin', 'guest'];
const permissions = prefixes.map(prefix => [`${prefix}_permissions`, `can_${prefix}`]);
const permissionsObj = Object.fromEntries(permissions);
console.log(permissionsObj);
// { user_permissions: 'can_user', admin_permissions: 'can_admin', guest_permissions: 'can_guest' }
```

## 5. Фильтрация и преобразование объектов

```javascript
const user = {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com',
  password: 'secret123',
  age: 25,
  city: 'New York'
};

// Фильтрация объекта по ключам
function pick(obj, keys) {
  return keys.reduce((result, key) => {
    if (key in obj) {
      result[key] = obj[key];
    }
    return result;
  }, {});
}

const publicUser = pick(user, ['id', 'name', 'email', 'age']);
console.log(publicUser); // { id: 1, name: 'Alice', email: 'alice@example.com', age: 25 }

// Исключение ключей из объекта
function omit(obj, keys) {
  const result = { ...obj };
  keys.forEach(key => delete result[key]);
  return result;
}

const userWithoutPassword = omit(user, ['password']);
console.log(userWithoutPassword); 
// { id: 1, name: 'Alice', email: 'alice@example.com', age: 25, city: 'New York' }

// Преобразование значений объекта
function mapValues(obj, fn) {
  const result = {};
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      result[key] = fn(obj[key], key);
    }
  }
  return result;
}

const userWithUppercase = mapValues(user, (value, key) => 
  typeof value === 'string' ? value.toUpperCase() : value
);
console.log(userWithUppercase);
// { id: 1, name: 'ALICE', email: 'ALICE@EXAMPLE.COM', password: 'SECRET123', age: 25, city: 'NEW YORK' }
```

## 6. Работа с вложенными объектами

```javascript
const complexObject = {
  user: {
    profile: {
      personal: {
        name: 'Alice',
        age: 25
      },
      contact: {
        email: 'alice@example.com',
        phone: '+1234567890'
      }
    },
    settings: {
      theme: 'dark',
      notifications: true
    }
  }
};

// Безопасное получение вложенного свойства
function getNested(obj, path, defaultValue = undefined) {
  const keys = path.split('.');
  let result = obj;
  
  for (const key of keys) {
    if (result === null || result === undefined) {
      return defaultValue;
    }
    result = result[key];
  }
  
  return result !== undefined ? result : defaultValue;
}

console.log(getNested(complexObject, 'user.profile.personal.name')); // 'Alice'
console.log(getNested(complexObject, 'user.profile.missing.key', 'default')); // 'default'

// Установка вложенного свойства
function setNested(obj, path, value) {
  const keys = path.split('.');
  let current = obj;
  
  for (let i = 0; i < keys.length - 1; i++) {
    if (!(keys[i] in current)) {
      current[keys[i]] = {};
    }
    current = current[keys[i]];
  }
  
  current[keys[keys.length - 1]] = value;
  return obj;
}

const newObject = {};
setNested(newObject, 'user.profile.name', 'Bob');
console.log(newObject); // { user: { profile: { name: 'Bob' } } }

// Обновление вложенных свойств
function updateNested(obj, path, updater) {
  const keys = path.split('.');
  let current = obj;
  
  for (let i = 0; i < keys.length - 1; i++) {
    if (!(keys[i] in current)) {
      current[keys[i]] = {};
    }
    current = current[keys[i]];
  }
  
  const lastKey = keys[keys.length - 1];
  current[lastKey] = updater(current[lastKey]);
  return obj;
}

updateNested(complexObject, 'user.profile.personal.age', age => age + 1);
console.log(complexObject.user.profile.personal.age); // 26
```

## 7. Работа с прототипами и наследованием

```javascript
// Создание объекта с определенным прототипом
const parent = { type: 'parent', parentMethod: () => 'from parent' };
const child = Object.create(parent);
child.name = 'child';
child.childMethod = () => 'from child';

console.log(child.type); // 'parent'
console.log(child.parentMethod()); // 'from parent'
console.log(child.name); // 'child'

// Проверка принадлежности свойства
console.log(child.hasOwnProperty('name')); // true
console.log(child.hasOwnProperty('type')); // false
console.log('type' in child); // true

// Получение прототипа объекта
console.log(Object.getPrototypeOf(child) === parent); // true

// Установка прототипа
const newParent = { newType: 'newParent' };
Object.setPrototypeOf(child, newParent);
console.log(child.newType); // 'newParent'
// console.log(child.type); // undefined (больше не наследуется от старого родителя)
```

## 8. Работа с дескрипторами свойств

```javascript
const obj = {};

// Определение свойства с дескриптором
Object.defineProperty(obj, 'readOnly', {
  value: 'Это значение только для чтения',
  writable: false, // нельзя изменить
  enumerable: true, // будет видно в перечислении
  configurable: false // нельзя удалить или изменить дескриптор
});

console.log(obj.readOnly); // 'Это значение только для чтения'
obj.readOnly = 'Новое значение';
console.log(obj.readOnly); // 'Это значение только для чтения' (не изменилось)

// Определение свойства с геттером и сеттером
let internalValue = 'initial';
Object.defineProperty(obj, 'computedValue', {
  get() {
    return internalValue.toUpperCase();
  },
  set(newValue) {
    internalValue = newValue.toLowerCase();
  },
  enumerable: true,
  configurable: true
});

console.log(obj.computedValue); // 'INITIAL'
obj.computedValue = 'New Value';
console.log(obj.computedValue); // 'NEW VALUE'
console.log(internalValue); // 'new value'

// Получение дескриптора свойства
console.log(Object.getOwnPropertyDescriptor(obj, 'readOnly'));
// { value: 'Это значение только для чтения', writable: false, enumerable: true, configurable: false }
```

## 9. Создание неизменяемых объектов

```javascript
// Создание поверхностно неизменяемого объекта
const obj = { a: 1, b: { c: 2 } };
Object.freeze(obj);

// Попытка изменить свойство
obj.a = 10;
console.log(obj.a); // 1 (не изменилось)

// Попытка добавить новое свойство
obj.newProp = 'new';
console.log(obj.newProp); // undefined (не добавилось)

// ВНИМАНИЕ: вложенные объекты по-прежнему можно изменить
obj.b.c = 20;
console.log(obj.b.c); // 20 (изменилось!)

// Глубокая заморозка
function deepFreeze(obj) {
  Object.getOwnPropertyNames(obj).forEach(prop => {
    if (obj[prop] !== null && typeof obj[prop] === 'object') {
      deepFreeze(obj[prop]);
    }
  });
  return Object.freeze(obj);
}

const deepFrozenObj = deepFreeze({ a: 1, b: { c: 2 } });
// deepFrozenObj.b.c = 20; // Ошибка в strict mode
```

## 10. Преобразование объектов в другие форматы

```javascript
const user = { name: 'Alice', age: 25, active: true };

// Преобразование в массив пар [ключ, значение]
console.log(Object.entries(user)); 
// [['name', 'Alice'], ['age', 25], ['active', true]]

// Преобразование ключей в массив
console.log(Object.keys(user)); // ['name', 'age', 'active']

// Преобразование значений в массив
console.log(Object.values(user)); // ['Alice', 25, true]

// Преобразование объекта в URL-параметры
function objectToUrlParams(obj) {
  return new URLSearchParams(obj).toString();
}

const params = { name: 'Alice', age: 25, city: 'New York' };
console.log(objectToUrlParams(params)); // 'name=Alice&age=25&city=New+York'

// Преобразование объекта в формат для отправки в форме
function objectToFormData(obj) {
  const formData = new FormData();
  for (const [key, value] of Object.entries(obj)) {
    formData.append(key, value);
  }
  return formData;
}
```

## 11. Сравнение объектов

```javascript
// Поверхностное сравнение объектов
function shallowEqual(obj1, obj2) {
  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);
  
  if (keys1.length !== keys2.length) {
    return false;
  }
  
  for (let key of keys1) {
    if (obj1[key] !== obj2[key]) {
      return false;
    }
  }
  
  return true;
}

// Глубокое сравнение объектов (упрощенная версия)
function deepEqual(obj1, obj2) {
  if (obj1 === obj2) return true;
  
  if (obj1 == null || obj2 == null) return false;
  
  if (typeof obj1 !== 'object' || typeof obj2 !== 'object') {
    return obj1 === obj2;
  }
  
  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);
  
  if (keys1.length !== keys2.length) return false;
  
  for (let key of keys1) {
    if (!keys2.includes(key)) return false;
    if (!deepEqual(obj1[key], obj2[key])) return false;
  }
  
  return true;
}

const objA = { name: 'Alice', age: 25 };
const objB = { name: 'Alice', age: 25 };
const objC = { name: 'Alice', age: 26 };

console.log(shallowEqual(objA, objB)); // true
console.log(shallowEqual(objA, objC)); // false
console.log(deepEqual(objA, objB)); // true
```

## 12. Работа с Symbol-свойствами

```javascript
// Создание символа
const sym1 = Symbol('description');
const sym2 = Symbol('description'); // другой символ, несмотря на одинаковое описание

const obj = {
  [sym1]: 'symbol value',
  regular: 'regular value'
};

// Символы не перечисляются в for...in
for (const key in obj) {
  console.log(key); // 'regular' (только регулярные свойства)
}

// Получение собственных символьных ключей
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(description)]

// Получение всех собственных ключей (включая символы)
console.log(Object.getOwnPropertyNames(obj)); // ['regular']
console.log([...Object.getOwnPropertyNames(obj), ...Object.getOwnPropertySymbols(obj)]); // ['regular', Symbol(description)]

// Общий символ (используется для общего доступа)
const sharedSym = Symbol.for('shared');
const obj1 = { [sharedSym]: 'shared value' };
const obj2 = {};
obj2[Symbol.for('shared')] = 'another shared value';

console.log(obj1[sharedSym]); // 'shared value'
console.log(obj2[Symbol.for('shared')]); // 'another shared value'
console.log(Symbol.keyFor(sharedSym)); // 'shared'
```

Эти хитрости помогут вам более эффективно работать с объектами в JavaScript, улучшая производительность и читаемость кода. Они особенно полезны при разработке frontend приложений, где часто требуется манипуляция сложными структурами данных.

См. также:
- [[js/tricks/arrays-tricks]] - Работа с массивами
- [[js/tricks/strings-tricks]] - Работа со строками
- [[js/tricks/numbers-tricks]] - Числа и математика
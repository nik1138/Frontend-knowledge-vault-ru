---
aliases: [Иммутабельность в JavaScript, Иммутабельность JS]
tags: [javascript, immutability, programming]
---

# Иммутабельность в JavaScript

JavaScript по умолчанию предоставляет мутабельные структуры данных, но при этом содержит множество возможностей для работы с неизменяемыми данными. Понимание иммутабельности в JavaScript критически важно для создания надежных и предсказуемых приложений.

## Встроенные возможности иммутабельности

JavaScript предоставляет несколько встроенных способов работы с неизменяемыми данными:

### Object.freeze()

Метод `Object.freeze()` делает объект иммутабельным, предотвращая добавление, удаление или изменение свойств:

```javascript
const user = Object.freeze({
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  }
});

// Попытка изменения будет проигнорирована в non-strict mode
user.age = 31; // игнорируется
user.newProperty = 'value'; // игнорируется

console.log(user.age); // 30
console.log(user.newProperty); // undefined

// В strict mode будет выброшена ошибка
'use strict';
// user.age = 32; // TypeError в strict mode
```

> [!warning] Важно
> `Object.freeze()` работает только на поверхностном уровне. Вложенные объекты остаются изменяемыми.

### Object.seal()

Метод `Object.seal()` предотвращает добавление и удаление свойств, но позволяет изменять существующие:

```javascript
const config = Object.seal({
  apiUrl: 'https://api.example.com',
  timeout: 5000
});

config.timeout = 10000; // разрешено
config.newProp = 'value'; // игнорируется в non-strict mode

console.log(config.timeout); // 10000
console.log(config.newProp); // undefined
```

### Object.preventExtensions()

Метод `Object.preventExtensions()` предотвращает добавление новых свойств:

```javascript
const obj = {};
Object.preventExtensions(obj);

obj.newProp = 'value'; // игнорируется в non-strict mode
console.log(obj.newProp); // undefined
```

## Иммутабельные операции с массивами

JavaScript предоставляет множество методов массивов, которые возвращают новые массивы вместо изменения существующих:

### Методы, возвращающие новые массивы

```javascript
const originalArray = [1, 2, 3, 4, 5];

// map - создает новый массив с результатами вызова функции для каждого элемента
const doubled = originalArray.map(x => x * 2);
console.log(originalArray); // [1, 2, 3, 4, 5]
console.log(doubled);       // [2, 4, 6, 8, 10]

// filter - создает новый массив с элементами, прошедшими проверку
const evens = originalArray.filter(x => x % 2 === 0);
console.log(originalArray); // [1, 2, 3, 4, 5]
console.log(evens);         // [2, 4]

// slice - создает новый массив с частью элементов
const subset = originalArray.slice(1, 3);
console.log(originalArray); // [1, 2, 3, 4, 5]
console.log(subset);        // [2, 3]

// concat - объединяет массивы, создавая новый
const combined = originalArray.concat([6, 7]);
console.log(originalArray); // [1, 2, 3, 4, 5]
console.log(combined);      // [1, 2, 3, 4, 5, 6, 7]

// spread оператор для создания копий
const copy = [...originalArray];
console.log(copy);          // [1, 2, 3, 4, 5]
console.log(copy === originalArray); // false
```

### Избегание мутабельных методов массивов

Следующие методы изменяют исходный массив и должны избегаться при работе с иммутабельностью:

- `push()`, `pop()`, `shift()`, `unshift()`
- `splice()`, `sort()`, `reverse()`
- `fill()`, `copyWithin()`

Вместо них используйте иммутабельные альтернативы:

```javascript
const arr = [1, 2, 3];

// Вместо push
const newArr1 = [...arr, 4]; // [1, 2, 3, 4]

// Вместо pop
const newArr2 = arr.slice(0, -1); // [1, 2]

// Вместо splice для вставки
const newArr3 = [...arr.slice(0, 1), 99, ...arr.slice(1)]; // [1, 99, 2, 3]

// Вместо sort
const newArr4 = [...arr].sort(); // [1, 2, 3] (новый массив)

// Вместо reverse
const newArr5 = [...arr].reverse(); // [3, 2, 1] (новый массив)
```

## Иммутабельные операции с объектами

### Object.assign() и spread оператор

Для создания новых объектов без изменения исходных:

```javascript
const user = { name: 'John', age: 30 };
const address = { city: 'New York', country: 'USA' };

// Object.assign для создания нового объекта
const userWithAddress1 = Object.assign({}, user, address);
console.log(userWithAddress1); // { name: 'John', age: 30, city: 'New York', country: 'USA' }

// Spread оператор (предпочтительный способ)
const userWithAddress2 = { ...user, ...address };
console.log(userWithAddress2); // { name: 'John', age: 30, city: 'New York', country: 'USA' }

// Обновление свойств
const updatedUser = { ...user, age: 31 };
console.log(user.age);        // 30
console.log(updatedUser.age); // 31
```

### Глубокое копирование

Для работы с вложенными объектами может потребоваться глубокое копирование:

```javascript
const original = {
  user: {
    name: 'John',
    profile: {
      age: 30,
      hobbies: ['reading', 'coding']
    }
  }
};

// Простое копирование копирует только верхний уровень
const shallowCopy = { ...original };
shallowCopy.user.profile.age = 31;
console.log(original.user.profile.age); // 31 - непреднамеренное изменение!

// Глубокое копирование вручную
const deepCopy = {
  ...original,
  user: {
    ...original.user,
    profile: {
      ...original.user.profile,
      hobbies: [...original.user.profile.hobbies]
    }
  }
};

// Или с помощью JSON (ограничения: функции, undefined, Symbol, Date не поддерживаются)
const deepCopyJSON = JSON.parse(JSON.stringify(original));
```

## Функции высшего порядка и иммутабельность

Функции высшего порядка, такие как `map`, `filter`, `reduce`, естественным образом работают с концепцией иммутабельности:

```javascript
const users = [
  { id: 1, name: 'Alice', active: true },
  { id: 2, name: 'Bob', active: false },
  { id: 3, name: 'Charlie', active: true }
];

// Иммутабельные операции
const activeUsers = users.filter(user => user.active);
const userIds = users.map(user => user.id);
const userNames = users.map(user => user.name.toUpperCase());
const totalUsers = users.reduce((count, user) => count + 1, 0);

console.log(users);        // Исходный массив не изменен
console.log(activeUsers);  // [{ id: 1, name: 'Alice', active: true }, { id: 3, name: 'Charlie', active: true }]
console.log(userIds);      // [1, 2, 3]
console.log(userNames);    // ['ALICE', 'BOB', 'CHARLIE']
console.log(totalUsers);   // 3
```

## Практические примеры иммутабельности

### Работа с состоянием в React

```javascript
import React, { useState } from 'react';

function UserProfile() {
  const [user, setUser] = useState({
    name: 'John',
    preferences: {
      theme: 'light',
      notifications: true
    }
  });
  
  const updateUserName = (newName) => {
    // Иммутабельное обновление
    setUser(prevUser => ({
      ...prevUser,
      name: newName
    }));
  };
  
  const updateTheme = (newTheme) => {
    // Глубокое иммутабельное обновление
    setUser(prevUser => ({
      ...prevUser,
      preferences: {
        ...prevUser.preferences,
        theme: newTheme
      }
    }));
  };
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Тема: {user.preferences.theme}</p>
      <button onClick={() => updateUserName('Jane')}>
        Изменить имя
      </button>
      <button onClick={() => updateTheme('dark')}>
        Сменить тему
      </button>
    </div>
  );
}
```

### Работа с массивами в Redux

```javascript
// Redux reducer с иммутабельными операциями
const todosReducer = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          id: action.id,
          text: action.text,
          completed: false
        }
      ];
    
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    
    case 'DELETE_TODO':
      return state.filter(todo => todo.id !== action.id);
    
    default:
      return state;
  }
};
```

## Заключение

Иммутабельность в JavaScript требует сознательного подхода и использования правильных паттернов. Хотя JavaScript не обеспечивает иммутабельность по умолчанию, его богатые возможности позволяют реализовать иммутабельные паттерны, которые улучшают надежность и предсказуемость приложений.

См. также: [[Неизменяемые-структуры-данных]], [[Преимущества-иммутабельности]], [[Библиотеки-иммутабельности]]
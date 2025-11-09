# Практические упражнения: Новые API ES6+

## Упражнение 1: Promise

### Задача
Создайте функцию fetchUserData, которая возвращает Promise и имитирует получение данных пользователя с сервера. Добавьте обработку ошибок и цепочку промисов для обработки данных:

### Требования
1. Функция должна возвращать Promise
2. Имитировать задержку выполнения (2 секунды)
3. Случайным образом разрешать или отклонять Promise
4. Обработать результаты с помощью then/catch
5. Создать цепочку промисов для обработки данных

### Решение
```javascript
function fetchUserData(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // Случайная имитация успеха или ошибки
      if (Math.random() > 0.3) {
        const userData = {
          id: userId,
          name: `User ${userId}`,
          email: `user${userId}@example.com`,
          age: Math.floor(Math.random() * 50) + 18
        };
        resolve(userData);
      } else {
        reject(new Error(`Ошибка загрузки данных для пользователя ${userId}`));
      }
    }, 2000);
  });
}

// Использование с then/catch
fetchUserData(123)
  .then(user => {
    console.log("Получены данные пользователя:", user);
    return user;
  })
  .then(user => {
    console.log(`Имя: ${user.name}, Возраст: ${user.age}`);
  })
  .catch(error => {
    console.error("Ошибка:", error.message);
  });

// Цепочка промисов для обработки нескольких пользователей
Promise.all([
  fetchUserData(1),
  fetchUserData(2),
  fetchUserData(3)
])
  .then(users => {
    console.log("Все пользователи загружены:", users);
    const totalAge = users.reduce((sum, user) => sum + user.age, 0);
    console.log(`Средний возраст: ${totalAge / users.length}`);
  })
  .catch(error => {
    console.error("Ошибка при загрузке пользователей:", error.message);
  });
```

## Упражнение 2: Map и Set

### Задача
Создайте систему управления библиотекой с использованием Map и Set для хранения информации о книгах и читателях:

### Требования
1. Использовать Map для хранения информации о книгах (ключ - ISBN, значение - объект книги)
2. Использовать Set для отслеживания уникальных читателей
3. Реализовать методы добавления/удаления книг и читателей
4. Реализовать методы поиска и фильтрации

### Решение
```javascript
class Library {
  constructor() {
    this.books = new Map(); // ISBN -> book object
    this.readers = new Set(); // уникальные читатели
    this.borrowedBooks = new Map(); // ISBN -> readerId
  }
  
  addBook(isbn, title, author, year) {
    const book = { isbn, title, author, year, available: true };
    this.books.set(isbn, book);
    return book;
  }
  
  removeBook(isbn) {
    if (this.books.has(isbn)) {
      this.books.delete(isbn);
      // Если книга была выдана, освобождаем её
      if (this.borrowedBooks.has(isbn)) {
        this.borrowedBooks.delete(isbn);
      }
      return true;
    }
    return false;
  }
  
  addReader(readerId, name) {
    this.readers.add({ id: readerId, name });
  }
  
  borrowBook(isbn, readerId) {
    const book = this.books.get(isbn);
    if (book && book.available) {
      book.available = false;
      this.borrowedBooks.set(isbn, readerId);
      return true;
    }
    return false;
  }
  
  returnBook(isbn) {
    const book = this.books.get(isbn);
    if (book && !book.available) {
      book.available = true;
      this.borrowedBooks.delete(isbn);
      return true;
    }
    return false;
  }
  
  getAvailableBooks() {
    return Array.from(this.books.values()).filter(book => book.available);
  }
  
  getBorrowedBooks() {
    const result = [];
    for (const [isbn, readerId] of this.borrowedBooks) {
      const book = this.books.get(isbn);
      result.push({ book, readerId });
    }
    return result;
  }
  
  findBooksByAuthor(author) {
    return Array.from(this.books.values()).filter(
      book => book.author.toLowerCase().includes(author.toLowerCase())
    );
  }
}

// Пример использования
const library = new Library();

// Добавляем книги
library.addBook("978-0134685991", "Effective Java", "Joshua Bloch", 2017);
library.addBook("978-0596517748", "JavaScript: The Good Parts", "Douglas Crockford", 2008);
library.addBook("978-1449331818", "Learning JavaScript Design Patterns", "Addy Osmani", 2012);

// Добавляем читателей
library.addReader(1, "Alice Johnson");
library.addReader(2, "Bob Smith");

// Выдаем книги
library.borrowBook("978-0134685991", 1);
library.borrowBook("978-0596517748", 2);

// Проверяем доступные книги
console.log("Доступные книги:", library.getAvailableBooks());

// Проверяем выданные книги
console.log("Выданные книги:", library.getBorrowedBooks());

// Поиск по автору
console.log("Книги Дугласа Крокфорда:", library.findBooksByAuthor("Crockford"));
```

## Упражнение 3: WeakMap и WeakSet

### Задача
Создайте систему приватных свойств для класса с использованием WeakMap и систему уникальных объектов с WeakSet:

### Требования
1. Использовать WeakMap для реализации приватных свойств класса
2. Использовать WeakSet для отслеживания уникальных объектов
3. Реализовать методы доступа к приватным свойствам
4. Продемонстрировать автоматическую очистку памяти

### Решение
```javascript
// WeakMap для приватных свойств
const privateProps = new WeakMap();

class User {
  constructor(name, email) {
    // Приватные свойства
    privateProps.set(this, {
      name: name,
      email: email,
      password: this.generatePassword(),
      loginAttempts: 0
    });
  }
  
  // Публичные методы для доступа к приватным свойствам
  getName() {
    return privateProps.get(this).name;
  }
  
  getEmail() {
    return privateProps.get(this).email;
  }
  
  setPassword(newPassword) {
    const props = privateProps.get(this);
    props.password = newPassword;
    privateProps.set(this, props);
  }
  
  validatePassword(password) {
    const props = privateProps.get(this);
    props.loginAttempts++;
    privateProps.set(this, props);
    return password === props.password;
  }
  
  getLoginAttempts() {
    return privateProps.get(this).loginAttempts;
  }
  
  generatePassword() {
    return Math.random().toString(36).slice(-8);
  }
}

// WeakSet для отслеживания активных сессий
const activeSessions = new WeakSet();

class SessionManager {
  constructor() {
    this.sessions = [];
  }
  
  createSession(user) {
    const session = {
      user: user,
      id: Math.random().toString(36).substr(2, 9),
      createdAt: new Date(),
      lastActivity: new Date()
    };
    
    this.sessions.push(session);
    activeSessions.add(session); // Добавляем в WeakSet
    
    return session;
  }
  
  isActive(session) {
    return activeSessions.has(session);
  }
  
  // Имитация очистки неактивных сессий
  cleanupSessions() {
    // В реальной реализации сборщик мусора автоматически очистит
    // сессии, на которые больше нет ссылок
    this.sessions = this.sessions.filter(session => 
      activeSessions.has(session)
    );
  }
}

// Пример использования
const user = new User("Alice", "alice@example.com");
console.log(user.getName()); // Alice
console.log(user.getEmail()); // alice@example.com

// Попытка доступа к приватным свойствам напрямую
console.log(user.name); // undefined (приватное свойство)
console.log(user.email); // undefined (приватное свойство)

// Проверка пароля
console.log(user.validatePassword("wrong")); // false
console.log(user.getLoginAttempts()); // 1

// Работа с сессиями
const sessionManager = new SessionManager();
const session = sessionManager.createSession(user);

console.log(sessionManager.isActive(session)); // true

// После удаления всех ссылок на сессию,
// WeakSet автоматически очистит её
// (в реальном коде это произойдёт при сборке мусора)
```

## Упражнение 4: Proxy и Reflect

### Задача
Создайте систему валидации объектов с использованием Proxy и Reflect:

### Требования
1. Использовать Proxy для перехвата операций над объектом
2. Реализовать валидацию при установке свойств
3. Использовать Reflect для выполнения оригинальных операций
4. Добавить логирование операций

### Решение
```javascript
// Валидаторы для разных типов данных
const validators = {
  string: (value) => typeof value === 'string' && value.length > 0,
  number: (value) => typeof value === 'number' && !isNaN(value),
  email: (value) => typeof value === 'string' && /\S+@\S+\.\S+/.test(value),
  age: (value) => typeof value === 'number' && value >= 0 && value <= 150
};

// Фабрика для создания валидируемых объектов
function createValidatedObject(schema) {
  const target = {};
  
  return new Proxy(target, {
    set(obj, prop, value) {
      // Проверяем, есть ли валидатор для этого свойства
      if (schema[prop]) {
        const { type, required = false, validator } = schema[prop];
        
        // Проверка обязательного поля
        if (required && (value === undefined || value === null)) {
          throw new Error(`Свойство ${prop} является обязательным`);
        }
        
        // Если значение не задано и не обязательно, пропускаем валидацию
        if (value === undefined || value === null) {
          return Reflect.set(obj, prop, value);
        }
        
        // Проверка типа
        if (type && !validators[type](value)) {
          throw new Error(`Неверный тип для свойства ${prop}. Ожидается ${type}`);
        }
        
        // Пользовательская валидация
        if (validator && !validator(value)) {
          throw new Error(`Валидация не пройдена для свойства ${prop}`);
        }
      }
      
      // Логирование
      console.log(`Установка свойства ${prop} = ${value}`);
      
      // Используем Reflect для выполнения оригинальной операции
      return Reflect.set(obj, prop, value);
    },
    
    get(obj, prop) {
      // Логирование доступа к свойству
      if (prop in obj) {
        console.log(`Чтение свойства ${prop} = ${obj[prop]}`);
      }
      
      return Reflect.get(obj, prop);
    },
    
    has(obj, prop) {
      console.log(`Проверка наличия свойства ${prop}`);
      return Reflect.has(obj, prop);
    },
    
    deleteProperty(obj, prop) {
      console.log(`Удаление свойства ${prop}`);
      return Reflect.deleteProperty(obj, prop);
    }
  });
}

// Пример использования
const userSchema = {
  name: { type: 'string', required: true },
  email: { type: 'email', required: true },
  age: { type: 'age', required: false },
  isActive: { 
    type: 'number', 
    validator: (value) => value === 0 || value === 1 
  }
};

const user = createValidatedObject(userSchema);

// Установка корректных значений
user.name = "Alice";
user.email = "alice@example.com";
user.age = 25;
user.isActive = 1;

console.log(user.name); // Alice
console.log(user.email); // alice@example.com

// Попытка установки некорректных значений
try {
  user.email = "invalid-email"; // Ошибка валидации
} catch (error) {
  console.error(error.message);
}

try {
  user.age = -5; // Ошибка валидации возраста
} catch (error) {
  console.error(error.message);
}

try {
  delete user.name; // Удаление свойства
  console.log('name' in user); // false
} catch (error) {
  console.error(error.message);
}
```

## Упражнение 5: Generators

### Задача
Создайте генератор для последовательности чисел Фибоначчи и генератор для обработки асинхронных операций:

### Требования
1. Создать генератор чисел Фибоначчи
2. Создать генератор для обработки асинхронных операций
3. Реализовать функцию для выполнения асинхронного генератора
4. Продемонстрировать использование генераторов

### Решение
```javascript
// Генератор чисел Фибоначчи
function* fibonacciGenerator() {
  let prev = 0;
  let curr = 1;
  
  yield prev;
  yield curr;
  
  while (true) {
    const next = prev + curr;
    yield next;
    prev = curr;
    curr = next;
  }
}

// Использование генератора Фибоначчи
const fib = fibonacciGenerator();
console.log("Первые 10 чисел Фибоначчи:");
for (let i = 0; i < 10; i++) {
  console.log(fib.next().value);
}

// Асинхронный генератор для обработки данных
async function* asyncDataProcessor(urls) {
  for (const url of urls) {
    try {
      console.log(`Загрузка данных с ${url}...`);
      // Имитация асинхронной операции
      const data = await new Promise(resolve => 
        setTimeout(() => resolve(`Данные с ${url}`), Math.random() * 1000)
      );
      yield { url, data, status: 'success' };
    } catch (error) {
      yield { url, error: error.message, status: 'error' };
    }
  }
}

// Функция для выполнения асинхронного генератора
async function processAsyncGenerator(generator) {
  const results = [];
  for await (const result of generator) {
    results.push(result);
    console.log(result);
  }
  return results;
}

// Пример использования асинхронного генератора
const urls = [
  'https://api.example1.com/data',
  'https://api.example2.com/data',
  'https://api.example3.com/data'
];

// processAsyncGenerator(asyncDataProcessor(urls))
//   .then(results => console.log('Все данные обработаны:', results));

// Генератор для пагинации данных
function* paginateData(data, pageSize = 5) {
  for (let i = 0; i < data.length; i += pageSize) {
    yield data.slice(i, i + pageSize);
  }
}

// Пример использования пагинации
const largeDataset = Array.from({ length: 23 }, (_, i) => `Item ${i + 1}`);
const paginator = paginateData(largeDataset, 5);

console.log("Страницы данных:");
let page = 1;
for (const dataPage of paginator) {
  console.log(`Страница ${page}:`, dataPage);
  page++;
}

// Генератор с передачей значений через next()
function* interactiveCounter() {
  let count = 0;
  while (true) {
    const increment = yield count;
    if (increment !== undefined) {
      count += increment;
    } else {
      count++;
    }
  }
}

// Использование генератора с передачей значений
const counter = interactiveCounter();
console.log(counter.next().value); // 0
console.log(counter.next().value); // 1
console.log(counter.next(5).value); // 6 (увеличиваем на 5)
console.log(counter.next(10).value); // 16 (увеличиваем на 10)
```

## Упражнение 6: Async/Await

### Задача
Создайте асинхронное приложение для обработки пользовательских данных с использованием async/await:

### Требования
1. Создать асинхронные функции для имитации API вызовов
2. Использовать async/await для последовательной и параллельной обработки
3. Реализовать обработку ошибок с try/catch
4. Использовать Promise.all и Promise.race

### Решение
```javascript
// Имитация асинхронных API вызовов
function fetchUser(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (userId > 0) {
        resolve({
          id: userId,
          name: `User ${userId}`,
          email: `user${userId}@example.com`
        });
      } else {
        reject(new Error('Неверный ID пользователя'));
      }
    }, Math.random() * 1000 + 500);
  });
}

function fetchUserPosts(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (userId > 0) {
        resolve([
          { id: 1, title: `Post 1 by User ${userId}`, content: "Content 1" },
          { id: 2, title: `Post 2 by User ${userId}`, content: "Content 2" }
        ]);
      } else {
        reject(new Error('Неверный ID пользователя для постов'));
      }
    }, Math.random() * 1000 + 300);
  });
}

function fetchUserComments(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (userId > 0) {
        resolve([
          { id: 1, postId: 1, content: `Comment by User ${userId}` }
        ]);
      } else {
        reject(new Error('Неверный ID пользователя для комментариев'));
      }
    }, Math.random() * 1000 + 200);
  });
}

// Асинхронная функция для получения полной информации о пользователе
async function getUserProfile(userId) {
  try {
    console.log(`Загрузка профиля пользователя ${userId}...`);
    
    // Последовательная загрузка данных
    const user = await fetchUser(userId);
    const posts = await fetchUserPosts(userId);
    const comments = await fetchUserComments(userId);
    
    return {
      user,
      posts,
      comments,
      totalItems: posts.length + comments.length
    };
  } catch (error) {
    console.error(`Ошибка при загрузке профиля пользователя ${userId}:`, error.message);
    throw error;
  }
}

// Асинхронная функция для параллельной загрузки данных
async function getUserProfileParallel(userId) {
  try {
    console.log(`Параллельная загрузка профиля пользователя ${userId}...`);
    
    // Параллельная загрузка данных
    const [user, posts, comments] = await Promise.all([
      fetchUser(userId),
      fetchUserPosts(userId),
      fetchUserComments(userId)
    ]);
    
    return {
      user,
      posts,
      comments,
      totalItems: posts.length + comments.length
    };
  } catch (error) {
    console.error(`Ошибка при параллельной загрузке профиля пользователя ${userId}:`, error.message);
    throw error;
  }
}

// Асинхательная функция с таймаутом
async function getUserProfileWithTimeout(userId, timeout = 2000) {
  try {
    console.log(`Загрузка профиля пользователя ${userId} с таймаутом ${timeout}мс...`);
    
    // Создаем промис с таймаутом
    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => reject(new Error('Таймаут загрузки данных')), timeout);
    });
    
    // Используем Promise.race для реализации таймаута
    const profile = await Promise.race([
      getUserProfileParallel(userId),
      timeoutPromise
    ]);
    
    return profile;
  } catch (error) {
    console.error(`Ошибка при загрузке профиля пользователя ${userId}:`, error.message);
    throw error;
  }
}

// Асинхронная функция для обработки нескольких пользователей
async function processMultipleUsers(userIds) {
  console.log('Обработка нескольких пользователей...');
  
  // Обрабатываем пользователей параллельно
  const profilePromises = userIds.map(id => 
    getUserProfileParallel(id).catch(error => ({
      error: error.message,
      userId: id
    }))
  );
  
  const profiles = await Promise.all(profilePromises);
  
  // Фильтруем успешные результаты
  const successfulProfiles = profiles.filter(profile => !profile.error);
  const failedProfiles = profiles.filter(profile => profile.error);
  
  console.log(`Успешно обработано: ${successfulProfiles.length}`);
  console.log(`Ошибок: ${failedProfiles.length}`);
  
  return { successfulProfiles, failedProfiles };
}

// Пример использования
async function main() {
  try {
    // Последовательная загрузка
    console.log('=== Последовательная загрузка ===');
    const profile1 = await getUserProfile(1);
    console.log('Профиль пользователя 1:', profile1);
    
    // Параллельная загрузка
    console.log('\n=== Параллельная загрузка ===');
    const profile2 = await getUserProfileParallel(2);
    console.log('Профиль пользователя 2:', profile2);
    
    // Загрузка с таймаутом
    console.log('\n=== Загрузка с таймаутом ===');
    try {
      const profile3 = await getUserProfileWithTimeout(3, 3000);
      console.log('Профиль пользователя 3:', profile3);
    } catch (error) {
      console.error('Таймаут при загрузке профиля пользователя 3');
    }
    
    // Обработка нескольких пользователей
    console.log('\n=== Обработка нескольких пользователей ===');
    const userIds = [1, 2, 3, -1, 4]; // -1 вызовет ошибку
    const results = await processMultipleUsers(userIds);
    console.log('Результаты обработки:', results);
    
  } catch (error) {
    console.error('Общая ошибка приложения:', error.message);
  }
}

// Запуск примера
main();
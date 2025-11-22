# Объекты в JavaScript: Основы

## Что такое объекты

Объекты в JavaScript - это коллекции свойств, где каждое свойство состоит из пары ключ-значение. Они являются фундаментальной частью языка и используются для хранения сложных структур данных.

## Создание объектов

### Литерал объекта

```javascript
// Пустой объект
let emptyObj = {};

// Объект с данными
let person = {
  name: "Иван",
  age: 30,
  isStudent: false
};

console.log(person); // { name: "Иван", age: 30, isStudent: false }
```

### Конструктор Object

```javascript
// Создание объекта с помощью конструктора
let car = new Object();
car.brand = "Toyota";
car.model = "Camry";
car.year = 2020;

console.log(car); // { brand: "Toyota", model: "Camry", year: 2020 }
```

### Функция-конструктор

```javascript
// Функция-конструктор
function User(name, email) {
  this.name = name;
  this.email = email;
  this.isActive = true;
}

let user1 = new User("Иван", "ivan@example.com");
let user2 = new User("Мария", "maria@example.com");

console.log(user1); // User { name: "Иван", email: "ivan@example.com", isActive: true }
```

### Object.create()

```javascript
// Создание объекта с заданным прототипом
let animal = {
  type: "животное",
  speak: function() {
    console.log(`${this.name} издает звук`);
  }
};

let dog = Object.create(animal);
dog.name = "Бобик";
dog.breed = "Овчарка";

dog.speak(); // "Бобик издает звук"
```

## Свойства объектов

### Доступ к свойствам

```javascript
let book = {
  title: "Изучаем JavaScript",
  author: "Иван Иванов",
  pages: 350,
  isAvailable: true
};

// Доступ через точечную нотацию
console.log(book.title); // "Изучаем JavaScript"
console.log(book.author); // "Иван Иванов"

// Доступ через квадратные скобки
console.log(book["title"]); // "Изучаем JavaScript"
console.log(book["pages"]); // 350

// Доступ к свойству с переменной
let propertyName = "author";
console.log(book[propertyName]); // "Иван Иванов"
```

### Добавление и изменение свойств

```javascript
let product = {
  name: "Ноутбук",
  price: 50000
};

// Добавление новых свойств
product.category = "Электроника";
product["inStock"] = true;

// Изменение существующих свойств
product.price = 45000;
product["name"] = "Ноутбук Dell";

console.log(product);
// { name: "Ноутбук Dell", price: 45000, category: "Электроника", inStock: true }
```

### Удаление свойств

```javascript
let user = {
  name: "Иван",
  age: 30,
  email: "ivan@example.com",
  password: "secret123"
};

// Удаление свойства
delete user.password;
console.log(user); // { name: "Иван", age: 30, email: "ivan@example.com" }

// Удаление свойства через квадратные скобки
delete user["email"];
console.log(user); // { name: "Иван", age: 30 }
```

## Методы объектов

### Определение методов

```javascript
let calculator = {
  // Метод объекта
  add: function(a, b) {
    return a + b;
  },
  
  // Сокращенный синтаксис метода (ES6)
  subtract(a, b) {
    return a - b;
  },
  
  // Стрелочная функция как метод
  multiply: (a, b) => a * b,
  
  // Метод с использованием this
  power(base, exponent) {
    return base ** exponent;
  },
  
  // Метод, использующий другие методы объекта
  calculate(operation, a, b) {
    switch (operation) {
      case "add":
        return this.add(a, b);
      case "subtract":
        return this.subtract(a, b);
      case "multiply":
        return this.multiply(a, b);
      default:
        return "Неизвестная операция";
    }
  }
};

console.log(calculator.add(5, 3)); // 8
console.log(calculator.subtract(10, 4)); // 6
console.log(calculator.calculate("add", 7, 2)); // 9
```

### Ключевое слово this

```javascript
let person = {
  name: "Иван",
  age: 30,
  
  // Метод, использующий this
  greet: function() {
    return `Привет, меня зовут ${this.name}`;
  },
  
  // Метод для обновления возраста
  haveBirthday: function() {
    this.age++;
    return `С днем рождения! Теперь мне ${this.age} лет`;
  },
  
  // Метод, возвращающий информацию о себе
  getInfo: function() {
    return `${this.name}, ${this.age} лет`;
  }
};

console.log(person.greet()); // "Привет, меня зовут Иван"
console.log(person.haveBirthday()); // "С днем рождения! Теперь мне 31 лет"
console.log(person.getInfo()); // "Иван, 31 лет"
```

## Перебор свойств объекта

### for...in цикл

```javascript
let user = {
  name: "Иван",
  age: 30,
  email: "ivan@example.com",
  isActive: true
};

// Перебор всех перечисляемых свойств
for (let key in user) {
  console.log(`${key}: ${user[key]}`);
}

// Вывод:
// name: Иван
// age: 30
// email: ivan@example.com
// isActive: true
```

### Object.keys(), Object.values(), Object.entries()

```javascript
let product = {
  name: "Смартфон",
  price: 30000,
  category: "Электроника"
};

// Получение массива ключей
let keys = Object.keys(product);
console.log(keys); // ["name", "price", "category"]

// Получение массива значений
let values = Object.values(product);
console.log(values); // ["Смартфон", 30000, "Электроника"]

// Получение массива пар [ключ, значение]
let entries = Object.entries(product);
console.log(entries); 
// [["name", "Смартфон"], ["price", 30000], ["category", "Электроника"]]

// Использование для перебора
Object.keys(product).forEach(key => {
  console.log(`${key}: ${product[key]}`);
});
```

## Проверка свойств

### Оператор in

```javascript
let car = {
  brand: "Toyota",
  model: "Camry",
  year: 2020
};

console.log("brand" in car); // true
console.log("color" in car); // false
console.log("toString" in car); // true (унаследованное свойство)
```

### hasOwnProperty()

```javascript
let person = {
  name: "Иван",
  age: 30
};

console.log(person.hasOwnProperty("name")); // true
console.log(person.hasOwnProperty("toString")); // false (унаследованное свойство)
console.log(person.hasOwnProperty("nonExistent")); // false
```

### Проверка на undefined

```javascript
let user = {
  name: "Иван",
  email: null
};

// Проверка на существование свойства
if (user.name !== undefined) {
  console.log(`Имя: ${user.name}`);
}

// Внимание: свойство может существовать, но иметь значение undefined
user.age = undefined;
console.log("age" in user); // true
console.log(user.age !== undefined); // false
```

## Копирование объектов

### Поверхностное копирование

```javascript
let original = {
  name: "Иван",
  age: 30,
  address: {
    city: "Москва",
    street: "Ленина"
  }
};

// Поверхностное копирование с помощью Object.assign()
let copy1 = Object.assign({}, original);

// Поверхностное копирование с помощью spread оператора (ES6)
let copy2 = { ...original };

// Поверхностное копирование с помощью Object.create()
let copy3 = Object.create(Object.getPrototypeOf(original));
Object.assign(copy3, original);

// Изменение копии влияет на вложенные объекты
copy1.name = "Мария"; // Не влияет на original
copy1.address.city = "Санкт-Петербург"; // Влияет на original!

console.log(original.address.city); // "Санкт-Петербург"
```

### Глубокое копирование

```javascript
let original = {
  name: "Иван",
  age: 30,
  address: {
    city: "Москва",
    street: "Ленина"
  },
  hobbies: ["чтение", "программирование"]
};

// Глубокое копирование с помощью JSON (ограничения: не работает с функциями, undefined, Symbol)
let deepCopy = JSON.parse(JSON.stringify(original));

deepCopy.address.city = "Санкт-Петербург";
deepCopy.hobbies.push("спорт");

console.log(original.address.city); // "Москва" (не изменился)
console.log(original.hobbies); // ["чтение", "программирование"] (не изменился)

// Альтернативный способ глубокого копирования (для простых случаев)
function deepClone(obj) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(item => deepClone(item));
  if (obj instanceof Object) {
    let clonedObj = {};
    for (let key in obj) {
      if (obj.hasOwnProperty(key)) {
        clonedObj[key] = deepClone(obj[key]);
      }
    }
    return clonedObj;
  }
}

let deepCopy2 = deepClone(original);
```

## Практические примеры

### Создание конфигурационного объекта

```javascript
// Фабрика конфигурационных объектов
function createAppConfig(options = {}) {
  return {
    // Значения по умолчанию
    name: options.name || "Мое приложение",
    version: options.version || "1.0.0",
    debug: options.debug || false,
    apiUrl: options.apiUrl || "https://api.example.com",
    
    // Методы
    getInfo: function() {
      return `${this.name} v${this.version}`;
    },
    
    toggleDebug: function() {
      this.debug = !this.debug;
      return this.debug ? "Режим отладки включен" : "Режим отладки выключен";
    },
    
    updateApiUrl: function(newUrl) {
      if (newUrl && typeof newUrl === "string") {
        this.apiUrl = newUrl;
        return true;
      }
      return false;
    }
  };
}

const config = createAppConfig({
  name: "Мой проект",
  version: "2.1.0",
  debug: true
});

console.log(config.getInfo()); // "Мой проект v2.1.0"
console.log(config.toggleDebug()); // "Режим отладки выключен"
```

### Работа с коллекциями объектов

```javascript
// Управление коллекцией пользователей
const userManagement = {
  users: [],
  
  // Добавление пользователя
  addUser: function(userData) {
    const user = {
      id: this.generateId(),
      ...userData,
      createdAt: new Date()
    };
    this.users.push(user);
    return user;
  },
  
  // Генерация уникального ID
  generateId: function() {
    return Math.random().toString(36).substr(2, 9);
  },
  
  // Поиск пользователя по ID
  findUserById: function(id) {
    return this.users.find(user => user.id === id);
  },
  
  // Поиск пользователей по свойству
  findUsersBy: function(property, value) {
    return this.users.filter(user => user[property] === value);
  },
  
  // Обновление пользователя
  updateUser: function(id, updates) {
    const user = this.findUserById(id);
    if (user) {
      Object.assign(user, updates);
      return user;
    }
    return null;
  },
  
  // Удаление пользователя
  removeUser: function(id) {
    const index = this.users.findIndex(user => user.id === id);
    if (index !== -1) {
      return this.users.splice(index, 1)[0];
    }
    return null;
  },
  
  // Получение всех пользователей
  getAllUsers: function() {
    return [...this.users]; // Возвращаем копию массива
  }
};

// Использование
userManagement.addUser({ name: "Иван", email: "ivan@example.com", role: "user" });
userManagement.addUser({ name: "Мария", email: "maria@example.com", role: "admin" });

console.log(userManagement.getAllUsers());
console.log(userManagement.findUsersBy("role", "admin"));
```

### Объект для работы с корзиной покупок

```javascript
// Корзина покупок
function createShoppingCart() {
  return {
    items: [],
    
    // Добавление товара
    addItem: function(product, quantity = 1) {
      const existingItem = this.items.find(item => item.id === product.id);
      
      if (existingItem) {
        existingItem.quantity += quantity;
      } else {
        this.items.push({
          ...product,
          quantity: quantity
        });
      }
      
      return this;
    },
    
    // Удаление товара
    removeItem: function(productId) {
      this.items = this.items.filter(item => item.id !== productId);
      return this;
    },
    
    // Обновление количества
    updateQuantity: function(productId, quantity) {
      const item = this.items.find(item => item.id === productId);
      if (item) {
        if (quantity <= 0) {
          this.removeItem(productId);
        } else {
          item.quantity = quantity;
        }
      }
      return this;
    },
    
    // Получение общего количества товаров
    getTotalItems: function() {
      return this.items.reduce((total, item) => total + item.quantity, 0);
    },
    
    // Получение общей стоимости
    getTotalPrice: function() {
      return this.items.reduce((total, item) => total + (item.price * item.quantity), 0);
    },
    
    // Очистка корзины
    clear: function() {
      this.items = [];
      return this;
    },
    
    // Получение содержимого корзины
    getItems: function() {
      return [...this.items];
    }
  };
}

// Использование
const cart = createShoppingCart();

const products = [
  { id: 1, name: "Ноутбук", price: 50000 },
  { id: 2, name: "Мышь", price: 1000 },
  { id: 3, name: "Клавиатура", price: 2000 }
];

cart.addItem(products[0], 1)
    .addItem(products[1], 2)
    .addItem(products[2], 1);

console.log("Товаров в корзине:", cart.getTotalItems()); // 4
console.log("Общая стоимость:", cart.getTotalPrice()); // 54000
```

## Рекомендации по использованию

1. Используйте литерал объекта `{}` для создания простых объектов
2. Используйте деструктуризацию для извлечения свойств объекта
3. Используйте spread оператор для поверхностного копирования объектов
4. Используйте `Object.keys()`, `Object.values()`, `Object.entries()` для перебора свойств
5. Будьте осторожны с поверхностным копированием вложенных объектов
6. Используйте методы объекта для инкапсуляции логики работы с данными

## Заключение

Объекты являются одной из ключевых концепций JavaScript, позволяя создавать сложные структуры данных и организовывать код. Понимание работы с объектами, их свойствами и методами критически важно для эффективного программирования на JavaScript.

#javascript #programming #objects #fundamentals #web-development
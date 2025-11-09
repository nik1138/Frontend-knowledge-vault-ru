# Упражнения по типам данных

## Упражнение 3: Работа с массивами
Практикуйте операции с массивами:

```javascript
// Задание: Создайте массив и выполните различные операции
// 1. Создайте массив с элементами разных типов
// 2. Добавьте элементы в начало и конец массива
// 3. Удалите элементы из массива
// 4. Найдите элементы в массиве

let mixedArray = [1, "строка", true, null, {name: "Иван"}];

// Добавление элементов
mixedArray.push("новый элемент в конце");
mixedArray.unshift("новый элемент в начале");

// Удаление элементов
let lastElement = mixedArray.pop(); // Удаляет последний элемент
let firstElement = mixedArray.shift(); // Удаляет первый элемент

// Поиск элементов
let index = mixedArray.indexOf("строка");
let found = mixedArray.find(element => typeof element === "object");

console.log(mixedArray);
console.log("Индекс 'строка':", index);
console.log("Найденный объект:", found);
```

## Упражнение 4: Работа с объектами
Практикуйте операции с объектами:

```javascript
// Задание: Создайте объект пользователя и выполните операции
// 1. Создайте объект с данными пользователя
// 2. Добавьте и удалите свойства
// 3. Переберите свойства объекта
// 4. Проверьте наличие свойств

let user = {
  name: "Иван",
  age: 30,
  email: "ivan@example.com",
  isActive: true
};

// Добавление свойств
user.city = "Москва";
user["phone"] = "+7 (123) 456-78-90";

// Удаление свойств
delete user.isActive;

// Перебор свойств
for (let key in user) {
  console.log(`${key}: ${user[key]}`);
}

// Проверка наличия свойств
console.log("name" in user); // true
console.log("isActive" in user); // false
console.log(user.hasOwnProperty("age")); // true

// Деструктуризация объекта
const {name, age, ...otherProps} = user;
console.log(name, age, otherProps);
---
aliases: [Принципы SOLID, SOLID принципы, Объектно-ориентированное проектирование]
tags: [programming, principles, oop, architecture]
---

# SOLID - Основные принципы объектно-ориентированного проектирования

SOLID — это мнемоника, обозначающая пять основных принципов объектно-ориентированного проектирования, предложенных Робертом Мартином. Эти принципы помогают создавать более гибкие, поддерживаемые и тестируемые приложения.

## Расшифровка SOLID

- **S** — Принцип единственной ответственности (Single Responsibility Principle)
- **O** — Принцип открытости/закрытости (Open/Closed Principle)
- **L** — Принцип подстановки Барбары Лисков (Liskov Substitution Principle)
- **I** — Принцип разделения интерфейсов (Interface Segregation Principle)
- **D** — Принцип инверсии зависимостей (Dependency Inversion Principle)

## 1. Принцип единственной ответственности (SRP)

> У класса должна быть только одна причина для изменения.

Каждый класс должен выполнять только одну задачу. Это упрощает тестирование, отладку и поддержку кода.

### Плохой пример:

```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  saveToDatabase() {
    // Сохранение пользователя в базу данных
  }

  sendEmail() {
    // Отправка электронного письма
  }

  generateReport() {
    // Генерация отчета о пользователе
  }
}
```

### Хороший пример:

```javascript
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

class UserRepository {
  save(user) {
    // Сохранение пользователя в базу данных
  }

  findById(id) {
    // Получение пользователя по ID
  }
}

class EmailService {
  sendEmail(user, message) {
    // Отправка электронного письма пользователю
  }
}

class ReportGenerator {
  generateUserReport(user) {
    // Генерация отчета о пользователе
  }
}
```

## 2. Принцип открытости/закрытости (OCP)

> Программные сущности (классы, модули, функции и т.д.) должны быть открыты для расширения, но закрыты для модификации.

Мы должны иметь возможность расширять поведение системы без изменения существующего кода.

### Пример:

```javascript
// Базовый класс для формы
class Shape {
  area() {
    throw new Error('Метод area() должен быть реализован');
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  area() {
    return Math.PI * this.radius * this.radius;
  }
}

// Калькулятор площади, который не требует изменения при добавлении новых форм
class AreaCalculator {
  static calculateTotalArea(shapes) {
    return shapes.reduce((total, shape) => total + shape.area(), 0);
  }
}

const shapes = [
  new Rectangle(5, 10),
  new Circle(3)
];

console.log(AreaCalculator.calculateTotalArea(shapes)); // Работает без изменений в AreaCalculator
```

## 3. Принцип подстановки Барбары Лисков (LSP)

> Объекты в программе должны быть заменяемыми экземплярами их подтипов без нарушения корректности программы.

Подклассы должны вести себя как экземпляры базового класса.

### Плохой пример:

```javascript
class Rectangle {
  constructor(width, height) {
    this._width = width;
    this._height = height;
  }

  set width(value) {
    this._width = value;
  }

  set height(value) {
    this._height = value;
  }

  get area() {
    return this._width * this._height;
  }
}

class Square extends Rectangle {
  set width(value) {
    this._width = value;
    this._height = value; // У квадрата ширина и высота равны
  }

  set height(value) {
    this._width = value;
    this._height = value; // У квадрата ширина и высота равны
  }
}

// Эта функция ожидает, что изменение ширины не повлияет на высоту
function testRectangle(rectangle) {
  rectangle.width = 5;
  rectangle.height = 4;
  console.log(rectangle.area); // Ожидается 20, но для Square будет 16
}

const rect = new Rectangle(2, 3);
testRectangle(rect); // Работает как ожидается

const square = new Square(2, 2);
testRectangle(square); // Нарушение LSP
```

### Хороший пример:

```javascript
class Shape {
  area() {
    throw new Error('Метод area() должен быть реализован');
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }

  area() {
    return this.side * this.side;
  }
}
```

## 4. Принцип разделения интерфейсов (ISP)

> Клиенты не должны зависеть от интерфейсов, которые они не используют.

Вместо одного "толстого" интерфейса лучше создать несколько специализированных.

### Плохой пример:

```javascript
class MultiFunctionPrinter {
  print(document) {
    // Печать документа
  }

  scan(document) {
    // Сканирование документа
  }

  fax(document) {
    // Отправка факса
  }
}

// Проблема: если нам нужен только принтер, мы все равно получаем все функции
class SimpleUser {
  constructor(printer) {
    this.printer = printer;
  }

  printDocument(doc) {
    this.printer.print(doc); // Используем только печать, но получаем все функции
  }
}
```

### Хороший пример:

```javascript
class Printer {
  print(document) {
    // Печать документа
  }
}

class Scanner {
  scan(document) {
    // Сканирование документа
  }
}

class FaxMachine {
  fax(document) {
    // Отправка факса
  }
}

// Композиция для многофункционального устройства
class MultiFunctionDevice {
  constructor() {
    this.printer = new Printer();
    this.scanner = new Scanner();
    this.faxMachine = new FaxMachine();
  }

  print(document) {
    return this.printer.print(document);
  }

  scan(document) {
    return this.scanner.scan(document);
  }

  fax(document) {
    return this.faxMachine.fax(document);
  }
}
```

## 5. Принцип инверсии зависимостей (DIP)

> 1. Модули верхних уровней не должны зависеть от модулей нижних уровней. Оба типа модулей должны зависеть от абстракций.
> 2. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

### Плохой пример:

```javascript
class MySQLDatabase {
  connect() {
    // Подключение к MySQL
  }

  query(sql) {
    // Выполнение SQL-запроса
  }
}

class UserService {
  constructor() {
    this.database = new MySQLDatabase(); // Жесткая зависимость
  }

  getUser(id) {
    return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}
```

### Хороший пример:

```javascript
class DatabaseInterface {
  connect() {
    throw new Error('Метод connect() должен быть реализован');
  }

  query(sql) {
    throw new Error('Метод query() должен быть реализован');
  }
}

class MySQLDatabase extends DatabaseInterface {
  connect() {
    // Подключение к MySQL
  }

  query(sql) {
    // Выполнение SQL-запроса
  }
}

class PostgreSQLDatabase extends DatabaseInterface {
  connect() {
    // Подключение к PostgreSQL
  }

  query(sql) {
    // Выполнение SQL-запроса
  }
}

class UserService {
  constructor(database) {
    this.database = database; // Зависимость от абстракции
  }

  getUser(id) {
    return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// Теперь мы можем легко переключаться между разными базами данных
const userService = new UserService(new PostgreSQLDatabase());
```

## Практические рекомендации

1. **Начните с SRP**: Разделяйте ответственность на небольшие, сфокусированные модули
2. **Используйте интерфейсы**: Определяйте контракты, от которых будут зависеть ваши классы
3. **Тестируйте каждый принцип**: Убедитесь, что ваш код соответствует каждому из принципов
4. **Избегайте перерегулирования**: Не применяйте принципы в ущерб простоте решения

## Заключение

Принципы SOLID помогают создавать более гибкий, поддерживаемый и тестируемый код. Однако важно применять их разумно — не все принципы подходят для всех ситуаций. Используйте их как руководство, а не как жесткие правила.

> [[Композиция-вместо-наследования]] | [[DRY]] | [[KISS]] | [[YAGNI]]
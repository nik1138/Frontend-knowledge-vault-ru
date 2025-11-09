# GRASP (General Responsibility Assignment Software Patterns)

GRASP (General Responsibility Assignment Software Patterns) — это набор принципов, используемых в объектно-ориентированном проектировании для распределения ответственностей между классами и объектами. Эти паттерны были разработаны Крейгом Ларманом и описаны в его книге "Применение UML и паттернов проектирования".

## Что такое GRASP

GRASP представляет собой набор из девяти принципов, которые помогают определить, какие классы должны быть ответственны за выполнение определенных задач в объектно-ориентированной системе. Эти принципы не являются строгими правилами, а скорее рекомендациями для создания гибких, поддерживаемых и понятных систем.

## Девять принципов GRASP

### 1. Информационный эксперт (Information Expert)

#### Определение
Класс, обладающий информацией, необходимой для выполнения какой-либо обязанности, должен быть тем классом, который выполняет эту обязанность.

#### Пример
```javascript
// Плохой пример
class OrderService {
  calculateTotal(order) {
    let total = 0;
    for (const item of order.items) {
      total += item.price * item.quantity;
    }
    return total;
  }
}

// Хороший пример
class Order {
  calculateTotal() {
    let total = 0;
    for (const item of this.items) {
      total += item.price * item.quantity;
    }
    return total;
  }
}

class OrderService {
  processOrder(order) {
    const total = order.calculateTotal(); // Order обладает информацией
    // Дальнейшая обработка
  }
}
```

#### Преимущества
- Высокая связность внутри класса
- Низкая связанность между классами
- Легкость сопровождения

### 2. Создатель (Creator)

#### Определение
Класс должен создавать экземпляры другого класса, если:
- Он агрегирует или содержит экземпляры создаваемого класса
- Он записывает экземпляры создаваемого класса
- Он активно использует экземпляры создаваемого класса
- Он обладает инициализационными данными для создаваемого класса

#### Пример
```javascript
// Хороший пример
class Order {
  constructor() {
    this.items = [];
  }
  
  // Order создает OrderItem, так как агрегирует его
  addItem(product, quantity) {
    const item = new OrderItem(product, quantity);
    this.items.push(item);
  }
}

class OrderItem {
  constructor(product, quantity) {
    this.product = product;
    this.quantity = quantity;
  }
}
```

#### Преимущества
- Четкое распределение обязанностей по созданию объектов
- Уменьшение связанности

### 3. Контроллер (Controller)

#### Определение
Класс, отвечающий за обработку системных событий, таких как запросы пользователя или сообщения от внешних систем.

#### Пример
```javascript
// Хороший пример
class UserController {
  constructor(userService) {
    this.userService = userService;
  }
  
  handleCreateUser(request) {
    try {
      const user = this.userService.createUser(request.userData);
      return { success: true, user };
    } catch (error) {
      return { success: false, error: error.message };
    }
  }
}

class UserService {
  createUser(userData) {
    // Бизнес-логика создания пользователя
    return new User(userData);
  }
}
```

#### Преимущества
- Четкое разделение между обработкой событий и бизнес-логикой
- Централизованная точка входа для системных операций

### 4. Слабое зацепление (Low Coupling)

#### Определение
Степень, в которой элементы системы связаны друг с другом. Слабое зацепление означает, что изменения в одном классе оказывают минимальное влияние на другие классы.

#### Пример
```javascript
// Плохой пример - сильное зацепление
class Order {
  constructor() {
    this.paymentProcessor = new PaymentProcessor(); // Прямая зависимость
    this.emailService = new EmailService(); // Прямая зависимость
  }
  
  process() {
    this.paymentProcessor.charge(this.total);
    this.emailService.sendConfirmation(this.customer.email);
  }
}

// Хороший пример - слабое зацепление
class Order {
  constructor(paymentProcessor, emailService) {
    this.paymentProcessor = paymentProcessor; // Зависимость через интерфейс
    this.emailService = emailService; // Зависимость через интерфейс
  }
  
  process() {
    this.paymentProcessor.charge(this.total);
    this.emailService.sendConfirmation(this.customer.email);
  }
}
```

#### Преимущества
- Легкость тестирования
- Легкость сопровождения
- Повторное использование компонентов

### 5. Высокая связность (High Cohesion)

#### Определение
Степень, в которой обязанности элемента системы связаны друг с другом. Высокая связность означает, что класс имеет четко определенную цель и все его методы работают в рамках этой цели.

#### Пример
```javascript
// Плохой пример - низкая связность
class User {
  // Управление данными пользователя
  getName() { /* ... */ }
  setName(name) { /* ... */ }
  
  // Отправка email
  sendEmail(to, subject, body) { /* ... */ }
  
  // Работа с базой данных
  saveToDatabase() { /* ... */ }
  loadFromDatabase(id) { /* ... */ }
  
  // Логирование
  log(message) { /* ... */ }
}

// Хороший пример - высокая связность
class User {
  // Только управление данными пользователя
  getName() { /* ... */ }
  setName(name) { /* ... */ }
}

class EmailService {
  sendEmail(to, subject, body) { /* ... */ }
}

class UserRepository {
  save(user) { /* ... */ }
  load(id) { /* ... */ }
}

class Logger {
  log(message) { /* ... */ }
}
```

#### Преимущества
- Легкость понимания
- Легкость сопровождения
- Повторное использование

### 6. Полиморфизм (Polymorphism)

#### Определение
Использование полиморфных операций для обработки альтернативных вариантов поведения на основе типа объекта.

#### Пример
```javascript
// Хороший пример
class PaymentProcessor {
  processPayment(paymentMethod, amount) {
    return paymentMethod.process(amount); // Полиморфный вызов
  }
}

class PaymentMethod {
  process(amount) {
    throw new Error('Метод process должен быть реализован');
  }
}

class CreditCardPayment extends PaymentMethod {
  process(amount) {
    console.log(`Обработка платежа по кредитной карте на сумму ${amount}`);
  }
}

class PayPalPayment extends PaymentMethod {
  process(amount) {
    console.log(`Обработка платежа через PayPal на сумму ${amount}`);
  }
}
```

#### Преимущества
- Гибкость системы
- Легкость добавления новых типов
- Уменьшение условных операторов

### 7. Чистая выдумка (Pure Fabrication)

#### Определение
Класс, который не представляет концепцию из предметной области, но создан для достижения низкой связанности, высокой связности и возможности повторного использования.

#### Пример
```javascript
// Хороший пример
class OrderRepository {
  // Чистая выдумка - не существует в реальном мире
  // но помогает достичь слабого зацепления
  save(order) {
    // Сохранение заказа в базе данных
  }
  
  findById(id) {
    // Поиск заказа по ID
  }
  
  findByCustomer(customerId) {
    // Поиск заказов по клиенту
  }
}

class OrderService {
  constructor(orderRepository) {
    this.orderRepository = orderRepository;
  }
  
  createOrder(orderData) {
    const order = new Order(orderData);
    this.orderRepository.save(order); // Использование репозитория
    return order;
  }
}
```

#### Преимущества
- Достижение слабого зацепления
- Повышение связности
- Возможность повторного использования

### 8. Посредник (Indirection)

#### Определение
Использование промежуточного объекта для связи между другими объектами, чтобы уменьшить прямую связанность.

#### Пример
```javascript
// Хороший пример
class UserService {
  constructor(notificationService) {
    this.notificationService = notificationService;
  }
  
  updateUser(user, newData) {
    user.update(newData);
    // Посредник для отправки уведомлений
    this.notificationService.sendUserUpdatedNotification(user);
  }
}

class NotificationService {
  sendUserUpdatedNotification(user) {
    // Отправка уведомлений через различные каналы
    this.emailService.send(user.email, 'User updated');
    this.smsService.send(user.phone, 'User updated');
  }
}
```

#### Преимущества
- Уменьшение прямой связанности
- Повышение гибкости системы

### 9. Устойчивость к изменениям (Protected Variations)

#### Определение
Шаблон проектирования, который защищает элементы от изменения другими элементами, выявляя точки, которые могут изменяться, и присваивая обязанности по созданию стабильного интерфейса вокруг них.

#### Пример
```javascript
// Хороший пример
class PaymentGateway {
  // Стабильный интерфейс
  processPayment(amount, paymentData) {
    const processor = this.getPaymentProcessor(paymentData.type);
    return processor.process(amount, paymentData);
  }
  
  // Точка изменений
  getPaymentProcessor(type) {
    switch(type) {
      case 'credit_card':
        return new CreditCardProcessor();
      case 'paypal':
        return new PayPalProcessor();
      default:
        throw new Error('Неизвестный тип платежа');
    }
  }
}
```

#### Преимущества
- Защита от изменений
- Стабильный интерфейс
- Легкость расширения

## Связь с другими концепциями

- [[SOLID]] - Принципы GRASP дополняют принципы SOLID
- [[DRY (Don't Repeat Yourself)]] - GRASP помогает избежать дублирования ответственностей
- [[KISS (Keep It Simple, Stupid)]] - GRASP способствует простоте проектирования
- [[YAGNI (You Aren't Gonna Need It)]] - GRASP помогает создавать только необходимую архитектуру
- [[Наследование]] - Принцип полиморфизма использует наследование
- [[Полиморфизм]] - Один из принципов GRASP
- [[Интерфейсы]] - Помогают реализовать принцип слабого зацепления
- [[Композиция]] - Альтернатива наследованию для достижения слабого зацепления

## Применение в TypeScript

В [[ts]] принципы GRASP реализуются особенно эффективно благодаря:
- Интерфейсам для достижения слабого зацепления
- Абстрактным классам для реализации полиморфизма
- Дженерикам для создания гибких решений
- Модулям для организации кода

## Теги
#grasp #design-patterns #oop #programming-concepts #software-architecture #responsibility-patterns
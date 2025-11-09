# Принцип DRY (Don't Repeat Yourself)

DRY (Don't Repeat Yourself) — принцип проектирования программного обеспечения, при котором каждая часть знаний или логики системы должна иметь единственное, недвусмысленное, авторитетное представление в рамках всей системы.

## Что означает DRY

Принцип DRY гласит: "Каждая часть знаний должна иметь единственное представление в системе". Это означает, что любая логика, правило, константа или информация не должны дублироваться в нескольких местах.

## Примеры нарушения принципа DRY

### Дублирование логики
```javascript
// Плохой пример
function calculateRectangleArea(width, height) {
  return width * height;
}

function calculateTriangleArea(base, height) {
  return 0.5 * base * height;
}

// Повторяющаяся валидация
function processUser(user) {
  if (!user.name || user.name.length < 2) {
    throw new Error('Имя пользователя должно содержать минимум 2 символа');
  }
  
  if (!user.email || !user.email.includes('@')) {
    throw new Error('Некорректный email');
  }
  
  // Обработка пользователя
}

function updateUser(user) {
  if (!user.name || user.name.length < 2) {
    throw new Error('Имя пользователя должно содержать минимум 2 символа');
  }
  
  if (!user.email || !user.email.includes('@')) {
    throw new Error('Некорректный email');
  }
  
  // Обновление пользователя
}
```

### Дублирование констант
```javascript
// Плохой пример
class PaymentService {
  processCreditCard(amount) {
    const taxRate = 0.08; // 8% налог
    return amount * (1 + taxRate);
  }
  
  processDebitCard(amount) {
    const taxRate = 0.08; // 8% налог (дублирование)
    return amount * (1 + taxRate);
  }
}

class OrderService {
  calculateTotal(items) {
    const taxRate = 0.08; // 8% налог (дублирование)
    const subtotal = items.reduce((sum, item) => sum + item.price, 0);
    return subtotal * (1 + taxRate);
  }
}
```

## Исправленные примеры

### Вынесение общей логики
```javascript
// Хороший пример
class ValidationService {
  static validateUser(user) {
    if (!user.name || user.name.length < 2) {
      throw new Error('Имя пользователя должно содержать минимум 2 символа');
    }
    
    if (!user.email || !user.email.includes('@')) {
      throw new Error('Некорректный email');
    }
  }
}

function processUser(user) {
  ValidationService.validateUser(user);
  // Обработка пользователя
}

function updateUser(user) {
  ValidationService.validateUser(user);
  // Обновление пользователя
}

// Вынесение математических операций
class MathUtils {
  static calculateArea(shape, ...params) {
    switch(shape) {
      case 'rectangle':
        return params[0] * params[1];
      case 'triangle':
        return 0.5 * params[0] * params[1];
      case 'circle':
        return Math.PI * params[0] * params[0];
      default:
        throw new Error('Неизвестная фигура');
    }
  }
}
```

### Централизация констант
```javascript
// Хороший пример
class Constants {
  static TAX_RATE = 0.08; // 8% налог
  static CURRENCY = 'RUB';
  static MAX_RETRY_ATTEMPTS = 3;
}

class PaymentService {
  processCreditCard(amount) {
    return amount * (1 + Constants.TAX_RATE);
  }
  
  processDebitCard(amount) {
    return amount * (1 + Constants.TAX_RATE);
  }
}

class OrderService {
  calculateTotal(items) {
    const subtotal = items.reduce((sum, item) => sum + item.price, 0);
    return subtotal * (1 + Constants.TAX_RATE);
  }
}
```

## Принцип DRY в CSS

### Дублирование стилей
```css
/* Плохой пример */
.header {
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  color: #333333;
  margin: 0;
  padding: 0;
}

.navigation {
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  color: #333333;
  margin: 0;
  padding: 0;
}

.footer {
  font-family: 'Arial', sans-serif;
  font-size: 16px;
  color: #333333;
  margin: 0;
  padding: 0;
}
```

### Использование CSS-переменных и классов
```css
/* Хороший пример */
:root {
  --base-font-family: 'Arial', sans-serif;
  --base-font-size: 16px;
  --base-text-color: #333333;
}

.base-text {
  font-family: var(--base-font-family);
  font-size: var(--base-font-size);
  color: var(--base-text-color);
  margin: 0;
  padding: 0;
}

.header {
  composes: base-text;
}

.navigation {
  composes: base-text;
}

.footer {
  composes: base-text;
}
```

## Принцип DRY в HTML

### Дублирование структуры
```html
<!-- Плохой пример -->
<header>
  <nav>
    <ul>
      <li><a href="/">Главная</a></li>
      <li><a href="/about">О нас</a></li>
      <li><a href="/services">Услуги</a></li>
      <li><a href="/contact">Контакты</a></li>
    </ul>
  </nav>
</header>

<!-- Повторяющаяся навигация в подвале -->
<footer>
  <nav>
    <ul>
      <li><a href="/">Главная</a></li>
      <li><a href="/about">О нас</a></li>
      <li><a href="/services">Услуги</a></li>
      <li><a href="/contact">Контакты</a></li>
    </ul>
  </nav>
</footer>
```

### Использование шаблонов
```html
<!-- Хороший пример -->
<!-- navigation.html -->
<nav>
  <ul>
    <li><a href="/">Главная</a></li>
    <li><a href="/about">О нас</a></li>
    <li><a href="/services">Услуги</a></li>
    <li><a href="/contact">Контакты</a></li>
  </ul>
</nav>

<!-- header.html -->
<header>
  @@include('navigation.html')
</header>

<!-- footer.html -->
<footer>
  @@include('navigation.html')
</footer>
```

## Преимущества соблюдения принципа DRY

### Упрощение сопровождения
Когда логика находится в одном месте, изменения нужно вносить только в одном месте, что снижает вероятность ошибок.

### Повышение надежности
Меньше дублирования означает меньше возможностей для рассинхронизации кода.

### Улучшение читаемости
Код становится более структурированным и понятным.

### Повышение переиспользуемости
Общие компоненты и функции можно использовать в разных частях системы.

## Возможные проблемы при применении DRY

### Преждевременная абстракция
Иногда разработчики создают абстракции слишком рано, когда еще не видна реальная необходимость.

### Сложность абстракций
Излишне сложные абстракции могут сделать код менее понятным.

### Связанность компонентов
Неправильное применение DRY может привести к чрезмерной связанности компонентов.

## Баланс между DRY и KISS

Иногда принципы DRY и [[KISS (Keep It Simple, Stupid)]] могут конфликтовать. Важно находить баланс между избеганием дублирования и сохранением простоты.

## Связь с другими принципами

- [[SOLID]] - Принципы SOLID помогают реализовать DRY на уровне архитектуры
- [[YAGNI (You Aren't Gonna Need It)]] - Не создавайте абстракции до необходимости
- [[Чистый код]] - DRY является частью концепции чистого кода
- [[Наследование]] - Наследование помогает избежать дублирования кода
- [[Композиция]] - Композиция может помочь избежать дублирования
- [[Интерфейсы]] - Интерфейсы помогают определить общий контракт для избежания дублирования
- [[Дженерики]] - Дженерики позволяют писать универсальный код без дублирования

## Применение в TypeScript

В [[ts]] принцип DRY реализуется через:
- Интерфейсы и типы для избежания дублирования структур данных
- Дженерики для создания переиспользуемых компонентов
- Модули и пространства имен для организации кода

## Теги
#dry #design-principles #programming-concepts #refactoring #clean-code
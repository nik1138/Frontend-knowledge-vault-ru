# JavaScript

JavaScript — это легковесный, интерпретируемый язык программирования с функциями первого класса. Это язык сценариев, который поддерживает объектно-ориентированное, императивное и функциональное стили программирования. JavaScript является основной языковой технологией для World Wide Web.

## Основные характеристики

### Динамическая типизация
JavaScript использует динамическую типизацию, что означает, что типы переменных проверяются во время выполнения.

```javascript
// Пример динамической типизации
let value = 42;        // число
value = "hello";       // строка
value = true;          // булево значение
```

### Функции первого класса
В JavaScript функции являются объектами первого класса, что означает, что их можно присваивать переменным, передавать как аргументы и возвращать из других функций.

```javascript
// Функция как значение переменной
const greet = function(name) {
  return `Hello, ${name}!`;
};

// Функция как аргумент
function execute(callback, arg) {
  return callback(arg);
}

execute(greet, "World"); // "Hello, World!"
```

### Прототипное наследование
JavaScript использует прототипное наследование вместо классического наследования, как в других языках.

```javascript
// Создание объекта
const animal = {
  speak() {
    console.log(`${this.name} makes a noise.`);
  }
};

// Создание объекта с прототипом
const dog = Object.create(animal);
dog.name = "Rex";
dog.speak(); // "Rex makes a noise."
```

## Парадигмы программирования

JavaScript поддерживает несколько парадигм программирования:

### Императивное программирование
```javascript
// Императивный стиль
const numbers = [1, 2, 3, 4, 5];
const doubled = [];
for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}
```

### Функциональное программирование
```javascript
// Функциональный стиль
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(x => x * 2);
```

### Объектно-ориентированное программирование
```javascript
// Современный синтаксис классов
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a noise.`);
  }
}

class Dog extends Animal {
  speak() {
    console.log(`${this.name} barks.`);
  }
}
```

## Преимущества JavaScript

1. **Универсальность** — Работает как в браузере, так и на сервере (Node.js)
2. **Гибкость** — Поддерживает различные стили программирования
3. **Большое сообщество** — Огромная экосистема библиотек и фреймворков
4. **Низкий порог входа** — Простота изучения для начинающих
5. **Асинхронность** — Встроенная поддержка асинхронных операций

## Связанные концепции

- [[Императивное программирование]] - JavaScript поддерживает императивный стиль программирования
- [[Функциональное программирование]] - JavaScript поддерживает функциональный стиль программирования
- [[Объектно-ориентированное программирование]] - JavaScript поддерживает ООП через прототипы и классы
- [[Наследование]] - JavaScript поддерживает наследование через прототипы
- [[Полиморфизм]] - JavaScript поддерживает полиморфизм через динамическую типизацию
- [[Композиция]] - JavaScript поддерживает композицию объектов
- [[DRY (Don't Repeat Yourself)]] - JavaScript предоставляет возможности для избегания дублирования
- [[KISS (Keep It Simple, Stupid)]] - JavaScript позволяет писать простой код для простых задач
- [[Чистый код]] - JavaScript поддерживает написание чистого кода в различных стилях

## В других технологиях

### В [[ts]]
TypeScript является надмножеством JavaScript, добавляя статическую типизацию:

```typescript
// JavaScript
function add(a, b) {
  return a + b;
}

// TypeScript
function add(a: number, b: number): number {
  return a + b;
}
```

### В [[react]]
React использует JavaScript (или TypeScript) для создания компонентов:

```jsx
// Функциональный компонент
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Компонент с хуками
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### В [[vue]]
Vue.js также использует JavaScript для создания реактивных компонентов:

```javascript
// Vue компонент
export default {
  data() {
    return {
      message: 'Hello Vue!'
    }
  },
  methods: {
    greet() {
      alert(this.message);
    }
  }
}
```

## Теги
#javascript #web-development #programming-languages #frontend #backend
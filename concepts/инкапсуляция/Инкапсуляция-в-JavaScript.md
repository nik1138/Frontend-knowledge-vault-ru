---
aliases: [Инкапсуляция в JS, Приватность в JavaScript]
tags: [programming, javascript, oop, encapsulation]
---

# Инкапсуляция в JavaScript

Инкапсуляция - это один из основных принципов объектно-ориентированного программирования, который позволяет скрыть внутренние детали реализации объекта от внешнего мира. В JavaScript инкапсуляция достигается различными способами, включая использование замыканий, приватных полей и других паттернов.

## Обзор инкапсуляции в JavaScript

JavaScript предоставляет несколько подходов для реализации инкапсуляции:
- Классы с приватными полями (начиная с ES2022)
- Замыкания и IIFE (Immediately Invoked Function Expression)
- Модульный паттерн
- Приватные методы и поля в классах

## Классы с приватными полями

Начиная с ES2022, JavaScript поддерживает приватные поля и методы в классах с использованием символа `#`:

```javascript
class BankAccount {
  #balance = 0;
  #accountNumber;

  constructor(accountNumber) {
    this.#accountNumber = accountNumber;
  }

  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
      console.log(`Внесено: ${amount}. Баланс: ${this.#balance}`);
    }
  }

  getBalance() {
    return this.#balance;
  }

  #validateTransaction(amount) {
    return amount > 0 && amount <= this.#balance;
  }

  withdraw(amount) {
    if (this.#validateTransaction(amount)) {
      this.#balance -= amount;
      console.log(`Снято: ${amount}. Баланс: ${this.#balance}`);
      return true;
    }
    console.error('Недопустимая транзакция');
    return false;
  }
}

const account = new BankAccount('12345');
account.deposit(100);
console.log(account.getBalance()); // 100
// console.log(account.#balance); // Ошибка: синтаксическая ошибка
```

## Замыкания для инкапсуляции

Замыкания позволяют создавать приватные переменные и методы:

```javascript
function createCounter() {
  let count = 0;

  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount());  // 2
// console.log(counter.count); // undefined - приватная переменная
```

## Модульный паттерн

Модульный паттерн использует IIFE для создания приватного пространства:

```javascript
const Calculator = (function() {
  let history = [];

  function addToHistory(operation, result) {
    history.push({ operation, result, timestamp: new Date() });
  }

  return {
    add(a, b) {
      const result = a + b;
      addToHistory(`${a} + ${b}`, result);
      return result;
    },
    
    subtract(a, b) {
      const result = a - b;
      addToHistory(`${a} - ${b}`, result);
      return result;
    },
    
    getHistory() {
      return [...history]; // возвращаем копию для безопасности
    },
    
    clearHistory() {
      history = [];
    }
  };
})();

console.log(Calculator.add(5, 3)); // 8
console.log(Calculator.getHistory()); // [{ operation: "5 + 3", result: 8, timestamp: ... }]
// history недоступна извне
```

## Сравнение подходов

| Подход | Преимущества | Недостатки |
|--------|--------------|------------|
| Приватные поля классов | Чистый синтаксис, настоящая приватность | Требует современного браузера |
| Замыкания | Работает во всех версиях JS | Может привести к утечке памяти |
| Модульный паттерн | Хорошо подходит для глобальных объектов | Более сложный синтаксис |

## Практические рекомендации

1. Используйте приватные поля классов, когда это возможно (современные браузеры)
2. Для старых браузеров используйте замыкания
3. Избегайте публичного доступа к внутреннему состоянию
4. Документируйте публичный интерфейс

## Заключение

Инкапсуляция в JavaScript может быть реализована различными способами в зависимости от требований проекта и поддерживаемых браузеров. Современные возможности языка позволяют создавать действительно приватные поля и методы, что делает код более безопасным и предсказуемым.

## См. также
- [[Приватные-поля-и-методы]]
- [[Инкапсуляция-в-React]]
- [[Инкапсуляция-в-Vue]]
- [[Инкапсуляция-в-Svelte]]
- [[Объектно-ориентированное программирование в JavaScript]]

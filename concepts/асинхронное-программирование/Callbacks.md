---
aliases: [Колбэки, Асинхронные колбэки]
tags: [javascript, async-programming, callbacks, frontend]
---

# Callbacks (Колбэки)

## Описание

Колбэки (callbacks) - это функции, которые передаются в другую функцию в качестве аргументов и вызываются после завершения определенной операции. Это один из самых ранних способов обработки асинхронных операций в JavaScript.

## Применение

Колбэки широко использовались до появления промисов и async/await, и до сих пор встречаются в старом коде, а также в некоторых API, таких как `setTimeout`, `addEventListener`, и многих методах массивов.

### Примеры использования

```javascript
// Пример асинхронного колбэка с setTimeout
function delayedGreeting(name, callback) {
    setTimeout(() => {
        console.log(`Привет, ${name}!`);
        callback();
    }, 1000);
}

delayedGreeting('Алексей', () => {
    console.log('Приветствие завершено');
});
```

```javascript
// Пример с обработчиком события
document.getElementById('myButton').addEventListener('click', function() {
    console.log('Кнопка нажата!');
});
```

```javascript
// Пример синхронного колбэка в методах массива
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
```

## Проблемы с колбэками

### Callback Hell (Ад колбэков)

Когда асинхронные операции зависят друг от друга, легко получить глубоко вложенные колбэки, что делает код трудночитаемым и труднообслуживаемым.

```javascript
// Пример "Callback Hell"
getData(function(a) {
    getMoreData(a, function(b) {
        getEvenMoreData(b, function(c) {
            getEvenEvenMoreData(c, function(d) {
                // И так далее...
            });
        });
    });
});
```

### Решение проблем с колбэками

1. **Именованные функции** - помогают уменьшить вложенность
2. **Обработка ошибок** - стандартный подход с передачей ошибки первым аргументом
3. **Переход на промисы** - более современный подход к асинхронному программированию

```javascript
// Использование именованных функций для уменьшения вложенности
function handleFirstStep(error, resultA) {
    if (error) {
        console.error('Ошибка на первом шаге:', error);
        return;
    }
    getMoreData(resultA, handleSecondStep);
}

function handleSecondStep(error, resultB) {
    if (error) {
        console.error('Ошибка на втором шаге:', error);
        return;
    }
    getEvenMoreData(resultB, handleThirdStep);
}

function handleThirdStep(error, resultC) {
    if (error) {
        console.error('Ошибка на третьем шаге:', error);
        return;
    }
    console.log('Все данные получены:', resultC);
}

getData(handleFirstStep);
```

## Практические советы

1. **Используйте именованные функции** вместо анонимных для лучшей читаемости
2. **Обрабатывайте ошибки** в колбэках, особенно в асинхронных операциях
3. **Избегайте глубокой вложенности** - при необходимости разбивайте на несколько функций
4. **Предпочитайте современные подходы** (примисы, async/await) при новой разработке

## Связанные темы

- [[Promises]] - более современный способ обработки асинхронных операций
- [[Async-Await]] - синтаксический сахар над промисами
- [[Event-Loop]] - как JavaScript обрабатывает асинхронные операции
- [[Обработка-ошибок-в-асинхронном-коде]] - особенности обработки ошибок в асинхронном коде
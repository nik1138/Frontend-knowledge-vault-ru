# Упражнения по области видимости и замыканиям

## Упражнение 9: Область видимости
Практикуйте понимание области видимости:

```javascript
// Задание: Исследуйте различные области видимости
// 1. Глобальная область видимости
// 2. Локальная область видимости
// 3. Блочная область видимости
// 4. Замыкания

// Глобальная переменная
let globalVar = "Я глобальная переменная";

function scopeExample() {
  // Локальная переменная
  let localVar = "Я локальная переменная";
  
  if (true) {
    // Блочная переменная
    let blockVar = "Я блочная переменная";
    var functionVar = "Я функциональная переменная";
    
    console.log(globalVar); // Доступна
    console.log(localVar);  // Доступна
    console.log(blockVar);  // Доступна
    console.log(functionVar); // Доступна
  }
  
  console.log(globalVar); // Доступна
  console.log(localVar);  // Доступна
  // console.log(blockVar);  // Ошибка: не определена
  console.log(functionVar); // Доступна (var всплывает)
}

// Замыкание
function createCounter() {
  let count = 0;
  
  return function() {
    count++;
    return count;
  };
}

const counter1 = createCounter();
const counter2 = createCounter();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1 (независимый счетчик)
console.log(counter1()); // 3
```

## Упражнение 10: Практическое замыкание
Создайте практический пример использования замыкания:

```javascript
// Задание: Создайте модуль для управления списком задач
function createTaskManager() {
  let tasks = [];
  
  return {
    addTask: function(task) {
      tasks.push({
        id: Date.now(),
        text: task,
        completed: false,
        createdAt: new Date()
      });
    },
    
    completeTask: function(taskId) {
      const task = tasks.find(t => t.id === taskId);
      if (task) {
        task.completed = true;
      }
    },
    
    removeTask: function(taskId) {
      tasks = tasks.filter(t => t.id !== taskId);
    },
    
    getTasks: function() {
      return [...tasks]; // Возвращаем копию массива
    },
    
    getCompletedTasks: function() {
      return tasks.filter(t => t.completed);
    },
    
    getPendingTasks: function() {
      return tasks.filter(t => !t.completed);
    }
  };
}

// Использование
const taskManager = createTaskManager();

taskManager.addTask("Изучить JavaScript");
taskManager.addTask("Сделать упражнения");
taskManager.addTask("Написать проект");

console.log("Все задачи:", taskManager.getTasks());

taskManager.completeTask(taskManager.getTasks()[0].id);
console.log("Завершенные задачи:", taskManager.getCompletedTasks());
console.log("Ожидающие задачи:", taskManager.getPendingTasks());
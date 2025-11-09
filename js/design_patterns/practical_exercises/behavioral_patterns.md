# Практические упражнения: Поведенческие паттерны проектирования

## Упражнение 1: Стратегия (Strategy)

### Задача
Создайте систему сортировки массивов с возможностью выбора различных алгоритмов сортировки, используя паттерн стратегия.

### Требования
1. Различные алгоритмы сортировки как стратегии
2. Контекст для выполнения сортировки
3. Возможность динамической смены алгоритма
4. Измерение времени выполнения каждого алгоритма

### Решение
```javascript
// Интерфейс стратегии
class SortStrategy {
  sort(array) {
    throw new Error('Метод sort должен быть реализован');
  }
}

// Стратегия: Пузырьковая сортировка
class BubbleSortStrategy extends SortStrategy {
  sort(array) {
    console.log('Используется пузырьковая сортировка');
    const arr = [...array]; // Создаем копию массива
    const n = arr.length;
    
    for (let i = 0; i < n - 1; i++) {
      for (let j = 0; j < n - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    
    return arr;
  }
}

// Стратегия: Быстрая сортировка
class QuickSortStrategy extends SortStrategy {
  sort(array) {
    console.log('Используется быстрая сортировка');
    return this.quickSort([...array], 0, array.length - 1);
  }
  
  quickSort(arr, low, high) {
    if (low < high) {
      const pi = this.partition(arr, low, high);
      this.quickSort(arr, low, pi - 1);
      this.quickSort(arr, pi + 1, high);
    }
    return arr;
  }
  
  partition(arr, low, high) {
    const pivot = arr[high];
    let i = low - 1;
    
    for (let j = low; j < high; j++) {
      if (arr[j] < pivot) {
        i++;
        [arr[i], arr[j]] = [arr[j], arr[i]];
      }
    }
    
    [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
    return i + 1;
  }
}

// Стратегия: Сортировка вставками
class InsertionSortStrategy extends SortStrategy {
  sort(array) {
    console.log('Используется сортировка вставками');
    const arr = [...array];
    
    for (let i = 1; i < arr.length; i++) {
      const key = arr[i];
      let j = i - 1;
      
      while (j >= 0 && arr[j] > key) {
        arr[j + 1] = arr[j];
        j--;
      }
      
      arr[j + 1] = key;
    }
    
    return arr;
  }
}

// Контекст
class SortContext {
  constructor(strategy) {
    this.setStrategy(strategy);
  }
  
  setStrategy(strategy) {
    if (!(strategy instanceof SortStrategy)) {
      throw new Error('Стратегия должна быть экземпляром SortStrategy');
    }
    this.strategy = strategy;
  }
  
  executeSort(array) {
    console.log(`Сортировка массива из ${array.length} элементов...`);
    const startTime = performance.now();
    
    const sortedArray = this.strategy.sort(array);
    
    const endTime = performance.now();
    console.log(`Сортировка выполнена за ${(endTime - startTime).toFixed(2)} мс`);
    
    return sortedArray;
  }
  
  // Метод для сравнения всех стратегий
  compareStrategies(array) {
    const strategies = [
      new BubbleSortStrategy(),
      new QuickSortStrategy(),
      new InsertionSortStrategy()
    ];
    
    const results = {};
    
    strategies.forEach(strategy => {
      this.setStrategy(strategy);
      const startTime = performance.now();
      this.executeSort(array);
      const endTime = performance.now();
      results[strategy.constructor.name] = (endTime - startTime).toFixed(2);
    });
    
    return results;
  }
}

// Пример использования
const context = new SortContext(new QuickSortStrategy());

// Тестовый массив
const testArray = [64, 34, 25, 12, 22, 11, 90, 88, 76, 50, 42];

// Сортировка с разными стратегиями
console.log('=== Быстрая сортировка ===');
console.log('Отсортированный массив:', context.executeSort(testArray));

console.log('\n=== Пузырьковая сортировка ===');
context.setStrategy(new BubbleSortStrategy());
console.log('Отсортированный массив:', context.executeSort(testArray));

console.log('\n=== Сортировка вставками ===');
context.setStrategy(new InsertionSortStrategy());
console.log('Отсортированный массив:', context.executeSort(testArray));

// Сравнение производительности
console.log('\n=== Сравнение производительности ===');
const largeArray = Array.from({ length: 1000 }, () => Math.floor(Math.random() * 1000));
const performanceResults = context.compareStrategies(largeArray);
console.log('Результаты сравнения (мс):', performanceResults);
```

## Упражнение 2: Команда (Command)

### Задача
Создайте систему управления историей команд текстового редактора с возможностью отмены и повтора действий, используя паттерн команда.

### Требования
1. Интерфейс команды с методами выполнения и отмены
2. Конкретные команды для различных действий редактора
3. История команд с возможностью отмены и повтора
4. Текстовый редактор как получатель команд

### Решение
```javascript
// Интерфейс команды
class Command {
  execute() {
    throw new Error('Метод execute должен быть реализован');
  }
  
  undo() {
    throw new Error('Метод undo должен быть реализован');
  }
  
  getDescription() {
    throw new Error('Метод getDescription должен быть реализован');
  }
}

// Получатель - текстовый редактор
class TextEditor {
  constructor() {
    this.content = '';
    this.cursorPosition = 0;
  }
  
  insertText(text, position = this.cursorPosition) {
    const before = this.content.substring(0, position);
    const after = this.content.substring(position);
    this.content = before + text + after;
    this.cursorPosition = position + text.length;
  }
  
  deleteText(start, length) {
    const before = this.content.substring(0, start);
    const after = this.content.substring(start + length);
    this.content = before + after;
    
    if (this.cursorPosition > start) {
      this.cursorPosition = Math.max(start, this.cursorPosition - length);
    }
  }
  
  replaceText(oldText, newText, position) {
    const index = this.content.indexOf(oldText, position);
    if (index !== -1) {
      this.deleteText(index, oldText.length);
      this.insertText(newText, index);
    }
  }
  
  getContent() {
    return this.content;
  }
  
  getCursorPosition() {
    return this.cursorPosition;
  }
  
  setCursorPosition(position) {
    this.cursorPosition = Math.max(0, Math.min(position, this.content.length));
  }
}

// Команда вставки текста
class InsertTextCommand extends Command {
  constructor(editor, text, position) {
    super();
    this.editor = editor;
    this.text = text;
    this.position = position !== undefined ? position : editor.getCursorPosition();
  }
  
  execute() {
    this.editor.insertText(this.text, this.position);
  }
  
  undo() {
    this.editor.deleteText(this.position, this.text.length);
  }
  
  getDescription() {
    return `Вставка текста "${this.text}" на позиции ${this.position}`;
  }
}

// Команда удаления текста
class DeleteTextCommand extends Command {
  constructor(editor, start, length) {
    super();
    this.editor = editor;
    this.start = start;
    this.length = length;
    this.deletedText = '';
  }
  
  execute() {
    this.deletedText = this.editor.getContent().substring(this.start, this.start + this.length);
    this.editor.deleteText(this.start, this.length);
  }
  
  undo() {
    this.editor.insertText(this.deletedText, this.start);
  }
  
  getDescription() {
    return `Удаление ${this.length} символов с позиции ${this.start}`;
  }
}

// Команда замены текста
class ReplaceTextCommand extends Command {
  constructor(editor, oldText, newText, position = 0) {
    super();
    this.editor = editor;
    this.oldText = oldText;
    this.newText = newText;
    this.position = position;
  }
  
  execute() {
    const content = this.editor.getContent();
    const index = content.indexOf(this.oldText, this.position);
    if (index !== -1) {
      this.editor.replaceText(this.oldText, this.newText, this.position);
      this.replacedPosition = index;
    }
  }
  
  undo() {
    if (this.replacedPosition !== undefined) {
      this.editor.replaceText(this.newText, this.oldText, this.replacedPosition);
    }
  }
  
  getDescription() {
    return `Замена "${this.oldText}" на "${this.newText}"`;
  }
}

// Команда перемещения курсора
class MoveCursorCommand extends Command {
  constructor(editor, newPosition) {
    super();
    this.editor = editor;
    this.newPosition = newPosition;
    this.oldPosition = editor.getCursorPosition();
  }
  
  execute() {
    this.editor.setCursorPosition(this.newPosition);
  }
  
  undo() {
    this.editor.setCursorPosition(this.oldPosition);
  }
  
  getDescription() {
    return `Перемещение курсора с ${this.oldPosition} на ${this.newPosition}`;
  }
}

// Инвокер - менеджер команд
class CommandManager {
  constructor() {
    this.history = [];
    this.undoStack = [];
  }
  
  executeCommand(command) {
    try {
      command.execute();
      this.history.push(command);
      this.undoStack = []; // Очищаем стек отмены при выполнении новой команды
      console.log(`Выполнена команда: ${command.getDescription()}`);
    } catch (error) {
      console.error(`Ошибка выполнения команды: ${error.message}`);
    }
  }
  
  undo() {
    if (this.history.length > 0) {
      const command = this.history.pop();
      try {
        command.undo();
        this.undoStack.push(command);
        console.log(`Отменена команда: ${command.getDescription()}`);
      } catch (error) {
        console.error(`Ошибка отмены команды: ${error.message}`);
        // Восстанавливаем команду в истории
        this.history.push(command);
      }
    } else {
      console.log('Нет команд для отмены');
    }
  }
  
  redo() {
    if (this.undoStack.length > 0) {
      const command = this.undoStack.pop();
      try {
        command.execute();
        this.history.push(command);
        console.log(`Повторена команда: ${command.getDescription()}`);
      } catch (error) {
        console.error(`Ошибка повтора команды: ${error.message}`);
        // Возвращаем команду в стек отмены
        this.undoStack.push(command);
      }
    } else {
      console.log('Нет команд для повтора');
    }
  }
  
  getHistory() {
    return this.history.map(cmd => cmd.getDescription());
  }
  
  getUndoStack() {
    return this.undoStack.map(cmd => cmd.getDescription());
  }
  
  clearHistory() {
    this.history = [];
    this.undoStack = [];
    console.log('История команд очищена');
  }
}

// Пример использования
const editor = new TextEditor();
const commandManager = new CommandManager();

// Выполняем команды
commandManager.executeCommand(new InsertTextCommand(editor, "Hello", 0));
console.log('Содержимое:', editor.getContent());

commandManager.executeCommand(new InsertTextCommand(editor, " World", editor.getCursorPosition()));
console.log('Содержимое:', editor.getContent());

commandManager.executeCommand(new InsertTextCommand(editor, "!", editor.getCursorPosition()));
console.log('Содержимое:', editor.getContent());

// Отменяем последнюю команду
commandManager.undo();
console.log('После отмены:', editor.getContent());

// Отменяем еще одну команду
commandManager.undo();
console.log('После второй отмены:', editor.getContent());

// Повторяем последнюю отмененную команду
commandManager.redo();
console.log('После повтора:', editor.getContent());

// Выполняем замену текста
commandManager.executeCommand(new ReplaceTextCommand(editor, "Hello", "Hi"));
console.log('После замены:', editor.getContent());

// Перемещаем курсор и вставляем текст
commandManager.executeCommand(new MoveCursorCommand(editor, 2));
commandManager.executeCommand(new InsertTextCommand(editor, " there", editor.getCursorPosition()));
console.log('После вставки:', editor.getContent());

// Показываем историю
console.log('\nИстория команд:', commandManager.getHistory());
console.log('Стек отмены:', commandManager.getUndoStack());
```

## Упражнение 3: Итератор (Iterator)

### Задача
Создайте собственную реализацию итератора для коллекции пользовательских данных с поддержкой различных порядков обхода.

### Требования
1. Интерфейс итератора с методами next, hasNext, reset
2. Различные типы итераторов (прямой, обратный, по четным/нечетным индексам)
3. Коллекцию данных с методом получения итератора
4. Поддержку фильтрации и преобразования данных при итерации

### Решение
```javascript
// Интерфейс итератора
class Iterator {
  next() {
    throw new Error('Метод next должен быть реализован');
  }
  
  hasNext() {
    throw new Error('Метод hasNext должен быть реализован');
  }
  
  reset() {
    throw new Error('Метод reset должен быть реализован');
  }
}

// Базовый класс итератора
class BaseIterator extends Iterator {
  constructor(collection) {
    super();
    this.collection = collection;
    this.position = 0;
  }
  
  hasNext() {
    return this.position < this.collection.size();
  }
  
  reset() {
    this.position = 0;
  }
}

// Итератор прямого обхода
class ForwardIterator extends BaseIterator {
  next() {
    if (this.hasNext()) {
      return this.collection.get(this.position++);
    }
    return null;
  }
}

// Итератор обратного обхода
class BackwardIterator extends BaseIterator {
  constructor(collection) {
    super(collection);
    this.position = collection.size() - 1;
  }
  
  next() {
    if (this.position >= 0) {
      return this.collection.get(this.position--);
    }
    return null;
  }
  
  hasNext() {
    return this.position >= 0;
  }
  
  reset() {
    this.position = this.collection.size() - 1;
  }
}

// Итератор четных индексов
class EvenIndexIterator extends BaseIterator {
  next() {
    while (this.position < this.collection.size()) {
      if (this.position % 2 === 0) {
        const result = this.collection.get(this.position);
        this.position++;
        return result;
      }
      this.position++;
    }
    return null;
  }
  
  hasNext() {
    let tempPosition = this.position;
    while (tempPosition < this.collection.size()) {
      if (tempPosition % 2 === 0) {
        return true;
      }
      tempPosition++;
    }
    return false;
  }
}

// Итератор нечетных индексов
class OddIndexIterator extends BaseIterator {
  next() {
    while (this.position < this.collection.size()) {
      if (this.position % 2 === 1) {
        const result = this.collection.get(this.position);
        this.position++;
        return result;
      }
      this.position++;
    }
    return null;
  }
  
  hasNext() {
    let tempPosition = this.position;
    while (tempPosition < this.collection.size()) {
      if (tempPosition % 2 === 1) {
        return true;
      }
      tempPosition++;
    }
    return false;
  }
}

// Итератор с фильтрацией
class FilterIterator extends BaseIterator {
  constructor(collection, filterFunction) {
    super(collection);
    this.filterFunction = filterFunction;
  }
  
  next() {
    while (this.position < this.collection.size()) {
      const item = this.collection.get(this.position++);
      if (this.filterFunction(item)) {
        return item;
      }
    }
    return null;
  }
  
  hasNext() {
    let tempPosition = this.position;
    while (tempPosition < this.collection.size()) {
      const item = this.collection.get(tempPosition);
      if (this.filterFunction(item)) {
        return true;
      }
      tempPosition++;
    }
    return false;
  }
}

// Итератор с преобразованием
class MapIterator extends BaseIterator {
  constructor(collection, mapFunction) {
    super(collection);
    this.mapFunction = mapFunction;
  }
  
  next() {
    if (this.hasNext()) {
      const item = this.collection.get(this.position++);
      return this.mapFunction(item);
    }
    return null;
  }
}

// Коллекция данных
class DataCollection {
  constructor() {
    this.items = [];
  }
  
  add(item) {
    this.items.push(item);
  }
  
  get(index) {
    return this.items[index];
  }
  
  size() {
    return this.items.length;
  }
  
  // Методы для получения различных итераторов
  createIterator(type = 'forward') {
    switch (type.toLowerCase()) {
      case 'forward':
        return new ForwardIterator(this);
      case 'backward':
        return new BackwardIterator(this);
      case 'even':
        return new EvenIndexIterator(this);
      case 'odd':
        return new OddIndexIterator(this);
      default:
        throw new Error(`Неизвестный тип итератора: ${type}`);
    }
  }
  
  createFilterIterator(filterFunction) {
    return new FilterIterator(this, filterFunction);
  }
  
  createMapIterator(mapFunction) {
    return new MapIterator(this, mapFunction);
  }
  
  // Метод для использования Symbol.iterator
  [Symbol.iterator]() {
    const iterator = new ForwardIterator(this);
    return {
      next: () => {
        if (iterator.hasNext()) {
          return { value: iterator.next(), done: false };
        }
        return { done: true };
      }
    };
  }
}

// Пример использования
const collection = new DataCollection();

// Добавляем данные
const users = [
  { id: 1, name: 'Alice', age: 25, role: 'developer' },
  { id: 2, name: 'Bob', age: 30, role: 'designer' },
  { id: 3, name: 'Charlie', age: 35, role: 'manager' },
  { id: 4, name: 'David', age: 28, role: 'developer' },
  { id: 5, name: 'Eve', age: 32, role: 'tester' },
  { id: 6, name: 'Frank', age: 27, role: 'developer' }
];

users.forEach(user => collection.add(user));

console.log('=== Прямой обход ===');
const forwardIterator = collection.createIterator('forward');
while (forwardIterator.hasNext()) {
  console.log(forwardIterator.next());
}

console.log('\n=== Обратный обход ===');
const backwardIterator = collection.createIterator('backward');
while (backwardIterator.hasNext()) {
  console.log(backwardIterator.next());
}

console.log('\n=== Четные индексы ===');
const evenIterator = collection.createIterator('even');
while (evenIterator.hasNext()) {
  console.log(evenIterator.next());
}

console.log('\n=== Нечетные индексы ===');
const oddIterator = collection.createIterator('odd');
while (oddIterator.hasNext()) {
  console.log(oddIterator.next());
}

console.log('\n=== Фильтрация по роли ===');
const developerIterator = collection.createFilterIterator(user => user.role === 'developer');
while (developerIterator.hasNext()) {
  console.log(developerIterator.next());
}

console.log('\n=== Преобразование данных ===');
const nameIterator = collection.createMapIterator(user => user.name);
while (nameIterator.hasNext()) {
  console.log(nameIterator.next());
}

console.log('\n=== Комбинирование итераторов ===');
// Итератор разработчиков, преобразованный в имена
const developers = collection.createFilterIterator(user => user.role === 'developer');
const developerNamesIterator = new MapIterator(developers.collection, user => user.name);
developers.position = 0; // Сбрасываем позицию

while (developers.hasNext()) {
  const user = developers.next();
  if (user && user.role === 'developer') {
    console.log(developerNamesIterator.mapFunction(user));
  }
}

console.log('\n=== Использование Symbol.iterator ===');
for (const user of collection) {
  console.log(user.name);
}

// Создание итератора с несколькими фильтрами
console.log('\n=== Сложная фильтрация ===');
const complexIterator = collection.createFilterIterator(user => 
  user.role === 'developer' && user.age > 26
);
while (complexIterator.hasNext()) {
  console.log(complexIterator.next());
}
```

## Упражнение 4: Посредник (Mediator)

### Задача
Создайте систему чат-комнаты с использованием паттерна посредник для управления взаимодействием между пользователями.

### Требования
1. Посредник для управления сообщениями между пользователями
2. Пользователи, которые могут отправлять и получать сообщения
3. Различные типы сообщений (личные, групповые, системные)
4. Возможность добавления/удаления пользователей из чата

### Решение
```javascript
// Интерфейс посредника
class Mediator {
  notify(sender, event, data) {
    throw new Error('Метод notify должен быть реализован');
  }
}

// Конкретный посредник - чат-комната
class ChatRoom extends Mediator {
  constructor() {
    super();
    this.users = new Map(); // id -> пользователь
    this.messages = []; // история сообщений
  }
  
  // Регистрация пользователя
  registerUser(user) {
    this.users.set(user.id, user);
    user.setMediator(this);
    console.log(`Пользователь ${user.name} присоединился к чату`);
    
    // Отправляем системное сообщение
    this.broadcastSystemMessage(`${user.name} присоединился к чату`);
  }
  
  // Удаление пользователя
  unregisterUser(user) {
    if (this.users.has(user.id)) {
      this.users.delete(user.id);
      console.log(`Пользователь ${user.name} покинул чат`);
      
      // Отправляем системное сообщение
      this.broadcastSystemMessage(`${user.name} покинул чат`);
    }
  }
  
  // Отправка личного сообщения
  sendPrivateMessage(from, toId, message) {
    const toUser = this.users.get(toId);
    if (toUser) {
      const messageData = {
        type: 'private',
        from: from.name,
        to: toUser.name,
        message: message,
        timestamp: new Date()
      };
      
      this.messages.push(messageData);
      toUser.receiveMessage(messageData);
    } else {
      console.log(`Ошибка: Пользователь с ID ${toId} не найден`);
    }
  }
  
  // Отправка группового сообщения
  broadcastMessage(from, message) {
    const messageData = {
      type: 'broadcast',
      from: from.name,
      message: message,
      timestamp: new Date()
    };
    
    this.messages.push(messageData);
    
    // Отправляем сообщение всем пользователям, кроме отправителя
    this.users.forEach(user => {
      if (user.id !== from.id) {
        user.receiveMessage(messageData);
      }
    });
  }
  
  // Отправка системного сообщения
  broadcastSystemMessage(message) {
    const messageData = {
      type: 'system',
      message: message,
      timestamp: new Date()
    };
    
    this.messages.push(messageData);
    
    // Отправляем системное сообщение всем пользователям
    this.users.forEach(user => {
      user.receiveMessage(messageData);
    });
  }
  
  // Получение списка пользователей
  getUserList() {
    return Array.from(this.users.values()).map(user => ({
      id: user.id,
      name: user.name,
      online: user.online
    }));
  }
  
  // Получение истории сообщений
  getMessageHistory(limit = 50) {
    return this.messages.slice(-limit);
  }
  
  // Поиск сообщений по ключевому слову
  searchMessages(keyword) {
    return this.messages.filter(message => 
      message.message && message.message.toLowerCase().includes(keyword.toLowerCase())
    );
  }
  
  // Реализация метода notify
  notify(sender, event, data) {
    switch (event) {
      case 'send_private':
        this.sendPrivateMessage(sender, data.toId, data.message);
        break;
      case 'send_broadcast':
        this.broadcastMessage(sender, data.message);
        break;
      case 'join':
        this.registerUser(sender);
        break;
      case 'leave':
        this.unregisterUser(sender);
        break;
      default:
        console.log(`Неизвестное событие: ${event}`);
    }
  }
}

// Коллега - пользователь чата
class User {
  constructor(id, name) {
    this.id = id;
    this.name = name;
    this.online = true;
    this.mediator = null;
  }
  
  setMediator(mediator) {
    this.mediator = mediator;
  }
  
  // Присоединение к чату
  joinChat(chatRoom) {
    this.mediator = chatRoom;
    this.mediator.notify(this, 'join');
  }
  
  // Покидание чата
  leaveChat() {
    if (this.mediator) {
      this.mediator.notify(this, 'leave');
      this.mediator = null;
    }
  }
  
  // Отправка личного сообщения
  sendMessageTo(toId, message) {
    if (this.mediator) {
      this.mediator.notify(this, 'send_private', { toId, message });
    }
  }
  
  // Отправка группового сообщения
  sendBroadcastMessage(message) {
    if (this.mediator) {
      this.mediator.notify(this, 'send_broadcast', { message });
    }
  }
  
  // Получение сообщения
  receiveMessage(messageData) {
    const time = messageData.timestamp.toLocaleTimeString();
    
    switch (messageData.type) {
      case 'private':
        console.log(`[${time}] [Личное] ${messageData.from} -> ${messageData.to}: ${messageData.message}`);
        break;
      case 'broadcast':
        console.log(`[${time}] [Групповое] ${messageData.from}: ${messageData.message}`);
        break;
      case 'system':
        console.log(`[${time}] [Система] ${messageData.message}`);
        break;
    }
  }
  
  // Установка статуса онлайн/оффлайн
  setOnline(online) {
    this.online = online;
    console.log(`${this.name} теперь ${online ? 'онлайн' : 'оффлайн'}`);
  }
}

// Расширенный пользователь с дополнительными возможностями
class AdvancedUser extends User {
  constructor(id, name, role = 'user') {
    super(id, name);
    this.role = role;
    this.blockedUsers = new Set();
  }
  
  // Блокировка пользователя
  blockUser(userId) {
    this.blockedUsers.add(userId);
    console.log(`${this.name} заблокировал пользователя ${userId}`);
  }
  
  // Разблокировка пользователя
  unblockUser(userId) {
    this.blockedUsers.delete(userId);
    console.log(`${this.name} разблокировал пользователя ${userId}`);
  }
  
  // Переопределение получения сообщений с проверкой блокировки
  receiveMessage(messageData) {
    // Проверяем, не заблокирован ли отправитель
    if (messageData.from && this.blockedUsers.has(messageData.from)) {
      console.log(`Сообщение от ${messageData.from} заблокировано`);
      return;
    }
    
    super.receiveMessage(messageData);
  }
  
  // Отправка сообщения с проверкой прав
  sendBroadcastMessage(message) {
    if (this.role === 'admin' || this.role === 'moderator') {
      super.sendBroadcastMessage(message);
    } else {
      console.log('У вас нет прав для отправки групповых сообщений');
    }
  }
}

// Пример использования
const chatRoom = new ChatRoom();

// Создаем пользователей
const alice = new User(1, 'Alice');
const bob = new User(2, 'Bob');
const charlie = new AdvancedUser(3, 'Charlie', 'admin');
const david = new User(4, 'David');

// Пользователи присоединяются к чату
alice.joinChat(chatRoom);
bob.joinChat(chatRoom);
charlie.joinChat(chatRoom);
david.joinChat(chatRoom);

console.log('\n=== Отправка сообщений ===');

// Отправка личных сообщений
alice.sendMessageTo(2, 'Привет, Bob!');
bob.sendMessageTo(1, 'Привет, Alice! Как дела?');
charlie.sendBroadcastMessage('Добро пожаловать в чат!');

console.log('\n=== Групповые сообщения ===');

// Отправка групповых сообщений
david.sendBroadcastMessage('Всем привет!'); // Обычный пользователь не может

// Администратор может отправлять групповые сообщения
charlie.sendBroadcastMessage('Объявление: встреча в 15:00');

console.log('\n=== Список пользователей ===');
console.log(chatRoom.getUserList());

console.log('\n=== История сообщений ===');
console.log(chatRoom.getMessageHistory(5));

console.log('\n=== Расширенные возможности ===');

// Блокировка пользователя
charlie.blockUser('David');
david.sendMessageTo(1, 'Это сообщение не дойдет'); // Будет заблокировано

// Поиск сообщений
console.log('\n=== Поиск сообщений ===');
console.log(chatRoom.searchMessages('привет'));

// Пользователь покидает чат
console.log('\n=== Покидание чата ===');
bob.leaveChat();

// Отправка сообщения после ухода пользователя
alice.sendBroadcastMessage('Bob уже ушел :(');
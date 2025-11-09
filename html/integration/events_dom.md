# HTML Интеграция: События и DOM

HTML, CSS и JavaScript тесно связаны через DOM (Document Object Model) и систему событий. Понимание этих механизмов критически важно для создания интерактивных веб-страниц.

## DOM (Document Object Model)

DOM - это программный интерфейс для HTML и XML документов. Он представляет документ как древовидную структуру, где каждый узел является элементом документа.

### Структура DOM

```html
<!DOCTYPE html>
<html>
<head>
    <title>Пример DOM</title>
</head>
<body>
    <div id="container">
        <h1>Заголовок</h1>
        <p class="content">Текст абзаца</p>
        <ul>
            <li>Элемент 1</li>
            <li>Элемент 2</li>
        </ul>
    </div>
</body>
</html>
```

```javascript
// Доступ к элементам DOM
const container = document.getElementById('container');
const heading = document.querySelector('h1');
const content = document.querySelector('.content');
const listItems = document.querySelectorAll('li');

// Навигация по DOM
console.log(container.firstChild);        // Первый дочерний узел
console.log(container.lastChild);         // Последний дочерний узел
console.log(heading.parentNode);          // Родительский элемент
console.log(content.nextSibling);         // Следующий соседний элемент
console.log(content.previousSibling);     // Предыдущий соседний элемент
```

### Основные методы DOM

#### Поиск элементов

```javascript
// Поиск по ID
const element = document.getElementById('myId');

// Поиск по тегу
const paragraphs = document.getElementsByTagName('p');

// Поиск по классу
const elements = document.getElementsByClassName('myClass');

// Современные методы поиска
const element1 = document.querySelector('#myId');           // Первый элемент
const elements1 = document.querySelectorAll('.myClass');    // Все элементы

// Примеры сложных селекторов
const specificElement = document.querySelector('div.container > p:first-child');
const allInputs = document.querySelectorAll('input[type="text"]');
```

#### Создание и изменение элементов

```javascript
// Создание нового элемента
const newDiv = document.createElement('div');
newDiv.id = 'new-element';
newDiv.className = 'new-class';
newDiv.textContent = 'Новый элемент';

// Создание элемента с HTML содержимым
const newParagraph = document.createElement('p');
newParagraph.innerHTML = 'Параграф с <strong>HTML</strong> содержимым';

// Добавление элемента в DOM
const container = document.getElementById('container');
container.appendChild(newDiv);
container.appendChild(newParagraph);

// Изменение существующих элементов
const heading = document.querySelector('h1');
heading.textContent = 'Новый заголовок';
heading.style.color = 'blue';
heading.setAttribute('data-updated', 'true');

// Удаление элементов
const elementToRemove = document.getElementById('old-element');
if (elementToRemove) {
    elementToRemove.remove();
}
```

## Система событий

### Типы событий

```javascript
// События мыши
element.addEventListener('click', handleClick);
element.addEventListener('dblclick', handleDoubleClick);
element.addEventListener('mouseenter', handleMouseEnter);
element.addEventListener('mouseleave', handleMouseLeave);
element.addEventListener('mousedown', handleMouseDown);
element.addEventListener('mouseup', handleMouseUp);

// События клавиатуры
document.addEventListener('keydown', handleKeyDown);
document.addEventListener('keyup', handleKeyUp);
document.addEventListener('keypress', handleKeyPress);

// События формы
form.addEventListener('submit', handleSubmit);
input.addEventListener('input', handleInput);
input.addEventListener('change', handleChange);
input.addEventListener('focus', handleFocus);
input.addEventListener('blur', handleBlur);

// События документа
document.addEventListener('DOMContentLoaded', handleDOMLoad);
window.addEventListener('load', handleWindowLoad);
window.addEventListener('resize', handleResize);
window.addEventListener('scroll', handleScroll);
```

### Обработка событий

```javascript
// Старый способ (не рекомендуется)
element.onclick = function() {
    console.log('Клик!');
};

// Современный способ с addEventListener
element.addEventListener('click', function(event) {
    console.log('Элемент кликнут:', event.target);
    console.log('Координаты клика:', event.clientX, event.clientY);
    
    // Предотвращение стандартного поведения
    event.preventDefault();
    
    // Остановка всплытия события
    event.stopPropagation();
});

// Стрелочная функция
element.addEventListener('click', (event) => {
    console.log('Стрелочная функция:', event.target);
});

// Объект с методом handleEvent
const handler = {
    handleEvent(event) {
        console.log('Обработчик события:', event.type);
    }
};
element.addEventListener('click', handler);
```

### Всплытие и захват событий

```html
<!DOCTYPE html>
<html>
<body>
    <div id="outer">
        <div id="middle">
            <button id="inner">Кнопка</button>
        </div>
    </div>
    
    <script>
        const outer = document.getElementById('outer');
        const middle = document.getElementById('middle');
        const inner = document.getElementById('inner');
        
        // Фаза захвата (capture phase)
        outer.addEventListener('click', () => console.log('Захват: outer'), true);
        middle.addEventListener('click', () => console.log('Захват: middle'), true);
        inner.addEventListener('click', () => console.log('Захват: inner'), true);
        
        // Фаза всплытия (bubbling phase) - по умолчанию
        outer.addEventListener('click', () => console.log('Всплытие: outer'));
        middle.addEventListener('click', () => console.log('Всплытие: middle'));
        inner.addEventListener('click', () => console.log('Всплытие: inner'));
        
        // При клике на кнопку:
        // 1. Захват: outer
        // 2. Захват: middle
        // 3. Захват: inner
        // 4. Всплытие: inner
        // 5. Всплытие: middle
        // 6. Всплытие: outer
    </script>
</body>
</html>
```

## Практические примеры интеграции

### Динамическая форма

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Динамическая форма</title>
    <style>
        .form-container {
            max-width: 500px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select, button {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
        }
        
        .dynamic-field {
            display: flex;
            gap: 10px;
            margin-top: 10px;
        }
        
        .dynamic-field input {
            flex: 1;
        }
        
        .remove-btn {
            width: auto;
            padding: 8px 12px;
            background-color: #dc3545;
            color: white;
            border: none;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <h2>Динамическая форма</h2>
        
        <form id="dynamicForm">
            <div class="form-group">
                <label for="name">Имя:</label>
                <input type="text" id="name" name="name" required>
            </div>
            
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
            </div>
            
            <div class="form-group">
                <label>Интересы:</label>
                <div id="interests-container">
                    <div class="dynamic-field">
                        <input type="text" name="interests" placeholder="Интерес">
                        <button type="button" class="remove-btn" onclick="removeInterest(this)">Удалить</button>
                    </div>
                </div>
                <button type="button" id="add-interest">Добавить интерес</button>
            </div>
            
            <button type="submit">Отправить</button>
        </form>
    </div>
    
    <script>
        document.getElementById('add-interest').addEventListener('click', addInterestField);
        document.getElementById('dynamicForm').addEventListener('submit', handleSubmit);
        
        function addInterestField() {
            const container = document.getElementById('interests-container');
            const newField = document.createElement('div');
            newField.className = 'dynamic-field';
            newField.innerHTML = `
                <input type="text" name="interests" placeholder="Интерес">
                <button type="button" class="remove-btn" onclick="removeInterest(this)">Удалить</button>
            `;
            container.appendChild(newField);
        }
        
        function removeInterest(button) {
            const field = button.closest('.dynamic-field');
            if (document.querySelectorAll('.dynamic-field').length > 1) {
                field.remove();
            } else {
                alert('Должно быть хотя бы одно поле интересов');
            }
        }
        
        function handleSubmit(event) {
            event.preventDefault();
            
            const formData = new FormData(event.target);
            const data = {
                name: formData.get('name'),
                email: formData.get('email'),
                interests: Array.from(formData.getAll('interests')).filter(i => i.trim() !== '')
            };
            
            console.log('Данные формы:', data);
            alert('Форма отправлена! Проверьте консоль для деталей.');
        }
    </script>
</body>
</html>
```

### Интерактивный список задач

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивный список задач</title>
    <style>
        .todo-app {
            max-width: 500px;
            margin: 20px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-family: Arial, sans-serif;
        }
        
        .input-group {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        
        .input-group input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        
        .input-group button {
            padding: 10px 15px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .task-list {
            list-style: none;
            padding: 0;
        }
        
        .task-item {
            display: flex;
            align-items: center;
            padding: 10px;
            border: 1px solid #eee;
            border-radius: 4px;
            margin-bottom: 5px;
        }
        
        .task-item.completed {
            text-decoration: line-through;
            color: #888;
            background-color: #f5f5f5;
        }
        
        .task-text {
            flex: 1;
            margin: 0 10px;
        }
        
        .task-actions {
            display: flex;
            gap: 5px;
        }
        
        .task-actions button {
            padding: 5px 10px;
            border: 1px solid #ccc;
            border-radius: 3px;
            cursor: pointer;
            font-size: 12px;
        }
        
        .delete-btn {
            background-color: #dc3545;
            color: white;
        }
        
        .edit-btn {
            background-color: #ffc107;
            color: black;
        }
        
        .complete-btn {
            background-color: #28a745;
            color: white;
        }
    </style>
</head>
<body>
    <div class="todo-app">
        <h2>Список задач</h2>
        
        <div class="input-group">
            <input type="text" id="taskInput" placeholder="Введите новую задачу...">
            <button onclick="addTask()">Добавить</button>
        </div>
        
        <ul id="taskList" class="task-list">
            <!-- Задачи будут добавляться сюда -->
        </ul>
    </div>
    
    <script>
        let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
        const taskInput = document.getElementById('taskInput');
        const taskList = document.getElementById('taskList');
        
        // Загрузка задач при загрузке страницы
        window.addEventListener('DOMContentLoaded', renderTasks);
        
        // Добавление задачи по нажатию Enter
        taskInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                addTask();
            }
        });
        
        function addTask() {
            const text = taskInput.value.trim();
            if (text) {
                const task = {
                    id: Date.now(),
                    text: text,
                    completed: false,
                    createdAt: new Date().toISOString()
                };
                
                tasks.push(task);
                saveTasks();
                renderTasks();
                taskInput.value = '';
                taskInput.focus();
            }
        }
        
        function deleteTask(id) {
            tasks = tasks.filter(task => task.id !== id);
            saveTasks();
            renderTasks();
        }
        
        function toggleComplete(id) {
            tasks = tasks.map(task => {
                if (task.id === id) {
                    return { ...task, completed: !task.completed };
                }
                return task;
            });
            saveTasks();
            renderTasks();
        }
        
        function editTask(id) {
            const task = tasks.find(task => task.id === id);
            if (task) {
                const newText = prompt('Изменить задачу:', task.text);
                if (newText !== null && newText.trim() !== '') {
                    task.text = newText.trim();
                    saveTasks();
                    renderTasks();
                }
            }
        }
        
        function saveTasks() {
            localStorage.setItem('tasks', JSON.stringify(tasks));
        }
        
        function renderTasks() {
            taskList.innerHTML = '';
            
            if (tasks.length === 0) {
                taskList.innerHTML = '<p>Нет задач. Добавьте первую задачу!</p>';
                return;
            }
            
            tasks.forEach(task => {
                const li = document.createElement('li');
                li.className = `task-item ${task.completed ? 'completed' : ''}`;
                li.innerHTML = `
                    <input type="checkbox" ${task.completed ? 'checked' : ''} 
                           onchange="toggleComplete(${task.id})">
                    <span class="task-text">${task.text}</span>
                    <div class="task-actions">
                        <button class="edit-btn" onclick="editTask(${task.id})">Редактировать</button>
                        <button class="complete-btn" onclick="toggleComplete(${task.id})">
                            ${task.completed ? 'Отменить' : 'Выполнено'}
                        </button>
                        <button class="delete-btn" onclick="deleteTask(${task.id})">Удалить</button>
                    </div>
                `;
                taskList.appendChild(li);
            });
        }
    </script>
</body>
</html>
```

## Современные паттерны интеграции

### Делегирование событий

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Делегирование событий</title>
    <style>
        .container {
            max-width: 600px;
            margin: 20px auto;
            padding: 20px;
        }
        
        .button-group {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }
        
        .button-group button {
            padding: 10px 15px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .item-list {
            list-style: none;
            padding: 0;
        }
        
        .list-item {
            padding: 10px;
            border: 1px solid #eee;
            margin-bottom: 5px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .item-actions {
            display: flex;
            gap: 5px;
        }
        
        .item-actions button {
            padding: 5px 10px;
            border: 1px solid #ccc;
            border-radius: 3px;
            cursor: pointer;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h2>Делегирование событий</h2>
        
        <div class="button-group">
            <button data-action="add">Добавить элемент</button>
            <button data-action="clear">Очистить список</button>
        </div>
        
        <ul id="itemList" class="item-list">
            <li class="list-item" data-id="1">
                Элемент 1
                <div class="item-actions">
                    <button class="edit-btn" data-action="edit">Редактировать</button>
                    <button class="delete-btn" data-action="delete">Удалить</button>
                </div>
            </li>
        </ul>
    </div>
    
    <script>
        const itemList = document.getElementById('itemList');
        let nextId = 2;
        
        // Делегирование событий для всего контейнера
        itemList.addEventListener('click', function(event) {
            const target = event.target;
            const listItem = target.closest('.list-item');
            
            if (!listItem) return;
            
            const id = listItem.dataset.id;
            const action = target.dataset.action;
            
            switch(action) {
                case 'edit':
                    editItem(id);
                    break;
                case 'delete':
                    deleteItem(id);
                    break;
            }
        });
        
        // Делегирование для кнопок вне списка
        document.addEventListener('click', function(event) {
            const action = event.target.dataset.action;
            
            switch(action) {
                case 'add':
                    addItem();
                    break;
                case 'clear':
                    clearList();
                    break;
            }
        });
        
        function addItem() {
            const li = document.createElement('li');
            li.className = 'list-item';
            li.dataset.id = nextId;
            li.innerHTML = `
                Новый элемент ${nextId}
                <div class="item-actions">
                    <button class="edit-btn" data-action="edit">Редактировать</button>
                    <button class="delete-btn" data-action="delete">Удалить</button>
                </div>
            `;
            itemList.appendChild(li);
            nextId++;
        }
        
        function deleteItem(id) {
            const item = document.querySelector(`[data-id="${id}"]`);
            if (item) {
                item.remove();
            }
        }
        
        function editItem(id) {
            const item = document.querySelector(`[data-id="${id}"]`);
            if (item) {
                const currentText = item.childNodes[0].nodeValue.trim();
                const newText = prompt('Изменить элемент:', currentText);
                if (newText) {
                    item.childNodes[0].nodeValue = newText;
                }
            }
        }
        
        function clearList() {
            if (confirm('Очистить весь список?')) {
                itemList.innerHTML = '<li class="list-item">Список пуст</li>';
                nextId = 1;
            }
        }
    </script>
</body>
</html>
```

### Паттерн "Observer" с DOM

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Observer Pattern с DOM</title>
    <style>
        .app-container {
            max-width: 600px;
            margin: 20px auto;
            padding: 20px;
            font-family: Arial, sans-serif;
        }
        
        .status-display {
            padding: 10px;
            margin: 10px 0;
            border: 1px solid #ddd;
            border-radius: 4px;
            background-color: #f9f9f9;
        }
        
        .controls {
            margin: 20px 0;
        }
        
        .controls button {
            margin-right: 10px;
            padding: 8px 12px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .log {
            margin-top: 20px;
            padding: 10px;
            background-color: #f0f0f0;
            border-radius: 4px;
            max-height: 200px;
            overflow-y: auto;
        }
        
        .log-entry {
            margin: 5px 0;
            padding: 2px 0;
            border-bottom: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <div class="app-container">
        <h2>Observer Pattern с DOM</h2>
        
        <div class="status-display">
            <strong>Состояние:</strong> 
            <span id="status">Инициализация</span>
        </div>
        
        <div class="controls">
            <button onclick="appController.start()">Начать</button>
            <button onclick="appController.pause()">Пауза</button>
            <button onclick="appController.stop()">Остановить</button>
            <button onclick="appController.reset()">Сброс</button>
        </div>
        
        <div class="log" id="log">
            <div class="log-entry">Приложение запущено</div>
        </div>
    </div>
    
    <script>
        // Observer Pattern Implementation
        class Observer {
            constructor() {
                this.observers = [];
            }
            
            subscribe(fn) {
                this.observers.push(fn);
            }
            
            unsubscribe(fn) {
                this.observers = this.observers.filter(observer => observer !== fn);
            }
            
            broadcast(data) {
                this.observers.forEach(observer => observer(data));
            }
        }
        
        // Application State Manager
        class AppState {
            constructor() {
                this.state = 'idle';
                this.observer = new Observer();
            }
            
            setState(newState) {
                const oldState = this.state;
                this.state = newState;
                
                // Уведомляем всех подписчиков об изменении состояния
                this.observer.broadcast({
                    type: 'stateChange',
                    oldState,
                    newState,
                    timestamp: new Date()
                });
            }
            
            getState() {
                return this.state;
            }
            
            subscribe(callback) {
                this.observer.subscribe(callback);
            }
        }
        
        // DOM Renderer
        class DOMRenderer {
            constructor(appState) {
                this.appState = appState;
                this.statusElement = document.getElementById('status');
                this.logElement = document.getElementById('log');
                
                // Подписываемся на изменения состояния
                this.appState.subscribe(this.handleStateChange.bind(this));
            }
            
            handleStateChange(data) {
                // Обновляем статус
                this.statusElement.textContent = data.newState;
                
                // Добавляем запись в лог
                const logEntry = document.createElement('div');
                logEntry.className = 'log-entry';
                logEntry.textContent = `[${data.timestamp.toLocaleTimeString()}] Состояние: ${data.oldState} → ${data.newState}`;
                this.logElement.appendChild(logEntry);
                
                // Прокручиваем лог вниз
                this.logElement.scrollTop = this.logElement.scrollHeight;
            }
        }
        
        // Application Controller
        class AppController {
            constructor(appState) {
                this.appState = appState;
            }
            
            start() {
                if (['idle', 'paused', 'stopped'].includes(this.appState.getState())) {
                    this.appState.setState('running');
                }
            }
            
            pause() {
                if (this.appState.getState() === 'running') {
                    this.appState.setState('paused');
                }
            }
            
            stop() {
                if (['running', 'paused'].includes(this.appState.getState())) {
                    this.appState.setState('stopped');
                }
            }
            
            reset() {
                this.appState.setState('idle');
            }
        }
        
        // Инициализация приложения
        const appState = new AppState();
        const domRenderer = new DOMRenderer(appState);
        const appController = new AppController(appState);
        
        // Глобальный доступ к контроллеру
        window.appController = appController;
    </script>
</body>
</html>
```

## Заключение

Интеграция HTML, CSS и JavaScript через DOM и систему событий - это основа интерактивных веб-приложений. Ключевые аспекты:

1. **DOM манипуляции** - изменение структуры и содержимого страницы
2. **Событийная модель** - обработка пользовательских действий
3. **Делегирование событий** - эффективная обработка событий для динамического контента
4. **Архитектурные паттерны** - организация кода для масштабируемости
5. **Производительность** - оптимизация взаимодействия с DOM

Понимание этих концепций позволяет создавать сложные, интерактивные веб-приложения с чистым, поддерживаемым кодом.

## Следующие темы
- [[HTML/Интеграция с CSS/Адаптивная верстка]]
- [[HTML/Продвинутые темы/Шаблоны и веб-компоненты]]
- [[HTML/Производительность]]

## Теги
#html #dom #events #javascript #web-development #frontend #interactivity
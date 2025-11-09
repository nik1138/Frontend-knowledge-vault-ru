# HTML Интеграция: Взаимодействие с JavaScript

Интеграция HTML с JavaScript является ключевым аспектом современной веб-разработки. HTML предоставляет структуру, CSS - стилизацию, а JavaScript - интерактивность и динамическое поведение. Правильная интеграция этих технологий позволяет создавать мощные и интерактивные веб-приложения.

## Основы интеграции HTML и JavaScript

### Варианты подключения JavaScript

#### 1. Встроенный JavaScript (inline)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Встроенный JavaScript</title>
</head>
<body>
    <h1>Пример встроенного JavaScript</h1>

    <button onclick="alert('Привет!')">Кликни меня</button>

    <div id="dynamic-content">Исходное содержимое</div>

    <script>
        // Встроенный скрипт
        document.addEventListener('DOMContentLoaded', function() {
            console.log('DOM полностью загружен');

            // Динамическое изменение содержимого
            document.getElementById('dynamic-content').innerHTML =
                '<p>Это динамически измененное содержимое</p>';
        });
    </script>
</body>
</html>
```

#### 2. Внутренний JavaScript (в теге script)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Внутренний JavaScript</title>
</head>
<body>
    <h1>Внутренний JavaScript</h1>

    <div id="counter-container">
        <p>Счетчик: <span id="counter">0</span></p>
        <button id="increment-btn">Увеличить</button>
        <button id="decrement-btn">Уменьшить</button>
    </div>

    <script>
        // Внутренний скрипт
        class Counter {
            constructor(elementId) {
                this.element = document.getElementById(elementId);
                this.count = 0;
                this.init();
            }

            init() {
                this.incrementBtn = document.getElementById('increment-btn');
                this.decrementBtn = document.getElementById('decrement-btn');

                this.incrementBtn.addEventListener('click', () => this.increment());
                this.decrementBtn.addEventListener('click', () => this.decrement());

                this.updateDisplay();
            }

            increment() {
                this.count++;
                this.updateDisplay();
            }

            decrement() {
                this.count--;
                this.updateDisplay();
            }

            updateDisplay() {
                this.element.textContent = this.count;
            }
        }

        // Инициализация счетчика
        const counter = new Counter('counter');
    </script>
</body>
</html>
```

#### 3. Внешние JavaScript файлы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Внешние JavaScript файлы</title>
</head>
<body>
    <h1>Внешние JavaScript файлы</h1>

    <div id="app">
        <h2>Заголовок приложения</h2>
        <div id="content">Содержимое будет загружено из JavaScript</div>
    </div>

    <!-- Подключение внешнего скрипта -->
    <script src="app.js"></script>

    <!-- Скрипт с атрибутами -->
    <script src="utils.js" defer></script>
    <script src="components.js" async></script>

    <!-- Модульный скрипт -->
    <script type="module" src="modules/main.js"></script>
</body>
</html>
```

### Атрибуты script тега

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Атрибуты script тега</title>
</head>
<body>
    <h1>Атрибуты скрипта</h1>

    <!-- defer - скрипт выполнится после парсинга HTML -->
    <script src="defer-script.js" defer></script>

    <!-- async - скрипт загрузится асинхронно -->
    <script src="async-script.js" async></script>

    <!-- module - скрипт как ES модуль -->
    <script type="module" src="module-script.js"></script>

    <!-- nomodule - для браузеров без поддержки модулей -->
    <script nomodule src="legacy-script.js"></script>

    <!-- crossorigin - для CORS запросов -->
    <script src="external-script.js" crossorigin="anonymous"></script>

    <!-- integrity - для проверки целостности -->
    <script src="https://cdn.example.com/script.js"
            integrity="sha384-..."
            crossorigin="anonymous"></script>

    <div id="content">Содержимое</div>

    <script>
        // Inline скрипт для демонстрации порядка выполнения
        console.log('Inline script executed');
    </script>
</body>
</html>
```

## Взаимодействие с DOM

### Выбор элементов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Выбор элементов DOM</title>
</head>
<body>
    <h1>Выбор элементов DOM</h1>

    <div class="container">
        <h2 id="main-title">Основной заголовок</h2>
        <p class="description">Описание</p>
        <ul class="list">
            <li data-id="1" class="item">Элемент 1</li>
            <li data-id="2" class="item">Элемент 2</li>
            <li data-id="3" class="item">Элемент 3</li>
        </ul>
        <form id="user-form">
            <input type="text" name="username" class="input-field">
            <input type="email" name="email" class="input-field">
            <button type="submit">Отправить</button>
        </form>
    </div>

    <script>
        // Выбор элементов различными способами

        // По ID
        const mainTitle = document.getElementById('main-title');
        console.log('Заголовок:', mainTitle.textContent);

        // По классу
        const description = document.getElementsByClassName('description')[0];
        console.log('Описание:', description.textContent);

        // По тегу
        const listItems = document.getElementsByTagName('li');
        console.log('Количество элементов списка:', listItems.length);

        // Query Selector (CSS селекторы)
        const container = document.querySelector('.container');
        const firstItem = document.querySelector('.item');
        const allItems = document.querySelectorAll('.item');

        console.log('Контейнер:', container);
        console.log('Первый элемент:', firstItem.textContent);
        console.log('Все элементы:', allItems.length);

        // Выбор по атрибутам
        const elementsWithData = document.querySelectorAll('[data-id]');
        console.log('Элементов с data-id:', elementsWithData.length);

        // Выбор по составным селекторам
        const inputFields = document.querySelectorAll('form input.input-field');
        console.log('Поля ввода:', inputFields.length);

        // Выбор по псевдоклассам
        const evenItems = document.querySelectorAll('li:nth-child(even)');
        console.log('Четные элементы:', evenItems.length);
    </script>
</body>
</html>
```

### Манипуляции с DOM

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Манипуляции с DOM</title>
    <style>
        .highlight { background-color: yellow; }
        .fade-in { opacity: 0; animation: fadeIn 1s forwards; }
        @keyframes fadeIn { to { opacity: 1; } }
    </style>
</head>
<body>
    <h1>Манипуляции с DOM</h1>

    <div id="dynamic-container">
        <p>Исходное содержимое</p>
    </div>

    <button id="add-content">Добавить контент</button>
    <button id="modify-content">Изменить контент</button>
    <button id="remove-content">Удалить контент</button>
    <button id="toggle-highlight">Переключить выделение</button>

    <script>
        class DOMManipulator {
            constructor() {
                this.container = document.getElementById('dynamic-container');
                this.setupEventListeners();
            }

            setupEventListeners() {
                document.getElementById('add-content').addEventListener('click', () => this.addContent());
                document.getElementById('modify-content').addEventListener('click', () => this.modifyContent());
                document.getElementById('remove-content').addEventListener('click', () => this.removeContent());
                document.getElementById('toggle-highlight').addEventListener('click', () => this.toggleHighlight());
            }

            addContent() {
                const newParagraph = document.createElement('p');
                newParagraph.textContent = `Новый параграф добавлен в ${new Date().toLocaleTimeString()}`;
                newParagraph.classList.add('fade-in');

                this.container.appendChild(newParagraph);
            }

            modifyContent() {
                const paragraphs = this.container.querySelectorAll('p');
                paragraphs.forEach((p, index) => {
                    p.textContent = `Измененный параграф ${index + 1} в ${new Date().toLocaleTimeString()}`;
                    p.style.color = '#007acc';
                });
            }

            removeContent() {
                const paragraphs = this.container.querySelectorAll('p');
                if (paragraphs.length > 1) {
                    paragraphs[paragraphs.length - 1].remove();
                } else {
                    this.container.innerHTML = '<p>Исходный контент</p>';
                }
            }

            toggleHighlight() {
                const paragraphs = this.container.querySelectorAll('p');
                paragraphs.forEach(p => p.classList.toggle('highlight'));
            }
        }

        // Инициализация
        const manipulator = new DOMManipulator();
    </script>
</body>
</html>
```

## Обработка событий

### Типы событий

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Обработка событий</title>
    <style>
        .event-demo {
            margin: 20px 0;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        .clickable {
            padding: 10px;
            background-color: #f0f0f0;
            border: 1px solid #ccc;
            cursor: pointer;
            margin: 5px 0;
        }

        .clickable:hover {
            background-color: #e0e0e0;
        }

        .clickable.active {
            background-color: #007acc;
            color: white;
        }

        .input-field {
            width: 100%;
            padding: 8px;
            margin: 5px 0;
            border: 1px solid #ddd;
        }

        .log {
            background-color: #f5f5f5;
            padding: 10px;
            margin: 10px 0;
            border-radius: 4px;
            font-family: monospace;
            font-size: 0.9em;
            max-height: 200px;
            overflow-y: auto;
        }
    </style>
</head>
<body>
    <h1>Обработка событий</h1>

    <div class="event-demo">
        <h3>События мыши</h3>
        <div class="clickable" id="mouse-demo">Кликни меня</div>
        <div class="clickable" id="hover-demo">Наведи на меня</div>
    </div>

    <div class="event-demo">
        <h3>События клавиатуры</h3>
        <input type="text" class="input-field" id="keyboard-demo" placeholder="Начни печатать...">
    </div>

    <div class="event-demo">
        <h3>События формы</h3>
        <form id="form-demo">
            <input type="text" name="name" class="input-field" placeholder="Имя">
            <input type="email" name="email" class="input-field" placeholder="Email">
            <button type="submit">Отправить</button>
        </form>
    </div>

    <div class="event-demo">
        <h3>События фокуса</h3>
        <input type="text" class="input-field" id="focus-demo" placeholder="Поле с фокусом">
    </div>

    <div class="log" id="event-log">
        <h4>Лог событий:</h4>
        <div id="log-content"></div>
    </div>

    <script>
        class EventManager {
            constructor() {
                this.logElement = document.getElementById('log-content');
                this.setupEventListeners();
            }

            setupEventListeners() {
                // События мыши
                document.getElementById('mouse-demo').addEventListener('click', (e) => {
                    this.logEvent('click', e);
                });

                document.getElementById('hover-demo').addEventListener('mouseenter', (e) => {
                    this.logEvent('mouseenter', e);
                });

                document.getElementById('hover-demo').addEventListener('mouseleave', (e) => {
                    this.logEvent('mouseleave', e);
                });

                // События клавиатуры
                document.getElementById('keyboard-demo').addEventListener('keydown', (e) => {
                    this.logEvent('keydown', e);
                });

                document.getElementById('keyboard-demo').addEventListener('keyup', (e) => {
                    this.logEvent('keyup', e);
                });

                document.getElementById('keyboard-demo').addEventListener('input', (e) => {
                    this.logEvent('input', e);
                });

                // События формы
                document.getElementById('form-demo').addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.logEvent('submit', e);
                });

                // События фокуса
                document.getElementById('focus-demo').addEventListener('focus', (e) => {
                    e.target.classList.add('active');
                    this.logEvent('focus', e);
                });

                document.getElementById('focus-demo').addEventListener('blur', (e) => {
                    e.target.classList.remove('active');
                    this.logEvent('blur', e);
                });

                // События загрузки
                window.addEventListener('load', (e) => {
                    this.logEvent('load', e);
                });

                window.addEventListener('resize', (e) => {
                    this.logEvent('resize', e);
                });

                window.addEventListener('scroll', (e) => {
                    this.logEvent('scroll', e);
                });
            }

            logEvent(eventName, event) {
                const timestamp = new Date().toLocaleTimeString();
                const logEntry = document.createElement('div');
                logEntry.textContent = `[${timestamp}] ${eventName}: ${this.getEventDetails(event)}`;
                logEntry.style.padding = '2px 0';

                this.logElement.appendChild(logEntry);
                this.logElement.scrollTop = this.logElement.scrollHeight;
            }

            getEventDetails(event) {
                switch(event.type) {
                    case 'click':
                        return `Координаты: (${event.clientX}, ${event.clientY})`;
                    case 'keydown':
                        return `Клавиша: ${event.key}, Код: ${event.code}`;
                    case 'input':
                        return `Значение: ${event.target.value}`;
                    case 'submit':
                        return 'Форма отправлена';
                    case 'resize':
                        return `Размеры: ${window.innerWidth}x${window.innerHeight}`;
                    case 'scroll':
                        return `Позиция: ${window.scrollY}`;
                    default:
                        return `Цель: ${event.target.tagName}`;
                }
            }
        }

        // Инициализация
        const eventManager = new EventManager();
    </script>
</body>
</html>
```

### Делегирование событий

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Делегирование событий</title>
    <style>
        .list-container {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            margin: 20px 0;
        }

        .list-item {
            padding: 10px;
            border: 1px solid #eee;
            margin: 5px 0;
            border-radius: 4px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .item-actions {
            display: flex;
            gap: 5px;
        }

        .btn {
            padding: 5px 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.9em;
        }

        .btn-edit { background-color: #e3f2fd; color: #1976d2; }
        .btn-delete { background-color: #ffebee; color: #d32f2f; }
    </style>
</head>
<body>
    <h1>Делегирование событий</h1>

    <div class="list-container">
        <h3>Список элементов</h3>
        <ul id="item-list">
            <li class="list-item" data-id="1">
                <span class="item-text">Элемент 1</span>
                <div class="item-actions">
                    <button class="btn btn-edit" data-action="edit">Редактировать</button>
                    <button class="btn btn-delete" data-action="delete">Удалить</button>
                </div>
            </li>
            <li class="list-item" data-id="2">
                <span class="item-text">Элемент 2</span>
                <div class="item-actions">
                    <button class="btn btn-edit" data-action="edit">Редактировать</button>
                    <button class="btn btn-delete" data-action="delete">Удалить</button>
                </div>
            </li>
            <li class="list-item" data-id="3">
                <span class="item-text">Элемент 3</span>
                <div class="item-actions">
                    <button class="btn btn-edit" data-action="edit">Редактировать</button>
                    <button class="btn btn-delete" data-action="delete">Удалить</button>
                </div>
            </li>
        </ul>
    </div>

    <button id="add-item">Добавить элемент</button>

    <script>
        class EventDelegationExample {
            constructor() {
                this.list = document.getElementById('item-list');
                this.itemIdCounter = 4;
                this.setupEventDelegation();
                this.setupAddButton();
            }

            setupEventDelegation() {
                // Делегирование событий для всего списка
                this.list.addEventListener('click', (e) => {
                    // Определение типа действия
                    const actionButton = e.target.closest('[data-action]');
                    if (actionButton) {
                        const listItem = e.target.closest('.list-item');
                        const action = actionButton.dataset.action;
                        const itemId = listItem.dataset.id;

                        switch(action) {
                            case 'edit':
                                this.editItem(itemId);
                                break;
                            case 'delete':
                                this.deleteItem(itemId);
                                break;
                        }
                    }
                });

                // Делегирование событий для редактирования текста
                this.list.addEventListener('dblclick', (e) => {
                    if (e.target.classList.contains('item-text')) {
                        this.editText(e.target);
                    }
                });
            }

            setupAddButton() {
                document.getElementById('add-item').addEventListener('click', () => {
                    this.addItem();
                });
            }

            addItem() {
                const newItem = document.createElement('li');
                newItem.className = 'list-item';
                newItem.dataset.id = this.itemIdCounter;
                newItem.innerHTML = `
                    <span class="item-text">Новый элемент ${this.itemIdCounter}</span>
                    <div class="item-actions">
                        <button class="btn btn-edit" data-action="edit">Редактировать</button>
                        <button class="btn btn-delete" data-action="delete">Удалить</button>
                    </div>
                `;

                this.list.appendChild(newItem);
                this.itemIdCounter++;
            }

            editItem(id) {
                console.log(`Редактирование элемента с ID: ${id}`);
                // Здесь может быть логика редактирования
                const item = document.querySelector(`[data-id="${id}"] .item-text`);
                if (item) {
                    this.editText(item);
                }
            }

            deleteItem(id) {
                if (confirm(`Удалить элемент с ID: ${id}?`)) {
                    const item = document.querySelector(`[data-id="${id}"]`);
                    if (item) {
                        item.remove();
                    }
                }
            }

            editText(element) {
                const currentValue = element.textContent;
                const input = document.createElement('input');
                input.type = 'text';
                input.value = currentValue;
                input.style.width = '100%';

                element.replaceWith(input);
                input.focus();

                const saveEdit = () => {
                    const newText = document.createElement('span');
                    newText.className = 'item-text';
                    newText.textContent = input.value;

                    input.replaceWith(newText);
                };

                input.addEventListener('blur', saveEdit);
                input.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') {
                        e.preventDefault();
                        saveEdit();
                    }
                });
            }
        }

        // Инициализация
        const eventDelegation = new EventDelegationExample();
    </script>
</body>
</html>
```

## Асинхронные операции и HTML

### Fetch API и динамический контент

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Fetch API и динамический контент</title>
    <style>
        .loading {
            opacity: 0.6;
            pointer-events: none;
        }

        .spinner {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid rgba(0, 122, 204, 0.3);
            border-top-color: #007acc;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        .error-message {
            color: #d32f2f;
            background-color: #ffebee;
            padding: 10px;
            border-radius: 4px;
            margin: 10px 0;
        }

        .success-message {
            color: #2e7d32;
            background-color: #e8f5e9;
            padding: 10px;
            border-radius: 4px;
            margin: 10px 0;
        }

        .user-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            margin: 10px 0;
            background-color: white;
        }

        .user-avatar {
            width: 50px;
            height: 50px;
            border-radius: 50%;
            margin-right: 10px;
            vertical-align: middle;
        }

        .data-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <h1>Fetch API и динамический контент</h1>

    <div class="controls">
        <button id="load-users">Загрузить пользователей</button>
        <button id="load-posts">Загрузить посты</button>
        <button id="load-comments">Загрузить комментарии</button>
    </div>

    <div id="status" aria-live="polite"></div>
    <div id="content-container"></div>

    <script>
        class DataFetcher {
            constructor() {
                this.contentContainer = document.getElementById('content-container');
                this.statusElement = document.getElementById('status');
                this.setupEventListeners();
            }

            setupEventListeners() {
                document.getElementById('load-users').addEventListener('click', () => this.loadUsers());
                document.getElementById('load-posts').addEventListener('click', () => this.loadPosts());
                document.getElementById('load-comments').addEventListener('click', () => this.loadComments());
            }

            async fetchData(url, options = {}) {
                this.showLoading(true);

                try {
                    const response = await fetch(url, {
                        headers: {
                            'Content-Type': 'application/json',
                            ...options.headers
                        },
                        ...options
                    });

                    if (!response.ok) {
                        throw new Error(`HTTP error! status: ${response.status}`);
                    }

                    const data = await response.json();
                    return data;
                } catch (error) {
                    this.showError(error.message);
                    return null;
                } finally {
                    this.showLoading(false);
                }
            }

            async loadUsers() {
                const users = await this.fetchData('https://jsonplaceholder.typicode.com/users');

                if (users) {
                    this.renderUsers(users);
                    this.showSuccess(`Загружено ${users.length} пользователей`);
                }
            }

            async loadPosts() {
                const posts = await this.fetchData('https://jsonplaceholder.typicode.com/posts');

                if (posts) {
                    this.renderPosts(posts);
                    this.showSuccess(`Загружено ${posts.length} постов`);
                }
            }

            async loadComments() {
                const comments = await this.fetchData('https://jsonplaceholder.typicode.com/comments');

                if (comments) {
                    this.renderComments(comments);
                    this.showSuccess(`Загружено ${comments.length} комментариев`);
                }
            }

            renderUsers(users) {
                this.contentContainer.innerHTML = `
                    <div class="data-grid">
                        ${users.map(user => `
                            <div class="user-card">
                                <h3>${user.name}</h3>
                                <p><strong>Username:</strong> ${user.username}</p>
                                <p><strong>Email:</strong> ${user.email}</p>
                                <p><strong>Phone:</strong> ${user.phone}</p>
                                <p><strong>Website:</strong> <a href="http://${user.website}" target="_blank">${user.website}</a></p>
                                <p><strong>Company:</strong> ${user.company.name}</p>
                            </div>
                        `).join('')}
                    </div>
                `;
            }

            renderPosts(posts) {
                this.contentContainer.innerHTML = `
                    <div class="data-grid">
                        ${posts.map(post => `
                            <div class="user-card">
                                <h3>${post.title}</h3>
                                <p><strong>Автор:</strong> Пост #${post.id}</p>
                                <p><strong>Содержимое:</strong></p>
                                <p>${post.body}</p>
                            </div>
                        `).join('')}
                    </div>
                `;
            }

            renderComments(comments) {
                // Показываем только первые 20 комментариев
                const limitedComments = comments.slice(0, 20);

                this.contentContainer.innerHTML = `
                    <div class="data-grid">
                        ${limitedComments.map(comment => `
                            <div class="user-card">
                                <h3>${comment.name}</h3>
                                <p><strong>Email:</strong> ${comment.email}</p>
                                <p><strong>Содержимое:</strong></p>
                                <p>${comment.body}</p>
                            </div>
                        `).join('')}
                    </div>
                `;
            }

            showLoading(show) {
                if (show) {
                    this.contentContainer.classList.add('loading');
                    this.statusElement.innerHTML = '<div class="spinner"></div> Загрузка...';
                } else {
                    this.contentContainer.classList.remove('loading');
                    this.statusElement.innerHTML = '';
                }
            }

            showError(message) {
                this.statusElement.innerHTML = `<div class="error-message">Ошибка: ${message}</div>`;
            }

            showSuccess(message) {
                this.statusElement.innerHTML = `<div class="success-message">${message}</div>`;

                // Автоматически скрываем сообщение через 3 секунды
                setTimeout(() => {
                    if (this.statusElement.querySelector('.success-message')) {
                        this.statusElement.innerHTML = '';
                    }
                }, 3000);
            }
        }

        // Инициализация
        const dataFetcher = new DataFetcher();
    </script>
</body>
</html>
```

### WebSockets и динамические обновления

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>WebSockets и динамические обновления</title>
    <style>
        .chat-container {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }

        .messages-container {
            height: 400px;
            overflow-y: auto;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
            background-color: #f9f9f9;
        }

        .message {
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 8px;
            background-color: white;
            border: 1px solid #eee;
        }

        .message.own {
            background-color: #e3f2fd;
            margin-left: 50px;
        }

        .message.other {
            background-color: #f1f8e9;
            margin-right: 50px;
        }

        .message-header {
            font-weight: bold;
            margin-bottom: 5px;
            font-size: 0.9em;
        }

        .message-time {
            font-size: 0.7em;
            color: #666;
            float: right;
        }

        .message-content {
            margin: 0;
        }

        .input-container {
            display: flex;
            gap: 10px;
        }

        .message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        .send-button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .connection-status {
            padding: 10px;
            border-radius: 4px;
            margin-bottom: 15px;
            text-align: center;
        }

        .status-connected {
            background-color: #e8f5e9;
            color: #2e7d32;
        }

        .status-disconnected {
            background-color: #ffebee;
            color: #c62828;
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <h1>Чат с WebSockets</h1>

        <div id="connection-status" class="connection-status status-disconnected">
            Отключено от сервера
        </div>

        <div id="messages-container" class="messages-container"></div>

        <div class="input-container">
            <input type="text" id="message-input" class="message-input" placeholder="Введите сообщение..." disabled>
            <button id="send-button" class="send-button" disabled>Отправить</button>
        </div>
    </div>

    <script>
        class ChatWebSocket {
            constructor(url) {
                this.url = url;
                this.ws = null;
                this.isConnected = false;

                this.messagesContainer = document.getElementById('messages-container');
                this.connectionStatus = document.getElementById('connection-status');
                this.messageInput = document.getElementById('message-input');
                this.sendButton = document.getElementById('send-button');

                this.setupEventListeners();
                this.connect();
            }

            setupEventListeners() {
                document.getElementById('send-button').addEventListener('click', () => this.sendMessage());

                document.getElementById('message-input').addEventListener('keypress', (e) => {
                    if (e.key === 'Enter' && !this.messageInput.disabled) {
                        this.sendMessage();
                    }
                });
            }

            connect() {
                try {
                    // В реальном приложении это будет реальный WebSocket URL
                    // this.ws = new WebSocket(this.url);

                    // Для демонстрации создаем mock-соединение
                    this.ws = {
                        send: (message) => {
                            console.log('Отправлено:', message);
                            // Имитация получения сообщения от других пользователей
                            setTimeout(() => {
                                this.receiveMessage({
                                    user: 'Система',
                                    text: 'Сообщение отправлено',
                                    timestamp: new Date().toISOString()
                                });
                            }, 500);
                        },
                        close: () => {
                            this.isConnected = false;
                            this.updateConnectionStatus();
                        }
                    };

                    // Имитация успешного подключения
                    setTimeout(() => {
                        this.isConnected = true;
                        this.updateConnectionStatus();
                        this.enableInputs();

                        // Имитация получения сообщений
                        setInterval(() => {
                            if (Math.random() > 0.8) { // 20% шанс получить сообщение
                                const users = ['Алексей', 'Мария', 'Иван', 'Елена', 'Дмитрий'];
                                const messages = [
                                    'Привет всем!', 'Как дела?', 'Работаю над проектом', 'Кто на связи?',
                                    'Нужна помощь?', 'Все прошло отлично!', 'Планирую встречу', 'Обновил документацию'
                                ];

                                this.receiveMessage({
                                    user: users[Math.floor(Math.random() * users.length)],
                                    text: messages[Math.floor(Math.random() * messages.length)],
                                    timestamp: new Date().toISOString()
                                });
                            }
                        }, 3000);
                    }, 1000);

                    /*
                    this.ws.onopen = () => {
                        this.isConnected = true;
                        this.updateConnectionStatus();
                        this.enableInputs();
                    };

                    this.ws.onmessage = (event) => {
                        const message = JSON.parse(event.data);
                        this.receiveMessage(message);
                    };

                    this.ws.onclose = () => {
                        this.isConnected = false;
                        this.updateConnectionStatus();
                        this.disableInputs();
                    };

                    this.ws.onerror = (error) => {
                        console.error('WebSocket error:', error);
                        this.isConnected = false;
                        this.updateConnectionStatus();
                        this.disableInputs();
                    };
                    */
                } catch (error) {
                    console.error('Ошибка подключения к WebSocket:', error);
                    this.updateConnectionStatus();
                    this.disableInputs();
                }
            }

            sendMessage() {
                if (!this.isConnected || !this.messageInput.value.trim()) return;

                const message = {
                    user: 'Вы',
                    text: this.messageInput.value.trim(),
                    timestamp: new Date().toISOString()
                };

                this.ws.send(JSON.stringify(message));
                this.addMessage(message, true);
                this.messageInput.value = '';
            }

            receiveMessage(message) {
                this.addMessage(message, false);
            }

            addMessage(message, isOwn) {
                const messageElement = document.createElement('div');
                messageElement.className = `message ${isOwn ? 'own' : 'other'}`;

                const time = new Date(message.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });

                messageElement.innerHTML = `
                    <div class="message-header">
                        ${message.user}
                        <span class="message-time">${time}</span>
                    </div>
                    <p class="message-content">${message.text}</p>
                `;

                this.messagesContainer.appendChild(messageElement);

                // Прокрутка вниз
                this.messagesContainer.scrollTop = this.messagesContainer.scrollHeight;
            }

            updateConnectionStatus() {
                if (this.isConnected) {
                    this.connectionStatus.textContent = 'Подключено к серверу';
                    this.connectionStatus.className = 'connection-status status-connected';
                } else {
                    this.connectionStatus.textContent = 'Отключено от сервера';
                    this.connectionStatus.className = 'connection-status status-disconnected';
                }
            }

            enableInputs() {
                this.messageInput.disabled = false;
                this.sendButton.disabled = false;
            }

            disableInputs() {
                this.messageInput.disabled = true;
                this.sendButton.disabled = true;
            }

            disconnect() {
                if (this.ws) {
                    this.ws.close();
                }
            }
        }

        // Инициализация чата
        const chat = new ChatWebSocket('ws://localhost:8080/chat');
    </script>
</body>
</html>
```

## Интеграция с современными фреймворками

### Использование Web Components с JavaScript

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Web Components с JavaScript</title>
</head>
<body>
    <h1>Web Components с JavaScript</h1>

    <user-profile-card
        username="john_doe"
        email="john@example.com"
        avatar="avatar.jpg">
    </user-profile-card>

    <collapsible-section title="Информация">
        <p>Содержимое collapsible секции</p>
        <ul>
            <li>Элемент 1</li>
            <li>Элемент 2</li>
            <li>Элемент 3</li>
        </ul>
    </collapsible-section>

    <data-table-component id="users-table"></data-table-component>

    <script>
        // Кастомный элемент профиля пользователя
        class UserProfileCard extends HTMLElement {
            constructor() {
                super();

                this.attachShadow({ mode: 'open' });
                this.render();
            }

            static get observedAttributes() {
                return ['username', 'email', 'avatar'];
            }

            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    this[name] = newValue;
                    this.render();
                }
            }

            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: inline-block;
                            font-family: Arial, sans-serif;
                        }

                        .card {
                            border: 1px solid #ddd;
                            border-radius: 8px;
                            padding: 20px;
                            max-width: 300px;
                            background-color: white;
                            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                        }

                        .avatar {
                            width: 80px;
                            height: 80px;
                            border-radius: 50%;
                            object-fit: cover;
                            margin-bottom: 15px;
                        }

                        .username {
                            font-size: 1.2em;
                            font-weight: bold;
                            margin: 0 0 10px 0;
                        }

                        .email {
                            color: #666;
                            margin: 0 0 15px 0;
                        }

                        .actions {
                            display: flex;
                            gap: 10px;
                        }

                        .action-btn {
                            padding: 8px 16px;
                            border: none;
                            border-radius: 4px;
                            cursor: pointer;
                            font-size: 0.9em;
                        }

                        .message-btn {
                            background-color: #007acc;
                            color: white;
                        }

                        .follow-btn {
                            background-color: #28a745;
                            color: white;
                        }
                    </style>

                    <div class="card">
                        <img src="${this.avatar || 'default-avatar.png'}"
                             alt="Аватар ${this.username}"
                             class="avatar">
                        <h3 class="username">${this.username || 'Анонимный пользователь'}</h3>
                        <p class="email">${this.email || 'Email не указан'}</p>
                        <div class="actions">
                            <button class="action-btn message-btn">Сообщение</button>
                            <button class="action-btn follow-btn">Подписаться</button>
                        </div>
                    </div>
                `;
            }
        }

        // Кастомный элемент сворачиваемой секции
        class CollapsibleSection extends HTMLElement {
            constructor() {
                super();

                this.attachShadow({ mode: 'open' });
                this.expanded = false;

                this.render();
                this.bindEvents();
            }

            static get observedAttributes() {
                return ['title', 'expanded'];
            }

            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    if (name === 'title') {
                        this.updateTitle();
                    } else if (name === 'expanded') {
                        this.expanded = newValue !== 'false';
                        this.toggleContent();
                    }
                }
            }

            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                            font-family: Arial, sans-serif;
                        }

                        .header {
                            background-color: #f5f5f5;
                            padding: 15px;
                            cursor: pointer;
                            border: 1px solid #ddd;
                            border-radius: 8px 8px 0 0;
                            display: flex;
                            justify-content: space-between;
                            align-items: center;
                        }

                        .header:hover {
                            background-color: #e9ecef;
                        }

                        .toggle-icon {
                            transition: transform 0.3s ease;
                        }

                        .content {
                            border: 1px solid #ddd;
                            border-top: none;
                            border-radius: 0 0 8px 8px;
                            padding: 20px;
                            display: none;
                        }

                        .content.expanded {
                            display: block;
                        }
                    </style>

                    <div class="header" tabindex="0">
                        <span class="title">${this.getAttribute('title') || 'Секция'}</span>
                        <span class="toggle-icon">▼</span>
                    </div>
                    <div class="content">
                        <slot></slot>
                    </div>
                `;
            }

            bindEvents() {
                const header = this.shadowRoot.querySelector('.header');

                header.addEventListener('click', () => {
                    this.toggle();
                });

                header.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter' || e.key === ' ') {
                        e.preventDefault();
                        this.toggle();
                    }
                });
            }

            toggle() {
                this.expanded = !this.expanded;
                this.setAttribute('expanded', this.expanded);
                this.toggleContent();
            }

            toggleContent() {
                const content = this.shadowRoot.querySelector('.content');
                const icon = this.shadowRoot.querySelector('.toggle-icon');

                if (this.expanded) {
                    content.classList.add('expanded');
                    icon.style.transform = 'rotate(180deg)';
                } else {
                    content.classList.remove('expanded');
                    icon.style.transform = 'rotate(0deg)';
                }
            }

            updateTitle() {
                const titleElement = this.shadowRoot.querySelector('.title');
                if (titleElement) {
                    titleElement.textContent = this.getAttribute('title') || 'Секция';
                }
            }
        }

        // Кастомный элемент таблицы данных
        class DataTableComponent extends HTMLElement {
            constructor() {
                super();

                this.attachShadow({ mode: 'open' });
                this.data = [];

                this.render();
            }

            static get observedAttributes() {
                return ['data', 'columns'];
            }

            connectedCallback() {
                // Загрузка данных из атрибута или через JavaScript
                const dataAttr = this.getAttribute('data');
                if (dataAttr) {
                    try {
                        this.data = JSON.parse(dataAttr);
                        this.renderTable();
                    } catch (e) {
                        console.error('Ошибка парсинга данных:', e);
                    }
                }
            }

            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                            font-family: Arial, sans-serif;
                        }

                        table {
                            width: 100%;
                            border-collapse: collapse;
                            margin: 20px 0;
                        }

                        th, td {
                            padding: 12px;
                            text-align: left;
                            border-bottom: 1px solid #ddd;
                        }

                        th {
                            background-color: #f8f9fa;
                            font-weight: bold;
                        }

                        tr:hover {
                            background-color: #f5f5f5;
                        }

                        .loading {
                            text-align: center;
                            padding: 20px;
                            color: #666;
                        }
                    </style>

                    <div class="table-container">
                        <table>
                            <thead id="table-header"></thead>
                            <tbody id="table-body"></tbody>
                        </table>
                        <div id="loading-indicator" class="loading" style="display: none;">
                            Загрузка данных...
                        </div>
                    </div>
                `;
            }

            setData(data) {
                this.data = data;
                this.renderTable();
            }

            renderTable() {
                if (!this.data || this.data.length === 0) {
                    this.showLoading(false);
                    this.shadowRoot.getElementById('table-body').innerHTML = '<tr><td colspan="100%">Нет данных</td></tr>';
                    return;
                }

                this.showLoading(false);

                // Определяем колонки на основе первого элемента данных
                const columns = Object.keys(this.data[0]);

                // Рендерим заголовки
                const headerRow = this.shadowRoot.getElementById('table-header');
                headerRow.innerHTML = '';

                const headerTr = document.createElement('tr');
                columns.forEach(col => {
                    const th = document.createElement('th');
                    th.textContent = this.formatColumnName(col);
                    headerTr.appendChild(th);
                });
                headerRow.appendChild(headerTr);

                // Рендерим тело таблицы
                const body = this.shadowRoot.getElementById('table-body');
                body.innerHTML = '';

                this.data.forEach(row => {
                    const tr = document.createElement('tr');

                    columns.forEach(col => {
                        const td = document.createElement('td');
                        td.textContent = row[col];
                        tr.appendChild(td);
                    });

                    body.appendChild(tr);
                });
            }

            formatColumnName(name) {
                // Форматируем имя колонки (например, camelCase -> "Camel Case")
                return name
                    .replace(/([A-Z])/g, ' $1')
                    .replace(/^./, str => str.toUpperCase());
            }

            showLoading(show) {
                const loadingIndicator = this.shadowRoot.getElementById('loading-indicator');
                loadingIndicator.style.display = show ? 'block' : 'none';
            }
        }

        // Регистрация кастомных элементов
        customElements.define('user-profile-card', UserProfileCard);
        customElements.define('collapsible-section', CollapsibleSection);
        customElements.define('data-table-component', DataTableComponent);

        // Пример использования JavaScript для динамического добавления элементов
        document.addEventListener('DOMContentLoaded', () => {
            // Динамическое создание элементов
            const newUserCard = document.createElement('user-profile-card');
            newUserCard.setAttribute('username', 'jane_doe');
            newUserCard.setAttribute('email', 'jane@example.com');
            newUserCard.setAttribute('avatar', 'jane-avatar.jpg');

            document.body.appendChild(newUserCard);

            // Загрузка данных в таблицу
            const tableComponent = document.getElementById('users-table');
            const sampleData = [
                { id: 1, name: 'Иван Иванов', email: 'ivan@example.com', role: 'Администратор' },
                { id: 2, name: 'Мария Петрова', email: 'maria@example.com', role: 'Модератор' },
                { id: 3, name: 'Алексей Сидоров', email: 'alexey@example.com', role: 'Пользователь' }
            ];

            tableComponent.setData(sampleData);
        });
    </script>
</body>
</html>
```

## Практические примеры и шаблоны

### Шаблон для динамической формы
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Динамическая форма</title>
</head>
<body>
    <h1>Динамическая форма</h1>

    <form id="dynamic-form">
        <div id="form-fields"></div>
        <button type="button" onclick="addField()">Добавить поле</button>
        <button type="submit">Отправить</button>
    </form>

    <!-- Шаблон для поля формы -->
    <template id="field-template">
        <div class="form-field">
            <label>Поле <span class="field-index"></span>:</label>
            <input type="text" name="field" class="field-input">
            <button type="button" class="remove-field" onclick="removeField(this)">Удалить</button>
        </div>
    </template>

    <script>
        let fieldCount = 0;

        function addField() {
            const template = document.getElementById('field-template');
            const clone = template.content.cloneNode(true);

            fieldCount++;
            clone.querySelector('.field-index').textContent = fieldCount;
            clone.querySelector('.field-input').name = `field_${fieldCount}`;

            document.getElementById('form-fields').appendChild(clone);
        }

        function removeField(button) {
            button.parentElement.remove();
        }

        document.getElementById('dynamic-form').addEventListener('submit', (e) => {
            e.preventDefault();

            const formData = new FormData(e.target);
            const data = Object.fromEntries(formData);

            console.log('Данные формы:', data);
            alert('Форма отправлена! Проверьте консоль для данных.');
        });
    </script>

    <style>
        .form-field {
            margin-bottom: 15px;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        .remove-field {
            background-color: #dc3545;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 4px;
            cursor: pointer;
            margin-left: 10px;
        }
    </style>
</body>
</html>
```

### Шаблон для карточки продукта с динамическим содержимым
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Карточка продукта с динамическим содержимым</title>
</head>
<body>
    <h1>Каталог товаров</h1>

    <div id="products-container"></div>

    <!-- Шаблон карточки продукта -->
    <template id="product-card-template">
        <div class="product-card">
            <img class="product-image" src="" alt="">
            <div class="product-info">
                <h3 class="product-title"></h3>
                <p class="product-description"></p>
                <div class="product-price-container">
                    <span class="product-price"></span>
                    <span class="product-old-price"></span>
                </div>
                <div class="product-actions">
                    <button class="add-to-cart">Добавить в корзину</button>
                    <button class="add-to-wishlist">В избранное</button>
                </div>
            </div>
        </div>
    </template>

    <script>
        class ProductCatalog {
            constructor(containerId) {
                this.container = document.getElementById(containerId);
                this.template = document.getElementById('product-card-template');
            }

            addProduct(product) {
                const clone = this.template.content.cloneNode(true);

                // Заполнение данных
                clone.querySelector('.product-image').src = product.image;
                clone.querySelector('.product-image').alt = product.title;
                clone.querySelector('.product-title').textContent = product.title;
                clone.querySelector('.product-description').textContent = product.description;
                clone.querySelector('.product-price').textContent = this.formatPrice(product.price);

                if (product.oldPrice) {
                    clone.querySelector('.product-old-price').textContent = this.formatPrice(product.oldPrice);
                    clone.querySelector('.product-old-price').style.textDecoration = 'line-through';
                } else {
                    clone.querySelector('.product-old-price').style.display = 'none';
                }

                // Добавление обработчиков
                clone.querySelector('.add-to-cart').addEventListener('click', () => {
                    this.addToCart(product);
                });

                clone.querySelector('.add-to-wishlist').addEventListener('click', () => {
                    this.addToWishlist(product);
                });

                this.container.appendChild(clone);
            }

            formatPrice(price) {
                return new Intl.NumberFormat('ru-RU', {
                    style: 'currency',
                    currency: 'RUB'
                }).format(price);
            }

            addToCart(product) {
                console.log('Добавлено в корзину:', product.title);
                // Реализация добавления в корзину
            }

            addToWishlist(product) {
                console.log('Добавлено в избранное:', product.title);
                // Реализация добавления в избранное
            }
        }

        // Использование
        const catalog = new ProductCatalog('products-container');

        // Пример данных продуктов
        const products = [
            {
                id: 1,
                title: 'Смартфон iPhone 15',
                description: 'Новейший iPhone с передовыми технологиями',
                price: 99990,
                oldPrice: 109990,
                image: 'iphone15.jpg'
            },
            {
                id: 2,
                title: 'Ноутбук MacBook Air',
                description: 'Ультрабук для работы и развлечений',
                price: 89990,
                image: 'macbook-air.jpg'
            },
            {
                id: 3,
                title: 'Планшет iPad Pro',
                description: 'Профессиональный планшет для творчества',
                price: 79990,
                oldPrice: 84990,
                image: 'ipad-pro.jpg'
            }
        ];

        // Добавление продуктов
        products.forEach(product => catalog.addProduct(product));
    </script>

    <style>
        #products-container {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
        }

        .product-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            overflow: hidden;
            background-color: white;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }

        .product-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
        }

        .product-info {
            padding: 15px;
        }

        .product-title {
            margin: 0 0 10px 0;
            color: #333;
        }

        .product-description {
            color: #666;
            margin-bottom: 15px;
            line-height: 1.4;
        }

        .product-price-container {
            margin-bottom: 15px;
        }

        .product-price {
            font-size: 1.2em;
            font-weight: bold;
            color: #007acc;
        }

        .product-old-price {
            margin-left: 10px;
            color: #999;
        }

        .product-actions {
            display: flex;
            gap: 10px;
        }

        .add-to-cart, .add-to-wishlist {
            flex: 1;
            padding: 10px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .add-to-cart {
            background-color: #007acc;
            color: white;
        }

        .add-to-wishlist {
            background-color: #f8f9fa;
            color: #333;
            border: 1px solid #ddd;
        }
    </style>
</body>
</html>
```

## Современные возможности интеграции

### Web Components с безопасностью
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасные Web Components</title>
</head>
<body>
    <h1>Безопасные Web Components</h1>
    
    <secure-data-display id="secure-data" data-source="api/users"></secure-data-display>
    
    <form-validator id="form-validator" form-id="user-form"></form-validator>
    
    <script>
        // Безопасный компонент отображения данных
        class SecureDataDisplay extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'open' });
                
                // Санитизированный шаблон
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                            font-family: Arial, sans-serif;
                        }
                        
                        .data-container {
                            padding: 20px;
                            border: 1px solid #ddd;
                            border-radius: 8px;
                            background-color: white;
                        }
                        
                        .loading {
                            text-align: center;
                            padding: 20px;
                            color: #666;
                        }
                        
                        .error {
                            color: #d32f2f;
                            background-color: #ffebee;
                            padding: 15px;
                            border-radius: 4px;
                        }
                        
                        .data-content {
                            line-height: 1.6;
                        }
                    </style>
                    
                    <div class="data-container">
                        <div class="loading" id="loading">Загрузка данных...</div>
                        <div class="error" id="error" style="display: none;"></div>
                        <div class="data-content" id="data-content"></div>
                    </div>
                `;
                
                this.dataSource = this.getAttribute('data-source');
                this.loadingElement = this.shadowRoot.getElementById('loading');
                this.errorElement = this.shadowRoot.getElementById('error');
                this.contentElement = this.shadowRoot.getElementById('data-content');
            }
            
            async connectedCallback() {
                if (this.dataSource) {
                    await this.loadData();
                }
            }
            
            async loadData() {
                try {
                    // Проверка безопасности источника данных
                    if (!this.isValidDataSource(this.dataSource)) {
                        throw new Error('Недопустимый источник данных');
                    }
                    
                    this.showLoading(true);
                    
                    // Загрузка данных с безопасной обработкой
                    const response = await fetch(this.dataSource, {
                        headers: {
                            'Content-Type': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest'
                        }
                    });
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка: ${response.status}`);
                    }
                    
                    const data = await response.json();
                    
                    // Санитизация данных перед отображением
                    this.displayData(this.sanitizeData(data));
                    
                } catch (error) {
                    this.showError(error.message);
                } finally {
                    this.showLoading(false);
                }
            }
            
            isValidDataSource(source) {
                // Проверка, что источник данных безопасен
                const allowedSources = [
                    /^\/api\//,
                    /^https:\/\/trusted-api\.example\.com\//
                ];
                
                return allowedSources.some(regex => regex.test(source));
            }
            
            sanitizeData(data) {
                // Рекурсивная санитизация данных
                if (typeof data === 'string') {
                    return this.escapeHTML(data);
                } else if (Array.isArray(data)) {
                    return data.map(item => this.sanitizeData(item));
                } else if (typeof data === 'object' && data !== null) {
                    const sanitized = {};
                    for (const [key, value] of Object.entries(data)) {
                        sanitized[key] = this.sanitizeData(value);
                    }
                    return sanitized;
                }
                return data;
            }
            
            escapeHTML(str) {
                if (typeof str !== 'string') return str;
                
                return str
                    .replace(/&/g, '&amp;')
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/\//g, '&#x2F;');
            }
            
            displayData(data) {
                this.contentElement.innerHTML = `
                    <pre>${JSON.stringify(data, null, 2)}</pre>
                `;
            }
            
            showError(message) {
                this.errorElement.textContent = `Ошибка: ${message}`;
                this.errorElement.style.display = 'block';
                this.loadingElement.style.display = 'none';
            }
            
            showLoading(show) {
                this.loadingElement.style.display = show ? 'block' : 'none';
            }
        }
        
        // Безопасный валидатор форм
        class FormValidator extends HTMLElement {
            constructor() {
                super();
                
                this.attachShadow({ mode: 'open' });
                this.form = null;
                
                this.render();
            }
            
            static get observedAttributes() {
                return ['form-id'];
            }
            
            attributeChangedCallback(name, oldValue, newValue) {
                if (oldValue !== newValue) {
                    if (name === 'form-id') {
                        this.setupFormValidation(newValue);
                    }
                }
            }
            
            render() {
                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                        }
                        
                        .validation-results {
                            margin-top: 10px;
                            padding: 10px;
                            border-radius: 4px;
                            font-size: 0.9em;
                        }
                        
                        .validation-success {
                            background-color: #e8f5e8;
                            color: #2e7d32;
                        }
                        
                        .validation-error {
                            background-color: #ffebee;
                            color: #c62828;
                        }
                    </style>
                    
                    <div class="validation-results" id="validation-results"></div>
                `;
            }
            
            setupFormValidation(formId) {
                this.form = document.getElementById(formId);
                
                if (this.form) {
                    this.form.addEventListener('submit', (e) => {
                        e.preventDefault();
                        this.validateForm();
                    });
                    
                    // Валидация в реальном времени
                    this.form.addEventListener('input', (e) => {
                        if (e.target.matches('input[required], textarea[required]')) {
                            this.validateField(e.target);
                        }
                    });
                }
            }
            
            validateForm() {
                const fields = this.form.querySelectorAll('input[required], textarea[required]');
                let isValid = true;
                
                fields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                });
                
                if (isValid) {
                    this.showResult('Форма валидна', 'success');
                    this.form.submit(); // Отправка формы после валидации
                } else {
                    this.showResult('Форма содержит ошибки', 'error');
                }
            }
            
            validateField(field) {
                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                
                if (field.hasAttribute('required') && !field.value.trim()) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    this.showFieldError(field, 'Это поле обязательно');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    this.showFieldError(field, 'Введите действительный email');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    this.showFieldError(field, `Минимум ${field.minLength} символов`);
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    this.showFieldError(field, `Максимум ${field.maxLength} символов`);
                    return false;
                }
                
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                this.clearFieldError(field);
                return true;
            }
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            showFieldError(field, message) {
                // Создаем или обновляем элемент ошибки
                let errorElement = field.parentNode.querySelector('.field-error');
                if (!errorElement) {
                    errorElement = document.createElement('div');
                    errorElement.className = 'field-error';
                    errorElement.style.color = '#c62828';
                    errorElement.style.fontSize = '0.875em';
                    errorElement.style.marginTop = '5px';
                    field.parentNode.appendChild(errorElement);
                }
                
                errorElement.textContent = message;
            }
            
            clearFieldError(field) {
                const errorElement = field.parentNode.querySelector('.field-error');
                if (errorElement) {
                    errorElement.remove();
                }
            }
            
            showResult(message, type) {
                const resultElement = this.shadowRoot.getElementById('validation-results');
                resultElement.textContent = message;
                resultElement.className = `validation-results validation-${type}`;
            }
        }
        
        // Регистрация компонентов
        customElements.define('secure-data-display', SecureDataDisplay);
        customElements.define('form-validator', FormValidator);
    </script>
</body>
</html>
```

### Интеграция с WebSockets и безопасностью
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасная интеграция с WebSockets</title>
</head>
<body>
    <h1>Безопасная WebSocket интеграция</h1>
    
    <div id="secure-chat-container">
        <div id="secure-messages" class="messages-container"></div>
        <form id="secure-message-form">
            <input type="text" id="secure-message-input" placeholder="Введите сообщение..." required>
            <button type="submit">Отправить</button>
        </form>
    </div>

    <script>
        class SecureWebSocketClient {
            constructor(url, options = {}) {
                this.url = url;
                this.options = options;
                this.ws = null;
                this.isConnected = false;
                this.reconnectAttempts = 0;
                this.maxReconnectAttempts = 5;
                this.reconnectInterval = 3000;

                this.messageContainer = document.getElementById('secure-messages');
                this.messageForm = document.getElementById('secure-message-form');
                this.messageInput = document.getElementById('secure-message-input');

                this.setupEventListeners();
                this.connect();
            }

            setupEventListeners() {
                this.messageForm.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.sendMessage(this.messageInput.value.trim());
                    this.messageInput.value = '';
                });
            }

            connect() {
                try {
                    // В реальном приложении: this.ws = new WebSocket(this.url);

                    // Для демонстрации создаем mock-соединение
                    this.ws = {
                        send: (message) => {
                            // Санитизация сообщения перед отправкой
                            const sanitizedMessage = this.sanitizeMessage(JSON.parse(message));
                            console.log('Отправлено:', sanitizedMessage);

                            // Имитация получения сообщения от других пользователей
                            setTimeout(() => {
                                this.handleMessage(JSON.stringify({
                                    user: 'Система',
                                    text: 'Сообщение доставлено',
                                    timestamp: new Date().toISOString()
                                }));
                            }, 500);
                        },
                        close: () => {
                            this.isConnected = false;
                            this.updateConnectionStatus();
                        }
                    };

                    // Имитация успешного подключения
                    setTimeout(() => {
                        this.isConnected = true;
                        this.reconnectAttempts = 0;
                        this.updateConnectionStatus();

                        // Имитация получения сообщений
                        setInterval(() => {
                            if (Math.random() > 0.8) { // 20% шанс получить сообщение
                                const users = ['Алексей', 'Мария', 'Иван', 'Елена', 'Дмитрий'];
                                const messages = [
                                    'Привет всем!', 'Как дела?', 'Работаю над проектом', 'Кто на связи?',
                                    'Нужна помощь?', 'Все прошло отлично!', 'Планирую встречу', 'Обновил документацию'
                                ];

                                this.handleMessage(JSON.stringify({
                                    user: users[Math.floor(Math.random() * users.length)],
                                    text: messages[Math.floor(Math.random() * messages.length)],
                                    timestamp: new Date().toISOString()
                                }));
                            }
                        }, 3000);
                    }, 1000);

                    /*
                    this.ws.onopen = () => {
                        this.isConnected = true;
                        this.reconnectAttempts = 0;
                        this.updateConnectionStatus();
                    };

                    this.ws.onmessage = (event) => {
                        this.handleMessage(event.data);
                    };

                    this.ws.onclose = () => {
                        this.isConnected = false;
                        this.updateConnectionStatus();

                        // Автоматическое переподключение
                        if (this.reconnectAttempts < this.maxReconnectAttempts) {
                            setTimeout(() => {
                                this.reconnectAttempts++;
                                this.connect();
                            }, this.reconnectInterval);
                        }
                    };

                    this.ws.onerror = (error) => {
                        console.error('WebSocket ошибка:', error);
                    };
                    */
                } catch (error) {
                    console.error('Ошибка подключения к WebSocket:', error);
                    this.updateConnectionStatus();
                }
            }

            sendMessage(text) {
                if (!this.isConnected || !text) return;

                // Проверка безопасности сообщения
                if (this.containsMaliciousContent(text)) {
                    console.warn('Обнаружено потенциально опасное содержимое в сообщении:', text);
                    return;
                }

                const message = {
                    text: this.sanitizeInput(text),
                    timestamp: new Date().toISOString(),
                    userId: this.getUserId(),
                    sessionId: this.getSessionId()
                };

                this.ws.send(JSON.stringify(message));
                this.displayMessage(message, true); // true - собственное сообщение
            }

            handleMessage(data) {
                try {
                    const message = JSON.parse(data);

                    // Дополнительная проверка безопасности
                    if (!this.validateMessage(message)) {
                        console.warn('Небезопасное сообщение отклонено:', message);
                        return;
                    }

                    // Санитизация полученного сообщения
                    const sanitizedMessage = this.sanitizeMessage(message);
                    this.displayMessage(sanitizedMessage, false); // false - чужое сообщение
                } catch (error) {
                    console.error('Ошибка обработки сообщения:', error);
                }
            }

            validateMessage(message) {
                // Проверка структуры сообщения
                if (!message || typeof message !== 'object') {
                    return false;
                }

                // Проверка на вредоносное содержимое
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i
                ];

                const messageText = message.text || '';
                return !dangerousPatterns.some(pattern => pattern.test(messageText));
            }

            sanitizeMessage(message) {
                // Санитизация сообщения
                if (message.text) {
                    message.text = this.sanitizeInput(message.text);
                }
                if (message.user) {
                    message.user = this.sanitizeInput(message.user);
                }
                return message;
            }

            sanitizeInput(input) {
                if (typeof input !== 'string') return input;

                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
            }

            displayMessage(message, isOwn) {
                const messageElement = document.createElement('div');
                messageElement.className = `message ${isOwn ? 'own' : 'other'}`;

                const time = new Date(message.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });

                messageElement.innerHTML = `
                    <div class="message-header">
                        <span class="message-user">${this.escapeHTML(message.user)}</span>
                        <span class="message-time">${time}</span>
                    </div>
                    <div class="message-content">${this.escapeHTML(message.text)}</div>
                `;

                this.messageContainer.appendChild(messageElement);

                // Прокрутка вниз
                this.messageContainer.scrollTop = this.messageContainer.scrollHeight;
            }

            updateConnectionStatus() {
                const statusElement = document.getElementById('connection-status');
                if (this.isConnected) {
                    statusElement.textContent = 'Подключено к серверу';
                    statusElement.className = 'connection-status status-connected';
                } else {
                    statusElement.textContent = 'Отключено от сервера';
                    statusElement.className = 'connection-status status-disconnected';
                }
            }

            getUserId() {
                // В реальном приложении - из сессии или токена
                return localStorage.getItem('userId') || 'anonymous';
            }

            getSessionId() {
                // В реальном приложении - из сессии
                return sessionStorage.getItem('sessionId') || 'session_' + Date.now();
            }

            escapeHTML(str) {
                if (typeof str !== 'string') return str;

                return str
                    .replace(/&/g, '&amp;')
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;');
            }

            disconnect() {
                if (this.ws) {
                    this.ws.close();
                }
            }
        }

        // Инициализация
        const secureWS = new SecureWebSocketClient('wss://secure-chat.example.com/ws');
    </script>

    <style>
        #secure-chat-container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }

        .messages-container {
            height: 400px;
            overflow-y: auto;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
            background-color: #f9f9f9;
        }

        .message {
            padding: 10px;
            margin-bottom: 10px;
            border-radius: 8px;
            background-color: white;
            border: 1px solid #eee;
        }

        .message.own {
            background-color: #e3f2fd;
            margin-left: 50px;
        }

        .message.other {
            background-color: #f1f8e9;
            margin-right: 50px;
        }

        .message-header {
            display: flex;
            justify-content: space-between;
            font-size: 0.9em;
            margin-bottom: 5px;
        }

        .message-user {
            font-weight: bold;
        }

        .message-time {
            color: #666;
        }

        .message-content {
            margin: 0;
            line-height: 1.5;
        }

        #secure-message-form {
            display: flex;
            gap: 10px;
        }

        #secure-message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        #secure-message-form button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .connection-status {
            padding: 8px 12px;
            border-radius: 4px;
            margin-bottom: 10px;
            text-align: center;
            font-weight: bold;
        }

        .status-connected {
            background-color: #e8f5e9;
            color: #2e7d32;
        }

        .status-disconnected {
            background-color: #ffebee;
            color: #c62828;
        }
    </style>
</body>
</html>
```

## Проверка и тестирование интеграции

### Инструменты для проверки:
1. **Chrome DevTools** - для отладки JavaScript интеграции
2. **Firefox Developer Tools** - для проверки
3. **Lighthouse** - для комплексной проверки
4. **WebPageTest** - для анализа производительности
5. **Selenium WebDriver** - для автоматизированного тестирования
6. **Jest/Puppeteer** - для unit и integration тестов

### Проверка безопасности интеграции:
1. Проверка на XSS уязвимости
2. Проверка на CSRF уязвимости
3. Проверка использования безопасных API
4. Проверка обработки пользовательского ввода
5. Проверка использования CSP и SRI
6. Проверка обработки ошибок

### Проверка производительности:
1. Время загрузки ресурсов
2. Использование памяти
3. Эффективность DOM манипуляций
4. Обработка событий
5. Асинхронные операции

## Лучшие практики интеграции

### 1. Безопасность при интеграции
```javascript
// Безопасная обработка данных
class SecureDataHandler {
    constructor() {
        this.allowedDomains = [
            'trusted-api.com',
            'secure-service.com',
            'our-domain.com'
        ];
    }
    
    async fetchData(url, options = {}) {
        if (!this.isTrustedDomain(url)) {
            throw new Error('Запрос к ненадежному домену');
        }
        
        const response = await fetch(url, {
            ...options,
            headers: {
                'Content-Type': 'application/json',
                'X-Requested-With': 'XMLHttpRequest',
                'X-Client-Version': '1.0.0',
                ...options.headers
            }
        });
        
        if (!response.ok) {
            throw new Error(`HTTP ошибка: ${response.status}`);
        }
        
        const data = await response.json();
        
        // Проверка целостности данных
        if (!this.validateDataStructure(data)) {
            throw new Error('Неверная структура данных');
        }
        
        return this.sanitizeData(data);
    }
    
    isTrustedDomain(url) {
        try {
            const parsedUrl = new URL(url);
            return this.allowedDomains.some(domain => 
                parsedUrl.hostname === domain || 
                parsedUrl.hostname.endsWith('.' + domain)
            );
        } catch {
            return false;
        }
    }
    
    validateDataStructure(data) {
        // Проверка структуры данных
        if (typeof data !== 'object' || data === null) return false;
        
        // Проверка на наличие потенциально опасного содержимого
        const jsonString = JSON.stringify(data);
        const dangerousPatterns = [
            /<script/i,
            /javascript:/i,
            /on\w+\s*=/i,
            /<iframe/i,
            /<object/i,
            /<embed/i
        ];
        
        return !dangerousPatterns.some(pattern => pattern.test(jsonString));
    }
    
    sanitizeData(data) {
        // Рекурсивная санитизация данных
        if (typeof data === 'string') {
            return this.escapeHTML(data);
        } else if (Array.isArray(data)) {
            return data.map(item => this.sanitizeData(item));
        } else if (typeof data === 'object' && data !== null) {
            const sanitized = {};
            for (const [key, value] of Object.entries(data)) {
                sanitized[key] = this.sanitizeData(value);
            }
            return sanitized;
        }
        return data;
    }
    
    escapeHTML(str) {
        if (typeof str !== 'string') return str;
        
        return str
            .replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#x27;');
    }
}
```

### 2. Производительность и оптимизация
```javascript
// Оптимизированная обработка DOM
class OptimizedDOMHandler {
    constructor() {
        this.fragment = document.createDocumentFragment();
        this.batchUpdates = [];
    }
    
    // Батчевое обновление DOM
    batchUpdate(updates) {
        this.batchUpdates.push(...updates);
        
        if (!this.updateScheduled) {
            this.updateScheduled = true;
            requestAnimationFrame(() => this.executeBatchUpdate());
        }
    }
    
    executeBatchUpdate() {
        this.batchUpdates.forEach(update => {
            update.element[update.property] = update.value;
        });
        
        this.batchUpdates = [];
        this.updateScheduled = false;
    }
    
    // Эффективное добавление элементов
    appendElements(elements) {
        elements.forEach(element => {
            this.fragment.appendChild(element);
        });
        
        // Однократное добавление в DOM
        document.getElementById('container').appendChild(this.fragment);
    }
    
    // Оптимизация событий
    setupEventDelegation() {
        document.addEventListener('click', (e) => {
            // Делегирование событий для оптимизации
            const button = e.target.closest('.action-button');
            if (button) {
                this.handleButtonClick(button);
            }
        });
    }
    
    handleButtonClick(button) {
        const action = button.dataset.action;
        const id = button.dataset.id;
        
        switch(action) {
            case 'edit':
                this.editItem(id);
                break;
            case 'delete':
                this.deleteItem(id);
                break;
        }
    }
    
    // Оптимизация рендеринга
    renderTemplate(templateId, data) {
        const template = document.getElementById(templateId);
        if (!template) return null;
        
        const clone = template.content.cloneNode(true);
        
        // Заполнение шаблона данными
        const elements = clone.querySelectorAll('[data-bind]');
        elements.forEach(element => {
            const property = element.dataset.bind;
            element.textContent = this.sanitizeData(data[property]);
        });
        
        return clone;
    }
}
```

## Заключение

Интеграция HTML с JavaScript открывает широкие возможности для создания интерактивных и динамических веб-приложений. Современные подходы включают:

1. **Эффективное взаимодействие с DOM** - правильное использование методов выбора и манипуляции элементов
2. **Обработка событий** - включая делегирование и асинхронные операции
3. **Использование современных API** - Fetch, WebSockets, IndexedDB и других
4. **Создание веб-компонентов** - для переиспользуемых и инкапсулированных элементов
5. **Безопасность** - обеспечение безопасности при интеграции
6. **Производительность** - оптимизация рендеринга и управления памятью
7. **Доступность** - обеспечение доступности для всех пользователей
8. **Совместимость** - поддержка различных браузеров и устройств

Эти практики позволяют создавать веб-приложения, которые эффективно взаимодействуют с JavaScript и обеспечивают отличный пользовательский опыт.

Современные подходы к интеграции включают:
- Использование Web Components для создания переиспользуемых компонентов
- Интеграцию с Service Workers для оффлайн-функциональности
- WebSockets для реального времени
- IndexedDB для локального хранения данных
- Современные API для безопасности (CSP, SRI)
- Эффективные методы обработки и валидации данных
- Оптимизацию производительности с кешированием
- Поддержку доступности и семантики

Эти технологии позволяют создавать современные, безопасные и высокопроизводительные веб-приложения, которые соответствуют актуальным веб-стандартам и обеспечивают отличный пользовательский опыт на всех устройствах и в различных условиях использования.

## Следующие темы
- [[HTML/Производительность]]
- [[HTML/Безопасность]]
- [[HTML/Доступность]]

## Теги
#html #javascript #integration #dom #events #web-components #indexeddb #websockets #fetch-api #security #accessibility #performance #frontend #web-development #modern-web #csp #sri #csrf #xss-protection #secure-coding #best-practices
# HTML Интеграция: Работа с API

Интеграция HTML с внешними API позволяет создавать динамические и интерактивные веб-приложения. Современные веб-технологии предоставляют мощные инструменты для взаимодействия с серверами, базами данных и внешними сервисами.

## Основы интеграции с API

### Fetch API

Fetch API предоставляет современный способ выполнения HTTP-запросов:

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Fetch API интеграция</title>
</head>
<body>
    <h1>Интеграция с API через Fetch</h1>
    
    <div class="api-controls">
        <button onclick="fetchUsers()">Загрузить пользователей</button>
        <button onclick="createUser()">Создать пользователя</button>
        <button onclick="clearResults()">Очистить</button>
    </div>
    
    <div id="api-results"></div>

    <script>
        class APIIntegration {
            constructor(baseURL = 'https://jsonplaceholder.typicode.com') {
                this.baseURL = baseURL;
                this.resultsContainer = document.getElementById('api-results');
            }
            
            async fetchUsers() {
                try {
                    this.showLoading('Загрузка пользователей...');
                    
                    const response = await fetch(`${this.baseURL}/users`);
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка! статус: ${response.status}`);
                    }
                    
                    const users = await response.json();
                    this.displayUsers(users);
                    
                } catch (error) {
                    this.showError(`Ошибка загрузки пользователей: ${error.message}`);
                }
            }
            
            async createUser(userData) {
                try {
                    this.showLoading('Создание пользователя...');
                    
                    const response = await fetch(`${this.baseURL}/users`, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify({
                            name: userData.name || 'Новый пользователь',
                            email: userData.email || 'new@example.com',
                            username: userData.username || 'newuser'
                        })
                    });
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка! статус: ${response.status}`);
                    }
                    
                    const newUser = await response.json();
                    this.showSuccess(`Пользователь создан: ${newUser.name}`);
                    
                } catch (error) {
                    this.showError(`Ошибка создания пользователя: ${error.message}`);
                }
            }
            
            async updateUser(userId, userData) {
                try {
                    const response = await fetch(`${this.baseURL}/users/${userId}`, {
                        method: 'PUT',
                        headers: {
                            'Content-Type': 'application/json'
                        },
                        body: JSON.stringify(userData)
                    });
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка! статус: ${response.status}`);
                    }
                    
                    const updatedUser = await response.json();
                    this.showSuccess(`Пользователь обновлен: ${updatedUser.name}`);
                    
                    return updatedUser;
                } catch (error) {
                    this.showError(`Ошибка обновления пользователя: ${error.message}`);
                    throw error;
                }
            }
            
            async deleteUser(userId) {
                try {
                    const response = await fetch(`${this.baseURL}/users/${userId}`, {
                        method: 'DELETE'
                    });
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка! статус: ${response.status}`);
                    }
                    
                    this.showSuccess(`Пользователь с ID ${userId} удален`);
                    
                } catch (error) {
                    this.showError(`Ошибка удаления пользователя: ${error.message}`);
                    throw error;
                }
            }
            
            displayUsers(users) {
                this.resultsContainer.innerHTML = `
                    <div class="users-list">
                        <h2>Список пользователей (${users.length})</h2>
                        ${users.map(user => `
                            <div class="user-card" data-user-id="${user.id}">
                                <h3>${user.name}</h3>
                                <p><strong>Имя пользователя:</strong> ${user.username}</p>
                                <p><strong>Email:</strong> ${user.email}</p>
                                <p><strong>Телефон:</strong> ${user.phone}</p>
                                <p><strong>Сайт:</strong> <a href="http://${user.website}" target="_blank">${user.website}</a></p>
                                <div class="user-actions">
                                    <button onclick="api.updateUser(${user.id}, {name: 'Обновленное имя'})">Обновить</button>
                                    <button onclick="api.deleteUser(${user.id})">Удалить</button>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            }
            
            showLoading(message) {
                this.resultsContainer.innerHTML = `
                    <div class="loading">
                        <div class="spinner"></div>
                        <p>${message}</p>
                    </div>
                `;
            }
            
            showSuccess(message) {
                this.resultsContainer.innerHTML = `
                    <div class="success-message">
                        <p>✓ ${message}</p>
                    </div>
                `;
            }
            
            showError(message) {
                this.resultsContainer.innerHTML = `
                    <div class="error-message">
                        <p>✗ ${message}</p>
                    </div>
                `;
            }
            
            clearResults() {
                this.resultsContainer.innerHTML = '';
            }
        }
        
        // Инициализация
        const api = new APIIntegration();
        
        // Глобальные функции для кнопок
        function fetchUsers() {
            api.fetchUsers();
        }
        
        function createUser() {
            api.createUser({
                name: 'Иван Иванов',
                email: 'ivan@example.com',
                username: 'ivanivanov'
            });
        }
        
        function clearResults() {
            api.clearResults();
        }
    </script>
    
    <style>
        .api-controls {
            margin: 20px 0;
        }
        
        .api-controls button {
            padding: 10px 20px;
            margin-right: 10px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .users-list {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        
        .user-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            background-color: #f9f9f9;
        }
        
        .user-actions {
            margin-top: 15px;
            display: flex;
            gap: 10px;
        }
        
        .user-actions button {
            padding: 5px 10px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .loading {
            text-align: center;
            padding: 40px;
        }
        
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #007acc;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 15px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .success-message {
            background-color: #d4edda;
            color: #155724;
            padding: 15px;
            border-radius: 4px;
            margin: 10px 0;
        }
        
        .error-message {
            background-color: #f8d7da;
            color: #721c24;
            padding: 15px;
            border-radius: 4px;
            margin: 10px 0;
        }
    </style>
</body>
</html>
```

### Axios для более продвинутых сценариев

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интеграция с API через Axios</title>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
</head>
<body>
    <h1>Интеграция с API через Axios</h1>
    
    <div class="api-controls">
        <button onclick="fetchPosts()">Загрузить посты</button>
        <button onclick="createPost()">Создать пост</button>
        <button onclick="searchPosts()">Поиск постов</button>
    </div>
    
    <div class="search-controls">
        <input type="text" id="search-input" placeholder="Поиск постов...">
        <button onclick="searchPosts()">Найти</button>
    </div>
    
    <div id="posts-container"></div>

    <script>
        class AxiosAPIIntegration {
            constructor(baseURL = 'https://jsonplaceholder.typicode.com') {
                this.baseURL = baseURL;
                this.postsContainer = document.getElementById('posts-container');
                
                // Настройка axios
                this.api = axios.create({
                    baseURL: this.baseURL,
                    timeout: 10000,
                    headers: {
                        'Content-Type': 'application/json',
                        'X-Requested-With': 'XMLHttpRequest'
                    }
                });
                
                // Перехватчики запросов и ответов
                this.setupInterceptors();
            }
            
            setupInterceptors() {
                // Перехватчик запросов
                this.api.interceptors.request.use(
                    (config) => {
                        console.log('Отправка запроса:', config.method, config.url);
                        this.showLoading('Отправка запроса...');
                        return config;
                    },
                    (error) => {
                        console.error('Ошибка запроса:', error);
                        return Promise.reject(error);
                    }
                );
                
                // Перехватчик ответов
                this.api.interceptors.response.use(
                    (response) => {
                        console.log('Получен ответ:', response.status, response.config.url);
                        this.hideLoading();
                        return response;
                    },
                    (error) => {
                        console.error('Ошибка ответа:', error);
                        this.hideLoading();
                        this.showError(`Ошибка API: ${error.message}`);
                        return Promise.reject(error);
                    }
                );
            }
            
            async fetchPosts() {
                try {
                    const response = await this.api.get('/posts');
                    this.displayPosts(response.data);
                } catch (error) {
                    this.showError(`Ошибка загрузки постов: ${error.message}`);
                }
            }
            
            async createPost(postData) {
                try {
                    const response = await this.api.post('/posts', {
                        title: postData.title || 'Новый пост',
                        body: postData.body || 'Содержимое нового поста',
                        userId: postData.userId || 1
                    });
                    
                    this.showSuccess(`Пост создан: ${response.data.title}`);
                    this.addPostToDisplay(response.data);
                    
                } catch (error) {
                    this.showError(`Ошибка создания поста: ${error.message}`);
                }
            }
            
            async updatePost(postId, postData) {
                try {
                    const response = await this.api.put(`/posts/${postId}`, postData);
                    this.showSuccess(`Пост обновлен: ${response.data.title}`);
                    this.updatePostInDisplay(response.data);
                    
                    return response.data;
                } catch (error) {
                    this.showError(`Ошибка обновления поста: ${error.message}`);
                    throw error;
                }
            }
            
            async deletePost(postId) {
                try {
                    await this.api.delete(`/posts/${postId}`);
                    this.showSuccess(`Пост с ID ${postId} удален`);
                    this.removePostFromDisplay(postId);
                } catch (error) {
                    this.showError(`Ошибка удаления поста: ${error.message}`);
                    throw error;
                }
            }
            
            async searchPosts(query) {
                try {
                    const response = await this.api.get('/posts', {
                        params: { q: query }
                    });
                    
                    this.displayPosts(response.data);
                } catch (error) {
                    this.showError(`Ошибка поиска: ${error.message}`);
                }
            }
            
            displayPosts(posts) {
                this.postsContainer.innerHTML = `
                    <div class="posts-list">
                        <h2>Посты (${posts.length})</h2>
                        ${posts.map(post => `
                            <div class="post-card" data-post-id="${post.id}">
                                <h3>${post.title}</h3>
                                <p class="post-body">${post.body}</p>
                                <div class="post-meta">
                                    <span>Пользователь: ${post.userId}</span>
                                    <span>ID: ${post.id}</span>
                                </div>
                                <div class="post-actions">
                                    <button onclick="api.updatePost(${post.id}, {title: 'Обновленный заголовок'})">Редактировать</button>
                                    <button onclick="api.deletePost(${post.id})">Удалить</button>
                                </div>
                            </div>
                        `).join('')}
                    </div>
                `;
            }
            
            addPostToDisplay(post) {
                const postElement = document.createElement('div');
                postElement.className = 'post-card';
                postElement.dataset.postId = post.id;
                postElement.innerHTML = `
                    <h3>${post.title}</h3>
                    <p class="post-body">${post.body}</p>
                    <div class="post-meta">
                        <span>Пользователь: ${post.userId}</span>
                        <span>ID: ${post.id}</span>
                    </div>
                    <div class="post-actions">
                        <button onclick="api.updatePost(${post.id}, {title: 'Обновленный заголовок'})">Редактировать</button>
                        <button onclick="api.deletePost(${post.id})">Удалить</button>
                    </div>
                `;
                
                this.postsContainer.insertBefore(postElement, this.postsContainer.firstChild);
            }
            
            updatePostInDisplay(updatedPost) {
                const postElement = document.querySelector(`[data-post-id="${updatedPost.id}"]`);
                if (postElement) {
                    postElement.querySelector('h3').textContent = updatedPost.title;
                    postElement.querySelector('.post-body').textContent = updatedPost.body;
                }
            }
            
            removePostFromDisplay(postId) {
                const postElement = document.querySelector(`[data-post-id="${postId}"]`);
                if (postElement) {
                    postElement.remove();
                }
            }
            
            showLoading(message) {
                this.postsContainer.innerHTML = `
                    <div class="loading">
                        <div class="spinner"></div>
                        <p>${message}</p>
                    </div>
                `;
            }
            
            hideLoading() {
                const loadingElement = this.postsContainer.querySelector('.loading');
                if (loadingElement) {
                    loadingElement.remove();
                }
            }
            
            showSuccess(message) {
                const successDiv = document.createElement('div');
                successDiv.className = 'success-message';
                successDiv.textContent = `✓ ${message}`;
                this.postsContainer.prepend(successDiv);
                
                setTimeout(() => {
                    if (successDiv.parentNode) {
                        successDiv.remove();
                    }
                }, 3000);
            }
            
            showError(message) {
                const errorDiv = document.createElement('div');
                errorDiv.className = 'error-message';
                errorDiv.textContent = `✗ ${message}`;
                this.postsContainer.prepend(errorDiv);
                
                setTimeout(() => {
                    if (errorDiv.parentNode) {
                        errorDiv.remove();
                    }
                }, 5000);
            }
        }
        
        // Инициализация
        const api = new AxiosAPIIntegration();
        
        // Глобальные функции
        function fetchPosts() {
            api.fetchPosts();
        }
        
        function createPost() {
            api.createPost({
                title: 'Мой новый пост',
                body: 'Содержимое моего нового поста...',
                userId: 1
            });
        }
        
        function searchPosts() {
            const query = document.getElementById('search-input').value;
            api.searchPosts(query);
        }
    </script>
    
    <style>
        .api-controls {
            margin: 20px 0;
        }
        
        .search-controls {
            margin: 20px 0;
            display: flex;
            gap: 10px;
        }
        
        .search-controls input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .search-controls button {
            padding: 10px 20px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .posts-list {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        
        .post-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            background-color: white;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        
        .post-meta {
            display: flex;
            justify-content: space-between;
            font-size: 0.8em;
            color: #666;
            margin: 10px 0;
        }
        
        .post-actions {
            margin-top: 15px;
            display: flex;
            gap: 10px;
        }
        
        .post-actions button {
            padding: 8px 16px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .post-actions button:first-child {
            background-color: #007acc;
            color: white;
        }
        
        .post-actions button:last-child {
            background-color: #dc3545;
            color: white;
        }
        
        .loading {
            text-align: center;
            padding: 40px;
        }
        
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #007acc;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 15px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .success-message {
            background-color: #d4edda;
            color: #155724;
            padding: 10px;
            border-radius: 4px;
            margin: 10px 0;
            text-align: center;
        }
        
        .error-message {
            background-color: #f8d7da;
            color: #721c24;
            padding: 10px;
            border-radius: 4px;
            margin: 10px 0;
            text-align: center;
        }
    </style>
</body>
</html>
```

## WebSocket интеграция

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>WebSocket интеграция</title>
</head>
<body>
    <h1>WebSocket интеграция</h1>
    
    <div class="websocket-controls">
        <button id="connect-btn">Подключиться</button>
        <button id="disconnect-btn" disabled>Отключиться</button>
        <button id="send-btn" disabled>Отправить сообщение</button>
    </div>
    
    <div class="message-controls">
        <input type="text" id="message-input" placeholder="Введите сообщение..." disabled>
        <select id="message-type" disabled>
            <option value="text">Текстовое сообщение</option>
            <option value="data">Данные</option>
            <option value="command">Команда</option>
        </select>
    </div>
    
    <div id="status-container"></div>
    <div id="messages-container"></div>

    <script>
        class WebSocketIntegration {
            constructor(url) {
                this.url = url;
                this.ws = null;
                this.isConnected = false;
                
                this.statusContainer = document.getElementById('status-container');
                this.messagesContainer = document.getElementById('messages-container');
                this.messageInput = document.getElementById('message-input');
                this.messageType = document.getElementById('message-type');
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('connect-btn').addEventListener('click', () => {
                    this.connect();
                });
                
                document.getElementById('disconnect-btn').addEventListener('click', () => {
                    this.disconnect();
                });
                
                document.getElementById('send-btn').addEventListener('click', () => {
                    this.sendMessage();
                });
                
                this.messageInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter') {
                        this.sendMessage();
                    }
                });
            }
            
            connect() {
                try {
                    this.ws = new WebSocket(this.url);
                    
                    this.ws.onopen = (event) => {
                        this.isConnected = true;
                        this.updateConnectionStatus('Подключено', 'success');
                        this.enableControls(true);
                        this.addMessage('Соединение установлено', 'system');
                    };
                    
                    this.ws.onmessage = (event) => {
                        this.handleMessage(event.data);
                    };
                    
                    this.ws.onclose = (event) => {
                        this.isConnected = false;
                        this.updateConnectionStatus('Отключено', 'warning');
                        this.enableControls(false);
                        this.addMessage('Соединение закрыто', 'system');
                    };
                    
                    this.ws.onerror = (error) => {
                        this.updateConnectionStatus('Ошибка соединения', 'error');
                        this.addMessage(`Ошибка: ${error.message}`, 'error');
                    };
                } catch (error) {
                    this.updateConnectionStatus(`Ошибка подключения: ${error.message}`, 'error');
                }
            }
            
            disconnect() {
                if (this.ws) {
                    this.ws.close();
                }
            }
            
            sendMessage() {
                if (!this.isConnected || !this.messageInput.value.trim()) return;
                
                const message = {
                    type: this.messageType.value,
                    content: this.messageInput.value.trim(),
                    timestamp: new Date().toISOString(),
                    userId: this.getUserId()
                };
                
                this.ws.send(JSON.stringify(message));
                this.addMessage(`Вы: ${message.content}`, 'outgoing');
                this.messageInput.value = '';
            }
            
            handleMessage(data) {
                try {
                    const message = JSON.parse(data);
                    
                    let displayMessage = '';
                    if (message.type === 'text') {
                        displayMessage = message.content;
                    } else if (message.type === 'data') {
                        displayMessage = `Данные: ${JSON.stringify(message.content)}`;
                    } else if (message.type === 'command') {
                        displayMessage = `Команда: ${message.content}`;
                        this.executeCommand(message.content);
                    }
                    
                    this.addMessage(displayMessage, 'incoming');
                } catch (error) {
                    this.addMessage(data, 'incoming');
                }
            }
            
            executeCommand(command) {
                // Обработка команд от сервера
                switch(command) {
                    case 'refresh':
                        location.reload();
                        break;
                    case 'clear':
                        this.messagesContainer.innerHTML = '';
                        break;
                    default:
                        console.log('Неизвестная команда:', command);
                }
            }
            
            addMessage(content, type) {
                const messageDiv = document.createElement('div');
                messageDiv.className = `message ${type}`;
                
                const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                
                messageDiv.innerHTML = `
                    <div class="message-content">${content}</div>
                    <div class="message-time">${time}</div>
                `;
                
                this.messagesContainer.appendChild(messageDiv);
                
                // Прокрутка к последнему сообщению
                this.messagesContainer.scrollTop = this.messagesContainer.scrollHeight;
            }
            
            updateConnectionStatus(status, type) {
                this.statusContainer.innerHTML = `
                    <div class="connection-status ${type}">
                        Статус: ${status}
                    </div>
                `;
            }
            
            enableControls(enabled) {
                document.getElementById('connect-btn').disabled = enabled;
                document.getElementById('disconnect-btn').disabled = !enabled;
                document.getElementById('send-btn').disabled = !enabled;
                this.messageInput.disabled = !enabled;
                this.messageType.disabled = !enabled;
            }
            
            getUserId() {
                // В реальном приложении это будет идентификатор пользователя
                return 'user_' + Math.random().toString(36).substr(2, 9);
            }
        }
        
        // Инициализация
        // Примечание: для реального использования нужно указать реальный WebSocket URL
        // const ws = new WebSocketIntegration('ws://localhost:8080/ws');
        
        // Для демонстрации создадим mock-соединение
        class MockWebSocketIntegration {
            constructor(url) {
                this.url = url;
                this.isConnected = false;
                this.setupEventListeners();
                
                // Имитация получения сообщений
                setInterval(() => {
                    if (this.isConnected && Math.random() > 0.7) {
                        const messages = [
                            'Новое сообщение от сервера',
                            'Системное уведомление',
                            'Обновление данных',
                            'Доступна новая версия'
                        ];
                        
                        const randomMessage = messages[Math.floor(Math.random() * messages.length)];
                        this.handleMessage({ type: 'text', content: randomMessage });
                    }
                }, 3000);
            }
            
            connect() {
                this.isConnected = true;
                this.updateConnectionStatus('Подключено (Mock)', 'success');
                this.enableControls(true);
                this.addMessage('Соединение с сервером установлено', 'system');
            }
            
            disconnect() {
                this.isConnected = false;
                this.updateConnectionStatus('Отключено', 'warning');
                this.enableControls(false);
                this.addMessage('Соединение с сервером закрыто', 'system');
            }
            
            sendMessage() {
                if (!this.isConnected || !this.messageInput.value.trim()) return;
                
                const message = this.messageInput.value.trim();
                this.addMessage(`Вы: ${message}`, 'outgoing');
                
                // Имитация отправки сообщения
                setTimeout(() => {
                    this.addMessage(`Сервер: Принято сообщение "${message}"`, 'incoming');
                }, 500);
                
                this.messageInput.value = '';
            }
            
            handleMessage(message) {
                if (typeof message === 'object') {
                    this.addMessage(message.content, 'incoming');
                } else {
                    this.addMessage(message, 'incoming');
                }
            }
            
            addMessage(content, type) {
                const messageDiv = document.createElement('div');
                messageDiv.className = `message ${type}`;
                
                const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                
                messageDiv.innerHTML = `
                    <div class="message-content">${content}</div>
                    <div class="message-time">${time}</div>
                `;
                
                this.messagesContainer.appendChild(messageDiv);
                this.messagesContainer.scrollTop = this.messagesContainer.scrollHeight;
            }
            
            updateConnectionStatus(status, type) {
                this.statusContainer.innerHTML = `
                    <div class="connection-status ${type}">
                        Статус: ${status}
                    </div>
                `;
            }
            
            enableControls(enabled) {
                document.getElementById('connect-btn').disabled = enabled;
                document.getElementById('disconnect-btn').disabled = !enabled;
                document.getElementById('send-btn').disabled = !enabled;
                this.messageInput.disabled = !enabled;
                this.messageType.disabled = !enabled;
            }
        }
        
        // Инициализация mock-версии
        const ws = new MockWebSocketIntegration('mock://localhost:8080/ws');
    </script>
    
    <style>
        .websocket-controls {
            margin: 20px 0;
        }
        
        .websocket-controls button {
            padding: 10px 20px;
            margin-right: 10px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        #connect-btn {
            background-color: #28a745;
            color: white;
        }
        
        #disconnect-btn {
            background-color: #dc3545;
            color: white;
        }
        
        #send-btn {
            background-color: #007acc;
            color: white;
        }
        
        .message-controls {
            margin: 20px 0;
            display: flex;
            gap: 10px;
        }
        
        .message-controls input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .message-controls select {
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        #status-container {
            margin: 10px 0;
        }
        
        .connection-status {
            padding: 10px;
            border-radius: 4px;
            font-weight: bold;
        }
        
        .connection-status.success {
            background-color: #d4edda;
            color: #155724;
        }
        
        .connection-status.warning {
            background-color: #fff3cd;
            color: #856404;
        }
        
        .connection-status.error {
            background-color: #f8d7da;
            color: #721c24;
        }
        
        #messages-container {
            height: 400px;
            border: 1px solid #ddd;
            border-radius: 4px;
            padding: 15px;
            overflow-y: auto;
            background-color: #f9f9f9;
        }
        
        .message {
            margin-bottom: 10px;
            padding: 8px;
            border-radius: 4px;
        }
        
        .message.system {
            background-color: #e3f2fd;
            color: #1976d2;
        }
        
        .message.incoming {
            background-color: #e8f5e8;
            color: #2e7d32;
        }
        
        .message.outgoing {
            background-color: #e3f2fd;
            color: #1976d2;
            margin-left: 50px;
        }
        
        .message-time {
            font-size: 0.7em;
            color: #666;
            text-align: right;
        }
    </style>
</body>
</html>
```

## Интеграция с REST API

### Продвинутая работа с API
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Продвинутая интеграция с REST API</title>
</head>
<body>
    <h1>Продвинутая интеграция с REST API</h1>
    
    <div class="api-client-container">
        <div class="api-controls">
            <select id="api-endpoint">
                <option value="/users">Пользователи</option>
                <option value="/posts">Посты</option>
                <option value="/comments">Комментарии</option>
                <option value="/albums">Альбомы</option>
                <option value="/photos">Фото</option>
            </select>
            
            <select id="api-method">
                <option value="GET">GET</option>
                <option value="POST">POST</option>
                <option value="PUT">PUT</option>
                <option value="DELETE">DELETE</option>
            </select>
            
            <input type="text" id="api-params" placeholder="Параметры (JSON)">
            <button id="execute-request">Выполнить запрос</button>
        </div>
        
        <div class="request-builder">
            <h3>Построитель запросов</h3>
            <textarea id="request-body" placeholder="Тело запроса (JSON)" rows="6"></textarea>
        </div>
        
        <div class="response-container">
            <h3>Ответ</h3>
            <div id="response-content"></div>
        </div>
    </div>

    <script>
        class AdvancedAPIIntegration {
            constructor(baseURL = 'https://jsonplaceholder.typicode.com') {
                this.baseURL = baseURL;
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('execute-request').addEventListener('click', () => {
                    this.executeRequest();
                });
                
                document.getElementById('api-endpoint').addEventListener('change', (e) => {
                    this.updateRequestBody(e.target.value);
                });
            }
            
            async executeRequest() {
                const endpoint = document.getElementById('api-endpoint').value;
                const method = document.getElementById('api-method').value;
                const params = document.getElementById('api-params').value;
                const body = document.getElementById('request-body').value;
                
                const url = this.baseURL + endpoint + (params ? '?' + new URLSearchParams(JSON.parse(params)) : '');
                
                try {
                    this.showLoading('Выполнение запроса...');
                    
                    const response = await fetch(url, {
                        method: method,
                        headers: {
                            'Content-Type': 'application/json',
                            'Accept': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: method !== 'GET' && body ? body : undefined
                    });
                    
                    const responseData = await response.json();
                    
                    this.displayResponse(response, responseData);
                } catch (error) {
                    this.displayError(error.message);
                }
            }
            
            updateRequestBody(endpoint) {
                const requestBody = document.getElementById('request-body');
                
                switch(endpoint) {
                    case '/users':
                        if (document.getElementById('api-method').value === 'POST') {
                            requestBody.value = JSON.stringify({
                                name: "Иван Иванов",
                                username: "ivan_ivanov",
                                email: "ivan@example.com"
                            }, null, 2);
                        }
                        break;
                        
                    case '/posts':
                        if (document.getElementById('api-method').value === 'POST') {
                            requestBody.value = JSON.stringify({
                                title: "Заголовок поста",
                                body: "Содержимое поста",
                                userId: 1
                            }, null, 2);
                        }
                        break;
                        
                    case '/comments':
                        if (document.getElementById('api-method').value === 'POST') {
                            requestBody.value = JSON.stringify({
                                postId: 1,
                                name: "Комментатор",
                                email: "comment@example.com",
                                body: "Текст комментария"
                            }, null, 2);
                        }
                        break;
                        
                    default:
                        requestBody.value = '';
                }
            }
            
            displayResponse(response, data) {
                const responseContainer = document.getElementById('response-content');
                
                responseContainer.innerHTML = `
                    <div class="response-info">
                        <div class="response-status">Статус: ${response.status} ${response.statusText}</div>
                        <div class="response-headers">Заголовки: ${JSON.stringify([...response.headers], null, 2)}</div>
                    </div>
                    <div class="response-body">
                        <pre>${JSON.stringify(data, null, 2)}</pre>
                    </div>
                `;
            }
            
            displayError(error) {
                const responseContainer = document.getElementById('response-content');
                responseContainer.innerHTML = `
                    <div class="error-response">
                        <h4>Ошибка:</h4>
                        <p>${error}</p>
                    </div>
                `;
            }
            
            showLoading(message) {
                const responseContainer = document.getElementById('response-content');
                responseContainer.innerHTML = `
                    <div class="loading-response">
                        <div class="spinner"></div>
                        <p>${message}</p>
                    </div>
                `;
            }
            
            // Методы для работы с конкретными сущностями
            async getUsers() {
                return this.makeRequest('/users', 'GET');
            }
            
            async getUser(id) {
                return this.makeRequest(`/users/${id}`, 'GET');
            }
            
            async createUser(userData) {
                return this.makeRequest('/users', 'POST', userData);
            }
            
            async updateUser(id, userData) {
                return this.makeRequest(`/users/${id}`, 'PUT', userData);
            }
            
            async deleteUser(id) {
                return this.makeRequest(`/users/${id}`, 'DELETE');
            }
            
            async makeRequest(endpoint, method = 'GET', data = null) {
                const config = {
                    method: method,
                    headers: {
                        'Content-Type': 'application/json',
                        'Accept': 'application/json'
                    }
                };
                
                if (data && method !== 'GET') {
                    config.body = JSON.stringify(data);
                }
                
                const response = await fetch(this.baseURL + endpoint, config);
                
                if (!response.ok) {
                    throw new Error(`HTTP ошибка! статус: ${response.status}`);
                }
                
                return response.json();
            }
        }
        
        // Инициализация
        const apiClient = new AdvancedAPIIntegration();
        
        // Пример использования методов для работы с пользователями
        async function exampleUsage() {
            try {
                // Получить всех пользователей
                const users = await apiClient.getUsers();
                console.log('Пользователи:', users);
                
                // Создать нового пользователя
                const newUser = await apiClient.createUser({
                    name: 'Новый пользователь',
                    email: 'newuser@example.com',
                    username: 'newuser'
                });
                console.log('Созданный пользователь:', newUser);
                
                // Обновить пользователя
                const updatedUser = await apiClient.updateUser(newUser.id, {
                    ...newUser,
                    name: 'Обновленное имя'
                });
                console.log('Обновленный пользователь:', updatedUser);
            } catch (error) {
                console.error('Ошибка при работе с API:', error);
            }
        }
    </script>
    
    <style>
        .api-client-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .api-controls {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }
        
        .api-controls select,
        .api-controls input {
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .api-controls button {
            padding: 8px 16px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .request-builder {
            margin-bottom: 20px;
        }
        
        #request-body {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-family: monospace;
            font-size: 14px;
        }
        
        .response-container {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            background-color: #f9f9f9;
        }
        
        .response-info {
            margin-bottom: 15px;
            padding: 10px;
            background-color: #e3f2fd;
            border-radius: 4px;
        }
        
        .response-body pre {
            background-color: #f5f5f5;
            padding: 15px;
            border-radius: 4px;
            overflow-x: auto;
            white-space: pre-wrap;
        }
        
        .error-response {
            color: #d32f2f;
            padding: 15px;
            background-color: #ffebee;
            border-radius: 4px;
        }
        
        .loading-response {
            text-align: center;
            padding: 40px;
        }
        
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #007acc;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 15px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</body>
</html>
```

## Безопасная интеграция с API

### CORS и безопасность

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасная интеграция с API</title>
</head>
<body>
    <h1>Безопасная интеграция с API</h1>
    
    <div class="security-info">
        <h2>Информация о безопасности</h2>
        <ul>
            <li>Использование HTTPS для всех API запросов</li>
            <li>Валидация и санитизация данных</li>
            <li>Проверка CORS заголовков</li>
            <li>Использование токенов аутентификации</li>
            <li>Ограничение скорости запросов</li>
        </ul>
    </div>
    
    <div class="secure-api-form">
        <h2>Безопасный API клиент</h2>
        
        <form id="secure-api-form">
            <div class="form-group">
                <label for="api-token">Токен доступа:</label>
                <input type="password" id="api-token" name="api_token" required>
            </div>
            
            <div class="form-group">
                <label for="secure-endpoint">Эндпоинт:</label>
                <select id="secure-endpoint" name="endpoint" required>
                    <option value="/users">Пользователи</option>
                    <option value="/posts">Посты</option>
                    <option value="/protected">Защищенные данные</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="secure-method">Метод:</label>
                <select id="secure-method" name="method">
                    <option value="GET">GET</option>
                    <option value="POST">POST</option>
                    <option value="PUT">PUT</option>
                    <option value="DELETE">DELETE</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="secure-body">Тело запроса:</label>
                <textarea id="secure-body" name="body" rows="4"></textarea>
            </div>
            
            <button type="submit">Выполнить запрос</button>
        </form>
        
        <div id="secure-response"></div>
    </div>

    <script>
        class SecureAPIIntegration {
            constructor(baseURL) {
                this.baseURL = baseURL;
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('secure-api-form').addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.secureRequest();
                });
            }
            
            async secureRequest() {
                const token = document.getElementById('api-token').value;
                const endpoint = document.getElementById('secure-endpoint').value;
                const method = document.getElementById('secure-method').value;
                const body = document.getElementById('secure-body').value;
                
                if (!token) {
                    this.showSecureError('Требуется токен доступа');
                    return;
                }
                
                try {
                    this.showLoading('Выполнение безопасного запроса...');
                    
                    const config = {
                        method: method,
                        headers: {
                            'Content-Type': 'application/json',
                            'Accept': 'application/json',
                            'Authorization': `Bearer ${token}`,
                            'X-Requested-With': 'XMLHttpRequest'
                        }
                    };
                    
                    if (method !== 'GET' && body.trim()) {
                        // Санитизация тела запроса
                        const sanitizedBody = this.sanitizeRequestData(JSON.parse(body));
                        config.body = JSON.stringify(sanitizedBody);
                    }
                    
                    const response = await fetch(this.baseURL + endpoint, config);
                    
                    if (response.status === 401) {
                        this.showSecureError('Неавторизованный доступ. Проверьте токен.');
                        return;
                    }
                    
                    if (response.status === 403) {
                        this.showSecureError('Доступ запрещен. Недостаточно прав.');
                        return;
                    }
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка! статус: ${response.status}`);
                    }
                    
                    const data = await response.json();
                    this.displaySecureResponse(data, response.status);
                    
                } catch (error) {
                    this.showSecureError(`Ошибка запроса: ${error.message}`);
                }
            }
            
            sanitizeRequestData(data) {
                // Рекурсивная санитизация данных
                if (typeof data === 'string') {
                    return data
                        .replace(/</g, '&lt;')
                        .replace(/>/g, '&gt;')
                        .replace(/"/g, '&quot;')
                        .replace(/'/g, '&#x27;')
                        .trim();
                } else if (typeof data === 'object' && data !== null) {
                    const sanitized = {};
                    for (const [key, value] of Object.entries(data)) {
                        sanitized[key] = this.sanitizeRequestData(value);
                    }
                    return sanitized;
                }
                return data;
            }
            
            validateResponseData(data) {
                // Проверка данных ответа на наличие вредоносного содержимого
                if (typeof data === 'string') {
                    const dangerousPatterns = [
                        /<script/i,
                        /javascript:/i,
                        /on\w+\s*=/i,
                        /<iframe/i,
                        /<object/i,
                        /<embed/i
                    ];
                    
                    return !dangerousPatterns.some(pattern => pattern.test(data));
                } else if (typeof data === 'object' && data !== null) {
                    return Object.values(data).every(value => this.validateResponseData(value));
                }
                return true;
            }
            
            displaySecureResponse(data, status) {
                if (!this.validateResponseData(data)) {
                    this.showSecureError('Ответ содержит потенциально опасное содержимое');
                    return;
                }
                
                const responseContainer = document.getElementById('secure-response');
                
                responseContainer.innerHTML = `
                    <div class="secure-response">
                        <div class="response-status">Статус: ${status}</div>
                        <div class="response-body">
                            <pre>${JSON.stringify(data, null, 2)}</pre>
                        </div>
                    </div>
                `;
            }
            
            showLoading(message) {
                const responseContainer = document.getElementById('secure-response');
                responseContainer.innerHTML = `
                    <div class="loading-indicator">
                        <div class="spinner"></div>
                        <p>${message}</p>
                    </div>
                `;
            }
            
            showSecureError(message) {
                const responseContainer = document.getElementById('secure-response');
                responseContainer.innerHTML = `
                    <div class="secure-error">
                        <h4>Ошибка безопасности:</h4>
                        <p>${message}</p>
                    </div>
                `;
            }
            
            // Методы для безопасной работы с API
            async secureGet(endpoint, token) {
                return this.secureRequest(endpoint, 'GET', null, token);
            }
            
            async securePost(endpoint, data, token) {
                return this.secureRequest(endpoint, 'POST', data, token);
            }
            
            async securePut(endpoint, data, token) {
                return this.secureRequest(endpoint, 'PUT', data, token);
            }
            
            async secureDelete(endpoint, token) {
                return this.secureRequest(endpoint, 'DELETE', null, token);
            }
            
            async secureRequest(endpoint, method = 'GET', data = null, token) {
                if (!token) {
                    throw new Error('Требуется токен аутентификации');
                }
                
                const config = {
                    method: method,
                    headers: {
                        'Content-Type': 'application/json',
                        'Accept': 'application/json',
                        'Authorization': `Bearer ${token}`
                    }
                };
                
                if (data && method !== 'GET') {
                    config.body = JSON.stringify(this.sanitizeRequestData(data));
                }
                
                const response = await fetch(this.baseURL + endpoint, config);
                
                if (response.status === 401 || response.status === 403) {
                    throw new Error('Неавторизованный доступ');
                }
                
                if (!response.ok) {
                    throw new Error(`HTTP ошибка: ${response.status}`);
                }
                
                const responseData = await response.json();
                
                if (!this.validateResponseData(responseData)) {
                    throw new Error('Ответ содержит потенциально опасное содержимое');
                }
                
                return responseData;
            }
        }
        
        // Инициализация
        const secureApi = new SecureAPIIntegration('https://api.example.com');
    </script>
    
    <style>
        .security-info {
            background-color: #e8f5e8;
            border: 1px solid #c8e6c9;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 20px;
        }
        
        .secure-api-form {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            background-color: white;
        }
        
        .form-group {
            margin-bottom: 15px;
        }
        
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .secure-response {
            margin-top: 20px;
            padding: 15px;
            background-color: #e3f2fd;
            border: 1px solid #bbdefb;
            border-radius: 4px;
        }
        
        .response-status {
            font-weight: bold;
            margin-bottom: 10px;
        }
        
        .response-body pre {
            background-color: #f5f5f5;
            padding: 10px;
            border-radius: 4px;
            overflow-x: auto;
        }
        
        .secure-error {
            margin-top: 20px;
            padding: 15px;
            background-color: #ffebee;
            border: 1px solid #f5c6cb;
            border-radius: 4px;
            color: #721c24;
        }
        
        .loading-indicator {
            text-align: center;
            padding: 20px;
        }
        
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #007acc;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 0 auto 10px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</body>
</html>
```

## Практические примеры интеграции

### Интеграция с GraphQL API
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интеграция с GraphQL API</title>
</head>
<body>
    <h1>Интеграция с GraphQL API</h1>
    
    <div class="graphql-client">
        <h2>GraphQL клиент</h2>
        
        <div class="query-builder">
            <textarea id="graphql-query" rows="8" placeholder="Введите GraphQL запрос...">
query GetUsers {
  users {
    id
    name
    email
    posts {
      id
      title
      content
    }
  }
}
            </textarea>
        </div>
        
        <button onclick="executeGraphQLQuery()">Выполнить запрос</button>
        
        <div id="graphql-response"></div>
    </div>

    <script>
        class GraphQLIntegration {
            constructor(endpoint) {
                this.endpoint = endpoint;
            }
            
            async executeQuery(query, variables = {}) {
                try {
                    const response = await fetch(this.endpoint, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'Accept': 'application/json'
                        },
                        body: JSON.stringify({
                            query: query,
                            variables: variables
                        })
                    });
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка! статус: ${response.status}`);
                    }
                    
                    const result = await response.json();
                    
                    if (result.errors) {
                        throw new Error(result.errors.map(error => error.message).join('\n'));
                    }
                    
                    return result.data;
                } catch (error) {
                    throw new Error(`GraphQL ошибка: ${error.message}`);
                }
            }
            
            async executeMutation(mutation, variables = {}) {
                return this.executeQuery(mutation, variables);
            }
        }
        
        async function executeGraphQLQuery() {
            const query = document.getElementById('graphql-query').value;
            const responseContainer = document.getElementById('graphql-response');
            
            if (!query.trim()) {
                responseContainer.innerHTML = '<div class="error">Введите GraphQL запрос</div>';
                return;
            }
            
            responseContainer.innerHTML = '<div class="loading">Выполнение запроса...</div>';
            
            try {
                // В реальном приложении здесь будет реальный GraphQL endpoint
                const client = new GraphQLIntegration('/graphql');
                const result = await client.executeQuery(query);
                
                responseContainer.innerHTML = `
                    <div class="success-response">
                        <h3>Результат:</h3>
                        <pre>${JSON.stringify(result, null, 2)}</pre>
                    </div>
                `;
            } catch (error) {
                responseContainer.innerHTML = `
                    <div class="error-response">
                        <h3>Ошибка:</h3>
                        <p>${error.message}</p>
                    </div>
                `;
            }
        }
    </script>
    
    <style>
        .graphql-client {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .query-builder textarea {
            width: 100%;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-family: 'Courier New', monospace;
            font-size: 14px;
            line-height: 1.5;
        }
        
        button {
            margin-top: 10px;
            padding: 10px 20px;
            background-color: #e53935;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        
        .success-response {
            margin-top: 20px;
            padding: 15px;
            background-color: #e8f5e8;
            border: 1px solid #c8e6c9;
            border-radius: 4px;
        }
        
        .error-response {
            margin-top: 20px;
            padding: 15px;
            background-color: #ffebee;
            border: 1px solid #f5c6cb;
            border-radius: 4px;
            color: #721c24;
        }
        
        .loading {
            padding: 20px;
            text-align: center;
            color: #666;
        }
        
        pre {
            background-color: #f5f5f5;
            padding: 10px;
            border-radius: 4px;
            overflow-x: auto;
            white-space: pre-wrap;
        }
    </style>
</body>
</html>
```

### Интеграция с REST API и кешированием
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>REST API с кешированием</title>
</head>
<body>
    <h1>REST API с кешированием</h1>
    
    <div class="api-with-cache">
        <h2>API клиент с кешированием</h2>
        
        <div class="controls">
            <button onclick="loadUsers()">Загрузить пользователей</button>
            <button onclick="clearCache()">Очистить кеш</button>
            <button onclick="showCacheInfo()">Информация о кеше</button>
        </div>
        
        <div id="api-results"></div>
        <div id="cache-info"></div>
    </div>

    <script>
        class APICache {
            constructor() {
                this.cache = new Map();
                this.maxCacheSize = 100;
                this.defaultTTL = 5 * 60 * 1000; // 5 минут
            }
            
            async get(key, fetchFn, ttl = this.defaultTTL) {
                const cached = this.cache.get(key);
                
                if (cached && Date.now() - cached.timestamp < ttl) {
                    console.log('Данные получены из кеша:', key);
                    return cached.data;
                }
                
                console.log('Данные загружаются с сервера:', key);
                const data = await fetchFn();
                
                this.set(key, data, ttl);
                return data;
            }
            
            set(key, data, ttl = this.defaultTTL) {
                if (this.cache.size >= this.maxCacheSize) {
                    // Удаляем самые старые записи
                    const oldestKey = this.cache.keys().next().value;
                    this.cache.delete(oldestKey);
                }
                
                this.cache.set(key, {
                    data: data,
                    timestamp: Date.now(),
                    ttl: ttl
                });
            }
            
            has(key) {
                const cached = this.cache.get(key);
                return cached && Date.now() - cached.timestamp < cached.ttl;
            }
            
            delete(key) {
                return this.cache.delete(key);
            }
            
            clear() {
                this.cache.clear();
            }
            
            getStats() {
                const entries = Array.from(this.cache.entries());
                return {
                    size: entries.length,
                    entries: entries.map(([key, value]) => ({
                        key: key,
                        age: Date.now() - value.timestamp,
                        ttl: value.ttl
                    }))
                };
            }
        }
        
        class CachedAPIIntegration {
            constructor(baseURL) {
                this.baseURL = baseURL;
                this.cache = new APICache();
            }
            
            async getUsers() {
                return this.cache.get('/users', async () => {
                    const response = await fetch(`${this.baseURL}/users`);
                    if (!response.ok) throw new Error(`HTTP ошибка: ${response.status}`);
                    return response.json();
                });
            }
            
            async getUser(id) {
                return this.cache.get(`/users/${id}`, async () => {
                    const response = await fetch(`${this.baseURL}/users/${id}`);
                    if (!response.ok) throw new Error(`HTTP ошибка: ${response.status}`);
                    return response.json();
                });
            }
            
            async getPosts(userId = null) {
                const endpoint = userId ? `/users/${userId}/posts` : '/posts';
                return this.cache.get(endpoint, async () => {
                    const response = await fetch(`${this.baseURL}${endpoint}`);
                    if (!response.ok) throw new Error(`HTTP ошибка: ${response.status}`);
                    return response.json();
                });
            }
        }
        
        // Инициализация
        const cachedAPI = new CachedAPIIntegration('https://jsonplaceholder.typicode.com');
        
        async function loadUsers() {
            const resultsContainer = document.getElementById('api-results');
            resultsContainer.innerHTML = '<div class="loading">Загрузка пользователей...</div>';
            
            try {
                const users = await cachedAPI.getUsers();
                
                resultsContainer.innerHTML = `
                    <div class="users-list">
                        <h3>Пользователи (${users.length})</h3>
                        ${users.map(user => `
                            <div class="user-card" onclick="loadUserPosts(${user.id})">
                                <h4>${user.name}</h4>
                                <p>${user.email}</p>
                            </div>
                        `).join('')}
                    </div>
                `;
            } catch (error) {
                resultsContainer.innerHTML = `<div class="error">Ошибка: ${error.message}</div>`;
            }
        }
        
        async function loadUserPosts(userId) {
            const resultsContainer = document.getElementById('api-results');
            resultsContainer.innerHTML = '<div class="loading">Загрузка постов...</div>';
            
            try {
                const posts = await cachedAPI.getPosts(userId);
                
                resultsContainer.innerHTML = `
                    <div class="posts-list">
                        <h3>Посты пользователя (ID: ${userId})</h3>
                        ${posts.map(post => `
                            <div class="post-card">
                                <h4>${post.title}</h4>
                                <p>${post.body}</p>
                            </div>
                        `).join('')}
                    </div>
                `;
            } catch (error) {
                resultsContainer.innerHTML = `<div class="error">Ошибка: ${error.message}</div>`;
            }
        }
        
        function clearCache() {
            cachedAPI.cache.clear();
            document.getElementById('cache-info').innerHTML = '<div class="info">Кеш очищен</div>';
        }
        
        function showCacheInfo() {
            const stats = cachedAPI.cache.getStats();
            const infoContainer = document.getElementById('cache-info');
            
            infoContainer.innerHTML = `
                <div class="cache-stats">
                    <h3>Статистика кеша</h3>
                    <p>Размер: ${stats.size} записей</p>
                    <ul>
                        ${stats.entries.map(entry => `
                            <li>
                                <strong>${entry.key}</strong> - 
                                возраст: ${(entry.age / 1000).toFixed(1)}с, 
                                TTL: ${(entry.ttl / 1000).toFixed(1)}с
                            </li>
                        `).join('')}
                    </ul>
                </div>
            `;
        }
    </script>
    
    <style>
        .api-with-cache {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .controls {
            margin-bottom: 20px;
        }
        
        .controls button {
            margin-right: 10px;
            padding: 8px 16px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .users-list, .posts-list {
            display: grid;
            gap: 15px;
        }
        
        .user-card, .post-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        
        .user-card:hover, .post-card:hover {
            background-color: #f5f5f5;
        }
        
        .loading {
            text-align: center;
            padding: 40px;
            color: #666;
        }
        
        .error {
            background-color: #ffebee;
            color: #c62828;
            padding: 15px;
            border-radius: 4px;
            border: 1px solid #f5c6cb;
        }
        
        .info {
            background-color: #d1ecf1;
            color: #0c5460;
            padding: 15px;
            border-radius: 4px;
            border: 1px solid #bee5eb;
        }
        
        .cache-stats {
            background-color: #f8f9fa;
            border: 1px solid #dee2e6;
            border-radius: 4px;
            padding: 15px;
            margin-top: 20px;
        }
        
        .cache-stats ul {
            margin: 10px 0;
            padding-left: 20px;
        }
        
        .cache-stats li {
            margin-bottom: 5px;
        }
    </style>
</body>
</html>
```

## Современные возможности безопасности API интеграции

### Content Security Policy для API запросов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Content Security Policy для API</title>
    
    <!-- Строгая CSP политика для API интеграции -->
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'unsafe-inline' https://trusted-cdn.com;
                   style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
                   img-src 'self' data: https:;
                   font-src 'self' https://fonts.gstatic.com;
                   connect-src 'self' https://api.trusted.com https://*.api.trusted.com;
                   frame-ancestors 'none';
                   object-src 'none';
                   base-uri 'self';
                   form-action 'self';
                   upgrade-insecure-requests;">
</head>
<body>
    <h1>Безопасная интеграция с API через CSP</h1>
    
    <div class="api-security-demo">
        <form id="secure-api-form">
            <div class="form-group">
                <label for="api-endpoint">Эндпоинт:</label>
                <select id="api-endpoint" name="endpoint">
                    <option value="/users">Пользователи</option>
                    <option value="/posts">Посты</option>
                    <option value="/comments">Комментарии</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="api-method">Метод:</label>
                <select id="api-method" name="method">
                    <option value="GET">GET</option>
                    <option value="POST">POST</option>
                    <option value="PUT">PUT</option>
                    <option value="DELETE">DELETE</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="api-body">Тело запроса (JSON):</label>
                <textarea id="api-body" name="body" rows="4" placeholder='{ "key": "value" }'></textarea>
            </div>
            
            <button type="submit">Выполнить безопасный запрос</button>
        </form>
        
        <div id="api-response" aria-live="polite"></div>
    </div>

    <script>
        class SecureAPIIntegration {
            constructor() {
                this.form = document.getElementById('secure-api-form');
                this.responseContainer = document.getElementById('api-response');
                
                this.allowedOrigins = [
                    'https://api.trusted.com',
                    'https://secure-api.example.com'
                ];
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.secureAPIRequest();
                });
            }
            
            async secureAPIRequest() {
                const endpoint = document.getElementById('api-endpoint').value;
                const method = document.getElementById('api-method').value;
                const body = document.getElementById('api-body').value;
                
                // Проверка безопасности эндпоинта
                if (!this.isValidEndpoint(endpoint)) {
                    this.showSecurityError('Недопустимый эндпоинт API');
                    return;
                }
                
                // Проверка безопасности метода
                if (!this.isValidMethod(method)) {
                    this.showSecurityError('Недопустимый HTTP метод');
                    return;
                }
                
                // Санитизация тела запроса
                let sanitizedBody = null;
                if (body.trim()) {
                    try {
                        sanitizedBody = this.sanitizeJSON(body);
                    } catch (error) {
                        this.showSecurityError('Неверный формат JSON в теле запроса');
                        return;
                    }
                }
                
                try {
                    this.showLoading('Выполнение безопасного запроса...');
                    
                    const response = await fetch(endpoint, {
                        method: method,
                        headers: {
                            'Content-Type': 'application/json',
                            'Accept': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest',
                            'X-Client-Security': 'csp-protected'
                        },
                        body: sanitizedBody ? JSON.stringify(sanitizedBody) : undefined
                    });
                    
                    if (!response.ok) {
                        throw new Error(`HTTP ошибка: ${response.status} ${response.statusText}`);
                    }
                    
                    const data = await response.json();
                    
                    // Проверка ответа на безопасность
                    if (this.containsMaliciousContent(data)) {
                        throw new Error('Ответ содержит потенциально опасный контент');
                    }
                    
                    this.displaySecureResponse(data);
                    
                } catch (error) {
                    this.showSecurityError(`Ошибка API: ${error.message}`);
                }
            }
            
            isValidEndpoint(endpoint) {
                // Проверка, что эндпоинт начинается с разрешенного префикса
                return this.allowedOrigins.some(origin => 
                    endpoint.startsWith(origin) || 
                    endpoint.startsWith('/')); // Относительные пути разрешены
            }
            
            isValidMethod(method) {
                return ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'].includes(method.toUpperCase());
            }
            
            sanitizeJSON(jsonString) {
                const parsed = JSON.parse(jsonString);
                return this.sanitizeObject(parsed);
            }
            
            sanitizeObject(obj) {
                if (obj === null || typeof obj !== 'object') {
                    return obj;
                }
                
                if (Array.isArray(obj)) {
                    return obj.map(item => this.sanitizeObject(item));
                }
                
                const sanitized = {};
                for (const [key, value] of Object.entries(obj)) {
                    sanitized[key] = typeof value === 'string' ? this.sanitizeString(value) : this.sanitizeObject(value);
                }
                return sanitized;
            }
            
            sanitizeString(str) {
                return str
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/\//g, '&#x2F;')
                    .trim();
            }
            
            containsMaliciousContent(data) {
                const str = JSON.stringify(data);
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                return dangerousPatterns.some(pattern => pattern.test(str));
            }
            
            showLoading(message) {
                this.responseContainer.innerHTML = `
                    <div class="loading-indicator" role="status" aria-live="polite">
                        <div class="spinner"></div>
                        <p>${message}</p>
                    </div>
                `;
            }
            
            showSecurityError(message) {
                this.responseContainer.innerHTML = `
                    <div class="security-error" role="alert" aria-live="assertive">
                        <h3>Ошибка безопасности:</h3>
                        <p>${message}</p>
                    </div>
                `;
            }
            
            displaySecureResponse(data) {
                this.responseContainer.innerHTML = `
                    <div class="secure-response">
                        <h3>Безопасный ответ от API:</h3>
                        <pre>${JSON.stringify(data, null, 2)}</pre>
                    </div>
                `;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new SecureAPIIntegration();
        });
    </script>
    
    <style>
        .api-security-demo {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .form-group {
            margin-bottom: 1.5rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        select, textarea {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .loading-indicator {
            text-align: center;
            padding: 2rem;
        }
        
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #007acc;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 1rem;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .security-error {
            background-color: #f8d7da;
            color: #721c24;
            padding: 1rem;
            border: 1px solid #f5c6cb;
            border-radius: 4px;
        }
        
        .secure-response {
            background-color: #d4edda;
            color: #155724;
            padding: 1rem;
            border: 1px solid #c3e6cb;
            border-radius: 4px;
        }
        
        pre {
            background-color: #f8f9fa;
            padding: 1rem;
            border-radius: 4px;
            overflow-x: auto;
            margin-top: 1rem;
        }
    </style>
</body>
</html>
```

### Безопасная работа с JWT токенами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасная работа с JWT</title>
</head>
<body>
    <h1>Безопасная работа с JWT токенами</h1>
    
    <div class="jwt-security-container">
        <form id="jwt-auth-form">
            <div class="form-group">
                <label for="jwt-username">Имя пользователя:</label>
                <input type="text" id="jwt-username" name="username" required>
            </div>
            
            <div class="form-group">
                <label for="jwt-password">Пароль:</label>
                <input type="password" id="jwt-password" name="password" required>
            </div>
            
            <button type="submit">Войти</button>
        </form>
        
        <div class="jwt-actions" style="display: none;">
            <h3>Действия с токеном</h3>
            <button onclick="refreshToken()">Обновить токен</button>
            <button onclick="logout()">Выйти</button>
            <button onclick="showTokenInfo()">Информация о токене</button>
        </div>
        
        <div id="jwt-status" aria-live="polite"></div>
        <div id="jwt-content" style="display: none;"></div>
    </div>

    <script>
        class JWTSecurityManager {
            constructor() {
                this.authForm = document.getElementById('jwt-auth-form');
                this.jwtActions = document.querySelector('.jwt-actions');
                this.statusContainer = document.getElementById('jwt-status');
                this.contentContainer = document.getElementById('jwt-content');
                
                this.token = null;
                this.refreshToken = null;
                this.tokenExpiry = null;
                
                this.setupEventListeners();
                this.checkExistingToken();
            }
            
            setupEventListeners() {
                this.authForm.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.authenticate();
                });
            }
            
            async authenticate() {
                const username = document.getElementById('jwt-username').value;
                const password = document.getElementById('jwt-password').value;
                
                try {
                    this.showStatus('Аутентификация...', 'loading');
                    
                    // В реальном приложении это будет запрос к API
                    const response = await fetch('/api/auth/login', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify({ username, password })
                    });
                    
                    if (!response.ok) {
                        throw new Error('Неверные учетные данные');
                    }
                    
                    const authData = await response.json();
                    
                    if (authData.token && authData.refreshToken) {
                        this.storeTokens(authData.token, authData.refreshToken, authData.expiresIn);
                        this.showStatus('Успешная аутентификация', 'success');
                        this.showJWTActions();
                        this.loadProtectedContent();
                    }
                    
                } catch (error) {
                    this.showStatus(`Ошибка аутентификации: ${error.message}`, 'error');
                }
            }
            
            storeTokens(token, refreshToken, expiresIn) {
                // Проверка безопасности токена
                if (!this.isValidJWT(token)) {
                    throw new Error('Невалидный JWT токен');
                }
                
                // Хранение токена в сессии (не в localStorage для чувствительных данных)
                sessionStorage.setItem('authToken', token);
                localStorage.setItem('refreshToken', refreshToken); // Может быть в secure cookie
                
                this.token = token;
                this.refreshToken = refreshToken;
                this.tokenExpiry = Date.now() + (expiresIn * 1000);
                
                // Установка таймера для автоматического обновления
                this.scheduleTokenRefresh(expiresIn);
            }
            
            isValidJWT(token) {
                try {
                    const parts = token.split('.');
                    if (parts.length !== 3) return false;
                    
                    const header = JSON.parse(atob(parts[0]));
                    const payload = JSON.parse(atob(parts[1]));
                    
                    // Проверка алгоритма (не должен быть 'none')
                    if (header.alg === 'none' || header.alg.toLowerCase() === 'none') {
                        console.warn('Обнаружен небезопасный алгоритм JWT: none');
                        return false;
                    }
                    
                    // Проверка времени истечения
                    if (payload.exp && Date.now() >= payload.exp * 1000) {
                        return false;
                    }
                    
                    // Проверка времени начала действия
                    if (payload.nbf && Date.now() < payload.nbf * 1000) {
                        return false;
                    }
                    
                    return true;
                } catch (error) {
                    console.error('Ошибка проверки JWT:', error);
                    return false;
                }
            }
            
            scheduleTokenRefresh(expiresIn) {
                // Обновление токена за 5 минут до истечения
                const refreshTime = (expiresIn - 300) * 1000; // 5 минут = 300 секунд
                
                if (refreshTime > 0) {
                    setTimeout(() => {
                        this.refreshTokenAutomatically();
                    }, refreshTime);
                }
            }
            
            async refreshTokenAutomatically() {
                if (!this.refreshToken) return;
                
                try {
                    const response = await fetch('/api/auth/refresh', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'Authorization': `Bearer ${this.refreshToken}`
                        }
                    });
                    
                    if (response.ok) {
                        const data = await response.json();
                        this.storeTokens(data.token, data.refreshToken, data.expiresIn);
                        this.showStatus('Токен обновлен автоматически', 'success');
                    } else {
                        this.logout();
                        this.showStatus('Сессия истекла, пожалуйста, войдите снова', 'warning');
                    }
                } catch (error) {
                    console.error('Ошибка автоматического обновления токена:', error);
                }
            }
            
            async loadProtectedContent() {
                if (!this.token) {
                    this.showStatus('Требуется аутентификация', 'error');
                    return;
                }
                
                try {
                    const response = await fetch('/api/protected-content', {
                        headers: {
                            'Authorization': `Bearer ${this.token}`,
                            'Content-Type': 'application/json'
                        }
                    });
                    
                    if (!response.ok) {
                        if (response.status === 401) {
                            // Токен истек, попробуем обновить
                            await this.refreshToken();
                            return this.loadProtectedContent();
                        }
                        throw new Error('Ошибка загрузки защищенного контента');
                    }
                    
                    const content = await response.json();
                    this.displayProtectedContent(content);
                    
                } catch (error) {
                    this.showStatus(`Ошибка загрузки контента: ${error.message}`, 'error');
                }
            }
            
            displayProtectedContent(content) {
                this.contentContainer.innerHTML = `
                    <div class="protected-content">
                        <h3>Защищенный контент:</h3>
                        <pre>${JSON.stringify(content, null, 2)}</pre>
                    </div>
                `;
                this.contentContainer.style.display = 'block';
            }
            
            showJWTActions() {
                this.jwtActions.style.display = 'block';
            }
            
            async refreshToken() {
                if (!this.refreshToken) {
                    this.showStatus('Отсутствует refresh токен', 'error');
                    return;
                }
                
                try {
                    const response = await fetch('/api/auth/refresh', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'Authorization': `Bearer ${this.refreshToken}`
                        }
                    });
                    
                    if (response.ok) {
                        const data = await response.json();
                        this.storeTokens(data.token, data.refreshToken, data.expiresIn);
                        this.showStatus('Токен успешно обновлен', 'success');
                    } else {
                        throw new Error('Не удалось обновить токен');
                    }
                } catch (error) {
                    this.showStatus(`Ошибка обновления токена: ${error.message}`, 'error');
                    this.logout();
                }
            }
            
            logout() {
                sessionStorage.removeItem('authToken');
                localStorage.removeItem('refreshToken');
                
                this.token = null;
                this.refreshToken = null;
                this.tokenExpiry = null;
                
                this.jwtActions.style.display = 'none';
                this.contentContainer.style.display = 'none';
                this.showStatus('Вы успешно вышли из системы', 'info');
            }
            
            checkExistingToken() {
                const token = sessionStorage.getItem('authToken');
                if (token && this.isValidJWT(token)) {
                    this.token = token;
                    this.showJWTActions();
                    this.loadProtectedContent();
                }
            }
            
            showStatus(message, type) {
                const statusElement = document.getElementById('jwt-status');
                statusElement.innerHTML = `<div class="status-${type}">${message}</div>`;
            }
        }
        
        // Инициализация
        const jwtManager = new JWTSecurityManager();
        
        // Глобальные функции для кнопок
        function refreshToken() {
            jwtManager.refreshToken();
        }
        
        function logout() {
            jwtManager.logout();
        }
        
        function showTokenInfo() {
            if (jwtManager.token) {
                const payload = jwtManager.decodeToken(jwtManager.token);
                alert(`Информация о токене:\n${JSON.stringify(payload, null, 2)}`);
            } else {
                alert('Токен не найден');
            }
        }
    </script>
    
    <style>
        .jwt-security-container {
            max-width: 500px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .form-group {
            margin-bottom: 1.5rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin-right: 10px;
        }
        
        .jwt-actions {
            margin-top: 20px;
        }
        
        .status-loading { color: #007acc; }
        .status-success { color: #28a745; }
        .status-error { color: #dc3545; }
        .status-warning { color: #ffc107; }
        .status-info { color: #17a2b8; }
        
        .protected-content {
            margin-top: 20px;
            padding: 15px;
            background-color: #f8f9fa;
            border-radius: 8px;
        }
        
        pre {
            background-color: #f5f5f5;
            padding: 10px;
            border-radius: 4px;
            overflow-x: auto;
        }
    </style>
</body>
</html>
```

### Безопасные WebSockets

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Безопасные WebSockets</title>
</head>
<body>
    <h1>Безопасные WebSocket соединения</h1>
    
    <div class="websocket-security-container">
        <div class="connection-controls">
            <button id="connect-btn">Подключиться</button>
            <button id="disconnect-btn" disabled>Отключиться</button>
            <span id="connection-status">Отключено</span>
        </div>
        
        <div class="message-controls">
            <input type="text" id="message-input" placeholder="Введите сообщение..." disabled>
            <button id="send-btn" disabled>Отправить</button>
        </div>
        
        <div id="messages-container" class="messages-container"></div>
    </div>

    <script>
        class SecureWebSocketClient {
            constructor(url) {
                this.url = url;
                this.ws = null;
                this.isConnected = false;
                this.connectionAttempts = 0;
                this.maxConnectionAttempts = 3;
                this.connectionRetryDelay = 3000;
                
                this.setupSecureConnection();
            }
            
            setupSecureConnection() {
                this.connectBtn = document.getElementById('connect-btn');
                this.disconnectBtn = document.getElementById('disconnect-btn');
                this.connectionStatus = document.getElementById('connection-status');
                this.messageInput = document.getElementById('message-input');
                this.sendBtn = document.getElementById('send-btn');
                this.messagesContainer = document.getElementById('messages-container');
                
                this.connectBtn.addEventListener('click', () => this.connectSecurely());
                this.disconnectBtn.addEventListener('click', () => this.disconnect());
                this.sendBtn.addEventListener('click', () => this.sendMessage());
                
                this.messageInput.addEventListener('keypress', (e) => {
                    if (e.key === 'Enter' && !this.messageInput.disabled) {
                        this.sendMessage();
                    }
                });
            }
            
            connectSecurely() {
                if (this.connectionAttempts >= this.maxConnectionAttempts) {
                    this.showSecurityError('Превышено максимальное количество попыток подключения');
                    return;
                }
                
                try {
                    // Проверка безопасности URL
                    if (!this.isValidSecureWebSocketURL(this.url)) {
                        throw new Error('Небезопасный WebSocket URL');
                    }
                    
                    this.ws = new WebSocket(this.url);
                    
                    this.ws.onopen = () => {
                        this.isConnected = true;
                        this.connectionAttempts = 0;
                        this.updateConnectionStatus('Подключено', 'connected');
                        this.enableInputs();
                    };
                    
                    this.ws.onmessage = (event) => {
                        this.handleSecureMessage(event.data);
                    };
                    
                    this.ws.onclose = (event) => {
                        this.isConnected = false;
                        this.updateConnectionStatus('Отключено', 'disconnected');
                        this.disableInputs();
                        
                        // Автоматическое переподключение
                        if (event.code !== 1000) { // Нормальное закрытие
                            this.connectionAttempts++;
                            setTimeout(() => {
                                if (this.connectionAttempts < this.maxConnectionAttempts) {
                                    this.connectSecurely();
                                }
                            }, this.connectionRetryDelay);
                        }
                    };
                    
                    this.ws.onerror = (error) => {
                        console.error('WebSocket ошибка:', error);
                        this.updateConnectionStatus('Ошибка', 'error');
                        this.disableInputs();
                    };
                    
                } catch (error) {
                    console.error('Ошибка подключения:', error);
                    this.showSecurityError(`Ошибка подключения: ${error.message}`);
                }
            }
            
            isValidSecureWebSocketURL(url) {
                try {
                    const parsedUrl = new URL(url);
                    
                    // Проверка протокола (только wss для безопасности)
                    if (parsedUrl.protocol !== 'wss:' && !this.isLocalhost(url)) {
                        console.warn('Используйте wss:// для безопасного WebSocket соединения');
                        return false;
                    }
                    
                    // Проверка домена
                    const allowedDomains = [
                        'our-secure-domain.com',
                        'api.our-app.com',
                        'localhost',
                        '127.0.0.1'
                    ];
                    
                    if (!this.isLocalhost(url)) {
                        return allowedDomains.some(domain => 
                            parsedUrl.hostname === domain || 
                            parsedUrl.hostname.endsWith('.' + domain)
                        );
                    }
                    
                    return true;
                } catch {
                    return false;
                }
            }
            
            isLocalhost(url) {
                try {
                    const parsedUrl = new URL(url);
                    return ['localhost', '127.0.0.1', '[::1]'].includes(parsedUrl.hostname);
                } catch {
                    return false;
                }
            }
            
            handleSecureMessage(data) {
                try {
                    // Проверка целостности сообщения
                    const message = JSON.parse(data);
                    
                    if (!this.validateMessageStructure(message)) {
                        console.error('Невалидная структура сообщения:', message);
                        return;
                    }
                    
                    // Санитизация содержимого сообщения
                    const sanitizedMessage = this.sanitizeMessage(message);
                    
                    this.displayMessage(sanitizedMessage);
                    
                } catch (error) {
                    console.error('Ошибка обработки сообщения:', error);
                    this.showSecurityError('Получено недопустимое сообщение');
                }
            }
            
            validateMessageStructure(message) {
                // Проверка обязательных полей
                const requiredFields = ['id', 'timestamp', 'content'];
                return requiredFields.every(field => field in message);
            }
            
            sanitizeMessage(message) {
                // Санитизация содержимого сообщения
                return {
                    ...message,
                    content: this.sanitizeContent(message.content),
                    sender: this.sanitizeContent(message.sender || 'Unknown')
                };
            }
            
            sanitizeContent(content) {
                if (typeof content !== 'string') return content;
                
                return content
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .trim();
            }
            
            sendMessage() {
                if (!this.isConnected || !this.messageInput.value.trim()) return;
                
                const message = {
                    content: this.messageInput.value.trim(),
                    timestamp: new Date().toISOString(),
                    userId: this.getUserId()
                };
                
                // Проверка безопасности перед отправкой
                if (!this.isMessageSecure(message)) {
                    this.showSecurityError('Сообщение содержит небезопасный контент');
                    return;
                }
                
                try {
                    this.ws.send(JSON.stringify(message));
                    this.messageInput.value = '';
                    
                    // Отображение собственного сообщения
                    this.displayMessage({ ...message, isOwn: true });
                    
                } catch (error) {
                    console.error('Ошибка отправки сообщения:', error);
                    this.showSecurityError('Ошибка отправки сообщения');
                }
            }
            
            isMessageSecure(message) {
                // Проверка на потенциально опасный контент
                const dangerousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i
                ];
                
                const content = message.content || '';
                return !dangerousPatterns.some(pattern => pattern.test(content));
            }
            
            displayMessage(message) {
                const messageElement = document.createElement('div');
                messageElement.className = `message ${message.isOwn ? 'own' : 'received'}`;
                
                const time = new Date(message.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                
                messageElement.innerHTML = `
                    <div class="message-header">
                        <span class="message-sender">${this.escapeHTML(message.sender || 'Система')}</span>
                        <span class="message-time">${time}</span>
                    </div>
                    <div class="message-content">${this.escapeHTML(message.content)}</div>
                `;
                
                this.messagesContainer.appendChild(messageElement);
                
                // Прокрутка к последнему сообщению
                this.messagesContainer.scrollTop = this.messagesContainer.scrollHeight;
            }
            
            updateConnectionStatus(status, type) {
                this.connectionStatus.textContent = status;
                this.connectionStatus.className = `connection-status ${type}`;
            }
            
            enableInputs() {
                this.messageInput.disabled = false;
                this.sendBtn.disabled = false;
                this.connectBtn.disabled = true;
                this.disconnectBtn.disabled = false;
            }
            
            disableInputs() {
                this.messageInput.disabled = true;
                this.sendBtn.disabled = true;
                this.connectBtn.disabled = false;
                this.disconnectBtn.disabled = true;
            }
            
            disconnect() {
                if (this.ws) {
                    this.ws.close(1000, 'Нормальное закрытие');
                }
            }
            
            showSecurityError(message) {
                const errorDiv = document.createElement('div');
                errorDiv.className = 'security-error';
                errorDiv.textContent = message;
                
                this.messagesContainer.prepend(errorDiv);
                
                setTimeout(() => {
                    if (errorDiv.parentNode) {
                        errorDiv.remove();
                    }
                }, 5000);
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
            
            getUserId() {
                // В реальном приложении ID пользователя должен быть получен из сессии
                return sessionStorage.getItem('userId') || 'anonymous';
            }
        }
        
        // Инициализация (используем безопасный URL)
        const secureWS = new SecureWebSocketClient('wss://secure-chat.example.com/ws');
    </script>
    
    <style>
        .websocket-security-container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .connection-controls, .message-controls {
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .connection-controls button, .message-controls button {
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        #connect-btn {
            background-color: #28a745;
            color: white;
        }
        
        #disconnect-btn {
            background-color: #dc3545;
            color: white;
        }
        
        #send-btn {
            background-color: #007acc;
            color: white;
        }
        
        .connection-status {
            padding: 0.5rem 1rem;
            border-radius: 4px;
            font-weight: bold;
        }
        
        .connection-status.connected {
            background-color: #d4edda;
            color: #155724;
        }
        
        .connection-status.disconnected {
            background-color: #f8d7da;
            color: #721c24;
        }
        
        .connection-status.error {
            background-color: #f8d7da;
            color: #721c24;
        }
        
        #message-input {
            flex: 1;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        #messages-container {
            height: 400px;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            overflow-y: auto;
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
        
        .message.received {
            background-color: #f1f8e9;
        }
        
        .message-header {
            display: flex;
            justify-content: space-between;
            font-size: 0.9em;
            margin-bottom: 5px;
            color: #666;
        }
        
        .message-sender {
            font-weight: bold;
        }
        
        .message-content {
            margin: 0;
            line-height: 1.5;
        }
        
        .security-error {
            background-color: #f8d7da;
            color: #721c24;
            padding: 10px;
            border-radius: 4px;
            margin: 10px 0;
            text-align: center;
        }
    </style>
</body>
</html>
```

## Практические примеры и шаблоны

### Шаблон безопасной формы с многоуровневой защитой
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy" 
          content="default-src 'self';
                   script-src 'self' 'nonce-abc123';
                   style-src 'self' 'unsafe-inline';
                   img-src 'self' data: https:;
                   connect-src 'self' https://api.secure.com;">
    <title>Безопасная форма с многоуровневой защитой</title>
</head>
<body>
    <h1>Безопасная форма с многоуровневой защитой</h1>
    
    <form id="multi-layer-form" novalidate>
        <!-- Уровень 1: CSRF защита -->
        <input type="hidden" name="csrf_token" value="GENERATED_CSRF_TOKEN">
        
        <!-- Уровень 2: Валидация на клиенте -->
        <div class="form-group">
            <label for="secure-username">Имя пользователя:</label>
            <input type="text"
                   id="secure-username"
                   name="username"
                   required
                   minlength="3"
                   maxlength="30"
                   pattern="[A-Za-z0-9_]+"
                   autocomplete="username"
                   aria-describedby="username-help username-error">
            <div id="username-help" class="help-text">Только буквы, цифры и подчеркивание</div>
            <div id="username-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-email">Email:</label>
            <input type="email"
                   id="secure-email"
                   name="email"
                   required
                   autocomplete="email"
                   aria-describedby="email-help email-error">
            <div id="email-help" class="help-text">Введите действительный email адрес</div>
            <div id="email-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-password">Пароль:</label>
            <input type="password"
                   id="secure-password"
                   name="password"
                   required
                   minlength="8"
                   aria-describedby="password-requirements password-error">
            <ul id="password-requirements" class="requirements-list">
                <li id="req-length">Не менее 8 символов</li>
                <li id="req-upper">С заглавной буквой</li>
                <li id="req-lower">Со строчной буквой</li>
                <li id="req-number">С цифрой</li>
                <li id="req-special">Со специальным символом</li>
            </ul>
            <div id="password-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-confirm">Подтверждение пароля:</label>
            <input type="password"
                   id="secure-confirm"
                   name="confirm_password"
                   required
                   aria-describedby="confirm-error">
            <div id="confirm-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label for="secure-bio">О себе:</label>
            <textarea id="secure-bio"
                      name="bio"
                      rows="4"
                      maxlength="500"
                      aria-describedby="bio-help bio-error"></textarea>
            <div id="bio-help" class="help-text">Краткое описание (максимум 500 символов)</div>
            <div id="bio-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox"
                       name="terms"
                       required
                       aria-describedby="terms-error">
                Я согласен с <a href="/terms" target="_blank">условиями использования</a>
            </label>
            <div id="terms-error" class="error-message" role="alert" aria-live="assertive"></div>
        </div>
        
        <button type="submit">Зарегистрироваться</button>
    </form>

    <script nonce="abc123">
        class MultiLayerSecurityForm {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.securityLayers = [];
                this.setupSecurityLayers();
                this.setupValidation();
            }
            
            setupSecurityLayers() {
                // Уровень 1: Защита от XSS
                this.securityLayers.push(new XSSProtectionLayer());
                
                // Уровень 2: Защита от CSRF
                this.securityLayers.push(new CSRFProtectionLayer());
                
                // Уровень 3: Защита от brute force
                this.securityLayers.push(new RateLimitingLayer());
                
                // Уровень 4: Проверка целостности данных
                this.securityLayers.push(new DataIntegrityLayer());
            }
            
            setupValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
                });
                
                // Валидация в реальном времени
                this.form.addEventListener('input', (e) => {
                    if (e.target.matches('input[required], textarea[required]')) {
                        this.validateField(e.target);
                    }
                });
            }
            
            async validateAndSubmit() {
                // Очистка предыдущих ошибок
                this.clearAllErrors();
                
                let isValid = true;
                
                // Проверка всех полей
                const fields = this.form.querySelectorAll('input[required], textarea[required]');
                for (const field of fields) {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                }
                
                // Проверка пароля
                if (!this.validatePasswordRequirements()) {
                    isValid = false;
                }
                
                if (!this.validatePasswordMatch()) {
                    isValid = false;
                }
                
                if (!this.validateTerms()) {
                    isValid = false;
                }
                
                // Проверка всех уровней безопасности
                for (const layer of this.securityLayers) {
                    if (!await layer.validate(this.form)) {
                        isValid = false;
                    }
                }
                
                if (isValid) {
                    await this.submitSecurely();
                }
            }
            
            validateField(field) {
                const fieldName = field.name;
                const errorElement = document.getElementById(`${fieldName.replace(/-/g, '')}-error`);
                
                // Сброс предыдущего состояния
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                if (errorElement) errorElement.textContent = '';
                
                // Проверки
                if (field.hasAttribute('required') && !field.value.trim()) {
                    this.showError(field, errorElement, 'Это поле обязательно для заполнения');
                    return false;
                }
                
                if (field.type === 'email' && field.value && !this.isValidEmail(field.value)) {
                    this.showError(field, errorElement, 'Введите действительный email адрес');
                    return false;
                }
                
                if (field.minLength && field.value.length < field.minLength) {
                    this.showError(field, errorElement, `Минимум ${field.minLength} символов`);
                    return false;
                }
                
                if (field.maxLength && field.value.length > field.maxLength) {
                    this.showError(field, errorElement, `Максимум ${field.maxLength} символов`);
                    return false;
                }
                
                if (field.pattern && field.value && !new RegExp(field.pattern).test(field.value)) {
                    this.showError(field, errorElement, 'Неверный формат данных');
                    return false;
                }
                
                // Проверка на опасный контент
                if (this.containsMaliciousContent(field.value)) {
                    this.showError(field, errorElement, 'Поле содержит недопустимый контент');
                    return false;
                }
                
                field.classList.add('valid');
                return true;
            }
            
            validatePasswordRequirements() {
                const password = document.getElementById('secure-password').value;
                const requirements = {
                    length: password.length >= 8,
                    upper: /[A-Z]/.test(password),
                    lower: /[a-z]/.test(password),
                    number: /[0-9]/.test(password),
                    special: /[!@#$%^&*(),.?":{}|<>]/.test(password)
                };
                
                // Обновление индикаторов требований
                Object.entries(requirements).forEach(([key, met]) => {
                    const element = document.getElementById(`req-${key}`);
                    if (element) {
                        element.classList.toggle('met', met);
                    }
                });
                
                const allMet = Object.values(requirements).every(req => req);
                
                if (!allMet && password) {
                    const errorElement = document.getElementById('password-error');
                    this.showError(document.getElementById('secure-password'), errorElement, 'Пароль не соответствует требованиям');
                    return false;
                }
                
                return allMet;
            }
            
            validatePasswordMatch() {
                const password = document.getElementById('secure-password').value;
                const confirmPassword = document.getElementById('secure-confirm').value;
                const errorElement = document.getElementById('confirm-error');
                
                if (confirmPassword && password !== confirmPassword) {
                    this.showError(document.getElementById('secure-confirm'), errorElement, 'Пароли не совпадают');
                    return false;
                }
                
                if (errorElement) errorElement.textContent = '';
                return true;
            }
            
            validateTerms() {
                const termsCheckbox = this.form.querySelector('input[name="terms"]');
                const errorElement = document.getElementById('terms-error');
                
                if (!termsCheckbox.checked) {
                    this.showError(termsCheckbox, errorElement, 'Необходимо согласие с условиями');
                    return false;
                }
                
                if (errorElement) errorElement.textContent = '';
                return true;
            }
            
            showError(field, errorElement, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                
                if (errorElement) {
                    errorElement.textContent = message;
                }
                
                // Фокус на поле с ошибкой
                field.focus();
            }
            
            clearAllErrors() {
                this.form.querySelectorAll('.error-message').forEach(el => el.textContent = '');
                this.form.querySelectorAll('input, textarea').forEach(field => {
                    field.classList.remove('invalid', 'valid');
                    field.setAttribute('aria-invalid', 'false');
                });
            }
            
            async submitSecurely() {
                const submitBtn = this.form.querySelector('button[type="submit"]');
                const originalText = submitBtn.textContent;
                
                // Показываем состояние загрузки
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    // Сбор и санитизация данных
                    const formData = new FormData(this.form);
                    const sanitizedData = {};
                    
                    for (const [key, value] of formData.entries()) {
                        if (key !== 'csrf_token') {
                            sanitizedData[key] = this.sanitizeInput(value);
                        }
                    }
                    
                    // Добавление дополнительных данных безопасности
                    sanitizedData.clientSecurity = {
                        timestamp: Date.now(),
                        userAgent: navigator.userAgent,
                        language: navigator.language
                    };
                    
                    const response = await fetch('/api/secure-register', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-CSRF-Token': formData.get('csrf_token'),
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify(sanitizedData)
                    });
                    
                    if (response.ok) {
                        alert('Регистрация прошла успешно!');
                        this.form.reset();
                    } else {
                        const error = await response.json();
                        alert('Ошибка регистрации: ' + error.message);
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
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
            
            isValidEmail(email) {
                const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
                return re.test(email);
            }
            
            containsMaliciousContent(content) {
                if (typeof content !== 'string') return false;
                
                const maliciousPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i,
                    /eval\(/i,
                    /execScript/i
                ];
                
                return maliciousPatterns.some(pattern => pattern.test(content));
            }
        }
        
        // Классы уровней безопасности
        class XSSProtectionLayer {
            async validate(form) {
                const inputs = form.querySelectorAll('input, textarea');
                for (const input of inputs) {
                    if (this.containsXSS(input.value)) {
                        throw new Error('Обнаружен потенциальный XSS');
                    }
                }
                return true;
            }
            
            containsXSS(value) {
                if (typeof value !== 'string') return false;
                
                const xssPatterns = [
                    /<script/i,
                    /javascript:/i,
                    /on\w+\s*=/i,
                    /<iframe/i,
                    /<object/i,
                    /<embed/i
                ];
                
                return xssPatterns.some(pattern => pattern.test(value));
            }
        }
        
        class CSRFProtectionLayer {
            async validate(form) {
                const token = form.querySelector('input[name="csrf_token"]').value;
                if (!token || token.length < 32) {
                    throw new Error('Неверный CSRF токен');
                }
                return true;
            }
        }
        
        class RateLimitingLayer {
            constructor() {
                this.attempts = 0;
                this.lastAttempt = 0;
                this.maxAttempts = 5;
                this.cooldown = 60000; // 1 минута
            }
            
            async validate(form) {
                const now = Date.now();
                
                if (now - this.lastAttempt < this.cooldown) {
                    if (this.attempts >= this.maxAttempts) {
                        throw new Error('Превышено количество попыток. Попробуйте позже.');
                    }
                } else {
                    this.attempts = 0;
                }
                
                this.attempts++;
                this.lastAttempt = now;
                
                return true;
            }
        }
        
        class DataIntegrityLayer {
            async validate(form) {
                // Проверка целостности данных
                const formData = new FormData(form);
                
                for (const [key, value] of formData.entries()) {
                    if (typeof value === 'string' && this.hasDataIntegrityIssues(value)) {
                        throw new Error(`Нарушена целостность данных в поле ${key}`);
                    }
                }
                
                return true;
            }
            
            hasDataIntegrityIssues(value) {
                // Проверка на подозрительные паттерны
                return /(?:\b(?:exec|select|insert|update|delete|drop|create|alter|grant|revoke)\b)/i.test(value);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new MultiLayerSecurityForm('multi-layer-form');
        });
    </script>
    
    <style>
        .form-group {
            margin-bottom: 1.5rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input, textarea {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        input:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
        }
        
        input.valid {
            border-color: #28a745;
        }
        
        input.invalid {
            border-color: #dc3545;
        }
        
        .help-text {
            color: #666;
            font-size: 0.875rem;
            margin-top: 0.25rem;
        }
        
        .error-message {
            color: #dc3545;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }
        
        .requirements-list {
            list-style: none;
            padding: 0;
            margin: 0.5rem 0 0;
        }
        
        .requirements-list li {
            padding: 0.25rem 0;
            font-size: 0.875rem;
        }
        
        .requirements-list li.met {
            color: #28a745;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }
        
        button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
    </style>
</body>
</html>
```

## Заключение

Безопасность HTML-форм и интеграции с API является критически важным аспектом современной веб-разработки. Правильное применение мер безопасности, таких как Content Security Policy, Subresource Integrity, CSRF токены, XSS защита и другие, помогает защитить как пользователей, так и веб-приложения от потенциальных угроз.

Ключевые аспекты безопасности HTML:
1. Использование Content Security Policy для предотвращения XSS
2. Применение Subresource Integrity для проверки целостности ресурсов
3. Защита от CSRF с помощью токенов
4. Санитизация пользовательского ввода
5. Правильная обработка и отображение данных
6. Использование безопасных протоколов (HTTPS, WSS)
7. Защита от clickjacking и других атак
8. Многоуровневая проверка и валидация данных
9. Совместимость с вспомогательными технологиями
10. Следование современным веб-стандартам безопасности

Эти практики обеспечивают комплексную защиту веб-приложений и создают безопасную среду для пользователей.

Современные подходы к безопасности включают:
- Использование Content Security Policy для ограничения источников контента
- Subresource Integrity для проверки целостности внешних ресурсов
- Безопасную обработку пользовательского ввода
- Защиту от различных типов атак (XSS, CSRF, clickjacking и др.)
- Использование безопасных протоколов и методов передачи данных
- Многоуровневую проверку и валидацию
- Совместимость с современными веб-стандартами безопасности
- Регулярный аудит и тестирование безопасности

Эти методы позволяют создавать защищенные веб-приложения, которые обеспечивают безопасное взаимодействие пользователей с серверными ресурсами и защищают конфиденциальные данные.

## Следующие темы
- [[HTML/Производительность]]
- [[HTML/Доступность]]
- [[HTML/Совместимость]]

## Теги
#html #security #web-development #frontend #xss #csrf #csp #sri #clickjacking #web-components #javascript #dom-manipulation #input-sanitization #content-security-policy #subresource-integrity #authentication #authorization #secure-coding #best-practices #security-headers #security-auditing #security-monitoring
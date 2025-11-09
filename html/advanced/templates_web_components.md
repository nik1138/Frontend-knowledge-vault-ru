# HTML Продвинутые темы: Шаблоны и веб-компоненты

HTML продолжает развиваться, и современные веб-стандарты включают в себя продвинутые возможности, такие как HTML шаблоны и веб-компоненты. Эти технологии позволяют создавать более структурированный, переиспользуемый и модульный код.

## HTML Шаблоны с `<template>`

Элемент `<template>` позволяет создавать инертные шаблоны HTML, которые не отображаются на странице до тех пор, пока не будут активированы с помощью JavaScript.

### Основы использования `<template>`

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>HTML Шаблоны</title>
</head>
<body>
    <h1>Список пользователей</h1>
    <div id="users-container"></div>
    
    <!-- Шаблон для карточки пользователя -->
    <template id="user-card-template">
        <div class="user-card">
            <img class="user-avatar" src="" alt="Аватар пользователя">
            <h3 class="user-name"></h3>
            <p class="user-email"></p>
            <button class="user-contact">Связаться</button>
        </div>
    </template>
    
    <script>
        // Данные пользователей
        const users = [
            { name: "Иван Иванов", email: "ivan@example.com", avatar: "avatar1.jpg" },
            { name: "Мария Петрова", email: "maria@example.com", avatar: "avatar2.jpg" },
            { name: "Алексей Сидоров", email: "alexey@example.com", avatar: "avatar3.jpg" }
        ];
        
        // Функция для создания карточки пользователя
        function createUserCard(user) {
            const template = document.getElementById('user-card-template');
            const clone = template.content.cloneNode(true);
            
            clone.querySelector('.user-name').textContent = user.name;
            clone.querySelector('.user-email').textContent = user.email;
            clone.querySelector('.user-avatar').src = user.avatar;
            clone.querySelector('.user-avatar').alt = `Аватар ${user.name}`;
            
            // Добавить обработчик события
            clone.querySelector('.user-contact').addEventListener('click', () => {
                alert(`Связаться с ${user.name}: ${user.email}`);
            });
            
            return clone;
        }
        
        // Добавить карточки пользователей на страницу
        const container = document.getElementById('users-container');
        users.forEach(user => {
            const card = createUserCard(user);
            container.appendChild(card);
        });
    </script>
</body>
</html>
```

### Пример сложного шаблона

```html
<template id="product-template">
    <article class="product-card">
        <div class="product-image-container">
            <img class="product-image" src="" alt="">
            <div class="product-badge"></div>
        </div>
        <div class="product-info">
            <h3 class="product-title"></h3>
            <div class="product-rating">
                <span class="rating-stars"></span>
                <span class="rating-count"></span>
            </div>
            <p class="product-description"></p>
            <div class="product-price-container">
                <span class="product-price"></span>
                <span class="product-old-price"></span>
            </div>
            <div class="product-actions">
                <button class="add-to-cart">Добавить в корзину</button>
                <button class="wishlist">В избранное</button>
            </div>
        </div>
    </article>
</template>

<script>
function createProductCard(product) {
    const template = document.getElementById('product-template');
    const clone = template.content.cloneNode(true);
    
    // Заполнение данными
    const img = clone.querySelector('.product-image');
    img.src = product.image;
    img.alt = product.title;
    
    clone.querySelector('.product-title').textContent = product.title;
    clone.querySelector('.product-description').textContent = product.description;
    clone.querySelector('.product-price').textContent = formatPrice(product.price);
    
    if (product.oldPrice) {
        const oldPrice = clone.querySelector('.product-old-price');
        oldPrice.textContent = formatPrice(product.oldPrice);
        oldPrice.style.textDecoration = 'line-through';
    }
    
    // Установка рейтинга
    const stars = clone.querySelector('.rating-stars');
    stars.innerHTML = '★'.repeat(Math.floor(product.rating)) + '☆'.repeat(5 - Math.floor(product.rating));
    
    // Добавление обработчиков
    clone.querySelector('.add-to-cart').addEventListener('click', () => {
        addToCart(product);
    });
    
    return clone;
}
</script>
```

## Веб-компоненты

Веб-компоненты - это набор веб-технологий, позволяющих создавать переиспользуемые настраиваемые элементы с изолированным DOM и стилями.

### Основные технологии веб-компонентов:

1. **Custom Elements** - позволяет определять пользовательские HTML элементы
2. **Shadow DOM** - позволяет инкапсулировать DOM и стили
3. **HTML Templates** - элемент `<template>` для шаблонов
4. **ES Modules** - для загрузки компонентов

### Создание пользовательского элемента

```javascript
// custom-elements/user-card.js
class UserCard extends HTMLElement {
    constructor() {
        super();
        
        // Создание Shadow DOM
        const shadow = this.attachShadow({mode: 'open'});
        
        // Стили компонента
        const style = document.createElement('style');
        style.textContent = `
            .card {
                border: 1px solid #ddd;
                border-radius: 8px;
                padding: 16px;
                margin: 8px;
                box-shadow: 0 2px 4px rgba(0,0,0,0.1);
                display: flex;
                align-items: center;
                max-width: 400px;
            }
            
            .avatar {
                width: 50px;
                height: 50px;
                border-radius: 50%;
                margin-right: 16px;
            }
            
            .info {
                flex: 1;
            }
            
            .name {
                font-weight: bold;
                margin: 0 0 4px 0;
            }
            
            .email {
                color: #666;
                margin: 0;
                font-size: 0.9em;
            }
        `;
        
        // Шаблон компонента
        const card = document.createElement('div');
        card.className = 'card';
        
        const avatar = document.createElement('img');
        avatar.className = 'avatar';
        avatar.alt = 'Аватар пользователя';
        
        const info = document.createElement('div');
        info.className = 'info';
        
        const name = document.createElement('h3');
        name.className = 'name';
        
        const email = document.createElement('p');
        email.className = 'email';
        
        // Сборка компонента
        info.appendChild(name);
        info.appendChild(email);
        card.appendChild(avatar);
        card.appendChild(info);
        
        // Добавление в Shadow DOM
        shadow.appendChild(style);
        shadow.appendChild(card);
    }
    
    // Атрибуты, за которыми нужно наблюдать
    static get observedAttributes() {
        return ['name', 'email', 'avatar'];
    }
    
    // Вызывается при изменении наблюдаемого атрибута
    attributeChangedCallback(name, oldValue, newValue) {
        const shadow = this.shadowRoot;
        switch(name) {
            case 'name':
                shadow.querySelector('.name').textContent = newValue;
                break;
            case 'email':
                shadow.querySelector('.email').textContent = newValue;
                break;
            case 'avatar':
                shadow.querySelector('.avatar').src = newValue;
                break;
        }
    }
    
    // Вызывается при добавлении элемента в DOM
    connectedCallback() {
        // Можно добавить дополнительную логику при подключении
        this.addEventListener('click', this.handleClick);
    }
    
    // Вызывается при удалении элемента из DOM
    disconnectedCallback() {
        this.removeEventListener('click', this.handleClick);
    }
    
    handleClick = (event) => {
        const name = this.getAttribute('name');
        const email = this.getAttribute('email');
        alert(`Информация о пользователе:\nИмя: ${name}\nEmail: ${email}`);
    }
}

// Регистрация пользовательского элемента
customElements.define('user-card', UserCard);
```

### Использование пользовательского элемента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Веб-компоненты</title>
</head>
<body>
    <h1>Список пользователей</h1>
    
    <!-- Использование пользовательского элемента -->
    <user-card 
        name="Иван Иванов" 
        email="ivan@example.com" 
        avatar="https://example.com/avatar1.jpg">
    </user-card>
    
    <user-card 
        name="Мария Петрова" 
        email="maria@example.com" 
        avatar="https://example.com/avatar2.jpg">
    </user-card>
    
    <!-- Динамическое создание элементов -->
    <div id="dynamic-users"></div>
    
    <script src="custom-elements/user-card.js"></script>
    
    <script>
        // Динамическое создание пользовательских элементов
        const userData = [
            { name: "Алексей Сидоров", email: "alexey@example.com", avatar: "avatar3.jpg" },
            { name: "Елена Козлова", email: "elena@example.com", avatar: "avatar4.jpg" }
        ];
        
        const container = document.getElementById('dynamic-users');
        
        userData.forEach(user => {
            const userCard = document.createElement('user-card');
            userCard.setAttribute('name', user.name);
            userCard.setAttribute('email', user.email);
            userCard.setAttribute('avatar', user.avatar);
            
            container.appendChild(userCard);
        });
    </script>
</body>
</html>
```

## Пример сложного веб-компонента

```javascript
// custom-elements/toggle-switch.js
class ToggleSwitch extends HTMLElement {
    constructor() {
        super();
        
        this.shadow = this.attachShadow({mode: 'open'});
        
        this.state = false;
        
        this.render();
    }
    
    static get observedAttributes() {
        return ['checked', 'disabled', 'label'];
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        switch(name) {
            case 'checked':
                this.state = newValue !== null;
                this.updateState();
                break;
            case 'disabled':
                this.disabled = newValue !== null;
                this.updateState();
                break;
            case 'label':
                this.label = newValue;
                this.render();
                break;
        }
    }
    
    render() {
        this.shadow.innerHTML = `
            <style>
                :host {
                    display: inline-block;
                    font-family: Arial, sans-serif;
                }
                
                .switch-container {
                    display: flex;
                    align-items: center;
                    cursor: ${this.disabled ? 'not-allowed' : 'pointer'};
                    opacity: ${this.disabled ? '0.6' : '1'};
                }
                
                .switch {
                    position: relative;
                    display: inline-block;
                    width: 50px;
                    height: 24px;
                }
                
                .switch input {
                    opacity: 0;
                    width: 0;
                    height: 0;
                }
                
                .slider {
                    position: absolute;
                    cursor: ${this.disabled ? 'not-allowed' : 'pointer'};
                    top: 0;
                    left: 0;
                    right: 0;
                    bottom: 0;
                    background-color: #ccc;
                    transition: .4s;
                    border-radius: 24px;
                }
                
                .slider:before {
                    position: absolute;
                    content: "";
                    height: 16px;
                    width: 16px;
                    left: 4px;
                    bottom: 4px;
                    background-color: white;
                    transition: .4s;
                    border-radius: 50%;
                }
                
                input:checked + .slider {
                    background-color: #2196F3;
                }
                
                input:checked + .slider:before {
                    transform: translateX(26px);
                }
                
                .label {
                    margin-left: 8px;
                    user-select: none;
                }
            </style>
            
            <div class="switch-container">
                <label class="switch">
                    <input type="checkbox" ?checked="${this.state}" ?disabled="${this.disabled}">
                    <span class="slider"></span>
                </label>
                <span class="label">${this.label || ''}</span>
            </div>
        `;
        
        // Добавляем обработчики событий
        const input = this.shadow.querySelector('input');
        input.addEventListener('change', this.handleChange);
    }
    
    handleChange = (event) => {
        if (!this.disabled) {
            this.state = event.target.checked;
            this.dispatchEvent(new CustomEvent('change', {
                detail: { checked: this.state }
            }));
        }
    }
    
    updateState() {
        const input = this.shadow.querySelector('input');
        if (input) {
            input.checked = this.state;
            input.disabled = this.disabled;
        }
    }
    
    connectedCallback() {
        // Устанавливаем начальное состояние
        this.state = this.hasAttribute('checked');
        this.disabled = this.hasAttribute('disabled');
        this.label = this.getAttribute('label') || '';
        
        this.updateState();
    }
}

customElements.define('toggle-switch', ToggleSwitch);
```

### Использование компонента переключателя

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Компонент переключателя</title>
</head>
<body>
    <h1>Настройки пользователя</h1>
    
    <div>
        <toggle-switch 
            label="Уведомления по email" 
            checked>
        </toggle-switch>
    </div>
    
    <div>
        <toggle-switch 
            label="Темная тема" 
            id="theme-toggle">
        </toggle-switch>
    </div>
    
    <div>
        <toggle-switch 
            label="Режим разработчика" 
            disabled>
        </toggle-switch>
    </div>
    
    <script src="custom-elements/toggle-switch.js"></script>
    
    <script>
        // Обработка событий переключателя
        document.getElementById('theme-toggle').addEventListener('change', (event) => {
            if (event.detail.checked) {
                document.body.classList.add('dark-theme');
            } else {
                document.body.classList.remove('dark-theme');
            }
        });
    </script>
</body>
</html>
```

## Пользовательские встроенные элементы

Можно расширять существующие HTML элементы:

```javascript
class FancyButton extends HTMLButtonElement {
    constructor() {
        super();
        
        // Добавляем функциональность
        this.addEventListener('mouseenter', this.onMouseEnter);
        this.addEventListener('mouseleave', this.onMouseLeave);
    }
    
    connectedCallback() {
        // Добавляем стили и функциональность
        this.classList.add('fancy-button');
        this.innerHTML = `
            <span class="ripple"></span>
            ${this.innerHTML}
        `;
    }
    
    onMouseEnter = () => {
        this.style.transform = 'scale(1.05)';
    }
    
    onMouseLeave = () => {
        this.style.transform = 'scale(1)';
    }
}

// Регистрация с опцией extends
customElements.define('fancy-button', FancyButton, { extends: 'button' });
```

### Использование пользовательского встроенного элемента

```html
<button is="fancy-button" class="primary">Нажми меня</button>
<button is="fancy-button" class="secondary">Еще одна кнопка</button>
```

## Модули ES для веб-компонентов

```javascript
// components/user-profile.js
export class UserProfile extends HTMLElement {
    constructor() {
        super();
        this.shadow = this.attachShadow({mode: 'open'});
        this.render();
    }
    
    static get observedAttributes() {
        return ['username', 'avatar', 'bio'];
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        if (oldValue !== newValue) {
            this[name] = newValue;
            this.render();
        }
    }
    
    render() {
        this.shadow.innerHTML = `
            <style>
                .profile {
                    display: flex;
                    align-items: center;
                    padding: 16px;
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    max-width: 300px;
                }
                
                .avatar {
                    width: 50px;
                    height: 50px;
                    border-radius: 50%;
                    margin-right: 16px;
                }
                
                .info {
                    flex: 1;
                }
                
                .username {
                    font-weight: bold;
                    margin: 0 0 4px 0;
                }
                
                .bio {
                    margin: 0;
                    color: #666;
                    font-size: 0.9em;
                }
            </style>
            
            <div class="profile">
                <img class="avatar" src="${this.avatar || 'default-avatar.png'}" alt="Avatar">
                <div class="info">
                    <h3 class="username">${this.username || 'Анонимный пользователь'}</h3>
                    <p class="bio">${this.bio || 'Нет описания'}</p>
                </div>
            </div>
        `;
    }
}

customElements.define('user-profile', UserProfile);

// Экспорт для использования в других модулях
export default UserProfile;
```

### Импорт и использование модуля

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Модульные веб-компоненты</title>
</head>
<body>
    <h1>Профили пользователей</h1>
    
    <user-profile 
        username="Иван Иванов" 
        avatar="avatar1.jpg" 
        bio="Разработчик с 5-летним опытом">
    </user-profile>
    
    <script type="module">
        import UserProfile from './components/user-profile.js';
        
        // Динамическое создание компонента
        const profile = document.createElement('user-profile');
        profile.setAttribute('username', 'Мария Петрова');
        profile.setAttribute('avatar', 'avatar2.jpg');
        profile.setAttribute('bio', 'Дизайнер интерфейсов');
        
        document.body.appendChild(profile);
    </script>
</body>
</html>
```

## Современные возможности шаблонов и веб-компонентов

### Шаблоны с Web Components API
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Современные шаблоны с Web Components</title>
</head>
<body>
    <h1>Современные шаблоны и компоненты</h1>

    <!-- Использование компонента с шаблоном -->
    <data-table-component id="users-table">
        <template slot="row-template">
            <tr>
                <td class="name-cell"></td>
                <td class="email-cell"></td>
                <td class="actions-cell">
                    <button class="edit-btn">Редактировать</button>
                    <button class="delete-btn">Удалить</button>
                </td>
            </tr>
        </template>
    </data-table-component>

    <script>
        class DataTableComponent extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                
                this.data = [];
                
                this.render();
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
                            background-color: #f2f2f2;
                            font-weight: bold;
                        }

                        tr:hover {
                            background-color: #f5f5f5;
                        }

                        .edit-btn, .delete-btn {
                            padding: 4px 8px;
                            margin: 0 2px;
                            border: none;
                            border-radius: 4px;
                            cursor: pointer;
                            font-size: 0.8em;
                        }

                        .edit-btn {
                            background-color: #007acc;
                            color: white;
                        }

                        .delete-btn {
                            background-color: #d32f2f;
                            color: white;
                        }
                    </style>

                    <table>
                        <thead>
                            <tr>
                                <th>Имя</th>
                                <th>Email</th>
                                <th>Действия</th>
                            </tr>
                        </thead>
                        <tbody id="table-body">
                            <!-- Данные будут добавлены сюда -->
                        </tbody>
                    </table>
                `;
            }

            connectedCallback() {
                // Загрузка шаблона из слота
                const templateSlot = this.querySelector('template[slot="row-template"]');
                if (templateSlot) {
                    this.rowTemplate = templateSlot.content.cloneNode(true);
                }

                // Загрузка данных
                this.loadData();
            }

            async loadData() {
                // Имитация загрузки данных
                this.data = [
                    { id: 1, name: 'Иван Иванов', email: 'ivan@example.com' },
                    { id: 2, name: 'Мария Петрова', email: 'maria@example.com' },
                    { id: 3, name: 'Алексей Сидоров', email: 'alexey@example.com' }
                ];

                this.renderData();
            }

            renderData() {
                const tbody = this.shadowRoot.getElementById('table-body');
                tbody.innerHTML = '';

                this.data.forEach(item => {
                    const row = this.createRow(item);
                    tbody.appendChild(row);
                });
            }

            createRow(item) {
                if (!this.rowTemplate) {
                    // Резервный шаблон
                    const row = document.createElement('tr');
                    row.innerHTML = `
                        <td>${item.name}</td>
                        <td>${item.email}</td>
                        <td>
                            <button class="edit-btn" data-id="${item.id}">Редактировать</button>
                            <button class="delete-btn" data-id="${item.id}">Удалить</button>
                        </td>
                    `;
                    return row;
                }

                const row = this.rowTemplate.cloneNode(true);
                
                // Заполнение данных
                row.querySelector('.name-cell').textContent = item.name;
                row.querySelector('.email-cell').textContent = item.email;
                
                // Установка данных для обработчиков
                row.querySelector('.edit-btn').dataset.id = item.id;
                row.querySelector('.delete-btn').dataset.id = item.id;
                
                // Добавление обработчиков
                row.querySelector('.edit-btn').addEventListener('click', () => this.editItem(item.id));
                row.querySelector('.delete-btn').addEventListener('click', () => this.deleteItem(item.id));

                return row;
            }

            editItem(id) {
                const item = this.data.find(i => i.id === id);
                if (item) {
                    const newName = prompt('Введите новое имя:', item.name);
                    if (newName) {
                        item.name = newName;
                        this.renderData();
                    }
                }
            }

            deleteItem(id) {
                if (confirm('Удалить запись?')) {
                    this.data = this.data.filter(item => item.id !== id);
                    this.renderData();
                }
            }
        }

        customElements.define('data-table-component', DataTableComponent);
    </script>
</body>
</html>
```

### Шаблоны с IndexedDB и оффлайн-функциональностью
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Шаблоны с IndexedDB</title>
</head>
<body>
    <h1>Оффлайн заметки с шаблонами</h1>

    <div id="app">
        <form id="note-form">
            <input type="text" id="note-title" placeholder="Заголовок заметки" required>
            <textarea id="note-content" placeholder="Содержимое заметки" required></textarea>
            <button type="submit">Сохранить заметку</button>
        </form>

        <div id="notes-container"></div>
    </div>

    <!-- Шаблон для заметки -->
    <template id="note-template">
        <div class="note-card">
            <div class="note-header">
                <h3 class="note-title"></h3>
                <div class="note-meta">
                    <span class="note-date"></span>
                    <button class="note-edit-btn">Редактировать</button>
                    <button class="note-delete-btn">Удалить</button>
                </div>
            </div>
            <div class="note-content"></div>
        </div>
    </template>

    <script>
        class OfflineNotesApp {
            constructor() {
                this.db = null;
                this.initDB();
                this.bindEvents();
                this.loadNotes();
            }

            async initDB() {
                const request = indexedDB.open('OfflineNotesDB', 1);

                request.onerror = (event) => {
                    console.error('Ошибка при открытии базы данных:', event.target.error);
                };

                request.onsuccess = (event) => {
                    this.db = event.target.result;
                    console.log('База данных открыта');
                };

                request.onupgradeneeded = (event) => {
                    this.db = event.target.result;

                    if (!this.db.objectStoreNames.contains('notes')) {
                        const store = this.db.createObjectStore('notes', { keyPath: 'id', autoIncrement: true });
                        store.createIndex('title', 'title', { unique: false });
                        store.createIndex('date', 'date', { unique: false });
                    }
                };
            }

            bindEvents() {
                document.getElementById('note-form').addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.saveNote();
                });
            }

            async saveNote() {
                const title = document.getElementById('note-title').value.trim();
                const content = document.getElementById('note-content').value.trim();

                if (!title || !content) return;

                const note = {
                    title: title,
                    content: content,
                    date: new Date().toISOString()
                };

                const transaction = this.db.transaction(['notes'], 'readwrite');
                const store = transaction.objectStore('notes');
                const request = store.add(note);

                request.onsuccess = () => {
                    console.log('Заметка сохранена');
                    document.getElementById('note-form').reset();
                    this.loadNotes();
                };

                request.onerror = (e) => {
                    console.error('Ошибка при сохранении заметки:', e.target.error);
                };
            }

            async loadNotes() {
                if (!this.db) return;

                const transaction = this.db.transaction(['notes'], 'readonly');
                const store = transaction.objectStore('notes');
                const request = store.getAll();

                request.onsuccess = (e) => {
                    const notes = e.target.result;
                    this.renderNotes(notes);
                };

                request.onerror = (e) => {
                    console.error('Ошибка при загрузке заметок:', e.target.error);
                };
            }

            renderNotes(notes) {
                const container = document.getElementById('notes-container');
                container.innerHTML = '';

                if (notes.length === 0) {
                    container.innerHTML = '<p>Нет сохраненных заметок.</p>';
                    return;
                }

                // Сортировка по дате (новые первыми)
                notes.sort((a, b) => new Date(b.date) - new Date(a.date));

                notes.forEach(note => {
                    const template = document.getElementById('note-template');
                    const clone = template.content.cloneNode(true);

                    // Заполнение данных
                    clone.querySelector('.note-title').textContent = note.title;
                    clone.querySelector('.note-content').textContent = note.content;
                    
                    const date = new Date(note.date).toLocaleString('ru-RU');
                    clone.querySelector('.note-date').textContent = date;

                    // Установка данных для обработчиков
                    clone.querySelector('.note-edit-btn').dataset.noteId = note.id;
                    clone.querySelector('.note-delete-btn').dataset.noteId = note.id;

                    // Добавление обработчиков
                    clone.querySelector('.note-edit-btn').addEventListener('click', () => this.editNote(note.id));
                    clone.querySelector('.note-delete-btn').addEventListener('click', () => this.deleteNote(note.id));

                    container.appendChild(clone);
                });
            }

            async editNote(id) {
                if (!this.db) return;

                const transaction = this.db.transaction(['notes'], 'readonly');
                const store = transaction.objectStore('notes');
                const request = store.get(Number(id));

                request.onsuccess = (e) => {
                    const note = e.target.result;
                    if (note) {
                        document.getElementById('note-title').value = note.title;
                        document.getElementById('note-content').value = note.content;
                        
                        // Удалить заметку для редактирования
                        this.deleteNote(id, false); // не перезагружать список
                    }
                };
            }

            async deleteNote(id, reload = true) {
                if (!this.db) return;

                const transaction = this.db.transaction(['notes'], 'readwrite');
                const store = transaction.objectStore('notes');
                const request = store.delete(Number(id));

                request.onsuccess = () => {
                    console.log('Заметка удалена');
                    if (reload) {
                        this.loadNotes();
                    }
                };

                request.onerror = (e) => {
                    console.error('Ошибка при удалении заметки:', e.target.error);
                };
            }
        }

        // Инициализация при загрузке страницы
        document.addEventListener('DOMContentLoaded', () => {
            new OfflineNotesApp();
        });
    </script>

    <style>
        #app {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }

        #note-form {
            margin-bottom: 30px;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        #note-form input,
        #note-form textarea {
            width: 100%;
            padding: 10px;
            margin-bottom: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }

        #note-form textarea {
            height: 100px;
            resize: vertical;
        }

        #note-form button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .note-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
            background-color: #fafafa;
        }

        .note-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }

        .note-title {
            margin: 0;
            color: #333;
        }

        .note-meta {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .note-date {
            font-size: 0.8em;
            color: #666;
        }

        .note-edit-btn,
        .note-delete-btn {
            padding: 4px 8px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.8em;
        }

        .note-edit-btn {
            background-color: #007acc;
            color: white;
        }

        .note-delete-btn {
            background-color: #d32f2f;
            color: white;
        }

        .note-content {
            color: #555;
            line-height: 1.5;
        }
    </style>
</body>
</html>
```

### Шаблоны с WebSocket и динамическим обновлением
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Шаблоны с WebSocket</title>
</head>
<body>
    <h1>Живой чат с шаблонами</h1>

    <div id="chat-container">
        <div id="messages-container"></div>
        
        <form id="message-form">
            <input type="text" id="message-input" placeholder="Введите сообщение..." required>
            <button type="submit">Отправить</button>
        </form>
    </div>

    <!-- Шаблон для сообщения -->
    <template id="message-template">
        <div class="message">
            <div class="message-header">
                <span class="message-user"></span>
                <span class="message-time"></span>
            </div>
            <div class="message-content"></div>
        </div>
    </template>

    <script>
        class LiveChatApp {
            constructor() {
                this.ws = null;
                this.userId = this.generateUserId();
                this.userName = this.getUserName();
                
                this.connectWebSocket();
                this.bindEvents();
            }

            generateUserId() {
                return 'user_' + Math.random().toString(36).substr(2, 9);
            }

            getUserName() {
                return localStorage.getItem('chat_username') || 'Аноним';
            }

            connectWebSocket() {
                try {
                    // В реальном приложении здесь будет реальный WebSocket URL
                    // this.ws = new WebSocket('ws://localhost:8080/chat');
                    
                    // Для демонстрации создадим mock-соединение
                    this.ws = {
                        send: (message) => {
                            // Эмуляция отправки сообщения
                            console.log('Отправлено:', message);
                            
                            // Эмуляция получения сообщения
                            setTimeout(() => {
                                this.displayMessage({
                                    userId: this.userId,
                                    userName: this.userName,
                                    content: JSON.parse(message).content,
                                    timestamp: new Date().toISOString()
                                });
                            }, 100);
                        },
                        close: () => {}
                    };
                    
                    // Эмуляция получения сообщений
                    setInterval(() => {
                        if (Math.random() > 0.8) { // 20% шанс получить сообщение
                            const users = ['Алексей', 'Мария', 'Иван', 'Елена', 'Дмитрий'];
                            const messages = [
                                'Привет всем!', 'Как дела?', 'Работаю над проектом', 'Кто на связи?',
                                'Нужна помощь?', 'Все прошло отлично!', 'Планирую встречу', 'Обновил документацию'
                            ];
                            
                            const randomUser = users[Math.floor(Math.random() * users.length)];
                            const randomMessage = messages[Math.floor(Math.random() * messages.length)];
                            
                            this.displayMessage({
                                userId: 'system_' + Math.random().toString(36).substr(2, 5),
                                userName: randomUser,
                                content: randomMessage,
                                timestamp: new Date().toISOString()
                            });
                        }
                    }, 3000);
                    
                } catch (error) {
                    console.error('Ошибка подключения к чату:', error);
                }
            }

            bindEvents() {
                document.getElementById('message-form').addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.sendMessage();
                });
            }

            sendMessage() {
                const input = document.getElementById('message-input');
                const content = input.value.trim();

                if (!content || !this.ws) return;

                const message = {
                    userId: this.userId,
                    userName: this.userName,
                    content: content,
                    timestamp: new Date().toISOString()
                };

                this.ws.send(JSON.stringify(message));
                input.value = '';
            }

            displayMessage(messageData) {
                const template = document.getElementById('message-template');
                const clone = template.content.cloneNode(true);

                // Заполнение данных
                clone.querySelector('.message-user').textContent = messageData.userName;
                clone.querySelector('.message-content').textContent = messageData.content;
                
                const time = new Date(messageData.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
                clone.querySelector('.message-time').textContent = time;

                // Добавление класса для сообщения пользователя
                if (messageData.userId === this.userId) {
                    clone.querySelector('.message').classList.add('own-message');
                }

                document.getElementById('messages-container').appendChild(clone);

                // Прокрутка вниз
                const container = document.getElementById('messages-container');
                container.scrollTop = container.scrollHeight;
            }
        }

        // Инициализация при загрузке страницы
        document.addEventListener('DOMContentLoaded', () => {
            new LiveChatApp();
        });
    </script>

    <style>
        #chat-container {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            height: 500px;
            display: flex;
            flex-direction: column;
        }

        #messages-container {
            flex: 1;
            overflow-y: auto;
            margin-bottom: 15px;
            padding: 10px;
            background-color: #f9f9f9;
            border-radius: 4px;
        }

        .message {
            margin-bottom: 10px;
            padding: 10px;
            border-radius: 8px;
            background-color: white;
            border: 1px solid #eee;
        }

        .own-message {
            background-color: #e3f2fd;
            border-color: #bbdefb;
            margin-left: 20%;
        }

        .message-header {
            display: flex;
            justify-content: space-between;
            margin-bottom: 5px;
            font-size: 0.8em;
            color: #666;
        }

        .message-user {
            font-weight: bold;
            color: #333;
        }

        .message-content {
            color: #333;
            line-height: 1.4;
        }

        #message-form {
            display: flex;
            gap: 10px;
        }

        #message-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 16px;
        }

        #message-form button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
    </style>
</body>
</html>
```

## Практические примеры и шаблоны

### Шаблон для карточки товара с различными вариантами
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Шаблоны карточек товаров</title>
</head>
<body>
    <h1>Каталог товаров</h1>

    <div class="products-grid" id="products-container"></div>

    <!-- Шаблон стандартной карточки товара -->
    <template id="product-card-template">
        <div class="product-card standard">
            <div class="product-image-container">
                <img class="product-image" src="" alt="">
                <div class="product-badge" style="display: none;"></div>
            </div>
            <div class="product-info">
                <h3 class="product-title"></h3>
                <p class="product-description"></p>
                <div class="product-rating">
                    <span class="rating-stars"></span>
                    <span class="rating-count"></span>
                </div>
                <div class="product-price-container">
                    <span class="product-price"></span>
                    <span class="product-old-price" style="display: none;"></span>
                </div>
                <div class="product-actions">
                    <button class="add-to-cart">Добавить в корзину</button>
                    <button class="wishlist">В избранное</button>
                </div>
            </div>
        </div>
    </template>

    <!-- Шаблон мини-карточки товара -->
    <template id="mini-product-card-template">
        <div class="product-card mini">
            <img class="product-image" src="" alt="">
            <div class="product-info">
                <h4 class="product-title"></h4>
                <div class="product-price-container">
                    <span class="product-price"></span>
                </div>
            </div>
        </div>
    </template>

    <!-- Шаблон расширенной карточки товара -->
    <template id="extended-product-card-template">
        <div class="product-card extended">
            <div class="product-main">
                <div class="product-images">
                    <img class="main-image" src="" alt="">
                    <div class="thumbnail-container">
                        <!-- Миниатюры будут добавлены сюда -->
                    </div>
                </div>
                <div class="product-details">
                    <h2 class="product-title"></h2>
                    <div class="product-rating">
                        <span class="rating-stars"></span>
                        <span class="rating-count"></span>
                    </div>
                    <p class="product-description"></p>
                    <div class="product-specs">
                        <!-- Характеристики будут добавлены сюда -->
                    </div>
                    <div class="product-price-container">
                        <span class="product-price"></span>
                        <span class="product-old-price" style="display: none;"></span>
                    </div>
                    <div class="product-actions">
                        <button class="add-to-cart">Добавить в корзину</button>
                        <button class="buy-now">Купить сейчас</button>
                        <button class="compare">Сравнить</button>
                    </div>
                </div>
            </div>
        </div>
    </template>

    <script>
        class ProductCardFactory {
            static createProductCard(product, variant = 'standard') {
                let templateId;
                
                switch(variant) {
                    case 'mini':
                        templateId = 'mini-product-card-template';
                        break;
                    case 'extended':
                        templateId = 'extended-product-card-template';
                        break;
                    case 'standard':
                    default:
                        templateId = 'product-card-template';
                }
                
                const template = document.getElementById(templateId);
                const clone = template.content.cloneNode(true);
                
                this.populateCard(clone, product, variant);
                this.bindCardEvents(clone, product, variant);
                
                return clone;
            }
            
            static populateCard(fragment, product, variant) {
                // Заполнение общих элементов
                const image = fragment.querySelector('.product-image') || fragment.querySelector('.main-image');
                if (image) {
                    image.src = product.image;
                    image.alt = product.title;
                }
                
                const title = fragment.querySelector('.product-title');
                if (title) {
                    title.textContent = product.title;
                    if (variant === 'extended') {
                        title.textContent = product.fullTitle || product.title;
                    }
                }
                
                const price = fragment.querySelector('.product-price');
                if (price) {
                    price.textContent = this.formatPrice(product.price);
                }
                
                // Заполнение специфичных для варианта элементов
                switch(variant) {
                    case 'standard':
                        this.populateStandardCard(fragment, product);
                        break;
                    case 'mini':
                        this.populateMiniCard(fragment, product);
                        break;
                    case 'extended':
                        this.populateExtendedCard(fragment, product);
                        break;
                }
            }
            
            static populateStandardCard(fragment, product) {
                const description = fragment.querySelector('.product-description');
                if (description) {
                    description.textContent = product.description;
                }
                
                if (product.badge) {
                    const badge = fragment.querySelector('.product-badge');
                    badge.textContent = product.badge;
                    badge.style.display = 'block';
                    if (product.badgeType) {
                        badge.className = `product-badge ${product.badgeType}`;
                    }
                }
                
                if (product.oldPrice) {
                    const oldPrice = fragment.querySelector('.product-old-price');
                    oldPrice.textContent = this.formatPrice(product.oldPrice);
                    oldPrice.style.display = 'inline';
                    oldPrice.style.textDecoration = 'line-through';
                    oldPrice.style.marginLeft = '8px';
                    oldPrice.style.color = '#d32f2f';
                }
                
                if (product.rating) {
                    const stars = fragment.querySelector('.rating-stars');
                    const count = fragment.querySelector('.rating-count');
                    
                    if (stars) {
                        stars.innerHTML = '★'.repeat(Math.floor(product.rating)) + 
                                         '☆'.repeat(5 - Math.floor(product.rating));
                    }
                    
                    if (count) {
                        count.textContent = `(${product.reviewCount || 0} отзывов)`;
                    }
                }
            }
            
            static populateMiniCard(fragment, product) {
                // Мини-карточка содержит только основную информацию
            }
            
            static populateExtendedCard(fragment, product) {
                const description = fragment.querySelector('.product-description');
                if (description) {
                    description.textContent = product.fullDescription || product.description;
                }
                
                if (product.specifications) {
                    const specsContainer = fragment.querySelector('.product-specs');
                    specsContainer.innerHTML = '';
                    
                    Object.entries(product.specifications).forEach(([key, value]) => {
                        const specItem = document.createElement('div');
                        specItem.className = 'spec-item';
                        specItem.innerHTML = `<strong>${key}:</strong> ${value}`;
                        specsContainer.appendChild(specItem);
                    });
                }
                
                if (product.images && product.images.length > 1) {
                    const thumbContainer = fragment.querySelector('.thumbnail-container');
                    thumbContainer.innerHTML = '';
                    
                    product.images.slice(1).forEach((imgSrc, index) => {
                        const thumb = document.createElement('img');
                        thumb.src = imgSrc;
                        thumb.alt = `Миниатюра ${index + 1}`;
                        thumb.className = 'thumbnail';
                        thumbContainer.appendChild(thumb);
                    });
                }
            }
            
            static bindCardEvents(fragment, product, variant) {
                const addToCartBtn = fragment.querySelector('.add-to-cart');
                if (addToCartBtn) {
                    addToCartBtn.addEventListener('click', () => {
                        console.log('Добавление в корзину:', product.title);
                        // Реализация добавления в корзину
                    });
                }
                
                const wishlistBtn = fragment.querySelector('.wishlist');
                if (wishlistBtn) {
                    wishlistBtn.addEventListener('click', () => {
                        console.log('Добавление в избранное:', product.title);
                        // Реализация добавления в избранное
                    });
                }
            }
            
            static formatPrice(price) {
                return new Intl.NumberFormat('ru-RU', {
                    style: 'currency',
                    currency: 'RUB',
                    minimumFractionDigits: 0
                }).format(price);
            }
        }

        // Пример использования
        const products = [
            {
                id: 1,
                title: 'Смартфон iPhone 15',
                fullTitle: 'Apple iPhone 15 Pro Max 256GB Titanium Natural',
                description: 'Флагманский смартфон от Apple',
                fullDescription: 'Флагманский смартфон от Apple с передовыми технологиями и камерой Pro.',
                price: 129990,
                oldPrice: 139990,
                image: 'iphone15.jpg',
                badge: 'Хит продаж',
                badgeType: 'best-seller',
                rating: 4.8,
                reviewCount: 124,
                specifications: {
                    'Дисплей': '6.7" Super Retina XDR',
                    'Память': '256 ГБ',
                    'Цвет': 'Титан естественный',
                    'Камера': '48 МП тройная система'
                },
                images: ['iphone15_main.jpg', 'iphone15_back.jpg', 'iphone15_side.jpg']
            },
            {
                id: 2,
                title: 'Ноутбук MacBook Air',
                description: 'Ультрабук для работы и развлечений',
                price: 99990,
                image: 'macbook-air.jpg',
                rating: 4.7,
                reviewCount: 89
            }
        ];

        // Рендеринг карточек
        const container = document.getElementById('products-container');
        
        products.forEach(product => {
            // Стандартная карточка для основного каталога
            const standardCard = ProductCardFactory.createProductCard(product, 'standard');
            container.appendChild(standardCard);
        });
    </script>

    <style>
        .products-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 20px;
            padding: 20px;
        }

        .product-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            overflow: hidden;
            transition: transform 0.3s, box-shadow 0.3s;
        }

        .product-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 16px rgba(0,0,0,0.1);
        }

        .product-image-container {
            position: relative;
            width: 100%;
            height: 200px;
            overflow: hidden;
        }

        .product-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        .product-badge {
            position: absolute;
            top: 10px;
            left: 10px;
            background-color: #ff5722;
            color: white;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.8em;
            font-weight: bold;
        }

        .product-info {
            padding: 15px;
        }

        .product-title {
            margin: 0 0 10px 0;
            font-size: 1.1em;
        }

        .product-description {
            margin: 0 0 10px 0;
            color: #666;
            font-size: 0.9em;
            line-height: 1.4;
        }

        .product-rating {
            display: flex;
            align-items: center;
            margin-bottom: 10px;
            font-size: 0.9em;
        }

        .rating-stars {
            color: #ffc107;
            margin-right: 5px;
        }

        .product-price-container {
            margin: 10px 0;
            font-weight: bold;
            font-size: 1.2em;
        }

        .product-actions {
            display: flex;
            gap: 10px;
            margin-top: 15px;
        }

        .product-actions button {
            flex: 1;
            padding: 8px 12px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.9em;
        }

        .add-to-cart {
            background-color: #007acc;
            color: white;
        }

        .wishlist {
            background-color: #f5f5f5;
            color: #333;
        }

        /* Стили для мини-карточки */
        .product-card.mini {
            display: flex;
            align-items: center;
            padding: 10px;
        }

        .product-card.mini .product-image {
            width: 60px;
            height: 60px;
            margin-right: 10px;
        }

        .product-card.mini .product-info {
            padding: 0;
        }

        .product-card.mini .product-title {
            margin: 0;
            font-size: 1em;
        }

        /* Стили для расширенной карточки */
        .product-card.extended {
            border: none;
            box-shadow: none;
        }

        .product-main {
            display: flex;
            gap: 20px;
        }

        .product-images {
            flex: 1;
        }

        .main-image {
            width: 100%;
            height: 300px;
            object-fit: contain;
        }

        .thumbnail-container {
            display: flex;
            gap: 10px;
            margin-top: 10px;
        }

        .thumbnail {
            width: 60px;
            height: 60px;
            object-fit: cover;
            cursor: pointer;
            border: 2px solid transparent;
        }

        .thumbnail:hover {
            border-color: #007acc;
        }

        .product-details {
            flex: 2;
        }

        .spec-item {
            margin-bottom: 8px;
            padding-bottom: 8px;
            border-bottom: 1px solid #eee;
        }

        .buy-now {
            background-color: #4caf50;
            color: white;
        }

        .compare {
            background-color: #ff9800;
            color: white;
        }
    </style>
</body>
</html>
```

### Шаблоны с кастомными элементами и сложной логикой
```javascript
// components/advanced-data-grid.js
class AdvancedDataGrid extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        this.data = [];
        this.filteredData = [];
        this.sortConfig = { key: null, direction: 'asc' };
        this.page = 1;
        this.pageSize = 10;
        this.filters = {};
        
        this.render();
        this.initializeTemplates();
    }

    static get observedAttributes() {
        return ['data', 'columns', 'page-size'];
    }

    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    font-family: Arial, sans-serif;
                }

                .grid-container {
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    overflow: hidden;
                }

                .grid-header {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    padding: 15px;
                    background-color: #f5f5f5;
                    border-bottom: 1px solid #ddd;
                }

                .controls {
                    display: flex;
                    gap: 10px;
                    align-items: center;
                }

                .search-box {
                    padding: 8px;
                    border: 1px solid #ccc;
                    border-radius: 4px;
                }

                .grid-table {
                    width: 100%;
                    border-collapse: collapse;
                }

                .grid-table th {
                    background-color: #f0f0f0;
                    padding: 12px;
                    text-align: left;
                    cursor: pointer;
                    user-select: none;
                }

                .grid-table th:hover {
                    background-color: #e0e0e0;
                }

                .grid-table th.sorted-asc::after {
                    content: ' ↑';
                    color: #007acc;
                }

                .grid-table th.sorted-desc::after {
                    content: ' ↓';
                    color: #007acc;
                }

                .grid-table td {
                    padding: 12px;
                    border-bottom: 1px solid #eee;
                }

                .grid-table tr:hover {
                    background-color: #f9f9f9;
                }

                .pagination {
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    padding: 15px;
                    gap: 10px;
                }

                .page-btn {
                    padding: 8px 12px;
                    border: 1px solid #ddd;
                    background-color: white;
                    cursor: pointer;
                    border-radius: 4px;
                }

                .page-btn.active {
                    background-color: #007acc;
                    color: white;
                }

                .page-btn:disabled {
                    opacity: 0.5;
                    cursor: not-allowed;
                }

                .status-cell {
                    display: inline-block;
                    padding: 4px 8px;
                    border-radius: 12px;
                    font-size: 0.8em;
                }

                .status-active {
                    background-color: #4caf50;
                    color: white;
                }

                .status-inactive {
                    background-color: #f44336;
                    color: white;
                }

                .status-pending {
                    background-color: #ff9800;
                    color: white;
                }
            </style>

            <div class="grid-container">
                <div class="grid-header">
                    <h3 id="grid-title">Данные</h3>
                    <div class="controls">
                        <input type="text" class="search-box" placeholder="Поиск..." id="search-input">
                        <button class="page-btn" id="refresh-btn">🔄</button>
                    </div>
                </div>

                <table class="grid-table">
                    <thead id="table-head">
                        <!-- Заголовки будут добавлены сюда -->
                    </thead>
                    <tbody id="table-body">
                        <!-- Данные будут добавлены сюда -->
                    </tbody>
                </table>

                <div class="pagination" id="pagination">
                    <!-- Пагинация будет добавлена сюда -->
                </div>
            </div>
        `;
        
        this.bindEvents();
    }

    initializeTemplates() {
        // Шаблон для строки данных
        this.rowTemplate = document.createElement('template');
        this.rowTemplate.innerHTML = `
            <tr class="data-row">
                <!-- Ячейки будут добавлены сюда -->
            </tr>
        `;

        // Шаблон для ячейки
        this.cellTemplate = document.createElement('template');
        this.cellTemplate.innerHTML = `
            <td class="data-cell">
                <!-- Содержимое ячейки -->
            </td>
        `;
    }

    bindEvents() {
        const searchInput = this.shadowRoot.getElementById('search-input');
        searchInput.addEventListener('input', (e) => {
            this.searchTerm = e.target.value;
            this.filterData();
            this.renderData();
        });

        const refreshBtn = this.shadowRoot.getElementById('refresh-btn');
        refreshBtn.addEventListener('click', () => {
            this.refreshData();
        });
    }

    async refreshData() {
        // Имитация обновления данных
        this.dispatchEvent(new CustomEvent('refresh', { bubbles: true }));
    }

    setColumns(columns) {
        this.columns = columns;
        this.renderHeaders();
    }

    setData(data) {
        this.data = data;
        this.filteredData = [...data];
        this.renderData();
        this.renderPagination();
    }

    renderHeaders() {
        const thead = this.shadowRoot.getElementById('table-head');
        thead.innerHTML = '';

        const headerRow = document.createElement('tr');
        
        this.columns.forEach(col => {
            const th = document.createElement('th');
            th.textContent = col.title || col.key;
            th.dataset.sortKey = col.key;
            th.classList.toggle('sortable', col.sortable !== false);
            
            if (this.sortConfig.key === col.key) {
                th.classList.add(`sorted-${this.sortConfig.direction}`);
            }
            
            if (col.sortable !== false) {
                th.addEventListener('click', () => this.sortByColumn(col.key));
            }
            
            headerRow.appendChild(th);
        });

        thead.appendChild(headerRow);
    }

    renderData() {
        const tbody = this.shadowRoot.getElementById('table-body');
        tbody.innerHTML = '';

        const startIndex = (this.page - 1) * this.pageSize;
        const endIndex = startIndex + this.pageSize;
        const pageData = this.filteredData.slice(startIndex, endIndex);

        pageData.forEach(rowData => {
            const row = this.createRow(rowData);
            tbody.appendChild(row);
        });
    }

    createRow(rowData) {
        const row = this.rowTemplate.content.cloneNode(true);
        const rowElement = row.querySelector('.data-row');

        this.columns.forEach(col => {
            const cell = this.createCell(rowData[col.key], col);
            rowElement.appendChild(cell);
        });

        return rowElement;
    }

    createCell(value, column) {
        const cell = this.cellTemplate.content.cloneNode(true);
        const cellElement = cell.querySelector('.data-cell');

        if (column.template) {
            // Используем пользовательский шаблон
            cellElement.innerHTML = column.template(value, column);
        } else if (column.type === 'status') {
            // Специальная обработка статуса
            cellElement.innerHTML = `
                <span class="status-cell status-${value.toLowerCase()}">${value}</span>
            `;
        } else if (column.type === 'date') {
            // Форматирование даты
            const date = new Date(value);
            cellElement.textContent = date.toLocaleDateString('ru-RU');
        } else {
            // Обычное значение
            cellElement.textContent = value;
        }

        return cellElement;
    }

    sortByColumn(key) {
        const direction = this.sortConfig.key === key && this.sortConfig.direction === 'asc' ? 'desc' : 'asc';
        
        this.sortConfig = { key, direction };
        
        this.filteredData.sort((a, b) => {
            const valA = a[key];
            const valB = b[key];
            
            if (valA < valB) return direction === 'asc' ? -1 : 1;
            if (valA > valB) return direction === 'asc' ? 1 : -1;
            return 0;
        });
        
        this.renderData();
        this.renderHeaders(); // Обновляем индикаторы сортировки
    }

    filterData() {
        this.filteredData = this.data.filter(row => {
            if (!this.searchTerm) return true;
            
            return Object.values(row).some(value => 
                String(value).toLowerCase().includes(this.searchTerm.toLowerCase())
            );
        });
        
        this.page = 1; // Сброс на первую страницу при фильтрации
        this.renderPagination();
    }

    renderPagination() {
        const totalPages = Math.ceil(this.filteredData.length / this.pageSize);
        const pagination = this.shadowRoot.getElementById('pagination');
        pagination.innerHTML = '';

        if (totalPages <= 1) return;

        // Кнопка "Назад"
        const prevBtn = document.createElement('button');
        prevBtn.className = 'page-btn';
        prevBtn.textContent = 'Пред.';
        prevBtn.disabled = this.page === 1;
        prevBtn.addEventListener('click', () => {
            if (this.page > 1) {
                this.page--;
                this.renderData();
                this.renderPagination();
            }
        });
        pagination.appendChild(prevBtn);

        // Кнопки страниц
        for (let i = 1; i <= totalPages; i++) {
            const pageBtn = document.createElement('button');
            pageBtn.className = `page-btn ${i === this.page ? 'active' : ''}`;
            pageBtn.textContent = i;
            pageBtn.addEventListener('click', () => {
                this.page = i;
                this.renderData();
                this.renderPagination();
            });
            pagination.appendChild(pageBtn);
        }

        // Кнопка "Вперед"
        const nextBtn = document.createElement('button');
        nextBtn.className = 'page-btn';
        nextBtn.textContent = 'След.';
        nextBtn.disabled = this.page === totalPages;
        nextBtn.addEventListener('click', () => {
            if (this.page < totalPages) {
                this.page++;
                this.renderData();
                this.renderPagination();
            }
        });
        pagination.appendChild(nextBtn);
    }

    connectedCallback() {
        // Устанавливаем размер страницы
        const pageSizeAttr = this.getAttribute('page-size');
        if (pageSizeAttr) {
            this.pageSize = parseInt(pageSizeAttr) || 10;
        }
    }
}

customElements.define('advanced-data-grid', AdvancedDataGrid);
```

## Проверка и тестирование шаблонов и веб-компонентов

### Инструменты для проверки:
1. **Web Component Tester** - для тестирования веб-компонентов
2. **Chrome DevTools** - для отладки Shadow DOM
3. **Selenium WebDriver** - для автоматизированного тестирования
4. **Jest** - для unit-тестирования
5. **Playwright** - для end-to-end тестирования
6. **Lighthouse** - для проверки доступности и производительности

### Проверка производительности шаблонов:
1. Использование DocumentFragment для массовых вставок
2. Оптимизация рендеринга с помощью Virtual DOM подходов
3. Эффективное управление событиями
4. Использование requestAnimationFrame для анимаций

### Проверка доступности:
1. Проверка доступности элементов внутри Shadow DOM
2. Корректное использование ARIA-атрибутов
3. Клавиатурная навигация
4. Поддержка скринридеров

## Лучшие практики шаблонов и веб-компонентов

### 1. Эффективное управление памятью
```javascript
// Правильная очистка ресурсов
class MemoryEfficientComponent extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        // Хранение ссылок на обработчики для последующей очистки
        this.eventHandlers = new Map();
        this.intervals = new Set();
        this.timeouts = new Set();
    }

    connectedCallback() {
        this.render();
        this.setupEventListeners();
        this.startPeriodicTasks();
    }

    disconnectedCallback() {
        this.cleanupEventListeners();
        this.stopPeriodicTasks();
        this.clearAllIntervals();
        this.clearAllTimeouts();
    }

    setupEventListeners() {
        const handler = (e) => this.handleEvent(e);
        this.addEventListener('click', handler);
        this.eventHandlers.set('click', handler);
    }

    cleanupEventListeners() {
        this.eventHandlers.forEach((handler, eventType) => {
            this.removeEventListener(eventType, handler);
        });
        this.eventHandlers.clear();
    }

    startPeriodicTasks() {
        const intervalId = setInterval(() => {
            this.updateData();
        }, 30000); // каждые 30 секунд
        
        this.intervals.add(intervalId);
    }

    stopPeriodicTasks() {
        this.intervals.forEach(id => clearInterval(id));
        this.intervals.clear();
    }

    clearAllIntervals() {
        this.intervals.forEach(id => clearInterval(id));
        this.intervals.clear();
    }

    clearAllTimeouts() {
        this.timeouts.forEach(id => clearTimeout(id));
        this.timeouts.clear();
    }
}
```

### 2. Оптимизация рендеринга
```javascript
// Оптимизированный рендеринг с виртуальным скроллом
class OptimizedListComponent extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        this.items = [];
        this.visibleItems = [];
        this.itemHeight = 50; // высота одного элемента в px
        this.containerHeight = 400; // высота контейнера в px
        this.startIndex = 0;
        this.endIndex = 0;
    }

    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    height: 400px;
                }

                .scroller {
                    height: ${this.getTotalHeight()}px;
                    position: relative;
                }

                .visible-items {
                    position: absolute;
                    top: ${this.startIndex * this.itemHeight}px;
                    width: 100%;
                }

                .item {
                    height: ${this.itemHeight}px;
                    padding: 8px;
                    border-bottom: 1px solid #eee;
                }
            </style>

            <div class="scroller" id="scroller">
                <div class="visible-items" id="visible-items"></div>
            </div>
        `;
        
        this.bindScrollEvents();
    }

    getTotalHeight() {
        return this.items.length * this.itemHeight;
    }

    bindScrollEvents() {
        const scroller = this.shadowRoot.getElementById('scroller');
        scroller.addEventListener('scroll', this.throttledRender.bind(this), { passive: true });
    }

    throttledRender = this.throttle(() => {
        this.calculateVisibleRange();
        this.renderVisibleItems();
    }, 16); // ~60fps

    calculateVisibleRange() {
        const scrollTop = this.shadowRoot.getElementById('scroller').scrollTop;
        this.startIndex = Math.floor(scrollTop / this.itemHeight);
        this.endIndex = Math.min(
            this.startIndex + Math.ceil(this.containerHeight / this.itemHeight) + 1,
            this.items.length
        );
    }

    renderVisibleItems() {
        const container = this.shadowRoot.getElementById('visible-items');
        container.innerHTML = '';

        for (let i = this.startIndex; i < this.endIndex; i++) {
            const item = this.items[i];
            const itemElement = this.createItemElement(item, i);
            container.appendChild(itemElement);
        }
    }

    createItemElement(item, index) {
        const div = document.createElement('div');
        div.className = 'item';
        div.textContent = `${index}: ${item.title}`;
        div.style.position = 'absolute';
        div.style.top = `${(index - this.startIndex) * this.itemHeight}px`;
        div.style.width = '100%';
        return div;
    }

    throttle(func, limit) {
        let inThrottle;
        return function() {
            const args = arguments;
            const context = this;
            if (!inThrottle) {
                func.apply(context, args);
                inThrottle = true;
                setTimeout(() => inThrottle = false, limit);
            }
        }
    }
}
```

## Заключение

HTML шаблоны и веб-компоненты предоставляют мощные инструменты для создания модульного, переиспользуемого и инкапсулированного кода. Современные возможности включают интеграцию с Web API, базами данных, WebSocket, а также продвинутые техники оптимизации и управления состоянием.

Они позволяют создавать сложные интерфейсы с изолированной логикой и стилями, улучшая структуру и поддерживаемость веб-приложений. Понимание этих технологий важно для современной веб-разработки.

Ключевые аспекты современных шаблонов и веб-компонентов:
1. Эффективное управление памятью и ресурсами
2. Оптимизация производительности рендеринга
3. Поддержка оффлайн-функциональности
4. Интеграция с современными веб-API
5. Поддержка доступности и семантики
6. Масштабируемость и поддерживаемость
7. Совместимость с различными браузерами
8. Следование современным веб-стандартам

Эти технологии становятся все более важными для создания современных, производительных и доступных веб-приложений, которые работают эффективно как в онлайн, так и в оффлайн режимах.

## Следующие темы
- [[HTML/Продвинутые темы/Кастомные элементы]]
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Производительность]]

## Теги
#html #web-components #templates #custom-elements #shadow-dom #web-development #frontend #modularity #indexeddb #websocket #performance #accessibility #virtual-scrolling #modern-web
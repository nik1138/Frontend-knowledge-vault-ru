# HTML Продвинутые темы: Кастомные элементы

Кастомные элементы (Custom Elements) - это часть Web Components API, которая позволяет разработчикам создавать свои собственные HTML элементы с определенным поведением и стилями. Это мощная технология для создания переиспользуемых компонентов.

## Основы кастомных элементов

### Создание кастомного элемента

Для создания кастомного элемента нужно:

1. Создать класс, расширяющий `HTMLElement` (или его подкласс)
2. Зарегистрировать элемент с помощью `customElements.define()`

```javascript
// simple-card.js
class SimpleCard extends HTMLElement {
    constructor() {
        super(); // Вызов конструктора родителя
        
        // Создание Shadow DOM для инкапсуляции
        this.attachShadow({ mode: 'open' });
        
        // Создание содержимого элемента
        this.shadowRoot.innerHTML = `
            <style>
                .card {
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    padding: 16px;
                    margin: 8px;
                    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
                    background-color: white;
                }
                
                .card-header {
                    font-weight: bold;
                    margin-bottom: 8px;
                    color: #333;
                }
                
                .card-content {
                    color: #666;
                    line-height: 1.5;
                }
            </style>
            
            <div class="card">
                <div class="card-header">Заголовок карточки</div>
                <div class="card-content">Содержимое карточки</div>
            </div>
        `;
    }
}

// Регистрация кастомного элемента
customElements.define('simple-card', SimpleCard);
```

### Использование кастомного элемента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Кастомные элементы</title>
</head>
<body>
    <h1>Пример кастомных элементов</h1>
    
    <!-- Использование кастомного элемента -->
    <simple-card></simple-card>
    
    <script src="simple-card.js"></script>
</body>
</html>
```

## Жизненный цикл кастомных элементов

Кастомные элементы имеют специальные методы жизненного цикла:

- `constructor` - вызывается при создании элемента
- `connectedCallback` - вызывается при добавлении элемента в DOM
- `disconnectedCallback` - вызывается при удалении элемента из DOM
- `adoptedCallback` - вызывается при перемещении элемента в новый документ
- `attributeChangedCallback` - вызывается при изменении наблюдаемого атрибута

```javascript
class LifecycleDemo extends HTMLElement {
    // Атрибуты, за которыми нужно наблюдать
    static get observedAttributes() {
        return ['title', 'visible'];
    }
    
    constructor() {
        super();
        console.log('1. Конструктор вызван');
        
        this.attachShadow({ mode: 'open' });
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    padding: 10px;
                    border: 1px solid #ccc;
                    margin: 5px;
                }
                
                :host([visible="false"]) {
                    display: none;
                }
                
                h3 {
                    margin: 0 0 10px 0;
                    color: #333;
                }
            </style>
            
            <h3 id="title">Заголовок по умолчанию</h3>
            <p>Содержимое элемента</p>
        `;
    }
    
    connectedCallback() {
        console.log('2. Элемент добавлен в DOM');
        
        // Дополнительная инициализация
        this.updateTitle();
    }
    
    disconnectedCallback() {
        console.log('3. Элемент удален из DOM');
        
        // Очистка ресурсов
        document.removeEventListener('click', this.boundClickHandler);
    }
    
    adoptedCallback() {
        console.log('4. Элемент перемещен в новый документ');
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        console.log(`5. Атрибут ${name} изменился с "${oldValue}" на "${newValue}"`);
        
        if (name === 'title') {
            this.updateTitle();
        } else if (name === 'visible') {
            this.toggleVisibility();
        }
    }
    
    updateTitle() {
        const title = this.getAttribute('title');
        const titleElement = this.shadowRoot.getElementById('title');
        if (title) {
            titleElement.textContent = title;
        }
    }
    
    toggleVisibility() {
        const isVisible = this.getAttribute('visible') !== 'false';
        this.style.display = isVisible ? 'block' : 'none';
    }
}

customElements.define('lifecycle-demo', LifecycleDemo);
```

## Передача данных в кастомные элементы

### Атрибуты

```javascript
class UserCard extends HTMLElement {
    static get observedAttributes() {
        return ['name', 'email', 'avatar', 'role'];
    }
    
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
    }
    
    connectedCallback() {
        this.render();
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        if (oldValue !== newValue) {
            this[name] = newValue;
            this.render();
        }
    }
    
    render() {
        const name = this.getAttribute('name') || 'Анонимный пользователь';
        const email = this.getAttribute('email') || 'email не указан';
        const avatar = this.getAttribute('avatar') || 'default-avatar.png';
        const role = this.getAttribute('role') || 'Пользователь';
        
        this.shadowRoot.innerHTML = `
            <style>
                .user-card {
                    display: flex;
                    align-items: center;
                    padding: 16px;
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    max-width: 400px;
                    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
                }
                
                .avatar {
                    width: 50px;
                    height: 50px;
                    border-radius: 50%;
                    margin-right: 16px;
                    object-fit: cover;
                }
                
                .info {
                    flex: 1;
                }
                
                .name {
                    font-weight: bold;
                    margin: 0 0 4px 0;
                    color: #333;
                }
                
                .email {
                    margin: 0 0 4px 0;
                    color: #666;
                    font-size: 0.9em;
                }
                
                .role {
                    font-size: 0.8em;
                    color: #888;
                    background-color: #f0f0f0;
                    padding: 2px 6px;
                    border-radius: 4px;
                    display: inline-block;
                }
            </style>
            
            <div class="user-card">
                <img class="avatar" src="${avatar}" alt="Аватар ${name}">
                <div class="info">
                    <div class="name">${name}</div>
                    <div class="email">${email}</div>
                    <div class="role">${role}</div>
                </div>
            </div>
        `;
    }
}

customElements.define('user-card', UserCard);
```

### Слоты (Slots)

Слоты позволяют передавать содержимое в кастомный элемент:

```javascript
class ModalDialog extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
    }
    
    connectedCallback() {
        this.render();
        this.bindEvents();
    }
    
    render() {
        this.shadowRoot.innerHTML = `
            <style>
                .overlay {
                    position: fixed;
                    top: 0;
                    left: 0;
                    width: 100%;
                    height: 100%;
                    background-color: rgba(0,0,0,0.5);
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    z-index: 1000;
                }
                
                .dialog {
                    background-color: white;
                    border-radius: 8px;
                    padding: 20px;
                    max-width: 500px;
                    width: 90%;
                    position: relative;
                }
                
                .header {
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    margin-bottom: 15px;
                    padding-bottom: 10px;
                    border-bottom: 1px solid #eee;
                }
                
                .title {
                    margin: 0;
                    font-size: 1.2em;
                    font-weight: bold;
                }
                
                .close-btn {
                    background: none;
                    border: none;
                    font-size: 1.5em;
                    cursor: pointer;
                    color: #999;
                }
                
                .content {
                    margin-bottom: 20px;
                }
                
                .footer {
                    display: flex;
                    justify-content: flex-end;
                    gap: 10px;
                }
                
                .btn {
                    padding: 8px 16px;
                    border: 1px solid #ccc;
                    border-radius: 4px;
                    cursor: pointer;
                }
                
                .btn-primary {
                    background-color: #007acc;
                    color: white;
                    border-color: #007acc;
                }
            </style>
            
            <div class="overlay">
                <div class="dialog">
                    <div class="header">
                        <h2 class="title"><slot name="title">Модальное окно</slot></h2>
                        <button class="close-btn" id="close-btn">&times;</button>
                    </div>
                    <div class="content">
                        <slot name="content">Содержимое модального окна</slot>
                    </div>
                    <div class="footer">
                        <slot name="footer">
                            <button class="btn" id="cancel-btn">Отмена</button>
                            <button class="btn btn-primary" id="confirm-btn">Подтвердить</button>
                        </slot>
                    </div>
                </div>
            </div>
        `;
    }
    
    bindEvents() {
        const closeBtn = this.shadowRoot.getElementById('close-btn');
        const cancelBtn = this.shadowRoot.getElementById('cancel-btn');
        
        closeBtn.addEventListener('click', () => this.close());
        cancelBtn.addEventListener('click', () => this.close());
        
        // Закрытие при клике на оверлей
        const overlay = this.shadowRoot.querySelector('.overlay');
        overlay.addEventListener('click', (e) => {
            if (e.target === overlay) {
                this.close();
            }
        });
    }
    
    open() {
        this.style.display = 'block';
    }
    
    close() {
        this.style.display = 'none';
        this.dispatchEvent(new CustomEvent('modal-closed', { bubbles: true }));
    }
}

customElements.define('modal-dialog', ModalDialog);
```

### Использование слотов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Кастомные элементы со слотами</title>
</head>
<body>
    <h1>Пример модального окна</h1>
    
    <button onclick="openModal()">Открыть модальное окно</button>
    
    <modal-dialog id="my-modal">
        <h3 slot="title">Заголовок модального окна</h3>
        <p slot="content">Это содержимое модального окна. Здесь может быть любой HTML контент.</p>
        <div slot="footer">
            <button class="btn" onclick="cancelModal()">Отмена</button>
            <button class="btn btn-primary" onclick="confirmModal()">Сохранить</button>
        </div>
    </modal-dialog>
    
    <script src="modal-dialog.js"></script>
    
    <script>
        function openModal() {
            document.getElementById('my-modal').style.display = 'block';
        }
        
        function cancelModal() {
            document.getElementById('my-modal').close();
        }
        
        function confirmModal() {
            alert('Действие подтверждено!');
            document.getElementById('my-modal').close();
        }
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
        
        // Добавляем стили и функциональность
        this.classList.add('fancy-button');
        
        // Добавляем ripple эффект
        this.addEventListener('click', this.createRipple);
    }
    
    connectedCallback() {
        // Добавляем атрибуты по умолчанию
        if (!this.hasAttribute('type')) {
            this.setAttribute('type', 'button');
        }
        
        // Добавляем обработчики
        this.addEventListener('mouseenter', this.onMouseEnter);
        this.addEventListener('mouseleave', this.onMouseLeave);
    }
    
    disconnectedCallback() {
        // Удаляем обработчики
        this.removeEventListener('mouseenter', this.onMouseEnter);
        this.removeEventListener('mouseleave', this.onMouseLeave);
    }
    
    createRipple = (event) => {
        const ripple = document.createElement('span');
        const rect = this.getBoundingClientRect();
        const size = Math.max(rect.width, rect.height);
        const x = event.clientX - rect.left - size / 2;
        const y = event.clientY - rect.top - size / 2;
        
        ripple.style.width = ripple.style.height = `${size}px`;
        ripple.style.left = `${x}px`;
        ripple.style.top = `${y}px`;
        ripple.classList.add('ripple');
        
        const rippleContainer = this.querySelector('.ripple-container') || 
                               (() => {
                                   const container = document.createElement('span');
                                   container.classList.add('ripple-container');
                                   this.appendChild(container);
                                   return container;
                               })();
        
        rippleContainer.appendChild(ripple);
        
        setTimeout(() => {
            ripple.remove();
        }, 600);
    }
    
    onMouseEnter = () => {
        this.style.transform = 'translateY(-2px)';
        this.style.boxShadow = '0 4px 8px rgba(0,0,0,0.2)';
    }
    
    onMouseLeave = () => {
        this.style.transform = 'translateY(0)';
        this.style.boxShadow = '0 2px 4px rgba(0,0,0,0.1)';
    }
}

// Регистрация с опцией extends
customElements.define('fancy-button', FancyButton, { extends: 'button' });
```

### Использование пользовательского встроенного элемента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Пользовательские встроенные элементы</title>
    <style>
        .fancy-button {
            position: relative;
            overflow: hidden;
            border: none;
            padding: 12px 24px;
            font-size: 16px;
            border-radius: 4px;
            background-color: #007acc;
            color: white;
            cursor: pointer;
            transition: all 0.3s ease;
            margin: 5px;
        }
        
        .fancy-button:hover {
            background-color: #005a9e;
        }
        
        .ripple-container {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            overflow: hidden;
            border-radius: inherit;
        }
        
        .ripple {
            position: absolute;
            border-radius: 50%;
            background-color: rgba(255,255,255,0.6);
            transform: scale(0);
            animation: ripple-animation 0.6s linear;
        }
        
        @keyframes ripple-animation {
            to {
                transform: scale(2);
                opacity: 0;
            }
        }
    </style>
</head>
<body>
    <h1>Пользовательские встроенные элементы</h1>
    
    <button is="fancy-button">Обычная кнопка</button>
    <button is="fancy-button" class="secondary">Второстепенная кнопка</button>
    <button is="fancy-button" disabled>Неактивная кнопка</button>
    
    <script src="fancy-button.js"></script>
</body>
</html>
```

## Сложный пример: Компонент табов

```javascript
class TabContainer extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
    }
    
    connectedCallback() {
        this.render();
        this.bindEvents();
    }
    
    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                }
                
                .tabs {
                    display: flex;
                    border-bottom: 1px solid #ddd;
                    margin-bottom: 10px;
                }
                
                .tab-button {
                    padding: 10px 20px;
                    cursor: pointer;
                    border: 1px solid transparent;
                    border-bottom: none;
                    background: #f0f0f0;
                    margin-right: 2px;
                    border-radius: 4px 4px 0 0;
                }
                
                .tab-button.active {
                    background: white;
                    border-color: #ddd;
                    border-bottom: 1px solid white;
                    margin-bottom: -1px;
                }
                
                .tab-panel {
                    display: none;
                    padding: 20px;
                    border: 1px solid #ddd;
                    border-radius: 0 4px 4px 4px;
                }
                
                .tab-panel.active {
                    display: block;
                }
            </style>
            
            <div class="tabs" part="tabs">
                <slot name="tab-buttons"></slot>
            </div>
            <div class="tab-panels" part="tab-panels">
                <slot name="tab-panels"></slot>
            </div>
        `;
    }
    
    bindEvents() {
        this.addEventListener('click', this.handleClick);
    }
    
    handleClick = (event) => {
        const tabButton = event.target.closest('[slot="tab-buttons"]');
        if (tabButton) {
            this.activateTab(tabButton.dataset.tab);
        }
    }
    
    activateTab(tabId) {
        // Снять активность с текущих вкладок
        this.querySelectorAll('[slot="tab-buttons"]').forEach(button => {
            button.classList.remove('active');
        });
        
        this.querySelectorAll('[slot="tab-panels"]').forEach(panel => {
            panel.classList.remove('active');
        });
        
        // Активировать выбранную вкладку
        const activeButton = this.querySelector(`[slot="tab-buttons"][data-tab="${tabId}"]`);
        const activePanel = this.querySelector(`[slot="tab-panels"][data-tab="${tabId}"]`);
        
        if (activeButton) activeButton.classList.add('active');
        if (activePanel) activePanel.classList.add('active');
        
        // Вызвать кастомное событие
        this.dispatchEvent(new CustomEvent('tab-changed', {
            detail: { tabId },
            bubbles: true
        }));
    }
    
    // Метод для программного переключения вкладки
    setActiveTab(tabId) {
        this.activateTab(tabId);
    }
}

customElements.define('tab-container', TabContainer);
```

### Использование компонента табов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Компонент табов</title>
</head>
<body>
    <h1>Компонент табов</h1>
    
    <tab-container id="my-tabs">
        <button slot="tab-buttons" data-tab="tab1" class="tab-button active">Вкладка 1</button>
        <button slot="tab-buttons" data-tab="tab2" class="tab-button">Вкладка 2</button>
        <button slot="tab-buttons" data-tab="tab3" class="tab-button">Вкладка 3</button>
        
        <div slot="tab-panels" data-tab="tab1" class="tab-panel active">
            <h3>Содержимое первой вкладки</h3>
            <p>Это содержимое первой вкладки. Здесь может быть любой контент.</p>
        </div>
        
        <div slot="tab-panels" data-tab="tab2" class="tab-panel">
            <h3>Содержимое второй вкладки</h3>
            <p>Это содержимое второй вкладки.</p>
        </div>
        
        <div slot="tab-panels" data-tab="tab3" class="tab-panel">
            <h3>Содержимое третьей вкладки</h3>
            <p>Это содержимое третьей вкладки.</p>
        </div>
    </tab-container>
    
    <div style="margin-top: 20px;">
        <button onclick="switchTab('tab1')">Переключить на вкладку 1</button>
        <button onclick="switchTab('tab2')">Переключить на вкладку 2</button>
        <button onclick="switchTab('tab3')">Переключить на вкладку 3</button>
    </div>
    
    <script src="tab-container.js"></script>
    
    <script>
        const tabs = document.getElementById('my-tabs');
        
        tabs.addEventListener('tab-changed', (e) => {
            console.log('Переключена вкладка:', e.detail.tabId);
        });
        
        function switchTab(tabId) {
            tabs.setActiveTab(tabId);
        }
    </script>
</body>
</html>
```

## Модульные кастомные элементы

### Создание модуля

```javascript
// components/counter-element.js
export class CounterElement extends HTMLElement {
    constructor() {
        super();
        
        this.attachShadow({ mode: 'open' });
        
        this.count = 0;
        this.min = parseInt(this.getAttribute('min')) || 0;
        this.max = parseInt(this.getAttribute('max')) || 100;
        
        this.render();
        this.bindEvents();
    }
    
    static get observedAttributes() {
        return ['min', 'max', 'value'];
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        if (name === 'min') {
            this.min = parseInt(newValue) || 0;
        } else if (name === 'max') {
            this.max = parseInt(newValue) || 100;
        } else if (name === 'value') {
            this.count = parseInt(newValue) || 0;
            this.updateDisplay();
        }
    }
    
    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: inline-block;
                    font-family: Arial, sans-serif;
                }
                
                .counter {
                    display: flex;
                    align-items: center;
                    gap: 10px;
                    padding: 10px;
                    border: 1px solid #ddd;
                    border-radius: 4px;
                }
                
                .count-display {
                    font-size: 1.2em;
                    font-weight: bold;
                    min-width: 40px;
                    text-align: center;
                }
                
                .btn {
                    padding: 5px 10px;
                    border: 1px solid #ccc;
                    background: #f0f0f0;
                    cursor: pointer;
                    border-radius: 3px;
                }
                
                .btn:disabled {
                    opacity: 0.5;
                    cursor: not-allowed;
                }
                
                .btn:hover:not(:disabled) {
                    background: #e0e0e0;
                }
            </style>
            
            <div class="counter">
                <button class="btn" id="decrement-btn" ?disabled="${this.count <= this.min}">-</button>
                <span class="count-display" id="count-display">${this.count}</span>
                <button class="btn" id="increment-btn" ?disabled="${this.count >= this.max}">+</button>
            </div>
        `;
    }
    
    bindEvents() {
        const decrementBtn = this.shadowRoot.getElementById('decrement-btn');
        const incrementBtn = this.shadowRoot.getElementById('increment-btn');
        
        decrementBtn.addEventListener('click', () => this.decrement());
        incrementBtn.addEventListener('click', () => this.increment());
    }
    
    increment() {
        if (this.count < this.max) {
            this.count++;
            this.updateDisplay();
            this.dispatchChange();
        }
    }
    
    decrement() {
        if (this.count > this.min) {
            this.count--;
            this.updateDisplay();
            this.dispatchChange();
        }
    }
    
    updateDisplay() {
        const display = this.shadowRoot.getElementById('count-display');
        const decrementBtn = this.shadowRoot.getElementById('decrement-btn');
        const incrementBtn = this.shadowRoot.getElementById('increment-btn');
        
        display.textContent = this.count;
        decrementBtn.disabled = this.count <= this.min;
        incrementBtn.disabled = this.count >= this.max;
    }
    
    dispatchChange() {
        this.dispatchEvent(new CustomEvent('counter-change', {
            detail: { value: this.count },
            bubbles: true
        }));
    }
    
    // Публичный метод для установки значения
    setValue(value) {
        const numValue = parseInt(value);
        if (!isNaN(numValue) && numValue >= this.min && numValue <= this.max) {
            this.count = numValue;
            this.updateDisplay();
            this.dispatchChange();
        }
    }
    
    // Публичный метод для получения значения
    getValue() {
        return this.count;
    }
}

// Регистрация элемента
customElements.define('counter-element', CounterElement);

// Экспорт для использования в других модулях
export default CounterElement;
```

### Использование модульного компонента

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Модульные кастомные элементы</title>
</head>
<body>
    <h1>Счетчики</h1>
    
    <counter-element id="counter1" min="0" max="10" value="5"></counter-element>
    <counter-element id="counter2" min="-5" max="5"></counter-element>
    
    <div style="margin-top: 20px;">
        <button onclick="resetCounters()">Сбросить все счетчики</button>
        <button onclick="logValues()">Показать значения</button>
    </div>
    
    <script type="module">
        import CounterElement from './components/counter-element.js';
        
        // Обработка событий
        document.querySelectorAll('counter-element').forEach(counter => {
            counter.addEventListener('counter-change', (e) => {
                console.log(`Счетчик ${counter.id} изменил значение на:`, e.detail.value);
            });
        });
        
        function resetCounters() {
            document.querySelectorAll('counter-element').forEach(counter => {
                counter.setValue(0);
            });
        }
        
        function logValues() {
            document.querySelectorAll('counter-element').forEach(counter => {
                console.log(`${counter.id}: ${counter.getValue()}`);
            });
        }
    </script>
</body>
</html>
```

## Современные возможности кастомных элементов

### Кастомные элементы с Web APIs
```javascript
// components/weather-widget.js
export class WeatherWidget extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        this.city = this.getAttribute('city') || 'Moscow';
        this.units = this.getAttribute('units') || 'metric';
        
        this.render();
        this.loadWeatherData();
    }

    static get observedAttributes() {
        return ['city', 'units'];
    }

    attributeChangedCallback(name, oldValue, newValue) {
        if (oldValue !== newValue) {
            if (name === 'city') {
                this.city = newValue;
            } else if (name === 'units') {
                this.units = newValue;
            }
            this.loadWeatherData();
        }
    }

    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    font-family: Arial, sans-serif;
                    max-width: 300px;
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    overflow: hidden;
                    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
                }

                .weather-header {
                    background-color: #007acc;
                    color: white;
                    padding: 15px;
                    text-align: center;
                }

                .weather-content {
                    padding: 15px;
                }

                .temperature {
                    font-size: 2em;
                    font-weight: bold;
                    text-align: center;
                    margin: 10px 0;
                }

                .weather-description {
                    text-align: center;
                    color: #666;
                    margin-bottom: 15px;
                }

                .weather-details {
                    display: grid;
                    grid-template-columns: repeat(2, 1fr);
                    gap: 10px;
                    text-align: center;
                }

                .detail-item {
                    padding: 8px;
                    background-color: #f9f9f9;
                    border-radius: 4px;
                }

                .loading {
                    text-align: center;
                    padding: 20px;
                    color: #666;
                }

                .error {
                    color: #d32f2f;
                    text-align: center;
                    padding: 20px;
                }
            </style>

            <div class="weather-header">
                <h3>Погода в <span id="city-name">${this.city}</span></h3>
            </div>
            <div class="weather-content">
                <div id="weather-display">
                    <div class="loading">Загрузка погоды...</div>
                </div>
            </div>
        `;
    }

    async loadWeatherData() {
        const loadingElement = this.shadowRoot.querySelector('.loading');
        const weatherDisplay = this.shadowRoot.getElementById('weather-display');

        try {
            // Используем Geolocation API для получения координат, если город не указан
            if (this.city.toLowerCase() === 'current') {
                const position = await this.getCurrentLocation();
                const weather = await this.getWeatherByCoords(position.coords.latitude, position.coords.longitude);
                this.displayWeather(weather);
            } else {
                const weather = await this.getWeatherByCity(this.city);
                this.displayWeather(weather);
            }
        } catch (error) {
            weatherDisplay.innerHTML = `<div class="error">Ошибка загрузки погоды: ${error.message}</div>`;
        }
    }

    async getCurrentLocation() {
        return new Promise((resolve, reject) => {
            if (!navigator.geolocation) {
                reject(new Error('Geolocation не поддерживается'));
                return;
            }

            navigator.geolocation.getCurrentPosition(resolve, reject, {
                enableHighAccuracy: true,
                timeout: 10000,
                maximumAge: 300000 // 5 минут
            });
        });
    }

    async getWeatherByCity(city) {
        // В реальном приложении здесь будет вызов реального API
        // Для демонстрации возвращаем mock данные
        return new Promise(resolve => {
            setTimeout(() => {
                resolve({
                    temperature: Math.floor(Math.random() * 30) - 5, // Случайная температура от -5 до 25
                    description: ['Облачно', 'Солнечно', 'Дождь', 'Снег'][Math.floor(Math.random() * 4)],
                    humidity: Math.floor(Math.random() * 100),
                    windSpeed: (Math.random() * 20).toFixed(1),
                    feelsLike: Math.floor(Math.random() * 30) - 5
                });
            }, 1000);
        });
    }

    async getWeatherByCoords(lat, lon) {
        // В реальном приложении здесь будет вызов реального API
        return this.getWeatherByCity('Current Location');
    }

    displayWeather(data) {
        const weatherDisplay = this.shadowRoot.getElementById('weather-display');
        
        weatherDisplay.innerHTML = `
            <div class="temperature">${data.temperature}°C</div>
            <div class="weather-description">${data.description}</div>
            <div class="weather-details">
                <div class="detail-item">
                    <div>Влажность</div>
                    <div>${data.humidity}%</div>
                </div>
                <div class="detail-item">
                    <div>Ветер</div>
                    <div>${data.windSpeed} м/с</div>
                </div>
                <div class="detail-item">
                    <div>Ощущается как</div>
                    <div>${data.feelsLike}°C</div>
                </div>
                <div class="detail-item">
                    <div>Обновлено</div>
                    <div>${new Date().toLocaleTimeString()}</div>
                </div>
            </div>
        `;
    }
}

customElements.define('weather-widget', WeatherWidget);
```

### Кастомные элементы с IndexedDB
```javascript
// components/todo-list.js
export class TodoList extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        this.dbName = 'TodoDB';
        this.version = 1;
        this.db = null;
        
        this.render();
        this.initDatabase();
    }

    async initDatabase() {
        const request = indexedDB.open(this.dbName, this.version);
        
        request.onerror = (event) => {
            console.error('Ошибка при открытии базы данных:', event.target.error);
        };
        
        request.onsuccess = (event) => {
            this.db = event.target.result;
            this.loadTodos();
        };
        
        request.onupgradeneeded = (event) => {
            this.db = event.target.result;
            
            if (!this.db.objectStoreNames.contains('todos')) {
                const store = this.db.createObjectStore('todos', { keyPath: 'id', autoIncrement: true });
                store.createIndex('title', 'title', { unique: false });
                store.createIndex('completed', 'completed', { unique: false });
                store.createIndex('createdAt', 'createdAt', { unique: false });
            }
        };
    }

    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    font-family: Arial, sans-serif;
                    max-width: 500px;
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    overflow: hidden;
                    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
                }

                .todo-header {
                    background-color: #007acc;
                    color: white;
                    padding: 15px;
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                }

                .todo-form {
                    padding: 15px;
                    border-bottom: 1px solid #eee;
                }

                .todo-input {
                    width: 100%;
                    padding: 10px;
                    border: 1px solid #ddd;
                    border-radius: 4px;
                    font-size: 16px;
                }

                .todo-list {
                    list-style: none;
                    margin: 0;
                    padding: 0;
                }

                .todo-item {
                    display: flex;
                    align-items: center;
                    padding: 12px 15px;
                    border-bottom: 1px solid #eee;
                }

                .todo-item:last-child {
                    border-bottom: none;
                }

                .todo-checkbox {
                    margin-right: 10px;
                }

                .todo-text {
                    flex: 1;
                }

                .todo-text.completed {
                    text-decoration: line-through;
                    color: #888;
                }

                .todo-delete {
                    background: none;
                    border: none;
                    color: #d32f2f;
                    cursor: pointer;
                    font-size: 1.2em;
                }

                .todo-stats {
                    padding: 10px 15px;
                    background-color: #f9f9f9;
                    text-align: center;
                    font-size: 0.9em;
                    color: #666;
                }
            </style>

            <div class="todo-header">
                <h3>Список дел</h3>
            </div>
            
            <div class="todo-form">
                <input type="text" 
                       class="todo-input" 
                       id="new-todo" 
                       placeholder="Добавить новое дело..."
                       aria-label="Добавить новое дело">
            </div>
            
            <ul class="todo-list" id="todo-list"></ul>
            
            <div class="todo-stats" id="todo-stats">Загрузка...</div>
        `;
        
        this.bindEvents();
    }

    bindEvents() {
        const input = this.shadowRoot.getElementById('new-todo');
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter' && input.value.trim()) {
                this.addTodo(input.value.trim());
                input.value = '';
            }
        });
    }

    async addTodo(title) {
        if (!this.db) return;
        
        const transaction = this.db.transaction(['todos'], 'readwrite');
        const store = transaction.objectStore('todos');
        
        const todo = {
            title: title,
            completed: false,
            createdAt: new Date().toISOString()
        };
        
        const request = store.add(todo);
        
        request.onsuccess = () => {
            this.loadTodos();
        };
        
        request.onerror = (e) => {
            console.error('Ошибка при добавлении дела:', e.target.error);
        };
    }

    async loadTodos() {
        if (!this.db) return;
        
        const transaction = this.db.transaction(['todos'], 'readonly');
        const store = transaction.objectStore('todos');
        const request = store.getAll();
        
        request.onsuccess = (e) => {
            const todos = e.target.result;
            this.renderTodos(todos);
        };
        
        request.onerror = (e) => {
            console.error('Ошибка при загрузке дел:', e.target.error);
        };
    }

    async toggleTodo(id) {
        if (!this.db) return;
        
        const transaction = this.db.transaction(['todos'], 'readwrite');
        const store = transaction.objectStore('todos');
        const request = store.get(id);
        
        request.onsuccess = (e) => {
            const todo = e.target.result;
            todo.completed = !todo.completed;
            
            const updateRequest = store.put(todo);
            updateRequest.onsuccess = () => {
                this.loadTodos();
            };
        };
    }

    async deleteTodo(id) {
        if (!this.db) return;
        
        const transaction = this.db.transaction(['todos'], 'readwrite');
        const store = transaction.objectStore('todos');
        const request = store.delete(id);
        
        request.onsuccess = () => {
            this.loadTodos();
        };
    }

    renderTodos(todos) {
        const list = this.shadowRoot.getElementById('todo-list');
        const stats = this.shadowRoot.getElementById('todo-stats');
        
        list.innerHTML = '';
        
        if (todos.length === 0) {
            stats.textContent = 'Нет дел';
            return;
        }
        
        const completedCount = todos.filter(todo => todo.completed).length;
        
        todos.forEach(todo => {
            const li = document.createElement('li');
            li.className = 'todo-item';
            li.innerHTML = `
                <input type="checkbox" 
                       class="todo-checkbox" 
                       ${todo.completed ? 'checked' : ''}>
                <span class="todo-text ${todo.completed ? 'completed' : ''}">${todo.title}</span>
                <button class="todo-delete" aria-label="Удалить дело: ${todo.title}">&times;</button>
            `;
            
            const checkbox = li.querySelector('.todo-checkbox');
            checkbox.addEventListener('change', () => this.toggleTodo(todo.id));
            
            const deleteBtn = li.querySelector('.todo-delete');
            deleteBtn.addEventListener('click', () => this.deleteTodo(todo.id));
            
            list.appendChild(li);
        });
        
        stats.textContent = `Всего: ${todos.length}, Выполнено: ${completedCount}, Осталось: ${todos.length - completedCount}`;
    }
}

customElements.define('todo-list', TodoList);
```

### Кастомные элементы с WebSocket
```javascript
// components/live-chat.js
export class LiveChat extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        this.ws = null;
        this.isConnected = false;
        this.username = this.getAttribute('username') || 'Аноним';
        
        this.render();
        this.connectToServer();
    }

    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    font-family: Arial, sans-serif;
                    max-width: 600px;
                    height: 500px;
                    border: 1px solid #ddd;
                    border-radius: 8px;
                    overflow: hidden;
                    box-shadow: 0 4px 8px rgba(0,0,0,0.1);
                }

                .chat-header {
                    background-color: #007acc;
                    color: white;
                    padding: 10px 15px;
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                }

                .chat-status {
                    font-size: 0.8em;
                    opacity: 0.8;
                }

                .chat-messages {
                    height: calc(100% - 120px);
                    overflow-y: auto;
                    padding: 15px;
                    background-color: #f9f9f9;
                }

                .message {
                    margin-bottom: 10px;
                    padding: 8px 12px;
                    border-radius: 18px;
                    max-width: 70%;
                    word-wrap: break-word;
                }

                .own-message {
                    background-color: #007acc;
                    color: white;
                    margin-left: auto;
                    text-align: right;
                }

                .other-message {
                    background-color: white;
                    border: 1px solid #ddd;
                }

                .message-info {
                    font-size: 0.7em;
                    opacity: 0.7;
                    margin-top: 3px;
                }

                .chat-input-area {
                    padding: 10px;
                    border-top: 1px solid #ddd;
                    background-color: white;
                    display: flex;
                }

                .chat-input {
                    flex: 1;
                    padding: 10px;
                    border: 1px solid #ddd;
                    border-radius: 20px;
                    margin-right: 10px;
                }

                .chat-send-btn {
                    background-color: #007acc;
                    color: white;
                    border: none;
                    border-radius: 20px;
                    padding: 10px 20px;
                    cursor: pointer;
                }

                .chat-send-btn:disabled {
                    background-color: #ccc;
                    cursor: not-allowed;
                }
            </style>

            <div class="chat-header">
                <h3>Чат</h3>
                <div class="chat-status" id="status">Подключение...</div>
            </div>
            
            <div class="chat-messages" id="messages"></div>
            
            <div class="chat-input-area">
                <input type="text" 
                       class="chat-input" 
                       id="message-input" 
                       placeholder="Введите сообщение..."
                       aria-label="Поле ввода сообщения">
                <button class="chat-send-btn" id="send-btn" disabled>Отправить</button>
            </div>
        `;
        
        this.bindEvents();
    }

    bindEvents() {
        const input = this.shadowRoot.getElementById('message-input');
        const sendBtn = this.shadowRoot.getElementById('send-btn');
        
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter' && input.value.trim() && this.isConnected) {
                this.sendMessage(input.value.trim());
                input.value = '';
            }
        });
        
        sendBtn.addEventListener('click', () => {
            if (input.value.trim() && this.isConnected) {
                this.sendMessage(input.value.trim());
                input.value = '';
            }
        });
    }

    connectToServer() {
        // В реальном приложении здесь будет реальный WebSocket URL
        // Для демонстрации используем mock-соединение
        try {
            // this.ws = new WebSocket('ws://localhost:8080/chat');
            
            // Для демонстрации эмулируем WebSocket
            this.ws = {
                send: (message) => {
                    // Эмуляция отправки сообщения
                    console.log('Отправлено:', message);
                },
                close: () => {
                    this.isConnected = false;
                    this.updateStatus('Отключено');
                    this.shadowRoot.getElementById('send-btn').disabled = true;
                }
            };
            
            // Эмуляция успешного подключения
            setTimeout(() => {
                this.isConnected = true;
                this.updateStatus('Подключено');
                this.shadowRoot.getElementById('send-btn').disabled = false;
                
                // Эмуляция получения сообщений
                setInterval(() => {
                    if (Math.random() > 0.7) { // 30% шанс получить сообщение
                        const users = ['Алексей', 'Мария', 'Иван', 'Елена'];
                        const messages = [
                            'Привет!', 'Как дела?', 'Что нового?', 'Рад тебя видеть!',
                            'Как продвигается проект?', 'Нужна помощь?'
                        ];
                        
                        const randomUser = users[Math.floor(Math.random() * users.length)];
                        const randomMessage = messages[Math.floor(Math.random() * messages.length)];
                        
                        this.displayMessage(randomUser, randomMessage, false);
                    }
                }, 5000);
            }, 1000);
            
            /*
            this.ws.onopen = () => {
                this.isConnected = true;
                this.updateStatus('Подключено');
                this.shadowRoot.getElementById('send-btn').disabled = false;
                
                // Отправить сообщение о присоединении
                this.sendSystemMessage(`${this.username} присоединился к чату`);
            };

            this.ws.onmessage = (event) => {
                const message = JSON.parse(event.data);
                this.displayMessage(message.username, message.text, message.username === this.username);
            };

            this.ws.onclose = () => {
                this.isConnected = false;
                this.updateStatus('Отключено');
                this.shadowRoot.getElementById('send-btn').disabled = true;
            };

            this.ws.onerror = (error) => {
                console.error('Ошибка WebSocket:', error);
                this.updateStatus('Ошибка подключения');
            };
            */
        } catch (error) {
            console.error('Ошибка подключения к чату:', error);
            this.updateStatus('Ошибка подключения');
        }
    }

    sendMessage(text) {
        if (!this.ws || !this.isConnected) return;
        
        const message = {
            username: this.username,
            text: text,
            timestamp: new Date().toISOString()
        };
        
        // this.ws.send(JSON.stringify(message));
        
        // Отображение собственного сообщения
        this.displayMessage(this.username, text, true);
    }

    displayMessage(username, text, isOwn) {
        const messagesContainer = this.shadowRoot.getElementById('messages');
        const messageDiv = document.createElement('div');
        messageDiv.className = `message ${isOwn ? 'own-message' : 'other-message'}`;
        
        const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
        
        messageDiv.innerHTML = `
            ${!isOwn ? `<strong>${username}</strong><br>` : ''}
            ${text}
            <div class="message-info">${time}</div>
        `;
        
        messagesContainer.appendChild(messageDiv);
        
        // Прокрутка вниз
        messagesContainer.scrollTop = messagesContainer.scrollHeight;
    }

    updateStatus(status) {
        this.shadowRoot.getElementById('status').textContent = status;
    }

    disconnectedCallback() {
        if (this.ws) {
            // this.ws.close();
        }
    }
}

customElements.define('live-chat', LiveChat);
```

## Практические примеры и шаблоны

### Шаблон для сложного кастомного элемента
```javascript
// components/advanced-component-template.js
export class AdvancedComponentTemplate extends HTMLElement {
    constructor() {
        super();
        
        // Подключение Shadow DOM
        this.attachShadow({ mode: 'open' });
        
        // Инициализация состояния
        this.state = {
            initialized: false,
            data: {},
            config: {
                theme: this.getAttribute('theme') || 'default',
                size: this.getAttribute('size') || 'medium',
                disabled: this.hasAttribute('disabled')
            }
        };
        
        // Подключение стилей
        this.setupStyles();
        
        // Рендер компонента
        this.render();
        
        // Подключение обработчиков событий
        this.bindEvents();
        
        // Установка наблюдения за атрибутами
        this.setupAttributeObservation();
        
        // Инициализация после подключения к DOM
        this.state.initialized = true;
    }

    static get observedAttributes() {
        return ['theme', 'size', 'disabled', 'data'];
    }

    attributeChangedCallback(name, oldValue, newValue) {
        if (oldValue !== newValue) {
            switch (name) {
                case 'theme':
                    this.state.config.theme = newValue;
                    this.updateTheme();
                    break;
                case 'size':
                    this.state.config.size = newValue;
                    this.updateSize();
                    break;
                case 'disabled':
                    this.state.config.disabled = newValue !== null;
                    this.updateDisabledState();
                    break;
                case 'data':
                    this.state.data = this.parseData(newValue);
                    this.updateData();
                    break;
            }
        }
    }

    connectedCallback() {
        // Вызывается при добавлении элемента в DOM
        this.onConnected();
    }

    disconnectedCallback() {
        // Вызывается при удалении элемента из DOM
        this.onDisconnected();
    }

    setupStyles() {
        // Вставка стилей в Shadow DOM
        const style = document.createElement('style');
        style.textContent = this.getStyles();
        this.shadowRoot.appendChild(style);
    }

    getStyles() {
        return `
            :host {
                display: block;
                font-family: Arial, sans-serif;
            }
            
            :host([disabled]) {
                opacity: 0.5;
                pointer-events: none;
            }
            
            .component-wrapper {
                position: relative;
                padding: 16px;
                border: 1px solid #ddd;
                border-radius: 8px;
            }
            
            .theme-default {
                background-color: #ffffff;
                color: #333;
            }
            
            .theme-dark {
                background-color: #333;
                color: #fff;
            }
            
            .size-small {
                font-size: 14px;
            }
            
            .size-medium {
                font-size: 16px;
            }
            
            .size-large {
                font-size: 18px;
            }
            
            .loading {
                display: flex;
                justify-content: center;
                align-items: center;
                padding: 20px;
            }
            
            .error {
                color: #d32f2f;
                text-align: center;
                padding: 20px;
            }
        `;
    }

    render() {
        this.shadowRoot.innerHTML = `
            <div class="component-wrapper">
                <div class="content">
                    <slot name="header"></slot>
                    <div id="main-content">Содержимое компонента</div>
                    <slot name="footer"></slot>
                </div>
            </div>
        `;
    }

    bindEvents() {
        // Подключение общих обработчиков событий
        this.addEventListener('click', this.handleClick);
        this.addEventListener('focus', this.handleFocus);
        this.addEventListener('blur', this.handleBlur);
    }

    setupAttributeObservation() {
        // Дополнительная логика наблюдения за атрибутами
    }

    // Методы для обработки событий
    handleClick = (event) => {
        // Обработка клика
        if (this.state.config.disabled) return;
        
        this.dispatchEvent(new CustomEvent('component-click', {
            detail: { event, state: this.state },
            bubbles: true,
            composed: true
        }));
    }

    handleFocus = (event) => {
        this.shadowRoot.querySelector('.component-wrapper').classList.add('focused');
    }

    handleBlur = (event) => {
        this.shadowRoot.querySelector('.component-wrapper').classList.remove('focused');
    }

    // Методы обновления состояния
    updateTheme() {
        const wrapper = this.shadowRoot.querySelector('.component-wrapper');
        wrapper.className = `component-wrapper theme-${this.state.config.theme}`;
    }

    updateSize() {
        const wrapper = this.shadowRoot.querySelector('.component-wrapper');
        wrapper.classList.remove('size-small', 'size-medium', 'size-large');
        wrapper.classList.add(`size-${this.state.config.size}`);
    }

    updateDisabledState() {
        if (this.state.config.disabled) {
            this.setAttribute('disabled', '');
        } else {
            this.removeAttribute('disabled');
        }
    }

    parseData(dataStr) {
        try {
            return JSON.parse(dataStr) || {};
        } catch {
            return {};
        }
    }

    updateData() {
        // Обновление содержимого на основе новых данных
        const content = this.shadowRoot.getElementById('main-content');
        content.textContent = JSON.stringify(this.state.data);
    }

    // Жизненные циклы
    onConnected() {
        console.log('Компонент подключен к DOM');
        
        // Выполнить дополнительную инициализацию
        this.loadInitialData();
    }

    onDisconnected() {
        console.log('Компонент отключен от DOM');
        
        // Очистить ресурсы
        this.cleanup();
    }

    async loadInitialData() {
        // Загрузка начальных данных
        try {
            // this.state.data = await this.fetchData();
            this.updateData();
        } catch (error) {
            this.showError(error.message);
        }
    }

    showError(message) {
        const content = this.shadowRoot.getElementById('main-content');
        content.innerHTML = `<div class="error">Ошибка: ${message}</div>`;
    }

    cleanup() {
        // Очистка ресурсов
        this.removeEventListener('click', this.handleClick);
        this.removeEventListener('focus', this.handleFocus);
        this.removeEventListener('blur', this.handleBlur);
    }

    // Публичные методы для взаимодействия извне
    setData(data) {
        this.state.data = { ...this.state.data, ...data };
        this.updateData();
    }

    getData() {
        return { ...this.state.data };
    }

    setState(newState) {
        this.state = { ...this.state, ...newState };
    }

    getState() {
        return { ...this.state };
    }
}

customElements.define('advanced-component-template', AdvancedComponentTemplate);
```

### Кастомный элемент с анимациями
```javascript
// components/animated-card.js
export class AnimatedCard extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        this.animationDuration = parseInt(this.getAttribute('animation-duration')) || 300;
        this.triggerAnimation = this.getAttribute('trigger-animation') || 'hover';
        
        this.render();
        this.bindEvents();
    }

    static get observedAttributes() {
        return ['animation-duration', 'trigger-animation', 'animate'];
    }

    attributeChangedCallback(name, oldValue, newValue) {
        if (name === 'animation-duration') {
            this.animationDuration = parseInt(newValue) || 300;
        } else if (name === 'trigger-animation') {
            this.triggerAnimation = newValue;
            this.updateTrigger();
        } else if (name === 'animate') {
            if (newValue !== null) {
                this.performAnimation();
            }
        }
    }

    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: inline-block;
                    margin: 10px;
                }

                .card {
                    width: 200px;
                    height: 250px;
                    perspective: 1000px;
                }

                .card-inner {
                    position: relative;
                    width: 100%;
                    height: 100%;
                    text-align: center;
                    transition: transform ${this.animationDuration}ms;
                    transform-style: preserve-3d;
                }

                .card-front, .card-back {
                    position: absolute;
                    width: 100%;
                    height: 100%;
                    -webkit-backface-visibility: hidden;
                    backface-visibility: hidden;
                    border-radius: 8px;
                    display: flex;
                    flex-direction: column;
                    justify-content: center;
                    align-items: center;
                    padding: 20px;
                    box-sizing: border-box;
                }

                .card-front {
                    background: linear-gradient(45deg, #007acc, #00bcd4);
                    color: white;
                }

                .card-back {
                    background: linear-gradient(45deg, #4caf50, #8bc34a);
                    color: white;
                    transform: rotateY(180deg);
                }

                .card.flipped .card-inner {
                    transform: rotateY(180deg);
                }

                .card.pulse .card-inner {
                    animation: pulse ${this.animationDuration}ms;
                }

                .card.shake .card-inner {
                    animation: shake ${this.animationDuration}ms;
                }

                .card.fade .card-inner {
                    animation: fade ${this.animationDuration}ms;
                }

                @keyframes flip {
                    0% { transform: rotateY(0deg); }
                    100% { transform: rotateY(180deg); }
                }

                @keyframes pulse {
                    0% { transform: scale(1); }
                    50% { transform: scale(1.05); }
                    100% { transform: scale(1); }
                }

                @keyframes shake {
                    0%, 100% { transform: translateX(0); }
                    25% { transform: translateX(-5px); }
                    75% { transform: translateX(5px); }
                }

                @keyframes fade {
                    0% { opacity: 1; }
                    50% { opacity: 0.5; }
                    100% { opacity: 1; }
                }

                .card-title {
                    font-size: 1.2em;
                    margin-bottom: 10px;
                }

                .card-content {
                    font-size: 0.9em;
                }
            </style>

            <div class="card" id="card">
                <div class="card-inner">
                    <div class="card-front">
                        <div class="card-title">Передняя сторона</div>
                        <div class="card-content">Наведите курсор для анимации</div>
                    </div>
                    <div class="card-back">
                        <div class="card-title">Задняя сторона</div>
                        <div class="card-content">Содержимое задней стороны</div>
                    </div>
                </div>
            </div>
        `;
    }

    bindEvents() {
        const card = this.shadowRoot.getElementById('card');
        
        if (this.triggerAnimation === 'hover') {
            card.addEventListener('mouseenter', () => this.performAnimation());
        } else if (this.triggerAnimation === 'click') {
            card.addEventListener('click', () => this.performAnimation());
        }
    }

    updateTrigger() {
        const card = this.shadowRoot.getElementById('card');
        card.removeEventListener('mouseenter', this.performAnimation);
        card.removeEventListener('click', this.performAnimation);
        
        if (this.triggerAnimation === 'hover') {
            card.addEventListener('mouseenter', () => this.performAnimation());
        } else if (this.triggerAnimation === 'click') {
            card.addEventListener('click', () => this.performAnimation());
        }
    }

    performAnimation() {
        const card = this.shadowRoot.getElementById('card');
        const animationType = this.getAttribute('animation-type') || 'flip';
        
        card.classList.remove('flipped', 'pulse', 'shake', 'fade');
        
        setTimeout(() => {
            card.classList.add(animationType);
        }, 10);
        
        setTimeout(() => {
            card.classList.remove(animationType);
        }, this.animationDuration + 100);
    }

    // Публичный метод для триггера анимации
    triggerAnimation() {
        this.performAnimation();
    }
}

customElements.define('animated-card', AnimatedCard);
```

## Проверка и тестирование кастомных элементов

### Инструменты для тестирования:
1. **Web Component Tester** - фреймворк для тестирования веб-компонентов
2. **Jest** - для unit-тестирования
3. **Playwright** - для end-to-end тестирования
4. **Chrome DevTools** - для отладки Shadow DOM
5. **Selenium WebDriver** - для автоматизированного тестирования

### Проверка доступности:
1. Проверьте, что элементы внутри Shadow DOM доступны для скринридеров
2. Убедитесь, что ARIA-атрибуты работают корректно
3. Проверьте клавиатурную навигацию
4. Убедитесь, что фокус управляется правильно

### Проверка производительности:
1. Используйте Performance API для измерения времени рендеринга
2. Проверьте использование памяти
3. Оцените время подключения к DOM
4. Проверьте частоту обновлений

## Лучшие практики кастомных элементов

### 1. Архитектура и организация
```javascript
// components/ui-kit.js - объединение нескольких компонентов
import { Button } from './button.js';
import { Input } from './input.js';
import { Card } from './card.js';
import { Modal } from './modal.js';

// Регистрация всех компонентов
export const registerComponents = () => {
    customElements.define('ui-button', Button);
    customElements.define('ui-input', Input);
    customElements.define('ui-card', Card);
    customElements.define('ui-modal', Modal);
};

// Условная регистрация (проверка на существование)
export const safeRegister = (name, constructor) => {
    if (!customElements.get(name)) {
        customElements.define(name, constructor);
    }
};
```

### 2. Управление состоянием
```javascript
// components/state-manager.js
export class StateManager {
    constructor(initialState = {}) {
        this.state = { ...initialState };
        this.listeners = [];
    }

    subscribe(listener) {
        this.listeners.push(listener);
        return () => {
            this.listeners = this.listeners.filter(l => l !== listener);
        };
    }

    setState(newState) {
        this.state = { ...this.state, ...newState };
        this.notifyListeners();
    }

    getState() {
        return { ...this.state };
    }

    notifyListeners() {
        this.listeners.forEach(listener => listener(this.state));
    }
}

// Использование в компоненте
export class StatefulComponent extends HTMLElement {
    constructor() {
        super();
        this.stateManager = new StateManager({ count: 0 });
        this.attachShadow({ mode: 'open' });
        this.render();
        
        this.stateManager.subscribe((state) => {
            this.updateView(state);
        });
    }

    updateView(state) {
        this.shadowRoot.getElementById('count').textContent = state.count;
    }

    increment() {
        const currentState = this.stateManager.getState();
        this.stateManager.setState({ count: currentState.count + 1 });
    }
}
```

### 3. Стилизация и темизация
```javascript
// components/themed-component.js
export class ThemedComponent extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
        
        // Поддержка CSS Custom Properties
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    --primary-color: #007acc;
                    --secondary-color: #6c757d;
                    --background-color: #ffffff;
                    --text-color: #333333;
                    --border-radius: 4px;
                    --spacing: 8px;
                }
                
                .container {
                    background-color: var(--background-color);
                    color: var(--text-color);
                    border: 1px solid var(--primary-color);
                    border-radius: var(--border-radius);
                    padding: var(--spacing);
                    margin: var(--spacing);
                }
                
                .primary-button {
                    background-color: var(--primary-color);
                    color: white;
                    border: none;
                    border-radius: var(--border-radius);
                    padding: calc(var(--spacing) * 1.5) calc(var(--spacing) * 2);
                    cursor: pointer;
                }
            </style>
            
            <div class="container">
                <button class="primary-button">Темизированный компонент</button>
            </div>
        `;
    }
}
```

## Заключение

Кастомные элементы предоставляют мощный способ создания переиспользуемых компонентов с инкапсулированным поведением и стилями. Они позволяют:

- Создавать собственные HTML теги
- Инкапсулировать функциональность и стили
- Обеспечивать повторное использование кода
- Взаимодействовать с существующими HTML элементами
- Использовать современные веб-стандарты
- Интегрироваться с современными веб-API
- Создавать сложные интерактивные компоненты

Современные возможности кастомных элементов включают интеграцию с Web API, базами данных, WebSocket, а также продвинутые техники стилизации и управления состоянием. Эти элементы становятся все более важными для создания масштабируемых и поддерживаемых веб-приложений.

Ключевые аспекты успешного использования кастомных элементов:
1. Понимание жизненного цикла элементов
2. Правильная инкапсуляция с Shadow DOM
3. Эффективное управление состоянием
4. Поддержка доступности
5. Оптимизация производительности
6. Тестирование и отладка
7. Следование лучшим практикам
8. Совместимость с различными браузерами

Эти практики позволяют создавать компоненты, которые не только функциональны, но и надежны, доступны и легко интегрируемы в современные веб-приложения.

## Следующие темы
- [[HTML/Продвинутые темы/Шаблоны и веб-компоненты]]
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Производительность]]

## Теги
#html #custom-elements #web-components #javascript #frontend #components #modularity #web-api #indexeddb #websocket #accessibility #performance #best-practices
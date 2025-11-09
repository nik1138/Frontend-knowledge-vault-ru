# Современные методы манипуляции DOM в JavaScript

## Введение

DOM (Document Object Model) — это программный интерфейс для HTML и XML документов. Современные методы манипуляции DOM обеспечивают более эффективную, безопасную и удобную работу с элементами страницы.

## Современные методы выбора элементов

### querySelector и querySelectorAll

Современные методы, использующие CSS-селекторы:

```javascript
// Выбор одного элемента
const header = document.querySelector('header');
const mainButton = document.querySelector('.btn-primary');
const firstListItem = document.querySelector('ul li:first-child');
const dataElement = document.querySelector('[data-role="navigation"]');

// Выбор всех подходящих элементов
const allButtons = document.querySelectorAll('button');
const menuItems = document.querySelectorAll('.menu-item');
const externalLinks = document.querySelectorAll('a[href^="http"]');
```

### dataset API

Современный способ работы с data-атрибутами:

```javascript
// HTML: <div id="user-card" data-user-id="123" data-role="admin"></div>

const userCard = document.getElementById('user-card');

// Чтение data-атрибутов
console.log(userCard.dataset.userId); // "123"
console.log(userCard.dataset.role); // "admin"

// Установка data-атрибутов
userCard.dataset.lastSeen = "2023-10-15";
userCard.dataset.isActive = true;

// Результат: <div data-user-id="123" data-role="admin" data-last-seen="2023-10-15" data-is-active="true">
```

## Создание и вставка элементов

### createElement и современные методы

```javascript
// Классический способ
const oldDiv = document.createElement('div');
oldDiv.className = 'card';
oldDiv.textContent = 'Привет, мир!';

// Современный способ с setAttribute
const newDiv = document.createElement('div');
newDiv.setAttribute('class', 'card modern');
newDiv.setAttribute('data-created', new Date().toISOString());
newDiv.textContent = 'Привет, современный мир!';

// Использование шаблонных строк для HTML
function createUserCard(user) {
    return `
        <div class="user-card" data-user-id="${user.id}">
            <h3>${user.name}</h3>
            <p>${user.email}</p>
            <button class="btn-delete" data-action="delete">Удалить</button>
        </div>
    `;
}

// Безопасная вставка HTML (предотвращение XSS)
function safeInsertHTML(element, htmlString) {
    const template = document.createElement('template');
    template.innerHTML = htmlString.trim();
    element.appendChild(template.content);
}
```

### insertAdjacentHTML и insertAdjacentElement

Более гибкие методы вставки:

```javascript
const container = document.querySelector('.container');

// Вставка HTML
container.insertAdjacentHTML('beforeend', '<p>Новый абзац</p>');
container.insertAdjacentHTML('afterbegin', '<h2>Заголовок</h2>');

// Возможные позиции:
// 'beforebegin' - перед элементом
// 'afterbegin' - внутрь в начало
// 'beforeend' - внутрь в конец
// 'afterend' - после элемента

// Вставка готового элемента
const newElement = document.createElement('div');
newElement.textContent = 'Новый элемент';
container.insertAdjacentElement('beforeend', newElement);
```

## Современные методы изменения DOM

### classList API

Управление классами без строковой манипуляции:

```javascript
const element = document.querySelector('.my-element');

// Добавление класса
element.classList.add('active');

// Удаление класса
element.classList.remove('inactive');

// Переключение класса
element.classList.toggle('highlighted');

// Проверка наличия класса
if (element.classList.contains('special')) {
    console.log('Элемент имеет класс special');
}

// Добавление нескольких классов
element.classList.add('class1', 'class2', 'class3');

// Замена класса
element.classList.replace('old-class', 'new-class');
```

### dataset и атрибуты

```javascript
const button = document.querySelector('#my-button');

// Работа с data-атрибутами
button.dataset.loading = true;
button.dataset.action = 'submit';

// Работа с обычными атрибутами
button.setAttribute('aria-label', 'Отправить форму');
button.setAttribute('disabled', '');
button.removeAttribute('disabled');

// Проверка наличия атрибута
if (button.hasAttribute('data-loading')) {
    console.log('Кнопка в состоянии загрузки');
}
```

## Современные события и делегирование

### addEventListener с опциями

```javascript
const container = document.querySelector('.container');

// Использование опций события
container.addEventListener('click', handleClick, {
    once: true,           // Обработчик сработает только один раз
    passive: true,        // Событие не будет вызывать preventDefault
    capture: false        // Фаза захвата/всплытия
});

// Делегирование событий
container.addEventListener('click', function(event) {
    // Проверяем, что клик был по кнопке удаления
    if (event.target.matches('button[data-action="delete"]')) {
        const userId = event.target.closest('.user-card').dataset.userId;
        deleteUser(userId);
    }
    
    // Проверяем клик по ссылке
    if (event.target.tagName === 'A') {
        event.preventDefault();
        handleLinkClick(event.target.href);
    }
});
```

### Современные типы событий

```javascript
// Intersection Observer API - для отслеживания видимости элементов
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            // Элемент появился в области видимости
            loadMoreContent();
            observer.unobserve(entry.target);
        }
    });
}, {
    threshold: 0.1 // 10% элемента должно быть видно
});

observer.observe(document.querySelector('.lazy-load-section'));

// Resize Observer API - для отслеживания изменения размеров
const resizeObserver = new ResizeObserver(entries => {
    for (let entry of entries) {
        const { width, height } = entry.contentRect;
        handleResize(width, height);
    }
});

resizeObserver.observe(document.querySelector('.resizable-element'));
```

## Работа с формами

### Современные методы работы с формами

```javascript
const form = document.querySelector('#user-form');

// FormData API
form.addEventListener('submit', function(event) {
    event.preventDefault();
    
    const formData = new FormData(form);
    
    // Получение данных
    const name = formData.get('name');
    const email = formData.get('email');
    
    // Перебор всех данных
    for (const [key, value] of formData.entries()) {
        console.log(`${key}: ${value}`);
    }
    
    // Отправка данных
    submitForm(formData);
});

// HTMLFormElement.requestSubmit() - современный способ отправки
function submitFormProgrammatically() {
    const form = document.querySelector('#my-form');
    form.requestSubmit(); // Вызывает submit событие
}

// Проверка валидности формы
function validateForm(form) {
    const isValid = form.checkValidity();
    
    if (!isValid) {
        // Показываем ошибки валидации
        const invalidFields = form.querySelectorAll(':invalid');
        invalidFields.forEach(field => {
            field.classList.add('error');
        });
    }
    
    return isValid;
}
```

## Производительность и оптимизация

### Virtual DOM подход (концепция)

```javascript
// Пример виртуального DOM (упрощенная реализация)
class VirtualDOM {
    constructor(tag, props = {}, children = []) {
        this.tag = tag;
        this.props = props;
        this.children = children;
    }
    
    render() {
        const element = document.createElement(this.tag);
        
        // Установка атрибутов
        Object.keys(this.props).forEach(key => {
            if (key.startsWith('on')) {
                // Обработчики событий
                const event = key.slice(2).toLowerCase();
                element.addEventListener(event, this.props[key]);
            } else if (key === 'className') {
                element.className = this.props[key];
            } else {
                element.setAttribute(key, this.props[key]);
            }
        });
        
        // Добавление дочерних элементов
        this.children.forEach(child => {
            if (typeof child === 'string') {
                element.appendChild(document.createTextNode(child));
            } else {
                element.appendChild(child.render());
            }
        });
        
        return element;
    }
}

// Использование
const vdom = new VirtualDOM('div', { className: 'container' }, [
    new VirtualDOM('h1', {}, ['Заголовок']),
    new VirtualDOM('p', {}, ['Текст параграфа'])
]);

document.body.appendChild(vdom.render());
```

### Batch DOM updates

```javascript
// Группировка обновлений DOM
function batchDOMUpdates(updates) {
    // Отложить макрозадачу для выполнения всех обновлений
    Promise.resolve().then(() => {
        updates.forEach(update => update());
    });
}

// Использование requestAnimationFrame для синхронизации с рендерингом
function updateDOMSmoothly(updates) {
    requestAnimationFrame(() => {
        updates.forEach(update => update());
    });
}

// Пример использования
const updates = [
    () => element1.style.display = 'block',
    () => element2.textContent = 'Новый текст',
    () => element3.classList.add('updated')
];

batchDOMUpdates(updates);
```

## Современные API для работы с DOM

### Custom Elements (Web Components)

```javascript
class UserCard extends HTMLElement {
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
    }
    
    connectedCallback() {
        const name = this.getAttribute('name');
        const email = this.getAttribute('email');
        
        this.shadowRoot.innerHTML = `
            <style>
                .card {
                    border: 1px solid #ccc;
                    padding: 10px;
                    margin: 10px;
                    border-radius: 4px;
                }
            </style>
            <div class="card">
                <h3>${name}</h3>
                <p>${email}</p>
            </div>
        `;
    }
    
    static get observedAttributes() {
        return ['name', 'email'];
    }
    
    attributeChangedCallback(name, oldValue, newValue) {
        // Обновление при изменении атрибутов
        this.connectedCallback();
    }
}

customElements.define('user-card', UserCard);

// Использование в HTML
// <user-card name="Иван" email="ivan@example.com"></user-card>
```

### Shadow DOM

```javascript
// Создание компонента с изолированной областью видимости
class StyledComponent extends HTMLElement {
    constructor() {
        super();
        const shadow = this.attachShadow({ mode: 'closed' });
        
        const style = document.createElement('style');
        style.textContent = `
            :host {
                display: block;
                padding: 10px;
                border: 1px solid #ddd;
            }
            
            .content {
                color: blue;
            }
        `;
        
        const content = document.createElement('div');
        content.className = 'content';
        content.textContent = 'Содержимое с изолированными стилями';
        
        shadow.appendChild(style);
        shadow.appendChild(content);
    }
}

customElements.define('styled-component', StyledComponent);
```

## Лучшие практики

### 1. Делегирование событий

```javascript
// Плохо - каждый элемент имеет свой обработчик
document.querySelectorAll('.button').forEach(button => {
    button.addEventListener('click', handleClick);
});

// Хорошо - один обработчик для всех элементов
document.addEventListener('click', function(event) {
    if (event.target.matches('.button')) {
        handleClick(event);
    }
});
```

### 2. Использование DocumentFragment

```javascript
// Плохо - множество операций DOM
const container = document.querySelector('.container');
for (let i = 0; i < 100; i++) {
    container.innerHTML += `<div>Элемент ${i}</div>`; // Много перерисовок
}

// Хорошо - одна операция DOM
const fragment = document.createDocumentFragment();
for (let i = 0; i < 100; i++) {
    const div = document.createElement('div');
    div.textContent = `Элемент ${i}`;
    fragment.appendChild(div);
}
container.appendChild(fragment);
```

### 3. Очистка событий

```javascript
class Component {
    constructor(element) {
        this.element = element;
        this.boundHandler = this.handleClick.bind(this);
        this.init();
    }
    
    init() {
        this.element.addEventListener('click', this.boundHandler);
    }
    
    destroy() {
        // Очистка событий
        this.element.removeEventListener('click', this.boundHandler);
        this.element = null;
    }
    
    handleClick(event) {
        // Обработка клика
    }
}
```

## Примеры из промышленной разработки

### Динамическая загрузка контента

```javascript
class DynamicContentLoader {
    constructor(containerSelector) {
        this.container = document.querySelector(containerSelector);
        this.isLoading = false;
    }
    
    async loadContent(url) {
        if (this.isLoading) return;
        
        this.showLoader();
        this.isLoading = true;
        
        try {
            const response = await fetch(url);
            const html = await response.text();
            
            // Безопасная вставка HTML
            this.container.innerHTML = '';
            const tempDiv = document.createElement('div');
            tempDiv.innerHTML = html;
            
            // Перемещение элементов в контейнер
            while (tempDiv.firstChild) {
                this.container.appendChild(tempDiv.firstChild);
            }
            
        } catch (error) {
            this.showError('Ошибка загрузки контента');
        } finally {
            this.hideLoader();
            this.isLoading = false;
        }
    }
    
    showLoader() {
        this.container.innerHTML = '<div class="loading">Загрузка...</div>';
    }
    
    hideLoader() {
        this.container.querySelector('.loading')?.remove();
    }
    
    showError(message) {
        this.container.innerHTML = `<div class="error">${message}</div>`;
    }
}
```

### Управление фокусом для доступности

```javascript
class FocusManager {
    static trapFocus(element) {
        const focusableElements = element.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        
        const firstFocusable = focusableElements[0];
        const lastFocusable = focusableElements[focusableElements.length - 1];
        
        element.addEventListener('keydown', function(event) {
            if (event.key === 'Tab') {
                if (event.shiftKey) {
                    if (document.activeElement === firstFocusable) {
                        lastFocusable.focus();
                        event.preventDefault();
                    }
                } else {
                    if (document.activeElement === lastFocusable) {
                        firstFocusable.focus();
                        event.preventDefault();
                    }
                }
            }
        });
        
        firstFocusable.focus();
    }
    
    static returnFocus(previousElement) {
        setTimeout(() => previousElement.focus(), 0);
    }
}
```

## Ссылки на связанные темы
- [[js/basics/events]] - События в JavaScript
- [[js/testing/dom]] - Тестирование DOM-элементов
- [[html/accessibility]] - Доступность HTML
- [[css/styling/components]] - Компонентная стилизация

#programming #javascript #dom #web-components #best-practices
# Современные методы работы с событиями в JavaScript

## Введение

События в JavaScript — это основной способ взаимодействия с пользователем и реакцией на действия в браузере. Современные методы работы с событиями обеспечивают более гибкую, безопасную и производительную обработку событий.

## addEventListener с современными опциями

### Основные опции

```javascript
// Классический способ
element.addEventListener('click', handler);

// Современный способ с опциями
element.addEventListener('click', handler, {
    once: true,      // Обработчик сработает только один раз
    passive: true,   // Событие не будет вызывать preventDefault
    capture: false,  // Фаза захвата/всплытия
    signal: abortSignal // Возможность отмены обработчика
});

// Использование AbortController для отмены обработчиков
const controller = new AbortController();

element.addEventListener('click', () => {
    console.log('Клик!');
}, { signal: controller.signal });

// Позже можно отменить все обработчики
controller.abort();
```

### Пассивные события для производительности

```javascript
// Для событий, которые не вызывают preventDefault
element.addEventListener('touchstart', handleTouchStart, { passive: true });
element.addEventListener('wheel', handleScroll, { passive: true });

// Полезно для оптимизации прокрутки
function handleScroll(event) {
    // Не вызываем event.preventDefault()
    updateScrollPosition(event.deltaY);
}
```

## Современные типы событий

### Pointer Events

Современная альтернатива mouse и touch событиям:

```javascript
const element = document.querySelector('#draggable');

element.addEventListener('pointerdown', handlePointerDown);
element.addEventListener('pointermove', handlePointerMove);
element.addEventListener('pointerup', handlePointerUp);

function handlePointerDown(event) {
    // Работает как для мыши, так и для сенсорных устройств
    event.preventDefault();
    
    // Уникальный ID указателя
    const pointerId = event.pointerId;
    
    // Тип указателя: 'mouse', 'pen', 'touch'
    const pointerType = event.pointerType;
    
    // Начало перетаскивания
    startDragging(pointerId);
}

function handlePointerMove(event) {
    if (isDragging) {
        updatePosition(event.clientX, event.clientY);
    }
}

function handlePointerUp(event) {
    stopDragging();
}
```

### Touch Events

Для работы с мультитач-взаимодействиями:

```javascript
const touchArea = document.querySelector('#touch-area');

touchArea.addEventListener('touchstart', handleTouchStart);
touchArea.addEventListener('touchmove', handleTouchMove);
touchArea.addEventListener('touchend', handleTouchEnd);

function handleTouchStart(event) {
    event.preventDefault();
    
    const touches = event.touches;
    for (let i = 0; i < touches.length; i++) {
        const touch = touches[i];
        // Обработка каждого прикосновения
        console.log(`Палец ${i}: x=${touch.clientX}, y=${touch.clientY}`);
    }
}

function handleTouchMove(event) {
    event.preventDefault();
    
    // Определение жестов
    const touch = event.touches[0];
    if (touch) {
        // Обработка движения
    }
}

// Жест масштабирования
function calculateDistance(touch1, touch2) {
    return Math.sqrt(
        Math.pow(touch2.clientX - touch1.clientX, 2) +
        Math.pow(touch2.clientY - touch1.clientY, 2)
    );
}
```

## Современные паттерны обработки событий

### Делегирование событий с современными методами

```javascript
class EventDelegator {
    constructor(container) {
        this.container = container;
        this.handlers = new Map();
        this.init();
    }
    
    init() {
        this.container.addEventListener('click', this.handleClick.bind(this));
        this.container.addEventListener('submit', this.handleSubmit.bind(this));
    }
    
    onClick(selector, handler) {
        this.addHandler('click', selector, handler);
    }
    
    onSubmit(selector, handler) {
        this.addHandler('submit', selector, handler);
    }
    
    addHandler(eventType, selector, handler) {
        if (!this.handlers.has(eventType)) {
            this.handlers.set(eventType, new Map());
        }
        
        const eventMap = this.handlers.get(eventType);
        eventMap.set(selector, handler);
    }
    
    handleClick(event) {
        this.handleEvent(event, 'click');
    }
    
    handleSubmit(event) {
        this.handleEvent(event, 'submit');
    }
    
    handleEvent(event, eventType) {
        const eventMap = this.handlers.get(eventType);
        if (!eventMap) return;
        
        for (let [selector, handler] of eventMap) {
            if (event.target.matches(selector)) {
                handler.call(this, event);
                break; // Обработка только первого совпадения
            }
        }
    }
}

// Использование
const delegator = new EventDelegator(document.body);
delegator.onClick('.button', function(event) {
    console.log('Кнопка нажата');
});
delegator.onSubmit('form[data-ajax]', function(event) {
    event.preventDefault();
    submitFormAjax(event.target);
});
```

### Custom Events

Создание и использование пользовательских событий:

```javascript
// Создание пользовательского события
function createCustomEvent(type, detail) {
    return new CustomEvent(type, {
        detail: detail,
        bubbles: true,    // Событие всплывает
        cancelable: true  // Можно отменить
    });
}

// Пример использования
class UserActionTracker {
    constructor() {
        this.init();
    }
    
    init() {
        document.addEventListener('click', this.trackClick.bind(this));
        document.addEventListener('user-action', this.handleUserAction.bind(this));
    }
    
    trackClick(event) {
        const eventData = {
            element: event.target.tagName,
            timestamp: Date.now(),
            coordinates: { x: event.clientX, y: event.clientY }
        };
        
        const customEvent = createCustomEvent('user-action', {
            type: 'click',
            data: eventData
        });
        
        document.dispatchEvent(customEvent);
    }
    
    handleUserAction(event) {
        console.log('Пользовательское действие:', event.detail);
        // Отправка данных аналитики
        this.sendAnalytics(event.detail);
    }
    
    sendAnalytics(data) {
        // Отправка данных в аналитическую систему
        console.log('Отправка аналитики:', data);
    }
}

// Глобальные пользовательские события
class GlobalEventManager {
    constructor() {
        this.eventTarget = new EventTarget();
    }
    
    emit(eventType, detail) {
        const event = new CustomEvent(eventType, { detail });
        this.eventTarget.dispatchEvent(event);
    }
    
    on(eventType, callback) {
        this.eventTarget.addEventListener(eventType, callback);
    }
    
    off(eventType, callback) {
        this.eventTarget.removeEventListener(eventType, callback);
    }
}

// Использование
const eventManager = new GlobalEventManager();

eventManager.on('user-login', (event) => {
    console.log('Пользователь вошел:', event.detail);
});

eventManager.emit('user-login', { userId: 123, timestamp: Date.now() });
```

## Современные методы передачи данных через события

### Data Transfer API

```javascript
// Для drag and drop событий
const draggableElement = document.querySelector('.draggable');
const dropZone = document.querySelector('.drop-zone');

draggableElement.addEventListener('dragstart', function(event) {
    // Установка данных для перетаскивания
    event.dataTransfer.setData('text/plain', this.textContent);
    event.dataTransfer.setData('application/json', JSON.stringify({
        id: this.dataset.id,
        type: this.dataset.type
    }));
    
    // Установка изображения для перетаскивания
    event.dataTransfer.setDragImage(this, 0, 0);
});

dropZone.addEventListener('drop', function(event) {
    event.preventDefault();
    
    // Получение текстовых данных
    const textData = event.dataTransfer.getData('text/plain');
    
    // Получение JSON данных
    const jsonData = event.dataTransfer.getData('application/json');
    const parsedData = JSON.parse(jsonData);
    
    // Обработка данных
    handleDrop(textData, parsedData);
});

dropZone.addEventListener('dragover', function(event) {
    event.preventDefault(); // Разрешить drop
    this.classList.add('drag-over');
});
```

### События с комплексными данными

```javascript
// Событие изменения состояния приложения
class StateChangeEvent extends CustomEvent {
    constructor(newState, oldState, changes) {
        super('state-change', {
            detail: {
                newState,
                oldState,
                changes,
                timestamp: Date.now()
            },
            bubbles: true,
            cancelable: true
        });
    }
}

// Использование
const stateManager = {
    state: { user: null, theme: 'light' },
    
    updateState(newState) {
        const oldState = { ...this.state };
        const changes = this.calculateChanges(this.state, newState);
        
        // Создание события с данными
        const event = new StateChangeEvent(newState, oldState, changes);
        
        // Если событие отменено, не обновляем состояние
        if (!document.dispatchEvent(event)) {
            console.log('Обновление состояния отменено');
            return false;
        }
        
        this.state = { ...newState };
        return true;
    },
    
    calculateChanges(oldState, newState) {
        const changes = {};
        for (const key in newState) {
            if (oldState[key] !== newState[key]) {
                changes[key] = {
                    from: oldState[key],
                    to: newState[key]
                };
            }
        }
        return changes;
    }
};

// Слушатель событий
document.addEventListener('state-change', function(event) {
    const { newState, oldState, changes } = event.detail;
    console.log('Изменения состояния:', changes);
    
    // Можно отменить событие
    // event.preventDefault();
});
```

## Современные паттерны управления событиями

### Event Bus (Шина событий)

```javascript
class EventBus {
    constructor() {
        this.eventTarget = new EventTarget();
        this.listeners = new Map();
    }
    
    // Подписка на событие
    on(eventType, callback, options = {}) {
        const handler = typeof options === 'object' && options.once ? 
            (...args) => {
                callback(...args);
                this.off(eventType, handler);
            } : 
            callback;
            
        this.eventTarget.addEventListener(eventType, handler, options);
        
        // Сохранение для отписки
        if (!this.listeners.has(eventType)) {
            this.listeners.set(eventType, []);
        }
        this.listeners.get(eventType).push({ callback, handler });
    }
    
    // Отписка от события
    off(eventType, callback) {
        const listeners = this.listeners.get(eventType);
        if (!listeners) return;
        
        const listener = listeners.find(l => l.callback === callback);
        if (listener) {
            this.eventTarget.removeEventListener(eventType, listener.handler);
            listeners.splice(listeners.indexOf(listener), 1);
        }
    }
    
    // Вызов события
    emit(eventType, detail) {
        const event = new CustomEvent(eventType, { detail });
        this.eventTarget.dispatchEvent(event);
    }
    
    // Очистка всех подписчиков
    clear() {
        for (const [eventType, listeners] of this.listeners) {
            listeners.forEach(({ handler }) => {
                this.eventTarget.removeEventListener(eventType, handler);
            });
        }
        this.listeners.clear();
    }
}

// Использование
const eventBus = new EventBus();

eventBus.on('user-login', (event) => {
    console.log('Пользователь вошел:', event.detail);
});

eventBus.emit('user-login', { userId: 123, username: 'john' });
```

### События с промисами

```javascript
// Ожидание события с таймаутом
function waitForEvent(element, eventType, timeout = 5000) {
    return new Promise((resolve, reject) => {
        const timeoutId = setTimeout(() => {
            element.removeEventListener(eventType, handler);
            reject(new Error(`Таймаут ожидания события ${eventType}`));
        }, timeout);

        const handler = (event) => {
            clearTimeout(timeoutId);
            element.removeEventListener(eventType, handler);
            resolve(event);
        };

        element.addEventListener(eventType, handler);
    });
}

// Использование
async function handleAsyncEvent() {
    try {
        const event = await waitForEvent(button, 'click', 3000);
        console.log('Кнопка нажата:', event);
    } catch (error) {
        console.log('Событие не произошло вовремя:', error.message);
    }
}

// Событие с асинхронной обработкой
class AsyncEventHandler {
    constructor() {
        this.handlers = new Map();
    }
    
    async emit(eventType, detail) {
        const handlers = this.handlers.get(eventType) || [];
        
        // Выполнение всех обработчиков параллельно
        const results = await Promise.allSettled(
            handlers.map(handler => handler(detail))
        );
        
        return results;
    }
    
    on(eventType, handler) {
        if (!this.handlers.has(eventType)) {
            this.handlers.set(eventType, []);
        }
        this.handlers.get(eventType).push(handler);
    }
}
```

## Лучшие практики

### 1. Правильная очистка обработчиков

```javascript
class ComponentWithEvents {
    constructor(element) {
        this.element = element;
        this.boundHandlers = new Map();
        this.abortController = new AbortController();
        this.init();
    }
    
    init() {
        // Привязка обработчиков к экземпляру
        this.boundHandlers.set('click', this.handleClick.bind(this));
        this.boundHandlers.set('focus', this.handleFocus.bind(this));
        
        // Добавление обработчиков с сигналом отмены
        this.element.addEventListener('click', this.boundHandlers.get('click'), {
            signal: this.abortController.signal
        });
        
        this.element.addEventListener('focus', this.boundHandlers.get('focus'), {
            signal: this.abortController.signal
        });
    }
    
    destroy() {
        // Отмена всех обработчиков
        this.abortController.abort();
        
        // Очистка привязанных обработчиков
        this.boundHandlers.clear();
        this.element = null;
    }
    
    handleClick(event) {
        // Обработка клика
    }
    
    handleFocus(event) {
        // Обработка фокуса
    }
}
```

### 2. Оптимизация производительности

```javascript
// Дебаунс для событий ввода
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

const debouncedInputHandler = debounce(function(event) {
    console.log('Поиск:', event.target.value);
}, 300);

inputElement.addEventListener('input', debouncedInputHandler);

// Троттлинг для событий прокрутки
function throttle(func, limit) {
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

const throttledScrollHandler = throttle(function() {
    console.log('Прокрутка');
}, 100);

window.addEventListener('scroll', throttledScrollHandler);
```

## Примеры из промышленной разработки

### Система уведомлений на событиях

```javascript
class NotificationSystem {
    constructor() {
        this.eventBus = new EventBus();
        this.notificationQueue = [];
        this.init();
    }
    
    init() {
        this.eventBus.on('notification:request', this.handleNotificationRequest.bind(this));
        this.eventBus.on('notification:dismiss', this.handleDismiss.bind(this));
    }
    
    showNotification(options) {
        const notification = {
            id: Date.now(),
            ...options,
            timestamp: Date.now()
        };
        
        this.notificationQueue.push(notification);
        
        // Вызов пользовательского события
        this.eventBus.emit('notification:created', notification);
        
        // Автоматическое скрытие
        if (options.duration) {
            setTimeout(() => {
                this.dismissNotification(notification.id);
            }, options.duration);
        }
        
        return notification.id;
    }
    
    handleNotificationRequest(event) {
        const { type, message, duration = 3000 } = event.detail;
        this.showNotification({ type, message, duration });
    }
    
    dismissNotification(id) {
        this.notificationQueue = this.notificationQueue.filter(n => n.id !== id);
        this.eventBus.emit('notification:dismissed', { id });
    }
    
    handleDismiss(event) {
        this.dismissNotification(event.detail.id);
    }
}

// Использование
const notifications = new NotificationSystem();

// Отправка уведомления
document.dispatchEvent(new CustomEvent('notification:request', {
    detail: { type: 'success', message: 'Операция выполнена успешно!', duration: 5000 }
}));
```

### Менеджер модальных окон

```javascript
class ModalManager {
    constructor() {
        this.currentModal = null;
        this.stack = [];
        this.init();
    }
    
    init() {
        // Обработка клавиатурных событий
        document.addEventListener('keydown', this.handleKeydown.bind(this));
        
        // Обработка кликов вне модального окна
        document.addEventListener('click', this.handleClickOutside.bind(this));
    }
    
    open(modalElement) {
        // Закрытие текущего модального окна при открытии нового
        if (this.currentModal) {
            this.close(this.currentModal);
        }
        
        this.currentModal = modalElement;
        this.stack.push(modalElement);
        
        modalElement.style.display = 'block';
        modalElement.setAttribute('aria-hidden', 'false');
        
        // Фокус на модальное окно
        const focusElement = modalElement.querySelector('[autofocus]') || modalElement;
        focusElement.focus();
        
        // Вызов события
        modalElement.dispatchEvent(new CustomEvent('modal:open', {
            detail: { modal: modalElement }
        }));
    }
    
    close(modalElement = this.currentModal) {
        if (!modalElement) return;
        
        modalElement.style.display = 'none';
        modalElement.setAttribute('aria-hidden', 'true');
        
        this.stack = this.stack.filter(m => m !== modalElement);
        this.currentModal = this.stack[this.stack.length - 1] || null;
        
        // Вызов события
        modalElement.dispatchEvent(new CustomEvent('modal:close', {
            detail: { modal: modalElement }
        }));
    }
    
    handleKeydown(event) {
        if (event.key === 'Escape' && this.currentModal) {
            this.close();
        }
    }
    
    handleClickOutside(event) {
        if (this.currentModal && !this.currentModal.contains(event.target)) {
            this.close();
        }
    }
}
```

## Ссылки на связанные темы
- [[js/dom/manipulation]] - Манипуляции DOM
- [[js/testing/events]] - Тестирование событий
- [[html/accessibility]] - Доступность
- [[css/animations]] - Анимации

#programming #javascript #events #best-practices #web-components
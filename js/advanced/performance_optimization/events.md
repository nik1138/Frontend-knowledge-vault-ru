# Оптимизация обработки событий в JavaScript

## Введение

Обработка событий - фундаментальная часть интерактивных веб-приложений. Однако неправильная реализация обработчиков событий может привести к проблемам с производительностью, утечкам памяти и плохому пользовательскому опыту. В этом разделе мы рассмотрим различные техники оптимизации обработки событий в JavaScript.

## Делегирование событий

Эффективное использование делегирования событий:

```javascript
// Делегирование событий
class EventDelegation {
  // Базовая реализация делегирования
  static setupEventDelegation(container, eventType, selector, handler) {
    container.addEventListener(eventType, (event) => {
      const target = event.target;
      const matchingElement = target.closest(selector);
      
      if (matchingElement && container.contains(matchingElement)) {
        handler.call(matchingElement, event);
      }
    });
  }
  
  // Расширенное делегирование событий
  static createEventDelegator(container) {
    const handlers = new Map();
    
    container.addEventListener('click', (event) => {
      const target = event.target;
      
      // Проверяем все зарегистрированные обработчики
      for (const [selector, handler] of handlers) {
        const matchingElement = target.closest(selector);
        if (matchingElement && container.contains(matchingElement)) {
          handler.call(matchingElement, event);
        }
      }
    });
    
    return {
      on(selector, handler) {
        handlers.set(selector, handler);
      },
      
      off(selector) {
        handlers.delete(selector);
      }
    };
  }
  
  // Делегирование с поддержкой нескольких типов событий
  static setupMultiEventDelegation(container, selectorsAndHandlers) {
    Object.entries(selectorsAndHandlers).forEach(([selector, handlers]) => {
      Object.entries(handlers).forEach(([eventType, handler]) => {
        container.addEventListener(eventType, (event) => {
          const target = event.target;
          const matchingElement = target.closest(selector);
          
          if (matchingElement && container.contains(matchingElement)) {
            handler.call(matchingElement, event);
          }
        });
      });
    });
  }
  
  // Пример использования делегирования
  static setupListDelegation(listContainer) {
    // Один обработчик для всех кнопок удаления
    this.setupEventDelegation(
      listContainer,
      'click',
      '.delete-btn',
      function(event) {
        const listItem = this.closest('.list-item');
        if (listItem) {
          listItem.remove();
        }
      }
    );
    
    // Один обработчик для всех чекбоксов
    this.setupEventDelegation(
      listContainer,
      'change',
      '.item-checkbox',
      function(event) {
        const listItem = this.closest('.list-item');
        if (listItem) {
          listItem.classList.toggle('completed', this.checked);
        }
      }
    );
  }
}

// Пример использования делегирования событий
class TodoList {
  constructor(container) {
    this.container = container;
    this.delegator = EventDelegation.createEventDelegator(container);
    this.init();
  }
  
  init() {
    // Регистрируем обработчики
    this.delegator.on('.add-btn', this.handleAddItem.bind(this));
    this.delegator.on('.delete-btn', this.handleDeleteItem.bind(this));
    this.delegator.on('.edit-btn', this.handleEditItem.bind(this));
    this.delegator.on('.item-checkbox', this.handleItemToggle.bind(this));
    
    // Обработчик для формы
    this.delegator.on('.todo-form', (event) => {
      event.preventDefault();
      this.addItem(this.container.querySelector('.todo-input').value);
    });
  }
  
  handleAddItem(event) {
    const input = this.container.querySelector('.todo-input');
    this.addItem(input.value);
    input.value = '';
  }
  
  handleDeleteItem(event) {
    const item = event.closest('.todo-item');
    if (item) {
      item.remove();
    }
  }
  
  handleEditItem(event) {
    const item = event.closest('.todo-item');
    const textElement = item.querySelector('.item-text');
    const newText = prompt('Редактировать задачу:', textElement.textContent);
    
    if (newText !== null) {
      textElement.textContent = newText;
    }
  }
  
  handleItemToggle(event) {
    const item = event.closest('.todo-item');
    item.classList.toggle('completed', event.target.checked);
  }
  
  addItem(text) {
    if (!text.trim()) return;
    
    const item = document.createElement('div');
    item.className = 'todo-item';
    item.innerHTML = `
      <input type="checkbox" class="item-checkbox">
      <span class="item-text">${text}</span>
      <button class="edit-btn">Редактировать</button>
      <button class="delete-btn">Удалить</button>
    `;
    
    this.container.querySelector('.todo-list').appendChild(item);
  }
}
```

## Throttling и Debouncing

Оптимизация частых событий:

```javascript
// Throttling и Debouncing
class EventRateLimiting {
  // Throttling - ограничение частоты выполнения
  static throttle(func, limit) {
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
  
  // Debouncing - задержка выполнения до окончания событий
  static debounce(func, delay) {
    let timeoutId;
    return function() {
      const context = this;
      const args = arguments;
      
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => func.apply(context, args), delay);
    }
  }
  
  // Продвинутый throttling с trailing вызовом
  static throttleWithTrailing(func, limit) {
    let timeoutId;
    let lastRun = 0;
    
    return function() {
      const context = this;
      const args = arguments;
      const now = Date.now();
      
      if (now - lastRun >= limit) {
        func.apply(context, args);
        lastRun = now;
      } else {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
          func.apply(context, args);
          lastRun = Date.now();
        }, limit - (now - lastRun));
      }
    }
  }
  
  // Adaptive throttling
  static adaptiveThrottle(func, baseDelay, maxDelay) {
    let delay = baseDelay;
    let lastRun = 0;
    let timeoutId;
    
    return function() {
      const context = this;
      const args = arguments;
      const now = Date.now();
      
      const timeSinceLastRun = now - lastRun;
      
      // Адаптируем задержку в зависимости от частоты вызовов
      if (timeSinceLastRun < delay * 2) {
        delay = Math.min(delay * 1.5, maxDelay);
      } else {
        delay = Math.max(delay * 0.8, baseDelay);
      }
      
      clearTimeout(timeoutId);
      
      if (now - lastRun >= delay) {
        func.apply(context, args);
        lastRun = now;
      } else {
        timeoutId = setTimeout(() => {
          func.apply(context, args);
          lastRun = Date.now();
        }, delay - (now - lastRun));
      }
    }
  }
  
  // Примеры использования
  static setupOptimizedEvents() {
    // Throttling для scroll событий
    window.addEventListener('scroll', this.throttle(() => {
      console.log('Scroll event handled');
      // Обновление позиции элементов, индикаторов прокрутки и т.д.
    }, 100));
    
    // Debouncing для input событий
    const searchInput = document.getElementById('search');
    searchInput.addEventListener('input', this.debounce((event) => {
      console.log('Search query:', event.target.value);
      // Выполнение поиска
      this.performSearch(event.target.value);
    }, 300));
    
    // Throttling для mousemove событий
    document.addEventListener('mousemove', this.throttle((event) => {
      // Обновление позиции курсора или анимаций
      this.updateCursorPosition(event.clientX, event.clientY);
    }, 16)); // Примерно 60 FPS
    
    // Adaptive throttling для resize событий
    window.addEventListener('resize', this.adaptiveThrottle(() => {
      console.log('Window resized');
      // Пересчет размеров элементов
      this.recalculateLayout();
    }, 100, 1000));
  }
  
  static performSearch(query) {
    // Имитация поиска
    console.log(`Searching for: ${query}`);
  }
  
  static updateCursorPosition(x, y) {
    // Имитация обновления позиции
    console.log(`Cursor at: ${x}, ${y}`);
  }
  
  static recalculateLayout() {
    // Имитация пересчета layout
    console.log('Layout recalculated');
  }
}

// Класс для управления оптимизированными событиями
class OptimizedEventManager {
  constructor() {
    this.throttledHandlers = new Map();
    this.debouncedHandlers = new Map();
  }
  
  // Регистрация throttled обработчика
  addThrottledListener(element, eventType, handler, delay) {
    const key = `${element}-${eventType}`;
    const throttledHandler = EventRateLimiting.throttle(handler, delay);
    
    this.throttledHandlers.set(key, throttledHandler);
    element.addEventListener(eventType, throttledHandler);
  }
  
  // Регистрация debounced обработчика
  addDebouncedListener(element, eventType, handler, delay) {
    const key = `${element}-${eventType}`;
    const debouncedHandler = EventRateLimiting.debounce(handler, delay);
    
    this.debouncedHandlers.set(key, debouncedHandler);
    element.addEventListener(eventType, debouncedHandler);
  }
  
  // Удаление обработчика
  removeListener(element, eventType, handler) {
    element.removeEventListener(eventType, handler);
  }
  
  // Очистка всех обработчиков
  clearAll() {
    this.throttledHandlers.clear();
    this.debouncedHandlers.clear();
  }
}
```

## Пассивные обработчики событий

Оптимизация прокрутки и других событий:

```javascript
// Пассивные обработчики событий
class PassiveEventHandlers {
  // Проверка поддержки passive events
  static supportsPassiveEvents() {
    let supportsPassive = false;
    try {
      const opts = Object.defineProperty({}, 'passive', {
        get: function() {
          supportsPassive = true;
          return true;
        }
      });
      window.addEventListener('testPassive', null, opts);
      window.removeEventListener('testPassive', null, opts);
    } catch (e) {}
    return supportsPassive;
  }
  
  // Добавление пассивного обработчика
  static addPassiveListener(element, eventType, handler) {
    const options = this.supportsPassiveEvents() ? { passive: true } : false;
    element.addEventListener(eventType, handler, options);
  }
  
  // Оптимизация touch событий
  static optimizeTouchEvents() {
    const touchElements = document.querySelectorAll('.touch-area');
    
    touchElements.forEach(element => {
      // Пассивные touch события для лучшей производительности прокрутки
      this.addPassiveListener(element, 'touchstart', this.handleTouchStart);
      this.addPassiveListener(element, 'touchmove', this.handleTouchMove);
      this.addPassiveListener(element, 'touchend', this.handleTouchEnd);
    });
  }
  
  // Обработчики touch событий
  static handleTouchStart(event) {
    console.log('Touch started');
    // Не вызываем preventDefault для пассивных обработчиков
  }
  
  static handleTouchMove(event) {
    console.log('Touch moved');
    // Не вызываем preventDefault для пассивных обработчиков
  }
  
  static handleTouchEnd(event) {
    console.log('Touch ended');
  }
  
  // Оптимизация scroll событий
  static optimizeScrollEvents() {
    // Пассивный scroll listener для лучшей производительности
    this.addPassiveListener(window, 'scroll', EventRateLimiting.throttle((event) => {
      // Обновление позиции элементов
      this.updateScrollPosition(window.scrollY);
    }, 16));
    
    // Пассивный wheel listener
    this.addPassiveListener(window, 'wheel', EventRateLimiting.throttle((event) => {
      // Обработка прокрутки колесом мыши
      this.handleWheelScroll(event);
    }, 16));
  }
  
  static updateScrollPosition(scrollY) {
    // Имитация обновления позиции
    console.log(`Scroll position: ${scrollY}`);
  }
  
  static handleWheelScroll(event) {
    // Имитация обработки прокрутки
    console.log(`Wheel delta: ${event.deltaY}`);
  }
  
  // Создание оптимизированного gesture recognizer
  static createGestureRecognizer(element) {
    let startX = 0;
    let startY = 0;
    let startTime = 0;
    
    const passiveOptions = this.supportsPassiveEvents() ? { passive: true } : false;
    
    element.addEventListener('touchstart', (event) => {
      const touch = event.touches[0];
      startX = touch.clientX;
      startY = touch.clientY;
      startTime = Date.now();
    }, passiveOptions);
    
    element.addEventListener('touchmove', (event) => {
      // Обработка swipe gestures
      const touch = event.touches[0];
      const deltaX = touch.clientX - startX;
      const deltaY = touch.clientY - startY;
      
      // Определяем направление свайпа
      if (Math.abs(deltaX) > Math.abs(deltaY)) {
        if (deltaX > 50) {
          element.dispatchEvent(new CustomEvent('swipeRight'));
        } else if (deltaX < -50) {
          element.dispatchEvent(new CustomEvent('swipeLeft'));
        }
      } else {
        if (deltaY > 50) {
          element.dispatchEvent(new CustomEvent('swipeDown'));
        } else if (deltaY < -50) {
          element.dispatchEvent(new CustomEvent('swipeUp'));
        }
      }
    }, passiveOptions);
    
    element.addEventListener('touchend', (event) => {
      const endTime = Date.now();
      const duration = endTime - startTime;
      
      // Определяем tap или long press
      if (duration < 200) {
        element.dispatchEvent(new CustomEvent('tap'));
      } else if (duration > 500) {
        element.dispatchEvent(new CustomEvent('longPress'));
      }
    }, passiveOptions);
  }
}

// Пример использования пассивных обработчиков
class TouchGallery {
  constructor(container) {
    this.container = container;
    this.currentIndex = 0;
    this.init();
  }
  
  init() {
    // Настройка пассивных обработчиков
    PassiveEventHandlers.createGestureRecognizer(this.container);
    
    // Обработка жестов
    this.container.addEventListener('swipeLeft', () => this.nextSlide());
    this.container.addEventListener('swipeRight', () => this.prevSlide());
    this.container.addEventListener('tap', () => this.toggleFullscreen());
    this.container.addEventListener('longPress', () => this.showInfo());
  }
  
  nextSlide() {
    this.currentIndex++;
    this.updateSlide();
  }
  
  prevSlide() {
    this.currentIndex = Math.max(0, this.currentIndex - 1);
    this.updateSlide();
  }
  
  updateSlide() {
    const slides = this.container.querySelectorAll('.slide');
    slides.forEach((slide, index) => {
      slide.style.display = index === this.currentIndex ? 'block' : 'none';
    });
  }
  
  toggleFullscreen() {
    if (!document.fullscreenElement) {
      this.container.requestFullscreen().catch(err => {
        console.error('Ошибка перехода в полноэкранный режим:', err);
      });
    } else {
      document.exitFullscreen();
    }
  }
  
  showInfo() {
    alert('Информация о галерее');
  }
}
```

## Управление памятью и утечки

Предотвращение утечек памяти в обработчиках событий:

```javascript
// Управление памятью и предотвращение утечек
class MemoryManagement {
  constructor() {
    this.eventListeners = new Map();
    this.weakRefs = new WeakMap();
  }
  
  // Добавление обработчика с отслеживанием
  addTrackedListener(element, eventType, handler, options = {}) {
    const key = `${element}-${eventType}-${handler.name || 'anonymous'}`;
    
    // Сохраняем информацию об обработчике
    this.eventListeners.set(key, {
      element,
      eventType,
      handler,
      options
    });
    
    element.addEventListener(eventType, handler, options);
    
    // Используем WeakMap для отслеживания элементов
    if (!this.weakRefs.has(element)) {
      this.weakRefs.set(element, new Set());
    }
    this.weakRefs.get(element).add(key);
  }
  
  // Удаление обработчика
  removeTrackedListener(element, eventType, handler) {
    const key = `${element}-${eventType}-${handler.name || 'anonymous'}`;
    
    if (this.eventListeners.has(key)) {
      element.removeEventListener(eventType, handler);
      this.eventListeners.delete(key);
      
      // Удаляем из WeakMap
      if (this.weakRefs.has(element)) {
        this.weakRefs.get(element).delete(key);
      }
    }
  }
  
  // Очистка всех обработчиков для элемента
  removeAllListenersForElement(element) {
    if (this.weakRefs.has(element)) {
      const keys = this.weakRefs.get(element);
      keys.forEach(key => {
        if (this.eventListeners.has(key)) {
          const { element, eventType, handler } = this.eventListeners.get(key);
          element.removeEventListener(eventType, handler);
          this.eventListeners.delete(key);
        }
      });
      this.weakRefs.delete(element);
    }
  }
  
  // Очистка всех обработчиков
  clearAllListeners() {
    for (const [key, listener] of this.eventListeners) {
      const { element, eventType, handler } = listener;
      element.removeEventListener(eventType, handler);
    }
    this.eventListeners.clear();
    this.weakRefs = new WeakMap();
  }
  
  // Получение статистики
  getStats() {
    return {
      totalListeners: this.eventListeners.size,
      trackedElements: this.weakRefs.size
    };
  }
  
  // Автоматическая очистка для удаленных элементов
  cleanupOrphanedListeners() {
    const orphanedKeys = [];
    
    for (const [key, listener] of this.eventListeners) {
      // Проверяем, существует ли элемент в DOM
      if (!document.contains(listener.element)) {
        orphanedKeys.push(key);
      }
    }
    
    orphanedKeys.forEach(key => {
      const listener = this.eventListeners.get(key);
      this.eventListeners.delete(key);
      console.warn(`Удален orphaned обработчик: ${key}`);
    });
    
    return orphanedKeys.length;
  }
}

// Контроллер событий с автоматическим управлением памятью
class EventController {
  constructor() {
    this.memoryManager = new MemoryManagement();
    this.activeControllers = new Set();
  }
  
  // Создание контроллера для компонента
  createController(component) {
    const controller = new ComponentEventController(component, this.memoryManager);
    this.activeControllers.add(controller);
    return controller;
  }
  
  // Удаление контроллера
  destroyController(controller) {
    controller.destroy();
    this.activeControllers.delete(controller);
  }
  
  // Очистка всех контроллеров
  destroyAll() {
    this.activeControllers.forEach(controller => controller.destroy());
    this.activeControllers.clear();
    this.memoryManager.clearAllListeners();
  }
  
  // Получение статистики
  getStats() {
    return {
      activeControllers: this.activeControllers.size,
      ...this.memoryManager.getStats()
    };
  }
}

// Контроллер событий для компонента
class ComponentEventController {
  constructor(component, memoryManager) {
    this.component = component;
    this.memoryManager = memoryManager;
    this.isDestroyed = false;
  }
  
  // Добавление обработчика
  on(element, eventType, handler, options) {
    if (this.isDestroyed) {
      console.warn('Невозможно добавить обработчик к уничтоженному контроллеру');
      return;
    }
    
    this.memoryManager.addTrackedListener(element, eventType, handler, options);
  }
  
  // Удаление обработчика
  off(element, eventType, handler) {
    if (this.isDestroyed) return;
    this.memoryManager.removeTrackedListener(element, eventType, handler);
  }
  
  // Уничтожение контроллера
  destroy() {
    if (this.isDestroyed) return;
    
    this.memoryManager.removeAllListenersForElement(this.component);
    this.isDestroyed = true;
  }
}

// Пример использования контроллера событий
class ModalDialog {
  constructor(element) {
    this.element = element;
    this.eventController = new EventController().createController(element);
    this.init();
  }
  
  init() {
    // Добавляем обработчики через контроллер
    this.eventController.on(
      this.element.querySelector('.close-btn'),
      'click',
      this.close.bind(this)
    );
    
    this.eventController.on(
      this.element,
      'keydown',
      this.handleKeydown.bind(this)
    );
    
    this.eventController.on(
      document,
      'click',
      this.handleOutsideClick.bind(this)
    );
  }
  
  handleKeydown(event) {
    if (event.key === 'Escape') {
      this.close();
    }
  }
  
  handleOutsideClick(event) {
    if (!this.element.contains(event.target)) {
      this.close();
    }
  }
  
  close() {
    this.element.style.display = 'none';
    // Автоматически очищаем обработчики
    this.eventController.destroy();
  }
}
```

## Практические рекомендации

1. **Используйте делегирование событий** - для списков и динамических элементов
2. **Применяйте throttling и debouncing** - для частых событий (scroll, resize, input)
3. **Используйте пассивные обработчики** - для touch и scroll событий
4. **Управляйте памятью** - удаляйте обработчики при удалении элементов
5. **Избегайте анонимных функций** - как обработчиков для возможности удаления
6. **Ограничивайте количество обработчиков** - используйте делегирование вместо множества отдельных обработчиков
7. **Мониторьте утечки памяти** - с помощью инструментов разработчика
8. **Тестируйте производительность** - на реальных устройствах и с реальными данными

Оптимизация обработки событий - важный аспект создания производительных и отзывчивых веб-приложений. Следование этим принципам поможет избежать проблем с производительностью и утечками памяти.

#javascript #events #optimization #performance #event-delegation #throttling #debouncing #memory-management
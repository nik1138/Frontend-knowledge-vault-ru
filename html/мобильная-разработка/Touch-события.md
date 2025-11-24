---
aliases: ["Сенсорные события", "Touch Events", "Мобильные жесты"]
tags: [javascript, mobile-development, touch-events]
---

# Touch-события в мобильной разработке

## Введение в Touch-события

Touch-события позволяют обрабатывать взаимодействие пользователя с сенсорными экранами мобильных устройств. В отличие от мышиных событий, touch-события учитывают особенности сенсорного взаимодействия, такие как касания, свайпы и жесты.

## Основные Touch-события

### touchstart
Срабатывает при первом касании экрана.

### touchmove
Срабатывает при движении пальца по экрану.

### touchend
Срабатывает при отрыве пальца от экрана.

### touchcancel
Срабатывает при прерывании touch-события (например, входящий звонок).

## Примеры использования

### Базовая обработка touch-событий

```javascript
const element = document.getElementById('touch-area');

element.addEventListener('touchstart', handleTouchStart, false);
element.addEventListener('touchmove', handleTouchMove, false);
element.addEventListener('touchend', handleTouchEnd, false);

let touchStartX = 0;
let touchStartY = 0;

function handleTouchStart(event) {
    event.preventDefault();
    const touch = event.touches[0];
    touchStartX = touch.clientX;
    touchStartY = touch.clientY;
}

function handleTouchMove(event) {
    event.preventDefault();
    const touch = event.touches[0];
    
    // Вычисление смещения
    const diffX = touch.clientX - touchStartX;
    const diffY = touch.clientY - touchStartY;
    
    // Обработка свайпа
    if (Math.abs(diffX) > Math.abs(diffY)) {
        // Горизонтальный свайп
        if (diffX > 0) {
            console.log('Свайп вправо');
        } else {
            console.log('Свайп влево');
        }
    } else {
        // Вертикальный свайп
        if (diffY > 0) {
            console.log('Свайп вниз');
        } else {
            console.log('Свайп вверх');
        }
    }
}

function handleTouchEnd(event) {
    console.log('Касание завершено');
}
```

## Объект TouchEvent

Каждое touch-событие содержит объект TouchEvent с важными свойствами:

- `touches` - список всех активных касаний на экране
- `targetTouches` - список касаний на текущем элементе
- `changedTouches` - список касаний, которые изменились в результате события

```javascript
function handleTouchStart(event) {
    console.log('Количество активных касаний:', event.touches.length);
    console.log('Касания на элементе:', event.targetTouches.length);
    console.log('Изменившиеся касания:', event.changedTouches.length);
    
    // Получение координат первого касания
    const firstTouch = event.touches[0];
    console.log('X:', firstTouch.clientX, 'Y:', firstTouch.clientY);
}
```

## Распознавание жестов

### Свайп влево/вправо

```javascript
class SwipeDetector {
    constructor(element) {
        this.element = element;
        this.startX = 0;
        this.startY = 0;
        this.minSwipeDistance = 30;
        
        this.init();
    }
    
    init() {
        this.element.addEventListener('touchstart', this.touchStart.bind(this), false);
        this.element.addEventListener('touchmove', this.touchMove.bind(this), false);
        this.element.addEventListener('touchend', this.touchEnd.bind(this), false);
    }
    
    touchStart(event) {
        const touch = event.touches[0];
        this.startX = touch.clientX;
        this.startY = touch.clientY;
    }
    
    touchMove(event) {
        if (!this.startX || !this.startY) return;
        
        const touch = event.touches[0];
        const diffX = this.startX - touch.clientX;
        const diffY = this.startY - touch.clientY;
        
        // Проверка, что это predominantly горизонтальный свайп
        if (Math.abs(diffX) > Math.abs(diffY)) {
            event.preventDefault();
        }
    }
    
    touchEnd(event) {
        if (!this.startX || !this.startY) return;
        
        const touch = event.changedTouches[0];
        const diffX = this.startX - touch.clientX;
        const diffY = this.startY - touch.clientY;
        
        // Проверка минимального расстояния свайпа
        if (Math.abs(diffX) < this.minSwipeDistance && Math.abs(diffY) < this.minSwipeDistance) {
            return;
        }
        
        if (Math.abs(diffX) > Math.abs(diffY)) {
            // Горизонтальный свайп
            if (diffX > 0) {
                this.onSwipeLeft();
            } else {
                this.onSwipeRight();
            }
        } else {
            // Вертикальный свайп
            if (diffY > 0) {
                this.onSwipeUp();
            } else {
                this.onSwipeDown();
            }
        }
        
        // Сброс значений
        this.startX = 0;
        this.startY = 0;
    }
    
    onSwipeLeft() {
        console.log('Свайп влево');
        this.element.dispatchEvent(new CustomEvent('swipeleft'));
    }
    
    onSwipeRight() {
        console.log('Свайп вправо');
        this.element.dispatchEvent(new CustomEvent('swiperight'));
    }
    
    onSwipeUp() {
        console.log('Свайп вверх');
        this.element.dispatchEvent(new CustomEvent('swipeup'));
    }
    
    onSwipeDown() {
        console.log('Свайп вниз');
        this.element.dispatchEvent(new CustomEvent('swipedown'));
    }
}

// Использование
const swipeDetector = new SwipeDetector(document.getElementById('swipe-area'));
swipeDetector.element.addEventListener('swipeleft', () => console.log('Обработка свайпа влево'));
swipeDetector.element.addEventListener('swiperight', () => console.log('Обработка свайпа вправо'));
```

### Зум с двумя пальцами

```javascript
class PinchZoom {
    constructor(element) {
        this.element = element;
        this.init();
    }
    
    init() {
        this.element.addEventListener('touchstart', this.handleTouchStart.bind(this), false);
        this.element.addEventListener('touchmove', this.handleTouchMove.bind(this), false);
        this.element.addEventListener('touchend', this.handleTouchEnd.bind(this), false);
    }
    
    handleTouchStart(event) {
        if (event.touches.length === 2) {
            this.initialDistance = this.getDistance(event.touches[0], event.touches[1]);
            this.currentScale = this.getScale();
        }
    }
    
    handleTouchMove(event) {
        if (event.touches.length === 2) {
            event.preventDefault();
            
            const currentDistance = this.getDistance(event.touches[0], event.touches[1]);
            const scale = currentDistance / this.initialDistance;
            const newScale = this.currentScale * scale;
            
            // Ограничение масштаба
            if (newScale >= 0.5 && newScale <= 3) {
                this.element.style.transform = `scale(${newScale})`;
            }
        }
    }
    
    handleTouchEnd(event) {
        // Сброс после завершения жеста
    }
    
    getDistance(touch1, touch2) {
        return Math.sqrt(
            Math.pow(touch2.clientX - touch1.clientX, 2) +
            Math.pow(touch2.clientY - touch1.clientY, 2)
        );
    }
    
    getScale() {
        const transform = window.getComputedStyle(this.element).transform;
        if (transform === 'none') return 1;
        
        const matrix = new DOMMatrix(transform);
        return matrix.a; // масштаб по оси X
    }
}
```

## Совместимость с мышью

Для обеспечения совместимости с десктопными устройствами рекомендуется также обрабатывать мышиные события:

```javascript
class UniversalTouchHandler {
    constructor(element) {
        this.element = element;
        this.isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;
        this.init();
    }
    
    init() {
        if (this.isTouchDevice) {
            this.element.addEventListener('touchstart', this.handleTouchStart.bind(this));
            this.element.addEventListener('touchmove', this.handleTouchMove.bind(this));
            this.element.addEventListener('touchend', this.handleTouchEnd.bind(this));
        } else {
            this.element.addEventListener('mousedown', this.handleMouseDown.bind(this));
            this.element.addEventListener('mousemove', this.handleMouseMove.bind(this));
            this.element.addEventListener('mouseup', this.handleMouseUp.bind(this));
        }
    }
    
    handleTouchStart(event) {
        // Обработка touch-событий
        this.startX = event.touches[0].clientX;
        this.startY = event.touches[0].clientY;
    }
    
    handleMouseDown(event) {
        // Обработка мышиных событий
        this.startX = event.clientX;
        this.startY = event.clientY;
        document.addEventListener('mousemove', this.handleMouseMove.bind(this));
        document.addEventListener('mouseup', this.handleMouseUp.bind(this));
    }
    
    // Другие методы...
}
```

## Практические рекомендации

### 1. Предотвращение стандартного поведения

Используйте `event.preventDefault()` для предотвращения стандартных действий браузера:

```javascript
element.addEventListener('touchstart', (e) => {
    e.preventDefault(); // Предотвращает прокрутку и зум
    // Обработка события
});
```

### 2. Оптимизация производительности

- Используйте `requestAnimationFrame` для анимаций в touch-событиях
- Ограничивайте частоту вызовов обработчиков
- Избегайте тяжелых вычислений в `touchmove`

```javascript
let ticking = false;

function updatePosition(diffX, diffY) {
    // Анимация или обновление позиции
    element.style.transform = `translateX(${diffX}px) translateY(${diffY}px)`;
}

element.addEventListener('touchmove', (e) => {
    const touch = e.touches[0];
    const diffX = touch.clientX - startX;
    const diffY = touch.clientY - startY;
    
    if (!ticking) {
        requestAnimationFrame(() => {
            updatePosition(diffX, diffY);
            ticking = false;
        });
        ticking = true;
    }
});
```

### 3. Учет особенностей российского рынка

В 2025 году в России важно учитывать:
- Разнообразие моделей мобильных устройств
- Возможные ограничения производительности на бюджетных устройствах
- Разные версии браузеров и их поддержку touch-событий
- Потребность в быстрой и отзывчивой работе интерфейса

## Популярные библиотеки для работы с touch-событиями

- **Hammer.js** - мощная библиотека для распознавания жестов
- **AlloyFinger** - легковесная альтернатива для touch-событий
- **TouchSwipe** - специализированная библиотека для свайпов

## Связанные темы

- [[Адаптивность]]
- [[Мобильные-API]]
- [[Оптимизация-для-мобильных]]
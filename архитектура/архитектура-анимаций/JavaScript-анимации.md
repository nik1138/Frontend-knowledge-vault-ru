---
aliases: ["JavaScript Анимации", "JS Анимации", "Анимации JS", "Web Animations API"]
tags: ["#javascript", "#анимации", "#frontend", "#производительность", "#архитектура"]
---

# JavaScript-анимации

JavaScript-анимации предоставляют более гибкий и мощный способ создания анимированных эффектов по сравнению с CSS-анимациями. Они позволяют динамически управлять параметрами анимации, создавать сложные сценарии и реагировать на пользовательские действия в реальном времени.

## Основные подходы к JavaScript-анимациям

### 1. Web Animations API

Современный стандарт, предоставляемый браузером:

```javascript
const element = document.querySelector('.animated-element');

const animation = element.animate([
  { transform: 'translateX(0px)' },
  { transform: 'translateX(100px)' }
], {
  duration: 1000,
  easing: 'ease-in-out',
  fill: 'forwards'
});

// Управление анимацией
animation.play();
animation.pause();
animation.reverse();
animation.currentTime = 500; // Перейти к середине анимации
```

### 2. requestAnimationFrame

Основа для плавных анимаций:

```javascript
function animateElement(element, targetValue) {
  let startTime = null;
  const startValue = 0;
  
  function animationStep(timestamp) {
    if (!startTime) startTime = timestamp;
    
    const progress = Math.min((timestamp - startTime) / 1000, 1); // 1 секунда
    
    // Линейная интерполяция
    const currentValue = startValue + (targetValue - startValue) * progress;
    element.style.transform = `translateX(${currentValue}px)`;
    
    if (progress < 1) {
      requestAnimationFrame(animationStep);
    } else {
      // Завершение анимации
      console.log('Анимация завершена');
    }
  }
  
  requestAnimationFrame(animationStep);
}
```

### 3. setTimeout/setInterval

Примитивный подход, менее предпочтительный:

```javascript
let position = 0;
const element = document.querySelector('.moving-element');

function moveElement() {
  position += 2;
  element.style.left = position + 'px';
  
  if (position < 200) {
    setTimeout(moveElement, 16); // ~60 FPS
  }
}

moveElement();
```

## Архитектура JavaScript-анимаций

### Класс для управления анимациями

```javascript
class AnimationManager {
  constructor() {
    this.animations = new Map();
    this.globalSpeed = 1;
  }
  
  createAnimation(element, keyframes, options = {}) {
    const defaultOptions = {
      duration: 300,
      easing: 'ease-in-out',
      fill: 'forwards'
    };
    
    const animation = element.animate(keyframes, {
      ...defaultOptions,
      ...options
    });
    
    // Учет глобальной скорости
    animation.playbackRate = this.globalSpeed;
    
    // Сохранение анимации для управления
    const id = this.generateId();
    this.animations.set(id, {
      animation,
      element,
      options
    });
    
    // Очистка после завершения
    animation.onfinish = () => {
      this.animations.delete(id);
    };
    
    return id;
  }
  
  pauseAnimation(id) {
    const animationData = this.animations.get(id);
    if (animationData) {
      animationData.animation.pause();
    }
  }
  
  resumeAnimation(id) {
    const animationData = this.animations.get(id);
    if (animationData) {
      animationData.animation.play();
    }
  }
  
  cancelAnimation(id) {
    const animationData = this.animations.get(id);
    if (animationData) {
      animationData.animation.cancel();
      this.animations.delete(id);
    }
  }
  
  generateId() {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }
  
  setGlobalSpeed(speed) {
    this.globalSpeed = speed;
    this.animations.forEach(data => {
      data.animation.playbackRate = speed;
    });
  }
}

// Использование
const animManager = new AnimationManager();
const element = document.querySelector('.my-element');

const animationId = animManager.createAnimation(
  element,
  [
    { transform: 'scale(1)', opacity: 1 },
    { transform: 'scale(1.2)', opacity: 0.8 },
    { transform: 'scale(1)', opacity: 1 }
  ],
  { duration: 500 }
);
```

## Сложные анимационные сценарии

### Последовательные анимации

```javascript
class SequentialAnimation {
  constructor() {
    this.queue = [];
    this.current = null;
  }
  
  add(animationFn) {
    this.queue.push(animationFn);
    return this;
  }
  
  async play() {
    while (this.queue.length > 0) {
      const animationFn = this.queue.shift();
      this.current = await animationFn();
    }
    this.current = null;
  }
  
  async parallel(animations) {
    return Promise.all(animations.map(fn => fn()));
  }
}

// Использование
const seqAnim = new SequentialAnimation();

seqAnim
  .add(() => {
    const el1 = document.querySelector('.element1');
    return el1.animate([
      { opacity: 0 },
      { opacity: 1 }
    ], { duration: 300 }).finished;
  })
  .add(() => {
    const el2 = document.querySelector('.element2');
    return el2.animate([
      { transform: 'translateY(50px)' },
      { transform: 'translateY(0)' }
    ], { duration: 500 }).finished;
  })
  .play();
```

### Анимации с физикой

```javascript
class PhysicsAnimation {
  constructor(element) {
    this.element = element;
    this.velocity = 0;
    this.position = 0;
    this.target = 0;
    this.friction = 0.85;
    this.spring = 0.1;
    this.animationFrame = null;
  }
  
  animateTo(target) {
    this.target = target;
    this.start();
  }
  
  start() {
    if (this.animationFrame) return;
    
    const update = () => {
      // Простая физика пружины
      const distance = this.target - this.position;
      this.velocity += distance * this.spring;
      this.velocity *= this.friction;
      this.position += this.velocity;
      
      // Применение позиции к элементу
      this.element.style.transform = `translateX(${this.position}px)`;
      
      // Продолжение анимации, если не достигнута цель
      if (Math.abs(distance) > 0.1 || Math.abs(this.velocity) > 0.1) {
        this.animationFrame = requestAnimationFrame(update);
      } else {
        this.velocity = 0;
        this.animationFrame = null;
      }
    };
    
    this.animationFrame = requestAnimationFrame(update);
  }
  
  stop() {
    if (this.animationFrame) {
      cancelAnimationFrame(this.animationFrame);
      this.animationFrame = null;
    }
  }
}
```

## Оптимизация производительности

### 1. Использование transform и opacity

```javascript
// Правильно - анимация через transform
element.style.transform = `translateX(${x}px)`;

// Неправильно - анимация свойств, вызывающих relayout
element.style.left = `${x}px`; // Избегать
element.style.width = `${width}px`; // Избегать
```

### 2. Повышение элементов на отдельный слой

```javascript
// Повышение элемента на GPU-слой
element.style.willChange = 'transform';
// или
element.style.transform = 'translateZ(0)';
```

### 3. Оптимизация рендеринга

```javascript
class OptimizedAnimation {
  constructor() {
    this.elements = new Set();
    this.isRunning = false;
    this.frameRate = 60;
    this.frameInterval = 1000 / this.frameRate;
    this.lastTime = 0;
  }
  
  addElement(element, updateFn) {
    this.elements.add({ element, updateFn });
    if (!this.isRunning) {
      this.start();
    }
  }
  
  start() {
    this.isRunning = true;
    const loop = (timestamp) => {
      if (timestamp - this.lastTime >= this.frameInterval) {
        this.lastTime = timestamp;
        
        // Обновление всех элементов
        this.elements.forEach(({ element, updateFn }) => {
          updateFn(element);
        });
      }
      
      if (this.elements.size > 0) {
        requestAnimationFrame(loop);
      } else {
        this.isRunning = false;
      }
    };
    
    requestAnimationFrame(loop);
  }
  
  removeElement(element) {
    this.elements.forEach((item, index) => {
      if (item.element === element) {
        this.elements.delete(index);
      }
    });
  }
}
```

## Практические рекомендации

### 1. Управление сложными анимациями

```javascript
class AnimationController {
  constructor() {
    this.groups = new Map();
    this.timelines = new Map();
  }
  
  createGroup(name, elements) {
    this.groups.set(name, elements);
  }
  
  createTimeline(name, animations) {
    this.timelines.set(name, {
      animations,
      currentIndex: 0,
      isPlaying: false
    });
  }
  
  playTimeline(name) {
    const timeline = this.timelines.get(name);
    if (!timeline || timeline.isPlaying) return;
    
    timeline.isPlaying = true;
    
    const playNext = () => {
      if (timeline.currentIndex >= timeline.animations.length) {
        timeline.isPlaying = false;
        timeline.currentIndex = 0;
        return;
      }
      
      const anim = timeline.animations[timeline.currentIndex];
      const promise = anim.play();
      
      promise.then(() => {
        timeline.currentIndex++;
        playNext();
      });
    };
    
    playNext();
  }
  
  stopTimeline(name) {
    const timeline = this.timelines.get(name);
    if (timeline) {
      timeline.isPlaying = false;
      timeline.currentIndex = 0;
    }
  }
}
```

### 2. Отключение анимаций для пользователей

```javascript
// Проверка предпочтений пользователя
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
const disableAnimations = prefersReducedMotion.matches;

class AnimationHandler {
  constructor() {
    this.disabled = disableAnimations;
  }
  
  animate(element, keyframes, options) {
    if (this.disabled) {
      // Мгновенное применение конечного состояния
      const finalFrame = keyframes[keyframes.length - 1];
      Object.keys(finalFrame).forEach(prop => {
        element.style[prop] = finalFrame[prop];
      });
      return Promise.resolve();
    }
    
    return element.animate(keyframes, options).finished;
  }
}
```

### 3. Адаптация под производительность устройства

```javascript
class AdaptiveAnimation {
  constructor() {
    this.performanceLevel = this.detectPerformanceLevel();
    this.multiplier = this.getMultiplier();
  }
  
  detectPerformanceLevel() {
    // Простая оценка производительности
    if ('deviceMemory' in navigator) {
      const memory = navigator.deviceMemory;
      if (memory < 4) return 'low';
      if (memory < 8) return 'medium';
      return 'high';
    }
    return 'medium'; // по умолчанию
  }
  
  getMultiplier() {
    switch (this.performanceLevel) {
      case 'low': return 0.5; // Уменьшить интенсивность
      case 'medium': return 1;
      case 'high': return 1.2;
      default: return 1;
    }
  }
  
  animate(element, keyframes, options) {
    // Адаптация продолжительности анимации
    const adaptedOptions = {
      ...options,
      duration: options.duration * this.multiplier
    };
    
    return element.animate(keyframes, adaptedOptions);
  }
}
```

## Современные возможности JavaScript-анимаций

### Animation Worklet API

```javascript
// Регистрация анимационного ворклета
if ('animationWorklet' in CSS) {
  CSS.animationWorklet.addModule('scroll-linked-animation.js');
  
  // Использование ворклета
  element.animate([], {
    timeline: new ViewTimeline({
      subject: element,
      axis: 'block'
    })
  });
}
```

### Scroll-driven animations через JavaScript

```javascript
class ScrollDrivenAnimation {
  constructor(element, triggerElement) {
    this.element = element;
    this.trigger = triggerElement;
    this.animation = null;
    this.isActive = false;
    
    this.handleScroll = this.handleScroll.bind(this);
    this.init();
  }
  
  init() {
    window.addEventListener('scroll', this.handleScroll);
  }
  
  handleScroll() {
    const rect = this.trigger.getBoundingClientRect();
    const viewportHeight = window.innerHeight;
    const progress = Math.max(0, Math.min(1, (viewportHeight - rect.top) / rect.height));
    
    // Обновление анимации в зависимости от прогресса
    if (this.animation) {
      this.animation.currentTime = this.animation.effect.getTiming().duration * progress;
    }
  }
  
  animate(keyframes, options) {
    this.animation = this.element.animate(keyframes, options);
    this.animation.pause(); // Управление через прокрутку
  }
  
  destroy() {
    window.removeEventListener('scroll', this.handleScroll);
    if (this.animation) {
      this.animation.cancel();
    }
  }
}
```

## Лучшие практики в российском контексте 2025 года

1. **Соответствие требованиям доступности**
   - Обязательное соблюдение российских стандартов доступности
   - Учет особенностей пользователей с ограниченными возможностями
   - Проверка анимаций на предмет потенциального риска для пользователей с эпилепсией

2. **Производительность на слабых устройствах**
   - Адаптация анимаций под производительность устройства
   - Учет особенностей бюджетных устройств, популярных в России
   - Оптимизация для различных брендов смартфонов

3. **Реагирование на сетевые условия**
   - Уменьшение интенсивности анимаций при медленном соединении
   - Использование Network Information API для адаптации анимаций

## Связь с другими архитектурными компонентами

JavaScript-анимации интегрируются с другими частями архитектуры фронтенда:

- [[CSS-анимации]] - комбинирование с CSS-анимациями
- [[Архитектура интерфейсов]] - анимации как часть UX
- [[Производительность фронтенда]] - оптимизация анимаций
- [[Библиотеки анимаций]] - интеграция с JS-библиотеками
- [[Тестирование анимаций]] - автоматизированное тестирование JS-анимаций
- [[State-менеджмент]] - синхронизация анимаций с состоянием приложения
- [[Архитектура данных]] - анимации при обновлении данных

## Заключение

JavaScript-анимации предоставляют мощные возможности для создания сложных и интерактивных анимационных эффектов. Правильная архитектура JavaScript-анимаций позволяет создавать гибкие, производительные и доступные интерфейсы, что особенно важно для российского рынка с его разнообразием устройств и пользовательских потребностей.
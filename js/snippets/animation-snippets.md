---
aliases: ["Сниппеты анимаций", "JS Animation Snippets"]
tags: [javascript, animation, frontend, snippets]
---

# Сниппеты для анимаций на JavaScript

Набор готовых сниппетов для создания анимаций на JavaScript. Эти сниппеты можно использовать для плавных переходов, анимаций элементов и создания интерактивных интерфейсов.

## Основные анимации

### 1. Плавное изменение стиля элемента

```javascript
/**
 * Плавно изменяет CSS свойство элемента
 * @param {HTMLElement} element - элемент для анимации
 * @param {string} property - CSS свойство для анимации
 * @param {number} startValue - начальное значение
 * @param {number} endValue - конечное значение
 * @param {number} duration - продолжительность анимации в миллисекундах
 * @param {function} callback - функция, вызываемая по окончании анимации
 */
function animateProperty(element, property, startValue, endValue, duration, callback) {
  const startTime = performance.now();
  
  function animate(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // Используем easing функцию для плавности (ease-in-out)
    const easeProgress = 0.5 - 0.5 * Math.cos(Math.PI * progress);
    
    const currentValue = startValue + (endValue - startValue) * easeProgress;
    
    // Применяем значение к свойству
    if (property === 'opacity') {
      element.style.opacity = currentValue;
    } else if (property === 'left' || property === 'top' || property === 'width' || property === 'height') {
      element.style[property] = currentValue + 'px';
    } else {
      element.style[property] = currentValue;
    }
    
    if (progress < 1) {
      requestAnimationFrame(animate);
    } else if (callback) {
      callback();
    }
  }
  
  requestAnimationFrame(animate);
}

// Пример использования
const element = document.getElementById('myElement');
animateProperty(element, 'opacity', 0, 1, 1000); // Плавное появление
animateProperty(element, 'left', 0, 200, 1000); // Перемещение вправо
```

### 2. Анимация появления/исчезновения

```javascript
/**
 * Плавное появление элемента
 * @param {HTMLElement} element - элемент для анимации
 * @param {number} duration - продолжительность анимации
 * @param {function} callback - функция, вызываемая по окончании
 */
function fadeIn(element, duration = 300, callback) {
  element.style.opacity = 0;
  element.style.display = 'block';
  
  animateProperty(element, 'opacity', 0, 1, duration, callback);
}

/**
 * Плавное исчезновение элемента
 * @param {HTMLElement} element - элемент для анимации
 * @param {number} duration - продолжительность анимации
 * @param {function} callback - функция, вызываемая по окончании
 */
function fadeOut(element, duration = 300, callback) {
  animateProperty(element, 'opacity', 1, 0, duration, () => {
    element.style.display = 'none';
    if (callback) callback();
  });
}

// Пример использования
const modal = document.getElementById('modal');
fadeIn(modal, 500, () => console.log('Модальное окно появилось'));
```

### 3. Анимация изменения размера

```javascript
/**
 * Анимация изменения размеров элемента
 * @param {HTMLElement} element - элемент для анимации
 * @param {number} startWidth - начальная ширина
 * @param {number} endWidth - конечная ширина
 * @param {number} startHeight - начальная высота
 * @param {number} endHeight - конечная высота
 * @param {number} duration - продолжительность анимации
 */
function animateResize(element, startWidth, endWidth, startHeight, endHeight, duration) {
  const startTime = performance.now();
  
  function animate(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // Easing для плавности
    const easeProgress = 0.5 - 0.5 * Math.cos(Math.PI * progress);
    
    const width = startWidth + (endWidth - startWidth) * easeProgress;
    const height = startHeight + (endHeight - startHeight) * easeProgress;
    
    element.style.width = width + 'px';
    element.style.height = height + 'px';
    
    if (progress < 1) {
      requestAnimationFrame(animate);
    }
  }
  
  requestAnimationFrame(animate);
}

// Пример использования
const box = document.getElementById('box');
animateResize(box, 100, 300, 100, 200, 1000);
```

## Перемещение и трансформации

### 1. Анимация перемещения элемента

```javascript
/**
 * Анимация перемещения элемента
 * @param {HTMLElement} element - элемент для анимации
 * @param {number} startX - начальная позиция X
 * @param {number} endX - конечная позиция X
 * @param {number} startY - начальная позиция Y
 * @param {number} endY - конечная позиция Y
 * @param {number} duration - продолжительность анимации
 */
function animateMove(element, startX, endX, startY, endY, duration) {
  // Убедимся, что элемент может быть позиционирован
  if (element.style.position !== 'absolute' && element.style.position !== 'relative') {
    element.style.position = 'relative';
  }
  
  const startTime = performance.now();
  
  function animate(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // Easing функция (ease-in-out)
    const easeProgress = 0.5 - 0.5 * Math.cos(Math.PI * progress);
    
    const currentX = startX + (endX - startX) * easeProgress;
    const currentY = startY + (endY - startY) * easeProgress;
    
    element.style.left = currentX + 'px';
    element.style.top = currentY + 'px';
    
    if (progress < 1) {
      requestAnimationFrame(animate);
    }
  }
  
  requestAnimationFrame(animate);
}

// Пример использования
const movableElement = document.getElementById('movable');
animateMove(movableElement, 0, 300, 0, 200, 1000);
```

### 2. Анимация трансформаций (rotate, scale, skew)

```javascript
/**
 * Анимация трансформаций элемента
 * @param {HTMLElement} element - элемент для анимации
 * @param {object} startTransforms - начальные трансформации
 * @param {object} endTransforms - конечные трансформации
 * @param {number} duration - продолжительность анимации
 */
function animateTransform(element, startTransforms, endTransforms, duration) {
  const startTime = performance.now();
  
  // Устанавливаем начальные трансформации
  applyTransform(element, startTransforms);
  
  function animate(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // Easing функция
    const easeProgress = 0.5 - 0.5 * Math.cos(Math.PI * progress);
    
    // Интерполируем каждую трансформацию
    const currentTransforms = {};
    
    for (const [transformType, startValue] of Object.entries(startTransforms)) {
      const endValue = endTransforms[transformType] || startValue;
      currentTransforms[transformType] = startValue + (endValue - startValue) * easeProgress;
    }
    
    applyTransform(element, currentTransforms);
    
    if (progress < 1) {
      requestAnimationFrame(animate);
    }
  }
  
  requestAnimationFrame(animate);
}

/**
 * Применяет трансформации к элементу
 * @param {HTMLElement} element - элемент
 * @param {object} transforms - объект с трансформациями
 */
function applyTransform(element, transforms) {
  let transformString = '';
  
  if (transforms.translateX !== undefined) {
    transformString += `translateX(${transforms.translateX}px) `;
  }
  if (transforms.translateY !== undefined) {
    transformString += `translateY(${transforms.translateY}px) `;
  }
  if (transforms.scale !== undefined) {
    transformString += `scale(${transforms.scale}) `;
  }
  if (transforms.rotate !== undefined) {
    transformString += `rotate(${transforms.rotate}deg) `;
  }
  if (transforms.skewX !== undefined) {
    transformString += `skewX(${transforms.skewX}deg) `;
  }
  if (transforms.skewY !== undefined) {
    transformString += `skewY(${transforms.skewY}deg) `;
  }
  
  element.style.transform = transformString.trim();
}

// Пример использования
const transformElement = document.getElementById('transformElement');
animateTransform(
  transformElement,
  { scale: 1, rotate: 0, translateX: 0, translateY: 0 },
  { scale: 1.5, rotate: 45, translateX: 100, translateY: 50 },
  1000
);
```

## Сложные анимации

### 1. Анимация по кривой (Безье)

```javascript
/**
 * Анимация по кривой Безье
 * @param {HTMLElement} element - элемент для анимации
 * @param {Array} start - начальная точка [x, y]
 * @param {Array} control1 - первая контрольная точка [x, y]
 * @param {Array} control2 - вторая контрольная точка [x, y]
 * @param {Array} end - конечная точка [x, y]
 * @param {number} duration - продолжительность анимации
 */
function animateBezierCurve(element, start, control1, control2, end, duration) {
  // Убедимся, что элемент может быть позиционирован
  if (element.style.position !== 'absolute' && element.style.position !== 'relative') {
    element.style.position = 'relative';
  }
  
  const startTime = performance.now();
  
  function animate(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // Кубическая кривая Безье
    const t = progress;
    const x = Math.pow(1 - t, 3) * start[0] +
              3 * Math.pow(1 - t, 2) * t * control1[0] +
              3 * (1 - t) * Math.pow(t, 2) * control2[0] +
              Math.pow(t, 3) * end[0];
              
    const y = Math.pow(1 - t, 3) * start[1] +
              3 * Math.pow(1 - t, 2) * t * control1[1] +
              3 * (1 - t) * Math.pow(t, 2) * control2[1] +
              Math.pow(t, 3) * end[1];
    
    element.style.left = x + 'px';
    element.style.top = y + 'px';
    
    if (progress < 1) {
      requestAnimationFrame(animate);
    }
  }
  
  requestAnimationFrame(animate);
}

// Пример использования
const bezierElement = document.getElementById('bezierElement');
animateBezierCurve(
  bezierElement,
  [0, 0],        // начальная точка
  [100, -100],   // первая контрольная точка
  [200, 100],    // вторая контрольная точка
  [300, 0]       // конечная точка
, 2000);
```

### 2. Анимация с физическими эффектами (пружина, затухание)

```javascript
/**
 * Анимация с физическим поведением (пружина)
 * @param {HTMLElement} element - элемент для анимации
 * @param {number} targetX - целевая позиция X
 * @param {number} targetY - целевая позиция Y
 * @param {object} physics - параметры физики { stiffness, damping, mass }
 */
function animateWithPhysics(element, targetX, targetY, physics = {}) {
  // Убедимся, что элемент может быть позиционирован
  if (element.style.position !== 'absolute' && element.style.position !== 'relative') {
    element.style.position = 'relative';
  }
  
  // Параметры по умолчанию
  const params = {
    stiffness: physics.stiffness || 0.3,  // Жесткость пружины
    damping: physics.damping || 0.7,      // Затухание
    mass: physics.mass || 1              // Масса
  };
  
  // Начальные значения
  let x = parseFloat(element.style.left) || 0;
  let y = parseFloat(element.style.top) || 0;
  let vx = 0; // Скорость по X
  let vy = 0; // Скорость по Y
  
  // Сила пружины
  let fx = 0;
  let fy = 0;
  
  function animate() {
    // Вычисляем силы
    fx = -params.stiffness * (x - targetX) - params.damping * vx;
    fy = -params.stiffness * (y - targetY) - params.damping * vy;
    
    // Вычисляем ускорение (a = F/m)
    const ax = fx / params.mass;
    const ay = fy / params.mass;
    
    // Обновляем скорость (v = v + a*dt)
    // Предполагаем dt = 1/60 секунды (60 FPS)
    vx += ax * (1/60);
    vy += ay * (1/60);
    
    // Обновляем позицию (p = p + v*dt)
    x += vx * (1/60);
    y += vy * (1/60);
    
    // Применяем новую позицию
    element.style.left = x + 'px';
    element.style.top = y + 'px';
    
    // Продолжаем анимацию, если разница больше порога
    const dx = Math.abs(x - targetX);
    const dy = Math.abs(y - targetY);
    const threshold = 0.1; // Порог остановки
    
    if (dx > threshold || dy > threshold || Math.abs(vx) > 0.1 || Math.abs(vy) > 0.1) {
      requestAnimationFrame(animate);
    }
  }
  
  requestAnimationFrame(animate);
}

// Пример использования
const physicsElement = document.getElementById('physicsElement');
animateWithPhysics(physicsElement, 300, 200, { stiffness: 0.5, damping: 0.6, mass: 1 });
```

## Анимация с использованием CSS

### 1. Динамическое добавление CSS классов для анимации

```javascript
/**
 * Добавляет анимационный класс и удаляет его после завершения
 * @param {HTMLElement} element - элемент для анимации
 * @param {string} className - CSS класс анимации
 * @param {number} duration - продолжительность анимации
 */
function animateWithCSSClass(element, className, duration) {
  element.classList.add(className);
  
  setTimeout(() => {
    element.classList.remove(className);
  }, duration);
}

// Пример CSS для анимации:
/*
@keyframes bounce {
  0%, 20%, 53%, 80%, 100% {
    transform: translate3d(0,0,0);
  }
  40%, 43% {
    transform: translate3d(0, -30px, 0);
  }
  70% {
    transform: translate3d(0, -15px, 0);
  }
  90% {
    transform: translate3d(0, -4px, 0);
  }
}

.bounce-animation {
  animation: bounce 1s ease-in-out;
}
*/

// Пример использования
const animatedElement = document.getElementById('animatedElement');
animateWithCSSClass(animatedElement, 'bounce-animation', 1000);
```

### 2. Анимация с использованием Web Animations API

```javascript
/**
 * Анимация с использованием Web Animations API
 * @param {HTMLElement} element - элемент для анимации
 * @param {Array} keyframes - ключевые кадры анимации
 * @param {object} options - опции анимации
 */
function animateWithWebAPI(element, keyframes, options = {}) {
  // Опции по умолчанию
  const defaultOptions = {
    duration: 1000,
    easing: 'ease-in-out',
    fill: 'forwards'
  };
  
  const animationOptions = { ...defaultOptions, ...options };
  
  return element.animate(keyframes, animationOptions);
}

// Пример использования
const webAnimationElement = document.getElementById('webAnimationElement');

// Анимация появления
const fadeInKeyframes = [
  { opacity: 0 },
  { opacity: 1 }
];

const fadeInAnimation = animateWithWebAPI(webAnimationElement, fadeInKeyframes, {
  duration: 800,
  easing: 'ease-out'
});

// Цепочка анимаций
fadeInAnimation.onfinish = () => {
  const moveKeyframes = [
    { transform: 'translateX(0px)' },
    { transform: 'translateX(200px)' }
  ];
  
  animateWithWebAPI(webAnimationElement, moveKeyframes, {
    duration: 1000,
    easing: 'ease-in-out'
  });
};
```

## Утилиты для анимации

### 1. Easing функции

```javascript
/**
 * Коллекция easing функций для анимаций
 */
const EasingFunctions = {
  // Linear
  linear: t => t,
  
  // Quadratic
  easeInQuad: t => t * t,
  easeOutQuad: t => t * (2 - t),
  easeInOutQuad: t => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t,
  
  // Cubic
  easeInCubic: t => t * t * t,
  easeOutCubic: t => (--t) * t * t + 1,
  easeInOutCubic: t => t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1,
  
  // Quartic
  easeInQuart: t => t * t * t * t,
  easeOutQuart: t => 1 - (--t) * t * t * t,
  easeInOutQuart: t => t < 0.5 ? 8 * t * t * t * t : 1 - 8 * (--t) * t * t * t,
  
  // Quintic
  easeInQuint: t => t * t * t * t * t,
  easeOutQuint: t => 1 + (--t) * t * t * t * t,
  easeInOutQuint: t => t < 0.5 ? 16 * t * t * t * t * t : 1 + 16 * (--t) * t * t * t * t
};

/**
 * Универсальная функция анимации с выбором easing функции
 * @param {HTMLElement} element - элемент для анимации
 * @param {string} property - CSS свойство
 * @param {number} startValue - начальное значение
 * @param {number} endValue - конечное значение
 * @param {number} duration - продолжительность
 * @param {string} easingType - тип easing функции
 * @param {function} callback - функция по окончании
 */
function animateWithEasing(element, property, startValue, endValue, duration, easingType = 'easeInOutQuad', callback) {
  const startTime = performance.now();
  const easingFunction = EasingFunctions[easingType] || EasingFunctions.easeInOutQuad;
  
  function animate(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    const easeProgress = easingFunction(progress);
    const currentValue = startValue + (endValue - startValue) * easeProgress;
    
    if (property === 'opacity') {
      element.style.opacity = currentValue;
    } else if (property === 'left' || property === 'top' || property === 'width' || property === 'height') {
      element.style[property] = currentValue + 'px';
    } else {
      element.style[property] = currentValue;
    }
    
    if (progress < 1) {
      requestAnimationFrame(animate);
    } else if (callback) {
      callback();
    }
  }
  
  requestAnimationFrame(animate);
}

// Пример использования
const easingElement = document.getElementById('easingElement');
animateWithEasing(easingElement, 'left', 0, 300, 1000, 'easeOutBounce', () => {
  console.log('Анимация завершена!');
});
```

### 2. Менеджер анимаций

```javascript
/**
 * Менеджер анимаций для управления несколькими анимациями
 */
class AnimationManager {
  constructor() {
    this.animations = new Map();
    this.globalPaused = false;
  }
  
  /**
   * Добавляет анимацию
   * @param {string} id - идентификатор анимации
   * @param {Function} animationFunction - функция анимации
   */
  addAnimation(id, animationFunction) {
    this.animations.set(id, {
      id,
      animation: animationFunction,
      active: true
    });
  }
  
  /**
   * Запускает анимацию
   * @param {string} id - идентификатор анимации
   */
  play(id) {
    const animation = this.animations.get(id);
    if (animation && !animation.active) {
      animation.active = true;
      animation.animation();
    }
  }
  
  /**
   * Приостанавливает анимацию
   * @param {string} id - идентификатор анимации
   */
  pause(id) {
    const animation = this.animations.get(id);
    if (animation) {
      animation.active = false;
    }
  }
  
  /**
   * Удаляет анимацию
   * @param {string} id - идентификатор анимации
   */
  remove(id) {
    this.animations.delete(id);
  }
  
  /**
   * Приостанавливает все анимации
   */
  pauseAll() {
    this.globalPaused = true;
    for (const animation of this.animations.values()) {
      animation.active = false;
    }
  }
  
  /**
   * Запускает все анимации
   */
  playAll() {
    this.globalPaused = false;
    for (const animation of this.animations.values()) {
      if (animation.active !== false) {
        animation.active = true;
        animation.animation();
      }
    }
  }
}

// Пример использования
const animationManager = new AnimationManager();

// Добавляем несколько анимаций
animationManager.addAnimation('fadeBox', () => {
  const box = document.getElementById('fadeBox');
  animateProperty(box, 'opacity', 0, 1, 2000);
});

animationManager.addAnimation('moveBox', () => {
  const box = document.getElementById('moveBox');
  animateMove(box, 0, 200, 0, 100, 2000);
});

// Запускаем анимации
animationManager.play('fadeBox');
animationManager.play('moveBox');
```

## Практические советы

- Используйте `transform` и `opacity` для анимаций, так как они не вызывают перерисовку
- Оптимизируйте анимации с помощью `will-change` CSS свойства
- Используйте `requestAnimationFrame` для плавных анимаций
- Учитывайте производительность при создании сложных анимаций
- Добавляйте опции отключения анимаций для пользователей с предпочтениями

## Связанные темы

- [[css-animation-techniques]]
- [[performance-optimization-techniques]]
- [[web-animations-api-guide]]
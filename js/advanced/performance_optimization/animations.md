# Оптимизация анимаций в JavaScript

## Введение

Анимации играют важную роль в создании привлекательных и интерактивных веб-приложений. Однако неправильно реализованные анимации могут значительно ухудшить производительность и пользовательский опыт. В этом разделе мы рассмотрим различные техники оптимизации анимаций в JavaScript.

## Использование requestAnimationFrame

Правильная синхронизация анимаций с браузером:

```javascript
// Оптимизация с requestAnimationFrame
class AnimationOptimization {
  // Базовая анимация с requestAnimationFrame
  static createAnimation(element, duration, from, to, property) {
    const startTime = performance.now();
    
    function animate(currentTime) {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      
      // Линейная интерполяция
      const value = from + (to - from) * progress;
      element.style[property] = `${value}px`;
      
      if (progress < 1) {
        requestAnimationFrame(animate);
      }
    }
    
    requestAnimationFrame(animate);
  }
  
  // Анимация с easing функциями
  static createEasedAnimation(element, duration, from, to, property, easing = 'easeInOutCubic') {
    const startTime = performance.now();
    const easingFunction = this.getEasingFunction(easing);
    
    function animate(currentTime) {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      const easedProgress = easingFunction(progress);
      
      const value = from + (to - from) * easedProgress;
      element.style[property] = `${value}px`;
      
      if (progress < 1) {
        requestAnimationFrame(animate);
      }
    }
    
    requestAnimationFrame(animate);
  }
  
  // Easing функции
  static getEasingFunction(name) {
    const easings = {
      linear: t => t,
      easeInQuad: t => t * t,
      easeOutQuad: t => t * (2 - t),
      easeInOutQuad: t => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t,
      easeInCubic: t => t * t * t,
      easeOutCubic: t => (--t) * t * t + 1,
      easeInOutCubic: t => t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1,
      easeInQuart: t => t * t * t * t,
      easeOutQuart: t => 1 - (--t) * t * t * t,
      easeInOutQuart: t => t < 0.5 ? 8 * t * t * t * t : 1 - 8 * (--t) * t * t * t
    };
    
    return easings[name] || easings.linear;
  }
  
  // Анимация с отменой
  static createCancellableAnimation(element, duration, from, to, property) {
    const startTime = performance.now();
    let cancelled = false;
    
    function animate(currentTime) {
      if (cancelled) return;
      
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      
      const value = from + (to - from) * progress;
      element.style[property] = `${value}px`;
      
      if (progress < 1) {
        requestAnimationFrame(animate);
      }
    }
    
    const animationId = requestAnimationFrame(animate);
    
    return {
      cancel: () => {
        cancelled = true;
        cancelAnimationFrame(animationId);
      }
    };
  }
  
  // Групповая анимация
  static createGroupAnimation(animations, onComplete) {
    let completedCount = 0;
    const totalAnimations = animations.length;
    
    animations.forEach(({ element, duration, from, to, property, easing }) => {
      this.createEasedAnimation(element, duration, from, to, property, easing);
      
      // Отслеживаем завершение анимации
      setTimeout(() => {
        completedCount++;
        if (completedCount === totalAnimations && onComplete) {
          onComplete();
        }
      }, duration);
    });
  }
}

// Пример использования оптимизированных анимаций
class AnimatedElement {
  constructor(element) {
    this.element = element;
    this.currentAnimations = new Set();
  }
  
  // Анимация перемещения
  move(x, y, duration = 300) {
    this.cancelAllAnimations();
    
    const animation = AnimationOptimization.createEasedAnimation(
      this.element,
      duration,
      parseInt(this.element.style.left || 0),
      x,
      'left',
      'easeOutCubic'
    );
    
    this.currentAnimations.add(animation);
    return animation;
  }
  
  // Анимация масштабирования
  scale(scale, duration = 300) {
    this.cancelAllAnimations();
    
    const animation = AnimationOptimization.createEasedAnimation(
      this.element,
      duration,
      parseFloat(this.element.style.transform.replace('scale(', '').replace(')', '')) || 1,
      scale,
      'transform',
      'easeInOutQuad'
    );
    
    // Для transform нужно специальное обращение
    const startTime = performance.now();
    const fromScale = parseFloat(this.element.style.transform.replace('scale(', '').replace(')', '')) || 1;
    
    const animate = (currentTime) => {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      const easedProgress = AnimationOptimization.getEasingFunction('easeInOutQuad')(progress);
      const currentScale = fromScale + (scale - fromScale) * easedProgress;
      this.element.style.transform = `scale(${currentScale})`;
      
      if (progress < 1) {
        requestAnimationFrame(animate);
      }
    };
    
    requestAnimationFrame(animate);
  }
  
  // Отмена всех анимаций
  cancelAllAnimations() {
    this.currentAnimations.forEach(animation => {
      if (animation.cancel) {
        animation.cancel();
      }
    });
    this.currentAnimations.clear();
  }
}
```

## Оптимизация CSS анимаций

Использование CSS для лучшей производительности:

```javascript
// Оптимизация CSS анимаций
class CSSAnimationOptimization {
  // Создание CSS анимации через JavaScript
  static createCSSAnimation(element, keyframes, options) {
    // Создаем уникальное имя для анимации
    const animationName = `anim_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    
    // Создаем CSS правило
    const styleSheet = document.createElement('style');
    styleSheet.textContent = `
      @keyframes ${animationName} {
        ${this.generateKeyframes(keyframes)}
      }
    `;
    document.head.appendChild(styleSheet);
    
    // Применяем анимацию к элементу
    element.style.animation = `${animationName} ${options.duration}ms ${options.easing || 'ease'} ${options.delay || 0}ms ${options.iterations || 1} ${options.direction || 'normal'} ${options.fillMode || 'forwards'}`;
    
    // Очистка после завершения анимации
    if (options.cleanup !== false) {
      element.addEventListener('animationend', () => {
        element.style.animation = '';
        document.head.removeChild(styleSheet);
      }, { once: true });
    }
    
    return {
      element,
      animationName,
      styleSheet,
      cancel: () => {
        element.style.animation = '';
        document.head.removeChild(styleSheet);
      }
    };
  }
  
  // Генерация keyframes из объекта
  static generateKeyframes(keyframes) {
    return Object.entries(keyframes)
      .map(([percentage, styles]) => {
        const styleString = Object.entries(styles)
          .map(([prop, value]) => `${this.camelToKebab(prop)}: ${value}`)
          .join('; ');
        return `${percentage} { ${styleString} }`;
      })
      .join('\n');
  }
  
  // Преобразование camelCase в kebab-case
  static camelToKebab(str) {
    return str.replace(/[A-Z]/g, match => `-${match.toLowerCase()}`);
  }
  
  // Оптимизация transform анимаций
  static optimizeTransformAnimations() {
    // Создаем CSS классы для оптимизированных анимаций
    const style = document.createElement('style');
    style.textContent = `
      .optimized-transform {
        will-change: transform;
        transform: translateZ(0); /* Создает новый compositing layer */
      }
      
      .smooth-transition {
        transition: transform 0.3s ease-out;
      }
      
      .bounce-animation {
        animation: bounce 0.6s ease-in-out;
      }
      
      @keyframes bounce {
        0%, 20%, 53%, 80%, 100% {
          transition-timing-function: cubic-bezier(0.215, 0.610, 0.355, 1.000);
          transform: translate3d(0, 0, 0);
        }
        40%, 43% {
          transition-timing-function: cubic-bezier(0.755, 0.050, 0.855, 0.060);
          transform: translate3d(0, -30px, 0);
        }
        70% {
          transition-timing-function: cubic-bezier(0.755, 0.050, 0.855, 0.060);
          transform: translate3d(0, -15px, 0);
        }
        90% {
          transform: translate3d(0, -4px, 0);
        }
      }
    `;
    document.head.appendChild(style);
  }
  
  // Использование will-change для оптимизации
  static applyWillChange(element, properties) {
    element.style.willChange = properties.join(', ');
    
    // Удаляем will-change после завершения анимации
    setTimeout(() => {
      element.style.willChange = 'auto';
    }, 1000);
  }
  
  // Создание hardware-accelerated анимаций
  static createHardwareAcceleratedAnimation(element) {
    // Принудительное создание compositing layer
    element.style.transform = 'translateZ(0)';
    element.style.backfaceVisibility = 'hidden';
    
    return {
      // Анимация с использованием transform вместо left/top
      move: (x, y, duration = 300) => {
        element.style.transition = `transform ${duration}ms ease-out`;
        element.style.transform = `translate(${x}px, ${y}px)`;
      },
      
      // Анимация масштабирования
      scale: (scale, duration = 300) => {
        element.style.transition = `transform ${duration}ms ease-out`;
        element.style.transform = `scale(${scale})`;
      },
      
      // Анимация поворота
      rotate: (angle, duration = 300) => {
        element.style.transition = `transform ${duration}ms ease-out`;
        element.style.transform = `rotate(${angle}deg)`;
      }
    };
  }
}

// Пример использования CSS оптимизаций
class OptimizedAnimator {
  constructor(element) {
    this.element = element;
    this.hardwareAnimator = CSSAnimationOptimization.createHardwareAcceleratedAnimation(element);
  }
  
  // Плавное перемещение
  smoothMove(x, y, duration = 300) {
    this.hardwareAnimator.move(x, y, duration);
  }
  
  // Анимация появления
  fadeIn(duration = 300) {
    this.element.style.opacity = '0';
    this.element.style.transition = `opacity ${duration}ms ease-out`;
    
    // Принудительный reflow
    this.element.offsetHeight;
    
    this.element.style.opacity = '1';
  }
  
  // Анимация исчезновения
  fadeOut(duration = 300) {
    this.element.style.transition = `opacity ${duration}ms ease-out`;
    this.element.style.opacity = '0';
    
    // Удаляем элемент после анимации
    setTimeout(() => {
      if (this.element.parentNode) {
        this.element.parentNode.removeChild(this.element);
      }
    }, duration);
  }
  
  // Bounce анимация
  bounce() {
    this.element.classList.add('bounce-animation');
    
    // Удаляем класс после завершения анимации
    this.element.addEventListener('animationend', () => {
      this.element.classList.remove('bounce-animation');
    }, { once: true });
  }
}
```

## Создание анимационного движка

Расширенный анимационный движок:

```javascript
// Анимационный движок
class AnimationEngine {
  constructor() {
    this.animations = new Map();
    this.rafId = null;
    this.isRunning = false;
    this.globalTime = 0;
  }
  
  // Добавление анимации
  add(animationConfig) {
    const animationId = Symbol('animation');
    const startTime = this.globalTime;
    
    const animation = {
      id: animationId,
      ...animationConfig,
      startTime,
      progress: 0,
      isFinished: false
    };
    
    this.animations.set(animationId, animation);
    
    if (!this.isRunning) {
      this.start();
    }
    
    return animationId;
  }
  
  // Удаление анимации
  remove(animationId) {
    this.animations.delete(animationId);
    
    if (this.animations.size === 0) {
      this.stop();
    }
  }
  
  // Запуск анимационного цикла
  start() {
    if (this.isRunning) return;
    
    this.isRunning = true;
    this.lastTime = performance.now();
    
    const animate = (currentTime) => {
      const deltaTime = currentTime - this.lastTime;
      this.globalTime += deltaTime;
      this.lastTime = currentTime;
      
      this.updateAnimations(deltaTime);
      
      if (this.isRunning) {
        this.rafId = requestAnimationFrame(animate);
      }
    };
    
    this.rafId = requestAnimationFrame(animate);
  }
  
  // Остановка анимационного цикла
  stop() {
    this.isRunning = false;
    if (this.rafId) {
      cancelAnimationFrame(this.rafId);
      this.rafId = null;
    }
  }
  
  // Обновление анимаций
  updateAnimations(deltaTime) {
    for (const [id, animation] of this.animations) {
      if (animation.isFinished) continue;
      
      const elapsed = this.globalTime - animation.startTime;
      const progress = Math.min(elapsed / animation.duration, 1);
      animation.progress = progress;
      
      // Применяем easing
      const easedProgress = animation.easing 
        ? this.getEasingFunction(animation.easing)(progress)
        : progress;
      
      // Обновляем свойства
      this.updateProperties(animation, easedProgress);
      
      // Проверяем завершение
      if (progress >= 1) {
        animation.isFinished = true;
        if (animation.onComplete) {
          animation.onComplete();
        }
        if (!animation.loop) {
          this.remove(id);
        } else {
          animation.startTime = this.globalTime;
          animation.progress = 0;
          animation.isFinished = false;
        }
      }
    }
  }
  
  // Обновление свойств анимации
  updateProperties(animation, progress) {
    const { element, from, to } = animation;
    
    Object.keys(from).forEach(property => {
      const fromValue = from[property];
      const toValue = to[property];
      const currentValue = fromValue + (toValue - fromValue) * progress;
      
      if (property === 'transform') {
        // Специальная обработка transform
        element.style.transform = this.interpolateTransform(
          animation.fromTransform || fromValue,
          animation.toTransform || toValue,
          progress
        );
      } else {
        element.style[property] = this.formatValue(currentValue, property);
      }
    });
  }
  
  // Интерполяция transform значений
  interpolateTransform(fromTransform, toTransform, progress) {
    // Простая реализация для translate, scale, rotate
    const translateMatch = fromTransform.match(/translate\(([^)]+)\)/);
    const scaleMatch = fromTransform.match(/scale\(([^)]+)\)/);
    const rotateMatch = fromTransform.match(/rotate\(([^)]+)\)/);
    
    let result = '';
    
    if (translateMatch) {
      const fromCoords = translateMatch[1].split(',').map(v => parseFloat(v));
      const toCoords = toTransform.match(/translate\(([^)]+)\)/)[1].split(',').map(v => parseFloat(v));
      const currentCoords = fromCoords.map((coord, i) => coord + (toCoords[i] - coord) * progress);
      result += `translate(${currentCoords.join(', ')}) `;
    }
    
    if (scaleMatch) {
      const fromScale = parseFloat(scaleMatch[1]);
      const toScale = parseFloat(toTransform.match(/scale\(([^)]+)\)/)[1]);
      const currentScale = fromScale + (toScale - fromScale) * progress;
      result += `scale(${currentScale}) `;
    }
    
    if (rotateMatch) {
      const fromAngle = parseFloat(rotateMatch[1]);
      const toAngle = parseFloat(toTransform.match(/rotate\(([^)]+)\)/)[1]);
      const currentAngle = fromAngle + (toAngle - fromAngle) * progress;
      result += `rotate(${currentAngle}deg) `;
    }
    
    return result.trim();
  }
  
  // Форматирование значений
  formatValue(value, property) {
    const unitlessProperties = ['opacity', 'zIndex'];
    if (unitlessProperties.includes(property)) {
      return value;
    }
    return `${value}px`;
  }
  
  // Easing функции
  getEasingFunction(name) {
    const easings = {
      linear: t => t,
      easeInQuad: t => t * t,
      easeOutQuad: t => t * (2 - t),
      easeInOutQuad: t => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t,
      easeInCubic: t => t * t * t,
      easeOutCubic: t => (--t) * t * t + 1,
      easeInOutCubic: t => t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1,
      easeInQuart: t => t * t * t * t,
      easeOutQuart: t => 1 - (--t) * t * t * t,
      easeInOutQuart: t => t < 0.5 ? 8 * t * t * t * t : 1 - 8 * (--t) * t * t * t,
      easeInExpo: t => t === 0 ? 0 : Math.pow(2, 10 * (t - 1)),
      easeOutExpo: t => t === 1 ? 1 : 1 - Math.pow(2, -10 * t),
      easeInOutExpo: t => {
        if (t === 0) return 0;
        if (t === 1) return 1;
        if ((t /= 0.5) < 1) return 0.5 * Math.pow(2, 10 * (t - 1));
        return 0.5 * (-Math.pow(2, -10 * --t) + 2);
      }
    };
    
    return easings[name] || easings.linear;
  }
  
  // Получение статистики
  getStats() {
    return {
      activeAnimations: this.animations.size,
      isRunning: this.isRunning,
      globalTime: this.globalTime
    };
  }
}

// Пример использования анимационного движка
class AnimatedScene {
  constructor(container) {
    this.container = container;
    this.engine = new AnimationEngine();
    this.elements = new Map();
    this.init();
  }
  
  init() {
    // Создаем элементы сцены
    this.createElements();
    
    // Настройка оптимизаций
    CSSAnimationOptimization.optimizeTransformAnimations();
  }
  
  createElements() {
    // Создаем анимируемые элементы
    for (let i = 0; i < 5; i++) {
      const element = document.createElement('div');
      element.className = 'animated-element';
      element.style.cssText = `
        position: absolute;
        width: 50px;
        height: 50px;
        background: hsl(${i * 72}, 70%, 50%);
        border-radius: 50%;
        left: 50px;
        top: 50px;
      `;
      
      this.container.appendChild(element);
      this.elements.set(i, element);
    }
  }
  
  // Запуск анимации для элемента
  animateElement(id, targetX, targetY) {
    const element = this.elements.get(id);
    if (!element) return;
    
    const currentLeft = parseInt(element.style.left) || 0;
    const currentTop = parseInt(element.style.top) || 0;
    
    this.engine.add({
      element,
      duration: 1000,
      easing: 'easeOutCubic',
      from: {
        left: currentLeft,
        top: currentTop
      },
      to: {
        left: targetX,
        top: targetY
      },
      onComplete: () => {
        console.log(`Анимация элемента ${id} завершена`);
      }
    });
  }
  
  // Анимация всех элементов
  animateAll() {
    let delay = 0;
    for (let [id, element] of this.elements) {
      setTimeout(() => {
        this.animateElement(
          id,
          100 + Math.random() * 300,
          100 + Math.random() * 300
        );
      }, delay);
      
      delay += 200;
    }
  }
  
  // Получение статистики
  getStats() {
    return this.engine.getStats();
  }
}
```

## Практические рекомендации

1. **Используйте requestAnimationFrame** - для синхронизации с браузером
2. **Применяйте CSS анимации** - когда это возможно для лучшей производительности
3. **Используйте transform и opacity** - вместо других свойств для анимаций
4. **Применяйте will-change** - для предварительной оптимизации
5. **Создавайте compositing layers** - с помощью translateZ(0) или will-change
6. **Избегайте layout thrashing** - в анимационных циклах
7. **Используйте easing функции** - для естественных анимаций
8. **Ограничивайте количество одновременных анимаций** - для лучшей производительности

Оптимизация анимаций - ключ к созданию плавных и отзывчивых интерфейсов. Следование этим принципам поможет создавать визуально привлекательные и производительные анимации.

#javascript #animations #optimization #performance #requestanimationframe #css-animations #transform #easing
---
aliases: [JS Animations, JavaScript Animation, JS Animation, Анимации JS]
tags: [javascript, animation, frontend, web-development]
---

# JavaScript-анимации

JavaScript-анимации предоставляют полный контроль над процессом анимации и позволяют создавать сложные интерактивные эффекты, которые невозможно реализовать только с помощью CSS.

## Основные методы создания анимаций

### requestAnimationFrame
Метод `requestAnimationFrame` оптимизирует анимации под частоту обновления экрана, обеспечивая плавность и эффективность.

```javascript
function animateElement(element, targetPosition) {
  const startPosition = 0;
  const duration = 1000; // 1 секунда
  const startTime = performance.now();

  function updateAnimation(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // Квадратичная функция easing
    const easeProgress = progress < 0.5 
      ? 2 * progress * progress 
      : 1 - Math.pow(-2 * progress + 2, 2) / 2;
    
    const currentPosition = startPosition + (targetPosition - startPosition) * easeProgress;
    element.style.left = currentPosition + 'px';
    
    if (progress < 1) {
      requestAnimationFrame(updateAnimation);
    } else {
      console.log('Анимация завершена');
    }
  }
  
  requestAnimationFrame(updateAnimation);
}

// Использование
const box = document.getElementById('animated-box');
animateElement(box, 300);
```

### setInterval и setTimeout
Классические методы для создания анимаций, менее эффективные по сравнению с `requestAnimationFrame`.

```javascript
function moveElementWithSetInterval(element, targetX, duration) {
  const startX = parseInt(getComputedStyle(element).left) || 0;
  const distance = targetX - startX;
  const startTime = Date.now();
  const interval = 16; // ~60 FPS
  let elapsed = 0;

  const timer = setInterval(() => {
    elapsed = Date.now() - startTime;
    if (elapsed >= duration) {
      element.style.left = targetX + 'px';
      clearInterval(timer);
      return;
    }
    
    const progress = elapsed / duration;
    const currentX = startX + distance * progress;
    element.style.left = currentX + 'px';
  }, interval);
}
```

## Библиотеки для JavaScript-анимаций

### GSAP (GreenSock Animation Platform)
Одна из самых мощных библиотек для анимаций на JavaScript.

```javascript
// Простая анимация
gsap.to("#box", {
  x: 300,
  rotation: 360,
  duration: 2,
  ease: "power2.inOut"
});

// Последовательные анимации
gsap.timeline()
  .to("#box1", { x: 300, duration: 1 })
  .to("#box2", { y: 200, duration: 1 }, "-=0.5") // Начинается за 0.5 сек до окончания предыдущей
  .to("#box3", { rotation: 360, duration: 1 });

// Анимация появления
gsap.from(".element", {
  opacity: 0,
  y: 50,
  duration: 1,
  stagger: 0.2 // Задержка между элементами
});
```

### Anime.js
Легковесная библиотека с интуитивным API.

```javascript
// Анимация одного элемента
anime({
  targets: '#element',
  translateX: 250,
  rotate: 180,
  backgroundColor: '#FFF',
  duration: 2000,
  easing: 'easeInOutExpo'
});

// Анимация массива элементов
anime({
  targets: ['.item1', '.item2', '.item3'],
  translateX: 100,
  delay: anime.stagger(200), // Задержка 200ms между анимациями
  direction: 'alternate', // Анимация туда-обратно
  loop: true
});
```

### Framer Motion (для React)
Хотя это библиотека для React, она основана на JavaScript-анимациях.

```javascript
// Пример использования в React-компоненте (для понимания)
const MotionBox = () => (
  <motion.div
    initial={{ opacity: 0, x: -100 }}
    animate={{ opacity: 1, x: 0 }}
    transition={{ duration: 0.5 }}
  >
    Animated Content
  </motion.div>
);
```

## Практические примеры

### Анимация прогресса
Создание индикатора прогресса с анимацией.

```javascript
class ProgressBar {
  constructor(element, maxValue = 100) {
    this.element = element;
    this.maxValue = maxValue;
    this.currentValue = 0;
  }
  
  animateTo(value, duration = 1000) {
    const startValue = this.currentValue;
    const startTime = performance.now();
    
    const updateProgress = (currentTime) => {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);
      
      // Easing function for smooth animation
      const easeProgress = 1 - Math.pow(1 - progress, 3);
      this.currentValue = startValue + (value - startValue) * easeProgress;
      
      this.element.style.width = `${(this.currentValue / this.maxValue) * 100}%`;
      
      if (progress < 1) {
        requestAnimationFrame(updateProgress);
      }
    };
    
    requestAnimationFrame(updateProgress);
  }
}

// Использование
const progressBar = new ProgressBar(document.getElementById('progress-bar'));
progressBar.animateTo(75, 2000); // Анимировать до 75% за 2 секунды
```

### Анимация скролла
Создание плавного скролла с кастомной анимацией.

```javascript
function smoothScrollTo(targetY, duration = 1000) {
  const startingY = window.pageYOffset;
  const diff = targetY - startingY;
  const startTime = performance.now();
  
  function step(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // Ease-in-out cubic
    const easeProgress = progress < 0.5 
      ? 4 * progress * progress * progress 
      : (progress - 1) * (2 * progress - 2) * (2 * progress - 2) + 1;
    
    window.scrollTo(0, startingY + diff * easeProgress);
    
    if (progress < 1) {
      requestAnimationFrame(step);
    }
  }
  
  requestAnimationFrame(step);
}

// Использование
document.getElementById('scroll-button').addEventListener('click', () => {
  smoothScrollTo(1000); // Прокрутить до позиции 1000px
});
```

## Сравнение с CSS-анимациями

JavaScript-анимации предлагают больше возможностей по сравнению с CSS:

- **Динамический контроль**: значения анимации могут изменяться во время выполнения
- **Сложная логика**: можно создавать условные анимации, реактивные анимации
- **Синхронизация с событиями**: анимации могут запускаться по различным событиям
- **Интерактивность**: анимации могут реагировать на пользовательский ввод

```javascript
// Пример интерактивной анимации
let isPlaying = false;
let currentFrame = 0;
const frames = [0, 10, 20, 30, 40, 50]; // Позиции анимации

function interactiveAnimation() {
  if (!isPlaying) return;
  
  element.style.left = frames[currentFrame] + 'px';
  currentFrame = (currentFrame + 1) % frames.length;
  
  if (currentFrame !== 0) {
    requestAnimationFrame(interactiveAnimation);
  }
}

// Управление анимацией
document.addEventListener('keydown', (e) => {
  if (e.key === ' ') { // Пробел
    isPlaying = !isPlaying;
    if (isPlaying) {
      interactiveAnimation();
    }
  }
});
```

## Производительность и оптимизация

### Использование transform и opacity
Для оптимизации производительности используйте свойства, которые не вызывают перерисовку.

```javascript
// Хорошо - использует transform
function optimizedMove(element, x, y) {
  element.style.transform = `translate(${x}px, ${y}px)`;
}

// Плохо - вызывает перерисовку
function unoptimizedMove(element, x, y) {
  element.style.left = x + 'px';
  element.style.top = y + 'px';
}
```

### Буферизация и предварительная загрузка
Для сложных анимаций можно использовать предварительную загрузку и буферизацию.

```javascript
class AnimationBuffer {
  constructor(frames, fps = 60) {
    this.frames = frames;
    this.fps = fps;
    this.frameDuration = 1000 / fps;
    this.precomputed = this.precomputeFrames();
  }
  
  precomputeFrames() {
    // Предварительно вычисляем все кадры анимации
    return this.frames.map((frame, index) => ({
      ...frame,
      timestamp: index * this.frameDuration
    }));
  }
  
  play(element) {
    let frameIndex = 0;
    const startTime = performance.now();
    
    const renderFrame = () => {
      if (frameIndex < this.precomputed.length) {
        const frame = this.precomputed[frameIndex];
        // Применяем свойства к элементу
        Object.keys(frame).forEach(prop => {
          if (prop !== 'timestamp') {
            element.style[prop] = frame[prop];
          }
        });
        
        frameIndex++;
        setTimeout(renderFrame, this.frameDuration);
      }
    };
    
    renderFrame();
  }
}
```

> [!tip]
> Используйте `requestAnimationFrame` для всех анимаций, чтобы обеспечить синхронизацию с частотой обновления экрана и оптимальную производительность.

## Заключение

JavaScript-анимации предоставляют разработчикам мощный инструмент для создания сложных, интерактивных и динамических анимационных эффектов. Они особенно полезны в ситуациях, когда требуется:

- Сложная логика анимации
- Реакция на пользовательский ввод
- Синхронизация с событиями
- Динамическое изменение параметров анимации

Для простых эффектов часто предпочтительнее использовать [[CSS-анимации]], так как они проще в реализации и имеют лучшую производительность для базовых задач.

## См. также
- [[CSS-анимации]]
- [[Анимации-в-React]]
- [[Анимации-в-Vue]]
- [[Анимации-в-Svelte]]
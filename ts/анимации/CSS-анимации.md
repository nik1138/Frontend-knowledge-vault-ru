---
aliases: [CSS-анимации, Анимации CSS, CSS Animations]
tags: [typescript, css, анимации, веб-разработка]
---

# CSS-анимации в TypeScript проектах

CSS-анимации являются мощным инструментом для создания плавных и привлекательных визуальных эффектов в веб-приложениях. При использовании TypeScript, можно повысить типизацию и безопасность при работе с CSS-анимациями, особенно при взаимодействии с DOM-элементами.

## Основы CSS-анимаций

CSS-анимации позволяют плавно изменять значения CSS-свойств элемента за определенный промежуток времени без использования JavaScript. Основные свойства:

- `@keyframes` — определяет анимацию
- `animation` — применяет анимацию к элементу
- `transition` — обеспечивает плавный переход между состояниями

```css
@keyframes slideIn {
  from {
    transform: translateX(-100%);
  }
  to {
    transform: translateX(0);
  }
}

.animated-element {
  animation: slideIn 0.5s ease-in-out;
}
```

## Использование TypeScript для типизации CSS-анимаций

TypeScript позволяет создавать строго типизированные интерфейсы для работы с CSS-анимациями. Это особенно полезно при работе с атрибутами `style` и `animation`.

```typescript
interface CSSAnimationProperties {
  animation: string;
  animationName: string;
  animationDuration: string;
  animationTimingFunction: string;
  animationDelay: string;
  animationIterationCount: string | number;
  animationDirection: string;
  animationFillMode: string;
  animationPlayState: string;
}

// Пример использования
const element = document.querySelector('.animated-element') as HTMLElement;

if (element) {
  element.style.animation = 'slideIn 0.5s ease-in-out';
  element.style.animationPlayState = 'running';
}
```

## Анимации с CSS-классами

Использование CSS-классов для управления анимациями позволяет избежать прямой манипуляции со стилями и делает код более поддерживаемым.

```typescript
class AnimationController {
  private element: HTMLElement;

  constructor(element: HTMLElement) {
    this.element = element;
  }

  public animateIn(): void {
    this.element.classList.add('animate-in');
    this.element.classList.remove('animate-out');
  }

  public animateOut(): void {
    this.element.classList.add('animate-out');
    this.element.classList.remove('animate-in');
  }
}

// Использование
const animatedDiv = document.getElementById('myAnimatedDiv');
if (animatedDiv) {
  const controller = new AnimationController(animatedDiv);
  controller.animateIn();
}
```

```css
.animate-in {
  animation: fadeIn 0.3s ease-out forwards;
}

.animate-out {
  animation: fadeOut 0.3s ease-out forwards;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to { opacity: 0; }
}
```

## Анимации с TypeScript и React

При работе с React и TypeScript, можно использовать хуки и компоненты для управления анимациями.

```typescript
import React, { useState, useEffect } from 'react';

interface AnimatedComponentProps {
  children: React.ReactNode;
  show: boolean;
}

const AnimatedComponent: React.FC<AnimatedComponentProps> = ({ children, show }) => {
  const [shouldRender, setShouldRender] = useState(show);

  useEffect(() => {
    if (show) {
      setShouldRender(true);
    } else {
      const timer = setTimeout(() => {
        setShouldRender(false);
      }, 300); // Длительность анимации
      return () => clearTimeout(timer);
    }
  }, [show]);

  if (!shouldRender) return null;

  return (
    <div className={`animated-component ${show ? 'show' : 'hide'}`}>
      {children}
    </div>
  );
};
```

```css
.animated-component {
  opacity: 0;
  transform: translateY(-20px);
  transition: opacity 0.3s ease-out, transform 0.3s ease-out;
}

.animated-component.show {
  opacity: 1;
  transform: translateY(0);
}

.animated-component.hide {
  opacity: 0;
  transform: translateY(-20px);
}
```

## Типизация CSS-свойств анимаций

Для обеспечения безопасности типов при работе с CSS-анимациями, можно создать специальные типы:

```typescript
type AnimationTimingFunction = 
  | 'ease' 
  | 'linear' 
  | 'ease-in' 
  | 'ease-out' 
  | 'ease-in-out'
  | `cubic-bezier(${number}, ${number}, ${number}, ${number})`;

type AnimationDirection = 
  | 'normal' 
  | 'reverse' 
  | 'alternate' 
  | 'alternate-reverse';

type AnimationFillMode = 
  | 'none' 
  | 'forwards' 
  | 'backwards' 
  | 'both';

interface AnimationConfig {
  name: string;
  duration: string;
  timingFunction: AnimationTimingFunction;
  delay: string;
  iterationCount: number | 'infinite';
  direction: AnimationDirection;
  fillMode: AnimationFillMode;
  playState: 'running' | 'paused';
}
```

## Работа с событиями анимаций

TypeScript позволяет типизировать события, связанные с анимациями:

```typescript
class AnimationEventController {
  private element: HTMLElement;

  constructor(element: HTMLElement) {
    this.element = element;
  }

  public onTransitionEnd(callback: (event: TransitionEvent) => void): void {
    this.element.addEventListener('transitionend', callback);
  }

  public onAnimationEnd(callback: (event: AnimationEvent) => void): void {
    this.element.addEventListener('animationend', callback);
  }

  public onAnimationStart(callback: (event: AnimationEvent) => void): void {
    this.element.addEventListener('animationstart', callback);
  }
}
```

## Лучшие практики

1. **Использование CSS-переменных для анимаций**:
   ```css
   :root {
     --animation-duration: 0.3s;
     --animation-timing: ease-in-out;
   }

   .animated-element {
     animation-duration: var(--animation-duration);
     animation-timing-function: var(--animation-timing);
   }
   ```

2. **Типизация CSS-классов анимаций**:
   ```typescript
   type AnimationClass = 
     | 'fade-in'
     | 'slide-up'
     | 'bounce'
     | 'scale-up'
     | 'rotate';

   function applyAnimation(element: HTMLElement, animationClass: AnimationClass): void {
     element.classList.add(animationClass);
   }
   ```

3. **Оптимизация производительности**:
   - Используйте `transform` и `opacity` для анимаций
   - Избегайте анимации свойств, вызывающих перерисовку

## Заключение

CSS-анимации в сочетании с TypeScript обеспечивают мощный инструмент для создания интерактивных и визуально привлекательных веб-приложений с высокой степенью типизации и безопасности.

## См. также

- [[JavaScript-анимации]]
- [[GSAP]]
- [[Framer-Motion]]
- [[WebGL-анимации]]
- [[React-анимации]]
- [[CSS-переходы]]
- [[CSS-трансформации]]
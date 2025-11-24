---
aliases: [JavaScript-анимации, Анимации JavaScript, JS Animations]
tags: [typescript, javascript, анимации, веб-разработка]
---

# JavaScript-анимации в TypeScript проектах

JavaScript-анимации позволяют создавать сложные и интерактивные анимационные эффекты, которые невозможно реализовать только с помощью CSS. TypeScript добавляет строгую типизацию к этим анимациям, обеспечивая более надежный и предсказуемый код.

## Основы JavaScript-анимаций

JavaScript-анимации строятся на основе изменения свойств DOM-элементов с течением времени. Наиболее распространенные подходы:

- `requestAnimationFrame` — для плавных анимаций
- Манипуляции с CSS-свойствами через JavaScript
- Интерполяция значений для плавного перехода

## requestAnimationFrame с TypeScript

Метод `requestAnimationFrame` является основой для создания плавных анимаций в JavaScript:

```typescript
interface AnimationFrameCallback {
  (time: number): void;
}

class SmoothAnimation {
  private startTime: number | null = null;
  private duration: number;
  private element: HTMLElement;
  private animationFrameId: number | null = null;

  constructor(element: HTMLElement, duration: number = 1000) {
    this.element = element;
    this.duration = duration;
  }

  public start(): void {
    this.startTime = null;
    this.animationFrameId = requestAnimationFrame((time) => this.animate(time));
  }

  private animate(currentTime: number): void {
    if (!this.startTime) {
      this.startTime = currentTime;
    }

    const elapsed = currentTime - this.startTime;
    const progress = Math.min(elapsed / this.duration, 1);

    // Пример анимации перемещения
    const startPosition = 0;
    const endPosition = 200;
    const currentPosition = startPosition + (endPosition - startPosition) * progress;

    this.element.style.transform = `translateX(${currentPosition}px)`;

    if (progress < 1) {
      this.animationFrameId = requestAnimationFrame((time) => this.animate(time));
    } else {
      this.onComplete();
    }
  }

  private onComplete(): void {
    console.log('Анимация завершена');
  }

  public stop(): void {
    if (this.animationFrameId !== null) {
      cancelAnimationFrame(this.animationFrameId);
      this.animationFrameId = null;
    }
  }
}

// Использование
const element = document.getElementById('animated-box');
if (element) {
  const animation = new SmoothAnimation(element, 2000); // 2 секунды
  animation.start();
}
```

## Интерполяция значений

Интерполяция позволяет плавно изменять значения от начального до конечного:

```typescript
namespace Interpolation {
  export function linear(start: number, end: number, progress: number): number {
    return start + (end - start) * progress;
  }

  export function easeIn(start: number, end: number, progress: number): number {
    return start + (end - start) * Math.pow(progress, 2);
  }

  export function easeOut(start: number, end: number, progress: number): number {
    return start + (end - start) * (1 - Math.pow(1 - progress, 2));
  }

  export function easeInOut(start: number, end: number, progress: number): number {
    return start + (end - start) * (
      progress < 0.5 
        ? 2 * progress * progress 
        : -1 + (4 - 2 * progress) * progress
    );
  }
}

// Использование интерполяции
function animateProperty(
  element: HTMLElement,
  property: string,
  startValue: number,
  endValue: number,
  duration: number,
  easing: (start: number, end: number, progress: number) => number = Interpolation.linear
): Promise<void> {
  return new Promise((resolve) => {
    const startTime = performance.now();

    function updateProperty(currentTime: number) {
      const elapsed = currentTime - startTime;
      const progress = Math.min(elapsed / duration, 1);

      const currentValue = easing(startValue, endValue, progress);
      element.style.setProperty(property, `${currentValue}px`);

      if (progress < 1) {
        requestAnimationFrame(updateProperty);
      } else {
        resolve();
      }
    }

    requestAnimationFrame(updateProperty);
  });
}
```

## Класс для управления анимациями

Создадим более сложный класс для управления несколькими анимациями:

```typescript
interface AnimationOptions {
  duration: number;
  easing?: (start: number, end: number, progress: number) => number;
  delay?: number;
  onProgress?: (progress: number) => void;
  onComplete?: () => void;
}

interface PropertyAnimation {
  property: string;
  startValue: number;
  endValue: number;
}

class AnimationManager {
  private element: HTMLElement;
  private animations: PropertyAnimation[];
  private options: Required<AnimationOptions>;
  private startTime: number | null = null;
  private animationFrameId: number | null = null;

  constructor(element: HTMLElement, properties: PropertyAnimation[], options: AnimationOptions) {
    this.element = element;
    this.animations = properties;
    
    // Установка значений по умолчанию
    this.options = {
      duration: options.duration,
      easing: options.easing || Interpolation.linear,
      delay: options.delay || 0,
      onProgress: options.onProgress || (() => {}),
      onComplete: options.onComplete || (() => {})
    };
  }

  public start(): void {
    this.startTime = null;
    this.animationFrameId = requestAnimationFrame((time) => this.runAnimation(time));
  }

  private runAnimation(currentTime: number): void {
    if (!this.startTime) {
      this.startTime = currentTime + this.options.delay;
    }

    const elapsed = currentTime - this.startTime;
    const progress = Math.min(elapsed / this.options.duration, 1);

    // Обновление значений свойств
    this.animations.forEach(anim => {
      const currentValue = this.options.easing(anim.startValue, anim.endValue, progress);
      this.element.style.setProperty(anim.property, `${currentValue}px`);
    });

    // Вызов коллбэка прогресса
    this.options.onProgress(progress);

    if (progress < 1) {
      this.animationFrameId = requestAnimationFrame((time) => this.runAnimation(time));
    } else {
      this.options.onComplete();
    }
  }

  public stop(): void {
    if (this.animationFrameId !== null) {
      cancelAnimationFrame(this.animationFrameId);
      this.animationFrameId = null;
    }
  }
}

// Пример использования
const elementToAnimate = document.getElementById('multi-anim');
if (elementToAnimate) {
  const animation = new AnimationManager(
    elementToAnimate,
    [
      { property: 'left', startValue: 0, endValue: 300 },
      { property: 'top', startValue: 0, endValue: 200 }
    ],
    {
      duration: 2000,
      easing: Interpolation.easeInOut,
      delay: 500,
      onProgress: (progress) => console.log(`Прогресс: ${Math.round(progress * 100)}%`),
      onComplete: () => console.log('Анимация завершена!')
    }
  );
  animation.start();
}
```

## Анимации с использованием CSS-свойств

TypeScript позволяет безопасно манипулировать CSS-свойствами:

```typescript
type CSSProperty = 
  | 'width' 
  | 'height' 
  | 'left' 
  | 'top' 
  | 'right' 
  | 'bottom' 
  | 'opacity' 
  | 'transform' 
  | 'background-color';

interface CSSAnimationConfig {
  property: CSSProperty;
  from: string | number;
  to: string | number;
  unit?: string;
}

class CSSPropertyAnimator {
  private element: HTMLElement;
  private config: CSSAnimationConfig;
  private duration: number;
  private startTime: number | null = null;

  constructor(element: HTMLElement, config: CSSAnimationConfig, duration: number = 1000) {
    this.element = element;
    this.config = config;
    this.duration = duration;
  }

  public animate(): Promise<void> {
    return new Promise((resolve) => {
      this.startTime = performance.now();

      const animateFrame = (currentTime: number) => {
        const elapsed = currentTime - this.startTime!;
        const progress = Math.min(elapsed / this.duration, 1);

        let value: string | number;

        if (typeof this.config.from === 'number' && typeof this.config.to === 'number') {
          const interpolatedValue = Interpolation.linear(
            this.config.from,
            this.config.to,
            progress
          );
          
          value = `${interpolatedValue}${this.config.unit || ''}`;
        } else {
          value = progress < 0.5 ? this.config.from : this.config.to;
        }

        this.element.style.setProperty(this.config.property, value.toString());

        if (progress < 1) {
          requestAnimationFrame(animateFrame);
        } else {
          resolve();
        }
      };

      requestAnimationFrame(animateFrame);
    });
  }
}
```

## Анимации с событиями

Работа с событиями позволяет создавать более интерактивные анимации:

```typescript
class InteractiveAnimation {
  private element: HTMLElement;
  private originalPosition: { x: number; y: number };
  private isAnimating: boolean = false;

  constructor(element: HTMLElement) {
    this.element = element;
    this.originalPosition = { x: 0, y: 0 };
    this.setupEventListeners();
  }

  private setupEventListeners(): void {
    this.element.addEventListener('mouseenter', () => this.startHoverAnimation());
    this.element.addEventListener('mouseleave', () => this.resetAnimation());
    this.element.addEventListener('click', () => this.startClickAnimation());
  }

  private startHoverAnimation(): void {
    if (this.isAnimating) return;
    
    this.isAnimating = true;
    const scaleAnimation = new AnimationManager(
      this.element,
      [{ property: 'transform', startValue: 1, endValue: 1.1 }],
      {
        duration: 300,
        easing: Interpolation.easeOut,
        onComplete: () => { this.isAnimating = false; }
      }
    );
    scaleAnimation.start();
  }

  private startClickAnimation(): void {
    if (this.isAnimating) return;
    
    this.isAnimating = true;
    const bounceAnimation = new AnimationManager(
      this.element,
      [
        { property: 'left', startValue: 0, endValue: 20 },
        { property: 'top', startValue: 0, endValue: -10 }
      ],
      {
        duration: 500,
        easing: Interpolation.easeInOut,
        onComplete: () => {
          // Возврат к исходной позиции
          setTimeout(() => {
            this.resetAnimation();
            this.isAnimating = false;
          }, 100);
        }
      }
    );
    bounceAnimation.start();
  }

  private resetAnimation(): void {
    this.element.style.transform = 'scale(1)';
    this.element.style.left = '0px';
    this.element.style.top = '0px';
  }
}
```

## Анимации с Canvas

TypeScript позволяет создавать сложные анимации с использованием Canvas API:

```typescript
class CanvasAnimation {
  private canvas: HTMLCanvasElement;
  private ctx: CanvasRenderingContext2D;
  private animationFrameId: number | null = null;
  private objects: CanvasObject[] = [];

  constructor(canvasId: string) {
    const canvas = document.getElementById(canvasId) as HTMLCanvasElement;
    if (!canvas) {
      throw new Error(`Canvas с id ${canvasId} не найден`);
    }
    
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d')!;
  }

  public addObject(obj: CanvasObject): void {
    this.objects.push(obj);
  }

  public start(): void {
    this.animationFrameId = requestAnimationFrame(() => this.draw());
  }

  private draw(): void {
    // Очистка холста
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

    // Обновление и рисование объектов
    this.objects.forEach(obj => {
      obj.update();
      obj.draw(this.ctx);
    });

    this.animationFrameId = requestAnimationFrame(() => this.draw());
  }

  public stop(): void {
    if (this.animationFrameId !== null) {
      cancelAnimationFrame(this.animationFrameId);
      this.animationFrameId = null;
    }
  }
}

interface CanvasObject {
  x: number;
  y: number;
  width: number;
  height: number;
  color: string;
  update(): void;
  draw(ctx: CanvasRenderingContext2D): void;
}

class MovingCircle implements CanvasObject {
  public x: number;
  public y: number;
  public width: number;
  public height: number;
  public color: string;
  private velocityX: number;
  private velocityY: number;

  constructor(x: number, y: number, radius: number, color: string) {
    this.x = x;
    this.y = y;
    this.width = radius * 2;
    this.height = radius * 2;
    this.color = color;
    this.velocityX = (Math.random() - 0.5) * 4;
    this.velocityY = (Math.random() - 0.5) * 4;
  }

  public update(): void {
    this.x += this.velocityX;
    this.y += this.velocityY;

    // Отскок от границ
    if (this.x <= 0 || this.x + this.width >= 800) {
      this.velocityX *= -1;
    }
    if (this.y <= 0 || this.y + this.height >= 600) {
      this.velocityY *= -1;
    }
  }

  public draw(ctx: CanvasRenderingContext2D): void {
    ctx.beginPath();
    ctx.arc(this.x + this.width/2, this.y + this.height/2, this.width/2, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.fill();
  }
}
```

## Заключение

JavaScript-анимации в TypeScript проектах обеспечивают высокую степень контроля над анимационными эффектами. Использование строгой типизации помогает избежать ошибок и делает код более понятным и поддерживаемым.

## См. также

- [[CSS-анимации]]
- [[GSAP]]
- [[Framer-Motion]]
- [[WebGL-анимации]]
- [[Canvas-анимации]]
- [[requestAnimationFrame]]
- [[TypeScript DOM манипуляции]]
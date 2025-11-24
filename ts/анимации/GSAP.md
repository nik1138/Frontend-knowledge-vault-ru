---
aliases: [GSAP, GreenSock, GreenSock Animation Platform]
tags: [typescript, javascript, анимации, библиотека, веб-разработка]
---

# GSAP (GreenSock Animation Platform) в TypeScript проектах

GSAP (GreenSock Animation Platform) — это мощная и гибкая библиотека анимаций для JavaScript, которая обеспечивает высокую производительность и широкие возможности для создания сложных анимационных эффектов. TypeScript позволяет использовать строгую типизацию при работе с GSAP, что повышает надежность и читаемость кода.

## Установка и настройка

Для использования GSAP в TypeScript проекте необходимо установить соответствующие пакеты:

```bash
npm install gsap
npm install @types/gsap --save-dev  # если доступны типы
```

Или импортировать через CDN:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
```

## Основы GSAP

GSAP предоставляет несколько основных инструментов:

- `gsap` — для создания отдельных анимаций
- `gsap.timeline()` — для создания последовательностей анимаций
- `gsap.to()`, `gsap.from()`, `gsap.fromTo()` — для различных типов анимаций

### Простая анимация

```typescript
import gsap from 'gsap';

// Анимация элемента к новому состоянию
const element = document.querySelector('.my-element') as HTMLElement;

if (element) {
  gsap.to(element, {
    x: 100,
    y: 50,
    rotation: 360,
    duration: 2,
    ease: 'power2.inOut',
    onComplete: () => {
      console.log('Анимация завершена!');
    }
  });
}
```

### Анимация с начальным состоянием

```typescript
// Анимация из начального состояния
gsap.from(element, {
  opacity: 0,
  scale: 0.5,
  duration: 1.5,
  ease: 'back.out(1.7)'
});
```

### Анимация с явно заданными начальным и конечным состояниями

```typescript
// Анимация с явно заданными начальным и конечным состояниями
gsap.fromTo(
  element,
  { 
    x: 0,
    opacity: 0 
  },
  {
    x: 300,
    opacity: 1,
    duration: 2,
    ease: 'elastic.out(1, 0.3)',
    onStart: () => console.log('Анимация началась'),
    onComplete: () => console.log('Анимация завершена')
  }
);
```

## Использование Timeline

Timeline позволяет создавать сложные последовательности анимаций:

```typescript
import { gsap } from 'gsap';

// Создание таймлайна
const tl = gsap.timeline({
  defaults: { 
    duration: 1,
    ease: 'power2.out'
  }
});

// Добавление анимаций в таймлайн
tl
  .to('.element1', { x: 100, opacity: 1 })
  .to('.element2', { y: 50, opacity: 1 }, '-=0.5') // начать на 0.5 сек раньше
  .to('.element3', { rotation: 360, scale: 1.2 }, '>-0.3') // начать на 0.3 сек позже предыдущей
  .add(() => console.log('Середина таймлайна'), '-=1') // добавить вызов функции
  .to('.element4', { backgroundColor: '#ff0000' });
```

## Типизация GSAP анимаций

TypeScript позволяет создавать типизированные конфигурации анимаций:

```typescript
import { gsap, TweenMax, TimelineMax } from 'gsap';

// Тип для свойств анимации
interface AnimationProperties {
  x?: number | string;
  y?: number | string;
  rotation?: number;
  scale?: number;
  opacity?: number;
  backgroundColor?: string;
  width?: number | string;
  height?: number | string;
}

// Тип для конфигурации анимации
interface GSAPAnimationConfig {
  target: gsap.TweenTarget;
  properties: AnimationProperties;
  duration: number;
  delay?: number;
  ease?: string;
  onStart?: () => void;
  onComplete?: () => void;
  onProgress?: (progress: number) => void;
}

// Функция для создания анимации
function createGSAPAnimation(config: GSAPAnimationConfig): gsap.core.Tween {
  return gsap.to(config.target, {
    ...config.properties,
    duration: config.duration,
    delay: config.delay || 0,
    ease: config.ease || 'power2.out',
    onStart: config.onStart,
    onComplete: config.onComplete,
    onUpdate: () => {
      if (config.onProgress) {
        // GSAP не предоставляет прогресс напрямую в onUpdate, 
        // но мы можем рассчитать его при необходимости
        config.onProgress(0); // Заглушка - в реальном коде потребуется вычисление прогресса
      }
    }
  });
}

// Использование типизированной функции
const element = document.querySelector('.animated-element');
if (element) {
  createGSAPAnimation({
    target: element,
    properties: {
      x: 200,
      rotation: 180,
      opacity: 0.8
    },
    duration: 2,
    ease: 'elastic.out(1, 0.3)',
    onStart: () => console.log('Анимация началась'),
    onComplete: () => console.log('Анимация завершена')
  });
}
```

## Анимации с плагинами GSAP

GSAP имеет множество плагинов для расширенных возможностей:

```typescript
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import { Draggable } from 'gsap/Draggable';

// Регистрация плагинов
gsap.registerPlugin(ScrollTrigger, Draggable);

// Анимация при прокрутке
gsap.to('.scroll-element', {
  scrollTrigger: {
    trigger: '.trigger-element',
    start: 'top center',
    end: 'bottom center',
    scrub: true,
    markers: true
  },
  x: 300,
  rotation: 360
});

// Драг-н-дроп элемент
Draggable.create('.draggable-element', {
  type: 'x,y',
  bounds: '.container',
  onDrag: function() {
    console.log('Перетаскивание', this.x, this.y);
  }
});
```

## Класс для управления GSAP анимациями

Создадим класс для централизованного управления анимациями:

```typescript
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

type AnimationType = 'to' | 'from' | 'fromTo';
type EaseType = 
  | 'power1.out' | 'power1.in' | 'power1.inOut'
  | 'power2.out' | 'power2.in' | 'power2.inOut'
  | 'power3.out' | 'power3.in' | 'power3.inOut'
  | 'power4.out' | 'power4.in' | 'power4.inOut'
  | 'elastic.out' | 'elastic.in' | 'elastic.inOut'
  | 'back.out' | 'back.in' | 'back.inOut'
  | 'bounce.out' | 'bounce.in' | 'bounce.inOut';

interface BaseAnimationConfig {
  duration: number;
  delay?: number;
  ease?: EaseType;
  stagger?: number;
  onStart?: () => void;
  onComplete?: () => void;
  onProgress?: (progress: number) => void;
}

interface ToAnimationConfig extends BaseAnimationConfig {
  target: gsap.TweenTarget;
  properties: gsap.TweenVars;
}

interface FromAnimationConfig extends BaseAnimationConfig {
  target: gsap.TweenTarget;
  from: gsap.TweenVars;
}

interface FromToAnimationConfig extends BaseAnimationConfig {
  target: gsap.TweenTarget;
  from: gsap.TweenVars;
  to: gsap.TweenVars;
}

interface ScrollTriggerConfig {
  trigger: string;
  start?: string;
  end?: string;
  scrub?: boolean | number;
  pin?: boolean;
  markers?: boolean;
}

class GSAPAnimationManager {
  private animations: gsap.core.Tween[] = [];
  private timelines: gsap.core.Timeline[] = [];

  /**
   * Создание анимации "к"
   */
  public animateTo(config: ToAnimationConfig): gsap.core.Tween {
    const animation = gsap.to(config.target, {
      ...config.properties,
      duration: config.duration,
      delay: config.delay || 0,
      ease: config.ease || 'power2.out',
      onStart: config.onStart,
      onComplete: config.onComplete
    });
    
    this.animations.push(animation);
    return animation;
  }

  /**
   * Создание анимации "из"
   */
  public animateFrom(config: FromAnimationConfig): gsap.core.Tween {
    const animation = gsap.from(config.target, {
      ...config.from,
      duration: config.duration,
      delay: config.delay || 0,
      ease: config.ease || 'power2.out',
      onStart: config.onStart,
      onComplete: config.onComplete
    });
    
    this.animations.push(animation);
    return animation;
  }

  /**
   * Создание анимации "из-в"
   */
  public animateFromTo(config: FromToAnimationConfig): gsap.core.Tween {
    const animation = gsap.fromTo(
      config.target,
      config.from,
      {
        ...config.to,
        duration: config.duration,
        delay: config.delay || 0,
        ease: config.ease || 'power2.out',
        onStart: config.onStart,
        onComplete: config.onComplete
      }
    );
    
    this.animations.push(animation);
    return animation;
  }

  /**
   * Создание анимации с ScrollTrigger
   */
  public animateWithScrollTrigger(
    config: ToAnimationConfig & { scrollTrigger: ScrollTriggerConfig }
  ): gsap.core.Tween {
    const animation = gsap.to(config.target, {
      ...config.properties,
      duration: config.duration,
      delay: config.delay || 0,
      ease: config.ease || 'power2.out',
      scrollTrigger: {
        trigger: config.scrollTrigger.trigger,
        start: config.scrollTrigger.start || 'top center',
        end: config.scrollTrigger.end || 'bottom center',
        scrub: config.scrollTrigger.scrub || true,
        pin: config.scrollTrigger.pin,
        markers: config.scrollTrigger.markers
      },
      onStart: config.onStart,
      onComplete: config.onComplete
    });
    
    this.animations.push(animation);
    return animation;
  }

  /**
   * Создание таймлайна
   */
  public createTimeline(): gsap.core.Timeline {
    const timeline = gsap.timeline();
    this.timelines.push(timeline);
    return timeline;
  }

  /**
   * Пауза всех анимаций
   */
  public pauseAll(): void {
    this.animations.forEach(anim => anim.pause());
    this.timelines.forEach(tl => tl.pause());
  }

  /**
   * Возобновление всех анимаций
   */
  public resumeAll(): void {
    this.animations.forEach(anim => anim.play());
    this.timelines.forEach(tl => tl.play());
  }

  /**
   * Остановка всех анимаций
   */
  public killAll(): void {
    this.animations.forEach(anim => anim.kill());
    this.timelines.forEach(tl => tl.kill());
    this.animations = [];
    this.timelines = [];
  }
}

// Пример использования
const animationManager = new GSAPAnimationManager();

const element = document.querySelector('.animated-box');
if (element) {
  // Простая анимация
  animationManager.animateTo({
    target: element,
    properties: {
      x: 200,
      y: 100,
      rotation: 180,
      opacity: 0.8
    },
    duration: 2,
    ease: 'elastic.out(1, 0.3)',
    onComplete: () => console.log('Анимация завершена')
  });

  // Анимация с прокруткой
  animationManager.animateWithScrollTrigger({
    target: element,
    properties: {
      scale: 1.5,
      rotation: 360
    },
    duration: 1,
    scrollTrigger: {
      trigger: '.scroll-trigger',
      start: 'top center',
      end: 'bottom center',
      scrub: true
    }
  });
}
```

## GSAP и React с TypeScript

При работе с React, можно использовать хуки для управления GSAP анимациями:

```typescript
import React, { useEffect, useRef } from 'react';
import gsap from 'gsap';

interface AnimatedComponentProps {
  children: React.ReactNode;
  animateIn?: boolean;
  duration?: number;
}

const AnimatedComponent: React.FC<AnimatedComponentProps> = ({
  children,
  animateIn = false,
  duration = 1
}) => {
  const elementRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (elementRef.current && animateIn) {
      gsap.fromTo(
        elementRef.current,
        {
          opacity: 0,
          y: 50
        },
        {
          opacity: 1,
          y: 0,
          duration: duration,
          ease: 'power2.out'
        }
      );
    }
  }, [animateIn, duration]);

  return (
    <div ref={elementRef}>
      {children}
    </div>
  );
};

// Использование компонента
const App: React.FC = () => {
  const [isVisible, setIsVisible] = React.useState(false);

  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        Переключить анимацию
      </button>
      <AnimatedComponent animateIn={isVisible}>
        <div>Анимированный контент</div>
      </AnimatedComponent>
    </div>
  );
};
```

## Практические примеры

### Анимация списка элементов

```typescript
function animateListItems(selector: string, stagger: number = 0.1): void {
  const items = gsap.utils.toArray(selector);
  
  gsap.fromTo(
    items,
    {
      opacity: 0,
      y: 30
    },
    {
      opacity: 1,
      y: 0,
      duration: 0.8,
      stagger: stagger,
      ease: 'power2.out'
    }
  );
}

// Использование
animateListItems('.list-item', 0.2);
```

### Анимация загрузки

```typescript
class LoadingAnimation {
  private progressBar: HTMLElement | null;
  private loader: HTMLElement | null;
  private animation: gsap.core.Tween | null = null;

  constructor(progressBarId: string, loaderId: string) {
    this.progressBar = document.getElementById(progressBarId);
    this.loader = document.getElementById(loaderId);
  }

  public start(): void {
    if (!this.progressBar) return;

    this.animation = gsap.to(this.progressBar, {
      width: '100%',
      duration: 3,
      ease: 'linear',
      onUpdate: () => {
        const width = this.progressBar?.style.width || '0%';
        console.log(`Загрузка: ${width}`);
      },
      onComplete: () => {
        console.log('Загрузка завершена!');
        if (this.loader) {
          gsap.to(this.loader, {
            opacity: 0,
            duration: 0.5,
            onComplete: () => {
              if (this.loader) this.loader.style.display = 'none';
            }
          });
        }
      }
    });
  }

  public stop(): void {
    if (this.animation) {
      this.animation.kill();
    }
  }
}
```

## Заключение

GSAP — мощный инструмент для создания сложных анимаций в веб-приложениях. При использовании с TypeScript, он обеспечивает высокую степень типизации и безопасности, что делает код более надежным и поддерживаемым.

## См. также

- [[CSS-анимации]]
- [[JavaScript-анимации]]
- [[Framer-Motion]]
- [[WebGL-анимации]]
- [[React-анимации]]
- [[Canvas-анимации]]
- [[ScrollTrigger]]
- [[GSAP плагины]]
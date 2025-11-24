---
aliases: [Framer Motion, Framer, React анимации]
tags: [typescript, javascript, анимации, библиотека, react, веб-разработка]
---

# Framer Motion в TypeScript проектах

Framer Motion — это мощная библиотека анимаций для React, разработанная командой Framer. Она предоставляет простой и интуитивный API для создания сложных анимационных эффектов с высокой производительностью. TypeScript обеспечивает строгую типизацию при работе с Framer Motion, что делает разработку более безопасной и предсказуемой.

## Установка и настройка

Для использования Framer Motion в TypeScript проекте необходимо установить соответствующие пакеты:

```bash
npm install framer-motion
```

## Основы Framer Motion

Framer Motion предоставляет компоненты для анимаций:

- `motion` — для анимации DOM-элементов
- `AnimatePresence` — для анимации при монтировании/размонтировании
- `motion`-компоненты — для анимации различных свойств

### Простая анимация

```typescript
import { motion } from 'framer-motion';

const SimpleAnimation: React.FC = () => {
  return (
    <motion.div
      initial={{ opacity: 0, scale: 0.5 }}
      animate={{ opacity: 1, scale: 1 }}
      transition={{ duration: 0.5 }}
    >
      Анимированный элемент
    </motion.div>
  );
};
```

### Анимация при наведении

```typescript
const HoverAnimation: React.FC = () => {
  return (
    <motion.div
      whileHover={{ scale: 1.1 }}
      whileTap={{ scale: 0.9 }}
      style={{
        width: 100,
        height: 100,
        backgroundColor: 'blue',
        borderRadius: 10
      }}
    >
      Наведи на меня
    </motion.div>
  );
};
```

### Анимация с вариациями

```typescript
import { motion, Variants } from 'framer-motion';

const containerVariants: Variants = {
  hidden: { opacity: 0, x: -100 },
  visible: {
    opacity: 1,
    x: 0,
    transition: {
      duration: 0.5,
      staggerChildren: 0.2
    }
  }
};

const itemVariants: Variants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 }
};

const StaggeredAnimation: React.FC = () => {
  return (
    <motion.div
      variants={containerVariants}
      initial="hidden"
      animate="visible"
    >
      <motion.div variants={itemVariants}>Элемент 1</motion.div>
      <motion.div variants={itemVariants}>Элемент 2</motion.div>
      <motion.div variants={itemVariants}>Элемент 3</motion.div>
    </motion.div>
  );
};
```

## Типизация Framer Motion

TypeScript позволяет создавать типизированные конфигурации анимаций:

```typescript
import { motion, MotionProps, Variants, Transition } from 'framer-motion';

// Тип для свойств анимации
interface AnimationConfig {
  initial?: MotionProps['initial'];
  animate?: MotionProps['animate'];
  whileHover?: MotionProps['whileHover'];
  whileTap?: MotionProps['whileTap'];
  transition?: Transition;
  variants?: Variants;
}

// Типизированный компонент анимации
interface AnimatedBoxProps extends AnimationConfig {
  children: React.ReactNode;
  className?: string;
}

const AnimatedBox: React.FC<AnimatedBoxProps> = ({
  children,
  initial,
  animate,
  whileHover,
  whileTap,
  transition,
  variants,
  className
}) => {
  return (
    <motion.div
      initial={initial}
      animate={animate}
      whileHover={whileHover}
      whileTap={whileTap}
      transition={transition}
      variants={variants}
      className={className}
    >
      {children}
    </motion.div>
  );
};

// Использование типизированного компонента
const MyComponent: React.FC = () => {
  const animationConfig: AnimationConfig = {
    initial: { opacity: 0, scale: 0.8 },
    animate: { opacity: 1, scale: 1 },
    whileHover: { scale: 1.05 },
    whileTap: { scale: 0.95 },
    transition: { duration: 0.3 }
  };

  return (
    <AnimatedBox {...animationConfig}>
      Анимированный контент
    </AnimatedBox>
  );
};
```

## Анимации переходов с AnimatePresence

Компонент `AnimatePresence` позволяет анимировать появление и исчезновение элементов:

```typescript
import { motion, AnimatePresence } from 'framer-motion';

interface FadeInOutProps {
  isVisible: boolean;
  children: React.ReactNode;
}

const FadeInOut: React.FC<FadeInOutProps> = ({ isVisible, children }) => {
  return (
    <AnimatePresence>
      {isVisible && (
        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
          transition={{ duration: 0.3 }}
        >
          {children}
        </motion.div>
      )}
    </AnimatePresence>
  );
};

// Использование
const App: React.FC = () => {
  const [isVisible, setIsVisible] = React.useState(true);

  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>
        Переключить видимость
      </button>
      <FadeInOut isVisible={isVisible}>
        <div>Содержимое с анимацией появления/исчезновения</div>
      </FadeInOut>
    </div>
  );
};
```

## Сложные анимации с Drag и Gestures

Framer Motion поддерживает drag-анимации и жесты:

```typescript
import { motion } from 'framer-motion';

const DraggableBox: React.FC = () => {
  const [position, setPosition] = React.useState({ x: 0, y: 0 });

  return (
    <motion.div
      drag
      dragConstraints={{ left: 0, right: 400, top: 0, bottom: 300 }}
      dragElastic={0.2}
      onDrag={(event, info) => {
        setPosition({ x: info.point.x, y: info.point.y });
      }}
      style={{
        width: 100,
        height: 100,
        backgroundColor: 'red',
        cursor: 'grab'
      }}
    >
      Перетащи меня
    </motion.div>
  );
};
```

## Анимации маршрутов

Framer Motion идеально подходит для анимации переходов между маршрутами:

```typescript
import { motion } from 'framer-motion';
import { Route, Routes, useLocation } from 'react-router-dom';

const AnimatedRoutes: React.FC = () => {
  const location = useLocation();

  return (
    <AnimatePresence mode="wait">
      <Routes location={location} key={location.pathname}>
        <Route
          path="/"
          element={
            <motion.div
              initial={{ opacity: 0, x: -100 }}
              animate={{ opacity: 1, x: 0 }}
              exit={{ opacity: 0, x: 100 }}
              transition={{ duration: 0.3 }}
            >
              Главная страница
            </motion.div>
          }
        />
        <Route
          path="/about"
          element={
            <motion.div
              initial={{ opacity: 0, x: 100 }}
              animate={{ opacity: 1, x: 0 }}
              exit={{ opacity: 0, x: -100 }}
              transition={{ duration: 0.3 }}
            >
              О нас
            </motion.div>
          }
        />
      </Routes>
    </AnimatePresence>
  );
};
```

## Класс для управления анимациями

Создадим класс для централизованного управления Framer Motion анимациями:

```typescript
import { motion, MotionProps, Variants, Transition } from 'framer-motion';

type AnimationType = 'fade' | 'slide' | 'scale' | 'bounce' | 'rotate';

interface AnimationPreset {
  initial: MotionProps['initial'];
  animate: MotionProps['animate'];
  exit?: MotionProps['exit'];
  transition: Transition;
}

class AnimationPresetManager {
  private presets: Map<AnimationType, AnimationPreset> = new Map();

  constructor() {
    this.initializePresets();
  }

  private initializePresets(): void {
    // Пресет для появления/исчезновения
    this.presets.set('fade', {
      initial: { opacity: 0 },
      animate: { opacity: 1 },
      exit: { opacity: 0 },
      transition: { duration: 0.3 }
    });

    // Пресет для слайд-анимации
    this.presets.set('slide', {
      initial: { x: -100, opacity: 0 },
      animate: { x: 0, opacity: 1 },
      exit: { x: 100, opacity: 0 },
      transition: { duration: 0.4 }
    });

    // Пресет для масштабирования
    this.presets.set('scale', {
      initial: { scale: 0.8, opacity: 0 },
      animate: { scale: 1, opacity: 1 },
      exit: { scale: 0.8, opacity: 0 },
      transition: { duration: 0.3, type: 'spring', stiffness: 300, damping: 20 }
    });

    // Пресет для прыжка
    this.presets.set('bounce', {
      initial: { y: 50, opacity: 0 },
      animate: { y: 0, opacity: 1 },
      exit: { y: 50, opacity: 0 },
      transition: { duration: 0.5, type: 'spring', stiffness: 500, damping: 25 }
    });

    // Пресет для вращения
    this.presets.set('rotate', {
      initial: { rotate: -15, opacity: 0 },
      animate: { rotate: 0, opacity: 1 },
      exit: { rotate: 15, opacity: 0 },
      transition: { duration: 0.4 }
    });
  }

  public getAnimationPreset(type: AnimationType): AnimationPreset | undefined {
    return this.presets.get(type);
  }

  public addCustomPreset(name: string, preset: AnimationPreset): void {
    this.presets.set(name as AnimationType, preset);
  }

  public createMotionComponent(
    type: AnimationType,
    children: React.ReactNode,
    additionalProps?: Partial<MotionProps>
  ): React.ReactElement {
    const preset = this.getAnimationPreset(type);
    if (!preset) {
      throw new Error(`Пресет анимации "${type}" не найден`);
    }

    return motion.div({
      ...preset,
      ...additionalProps,
      children
    });
  }
}

// Использование менеджера пресетов
const presetManager = new AnimationPresetManager();

const PresetAnimatedComponent: React.FC<{ type: AnimationType }> = ({ type }) => {
  const preset = presetManager.getAnimationPreset(type);
  
  if (!preset) {
    return <div>Неизвестный тип анимации</div>;
  }

  return (
    <motion.div
      initial={preset.initial}
      animate={preset.animate}
      exit={preset.exit}
      transition={preset.transition}
    >
      Анимированный компонент с пресетом: {type}
    </motion.div>
  );
};
```

## Анимации с TypeScript и пользовательскими хуками

Создадим пользовательские хуки для управления анимациями:

```typescript
import { useState, useEffect } from 'react';
import { motion, MotionProps } from 'framer-motion';

// Хук для анимации появления
export const useFadeIn = (
  initialOpacity: number = 0,
  targetOpacity: number = 1,
  duration: number = 0.5
): [MotionProps, (value: boolean) => void] => {
  const [isVisible, setIsVisible] = useState(false);

  const animationProps: MotionProps = {
    initial: { opacity: initialOpacity },
    animate: { opacity: isVisible ? targetOpacity : initialOpacity },
    transition: { duration }
  };

  return [animationProps, setIsVisible];
};

// Хук для анимации слайдов
export const useSlideIn = (
  direction: 'left' | 'right' | 'up' | 'down' = 'left',
  distance: number = 100,
  duration: number = 0.5
): [MotionProps, (value: boolean) => void] => {
  const [isVisible, setIsVisible] = useState(false);

  const getInitialPosition = () => {
    switch (direction) {
      case 'left': return { x: -distance };
      case 'right': return { x: distance };
      case 'up': return { y: -distance };
      case 'down': return { y: distance };
      default: return { x: -distance };
    }
  };

  const animationProps: MotionProps = {
    initial: { ...getInitialPosition(), opacity: 0 },
    animate: { x: 0, y: 0, opacity: isVisible ? 1 : 0 },
    transition: { duration }
  };

  return [animationProps, setIsVisible];
};

// Использование хуков
const HookAnimatedComponent: React.FC = () => {
  const [fadeInProps, setFadeInVisible] = useFadeIn();
  const [slideInProps, setSlideInVisible] = useSlideIn('up', 50);

  useEffect(() => {
    // Анимация запускается после рендера
    setTimeout(() => {
      setFadeInVisible(true);
      setSlideInVisible(true);
    }, 100);
  }, []);

  return (
    <div>
      <motion.div {...fadeInProps}>
        Элемент с fadeIn анимацией
      </motion.div>
      <motion.div {...slideInProps}>
        Элемент с slideIn анимацией
      </motion.div>
    </div>
  );
};
```

## Анимации для списков

Framer Motion отлично подходит для анимации списков:

```typescript
import { motion } from 'framer-motion';

interface ListItem {
  id: number;
  text: string;
}

interface AnimatedListProps {
  items: ListItem[];
  onRemove: (id: number) => void;
}

const AnimatedList: React.FC<AnimatedListProps> = ({ items, onRemove }) => {
  return (
    <div>
      {items.map((item) => (
        <motion.div
          key={item.id}
          layout
          initial={{ opacity: 0, height: 0 }}
          animate={{ opacity: 1, height: 'auto' }}
          exit={{ opacity: 0, height: 0 }}
          transition={{ duration: 0.3 }}
          style={{
            padding: '10px',
            margin: '5px 0',
            backgroundColor: '#f0f0f0',
            borderRadius: '4px'
          }}
        >
          {item.text}
          <button 
            onClick={() => onRemove(item.id)}
            style={{ marginLeft: '10px' }}
          >
            Удалить
          </button>
        </motion.div>
      ))}
    </div>
  );
};
```

## Заключение

Framer Motion — мощный инструмент для создания анимаций в React-приложениях. Его интеграция с TypeScript обеспечивает безопасность типов и улучшает качество кода. Библиотека предоставляет широкие возможности для создания как простых, так и сложных анимационных эффектов.

## См. также

- [[CSS-анимации]]
- [[JavaScript-анимации]]
- [[GSAP]]
- [[WebGL-анимации]]
- [[React-анимации]]
- [[React Spring]]
- [[Lottie]]
- [[Animation libraries comparison]]
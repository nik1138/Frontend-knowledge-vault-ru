---
aliases: [CSS Transform Specification, CSS Transform Functions, CSS 2D and 3D Transforms, CSS Transform Properties]
tags: [css, transform, 3d, specification, css-transforms]
---

# CSS Transform Module: 2D и 3D трансформации, перспектива - Полное руководство

## Введение

CSS Transform Module позволяет изменять визуальное представление элементов без влияния на документный поток. Модуль поддерживает 2D и 3D трансформации, включая перемещение, масштабирование, вращение и наклон.

## CSS Transforms Module Level 1

Level 1 определяет основные возможности 2D трансформаций.

### Основные свойства трансформации

```css
.transform-element {
  /* Основное свойство трансформации */
  transform: none;                  /* Без трансформации */
  transform: translate(10px, 20px); /* Перемещение */
  transform: rotate(45deg);         /* Вращение */
  transform: scale(1.5);            /* Масштабирование */
  transform: skew(30deg, 20deg);    /* Наклон */
  transform: matrix(1, 0.5, -0.5, 1, 0, 0); /* Матрица трансформации */
  
  /* Комбинация трансформаций */
  transform: translateX(50px) rotate(45deg) scale(1.2);
  
  /* Точка трансформации */
  transform-origin: 50% 50%;        /* По умолчанию - центр элемента */
  transform-origin: left top;       /* Левый верхний угол */
  transform-origin: 10px 20px;      /* Абсолютные координаты */
  
  /* Порядок трансформаций */
  transform: rotate(45deg) translateX(100px); /* Сначала вращение, потом перемещение */
  transform: translateX(100px) rotate(45deg); /* Сначала перемещение, потом вращение */
}
```

### 2D трансформации

#### Перемещение (Translate)

```css
.translate-examples {
  /* Перемещение по осям */
  transform: translate(10px, 20px);  /* X и Y */
  transform: translateX(15px);       /* Только по X */
  transform: translateY(25px);       /* Только по Y */
  
  /* Процентные значения */
  transform: translate(10%, 20%);    /* Относительно размеров элемента */
}
```

#### Масштабирование (Scale)

```css
.scale-examples {
  transform: scale(1.5);             /* Равномерное масштабирование */
  transform: scale(1.5, 0.8);        /* Масштабирование по X и Y */
  transform: scaleX(1.2);            /* Только по X */
  transform: scaleY(0.8);            /* Только по Y */
  
  /* Отрицательные значения создают отражение */
  transform: scaleX(-1);             /* Горизонтальное отражение */
  transform: scaleY(-1);             /* Вертикальное отражение */
}
```

#### Вращение (Rotate)

```css
.rotate-examples {
  transform: rotate(45deg);          /* Вращение в 2D */
  transform: rotateX(0);             /* Вращение вокруг X (2D) */
  transform: rotateY(0);             /* Вращение вокруг Y (2D) */
  transform: rotateZ(45deg);         /* Вращение вокруг Z (то же, что rotate) */
}
```

#### Наклон (Skew)

```css
.skew-examples {
  transform: skew(30deg);            /* Наклон по X и Y */
  transform: skew(30deg, 10deg);     /* Наклон по X и Y отдельно */
  transform: skewX(20deg);           /* Только наклон по X */
  transform: skewY(15deg);           /* Только наклон по Y */
}
```

## CSS Transforms Module Level 2

Level 2 добавляет поддержку 3D трансформаций и дополнительные функции.

### 3D трансформации

```css
.transform-3d-element {
  /* 3D перемещение */
  transform: translate3d(10px, 20px, 30px); /* X, Y, Z */
  transform: translateZ(50px);              /* Только по Z */
  
  /* 3D масштабирование */
  transform: scale3d(1.5, 1.2, 0.8);       /* X, Y, Z */
  transform: scaleZ(1.5);                   /* Только по Z */
  
  /* 3D вращение */
  transform: rotate3d(1, 0, 0, 45deg);      /* Вращение вокруг вектора (x, y, z) */
  transform: rotateX(30deg);                /* Вращение вокруг X */
  transform: rotateY(45deg);                /* Вращение вокруг Y */
  transform: rotateZ(60deg);                /* Вращение вокруг Z */
  
  /* 3D матрица */
  transform: matrix3d(
    1, 0, 0, 0,    /* 1st column */
    0, 1, 0, 0,    /* 2nd column */
    0, 0, 1, 0,    /* 3rd column */
    0, 0, 0, 1     /* 4th column */
  );
}
```

### Свойства 3D контекста

```css
.perspective-container {
  /* Перспектива для дочерних элементов */
  perspective: 1000px;              /* Глубина перспективы */
  perspective-origin: 50% 50%;      /* Точка схода перспективы */
  
  /* Стэк трансформаций */
  transform-style: flat;            /* Плоские дочерние элементы */
  transform-style: preserve-3d;     /* Сохранение 3D для дочерних элементов */
}
```

### Примеры 3D трансформаций

```css
/* 3D карточка */
.card-container {
  perspective: 1000px;
  width: 200px;
  height: 260px;
}

.card {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.5s;
}

.card.flipped {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;      /* Скрытие обратной стороны */
}

.card-back {
  transform: rotateY(180deg);       /* Обратная сторона перевернута */
}
```

## Практические применения

### Анимации с трансформациями

```css
/* Плавное перемещение */
.move-element {
  transform: translateX(0);
  transition: transform 0.3s ease;
}

.move-element:hover {
  transform: translateX(100px);
}

/* Плавное масштабирование */
.scale-element {
  transform: scale(1);
  transition: transform 0.2s ease;
}

.scale-element:hover {
  transform: scale(1.1);
}

/* Вращение с сохранением позиции */
.rotate-element {
  transform-origin: center center;
  transform: rotate(0deg);
  transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

.rotate-element.active {
  transform: rotate(360deg);
}
```

### Интерактивные элементы

```css
/* 3D кнопка */
.button-3d {
  transform: translateZ(0);          /* Активирует GPU ускорение */
  transition: transform 0.2s ease;
}

.button-3d:active {
  transform: translateZ(-5px);       /* Эффект нажатия */
}

/* Карусель слайдов */
.carousel-item {
  position: absolute;
  width: 100%;
  height: 100%;
  opacity: 0;
  transform: translateX(100%);
  transition: transform 0.5s ease, opacity 0.5s ease;
}

.carousel-item.active {
  opacity: 1;
  transform: translateX(0);
}

.carousel-item.next {
  transform: translateX(-100%);
}

.carousel-item.prev {
  transform: translateX(100%);
}
```

### Эффекты интерфейса

```css
/* Сворачивание панели */
.collapse-panel {
  transform-origin: top center;
  transform: scaleY(1);
  transition: transform 0.3s ease, height 0.3s ease;
  overflow: hidden;
}

.collapse-panel.collapsed {
  transform: scaleY(0);
  height: 0 !important;
}

/* Модальное окно с анимацией */
.modal {
  transform: scale(0.7) opacity(0);
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.modal.open {
  transform: scale(1) opacity(1);
}

/* Эффект параллакса */
.parallax-element {
  transform: translateZ(0) translateY(0);
  will-change: transform;           /* Подсказка браузеру */
}

.parallax-container:hover .parallax-element {
  transform: translateZ(-50px) translateY(-10px);
}
```

## Производительность и оптимизация

### GPU ускорение

```css
.gpu-accelerated {
  /* Активация GPU ускорения */
  transform: translateZ(0);          /* Или translate3d(0,0,0) */
  will-change: transform;            /* Подсказка браузеру */
}

/* Свойства, вызывающие перерисовку */
.needs-redraw {
  /* Избегать анимации этих свойств */
  width: 100px;                     /* Вызывает перерасчет макета */
  height: 100px;
  background-color: red;            /* Вызывает перерисовку */
}

/* Свойства, оптимизированные для анимации */
.optimized-animation {
  /* Анимировать только эти свойства для лучшей производительности */
  transform: translateX(100px);      /* GPU ускорено */
  opacity: 0.5;                     /* GPU ускорено */
}
```

## Поддержка браузерами

- 2D трансформации: Поддерживается во всех современных браузерах
- 3D трансформации: Хорошая поддержка в современных браузерах (ограничения в старых IE)
- Необходимы префиксы для старых браузеров: `-webkit-transform`, `-moz-transform`, `-ms-transform`

## Связанные темы

- [[CSS Animation и Transition Modules]] - анимации и переходы
- [[CSS Positioning Module]] - позиционирование элементов
- [[CSS Box Model]] - модель коробки

## Заключение

CSS Transform Module предоставляет мощные инструменты для визуального изменения элементов без влияния на документный поток. Понимание 2D и 3D трансформаций позволяет создавать интерактивные и визуально привлекательные интерфейсы с высокой производительностью.
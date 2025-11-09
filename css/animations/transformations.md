# Анимации - Трансформации

CSS-трансформации позволяют изменять форму, размер и положение элементов в 2D и 3D пространстве без влияния на поток документа. Это мощный инструмент для создания визуальных эффектов и анимаций.

## Основы трансформаций

### transform

Свойство `transform` применяет трансформации к элементу:

```css
.basic-transforms {
  /* Перевод (смещение) */
  transform: translate(10px, 20px);
  
  /* Масштабирование */
  transform: scale(1.5);
  
  /* Поворот */
  transform: rotate(45deg);
  
  /* Наклон */
  transform: skew(10deg, 5deg);
  
  /* Комбинация трансформаций */
  transform: translate(10px, 20px) rotate(45deg) scale(1.2);
}
```

## 2D трансформации

### translate()

Смещает элемент по осям X и Y:

```css
.translate-examples {
  /* Смещение по X */
  transform: translateX(50px);
  
  /* Смещение по Y */
  transform: translateY(-30px);
  
  /* Смещение по обоим осям */
  transform: translate(20px, 40px);
  
  /* Использование процентов */
  transform: translate(10%, 5%);
}
```

### scale()

Масштабирует элемент:

```css
.scale-examples {
  /* Масштабирование по X */
  transform: scaleX(1.5);
  
  /* Масштабирование по Y */
  transform: scaleY(0.5);
  
  /* Масштабирование по обоим осям */
  transform: scale(1.2);      /* Равномерное масштабирование */
  transform: scale(1.5, 0.8); /* Разное масштабирование по осям */
}
```

### rotate()

Поворачивает элемент:

```css
.rotate-examples {
  /* Поворот на 45 градусов */
  transform: rotate(45deg);
  
  /* Поворот на 180 градусов */
  transform: rotate(0.5turn);  /* Используя обороты */
  transform: rotate(3.14159rad); /* Используя радианы */
}
```

### skew()

Наклоняет элемент:

```css
.skew-examples {
  /* Наклон по X */
  transform: skewX(20deg);
  
  /* Наклон по Y */
  transform: skewY(-10deg);
  
  /* Наклон по обоим осям */
  transform: skew(20deg, -10deg);
}
```

### matrix()

Комбинированная 2D трансформация с использованием матрицы:

```css
.matrix-example {
  /* matrix(scaleX, skewY, skewX, scaleY, translateX, translateY) */
  transform: matrix(1, 0.5, -0.5, 1, 50, 100);
}
```

## 3D трансформации

### translate3d()

Смещает элемент в 3D пространстве:

```css
.translate3d-examples {
  /* translate3d(x, y, z) */
  transform: translate3d(10px, 20px, 30px);
  
  /* Смещение только по Z */
  transform: translateZ(50px);  /* Приближение/удаление */
}
```

### scale3d()

Масштабирует элемент в 3D пространстве:

```css
.scale3d-examples {
  /* scale3d(x, y, z) */
  transform: scale3d(1.2, 1.5, 0.8);
  
  /* Масштабирование только по Z */
  transform: scaleZ(1.5);  /* Влияет на толщину элемента */
}
```

### rotate3d()

Поворачивает элемент вокруг произвольной оси:

```css
.rotate3d-examples {
  /* rotate3d(x, y, z, angle) */
  transform: rotate3d(1, 1, 0, 45deg);  /* Поворот вокруг диагонали */
  
  /* Повороты вокруг отдельных осей */
  transform: rotateX(30deg);  /* Поворот вокруг X (наклон вперед/назад) */
  transform: rotateY(45deg);  /* Поворот вокруг Y (поворот влево/вправо) */
  transform: rotateZ(60deg);  /* Поворот вокруг Z (обычный поворот) */
}
```

### perspective()

Создает перспективу для дочерних элементов:

```css
.perspective-container {
  perspective: 1000px;  /* Расстояние до плоскости */
  perspective-origin: center;  /* Точка перспективы */
}

.perspective-element {
  transform: rotateY(45deg);  /* Эффект будет виден благодаря перспективе */
}
```

### matrix3d()

Комбинированная 3D трансформация с использованием матрицы:

```css
.matrix3d-example {
  /* matrix3d определяет 4x4 матрицу */
  transform: matrix3d(
    1, 0, 0, 0,
    0, 1, 0, 0,
    0, 0, 1, 0,
    50, 100, 0, 1
  );
}
```

## Свойства для 3D трансформаций

### transform-style

Определяет, как дочерние элементы воспринимают 3D-пространство:

```css
.preserve-3d {
  transform-style: preserve-3d;  /* Дочерние элементы находятся в 3D-пространстве */
}

.flat {
  transform-style: flat;  /* Дочерние элементы спроецированы в 2D-плоскость */
}
```

### backface-visibility

Определяет, будет ли видна задняя сторона элемента:

```css
.show-backface {
  backface-visibility: visible;  /* Задняя сторона видна */
}

.hide-backface {
  backface-visibility: hidden;   /* Задняя сторона скрыта */
}
```

## Практические примеры

### Пример 1: Карточка с эффектом переворота

```css
.flip-container {
  perspective: 1000px;
  width: 200px;
  height: 267px;
}

.flip-card {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

.flip-card.flipped {
  transform: rotateY(180deg);
}

.flip-card-front,
.flip-card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
}

.flip-card-front {
  background-color: #3498db;
  color: white;
}

.flip-card-back {
  background-color: #e74c3c;
  color: white;
  transform: rotateY(180deg);
}
```

### Пример 2: Анимация при наведении

```css
.hover-transform {
  transition: transform 0.3s ease;
  display: inline-block;
}

.hover-transform:hover {
  transform: scale(1.1) rotate(5deg);
}

/* Более сложная анимация */
.complex-hover {
  transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.complex-hover:hover {
  transform: translate3d(0, -10px, 0) scale3d(1.05, 1.05, 1);
}
```

### Пример 3: 3D меню

```css
.menu-3d {
  display: flex;
  gap: 20px;
  transform-style: preserve-3d;
  perspective: 1000px;
}

.menu-item {
  padding: 15px 25px;
  background-color: #34495e;
  color: white;
  border-radius: 5px;
  transform: translateZ(20px);
  transition: transform 0.3s ease;
  cursor: pointer;
}

.menu-item:hover {
  transform: translateZ(30px);
}
```

### Пример 4: Анимированный логотип

```css
@keyframes logo-animation {
  0% {
    transform: rotate(0) scale(1);
  }
  25% {
    transform: rotate(90deg) scale(1.2);
  }
  50% {
    transform: rotate(180deg) scale(0.8);
  }
  75% {
    transform: rotate(270deg) scale(1.1);
  }
  100% {
    transform: rotate(360deg) scale(1);
  }
}

.animated-logo {
  animation: logo-animation 3s ease-in-out infinite;
  display: inline-block;
}
```

### Пример 5: Аккордеон с 3D эффектом

```css
.accordion-3d {
  transform-style: preserve-3d;
  perspective: 1000px;
}

.accordion-item {
  transform: translateZ(0);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  margin-bottom: 10px;
}

.accordion-item:hover {
  transform: translateZ(10px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.1);
}

.accordion-item.active {
  transform: translateZ(20px);
}
```

## Точки трансформации

### transform-origin

Определяет точку, относительно которой применяются трансформации:

```css
.origin-examples {
  /* По умолчанию: center (50% 50%) */
  transform-origin: center;
  
  /* Ключевые слова */
  transform-origin: top left;
  transform-origin: bottom right;
  transform-origin: center top;
  
  /* Абсолютные значения */
  transform-origin: 10px 20px;
  
  /* Относительные значения */
  transform-origin: 25% 75%;
  
  /* 3D точка трансформации */
  transform-origin: 50% 50% 20px;
}
```

## Современные возможности

### CSS-переменные в трансформациях

```css
:root {
  --transform-scale: 1.1;
  --transform-rotate: 5deg;
  --transform-distance: 10px;
}

.modern-transform {
  transition: transform 0.3s ease;
}

.modern-transform:hover {
  transform: 
    translate3d(0, calc(-1 * var(--transform-distance)), 0) 
    scale3d(var(--transform-scale), var(--transform-scale), 1)
    rotate(var(--transform-rotate));
}
```

### Адаптивные трансформации

```css
.responsive-transform {
  transform: scale(clamp(0.8, 0.1 * (100vw / 375), 1.2));
  /* Адаптируется к ширине экрана */
}
```

## Лучшие практики

1. **Используйте transform вместо изменения layout свойств** - для лучшей производительности
2. **Активируйте GPU ускорение** - используйте translateZ(0) или will-change при необходимости
3. **Избегайте чрезмерного использования 3D трансформаций** - на слабых устройствах может замедлить работу
4. **Тестируйте на разных устройствах** - особенно на мобильных с ограниченными ресурсами
5. **Учитывайте доступность** - уважайте `prefers-reduced-motion`
6. **Комбинируйте с transition или animation** - для плавных эффектов

## Распространенные ошибки

### 1. Неправильное использование perspective

```css
/* Плохо: perspective на самом элементе, который трансформируется */
.wrong-perspective {
  perspective: 1000px;
  transform: rotateY(45deg);  /* Перспектива не будет работать */
}

/* Хорошо: perspective на родительском элементе */
.correct-perspective {
  perspective: 1000px;
}

.correct-perspective .child {
  transform: rotateY(45deg);  /* Теперь перспектива работает */
}
```

### 2. Игнорирование transform-origin

```css
/* Элемент вращается вокруг центра (по умолчанию) */
.rotate-default {
  transform: rotate(45deg);
}

/* Элемент вращается вокруг левого верхнего угла */
.rotate-corner {
  transform-origin: top left;
  transform: rotate(45deg);
}
```

## Производительность

Для оптимальной производительности трансформаций:

```css
.performant-transform {
  /* Используйте hardware-accelerated трансформации */
  transform: translate3d(0, 0, 0) scale3d(1, 1, 1);
  
  /* Включите GPU ускорение при необходимости */
  will-change: transform;
  
  /* Или используйте translateZ(0) для активации GPU */
  transform: translateX(100px) translateZ(0);
}
```

## Заключение

CSS-трансформации - мощный инструмент для создания визуальных эффектов и анимаций. Они позволяют изменять положение, размер и ориентацию элементов без влияния на поток документа. Правильное использование 2D и 3D трансформаций может значительно улучшить пользовательский интерфейс и сделать его более интерактивным. Важно понимать различия между 2D и 3D трансформациями и использовать их с учетом производительности и доступности.

#programming #css #transformations #animations #web-development #frontend
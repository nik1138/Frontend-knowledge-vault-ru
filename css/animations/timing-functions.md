---
aliases: ["CSS Timing Functions", "Функции времени CSS", "CSS Анимации"]
tags: [css, animation, timing, web-development]
---

# Функции времени CSS (CSS Timing Functions)

## Введение

Функции времени CSS определяют, как значения анимации интерполируются во времени, создавая плавные или резкие переходы между начальным и конечным состояниями элемента. Эти функции играют ключевую роль в создании естественных и привлекательных анимаций.

## Определение функций времени

Функция времени (timing function) — это математическая функция, которая описывает, как происходит изменение свойства во времени анимации. Она определяет скорость изменения значений на протяжении продолжительности анимации.

## Встроенные функции времени

### ease
Функция по умолчанию, обеспечивающая плавное ускорение в начале и замедление в конце анимации.

```css
.element {
  animation-timing-function: ease;
}
```

### linear
Линейная функция, при которой анимация происходит с постоянной скоростью на протяжении всей анимации.

```css
.element {
  animation-timing-function: linear;
}
```

### ease-in
Анимация начинается медленно и ускоряется к концу.

```css
.element {
  animation-timing-function: ease-in;
}
```

### ease-out
Анимация начинается быстро и замедляется к концу.

```css
.element {
  animation-timing-function: ease-out;
}
```

### ease-in-out
Комбинация ease-in и ease-out — медленное начало, ускорение в середине и замедление в конце.

```css
.element {
  animation-timing-function: ease-in-out;
}
```

## Кривые Безье (cubic-bezier)

Функция `cubic-bezier(x1, y1, x2, y2)` позволяет создавать пользовательские функции времени с помощью кривой Безье. Значения x1, x2 должны быть в диапазоне от 0 до 1.

```css
.element {
  animation-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
}
```

> [!note] Примечание
> Значения y1 и y2 могут быть за пределами диапазона [0, 1], что позволяет создавать более сложные эффекты, такие как "отскок" или "перекручивание".

## Функция шагов (steps)

Функция `steps(n, position)` разбивает анимацию на определенное количество равных промежутков, создавая эффект "ступенчатого" движения.

- `n` — количество шагов
- `position` — позиция шага (start, end)

```css
.element {
  animation-timing-function: steps(4, end);
}
```

## Практические примеры

### Пример 1: Плавающий элемент
```css
.floating-element {
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0% { transform: translateY(0); }
  50% { transform: translateY(-20px); }
  100% { transform: translateY(0); }
}
```

### Пример 2: Пульсирующая кнопка
```css
.pulse-button {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}
```

### Пример 3: Ступенчатая анимация
```css
.step-animation {
  animation: step 4s steps(4, end) infinite;
}

@keyframes step {
  0% { background-position: 0 0; }
  100% { background-position: 100px 0; }
}
```

## Визуальные демонстрации различных функций времени

| Функция времени | Описание | Визуализация |
|----------------|----------|--------------|
| `ease` | Плавное начало и конец | /mnt/e/YandexDisk/obsidian/programming/css/animations/ease-curve.png |
| `linear` | Постоянная скорость | /mnt/e/YandexDisk/obsidian/programming/css/animations/linear-curve.png |
| `ease-in` | Медленное начало | /mnt/e/YandexDisk/obsidian/programming/css/animations/ease-in-curve.png |
| `ease-out` | Медленный конец | /mnt/e/YandexDisk/obsidian/programming/css/animations/ease-out-curve.png |
| `ease-in-out` | Медленное начало и конец | /mnt/e/YandexDisk/obsidian/programming/css/animations/ease-in-out-curve.png |

> [!tip] Совет
> Используйте инструменты разработчика браузера для визуализации кривых анимации и подбора оптимальных значений функций времени.

## Кривые анимации

Кривые анимации визуализируют изменение значений свойств во времени. Они помогают понять, как будет выглядеть анимация до её реализации.

- **Положительный наклон** — увеличение значения
- **Отрицательный наклон** — уменьшение значения
- **Крутой наклон** — быстрое изменение
- **Пологий наклон** — медленное изменение

## Продвинутые примеры

### Эластичная анимация
```css
.elastic {
  animation: elastic 1s cubic-bezier(0.68, -0.55, 0.265, 1.55) forwards;
}

@keyframes elastic {
  0% { transform: scale(0); }
  70% { transform: scale(1.1); }
  100% { transform: scale(1); }
}
```

### Отскок
```css
.bounce {
  animation: bounce 1s cubic-bezier(0.5, 0.05, 0.5, 1.5) forwards;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-30px); }
}
```

## Связи с другими файлами CSS

- [[css/transform]] — для понимания свойств трансформации, используемых в анимациях
- [[css/keyframes]] — для создания ключевых кадров анимации
- [[css/transitions]] — для понимания переходов, использующих те же функции времени
- [[css/animation-properties]] — для полного понимания свойств анимации

## Заключение

Функции времени CSS являются мощным инструментом для создания естественных и привлекательных анимаций. Правильный выбор функции времени может значительно улучшить пользовательский опыт и восприятие интерфейса. Понимание встроенных функций и умение создавать пользовательские кривые Безье позволяет точно контролировать поведение анимаций на веб-страницах.

> [!warning] Важно
> Избегайте чрезмерного использования сложных анимаций, так как это может вызвать усталость глаз или дискомфорт у пользователей. Также учитывайте предпочтения пользователей по анимации (prefers-reduced-motion).

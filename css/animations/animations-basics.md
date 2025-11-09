# css/animations/basics

Этот файл описывает основы CSS-анимаций.

CSS Animations Basics охватывает фундаментальные концепции и техники создания анимационных эффектов с использованием CSS. Это основа для понимания более сложных анимационных возможностей.

## Основные концепции:

1. **Переходы (Transitions)**: Плавные изменения значений свойств при изменении состояния элемента
2. **Ключевые кадры (Keyframes)**: Определение промежуточных состояний анимации
3. **Свойства анимации**: Управление продолжительностью, таймингом и повторением
4. **Трансформации**: Изменение формы, размера и положения элементов

## Простой пример перехода:

```css
.button {
  background-color: blue;
  transition: background-color 0.3s ease;
}

.button:hover {
  background-color: red;
}
```

## Простой пример анимации с ключевыми кадрами:

```css
@keyframes slideIn {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

.slide-element {
  animation: slideIn 0.5s ease-out;
}
```

## См. также:
- [[animations]]
- [[transitions]]
- [[keyframes]]
- [[transformations]]

#css #animations #basics #transitions #keyframes #web-development
# CSS Animation

Этот файл описывает CSS-анимации.

CSS-анимации позволяют анимировать переходы между стилями CSS без использования JavaScript или Flash. В отличие от переходов (transitions), анимации могут иметь несколько промежуточных состояний и более сложное поведение.

## Основные свойства:

- animation-name: Определяет имя @keyframes правила
- animation-duration: Определяет, как долго длится анимация
- animation-timing-function: Определяет кривую скорости анимации
- animation-delay: Определяет задержку перед началом анимации
- animation-iteration-count: Определяет, сколько раз будет воспроизводиться анимация
- animation-direction: Определяет, будет ли анимация воспроизводиться вперед, назад или поочередно
- animation-fill-mode: Определяет стили, применяемые до начала и после окончания анимации
- animation-play-state: Определяет, запущена или приостановлена анимация

## @keyframes правило:

```css
@keyframes slideIn {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(0); }
}
```

## См. также:
- [[animations]]
- [[keyframes]]
- [[transitions]]

#css #animations #keyframes #web-development
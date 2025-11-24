---
aliases: [Сенсорные события, Touch Events]
tags: [javascript, mobile, touch, events]
---

# Touch-события

Touch-события — это специальные события, которые возникают при взаимодействии пользователя с сенсорным экраном. В 2025 году понимание и правильная обработка этих событий критически важна для создания качественных мобильных веб-приложений.

## Основные touch-события

- `touchstart`: Событие возникает при касании экрана.
- `touchmove`: Событие возникает при перемещении пальца по экрану.
- `touchend`: Событие возникает при отрыве пальца от экрана.
- `touchcancel`: Событие возникает при прерывании touch-события (например, входящий звонок).

## Обработка touch-событий

```javascript
element.addEventListener('touchstart', handleTouchStart, false);
element.addEventListener('touchmove', handleTouchMove, false);
element.addEventListener('touchend', handleTouchEnd, false);

var xDown = null;
var yDown = null;

function handleTouchStart(evt) {
  xDown = evt.touches[0].clientX;
  yDown = evt.touches[0].clientY;
}

function handleTouchMove(evt) {
  if (!xDown || !yDown) {
    return;
  }

  var xUp = evt.touches[0].clientX;
  var yUp = evt.touches[0].clientY;

  var xDiff = xDown - xUp;
  var yDiff = yDown - yUp;

  if (Math.abs(xDiff) > Math.abs(yDiff)) {
    // Свайп по горизонтали
    if (xDiff > 0) {
      /* Свайп влево */
    } else {
      /* Свайп вправо */
    }
  } else {
    // Свайп по вертикали
    if (yDiff > 0) {
      /* Свайп вверх */
    } else {
      /* Свайп вниз */
    }
  }

  xDown = null;
  yDown = null;
}
```

## Отличия от mouse-событий

Touch-события отличаются от mouse-событий тем, что они работают с несколькими точками касания одновременно. Также touch-события имеют дополнительные свойства, такие как `touches`, `targetTouches`, и `changedTouches`.

## Совместимость и полифиллы

Для обеспечения совместимости с более старыми браузерами можно использовать библиотеки, такие как `Hammer.js`, или создавать полифиллы для touch-событий.

## Особенности российского рынка

Российские пользователи активно используют жесты для навигации и взаимодействия с приложениями. Важно учитывать, что на некоторых устройствах могут быть проблемы с точностью touch-событий, особенно на более дешевых моделях.

## См. также

- [[Адаптивность]]
- [[Мобильные-API]]
- [[Обработка-жестов]]
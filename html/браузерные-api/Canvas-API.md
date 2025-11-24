---
aliases: [Canvas, HTML Canvas, Графика в браузере]
tags: [html, browser-api, canvas, javascript]
---

# Canvas API

## Обзор

Canvas API предоставляет мощные возможности для рендеринга графики в браузере. Он позволяет рисовать 2D и 3D графику, анимации и даже игры. Canvas представляет собой область на веб-странице, в которой можно рисовать с помощью JavaScript.

## Основные методы

- `getContext()` - получает контекст рисования (2D или WebGL).
- `fillRect()` - рисует закрашенный прямоугольник.
- `strokeRect()` - рисует контур прямоугольника.
- `fillText()` - рисует текст.
- `drawImage()` - рисует изображения.

## Практические рекомендации

### Создание Canvas

```html
<canvas id="myCanvas" width="500" height="500"></canvas>
```

### Рисование на Canvas

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// Рисование прямоугольника
ctx.fillStyle = 'blue';
ctx.fillRect(10, 10, 100, 100);

// Рисование текста
ctx.fillStyle = 'black';
ctx.font = '20px Arial';
ctx.fillText('Привет, Canvas!', 50, 50);
```

## Ограничения и особенности

- Canvas не поддерживает доступность, так как это растровое изображение.
- Для сложных интерактивных графических элементов может потребоваться дополнительная обработка событий.
- Производительность может быть проблемой при рендеринге сложных сцен.

## Ссылки

- [[HTML5]]
- [[JavaScript]]
- [[WebGL]]

## Теги

#html #browser-api #canvas #javascript #web-development
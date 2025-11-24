---
aliases: [HTML Canvas, Canvas 2D, WebGL, Растровый холст]
tags: [javascript, web-api, canvas, graphics, frontend, визуализация]
---

# Canvas API

## Обзор

Canvas API - это мощный интерфейс веб-платформы, позволяющий рисовать 2D и 3D графику с помощью JavaScript. HTML-элемент `<canvas>` предоставляет область, в которой сценарии могут отображать визуальные эффекты, анимации, игры, диаграммы и другие изображения. Canvas API особенно популярен в 2025 году для создания интерактивных веб-приложений, визуализаций данных и игр в браузере.

## Основы Canvas

### Создание Canvas элемента

```html
<canvas id="myCanvas" width="800" height="600">
    Ваш браузер не поддерживает элемент canvas.
</canvas>
```

### Получение контекста рисования

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d'); // 2D контекст
// или
const webglCtx = canvas.getContext('webgl'); // WebGL контекст
```

## 2D контекст Canvas

### Базовые операции рисования

```javascript
// Получение контекста
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// Установка стилей рисования
ctx.fillStyle = '#FF6B6B'; // Цвет заливки
ctx.strokeStyle = '#4ECDC4'; // Цвет обводки
ctx.lineWidth = 2; // Ширина линии

// Рисование прямоугольника
ctx.fillRect(10, 10, 100, 80); // Залитый прямоугольник
ctx.strokeRect(120, 10, 100, 80); // Обведенный прямоугольник
ctx.clearRect(130, 20, 80, 60); // Очистка области (создает "дыру" в прямоугольнике)
```

### Рисование фигур

```javascript
// Рисование линии
ctx.beginPath();
ctx.moveTo(50, 50); // Начальная точка
ctx.lineTo(150, 100); // Конечная точка
ctx.stroke(); // Отображение линии

// Рисование дуги/окружности
ctx.beginPath();
ctx.arc(200, 75, 50, 0, 2 * Math.PI); // x, y, радиус, начальный угол, конечный угол
ctx.stroke();

// Рисование сектора
ctx.beginPath();
ctx.moveTo(200, 75); // Центр
ctx.arc(200, 75, 50, 0, Math.PI / 2); // Дуга
ctx.closePath(); // Замыкание пути
ctx.fillStyle = 'rgba(78, 205, 196, 0.5)';
ctx.fill();

// Рисование сложных фигур
ctx.beginPath();
ctx.moveTo(300, 50);
ctx.lineTo(350, 25);
ctx.lineTo(400, 50);
ctx.lineTo(375, 100);
ctx.lineTo(325, 100);
ctx.closePath();
ctx.fillStyle = '#45B7D1';
ctx.fill();
ctx.strokeStyle = '#333';
ctx.stroke();
```

### Работа с текстом

```javascript
// Установка параметров текста
ctx.font = '20px Arial';
ctx.fillStyle = '#333';
ctx.textAlign = 'center';
ctx.textBaseline = 'middle';

// Рисование текста
ctx.fillText('Привет, Canvas!', 400, 100); // Залитый текст
ctx.strokeText('Обведенный текст', 400, 150); // Обведенный текст

// Измерение текста
const textMetrics = ctx.measureText('Пример текста');
console.log('Ширина текста:', textMetrics.width);
```

## Продвинутые возможности Canvas

### Паттерны и градиенты

```javascript
// Создание линейного градиента
const gradient = ctx.createLinearGradient(0, 0, 200, 0);
gradient.addColorStop(0, '#FF6B6B');
gradient.addColorStop(0.5, '#4ECDC4');
gradient.addColorStop(1, '#45B7D1');

ctx.fillStyle = gradient;
ctx.fillRect(10, 200, 200, 100);

// Создание радиального градиента
const radialGradient = ctx.createRadialGradient(300, 250, 10, 300, 250, 50);
radialGradient.addColorStop(0, '#FFFFFF');
radialGradient.addColorStop(1, '#000000');

ctx.fillStyle = radialGradient;
ctx.beginPath();
ctx.arc(300, 250, 50, 0, 2 * Math.PI);
ctx.fill();

// Создание паттерна изображения
const img = new Image();
img.onload = function() {
    const pattern = ctx.createPattern(img, 'repeat');
    ctx.fillStyle = pattern;
    ctx.fillRect(350, 200, 200, 100);
};
img.src = 'pattern.png';
```

### Трансформации

```javascript
// Сохранение и восстановление состояния
ctx.save();

// Перевод начала координат
ctx.translate(400, 300);

// Поворот
ctx.rotate(Math.PI / 4); // Поворот на 45 градусов

// Масштабирование
ctx.scale(1.5, 1.5);

// Рисование фигуры в новых координатах
ctx.fillStyle = '#96CEB4';
ctx.fillRect(-50, -50, 100, 100);

// Восстановление предыдущего состояния
ctx.restore();

// Применение произвольной матрицы трансформации
ctx.transform(1, 0.2, 0.8, 1, 0, 0);
```

### Работа с изображениями

```javascript
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
const img = new Image();

img.onload = function() {
    // Простое отображение изображения
    ctx.drawImage(img, 10, 350);
    
    // Отображение с изменением размера
    ctx.drawImage(img, 150, 350, 100, 100);
    
    // Отображение части изображения
    ctx.drawImage(img, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight);
};
img.src = 'image.jpg';
```

## Анимация с Canvas

### Базовая анимация

```javascript
class CanvasAnimation {
    constructor(canvasId) {
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.animationId = null;
        this.objects = [];
    }
    
    addObject(obj) {
        this.objects.push(obj);
    }
    
    animate() {
        // Очистка холста
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        // Обновление и рендеринг объектов
        this.objects.forEach(obj => {
            obj.update();
            obj.render(this.ctx);
        });
        
        // Запуск следующего кадра
        this.animationId = requestAnimationFrame(() => this.animate());
    }
    
    start() {
        this.animate();
    }
    
    stop() {
        if (this.animationId) {
            cancelAnimationFrame(this.animationId);
        }
    }
}

// Пример объекта для анимации
class MovingCircle {
    constructor(x, y, radius, color) {
        this.x = x;
        this.y = y;
        this.radius = radius;
        this.color = color;
        this.vx = (Math.random() - 0.5) * 4; // Случайная скорость по X
        this.vy = (Math.random() - 0.5) * 4; // Случайная скорость по Y
    }
    
    update() {
        // Обновление позиции
        this.x += this.vx;
        this.y += this.vy;
        
        // Отскок от границ
        if (this.x + this.radius > canvas.width || this.x - this.radius < 0) {
            this.vx = -this.vx;
        }
        if (this.y + this.radius > canvas.height || this.y - this.radius < 0) {
            this.vy = -this.vy;
        }
    }
    
    render(ctx) {
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, 2 * Math.PI);
        ctx.fillStyle = this.color;
        ctx.fill();
    }
}

// Использование
const animation = new CanvasAnimation('myCanvas');
for (let i = 0; i < 5; i++) {
    const circle = new MovingCircle(
        Math.random() * 400,
        Math.random() * 300,
        10 + Math.random() * 20,
        `hsl(${Math.random() * 360}, 70%, 60%)`
    );
    animation.addObject(circle);
}
animation.start();
```

### Работа с пиксельными данными

```javascript
// Получение пиксельных данных
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
const data = imageData.data; // Массив RGBA значений

// Манипуляции с пиксельными данными
for (let i = 0; i < data.length; i += 4) {
    // data[i] = R, data[i+1] = G, data[i+2] = B, data[i+3] = A
    const red = data[i];
    const green = data[i + 1];
    const blue = data[i + 2];
    
    // Применение эффекта сепия
    const avg = (red + green + blue) / 3;
    data[i] = avg * 1.07;     // R
    data[i + 1] = avg * 0.74; // G
    data[i + 2] = avg * 0.43; // B
}

// Запись измененных данных обратно на холст
ctx.putImageData(imageData, 0, 0);
```

## WebGL (3D контекст)

### Основы WebGL

```javascript
const canvas = document.getElementById('myCanvas');
const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');

if (!gl) {
    console.error('WebGL не поддерживается');
}

// Определение шейдеров
const vertexShaderSource = `
    attribute vec4 aVertexPosition;
    void main() {
        gl_Position = aVertexPosition;
    }
`;

const fragmentShaderSource = `
    void main() {
        gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0); // Белый цвет
    }
`;

// Создание шейдеров
function createShader(gl, type, source) {
    const shader = gl.createShader(type);
    gl.shaderSource(shader, source);
    gl.compileShader(shader);
    
    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        console.error('Ошибка компиляции шейдера:', gl.getShaderInfoLog(shader));
        gl.deleteShader(shader);
        return null;
    }
    
    return shader;
}

const vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
const fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);

// Создание программы
const shaderProgram = gl.createProgram();
gl.attachShader(shaderProgram, vertexShader);
gl.attachShader(shaderProgram, fragmentShader);
gl.linkProgram(shaderProgram);

if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
    console.error('Ошибка линковки программы:', gl.getProgramInfoLog(shaderProgram));
}

gl.useProgram(shaderProgram);

// Определение вершин
const vertices = new Float32Array([
    -0.5, -0.5,
     0.5, -0.5,
     0.0,  0.5
]);

// Создание буфера
const vertexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);

// Атрибуты
const positionAttributeLocation = gl.getAttribLocation(shaderProgram, 'aVertexPosition');
gl.enableVertexAttribArray(positionAttributeLocation);
gl.vertexAttribPointer(positionAttributeLocation, 2, gl.FLOAT, false, 0, 0);

// Отрисовка
gl.clearColor(0.0, 0.0, 0.0, 1.0);
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, 3);
```

## Практические применения Canvas API

### Интерактивные диаграммы

```javascript
class ChartRenderer {
    constructor(canvasId) {
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.data = [];
    }
    
    setData(data) {
        this.data = data;
        this.render();
    }
    
    render() {
        const { width, height } = this.canvas;
        const padding = 40;
        const chartWidth = width - 2 * padding;
        const chartHeight = height - 2 * padding;
        
        // Очистка
        this.ctx.clearRect(0, 0, width, height);
        
        // Ось X и Y
        this.ctx.beginPath();
        this.ctx.moveTo(padding, height - padding);
        this.ctx.lineTo(width - padding, height - padding);
        this.ctx.lineTo(width - padding, padding);
        this.ctx.strokeStyle = '#333';
        this.ctx.stroke();
        
        if (this.data.length === 0) return;
        
        // Масштабирование данных
        const maxValue = Math.max(...this.data.map(d => d.value));
        const barWidth = chartWidth / this.data.length;
        
        // Рендеринг столбцов
        this.data.forEach((item, index) => {
            const x = padding + index * barWidth;
            const barHeight = (item.value / maxValue) * chartHeight;
            const y = height - padding - barHeight;
            
            // Градиент для столбца
            const gradient = this.ctx.createLinearGradient(0, y, 0, y + barHeight);
            gradient.addColorStop(0, '#4ECDC4');
            gradient.addColorStop(1, '#45B7D1');
            
            this.ctx.fillStyle = gradient;
            this.ctx.fillRect(x, y, barWidth - 2, barHeight);
            
            // Подпись
            this.ctx.fillStyle = '#333';
            this.ctx.font = '12px Arial';
            this.ctx.textAlign = 'center';
            this.ctx.fillText(item.label, x + barWidth/2, height - padding + 20);
        });
    }
}

// Использование
const chart = new ChartRenderer('myCanvas');
chart.setData([
    { label: 'Янв', value: 120 },
    { label: 'Фев', value: 195 },
    { label: 'Мар', value: 180 },
    { label: 'Апр', value: 210 },
    { label: 'Май', value: 150 }
]);
```

### Игры и интерактивные приложения

```javascript
class CanvasGame {
    constructor(canvasId) {
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.gameObjects = [];
        this.isRunning = false;
        
        // Обработка событий мыши
        this.canvas.addEventListener('click', this.handleCanvasClick.bind(this));
    }
    
    handleCanvasClick(event) {
        const rect = this.canvas.getBoundingClientRect();
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;
        
        // Проверка столкновений с объектами
        this.gameObjects.forEach(obj => {
            if (obj.containsPoint && obj.containsPoint(x, y)) {
                obj.onClick();
            }
        });
    }
    
    addObject(obj) {
        this.gameObjects.push(obj);
    }
    
    update() {
        this.gameObjects.forEach(obj => {
            if (obj.update) obj.update();
        });
    }
    
    render() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.gameObjects.forEach(obj => {
            if (obj.render) obj.render(this.ctx);
        });
    }
    
    gameLoop() {
        if (!this.isRunning) return;
        
        this.update();
        this.render();
        requestAnimationFrame(() => this.gameLoop());
    }
    
    start() {
        this.isRunning = true;
        this.gameLoop();
    }
    
    stop() {
        this.isRunning = false;
    }
}
```

## Производительность и оптимизация

### Техники оптимизации

```javascript
// Использование offscreen canvas для тяжелых вычислений
const offscreenCanvas = new OffscreenCanvas(800, 600);
const offscreenCtx = offscreenCanvas.getContext('2d');

// Рендеринг в offscreen canvas в воркере
const worker = new Worker('render-worker.js');
worker.postMessage({
    canvas: offscreenCanvas.transferControlToOffscreen()
}, [offscreenCanvas]);

// Использование буферизации для сложных сцен
class BufferedRenderer {
    constructor(width, height) {
        this.bufferCanvas = document.createElement('canvas');
        this.bufferCanvas.width = width;
        this.bufferCanvas.height = height;
        this.bufferCtx = this.bufferCanvas.getContext('2d');
    }
    
    renderScene() {
        // Рендеринг сцены в буфер
        this.bufferCtx.clearRect(0, 0, this.bufferCanvas.width, this.bufferCanvas.height);
        // ... рендеринг сложной сцены
        
        // Копирование в основной canvas
        mainCtx.drawImage(this.bufferCanvas, 0, 0);
    }
}

// Оптимизация анимации
function optimizedAnimation() {
    let lastTime = 0;
    const frameInterval = 1000 / 60; // 60 FPS
    
    function animate(currentTime) {
        if (currentTime - lastTime >= frameInterval) {
            // Обновление логики
            update();
            // Рендеринг
            render();
            lastTime = currentTime;
        }
        requestAnimationFrame(animate);
    }
    
    requestAnimationFrame(animate);
}
```

## Совместимость с браузерами

| Браузер | Поддержка Canvas | Поддержка WebGL |
|---------|------------------|-----------------|
| Chrome | С 9.0 | С 9.0 |
| Firefox | С 3.6 | С 4.0 |
| Safari | С 3.1 | С 5.1 |
| Edge | С 12.0 | С 12.0 |
| Internet Explorer | С 9.0 (частично) | С 11.0 |

## Безопасность и ограничения

> [!caution]
> Canvas API имеет ограничения безопасности:
> - Нельзя получить пиксельные данные изображений с других доменов без CORS заголовков
> - Canvas может быть помечен как "загрязненный" (tainted) при использовании изображений с других доменов

### Обход ограничений CORS

```javascript
const img = new Image();
img.crossOrigin = 'anonymous'; // Включение CORS
img.onload = function() {
    ctx.drawImage(img, 0, 0);
    // Теперь можно использовать getImageData()
};
img.src = 'https://example.com/image.jpg';
```

## Практические применения в российских реалиях 2025

1. **Финансовые визуализации** - интерактивные графики и диаграммы для банковских приложений
2. **Образовательные платформы** - интерактивные уроки с визуализациями
3. **Геймификация** - встроенные мини-игры в обучающих и маркетинговых приложениях
4. **Дашборды** - визуализация метрик и KPI для бизнес-приложений
5. **Редакторы изображений** - клиентские редакторы для обработки фото и графики
6. **Картографические сервисы** - кастомные карты и навигация

## Альтернативы и дополнения

- [[WebGL]] - для 3D графики и сложных визуализаций
- [[SVG]] - для векторной графики и масштабируемых изображений
- [[WebGL2]] - расширенные возможности 3D графики
- [[WebGPU]] - новое поколение API для графики и вычислений

## Заключение

Canvas API предоставляет мощные возможности для создания динамического и интерактивного контента в браузере. В 2025 году он остается важным инструментом для разработчиков, особенно при создании:

- Интерактивных визуализаций данных
- Веб-игр и развлекательных приложений
- Образовательных и тренировочных платформ
- Графических редакторов
- Научных и инженерных приложений

При правильном использовании Canvas API может значительно улучшить пользовательский опыт, обеспечивая богатые визуальные возможности в веб-приложениях. Важно учитывать производительность, безопасность и доступность при разработке приложений с использованием Canvas API.

---
aliases: ["Примеры работы с canvas и графикой", "Canvas и графика в JavaScript"]
tags: [js, practical-examples, canvas, graphics, frontend]
---

# Примеры работы с canvas и графикой

Практические примеры работы с HTML5 Canvas API для создания графики, анимаций и визуализаций в браузере.

## 1. Основы работы с Canvas

```html
<!DOCTYPE html>
<html>
<head>
    <title>Canvas Примеры</title>
</head>
<body>
    <canvas id="myCanvas" width="800" height="600" style="border: 1px solid #000;"></canvas>
    <script>
        // Получение контекста рисования
        const canvas = document.getElementById('myCanvas');
        const ctx = canvas.getContext('2d');
        
        // Основные свойства для стиля рисования
        ctx.fillStyle = '#3498db'; // цвет заливки
        ctx.strokeStyle = '#e74c3c'; // цвет обводки
        ctx.lineWidth = 2; // толщина линии
        
        // Рисование прямоугольника
        ctx.fillRect(10, 10, 100, 50); // залитый прямоугольник
        ctx.strokeRect(120, 10, 100, 50); // обведенный прямоугольник
        ctx.clearRect(50, 25, 20, 20); // очистка области
        
        // Рисование линии
        ctx.beginPath();
        ctx.moveTo(250, 25);
        ctx.lineTo(300, 75);
        ctx.stroke();
        
        // Рисование круга
        ctx.beginPath();
        ctx.arc(400, 50, 30, 0, 2 * Math.PI);
        ctx.fillStyle = '#2ecc71';
        ctx.fill();
        ctx.strokeStyle = '#34495e';
        ctx.lineWidth = 3;
        ctx.stroke();
    </script>
</body>
</html>
```

## 2. Работа с путями и сложными фигурами

```javascript
// Функция для рисования многоугольника
function drawPolygon(ctx, x, y, radius, sides, startAngle = 0) {
    if (sides < 3) return;
    
    ctx.beginPath();
    for (let i = 0; i < sides; i++) {
        const angle = startAngle + (i * 2 * Math.PI / sides);
        const px = x + radius * Math.cos(angle);
        const py = y + radius * Math.sin(angle);
        
        if (i === 0) {
            ctx.moveTo(px, py);
        } else {
            ctx.lineTo(px, py);
        }
    }
    ctx.closePath();
    ctx.fill();
    ctx.stroke();
}

// Рисование звезды
function drawStar(ctx, cx, cy, spikes, outerRadius, innerRadius) {
    let rot = Math.PI / 2 * 3;
    let x = cx;
    let y = cy;
    let step = Math.PI / spikes;

    ctx.beginPath();
    ctx.moveTo(cx, cy - outerRadius);
    
    for (let i = 0; i < spikes; i++) {
        x = cx + Math.cos(rot) * outerRadius;
        y = cy + Math.sin(rot) * outerRadius;
        ctx.lineTo(x, y);
        rot += step;

        x = cx + Math.cos(rot) * innerRadius;
        y = cy + Math.sin(rot) * innerRadius;
        ctx.lineTo(x, y);
        rot += step;
    }
    
    ctx.lineTo(cx, cy - outerRadius);
    ctx.closePath();
    ctx.fill();
    ctx.stroke();
}

// Использование функций
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// Рисуем различные многоугольники
ctx.fillStyle = '#9b59b6';
drawPolygon(ctx, 100, 150, 40, 3); // треугольник

ctx.fillStyle = '#f39c12';
drawPolygon(ctx, 200, 150, 40, 5); // пятиугольник

ctx.fillStyle = '#1abc9c';
drawStar(ctx, 300, 150, 5, 40, 20); // пятиконечная звезда
```

## 3. Работа с текстом и шрифтами

```javascript
// Рисование текста с различными стилями
function drawStyledText(ctx, text, x, y) {
    // Основные свойства текста
    ctx.font = '20px Arial';
    ctx.fillStyle = '#2c3e50';
    ctx.textAlign = 'left';
    ctx.textBaseline = 'top';
    
    // Рисуем основной текст
    ctx.fillText(text, x, y);
    
    // Добавляем обводку
    ctx.strokeStyle = '#e74c3c';
    ctx.lineWidth = 1;
    ctx.strokeText(text, x, y);
    
    // Измеряем ширину текста
    const textWidth = ctx.measureText(text).width;
    console.log(`Ширина текста "${text}": ${textWidth}px`);
    
    // Рисуем линию под текстом
    ctx.beginPath();
    ctx.moveTo(x, y + 25);
    ctx.lineTo(x + textWidth, y + 25);
    ctx.strokeStyle = '#3498db';
    ctx.lineWidth = 2;
    ctx.stroke();
}

// Рисование текста по кругу
function drawTextOnCircle(ctx, text, centerX, centerY, radius, ccw = false) {
    const textWidth = ctx.measureText(text).width;
    const anglePerChar = (Math.PI * 2) / text.length;
    
    ctx.save();
    ctx.translate(centerX, centerY);
    
    for (let i = 0; i < text.length; i++) {
        const char = text[i];
        const angle = ccw ? 
            -i * anglePerChar : 
            i * anglePerChar - Math.PI / 2; // начинаем сверху
            
        ctx.save();
        ctx.translate(0, -radius);
        ctx.rotate(angle);
        ctx.textAlign = 'center';
        ctx.fillText(char, 0, 0);
        ctx.restore();
        
        ctx.rotate(ccw ? -anglePerChar : anglePerChar);
    }
    
    ctx.restore();
}

// Использование функций
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

drawStyledText(ctx, 'Canvas Text Example', 50, 250);

ctx.font = '16px Verdana';
ctx.fillStyle = '#8e44ad';
drawTextOnCircle(ctx, 'CIRCLE TEXT', 400, 300, 80);
```

## 4. Анимация с Canvas

```javascript
// Простая анимация движения
function animateMovingCircle() {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    let x = 0;
    let y = canvas.height / 2;
    const radius = 20;
    let directionX = 2;
    let directionY = 1;
    
    function animate() {
        // Очищаем холст
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        // Обновляем позицию
        x += directionX;
        y += directionY;
        
        // Проверяем столкновения со стенами
        if (x + radius > canvas.width || x - radius < 0) {
            directionX = -directionX;
        }
        if (y + radius > canvas.height || y - radius < 0) {
            directionY = -directionY;
        }
        
        // Рисуем круг
        ctx.beginPath();
        ctx.arc(x, y, radius, 0, Math.PI * 2);
        ctx.fillStyle = '#e74c3c';
        ctx.fill();
        ctx.strokeStyle = '#c0392b';
        ctx.lineWidth = 2;
        ctx.stroke();
        
        // Продолжаем анимацию
        requestAnimationFrame(animate);
    }
    
    animate();
}

// Анимация частиц
function particleAnimation() {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    const particles = [];
    const particleCount = 100;
    
    // Класс для частицы
    class Particle {
        constructor() {
            this.x = Math.random() * canvas.width;
            this.y = Math.random() * canvas.height;
            this.vx = (Math.random() - 0.5) * 2;
            this.vy = (Math.random() - 0.5) * 2;
            this.radius = Math.random() * 3 + 1;
            this.color = `hsl(${Math.random() * 360}, 70%, 60%)`;
        }
        
        update() {
            this.x += this.vx;
            this.y += this.vy;
            
            // Проверяем границы
            if (this.x < 0 || this.x > canvas.width) this.vx *= -1;
            if (this.y < 0 || this.y > canvas.height) this.vy *= -1;
        }
        
        draw() {
            ctx.beginPath();
            ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
            ctx.fillStyle = this.color;
            ctx.fill();
        }
    }
    
    // Создаем частицы
    for (let i = 0; i < particleCount; i++) {
        particles.push(new Particle());
    }
    
    function animate() {
        ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        particles.forEach(particle => {
            particle.update();
            particle.draw();
        });
        
        requestAnimationFrame(animate);
    }
    
    animate();
}

// Запуск одной из анимаций
// animateMovingCircle();
// particleAnimation();
```

## 5. Работа с изображениями

```javascript
// Загрузка и отображение изображения
function drawImageExample() {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    const img = new Image();
    img.src = 'https://via.placeholder.com/150'; // Замените на реальный URL
    
    img.onload = function() {
        // Простое отображение изображения
        ctx.drawImage(img, 50, 350);
        
        // Отображение с изменением размера
        ctx.drawImage(img, 250, 350, 100, 100);
        
        // Отображение части изображения (вырезка)
        ctx.drawImage(img, 0, 0, 50, 50, 400, 350, 100, 100);
    };
}

// Создание градиентов
function drawGradients() {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    // Линейный градиент
    const gradient = ctx.createLinearGradient(0, 450, 200, 450);
    gradient.addColorStop(0, '#ff9a9e');
    gradient.addColorStop(0.5, '#fecfef');
    gradient.addColorStop(1, '#fecfef');
    
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 450, 200, 100);
    
    // Радиальный градиент
    const radialGradient = ctx.createRadialGradient(300, 500, 10, 300, 500, 50);
    radialGradient.addColorStop(0, '#a8edea');
    radialGradient.addColorStop(1, '#fed6e3');
    
    ctx.fillStyle = radialGradient;
    ctx.beginPath();
    ctx.arc(300, 500, 50, 0, Math.PI * 2);
    ctx.fill();
}

// Работа с паттернами
function drawPatterns() {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    // Создаем небольшое изображение для паттерна
    const patternCanvas = document.createElement('canvas');
    patternCanvas.width = 20;
    patternCanvas.height = 20;
    const patternCtx = patternCanvas.getContext('2d');
    
    patternCtx.fillStyle = '#3498db';
    patternCtx.fillRect(0, 0, 10, 10);
    patternCtx.fillStyle = '#e74c3c';
    patternCtx.fillRect(10, 10, 10, 10);
    
    // Создаем паттерн
    const pattern = ctx.createPattern(patternCanvas, 'repeat');
    
    // Используем паттерн
    ctx.fillStyle = pattern;
    ctx.fillRect(400, 450, 200, 100);
}
```

## 6. Создание диаграмм и графиков

```javascript
// Простая столбчатая диаграмма
function drawBarChart(data, labels, colors) {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    const chartWidth = 600;
    const chartHeight = 300;
    const margin = { top: 20, right: 20, bottom: 50, left: 50 };
    const width = chartWidth - margin.left - margin.right;
    const height = chartHeight - margin.top - margin.bottom;
    
    // Очищаем область
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Находим максимальное значение для масштабирования
    const maxValue = Math.max(...data);
    
    // Рисуем оси
    ctx.strokeStyle = '#333';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(margin.left, margin.top);
    ctx.lineTo(margin.left, chartHeight - margin.bottom);
    ctx.lineTo(chartWidth - margin.right, chartHeight - margin.bottom);
    ctx.stroke();
    
    // Рисуем столбцы
    const barWidth = width / data.length * 0.8;
    const barSpacing = width / data.length * 0.2;
    
    for (let i = 0; i < data.length; i++) {
        const x = margin.left + i * (barWidth + barSpacing);
        const barHeight = (data[i] / maxValue) * height;
        const y = chartHeight - margin.bottom - barHeight;
        
        // Столбец
        ctx.fillStyle = colors[i] || '#3498db';
        ctx.fillRect(x, y, barWidth, barHeight);
        
        // Обводка
        ctx.strokeStyle = '#2c3e50';
        ctx.lineWidth = 1;
        ctx.strokeRect(x, y, barWidth, barHeight);
        
        // Подпись
        ctx.fillStyle = '#333';
        ctx.font = '12px Arial';
        ctx.textAlign = 'center';
        ctx.fillText(labels[i], x + barWidth / 2, chartHeight - margin.bottom + 20);
        
        // Значение
        ctx.fillText(data[i].toString(), x + barWidth / 2, y - 10);
    }
    
    // Заголовок
    ctx.font = '16px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('Столбчатая диаграмма', chartWidth / 2, 15);
}

// Круговая диаграмма
function drawPieChart(data, labels, colors) {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    const centerX = 400;
    const centerY = 300;
    const radius = 100;
    
    // Очищаем область
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Вычисляем общую сумму
    const total = data.reduce((sum, value) => sum + value, 0);
    
    let startAngle = 0;
    
    for (let i = 0; i < data.length; i++) {
        const sliceAngle = (data[i] / total) * (Math.PI * 2);
        
        // Рисуем сегмент
        ctx.beginPath();
        ctx.moveTo(centerX, centerY);
        ctx.arc(centerX, centerY, radius, startAngle, startAngle + sliceAngle);
        ctx.closePath();
        
        ctx.fillStyle = colors[i] || `hsl(${i * 60}, 70%, 60%)`;
        ctx.fill();
        
        ctx.strokeStyle = '#fff';
        ctx.lineWidth = 2;
        ctx.stroke();
        
        // Добавляем подпись
        const labelAngle = startAngle + sliceAngle / 2;
        const labelX = centerX + (radius + 20) * Math.cos(labelAngle);
        const labelY = centerY + (radius + 20) * Math.sin(labelAngle);
        
        ctx.fillStyle = '#333';
        ctx.font = '12px Arial';
        ctx.textAlign = 'center';
        ctx.fillText(labels[i], labelX, labelY);
        
        startAngle += sliceAngle;
    }
    
    // Заголовок
    ctx.font = '16px Arial';
    ctx.textAlign = 'center';
    ctx.fillText('Круговая диаграмма', centerX, centerY - radius - 30);
}

// Пример использования
const barData = [30, 50, 40, 70, 60];
const barLabels = ['Янв', 'Фев', 'Мар', 'Апр', 'Май'];
const barColors = ['#3498db', '#2ecc71', '#e74c3c', '#f39c12', '#9b59b6'];

// drawBarChart(barData, barLabels, barColors);

const pieData = [30, 25, 20, 15, 10];
const pieLabels = ['A', 'B', 'C', 'D', 'E'];
const pieColors = ['#3498db', '#2ecc71', '#e74c3c', '#f39c12', '#9b59b6'];

// drawPieChart(pieData, pieLabels, pieColors);
```

## 7. Интерактивные элементы

```javascript
// Интерактивные фигуры с обработкой событий
class InteractiveCanvas {
    constructor(canvasId) {
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.shapes = [];
        this.selectedShape = null;
        
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        this.canvas.addEventListener('click', (e) => this.handleClick(e));
        this.canvas.addEventListener('mousemove', (e) => this.handleMouseMove(e));
    }
    
    handleClick(e) {
        const rect = this.canvas.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;
        
        // Проверяем, кликнули ли на фигуру
        for (let i = this.shapes.length - 1; i >= 0; i--) {
            const shape = this.shapes[i];
            if (this.isPointInShape(x, y, shape)) {
                this.selectedShape = shape;
                shape.color = '#e74c3c'; // меняем цвет при выборе
                this.draw();
                return;
            }
        }
        
        // Если не кликнули на фигуру, сбрасываем выбор
        if (this.selectedShape) {
            this.selectedShape.color = this.originalColor;
            this.selectedShape = null;
            this.draw();
        }
    }
    
    handleMouseMove(e) {
        const rect = this.canvas.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;
        
        // Проверяем, навели ли на фигуру
        for (const shape of this.shapes) {
            if (this.isPointInShape(x, y, shape)) {
                this.canvas.style.cursor = 'pointer';
                return;
            }
        }
        this.canvas.style.cursor = 'default';
    }
    
    isPointInShape(x, y, shape) {
        if (shape.type === 'rectangle') {
            return x >= shape.x && x <= shape.x + shape.width && 
                   y >= shape.y && y <= shape.y + shape.height;
        } else if (shape.type === 'circle') {
            const distance = Math.sqrt(Math.pow(x - shape.x, 2) + Math.pow(y - shape.y, 2));
            return distance <= shape.radius;
        }
        return false;
    }
    
    addRectangle(x, y, width, height, color) {
        this.shapes.push({
            type: 'rectangle',
            x,
            y,
            width,
            height,
            color,
            originalColor: color
        });
    }
    
    addCircle(x, y, radius, color) {
        this.shapes.push({
            type: 'circle',
            x,
            y,
            radius,
            color,
            originalColor: color
        });
    }
    
    draw() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        for (const shape of this.shapes) {
            this.ctx.fillStyle = shape.color;
            
            if (shape.type === 'rectangle') {
                this.ctx.fillRect(shape.x, shape.y, shape.width, shape.height);
            } else if (shape.type === 'circle') {
                this.ctx.beginPath();
                this.ctx.arc(shape.x, shape.y, shape.radius, 0, Math.PI * 2);
                this.ctx.fill();
            }
        }
    }
    
    clear() {
        this.shapes = [];
        this.selectedShape = null;
        this.draw();
    }
}

// Использование интерактивного холста
const interactiveCanvas = new InteractiveCanvas('myCanvas');
interactiveCanvas.addRectangle(50, 50, 100, 50, '#3498db');
interactiveCanvas.addCircle(200, 75, 30, '#2ecc71');
interactiveCanvas.draw();
```

## 8. Работа с фильтрами и эффектами

```javascript
// Пример использования теней и композитных операций
function drawWithEffects() {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    // Очищаем холст
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // Рисуем с тенями
    ctx.shadowColor = 'rgba(0, 0, 0, 0.5)';
    ctx.shadowBlur = 10;
    ctx.shadowOffsetX = 5;
    ctx.shadowOffsetY = 5;
    
    ctx.fillStyle = '#3498db';
    ctx.fillRect(50, 50, 100, 100);
    
    // Сбрасываем тени
    ctx.shadowColor = 'transparent';
    ctx.shadowBlur = 0;
    ctx.shadowOffsetX = 0;
    ctx.shadowOffsetY = 0;
    
    // Рисуем с глобальной альфой
    ctx.globalAlpha = 0.7;
    ctx.fillStyle = '#e74c3c';
    ctx.fillRect(170, 50, 100, 100);
    ctx.globalAlpha = 1.0;
    
    // Пример композитных операций
    ctx.fillStyle = '#f39c12';
    ctx.fillRect(300, 50, 100, 100);
    
    ctx.globalCompositeOperation = 'multiply';
    ctx.fillStyle = '#9b59b6';
    ctx.beginPath();
    ctx.arc(350, 100, 50, 0, Math.PI * 2);
    ctx.fill();
    
    // Сбрасываем композитную операцию
    ctx.globalCompositeOperation = 'source-over';
    
    // Рисуем с заливкой градиентом
    const gradient = ctx.createLinearGradient(450, 50, 550, 150);
    gradient.addColorStop(0, '#2ecc71');
    gradient.addColorStop(1, '#1abc9c');
    
    ctx.fillStyle = gradient;
    ctx.fillRect(450, 50, 100, 100);
}

// Пример фильтрации пикселей (уменьшение яркости)
function applyPixelFilter() {
    const canvas = document.getElementById('myCanvas');
    const ctx = canvas.getContext('2d');
    
    // Рисуем что-то для примера
    const gradient = ctx.createLinearGradient(0, 0, canvas.width, 0);
    gradient.addColorStop(0, '#ff0000');
    gradient.addColorStop(0.5, '#00ff00');
    gradient.addColorStop(1, '#0000ff');
    
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 200, canvas.width, 100);
    
    // Получаем пиксельные данные
    const imageData = ctx.getImageData(0, 200, canvas.width, 100);
    const data = imageData.data;
    
    // Применяем фильтр (уменьшаем яркость)
    for (let i = 0; i < data.length; i += 4) {
        data[i] = data[i] * 0.7;     // R
        data[i + 1] = data[i + 1] * 0.7; // G
        data[i + 2] = data[i + 2] * 0.7; // B
    }
    
    // Возвращаем измененные данные на холст
    ctx.putImageData(imageData, 0, 200);
}

// Вызов функций
drawWithEffects();
applyPixelFilter();
```

Эти примеры демонстрируют различные возможности Canvas API для создания графики, анимаций и интерактивных элементов в веб-приложениях.

См. также:
- [[js/practical-examples/web-api-examples]] - Работа с Web API
- [[js/practical-examples/web-workers-examples]] - Примеры использования Web Workers
- [[js/practical-examples/fetch-websocket-examples]] - Примеры с Fetch API и WebSocket
# Canvas 2D API

Canvas 2D API предоставляет возможность рисовать 2D графику через элемент `<canvas>` в браузере. Это мощный инструмент для создания визуализаций, игр, редакторов изображений и других графических приложений.

## Основы Canvas

### Создание и настройка Canvas

```html
<!-- Базовая HTML структура -->
<canvas id="myCanvas" width="800" height="600">
    Ваш браузер не поддерживает элемент canvas.
</canvas>
```

```javascript
// Получение контекста 2D
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// Проверка поддержки
if (!ctx) {
    console.error('Canvas 2D API не поддерживается');
}

// Основные свойства canvas
console.log('Ширина:', canvas.width);
console.log('Высота:', canvas.height);
console.log('Контекст:', ctx);
```

### Основные свойства стиля

```javascript
// Цвет заливки
ctx.fillStyle = '#FF0000';        // Красный
ctx.fillStyle = 'rgb(255, 0, 0)'; // Красный в RGB
ctx.fillStyle = 'rgba(255, 0, 0, 0.5)'; // Красный с прозрачностью
ctx.fillStyle = 'hsl(0, 100%, 50%)'; // Красный в HSL

// Цвет обводки
ctx.strokeStyle = '#0000FF';       // Синий
ctx.strokeStyle = 'green';         // Зеленый

// Ширина линии
ctx.lineWidth = 5;

// Стиль соединения линий
ctx.lineJoin = 'round';   // 'round', 'bevel', 'miter'
ctx.lineCap = 'round';    // 'round', 'butt', 'square'

// Прозрачность
ctx.globalAlpha = 0.5;    // 50% прозрачность
```

## Основные фигуры

### Прямоугольники

```javascript
// Заливка прямоугольника
ctx.fillStyle = '#FF0000';
ctx.fillRect(10, 10, 100, 50); // x, y, ширина, высота

// Обводка прямоугольника
ctx.strokeStyle = '#0000FF';
ctx.lineWidth = 2;
ctx.strokeRect(120, 10, 100, 50);

// Очистка прямоугольной области
ctx.clearRect(230, 10, 100, 50);

// Рисование скругленного прямоугольника (вручную)
function drawRoundedRect(ctx, x, y, width, height, radius) {
    ctx.beginPath();
    ctx.moveTo(x + radius, y);
    ctx.lineTo(x + width - radius, y);
    ctx.quadraticCurveTo(x + width, y, x + width, y + radius);
    ctx.lineTo(x + width, y + height - radius);
    ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
    ctx.lineTo(x + radius, y + height);
    ctx.quadraticCurveTo(x, y + height, x, y + height - radius);
    ctx.lineTo(x, y + radius);
    ctx.quadraticCurveTo(x, y, x + radius, y);
    ctx.closePath();
    ctx.fill();
}

drawRoundedRect(ctx, 340, 10, 100, 50, 10);
```

### Пути и кривые

```javascript
// Рисование линии
ctx.beginPath();
ctx.moveTo(10, 100);      // Начальная точка
ctx.lineTo(100, 100);     // Конечная точка
ctx.stroke();             // Отобразить линию

// Рисование треугольника
ctx.beginPath();
ctx.moveTo(10, 150);
ctx.lineTo(50, 100);
ctx.lineTo(90, 150);
ctx.closePath();          // Замыкает путь
ctx.fillStyle = 'yellow';
ctx.fill();
ctx.strokeStyle = 'black';
ctx.stroke();

// Квадратичная кривая Безье
ctx.beginPath();
ctx.moveTo(10, 200);
ctx.quadraticCurveTo(55, 150, 100, 200); // контрольная точка, конечная точка
ctx.stroke();

// Кубическая кривая Безье
ctx.beginPath();
ctx.moveTo(10, 250);
ctx.bezierCurveTo(30, 200, 70, 300, 100, 250); // 2 контрольные точки, конечная точка
ctx.stroke();

// Дуги и окружности
ctx.beginPath();
ctx.arc(200, 100, 30, 0, Math.PI * 2); // центрX, центрY, радиус, начальный угол, конечный угол
ctx.fillStyle = 'blue';
ctx.fill();

// Дуга (не полный круг)
ctx.beginPath();
ctx.arc(250, 100, 30, 0, Math.PI); // Полуокружность
ctx.stroke();

// Сектор (с заливкой)
ctx.beginPath();
ctx.moveTo(300, 100); // Центр
ctx.arc(300, 100, 30, 0, Math.PI / 2); // Четверть круга
ctx.closePath();
ctx.fillStyle = 'green';
ctx.fill();
```

### Текст

```javascript
// Свойства текста
ctx.font = '20px Arial';
ctx.textAlign = 'left';      // 'left', 'right', 'center'
ctx.textBaseline = 'top';    // 'top', 'middle', 'bottom', 'alphabetic'

// Заливка текста
ctx.fillStyle = '#000000';
ctx.fillText('Привет, Canvas!', 10, 300);

// Обводка текста
ctx.strokeStyle = '#FF0000';
ctx.lineWidth = 1;
ctx.strokeText('Обведенный текст', 10, 330);

// Измерение текста
const textMetrics = ctx.measureText('Привет, Canvas!');
console.log('Ширина текста:', textMetrics.width);
console.log('Высота текста:', textMetrics.fontBoundingBoxAscent + textMetrics.fontBoundingBoxDescent);

// Центрирование текста
function drawCenteredText(text, x, y) {
    const width = ctx.measureText(text).width;
    ctx.fillText(text, x - width / 2, y);
}

drawCenteredText('Центрированный текст', 200, 360);
```

## Паттерны и градиенты

### Градиенты

```javascript
// Линейный градиент
const linearGradient = ctx.createLinearGradient(0, 0, 200, 0);
linearGradient.addColorStop(0, 'red');
linearGradient.addColorStop(0.5, 'yellow');
linearGradient.addColorStop(1, 'blue');

ctx.fillStyle = linearGradient;
ctx.fillRect(10, 400, 200, 50);

// Радиальный градиент
const radialGradient = ctx.createRadialGradient(100, 475, 0, 100, 475, 50);
radialGradient.addColorStop(0, 'white');
radialGradient.addColorStop(1, 'black');

ctx.fillStyle = radialGradient;
ctx.fillRect(10, 475, 200, 50);

// Использование градиента для круга
ctx.beginPath();
ctx.arc(300, 450, 40, 0, Math.PI * 2);
ctx.fillStyle = linearGradient;
ctx.fill();
```

### Паттерны

```javascript
// Создание паттерна из изображения
function createPattern(image) {
    // Создаем временный canvas для паттерна
    const patternCanvas = document.createElement('canvas');
    patternCanvas.width = 20;
    patternCanvas.height = 20;
    const patternCtx = patternCanvas.getContext('2d');
    
    // Рисуем узор на временном canvas
    patternCtx.fillStyle = '#FF0000';
    patternCtx.fillRect(0, 0, 10, 10);
    patternCtx.fillStyle = '#0000FF';
    patternCtx.fillRect(10, 10, 10, 10);
    
    // Создаем паттерн
    const pattern = ctx.createPattern(patternCanvas, 'repeat');
    return pattern;
}

const pattern = createPattern();
ctx.fillStyle = pattern;
ctx.fillRect(220, 400, 200, 100);
```

## Работа с изображениями

### Загрузка и отображение изображений

```javascript
// Отображение изображения
function drawImageExample() {
    const img = new Image();
    img.src = 'https://example.com/image.png';
    
    img.onload = function() {
        // Полное изображение
        ctx.drawImage(img, 10, 550);
        
        // Масштабированное изображение
        ctx.drawImage(img, 10, 600, 100, 50);
        
        // Вырезка части изображения
        ctx.drawImage(img, 0, 0, 100, 100, 120, 600, 50, 50); // исходное x, y, ширина, высота, целевое x, y, ширина, высота
    };
}

// Использование другого canvas как источника
function createSourceCanvas() {
    const sourceCanvas = document.createElement('canvas');
    sourceCanvas.width = 100;
    sourceCanvas.height = 100;
    const sourceCtx = sourceCanvas.getContext('2d');
    
    // Рисуем на исходном canvas
    sourceCtx.fillStyle = 'red';
    sourceCtx.fillRect(0, 0, 50, 50);
    sourceCtx.fillStyle = 'blue';
    sourceCtx.fillRect(50, 50, 50, 50);
    
    // Используем как источник
    ctx.drawImage(sourceCanvas, 250, 550);
    
    return sourceCanvas;
}
```

### Манипуляции с пикселями

```javascript
// Получение и изменение пикселей
function manipulatePixels() {
    // Рисуем что-то для манипуляции
    ctx.fillStyle = 'red';
    ctx.fillRect(400, 550, 100, 100);
    
    // Получаем пиксельные данные
    const imageData = ctx.getImageData(400, 550, 100, 100);
    const data = imageData.data;
    
    // Манипуляции с пикселями (например, сепия-эффект)
    for (let i = 0; i < data.length; i += 4) {
        const r = data[i];
        const g = data[i + 1];
        const b = data[i + 2];
        
        // Сепия-эффект
        const avg = 0.299 * r + 0.587 * g + 0.114 * b;
        data[i] = avg * 1.2;     // Red
        data[i + 1] = avg * 0.9; // Green
        data[i + 2] = avg * 0.7; // Blue
        // data[i + 3] остается без изменений (alpha)
    }
    
    // Возвращаем измененные данные на canvas
    ctx.putImageData(imageData, 400, 650);
}

manipulatePixels();
```

## Трансформации

### Смещение, вращение и масштабирование

```javascript
// Сохранение и восстановление состояния
ctx.save(); // Сохраняет текущее состояние

// Смещение (перемещение системы координат)
ctx.translate(200, 200);
ctx.fillStyle = 'green';
ctx.fillRect(0, 0, 50, 50);

// Вращение
ctx.rotate(Math.PI / 4); // 45 градусов
ctx.fillStyle = 'blue';
ctx.fillRect(0, 0, 50, 25);

// Масштабирование
ctx.scale(1.5, 1.5);
ctx.fillStyle = 'red';
ctx.fillRect(0, 0, 50, 25);

// Восстановление предыдущего состояния
ctx.restore();

// Нарисуем квадрат в оригинальной системе координат
ctx.fillStyle = 'purple';
ctx.fillRect(10, 10, 50, 50);
```

### Комплексные трансформации

```javascript
// Функция для рисования вращающегося прямоугольника
function drawRotatingRect(x, y, width, height, angle, color) {
    ctx.save();
    ctx.translate(x, y);
    ctx.rotate(angle);
    
    ctx.fillStyle = color;
    ctx.fillRect(-width/2, -height/2, width, height);
    
    ctx.restore();
}

// Анимация вращающихся прямоугольников
function animateRotatingRects() {
    let angle = 0;
    
    function animate() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        drawRotatingRect(400, 100, 100, 50, angle, 'red');
        drawRotatingRect(500, 100, 80, 40, -angle, 'blue');
        drawRotatingRect(450, 180, 60, 30, angle * 2, 'green');
        
        angle += 0.05;
        requestAnimationFrame(animate);
    }
    
    animate();
}
```

## События и взаимодействие

### Обработка событий мыши

```javascript
// Класс для обработки событий на canvas
class CanvasEventHandler {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.isDrawing = false;
        this.lastX = 0;
        this.lastY = 0;
        
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        // Мышь
        this.canvas.addEventListener('mousedown', this.handleMouseDown.bind(this));
        this.canvas.addEventListener('mousemove', this.handleMouseMove.bind(this));
        this.canvas.addEventListener('mouseup', this.handleMouseUp.bind(this));
        this.canvas.addEventListener('mouseout', this.handleMouseUp.bind(this));
        
        // Касания (для мобильных устройств)
        this.canvas.addEventListener('touchstart', this.handleTouchStart.bind(this));
        this.canvas.addEventListener('touchmove', this.handleTouchMove.bind(this));
        this.canvas.addEventListener('touchend', this.handleTouchEnd.bind(this));
    }
    
    handleMouseDown(e) {
        this.isDrawing = true;
        [this.lastX, this.lastY] = this.getMousePos(e);
    }
    
    handleMouseMove(e) {
        if (!this.isDrawing) return;
        
        const [currentX, currentY] = this.getMousePos(e);
        
        this.ctx.beginPath();
        this.ctx.moveTo(this.lastX, this.lastY);
        this.ctx.lineTo(currentX, currentY);
        this.ctx.strokeStyle = '#000000';
        this.ctx.lineWidth = 2;
        this.ctx.lineCap = 'round';
        this.ctx.stroke();
        
        [this.lastX, this.lastY] = [currentX, currentY];
    }
    
    handleMouseUp() {
        this.isDrawing = false;
    }
    
    handleTouchStart(e) {
        e.preventDefault();
        this.isDrawing = true;
        [this.lastX, this.lastY] = this.getTouchPos(e);
    }
    
    handleTouchMove(e) {
        e.preventDefault();
        if (!this.isDrawing) return;
        
        const [currentX, currentY] = this.getTouchPos(e);
        
        this.ctx.beginPath();
        this.ctx.moveTo(this.lastX, this.lastY);
        this.ctx.lineTo(currentX, currentY);
        this.ctx.strokeStyle = '#000000';
        this.ctx.lineWidth = 2;
        this.ctx.lineCap = 'round';
        this.ctx.stroke();
        
        [this.lastX, this.lastY] = [currentX, currentY];
    }
    
    handleTouchEnd() {
        this.isDrawing = false;
    }
    
    getMousePos(e) {
        const rect = this.canvas.getBoundingClientRect();
        return [
            e.clientX - rect.left,
            e.clientY - rect.top
        ];
    }
    
    getTouchPos(e) {
        const rect = this.canvas.getBoundingClientRect();
        const touch = e.touches[0];
        return [
            touch.clientX - rect.left,
            touch.clientY - rect.top
        ];
    }
}

// Использование
const canvasHandler = new CanvasEventHandler(canvas);
```

## Комплексные примеры

### Рисование графиков

```javascript
// Класс для рисования линейных графиков
class LineChart {
    constructor(canvas, data, options = {}) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.data = data;
        this.options = {
            margin: { top: 20, right: 20, bottom: 40, left: 50 },
            colors: ['#FF6384', '#36A2EB', '#FFCE56'],
            ...options
        };
        
        this.render();
    }
    
    render() {
        this.clear();
        this.drawAxes();
        this.drawGrid();
        this.drawLines();
        this.drawLabels();
    }
    
    clear() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    }
    
    getChartArea() {
        const width = this.canvas.width - this.options.margin.left - this.options.margin.right;
        const height = this.canvas.height - this.options.margin.top - this.options.margin.bottom;
        
        return {
            x: this.options.margin.left,
            y: this.options.margin.top,
            width: width,
            height: height
        };
    }
    
    drawAxes() {
        const chart = this.getChartArea();
        
        // X ось
        this.ctx.beginPath();
        this.ctx.moveTo(chart.x, chart.y + chart.height);
        this.ctx.lineTo(chart.x + chart.width, chart.y + chart.height);
        this.ctx.stroke();
        
        // Y ось
        this.ctx.beginPath();
        this.ctx.moveTo(chart.x, chart.y);
        this.ctx.lineTo(chart.x, chart.y + chart.height);
        this.ctx.stroke();
    }
    
    drawGrid() {
        const chart = this.getChartArea();
        const data = this.data[0]; // Берем первый набор данных для масштабирования
        
        // Находим минимум и максимум
        const values = data.values;
        const minValue = Math.min(...values);
        const maxValue = Math.max(...values);
        const range = maxValue - minValue || 1;
        
        // Горизонтальные линии сетки
        for (let i = 0; i <= 5; i++) {
            const y = chart.y + chart.height - (i / 5) * chart.height;
            
            this.ctx.beginPath();
            this.ctx.moveTo(chart.x, y);
            this.ctx.lineTo(chart.x + chart.width, y);
            this.ctx.strokeStyle = '#e0e0e0';
            this.ctx.lineWidth = 1;
            this.ctx.stroke();
            
            // Метки на Y оси
            this.ctx.fillStyle = '#000';
            this.ctx.font = '12px Arial';
            this.ctx.textAlign = 'right';
            this.ctx.textBaseline = 'middle';
            this.ctx.fillText(
                Math.round(minValue + (i / 5) * range).toString(),
                chart.x - 5,
                y
            );
        }
    }
    
    drawLines() {
        if (!this.data || this.data.length === 0) return;
        
        const chart = this.getChartArea();
        
        this.data.forEach((dataset, datasetIndex) => {
            if (!dataset.values || dataset.values.length === 0) return;
            
            const values = dataset.values;
            const minValue = Math.min(...values);
            const maxValue = Math.max(...values);
            const range = maxValue - minValue || 1;
            
            this.ctx.beginPath();
            this.ctx.strokeStyle = this.options.colors[datasetIndex % this.options.colors.length];
            this.ctx.lineWidth = 2;
            
            values.forEach((value, index) => {
                const x = chart.x + (index / (values.length - 1)) * chart.width;
                const y = chart.y + chart.height - ((value - minValue) / range) * chart.height;
                
                if (index === 0) {
                    this.ctx.moveTo(x, y);
                } else {
                    this.ctx.lineTo(x, y);
                }
            });
            
            this.ctx.stroke();
        });
    }
    
    drawLabels() {
        if (!this.data || this.data.length === 0) return;
        
        const chart = this.getChartArea();
        const values = this.data[0].values;
        
        // Метки на X оси
        this.ctx.fillStyle = '#000';
        this.ctx.font = '12px Arial';
        this.ctx.textAlign = 'center';
        this.ctx.textBaseline = 'top';
        
        values.forEach((_, index) => {
            if (index % Math.ceil(values.length / 10) === 0) { // Показываем каждую 10-ю метку
                const x = chart.x + (index / (values.length - 1)) * chart.width;
                this.ctx.fillText(index.toString(), x, chart.y + chart.height + 5);
            }
        });
    }
}

// Использование
const chartData = [
    {
        label: 'Данные 1',
        values: [10, 20, 15, 25, 30, 22, 18, 28, 35, 32]
    }
];

const chart = new LineChart(canvas, chartData);
```

### Создание простого редактора

```javascript
// Простой редактор изображений на Canvas
class ImageEditor {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.originalImage = null;
        this.currentImage = null;
        this.tools = {
            brush: 'brush',
            eraser: 'eraser',
            text: 'text',
            shape: 'shape'
        };
        this.currentTool = this.tools.brush;
        this.brushSize = 5;
        this.brushColor = '#000000';
        
        this.setupEventHandlers();
    }
    
    setupEventHandlers() {
        this.canvas.addEventListener('mousedown', this.handleMouseDown.bind(this));
        this.canvas.addEventListener('mousemove', this.handleMouseMove.bind(this));
        this.canvas.addEventListener('mouseup', this.handleMouseUp.bind(this));
    }
    
    loadImage(imageSrc) {
        const img = new Image();
        img.onload = () => {
            this.originalImage = img;
            this.currentImage = img;
            this.drawImage();
        };
        img.src = imageSrc;
    }
    
    drawImage() {
        if (this.currentImage) {
            // Очищаем canvas
            this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
            // Рисуем изображение
            this.ctx.drawImage(this.currentImage, 0, 0, this.canvas.width, this.canvas.height);
        }
    }
    
    applyFilter(filterType) {
        if (!this.currentImage) return;
        
        // Создаем временный canvas для манипуляций
        const tempCanvas = document.createElement('canvas');
        tempCanvas.width = this.canvas.width;
        tempCanvas.height = this.canvas.height;
        const tempCtx = tempCanvas.getContext('2d');
        
        // Рисуем текущее изображение на временном canvas
        tempCtx.drawImage(this.canvas, 0, 0);
        
        // Получаем пиксельные данные
        const imageData = tempCtx.getImageData(0, 0, tempCanvas.width, tempCanvas.height);
        const data = imageData.data;
        
        // Применяем фильтр
        switch (filterType) {
            case 'grayscale':
                for (let i = 0; i < data.length; i += 4) {
                    const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
                    data[i] = avg;     // R
                    data[i + 1] = avg; // G
                    data[i + 2] = avg; // B
                }
                break;
                
            case 'invert':
                for (let i = 0; i < data.length; i += 4) {
                    data[i] = 255 - data[i];     // R
                    data[i + 1] = 255 - data[i + 1]; // G
                    data[i + 2] = 255 - data[i + 2]; // B
                }
                break;
                
            case 'sepia':
                for (let i = 0; i < data.length; i += 4) {
                    const r = data[i];
                    const g = data[i + 1];
                    const b = data[i + 2];
                    
                    data[i] = Math.min(255, (r * 0.393) + (g * 0.769) + (b * 0.189));     // R
                    data[i + 1] = Math.min(255, (r * 0.349) + (g * 0.686) + (b * 0.168)); // G
                    data[i + 2] = Math.min(255, (r * 0.272) + (g * 0.534) + (b * 0.131)); // B
                }
                break;
        }
        
        // Возвращаем измененные данные
        tempCtx.putImageData(imageData, 0, 0);
        
        // Сохраняем результат
        this.currentImage = new Image();
        this.currentImage.src = tempCanvas.toDataURL();
        this.drawImage();
    }
    
    handleMouseDown(e) {
        this.isDrawing = true;
        [this.startX, this.startY] = this.getCanvasCoordinates(e);
        
        if (this.currentTool === this.tools.brush) {
            this.ctx.beginPath();
            this.ctx.moveTo(this.startX, this.startY);
            this.ctx.strokeStyle = this.brushColor;
            this.ctx.lineWidth = this.brushSize;
            this.ctx.lineCap = 'round';
        }
    }
    
    handleMouseMove(e) {
        if (!this.isDrawing) return;
        
        const [currentX, currentY] = this.getCanvasCoordinates(e);
        
        if (this.currentTool === this.tools.brush) {
            this.ctx.lineTo(currentX, currentY);
            this.ctx.stroke();
            this.ctx.beginPath();
            this.ctx.moveTo(currentX, currentY);
        }
    }
    
    handleMouseUp() {
        this.isDrawing = false;
    }
    
    getCanvasCoordinates(e) {
        const rect = this.canvas.getBoundingClientRect();
        return [
            e.clientX - rect.left,
            e.clientY - rect.top
        ];
    }
    
    setTool(tool) {
        this.currentTool = tool;
    }
    
    setBrushSize(size) {
        this.brushSize = size;
    }
    
    setBrushColor(color) {
        this.brushColor = color;
    }
    
    clearCanvas() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    }
    
    saveImage() {
        return this.canvas.toDataURL();
    }
}

// Использование
const editor = new ImageEditor(canvas);
// editor.loadImage('path/to/image.jpg');
```

## Анимации на Canvas

### Базовая анимация

```javascript
// Класс для создания анимаций
class CanvasAnimation {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.animationId = null;
        this.objects = [];
    }
    
    addObject(object) {
        this.objects.push(object);
    }
    
    start() {
        if (this.animationId) {
            cancelAnimationFrame(this.animationId);
        }
        
        const animate = () => {
            this.update();
            this.render();
            this.animationId = requestAnimationFrame(animate);
        };
        
        animate();
    }
    
    stop() {
        if (this.animationId) {
            cancelAnimationFrame(this.animationId);
            this.animationId = null;
        }
    }
    
    update() {
        this.objects.forEach(obj => {
            if (obj.update) {
                obj.update();
            }
        });
    }
    
    render() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        this.objects.forEach(obj => {
            if (obj.render) {
                obj.render(this.ctx);
            }
        });
    }
}

// Пример объекта для анимации
class AnimatedCircle {
    constructor(x, y, radius, color) {
        this.x = x;
        this.y = y;
        this.radius = radius;
        this.color = color;
        this.vx = (Math.random() - 0.5) * 4; // Случайная скорость по X
        this.vy = (Math.random() - 0.5) * 4; // Случайная скорость по Y
        this.canvasWidth = 800;
        this.canvasHeight = 600;
    }
    
    update() {
        this.x += this.vx;
        this.y += this.vy;
        
        // Отскок от стен
        if (this.x - this.radius < 0 || this.x + this.radius > this.canvasWidth) {
            this.vx = -this.vx;
        }
        if (this.y - this.radius < 0 || this.y + this.radius > this.canvasHeight) {
            this.vy = -this.vy;
        }
    }
    
    render(ctx) {
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
        ctx.fillStyle = this.color;
        ctx.fill();
    }
}

// Использование
const animation = new CanvasAnimation(canvas);

// Добавляем несколько анимированных кругов
for (let i = 0; i < 5; i++) {
    const circle = new AnimatedCircle(
        Math.random() * 700 + 50,
        Math.random() * 500 + 50,
        10 + Math.random() * 20,
        `hsl(${Math.random() * 360}, 70%, 60%)`
    );
    animation.addObject(circle);
}

// animation.start(); // Запускаем анимацию
```

### Сложные анимации с easing функциями

```javascript
// Библиотека easing функций
const Easing = {
    linear: t => t,
    easeInQuad: t => t * t,
    easeOutQuad: t => t * (2 - t),
    easeInOutQuad: t => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t,
    easeInCubic: t => t * t * t,
    easeOutCubic: t => (--t) * t * t + 1,
    easeInOutCubic: t => t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1,
    easeInQuart: t => t * t * t * t,
    easeOutQuart: t => 1 - (--t) * t * t * t,
    easeInOutQuart: t => t < 0.5 ? 8 * t * t * t * t : 1 - 8 * (--t) * t * t * t,
    easeInQuint: t => t * t * t * t * t,
    easeOutQuint: t => 1 + (--t) * t * t * t * t
};

// Класс анимации с easing
class TweenAnimation {
    constructor(object, properties, duration, easing = Easing.easeOutQuad) {
        this.object = object;
        this.startValues = {};
        this.endValues = properties;
        this.duration = duration;
        this.easing = easing;
        this.startTime = null;
        this.active = true;
        
        // Сохраняем начальные значения
        for (const prop in properties) {
            this.startValues[prop] = object[prop];
        }
    }
    
    update(timestamp) {
        if (!this.startTime) {
            this.startTime = timestamp;
        }
        
        const elapsed = timestamp - this.startTime;
        const progress = Math.min(elapsed / this.duration, 1);
        const easedProgress = this.easing(progress);
        
        // Интерполируем значения
        for (const prop in this.endValues) {
            const start = this.startValues[prop];
            const end = this.endValues[prop];
            this.object[prop] = start + (end - start) * easedProgress;
        }
        
        return progress < 1; // Возвращаем true, если анимация не завершена
    }
}

// Класс для управления анимациями
class AnimationManager {
    constructor() {
        this.animations = [];
        this.rafId = null;
    }
    
    addAnimation(animation) {
        this.animations.push(animation);
        this.start();
    }
    
    start() {
        if (this.rafId) return;
        
        const animate = (timestamp) => {
            // Обновляем все анимации
            this.animations = this.animations.filter(anim => anim.update(timestamp));
            
            if (this.animations.length > 0) {
                this.rafId = requestAnimationFrame(animate);
            } else {
                this.rafId = null;
            }
        };
        
        this.rafId = requestAnimationFrame(animate);
    }
    
    stop() {
        if (this.rafId) {
            cancelAnimationFrame(this.rafId);
            this.rafId = null;
        }
        this.animations = [];
    }
}

// Пример использования
const animManager = new AnimationManager();

const circle = { x: 50, y: 50, radius: 20, color: '#FF0000' };

// Анимация перемещения и изменения размера
const moveAnimation = new TweenAnimation(
    circle,
    { x: 700, y: 500, radius: 50 },
    2000, // 2 секунды
    Easing.easeOutBounce
);

animManager.addAnimation(moveAnimation);
```

## Лучшие практики

### 1. Оптимизация производительности

```javascript
// Класс для оптимизированной отрисовки
class OptimizedRenderer {
    constructor(canvas) {
        this.canvas = canvas;
        this.ctx = canvas.getContext('2d');
        this.offscreenCanvas = new OffscreenCanvas(canvas.width, canvas.height);
        this.offscreenCtx = this.offscreenCanvas.getContext('2d');
        this.animationId = null;
        this.objects = [];
        this.dirty = true;
    }
    
    addObject(object) {
        this.objects.push(object);
        this.dirty = true;
    }
    
    start() {
        if (this.animationId) {
            cancelAnimationFrame(this.animationId);
        }
        
        const render = () => {
            if (this.dirty) {
                this.renderToOffscreen();
                this.drawToScreen();
                this.dirty = false;
            }
            this.animationId = requestAnimationFrame(render);
        };
        
        render();
    }
    
    renderToOffscreen() {
        this.offscreenCtx.clearRect(0, 0, this.offscreenCanvas.width, this.offscreenCanvas.height);
        
        // Рендерим все объекты на offscreen canvas
        this.objects.forEach(obj => {
            if (obj.render) {
                obj.render(this.offscreenCtx);
            }
        });
    }
    
    drawToScreen() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        this.ctx.drawImage(this.offscreenCanvas, 0, 0);
    }
    
    markDirty() {
        this.dirty = true;
    }
}
```

### 2. Работа с памятью

```javascript
// Класс для управления ресурсами
class ResourceManager {
    constructor() {
        this.images = new Map();
        this.patterns = new Map();
        this.gradients = new Map();
    }
    
    async loadImage(src) {
        if (this.images.has(src)) {
            return this.images.get(src);
        }
        
        return new Promise((resolve, reject) => {
            const img = new Image();
            img.onload = () => {
                this.images.set(src, img);
                resolve(img);
            };
            img.onerror = reject;
            img.src = src;
        });
    }
    
    createPattern(image, repetition = 'repeat') {
        const key = `${image.src}-${repetition}`;
        
        if (this.patterns.has(key)) {
            return this.patterns.get(key);
        }
        
        const pattern = ctx.createPattern(image, repetition);
        this.patterns.set(key, pattern);
        return pattern;
    }
    
    clear() {
        this.images.clear();
        this.patterns.clear();
        this.gradients.clear();
    }
}

// Использование
const resourceManager = new ResourceManager();
// const image = await resourceManager.loadImage('path/to/image.png');
```

Canvas 2D API предоставляет мощные возможности для создания сложной 2D графики в браузере. При правильном использовании он может создавать впечатляющие визуализации, интерактивные элементы и даже простые игры с хорошей производительностью.

#javascript #canvas #2d-graphics #frontend #web-development #graphics
# HTML Мультимедиа: Canvas и SVG

Canvas и SVG - это два мощных способа работы с векторной графикой и анимациями в HTML. Они позволяют создавать сложные визуальные эффекты, интерактивные диаграммы, игры и другие графические элементы.

## Canvas API

Canvas предоставляет элемент `<canvas>` и API для рисования 2D графики с помощью JavaScript.

### Основы Canvas

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Canvas API</title>
</head>
<body>
    <h1>Canvas API</h1>
    
    <!-- Canvas элемент -->
    <canvas id="myCanvas" width="400" height="300">
        Ваш браузер не поддерживает Canvas.
    </canvas>
    
    <script>
        // Получение контекста рисования
        const canvas = document.getElementById('myCanvas');
        const ctx = canvas.getContext('2d');
        
        // Основные свойства стиля
        ctx.fillStyle = '#FF6B6B';  // Цвет заливки
        ctx.strokeStyle = '#4ECDC4';  // Цвет обводки
        ctx.lineWidth = 2;  // Толщина линии
        
        // Рисование прямоугольника
        ctx.fillRect(50, 50, 100, 80);  // Залитый прямоугольник
        ctx.strokeRect(50, 150, 100, 80);  // Обведенный прямоугольник
    </script>
</body>
</html>
```

### Рисование фигур

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Рисование фигур в Canvas</title>
    <style>
        canvas {
            border: 1px solid #ddd;
            margin: 10px;
        }
    </style>
</head>
<body>
    <h1>Рисование фигур в Canvas</h1>
    
    <canvas id="shapesCanvas" width="500" height="400"></canvas>
    
    <script>
        const canvas = document.getElementById('shapesCanvas');
        const ctx = canvas.getContext('2d');
        
        // Рисование линии
        ctx.beginPath();
        ctx.moveTo(50, 50);
        ctx.lineTo(150, 100);
        ctx.strokeStyle = '#333';
        ctx.lineWidth = 2;
        ctx.stroke();
        
        // Рисование треугольника
        ctx.beginPath();
        ctx.moveTo(200, 50);
        ctx.lineTo(250, 150);
        ctx.lineTo(150, 150);
        ctx.closePath();
        ctx.fillStyle = 'rgba(255, 0, 0, 0.5)';
        ctx.fill();
        ctx.strokeStyle = '#000';
        ctx.stroke();
        
        // Рисование круга
        ctx.beginPath();
        ctx.arc(350, 100, 50, 0, 2 * Math.PI);
        ctx.fillStyle = 'rgba(0, 255, 0, 0.5)';
        ctx.fill();
        ctx.strokeStyle = '#000';
        ctx.lineWidth = 3;
        ctx.stroke();
        
        // Рисование дуги
        ctx.beginPath();
        ctx.arc(100, 250, 40, 0, Math.PI, false);
        ctx.strokeStyle = '#0000FF';
        ctx.lineWidth = 4;
        ctx.stroke();
        
        // Рисование кривой Безье
        ctx.beginPath();
        ctx.moveTo(200, 250);
        ctx.bezierCurveTo(250, 200, 350, 300, 400, 250);
        ctx.strokeStyle = '#FF00FF';
        ctx.lineWidth = 3;
        ctx.stroke();
    </script>
</body>
</html>
```

### Работа с текстом в Canvas

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Текст в Canvas</title>
    <style>
        canvas {
            border: 1px solid #ddd;
            margin: 10px;
        }
    </style>
</head>
<body>
    <h1>Текст в Canvas</h1>
    
    <canvas id="textCanvas" width="600" height="300"></canvas>
    
    <script>
        const canvas = document.getElementById('textCanvas');
        const ctx = canvas.getContext('2d');
        
        // Настройка текста
        ctx.font = '24px Arial';
        ctx.fillStyle = '#333';
        ctx.textAlign = 'left';
        ctx.textBaseline = 'top';
        
        // Рисование текста
        ctx.fillText('Простой текст', 50, 50);
        ctx.strokeText('Обведенный текст', 50, 100);
        
        // Центрированный текст
        ctx.textAlign = 'center';
        ctx.font = 'bold 32px Georgia';
        ctx.fillStyle = '#007acc';
        ctx.fillText('Центрированный текст', 300, 150);
        
        // Измерение текста
        const textMetrics = ctx.measureText('Измеренный текст');
        ctx.fillText(`Измеренный текст (ширина: ${textMetrics.width}px)`, 300, 200);
        
        // Рисование текста вдоль кривой
        ctx.save();
        ctx.translate(100, 250);
        ctx.rotate(-Math.PI / 4);
        ctx.font = 'italic 20px Times New Roman';
        ctx.fillStyle = '#FF6B6B';
        ctx.fillText('Повернутый текст', 0, 0);
        ctx.restore();
    </script>
</body>
</html>
```

### Анимация в Canvas

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Анимация в Canvas</title>
    <style>
        canvas {
            border: 1px solid #ddd;
            margin: 10px;
        }
    </style>
</head>
<body>
    <h1>Анимация в Canvas</h1>
    
    <canvas id="animationCanvas" width="600" height="400"></canvas>
    
    <script>
        const canvas = document.getElementById('animationCanvas');
        const ctx = canvas.getContext('2d');
        
        // Параметры анимации
        let x = 50;
        let y = 50;
        let dx = 2;
        let dy = 2;
        let radius = 20;
        
        // Функция анимации
        function animate() {
            // Очистка холста
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Рисование движущегося круга
            ctx.beginPath();
            ctx.arc(x, y, radius, 0, Math.PI * 2);
            ctx.fillStyle = '#FF6B6B';
            ctx.fill();
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 2;
            ctx.stroke();
            
            // Обновление позиции
            x += dx;
            y += dy;
            
            // Проверка столкновений со стенками
            if (x + radius > canvas.width || x - radius < 0) {
                dx = -dx;
            }
            if (y + radius > canvas.height || y - radius < 0) {
                dy = -dy;
            }
            
            // Запуск следующего кадра
            requestAnimationFrame(animate);
        }
        
        // Запуск анимации
        animate();
    </script>
</body>
</html>
```

## SVG (Scalable Vector Graphics)

SVG - это XML-основанный язык разметки для описания двумерной векторной графики.

### Основы SVG

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>SVG основы</title>
</head>
<body>
    <h1>SVG основы</h1>
    
    <!-- Встроенный SVG -->
    <svg width="300" height="200" xmlns="http://www.w3.org/2000/svg">
        <!-- Прямоугольник -->
        <rect x="10" y="10" width="100" height="50" fill="#FF6B6B" stroke="#333" stroke-width="2" />
        
        <!-- Круг -->
        <circle cx="200" cy="50" r="30" fill="#4ECDC4" stroke="#333" stroke-width="2" />
        
        <!-- Линия -->
        <line x1="10" y1="100" x2="290" y2="100" stroke="#333" stroke-width="2" />
        
        <!-- Текст -->
        <text x="150" y="150" font-family="Arial" font-size="16" text-anchor="middle" fill="#333">
            SVG текст
        </text>
    </svg>
</body>
</html>
```

### Сложные SVG фигуры

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Сложные SVG фигуры</title>
    <style>
        svg {
            border: 1px solid #ddd;
            margin: 10px;
        }
    </style>
</head>
<body>
    <h1>Сложные SVG фигуры</h1>
    
    <svg width="500" height="400" xmlns="http://www.w3.org/2000/svg">
        <!-- Путь (path) -->
        <path d="M 50 50 L 150 50 L 100 150 Z" fill="#FF6B6B" stroke="#333" stroke-width="2" />
        
        <!-- Эллипс -->
        <ellipse cx="300" cy="80" rx="50" ry="30" fill="#4ECDC4" stroke="#333" stroke-width="2" />
        
        <!-- Полигон -->
        <polygon points="50,200 100,150 150,200 125,250 75,250" fill="#45B7D1" stroke="#333" stroke-width="2" />
        
        <!-- Полилиния -->
        <polyline points="200,200 250,180 300,200 350,180 400,200" 
                  fill="none" stroke="#FF9F43" stroke-width="3" />
        
        <!-- Дуга -->
        <path d="M 200 300 A 50 50 0 0 1 250 350" 
              fill="none" stroke="#A55EEA" stroke-width="3" />
        
        <!-- Сложный путь -->
        <path d="M 350 300 
                 C 370 280, 420 280, 440 300 
                 C 460 320, 460 350, 440 370 
                 C 420 390, 370 390, 350 370 
                 C 330 350, 330 320, 350 300 Z" 
              fill="#F06595" stroke="#333" stroke-width="2" />
    </svg>
</body>
</html>
```

### SVG с CSS и JavaScript

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>SVG с CSS и JavaScript</title>
    <style>
        .interactive-shape {
            cursor: pointer;
            transition: fill 0.3s ease;
        }
        
        .interactive-shape:hover {
            fill: #FFD93D !important;
        }
        
        .pulsing {
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.1); }
            100% { transform: scale(1); }
        }
    </style>
</head>
<body>
    <h1>SVG с CSS и JavaScript</h1>
    
    <svg width="400" height="300" xmlns="http://www.w3.org/2000/svg">
        <!-- Круг с обработчиком события -->
        <circle cx="100" cy="100" r="40" 
                class="interactive-shape" 
                fill="#FF6B6B" 
                onclick="changeColor(this)" />
        
        <!-- Круг с CSS анимацией -->
        <circle cx="250" cy="100" r="40" 
                class="pulsing" 
                fill="#4ECDC4" 
                onclick="toggleAnimation(this)" />
        
        <!-- Группа элементов -->
        <g onclick="rotateGroup(this)" style="cursor: pointer;">
            <rect x="150" y="200" width="50" height="30" fill="#45B7D1" />
            <circle cx="175" cy="190" r="15" fill="#F06595" />
        </g>
        
        <!-- Текст с анимацией -->
        <text x="200" y="280" 
              text-anchor="middle" 
              font-family="Arial" 
              font-size="16" 
              fill="#333"
              onclick="animateText(this)">
            Кликни меня
        </text>
    </svg>
    
    <script>
        function changeColor(element) {
            const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#F06595', '#FF9F43'];
            const randomColor = colors[Math.floor(Math.random() * colors.length)];
            element.setAttribute('fill', randomColor);
        }
        
        function toggleAnimation(element) {
            const isPulsing = element.classList.contains('pulsing');
            if (isPulsing) {
                element.classList.remove('pulsing');
            } else {
                element.classList.add('pulsing');
            }
        }
        
        function rotateGroup(group) {
            const currentTransform = group.getAttribute('transform') || '';
            const match = currentTransform.match(/rotate\(([\d.]+)\)/);
            let angle = 0;
            
            if (match) {
                angle = parseFloat(match[1]);
            }
            
            angle = (angle + 45) % 360;
            group.setAttribute('transform', `rotate(${angle} 175 215)`);
        }
        
        function animateText(textElement) {
            textElement.textContent = 'Анимация!';
            textElement.setAttribute('fill', '#FF6B6B');
            
            setTimeout(() => {
                textElement.textContent = 'Кликни меня';
                textElement.setAttribute('fill', '#333');
            }, 1000);
        }
    </script>
</body>
</html>
```

### Интерактивная SVG диаграмма

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивная SVG диаграмма</title>
    <style>
        .bar {
            cursor: pointer;
            transition: opacity 0.3s ease;
        }
        
        .bar:hover {
            opacity: 0.7;
        }
        
        .bar-label {
            font-family: Arial, sans-serif;
            font-size: 12px;
            text-anchor: middle;
        }
        
        .axis-label {
            font-family: Arial, sans-serif;
            font-size: 14px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Интерактивная SVG диаграмма</h1>
    
    <svg id="chart" width="600" height="400" xmlns="http://www.w3.org/2000/svg">
        <!-- Ось Y -->
        <line x1="50" y1="350" x2="50" y2="50" stroke="#333" stroke-width="2" />
        <!-- Ось X -->
        <line x1="50" y1="350" x2="550" y2="350" stroke="#333" stroke-width="2" />
        
        <!-- Подписи осей -->
        <text x="300" y="390" class="axis-label" text-anchor="middle">Категории</text>
        <text x="15" y="200" class="axis-label" text-anchor="middle" transform="rotate(-90, 15, 200)">Значения</text>
    </svg>
    
    <script>
        // Данные для диаграммы
        const data = [
            { category: 'Янв', value: 30 },
            { category: 'Фев', value: 70 },
            { category: 'Мар', value: 45 },
            { category: 'Апр', value: 80 },
            { category: 'Май', value: 60 },
            { category: 'Июн', value: 90 }
        ];
        
        const svg = document.getElementById('chart');
        const maxValue = Math.max(...data.map(item => item.value));
        const barWidth = 60;
        const barSpacing = 20;
        const chartHeight = 300; // 350 - 50
        
        // Создание столбцов диаграммы
        data.forEach((item, index) => {
            const x = 70 + index * (barWidth + barSpacing);
            const barHeight = (item.value / maxValue) * chartHeight;
            const y = 350 - barHeight;
            
            // Создание столбца
            const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
            rect.setAttribute('x', x);
            rect.setAttribute('y', y);
            rect.setAttribute('width', barWidth);
            rect.setAttribute('height', barHeight);
            rect.setAttribute('fill', '#4ECDC4');
            rect.setAttribute('class', 'bar');
            rect.setAttribute('data-value', item.value);
            rect.setAttribute('data-category', item.category);
            
            // Обработчик клика
            rect.addEventListener('click', function() {
                alert(`Категория: ${item.category}\nЗначение: ${item.value}`);
            });
            
            // Добавление столбца в SVG
            svg.appendChild(rect);
            
            // Добавление подписи
            const label = document.createElementNS('http://www.w3.org/2000/svg', 'text');
            label.setAttribute('x', x + barWidth / 2);
            label.setAttribute('y', 370);
            label.setAttribute('class', 'bar-label');
            label.textContent = item.category;
            
            svg.appendChild(label);
            
            // Добавление значения на столбце
            if (barHeight > 20) { // Показывать только если достаточно места
                const valueLabel = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                valueLabel.setAttribute('x', x + barWidth / 2);
                valueLabel.setAttribute('y', y + 15);
                valueLabel.setAttribute('class', 'bar-label');
                valueLabel.setAttribute('fill', 'white');
                valueLabel.setAttribute('font-size', '10');
                valueLabel.textContent = item.value;
                
                svg.appendChild(valueLabel);
            }
        });
        
        // Добавление сетки
        for (let i = 0; i <= maxValue; i += 20) {
            const y = 350 - (i / maxValue) * chartHeight;
            const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
            line.setAttribute('x1', '50');
            line.setAttribute('y1', y);
            line.setAttribute('x2', '550');
            line.setAttribute('y2', y);
            line.setAttribute('stroke', '#eee');
            line.setAttribute('stroke-dasharray', '2,2');
            
            svg.appendChild(line);
            
            // Подпись шкалы
            const scaleLabel = document.createElementNS('http://www.w3.org/2000/svg', 'text');
            scaleLabel.setAttribute('x', '45');
            scaleLabel.setAttribute('y', y + 4);
            scaleLabel.setAttribute('text-anchor', 'end');
            scaleLabel.setAttribute('font-size', '10');
            scaleLabel.setAttribute('fill', '#666');
            scaleLabel.textContent = i;
            
            svg.appendChild(scaleLabel);
        }
    </script>
</body>
</html>
```

## Canvas vs SVG: Когда что использовать

### Canvas преимущества:
- Быстрее для большого количества объектов
- Подходит для растровой графики и игр
- Лучше для частых обновлений

### SVG преимущества:
- Масштабируется без потери качества
- Доступен для SEO и доступности
- Легко стилизуется через CSS
- Поддерживает события на элементах

### Пример выбора технологии

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Canvas vs SVG</title>
    <style>
        .container {
            display: flex;
            gap: 20px;
            margin: 20px;
        }
        
        .demo {
            border: 1px solid #ddd;
            padding: 10px;
        }
    </style>
</head>
<body>
    <h1>Canvas vs SVG: Примеры использования</h1>
    
    <div class="container">
        <div class="demo">
            <h3>Canvas: Игра "Змейка"</h3>
            <canvas id="snakeCanvas" width="300" height="300"></canvas>
        </div>
        
        <div class="demo">
            <h3>SVG: Интерактивная карта</h3>
            <svg id="map" width="300" height="300" xmlns="http://www.w3.org/2000/svg">
                <rect x="50" y="50" width="200" height="200" 
                      fill="#E0E0E0" stroke="#333" stroke-width="2" 
                      onclick="alert('Клик по области карты')" />
                <circle cx="150" cy="150" r="10" fill="#FF6B6B" 
                        onclick="alert('Клик по маркеру')" />
            </svg>
        </div>
    </div>
    
    <script>
        // Простая анимация в Canvas (демонстрация частых обновлений)
        const canvas = document.getElementById('snakeCanvas');
        const ctx = canvas.getContext('2d');
        
        let angle = 0;
        function animate() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Анимированный элемент
            const x = 150 + Math.cos(angle) * 80;
            const y = 150 + Math.sin(angle) * 80;
            
            ctx.beginPath();
            ctx.arc(x, y, 10, 0, Math.PI * 2);
            ctx.fillStyle = '#FF6B6B';
            ctx.fill();
            
            angle += 0.1;
            requestAnimationFrame(animate);
        }
        
        animate();
    </script>
</body>
</html>
</body>
</html>
```

## Современные возможности Canvas и SVG

### Canvas с WebGL интеграцией
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Canvas с WebGL</title>
</head>
<body>
    <h1>Canvas с WebGL интеграцией</h1>
    
    <canvas id="webgl-canvas" width="800" height="600"></canvas>
    
    <script>
        // Проверка поддержки WebGL
        const canvas = document.getElementById('webgl-canvas');
        let gl;
        
        try {
            // Попытка получить WebGL контекст
            gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');
        } catch (e) {
            console.error('WebGL не поддерживается в этом браузере:', e);
        }
        
        if (gl) {
            // Настройка WebGL
            gl.viewport(0, 0, canvas.width, canvas.height);
            gl.clearColor(0.0, 0.0, 0.0, 1.0);
            gl.clear(gl.COLOR_BUFFER_BIT);
            
            console.log('WebGL контекст успешно инициализирован');
            
            // Создание простого треугольника
            const vertices = new Float32Array([
                -0.5, -0.5, 0.0,
                 0.5, -0.5, 0.0,
                 0.0,  0.5, 0.0
            ]);
            
            const vertexBuffer = gl.createBuffer();
            gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
            gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
            
            // Шейдеры для WebGL
            const vertexShaderSource = `
                attribute vec3 a_position;
                void main() {
                    gl_Position = vec4(a_position, 1.0);
                }
            `;
            
            const fragmentShaderSource = `
                precision mediump float;
                void main() {
                    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0); // Красный цвет
                }
            `;
            
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
            
            const shaderProgram = gl.createProgram();
            gl.attachShader(shaderProgram, vertexShader);
            gl.attachShader(shaderProgram, fragmentShader);
            gl.linkProgram(shaderProgram);
            
            if (!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS)) {
                console.error('Ошибка связывания программы:', gl.getProgramInfoLog(shaderProgram));
            }
            
            gl.useProgram(shaderProgram);
            
            const positionAttributeLocation = gl.getAttribLocation(shaderProgram, 'a_position');
            gl.enableVertexAttribArray(positionAttributeLocation);
            gl.vertexAttribPointer(positionAttributeLocation, 3, gl.FLOAT, false, 0, 0);
            
            gl.drawArrays(gl.TRIANGLES, 0, 3);
        } else {
            console.log('WebGL не поддерживается, используем 2D Canvas');
            
            // Резервный вариант с 2D Canvas
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#FF0000';
            ctx.beginPath();
            ctx.moveTo(400, 150);
            ctx.lineTo(350, 250);
            ctx.lineTo(450, 250);
            ctx.closePath();
            ctx.fill();
        }
    </script>
</body>
</html>
```

### SVG с анимациями и фильтрами
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>SVG с анимациями и фильтрами</title>
</head>
<body>
    <h1>SVG с продвинутыми возможностями</h1>
    
    <svg width="600" height="400" xmlns="http://www.w3.org/2000/svg">
        <!-- Определение фильтров -->
        <defs>
            <!-- Тень -->
            <filter id="drop-shadow" x="-20%" y="-20%" width="140%" height="140%">
                <feGaussianBlur in="SourceAlpha" stdDeviation="4"/>
                <feOffset dx="2" dy="2" result="offsetblur"/>
                <feFlood flood-color="rgba(0,0,0,0.5)"/>
                <feComposite in2="offsetblur" operator="in"/>
                <feMerge>
                    <feMergeNode/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
            
            <!-- Размытие -->
            <filter id="blur" x="-10%" y="-10%" width="120%" height="120%">
                <feGaussianBlur stdDeviation="5" />
            </filter>
            
            <!-- Градиент -->
            <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" style="stop-color:#ff7e5f;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#feb47b;stop-opacity:1" />
            </linearGradient>
            
            <!-- Радиальный градиент -->
            <radialGradient id="radial-gradient" cx="50%" cy="50%" r="50%">
                <stop offset="0%" style="stop-color:#a8edea;stop-opacity:1" />
                <stop offset="100%" style="stop-color:#fed6e3;stop-opacity:1" />
            </radialGradient>
            
            <!-- Шаблон -->
            <pattern id="pattern" x="0" y="0" width="20" height="20" patternUnits="userSpaceOnUse">
                <rect width="10" height="10" fill="#4ECDC4" />
                <rect x="10" y="10" width="10" height="10" fill="#4ECDC4" />
            </pattern>
        </defs>
        
        <!-- Применение фильтров и градиентов -->
        <rect x="50" y="50" width="100" height="100" 
              fill="url(#gradient)" 
              filter="url(#drop-shadow)"
              stroke="#333" stroke-width="2">
            <!-- Анимация -->
            <animateTransform 
                attributeName="transform"
                type="rotate"
                from="0 100 100"
                to="360 100 100"
                dur="4s"
                repeatCount="indefinite" />
        </rect>
        
        <!-- Круг с радиальным градиентом -->
        <circle cx="250" cy="100" r="50" 
                fill="url(#radial-gradient)" 
                filter="url(#blur)">
            <animate 
                attributeName="r" 
                from="50" 
                to="60" 
                dur="2s" 
                repeatCount="indefinite" 
                values="50;60;50" 
                keyTimes="0;0.5;1" />
        </circle>
        
        <!-- Паттерн -->
        <rect x="400" y="50" width="100" height="100" fill="url(#pattern)" />
        
        <!-- Анимированный путь -->
        <path d="M 50 250 Q 150 200, 250 250 T 450 250"
              stroke="#333" 
              stroke-width="3" 
              fill="none"
              stroke-dasharray="10,5">
            <animate 
                attributeName="stroke-dashoffset" 
                from="0" 
                to="-15" 
                dur="1s" 
                repeatCount="indefinite" />
        </path>
        
        <!-- Анимированный текст -->
        <text x="100" y="350" 
              font-family="Arial" 
              font-size="24" 
              fill="url(#gradient)">
            Анимированный текст
            <animate 
                attributeName="x" 
                values="100;150;100" 
                dur="3s" 
                repeatCount="indefinite" />
        </text>
    </svg>
</body>
</html>
```

### Canvas с изображениями и эффектами
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Canvas с изображениями и эффектами</title>
</head>
<body>
    <h1>Canvas с изображениями и графическими эффектами</h1>
    
    <canvas id="image-canvas" width="800" height="600"></canvas>
    
    <div class="controls">
        <label for="effect-select">Выберите эффект:</label>
        <select id="effect-select">
            <option value="original">Оригинал</option>
            <option value="grayscale">Оттенки серого</option>
            <option value="sepia">Сепия</option>
            <option value="invert">Инвертировать</option>
            <option value="blur">Размытие</option>
            <option value="sharpen">Резкость</option>
        </select>
    </div>

    <script>
        class CanvasImageProcessor {
            constructor(canvasId) {
                this.canvas = document.getElementById(canvasId);
                this.ctx = this.canvas.getContext('2d');
                this.image = new Image();
                this.effect = 'original';
                
                this.image.onload = () => {
                    this.drawImage();
                };
                
                // Загрузка изображения (в реальном приложении - с сервера)
                this.image.src = 'https://via.placeholder.com/800x600/4ECDC4/FFFFFF?text=Sample+Image';
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('effect-select').addEventListener('change', (e) => {
                    this.effect = e.target.value;
                    this.applyEffect();
                });
            }
            
            drawImage() {
                this.ctx.drawImage(this.image, 0, 0, this.canvas.width, this.canvas.height);
            }
            
            applyEffect() {
                // Получаем изображение как ImageData
                const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
                const data = imageData.data;
                
                switch(this.effect) {
                    case 'grayscale':
                        this.applyGrayscale(data);
                        break;
                    case 'sepia':
                        this.applySepia(data);
                        break;
                    case 'invert':
                        this.applyInvert(data);
                        break;
                    case 'blur':
                        this.applyBlur(data);
                        break;
                    case 'sharpen':
                        this.applySharpen(data);
                        break;
                    default:
                        // Оригинал - ничего не меняем
                        break;
                }
                
                // Возвращаем обработанные данные на canvas
                this.ctx.putImageData(imageData, 0, 0);
            }
            
            applyGrayscale(data) {
                for (let i = 0; i < data.length; i += 4) {
                    const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
                    data[i] = avg;     // R
                    data[i + 1] = avg; // G
                    data[i + 2] = avg; // B
                    // A остается без изменений
                }
            }
            
            applySepia(data) {
                for (let i = 0; i < data.length; i += 4) {
                    const r = data[i];
                    const g = data[i + 1];
                    const b = data[i + 2];
                    
                    data[i] = Math.min(255, (r * 0.393) + (g * 0.769) + (b * 0.189));     // R
                    data[i + 1] = Math.min(255, (r * 0.349) + (g * 0.686) + (b * 0.168)); // G
                    data[i + 2] = Math.min(255, (r * 0.272) + (g * 0.534) + (b * 0.131)); // B
                }
            }
            
            applyInvert(data) {
                for (let i = 0; i < data.length; i += 4) {
                    data[i] = 255 - data[i];     // R
                    data[i + 1] = 255 - data[i + 1]; // G
                    data[i + 2] = 255 - data[i + 2]; // B
                }
            }
            
            applyBlur(data) {
                // Простое размытие (усреднение соседних пикселей)
                const width = this.canvas.width;
                const height = this.canvas.height;
                const newData = new Uint8ClampedArray(data);
                
                for (let y = 1; y < height - 1; y++) {
                    for (let x = 1; x < width - 1; x++) {
                        const idx = (y * width + x) * 4;
                        
                        // Усреднение соседних пикселей
                        let r = 0, g = 0, b = 0, a = 0;
                        let count = 0;
                        
                        for (let dy = -1; dy <= 1; dy++) {
                            for (let dx = -1; dx <= 1; dx++) {
                                const neighborIdx = ((y + dy) * width + (x + dx)) * 4;
                                r += data[neighborIdx];
                                g += data[neighborIdx + 1];
                                b += data[neighborIdx + 2];
                                a += data[neighborIdx + 3];
                                count++;
                            }
                        }
                        
                        newData[idx] = r / count;
                        newData[idx + 1] = g / count;
                        newData[idx + 2] = b / count;
                        newData[idx + 3] = a / count;
                    }
                }
                
                data.set(newData);
            }
            
            applySharpen(data) {
                // Простое повышение резкости
                const width = this.canvas.width;
                const height = this.canvas.height;
                const newData = new Uint8ClampedArray(data);
                
                // Ядро свертки для повышения резкости
                const kernel = [
                    0, -1, 0,
                    -1, 5, -1,
                    0, -1, 0
                ];
                
                for (let y = 1; y < height - 1; y++) {
                    for (let x = 1; x < width - 1; x++) {
                        const idx = (y * width + x) * 4;
                        
                        let r = 0, g = 0, b = 0;
                        
                        for (let ky = -1; ky <= 1; ky++) {
                            for (let kx = -1; kx <= 1; kx++) {
                                const pixelIdx = ((y + ky) * width + (x + kx)) * 4;
                                const kernelValue = kernel[(ky + 1) * 3 + (kx + 1)];
                                
                                r += data[pixelIdx] * kernelValue;
                                g += data[pixelIdx + 1] * kernelValue;
                                b += data[pixelIdx + 2] * kernelValue;
                            }
                        }
                        
                        newData[idx] = Math.min(255, Math.max(0, r));
                        newData[idx + 1] = Math.min(255, Math.max(0, g));
                        newData[idx + 2] = Math.min(255, Math.max(0, b));
                    }
                }
                
                data.set(newData);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new CanvasImageProcessor('image-canvas');
        });
    </script>
    
    <style>
        .controls {
            margin: 20px 0;
        }
        
        select {
            padding: 8px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
    </style>
</body>
</html>
```

### Интерактивный редактор SVG
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивный SVG редактор</title>
</head>
<body>
    <h1>Интерактивный SVG редактор</h1>
    
    <div class="editor-container">
        <div class="toolbar">
            <button onclick="setTool('rectangle')">Прямоугольник</button>
            <button onclick="setTool('circle')">Круг</button>
            <button onclick="setTool('line')">Линия</button>
            <button onclick="setTool('text')">Текст</button>
            <button onclick="setTool('select')">Выделение</button>
            <button onclick="clearCanvas()">Очистить</button>
            <button onclick="downloadSVG()">Скачать SVG</button>
        </div>
        
        <div class="canvas-container">
            <svg id="drawing-canvas" width="800" height="600" xmlns="http://www.w3.org/2000/svg">
                <rect width="100%" height="100%" fill="#f9f9f9" />
            </svg>
        </div>
        
        <div class="properties-panel">
            <h3>Свойства</h3>
            <div id="element-properties">
                <label>Цвет заливки: <input type="color" id="fill-color" value="#4ECDC4"></label>
                <label>Цвет обводки: <input type="color" id="stroke-color" value="#333"></label>
                <label>Толщина обводки: <input type="number" id="stroke-width" value="2" min="1" max="20"></label>
            </div>
        </div>
    </div>

    <script>
        class SVGDrawingEditor {
            constructor(svgId) {
                this.svg = document.getElementById(svgId);
                this.currentTool = 'select';
                this.isDrawing = false;
                this.currentElement = null;
                this.startPoint = { x: 0, y: 0 };
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                this.svg.addEventListener('mousedown', (e) => this.handleMouseDown(e));
                this.svg.addEventListener('mousemove', (e) => this.handleMouseMove(e));
                this.svg.addEventListener('mouseup', (e) => this.handleMouseUp(e));
                
                document.addEventListener('keydown', (e) => {
                    if (e.key === 'Delete' && this.selectedElement) {
                        this.deleteSelectedElement();
                    }
                });
            }
            
            setTool(tool) {
                this.currentTool = tool;
                this.svg.style.cursor = tool === 'select' ? 'default' : 'crosshair';
                
                // Обновление активной кнопки
                document.querySelectorAll('.toolbar button').forEach(btn => {
                    btn.classList.remove('active');
                });
                
                event.target.classList.add('active');
            }
            
            handleMouseDown(e) {
                const rect = this.svg.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                
                this.startPoint = { x, y };
                
                switch(this.currentTool) {
                    case 'rectangle':
                        this.startDrawingRectangle(x, y);
                        break;
                    case 'circle':
                        this.startDrawingCircle(x, y);
                        break;
                    case 'line':
                        this.startDrawingLine(x, y);
                        break;
                    case 'text':
                        this.addTextElement(x, y);
                        break;
                    case 'select':
                        this.selectElement(e.target);
                        break;
                }
            }
            
            startDrawingRectangle(x, y) {
                this.isDrawing = true;
                this.currentElement = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                
                const fillColor = document.getElementById('fill-color').value;
                const strokeColor = document.getElementById('stroke-color').value;
                const strokeWidth = document.getElementById('stroke-width').value;
                
                this.currentElement.setAttribute('x', x);
                this.currentElement.setAttribute('y', y);
                this.currentElement.setAttribute('fill', fillColor);
                this.currentElement.setAttribute('stroke', strokeColor);
                this.currentElement.setAttribute('stroke-width', strokeWidth);
                
                this.svg.appendChild(this.currentElement);
            }
            
            startDrawingCircle(x, y) {
                this.isDrawing = true;
                this.currentElement = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
                
                const fillColor = document.getElementById('fill-color').value;
                const strokeColor = document.getElementById('stroke-color').value;
                const strokeWidth = document.getElementById('stroke-width').value;
                
                this.currentElement.setAttribute('cx', x);
                this.currentElement.setAttribute('cy', y);
                this.currentElement.setAttribute('r', 0);
                this.currentElement.setAttribute('fill', fillColor);
                this.currentElement.setAttribute('stroke', strokeColor);
                this.currentElement.setAttribute('stroke-width', strokeWidth);
                
                this.svg.appendChild(this.currentElement);
            }
            
            startDrawingLine(x, y) {
                this.isDrawing = true;
                this.currentElement = document.createElementNS('http://www.w3.org/2000/svg', 'line');
                
                const strokeColor = document.getElementById('stroke-color').value;
                const strokeWidth = document.getElementById('stroke-width').value;
                
                this.currentElement.setAttribute('x1', x);
                this.currentElement.setAttribute('y1', y);
                this.currentElement.setAttribute('x2', x);
                this.currentElement.setAttribute('y2', y);
                this.currentElement.setAttribute('stroke', strokeColor);
                this.currentElement.setAttribute('stroke-width', strokeWidth);
                
                this.svg.appendChild(this.currentElement);
            }
            
            addTextElement(x, y) {
                const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                
                text.setAttribute('x', x);
                text.setAttribute('y', y);
                text.setAttribute('fill', document.getElementById('fill-color').value);
                text.setAttribute('font-size', '16');
                text.setAttribute('font-family', 'Arial');
                text.textContent = 'Текст';
                
                this.svg.appendChild(text);
                
                // Сделать текст редактируемым
                text.setAttribute('contenteditable', 'true');
                text.focus();
            }
            
            handleMouseMove(e) {
                if (!this.isDrawing || !this.currentElement) return;
                
                const rect = this.svg.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                
                switch(this.currentTool) {
                    case 'rectangle':
                        const width = Math.abs(x - this.startPoint.x);
                        const height = Math.abs(y - this.startPoint.y);
                        const minX = Math.min(x, this.startPoint.x);
                        const minY = Math.min(y, this.startPoint.y);
                        
                        this.currentElement.setAttribute('x', minX);
                        this.currentElement.setAttribute('y', minY);
                        this.currentElement.setAttribute('width', width);
                        this.currentElement.setAttribute('height', height);
                        break;
                        
                    case 'circle':
                        const radius = Math.sqrt(
                            Math.pow(x - this.startPoint.x, 2) +
                            Math.pow(y - this.startPoint.y, 2)
                        );
                        
                        this.currentElement.setAttribute('r', radius);
                        break;
                        
                    case 'line':
                        this.currentElement.setAttribute('x2', x);
                        this.currentElement.setAttribute('y2', y);
                        break;
                }
            }
            
            handleMouseUp(e) {
                if (this.isDrawing) {
                    this.isDrawing = false;
                    this.currentElement = null;
                }
            }
            
            selectElement(element) {
                if (this.selectedElement) {
                    this.selectedElement.style.stroke = '';
                    this.selectedElement.style.strokeWidth = '';
                }
                
                if (element !== this.svg && element !== this.svg.querySelector('rect')) {
                    this.selectedElement = element;
                    element.style.stroke = '#FF0000';
                    element.style.strokeWidth = '2';
                    
                    this.showElementProperties(element);
                }
            }
            
            showElementProperties(element) {
                const propertiesPanel = document.getElementById('element-properties');
                
                // Показываем свойства в зависимости от типа элемента
                const elementType = element.tagName;
                let propertiesHTML = '';
                
                if (elementType === 'rect' || elementType === 'circle' || elementType === 'line') {
                    propertiesHTML = `
                        <label>Цвет заливки: <input type="color" value="${element.getAttribute('fill') || '#4ECDC4'}" onchange="updateElementProperty('${element.id}', 'fill', this.value)"></label>
                        <label>Цвет обводки: <input type="color" value="${element.getAttribute('stroke') || '#333'}" onchange="updateElementProperty('${element.id}', 'stroke', this.value)"></label>
                        <label>Толщина обводки: <input type="number" value="${element.getAttribute('stroke-width') || '2'}" min="1" max="20" onchange="updateElementProperty('${element.id}', 'stroke-width', this.value)"></label>
                    `;
                } else if (elementType === 'text') {
                    propertiesHTML = `
                        <label>Цвет текста: <input type="color" value="${element.getAttribute('fill') || '#333'}" onchange="updateElementProperty('${element.id}', 'fill', this.value)"></label>
                        <label>Размер шрифта: <input type="number" value="${element.getAttribute('font-size') || '16'}" min="8" max="72" onchange="updateElementProperty('${element.id}', 'font-size', this.value)"></label>
                    `;
                }
                
                propertiesPanel.innerHTML = propertiesHTML;
            }
            
            updateElementProperty(elementId, property, value) {
                const element = document.getElementById(elementId);
                if (element) {
                    element.setAttribute(property, value);
                }
            }
            
            clearCanvas() {
                // Удаляем все элементы, кроме фона
                while (this.svg.children.length > 1) {
                    this.svg.removeChild(this.svg.lastChild);
                }
            }
            
            downloadSVG() {
                const svgData = new XMLSerializer().serializeToString(this.svg);
                const blob = new Blob([svgData], { type: 'image/svg+xml' });
                const url = URL.createObjectURL(blob);
                
                const a = document.createElement('a');
                a.href = url;
                a.download = 'drawing.svg';
                document.body.appendChild(a);
                a.click();
                
                setTimeout(() => {
                    document.body.removeChild(a);
                    URL.revokeObjectURL(url);
                }, 0);
            }
        }
        
        // Инициализация редактора
        let editor;
        document.addEventListener('DOMContentLoaded', () => {
            editor = new SVGEditor('drawing-canvas');
        });
        
        // Глобальные функции для кнопок
        function setTool(tool) {
            editor.setTool(tool);
        }
        
        function clearCanvas() {
            editor.clearCanvas();
        }
        
        function downloadSVG() {
            editor.downloadSVG();
        }
        
        function updateElementProperty(id, property, value) {
            editor.updateElementProperty(id, property, value);
        }
    </script>
    
    <style>
        .editor-container {
            display: flex;
            flex-direction: column;
            max-width: 1200px;
            margin: 0 auto;
        }
        
        .toolbar {
            display: flex;
            gap: 10px;
            padding: 10px;
            background-color: #f0f0f0;
            border-bottom: 1px solid #ddd;
        }
        
        .toolbar button {
            padding: 8px 16px;
            border: none;
            border-radius: 4px;
            background-color: #007acc;
            color: white;
            cursor: pointer;
        }
        
        .toolbar button.active {
            background-color: #005a9e;
        }
        
        .canvas-container {
            border: 1px solid #ddd;
            margin: 10px 0;
        }
        
        #drawing-canvas {
            border: 1px solid #ccc;
            display: block;
        }
        
        .properties-panel {
            padding: 15px;
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .properties-panel label {
            display: block;
            margin-bottom: 10px;
        }
        
        input[type="color"], input[type="number"] {
            margin-left: 10px;
        }
    </style>
</body>
</html>
```

### Canvas с анимацией и физикой
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Canvas с физикой</title>
</head>
<body>
    <h1>Canvas с физическими симуляциями</h1>
    
    <canvas id="physics-canvas" width="800" height="600"></canvas>
    
    <div class="controls">
        <button onclick="addBall()">Добавить шар</button>
        <button onclick="togglePhysics()">Вкл/Выкл физику</button>
        <button onclick="clearBalls()">Очистить</button>
    </div>

    <script>
        class PhysicsCanvas {
            constructor(canvasId) {
                this.canvas = document.getElementById(canvasId);
                this.ctx = this.canvas.getContext('2d');
                this.balls = [];
                this.isPhysicsEnabled = true;
                this.gravity = 0.5;
                this.friction = 0.99;
                
                this.setupEventListeners();
                this.animate();
            }
            
            setupEventListeners() {
                this.canvas.addEventListener('click', (e) => {
                    const rect = this.canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    this.addBallAtPosition(x, y);
                });
            }
            
            addBallAtPosition(x, y) {
                const ball = {
                    x: x,
                    y: y,
                    vx: (Math.random() - 0.5) * 10, // Случайная начальная скорость X
                    vy: (Math.random() - 0.5) * 10, // Случайная начальная скорость Y
                    radius: Math.random() * 20 + 10, // Радиус от 10 до 30
                    color: this.getRandomColor(),
                    mass: 1
                };
                
                this.balls.push(ball);
            }
            
            getRandomColor() {
                const colors = [
                    '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', 
                    '#FFEAA7', '#DDA0DD', '#98D8C8', '#F7DC6F'
                ];
                return colors[Math.floor(Math.random() * colors.length)];
            }
            
            update() {
                if (!this.isPhysicsEnabled) return;
                
                this.balls.forEach((ball, index) => {
                    // Применение гравитации
                    ball.vy += this.gravity;
                    
                    // Обновление позиции
                    ball.x += ball.vx;
                    ball.y += ball.vy;
                    
                    // Отскок от стен
                    if (ball.x - ball.radius < 0) {
                        ball.x = ball.radius;
                        ball.vx = -ball.vx * 0.8; // Потеря энергии при отскоке
                    } else if (ball.x + ball.radius > this.canvas.width) {
                        ball.x = this.canvas.width - ball.radius;
                        ball.vx = -ball.vx * 0.8;
                    }
                    
                    // Отскок от пола
                    if (ball.y + ball.radius > this.canvas.height) {
                        ball.y = this.canvas.height - ball.radius;
                        ball.vy = -ball.vy * 0.8;
                        
                        // Применение трения
                        ball.vx *= this.friction;
                    }
                    
                    // Проверка столкновений между шарами
                    for (let j = index + 1; j < this.balls.length; j++) {
                        this.checkCollision(ball, this.balls[j]);
                    }
                });
            }
            
            checkCollision(ball1, ball2) {
                const dx = ball2.x - ball1.x;
                const dy = ball2.y - ball1.y;
                const distance = Math.sqrt(dx * dx + dy * dy);
                
                // Если шары сталкиваются
                if (distance < ball1.radius + ball2.radius) {
                    // Рассчитываем нормаль столкновения
                    const nx = dx / distance;
                    const ny = dy / distance;
                    
                    // Рассчитываем относительную скорость
                    const dvx = ball2.vx - ball1.vx;
                    const dvy = ball2.vy - ball1.vy;
                    
                    // Скорость в направлении нормали
                    const dvn = dvx * nx + dvy * ny;
                    
                    // Не обрабатываем, если шары расходятся
                    if (dvn > 0) return;
                    
                    // Импульс столкновения
                    const impulse = 2 * dvn / (ball1.mass + ball2.mass);
                    
                    // Обновляем скорости
                    ball1.vx += impulse * ball2.mass * nx;
                    ball1.vy += impulse * ball2.mass * ny;
                    ball2.vx -= impulse * ball1.mass * nx;
                    ball2.vy -= impulse * ball1.mass * ny;
                    
                    // Раздвигаем шары, чтобы они не перекрывались
                    const overlap = ball1.radius + ball2.radius - distance;
                    const separationX = nx * overlap * 0.5;
                    const separationY = ny * overlap * 0.5;
                    
                    ball1.x -= separationX;
                    ball1.y -= separationY;
                    ball2.x += separationX;
                    ball2.y += separationY;
                }
            }
            
            render() {
                // Очистка холста
                this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
                
                // Рисование шаров
                this.balls.forEach(ball => {
                    this.ctx.beginPath();
                    this.ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
                    this.ctx.fillStyle = ball.color;
                    this.ctx.fill();
                    
                    // Тень
                    this.ctx.shadowColor = 'rgba(0, 0, 0, 0.3)';
                    this.ctx.shadowBlur = 5;
                    this.ctx.shadowOffsetX = 2;
                    this.ctx.shadowOffsetY = 2;
                    
                    this.ctx.beginPath();
                    this.ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
                    this.ctx.fillStyle = ball.color;
                    this.ctx.fill();
                    
                    // Сброс тени
                    this.ctx.shadowColor = 'transparent';
                    this.ctx.shadowBlur = 0;
                    this.ctx.shadowOffsetX = 0;
                    this.ctx.shadowOffsetY = 0;
                });
            }
            
            animate() {
                this.update();
                this.render();
                requestAnimationFrame(() => this.animate());
            }
            
            togglePhysics() {
                this.isPhysicsEnabled = !this.isPhysicsEnabled;
            }
            
            clearBalls() {
                this.balls = [];
            }
        }
        
        // Инициализация
        let physicsCanvas;
        document.addEventListener('DOMContentLoaded', () => {
            physicsCanvas = new PhysicsCanvas('physics-canvas');
        });
        
        // Глобальные функции для кнопок
        function addBall() {
            physicsCanvas.addBallAtPosition(
                Math.random() * 600 + 100,
                Math.random() * 200 + 50
            );
        }
        
        function togglePhysics() {
            physicsCanvas.togglePhysics();
        }
        
        function clearBalls() {
            physicsCanvas.clearBalls();
        }
    </script>
    
    <style>
        #physics-canvas {
            border: 2px solid #333;
            display: block;
            margin: 20px auto;
        }
        
        .controls {
            text-align: center;
            margin: 20px 0;
        }
        
        .controls button {
            margin: 0 10px;
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .controls button:hover {
            background-color: #005a9e;
        }
    </style>
</body>
</html>
```

## Практические примеры и шаблоны

### Интерактивная диаграмма с Canvas
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Интерактивная Canvas диаграмма</title>
</head>
<body>
    <h1>Интерактивная диаграмма</h1>
    
    <div class="chart-container">
        <canvas id="interactive-chart" width="800" height="500"></canvas>
        
        <div class="chart-info">
            <div id="hover-info" style="display: none;"></div>
        </div>
    </div>

    <script>
        class InteractiveChart {
            constructor(canvasId) {
                this.canvas = document.getElementById(canvasId);
                this.ctx = this.canvas.getContext('2d');
                this.data = [
                    { label: 'Январь', value: 65, color: '#FF6B6B' },
                    { label: 'Февраль', value: 78, color: '#4ECDC4' },
                    { label: 'Март', value: 45, color: '#45B7D1' },
                    { label: 'Апрель', value: 90, color: '#96CEB4' },
                    { label: 'Май', value: 52, color: '#FFEAA7' },
                    { label: 'Июнь', value: 73, color: '#DDA0DD' }
                ];
                
                this.maxValue = Math.max(...this.data.map(d => d.value));
                this.barWidth = 60;
                this.barSpacing = 30;
                this.chartArea = {
                    x: 80,
                    y: 50,
                    width: this.canvas.width - 160,
                    height: this.canvas.height - 100
                };
                
                this.hoveredBar = null;
                
                this.setupEventListeners();
                this.render();
            }
            
            setupEventListeners() {
                this.canvas.addEventListener('mousemove', (e) => {
                    const rect = this.canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    
                    const barIndex = this.getBarAtPosition(x, y);
                    
                    if (barIndex !== null) {
                        this.hoveredBar = barIndex;
                        this.canvas.style.cursor = 'pointer';
                        this.showHoverInfo(barIndex, x, y);
                    } else {
                        this.hoveredBar = null;
                        this.canvas.style.cursor = 'default';
                        this.hideHoverInfo();
                    }
                    
                    this.render();
                });
                
                this.canvas.addEventListener('click', (e) => {
                    const rect = this.canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    
                    const barIndex = this.getBarAtPosition(x, y);
                    if (barIndex !== null) {
                        this.handleBarClick(barIndex);
                    }
                });
            }
            
            getBarAtPosition(x, y) {
                const chartStartX = this.chartArea.x;
                const chartStartY = this.chartArea.y;
                
                for (let i = 0; i < this.data.length; i++) {
                    const barX = chartStartX + i * (this.barWidth + this.barSpacing);
                    const barY = chartStartY + this.chartArea.height - (this.data[i].value / this.maxValue) * this.chartArea.height;
                    const barHeight = (this.data[i].value / this.maxValue) * this.chartArea.height;
                    
                    if (x >= barX && x <= barX + this.barWidth && 
                        y >= barY && y <= barY + barHeight) {
                        return i;
                    }
                }
                
                return null;
            }
            
            showHoverInfo(barIndex, x, y) {
                const info = document.getElementById('hover-info');
                const data = this.data[barIndex];
                
                info.innerHTML = `
                    <div style="background: rgba(0,0,0,0.8); color: white; padding: 8px; border-radius: 4px; position: absolute; left: ${x + 10}px; top: ${y - 30}px;">
                        <strong>${data.label}</strong><br>
                        Значение: ${data.value}
                    </div>
                `;
                
                info.style.display = 'block';
            }
            
            hideHoverInfo() {
                document.getElementById('hover-info').style.display = 'none';
            }
            
            render() {
                // Очистка
                this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
                
                // Ось Y
                this.ctx.beginPath();
                this.ctx.moveTo(this.chartArea.x, this.chartArea.y);
                this.ctx.lineTo(this.chartArea.x, this.chartArea.y + this.chartArea.height);
                this.ctx.strokeStyle = '#333';
                this.ctx.stroke();
                
                // Ось X
                this.ctx.beginPath();
                this.ctx.moveTo(this.chartArea.x, this.chartArea.y + this.chartArea.height);
                this.ctx.lineTo(this.chartArea.x + this.chartArea.width, this.chartArea.y + this.chartArea.height);
                this.ctx.strokeStyle = '#333';
                this.ctx.stroke();
                
                // Рисование столбцов
                this.data.forEach((item, index) => {
                    const barX = this.chartArea.x + index * (this.barWidth + this.barSpacing);
                    const barHeight = (item.value / this.maxValue) * this.chartArea.height;
                    const barY = this.chartArea.y + this.chartArea.height - barHeight;
                    
                    // Заливка столбца
                    this.ctx.fillStyle = this.hoveredBar === index ? 
                        this.adjustColor(item.color, -30) : item.color;
                    this.ctx.fillRect(barX, barY, this.barWidth, barHeight);
                    
                    // Граница столбца
                    this.ctx.strokeStyle = '#333';
                    this.ctx.lineWidth = 1;
                    this.ctx.strokeRect(barX, barY, this.barWidth, barHeight);
                    
                    // Подпись
                    this.ctx.fillStyle = '#333';
                    this.ctx.font = '12px Arial';
                    this.ctx.textAlign = 'center';
                    this.ctx.fillText(item.label, barX + this.barWidth / 2, this.chartArea.y + this.chartArea.height + 20);
                    
                    // Значение
                    this.ctx.fillText(item.value.toString(), barX + this.barWidth / 2, barY - 5);
                });
                
                // Подписи оси Y
                this.drawYAxisLabels();
            }
            
            adjustColor(hex, amount) {
                let r = parseInt(hex.substring(1, 3), 16);
                let g = parseInt(hex.substring(3, 5), 16);
                let b = parseInt(hex.substring(5, 7), 16);
                
                r = Math.max(0, Math.min(255, r + amount));
                g = Math.max(0, Math.min(255, g + amount));
                b = Math.max(0, Math.min(255, b + amount));
                
                return `#${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')}`;
            }
            
            drawYAxisLabels() {
                const step = this.maxValue / 5;
                
                for (let i = 0; i <= 5; i++) {
                    const value = Math.round(i * step);
                    const y = this.chartArea.y + this.chartArea.height - (value / this.maxValue) * this.chartArea.height;
                    
                    // Линия сетки
                    this.ctx.strokeStyle = '#eee';
                    this.ctx.beginPath();
                    this.ctx.moveTo(this.chartArea.x, y);
                    this.ctx.lineTo(this.chartArea.x + this.chartArea.width, y);
                    this.ctx.stroke();
                    
                    // Подпись
                    this.ctx.fillStyle = '#666';
                    this.ctx.font = '10px Arial';
                    this.ctx.textAlign = 'right';
                    this.ctx.fillText(value.toString(), this.chartArea.x - 5, y + 4);
                }
            }
            
            handleBarClick(barIndex) {
                const data = this.data[barIndex];
                alert(`Вы кликнули на столбец: ${data.label}\nЗначение: ${data.value}`);
            }
        }
        
        // Инициализация диаграммы
        document.addEventListener('DOMContentLoaded', () => {
            new InteractiveChart('interactive-chart');
        });
    </script>
    
    <style>
        .chart-container {
            position: relative;
            margin: 20px auto;
            max-width: 800px;
        }
        
        #interactive-chart {
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        
        .chart-info {
            position: absolute;
            top: 0;
            left: 0;
        }
    </style>
</body>
</html>
```

### SVG с анимацией и интерактивностью
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>SVG с анимацией и интерактивностью</title>
</head>
<body>
    <h1>Интерактивный SVG с анимацией</h1>
    
    <svg id="interactive-svg" width="800" height="600" xmlns="http://www.w3.org/2000/svg">
        <defs>
            <!-- Градиент для анимации -->
            <linearGradient id="anim-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" stop-color="#FF6B6B">
                    <animate attributeName="stop-color" 
                             values="#FF6B6B;#4ECDC4;#45B7D1;#FF6B6B" 
                             dur="4s" 
                             repeatCount="indefinite" />
                </stop>
                <stop offset="100%" stop-color="#4ECDC4">
                    <animate attributeName="stop-color" 
                             values="#4ECDC4;#45B7D1;#FF6B6B;#4ECDC4" 
                             dur="4s" 
                             repeatCount="indefinite" />
                </stop>
            </linearGradient>
            
            <!-- Фильтр для анимации -->
            <filter id="glow">
                <feGaussianBlur stdDeviation="3" result="coloredBlur"/>
                <feMerge> 
                    <feMergeNode in="coloredBlur"/>
                    <feMergeNode in="SourceGraphic"/>
                </feMerge>
            </filter>
        </defs>
        
        <!-- Анимированный круг -->
        <circle id="anim-circle" 
                cx="100" 
                cy="100" 
                r="50" 
                fill="url(#anim-gradient)"
                filter="url(#glow)"
                class="interactive-shape">
            <animateTransform 
                attributeName="transform"
                type="rotate"
                from="0 100 100"
                to="360 100 100"
                dur="3s"
                repeatCount="indefinite" />
        </circle>
        
        <!-- Анимированный путь -->
        <path id="anim-path"
              d="M 200 200 Q 300 150, 400 200 T 600 200"
              stroke="#333"
              stroke-width="3"
              fill="none"
              stroke-dasharray="10,5"
              class="interactive-shape">
            <animate 
                attributeName="stroke-dashoffset" 
                from="0" 
                to="-15" 
                dur="1s" 
                repeatCount="indefinite" />
        </path>
        
        <!-- Анимированный прямоугольник -->
        <rect id="anim-rect"
              x="500"
              y="300"
              width="100"
              height="60"
              fill="#96CEB4"
              rx="10"
              class="interactive-shape">
            <animate 
                attributeName="x" 
                values="500;550;500" 
                dur="2s" 
                repeatCount="indefinite" />
        </rect>
        
        <!-- Группа с анимацией -->
        <g id="anim-group" class="interactive-shape">
            <circle cx="700" cy="100" r="20" fill="#FFEAA7" />
            <circle cx="720" cy="120" r="20" fill="#DDA0DD" />
            <circle cx="680" cy="120" r="20" fill="#98D8C8" />
            
            <animateTransform 
                attributeName="transform"
                type="translate"
                values="0,0; 0,20; 0,0"
                dur="2s"
                repeatCount="indefinite" />
        </g>
        
        <!-- Текст с анимацией -->
        <text id="anim-text" 
              x="400" 
              y="400" 
              font-family="Arial" 
              font-size="24" 
              text-anchor="middle"
              fill="#333"
              class="interactive-shape">
            Анимированный текст
            <animate 
                attributeName="fill" 
                values="#333;#FF6B6B;#4ECDC4;#333" 
                dur="3s" 
                repeatCount="indefinite" />
        </text>
    </svg>

    <script>
        class InteractiveSVG {
            constructor(svgId) {
                this.svg = document.getElementById(svgId);
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                // Обработка кликов по анимированным элементам
                this.svg.querySelectorAll('.interactive-shape').forEach(shape => {
                    shape.addEventListener('click', (e) => {
                        this.handleShapeClick(e.target);
                    });
                    
                    // Обработка наведения
                    shape.addEventListener('mouseenter', (e) => {
                        this.handleShapeHover(e.target, true);
                    });
                    
                    shape.addEventListener('mouseleave', (e) => {
                        this.handleShapeHover(e.target, false);
                    });
                });
            }
            
            handleShapeClick(element) {
                const elementId = element.id;
                const rect = element.getBoundingClientRect();
                
                console.log(`Клик по элементу: ${elementId}`, {
                    x: rect.left,
                    y: rect.top,
                    width: rect.width,
                    height: rect.height
                });
                
                // Добавляем временную анимацию при клике
                this.addClickAnimation(element);
            }
            
            handleShapeHover(element, isHovered) {
                if (isHovered) {
                    element.style.filter = 'url(#glow)';
                    element.style.cursor = 'pointer';
                } else {
                    element.style.filter = '';
                }
            }
            
            addClickAnimation(element) {
                // Создаем анимацию масштабирования при клике
                const animate = document.createElementNS('http://www.w3.org/2000/svg', 'animateTransform');
                animate.setAttribute('attributeName', 'transform');
                animate.setAttribute('type', 'scale');
                animate.setAttribute('from', '1');
                animate.setAttribute('to', '1.2');
                animate.setAttribute('dur', '0.2s');
                animate.setAttribute('fill', 'freeze');
                
                element.appendChild(animate);
                
                // Возвращаем к исходному размеру
                setTimeout(() => {
                    const scaleAnimate = document.createElementNS('http://www.w3.org/2000/svg', 'animateTransform');
                    scaleAnimate.setAttribute('attributeName', 'transform');
                    scaleAnimate.setAttribute('type', 'scale');
                    scaleAnimate.setAttribute('from', '1.2');
                    scaleAnimate.setAttribute('to', '1');
                    scaleAnimate.setAttribute('dur', '0.2s');
                    scaleAnimate.setAttribute('fill', 'freeze');
                    
                    element.appendChild(scaleAnimate);
                    
                    // Удаляем анимации после завершения
                    setTimeout(() => {
                        if (element.contains(animate)) element.removeChild(animate);
                        if (element.contains(scaleAnimate)) element.removeChild(scaleAnimate);
                    }, 200);
                }, 200);
            }
            
            // Динамическое добавление анимированных элементов
            addAnimatedElement(type, options) {
                let element;
                
                switch(type) {
                    case 'circle':
                        element = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
                        element.setAttribute('cx', options.cx || 100);
                        element.setAttribute('cy', options.cy || 100);
                        element.setAttribute('r', options.r || 30);
                        element.setAttribute('fill', options.fill || '#FF6B6B');
                        break;
                        
                    case 'rect':
                        element = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
                        element.setAttribute('x', options.x || 100);
                        element.setAttribute('y', options.y || 100);
                        element.setAttribute('width', options.width || 50);
                        element.setAttribute('height', options.height || 50);
                        element.setAttribute('fill', options.fill || '#4ECDC4');
                        break;
                        
                    case 'text':
                        element = document.createElementNS('http://www.w3.org/2000/svg', 'text');
                        element.setAttribute('x', options.x || 100);
                        element.setAttribute('y', options.y || 100);
                        element.setAttribute('fill', options.fill || '#333');
                        element.textContent = options.text || 'Текст';
                        break;
                }
                
                if (options.animation) {
                    const animate = document.createElementNS('http://www.w3.org/2000/svg', 'animate');
                    Object.entries(options.animation).forEach(([attr, value]) => {
                        animate.setAttribute(attr, value);
                    });
                    element.appendChild(animate);
                }
                
                element.classList.add('interactive-shape');
                this.setupElementEvents(element);
                
                this.svg.appendChild(element);
                return element;
            }
            
            setupElementEvents(element) {
                element.addEventListener('click', (e) => {
                    this.handleShapeClick(e.target);
                });
                
                element.addEventListener('mouseenter', (e) => {
                    this.handleShapeHover(e.target, true);
                });
                
                element.addEventListener('mouseleave', (e) => {
                    this.handleShapeHover(e.target, false);
                });
            }
        }
        
        // Инициализация
        let interactiveSVG;
        document.addEventListener('DOMContentLoaded', () => {
            interactiveSVG = new InteractiveSVG('interactive-svg');
            
            // Пример динамического добавления элемента
            setTimeout(() => {
                interactiveSVG.addAnimatedElement('circle', {
                    cx: 300,
                    cy: 300,
                    r: 40,
                    fill: '#F7DC6F',
                    animation: {
                        attributeName: 'r',
                        values: '40;50;40',
                        dur: '1.5s',
                        repeatCount: 'indefinite'
                    }
                });
            }, 2000);
        });
    </script>
    
    <style>
        #interactive-svg {
            border: 1px solid #ddd;
            border-radius: 8px;
            margin: 20px;
            display: block;
        }
        
        .interactive-shape {
            transition: filter 0.2s ease;
        }
        
        .interactive-shape:hover {
            filter: brightness(1.1);
        }
    </style>
</body>
</html>
```

## Проверка и тестирование безопасности

### Инструменты для проверки:
1. **OWASP ZAP** - для тестирования безопасности веб-приложений
2. **Snyk** - для проверки уязвимостей в зависимостях
3. **npm audit** - для проверки уязвимостей в npm пакетах
4. **SonarQube** - для статического анализа кода
5. **Chrome DevTools Security Panel** - для проверки безопасности в браузере
6. **Mozilla Observatory** - для проверки конфигурации безопасности сайта

### Проверка на уязвимости:
1. Проверка на XSS
2. Проверка на CSRF
3. Проверка на clickjacking
4. Проверка на неправильную настройку CSP
5. Проверка на уязвимости в обработке пользовательского ввода

## Лучшие практики безопасности HTML

### 1. Санитизация ввода
```javascript
// Утилита для санитизации HTML
class HTMLSanitizer {
    static sanitizeHTML(html) {
        const temp = document.createElement('div');
        temp.textContent = html; // Экранируем HTML
        return temp.innerHTML;
    }
    
    static sanitizeInput(input) {
        if (typeof input !== 'string') return input;
        
        return input
            .replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#x27;')
            .replace(/\//g, '&#x2F;');
    }
    
    static sanitizeURL(url) {
        try {
            const parsedUrl = new URL(url);
            
            // Проверяем протокол
            if (!['http:', 'https:', 'mailto:'].includes(parsedUrl.protocol)) {
                throw new Error('Неподдерживаемый протокол');
            }
            
            return parsedUrl.href;
        } catch (error) {
            console.warn('Неверный URL:', url);
            return '#';
        }
    }
}
```

### 2. Защита от CSRF
```javascript
// Генерация CSRF токена
function generateCSRFToken() {
    return btoa(Math.random().toString(36).substring(2) + Math.random().toString(36).substring(2));
}

// Проверка CSRF токена
function validateCSRFToken(token, expectedToken) {
    return token && expectedToken && token === expectedToken;
}

// Добавление токена к формам
document.addEventListener('DOMContentLoaded', () => {
    const forms = document.querySelectorAll('form[method="post"]');
    const token = generateCSRFToken();
    
    forms.forEach(form => {
        const tokenInput = document.createElement('input');
        tokenInput.type = 'hidden';
        tokenInput.name = 'csrf_token';
        tokenInput.value = token;
        form.appendChild(tokenInput);
    });
});
```

### 3. Правильная настройка заголовков безопасности
```html
<!-- HTTP Security Headers -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' https://trusted-cdn.com; 
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; 
               img-src 'self' data: https:; 
               font-src 'self' https://fonts.gstatic.com; 
               connect-src 'self' https://api.example.com; 
               frame-ancestors 'none'; 
               base-uri 'self';">
<meta http-equiv="X-Content-Type-Options" content="nosniff">
<meta http-equiv="X-Frame-Options" content="DENY">
<meta http-equiv="X-XSS-Protection" content="1; mode=block">
<meta name="referrer" content="strict-origin-when-cross-origin">
```

## Заключение

Современные возможности Canvas и SVG предоставляют мощные инструменты для создания интерактивной и визуально привлекательной графики в вебе. Правильное использование этих технологий с учетом безопасности и доступности позволяет создавать высококачественные веб-приложения.

Ключевые аспекты безопасности при работе с Canvas и SVG:
1. Санитизация пользовательского ввода перед отрисовкой
2. Правильная настройка CSP для предотвращения XSS
3. Использование SRI для защиты внешних ресурсов
4. Правильная обработка событий и анимаций
5. Защита от манипуляций с DOM
6. Обеспечение доступности для всех пользователей
7. Проверка и тестирование на уязвимости
8. Следование современным веб-стандартам

Эти практики обеспечивают не только функциональность, но и безопасность, доступность и производительность веб-приложений, использующих Canvas и SVG.

## Следующие темы
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Производительность]]
- [[HTML/Доступность]]

## Теги
#html #canvas #svg #web-security #xss-protection #csrf-protection #content-security-policy #subresource-integrity #web-development #frontend #graphics #animation #interactivity #accessibility #best-practices #security-headers #dom-manipulation #input-sanitization #web-components
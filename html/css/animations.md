# CSS Анимации

CSS анимации позволяют создавать плавные переходы и анимации без использования JavaScript. Они обеспечивают лучшую производительность по сравнению с JavaScript-анимациями и обеспечивают плавное воспроизведение на различных устройствах.

## Введение в CSS анимации

CSS анимации состоят из двух основных компонентов:
1. **@keyframes** - определяет анимацию
2. **animation** - применяет анимацию к элементу

### Основы CSS анимаций

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Анимации</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
        }

        .demo-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 30px;
            max-width: 1200px;
            margin: 0 auto;
        }

        .animation-demo {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        .animation-box {
            width: 100px;
            height: 100px;
            background-color: #007acc;
            margin: 20px 0;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <h1>CSS Анимации</h1>

    <div class="demo-container">
        <!-- Простая анимация движения -->
        <div class="animation-demo">
            <h3>Простое движение</h3>
            <div class="animation-box move-animation"></div>
            <style>
                @keyframes move {
                    0% { transform: translateX(0); }
                    50% { transform: translateX(200px); }
                    100% { transform: translateX(0); }
                }

                .move-animation {
                    animation: move 2s ease-in-out infinite;
                }
            </style>
        </div>

        <!-- Анимация вращения -->
        <div class="animation-demo">
            <h3>Вращение</h3>
            <div class="animation-box rotate-animation"></div>
            <style>
                @keyframes rotate {
                    from { transform: rotate(0deg); }
                    to { transform: rotate(360deg); }
                }

                .rotate-animation {
                    animation: rotate 2s linear infinite;
                }
            </style>
        </div>

        <!-- Анимация изменения размера -->
        <div class="animation-demo">
            <h3>Изменение размера</h3>
            <div class="animation-box scale-animation"></div>
            <style>
                @keyframes scale {
                    0%, 100% { transform: scale(1); }
                    50% { transform: scale(1.5); }
                }

                .scale-animation {
                    animation: scale 1.5s ease-in-out infinite;
                }
            </style>
        </div>

        <!-- Анимация цвета -->
        <div class="animation-demo">
            <h3>Изменение цвета</h3>
            <div class="animation-box color-animation"></div>
            <style>
                @keyframes colorChange {
                    0% { background-color: #007acc; }
                    25% { background-color: #4caf50; }
                    50% { background-color: #ff9800; }
                    75% { background-color: #e91e63; }
                    100% { background-color: #007acc; }
                }

                .color-animation {
                    animation: colorChange 3s ease-in-out infinite;
                }
            </style>
        </div>

        <!-- Анимация с задержкой -->
        <div class="animation-demo">
            <h3>Анимация с задержкой</h3>
            <div class="animation-box delayed-animation"></div>
            <style>
                @keyframes delayedMove {
                    0% { transform: translateX(0) translateY(0); }
                    25% { transform: translateX(100px) translateY(0); }
                    50% { transform: translateX(100px) translateY(100px); }
                    75% { transform: translateX(0) translateY(100px); }
                    100% { transform: translateX(0) translateY(0); }
                }

                .delayed-animation {
                    animation: delayedMove 4s ease-in-out infinite;
                    animation-delay: 1s;
                }
            </style>
        </div>

        <!-- Анимация с разными функциями времени -->
        <div class="animation-demo">
            <h3>Разные функции времени</h3>
            <div class="animation-box bounce-animation"></div>
            <style>
                @keyframes bounce {
                    0%, 20%, 53%, 80%, 100% {
                        transition-timing-function: cubic-bezier(0.215, 0.610, 0.355, 1.000);
                        transform: translate3d(0,0,0);
                    }
                    40%, 43% {
                        transition-timing-function: cubic-bezier(0.755, 0.050, 0.855, 0.060);
                        transform: translate3d(0, -30px, 0);
                    }
                    70% {
                        transition-timing-function: cubic-bezier(0.755, 0.050, 0.855, 0.060);
                        transform: translate3d(0, -15px, 0);
                    }
                    90% {
                        transform: translate3d(0,-4px,0);
                    }
                }

                .bounce-animation {
                    animation: bounce 1s infinite;
                }
            </style>
        </div>
    </div>
</body>
</html>
```

## Свойства анимаций

### Основные свойства animation

```css
.element {
    /* Полная запись */
    animation: name duration timing-function delay iteration-count direction fill-mode play-state;
    
    /* Пример */
    animation: slideIn 1s ease-in-out 0.5s 3 alternate both running;
}

/* Отдельные свойства */
.element {
    animation-name: slideIn;           /* Имя анимации */
    animation-duration: 1s;            /* Длительность */
    animation-timing-function: ease-in-out; /* Функция времени */
    animation-delay: 0.5s;             /* Задержка */
    animation-iteration-count: 3;      /* Количество повторений */
    animation-direction: alternate;    /* Направление */
    animation-fill-mode: both;         /* Режим заполнения */
    animation-play-state: running;     /* Состояние воспроизведения */
}
```

### Значения свойств

#### animation-timing-function
```css
.timing-examples {
    /* Основные значения */
    animation-timing-function: ease;        /* по умолчанию */
    animation-timing-function: linear;      /* равномерно */
    animation-timing-function: ease-in;     /* ускорение в начале */
    animation-timing-function: ease-out;    /* замедление в конце */
    animation-timing-function: ease-in-out; /* ускорение и замедление */
    
    /* Кубические байзы */
    animation-timing-function: cubic-bezier(0.42, 0, 0.58, 1);
    
    /* Ступенчатые функции */
    animation-timing-function: step-start;  /* сразу к конечному значению */
    animation-timing-function: step-end;    /* сразу к начальному значению */
    animation-timing-function: steps(4, end); /* 4 ступени, конец */
}
```

#### animation-iteration-count
```css
.iteration-examples {
    animation-iteration-count: 1;     /* однократное воспроизведение */
    animation-iteration-count: 3;     /* три раза */
    animation-iteration-count: infinite; /* бесконечно */
}
```

#### animation-direction
```css
.direction-examples {
    animation-direction: normal;      /* по умолчанию */
    animation-direction: reverse;     /* в обратном направлении */
    animation-direction: alternate;   /* чередование */
    animation-direction: alternate-reverse; /* обратное чередование */
}
```

#### animation-fill-mode
```css
.fill-mode-examples {
    animation-fill-mode: none;        /* без стилей до/после */
    animation-fill-mode: forwards;    /* сохранить конечное состояние */
    animation-fill-mode: backwards;   /* применить начальное состояние */
    animation-fill-mode: both;        /* и то, и другое */
}
```

## Практические примеры анимаций

### Анимации загрузки (spinners)
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анимации загрузки</title>
    <style>
        .loader-container {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            padding: 20px;
        }

        .loader {
            width: 50px;
            height: 50px;
            display: inline-block;
        }

        .loader-label {
            margin-top: 10px;
            text-align: center;
        }

        .loader-wrapper {
            display: inline-block;
            margin: 20px;
            text-align: center;
        }

        /* Классический спиннер */
        @keyframes spinner {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .spinner {
            border: 4px solid rgba(0, 123, 255, 0.3);
            border-top: 4px solid #007acc;
            border-radius: 50%;
            animation: spinner 1s linear infinite;
        }

        /* Пульсирующий спиннер */
        @keyframes pulse {
            0% { transform: scale(0); opacity: 1; }
            100% { transform: scale(1); opacity: 0; }
        }

        .pulse-loader {
            position: relative;
        }

        .pulse-loader::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border: 4px solid #007acc;
            border-radius: 50%;
            animation: pulse 1.5s ease-out infinite;
        }

        .pulse-loader::after {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: #007acc;
            border-radius: 50%;
            animation: pulse 1.5s ease-out infinite;
            animation-delay: 0.5s;
        }

        /* Прыгающие точки */
        @keyframes bounce-dots {
            0%, 80%, 100% { 
                transform: scale(0);
            } 40% { 
                transform: scale(1.0);
            }
        }

        .bounce-dots {
            display: flex;
            gap: 5px;
        }

        .bounce-dots span {
            width: 10px;
            height: 10px;
            background-color: #007acc;
            border-radius: 50%;
            display: inline-block;
            animation: bounce-dots 1.4s infinite ease-in-out both;
        }

        .bounce-dots span:nth-child(1) { animation-delay: -0.32s; }
        .bounce-dots span:nth-child(2) { animation-delay: -0.16s; }
        .bounce-dots span:nth-child(3) { animation-delay: 0s; }

        /* Скользящий индикатор */
        @keyframes slide-progress {
            0% { left: -100%; }
            100% { left: 100%; }
        }

        .slide-progress {
            position: relative;
            height: 4px;
            background-color: #e0e0e0;
            overflow: hidden;
        }

        .slide-progress::before {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 100%;
            height: 100%;
            background-color: #007acc;
            animation: slide-progress 2s linear infinite;
        }

        /* Вращающийся квадрат */
        @keyframes rotate-square {
            0% { transform: rotate(0deg) scale(1); }
            50% { transform: rotate(180deg) scale(0.8); }
            100% { transform: rotate(360deg) scale(1); }
        }

        .rotate-square {
            background-color: #007acc;
            animation: rotate-square 2s ease-in-out infinite;
        }

        /* Пульсирующий квадрат */
        @keyframes pulse-square {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.1); opacity: 0.7; }
            100% { transform: scale(1); opacity: 1; }
        }

        .pulse-square {
            background-color: #007acc;
            animation: pulse-square 1.5s ease-in-out infinite;
        }
    </style>
</head>
<body>
    <h1>Анимации загрузки</h1>

    <div class="loader-container">
        <div class="loader-wrapper">
            <div class="loader spinner"></div>
            <div class="loader-label">Spinner</div>
        </div>

        <div class="loader-wrapper">
            <div class="loader pulse-loader"></div>
            <div class="loader-label">Pulse</div>
        </div>

        <div class="loader-wrapper">
            <div class="loader bounce-dots">
                <span></span>
                <span></span>
                <span></span>
            </div>
            <div class="loader-label">Bounce Dots</div>
        </div>

        <div class="loader-wrapper">
            <div class="loader slide-progress"></div>
            <div class="loader-label">Slide Progress</div>
        </div>

        <div class="loader-wrapper">
            <div class="loader rotate-square"></div>
            <div class="loader-label">Rotate Square</div>
        </div>

        <div class="loader-wrapper">
            <div class="loader pulse-square"></div>
            <div class="loader-label">Pulse Square</div>
        </div>
    </div>
</body>
</html>
```

### Анимации взаимодействия
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анимации взаимодействия</title>
    <style>
        .interaction-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 30px;
            padding: 20px;
        }

        .interaction-card {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: all 0.3s ease;
        }

        .interaction-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0,0,0,0.15);
        }

        .interactive-element {
            padding: 15px;
            margin: 10px 0;
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        /* Анимация при наведении */
        .hover-animation {
            background-color: #f8f9fa;
            border: 2px solid transparent;
        }

        .hover-animation:hover {
            background-color: #e3f2fd;
            border-color: #2196f3;
            transform: scale(1.02);
        }

        /* Анимация при фокусе */
        .focus-animation {
            background-color: #fff3cd;
            border: 2px solid #ffc107;
            outline: none;
        }

        .focus-animation:focus {
            background-color: #fff;
            border-color: #007acc;
            box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.25);
            transform: scale(1.01);
        }

        /* Анимация при клике */
        .click-animation {
            background-color: #e9ecef;
            border: 2px solid #6c757d;
            position: relative;
            overflow: hidden;
        }

        .click-animation:active {
            background-color: #dee2e6;
            transform: scale(0.98);
        }

        .click-animation::after {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 5px;
            height: 5px;
            background: rgba(255, 255, 255, 0.5);
            opacity: 0;
            border-radius: 100%;
            transform: scale(1, 1) translate(-50%);
            transform-origin: 50% 50%;
        }

        .click-animation:active::after {
            animation: ripple 0.6s linear;
        }

        @keyframes ripple {
            0% {
                transform: scale(0, 0);
                opacity: 0.5;
            }
            100% {
                transform: scale(50, 50);
                opacity: 0;
            }
        }

        /* Анимация появления */
        .appear-animation {
            opacity: 0;
            transform: translateY(20px);
            animation: appear 0.6s ease-out forwards;
        }

        @keyframes appear {
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        /* Анимация исчезновения */
        .disappear-animation {
            animation: disappear 0.3s ease-in forwards;
        }

        @keyframes disappear {
            to {
                opacity: 0;
                transform: translateY(-20px);
            }
        }

        /* Анимация пульсации */
        .pulse-animation {
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% {
                transform: scale(1);
            }
            50% {
                transform: scale(1.05);
            }
            100% {
                transform: scale(1);
            }
        }

        /* Анимация мерцания */
        .blink-animation {
            animation: blink 1s infinite;
        }

        @keyframes blink {
            0%, 50% { opacity: 1; }
            51%, 100% { opacity: 0.3; }
        }

        /* Анимация дыхания */
        .breathing-animation {
            animation: breathing 3s ease-in-out infinite;
        }

        @keyframes breathing {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }
    </style>
</head>
<body>
    <h1>Анимации взаимодействия</h1>

    <div class="interaction-container">
        <div class="interaction-card">
            <h3>Анимации при наведении</h3>
            <div class="interactive-element hover-animation">
                Наведите курсор на этот элемент
            </div>
        </div>

        <div class="interaction-card">
            <h3>Анимации при фокусе</h3>
            <input type="text" 
                   class="interactive-element focus-animation" 
                   placeholder="Нажмите сюда">
        </div>

        <div class="interaction-card">
            <h3>Анимации при клике</h3>
            <div class="interactive-element click-animation">
                Кликните на этот элемент
            </div>
        </div>

        <div class="interaction-card">
            <h3>Анимации появления</h3>
            <div class="interactive-element appear-animation">
                Элемент с анимацией появления
            </div>
        </div>

        <div class="interaction-card">
            <h3>Анимации пульсации</h3>
            <div class="interactive-element pulse-animation">
                Пульсирующий элемент
            </div>
        </div>

        <div class="interaction-card">
            <h3>Анимации мерцания</h3>
            <div class="interactive-element blink-animation">
                Мерцающий элемент
            </div>
        </div>

        <div class="interaction-card">
            <h3>Анимации дыхания</h3>
            <div class="interactive-element breathing-animation">
                Элемент "дыхания"
            </div>
        </div>
    </div>
</body>
</html>
```

### Анимации для уведомлений
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анимации уведомлений</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
        }

        .notification-container {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 1000;
        }

        .notification {
            background-color: #fff;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 10px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
            max-width: 350px;
            position: relative;
            opacity: 0;
            transform: translateX(100%);
        }

        .notification.success {
            border-left: 4px solid #28a745;
        }

        .notification.error {
            border-left: 4px solid #dc3545;
        }

        .notification.warning {
            border-left: 4px solid #ffc107;
        }

        .notification.info {
            border-left: 4px solid #17a2b8;
        }

        .notification-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 8px;
        }

        .notification-title {
            font-weight: bold;
            margin: 0;
        }

        .notification-close {
            background: none;
            border: none;
            font-size: 1.2em;
            cursor: pointer;
            color: #999;
        }

        .notification-close:hover {
            color: #333;
        }

        .notification-content {
            color: #666;
            margin: 0;
        }

        /* Анимации уведомлений */
        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateX(100%);
            }
            to {
                opacity: 1;
                transform: translateX(0);
            }
        }

        @keyframes slideOut {
            from {
                opacity: 1;
                transform: translateX(0);
            }
            to {
                opacity: 0;
                transform: translateX(100%);
            }
        }

        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        @keyframes fadeOut {
            from { opacity: 1; }
            to { opacity: 0; }
        }

        @keyframes scaleIn {
            from {
                opacity: 0;
                transform: scale(0.8);
            }
            to {
                opacity: 1;
                transform: scale(1);
            }
        }

        @keyframes scaleOut {
            from {
                opacity: 1;
                transform: scale(1);
            }
            to {
                opacity: 0;
                transform: scale(0.8);
            }
        }

        .slide-in {
            animation: slideIn 0.3s ease-out forwards;
        }

        .slide-out {
            animation: slideOut 0.3s ease-in forwards;
        }

        .fade-in {
            animation: fadeIn 0.3s ease-out forwards;
        }

        .fade-out {
            animation: fadeOut 0.3s ease-in forwards;
        }

        .scale-in {
            animation: scaleIn 0.3s ease-out forwards;
        }

        .scale-out {
            animation: scaleOut 0.3s ease-in forwards;
        }

        /* Анимации с различными эффектами */
        @keyframes bounceIn {
            0% {
                opacity: 0;
                transform: scale(0.3) translateY(-50%);
            }
            50% {
                opacity: 1;
                transform: scale(1.05) translateY(0);
            }
            70% {
                transform: scale(0.9) translateY(-5px);
            }
            100% {
                opacity: 1;
                transform: scale(1) translateY(0);
            }
        }

        .bounce-in {
            animation: bounceIn 0.6s ease-out forwards;
        }

        /* Анимация с отскоком */
        @keyframes rubberBand {
            0% { transform: scale3d(1, 1, 1); }
            30% { transform: scale3d(1.25, 0.75, 1); }
            40% { transform: scale3d(0.75, 1.25, 1); }
            50% { transform: scale3d(1.15, 0.85, 1); }
            65% { transform: scale3d(0.95, 1.05, 1); }
            75% { transform: scale3d(1.05, 0.95, 1); }
            100% { transform: scale3d(1, 1, 1); }
        }

        .rubber-band {
            animation: rubberBand 0.6s ease-in-out;
        }

        /* Анимация с вращением */
        @keyframes rotateIn {
            0% {
                opacity: 0;
                transform: rotate(-200deg) scale(0.5);
            }
            100% {
                opacity: 1;
                transform: rotate(0deg) scale(1);
            }
        }

        .rotate-in {
            animation: rotateIn 0.6s ease-out forwards;
        }
    </style>
</head>
<body>
    <h1>Анимации уведомлений</h1>

    <div class="controls">
        <button onclick="showNotification('success', 'slide-in')">Успешное действие (Slide)</button>
        <button onclick="showNotification('error', 'fade-in')">Ошибка (Fade)</button>
        <button onclick="showNotification('warning', 'scale-in')">Предупреждение (Scale)</button>
        <button onclick="showNotification('info', 'bounce-in')">Информация (Bounce)</button>
        <button onclick="showNotification('success', 'rubber-band')">Rubber Band</button>
        <button onclick="showNotification('info', 'rotate-in')">Rotate In</button>
    </div>

    <div id="notification-container" class="notification-container"></div>

    <script>
        let notificationId = 0;

        function showNotification(type, animationClass) {
            const container = document.getElementById('notification-container');
            const id = ++notificationId;
            
            const notification = document.createElement('div');
            notification.className = `notification ${type} ${animationClass}`;
            notification.id = `notification-${id}`;
            
            const titles = {
                success: 'Успех!',
                error: 'Ошибка!',
                warning: 'Внимание!',
                info: 'Информация'
            };
            
            const messages = {
                success: 'Операция выполнена успешно',
                error: 'Произошла ошибка при выполнении операции',
                warning: 'Обратите внимание на важную информацию',
                info: 'Дополнительная информация для пользователя'
            };
            
            notification.innerHTML = `
                <div class="notification-header">
                    <h4 class="notification-title">${titles[type]}</h4>
                    <button class="notification-close" onclick="hideNotification(${id})">&times;</button>
                </div>
                <p class="notification-content">${messages[type]}</p>
            `;
            
            container.appendChild(notification);
            
            // Автоматическое скрытие через 5 секунд
            setTimeout(() => {
                hideNotification(id);
            }, 5000);
        }

        function hideNotification(id) {
            const notification = document.getElementById(`notification-${id}`);
            if (notification) {
                notification.classList.remove('slide-in', 'fade-in', 'scale-in', 'bounce-in', 'rubber-band', 'rotate-in');
                notification.classList.add('slide-out');
                
                // Удаляем элемент после завершения анимации
                setTimeout(() => {
                    if (notification.parentNode) {
                        notification.parentNode.removeChild(notification);
                    }
                }, 300);
            }
        }
    </script>
</body>
</html>
```

## Современные возможности CSS анимаций

### CSS Animations с Custom Properties
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анимации с CSS переменными</title>
    <style>
        :root {
            --animation-duration: 1s;
            --animation-easing: ease-in-out;
            --primary-color: #007acc;
            --secondary-color: #6c757d;
            --success-color: #28a745;
            --danger-color: #dc3545;
        }

        .variable-animation-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 30px;
            padding: 20px;
        }

        .animation-demo-card {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        .variable-animated-element {
            width: 100px;
            height: 100px;
            background-color: var(--primary-color);
            margin: 20px 0;
            border-radius: 8px;
        }

        /* Анимация с использованием переменных */
        @keyframes variableMove {
            0% { 
                transform: translateX(0) scale(1);
                background-color: var(--primary-color);
            }
            50% { 
                transform: translateX(200px) scale(1.2);
                background-color: var(--success-color);
            }
            100% { 
                transform: translateX(0) scale(1);
                background-color: var(--primary-color);
            }
        }

        .variable-move {
            animation: variableMove var(--animation-duration) var(--animation-easing) infinite;
        }

        /* Анимация с параметрами */
        @keyframes parametricAnimation {
            0% { 
                transform: rotate(0deg) scale(1);
                background-color: var(--primary-color);
            }
            25% { 
                transform: rotate(90deg) scale(1.1);
                background-color: var(--secondary-color);
            }
            50% { 
                transform: rotate(180deg) scale(1.2);
                background-color: var(--success-color);
            }
            75% { 
                transform: rotate(270deg) scale(1.1);
                background-color: var(--danger-color);
            }
            100% { 
                transform: rotate(360deg) scale(1);
                background-color: var(--primary-color);
            }
        }

        .parametric {
            animation: parametricAnimation calc(var(--animation-duration) * 2) var(--animation-easing) infinite;
        }

        /* Анимация с динамическими параметрами */
        .dynamic-animation {
            animation-duration: var(--dynamic-duration, 1s);
            animation-timing-function: var(--dynamic-easing, ease-in-out);
            animation-iteration-count: var(--dynamic-iterations, infinite);
        }

        /* Контролы для изменения переменных */
        .control-panel {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .control-group {
            margin-bottom: 15px;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }

        input[type="range"] {
            width: 100%;
        }

        .color-picker {
            display: flex;
            gap: 10px;
            margin-top: 10px;
        }

        .color-option {
            width: 30px;
            height: 30px;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid transparent;
        }

        .color-option.active {
            border-color: #333;
        }
    </style>
</head>
<body>
    <h1>Анимации с CSS переменными</h1>

    <div class="control-panel">
        <div class="control-group">
            <label for="duration-control">Длительность анимации: <span id="duration-value">1</span>s</label>
            <input type="range" 
                   id="duration-control" 
                   min="0.5" 
                   max="3" 
                   step="0.1" 
                   value="1"
                   oninput="updateDuration(this.value)">
        </div>

        <div class="control-group">
            <label>Цвет анимации:</label>
            <div class="color-picker">
                <div class="color-option active" style="background-color: #007acc;" onclick="changeColor('#007acc', this)"></div>
                <div class="color-option" style="background-color: #28a745;" onclick="changeColor('#28a745', this)"></div>
                <div class="color-option" style="background-color: #dc3545;" onclick="changeColor('#dc3545', this)"></div>
                <div class="color-option" style="background-color: #ffc107;" onclick="changeColor('#ffc107', this)"></div>
            </div>
        </div>
    </div>

    <div class="variable-animation-container">
        <div class="animation-demo-card">
            <h3>Анимация с переменными</h3>
            <div class="variable-animated-element variable-move"></div>
        </div>

        <div class="animation-demo-card">
            <h3>Параметрическая анимация</h3>
            <div class="variable-animated-element parametric"></div>
        </div>

        <div class="animation-demo-card">
            <h3>Динамическая анимация</h3>
            <div class="variable-animated-element dynamic-animation" 
                 style="--dynamic-duration: 1.5s; --dynamic-easing: ease-out; --dynamic-iterations: infinite;">
            </div>
        </div>
    </div>

    <script>
        function updateDuration(value) {
            document.documentElement.style.setProperty('--animation-duration', value + 's');
            document.getElementById('duration-value').textContent = value;
        }

        function changeColor(color, element) {
            // Убираем активный класс у всех
            document.querySelectorAll('.color-option').forEach(opt => {
                opt.classList.remove('active');
            });
            
            // Добавляем активный класс выбранному
            element.classList.add('active');
            
            // Меняем переменную
            document.documentElement.style.setProperty('--primary-color', color);
        }

        // Добавляем дополнительные анимации с переменными
        const additionalAnimations = `
            @keyframes colorWave {
                0% { background-color: var(--primary-color); }
                25% { background-color: var(--secondary-color); }
                50% { background-color: var(--success-color); }
                75% { background-color: var(--danger-color); }
                100% { background-color: var(--primary-color); }
            }

            @keyframes sizeVariation {
                0%, 100% { transform: scale(1); }
                25% { transform: scale(1.2); }
                50% { transform: scale(0.8); }
                75% { transform: scale(1.1); }
            }

            .color-wave {
                animation: colorWave var(--animation-duration) linear infinite;
            }

            .size-variation {
                animation: sizeVariation calc(var(--animation-duration) * 0.8) ease-in-out infinite;
            }
        `;

        const styleSheet = document.createElement('style');
        styleSheet.textContent = additionalAnimations;
        document.head.appendChild(styleSheet);
    </script>
</body>
</html>
```

### Анимации с использованием CSS Container Queries
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анимации с Container Queries</title>
    <style>
        .container-query-demo {
            container-type: inline-size;
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            padding: 20px;
        }

        .container-card {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            position: relative;
            overflow: hidden;
        }

        .container-animated-element {
            width: 50px;
            height: 50px;
            background-color: #007acc;
            border-radius: 50%;
            position: relative;
        }

        /* Анимация, зависящая от размера контейнера */
        @keyframes containerAware {
            0% { 
                transform: translateX(0) scale(1);
                background-color: #007acc;
            }
            50% { 
                transform: translateX(100px) scale(1.5);
                background-color: #28a745;
            }
            100% { 
                transform: translateX(0) scale(1);
                background-color: #007acc;
            }
        }

        .container-aware-animation {
            animation: containerAware 2s ease-in-out infinite;
        }

        /* Адаптация анимации к размеру контейнера */
        @container (min-width: 300px) {
            .container-aware-animation {
                width: 60px;
                height: 60px;
                animation-duration: 1.5s;
            }
        }

        @container (min-width: 400px) {
            .container-aware-animation {
                width: 70px;
                height: 70px;
                animation-duration: 1s;
                animation-timing-function: ease-out;
            }
        }

        @container (min-width: 500px) {
            .container-aware-animation {
                width: 80px;
                height: 80px;
                animation: containerAware 0.8s linear infinite;
            }
        }

        /* Анимация с различными режимами в зависимости от контейнера */
        @keyframes adaptiveAnimation {
            0% { transform: translate(0, 0) rotate(0deg); }
            25% { transform: translate(50px, 0) rotate(90deg); }
            50% { transform: translate(50px, 50px) rotate(180deg); }
            75% { transform: translate(0, 50px) rotate(270deg); }
            100% { transform: translate(0, 0) rotate(360deg); }
        }

        .adaptive-animation {
            animation: adaptiveAnimation 3s linear infinite;
        }

        @container (max-width: 300px) {
            .adaptive-animation {
                animation-name: simpleMove;
            }
        }

        @keyframes simpleMove {
            0% { transform: translateX(0); }
            100% { transform: translateX(50px); }
        }

        /* Анимация с изменением цвета в зависимости от контейнера */
        @keyframes colorAdaptive {
            0% { background-color: #ff6b6b; }
            25% { background-color: #4ecdc4; }
            50% { background-color: #45b7d1; }
            75% { background-color: #96ceb4; }
            100% { background-color: #ff6b6b; }
        }

        .color-adaptive {
            animation: colorAdaptive 4s ease-in-out infinite;
        }

        @container (min-width: 350px) {
            .color-adaptive {
                animation-duration: 2s;
            }
        }

        @container (min-width: 450px) {
            .color-adaptive {
                animation-duration: 1s;
                animation-timing-function: linear;
            }
        }
    </style>
</head>
<body>
    <h1>Анимации с Container Queries</h1>

    <div class="container-query-demo">
        <div class="container-card" style="width: 280px;">
            <h3>Контейнер 280px</h3>
            <div class="container-animated-element container-aware-animation"></div>
        </div>

        <div class="container-card" style="width: 350px;">
            <h3>Контейнер 350px</h3>
            <div class="container-animated-element adaptive-animation"></div>
        </div>

        <div class="container-card" style="width: 420px;">
            <h3>Контейнер 420px</h3>
            <div class="container-animated-element color-adaptive"></div>
        </div>

        <div class="container-card" style="width: 500px;">
            <h3>Контейнер 500px</h3>
            <div class="container-animated-element container-aware-animation"></div>
        </div>
    </div>
</body>
</html>
```

### Анимации с Web Animations API
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Web Animations API</title>
    <style>
        .web-animation-container {
            padding: 20px;
            max-width: 1200px;
            margin: 0 auto;
        }

        .animation-target {
            width: 100px;
            height: 100px;
            background-color: #007acc;
            margin: 20px;
            border-radius: 8px;
        }

        .controls {
            margin: 20px 0;
        }

        button {
            margin: 5px;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            background-color: #007acc;
            color: white;
            cursor: pointer;
        }

        button:hover {
            background-color: #005a9e;
        }

        .timeline-controls {
            margin: 20px 0;
        }

        .effect-examples {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 30px;
        }

        .effect-card {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <h1>Web Animations API</h1>

    <div class="web-animation-container">
        <div class="controls">
            <button onclick="animateWithAPI()">Запустить анимацию</button>
            <button onclick="pauseAnimation()">Пауза</button>
            <button onclick="resumeAnimation()">Продолжить</button>
            <button onclick="reverseAnimation()">Реверс</button>
            <button onclick="cancelAnimation()">Отменить</button>
            <button onclick="updatePlaybackRate()">Изменить скорость</button>
        </div>

        <div id="animation-target" class="animation-target"></div>

        <div class="timeline-controls">
            <label for="playback-rate">Скорость воспроизведения:</label>
            <input type="range" id="playback-rate" min="0.1" max="3" step="0.1" value="1" onchange="changePlaybackRate(this.value)">
        </div>

        <div class="effect-examples">
            <div class="effect-card">
                <h3>Сложная анимация</h3>
                <div class="animation-target" id="complex-animation"></div>
                <button onclick="runComplexAnimation()">Запустить</button>
            </div>

            <div class="effect-card">
                <h3>Анимация по таймлайну</h3>
                <div class="animation-target" id="timeline-animation"></div>
                <button onclick="runTimelineAnimation()">Запустить</button>
            </div>

            <div class="effect-card">
                <h3>Серия анимаций</h3>
                <div class="animation-target" id="sequence-animation"></div>
                <button onclick="runSequenceAnimation()">Запустить</button>
            </div>
        </div>
    </div>

    <script>
        let animation;
        let complexAnimation;
        let timelineAnimation;
        let sequenceAnimation;

        function animateWithAPI() {
            const element = document.getElementById('animation-target');
            
            // Определяем ключевые кадры
            const keyframes = [
                { transform: 'translateX(0px)', backgroundColor: '#007acc' },
                { transform: 'translateX(200px)', backgroundColor: '#28a745' },
                { transform: 'translateX(0px)', backgroundColor: '#dc3545' }
            ];

            // Параметры анимации
            const options = {
                duration: 2000,
                easing: 'ease-in-out',
                iterations: Infinity
            };

            // Создаем и запускаем анимацию
            animation = element.animate(keyframes, options);
        }

        function pauseAnimation() {
            if (animation) animation.pause();
        }

        function resumeAnimation() {
            if (animation) animation.play();
        }

        function reverseAnimation() {
            if (animation) animation.reverse();
        }

        function cancelAnimation() {
            if (animation) animation.cancel();
        }

        function updatePlaybackRate() {
            if (animation) {
                animation.playbackRate = animation.playbackRate === 1 ? 2 : 1;
            }
        }

        function changePlaybackRate(rate) {
            if (animation) {
                animation.playbackRate = parseFloat(rate);
            }
        }

        function runComplexAnimation() {
            const element = document.getElementById('complex-animation');
            
            const keyframes = [
                { transform: 'translate(0, 0) rotate(0deg) scale(1)', offset: 0 },
                { transform: 'translate(100px, 0) rotate(90deg) scale(1.2)', offset: 0.25 },
                { transform: 'translate(100px, 100px) rotate(180deg) scale(0.8)', offset: 0.5 },
                { transform: 'translate(0, 100px) rotate(270deg) scale(1.1)', offset: 0.75 },
                { transform: 'translate(0, 0) rotate(360deg) scale(1)', offset: 1.0 }
            ];

            const options = {
                duration: 3000,
                easing: 'cubic-bezier(0.42, 0, 0.58, 1)',
                iterations: Infinity
            };

            if (complexAnimation) complexAnimation.cancel();
            complexAnimation = element.animate(keyframes, options);
        }

        function runTimelineAnimation() {
            const element = document.getElementById('timeline-animation');
            
            // Создаем анимацию с временной шкалой
            const animation1 = element.animate([
                { transform: 'scale(1)', opacity: 1 },
                { transform: 'scale(1.2)', opacity: 0.7 }
            ], {
                duration: 1000,
                fill: 'forwards'
            });

            // Вторая анимация, запускаемая после первой
            animation1.onfinish = () => {
                const animation2 = element.animate([
                    { backgroundColor: '#007acc' },
                    { backgroundColor: '#ff6b6b' },
                    { backgroundColor: '#007acc' }
                ], {
                    duration: 500,
                    fill: 'forwards'
                });

                animation2.onfinish = () => {
                    element.animate([
                        { transform: 'scale(1.2)', opacity: 0.7 },
                        { transform: 'scale(1)', opacity: 1 }
                    ], {
                        duration: 1000,
                        fill: 'forwards'
                    });
                };
            };
        }

        function runSequenceAnimation() {
            const element = document.getElementById('sequence-animation');
            
            // Создаем последовательность анимаций
            const effects = [
                {
                    keyframes: [{ transform: 'translateX(0)' }, { transform: 'translateX(150px)' }],
                    options: { duration: 1000, fill: 'forwards' }
                },
                {
                    keyframes: [{ backgroundColor: '#007acc' }, { backgroundColor: '#28a745' }],
                    options: { duration: 800, fill: 'forwards' }
                },
                {
                    keyframes: [{ borderRadius: '8px' }, { borderRadius: '50%' }],
                    options: { duration: 600, fill: 'forwards' }
                }
            ];

            // Запускаем анимации последовательно
            let currentEffect = 0;
            
            function runNextEffect() {
                if (currentEffect >= effects.length) return;
                
                const effect = effects[currentEffect];
                const anim = element.animate(effect.keyframes, effect.options);
                
                anim.onfinish = () => {
                    currentEffect++;
                    runNextEffect();
                };
            }
            
            // Сбрасываем начальное состояние
            element.style.transform = '';
            element.style.backgroundColor = '';
            element.style.borderRadius = '';
            
            runNextEffect();
        }

        // Анимация с пользовательским easing
        function runCustomEasing() {
            const element = document.getElementById('animation-target');
            
            const keyframes = [
                { transform: 'translateY(0)' },
                { transform: 'translateY(-100px)' },
                { transform: 'translateY(0)' }
            ];

            // Пользовательская easing функция
            const customEasing = [0.25, 0.1, 0.25, 1.0]; // cubic-bezier
            
            const options = {
                duration: 2000,
                easing: `cubic-bezier(${customEasing.join(',')})`,
                iterations: Infinity
            };

            if (animation) animation.cancel();
            animation = element.animate(keyframes, options);
        }

        // Анимация с событиями
        function animateWithEvents() {
            const element = document.getElementById('animation-target');
            
            const keyframes = [
                { transform: 'rotate(0deg)', backgroundColor: '#007acc' },
                { transform: 'rotate(360deg)', backgroundColor: '#ff6b6b' }
            ];

            const options = {
                duration: 2000,
                iterations: Infinity
            };

            animation = element.animate(keyframes, options);

            // Добавляем обработчики событий
            animation.onfinish = () => {
                console.log('Анимация завершена');
            };

            animation.oncancel = () => {
                console.log('Анимация отменена');
            };

            // Отслеживание прогресса
            const observer = new PerformanceObserver((list) => {
                for (const entry of list.getEntries()) {
                    if (entry.entryType === 'measure') {
                        console.log('Прогресс анимации:', entry.startTime);
                    }
                }
            });

            observer.observe({ entryTypes: ['measure'] });
        }
    </script>
</body>
</html>
```

### Анимации с Canvas и CSS
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анимации с Canvas и CSS</title>
    <style>
        .canvas-animation-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
        }

        canvas {
            border: 1px solid #ddd;
            border-radius: 8px;
            margin: 20px 0;
        }

        .mixed-animation {
            display: flex;
            gap: 20px;
            align-items: center;
            margin: 20px 0;
        }

        .css-animated {
            width: 100px;
            height: 100px;
            background-color: #007acc;
            border-radius: 8px;
        }

        @keyframes floating {
            0%, 100% { transform: translateY(0px); }
            50% { transform: translateY(-20px); }
        }

        .floating {
            animation: floating 3s ease-in-out infinite;
        }

        .controls {
            margin: 20px 0;
        }

        button {
            margin: 5px;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            background-color: #007acc;
            color: white;
            cursor: pointer;
        }

        button:hover {
            background-color: #005a9e;
        }
    </style>
</head>
<body>
    <h1>Анимации с Canvas и CSS</h1>

    <div class="canvas-animation-container">
        <div class="controls">
            <button onclick="startCanvasAnimation()">Запустить Canvas анимацию</button>
            <button onclick="stopCanvasAnimation()">Остановить</button>
            <button onclick="startMixedAnimation()">Смешанная анимация</button>
        </div>

        <canvas id="particle-canvas" width="600" height="400"></canvas>

        <div class="mixed-animation">
            <div id="css-element" class="css-animated"></div>
            <div>+</div>
            <canvas id="overlay-canvas" width="100" height="100"></canvas>
        </div>
    </div>

    <script>
        let canvas = document.getElementById('particle-canvas');
        let ctx = canvas.getContext('2d');
        let animationId;
        let particles = [];
        let isAnimating = false;

        class Particle {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.vx = (Math.random() - 0.5) * 2;
                this.vy = (Math.random() - 0.5) * 2;
                this.radius = Math.random() * 5 + 2;
                this.color = `hsl(${Math.random() * 360}, 70%, 60%)`;
                this.life = 1;
                this.decay = Math.random() * 0.02 + 0.005;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.vy += 0.1; // гравитация
                this.life -= this.decay;

                // Отскок от стен
                if (this.x <= this.radius || this.x >= canvas.width - this.radius) {
                    this.vx *= -1;
                }
                if (this.y <= this.radius || this.y >= canvas.height - this.radius) {
                    this.vy *= -0.8; // затухание при отскоке
                    this.vx *= 0.9;
                }
            }

            draw(ctx) {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.globalAlpha = this.life;
                ctx.fill();
                ctx.globalAlpha = 1;
            }
        }

        function createParticles(count) {
            particles = [];
            for (let i = 0; i < count; i++) {
                particles.push(new Particle(
                    Math.random() * canvas.width,
                    Math.random() * canvas.height
                ));
            }
        }

        function animateParticles() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Обновляем и рисуем частицы
            for (let i = particles.length - 1; i >= 0; i--) {
                particles[i].update();
                particles[i].draw(ctx);

                // Удаляем умершие частицы
                if (particles[i].life <= 0) {
                    particles.splice(i, 1);
                }
            }

            // Добавляем новые частицы
            if (particles.length < 100 && Math.random() < 0.3) {
                particles.push(new Particle(
                    Math.random() * canvas.width,
                    0
                ));
            }

            if (isAnimating) {
                animationId = requestAnimationFrame(animateParticles);
            }
        }

        function startCanvasAnimation() {
            if (!isAnimating) {
                isAnimating = true;
                createParticles(50);
                animateParticles();
            }
        }

        function stopCanvasAnimation() {
            isAnimating = false;
            if (animationId) {
                cancelAnimationFrame(animationId);
            }
        }

        function startMixedAnimation() {
            // Анимация CSS элемента
            const cssElement = document.getElementById('css-element');
            cssElement.classList.add('floating');

            // Анимация Canvas поверх CSS элемента
            const overlayCanvas = document.getElementById('overlay-canvas');
            const overlayCtx = overlayCanvas.getContext('2d');

            let angle = 0;
            let radius = 30;

            function animateOverlay() {
                overlayCtx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);

                // Рисуем орбиту
                overlayCtx.beginPath();
                overlayCtx.arc(overlayCanvas.width/2, overlayCanvas.height/2, radius, 0, Math.PI * 2);
                overlayCtx.strokeStyle = 'rgba(0, 122, 204, 0.3)';
                overlayCtx.stroke();

                // Рисуем движущуюся точку
                const x = overlayCanvas.width/2 + Math.cos(angle) * radius;
                const y = overlayCanvas.height/2 + Math.sin(angle) * radius;

                overlayCtx.beginPath();
                overlayCtx.arc(x, y, 5, 0, Math.PI * 2);
                overlayCtx.fillStyle = '#ff6b6b';
                overlayCtx.fill();

                angle += 0.1;

                if (isAnimating) {
                    requestAnimationFrame(animateOverlay);
                }
            }

            animateOverlay();
        }

        // Инициализация
        createParticles(30);
    </script>
</body>
</html>
```

## Практические примеры анимаций

### Анимации для форм
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анимации форм</title>
    <style>
        .form-animation-container {
            max-width: 500px;
            margin: 0 auto;
            padding: 20px;
        }

        .form-group {
            margin-bottom: 20px;
            position: relative;
        }

        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }

        .animated-input {
            width: 100%;
            padding: 12px;
            border: 2px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
            transition: all 0.3s ease;
        }

        .animated-input:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 3px rgba(0, 122, 204, 0.2);
        }

        /* Анимация при фокусе */
        @keyframes inputFocus {
            0% {
                transform: scaleX(1);
                border-color: #ddd;
            }
            50% {
                transform: scaleX(1.02);
                border-color: #007acc;
            }
            100% {
                transform: scaleX(1);
                border-color: #007acc;
            }
        }

        .animated-input:focus {
            animation: inputFocus 0.6s ease;
        }

        /* Анимация ошибки */
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            20%, 60% { transform: translateX(-5px); }
            40%, 80% { transform: translateX(5px); }
        }

        .input-error {
            animation: shake 0.6s ease;
            border-color: #dc3545;
            box-shadow: 0 0 0 3px rgba(220, 53, 69, 0.2);
        }

        .error-message {
            color: #dc3545;
            font-size: 0.875em;
            margin-top: 5px;
            opacity: 0;
            transform: translateY(-10px);
            transition: all 0.3s ease;
        }

        .error-message.show {
            opacity: 1;
            transform: translateY(0);
        }

        /* Анимация кнопки отправки */
        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(0, 122, 204, 0.7); }
            70% { box-shadow: 0 0 0 10px rgba(0, 122, 204, 0); }
            100% { box-shadow: 0 0 0 0 rgba(0, 122, 204, 0); }
        }

        .pulse-button {
            animation: pulse 2s infinite;
        }

        .submit-button {
            width: 100%;
            padding: 12px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .submit-button:hover {
            background-color: #005a9e;
            transform: translateY(-2px);
        }

        .submit-button:active {
            transform: translateY(0);
        }

        /* Анимация загрузки */
        .loading-button {
            position: relative;
            pointer-events: none;
        }

        .loading-button::after {
            content: '';
            position: absolute;
            width: 20px;
            height: 20px;
            top: 50%;
            right: 15px;
            transform: translateY(-50%);
            border: 2px solid #ffffff;
            border-top-color: transparent;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            to { transform: translateY(-50%) rotate(360deg); }
        }

        /* Анимация успеха */
        @keyframes successPulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        .success-animation {
            animation: successPulse 0.6s ease;
            background-color: #d4edda;
            border-color: #c3e6cb;
        }

        .success-message {
            background-color: #d4edda;
            color: #155724;
            padding: 12px;
            border-radius: 4px;
            margin-top: 15px;
            opacity: 0;
            transform: scaleY(0);
            transition: all 0.3s ease;
        }

        .success-message.show {
            opacity: 1;
            transform: scaleY(1);
        }

        /* Анимация прогресса */
        .progress-container {
            height: 8px;
            background-color: #e9ecef;
            border-radius: 4px;
            margin: 10px 0;
            overflow: hidden;
        }

        .progress-bar {
            height: 100%;
            background-color: #28a745;
            width: 0%;
            transition: width 0.3s ease;
        }

        .progress-bar.animated {
            animation: progressAnimation 1s ease;
        }

        @keyframes progressAnimation {
            0% { width: 0%; }
            100% { width: var(--progress-width, 0%); }
        }
    </style>
</head>
<body>
    <div class="form-animation-container">
        <h1>Анимации форм</h1>

        <form id="animated-form">
            <div class="form-group">
                <label for="name">Имя:</label>
                <input type="text" 
                       id="name" 
                       name="name" 
                       class="animated-input"
                       required>
                <div class="error-message" id="name-error"></div>
            </div>

            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" 
                       id="email" 
                       name="email" 
                       class="animated-input"
                       required>
                <div class="error-message" id="email-error"></div>
            </div>

            <div class="form-group">
                <label for="password">Пароль:</label>
                <input type="password" 
                       id="password" 
                       name="password" 
                       class="animated-input"
                       required
                       minlength="8">
                <div class="error-message" id="password-error"></div>
            </div>

            <div class="progress-container">
                <div class="progress-bar" id="form-progress"></div>
            </div>

            <button type="submit" class="submit-button" id="submit-btn">Отправить</button>
        </form>

        <div class="success-message" id="success-message">
            Форма успешно отправлена!
        </div>
    </div>

    <script>
        class FormAnimator {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupValidation();
            }

            setupValidation() {
                const inputs = this.form.querySelectorAll('.animated-input');
                
                inputs.forEach(input => {
                    input.addEventListener('blur', (e) => {
                        this.validateField(e.target);
                    });
                    
                    input.addEventListener('input', () => {
                        this.updateProgressBar();
                    });
                });

                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.handleSubmit();
                });
            }

            validateField(field) {
                const fieldName = field.name;
                const errorElement = document.getElementById(`${fieldName}-error`);
                
                if (field.validity.valid) {
                    field.classList.remove('input-error');
                    errorElement.classList.remove('show');
                    errorElement.textContent = '';
                } else {
                    field.classList.add('input-error');
                    errorElement.textContent = this.getErrorMessage(field);
                    errorElement.classList.add('show');
                }
            }

            getErrorMessage(field) {
                if (field.validity.valueMissing) {
                    return 'Это поле обязательно для заполнения';
                } else if (field.validity.typeMismatch && field.type === 'email') {
                    return 'Введите действительный email адрес';
                } else if (field.validity.tooShort) {
                    return `Минимум ${field.minLength} символов`;
                }
                return 'Неверный формат данных';
            }

            updateProgressBar() {
                const inputs = this.form.querySelectorAll('.animated-input');
                const total = inputs.length;
                const filled = Array.from(inputs).filter(input => input.value.trim() !== '').length;
                const progress = (filled / total) * 100;

                const progressBar = document.getElementById('form-progress');
                progressBar.style.setProperty('--progress-width', `${progress}%`);
                progressBar.style.width = `${progress}%`;
                progressBar.classList.add('animated');
                
                // Убираем класс анимации через короткое время
                setTimeout(() => {
                    progressBar.classList.remove('animated');
                }, 1000);
            }

            async handleSubmit() {
                // Проверяем все поля
                const inputs = this.form.querySelectorAll('.animated-input');
                let isValid = true;

                inputs.forEach(input => {
                    if (!input.validity.valid) {
                        input.classList.add('input-error');
                        const errorElement = document.getElementById(`${input.name}-error`);
                        errorElement.textContent = this.getErrorMessage(input);
                        errorElement.classList.add('show');
                        isValid = false;
                    }
                });

                if (!isValid) return;

                // Анимация отправки
                const submitBtn = document.getElementById('submit-btn');
                submitBtn.classList.add('loading-button');
                submitBtn.textContent = 'Отправка...';

                // Имитация отправки
                await new Promise(resolve => setTimeout(resolve, 2000));

                // Анимация успеха
                submitBtn.classList.remove('loading-button');
                submitBtn.textContent = '✓ Отправлено!';
                submitBtn.classList.add('pulse-button');

                // Показать сообщение об успехе
                const successMsg = document.getElementById('success-message');
                successMsg.classList.add('show');

                // Сброс формы через 3 секунды
                setTimeout(() => {
                    this.form.reset();
                    submitBtn.classList.remove('pulse-button');
                    submitBtn.textContent = 'Отправить';
                    
                    // Скрыть сообщение об успехе
                    successMsg.classList.remove('show');
                    
                    // Сбросить прогресс
                    const progressBar = document.getElementById('form-progress');
                    progressBar.style.width = '0%';
                }, 3000);
            }
        }

        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new FormAnimator('animated-form');
        });
    </script>
</body>
</html>
```

## Лучшие практики CSS анимаций

### 1. Производительность анимаций
```css
/* Хорошо: анимация свойств, не вызывающих перерисовку */
.element-good {
    animation: smoothTransform 1s ease-in-out infinite;
}

@keyframes smoothTransform {
    0% { transform: translateX(0); }
    100% { transform: translateX(100px); }
}

/* Плохо: анимация свойств, вызывающих перерисовку */
.element-bad {
    animation: layoutChange 1s ease-in-out infinite;
}

@keyframes layoutChange {
    0% { left: 0; width: 100px; }
    100% { left: 100px; width: 150px; }
}

/* Использование will-change для оптимизации */
.optimized-animation {
    will-change: transform, opacity;
    animation: moveAndFade 1s ease-in-out infinite;
}

/* Использование transform и opacity для производительных анимаций */
.performant-animation {
    transform: translateZ(0); /* Включает GPU ускорение */
    backface-visibility: hidden; /* Предотвращает мерцание */
}
```

### 2. Доступность анимаций
```css
/* Уважение предпочтений пользователя по анимации */
@media (prefers-reduced-motion: reduce) {
    * {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
    }
    
    .animation-container {
        animation: none !important;
        transition: none !important;
    }
}

/* Альтернативные стили для пользователей с чувствительностью к анимации */
.sensitive-user .animated-element {
    animation: none;
    transition: none;
}

/* Постепенное уменьшение интенсивности анимации */
@media (prefers-reduced-motion: no-preference) {
    .gentle-animation {
        animation: gentleMove 2s ease-in-out infinite;
    }
}

@keyframes gentleMove {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(5px); } /* Меньше движения */
}
```

### 3. Семантические анимации
```html
<!-- Анимации с семантической информацией -->
<button id="status-button" 
        aria-live="polite" 
        aria-busy="false"
        onclick="performAction()">
    Выполнить действие
</button>

<div id="status-indicator" 
     class="status-indicator" 
     role="status" 
     aria-live="polite">
    Готово
</div>

<script>
function performAction() {
    const button = document.getElementById('status-button');
    const indicator = document.getElementById('status-indicator');
    
    // Установка состояния загрузки
    button.setAttribute('aria-busy', 'true');
    indicator.textContent = 'Выполняется...';
    button.classList.add('loading-animation');
    
    // Симуляция действия
    setTimeout(() => {
        button.setAttribute('aria-busy', 'false');
        indicator.textContent = 'Готово!';
        button.classList.remove('loading-animation');
        button.classList.add('success-animation');
        
        // Через некоторое время убираем анимацию успеха
        setTimeout(() => {
            button.classList.remove('success-animation');
        }, 2000);
    }, 2000);
}
</script>
```

## Проверка и тестирование анимаций

### Инструменты для тестирования:
1. **Chrome DevTools** - Performance панель для анализа анимаций
2. **Firefox Developer Tools** - Animation inspector
3. **Lighthouse** - проверка производительности анимаций
4. **WebPageTest** - анализ производительности
5. **CSS Triggers** - сайт для проверки свойств, вызывающих перерисовку

### Проверка доступности:
1. Проверка настройки `prefers-reduced-motion`
2. Убедиться, что анимации не мигают слишком быстро (менее 3 раз в секунду)
3. Проверить, что анимации не мешают восприятию контента
4. Обеспечить альтернативный способ взаимодействия

### Проверка производительности:
1. Использовать `transform` и `opacity` вместо `left`, `top`, `width`, `height`
2. Использовать `will-change` для элементов, которые будут анимироваться
3. Избегать анимации большого количества элементов одновременно
4. Использовать `requestAnimationFrame` для JavaScript анимаций

## Заключение

CSS анимации предоставляют мощные возможности для создания интерактивных и привлекательных веб-сайтов. Правильное использование анимаций может значительно улучшить пользовательский опыт, сделать интерфейс более интуитивным и помочь пользователям лучше понимать происходящие процессы. 

Ключевые аспекты современных CSS анимаций:
1. Производительность - использование оптимизированных свойств
2. Доступность - уважение пользовательских предпочтений
3. Семантика - понятные и информативные анимации
4. Совместимость - поддержка различных браузеров
5. Управление - контроль воспроизведения анимаций
6. Интеграция - сочетание с JavaScript и другими технологиями

Современные возможности CSS, такие как Web Animations API, Container Queries и CSS Custom Properties, открывают новые возможности для создания сложных и адаптивных анимаций, которые реагируют на изменения условий окружающей среды и пользовательские настройки.

## Следующие темы
- [[HTML/Продвинутые темы/Шаблоны и веб-компоненты]]
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Производительность]]

## Теги
#css #animations #web-development #frontend #user-experience #accessibility #performance #keyframes #transitions #web-animations-api #css-variables #container-queries #motion-reduction #gpu-acceleration #ui-ux
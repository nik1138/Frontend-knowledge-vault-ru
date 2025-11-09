# CSS Продвинутые возможности

CSS продолжает развиваться, и современные возможности позволяют создавать более сложные, адаптивные и интерактивные веб-страницы. В этой главе мы рассмотрим продвинутые возможности CSS, включая современные методы раскладки, переменные, функции и другие инновационные возможности.

## CSS Custom Properties (Переменные)

CSS переменные (Custom Properties) позволяют определять переиспользуемые значения, которые могут быть изменены динамически.

### Основы CSS переменных

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Переменные</title>
    <style>
        :root {
            /* Определение переменных на уровне документа */
            --primary-color: #007acc;
            --secondary-color: #6c757d;
            --success-color: #28a745;
            --danger-color: #dc3545;
            --warning-color: #ffc107;
            --info-color: #17a2b8;
            
            --font-size-xs: 0.75rem;
            --font-size-sm: 0.875rem;
            --font-size-base: 1rem;
            --font-size-lg: 1.125rem;
            --font-size-xl: 1.25rem;
            --font-size-xxl: 1.5rem;
            
            --border-radius-sm: 0.2rem;
            --border-radius: 0.25rem;
            --border-radius-lg: 0.3rem;
            
            --spacing-xs: 0.25rem;
            --spacing-sm: 0.5rem;
            --spacing-base: 1rem;
            --spacing-lg: 1.5rem;
            --spacing-xl: 2rem;
            
            --shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
            --shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
            --shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);
            
            --transition-base: all 0.2s ease-in-out;
            --transition-fast: all 0.1s ease-in-out;
        }

        /* Использование переменных */
        .button {
            background-color: var(--primary-color);
            color: white;
            padding: var(--spacing-base) var(--spacing-lg);
            border: none;
            border-radius: var(--border-radius);
            font-size: var(--font-size-base);
            cursor: pointer;
            transition: var(--transition-base);
        }

        .button:hover {
            background-color: #005a9e;
            transform: translateY(-2px);
            box-shadow: var(--shadow);
        }

        .card {
            background-color: white;
            border-radius: var(--border-radius-lg);
            padding: var(--spacing-xl);
            box-shadow: var(--shadow);
            margin: var(--spacing-xl) 0;
        }

        .text-primary { color: var(--primary-color); }
        .text-success { color: var(--success-color); }
        .text-danger { color: var(--danger-color); }
        .text-warning { color: var(--warning-color); }
        .text-info { color: var(--info-color); }
    </style>
</head>
<body>
    <h1>CSS Переменные</h1>
    
    <div class="card">
        <h2 class="text-primary">Заголовок карточки</h2>
        <p>Это пример использования CSS переменных для стилизации элементов.</p>
        <button class="button">Кнопка с переменными</button>
    </div>
</body>
</html>
```

### Темизация с переменными

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Темизация с CSS переменными</title>
    <style>
        :root {
            /* Светлая тема по умолчанию */
            --bg-primary: #ffffff;
            --bg-secondary: #f8f9fa;
            --text-primary: #212529;
            --text-secondary: #6c757d;
            --border-color: #dee2e6;
            --shadow: 0 2px 4px rgba(0,0,0,0.1);
            --accent-color: #007acc;
        }

        /* Темная тема */
        .dark-theme {
            --bg-primary: #212529;
            --bg-secondary: #343a40;
            --text-primary: #f8f9fa;
            --text-secondary: #adb5bd;
            --border-color: #495057;
            --shadow: 0 2px 4px rgba(0,0,0,0.3);
            --accent-color: #0d6efd;
        }

        /* Тема с высоким контрастом */
        .high-contrast-theme {
            --bg-primary: #ffffff;
            --bg-secondary: #000000;
            --text-primary: #000000;
            --text-secondary: #000000;
            --border-color: #000000;
            --shadow: 0 2px 4px rgba(0,0,0,0.5);
            --accent-color: #0000ff;
        }

        body {
            background-color: var(--bg-primary);
            color: var(--text-primary);
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            transition: background-color 0.3s, color 0.3s;
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
        }

        .card {
            background-color: var(--bg-secondary);
            color: var(--text-primary);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: var(--shadow);
        }

        .button {
            background-color: var(--accent-color);
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        .button:hover {
            background-color: var(--accent-color);
            filter: brightness(1.2);
        }

        .theme-selector {
            margin-bottom: 20px;
        }

        .theme-selector button {
            margin-right: 10px;
            padding: 8px 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Темизация с CSS переменными</h1>
        
        <div class="theme-selector">
            <button onclick="setTheme('light')">Светлая тема</button>
            <button onclick="setTheme('dark')">Темная тема</button>
            <button onclick="setTheme('high-contrast')">Высокий контраст</button>
        </div>
        
        <div class="card">
            <h2>Карточка контента</h2>
            <p>Этот контент адаптируется к выбранной теме благодаря CSS переменным.</p>
            <button class="button">Пример кнопки</button>
        </div>
        
        <div class="card">
            <h2>Еще одна карточка</h2>
            <p>Все элементы автоматически адаптируются к выбранной теме.</p>
        </div>
    </div>

    <script>
        function setTheme(theme) {
            document.body.className = '';
            document.body.classList.add(`${theme}-theme`);
            
            // Сохраняем выбранную тему в localStorage
            localStorage.setItem('selected-theme', theme);
        }

        // Загружаем сохраненную тему при загрузке страницы
        document.addEventListener('DOMContentLoaded', () => {
            const savedTheme = localStorage.getItem('selected-theme') || 'light';
            setTheme(savedTheme);
        });
    </script>
</body>
</html>
```

### Переменные с fallback значениями

```css
/* Использование fallback значений */
.element {
    /* Если --primary-color не определена, используется #007acc */
    background-color: var(--primary-color, #007acc);
    
    /* Сложный fallback */
    font-size: var(--font-size, var(--base-font-size, 16px));
    
    /* Fallback для ширины */
    width: var(--element-width, 100%);
    
    /* Fallback для тени */
    box-shadow: var(--element-shadow, 0 2px 4px rgba(0,0,0,0.1));
}

/* Динамическое изменение переменных */
.dynamic-theme {
    --primary-color: #ff6b6b;
    --secondary-color: #4ecdc4;
    --accent-color: #45b7d1;
}

/* Переменные с вычислениями */
.calculated-values {
    --base-size: 20px;
    --multiplier: 1.5;
    
    width: calc(var(--base-size) * var(--multiplier));
    height: calc(var(--base-size) * 2);
    padding: calc(var(--base-size) / 4);
}
```

## CSS Functions (calc, clamp, min, max)

### Функция calc()

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS calc() функция</title>
    <style>
        .calc-demo {
            margin: 20px 0;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        /* Основные примеры calc() */
        .calc-basic {
            width: calc(100% - 40px); /* 100% ширины минус 40px */
            margin: calc(5vh - 10px) auto; /* Вертикальный отступ */
            font-size: calc(16px + 0.5vw); /* Адаптивный размер шрифта */
        }

        /* Сложные вычисления */
        .calc-complex {
            width: calc((100% - 60px) / 3); /* 3 колонки с отступами */
            height: calc(100vh - 100px); /* Высота за вычетом заголовка */
            margin: calc(2rem + 1vw);
        }

        /* Использование с переменными */
        .calc-with-vars {
            --container-width: 800px;
            --sidebar-width: 200px;
            --gutter: 20px;
            
            width: calc(var(--container-width) - var(--sidebar-width) - var(--gutter));
            padding: calc(var(--gutter) / 2);
        }

        /* Адаптивные отступы */
        .adaptive-spacing {
            padding: calc(1rem + 2vw);
            margin: calc(2vh + 1rem);
        }

        /* Расчет размеров сетки */
        .grid-item {
            width: calc((100% - 60px) / 4); /* 4 колонки с 20px отступами между ними */
            margin-right: 20px;
        }

        .grid-item:nth-child(4n) {
            margin-right: 0;
        }

        /* Адаптивные шрифты */
        .responsive-typography {
            font-size: calc(1rem + 0.5vw);
            line-height: calc(1.4 + 0.1vw);
        }

        /* Вычисления с разными единицами */
        .mixed-units {
            width: calc(50% + 100px - 2rem);
            height: calc(30vh + 200px);
            padding: calc(1em + 10px);
        }

        /* Использование в анимациях */
        @keyframes calcAnimation {
            0% { transform: translateX(0); }
            100% { transform: translateX(calc(100% - 50px)); }
        }

        .animated-element {
            animation: calcAnimation 2s ease-in-out infinite alternate;
        }
    </style>
</head>
<body>
    <h1>Функция calc()</h1>
    
    <div class="calc-demo">
        <h2>Основные примеры</h2>
        <div class="calc-basic">Элемент с адаптивной шириной: 100% - 40px</div>
    </div>
    
    <div class="calc-demo">
        <h2>Сложные вычисления</h2>
        <div class="calc-complex">Элемент с вычисленной шириной и высотой</div>
    </div>
    
    <div class="calc-demo">
        <h2>С переменными</h2>
        <div class="calc-with-vars">Элемент с вычисленными значениями на основе переменных</div>
    </div>
    
    <div class="calc-demo">
        <h2>Адаптивная сетка</h2>
        <div class="grid-container">
            <div class="grid-item">Колонка 1</div>
            <div class="grid-item">Колонка 2</div>
            <div class="grid-item">Колонка 3</div>
            <div class="grid-item">Колонка 4</div>
        </div>
    </div>
</body>
</html>
```

### Функция clamp()

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS clamp() функция</title>
    <style>
        .clamp-demo {
            margin: 20px 0;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        /* Адаптивный размер шрифта */
        .responsive-heading {
            font-size: clamp(1.2rem, 4vw, 2.5rem);
            line-height: 1.2;
        }

        /* Адаптивная высота элемента */
        .responsive-height {
            height: clamp(200px, 50vh, 600px);
            background-color: #e3f2fd;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* Адаптивный отступ */
        .responsive-padding {
            padding: clamp(1rem, 3vw, 3rem);
        }

        /* Адаптивная ширина */
        .responsive-width {
            width: clamp(300px, 50%, 800px);
        }

        /* Сложные вычисления с clamp */
        .complex-clamp {
            font-size: clamp(1rem, 2.5vw + 0.5rem, 2rem);
            padding: clamp(0.5rem, 2vw, 2rem);
            margin: clamp(1rem, 4vw, 4rem);
        }

        /* Использование с переменными */
        .clamp-with-vars {
            --min-width: 250px;
            --max-width: 1000px;
            --preferred-width: 50%;
            
            width: clamp(var(--min-width), var(--preferred-width), var(--max-width));
        }

        /* Адаптивные отступы для карточек */
        .card-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(clamp(250px, 30%, 400px), 1fr));
            gap: clamp(1rem, 2vw, 2rem);
        }

        .card {
            background: white;
            padding: clamp(1rem, 2vw, 2rem);
            border-radius: clamp(4px, 0.5vw, 8px);
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        /* Адаптивные размеры изображений */
        .adaptive-image {
            width: clamp(200px, 30vw, 500px);
            height: auto;
        }

        /* Адаптивные размеры кнопок */
        .adaptive-button {
            padding: clamp(0.5rem, 1vw, 1rem) clamp(1rem, 2vw, 2rem);
            font-size: clamp(0.875rem, 1.5vw, 1.125rem);
            border-radius: clamp(4px, 0.5vw, 8px);
        }
    </style>
</head>
<body>
    <h1>Функция clamp()</h1>
    
    <div class="clamp-demo">
        <h2 class="responsive-heading">Адаптивный заголовок</h2>
        <p>Этот заголовок изменяет размер в зависимости от ширины экрана, но остается в пределах заданных значений.</p>
    </div>
    
    <div class="clamp-demo">
        <div class="responsive-height">
            Элемент с адаптивной высотой
        </div>
    </div>
    
    <div class="clamp-demo">
        <div class="card-container">
            <div class="card">
                <h3>Карточка 1</h3>
                <p>Содержимое первой карточки с адаптивными размерами.</p>
            </div>
            <div class="card">
                <h3>Карточка 2</h3>
                <p>Содержимое второй карточки с адаптивными размерами.</p>
            </div>
            <div class="card">
                <h3>Карточка 3</h3>
                <p>Содержимое третьей карточки с адаптивными размерами.</p>
            </div>
        </div>
    </div>
    
    <div class="clamp-demo">
        <button class="adaptive-button">Адаптивная кнопка</button>
    </div>
</body>
</html>
```

### Функции min() и max()

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS min() и max() функции</title>
    <style>
        .min-max-demo {
            margin: 20px 0;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        /* Использование min() - выбирает наименьшее значение */
        .min-example {
            width: min(100%, 600px); /* Не больше 600px, но не больше 100% ширины */
            background-color: #f0f8ff;
            padding: 20px;
        }

        /* Использование max() - выбирает наибольшее значение */
        .max-example {
            width: max(300px, 50%); /* Не меньше 300px, но не больше 50% ширины */
            background-color: #fff8dc;
            padding: 20px;
        }

        /* Комбинация min() и max() */
        .combined-example {
            width: min(800px, max(300px, 70%)); /* От 300px до 800px, но не больше 70% */
            background-color: #f5f5dc;
            padding: 20px;
        }

        /* Использование в размерах шрифта */
        .min-max-typography {
            font-size: min(2rem, max(1.2rem, 4vw));
        }

        /* Использование в отступах */
        .min-max-padding {
            padding: min(3rem, max(1rem, 5vw));
        }

        /* Использование в высоте */
        .min-max-height {
            height: min(500px, max(200px, 60vh));
            background-color: #e8f5e8;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* Адаптивные размеры сетки */
        .adaptive-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
            gap: 20px;
        }

        /* Использование с переменными */
        .min-max-vars {
            --min-width: 200px;
            --max-width: 800px;
            --preferred-ratio: 30%;
            
            width: min(var(--max-width), max(var(--min-width), var(--preferred-ratio)));
        }

        /* Адаптивные изображения */
        .adaptive-image-container {
            width: min(100%, max(250px, 30vw));
            height: min(300px, max(150px, 20vh));
            overflow: hidden;
        }

        .adaptive-image {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        /* Адаптивные карточки */
        .adaptive-card {
            min-width: min(300px, 90vw);
            max-width: min(600px, 100%);
            padding: min(1rem, max(0.5rem, 2vw));
        }

        /* Адаптивные кнопки */
        .adaptive-button {
            min-width: max(120px, 20vw);
            padding: min(0.75rem, max(0.5rem, 1vw));
        }
    </style>
</head>
<body>
    <h1>Функции min() и max()</h1>
    
    <div class="min-max-demo">
        <h2>Функция min()</h2>
        <div class="min-example">
            <p>Этот элемент никогда не будет шире 600px, даже если ширина экрана больше.</p>
        </div>
    </div>
    
    <div class="min-max-demo">
        <h2>Функция max()</h2>
        <div class="max-example">
            <p>Этот элемент всегда будет не меньше 300px, даже если ширина экрана меньше.</p>
        </div>
    </div>
    
    <div class="min-max-demo">
        <h2>Комбинированное использование</h2>
        <div class="combined-example">
            <p>Этот элемент имеет ширину от 300px до 800px, но не превышает 70% ширины родителя.</p>
        </div>
    </div>
    
    <div class="min-max-demo">
        <h2>Адаптивная сетка</h2>
        <div class="adaptive-grid">
            <div class="adaptive-card">
                <h3>Карточка 1</h3>
                <p>Содержимое карточки с адаптивной шириной.</p>
            </div>
            <div class="adaptive-card">
                <h3>Карточка 2</h3>
                <p>Содержимое карточки с адаптивной шириной.</p>
            </div>
            <div class="adaptive-card">
                <h3>Карточка 3</h3>
                <p>Содержимое карточки с адаптивной шириной.</p>
            </div>
        </div>
    </div>
    
    <div class="min-max-demo">
        <h2>Адаптивное изображение</h2>
        <div class="adaptive-image-container">
            <img src="https://via.placeholder.com/600x400" alt="Адаптивное изображение" class="adaptive-image">
        </div>
    </div>
</body>
</html>
```

## Современные возможности CSS Grid

### Расширенные возможности Grid

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Современные возможности CSS Grid</title>
    <style>
        .grid-demo {
            margin: 20px 0;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }

        /* Именованные области с адаптивностью */
        .named-areas-grid {
            display: grid;
            grid-template-areas: 
                "header header header"
                "sidebar main aside" 
                "footer footer footer";
            grid-template-columns: minmax(200px, 1fr) 3fr minmax(200px, 1fr);
            grid-template-rows: auto 1fr auto;
            min-height: 100vh;
            gap: 20px;
        }

        .header { grid-area: header; background-color: #e3f2fd; padding: 20px; }
        .sidebar { grid-area: sidebar; background-color: #f3e5f5; padding: 20px; }
        .main { grid-area: main; background-color: #fff3e0; padding: 20px; }
        .aside { grid-area: aside; background-color: #e8f5e8; padding: 20px; }
        .footer { grid-area: footer; background-color: #fce4ec; padding: 20px; }

        /* Адаптивная сетка с auto-fit и minmax */
        .auto-fit-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(min(250px, 100%), 1fr));
            gap: 20px;
            margin: 20px 0;
        }

        /* Адаптивная сетка с auto-fill */
        .auto-fill-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 15px;
            margin: 20px 0;
        }

        /* Сетка с плотным размещением */
        .dense-grid {
            display: grid;
            grid-template-columns: repeat(4, 200px);
            grid-auto-rows: minmax(100px, auto);
            grid-auto-flow: dense; /* Плотное размещение элементов */
            gap: 10px;
            margin: 20px 0;
        }

        .dense-item {
            padding: 15px;
            border-radius: 8px;
            background-color: #e3f2fd;
        }

        .dense-item.tall { grid-row: span 2; } /* Занимает 2 строки */
        .dense-item.wide { grid-column: span 2; } /* Занимает 2 колонки */

        /* Сетка с функцией fit-content */
        .fit-content-grid {
            display: grid;
            grid-template-columns: fit-content(200px) 1fr fit-content(150px);
            gap: 20px;
            margin: 20px 0;
        }

        /* Сетка с процентными и дробными значениями */
        .fractional-grid {
            display: grid;
            grid-template-columns: 1fr 2fr 1fr; /* 25% 50% 25% */
            grid-template-rows: repeat(3, minmax(100px, auto));
            gap: 15px;
            margin: 20px 0;
        }

        /* Адаптивная сетка с container queries (когда поддерживается) */
        .container-grid {
            container-type: inline-size;
        }

        @container (min-width: 600px) {
            .container-grid {
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 20px;
            }
        }

        @container (min-width: 900px) {
            .container-grid {
                grid-template-columns: 1fr 1fr 1fr;
            }
        }

        /* Стили для карточек */
        .grid-card {
            background-color: white;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .grid-card h3 {
            margin-top: 0;
            color: #333;
        }

        /* Адаптивные карточки с grid */
        .card-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(clamp(250px, 30%, 400px), 1fr));
            gap: clamp(1rem, 2vw, 2rem);
        }

        /* Сложная сетка с различными размерами элементов */
        .complex-layout {
            display: grid;
            grid-template-columns: repeat(12, 1fr);
            grid-template-rows: repeat(8, minmax(80px, auto));
            gap: 10px;
            height: 800px;
        }

        .complex-header {
            grid-column: 1 / -1;
            grid-row: 1;
            background-color: #e3f2fd;
        }

        .complex-sidebar {
            grid-column: 1 / 3;
            grid-row: 2 / -1;
            background-color: #f3e5f5;
        }

        .complex-main {
            grid-column: 3 / 10;
            grid-row: 2 / -2;
            background-color: #fff3e0;
        }

        .complex-aside {
            grid-column: 10 / -1;
            grid-row: 2 / 5;
            background-color: #e8f5e8;
        }

        .complex-footer {
            grid-column: 1 / -1;
            grid-row: -2 / -1;
            background-color: #fce4ec;
        }

        /* Стили для демонстрации */
        .demo-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
    </style>
</head>
<body>
    <div class="demo-container">
        <h1>Современные возможности CSS Grid</h1>

        <div class="grid-demo">
            <h2>Именованные области</h2>
            <div class="named-areas-grid">
                <header class="header">Шапка сайта</header>
                <aside class="sidebar">Боковая панель</aside>
                <main class="main">Основное содержимое</main>
                <aside class="aside">Дополнительная информация</aside>
                <footer class="footer">Подвал сайта</footer>
            </div>
        </div>

        <div class="grid-demo">
            <h2>Адаптивная сетка с auto-fit</h2>
            <div class="auto-fit-grid">
                <div class="grid-card">
                    <h3>Карточка 1</h3>
                    <p>Содержимое первой карточки</p>
                </div>
                <div class="grid-card">
                    <h3>Карточка 2</h3>
                    <p>Содержимое второй карточки</p>
                </div>
                <div class="grid-card">
                    <h3>Карточка 3</h3>
                    <p>Содержимое третьей карточки</p>
                </div>
                <div class="grid-card">
                    <h3>Карточка 4</h3>
                    <p>Содержимое четвертой карточки</p>
                </div>
            </div>
        </div>

        <div class="grid-demo">
            <h2>Плотное размещение элементов</h2>
            <div class="dense-grid">
                <div class="dense-item">1</div>
                <div class="dense-item tall">2 (высокий)</div>
                <div class="dense-item">3</div>
                <div class="dense-item wide">4 (широкий)</div>
                <div class="dense-item">5</div>
                <div class="dense-item">6</div>
                <div class="dense-item tall wide">7 (высокий и широкий)</div>
                <div class="dense-item">8</div>
                <div class="dense-item">9</div>
            </div>
        </div>

        <div class="grid-demo">
            <h2>Адаптивная карточная сетка</h2>
            <div class="card-grid">
                <div class="grid-card">
                    <h3>Продукт 1</h3>
                    <p>Описание продукта с адаптивной шириной</p>
                </div>
                <div class="grid-card">
                    <h3>Продукт 2</h3>
                    <p>Описание продукта с адаптивной шириной</p>
                </div>
                <div class="grid-card">
                    <h3>Продукт 3</h3>
                    <p>Описание продукта с адаптивной шириной</p>
                </div>
                <div class="grid-card">
                    <h3>Продукт 4</h3>
                    <p>Описание продукта с адаптивной шириной</p>
                </div>
            </div>
        </div>

        <div class="grid-demo">
            <h2>Сложная сетка</h2>
            <div class="complex-layout">
                <div class="complex-header">Шапка</div>
                <div class="complex-sidebar">Боковая панель</div>
                <div class="complex-main">Основное содержимое</div>
                <div class="complex-aside">Боковая панель 2</div>
                <div class="complex-footer">Подвал</div>
            </div>
        </div>
    </div>
</body>
</html>
```

### Grid с динамическим контентом

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grid с динамическим контентом</title>
    <style>
        .dynamic-grid-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }

        .dynamic-grid-controls {
            margin-bottom: 20px;
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }

        .grid-controls button {
            padding: 8px 16px;
            border: 1px solid #ddd;
            background-color: white;
            border-radius: 4px;
            cursor: pointer;
        }

        .grid-controls button.active {
            background-color: #007acc;
            color: white;
        }

        .dynamic-grid {
            display: grid;
            gap: 20px;
            margin-bottom: 30px;
        }

        .grid-item {
            background-color: #f8f9fa;
            border: 1px solid #dee2e6;
            border-radius: 8px;
            padding: 20px;
            transition: all 0.3s ease;
        }

        .grid-item:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 16px rgba(0,0,0,0.1);
        }

        .grid-item.featured {
            grid-column: 1 / -1; /* Занимает всю ширину */
            background-color: #e3f2fd;
            border-color: #2196f3;
        }

        .grid-item.large {
            grid-column: span 2; /* Занимает 2 колонки */
        }

        .grid-item.tall {
            grid-row: span 2; /* Занимает 2 строки */
        }

        /* Адаптивные размеры сетки */
        .grid-1 { grid-template-columns: 1fr; }
        .grid-2 { grid-template-columns: repeat(2, 1fr); }
        .grid-3 { grid-template-columns: repeat(3, 1fr); }
        .grid-4 { grid-template-columns: repeat(4, 1fr); }

        @media (max-width: 768px) {
            .grid-3, .grid-4 {
                grid-template-columns: repeat(2, 1fr);
            }
        }

        @media (max-width: 480px) {
            .grid-2, .grid-3, .grid-4 {
                grid-template-columns: 1fr;
            }
        }

        .item-content {
            margin-bottom: 10px;
        }

        .item-actions {
            display: flex;
            gap: 10px;
            margin-top: 10px;
        }

        .item-actions button {
            padding: 4px 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.8em;
        }
    </style>
</head>
<body>
    <div class="dynamic-grid-container">
        <h1>Grid с динамическим контентом</h1>

        <div class="grid-controls">
            <button onclick="changeGridLayout(1)" class="active" data-cols="1">1 колонка</button>
            <button onclick="changeGridLayout(2)" data-cols="2">2 колонки</button>
            <button onclick="changeGridLayout(3)" data-cols="3">3 колонки</button>
            <button onclick="changeGridLayout(4)" data-cols="4">4 колонки</button>
        </div>

        <div class="grid-controls">
            <button onclick="addItem()">Добавить элемент</button>
            <button onclick="addFeaturedItem()">Добавить featured</button>
            <button onclick="addLargeItem()">Добавить large</button>
            <button onclick="addTallItem()">Добавить tall</button>
            <button onclick="shuffleItems()">Перемешать</button>
            <button onclick="clearGrid()">Очистить</button>
        </div>

        <div id="dynamic-grid" class="dynamic-grid grid-3">
            <!-- Динамические элементы будут добавляться сюда -->
        </div>
    </div>

    <script>
        let itemCount = 0;
        let gridElement = document.getElementById('dynamic-grid');

        // Инициализация с несколькими элементами
        for (let i = 0; i < 6; i++) {
            addItem();
        }

        function changeGridLayout(cols) {
            // Удаляем все классы сетки
            gridElement.className = gridElement.className.replace(/grid-\d/g, '');
            gridElement.classList.add(`grid-${cols}`);

            // Обновляем активную кнопку
            document.querySelectorAll('.grid-controls button[data-cols]').forEach(btn => {
                btn.classList.remove('active');
            });
            document.querySelector(`.grid-controls button[data-cols="${cols}"]`).classList.add('active');
        }

        function addItem() {
            itemCount++;
            const item = document.createElement('div');
            item.className = 'grid-item';
            item.id = `item-${itemCount}`;
            item.innerHTML = `
                <div class="item-content">
                    <h3>Элемент ${itemCount}</h3>
                    <p>Содержимое элемента ${itemCount}. Добавлен динамически.</p>
                    <p>Создан: ${new Date().toLocaleTimeString()}</p>
                </div>
                <div class="item-actions">
                    <button onclick="removeItem(${itemCount})">Удалить</button>
                    <button onclick="toggleFeatured(${itemCount})">Featured</button>
                    <button onclick="toggleLarge(${itemCount})">Large</button>
                    <button onclick="toggleTall(${itemCount})">Tall</button>
                </div>
            `;

            gridElement.appendChild(item);
        }

        function addFeaturedItem() {
            itemCount++;
            const item = document.createElement('div');
            item.className = 'grid-item featured';
            item.id = `item-${itemCount}`;
            item.innerHTML = `
                <div class="item-content">
                    <h3>Featured элемент ${itemCount}</h3>
                    <p>Этот элемент занимает всю ширину сетки.</p>
                    <p>Создан: ${new Date().toLocaleTimeString()}</p>
                </div>
                <div class="item-actions">
                    <button onclick="removeItem(${itemCount})">Удалить</button>
                    <button onclick="toggleFeatured(${itemCount})">Убрать Featured</button>
                </div>
            `;

            gridElement.appendChild(item);
        }

        function addLargeItem() {
            itemCount++;
            const item = document.createElement('div');
            item.className = 'grid-item large';
            item.id = `item-${itemCount}`;
            item.innerHTML = `
                <div class="item-content">
                    <h3>Large элемент ${itemCount}</h3>
                    <p>Этот элемент занимает 2 колонки сетки.</p>
                    <p>Создан: ${new Date().toLocaleTimeString()}</p>
                </div>
                <div class="item-actions">
                    <button onclick="removeItem(${itemCount})">Удалить</button>
                    <button onclick="toggleLarge(${itemCount})">Убрать Large</button>
                </div>
            `;

            gridElement.appendChild(item);
        }

        function addTallItem() {
            itemCount++;
            const item = document.createElement('div');
            item.className = 'grid-item tall';
            item.id = `item-${itemCount}`;
            item.innerHTML = `
                <div class="item-content">
                    <h3>Tall элемент ${itemCount}</h3>
                    <p>Этот элемент занимает 2 строки сетки.</p>
                    <p>Создан: ${new Date().toLocaleTimeString()}</p>
                </div>
                <div class="item-actions">
                    <button onclick="removeItem(${itemCount})">Удалить</button>
                    <button onclick="toggleTall(${itemCount})">Убрать Tall</button>
                </div>
            `;

            gridElement.appendChild(item);
        }

        function removeItem(id) {
            const item = document.getElementById(`item-${id}`);
            if (item) {
                item.remove();
            }
        }

        function toggleFeatured(id) {
            const item = document.getElementById(`item-${id}`);
            if (item) {
                item.classList.toggle('featured');
                const button = event.target;
                button.textContent = item.classList.contains('featured') ? 'Убрать Featured' : 'Featured';
            }
        }

        function toggleLarge(id) {
            const item = document.getElementById(`item-${id}`);
            if (item) {
                item.classList.toggle('large');
                const button = event.target;
                button.textContent = item.classList.contains('large') ? 'Убрать Large' : 'Large';
            }
        }

        function toggleTall(id) {
            const item = document.getElementById(`item-${id}`);
            if (item) {
                item.classList.toggle('tall');
                const button = event.target;
                button.textContent = item.classList.contains('tall') ? 'Убрать Tall' : 'Tall';
            }
        }

        function shuffleItems() {
            const items = Array.from(gridElement.children);
            // Перемешиваем массив
            for (let i = items.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [items[i], items[j]] = [items[j], items[i]];
            }

            // Очищаем контейнер и добавляем элементы в новом порядке
            gridElement.innerHTML = '';
            items.forEach(item => gridElement.appendChild(item));
        }

        function clearGrid() {
            gridElement.innerHTML = '';
            itemCount = 0;
        }
    </script>
</body>
</html>
```

## Container Queries (экспериментальная возможность)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Container Queries</title>
    <style>
        .container-query-demo {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }

        /* Определение контейнера */
        .card-container {
            container-type: inline-size;
            container-name: card-container;
            margin-bottom: 20px;
        }

        /* Стили для карточки */
        .card {
            background: white;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            margin-bottom: 15px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            transition: all 0.3s ease;
        }

        .card:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 16px rgba(0,0,0,0.15);
        }

        .card-image {
            width: 100%;
            height: 200px;
            object-fit: cover;
            border-radius: 4px;
            margin-bottom: 15px;
        }

        .card-title {
            margin: 0 0 10px 0;
            font-size: 1.2em;
        }

        .card-content {
            color: #666;
            line-height: 1.5;
        }

        /* Container queries для адаптивности карточки */
        @container card-container (min-width: 300px) {
            .card {
                display: flex;
                flex-direction: row;
                align-items: center;
            }

            .card-image {
                width: 150px;
                height: 150px;
                margin-right: 20px;
                margin-bottom: 0;
            }

            .card-content-container {
                flex: 1;
            }
        }

        @container card-container (min-width: 500px) {
            .card {
                flex-direction: row;
            }

            .card-image {
                width: 200px;
                height: 150px;
            }
        }

        @container card-container (min-width: 700px) {
            .card {
                padding: 30px;
            }

            .card-title {
                font-size: 1.5em;
            }

            .card-image {
                width: 250px;
                height: 180px;
            }
        }

        /* Вложенные container queries */
        @container (min-width: 400px) {
            @container (orientation: landscape) {
                .adaptive-component {
                    flex-direction: row;
                }
            }
        }

        /* Контейнеры с разными типами */
        .size-container {
            container-type: size;
        }

        .normal-container {
            container-type: normal;
        }

        /* Сложные примеры */
        .product-showcase {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
        }

        .product-card {
            container-type: inline-size;
            background: white;
            border: 1px solid #eee;
            border-radius: 8px;
            padding: 15px;
            transition: box-shadow 0.3s;
        }

        .product-card:hover {
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }

        .product-image {
            width: 100%;
            height: 150px;
            object-fit: cover;
            border-radius: 4px;
            margin-bottom: 10px;
        }

        .product-info h3 {
            margin: 0 0 8px 0;
        }

        .product-price {
            font-weight: bold;
            color: #007acc;
            font-size: 1.1em;
        }

        /* Адаптация продукта в зависимости от размера контейнера */
        @container (min-width: 280px) {
            .product-card {
                display: flex;
                flex-direction: column;
            }

            .product-image {
                height: 120px;
            }
        }

        @container (min-width: 350px) {
            .product-card {
                flex-direction: row;
                align-items: center;
            }

            .product-image {
                width: 100px;
                height: 100px;
                margin-right: 15px;
                margin-bottom: 0;
            }

            .product-info {
                flex: 1;
            }
        }

        @container (min-width: 450px) {
            .product-card {
                padding: 20px;
            }

            .product-image {
                width: 120px;
                height: 120px;
            }
        }

        /* Контейнеры для навигации */
        .nav-container {
            container-type: inline-size;
            background: #333;
            padding: 10px;
            border-radius: 8px;
        }

        .nav-list {
            display: flex;
            list-style: none;
            margin: 0;
            padding: 0;
            gap: 10px;
        }

        .nav-item {
            padding: 8px 16px;
            background: #555;
            border-radius: 4px;
            color: white;
            text-decoration: none;
        }

        /* Адаптация навигации */
        @container (max-width: 300px) {
            .nav-list {
                flex-direction: column;
                gap: 5px;
            }
        }

        @container (min-width: 301px) and (max-width: 500px) {
            .nav-list {
                flex-wrap: wrap;
            }
        }

        @container (min-width: 501px) {
            .nav-list {
                flex-direction: row;
            }
        }

        /* Контейнеры для формы */
        .form-container {
            container-type: inline-size;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }

        .form-group {
            margin-bottom: 15px;
        }

        .form-input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        /* Адаптация формы */
        @container (min-width: 400px) {
            .form-grid {
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 15px;
            }
        }
    </style>
</head>
<body>
    <div class="container-query-demo">
        <h1>Container Queries</h1>

        <h2>Адаптивные карточки</h2>
        
        <!-- Контейнер с изменяющимся размером -->
        <div class="resizable-container" style="resize: horizontal; overflow: auto; width: 300px; border: 1px solid #ccc; padding: 10px;">
            <div class="card-container">
                <div class="card">
                    <img src="https://via.placeholder.com/300x200" alt="Карточка" class="card-image">
                    <div class="card-content-container">
                        <h3 class="card-title">Адаптивная карточка</h3>
                        <p class="card-content">Содержимое карточки, которое адаптируется в зависимости от размера контейнера.</p>
                    </div>
                </div>
            </div>
        </div>

        <h2>Галерея продуктов</h2>
        <div class="product-showcase">
            <div class="product-card">
                <img src="https://via.placeholder.com/200x150" alt="Продукт" class="product-image">
                <div class="product-info">
                    <h3>Продукт 1</h3>
                    <p>Описание продукта</p>
                    <div class="product-price">1 990 ₽</div>
                </div>
            </div>
            <div class="product-card">
                <img src="https://via.placeholder.com/200x150" alt="Продукт" class="product-image">
                <div class="product-info">
                    <h3>Продукт 2</h3>
                    <p>Описание продукта</p>
                    <div class="product-price">2 490 ₽</div>
                </div>
            </div>
            <div class="product-card">
                <img src="https://via.placeholder.com/200x150" alt="Продукт" class="product-image">
                <div class="product-info">
                    <h3>Продукт 3</h3>
                    <p>Описание продукта</p>
                    <div class="product-price">1 290 ₽</div>
                </div>
            </div>
        </div>

        <h2>Адаптивная навигация</h2>
        <div class="nav-container" style="width: 280px;">
            <ul class="nav-list">
                <li><a href="#" class="nav-item">Главная</a></li>
                <li><a href="#" class="nav-item">О нас</a></li>
                <li><a href="#" class="nav-item">Услуги</a></li>
                <li><a href="#" class="nav-item">Контакты</a></li>
            </ul>
        </div>

        <h2>Адаптивная форма</h2>
        <div class="form-container" style="width: 350px;">
            <form class="form-grid">
                <div class="form-group">
                    <label for="fname">Имя:</label>
                    <input type="text" id="fname" class="form-input">
                </div>
                <div class="form-group">
                    <label for="lname">Фамилия:</label>
                    <input type="text" id="lname" class="form-input">
                </div>
                <div class="form-group">
                    <label for="email">Email:</label>
                    <input type="email" id="email" class="form-input">
                </div>
                <div class="form-group">
                    <label for="phone">Телефон:</label>
                    <input type="tel" id="phone" class="form-input">
                </div>
            </form>
        </div>
    </div>
</body>
</html>
```

## CSS Scroll-Driven Animations

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CSS Scroll-Driven Animations</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            font-family: Arial, sans-serif;
        }

        .scroll-animation-demo {
            max-width: 800px;
            margin: 0 auto;
        }

        .scroll-container {
            height: 200vh; /* Длинный контент для прокрутки */
            position: relative;
        }

        .scroll-triggered-element {
            position: sticky;
            top: 50px;
            margin: 20px 0;
            padding: 20px;
            background: #f0f0f0;
            border-radius: 8px;
        }

        /* Пример анимации при прокрутке */
        .parallax-element {
            height: 300px;
            background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
            margin: 20px 0;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 2em;
            position: relative;
        }

        /* Анимация появления при прокрутке */
        .fade-in-element {
            opacity: 0;
            transform: translateY(50px);
            transition: opacity 0.6s ease, transform 0.6s ease;
        }

        .fade-in-element.visible {
            opacity: 1;
            transform: translateY(0);
        }

        /* Прогресс-бар прокрутки */
        .scroll-progress {
            position: fixed;
            top: 0;
            left: 0;
            width: 0%;
            height: 4px;
            background: #007acc;
            z-index: 1000;
            transition: width 0.1s ease;
        }

        /* Секции для демонстрации */
        .demo-section {
            height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5em;
            margin: 20px 0;
            border-radius: 8px;
        }

        .section-1 { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); color: white; }
        .section-2 { background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); color: white; }
        .section-3 { background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%); color: white; }
        .section-4 { background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%); color: white; }

        /* Анимация заголовка при прокрутке */
        .scroll-animated-title {
            font-size: 3em;
            text-align: center;
            margin: 50px 0;
            opacity: 0.3;
            transition: all 0.3s ease;
        }

        .scroll-animated-title.scrolled {
            opacity: 1;
            transform: scale(1.1);
        }

        /* Параллакс с CSS */
        .parallax-container {
            position: relative;
            overflow: hidden;
            height: 400px;
            margin: 20px 0;
        }

        .parallax-background {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: url('https://via.placeholder.com/1200x400/4ECDC4/FFFFFF?text=Background') center/cover;
            transform: translateZ(0);
            will-change: transform;
        }

        .parallax-foreground {
            position: relative;
            z-index: 2;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100%;
            color: white;
            text-align: center;
            padding: 20px;
        }

        /* Индикаторы прокрутки */
        .scroll-indicator {
            position: fixed;
            right: 20px;
            top: 50%;
            transform: translateY(-50%);
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .indicator-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #ccc;
            transition: background 0.3s;
        }

        .indicator-dot.active {
            background: #007acc;
        }

        /* Анимация при скролле с использованием view-based анимаций */
        .scroll-reveal {
            opacity: 0;
            transform: translateY(30px);
            transition: all 0.8s ease;
        }

        .scroll-reveal.revealed {
            opacity: 1;
            transform: translateY(0);
        }

        /* Стили для демонстрации */
        .content-block {
            background: white;
            padding: 30px;
            margin: 20px 0;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <div class="scroll-animation-container">
        <h1>CSS Scroll-Driven Animations</h1>

        <div class="scroll-progress" id="scroll-progress"></div>

        <div class="scroll-indicator" id="scroll-indicator">
            <div class="indicator-dot active" data-section="1"></div>
            <div class="indicator-dot" data-section="2"></div>
            <div class="indicator-dot" data-section="3"></div>
            <div class="indicator-dot" data-section="4"></div>
        </div>

        <div class="scroll-animated-title" id="animated-title">Прокрутите страницу</div>

        <div class="content-block">
            <h2>Демонстрация Scroll-Driven Animations</h2>
            <p>Эта страница демонстрирует различные техники анимации при прокрутке.</p>
        </div>

        <div class="demo-section section-1" id="section1">
            <h3>Секция 1</h3>
        </div>

        <div class="content-block scroll-reveal">
            <h3>Контент, появляющийся при прокрутке</h3>
            <p>Этот контент постепенно появляется при прокрутке страницы до него.</p>
        </div>

        <div class="demo-section section-2" id="section2">
            <h3>Секция 2</h3>
        </div>

        <div class="parallax-container">
            <div class="parallax-background" id="parallax-bg"></div>
            <div class="parallax-foreground">
                <h3>Параллакс-эффект</h3>
            </div>
        </div>

        <div class="demo-section section-3" id="section3">
            <h3>Секция 3</h3>
        </div>

        <div class="content-block scroll-reveal">
            <h3>Еще один элемент с анимацией</h3>
            <p>Этот элемент также появляется при прокрутке до него.</p>
        </div>

        <div class="demo-section section-4" id="section4">
            <h3>Секция 4</h3>
        </div>
    </div>

    <script>
        // Анимация прогресса прокрутки
        function updateScrollProgress() {
            const scrollTop = window.pageYOffset;
            const docHeight = document.documentElement.scrollHeight - window.innerHeight;
            const scrollPercent = (scrollTop / docHeight) * 100;
            
            document.getElementById('scroll-progress').style.width = scrollPercent + '%';
        }

        // Анимация заголовка при прокрутке
        function animateTitleOnScroll() {
            const title = document.getElementById('animated-title');
            const scrollPosition = window.pageYOffset;
            
            if (scrollPosition > 100) {
                title.classList.add('scrolled');
            } else {
                title.classList.remove('scrolled');
            }
        }

        // Анимация параллакса
        function parallaxOnScroll() {
            const parallaxBg = document.getElementById('parallax-bg');
            const scrollPosition = window.pageYOffset;
            
            // Двигаем фон медленнее, чем прокрутка
            const parallaxSpeed = 0.5;
            const yPos = -(scrollPosition * parallaxSpeed);
            
            parallaxBg.style.transform = `translateY(${yPos}px)`;
        }

        // Анимация появления элементов
        function revealOnScroll() {
            const elements = document.querySelectorAll('.scroll-reveal');
            
            elements.forEach(element => {
                const elementTop = element.getBoundingClientRect().top;
                const elementVisible = 150; // Появляется за 150px до появления
                
                if (elementTop < window.innerHeight - elementVisible) {
                    element.classList.add('revealed');
                }
            });
        }

        // Индикаторы секций
        function updateSectionIndicators() {
            const sections = document.querySelectorAll('.demo-section');
            const indicatorDots = document.querySelectorAll('.indicator-dot');
            const scrollPosition = window.pageYOffset + window.innerHeight / 2;
            
            sections.forEach((section, index) => {
                const sectionTop = section.offsetTop;
                const sectionBottom = sectionTop + section.offsetHeight;
                
                if (scrollPosition >= sectionTop && scrollPosition < sectionBottom) {
                    indicatorDots.forEach(dot => dot.classList.remove('active'));
                    indicatorDots[index].classList.add('active');
                }
            });
        }

        // Обработчики событий
        window.addEventListener('scroll', () => {
            updateScrollProgress();
            animateTitleOnScroll();
            parallaxOnScroll();
            revealOnScroll();
            updateSectionIndicators();
        });

        // Инициализация
        window.addEventListener('load', () => {
            updateScrollProgress();
            revealOnScroll(); // Проверяем элементы при загрузке
        });
    </script>
</body>
</html>
```

## Практические примеры продвинутых возможностей

### Темизация с CSS Custom Properties и JavaScript
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Продвинутая темизация</title>
    <style>
        :root {
            /* Базовые цвета темы */
            --color-primary: #007acc;
            --color-primary-dark: #005a9e;
            --color-primary-light: #42a5f5;
            --color-secondary: #6c757d;
            --color-success: #28a745;
            --color-danger: #dc3545;
            --color-warning: #ffc107;
            --color-info: #17a2b8;

            /* Цвета фона */
            --bg-body: #ffffff;
            --bg-surface: #f8f9fa;
            --bg-card: #ffffff;
            --bg-header: #f8f9fa;

            /* Цвета текста */
            --text-primary: #212529;
            --text-secondary: #6c757d;
            --text-muted: #868e96;
            --text-inverted: #ffffff;

            /* Цвета границ */
            --border-color: #dee2e6;
            --border-focus: #80bdff;

            /* Типографика */
            --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            --font-size-xs: 0.75rem;
            --font-size-sm: 0.875rem;
            --font-size-base: 1rem;
            --font-size-lg: 1.125rem;
            --font-size-xl: 1.25rem;
            --font-size-xxl: 1.5rem;

            /* Размеры */
            --border-radius: 0.25rem;
            --border-radius-sm: 0.2rem;
            --border-radius-lg: 0.3rem;
            --spacing-xs: 0.25rem;
            --spacing-sm: 0.5rem;
            --spacing-base: 1rem;
            --spacing-lg: 1.5rem;
            --spacing-xl: 2rem;

            /* Тени */
            --shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
            --shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
            --shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);

            /* Анимации */
            --transition-base: all 0.2s ease-in-out;
            --transition-fast: all 0.1s ease-in-out;
            --transition-slow: all 0.4s ease-in-out;
        }

        /* Темная тема */
        .theme-dark {
            --color-primary: #2196f3;
            --color-primary-dark: #1976d2;
            --color-primary-light: #64b5f6;
            --color-secondary: #868e96;
            --color-success: #4caf50;
            --color-danger: #f44336;
            --color-warning: #ff9800;
            --color-info: #2196f3;

            --bg-body: #121212;
            --bg-surface: #1e1e1e;
            --bg-card: #2d2d2d;
            --bg-header: #1e1e1e;

            --text-primary: #e0e0e0;
            --text-secondary: #b0b0b0;
            --text-muted: #888888;
            --text-inverted: #000000;

            --border-color: #444444;
            --border-focus: #42a5f5;
        }

        /* Тема с высоким контрастом */
        .theme-high-contrast {
            --color-primary: #0000ff;
            --color-primary-dark: #0000cc;
            --color-primary-light: #3333ff;
            --color-secondary: #000000;
            --color-success: #008000;
            --color-danger: #ff0000;
            --color-warning: #ffa500;
            --color-info: #0000ff;

            --bg-body: #ffffff;
            --bg-surface: #ffffff;
            --bg-card: #ffffff;
            --bg-header: #ffffff;

            --text-primary: #000000;
            --text-secondary: #000000;
            --text-muted: #000000;
            --text-inverted: #ffffff;

            --border-color: #000000;
            --border-focus: #0000ff;

            --shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.5);
            --shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.7);
            --shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.9);
        }

        /* Тема с оттенками серого */
        .theme-grayscale {
            --color-primary: #607d8b;
            --color-primary-dark: #455a64;
            --color-primary-light: #78909c;
            --color-secondary: #90a4ae;
            --color-success: #8bc34a;
            --color-danger: #ff5252;
            --color-warning: #ffd54f;
            --color-info: #80deea;

            --bg-body: #fafafa;
            --bg-surface: #eceff1;
            --bg-card: #ffffff;
            --bg-header: #e0e0e0;

            --text-primary: #263238;
            --text-secondary: #546e7a;
            --text-muted: #78909c;
            --text-inverted: #ffffff;

            --border-color: #cfd8dc;
            --border-focus: #90a4ae;
        }

        /* Тема с пользовательскими цветами */
        .theme-custom {
            --color-primary: var(--custom-primary, #007acc);
            --color-primary-dark: var(--custom-primary-dark, #005a9e);
            --color-secondary: var(--custom-secondary, #6c757d);
            --bg-body: var(--custom-bg, #ffffff);
            --text-primary: var(--custom-text, #212529);
        }

        /* Основные стили */
        body {
            background-color: var(--bg-body);
            color: var(--text-primary);
            font-family: var(--font-family);
            margin: 0;
            padding: 20px;
            transition: background-color 0.3s, color 0.3s;
        }

        .theme-container {
            max-width: 1200px;
            margin: 0 auto;
        }

        .theme-selector {
            margin-bottom: 30px;
            padding: 20px;
            background-color: var(--bg-surface);
            border-radius: var(--border-radius);
        }

        .theme-buttons {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-top: 15px;
        }

        .theme-btn {
            padding: 10px 20px;
            border: 1px solid var(--border-color);
            background-color: var(--bg-card);
            color: var(--text-primary);
            border-radius: var(--border-radius);
            cursor: pointer;
            transition: var(--transition-base);
        }

        .theme-btn:hover {
            background-color: var(--color-primary);
            color: var(--text-inverted);
        }

        .theme-btn.active {
            background-color: var(--color-primary);
            color: var(--text-inverted);
            border-color: var(--color-primary);
        }

        .component-demo {
            margin-bottom: 30px;
            padding: 20px;
            background-color: var(--bg-card);
            border: 1px solid var(--border-color);
            border-radius: var(--border-radius);
            box-shadow: var(--shadow);
        }

        .demo-title {
            color: var(--text-primary);
            margin-top: 0;
            margin-bottom: 15px;
        }

        .button {
            padding: var(--spacing-sm) var(--spacing-base);
            background-color: var(--color-primary);
            color: var(--text-inverted);
            border: none;
            border-radius: var(--border-radius);
            cursor: pointer;
            font-size: var(--font-size-base);
            transition: var(--transition-base);
        }

        .button:hover {
            background-color: var(--color-primary-dark);
            transform: translateY(-2px);
            box-shadow: var(--shadow);
        }

        .button:disabled {
            background-color: var(--color-secondary);
            cursor: not-allowed;
            opacity: 0.6;
        }

        .card {
            background-color: var(--bg-card);
            color: var(--text-primary);
            border: 1px solid var(--border-color);
            border-radius: var(--border-radius);
            padding: var(--spacing-lg);
            margin-bottom: var(--spacing-base);
            box-shadow: var(--shadow-sm);
            transition: var(--transition-base);
        }

        .card:hover {
            box-shadow: var(--shadow);
        }

        .input-field {
            width: 100%;
            padding: var(--spacing-sm);
            background-color: var(--bg-surface);
            color: var(--text-primary);
            border: 1px solid var(--border-color);
            border-radius: var(--border-radius);
            font-size: var(--font-size-base);
            transition: var(--transition-base);
        }

        .input-field:focus {
            outline: none;
            border-color: var(--border-focus);
            box-shadow: 0 0 0 0.2rem rgba(0, 122, 204, 0.25);
        }

        .progress-bar {
            height: 8px;
            background-color: var(--bg-surface);
            border-radius: 4px;
            overflow: hidden;
        }

        .progress-fill {
            height: 100%;
            background-color: var(--color-primary);
            width: 0%;
            transition: width 0.3s ease;
        }

        .color-palette {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 15px;
            margin-top: 20px;
        }

        .color-swatch {
            padding: 20px;
            border-radius: var(--border-radius);
            color: white;
            text-align: center;
            font-weight: bold;
        }

        .swatch-primary { background-color: var(--color-primary); }
        .swatch-primary-dark { background-color: var(--color-primary-dark); }
        .swatch-secondary { background-color: var(--color-secondary); }
        .swatch-success { background-color: var(--color-success); }
        .swatch-danger { background-color: var(--color-danger); }
        .swatch-warning { background-color: var(--color-warning); }
        .swatch-info { background-color: var(--color-info); }
    </style>
</head>
<body>
    <div class="theme-container">
        <h1>Продвинутая темизация с CSS Custom Properties</h1>

        <div class="theme-selector">
            <h2>Выберите тему:</h2>
            <div class="theme-buttons">
                <button class="theme-btn active" onclick="setTheme('light')">Светлая</button>
                <button class="theme-btn" onclick="setTheme('dark')">Темная</button>
                <button class="theme-btn" onclick="setTheme('high-contrast')">Высокий контраст</button>
                <button class="theme-btn" onclick="setTheme('grayscale')">Оттенки серого</button>
                <button class="theme-btn" onclick="showCustomThemeDialog()">Пользовательская</button>
            </div>
        </div>

        <div class="component-demo">
            <h3 class="demo-title">Демонстрация компонентов</h3>

            <div class="card">
                <h4>Карточка компонента</h4>
                <p>Этот компонент адаптируется к выбранной теме.</p>
                <button class="button">Пример кнопки</button>
            </div>

            <div class="card">
                <h4>Форма ввода</h4>
                <input type="text" class="input-field" placeholder="Поле ввода с темой">
            </div>

            <div class="card">
                <h4>Прогресс-бар</h4>
                <div class="progress-bar">
                    <div class="progress-fill" id="demo-progress" style="width: 65%;"></div>
                </div>
            </div>
        </div>

        <div class="component-demo">
            <h3 class="demo-title">Палитра цветов темы</h3>
            <div class="color-palette">
                <div class="color-swatch swatch-primary">Primary</div>
                <div class="color-swatch swatch-primary-dark">Primary Dark</div>
                <div class="color-swatch swatch-secondary">Secondary</div>
                <div class="color-swatch swatch-success">Success</div>
                <div class="color-swatch swatch-danger">Danger</div>
                <div class="color-swatch swatch-warning">Warning</div>
                <div class="color-swatch swatch-info">Info</div>
            </div>
        </div>
    </div>

    <!-- Модальное окно для пользовательской темы -->
    <div id="custom-theme-modal" class="modal" style="display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); z-index: 1000;">
        <div style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: var(--bg-card); padding: 30px; border-radius: var(--border-radius); min-width: 400px;">
            <h3>Пользовательская тема</h3>
            
            <div class="form-group">
                <label>Основной цвет:</label>
                <input type="color" id="primary-color" value="#007acc">
            </div>
            
            <div class="form-group">
                <label>Темный оттенок основного:</label>
                <input type="color" id="primary-dark" value="#005a9e">
            </div>
            
            <div class="form-group">
                <label>Вторичный цвет:</label>
                <input type="color" id="secondary-color" value="#6c757d">
            </div>
            
            <div class="form-group">
                <label>Цвет фона:</label>
                <input type="color" id="bg-color" value="#ffffff">
            </div>
            
            <div class="form-group">
                <label>Цвет текста:</label>
                <input type="color" id="text-color" value="#212529">
            </div>
            
            <div class="form-actions" style="margin-top: 20px;">
                <button class="button" onclick="applyCustomTheme()">Применить</button>
                <button class="button" style="background-color: var(--color-secondary);" onclick="closeCustomThemeDialog()">Отмена</button>
            </div>
        </div>
    </div>

    <script>
        class ThemeManager {
            constructor() {
                this.currentTheme = localStorage.getItem('selected-theme') || 'light';
                this.applyTheme(this.currentTheme);
                this.updateActiveButton();
            }

            setTheme(themeName) {
                this.currentTheme = themeName;
                this.applyTheme(themeName);
                this.updateActiveButton();
                localStorage.setItem('selected-theme', themeName);
            }

            applyTheme(themeName) {
                document.body.className = '';
                
                if (themeName !== 'light') {
                    document.body.classList.add(`theme-${themeName}`);
                }

                // Для пользовательской темы устанавливаем переменные
                if (themeName === 'custom') {
                    this.applyCustomVariables();
                }
            }

            updateActiveButton() {
                document.querySelectorAll('.theme-btn').forEach(btn => {
                    btn.classList.remove('active');
                });

                const activeBtn = document.querySelector(`.theme-btn[onclick*="${this.currentTheme}"]`);
                if (activeBtn) {
                    activeBtn.classList.add('active');
                }
            }

            showCustomThemeDialog() {
                document.getElementById('custom-theme-modal').style.display = 'block';
            }

            closeCustomThemeDialog() {
                document.getElementById('custom-theme-modal').style.display = 'none';
            }

            applyCustomTheme() {
                const primary = document.getElementById('primary-color').value;
                const primaryDark = document.getElementById('primary-dark').value;
                const secondary = document.getElementById('secondary-color').value;
                const bg = document.getElementById('bg-color').value;
                const text = document.getElementById('text-color').value;

                document.body.style.setProperty('--custom-primary', primary);
                document.body.style.setProperty('--custom-primary-dark', primaryDark);
                document.body.style.setProperty('--custom-secondary', secondary);
                document.body.style.setProperty('--custom-bg', bg);
                document.body.style.setProperty('--custom-text', text);

                document.body.classList.add('theme-custom');
                this.currentTheme = 'custom';
                localStorage.setItem('selected-theme', 'custom');
                this.updateActiveButton();
                
                this.closeCustomThemeDialog();
            }

            applyCustomVariables() {
                const customVars = localStorage.getItem('custom-theme-vars');
                if (customVars) {
                    const vars = JSON.parse(customVars);
                    Object.entries(vars).forEach(([key, value]) => {
                        document.documentElement.style.setProperty(key, value);
                    });
                }
            }
        }

        // Инициализация менеджера тем
        const themeManager = new ThemeManager();

        // Глобальные функции для кнопок
        function setTheme(theme) {
            themeManager.setTheme(theme);
        }

        function showCustomThemeDialog() {
            themeManager.showCustomThemeDialog();
        }

        function closeCustomThemeDialog() {
            themeManager.closeCustomThemeDialog();
        }

        function applyCustomTheme() {
            themeManager.applyCustomTheme();
        }
    </script>
</body>
</html>
```

## Заключение

Современные возможности CSS, включая переменные, функции, Grid, Container Queries и Scroll-Driven Animations, предоставляют разработчикам мощные инструменты для создания адаптивных, доступных и высокопроизводительных веб-приложений. Эти технологии позволяют:

1. **CSS Custom Properties** - создавать гибкие и настраиваемые темы
2. **CSS Functions** - выполнять вычисления и создавать адаптивные размеры
3. **CSS Grid** - создавать сложные и адаптивные макеты
4. **Container Queries** - создавать компоненты, адаптирующиеся к своим контейнерам
5. **Scroll-Driven Animations** - создавать интерактивные эффекты на основе прокрутки
6. **Advanced Selectors** - более точную и гибкую стилизацию
7. **Modern Layout Techniques** - эффективные и семантически правильные макеты

Эти возможности позволяют создавать современные веб-сайты, которые не только выглядят профессионально, но и обеспечивают отличный пользовательский опыт на различных устройствах и с различными потребностями пользователей.

Ключевые аспекты современных возможностей CSS:
1. Поддержка доступности и семантики
2. Производительность и оптимизация
3. Адаптивность и отзывчивость
4. Повторное использование и модульность
5. Совместимость с различными браузерами
6. Интеграция с JavaScript и другими технологиями

Использование этих возможностей позволяет создавать веб-сайты, которые соответствуют современным стандартам и обеспечивают качественный пользовательский опыт.

## Следующие темы
- [[HTML/Продвинутые темы/Шаблоны и веб-компоненты]]
- [[HTML/Интеграция с JavaScript]]
- [[HTML/Производительность]]

## Теги
#css #advanced-features #custom-properties #css-grid #container-queries #scroll-driven-animations #web-development #frontend #styling #modern-css #theming #responsive-design #accessibility #performance #css-functions #calc #clamp #min #max #variables #web-components
# HTML Производительность: Техники оптимизации

Оптимизация производительности HTML-страниц - это комплексный процесс, направленный на улучшение скорости загрузки, времени до интерактивности и общего пользовательского опыта. Современные техники оптимизации включают в себя оптимизацию DOM, управление ресурсами, эффективное кеширование и использование передовых веб-технологий.

## Основы оптимизации производительности

### Критические пути рендеринга

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Оптимизация критического пути рендеринга</title>
    
    <!-- Критические стили встроены для быстрого рендеринга -->
    <style>
        /* Критические стили для верхней части страницы */
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #ffffff;
            color: #333333;
        }
        
        .above-fold {
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            text-align: center;
        }
        
        .hero-content h1 {
            font-size: 2.5rem;
            margin-bottom: 1rem;
        }
        
        .hero-content p {
            font-size: 1.2rem;
            margin-bottom: 2rem;
        }
        
        .cta-button {
            display: inline-block;
            padding: 1rem 2rem;
            background-color: #ff6b6b;
            color: white;
            text-decoration: none;
            border-radius: 4px;
            font-weight: bold;
        }
    </style>
    
    <!-- Загрузка некритических стилей асинхронно -->
    <link rel="preload" href="non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
    <noscript><link rel="stylesheet" href="non-critical.css"></noscript>
    
    <!-- Предзагрузка критических ресурсов -->
    <link rel="preload" href="hero-image.jpg" as="image">
    <link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
    
    <!-- DNS prefetch для внешних ресурсов -->
    <link rel="dns-prefetch" href="//fonts.googleapis.com">
    <link rel="dns-prefetch" href="//analytics.google.com">
    <link rel="preconnect" href="https://fonts.googleapis.com" crossorigin>
</head>
<body>
    <!-- Контент выше складки (above the fold) -->
    <div class="above-fold">
        <div class="hero-content">
            <h1>Быстрая загрузка страницы</h1>
            <p>Этот контент загружается мгновенно благодаря оптимизации критического пути рендеринга</p>
            <a href="#features" class="cta-button">Узнать больше</a>
        </div>
    </div>
    
    <!-- Остальной контент -->
    <div class="below-fold">
        <section id="features">
            <h2>Особенности</h2>
            <div class="feature-grid">
                <div class="feature-card">
                    <h3>Быстрая загрузка</h3>
                    <p>Оптимизированные ресурсы и структура документа обеспечивают быструю загрузку</p>
                </div>
                <div class="feature-card">
                    <h3>Минимизация ресурсов</h3>
                    <p>Эффективное сжатие и объединение файлов</p>
                </div>
                <div class="feature-card">
                    <h3>Кеширование</h3>
                    <p>Использование различных уровней кеширования для ускорения повторных посещений</p>
                </div>
            </div>
        </section>
        
        <section id="content">
            <h2>Дополнительный контент</h2>
            <p>Этот контент загружается после основного содержимого.</p>
        </section>
    </div>
    
    <!-- Скрипты подключаются в конце body для улучшения загрузки -->
    <script src="critical.js" defer></script>
    <script src="non-critical.js" async></script>
</body>
</html>
```

### Оптимизация DOM

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оптимизация DOM</title>
</head>
<body>
    <h1>Оптимизация DOM</h1>
    
    <div id="list-container"></div>
    <button id="add-items-btn">Добавить 1000 элементов</button>
    <button id="clear-list-btn">Очистить список</button>
    <button id="benchmark-btn">Тест производительности</button>
    
    <div id="results"></div>

    <script>
        class DOMOptimizer {
            constructor() {
                this.container = document.getElementById('list-container');
                this.resultsElement = document.getElementById('results');
            }
            
            // Плохо: многократные изменения DOM
            addItemsInefficiently(count) {
                console.time('inefficient');
                
                for (let i = 0; i < count; i++) {
                    const item = document.createElement('div');
                    item.textContent = `Элемент ${i + 1}`;
                    item.className = 'list-item';
                    this.container.appendChild(item);
                }
                
                console.timeEnd('inefficient');
            }
            
            // Хорошо: использование DocumentFragment
            addItemsEfficiently(count) {
                console.time('efficient');
                
                const fragment = document.createDocumentFragment();
                
                for (let i = 0; i < count; i++) {
                    const item = document.createElement('div');
                    item.textContent = `Элемент ${i + 1}`;
                    item.className = 'list-item';
                    fragment.appendChild(item);
                }
                
                this.container.appendChild(fragment);
                console.timeEnd('efficient');
            }
            
            // Хорошо: использование innerHTML для больших изменений
            addItemsWithInnerHTML(count) {
                console.time('innerhtml');
                
                let html = '';
                for (let i = 0; i < count; i++) {
                    html += `<div class="list-item">Элемент ${i + 1}</div>`;
                }
                
                this.container.innerHTML = html;
                console.timeEnd('innerhtml');
            }
            
            // Использование requestAnimationFrame для анимаций
            animateElement(element, targetPosition) {
                const startPosition = element.getBoundingClientRect().top;
                const distance = targetPosition - startPosition;
                const duration = 1000;
                let startTime = null;
                
                const animation = (currentTime) => {
                    if (!startTime) startTime = currentTime;
                    
                    const elapsed = currentTime - startTime;
                    const progress = Math.min(elapsed / duration, 1);
                    
                    // Используем transform вместо изменения layout свойств
                    element.style.transform = `translateY(${distance * progress}px)`;
                    
                    if (progress < 1) {
                        requestAnimationFrame(animation);
                    }
                };
                
                requestAnimationFrame(animation);
            }
            
            // Оптимизация перерисовки
            batchDOMUpdates() {
                // Группировка нескольких DOM изменений
                requestAnimationFrame(() => {
                    // Все изменения происходят в одном цикле рендеринга
                    this.updateElement1();
                    this.updateElement2();
                    this.updateElement3();
                });
            }
            
            // Использование CSS вместо JavaScript для анимаций
            useCSSTransitions() {
                // CSS transitions более эффективны, чем JavaScript анимации
                // Потому что могут быть аппаратно ускорены
                document.body.classList.add('transition-enabled');
            }
            
            // Оптимизация событий с делегированием
            setupEventDelegation() {
                this.container.addEventListener('click', (e) => {
                    if (e.target.classList.contains('list-item')) {
                        e.target.classList.toggle('selected');
                    }
                });
            }
            
            // Использование Intersection Observer вместо scroll событий
            setupIntersectionObserver() {
                const observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            // Загрузка или обновление элемента при появлении в области видимости
                            this.processVisibleElement(entry.target);
                        }
                    });
                }, {
                    rootMargin: '50px' // Начинать обработку за 50px до появления
                });
                
                // Наблюдение за элементами
                document.querySelectorAll('.lazy-process').forEach(el => {
                    observer.observe(el);
                });
            }
            
            processVisibleElement(element) {
                // Обработка элемента, который стал видимым
                element.classList.add('processed');
                element.textContent = 'Обработано при появлении';
            }
            
            benchmarkTechniques() {
                const count = 1000;
                
                // Тест неэффективного метода
                this.container.innerHTML = '';
                this.addItemsInefficiently(count);
                
                // Тест эффективного метода
                this.container.innerHTML = '';
                this.addItemsEfficiently(count);
                
                // Тест innerHTML метода
                this.container.innerHTML = '';
                this.addItemsWithInnerHTML(count);
            }
        }
        
        // Инициализация
        const domOptimizer = new DOMOptimizer();
        
        document.getElementById('add-items-btn').addEventListener('click', () => {
            domOptimizer.addItemsEfficiently(1000);
        });
        
        document.getElementById('clear-list-btn').addEventListener('click', () => {
            domOptimizer.container.innerHTML = '';
        });
        
        document.getElementById('benchmark-btn').addEventListener('click', () => {
            domOptimizer.benchmarkTechniques();
        });
        
        // Настройка делегирования событий
        domOptimizer.setupEventDelegation();
    </script>
    
    <style>
        #list-container {
            max-height: 400px;
            overflow-y: auto;
            border: 1px solid #ddd;
            margin: 20px 0;
        }
        
        .list-item {
            padding: 10px;
            border-bottom: 1px solid #eee;
            cursor: pointer;
            transition: background-color 0.2s;
        }
        
        .list-item:hover {
            background-color: #f5f5f5;
        }
        
        .list-item.selected {
            background-color: #e3f2fd;
        }
        
        button {
            padding: 10px 20px;
            margin: 5px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .transition-enabled {
            transition: all 0.3s ease;
        }
    </style>
</body>
</html>
```

## Оптимизация загрузки ресурсов

### Resource Hints

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Resource Hints</title>
    
    <!-- Preload критических ресурсов -->
    <link rel="preload" href="critical.css" as="style">
    <link rel="preload" href="hero-image.jpg" as="image">
    <link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
    <link rel="preload" href="main.js" as="script">
    
    <!-- Prefetch потенциально нужных ресурсов -->
    <link rel="prefetch" href="next-page.html">
    <link rel="prefetch" href="secondary-font.woff2">
    
    <!-- Preconnect для внешних доменов -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link rel="preconnect" href="https://api.example.com">
    <link rel="preconnect" href="https://cdn.example.com">
    
    <!-- DNS Prefetch -->
    <link rel="dns-prefetch" href="//analytics.google.com">
    <link rel="dns-prefetch" href="//maps.googleapis.com">
    <link rel="dns-prefetch" href="//api.external-service.com">
    
    <!-- Prerender следующей страницы (если уверен в переходе) -->
    <!-- <link rel="prerender" href="next-page.html"> -->
    
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap" rel="stylesheet">
</head>
<body>
    <h1>Resource Hints для оптимизации производительности</h1>
    
    <div class="content">
        <p>Resource Hints позволяют браузеру заранее подготовиться к загрузке ресурсов.</p>
    </div>
</body>
</html>
```

### Lazy Loading и Dynamic Imports

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Ленивая загрузка</title>
</head>
<body>
    <h1>Ленивая загрузка ресурсов</h1>
    
    <!-- Ленивая загрузка изображений -->
    <img src="placeholder.jpg" 
         data-src="actual-image.jpg" 
         alt="Изображение" 
         class="lazy-image"
         loading="lazy">
    
    <!-- Ленивая загрузка iframe -->
    <iframe src="about:blank" 
            data-src="https://example.com/embed" 
            loading="lazy"
            width="560" 
            height="315">
    </iframe>
    
    <!-- Контейнер для динамического контента -->
    <div id="dynamic-content-container"></div>
    
    <button id="load-content-btn">Загрузить дополнительный контент</button>

    <script>
        class LazyLoader {
            constructor() {
                this.setupLazyLoading();
                this.setupDynamicImports();
            }
            
            setupLazyLoading() {
                // Использование Intersection Observer для ленивой загрузки изображений
                const imageObserver = new IntersectionObserver((entries, observer) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            this.loadImage(entry.target);
                            observer.unobserve(entry.target);
                        }
                    });
                }, {
                    rootMargin: '50px' // Начинать загрузку за 50px до появления
                });
                
                // Наблюдение за изображениями с классом lazy-image
                document.querySelectorAll('img.lazy-image').forEach(img => {
                    imageObserver.observe(img);
                });
                
                // Ленивая загрузка iframe
                const iframeObserver = new IntersectionObserver((entries, observer) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            this.loadIframe(entry.target);
                            observer.unobserve(entry.target);
                        }
                    });
                });
                
                document.querySelectorAll('iframe[data-src]').forEach(iframe => {
                    iframeObserver.observe(iframe);
                });
            }
            
            loadImage(imgElement) {
                const src = imgElement.dataset.src;
                if (src) {
                    imgElement.src = src;
                    imgElement.removeAttribute('data-src');
                    imgElement.classList.add('loaded');
                }
            }
            
            loadIframe(iframeElement) {
                const src = iframeElement.dataset.src;
                if (src) {
                    iframeElement.src = src;
                    iframeElement.removeAttribute('data-src');
                }
            }
            
            setupDynamicImports() {
                document.getElementById('load-content-btn').addEventListener('click', async () => {
                    await this.loadAdditionalContent();
                });
            }
            
            async loadAdditionalContent() {
                try {
                    // Динамический импорт модуля
                    const { default: ContentLoader } = await import('./modules/content-loader.js');
                    const loader = new ContentLoader();
                    await loader.loadContent();
                } catch (error) {
                    console.error('Ошибка загрузки дополнительного контента:', error);
                    
                    // Резервный вариант
                    this.loadFallbackContent();
                }
            }
            
            loadFallbackContent() {
                const container = document.getElementById('dynamic-content-container');
                container.innerHTML = '<p>Дополнительный контент не может быть загружен</p>';
            }
            
            // Оптимизация загрузки изображений с разными размерами
            loadResponsiveImage(imageElement, sources) {
                const observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            const devicePixelRatio = window.devicePixelRatio || 1;
                            const screenWidth = window.screen.width;
                            
                            // Выбор подходящего изображения
                            let selectedSource = sources[0]; // по умолчанию
                            
                            sources.forEach(source => {
                                if (screenWidth <= source.maxWidth && devicePixelRatio <= source.maxDPR) {
                                    selectedSource = source;
                                }
                            });
                            
                            imageElement.src = selectedSource.src;
                            imageElement.classList.add('loaded');
                            
                            observer.unobserve(imageElement);
                        }
                    });
                });
                
                observer.observe(imageElement);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new LazyLoader();
        });
    </script>
    
    <style>
        .lazy-image {
            opacity: 0;
            transition: opacity 0.3s ease;
        }
        
        .lazy-image.loaded {
            opacity: 1;
        }
        
        .placeholder {
            background-color: #f0f0f0;
            min-height: 200px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        .loading-spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #007acc;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</body>
</html>
```

## Оптимизация изображений

### Современные форматы и техники

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оптимизация изображений</title>
</head>
<body>
    <h1>Оптимизация изображений</h1>
    
    <!-- Использование современных форматов -->
    <picture>
        <source srcset="image.avif" type="image/avif">
        <source srcset="image.webp" type="image/webp">
        <source srcset="image.jpg" type="image/jpeg">
        <img src="image.jpg" 
             alt="Описание изображения" 
             loading="lazy"
             width="800" 
             height="600">
    </picture>
    
    <!-- Responsive images -->
    <picture>
        <source media="(max-width: 768px)" srcset="mobile.jpg">
        <source media="(max-width: 1024px)" srcset="tablet.jpg">
        <img src="desktop.jpg" 
             alt="Адаптивное изображение"
             sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
             loading="lazy">
    </picture>
    
    <!-- Изображение с низким качеством для предварительного просмотра -->
    <div class="image-container">
        <img class="low-quality-preview" 
             src="low-quality.jpg" 
             data-src="high-quality.jpg" 
             alt="Изображение с предварительным просмотром"
             loading="lazy">
    </div>
    
    <!-- Изображение с progressive загрузкой -->
    <img id="progressive-image" 
         src="placeholder.jpg" 
         data-src="full-quality.jpg" 
         alt="Прогрессивное изображение"
         width="800" 
         height="600">

    <script>
        class ImageOptimizer {
            constructor() {
                this.setupProgressiveLoading();
                this.setupLowQualityPreviews();
            }
            
            setupProgressiveLoading() {
                const imageObserver = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            this.progressiveLoadImage(entry.target);
                            imageObserver.unobserve(entry.target);
                        }
                    });
                });
                
                document.querySelectorAll('#progressive-image').forEach(img => {
                    imageObserver.observe(img);
                });
            }
            
            progressiveLoadImage(imgElement) {
                const highQualitySrc = imgElement.dataset.src;
                
                if (highQualitySrc) {
                    // Показываем низкокачественный плейсхолдер
                    imgElement.style.filter = 'blur(5px)';
                    
                    const highQualityImg = new Image();
                    highQualityImg.onload = () => {
                        // Заменяем изображение на высококачественное
                        imgElement.src = highQualitySrc;
                        imgElement.style.filter = 'none';
                        imgElement.classList.add('loaded');
                    };
                    
                    highQualityImg.src = highQualitySrc;
                }
            }
            
            setupLowQualityPreviews() {
                const previewObserver = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            this.loadHighQualityImage(entry.target);
                            previewObserver.unobserve(entry.target);
                        }
                    });
                });
                
                document.querySelectorAll('.low-quality-preview').forEach(img => {
                    previewObserver.observe(img);
                });
            }
            
            loadHighQualityImage(imgElement) {
                const highQualitySrc = imgElement.dataset.src;
                
                if (highQualitySrc) {
                    imgElement.src = highQualitySrc;
                    imgElement.classList.add('loaded');
                }
            }
            
            // Оптимизация для разных плотностей экрана
            optimizeForDevicePixelRatio() {
                const images = document.querySelectorAll('img[data-src-2x]');
                
                if (window.devicePixelRatio > 1) {
                    images.forEach(img => {
                        if (img.dataset.src2x) {
                            img.src = img.dataset.src2x;
                        }
                    });
                }
            }
            
            // Генерация адаптивных изображений
            generateResponsiveImageSet(imageBase, sizes) {
                return sizes.map(size => {
                    return {
                        src: `${imageBase}_${size.width}x${size.height}.${size.format}`,
                        width: size.width,
                        height: size.height,
                        media: size.media
                    };
                });
            }
            
            // Использование picturefill для поддержки старых браузеров
            setupPictureFill() {
                // Для браузеров без поддержки picture элемента
                if (!window.HTMLPictureElement) {
                    // Подключаем polyfill
                    const script = document.createElement('script');
                    script.src = 'https://cdn.jsdelivr.net/npm/picturefill@3.0.3/dist/picturefill.min.js';
                    script.async = true;
                    document.head.appendChild(script);
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new ImageOptimizer();
        });
    </script>
    
    <style>
        .image-container {
            position: relative;
            width: 800px;
            height: 600px;
            overflow: hidden;
        }
        
        .low-quality-preview {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: filter 0.3s ease;
        }
        
        .low-quality-preview:not(.loaded) {
            filter: blur(10px);
        }
        
        .low-quality-preview.loaded {
            filter: none;
        }
        
        #progressive-image {
            width: 100%;
            height: auto;
            transition: filter 0.3s ease;
        }
        
        #progressive-image:not(.loaded) {
            filter: blur(5px);
        }
    </style>
</body>
</html>
```

## Оптимизация скриптов

### Модульная загрузка и оптимизация

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оптимизация скриптов</title>
</head>
<body>
    <h1>Оптимизация загрузки и выполнения скриптов</h1>
    
    <!-- Скрипты с различными атрибутами оптимизации -->
    <script src="critical.js" defer></script>
    <script src="non-critical.js" async></script>
    <script type="module" src="es-module.js"></script>
    <script nomodule src="legacy-bundle.js"></script>
    
    <!-- Inline critical script -->
    <script>
        // Критический скрипт, который должен выполниться немедленно
        (function() {
            'use strict';
            
            // Добавляем класс для индикации загрузки JavaScript
            document.documentElement.classList.add('js');
            
            // Устанавливаем тему на основе предпочтений пользователя
            if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
                document.documentElement.classList.add('dark-theme');
            }
            
            // Оптимизированная функция для первой загрузки
            function initializeCriticalFeatures() {
                // Инициализация критических функций
                setupServiceWorker();
                initializeEssentialEventListeners();
            }
            
            // Выполняем критическую инициализацию
            if (document.readyState === 'loading') {
                document.addEventListener('DOMContentLoaded', initializeCriticalFeatures);
            } else {
                initializeCriticalFeatures();
            }
            
            // Оптимизация Service Worker регистрации
            function setupServiceWorker() {
                if ('serviceWorker' in navigator) {
                    window.addEventListener('load', () => {
                        navigator.serviceWorker.register('/sw.js')
                            .then(registration => {
                                console.log('SW зарегистрирован: ', registration.scope);
                            })
                            .catch(err => {
                                console.log('SW регистрация не удалась: ', err);
                            });
                    });
                }
            }
            
            function initializeEssentialEventListeners() {
                // Инициализация только критических обработчиков событий
                document.addEventListener('click', handleEssentialClicks, true);
            }
            
            function handleEssentialClicks(event) {
                // Обработка только критических кликов
                if (event.target.matches('[data-critical-action]')) {
                    event.preventDefault();
                    executeCriticalAction(event.target.dataset.action);
                }
            }
            
            function executeCriticalAction(action) {
                // Выполнение критического действия
                console.log('Выполнение критического действия:', action);
            }
        })();
    </script>
    
    <div id="dynamic-content"></div>
    <button id="load-module-btn">Загрузить модуль</button>

    <script>
        class ScriptOptimizer {
            constructor() {
                this.modules = new Map();
                this.loadingPromises = new Map();
            }
            
            // Оптимизированная загрузка модулей
            async loadModule(modulePath) {
                if (this.modules.has(modulePath)) {
                    return this.modules.get(modulePath);
                }
                
                // Предотвращаем дублирование загрузок
                if (this.loadingPromises.has(modulePath)) {
                    return this.loadingPromises.get(modulePath);
                }
                
                const promise = import(modulePath);
                this.loadingPromises.set(modulePath, promise);
                
                try {
                    const module = await promise;
                    this.modules.set(modulePath, module);
                    return module;
                } finally {
                    this.loadingPromises.delete(modulePath);
                }
            }
            
            // Динамическая загрузка компонентов
            async loadComponent(componentName) {
                const componentPath = `./components/${componentName}.js`;
                
                try {
                    const { default: Component } = await this.loadModule(componentPath);
                    return new Component();
                } catch (error) {
                    console.error(`Ошибка загрузки компонента ${componentName}:`, error);
                    throw error;
                }
            }
            
            // Оптимизация выполнения скриптов
            executeAsync(task, priority = 'normal') {
                return new Promise((resolve, reject) => {
                    switch(priority) {
                        case 'high':
                            // Выполняем немедленно
                            try {
                                const result = task();
                                resolve(result);
                            } catch (error) {
                                reject(error);
                            }
                            break;
                        case 'normal':
                            // Выполняем в следующем тике
                            setTimeout(() => {
                                try {
                                    const result = task();
                                    resolve(result);
                                } catch (error) {
                                    reject(error);
                                }
                            }, 0);
                            break;
                        case 'low':
                            // Выполняем при простое (если поддерживается)
                            if ('requestIdleCallback' in window) {
                                requestIdleCallback(() => {
                                    try {
                                        const result = task();
                                        resolve(result);
                                    } catch (error) {
                                        reject(error);
                                    }
                                });
                            } else {
                                setTimeout(() => {
                                    try {
                                        const result = task();
                                        resolve(result);
                                    } catch (error) {
                                        reject(error);
                                    }
                                }, 1);
                            }
                            break;
                    }
                });
            }
            
            // Оптимизация памяти
            cleanupUnusedModules() {
                // Удаляем неиспользуемые модули (в теории, на практике сложнее)
                const unusedModules = [];
                
                this.modules.forEach((module, path) => {
                    if (!this.isModuleUsed(path)) {
                        unusedModules.push(path);
                    }
                });
                
                unusedModules.forEach(path => {
                    this.modules.delete(path);
                });
                
                return unusedModules.length;
            }
            
            isModuleUsed(path) {
                // Проверяем, используется ли модуль (упрощенная проверка)
                return document.querySelectorAll(`[data-module="${path}"]`).length > 0;
            }
            
            // Управление зависимостями
            async loadDependencies(dependencies) {
                const loaded = [];
                
                for (const dep of dependencies) {
                    try {
                        const module = await this.loadModule(dep.path);
                        loaded.push({ name: dep.name, module });
                        
                        // Инициализация зависимости
                        if (dep.init && typeof dep.init === 'function') {
                            await dep.init(module);
                        }
                    } catch (error) {
                        console.error(`Ошибка загрузки зависимости ${dep.name}:`, error);
                        
                        if (dep.fallback) {
                            // Используем резервный вариант
                            const fallback = await this.loadModule(dep.fallback);
                            loaded.push({ name: dep.name, module: fallback, isFallback: true });
                        }
                    }
                }
                
                return loaded;
            }
        }
        
        // Инициализация
        const scriptOptimizer = new ScriptOptimizer();
        
        document.getElementById('load-module-btn').addEventListener('click', async () => {
            try {
                const dynamicContent = document.getElementById('dynamic-content');
                dynamicContent.innerHTML = '<p>Загрузка модуля...</p>';
                
                const module = await scriptOptimizer.loadModule('./modules/dynamic-content.js');
                dynamicContent.innerHTML = await module.getContent();
            } catch (error) {
                console.error('Ошибка загрузки модуля:', error);
                document.getElementById('dynamic-content').innerHTML = '<p>Ошибка загрузки контента</p>';
            }
        });
    </script>
</body>
</html>
```

## Оптимизация с Web Workers

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оптимизация с Web Workers</title>
</head>
<body>
    <h1>Оптимизация с Web Workers</h1>
    
    <div class="worker-demo">
        <h2>Обработка данных в Web Worker</h2>
        
        <div class="controls">
            <input type="number" id="data-size" value="10000" placeholder="Размер данных">
            <button id="process-data-btn">Обработать данные</button>
            <button id="process-main-thread">Обработать в основном потоке</button>
        </div>
        
        <div class="results">
            <div id="worker-result"></div>
            <div id="main-thread-result"></div>
        </div>
    </div>

    <script>
        // Код Web Worker (в реальном приложении - отдельный файл)
        const workerCode = `
            // worker.js
            self.onmessage = function(e) {
                const { data, operation } = e.data;
                
                switch(operation) {
                    case 'sort':
                        // Сортировка данных
                        const sortedData = data.sort((a, b) => a - b);
                        self.postMessage({ operation: 'sort', result: sortedData });
                        break;
                        
                    case 'filter':
                        // Фильтрация данных
                        const filteredData = data.filter(num => num % 2 === 0);
                        self.postMessage({ operation: 'filter', result: filteredData });
                        break;
                        
                    case 'calculate':
                        // Вычисления
                        const sum = data.reduce((acc, curr) => acc + curr, 0);
                        const average = sum / data.length;
                        self.postMessage({ operation: 'calculate', result: { sum, average } });
                        break;
                        
                    default:
                        self.postMessage({ error: 'Неизвестная операция' });
                }
            };
        `;
        
        class WorkerOptimizer {
            constructor() {
                // Создание Web Worker из строки
                const blob = new Blob([workerCode], { type: 'application/javascript' });
                this.worker = new Worker(URL.createObjectURL(blob));
                
                this.setupWorkerMessaging();
            }
            
            setupWorkerMessaging() {
                this.worker.onmessage = (e) => {
                    const { operation, result, error } = e.data;
                    
                    if (error) {
                        console.error('Ошибка в Web Worker:', error);
                        return;
                    }
                    
                    this.displayWorkerResult(operation, result);
                };
            }
            
            processDataInWorker(data, operation) {
                this.worker.postMessage({ data, operation });
            }
            
            displayWorkerResult(operation, result) {
                const resultElement = document.getElementById('worker-result');
                
                resultElement.innerHTML = `
                    <h3>Результат работы Web Worker (${operation}):</h3>
                    <p>Время обработки: ${this.getProcessingTime()}</p>
                    <pre>${JSON.stringify(result, null, 2)}</pre>
                `;
            }
            
            getProcessingTime() {
                return new Date().toLocaleTimeString();
            }
            
            // Обработка в основном потоке (для сравнения)
            async processDataInMainThread(data, operation) {
                const startTime = performance.now();
                
                let result;
                switch(operation) {
                    case 'sort':
                        result = [...data].sort((a, b) => a - b);
                        break;
                    case 'filter':
                        result = data.filter(num => num % 2 === 0);
                        break;
                    case 'calculate':
                        const sum = data.reduce((acc, curr) => acc + curr, 0);
                        const average = sum / data.length;
                        result = { sum, average };
                        break;
                }
                
                const endTime = performance.now();
                const processingTime = (endTime - startTime).toFixed(2);
                
                const resultElement = document.getElementById('main-thread-result');
                resultElement.innerHTML = `
                    <h3>Результат работы в основном потоке (${operation}):</h3>
                    <p>Время обработки: ${processingTime}мс</p>
                    <pre>${JSON.stringify(result, null, 2)}</pre>
                `;
                
                return { result, processingTime };
            }
        }
        
        // Инициализация
        const workerOptimizer = new WorkerOptimizer();
        
        document.getElementById('process-data-btn').addEventListener('click', () => {
            const dataSize = parseInt(document.getElementById('data-size').value) || 10000;
            const testData = Array.from({ length: dataSize }, () => Math.floor(Math.random() * 1000));
            
            workerOptimizer.processDataInWorker(testData, 'sort');
            workerOptimizer.processDataInWorker(testData, 'filter');
            workerOptimizer.processDataInWorker(testData, 'calculate');
        });
        
        document.getElementById('process-main-thread').addEventListener('click', async () => {
            const dataSize = parseInt(document.getElementById('data-size').value) || 10000;
            const testData = Array.from({ length: dataSize }, () => Math.floor(Math.random() * 1000));
            
            await workerOptimizer.processDataInMainThread(testData, 'sort');
            await workerOptimizer.processDataInMainThread(testData, 'filter');
            await workerOptimizer.processDataInMainThread(testData, 'calculate');
        });
    </script>
    
    <style>
        .worker-demo {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .controls {
            margin-bottom: 20px;
            display: flex;
            gap: 10px;
            align-items: center;
        }
        
        input, button {
            padding: 8px 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        button {
            background-color: #007acc;
            color: white;
            border: none;
            cursor: pointer;
        }
        
        .results {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
        }
        
        .result-box {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            background-color: #f9f9f9;
        }
        
        pre {
            background-color: #f5f5f5;
            padding: 10px;
            border-radius: 4px;
            overflow-x: auto;
        }
    </style>
</body>
</html>
```

## Практические примеры оптимизации

### Оптимизация сложной формы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оптимизация сложной формы</title>
</head>
<body>
    <h1>Оптимизированная сложная форма</h1>
    
    <form id="optimized-form">
        <div class="form-section">
            <h2>Личная информация</h2>
            
            <div class="form-row">
                <div class="form-group">
                    <label for="first-name">Имя:</label>
                    <input type="text" 
                           id="first-name" 
                           name="first-name" 
                           required
                           data-validate="name">
                </div>
                
                <div class="form-group">
                    <label for="last-name">Фамилия:</label>
                    <input type="text" 
                           id="last-name" 
                           name="last-name" 
                           required
                           data-validate="name">
                </div>
            </div>
            
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" 
                       id="email" 
                       name="email" 
                       required
                       data-validate="email">
            </div>
        </div>
        
        <div class="form-section">
            <h2>Дополнительная информация</h2>
            
            <div class="form-group">
                <label for="bio">О себе:</label>
                <textarea id="bio" 
                          name="bio" 
                          rows="4"
                          data-validate="bio"></textarea>
            </div>
            
            <div class="form-group">
                <label for="country">Страна:</label>
                <select id="country" name="country" data-validate="country">
                    <option value="">Выберите страну</option>
                    <option value="ru">Россия</option>
                    <option value="ua">Украина</option>
                    <option value="by">Беларусь</option>
                </select>
            </div>
        </div>
        
        <button type="submit">Отправить</button>
    </form>

    <script>
        class OptimizedForm {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.validationRules = new Map();
                this.setupValidationRules();
                this.setupOptimizedValidation();
            }
            
            setupValidationRules() {
                // Определение правил валидации
                this.validationRules.set('name', {
                    pattern: /^[A-Za-zА-Яа-яЁё\s\-]{2,50}$/,
                    message: 'Только буквы, пробелы и дефисы, 2-50 символов'
                });
                
                this.validationRules.set('email', {
                    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
                    message: 'Введите действительный email адрес'
                });
                
                this.validationRules.set('bio', {
                    minLength: 10,
                    maxLength: 500,
                    message: 'Минимум 10, максимум 500 символов'
                });
            }
            
            setupOptimizedValidation() {
                // Используем делегирование событий для оптимизации
                this.form.addEventListener('blur', (e) => {
                    if (e.target.hasAttribute('data-validate')) {
                        this.validateFieldOptimized(e.target);
                    }
                }, true);
                
                // Оптимизированная валидация при отправке
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateFormOptimized();
                });
                
                // Дебаунс для полей, требующих частой валидации
                this.form.addEventListener('input', (e) => {
                    if (e.target.hasAttribute('data-validate')) {
                        this.debouncedValidate(e.target);
                    }
                });
            }
            
            validateFieldOptimized(field) {
                const fieldType = field.getAttribute('data-validate');
                const rule = this.validationRules.get(fieldType);
                
                if (!rule) return true;
                
                const value = field.value.trim();
                
                // Быстрая проверка для пустых полей
                if (field.hasAttribute('required') && !value) {
                    this.showFieldError(field, 'Это поле обязательно');
                    return false;
                }
                
                if (value) {
                    if (rule.pattern && !rule.pattern.test(value)) {
                        this.showFieldError(field, rule.message);
                        return false;
                    }
                    
                    if (rule.minLength && value.length < rule.minLength) {
                        this.showFieldError(field, `Минимум ${rule.minLength} символов`);
                        return false;
                    }
                    
                    if (rule.maxLength && value.length > rule.maxLength) {
                        this.showFieldError(field, `Максимум ${rule.maxLength} символов`);
                        return false;
                    }
                }
                
                this.clearFieldError(field);
                return true;
            }
            
            validateFormOptimized() {
                let isValid = true;
                
                // Используем быструю проверку - прекращаем при первой ошибке
                const fields = this.form.querySelectorAll('[data-validate]');
                
                for (const field of fields) {
                    if (!this.validateFieldOptimized(field)) {
                        isValid = false;
                        // Устанавливаем фокус на первое поле с ошибкой
                        if (isValid === false) { // Только для первого
                            field.focus();
                        }
                    }
                }
                
                if (isValid) {
                    this.submitForm();
                }
            }
            
            showFieldError(field, message) {
                field.classList.add('invalid');
                field.setAttribute('aria-invalid', 'true');
                
                // Используем существующий элемент ошибки или создаем новый
                let errorElement = field.parentNode.querySelector('.error-message');
                if (!errorElement) {
                    errorElement = document.createElement('div');
                    errorElement.className = 'error-message';
                    errorElement.setAttribute('role', 'alert');
                    field.parentNode.appendChild(errorElement);
                }
                
                errorElement.textContent = message;
            }
            
            clearFieldError(field) {
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                
                const errorElement = field.parentNode.querySelector('.error-message');
                if (errorElement) {
                    errorElement.textContent = '';
                }
            }
            
            debouncedValidate = this.debounce((field) => {
                // Валидация при вводе с задержкой
                this.validateFieldOptimized(field);
            }, 300);
            
            debounce(func, wait) {
                let timeout;
                return function executedFunction(...args) {
                    const later = () => {
                        clearTimeout(timeout);
                        func(...args);
                    };
                    clearTimeout(timeout);
                    timeout = setTimeout(later, wait);
                };
            }
            
            async submitForm() {
                const submitBtn = this.form.querySelector('button[type="submit"]');
                const originalText = submitBtn.textContent;
                
                // Показываем состояние загрузки
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    const formData = new FormData(this.form);
                    
                    // Оптимизированная отправка
                    const response = await fetch('/api/submit', {
                        method: 'POST',
                        body: formData
                    });
                    
                    if (response.ok) {
                        alert('Форма успешно отправлена!');
                        this.form.reset();
                    } else {
                        alert('Ошибка при отправке формы');
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    // Восстанавливаем кнопку
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new OptimizedForm('optimized-form');
        });
    </script>
    
    <style>
        .form-section {
            margin-bottom: 2rem;
            padding: 1.5rem;
            border: 1px solid #eee;
            border-radius: 8px;
        }
        
        .form-row {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 1rem;
        }
        
        .form-group {
            margin-bottom: 1rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input, select, textarea {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        input:focus, select:focus, textarea:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
        }
        
        input.invalid {
            border-color: #d32f2f;
            background-color: #ffebee;
        }
        
        .error-message {
            color: #d32f2f;
            font-size: 0.875rem;
            margin-top: 0.25rem;
            display: block;
        }
        
        button {
            background-color: #007acc;
            color: white;
            padding: 0.75rem 1.5rem;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 1rem;
        }
        
        button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
    </style>
</body>
</html>
```

### Оптимизация производительности с кешированием

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Оптимизация с кешированием</title>
</head>
<body>
    <h1>Оптимизация с использованием кеширования</h1>
    
    <div class="cache-demo">
        <h2>Демонстрация кеширования данных</h2>
        
        <div class="controls">
            <button id="load-data-btn">Загрузить данные</button>
            <button id="clear-cache-btn">Очистить кеш</button>
            <button id="show-stats-btn">Показать статистику</button>
        </div>
        
        <div id="data-container"></div>
        <div id="cache-stats"></div>
    </div>

    <script>
        class CacheOptimizer {
            constructor() {
                this.cache = new Map();
                this.stats = {
                    hits: 0,
                    misses: 0,
                    totalRequests: 0
                };
                
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('load-data-btn').addEventListener('click', () => {
                    this.loadDataOptimized();
                });
                
                document.getElementById('clear-cache-btn').addEventListener('click', () => {
                    this.clearCache();
                });
                
                document.getElementById('show-stats-btn').addEventListener('click', () => {
                    this.showCacheStats();
                });
            }
            
            async loadDataOptimized() {
                const cacheKey = 'api-data-' + new Date().toISOString().split('T')[0]; // Кешируем по дням
                
                this.stats.totalRequests++;
                
                if (this.cache.has(cacheKey)) {
                    // Данные в кеше - быстрое возвращение
                    this.stats.hits++;
                    this.displayData(this.cache.get(cacheKey));
                    console.log('Данные получены из кеша');
                } else {
                    // Данные не в кеше - загрузка с сервера
                    this.stats.misses++;
                    
                    try {
                        const data = await this.fetchDataFromServer();
                        this.cache.set(cacheKey, data);
                        this.displayData(data);
                        console.log('Данные загружены с сервера и закешированы');
                    } catch (error) {
                        console.error('Ошибка загрузки данных:', error);
                        this.displayError('Ошибка загрузки данных');
                    }
                }
            }
            
            async fetchDataFromServer() {
                // Симуляция API вызова
                return new Promise((resolve, reject) => {
                    setTimeout(() => {
                        if (Math.random() > 0.1) { // 90% успеха
                            resolve({
                                timestamp: new Date().toISOString(),
                                data: Array.from({ length: 100 }, (_, i) => ({
                                    id: i + 1,
                                    name: `Элемент ${i + 1}`,
                                    value: Math.random() * 100
                                })),
                                cacheKey: 'api-data-' + new Date().toISOString().split('T')[0]
                            });
                        } else {
                            reject(new Error('Сервер недоступен'));
                        }
                    }, 1000); // Имитация задержки
                });
            }
            
            displayData(data) {
                const container = document.getElementById('data-container');
                container.innerHTML = `
                    <div class="data-display">
                        <h3>Данные загружены: ${new Date(data.timestamp).toLocaleString()}</h3>
                        <p>Количество элементов: ${data.data.length}</p>
                        <div class="data-grid">
                            ${data.data.slice(0, 10).map(item => `
                                <div class="data-item">
                                    <strong>${item.name}</strong>: ${item.value.toFixed(2)}
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
            }
            
            displayError(message) {
                const container = document.getElementById('data-container');
                container.innerHTML = `
                    <div class="error-display">
                        <p>Ошибка: ${message}</p>
                    </div>
                `;
            }
            
            clearCache() {
                this.cache.clear();
                console.log('Кеш очищен');
                this.stats = { hits: 0, misses: 0, totalRequests: 0 };
            }
            
            showCacheStats() {
                const statsContainer = document.getElementById('cache-stats');
                const hitRate = this.stats.totalRequests > 0 ? 
                    (this.stats.hits / this.stats.totalRequests * 100).toFixed(2) : 0;
                
                statsContainer.innerHTML = `
                    <div class="stats-display">
                        <h3>Статистика кеширования:</h3>
                        <ul>
                            <li>Всего запросов: ${this.stats.totalRequests}</li>
                            <li>Хиты кеша: ${this.stats.hits}</li>
                            <li>Миссы кеша: ${this.stats.misses}</li>
                            <li>Процент попаданий: ${hitRate}%</li>
                            <li>Размер кеша: ${this.cache.size} элементов</li>
                        </ul>
                    </div>
                `;
            }
            
            // Кеширование с TTL (Time To Live)
            setWithExpiry(key, value, ttl = 300000) { // 5 минут по умолчанию
                const item = {
                    value: value,
                    expiry: new Date().getTime() + ttl,
                };
                this.cache.set(key, item);
            }
            
            getWithExpiry(key) {
                const item = this.cache.get(key);
                if (!item) return null;
                
                const now = new Date().getTime();
                if (now > item.expiry) {
                    this.cache.delete(key);
                    return null;
                }
                
                return item.value;
            }
            
            // Кеширование изображений
            async cacheImage(src) {
                const cacheKey = `img-${src}`;
                
                if (this.cache.has(cacheKey)) {
                    return this.cache.get(cacheKey);
                }
                
                const image = new Image();
                image.src = src;
                
                return new Promise((resolve, reject) => {
                    image.onload = () => {
                        this.cache.set(cacheKey, image);
                        resolve(image);
                    };
                    
                    image.onerror = (error) => {
                        reject(error);
                    };
                });
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new CacheOptimizer();
        });
    </script>
    
    <style>
        .cache-demo {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .controls {
            margin-bottom: 20px;
            display: flex;
            gap: 10px;
        }
        
        button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .data-display, .error-display, .stats-display {
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            margin-top: 20px;
        }
        
        .data-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
            gap: 10px;
            margin-top: 15px;
        }
        
        .data-item {
            padding: 10px;
            background-color: #f8f9fa;
            border-radius: 4px;
            border: 1px solid #eee;
        }
        
        .stats-display ul {
            list-style: none;
            padding: 0;
        }
        
        .stats-display li {
            padding: 5px 0;
            border-bottom: 1px solid #eee;
        }
        
        .stats-display li:last-child {
            border-bottom: none;
        }
    </style>
</body>
</html>
```

## Проверка и тестирование производительности

### Инструменты для анализа производительности:
1. **Chrome DevTools Performance** - для анализа времени загрузки и выполнения
2. **Lighthouse** - для комплексной проверки производительности
3. **WebPageTest** - для анализа загрузки страницы
4. **GTmetrix** - для анализа производительности и оптимизации
5. **PageSpeed Insights** - для рекомендаций по оптимизации
6. **Performance API** - для измерения производительности в коде

### Метрики производительности:
- **FCP (First Contentful Paint)** - время до первого отображения контента
- **LCP (Largest Contentful Paint)** - время до отображения largest content
- **FID (First Input Delay)** - задержка первого ввода
- **CLS (Cumulative Layout Shift)** - cumulative layout shift
- **TTFB (Time to First Byte)** - время до первого байта

### Проверка производительности DOM:
1. Избегать частых reflows и repaints
2. Использовать DocumentFragment для массовых изменений
3. Оптимизировать обработку событий
4. Использовать делегирование событий
5. Избегать глубокой вложенности DOM
6. Оптимизировать CSS селекторы

## Лучшие практики оптимизации

### 1. Структура HTML
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <!-- Критические стили встроены -->
    <style>
        /* Критические стили для верхней части страницы */
        body { margin: 0; font-family: system-ui, sans-serif; }
        .above-fold { min-height: 100vh; display: flex; align-items: center; }
    </style>
    
    <!-- Предзагрузка критических ресурсов -->
    <link rel="preload" href="critical.css" as="style">
    <link rel="preload" href="hero-image.jpg" as="image">
    
    <!-- Заголовки для SEO и производительности -->
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Оптимизированная страница</title>
</head>
<body>
    <!-- Семантическая разметка -->
    <header>
        <h1>Заголовок страницы</h1>
    </header>
    
    <main>
        <article>
            <h2>Основной контент</h2>
            <p>Содержимое статьи...</p>
        </article>
    </main>
    
    <footer>
        <p>Footer информации</p>
    </footer>
    
    <!-- Скрипты в конце body -->
    <script src="app.js" defer></script>
</body>
</html>
```

### 2. Оптимизация изображений
```html
<!-- Использование современных форматов -->
<picture>
    <source srcset="image.avif" type="image/avif">
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="Описание изображения" loading="lazy">
</picture>

<!-- Адаптивные изображения -->
<img src="image-400.jpg"
     srcset="image-400.jpg 400w,
             image-800.jpg 800w,
             image-1200.jpg 1200w"
     sizes="(max-width: 600px) 100vw,
            (max-width: 1000px) 50vw,
            33vw"
     alt="Адаптивное изображение">
```

### 3. Управление ресурсами
```javascript
// Эффективное управление памятью
class ResourceManager {
    constructor() {
        this.resources = new Map();
        this.cleanupQueue = [];
    }
    
    loadResource(key, loader) {
        if (this.resources.has(key)) {
            return this.resources.get(key);
        }
        
        const promise = loader();
        this.resources.set(key, promise);
        
        // Автоматическая очистка через 5 минут
        setTimeout(() => {
            this.resources.delete(key);
        }, 5 * 60 * 1000);
        
        return promise;
    }
    
    cleanupUnusedResources() {
        // Очистка неиспользуемых ресурсов
        this.resources.forEach((resource, key) => {
            if (this.isResourceUnused(key)) {
                this.resources.delete(key);
            }
        });
    }
    
    isResourceUnused(key) {
        // Проверка использования ресурса
        return !document.querySelector(`[data-resource="${key}"]`);
    }
}
```

## Заключение

Оптимизация производительности HTML-страниц - это комплексный процесс, включающий оптимизацию структуры документа, загрузки ресурсов, работы с DOM, использования кеширования и современных веб-технологий. Правильное применение этих техник позволяет создавать быстрые, отзывчивые и эффективные веб-приложения, которые обеспечивают отличный пользовательский опыт.

Ключевые аспекты оптимизации:
1. Минимизация количества DOM-элементов
2. Эффективное использование CSS селекторов
3. Оптимизация загрузки и кеширования ресурсов
4. Использование современных форматов изображений
5. Правильная организация JavaScript кода
6. Использование Web Workers для тяжелых вычислений
7. Применение техник ленивой загрузки
8. Следование современным веб-стандартам

Эти практики помогают создавать веб-сайты, которые быстро загружаются, плавно работают и эффективно используют ресурсы устройства пользователя.

Современные подходы к оптимизации включают:
- Использование Resource Hints для предварительной загрузки
- Внедрение критических стилей в HTML
- Применение техник lazy loading и intersection observer
- Использование современных форматов изображений (AVIF, WebP)
- Оптимизацию JavaScript с помощью модулей и динамических импортов
- Кеширование данных с помощью IndexedDB и Cache API
- Использование Web Workers для тяжелых вычислений
- Применение современных API для оптимизации производительности

Эти техники позволяют создавать высокопроизводительные веб-приложения, которые обеспечивают отличный пользовательский опыт независимо от устройства или подключения к интернету.

## Следующие темы
- [[HTML/Безопасность]]
- [[HTML/Доступность]]
- [[HTML/Совместимость]]

## Теги
#html #performance #optimization #web-development #frontend #speed #loading #rendering #dom #caching #lazy-loading #resource-hints #web-workers #indexeddb #best-practices #accessibility #security
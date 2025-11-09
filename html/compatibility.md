# HTML Совместимость

Совместимость HTML-страниц - это способность веб-страниц корректно работать в различных браузерах, устройствах и версиях веб-технологий. Современная веб-разработка требует учета совместимости с разными браузерами, операционными системами и вспомогательными технологиями.

## Основы совместимости

### Современные и устаревшие браузеры

Современные веб-браузеры обеспечивают поддержку последних стандартов HTML, CSS и JavaScript, в то время как устаревшие браузеры могут не поддерживать некоторые функции. Важно разрабатывать с учетом этого различия.

#### Уровни поддержки браузеров:

1. **Полная поддержка** - все функции работают как ожидается
2. **Частичная поддержка** - некоторые функции работают, другие нет
3. **Нет поддержки** - функция не поддерживается вообще
4. **Экспериментальная поддержка** - функция в бета-версии

### Проверка поддержки возможностей

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Проверка поддержки возможностей</title>
</head>
<body>
    <h1>Проверка поддержки возможностей</h1>
    
    <div id="feature-support">
        <h2>Поддержка возможностей:</h2>
        <ul id="features-list"></ul>
    </div>
    
    <div id="browser-info">
        <h2>Информация о браузере:</h2>
        <div id="browser-details"></div>
    </div>

    <script>
        class BrowserCompatibilityChecker {
            constructor() {
                this.featuresList = document.getElementById('features-list');
                this.browserDetails = document.getElementById('browser-details');
                this.supportedFeatures = [];
                this.unsupportedFeatures = [];
                
                this.checkAllFeatures();
                this.displayBrowserInfo();
            }
            
            checkAllFeatures() {
                const features = [
                    { name: 'HTML5 Canvas', test: () => !!window.CanvasRenderingContext2D },
                    { name: 'CSS Grid', test: () => CSS.supports('display', 'grid') },
                    { name: 'CSS Flexbox', test: () => CSS.supports('display', 'flex') },
                    { name: 'CSS Variables', test: () => CSS.supports('--custom', 'property') },
                    { name: 'Fetch API', test: () => !!window.fetch },
                    { name: 'Promise', test: () => !!window.Promise },
                    { name: 'WebSockets', test: () => !!window.WebSocket },
                    { name: 'IndexedDB', test: () => !!window.indexedDB },
                    { name: 'Geolocation', test: () => !!navigator.geolocation },
                    { name: 'LocalStorage', test: () => !!window.localStorage },
                    { name: 'SessionStorage', test: () => !!window.sessionStorage },
                    { name: 'Intersection Observer', test: () => !!window.IntersectionObserver },
                    { name: 'Mutation Observer', test: () => !!window.MutationObserver },
                    { name: 'Web Workers', test: () => !!window.Worker },
                    { name: 'Web Components', test: () => !!window.customElements },
                    { name: 'Shadow DOM', test: () => !!Element.prototype.attachShadow },
                    { name: 'Template', test: () => !!document.querySelector('template') },
                    { name: 'ES6 Arrow Functions', test: () => {
                        try { eval('var fn = () => {}'); return true; } catch(e) { return false; }
                    }},
                    { name: 'ES6 Classes', test: () => {
                        try { eval('class Test {}'); return true; } catch(e) { return false; }
                    }},
                    { name: 'ES6 Modules', test: () => !!HTMLScriptElement.prototype.noModule }
                ];
                
                features.forEach(feature => {
                    const supported = feature.test();
                    const li = document.createElement('li');
                    li.innerHTML = `
                        <span class="feature-name">${feature.name}</span>:
                        <span class="feature-status ${supported ? 'supported' : 'unsupported'}">
                            ${supported ? '✓ Поддерживается' : '✗ Не поддерживается'}
                        </span>
                    `;
                    
                    this.featuresList.appendChild(li);
                    
                    if (supported) {
                        this.supportedFeatures.push(feature.name);
                    } else {
                        this.unsupportedFeatures.push(feature.name);
                    }
                });
            }
            
            displayBrowserInfo() {
                const browserInfo = {
                    userAgent: navigator.userAgent,
                    platform: navigator.platform,
                    language: navigator.language,
                    cookieEnabled: navigator.cookieEnabled,
                    online: navigator.onLine,
                    hardwareConcurrency: navigator.hardwareConcurrency || 'N/A',
                    maxTouchPoints: navigator.maxTouchPoints || 'N/A',
                    doNotTrack: navigator.doNotTrack || 'N/A',
                    vendor: navigator.vendor || 'N/A'
                };
                
                this.browserDetails.innerHTML = `
                    <div class="browser-info-item">
                        <strong>Платформа:</strong> ${browserInfo.platform}
                    </div>
                    <div class="browser-info-item">
                        <strong>Язык:</strong> ${browserInfo.language}
                    </div>
                    <div class="browser-info-item">
                        <strong>Куки разрешены:</strong> ${browserInfo.cookieEnabled ? 'Да' : 'Нет'}
                    </div>
                    <div class="browser-info-item">
                        <strong>Онлайн:</strong> ${browserInfo.online ? 'Да' : 'Нет'}
                    </div>
                    <div class="browser-info-item">
                        <strong>Количество ядер:</strong> ${browserInfo.hardwareConcurrency}
                    </div>
                    <div class="browser-info-item">
                        <strong>Тач-точек:</strong> ${browserInfo.maxTouchPoints}
                    </div>
                    <div class="browser-info-item">
                        <strong>Do Not Track:</strong> ${browserInfo.doNotTrack}
                    </div>
                `;
            }
            
            getCompatibilityScore() {
                const totalFeatures = this.supportedFeatures.length + this.unsupportedFeatures.length;
                const score = (this.supportedFeatures.length / totalFeatures) * 100;
                return Math.round(score);
            }
            
            generateCompatibilityReport() {
                const score = this.getCompatibilityScore();
                
                return {
                    score: score,
                    supported: this.supportedFeatures,
                    unsupported: this.unsupportedFeatures,
                    browser: {
                        userAgent: navigator.userAgent,
                        platform: navigator.platform,
                        language: navigator.language
                    },
                    timestamp: new Date().toISOString()
                };
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new BrowserCompatibilityChecker();
        });
    </script>
    
    <style>
        .feature-status {
            font-weight: bold;
        }
        
        .feature-status.supported {
            color: #2e7d32;
        }
        
        .feature-status.unsupported {
            color: #d32f2f;
        }
        
        .browser-info-item {
            margin-bottom: 8px;
            padding: 5px 0;
            border-bottom: 1px solid #eee;
        }
        
        #features-list {
            list-style: none;
            padding: 0;
        }
        
        #features-list li {
            padding: 8px 0;
            border-bottom: 1px solid #f0f0f0;
        }
    </style>
</body>
</html>
```

### Проверка поддержки HTML5 элементов

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>HTML5 Элементы Совместимость</title>
</head>
<body>
    <h1>Совместимость HTML5 элементов</h1>
    
    <div class="compatibility-test">
        <h2>Тестирование поддержки HTML5 элементов:</h2>
        
        <!-- Новые HTML5 элементы -->
        <header>
            <h1>Шапка сайта</h1>
        </header>
        
        <nav>
            <ul>
                <li><a href="#">Главная</a></li>
                <li><a href="#">О нас</a></li>
                <li><a href="#">Контакты</a></li>
            </ul>
        </nav>
        
        <main>
            <article>
                <h2>Статья</h2>
                <p>Содержимое статьи</p>
            </article>
            
            <aside>
                <h3>Боковая панель</h3>
                <p>Дополнительная информация</p>
            </aside>
        </main>
        
        <footer>
            <p>Подвал сайта</p>
        </footer>
        
        <section>
            <h2>Раздел</h2>
            <p>Содержимое раздела</p>
        </section>
        
        <figure>
            <img src="image.jpg" alt="Изображение">
            <figcaption>Подпись к изображению</figcaption>
        </figure>
        
        <details>
            <summary>Детали</summary>
            <p>Дополнительная информация</p>
        </details>
        
        <dialog>
            <p>Диалоговое окно</p>
            <button>Закрыть</button>
        </dialog>
    </div>
    
    <div id="html5-results"></div>

    <script>
        class HTML5CompatibilityTester {
            constructor() {
                this.resultsContainer = document.getElementById('html5-results');
                this.testHTML5Elements();
            }
            
            testHTML5Elements() {
                const elements = [
                    'article', 'aside', 'details', 'figcaption', 'figure', 
                    'footer', 'header', 'main', 'nav', 'section', 'summary', 'dialog'
                ];
                
                const results = [];
                
                elements.forEach(elementName => {
                    const element = document.createElement(elementName);
                    const supportsElement = element.constructor !== HTMLUnknownElement;
                    
                    results.push({
                        element: elementName,
                        supported: supportsElement,
                        tagName: element.tagName
                    });
                });
                
                this.displayResults(results);
                this.checkLegacySupport();
            }
            
            displayResults(results) {
                const supportedElements = results.filter(r => r.supported);
                const unsupportedElements = results.filter(r => !r.supported);
                
                this.resultsContainer.innerHTML = `
                    <div class="compatibility-results">
                        <div class="supported-section">
                            <h3>Поддерживаемые элементы (${supportedElements.length}):</h3>
                            <ul>
                                ${supportedElements.map(el => `<li>✓ &lt;${el.element}&gt;</li>`).join('')}
                            </ul>
                        </div>
                        
                        <div class="unsupported-section">
                            <h3>Неподдерживаемые элементы (${unsupportedElements.length}):</h3>
                            <ul>
                                ${unsupportedElements.map(el => `<li>✗ &lt;${el.element}&gt;</li>`).join('')}
                            </ul>
                        </div>
                        
                        <div class="compatibility-score">
                            <h3>Совместимость: ${Math.round((supportedElements.length / results.length) * 100)}%</h3>
                        </div>
                    </div>
                `;
            }
            
            checkLegacySupport() {
                // Проверка поддержки legacy элементов для IE8 и ниже
                if (!document.createElement('header').constructor === HTMLUnknownElement) {
                    // Добавляем стили для IE8 и ниже
                    const style = document.createElement('style');
                    style.textContent = `
                        header, nav, main, article, aside, footer, section, figure {
                            display: block;
                        }
                    `;
                    document.head.appendChild(style);
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new HTML5CompatibilityTester();
        });
    </script>
    
    <style>
        .compatibility-results {
            margin-top: 20px;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        
        .supported-section {
            color: #2e7d32;
        }
        
        .unsupported-section {
            color: #d32f2f;
        }
        
        .compatibility-score {
            margin-top: 20px;
            padding: 15px;
            background-color: #f5f5f5;
            border-radius: 4px;
            text-align: center;
            font-weight: bold;
        }
        
        ul {
            list-style: none;
            padding: 0;
        }
        
        li {
            padding: 5px 0;
        }
    </style>
</body>
</html>
```

### Проверка поддержки CSS возможностей

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSS Совместимость</title>
</head>
<body>
    <h1>Совместимость CSS возможностей</h1>
    
    <div class="css-compatibility-test">
        <h2>Тестирование поддержки CSS возможностей:</h2>
        
        <div class="css-feature-test" id="css-tests-container">
            <!-- Тесты будут добавлены сюда -->
        </div>
    </div>

    <script>
        class CSSCompatibilityTester {
            constructor() {
                this.container = document.getElementById('css-tests-container');
                this.testCSSFeatures();
            }
            
            testCSSFeatures() {
                const features = [
                    { name: 'CSS Grid', property: 'display', value: 'grid' },
                    { name: 'CSS Flexbox', property: 'display', value: 'flex' },
                    { name: 'CSS Variables', property: '--test-var', value: 'red' },
                    { name: 'CSS Transitions', property: 'transition', value: 'all 0.3s ease' },
                    { name: 'CSS Animations', property: 'animation', value: 'test 1s' },
                    { name: 'CSS Transform', property: 'transform', value: 'rotate(90deg)' },
                    { name: 'CSS Filters', property: 'filter', value: 'blur(2px)' },
                    { name: 'CSS Calc()', property: 'width', value: 'calc(50% - 10px)' },
                    { name: 'CSS Clamp()', property: 'font-size', value: 'clamp(1rem, 2.5vw, 2rem)' },
                    { name: 'CSS Media Queries', test: this.testMediaQueries.bind(this) },
                    { name: 'CSS Custom Properties', test: this.testCustomProperties.bind(this) },
                    { name: 'CSS Container Queries', test: this.testContainerQueries.bind(this) }
                ];
                
                features.forEach(feature => {
                    const supported = feature.test ? feature.test() : CSS.supports(feature.property, feature.value);
                    
                    const testDiv = document.createElement('div');
                    testDiv.className = `feature-test ${supported ? 'supported' : 'unsupported'}`;
                    testDiv.innerHTML = `
                        <strong>${feature.name}:</strong> 
                        ${supported ? '✓ Поддерживается' : '✗ Не поддерживается'}
                    `;
                    
                    this.container.appendChild(testDiv);
                });
            }
            
            testMediaQueries() {
                return CSS.supports('(min-width: 1px)');
            }
            
            testCustomProperties() {
                const element = document.createElement('div');
                element.style.setProperty('--test-prop', 'red');
                return element.style.getPropertyValue('--test-prop') === 'red';
            }
            
            testContainerQueries() {
                // Проверка поддержки container queries
                return CSS.supports('container-type', 'inline-size');
            }
            
            // Метод для проверки дополнительных CSS возможностей
            checkAdvancedCSSSupport() {
                const checks = {
                    backdropFilter: CSS.supports('backdrop-filter', 'blur(10px)'),
                    maskImage: CSS.supports('mask-image', 'url(image.png)'),
                    appearance: CSS.supports('-webkit-appearance', 'none'),
                    hyphens: CSS.supports('hyphens', 'auto'),
                    writingMode: CSS.supports('writing-mode', 'vertical-rl')
                };
                
                console.log('Расширенные проверки CSS:', checks);
                return checks;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new CSSCompatibilityTester();
        });
    </script>
    
    <style>
        .css-compatibility-test {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .feature-test {
            padding: 10px;
            margin: 5px 0;
            border-radius: 4px;
        }
        
        .feature-test.supported {
            background-color: #e8f5e8;
            color: #2e7d32;
        }
        
        .feature-test.unsupported {
            background-color: #ffebee;
            color: #c62828;
        }
    </style>
</body>
</html>
```

## Совместимость с различными браузерами

### Поддержка IE и других устаревших браузеров

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Совместимость с устаревшими браузерами</title>
    
    <!-- Условные комментарии для IE (работает только в IE) -->
    <!--[if lt IE 9]>
        <script src="https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
        <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->
    
    <!-- Modernizr для определения возможностей -->
    <script src="https://cdn.jsdelivr.net/npm/modernizr@3.12.0/modernizr.min.js"></script>
</head>
<body>
    <h1>Совместимость с устаревшими браузерами</h1>
    
    <div class="ie-compatibility-demo">
        <h2>Тестирование совместимости с IE</h2>
        
        <!-- Совместимость с IE через полифиллы -->
        <div class="feature-detection">
            <h3>Обнаружение возможностей:</h3>
            <div id="feature-results"></div>
        </div>
        
        <!-- Резервные стили для IE -->
        <div class="ie-fallback" id="ie-fallback" style="display: none;">
            <h3>Совместимость с IE</h3>
            <p>Ваш браузер не поддерживает современные возможности HTML5/CSS3.</p>
            <p>Используются резервные стили и функции.</p>
        </div>
        
        <!-- Современный контент -->
        <div class="modern-content" id="modern-content">
            <h3>Современные возможности</h3>
            <p>Этот контент использует современные HTML5 и CSS3 возможности.</p>
        </div>
    </div>

    <script>
        class IECompatibilityManager {
            constructor() {
                this.checkIECompatibility();
                this.setupFeatureDetection();
            }
            
            checkIECompatibility() {
                // Обнаружение IE через userAgent
                const isIE = /MSIE|Trident/.test(navigator.userAgent);
                
                if (isIE) {
                    this.showIEFallback();
                } else {
                    this.showModernContent();
                }
            }
            
            showIEFallback() {
                document.getElementById('ie-fallback').style.display = 'block';
                document.getElementById('modern-content').style.display = 'none';
                
                console.log('Обнаружен Internet Explorer, активирован резервный режим');
            }
            
            showModernContent() {
                document.getElementById('ie-fallback').style.display = 'none';
                document.getElementById('modern-content').style.display = 'block';
                
                console.log('Обнаружен современный браузер');
            }
            
            setupFeatureDetection() {
                // Современное определение возможностей
                const features = {
                    flexbox: CSS.supports('display', 'flex'),
                    grid: CSS.supports('display', 'grid'),
                    customProperties: CSS.supports('--custom', 'property'),
                    fetch: !!window.fetch,
                    promises: !!window.Promise,
                    arrowFunctions: (() => {
                        try {
                            eval('var fn = () => {}');
                            return true;
                        } catch (e) {
                            return false;
                        }
                    })(),
                    template: !!document.querySelector('template'),
                    shadowDOM: !!Element.prototype.attachShadow,
                    customElements: !!window.customElements,
                    es6Classes: (() => {
                        try {
                            eval('class Test {}');
                            return true;
                        } catch (e) {
                            return false;
                        }
                    })(),
                    modules: !!HTMLScriptElement.prototype.noModule
                };
                
                this.displayFeatureResults(features);
            }
            
            displayFeatureResults(features) {
                const resultsContainer = document.getElementById('feature-results');
                
                resultsContainer.innerHTML = `
                    <ul>
                        ${Object.entries(features).map(([feature, supported]) => `
                            <li class="${supported ? 'supported' : 'unsupported'}">
                                <strong>${feature}:</strong> 
                                ${supported ? '✓ Поддерживается' : '✗ Не поддерживается'}
                            </li>
                        `).join('')}
                    </ul>
                `;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new IECompatibilityManager();
        });
    </script>
    
    <style>
        .ie-compatibility-demo {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .feature-detection {
            margin-bottom: 20px;
            padding: 15px;
            background-color: #f5f5f5;
            border-radius: 8px;
        }
        
        .ie-fallback {
            background-color: #ffebee;
            color: #c62828;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #f5c6cb;
        }
        
        .modern-content {
            background-color: #e8f5e8;
            color: #2e7d32;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #c3e6cb;
        }
        
        ul {
            list-style: none;
            padding: 0;
        }
        
        li {
            padding: 5px 0;
            border-bottom: 1px solid #eee;
        }
        
        li:last-child {
            border-bottom: none;
        }
        
        .supported {
            color: #2e7d32;
        }
        
        .unsupported {
            color: #c62828;
        }
        
        /* Резервные стили для IE */
        .ie-fallback .unsupported {
            display: none;
        }
    </style>
</body>
</html>
```

### Совместимость с мобильными браузерами

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Совместимость с мобильными браузерами</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
    <h1>Совместимость с мобильными браузерами</h1>
    
    <div class="mobile-compatibility-demo">
        <h2>Определение и тестирование мобильных возможностей</h2>
        
        <div class="device-info">
            <h3>Информация об устройстве:</h3>
            <div id="device-info-content"></div>
        </div>
        
        <div class="mobile-features">
            <h3>Мобильные возможности:</h3>
            <div id="mobile-features-content"></div>
        </div>
        
        <div class="touch-controls">
            <h3>Тач-управление:</h3>
            <div id="touch-test-area" class="touch-area">
                <p>Коснитесь области для теста тач-событий</p>
            </div>
        </div>
    </div>

    <script>
        class MobileCompatibilityManager {
            constructor() {
                this.detectMobileCapabilities();
                this.testTouchEvents();
                this.displayDeviceInfo();
            }
            
            detectMobileCapabilities() {
                const capabilities = {
                    touch: 'ontouchstart' in window || navigator.maxTouchPoints > 0,
                    orientation: 'onorientationchange' in window,
                    deviceMemory: navigator.deviceMemory || 'N/A',
                    hardwareConcurrency: navigator.hardwareConcurrency || 'N/A',
                    connection: navigator.connection || null,
                    userAgent: navigator.userAgent.toLowerCase(),
                    platform: navigator.platform.toLowerCase()
                };
                
                // Проверка, является ли устройство мобильным
                const isMobile = /android|webos|iphone|ipad|ipod|blackberry|iemobile|opera mini/i.test(navigator.userAgent);
                
                capabilities.isMobile = isMobile;
                
                this.displayMobileFeatures(capabilities);
                this.setupMobileSpecificFeatures(capabilities);
                
                return capabilities;
            }
            
            testTouchEvents() {
                const touchArea = document.getElementById('touch-test-area');
                let touchStarted = false;
                
                touchArea.addEventListener('touchstart', (e) => {
                    touchStarted = true;
                    touchArea.innerHTML = '<p>✓ Touch start detected</p>';
                });
                
                touchArea.addEventListener('touchmove', (e) => {
                    if (touchStarted) {
                        touchArea.innerHTML = '<p>✓ Touch move detected</p>';
                    }
                });
                
                touchArea.addEventListener('touchend', (e) => {
                    if (touchStarted) {
                        touchArea.innerHTML = '<p>✓ Touch end detected</p>';
                        setTimeout(() => {
                            touchArea.innerHTML = '<p>Коснитесь области для теста тач-событий</p>';
                        }, 2000);
                    }
                });
                
                // Для совместимости с мышью
                touchArea.addEventListener('mousedown', (e) => {
                    touchArea.innerHTML = '<p>✓ Mouse event detected (fallback)</p>';
                    setTimeout(() => {
                        touchArea.innerHTML = '<p>Коснитесь области для теста тач-событий</p>';
                    }, 2000);
                });
            }
            
            displayDeviceInfo() {
                const deviceInfo = {
                    userAgent: navigator.userAgent,
                    platform: navigator.platform,
                    language: navigator.language,
                    cookieEnabled: navigator.cookieEnabled,
                    online: navigator.onLine,
                    screenResolution: `${screen.width}x${screen.height}`,
                    viewportSize: `${window.innerWidth}x${window.innerHeight}`,
                    pixelRatio: window.devicePixelRatio || 1
                };
                
                const infoContainer = document.getElementById('device-info-content');
                infoContainer.innerHTML = `
                    <ul>
                        <li><strong>User Agent:</strong> ${deviceInfo.userAgent}</li>
                        <li><strong>Платформа:</strong> ${deviceInfo.platform}</li>
                        <li><strong>Язык:</strong> ${deviceInfo.language}</li>
                        <li><strong>Куки разрешены:</strong> ${deviceInfo.cookieEnabled ? 'Да' : 'Нет'}</li>
                        <li><strong>Онлайн:</strong> ${deviceInfo.online ? 'Да' : 'Нет'}</li>
                        <li><strong>Разрешение экрана:</strong> ${deviceInfo.screenResolution}</li>
                        <li><strong>Размер viewport:</strong> ${deviceInfo.viewportSize}</li>
                        <li><strong>Pixel Ratio:</strong> ${deviceInfo.pixelRatio}</li>
                    </ul>
                `;
            }
            
            displayMobileFeatures(capabilities) {
                const featuresContainer = document.getElementById('mobile-features-content');
                featuresContainer.innerHTML = `
                    <ul>
                        <li class="${capabilities.touch ? 'supported' : 'unsupported'}">
                            <strong>Touch события:</strong> 
                            ${capabilities.touch ? '✓ Поддерживается' : '✗ Не поддерживается'}
                        </li>
                        <li class="${capabilities.orientation ? 'supported' : 'unsupported'}">
                            <strong>Ориентация:</strong> 
                            ${capabilities.orientation ? '✓ Поддерживается' : '✗ Не поддерживается'}
                        </li>
                        <li>
                            <strong>Мобильное устройство:</strong> 
                            ${capabilities.isMobile ? '✓ Да' : '✗ Нет'}
                        </li>
                        <li>
                            <strong>Память устройства:</strong> 
                            ${capabilities.deviceMemory}
                        </li>
                        <li>
                            <strong>Количество ядер:</strong> 
                            ${capabilities.hardwareConcurrency}
                        </li>
                    </ul>
                `;
            }
            
            setupMobileSpecificFeatures(capabilities) {
                if (capabilities.connection) {
                    // Проверка сетевых возможностей
                    this.displayNetworkInfo(capabilities.connection);
                }
                
                // Установка мобильных стилей
                if (capabilities.isMobile) {
                    this.applyMobileStyles();
                }
                
                // Управление событиями ориентации
                if (capabilities.orientation) {
                    this.setupOrientationHandling();
                }
            }
            
            displayNetworkInfo(connection) {
                const networkInfo = document.createElement('div');
                networkInfo.innerHTML = `
                    <h4>Информация о соединении:</h4>
                    <ul>
                        <li><strong>Тип соединения:</strong> ${connection.effectiveType || 'N/A'}</li>
                        <li><strong>Скорость:</strong> ${connection.downlink || 'N/A'} Mbps</li>
                        <li><strong>RTT:</strong> ${connection.rtt || 'N/A'} ms</li>
                        <li><strong>Save Data:</strong> ${connection.saveData ? 'Да' : 'Нет'}</li>
                    </ul>
                `;
                
                document.querySelector('.mobile-features').appendChild(networkInfo);
            }
            
            applyMobileStyles() {
                const mobileStyles = document.createElement('style');
                mobileStyles.textContent = `
                    /* Мобильные специфичные стили */
                    body {
                        font-size: 16px; /* Больше для удобства чтения */
                    }
                    
                    .touch-area {
                        min-height: 150px;
                        background-color: #e3f2fd;
                        border: 2px dashed #1976d2;
                        border-radius: 8px;
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        text-align: center;
                    }
                    
                    /* Увеличенные сенсорные зоны */
                    button, input, select, textarea {
                        min-height: 44px;
                        min-width: 44px;
                    }
                `;
                
                document.head.appendChild(mobileStyles);
            }
            
            setupOrientationHandling() {
                window.addEventListener('orientationchange', () => {
                    setTimeout(() => {
                        // Обновление информации об устройстве при изменении ориентации
                        this.displayDeviceInfo();
                        
                        // Манипуляции с макетом при изменении ориентации
                        this.handleOrientationChange();
                    }, 300); // Небольшая задержка для корректного определения размеров
                });
            }
            
            handleOrientationChange() {
                const orientation = window.orientation;
                const body = document.body;
                
                switch(orientation) {
                    case 0:
                        body.setAttribute('data-orientation', 'portrait');
                        console.log('Портретная ориентация');
                        break;
                    case 90:
                    case -90:
                        body.setAttribute('data-orientation', 'landscape');
                        console.log('Альбомная ориентация');
                        break;
                    case 180:
                        body.setAttribute('data-orientation', 'reverse-portrait');
                        console.log('Обратная портретная ориентация');
                        break;
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new MobileCompatibilityManager();
        });
    </script>
    
    <style>
        .mobile-compatibility-demo {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .device-info, .mobile-features {
            margin-bottom: 20px;
            padding: 15px;
            background-color: #f5f5f5;
            border-radius: 8px;
        }
        
        .touch-area {
            min-height: 150px;
            background-color: #f0f0f0;
            border: 2px dashed #ccc;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
        }
        
        ul {
            list-style: none;
            padding: 0;
        }
        
        li {
            padding: 8px 0;
            border-bottom: 1px solid #eee;
        }
        
        li:last-child {
            border-bottom: none;
        }
        
        .supported {
            color: #2e7d32;
        }
        
        .unsupported {
            color: #c62828;
        }
    </style>
</body>
</html>
```

## Резервные варианты и прогрессивное улучшение

### Прогрессивное улучшение

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Прогрессивное улучшение</title>
</head>
<body>
    <h1>Прогрессивное улучшение</h1>
    
    <div class="progressive-enhancement-demo">
        <h2>Пример прогрессивного улучшения</h2>
        
        <!-- Базовая функциональность -->
        <form action="/submit" method="post">
            <input type="text" name="query" placeholder="Поиск...">
            <button type="submit">Найти</button>
        </form>
        
        <!-- Улучшенная функциональность для современных браузеров -->
        <div id="enhanced-search" style="display: none;">
            <input type="search" 
                   id="enhanced-query" 
                   placeholder="Прогрессивный поиск..."
                   autocomplete="off">
            <div id="suggestions-container"></div>
        </div>
        
        <!-- Современный компонент для поддерживаемых браузеров -->
        <div id="modern-component">
            <h3>Современный интерактивный компонент</h3>
            <div id="component-content">
                <p>Этот контент доступен только в современных браузерах</p>
            </div>
        </div>
        
        <!-- Резервный компонент для старых браузеров -->
        <div id="legacy-component">
            <h3>Базовый компонент</h3>
            <div id="legacy-content">
                <p>Этот контент доступен во всех браузерах</p>
            </div>
        </div>
    </div>

    <script>
        class ProgressiveEnhancement {
            constructor() {
                this.baseForm = document.querySelector('form[action="/submit"]');
                this.enhancedContainer = document.getElementById('enhanced-search');
                this.modernComponent = document.getElementById('modern-component');
                this.legacyComponent = document.getElementById('legacy-component');
                
                this.checkModernCapabilities();
                this.setupEnhancedFeatures();
            }
            
            checkModernCapabilities() {
                const capabilities = {
                    fetch: !!window.fetch,
                    promise: !!window.Promise,
                    customElements: !!window.customElements,
                    shadowDOM: !!Element.prototype.attachShadow,
                    intersectionObserver: !!window.IntersectionObserver,
                    webAnimations: !!Element.prototype.animate,
                    es6: this.testES6Features(),
                    modules: !!HTMLScriptElement.prototype.noModule
                };
                
                // Если все основные возможности поддерживаются, включаем улучшенный интерфейс
                if (capabilities.fetch && capabilities.promise && capabilities.es6) {
                    this.enableEnhancedFeatures();
                } else {
                    this.showLegacyComponent();
                }
                
                return capabilities;
            }
            
            testES6Features() {
                try {
                    // Проверка основных ES6 возможностей
                    eval('const a = 1; let b = 2; const c = (x) => x * 2; class Test {}');
                    return true;
                } catch (e) {
                    return false;
                }
            }
            
            enableEnhancedFeatures() {
                this.baseForm.style.display = 'none';
                this.enhancedContainer.style.display = 'block';
                
                // Инициализация современных функций
                this.setupAdvancedSearch();
                this.setupModernComponent();
            }
            
            showLegacyComponent() {
                this.modernComponent.style.display = 'none';
                this.legacyComponent.style.display = 'block';
                
                console.log('Активирован резервный режим для старых браузеров');
            }
            
            setupAdvancedSearch() {
                const searchInput = document.getElementById('enhanced-query');
                
                let typingTimer;
                searchInput.addEventListener('input', () => {
                    clearTimeout(typingTimer);
                    typingTimer = setTimeout(() => {
                        this.fetchSuggestions(searchInput.value);
                    }, 300);
                });
                
                // Добавление поддержки для клавиатурной навигации
                searchInput.addEventListener('keydown', (e) => {
                    if (e.key === 'ArrowDown') {
                        this.navigateSuggestions(1);
                    } else if (e.key === 'ArrowUp') {
                        this.navigateSuggestions(-1);
                    } else if (e.key === 'Enter') {
                        this.selectSuggestion();
                    }
                });
            }
            
            async fetchSuggestions(query) {
                if (query.length < 2) {
                    document.getElementById('suggestions-container').innerHTML = '';
                    return;
                }
                
                try {
                    // Имитация API вызова
                    const suggestions = await this.getMockSuggestions(query);
                    this.displaySuggestions(suggestions);
                } catch (error) {
                    console.error('Ошибка получения подсказок:', error);
                }
            }
            
            getMockSuggestions(query) {
                return new Promise(resolve => {
                    setTimeout(() => {
                        const mockSuggestions = [
                            'HTML5',
                            'CSS3',
                            'JavaScript',
                            'React',
                            'Vue.js',
                            'Angular',
                            'Webpack',
                            'Babel',
                            'Node.js'
                        ].filter(item => 
                            item.toLowerCase().includes(query.toLowerCase())
                        );
                        
                        resolve(mockSuggestions.slice(0, 5));
                    }, 500);
                });
            }
            
            displaySuggestions(suggestions) {
                const container = document.getElementById('suggestions-container');
                
                if (suggestions.length === 0) {
                    container.innerHTML = '';
                    return;
                }
                
                container.innerHTML = `
                    <ul class="suggestions-list">
                        ${suggestions.map((suggestion, index) => `
                            <li class="suggestion-item" 
                                data-index="${index}"
                                onclick="enhancedApp.selectSuggestion(this)">
                                ${suggestion}
                            </li>
                        `).join('')}
                    </ul>
                `;
            }
            
            navigateSuggestions(direction) {
                const suggestions = document.querySelectorAll('.suggestion-item');
                if (suggestions.length === 0) return;
                
                let currentIndex = -1;
                const currentSelected = document.querySelector('.suggestion-item.selected');
                if (currentSelected) {
                    currentIndex = parseInt(currentSelected.dataset.index);
                }
                
                let newIndex = currentIndex + direction;
                if (newIndex < 0) newIndex = suggestions.length - 1;
                if (newIndex >= suggestions.length) newIndex = 0;
                
                suggestions.forEach(item => item.classList.remove('selected'));
                suggestions[newIndex].classList.add('selected');
                suggestions[newIndex].scrollIntoView({ block: 'nearest' });
            }
            
            selectSuggestion(element) {
                const suggestion = element ? element.textContent : 
                                 document.querySelector('.suggestion-item.selected')?.textContent;
                
                if (suggestion) {
                    document.getElementById('enhanced-query').value = suggestion;
                    document.getElementById('suggestions-container').innerHTML = '';
                }
            }
            
            setupModernComponent() {
                // Инициализация современного компонента
                const modernContent = document.getElementById('component-content');
                
                if ('requestAnimationFrame' in window) {
                    // Используем современные API для анимации
                    this.setupSmoothAnimations(modernContent);
                }
                
                if ('IntersectionObserver' in window) {
                    // Используем современные API для наблюдения за элементами
                    this.setupIntersectionObserver(modernContent);
                }
                
                if ('WebAnimations' in window) {
                    // Используем современные API для анимации
                    this.setupWebAnimations(modernContent);
                }
            }
            
            setupSmoothAnimations(element) {
                // Пример использования requestAnimationFrame
                let start = null;
                const duration = 2000;
                
                const animate = (timestamp) => {
                    if (!start) start = timestamp;
                    const progress = timestamp - start;
                    
                    // Применение анимации
                    element.style.transform = `translateX(${Math.min(progress / duration * 100, 100)}%)`;
                    
                    if (progress < duration) {
                        requestAnimationFrame(animate);
                    }
                };
                
                requestAnimationFrame(animate);
            }
            
            setupIntersectionObserver(element) {
                // Пример использования Intersection Observer
                const observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            element.classList.add('visible');
                        } else {
                            element.classList.remove('visible');
                        }
                    });
                });
                
                observer.observe(element);
            }
            
            setupWebAnimations(element) {
                // Пример использования Web Animations API
                if (element.animate) {
                    element.animate([
                        { opacity: 0, transform: 'translateY(20px)' },
                        { opacity: 1, transform: 'translateY(0)' }
                    ], {
                        duration: 1000,
                        easing: 'ease-out'
                    });
                }
            }
        }
        
        // Инициализация
        const enhancedApp = new ProgressiveEnhancement();
        
        // Глобальная переменная для доступа из HTML
        window.enhancedApp = enhancedApp;
    </script>
    
    <style>
        .progressive-enhancement-demo {
            max-width: 600px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .enhanced-search input {
            width: 100%;
            padding: 10px;
            border: 2px solid #007acc;
            border-radius: 4px;
            font-size: 16px;
        }
        
        .suggestions-list {
            list-style: none;
            padding: 0;
            margin: 0;
            border: 1px solid #ddd;
            border-top: none;
            background: white;
            max-height: 200px;
            overflow-y: auto;
        }
        
        .suggestion-item {
            padding: 10px;
            cursor: pointer;
            border-bottom: 1px solid #eee;
        }
        
        .suggestion-item:hover, .suggestion-item.selected {
            background-color: #e3f2fd;
        }
        
        #modern-component {
            background-color: #e8f5e8;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #c8e6c9;
        }
        
        #legacy-component {
            background-color: #ffebee;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #f5c6cb;
        }
        
        .visible {
            animation: fadeIn 0.5s ease-out;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
    </style>
</body>
</html>
```

### Резервные варианты (fallbacks)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Резервные варианты</title>
</head>
<body>
    <h1>Резервные варианты</h1>
    
    <div class="fallback-demo">
        <h2>Пример резервных вариантов</h2>
        
        <div class="modern-component">
            <h3>Современный компонент</h3>
            
            <!-- Резервный вариант для старых браузеров -->
            <div class="fallback" id="fallback-content">
                <p>Ваш браузер не поддерживает современные возможности.</p>
                <p>Пожалуйста, обновите браузер или включите JavaScript.</p>
            </div>
            
            <!-- Современный контент будет показан поверх резервного -->
            <div class="modern-content" id="modern-content" style="display: none;">
                <p>Современный интерактивный контент загружен.</p>
                <button id="modern-button">Современная кнопка</button>
            </div>
        </div>
        
        <!-- Аудио с резервными вариантами -->
        <audio controls>
            <source src="audio.webm" type="audio/webm">
            <source src="audio.mp3" type="audio/mpeg">
            Ваш браузер не поддерживает аудио элемент.
            <a href="audio.mp3">Скачать аудио</a>
        </audio>
        
        <!-- Видео с резервными вариантами -->
        <video controls width="640" height="360">
            <source src="video.webm" type="video/webm">
            <source src="video.mp4" type="video/mp4">
            Ваш браузер не поддерживает видео элемент.
            <a href="video.mp4">Скачать видео</a>
        </video>
        
        <!-- Изображение с современными форматами и резервом -->
        <picture>
            <source srcset="image.avif" type="image/avif">
            <source srcset="image.webp" type="image/webp">
            <img src="image.jpg" alt="Описание изображения">
        </picture>
    </div>

    <script>
        class FallbackHandler {
            constructor() {
                this.checkModernCapabilities();
                this.setupFallbackHandling();
            }
            
            checkModernCapabilities() {
                const capabilities = {
                    es6: this.testES6Features(),
                    fetch: !!window.fetch,
                    promise: !!window.Promise,
                    arrowFunctions: this.testArrowFunctions(),
                    modules: this.testModules(),
                    shadowDOM: !!Element.prototype.attachShadow,
                    customElements: !!window.customElements,
                    template: !!document.querySelector('template'),
                    webAnimations: !!Element.prototype.animate,
                    intersectionObserver: !!window.IntersectionObserver,
                    webGL: this.testWebGL(),
                    canvas: !!document.createElement('canvas').getContext('2d')
                };
                
                console.log('Поддерживаемые возможности:', capabilities);
                
                // Проверяем, поддерживает ли браузер минимальные возможности
                if (capabilities.es6 && capabilities.fetch && capabilities.promise) {
                    this.enableModernFeatures(capabilities);
                } else {
                    this.showFallbackNotice();
                }
                
                return capabilities;
            }
            
            testES6Features() {
                try {
                    // Проверяем основные ES6 возможности
                    eval('const a = 1; let b = 2; const c = (x) => x * 2; class Test {}');
                    return true;
                } catch (e) {
                    return false;
                }
            }
            
            testArrowFunctions() {
                try {
                    eval('var fn = () => {};');
                    return true;
                } catch (e) {
                    return false;
                }
            }
            
            testModules() {
                return 'noModule' in HTMLScriptElement.prototype;
            }
            
            testWebGL() {
                try {
                    const canvas = document.createElement('canvas');
                    return !!(window.WebGLRenderingContext && 
                             (canvas.getContext('webgl') || canvas.getContext('experimental-webgl')));
                } catch (e) {
                    return false;
                }
            }
            
            enableModernFeatures(capabilities) {
                // Скрываем резервный контент и показываем современный
                document.getElementById('fallback-content').style.display = 'none';
                document.getElementById('modern-content').style.display = 'block';
                
                // Инициализируем современные функции
                this.initializeModernFeatures();
            }
            
            showFallbackNotice() {
                // Показываем уведомление о необходимости обновления
                const notice = document.createElement('div');
                notice.className = 'browser-notice';
                notice.innerHTML = `
                    <p>⚠️ Ваш браузер устарел. Некоторые функции могут работать некорректно.</p>
                    <p>Рекомендуется <a href="https://browsehappy.com/" target="_blank">обновить браузер</a>.</p>
                `;
                
                document.body.prepend(notice);
            }
            
            initializeModernFeatures() {
                // Инициализация современных функций
                document.getElementById('modern-button').addEventListener('click', () => {
                    alert('Современная кнопка нажата!');
                });
                
                // Проверка и подключение полифиллов
                this.loadPolyfillsIfNeeded();
            }
            
            loadPolyfillsIfNeeded() {
                const polyfills = [];
                
                if (!window.fetch) {
                    polyfills.push('https://cdn.jsdelivr.net/npm/whatwg-fetch@3.6.2/dist/fetch.umd.js');
                }
                
                if (!window.Promise) {
                    polyfills.push('https://cdn.jsdelivr.net/npm/es6-promise@4.2.8/dist/es6-promise.auto.min.js');
                }
                
                if (!window.Intl) {
                    polyfills.push('https://cdn.jsdelivr.net/npm/intl@1.2.5/Intl.min.js');
                }
                
                if (!Array.prototype.includes) {
                    polyfills.push('https://cdn.jsdelivr.net/npm/core-js-bundle@3.26.1/minified.js');
                }
                
                // Загрузка полифиллов
                polyfills.forEach(polyfill => {
                    this.loadScript(polyfill).then(() => {
                        console.log('Полифилл загружен:', polyfill);
                    }).catch(error => {
                        console.error('Ошибка загрузки полифилла:', polyfill, error);
                    });
                });
            }
            
            loadScript(src) {
                return new Promise((resolve, reject) => {
                    const script = document.createElement('script');
                    script.src = src;
                    script.onload = resolve;
                    script.onerror = reject;
                    document.head.appendChild(script);
                });
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new FallbackHandler();
        });
    </script>
    
    <style>
        .fallback-demo {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .modern-component {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
            background-color: #f5f5f5;
        }
        
        .fallback {
            background-color: #ffebee;
            color: #c62828;
            padding: 15px;
            border-radius: 4px;
            margin-bottom: 10px;
        }
        
        .modern-content {
            background-color: #e8f5e8;
            color: #2e7d32;
            padding: 15px;
            border-radius: 4px;
        }
        
        .browser-notice {
            background-color: #fff3cd;
            color: #856404;
            padding: 15px;
            border: 1px solid #ffeaa7;
            border-radius: 4px;
            margin: 20px 0;
        }
        
        .browser-notice a {
            color: #856404;
            font-weight: bold;
        }
        
        audio, video {
            width: 100%;
            margin: 10px 0;
        }
        
        picture, img {
            max-width: 100%;
            height: auto;
        }
    </style>
</body>
</html>
```

## Современные подходы к совместимости

### Использование Feature Detection

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Feature Detection</title>
</head>
<body>
    <h1>Feature Detection для совместимости</h1>
    
    <div class="feature-detection-demo">
        <h2>Определение возможностей браузера</h2>
        
        <div class="capability-grid" id="capabilities-grid"></div>
        
        <div class="feature-usage" id="feature-usage">
            <h3>Использование возможностей:</h3>
            <div id="usage-examples"></div>
        </div>
    </div>

    <script>
        class FeatureDetectionManager {
            constructor() {
                this.capabilitiesGrid = document.getElementById('capabilities-grid');
                this.usageExamples = document.getElementById('usage-examples');
                
                this.detectCapabilities();
                this.showUsageExamples();
            }
            
            detectCapabilities() {
                const capabilities = {
                    // HTML5 API
                    canvas: () => !!document.createElement('canvas').getContext,
                    webgl: () => this.testWebGL(),
                    video: () => !!document.createElement('video').canPlayType,
                    audio: () => !!document.createElement('audio').canPlayType,
                    localStorage: () => !!window.localStorage,
                    sessionStorage: () => !!window.sessionStorage,
                    indexedDB: () => !!window.indexedDB,
                    geolocation: () => !!navigator.geolocation,
                    fileAPI: () => !!window.File && !!window.FileReader,
                    
                    // CSS
                    flexbox: () => CSS.supports('display', 'flex'),
                    grid: () => CSS.supports('display', 'grid'),
                    customProperties: () => CSS.supports('--custom', 'property'),
                    transforms: () => CSS.supports('transform', 'rotate(1deg)'),
                    transitions: () => CSS.supports('transition', 'all 1s'),
                    
                    // JavaScript
                    fetch: () => !!window.fetch,
                    promise: () => !!window.Promise,
                    arrowFunctions: () => {
                        try { eval('()=>{}'); return true; } catch(e) { return false; }
                    },
                    classes: () => {
                        try { eval('class T{}'); return true; } catch(e) { return false; }
                    },
                    modules: () => !!HTMLScriptElement.prototype.noModule,
                    template: () => !!document.querySelector('template'),
                    shadowDOM: () => !!Element.prototype.attachShadow,
                    customElements: () => !!window.customElements,
                    intersectionObserver: () => !!window.IntersectionObserver,
                    mutationObserver: () => !!window.MutationObserver,
                    webAnimations: () => !!Element.prototype.animate,
                    
                    // Device API
                    touch: () => 'ontouchstart' in window || navigator.maxTouchPoints > 0,
                    orientation: () => 'onorientationchange' in window,
                    vibration: () => !!navigator.vibrate,
                    battery: () => !!navigator.getBattery,
                    clipboard: () => !!navigator.clipboard,
                    
                    // Network API
                    serviceWorker: () => 'serviceWorker' in navigator,
                    webSockets: () => !!window.WebSocket,
                    webRTC: () => !!window.RTCPeerConnection,
                    beacon: () => !!navigator.sendBeacon
                };
                
                const results = {};
                for (const [feature, detector] of Object.entries(capabilities)) {
                    results[feature] = detector();
                }
                
                this.displayCapabilities(results);
                this.storeCapabilities(results);
                
                return results;
            }
            
            testWebGL() {
                try {
                    const canvas = document.createElement('canvas');
                    return !!(window.WebGLRenderingContext && 
                             (canvas.getContext('webgl') || canvas.getContext('experimental-webgl')));
                } catch (e) {
                    return false;
                }
            }
            
            displayCapabilities(capabilities) {
                const grid = this.capabilitiesGrid;
                grid.innerHTML = '';
                
                const supportedCount = Object.values(capabilities).filter(Boolean).length;
                const totalCount = Object.keys(capabilities).length;
                const score = Math.round((supportedCount / totalCount) * 100);
                
                grid.innerHTML = `
                    <div class="compatibility-score">
                        <h3>Совместимость: ${score}% (${supportedCount}/${totalCount})</h3>
                    </div>
                    
                    <div class="capabilities-list">
                        ${Object.entries(capabilities).map(([feature, supported]) => `
                            <div class="capability-item ${supported ? 'supported' : 'unsupported'}">
                                <strong>${feature}:</strong> 
                                ${supported ? '✓ Поддерживается' : '✗ Не поддерживается'}
                            </div>
                        `).join('')}
                    </div>
                `;
            }
            
            storeCapabilities(capabilities) {
                // Сохраняем результаты определения возможностей для использования в других частях приложения
                window.browserCapabilities = capabilities;
                
                // Также сохраняем в localStorage для последующих посещений
                localStorage.setItem('browser-capabilities', JSON.stringify(capabilities));
            }
            
            showUsageExamples() {
                const examples = [];
                
                if (window.browserCapabilities?.fetch) {
                    examples.push(`
                        <div class="usage-example">
                            <h4>Fetch API:</h4>
                            <pre><code>fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data));</code></pre>
                        </div>
                    `);
                }
                
                if (window.browserCapabilities?.intersectionObserver) {
                    examples.push(`
                        <div class="usage-example">
                            <h4>Intersection Observer:</h4>
                            <pre><code>const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Элемент появился в области видимости');
    }
  });
});
observer.observe(targetElement);</code></pre>
                        </div>
                    `);
                }
                
                if (window.browserCapabilities?.localStorage) {
                    examples.push(`
                        <div class="usage-example">
                            <h4>Local Storage:</h4>
                            <pre><code>localStorage.setItem('key', 'value');
const value = localStorage.getItem('key');</code></pre>
                        </div>
                    `);
                }
                
                if (window.browserCapabilities?.serviceWorker) {
                    examples.push(`
                        <div class="usage-example">
                            <h4>Service Worker:</h4>
                            <pre><code>navigator.serviceWorker.register('/sw.js')
  .then(registration => console.log('SW зарегистрирован'))
  .catch(error => console.error('Ошибка регистрации:', error));</code></pre>
                        </div>
                    `);
                }
                
                if (examples.length === 0) {
                    examples.push(`
                        <div class="usage-example">
                            <p>Ваш браузер не поддерживает современные возможности. 
                               Используйте резервные методы и полифиллы.</p>
                        </div>
                    `);
                }
                
                this.usageExamples.innerHTML = examples.join('');
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new FeatureDetectionManager();
        });
    </script>
    
    <style>
        .feature-detection-demo {
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .compatibility-score {
            text-align: center;
            padding: 20px;
            background-color: #f5f5f5;
            border-radius: 8px;
            margin-bottom: 20px;
        }
        
        .capabilities-list {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
            gap: 10px;
        }
        
        .capability-item {
            padding: 10px;
            border-radius: 4px;
            margin: 5px 0;
        }
        
        .capability-item.supported {
            background-color: #e8f5e8;
            color: #2e7d32;
        }
        
        .capability-item.unsupported {
            background-color: #ffebee;
            color: #c62828;
        }
        
        .usage-examples {
            margin-top: 30px;
        }
        
        .usage-example {
            margin-bottom: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            background-color: #f9f9f9;
        }
        
        pre {
            background-color: #f5f5f5;
            padding: 10px;
            border-radius: 4px;
            overflow-x: auto;
            font-size: 0.9em;
        }
        
        code {
            font-family: 'Courier New', monospace;
        }
    </style>
</body>
</html>
```

### Совместимость с Web Components

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Совместимость с Web Components</title>
</head>
<body>
    <h1>Совместимость с Web Components</h1>
    
    <div class="web-components-compatibility">
        <h2>Тестирование поддержки Web Components</h2>
        
        <!-- Компонент будет добавлен динамически -->
        <div id="web-component-container"></div>
    </div>

    <script>
        class WebComponentsCompatibility {
            constructor() {
                this.testSupport();
                this.setupPolyfillsIfNeeded();
            }
            
            testSupport() {
                const support = {
                    customElements: 'customElements' in window,
                    shadowDOM: 'attachShadow' in Element.prototype,
                    templates: 'content' in document.createElement('template'),
                    htmlImports: 'import' in document.createElement('link') // Устаревший API
                };
                
                console.log('Поддержка Web Components:', support);
                
                if (support.customElements && support.shadowDOM && support.templates) {
                    console.log('Полная поддержка Web Components');
                    this.enableWebComponents();
                } else {
                    console.log('Частичная поддержка Web Components, подключение полифиллов');
                    this.enableFallback();
                }
                
                return support;
            }
            
            async setupPolyfillsIfNeeded() {
                const support = this.testSupport();
                
                if (!support.customElements) {
                    await this.loadScript('https://cdn.jsdelivr.net/npm/@webcomponents/custom-elements@1.5.0/custom-elements.min.js');
                }
                
                if (!support.shadowDOM) {
                    await this.loadScript('https://cdn.jsdelivr.net/npm/@webcomponents/shadydom@1.8.0/shadydom.min.js');
                }
                
                if (!support.templates) {
                    await this.loadScript('https://cdn.jsdelivr.net/npm/webcomponentsjs@2.8.0/webcomponents-bundle.js');
                }
            }
            
            enableWebComponents() {
                // Регистрация компонентов для поддерживаемых браузеров
                this.registerModernComponents();
                
                // Создание компонентов
                this.createModernComponents();
            }
            
            enableFallback() {
                // Резервная реализация для неподдерживаемых браузеров
                this.registerFallbackComponents();
                
                // Создание резервных компонентов
                this.createFallbackComponents();
            }
            
            registerModernComponents() {
                // Кастомный элемент с полной поддержкой
                class ModernUserCard extends HTMLElement {
                    constructor() {
                        super();
                        
                        this.attachShadow({ mode: 'open' });
                        this.render();
                    }
                    
                    static get observedAttributes() {
                        return ['username', 'email', 'avatar'];
                    }
                    
                    attributeChangedCallback(name, oldValue, newValue) {
                        if (oldValue !== newValue) {
                            this[name] = newValue;
                            this.render();
                        }
                    }
                    
                    render() {
                        this.shadowRoot.innerHTML = `
                            <style>
                                :host {
                                    display: inline-block;
                                    font-family: Arial, sans-serif;
                                }
                                
                                .card {
                                    border: 1px solid #ddd;
                                    border-radius: 8px;
                                    padding: 20px;
                                    background-color: white;
                                    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                                }
                                
                                .avatar {
                                    width: 60px;
                                    height: 60px;
                                    border-radius: 50%;
                                    object-fit: cover;
                                    margin-bottom: 15px;
                                }
                                
                                .username {
                                    font-size: 1.2em;
                                    font-weight: bold;
                                    margin: 0 0 10px 0;
                                }
                                
                                .email {
                                    color: #666;
                                    margin: 0;
                                }
                            </style>
                            
                            <div class="card">
                                <img class="avatar" 
                                     src="${this.avatar || 'default-avatar.png'}" 
                                     alt="Аватар ${this.username}">
                                <h3 class="username">${this.username || 'Анонимный пользователь'}</h3>
                                <p class="email">${this.email || 'Email не указан'}</p>
                            </div>
                        `;
                    }
                }
                
                customElements.define('modern-user-card', ModernUserCard);
            }
            
            createModernComponents() {
                const container = document.getElementById('web-component-container');
                
                const userCard = document.createElement('modern-user-card');
                userCard.setAttribute('username', 'Иван Иванов');
                userCard.setAttribute('email', 'ivan@example.com');
                userCard.setAttribute('avatar', 'avatar.jpg');
                
                container.appendChild(userCard);
            }
            
            registerFallbackComponents() {
                // Резервная реализация для старых браузеров
                class FallbackUserCard extends HTMLElement {
                    constructor() {
                        super();
                        
                        this.render();
                    }
                    
                    static get observedAttributes() {
                        return ['username', 'email', 'avatar'];
                    }
                    
                    connectedCallback() {
                        this.render();
                    }
                    
                    render() {
                        // В старых браузерах просто добавляем содержимое в обычный DOM
                        this.innerHTML = `
                            <div class="fallback-card">
                                <img class="avatar" 
                                     src="${this.getAttribute('avatar') || 'default-avatar.png'}" 
                                     alt="Аватар ${this.getAttribute('username') || 'Пользователь'}">
                                <h3 class="username">${this.getAttribute('username') || 'Анонимный пользователь'}</h3>
                                <p class="email">${this.getAttribute('email') || 'Email не указан'}</p>
                            </div>
                        `;
                        
                        // Добавляем стили
                        const style = document.createElement('style');
                        style.textContent = `
                            .fallback-card {
                                border: 1px solid #ddd;
                                border-radius: 8px;
                                padding: 20px;
                                background-color: white;
                                box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                                display: inline-block;
                            }
                            
                            .avatar {
                                width: 60px;
                                height: 60px;
                                border-radius: 50%;
                                object-fit: cover;
                                margin-bottom: 15px;
                            }
                            
                            .username {
                                font-size: 1.2em;
                                font-weight: bold;
                                margin: 0 0 10px 0;
                            }
                            
                            .email {
                                color: #666;
                                margin: 0;
                            }
                        `;
                        
                        document.head.appendChild(style);
                    }
                }
                
                customElements.define('fallback-user-card', FallbackUserCard);
            }
            
            createFallbackComponents() {
                const container = document.getElementById('web-component-container');
                
                const userCard = document.createElement('fallback-user-card');
                userCard.setAttribute('username', 'Иван Иванов');
                userCard.setAttribute('email', 'ivan@example.com');
                userCard.setAttribute('avatar', 'avatar.jpg');
                
                container.appendChild(userCard);
            }
            
            async loadScript(src) {
                return new Promise((resolve, reject) => {
                    const script = document.createElement('script');
                    script.src = src;
                    script.onload = resolve;
                    script.onerror = reject;
                    document.head.appendChild(script);
                });
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new WebComponentsCompatibility();
        });
    </script>
    
    <style>
        .web-components-compatibility {
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        
        #web-component-container {
            display: flex;
            gap: 20px;
            flex-wrap: wrap;
        }
    </style>
</body>
</html>
```

## Практические примеры и шаблоны

### Шаблон для совместимой формы
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Совместимая форма</title>
</head>
<body>
    <h1>Совместимая форма с резервными вариантами</h1>
    
    <form id="compatible-form" novalidate>
        <!-- Проверка на поддержку HTML5 -->
        <div class="form-group">
            <label for="name">Имя:</label>
            <input type="text"
                   id="name"
                   name="name"
                   required
                   minlength="2"
                   maxlength="50"
                   pattern="[A-Za-zА-Яа-яЁё\s\-]+"
                   title="Только буквы, пробелы и дефисы">
        </div>
        
        <div class="form-group">
            <label for="email">Email:</label>
            <input type="email"
                   id="email"
                   name="email"
                   required
                   autocomplete="email">
        </div>
        
        <div class="form-group">
            <label for="phone">Телефон:</label>
            <input type="tel"
                   id="phone"
                   name="phone"
                   autocomplete="tel"
                   pattern="[0-9\-\+\s\(\)]+"
                   title="Введите действительный номер телефона">
        </div>
        
        <div class="form-group">
            <label for="date">Дата рождения:</label>
            <input type="date" id="date" name="date">
        </div>
        
        <div class="form-group">
            <label for="range">Возраст:</label>
            <input type="range" 
                   id="range" 
                   name="age" 
                   min="1" 
                   max="120" 
                   value="25"
                   onchange="updateRangeValue(this.value)">
            <span id="range-value">25</span>
        </div>
        
        <div class="form-group">
            <label for="color">Любимый цвет:</label>
            <input type="color" id="color" name="color" value="#007acc">
        </div>
        
        <div class="form-group">
            <label for="file">Файл:</label>
            <input type="file" id="file" name="file" accept="image/*">
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox" name="newsletter" value="yes">
                Подписаться на рассылку
            </label>
        </div>
        
        <div class="form-group">
            <label>
                <input type="checkbox" name="terms" required>
                Я согласен с условиями использования
            </label>
        </div>
        
        <button type="submit">Отправить</button>
    </form>

    <script>
        class CompatibleFormHandler {
            constructor(formId) {
                this.form = document.getElementById(formId);
                this.setupCompatibilityCheck();
                this.setupValidation();
            }
            
            setupCompatibilityCheck() {
                // Проверка поддержки HTML5 возможностей
                const features = {
                    inputTypes: this.testInputTypes(),
                    formValidation: this.testFormValidation(),
                    fileAPI: this.testFileAPI(),
                    dateInput: this.testDateInput(),
                    rangeInput: this.testRangeInput(),
                    colorInput: this.testColorInput()
                };
                
                console.log('Возможности формы:', features);
                
                // Настройка резервных вариантов для неподдерживаемых возможностей
                this.setupFallbacks(features);
            }
            
            testInputTypes() {
                const input = document.createElement('input');
                
                return {
                    email: input.type === 'email',
                    tel: input.type === 'tel',
                    date: input.type === 'date',
                    range: input.type === 'range',
                    color: input.type === 'color',
                    url: input.type === 'url',
                    search: input.type === 'search'
                };
            }
            
            testFormValidation() {
                return 'checkValidity' in HTMLInputElement.prototype;
            }
            
            testFileAPI() {
                return 'FileReader' in window && 'File' in window;
            }
            
            testDateInput() {
                const input = document.createElement('input');
                input.type = 'date';
                return input.type === 'date';
            }
            
            testRangeInput() {
                const input = document.createElement('input');
                input.type = 'range';
                return input.type === 'range';
            }
            
            testColorInput() {
                const input = document.createElement('input');
                input.type = 'color';
                return input.type === 'color';
            }
            
            setupFallbacks(features) {
                // Установка резервных вариантов для неподдерживаемых возможностей
                
                if (!features.inputTypes.date) {
                    // Резервный вариант для date input
                    this.setupDateFallback();
                }
                
                if (!features.inputTypes.color) {
                    // Резервный вариант для color input
                    this.setupColorFallback();
                }
                
                if (!features.inputTypes.range) {
                    // Резервный вариант для range input
                    this.setupRangeFallback();
                }
            }
            
            setupDateFallback() {
                const dateInput = document.getElementById('date');
                const dateFallback = document.createElement('input');
                dateFallback.type = 'text';
                dateFallback.id = 'date-fallback';
                dateFallback.name = 'date';
                dateFallback.placeholder = 'ДД.ММ.ГГГГ';
                dateFallback.pattern = '\\d{2}\\.\\d{2}\\.\\d{4}';
                
                dateInput.parentNode.replaceChild(dateFallback, dateInput);
            }
            
            setupColorFallback() {
                const colorInput = document.getElementById('color');
                const colorFallback = document.createElement('select');
                colorFallback.id = 'color-fallback';
                colorFallback.name = 'color';
                
                const colors = [
                    { value: '#007acc', name: 'Синий' },
                    { value: '#28a745', name: 'Зеленый' },
                    { value: '#dc3545', name: 'Красный' },
                    { value: '#ffc107', name: 'Желтый' },
                    { value: '#6f42c1', name: 'Фиолетовый' }
                ];
                
                colors.forEach(color => {
                    const option = document.createElement('option');
                    option.value = color.value;
                    option.textContent = color.name;
                    colorFallback.appendChild(option);
                });
                
                colorInput.parentNode.replaceChild(colorFallback, colorInput);
            }
            
            setupRangeFallback() {
                const rangeInput = document.getElementById('range');
                const rangeFallback = document.createElement('input');
                rangeFallback.type = 'number';
                rangeFallback.id = 'range-fallback';
                rangeFallback.name = 'age';
                rangeFallback.min = '1';
                rangeFallback.max = '120';
                rangeFallback.value = '25';
                
                rangeInput.parentNode.replaceChild(rangeFallback, rangeInput);
            }
            
            setupValidation() {
                this.form.addEventListener('submit', (e) => {
                    e.preventDefault();
                    this.validateAndSubmit();
                });
                
                // Валидация в реальном времени
                this.form.addEventListener('input', (e) => {
                    if (e.target.matches('input[required]')) {
                        this.validateField(e.target);
                    }
                });
            }
            
            validateAndSubmit() {
                let isValid = true;
                
                const requiredFields = this.form.querySelectorAll('input[required]');
                requiredFields.forEach(field => {
                    if (!this.validateField(field)) {
                        isValid = false;
                    }
                });
                
                if (isValid) {
                    this.submitForm();
                } else {
                    alert('Пожалуйста, исправьте ошибки в форме');
                }
            }
            
            validateField(field) {
                field.classList.remove('invalid');
                
                if (field.hasAttribute('required') && !field.value.trim()) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                if (field.pattern && field.value && !new RegExp(field.pattern).test(field.value)) {
                    field.classList.add('invalid');
                    field.setAttribute('aria-invalid', 'true');
                    return false;
                }
                
                field.classList.remove('invalid');
                field.setAttribute('aria-invalid', 'false');
                return true;
            }
            
            async submitForm() {
                const submitBtn = this.form.querySelector('button[type="submit"]');
                const originalText = submitBtn.textContent;
                
                submitBtn.textContent = 'Отправка...';
                submitBtn.disabled = true;
                
                try {
                    const formData = new FormData(this.form);
                    const data = Object.fromEntries(formData);
                    
                    // Санитизация данных
                    const sanitizedData = {};
                    for (const [key, value] of Object.entries(data)) {
                        sanitizedData[key] = this.sanitizeInput(value);
                    }
                    
                    const response = await fetch('/api/submit', {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            'X-Requested-With': 'XMLHttpRequest'
                        },
                        body: JSON.stringify(sanitizedData)
                    });
                    
                    if (response.ok) {
                        alert('Форма успешно отправлена!');
                        this.form.reset();
                    } else {
                        const error = await response.json();
                        alert('Ошибка: ' + error.message);
                    }
                } catch (error) {
                    console.error('Ошибка отправки формы:', error);
                    alert('Ошибка сети при отправке формы');
                } finally {
                    submitBtn.textContent = originalText;
                    submitBtn.disabled = false;
                }
            }
            
            sanitizeInput(input) {
                if (typeof input !== 'string') return input;
                
                return input
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;')
                    .replace(/&/g, '&amp;')
                    .trim();
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new CompatibleFormHandler('compatible-form');
        });
        
        // Обновление значения range
        function updateRangeValue(value) {
            document.getElementById('range-value').textContent = value;
        }
    </script>
    
    <style>
        .form-group {
            margin-bottom: 1.5rem;
        }
        
        label {
            display: block;
            margin-bottom: 0.5rem;
            font-weight: bold;
        }
        
        input {
            width: 100%;
            padding: 0.75rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }
        
        input:focus {
            outline: none;
            border-color: #007acc;
            box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.2);
        }
        
        input.invalid {
            border-color: #dc3545;
            background-color: #ffebee;
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
        
        #range-value {
            margin-left: 10px;
            font-weight: bold;
        }
    </style>
</body>
</html>
```

### Шаблон для адаптивной таблицы с совместимостью
```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Адаптивная таблица с совместимостью</title>
</head>
<body>
    <h1>Адаптивная таблица с совместимостью</h1>
    
    <div class="table-container">
        <div class="table-controls">
            <input type="text" id="table-search" placeholder="Поиск...">
            <select id="table-sort">
                <option value="">Сортировка</option>
                <option value="name-asc">Имя (А-Я)</option>
                <option value="name-desc">Имя (Я-А)</option>
                <option value="date-asc">Дата (старые)</option>
                <option value="date-desc">Дата (новые)</option>
            </select>
        </div>
        
        <div class="table-wrapper">
            <table id="compatible-table">
                <thead>
                    <tr>
                        <th data-sort="name">Имя</th>
                        <th data-sort="email">Email</th>
                        <th data-sort="role">Роль</th>
                        <th data-sort="date">Дата регистрации</th>
                        <th>Действия</th>
                    </tr>
                </thead>
                <tbody id="table-body">
                    <!-- Данные будут добавлены сюда -->
                </tbody>
            </table>
        </div>
        
        <div id="pagination" class="pagination"></div>
    </div>

    <script>
        class CompatibleTable {
            constructor(tableId) {
                this.table = document.getElementById(tableId);
                this.body = document.getElementById('table-body');
                this.searchInput = document.getElementById('table-search');
                this.sortSelect = document.getElementById('table-sort');
                this.pagination = document.getElementById('pagination');
                
                this.data = [
                    { id: 1, name: 'Иван Иванов', email: 'ivan@example.com', role: 'Администратор', date: '2023-01-15' },
                    { id: 2, name: 'Мария Петрова', email: 'maria@example.com', role: 'Модератор', date: '2023-02-20' },
                    { id: 3, name: 'Алексей Сидоров', email: 'alexey@example.com', role: 'Пользователь', date: '2023-03-10' },
                    { id: 4, name: 'Елена Козлова', email: 'elena@example.com', role: 'Пользователь', date: '2023-04-05' },
                    { id: 5, name: 'Дмитрий Волков', email: 'dmitry@example.com', role: 'Модератор', date: '2023-05-12' }
                ];
                
                this.filteredData = [...this.data];
                this.currentPage = 1;
                this.pageSize = 5;
                
                this.setupEventListeners();
                this.renderTable();
                this.setupCompatibility();
            }
            
            setupEventListeners() {
                this.searchInput.addEventListener('input', this.debounce(() => {
                    this.filterData();
                }, 300));
                
                this.sortSelect.addEventListener('change', () => {
                    this.sortData();
                });
            }
            
            setupCompatibility() {
                // Проверка поддержки CSS Grid и Flexbox
                const supportsGrid = CSS.supports('display', 'grid');
                const supportsFlex = CSS.supports('display', 'flex');
                
                if (!supportsGrid && !supportsFlex) {
                    // Резервный режим для старых браузеров
                    this.enableTableFallback();
                }
            }
            
            enableTableFallback() {
                // Добавление резервных стилей для старых браузеров
                const fallbackStyles = document.createElement('style');
                fallbackStyles.textContent = `
                    .table-wrapper {
                        overflow-x: auto;
                    }
                    
                    #compatible-table {
                        width: 100%;
                        border-collapse: collapse;
                    }
                    
                    th, td {
                        padding: 10px;
                        text-align: left;
                        border-bottom: 1px solid #ddd;
                    }
                    
                    th {
                        background-color: #f8f9fa;
                        cursor: pointer;
                    }
                `;
                
                document.head.appendChild(fallbackStyles);
            }
            
            filterData() {
                const searchTerm = this.searchInput.value.toLowerCase();
                
                if (!searchTerm) {
                    this.filteredData = [...this.data];
                } else {
                    this.filteredData = this.data.filter(row => 
                        Object.values(row).some(value => 
                            String(value).toLowerCase().includes(searchTerm)
                        )
                    );
                }
                
                this.currentPage = 1;
                this.renderTable();
                this.renderPagination();
            }
            
            sortData() {
                const sortValue = this.sortSelect.value;
                if (!sortValue) return;
                
                const [field, direction] = sortValue.split('-');
                
                this.filteredData.sort((a, b) => {
                    let aVal = a[field];
                    let bVal = b[field];
                    
                    // Для дат преобразуем в объект Date
                    if (field === 'date') {
                        aVal = new Date(aVal);
                        bVal = new Date(bVal);
                    }
                    
                    if (aVal < bVal) return direction === 'asc' ? -1 : 1;
                    if (aVal > bVal) return direction === 'asc' ? 1 : -1;
                    return 0;
                });
                
                this.renderTable();
            }
            
            renderTable() {
                const startIndex = (this.currentPage - 1) * this.pageSize;
                const endIndex = startIndex + this.pageSize;
                const pageData = this.filteredData.slice(startIndex, endIndex);
                
                this.body.innerHTML = pageData.map(row => `
                    <tr data-id="${row.id}">
                        <td>${this.escapeHTML(row.name)}</td>
                        <td>${this.escapeHTML(row.email)}</td>
                        <td>${this.escapeHTML(row.role)}</td>
                        <td>${this.formatDate(row.date)}</td>
                        <td>
                            <button class="action-btn edit-btn" onclick="table.editRow(${row.id})">Редактировать</button>
                            <button class="action-btn delete-btn" onclick="table.deleteRow(${row.id})">Удалить</button>
                        </td>
                    </tr>
                `).join('');
            }
            
            renderPagination() {
                const totalPages = Math.ceil(this.filteredData.length / this.pageSize);
                if (totalPages <= 1) {
                    this.pagination.innerHTML = '';
                    return;
                }
                
                const pages = [];
                
                // Кнопка "Назад"
                pages.push(`<button ${this.currentPage === 1 ? 'disabled' : ''} onclick="table.goToPage(${this.currentPage - 1})">←</button>`);
                
                // Номера страниц
                for (let i = 1; i <= totalPages; i++) {
                    pages.push(`<button ${i === this.currentPage ? 'class="active"' : ''} onclick="table.goToPage(${i})">${i}</button>`);
                }
                
                // Кнопка "Вперед"
                pages.push(`<button ${this.currentPage === totalPages ? 'disabled' : ''} onclick="table.goToPage(${this.currentPage + 1})">→</button>`);
                
                this.pagination.innerHTML = pages.join('');
            }
            
            goToPage(page) {
                if (page < 1 || page > Math.ceil(this.filteredData.length / this.pageSize)) return;
                
                this.currentPage = page;
                this.renderTable();
                this.renderPagination();
            }
            
            editRow(id) {
                console.log('Редактирование строки с ID:', id);
                // Реализация редактирования
            }
            
            deleteRow(id) {
                if (confirm('Удалить запись?')) {
                    this.data = this.data.filter(row => row.id !== id);
                    this.filteredData = this.filteredData.filter(row => row.id !== id);
                    this.renderTable();
                    this.renderPagination();
                }
            }
            
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
            
            escapeHTML(str) {
                if (typeof str !== 'string') return str;
                
                return str
                    .replace(/&/g, '&amp;')
                    .replace(/</g, '&lt;')
                    .replace(/>/g, '&gt;')
                    .replace(/"/g, '&quot;')
                    .replace(/'/g, '&#x27;');
            }
            
            formatDate(dateString) {
                return new Date(dateString).toLocaleDateString('ru-RU');
            }
        }
        
        // Инициализация
        let table;
        document.addEventListener('DOMContentLoaded', () => {
            table = new CompatibleTable('compatible-table');
        });
    </script>
    
    <style>
        .table-container {
            max-width: 1000px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .table-controls {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }
        
        #table-search, #table-sort {
            padding: 8px 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        
        .table-wrapper {
            overflow-x: auto;
        }
        
        #compatible-table {
            width: 100%;
            border-collapse: collapse;
        }
        
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #eee;
        }
        
        th {
            background-color: #f8f9fa;
            font-weight: bold;
            position: sticky;
            top: 0;
            cursor: pointer;
        }
        
        th:hover {
            background-color: #e9ecef;
        }
        
        tr:hover {
            background-color: #f5f5f5;
        }
        
        .pagination {
            margin-top: 20px;
            display: flex;
            gap: 5px;
            justify-content: center;
        }
        
        .pagination button {
            padding: 8px 12px;
            border: 1px solid #ddd;
            background-color: white;
            cursor: pointer;
        }
        
        .pagination button.active {
            background-color: #007acc;
            color: white;
        }
        
        .pagination button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }
        
        .action-btn {
            padding: 4px 8px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.8em;
        }
        
        .edit-btn {
            background-color: #ffc107;
            color: #333;
        }
        
        .delete-btn {
            background-color: #dc3545;
            color: white;
        }
    </style>
</body>
</html>
```

## Проверка и тестирование совместимости

### Инструменты для проверки совместимости:
1. **Can I Use** - проверка поддержки возможностей браузерами
2. **BrowserStack** - тестирование на реальных устройствах и браузерах
3. **Sauce Labs** - облачное тестирование браузеров
4. **CrossBrowserTesting** - тестирование совместимости
5. **Babel** - транспиляция современного JavaScript
6. **PostCSS** - автоматическое добавление префиксов CSS
7. **Autoprefixer** - автоматическое добавление вендорных префиксов
8. **Modernizr** - определение возможностей браузера
9. **Polyfill.io** - автоматическая загрузка полифиллов
10. **Browserslist** - настройка поддерживаемых браузеров

### Проверка совместимости:
1. Тестирование в различных браузерах (Chrome, Firefox, Safari, Edge, IE)
2. Проверка на мобильных устройствах
3. Тестирование с различными версиями браузеров
4. Проверка с отключенными JavaScript
5. Проверка с включенным/выключенным CSS
6. Тестирование с различными размерами экрана
7. Проверка производительности на слабых устройствах
8. Тестирование с различными сетевыми условиями

### Проверка на устаревших браузерах:
1. Использование полифиллов для недостающих возможностей
2. Проверка резервных вариантов
3. Тестирование с IE 11 и более старыми версиями
4. Проверка на старых мобильных браузерах
5. Тестирование с устаревшими версиями Safari

## Лучшие практики совместимости HTML

### 1. Прогрессивное улучшение
```html
<!-- Базовая функциональность -->
<form action="/submit" method="post">
    <input type="text" name="data" placeholder="Введите данные">
    <button type="submit">Отправить</button>
</form>

<!-- Улучшенная функциональность для современных браузеров -->
<script>
if ('fetch' in window && 'Promise' in window) {
    // Использовать современные возможности
    document.addEventListener('DOMContentLoaded', () => {
        // Инициализировать улучшенные функции
        new ModernFormHandler();
    });
} else {
    // Оставить базовую функциональность
    console.log('Используется базовая функциональность');
}
</script>
```

### 2. Feature Detection
```javascript
// Правильная проверка возможностей
class FeatureDetector {
    static detect() {
        return {
            // HTML5 API
            canvas: !!document.createElement('canvas').getContext,
            webgl: (function() {
                try {
                    const canvas = document.createElement('canvas');
                    return !!(window.WebGLRenderingContext && 
                             (canvas.getContext('webgl') || canvas.getContext('experimental-webgl')));
                } catch(e) {
                    return false;
                }
            })(),
            
            // CSS возможности
            cssGrid: CSS.supports('display', 'grid'),
            cssFlexbox: CSS.supports('display', 'flex'),
            cssVariables: CSS.supports('--custom', 'property'),
            mediaQueries: CSS.supports('(min-width: 1px)'),
            
            // JavaScript возможности
            fetch: !!window.fetch,
            promise: !!window.Promise,
            asyncAwait: (function() {
                try {
                    eval('async function test() {}');
                    return true;
                } catch(e) {
                    return false;
                }
            })(),
            
            // DOM возможности
            querySelector: !!document.querySelector,
            addEventListener: !!document.addEventListener,
            classList: !!document.createElement('div').classList,
            
            // Storage API
            localStorage: !!window.localStorage,
            sessionStorage: !!window.sessionStorage,
            indexedDB: !!window.indexedDB,
            
            // Device API
            geolocation: !!navigator.geolocation,
            touch: ('ontouchstart' in window) || (navigator.maxTouchPoints > 0),
            webSockets: !!window.WebSocket,
            
            // Modern APIs
            intersectionObserver: !!window.IntersectionObserver,
            mutationObserver: !!window.MutationObserver,
            webWorkers: !!window.Worker,
            serviceWorker: 'serviceWorker' in navigator,
            webComponents: !!window.customElements,
            shadowDOM: !!Element.prototype.attachShadow
        };
    }
    
    static getCompatibilityLevel(features) {
        const supportedCount = Object.values(features).filter(f => f).length;
        const totalCount = Object.keys(features).length;
        const percentage = (supportedCount / totalCount) * 100;
        
        if (percentage >= 90) return 'excellent';
        if (percentage >= 75) return 'good';
        if (percentage >= 50) return 'fair';
        return 'poor';
    }
}

// Использование
const features = FeatureDetector.detect();
const level = FeatureDetector.getCompatibilityLevel(features);

console.log('Уровень совместимости:', level);
console.log('Возможности:', features);
```

### 3. Резервные варианты и полифиллы
```html
<!-- Условная загрузка полифиллов -->
<script>
(function() {
    'use strict';
    
    // Проверка поддержки fetch
    if (!window.fetch) {
        var script = document.createElement('script');
        script.src = 'https://cdn.jsdelivr.net/npm/whatwg-fetch@3.6.2/dist/fetch.umd.js';
        document.head.appendChild(script);
    }
    
    // Проверка поддержки Promise
    if (!window.Promise) {
        var promiseScript = document.createElement('script');
        promiseScript.src = 'https://cdn.jsdelivr.net/npm/es6-promise@4.2.8/dist/es6-promise.auto.min.js';
        document.head.appendChild(promiseScript);
    }
    
    // Проверка поддержки Web Components
    if (!window.customElements) {
        var wcScript = document.createElement('script');
        wcScript.src = 'https://cdn.jsdelivr.net/npm/@webcomponents/webcomponentsjs@2.8.0/webcomponents-bundle.js';
        document.head.appendChild(wcScript);
    }
})();
</script>
```

### 4. Универсальные стили для совместимости
```css
/* Резервные стили для старых браузеров */
.container {
    /* Резервный метод для старых браузеров */
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
}

/* Современные стили с резервными вариантами */
.grid-container {
    /* Резерв для старых браузеров */
    display: block;
}

/* Современная реализация */
@supports (display: grid) {
    .grid-container {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 20px;
    }
}

.flex-container {
    /* Резерв для старых браузеров */
    display: block;
}

@supports (display: flex) {
    .flex-container {
        display: flex;
        flex-wrap: wrap;
        gap: 20px;
    }
}
```

## Заключение

Совместимость HTML-страниц - это комплексный подход, который включает в себя оптимизацию структуры документа, загрузки ресурсов, работы с DOM, валидации форм и других аспектов. Современные возможности, такие как Web Workers, WebAssembly, Intersection Observer и requestAnimationFrame, позволяют создавать высокопроизводительные веб-приложения, которые обеспечивают отличный пользовательский опыт.

Ключевые аспекты совместимости HTML:
1. Оптимизация загрузки ресурсов
2. Эффективная работа с DOM
3. Использование современных API
4. Кеширование данных и ресурсов
5. Оптимизация изображений
6. Асинхронная загрузка контента
7. Использование веб-компонентов
8. Управление памятью
9. Оптимизация CSS и JavaScript
10. Следование современным веб-стандартам
11. Регулярное тестирование производительности
12. Использование инструментов мониторинга
13. Поддержка резервных вариантов для устаревших браузеров
14. Использование feature detection вместо browser detection
15. Правильная реализация прогрессивного улучшения

Эти практики обеспечивают создание быстрых, отзывчивых и эффективных веб-приложений, которые работают хорошо на всех устройствах и в различных сетевых условиях.

Современные подходы к совместимости включают:
- Использование Web Workers для тяжелых вычислений
- WebAssembly для высокопроизводительных операций
- Intersection Observer для ленивой загрузки
- Virtual scrolling для больших списков
- Efficient DOM manipulation techniques
- Proper caching strategies with Service Workers and IndexedDB
- Optimized resource loading with modern HTML attributes
- Performance monitoring and optimization
- Following web performance best practices and standards
- Using feature detection instead of browser detection
- Implementing progressive enhancement patterns
- Providing fallbacks for older browsers
- Supporting graceful degradation
- Using polyfills strategically

Эти технологии позволяют создавать действительно совместимые веб-приложения, которые обеспечивают отличный пользовательский опыт независимо от устройства или условий подключения.

Ключевые принципы совместимости:
1. Progressive enhancement - создание базовой функциональности для всех браузеров
2. Graceful degradation - постепенное снижение функциональности в старых браузерах
3. Feature detection - определение возможностей браузера вместо определения версии
4. Fallback strategies - резервные варианты для недостающих возможностей
5. Polyfill management - стратегическое использование полифиллов
6. Cross-browser testing - регулярное тестирование в разных браузерах
7. Performance optimization - оптимизация для слабых устройств
8. Standard compliance - следование современным веб-стандартам
9. Accessibility considerations - обеспечение доступности для всех пользователей
10. Security considerations - обеспечение безопасности во всех браузерах

Эти принципы помогают создавать веб-приложения, которые работают корректно во всех поддерживаемых браузерах и обеспечивают качественный пользовательский опыт.

## Следующие темы
- [[HTML/Безопасность]]
- [[HTML/Доступность]]
- [[HTML/Производительность]]

## Теги
#html #compatibility #web-development #frontend #cross-browser #legacy-browsers #web-components #polyfills #feature-detection #progressive-enhancement #graceful-degradation #web-standards #caniuse #browserslist #modernizr #web-apis #dom-manipulation #css-grid #flexbox #javascript #es6 #es2015 #accessibility #performance #security #responsive-design #mobile-compatibility #ie-support #safari-compatibility #firefox-compatibility #chrome-compatibility #edge-compatibility #web-standards #best-practices
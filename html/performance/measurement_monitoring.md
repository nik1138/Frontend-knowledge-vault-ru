# HTML Производительность: Измерение и мониторинг

Измерение и мониторинг производительности HTML-страниц является критически важным аспектом разработки высококачественных веб-приложений. Современные инструменты и API позволяют разработчикам отслеживать ключевые метрики, выявлять узкие места и оптимизировать пользовательский опыт.

## Ключевые метрики производительности

### Core Web Vitals

Core Web Vitals - это набор метрик, разработанных Google для измерения качества пользовательского опыта:

1. **LCP (Largest Contentful Paint)** - время отображения largest content
2. **FID (First Input Delay)** - задержка первого ввода
3. **CLS (Cumulative Layout Shift)** - cumulative layout shift

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Core Web Vitals</title>
</head>
<body>
    <h1>Измерение Core Web Vitals</h1>
    
    <div class="content">
        <img src="hero-image.jpg" alt="Главное изображение" id="hero-image">
        <p>Это содержимое будет использоваться для измерения Largest Contentful Paint.</p>
    </div>
    
    <button id="interactive-btn">Кнопка для измерения FID</button>
    
    <div id="shift-test">
        <p>Этот контент может вызвать сдвиг макета.</p>
    </div>

    <script>
        // Измерение Largest Contentful Paint
        new PerformanceObserver((entryList) => {
            const entries = entryList.getEntries();
            const lastEntry = entries[entries.length - 1];
            
            console.log('LCP:', lastEntry.startTime);
            
            // Отправка метрики
            analytics.track('LCP', {
                value: lastEntry.startTime,
                element: lastEntry.element ? lastEntry.element.tagName : 'unknown'
            });
        }).observe({ entryTypes: ['largest-contentful-paint'] });

        // Измерение First Input Delay
        new PerformanceObserver((entryList) => {
            const entries = entryList.getEntries();
            
            entries.forEach(entry => {
                console.log('FID:', entry.processingStart - entry.startTime);
                
                // Отправка метрики
                analytics.track('FID', {
                    value: entry.processingStart - entry.startTime,
                    interactionType: entry.name
                });
            });
        }).observe({ entryTypes: ['first-input'] });

        // Измерение Cumulative Layout Shift
        let clsValue = 0;
        let clsEntries = [];

        new PerformanceObserver((entryList) => {
            const entries = entryList.getEntries();
            
            entries.forEach(entry => {
                if (!entry.hadRecentInput) {
                    clsValue += entry.value;
                    clsEntries.push(entry);
                }
            });
            
            console.log('Current CLS:', clsValue);
            
            // Отправка метрики
            analytics.track('CLS', {
                value: clsValue,
                entries: clsEntries.length
            });
        }).observe({ entryTypes: ['layout-shift'] });
    </script>
</body>
</html>
```

### Время загрузки страницы

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Время загрузки страницы</title>
</head>
<body>
    <h1>Измерение времени загрузки</h1>
    
    <div id="timing-results"></div>

    <script>
        class PageTimingAnalyzer {
            constructor() {
                this.timingResults = document.getElementById('timing-results');
                this.measurePageTiming();
            }
            
            measurePageTiming() {
                // Использование Navigation Timing API
                const timing = performance.timing;
                
                const metrics = {
                    // Время загрузки ресурсов
                    dnsLookup: timing.domainLookupEnd - timing.domainLookupStart,
                    tcpConnection: timing.connectEnd - timing.connectStart,
                    requestResponse: timing.responseEnd - timing.requestStart,
                    domProcessing: timing.domComplete - timing.domLoading,
                    pageLoad: timing.loadEventEnd - timing.navigationStart,
                    
                    // Ключевые моменты
                    domContentLoaded: timing.domContentLoadedEventEnd - timing.navigationStart,
                    firstPaint: this.getFirstPaintTime(),
                    firstContentfulPaint: this.getFirstContentfulPaintTime()
                };
                
                this.displayTimingResults(metrics);
                this.reportTimingToAnalytics(metrics);
            }
            
            getFirstPaintTime() {
                // Использование Paint Timing API
                const paintEntries = performance.getEntriesByType('paint');
                const firstPaint = paintEntries.find(entry => entry.name === 'first-paint');
                return firstPaint ? firstPaint.startTime : null;
            }
            
            getFirstContentfulPaintTime() {
                // Использование Paint Timing API для FCP
                const paintEntries = performance.getEntriesByType('paint');
                const fcp = paintEntries.find(entry => entry.name === 'first-contentful-paint');
                return fcp ? fcp.startTime : null;
            }
            
            displayTimingResults(metrics) {
                this.timingResults.innerHTML = `
                    <div class="timing-metrics">
                        <h3>Метрики времени загрузки:</h3>
                        <ul>
                            <li><strong>DNS Lookup:</strong> ${metrics.dnsLookup}ms</li>
                            <li><strong>TCP Connection:</strong> ${metrics.tcpConnection}ms</li>
                            <li><strong>Request/Response:</strong> ${metrics.requestResponse}ms</li>
                            <li><strong>DOM Processing:</strong> ${metrics.domProcessing}ms</li>
                            <li><strong>Page Load Time:</strong> ${metrics.pageLoad}ms</li>
                            <li><strong>DOM Content Loaded:</strong> ${metrics.domContentLoaded}ms</li>
                            <li><strong>First Paint:</strong> ${metrics.firstPaint}ms</li>
                            <li><strong>First Contentful Paint:</strong> ${metrics.firstContentfulPaint}ms</li>
                        </ul>
                    </div>
                `;
            }
            
            reportTimingToAnalytics(metrics) {
                // Отправка метрик в систему аналитики
                if ('sendBeacon' in navigator) {
                    navigator.sendBeacon('/analytics/timing', JSON.stringify({
                        page: window.location.href,
                        timestamp: new Date().toISOString(),
                        metrics: metrics
                    }));
                } else {
                    // Резервный метод для старых браузеров
                    fetch('/analytics/timing', {
                        method: 'POST',
                        body: JSON.stringify({
                            page: window.location.href,
                            timestamp: new Date().toISOString(),
                            metrics: metrics
                        }),
                        headers: {
                            'Content-Type': 'application/json'
                        }
                    }).catch(console.error);
                }
            }
        }
        
        // Инициализация при загрузке
        window.addEventListener('load', () => {
            new PageTimingAnalyzer();
        });
    </script>
    
    <style>
        .timing-metrics {
            background-color: #f8f9fa;
            border: 1px solid #dee2e6;
            border-radius: 8px;
            padding: 20px;
            margin: 20px 0;
        }
        
        .timing-metrics ul {
            list-style: none;
            padding: 0;
        }
        
        .timing-metrics li {
            padding: 5px 0;
            border-bottom: 1px solid #eee;
        }
        
        .timing-metrics li:last-child {
            border-bottom: none;
        }
    </style>
</body>
</html>
```

## Performance API

### Navigation Timing

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Navigation Timing API</title>
</head>
<body>
    <h1>Navigation Timing API</h1>
    
    <div id="navigation-timing"></div>
    
    <script>
        class NavigationTimingAnalyzer {
            constructor() {
                this.analyzeNavigationTiming();
            }
            
            analyzeNavigationTiming() {
                const timing = performance.getEntriesByType('navigation')[0];
                
                if (!timing) {
                    console.error('Navigation timing недоступен');
                    return;
                }
                
                const metrics = {
                    // Время загрузки
                    unloadEventTime: timing.unloadEventEnd - timing.unloadEventStart,
                    redirectTime: timing.redirectEnd - timing.redirectStart,
                    appCacheTime: timing.domainLookupStart - timing.fetchStart,
                    dnsTime: timing.domainLookupEnd - timing.domainLookupStart,
                    tcpTime: timing.connectEnd - timing.connectStart,
                    requestTime: timing.responseStart - timing.requestStart,
                    responseTime: timing.responseEnd - timing.responseStart,
                    domProcessingTime: timing.domComplete - timing.domInteractive,
                    loadEventTime: timing.loadEventEnd - timing.loadEventStart,
                    
                    // Общее время загрузки
                    totalTime: timing.loadEventEnd - timing.navigationStart,
                    
                    // Важные моменты
                    domContentLoadedTime: timing.domContentLoadedEventEnd - timing.navigationStart,
                    firstByteTime: timing.responseStart - timing.navigationStart,
                    ttfb: timing.responseStart - timing.requestStart
                };
                
                this.displayNavigationMetrics(metrics);
                this.checkPerformanceThresholds(metrics);
            }
            
            displayNavigationMetrics(metrics) {
                const container = document.getElementById('navigation-timing');
                
                container.innerHTML = `
                    <div class="metrics-container">
                        <h3>Метрики Navigation Timing:</h3>
                        <div class="metrics-grid">
                            <div class="metric-card">
                                <h4>Время загрузки</h4>
                                <p><strong>Общее:</strong> ${metrics.totalTime}ms</p>
                                <p><strong>DOM Content Loaded:</strong> ${metrics.domContentLoadedTime}ms</p>
                                <p><strong>Time to First Byte:</strong> ${metrics.ttfb}ms</p>
                            </div>
                            
                            <div class="metric-card">
                                <h4>Сетевые метрики</h4>
                                <p><strong>DNS:</strong> ${metrics.dnsTime}ms</p>
                                <p><strong>TCP:</strong> ${metrics.tcpTime}ms</p>
                                <p><strong>Запрос:</strong> ${metrics.requestTime}ms</p>
                                <p><strong>Ответ:</strong> ${metrics.responseTime}ms</p>
                            </div>
                            
                            <div class="metric-card">
                                <h4>Обработка DOM</h4>
                                <p><strong>DOM Processing:</strong> ${metrics.domProcessingTime}ms</p>
                                <p><strong>Load Event:</strong> ${metrics.loadEventTime}ms</p>
                                <p><strong>Redirect:</strong> ${metrics.redirectTime}ms</p>
                            </div>
                        </div>
                    </div>
                `;
            }
            
            checkPerformanceThresholds(metrics) {
                const thresholds = {
                    totalTime: 3000, // 3 секунды
                    ttfb: 200,     // 200мс
                    domContentLoadedTime: 2000 // 2 секунды
                };
                
                const issues = [];
                
                if (metrics.totalTime > thresholds.totalTime) {
                    issues.push(`Общее время загрузки превышает порог: ${metrics.totalTime}ms > ${thresholds.totalTime}ms`);
                }
                
                if (metrics.ttfb > thresholds.ttfb) {
                    issues.push(`Time to First Byte превышает порог: ${metrics.ttfb}ms > ${thresholds.ttfb}ms`);
                }
                
                if (metrics.domContentLoadedTime > thresholds.domContentLoadedTime) {
                    issues.push(`DOM Content Loaded превышает порог: ${metrics.domContentLoadedTime}ms > ${thresholds.domContentLoadedTime}ms`);
                }
                
                if (issues.length > 0) {
                    this.reportPerformanceIssues(issues);
                }
            }
            
            reportPerformanceIssues(issues) {
                console.warn('Проблемы с производительностью:', issues);
                
                // Отправка отчета о проблемах
                fetch('/api/performance-issues', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        url: window.location.href,
                        timestamp: new Date().toISOString(),
                        issues: issues
                    })
                }).catch(console.error);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new NavigationTimingAnalyzer();
        });
    </script>
    
    <style>
        .metrics-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .metrics-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        
        .metric-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
            background-color: white;
        }
        
        .metric-card h4 {
            margin-top: 0;
            color: #333;
        }
        
        .metric-card p {
            margin: 8px 0;
        }
    </style>
</body>
</html>
```

### Resource Timing

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Resource Timing API</title>
    <link rel="stylesheet" href="main.css">
    <link rel="preload" href="critical.css" as="style">
</head>
<body>
    <h1>Resource Timing API</h1>
    
    <div id="resource-timing"></div>
    
    <img src="hero-image.jpg" alt="Герой изображение">
    <script src="app.js"></script>

    <script>
        class ResourceTimingAnalyzer {
            constructor() {
                this.analyzeResourceTiming();
            }
            
            analyzeResourceTiming() {
                const resources = performance.getEntriesByType('resource');
                
                const resourceMetrics = resources.map(resource => ({
                    name: resource.name,
                    entryType: resource.entryType,
                    startTime: resource.startTime,
                    duration: resource.duration,
                    transferSize: resource.transferSize,
                    encodedBodySize: resource.encodedBodySize,
                    decodedBodySize: resource.decodedBodySize,
                    contentType: resource.responseEnd > resource.requestStart ? 
                               this.getContentType(resource.name) : 'unknown',
                    
                    // Временные метрики
                    redirectTime: resource.redirectEnd - resource.redirectStart,
                    dnsTime: resource.domainLookupEnd - resource.domainLookupStart,
                    tcpTime: resource.connectEnd - resource.connectStart,
                    requestTime: resource.responseStart - resource.requestStart,
                    responseTime: resource.responseEnd - resource.responseStart,
                    processingTime: performance.now() - resource.responseEnd
                }));
                
                this.displayResourceMetrics(resourceMetrics);
                this.identifySlowResources(resourceMetrics);
                this.optimizeResourceLoading(resourceMetrics);
            }
            
            displayResourceMetrics(resources) {
                const container = document.getElementById('resource-timing');
                
                container.innerHTML = `
                    <div class="resources-analysis">
                        <h3>Анализ ресурсов (${resources.length} загружено):</h3>
                        <table class="resources-table">
                            <thead>
                                <tr>
                                    <th>Ресурс</th>
                                    <th>Тип</th>
                                    <th>Время (мс)</th>
                                    <th>Размер (байт)</th>
                                    <th>Тип контента</th>
                                    <th>Статус</th>
                                </tr>
                            </thead>
                            <tbody>
                                ${resources.map(resource => `
                                    <tr class="${this.getResourceStatusClass(resource)}">
                                        <td>${this.shortenUrl(resource.name)}</td>
                                        <td>${resource.contentType}</td>
                                        <td>${resource.duration.toFixed(2)}</td>
                                        <td>${this.formatBytes(resource.transferSize)}</td>
                                        <td>${resource.contentType}</td>
                                        <td>${this.getResourceStatus(resource)}</td>
                                    </tr>
                                `).join('')}
                            </tbody>
                        </table>
                    </div>
                `;
            }
            
            getResourceStatusClass(resource) {
                if (resource.duration > 1000) return 'slow-resource';
                if (resource.transferSize > 1024 * 100) return 'large-resource'; // > 100KB
                return 'normal-resource';
            }
            
            getResourceStatus(resource) {
                if (resource.duration > 1000) return ' медленный';
                if (resource.transferSize > 1024 * 100) return ' большой';
                return ' нормальный';
            }
            
            identifySlowResources(resources) {
                const slowResources = resources.filter(r => r.duration > 1000);
                
                if (slowResources.length > 0) {
                    console.warn('Медленные ресурсы (>1000ms):', slowResources);
                    
                    // Отчет о медленных ресурсах
                    this.reportSlowResources(slowResources);
                }
            }
            
            optimizeResourceLoading(resources) {
                // Анализ ресурсов для оптимизации
                const largeImages = resources.filter(r => 
                    r.contentType.includes('image') && r.transferSize > 1024 * 500 // > 500KB
                );
                
                const slowScripts = resources.filter(r => 
                    r.contentType.includes('javascript') && r.duration > 500
                );
                
                if (largeImages.length > 0) {
                    console.log('Большие изображения для оптимизации:', largeImages);
                }
                
                if (slowScripts.length > 0) {
                    console.log('Медленные скрипты для оптимизации:', slowScripts);
                }
            }
            
            reportSlowResources(resources) {
                const report = {
                    page: window.location.href,
                    timestamp: new Date().toISOString(),
                    slowResources: resources.map(r => ({
                        url: r.name,
                        duration: r.duration,
                        size: r.transferSize,
                        type: r.contentType
                    }))
                };
                
                // Отправка отчета
                if (navigator.sendBeacon) {
                    navigator.sendBeacon('/api/slow-resources', JSON.stringify(report));
                } else {
                    fetch('/api/slow-resources', {
                        method: 'POST',
                        body: JSON.stringify(report),
                        headers: { 'Content-Type': 'application/json' }
                    }).catch(console.error);
                }
            }
            
            shortenUrl(url) {
                try {
                    const urlObj = new URL(url);
                    return urlObj.pathname.split('/').pop() || urlObj.hostname;
                } catch {
                    return url;
                }
            }
            
            formatBytes(bytes) {
                if (bytes === 0) return '0 Bytes';
                
                const k = 1024;
                const sizes = ['Bytes', 'KB', 'MB', 'GB'];
                const i = Math.floor(Math.log(bytes) / Math.log(k));
                
                return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
            }
            
            getContentType(url) {
                if (url.endsWith('.css')) return 'stylesheet';
                if (url.endsWith('.js')) return 'script';
                if (url.endsWith('.jpg') || url.endsWith('.jpeg') || url.endsWith('.png') || url.endsWith('.gif') || url.endsWith('.webp')) return 'image';
                if (url.endsWith('.woff') || url.endsWith('.woff2')) return 'font';
                if (url.endsWith('.json')) return 'json';
                if (url.endsWith('.xml')) return 'xml';
                return 'other';
            }
        }
        
        // Инициализация после полной загрузки страницы
        window.addEventListener('load', () => {
            new ResourceTimingAnalyzer();
        });
    </script>
    
    <style>
        .resources-analysis {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .resources-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 20px;
        }
        
        .resources-table th,
        .resources-table td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        
        .resources-table th {
            background-color: #f8f9fa;
            font-weight: bold;
        }
        
        .slow-resource {
            background-color: #ffebee;
        }
        
        .large-resource {
            background-color: #fff3cd;
        }
        
        .normal-resource {
            background-color: #f8f9fa;
        }
    </style>
</body>
</html>
```

## User Timing API

### Измерение пользовательских метрик

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>User Timing API</title>
</head>
<body>
    <h1>User Timing API</h1>
    
    <div id="user-timing"></div>
    
    <button id="measure-btn">Измерить производительность</button>
    <button id="complex-operation-btn">Выполнить сложную операцию</button>

    <script>
        class UserTimingAnalyzer {
            constructor() {
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                document.getElementById('measure-btn').addEventListener('click', () => {
                    this.measureUserOperation();
                });
                
                document.getElementById('complex-operation-btn').addEventListener('click', () => {
                    this.measureComplexOperation();
                });
            }
            
            measureUserOperation() {
                // Отметка начала операции
                performance.mark('user-operation-start');
                
                // Выполнение операции
                this.performUserOperation();
                
                // Отметка окончания операции
                performance.mark('user-operation-end');
                
                // Создание меры для операции
                performance.measure(
                    'user-operation-duration',
                    'user-operation-start',
                    'user-operation-end'
                );
                
                // Анализ пользовательских метрик
                this.analyzeUserTiming();
            }
            
            performUserOperation() {
                // Симуляция пользовательской операции
                const container = document.getElementById('user-timing');
                container.innerHTML = '<p>Выполняется пользовательская операция...</p>';
                
                // Сложные вычисления
                for (let i = 0; i < 1000000; i++) {
                    Math.sqrt(i);
                }
                
                container.innerHTML = '<p>Пользовательская операция завершена</p>';
            }
            
            measureComplexOperation() {
                performance.mark('complex-operation-start');
                
                // Сложная операция с несколькими этапами
                this.step1();
                this.step2();
                this.step3();
                
                performance.mark('complex-operation-end');
                
                // Создание мер для каждого этапа
                performance.measure('complex-operation-total', 'complex-operation-start', 'complex-operation-end');
                performance.measure('step1-duration', 'step1-start', 'step1-end');
                performance.measure('step2-duration', 'step2-start', 'step2-end');
                performance.measure('step3-duration', 'step3-start', 'step3-end');
                
                this.analyzeUserTiming();
            }
            
            step1() {
                performance.mark('step1-start');
                
                // Выполнение первого этапа
                const data = Array.from({ length: 10000 }, (_, i) => i);
                const processed = data.map(item => item * 2);
                
                performance.mark('step1-end');
            }
            
            step2() {
                performance.mark('step2-start');
                
                // Выполнение второго этапа
                const container = document.createElement('div');
                for (let i = 0; i < 1000; i++) {
                    const element = document.createElement('p');
                    element.textContent = `Элемент ${i}`;
                    container.appendChild(element);
                }
                
                performance.mark('step2-end');
            }
            
            step3() {
                performance.mark('step3-start');
                
                // Выполнение третьего этапа
                const promises = Array.from({ length: 100 }, () => {
                    return new Promise(resolve => setTimeout(resolve, 1));
                });
                
                Promise.all(promises);
                
                performance.mark('step3-end');
            }
            
            analyzeUserTiming() {
                const measures = performance.getEntriesByType('measure');
                const marks = performance.getEntriesByType('mark');
                
                const container = document.getElementById('user-timing');
                
                container.innerHTML += `
                    <div class="timing-analysis">
                        <h3>Пользовательские метрики:</h3>
                        <div class="measures-section">
                            <h4>Меры:</h4>
                            <ul>
                                ${measures.map(measure => `
                                    <li>
                                        <strong>${measure.name}:</strong> 
                                        ${measure.duration.toFixed(2)}ms
                                        (start: ${measure.startTime.toFixed(2)}ms, 
                                        end: ${(measure.startTime + measure.duration).toFixed(2)}ms)
                                    </li>
                                `).join('')}
                            </ul>
                        </div>
                        
                        <div class="marks-section">
                            <h4>Отметки:</h4>
                            <ul>
                                ${marks.map(mark => `
                                    <li>
                                        <strong>${mark.name}:</strong> 
                                        ${mark.startTime.toFixed(2)}ms
                                    </li>
                                `).join('')}
                            </ul>
                        </div>
                    </div>
                `;
                
                this.reportUserTiming(measures, marks);
            }
            
            reportUserTiming(measures, marks) {
                const report = {
                    page: window.location.href,
                    timestamp: new Date().toISOString(),
                    measures: measures.map(m => ({
                        name: m.name,
                        duration: m.duration,
                        startTime: m.startTime
                    })),
                    marks: marks.map(m => ({
                        name: m.name,
                        startTime: m.startTime
                    }))
                };
                
                // Отправка отчета о пользовательских метриках
                if (navigator.sendBeacon) {
                    navigator.sendBeacon('/api/user-timing', JSON.stringify(report));
                } else {
                    fetch('/api/user-timing', {
                        method: 'POST',
                        body: JSON.stringify(report),
                        headers: { 'Content-Type': 'application/json' }
                    }).catch(console.error);
                }
            }
            
            // Методы для отслеживания производительности компонентов
            startComponentTiming(componentName) {
                performance.mark(`${componentName}-start`);
            }
            
            endComponentTiming(componentName) {
                performance.mark(`${componentName}-end`);
                performance.measure(
                    `${componentName}-duration`,
                    `${componentName}-start`,
                    `${componentName}-end`
                );
            }
            
            getComponentTiming(componentName) {
                const measures = performance.getEntriesByName(`${componentName}-duration`);
                return measures.length > 0 ? measures[0].duration : null;
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new UserTimingAnalyzer();
        });
    </script>
    
    <style>
        .timing-analysis {
            margin-top: 20px;
            padding: 20px;
            background-color: #f8f9fa;
            border-radius: 8px;
        }
        
        .measures-section, .marks-section {
            margin-bottom: 20px;
        }
        
        .measures-section h4, .marks-section h4 {
            margin-top: 0;
        }
        
        ul {
            padding-left: 20px;
        }
        
        li {
            margin-bottom: 5px;
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
        
        button:hover {
            background-color: #005a9e;
        }
    </style>
</body>
</html>
```

## Мониторинг производительности в реальном времени

### Performance Observer API

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Performance Observer API</title>
</head>
<body>
    <h1>Performance Observer API</h1>
    
    <div id="observer-results"></div>
    
    <button id="trigger-heavy-task">Выполнить тяжелую задачу</button>
    <button id="simulate-layout-shift">Симулировать сдвиг макета</button>
    <button id="generate-long-task">Сгенерировать long task</button>

    <script>
        class PerformanceObserverAnalyzer {
            constructor() {
                this.setupPerformanceObservers();
                this.setupTestButtons();
            }
            
            setupPerformanceObservers() {
                // Наблюдение за long tasks
                new PerformanceObserver((list) => {
                    list.getEntries().forEach(entry => {
                        console.warn('Long task detected:', {
                            duration: entry.duration,
                            startTime: entry.startTime,
                            name: entry.name
                        });
                        
                        this.reportLongTask(entry);
                    });
                }).observe({ entryTypes: ['longtask'] });
                
                // Наблюдение за layout shifts
                new PerformanceObserver((list) => {
                    let cls = 0;
                    const entries = list.getEntries();
                    
                    for (const entry of entries) {
                        if (!entry.hadRecentInput) {
                            cls += entry.value;
                        }
                    }
                    
                    if (cls > 0) {
                        console.log('Layout shift detected:', {
                            cls: cls,
                            entries: entries.length
                        });
                        
                        this.reportLayoutShift(entries, cls);
                    }
                }).observe({ entryTypes: ['layout-shift'] });
                
                // Наблюдение за paint metrics
                new PerformanceObserver((list) => {
                    const entries = list.getEntries();
                    
                    entries.forEach(entry => {
                        console.log(`${entry.name}: ${entry.startTime}ms`);
                        
                        if (entry.name === 'first-contentful-paint') {
                            this.reportFCP(entry.startTime);
                        }
                    });
                }).observe({ entryTypes: ['paint'] });
                
                // Наблюдение за navigation metrics
                new PerformanceObserver((list) => {
                    const entries = list.getEntries();
                    
                    entries.forEach(entry => {
                        console.log('Navigation:', {
                            loadTime: entry.loadEventEnd - entry.navigationStart,
                            domContentLoaded: entry.domContentLoadedEventEnd - entry.navigationStart
                        });
                        
                        this.reportNavigation(entry);
                    });
                }).observe({ entryTypes: ['navigation'] });
                
                // Наблюдение за resource metrics
                new PerformanceObserver((list) => {
                    const entries = list.getEntries();
                    
                    entries.forEach(entry => {
                        console.log('Resource loaded:', {
                            name: entry.name,
                            duration: entry.duration,
                            size: entry.transferSize
                        });
                        
                        this.reportResource(entry);
                    });
                }).observe({ entryTypes: ['resource'] });
            }
            
            setupTestButtons() {
                document.getElementById('trigger-heavy-task').addEventListener('click', () => {
                    this.executeHeavyTask();
                });
                
                document.getElementById('simulate-layout-shift').addEventListener('click', () => {
                    this.simulateLayoutShift();
                });
                
                document.getElementById('generate-long-task').addEventListener('click', () => {
                    this.generateLongTask();
                });
            }
            
            executeHeavyTask() {
                performance.mark('heavy-task-start');
                
                // Тяжелая задача
                const result = [];
                for (let i = 0; i < 1000000; i++) {
                    result.push(Math.sqrt(i) * Math.sin(i));
                }
                
                performance.mark('heavy-task-end');
                performance.measure('heavy-task', 'heavy-task-start', 'heavy-task-end');
                
                console.log('Тяжелая задача выполнена');
            }
            
            simulateLayoutShift() {
                const container = document.getElementById('observer-results');
                
                // Сначала показываем небольшой элемент
                container.innerHTML = '<p>Маленький элемент</p>';
                
                // Затем добавляем большой элемент, вызывая сдвиг
                setTimeout(() => {
                    container.innerHTML = `
                        <p>Маленький элемент</p>
                        <div style="height: 300px; background-color: #e3f2fd;">Большой элемент, вызывающий сдвиг</div>
                    `;
                }, 1000);
            }
            
            generateLongTask() {
                // Генерация long task (> 50ms)
                const start = performance.now();
                
                while (performance.now() - start < 60) {
                    // Тяжелая синхронная операция
                    for (let i = 0; i < 10000; i++) {
                        Math.sqrt(i);
                    }
                }
                
                console.log('Long task generated');
            }
            
            reportLongTask(entry) {
                const report = {
                    type: 'long-task',
                    duration: entry.duration,
                    startTime: entry.startTime,
                    containerType: entry.containerType,
                    url: window.location.href,
                    timestamp: new Date().toISOString()
                };
                
                this.sendPerformanceReport(report);
            }
            
            reportLayoutShift(entries, cls) {
                const report = {
                    type: 'layout-shift',
                    cls: cls,
                    entries: entries.map(e => ({
                        value: e.value,
                        hadRecentInput: e.hadRecentInput,
                        lastInputTime: e.lastInputTime
                    })),
                    url: window.location.href,
                    timestamp: new Date().toISOString()
                };
                
                this.sendPerformanceReport(report);
            }
            
            reportFCP(startTime) {
                const report = {
                    type: 'first-contentful-paint',
                    startTime: startTime,
                    url: window.location.href,
                    timestamp: new Date().toISOString()
                };
                
                this.sendPerformanceReport(report);
            }
            
            reportNavigation(entry) {
                const report = {
                    type: 'navigation',
                    metrics: {
                        loadTime: entry.loadEventEnd - entry.navigationStart,
                        domContentLoaded: entry.domContentLoadedEventEnd - entry.navigationStart,
                        firstByte: entry.responseStart - entry.navigationStart
                    },
                    url: window.location.href,
                    timestamp: new Date().toISOString()
                };
                
                this.sendPerformanceReport(report);
            }
            
            reportResource(entry) {
                const report = {
                    type: 'resource',
                    name: entry.name,
                    duration: entry.duration,
                    size: entry.transferSize,
                    url: window.location.href,
                    timestamp: new Date().toISOString()
                };
                
                this.sendPerformanceReport(report);
            }
            
            sendPerformanceReport(report) {
                // Отправка отчета в систему мониторинга
                if ('sendBeacon' in navigator) {
                    navigator.sendBeacon('/api/performance-report', JSON.stringify(report));
                } else {
                    fetch('/api/performance-report', {
                        method: 'POST',
                        body: JSON.stringify(report),
                        headers: { 'Content-Type': 'application/json' }
                    }).catch(console.error);
                }
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new PerformanceObserverAnalyzer();
        });
    </script>
    
    <style>
        #observer-results {
            margin: 20px 0;
            padding: 20px;
            background-color: #f8f9fa;
            border-radius: 8px;
            min-height: 100px;
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
        
        button:hover {
            background-color: #005a9e;
        }
    </style>
</body>
</html>
```

## Инструменты мониторинга производительности

### Web Vitals Monitor

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Web Vitals Monitor</title>
    <script src="https://unpkg.com/web-vitals@3/dist/web-vitals.iife.js"></script>
</head>
<body>
    <h1>Web Vitals Monitor</h1>
    
    <div id="vitals-monitor">
        <div class="metric" id="lcp-metric">
            <h3>LCP: <span class="value">-</span>ms</h3>
            <div class="status">-</div>
        </div>
        
        <div class="metric" id="fid-metric">
            <h3>FID: <span class="value">-</span>ms</h3>
            <div class="status">-</div>
        </div>
        
        <div class="metric" id="cls-metric">
            <h3>CLS: <span class="value">-</span></h3>
            <div class="status">-</div>
        </div>
        
        <div class="metric" id="fcp-metric">
            <h3>FCP: <span class="value">-</span>ms</h3>
            <div class="status">-</div>
        </div>
        
        <div class="metric" id="tbt-metric">
            <h3>TBT: <span class="value">-</span>ms</h3>
            <div class="status">-</div>
        </div>
        
        <div class="metric" id="inp-metric">
            <h3>INP: <span class="value">-</span>ms</h3>
            <div class="status">-</div>
        </div>
    </div>
    
    <button id="heavy-task-btn">Выполнить тяжелую задачу</button>
    <button id="layout-shift-btn">Вызвать сдвиг макета</button>

    <script>
        class WebVitalsMonitor {
            constructor() {
                this.metrics = {
                    lcp: null,
                    fid: null,
                    cls: 0,
                    fcp: null,
                    tbt: 0,
                    inp: null
                };
                
                this.thresholds = {
                    lcp: 2500,  // 2.5s
                    fid: 100,   // 100ms
                    cls: 0.1,   // 0.1
                    fcp: 1800,  // 1.8s
                    tbt: 300,   // 300ms
                    inp: 200    // 200ms
                };
                
                this.setupVitalsMonitoring();
                this.setupTestButtons();
            }
            
            setupVitalsMonitoring() {
                // LCP monitoring
                webVitals.getLCP(metric => {
                    this.metrics.lcp = metric.value;
                    this.updateMetric('lcp', metric.value, 'LCP');
                    this.checkThreshold('lcp', metric.value);
                    this.sendVitalsReport('LCP', metric);
                });
                
                // FID monitoring
                webVitals.getFID(metric => {
                    this.metrics.fid = metric.value;
                    this.updateMetric('fid', metric.value, 'FID');
                    this.checkThreshold('fid', metric.value);
                    this.sendVitalsReport('FID', metric);
                });
                
                // CLS monitoring
                webVitals.getCLS(metric => {
                    this.metrics.cls += metric.value;
                    this.updateMetric('cls', this.metrics.cls, 'CLS');
                    this.checkThreshold('cls', this.metrics.cls);
                    this.sendVitalsReport('CLS', metric);
                });
                
                // FCP monitoring
                webVitals.getFCP(metric => {
                    this.metrics.fcp = metric.value;
                    this.updateMetric('fcp', metric.value, 'FCP');
                    this.checkThreshold('fcp', metric.value);
                    this.sendVitalsReport('FCP', metric);
                });
                
                // TTFB monitoring
                webVitals.getTTFB(metric => {
                    this.updateMetric('ttfb', metric.value, 'TTFB');
                    this.sendVitalsReport('TTFB', metric);
                });
            }
            
            updateMetric(metricName, value, displayName) {
                const element = document.getElementById(`${metricName}-metric`);
                if (element) {
                    const valueElement = element.querySelector('.value');
                    const statusElement = element.querySelector('.status');
                    
                    if (valueElement) {
                        valueElement.textContent = typeof value === 'number' ? value.toFixed(2) : value;
                    }
                    
                    if (statusElement) {
                        statusElement.textContent = this.getMetricStatus(metricName, value);
                    }
                }
            }
            
            getMetricStatus(metricName, value) {
                if (value === null || value === undefined) return 'ожидание';
                
                const threshold = this.thresholds[metricName];
                if (threshold === undefined) return 'ок';
                
                return value <= threshold ? 'хорошо' : 'плохо';
            }
            
            checkThreshold(metricName, value) {
                if (value > this.thresholds[metricName]) {
                    console.warn(`${metricName.toUpperCase()} превышает порог: ${value} > ${this.thresholds[metricName]}`);
                    
                    this.reportThresholdExceeded(metricName, value);
                }
            }
            
            reportThresholdExceeded(metricName, value) {
                const report = {
                    type: 'threshold-exceeded',
                    metric: metricName.toUpperCase(),
                    value: value,
                    threshold: this.thresholds[metricName],
                    url: window.location.href,
                    timestamp: new Date().toISOString()
                };
                
                this.sendPerformanceReport(report);
            }
            
            sendVitalsReport(metricName, metric) {
                const report = {
                    type: 'web-vitals',
                    metric: metricName,
                    value: metric.value,
                    id: metric.id,
                    name: metric.name,
                    url: window.location.href,
                    timestamp: new Date().toISOString()
                };
                
                this.sendPerformanceReport(report);
            }
            
            sendPerformanceReport(report) {
                // В реальном приложении отправка в систему аналитики
                console.log('Performance report:', report);
                
                if (navigator.sendBeacon) {
                    navigator.sendBeacon('/api/vitals', JSON.stringify(report));
                }
            }
            
            setupTestButtons() {
                document.getElementById('heavy-task-btn').addEventListener('click', () => {
                    this.executeHeavyTask();
                });
                
                document.getElementById('layout-shift-btn').addEventListener('click', () => {
                    this.simulateLayoutShift();
                });
            }
            
            executeHeavyTask() {
                // Симуляция тяжелой задачи для ухудшения производительности
                const start = performance.now();
                
                while (performance.now() - start < 100) {
                    // Тяжелая синхронная операция
                    for (let i = 0; i < 50000; i++) {
                        Math.sqrt(i);
                    }
                }
                
                console.log('Heavy task completed');
            }
            
            simulateLayoutShift() {
                const container = document.getElementById('vitals-monitor');
                
                setTimeout(() => {
                    const newElement = document.createElement('div');
                    newElement.style.height = '200px';
                    newElement.style.backgroundColor = '#ffebee';
                    newElement.textContent = 'Этот элемент вызывает сдвиг макета';
                    
                    container.appendChild(newElement);
                }, 1000);
            }
        }
        
        // Инициализация
        document.addEventListener('DOMContentLoaded', () => {
            new WebVitalsMonitor();
        });
    </script>
    
    <style>
        #vitals-monitor {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }
        
        .metric {
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            background-color: white;
        }
        
        .metric h3 {
            margin-top: 0;
            color: #333;
        }
        
        .value {
            font-weight: bold;
            color: #007acc;
        }
        
        .status.good {
            color: #2e7d32;
        }
        
        .status.bad {
            color: #d32f2f;
        }
        
        .status.warning {
            color: #ff8f00;
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
        
        button:hover {
            background-color: #005a9e;
        }
    </style>
</body>
</html>
```

## Практические примеры мониторинга

### Система мониторинга производительности

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Система мониторинга производительности</title>
</head>
<body>
    <h1>Система мониторинга производительности</h1>
    
    <div id="app-monitoring">
        <div class="monitoring-controls">
            <button id="start-monitoring">Начать мониторинг</button>
            <button id="stop-monitoring">Остановить мониторинг</button>
            <button id="export-data">Экспортировать данные</button>
            <button id="clear-data">Очистить данные</button>
        </div>
        
        <div class="performance-dashboard">
            <div class="metric-card" id="lcp-card">
                <h3>Largest Contentful Paint</h3>
                <div class="metric-value" id="lcp-value">-</div>
                <div class="metric-status" id="lcp-status">-</div>
            </div>
            
            <div class="metric-card" id="cls-card">
                <h3>Cumulative Layout Shift</h3>
                <div class="metric-value" id="cls-value">-</div>
                <div class="metric-status" id="cls-status">-</div>
            </div>
            
            <div class="metric-card" id="fid-card">
                <h3>First Input Delay</h3>
                <div class="metric-value" id="fid-value">-</div>
                <div class="metric-status" id="fid-status">-</div>
            </div>
        </div>
        
        <div class="performance-graphs">
            <h3>Графики производительности</h3>
            <canvas id="performance-chart" width="800" height="400"></canvas>
        </div>
        
        <div class="performance-logs">
            <h3>Логи производительности</h3>
            <div id="performance-logs-container"></div>
        </div>
    </div>

    <script>
        class PerformanceMonitoringSystem {
            constructor() {
                this.isMonitoring = false;
                this.performanceData = [];
                this.metrics = {
                    lcp: null,
                    cls: 0,
                    fid: null,
                    fcp: null,
                    ttfb: null
                };
                
                this.setupMonitoring();
                this.setupControls();
            }
            
            setupMonitoring() {
                // Установка наблюдателей производительности
                this.setupLCPObserver();
                this.setupCLSObserver();
                this.setupFIDObserver();
                this.setupFCPObserver();
            }
            
            setupLCPObserver() {
                new PerformanceObserver((list) => {
                    const entries = list.getEntries();
                    const lastEntry = entries[entries.length - 1];
                    
                    this.metrics.lcp = lastEntry.startTime;
                    this.updateMetric('lcp', lastEntry.startTime);
                    this.logPerformanceData('LCP', lastEntry);
                }).observe({ entryTypes: ['largest-contentful-paint'] });
            }
            
            setupCLSObserver() {
                new PerformanceObserver((list) => {
                    const entries = list.getEntries();
                    
                    for (const entry of entries) {
                        if (!entry.hadRecentInput) {
                            this.metrics.cls += entry.value;
                        }
                    }
                    
                    this.updateMetric('cls', this.metrics.cls);
                    this.logPerformanceData('CLS', { value: this.metrics.cls });
                }).observe({ entryTypes: ['layout-shift'] });
            }
            
            setupFIDObserver() {
                new PerformanceObserver((list) => {
                    const entries = list.getEntries();
                    
                    entries.forEach(entry => {
                        this.metrics.fid = entry.processingStart - entry.startTime;
                        this.updateMetric('fid', this.metrics.fid);
                        this.logPerformanceData('FID', entry);
                    });
                }).observe({ entryTypes: ['first-input'] });
            }
            
            setupFCPObserver() {
                new PerformanceObserver((list) => {
                    const entries = list.getEntries();
                    const fcp = entries[entries.length - 1];
                    
                    this.metrics.fcp = fcp.startTime;
                    this.updateMetric('fcp', fcp.startTime);
                    this.logPerformanceData('FCP', fcp);
                }).observe({ entryTypes: ['paint'] });
            }
            
            updateMetric(metricName, value) {
                const valueElement = document.getElementById(`${metricName}-value`);
                const statusElement = document.getElementById(`${metricName}-status`);
                const cardElement = document.getElementById(`${metricName}-card`);
                
                if (valueElement) {
                    valueElement.textContent = typeof value === 'number' ? value.toFixed(2) : value;
                }
                
                if (statusElement) {
                    const status = this.evaluateMetricStatus(metricName, value);
                    statusElement.textContent = status;
                    statusElement.className = `metric-status ${status.toLowerCase()}`;
                }
                
                if (cardElement) {
                    cardElement.className = `metric-card ${this.getMetricColor(metricName, value)}`;
                }
            }
            
            evaluateMetricStatus(metricName, value) {
                if (value === null || value === undefined) return 'ожидание';
                
                const thresholds = {
                    lcp: { good: 2500, poor: 4000 },
                    cls: { good: 0.1, poor: 0.25 },
                    fid: { good: 100, poor: 300 }
                };
                
                if (!thresholds[metricName]) return 'ок';
                
                const { good, poor } = thresholds[metricName];
                
                if (value <= good) return 'хорошо';
                if (value <= poor) return 'удовлетворительно';
                return 'плохо';
            }
            
            getMetricColor(metricName, value) {
                const status = this.evaluateMetricStatus(metricName, value);
                
                const colors = {
                    'хорошо': 'good',
                    'удовлетворительно': 'average',
                    'плохо': 'poor'
                };
                
                return colors[status] || 'neutral';
            }
            
            logPerformanceData(type, data) {
                if (!this.isMonitoring) return;
                
                const logEntry = {
                    type: type,
                    data: data,
                    timestamp: new Date().toISOString(),
                    url: window.location.href
                };
                
                this.performanceData.push(logEntry);
                
                // Обновление логов на странице
                this.updateLogsDisplay();
                
                // Отправка данных в систему аналитики
                this.sendPerformanceData(logEntry);
            }
            
            updateLogsDisplay() {
                const logsContainer = document.getElementById('performance-logs-container');
                
                logsContainer.innerHTML = this.performanceData.slice(-20).map(entry => `
                    <div class="log-entry">
                        <div class="log-header">
                            <span class="log-type">${entry.type}</span>
                            <span class="log-time">${new Date(entry.timestamp).toLocaleTimeString()}</span>
                        </div>
                        <div class="log-data">
                            <pre>${JSON.stringify(entry.data, null, 2)}</pre>
                        </div>
                    </div>
                `).join('');
                
                // Прокрутка к последней записи
                logsContainer.scrollTop = logsContainer.scrollHeight;
            }
            
            sendPerformanceData(data) {
                // Отправка данных в систему мониторинга
                if (navigator.sendBeacon) {
                    navigator.sendBeacon('/api/performance-data', JSON.stringify(data));
                } else {
                    fetch('/api/performance-data', {
                        method: 'POST',
                        body: JSON.stringify(data),
                        headers: { 'Content-Type': 'application/json' }
                    }).catch(console.error);
                }
            }
            
            setupControls() {
                document.getElementById('start-monitoring').addEventListener('click', () => {
                    this.startMonitoring();
                });
                
                document.getElementById('stop-monitoring').addEventListener('click', () => {
                    this.stopMonitoring();
                });
                
                document.getElementById('export-data').addEventListener('click', () => {
                    this.exportData();
                });
                
                document.getElementById('clear-data').addEventListener('click', () => {
                    this.clearData();
                });
            }
            
            startMonitoring() {
                this.isMonitoring = true;
                document.getElementById('start-monitoring').disabled = true;
                document.getElementById('stop-monitoring').disabled = false;
                console.log('Мониторинг производительности запущен');
            }
            
            stopMonitoring() {
                this.isMonitoring = false;
                document.getElementById('start-monitoring').disabled = false;
                document.getElementById('stop-monitoring').disabled = true;
                console.log('Мониторинг производительности остановлен');
            }
            
            exportData() {
                const dataStr = JSON.stringify(this.performanceData, null, 2);
                const dataUri = 'data:application/json;charset=utf-8,'+ encodeURIComponent(dataStr);
                
                const exportFile = document.createElement('a');
                exportFile.setAttribute('href', dataUri);
                exportFile.setAttribute('download', 'performance-data.json');
                exportFile.click();
            }
            
            clearData() {
                this.performanceData = [];
                document.getElementById('performance-logs-container').innerHTML = '';
                console.log('Данные мониторинга очищены');
            }
            
            // Методы для анализа производительности
            getPerformanceSummary() {
                return {
                    totalMeasurements: this.performanceData.length,
                    metrics: this.metrics,
                    averageLCP: this.calculateAverage('lcp'),
                    averageCLS: this.calculateAverage('cls'),
                    averageFID: this.calculateAverage('fid')
                };
            }
            
            calculateAverage(metricName) {
                const values = this.performanceData
                    .filter(entry => entry.type === metricName.toUpperCase())
                    .map(entry => entry.data.value || entry.data.startTime);
                
                if (values.length === 0) return null;
                
                const sum = values.reduce((acc, val) => acc + val, 0);
                return sum / values.length;
            }
            
            generatePerformanceReport() {
                const summary = this.getPerformanceSummary();
                
                const report = {
                    page: window.location.href,
                    timestamp: new Date().toISOString(),
                    summary: summary,
                    recommendations: this.getPerformanceRecommendations(summary)
                };
                
                return report;
            }
            
            getPerformanceRecommendations(summary) {
                const recommendations = [];
                
                if (summary.averageLCP > 2500) {
                    recommendations.push('Оптимизируйте время до Largest Contentful Paint');
                }
                
                if (summary.averageCLS > 0.1) {
                    recommendations.push('Уменьшите Cumulative Layout Shift');
                }
                
                if (summary.averageFID > 100) {
                    recommendations.push('Улучшите First Input Delay');
                }
                
                return recommendations;
            }
        }
        
        // Инициализация системы мониторинга
        const perfMonitor = new PerformanceMonitoringSystem();
        
        // Добавление глобальной функции для получения отчета
        window.getPerformanceReport = () => {
            return perfMonitor.generatePerformanceReport();
        };
    </script>
    
    <style>
        #app-monitoring {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        .monitoring-controls {
            display: flex;
            gap: 10px;
            margin-bottom: 30px;
            flex-wrap: wrap;
        }
        
        .monitoring-controls button {
            padding: 10px 20px;
            background-color: #007acc;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        
        .monitoring-controls button:hover {
            background-color: #005a9e;
        }
        
        .monitoring-controls button:disabled {
            background-color: #ccc;
            cursor: not-allowed;
        }
        
        .performance-dashboard {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .metric-card {
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            text-align: center;
        }
        
        .metric-card h3 {
            margin-top: 0;
            margin-bottom: 15px;
        }
        
        .metric-value {
            font-size: 2em;
            font-weight: bold;
            margin-bottom: 10px;
        }
        
        .metric-status {
            font-weight: bold;
            padding: 5px 10px;
            border-radius: 20px;
            display: inline-block;
        }
        
        .metric-card.good {
            background-color: #e8f5e8;
            border: 2px solid #4caf50;
        }
        
        .metric-card.average {
            background-color: #fff3cd;
            border: 2px solid #ffc107;
        }
        
        .metric-card.poor {
            background-color: #ffebee;
            border: 2px solid #f44336;
        }
        
        .metric-status.good {
            background-color: #4caf50;
            color: white;
        }
        
        .metric-status.average {
            background-color: #ffc107;
            color: #333;
        }
        
        .metric-status.poor {
            background-color: #f44336;
            color: white;
        }
        
        .performance-graphs {
            margin-bottom: 30px;
        }
        
        #performance-chart {
            width: 100%;
            border: 1px solid #ddd;
            border-radius: 8px;
        }
        
        .performance-logs {
            max-height: 400px;
            overflow-y: auto;
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 15px;
        }
        
        .log-entry {
            margin-bottom: 15px;
            padding: 10px;
            border-left: 3px solid #007acc;
            background-color: #f9f9f9;
        }
        
        .log-header {
            display: flex;
            justify-content: space-between;
            margin-bottom: 5px;
            font-weight: bold;
        }
        
        .log-type {
            color: #007acc;
        }
        
        .log-time {
            color: #666;
            font-size: 0.8em;
        }
        
        .log-data pre {
            margin: 0;
            font-size: 0.9em;
            color: #333;
        }
    </style>
</body>
</html>
```

## Лучшие практики мониторинга производительности

### 1. Регулярный мониторинг
```javascript
// Периодическая проверка производительности
class ContinuousPerformanceMonitor {
    constructor() {
        this.checkInterval = null;
        this.monitoringEnabled = false;
    }
    
    startContinuousMonitoring() {
        if (this.monitoringEnabled) return;
        
        this.monitoringEnabled = true;
        
        // Проверка каждые 5 минут
        this.checkInterval = setInterval(() => {
            this.performHealthCheck();
        }, 5 * 60 * 1000);
        
        // Проверка при активации вкладки
        document.addEventListener('visibilitychange', () => {
            if (document.visibilityState === 'visible') {
                this.performHealthCheck();
            }
        });
    }
    
    performHealthCheck() {
        const healthData = {
            memory: this.getMemoryUsage(),
            cpu: this.getCPUUsage(),
            network: this.getNetworkConditions(),
            battery: this.getBatteryStatus(),
            timestamp: new Date().toISOString()
        };
        
        this.reportHealthData(healthData);
    }
    
    stopContinuousMonitoring() {
        if (this.checkInterval) {
            clearInterval(this.checkInterval);
            this.checkInterval = null;
        }
        this.monitoringEnabled = false;
    }
    
    getMemoryUsage() {
        if ('memory' in performance) {
            return {
                used: performance.memory.usedJSHeapSize,
                total: performance.memory.totalJSHeapSize,
                limit: performance.memory.jsHeapSizeLimit
            };
        }
        return null;
    }
    
    async getCPUUsage() {
        // Использование Performance API для оценки загрузки CPU
        const start = performance.now();
        let iterations = 0;
        
        while (performance.now() - start < 100) { // 100ms тест
            iterations++;
            // Тяжелая операция для нагрузки
            Math.sqrt(iterations);
        }
        
        return {
            iterations: iterations,
            duration: performance.now() - start
        };
    }
    
    getNetworkConditions() {
        if ('connection' in navigator) {
            return {
                effectiveType: navigator.connection.effectiveType,
                downlink: navigator.connection.downlink,
                rtt: navigator.connection.rtt,
                saveData: navigator.connection.saveData
            };
        }
        return null;
    }
    
    async getBatteryStatus() {
        if ('getBattery' in navigator) {
            try {
                const battery = await navigator.getBattery();
                return {
                    level: battery.level,
                    charging: battery.charging,
                    chargingTime: battery.chargingTime,
                    dischargingTime: battery.dischargingTime
                };
            } catch (error) {
                return null;
            }
        }
        return null;
    }
    
    reportHealthData(data) {
        // Отправка данных о здоровье системы
        fetch('/api/system-health', {
            method: 'POST',
            body: JSON.stringify(data),
            headers: { 'Content-Type': 'application/json' }
        }).catch(console.error);
    }
}
```

### 2. Мониторинг пользовательского опыта
```javascript
// Мониторинг пользовательского опыта
class UserExperienceMonitor {
    constructor() {
        this.interactions = [];
        this.setupUXMonitoring();
    }
    
    setupUXMonitoring() {
        // Мониторинг взаимодействий
        document.addEventListener('click', (e) => {
            this.logInteraction('click', e);
        });
        
        document.addEventListener('keypress', (e) => {
            this.logInteraction('keypress', e);
        });
        
        document.addEventListener('scroll', this.debounce(() => {
            this.logInteraction('scroll', { 
                scrollY: window.scrollY, 
                scrollX: window.scrollX 
            });
        }, 250));
        
        // Мониторинг загрузки ресурсов
        window.addEventListener('load', () => {
            this.logInteraction('page_load', { 
                loadTime: performance.timing.loadEventEnd - performance.timing.navigationStart 
            });
        });
    }
    
    logInteraction(type, event) {
        const interaction = {
            type: type,
            timestamp: new Date().toISOString(),
            element: event.target ? event.target.tagName : 'window',
            coordinates: event.clientX ? { x: event.clientX, y: event.clientY } : null,
            url: window.location.href
        };
        
        this.interactions.push(interaction);
        
        // Отправка данных при накоплении
        if (this.interactions.length >= 10) {
            this.reportInteractions();
        }
    }
    
    reportInteractions() {
        const data = {
            page: window.location.href,
            interactions: this.interactions.slice(-50), // Последние 50 взаимодействий
            timestamp: new Date().toISOString()
        };
        
        if (navigator.sendBeacon) {
            navigator.sendBeacon('/api/ux-data', JSON.stringify(data));
        } else {
            fetch('/api/ux-data', {
                method: 'POST',
                body: JSON.stringify(data),
                headers: { 'Content-Type': 'application/json' }
            }).catch(console.error);
        }
        
        this.interactions = [];
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
}
```

## Заключение

Мониторинг производительности HTML-страниц - это непрерывный процесс, который включает в себя измерение ключевых метрик, выявление узких мест и оптимизацию пользовательского опыта. Современные инструменты и API позволяют разработчикам получать подробную информацию о производительности и принимать обоснованные решения по улучшению веб-приложений.

Ключевые аспекты мониторинга производительности:
1. Использование современных API (Performance, Navigation Timing, Resource Timing)
2. Регулярный сбор и анализ метрик
3. Мониторинг Core Web Vitals
4. Отслеживание пользовательских взаимодействий
5. Выявление и устранение узких мест
6. Интеграция с системами аналитики
7. Автоматизированное тестирование производительности
8. Следование современным веб-стандартам

Эти практики позволяют создавать быстрые, отзывчивые и эффективные веб-приложения, которые обеспечивают отличный пользовательский опыт и высокие показатели производительности.

Современные подходы к мониторингу производительности включают:
- Использование Performance Observer API для непрерывного мониторинга
- Интеграцию с системами аналитики и мониторинга
- Автоматизированное тестирование производительности
- Мониторинг в реальном времени
- Анализ пользовательского опыта
- Использование Web Vitals для оценки качества
- Мониторинг ресурсов и их загрузки
- Отслеживание узких мест и проблем производительности

Эти методы помогают поддерживать высокую производительность веб-приложений и обеспечивать отличный пользовательский опыт.

## Следующие темы
- [[HTML/Безопасность]]
- [[HTML/Доступность]]
- [[HTML/Совместимость]]

## Теги
#html #performance #monitoring #web-development #frontend #core-web-vitals #csp #sri #security #xss #csrf #clickjacking #accessibility #user-experience #metrics #timing #navigation-timing #resource-timing #user-timing #web-vitals #performance-api #best-practices
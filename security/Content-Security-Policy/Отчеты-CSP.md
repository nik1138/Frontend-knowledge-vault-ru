---
aliases: ["CSP отчеты", "Отчеты о нарушениях CSP", "CSP reporting"]
tags: [security, csp, content-security-policy, web-security, reporting]
---

# Отчеты Content Security Policy

## Введение

Отчеты Content Security Policy (CSP) — это механизм, позволяющий веб-приложениям получать информацию о нарушениях политики безопасности. Эти отчеты помогают разработчикам понимать, какие элементы нарушают CSP, и корректировать политику без нарушения функциональности приложения.

## Структура CSP отчета

Когда происходит нарушение CSP, браузер отправляет JSON-объект с информацией о нарушении на указанный URL. Структура отчета включает:

```json
{
  "csp-report": {
    "document-uri": "https://example.com/page.html",
    "referrer": "https://example.com/",
    "blocked-uri": "https://evil.com/evil.js",
    "violated-directive": "script-src 'self'",
    "effective-directive": "script-src",
    "original-policy": "default-src 'self'; script-src 'self'; report-uri /csp-report",
    "disposition": "enforce",
    "status-code": 200,
    "script-sample": "console.log('evil code')",
    "source-file": "https://evil.com/evil.js",
    "line-number": 10,
    "column-number": 5
  }
}
```

### Основные поля отчета

- `document-uri` — URL документа, в котором произошло нарушение
- `referrer` — реферер документа
- `blocked-uri` — URI ресурса, который был заблокирован
- `violated-directive` — директива, которая была нарушена
- `effective-directive` — фактическая директива, которая была применена
- `original-policy` — первоначальная CSP политика
- `disposition` — режим политики ('enforce' или 'report')
- `status-code` — HTTP статус код документа
- `script-sample` — фрагмент проблемного скрипта (только в некоторых браузерах)
- `source-file`, `line-number`, `column-number` — информация об источнике ошибки

## Настройка отчетности

### 1. Использование report-uri (устаревший метод)

```javascript
// В Express.js
app.use((req, res, next) => {
  res.setHeader('Content-Security-Policy', 
    "default-src 'self'; " +
    "script-src 'self'; " +
    "report-uri /csp-report"
  );
  next();
});
```

### 2. Использование report-to (современный метод)

```javascript
// Установка Reporting API endpoint
app.use((req, res, next) => {
  // Установка отчетного эндпоинта
  res.setHeader('Reporting-Endpoints', 
    'csp-endpoint="https://example.com/api/csp-reports"');
  
  // Использование report-to в CSP
  res.setHeader('Content-Security-Policy', 
    "default-src 'self'; " +
    "script-src 'self'; " +
    "report-to csp-endpoint"
  );
  next();
});
```

## Реализация обработчика отчетов

### 1. Базовый обработчик отчетов

```javascript
const express = require('express');
const app = express();

// Middleware для парсинга CSP отчетов
app.use('/csp-report', express.json({ 
  type: 'application/csp-report',
  limit: '1mb'
}));

// Альтернативный способ для разных типов контента
app.use('/csp-report', (req, res, next) => {
  if (req.is('application/json') || req.path.includes('csp-report')) {
    express.json({ limit: '1mb' })(req, res, next);
  } else {
    next();
  }
});

// Обработчик CSP отчетов
app.post('/csp-report', (req, res) => {
  const report = req.body['csp-report'] || {};
  
  console.log('CSP Violation Report:', {
    documentUri: report['document-uri'],
    violatedDirective: report['violated-directive'],
    blockedUri: report['blocked-uri'],
    sourceFile: report['source-file'],
    lineNumber: report['line-number'],
    columnNumber: report['column-number'],
    timestamp: new Date().toISOString()
  });
  
  // Возврат 204 No Content
  res.status(204).end();
});
```

### 2. Расширенный обработчик с логированием

```javascript
class CspReportHandler {
  constructor(options = {}) {
    this.storage = options.storage || new InMemoryStorage();
    this.maxReports = options.maxReports || 1000;
    this.logLevel = options.logLevel || 'info';
  }
  
  async handleReport(req, res) {
    try {
      const report = req.body['csp-report'] || {};
      
      // Валидация отчета
      if (!this.isValidReport(report)) {
        console.warn('Invalid CSP report received');
        return res.status(400).end();
      }
      
      // Обогащение отчета
      const enrichedReport = {
        ...report,
        id: this.generateId(),
        timestamp: new Date().toISOString(),
        userAgent: req.get('User-Agent'),
        ip: req.ip,
        severity: this.calculateSeverity(report)
      };
      
      // Сохранение отчета
      await this.storage.save(enrichedReport);
      
      // Логирование
      this.logReport(enrichedReport);
      
      // Отправка уведомления при высоком приоритете
      if (enrichedReport.severity === 'high') {
        await this.sendAlert(enrichedReport);
      }
      
      res.status(204).end();
    } catch (error) {
      console.error('Error handling CSP report:', error);
      res.status(500).end();
    }
  }
  
  isValidReport(report) {
    return report['document-uri'] && report['violated-directive'];
  }
  
  calculateSeverity(report) {
    const directive = report['violated-directive'];
    
    if (directive.includes('script-src') || directive.includes('object-src')) {
      return 'high';
    } else if (directive.includes('style-src') || directive.includes('img-src')) {
      return 'medium';
    }
    return 'low';
  }
  
  logReport(report) {
    const logLevel = this.logLevel;
    const message = `CSP Violation: ${report['violated-directive']} in ${report['document-uri']}`;
    
    switch (logLevel) {
      case 'debug':
        console.debug(message, report);
        break;
      case 'info':
        console.info(message);
        break;
      case 'warn':
        console.warn(message);
        break;
    }
  }
  
  generateId() {
    return Math.random().toString(36).substr(2, 9);
  }
  
  async sendAlert(report) {
    // Отправка уведомления (email, Slack, etc.)
    console.log('High severity CSP violation detected:', report);
  }
}

// Использование
const cspHandler = new CspReportHandler({
  logLevel: 'info'
});

app.post('/csp-report', (req, res) => cspHandler.handleReport(req, res));
```

## Хранение и анализ отчетов

### 1. Хранение в базе данных

```javascript
// Модель для хранения CSP отчетов
class CspReportModel {
  constructor(db) {
    this.db = db;
    this.collection = db.collection('csp_reports');
  }
  
  async saveReport(report) {
    return await this.collection.insertOne({
      ...report,
      createdAt: new Date(),
      processed: false
    });
  }
  
  async getViolationsByDirective(directive, days = 7) {
    const since = new Date();
    since.setDate(since.getDate() - days);
    
    return await this.collection
      .find({
        'violated-directive': { $regex: directive },
        createdAt: { $gte: since }
      })
      .sort({ createdAt: -1 })
      .limit(100)
      .toArray();
  }
  
  async getTopViolations(days = 7) {
    const since = new Date();
    since.setDate(since.getDate() - days);
    
    return await this.collection
      .aggregate([
        {
          $match: {
            createdAt: { $gte: since }
          }
        },
        {
          $group: {
            _id: '$violated-directive',
            count: { $sum: 1 },
            uniqueDocuments: { $addToSet: '$document-uri' }
          }
        },
        {
          $project: {
            directive: '$_id',
            count: 1,
            uniqueDocumentsCount: { $size: '$uniqueDocuments' }
          }
        },
        {
          $sort: { count: -1 }
        }
      ])
      .toArray();
  }
}
```

### 2. Агрегация и анализ данных

```javascript
// Сервис анализа CSP отчетов
class CspReportAnalyzer {
  constructor(reportModel) {
    this.model = reportModel;
  }
  
  async generateDailyReport(date = new Date()) {
    const start = new Date(date);
    start.setHours(0, 0, 0, 0);
    
    const end = new Date(date);
    end.setHours(23, 59, 59, 999);
    
    const reports = await this.model.collection
      .find({
        createdAt: { $gte: start, $lte: end }
      })
      .toArray();
    
    return {
      totalViolations: reports.length,
      byDirective: this.groupByDirective(reports),
      bySeverity: this.groupBySeverity(reports),
      byPage: this.groupByPage(reports),
      newPatterns: await this.detectNewPatterns(reports)
    };
  }
  
  groupByDirective(reports) {
    return reports.reduce((acc, report) => {
      const directive = report['violated-directive'];
      acc[directive] = (acc[directive] || 0) + 1;
      return acc;
    }, {});
  }
  
  groupBySeverity(reports) {
    return reports.reduce((acc, report) => {
      const severity = this.calculateSeverity(report);
      acc[severity] = (acc[severity] || 0) + 1;
      return acc;
    }, {});
  }
  
  groupByPage(reports) {
    return reports.reduce((acc, report) => {
      const page = report['document-uri'];
      acc[page] = (acc[page] || 0) + 1;
      return acc;
    }, {});
  }
  
  async detectNewPatterns(reports) {
    // Простой алгоритм обнаружения новых паттернов
    const recentReports = await this.model.collection
      .find({})
      .sort({ createdAt: -1 })
      .limit(1000)
      .toArray();
    
    const knownPatterns = new Set();
    recentReports.forEach(report => {
      knownPatterns.add(this.createPatternSignature(report));
    });
    
    const newViolations = reports.filter(report => {
      return !knownPatterns.has(this.createPatternSignature(report));
    });
    
    return newViolations.slice(0, 10); // Только первые 10 новых паттернов
  }
  
  createPatternSignature(report) {
    return `${report['violated-directive']}_${report['blocked-uri']}`;
  }
  
  calculateSeverity(report) {
    const directive = report['violated-directive'];
    
    if (directive.includes('script-src') || directive.includes('object-src')) {
      return 'high';
    } else if (directive.includes('style-src') || directive.includes('img-src')) {
      return 'medium';
    }
    return 'low';
  }
}
```

## Визуализация отчетов

### 1. API для получения статистики

```javascript
// API для получения статистики CSP
app.get('/api/csp-stats', async (req, res) => {
  try {
    const days = parseInt(req.query.days) || 7;
    const analyzer = new CspReportAnalyzer(cspReportModel);
    const report = await analyzer.generateDailyReport();
    
    res.json(report);
  } catch (error) {
    console.error('Error generating CSP stats:', error);
    res.status(500).json({ error: 'Failed to generate CSP stats' });
  }
});

// API для получения топ нарушений
app.get('/api/csp-violations-top', async (req, res) => {
  try {
    const days = parseInt(req.query.days) || 7;
    const topViolations = await cspReportModel.getTopViolations(days);
    
    res.json(topViolations);
  } catch (error) {
    console.error('Error fetching top violations:', error);
    res.status(500).json({ error: 'Failed to fetch top violations' });
  }
});
```

### 2. Пример дашборда

```html
<!DOCTYPE html>
<html>
<head>
    <title>CSP Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>CSP Violations Dashboard</h1>
    
    <div style="width: 800px; margin: 0 auto;">
        <canvas id="violationsChart"></canvas>
    </div>
    
    <div id="topViolations"></div>
    
    <script>
        // Загрузка данных и построение графиков
        async function loadCspData() {
            try {
                const [stats, topViolations] = await Promise.all([
                    fetch('/api/csp-stats').then(r => r.json()),
                    fetch('/api/csp-violations-top').then(r => r.json())
                ]);
                
                renderViolationsChart(stats.byDirective);
                renderTopViolations(topViolations);
            } catch (error) {
                console.error('Error loading CSP data:', error);
            }
        }
        
        function renderViolationsChart(data) {
            const ctx = document.getElementById('violationsChart').getContext('2d');
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: Object.keys(data),
                    datasets: [{
                        label: 'CSP Violations Count',
                        data: Object.values(data),
                        backgroundColor: 'rgba(255, 99, 132, 0.2)',
                        borderColor: 'rgba(255, 99, 132, 1)',
                        borderWidth: 1
                    }]
                },
                options: {
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            });
        }
        
        function renderTopViolations(violations) {
            const container = document.getElementById('topViolations');
            container.innerHTML = '<h2>Top Violations</h2><ul>' + 
                violations.map(v => 
                    `<li>${v.directive}: ${v.count} violations</li>`
                ).join('') + '</ul>';
        }
        
        loadCspData();
    </script>
</body>
</html>
```

## Управление отчетами

### 1. Очистка старых отчетов

```javascript
// Сервис для очистки старых отчетов
class CspReportCleanup {
  constructor(reportModel, retentionDays = 30) {
    this.model = reportModel;
    this.retentionDays = retentionDays;
  }
  
  async cleanup() {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - this.retentionDays);
    
    const result = await this.model.collection.deleteMany({
      createdAt: { $lt: cutoffDate }
    });
    
    console.log(`Cleaned up ${result.deletedCount} old CSP reports`);
    return result.deletedCount;
  }
  
  async scheduleCleanup() {
    // Очистка каждые 24 часа
    setInterval(async () => {
      try {
        await this.cleanup();
      } catch (error) {
        console.error('Error during CSP report cleanup:', error);
      }
    }, 24 * 60 * 60 * 1000); // 24 часа
  }
}

// Использование
const cleanupService = new CspReportCleanup(cspReportModel);
cleanupService.scheduleCleanup();
```

### 2. Алгоритмы дедупликации

```javascript
// Сервис дедупликации CSP отчетов
class CspReportDeduplicator {
  constructor() {
    this.recentReports = new Map();
    this.timeWindow = 5 * 60 * 1000; // 5 минут
  }
  
  isDuplicate(report) {
    const signature = this.generateSignature(report);
    const now = Date.now();
    
    if (this.recentReports.has(signature)) {
      const lastSeen = this.recentReports.get(signature);
      if (now - lastSeen < this.timeWindow) {
        return true;
      }
    }
    
    this.recentReports.set(signature, now);
    this.cleanupOldEntries(now);
    
    return false;
  }
  
  generateSignature(report) {
    return `${report['violated-directive']}_${report['blocked-uri']}_${report['document-uri']}`;
  }
  
  cleanupOldEntries(now) {
    for (const [signature, timestamp] of this.recentReports) {
      if (now - timestamp >= this.timeWindow) {
        this.recentReports.delete(signature);
      }
    }
  }
}

const deduplicator = new CspReportDeduplicator();

// Использование в обработчике
app.post('/csp-report', (req, res) => {
  const report = req.body['csp-report'] || {};
  
  if (deduplicator.isDuplicate(report)) {
    console.log('Duplicate CSP report ignored');
    return res.status(204).end();
  }
  
  // Обработка уникального отчета
  cspHandler.handleReport(req, res);
});
```

## Мониторинг и оповещения

### 1. Система оповещений

```javascript
// Система оповещений для CSP нарушений
class CspAlertSystem {
  constructor(options = {}) {
    this.thresholds = options.thresholds || {
      high: 5,    // 5 высокоприоритетных нарушений в минуту
      medium: 20, // 20 средних нарушений в минуту
      low: 50     // 50 низкоприоритетных нарушений в минуту
    };
    
    this.alertWindow = options.alertWindow || 60000; // 1 минута
    this.violationsInWindow = new Map();
  }
  
  async checkAndAlert(report) {
    const severity = this.calculateSeverity(report);
    const now = Date.now();
    
    // Добавление нарушения в окно
    if (!this.violationsInWindow.has(severity)) {
      this.violationsInWindow.set(severity, []);
    }
    
    this.violationsInWindow.get(severity).push(now);
    
    // Очистка старых нарушений
    this.cleanupOldViolations(now);
    
    // Проверка превышения порога
    const count = this.violationsInWindow.get(severity).length;
    if (count >= this.thresholds[severity]) {
      await this.sendAlert(severity, count, report);
      this.violationsInWindow.get(severity).length = 0; // Сброс
    }
  }
  
  cleanupOldViolations(now) {
    for (const [severity, timestamps] of this.violationsInWindow) {
      const validTimestamps = timestamps.filter(ts => now - ts < this.alertWindow);
      this.violationsInWindow.set(severity, validTimestamps);
    }
  }
  
  calculateSeverity(report) {
    const directive = report['violated-directive'];
    
    if (directive.includes('script-src') || directive.includes('object-src')) {
      return 'high';
    } else if (directive.includes('style-src') || directive.includes('img-src')) {
      return 'medium';
    }
    return 'low';
  }
  
  async sendAlert(severity, count, sampleReport) {
    console.log(`🚨 CSP ALERT: ${count} ${severity} severity violations detected!`);
    console.log('Sample violation:', sampleReport);
    
    // Здесь можно добавить отправку в Slack, email, etc.
    // await this.sendToSlack(severity, count, sampleReport);
  }
}

const alertSystem = new CspAlertSystem();
```

## Лучшие практики

> [!tip] Лучшие практики CSP отчетности
> 1. Используйте режим отчетности перед включением блокировки
> 2. Регулярно анализируйте отчеты для улучшения политики
> 3. Реализуйте дедупликацию для уменьшения шума
> 4. Настройте оповещения для критических нарушений
> 5. Архивируйте отчеты для долгосрочного анализа

### Пример комплексной системы

```javascript
// Комплексная система CSP отчетности
class ComprehensiveCspReporting {
  constructor(options = {}) {
    this.handler = new CspReportHandler(options);
    this.analyzer = new CspReportAnalyzer(options.reportModel);
    this.deduplicator = new CspReportDeduplicator();
    this.alertSystem = new CspAlertSystem(options.alerts);
    this.cleanup = new CspReportCleanup(options.reportModel, options.retentionDays || 30);
  }
  
  async initialize() {
    // Запуск очистки
    this.cleanup.scheduleCleanup();
  }
  
  async handleReport(req, res) {
    const report = req.body['csp-report'] || {};
    
    // Проверка дубликатов
    if (this.deduplicator.isDuplicate(report)) {
      return res.status(204).end();
    }
    
    // Обработка отчета
    await this.handler.handleReport(req, res);
    
    // Проверка оповещений
    await this.alertSystem.checkAndAlert(report);
  }
}

// Использование
const cspReporting = new ComprehensiveCspReporting({
  retentionDays: 60,
  alerts: {
    thresholds: { high: 3, medium: 10, low: 25 }
  }
});

await cspReporting.initialize();
app.post('/csp-report', (req, res) => cspReporting.handleReport(req, res));
```

## Связь с другими аспектами безопасности

Отчеты CSP тесно связаны с:
- [[Директивы-CSP]] — понимание директив для анализа отчетов
- [[Реализация-CSP]] — настройка отчетности в политике
- [[Оценка-CSP]] — анализ эффективности политик
- [[Тестирование-безопасности]] — методы проверки CSP
- [[Мониторинг-безопасности]] — общая концепция мониторинга
- [[Аудит-безопасности]] — анализ безопасности системы

## Заключение

Отчеты Content Security Policy являются критически важным инструментом для понимания и улучшения безопасности веб-приложений. Правильно реализованная система отчетности позволяет разработчикам выявлять проблемы с политиками безопасности, предотвращать ложные срабатывания и обеспечивать баланс между безопасностью и функциональностью приложения.

## Дополнительные ресурсы

- CSP Reporting Specification
- OWASP CSP Cheat Sheet
- Browser Security Handbook
- CSP Violation Reports Guide
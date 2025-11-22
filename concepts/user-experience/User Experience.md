---
aliases: [UX, User Experience, Пользовательский опыт]
tags: 
  - user-experience
  - product-design
  - ci-cd
  - analytics
  - experimentation
  - human-computer-interaction
---

# Пользовательский опыт в CI/CD

## Обзор

Пользовательский опыт (User Experience, UX) в CI/CD - это интеграция принципов и практик UX-дизайна в процессы непрерывной интеграции и доставки. Это включает в себя систематическое измерение, анализ и улучшение взаимодействия пользователей с продуктом на каждом этапе разработки и доставки. В контексте CI/CD UX обеспечивает, что изменения программного обеспечения не только технически корректны, но и улучшают или, по крайней мере, не ухудшают опыт пользователей.

UX в CI/CD представляет собой синтез традиционного UX-дизайна с современными практиками DevOps, обеспечивая, что пользовательские потребности остаются в центре процесса доставки программного обеспечения. Это критически важно для создания продуктов, которые не только функциональны, но и удобны, эффективны и приятны в использовании.

## Основы пользовательского опыта

### Что такое пользовательский опыт

Пользовательский опыт - это восприятие и реакция пользователя на взаимодействие с продуктом, системой или услугой. В контексте цифровых продуктов UX включает:

- **Юзабилити** - удобство использования
- **Доступность** - возможность использования всеми пользователями
- **Эмоциональную реакцию** - чувства, возникающие при использовании
- **Ценность** - полезность и значимость для пользователя
- **Удовлетворенность** - общий уровень удовлетворенности

### Принципы UX-дизайна

#### Приоритет пользователя
Потребности, ожидания и ограничения пользователей должны быть главным приоритетом при проектировании и разработке.

#### Доступность
Продукт должен быть доступен для всех пользователей, включая людей с ограниченными возможностями.

#### Прозрачность
Интерфейс должен быть интуитивно понятным, а действия пользователя - предсказуемыми.

#### Эффективность
Пользователи должны иметь возможность достигать своих целей с минимальными усилиями.

#### Удовлетворенность
Взаимодействие должно быть не только функциональным, но и приятным.

## UX в процессе CI/CD

### Интеграция UX в CI/CD пайплайн

```yaml
# .github/workflows/ux-cicd.yml
name: UX-Centric CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_ENV: test

jobs:
  ux-audit:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run accessibility audit
      run: |
        npm run test:accessibility
        npx lighthouse --quiet --output=json --output-path=./lighthouse-report.json \
          --chrome-flags="--headless" http://localhost:3000
        npx pa11y-ci --sitemap-base http://localhost:3000/ \
          --sitemap-url http://localhost:3000/sitemap.xml
    
    - name: Run UX metrics collection
      run: |
        npm run test:ux-metrics
    
    - name: Upload UX audit results
      uses: actions/upload-artifact@v3
      with:
        name: ux-audit-results
        path: |
          lighthouse-report.json
          pa11y-report.json
          ux-metrics.json

  visual-regression-test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Setup test environment
      run: |
        npm run build
        npx serve -s build &
        sleep 10  # Wait for server to start
    
    - name: Run visual regression tests
      run: |
        npm run test:visual-regression
    
    - name: Upload visual regression results
      uses: actions/upload-artifact@v3
      with:
        name: visual-regression-results
        path: visual-regression-results/
    
    - name: Fail if visual regressions detected
      run: |
        if [ -f "visual-regression-results/failures.txt" ]; then
          echo "Visual regressions detected!"
          cat visual-regression-results/failures.txt
          exit 1
        fi

  user-feedback-integration:
    runs-on: ubuntu-latest
    needs: [ux-audit, visual-regression-test]
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Collect user feedback
      run: |
        # Сбор и анализ пользовательского фидбека
        npm run analyze:user-feedback -- --since=$(date -d "1 day ago" +%Y-%m-%d)
    
    - name: Generate UX impact report
      run: |
        npm run report:ux-impact > ux-impact-report.md
    
    - name: Notify UX team
      run: |
        # Отправка уведомления команде UX
        curl -X POST -H 'Content-type: application/json' \
          --data '{"text":"New deployment with UX impact report: `ux-impact-report.md`"}' \
          ${{ secrets.SLACK_WEBHOOK_UX_TEAM }}

  deploy-staging-ux:
    runs-on: ubuntu-latest
    needs: [ux-audit, visual-regression-test]
    environment: staging
    steps:
    - name: Deploy to staging with UX monitoring
      run: |
        # Деплой в staging с включенным UX мониторингом
        kubectl set env deployment/myapp UX_MONITORING_ENABLED=true
        kubectl rollout status deployment/myapp -n staging
    
    - name: Run UX smoke tests
      run: |
        npm run test:ux-smoke -- --target=staging
    
    - name: Collect UX metrics from staging
      run: |
        npm run collect:ux-metrics -- --environment=staging --duration=300

  deploy-production:
    runs-on: ubuntu-latest
    needs: deploy-staging-ux
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
    - name: Deploy to production
      run: |
        kubectl set env deployment/myapp UX_MONITORING_ENABLED=true
        kubectl rollout status deployment/myapp -n production
    
    - name: Monitor UX metrics in production
      run: |
        npm run monitor:ux-metrics -- --environment=production --alert-thresholds
    
    - name: Generate UX release report
      run: |
        npm run report:ux-release > ux-release-report.md
```

### UX-метрики в CI/CD

```javascript
// ux-metrics-collector.js
const puppeteer = require('puppeteer');
const lighthouse = require('lighthouse');
const { get } = require('lodash');

class UXMetricsCollector {
  constructor(options = {}) {
    this.options = {
      port: 8042,
      outputDir: './ux-metrics',
      thresholds: {
        performance: 90,
        accessibility: 90,
        bestPractices: 90,
        seo: 90,
        pwa: 80,
        firstContentfulPaint: 1800, // ms
        largestContentfulPaint: 2500, // ms
        cumulativeLayoutShift: 0.1,
        firstInputDelay: 100, // ms
        timeToInteractive: 3800 // ms
      },
      ...options
    };
  }

  async collectMetrics(url) {
    const browser = await puppeteer.launch({ 
      headless: true,
      args: ['--no-sandbox', '--disable-setuid-sandbox']
    });
    
    try {
      // Сбор метрик Lighthouse
      const lighthouseResult = await lighthouse(url, {
        port: this.options.port,
        output: 'json',
        onlyCategories: ['performance', 'accessibility', 'best-practices', 'seo', 'pwa']
      });

      // Сбор пользовательских UX метрик
      const customMetrics = await this.collectCustomMetrics(browser, url);
      
      // Комбинирование результатов
      const combinedMetrics = {
        timestamp: new Date().toISOString(),
        url,
        lighthouse: lighthouseResult.lhr,
        custom: customMetrics,
        overallScore: this.calculateOverallUXScore(lighthouseResult.lhr, customMetrics)
      };

      return combinedMetrics;
    } finally {
      await browser.close();
    }
  }

  async collectCustomMetrics(browser, url) {
    const page = await browser.newPage();
    await page.goto(url, { waitUntil: 'networkidle2' });

    // Сбор кастомных UX метрик
    const customMetrics = await page.evaluate(() => {
      return {
        // Метрики загрузки
        domContentLoaded: performance.timing.domContentLoadedEventEnd - performance.timing.navigationStart,
        pageLoadComplete: performance.timing.loadEventEnd - performance.timing.navigationStart,
        
        // Метрики взаимодействия
        hasAnimations: document.querySelectorAll('[class*="animate"], [style*="animation"], [style*="transition"]').length > 0,
        interactiveElements: document.querySelectorAll('button, a, input, select, textarea, [tabindex]:not([tabindex="-1"])').length,
        
        // Метрики контента
        textToImageRatio: document.body.innerText.length / Math.max(1, document.querySelectorAll('img, svg').length),
        headingStructure: Array.from(document.querySelectorAll('h1, h2, h3, h4, h5, h6'))
          .map(h => h.tagName.toLowerCase())
          .reduce((acc, tag) => ({ ...acc, [tag]: (acc[tag] || 0) + 1 }), {}),
        
        // Метрики формы
        formComplexity: document.querySelectorAll('form input, form select, form textarea').length,
        requiredFields: document.querySelectorAll('input[required], select[required], textarea[required]').length
      };
    });

    // Дополнительные метрики с помощью Puppeteer
    const screenshot = await page.screenshot({ fullPage: true });
    customMetrics.pageHeight = screenshot.height;
    
    return customMetrics;
  }

  calculateOverallUXScore(lighthouseResult, customMetrics) {
    const scores = lighthouseResult.categories;
    
    // Взвешенный расчет общего UX балла
    const performanceWeight = 0.3;
    const accessibilityWeight = 0.25;
    const bestPracticesWeight = 0.15;
    const seoWeight = 0.15;
    const pwaWeight = 0.15;
    
    const overallScore = 
      (scores.performance.score * 100 * performanceWeight) +
      (scores.accessibility.score * 100 * accessibilityWeight) +
      (scores['best-practices'].score * 100 * bestPracticesWeight) +
      (scores.seo.score * 100 * seoWeight) +
      (scores.pwa.score * 100 * pwaWeight);
    
    return Math.round(overallScore);
  }

  async validateAgainstThresholds(metrics) {
    const violations = [];
    
    // Проверка Lighthouse метрик
    const lhr = metrics.lighthouse;
    if (lhr.categories.performance.score < 0.9) {
      violations.push({
        type: 'performance',
        severity: 'high',
        current: lhr.categories.performance.score * 100,
        threshold: 90,
        message: 'Performance score below threshold'
      });
    }
    
    if (lhr.categories.accessibility.score < 0.9) {
      violations.push({
        type: 'accessibility',
        severity: 'high',
        current: lhr.categories.accessibility.score * 100,
        threshold: 90,
        message: 'Accessibility score below threshold'
      });
    }
    
    // Проверка Core Web Vitals
    const audits = lhr.audits;
    if (audits['largest-contentful-paint'] && audits['largest-contentful-paint'].numericValue > 2500) {
      violations.push({
        type: 'largest-contentful-paint',
        severity: 'medium',
        current: audits['largest-contentful-paint'].numericValue,
        threshold: 2500,
        message: 'LCP above recommended threshold'
      });
    }
    
    if (audits['cumulative-layout-shift'] && audits['cumulative-layout-shift'].numericValue > 0.1) {
      violations.push({
        type: 'cumulative-layout-shift',
        severity: 'medium',
        current: audits['cumulative-layout-shift'].numericValue,
        threshold: 0.1,
        message: 'CLS above recommended threshold'
      });
    }
    
    if (audits['first-input-delay'] && audits['first-input-delay'].numericValue > 100) {
      violations.push({
        type: 'first-input-delay',
        severity: 'medium',
        current: audits['first-input-delay'].numericValue,
        threshold: 100,
        message: 'FID above recommended threshold'
      });
    }
    
    // Проверка кастомных метрик
    if (metrics.custom.formComplexity > 10) {
      violations.push({
        type: 'form-complexity',
        severity: 'low',
        current: metrics.custom.formComplexity,
        threshold: 10,
        message: 'Form has too many input fields'
      });
    }
    
    return {
      isValid: violations.length === 0,
      violations,
      overallScore: metrics.overallScore
    };
  }

  async generateUXReport(metrics, outputPath) {
    const fs = require('fs').promises;
    
    const report = {
      summary: {
        overallUXScore: metrics.overallScore,
        timestamp: metrics.timestamp,
        testedUrl: metrics.url,
        isValid: (await this.validateAgainstThresholds(metrics)).isValid
      },
      lighthouse: {
        performance: metrics.lighthouse.categories.performance.score * 100,
        accessibility: metrics.lighthouse.categories.accessibility.score * 100,
        bestPractices: metrics.lighthouse.categories['best-practices'].score * 100,
        seo: metrics.lighthouse.categories.seo.score * 100,
        pwa: metrics.lighthouse.categories.pwa.score * 100
      },
      coreWebVitals: {
        lcp: metrics.lighthouse.audits['largest-contentful-paint']?.numericValue,
        cls: metrics.lighthouse.audits['cumulative-layout-shift']?.numericValue,
        fid: metrics.lighthouse.audits['first-input-delay']?.numericValue
      },
      customMetrics: metrics.custom,
      recommendations: this.generateRecommendations(metrics)
    };
    
    await fs.writeFile(outputPath, JSON.stringify(report, null, 2));
    return report;
  }

  generateRecommendations(metrics) {
    const recommendations = [];
    
    if (metrics.lighthouse.categories.performance.score < 0.9) {
      recommendations.push({
        priority: 'high',
        category: 'performance',
        suggestion: 'Optimize images, implement lazy loading, and minimize JavaScript bundles'
      });
    }
    
    if (metrics.lighthouse.categories.accessibility.score < 0.9) {
      recommendations.push({
        priority: 'high',
        category: 'accessibility',
        suggestion: 'Add proper alt texts, ensure sufficient color contrast, and implement keyboard navigation'
      });
    }
    
    if (metrics.custom.formComplexity > 5) {
      recommendations.push({
        priority: 'medium',
        category: 'usability',
        suggestion: 'Consider breaking complex forms into multiple steps or implementing progressive disclosure'
      });
    }
    
    return recommendations;
  }
}

module.exports = UXMetricsCollector;
```

## UX-тестирование в CI/CD

### Визуальное регрессионное тестирование

```javascript
// visual-regression-tests.js
const Percy = require('@percy/sdk-utils');
const puppeteer = require('puppeteer');
const fs = require('fs').promises;
const path = require('path');

class VisualRegressionTester {
  constructor(options = {}) {
    this.options = {
      percyToken: process.env.PERCY_TOKEN,
      snapshotDir: './visual-snapshots',
      comparisonThreshold: 0.1, // 10% difference threshold
      viewportSizes: [
        { width: 375, height: 667, deviceScaleFactor: 2, isMobile: true }, // Mobile
        { width: 1280, height: 720, deviceScaleFactor: 1, isMobile: false }, // Desktop
        { width: 1024, height: 768, deviceScaleFactor: 1, isMobile: false } // Tablet
      ],
      ...options
    };
  }

  async runVisualRegressionTests(urls, buildName = 'default') {
    const browser = await puppeteer.launch({ 
      headless: true,
      args: ['--no-sandbox', '--disable-setuid-sandbox']
    });
    
    const results = {
      passed: [],
      failed: [],
      errors: []
    };

    try {
      // Инициализация Percy (если токен предоставлен)
      let percy = null;
      if (this.options.percyToken) {
        percy = new Percy({ token: this.options.percyToken });
        await percy.startBuild({ name: buildName });
      }

      for (const url of urls) {
        for (const viewport of this.options.viewportSizes) {
          try {
            const page = await browser.newPage();
            await page.setViewport(viewport);
            
            // Дополнительные настройки для более стабильных скриншотов
            await page.emulateMediaFeatures([
              { name: 'prefers-reduced-motion', value: 'reduce' }
            ]);
            
            await page.goto(url, { waitUntil: 'networkidle2', timeout: 30000 });
            
            // Дождемся загрузки всех асинхронных компонентов
            await page.waitForFunction(() => {
              return document.readyState === 'complete' && 
                     typeof window.REACT_APP_LOADED !== 'undefined';
            }, { timeout: 10000 });
            
            // Скрываем динамические элементы для стабильности
            await page.evaluate(() => {
              // Скрытие элементов с анимацией
              const animatedElements = document.querySelectorAll('[class*="animate"], [style*="animation"]');
              animatedElements.forEach(el => {
                el.style.visibility = 'hidden';
              });
              
              // Скрытие таймеров и динамических данных
              const dynamicElements = document.querySelectorAll('[data-timer], [data-timestamp], .live-data');
              dynamicElements.forEach(el => {
                el.style.visibility = 'hidden';
              });
            });
            
            // Снятие скриншота
            const screenshotBuffer = await page.screenshot({
              fullPage: true,
              omitBackground: false
            });
            
            const fileName = `${this.slugify(url)}-${viewport.width}x${viewport.height}.png`;
            const filePath = path.join(this.options.snapshotDir, fileName);
            
            // Создание директории если не существует
            await fs.mkdir(this.options.snapshotDir, { recursive: true });
            await fs.writeFile(filePath, screenshotBuffer);
            
            // Сравнение с эталоном (упрощенная логика)
            const comparisonResult = await this.compareWithBaseline(filePath, fileName);
            
            if (comparisonResult.isDifferent) {
              results.failed.push({
                url,
                viewport,
                fileName,
                differencePercentage: comparisonResult.differencePercentage,
                baselinePath: comparisonResult.baselinePath,
                currentPath: filePath
              });
              
              // Сохранение скриншота различий
              await this.generateDiffImage(
                comparisonResult.baselinePath,
                filePath,
                path.join(this.options.snapshotDir, `diff-${fileName}`)
              );
            } else {
              results.passed.push({
                url,
                viewport,
                fileName,
                similarity: comparisonResult.similarity
              });
            }
            
            await page.close();
            
            // Использование Percy для облачного сравнения
            if (percy) {
              await percy.snapshot({
                name: `${url}-${viewport.width}x${viewport.height}`,
                url,
                widths: [viewport.width],
                minHeight: viewport.height
              });
            }
          } catch (error) {
            results.errors.push({
              url,
              viewport,
              error: error.message
            });
          }
        }
      }
      
      // Завершение Percy билда
      if (percy) {
        await percy.finishBuild();
      }
      
    } finally {
      await browser.close();
    }
    
    return results;
  }

  async compareWithBaseline(currentPath, fileName) {
    const baselinePath = path.join(this.options.snapshotDir, 'baseline', fileName);
    
    try {
      // Проверяем существует ли эталон
      await fs.access(baselinePath);
      
      // В реальной реализации здесь будет сравнение изображений
      // Используя библиотеки типа pixelmatch, resemblejs и т.д.
      const similarity = await this.calculateImageSimilarity(currentPath, baselinePath);
      
      return {
        isDifferent: similarity < (1 - this.options.comparisonThreshold),
        similarity,
        differencePercentage: (1 - similarity) * 100,
        baselinePath
      };
    } catch (error) {
      // Если эталон не существует, создаем его как эталон
      const baselineDir = path.dirname(baselinePath);
      await fs.mkdir(baselineDir, { recursive: true });
      await fs.copyFile(currentPath, baselinePath);
      
      return {
        isDifferent: false,
        similarity: 1.0,
        differencePercentage: 0,
        baselinePath
      };
    }
  }

  async calculateImageSimilarity(image1Path, image2Path) {
    // В реальной реализации использовать библиотеку для сравнения изображений
    // Такую как pixelmatch, resemblejs, или opencv
    return 0.95; // Заглушка
  }

  async generateDiffImage(image1Path, image2Path, outputPath) {
    // В реальной реализации генерировать изображение с визуализацией различий
    // Используя библиотеки типа gm, sharp, или canvas
    console.log(`Generating diff image: ${outputPath}`);
  }

  slugify(text) {
    return text.toString().toLowerCase()
      .replace(/\s+/g, '-')           // Replace spaces with -
      .replace(/[^\w\-]+/g, '')       // Remove all non-word chars
      .replace(/\-\-+/g, '-')         // Replace multiple - with single -
      .replace(/^-+/, '')             // Trim - from start
      .replace(/-+$/, '');            // Trim - from end
  }

  async generateTestReport(results) {
    const report = {
      timestamp: new Date().toISOString(),
      totalTests: results.passed.length + results.failed.length + results.errors.length,
      passed: results.passed.length,
      failed: results.failed.length,
      errors: results.errors.length,
      passRate: (results.passed.length / 
                (results.passed.length + results.failed.length + results.errors.length)) * 100,
      details: {
        passed: results.passed,
        failed: results.failed,
        errors: results.errors
      }
    };

    const reportPath = path.join(this.options.snapshotDir, 'visual-test-report.json');
    await fs.writeFile(reportPath, JSON.stringify(report, null, 2));
    
    return report;
  }
}

module.exports = VisualRegressionTester;
```

### Тестирование доступности

```javascript
// accessibility-tests.js
const pa11y = require('pa11y');
const fs = require('fs').promises;
const path = require('path');

class AccessibilityTester {
  constructor(options = {}) {
    this.options = {
      standard: 'WCAG2AA', // WCAG2A, WCAG2AA, WCAG2AAA, Section508
      runners: ['axe', 'htmlcs'], // Используемые инструменты проверки
      ignore: [
        'notice', // Игнорировать замечания
        'warning' // Игнорировать предупреждения
      ],
      includeWarnings: false,
      includeNotices: false,
      threshold: 0, // Максимальное количество ошибок
      viewport: {
        width: 1280,
        height: 720
      },
      ...options
    };
  }

  async runAccessibilityTests(urls, outputDir = './accessibility-reports') {
    const results = {
      passed: [],
      failed: [],
      errors: [],
      summary: {
        totalIssues: 0,
        totalErrors: 0,
        totalWarnings: 0,
        totalNotices: 0
      }
    };

    // Создание директории для отчетов
    await fs.mkdir(outputDir, { recursive: true });

    for (const url of urls) {
      try {
        const testResult = await pa11y(url, {
          ...this.options,
          chromeLaunchConfig: {
            args: ['--no-sandbox', '--disable-setuid-sandbox']
          }
        });

        // Обновление сводки
        results.summary.totalIssues += testResult.issues.length;
        testResult.issues.forEach(issue => {
          if (issue.type === 'error') results.summary.totalErrors++;
          else if (issue.type === 'warning') results.summary.totalWarnings++;
          else if (issue.type === 'notice') results.summary.totalNotices++;
        });

        if (testResult.issues.length > this.options.threshold) {
          results.failed.push({
            url,
            issues: testResult.issues,
            errorCount: testResult.issues.filter(i => i.type === 'error').length,
            warningCount: testResult.issues.filter(i => i.type === 'warning').length,
            noticeCount: testResult.issues.filter(i => i.type === 'notice').length
          });
        } else {
          results.passed.push({
            url,
            issues: testResult.issues,
            issueCount: testResult.issues.length
          });
        }

        // Сохранение детального отчета
        const reportPath = path.join(outputDir, `${this.slugify(url)}-accessibility-report.json`);
        await fs.writeFile(reportPath, JSON.stringify(testResult, null, 2));

      } catch (error) {
        results.errors.push({
          url,
          error: error.message
        });
      }
    }

    return results;
  }

  async runBatchAccessibilityTest(sitemapUrl, options = {}) {
    // В реальной реализации загружать sitemap и тестировать все URL
    // Здесь упрощенная версия
    const urls = await this.extractUrlsFromSitemap(sitemapUrl);
    return await this.runAccessibilityTests(urls, options.outputDir);
  }

  async extractUrlsFromSitemap(sitemapUrl) {
    // Реализация извлечения URL из sitemap
    // Возвращает массив URL для тестирования
    return [
      'https://example.com/',
      'https://example.com/about',
      'https://example.com/contact',
      'https://example.com/products'
    ];
  }

  async generateAccessibilityReport(results, outputPath) {
    const report = {
      summary: results.summary,
      details: {
        passed: results.passed.length,
        failed: results.failed.length,
        errors: results.errors.length
      },
      compliance: this.calculateCompliance(results),
      recommendations: this.generateRecommendations(results),
      timestamp: new Date().toISOString()
    };

    await fs.writeFile(outputPath, JSON.stringify(report, null, 2));
    return report;
  }

  calculateCompliance(results) {
    const totalPages = results.passed.length + results.failed.length;
    const compliantPages = results.passed.length;
    
    return {
      percentage: totalPages > 0 ? (compliantPages / totalPages) * 100 : 0,
      compliantPages,
      totalPages,
      needsAttention: results.failed.length
    };
  }

  generateRecommendations(results) {
    const recommendations = [];
    
    // Анализ частых проблем
    const allIssues = results.failed.flatMap(f => f.issues);
    const issueCounts = {};
    
    allIssues.forEach(issue => {
      issueCounts[issue.code] = (issueCounts[issue.code] || 0) + 1;
    });
    
    // Рекомендации на основе частых проблем
    Object.entries(issueCounts).forEach(([issueCode, count]) => {
      if (count > 5) { // Если проблема встречается более 5 раз
        recommendations.push({
          issue: issueCode,
          occurrences: count,
          recommendation: this.getRecommendationForIssue(issueCode)
        });
      }
    });
    
    return recommendations;
  }

  getRecommendationForIssue(issueCode) {
    const recommendations = {
      'WCAG2AA.Principle1.Guideline1_1.1_1_1.H30.2': 'Add descriptive alt text to images',
      'WCAG2AA.Principle1.Guideline1_3.1_3_1_AAA.H42.2': 'Use proper heading hierarchy (h1, h2, h3...)',
      'WCAG2AA.Principle1.Guideline1_4.1_4_3.G18.Fail': 'Improve color contrast ratio',
      'WCAG2AA.Principle2.Guideline2_4.2_4_4.H77,H78,H79,H80,H81': 'Add skip links for navigation',
      'WCAG2AA.Principle4.Guideline4_1.4_1_2.H91.A.NoContent': 'Add meaningful link text',
      'WCAG2AA.Principle1.Guideline1_3.1_3_5.H98': 'Implement proper form labels'
    };
    
    return recommendations[issueCode] || 'Review accessibility guidelines for this issue';
  }

  slugify(text) {
    return text.toString().toLowerCase()
      .replace(/\s+/g, '-')           
      .replace(/[^\w\-]+/g, '')       
      .replace(/\-\-+/g, '-')         
      .replace(/^-+/, '')             
      .replace(/-+$/, '');
  }
}

module.exports = AccessibilityTester;
```

## UX-метрики и аналитика

### Сбор UX-метрик в реальном времени

```javascript
// real-time-ux-monitoring.js
const express = require('express');
const WebSocket = require('ws');
const http = require('http');
const fs = require('fs').promises;

class RealTimeUXMonitor {
  constructor(options = {}) {
    this.options = {
      port: 8080,
      wsPort: 8081,
      metricsStorage: './ux-metrics-store',
      alertThresholds: {
        errorRate: 0.05, // 5% error rate
        bounceRate: 0.6, // 60% bounce rate
        sessionDuration: 10, // 10 seconds minimum
        pageLoadTime: 3000 // 3 seconds max load time
      },
      ...options
    };
    
    this.app = express();
    this.server = http.createServer(this.app);
    this.wss = new WebSocket.Server({ port: this.options.wsPort });
    this.metricsBuffer = [];
    this.userSessions = new Map();
    this.alerts = [];
  }

  async initialize() {
    await fs.mkdir(this.options.metricsStorage, { recursive: true });
    this.setupRoutes();
    this.setupWebSocketHandlers();
    this.startMetricsProcessing();
    
    this.server.listen(this.options.port, () => {
      console.log(`UX Monitor server running on port ${this.options.port}`);
    });
  }

  setupRoutes() {
    // Эндпоинт для получения UX метрик
    this.app.get('/api/ux-metrics', (req, res) => {
      const { timeframe = 'hour', metric } = req.query;
      const filteredMetrics = this.getFilteredMetrics(timeframe, metric);
      res.json(filteredMetrics);
    });

    // Эндпоинт для получения UX алертов
    this.app.get('/api/ux-alerts', (req, res) => {
      res.json(this.alerts);
    });

    // Эндпоинт для получения сводки UX
    this.app.get('/api/ux-summary', (req, res) => {
      const summary = this.getUXSummary();
      res.json(summary);
    });
  }

  setupWebSocketHandlers() {
    this.wss.on('connection', (ws) => {
      console.log('UX Monitor client connected');
      
      // Отправка последних метрик новому клиенту
      ws.send(JSON.stringify({
        type: 'initial_data',
        metrics: this.getLastMetrics(10)
      }));

      ws.on('message', (message) => {
        try {
          const data = JSON.parse(message);
          this.handleIncomingMetric(data);
        } catch (error) {
          console.error('Error parsing UX metric:', error);
        }
      });

      ws.on('close', () => {
        console.log('UX Monitor client disconnected');
      });
    });
  }

  handleIncomingMetric(metricData) {
    // Валидация данных
    if (!this.isValidMetric(metricData)) {
      console.error('Invalid metric data:', metricData);
      return;
    }

    // Добавление временной метки
    metricData.timestamp = new Date().toISOString();
    
    // Добавление в буфер
    this.metricsBuffer.push(metricData);
    
    // Ограничение размера буфера
    if (this.metricsBuffer.length > 10000) {
      this.metricsBuffer = this.metricsBuffer.slice(-5000);
    }

    // Обработка сессии пользователя
    this.processUserSession(metricData);

    // Проверка пороговых значений
    this.checkAlertThresholds(metricData);

    // Рассылка обновлений подписчикам
    this.broadcastMetricUpdate(metricData);
  }

  isValidMetric(metricData) {
    const requiredFields = ['userId', 'sessionId', 'eventType', 'timestamp'];
    return requiredFields.every(field => metricData.hasOwnProperty(field));
  }

  processUserSession(metricData) {
    const sessionId = metricData.sessionId;
    
    if (!this.userSessions.has(sessionId)) {
      this.userSessions.set(sessionId, {
        userId: metricData.userId,
        startTime: new Date(metricData.timestamp),
        events: [],
        metrics: {
          pageViews: 0,
          clicks: 0,
          scrollDepth: 0,
          timeOnPage: 0,
          errors: 0
        }
      });
    }

    const session = this.userSessions.get(sessionId);
    session.events.push(metricData);

    // Обновление метрик сессии
    switch (metricData.eventType) {
      case 'page_view':
        session.metrics.pageViews++;
        break;
      case 'click':
        session.metrics.clicks++;
        break;
      case 'scroll':
        session.metrics.scrollDepth = Math.max(
          session.metrics.scrollDepth,
          metricData.scrollPercentage || 0
        );
        break;
      case 'error':
        session.metrics.errors++;
        break;
    }

    // Ограничение количества событий на сессию
    if (session.events.length > 1000) {
      session.events = session.events.slice(-500);
    }
  }

  checkAlertThresholds(metricData) {
    const currentMetrics = this.getCurrentMetrics();

    // Проверка уровня ошибок
    if (currentMetrics.errorRate > this.options.alertThresholds.errorRate) {
      this.triggerAlert({
        type: 'high_error_rate',
        severity: 'high',
        message: `Error rate is ${currentMetrics.errorRate * 100}%`,
        currentValue: currentMetrics.errorRate,
        threshold: this.options.alertThresholds.errorRate
      });
    }

    // Проверка времени загрузки страницы
    if (metricData.eventType === 'page_load' && 
        metricData.loadTime > this.options.alertThresholds.pageLoadTime) {
      this.triggerAlert({
        type: 'slow_page_load',
        severity: 'medium',
        message: `Page load time is ${metricData.loadTime}ms`,
        currentValue: metricData.loadTime,
        threshold: this.options.alertThresholds.pageLoadTime
      });
    }

    // Проверка продолжительности сессии
    if (metricData.eventType === 'session_end') {
      const sessionDuration = (new Date(metricData.timestamp) - 
                              new Date(metricData.startTime)) / 1000; // в секундах
      
      if (sessionDuration < this.options.alertThresholds.sessionDuration) {
        this.triggerAlert({
          type: 'short_session',
          severity: 'low',
          message: `Session duration is ${sessionDuration}s`,
          currentValue: sessionDuration,
          threshold: this.options.alertThresholds.sessionDuration
        });
      }
    }
  }

  triggerAlert(alertData) {
    alertData.timestamp = new Date().toISOString();
    alertData.id = `alert_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    
    this.alerts.push(alertData);
    
    // Ограничение истории алертов
    if (this.alerts.length > 1000) {
      this.alerts = this.alerts.slice(-500);
    }

    // Рассылка алерта подписчикам
    this.broadcastAlert(alertData);
    
    // Логирование алерта
    console.log(`UX Alert: ${alertData.type} - ${alertData.message}`);
  }

  broadcastMetricUpdate(metricData) {
    this.wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({
          type: 'metric_update',
          data: metricData
        }));
      }
    });
  }

  broadcastAlert(alertData) {
    this.wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({
          type: 'alert',
          data: alertData
        }));
      }
    });
  }

  getCurrentMetrics() {
    const now = new Date();
    const oneHourAgo = new Date(now.getTime() - 60 * 60 * 1000);
    
    const recentMetrics = this.metricsBuffer.filter(
      m => new Date(m.timestamp) > oneHourAgo
    );

    const totalEvents = recentMetrics.length;
    const errorEvents = recentMetrics.filter(m => m.eventType === 'error').length;
    
    return {
      totalEvents,
      errorEvents,
      errorRate: totalEvents > 0 ? errorEvents / totalEvents : 0,
      activeSessions: this.userSessions.size,
      avgSessionDuration: this.calculateAvgSessionDuration(),
      bounceRate: this.calculateBounceRate()
    };
  }

  calculateAvgSessionDuration() {
    let totalDuration = 0;
    let completedSessions = 0;

    for (const [sessionId, session] of this.userSessions) {
      if (session.endTime) {
        const duration = (new Date(session.endTime) - new Date(session.startTime)) / 1000;
        totalDuration += duration;
        completedSessions++;
      }
    }

    return completedSessions > 0 ? totalDuration / completedSessions : 0;
  }

  calculateBounceRate() {
    let bouncedSessions = 0;
    let totalSessions = this.userSessions.size;

    for (const [sessionId, session] of this.userSessions) {
      if (session.metrics.pageViews === 1) {
        bouncedSessions++;
      }
    }

    return totalSessions > 0 ? bouncedSessions / totalSessions : 0;
  }

  getFilteredMetrics(timeframe, metricType) {
    const now = new Date();
    let startTime;

    switch (timeframe) {
      case 'minute':
        startTime = new Date(now.getTime() - 60 * 1000);
        break;
      case 'hour':
        startTime = new Date(now.getTime() - 60 * 60 * 1000);
        break;
      case 'day':
        startTime = new Date(now.getTime() - 24 * 60 * 60 * 1000);
        break;
      default:
        startTime = new Date(now.getTime() - 60 * 60 * 1000); // по умолчанию 1 час
    }

    let filtered = this.metricsBuffer.filter(
      m => new Date(m.timestamp) > startTime
    );

    if (metricType) {
      filtered = filtered.filter(m => m.eventType === metricType);
    }

    return {
      timeframe,
      metricType: metricType || 'all',
      data: filtered,
      summary: this.calculateMetricsSummary(filtered)
    };
  }

  calculateMetricsSummary(metrics) {
    if (metrics.length === 0) return {};

    const summary = {
      total: metrics.length,
      eventsByType: {},
      avgValues: {},
      timeRange: {
        start: new Date(Math.min(...metrics.map(m => new Date(m.timestamp)))),
        end: new Date(Math.max(...metrics.map(m => new Date(m.timestamp))))
      }
    };

    // Подсчет событий по типам
    metrics.forEach(metric => {
      summary.eventsByType[metric.eventType] = 
        (summary.eventsByType[metric.eventType] || 0) + 1;
    });

    return summary;
  }

  getLastMetrics(count = 10) {
    return this.metricsBuffer.slice(-count);
  }

  getUXSummary() {
    const currentMetrics = this.getCurrentMetrics();
    const complianceMetrics = this.getComplianceMetrics();
    
    return {
      realTime: currentMetrics,
      compliance: complianceMetrics,
      alerts: {
        active: this.alerts.filter(a => !a.resolved).length,
        resolved: this.alerts.filter(a => a.resolved).length,
        highSeverity: this.alerts.filter(a => a.severity === 'high').length
      },
      sessions: {
        active: this.userSessions.size,
        avgDuration: this.calculateAvgSessionDuration(),
        bounceRate: this.calculateBounceRate()
      },
      timestamp: new Date().toISOString()
    };
  }

  getComplianceMetrics() {
    // Здесь могла бы быть логика для получения метрик соответствия
    // например, доступности, юзабилити и т.д.
    return {
      accessibilityScore: 95, // Пример значения
      usabilityScore: 88, // Пример значения
      performanceScore: 92 // Пример значения
    };
  }

  startMetricsProcessing() {
    // Регулярная обработка метрик (например, агрегация, очистка старых данных)
    setInterval(() => {
      this.processMetricsAggregation();
      this.cleanupOldData();
    }, 60000); // Каждую минуту
  }

  processMetricsAggregation() {
    // Агрегация метрик для долгосрочного хранения
    // В реальной реализации здесь будет логика агрегации
    console.log('Processing metrics aggregation...');
  }

  cleanupOldData() {
    // Очистка старых данных, которые уже агрегированы
    const twoHoursAgo = new Date(Date.now() - 2 * 60 * 60 * 1000);
    
    this.metricsBuffer = this.metricsBuffer.filter(
      m => new Date(m.timestamp) > twoHoursAgo
    );
    
    // Очистка завершенных сессий старше 1 часа
    const oneHourAgo = new Date(Date.now() - 60 * 60 * 1000);
    for (const [sessionId, session] of this.userSessions) {
      if (session.endTime && new Date(session.endTime) < oneHourAgo) {
        this.userSessions.delete(sessionId);
      }
    }
  }
}

// Использование
async function startUXMonitor() {
  const monitor = new RealTimeUXMonitor({
    port: 8080,
    wsPort: 8081,
    alertThresholds: {
      errorRate: 0.03,
      bounceRate: 0.5,
      sessionDuration: 15,
      pageLoadTime: 2500
    }
  });

  await monitor.initialize();
  return monitor;
}

if (require.main === module) {
  startUXMonitor().catch(console.error);
}

module.exports = RealTimeUXMonitor;
```

## UX-ориентированные практики CI/CD

### UX-ревью в процессе разработки

```yaml
# ux-review-process.yml
name: UX Review Process

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  ux-review-required:
    runs-on: ubuntu-latest
    if: contains(github.event.pull_request.labels.*.name, 'ux-needed')
    steps:
    - name: Check UX review status
      run: |
        # Проверка наличия UX ревью
        UX_REVIEWS=$(curl -s -H "Authorization: Bearer ${{ secrets.GITHUB_TOKEN }}" \
          "${{ github.event.pull_request.url }}/reviews" | \
          jq -r '.[] | select(.state == "APPROVED" and (.author_association == "MEMBER" or .author_association == "OWNER")) | .user.login' | \
          grep -c "ux-reviewer\|designer")
        
        if [ "$UX_REVIEWS" -eq 0 ]; then
          echo "❌ UX review required but not completed"
          exit 1
        else
          echo "✅ UX review completed"
        fi

  automated-ux-checks:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run design token validation
      run: |
        # Проверка соответствия дизайн-токенам
        npm run validate:design-tokens
    
    - name: Run component library compliance
      run: |
        # Проверка соответствия компонентной библиотеке
        npm run validate:component-library
    
    - name: Run accessibility check
      run: |
        # Автоматическая проверка доступности
        npm run test:accessibility -- --ci
    
    - name: Run visual regression test
      run: |
        # Визуальное регрессионное тестирование
        npm run test:visual-regression -- --ci

  ux-story-validation:
    runs-on: ubuntu-latest
    steps:
    - name: Validate UX stories
      run: |
        # Проверка соответствия юзабилити историям
        python scripts/validate-ux-stories.py \
          --pr-number ${{ github.event.number }} \
          --repo ${{ github.repository }}
```

### UX-документация в CI/CD

```javascript
// ux-documentation-generator.js
const fs = require('fs').promises;
const path = require('path');
const matter = require('gray-matter');

class UXDocumentationGenerator {
  constructor(options = {}) {
    this.options = {
      sourceDir: './src',
      docsDir: './docs/ux',
      templateDir: './templates/ux',
      outputFormat: 'mdx', // md, mdx, html
      ...options
    };
  }

  async generateDocumentation() {
    const components = await this.extractUXComponents();
    const patterns = await this.extractUXPatterns();
    const guidelines = await this.extractUXGuidelines();
    
    await this.generateComponentDocs(components);
    await this.generatePatternDocs(patterns);
    await this.generateGuidelineDocs(guidelines);
    await this.generateUXDashboard();
  }

  async extractUXComponents() {
    // Извлечение UX компонентов из исходного кода
    const components = [];
    const componentFiles = await this.findFiles(
      this.options.sourceDir, 
      /\.(jsx?|tsx?|vue|svelte)$/
    );
    
    for (const file of componentFiles) {
      const content = await fs.readFile(file, 'utf8');
      const uxProps = this.extractUXProps(content, file);
      
      if (uxProps.length > 0) {
        components.push({
          file,
          name: path.basename(file, path.extname(file)),
          uxProps,
          accessibility: this.extractAccessibilityInfo(content),
          userJourney: this.extractUserJourney(content)
        });
      }
    }
    
    return components;
  }

  extractUXProps(content, filePath) {
    // Извлечение UX свойств из компонентов
    const props = [];
    
    // Пример для React компонентов
    const propRegex = /\/\*\*[\s\S]*?@ux\s+(.*?)\*\//g;
    let match;
    
    while ((match = propRegex.exec(content)) !== null) {
      const propInfo = match[1].trim();
      const [name, description] = propInfo.split(' - ');
      
      props.push({
        name: name.trim(),
        description: description ? description.trim() : '',
        file: filePath,
        line: content.substring(0, match.index).split('\n').length
      });
    }
    
    return props;
  }

  extractAccessibilityInfo(content) {
    // Извлечение информации о доступности
    const accessibility = {
      roles: [],
      landmarks: [],
      keyboardSupport: false,
      screenReaderSupport: false
    };
    
    // Поиск ARIA ролей
    const ariaRoles = content.match(/role=["']([^"']*)["']/g);
    if (ariaRoles) {
      accessibility.roles = [...new Set(ariaRoles.map(r => r.match(/role=["']([^"']*)["']/)[1]))];
    }
    
    // Поиск клавиатурных взаимодействий
    if (content.includes('onKeyDown') || content.includes('onKeyUp') || content.includes('onKeyPress')) {
      accessibility.keyboardSupport = true;
    }
    
    // Поиск атрибутов для скринридеров
    if (content.includes('aria-') || content.includes('role=')) {
      accessibility.screenReaderSupport = true;
    }
    
    return accessibility;
  }

  extractUserJourney(content) {
    // Извлечение информации о пользовательском пути
    const journey = {
      trigger: null,
      actions: [],
      outcomes: []
    };
    
    // Поиск паттернов пользовательских действий
    const actionMatches = content.match(/(onClick|onChange|onSubmit|onFocus|onBlur)/g);
    if (actionMatches) {
      journey.actions = [...new Set(actionMatches)];
    }
    
    return journey;
  }

  async generateComponentDocs(components) {
    await fs.mkdir(path.join(this.options.docsDir, 'components'), { recursive: true });
    
    for (const component of components) {
      const docContent = this.createComponentDoc(component);
      const docPath = path.join(
        this.options.docsDir, 
        'components', 
        `${component.name.toLowerCase()}.mdx`
      );
      
      await fs.writeFile(docPath, docContent);
    }
  }

  createComponentDoc(component) {
    const frontmatter = {
      title: `${component.name} UX Documentation`,
      tags: ['ux', 'component', 'accessibility'],
      status: 'draft'
    };
    
    return `---
${Object.entries(frontmatter).map(([key, value]) => 
  `${key}: ${typeof value === 'string' ? `"${value}"` : JSON.stringify(value)}`
).join('\n')}
---

# ${component.name} UX Guidelines

## Overview
Component for ${component.uxProps.map(p => p.description).join(', ') || 'user interaction'}.

## Accessibility
- Roles: ${component.accessibility.roles.join(', ') || 'None'}
- Keyboard Support: ${component.accessibility.keyboardSupport ? 'Yes' : 'No'}
- Screen Reader Support: ${component.accessibility.screenReaderSupport ? 'Yes' : 'No'}

## User Journey
- Actions: ${component.userJourney.actions.join(', ') || 'None'}
- Outcomes: ${component.userJourney.outcomes.join(', ') || 'None'}

## Best Practices
${component.uxProps.map(prop => `- **${prop.name}**: ${prop.description}`).join('\n')}

## Code Example
\`\`\`jsx
// Example usage with UX considerations
<${component.name} 
  ${component.uxProps.map(prop => `/* UX: ${prop.description} */`).join('\n  ')}
/>
\`\`\`

## Related Components
<!-- List related components here -->

## Changelog
- Initial UX guidelines created
`;
  }

  async extractUXPatterns() {
    // Извлечение UX паттернов из кода и документации
    const patterns = [];
    const patternFiles = await this.findFiles(
      path.join(this.options.sourceDir, 'patterns'),
      /\.md$/
    );
    
    for (const file of patternFiles) {
      const content = await fs.readFile(file, 'utf8');
      const frontmatter = matter(content);
      
      patterns.push({
        name: path.basename(file, '.md'),
        title: frontmatter.data.title || 'Untitled Pattern',
        description: frontmatter.data.description || '',
        category: frontmatter.data.category || 'general',
        components: frontmatter.data.components || [],
        accessibility: frontmatter.data.accessibility || {},
        content: frontmatter.content
      });
    }
    
    return patterns;
  }

  async generatePatternDocs(patterns) {
    await fs.mkdir(path.join(this.options.docsDir, 'patterns'), { recursive: true });
    
    for (const pattern of patterns) {
      const docContent = this.createPatternDoc(pattern);
      const docPath = path.join(
        this.options.docsDir, 
        'patterns', 
        `${pattern.name.toLowerCase()}.mdx`
      );
      
      await fs.writeFile(docPath, docContent);
    }
  }

  createPatternDoc(pattern) {
    return `---
title: "${pattern.title}"
description: "${pattern.description}"
category: "${pattern.category}"
tags: ["ux", "pattern", "design"]
components: [${pattern.components.map(c => `"${c}"`).join(', ')}]
accessibility:
  roles: [${pattern.accessibility.roles?.map(r => `"${r}"`)?.join(', ') || ''}]
  considerations: "${pattern.accessibility.considerations || ''}"
---

# ${pattern.title}

${pattern.description}

## When to Use
${pattern.content}

## Implementation
${pattern.components.length > 0 ? 
  pattern.components.map(c => `- [${c}](../components/${c.toLowerCase()}.mdx)`).join('\n') :
  'No specific components referenced'}

## Accessibility Considerations
${pattern.accessibility.considerations || 'Follow general accessibility guidelines'}

## Examples
<!-- Add visual examples here -->
`;
  }

  async generateUXDashboard() {
    const dashboardContent = this.createUXDashboard();
    const dashboardPath = path.join(this.options.docsDir, 'dashboard.mdx');
    
    await fs.writeFile(dashboardPath, dashboardContent);
  }

  createUXDashboard() {
    return `---
title: "UX Documentation Dashboard"
description: "Overview of all UX components and patterns"
tags: ["ux", "dashboard", "documentation"]
---

# UX Documentation Dashboard

## Components
<!-- Generated component list will appear here -->

## Patterns
<!-- Generated pattern list will appear here -->

## Guidelines
<!-- Generated guideline list will appear here -->

## Status
- Components Documented: [COUNT]
- Patterns Documented: [COUNT]
- Accessibility Reviewed: [PERCENTAGE]%
- Last Updated: [DATE]

## Navigation
- [Component Library](./components/)
- [Pattern Library](./patterns/)
- [Style Guide](./style-guide.mdx)
- [Accessibility Guidelines](./accessibility.mdx)
`;
  }

  async findFiles(dir, pattern) {
    const files = [];
    const items = await fs.readdir(dir, { withFileTypes: true });
    
    for (const item of items) {
      const fullPath = path.join(dir, item.name);
      
      if (item.isDirectory()) {
        files.push(...await this.findFiles(fullPath, pattern));
      } else if (pattern.test(item.name)) {
        files.push(fullPath);
      }
    }
    
    return files;
  }
}

module.exports = UXDocumentationGenerator;
```

## UX-тестирование пользовательских сценариев

### Тестирование пользовательских путей

```javascript
// user-journey-tests.js
const puppeteer = require('puppeteer');
const fs = require('fs').promises;
const path = require('path');

class UserJourneyTester {
  constructor(options = {}) {
    this.options = {
      headless: true,
      slowMo: 0,
      viewport: { width: 1280, height: 720 },
      timeout: 30000,
      retries: 2,
      journeyDefinitionsDir: './journeys',
      resultsDir: './journey-results',
      ...options
    };
    
    this.browser = null;
    this.results = {
      passed: [],
      failed: [],
      errors: []
    };
  }

  async initialize() {
    this.browser = await puppeteer.launch({
      headless: this.options.headless,
      slowMo: this.options.slowMo,
      args: ['--no-sandbox', '--disable-setuid-sandbox']
    });
    
    await fs.mkdir(this.options.resultsDir, { recursive: true });
  }

  async runAllJourneys(baseUrl) {
    const journeyFiles = await this.findJourneyFiles();
    
    for (const journeyFile of journeyFiles) {
      const journeyDefinition = await this.loadJourneyDefinition(journeyFile);
      await this.runJourney(journeyDefinition, baseUrl);
    }
    
    await this.generateJourneyReport();
    return this.results;
  }

  async findJourneyFiles() {
    const files = await fs.readdir(this.options.journeyDefinitionsDir);
    return files.filter(file => file.endsWith('.journey.json'));
  }

  async loadJourneyDefinition(filePath) {
    const content = await fs.readFile(filePath, 'utf8');
    return JSON.parse(content);
  }

  async runJourney(journey, baseUrl) {
    let page = null;
    let journeyResult = {
      name: journey.name,
      description: journey.description,
      steps: [],
      startTime: new Date(),
      endTime: null,
      status: 'running',
      error: null
    };

    try {
      page = await this.browser.newPage();
      await page.setViewport(this.options.viewport);
      
      // Установка базового URL
      const fullBaseUrl = baseUrl.endsWith('/') ? baseUrl : baseUrl + '/';
      
      for (let i = 0; i < journey.steps.length; i++) {
        const step = journey.steps[i];
        const stepResult = await this.executeStep(page, step, fullBaseUrl, i + 1);
        
        journeyResult.steps.push(stepResult);
        
        if (!stepResult.success) {
          journeyResult.status = 'failed';
          journeyResult.error = stepResult.error;
          break;
        }
      }
      
      if (journeyResult.status !== 'failed') {
        journeyResult.status = 'passed';
      }
    } catch (error) {
      journeyResult.status = 'error';
      journeyResult.error = error.message;
      this.results.errors.push(journeyResult);
    } finally {
      journeyResult.endTime = new Date();
      if (page) {
        await page.close();
      }
      
      if (journeyResult.status === 'passed') {
        this.results.passed.push(journeyResult);
      } else if (journeyResult.status === 'failed') {
        this.results.failed.push(journeyResult);
      }
      
      // Сохранение результатов шага
      await this.saveJourneyResult(journeyResult);
    }
  }

  async executeStep(page, step, baseUrl, stepNumber) {
    const stepResult = {
      stepNumber,
      action: step.action,
      target: step.target,
      startTime: new Date(),
      endTime: null,
      success: false,
      error: null,
      screenshot: null
    };

    try {
      await page.setDefaultTimeout(this.options.timeout);
      
      switch (step.action.toLowerCase()) {
        case 'navigate':
          await page.goto(new URL(step.target, baseUrl).href);
          break;
          
        case 'click':
          await page.waitForSelector(step.target, { timeout: 5000 });
          await page.click(step.target);
          break;
          
        case 'fill':
          await page.waitForSelector(step.target, { timeout: 5000 });
          await page.type(step.target, step.value);
          break;
          
        case 'select':
          await page.waitForSelector(step.target, { timeout: 5000 });
          await page.select(step.target, step.value);
          break;
          
        case 'wait':
          if (step.condition === 'element') {
            await page.waitForSelector(step.target, { timeout: step.timeout || 5000 });
          } else if (step.condition === 'navigation') {
            await page.waitForNavigation({ timeout: step.timeout || 5000 });
          } else {
            await page.waitForTimeout(step.timeout || 1000);
          }
          break;
          
        case 'assert':
          await this.performAssertion(page, step);
          break;
          
        case 'screenshot':
          const screenshotPath = path.join(
            this.options.resultsDir,
            `screenshot-${stepNumber}-${Date.now()}.png`
          );
          await page.screenshot({ path: screenshotPath });
          stepResult.screenshot = screenshotPath;
          break;
          
        default:
          throw new Error(`Unknown action: ${step.action}`);
      }
      
      stepResult.success = true;
    } catch (error) {
      stepResult.error = error.message;
      stepResult.success = false;
      
      // Сделать скриншот при ошибке
      const errorScreenshotPath = path.join(
        this.options.resultsDir,
        `error-screenshot-${stepNumber}-${Date.now()}.png`
      );
      try {
        await page.screenshot({ path: errorScreenshotPath });
        stepResult.screenshot = errorScreenshotPath;
      } catch (screenshotError) {
        console.warn('Could not take screenshot on error:', screenshotError.message);
      }
    } finally {
      stepResult.endTime = new Date();
    }
    
    return stepResult;
  }

  async performAssertion(page, assertion) {
    switch (assertion.type.toLowerCase()) {
      case 'text':
        const element = await page.$(assertion.target);
        const text = await page.evaluate(el => el.textContent, element);
        if (assertion.operator === 'contains') {
          if (!text.includes(assertion.value)) {
            throw new Error(`Text does not contain "${assertion.value}", got: "${text}"`);
          }
        } else if (assertion.operator === 'equals') {
          if (text.trim() !== assertion.value) {
            throw new Error(`Text does not equal "${assertion.value}", got: "${text.trim()}"`);
          }
        }
        break;
        
      case 'visible':
        await page.waitForSelector(assertion.target, { timeout: 5000 });
        break;
        
      case 'notvisible':
        await page.waitForSelector(assertion.target, { 
          timeout: 1000,
          state: 'detached'
        }).catch(() => {
          // Не ошибка, если элемент не найден
        });
        break;
        
      case 'enabled':
        const isEnabled = await page.$eval(assertion.target, el => !el.disabled);
        if (!isEnabled) {
          throw new Error(`Element ${assertion.target} is not enabled`);
        }
        break;
        
      case 'value':
        const value = await page.$eval(assertion.target, el => el.value);
        if (value !== assertion.value) {
          throw new Error(`Value does not match "${assertion.value}", got: "${value}"`);
        }
        break;
        
      default:
        throw new Error(`Unknown assertion type: ${assertion.type}`);
    }
  }

  async saveJourneyResult(result) {
    const fileName = `${result.name.replace(/\s+/g, '-').toLowerCase()}-${Date.now()}.json`;
    const filePath = path.join(this.options.resultsDir, fileName);
    
    await fs.writeFile(filePath, JSON.stringify(result, null, 2));
  }

  async generateJourneyReport() {
    const report = {
      summary: {
        total: this.results.passed.length + this.results.failed.length + this.results.errors.length,
        passed: this.results.passed.length,
        failed: this.results.failed.length,
        errors: this.results.errors.length,
        passRate: (this.results.passed.length / 
                  (this.results.passed.length + this.results.failed.length + this.results.errors.length)) * 100
      },
      details: {
        passed: this.results.passed.map(r => ({
          name: r.name,
          duration: r.endTime - r.startTime
        })),
        failed: this.results.failed.map(r => ({
          name: r.name,
          error: r.error,
          failedStep: r.steps.find(s => !s.success)?.stepNumber
        })),
        errors: this.results.errors.map(r => ({
          name: r.name,
          error: r.error
        }))
      },
      timestamp: new Date().toISOString()
    };

    const reportPath = path.join(this.options.resultsDir, 'journey-report.json');
    await fs.writeFile(reportPath, JSON.stringify(report, null, 2));
    
    // Также создать читаемый отчет
    const readableReport = this.generateReadableReport(report);
    const readableReportPath = path.join(this.options.resultsDir, 'journey-report.md');
    await fs.writeFile(readableReportPath, readableReport);
    
    return report;
  }

  generateReadableReport(report) {
    return `# User Journey Test Report

## Summary
- Total Journeys: ${report.summary.total}
- Passed: ${report.summary.passed}
- Failed: ${report.summary.failed}
- Errors: ${report.summary.errors}
- Pass Rate: ${report.summary.passRate.toFixed(2)}%

## Details

### Passed Journeys (${report.summary.passed})
${report.details.passed.map(j => `- ${j.name} (${Math.round(j.duration/1000)}s)`).join('\n')}

### Failed Journeys (${report.summary.failed})
${report.details.failed.map(j => `- ${j.name}: ${j.error} (Failed at step ${j.failedStep})`).join('\n')}

### Error Journeys (${report.summary.errors})
${report.details.errors.map(j => `- ${j.name}: ${j.error}`).join('\n')}

## Recommendations
${this.generateRecommendations(report)}

---
Report generated at: ${report.timestamp}
`;
  }

  generateRecommendations(report) {
    const recommendations = [];

    if (report.summary.passRate < 80) {
      recommendations.push('⚠️ **Critical**: Pass rate below 80%. Investigate failing journeys immediately.');
    } else if (report.summary.passRate < 95) {
      recommendations.push('⚠️ **Warning**: Pass rate could be improved. Review failing journeys.');
    }

    if (report.summary.errors.length > 0) {
      recommendations.push('🐛 **Action Needed**: Address setup/teardown errors in journey tests.');
    }

    if (report.details.failed.some(f => f.failedStep === 1)) {
      recommendations.push('🔍 **Investigate**: Multiple journeys failing at navigation step - possible environment issue.');
    }

    return recommendations.length > 0 ? recommendations.join('\n') : '✅ All journeys passing!';
  }

  async close() {
    if (this.browser) {
      await this.browser.close();
    }
  }
}

// Пример определения пользовательского пути
const exampleJourney = {
  name: "User Registration Flow",
  description: "Complete user registration process",
  steps: [
    {
      action: "navigate",
      target: "/register"
    },
    {
      action: "fill",
      target: "#firstName",
      value: "John"
    },
    {
      action: "fill",
      target: "#lastName", 
      value: "Doe"
    },
    {
      action: "fill",
      target: "#email",
      value: "john.doe@example.com"
    },
    {
      action: "fill",
      target: "#password",
      value: "SecurePassword123"
    },
    {
      action: "click",
      target: "#acceptTerms"
    },
    {
      action: "click", 
      target: "#registerButton"
    },
    {
      action: "wait",
      condition: "navigation"
    },
    {
      action: "assert",
      type: "text",
      target: ".success-message",
      operator: "contains",
      value: "Registration successful"
    }
  ]
};

module.exports = UserJourneyTester;
```

## Лучшие практики UX в CI/CD

### Планирование UX-тестов

- Определение ключевых пользовательских путей
- Создание автоматизированных тестов для критических сценариев
- Регулярное обновление тестов при изменении интерфейса
- Интеграция с системами аналитики пользователей

### Мониторинг UX-метрик

- Отслеживание производительности загрузки страниц
- Мониторинг коэффициента отказов
- Анализ времени взаимодействия пользователей
- Сбор обратной связи от пользователей

### Интеграция с командной работой

- Совместное планирование UX и технических задач
- Регулярные ревью интерфейсов
- Обратная связь от команды поддержки
- Вовлечение UX-дизайнеров в процесс CI/CD

## Заключение

Пользовательский опыт в CI/CD обеспечивает, что изменения программного обеспечения не только технически корректны, но и соответствуют ожиданиям пользователей. Это критически важно для создания успешных цифровых продуктов, которые приносят пользу пользователям и бизнесу.

Успешная интеграция UX в CI/CD требует:

- Понимания принципов UX-дизайна
- Автоматизации UX-тестов
- Мониторинга пользовательского опыта
- Сотрудничества между командами

Организации, инвестирующие в UX в CI/CD, получают конкурентное преимущество за счет более удовлетворенных пользователей и более высокой эффективности своих продуктов.

## Связанные концепции

- [[User Interface Design]]
- [[Accessibility Standards]]
- [[A/B Testing]]
- [[Product Analytics]]
- [[Continuous Integration]]
- [[Continuous Deployment]]
- [[Quality Assurance]]
- [[Design Systems]]

## Внешние ресурсы

- [Nielsen Norman Group UX Guidelines](https://www.nngroup.com/)
- [Web Content Accessibility Guidelines (WCAG)](https://www.w3.org/WAI/standards-guidelines/wcag/)
- [Google UX Design Courses](https://design.google/design-resources/)
- [Material Design Guidelines](https://material.io/design)
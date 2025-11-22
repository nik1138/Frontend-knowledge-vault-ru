---
aliases: ["Метрики тестирования", "Testing Metrics", "Test Coverage", "Code Coverage"]
tags: 
  - testing
  - quality-assurance
  - ci-cd
  - metrics
  - frontend
---

# Метрики тестирования в архитектуре приложений

## Введение

Метрики тестирования - это количественные показатели, используемые для оценки качества, полноты и эффективности тестирования программного обеспечения. Эти метрики помогают командам разработчиков принимать обоснованные решения о состоянии качества кода, эффективности тестов и прогрессе в улучшении качества приложения.

## Основные метрики тестирования

### 1. Покрытие кода (Code Coverage)

Покрытие кода - это процент строк, ветвей или функций в коде, которые были выполнены во время тестирования. Это одна из наиболее распространенных метрик, хотя и не всегда отражает реальное качество тестирования.

**Типы покрытия:**
- **Покрытие строк (Line Coverage)**: Процент строк кода, выполненных во время тестирования
- **Покрытие ветвей (Branch Coverage)**: Процент возможных ветвлений кода, пройденных тестами
- **Покрытие функций (Function Coverage)**: Процент функций, вызванных во время тестирования
- **Покрытие условий (Condition Coverage)**: Процент логических условий, проверенных тестами

```javascript
// Пример измерения покрытия кода с использованием Jest
// package.json
{
  "scripts": {
    "test:coverage": "jest --coverage",
    "test:coverage:ci": "jest --coverage --coverageReporters=text-lcov | coveralls"
  }
}

// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.js',
    '!src/reportWebVitals.js'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 85,
      lines: 90,
      statements: 90
    }
  },
  coverageReporters: [
    'json',
    'text',
    'lcov',
    'clover'
  ]
};

// Пример кода с разными уровнями покрытия
function calculateDiscount(price, discountType) {
  if (discountType === 'student') {
    return price * 0.1;
  } else if (discountType === 'senior') {
    return price * 0.15;
  } else if (price > 100) {
    return price * 0.05;
  }
  return 0;
}

// Тесты, обеспечивающие полное покрытие
describe('calculateDiscount', () => {
  test('should return 10% discount for student', () => {
    expect(calculateDiscount(100, 'student')).toBe(10);
  });
  
  test('should return 15% discount for senior', () => {
    expect(calculateDiscount(100, 'senior')).toBe(15);
  });
  
  test('should return 5% discount for orders over 100', () => {
    expect(calculateDiscount(150, 'regular')).toBe(7.5);
  });
  
  test('should return 0 for regular orders under 100', () => {
    expect(calculateDiscount(50, 'regular')).toBe(0);
  });
});
```

### 2. Плотность дефектов (Defect Density)

Плотность дефектов - это количество обнаруженных дефектов, деленное на размер программного обеспечения (обычно измеряется в строках кода или функциональных точках). Эта метрика помогает оценить качество кода и эффективность процесса разработки.

### 3. Время нахождения дефекта (Defect Detection Time)

Время нахождения дефекта - это продолжительность времени между появлением дефекта и его обнаружением. Чем раньше дефект обнаружен, тем дешевле его устранение.

### 4. Процент прохождения тестов (Test Pass Rate)

Процент прохождения тестов - это отношение количества успешно пройденных тестов к общему количеству выполненных тестов. Эта метрика показывает стабильность кода и эффективность тестов.

```javascript
// Пример отчета о тестах с использованием Jest
const testResults = {
  numTotalTests: 150,
  numPassedTests: 142,
  numFailedTests: 5,
  numPendingTests: 3,
  testPassRate: 94.7 // (142 / 150) * 100
};

// Пример настройки пороговых значений для прохождения тестов
// jest.config.js
module.exports = {
  testResultsProcessor: '<rootDir>/test-results-processor.js',
  // Установка минимального порога прохождения тестов
  collectCoverageFrom: ['src/**/*.{js,jsx,ts,tsx}'],
  coverageThreshold: {
    global: {
      lines: 90,
      branches: 80,
      functions: 85,
      statements: 90
    }
  }
};
```

### 5. Время выполнения тестов (Test Execution Time)

Время выполнения тестов - это продолжительность времени, необходимая для выполнения всех тестов. В контексте CI/CD важно, чтобы время выполнения тестов не увеличивалось экспоненциально, так как это замедляет цикл разработки.

## Метрики, специфичные для разных типов тестирования

### Unit тестирование

- Покрытие кода (Code Coverage)
- Количество unit тестов
- Время выполнения unit тестов
- Процент прохождения unit тестов

### Интеграционное тестирование

- Количество интеграционных тестов
- Время выполнения интеграционных тестов
- Процент прохождения интеграционных тестов
- Покрытие интеграционных точек

### End-to-End тестирование

- Количество E2E тестов
- Время выполнения E2E тестов
- Процент прохождения E2E тестов
- Надежность E2E тестов (процент стабильного выполнения)

## Инструменты для измерения метрик

### 1. Istanbul (nyc)

Istanbul - это фреймворк для измерения покрытия кода в JavaScript-приложениях.

```bash
# Установка Istanbul
npm install --save-dev nyc

# Измерение покрытия при выполнении тестов
nyc npm test

# Генерация отчетов о покрытии
nyc report --reporter=html --reporter=text
```

### 2. Coveralls

Coveralls - сервис для отслеживания покрытия кода с течением времени.

```yaml
# .travis.yml
after_success:
  - 'cat coverage/lcov.info | node_modules/.bin/coveralls'
```

### 3. SonarQube

SonarQube - платформа для непрерывного анализа качества кода.

```xml
<!-- pom.xml для Maven проекта -->
<plugin>
  <groupId>org.sonarsource.scanner.maven</groupId>
  <artifactId>sonar-maven-plugin</artifactId>
  <version>3.9.1.2184</version>
</plugin>

<!-- Выполнение анализа -->
mvn clean verify sonar:sonar
```

## Метрики в контексте CI/CD

В контексте CI/CD метрики тестирования играют важную роль в принятии решений о развертывании:

- **Пороги качества**: Автоматические проверки, блокирующие развертывание при недостаточном покрытии
- **Сравнение метрик**: Сравнение метрик между разными версиями приложения
- **Мониторинг трендов**: Отслеживание изменений метрик с течением времени

```yaml
# Пример GitHub Actions с проверкой метрик
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm ci
      
    - name: Run tests with coverage
      run: npm run test:coverage
      
    - name: Check coverage thresholds
      run: |
        npx jest --coverage --coverageThreshold='{"global":{"branches":80,"functions":85,"lines":90,"statements":90}}'
        
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
```

## Интерпретация метрик

### Высокое покрытие кода ≠ Высокое качество

Важно понимать, что высокий процент покрытия кода не всегда означает высокое качество тестов. Тесты должны быть значимыми и проверять логику приложения, а не просто "покрывать" строки кода.

### Баланс между метриками

Необходимо стремиться к балансу между различными метриками:
- Высокое покрытие кода
- Высокий процент прохождения тестов
- Приемлемое время выполнения тестов
- Минимальное количество ложных срабатываний

## Заключение

Метрики тестирования являются важным инструментом для оценки и улучшения качества программного обеспечения. Они помогают командам разработчиков принимать обоснованные решения, выявлять области, требующие улучшения, и отслеживать прогресс в обеспечении качества.

Важно использовать метрики в сочетании с другими практиками обеспечения качества, такими как code review, статический анализ кода и мониторинг приложений в продакшене. Только комплексный подход позволяет достичь высокого уровня качества программного обеспечения.

[[Testing Principles]] - Основные принципы тестирования
[[Unit Testing]] - Подробнее о модульном тестировании
[[CI/CD]] - Интеграция метрик тестирования в CI/CD процессы
[[Testing Architecture]] - Архитектура тестирования в приложениях
---
aliases: [Coverage, Test Coverage, Покрытие тестами]
tags: [testing, quality, metrics]
---

# Покрытие тестами (Test Coverage): Метрики и Стратегии

## Обзор

Покрытие тестами (test coverage) - это метрика, которая измеряет процент кода, который был выполнен во время тестирования. Это важный индикатор качества кода и эффективности тестирования. Высокое покрытие тестами не гарантирует отсутствие багов, но указывает на то, что большая часть кода была проверена.

## Метрики покрытия тестами

### 1. Покрытие строк (Line Coverage)

Измеряет процент строк кода, которые были выполнены во время тестирования.

```javascript
// Пример кода с разным покрытием строк
function calculateDiscount(price, discount) {
  if (discount > 0) {  // строка 1
    return price - (price * discount / 100);  // строка 2
  }
  return price;  // строка 3
}

// Тест, который дает 66% покрытия строк:
test('calculateDiscount без скидки', () => {
  expect(calculateDiscount(100, 0)).toBe(100);
});
// Покрыты строки 1 и 3, строка 2 не покрыта
```

### 2. Покрытие ветвлений (Branch Coverage)

Измеряет процент условных ветвлений, которые были выполнены.

```javascript
// Пример для покрытия ветвлений
function validateUser(user) {
  if (user && user.age) {  // ветвь 1
    if (user.age >= 18) {  // ветвь 2
      return 'adult';
    } else {  // ветвь 3
      return 'minor';
    }
  } else {  // ветвь 4
    return 'invalid';
  }
}

// Для 100% покрытия ветвлений необходимы 4 теста:
test('validateUser - взрослый', () => {
  expect(validateUser({ age: 25 })).toBe('adult');
});

test('validateUser - несовершеннолетний', () => {
  expect(validateUser({ age: 15 })).toBe('minor');
});

test('validateUser - недопустимый пользователь', () => {
  expect(validateUser(null)).toBe('invalid');
});

test('validateUser - пользователь без возраста', () => {
  expect(validateUser({})).toBe('invalid');
});
```

### 3. Покрытие функций (Function Coverage)

Измеряет процент функций, которые были вызваны во время тестирования.

### 4. Покрытие операторов (Statement Coverage)

Измеряет процент исполняемых операторов, которые были выполнены.

## Инструменты для измерения покрытия

### Istanbul/NYC (JavaScript)

```json
// package.json
{
  "scripts": {
    "test:coverage": "nyc --reporter=html --reporter=text mocha",
    "test:check": "nyc check-coverage --lines 80 --functions 80 --branches 80"
  },
  "devDependencies": {
    "nyc": "^15.1.0"
  }
}
```

```javascript
// .nycrc
{
  "extends": "@istanbuljs/nyc-config-typescript",
  "all": true,
  "include": [
    "src/**/*.js",
    "src/**/*.ts"
  ],
  "exclude": [
    "**/node_modules/**",
    "**/test/**",
    "**/tests/**"
  ],
  "reporter": [
    "text",
    "html",
    "lcov"
  ],
  "check-coverage": true,
  "lines": 80,
  "functions": 80,
  "branches": 70,
  "statements": 80
}
```

### Jest с покрытием

```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/mocks/**',
    '!src/test-utils/**'
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### Pytest-cov (Python)

```bash
# Установка
pip install pytest-cov

# Запуск с покрытием
pytest --cov=src --cov-report=html --cov-report=term
```

## Стратегии измерения покрытия

### 1. Базовая стратегия

Начните с простого измерения общего покрытия строк и функций:

```javascript
// Пример конфигурации для начального уровня
const coverageConfig = {
  threshold: {
    global: {
      lines: 70,      // Минимум 70% покрытия строк
      functions: 70,  // Минимум 70% покрытия функций
      branches: 60,   // Минимум 60% покрытия ветвлений
      statements: 70  // Минимум 70% покрытия операторов
    }
  }
};
```

### 2. Пофайловая стратегия

Установка порогов покрытия для отдельных файлов:

```javascript
// jest.config.js - пофайловая стратегия
module.exports = {
  coverageThreshold: {
    './src/core/business-logic.js': {
      branches: 90,
      functions: 95,
      lines: 95,
      statements: 95
    },
    './src/utils/helpers.js': {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  }
};
```

### 3. Стратифицированная стратегия

Разные пороги для разных типов кода:

```javascript
// Пример стратифицированной стратегии
const coverageStrategy = {
  businessLogic: { lines: 95, branches: 90, functions: 95 },
  utils: { lines: 80, branches: 70, functions: 80 },
  components: { lines: 85, branches: 75, functions: 85 },
  configuration: { lines: 50, branches: 30, functions: 50 }
};
```

## Практические примеры

### Пример 1: Покрытие компонента React

```jsx
// Button.jsx
import React from 'react';

const Button = ({ onClick, disabled, children, variant = 'primary' }) => {
  const handleClick = () => {
    if (!disabled && onClick) {
      onClick();
    }
  };

  const buttonClass = `btn btn-${variant}${disabled ? ' disabled' : ''}`;

  return (
    <button className={buttonClass} onClick={handleClick} disabled={disabled}>
      {children}
    </button>
  );
};

export default Button;
```

```jsx
// Button.test.jsx
import { render, fireEvent, screen } from '@testing-library/react';
import Button from './Button';

describe('Button Component', () => {
  test('рендерит дочерние элементы', () => {
    render(<Button>Нажми меня</Button>);
    expect(screen.getByText('Нажми меня')).toBeInTheDocument();
  });

  test('вызывает onClick при клике', () => {
    const mockOnClick = jest.fn();
    render(<Button onClick={mockOnClick}>Кнопка</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(mockOnClick).toHaveBeenCalledTimes(1);
  });

  test('не вызывает onClick когда disabled', () => {
    const mockOnClick = jest.fn();
    render(
      <Button onClick={mockOnClick} disabled>
        Кнопка
      </Button>
    );
    
    fireEvent.click(screen.getByRole('button'));
    expect(mockOnClick).toHaveBeenCalledTimes(0);
  });

  test('применяет правильный CSS класс', () => {
    const { container } = render(
      <Button variant="secondary">Кнопка</Button>
    );
    
    expect(container.firstChild).toHaveClass('btn-secondary');
  });

  test('устанавливает disabled атрибут', () => {
    render(
      <Button disabled>Кнопка</Button>
    );
    
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

### Пример 2: Покрытие бизнес-логики

```javascript
// discountCalculator.js
export class DiscountCalculator {
  static calculate(itemPrice, quantity, customerType) {
    let discount = 0;
    
    // Проверка количества
    if (quantity >= 10) {
      discount = 0.1; // 10% скидка
    } else if (quantity >= 5) {
      discount = 0.05; // 5% скидка
    }
    
    // Дополнительная скидка для VIP клиентов
    if (customerType === 'VIP') {
      discount += 0.05; // Дополнительные 5%
    }
    
    // Максимальная скидка 20%
    discount = Math.min(discount, 0.2);
    
    const discountAmount = itemPrice * quantity * discount;
    const finalPrice = itemPrice * quantity - discountAmount;
    
    return {
      originalPrice: itemPrice * quantity,
      discount: discount * 100,
      discountAmount,
      finalPrice
    };
  }
}
```

```javascript
// discountCalculator.test.js
import { DiscountCalculator } from './discountCalculator';

describe('DiscountCalculator', () => {
  test('рассчитывает без скидки для малого количества', () => {
    const result = DiscountCalculator.calculate(100, 2, 'regular');
    expect(result).toEqual({
      originalPrice: 200,
      discount: 0,
      discountAmount: 0,
      finalPrice: 200
    });
  });

  test('рассчитывает 5% скидку для 5-9 единиц', () => {
    const result = DiscountCalculator.calculate(100, 5, 'regular');
    expect(result).toEqual({
      originalPrice: 500,
      discount: 5,
      discountAmount: 25,
      finalPrice: 475
    });
  });

  test('рассчитывает 10% скидку для 10+ единиц', () => {
    const result = DiscountCalculator.calculate(100, 10, 'regular');
    expect(result).toEqual({
      originalPrice: 1000,
      discount: 10,
      discountAmount: 100,
      finalPrice: 900
    });
  });

  test('рассчитывает дополнительную скидку для VIP', () => {
    const result = DiscountCalculator.calculate(100, 5, 'VIP');
    expect(result).toEqual({
      originalPrice: 500,
      discount: 10, // 5% + 5% для VIP
      discountAmount: 50,
      finalPrice: 450
    });
  });

  test('ограничивает максимальную скидку 20%', () => {
    const result = DiscountCalculator.calculate(100, 50, 'VIP');
    expect(result).toEqual({
      originalPrice: 5000,
      discount: 20, // Ограничен до 20% несмотря на 25% (20% + 5% для VIP)
      discountAmount: 1000,
      finalPrice: 4000
    });
  });
});
```

## Интеграция с CI/CD

### GitHub Actions

```yaml
name: Test Coverage

on: [push, pull_request]

jobs:
  coverage:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Установка Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16'
    
    - name: Установка зависимостей
      run: npm ci
    
    - name: Запуск тестов с покрытием
      run: npm run test:coverage
    
    - name: Отправка покрытия в Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella
        fail_ci_if_error: true
```

### Проверка порога покрытия

```bash
# Скрипт проверки покрытия
#!/bin/bash
echo "Проверка покрытия тестами..."

# Запуск тестов с покрытием
npm run test:coverage

# Проверка, что покрытие не ниже порога
if [ $? -ne 0 ]; then
  echo "Ошибка: Тесты не прошли"
  exit 1
fi

# Проверка покрытия строк
lines_coverage=$(nyc report --reporter=text | grep "Lines" | awk '{print $2}' | sed 's/%//')
if (( $(echo "$lines_coverage < 80" | bc -l) )); then
  echo "Ошибка: Покрытие строк ниже 80% ($lines_coverage%)"
  exit 1
fi

echo "Покрытие тестами в пределах нормы: $lines_coverage%"
```

## Лучшие практики

### 1. Установка реалистичных порогов

- Не стремитесь к 100% покрытию сразу
- Начните с 70-80% и постепенно повышайте
- Учитывайте тип кода при установке порогов

### 2. Фокус на качество, а не на количество

- Покрытие не гарантирует качество тестов
- Пишите осмысленные тесты, а не просто для покрытия
- Проверяйте граничные условия и сценарии ошибок

### 3. Использование инструментов визуализации

```javascript
// Конфигурация для визуализации покрытия
module.exports = {
  collectCoverage: true,
  coverageReporters: ['html', 'text-summary', 'lcov'],
  coverageDirectory: 'coverage',
  coveragePathIgnorePatterns: [
    '/node_modules/',
    '/test/',
    '/tests/',
    '\\.(test|spec)\\.js$'
  ]
};
```

### 4. Отслеживание изменений покрытия

- Используйте сервисы типа Codecov или Coveralls
- Сравнивайте покрытие между коммитами
- Получайте уведомления о снижении покрытия

## Заключение

Покрытие тестами - важная метрика качества кода, но не следует стремиться к 100% покрытию любой ценой. Важно понимать, что высокое покрытие не гарантирует отсутствие багов, но указывает на то, что код был проверен. Лучшая практика - это сочетание адекватного покрытия с качественными тестами, которые проверяют реальные сценарии использования и граничные условия.

Связанные концепции: [[Testing Strategies]], [[Code Quality]], [[CI/CD Pipeline]], [[Quality Metrics]]
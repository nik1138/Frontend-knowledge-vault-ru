---
aliases: ["Регулярные выражения в тестировании", "Тестирование с регулярками", "Unit и e2e тесты"]
tags: ["#regexp", "#testing", "#unit-tests", "#e2e", "#jest", "#cypress", "#frontend"]
---

# Регулярные выражения для тестирования (unit и e2e тесты)

## Введение

Регулярные выражения находят широкое применение в тестировании программного обеспечения, особенно при unit и e2e тестировании. Они позволяют проверять форматы данных, искать определенные паттерны в строках, валидировать ответы API и проверять корректность отображения информации в пользовательском интерфейсе.

## Unit тестирование

### 1. Проверка форматов данных

Регулярные выражения часто используются в unit тестах для проверки корректности форматов данных:

```javascript
// utils/validation.js
export const validateEmail = (email) => {
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  return emailRegex.test(email);
};

export const validatePhone = (phone) => {
  const phoneRegex = /^(\+7|7|8)?[\s-]?\(?[0-9]{3}\)?[\s-]?[0-9]{3}[\s-]?[0-9]{2}[\s-]?[0-9]{2}$/;
  return phoneRegex.test(phone);
};

export const validatePassword = (password) => {
  const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
  return passwordRegex.test(password);
};
```

```javascript
// __tests__/validation.test.js
import { validateEmail, validatePhone, validatePassword } from '../utils/validation';

describe('Валидация данных', () => {
  describe('validateEmail', () => {
    test('должен вернуть true для корректного email', () => {
      expect(validateEmail('user@example.com')).toBe(true);
      expect(validateEmail('test.user+tag@domain.co.uk')).toBe(true);
    });

    test('должен вернуть false для некорректного email', () => {
      expect(validateEmail('invalid-email')).toBe(false);
      expect(validateEmail('@example.com')).toBe(false);
      expect(validateEmail('user@')).toBe(false);
    });
  });

  describe('validatePhone', () => {
    test('должен вернуть true для корректного российского номера телефона', () => {
      expect(validatePhone('+79991234567')).toBe(true);
      expect(validatePhone('8 (999) 123-45-67')).toBe(true);
      expect(validatePhone('79991234567')).toBe(true);
    });

    test('должен вернуть false для некорректного номера телефона', () => {
      expect(validatePhone('12345')).toBe(false);
      expect(validatePhone('not-a-phone')).toBe(false);
    });
  });

  describe('validatePassword', () => {
    test('должен вернуть true для надежного пароля', () => {
      expect(validatePassword('SecurePass123!')).toBe(true);
      expect(validatePassword('AnotherValidP@ss9')).toBe(true);
    });

    test('должен вернуть false для слабого пароля', () => {
      expect(validatePassword('weak')).toBe(false);
      expect(validatePassword('NoNumbers')).toBe(false);
      expect(validatePassword('nouppercase123')).toBe(false);
    });
  });
});
```

### 2. Проверка форматов дат и времени

```javascript
// utils/date.js
export const validateDateFormat = (dateString) => {
  const dateRegex = /^\d{4}-\d{2}-\d{2}$/;
  return dateRegex.test(dateString);
};

export const validateDateTimeFormat = (dateTimeString) => {
  const dateTimeRegex = /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(?:\.\d{3})?Z?$/;
  return dateTimeRegex.test(dateTimeString);
};

export const validateTimeFormat = (timeString) => {
  const timeRegex = /^([01]?[0-9]|2[0-3]):[0-5][0-9]$/;
  return timeRegex.test(timeString);
};
```

```javascript
// __tests__/date.test.js
import { validateDateFormat, validateDateTimeFormat, validateTimeFormat } from '../utils/date';

describe('Валидация дат и времени', () => {
  test('validateDateFormat', () => {
    expect(validateDateFormat('2023-12-25')).toBe(true);
    expect(validateDateFormat('2023-1-1')).toBe(false); // Неправильный формат
    expect(validateDateFormat('25-12-2023')).toBe(false); // Неправильный формат
  });

  test('validateDateTimeFormat', () => {
    expect(validateDateTimeFormat('2023-12-25T10:30:00Z')).toBe(true);
    expect(validateDateTimeFormat('2023-12-25T10:30:00.123Z')).toBe(true);
    expect(validateDateTimeFormat('2023-12-25 10:30:00')).toBe(false);
  });

  test('validateTimeFormat', () => {
    expect(validateTimeFormat('10:30')).toBe(true);
    expect(validateTimeFormat('23:59')).toBe(true);
    expect(validateTimeFormat('00:00')).toBe(true);
    expect(validateTimeFormat('25:00')).toBe(false); // Неправильное время
    expect(validateTimeFormat('10:60')).toBe(false); // Неправильные минуты
  });
});
```

### 3. Проверка URL и путей

```javascript
// utils/url.js
export const validateUrl = (url) => {
  const urlRegex = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/;
  return urlRegex.test(url);
};

export const validateRelativePath = (path) => {
  const pathRegex = /^\/[\w\-./]*$/;
  return pathRegex.test(path);
};

export const extractDomain = (url) => {
  const domainRegex = /https?:\/\/(?:[-\w.])+(?:\:[0-9]+)?/;
  const match = url.match(domainRegex);
  return match ? match[0] : null;
};
```

```javascript
// __tests__/url.test.js
import { validateUrl, validateRelativePath, extractDomain } from '../utils/url';

describe('Валидация URL', () => {
  test('validateUrl', () => {
    expect(validateUrl('https://example.com')).toBe(true);
    expect(validateUrl('http://subdomain.example.com/path')).toBe(true);
    expect(validateUrl('not-a-url')).toBe(false);
  });

  test('validateRelativePath', () => {
    expect(validateRelativePath('/path/to/resource')).toBe(true);
    expect(validateRelativePath('/')).toBe(true);
    expect(validateRelativePath('/path/with-dash_and_underscore')).toBe(true);
    expect(validateRelativePath('http://example.com')).toBe(false); // Не относительный путь
  });

  test('extractDomain', () => {
    expect(extractDomain('https://example.com/page')).toBe('https://example.com');
    expect(extractDomain('http://subdomain.example.com:8080/path')).toBe('http://subdomain.example.com:8080');
    expect(extractDomain('not-a-url')).toBeNull();
  });
});
```

## E2E тестирование

### 1. Использование регулярных выражений в Cypress

```javascript
// cypress/e2e/form-validation.cy.js
describe('Валидация формы регистрации', () => {
  beforeEach(() => {
    cy.visit('/registration');
  });

  it('должна отображать ошибки при некорректном email', () => {
    cy.get('[data-cy="email-input"]').type('invalid-email');
    cy.get('[data-cy="submit-button"]').click();
    
    // Проверка сообщения об ошибке с использованием регулярного выражения
    cy.get('[data-cy="error-message"]')
      .should('contain.text', 'Некорректный email')
      .and('match', /.*[Ee]mail.*/); // Проверяем, что сообщение содержит слово "email" (без учета регистра)
  });

  it('должна проверять формат пароля', () => {
    // Ввод слабого пароля
    cy.get('[data-cy="password-input"]').type('weak');
    cy.get('[data-cy="password-input"]').blur();
    
    // Проверка требования к паролю
    cy.get('[data-cy="password-requirements"]')
      .should('contain.text', '8 символов')
      .and('contain.text', 'заглавная буква')
      .and('contain.text', 'цифра')
      .and('contain.text', 'специальный символ');
  });

  it('должна проверять динамическое обновление поля', () => {
    cy.get('[data-cy="username-input"]').type('testuser');
    
    // Проверка формата сгенерированного slug
    cy.get('[data-cy="slug-preview"]')
      .invoke('text')
      .should('match', /^[a-z0-9-]+$/); // Только строчные буквы, цифры и дефисы
  });
});
```

### 2. Проверка содержимого API ответов

```javascript
// cypress/e2e/api-validation.cy.js
describe('Валидация API ответов', () => {
  it('должна проверять формат данных пользователя', () => {
    cy.request('/api/users/1').then((response) => {
      expect(response.status).to.eq(200);
      
      const userData = response.body;
      
      // Проверка формата email
      expect(userData.email).to.match(/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/);
      
      // Проверка формата даты регистрации
      expect(userData.createdAt).to.match(/^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}/);
      
      // Проверка формата UUID
      expect(userData.id).to.match(/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i);
    });
  });

  it('должна проверять формат URL в ответе', () => {
    cy.request('/api/products').then((response) => {
      const product = response.body[0];
      
      // Проверка формата URL изображения
      expect(product.imageUrl).to.match(/^https:\/\/.*\.(jpg|jpeg|png|gif|webp)$/i);
      
      // Проверка формата SKU
      expect(product.sku).to.match(/^[A-Z0-9-]+$/);
    });
  });
});
```

### 3. Использование регулярных выражений в Puppeteer

```javascript
// tests/e2e/search.test.js
const puppeteer = require('puppeteer');

describe('Поиск на сайте', () => {
  let browser;
  let page;

  beforeAll(async () => {
    browser = await puppeteer.launch();
    page = await browser.newPage();
  });

  afterAll(async () => {
    await browser.close();
  });

  test('должен находить результаты поиска', async () => {
    await page.goto('https://example.com/search');
    
    // Ввод поискового запроса
    await page.type('[data-testid="search-input"]', 'JavaScript');
    await page.click('[data-testid="search-button"]');
    
    // Ожидание результатов
    await page.waitForSelector('[data-testid="search-results"]');
    
    // Получение текста результатов
    const resultsText = await page.evaluate(() => {
      const results = document.querySelectorAll('[data-testid="search-result"]');
      return Array.from(results).map(el => el.textContent);
    });
    
    // Проверка, что результаты содержат поисковый термин
    const searchTerm = /javascript/i; // Регулярное выражение без учета регистра
    const hasResults = resultsText.some(text => searchTerm.test(text));
    
    expect(hasResults).toBe(true);
  });

  test('должен проверять формат отображения цен', async () => {
    await page.goto('https://example.com/products');
    
    // Получение цен товаров
    const prices = await page.evaluate(() => {
      const priceElements = document.querySelectorAll('[data-testid="price"]');
      return Array.from(priceElements).map(el => el.textContent.trim());
    });
    
    // Проверка формата цены (например, "1 234,56 руб.")
    const priceRegex = /^\d{1,3}( \d{3})*,[0-9]{2} руб\.$/;
    const allValidPrices = prices.every(price => priceRegex.test(price));
    
    expect(allValidPrices).toBe(true);
  });
});
```

## Практические примеры

### 1. Тестирование компонента ввода email

```javascript
// __tests__/EmailInput.test.js
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import EmailInput from '../components/EmailInput';

describe('EmailInput компонент', () => {
  test('должен отображать ошибку при некорректном email', () => {
    render(<EmailInput />);
    
    const input = screen.getByRole('textbox');
    fireEvent.change(input, { target: { value: 'invalid-email' } });
    fireEvent.blur(input);
    
    expect(screen.getByText(/некорректный email/i)).toBeInTheDocument();
  });

  test('не должен отображать ошибку при корректном email', () => {
    render(<EmailInput />);
    
    const input = screen.getByRole('textbox');
    fireEvent.change(input, { target: { value: 'valid@example.com' } });
    fireEvent.blur(input);
    
    expect(screen.queryByText(/некорректный email/i)).not.toBeInTheDocument();
  });
});
```

### 2. Тестирование API сервиса

```javascript
// __tests__/apiService.test.js
import { validateApiResponse } from '../services/apiService';

describe('API сервис', () => {
  test('должен валидировать формат ответа пользователя', async () => {
    const mockResponse = {
      id: '123e4567-e89b-12d3-a456-426614174000',
      email: 'user@example.com',
      createdAt: '2023-12-25T10:30:00Z',
      profile: {
        firstName: 'Иван',
        lastName: 'Иванов'
      }
    };
    
    const validationResult = validateApiResponse(mockResponse);
    
    // Проверка формата UUID
    expect(mockResponse.id).toMatch(/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i);
    
    // Проверка формата email
    expect(mockResponse.email).toMatch(/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/);
    
    // Проверка формата даты
    expect(mockResponse.createdAt).toMatch(/^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z$/);
    
    expect(validationResult.isValid).toBe(true);
  });
});
```

### 3. Тестирование парсера логов

```javascript
// utils/logParser.js
export const parseLogLine = (line) => {
  // Регулярное выражение для парсинга логов Apache
  const logRegex = /^(\S+) (\S+) (\S+) \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+)$/;
  const match = line.match(logRegex);
  
  if (match) {
    return {
      ip: match[1],
      identity: match[2],
      user: match[3],
      timestamp: match[4],
      method: match[5],
      url: match[6],
      protocol: match[7],
      status: parseInt(match[8]),
      size: parseInt(match[9])
    };
  }
  
  return null;
};

// __tests__/logParser.test.js
import { parseLogLine } from '../utils/logParser';

describe('Парсер логов', () => {
  test('должен корректно парсить строку лога', () => {
    const logLine = '127.0.0.1 - frank [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326';
    const parsed = parseLogLine(logLine);
    
    expect(parsed).toEqual({
      ip: '127.0.0.1',
      identity: '-',
      user: 'frank',
      timestamp: '10/Oct/2000:13:55:36 -0700',
      method: 'GET',
      url: '/apache_pb.gif',
      protocol: 'HTTP/1.0',
      status: 200,
      size: 2326
    });
  });

  test('должен возвращать null для некорректной строки лога', () => {
    const invalidLogLine = 'invalid log line';
    const parsed = parseLogLine(invalidLogLine);
    
    expect(parsed).toBeNull();
  });
});
```

## Лучшие практики

### 1. Использование именованных групп захвата

```javascript
// Вместо этого:
const dateRegex = /(\d{4})-(\d{2})-(\d{2})/;

// Лучше использовать именованные группы:
const dateRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const match = dateString.match(dateRegex);
if (match) {
  const { year, month, day } = match.groups;
  // Использование именованных групп
}
```

### 2. Кэширование регулярных выражений

```javascript
// Кэширование регулярных выражений для лучшей производительности
class Validator {
  constructor() {
    this.emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    this.phoneRegex = /^(\+7|7|8)?[\s-]?\(?[0-9]{3}\)?[\s-]?[0-9]{3}[\s-]?[0-9]{2}[\s-]?[0-9]{2}$/;
    this.urlRegex = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/;
  }
  
  validateEmail(email) {
    return this.emailRegex.test(email);
  }
  
  validatePhone(phone) {
    return this.phoneRegex.test(phone);
  }
  
  validateUrl(url) {
    return this.urlRegex.test(url);
  }
}
```

### 3. Тестирование регулярных выражений

```javascript
// __tests__/regexValidation.test.js
describe('Регулярные выражения', () => {
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  
  test('emailRegex - должен соответствовать спецификации', () => {
    // Позитивные тесты
    expect(emailRegex.test('user@example.com')).toBe(true);
    expect(emailRegex.test('user.name+tag@example.co.uk')).toBe(true);
    
    // Негативные тесты
    expect(emailRegex.test('invalid-email')).toBe(false);
    expect(emailRegex.test('@example.com')).toBe(false);
    expect(emailRegex.test('user@')).toBe(false);
  });
});
```

## Заключение

Регулярные выражения являются мощным инструментом в тестировании программного обеспечения. Они позволяют эффективно проверять форматы данных, валидировать ответы API, проверять корректность отображения информации в интерфейсе и многое другое.

При использовании регулярных выражений в тестировании важно:
- Писать понятные и документированные регулярные выражения
- Покрывать тестами как позитивные, так и негативные сценарии
- Учитывать производительность при частом использовании
- Использовать именованные группы захвата для лучшей читаемости
- Кэшировать регулярные выражения при необходимости

Правильное применение регулярных выражений в тестировании помогает повысить надежность и качество программного обеспечения.
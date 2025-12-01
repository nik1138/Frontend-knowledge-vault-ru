---
aliases: [E2E тестирование, Интеграционное тестирование, Приемочное тестирование]
tags: [vue, testing, e2e, integration, cypress, playwright]
---

# E2E-тестирование

## Общие сведения

E2E (End-to-End) тестирование в Vue-приложениях - это процесс проверки полного пользовательского сценария от начала до конца. В 2025 году для E2E тестирования Vue-приложений наиболее популярны [[Cypress]] и [[Playwright]], каждый из которых имеет свои особенности и преимущества.

## Инструменты для E2E тестирования

### Cypress

[[Cypress]] - мощный фреймворк для E2E тестирования, особенно популярный в российском сообществе разработчиков.

```javascript
// cypress/e2e/login.cy.js
describe('Страница входа', () => {
  beforeEach(() => {
    cy.visit('/login')
  })

  it('позволяет пользователю войти в систему', () => {
    cy.get('[data-cy=username]').type('testuser')
    cy.get('[data-cy=password]').type('password123')
    cy.get('[data-cy=submit]').click()

    cy.url().should('include', '/dashboard')
    cy.get('[data-cy=welcome-message]').should('contain', 'Добро пожаловать')
  })

  it('показывает ошибку при неверных данных', () => {
    cy.get('[data-cy=username]').type('invalid')
    cy.get('[data-cy=password]').type('wrong')
    cy.get('[data-cy=submit]').click()

    cy.get('[data-cy=error-message]').should('be.visible')
  })
})
```

### Playwright

[[Playwright]] - современный инструмент от Microsoft, обеспечивающий высокую стабильность тестов.

```javascript
// tests/login.spec.js
import { test, expect } from '@playwright/test'

test.describe('Страница входа', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login')
  })

  test('позволяет пользователю войти в систему', async ({ page }) => {
    await page.locator('[data-testid=username]').fill('testuser')
    await page.locator('[data-testid=password]').fill('password123')
    await page.locator('[data-testid=submit]').click()

    await expect(page).toHaveURL(/.*dashboard/)
    await expect(page.locator('[data-testid=welcome-message]')).toContainText('Добро пожаловать')
  })

  test('показывает ошибку при неверных данных', async ({ page }) => {
    await page.locator('[data-testid=username]').fill('invalid')
    await page.locator('[data-testid=password]').fill('wrong')
    await page.locator('[data-testid=submit]').click()

    await expect(page.locator('[data-testid=error-message]')).toBeVisible()
  })
})
```

## Настройка E2E тестирования

### Конфигурация Cypress

```javascript
// cypress.config.js
import { defineConfig } from 'cypress'

export default defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      // настройка событий
    },
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
    env: {
      apiUrl: 'http://localhost:8080/api'
    }
  }
})
```

### Конфигурация Playwright

```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 1,
  workers: process.env.CI ? 1 : undefined,
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
})
```

## Практические рекомендации

### Использование Page Object паттерна

Page Object паттерн улучшает поддерживаемость тестов:

```javascript
// pages/LoginPage.js
class LoginPage {
  constructor(page) {
    this.page = page
    this.usernameInput = page.locator('[data-testid=username]')
    this.passwordInput = page.locator('[data-testid=password]')
    this.submitButton = page.locator('[data-testid=submit]')
    this.errorMessage = page.locator('[data-testid=error-message]')
  }

  async login(username, password) {
    await this.usernameInput.fill(username)
    await this.passwordInput.fill(password)
    await this.submitButton.click()
  }

  async isErrorMessageVisible() {
    return await this.errorMessage.isVisible()
  }
}

// tests/login.spec.js
import LoginPage from '../pages/LoginPage'

test('проверка входа в систему', async ({ page }) => {
  const loginPage = new LoginPage(page)
  await loginPage.login('testuser', 'password123')
  
  // Проверки
})
```

### Тестирование формы регистрации

```javascript
// cypress/e2e/registration.cy.js
describe('Регистрация пользователя', () => {
  beforeEach(() => {
    cy.visit('/register')
  })

  it('регистрирует нового пользователя', () => {
    cy.get('[data-cy=first-name]').type('Иван')
    cy.get('[data-cy=last-name]').type('Петров')
    cy.get('[data-cy=email]').type('ivan@example.com')
    cy.get('[data-cy=password]').type('SecurePassword123!')
    cy.get('[data-cy=confirm-password]').type('SecurePassword123!')
    
    cy.get('[data-cy=terms]').check()
    cy.get('[data-cy=submit]').click()

    cy.url().should('include', '/dashboard')
    cy.get('[data-cy=success-message]').should('contain', 'Регистрация успешна')
  })

  it('показывает ошибки при неверных данных', () => {
    cy.get('[data-cy=submit]').click()

    cy.get('[data-cy=first-name-error]').should('be.visible')
    cy.get('[data-cy=email-error]').should('be.visible')
    cy.get('[data-cy=password-error]').should('be.visible')
  })
})
```

### Тестирование сложных сценариев

```javascript
// cypress/e2e/shopping-cart.cy.js
describe('Корзина покупок', () => {
  beforeEach(() => {
    cy.login('testuser', 'password123') // кастомная команда
    cy.visit('/products')
  })

  it('добавляет товар в корзину и оформляет заказ', () => {
    // Поиск и выбор товара
    cy.get('[data-cy=product-card]').first().find('[data-cy=add-to-cart]').click()
    
    // Переход в корзину
    cy.get('[data-cy=cart-icon]').click()
    
    // Проверка содержимого корзины
    cy.get('[data-cy=cart-item]').should('have.length', 1)
    
    // Оформление заказа
    cy.get('[data-cy=checkout-button]').click()
    
    // Заполнение данных доставки
    cy.get('[data-cy=address]').type('ул. Тестовая, д. 1')
    cy.get('[data-cy=payment-method]').select('card')
    
    // Подтверждение заказа
    cy.get('[data-cy=confirm-order]').click()
    
    // Проверка результата
    cy.get('[data-cy=order-confirmation]').should('be.visible')
    cy.get('[data-cy=order-number]').should('not.be.empty')
  })
})
```

## Тестирование с API

E2E тесты часто взаимодействуют с API:

```javascript
// cypress/e2e/api-integration.cy.js
describe('Интеграция с API', () => {
  it('загружает данные из API', () => {
    // Мокаем API ответ
    cy.intercept('GET', '/api/users/current', {
      statusCode: 200,
      body: {
        id: 1,
        name: 'Иван Петров',
        email: 'ivan@example.com'
      }
    }).as('getCurrentUser')

    cy.visit('/profile')
    cy.wait('@getCurrentUser')

    cy.get('[data-cy=user-name]').should('contain', 'Иван Петров')
  })

  it('создает новую запись через API', () => {
    cy.intercept('POST', '/api/posts', {
      statusCode: 201,
      body: {
        id: 123,
        title: 'Новый пост',
        content: 'Содержимое поста'
      }
    }).as('createPost')

    cy.visit('/posts/new')
    cy.get('[data-cy=title]').type('Новый пост')
    cy.get('[data-cy=content]').type('Содержимое поста')
    cy.get('[data-cy=submit]').click()

    cy.wait('@createPost')
    cy.url().should('include', '/posts/123')
  })
})
```

## Особенности российского рынка в 2025 году

- Совместимость с отечественными браузерами и ОС
- Учет требований законодательства (ФЗ-152 о персональных данных)
- Локализация интерфейса: тесты должны учитывать русский язык
- Ограничения на использование западных сервисов: возможно использование отечественных альтернатив
- Требования к отказоустойчивости и безопасности

## Интеграция с CI/CD

Пример конфигурации GitHub Actions для E2E тестов:

```yaml
# .github/workflows/e2e.yml
name: E2E Tests
on: [push, pull_request]
jobs:
  e2e-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Установка зависимостей
        run: npm ci
      - name: Запуск сервера
        run: npm run serve &
        env:
          NODE_ENV: test
      - name: Запуск E2E тестов
        run: npm run test:e2e
        env:
          CYPRESS_baseUrl: http://localhost:3000
```

## Рекомендуемые практики

- Тестировать ключевые пользовательские сценарии
- Использовать семантические селекторы (data-testid, data-cy)
- Избегать хрупких селекторов, зависящих от структуры DOM
- Тестировать как успешные, так и неуспешные сценарии
- Использовать фикстуры для тестовых данных
- Разделять тесты на независимые блоки
- Регулярно обновлять скриншоты и записи для визуального тестирования
- Использовать headless режим для CI/CD

## Типичные ошибки и решения

### Проблема: Нестабильные тесты (flaky tests)

**Решение:** Использовать явные ожидания вместо фиксированных задержек:

```javascript
// ❌ Плохо
cy.wait(2000)
cy.get('.result').should('be.visible')

// ✅ Хорошо
cy.get('.loading').should('not.exist')
cy.get('.result').should('be.visible')
```

### Проблема: Зависимость от внешних сервисов

**Решение:** Мокать внешние API:

```javascript
cy.intercept('GET', '/api/external-service', {
  fixture: 'external-service-data.json'
})
```

## См. также

- [[Тестирование-компонентов]]
- [[Тестирование-хранилищ]]
- [[Cypress]]
- [[Playwright]]
- [[Jest]]
# E2E-тесты

## Свойства
```yaml
tags: [testing, frontend, e2e-testing, javascript, automation]
aliases: [Сквозное тестирование, End-to-End Testing, E2E Testing]
```

## Обзор
E2E (End-to-End) тесты — это тип тестирования, который проверяет полный рабочий процесс приложения от начала до конца, имитируя реальное поведение пользователя. Эти тесты проверяют, как различные слои приложения (фронтенд, бэкенд, база данных, внешние сервисы) работают вместе в условиях, максимально приближенных к production.

## Подробности

### Что тестируется
- Полные пользовательские сценарии
- Взаимодействие с UI
- Интеграция с серверной частью
- Работа с базами данных
- Внешние API и сервисы
- Производительность приложения в целом

### Принципы
- **Реалистичность**: максимально приближены к реальному использованию
- **Полнота**: охватывают весь путь пользователя
- **Сложность**: требуют сложной настройки и поддержки
- **Время выполнения**: самые медленные из всех типов тестов
- **Надежность**: могут быть хрупкими из-за зависимости от внешних факторов

### Популярные фреймворки и инструменты
- Cypress
- Playwright
- Selenium
- Puppeteer
- TestCafe

### Примеры

#### Cypress тест
```javascript
// cypress/e2e/login.cy.js
describe('Login functionality', () => {
  it('should allow user to login successfully', () => {
    cy.visit('/login');
    
    cy.get('[data-testid="email-input"]').type('user@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="login-button"]').click();
    
    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="welcome-message"]').should('contain', 'Welcome');
  });
});
```

#### Playwright тест
```javascript
// tests/login.spec.js
import { test, expect } from '@playwright/test';

test('should allow user to login successfully', async ({ page }) => {
  await page.goto('/login');
  
  await page.locator('[data-testid="email-input"]').fill('user@example.com');
  await page.locator('[data-testid="password-input"]').fill('password123');
  await page.locator('[data-testid="login-button"]').click();
  
  await expect(page).toHaveURL(/.*dashboard/);
  await expect(page.locator('[data-testid="welcome-message"]')).toContainText('Welcome');
});
```

### Лучшие практики
- Пишите тесты для критических пользовательских сценариев
- Используйте стабильные селекторы (например, data-testid)
- Избегайте хрупких тестов, чувствительных к мелким изменениям
- Запускайте E2E тесты в окружении, максимально приближенном к production
- Ограничьте количество E2E тестов, используя их рационально

## Связанные темы
- [[Unit-тесты]]
- [[Интеграционные-тесты]]
- [[Тестирование-производительности]]
- [[Тестирование-компонентов]]
- [[CI/CD]]

> [!tip] Совет
> Используйте E2E тесты для проверки критических пользовательских путей, таких как регистрация, логин или оформление заказа.

> [!warning] Важно
> Не пытайтесь покрыть все сценарии E2E тестами. Они дорогостоящи в поддержке и выполнении.

## Заключение
E2E тесты обеспечивают уверенность в том, что приложение работает корректно в целом, как единая система, и являются важной частью стратегии тестирования фронтенд-приложений.
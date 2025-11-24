---
aliases: ["Сквозное тестирование", "End-to-End Testing", "Тестирование пользовательских сценариев"]
tags: [html/testing, e2e-testing, front-end-testing, automation]
---

# E2E-тесты

## Введение

E2E-тесты (End-to-End, сквозное тестирование) в контексте HTML-разработки имитируют реальные пользовательские сценарии и проверяют работу всего приложения от начала до конца. Эти тесты обеспечивают уверенность в том, что пользовательские потоки работают корректно в реальных условиях.

## Основные принципы E2E-тестирования HTML

E2E-тесты для HTML-приложений фокусируются на:

- Проверке пользовательских сценариев от начала до конца
- Взаимодействии с реальными API и базами данных
- Проверке корректности отображения на разных устройствах и браузерах
- Тестировании сложных сценариев с несколькими страницами
- Проверке производительности пользовательского интерфейса

## Инструменты для E2E-тестирования HTML

### Playwright
Microsoft Playwright - один из самых мощных инструментов для E2E-тестирования HTML-приложений.

```javascript
// Пример E2E-теста с использованием Playwright
import { test, expect } from '@playwright/test';

test.describe('Регистрация пользователя', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('https://example.com/register');
  });

  test('должна успешно зарегистрировать нового пользователя', async ({ page }) => {
    // Заполнение формы регистрации
    await page.locator('#username').fill('testuser123');
    await page.locator('#email').fill('test@example.com');
    await page.locator('#password').fill('SecurePassword123!');
    await page.locator('#confirmPassword').fill('SecurePassword123!');

    // Отправка формы
    await page.locator('button[type="submit"]').click();

    // Проверка результата
    await expect(page.locator('.success-message')).toBeVisible();
    await expect(page).toHaveURL('https://example.com/dashboard');
  });

  test('должна показать ошибку при слабом пароле', async ({ page }) => {
    await page.locator('#username').fill('testuser');
    await page.locator('#email').fill('test@example.com');
    await page.locator('#password').fill('123'); // Слабый пароль
    await page.locator('#confirmPassword').fill('123');
    await page.locator('button[type="submit"]').click();

    await expect(page.locator('.error-message')).toContainText('Пароль слишком слабый');
  });
});
```

### Cypress
Cypress - популярный инструмент для E2E-тестирования с отличной интеграцией с HTML-приложениями.

```javascript
// Пример E2E-теста с использованием Cypress
describe('Поиск товаров', () => {
  beforeEach(() => {
    cy.visit('https://example-shop.com');
  });

  it('должен найти товар по ключевому слову', () => {
    cy.get('#search-input').type('смартфон');
    cy.get('#search-button').click();

    cy.get('.product-card').should('have.length.greaterThan', 0);
    cy.get('.product-card').first().should('contain.text', 'смартфон');
  });

  it('должен добавить товар в корзину', () => {
    cy.get('#search-input').type('ноутбук');
    cy.get('#search-button').click();
    cy.get('.product-card').first().find('.add-to-cart').click();

    cy.get('#cart-count').should('contain.text', '1');
  });
});
```

### Selenium WebDriver
Классический инструмент для автоматизации браузеров, поддерживающий все основные браузеры.

## Практические примеры E2E-тестов

### Тестирование процесса оформления заказа

```javascript
// E2E-тест процесса оформления заказа
import { test, expect } from '@playwright/test';

test.describe('Оформление заказа', () => {
  test('должно завершить заказ с новым пользователем', async ({ page }) => {
    // Переход на главную страницу
    await page.goto('https://shop.example.com');

    // Поиск и выбор товара
    await page.locator('#search-input').fill('iPhone');
    await page.locator('#search-button').click();
    await page.locator('.product-card').first().click();

    // Добавление в корзину
    await page.locator('#add-to-cart').click();
    await page.waitForSelector('#cart-count');

    // Переход в корзину
    await page.locator('#cart-icon').click();
    await expect(page).toHaveURL(/.*cart/);

    // Оформление заказа
    await page.locator('#checkout-button').click();
    await expect(page).toHaveURL(/.*checkout/);

    // Заполнение формы доставки
    await page.locator('#delivery-name').fill('Иван Иванов');
    await page.locator('#delivery-address').fill('ул. Примерная, д. 1');
    await page.locator('#delivery-phone').fill('+79991234567');
    await page.locator('#delivery-email').fill('ivan@example.com');

    // Выбор способа оплаты
    await page.locator('#payment-method-card').click();

    // Подтверждение заказа
    await page.locator('#confirm-order').click();

    // Проверка результата
    await expect(page.locator('.order-success')).toBeVisible();
    await expect(page.locator('.order-number')).not.toBeNull();
  });
});
```

### Тестирование адаптивного дизайна

```javascript
// E2E-тест адаптивности HTML-страницы
import { test, expect } from '@playwright/test';

test.describe('Адаптивный дизайн', () => {
  test('должен корректно отображаться на мобильном устройстве', async ({ page }) => {
    // Установка мобильного размера экрана
    await page.setViewportSize({ width: 375, height: 667 });

    await page.goto('https://example.com');
    
    // Проверка видимости мобильного меню
    await expect(page.locator('#mobile-menu-toggle')).toBeVisible();
    
    // Проверка скрытия десктопного меню
    await expect(page.locator('#desktop-menu')).not.toBeVisible();
    
    // Проверка размеров элементов
    const header = page.locator('header');
    await expect(header).toHaveCSS('height', '60px'); // Пример значения
  });

  test('должен корректно отображаться на планшете', async ({ page }) => {
    await page.setViewportSize({ width: 768, height: 1024 });
    
    await page.goto('https://example.com');
    
    // Проверка адаптивности
    const layout = page.locator('.main-layout');
    await expect(layout).toHaveCSS('grid-template-columns', 'repeat(2, 1fr)');
  });
});
```

## Лучшие практики E2E-тестирования HTML

1. **Тестирование реальных пользовательских сценариев**: Фокусируйтесь на том, что действительно делают пользователи
2. **Использование Page Object Model**: Организуйте тесты с использованием паттерна Page Object
3. **Параллельное выполнение**: Запускайте тесты параллельно для ускорения процесса
4. **Устойчивость тестов**: Обеспечьте надежность тестов с использованием ожиданий и явных задержек
5. **Тестирование на разных браузерах**: Проверяйте приложение в разных браузерах и версиях

## Особенности в российских реалиях 2025

В 2025 году в России E2E-тестирование HTML-приложений приобретает особое значение из-за:

- Ужесточения требований к качеству цифровых сервисов
- Роста числа пользователей цифровых платформ
- Повышенного внимания к [[Тесты-доступности]] и [[Безопасность HTML|безопасности HTML]]
- Необходимости соответствия отечественным стандартам (например, требованиям Минцифры)
- Перехода на отечественные решения для тестирования и CI/CD

Многие российские компании уделяют особое внимание E2E-тестированию для обеспечения стабильности работы государственных и коммерческих порталов.

## Заключение

E2E-тесты обеспечивают уверенность в том, что пользовательские сценарии работают корректно в реальных условиях. Они являются важной частью процесса обеспечения качества HTML-приложений и помогают выявлять проблемы, которые могут быть пропущены на других уровнях тестирования.

См. также:
- [[Unit-тесты]]
- [[Интеграционные-тесты]]
- [[HTML тестирование|Общее руководство по тестированию HTML]]
- [[Тестирование производительности]]
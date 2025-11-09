# E2E (End-to-End) тестирование в JavaScript

## Обзор

E2E (End-to-End) тестирование - это метод тестирования, который проверяет поток выполнения приложения от начала до конца. E2E тесты имитируют реальные сценарии использования, взаимодействуя с приложением так же, как это делает реальный пользователь. Это позволяет проверить интеграцию всех компонентов системы: фронтенд, бэкенд, базы данных и внешние сервисы.

## Основные концепции E2E тестирования

### Отличие от других видов тестирования

```javascript
// E2E тест проверяет полный пользовательский сценарий
// В отличие от модульных тестов, которые тестируют отдельные функции,
// и интеграционных тестов, которые тестируют взаимодействие между модулями

// Пример полного пользовательского сценария:
// 1. Пользователь заходит на сайт
// 2. Регистрируется
// 3. Авторизуется
// 4. Создает профиль
// 5. Публикует пост
// 6. Проверяет результат

// Это невозможно проверить с помощью модульных или интеграционных тестов
```

## Инструменты для E2E тестирования

### Playwright - современный инструмент для E2E тестирования

```javascript
// playwright.config.js
import { defineConfig } from '@playwright/test';

export default defineConfig({
  // Путь к тестам
  testDir: './e2e-tests',
  
  // Настройки запуска
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 1,
  workers: process.env.CI ? 1 : undefined,
  
  // Настройки браузеров
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
  
  // Настройки веб-сервера для тестов
  webServer: {
    command: 'npm run serve', // команда для запуска приложения
    url: 'http://localhost:3000', // URL для проверки готовности
    timeout: 120 * 1000,
    reuseExistingServer: !process.env.CI,
  },
});
```

```javascript
// e2e-tests/user-registration.spec.js
import { test, expect } from '@playwright/test';

test.describe('Регистрация пользователя', () => {
  test.beforeEach(async ({ page }) => {
    // Переход на главную страницу перед каждым тестом
    await page.goto('/');
  });

  test('успешная регистрация нового пользователя', async ({ page }) => {
    // 1. Переход на страницу регистрации
    await page.getByRole('link', { name: 'Регистрация' }).click();
    
    // 2. Заполнение формы регистрации
    await page.locator('#name').fill('Иван Иванов');
    await page.locator('#email').fill(`user-${Date.now()}@example.com`);
    await page.locator('#password').fill('securePassword123');
    await page.locator('#confirm-password').fill('securePassword123');
    
    // 3. Отправка формы
    await page.getByRole('button', { name: 'Зарегистрироваться' }).click();
    
    // 4. Проверка успешной регистрации
    await expect(page.getByText('Регистрация прошла успешно')).toBeVisible();
    await expect(page).toHaveURL(/\/dashboard/);
  });

  test('регистрация с некорректными данными показывает ошибки', async ({ page }) => {
    await page.getByRole('link', { name: 'Регистрация' }).click();
    
    // Отправка пустой формы
    await page.getByRole('button', { name: 'Зарегистрироваться' }).click();
    
    // Проверка ошибок валидации
    await expect(page.getByText('Имя обязательно')).toBeVisible();
    await expect(page.getByText('Email обязателен')).toBeVisible();
    await expect(page.getByText('Пароль обязателен')).toBeVisible();
    
    // Попытка регистрации с коротким паролем
    await page.locator('#password').fill('123');
    await page.getByRole('button', { name: 'Зарегистрироваться' }).click();
    
    await expect(page.getByText('Пароль должен содержать не менее 8 символов')).toBeVisible();
  });
});
```

### Cypress - альтернативный инструмент для E2E тестирования

```javascript
// cypress.config.js
import { defineConfig } from 'cypress';

export default defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      // implement node event listeners here
    },
    baseUrl: 'http://localhost:3000',
    viewportWidth: 1280,
    viewportHeight: 720,
  },
});
```

```javascript
// cypress/e2e/user-authentication.cy.js
describe('Аутентификация пользователя', () => {
  beforeEach(() => {
    cy.visit('/');
  });

  it('успешный вход в систему', () => {
    // 1. Переход на страницу входа
    cy.get('[data-testid="login-link"]').click();
    
    // 2. Заполнение формы входа
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    
    // 3. Отправка формы
    cy.get('[data-testid="login-button"]').click();
    
    // 4. Проверка успешного входа
    cy.url().should('include', '/dashboard');
    cy.get('[data-testid="user-menu"]').should('be.visible');
  });

  it('показывает ошибку при неверных данных', () => {
    cy.get('[data-testid="login-link"]').click();
    
    cy.get('[data-testid="email-input"]').type('invalid@example.com');
    cy.get('[data-testid="password-input"]').type('wrongpassword');
    cy.get('[data-testid="login-button"]').click();
    
    cy.get('[data-testid="error-message"]')
      .should('be.visible')
      .and('contain', 'Неверный email или пароль');
  });
});
```

## Продвинутые сценарии E2E тестирования

### Тестирование API и UI интеграции

```javascript
// e2e-tests/api-ui-integration.spec.js
import { test, expect } from '@playwright/test';

test.describe('Интеграция API и UI', () => {
  test('создание задачи через UI отражается в API', async ({ page, request }) => {
    // Авторизация
    await page.goto('/login');
    await page.locator('#email').fill('test@example.com');
    await page.locator('#password').fill('password123');
    await page.getByRole('button', { name: 'Войти' }).click();
    
    // Создание задачи через UI
    await page.getByRole('link', { name: 'Задачи' }).click();
    await page.getByRole('button', { name: 'Добавить задачу' }).click();
    
    const taskTitle = `Тестовая задача ${Date.now()}`;
    await page.locator('#task-title').fill(taskTitle);
    await page.locator('#task-description').fill('Описание тестовой задачи');
    await page.getByRole('button', { name: 'Сохранить' }).click();
    
    // Проверка, что задача появилась в UI
    await expect(page.getByText(taskTitle)).toBeVisible();
    
    // Проверка, что задача доступна через API
    const response = await request.get('/api/tasks');
    const tasks = await response.json();
    const createdTask = tasks.find(task => task.title === taskTitle);
    
    expect(createdTask).toBeDefined();
    expect(createdTask.title).toBe(taskTitle);
  });

  test('удаление задачи через API отражается в UI', async ({ page, request }) => {
    // Создаем задачу через API
    const newTask = {
      title: `API задача ${Date.now()}`,
      description: 'Задача создана через API'
    };
    
    const createResponse = await request.post('/api/tasks', {
      data: newTask
    });
    
    const createdTask = await createResponse.json();
    
    // Открываем UI и проверяем, что задача отображается
    await page.goto('/tasks');
    await page.locator(`[data-task-id="${createdTask.id}"]`).waitFor();
    
    // Удаляем задачу через API
    await request.delete(`/api/tasks/${createdTask.id}`);
    
    // Перезагружаем страницу и проверяем, что задача исчезла
    await page.reload();
    await expect(page.locator(`[data-task-id="${createdTask.id}"]`)).not.toBeVisible();
  });
});
```

### Тестирование с различными состояниями приложения

```javascript
// e2e-tests/application-states.spec.js
import { test, expect } from '@playwright/test';

test.describe('Тестирование различных состояний приложения', () => {
  test('страница загрузки данных', async ({ page }) => {
    // Создаем замедленный ответ от API для тестирования состояния загрузки
    await page.route('**/api/data', async route => {
      // Задержка для симуляции загрузки
      await new Promise(resolve => setTimeout(resolve, 2000));
      await route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({ data: 'some data' })
      });
    });
    
    await page.goto('/data-page');
    
    // Проверяем, что индикатор загрузки отображается
    await expect(page.getByText('Загрузка...')).toBeVisible();
    
    // Проверяем, что данные загружаются после индикатора
    await expect(page.getByText('some data')).toBeVisible();
  });

  test('обращение с ошибками API', async ({ page }) => {
    // Мокаем ошибку API
    await page.route('**/api/user', route => {
      route.fulfill({
        status: 500,
        body: JSON.stringify({ error: 'Внутренняя ошибка сервера' })
      });
    });
    
    await page.goto('/profile');
    
    // Проверяем, что отображается сообщение об ошибке
    await expect(page.getByText('Ошибка загрузки профиля')).toBeVisible();
    await expect(page.getByText('Внутренняя ошибка сервера')).toBeVisible();
  });

  test('пустое состояние списка', async ({ page }) => {
    // Мокаем пустой ответ от API
    await page.route('**/api/tasks', route => {
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify([])
      });
    });
    
    await page.goto('/tasks');
    
    // Проверяем, что отображается сообщение о пустом списке
    await expect(page.getByText('У вас пока нет задач')).toBeVisible();
    await expect(page.getByRole('button', { name: 'Создать первую задачу' })).toBeVisible();
  });
});
```

## Практические примеры сложных сценариев

### Тестирование процесса оформления заказа

```javascript
// e2e-tests/order-process.spec.js
import { test, expect } from '@playwright/test';

test.describe('Процесс оформления заказа', () => {
  test('полный процесс покупки товара', async ({ page }) => {
    // 1. Просмотр каталога товаров
    await page.goto('/catalog');
    
    // 2. Поиск и выбор товара
    await page.locator('#search-input').fill('ноутбук');
    await page.getByRole('button', { name: 'Поиск' }).click();
    
    const firstProduct = page.locator('.product-card').first();
    const productName = await firstProduct.locator('.product-name').textContent();
    const productPrice = await firstProduct.locator('.product-price').textContent();
    
    await firstProduct.locator('.add-to-cart').click();
    
    // 3. Проверка добавления в корзину
    await expect(page.locator('.cart-icon .badge')).toContainText('1');
    
    // 4. Переход в корзину
    await page.locator('.cart-icon').click();
    await page.getByRole('link', { name: 'Перейти в корзину' }).click();
    
    // 5. Проверка содержимого корзины
    await expect(page.getByText(productName)).toBeVisible();
    await expect(page.getByText(productPrice)).toBeVisible();
    
    // 6. Оформление заказа
    await page.getByRole('button', { name: 'Оформить заказ' }).click();
    
    // 7. Заполнение данных доставки
    await page.locator('#delivery-name').fill('Иван Иванов');
    await page.locator('#delivery-address').fill('ул. Тестовая, д. 1');
    await page.locator('#delivery-phone').fill('+79991234567');
    
    // 8. Выбор способа оплаты
    await page.locator('input[name="payment-method"][value="card"]').check();
    
    // 9. Подтверждение заказа
    await page.getByRole('button', { name: 'Подтвердить заказ' }).click();
    
    // 10. Проверка успешного оформления
    await expect(page.getByText('Заказ успешно оформлен')).toBeVisible();
    await expect(page.getByText('Номер заказа:')).toBeVisible();
  });
});
```

### Тестирование авторизации и управления сессией

```javascript
// e2e-tests/auth-session.spec.js
import { test, expect } from '@playwright/test';

test.describe('Управление сессией и авторизация', () => {
  test('автоматический выход при истечении сессии', async ({ page }) => {
    // Настройка короткого времени жизни сессии для тестирования
    await page.context().route('**/api/session', route => {
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({ 
          expiresAt: new Date(Date.now() + 5000).toISOString() // 5 секунд
        })
      });
    });
    
    // Вход в систему
    await page.goto('/login');
    await page.locator('#email').fill('test@example.com');
    await page.locator('#password').fill('password123');
    await page.getByRole('button', { name: 'Войти' }).click();
    
    await expect(page).toHaveURL('/dashboard');
    
    // Ожидание истечения сессии (больше 5 секунд)
    await page.waitForTimeout(6000);
    
    // Попытка доступа к защищенному ресурсу
    await page.goto('/profile');
    
    // Проверка, что пользователь был автоматически разлогинен
    await expect(page).toHaveURL('/login');
    await expect(page.getByText('Сессия истекла, пожалуйста, войдите снова')).toBeVisible();
  });

  test('сохранение состояния при перезагрузке страницы', async ({ page }) => {
    // Вход в систему
    await page.goto('/login');
    await page.locator('#email').fill('test@example.com');
    await page.locator('#password').fill('password123');
    await page.getByRole('button', { name: 'Войти' }).click();
    
    // Проверка, что пользователь вошел
    await expect(page.getByText('Добро пожаловать, test@example.com')).toBeVisible();
    
    // Перезагрузка страницы
    await page.reload();
    
    // Проверка, что пользователь все еще залогинен
    await expect(page.getByText('Добро пожаловать, test@example.com')).toBeVisible();
    await expect(page).not.toHaveURL('/login');
  });
});
```

## Паттерны и практики E2E тестирования

### Использование Page Object модели

```javascript
// pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.locator('#email');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.getByRole('button', { name: 'Войти' });
    this.errorMessage = page.locator('.error-message');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}

// pages/DashboardPage.js
export class DashboardPage {
  constructor(page) {
    this.page = page;
    this.welcomeMessage = page.getByText('Добро пожаловать');
    this.logoutButton = page.getByRole('button', { name: 'Выйти' });
  }

  async waitForWelcomeMessage() {
    await this.welcomeMessage.waitFor();
  }

  async logout() {
    await this.logoutButton.click();
  }
}

// e2e-tests/login-with-pom.spec.js
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';
import { DashboardPage } from '../pages/DashboardPage';

test.describe('Тестирование с Page Object моделью', () => {
  test('успешный вход и выход', async ({ page }) => {
    const loginPage = new LoginPage(page);
    const dashboardPage = new DashboardPage(page);

    await loginPage.goto();
    await loginPage.login('test@example.com', 'password123');

    await dashboardPage.waitForWelcomeMessage();
    await expect(page).toHaveURL('/dashboard');

    await dashboardPage.logout();
    await expect(page).toHaveURL('/login');
  });
});
```

### Использование фикстур для настройки тестов

```javascript
// tests/fixtures/authenticated-user.js
import { test as base } from '@playwright/test';

export const test = base.extend({
  // Автоматическая аутентификация для всех тестов в этом файле
  page: async ({ page }, use) => {
    // Вход до начала теста
    await page.goto('/login');
    await page.locator('#email').fill('test@example.com');
    await page.locator('#password').fill('password123');
    await page.getByRole('button', { name: 'Войти' }).click();
    
    // Ожидание загрузки главной страницы
    await page.waitForURL('/dashboard');
    
    // Использование страницы в тесте
    await use(page);
    
    // Опционально: выход после теста
    await page.getByRole('button', { name: 'Выйти' }).click();
  },
});

export { expect } from '@playwright/test';

// e2e-tests/authenticated-tests.spec.js
import { test, expect } from './fixtures/authenticated-user';

test.describe('Тесты для аутентифицированного пользователя', () => {
  test('доступ к профилю', async ({ page }) => {
    await page.goto('/profile');
    await expect(page.getByText('Мой профиль')).toBeVisible();
  });

  test('создание нового поста', async ({ page }) => {
    await page.goto('/posts/create');
    await page.locator('#title').fill('Новый пост');
    await page.locator('#content').fill('Содержимое поста');
    await page.getByRole('button', { name: 'Опубликовать' }).click();
    
    await expect(page.getByText('Пост успешно опубликован')).toBeVisible();
  });
});
```

## Лучшие практики E2E тестирования

### Организация тестов

```javascript
// e2e-tests/best-practices.spec.js
import { test, expect } from '@playwright/test';

test.describe('Лучшие практики E2E тестирования', () => {
  // 1. Использование семантических селекторов
  test('использование data-testid атрибутов', async ({ page }) => {
    await page.goto('/form');
    
    // Хорошо: использование data-testid
    await page.locator('[data-testid="submit-button"]').click();
    
    // Избегайте: использование CSS классов или текста, которые могут измениться
    // await page.locator('.btn-primary').click();
    // await page.getByText('Отправить').click();
  });

  // 2. Проверка ожидаемого состояния, а не конкретных элементов
  test('проверка состояния вместо элементов', async ({ page }) => {
    await page.goto('/loading-state');
    
    // Хорошо: проверка ожидаемого состояния
    await expect(page.getByText('Данные загружены')).toBeVisible();
    
    // Вместо проверки наличия/отсутствия элементов
  });

  // 3. Избегание хардкода ожиданий
  test('использование встроенных ожиданий Playwright', async ({ page }) => {
    await page.goto('/async-content');
    
    // Хорошо: Playwright автоматически ждет элемент
    await expect(page.getByText('Динамический контент')).toBeVisible();
    
    // Плохо: хардкод ожидания
    // await page.waitForTimeout(2000);
    // await expect(page.locator('#dynamic-content')).toBeVisible();
  });

  // 4. Тестирование пользовательских сценариев, а не функций UI
  test('тестирование пользовательской цели', async ({ page }) => {
    await page.goto('/search');
    
    // Хорошо: тестирование цели пользователя
    await page.locator('#search-input').fill('ноутбук');
    await page.getByRole('button', { name: 'Поиск' }).click();
    
    // Проверка, что пользователь нашел то, что искал
    await expect(page.getByText('Результаты поиска')).toBeVisible();
    await expect(page.locator('.search-result')).toHaveCountGreaterThan(0);
    
    // Вместо тестирования конкретных элементов интерфейса
  });
});
```

## Заключение

E2E тестирование является важной частью процесса разработки, обеспечивающей:

1. Проверку полных пользовательских сценариев
2. Обнаружение интеграционных проблем между компонентами
3. Уверенность в стабильности пользовательского опыта
4. Раннее обнаружение критических багов
5. Автоматизацию процесса тестирования

Правильное использование E2E тестов требует баланса между полнотой покрытия и скоростью выполнения тестов.
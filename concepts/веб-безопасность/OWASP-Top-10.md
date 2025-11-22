---
aliases: ["OWASP Top 10", "Топ 10 угроз безопасности", "OWASP"]
tags: ["#security", "#owasp", "#web-security", "#frontend-security"]
---

# OWASP Top 10

**OWASP Top 10** — это список из десяти наиболее критических угроз безопасности для веб-приложений, составленный Open Web Application Security Project (OWASP). Он обновляется каждые несколько лет и служит руководством для разработчиков и специалистов по безопасности для понимания и предотвращения распространенных уязвимостей.

## Обзор угроз

Согласно последнему списку OWASP Top 10 (2021), основные категории угроз включают:

1. [**Broken Access Control**](broken-access-control) (Нарушенный контроль доступа)
2. [**Cryptographic Failures**](cryptographic-failures) (Криптографические сбои)
3. [**Injection**](injection) (Инъекции)
4. [**Insecure Design**](insecure-design) (Небезопасный дизайн)
5. [**Security Misconfiguration**](security-misconfiguration) (Неправильная настройка безопасности)
6. [**Vulnerable and Outdated Components**](vulnerable-components) (Уязвимые и устаревшие компоненты)
7. [**Identification and Authentication Failures**](auth-failures) (Сбои идентификации и аутентификации)
8. [**Software and Data Integrity Failures**](integrity-failures) (Сбои целостности программного обеспечения и данных)
9. [**Security Logging and Monitoring Failures**](logging-monitoring-failures) (Сбои ведения журналов и мониторинга безопасности)
10. [**Server-Side Request Forgery (SSRF)**](ssrf) (Подделка запросов на стороне сервера)

## Влияние на фронтенд-разработку

Хотя многие угрозы из OWASP Top 10 касаются серверной стороны, фронтенд-разработчики также играют важную роль в обеспечении безопасности. Например:

- [**Injection**](injection) может происходить через клиентские формы, если данные не валидируются должным образом.
- [**Broken Access Control**](broken-access-control) может быть усугублен неправильным отображением данных на клиенте.
- [**Security Misconfiguration**](security-misconfiguration) может включать неправильные настройки HTTP-заголовков безопасности, таких как CSP.

## Практические рекомендации для фронтенд-разработчиков

### 1. Валидация данных на клиенте и сервере

Хотя валидация на клиенте улучшает UX, она не должна быть единственной линией защиты:

```javascript
// Неправильно: только клиентская валидация
function submitForm(data) {
  // Предполагаем, что данные уже валидированы
  sendToServer(data);
}

// Правильно: валидация как на клиенте, так и на сервере
function submitForm(data) {
  if (!validateInput(data)) {
    showErrorMessage("Неверные данные");
    return;
  }
  sendToServer(data);
}

function validateInput(data) {
  // Простая проверка на клиенте
  return typeof data === 'object' && data.email && data.email.includes('@');
}
```

### 2. Защита от XSS

Используйте безопасные методы вставки данных в DOM:

```javascript
// Неправильно: уязвимо к XSS
function displayUserInput(input) {
  document.getElementById('output').innerHTML = input;
}

// Правильно: безопасное вставление
function displayUserInput(input) {
  document.getElementById('output').textContent = input;
}
```

### 3. Использование CSP

Убедитесь, что ваше приложение использует Content Security Policy (CSP) через HTTP-заголовки:

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
```

## Связанные темы

- [[Защита-от-XSS]]
- [[CORS-и-безопасность]]
- [[HTTPS-и-шифрование]]
- [[Защита-от-CSRF]]

## Заключение

Понимание OWASP Top 10 критически важно для всех разработчиков. Фронтенд-разработчики должны сотрудничать с бэкенд-разработчиками, чтобы обеспечить комплексную защиту от этих угроз.

> [!tip] Совет
> Регулярно обновляйте знания по OWASP Top 10, так как список периодически обновляется с учетом новых угроз и изменений в веб-безопасности.

> [!warning] Важно
> Не полагайтесь исключительно на фронтенд-защиту. Безопасность должна быть реализована на всех уровнях приложения.
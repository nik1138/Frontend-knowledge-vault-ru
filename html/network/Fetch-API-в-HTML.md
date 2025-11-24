---
aliases: [Fetch API в HTML, Работа с Fetch API]
tags: [html, networking, javascript, web-development]
---

# Fetch API в HTML

## Введение в Fetch API

Fetch API предоставляет современный интерфейс для выполнения сетевых запросов в браузере. Он заменяет устаревший [[XMLHttpRequest]] и предоставляет более мощный и гибкий способ работы с HTTP-запросами и ответами.

Fetch API возвращает [[Promise]], что делает его идеальным для работы с асинхронными операциями и позволяет использовать современные паттерны программирования, такие как `async/await`.

## Основы использования Fetch API

### Простой GET-запрос

```javascript
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Ошибка:', error));
```

### Асинхронный подход с async/await

```javascript
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP ошибка! статус: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Ошибка при получении данных:', error);
  }
}
```

## Настройка параметров запроса

Fetch API позволяет настраивать различные параметры запроса через второй аргумент - объект конфигурации:

```javascript
fetch('/api/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer ' + token
  },
  body: JSON.stringify({
    name: 'John',
    email: 'john@example.com'
  })
})
```

## Типичные заголовки запроса

- `Content-Type`: Определяет тип отправляемых данных
- `Authorization`: Для аутентификации
- `Accept`: Указывает, какие типы ответов клиент может обработать
- `X-Requested-With`: Иногда используется для указания типа запроса

## Работа с различными типами данных

### JSON-данные

```javascript
const response = await fetch('/api/json-endpoint');
const jsonData = await response.json();
```

### Текстовые данные

```javascript
const response = await fetch('/api/text-endpoint');
const textData = await response.text();
```

### Бинарные данные

```javascript
const response = await fetch('/api/image');
const blobData = await response.blob();
```

## Обработка HTTP-статусов

Важно проверять статус ответа, так как Fetch API не отклоняет [[Promise]] при HTTP-ошибках (4xx, 5xx):

```javascript
async function checkStatus() {
  const response = await fetch('/api/endpoint');
  
  if (!response.ok) {
    switch(response.status) {
      case 401:
        console.error('Неавторизованный доступ');
        break;
      case 403:
        console.error('Доступ запрещен');
        break;
      case 404:
        console.error('Ресурс не найден');
        break;
      case 500:
        console.error('Внутренняя ошибка сервера');
        break;
      default:
        console.error(`HTTP ошибка: ${response.status}`);
    }
    throw new Error(`HTTP ошибка! статус: ${response.status}`);
  }
  
  return response.json();
}
```

## Практические рекомендации для российских разработчиков (2025)

### Работа с российскими API

При интеграции с российскими API, такими как:
- [[Яндекс API]]
- [[VK API]]
- API банковских сервисов (Сбербанк, Тинькофф и др.)
- Государственные API (Госуслуги, ФНС и др.)

Важно учитывать:
- Требования к безопасности данных
- Необходимость использования HTTPS
- Возможные ограничения на стороне сервера
- Требования к заголовкам и аутентификации

### Обход ограничений CORS

В российской экосистеме часто сталкиваются с ограничениями CORS. Решения:
- Настройка сервера для корректной обработки CORS-запросов
- Использование прокси-сервера
- Обработка на бэкенде с последующим предоставлением данных фронтенду

### Безопасность запросов

- Не передавайте чувствительные данные в URL
- Используйте HTTPS для всех запросов
- Корректно обрабатывайте токены аутентификации
- Валидируйте ответы сервера перед использованием

## Современные возможности Fetch API

### AbortController для отмены запросов

```javascript
const controller = new AbortController();
const signal = controller.signal;

fetch('/api/long-request', { signal })
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('Запрос был отменен');
    }
  });

// Отмена запроса через 5 секунд
setTimeout(() => controller.abort(), 5000);
```

### Повторные попытки запросов

```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  for (let i = 0; i <= maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      if (!response.ok) {
        throw new Error(`HTTP ошибка: ${response.status}`);
      }
      return response;
    } catch (error) {
      if (i === maxRetries) {
        throw error;
      }
      // Ожидание перед повторной попыткой
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Сравнение с XMLHttpRequest

| Характеристика | Fetch API | XMLHttpRequest |
|----------------|-----------|----------------|
| Синтаксис | Современный и чистый | Устаревший и громоздкий |
| Promise | Поддерживается | Требует обертки |
| Читаемость | Высокая | Средняя |
| Поддержка | Современные браузеры | Все браузеры |
| Удобство | Высокое | Среднее |

## Полезные ресурсы

- [[XMLHttpRequest-в-HTML]] - альтернативный подход
- [[Обработка-ответов-и-ошибок]] - детальное рассмотрение обработки ошибок
- [[CORS-и-безопасность-запросов]] - вопросы безопасности
- [[FormData-и-загрузка-файлов]] - работа с формами и файлами

## Заключение

Fetch API - это современный стандарт для выполнения сетевых запросов в браузере. Его использование упрощает разработку и делает код более читаемым и поддерживаемым. При разработке в российской экосистеме важно учитывать особенности интеграции с отечественными API и требования к безопасности данных.

> [!tip] Совет
> Используйте Fetch API для новых проектов, а также постепенно мигрируйте старые проекты с [[XMLHttpRequest]] для улучшения читаемости и поддержки кода.

> [!warning] Важно
> Не забывайте проверять статус ответа сервера, так как Fetch API не отклоняет Promise при HTTP-ошибках.
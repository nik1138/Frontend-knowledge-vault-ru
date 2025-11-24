---
aliases: [XMLHttpRequest в HTML, Работа с XMLHttpRequest]
tags: [html, networking, javascript, web-development]
---

# XMLHttpRequest в HTML

## Введение в XMLHttpRequest

XMLHttpRequest (XHR) - это встроенный объект браузера, который позволяет выполнять HTTP-запросы из JavaScript. Это устаревший, но все еще широко используемый способ работы с сетевыми запросами, особенно в унаследованном коде и проектах, требующих поддержки старых браузеров.

## Основы использования XMLHttpRequest

### Создание и настройка запроса

```javascript
const xhr = new XMLHttpRequest();

// Настройка запроса
xhr.open('GET', '/api/data', true); // метод, URL, асинхронный/синхронный

// Установка заголовков (если нужно)
xhr.setRequestHeader('Content-Type', 'application/json');

// Установка обработчиков событий
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4 && xhr.status === 200) {
    console.log(xhr.responseText);
  }
};

// Отправка запроса
xhr.send();
```

### Асинхронный и синхронный режимы

```javascript
// Асинхронный запрос (рекомендуется)
xhr.open('GET', '/api/data', true);

// Синхронный запрос (не рекомендуется - блокирует поток выполнения)
xhr.open('GET', '/api/data', false);
```

## Состояния запроса (readyState)

XMLHttpRequest имеет пять состояний:

1. `0` - UNSENT: объект создан, но метод open() еще не вызван
2. `1` - OPENED: метод open() вызван
3. `2` - HEADERS_RECEIVED: метод send() вызван, получены заголовки
4. `3` - LOADING: получены частичные данные
5. `4` - DONE: операция завершена

## Типичные примеры использования

### GET-запрос

```javascript
function getData(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open('GET', url);
    xhr.responseType = 'json'; // автоматическая десериализация JSON
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(xhr.response);
      } else {
        reject(new Error(`HTTP ошибка: ${xhr.status}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Сетевая ошибка'));
    };
    
    xhr.send();
  });
}

// Использование
getData('/api/users')
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### POST-запрос

```javascript
function postData(url, data) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open('POST', url);
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.responseType = 'json';
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(xhr.response);
      } else {
        reject(new Error(`HTTP ошибка: ${xhr.status}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Сетевая ошибка'));
    };
    
    xhr.send(JSON.stringify(data));
  });
}

// Использование
postData('/api/users', { name: 'Иван', email: 'ivan@example.com' })
  .then(response => console.log(response))
  .catch(error => console.error(error));
```

## Работа с FormData

XMLHttpRequest отлично работает с [[FormData]] для отправки форм и файлов:

```javascript
function uploadFile(fileInput) {
  const formData = new FormData();
  formData.append('file', fileInput.files[0]);
  
  const xhr = new XMLHttpRequest();
  
  xhr.upload.onprogress = function(event) {
    const percentComplete = (event.loaded / event.total) * 100;
    console.log(`Загрузка: ${percentComplete}%`);
  };
  
  xhr.onload = function() {
    if (xhr.status === 200) {
      console.log('Файл успешно загружен');
    } else {
      console.error('Ошибка загрузки файла');
    }
  };
  
  xhr.open('POST', '/api/upload');
  xhr.send(formData);
}
```

## Обработка ошибок

```javascript
function requestWithErrorHandling(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open('GET', url);
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(xhr.response);
      } else {
        reject(new Error(`HTTP ошибка: ${xhr.status} - ${xhr.statusText}`));
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Сетевая ошибка'));
    };
    
    xhr.ontimeout = function() {
      reject(new Error('Превышено время ожидания'));
    };
    
    xhr.timeout = 10000; // 10 секунд
    
    xhr.send();
  });
}
```

## Сравнение с современными альтернативами

| Характеристика | XMLHttpRequest | [[Fetch API]] |
|----------------|----------------|---------------|
| Синтаксис | Устаревший и громоздкий | Современный и чистый |
| Promise | Требует обертки | Встроенный |
| Читаемость | Средняя | Высокая |
| Поддержка | Все браузеры | Современные браузеры |
| Удобство | Среднее | Высокое |
| Поддержка FormData | Да | Да |
| Поддержка загрузки файлов | Да | Да (в современных версиях) |

## Практические рекомендации для российских разработчиков (2025)

### Поддержка унаследованного кода

В российской экосистеме часто встречаются проекты с унаследованным кодом, где используется XMLHttpRequest:

- Постепенная миграция на [[Fetch API]] или библиотеки типа axios
- Создание оберток для упрощения работы с XHR
- Документирование существующих XHR-запросов

### Работа с российскими API

При интеграции с российскими API через XMLHttpRequest:
- Обратите внимание на требования к заголовкам
- Учитывайте особенности аутентификации
- Проверяйте политику CORS на стороне сервера
- Используйте HTTPS для передачи чувствительных данных

### Обход ограничений CORS

Для работы с API, которые не поддерживают CORS:

```javascript
// Использование прокси-сервера
const xhr = new XMLHttpRequest();
xhr.open('GET', '/proxy/api/external-data'); // прокси на вашем сервере
xhr.send();
```

### Безопасность

- Не передавайте чувствительные данные в URL
- Используйте HTTPS для всех запросов
- Правильно устанавливайте заголовки безопасности
- Валидируйте ответы сервера перед использованием

## Современные возможности XMLHttpRequest

### Отслеживание прогресса загрузки

```javascript
function uploadWithProgress(file) {
  const xhr = new XMLHttpRequest();
  const formData = new FormData();
  
  formData.append('file', file);
  
  // Прогресс отправки
  xhr.upload.onprogress = function(event) {
    if (event.lengthComputable) {
      const percentComplete = (event.loaded / event.total) * 100;
      console.log(`Прогресс: ${percentComplete}%`);
    }
  };
  
  xhr.onload = function() {
    if (xhr.status === 200) {
      console.log('Загрузка завершена');
    }
  };
  
  xhr.open('POST', '/upload');
  xhr.send(formData);
}
```

### Таймауты и отмена

```javascript
function requestWithTimeout(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.timeout = 5000; // 5 секунд
    
    xhr.ontimeout = function() {
      reject(new Error('Время ожидания истекло'));
    };
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(xhr.response);
      } else {
        reject(new Error(`HTTP ошибка: ${xhr.status}`));
      }
    };
    
    xhr.open('GET', url);
    xhr.send();
  });
}
```

## Полезные ресурсы

- [[Fetch-API-в-HTML]] - современная альтернатива
- [[FormData-и-загрузка-файлов]] - работа с формами и файлами
- [[Обработка-ответов-и-ошибок]] - детальное рассмотрение обработки ошибок
- [[CORS-и-безопасность-запросов]] - вопросы безопасности

## Заключение

XMLHttpRequest остается важным инструментом для работы с сетевыми запросами, особенно при поддержке унаследованного кода. Хотя он уступает по удобству [[Fetch API]], понимание XHR важно для разработчиков, работающих с существующими проектами.

> [!tip] Совет
> При создании новых проектов предпочтительно использовать [[Fetch API]] или современные библиотеки, но знание XMLHttpRequest необходимо для поддержки старого кода.

> [!warning] Важно
> XMLHttpRequest устарел по сравнению с [[Fetch API]], но все еще поддерживается во всех браузерах. Используйте его только при необходимости поддержки старых браузеров или унаследованного кода.
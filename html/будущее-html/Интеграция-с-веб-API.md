---
aliases: ["Веб API интеграция", "HTML и API", "Современные API"]
tags: ["#web-api", "#integration", "#html", "#javascript", "#frontend"]
---

# Интеграция с веб API

## Обзор

Интеграция HTML с веб API становится все более тесной в 2025 году. Современные браузеры предоставляют доступ к широкому спектру API, которые можно использовать напрямую или через JavaScript для создания мощных веб-приложений. Эта интеграция позволяет HTML-элементам взаимодействовать с различными системами и сервисами, обеспечивая богатый пользовательский опыт.

## Современные Web API

### Geolocation API

API геолокации позволяет определять местоположение пользователя:

```html
<button id="getLocation">Определить местоположение</button>
<div id="location-info"></div>

<script>
async function getUserLocation() {
  try {
    const position = await navigator.geolocation.getCurrentPosition();
    const { latitude, longitude } = position.coords;
    
    document.getElementById('location-info').innerHTML = 
      `Широта: ${latitude}, Долгота: ${longitude}`;
  } catch (error) {
    console.error('Ошибка получения местоположения:', error);
  }
}

document.getElementById('getLocation').addEventListener('click', getUserLocation);
</script>
```

### Web Storage API

API локального хранения данных:

```html
<input type="text" id="inputField" placeholder="Введите данные">
<button id="saveBtn">Сохранить</button>
<button id="loadBtn">Загрузить</button>
<div id="output"></div>

<script>
document.getElementById('saveBtn').addEventListener('click', () => {
  const value = document.getElementById('inputField').value;
  localStorage.setItem('userInput', value);
});

document.getElementById('loadBtn').addEventListener('click', () => {
  const savedValue = localStorage.getItem('userInput');
  document.getElementById('output').textContent = savedValue || 'Нет сохраненных данных';
});
</script>
```

### Fetch API

Современный способ выполнения HTTP-запросов:

```html
<button id="fetchData">Загрузить данные</button>
<div id="dataContainer"></div>

<script>
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    
    document.getElementById('dataContainer').innerHTML = 
      `<pre>${JSON.stringify(data, null, 2)}</pre>`;
  } catch (error) {
    console.error('Ошибка загрузки данных:', error);
  }
}

document.getElementById('fetchData').addEventListener('click', fetchData);
</script>
```

## Интеграция с отечественными API

### Государственные API

В 2025 году российские веб-приложения активно интегрируются с государственными API:

- **Госуслуги API** - для аутентификации и получения данных граждан
- **ФНС API** - для работы с налоговыми данными
- **Росстат API** - для получения статистических данных

```html
<button id="getGovData">Получить данные из Госуслуг</button>
<div id="govDataContainer"></div>

<script>
async function getGovernmentData() {
  try {
    // Использование JWT-токена для аутенцификации
    const token = sessionStorage.getItem('govToken');
    
    const response = await fetch('/api/gov/data', {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    });
    
    if (response.ok) {
      const data = await response.json();
      displayGovernmentData(data);
    } else {
      throw new Error(`Ошибка: ${response.status}`);
    }
  } catch (error) {
    console.error('Ошибка получения государственных данных:', error);
  }
}

function displayGovernmentData(data) {
  const container = document.getElementById('govDataContainer');
  container.innerHTML = `
    <h3>Данные из Госуслуг</h3>
    <p>ФИО: ${data.fullName}</p>
    <p>ИНН: ${data.inn}</p>
    <p>Статус: ${data.status}</p>
  `;
}

document.getElementById('getGovData').addEventListener('click', getGovernmentData);
</script>
```

### Банковские API

Интеграция с российскими банками:

```html
<form id="paymentForm">
  <input type="text" name="cardNumber" placeholder="Номер карты" required>
  <input type="text" name="expiry" placeholder="Срок действия" required>
  <input type="text" name="cvv" placeholder="CVV" required>
  <button type="submit">Оплатить</button>
</form>

<script>
document.getElementById('paymentForm').addEventListener('submit', async (e) => {
  e.preventDefault();
  
  const formData = new FormData(e.target);
  const paymentData = Object.fromEntries(formData);
  
  try {
    const response = await fetch('/api/bank/process-payment', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': getCSRFToken() // Защита от CSRF
      },
      body: JSON.stringify(paymentData)
    });
    
    const result = await response.json();
    
    if (result.success) {
      alert('Платеж успешно обработан');
    } else {
      alert(`Ошибка платежа: ${result.message}`);
    }
  } catch (error) {
    console.error('Ошибка обработки платежа:', error);
  }
});
</script>
```

## Интеграция с отечественными системами

### Системы авторизации

Использование российских систем аутентификации:

```html
<div class="auth-container">
  <button id="ssoLogin">Войти через Госуслуги</button>
  <button id="qrLogin">Войти по QR-коду</button>
</div>

<script>
class RussianAuth {
  static async loginViaGosuslugi() {
    // Перенаправление на государственный SSO
    window.location.href = 'https://esia.gosuslugi.ru/...';
  }
  
  static async loginViaQR() {
    // Инициализация сканера QR-кода
    const qrScanner = new QrScanner(
      document.getElementById('qr-video'),
      result => this.handleQRResult(result)
    );
    
    await qrScanner.start();
  }
  
  static handleQRResult(result) {
    // Обработка результата QR-сканирования
    fetch('/api/auth/qr-validate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ qrData: result.data })
    })
    .then(response => response.json())
    .then(data => {
      if (data.authenticated) {
        sessionStorage.setItem('authToken', data.token);
        window.location.href = '/dashboard';
      }
    });
  }
}

document.getElementById('ssoLogin').addEventListener('click', RussianAuth.loginViaGosuslugi);
document.getElementById('qrLogin').addEventListener('click', RussianAuth.loginViaQR);
</script>
```

### Интеграция с СМЭВ

Система межведомственного электронного взаимодействия:

```javascript
class SMEVIntegration {
  static async requestCitizenData(snils) {
    const requestData = {
      service: 'citizen-info',
      identifier: snils,
      requestor: 'my-app',
      timestamp: new Date().toISOString()
    };
    
    try {
      const response = await fetch('/api/smev/request', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'X-Request-ID': generateRequestId()
        },
        body: JSON.stringify(requestData)
      });
      
      return await response.json();
    } catch (error) {
      console.error('Ошибка запроса к СМЭВ:', error);
      throw error;
    }
  }
}
```

## Безопасность интеграции

### CORS и безопасность

```html
<script>
// Настройка безопасного доступа к API
const secureApiCall = async (endpoint, options = {}) => {
  const defaultHeaders = {
    'Content-Type': 'application/json',
    'X-Requested-With': 'XMLHttpRequest',
    'X-CSRF-Token': document.querySelector('meta[name="csrf-token"]')?.content
  };
  
  const response = await fetch(endpoint, {
    ...options,
    headers: {
      ...defaultHeaders,
      ...options.headers
    },
    credentials: 'include' // Включаем куки для аутентификации
  });
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  return response.json();
};
</script>
```

### Защита персональных данных

```html
<script>
// Класс для безопасной обработки персональных данных
class PersonalDataHandler {
  static encryptData(data) {
    // Использование отечественного алгоритма шифрования
    return CryptoJS.GOST(data, 'secret-key');
  }
  
  static sanitizePersonalData(data) {
    // Маскировка чувствительных данных
    return {
      ...data,
      phone: this.maskPhone(data.phone),
      email: this.maskEmail(data.email)
    };
  }
  
  static maskPhone(phone) {
    if (!phone) return '';
    return phone.replace(/(\d{3})(\d{3})(\d{2})(\d{2})/, '+7 ($1) $2-$3-$4');
  }
  
  static maskEmail(email) {
    if (!email) return '';
    const [localPart, domain] = email.split('@');
    const maskedLocal = localPart.slice(0, 2) + '***' + localPart.slice(-1);
    return `${maskedLocal}@${domain}`;
  }
}
</script>
```

## Практические рекомендации

### Оптимизация производительности

```html
<script>
// Оптимизированная загрузка данных с API
class OptimizedAPIClient {
  constructor() {
    this.cache = new Map();
    this.pendingRequests = new Map();
  }
  
  async fetchData(url, options = {}) {
    // Проверка кэша
    if (this.cache.has(url)) {
      const cached = this.cache.get(url);
      if (Date.now() - cached.timestamp < 300000) { // 5 минут
        return cached.data;
      }
    }
    
    // Проверка на ожидающий запрос
    if (this.pendingRequests.has(url)) {
      return this.pendingRequests.get(url);
    }
    
    const request = fetch(url, options)
      .then(response => response.json())
      .then(data => {
        // Кэширование результата
        this.cache.set(url, {
          data,
          timestamp: Date.now()
        });
        this.pendingRequests.delete(url);
        return data;
      });
    
    this.pendingRequests.set(url, request);
    return request;
  }
}
</script>
```

### Обработка ошибок

```html
<script>
// Универсальный обработчик ошибок API
class APIErrorHandler {
  static async handleAPIError(error, context = '') {
    const errorInfo = {
      message: error.message,
      context,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent
    };
    
    // Логирование ошибки
    console.error('API Error:', errorInfo);
    
    // Отправка ошибки в систему мониторинга
    await fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(errorInfo)
    });
    
    // Показ пользовательского сообщения об ошибке
    this.showUserError(error);
  }
  
  static showUserError(error) {
    const errorDiv = document.createElement('div');
    errorDiv.className = 'api-error-message';
    errorDiv.innerHTML = `
      <p>Произошла ошибка при выполнении запроса</p>
      <p>Попробуйте позже или обратитесь в техподдержку</p>
    `;
    
    document.body.appendChild(errorDiv);
    
    setTimeout(() => {
      errorDiv.remove();
    }, 5000);
  }
}
</script>
```

## Связанные темы

- [[HTML6-и-новые-функции]]
- [[Web-Components-стандарты]]
- [[Совместимость-и-эволюция]]
- [[Российские-перспективы]]

## Заключение

Интеграция HTML с веб API в 2025 году требует учета как международных стандартов, так и российских особенностей. Важно обеспечивать безопасность, производительность и совместимость с отечественными системами при разработке веб-приложений.
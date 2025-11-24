---
aliases: [Web Storage API, Local Storage, Session Storage]
tags: [javascript, web-api, storage, frontend]
---

# LocalStorage и SessionStorage

## Обзор

LocalStorage и SessionStorage являются частью Web Storage API, предоставляемого браузером для хранения данных на стороне клиента. Эти механизмы позволяют сохранять пары ключ-значение в браузере без отправки данных на сервер, что делает их полезными для хранения настроек пользователя, временных данных и кэширования информации в веб-приложениях.

## Основные различия

| Характеристика | LocalStorage | SessionStorage |
|----------------|--------------|----------------|
| Время жизни данных | Постоянное (до удаления пользователем или приложения) | Сессионное (до закрытия вкладки/окна) |
| Персистентность | Сохраняются после перезапуска браузера | Удаляются при закрытии вкладки/окна |
| Область действия | Для одного домена | Для одной вкладки/окна |

## LocalStorage

### Базовое использование

```javascript
// Сохранение данных
localStorage.setItem('ключ', 'значение');

// Чтение данных
const значение = localStorage.getItem('ключ');

// Удаление данных
localStorage.removeItem('ключ');

// Очистка всего хранилища
localStorage.clear();
```

### Примеры использования

```javascript
// Сохранение настроек пользователя
localStorage.setItem('theme', 'dark');
localStorage.setItem('language', 'ru');

// Получение настроек
const theme = localStorage.getItem('theme') || 'light';
const language = localStorage.getItem('language') || 'en';

// Сохранение объекта (необходима сериализация)
const userPreferences = {
    fontSize: 14,
    notifications: true,
    lastVisit: new Date().toISOString()
};
localStorage.setItem('userPreferences', JSON.stringify(userPreferences));

// Получение и десериализация объекта
const storedPreferences = JSON.parse(localStorage.getItem('userPreferences'));
```

### Обработка ошибок

```javascript
function safeLocalStorageSet(key, value) {
    try {
        localStorage.setItem(key, value);
    } catch (e) {
        if (e instanceof DOMException && (
            // Проверка на переполнение квоты
            e.name === 'QuotaExceededError' ||
            // Firefox
            e.name === 'NS_ERROR_DOM_QUOTA_REACHED')) {
            console.error('LocalStorage переполнен');
            // Здесь можно реализовать стратегию очистки или уведомления пользователя
        } else {
            console.error('Ошибка при работе с LocalStorage:', e);
        }
    }
}
```

## SessionStorage

SessionStorage работает аналогично LocalStorage, но с ограниченным временем жизни данных.

```javascript
// Работа с SessionStorage
sessionStorage.setItem('tempData', 'временные данные');
const tempData = sessionStorage.getItem('tempData');
sessionStorage.removeItem('tempData');

// Данные в SessionStorage специфичны для вкладки
// Открытие той же страницы в новой вкладке создаст отдельное хранилище
```

## Практические применения

### Сохранение состояния приложения

```javascript
// Сохранение состояния формы при навигации
window.addEventListener('beforeunload', () => {
    const formData = {
        username: document.getElementById('username').value,
        message: document.getElementById('message').value
    };
    sessionStorage.setItem('formState', JSON.stringify(formData));
});

// Восстановление состояния формы
window.addEventListener('load', () => {
    const savedState = sessionStorage.getItem('formState');
    if (savedState) {
        const formData = JSON.parse(savedState);
        document.getElementById('username').value = formData.username;
        document.getElementById('message').value = formData.message;
    }
});
```

### Кэширование данных

```javascript
// Кэширование API-ответов с проверкой срока действия
class DataCache {
    constructor(expirationTime = 3600000) { // 1 час по умолчанию
        this.expirationTime = expirationTime;
    }

    set(key, data) {
        const item = {
            data: data,
            timestamp: Date.now(),
            expiration: Date.now() + this.expirationTime
        };
        localStorage.setItem(key, JSON.stringify(item));
    }

    get(key) {
        const itemStr = localStorage.getItem(key);
        if (!itemStr) return null;

        const item = JSON.parse(itemStr);
        if (Date.now() > item.expiration) {
            localStorage.removeItem(key);
            return null;
        }

        return item.data;
    }

    clearExpired() {
        Object.keys(localStorage).forEach(key => {
            const itemStr = localStorage.getItem(key);
            if (itemStr) {
                const item = JSON.parse(itemStr);
                if (Date.now() > item.expiration) {
                    localStorage.removeItem(key);
                }
            }
        });
    }
}

// Использование
const cache = new DataCache();
cache.set('userData', { name: 'Иван', role: 'admin' });
const userData = cache.get('userData');
```

## Ограничения и особенности

### Объем хранилища

- **Chrome, Firefox, Edge**: ~5-10 МБ на домен
- **Safari**: ~5 МБ на домен (включая cookie)
- **Internet Explorer**: ~10 МБ на домен

> [!warning]
> Превышение квоты вызывает `QuotaExceededError`. Всегда оборачивайте операции в блок `try-catch`.

### Типы данных

LocalStorage и SessionStorage поддерживают только строковые значения. Для хранения объектов и массивов используйте `JSON.stringify()` и `JSON.parse()`.

```javascript
// Правильное хранение объектов
const user = { id: 1, name: 'Алексей', email: 'alex@example.com' };
localStorage.setItem('currentUser', JSON.stringify(user));

const storedUser = JSON.parse(localStorage.getItem('currentUser'));
```

### Безопасность

> [!caution]
> Не храните конфиденциальные данные (пароли, токены доступа) в LocalStorage/SessionStorage, так как:
> - Данные доступны через JavaScript
> - Данные могут быть скомпрометированы при XSS-атаках
> - Данные сохраняются в браузере пользователя

### Синхронность

Операции с LocalStorage и SessionStorage синхронные, что может блокировать основной поток при работе с большими объемами данных.

## События Storage

При изменении данных в другом окне/вкладке генерируется событие `storage`:

```javascript
window.addEventListener('storage', (event) => {
    console.log('Ключ:', event.key);
    console.log('Старое значение:', event.oldValue);
    console.log('Новое значение:', event.newValue);
    console.log('URL документа:', event.url);
    
    // Обновление UI при изменении данных в другом окне
    if (event.key === 'theme') {
        applyTheme(event.newValue);
    }
});
```

## Совместимость с браузерами

| Браузер | Поддержка |
|---------|-----------|
| Chrome | С 4.0 |
| Firefox | С 3.5 |
| Safari | С 4.0 |
| Edge | С 12.0 |
| Internet Explorer | С 8.0 |

## Альтернативы

- [[IndexedDB]] - более мощная база данных для браузера
- [[Cookies]] - для данных, которые должны отправляться с каждым запросом
- [[WebSQL]] - устаревший API (не рекомендуется)

## Рекомендации по использованию

1. **Используйте SessionStorage для временных данных** - например, для сохранения состояния формы во время сессии
2. **Используйте LocalStorage для долгосрочных настроек** - например, тема интерфейса, язык, предпочтения пользователя
3. **Всегда проверяйте поддержку API**:
   ```javascript
   if (typeof(Storage) !== "undefined") {
       // Web Storage API поддерживается
   } else {
       // Используйте альтернативный способ хранения данных
   }
   ```
4. **Реализуйте стратегию очистки** для избежания переполнения хранилища
5. **Шифруйте чувствительные данные** при необходимости (хотя это не заменяет безопасное хранение)

## Заключение

LocalStorage и SessionStorage предоставляют простой и эффективный способ хранения данных на стороне клиента. Они идеально подходят для хранения настроек пользователя, кэширования временных данных и сохранения состояния приложения между сессиями. Однако важно учитывать их ограничения и особенности безопасности при проектировании приложения.

Для более сложных сценариев хранения данных рекомендуется рассмотреть [[IndexedDB]], который предоставляет более мощные возможности для работы с большими объемами структурированных данных.

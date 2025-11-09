# IndexedDB

IndexedDB - это низкоуровневое API для клиентского хранения значений большого объема, включая файлы/объекты бинарных данных. Это транзакционная база данных системы, аналогичная SQL, но с более широкими возможностями.

## Основные концепции

### Архитектура IndexedDB

```
Database (База данных)
├── Object Store 1 (Объектное хранилище 1)
│   ├── Index 1 (Индекс 1)
│   ├── Index 2 (Индекс 2)
│   └── Records (Записи)
├── Object Store 2 (Объектное хранилище 2)
└── Object Store 3 (Объектное хранилище 3)
```

### Основные термины

- **Database** - контейнер для объектных хранилищ
- **Object Store** - коллекция записей с ключами и значениями
- **Index** - способ индексации записей в объектном хранилище
- **Transaction** - контекст выполнения операций с БД
- **Key** - уникальный идентификатор записи
- **Value** - данные, ассоциированные с ключом

## Основные операции

### Открытие базы данных

```javascript
// Открытие/создание базы данных
function openDatabase(dbName, version = 1) {
    return new Promise((resolve, reject) => {
        const request = indexedDB.open(dbName, version);
        
        request.onerror = (event) => {
            console.error('Ошибка при открытии базы данных:', event.target.error);
            reject(event.target.error);
        };
        
        request.onsuccess = (event) => {
            const db = event.target.result;
            console.log('База данных открыта успешно');
            resolve(db);
        };
        
        // Обновление схемы при изменении версии
        request.onupgradeneeded = (event) => {
            const db = event.target.result;
            console.log(`Обновление схемы до версии ${event.oldVersion} -> ${event.newVersion}`);
            
            // Создание объектных хранилищ
            if (!db.objectStoreNames.contains('users')) {
                const store = db.createObjectStore('users', { keyPath: 'id', autoIncrement: true });
                store.createIndex('email', 'email', { unique: true });
                store.createIndex('name', 'name', { unique: false });
            }
            
            if (!db.objectStoreNames.contains('products')) {
                const store = db.createObjectStore('products', { keyPath: 'id', autoIncrement: true });
                store.createIndex('category', 'category', { unique: false });
                store.createIndex('price', 'price', { unique: false });
            }
        };
    });
}
```

### Добавление данных

```javascript
// Добавление одной записи
async function addUser(db, user) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        
        const request = store.add(user);
        
        request.onsuccess = () => {
            console.log('Пользователь добавлен с ID:', request.result);
            resolve(request.result);
        };
        
        request.onerror = (event) => {
            console.error('Ошибка при добавлении пользователя:', event.target.error);
            reject(event.target.error);
        };
    });
}

// Массовое добавление данных
async function addUsers(db, users) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        
        let addedCount = 0;
        let errorCount = 0;
        
        users.forEach(user => {
            const request = store.add(user);
            
            request.onsuccess = () => {
                addedCount++;
                if (addedCount + errorCount === users.length) {
                    resolve({ added: addedCount, errors: errorCount });
                }
            };
            
            request.onerror = () => {
                errorCount++;
                if (addedCount + errorCount === users.length) {
                    resolve({ added: addedCount, errors: errorCount });
                }
            };
        });
    });
}
```

### Чтение данных

```javascript
// Получение записи по ключу
async function getUser(db, userId) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        
        const request = store.get(userId);
        
        request.onsuccess = (event) => {
            const user = event.target.result;
            if (user) {
                console.log('Пользователь найден:', user);
                resolve(user);
            } else {
                console.log('Пользователь не найден');
                resolve(null);
            }
        };
        
        request.onerror = (event) => {
            console.error('Ошибка при получении пользователя:', event.target.error);
            reject(event.target.error);
        };
    });
}

// Получение всех записей
async function getAllUsers(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        
        const request = store.getAll();
        
        request.onsuccess = (event) => {
            console.log('Все пользователи получены:', event.target.result);
            resolve(event.target.result);
        };
        
        request.onerror = (event) => {
            console.error('Ошибка при получении пользователей:', event.target.error);
            reject(event.target.error);
        };
    });
}

// Поиск с использованием индекса
async function getUserByEmail(db, email) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const index = store.index('email');
        
        const request = index.get(email);
        
        request.onsuccess = (event) => {
            const user = event.target.result;
            if (user) {
                console.log('Пользователь найден по email:', user);
                resolve(user);
            } else {
                console.log('Пользователь с таким email не найден');
                resolve(null);
            }
        };
        
        request.onerror = (event) => {
            console.error('Ошибка при поиске пользователя по email:', event.target.error);
            reject(event.target.error);
        };
    });
}
```

### Обновление данных

```javascript
// Обновление записи
async function updateUser(db, user) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        
        const request = store.put(user);
        
        request.onsuccess = () => {
            console.log('Пользователь обновлен:', user.id);
            resolve(user.id);
        };
        
        request.onerror = (event) => {
            console.error('Ошибка при обновлении пользователя:', event.target.error);
            reject(event.target.error);
        };
    });
}
```

### Удаление данных

```javascript
// Удаление записи
async function deleteUser(db, userId) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        
        const request = store.delete(userId);
        
        request.onsuccess = () => {
            console.log('Пользователь удален:', userId);
            resolve(userId);
        };
        
        request.onerror = (event) => {
            console.error('Ошибка при удалении пользователя:', event.target.error);
            reject(event.target.error);
        };
    });
}

// Удаление всех записей
async function clearUsers(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readwrite');
        const store = transaction.objectStore('users');
        
        const request = store.clear();
        
        request.onsuccess = () => {
            console.log('Все пользователи удалены');
            resolve();
        };
        
        request.onerror = (event) => {
            console.error('Ошибка при очистке пользователей:', event.target.error);
            reject(event.target.error);
        };
    });
}
```

## Индексы и запросы

### Создание и использование индексов

```javascript
// Создание сложных индексов
function createComplexIndexes(db) {
    const transaction = db.transaction(['products'], 'versionchange');
    const store = transaction.objectStore('products');
    
    // Создание составного индекса (вспомогательная функция)
    // В IndexedDB составные индексы не поддерживаются напрямую,
    // но можно создать индекс на объекте с несколькими полями
    
    // Индекс для поиска по категории и цене
    if (!store.indexNames.contains('category_price')) {
        store.createIndex('category_price', ['category', 'price'], { unique: false });
    }
}

// Использование индексов для запросов
async function getUsersByName(db, name) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        const index = store.index('name');
        
        const request = index.getAll(name);
        
        request.onsuccess = (event) => {
            resolve(event.target.result);
        };
        
        request.onerror = (event) => {
            reject(event.target.error);
        };
    });
}
```

### Использование курсоров

```javascript
// Обход данных с помощью курсора
async function iterateUsers(db) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        
        const users = [];
        
        // Открытие курсора
        const cursorRequest = store.openCursor();
        
        cursorRequest.onsuccess = (event) => {
            const cursor = event.target.result;
            
            if (cursor) {
                users.push(cursor.value);
                console.log(`Обработка пользователя: ${cursor.value.name}`);
                cursor.continue(); // Переход к следующей записи
            } else {
                console.log('Все пользователи обработаны');
                resolve(users);
            }
        };
        
        cursorRequest.onerror = (event) => {
            reject(event.target.error);
        };
    });
}

// Обход с фильтрацией
async function getUsersWithCondition(db, condition) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction(['users'], 'readonly');
        const store = transaction.objectStore('users');
        
        const users = [];
        
        const cursorRequest = store.openCursor();
        
        cursorRequest.onsuccess = (event) => {
            const cursor = event.target.result;
            
            if (cursor) {
                if (condition(cursor.value)) {
                    users.push(cursor.value);
                }
                cursor.continue();
            } else {
                resolve(users);
            }
        };
        
        cursorRequest.onerror = (event) => {
            reject(event.target.error);
        };
    });
}
```

## Класс для управления IndexedDB

```javascript
class IndexedDBManager {
    constructor(dbName, version = 1, storesConfig = {}) {
        this.dbName = dbName;
        this.version = version;
        this.storesConfig = storesConfig;
        this.db = null;
    }
    
    async init() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, this.version);
            
            request.onerror = (event) => {
                reject(event.target.error);
            };
            
            request.onsuccess = (event) => {
                this.db = event.target.result;
                console.log(`База данных ${this.dbName} открыта`);
                resolve(this.db);
            };
            
            request.onupgradeneeded = (event) => {
                this.db = event.target.result;
                
                // Создание объектных хранилищ на основе конфигурации
                for (const [storeName, config] of Object.entries(this.storesConfig)) {
                    if (!this.db.objectStoreNames.contains(storeName)) {
                        const store = this.db.createObjectStore(storeName, config.options);
                        
                        // Создание индексов
                        if (config.indexes) {
                            for (const [indexName, indexConfig] of Object.entries(config.indexes)) {
                                store.createIndex(indexName, indexConfig.keyPath, indexConfig.options);
                            }
                        }
                    }
                }
            };
        });
    }
    
    async add(storeName, data) {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction([storeName], 'readwrite');
            const store = transaction.objectStore(storeName);
            
            const request = store.add(data);
            
            request.onsuccess = () => resolve(request.result);
            request.onerror = (event) => reject(event.target.error);
        });
    }
    
    async get(storeName, key) {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction([storeName], 'readonly');
            const store = transaction.objectStore(storeName);
            
            const request = store.get(key);
            
            request.onsuccess = (event) => resolve(event.target.result);
            request.onerror = (event) => reject(event.target.error);
        });
    }
    
    async getAll(storeName) {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction([storeName], 'readonly');
            const store = transaction.objectStore(storeName);
            
            const request = store.getAll();
            
            request.onsuccess = (event) => resolve(event.target.result);
            request.onerror = (event) => reject(event.target.error);
        });
    }
    
    async update(storeName, data) {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction([storeName], 'readwrite');
            const store = transaction.objectStore(storeName);
            
            const request = store.put(data);
            
            request.onsuccess = () => resolve(request.result);
            request.onerror = (event) => reject(event.target.error);
        });
    }
    
    async delete(storeName, key) {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction([storeName], 'readwrite');
            const store = transaction.objectStore(storeName);
            
            const request = store.delete(key);
            
            request.onsuccess = () => resolve(key);
            request.onerror = (event) => reject(event.target.error);
        });
    }
    
    async clear(storeName) {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction([storeName], 'readwrite');
            const store = transaction.objectStore(storeName);
            
            const request = store.clear();
            
            request.onsuccess = () => resolve();
            request.onerror = (event) => reject(event.target.error);
        });
    }
    
    async query(storeName, indexName, query, direction = 'next') {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction([storeName], 'readonly');
            const store = transaction.objectStore(storeName);
            const index = store.index(indexName);
            
            const results = [];
            
            const cursorRequest = index.openCursor(query, direction);
            
            cursorRequest.onsuccess = (event) => {
                const cursor = event.target.result;
                
                if (cursor) {
                    results.push(cursor.value);
                    cursor.continue();
                } else {
                    resolve(results);
                }
            };
            
            cursorRequest.onerror = (event) => {
                reject(event.target.error);
            };
        });
    }
    
    close() {
        if (this.db) {
            this.db.close();
        }
    }
    
    async deleteDatabase() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.deleteDatabase(this.dbName);
            
            request.onsuccess = () => resolve();
            request.onerror = (event) => reject(event.target.error);
        });
    }
}

// Пример использования
const dbConfig = {
    users: {
        options: { keyPath: 'id', autoIncrement: true },
        indexes: {
            email: { keyPath: 'email', options: { unique: true } },
            name: { keyPath: 'name', options: { unique: false } }
        }
    },
    products: {
        options: { keyPath: 'id', autoIncrement: true },
        indexes: {
            category: { keyPath: 'category', options: { unique: false } },
            price: { keyPath: 'price', options: { unique: false } }
        }
    }
};

const dbManager = new IndexedDBManager('MyAppDB', 1, dbConfig);
await dbManager.init();

// Использование
const userId = await dbManager.add('users', { name: 'Иван', email: 'ivan@example.com' });
const user = await dbManager.get('users', userId);
console.log('Полученный пользователь:', user);
```

## Продвинутые возможности

### Транзакции и обработка ошибок

```javascript
// Работа с транзакциями
class TransactionManager {
    constructor(db) {
        this.db = db;
    }
    
    async executeTransaction(storeNames, mode, operations) {
        return new Promise((resolve, reject) => {
            const transaction = this.db.transaction(storeNames, mode);
            
            // Обработка ошибок транзакции
            transaction.onerror = (event) => {
                console.error('Ошибка транзакции:', event.target.error);
                reject(event.target.error);
            };
            
            transaction.oncomplete = () => {
                console.log('Транзакция завершена успешно');
                resolve();
            };
            
            // Выполнение операций
            operations(transaction);
        });
    }
    
    // Атомарная операция: добавление пользователя и продукта
    async addUserAndProduct(userData, productData) {
        return this.executeTransaction(['users', 'products'], 'readwrite', (transaction) => {
            const userStore = transaction.objectStore('users');
            const productStore = transaction.objectStore('products');
            
            // Добавление пользователя
            const userRequest = userStore.add(userData);
            
            userRequest.onsuccess = (event) => {
                const userId = event.target.result;
                
                // Добавление продукта с ссылкой на пользователя
                productData.userId = userId;
                const productRequest = productStore.add(productData);
                
                productRequest.onsuccess = () => {
                    console.log('Пользователь и продукт добавлены атомарно');
                };
            };
        });
    }
}
```

### Обработка версий и миграций

```javascript
class DatabaseMigrator {
    constructor(dbName) {
        this.dbName = dbName;
    }
    
    async migrate(currentVersion, newVersion, migrations) {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(this.dbName, newVersion);
            
            request.onupgradeneeded = (event) => {
                const db = event.target.result;
                const transaction = event.target.transaction;
                
                // Выполнение миграций
                for (let version = currentVersion + 1; version <= newVersion; version++) {
                    if (migrations[version]) {
                        migrations[version](db, transaction);
                    }
                }
            };
            
            request.onsuccess = (event) => {
                resolve(event.target.result);
            };
            
            request.onerror = (event) => {
                reject(event.target.error);
            };
        });
    }
}

// Пример миграций
const migrations = {
    2: (db, transaction) => {
        // Добавление нового объектного хранилища
        const store = db.createObjectStore('orders', { keyPath: 'id', autoIncrement: true });
        store.createIndex('userId', 'userId', { unique: false });
        store.createIndex('date', 'date', { unique: false });
    },
    3: (db, transaction) => {
        // Добавление индекса к существующему хранилищу
        const store = transaction.objectStore('users');
        store.createIndex('registrationDate', 'registrationDate', { unique: false });
    }
};
```

## Практические примеры

### Кэширование данных приложения

```javascript
class DataCache {
    constructor(dbManager) {
        this.db = dbManager;
        this.cacheTTL = 5 * 60 * 1000; // 5 минут
    }
    
    async set(key, data) {
        const cacheEntry = {
            key: key,
            data: data,
            timestamp: Date.now(),
            ttl: this.cacheTTL
        };
        
        await this.db.add('cache', cacheEntry);
    }
    
    async get(key) {
        try {
            const cacheEntry = await this.db.get('cache', key);
            
            if (cacheEntry && (Date.now() - cacheEntry.timestamp) < cacheEntry.ttl) {
                console.log('Данные получены из кэша');
                return cacheEntry.data;
            } else if (cacheEntry) {
                // Удаление устаревшей записи
                await this.db.delete('cache', key);
            }
        } catch (error) {
            console.error('Ошибка при чтении из кэша:', error);
        }
        
        return null;
    }
    
    async clearExpired() {
        const allEntries = await this.db.getAll('cache');
        const now = Date.now();
        
        for (const entry of allEntries) {
            if ((now - entry.timestamp) >= entry.ttl) {
                await this.db.delete('cache', entry.key);
            }
        }
    }
}
```

### Оффлайн-приложение с синхронизацией

```javascript
class OfflineSyncManager {
    constructor(dbManager) {
        this.db = dbManager;
        this.syncQueue = [];
        this.isOnline = navigator.onLine;
        
        // Мониторинг состояния сети
        window.addEventListener('online', () => {
            this.isOnline = true;
            this.processSyncQueue();
        });
        
        window.addEventListener('offline', () => {
            this.isOnline = false;
        });
    }
    
    async saveForSync(operation, data) {
        const syncRecord = {
            operation: operation, // 'create', 'update', 'delete'
            data: data,
            timestamp: Date.now(),
            synced: false
        };
        
        const id = await this.db.add('sync_queue', syncRecord);
        
        // Если онлайн, сразу пытаемся синхронизировать
        if (this.isOnline) {
            await this.processSyncQueue();
        }
        
        return id;
    }
    
    async processSyncQueue() {
        if (!this.isOnline) return;
        
        const unsyncedRecords = await this.db.query('sync_queue', 'synced', IDBKeyRange.only(false));
        
        for (const record of unsyncedRecords) {
            try {
                // Отправка данных на сервер
                await this.sendToServer(record.operation, record.data);
                
                // Отметка как синхронизированное
                record.synced = true;
                await this.db.update('sync_queue', record);
            } catch (error) {
                console.error('Ошибка синхронизации:', error);
                // Можно добавить логику повторных попыток
            }
        }
    }
    
    async sendToServer(operation, data) {
        // Реализация отправки данных на сервер
        const response = await fetch('/api/sync', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ operation, data })
        });
        
        if (!response.ok) {
            throw new Error(`Сервер вернул ошибку: ${response.status}`);
        }
        
        return response.json();
    }
}
```

## Лучшие практики

### 1. Обработка ошибок

```javascript
// Обработка различных типов ошибок IndexedDB
class IndexedDBErrorHandler {
    static handle(error) {
        switch (error.name) {
            case 'QuotaExceededError':
                console.error('Превышено ограничение хранилища');
                // Очистка старых данных или уведомление пользователя
                break;
                
            case 'UnknownError':
                console.error('Неизвестная ошибка IndexedDB');
                break;
                
            case 'ConstraintError':
                console.error('Нарушено ограничение целостности');
                break;
                
            case 'TransactionInactiveError':
                console.error('Транзакция неактивна');
                break;
                
            default:
                console.error('Ошибка IndexedDB:', error);
        }
    }
}
```

### 2. Оптимизация производительности

```javascript
// Использование фрагментов для массовых операций
async function bulkAddOptimized(db, storeName, items) {
    return new Promise((resolve, reject) => {
        const transaction = db.transaction([storeName], 'readwrite');
        const store = transaction.objectStore(storeName);
        
        let completed = 0;
        const total = items.length;
        
        items.forEach(item => {
            const request = store.add(item);
            
            request.onsuccess = () => {
                completed++;
                if (completed === total) {
                    resolve();
                }
            };
            
            request.onerror = (event) => {
                completed++;
                console.error('Ошибка при добавлении элемента:', event.target.error);
                if (completed === total) {
                    resolve(); // Продолжаем даже при ошибках
                }
            };
        });
    });
}
```

### 3. Управление версиями

```javascript
// Управление версиями базы данных
class DatabaseVersionManager {
    constructor(currentVersion = 1) {
        this.currentVersion = currentVersion;
        this.migrations = new Map();
    }
    
    addMigration(version, migrationFunction) {
        this.migrations.set(version, migrationFunction);
    }
    
    async ensureLatestVersion(dbName) {
        // Определение текущей версии базы данных
        const currentDbVersion = await this.getCurrentVersion(dbName);
        
        if (currentDbVersion < this.currentVersion) {
            // Выполнение миграций
            for (let v = currentDbVersion + 1; v <= this.currentVersion; v++) {
                if (this.migrations.has(v)) {
                    await this.applyMigration(dbName, v);
                }
            }
        }
    }
    
    async getCurrentVersion(dbName) {
        return new Promise((resolve) => {
            const request = indexedDB.open(dbName);
            
            request.onsuccess = (event) => {
                const db = event.target.result;
                const version = db.version;
                db.close();
                resolve(version);
            };
            
            request.onerror = () => resolve(0);
        });
    }
}
```

IndexedDB предоставляет мощную возможность для хранения больших объемов структурированных данных в браузере. При правильном использовании он может значительно улучшить производительность приложений и обеспечить автономную работу.

#javascript #indexeddb #storage #database #frontend #web-development
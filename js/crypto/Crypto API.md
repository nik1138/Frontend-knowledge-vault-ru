# Crypto API

## Введение

Web Crypto API предоставляет набор криптографических функций для безопасной генерации ключей, шифрования, дешифрования, хеширования и подписи данных в веб-приложениях. В этой главе мы рассмотрим все аспекты Web Crypto API: генерация ключей, симметричное и асимметричное шифрование, хеширование, цифровые подписи и другие криптографические операции.

## Содержание

- [[Основы Web Crypto API]]
- [[Генерация ключей]]
- [[Симметричное шифрование]]
- [[Асимметричное шифрование]]
- [[Хеширование]]
- [[Цифровые подписи]]
- [[Обмен ключами]]
- [[Безопасность криптографии в браузере]]

## Основы Web Crypto API

### Введение в Web Crypto API

Web Crypto API предоставляет доступ к криптографическим примитивам в браузере, позволяя выполнять криптографические операции без отправки данных на сервер.

```javascript
// Проверка поддержки Web Crypto API
class CryptoAPIBasics {
    static isSupported() {
        return 'crypto' in window && 'subtle' in window.crypto;
    }
    
    // Получение криптографически безопасного случайного числа
    static getRandomValues(array) {
        if (!this.isSupported()) {
            throw new Error('Web Crypto API не поддерживается');
        }
        
        return window.crypto.getRandomValues(array);
    }
    
    // Генерация случайного UUID
    static generateUUID() {
        const array = new Uint8Array(16);
        this.getRandomValues(array);
        
        // Установка версии (4) и варианта (RFC4122)
        array[6] = (array[6] & 0x0f) | 0x40; // Версия 4
        array[8] = (array[8] & 0x3f) | 0x80; // Вариант RFC4122
        
        const hex = Array.from(array)
            .map(b => b.toString(16).padStart(2, '0'))
            .join('');
        
        return [
            hex.slice(0, 8),
            hex.slice(8, 12),
            hex.slice(12, 16),
            hex.slice(16, 20),
            hex.slice(20, 32)
        ].join('-');
    }
    
    // Преобразование между форматами
    static arrayBufferToBase64(buffer) {
        const bytes = new Uint8Array(buffer);
        let binary = '';
        for (let i = 0; i < bytes.byteLength; i++) {
            binary += String.fromCharCode(bytes[i]);
        }
        return btoa(binary);
    }
    
    static base64ToArrayBuffer(base64) {
        const binary = atob(base64);
        const bytes = new Uint8Array(binary.length);
        for (let i = 0; i < binary.length; i++) {
            bytes[i] = binary.charCodeAt(i);
        }
        return bytes.buffer;
    }
    
    static arrayBufferToHex(buffer) {
        const bytes = new Uint8Array(buffer);
        return Array.from(bytes)
            .map(b => b.toString(16).padStart(2, '0'))
            .join('');
    }
    
    static hexToArrayBuffer(hex) {
        const bytes = new Uint8Array(Math.ceil(hex.length / 2));
        for (let i = 0; i < bytes.length; i++) {
            bytes[i] = parseInt(hex.substr(i * 2, 2), 16);
        }
        return bytes.buffer;
    }
    
    static stringToArrayBuffer(str) {
        const encoder = new TextEncoder();
        return encoder.encode(str).buffer;
    }
    
    static arrayBufferToString(buffer) {
        const decoder = new TextDecoder();
        return decoder.decode(buffer);
    }
}

// Пример использования
console.log('Web Crypto API поддерживается:', CryptoAPIBasics.isSupported());
console.log('Случайный UUID:', CryptoAPIBasics.generateUUID());

const randomBytes = CryptoAPIBasics.getRandomValues(new Uint8Array(16));
console.log('Случайные байты:', CryptoAPIBasics.arrayBufferToHex(randomBytes.buffer));
```

### Класс для управления криптографическими операциями

```javascript
// Класс для управления криптографическими операциями
class CryptoManager {
    constructor() {
        this.isSupported = CryptoAPIBasics.isSupported();
        if (!this.isSupported) {
            throw new Error('Web Crypto API не поддерживается в этом браузере');
        }
        
        this.subtle = window.crypto.subtle;
        this.keyCache = new Map();
    }
    
    // Генерация ключа
    async generateKey(algorithm, extractable = false, keyUsages) {
        try {
            const key = await this.subtle.generateKey(algorithm, extractable, keyUsages);
            return key;
        } catch (error) {
            console.error('Ошибка генерации ключа:', error);
            throw error;
        }
    }
    
    // Импорт ключа
    async importKey(format, keyData, algorithm, extractable, keyUsages) {
        try {
            const key = await this.subtle.importKey(format, keyData, algorithm, extractable, keyUsages);
            return key;
        } catch (error) {
            console.error('Ошибка импорта ключа:', error);
            throw error;
        }
    }
    
    // Экспорт ключа
    async exportKey(format, key) {
        try {
            const exportedKey = await this.subtle.exportKey(format, key);
            return exportedKey;
        } catch (error) {
            console.error('Ошибка экспорта ключа:', error);
            throw error;
        }
    }
    
    // Получение дайджеста (хеш)
    async digest(algorithm, data) {
        try {
            const buffer = typeof data === 'string' ? 
                CryptoAPIBasics.stringToArrayBuffer(data) : 
                data;
                
            const hashBuffer = await this.subtle.digest(algorithm, buffer);
            return hashBuffer;
        } catch (error) {
            console.error('Ошибка вычисления дайджеста:', error);
            throw error;
        }
    }
    
    // Подпись данных
    async sign(algorithm, key, data) {
        try {
            const buffer = typeof data === 'string' ? 
                CryptoAPIBasics.stringToArrayBuffer(data) : 
                data;
                
            const signature = await this.subtle.sign(algorithm, key, buffer);
            return signature;
        } catch (error) {
            console.error('Ошибка подписи:', error);
            throw error;
        }
    }
    
    // Проверка подписи
    async verify(algorithm, key, signature, data) {
        try {
            const dataBuffer = typeof data === 'string' ? 
                CryptoAPIBasics.stringToArrayBuffer(data) : 
                data;
                
            const isValid = await this.subtle.verify(algorithm, key, signature, dataBuffer);
            return isValid;
        } catch (error) {
            console.error('Ошибка проверки подписи:', error);
            throw error;
        }
    }
    
    // Шифрование
    async encrypt(algorithm, key, data) {
        try {
            const buffer = typeof data === 'string' ? 
                CryptoAPIBasics.stringToArrayBuffer(data) : 
                data;
                
            const encrypted = await this.subtle.encrypt(algorithm, key, buffer);
            return encrypted;
        } catch (error) {
            console.error('Ошибка шифрования:', error);
            throw error;
        }
    }
    
    // Дешифрование
    async decrypt(algorithm, key, encryptedData) {
        try {
            const decrypted = await this.subtle.decrypt(algorithm, key, encryptedData);
            return decrypted;
        } catch (error) {
            console.error('Ошибка дешифрования:', error);
            throw error;
        }
    }
    
    // Получение ключа из пароля (PBKDF2)
    async deriveKeyFromPassword(password, salt, algorithm = 'PBKDF2') {
        // Сначала нужно создать ключ из пароля
        const passwordBuffer = CryptoAPIBasics.stringToArrayBuffer(password);
        
        // Создаем имитационный ключ
        const passwordKey = await this.importKey(
            'raw',
            passwordBuffer,
            'PBKDF2',
            false,
            ['deriveKey']
        );
        
        // Производим ключ
        const derivedKey = await this.subtle.deriveKey(
            {
                name: 'PBKDF2',
                salt: salt,
                iterations: 100000,
                hash: 'SHA-256'
            },
            passwordKey,
            { name: 'AES-GCM', length: 256 },
            false,
            ['encrypt', 'decrypt']
        );
        
        return derivedKey;
    }
    
    // Создание соли
    createSalt(length = 16) {
        const salt = new Uint8Array(length);
        CryptoAPIBasics.getRandomValues(salt);
        return salt;
    }
}

// Использование CryptoManager
const cryptoManager = new CryptoManager();
```

## Генерация ключей

### Симметричные ключи

```javascript
// Класс для генерации симметричных ключей
class SymmetricKeyGenerator {
    constructor(cryptoManager) {
        this.cryptoManager = cryptoManager;
    }
    
    // Генерация AES ключа
    async generateAESKey(length = 256) {
        const key = await this.cryptoManager.generateKey(
            {
                name: 'AES-GCM',
                length: length
            },
            true, // extractable
            ['encrypt', 'decrypt'] // key usages
        );
        
        return key;
    }
    
    // Генерация AES ключа для импорта
    async generateAESKeyForImport() {
        const key = await this.cryptoManager.generateKey(
            {
                name: 'AES-CBC',
                length: 256
            },
            true, // extractable - для возможности экспорта
            ['encrypt', 'decrypt', 'wrapKey', 'unwrapKey']
        );
        
        return key;
    }
    
    // Генерация ключа HMAC
    async generateHMACKey(hash = 'SHA-256') {
        const key = await this.cryptoManager.generateKey(
            {
                name: 'HMAC',
                hash: { name: hash },
                length: 256
            },
            true,
            ['sign', 'verify']
        );
        
        return key;
    }
    
    // Генерация ключа HKDF
    async generateHKDFKey() {
        const key = await this.cryptoManager.generateKey(
            {
                name: 'HKDF',
                hash: 'SHA-256'
            },
            false, // не extractable для безопасности
            ['deriveKey', 'deriveBits']
        );
        
        return key;
    }
    
    // Экспорт/импорт AES ключа
    async exportAESKey(key, format = 'jwk') {
        const exported = await this.cryptoManager.exportKey(format, key);
        return exported;
    }
    
    async importAESKey(keyData, format = 'jwk') {
        const key = await this.cryptoManager.importKey(
            format,
            keyData,
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        return key;
    }
    
    // Генерация ключа из пароля
    async generateKeyFromPassword(password, salt) {
        const passwordBuffer = CryptoAPIBasics.stringToArrayBuffer(password);
        
        // Создаем ключ из пароля
        const passwordKey = await this.cryptoManager.importKey(
            'raw',
            passwordBuffer,
            'PBKDF2',
            false,
            ['deriveKey']
        );
        
        // Выводим ключ с использованием PBKDF2
        const key = await this.cryptoManager.subtle.deriveKey(
            {
                name: 'PBKDF2',
                salt: salt,
                iterations: 100000,
                hash: 'SHA-256'
            },
            passwordKey,
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        return key;
    }
}

// Использование генератора симметричных ключей
const symmetricGenerator = new SymmetricKeyGenerator(cryptoManager);

async function exampleSymmetricKeys() {
    // Генерация AES ключа
    const aesKey = await symmetricGenerator.generateAESKey();
    console.log('AES ключ сгенерирован');
    
    // Экспорт ключа
    const exportedKey = await symmetricGenerator.exportAESKey(aesKey, 'jwk');
    console.log('Ключ экспортирован:', exportedKey.k?.substring(0, 10) + '...');
    
    // Импорт ключа
    const importedKey = await symmetricGenerator.importAESKey(exportedKey, 'jwk');
    console.log('Ключ импортирован');
    
    // Генерация HMAC ключа
    const hmacKey = await symmetricGenerator.generateHMACKey();
    console.log('HMAC ключ сгенерирован');
    
    // Генерация ключа из пароля
    const salt = cryptoManager.createSalt();
    const passwordKey = await symmetricGenerator.generateKeyFromPassword('myPassword123', salt);
    console.log('Ключ из пароля сгенерирован');
}
```

### Асимметричные ключи

```javascript
// Класс для генерации асимметричных ключей
class AsymmetricKeyGenerator {
    constructor(cryptoManager) {
        this.cryptoManager = cryptoManager;
    }
    
    // Генерация RSA ключей
    async generateRSAKeys(modulusLength = 2048, publicExponent = 65537) {
        const keyPair = await this.cryptoManager.generateKey(
            {
                name: 'RSA-OAEP',
                modulusLength: modulusLength,
                publicExponent: new Uint8Array([0x01, 0x00, 0x01]), // 65537 в байтах
                hash: { name: 'SHA-256' }
            },
            true, // extractable
            ['encrypt', 'decrypt'] // для RSA-OAEP
        );
        
        return keyPair;
    }
    
    // Генерация RSA ключей для подписи
    async generateRSASigningKeys(modulusLength = 2048) {
        const keyPair = await this.cryptoManager.generateKey(
            {
                name: 'RSASSA-PKCS1-v1_5',
                modulusLength: modulusLength,
                publicExponent: new Uint8Array([0x01, 0x00, 0x01]),
                hash: { name: 'SHA-256' }
            },
            true,
            ['sign', 'verify']
        );
        
        return keyPair;
    }
    
    // Генерация ECDSA ключей
    async generateECDSAKeys(namedCurve = 'P-256') {
        const keyPair = await this.cryptoManager.generateKey(
            {
                name: 'ECDSA',
                namedCurve: namedCurve
            },
            true,
            ['sign', 'verify']
        );
        
        return keyPair;
    }
    
    // Генерация ECDH ключей для обмена
    async generateECDHKeys(namedCurve = 'P-256') {
        const keyPair = await this.cryptoManager.generateKey(
            {
                name: 'ECDH',
                namedCurve: namedCurve
            },
            true,
            ['deriveKey', 'deriveBits']
        );
        
        return keyPair;
    }
    
    // Экспорт RSA ключей
    async exportRSAKey(key, format = 'jwk', keyType = 'public') {
        const exported = await this.cryptoManager.exportKey(format, key);
        return exported;
    }
    
    // Импорт RSA ключей
    async importRSAKey(keyData, format = 'jwk', keyType = 'public') {
        let algorithm;
        
        if (format === 'jwk' && keyData.kty === 'RSA') {
            algorithm = { name: 'RSA-OAEP', hash: { name: 'SHA-256' } };
        } else if (format === 'jwk' && keyData.kty === 'EC') {
            algorithm = { name: 'ECDSA', namedCurve: keyData.crv };
        }
        
        const key = await this.cryptoManager.importKey(
            format,
            keyData,
            algorithm,
            true,
            keyType === 'public' ? ['encrypt', 'verify'] : ['decrypt', 'sign']
        );
        
        return key;
    }
    
    // Генерация ключей Ed25519 (если поддерживается)
    async generateEd25519Keys() {
        // Проверяем поддержку Ed25519 (поддерживается не во всех браузерах)
        try {
            const keyPair = await this.cryptoManager.generateKey(
                {
                    name: 'Ed25519' // Поддерживается в некоторых браузерах
                },
                true,
                ['sign', 'verify']
            );
            
            return keyPair;
        } catch (error) {
            console.warn('Ed25519 не поддерживается в этом браузере');
            throw error;
        }
    }
}

// Использование генератора асимметричных ключей
const asymmetricGenerator = new AsymmetricKeyGenerator(cryptoManager);

async function exampleAsymmetricKeys() {
    // Генерация RSA ключей
    const rsaKeys = await asymmetricGenerator.generateRSAKeys();
    console.log('RSA ключи сгенерированы');
    
    // Экспорт публичного ключа
    const exportedPublic = await asymmetricGenerator.exportRSAKey(rsaKeys.publicKey, 'jwk', 'public');
    console.log('Публичный ключ экспортирован');
    
    // Генерация ECDSA ключей
    const ecdsaKeys = await asymmetricGenerator.generateECDSAKeys();
    console.log('ECDSA ключи сгенерированы');
    
    // Генерация ECDH ключей для обмена
    const ecdhKeys = await asymmetricGenerator.generateECDHKeys();
    console.log('ECDH ключи сгенерированы');
}
```

## Симметричное шифрование

### AES шифрование

```javascript
// Класс для симметричного шифрования
class SymmetricEncryption {
    constructor(cryptoManager) {
        this.cryptoManager = cryptoManager;
    }
    
    // AES-GCM шифрование
    async encryptAESGCM(data, key, additionalData = null) {
        const iv = cryptoManager.createSalt(12); // 96-битный IV для GCM
        
        const encrypted = await this.cryptoManager.subtle.encrypt(
            {
                name: 'AES-GCM',
                iv: iv,
                additionalData: additionalData ? CryptoAPIBasics.stringToArrayBuffer(additionalData) : null
            },
            key,
            typeof data === 'string' ? CryptoAPIBasics.stringToArrayBuffer(data) : data
        );
        
        // Возвращаем IV вместе с зашифрованными данными
        const result = new Uint8Array(iv.length + encrypted.byteLength);
        result.set(iv, 0);
        result.set(new Uint8Array(encrypted), iv.length);
        
        return result.buffer;
    }
    
    // AES-GCM дешифрование
    async decryptAESGCM(encryptedData, key) {
        // Извлекаем IV из начала данных (первые 12 байт)
        const iv = new Uint8Array(encryptedData, 0, 12);
        const data = new Uint8Array(encryptedData, 12);
        
        const decrypted = await this.cryptoManager.subtle.decrypt(
            {
                name: 'AES-GCM',
                iv: iv
            },
            key,
            data
        );
        
        return decrypted;
    }
    
    // AES-CBC шифрование
    async encryptAESCBC(data, key) {
        const iv = cryptoManager.createSalt(16); // 128-битный IV для CBC
        
        const encrypted = await this.cryptoManager.subtle.encrypt(
            {
                name: 'AES-CBC',
                iv: iv
            },
            key,
            typeof data === 'string' ? CryptoAPIBasics.stringToArrayBuffer(data) : data
        );
        
        // Возвращаем IV вместе с зашифрованными данными
        const result = new Uint8Array(iv.length + encrypted.byteLength);
        result.set(iv, 0);
        result.set(new Uint8Array(encrypted), iv.length);
        
        return result.buffer;
    }
    
    // AES-CBC дешифрование
    async decryptAESCBC(encryptedData, key) {
        // Извлекаем IV из начала данных (первые 16 байт)
        const iv = new Uint8Array(encryptedData, 0, 16);
        const data = new Uint8Array(encryptedData, 16);
        
        const decrypted = await this.cryptoManager.subtle.decrypt(
            {
                name: 'AES-CBC',
                iv: iv
            },
            key,
            data
        );
        
        return decrypted;
    }
    
    // AES-CTR шифрование
    async encryptAESCTR(data, key) {
        const iv = cryptoManager.createSalt(16); // 128-битный IV для CTR
        
        const encrypted = await this.cryptoManager.subtle.encrypt(
            {
                name: 'AES-CTR',
                counter: iv,
                length: 64 // длина счетчика в битах
            },
            key,
            typeof data === 'string' ? CryptoAPIBasics.stringToArrayBuffer(data) : data
        );
        
        // Возвращаем IV вместе с зашифрованными данными
        const result = new Uint8Array(iv.length + encrypted.byteLength);
        result.set(iv, 0);
        result.set(new Uint8Array(encrypted), iv.length);
        
        return result.buffer;
    }
    
    // AES-CTR дешифрование (тот же процесс, что и шифрование)
    async decryptAESCTR(encryptedData, key) {
        // Извлекаем IV из начала данных (первые 16 байт)
        const iv = new Uint8Array(encryptedData, 0, 16);
        const data = new Uint8Array(encryptedData, 16);
        
        const decrypted = await this.cryptoManager.subtle.decrypt(
            {
                name: 'AES-CTR',
                counter: iv,
                length: 64
            },
            key,
            data
        );
        
        return decrypted;
    }
    
    // Шифрование с аутентификацией (AES-GCM с дополнительными данными)
    async encryptWithAuth(data, key, additionalData) {
        return await this.encryptAESGCM(data, key, additionalData);
    }
    
    // Проверка целостности и дешифрование
    async decryptWithAuth(encryptedData, key, additionalData) {
        // Извлекаем IV
        const iv = new Uint8Array(encryptedData, 0, 12);
        const data = new Uint8Array(encryptedData, 12);
        
        try {
            const decrypted = await this.cryptoManager.subtle.decrypt(
                {
                    name: 'AES-GCM',
                    iv: iv,
                    additionalData: CryptoAPIBasics.stringToArrayBuffer(additionalData)
                },
                key,
                data
            );
            
            return decrypted;
        } catch (error) {
            throw new Error('Аутентификация не удалась - данные могли быть изменены');
        }
    }
}

// Использование симметричного шифрования
const symmetricEncryption = new SymmetricEncryption(cryptoManager);

async function exampleSymmetricEncryption() {
    // Генерация ключа
    const key = await symmetricGenerator.generateAESKey();
    
    // Тестовые данные
    const originalData = 'Секретные данные для шифрования';
    
    // AES-GCM шифрование/дешифрование
    const encryptedGCM = await symmetricEncryption.encryptAESGCM(originalData, key);
    const decryptedGCM = await symmetricEncryption.decryptAESGCM(encryptedGCM, key);
    const decryptedStringGCM = CryptoAPIBasics.arrayBufferToString(decryptedGCM);
    
    console.log('Оригинал:', originalData);
    console.log('GCM Расшифровано:', decryptedStringGCM);
    console.log('GCM Совпадает:', originalData === decryptedStringGCM);
    
    // AES-CBC шифрование/дешифрование
    const encryptedCBC = await symmetricEncryption.encryptAESCBC(originalData, key);
    const decryptedCBC = await symmetricEncryption.decryptAESCBC(encryptedCBC, key);
    const decryptedStringCBC = CryptoAPIBasics.arrayBufferToString(decryptedCBC);
    
    console.log('CBC Расшифровано:', decryptedStringCBC);
    console.log('CBC Совпадает:', originalData === decryptedStringCBC);
    
    // Шифрование с аутентификацией
    const authData = 'Дополнительные данные для аутентификации';
    const encryptedAuth = await symmetricEncryption.encryptWithAuth(originalData, key, authData);
    
    try {
        const decryptedAuth = await symmetricEncryption.decryptWithAuth(encryptedAuth, key, authData);
        const decryptedAuthString = CryptoAPIBasics.arrayBufferToString(decryptedAuth);
        console.log('С аутентификацией расшифровано:', decryptedAuthString);
    } catch (error) {
        console.error('Ошибка аутентификации:', error.message);
    }
}
```

## Асимметричное шифрование

### RSA шифрование

```javascript
// Класс для асимметричного шифрования
class AsymmetricEncryption {
    constructor(cryptoManager) {
        this.cryptoManager = cryptoManager;
    }
    
    // RSA-OAEP шифрование
    async encryptRSA(data, publicKey) {
        const encrypted = await this.cryptoManager.subtle.encrypt(
            {
                name: 'RSA-OAEP'
            },
            publicKey,
            typeof data === 'string' ? CryptoAPIBasics.stringToArrayBuffer(data) : data
        );
        
        return encrypted;
    }
    
    // RSA-OAEP дешифрование
    async decryptRSA(encryptedData, privateKey) {
        const decrypted = await this.cryptoManager.subtle.decrypt(
            {
                name: 'RSA-OAEP'
            },
            privateKey,
            encryptedData
        );
        
        return decrypted;
    }
    
    // RSA схема с гибридным шифрованием
    async hybridEncrypt(data, publicKey, symmetricAlgorithm = 'AES-GCM') {
        // Генерируем случайный симметричный ключ
        const symmetricKey = await cryptoManager.generateKey(
            { name: symmetricAlgorithm, length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        // Шифруем данные симметричным ключом
        const iv = cryptoManager.createSalt(12);
        const encryptedData = await cryptoManager.subtle.encrypt(
            {
                name: symmetricAlgorithm,
                iv: iv
            },
            symmetricKey,
            typeof data === 'string' ? CryptoAPIBasics.stringToArrayBuffer(data) : data
        );
        
        // Шифруем симметричный ключ асимметричным ключом
        const encryptedKey = await this.encryptRSA(
            await cryptoManager.exportKey('raw', symmetricKey),
            publicKey
        );
        
        // Возвращаем зашифрованный ключ, IV и зашифрованные данные
        const result = {
            encryptedKey: Array.from(new Uint8Array(encryptedKey)),
            iv: Array.from(new Uint8Array(iv)),
            encryptedData: Array.from(new Uint8Array(encryptedData))
        };
        
        return result;
    }
    
    // Гибридное дешифрование
    async hybridDecrypt(encryptedPackage, privateKey) {
        // Дешифруем симметричный ключ
        const decryptedKeyBuffer = await this.decryptRSA(
            new Uint8Array(encryptedPackage.encryptedKey).buffer,
            privateKey
        );
        
        // Импортируем симметричный ключ
        const symmetricKey = await cryptoManager.importKey(
            'raw',
            decryptedKeyBuffer,
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        // Дешифруем данные
        const decryptedData = await cryptoManager.subtle.decrypt(
            {
                name: 'AES-GCM',
                iv: new Uint8Array(encryptedPackage.iv)
            },
            symmetricKey,
            new Uint8Array(encryptedPackage.encryptedData).buffer
        );
        
        return decryptedData;
    }
    
    // Обмен ключами с использованием ECDH
    async performECDH(privateKey, publicKey) {
        const sharedSecret = await this.cryptoManager.subtle.deriveBits(
            {
                name: 'ECDH',
                public: publicKey
            },
            privateKey,
            256 // длина в битах
        );
        
        return sharedSecret;
    }
    
    // Создание общего ключа из общего секрета
    async deriveKeyFromSecret(sharedSecret, algorithm = 'HKDF') {
        const baseKey = await this.cryptoManager.importKey(
            'raw',
            sharedSecret,
            algorithm,
            false,
            ['deriveKey']
        );
        
        const derivedKey = await this.cryptoManager.subtle.deriveKey(
            {
                name: algorithm,
                hash: 'SHA-256',
                info: CryptoAPIBasics.stringToArrayBuffer('key-derivation'),
                salt: cryptoManager.createSalt()
            },
            baseKey,
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        return derivedKey;
    }
}

// Использование асимметричного шифрования
const asymmetricEncryption = new AsymmetricEncryption(cryptoManager);

async function exampleAsymmetricEncryption() {
    // Генерация RSA ключей
    const rsaKeys = await asymmetricGenerator.generateRSAKeys();
    
    // Тестовые данные (ограничены по размеру для RSA)
    const originalData = 'Короткие данные для RSA шифрования';
    
    // RSA шифрование/дешифрование
    const encryptedRSA = await asymmetricEncryption.encryptRSA(originalData, rsaKeys.publicKey);
    const decryptedRSA = await asymmetricEncryption.decryptRSA(encryptedRSA, rsaKeys.privateKey);
    const decryptedStringRSA = CryptoAPIBasics.arrayBufferToString(decryptedRSA);
    
    console.log('RSA Оригинал:', originalData);
    console.log('RSA Расшифровано:', decryptedStringRSA);
    console.log('RSA Совпадает:', originalData === decryptedStringRSA);
    
    // Гибридное шифрование (для больших данных)
    const largeData = 'Длинные данные для гибридного шифрования, которые не поместятся в RSA ключ';
    const hybridEncrypted = await asymmetricEncryption.hybridEncrypt(largeData, rsaKeys.publicKey);
    const hybridDecrypted = await asymmetricEncryption.hybridDecrypt(hybridEncrypted, rsaKeys.privateKey);
    const hybridDecryptedString = CryptoAPIBasics.arrayBufferToString(hybridDecrypted);
    
    console.log('Гибридное совпадает:', largeData === hybridDecryptedString);
    
    // ECDH обмен ключами
    const ecdhKeys1 = await asymmetricGenerator.generateECDHKeys();
    const ecdhKeys2 = await asymmetricGenerator.generateECDHKeys();
    
    const secret1 = await asymmetricEncryption.performECDH(ecdhKeys1.privateKey, ecdhKeys2.publicKey);
    const secret2 = await asymmetricEncryption.performECDH(ecdhKeys2.privateKey, ecdhKeys1.publicKey);
    
    console.log('ECDH секреты совпадают:', CryptoAPIBasics.arrayBufferToHex(secret1) === 
                                      CryptoAPIBasics.arrayBufferToHex(secret2));
}
```

## Хеширование

### Операции хеширования

```javascript
// Класс для операций хеширования
class HashOperations {
    constructor(cryptoManager) {
        this.cryptoManager = cryptoManager;
    }
    
    // Вычисление хеша
    async hash(data, algorithm = 'SHA-256') {
        const buffer = typeof data === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(data) : 
            data;
            
        const hashBuffer = await this.cryptoManager.digest(algorithm, buffer);
        return hashBuffer;
    }
    
    // Вычисление хеша и возврат в hex формате
    async hashToHex(data, algorithm = 'SHA-256') {
        const hashBuffer = await this.hash(data, algorithm);
        return CryptoAPIBasics.arrayBufferToHex(hashBuffer);
    }
    
    // Вычисление хеша и возврат в base64 формате
    async hashToBase64(data, algorithm = 'SHA-256') {
        const hashBuffer = await this.hash(data, algorithm);
        return CryptoAPIBasics.arrayBufferToBase64(hashBuffer);
    }
    
    // HMAC (хеш с ключом)
    async hmac(data, key, hashAlgorithm = 'SHA-256') {
        const buffer = typeof data === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(data) : 
            data;
            
        const signature = await this.cryptoManager.subtle.sign(
            {
                name: 'HMAC',
                hash: { name: hashAlgorithm }
            },
            key,
            buffer
        );
        
        return signature;
    }
    
    // Проверка HMAC
    async verifyHmac(data, key, signature, hashAlgorithm = 'SHA-256') {
        const buffer = typeof data === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(data) : 
            data;
            
        const isValid = await this.cryptoManager.subtle.verify(
            {
                name: 'HMAC',
                hash: { name: hashAlgorithm }
            },
            key,
            signature,
            buffer
        );
        
        return isValid;
    }
    
    // PBKDF2 для хеширования паролей
    async pbkdf2(password, salt, iterations = 100000, hash = 'SHA-256') {
        // Создаем ключ из пароля
        const passwordBuffer = CryptoAPIBasics.stringToArrayBuffer(password);
        const passwordKey = await this.cryptoManager.importKey(
            'raw',
            passwordBuffer,
            'PBKDF2',
            false,
            ['deriveBits']
        );
        
        // Выводим биты с использованием PBKDF2
        const derivedBits = await this.cryptoManager.subtle.deriveBits(
            {
                name: 'PBKDF2',
                salt: salt,
                iterations: iterations,
                hash: { name: hash }
            },
            passwordKey,
            256 // 256 бит = 32 байта
        );
        
        return derivedBits;
    }
    
    // HKDF для деривации ключей
    async hkdf(inputKeyMaterial, salt, info, length = 256) {
        // Импортируем ключевой материал
        const ikmBuffer = typeof inputKeyMaterial === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(inputKeyMaterial) : 
            inputKeyMaterial;
            
        const ikmKey = await this.cryptoManager.importKey(
            'raw',
            ikmBuffer,
            'HKDF',
            false,
            ['deriveKey']
        );
        
        // Выводим ключ с использованием HKDF
        const derivedKey = await this.cryptoManager.subtle.deriveKey(
            {
                name: 'HKDF',
                hash: 'SHA-256',
                salt: salt,
                info: typeof info === 'string' ? 
                    CryptoAPIBasics.stringToArrayBuffer(info) : 
                    info
            },
            ikmKey,
            { name: 'AES-GCM', length: length },
            true,
            ['encrypt', 'decrypt']
        );
        
        return derivedKey;
    }
    
    // Создание хеша файла (в браузере - для Blob/File объектов)
    async hashFile(file, algorithm = 'SHA-256') {
        return new Promise((resolve, reject) => {
            const = new FileReader(); = new FileReader();
            
            reader.onload = async (event) => {
                try {
                    const arrayBuffer = event.target.result;
                    const hashBuffer = await this.hash(arrayBuffer, algorithm);
                    resolve(CryptoAPIBasics.arrayBufferToHex(hashBuffer));
                } catch (error) {
                    reject(error);
                }
            };
            
            reader.onerror = () => reject(new Error('Ошибка чтения файла'));
            reader.readAsArrayBuffer(file);
        });
    }
    
    // Сравнение хешей (безопасное, устойчивое к timing атакам)
    async secureCompare(hash1, hash2) {
        if (hash1.byteLength !== hash2.byteLength) {
            return false;
        }
        
        const arr1 = new Uint8Array(hash1);
        const arr2 = new Uint8Array(hash2);
        
        let result = 0;
        for (let i = 0; i < arr1.length; i++) {
            result |= arr1[i] ^ arr2[i];
        }
        
        return result === 0;
    }
}

// Использование операций хеширования
const hashOperations = new HashOperations(cryptoManager);

async function exampleHashOperations() {
    const testData = 'Данные для хеширования';
    
    // SHA-256 хеширование
    const hash256 = await hashOperations.hashToHex(testData, 'SHA-256');
    console.log('SHA-256:', hash256);
    
    // SHA-1 хеширование (не рекомендуется для криптографии, но иногда используется)
    const hash1 = await hashOperations.hashToHex(testData, 'SHA-1');
    console.log('SHA-1:', hash1);
    
    // SHA-512 хеширование
    const hash512 = await hashOperations.hashToHex(testData, 'SHA-512');
    console.log('SHA-512:', hash512);
    
    // HMAC
    const hmacKey = await symmetricGenerator.generateHMACKey();
    const hmac = await hashOperations.hmac(testData, hmacKey);
    const hmacHex = CryptoAPIBasics.arrayBufferToHex(hmac);
    console.log('HMAC:', hmacHex);
    
    // Проверка HMAC
    const isValid = await hashOperations.verifyHmac(testData, hmacKey, hmac);
    console.log('HMAC валидация:', isValid);
    
    // PBKDF2 для пароля
    const salt = cryptoManager.createSalt();
    const pbkdf2Hash = await hashOperations.pbkdf2('myPassword123', salt);
    const pbkdf2Hex = CryptoAPIBasics.arrayBufferToHex(pbkdf2Hash);
    console.log('PBKDF2:', pbkdf2Hex);
    
    // HKDF
    const ikm = cryptoManager.createSalt(); // исходный ключевой материал
    const hkdfKey = await hashOperations.hkdf(ikm, salt, 'application-key');
    console.log('HKDF ключ сгенерирован');
    
    // Безопасное сравнение
    const hashA = await hashOperations.hash('same data');
    const hashB = await hashOperations.hash('same data');
    const hashC = await hashOperations.hash('different data');
    
    console.log('Безопасное сравнение (одинаковые):', await hashOperations.secureCompare(hashA, hashB));
    console.log('Безопасное сравнение (разные):', await hashOperations.secureCompare(hashA, hashC));
}
```

## Цифровые подписи

### Создание и проверка подписей

```javascript
// Класс для работы с цифровыми подписями
class DigitalSignatures {
    constructor(cryptoManager) {
        this.cryptoManager = cryptoManager;
    }
    
    // Подпись данных с использованием RSASSA-PKCS1-v1_5
    async signRSASSA(data, privateKey, hash = 'SHA-256') {
        const buffer = typeof data === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(data) : 
            data;
            
        const signature = await this.cryptoManager.subtle.sign(
            {
                name: 'RSASSA-PKCS1-v1_5',
                hash: { name: hash }
            },
            privateKey,
            buffer
        );
        
        return signature;
    }
    
    // Проверка RSASSA подписи
    async verifyRSASSA(data, signature, publicKey, hash = 'SHA-256') {
        const buffer = typeof data === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(data) : 
            data;
            
        const isValid = await this.cryptoManager.subtle.verify(
            {
                name: 'RSASSA-PKCS1-v1_5',
                hash: { name: hash }
            },
            publicKey,
            signature,
            buffer
        );
        
        return isValid;
    }
    
    // Подпись данных с использованием ECDSA
    async signECDSA(data, privateKey, hash = 'SHA-256') {
        const buffer = typeof data === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(data) : 
            data;
            
        const signature = await this.cryptoManager.subtle.sign(
            {
                name: 'ECDSA',
                hash: { name: hash }
            },
            privateKey,
            buffer
        );
        
        return signature;
    }
    
    // Проверка ECDSA подписи
    async verifyECDSA(data, signature, publicKey, hash = 'SHA-256') {
        const buffer = typeof data === 'string' ? 
            CryptoAPIBasics.stringToArrayBuffer(data) : 
            data;
            
        const isValid = await this.cryptoManager.subtle.verify(
            {
                name: 'ECDSA',
                hash: { name: hash }
            },
            publicKey,
            signature,
            buffer
        );
        
        return isValid;
    }
    
    // Создание подписи с сертификатом (упрощенно)
    async createSignedDocument(documentData, privateKey, publicKey, algorithm = 'RSASSA-PKCS1-v1_5') {
        // Создаем хеш документа
        const documentHash = await hashOperations.hash(documentData);
        
        // Подписываем хеш
        const signature = await this.cryptoManager.subtle.sign(
            {
                name: algorithm,
                hash: { name: 'SHA-256' }
            },
            privateKey,
            documentHash
        );
        
        // Возвращаем документ с подписью
        return {
            data: documentData,
            signature: Array.from(new Uint8Array(signature)),
            publicKey: await asymmetricGenerator.exportRSAKey(publicKey, 'jwk', 'public'),
            timestamp: Date.now()
        };
    }
    
    // Проверка подписанного документа
    async verifySignedDocument(signedDocument, algorithm = 'RSASSA-PKCS1-v1_5') {
        // Восстанавливаем ключ
        const publicKey = await asymmetricGenerator.importRSAKey(
            signedDocument.publicKey, 
            'jwk', 
            'public'
        );
        
        // Создаем хеш документа
        const documentHash = await hashOperations.hash(signedDocument.data);
        
        // Проверяем подпись
        const isValid = await this.cryptoManager.subtle.verify(
            {
                name: algorithm,
                hash: { name: 'SHA-256' }
            },
            publicKey,
            new Uint8Array(signedDocument.signature).buffer,
            documentHash
        );
        
        return isValid;
    }
    
    // Подпись с использованием EdDSA (если поддерживается)
    async signEdDSA(data, privateKey) {
        try {
            const buffer = typeof data === 'string' ? 
                CryptoAPIBasics.stringToArrayBuffer(data) : 
                data;
                
            const signature = await this.cryptoManager.subtle.sign(
                {
                    name: 'EdDSA'
                },
                privateKey,
                buffer
            );
            
            return signature;
        } catch (error) {
            console.warn('EdDSA не поддерживается');
            throw error;
        }
    }
    
    // Проверка EdDSA подписи
    async verifyEdDSA(data, signature, publicKey) {
        try {
            const buffer = typeof data === 'string' ? 
                CryptoAPIBasics.stringToArrayBuffer(data) : 
                data;
                
            const isValid = await this.cryptoManager.subtle.verify(
                {
                    name: 'EdDSA'
                },
                publicKey,
                signature,
                buffer
            );
            
            return isValid;
        } catch (error) {
            console.warn('EdDSA не поддерживается');
            throw error;
        }
    }
}

// Использование цифровых подписей
const digitalSignatures = new DigitalSignatures(cryptoManager);

async function exampleDigitalSignatures() {
    // Генерация RSA ключей для подписи
    const signingKeys = await asymmetricGenerator.generateRSASigningKeys();
    
    const documentData = 'Важный документ для подписания';
    
    // Создание подписи RSA
    const rsaSignature = await digitalSignatures.signRSASSA(documentData, signingKeys.privateKey);
    const rsaIsValid = await digitalSignatures.verifyRSASSA(documentData, rsaSignature, signingKeys.publicKey);
    
    console.log('RSA подпись действительна:', rsaIsValid);
    
    // Генерация ECDSA ключей
    const ecdsaKeys = await asymmetricGenerator.generateECDSAKeys();
    
    // Создание подписи ECDSA
    const ecdsaSignature = await digitalSignatures.signECDSA(documentData, ecdsaKeys.privateKey);
    const ecdsaIsValid = await digitalSignatures.verifyECDSA(documentData, ecdsaSignature, ecdsaKeys.publicKey);
    
    console.log('ECDSA подпись действительна:', ecdsaIsValid);
    
    // Создание подписанного документа
    const signedDoc = await digitalSignatures.createSignedDocument(
        documentData, 
        signingKeys.privateKey, 
        signingKeys.publicKey
    );
    
    // Проверка подписанного документа
    const docIsValid = await digitalSignatures.verifySignedDocument(signedDoc);
    console.log('Подписанный документ действителен:', docIsValid);
}
```

## Примеры из промышленной разработки

### Система безопасного хранения паролей

```javascript
// Класс для безопасного хранения паролей
class SecurePasswordStorage {
    constructor() {
        this.hashOperations = new HashOperations(cryptoManager);
    }
    
    // Хеширование пароля с солью
    async hashPassword(password) {
        const salt = cryptoManager.createSalt(32); // 32-байтовая соль
        const iterations = 100000; // количество итераций PBKDF2
        
        const hashedPassword = await this.hashOperations.pbkdf2(password, salt, iterations);
        
        return {
            hash: Array.from(new Uint8Array(hashedPassword)),
            salt: Array.from(salt),
            iterations: iterations,
            algorithm: 'PBKDF2-SHA256'
        };
    }
    
    // Проверка пароля
    async verifyPassword(password, storedHashData) {
        const { hash: storedHash, salt, iterations } = storedHashData;
        
        const calculatedHash = await this.hashOperations.pbkdf2(
            password, 
            new Uint8Array(salt), 
            iterations
        );
        
        // Безопасное сравнение хешей
        return await this.hashOperations.secureCompare(
            calculatedHash, 
            new Uint8Array(storedHash).buffer
        );
    }
    
    // Обновление хеша пароля (с новыми параметрами)
    async updatePasswordHash(password, oldHashData) {
        // Проверяем старый пароль
        const isValid = await this.verifyPassword(password, oldHashData);
        if (!isValid) {
            throw new Error('Неверный пароль');
        }
        
        // Создаем новый хеш с улучшенными параметрами
        return await this.hashPassword(password);
    }
    
    // Генерация случайного пароля
    generateRandomPassword(length = 12) {
        const charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()_+-=[]{}|;:,.<>?';
        let password = '';
        
        const randomBytes = CryptoAPIBasics.getRandomValues(new Uint8Array(length));
        for (let i = 0; i < length; i++) {
            password += charset[randomBytes[i] % charset.length];
        }
        
        return password;
    }
}

// Использование системы хранения паролей
const passwordStorage = new SecurePasswordStorage();

async function examplePasswordStorage() {
    const password = 'MySecurePassword123!';
    
    // Хеширование пароля
    const hashedPassword = await passwordStorage.hashPassword(password);
    console.log('Пароль захеширован');
    
    // Проверка пароля
    const isValid = await passwordStorage.verifyPassword(password, hashedPassword);
    console.log('Проверка пароля:', isValid);
    
    // Проверка с неправильным паролем
    const isInvalid = await passwordStorage.verifyPassword('WrongPassword', hashedPassword);
    console.log('Проверка неправильного пароля:', isInvalid);
    
    // Генерация случайного пароля
    const randomPassword = passwordStorage.generateRandomPassword(16);
    console.log('Случайный пароль:', randomPassword);
}
```

### Система шифрования данных пользователя

```javascript
// Класс для шифрования данных пользователя
class UserDataEncryption {
    constructor() {
        this.symmetricEncryption = new SymmetricEncryption(cryptoManager);
        this.asymmetricEncryption = new AsymmetricEncryption(cryptoManager);
        this.hashOperations = new HashOperations(cryptoManager);
    }
    
    // Создание ключа шифрования из пароля пользователя
    async createEncryptionKeyFromPassword(password, salt) {
        // Используем PBKDF2 для создания ключа из пароля
        const keyMaterial = await this.hashOperations.pbkdf2(password, salt, 100000);
        
        // Импортируем как симметричный ключ
        const encryptionKey = await cryptoManager.importKey(
            'raw',
            keyMaterial,
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        return encryptionKey;
    }
    
    // Шифрование чувствительных данных
    async encryptSensitiveData(data, password) {
        const salt = cryptoManager.createSalt(32);
        const key = await this.createEncryptionKeyFromPassword(password, salt);
        
        // Шифруем данные
        const encrypted = await this.symmetricEncryption.encryptAESGCM(data, key);
        
        // Возвращаем зашифрованные данные с солью
        return {
            encryptedData: Array.from(new Uint8Array(encrypted)),
            salt: Array.from(salt),
            format: 'AES-GCM'
        };
    }
    
    // Дешифрование чувствительных данных
    async decryptSensitiveData(encryptedPackage, password) {
        const { encryptedData, salt } = encryptedPackage;
        
        // Восстанавливаем ключ из пароля
        const key = await this.createEncryptionKeyFromPassword(
            password, 
            new Uint8Array(salt)
        );
        
        // Дешифруем данные
        const decrypted = await this.symmetricEncryption.decryptAESGCM(
            new Uint8Array(encryptedData).buffer, 
            key
        );
        
        return CryptoAPIBasics.arrayBufferToString(decrypted);
    }
    
    // Шифрование данных с использованием асимметричного ключа
    async encryptWithPublicKey(data, publicKey) {
        // Генерируем случайный симметричный ключ
        const symmetricKey = await cryptoManager.generateKey(
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        // Шифруем данные симметричным ключом
        const encryptedData = await this.symmetricEncryption.encryptAESGCM(data, symmetricKey);
        
        // Шифруем симметричный ключ асимметричным ключом
        const encryptedKey = await this.asymmetricEncryption.encryptRSA(
            await cryptoManager.exportKey('raw', symmetricKey),
            publicKey
        );
        
        return {
            encryptedData: Array.from(new Uint8Array(encryptedData)),
            encryptedKey: Array.from(new Uint8Array(encryptedKey)),
            format: 'RSA-AES-GCM'
        };
    }
    
    // Создание защищенного контейнера для данных
    async createSecureContainer(data, password) {
        // Хешируем пароль для аутентификации
        const authHash = await this.hashOperations.hash(password);
        
        // Шифруем данные
        const encryptedData = await this.encryptSensitiveData(data, password);
        
        // Создаем подпись для целостности
        const signature = await digitalSignatures.signRSASSA(
            JSON.stringify(encryptedData), 
            rsaKeys.privateKey
        );
        
        return {
            encryptedData,
            authHash: Array.from(new Uint8Array(authHash)),
            signature: Array.from(new Uint8Array(signature)),
            timestamp: Date.now(),
            version: '1.0'
        };
    }
    
    // Проверка и расшифровка защищенного контейнера
    async openSecureContainer(container, password) {
        // Проверяем аутентификацию
        const providedAuthHash = await this.hashOperations.hash(password);
        const isValidAuth = await this.hashOperations.secureCompare(
            providedAuthHash,
            new Uint8Array(container.authHash).buffer
        );
        
        if (!isValidAuth) {
            throw new Error('Неверный пароль');
        }
        
        // Проверяем подпись для целостности
        const signatureValid = await digitalSignatures.verifyRSASSA(
            JSON.stringify(container.encryptedData),
            new Uint8Array(container.signature).buffer,
            rsaKeys.publicKey
        );
        
        if (!signatureValid) {
            throw new Error('Нарушена целостность данных');
        }
        
        // Расшифровываем данные
        return await this.decryptSensitiveData(container.encryptedData, password);
    }
}

// Использование системы шифрования данных
const dataEncryption = new UserDataEncryption();
let rsaKeys; // Определяем в области видимости

async function exampleDataEncryption() {
    // Сначала генерируем RSA ключи
    rsaKeys = await asymmetricGenerator.generateRSAKeys();
    
    const sensitiveData = 'Конфиденциальная информация пользователя';
    const password = 'UserSecurePassword123!';
    
    // Шифрование данных
    const encrypted = await dataEncryption.encryptSensitiveData(sensitiveData, password);
    console.log('Данные зашифрованы');
    
    // Дешифрование данных
    const decrypted = await dataEncryption.decryptSensitiveData(encrypted, password);
    console.log('Дешифрованные данные:', decrypted);
    console.log('Совпадение:', sensitiveData === decrypted);
    
    // Создание защищенного контейнера
    const container = await dataEncryption.createSecureContainer(sensitiveData, password);
    console.log('Защищенный контейнер создан');
    
    // Открытие защищенного контейнера
    const openedData = await dataEncryption.openSecureContainer(container, password);
    console.log('Открытые данные:', openedData);
}
```

### Система аутентификации с токенами

```javascript
// Класс для системы аутентификации
class AuthenticationSystem {
    constructor() {
        this.hashOperations = new HashOperations(cryptoManager);
        this.symmetricEncryption = new SymmetricEncryption(cryptoManager);
    }
    
    // Создание JWT-подобного токена
    async createToken(payload, secret) {
        // Создаем заголовок
        const header = {
            alg: 'HS256',
            typ: 'JWT'
        };
        
        // Кодируем заголовок и payload в base64url
        const headerBase64 = this.base64UrlEncode(JSON.stringify(header));
        const payloadBase64 = this.base64UrlEncode(JSON.stringify({
            ...payload,
            iat: Math.floor(Date.now() / 1000), // время выдачи
            exp: Math.floor(Date.now() / 1000) + 3600 // время истечения (1 час)
        }));
        
        // Создаем строку для подписи
        const signingInput = `${headerBase64}.${payloadBase64}`;
        
        // Создаем HMAC подпись
        const secretBuffer = CryptoAPIBasics.stringToArrayBuffer(secret);
        const signingKey = await cryptoManager.importKey(
            'raw',
            secretBuffer,
            { name: 'HMAC', hash: 'SHA-256' },
            false,
            ['sign']
        );
        
        const signature = await cryptoManager.subtle.sign(
            { name: 'HMAC', hash: { name: 'SHA-256' } },
            signingKey,
            CryptoAPIBasics.stringToArrayBuffer(signingInput)
        );
        
        // Кодируем подпись
        const signatureBase64 = this.base64UrlEncode(
            String.fromCharCode(...new Uint8Array(signature))
        );
        
        // Возвращаем токен
        return `${signingInput}.${signatureBase64}`;
    }
    
    // Проверка токена
    async verifyToken(token, secret) {
        const [headerBase64, payloadBase64, signatureBase64] = token.split('.');
        
        if (!headerBase64 || !payloadBase64 || !signatureBase64) {
            throw new Error('Неверный формат токена');
        }
        
        // Проверяем подпись
        const signingInput = `${headerBase64}.${payloadBase64}`;
        const expectedSignature = await this.createHmacSignature(signingInput, secret);
        
        const actualSignature = this.base64UrlDecode(signatureBase64);
        
        const isValid = await this.hashOperations.secureCompare(
            expectedSignature,
            CryptoAPIBasics.stringToArrayBuffer(actualSignature).buffer
        );
        
        if (!isValid) {
            throw new Error('Неверная подпись токена');
        }
        
        // Проверяем срок действия
        const payload = JSON.parse(this.base64UrlDecode(payloadBase64));
        const now = Math.floor(Date.now() / 1000);
        
        if (payload.exp && payload.exp < now) {
            throw new Error('Токен истек');
        }
        
        return payload;
    }
    
    // Вспомогательные методы для JWT
    base64UrlEncode(str) {
        return btoa(str)
            .replace(/\+/g, '-')
            .replace(/\//g, '_')
            .replace(/=/g, '');
    }
    
    base64UrlDecode(str) {
        // Восстанавливаем стандартный base64
        str = str.replace(/-/g, '+').replace(/_/g, '/');
        // Добавляем padding если нужно
        while (str.length % 4) str += '=';
        return atob(str);
    }
    
    async createHmacSignature(data, secret) {
        const secretBuffer = CryptoAPIBasics.stringToArrayBuffer(secret);
        const signingKey = await cryptoManager.importKey(
            'raw',
            secretBuffer,
            { name: 'HMAC', hash: 'SHA-256' },
            false,
            ['sign']
        );
        
        const signature = await cryptoManager.subtle.sign(
            { name: 'HMAC', hash: { name: 'SHA-256' } },
            signingKey,
            CryptoAPIBasics.stringToArrayBuffer(data)
        );
        
        return signature;
    }
    
    // Создание сессионного ключа
    async createSessionKey() {
        const sessionKey = await cryptoManager.generateKey(
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
        
        // Экспортируем ключ
        const exportedKey = await cryptoManager.exportKey('jwk', sessionKey);
        
        // Шифруем ключ паролем пользователя (упрощенно)
        const salt = cryptoManager.createSalt();
        const encryptedKey = await symmetricEncryption.encryptAESGCM(
            JSON.stringify(exportedKey),
            await this.deriveKeyFromPassword('user-session-key', salt)
        );
        
        return {
            encryptedKey: Array.from(new Uint8Array(encryptedKey)),
            salt: Array.from(salt),
            format: 'AES-GCM-encrypted-JWK'
        };
    }
    
    async deriveKeyFromPassword(password, salt) {
        const keyMaterial = await this.hashOperations.pbkdf2(password, salt, 10000);
        return await cryptoManager.importKey(
            'raw',
            keyMaterial,
            { name: 'AES-GCM', length: 256 },
            true,
            ['encrypt', 'decrypt']
        );
    }
}

// Использование системы аутентификации
const authSystem = new AuthenticationSystem();

async function exampleAuthenticationSystem() {
    const secret = 'very-secret-jwt-key';
    const payload = {
        userId: 123,
        username: 'john_doe',
        role: 'user'
    };
    
    // Создание токена
    const token = await authSystem.createToken(payload, secret);
    console.log('Токен создан:', token.substring(0, 50) + '...');
    
    // Проверка токена
    try {
        const verifiedPayload = await authSystem.verifyToken(token, secret);
        console.log('Токен действителен, данные:', verifiedPayload);
    } catch (error) {
        console.error('Ошибка проверки токена:', error.message);
    }
    
    // Проверка с неправильным секретом
    try {
        await authSystem.verifyToken(token, 'wrong-secret');
    } catch (error) {
        console.log('Проверка с неправильным секретом отклонена:', error.message);
    }
}
```

## Лучшие практики

### 1. Безопасная генерация ключей

```javascript
// Лучшие практики для генерации ключей
class KeyGenerationBestPractices {
    // Использование надежных параметров
    static getSecureAlgorithms() {
        return {
            rsa: {
                name: 'RSA-OAEP',
                modulusLength: 2048, // или 3072 для высокой безопасности
                publicExponent: new Uint8Array([0x01, 0x00, 0x01]),
                hash: { name: 'SHA-256' }
            },
            aes: {
                name: 'AES-GCM',
                length: 256
            },
            hmac: {
                name: 'HMAC',
                hash: { name: 'SHA-256' },
                length: 256
            },
            ecdsa: {
                name: 'ECDSA',
                namedCurve: 'P-256' // или P-384 для высокой безопасности
            }
        };
    }
    
    // Проверка надежности ключа
    static validateKeyStrength(key, algorithm) {
        // Проверка длины ключа
        if (algorithm.name === 'AES-GCM' && algorithm.length < 256) {
            console.warn('Ключ AES слишком короткий для безопасного использования');
        }
        
        // Проверка надежности RSA
        if (algorithm.name === 'RSA-OAEP' && algorithm.modulusLength < 2048) {
            console.warn('Модуль RSA слишком мал для безопасного использования');
        }
        
        return true;
    }
}
```

### 2. Обработка ошибок

```javascript
// Безопасная обработка ошибок в криптографических операциях
class CryptoErrorHandling {
    static async safeExecute(operation, ...args) {
        try {
            return await operation(...args);
        } catch (error) {
            // Логирование ошибки без раскрытия чувствительной информации
            console.error('Криптографическая ошибка:', error.constructor.name);
            
            // Возврат обобщенной ошибки
            throw new Error('Криптографическая операция не удалась');
        }
    }
    
    // Обертка для операций с ключами
    static async withKeySafety(keyOperation, ...args) {
        try {
            const result = await keyOperation(...args);
            
            // Уничтожение чувствительных данных если возможно
            if (result.key) {
                // В реальности ключи уничтожаются автоматически
                // при выходе из области видимости, если extractable = false
            }
            
            return result;
        } catch (error) {
            // Не раскрываем информацию об ошибке
            throw new Error('Ошибка операции с ключом');
        }
    }
}
```

## Безопасность

При работе с Web Crypto API важно учитывать безопасность:

```javascript
// Безопасность Web Crypto API
class CryptoSecurity {
    // Проверка надежности генерации случайных чисел
    static async validateRandomness() {
        // Тест на статистическую случайность (упрощенно)
        const samples = new Uint8Array(1000);
        CryptoAPIBasics.getRandomValues(samples);
        
        // Проверка равномерности распределения
        const counts = new Array(256).fill(0);
        for (const byte of samples) {
            counts[byte]++;
        }
        
        // Проверка, что распределение примерно равномерное
        const expected = samples.length / 256;
        const tolerance = 0.5; // 50% отклонение
        
        for (const count of counts) {
            if (Math.abs(count - expected) / expected > tolerance) {
                console.warn('Обнаружено неравномерное распределение случайных чисел');
                return false;
            }
        }
        
        return true;
    }
    
    // Защита от side-channel атак
    static secureCompare(a, b) {
        // Время выполнения не должно зависеть от содержимого
        if (a.length !== b.length) {
            // Добавляем фиктивные операции, чтобы время было постоянным
            let dummy = 0;
            for (let i = 0; i < Math.max(a.length, b.length); i++) {
                dummy |= (i < a.length ? a[i] : 0) ^ (i < b.length ? b[i] : 0);
            }
            return false;
        }
        
        let result = 0;
        for (let i = 0; i < a.length; i++) {
            result |= a[i] ^ b[i];
        }
        
        return result === 0;
    }
    
    // Проверка соответствия стандартам
    static validateCryptoImplementation() {
        const checks = {
            supportsSecureRandom: 'crypto' in window && 'getRandomValues' in window.crypto,
            supportsRequiredAlgorithms: [
                'SHA-256', 'SHA-512', 'AES-GCM', 'RSA-OAEP', 'HMAC'
            ].every(alg => {
                try {
                    // Проверка поддержки алгоритма
                    return true; // В реальности нужно проверить конкретные алгоритмы
                } catch {
                    return false;
                }
            }),
            runsOnSecureContext: window.isSecureContext
        };
        
        return checks;
    }
}

// Использование безопасных практик
async function secureCryptoExample() {
    // Проверка безопасности окружения
    const securityChecks = CryptoSecurity.validateCryptoImplementation();
    console.log('Проверки безопасности:', securityChecks);
    
    // Проверка надежности генерации случайных чисел
    const isRandomSecure = await CryptoSecurity.validateRandomness();
    console.log('Генерация случайных чисел безопасна:', isRandomSecure);
    
    // Демонстрация безопасного сравнения
    const arr1 = new Uint8Array([1, 2, 3, 4]);
    const arr2 = new Uint8Array([1, 2, 3, 4]);
    const arr3 = new Uint8Array([1, 2, 3, 5]);
    
    console.log('Безопасное сравнение (одинаковые):', CryptoSecurity.secureCompare(arr1, arr2));
    console.log('Безопасное сравнение (разные):', CryptoSecurity.secureCompare(arr1, arr3));
}
```

## Теги

#javascript #webcrypto #cryptography #encryption #decryption #hashing #signatures #security #rsa #aes #hmac
---
aliases: [IP Patterns, Network Addresses, MAC Patterns]
tags: [regexp, network, ip, mac, patterns]
---

# Паттерны для работы с IP-адресами и MAC-адресами

## Обзор

IP-адреса и MAC-адреса являются фундаментальными элементами сетевых приложений. В этой статье рассмотрим регулярные выражения для валидации, извлечения и обработки этих сетевых адресов.

## Валидация IPv4-адресов

```javascript
// Простой паттерн для IPv4 (без строгой проверки диапазона)
const basicIpv4Pattern = /\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b/;

console.log(basicIpv4Pattern.test('192.168.1.1')); // true
console.log(basicIpv4Pattern.test('999.999.999.999')); // true (некорректный адрес)

// Строгий паттерн для IPv4 (проверяет диапазон 0-255)
const strictIpv4Pattern = /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;

function isValidIpv4(ip) {
  if (!strictIpv4Pattern.test(ip)) {
    return false;
  }
  
  // Дополнительная проверка: числа не должны начинаться с 0, если это не сам 0
  const parts = ip.split('.');
  for (const part of parts) {
    if (part.length > 1 && part[0] === '0') {
      return false; // Содержит ведущий ноль
    }
  }
  
  return true;
}

console.log(isValidIpv4('192.168.1.1')); // true
console.log(isValidIpv4('255.255.255.255')); // true
console.log(isValidIpv4('0.0.0.0')); // true
console.log(isValidIpv4('192.168.01.1')); // false (ведущий ноль)
console.log(isValidIpv4('999.999.999.999')); // false
```

## Извлечение IPv4-адресов из текста

```javascript
// Паттерн для извлечения IPv4-адресов из текста
const ipv4ExtractionPattern = /(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)/g;

const textWithIps = 'Сервера: 192.168.1.10, 10.0.0.1, 172.16.254.1, и неверный 999.999.999.999';
const extractedIps = textWithIps.match(ipv4ExtractionPattern);
console.log(extractedIps); // ['192.168.1.10', '10.0.0.1', '172.16.254.1']

// Функция для валидации извлеченных IP-адресов
function extractAndValidateIpv4(text) {
  const matches = text.match(ipv4ExtractionPattern) || [];
  return matches.filter(ip => isValidIpv4(ip));
}

console.log(extractAndValidateIpv4(textWithIps)); // ['192.168.1.10', '10.0.0.1', '172.16.254.1']
```

## Валидация IPv6-адресов

```javascript
// Паттерн для базовой проверки IPv6 (сокращенные формы)
const ipv6Pattern = /^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$|^::1$|^::$|^([0-9a-fA-F]{1,4}:){1,7}:$|^:([0-9a-fA-F]{1,4}:){1,7}$|^([0-9a-fA-F]{1,4}:){1,6}:[0-9a-fA-F]{1,4}$|^([0-9a-fA-F]{1,4}:){1,5}(:[0-9a-fA-F]{1,4}){1,2}$|^([0-9a-fA-F]{1,4}:){1,4}(:[0-9a-fA-F]{1,4}){1,3}$|^([0-9a-fA-F]{1,4}:){1,3}(:[0-9a-fA-F]{1,4}){1,4}$|^([0-9a-fA-F]{1,4}:){1,2}(:[0-9a-fA-F]{1,4}){1,5}$|^[0-9a-fA-F]{1,4}:((:[0-9a-fA-F]{1,4}){1,6})$|^:((:[0-9a-fA-F]{1,4}){1,7}|:)$|^((:[0-9a-fA-F]{1,4}){1,7}|:)$|^([0-9a-fA-F]{1,4}:){1,4}:(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;

function isValidIpv6(ip) {
  return ipv6Pattern.test(ip);
}

console.log(isValidIpv6('2001:0db8:85a3:0000:0000:8a2e:0370:7334')); // true
console.log(isValidIpv6('2001:db8:85a3::8a2e:370:7334')); // true (сокращенная форма)
console.log(isValidIpv6('::1')); // true (localhost)
console.log(isValidIpv6('invalid')); // false
```

## Комбинированный класс для работы с IP-адресами

```javascript
// Класс для работы с IP-адресами
class IpAddressValidator {
  static ipv4Pattern = /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
  
  static ipv6Pattern = /^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$|^::1$|^::$|^([0-9a-fA-F]{1,4}:){1,7}:$|^:([0-9a-fA-F]{1,4}:){1,7}$|^([0-9a-fA-F]{1,4}:){1,6}:[0-9a-fA-F]{1,4}$|^([0-9a-fA-F]{1,4}:){1,5}(:[0-9a-fA-F]{1,4}){1,2}$|^([0-9a-fA-F]{1,4}:){1,4}(:[0-9a-fA-F]{1,4}){1,3}$|^([0-9a-fA-F]{1,4}:){1,3}(:[0-9a-fA-F]{1,4}){1,4}$|^([0-9a-fA-F]{1,4}:){1,2}(:[0-9a-fA-F]{1,4}){1,5}$|^[0-9a-fA-F]{1,4}:((:[0-9a-fA-F]{1,4}){1,6})$|^:((:[0-9a-fA-F]{1,4}){1,7}|:)$|^((:[0-9a-fA-F]{1,4}){1,7}|:)$|^([0-9a-fA-F]{1,4}:){1,4}:(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
  
  static validate(ip) {
    if (this.isIpv4(ip)) {
      return { type: 'IPv4', valid: true };
    } else if (this.isIpv6(ip)) {
      return { type: 'IPv6', valid: true };
    }
    return { type: null, valid: false };
  }
  
  static isIpv4(ip) {
    if (!this.ipv4Pattern.test(ip)) {
      return false;
    }
    
    // Проверка на ведущие нули
    const parts = ip.split('.');
    for (const part of parts) {
      if (part.length > 1 && part[0] === '0') {
        return false;
      }
    }
    
    return true;
  }
  
  static isIpv6(ip) {
    return this.ipv6Pattern.test(ip);
  }
  
  static extractIps(text) {
    // Паттерн для извлечения потенциальных IP-адресов
    const potentialIps = text.match(/\b[\da-fA-F:.]{3,39}\b/g) || [];
    
    const result = {
      ipv4: [],
      ipv6: [],
      all: []
    };
    
    for (const ip of potentialIps) {
      if (this.isIpv4(ip)) {
        result.ipv4.push(ip);
        result.all.push({ address: ip, type: 'IPv4' });
      } else if (this.isIpv6(ip)) {
        result.ipv6.push(ip);
        result.all.push({ address: ip, type: 'IPv6' });
      }
    }
    
    return result;
  }
  
  // Проверка, является ли IP-адрес частным
  static isPrivate(ip) {
    if (!this.isIpv4(ip)) return false;
    
    const octets = ip.split('.').map(Number);
    const [first, second] = octets;
    
    // 10.x.x.x
    if (first === 10) return true;
    // 172.16.x.x - 172.31.x.x
    if (first === 172 && second >= 16 && second <= 31) return true;
    // 192.168.x.x
    if (first === 192 && second === 168) return true;
    // 127.x.x.x (localhost)
    if (first === 127) return true;
    
    return false;
  }
}

// Пример использования
console.log(IpAddressValidator.validate('192.168.1.1')); // { type: 'IPv4', valid: true }
console.log(IpAddressValidator.validate('2001:db8::1')); // { type: 'IPv6', valid: true }
console.log(IpAddressValidator.isPrivate('192.168.1.1')); // true
console.log(IpAddressValidator.isPrivate('8.8.8.8')); // false

const networkText = 'Сервера: 192.168.1.10 (частный), 2001:db8::1, 8.8.8.8 (публичный), и localhost 127.0.0.1';
console.log(IpAddressValidator.extractIps(networkText));
```

## Работа с MAC-адресами

```javascript
// Паттерны для различных форматов MAC-адресов
const macPatterns = {
  standard: /^([0-9A-Fa-f]{2}[:]){5}([0-9A-Fa-f]{2})$/,       // AA:BB:CC:DD:EE:FF
  cisco: /^([0-9A-Fa-f]{4}\.){2}([0-9A-Fa-f]{4})$/,          // AABB.CCDD.EEFF
  bare: /^[0-9A-Fa-f]{12}$/,                                  // AABBCCDDEEFF
  hyphen: /^([0-9A-Fa-f]{2}[-]){5}([0-9A-Fa-f]{2})$/,        // AA-BB-CC-DD-EE-FF
  space: /^([0-9A-Fa-f]{2}[ ]){5}([0-9A-Fa-f]{2})$/          // AA BB CC DD EE FF
};

function validateMacAddress(mac, format = 'standard') {
  const pattern = macPatterns[format];
  if (!pattern) {
    throw new Error(`Неизвестный формат: ${format}`);
  }
  
  return pattern.test(mac);
}

console.log(validateMacAddress('AA:BB:CC:DD:EE:FF')); // true
console.log(validateMacAddress('AABB.CCDD.EEFF', 'cisco')); // true
console.log(validateMacAddress('AABBCCDDEEFF', 'bare')); // true
console.log(validateMacAddress('AA-BB-CC-DD-EE-FF', 'hyphen')); // true

// Функция для нормализации MAC-адреса
function normalizeMacAddress(mac) {
  // Удаляем все разделители и приводим к верхнему регистру
  const normalized = mac.replace(/[:\-.\s]/g, '').toUpperCase();
  
  // Проверяем, является ли результат валидным MAC-адресом
  if (normalized.length !== 12) {
    throw new Error('Неверная длина MAC-адреса');
  }
  
  // Проверяем, содержит ли только шестнадцатеричные символы
  if (!/^[0-9A-F]{12}$/.test(normalized)) {
    throw new Error('MAC-адрес содержит недопустимые символы');
  }
  
  return normalized;
}

console.log(normalizeMacAddress('aa:bb:cc:dd:ee:ff')); // AABBCCDDEEFF
console.log(normalizeMacAddress('AA-BB-CC-DD-EE-FF')); // AABBCCDDEEFF
```

## Извлечение MAC-адресов из текста

```javascript
// Функция для извлечения MAC-адресов в различных форматах
function extractMacAddresses(text) {
  // Комбинированный паттерн для всех форматов MAC-адресов
  const macPattern = /(?:[0-9A-Fa-f]{2}[:-]){5}[0-9A-Fa-f]{2}|[0-9A-Fa-f]{4}\.[0-9A-Fa-f]{4}\.[0-9A-Fa-f]{4}|[0-9A-Fa-f]{12}|[0-9A-Fa-f]{2}\s[0-9A-Fa-f]{2}\s[0-9A-Fa-f]{2}\s[0-9A-Fa-f]{2}\s[0-9A-Fa-f]{2}\s[0-9A-Fa-f]{2}/g;
  
  const matches = text.match(macPattern) || [];
  
  // Валидируем каждый найденный MAC-адрес
  return matches.filter(mac => {
    try {
      normalizeMacAddress(mac);
      return true;
    } catch (e) {
      return false;
    }
  });
}

const networkLog = 'Устройства: AA:BB:CC:DD:EE:FF, AABB.CCDD.EEFF, и неверный ZZ:ZZ:ZZ:ZZ:ZZ:ZZ';
console.log(extractMacAddresses(networkLog)); // ['AA:BB:CC:DD:EE:FF', 'AABB.CCDD.EEFF']
```

## Класс для комплексной работы с сетевыми адресами

```javascript
// Класс для комплексной работы с IP и MAC-адресами
class NetworkAddressProcessor {
  constructor() {
    this.ipValidator = IpAddressValidator;
  }
  
  // Проверка, является ли строка сетевым адресом
  identifyAddress(address) {
    if (IpAddressValidator.isIpv4(address)) {
      return { type: 'IPv4', address, private: IpAddressValidator.isPrivate(address) };
    } else if (IpAddressValidator.isIpv6(address)) {
      return { type: 'IPv6', address };
    } else if (this.isValidMacAddress(address)) {
      return { type: 'MAC', address: normalizeMacAddress(address) };
    }
    
    return { type: 'unknown', address };
  }
  
  isValidMacAddress(mac) {
    try {
      normalizeMacAddress(mac);
      return true;
    } catch (e) {
      return false;
    }
  }
  
  // Извлечение всех сетевых адресов из текста
  extractAddresses(text) {
    const result = {
      ipv4: [],
      ipv6: [],
      mac: [],
      all: []
    };
    
    // Извлекаем IP-адреса
    const ipResults = IpAddressValidator.extractIps(text);
    result.ipv4 = ipResults.ipv4;
    result.ipv6 = ipResults.ipv6;
    
    // Извлекаем MAC-адреса
    result.mac = extractMacAddresses(text);
    
    // Объединяем все адреса
    for (const ip of ipResults.all) {
      result.all.push(ip);
    }
    
    for (const mac of result.mac) {
      result.all.push({ address: mac, type: 'MAC' });
    }
    
    return result;
  }
  
  // Фильтрация адресов по типу
  filterAddresses(addresses, type) {
    return addresses.filter(addr => addr.type === type);
  }
  
  // Анонимизация IP-адресов в тексте
  anonymizeIpAddresses(text) {
    return text.replace(/\b(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)/g, 
                       ip => {
                         const parts = ip.split('.');
                         return `${parts[0]}.${parts[1]}.${parts[2]}.*`;
                       });
  }
  
  // Проверка принадлежности IP к диапазону
  isIpInRange(ip, start, end) {
    if (!IpAddressValidator.isIpv4(ip) || !IpAddressValidator.isIpv4(start) || !IpAddressValidator.isIpv4(end)) {
      return false;
    }
    
    const ipNum = this.ipToNumber(ip);
    const startNum = this.ipToNumber(start);
    const endNum = this.ipToNumber(end);
    
    return ipNum >= startNum && ipNum <= endNum;
  }
  
  // Преобразование IP в число
  ipToNumber(ip) {
    return ip.split('.').reduce((acc, octet) => (acc << 8) + parseInt(octet, 10), 0);
  }
}

// Пример использования
const processor = new NetworkAddressProcessor();

// Идентификация адресов
console.log(processor.identifyAddress('192.168.1.1')); // { type: 'IPv4', address: '192.168.1.1', private: true }
console.log(processor.identifyAddress('AA:BB:CC:DD:EE:FF')); // { type: 'MAC', address: 'AABBCCDDEEFF' }

// Извлечение из текста
const networkText = 'Сервер: 192.168.1.10, MAC: AA:BB:CC:DD:EE:FF, IPv6: 2001:db8::1';
console.log(processor.extractAddresses(networkText));

// Анонимизация
const logWithIps = 'Пользователь с 192.168.1.100 подключился к 10.0.0.1';
console.log(processor.anonymizeIpAddresses(logWithIps)); // Пользователь с 192.168.1.* подключился к 10.0.0.*

// Проверка диапазона
console.log(processor.isIpInRange('192.168.1.50', '192.168.1.1', '192.168.1.100')); // true
console.log(processor.isIpInRange('10.0.0.1', '192.168.1.1', '192.168.1.100')); // false
```

## Валидация подсетей (CIDR)

```javascript
// Функция для валидации CIDR нотации
function isValidCidr(cidr) {
  const parts = cidr.split('/');
  if (parts.length !== 2) {
    return false;
  }
  
  const ip = parts[0];
  const prefix = parseInt(parts[1], 10);
  
  if (!IpAddressValidator.isIpv4(ip)) {
    return false;
  }
  
  return prefix >= 0 && prefix <= 32;
}

console.log(isValidCidr('192.168.1.0/24')); // true
console.log(isValidCidr('10.0.0.0/8')); // true
console.log(isValidCidr('192.168.1.0/33')); // false
console.log(isValidCidr('invalid')); // false

// Извлечение CIDR из текста
function extractCidrBlocks(text) {
  const cidrPattern = /(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\/(?:[0-9]|[1-2][0-9]|3[0-2])/g;
  return text.match(cidrPattern) || [];
}

const configText = 'Сеть: 192.168.1.0/24, Шлюз: 192.168.1.1, Диапазон: 10.0.0.0/8';
console.log(extractCidrBlocks(configText)); // ['192.168.1.0/24', '10.0.0.0/8']
```

## Заключение

Эффективная работа с IP- и MAC-адресами важна для сетевых приложений, систем мониторинга и безопасности. Правильная валидация и нормализация этих адресов помогает избежать ошибок и уязвимостей в приложениях.

## См. также

- [[Network Programming Patterns]]
- [[Security Validation Techniques]]
- [[System Administration Tools]]
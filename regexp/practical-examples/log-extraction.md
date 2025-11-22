---
aliases: [Log Parsing, Log Extraction, Log Analysis]
tags: [regexp, logs, parsing, analysis]
---

# Извлечение информации из логов

## Обзор

Регулярные выражения особенно полезны для анализа логов. В этой статье рассмотрим различные паттерны для извлечения информации из логов веб-серверов, приложений и системных логов.

## Анализ логов Apache/Nginx

```javascript
// Паттерн для разбора логов Apache в формате Common Log Format
// 127.0.0.1 - frank [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326
const apacheLogPattern = /^(\S+) \S+ (\S+) \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+|-)/;

function parseApacheLog(logLine) {
  const match = logLine.match(apacheLogPattern);
  if (!match) return null;
  
  return {
    ip: match[1],
    user: match[2],
    timestamp: match[3],
    method: match[4],
    url: match[5],
    protocol: match[6],
    status: match[7],
    size: match[8]
  };
}

const logLine = '127.0.0.1 - - [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326';
const parsed = parseApacheLog(logLine);
console.log(parsed);
// {
//   ip: '127.0.0.1',
//   user: '-',
//   timestamp: '10/Oct/2000:13:55:36 -0700',
//   method: 'GET',
//   url: '/apache_pb.gif',
//   protocol: 'HTTP/1.0',
//   status: '200',
//   size: '2326'
// }
```

## Анализ логов в формате Combined Log Format

```javascript
// Паттерн для Combined Log Format
// Включает Referer и User-Agent
const combinedLogPattern = /^(\S+) \S+ (\S+) \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+|-) "([^"]*)" "([^"]*)"$/;

function parseCombinedLog(logLine) {
  const match = logLine.match(combinedLogPattern);
  if (!match) return null;
  
  return {
    ip: match[1],
    user: match[2],
    timestamp: match[3],
    method: match[4],
    url: match[5],
    protocol: match[6],
    status: match[7],
    size: match[8],
    referer: match[9],
    userAgent: match[10]
  };
}

const combinedLogLine = '127.0.0.1 - - [10/Oct/2000:13:55:36 -0700] "GET /apache_pb.gif HTTP/1.0" 200 2326 "http://www.example.com/start.html" "Mozilla/4.08 [en] (Win98; I ;Nav)"';
const parsedCombined = parseCombinedLog(combinedLogLine);
console.log(parsedCombined);
```

## Извлечение ошибок из логов

```javascript
// Паттерн для извлечения ошибок из логов приложений
const errorLogPattern = /(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) \[(\w+)\] (.+?): (\d+) - (.+)/;

function extractErrors(logLines) {
  const errors = [];
  
  for (const line of logLines) {
    const match = line.match(errorLogPattern);
    if (match && match[2] === 'ERROR') { // Только ошибки
      errors.push({
        timestamp: match[1],
        level: match[2],
        source: match[3],
        code: match[4],
        message: match[5]
      });
    }
  }
  
  return errors;
}

const logLines = [
  '2023-10-15 10:30:25 [INFO] App: 200 - Request processed',
  '2023-10-15 10:31:10 [ERROR] Database: 500 - Connection failed',
  '2023-10-15 10:32:05 [WARN] Auth: 401 - Unauthorized access attempt',
  '2023-10-15 10:33:15 [ERROR] API: 404 - Endpoint not found'
];

const errors = extractErrors(logLines);
console.log(errors);
// [
//   { timestamp: '2023-10-15 10:31:10', level: 'ERROR', source: 'Database', code: '500', message: 'Connection failed' },
//   { timestamp: '2023-10-15 10:33:15', level: 'ERROR', source: 'API', code: '404', message: 'Endpoint not found' }
// ]
```

## Анализ JSON-логов

```javascript
// Паттерн для извлечения JSON-объектов из логов
const jsonLogPattern = /\{.*\}/g;

function parseJsonLogs(logLines) {
  const parsedLogs = [];
  
  for (const line of logLines) {
    const jsonMatches = line.match(jsonLogPattern);
    if (jsonMatches) {
      try {
        const jsonLog = JSON.parse(jsonMatches[0]);
        parsedLogs.push(jsonLog);
      } catch (e) {
        console.warn('Невозможно разобрать JSON:', jsonMatches[0]);
      }
    }
  }
  
  return parsedLogs;
}

const jsonLogLines = [
  '2023-10-15 10:30:25 [INFO] { "user": "john", "action": "login", "status": "success" }',
  '2023-10-15 10:31:10 [ERROR] { "user": "jane", "action": "payment", "status": "failed", "error": "insufficient funds" }'
];

const parsedJsonLogs = parseJsonLogs(jsonLogLines);
console.log(parsedJsonLogs);
```

## Извлечение IP-адресов из логов

```javascript
// Паттерн для извлечения IP-адресов из логов
const ipPattern = /\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b/g;

function extractIPs(logLine) {
  const ips = logLine.match(ipPattern);
  return ips ? [...new Set(ips)] : []; // Уникальные IP-адреса
}

const logWithIPs = 'Connection from 192.168.1.100 to 10.0.0.1, blocked 192.168.1.100';
console.log(extractIPs(logWithIPs)); // ['192.168.1.100', '10.0.0.1']
```

## Анализ времени выполнения запросов

```javascript
// Паттерн для извлечения времени выполнения запросов
const responseTimePattern = /response_time=(\d+(?:\.\d+)?)ms/;

function extractResponseTimes(logLines) {
  const times = [];
  
  for (const line of logLines) {
    const match = line.match(responseTimePattern);
    if (match) {
      times.push(parseFloat(match[1]));
    }
  }
  
  return times;
}

const logWithResponseTimes = [
  'GET /api/users response_time=125.5ms status=200',
  'POST /api/login response_time=89.2ms status=201',
  'GET /api/products response_time=210.7ms status=200'
];

const responseTimes = extractResponseTimes(logWithResponseTimes);
console.log('Среднее время ответа:', responseTimes.reduce((a, b) => a + b, 0) / responseTimes.length);
```

## Извлечение URL-адресов из логов

```javascript
// Паттерн для извлечения URL из логов
const urlPattern = /https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/g;

function extractUrls(logLine) {
  const urls = logLine.match(urlPattern);
  return urls || [];
}

const logWithUrls = 'User visited https://example.com/page and clicked on https://api.example.com/v1/data';
console.log(extractUrls(logWithUrls)); // ['https://example.com/page', 'https://api.example.com/v1/data']
```

## Поиск подозрительной активности

```javascript
// Паттерн для поиска подозрительной активности (например, SQL-инъекции)
const suspiciousPatterns = [
  /(\b(UNION|SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|DECLARE|CAST|CONVERT)\b)/gi,
  /('|--|\/\*|\*\/|;)/g,
  /(%27|%3D|%3C|%3E|%22)/gi  // URL-кодированные символы
];

function detectSuspiciousActivity(logLine) {
  for (const pattern of suspiciousPatterns) {
    if (pattern.test(logLine)) {
      return true;
    }
  }
  return false;
}

const suspiciousLog = 'GET /api/search?q=1%27%20UNION%20SELECT%20*%20FROM%20users';
console.log('Подозрительная активность:', detectSuspiciousActivity(suspiciousLog)); // true
```

## Агрегация статистики из логов

```javascript
// Функция для агрегации статистики из логов
function aggregateLogStats(logLines) {
  const stats = {
    totalRequests: 0,
    statusCodes: {},
    ips: {},
    urls: {},
    errors: 0
  };
  
  const logPattern = /^(\S+) \S+ (\S+) \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+|-)/;
  
  for (const line of logLines) {
    const match = line.match(logPattern);
    if (match) {
      stats.totalRequests++;
      
      const statusCode = match[7];
      stats.statusCodes[statusCode] = (stats.statusCodes[statusCode] || 0) + 1;
      
      const ip = match[1];
      stats.ips[ip] = (stats.ips[ip] || 0) + 1;
      
      const url = match[5];
      stats.urls[url] = (stats.urls[url] || 0) + 1;
      
      if (statusCode.startsWith('4') || statusCode.startsWith('5')) {
        stats.errors++;
      }
    }
  }
  
  return stats;
}

// Пример использования
const sampleLogs = [
  '127.0.0.1 - - [10/Oct/2000:13:55:36 -0700] "GET /index.html HTTP/1.0" 200 2326',
  '127.0.0.1 - - [10/Oct/2000:13:55:37 -0700] "GET /styles.css HTTP/1.0" 200 1234',
  '192.168.1.100 - - [10/Oct/2000:13:55:38 -0700] "GET /api/data HTTP/1.0" 500 567',
  '192.168.1.101 - - [10/Oct/2000:13:55:39 -0700] "POST /login HTTP/1.0" 401 890'
];

const logStats = aggregateLogStats(sampleLogs);
console.log(logStats);
```

## Заключение

Регулярные выражения являются мощным инструментом для анализа логов. Они позволяют эффективно извлекать структурированную информацию из неструктурированных данных логов, что особенно полезно для мониторинга, аудита и анализа безопасности.

## См. также

- [[Log Analysis Tools]]
- [[RegExp Performance Optimization]]
- [[Security Log Monitoring]]
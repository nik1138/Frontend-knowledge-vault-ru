---
aliases: [Отчеты о нарушениях CSP, CSP Violation Reports, Мониторинг CSP]
tags: [security, csp, content-security-policy, web-security, monitoring]
---

# Отчеты-о-нарушениях-CSP

## Обзор

Отчеты о нарушениях CSP (Content Security Policy Violation Reports) - это механизм, позволяющий веб-приложениям получать информацию о попытках нарушения политики безопасности контента. Эти отчеты помогают разработчикам выявлять проблемы с CSP, улучшать политику и обеспечивать баланс между безопасностью и функциональностью.

## Структура отчета о нарушении CSP

Когда происходит нарушение CSP, браузер отправляет JSON-объект с информацией о нарушении на указанный URL. Пример структуры отчета:

```json
{
  "csp-report": {
    "document-uri": "https://example.com/page.html",
    "referrer": "https://example.com/",
    "blocked-uri": "https://evil.com/script.js",
    "violated-directive": "script-src 'self'",
    "effective-directive": "script-src",
    "original-policy": "default-src 'self'; script-src 'self'; report-uri /csp-report",
    "disposition": "enforce",
    "status-code": 200,
    "script-sample": "console.log('XSS attempt')"
  }
}
```

### Поля отчета

- `document-uri`: URL документа, в котором произошло нарушение
- `referrer`: Реферер документа
- `blocked-uri`: URI ресурса, который был заблокирован
- `violated-directive`: Нарушенная директива CSP
- `effective-directive`: Эффективная директива, которая вызвала блокировку
- `original-policy`: Оригинальная политика CSP
- `disposition`: Режим политики (enforce или report)
- `status-code`: HTTP-статус код документа
- `script-sample`: Образец вредоносного скрипта (если доступен)

## Настройка отчетности CSP

### 1. Использование report-uri (устаревший метод)
```
Content-Security-Policy: default-src 'self'; report-uri /csp-report
```

### 2. Использование report-to (современный метод)
Сначала определите endpoint в заголовке `Content-Security-Policy-Report-Only`:
```json
{
  "csp-endpoint": {
    "url": "https://example.com/csp-report",
    "include-subdomains": true,
    "success-frequency": 1,
    "failure-frequency": 1
  }
}
```

Затем используйте в CSP:
```
Content-Security-Policy: default-src 'self'; report-to csp-endpoint
```

## Реализация обработчика отчетов

### Node.js (Express) пример
```javascript
app.use(express.json({ type: 'application/csp-report' }));
app.use(express.json({ type: 'application/json' }));

app.post('/csp-report', (req, res) => {
  const cspReport = req.body['csp-report'] || req.body;
  
  console.log('CSP Violation Report:');
  console.log('Document URI:', cspReport['document-uri']);
  console.log('Blocked URI:', cspReport['blocked-uri']);
  console.log('Violated Directive:', cspReport['violated-directive']);
  console.log('Effective Directive:', cspReport['effective-directive']);
  
  // Логирование в систему мониторинга
  logCSPViolation(cspReport);
  
  // Ответ сервера
  res.status(204).end();
});

function logCSPViolation(report) {
  // Сохранение в базу данных или отправка в систему мониторинга
  // Логирование для дальнейшего анализа
}
```

### Python (Flask) пример
```python
from flask import Flask, request, jsonify
import json

app = Flask(__name__)

@app.route('/csp-report', methods=['POST'])
def csp_report():
    # Получение JSON-данных из тела запроса
    csp_report = request.get_json()
    
    if csp_report:
        # Извлечение отчета о нарушении
        report = csp_report.get('csp-report', {})
        
        print(f"CSP Violation: {report}")
        
        # Обработка отчета (логирование, анализ и т.д.)
        process_csp_violation(report)
    
    return '', 204

def process_csp_violation(report):
    # Логика обработки отчета
    pass
```

## Анализ отчетов о нарушениях

### Категории нарушений

#### 1. Inline-скрипты и стили
- `blocked-uri`: 'inline'
- Часто указывает на необходимость использования nonce или hash

#### 2. Внешние ресурсы
- `blocked-uri`: URL внешнего ресурса
- Потребуется обновление политики для разрешения легитимных ресурсов

#### 3. eval и подобные функции
- `blocked-uri`: 'eval'
- Указывает на использование небезопасных функций

### Метрики для анализа

#### 1. Частота нарушений
- Количество отчетов в единицу времени
- Помогает определить серьезность проблемы

#### 2. Типы нарушений
- Распределение по типам директив
- Помогает приоритизировать исправления

#### 3. Источники нарушений
- URL-адреса, вызывающие нарушения
- Помогает локализовать проблемные участки кода

## Использование отчетов для улучшения CSP

### 1. Режим отчетности (Report-Only)
Сначала используйте CSP в режиме отчетности для выявления проблем:
```
Content-Security-Policy-Report-Only: default-src 'self'; script-src 'self'; report-uri /csp-report
```

### 2. Анализ данных
- Собирайте данные в течение нескольких дней/недель
- Анализируйте типичные нарушения
- Идентифицируйте легитимные источники, блокируемые CSP

### 3. Итеративное улучшение
- На основе отчетов постепенно усиливайте политику
- Добавляйте разрешения для легитимных ресурсов
- Устраняйте причины нарушений

## Интеграция с системами мониторинга

### 1. Логирование в ELK стек
```javascript
// Пример отправки в Elasticsearch
const sendToElasticsearch = (report) => {
  fetch('http://elasticsearch:9200/csp-violations/_doc', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      timestamp: new Date().toISOString(),
      report: report,
      userAgent: navigator.userAgent
    })
  });
};
```

### 2. Интеграция с системами оповещения
- Настройка оповещений при превышении порога нарушений
- Отправка уведомлений о новых типах нарушений
- Интеграция с системами мониторинга (Prometheus, Grafana и др.)

## Практические примеры анализа

### Пример 1: Обнаружение XSS-атак
Если в отчетах часто встречается:
```json
{
  "blocked-uri": "data:text/html;base64,...",
  "violated-directive": "default-src 'self'"
}
```
Это может указывать на попытки XSS-атак через data-URI.

### Пример 2: Проблемы с внешними библиотеками
Если отчеты содержат:
```json
{
  "blocked-uri": "https://cdn.example.com/library.js",
  "violated-directive": "script-src 'self'"
}
```
Необходимо обновить политику для разрешения CDN.

## Приватность и безопасность отчетов

### 1. Защита данных
- Не отправляйте конфиденциальные данные в отчетах
- Ограничьте доступ к системе сбора отчетов
- Используйте HTTPS для передачи отчетов

### 2. Защита от атак через отчеты
- Проверяйте и валидируйте полученные отчеты
- Ограничьте объем принимаемых данных
- Используйте безопасные методы обработки отчетов

## Современные подходы к отчетности

### 1. Reporting API
Современный подход, заменяющий report-uri:
```
Reporting-Endpoints: csp-endpoint="https://example.com/csp-reports"
Content-Security-Policy: default-src 'self'; report-to csp-endpoint
```

### 2. Отчеты о групповых нарушениях
- Снижение нагрузки на сервер за счет группировки отчетов
- Возможность настройки частоты отправки отчетов

## Лучшие практики

1. **Используйте режим отчетности перед включением CSP** - это позволяет выявить проблемы без блокировки функций
2. **Регулярно анализируйте отчеты** - отчеты должны обрабатываться и использоваться для улучшения политики
3. **Настройте мониторинг аномалий** - внезапное увеличение отчетов может указывать на атаки
4. **Документируйте легитимные исключения** - фиксируйте причины разрешения определенных источников
5. **Обеспечьте безопасность системы отчетности** - система сбора отчетов сама должна быть защищена

## Связанные темы

- [[Оценка-CSP]]
- [[Реализация-CSP]]
- [[Директивы-CSP]]
- [[Формирование-CSP-политики]]

> [!tip] Совет
> Используйте отчеты о нарушениях CSP как ценный источник информации для постепенного усиления политики безопасности контента.

> [!warning] Важно
> Отчеты о нарушениях CSP могут содержать чувствительную информацию, поэтому необходимо обеспечить безопасность системы их обработки.
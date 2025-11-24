---
aliases: [Web Workers, Потоки в браузере, Многопоточность в JavaScript]
tags: [html, browser-api, web-workers, javascript]
---

# Web Workers

## Обзор

Web Workers позволяют запускать JavaScript в фоновом потоке, не блокируя основной поток выполнения. Это особенно полезно для выполнения тяжелых вычислений или обработки данных, которые могут замедлить пользовательский интерфейс.

## Типы Web Workers

- **Dedicated Workers** - привязаны к одному скрипту и не могут быть использованы другими скриптами.
- **Shared Workers** - могут использоваться несколькими скриптами в одном домене.
- **Service Workers** - специальный тип worker'ов, используемых для кэширования и управления сетевыми запросами (см. [[Service Workers]]).

## Практические рекомендации

### Создание Dedicated Worker

```javascript
// main.js
const worker = new Worker('worker.js');

worker.postMessage('Привет, Worker!');

worker.onmessage = function(e) {
  console.log('Получено от Worker:', e.data);
};
```

```javascript
// worker.js
self.onmessage = function(e) {
  console.log('Получено в Worker:', e.data);
  self.postMessage('Привет, Main!');
};
```

## Ограничения и особенности

- Web Workers не имеют доступа к DOM, `window`, `document` и другим объектам основного потока.
- Общение между основным потоком и worker'ом осуществляется через сообщения.
- Web Workers не могут использовать синхронные методы, такие как `XMLHttpRequest` (только асинхронные).

## Ссылки

- [[JavaScript]]
- [[Service Workers]]
- [[Browser API]]

## Теги

#html #browser-api #web-workers #javascript #web-development
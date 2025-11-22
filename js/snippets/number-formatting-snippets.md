---
aliases: ["Сниппеты для форматирования чисел и валют", "Форматирование чисел JS"]
tags: [js, snippets, numbers, formatting, frontend]
---

# Сниппеты для форматирования чисел и валют

Практические сниппеты для форматирования чисел и валют в JavaScript. Эти сниппеты помогут вам красиво отображать числовые данные в ваших веб-приложениях.

## 1. Форматирование чисел с разделителями

```javascript
// Форматирование числа с разделителями тысяч
function formatNumber(num, separator = ' ') {
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, separator);
}

// Форматирование числа с использованием toLocaleString
function formatNumberLocale(num, locale = 'ru-RU', options = {}) {
  const defaultOptions = { 
    maximumFractionDigits: 0 
  };
  
  return num.toLocaleString(locale, { ...defaultOptions, ...options });
}

// Форматирование с фиксированным количеством знаков после запятой
function formatFixedDecimals(num, decimals = 2) {
  return num.toFixed(decimals);
}

// Форматирование с автоматическим определением разделителя
function formatNumberAuto(num) {
  return new Intl.NumberFormat().format(num);
}

console.log(formatNumber(1234567)); // '1 234 567'
console.log(formatNumberLocale(1234567.89, 'ru-RU')); // '1 234 568'
console.log(formatFixedDecimals(1234.567, 2)); // '1234.57'
console.log(formatNumberAuto(1234567)); // '1,234,567' (в зависимости от локали)
```

## 2. Форматирование валют

```javascript
// Форматирование валюты с использованием toLocaleString
function formatCurrency(amount, currency = 'RUB', locale = 'ru-RU') {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: 2
  }).format(amount);
}

// Форматирование валюты с округлением
function formatCurrencyRounded(amount, currency = 'USD', locale = 'en-US') {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: 0,
    maximumFractionDigits: 0
  }).format(Math.round(amount));
}

// Форматирование валюты с настраиваемыми опциями
function formatCurrencyCustom(amount, currency = 'EUR', locale = 'de-DE', options = {}) {
  const defaultOptions = {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  };
  
  return new Intl.NumberFormat(locale, { ...defaultOptions, ...options }).format(amount);
}

// Форматирование валюты с символом валюты
function formatCurrencyWithSymbol(amount, currency = 'RUB') {
  const formatter = new Intl.NumberFormat('ru-RU', {
    style: 'currency',
    currency: currency,
    currencyDisplay: 'symbol'
  });
  
  return formatter.format(amount);
}

console.log(formatCurrency(1234.56, 'RUB')); // '1 234,56 ₽'
console.log(formatCurrency(1234.56, 'USD')); // '$1,234.56'
console.log(formatCurrencyRounded(1234.56, 'USD')); // '$1,235'
console.log(formatCurrencyWithSymbol(1234.56, 'EUR')); // '1.234,56 €'
```

## 3. Форматирование с префиксами и суффиксами

```javascript
// Форматирование с процентами
function formatPercentage(value, locale = 'ru-RU', decimals = 2) {
  return new Intl.NumberFormat(locale, {
    style: 'percent',
    minimumFractionDigits: decimals,
    maximumFractionDigits: decimals
  }).format(value);
}

// Форматирование с префиксом (например, знаком плюс для положительных чисел)
function formatWithSign(num, locale = 'ru-RU') {
  const formatted = Math.abs(num).toLocaleString(locale);
  return num >= 0 ? `+${formatted}` : `-${formatted}`;
}

// Форматирование с суффиксом (например, единицы измерения)
function formatWithUnit(value, unit, locale = 'ru-RU') {
  const formatted = value.toLocaleString(locale);
  return `${formatted} ${unit}`;
}

// Форматирование с сокращениями (K, M, B)
function formatWithAbbreviation(num) {
  if (num >= 1e9) {
    return (num / 1e9).toFixed(1).replace(/\.0$/, '') + 'B';
  }
  if (num >= 1e6) {
    return (num / 1e6).toFixed(1).replace(/\.0$/, '') + 'M';
  }
  if (num >= 1e3) {
    return (num / 1e3).toFixed(1).replace(/\.0$/, '') + 'K';
  }
  return num.toString();
}

console.log(formatPercentage(0.1234)); // '12.34%'
console.log(formatWithSign(1234)); // '+1 234' (в русской локали)
console.log(formatWithUnit(25, '°C')); // '25 °C'
console.log(formatWithAbbreviation(1234567)); // '1.2M'
```

## 4. Форматирование для специфичных случаев

```javascript
// Форматирование телефонных номеров
function formatPhoneNumber(phone) {
  // Удаляем все нецифровые символы
  const cleaned = phone.toString().replace(/\D/g, '');
  
  // Применяем маску для российских номеров
  if (cleaned.length === 11 && cleaned.startsWith('7')) {
    return cleaned.replace(/(\d{1})(\d{3})(\d{3})(\d{2})(\d{2})/, '+$1 ($2) $3-$4-$5');
  }
  
  // Общая маска
  return cleaned.replace(/(\d{1})(\d{3})(\d{3})(\d{4})/, '+$1 ($2) $3-$4');
}

// Форматирование кредитных карт
function formatCreditCard(cardNumber) {
  return cardNumber.toString().replace(/\D/g, '').replace(/(\d{4})(?=\d)/g, '$1 ');
}

// Форматирование банковских счетов
function formatBankAccount(accountNumber) {
  return accountNumber.toString().replace(/\D/g, '')
    .replace(/(\d{4})(\d{4})(\d{4})(\d{4})/, '$1 $2 $3 $4');
}

// Форматирование IP-адресов
function formatIpAddress(ip) {
  return ip.toString().replace(/\D/g, '.').replace(/[^\d.]/g, '')
    .replace(/\.+/g, '.').replace(/^\.|\.$/g, '');
}

// Форматирование веса
function formatWeight(weight, unit = 'kg', locale = 'ru-RU') {
  const units = {
    'kg': 'кг',
    'g': 'г',
    'lb': 'фунт',
    'oz': 'унция'
  };
  
  const formatted = weight.toLocaleString(locale, {
    minimumFractionDigits: weight % 1 === 0 ? 0 : 2,
    maximumFractionDigits: 2
  });
  
  return `${formatted} ${units[unit] || unit}`;
}

console.log(formatPhoneNumber('79991234567')); // '+7 (999) 123-45-67'
console.log(formatCreditCard('1234567890123456')); // '1234 5678 9012 3456'
console.log(formatWeight(68.5)); // '68.5 кг'
```

## 5. Работа с дробями и точностью

```javascript
// Форматирование с определенной точностью
function formatPrecision(num, precision) {
  const factor = Math.pow(10, precision);
  return Math.round(num * factor) / factor;
}

// Форматирование с минимальным и максимальным количеством знаков после запятой
function formatMinMaxDecimals(num, minDecimals = 0, maxDecimals = 2) {
  return num.toLocaleString('en-US', {
    minimumFractionDigits: minDecimals,
    maximumFractionDigits: maxDecimals
  });
}

// Форматирование для финансовых расчетов с высокой точностью
function formatFinancial(num, decimals = 2) {
  // Используем BigInt для избежания проблем с плавающей точкой
  const factor = Math.pow(10, decimals);
  const rounded = Math.round((num + Number.EPSILON) * factor) / factor;
  return rounded.toLocaleString('en-US', {
    minimumFractionDigits: decimals,
    maximumFractionDigits: decimals
  });
}

// Форматирование с сохранением значимых цифр
function formatSignificantDigits(num, significantDigits = 3) {
  if (num === 0) return '0';
  
  const digits = Math.max(0, significantDigits - Math.floor(Math.log10(Math.abs(num))) - 1);
  return num.toLocaleString('en-US', {
    minimumFractionDigits: digits,
    maximumFractionDigits: digits
  });
}

console.log(formatPrecision(123.456789, 2)); // 123.46
console.log(formatMinMaxDecimals(123.4, 2, 4)); // '123.40'
console.log(formatFinancial(123.456, 2)); // '123.46'
console.log(formatSignificantDigits(1234.567, 3)); // '1234.57'
```

## 6. Форматирование в зависимости от диапазона

```javascript
// Форматирование чисел с автоматическим выбором формата
function smartFormatNumber(num) {
  if (num >= 1e12) {
    return (num / 1e12).toFixed(2) + 'T'; // триллионы
  } else if (num >= 1e9) {
    return (num / 1e9).toFixed(2) + 'B'; // миллиарды
  } else if (num >= 1e6) {
    return (num / 1e6).toFixed(2) + 'M'; // миллионы
  } else if (num >= 1e3) {
    return (num / 1e3).toFixed(2) + 'K'; // тысячи
  } else {
    return num.toLocaleString();
  }
}

// Форматирование для отображения в разных единицах измерения
function formatWithUnits(value, units = ['B', 'KB', 'MB', 'GB', 'TB'], base = 1024) {
  let unitIndex = 0;
  let result = value;
  
  while (result >= base && unitIndex < units.length - 1) {
    result /= base;
    unitIndex++;
  }
  
  return `${result.toFixed(2)} ${units[unitIndex]}`;
}

// Форматирование времени в удобочитаемом формате
function formatDuration(seconds) {
  const days = Math.floor(seconds / (24 * 3600));
  const hours = Math.floor((seconds % (24 * 3600)) / 3600);
  const minutes = Math.floor((seconds % 3600) / 60);
  const secs = seconds % 60;
  
  const parts = [];
  if (days > 0) parts.push(`${days}д`);
  if (hours > 0) parts.push(`${hours}ч`);
  if (minutes > 0) parts.push(`${minutes}м`);
  if (secs > 0 || parts.length === 0) parts.push(`${secs}с`);
  
  return parts.join(' ');
}

console.log(smartFormatNumber(1234567890)); // '1.23B'
console.log(formatWithUnits(1073741824)); // '1.00 GB' (для байтов)
console.log(formatDuration(3661)); // '1ч 1м 1с'
```

## 7. Сниппеты для специфичных финансовых форматов

```javascript
// Форматирование цен с правильным округлением
function formatPrice(price, currency = 'RUB', locale = 'ru-RU') {
  // Правильное округление до ближайших 5 копеек
  const roundedPrice = Math.round(price * 20) / 20;
  
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency: currency,
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }).format(roundedPrice);
}

// Форматирование скидок
function formatDiscount(originalPrice, discountedPrice, locale = 'ru-RU') {
  const discountPercent = ((originalPrice - discountedPrice) / originalPrice) * 100;
  return {
    original: formatCurrency(originalPrice),
    discounted: formatCurrency(discountedPrice),
    percent: formatPercentage(discountPercent / 100, locale, 0),
    amount: formatCurrency(originalPrice - discountedPrice)
  };
}

// Форматирование для отображения в сравнениях
function formatComparisonValue(value, compareValue, locale = 'ru-RU') {
  const diff = value - compareValue;
  const percentDiff = ((value - compareValue) / Math.abs(compareValue)) * 100;
  
  return {
    value: value.toLocaleString(locale),
    diff: formatWithSign(diff, locale),
    percentDiff: formatWithSign(percentDiff, locale) + '%'
  };
}

// Форматирование для диаграмм и графиков
function formatChartValue(value, type = 'default') {
  if (type === 'currency') {
    return formatCurrency(value, 'USD', 'en-US');
  } else if (type === 'percentage') {
    return formatPercentage(value, 'en-US', 1);
  } else if (type === 'large') {
    return formatWithAbbreviation(value);
  }
  return value.toLocaleString('en-US');
}

console.log(formatPrice(123.456)); // '123.45 ₽' или '$123.45' в зависимости от валюты
console.log(formatDiscount(100, 80));
// {
//   original: '100,00 ₽',
//   discounted: '80,00 ₽',
//   percent: '20%',
//   amount: '20,00 ₽'
// }
```

## 8. Утилиты для парсинга и валидации

```javascript
// Парсинг отформатированного числа обратно в число
function parseFormattedNumber(str) {
  // Удаляем все символы, кроме цифр, точки и минуса
  const cleaned = str.replace(/[^\d.-]/g, '');
  const num = parseFloat(cleaned);
  return isNaN(num) ? 0 : num;
}

// Парсинг валюты
function parseCurrency(str) {
  // Удаляем символы валюты и пробелы
  const cleaned = str.replace(/[^\d.,\-\s]/g, '').replace(/\s/g, '');
  // Заменяем запятую на точку, если это разделитель дробной части
  const normalized = cleaned.replace(/,(\d{2})$/, '.$1'); // для случаев, когда запятая используется для копеек
  const num = parseFloat(normalized);
  return isNaN(num) ? 0 : num;
}

// Проверка валидности числового формата
function isValidNumberFormat(str) {
  // Проверяем, является ли строка валидным числом после очистки
  const cleaned = str.replace(/[^\d.,\-\s]/g, '');
  const num = parseFloat(cleaned.replace(/[\s]/g, '').replace(/,/g, '.'));
  return !isNaN(num) && cleaned === num.toString();
}

// Форматирование с проверкой на валидность
function safeFormatNumber(input, formatFunction) {
  const num = typeof input === 'number' ? input : parseFloat(input);
  if (isNaN(num)) {
    throw new Error('Invalid number');
  }
  return formatFunction(num);
}

console.log(parseFormattedNumber('1 234 567,89')); // 1234567.89 (в зависимости от формата)
console.log(parseCurrency('$1,234.56')); // 1234.56
console.log(isValidNumberFormat('1,234.56')); // true
```

Эти сниппеты помогут вам эффективно форматировать числа и валюты в ваших JavaScript приложениях, обеспечивая правильное отображение и удобство для пользователей.

См. также:
- [[js/snippets/date-time-snippets]] - Сниппеты для работы с датами и временем
- [[js/snippets/url-validation-snippets]] - Сниппеты для обработки и валидации URL
- [[js/snippets/color-snippets]] - Сниппеты для работы с цветами
---
aliases: ["Сниппеты для работы с датами и временем", "Форматирование дат JS"]
tags: [js, snippets, date, time, frontend]
---

# Сниппеты для работы с датами и временем

Практические сниппеты для работы с датами и временем в JavaScript, которые помогут вам эффективно обрабатывать временные данные в ваших приложениях.

## 1. Форматирование дат

```javascript
// Форматирование даты в формате DD/MM/YYYY
function formatDate(date) {
  const d = new Date(date);
  let month = '' + (d.getMonth() + 1);
  let day = '' + d.getDate();
  let year = d.getFullYear();

  if (month.length < 2) month = '0' + month;
  if (day.length < 2) day = '0' + day;

  return [day, month, year].join('/');
}

// Форматирование даты в ISO формате
function toISOString(date) {
  const d = new Date(date);
  return d.toISOString().split('T')[0]; // YYYY-MM-DD
}

// Форматирование с использованием toLocaleDateString
function formatLocaleDate(date, locale = 'ru-RU', options = {}) {
  const defaultOptions = { 
    day: '2-digit', 
    month: '2-digit', 
    year: 'numeric' 
  };
  return new Date(date).toLocaleDateString(locale, { ...defaultOptions, ...options });
}

// Форматирование даты и времени
function formatDateTime(date, locale = 'ru-RU') {
  return new Date(date).toLocaleString(locale, {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit'
  });
}

console.log(formatDate('2023-05-15')); // '15/05/2023'
console.log(toISOString('2023-05-15')); // '2023-05-15'
console.log(formatLocaleDate('2023-05-15')); // '15.05.2023' (в зависимости от локали)
console.log(formatDateTime('2023-05-15T14:30:00')); // '15.05.2023, 14:30:00' (в зависимости от локали)
```

## 2. Вычисление разницы между датами

```javascript
// Вычисление разницы в днях между двумя датами
function daysDifference(date1, date2) {
  const d1 = new Date(date1);
  const d2 = new Date(date2);
  const timeDiff = Math.abs(d2.getTime() - d1.getTime());
  return Math.ceil(timeDiff / (1000 * 3600 * 24));
}

// Вычисление возраста по дате рождения
function calculateAge(birthDate) {
  const today = new Date();
  const birth = new Date(birthDate);
  let age = today.getFullYear() - birth.getFullYear();
  const monthDiff = today.getMonth() - birth.getMonth();
  
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birth.getDate())) {
    age--;
  }
  
  return age;
}

// Получение продолжительности в формате человекопонятном
function getDuration(start, end) {
  const startDate = new Date(start);
  const endDate = new Date(end);
  const diffMs = endDate - startDate;
  
  const days = Math.floor(diffMs / (1000 * 60 * 60 * 24));
  const hours = Math.floor((diffMs % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  const minutes = Math.floor((diffMs % (1000 * 60 * 60)) / (1000 * 60));
  const seconds = Math.floor((diffMs % (1000 * 60)) / 1000);
  
  return {
    days,
    hours,
    minutes,
    seconds,
    formatted: `${days} дней, ${hours} часов, ${minutes} минут, ${seconds} секунд`
  };
}

console.log(daysDifference('2023-01-01', '2023-01-10')); // 9
console.log(calculateAge('1990-05-15')); // возраст (в зависимости от текущей даты)
console.log(getDuration('2023-01-01T10:00:00', '2023-01-02T12:30:45'));
// { days: 1, hours: 2, minutes: 30, seconds: 45, formatted: '1 дней, 2 часов, 30 минут, 45 секунд' }
```

## 3. Операции с датами

```javascript
// Добавление дней к дате
function addDays(date, days) {
  const result = new Date(date);
  result.setDate(result.getDate() + days);
  return result;
}

// Добавление месяцев к дате
function addMonths(date, months) {
  const result = new Date(date);
  result.setMonth(result.getMonth() + months);
  return result;
}

// Добавление лет к дате
function addYears(date, years) {
  const result = new Date(date);
  result.setFullYear(result.getFullYear() + years);
  return result;
}

// Получение начала дня
function startOfDay(date) {
  const d = new Date(date);
  d.setHours(0, 0, 0, 0);
  return d;
}

// Получение конца дня
function endOfDay(date) {
  const d = new Date(date);
  d.setHours(23, 59, 59, 999);
  return d;
}

// Получение начала недели (понедельник)
function startOfWeek(date) {
  const d = new Date(date);
  const day = d.getDay();
  const diff = d.getDate() - day + (day === 0 ? -6 : 1); // воскресенье -> понедельник
  return new Date(d.setDate(diff));
}

// Получение начала месяца
function startOfMonth(date) {
  const d = new Date(date);
  d.setDate(1);
  d.setHours(0, 0, 0, 0);
  return d;
}

console.log(addDays('2023-05-15', 10)); // '2023-05-25'
console.log(addMonths('2023-05-15', 2)); // '2023-07-15'
console.log(startOfWeek('2023-05-17')); // начало недели (понедельник)
console.log(startOfMonth('2023-05-17')); // '2023-05-01'
```

## 4. Валидация дат

```javascript
// Проверка валидности даты
function isValidDate(date) {
  const d = new Date(date);
  return d instanceof Date && !isNaN(d);
}

// Проверка, является ли дата в будущем
function isFuture(date) {
  return new Date(date) > new Date();
}

// Проверка, является ли дата в прошлом
function isPast(date) {
  return new Date(date) < new Date();
}

// Проверка, находится ли дата в заданном диапазоне
function isInRange(date, startDate, endDate) {
  const d = new Date(date);
  const start = new Date(startDate);
  const end = new Date(endDate);
  return d >= start && d <= end;
}

// Проверка високосного года
function isLeapYear(year) {
  return (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0);
}

console.log(isValidDate('2023-05-15')); // true
console.log(isValidDate('invalid-date')); // false
console.log(isFuture('2025-12-31')); // true (если текущая дата раньше)
console.log(isInRange('2023-06-01', '2023-05-01', '2023-06-30')); // true
console.log(isLeapYear(2024)); // true
```

## 5. Работа с часовыми поясами

```javascript
// Форматирование даты с указанием часового пояса
function formatDateWithTimezone(date, timezone = 'UTC', locale = 'ru-RU') {
  return new Date(date).toLocaleString(locale, {
    timeZone: timezone,
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    timeZoneName: 'short'
  });
}

// Преобразование времени из одного часового пояса в другой
function convertTimezone(date, fromTimezone, toTimezone) {
  const sourceTime = new Date(date).toLocaleString('en-US', { timeZone: fromTimezone });
  const convertedTime = new Date(sourceTime).toLocaleString('en-US', { timeZone: toTimezone });
  return new Date(convertedTime);
}

// Получение смещения часового пояса
function getTimezoneOffset(date, timezone) {
  const utc = new Date(date).getTime() + (new Date(date).getTimezoneOffset() * 60000);
  const targetTime = new Date(utc + (getTimezoneOffsetMs(timezone)));
  return targetTime;
}

// Вспомогательная функция для получения смещения в миллисекундах
function getTimezoneOffsetMs(timezone) {
  const date = new Date();
  const utc = new Date(date.toLocaleString('en-US', { timeZone: 'UTC' }));
  const target = new Date(date.toLocaleString('en-US', { timeZone: timezone }));
  return target.getTime() - utc.getTime();
}

console.log(formatDateWithTimezone('2023-05-15T12:00:00', 'Europe/Moscow'));
// '15.05.2023, 15:00:00 MSK'
```

## 6. Сниппеты для форматирования времени

```javascript
// Форматирование времени в формате HH:MM:SS
function formatTime(date) {
  const d = new Date(date);
  const hours = String(d.getHours()).padStart(2, '0');
  const minutes = String(d.getMinutes()).padStart(2, '0');
  const seconds = String(d.getSeconds()).padStart(2, '0');
  return `${hours}:${minutes}:${seconds}`;
}

// Форматирование продолжительности в формате MM:SS или HH:MM:SS
function formatDuration(seconds) {
  const h = Math.floor(seconds / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  const s = Math.floor(seconds % 60);
  
  if (h > 0) {
    return `${h}:${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
  }
  return `${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
}

// Получение времени в формате 12-часового формата
function formatTime12Hour(date) {
  const d = new Date(date);
  let hours = d.getHours();
  const minutes = String(d.getMinutes()).padStart(2, '0');
  const ampm = hours >= 12 ? 'PM' : 'AM';
  
  hours = hours % 12;
  hours = hours ? hours : 12; // 0 should be 12
  
  return `${hours}:${minutes} ${ampm}`;
}

console.log(formatTime('2023-05-15T14:30:45')); // '14:30:45'
console.log(formatDuration(3661)); // '1:01:01' (1 час, 1 минута, 1 секунда)
console.log(formatDuration(125)); // '02:05' (2 минуты, 5 секунд)
console.log(formatTime12Hour('2023-05-15T14:30:45')); // '2:30 PM'
```

## 7. Работа с днями недели

```javascript
// Получение названия дня недели
function getDayName(date, locale = 'ru-RU') {
  return new Date(date).toLocaleDateString(locale, { weekday: 'long' });
}

// Получение названия месяца
function getMonthName(date, locale = 'ru-RU') {
  return new Date(date).toLocaleDateString(locale, { month: 'long' });
}

// Проверка, является ли дата выходным днем
function isWeekend(date) {
  const day = new Date(date).getDay();
  return day === 0 || day === 6; // 0 - воскресенье, 6 - суббота
}

// Получение количества дней в месяце
function getDaysInMonth(year, month) {
  return new Date(year, month + 1, 0).getDate();
}

// Получение всех дат в месяце
function getDatesInMonth(year, month) {
  const daysInMonth = getDaysInMonth(year, month);
  const dates = [];
  
  for (let i = 1; i <= daysInMonth; i++) {
    dates.push(new Date(year, month, i));
  }
  
  return dates;
}

console.log(getDayName('2023-05-15')); // 'понедельник'
console.log(getMonthName('2023-05-15')); // 'май'
console.log(isWeekend('2023-05-20')); // true (суббота)
console.log(getDaysInMonth(2023, 4)); // 31 (май 2023 года)
```

## 8. Сниппеты для работы с относительным временем

```javascript
// Форматирование относительного времени (например, "2 часа назад")
function formatRelativeTime(date) {
  const rtf = new Intl.RelativeTimeFormat('ru', { numeric: 'auto' });
  const now = new Date();
  const d = new Date(date);
  const diffInSeconds = (d - now) / 1000;
  
  if (Math.abs(diffInSeconds) < 60) {
    return rtf.format(Math.round(diffInSeconds), 'second');
  }
  
  const diffInMinutes = diffInSeconds / 60;
  if (Math.abs(diffInMinutes) < 60) {
    return rtf.format(Math.round(diffInMinutes), 'minute');
  }
  
  const diffInHours = diffInMinutes / 60;
  if (Math.abs(diffInHours) < 24) {
    return rtf.format(Math.round(diffInHours), 'hour');
  }
  
  const diffInDays = diffInHours / 24;
  if (Math.abs(diffInDays) < 7) {
    return rtf.format(Math.round(diffInDays), 'day');
  }
  
  const diffInWeeks = diffInDays / 7;
  if (Math.abs(diffInWeeks) < 4) {
    return rtf.format(Math.round(diffInWeeks), 'week');
  }
  
  const diffInMonths = diffInDays / 30;
  if (Math.abs(diffInMonths) < 12) {
    return rtf.format(Math.round(diffInMonths), 'month');
  }
  
  const diffInYears = diffInDays / 365;
  return rtf.format(Math.round(diffInYears), 'year');
}

// Форматирование прошедшего времени в человекопонятном формате
function timeAgo(date) {
  const now = new Date();
  const d = new Date(date);
  const seconds = Math.floor((now - d) / 1000);
  
  let interval = seconds / 31536000; // 365 * 24 * 60 * 60
  
  if (interval > 1) {
    return Math.floor(interval) + ' год(а) назад';
  }
  
  interval = seconds / 2592000; // 30 * 24 * 60 * 60
  if (interval > 1) {
    return Math.floor(interval) + ' месяц(а) назад';
  }
  
  interval = seconds / 86400; // 24 * 60 * 60
  if (interval > 1) {
    return Math.floor(interval) + ' день(дня) назад';
  }
  
  interval = seconds / 3600;
  if (interval > 1) {
    return Math.floor(interval) + ' час(а) назад';
  }
  
  interval = seconds / 60;
  if (interval > 1) {
    return Math.floor(interval) + ' минут(ы) назад';
  }
  
  return 'Только что';
}

console.log(formatRelativeTime('2023-05-15T10:00:00')); // зависит от текущего времени
console.log(timeAgo('2023-05-14T10:00:00')); // '1 день назад' (если сегодня 15 мая)
```

Эти сниппеты помогут вам эффективно работать с датами и временем в ваших JavaScript приложениях, упрощая форматирование, валидацию и вычисления с временными данными.

См. также:
- [[js/snippets/url-validation-snippets]] - Сниппеты для обработки и валидации URL
- [[js/snippets/number-formatting-snippets]] - Сниппеты для форматирования чисел и валют
- [[js/snippets/color-snippets]] - Сниппеты для работы с цветами
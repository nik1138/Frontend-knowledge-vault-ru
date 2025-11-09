# Упражнения по модулям

## Упражнение 2: Создание модульной системы
Создайте модульную систему для математических операций:

```javascript
// math-core.js - основной модуль
const MathCore = (function() {
  // Приватные переменные и функции
  const PI = Math.PI;
  const E = Math.E;
  
  function validateNumber(num) {
    if (typeof num !== 'number' || isNaN(num)) {
      throw new Error('Аргумент должен быть числом');
    }
  }
  
  function validateArray(arr) {
    if (!Array.isArray(arr) || arr.length === 0) {
      throw new Error('Аргумент должен быть непустым массивом');
    }
  }
  
  // Публичный API
  return {
    // Базовые операции
    add: (a, b) => {
      validateNumber(a);
      validateNumber(b);
      return a + b;
    },
    
    subtract: (a, b) => {
      validateNumber(a);
      validateNumber(b);
      return a - b;
    },
    
    multiply: (a, b) => {
      validateNumber(a);
      validateNumber(b);
      return a * b;
    },
    
    divide: (a, b) => {
      validateNumber(a);
      validateNumber(b);
      if (b === 0) {
        throw new Error('Деление на ноль');
      }
      return a / b;
    },
    
    // Продвинутые операции
    power: (base, exponent) => {
      validateNumber(base);
      validateNumber(exponent);
      return Math.pow(base, exponent);
    },
    
    sqrt: (num) => {
      validateNumber(num);
      if (num < 0) {
        throw new Error('Квадратный корень из отрицательного числа');
      }
      return Math.sqrt(num);
    },
    
    // Статистические функции
    mean: (arr) => {
      validateArray(arr);
      return arr.reduce((sum, num) => {
        validateNumber(num);
        return sum + num;
      }, 0) / arr.length;
    },
    
    median: (arr) => {
      validateArray(arr);
      const sorted = arr.map(num => {
        validateNumber(num);
        return num;
      }).sort((a, b) => a - b);
      
      const mid = Math.floor(sorted.length / 2);
      return sorted.length % 2 !== 0 ? sorted[mid] : (sorted[mid - 1] + sorted[mid]) / 2;
    },
    
    // Константы
    get PI() { return PI; },
    get E() { return E; }
  };
})();

// math-advanced.js - расширенный модуль
const MathAdvanced = (function(core) {
  // Проверка зависимости
  if (!core) {
    throw new Error('Требуется MathCore');
  }
  
  return {
    // Тригонометрические функции
    sin: (angle) => {
      core.validateNumber(angle);
      return Math.sin(angle);
    },
    
    cos: (angle) => {
      core.validateNumber(angle);
      return Math.cos(angle);
    },
    
    tan: (angle) => {
      core.validateNumber(angle);
      return Math.tan(angle);
    },
    
    // Логарифмы
    log: (num, base = Math.E) => {
      core.validateNumber(num);
      core.validateNumber(base);
      if (num <= 0) {
        throw new Error('Логарифм определен только для положительных чисел');
      }
      if (base <= 0 || base === 1) {
        throw new Error('Основание логарифма должно быть положительным и не равным 1');
      }
      
      return base === Math.E ? Math.log(num) : Math.log(num) / Math.log(base);
    },
    
    // Факториал
    factorial: (n) => {
      core.validateNumber(n);
      if (n < 0 || !Number.isInteger(n)) {
        throw new Error('Факториал определен только для неотрицательных целых чисел');
      }
      
      if (n === 0 || n === 1) return 1;
      
      let result = 1;
      for (let i = 2; i <= n; i++) {
        result *= i;
      }
      return result;
    },
    
    // Комбинаторика
    combinations: (n, k) => {
      core.validateNumber(n);
      core.validateNumber(k);
      
      if (n < 0 || k < 0 || !Number.isInteger(n) || !Number.isInteger(k)) {
        throw new Error('n и k должны быть неотрицательными целыми числами');
      }
      
      if (k > n) return 0;
      
      return MathAdvanced.factorial(n) / (MathAdvanced.factorial(k) * MathAdvanced.factorial(n - k));
    }
  };
})(MathCore);

// math-utils.js - утилиты
const MathUtils = (function(core, advanced) {
  return {
    // Генератор случайных чисел в диапазоне
    randomRange: (min, max) => {
      core.validateNumber(min);
      core.validateNumber(max);
      if (min > max) {
        [min, max] = [max, min]; //_swap
      }
      return Math.random() * (max - min) + min;
    },
    
    // Округление до заданного количества знаков
    roundTo: (num, decimals) => {
      core.validateNumber(num);
      core.validateNumber(decimals);
      if (!Number.isInteger(decimals) || decimals < 0) {
        throw new Error('Количество знаков должно быть неотрицательным целым числом');
      }
      
      const factor = Math.pow(10, decimals);
      return Math.round(num * factor) / factor;
    },
    
    // Преобразование углов
    degreesToRadians: (degrees) => {
      core.validateNumber(degrees);
      return degrees * (core.PI / 180);
    },
    
    radiansToDegrees: (radians) => {
      core.validateNumber(radians);
      return radians * (180 / core.PI);
    },
    
    // Форматирование чисел
    formatNumber: (num, options = {}) => {
      core.validateNumber(num);
      const {
        decimals = 2,
        thousandSeparator = ' ',
        decimalSeparator = '.'
      } = options;
      
      const rounded = MathUtils.roundTo(num, decimals);
      const [integer, decimal] = rounded.toString().split('.');
      
      // Форматирование целой части
      let formattedInteger = integer.replace(/\B(?=(\d{3})+(?!\d))/g, thousandSeparator);
      
      // Добавление дробной части
      if (decimal) {
        return formattedInteger + decimalSeparator + decimal;
      }
      
      return formattedInteger;
    }
  };
})(MathCore, MathAdvanced);

// Тестирование модулей
console.log('=== Тестирование MathCore ===');
console.log('Сложение:', MathCore.add(5, 3));
console.log('Вычитание:', MathCore.subtract(10, 4));
console.log('Умножение:', MathCore.multiply(6, 7));
console.log('Деление:', MathCore.divide(15, 3));
console.log('Степень:', MathCore.power(2, 8));
console.log('Квадратный корень:', MathCore.sqrt(16));
console.log('Среднее:', MathCore.mean([1, 2, 3, 4, 5]));
console.log('Медиана:', MathCore.median([1, 2, 3, 4, 5]));
console.log('PI:', MathCore.PI);
console.log('E:', MathCore.E);

console.log('\n=== Тестирование MathAdvanced ===');
console.log('Синус 90°:', MathAdvanced.sin(MathUtils.degreesToRadians(90)));
console.log('Косинус 0°:', MathAdvanced.cos(0));
console.log('Логарифм 100 по основанию 10:', MathAdvanced.log(100, 10));
console.log('Факториал 5:', MathAdvanced.factorial(5));
console.log('Сочетания из 5 по 2:', MathAdvanced.combinations(5, 2));

console.log('\n=== Тестирование MathUtils ===');
console.log('Случайное число от 1 до 10:', MathUtils.randomRange(1, 10));
console.log('Округление 3.14159 до 2 знаков:', MathUtils.roundTo(3.14159, 2));
console.log('45° в радианах:', MathUtils.degreesToRadians(45));
console.log('π радиан в градусах:', MathUtils.radiansToDegrees(Math.PI));
console.log('Форматирование 1234567.89:', MathUtils.formatNumber(1234567.89, {
  decimals: 2,
  thousandSeparator: ' ',
  decimalSeparator: ','
}));
```

#javascript #modules #advanced #math #utilities
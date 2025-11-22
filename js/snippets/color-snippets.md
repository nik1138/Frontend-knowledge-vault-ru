---
aliases: ["Сниппеты для работы с цветами", "Цвета в JavaScript"]
tags: [js, snippets, colors, frontend]
---

# Сниппеты для работы с цветами (HEX, RGB, HSL)

Практические сниппеты для работы с цветами в различных форматах (HEX, RGB, HSL) в JavaScript. Эти сниппеты помогут вам конвертировать, манипулировать и анализировать цвета в ваших веб-приложениях.

## 1. Конвертация между форматами цветов

```javascript
// Конвертация HEX в RGB
function hexToRgb(hex) {
  // Удаляем # если он есть
  const cleanHex = hex.replace(/^#/, '');
  
  // Обработка сокращенного формата (3 символа)
  const fullHex = cleanHex.length === 3 
    ? cleanHex.split('').map(char => char + char).join('') 
    : cleanHex;
  
  const r = parseInt(fullHex.substring(0, 2), 16);
  const g = parseInt(fullHex.substring(2, 4), 16);
  const b = parseInt(fullHex.substring(4, 6), 16);
  
  return { r, g, b };
}

// Конвертация RGB в HEX
function rgbToHex(r, g, b) {
  return '#' + [r, g, b].map(x => {
    const hex = x.toString(16);
    return hex.length === 1 ? '0' + hex : hex;
  }).join('');
}

// Конвертация RGB в HSL
function rgbToHsl(r, g, b) {
  r /= 255;
  g /= 255;
  b /= 255;
  
  const max = Math.max(r, g, b);
  const min = Math.min(r, g, b);
  let h, s, l = (max + min) / 2;
  
  if (max === min) {
    h = s = 0; // серый цвет
  } else {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    
    switch (max) {
      case r: h = (g - b) / d + (g < b ? 6 : 0); break;
      case g: h = (b - r) / d + 2; break;
      case b: h = (r - g) / d + 4; break;
    }
    
    h /= 6;
  }
  
  return {
    h: Math.round(h * 360),
    s: Math.round(s * 100),
    l: Math.round(l * 100)
  };
}

// Конвертация HSL в RGB
function hslToRgb(h, s, l) {
  h /= 360;
  s /= 100;
  l /= 100;
  
  const hue2rgb = (p, q, t) => {
    if (t < 0) t += 1;
    if (t > 1) t -= 1;
    if (t < 1/6) return p + (q - p) * 6 * t;
    if (t < 1/2) return q;
    if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
    return p;
  };
  
  let r, g, b;
  
  if (s === 0) {
    r = g = b = l; // серый цвет
  } else {
    const q = l < 0.5 ? l * (1 + s) : l + s - l * s;
    const p = 2 * l - q;
    r = hue2rgb(p, q, h + 1/3);
    g = hue2rgb(p, q, h);
    b = hue2rgb(p, q, h - 1/3);
  }
  
  return {
    r: Math.round(r * 255),
    g: Math.round(g * 255),
    b: Math.round(b * 255)
  };
}

// Конвертация HSL в HEX
function hslToHex(h, s, l) {
  const rgb = hslToRgb(h, s, l);
  return rgbToHex(rgb.r, rgb.g, rgb.b);
}

// Конвертация HEX в HSL
function hexToHsl(hex) {
  const rgb = hexToRgb(hex);
  return rgbToHsl(rgb.r, rgb.g, rgb.b);
}

console.log(hexToRgb('#ff5733')); // { r: 255, g: 87, b: 51 }
console.log(rgbToHex(255, 87, 51)); // '#ff5733'
console.log(rgbToHsl(255, 87, 51)); // { h: 11, s: 100, l: 60 }
console.log(hslToRgb(11, 100, 60)); // { r: 255, g: 87, b: 50 } (округление)
```

## 2. Работа с цветовыми значениями

```javascript
// Проверка валидности HEX цвета
function isValidHex(hex) {
  const hexRegex = /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/;
  return hexRegex.test(hex);
}

// Проверка валидности RGB цвета
function isValidRgb(r, g, b) {
  return r >= 0 && r <= 255 && g >= 0 && g <= 255 && b >= 0 && b <= 255;
}

// Проверка валидности HSL цвета
function isValidHsl(h, s, l) {
  return h >= 0 && h <= 360 && s >= 0 && s <= 100 && l >= 0 && l <= 100;
}

// Получение яркости цвета (от 0 до 255)
function getColorBrightness(hex) {
  const rgb = hexToRgb(hex);
  // Используем взвешенную формулу для восприятия яркости
  return Math.sqrt(
    0.299 * (rgb.r * rgb.r) + 
    0.587 * (rgb.g * rgb.g) + 
    0.114 * (rgb.b * rgb.b)
  );
}

// Определение, является ли цвет "светлым" или "темным"
function isLightColor(hex) {
  return getColorBrightness(hex) > 130;
}

// Определение, является ли цвет "теплым" или "холодным"
function isWarmColor(hex) {
  const rgb = hexToRgb(hex);
  return rgb.r > rgb.b; // упрощенная проверка
}

console.log(isValidHex('#ff5733')); // true
console.log(isValidHex('#xyz123')); // false
console.log(getColorBrightness('#ff5733')); // примерно 154.6
console.log(isLightColor('#ff5733')); // true
console.log(isWarmColor('#ff5733')); // true
```

## 3. Манипуляции с цветами

```javascript
// Изменение яркости цвета (от -100 до 100)
function adjustBrightness(hex, percent) {
  const rgb = hexToRgb(hex);
  
  // Применяем изменение к каждому компоненту
  const adjust = (color) => {
    const adjusted = Math.round(color * (1 + percent / 100));
    return Math.min(255, Math.max(0, adjusted));
  };
  
  return rgbToHex(adjust(rgb.r), adjust(rgb.g), adjust(rgb.b));
}

// Изменение насыщенности цвета (от -100 до 100)
function adjustSaturation(hex, percent) {
  const hsl = hexToHsl(hex);
  const adjusted = Math.min(100, Math.max(0, hsl.s + percent));
  return hslToHex(hsl.h, adjusted, hsl.l);
}

// Изменение оттенка цвета (от -360 до 360)
function adjustHue(hex, degrees) {
  const hsl = hexToHsl(hex);
  let adjusted = hsl.h + degrees;
  // Обработка переполнения
  while (adjusted < 0) adjusted += 360;
  while (adjusted >= 360) adjusted -= 360;
  
  return hslToHex(adjusted, hsl.s, hsl.l);
}

// Смешивание двух цветов
function mixColors(hex1, hex2, ratio = 0.5) {
  const rgb1 = hexToRgb(hex1);
  const rgb2 = hexToRgb(hex2);
  
  const r = Math.round(rgb1.r * (1 - ratio) + rgb2.r * ratio);
  const g = Math.round(rgb1.g * (1 - ratio) + rgb2.g * ratio);
  const b = Math.round(rgb1.b * (1 - ratio) + rgb2.b * ratio);
  
  return rgbToHex(r, g, b);
}

// Получение контрастного цвета
function getContrastColor(hex) {
  return isLightColor(hex) ? '#000000' : '#ffffff';
}

// Инвертирование цвета
function invertColor(hex) {
  const rgb = hexToRgb(hex);
  return rgbToHex(255 - rgb.r, 255 - rgb.g, 255 - rgb.b);
}

console.log(adjustBrightness('#ff5733', 20)); // более светлый цвет
console.log(adjustSaturation('#ff5733', -30)); // менее насыщенный цвет
console.log(adjustHue('#ff5733', 60)); // измененный оттенок
console.log(mixColors('#ff5733', '#3357ff', 0.5)); // смешанный цвет
console.log(getContrastColor('#ff5733')); // '#000000' или '#ffffff'
console.log(invertColor('#ff5733')); // инвертированный цвет
```

## 4. Генерация цветовых палитр

```javascript
// Генерация случайного цвета
function randomColor() {
  return '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0');
}

// Генерация случайного цвета в определенном диапазоне
function randomColorInRange(minH = 0, maxH = 360, minS = 50, maxS = 100, minL = 30, maxL = 80) {
  const h = Math.floor(Math.random() * (maxH - minH + 1)) + minH;
  const s = Math.floor(Math.random() * (maxS - minS + 1)) + minS;
  const l = Math.floor(Math.random() * (maxL - minL + 1)) + minL;
  
  return hslToHex(h, s, l);
}

// Генерация аналогичных цветов (находящихся рядом на цветовом круге)
function getAnalogousColors(hex, angle = 30) {
  const hsl = hexToHsl(hex);
  const color1 = hslToHex((hsl.h - angle + 360) % 360, hsl.s, hsl.l);
  const color2 = hslToHex((hsl.h + angle) % 360, hsl.s, hsl.l);
  
  return [color1, hex, color2];
}

// Генерация комплементарного цвета (противоположного на цветовом круге)
function getComplementaryColor(hex) {
  const hsl = hexToHsl(hex);
  const complementaryHue = (hsl.h + 180) % 360;
  return hslToHex(complementaryHue, hsl.s, hsl.l);
}

// Генерация триадической палитры
function getTriadicColors(hex) {
  const hsl = hexToHsl(hex);
  const hue1 = (hsl.h + 120) % 360;
  const hue2 = (hsl.h + 240) % 360;
  
  return [
    hex,
    hslToHex(hue1, hsl.s, hsl.l),
    hslToHex(hue2, hsl.s, hsl.l)
  ];
}

// Генерация цветов по оттенкам (оттенки одного цвета)
function getShades(hex, count = 5) {
  const hsl = hexToHsl(hex);
  const shades = [];
  
  for (let i = 0; i < count; i++) {
    const lightness = Math.round((100 / count) * i);
    shades.push(hslToHex(hsl.h, hsl.s, lightness));
  }
  
  return shades;
}

// Генерация оттенков (lighter variations)
function getTints(hex, count = 5) {
  const hsl = hexToHsl(hex);
  const tints = [];
  
  for (let i = 0; i < count; i++) {
    const lightness = Math.round(hsl.l + ((100 - hsl.l) / count) * i);
    tints.push(hslToHex(hsl.h, hsl.s, lightness));
  }
  
  return tints;
}

// Генерация тонов (darker variations)
function getTones(hex, count = 5) {
  const hsl = hexToHsl(hex);
  const tones = [];
  
  for (let i = 0; i < count; i++) {
    const lightness = Math.round(hsl.l - (hsl.l / count) * i);
    tones.push(hslToHex(hsl.h, hsl.s, lightness));
  }
  
  return tones;
}

console.log(randomColor()); // случайный цвет
console.log(randomColorInRange(0, 60, 70, 100, 40, 60)); // теплый насыщенный цвет
console.log(getAnalogousColors('#ff5733')); // аналогичные цвета
console.log(getComplementaryColor('#ff5733')); // комплементарный цвет
console.log(getTriadicColors('#ff5733')); // триадическая палитра
console.log(getShades('#ff5733', 5)); // 5 оттенков цвета
```

## 5. Работа с прозрачностью

```javascript
// Добавление альфа-канала к HEX
function hexToRgba(hex, alpha = 1) {
  const rgb = hexToRgb(hex);
  return `rgba(${rgb.r}, ${rgb.g}, ${rgb.b}, ${alpha})`;
}

// Извлечение альфа-канала из RGBA
function rgbaToHex(rgba) {
  const match = rgba.match(/rgba?\((\d+),\s*(\d+),\s*(\d+)(?:,\s*([\d.]+))?\)/);
  if (!match) return null;
  
  const r = parseInt(match[1]);
  const g = parseInt(match[2]);
  const b = parseInt(match[3]);
  
  return rgbToHex(r, g, b);
}

// Изменение прозрачности цвета
function adjustAlpha(hex, alpha) {
  return hexToRgba(hex, Math.max(0, Math.min(1, alpha)));
}

// Смешивание цветов с учетом прозрачности
function blendColorsWithAlpha(color1, alpha1, color2, alpha2) {
  const rgba1 = hexToRgb(color1);
  const rgba2 = hexToRgb(color2);
  
  // Формула альфа-смешивания
  const alpha = alpha1 + alpha2 * (1 - alpha1);
  const r = (rgba1.r * alpha1 + rgba2.r * alpha2 * (1 - alpha1)) / alpha;
  const g = (rgba1.g * alpha1 + rgba2.g * alpha2 * (1 - alpha1)) / alpha;
  const b = (rgba1.b * alpha1 + rgba2.b * alpha2 * (1 - alpha1)) / alpha;
  
  return {
    hex: rgbToHex(Math.round(r), Math.round(g), Math.round(b)),
    alpha: alpha
  };
}

console.log(hexToRgba('#ff5733', 0.5)); // 'rgba(255, 87, 51, 0.5)'
console.log(adjustAlpha('#ff5733', 0.7)); // 'rgba(255, 87, 51, 0.7)'
console.log(blendColorsWithAlpha('#ff5733', 0.5, '#3357ff', 0.3));
// смешанный цвет с альфа-каналом
```

## 6. Работа с именованными цветами

```javascript
// Словарь именованных цветов
const namedColors = {
  'aliceblue': '#f0f8ff',
  'antiquewhite': '#faebd7',
  'aqua': '#00ffff',
  'aquamarine': '#7fffd4',
  'azure': '#f0ffff',
  'beige': '#f5f5dc',
  'bisque': '#ffe4c4',
  'black': '#000000',
  'blanchedalmond': '#ffebcd',
  'blue': '#0000ff',
  'blueviolet': '#8a2be2',
  'brown': '#a52a2a',
  'burlywood': '#deb887',
  'cadetblue': '#5f9ea0',
  'chartreuse': '#7fff00',
  'chocolate': '#d2691e',
  'coral': '#ff7f50',
  'cornflowerblue': '#6495ed',
  'cornsilk': '#fff8dc',
  'crimson': '#dc143c',
  'cyan': '#00ffff',
  'darkblue': '#00008b',
  'darkcyan': '#008b8b',
  'darkgoldenrod': '#b8860b',
  'darkgray': '#a9a9a9',
  'darkgreen': '#006400',
  'darkkhaki': '#bdb76b',
  'darkmagenta': '#8b008b',
  'darkolivegreen': '#556b2f',
  'darkorange': '#ff8c00',
  'darkorchid': '#9932cc',
  'darkred': '#8b0000',
  'darksalmon': '#e9967a',
  'darkseagreen': '#8fbc8f',
  'darkslateblue': '#483d8b',
  'darkslategray': '#2f4f4f',
  'darkturquoise': '#00ced1',
  'darkviolet': '#9400d3',
  'deeppink': '#ff1493',
  'deepskyblue': '#00bfff',
  'dimgray': '#696969',
  'dodgerblue': '#1e90ff',
  'firebrick': '#b22222',
  'floralwhite': '#fffaf0',
  'forestgreen': '#228b22',
  'fuchsia': '#ff00ff',
  'gainsboro': '#dcdcdc',
  'ghostwhite': '#f8f8ff',
  'gold': '#ffd700',
  'goldenrod': '#daa520',
  'gray': '#808080',
  'green': '#008000',
  'greenyellow': '#adff2f',
  'honeydew': '#f0fff0',
  'hotpink': '#ff69b4',
  'indianred': '#cd5c5c',
  'indigo': '#4b0082',
  'ivory': '#fffff0',
  'khaki': '#f0e68c',
  'lavender': '#e6e6fa',
  'lavenderblush': '#fff0f5',
  'lawngreen': '#7cfc00',
  'lemonchiffon': '#fffacd',
  'lightblue': '#add8e6',
  'lightcoral': '#f08080',
  'lightcyan': '#e0ffff',
  'lightgoldenrodyellow': '#fafad2',
  'lightgray': '#d3d3d3',
  'lightgreen': '#90ee90',
  'lightpink': '#ffb6c1',
  'lightsalmon': '#ffa07a',
  'lightseagreen': '#20b2aa',
  'lightskyblue': '#87cefa',
  'lightslategray': '#778899',
  'lightsteelblue': '#b0c4de',
  'lightyellow': '#ffffe0',
  'lime': '#00ff00',
  'limegreen': '#32cd32',
  'linen': '#faf0e6',
  'magenta': '#ff00ff',
  'maroon': '#800000',
  'mediumaquamarine': '#66cdaa',
  'mediumblue': '#0000cd',
  'mediumorchid': '#ba55d3',
  'mediumpurple': '#9370db',
  'mediumseagreen': '#3cb371',
  'mediumslateblue': '#7b68ee',
  'mediumspringgreen': '#00fa9a',
  'mediumturquoise': '#48d1cc',
  'mediumvioletred': '#c71585',
  'midnightblue': '#191970',
  'mintcream': '#f5fffa',
  'mistyrose': '#ffe4e1',
  'moccasin': '#ffe4b5',
  'navajowhite': '#ffdead',
  'navy': '#000080',
  'oldlace': '#fdf5e6',
  'olive': '#808000',
  'olivedrab': '#6b8e23',
  'orange': '#ffa500',
  'orangered': '#ff4500',
  'orchid': '#da70d6',
  'palegoldenrod': '#eee8aa',
  'palegreen': '#98fb98',
  'paleturquoise': '#afeeee',
  'palevioletred': '#db7093',
  'papayawhip': '#ffefd5',
  'peachpuff': '#ffdab9',
  'peru': '#cd853f',
  'pink': '#ffc0cb',
  'plum': '#dda0dd',
  'powderblue': '#b0e0e6',
  'purple': '#800080',
  'rebeccapurple': '#663399',
  'red': '#ff0000',
  'rosybrown': '#bc8f8f',
  'royalblue': '#4169e1',
  'saddlebrown': '#8b4513',
  'salmon': '#fa8072',
  'sandybrown': '#f4a460',
  'seagreen': '#2e8b57',
  'seashell': '#fff5ee',
  'sienna': '#a0522d',
  'silver': '#c0c0c0',
  'skyblue': '#87ceeb',
  'slateblue': '#6a5acd',
  'slategray': '#708090',
  'snow': '#fffafa',
  'springgreen': '#00ff7f',
  'steelblue': '#4682b4',
  'tan': '#d2b48c',
  'teal': '#008080',
  'thistle': '#d8bfd8',
  'tomato': '#ff6347',
  'turquoise': '#40e0d0',
  'violet': '#ee82ee',
  'wheat': '#f5deb3',
  'white': '#ffffff',
  'whitesmoke': '#f5f5f5',
  'yellow': '#ffff00',
  'yellowgreen': '#9acd32'
};

// Преобразование именованного цвета в HEX
function namedColorToHex(colorName) {
  return namedColors[colorName.toLowerCase()] || null;
}

// Нахождение ближайшего именованного цвета
function findNearestNamedColor(hex) {
  const rgb = hexToRgb(hex);
  let minDistance = Infinity;
  let nearestColor = '';
  
  for (const [name, hexValue] of Object.entries(namedColors)) {
    const colorRgb = hexToRgb(hexValue);
    const distance = Math.sqrt(
      Math.pow(rgb.r - colorRgb.r, 2) +
      Math.pow(rgb.g - colorRgb.g, 2) +
      Math.pow(rgb.b - colorRgb.b, 2)
    );
    
    if (distance < minDistance) {
      minDistance = distance;
      nearestColor = name;
    }
  }
  
  return nearestColor;
}

console.log(namedColorToHex('red')); // '#ff0000'
console.log(findNearestNamedColor('#ff5733')); // ближайший именованный цвет
```

## 7. Утилиты для CSS

```javascript
// Преобразование цвета в CSS-совместимый формат
function toCssColor(color, format = 'hex') {
  if (format === 'hex') return color;
  if (format === 'rgb') {
    const rgb = hexToRgb(color);
    return `rgb(${rgb.r}, ${rgb.g}, ${rgb.b})`;
  }
  if (format === 'rgba') {
    const rgb = hexToRgb(color);
    return `rgba(${rgb.r}, ${rgb.g}, ${rgb.b}, 1)`;
  }
  if (format === 'hsl') {
    const hsl = hexToHsl(color);
    return `hsl(${hsl.h}, ${hsl.s}%, ${hsl.l}%)`;
  }
  return color;
}

// Генерация CSS-переменных для цветовой палитры
function generateColorCssVariables(colors, prefix = 'color') {
  return colors.map((color, index) => 
    `--${prefix}-${index + 1}: ${color};`
  ).join('\n');
}

// Создание градиента из двух цветов
function createGradient(color1, color2, direction = 'to right') {
  return `linear-gradient(${direction}, ${toCssColor(color1, 'rgb')}, ${toCssColor(color2, 'rgb')})`;
}

// Проверка контрастности двух цветов (для доступности)
function checkColorContrast(hex1, hex2) {
  const brightness1 = getColorBrightness(hex1);
  const brightness2 = getColorBrightness(hex2);
  const contrastRatio = brightness1 > brightness2 
    ? (brightness1 + 0.05) / (brightness2 + 0.05)
    : (brightness2 + 0.05) / (brightness1 + 0.05);
  
  return {
    ratio: parseFloat(contrastRatio.toFixed(2)),
    isAccessible: contrastRatio >= 4.5 // WCAG AA стандарт
  };
}

console.log(toCssColor('#ff5733', 'rgb')); // 'rgb(255, 87, 51)'
console.log(createGradient('#ff5733', '#3357ff')); // 'linear-gradient(to right, rgb(255, 87, 51), rgb(51, 87, 255))'
console.log(checkColorContrast('#ffffff', '#000000')); // { ratio: 21, isAccessible: true }
```

Эти сниппеты помогут вам эффективно работать с цветами в ваших JavaScript приложениях, позволяя конвертировать, манипулировать и анализировать цвета в различных форматах.

См. также:
- [[js/snippets/date-time-snippets]] - Сниппеты для работы с датами и временем
- [[js/snippets/url-validation-snippets]] - Сниппеты для обработки и валидации URL
- [[js/snippets/number-formatting-snippets]] - Сниппеты для форматирования чисел и валют
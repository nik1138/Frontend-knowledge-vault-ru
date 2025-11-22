---
aliases: [CSS Text Specification, CSS Text Properties, CSS Text Layout, CSS Text Module]
tags: [css, text, typography, specification, css-text]
---

# CSS Text Module: Текст, шрифты, выравнивание, разметка - Полное руководство

## Введение

CSS Text Module определяет свойства, влияющие на обработку и отображение текста в веб-документах. Модуль включает в себя свойства для управления выравниванием текста, разрывами строк, трансформациями текста, интервалами и другими аспектами текстовой разметки.

## CSS Text Module Level 3

Level 3 определяет основные свойства обработки текста.

### Выравнивание текста

```css
.text-alignment {
  /* Горизонтальное выравнивание */
  text-align: left;                 /* По левому краю */
  text-align: right;                /* По правому краю */
  text-align: center;               /* По центру */
  text-align: justify;              /* По ширине */
  
  /* Новые значения для выравнивания */
  text-align: start;                /* Начало направления чтения */
  text-align: end;                  /* Конец направления чтения */
  text-align: match-parent;         /* Соответствие родительскому элементу */
  
  /* Выравнивание последней строки */
  text-align-last: auto;            /* Автоматическое выравнивание */
  text-align-last: left;            /* По левому краю */
  text-align-last: right;           /* По правому краю */
  text-align-last: center;          /* По центру */
  text-align-last: justify;         /* По ширине */
}
```

### Интервалы текста

```css
.text-spacing {
  /* Межбуквенный интервал */
  letter-spacing: normal;           /* Нормальный интервал */
  letter-spacing: 2px;              /* Фиксированный интервал */
  letter-spacing: 0.1em;            /* Относительный интервал */
  
  /* Межсловный интервал */
  word-spacing: normal;             /* Нормальный интервал */
  word-spacing: 5px;                /* Фиксированный интервал */
  word-spacing: 0.2em;              /* Относительный интервал */
  
  /* Межстрочный интервал */
  line-height: normal;              /* Нормальная высота строки */
  line-height: 1.5;                 /* Относительный множитель */
  line-height: 24px;                /* Фиксированная высота */
  line-height: 150%;                /* Процентная высота */
}
```

### Преобразование текста

```css
.text-transformation {
  /* Преобразование регистра */
  text-transform: none;             /* Без преобразования */
  text-transform: capitalize;       /* Каждое слово с заглавной буквы */
  text-transform: uppercase;        /* Все заглавные */
  text-transform: lowercase;        /* Все строчные */
  
  /* Новые значения */
  text-transform: full-width;       /* Полноширинные символы */
  text-transform: full-size-kana;   /* Полноширинная катакана */
}
```

### Декорирование текста

```css
.text-decoration {
  /* Основные свойства */
  text-decoration-line: none;       /* Без линии */
  text-decoration-line: underline;  /* Подчеркивание */
  text-decoration-line: overline;   /* Надчеркивание */
  text-decoration-line: line-through; /* Зачеркивание */
  text-decoration-line: underline overline; /* Несколько линий */
  
  /* Цвет декорации */
  text-decoration-color: currentColor; /* Цвет текста */
  text-decoration-color: red;       /* Фиксированный цвет */
  
  /* Стиль линии */
  text-decoration-style: solid;     /* Сплошная */
  text-decoration-style: double;    /* Двойная */
  text-decoration-style: dotted;    /* Пунктир */
  text-decoration-style: dashed;    /* Штрих */
  text-decoration-style: wavy;      /* Волнистая */
  
  /* Толщина линии */
  text-decoration-thickness: auto;  /* Автоматическая толщина */
  text-decoration-thickness: 2px;   /* Фиксированная толщина */
  text-decoration-thickness: from-font; /* Из шрифта */
  
  /* Краткая запись */
  text-decoration: underline red wavy;
  text-decoration: underline overline red solid 2px;
}
```

### Теневое оформление текста

```css
.text-shadow {
  /* Без тени */
  text-shadow: none;
  
  /* Одна тень */
  text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
  
  /* Несколько теней */
  text-shadow: 
    1px 1px 2px black, 
    0 0 1em blue, 
    0 0 0.2em blue;
}
```

### Разрывы строк и слов

```css
.line-breaks {
  /* Перенос строк */
  word-break: normal;               /* Нормальный перенос */
  word-break: break-all;            /* Перенос всех слов */
  word-break: keep-all;             /* Сохранение слов без переноса */
  
  /* Перенос слов */
  overflow-wrap: normal;            /* Нормальный перенос */
  overflow-wrap: break-word;        /* Перенос длинных слов */
  
  /* Перенос в словах */
  hyphens: none;                    /* Без переносов */
  hyphens: manual;                  /* Только ручные переносы */
  hyphens: auto;                    /* Автоматические переносы */
  
  /* Разрыв строк */
  line-break: auto;                 /* Автоматический разрыв */
  line-break: loose;                /* Свободный разрыв */
  line-break: normal;               /* Нормальный разрыв */
  line-break: strict;               /* Строгий разрыв */
}
```

## CSS Text Module Level 4

Level 4 добавляет расширенные возможности обработки текста.

### Улучшенные свойства декорирования

```css
.enhanced-text-decoration {
  /* Подчеркивание текста */
  text-underline-position: auto;    /* Автоматическая позиция */
  text-underline-position: under;   /* Под базовой линией */
  text-underline-position: left;    /* Для вертикального текста */
  text-underline-position: right;   /* Для вертикального текста */
  
  /* Высота подчеркивания */
  text-underline-offset: auto;      /* Автоматическое смещение */
  text-underline-offset: 3px;       /* Фиксированное смещение */
}
```

### Свойства текстового выравнивания

```css
.advanced-text-align {
  /* Пространство между словами */
  text-justify: auto;               /* Автоматическое выравнивание */
  text-justify: inter-word;         /* Пространство между словами */
  text-justify: inter-character;    /* Пространство между символами */
  text-justify: none;               /* Без выравнивания */
}
```

### Свойства текстовой ориентации

```css
.text-orientation {
  /* Ориентация текста */
  text-orientation: mixed;          /* Смешанная ориентация */
  text-orientation: upright;        /* Вертикальные символы */
  text-orientation: sideways;       /* Повернутый текст */
  text-orientation: sideways-right; /* Повернутый текст вправо */
}
```

### Свойства направления текста

```css
.writing-modes {
  /* Направление письма */
  writing-mode: horizontal-tb;      /* Горизонтальное сверху вниз */
  writing-mode: vertical-rl;        /* Вертикальное справа налево */
  writing-mode: vertical-lr;        /* Вертикальное слева направо */
  
  /* Направление строк */
  direction: ltr;                   /* Слева направо */
  direction: rtl;                   /* Справа налево */
  
  /* Упорядочивание столбцов */
  text-combine-upright: none;       /* Без объединения */
  text-combine-upright: all;        /* Объединить все символы */
  text-combine-upright: digits 2;   /* Объединить 2 цифры */
}
```

## Практические применения

### Типографика

```css
.typography-styles {
  font-family: 'Georgia', 'Times New Roman', serif;
  font-size: 16px;
  line-height: 1.6;
  letter-spacing: 0.01em;
  text-align: justify;
  text-justify: inter-word;
}

.heading {
  font-family: 'Arial', 'Helvetica', sans-serif;
  font-size: 2em;
  font-weight: bold;
  line-height: 1.2;
  text-align: center;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  margin-bottom: 1em;
}

.quote {
  font-style: italic;
  text-align: center;
  padding: 20px;
  margin: 20px 0;
  border-left: 4px solid #ccc;
  color: #666;
  font-size: 1.2em;
  line-height: 1.5;
}
```

### Адаптивный текст

```css
.responsive-text {
  font-size: clamp(1rem, 2.5vw, 2rem); /* Адаптивный размер шрифта */
  line-height: 1.4;
  text-align: left;
  overflow-wrap: break-word;        /* Перенос длинных слов */
  word-break: break-word;           /* Альтернатива overflow-wrap */
}

.truncated-text {
  display: -webkit-box;
  -webkit-line-clamp: 3;            /* Показывать только 3 строки */
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
  line-height: 1.5;
}
```

### Эффекты текста

```css
.text-effects {
  /* Градиентный текст */
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.glowing-text {
  text-shadow: 
    0 0 5px #fff, 
    0 0 10px #fff, 
    0 0 15px #007bff, 
    0 0 20px #007bff;
}

.embossed-text {
  text-shadow: 
    0 1px 0 #fff,
    0 -1px 0 #000;
  color: #888;
}

/* Анимированный текст */
.animated-text {
  animation: textPulse 2s infinite;
}

@keyframes textPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}
```

### Вертикальные текстовые режимы

```css
.vertical-text {
  writing-mode: vertical-rl;        /* Вертикальный режим */
  text-orientation: upright;        /* Вертикальные символы */
  letter-spacing: 0.2em;            /* Увеличенное межбуквенное расстояние */
}

.vertical-headers {
  writing-mode: vertical-lr;
  text-align: center;
  padding: 10px;
}
```

### Интернационализация

```css
.rtl-text {
  direction: rtl;                   /* Справа налево */
  text-align: right;
}

.bidi-override {
  unicode-bidi: embed;              /* Встраивание двунаправленного текста */
  direction: ltr;
}

.mixed-text {
  unicode-bidi: bidi-override;      /* Переопределение двунаправленности */
}
```

## Современные возможности

### CSS Logical Properties в тексте

```css
.logical-text {
  text-align: start;                /* Логическое выравнивание */
  margin-inline-start: 10px;        /* Логические отступы */
  margin-inline-end: 10px;
  padding-inline-start: 15px;
  padding-inline-end: 15px;
}
```

### Улучшенные шрифтовые свойства

```css
.font-enhancements {
  font-variant-ligatures: common-ligatures; /* Обычные лигатуры */
  font-kerning: normal;             /* Кернинг шрифта */
  font-feature-settings: "kern", "liga"; /* Дополнительные настройки шрифта */
}
```

### Адаптивные текстовые свойства

```css
.adaptive-typography {
  font-size: clamp(1rem, 2.5vw, 3rem);
  line-height: clamp(1.2, 0.1vw + 1.4, 1.6);
  letter-spacing: clamp(-0.01em, 0.01vw, 0.02em);
}
```

## Поддержка браузерами

- `text-align`, `text-transform`, `text-decoration`: Поддерживается во всех браузерах
- `text-shadow`: Поддерживается во всех современных браузерах
- `line-height`, `letter-spacing`, `word-spacing`: Поддерживается во всех браузерах
- `hyphens`: Ограниченная поддержка, особенно в IE
- `text-underline-position`, `text-underline-offset`: Хорошая поддержка в современных браузерах
- `writing-mode`: Хорошая поддержка, с префиксами для старых браузеров

## Связанные темы

- [[CSS Typography]] - типографика
- [[CSS Writing Modes]] - режимы письма
- [[CSS Fonts Module]] - модуль шрифтов

## Заключение

CSS Text Module предоставляет мощные инструменты для управления отображением текста. Понимание свойств текста позволяет создавать читаемые, доступные и визуально привлекательные интерфейсы с качественной типографикой.
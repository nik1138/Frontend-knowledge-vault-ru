---
aliases: ["Руководящие принципы доступности веб-контента", "WCAG", "Стандарты доступности"]
tags: ["#web-accessibility", "#standards", "#wcag", "#typescript"]
---

# WCAG-стандарты

## Обзор

Руководящие принципы доступности веб-контента (Web Content Accessibility Guidelines, WCAG) представляют собой международный стандарт, определяющий, как веб-контент должен быть разработан и разработан для обеспечения доступности людям с ограниченными возможностями. Эти принципы являются основой для создания доступных веб-приложений, особенно при использовании таких технологий, как TypeScript.

## Уровни соответствия

WCAG 2.1 (и более новые версии) определяют три уровня соответствия:

- **A (Минимальный уровень)**: Базовые функции доступности
- **AA (Средний уровень)**: Наиболее распространенный уровень для правительственных и коммерческих сайтов
- **AAA (Высший уровень)**: Расширенные критерии доступности

## Четыре основных принципа (POUR)

### 1. Воспринимаемость (Perceivable)
Информация должна быть представлена способами, которые пользователи могут воспринимать.

```typescript
// Пример: Обеспечение альтернативного текста для изображений
interface ImageProps {
  src: string;
  alt: string; // Критически важный атрибут для доступности
  title?: string;
}

const AccessibleImage: React.FC<ImageProps> = ({ src, alt, title }) => (
  <img src={src} alt={alt} title={title} />
);
```

### 2. Управляемость (Operable)
Интерфейс элементы должны быть управляемыми с помощью различных способов ввода.

```typescript
// Пример: Обеспечение клавиатурной навигации
interface ButtonProps {
  onClick: () => void;
  onKeyDown?: (e: React.KeyboardEvent) => void;
  children: React.ReactNode;
}

const AccessibleButton: React.FC<ButtonProps> = ({ onClick, onKeyDown, children }) => {
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      onClick();
    }
    onKeyDown?.(e);
  };

  return (
    <button 
      onClick={onClick} 
      onKeyDown={handleKeyDown}
      tabIndex={0}
    >
      {children}
    </button>
  );
};
```

### 3. Понятность (Understandable)
Информация и работа пользовательского интерфейса должны быть понятными.

```typescript
// Пример: Понятные и описательные метки для форм
interface FormFieldProps {
  label: string;
  id: string;
  type: string;
  required?: boolean;
}

const FormField: React.FC<FormFieldProps> = ({ label, id, type, required }) => (
  <div className="form-field">
    <label htmlFor={id}>
      {label}{required && <span aria-label="Обязательное поле">*</span>}
    </label>
    <input 
      type={type} 
      id={id} 
      required={required}
      aria-describedby={required ? `${id}-error` : undefined}
    />
    {required && (
      <span id={`${id}-error`} className="error-message" role="alert">
        Это поле обязательно
      </span>
    )}
  </div>
);
```

### 4. Масштабируемость (Robust)
Контент должен быть достаточно надежным, чтобы его можно было интерпретировать различными вспомогательными технологиями.

```typescript
// Пример: Использование семантически правильных HTML-элементов
interface NavigationProps {
  items: { text: string; href: string }[];
}

const Navigation: React.FC<NavigationProps> = ({ items }) => (
  <nav role="navigation" aria-label="Основная навигация">
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          <a href={item.href}>{item.text}</a>
        </li>
      ))}
    </ul>
  </nav>
);
```

## Критерии доступности (Success Criteria)

Каждый принцип имеет конкретные критерии доступности, которые можно проверить. Вот несколько примеров:

### 1.1.1 Нетекстовое содержимое (A)
Все нетекстовое содержимое, которое представлено пользователю, имеет текстовую альтернативу, которая служит эквивалентной целью.

```typescript
// Плохо
const BadImageExample = () => <img src="chart.png" />;

// Хорошо
const GoodImageExample = () => <img src="chart.png" alt="Диаграмма продаж за последний квартал" />;
```

### 1.3.1 Информация и отношения (A)
Информация, структура и отношения, представленные в содержимом, могут быть определены программно или доступны в тексте.

```typescript
// Плохо - семантически неправильная структура
const BadStructure = () => (
  <div>Заголовок</div>
  <div>Подзаголовок</div>
  <div>Параграф текста</div>
);

// Хорошо - семантически правильная структура
const GoodStructure = () => (
  <header>
    <h1>Заголовок</h1>
    <h2>Подзаголовок</h2>
    <p>Параграф текста</p>
  </header>
);
```

### 1.4.3 Контрастность (AA)
Цвет не должен быть единственным способом передачи информации. Контраст между цветами переднего плана и фона должен быть не менее 4.5:1 для нормального текста и 3:1 для крупного текста.

```typescript
// Пример проверки контрастности в TypeScript
interface ColorPair {
  foreground: string;
  background: string;
}

const calculateContrastRatio = (color1: string, color2: string): number => {
  // Реализация расчета контрастности
  const luminance1 = getRelativeLuminance(color1);
  const luminance2 = getRelativeLuminance(color2);
  const lighter = Math.max(luminance1, luminance2);
  const darker = Math.min(luminance1, luminance2);
  
  return (lighter + 0.05) / (darker + 0.05);
};

const getRelativeLuminance = (color: string): number => {
  // Конвертация HEX в RGB и расчет яркости
  const hex = color.replace('#', '');
  const r = parseInt(hex.substr(0, 2), 16) / 255;
  const g = parseInt(hex.substr(2, 2), 16) / 255;
  const b = parseInt(hex.substr(4, 2), 16) / 255;
  
  const sRGB = [r, g, b].map(c => 
    c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4)
  );
  
  return 0.2126 * sRGB[0] + 0.7152 * sRGB[1] + 0.0722 * sRGB[2];
};

const validateColorContrast = (colors: ColorPair): boolean => {
  const ratio = calculateContrastRatio(colors.foreground, colors.background);
  return ratio >= 4.5; // Для нормального текста
};
```

### 2.4.1 Обход (A)
Предоставляется возможность обхода повторяющихся блоков контента.

```typescript
// Пример: Пропуск к основному контенту
const SkipNavigation = () => (
  <a href="#main-content" className="skip-link">
    Перейти к основному содержанию
  </a>
);

const Layout: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <>
    <SkipNavigation />
    <header>Навигация и другие элементы</header>
    <main id="main-content">{children}</main>
    <footer>Футер</footer>
  </>
);
```

## TypeScript и WCAG

TypeScript может помочь в реализации WCAG-стандартов через:

1. **Строгую типизацию** для атрибутов доступности
2. **Интерфейсы и типы** для определения доступных компонентов
3. **Утилиты** для проверки соответствия стандартам

```typescript
// Пример: Типы для атрибутов доступности
type AriaAttributes = {
  'aria-label'?: string;
  'aria-labelledby'?: string;
  'aria-describedby'?: string;
  'aria-hidden'?: boolean;
  'aria-expanded'?: boolean;
  'aria-controls'?: string;
  'role'?: string;
};

interface AccessibleComponentProps extends AriaAttributes {
  children: React.ReactNode;
  className?: string;
}

// Пример: Утилита для проверки WCAG-критериев
class WCAGValidator {
  static validateImageAlt(img: HTMLImageElement): boolean {
    return img.hasAttribute('alt') && img.getAttribute('alt') !== null;
  }

  static validateLinkText(link: HTMLAnchorElement): boolean {
    const text = link.textContent?.trim();
    return text && text.length > 0;
  }
}
```

## Практические рекомендации

1. **Используйте семантические HTML-элементы** вместо div и span, когда это возможно
2. **Обеспечьте надлежащую иерархию заголовков** (h1-h6)
3. **Добавляйте атрибуты aria** только когда семантических элементов недостаточно
4. **Тестируйте с помощью вспомогательных технологий** (скринридеры, клавиатурная навигация)
5. **Используйте TypeScript для создания строго типизированных компонентов доступности**

## Связанные темы

- [[ARIA-атрибуты]]
- [[Семантический-HTML]]
- [[Доступные-компоненты]]
- [[Тестирование-доступности]]
- [[Интернационализация в TypeScript]]

## Внешние ресурсы

- [Официальный сайт WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/)
- [WCAG 2.1 на русском](https://docs-ru.translate.w3.org/WAI/WCAG21/quickref/)
- [Документация MDN по доступности](https://developer.mozilla.org/ru/docs/Web/Accessibility)

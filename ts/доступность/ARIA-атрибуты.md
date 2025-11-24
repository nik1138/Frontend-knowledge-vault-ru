---
aliases: ["Атрибуты доступности", "Роль, состояние и свойства", "Accessible Rich Internet Applications"]
tags: ["#web-accessibility", "#aria", "#accessibility", "#typescript"]
---

# ARIA-атрибуты

## Обзор

Accessible Rich Internet Applications (ARIA) - это набор атрибутов, которые помогают сделать веб-приложения более доступными для людей с ограниченными возможностями. ARIA-атрибуты предоставляют дополнительную информацию вспомогательным технологиям, таким как скринридеры, позволяя им лучше понимать структуру и поведение веб-страницы.

## Основные категории ARIA-атрибутов

### 1. Роли (Roles)
Определяют тип элемента (кнопка, заголовок, меню и т.д.)

```typescript
// Пример использования ARIA-ролей в React-компонентах
interface MenuItemProps {
  children: React.ReactNode;
  active?: boolean;
  disabled?: boolean;
}

const MenuItem: React.FC<MenuItemProps> = ({ children, active, disabled }) => (
  <div 
    role="menuitem" 
    aria-disabled={disabled}
    aria-current={active ? "page" : undefined}
    className={`menu-item ${active ? 'active' : ''} ${disabled ? 'disabled' : ''}`}
  >
    {children}
  </div>
);

interface DialogProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

const Dialog: React.FC<DialogProps> = ({ isOpen, onClose, title, children }) => {
  if (!isOpen) return null;

  return (
    <div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
      <h2 id="dialog-title">{title}</h2>
      <button aria-label="Закрыть" onClick={onClose}>×</button>
      {children}
    </div>
  );
};
```

### 2. Свойства (Properties)
Описывают характеристики элементов, которые не изменяются часто

```typescript
// Пример использования ARIA-свойств
interface ProgressBarProps {
  value: number;
  max: number;
  label: string;
}

const ProgressBar: React.FC<ProgressBarProps> = ({ value, max, label }) => (
  <div>
    <span id="progress-label">{label}</span>
    <div 
      role="progressbar"
      aria-labelledby="progress-label"
      aria-valuenow={value}
      aria-valuemin={0}
      aria-valuemax={max}
    >
      <div style={{ width: `${(value / max) * 100}%` }} />
    </div>
  </div>
);

interface TabPanelProps {
  children: React.ReactNode;
  active: boolean;
  id: string;
}

const TabPanel: React.FC<TabPanelProps> = ({ children, active, id }) => (
  <div 
    role="tabpanel" 
    id={id}
    hidden={!active}
    aria-hidden={!active}
  >
    {children}
  </div>
);
```

### 3. Состояния (States)
Описывают текущее состояние элемента, которое может изменяться

```typescript
// Пример использования ARIA-состояний
interface ToggleButtonProps {
  checked: boolean;
  onChange: (checked: boolean) => void;
  label: string;
}

const ToggleButton: React.FC<ToggleButtonProps> = ({ checked, onChange, label }) => (
  <button
    role="switch"
    aria-checked={checked}
    onClick={() => onChange(!checked)}
    aria-label={label}
  >
    {checked ? 'ВКЛ' : 'ВЫКЛ'}
  </button>
);

interface ComboboxProps {
  options: string[];
  value: string;
  onChange: (value: string) => void;
  label: string;
}

const Combobox: React.FC<ComboboxProps> = ({ options, value, onChange, label }) => {
  const [expanded, setExpanded] = React.useState(false);
  
  return (
    <div>
      <label id="combobox-label">{label}</label>
      <div
        role="combobox"
        aria-expanded={expanded}
        aria-haspopup="listbox"
        aria-owns="option-list"
        aria-labelledby="combobox-label"
      >
        <input
          type="text"
          value={value}
          onChange={(e) => onChange(e.target.value)}
          onKeyDown={(e) => {
            if (e.key === 'ArrowDown') setExpanded(true);
            if (e.key === 'Escape') setExpanded(false);
          }}
        />
        <ul 
          role="listbox" 
          id="option-list"
          hidden={!expanded}
        >
          {options.map((option, index) => (
            <li 
              role="option" 
              id={`option-${index}`}
              aria-selected={value === option}
              onClick={() => {
                onChange(option);
                setExpanded(false);
              }}
            >
              {option}
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
};
```

## Основные ARIA-атрибуты

### aria-label
Предоставляет текстовую метку элементу, когда визуальная метка отсутствует

```typescript
// Пример использования aria-label
interface IconButtonProps {
  onClick: () => void;
  icon: string;
  label: string;
}

const IconButton: React.FC<IconButtonProps> = ({ onClick, icon, label }) => (
  <button 
    onClick={onClick} 
    aria-label={label}
    className="icon-button"
  >
    {icon}
  </button>
);
```

### aria-labelledby
Связывает элемент с другим элементом, который служит его меткой

```typescript
// Пример использования aria-labelledby
interface FormSectionProps {
  titleId: string;
  children: React.ReactNode;
}

const FormSection: React.FC<FormSectionProps> = ({ titleId, children }) => (
  <section aria-labelledby={titleId}>
    <h2 id={titleId}>Настройки профиля</h2>
    {children}
  </section>
);
```

### aria-describedby
Связывает элемент с другим элементом, который содержит описание

```typescript
// Пример использования aria-describedby
interface InputWithDescriptionProps {
  id: string;
  label: string;
  description: string;
  value: string;
  onChange: (value: string) => void;
}

const InputWithDescription: React.FC<InputWithDescriptionProps> = ({ 
  id, 
  label, 
  description, 
  value, 
  onChange 
}) => (
  <div>
    <label htmlFor={id}>{label}</label>
    <input 
      id={id}
      type="text" 
      value={value} 
      onChange={(e) => onChange(e.target.value)}
      aria-describedby={`${id}-description`}
    />
    <span id={`${id}-description`} className="description">
      {description}
    </span>
  </div>
);
```

### aria-hidden
Скрывает элемент от вспомогательных технологий

```typescript
// Пример использования aria-hidden
interface DecorativeIconProps {
  icon: string;
  decorative?: boolean;
}

const DecorativeIcon: React.FC<DecorativeIconProps> = ({ icon, decorative = true }) => (
  <span 
    className="icon" 
    aria-hidden={decorative}
  >
    {icon}
  </span>
);
```

### aria-expanded
Указывает, расширен ли элемент или свернут

```typescript
// Пример использования aria-expanded
interface AccordionItemProps {
  title: string;
  children: React.ReactNode;
  expanded: boolean;
  onToggle: () => void;
}

const AccordionItem: React.FC<AccordionItemProps> = ({ 
  title, 
  children, 
  expanded, 
  onToggle 
}) => (
  <div className="accordion-item">
    <button
      aria-expanded={expanded}
      onClick={onToggle}
      aria-controls="accordion-content"
    >
      {title}
    </button>
    <div 
      id="accordion-content"
      hidden={!expanded}
      role="region"
      aria-labelledby="accordion-title"
    >
      {children}
    </div>
  </div>
);
```

## TypeScript-типы для ARIA-атрибутов

```typescript
// Типы для ARIA-атрибутов
type AriaRole = 
  | 'alert'
  | 'alertdialog'
  | 'button'
  | 'checkbox'
  | 'combobox'
  | 'dialog'
  | 'grid'
  | 'link'
  | 'listbox'
  | 'menu'
  | 'menubar'
  | 'option'
  | 'progressbar'
  | 'radio'
  | 'radiogroup'
  | 'scrollbar'
  | 'searchbox'
  | 'slider'
  | 'spinbutton'
  | 'switch'
  | 'tab'
  | 'table'
  | 'tablist'
  | 'tabpanel'
  | 'textbox'
  | 'toolbar'
  | 'tooltip';

interface AriaAttributes {
  'aria-activedescendant'?: string;
  'aria-atomic'?: boolean | 'false' | 'true';
  'aria-autocomplete'?: 'none' | 'inline' | 'list' | 'both';
  'aria-busy'?: boolean | 'false' | 'true';
  'aria-checked'?: boolean | 'false' | 'mixed' | 'true';
  'aria-colcount'?: number;
  'aria-colindex'?: number;
  'aria-colspan'?: number;
  'aria-controls'?: string;
  'aria-current'?: boolean | 'false' | 'true' | 'page' | 'step' | 'location' | 'date' | 'time';
  'aria-describedby'?: string;
  'aria-details'?: string;
  'aria-disabled'?: boolean | 'false' | 'true';
  'aria-dropeffect'?: 'none' | 'copy' | 'execute' | 'link' | 'move' | 'popup';
  'aria-errormessage'?: string;
  'aria-expanded'?: boolean | 'false' | 'true';
  'aria-flowto'?: string;
  'aria-grabbed'?: boolean | 'false' | 'true';
  'aria-haspopup'?: boolean | 'false' | 'true' | 'menu' | 'listbox' | 'tree' | 'grid' | 'dialog';
  'aria-hidden'?: boolean | 'false' | 'true';
  'aria-invalid'?: boolean | 'false' | 'true' | 'grammar' | 'spelling';
  'aria-keyshortcuts'?: string;
  'aria-label'?: string;
  'aria-labelledby'?: string;
  'aria-level'?: number;
  'aria-live'?: 'off' | 'polite' | 'assertive';
  'aria-modal'?: boolean | 'false' | 'true';
  'aria-multiline'?: boolean | 'false' | 'true';
  'aria-multiselectable'?: boolean | 'false' | 'true';
  'aria-orientation'?: 'horizontal' | 'vertical';
  'aria-owns'?: string;
  'aria-placeholder'?: string;
  'aria-posinset'?: number;
  'aria-pressed'?: boolean | 'false' | 'mixed' | 'true';
  'aria-readonly'?: boolean | 'false' | 'true';
  'aria-relevant'?: 'additions' | 'removals' | 'text' | 'all' | 'additions text';
  'aria-required'?: boolean | 'false' | 'true';
  'aria-roledescription'?: string;
  'aria-rowcount'?: number;
  'aria-rowindex'?: number;
  'aria-rowspan'?: number;
  'aria-selected'?: boolean | 'false' | 'true';
  'aria-setsize'?: number;
  'aria-sort'?: 'none' | 'ascending' | 'descending' | 'other';
  'aria-valuemax'?: number;
  'aria-valuemin'?: number;
  'aria-valuenow'?: number;
  'aria-valuetext'?: string;
}

// Комбинированный интерфейс для доступного компонента
interface AccessibleComponentProps extends AriaAttributes {
  role?: AriaRole;
  children: React.ReactNode;
  className?: string;
}
```

## Практические рекомендации

1. **Используйте семантически правильные HTML-элементы** перед добавлением ARIA-атрибутов
2. **Не переопределяйте нативные семантические роли** с помощью ARIA
3. **Обновляйте ARIA-атрибуты динамически** при изменении состояния компонента
4. **Используйте TypeScript для проверки корректности ARIA-атрибутов**

```typescript
// Пример: Компонент с динамическим обновлением ARIA-атрибутов
interface SearchInputProps {
  placeholder: string;
  onSearch: (query: string) => void;
}

const SearchInput: React.FC<SearchInputProps> = ({ placeholder, onSearch }) => {
  const [query, setQuery] = React.useState('');
  const [resultsCount, setResultsCount] = React.useState(0);
  const [searching, setSearching] = React.useState(false);
  const [resultsId] = React.useState(`search-results-${Math.random().toString(36).substr(2, 9)}`);

  const handleSearch = (e: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value);
    setSearching(true);
    
    // Имитация поиска
    setTimeout(() => {
      setResultsCount(Math.floor(Math.random() * 10)); // В реальном приложении - реальный результат поиска
      setSearching(false);
    }, 500);
  };

  return (
    <div className="search-container">
      <label htmlFor="search-input">Поиск</label>
      <input
        id="search-input"
        type="search"
        placeholder={placeholder}
        value={query}
        onChange={handleSearch}
        aria-describedby={resultsId}
        aria-busy={searching}
        aria-controls={resultsId}
      />
      <div 
        id={resultsId}
        role="status"
        aria-live="polite"
        className="search-results"
      >
        {searching ? 'Поиск...' : `${resultsCount} результатов`}
      </div>
    </div>
  );
};
```

## Распространенные ошибки

1. **Использование ARIA-атрибутов вместо семантических HTML-элементов**
2. **Неправильное использование ролей** (например, применение роли button к span без обработчиков)
3. **Забытые обновления атрибутов состояния** при изменении состояния компонента

## Связанные темы

- [[WCAG-стандарты]]
- [[Семантический-HTML]]
- [[Доступные-компоненты]]
- [[Тестирование-доступности]]

## Внешние ресурсы

- [Документация ARIA на MDN](https://developer.mozilla.org/ru/docs/Web/Accessibility/ARIA)
- [ARIA в спецификации W3C](https://www.w3.org/TR/wai-aria-1.1/)
- [Практическое руководство по ARIA](https://www.w3.org/TR/2017/NOTE-wai-aria-practices-1.1-20171214/)

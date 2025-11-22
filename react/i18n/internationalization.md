---
aliases: ["React i18n", "React перевод", "Мультиязычность в React"]
tags: 
  - #react
  - #internationalization
  - #localization
  - #translation
  - #frontend
---

# React Internationalization (i18n)

## Что такое международизация (i18n) и локализация (l10n)

**Международизация (i18n)** — это процесс разработки приложения таким образом, чтобы его можно было легко адаптировать для разных языков и культур без изменений в коде. Это подготовка инфраструктуры для мультиязычной поддержки.

**Локализация (l10n)** — это процесс адаптации международизированного приложения для конкретного языка и региона, включая перевод интерфейса, форматирование дат, чисел и т.д.

## Основные библиотеки для i18n в React

### react-i18next

Самая популярная библиотека для i18n в React, построенная на i18next:

```jsx
import { useTranslation } from 'react-i18next';

function MyComponent() {
  const { t, i18n } = useTranslation();
  
  const changeLanguage = (lng) => {
    i18n.changeLanguage(lng);
  };

  return (
    <div>
      <h1>{t('welcome')}</h1>
      <button onClick={() => changeLanguage('en')}>EN</button>
      <button onClick={() => changeLanguage('ru')}>RU</button>
    </div>
  );
}
```

### react-intl

Библиотека от FormatJS, обеспечивающая полную поддержку ICU сообщений:

```jsx
import { FormattedMessage, useIntl } from 'react-intl';

function MyComponent() {
  const intl = useIntl();
  
  return (
    <div>
      <FormattedMessage id="welcome" defaultMessage="Welcome" />
    </div>
  );
}
```

## Определение и переключение языка

### Автоматическое определение языка

```jsx
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

i18n
  .use(initReactI18next)
  .init({
    detection: {
      order: ['localStorage', 'navigator', 'htmlTag', 'path', 'subdomain'],
      caches: ['localStorage'],
    },
    fallbackLng: 'en',
    resources: {
      en: { translation: { welcome: 'Welcome' } },
      ru: { translation: { welcome: 'Добро пожаловать' } },
    }
  });
```

## Управление файлами перевода

Файлы переводов обычно организуются по языкам и пространствам имен:

```
locales/
├── en/
│   ├── translation.json
│   └── common.json
└── ru/
    ├── translation.json
    └── common.json
```

## Форматирование дат и чисел

### С помощью react-intl

```jsx
import { FormattedDate, FormattedNumber } from 'react-intl';

function FormattedComponent() {
  return (
    <div>
      <FormattedDate value={new Date()} />
      <FormattedNumber value={12345.67} style="currency" currency="USD" />
    </div>
  );
}
```

## Плюрализация в разных языках

### С react-i18next

```jsx
const count = 3;
const message = t('items', { count });

// В файле перевода:
// {
//   "items_one": "один элемент",
//   "items_few": "{{count}} элемента",
//   "items_many": "{{count}} элементов"
// }
```

### С react-intl

```jsx
<FormattedMessage
  id="items"
  defaultMessage="{count, plural, one {один элемент} few {# элемента} many {# элементов} other {# элементов}}"
  values={{ count: 3 }}
/>
```

## Поддержка RTL (справа налево)

```jsx
// Управление направлением текста
const [dir, setDir] = useState('ltr');

useEffect(() => {
  document.dir = dir;
}, [dir]);

// В CSS
[dir='rtl'] .container {
  text-align: right;
  direction: rtl;
}
```

## Контекст и хуки для перевода

### Создание контекста i18n

```jsx
import { createContext, useContext } from 'react';

const I18nContext = createContext();

export const I18nProvider = ({ children, translations, currentLang }) => {
  return (
    <I18nContext.Provider value={{ translations, currentLang }}>
      {children}
    </I18nContext.Provider>
  );
};

export const useI18n = () => useContext(I18nContext);
```

## Пространства имен и управление областью

```jsx
// Использование namespaces в react-i18next
const { t } = useTranslation('common'); // Использует namespace 'common'

// Загрузка нескольких namespaces
const { t } = useTranslation(['common', 'dashboard']);
```

## Ленивая загрузка переводов

```jsx
// Динамическая загрузка переводов
i18n.loadLanguages(['fr', 'de'], () => {
  // Переводы загружены
});
```

## SEO для мультиязычных сайтов

```jsx
import { Helmet } from 'react-helmet';

function SEOComponent() {
  return (
    <Helmet>
      <html lang={currentLanguage} />
      <link rel="alternate" hrefLang="en" href="https://example.com/en/" />
      <link rel="alternate" hrefLang="ru" href="https://example.com/ru/" />
      <link rel="alternate" hrefLang="x-default" href="https://example.com/" />
    </Helmet>
  );
}
```

## Производительность и соображения

- Кэширование переводов
- Ленивая загрузка больших файлов переводов
- Использование CDN для файлов переводов
- Оптимизация рендера компонентов с переводами

## Культурные различия

При разработке мультиязычных приложений учитывайте:
- Визуальные различия (цвета, иконки)
- Форматы дат, времени, чисел
- Направление текста
- Законодательные требования (GDPR, CCPA)

## Тестирование i18n

```jsx
// Тестирование компонентов с разными языками
it('renders in English', () => {
  render(<App />, { wrapper: I18nextProvider });
  expect(screen.getByText('Welcome')).toBeInTheDocument();
});

it('renders in Russian', () => {
  i18n.changeLanguage('ru');
  render(<App />, { wrapper: I18nextProvider });
  expect(screen.getByText('Добро пожаловать')).toBeInTheDocument();
});
```

## Связь с другими файлами React

- [[Типизация контекста в React с TypeScript]] - для управления состоянием языка
- [[Типизация хуков React в TypeScript]] - для создания кастомных хуков i18n
- [[Типизация компонентов React в TypeScript]] - для создания мультиязычных компонентов
- [[react/performance]] - для оптимизации производительности i18n
- [[react/testing]] - для тестирования мультиязычных компонентов
- [[react/styling]] - для поддержки RTL стилей

## Заключение

Реализация i18n в React требует тщательного планирования архитектуры приложения, выбора подходящей библиотеки и учета культурных различий. Правильная реализация i18n делает приложение доступным для глобальной аудитории.

> [!tip] 
> Используйте инструменты автоматизации для извлечения переводов и управления файлами локализации.

> [!warning]
> Не забывайте тестировать приложение на разных языках, особенно с RTL языками.
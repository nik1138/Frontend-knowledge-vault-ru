---
aliases: ["Семантическая разметка", "Семантические HTML-элементы", "Семантическая структура"]
tags: ["#web-accessibility", "#html", "#semantics", "#typescript"]
---

# Семантический HTML

## Обзор

Семантический HTML - это использование HTML-элементов по их прямому назначению, что помогает вспомогательным технологиям, таким как скринридеры, лучше понимать структуру и содержание веб-страницы. Это важный аспект доступности, особенно при разработке приложений с использованием TypeScript.

## Основные семантические элементы

### Структурные элементы

```typescript
// Пример использования семантических элементов в React-компонентах
interface PageLayoutProps {
  children: React.ReactNode;
  title: string;
}

const PageLayout: React.FC<PageLayoutProps> = ({ children, title }) => (
  <div className="page">
    <header>
      <h1>{title}</h1>
      <nav aria-label="Основная навигация">
        <ul>
          <li><a href="/">Главная</a></li>
          <li><a href="/about">О нас</a></li>
          <li><a href="/contact">Контакты</a></li>
        </ul>
      </nav>
    </header>
    
    <main>
      {children}
    </main>
    
    <footer>
      <p>&copy; {new Date().getFullYear()} Моя компания</p>
    </footer>
  </div>
);

interface ArticleLayoutProps {
  title: string;
  author: string;
  date: string;
  content: string;
}

const ArticleLayout: React.FC<ArticleLayoutProps> = ({ title, author, date, content }) => (
  <article>
    <header>
      <h1>{title}</h1>
      <p>
        Автор: <address>{author}</address> | 
        Опубликовано: <time dateTime={date}>{new Date(date).toLocaleDateString()}</time>
      </p>
    </header>
    <section>
      <p>{content}</p>
    </section>
    <footer>
      <p>Статья опубликована {new Date(date).toLocaleDateString()}</p>
    </footer>
  </article>
);
```

### Заголовки (h1-h6)

```typescript
// Пример правильной иерархии заголовков
interface ContentSectionProps {
  title: string;
  level: 1 | 2 | 3 | 4 | 5 | 6;
  children: React.ReactNode;
}

const ContentSection: React.FC<ContentSectionProps> = ({ title, level, children }) => {
  const HeadingTag = `h${level}` as keyof JSX.IntrinsicElements;
  
  return (
    <section>
      <HeadingTag>{title}</HeadingTag>
      {children}
    </section>
  );
};

// Использование:
const ExamplePage = () => (
  <main>
    <ContentSection title="Основной заголовок" level={1}>
      <ContentSection title="Подраздел 1" level={2}>
        <ContentSection title="Подподраздел 1.1" level={3}>
          <p>Содержимое подподраздела</p>
        </ContentSection>
      </ContentSection>
      <ContentSection title="Подраздел 2" level={2}>
        <p>Содержимое подраздела</p>
      </ContentSection>
    </ContentSection>
  </main>
);
```

### Списки

```typescript
// Пример использования различных типов списков
interface TodoListProps {
  items: { id: string; text: string; completed: boolean }[];
  onToggle: (id: string) => void;
}

const TodoList: React.FC<TodoListProps> = ({ items, onToggle }) => (
  <section>
    <h2>Список задач</h2>
    <ul role="list">
      {items.map(item => (
        <li key={item.id} role="listitem">
          <label>
            <input
              type="checkbox"
              checked={item.completed}
              onChange={() => onToggle(item.id)}
            />
            <span className={item.completed ? 'completed' : ''}>
              {item.text}
            </span>
          </label>
        </li>
      ))}
    </ul>
  </section>
);

interface NavigationListProps {
  items: { href: string; text: string }[];
}

const NavigationList: React.FC<NavigationListProps> = ({ items }) => (
  <nav aria-label="Навигация по сайту">
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

### Формы и элементы ввода

```typescript
// Пример семантически правильной формы
interface ContactFormProps {
  onSubmit: (data: { name: string; email: string; message: string }) => void;
}

const ContactForm: React.FC<ContactFormProps> = ({ onSubmit }) => {
  const [formData, setFormData] = React.useState({
    name: '',
    email: '',
    message: ''
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <fieldset>
        <legend>Контактная информация</legend>
        
        <div className="form-group">
          <label htmlFor="name">Имя:</label>
          <input
            type="text"
            id="name"
            name="name"
            value={formData.name}
            onChange={handleChange}
            required
          />
        </div>
        
        <div className="form-group">
          <label htmlFor="email">Email:</label>
          <input
            type="email"
            id="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            required
          />
        </div>
      </fieldset>
      
      <div className="form-group">
        <label htmlFor="message">Сообщение:</label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
          rows={5}
          required
        />
      </div>
      
      <button type="submit">Отправить</button>
    </form>
  );
};
```

## TypeScript-типы для семантических элементов

```typescript
// Типы для семантических элементов
type SemanticElement = 
  | 'article'
  | 'aside'
  | 'details'
  | 'figcaption'
  | 'figure'
  | 'footer'
  | 'header'
  | 'main'
  | 'mark'
  | 'nav'
  | 'section'
  | 'summary'
  | 'time';

type SemanticHeading = 
  | 'h1' 
  | 'h2' 
  | 'h3' 
  | 'h4' 
  | 'h5' 
  | 'h6';

type SemanticList = 
  | 'ul' 
  | 'ol' 
  | 'li' 
  | 'dl' 
  | 'dt' 
  | 'dd';

// Интерфейсы для семантических компонентов
interface SemanticSectionProps {
  heading: string;
  headingLevel: SemanticHeading;
  children: React.ReactNode;
  className?: string;
}

const SemanticSection: React.FC<SemanticSectionProps> = ({ 
  heading, 
  headingLevel, 
  children, 
  className 
}) => {
  const HeadingTag = headingLevel;
  
  return (
    <section className={className}>
      <HeadingTag>{heading}</HeadingTag>
      {children}
    </section>
  );
};

interface SemanticListProps {
  items: string[];
  ordered?: boolean;
  className?: string;
}

const SemanticList: React.FC<SemanticListProps> = ({ items, ordered = false, className }) => {
  const ListTag = ordered ? 'ol' : 'ul';
  
  return (
    <ListTag className={className}>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ListTag>
  );
};
```

## Практические рекомендации

### 1. Используйте правильные HTML-элементы

```typescript
// Плохо: использование div для кнопки
const BadButton = () => (
  <div 
    className="button-style" 
    onClick={() => console.log('Clicked')}
    tabIndex={0}
    role="button"
  >
    Нажми меня
  </div>
);

// Хорошо: использование семантически правильного элемента
const GoodButton: React.FC<{ onClick: () => void }> = ({ onClick }) => (
  <button onClick={onClick}>Нажми меня</button>
);

// Плохо: использование div для ссылки
const BadLink = () => (
  <div 
    className="link-style" 
    onClick={() => window.location.href = '/page'}
    tabIndex={0}
    role="link"
  >
    Перейти на страницу
  </div>
);

// Хорошо: использование семантически правильного элемента
const GoodLink: React.FC<{ href: string }> = ({ href }) => (
  <a href={href}>Перейти на страницу</a>
);
```

### 2. Поддерживайте иерархию заголовков

```typescript
// Пример компонента с правильной иерархией заголовков
interface DocumentStructureProps {
  title: string;
  sections: {
    title: string;
    subsections?: { title: string; content: string }[];
  }[];
}

const DocumentStructure: React.FC<DocumentStructureProps> = ({ title, sections }) => (
  <article>
    <header>
      <h1>{title}</h1>
    </header>
    
    <main>
      {sections.map((section, index) => (
        <section key={index}>
          <h2>{section.title}</h2>
          
          {section.subsections && section.subsections.map((subsection, subIndex) => (
            <section key={subIndex}>
              <h3>{subsection.title}</h3>
              <p>{subsection.content}</p>
            </section>
          ))}
        </section>
      ))}
    </main>
  </article>
);
```

### 3. Используйте семантические элементы для группировки

```typescript
// Пример семантически правильной структуры статьи
interface ArticleComponentProps {
  title: string;
  author: string;
  date: string;
  content: string;
  tags: string[];
  relatedArticles: { title: string; url: string }[];
}

const ArticleComponent: React.FC<ArticleComponentProps> = ({ 
  title, 
  author, 
  date, 
  content, 
  tags, 
  relatedArticles 
}) => (
  <article>
    <header>
      <h1>{title}</h1>
      <p>
        Автор: <address>{author}</address> | 
        <time dateTime={date}>{new Date(date).toLocaleDateString()}</time>
      </p>
    </header>
    
    <section>
      <p>{content}</p>
    </section>
    
    <aside>
      <h3>Теги</h3>
      <ul>
        {tags.map((tag, index) => (
          <li key={index}>{tag}</li>
        ))}
      </ul>
    </aside>
    
    <nav aria-label="Связанные статьи">
      <h3>Связанные статьи</h3>
      <ul>
        {relatedArticles.map((article, index) => (
          <li key={index}>
            <a href={article.url}>{article.title}</a>
          </li>
        ))}
      </ul>
    </nav>
    
    <footer>
      <p>Последнее обновление: <time dateTime={new Date().toISOString()}>{new Date().toLocaleDateString()}</time></p>
    </footer>
  </article>
);
```

## Семантические элементы и TypeScript

```typescript
// Типы для работы с семантическими элементами
type SemanticElementType = 
  | 'header' 
  | 'nav' 
  | 'main' 
  | 'article' 
  | 'section' 
  | 'aside' 
  | 'footer'
  | 'figure'
  | 'figcaption';

interface SemanticElementProps {
  element?: SemanticElementType;
  children: React.ReactNode;
  className?: string;
}

const SemanticWrapper: React.FC<SemanticElementProps> = ({ 
  element = 'div', 
  children, 
  className 
}) => {
  const Element = element as keyof JSX.IntrinsicElements;
  return <Element className={className}>{children}</Element>;
};

// Пример использования с TypeScript-проверками
const PageContent = () => (
  <main>
    <SemanticWrapper element="header" className="page-header">
      <h1>Заголовок страницы</h1>
    </SemanticWrapper>
    
    <SemanticWrapper element="article" className="content-article">
      <p>Основное содержимое статьи</p>
    </SemanticWrapper>
    
    <SemanticWrapper element="aside" className="sidebar">
      <h2>Боковая панель</h2>
      <p>Дополнительная информация</p>
    </SemanticWrapper>
  </main>
);
```

## Проверка семантической структуры

```typescript
// Утилита для проверки семантической структуры
class SemanticValidator {
  static validateHeadingHierarchy(elements: HTMLElement[]): boolean {
    let lastLevel = 0;
    
    for (const element of elements) {
      if (element.tagName.match(/^H[1-6]$/)) {
        const currentLevel = parseInt(element.tagName.charAt(1));
        
        // Проверяем, что уровень заголовка не превышает предыдущий + 1
        if (currentLevel > lastLevel + 1) {
          console.warn(`Нарушение иерархии заголовков: H${currentLevel} после H${lastLevel}`);
          return false;
        }
        
        lastLevel = currentLevel;
      }
    }
    
    return true;
  }
  
  static checkSemanticStructure(rootElement: HTMLElement): {
    hasMain: boolean;
    hasNav: boolean;
    hasHeader: boolean;
    hasFooter: boolean;
    headingHierarchyValid: boolean;
  } {
    const hasMain = !!rootElement.querySelector('main');
    const hasNav = !!rootElement.querySelector('nav');
    const hasHeader = !!rootElement.querySelector('header');
    const hasFooter = !!rootElement.querySelector('footer');
    
    const headings = Array.from(rootElement.querySelectorAll('h1, h2, h3, h4, h5, h6'));
    const headingHierarchyValid = this.validateHeadingHierarchy(headings as HTMLElement[]);
    
    return {
      hasMain,
      hasNav,
      hasHeader,
      hasFooter,
      headingHierarchyValid
    };
  }
}
```

## Распространенные ошибки

1. **Использование div и span вместо семантических элементов**
2. **Неправильная иерархия заголовков**
3. **Отсутствие основных структурных элементов (header, nav, main, footer)**
4. **Неправильное использование article и section**

## Связанные темы

- [[WCAG-стандарты]]
- [[ARIA-атрибуты]]
- [[Доступные-компоненты]]
- [[Тестирование-доступности]]
- [[Типизация DOM в TypeScript]]

## Внешние ресурсы

- [Документация HTML Semantics на MDN](https://developer.mozilla.org/ru/docs/Web/HTML/Element)
- [Руководство по семантическому HTML](https://web.dev/learn/html/semantic-html/)
- [HTML5 Semantics от W3C](https://www.w3.org/TR/html52/dom.html#kinds-of-content)

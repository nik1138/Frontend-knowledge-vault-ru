# Рекурсивные типы в TypeScript

Рекурсивные типы (recursive types) - это типы, которые ссылаются на самих себя в своей декларации. Они особенно полезны для представления древовидных структур данных, вложенных объектов, списков и других структур, которые могут иметь произвольную глубину вложенности.

## Основные примеры рекурсивных типов

### 1. Простое бинарное дерево

```typescript
interface TreeNode<T> {
  value: T;
  left?: TreeNode<T>;
  right?: TreeNode<T>;
}

// Создание дерева
const tree: TreeNode<number> = {
  value: 1,
  left: {
    value: 2,
    left: { value: 4 },
    right: { value: 5 }
  },
  right: {
    value: 3,
    left: { value: 6 },
    right: { value: 7 }
  }
};
```

### 2. Связный список

```typescript
interface ListNode<T> {
  value: T;
  next?: ListNode<T>;
}

// Создание списка: 1 -> 2 -> 3 -> null
const list: ListNode<number> = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3
    }
  }
};
```

### 3. Вложенные комментарии

```typescript
interface Comment {
  id: number;
  text: string;
  author: string;
  replies?: Comment[];
}

const comment: Comment = {
  id: 1,
  text: "Это отличный пост!",
  author: "Alice",
  replies: [
    {
      id: 2,
      text: "Спасибо за комментарий!",
      author: "Bob",
      replies: [
        {
          id: 3,
          text: "Действительно полезно!",
          author: "Charlie"
          // replies: undefined (может быть пустым)
        }
      ]
    }
  ]
};
```

## Продвинутые рекурсивные структуры

### 1. N-арное дерево (дерево с произвольным количеством потомков)

```typescript
interface NTreeNode<T> {
  value: T;
  children: NTreeNode<T>[];
}

const nTree: NTreeNode<string> = {
  value: "root",
  children: [
    {
      value: "child1",
      children: [
        { value: "grandchild1", children: [] },
        { value: "grandchild2", children: [] }
      ]
    },
    {
      value: "child2",
      children: [
        { value: "grandchild3", children: [] }
      ]
    }
  ]
};
```

### 2. Вложенные категории

```typescript
interface Category {
  id: number;
  name: string;
  parent?: Category;
  children: Category[];
  products: Product[];
}

interface Product {
  id: number;
  name: string;
  price: number;
  category: Category;
}

// Пример вложенной структуры
const electronics: Category = {
  id: 1,
  name: "Электроника",
  children: [
    {
      id: 2,
      name: "Телефоны",
      children: [
        {
          id: 3,
          name: "Смартфоны",
          children: [],
          products: [
            { id: 101, name: "iPhone", price: 999, category: null as any }
          ]
        }
      ],
      products: []
    }
  ],
  products: []
};

// Исправим ссылки на категории в продуктах
electronics.children[0].children[0].products[0].category = electronics.children[0].children[0];
```

### 3. Рекурсивные типы с условными выражениями

```typescript
// Тип для представления вложенных настроек
type NestedConfig<T> = {
  [K in keyof T]: T[K] extends object 
    ? T[K] extends Function 
      ? T[K] 
      : NestedConfig<T[K]> 
    : T[K]
};

interface AppSettings {
  server: {
    host: string;
    port: number;
    ssl: {
      enabled: boolean;
      certPath: string;
    };
  };
  database: {
    url: string;
    options: {
      timeout: number;
      retries: number;
    };
  };
  features: {
    cache: boolean;
    logging: {
      level: "debug" | "info" | "warn" | "error";
      format: "json" | "text";
    };
  };
}

type NestedAppSettings = NestedConfig<AppSettings>;
// Тип будет рекурсивно применяться ко всем вложенным объектам
```

## Практические применения рекурсивных типов

### 1. Работа с JSON-подобными структурами

```typescript
// Тип для представления любого JSON-объекта
type JsonValue = 
  | string 
  | number 
  | boolean 
  | null 
  | JsonObject 
  | JsonArray;

interface JsonObject {
  [key: string]: JsonValue;
}

interface JsonArray extends Array<JsonValue> {}

// Пример использования
const jsonData: JsonValue = {
  name: "John",
  age: 30,
  address: {
    street: "123 Main St",
    city: "New York",
    coordinates: [40.7128, -74.0060]
  },
  hobbies: ["reading", "swimming", null],
  isActive: true
};
```

### 2. Система прав доступа

```typescript
interface Permission {
  name: string;
  description: string;
  children?: Permission[];
}

const rootPermission: Permission = {
  name: "root",
  description: "Полный доступ",
  children: [
    {
      name: "user_management",
      description: "Управление пользователями",
      children: [
        { name: "create_user", description: "Создание пользователей" },
        { name: "delete_user", description: "Удаление пользователей" }
      ]
    },
    {
      name: "content_management",
      description: "Управление контентом",
      children: [
        { name: "create_post", description: "Создание постов" },
        { name: "delete_post", description: "Удаление постов" }
      ]
    }
  ]
};
```

### 3. Репрезентация файловой системы

```typescript
interface FileSystemNode {
  name: string;
  type: 'file' | 'directory';
  size?: number; // для файлов
  children?: FileSystemNode[]; // для директорий
}

const fileSystem: FileSystemNode = {
  name: "root",
  type: "directory",
  children: [
    {
      name: "documents",
      type: "directory",
      children: [
        {
          name: "report.pdf",
          type: "file",
          size: 1024000
        },
        {
          name: "presentation.pptx",
          type: "file",
          size: 2048000
        }
      ]
    },
    {
      name: "images",
      type: "directory",
      children: [
        {
          name: "photo.jpg",
          type: "file",
          size: 512000
        }
      ]
    }
  ]
};
```

## Рекурсивные типы с обобщениями

### 1. Универсальное дерево

```typescript
interface GenericTree<T> {
  data: T;
  children: GenericTree<T>[];
  parent?: GenericTree<T>;
}

// Использование
const stringTree: GenericTree<string> = {
  data: "root",
  children: [
    { data: "child1", children: [] },
    { data: "child2", children: [] }
  ]
};

const numberTree: GenericTree<number> = {
  data: 1,
  children: [
    { data: 2, children: [] },
    { data: 3, children: [] }
  ]
};
```

### 2. Рекурсивные типы с ограничениями

```typescript
interface ValidatedTree<T extends { isValid: boolean }> {
  node: T;
  children: ValidatedTree<T>[];
}

interface ValidatedData {
  id: number;
  name: string;
  isValid: boolean;
}

const validatedTree: ValidatedTree<ValidatedData> = {
  node: { id: 1, name: "root", isValid: true },
  children: [
    { 
      node: { id: 2, name: "child", isValid: false }, 
      children: [] 
    }
  ]
};
```

## Работа с рекурсивными типами

### 1. Рекурсивные функции для работы с деревьями

```typescript
// Функция для подсчета узлов в дереве
function countNodes<T>(tree: TreeNode<T>): number {
  let count = 1; // текущий узел
  
  if (tree.left) {
    count += countNodes(tree.left);
  }
  
  if (tree.right) {
    count += countNodes(tree.right);
  }
  
  return count;
}

// Функция для поиска значения в дереве
function findInTree<T>(tree: TreeNode<T>, value: T): boolean {
  if (tree.value === value) {
    return true;
  }
  
  if (tree.left && findInTree(tree.left, value)) {
    return true;
  }
  
  if (tree.right && findInTree(tree.right, value)) {
    return true;
  }
  
  return false;
}

// Функция для обхода дерева (in-order traversal)
function traverseTree<T>(tree: TreeNode<T>, callback: (value: T) => void): void {
  if (tree.left) {
    traverseTree(tree.left, callback);
  }
  
  callback(tree.value);
  
  if (tree.right) {
    traverseTree(tree.right, callback);
  }
}
```

### 2. Рекурсивные функции для вложенных структур

```typescript
// Функция для плоского представления вложенной структуры
function flattenTree<T>(node: TreeNode<T>): T[] {
  const result: T[] = [node.value];
  
  if (node.left) {
    result.push(...flattenTree(node.left));
  }
  
  if (node.right) {
    result.push(...flattenTree(node.right));
  }
  
  return result;
}

// Функция для поиска в комментариях
function findComment(comments: Comment[], id: number): Comment | undefined {
  for (const comment of comments) {
    if (comment.id === id) {
      return comment;
    }
    
    if (comment.replies) {
      const found = findComment(comment.replies, id);
      if (found) {
        return found;
      }
    }
  }
  
  return undefined;
}
```

## Ограничения и особенности рекурсивных типов

### 1. Ограничения компилятора

```typescript
// Иногда TypeScript может жаловаться на слишком глубокую рекурсию
// type DeepNest<T> = { value: T } | { value: DeepNest<T> }; // Может вызвать ошибку

// Вместо этого используйте более конкретные структуры или ограничьте глубину
type LimitedDepth<T, Depth extends number = 5> = 
  Depth extends 0 
    ? T 
    : { value: T } | { value: LimitedDepth<T, Prev[Depth]> };

// Вспомогательный тип для уменьшения числа
type Prev = [never, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

// Это позволяет ограничить глубину рекурсии
type ShallowNest<T> = LimitedDepth<T, 3>;
```

### 2. Проблемы с производительностью

```typescript
// Очень глубокие рекурсивные типы могут замедлить компиляцию
// interface VeryDeepType {
//   level1: { level2: { level3: { ... { level100: string } ... } } };
// }

// Вместо этого используйте более простые структуры или альтернативные подходы
```

## Практические примеры из реальной разработки

### 1. Работа с конфигурацией приложения

```typescript
interface ConfigNode {
  value?: any;
  children?: { [key: string]: ConfigNode };
  description?: string;
  defaultValue?: any;
}

const appConfig: ConfigNode = {
  children: {
    server: {
      children: {
        port: {
          value: 3000,
          description: "Порт сервера",
          defaultValue: 3000
        },
        host: {
          value: "localhost",
          description: "Хост сервера",
          defaultValue: "localhost"
        }
      }
    },
    database: {
      children: {
        url: {
          value: "mongodb://localhost:27017/myapp",
          description: "URL базы данных",
          defaultValue: "mongodb://localhost:27017/myapp"
        }
      }
    }
  }
};
```

### 2. Система шаблонов с вложенными компонентами

```typescript
interface ComponentNode {
  type: string;
  props: { [key: string]: any };
  children: (ComponentNode | string)[];
}

const componentTree: ComponentNode = {
  type: "div",
  props: { className: "container" },
  children: [
    "Hello, ",
    {
      type: "span",
      props: { className: "highlight" },
      children: ["World"]
    },
    "!"
  ]
};
```

### 3. Работа с вложенными формами

```typescript
interface FormField {
  name: string;
  type: "text" | "number" | "checkbox" | "group";
  value?: any;
  children?: FormField[];
  validators?: ((value: any) => boolean)[];
}

const userForm: FormField = {
  name: "user",
  type: "group",
  children: [
    {
      name: "personalInfo",
      type: "group",
      children: [
        { name: "firstName", type: "text" },
        { name: "lastName", type: "text" },
        { name: "age", type: "number" }
      ]
    },
    {
      name: "preferences",
      type: "group",
      children: [
        { name: "newsletter", type: "checkbox" },
        { name: "notifications", type: "checkbox" }
      ]
    }
  ]
};
```

## Лучшие практики

1. **Ограничивайте глубину рекурсии** - чтобы избежать проблем с производительностью
2. **Используйте обобщения** - для создания универсальных рекурсивных типов
3. **Предоставляйте функции для работы с рекурсивными структурами** - обход, поиск, фильтрация
4. **Документируйте рекурсивные типы** - особенно когда они сложные
5. **Рассмотрите альтернативы** - иногда проще использовать плоские структуры с ID-ссылками
6. **Тестируйте граничные случаи** - пустые структуры, максимальная глубина и т.д.

## Связи с другими концепциями

- [[ts/advanced-types/conditional-types|Условные типы]] - часто используются в рекурсивных типах
- [[ts/generics/TS Generics|Обобщения]] - основа для универсальных рекурсивных типов
- [[ts/advanced-types/mapped-types|Сопоставленные типы]] - могут быть рекурсивными
- [[ts/types/type-aliases|Псевдонимы типов]] - способ определения рекурсивных типов
- [[js/data-structures]] - JavaScript структуры данных, которые могут быть типизированы
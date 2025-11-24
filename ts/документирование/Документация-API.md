---
aliases: [API Documentation, Документирование API]
tags: [typescript, documentation, api, rest-api]
---

# Документация API в TypeScript проектах

## Обзор

Документация API играет ключевую роль в разработке TypeScript проектов, особенно когда речь идет о создании библиотек, фреймворков или сервисов. Хорошо документированный API позволяет другим разработчикам легко понимать, использовать и расширять ваш код. В этом руководстве мы рассмотрим подходы к созданию качественной документации API для TypeScript проектов.

## Принципы хорошей документации API

### 1. Ясность и понятность

Хорошая документация API должна быть понятной как опытным, так и начинающим разработчикам:

```typescript
/**
 * Асинхронно загружает пользователей из удаленного API
 * 
 * @param filters - Опциональные фильтры для поиска пользователей
 * @param filters.role - Фильтр по роли пользователя
 * @param filters.active - Фильтр по статусу активности
 * @param pagination - Опции пагинации
 * @param pagination.page - Номер страницы (по умолчанию 1)
 * @param pagination.limit - Количество элементов на странице (по умолчанию 10)
 * 
 * @returns Promise, разрешающийся массивом пользователей
 * 
 * @throws {NetworkError} Если произошла ошибка сети
 * @throws {AuthenticationError} Если токен аутентификации недействителен
 * 
 * @example
 * ```ts
 * // Загрузка всех активных администраторов
 * const admins = await fetchUsers({ 
 *   filters: { role: 'admin', active: true },
 *   pagination: { page: 1, limit: 20 }
 * });
 * 
 * console.log(admins); // [{ id: 1, name: 'John', role: 'admin', active: true }, ...]
 * ```
 */
async function fetchUsers(
    filters?: { role?: string; active?: boolean },
    pagination?: { page?: number; limit?: number }
): Promise<User[]> {
    // реализация
}
```

### 2. Полное покрытие API

Каждый публичный элемент API должен быть задокументирован:

```typescript
/**
 * Менеджер сессий пользователей
 * Обеспечивает создание, проверку и завершение сессий
 */
export class SessionManager {
    /**
     * Создает новую сессию для пользователя
     * @param userId - ID пользователя
     * @param deviceInfo - Информация об устройстве (опционально)
     * @returns Объект сессии с токеном и временем истечения
     * @throws {UserNotFoundError} Если пользователь не найден
     * @throws {RateLimitError} Если превышен лимит попыток создания сессии
     */
    async createSession(
        userId: string, 
        deviceInfo?: DeviceInfo
    ): Promise<Session> {
        // реализация
    }

    /**
     * Проверяет валидность токена сессии
     * @param token - Токен сессии для проверки
     * @returns Информация о сессии или null, если токен недействителен
     */
    async validateSession(token: string): Promise<SessionInfo | null> {
        // реализация
    }

    /**
     * Завершает сессию
     * @param token - Токен сессии для завершения
     * @returns true, если сессия была успешно завершена
     */
    async endSession(token: string): Promise<boolean> {
        // реализация
    }
}
```

## Структура документации API

### 1. Заголовок и краткое описание

Каждый элемент API должен начинаться с краткого описания:

```typescript
/**
 * Вычисляет факториал числа с использованием рекурсии
 */
function factorial(n: number): number {
    // реализация
}
```

### 2. Подробное описание

Для сложных элементов добавляйте подробное описание:

```typescript
/**
 * Управляет очередью задач для асинхронной обработки
 * 
 * Очередь задач обеспечивает:
 * - Последовательное выполнение задач
 * - Обработку ошибок с возможностью повтора
 * - Ограничение количества одновременно выполняемых задач
 * - Отслеживание прогресса выполнения
 * 
 * @remarks
 * Все задачи в очереди выполняются в порядке поступления.
 * Если задача завершается с ошибкой, она может быть повторно 
 * поставлена в очередь в зависимости от настроек повтора.
 */
export class TaskQueue {
    // реализация
}
```

### 3. Параметры и возвращаемые значения

Документируйте все параметры и возвращаемые значения:

```typescript
/**
 * Фильтрует массив объектов по заданным критериям
 * 
 * @typeParam T - Тип элементов массива
 * @param array - Массив для фильтрации
 * @param predicate - Функция-предикат для проверки элементов
 * @param options - Дополнительные опции фильтрации
 * @param options.caseSensitive - Учитывать регистр при строковом сравнении (по умолчанию true)
 * @param options.deep - Выполнять глубокое сравнение объектов (по умолчанию false)
 * 
 * @returns Новый массив, содержащий элементы, удовлетворяющие предикату
 * 
 * @example
 * ```ts
 * const users = [
 *   { name: 'John', age: 30 },
 *   { name: 'Jane', age: 25 }
 * ];
 * 
 * const adults = filterByCriteria(
 *   users, 
 *   user => user.age >= 30,
 *   { caseSensitive: false }
 * );
 * // Результат: [{ name: 'John', age: 30 }]
 * ```
 */
function filterByCriteria<T>(
    array: T[],
    predicate: (item: T) => boolean,
    options?: {
        caseSensitive?: boolean;
        deep?: boolean;
    }
): T[] {
    // реализация
}
```

## Документирование различных элементов API

### Интерфейсов и типов

```typescript
/**
 * Представляет информацию о пользователе
 * Используется в системе аутентификации и авторизации
 */
export interface User {
    /**
     * Уникальный идентификатор пользователя
     * Генерируется автоматически при регистрации
     */
    id: string;
    
    /**
     * Имя пользователя
     * Должно быть уникальным в системе
     */
    name: string;
    
    /**
     * Электронная почта пользователя
     * Используется для аутентификации
     */
    email: string;
    
    /**
     * Роль пользователя в системе
     * Определяет уровень доступа к функциям
     */
    role: 'admin' | 'moderator' | 'user';
    
    /**
     * Дата регистрации пользователя
     * Формат: ISO 8601
     */
    createdAt: Date;
    
    /**
     * Статус активности пользователя
     * Неактивные пользователи не могут входить в систему
     */
    isActive: boolean;
}

/**
 * Конфигурационные опции для подключения к базе данных
 * @alpha - Этот тип экспериментальный и может измениться
 */
export type DatabaseConfig = {
    /**
     * Адрес сервера базы данных
     */
    host: string;
    
    /**
     * Номер порта для подключения
     * @defaultValue 5432
     */
    port: number;
    
    /**
     * Имя базы данных
     */
    database: string;
    
    /**
     * Имя пользователя для аутентификации
     */
    username: string;
    
    /**
     * Пароль для аутентификации
     * @privateRemarks
     * Пароль должен соответствовать требованиям безопасности:
     * - не менее 8 символов
     * - содержать заглавные и строчные буквы
     * - содержать цифры
     */
    password: string;
};
```

### Классов и методов

```typescript
/**
 * Клиент для взаимодействия с REST API
 * 
 * @example
 * ```ts
 * const client = new ApiClient({
 *   baseUrl: 'https://api.example.com',
 *   apiKey: 'your-api-key'
 * });
 * 
 * const user = await client.getUser('123');
 * ```
 * 
 * @public
 */
export class ApiClient {
    private readonly config: ApiConfig;
    
    /**
     * @param config - Конфигурация клиента
     */
    constructor(config: ApiConfig) {
        this.config = config;
    }
    
    /**
     * Получает информацию о пользователе по ID
     * 
     * @param userId - Уникальный идентификатор пользователя
     * @param options - Дополнительные опции запроса
     * @returns Promise, разрешающийся объектом пользователя
     * 
     * @throws {NotFoundError} Если пользователь с указанным ID не найден
     * @throws {UnauthorizedError} Если токен доступа недействителен
     * @throws {NetworkError} Если произошла сетевая ошибка
     * 
     * @remarks
     * Для выполнения этого метода требуются права 'read:users'
     */
    async getUser(
        userId: string, 
        options?: RequestOptions
    ): Promise<User> {
        // реализация
    }
    
    /**
     * Создает нового пользователя
     * 
     * @param userData - Данные нового пользователя
     * @returns Promise, разрешающийся созданным пользователем
     * 
     * @throws {ValidationError} Если переданные данные некорректны
     * @throws {ConflictError} Если пользователь с таким email уже существует
     * 
     * @example
     * ```ts
     * const newUser = await client.createUser({
     *   name: 'John Doe',
     *   email: 'john@example.com',
     *   role: 'user'
     * });
     * ```
     */
    async createUser(userData: CreateUserRequest): Promise<User> {
        // реализация
    }
}
```

### Функций и хелперов

```typescript
/**
 * Преобразует строку в безопасный URL slug
 * 
 * @param text - Входная строка для преобразования
 * @returns Строка, подходящая для использования в URL
 * 
 * @example
 * ```ts
 * const slug = toSlug('Hello World!'); // 'hello-world'
 * const slug2 = toSlug('Привет, мир!'); // 'privet-mir'
 * ```
 * 
 * @remarks
 * Функция:
 * - Приводит к нижнему регистру
 * - Заменяет пробелы и специальные символы на дефисы
 * - Удаляет специальные символы
 * - Обрезает лишние дефисы в начале и конце
 */
function toSlug(text: string): string {
    return text
        .toLowerCase()
        .replace(/[^\w\s-]/g, '')  // Удаляем специальные символы
        .trim()                    // Убираем пробелы по краям
        .replace(/[\s_-]+/g, '-'); // Заменяем пробелы и подчеркивания на дефисы
}

/**
 * Асинхронно откладывает выполнение на заданное количество миллисекунд
 * 
 * @param ms - Количество миллисекунд для задержки
 * @returns Promise, который разрешается после заданной задержки
 * 
 * @example
 * ```ts
 * console.log('Начало');
 * await delay(1000); // Пауза на 1 секунду
 * console.log('Конец');
 * ```
 */
function delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
}
```

## Примеры использования

### Простые примеры

```typescript
/**
 * Складывает два числа
 * @param a - Первое слагаемое
 * @param b - Второе слагаемое
 * @returns Сумма a и b
 * 
 * @example
 * ```ts
 * add(2, 3); // 5
 * add(-1, 5); // 4
 * ```
 */
function add(a: number, b: number): number {
    return a + b;
}
```

### Сложные примеры с контекстом

```typescript
/**
 * Создает цепочку middleware для обработки HTTP запросов
 * 
 * @param middlewares - Массив middleware функций
 * @returns Функция, которая принимает обработчик запроса и возвращает
 *          новый обработчик с примененными middleware
 * 
 * @example
 * ```ts
 * // Определение middleware
 * const authMiddleware: Middleware = (req, res, next) => {
 *   if (!req.headers.authorization) {
 *     res.status(401).send('Unauthorized');
 *     return;
 *   }
 *   next();
 * };
 * 
 * const loggerMiddleware: Middleware = (req, res, next) => {
 *   console.log(`${req.method} ${req.url}`);
 *   next();
 * };
 * 
 * // Создание цепочки middleware
 * const applyMiddleware = createMiddlewareChain([
 *   loggerMiddleware,
 *   authMiddleware
 * ]);
 * 
 * // Использование в маршруте
 * app.get('/api/data', applyMiddleware((req, res) => {
 *   res.json({ message: 'Protected data' });
 * }));
 * ```
 */
function createMiddlewareChain(middlewares: Middleware[]): MiddlewareChain {
    // реализация
}
```

## Стандарты и шаблоны документации

### Шаблон для методов класса

```typescript
/**
 * [Краткое описание действия]
 * 
 * [Подробное описание, если необходимо]
 * 
 * @param paramName - [Описание параметра]
 * @param options - [Описание объекта опций, если есть]
 * @param options.optionName - [Описание опции]
 * @returns [Описание возвращаемого значения]
 * 
 * @throws {ErrorType} [Условия выброса ошибки]
 * 
 * @example
 * ```ts
 * // Пример использования
 * ```
 * 
 * @remarks
 * [Дополнительная информация, которая полезна разработчику]
 */
```

### Шаблон для интерфейсов

```typescript
/**
 * [Описание назначения интерфейса]
 * 
 * @example
 * ```ts
 * // Пример использования интерфейса
 * ```
 * 
 * @public
 */
export interface InterfaceName {
    // свойства интерфейса
}
```

## Инструменты для генерации документации

### TypeDoc

TypeDoc - популярный инструмент для генерации документации из TypeScript кода:

```bash
npm install -g typedoc
typedoc src/ --out docs/api --name "My TypeScript Project"
```

### Настройка TypeDoc

```json
{
  "entryPoints": ["src/index.ts"],
  "out": "docs/api",
  "name": "My TypeScript Project",
  "readme": "README.md",
  "theme": "default",
  "excludeNotExported": true,
  "includeDeclarations": true
}
```

## Лучшие практики

1. **Документируйте только публичный API**: Не документируйте внутренние (private/internal) элементы, если только это не требуется для понимания архитектуры.

2. **Используйте примеры**: Всегда включайте примеры использования в документацию.

3. **Поддерживайте актуальность**: Регулярно обновляйте документацию при изменении API.

4. **Будьте последовательны**: Используйте единый стиль документирования во всем проекте.

5. **Используйте теги TSDoc**: Применяйте стандартные теги для лучшей интеграции с инструментами.

6. **Пишите для пользователей API**: Документация должна быть полезной для разработчиков, использующих ваш API.

## Связанные темы

- [[JSDoc]] - Классический подход к документированию JavaScript
- [[TSDoc]] - Современный стандарт документирования TypeScript
- [[Комментарии-в-коде]] - Принципы написания качественных комментариев
- [[Документирование-библиотек]] - Рекомендации по документированию библиотек
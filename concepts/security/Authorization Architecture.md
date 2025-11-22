---
aliases: [Архитектура авторизации, Контроль доступа]
tags: [security, architecture, authorization, access-control]
---

# Архитектура авторизации в программных системах

## Обзор

Архитектура авторизации представляет собой комплекс стратегий, паттернов и компонентов, предназначенных для контроля доступа к ресурсам и операциям в программных системах. В отличие от аутентификации, которая проверяет подлинность субъекта, авторизация определяет, какие действия может выполнять аутентифицированный субъект в системе.

## Паттерны архитектуры авторизации

### Role-Based Access Control (RBAC)

Система контроля доступа на основе ролей - один из самых распространенных подходов:

- **Преимущества**: Простота понимания и управления, хорошая масштабируемость
- **Компоненты**: Пользователи, роли, разрешения, отношения между ними
- **Применение**: Веб-приложения, корпоративные системы, системы управления контентом

Основные компоненты RBAC:

- **Users (Пользователи)**: Субъекты системы, которым назначаются роли
- **Roles (Роли)**: Наборы разрешений, ассоциированные с определенными функциями
- **Permissions (Разрешения)**: Конкретные действия, которые можно выполнять
- **Assignments (Назначения)**: Отношения между пользователями и ролями

```javascript
// Пример реализации RBAC
class RBACManager {
    constructor() {
        this.roles = new Map(); // role -> permissions
        this.userRoles = new Map(); // user -> roles
        this.roleHierarchy = new Map(); // role -> parent roles
    }
    
    addRole(roleName, permissions = []) {
        this.roles.set(roleName, new Set(permissions));
    }
    
    assignRoleToUser(userId, roleName) {
        if (!this.userRoles.has(userId)) {
            this.userRoles.set(userId, new Set());
        }
        this.userRoles.get(userId).add(roleName);
    }
    
    checkPermission(userId, permission) {
        const userRoles = this.userRoles.get(userId);
        if (!userRoles) return false;
        
        for (const role of userRoles) {
            const rolePermissions = this.roles.get(role);
            if (rolePermissions && rolePermissions.has(permission)) {
                return true;
            }
            
            // Проверка иерархии ролей
            if (this.checkRoleHierarchy(role, permission)) {
                return true;
            }
        }
        
        return false;
    }
    
    checkRoleHierarchy(role, permission) {
        const parentRoles = this.roleHierarchy.get(role) || [];
        for (const parentRole of parentRoles) {
            const parentPermissions = this.roles.get(parentRole);
            if (parentPermissions && parentPermissions.has(permission)) {
                return true;
            }
            // Рекурсивная проверка иерархии
            if (this.checkRoleHierarchy(parentRole, permission)) {
                return true;
            }
        }
        return false;
    }
}
```

### Claims-Based Authorization

Система авторизации на основе утверждений (claims):

- **Преимущества**: Гибкость, возможность делегирования, поддержка федерации
- **Компоненты**: Утверждения (claims), токены, системы выдачи утверждений
- **Применение**: Веб-приложения, API, распределенные системы

Пример структуры claims:
```javascript
const userClaims = {
    userId: "12345",
    email: "user@example.com",
    role: "admin",
    department: "engineering",
    clearanceLevel: "confidential",
    permissions: ["read", "write", "delete"],
    organization: "acme-corp"
};
```

### Attribute-Based Access Control (ABAC)

Система контроля доступа на основе атрибутов:

- **Преимущества**: Высокая гибкость, поддержка сложных политик
- **Компоненты**: Атрибуты субъектов, ресурсов, окружающей среды
- **Язык политик**: XACML, собственные DSL

Пример политики ABAC:
```
Доступ разрешен, если:
- Роль пользователя = "manager" И
- Отдел пользователя = Отдел ресурса И
- Время запроса между 9:00 и 18:00 И
- Уровень безопасности пользователя >= Уровень безопасности ресурса
```

### Policy-Based Authorization

Система на основе политик с централизованным управлением:

- **Преимущества**: Гибкость, централизованное управление, поддержка сложных сценариев
- **Компоненты**: Политики, движок оценки, репозиторий политик
- **Язык политик**: Регулярные выражения, DSL, JSON

## Стратегии реализации

### Централизованная архитектура

Единая точка принятия решений об авторизации:

- **Authorization Service**: Централизованный сервис для проверки доступа
- **Policy Decision Point (PDP)**: Компонент, принимающий решения об авторизации
- **Policy Enforcement Point (PEP)**: Компонент, реализующий решения об авторизации

Преимущества:
- Единая точка управления политиками
- Централизованное логирование и аудит
- Легкость обновления политик

Недостатки:
- Потенциальная точка отказа
- Возможные задержки при проверке доступа
- Сложность масштабирования

```javascript
// Пример централизованного сервиса авторизации
class CentralizedAuthService {
    constructor(policyRepository, cache) {
        this.policyRepository = policyRepository;
        this.cache = cache;
        this.cacheTimeout = 300000; // 5 minutes
    }
    
    async checkAccess(userId, resource, action) {
        // Проверяем кэш
        const cacheKey = `${userId}:${resource}:${action}`;
        const cachedResult = await this.cache.get(cacheKey);
        
        if (cachedResult !== null) {
            return cachedResult;
        }
        
        // Получаем профиль пользователя
        const userProfile = await this.getUserProfile(userId);
        
        // Загружаем политики для ресурса
        const policies = await this.policyRepository.getPoliciesForResource(resource);
        
        // Оцениваем политики
        const decision = this.evaluatePolicies(userProfile, policies, action);
        
        // Кэшируем результат
        await this.cache.set(cacheKey, decision, this.cacheTimeout);
        
        return decision;
    }
    
    evaluatePolicies(userProfile, policies, action) {
        for (const policy of policies) {
            if (this.matchesPolicy(userProfile, policy, action)) {
                return policy.effect === 'allow';
            }
        }
        return false; // По умолчанию запрещено
    }
    
    matchesPolicy(userProfile, policy, action) {
        // Проверяем соответствие политики
        return policy.actions.includes(action) && 
               this.evaluateConditions(userProfile, policy.conditions);
    }
    
    evaluateConditions(userProfile, conditions) {
        // Оценка условий политики
        for (const condition of conditions) {
            if (!this.evaluateCondition(userProfile, condition)) {
                return false;
            }
        }
        return true;
    }
}
```

### Децентрализованная архитектура

Распределенная модель с локальной обработкой запросов:

- **Local Policy Evaluation**: Каждый сервис принимает собственные решения
- **Policy Distribution**: Распределение политик по системе
- **Eventual Consistency**: Согласованность политик со временем

Преимущества:
- Высокая доступность
- Низкая задержка
- Масштабируемость

Недостатки:
- Сложность синхронизации политик
- Потенциальные проблемы с консистентностью
- Сложность аудита

### Гибридная архитектура

Сочетание централизованных и децентрализованных подходов:

- **Critical Operations**: Централизованная проверка для критических операций
- **Routine Operations**: Локальная проверка для обычных операций
- **Fallback Mechanisms**: Резервные механизмы при недоступности центрального сервиса

## Ролевая модель

### Иерархия ролей

Организация ролей в иерархическую структуру:

- **Base Roles**: Базовые роли с минимальными правами
- **Intermediate Roles**: Роли с расширенными правами
- **Admin Roles**: Роли с максимальными правами

Пример иерархии:
```
        System Admin
            |
        Department Admin
            |
        Team Lead
            |
        Senior Developer
            |
        Developer
            |
        Guest
```

### Динамические роли

Роли, которые могут изменяться в зависимости от контекста:

- **Context-Aware Roles**: Роли, зависящие от текущего контекста
- **Time-Based Roles**: Роли, действующие в определенное время
- **Resource-Based Roles**: Роли, зависящие от ресурса

```javascript
// Пример динамической ролевой модели
class DynamicRoleManager {
    constructor() {
        this.roleProviders = new Map();
    }
    
    registerRoleProvider(name, provider) {
        this.roleProviders.set(name, provider);
    }
    
    async getUserRoles(userId, context = {}) {
        const allRoles = new Set();
        
        for (const [name, provider] of this.roleProviders) {
            const roles = await provider.getUserRoles(userId, context);
            for (const role of roles) {
                allRoles.add(role);
            }
        }
        
        return Array.from(allRoles);
    }
    
    async checkDynamicPermission(userId, resource, action, context = {}) {
        const userRoles = await this.getUserRoles(userId, context);
        
        for (const role of userRoles) {
            if (await this.checkRolePermission(role, resource, action, context)) {
                return true;
            }
        }
        
        return false;
    }
    
    async checkRolePermission(role, resource, action, context) {
        // Проверка разрешений для роли с учетом контекста
        const permissions = await this.getRolePermissions(role);
        
        for (const permission of permissions) {
            if (this.matchesPermission(permission, resource, action, context)) {
                return true;
            }
        }
        
        return false;
    }
}
```

## Реализация систем авторизации

### Middleware-подход

Интеграция авторизации через middleware:

```javascript
// Пример авторизационного middleware
class AuthzMiddleware {
    constructor(authzService) {
        this.authzService = authzService;
    }
    
    requirePermission(permission) {
        return async (req, res, next) => {
            if (!req.user) {
                return res.status(401).json({ error: 'Authentication required' });
            }
            
            const hasPermission = await this.authzService.checkPermission(
                req.user.id, 
                req.route.path, 
                req.method,
                permission
            );
            
            if (!hasPermission) {
                return res.status(403).json({ error: 'Insufficient permissions' });
            }
            
            next();
        };
    }
    
    requireRole(role) {
        return async (req, res, next) => {
            if (!req.user) {
                return res.status(401).json({ error: 'Authentication required' });
            }
            
            const hasRole = await this.authzService.checkRole(req.user.id, role);
            
            if (!hasRole) {
                return res.status(403).json({ error: 'Role required' });
            }
            
            next();
        };
    }
    
    requireOwnership(resourceIdParam) {
        return async (req, res, next) => {
            if (!req.user) {
                return res.status(401).json({ error: 'Authentication required' });
            }
            
            const resourceId = req.params[resourceIdParam];
            const isOwner = await this.authzService.checkOwnership(
                req.user.id, 
                resourceId
            );
            
            if (!isOwner) {
                return res.status(403).json({ error: 'Resource ownership required' });
            }
            
            next();
        };
    }
}
```

### Атрибутно-ориентированный подход

Использование атрибутов для принятия решений:

```javascript
// Пример атрибутно-ориентированной системы
class AttributeBasedAuthz {
    constructor() {
        this.attributeProviders = new Map();
        this.policies = [];
    }
    
    registerAttributeProvider(name, provider) {
        this.attributeProviders.set(name, provider);
    }
    
    addPolicy(policy) {
        this.policies.push(policy);
    }
    
    async evaluateAccess(subject, resource, action) {
        // Собираем атрибуты субъекта
        const subjectAttributes = await this.getSubjectAttributes(subject);
        
        // Собираем атрибуты ресурса
        const resourceAttributes = await this.getResourceAttributes(resource);
        
        // Собираем атрибуты окружающей среды
        const environmentAttributes = await this.getEnvironmentAttributes();
        
        // Оцениваем политики
        for (const policy of this.policies) {
            if (this.evaluatePolicy(
                policy, 
                subjectAttributes, 
                resourceAttributes, 
                environmentAttributes, 
                action
            )) {
                return policy.effect === 'Permit';
            }
        }
        
        return false; // По умолчанию запрещено
    }
    
    evaluatePolicy(policy, subjectAttrs, resourceAttrs, envAttrs, action) {
        return this.evaluateCondition(
            policy.condition, 
            subjectAttrs, 
            resourceAttrs, 
            envAttrs, 
            action
        );
    }
    
    evaluateCondition(condition, subjectAttrs, resourceAttrs, envAttrs, action) {
        switch (condition.type) {
            case 'attribute-match':
                return this.evaluateAttributeMatch(
                    condition, 
                    subjectAttrs, 
                    resourceAttrs, 
                    envAttrs
                );
            case 'time-restriction':
                return this.evaluateTimeRestriction(condition, envAttrs);
            case 'complex':
                return this.evaluateComplexCondition(
                    condition, 
                    subjectAttrs, 
                    resourceAttrs, 
                    envAttrs, 
                    action
                );
            default:
                return false;
        }
    }
    
    async getSubjectAttributes(subject) {
        const attributes = {};
        
        for (const [name, provider] of this.attributeProviders) {
            const attrs = await provider.getSubjectAttributes(subject);
            Object.assign(attributes, attrs);
        }
        
        return attributes;
    }
    
    async getResourceAttributes(resource) {
        const attributes = {};
        
        for (const [name, provider] of this.attributeProviders) {
            const attrs = await provider.getResourceAttributes(resource);
            Object.assign(attributes, attrs);
        }
        
        return attributes;
    }
    
    async getEnvironmentAttributes() {
        return {
            currentTime: new Date(),
            clientIP: '', // будет заполнено из запроса
            userAgent: '' // будет заполнено из запроса
        };
    }
}
```

## Рассмотрение безопасности

### Защита от атак на уровень авторизации

#### Vertical Privilege Escalation

Защита от повышения привилегий:

- **Role Verification**: Проверка ролей для каждого запроса
- **Permission Validation**: Проверка разрешений на уровне данных
- **Input Sanitization**: Санитизация входных данных

#### Horizontal Privilege Escalation

Защита от доступа к чужим данным:

- **Ownership Checks**: Проверка принадлежности ресурсов
- **Row-Level Security**: Защита на уровне записей в базе данных
- **Parameter Validation**: Проверка параметров запроса

```javascript
// Пример защиты от горизонтального повышения привилегий
class ResourceAccessValidator {
    constructor(database) {
        this.db = database;
    }
    
    async validateUserResourceAccess(userId, resourceId, resourceType) {
        // Проверяем, что пользователь имеет доступ к ресурсу
        const resource = await this.db.findOne({
            _id: resourceId,
            type: resourceType
        });
        
        if (!resource) {
            throw new Error('Resource not found');
        }
        
        // Проверяем права доступа
        if (resource.ownerId !== userId) {
            // Может быть доступ через разрешения
            const hasPermission = await this.checkUserPermission(
                userId, 
                resourceId, 
                'read'
            );
            
            if (!hasPermission) {
                throw new Error('Access denied');
            }
        }
        
        return resource;
    }
    
    async checkUserPermission(userId, resourceId, action) {
        // Проверяем разрешения через систему авторизации
        const user = await this.db.findById('users', userId);
        const resource = await this.db.findById('resources', resourceId);
        
        // Проверяем через RBAC
        if (this.rbacCheck(user, resource, action)) {
            return true;
        }
        
        // Проверяем через ACL
        if (await this.aclCheck(user, resource, action)) {
            return true;
        }
        
        // Проверяем через ABAC
        if (this.abacCheck(user, resource, action)) {
            return true;
        }
        
        return false;
    }
}
```

### Системы управления политиками

#### Централизованные политики

Хранение и управление политиками в централизованной системе:

- **Policy Repository**: Централизованное хранилище политик
- **Policy Administration Point (PAP)**: Интерфейс для управления политиками
- **Policy Information Point (PIP)**: Источник атрибутов для принятия решений

#### Динамическое обновление политик

Возможность обновления политик без перезапуска системы:

```javascript
// Пример динамического обновления политик
class DynamicPolicyManager {
    constructor(policyStore) {
        this.policyStore = policyStore;
        this.policyCache = new Map();
        this.policyListeners = [];
        this.startPolicyWatcher();
    }
    
    async updatePolicy(policyId, policy) {
        await this.policyStore.update(policyId, policy);
        // Инвалидируем кэш
        this.policyCache.delete(policyId);
        // Уведомляем слушателей
        this.notifyPolicyUpdate(policyId, policy);
    }
    
    async getPolicy(policyId) {
        // Проверяем кэш
        if (this.policyCache.has(policyId)) {
            return this.policyCache.get(policyId);
        }
        
        // Загружаем из хранилища
        const policy = await this.policyStore.findById(policyId);
        this.policyCache.set(policyId, policy);
        
        return policy;
    }
    
    addPolicyListener(listener) {
        this.policyListeners.push(listener);
    }
    
    notifyPolicyUpdate(policyId, policy) {
        for (const listener of this.policyListeners) {
            try {
                listener(policyId, policy);
            } catch (error) {
                console.error('Policy listener error:', error);
            }
        }
    }
    
    startPolicyWatcher() {
        // Наблюдение за изменениями политик
        setInterval(async () => {
            const changedPolicies = await this.policyStore.getChangedSince(
                this.lastCheckTime
            );
            
            for (const policy of changedPolicies) {
                this.policyCache.delete(policy.id);
                this.notifyPolicyUpdate(policy.id, policy);
            }
            
            this.lastCheckTime = new Date();
        }, 30000); // Проверка каждые 30 секунд
    }
}
```

## Практические примеры

### Пример сложной ролевой системы

```javascript
// role-based-access-control.js
class AdvancedRBAC {
    constructor() {
        this.roles = new Map();
        this.permissions = new Map();
        this.rolePermissions = new Map(); // role -> Set of permissions
        this.userRoles = new Map(); // user -> Set of roles
        this.inheritance = new Map(); // role -> Set of parent roles
    }
    
    createRole(roleName, description = '') {
        this.roles.set(roleName, { name: roleName, description });
        this.rolePermissions.set(roleName, new Set());
    }
    
    createPermission(permissionName, description = '') {
        this.permissions.set(permissionName, { 
            name: permissionName, 
            description 
        });
    }
    
    assignPermissionToRole(roleName, permissionName) {
        if (!this.rolePermissions.has(roleName)) {
            this.rolePermissions.set(roleName, new Set());
        }
        this.rolePermissions.get(roleName).add(permissionName);
    }
    
    assignRoleToUser(userId, roleName) {
        if (!this.userRoles.has(userId)) {
            this.userRoles.set(userId, new Set());
        }
        this.userRoles.get(userId).add(roleName);
    }
    
    setupRoleInheritance(childRole, parentRole) {
        if (!this.inheritance.has(childRole)) {
            this.inheritance.set(childRole, new Set());
        }
        this.inheritance.get(childRole).add(parentRole);
    }
    
    async getUserPermissions(userId) {
        const userRoles = this.userRoles.get(userId);
        if (!userRoles) return new Set();
        
        const permissions = new Set();
        
        for (const role of userRoles) {
            // Прямые разрешения роли
            const rolePerms = this.rolePermissions.get(role) || new Set();
            for (const perm of rolePerms) {
                permissions.add(perm);
            }
            
            // Разрешения через наследование
            const inheritedPerms = await this.getInheritedPermissions(role);
            for (const perm of inheritedPerms) {
                permissions.add(perm);
            }
        }
        
        return permissions;
    }
    
    async checkPermission(userId, permissionName) {
        const userPermissions = await this.getUserPermissions(userId);
        return userPermissions.has(permissionName);
    }
    
    async getInheritedPermissions(roleName, visited = new Set()) {
        if (visited.has(roleName)) return new Set(); // Предотвращение циклов
        visited.add(roleName);
        
        const permissions = new Set();
        
        // Прямые разрешения роли
        const directPerms = this.rolePermissions.get(roleName) || new Set();
        for (const perm of directPerms) {
            permissions.add(perm);
        }
        
        // Разрешения через наследование
        const parentRoles = this.inheritance.get(roleName) || new Set();
        for (const parentRole of parentRoles) {
            const inheritedPerms = await this.getInheritedPermissions(parentRole, new Set(visited));
            for (const perm of inheritedPerms) {
                permissions.add(perm);
            }
        }
        
        return permissions;
    }
}

// Использование
const rbac = new AdvancedRBAC();

// Создание ролей
rbac.createRole('user', 'Обычный пользователь');
rbac.createRole('moderator', 'Модератор');
rbac.createRole('admin', 'Администратор');

// Создание разрешений
rbac.createPermission('read:content', 'Чтение контента');
rbac.createPermission('write:content', 'Запись контента');
rbac.createPermission('delete:content', 'Удаление контента');
rbac.createPermission('moderate:content', 'Модерация контента');
rbac.createPermission('manage:users', 'Управление пользователями');

// Назначение разрешений ролям
rbac.assignPermissionToRole('user', 'read:content');
rbac.assignPermissionToRole('moderator', 'read:content');
rbac.assignPermissionToRole('moderator', 'moderate:content');
rbac.assignPermissionToRole('admin', 'read:content');
rbac.assignPermissionToRole('admin', 'write:content');
rbac.assignPermissionToRole('admin', 'delete:content');
rbac.assignPermissionToRole('admin', 'moderate:content');
rbac.assignPermissionToRole('admin', 'manage:users');

// Установка наследования
rbac.setupRoleInheritance('moderator', 'user');
rbac.setupRoleInheritance('admin', 'moderator');

// Назначение ролей пользователям
rbac.assignRoleToUser('user123', 'admin');
```

### Пример авторизации на основе атрибутов

```javascript
// attribute-based-access-control.js
class ABACAuthorization {
    constructor() {
        this.policies = [];
        this.attributeProviders = [];
    }
    
    addPolicy(policy) {
        this.policies.push(policy);
    }
    
    addAttributeProvider(provider) {
        this.attributeProviders.push(provider);
    }
    
    async evaluate(subject, resource, action, environment = {}) {
        // Собираем атрибуты
        const subjectAttrs = await this.getSubjectAttributes(subject);
        const resourceAttrs = await this.getResourceAttributes(resource);
        const envAttrs = { ...environment, timestamp: new Date() };
        
        // Оцениваем политики
        for (const policy of this.policies) {
            if (this.matchesPolicy(policy, subjectAttrs, resourceAttrs, envAttrs, action)) {
                return policy.effect === 'Permit';
            }
        }
        
        return false; // По умолчанию запрещено
    }
    
    matchesPolicy(policy, subjectAttrs, resourceAttrs, envAttrs, action) {
        // Проверяем действие
        if (!policy.actions.includes(action)) {
            return false;
        }
        
        // Проверяем условия
        return this.evaluateCondition(policy.condition, subjectAttrs, resourceAttrs, envAttrs);
    }
    
    evaluateCondition(condition, subjectAttrs, resourceAttrs, envAttrs) {
        switch (condition.type) {
            case 'and':
                return condition.expressions.every(expr => 
                    this.evaluateCondition(expr, subjectAttrs, resourceAttrs, envAttrs)
                );
            case 'or':
                return condition.expressions.some(expr => 
                    this.evaluateCondition(expr, subjectAttrs, resourceAttrs, envAttrs)
                );
            case 'not':
                return !this.evaluateCondition(condition.expression, subjectAttrs, resourceAttrs, envAttrs);
            case 'comparison':
                return this.evaluateComparison(condition, subjectAttrs, resourceAttrs, envAttrs);
            case 'time-restriction':
                return this.evaluateTimeRestriction(condition, envAttrs);
            default:
                return false;
        }
    }
    
    evaluateComparison(condition, subjectAttrs, resourceAttrs, envAttrs) {
        const { left, operator, right } = condition;
        
        let leftValue, rightValue;
        
        // Определяем левое значение
        if (left.startsWith('subject.')) {
            leftValue = subjectAttrs[left.substring(8)];
        } else if (left.startsWith('resource.')) {
            leftValue = resourceAttrs[left.substring(9)];
        } else if (left.startsWith('environment.')) {
            leftValue = envAttrs[left.substring(12)];
        } else {
            leftValue = left; // литерал
        }
        
        // Определяем правое значение
        if (right.startsWith('subject.')) {
            rightValue = subjectAttrs[right.substring(8)];
        } else if (right.startsWith('resource.')) {
            rightValue = resourceAttrs[right.substring(9)];
        } else if (right.startsWith('environment.')) {
            rightValue = envAttrs[right.substring(12)];
        } else {
            rightValue = right; // литерал
        }
        
        // Выполняем сравнение
        switch (operator) {
            case '==':
                return leftValue == rightValue;
            case '!=':
                return leftValue != rightValue;
            case '>':
                return leftValue > rightValue;
            case '<':
                return leftValue < rightValue;
            case '>=':
                return leftValue >= rightValue;
            case '<=':
                return leftValue <= rightValue;
            case 'in':
                return Array.isArray(rightValue) && rightValue.includes(leftValue);
            case 'contains':
                return typeof rightValue === 'string' && rightValue.includes(leftValue);
            default:
                return false;
        }
    }
    
    evaluateTimeRestriction(condition, envAttrs) {
        const currentTime = envAttrs.timestamp || new Date();
        const currentHour = currentTime.getHours();
        
        return currentHour >= condition.startHour && currentHour < condition.endHour;
    }
    
    async getSubjectAttributes(subject) {
        let attrs = {};
        for (const provider of this.attributeProviders) {
            const providerAttrs = await provider.getSubjectAttributes(subject);
            attrs = { ...attrs, ...providerAttrs };
        }
        return attrs;
    }
    
    async getResourceAttributes(resource) {
        let attrs = {};
        for (const provider of this.attributeProviders) {
            const providerAttrs = await provider.getResourceAttributes(resource);
            attrs = { ...attrs, ...providerAttrs };
        }
        return attrs;
    }
}

// Пример политики
const policy = {
    id: 'employee-time-sheet-access',
    description: 'Сотрудники могут просматривать свои табели в рабочее время',
    actions: ['read'],
    condition: {
        type: 'and',
        expressions: [
            {
                type: 'comparison',
                left: 'subject.role',
                operator: '==',
                right: 'employee'
            },
            {
                type: 'comparison',
                left: 'resource.type',
                operator: '==',
                right: 'timesheet'
            },
            {
                type: 'comparison',
                left: 'resource.ownerId',
                operator: '==',
                right: 'subject.id'
            },
            {
                type: 'time-restriction',
                startHour: 9,
                endHour: 18
            }
        ]
    },
    effect: 'Permit'
};

const abac = new ABACAuthorization();
abac.addPolicy(policy);
```

## Лучшие практики

### Принципы проектирования

- **Least Privilege**: Предоставлять минимально необходимые права
- **Defense in Depth**: Многоуровневая защита
- **Fail-Safe Defaults**: По умолчанию запрещать, разрешать явно
- **Separation of Duties**: Разделение критических функций между несколькими ролями

### Масштабируемость

- **Caching Strategies**: Эффективное кэширование решений об авторизации
- **Policy Distribution**: Распределение политик по системе
- **Asynchronous Updates**: Асинхронное обновление политик
- **Horizontal Scaling**: Возможность масштабирования сервисов авторизации

### Мониторинг и аудит

- **Access Logging**: Ведение логов всех попыток доступа
- **Anomaly Detection**: Обнаружение подозрительной активности
- **Compliance Reporting**: Отчеты для соответствия требованиям
- **Real-time Alerts**: Оповещения о критических событиях

### Тестирование

- **Unit Testing**: Тестирование отдельных компонентов авторизации
- **Integration Testing**: Тестирование интеграции с другими системами
- **Penetration Testing**: Тестирование на проникновение
- **Load Testing**: Тестирование под нагрузкой

## Заключение

Эффективная архитектура авторизации является критическим компонентом безопасности современных программных систем. Она должна обеспечивать:

- Гибкость для поддержки различных сценариев доступа
- Масштабируемость для роста системы
- Безопасность для защиты ресурсов
- Производительность для обеспечения хорошего пользовательского опыта

При выборе архитектуры авторизации необходимо учитывать специфику приложения, требования безопасности, ожидаемую нагрузку и сложность бизнес-логики. Регулярный аудит и обновление архитектуры помогают поддерживать актуальность и безопасность системы.

## Связанные концепции

- [[Authentication Architecture]] - Архитектура аутентификации в системах
- [[RBAC Implementation]] - Реализация ролевого контроля доступа
- [[ABAC Framework]] - Фреймворк атрибутно-ориентированного контроля
- [[Security Best Practices]] - Лучшие практики безопасности
- [[Identity Management]] - Управление идентичностью
- [[OAuth 2.0 Protocol]] - Протокол OAuth 2.0
- [[JWT Implementation]] - Реализация JWT токенов
- [[Access Control Models]] - Модели контроля доступа
- [[Policy Decision Points]] - Точки принятия решений о политике
- [[Zero Trust Architecture]] - Архитектура нулевого доверия
- [[API Security]] - Безопасность API
- [[Data Classification]] - Классификация данных
- [[Compliance Frameworks]] - Фреймворки соответствия требованиям
- [[Threat Modeling]] - Моделирование угроз
- [[Security Architecture]] - Архитектура безопасности
- [[Identity Federation]] - Федерация идентичности
- [[Multi-Tenant Security]] - Безопасность в мультитенантных системах
- [[Privacy by Design]] - Конфиденциальность по проекту
- [[Secure Coding]] - Безопасное программирование
- [[Security Testing]] - Тестирование безопасности
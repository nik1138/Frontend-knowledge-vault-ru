---
aliases: [Yet Another Resource Negotiator, Менеджер пакетов Yarn]
tags: [yarn, package-manager, frontend, dependencies]
---

# Yarn (Yet Another Resource Negotiator)

## Введение

Yarn - это быстрый, надежный и безопасный менеджер пакетов для JavaScript, разработанный Facebook (ныне Meta) в 2016 году. Он был создан как альтернатива NPM с упором на производительность, детерминированность и безопасность. Yarn быстро завоевал популярность среди разработчиков благодаря своим уникальным возможностям и улучшенной производительности.

## История и развитие

Yarn был представлен в октябре 2016 года как ответ на проблемы, с которыми сталкивались разработчики при использовании NPM в то время:
- Непредсказуемость установки пакетов
- Медленная производительность
- Потенциальные уязвимости безопасности

С тех пор Yarn прошел через несколько версий:
- Yarn v1 (Classic) - первая версия с основными функциями
- Yarn v2 (Berry) - полная переписка с улучшенными возможностями
- Yarn v3 и v4 - дальнейшие улучшения и оптимизации

## Основные особенности

### Производительность

Yarn достигает высокой производительности за счет:
- Параллельной установки пакетов
- Кэширования установленных пакетов
- Эффективного алгоритма разрешения зависимостей

```bash
# Установка пакетов с высокой скоростью
yarn install

# Установка с отключением кэширования
yarn install --no-cache
```

### Детерминированность

Yarn гарантирует, что установка пакетов даст одинаковый результат на разных машинах благодаря файлу yarn.lock, который фиксирует точные версии всех зависимостей.

### Безопасность

Yarn включает встроенную проверку безопасности:
```bash
# Аудит зависимостей
yarn audit

# Проверка с отчетом
yarn audit --json
```

## Установка и настройка

### Установка Yarn

Yarn можно установить несколькими способами:

```bash
# Установка через NPM
npm install -g yarn

# Установка через Homebrew (macOS)
brew install yarn

# Установка через Chocolatey (Windows)
choco install yarn

# Установка через скрипт
curl -o- -L https://yarnpkg.com/install.sh | bash
```

### Инициализация проекта

```bash
# Создание нового проекта
yarn init

# Инициализация с ответами по умолчанию
yarn init --yes
```

## Основные команды

### Управление пакетами

```bash
# Установка всех зависимостей
yarn install

# Установка конкретного пакета
yarn add package-name

# Установка как зависимости разработки
yarn add package-name --dev

# Установка как peer зависимости
yarn add package-name --peer

# Удаление пакета
yarn remove package-name

# Обновление пакета
yarn upgrade package-name

# Обновление всех пакетов
yarn upgrade-interactive
```

### Работа со скриптами

```bash
# Запуск скриптов из package.json
yarn run script-name

# Запуск скрипта без слова run
yarn script-name

# Просмотр доступных скриптов
yarn run
```

### Информация о проекте

```bash
# Просмотр установленных пакетов
yarn list

# Просмотр зависимостей
yarn info package-name

# Проверка устаревших пакетов
yarn outdated
```

## Файлы конфигурации

### package.json

Yarn использует стандартный файл package.json для определения зависимостей и скриптов:

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "jest",
    "build": "webpack --mode production"
  },
  "dependencies": {
    "react": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "jest": "^29.0.0"
  }
}
```

### yarn.lock

Файл yarn.lock фиксирует точные версии всех установленных пакетов и их зависимостей, обеспечивая воспроизводимость сборки:

```yaml
# yarn lockfile v1
react@^18.0.0:
  version "18.2.0"
  resolved "https://registry.yarnpkg.com/react/-/react-18.2.0.tgz#..."
  integrity sha512-/3IjMdb2L9QbBdWiW5e3P2/npwMBaU9mHCSCUzNln0ZCYbcfTsGbTJrU/kGemdH2IWmB2ioZ+zkxtmq2Y5ZQhg==
```

### .yarnrc

Файл .yarnrc позволяет настраивать поведение Yarn:

```
# Установка глобального пути установки
yarn-path ".yarn/releases/yarn-3.2.0.cjs"

# Настройка прокси
proxy "http://proxy.company.com:8080/"

# Установка режима работы
nodeLinker "node-modules"

# Настройка кэша
enableGlobalCache true
```

## Работа с версиями

### Стратегии версионирования

Yarn поддерживает все стандартные стратегии версионирования:
- Точная версия: "1.0.0"
- Карет: "^1.0.0" (совместимость с мажорной версией)
- Тильда: "~1.0.0" (совместимость с минорной версией)
- Диапазоны: ">=1.0.0 <2.0.0"

### Управление версиями

```bash
# Просмотр текущей версии Yarn
yarn --version

# Установка новой версии Yarn
yarn set version berry

# Повышение версии проекта
yarn version major
yarn version minor
yarn version patch
```

## Безопасность

### Встроенные механизмы безопасности

Yarn предоставляет несколько уровней безопасности:

```bash
# Проверка уязвимостей
yarn audit

# Проверка с отчетом в формате JSON
yarn audit --json

# Автоматическое исправление
yarn audit --fix
```

### Проверка целостности

Yarn проверяет целостность пакетов с помощью хэшей, указанных в yarn.lock файле, предотвращая установку измененных или вредоносных пакетов.

### Работа с приватными репозиториями

```bash
# Настройка приватного репозитория
yarn config set npmScopes.mycompany.npmRegistryServer "https://npm.mycompany.com"

# Установка токена аутентификации
yarn config set npmScopes.mycompany.npmAlwaysAuth true
```

## Производительность и оптимизация

### Кэширование

Yarn активно использует кэширование для ускорения установки пакетов:

```bash
# Просмотр информации о кэше
yarn cache dir
yarn cache list

# Очистка кэша
yarn cache clean

# Проверка кэша
yarn cache check
```

### Параллельная установка

Yarn устанавливает пакеты параллельно, что значительно ускоряет процесс:

```bash
# Установка с ограничением количества параллельных операций
yarn install --network-concurrency 1
```

### Zero-Installs (Yarn v2+)

Yarn v2+ вводит концепцию Zero-Installs, при которой зависимости хранятся в ZIP-архивах и не требуют распаковки:

```bash
# Включение Zero-Installs
yarn config set pnpEnable true
```

## Работа в команде

### Совместимость между разработчиками

Yarn обеспечивает согласованность между разными разработчиками за счет:
- Файла yarn.lock
- Стандартизированного процесса установки
- Проверки версий зависимостей

### Работа с моно-репозиториями

Yarn Workspace позволяет управлять несколькими пакетами в одном репозитории:

```json
// package.json
{
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

```bash
# Установка зависимостей для всех рабочих областей
yarn install

# Запуск скрипта во всех рабочих областях
yarn workspaces foreach run build

# Запуск скрипта в конкретной рабочей области
yarn workspace package-name run script
```

## Сравнение с NPM

### Преимущества Yarn

1. **Более высокая производительность** - благодаря параллельной установке
2. **Детерминированность** - гарантия одинакового результата установки
3. **Безопасность** - встроенные механизмы проверки
4. **Офлайн установка** - возможность установки из кэша
5. **Интерактивное обновление** - удобный интерфейс для обновления пакетов

### Различия в работе

```bash
# NPM
npm install
npm install package-name --save-dev

# Yarn
yarn install
yarn add package-name --dev
```

## Практические примеры

### Создание нового проекта

```bash
mkdir my-yarn-project
cd my-yarn-project
yarn init
yarn add react react-dom
yarn add --dev jest eslint
```

### Управление скриптами

```json
{
  "scripts": {
    "dev": "webpack serve --mode development",
    "build": "webpack --mode production",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  }
}
```

### Работа с зависимостями

```bash
# Интерактивное обновление зависимостей
yarn upgrade-interactive

# Обновление конкретной зависимости
yarn up react@^18.2.0

# Проверка устаревших пакетов
yarn outdated
```

## Лучшие практики

### Управление зависимостями

- Регулярное обновление зависимостей
- Использование yarn.lock для обеспечения стабильности
- Проверка безопасности перед установкой новых пакетов
- Удаление неиспользуемых зависимостей

### Оптимизация производительности

- Использование кэширования в CI/CD
- Регулярная очистка кэша
- Использование yarn install --frozen-lockfile в продакшене
- Оптимизация структуры зависимостей

### Работа в команде

- Всегда коммитить yarn.lock
- Использовать одинаковую версию Yarn
- Регулярно обновлять yarn.lock
- Обучать команду работе с Yarn

## Миграция с NPM

### Преобразование проекта

```bash
# Удаление node_modules и package-lock.json
rm -rf node_modules package-lock.json

# Инициализация Yarn
yarn install
```

### Преобразование команд

| NPM | Yarn |
|-----|------|
| npm install | yarn install |
| npm install pkg | yarn add pkg |
| npm install pkg --save-dev | yarn add pkg --dev |
| npm uninstall pkg | yarn remove pkg |
| npm run script | yarn script |

## Расширенные возможности (Yarn v2+)

### Plug'n'Play (PnP)

Yarn v2+ вводит систему Plug'n'Play, которая устраняет необходимость в node_modules:

```bash
# Включение PnP
yarn config set nodeLinker pnp

# Генерация .pnp.cjs файла
yarn install
```

### Zero-Install Workspaces

Хранение зависимостей в архивах для быстрой установки:

```bash
# Включение zero-installs
yarn config set pnpEnable true
yarn config set pnpMode loose
```

## Заключение

Yarn остается мощным инструментом для управления зависимостями в JavaScript проектах. Его акцент на производительности, безопасности и детерминированности делает его отличным выбором для команд разработчиков, особенно в крупных проектах.

Понимание возможностей Yarn и следование лучшим практикам позволяет эффективно управлять зависимостями, ускорять процессы разработки и обеспечивать стабильность проектов.

## Связанные концепции

- [[Dependency Management]] - Управление зависимостями
- [[Version Management]] - Управление версиями
- [[NPM]] - Стандартный менеджер пакетов
- [[PNPM]] - Альтернативный быстрый менеджер пакетов
- [[Package Management]] - Общие принципы управления пакетами
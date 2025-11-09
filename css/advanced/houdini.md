# CSS Houdini

CSS Houdini — это набор низкоуровневых API, которые позволяют разработчикам расширять возможности CSS, создавая собственные свойства, функции и даже алгоритмы отрисовки и компоновки. Эти API обеспечивают прямой доступ к CSS-движку браузера, позволяя писать код, который интегрируется на уровне браузера, а не работает поверх него, как это делает JavaScript.

## Назначение

Основная цель CSS Houdini — предоставить разработчикам инструменты для:

- Создания пользовательских CSS-свойств с определённым поведением
- Определения пользовательских функций отрисовки (например, сложных градиентов или узоров)
- Управления процессом компоновки элементов на странице
- Повышения производительности за счёт более тесной интеграции с движком браузера

## Основные API

### 1. Paint API

Paint API позволяет создавать пользовательские функции отрисовки для CSS-свойств `background`, `border`, `mask` и других. Это позволяет рисовать сложные изображения на canvas уровне, но с использованием CSS-свойств.

**Пример:**

```css
.element {
  background-image: paint(myAwesomePattern);
}
```

```js
// my-awesome-pattern.js
registerPaint('myAwesomePattern', class {
  static get inputProperties() {
    return ['--color1', '--color2'];
  }

  paint(ctx, geom, properties) {
    const color1 = properties.get('--color1').toString();
    const color2 = properties.get('--color2').toString();
    // Рисуем градиент или узор
    const gradient = ctx.createLinearGradient(0, 0, geom.width, geom.height);
    gradient.addColorStop(0, color1);
    gradient.addColorStop(1, color2);
    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, geom.width, geom.height);
  }
});
```

### 2. Layout API

Layout API позволяет разработчикам определять собственные алгоритмы компоновки. Это даёт возможность создавать уникальные системы выравнивания и распределения элементов.

**Пример:**

```css
.container {
  display: layout(myCustomLayout);
}
```

> [!warning]
> Layout API пока не реализован в большинстве браузеров и находится в стадии разработки.

### 3. Properties and Values API

Properties and Values API позволяет объявлять пользовательские CSS-свойства с определёнными типами, начальными значениями и наследованием. Это позволяет использовать кастомные свойства более безопасно и предсказуемо.

**Пример:**

```js
CSS.registerProperty({
  name: '--my-color',
  syntax: '<color>',
  inherits: false,
  initialValue: 'red'
});
```

```css
.element {
  background-color: var(--my-color);
}
```

## Примеры использования

- Адаптивные градиенты и узоры, которые реагируют на размеры элемента
- Сложные анимации, управляемые через CSS-переменные
- Создание собственных систем макета
- Улучшение производительности за счёт избежания JavaScript для стилизации

## Поддержка браузерами

| API | Chrome | Firefox | Safari | Edge |
|-----|--------|---------|--------|------|
| Paint API | ✅ | ❌ | ❌ | ✅ |
| Properties and Values API | ✅ | ✅ | ❌ | ✅ |
| Layout API | ❌ (в разработке) | ❌ | ❌ | ❌ |

> [!note]
> Поддержка браузерами может меняться. Рекомендуется проверять актуальность на [Can I use](https://caniuse.com/) и других ресурсах.

## См. также

- [[css/custom-properties]] — пользовательские свойства CSS
- [[css/animations]] — анимации в CSS
- [[css/grid]] — гриды в CSS
- [[js/web-components]] — веб-компоненты, которые могут использовать Houdini

## Теги

#css #houdini #advanced #web-development #frontend
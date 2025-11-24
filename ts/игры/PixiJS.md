---
aliases: [PixiJS, PixiJS Framework]
tags: [typescript, game-development, pixijs, webgl, canvas]
---

# PixiJS

PixiJS — это мощный 2D WebGL рендерер, который также поддерживает резервный Canvas2D. Он идеально подходит для создания быстрых, интерактивных 2D-контентов и игр с высокой производительностью.

## Установка

```bash
npm install pixi.js
npm install @types/pixi.js
```

## Базовое использование

### Создание приложения

```typescript
import * as PIXI from 'pixi.js';

// Создание приложения
const app = new PIXI.Application({
    width: 800,
    height: 600,
    backgroundColor: 0x1099bb,
    resolution: window.devicePixelRatio || 1
});

document.body.appendChild(app.view);

// Загрузка ресурсов
app.loader
    .add('player', 'assets/player.png')
    .add('background', 'assets/background.png')
    .load(setup);

function setup(loader: PIXI.Loader, resources: any) {
    // Создание спрайтов
    const background = new PIXI.Sprite(resources.background.texture);
    app.stage.addChild(background);
    
    const player = new PIXI.Sprite(resources.player.texture);
    player.x = 100;
    player.y = 100;
    app.stage.addChild(player);
    
    // Анимация
    app.ticker.add(() => {
        player.rotation += 0.01;
    });
}
```

## Сцены и слои

### Использование контейнеров

```typescript
// Создание контейнера для группы объектов
const gameContainer = new PIXI.Container();
app.stage.addChild(gameContainer);

// Добавление объектов в контейнер
const player = new PIXI.Sprite(PIXI.Texture.from('player.png'));
gameContainer.addChild(player);

// Перемещение всей группы
gameContainer.x = 100;
gameContainer.y = 100;
```

## Текстуры и спрайты

### Создание и управление текстурами

```typescript
// Загрузка текстуры
const texture = PIXI.Texture.from('assets/texture.png');

// Создание спрайта
const sprite = new PIXI.Sprite(texture);

// Изменение свойств спрайта
sprite.x = 100;
sprite.y = 100;
sprite.width = 100;
sprite.height = 100;
sprite.alpha = 0.5; // Прозрачность
sprite.rotation = Math.PI / 4; // Поворот
sprite.scale.set(2); // Масштаб
```

### Спрайт-атласы

```typescript
// Загрузка спрайт-атласа
app.loader.add('atlas', 'assets/atlas.json').load(() => {
    const player = new PIXI.Sprite(app.loader.resources.atlas.textures['player.png']);
    app.stage.addChild(player);
});
```

## Анимации

### Анимация с помощью текстур

```typescript
// Загрузка текстур для анимации
app.loader.add('run', 'assets/run.json').load(() => {
    const textures = [];
    for (let i = 0; i < 4; i++) {
        textures.push(app.loader.resources.run.textures[`run${i}.png`]);
    }
    
    // Создание анимированного спрайта
    const animSprite = new PIXI.AnimatedSprite(textures);
    animSprite.animationSpeed = 0.1;
    animSprite.play();
    app.stage.addChild(animSprite);
});
```

### Использование ticker для анимации

```typescript
// Плавное движение
const player = new PIXI.Sprite(PIXI.Texture.from('player.png'));
app.stage.addChild(player);

app.ticker.add((delta) => {
    player.x += 1 * delta;
    
    // Ограничение движения
    if (player.x > app.screen.width) {
        player.x = 0;
    }
});
```

## Взаимодействие

### Обработка событий мыши

```typescript
const button = new PIXI.Sprite(PIXI.Texture.from('button.png'));
button.interactive = true;
button.buttonMode = true;

button.on('pointerdown', () => {
    console.log('Кнопка нажата!');
});

button.on('pointerover', () => {
    button.alpha = 0.8;
});

button.on('pointerout', () => {
    button.alpha = 1;
});

app.stage.addChild(button);
```

## Фильтры и эффекты

### Применение фильтров

```typescript
// Создание фильтра размытия
const blurFilter = new PIXI.filters.BlurFilter();
blurFilter.blur = 5;

const sprite = new PIXI.Sprite(PIXI.Texture.from('image.png'));
sprite.filters = [blurFilter];

app.stage.addChild(sprite);
```

### Доступные фильтры

- `BlurFilter` - размытие
- `ColorMatrixFilter` - цветовые преобразования
- `DisplacementFilter` - искажение
- `DropShadowFilter` - тени
- `GlowFilter` - свечение

## Тайлинг и повторение текстур

```typescript
// Повторяющаяся текстура
const texture = PIXI.Texture.from('tile.png');
const tilingSprite = new PIXI.TilingSprite(
    texture,
    app.screen.width,
    app.screen.height
);

app.stage.addChild(tilingSprite);

// Анимация тайлинга
app.ticker.add(() => {
    tilingSprite.tilePosition.x -= 1;
});
```

## Производительность

### Использование Particle Containers

```typescript
// Для большого количества однотипных объектов
const container = new PIXI.ParticleContainer(10000, {
    scale: true,
    position: true,
    rotation: true,
    uvs: true,
    alpha: true
});

app.stage.addChild(container);

// Добавление спрайтов в контейнер
for (let i = 0; i < 1000; i++) {
    const sprite = new PIXI.Sprite(PIXI.Texture.from('particle.png'));
    sprite.x = Math.random() * app.screen.width;
    sprite.y = Math.random() * app.screen.height;
    container.addChild(sprite);
}
```

## Слои и Z-индекс

```typescript
// Управление порядком отображения
const sprite1 = new PIXI.Sprite(PIXI.Texture.from('sprite1.png'));
const sprite2 = new PIXI.Sprite(PIXI.Texture.from('sprite2.png'));

// sprite2 будет отображаться поверх sprite1
app.stage.addChild(sprite1);
app.stage.addChild(sprite2);

// Изменение Z-индекса
sprite1.zIndex = 2; // sprite1 теперь поверх sprite2
app.stage.sortableChildren = true; // Включить сортировку
```

## Загрузка ресурсов

```typescript
// Продвинутая загрузка
app.loader
    .add('bg', 'assets/background.png')
    .add('player', 'assets/player.png')
    .add('sounds', 'assets/sounds.json')
    .on('progress', (loader, resource) => {
        console.log(`Загружено: ${loader.progress}%`);
    })
    .load(() => {
        console.log('Все ресурсы загружены!');
    });
```

## Ресурсы

- [Официальный сайт PixiJS](https://pixijs.com/)
- [GitHub репозиторий](https://github.com/pixijs/pixijs)
- [[Игровые-паттерны]]
- [[Анимации-в-играх]]

## См. также

- [[Phaser]]
- [[WebGL]]
- [[Canvas API]]
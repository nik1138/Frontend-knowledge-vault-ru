---
aliases: [Phaser Framework, Игровой движок Phaser]
tags: [typescript, game-development, phaser, canvas]
---

# Phaser

Phaser — это мощный и гибкий фреймворк для разработки 2D-игр на JavaScript и TypeScript. Он поддерживает как Canvas, так и WebGL рендеринг и предоставляет богатый набор инструментов для создания игр с высокой производительностью.

## Установка

Для использования Phaser с TypeScript:

```bash
npm install phaser
npm install @types/phaser
```

## Основы создания игры

### Базовая структура игры

```typescript
import Phaser from 'phaser';

const config: Phaser.Types.Core.GameConfig = {
    type: Phaser.AUTO,
    width: 800,
    height: 600,
    physics: {
        default: 'arcade',
        arcade: {
            gravity: { y: 200 }
        }
    },
    scene: {
        preload: preload,
        create: create,
        update: update
    }
};

function preload(this: Phaser.Scene) {
    // Загрузка ресурсов
    this.load.image('player', 'assets/player.png');
}

function create(this: Phaser.Scene) {
    // Создание игровых объектов
    const player = this.scene.add.sprite(400, 300, 'player');
}

function update(this: Phaser.Scene, time: number, delta: number) {
    // Логика обновления
}

const game = new Phaser.Game(config);
```

## Сцены

Phaser использует систему сцен для организации игры:

```typescript
class GameScene extends Phaser.Scene {
    constructor() {
        super({ key: 'GameScene' });
    }

    preload() {
        this.load.image('background', 'assets/bg.png');
        this.load.spritesheet('dude', 'assets/dude.png', {
            frameWidth: 32,
            frameHeight: 48
        });
    }

    create() {
        this.add.image(400, 300, 'background');
        
        // Создание игрока
        this.player = this.physics.add.sprite(100, 450, 'dude');
        this.player.setBounce(0.2);
        this.player.setCollideWorldBounds(true);
        
        // Анимации
        this.anims.create({
            key: 'left',
            frames: this.anims.generateFrameNumbers('dude', { start: 0, end: 3 }),
            frameRate: 10,
            repeat: -1
        });
    }

    update(time: number, delta: number) {
        // Обновление логики сцены
    }
}
```

## Физика

Phaser включает встроенные физические движки:

### Arcade Physics

```typescript
create() {
    // Создание игрока и платформы
    this.player = this.physics.add.sprite(100, 450, 'player');
    this.platforms = this.physics.add.staticGroup();
    
    this.platforms.create(400, 568, 'ground').setScale(2).refreshBody();
    this.platforms.create(600, 400, 'ground');
    this.platforms.create(50, 250, 'ground');
    this.platforms.create(750, 220, 'ground');
    
    // Коллизии
    this.physics.add.collider(this.player, this.platforms);
}
```

## Управление

```typescript
create() {
    this.cursors = this.input.keyboard.createCursorKeys();
}

update() {
    if (this.cursors.left.isDown) {
        this.player.setVelocityX(-160);
    } else if (this.cursors.right.isDown) {
        this.player.setVelocityX(160);
    } else {
        this.player.setVelocityX(0);
    }

    if (this.cursors.up.isDown && this.player.body.touching.down) {
        this.player.setVelocityY(-330);
    }
}
```

## Анимации

```typescript
create() {
    // Создание анимации
    this.anims.create({
        key: 'walk',
        frames: this.anims.generateFrameNumbers('player', { start: 0, end: 3 }),
        frameRate: 10,
        repeat: -1
    });
    
    // Запуск анимации
    this.player.play('walk');
}
```

## Звук

```typescript
preload() {
    this.load.audio('jump', 'assets/jump.wav');
}

create() {
    this.jumpSound = this.sound.add('jump');
}

// Воспроизведение звука
this.jumpSound.play();
```

## Состояния игры

```typescript
class GameState {
    score: number = 0;
    lives: number = 3;
    gameOver: boolean = false;
    
    updateScore(points: number) {
        this.score += points;
    }
    
    loseLife() {
        this.lives--;
        if (this.lives <= 0) {
            this.gameOver = true;
        }
    }
}
```

## Ресурсы

- [Официальный сайт Phaser](https://phaser.io/)
- [Документация](https://photonstorm.github.io/phaser3-docs/)
- [[Игровые-паттерны]]
- [[Анимации-в-играх]]
- [[Физика-в-играх]]

## См. также

- [[PixiJS]]
- [[TypeScript для игр]]
- [[Canvas API]]
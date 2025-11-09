# WebGL и Three.js

WebGL (Web Graphics Library) - это JavaScript API для рендеринга сложной 3D графики в браузере без использования плагинов. Three.js - популярная библиотека, упрощающая работу с WebGL.

## Основы WebGL

### Создание WebGL контекста

```javascript
// Получение canvas и WebGL контекста
const canvas = document.getElementById('webgl-canvas');
const gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');

if (!gl) {
    console.error('WebGL не поддерживается в вашем браузере');
}

// Проверка возможностей WebGL
console.log('WebGL поддерживается:', !!gl);
console.log('WebGL версия:', gl.getParameter(gl.VERSION));
console.log('Вендор:', gl.getParameter(gl.VENDOR));
console.log('Рендерер:', gl.getParameter(gl.RENDERER));
```

### Основные понятия WebGL

```javascript
// Основные компоненты WebGL конвейера

// 1. Вершинный шейдер - обрабатывает вершины
const vertexShaderSource = `
    attribute vec4 a_position;
    attribute vec4 a_color;
    varying vec4 v_color;
    
    void main() {
        gl_Position = a_position;
        v_color = a_color;
    }
`;

// 2. Фрагментный шейдер - определяет цвет пикселей
const fragmentShaderSource = `
    precision mediump float;
    varying vec4 v_color;
    
    void main() {
        gl_FragColor = v_color;
    }
`;

// Компиляция шейдеров
function createShader(gl, type, source) {
    const shader = gl.createShader(type);
    gl.shaderSource(shader, source);
    gl.compileShader(shader);
    
    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
        console.error('Ошибка компиляции шейдера:', gl.getShaderInfoLog(shader));
        gl.deleteShader(shader);
        return null;
    }
    
    return shader;
}

const vertexShader = createShader(gl, gl.VERTEX_SHADER, vertexShaderSource);
const fragmentShader = createShader(gl, gl.FRAGMENT_SHADER, fragmentShaderSource);

// Создание программы
function createProgram(gl, vertexShader, fragmentShader) {
    const program = gl.createProgram();
    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);
    gl.linkProgram(program);
    
    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
        console.error('Ошибка линковки программы:', gl.getProgramInfoLog(program));
        gl.deleteProgram(program);
        return null;
    }
    
    return program;
}

const program = createProgram(gl, vertexShader, fragmentShader);
gl.useProgram(program);
```

### Буферы и атрибуты

```javascript
// Создание буфера вершин
function createVertexBuffer(gl, data) {
    const buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(data), gl.STATIC_DRAW);
    return buffer;
}

// Пример данных для треугольника
const triangleVertices = [
    // Позиции      // Цвета
     0.0,  0.5, 0.0,  1.0, 0.0, 0.0, 1.0,  // Красная вершина
    -0.5, -0.5, 0.0,  0.0, 1.0, 0.0, 1.0,  // Зеленая вершина
     0.5, -0.5, 0.0,  0.0, 0.0, 1.0, 1.0   // Синяя вершина
];

const vertexBuffer = createVertexBuffer(gl, triangleVertices);

// Настройка атрибутов
const positionLocation = gl.getAttribLocation(program, 'a_position');
const colorLocation = gl.getAttribLocation(program, 'a_color');

// Позиции
gl.enableVertexAttribArray(positionLocation);
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
gl.vertexAttribPointer(positionLocation, 3, gl.FLOAT, false, 28, 0); // 28 = размер одной вершины в байтах

// Цвета
gl.enableVertexAttribArray(colorLocation);
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
gl.vertexAttribPointer(colorLocation, 4, gl.FLOAT, false, 28, 12); // смещение 12 байт до цвета
```

### Отрисовка

```javascript
// Очистка и отрисовка
function render() {
    // Очистка цветового буфера
    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
    
    // Отрисовка треугольника
    gl.drawArrays(gl.TRIANGLES, 0, 3); // режим, стартовая точка, количество вершин
}

// Анимация
function animate() {
    render();
    requestAnimationFrame(animate);
}

// animate();
```

## Three.js - упрощенная работа с WebGL

### Основные компоненты Three.js

```javascript
// Установка Three.js сцены
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ canvas: document.getElementById('threejs-canvas') });

renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setClearColor(0x222222); // Темный фон

// Добавление источника света
const ambientLight = new THREE.AmbientLight(0x404040);
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
directionalLight.position.set(1, 1, 1);
scene.add(directionalLight);
```

### Создание геометрии и материалов

```javascript
// Создание куба
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshPhongMaterial({ 
    color: 0x00ff00,
    shininess: 30
});

const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// Позиционирование камеры
camera.position.z = 5;

// Отрисовка сцены
function animate() {
    requestAnimationFrame(animate);
    
    // Анимация вращения куба
    cube.rotation.x += 0.01;
    cube.rotation.y += 0.01;
    
    renderer.render(scene, camera);
}

// animate();
```

### Сложные геометрии

```javascript
// Создание сложных геометрий
class ComplexGeometries {
    constructor(scene) {
        this.scene = scene;
        this.objects = [];
    }
    
    createSphere(radius, segments = 32) {
        const geometry = new THREE.SphereGeometry(radius, segments, segments);
        const material = new THREE.MeshPhongMaterial({ 
            color: 0xff0000,
            shininess: 50
        });
        
        const sphere = new THREE.Mesh(geometry, material);
        this.scene.add(sphere);
        this.objects.push(sphere);
        return sphere;
    }
    
    createCylinder(radiusTop, radiusBottom, height, radialSegments = 32) {
        const geometry = new THREE.CylinderGeometry(radiusTop, radiusBottom, height, radialSegments);
        const material = new THREE.MeshPhongMaterial({ 
            color: 0x0000ff,
            shininess: 30
        });
        
        const cylinder = new THREE.Mesh(geometry, material);
        this.scene.add(cylinder);
        this.objects.push(cylinder);
        return cylinder;
    }
    
    createTorus(radius, tube, radialSegments = 16, tubularSegments = 100) {
        const geometry = new THREE.TorusGeometry(radius, tube, radialSegments, tubularSegments);
        const material = new THREE.MeshPhongMaterial({ 
            color: 0xffff00,
            shininess: 20
        });
        
        const torus = new THREE.Mesh(geometry, material);
        this.scene.add(torus);
        this.objects.push(torus);
        return torus;
    }
    
    createCustomGeometry() {
        // Создание пользовательской геометрии
        const geometry = new THREE.BufferGeometry();
        
        // Определение вершин
        const vertices = new Float32Array([
            -1.0, -1.0,  1.0,
             1.0, -1.0,  1.0,
             1.0,  1.0,  1.0,
            -1.0,  1.0,  1.0,
            -1.0, -1.0, -1.0,
             1.0, -1.0, -1.0,
             1.0,  1.0, -1.0,
            -1.0,  1.0, -1.0,
        ]);
        
        // Определение индексов для граней
        const indices = [
            0, 1, 2,  0, 2, 3,  // передняя грань
            4, 6, 5,  4, 7, 6,  // задняя грань
            4, 5, 1,  4, 1, 0,  // нижняя грань
            7, 2, 6,  7, 3, 2,  // верхняя грань
            1, 5, 6,  1, 6, 2,  // правая грань
            4, 0, 3,  4, 3, 7   // левая грань
        ];
        
        geometry.setIndex(indices);
        geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
        geometry.computeVertexNormals(); // вычисление нормалей для правильного освещения
        
        const material = new THREE.MeshPhongMaterial({ 
            color: 0xff00ff,
            shininess: 40,
            side: THREE.DoubleSide // отображение с обеих сторон
        });
        
        const customMesh = new THREE.Mesh(geometry, material);
        this.scene.add(customMesh);
        this.objects.push(customMesh);
        return customMesh;
    }
}

// Использование
const complexGeos = new ComplexGeometries(scene);
complexGeos.createSphere(1);
complexGeos.createCylinder(0.5, 0.5, 2);
complexGeos.createTorus(1, 0.4);
complexGeos.createCustomGeometry();
```

### Текстуры и материалы

```javascript
// Загрузка текстур
const textureLoader = new THREE.TextureLoader();

// Простая текстура
const texture = textureLoader.load('path/to/texture.jpg');

// Создание материала с текстурой
const texturedMaterial = new THREE.MeshPhongMaterial({ 
    map: texture,
    shininess: 30
});

// Повторяющаяся текстура
const repeatingTexture = textureLoader.load('path/to/pattern.jpg');
repeatingTexture.wrapS = THREE.RepeatWrapping;
repeatingTexture.wrapT = THREE.RepeatWrapping;
repeatingTexture.repeat.set(4, 4);

const repeatingMaterial = new THREE.MeshPhongMaterial({ 
    map: repeatingTexture
});

// Создание шейдрового материала
const shaderMaterial = new THREE.ShaderMaterial({
    vertexShader: `
        varying vec2 vUv;
        void main() {
            vUv = uv;
            gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
    `,
    fragmentShader: `
        uniform float time;
        varying vec2 vUv;
        void main() {
            vec3 color = 0.5 + 0.5 * cos(time + vUv.xyx + vec3(0, 2, 4));
            gl_FragColor = vec4(color, 1.0);
        }
    `,
    uniforms: {
        time: { value: 0 }
    }
});

// Анимированный шейдер
function animateShader(time) {
    shaderMaterial.uniforms.time.value = time * 0.001;
    requestAnimationFrame(animateShader);
}

// animateShader(0);
```

## Продвинутые возможности Three.js

### Загрузка 3D моделей

```javascript
// Загрузка GLTF моделей
import { GLTFLoader } from 'https://cdn.jsdelivr.net/npm/three@0.144.0/examples/jsm/loaders/GLTFLoader.js';

class ModelLoader {
    constructor(scene) {
        this.scene = scene;
        this.loader = new GLTFLoader();
    }
    
    async loadModel(path, position = { x: 0, y: 0, z: 0 }) {
        try {
            const gltf = await this.loader.loadAsync(path);
            const model = gltf.scene;
            
            model.position.set(position.x, position.y, position.z);
            this.scene.add(model);
            
            return model;
        } catch (error) {
            console.error('Ошибка загрузки модели:', error);
            throw error;
        }
    }
    
    // Загрузка с анимацией
    async loadAnimatedModel(path) {
        const gltf = await this.loader.loadAsync(path);
        const model = gltf.scene;
        const animations = gltf.animations;
        
        this.scene.add(model);
        
        // Настройка анимации
        if (animations && animations.length > 0) {
            const mixer = new THREE.AnimationMixer(model);
            const action = mixer.clipAction(animations[0]);
            action.play();
            
            return { model, mixer, action };
        }
        
        return { model };
    }
}

// Использование
// const modelLoader = new ModelLoader(scene);
// const robotModel = await modelLoader.loadModel('path/to/robot.gltf', { x: 2, y: 0, z: 0 });
```

### Пост-обработка

```javascript
// Импорт необходимых компонентов
import { EffectComposer } from 'https://cdn.jsdelivr.net/npm/three@0.144.0/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'https://cdn.jsdelivr.net/npm/three@0.144.0/examples/jsm/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'https://cdn.jsdelivr.net/npm/three@0.144.0/examples/jsm/postprocessing/UnrealBloomPass.js';

class PostProcessing {
    constructor(renderer, scene, camera) {
        this.renderer = renderer;
        this.scene = scene;
        this.camera = camera;
        
        // Создание composer для пост-обработки
        this.composer = new EffectComposer(renderer);
        this.composer.addPass(new RenderPass(scene, camera));
        
        // Добавление эффекта свечения
        const bloomPass = new UnrealBloomPass(
            new THREE.Vector2(window.innerWidth, window.innerHeight),
            1.5,   // strength
            0.4,   // radius
            0.85   // threshold
        );
        this.composer.addPass(bloomPass);
    }
    
    render() {
        this.composer.render();
    }
}

// Использование
// const postProcessing = new PostProcessing(renderer, scene, camera);
// postProcessing.render();
```

### Физика с использованием Ammo.js

```javascript
// Импорт Ammo.js (библиотека физики)
// <script src="https://cdn.jsdelivr.net/npm/ammo.js@2.0.1/builds/ammo.wasm.js"></script>

class PhysicsWorld {
    constructor() {
        this.world = null;
        this.objects = [];
        this.raycaster = new THREE.Raycaster();
    }
    
    async init() {
        // Ammo.js использует WebAssembly, поэтому нужен асинхронный вызов
        Ammo().then((AmmoLib) => {
            this.Ammo = AmmoLib;
            this.setupPhysics();
        });
    }
    
    setupPhysics() {
        // Создание физического мира
        const collisionConfiguration = new this.Ammo.btDefaultCollisionConfiguration();
        const dispatcher = new this.Ammo.btCollisionDispatcher(collisionConfiguration);
        const overlappingPairCache = new this.Ammo.btDbvtBroadphase();
        const solver = new this.Ammo.btSequentialImpulseConstraintSolver();
        
        this.world = new this.Ammo.btDiscreteDynamicsWorld(
            dispatcher, 
            overlappingPairCache, 
            solver, 
            collisionConfiguration
        );
        
        this.world.setGravity(new this.Ammo.btVector3(0, -9.8, 0));
    }
    
    addRigidBody(mesh, mass, friction = 0.5, restitution = 0.3) {
        if (!this.world) return;
        
        // Создание формы тела
        const shape = new this.Ammo.btBoxShape(new this.Ammo.btVector3(
            mesh.geometry.parameters.width / 2,
            mesh.geometry.parameters.height / 2,
            mesh.geometry.parameters.depth / 2
        ));
        
        const transform = new this.Ammo.btTransform();
        transform.setIdentity();
        transform.setOrigin(new this.Ammo.btVector3(mesh.position.x, mesh.position.y, mesh.position.z));
        
        const motionState = new this.Ammo.btDefaultMotionState(transform);
        const localInertia = new this.Ammo.btVector3(0, 0, 0);
        shape.calculateLocalInertia(mass, localInertia);
        
        const rbInfo = new this.Ammo.btRigidBodyConstructionInfo(
            mass, 
            motionState, 
            shape, 
            localInertia
        );
        
        rbInfo.set_m_friction(friction);
        rbInfo.set_m_restitution(restitution);
        
        const body = new this.Ammo.btRigidBody(rbInfo);
        this.world.addRigidBody(body);
        
        this.objects.push({ mesh, body });
    }
    
    update(deltaTime) {
        if (!this.world) return;
        
        this.world.stepSimulation(deltaTime, 10);
        
        // Синхронизация позиций Three.js объектов с физическим миром
        for (const obj of this.objects) {
            const motionState = obj.body.getMotionState();
            const worldTransform = new this.Ammo.btTransform();
            motionState.getWorldTransform(worldTransform);
            
            const pos = worldTransform.getOrigin();
            obj.mesh.position.set(pos.x(), pos.y(), pos.z());
            
            const rot = worldTransform.getRotation();
            obj.mesh.quaternion.set(rot.x(), rot.y(), rot.z(), rot.w());
        }
    }
}

// Использование
// const physics = new PhysicsWorld();
// await physics.init();
// physics.addRigidBody(cube, 1); // масса 1
```

## Создание 3D приложений

### 3D редактор сцены

```javascript
// Класс для 3D редактора
class SceneEditor {
    constructor(containerId) {
        this.container = document.getElementById(containerId);
        this.scene = new THREE.Scene();
        this.camera = new THREE.PerspectiveCamera(75, this.container.clientWidth / this.container.clientHeight, 0.1, 1000);
        this.renderer = new THREE.WebGLRenderer({ antialias: true });
        
        this.init();
        this.setupControls();
        this.setupLights();
    }
    
    init() {
        this.renderer.setSize(this.container.clientWidth, this.container.clientHeight);
        this.renderer.setClearColor(0x333333);
        this.container.appendChild(this.renderer.domElement);
        
        this.camera.position.z = 5;
    }
    
    setupControls() {
        // Управление камерой
        this.controls = new THREE.OrbitControls(this.camera, this.renderer.domElement);
        this.controls.enableDamping = true;
        this.controls.dampingFactor = 0.05;
    }
    
    setupLights() {
        const ambientLight = new THREE.AmbientLight(0x404040, 0.6);
        this.scene.add(ambientLight);
        
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
        directionalLight.position.set(1, 1, 1);
        this.scene.add(directionalLight);
    }
    
    addObject(geometry, material) {
        const mesh = new THREE.Mesh(geometry, material);
        this.scene.add(mesh);
        return mesh;
    }
    
    removeObject(object) {
        this.scene.remove(object);
        object.geometry.dispose();
        if (object.material.map) object.material.map.dispose();
        object.material.dispose();
    }
    
    createPrimitive(type, params = {}) {
        let geometry, material;
        
        switch (type) {
            case 'box':
                geometry = new THREE.BoxGeometry(
                    params.width || 1,
                    params.height || 1,
                    params.depth || 1
                );
                break;
            case 'sphere':
                geometry = new THREE.SphereGeometry(
                    params.radius || 1,
                    params.widthSegments || 32,
                    params.heightSegments || 32
                );
                break;
            case 'cylinder':
                geometry = new THREE.CylinderGeometry(
                    params.radiusTop || 1,
                    params.radiusBottom || 1,
                    params.height || 1,
                    params.radialSegments || 32
                );
                break;
            default:
                throw new Error(`Неизвестный тип примитива: ${type}`);
        }
        
        material = new THREE.MeshPhongMaterial({
            color: params.color || 0x00ff00,
            shininess: params.shininess || 30
        });
        
        return this.addObject(geometry, material);
    }
    
    animate() {
        requestAnimationFrame(() => this.animate());
        
        this.controls.update();
        this.renderer.render(this.scene, this.camera);
    }
    
    resize() {
        this.camera.aspect = this.container.clientWidth / this.container.clientHeight;
        this.camera.updateProjectionMatrix();
        this.renderer.setSize(this.container.clientWidth, this.container.clientHeight);
    }
}

// Использование
// const editor = new SceneEditor('scene-container');
// editor.createPrimitive('box', { width: 2, height: 2, depth: 2, color: 0xff0000 });
// editor.createPrimitive('sphere', { radius: 1, color: 0x0000ff });
// editor.animate();
```

### 3D игры и интерактивные приложения

```javascript
// Класс для 3D игры
class Game3D {
    constructor(containerId) {
        this.container = document.getElementById(containerId);
        this.scene = new THREE.Scene();
        this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        this.renderer = new THREE.WebGLRenderer({ antialias: true });
        
        this.player = null;
        this.enemies = [];
        this.projectiles = [];
        this.score = 0;
        
        this.keys = {};
        
        this.init();
        this.setupControls();
        this.setupGameplay();
    }
    
    init() {
        this.renderer.setSize(window.innerWidth, window.innerHeight);
        this.renderer.setClearColor(0x000022);
        this.container.appendChild(this.renderer.domElement);
        
        this.camera.position.set(0, 5, 10);
        this.camera.lookAt(0, 0, 0);
    }
    
    setupControls() {
        // Клавиатурный контроль
        document.addEventListener('keydown', (e) => {
            this.keys[e.code] = true;
        });
        
        document.addEventListener('keyup', (e) => {
            this.keys[e.code] = false;
        });
        
        // Управление мышью
        this.container.addEventListener('click', () => {
            this.shoot();
        });
    }
    
    setupGameplay() {
        // Создание игрока
        const playerGeometry = new THREE.ConeGeometry(0.5, 1, 8);
        const playerMaterial = new THREE.MeshPhongMaterial({ color: 0x00ff00 });
        this.player = new THREE.Mesh(playerGeometry, playerMaterial);
        this.player.position.y = 1;
        this.scene.add(this.player);
        
        // Создание игрового поля
        const groundGeometry = new THREE.PlaneGeometry(20, 20);
        const groundMaterial = new THREE.MeshPhongMaterial({ 
            color: 0x228822,
            side: THREE.DoubleSide
        });
        const ground = new THREE.Mesh(groundGeometry, groundMaterial);
        ground.rotation.x = -Math.PI / 2;
        this.scene.add(ground);
        
        // Добавление освещения
        const ambientLight = new THREE.AmbientLight(0x404040);
        this.scene.add(ambientLight);
        
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
        directionalLight.position.set(5, 10, 7);
        this.scene.add(directionalLight);
    }
    
    shoot() {
        // Создание снаряда
        const projectileGeometry = new THREE.SphereGeometry(0.1, 8, 8);
        const projectileMaterial = new THREE.MeshPhongMaterial({ color: 0xffff00 });
        const projectile = new THREE.Mesh(projectileGeometry, projectileMaterial);
        
        // Позиционирование снаряда впереди игрока
        projectile.position.copy(this.player.position);
        projectile.position.y += 1;
        
        // Направление снаряда
        const direction = new THREE.Vector3(0, 0, -1);
        direction.applyQuaternion(this.player.quaternion);
        projectile.userData = { direction: direction.normalize() };
        
        this.scene.add(projectile);
        this.projectiles.push(projectile);
    }
    
    updatePlayer(deltaTime) {
        const speed = 5 * deltaTime;
        
        if (this.keys['KeyW'] || this.keys['ArrowUp']) {
            this.player.position.z -= speed;
        }
        if (this.keys['KeyS'] || this.keys['ArrowDown']) {
            this.player.position.z += speed;
        }
        if (this.keys['KeyA'] || this.keys['ArrowLeft']) {
            this.player.position.x -= speed;
        }
        if (this.keys['KeyD'] || this.keys['ArrowRight']) {
            this.player.position.x += speed;
        }
        
        // Ограничение перемещения
        this.player.position.x = Math.max(-9, Math.min(9, this.player.position.x));
        this.player.position.z = Math.max(-9, Math.min(9, this.player.position.z));
    }
    
    updateProjectiles(deltaTime) {
        for (let i = this.projectiles.length - 1; i >= 0; i--) {
            const projectile = this.projectiles[i];
            
            // Перемещение снаряда
            projectile.position.add(projectile.userData.direction.clone().multiplyScalar(10 * deltaTime));
            
            // Удаление снарядов, вышедших за границы
            if (Math.abs(projectile.position.x) > 10 || Math.abs(projectile.position.z) > 10) {
                this.scene.remove(projectile);
                this.projectiles.splice(i, 1);
                projectile.geometry.dispose();
                projectile.material.dispose();
            }
        }
    }
    
    spawnEnemy() {
        if (Math.random() < 0.02) { // 2% шанс в секунду
            const enemyGeometry = new THREE.TetrahedronGeometry(0.5, 0);
            const enemyMaterial = new THREE.MeshPhongMaterial({ color: 0xff0000 });
            const enemy = new THREE.Mesh(enemyGeometry, enemyMaterial);
            
            // Случайная позиция по краям
            const side = Math.floor(Math.random() * 4);
            switch (side) {
                case 0: // верх
                    enemy.position.set((Math.random() - 0.5) * 18, 0.5, -9);
                    break;
                case 1: // низ
                    enemy.position.set((Math.random() - 0.5) * 18, 0.5, 9);
                    break;
                case 2: // лево
                    enemy.position.set(-9, 0.5, (Math.random() - 0.5) * 18);
                    break;
                case 3: // право
                    enemy.position.set(9, 0.5, (Math.random() - 0.5) * 18);
                    break;
            }
            
            this.scene.add(enemy);
            this.enemies.push(enemy);
        }
    }
    
    updateEnemies(deltaTime) {
        for (let i = this.enemies.length - 1; i >= 0; i--) {
            const enemy = this.enemies[i];
            
            // Движение к игроку
            const direction = new THREE.Vector3();
            direction.subVectors(this.player.position, enemy.position).normalize();
            enemy.position.add(direction.multiplyScalar(2 * deltaTime));
            
            // Проверка столкновения с игроком
            const distance = this.player.position.distanceTo(enemy.position);
            if (distance < 0.8) {
                // Игрок поражен
                console.log('Игрок поражен!');
                this.scene.remove(enemy);
                this.enemies.splice(i, 1);
                enemy.geometry.dispose();
                enemy.material.dispose();
            }
        }
    }
    
    update(deltaTime) {
        this.updatePlayer(deltaTime);
        this.updateProjectiles(deltaTime);
        this.updateEnemies(deltaTime);
        this.spawnEnemy();
    }
    
    animate() {
        const currentTime = performance.now();
        const deltaTime = currentTime - (this.lastTime || currentTime);
        this.lastTime = currentTime;
        
        this.update(deltaTime / 1000);
        
        this.renderer.render(this.scene, this.camera);
        requestAnimationFrame(() => this.animate());
    }
}

// Использование
// const game = new Game3D('game-container');
// game.animate();
```

## Оптимизация и производительность

### Управление производительностью

```javascript
// Класс для оптимизации Three.js приложений
class PerformanceOptimizer {
    constructor(renderer, scene, camera) {
        this.renderer = renderer;
        this.scene = scene;
        this.camera = camera;
        this.objectPool = new Map(); // Пул объектов для повторного использования
        this.lodGroups = []; // Уровни детализации
    }
    
    // Создание пула объектов
    createObjectPool(type, count, ...args) {
        const pool = [];
        
        for (let i = 0; i < count; i++) {
            const object = this.createObject(type, ...args);
            object.visible = false;
            pool.push(object);
            this.scene.add(object);
        }
        
        this.objectPool.set(type, pool);
        return pool;
    }
    
    createObject(type, ...args) {
        switch (type) {
            case 'box':
                return new THREE.Mesh(
                    new THREE.BoxGeometry(...args),
                    new THREE.MeshBasicMaterial({ color: 0x00ff00 })
                );
            case 'sphere':
                return new THREE.Mesh(
                    new THREE.SphereGeometry(...args),
                    new THREE.MeshBasicMaterial({ color: 0x0000ff })
                );
            default:
                throw new Error(`Неизвестный тип объекта: ${type}`);
        }
    }
    
    // Получение объекта из пула
    getObject(type) {
        const pool = this.objectPool.get(type);
        if (!pool) return null;
        
        for (const obj of pool) {
            if (!obj.visible) {
                obj.visible = true;
                return obj;
            }
        }
        
        // Если все объекты заняты, создаем новый
        const newObject = this.createObject(type, ...arguments);
        pool.push(newObject);
        this.scene.add(newObject);
        newObject.visible = true;
        return newObject;
    }
    
    // Возврат объекта в пул
    returnObject(type, object) {
        object.visible = false;
        object.position.set(0, -1000, 0); // Перемещаем за пределы видимости
    }
    
    // Настройка уровней детализации
    addLOD(object, distances) {
        const lod = new THREE.LOD();
        
        for (let i = 0; i < distances.length; i++) {
            const levelObject = object.clone();
            // Упрощаем геометрию для дальнего расстояния
            if (i > 0) {
                this.simplifyGeometry(levelObject.geometry, Math.pow(0.5, i));
            }
            lod.addLevel(levelObject, distances[i]);
        }
        
        this.scene.add(lod);
        this.lodGroups.push(lod);
        return lod;
    }
    
    simplifyGeometry(geometry, factor) {
        // Упрощение геометрии (упрощенный пример)
        if (geometry.index) {
            const indexCount = geometry.index.count;
            const newCount = Math.floor(indexCount * factor);
            // Упрощение индексов...
        }
    }
    
    // Оптимизация рендеринга
    optimizeRendering() {
        // Отключение теней для повышения производительности
        this.renderer.shadowMap.enabled = false;
        
        // Использование фиксированного размера буфера
        this.renderer.setSize(window.innerWidth, window.innerHeight, false);
        
        // Оптимизация материалов
        this.scene.traverse((object) => {
            if (object.isMesh) {
                if (object.material) {
                    object.material.needsUpdate = true;
                    object.material.side = THREE.FrontSide; // Рендер только передней стороны
                }
            }
        });
    }
    
    // Управление детализацией на основе расстояния
    updateLOD(camera) {
        for (const lod of this.lodGroups) {
            lod.update(camera);
        }
    }
}
```

### Система фрустум-кастинга

```javascript
// Фрустум-кастинг для отсечения невидимых объектов
class FrustumCulling {
    constructor(camera) {
        this.camera = camera;
        this.frustum = new THREE.Frustum();
        this.planes = new Array(6);
        for (let i = 0; i < 6; i++) {
            this.planes[i] = new THREE.Plane();
        }
    }
    
    update() {
        // Обновление матрицы проекции
        this.camera.updateMatrixWorld();
        const projScreenMatrix = new THREE.Matrix4();
        projScreenMatrix.multiplyMatrices(
            this.camera.projectionMatrix,
            this.camera.matrixWorldInverse
        );
        
        // Извлечение плоскостей фрустума
        this.frustum.setFromProjectionMatrix(projScreenMatrix);
    }
    
    isVisible(object) {
        // Проверка видимости объекта
        const sphere = new THREE.Sphere();
        object.geometry.computeBoundingSphere();
        sphere.copy(object.geometry.boundingSphere);
        sphere.applyMatrix4(object.matrixWorld);
        
        return this.frustum.containsPoint(sphere.center) || 
               this.frustum.intersectsSphere(sphere);
    }
    
    cullObjects(objects) {
        this.update();
        return objects.filter(obj => this.isVisible(obj));
    }
}

// Использование
// const culling = new FrustumCulling(camera);
// const visibleObjects = culling.cullObjects(scene.children);
```

WebGL и Three.js предоставляют мощные возможности для создания сложной 3D графики в браузере. Three.js значительно упрощает работу с WebGL, позволяя создавать впечатляющие 3D приложения с меньшими усилиями.

#javascript #webgl #threejs #3d-graphics #frontend #web-development #graphics
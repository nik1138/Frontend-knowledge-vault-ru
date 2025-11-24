---
aliases: [Three.js, Three-js, 3D-графика с Three.js]
tags: [vr, ar, typescript, threejs, 3d-graphics, webgl, vr-rendering]
---

# Three.js для VR-приложений

## Введение в Three.js

Three.js - это мощная библиотека JavaScript для создания и отображения анимированной 3D-графики в браузере с использованием WebGL. Она предоставляет более высокий уровень абстракции по сравнению с низкоуровневым WebGL API, что делает создание 3D-сцен более доступным и продуктивным.

## Основные компоненты Three.js

### Сцена (Scene)

Сцена - это контейнер для всех 3D-объектов, света и камер.

```typescript
import * as THREE from 'three';

// Создание сцены
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xaaaaaa); // Установка цвета фона
scene.fog = new THREE.Fog(0xaaaaaa, 10, 100); // Установка тумана
```

### Камера (Camera)

Камера определяет точку зрения, из которой наблюдается сцена.

```typescript
// Перспективная камера
const camera = new THREE.PerspectiveCamera(
  75,                               // Поле зрения (FOV)
  window.innerWidth / window.innerHeight, // Соотношение сторон
  0.1,                              // Ближняя плоскость отсечения
  1000                              // Дальняя плоскость отсечения
);

// Установка позиции камеры
camera.position.z = 5;
```

### Рендерер (Renderer)

Рендерер отвечает за отображение сцены с помощью камеры.

```typescript
// Создание WebGL рендерера
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

// Активация сглаживания
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
```

## Основные элементы 3D-сцены

### Геометрии и материалы

```typescript
// Создание геометрии
const geometry = new THREE.BoxGeometry(1, 1, 1);

// Создание материала
const material = new THREE.MeshStandardMaterial({
  color: 0x00ff00,
  roughness: 0.5,
  metalness: 0.5
});

// Создание меша (геометрия + материал)
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// Включение теней для объекта
cube.castShadow = true;
cube.receiveShadow = true;
```

### Свет (Light)

```typescript
// Окружающий свет
const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
scene.add(ambientLight);

// Направленный свет
const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
directionalLight.position.set(5, 5, 5);
directionalLight.castShadow = true;
scene.add(directionalLight);

// Настройка теней
directionalLight.shadow.mapSize.width = 1024;
directionalLight.shadow.mapSize.height = 1024;
directionalLight.shadow.camera.near = 0.5;
directionalLight.shadow.camera.far = 50;
```

## Анимация и цикл рендеринга

```typescript
// Функция анимации
function animate() {
  requestAnimationFrame(animate);
  
  // Обновление объектов
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  
  // Рендеринг сцены
  renderer.render(scene, camera);
}

animate();
```

## Загрузка 3D-моделей

### GLTF-модели

```typescript
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

const loader = new GLTFLoader();

loader.load(
  'path/to/model.gltf',
  (gltf) => {
    scene.add(gltf.scene);
    
    // Доступ к анимациям
    if (gltf.animations && gltf.animations.length) {
      const mixer = new THREE.AnimationMixer(gltf.scene);
      const action = mixer.clipAction(gltf.animations[0]);
      action.play();
      
      // Воспроизведение анимаций
      const clock = new THREE.Clock();
      
      function animate() {
        requestAnimationFrame(animate);
        const delta = clock.getDelta();
        mixer.update(delta);
        renderer.render(scene, camera);
      }
      
      animate();
    }
  },
  (progress) => {
    console.log('Загрузка модели:', (progress.loaded / progress.total) * 100 + '%');
  },
  (error) => {
    console.error('Ошибка загрузки модели:', error);
  }
);
```

## Виртуальная реальность с Three.js

### WebXR и VR

```typescript
// Проверка поддержки WebXR
if ('xr' in navigator) {
  // Установка XR сессии
  navigator.xr.requestSession('immersive-vr').then((session) => {
    // Настройка сессии
    session.updateRenderState({ 
      baseLayer: new XRWebGLLayer(session, renderer.getContext()) 
    });
    
    // Получение ссылочной системы
    session.requestReferenceSpace('local').then((refSpace) => {
      // Запуск XR сессии
      renderer.xr.enabled = true;
      renderer.xr.setSession(session);
      
      // Связывание камеры с XR
      camera = new THREE.PerspectiveCamera();
      renderer.xr.getCamera(camera);
      
      // Анимация для XR
      renderer.setAnimationLoop(animate);
    });
  });
} else {
  console.warn('WebXR не поддерживается в этом браузере');
}
```

### VR-контроллеры

```typescript
// Настройка контроллеров
function setupControllers() {
  // Создание контроллеров
  const controller1 = renderer.xr.getController(0);
  scene.add(controller1);
  
  const controller2 = renderer.xr.getController(1);
  scene.add(controller2);
  
  // События контроллеров
  controller1.addEventListener('selectstart', onSelectStart);
  controller1.addEventListener('selectend', onSelectEnd);
  
  controller2.addEventListener('selectstart', onSelectStart);
  controller2.addEventListener('selectend', onSelectEnd);
  
  // Создание визуального представления контроллеров
  const controllerModelFactory = new THREE.XRControllerModelFactory();
  const handModelFactory = new THREE.OculusHandModelFactory();
  
  controller1.addEventListener('connected', (event) => {
    const grip = renderer.xr.getControllerGrip(0);
    grip.add(controllerModelFactory.createControllerModel(event.data));
    scene.add(grip);
  });
  
  controller2.addEventListener('connected', (event) => {
    const grip = renderer.xr.getControllerGrip(1);
    grip.add(controllerModelFactory.createControllerModel(event.data));
    scene.add(grip);
  });
}

function onSelectStart(event: XRInputSourceEvent) {
  console.log('Начало взаимодействия с контроллером');
}

function onSelectEnd(event: XRInputSourceEvent) {
  console.log('Завершение взаимодействия с контроллером');
}
```

## Оптимизация производительности

### Уровни детализации (LOD)

```typescript
// Создание LOD системы
const lod = new THREE.LOD();

// Добавление объектов с разным уровнем детализации
const highDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 32, 32),
  new THREE.MeshStandardMaterial({ color: 0xff0000 })
);

const mediumDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 16, 16),
  new THREE.MeshStandardMaterial({ color: 0xff0000 })
);

const lowDetail = new THREE.Mesh(
  new THREE.SphereGeometry(1, 8, 8),
  new THREE.MeshStandardMaterial({ color: 0xff0000 })
);

lod.addLevel(highDetail, 0);
lod.addLevel(mediumDetail, 20);
lod.addLevel(lowDetail, 50);

scene.add(lod);
```

### Instanced Mesh

```typescript
// Создание инстансированного меша для отрисовки множества одинаковых объектов
const count = 1000;
const geometry = new THREE.BoxGeometry(0.1, 0.1, 0.1);
const material = new THREE.MeshStandardMaterial({ color: 0x00ff00 });

const instancedMesh = new THREE.InstancedMesh(geometry, material, count);
scene.add(instancedMesh);

// Установка позиций для каждого инстанса
const matrix = new THREE.Matrix4();
for (let i = 0; i < count; i++) {
  matrix.setPosition(
    Math.random() * 10 - 5,
    Math.random() * 10 - 5,
    Math.random() * 10 - 5
  );
  instancedMesh.setMatrixAt(i, matrix);
}

instancedMesh.instanceMatrix.needsUpdate = true;
```

## Типизация для Three.js

```typescript
// Определение типов для Three.js компонентов
interface VRSceneComponents {
  scene: THREE.Scene;
  camera: THREE.PerspectiveCamera | THREE.ArrayCamera;
  renderer: THREE.WebGLRenderer;
  controls?: THREE.OrbitControls;
  objects: THREE.Mesh[];
  lights: THREE.Light[];
  vrSession?: XRSession;
  vrRefSpace?: XRReferenceSpace;
}

// Создание типа для VR-сцены
class VRScene {
  private components: VRSceneComponents;
  
  constructor() {
    this.components = {
      scene: new THREE.Scene(),
      camera: new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000),
      renderer: new THREE.WebGLRenderer({ antialias: true }),
      objects: [],
      lights: []
    };
    
    this.init();
  }
  
  private init(): void {
    // Инициализация сцены
    this.components.renderer.setSize(window.innerWidth, window.innerHeight);
    this.components.renderer.setPixelRatio(window.devicePixelRatio);
    document.body.appendChild(this.components.renderer.domElement);
    
    // Добавление света
    const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
    this.components.scene.add(ambientLight);
    
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
    directionalLight.position.set(5, 5, 5);
    this.components.scene.add(directionalLight);
    this.components.lights.push(ambientLight, directionalLight);
  }
  
  public addObject(object: THREE.Object3D): void {
    this.components.scene.add(object);
    if (object instanceof THREE.Mesh) {
      this.components.objects.push(object);
    }
  }
  
  public getScene(): THREE.Scene {
    return this.components.scene;
  }
  
  public getCamera(): THREE.PerspectiveCamera {
    return this.components.camera as THREE.PerspectiveCamera;
  }
  
  public getRenderer(): THREE.WebGLRenderer {
    return this.components.renderer;
  }
}
```

## Работа с текстурами

```typescript
import { TextureLoader } from 'three';

// Загрузка текстур
const textureLoader = new TextureLoader();
const texture = textureLoader.load('path/to/texture.jpg');

// Создание материала с текстурой
const material = new THREE.MeshStandardMaterial({ 
  map: texture,
  roughness: 0.2,
  metalness: 0.8
});

// Повторение текстуры
texture.wrapS = THREE.RepeatWrapping;
texture.wrapT = THREE.RepeatWrapping;
texture.repeat.set(4, 4);
```

## Практический пример: VR-галерея

```typescript
// Пример реализации VR-галереи
class VRGallery {
  private scene: THREE.Scene;
  private camera: THREE.PerspectiveCamera;
  private renderer: THREE.WebGLRenderer;
  private images: string[];
  private currentImageIndex: number = 0;
  private imagePlanes: THREE.Mesh[] = [];
  
  constructor(images: string[]) {
    this.images = images;
    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    
    this.init();
  }
  
  private init(): void {
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(this.renderer.domElement);
    
    // Создание плоскостей для изображений
    this.createImagePlanes();
    
    // Добавление освещения
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
    this.scene.add(ambientLight);
    
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
    directionalLight.position.set(0, 1, 1);
    this.scene.add(directionalLight);
    
    // Запуск анимации
    this.animate();
  }
  
  private createImagePlanes(): void {
    const textureLoader = new THREE.TextureLoader();
    
    this.images.forEach((image, index) => {
      textureLoader.load(image, (texture) => {
        const geometry = new THREE.PlaneGeometry(4, 3);
        const material = new THREE.MeshBasicMaterial({ 
          map: texture,
          side: THREE.DoubleSide
        });
        
        const plane = new THREE.Mesh(geometry, material);
        plane.position.x = index * 5;
        this.scene.add(plane);
        this.imagePlanes.push(plane);
      });
    });
  }
  
  private animate = (): void => {
    requestAnimationFrame(this.animate);
    
    // Анимация плоскостей
    this.imagePlanes.forEach((plane, index) => {
      plane.rotation.y = Date.now() * 0.0005 + index * 0.1;
    });
    
    this.renderer.render(this.scene, this.camera);
  }
}

// Использование VR-галереи
const images = [
  'path/to/image1.jpg',
  'path/to/image2.jpg',
  'path/to/image3.jpg'
];

const gallery = new VRGallery(images);
```

## Заключение

Three.js предоставляет мощную платформу для создания 3D-контента и VR-приложений. Его гибкая архитектура позволяет создавать как простые, так и сложные сцены с высокой производительностью.

Для разработки VR-приложений с использованием Three.js рекомендуется:
- Использовать [[WebXR]] для подключения к VR-устройствам
- Применять оптимизации производительности из [[Оптимизация-рендеринга]]
- Обеспечивать качественное взаимодействие с пользователем через [[Взаимодействие-с-пользователем]]

## Ключевые теги
#vr #ar #typescript #threejs #3d-graphics #webgl #vr-rendering #virtual-reality #augmented-reality
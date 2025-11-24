---
aliases: [WebGL-анимации, WebGL, 3D-анимации, Three.js, 3D-графика]
tags: [typescript, javascript, анимации, веб-разработка, 3d, webgl, threejs]
---

# WebGL-анимации в TypeScript проектах

WebGL-анимации позволяют создавать сложные 3D-эффекты и анимации непосредственно в браузере с использованием GPU. TypeScript обеспечивает строгую типизацию при работе с WebGL и 3D-библиотеками, что делает разработку более безопасной и предсказуемой.

## Основы WebGL-анимаций

WebGL (Web Graphics Library) — это JavaScript API для рендеринга сложной 2D и 3D-графики без использования плагинов. Для анимаций WebGL использует `requestAnimationFrame` для плавного обновления кадров.

### Простой пример WebGL-анимации

```typescript
interface WebGLAnimationState {
  gl: WebGLRenderingContext;
  program: WebGLProgram;
  positionBuffer: WebGLBuffer;
  colorBuffer: WebGLBuffer;
  vertexCount: number;
  rotation: number;
  animationId: number | null;
}

class BasicWebGLAnimation {
  private canvas: HTMLCanvasElement;
  private state: WebGLAnimationState | null = null;

  constructor(canvasId: string) {
    const canvas = document.getElementById(canvasId) as HTMLCanvasElement;
    if (!canvas) {
      throw new Error(`Canvas с id ${canvasId} не найден`);
    }
    
    this.canvas = canvas;
  }

  public async init(): Promise<void> {
    const gl = this.canvas.getContext('webgl');
    if (!gl) {
      throw new Error('WebGL не поддерживается');
    }

    // Создание шейдера
    const vertexShader = this.createShader(gl, gl.VERTEX_SHADER, this.getVertexShaderSource());
    const fragmentShader = this.createShader(gl, gl.FRAGMENT_SHADER, this.getFragmentShaderSource());
    
    if (!vertexShader || !fragmentShader) {
      throw new Error('Не удалось создать шейдеры');
    }

    // Создание программы
    const program = this.createProgram(gl, vertexShader, fragmentShader);
    if (!program) {
      throw new Error('Не удалось создать программу');
    }

    // Установка атрибутов
    const positionAttributeLocation = gl.getAttribLocation(program, 'a_position');
    const colorAttributeLocation = gl.getAttribLocation(program, 'a_color');
    
    // Создание буферов
    const positionBuffer = gl.createBuffer();
    const colorBuffer = gl.createBuffer();
    
    if (!positionBuffer || !colorBuffer) {
      throw new Error('Не удалось создать буферы');
    }

    // Установка данных для треугольника
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
      0, 0.5,   // вершина 1
      -0.5, -0.5, // вершина 2
      0.5, -0.5   // вершина 3
    ]), gl.STATIC_DRAW);

    gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
      1, 0, 0, 1,  // красный
      0, 1, 0, 1,  // зеленый
      0, 0, 1, 1   // синий
    ]), gl.STATIC_DRAW);

    this.state = {
      gl,
      program,
      positionBuffer,
      colorBuffer,
      vertexCount: 3,
      rotation: 0,
      animationId: null
    };
  }

  private createShader(gl: WebGLRenderingContext, type: number, source: string): WebGLShader | null {
    const shader = gl.createShader(type);
    if (!shader) return null;

    gl.shaderSource(shader, source);
    gl.compileShader(shader);

    const success = gl.getShaderParameter(shader, gl.COMPILE_STATUS);
    if (!success) {
      console.error(gl.getShaderInfoLog(shader));
      gl.deleteShader(shader);
      return null;
    }

    return shader;
  }

  private createProgram(gl: WebGLRenderingContext, vertexShader: WebGLShader, fragmentShader: WebGLShader): WebGLProgram | null {
    const program = gl.createProgram();
    if (!program) return null;

    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);
    gl.linkProgram(program);

    const success = gl.getProgramParameter(program, gl.LINK_STATUS);
    if (!success) {
      console.error(gl.getProgramInfoLog(program));
      gl.deleteProgram(program);
      return null;
    }

    return program;
  }

  private getVertexShaderSource(): string {
    return `
      attribute vec2 a_position;
      attribute vec4 a_color;
      uniform mat4 u_matrix;
      varying vec4 v_color;
      
      void main() {
        gl_Position = u_matrix * vec4(a_position, 0, 1);
        v_color = a_color;
      }
    `;
  }

  private getFragmentShaderSource(): string {
    return `
      precision mediump float;
      varying vec4 v_color;
      
      void main() {
        gl_FragColor = v_color;
      }
    `;
  }

  public startAnimation(): void {
    if (!this.state) {
      throw new Error('Анимация не инициализирована');
    }

    const animate = (time: number) => {
      this.render(time);
      this.state.animationId = requestAnimationFrame(animate);
    };

    this.state.animationId = requestAnimationFrame(animate);
  }

  private render(time: number): void {
    if (!this.state) return;

    const { gl, program, positionBuffer, colorBuffer, vertexCount } = this.state;
    
    // Очистка холста
    gl.clearColor(0, 0, 0, 0);
    gl.clear(gl.COLOR_BUFFER_BIT);

    // Использование программы
    gl.useProgram(program);

    // Установка атрибута позиции
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
    const positionAttributeLocation = gl.getAttribLocation(program, 'a_position');
    gl.enableVertexAttribArray(positionAttributeLocation);
    gl.vertexAttribPointer(positionAttributeLocation, 2, gl.FLOAT, false, 0, 0);

    // Установка атрибута цвета
    gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer);
    const colorAttributeLocation = gl.getAttribLocation(program, 'a_color');
    gl.enableVertexAttribArray(colorAttributeLocation);
    gl.vertexAttribPointer(colorAttributeLocation, 4, gl.FLOAT, false, 0, 0);

    // Установка матрицы трансформации
    const matrixLocation = gl.getUniformLocation(program, 'u_matrix');
    if (matrixLocation) {
      // Создание матрицы вращения
      const rotation = time * 0.001; // вращение со временем
      const s = Math.sin(rotation);
      const c = Math.cos(rotation);
      
      const matrix = new Float32Array([
        c, -s, 0, 0,
        s, c, 0, 0,
        0, 0, 1, 0,
        0, 0, 0, 1
      ]);
      
      gl.uniformMatrix4fv(matrixLocation, false, matrix);
    }

    // Рендеринг
    gl.drawArrays(gl.TRIANGLES, 0, vertexCount);
  }

  public stopAnimation(): void {
    if (this.state && this.state.animationId) {
      cancelAnimationFrame(this.state.animationId);
      this.state.animationId = null;
    }
  }
}

// Использование
const webglAnimation = new BasicWebGLAnimation('webgl-canvas');
webglAnimation.init().then(() => {
  webglAnimation.startAnimation();
});
```

## Three.js с TypeScript

Three.js — популярная библиотека для 3D-графики на WebGL. TypeScript обеспечивает отличную типизацию при работе с Three.js.

Установка зависимостей:
```bash
npm install three
npm install @types/three --save-dev
```

### Простая 3D-анимация с Three.js

```typescript
import * as THREE from 'three';

interface ThreeJSAnimationConfig {
  containerId: string;
  sceneColor?: string;
  cameraPosition?: THREE.Vector3;
  rendererOptions?: Partial<THREE.WebGLRendererParameters>;
}

class ThreeJSAnimation {
  private scene: THREE.Scene;
  private camera: THREE.PerspectiveCamera;
  private renderer: THREE.WebGLRenderer;
  private container: HTMLElement;
  private cube: THREE.Mesh;
  private animationId: number | null = null;

  constructor(private config: ThreeJSAnimationConfig) {
    this.container = document.getElementById(config.containerId) as HTMLElement;
    if (!this.container) {
      throw new Error(`Контейнер с id ${config.containerId} не найден`);
    }

    // Создание сцены
    this.scene = new THREE.Scene();
    this.scene.background = new THREE.Color(config.sceneColor || 0x222222);

    // Создание камеры
    this.camera = new THREE.PerspectiveCamera(
      75,
      this.container.clientWidth / this.container.clientHeight,
      0.1,
      1000
    );
    this.camera.position.copy(config.cameraPosition || new THREE.Vector3(0, 0, 5));

    // Создание рендера
    this.renderer = new THREE.WebGLRenderer(config.rendererOptions);
    this.renderer.setSize(this.container.clientWidth, this.container.clientHeight);
    this.container.appendChild(this.renderer.domElement);

    // Создание куба
    const geometry = new THREE.BoxGeometry();
    const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
    this.cube = new THREE.Mesh(geometry, material);
    this.scene.add(this.cube);

    // Обработка изменения размера окна
    window.addEventListener('resize', this.onWindowResize.bind(this), false);
  }

  private onWindowResize(): void {
    if (!this.container) return;

    this.camera.aspect = this.container.clientWidth / this.container.clientHeight;
    this.camera.updateProjectionMatrix();
    this.renderer.setSize(this.container.clientWidth, this.container.clientHeight);
  }

  public startAnimation(): void {
    const animate = () => {
      this.animationId = requestAnimationFrame(animate);

      // Анимация куба
      this.cube.rotation.x += 0.01;
      this.cube.rotation.y += 0.01;

      this.renderer.render(this.scene, this.camera);
    };

    animate();
  }

  public stopAnimation(): void {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId);
      this.animationId = null;
    }
  }

  public addLight(): void {
    const light = new THREE.AmbientLight(0x404040); // мягкий свет
    this.scene.add(light);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
    directionalLight.position.set(1, 1, 1);
    this.scene.add(directionalLight);
  }

  public addSphere(): THREE.Mesh {
    const geometry = new THREE.SphereGeometry(0.5, 32, 32);
    const material = new THREE.MeshPhongMaterial({ color: 0xff0000 });
    const sphere = new THREE.Mesh(geometry, material);
    sphere.position.x = 2;
    this.scene.add(sphere);
    return sphere;
  }
}

// Использование
const threeAnimation = new ThreeJSAnimation({
  containerId: 'three-container',
  sceneColor: 0x333333
});

threeAnimation.addLight();
threeAnimation.addSphere();
threeAnimation.startAnimation();
```

## Анимации с загрузкой 3D-моделей

```typescript
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

class ModelAnimation {
  private scene: THREE.Scene;
  private camera: THREE.PerspectiveCamera;
  private renderer: THREE.WebGLRenderer;
  private container: HTMLElement;
  private mixer: THREE.AnimationMixer | null = null;
  private clock: THREE.Clock;
  private animationId: number | null = null;

  constructor(containerId: string) {
    this.container = document.getElementById(containerId) as HTMLElement;
    if (!this.container) {
      throw new Error(`Контейнер с id ${containerId} не найден`);
    }

    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    this.camera.position.set(0, 1, 5);
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.container.appendChild(this.renderer.domElement);

    this.clock = new THREE.Clock();

    // Добавление освещения
    const ambientLight = new THREE.AmbientLight(0x404040, 2);
    this.scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
    directionalLight.position.set(1, 1, 1);
    this.scene.add(directionalLight);

    window.addEventListener('resize', this.onWindowResize.bind(this));
  }

  private onWindowResize(): void {
    this.camera.aspect = window.innerWidth / window.innerHeight;
    this.camera.updateProjectionMatrix();
    this.renderer.setSize(window.innerWidth, window.innerHeight);
  }

  public async loadModel(modelPath: string): Promise<void> {
    return new Promise((resolve, reject) => {
      const loader = new GLTFLoader();
      loader.load(
        modelPath,
        (gltf) => {
          // Добавление модели на сцену
          this.scene.add(gltf.scene);

          // Поиск и настройка анимаций
          if (gltf.animations && gltf.animations.length > 0) {
            this.mixer = new THREE.AnimationMixer(gltf.scene);
            const action = this.mixer.clipAction(gltf.animations[0]);
            action.play();
          }

          resolve();
        },
        undefined,
        (error) => {
          console.error('Ошибка загрузки модели:', error);
          reject(error);
        }
      );
    });
  }

  public startAnimation(): void {
    const animate = () => {
      this.animationId = requestAnimationFrame(animate);

      const delta = this.clock.getDelta();
      if (this.mixer) {
        this.mixer.update(delta);
      }

      this.renderer.render(this.scene, this.camera);
    };

    animate();
  }

  public stopAnimation(): void {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId);
      this.animationId = null;
    }
  }
}
```

## Типизированные анимационные системы

Создадим типизированную систему для управления WebGL-анимациями:

```typescript
import * as THREE from 'three';

type AnimationType = 'rotation' | 'translation' | 'scale' | 'color' | 'custom';
type EasingFunction = (t: number) => number;

interface BaseAnimation {
  type: AnimationType;
  duration: number;
  delay?: number;
  easing?: EasingFunction;
  onStart?: () => void;
  onUpdate?: (progress: number) => void;
  onComplete?: () => void;
}

interface RotationAnimation extends BaseAnimation {
  type: 'rotation';
  axis: 'x' | 'y' | 'z';
  from: number;
  to: number;
}

interface TranslationAnimation extends BaseAnimation {
  type: 'translation';
  from: THREE.Vector3;
  to: THREE.Vector3;
}

interface ScaleAnimation extends BaseAnimation {
  type: 'scale';
  from: number;
  to: number;
}

interface ColorAnimation extends BaseAnimation {
  type: 'color';
  from: THREE.Color;
  to: THREE.Color;
}

interface CustomAnimation extends BaseAnimation {
  type: 'custom';
  update: (object: THREE.Object3D, progress: number) => void;
}

type Animation = 
  | RotationAnimation 
  | TranslationAnimation 
  | ScaleAnimation 
  | ColorAnimation 
  | CustomAnimation;

class TypedAnimationSystem {
  private animations: Animation[] = [];
  private activeAnimations: Array<{
    animation: Animation;
    object: THREE.Object3D;
    startTime: number;
    isActive: boolean;
  }> = [];
  private clock: THREE.Clock = new THREE.Clock();

  public addAnimation(object: THREE.Object3D, animation: Animation): void {
    this.animations.push(animation);
    // Анимация начнется при вызове start()
  }

  public startAnimation(object: THREE.Object3D, animation: Animation): void {
    this.activeAnimations.push({
      animation,
      object,
      startTime: performance.now() + (animation.delay || 0),
      isActive: true
    });

    if (animation.onStart) {
      animation.onStart();
    }
  }

  public startAll(object: THREE.Object3D): void {
    this.animations.forEach(anim => this.startAnimation(object, anim));
  }

  public update(): void {
    const delta = this.clock.getDelta();
    const currentTime = performance.now();

    this.activeAnimations = this.activeAnimations.filter(animData => {
      if (!animData.isActive) return false;

      const elapsed = currentTime - animData.startTime;
      if (elapsed < 0) return true; // Анимация еще не начала

      const progress = Math.min(elapsed / animData.animation.duration, 1);
      const easing = animData.animation.easing || this.linearEasing;
      const easedProgress = easing(progress);

      // Применение анимации в зависимости от типа
      this.applyAnimation(animData.object, animData.animation, easedProgress);

      if (animData.animation.onUpdate) {
        animData.animation.onUpdate(progress);
      }

      if (progress >= 1) {
        if (animData.animation.onComplete) {
          animData.animation.onComplete();
        }
        return false; // Удалить из активных
      }

      return true;
    });
  }

  private applyAnimation(object: THREE.Object3D, animation: Animation, progress: number): void {
    switch (animation.type) {
      case 'rotation':
        const rotationValue = animation.from + (animation.to - animation.from) * progress;
        switch (animation.axis) {
          case 'x': object.rotation.x = rotationValue; break;
          case 'y': object.rotation.y = rotationValue; break;
          case 'z': object.rotation.z = rotationValue; break;
        }
        break;

      case 'translation':
        object.position.lerpVectors(
          animation.from,
          animation.to,
          progress
        );
        break;

      case 'scale':
        const scaleValue = animation.from + (animation.to - animation.from) * progress;
        object.scale.setScalar(scaleValue);
        break;

      case 'color':
        // Для анимации цвета нужно использовать материал объекта
        if (object instanceof THREE.Mesh) {
          const mesh = object as THREE.Mesh;
          if (mesh.material instanceof THREE.MeshBasicMaterial) {
            const material = mesh.material as THREE.MeshBasicMaterial;
            material.color.lerpColors(
              animation.from,
              animation.to,
              progress
            );
          }
        }
        break;

      case 'custom':
        animation.update(object, progress);
        break;
    }
  }

  private linearEasing(t: number): number {
    return t;
  }

  private easeInOutCubic(t: number): number {
    return t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1;
  }
}

// Использование типизированной системы анимаций
const animationSystem = new TypedAnimationSystem();

// Создание объекта для анимации
const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
const cube = new THREE.Mesh(geometry, material);

// Добавление анимаций
const rotationAnim: RotationAnimation = {
  type: 'rotation',
  axis: 'y',
  from: 0,
  to: Math.PI * 2,
  duration: 2000,
  easing: animationSystem['easeInOutCubic'].bind(animationSystem),
  onComplete: () => console.log('Вращение завершено')
};

const translationAnim: TranslationAnimation = {
  type: 'translation',
  from: new THREE.Vector3(0, 0, 0),
  to: new THREE.Vector3(2, 2, 0),
  duration: 2000,
  onComplete: () => console.log('Перемещение завершено')
};

animationSystem.startAnimation(cube, rotationAnim);
animationSystem.startAnimation(cube, translationAnim);

// В цикле рендеринга
function animate() {
  animationSystem.update();
  requestAnimationFrame(animate);
}
animate();
```

## Анимации с шейдерами

WebGL позволяет создавать сложные анимации с использованием шейдеров:

```typescript
import * as THREE from 'three';

class ShaderAnimation {
  private scene: THREE.Scene;
  private camera: THREE.PerspectiveCamera;
  private renderer: THREE.WebGLRenderer;
  private material: THREE.ShaderMaterial;
  private mesh: THREE.Mesh;
  private container: HTMLElement;

  constructor(containerId: string) {
    this.container = document.getElementById(containerId) as HTMLElement;
    if (!this.container) {
      throw new Error(`Контейнер с id ${containerId} не найден`);
    }

    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    this.camera.position.z = 5;
    this.renderer = new THREE.WebGLRenderer();
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.container.appendChild(this.renderer.domElement);

    // Создание шейдерного материала
    this.material = new THREE.ShaderMaterial({
      uniforms: {
        time: { value: 0 },
        resolution: { value: new THREE.Vector2(window.innerWidth, window.innerHeight) }
      },
      vertexShader: `
        varying vec2 vUv;
        
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `,
      fragmentShader: `
        uniform float time;
        uniform vec2 resolution;
        varying vec2 vUv;
        
        void main() {
          vec2 uv = (vUv - 0.5) * 2.0;
          
          // Анимация с использованием времени
          float r = sin(uv.x * 10.0 + time) * 0.5 + 0.5;
          float g = sin(uv.y * 10.0 + time * 1.2) * 0.5 + 0.5;
          float b = sin((uv.x + uv.y) * 5.0 + time * 0.8) * 0.5 + 0.5;
          
          gl_FragColor = vec4(r, g, b, 1.0);
        }
      `
    });

    // Создание плоскости с шейдерным материалом
    const geometry = new THREE.PlaneGeometry(5, 5);
    this.mesh = new THREE.Mesh(geometry, this.material);
    this.scene.add(this.mesh);

    window.addEventListener('resize', this.onWindowResize.bind(this));
  }

  private onWindowResize(): void {
    this.camera.aspect = window.innerWidth / window.innerHeight;
    this.camera.updateProjectionMatrix();
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.material.uniforms.resolution.value.set(window.innerWidth, window.innerHeight);
  }

  public startAnimation(): void {
    const animate = (timestamp: number) => {
      // Обновление времени для шейдера
      this.material.uniforms.time.value = timestamp / 1000;

      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(animate);
    };

    animate(0);
  }
}
```

## Практические примеры

### Анимация частиц

```typescript
import * as THREE from 'three';

class ParticleSystem {
  private scene: THREE.Scene;
  private particles: THREE.BufferGeometry;
  private material: THREE.PointsMaterial;
  private particleSystem: THREE.Points;
  private positions: Float32Array;
  private velocities: Array<THREE.Vector3>;

  constructor(count: number = 1000) {
    this.scene = new THREE.Scene();
    
    // Создание позиций частиц
    this.positions = new Float32Array(count * 3);
    this.velocities = [];
    
    for (let i = 0; i < count; i++) {
      this.positions[i * 3] = (Math.random() - 0.5) * 10;
      this.positions[i * 3 + 1] = (Math.random() - 0.5) * 10;
      this.positions[i * 3 + 2] = (Math.random() - 0.5) * 10;
      
      this.velocities.push(new THREE.Vector3(
        (Math.random() - 0.5) * 0.02,
        (Math.random() - 0.5) * 0.02,
        (Math.random() - 0.5) * 0.02
      ));
    }

    this.particles = new THREE.BufferGeometry();
    this.particles.setAttribute('position', new THREE.BufferAttribute(this.positions, 3));

    this.material = new THREE.PointsMaterial({
      color: 0xffffff,
      size: 0.05,
      transparent: true,
      opacity: 0.8
    });

    this.particleSystem = new THREE.Points(this.particles, this.material);
    this.scene.add(this.particleSystem);
  }

  public update(): void {
    for (let i = 0; i < this.velocities.length; i++) {
      // Обновление позиций частиц
      this.positions[i * 3] += this.velocities[i].x;
      this.positions[i * 3 + 1] += this.velocities[i].y;
      this.positions[i * 3 + 2] += this.velocities[i].z;

      // Простая физика для возвращения частиц в центр
      const x = this.positions[i * 3];
      const y = this.positions[i * 3 + 1];
      const z = this.positions[i * 3 + 2];

      if (Math.abs(x) > 5) this.velocities[i].x *= -1;
      if (Math.abs(y) > 5) this.velocities[i].y *= -1;
      if (Math.abs(z) > 5) this.velocities[i].z *= -1;
    }

    this.particles.attributes.position.needsUpdate = true;
  }

  public getScene(): THREE.Scene {
    return this.scene;
  }
}
```

## Заключение

WebGL-анимации открывают широкие возможности для создания сложной 3D-графики и визуальных эффектов в веб-приложениях. TypeScript обеспечивает строгую типизацию при работе с WebGL и 3D-библиотеками, что делает разработку более безопасной и предсказуемой. Использование библиотек, таких как Three.js, значительно упрощает работу с WebGL и позволяет сосредоточиться на создании анимаций, а не на низкоуровневых деталях.

## См. также

- [[CSS-анимации]]
- [[JavaScript-анимации]]
- [[GSAP]]
- [[Framer-Motion]]
- [[Canvas-анимации]]
- [[Three.js]]
- [[WebGL основы]]
- [[Шейдеры в WebGL]]
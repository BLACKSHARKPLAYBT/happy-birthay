<template>
  <div class="scene-container" ref="sceneContainer"></div>
</template>

<script setup lang="ts">
import { onMounted, ref, onUnmounted } from 'vue';
import * as THREE from 'three';
import { ShaderMaterial } from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import gsap from 'gsap';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass.js';
import { ShaderPass } from 'three/examples/jsm/postprocessing/ShaderPass.js';

const sceneContainer = ref<HTMLElement | null>(null);

// --- Three.js 变量声明 ---
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let controls: OrbitControls;
let particles: THREE.Points;
let mixer: THREE.AnimationMixer | undefined;
let animationFrameId: number;
let composer: EffectComposer;
let outlinePass: OutlinePass;

// --- 卡通着色器定义 ---
const createToonMaterial = (color: number | string, darkColor: number | string) => {
  const vertexShader = `
    varying vec3 vNormal;
    varying vec3 vPosition;
    
    void main() {
      vNormal = normalize(normalMatrix * normal);
      vPosition = (modelViewMatrix * vec4(position, 1.0)).xyz;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `;
  
  const fragmentShader = `
    uniform vec3 color;
    uniform vec3 darkColor;
    varying vec3 vNormal;
    varying vec3 vPosition;
    
    void main() {
      vec3 lightDir = normalize(vec3(1.0, 1.0, 1.0));
      float intensity = max(0.0, dot(vNormal, lightDir));
      
      float toonIntensity;
      if (intensity > 0.9) toonIntensity = 1.0;
      else if (intensity > 0.6) toonIntensity = 0.8;
      else if (intensity > 0.3) toonIntensity = 0.6;
      else toonIntensity = 0.4;
      
      float edgeFactor = max(0.0, dot(normalize(-vPosition), vNormal));
      float edge = (edgeFactor < 0.3) ? 0.0 : 1.0;
      
      vec3 finalColor = mix(darkColor, color, toonIntensity * edge);
      gl_FragColor = vec4(finalColor, 1.0);
    }
  `;
  
  return new ShaderMaterial({
    uniforms: {
      color: { value: new THREE.Color(color) },
      darkColor: { value: new THREE.Color(darkColor) }
    },
    vertexShader: vertexShader,
    fragmentShader: fragmentShader,
    side: THREE.DoubleSide
  });
};

// --- Three.js 场景初始化 ---
const initScene = () => {
  try {
    if (!sceneContainer.value) return;

    scene = new THREE.Scene();
    scene.background = new THREE.Color(0x0a192f); // 更深邃的深蓝色背景

    camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 4, 8);

    renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    sceneContainer.value.appendChild(renderer.domElement);

    // 添加光源
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
    scene.add(ambientLight);
    const mainLight = new THREE.DirectionalLight(0xffffff, 1.0);
    mainLight.position.set(5, 10, 5);
    scene.add(mainLight);
    const fillLight = new THREE.DirectionalLight(0xffffff, 0.4);
    fillLight.position.set(-5, 5, -5);
    scene.add(fillLight);

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.enableRotate = false;
    controls.enableZoom = false;
    controls.enablePan = false;
    controls.minPolarAngle = Math.PI / 2.5;
    controls.maxPolarAngle = Math.PI / 1.8;
    controls.minAzimuthAngle = -Math.PI / 4;
    controls.maxAzimuthAngle = Math.PI / 4;

    // 后期处理
    composer = new EffectComposer(renderer);
    const renderPass = new RenderPass(scene, camera);
    composer.addPass(renderPass);

    outlinePass = new OutlinePass(
      new THREE.Vector2(window.innerWidth, window.innerHeight),
      scene,
      camera
    );
    outlinePass.edgeStrength = 3.0;
    outlinePass.edgeGlow = 0.0;
    outlinePass.edgeThickness = 1.0;
    outlinePass.pulsePeriod = 0;
    outlinePass.visibleEdgeColor.set(0x000000);
    outlinePass.hiddenEdgeColor.set(0x000000);
    composer.addPass(outlinePass);

    const toonShader = {
      uniforms: {
        tDiffuse: { value: null },
        numColors: { value: 4.0 }
      },
      vertexShader: `
        varying vec2 vUv;
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `,
      fragmentShader: `
        uniform sampler2D tDiffuse;
        uniform float numColors;
        varying vec2 vUv;
        
        void main() {
          vec4 texel = texture2D(tDiffuse, vUv);
          vec3 color = texel.rgb;
          color = floor(color * numColors) / numColors;
          gl_FragColor = vec4(color, texel.a);
        }
      `
    };
    const toonPass = new ShaderPass(toonShader);
    composer.addPass(toonPass);

    createBirthdayScene();
    createParticles();
    animate();

    window.addEventListener('resize', onWindowResize);

  } catch (error) {
    console.error('初始化Three.js场景时出错:', error);
  }
};

// --- 创建生日场景 ---
const createBirthdayScene = () => {
  const cakeGroup = new THREE.Group();
  scene.add(cakeGroup);

  // 蛋糕底座
  const cakeBasePoints = [];
  const cakeBaseSegments = 64;
  const cakeBaseRadiusOuter = 2.5;
  const cakeBaseRadiusInner = 2.3; // <-- 添加这行定义
  const waveAmplitudeBase = 0.1;
  const waveFrequencyBase = 5.0;
  const baseHeight = 0.5;

  for (let i = 0; i <= cakeBaseSegments; i++) {
    const theta = (i / cakeBaseSegments) * Math.PI * 2;
    const radiusOuter = cakeBaseRadiusOuter + Math.sin(theta * waveFrequencyBase) * waveAmplitudeBase;
    cakeBasePoints.push(new THREE.Vector2(Math.cos(theta) * radiusOuter, Math.sin(theta) * radiusOuter));
  }
  const cakeBaseShape = new THREE.Shape(cakeBasePoints);
  const cakeBaseGeometry = new THREE.ExtrudeGeometry(cakeBaseShape, { depth: 0.5, bevelEnabled: true, bevelThickness: 0.1, bevelSize: 0.1, bevelSegments: 3 });
  const cakeBaseMaterial = createToonMaterial(0xffefd5, 0xdeb887);
  const cakeBase = new THREE.Mesh(cakeBaseGeometry, cakeBaseMaterial);
  cakeBase.rotation.x = Math.PI / 2;
  cakeBase.position.y = 0.25;
  cakeGroup.add(cakeBase);

  // 蛋糕主体
  const cakeMainPoints = [];
  const cakeMainSegments = 64;
  const cakeMainRadius = 2.0;
  const waveAmplitudeMain = 0.08;
  const waveFrequencyMain = 12;
  for (let i = 0; i <= cakeMainSegments; i++) {
    const theta = (i / cakeMainSegments) * Math.PI * 2;
    const radius = cakeMainRadius + Math.sin(theta * waveFrequencyMain) * waveAmplitudeMain;
    cakeMainPoints.push(new THREE.Vector2(Math.cos(theta) * radius, Math.sin(theta) * radius));
  }
  const cakeMainShape = new THREE.Shape(cakeMainPoints);
  const cakeMainGeometry = new THREE.ExtrudeGeometry(cakeMainShape, { depth: 1.2, bevelEnabled: true, bevelThickness: 0.05, bevelSize: 0.05, bevelSegments: 3 });
  const cakeMainMaterial = createToonMaterial(0xffc0cb, 0xff9eaa); // 淡粉色系
  const cakeMain = new THREE.Mesh(cakeMainGeometry, cakeMainMaterial);
  cakeMain.rotation.x = Math.PI / 2;
  cakeMain.position.y = 1.1;
  cakeGroup.add(cakeMain);

  // 蛋糕顶层
  const cakeTopPoints = [];
  const cakeTopSegments = 64;
  const cakeTopRadius = 1.5;
  const petalCount = 8;
  const petalDepth = 0.15;
  for (let i = 0; i <= cakeTopSegments; i++) {
    const theta = (i / cakeTopSegments) * Math.PI * 2;
    const radiusTop = cakeTopRadius + Math.sin(theta * petalCount) * petalDepth;
    cakeTopPoints.push(new THREE.Vector2(Math.cos(theta) * radiusTop, Math.sin(theta) * radiusTop));
  }
  const cakeTopShape = new THREE.Shape(cakeTopPoints);
  const cakeTopGeometry = new THREE.ExtrudeGeometry(cakeTopShape, { depth: 0.8, bevelEnabled: true, bevelThickness: 0.05, bevelSize: 0.05, bevelSegments: 3 });
  const cakeTopMaterial = createToonMaterial(0xfff0f5, 0xffdab9); // 更柔和的奶油色
  const cakeTop = new THREE.Mesh(cakeTopGeometry, cakeTopMaterial);
  cakeTop.rotation.x = Math.PI / 2;
  cakeTop.position.y = 2.1;
  cakeGroup.add(cakeTop);

  // 巧克力边缘
  const chocolateGeometry = new THREE.TorusGeometry(1.5, 0.1, 16, 32);
  const chocolateMaterial = createToonMaterial(0x8b4513, 0x654321); // 保持原有的巧克力色
  const chocolate = new THREE.Mesh(chocolateGeometry, chocolateMaterial);
  chocolate.position.y = 2.5;
  chocolate.rotation.x = Math.PI / 2;
  cakeGroup.add(chocolate);

  // 蜡烛和火焰
  const candleCount = 8;
  const candleGeometry = new THREE.CylinderGeometry(0.08, 0.08, 0.8, 16);
  const candleMaterial = createToonMaterial(0xffffff, 0xf0f0f0);
  const flameGeometry = new THREE.SphereGeometry(0.12, 16, 16);
  const flameMaterial = new THREE.MeshPhongMaterial({ color: 0xff9900, emissive: 0xff6600, emissiveIntensity: 2, transparent: true, opacity: 0.9 });

  for (let i = 0; i < candleCount; i++) {
    const angle = (i / candleCount) * Math.PI * 2;
    const radius = 1.2;
    const candle = new THREE.Mesh(candleGeometry, candleMaterial);
    candle.position.set(Math.cos(angle) * radius, 2.6, Math.sin(angle) * radius);
    candle.rotation.x = (Math.random() - 0.5) * 0.2;
    candle.rotation.z = (Math.random() - 0.5) * 0.2;
    cakeGroup.add(candle);

    const flame = new THREE.Mesh(flameGeometry, flameMaterial.clone()); // Clone material for individual control if needed
    flame.position.y = 0.5;
    flame.scale.y = 1.5;
    candle.add(flame);

    gsap.to(flame.scale, { y: 1.2, x: () => 0.8 + Math.random() * 0.4, z: () => 0.8 + Math.random() * 0.4, duration: () => 0.5 + Math.random() * 0.5, repeat: -1, yoyo: true, ease: "sine.inOut" });
    gsap.to(flame.rotation, { y: Math.PI * 2, duration: () => 3 + Math.random() * 2, repeat: -1, ease: "none" });
  }

  // 糖果装饰
  const decorationCount = 150; // 增加糖果数量
  const decorGeometry = new THREE.SphereGeometry(0.05, 8, 8); // 糖果大小
  const decorMaterials = [
    new THREE.MeshStandardMaterial({ color: 0xff69b4 }), // Pink
    new THREE.MeshStandardMaterial({ color: 0xadd8e6 }), // Light Blue
    new THREE.MeshStandardMaterial({ color: 0xffffe0 }), // Light Yellow
    new THREE.MeshStandardMaterial({ color: 0x90ee90 }), // Light Green
    new THREE.MeshStandardMaterial({ color: 0xffb6c1 }), // Light Pink
  ];

  for (let i = 0; i < decorationCount; i++) {
    const decorMesh = new THREE.Mesh(decorGeometry, decorMaterials[Math.floor(Math.random() * decorMaterials.length)]);
    const angle = Math.random() * Math.PI * 2;
    let radius;
    let yPos;
    const decorSize = 0.05; // 糖果半径，用于计算边界

    // 随机决定装饰在哪一层 (0: 底座, 1: 蛋糕体)
    const layerChoice = Math.random() < 0.4 ? 0 : 1; // 40% 概率在底座

    if (layerChoice === 0) { // Base layer
      yPos = 0.5 + (Math.random() - 0.5) * 0.1; // 在底座高度附近随机
      radius = cakeBaseRadiusInner + Math.random() * (cakeBaseRadiusOuter - cakeBaseRadiusInner - decorSize);
    } else { // Cake body layer
      // 使用实际定义的蛋糕主体半径和挤出深度 (高度)
      const cakeRadius = cakeMainRadius; // 使用 cakeMainRadius (值为 2.0)
      const cakeHeight = 1.2; // 使用 cakeMainGeometry 的 depth (值为 1.2)
      yPos = baseHeight + Math.random() * cakeHeight; // 在蛋糕体的高度范围内 (从 baseHeight 开始)
      radius = Math.random() * (cakeRadius - decorSize);
    }

    decorMesh.position.set(
      Math.cos(angle) * radius,
      yPos,
      Math.sin(angle) * radius
    );
    // 随机旋转
    decorMesh.rotation.set(Math.random() * Math.PI, Math.random() * Math.PI, Math.random() * Math.PI);
    cakeGroup.add(decorMesh);
  }

  // 蛋糕整体旋转动画
  gsap.to(cakeGroup.rotation, {
    y: Math.PI * 2,
    duration: 40, // 减慢旋转速度
    repeat: -1,
    ease: "none"
  });
};

// --- 创建粒子 ---
const createParticles = () => {
  const particleCount = 200; // 减少粒子数量
  const particlesGeometry = new THREE.BufferGeometry();
  const positions = new Float32Array(particleCount * 3);
  const velocities = new Float32Array(particleCount * 3);
  const sizes = new Float32Array(particleCount);

  for (let i = 0; i < particleCount; i++) {
    const i3 = i * 3;
    // 随机位置
    positions[i3] = (Math.random() - 0.5) * 100;
    positions[i3 + 1] = Math.random() * 50 + 25; // 高度范围
    positions[i3 + 2] = (Math.random() - 0.5) * 100;
    
    // 速度向量
    velocities[i3] = -0.5 - Math.random(); // 向左飞
    velocities[i3 + 1] = -0.5 - Math.random(); // 向下飞
    velocities[i3 + 2] = 0.1 * (Math.random() - 0.5); // 略微的z轴变化
    
    // 随机大小
    sizes[i] = Math.random() * 0.5 + 0.1;
  }

  particlesGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
  particlesGeometry.setAttribute('size', new THREE.BufferAttribute(sizes, 1));

  // 创建着色器材质
  const particleMaterial = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 0 },
      size: { value: 15.0 }
    },
    vertexShader: `
      attribute float size;
      varying float vAlpha;
      void main() {
        vec4 mvPosition = modelViewMatrix * vec4(position, 1.0);
        gl_Position = projectionMatrix * mvPosition;
        gl_PointSize = size * (300.0 / -mvPosition.z);
        vAlpha = size * 0.8; // 降低整体透明度
      }
    `,
    fragmentShader: `
      varying float vAlpha;
      void main() {
        float r = length(gl_PointCoord - vec2(0.5));
        if (r > 0.5) discard;
        gl_FragColor = vec4(1.0, 0.95, 0.8, vAlpha * 0.7); // 温暖的米黄色调
      }
    `,
    transparent: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending
  });

  particles = new THREE.Points(particlesGeometry, particleMaterial);
  scene.add(particles);

  // 添加特殊流星
  const specialMeteor = createSpecialMeteor();
  scene.add(specialMeteor);
};

// 创建特殊流星
const createSpecialMeteor = () => {
  const geometry = new THREE.BufferGeometry();
  const material = new THREE.LineBasicMaterial({
    color: 0x00ffff,
    transparent: true,
    opacity: 0.8
  });

  const points = [];
  points.push(new THREE.Vector3(-20, 20, 0));
  points.push(new THREE.Vector3(20, -20, 0));
  
  geometry.setFromPoints(points);
  const meteor = new THREE.Line(geometry, material);
  
  // 添加动画
  gsap.to(meteor.position, {
    x: 100,
    y: -100,
    duration: 2,
    repeat: -1,
    ease: "none",
    onRepeat: () => {
      meteor.position.set(-100, 100, 0);
    }
  });

  return meteor;
};

// 修改动画循环
const animate = () => {
  animationFrameId = requestAnimationFrame(animate);
  controls.update();
  
  if (particles) {
    const positions = particles.geometry.attributes.position.array;
    const particleCount = positions.length / 3;
    
    for (let i = 0; i < particleCount; i++) {
      const i3 = i * 3;
      positions[i3] += -0.5 - Math.random(); // x
      positions[i3 + 1] += -0.5 - Math.random(); // y
      positions[i3 + 2] += 0.1 * (Math.random() - 0.5); // z
      
      // 如果粒子飞出视野，重置位置
      if (positions[i3 + 1] < -25) {
        positions[i3] = (Math.random() - 0.5) * 100;
        positions[i3 + 1] = 50; // 回到顶部
        positions[i3 + 2] = (Math.random() - 0.5) * 100;
      }
    }
    particles.geometry.attributes.position.needsUpdate = true;
  }
  
  composer.render();
};

// --- 窗口大小调整处理 ---
const onWindowResize = () => {
  if (camera && renderer && composer && outlinePass) {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    composer.setSize(window.innerWidth, window.innerHeight);
    outlinePass.resolution.set(window.innerWidth, window.innerHeight);
  }
};

// --- Vue 生命周期钩子 ---
onMounted(() => {
  initScene();
});

onUnmounted(() => {
  cancelAnimationFrame(animationFrameId);
  window.removeEventListener('resize', onWindowResize);
  // 清理 Three.js 资源
  if (renderer) {
    renderer.dispose();
  }
  if (scene) {
    scene.traverse((object) => {
      if (object instanceof THREE.Mesh) {
        if (object.geometry) {
          object.geometry.dispose();
        }
        if (object.material) {
          if (Array.isArray(object.material)) {
            object.material.forEach(material => material.dispose());
          } else {
            object.material.dispose();
          }
        }
      }
    });
  }
  // 移除 DOM 元素
  if (sceneContainer.value && renderer) {
      sceneContainer.value.removeChild(renderer.domElement);
  }
});

</script>

<style lang="less" scoped>
/* 只保留正确的导入语句 */
@import '@/styles/threeScene.less'; 

/* 移除这里的 .scene-container 规则 */
</style>
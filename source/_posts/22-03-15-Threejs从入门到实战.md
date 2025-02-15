---
title: Threejs从入门到实战一文说明白
date: 2022-03-15
comments: true
tags:
- Threejs
- 场景
- 相机
- 渲染器
- 几何体
- 材质
- 动画循环
- Vue
- 脚手架
categories:
- 软件开发
---


## Three.js的主要知识点：

### 基础环境搭建与核心概念
- **环境搭建**：了解如何引入Three.js库文件，可通过下载到本地项目或者使用CDN链接等方式，创建基本的HTML页面结构来承载Three.js场景。
- **场景（Scene）、相机（Camera）和渲染器（Renderer）**：
    - **场景**：是所有物体、光源等元素的容器，类似于舞台，可往其中添加各种要展示的对象。
    - **相机**：定义了观察场景的视角和位置，比如常用的透视相机（PerspectiveCamera）模拟人眼的近大远小效果，正交相机（OrthographicCamera）用于等比例显示物体等。
    - **渲染器**：负责将场景和相机所确定的画面渲染到页面上，如WebGLRenderer用于在支持WebGL的浏览器中渲染出三维场景。

### 几何图形（Geometries）与材质（Materials）
- **几何图形**：
    - 基础的几何形状如立方体（BoxGeometry）、球体（SphereGeometry）、平面（PlaneGeometry）等，掌握它们的创建参数，例如立方体的长宽高，球体的半径、分段数等。
    - 复杂几何图形的创建，可以通过组合基本几何图形或者利用自定义顶点、面等方式来构建独特形状。
- **材质**：
    - 不同类型的材质表现各异，如基础材质（MeshBasicMaterial）不受光照影响，常用于简单的形状展示； Lambert材质（MeshLambertMaterial）模拟漫反射效果，能呈现出真实光照下的质感； Phong材质（MeshPhongMaterial）在处理高光等方面表现出色。
    - 材质的属性设置，像颜色、透明度、纹理映射等，通过纹理（Texture）可以给物体表面添加图片等细节，例如为一个立方体的各个面贴上不同的图像。

### 灯光（Lights）
- **不同类型灯光**：
    - 点光源（PointLight）类似灯泡，向四周发光；平行光（DirectionalLight）模拟太阳光，光线平行照射；聚光灯（SpotLight）有特定的照射范围和角度，像手电筒光一样聚焦。
    - 灯光的属性调节，比如强度、颜色、衰减范围等，合理设置灯光可以营造出逼真的光影效果。

### 模型加载与动画
- **模型加载**：学会使用Three.js提供的加载器（如GLTF加载器、OBJ加载器等）加载外部创建的三维模型文件，了解模型的格式特点以及如何处理加载后的模型，包括调整位置、缩放等操作。
- **动画**：
    - 利用Three.js的动画循环（requestAnimationFrame结合相关函数）来更新物体的位置、旋转、缩放等属性，实现物体的动态效果，例如让一个物体绕某轴旋转或者沿某路径移动。
    - 关键帧动画的实现，通过定义不同时间点物体的状态来创建复杂的动画序列，配合缓动函数等让动画过渡更自然。

### 交互与后期处理
- **交互**：使用JavaScript的事件机制（如鼠标点击、移动等事件）结合Three.js的对象操作，实现与场景中物体的交互，比如点击物体弹出信息、拖动物体改变位置等。
- **后期处理**：掌握如EffectComposer等后期处理组件，添加各种效果，像模糊效果、辉光效果、色调映射等，增强画面的整体视觉质感。

### 性能优化与跨平台应用
- **性能优化**：了解如何减少绘制调用、合理使用纹理压缩、进行场景裁剪、优化模型面数等方法，确保在不同设备上（尤其是移动端）都能流畅渲染三维场景。
- **跨平台应用**：认识到Three.js在不同浏览器环境下的兼容性处理，以及在移动端通过适配触摸操作等手段，让创建的三维应用能有更广泛的应用场景，例如嵌入到Web页面、移动端APP的WebView中展示等。 


## 写一个Hello Three.js的例子

我们基于 Three.js 写一个 “Hello World”，以下是示例代码，效果是在网页上创建一个旋转的立方体。

### 示例代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Three.js Hello World</title>
  <!-- 引入 Three.js 库 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>

<body>
  <script>
    // 1. 创建场景
    const scene = new THREE.Scene();

    // 2. 创建相机
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.z = 5;

    // 3. 创建渲染器
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // 4. 创建几何体和材质
    const geometry = new THREE.BoxGeometry();
    const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });

    // 5. 创建网格对象
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    // 6. 动画循环函数
    function animate() {
      requestAnimationFrame(animate);

      // 让立方体旋转
      cube.rotation.x += 0.01;
      cube.rotation.y += 0.01;

      // 渲染场景
      renderer.render(scene, camera);
    }

    // 7. 启动动画循环
    animate();
  </script>
</body>

</html>
```

### 代码解释

#### 1. 引入 Three.js 库
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
```
通过 CDN 链接引入 Three.js 库文件，这样可以方便地使用 Three.js 提供的各种功能。

#### 2. 创建场景
```javascript
const scene = new THREE.Scene();
```
`THREE.Scene` 是 Three.js 中的场景对象，它是所有物体、光源等元素的容器，类似于舞台，所有要展示的对象都需要添加到这个场景中。

#### 3. 创建相机
```javascript
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 5;
```
- `THREE.PerspectiveCamera` 创建一个透视相机，模拟人眼的近大远小效果。
  - `75` 是相机的视野角度（Field of View），表示相机可以看到的垂直角度范围。
  - `window.innerWidth / window.innerHeight` 是相机的纵横比，确保渲染的画面不会变形。
  - `0.1` 和 `1000` 分别是相机的近裁剪面和远裁剪面，只有在这个范围内的物体才会被渲染。
- `camera.position.z = 5` 将相机沿 z 轴正方向移动 5 个单位，避免相机和物体重叠。

#### 4. 创建渲染器
```javascript
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);
```
- `THREE.WebGLRenderer` 创建一个基于 WebGL 的渲染器，用于将场景和相机所确定的画面渲染到页面上。
- `renderer.setSize(window.innerWidth, window.innerHeight)` 设置渲染器的尺寸为浏览器窗口的宽度和高度。
- `document.body.appendChild(renderer.domElement)` 将渲染器生成的 `<canvas>` 元素添加到 HTML 页面的 `<body>` 中，这样才能在页面上看到渲染的结果。

#### 5. 创建几何体和材质
```javascript
const geometry = new THREE.BoxGeometry();
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
```
- `THREE.BoxGeometry` 创建一个立方体的几何体，它定义了立方体的形状和结构。
- `THREE.MeshBasicMaterial` 创建一个基础材质，这种材质不受光照影响，`{ color: 0x00ff00 }` 设置材质的颜色为绿色。

#### 6. 创建网格对象
```javascript
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);
```
- `THREE.Mesh` 将几何体和材质组合成一个网格对象，这个对象就是最终要在场景中显示的立方体。
- `scene.add(cube)` 将立方体添加到场景中。

#### 7. 动画循环函数
```javascript
function animate() {
  requestAnimationFrame(animate);

  // 让立方体旋转
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;

  // 渲染场景
  renderer.render(scene, camera);
}

animate();
```
- `requestAnimationFrame(animate)` 是一个浏览器提供的函数，用于在浏览器下次重绘之前调用指定的函数，实现动画循环。
- `cube.rotation.x += 0.01` 和 `cube.rotation.y += 0.01` 分别让立方体绕 x 轴和 y 轴旋转，每次旋转 0.01 弧度。
- `renderer.render(scene, camera)` 调用渲染器的 `render` 方法，将场景和相机所确定的画面渲染到页面上。


## 通过vue脚手架搭建threejs项目

日常开发中，我们使用 Vue 脚手架的场景居多，也是目前主流的前端项目的开发方法，所以，我这里再基于vue项目，实现上述 Three.js 旋转立方体功能。

### 步骤
1. **创建 Vue 项目**：使用 Vue CLI 创建一个新的 Vue 项目。
```bash
vue create threejs-vue-demo
cd threejs-vue-demo
```
2. **安装 Three.js**：通过 npm 安装 Three.js 库。
```bash
npm install three
```
3. **编写 Vue 组件代码**：在 `src/components` 目录下创建一个新的组件，例如 `ThreeCube.vue`，并在 `App.vue` 中引入该组件。

### 示例代码

#### `src/components/ThreeCube.vue`
```vue
<template>
  <div ref="container"></div>
</template>

<script>
import * as THREE from 'three';

export default {
  name: 'ThreeCube',
  mounted() {
    // 1. 获取容器元素
    const container = this.$refs.container;

    // 2. 创建场景
    const scene = new THREE.Scene();

    // 3. 创建相机
    const camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.z = 5;

    // 4. 创建渲染器
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    // 5. 创建几何体和材质
    const geometry = new THREE.BoxGeometry();
    const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });

    // 6. 创建网格对象
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    // 7. 动画循环函数
    const animate = () => {
      requestAnimationFrame(animate);

      // 让立方体旋转
      cube.rotation.x += 0.01;
      cube.rotation.y += 0.01;

      // 渲染场景
      renderer.render(scene, camera);
    };

    // 8. 启动动画循环
    animate();
  },
};
</script>

<style scoped>
div {
  width: 100%;
  height: 100vh;
}
</style>
```

#### `src/App.vue`
```vue
<template>
  <div id="app">
    <ThreeCube />
  </div>
</template>

<script>
import ThreeCube from './components/ThreeCube.vue';

export default {
  name: 'App',
  components: {
    ThreeCube,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

### 代码解释

#### `ThreeCube.vue`

##### 模板部分 (`<template>`)
```vue
<template>
  <div ref="container"></div>
</template>
```
- 使用 `<div>` 元素作为 Three.js 场景的容器，并通过 `ref="container"` 给该元素添加一个引用，方便在 JavaScript 中获取该元素。

##### 脚本部分 (`<script>`)
```javascript
import * as THREE from 'three';

export default {
  name: 'ThreeCube',
  mounted() {
    // 1. 获取容器元素
    const container = this.$refs.container;

    // 2. 创建场景
    const scene = new THREE.Scene();

    // 3. 创建相机
    const camera = new THREE.PerspectiveCamera(75, container.clientWidth / container.clientHeight, 0.1, 1000);
    camera.position.z = 5;

    // 4. 创建渲染器
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(container.clientWidth, container.clientHeight);
    container.appendChild(renderer.domElement);

    // 5. 创建几何体和材质
    const geometry = new THREE.BoxGeometry();
    const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });

    // 6. 创建网格对象
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    // 7. 动画循环函数
    const animate = () => {
      requestAnimationFrame(animate);

      // 让立方体旋转
      cube.rotation.x += 0.01;
      cube.rotation.y += 0.01;

      // 渲染场景
      renderer.render(scene, camera);
    };

    // 8. 启动动画循环
    animate();
  },
};
```
- `import * as THREE from 'three';`：引入 Three.js 库。
- `mounted()`：Vue 组件的生命周期钩子函数，当组件挂载到 DOM 后执行。
  - **获取容器元素**：通过 `this.$refs.container` 获取模板中定义的容器元素。
  - **创建场景**：使用 `THREE.Scene()` 创建一个场景对象。
  - **创建相机**：创建一个透视相机，根据容器的宽高比设置相机的纵横比，并将相机沿 z 轴正方向移动 5 个单位。
  - **创建渲染器**：使用 `THREE.WebGLRenderer()` 创建一个渲染器，设置渲染器的尺寸为容器的宽高，并将渲染器生成的 `<canvas>` 元素添加到容器中。
  - **创建几何体和材质**：创建一个立方体的几何体和一个绿色的基础材质。
  - **创建网格对象**：将几何体和材质组合成一个网格对象，并添加到场景中。
  - **动画循环函数**：使用 `requestAnimationFrame` 实现动画循环，在每次循环中让立方体绕 x 轴和 y 轴旋转，并调用渲染器的 `render` 方法渲染场景。
  - **启动动画循环**：调用 `animate()` 函数启动动画循环。

##### 样式部分 (`<style>`)
```css
div {
  width: 100%;
  height: 100vh;
}
```
设置容器元素的宽度为 100%，高度为视口的高度。

#### `App.vue`
在 `App.vue` 中引入 `ThreeCube` 组件并使用，将 `ThreeCube` 组件渲染到页面上。

通过以上步骤和代码，你就可以在 Vue 项目中实现一个旋转的立方体效果。
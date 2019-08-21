# 一步步带你实现web全景看房——three.js

> canvas画2d相信大家都很熟悉了，但3d世界更加炫酷。我们直接从three.js入手。下面我们从0开始来摸索一下3d世界

# 1. 基本概念

在THREEjs中，渲染一个3d世界的必要因素是场景（scene）、相机（camera）、渲染器（renderer）。渲染出一个3d世界后，可以往里面增加各种各样的物体、光源等，形成一个3d世界：

![img](https://user-gold-cdn.xitu.io/2019/8/17/16c9e27c783ed919?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 场景:右手坐标系,一切要素都在场景里面，相当于“世界”，包括各种物质和物理变化

![img](https://user-gold-cdn.xitu.io/2019/8/17/16c9e289e01cce9c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```js
// 创建场景
const scene = new THREE.Scene();
```

> 照相机:摄像机就相当于人眼，有摄像机才可以看见场景里面的一切物体和光源。常用的是正交摄像机和透视摄像机

![img](https://user-gold-cdn.xitu.io/2019/8/17/16c9e2986f13a330?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

正交摄像机是一个矩形可视区域，物体只有在这个区域内才是可见的物体无论距离摄像机是远或事近，物体都会被渲染成一个大小。一般应用场景是2.5d游戏如跳一跳、机械模型

```
// 创建正交相机
const camera = new THREE.OrthographicCamera(
    -window.innerWidth / 200,
    window.innerWidth /200 ,
    window.innerHeight/ 200,
    -window.innerHeight/ 200,
    1,
    1000
);
```

![img](https://user-gold-cdn.xitu.io/2019/8/18/16ca351c6f312234?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们可以看见上图的效果，有一个正方体已经走了很远但是大小不变。另外还可以看见角落有一个正方体已经被截断了一部分，那是因为正交摄像机仅仅展示一个空间内的场景，所以会有截断效果。

**透视摄像机**是最常用的摄像机类型，模拟人眼的视觉，近大远小（透视）。Fov表示的是视角，Fov越大，表示眼睛睁得越大，离得越远，看得更多。如果是需要模拟现实，基本都是用这个相机

![img](https://user-gold-cdn.xitu.io/2019/8/17/16c9e29a5247ccc3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```js
// 创建透视相机
const camera = new THREE.PerspectiveCamera(
          90,
          window.innerWidth / window.innerHeight,
          1,
          10000
        );
```

![img](https://user-gold-cdn.xitu.io/2019/8/18/16ca350e6198139f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

近大远小的效果就出来了，比较符合现实

> 渲染器

最后需要把所有的内容渲染到页面上，需要一个渲染器：

```js
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight); // canvas大小
      document.body.appendChild(renderer.domElement);
```

# 2. 给画面增加内容

上面的确是把3d世界画出来了，只是没有什么东西。在three.js中，我们需要增加光源和mesh

## mesh

mesh即是网格。在计算机里，3D世界是由点组成的，无数的面拼接成各种形状的物体。这种模型叫做网格模型。一条线是两个点组成，一个面是3个点组成，一个物体由多个3点组成的面组成：

![img](https://user-gold-cdn.xitu.io/2019/8/18/16ca33f4feda9dba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

而网格（mesh）又是由几何体（geometry）和材质（material）构成的

### geometry

![img](https://user-gold-cdn.xitu.io/2019/8/18/16ca346a5297527c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![img](https://user-gold-cdn.xitu.io/2019/8/18/16ca347be85ba6de?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们所能想象到的几何体，框架都自带了，我们只需要调用对应的几何体构造函数即可创建。几何体的创建方法都是new，如BoxBuffer：

```
const geometry = new THREE.BoxBufferGeometry( 1, 1, 1 );
```

创建的时候，一般定义了渲染一个 3D 物体所需要的基本数据：Face 面、Vertex 顶点等信息。THREE.xxxGeometry指的是框架自带的几何体，不同几何体所需要的参数有所不同，大概是width、height、radius、depth、segment、detail、angle等属性

[更多geometry相关api](https://link.juejin.im/?target=https%3A%2F%2Fthreejs.org%2Fexamples%2F%3Fq%3Dgeo%23webgl_geometries)

> BufferGeometry和Geometry有什么不同?就实现的效果来说它们都是一样，但是BufferGeometry的多了一些顶点属性，且性能较好。对于开发者来说，Geometry对象属性少体验更好。THREE解析几何体对象的时候，如果是Geometry，则会把对象转换成ufferGeometry对象，再进行下一步渲染

### material

![img](一步步带你实现web全景看房——three.js.assets/16ca34af011fcea5)

一个物体很多的物理性质，取决于其材料，材料也决定了几何体的外表。材料的创建方法也是new，如Lambert材料：

```js
const material = new THREE.MeshLambertMaterial();
```

一个物体是否有镜面感、亮暗、颜色、透明、是否反光等性质，取决于使用什么材料。THREE.xxxMaterial指的是框架自带的材料，不同材料所需要的参数也是有所不同

[更多material相关api](https://link.juejin.im/?target=https%3A%2F%2Fthreejs.org%2Fdocs%2F%23api%2Fen%2Fmaterials%2FLineBasicMaterial)

有了geometry和material,就可以创建一个mesh并追加到场景中：

```js
const mesh = new THREE.Mesh(geometry, material);
scene.add(mesh);
```

## 光源

一个3d世界，如果需要更加逼真，那就需要光源了。光也有很多种，常见的有平行光（图2）、点光源（图3）、环境光（环境光充满所有的几何体表面）、聚光灯（图1）

![img](一步步带你实现web全景看房——three.js.assets/16ca355c2c04c6c2)

其中，只有平行光、点光源才能产生阴影。而且有的材料是受光源影响，没有光就是黑的。而一些材料是不受光影响的。光源的创建，如直射光：

```js
const light = new THREE.DirectionalLight(0xffffff, 0.9)
```

THREE.xxxLight指的是框架自带的光源构造函数，一般实例化的时候需要的参数是color、intensity、distance等配置。另外，一个3d世界当然不是一种光构成，所以光可以叠加，叠加的结果作用与物体上。

而且物体的影子也不是白送的，需要某些支持影子的光加上开发者配置：

```js
// 光产生影子
light.castShadow = true;
// 地面接受影子
ground.receiveShadow = true;
// 物体产生影子
mesh.castShadow = true;
```

[更多光源相关的api](https://link.juejin.im/?target=https%3A%2F%2Fthreejs.org%2Fdocs%2F%23api%2Fen%2Flights%2FAmbientLight)

[更多影子相关的api](https://link.juejin.im/?target=https%3A%2F%2Fthreejs.org%2Fdocs%2F%23api%2Fzh%2Flights%2Fshadows%2FLightShadow)

# 3. 调试工具

## 轨道控制器

加上此控制器，就可以通过鼠标拖拽、滚动对整个画面进行拖拽放缩 轨道控制器代码在[THREE官方github](https://link.juejin.im/?target=https%3A%2F%2Fraw.githubusercontent.com%2Fmrdoob%2Fthree.js%2Fmaster%2Fexamples%2Fjs%2Fcontrols%2FOrbitControls.js)上，如果使用的时候报错`THREE.OrbitControls is not a constructor`，那么就copy一份下来，第一行加一个window：`window.THREE.OrbitControls = ...`

使用方法就是new一个控制器，然后监听变化，触发render

```js
        const controls = new THREE.OrbitControls(camera, renderer.domElement);
        controls.addEventListener("change", () => {
          renderer.render(scene, camera);
        });
        controls.minDistance = 1;
        controls.maxDistance = 2000;
        controls.enablePan = false;
```

## 性能监控

[源代码](https://link.juejin.im/?target=https%3A%2F%2Fraw.githubusercontent.com%2Fmrdoob%2Fthree.js%2Fmaster%2Fexamples%2Fjs%2Flibs%2Fstats.min.js)。可以拷贝下来，挂在window上

官方大部分例子都使用了一个stat的插件，在左上角会出现性能变化的曲线，供我们调试使用。使用方法：

```js
    const stat = new Stats();
    document.body.appendChild(stat.dom);
    
    // 改造render函数
    function render() {
      renderer.render(scene, camera);
      stat.update();
    }
```

![img](一步步带你实现web全景看房——three.js.assets/16ca36af3451018c)

# 4. let's coding

先把场景、摄像机、渲染器弄出来，然后添加一个红色的球

```js
      function init() {
      const renderer = new THREE.WebGLRenderer();
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      // 场景
      const scene = new THREE.Scene();
      // 相机
      const camera = new THREE.PerspectiveCamera(
        90,
        window.innerWidth / window.innerHeight,
        0.1,
        100
      );
      camera.position.set(10, 0, 0);

      // 轨道控制器
      const controls = new THREE.OrbitControls(camera, renderer.domElement);
      controls.addEventListener("change", render);
      controls.minDistance = 1;
      controls.maxDistance = 200;
      controls.enablePan = false;

      // 新增一个红色球
      const geometry = new THREE.SphereGeometry(1, 10, 10);
      const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
      const mesh = new THREE.Mesh(geometry, material);
      scene.add(mesh);
      // 坐标轴辅助线
      scene.add(new THREE.AxisHelper(1000));

      controls.update(); // 控制器需要
      controls.target.copy(mesh.position);

      function render() {
        renderer.render(scene, camera);
      }

      function r() {
        render();
        requestAnimationFrame(r)
      }
      r()
    }
    
    init();
```

此时，可以看见坐标原点上有一个球。其实，一个几何体纹理是可以使用图片的，甚至还可以使用视频，此时不能双击打开html，需要本地起一个服务器打开。我们改造一下mesh:

```js
    function addImg(url, scene, n = 1) {
      const texture = THREE.ImageUtils.loadTexture(url);
      const material = new THREE.MeshBasicMaterial({ map: texture });
      const geometry = new THREE.SphereGeometry(1, 10, 10);
      const mesh = new THREE.Mesh(geometry, material);
      scene.add(mesh);
      return mesh;
    }
    
    
      // const geometry = new THREE.SphereGeometry(1, 10, 10);
      // const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
      // const mesh = new THREE.Mesh(geometry, material);
      // 去酷家乐找了一个图
      const mesh = addImg("https://qhyxpicoss.kujiale.com/r/2019/07/01/L3D137S8ENDIADDWAYUI5L7GLUF3P3WS888_3000x4000.jpg?x-oss-process=image/resize,m_fill,w_1600,h_920/format,webp", scene, 1); 
      scene.add(mesh);
```

![](一步步带你实现web全景看房——three.js.assets/L3D137S8ENDIADDWAYUI5L7GLUF3P3WS888_3000x4000[1].jpg)

原点显示一个图作为纹理的球

![img](https://user-gold-cdn.xitu.io/2019/8/18/16ca38ba36e51165?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 基本都ok了，怎么实现全景看房呢？我们上面的条件都ok了，最后需要做的事情是：将摄像机放在球体中心、轨道控制器放缩上限最小最大设置成1和2、渲染mesh内表面

```js
    // 调整max
      controls.minDistance = 1;
      // controls.maxDistance = 200;
      controls.maxDistance = 2;
      
      // 调整球大小
      // const geometry = new THREE.SphereGeometry(1, 10, 10);
      const geometry = new THREE.SphereGeometry(50, 256, 256);
      
      // 摄像机放球体中心
      // camera.position.set(10, 0, 0);
      camera.position.set(-0.3, 0, 0);
      
      // 渲染球体的双面
      const material = new THREE.MeshLambertMaterial({ map: texture });
      material.side = THREE.DoubleSide;
```

全景看房的效果就出来了，然后只需拖动就可以调整角度了。引入是普通平面图，所以图的首尾交接有一点问题。

![img](https://user-gold-cdn.xitu.io/2019/8/18/16ca392a3d0351f5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 这只是实现的一个思路，实现的方法有很多，如柱体、立方体，图片可能是扇形的全景图也可能是多个图片拼接起来的。具体的细节根据业务进行调整

全部代码如下，**需要引入three.js、orbitcontrol**

<details style="display: block;"><summary style="display: block;">全部代码</summary></details>

> 关注公众号《不一样的前端》，以不一样的视角学习前端，快速成长，一起把玩最新的技术、探索各种黑科技
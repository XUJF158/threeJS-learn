<script setup lang="ts">
import {
  BoxGeometry,
  DirectionalLight,
  Mesh,
  MeshBasicMaterial,
  MeshPhongMaterial,
  PerspectiveCamera,
  Scene,
  WebGLRenderer,
} from "three";
import { onMounted, onUnmounted, ref } from "vue";
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader.js";

const canvasDom = ref<HTMLCanvasElement>();

onMounted(() => {
  init();
});

function init() {
  if (canvasDom.value) {
    // 创建渲染器
    const renderer = new WebGLRenderer({
      canvas: canvasDom.value,
      antialias: true,
    });

    //创建镜头
    //PerspectiveCamera() 中的 4 个参数分别为：
    //1、fov(field of view 的缩写)，可选参数，默认值为 50，指垂直方向上的角度，注意该值是度数而不是弧度
    //2、aspect，可选参数，默认值为 1，画布的宽高比(宽/高)，例如画布宽300像素，高150像素，那么意味着宽高比为 2
    //3、near，可选参数，默认值为 0.1，近平面，限制摄像机可绘制最近的距离，若小于该距离则不会绘制(相当于被裁切掉)
    //4、far，可选参数，默认值为 2000，远平面，限制摄像机可绘制最远的距离，若超出该距离则不会绘制(相当于被裁切掉)
    //以上 4 个参数在一起，构成了一个 “视椎”
    const camera = new PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      5
    );

    // 创建场景
    const scene = new Scene();

    // 创建几何体
    const geometry = new BoxGeometry(1, 1, 1);

    // 材质
    // const material = new MeshBasicMaterial({ color: 0x44aa88 })
    // MeshPhongMaterial 能够反射光
    const material = new MeshPhongMaterial({ color: 0x44aacc });

    // 创建网格 网格 = 几何体 + 材质
    const cube = new Mesh(geometry, material);
    scene.add(cube); // 添加到场景

    // 创建光源
    const light = new DirectionalLight(0xfffffff, 1);
    light.position.set(-1, 2, 13);
    scene.add(light); // 将光源添加到场景中，否则一片漆黑

    //设置透视镜头的Z轴距离，以便我们以某个距离来观察几何体
    //之前初始化透视镜头时，设置的近平面为 0.1，远平面为 5
    //因此 camera.position.z 的值一定要在 0.1 - 5 的范围内，超出这个范围则画面不会被渲染
    camera.position.z = 3;

    //渲染器根据场景、透视镜头来渲染画面，并将该画面内容填充到 DOM 的 canvas 元素中
    renderer.render(scene, camera); //由于后面我们添加了自动渲染渲染动画，所以此处的渲染可以注释掉

    //添加自动旋转渲染动画
    const render = () => {
      const time = 0.01; //原本 time 为毫秒，我们这里对 time 进行转化，修改成 秒，以便于我们动画旋转角度的递增
      cube.rotation.x += time;
      cube.rotation.y += time;

      renderer.render(scene, camera);
      window.requestAnimationFrame(render);
    };

    const handleResize = () => {
      const canvas = renderer.domElement; // 获取 canvas
      camera.aspect = canvas.clientWidth / canvas.clientHeight; // 设置镜头宽高比
      camera.updateProjectionMatrix(); // 通知镜头更新视椎(视野)
      renderer.setSize(canvas.clientWidth, canvas.clientHeight, false);
    };

    handleResize();

    window.requestAnimationFrame(render);

    window.addEventListener("resize", handleResize); //添加窗口 resize 事件处理函数
    onUnmounted(() => {
      window.removeEventListener("reset", handleResize);
    });

    const loader = new FBXLoader();
    const url = "./model/test.fbx";
    loader.load(url, function (loadedModel) {
      console.log(loadedModel);
      loadedModel.translateY(-80);
      const mesh = loadedModel.children;
      scene.add(...mesh);
    });
  }
}
</script>

<template>
  <canvas ref="canvasDom"></canvas>
</template>

<style lang="scss" scoped>
canvas {
  display: block;
  width: inherit;
  height: inherit;
}
</style>

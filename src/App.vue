<template>
    <!-- <el-button type="primary">
    <router-link to="/">Go to Home</router-link>
  </el-button>
   -->
    <Header></Header>
    <div id="map"></div>
    <div id="components">
        <router-view></router-view>
    </div>
<!-- 
    <button
        style="position: absolute; left: 100px; top: 30px; z-index: 1999"
        @click="removeModel"
    >
        删除模型
    </button>
    <button
        style="position: absolute; left: 100px; top: 60px; z-index: 1999"
        @click="addModel"
    >
        添加模型
    </button> -->
</template>

<script setup>
import { onMounted } from 'vue';
import { app } from './main';
import Header from './components/Header.vue';

// 引入mapboxgl
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';
// 引入L7
import { Scene } from '@antv/l7';
// 引入L7-mapboxgl
import { Mapbox } from '@antv/l7-maps';

// 引入加载三维建筑的类
import modelLoadHelper from './loadObjModels';

// 在组件加载的时候初始化地图
onMounted(() => {
    // 注册token
    const token = import.meta.env.VITE_TOKEN;
    mapboxgl.accessToken = token;

    // 创建mapbox地图
    const map = new mapboxgl.Map({
        container: 'map',
        style: 'mapbox://styles/mapbox/dark-v11',
        center: [114.3, 30.5],
        // zoom: 14,
        zoom: 2,
        projection: 'globe',
        // 设置倾角
        // pitch: 75,
    });

    // 创建L7实例
    const scene = new Scene({
        id: 'map',
        map: new Mapbox({
            mapInstance: map,
        }),
        logoVisible: false,
    });

    // 设置雾效
    map.on('style.load', () => {
        map.setFog({});
    });

    // 使用provide 将地图实例化对象全局共享
    app.provide('$scene_map', { scene, map });

    // // 加载模型
    // const modelLoadOpt = {
    //     center: [114.298, 30.502],
    //     angle: 0,
    //     scale: {
    //         x: 6,
    //         y: 6,
    //         z: 20,
    //     },
    //     objUrl: '/src/assets/models/model3/LeosVillage.obj',
    //     mtlUrl: '/src/assets/models/model3/LeosVillage.mtl',
    // };

    // loader = new modelLoadHelper(map, modelLoadOpt);

    // const removeModel = () => {
    //     console.log(layer);
    //     layer && loader && loader.removeModel(layer.id);
    // };

    // const addModel = () => {
    //     loader && loader.addModel();
    // };
});
</script>

<style scoped>
#map {
    width: 100vw;
    height: 100vh;
}

Header {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 10;
}

#components {
    background-color: rgba(0, 0, 0, 0.331);
    color: white;
    position: absolute;
    top: 0;
    left: 0;
    z-index: 2;
}
</style>

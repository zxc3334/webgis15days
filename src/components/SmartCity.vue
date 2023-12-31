<template>
    <!-- 构造城市场景组件 -->
    <div></div>
</template>

<script setup>
import { h, inject, onMounted } from 'vue';
import axios from 'axios';
import {
    Scene,
    CityBuildingLayer,
    LineLayer,
    LayerSwitch,
    HeatmapLayer,
} from '@antv/l7';

const { scene, map } = inject('$scene_map');

// 封装加载建筑的函数
const renderBuilding = async () => {
    // 加载楼房
    // 通过服务，请求到武汉市建筑数据
    const buildData = await axios('http://localhost:8080/wuhan_builds');
    // 查看一下我们获取的数据
    // console.log(buildData);
    const wuhan_builds_layer = new CityBuildingLayer({
        name: '武汉市建筑',
    });
    wuhan_builds_layer
        .source(buildData.data)
        // 设置高度字段
        .size('Elevation', (h) => h)
        // 楼房颜色
        .color('rgba(242,246,250,1.0)')
        // 是否开启动画效果 在开启 animate 后默认会打开点亮窗户的动画
        .animate({
            enable: true,
        })
        // 鼠标悬浮，元素高亮效果
        .active({
            color: '#0ff',
            mix: 0.5,
        })
        // 设置楼房样式，可以参考文档
        .style({
            opacity: 0.7,
            baseColor: 'rgb(16, 16, 16)',
            windowColor: 'rgb(30, 60, 89)',
            brightColor: 'rgb(255, 176, 38)',
            sweep: {
                enable: true,
                sweepRadius: 2,
                sweepColor: '#0274fe',
                sweepSpeed: 0.3,
                sweepCenter: [114.3, 30.5],
            },
        })
        // 只保留高度大于40的楼房
        .filter('Elevation', (h) => h > 40);
    return wuhan_builds_layer;
};

// 封装加载道路的函数
const renderRoad = async () => {
    // 加载道路
    const roadsData = await axios('http://localhost:8080/wuhan_roads');
    console.log(roadsData);
    const wuhan_roads_layer = new LineLayer({
        name: '武汉市道路',
        zIndex: 0,
        depth: true,
    })
        // 添加道路数据
        .source(roadsData.data)
        // 设置道路宽度
        .size(1)
        .shape('line')
        // 设置颜色
        .color('#1990FF')
        //设置道路流线的效果
        .animate({
            interval: 1, // 间隔
            duration: 2, // 持续时间，延时
            trailLength: 2, // 流线长度
        })
        // 将过短的数据筛选掉
        .filter('coordinates', (evt) => {
            // console.log(evt)
            return evt.length > 20;
        });
    return wuhan_roads_layer;
};

// 封装加载高亮边界的函数
const renderLight = async () => {
    const wuhanData = await axios(
        'https://geo.datav.aliyun.com/areas_v3/bound/420100_full.json'
    );

    const highlight_layer = new LineLayer({
        name: '武汉市高亮边界',
    })
        .source(wuhanData.data)
        .size(40)
        .shape('wall')
        .style({
            heightfixed: true,
            opacity: 'testOpacity',
            sourceColor: '#0DCCFF',
            targetColor: 'rbga(255,255,255,0)',
        });
    return highlight_layer;
};

// 封装加载热力图的函数
const renderHeatmap = async () => {
    const heatmapData = await axios('http://localhost:8080/wuhan_population');
    console.log(heatmapData);
    const heatmap_layer = new HeatmapLayer({
        name: '武汉市人口热力图',
    })
        .source(heatmapData.data)
        .shape('heatmap')
        .size('F2022', [0, 2]) // weight映射通道
        .style({
            intensity: 2,
            radius: 40,
            rampColors: {
                colors: [
                    '#FF4818',
                    '#F7B74A',
                    '#FFF598',
                    '#91EABC',
                    '#2EA9A1',
                    '#206C7C',
                ].reverse(),
                positions: [0, 0.2, 0.4, 0.6, 0.8, 1.0],
            },
        });
    console.log('热力图');
    return heatmap_layer;
};

// 封装加载地铁线路图的函数
const renderSubLine = async () => {
    const subLineData = await axios('http://localhost:8080/subLine');
    console.log(subLineData);
    const subLine_layer = new LineLayer({
        name: '武汉市地铁线路图',
    })
        .source(subLineData.data)
        .size(2)
        .shape('line')
        .color('name', ['#5B8FF9', '#5CCEA1', '#F6BD16'])
        .style({
            borderWidth: 0.4,
            borderColor: '#fff',
        })
        .active(true)
        .animate({
            interval: 1, // 间隔
            duration: 2, // 持续时间，延时
            trailLength: 2, // 流线长度
        })
        .filter('coordinates', (evt) => {
            // console.log(evt)
            return evt.length > 20;
        });
    return subLine_layer;
};

onMounted(async () => {
    // 调用封装好的函数获取建筑和道路图层
    const wuhan_builds_layer = await renderBuilding();
    const wuhan_roads_layer = await renderRoad();
    const highlight_layer = await renderLight();
    const heatmap_layer = await renderHeatmap();
    const subLine_layer = await renderSubLine();

    // 运行阶段用
    // let hasLayers = false;
    // let hasControl = false;
    // map.on('zoom', () => {
    //     let mapZoom = map.getZoom();
    //     if (mapZoom > 5 && !hasLayers) {
    //         // 将图层添加到场景中
    //         scene.addLayer(wuhan_builds_layer);
    //         scene.addLayer(wuhan_roads_layer);
    //         scene.addLayer(heatmap_layer);
    //         scene.addLayer(subLine_layer);
    //         scene.addLayer(highlight_layer);
    //         // 不显示多余图层
    //         heatmap_layer.hide();
    //         wuhan_builds_layer.hide();
    //         wuhan_roads_layer.hide();
    //         subLine_layer.hide();
    //         // 添加图层控制器
    //         const layerControl = new LayerSwitch({
    //             appendTo: document.body,
    //             layers: [
    //                 wuhan_builds_layer,
    //                 wuhan_roads_layer,
    //                 heatmap_layer,
    //                 subLine_layer,
    //                 highlight_layer
    //             ],
    //             title: '图层控制',
    //             position: 'topright',
    //         });
    //         hasLayers = true;
    //         if (!hasControl) {
    //             scene.addControl(layerControl);
    //             hasControl = true;
    //         }
    //     } else if (mapZoom <= 5 && hasLayers) {
    //         scene.removeLayer(wuhan_builds_layer);
    //         scene.removeLayer(wuhan_roads_layer);
    //         scene.removeLayer(highlight_layer);
    //         scene.removeLayer(heatmap_layer);
    //         scene.removeLayer(subLine_layer);
    //         hasLayers = false;
    //     }
    // });
    
    // 开发阶段用
    // 将图层添加到场景中
    scene.addLayer(wuhan_builds_layer);
    scene.addLayer(wuhan_roads_layer);
    scene.addLayer(heatmap_layer);
    scene.addLayer(subLine_layer);
    scene.addLayer(highlight_layer);
    // 不显示多余图层
    heatmap_layer.hide();
    wuhan_builds_layer.hide();
    wuhan_roads_layer.hide();
    subLine_layer.hide();
    // 添加图层控制器
    const layerControl = new LayerSwitch({
        appendTo: document.body,
        layers: [
            wuhan_builds_layer,
            wuhan_roads_layer,
            heatmap_layer,
            subLine_layer,
            highlight_layer,
        ],
        title: '图层控制',
        position: 'topright',
    });
    scene.addControl(layerControl);
});
</script>
<style scoped>
.layer-control {
    width: 150px;
    height: 250px;
    background-color: #fff;
    border-radius: 5px;
    z-index: 100;
    float: right;
    position: fixed;
    top: 144px;
    right: 10px;
    color: black;
}
</style>

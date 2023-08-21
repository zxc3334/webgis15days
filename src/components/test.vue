<template>
    <el-table :data="tableData" border style="width: 100%" @row-click="clickRow">
        <el-table-column prop="name" label="名称" width="180" />
        <el-table-column prop="address" label="地址"/>
        <el-table-column prop="Lat" label="纬度" width="120"/>
        <el-table-column prop="Lng" label="经度" width="120"/>
        <el-table-column prop="tel" label="电话" width="180" />
    </el-table>
</template>

<script setup>
import { onMounted, onUnmounted, watch, ref, inject, } from 'vue';
const { scene, map } = inject('$scene_map');
import { Marker, Popup } from '@antv/l7';

const tableData = [
    {
        name: '武汉市公安局',
        address: '湖北省武汉市江汉区发展大道188号',
        Lat: '30.59',
        Lng: '114.29',
        tel: '0571-88888888',
    },
    {
        name: '武汉市公安局',
        address: '湖北省武汉市江汉区发展大道188号',
        Lat: '30.59',
        Lng: '114.29',
        tel: '0571-88888888',
    },
    {
        name: '武汉市公安局',
        address: '湖北省武汉市江汉区发展大道188号',
        Lat: '30.59',
        Lng: '114.29',
        tel: '0571-88888888',
    },
    {
        name: '武汉市公安局',
        address: '湖北省武汉市江汉区发展大道188号',
        Lat: '30.59',
        Lng: '114.29',
        tel: '0571-88888888',
    },
    {
        name: '武汉市公安局',
        address: '湖北省武汉市江汉区发展大道188号',
        Lat: '30.59',
        Lng: '114.29',
        tel: '0571-88888888',
    },
];

// 点击行跳转至行中的经纬度
const clickRow = (row) => {
    console.log(row);
    const { Lat, Lng, name } = row;
    map.flyTo({
        center: [Lng, Lat],
        zoom: 15,
        speed: 0.8,
        curve: 1,
    });

    // 添加popup    
    const popup = new Popup({
        offsets: [0, 40],
        closeButton: false,
    }).setLnglat([Lng, Lat]).setHTML(`<div style="color: #000; font-size: 14px; font-weight: bold;">${name}</div>`);

    // 添加标记
    const marker = new Marker({
        color: '#f47784',
    })
        .setLnglat([Lng, Lat])
        .setPopup(popup);
    scene.addMarker(marker);


    // 右键marker移除marker
    marker.on('contextmenu', () => {
        marker.remove(marker);
    });

};
</script>
<style scoped></style>

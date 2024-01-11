<script setup>
import {onMounted, ref, watch} from 'vue';
import {Map, View} from 'ol';
import TileLayer from 'ol/layer/Tile.js';
import {TileWMS} from 'ol/source';
import {Style} from 'ol/style';
import * as proj from 'ol/proj';
import ScaleLine from 'ol/control/ScaleLine'
import Zoom from 'ol/control/Zoom';
import ZoomSlider from 'ol/control/ZoomSlider';
import Rotate from 'ol/control/Rotate';
import FullScreen from 'ol/control/FullScreen';
import MousePosition from 'ol/control/MousePosition';
import * as olCoordinate from 'ol/coordinate';
import {defaults} from 'ol/control/defaults';
import TileArcGISRest from 'ol/source/TileArcGISRest';


let map = null;
let source = null;
const selectVal = ref('World_Street_Map');
const options = [
  {
    value: 'World_Street_Map',
    label: '世界地图服务',
    url: 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer'
  },
  {
    value: 'World_Imagery',
    label: '卫星影像地图服务',
    url: 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer'
  },
  {
    value: 'World_Topo_Map',
    label: '地形地图服务',
    url: 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer'
  }
]

let layers = [];

// // 定义 ArcGIS 世界地图服务的 REST 终端地址
// const arcgisWorldMapServiceUrl = 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer';
//
// // 创建 ArcGIS 世界地图服务的源
// const arcgisSource = new TileArcGISRest({
//   url: arcgisWorldMapServiceUrl,
// });
//
// // 创建 ArcGIS 世界地图服务的图层
// const arcgisLayer = new TileLayer({
//   source: arcgisSource,
// });
//
// // 使用XYZ的写法，地址后面加/tile/{z}/{y}/{x}
// // 定义 ArcGIS 影像地图服务的 REST 终端地址
// const arcgisImageryServiceUrl = 'https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}';
//
// // 创建 ArcGIS 影像地图服务的源
// const arcgisImagerySource = new XYZ({
//   url: arcgisImageryServiceUrl,
//   projection: 'EPSG:3857', // 使用Web Mercator投影坐标系
// });
//
// // 创建 ArcGIS 影像地图服务的图层
// const arcgisImageryLayer = new TileLayer({
//   source: arcgisImagerySource,
// });

onMounted(()=>{
  map = createMap()
  layers = getLayer(options);
  console.log('layers', layers);
  map.addLayer(layers[0]);
  window.map = map;
});

const getLayer = function (layerConfigs) {
  const _layers = [];
  layerConfigs.forEach(config=>{
    const source = new TileArcGISRest({
      url: config.url,
      projection: 'EPSG:3857', // 使用Web Mercator投影坐标系
    });

    const layer = new TileLayer({
      source,
      id: config.value
    });
    _layers.push(layer);
  });

  return _layers;
}

const getAnnotationLayer = function (){
  // 定义 ArcGIS 影像地图服务的注记图层的 REST 终端地址
  const arcgisImageryAnnotationServiceUrl = 'https://services.arcgisonline.com/ArcGIS/rest/services/Reference/World_Boundaries_and_Places/MapServer';

// 创建 ArcGIS 影像地图服务的注记图层的源
  const arcgisImageryAnnotationSource = new TileWMS({
    url: arcgisImageryAnnotationServiceUrl,
    params: { 'LAYERS': '1', 'TILED': true },
  });

  // 创建 ArcGIS 影像地图服务的注记图层
  return new TileLayer({
    source: arcgisImageryAnnotationSource,
  });
}


const createMap = function (){
  const map = new Map({
    target: 'map',
    layers: [],
    view: new View({
      // 用于将经度和纬度坐标（经纬度）转换为地图投影坐标
      // center: proj.fromLonLat([115.89, 28.68]),
      projection: 'EPSG:3857', // 使用Web Mercator投影坐标系
      center: proj.transform([115.89, 28.58], 'EPSG:4326', 'EPSG:3857'),
      zoom: 12,
    }),
    controls:  // 控件类
    // ol.control.defaults() 返回一个包含默认控件的数组
        defaults().extend([
          new ScaleLine(), // 比例尺
          new Zoom(),            // 添加缩放控件
          new ZoomSlider(),      // 添加缩放滑块控件
          new Rotate(),          // 添加旋转控件
          new FullScreen(),      // 添加全屏控件
          new MousePosition({    // 添加鼠标位置控件
            coordinateFormat: olCoordinate.createStringXY(4),
            projection: 'EPSG:4326',
            className: 'custom-mouse-position',
            target: document.getElementById('mouse-position'),
            undefinedHTML: '&nbsp;'
          })
        ])
  });

  console.log('map', map);
  // switchProject(source);

  return map;
}

// 监听选择项的变化
watch(selectVal, (newVal, oldVal)=>{
  console.log('watch', newVal, oldVal);
  let layerCollection = map.getLayers();
  layerCollection.forEach(layer=>{
    map.removeLayer(layer);
  })

  let layer = layers.find(layer=>layer.get('id') === newVal);
  map.addLayer(layer);
  if(newVal === 'World_Imagery') {
    let layer = getAnnotationLayer();
    map.addLayer(layer);
  }
})

</script>

<template>
  <el-row>
    <el-col :span="8">
      arcgis地图服务
    </el-col>
    <el-col :span="16">
      选择地图服务类型：
      <el-select v-model="selectVal" class="m-2" placeholder="Select">
        <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
        />
      </el-select>
    </el-col>
  </el-row>


  <div id="map">

  </div>
</template>

<style scoped>
  .header{
    height: 32px;
  }
  #map {
    height: calc(100% - 32px);
    border: 1px solid blue;
  }
</style>
<style>
.ol-zoom{
  top: 1.5em !important;
}
</style>
<script setup>
import {ref, watch} from 'vue';
import { onMounted } from 'vue';
import { Map, View } from 'ol';
import OSM from 'ol/source/OSM.js';
import TileLayer from 'ol/layer/Tile.js';
import VectorLayer from 'ol/layer/Vector';
import VectorImageLayer from 'ol/layer/VectorImage.js';
import { Vector, TileWMS } from 'ol/source';
import {Circle, Fill, Stroke, Style, Text} from 'ol/style';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import XYZ from 'ol/source/XYZ.js';
import * as proj from 'ol/proj';
import ScaleLine from 'ol/control/ScaleLine'
import Zoom from 'ol/control/Zoom';
import ZoomSlider from 'ol/control/ZoomSlider';
import Rotate from 'ol/control/Rotate';
import FullScreen from 'ol/control/FullScreen';
import MousePosition from 'ol/control/MousePosition';
import * as olCoordinate from 'ol/coordinate';
import {defaults as interactionDefaults} from 'ol/interaction/defaults';
import {defaults} from 'ol/control/defaults';
import GeoJSON from 'ol/format/GeoJSON.js';
import TileGrid from 'ol/tilegrid/TileGrid';
import TileImage from 'ol/source/TileImage';


let map = null;
let source = null;
onMounted(()=>{
  map = createMap()
  window.map = map;
});

// 创建百度地图源
let resolutions = [];
for (let i = 0; i < 20; i++) {
  resolutions[i] = Math.pow(2, 18 - i);
}

let tilegrid = new TileGrid({
  origin: [0, 0],
  resolutions: resolutions,
  tileSize: [256, 256],
});

const baiduMapLayer = new TileLayer({
  title: '百度地图瓦片图',
  source: new TileImage({
    projection: 'EPSG:3857',
    tileGrid: tilegrid,
    tilePixelRatio: devicePixelRatio,
    tileUrlFunction: function (tileCoord, pixelRatio, proj) {
      if (!tileCoord) {
        return '';
      }
      let z = tileCoord[0];
      let x = tileCoord[1];
      let y = -tileCoord[2] - 1;

      if (x < 0) {
        x = 'M' + (-x);
      }

      if (y < 0) {
        y = 'M' + (-y);
      }

      return 'https://maponline0.bdimg.com/tile/?qt=vtile&x=' + x + '&y=' + y + '&z=' + z + '&styles=pl&scaler=1&udt=20210730&from=jsapi3_0';
    }
  })
});


// 创建百度影像图层源
const baiduSource = new TileImage({
  projection: 'EPSG:3857',
  tileGrid: new TileGrid({
    origin: [0, 0],
    resolutions: (function () {
      const res = [];
      for (let i = 0; i < 19; i++) {
        res[i] = Math.pow(2, 18 - i);
      }
      return res;
    })(),
    tileSize: [256, 256],
  }),
  tileUrlFunction: function (tileCoord, pixelRatio, proj) {
    if (!tileCoord) {
      return '';
    }
    const z = tileCoord[0];
    let x = tileCoord[1];
    let y = -tileCoord[2] - 1;

    if (x < 0) {
      x = 'M' + (-x);
    }

    if (y < 0) {
      y = 'M' + (-y);
    }

    // https://maponline0.bdimg.com/starpic/?qt=satepc&u=x=6287;y=1621;z=15;v=009;type=sate&fm=46&udt=20231212
    return `https://maponline0.bdimg.com/starpic/?qt=satepc&u=x=${x};y=${y};z=${z};v=009;type=sate&fm=46&udt=20231212`;
  },
});

// 创建百度影像图层
const baiduLayer2 = new TileLayer({
  source: baiduSource,
});

// 标注图层
const baiduSourceLabel = new TileImage({
  projection: 'EPSG:3857',
  tileGrid: new TileGrid({
    origin: [0, 0],
    resolutions: (function () {
      const res = [];
      for (let i = 0; i < 19; i++) {
        res[i] = Math.pow(2, 18 - i);
      }
      return res;
    })(),
    tileSize: [256, 256],
  }),
  tileUrlFunction: function (tileCoord, pixelRatio, proj) {
    if (!tileCoord) {
      return '';
    }
    const z = tileCoord[0];
    let x = tileCoord[1];
    let y = -tileCoord[2] - 1;

    if (x < 0) {
      x = 'M' + (-x);
    }

    if (y < 0) {
      y = 'M' + (-y);
    }
    return `https://maponline2.bdimg.com/tile/?qt=vtile&x=${x}&y=${y}&z=${z}&styles=sl&v=083&udt=20231212$scaler=1&showtext=1`;
    // return `https://maponline0.bdimg.com/starpic/?qt=satepc&u=x=${6287};y=${1621};z=${15};v=009;type=sate&fm=46&udt=20231212`;
  },
});

// 创建百度影像图层
const baiduLayerLabel = new TileLayer({
  source: baiduSourceLabel,
});



// 创建一个GeoJSON格式的要素
// var geojsonObject = {
//   'type': 'FeatureCollection',
//   'features': [{
//     'type': 'Feature',
//     'geometry': {
//       'type': 'Point',
//       'coordinates': [0, 0]  // 这里是经度和纬度
//     },
//     'properties': {
//       'name': 'Null Island'
//     }
//   }]
// };
const createMap = function (){
  // 添加矢量图层
  // 矢量数据在客户端渲染成图像。该图层类型在平移和缩放期间提供了出色的性能，但是点符号和文本始终与视图一同旋转，
  // 并且在缩放动画期间像素被缩放
  source = new Vector({
    attributions: 'VectorImageLayer',
    url: '/src/data/layers/7day-M2.5.json',
    format: new GeoJSON()
  });
  let vectorLayer2 = new VectorImageLayer({
    title: 'Earthquakes2',
    source,
    style: new Style({
      image: new Circle({
        radius: 3,
        fill: new Fill({color: 'blue'})
      })
    })
  });

  const map = new Map({
    target: 'map',
    layers: [
      baiduLayer2,
      baiduLayerLabel,
      // vectorLayer2
    ],
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

const switchProject = function(source){
  // 等待 GeoJSON 加载完成后执行坐标转换 地理坐标系转投影坐标系
  source.once('change', function () {
    // 获取所有要素
    let features = source.getFeatures();
    console.log('features', features);

    // 遍历每个要素
    features.forEach(function (feature) {
      // 获取要素的几何对象
      let geometry = feature.getGeometry();

      // 转换几何对象的坐标
      geometry.transform('EPSG:4326', 'EPSG:3857');
    });
  });
}


</script>

<template>
  <header class="header">
    加载百度地图服务： K9aUq8K5Npjq2HNdfiTXTlQeOk2hYmGq
  </header>
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
<script setup>
import {ref, watch} from 'vue';
// import TheWelcome from '../components/TheWelcome.vue'
import { onMounted } from 'vue';
import Map from 'ol/Map.js';
import OSM from 'ol/source/OSM.js';
import TileLayer from 'ol/layer/Tile.js';
import VectorLayer from 'ol/layer/Vector';
import VectorImageLayer from 'ol/layer/VectorImage.js';
import View from 'ol/View.js';
import { Vector, TileWMS } from 'ol/source';
import {Circle, Fill, Stroke, Style, Text} from 'ol/style';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import BingMaps from 'ol/source/BingMaps.js';
import XYZ from 'ol/source/XYZ.js';
// import proj from 'ol/proj.js';
import * as proj from 'ol/proj';
import ScaleLine from 'ol/control/ScaleLine'
import Control from 'ol/control/Control';
import Zoom from 'ol/control/Zoom';
import ZoomSlider from 'ol/control/ZoomSlider';
import Rotate from 'ol/control/Rotate';
import FullScreen from 'ol/control/FullScreen';
import MousePosition from 'ol/control/MousePosition';
import * as olCoordinate from 'ol/coordinate';
import Select from 'ol/interaction/Select';
import Modify from 'ol/interaction/Modify';
import {defaults as interactionDefaults} from 'ol/interaction/defaults';

import Draw from 'ol/interaction/Draw';
import {defaults} from 'ol/control/defaults';
import GeoJSON from 'ol/format/GeoJSON.js';


const selectVal = ref('Point');
const options = [
  {
    value: 'Point',
    label: '点',
  },
  {
    value: 'LineString',
    label: '线',
  },
  {
    value: 'Polygon',
    label: '面',
  }
]

let map = null;
let source = null;
onMounted(()=>{
  map = createMap()
  window.map = map;
});


const getStyle = function () {
  let ArrColor = ["#990033","#FF3399","#FF66FF","#CC33FF","#660099","#333366","#0066CC","#99CCFF","#663399","#0033FF"];
  let radius = 5;
  let index = Math.floor(10*Math.random());
  let style = new Style({
    fill: new Fill({
      color: 'rgba(255, 255, 255, 0.8)'
    }),
    image: new Circle({
      radius: radius ,
      fill: new Fill({
        color: ArrColor[index]
      }),
      stroke: new Stroke({
        color: ArrColor[index],
        width: 10
      })
    }),

    text: new Text({ //文本样式
      font: '12px Calibri,sans-serif',  //字体
      text:'目标点'  , //
      offsetX: -30 ,
      offsetY: 10 ,
      rotation: 1.57 ,  //顺时针水平方向起算，弧度制
      textAlign: 'center' ,
      textBaseline: 'top' ,  //文本基线
      fill:new Fill({ color:'blue'}) ,  //文字填充色
      stroke: new Stroke({   //文字边界宽度与颜色
        color:'#fff',
        width:5
      })
    })

  });

  return style;
}

const getFeatures = function(){

  let features = new Array(10);
  let x ;
  let y ;
  for(let n=0; n<10; n++) {
    x = Math.random()*10000000;
    y = Math.random()*10000000;
    features[n] = new Feature({
      geometry: new Point([x, y]),
    });
    features[n].setStyle(getStyle()); // 分别为每个点设置样式
  }

  return features;
}

const getDraw = function (type = 'Point'){
  return new Draw({
    source: source,
    type
  })
}
let draw = getDraw();
// 监听选择项的变化
watch(selectVal, (newVal, oldVal)=>{
  map.removeInteraction(draw);
  draw = getDraw(newVal);
  map.addInteraction(draw);
})

const createMap = function (){
  const featureLayer = new VectorLayer({
    source: new Vector({
      features: getFeatures()
    })
  })

  const titleLayer = new TileLayer({
    source: new XYZ({
      attributions:
          'Tiles © <a href="https://services.arcgisonline.com/ArcGIS/' +
          'rest/services/World_Topo_Map/MapServer">ArcGIS</a>',
      url:
          'https://server.arcgisonline.com/ArcGIS/rest/services/' +
          'World_Topo_Map/MapServer/tile/{z}/{y}/{x}',
    }),
  });

  // 添加矢量图层
  let vectorLayer1 = new VectorLayer({
    title: 'Earthquakes',
    source: new Vector({
      url: '/src/data/layers/7day-M2.5.json',
      format: new GeoJSON()
    }),
    style: new Style({
      image: new Circle({
        radius: 3,
        fill: new Fill({color: 'red'})
      })
    })
  })
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

  const select = new Select({
    style: new Style({
      image: new Circle({
        radius: 10,
        fill: new Fill({
          color: '#FF0000'
        }),
        stroke: new Stroke({
          color: '#000000'
        })
      }),
      zIndex: 1000
    })
  });

  const modify = new Modify({
    features: select.getFeatures()
  });

  const map = new Map({
    target: 'map',
    layers: [
      titleLayer,
      vectorLayer2
    ],
    view: new View({
      // 用于将经度和纬度坐标（经纬度）转换为地图投影坐标
      center: proj.fromLonLat([115.89, 28.68]),
      zoom: 2,
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
        ]),
    // 交互类
    interactions: interactionDefaults().extend([
      select,
      modify,
      // draw
    ])
  });

  console.log('map', map);

  return map;
}


</script>

<template>
  <header>
    选择绘图类型：
    <el-select v-model="selectVal" class="m-2" placeholder="Select">
      <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value"
      />
    </el-select>
  </header>
  <div id="map">

  </div>
<!--  <div id="scale-line" class="scale-line"></div>-->
</template>

<style scoped>
#map {
  height: 100%;
  border: 1px solid blue;
}
</style>
<!--<style>-->
<!--  .ol-scale-line, .ol-scale-line:not([ie8andbelow]) {-->
<!--    background: black;-->
<!--    padding: 5px;-->
<!--  }-->
<!--</style>-->
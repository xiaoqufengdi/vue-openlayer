<script setup>
import {ref, watch} from 'vue';
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
import Draw from 'ol/interaction/Draw';
import {defaults} from 'ol/control/defaults';

let map = null;
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
const source = new Vector({wrapX: false});
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
  const titleLayer = new TileLayer({
    source: new XYZ({  // arcgis在线地图
      attributions:
          'Tiles © <a href="https://services.arcgisonline.com/ArcGIS/' +
          'rest/services/World_Topo_Map/MapServer">ArcGIS</a>',
      url:
          'https://server.arcgisonline.com/ArcGIS/rest/services/' +
          'World_Topo_Map/MapServer/tile/{z}/{y}/{x}',
    }),
  });
  const vector = new VectorLayer({
    source: source,
  });

  const map = new Map({
    target: 'map',
    layers: [
      titleLayer,
      vector
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
      draw
    ])
  });

  console.log('map', map);
  console.log(map.getAllLayers()[1].getSource().getFeatures());

  return map;
}

onMounted(()=>{
  map = createMap()
  window.map = map;
});

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
</template>

<style scoped>
#map {
  height: 100%;
  border: 1px solid blue;
}
</style>

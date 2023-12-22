<script setup>
// import TheWelcome from '../components/TheWelcome.vue'
import { onMounted } from 'vue';
import Map from 'ol/Map.js';
import OSM from 'ol/source/OSM.js';
import TileLayer from 'ol/layer/Tile.js';
import VectorLayer from 'ol/layer/Vector';
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
import {defaults} from 'ol/control/defaults';
import Attribution from 'ol/control/Attribution.js';


let map = null;
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

const createMap = function (){
  const featureLayer = new VectorLayer({
    source: new Vector({
      features: getFeatures()
    })
  })

  // const titleLayer = new TileLayer({
  //   source: new XYZ({
  //     url:  'https://a.tile.geofabrik.de/549e80f319af070f8ea8d0f149a149c2/{z}/{y}/{x}'
  //     // https://tile.openstreetmap.org/19/420803/208977.png
  //   }),
  // })
  // const titleLayer = new TileLayer({
  //   title: "Global Imagery",
  //   source: new TileWMS({
  //     // url: 'http://maps.opengeo.org/geowebcache/service/wms',
  //     // params: {LAYERS: 'openstreetmap', VERSION: '1.1.1'}
  //     url: 'http://suite.opengeo.org/geoserver/wms',
  //     params: {LAYERS: 'opengeo:countries', VERSION: '1.1.1'}
  //   })
  // });

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

  const map = new Map({
    target: 'map',
    layers: [
      titleLayer,
      featureLayer
    ],
    view: new View({
      // center: [-1000000, 4600000],
      // center: proj.fromLonLat([1.72, 52.4]),
      // 地理坐标系转墨卡托坐标系
      center: proj.transform([115.89, 28.68], 'EPSG:4326', 'EPSG:3857'),
      zoom: 12,
    }),
    // ol.control.Attribution控件在页面上添加了一个i（information）按钮，点击即可显示归属地信息
    // 隐藏i按钮，直接铺出来info
    Controls: [defaults({
      attributionOptions: {
        collapsible: false,
        collapsed: false
      }
    })]
  });

  console.log('map', map);
  map.addControl(new ScaleLine());
  return map;
}


</script>

<template>
  <div id="map">

  </div>
</template>

<style scoped>
#map {
  height: 100%;
  border: 1px solid blue;
}
</style>

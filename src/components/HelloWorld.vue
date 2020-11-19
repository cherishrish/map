<template>
  <div class="hello">
    <div id="cesiumContainer"></div>
    <div class="operation">
      <div class="operation-item">
          <a href="javascript:" @click="onOperationLayerShow">
            <span class="change-color">
            <i class="right-icon"/>数据图层
            </span>
          </a>
      </div>
      <!-- 地图资讯-->
      <div class="operation-item">
        <a href="javascript:" @click="onOperationMapInfoShow">
          <span class="change-color">
            <i class="right-icon ivu-icon el-icon-tickets"/>地图资讯
          </span>
        </a>
      </div>
    </div>
    <el-dialog
      align="left"
      :visible.sync="mapInfo.confirm"
      width="30%"
      :destroy-on-close="true">
<!--      设置对话框标题样式-->
      <template slot="title">
        <div style="font-size: 18px;color: #81d8d0;font-weight: bold;text-align: center">提示</div>
      </template>
      <span>
        确定打开地图资讯吗？这将重置地图（清除标注、标绘、业务绘图等）再加载资讯信息？
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="mapInfo.confirm = false" size="mini">取 消</el-button>
        <el-button type="primary" @click="onOperationMapInfoSwitch(true)" size="mini">确 定</el-button>
      </span>
    </el-dialog>
    <MapInfo v-if="mapInfo.show" :pointShow="pointShow" :port="port" :point="point" @subject-click="onOperationMapInfoSubjectClick" @place-click="onOperationPoint"></MapInfo>

    <WebEarthF1_MapInfo_Dialog v-if="mapInfo.showDialog"
                               :id="mapInfo.id" :place="mapInfo.place" :lng="mapInfo.lng" :lat="mapInfo.lat"
                               @close="onOperationMapInfoSubjectClose"/>

    <MapLayer v-if="layerShow" :point="point" @heat-click="onHeatClick" @bar-click="onBarClick"/>
    <bar v-if="barShow" :objectData = "objectData"></bar>
  </div>


</template>

<script>

import '../../node_modules/cesium/Build/Cesium/Widgets/widgets.css'
import MapInfo from "./MapInfo"
import MapLayer from './MapLayer'
import WebEarthF1_MapInfo_Dialog from "./WebEarthF1_MapInfo_Dialog";
import bar from "./bar"
import * as Cesium from '../../static/Cesium/Cesium'

export default {
  name: 'HelloWorld',
  components:{
    MapInfo,
    WebEarthF1_MapInfo_Dialog,
    MapLayer,
    bar,
  },

  data () {
    return {
      viewer: '',
      mapInfo: {
        confirm: false,
        confirmNext: false,

        show: false,
        showDialog: false,
        id: 0,
        place: undefined,
        lng: undefined,
        lat: undefined,
      },
      objectData:{
        port:'',
        barType:1,
        inPort:[],
        outPort:[]
      },
      layerShow:false,
      port:'',
      pointShow: false,
      point:[],
      entities:[],
      barShow:false,
    }
  },

  created(){

  },

  mounted () {
    let me = this;
    this.init();
    new Cesium.ScreenSpaceEventHandler(this.viewer.canvas).setInputAction(function (e) {
      var earthPosition = me.viewer.scene.pickPosition(e.position);
      var entity = me.viewer.scene.pick(e.position);
      if(Cesium.defined(earthPosition)){
        var cartographic1 = me.viewer.scene.globe.ellipsoid.cartesianToCartographic(earthPosition);
        var log = Cesium.Math.toDegrees(cartographic1.longitude).toFixed(5);//经度
        var lat = Cesium.Math.toDegrees(cartographic1.latitude).toFixed(5);//纬度

        me.viewer.camera.flyTo({   //相机飞往该点
          destination: new Cesium.Cartesian3.fromDegrees(parseFloat(log), parseFloat(lat), 700), //摄像机的最终位置
        })
        me.pointShow = true;
        me.port = entity.id.name;

        console.log(entity);
      }
    }, Cesium.ScreenSpaceEventType.LEFT_CLICK)
  },
  methods: {
    init () {

      this.viewer = new Cesium.Viewer('cesiumContainer', {
        geocoder: false, // 地理位置查询定位控件
        homeButton: false, // 默认相机位置控件
        timeline: false, // 时间滚动条控件
        navigationHelpButton: false, // 默认的相机控制提示控件
        fullscreenButton: false, // 全屏控件
        scene3DOnly: true, // 每个几何实例仅以3D渲染以节省GPU内存
        baseLayerPicker: false, // 底图切换控件
        animation: false, // 控制场景动画的播放速度控件
        infoBox: false,
        selectionIndicator: false,
      })

      this.viewer.camera.setView({
        destination: new Cesium.Cartesian3.fromDegrees(114.28088, 30.55711, 6071.32), // eslint-disable-next-line
        orientation: {
          heading: Cesium.Math.toRadians(356.7),
          pitch: Cesium.Math.toRadians(-61),
          roll: 0.0
        }
      })
      this.viewer._cesiumWidget._creditContainer.style.display = 'none'
    },

    onOperationMapInfoShow() {
      if (this.mapInfo.show) {
        this.onOperationMapInfoSwitch(false);
        return;
      }
      this.mapInfo.confirm = true;
    },

    onOperationMapInfoSwitch(show) {
      if(show){
        this.$axios.get('http://localhost:8080/static/db.json').then(res=>{
          this.point = res.data;
          console.log(this.point);
          for(var i=0; i<this.point.length;i++){
            var board = this.viewer.entities.add({
              name: this.point[i].port,
              position: new Cesium.Cartesian3.fromDegrees(this.point[i].lng, this.point[i].lon),
              billboard: {
                image: require("../assets/marker3.png"),
                scale: 1,
                horizontalOrigin: Cesium.HorizontalOrigin.CENTER,
                verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
                heightReference: Cesium.HeightReference.CLAMP_TO_GROUND
              },
            });
            this.entities.push(board);
          }
          this.mapInfo.confirm = false;
          this.mapInfo.show = true;
        }).catch(error=>{
          console.log('error')
        })
      }else{
        this.mapInfo.show = false;
        var primitives = this.viewer.entities;
        for (var i = 0; i <  this.entities.length; i++) {
          primitives.remove(this.entities[i]);
        }
        this.entities=[];
      }

    },
    onOperationMapInfoSubjectClose() {
      this.mapInfo.showDialog = false;
    },
    onOperationMapInfoSubjectClick(item, port) {
      this.mapInfo.showDialog = true;
      this.mapInfo.id = item.id;
      this.mapInfo.place = port;
      this.mapInfo.lng = item.lng;
      this.mapInfo.lat = item.lon;
    },
    onOperationPoint(item){
      this.viewer.camera.flyTo({   //相机飞往该点
        destination: new Cesium.Cartesian3.fromDegrees(item.lng, item.lon, 600), //摄像机的最终位置
      })
      this.pointShow = true;
      this.port = item.port;
    },
    onOperationLayerShow(){
      if(this.layerShow){
        this.layerShow = false;
        this.barShow = false;
      }else{
        this.layerShow = true;
      }
    },
    onHeatClick(item){
      // 矩形坐标
      var east = parseFloat(item.lng)+0.0002;
      var west = parseFloat(item.lng)-0.0002;
      var north = parseFloat(item.lon)+0.0002;
      var south = parseFloat(item.lon)-0.0002;

      var bounds = {
          west: west, south: south, east: east, north: north
      };

// 初始化CesiumHeatmap
      var heatMap = CesiumHeatmap.create(
        this.viewer, // 视图层
        bounds, // 矩形坐标
        { // heatmap相应参数
          backgroundColor: "rgba(0,0,0,0)",
          radius: 50,
          maxOpacity: .5,
          minOpacity: 0,
          blur: .75
        }
      );

      // 添加数据 最小值，最大值，数据集
      heatMap.setWGS84Data(0, 100, this.getData(50,east,west,north,south));

      this.viewer.camera.flyTo({
        destination:Cesium.Cartesian3.fromDegrees(parseFloat(item.lng),parseFloat(item.lon),200),
        orientation: {
          heading: Cesium.Math.toRadians(0.0),
          pitch: Cesium.Math.toRadians(-90.0),
          roll: 0.0
        }
      });
    },

    onBarClick(item,index){
      this.$axios.get('http://localhost:8080/static/chart.json').then(res=>{
        this.barShow = true;
        var portList = res.data;
        console.log(portList[0])
        for(var i=0;i<portList.length;i++){
          if(portList[i].port==item.port){
            var bar =  portList[i];
            this.objectData.port = item.port;
            this.objectData.barType = index;
            this.objectData.inPort = bar.in;
            this.objectData.outPort = bar.out;
            console.log(this.objectData)
            break;
          }
        }

      }).catch(error=>{
        console.log('error')
      })
    },

    getData(length,east,west,north,south) {
      var data = [];
      for (var i = 0; i < length; i++) {
        var x = Math.random() * ( east -west) + west;
        var y = Math.random() * ( north - south) + south;
        var value = Math.random() * 100;
        data.push({ x: x, y: y, value: value });
      }
      return data;
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" src="./WebEarthF1.less">

</style>

<template>
    <div>
      <div class="mapInfo_view">
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th width="30%">港口</th>
              <th width="20%">热力图</th>
              <th width="50%">进出港口货物统计图</th>
            </tr>
            <tr v-for="(item, index) in statList" :key="index">
              <td>{{item.port}}</td>
              <td>
                <a href="javascript:;" @click="onShowClick(item)">前往</a>
              </td>
              <td>
                <a href="javascript:;" @click="onChartClick(item,1)">折线图</a>
                <a href="javascript:;" @click="onChartClick(item,2)">柱状图</a>
              </td>
            </tr>
          </table>
        </div>
      </div>
    </div>
</template>

<script>
  import * as Cesium from '../../static/Cesium/Cesium'
  export default {
    name: 'MapInfo',
    props:['port'],
    data(){
      return{
        // pointShow: false,
        statList:[],
        activeIndex:-1,
        show:false,
        displayDataLayerItem: undefined
      }
    },
    mounted(){
      this.$axios.get('http://localhost:8080/static/state.json').then(res=>{
        this.statList = res.data;
        // for(var i=0;i<this.statList.length;i++){
        //   var value = false;
        //   this.statList[i]["show"]=value;
        // }
        console.log(this.statList);
      }).catch(error=>{
        console.log('error')
      })
    },
    methods:{
      onShowClick(item) {
        this.$emit('heat-click', item);
      },
      onChartClick(item,index){
        console.log(item)
        this.$emit('bar-click', item,index);
      }
    }
  }
</script>

<style scoped lang="less">
  /*修改a标签的颜色*/
  a{
    color: #515a6e;
  }

  a:hover{
    color: #81d8d0;
  }

  .mapInfo_view {
    position: absolute;
    bottom: 10px;
    left: 96px;
    width: 800px;
    display: block;
  }

  .mapInfo_view_point {
    position: absolute;
    top: 75px;
    left: 20px;
    /*修改宽度*/
    width: 300px;
    display: block;
  }

  .mapInfo_view_table {
    display: block;
    width: 100%;
    padding: 4px;
    margin-bottom: 10px;
    /*去掉外边框*/
    /*border: 2px solid #f7f7f7;*/
    border-radius: 4px;
    background-color: rgba(255, 255, 255, 0.9);
    box-shadow: 1px 2px 4px #11035440;

    table th {
      font-weight: bold;
    }

    table td[align='left'] a {
      width: 300px;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      display: block;
    }
  }
</style>

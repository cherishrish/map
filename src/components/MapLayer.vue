<template>
    <div>
      <div class="mapInfo_view">
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th>热力图展示</th>
              <th width="120">操作</th>
            </tr>
            <tr v-for="(item, index) in statList" :key="index">
              <td>{{item.port}}</td>
              <td>
                <a href="javascript:;" @click="onShowClick(item)">前往</a>
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
    }
  }
</script>

<style scoped lang="less">
  .mapInfo_view {
    position: absolute;
    bottom: 10px;
    left: 96px;
    width: 460px;
    display: block;
  }

  .mapInfo_view_point {
    position: absolute;
    top: 75px;
    left: 20px;
    width: 375px;
    display: block;
  }

  .mapInfo_view_table {
    display: block;
    width: 100%;
    padding: 4px;
    margin-bottom: 10px;
    border: 2px solid #f7f7f7;
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

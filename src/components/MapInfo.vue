<template>
    <div>
      <div class="mapInfo_view_point" v-show="pointShow">
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th width="300">地点主题列表</th>
              <th>时间</th>
            </tr>
            <tr v-for="(item, index) in point" :key="index" >
              <td v-if="item.port==port" align="left" width="300">
                <a href="javascript:;" @click="onSubjectClick(item, item.port)"
                   :title="item.subject">{{item.subject}}</a>
              </td>
              <td v-if="item.port==port">{{formatRDate(item.subject_time)}}</td>
            </tr>
          </table>
        </div>
      </div>
      <div class="mapInfo_view">
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th width="60%">最新主题</th>
              <th width="20%">地点</th>
              <th width="20%">时间</th>
            </tr>
            <tr v-for="(item, index) in point" :key="index">
              <td align="left">
                <a href="javascript:;" @click="onSubjectClick(item, item.port)" :title="item.subject">{{item.subject}}</a>
              </td>
              <td>
                <a href="javascript:;" @click="onPlaceClick(item)">{{item.port}}</a>
              </td>
              <td>{{formatRDate(item.subject_time)}}</td>
            </tr>
          </table>
        </div>
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th width="60%">热门地点</th>
              <th width="40%">主题数</th>
            </tr>
            <tr v-for="(item, index) in statList" :key="index">
              <td>
                <a href="javascript:;" @click="onPlaceClick(item)">{{item.port}}</a>
              </td>
              <td>
                <a href="javascript:;" @click="onPlaceClick(item)">{{item.c}}</a>
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
    props:['pointShow','port','point'],
    data(){
      return{
        // pointShow: false,
        pointList:[],
        statList:[]
      }
    },
    mounted(){
      this.$axios.get('http://localhost:8080/static/state.json').then(res=>{
        this.statList = res.data;
        console.log(this.statList);
      }).catch(error=>{
        console.log('error')
      })
    },
    methods:{
      formatRDate(date) {
        let d = new Date(date);
        let now = new Date().getTime();
        let tick = d.getTime();

        let minute = 1000 * 60;
        let hour = minute * 60;
        let day = hour * 24;
        let week = day * 7;
        let month = day * 30;
        let diff = now - tick;

        let result = null;
        if (diff < 0) {
          result = this.formatDate(date);
        } else if (diff / month >= 1) {
          result = Math.ceil(diff / month) + "月前";
        } else if (diff / week >= 1) {
          result = Math.ceil(diff / week) + "周前";
        } else if (diff / day >= 1) {
          result = Math.ceil(diff / day) + "天前";
        } else if (diff / hour >= 1) {
          result = Math.ceil(diff / hour) + "小时前";
        } else if (diff / minute >= 1) {
          result = Math.ceil(diff / minute) + "分钟前";
        } else {
          result = "刚刚";
        }

        return result;
      },

      onPlaceClick(item) {
        this.$emit('place-click',item);
      },

      onSubjectClick(item, port) {
        console.log(this.port)
        this.onPlaceClick(item);
        this.$emit('subject-click', item, port);
      },
    }
  }
</script>

<style scoped lang="less">
  /*修改新闻列表中的链接颜色*/
  a{
    color: #515a6e;
  }

  a:hover{
    color: #81d8d0;
  }

  .mapInfo_view {
    position: absolute;
    bottom: 10px;
    right: 96px;
    width: 460px;
    display: block;
  }

  .mapInfo_view_point {
    position: absolute;
    top: 75px;
    /*将地点主题列表置于右侧选择按钮下*/
    right: 96px;
    width: 460px;
    display: block;
  }

  .mapInfo_view_table {
    display: block;
    width: 100%;
    padding: 4px;
    margin-bottom: 10px;
    /*删除列表白色边框*/
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

<template>
    <div>
      <div class="mapInfo_view_point" v-show="portShow">
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th width="300">{{port}}新闻列表</th>
              <th>时间</th>
            </tr>
            <tr v-for="(item, index) in newsList" :key="index" >
              <td v-if="item.port==port" align="left" width="300">
                <a href="javascript:;" @click="onSubjectClick(item, item.port)"
                   :title="item.subject">{{item.subject}}</a>
              </td>
              <td v-if="item.port==port">{{formatRDate(item.subject_time)}}</td>
            </tr>
          </table>
        </div>
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th width="300">{{port}}数据展示</th>
              <th>操作</th>
            </tr>
            <tr>
              <td>热力图</td>
              <td>
                <a v-if="heatDisplay" href="javascript:;" @click="onHeatClick(heatDisplay)">显示</a>
                <a v-if="!heatDisplay" href="javascript:;" @click="onHeatClick(heatDisplay)">隐藏</a>
              </td>
            </tr>
            <tr>
              <td>进出港口货物统计图</td>
              <td>
                <a v-if="barDisplay" href="javascript:;" @click="onChartClick(barDisplay)">折线图</a>
                <a v-if="!barDisplay" href="javascript:;" @click="onChartClick(barDisplay)">柱状图</a>
              </td>
            </tr>
          </table>
        </div>
      </div>
      <div class="mapInfo_view">
        <div class="mapInfo_view_table">
          <table class="table" width="100%">
            <tr>
              <th width="60%">热门港口</th>
              <th width="40%">新闻数</th>
            </tr>
            <tr v-for="(item, index) in portList" :key="index">
              <td>
                <a :class="{show:item.isShow}" href="javascript:;" @click="onPlaceClick(item)">{{item.port}}</a>
              </td>
              <td>
                <a href="javascript:;" @click="onPlaceClick(item)">{{item.c}}</a>
              </td>
            </tr>
          </table>
        </div>
        <div class="mapInfo_view_table">
          <table class="table">
            <thead>
              <tr>
                <th width="60%">最新新闻</th>
                <th width="20%">港口</th>
                <th >时间</th>
              </tr>
            </thead>
            <tbody :class="{anim:animate==true, hei: isLenght===true}">
              <tr v-for="(item, index) in newsList" :key="index">
                <td width="60%" >
                  <a href="javascript:;" @click="onSubjectClick(item, item.port)" :title="item.subject">{{item.subject}}</a>
                </td>
                <td width="20%">
                  <a :class="{show:item.isShow}" href="javascript:;" @click="onPlaceClick(item)">{{item.port}}</a>
                </td >
                <td >{{formatRDate(item.subject_time)}}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
</template>

<script>
  import * as Cesium from '../../static/Cesium/Cesium'
  export default {
    name: 'MapInfo',
    props:['port','portList','viewer'],
    data(){
      return{
        pointList:[],
        newsList: [],
        heatDisplay: true,
        barDisplay: true,
        animate:false,
        timer:'',
        endId:'',
        isLenght:false,
        portShow:false,
        num:0,
        item:''
      }
    },
    created(){
      this.getList()
    },
    mounted(){
    },
    methods:{
      scroll () {
        this.animate = true
        setTimeout(() => {
          this.newsList.push(this.newsList[0])
          this.newsList.shift()
          this.animate = false
          if (this.newsList[0].id === this.endId) {
            // console.log('请求刷新数据')
            this.getList()
            clearInterval(this.timer)
          }
        }, 500)
      },

      getList(){
        this.$axios.get('http://localhost:8080/static/db.json').then(res=>{
          this.newsList=res.data;
          this.endId = this.newsList[this.newsList.length-1].id
          if (this.newsList.length > 4) {
            this.isLenght = true
            this.timer = setInterval(this.scroll, 10000)
          } else {
            this.isLenght = false
            clearInterval(this.timer)
           }
        }).catch(error=>{
          console.log('error')
        })
      },

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
        if(this.item!=''){
          this.item.isShow = false;
        }
        console.log(this.item)
        item.isShow = true;
        this.item = item;
        this.portShow = true;
        this.heatDisplay = true;
        this.barDisplay = true;
      },

      onSubjectClick(item, port) {
        this.onPlaceClick(item);
        this.$emit('subject-click', item, port);
      },

      onHeatClick(display){
        this.heatDisplay = !display;
        this.$emit('heat-click', this.port,display);
      },

      onChartClick(display){
        this.barDisplay = !display;
        var index = 2;
        if(display){
          index = 1
        }
        this.$emit('bar-click', this.port,index);
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
    left: 10px;
    width: 460px;
    display: block;
  }

  .mapInfo_view_point {
    position: absolute;
    top: 10px;
    /*将地点主题列表置于右侧选择按钮下*/
    right: 10px;
    width: 460px;
    display: block;
  }

  .show{
    color:red;
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

    .table th {
      font-weight: bold;
    }

    .table td[align='left'] a {
      width: 300px;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      display: block;
    }

    .table{
       width: 100%;
        border: 1px solid #86c7ff;
       position: relative;
    }

    .table thead {
      width: 100%;
      text-align: center;
      line-height: 40px;
      font-size: 16px;
      display: table;
      color: #000000;
      table-layout: fixed;
      border-bottom: none;
      box-sizing: border-box;
    }
    .table thead th {
      font-weight: 300;
      table-layout: fixed;
      box-sizing: border-box;
    }
    .table tbody.hei{
      height: 200px;
    }
    .table tbody {
      display: block;
      text-align: left;
      width: 100%;
      /* height: 352px; */
      /* 隐藏滚动条兼容 */
      -ms-scroll-chaining: chained;
      -ms-content-zooming: zoom;
      -ms-scroll-rails: none;
      -ms-content-zoom-limit-min: 100%;
      -ms-content-zoom-limit-max: 500%;
      /* -ms-scroll-snap-type: proximity; */
      -ms-scroll-snap-points-x: snapList(100%, 200%, 300%, 400%, 500%);
      -ms-overflow-style: none;
      overflow-y: scroll;
      /* 火狐 */
      scrollbar-width: none;
      /* ie */
      -ms-overflow-style: none;
      table-layout: fixed;
    }
    .table tbody::-webkit-scrollbar {
      display: none;
    }
    .anim{
      transition: all 0.5s;
      margin-top: -32px;
    }
  }
</style>

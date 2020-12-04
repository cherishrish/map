<template>
  <div class="map-info-detail">
<!--    <div class="btn-return">-->
<!--      <button type="button" @click="onClose">-->
<!--        <i class="el-icon-arrow-left"></i>-->
<!--      </button>-->
<!--    </div>-->

    <el-dialog align="left" :visible.sync="showDialog" :close-on-click-modal="false"
               width="60%" :show-close="false" :destroy-on-close="true">
      <!--      设置对话框title样式-->
        <template slot="title">
<!--          修改标题与顶部的距离，避免按钮与标题重叠-->
          <div style="color: black;font-size: 30px;font-weight: bold;text-align: center;margin-top:20px;">{{model.subject}}</div>
        </template>
        <div class="btn-close">
          <button type="button" @click="onClose">
            <i class="el-icon-close"></i>
          </button>
        </div>
      <div class="item info">
        地点：{{model.port}} ({{model.lng}}, {{model.lon}})
      </div>
      <div class="item info">
        时间：{{formatDate(model.subject_time)}}
      </div>
      <div class="item content">
        <div v-html="model.content"></div>
      </div>
      <div class="btn-left" v-show="hasUp">
        <button type="button" @click="onHandlePrevInfo" title="查看上一条资讯">
          <i class="el-icon-arrow-left"></i>
        </button>
      </div>
      <div class="btn-right" v-show="hasDown">
        <button type="button" @click="onHandleNextInfo" title="查看下一条资讯">
          <i class="el-icon-arrow-right"></i>
        </button>
      </div>
    </el-dialog>
  </div>
</template>

<script>

  export default {
    name: "WebEarthF1_MapInfo_Dialog",
    props: {
      id: 0,
      place: undefined,
      lat: undefined,
      lng: undefined
    },

    data() {
      return {
        showDialog: true,
        hasUp: true,
        hasDown: true,
        newsList:[],
        index:0,
        model: {
          id: '',
          lng: 0,
          lat: 0,
          place: '',
          subject: '',
          subjectTime: '',
          content: '<p></p>',
        },
        news:[]
      };
    },

    methods: {
      formatDate(date) {
        let d = new Date(date),
          month = d.getMonth() + 1,
          day = d.getDate(),
          year = d.getFullYear(),
          hour = d.getHours(),
          min = d.getMinutes();

        if (month.length < 2) month = "0" + month;
        if (day.length < 2) day = "0" + day;

        return [year, month, day].join("-") + " " + [hour, min].join(":");
      },

      onHandlePrevInfo() {
        this.loadUpDown("up");
      },

      onHandleNextInfo() {
        this.loadUpDown("down");
      },

      onClose() {
        this.showDialog = false;
        this.$emit("close");
      },

      load(id) {
        let me = this;
        this.$axios.get("http://localhost:8080/static/db.json").then(res => {
          this.newsList = res.data;
          for(var i=0;i<res.data.length;i++){
            if(id==res.data[i].id){
              me.model = res.data[i];
              console.log(me.model)
              break;
            }
          }
          for(var i=0;i<this.newsList.length;i++){
            if(me.newsList[i].port==me.model.port){
              this.news.push(me.newsList[i]);
            }
          }
          console.log(me.news);
          for(var i=0;i<me.news.length;i++){
            if(me.news[i].id==me.id){
              me.index = i;
              break;
            }
          }
          console.log(me.index);
        });
      },

      loadUpDown(type) {
        let me = this;
        if (type === "up") {
          this.index--;
          if(this.index<0){
            me.$message.info("没有上一条资讯了");
          }else {
            me.model = me.news[me.index];
          }
        }
        if (type === "down") {
          this.index++;
          if(this.index>this.news.length-1){
            me.$message.info("没有下一条资讯了");
          }else {
            me.model = me.news[me.index];
          }
        }
      }
    },

    mounted() {
      this.load(this.id)
    }
  }
</script>

<style scoped lang="less">
  .map-info-detail {
    overflow: hidden;

    div.btn-close,div.btn-more, div.btn-left, div.btn-right {
      position: absolute;
      display: block;
      z-index: 9999;

      button, a {
        display: block;
        /*修改按钮的颜色以及透明度*/
        background-color: rgba(0,0,0,0.1);
        /*光标的指针样式*/
        cursor: pointer;
        padding: 10px;
        font-size: 24px;
        /*修改按钮上的字体颜色*/
        color: #fff;
        /*去除灰色按钮周围灰色背景*/
        border: 0;
        /*几个button的样式统一，集合写在这里，简化代码*/
        width: 40px;
        height: 40px;
        /*icon居中显示*/
        line-height: 20px;
        border-radius: 50%;

        i {
          font-size: 20px;
          color: #fff;
          font-weight: bolder;
        }
      }
    }
    /*鼠标经过背景色为主色调*/
    div.btn-more, div.btn-left, div.btn-right {
      button{
        &:hover {
          background-color: #81d8d0;

          i {
            background-color: #81d8d0;
          }
        }
      }
    }
    /*鼠标经过背景色为红色警告色*/
    div.btn-close {
      top: 5px;
      right: 5px;

      button{
        &:hover {
          background-color: red;

          i {
            background-color: red;
          }
        }
      }
    }
    /*更多按钮在这里没有用上*/
    div.btn-more {
      top: 10px;
      left: 150px;

      a {
        text-decoration: none;
      }
    }

    div.btn-left {
      top: 50%;
      left: 1px;
    }

    div.btn-right {
      top: 50%;
      right: 1px;
    }

    div.item {
      margin-bottom: 5px;
      line-height: 14px;
    }

    div.info {
      font-size: 12px;
    }

    div.content {
      margin-top: 10px;
      border-top: 1px solid #6e6e6e;
      padding: 20px;
    }
  }
</style>

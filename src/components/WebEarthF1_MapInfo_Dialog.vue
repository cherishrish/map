<template>
  <div class="map-info-detail">
    <div class="btn-return">
      <button type="button" @click="onClose">
        <i class="el-icon-arrow-left"></i>
        返回
      </button>
    </div>
    <el-dialog align="left" :title="model.subject" :visible.sync="showDialog" :close-on-click-modal="false"
               width="60%" :show-close="false" :destroy-on-close="true">
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

        model: {
          id: '',
          lng: 0,
          lat: 0,
          place: '',
          subject: '',
          subjectTime: '',
          content: '<p></p>',
        }
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
          console.log(res.data[0])
          for(var i=0;i<res.data.length;i++){
            if(id==res.data[i].id){
              me.model = res.data[i];
              break;
            }
          }
        });
      },

      loadUpDown(type) {
        let me = this;
        if (type === "up") {
          me.$message.info("没有上一条资讯了");
        }
        if (type === "down") {
          me.$message.info("没有下一条资讯了");
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

    .el-dialog__body {
      margin: 0;
      padding: 0;
    }

    div.btn-return, div.btn-more, div.btn-left, div.btn-right {
      position: absolute;
      display: block;
      z-index: 9999;

      button, a {
        display: block;
        background-color: #f3f3f3;
        cursor: pointer;
        padding: 10px;

        font-size: 24px;
        color: #333;

        i {
          font-size: 24px;
          color: #333;
          font-weight: 500;
        }

        &:hover {
          color: #6850d8;

          i {
            color: #6850d8;
          }
        }
      }
    }

    div.btn-return {
      top: 10px;
      left: 40px;
    }

    div.btn-more {
      top: 10px;
      left: 150px;

      a {
        text-decoration: none;
      }
    }

    div.btn-left {
      top: 50%;
      left: -100px;

      button {
        padding: 10px;
        width: 60px;
        border-radius: 50px;
      }
    }

    div.btn-right {
      top: 50%;
      right: -100px;

      button {
        padding: 10px;
        width: 60px;
        border-radius: 50px;
      }
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

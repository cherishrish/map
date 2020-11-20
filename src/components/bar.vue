<template>
  <div class="bar" id="bar"></div>
</template>

<script>
  export default {
    name: 'bar',
    props:['objectData'],
    data(){
      return{};
    },
    watch:{
      objectData:{
        handler:function(val){
            this.$nextTick(()=>{
              console.log(val);
              this.drawLine(val)
            })
        },
        deep: true,
        immediate:true
      }
    },
    mounted(){
      this.drawLine();
    },
    methods: {
      drawLine ({port="",barType=1,inPort=[],outPort=[]}={}) {
        // 基于准备好的dom，初始化echarts实例
        let myChart = this.$echarts.init(document.getElementById('bar'))
        console.log(11)
        // 绘制图表
        myChart.setOption({
          title: {
            text: port+'进出港口货物'+(barType==1?"折线图":"柱状图"),
            left:'center'
          },
          tooltip: {},
          xAxis: {
            data: ["一月", "二月", "三月", "四月","五月","六月","七月","八月","九月","十月","十一月","十二月"]
          },
          yAxis: {
            name: '出货量',
            type: 'value',

          },
          legend: {
            x:'right',   // 图例的位置
            textStyle:{
              color:'#000000',  // 图例文字的颜色
            },
          },
          series: [{
            name: '进口',
            type: barType==1?"line":"bar",
            //修改bar的颜色
            // itemStyle:{
            //   normal:{
            //     color: 'rgb(164,205,238)',
            //   },
            // },
            data: inPort,
          },{
            name: '出口',
            type: barType==1?"line":"bar",
            //修改bar的颜色
            // itemStyle:{
            //   normal:{
            //     color: 'rgb(195,229,235)',
            //   },
            // },
            data: outPort,
          }]
        },true);

      }
    }
  }
</script>

<style scoped>
  .bar {
    position: absolute;
    top: 75px;
    left: 96px;
    width: 800px;
    height: 380px;
    display: block;
    background: rgba(255, 255, 255, 0.3);
    border-radius: 4px;
  }
</style>

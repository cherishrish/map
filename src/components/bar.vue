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
        // 绘制图表,折线图和柱状图在点击详细新闻页面后会消失
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
            name: '出货量/万吨',
            type: 'value',
          },
          legend: {
            x:'right',   // 图例的位置
            top: "8%",//避免 title 和 legend 重叠
            textStyle:{
              color:'#000000',  // 图例文字的颜色
            },
          },
          //添加数据视图和保存为图片按钮
          toolbox:{
            show:true,
            feature:{
              dataView:{show:true,readOnly:true},
              restore:{show:false},
              //保存为图片
              saveAsImage:{show:true},
            },
          },
          series: [{
            name: '进口',
            type: barType == 1 ? "line" : "bar",
            data: inPort,
            itemStyle: {
              normal: {
                color: 'rgb(135,206,250)',
              },
            },
          },
            {
            name: '出口',
            type: barType==1?"line":"bar",
            //修改bar的颜色
            itemStyle:{
              normal:{
                color: 'rgb(0,191,255)',
              },
            },
            data: outPort,
          }]
        },true);
      },
    }
  }
</script>

<style scoped>
  .bar {
    position: absolute;
    /*修改距离底部的距离，避免重叠*/
    bottom: 10px;
    right: 10px;
    /*宽度太窄会无法显示完整的纵坐标值*/
    width: 500px;
    height: 300px;
    display: block;
    background: rgba(255, 255, 255, 0.4);
    border-radius: 4px;
  }
</style>

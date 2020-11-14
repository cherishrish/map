<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
  /*初始化body*/
  body{
    margin: 0;
    padding: 0;
  }

  /*设置p标签的外间距,设置文字首行缩进*/
  p{
    margin: 5px;
    padding: 0;
    text-indent: 30px;
  }

</style>

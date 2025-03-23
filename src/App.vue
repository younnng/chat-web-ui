<template>
  <div class="app-container">
    <router-view />
  </div>
</template>

<script setup>
// App.vue作为根组件，主要负责渲染路由视图
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  height: 100%;
  width: 100%;
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', Arial, sans-serif;
}

#app, .app-container {
  height: 100%;
  width: 100%;
}
</style>
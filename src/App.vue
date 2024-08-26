<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
<style>
/* 隐藏侧边栏 */
.sidebar-container {
  display: none;
}

/* 调整主内容区域的宽度 */
.main-container {
  margin-left: 0;
}
</style>

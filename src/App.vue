<template>
  <div id="app">
    <div class="main">
      <el-menu :default-active="activeIndex" mode="horizontal" @select="handleSelect" router>
        <el-menu-item index="/">首页</el-menu-item>
        <el-submenu index="report">
          <template slot="title">用户</template>
          <el-menu-item index="/user/list">用户列表</el-menu-item>
        </el-submenu>
      </el-menu>
      <router-view/>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      activeIndex: '/',
      showMenu: true
    }
  },
  created: function () {
    this.activeIndex = this.$router.path
  },
  watch: {
    '$route' (to, from) {
      this.activeIndex = to.path
    }
  }
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
  .main{
    /* 浮动 */
    float: left;
    /* 设置宽度，注意左边导航栏是5% */
    width: 100%;
    /* 设置背景色 */
    background-color:#eff2f7;
    /*设置高  */
  }
</style>

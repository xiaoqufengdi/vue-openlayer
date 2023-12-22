<script setup>
import { RouterLink, RouterView } from 'vue-router'
// import { ElContainer, ElHeader, ElMain, ElFooter} from 'element-plus';

</script>

<template>
  <el-container class="container">
    <el-header class="header">
      <el-row class="wrapper">
        <el-col :span="24">
          <RouterLink to="/">Home</RouterLink> &nbsp;&nbsp;
          <RouterLink to="/about">About</RouterLink> &nbsp;&nbsp;
          <RouterLink to="/demo2">Demo2</RouterLink>&nbsp;&nbsp;
          <RouterLink to="/demo3">Demo3</RouterLink>
        </el-col>
      </el-row>
    </el-header>
    <el-main>
      <RouterView />
    </el-main>
<!--    <el-footer>-->
<!--      footer-->
<!--    </el-footer>-->
  </el-container>


</template>

<style scoped lang="less">

.container {
  width: 100%;
  height: 100vh;
  .header {
    height: 32px;
    border-bottom: 1px solid blue;
  }

}


</style>

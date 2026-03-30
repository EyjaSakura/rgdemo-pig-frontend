<template>
  <div class="app-container">
    <el-card>
      <h2>恭喜，前端路由通了！🎉</h2>
      
      <el-divider></el-divider>
      
      <el-button v-auth="'test_btn'" type="primary">测试按钮 (v-auth)</el-button>

      <el-button v-hasPermi="['test_btn']" type="primary">测试按钮 (v-hasPermi)</el-button>
      
    </el-card>
  </div>
</template>

<script setup name="DemoTest">
// 极其清爽，什么都不用写！
</script>

<style scoped>
.app-container { padding: 20px; }
</style>
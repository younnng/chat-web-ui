<template>
  <div class="not-found-container">
    <div class="not-found-content">
      <el-result
        icon="error"
        title="404"
        sub-title="抱歉，您访问的页面不存在"
      >
        <template #extra>
          <el-button type="primary" @click="goHome">返回首页</el-button>
        </template>
      </el-result>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from 'vue-router'

const router = useRouter()

const goHome = () => {
  router.push('/')
}
</script>

<style scoped>
.not-found-container {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--background-color);
}

.not-found-content {
  max-width: 500px;
  padding: 40px;
  text-align: center;
}
</style>
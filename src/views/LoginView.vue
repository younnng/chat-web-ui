<template>
  <div class="login-container">
    <div class="login-box">
      <div class="login-header">
        <h2>智能文档助手</h2>
        <p>欢迎使用智能文档助手</p>
      </div>
      <div class="login-form">
        <el-form :model="loginForm" :rules="loginRules" ref="loginFormRef" label-position="top">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="loginForm.username" placeholder="请输入用户名" prefix-icon="User" />
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="loginForm.password" type="password" placeholder="请输入密码" prefix-icon="Lock" show-password />
          </el-form-item>
          <el-form-item>
            <el-button type="primary" :loading="loading" class="login-button" @click="handleLogin">登录</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { User, Lock } from '@element-plus/icons-vue'
import axios from 'axios'

const router = useRouter()
const loginFormRef = ref(null)
const loading = ref(false)

// 登录表单数据
const loginForm = reactive({
  username: '',
  password: ''
})

// 表单验证规则
const loginRules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '用户名长度应为3-20个字符', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 20, message: '密码长度应为6-20个字符', trigger: 'blur' }
  ]
}

// 处理登录
const handleLogin = async () => {
  if (!loginFormRef.value) return
  
  await loginFormRef.value.validate(async (valid) => {
    if (valid) {
      loading.value = true
      
      try {
        const response = await axios.post('/api/auth/login', {
          username: loginForm.username,
          password: loginForm.password
        })
        
        // 保存token和用户信息到本地存储
        localStorage.setItem('token', response.data.data.token)
        localStorage.setItem('user', JSON.stringify(response.data.data.user))
        localStorage.setItem('isAdmin', response.data.data.user.isAdmin || false)

        ElMessage.success('登录成功')
        
        // 根据用户角色跳转到不同页面
        if (response.data.data.user.isAdmin) {
          //router.push('/admin')
          router.push('/chat')
        } else {
          router.push('/chat')
        }
      } catch (error) {
        console.error('登录失败:', error)
        
        // 显示错误信息
        if (error.response && error.response.data && error.response.data.data.message) {
          ElMessage.error(error.response.data.data.message)
        } else {
          ElMessage.error('登录失败，请检查用户名和密码')
        }
      } finally {
        loading.value = false
      }
    }
  })
}
</script>

<style scoped>
.login-container {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--background-color);
  background-image: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.login-box {
  width: 400px;
  padding: 40px;
  border-radius: 8px;
  background-color: #fff;
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
}

.login-header {
  text-align: center;
  margin-bottom: 30px;
}

.login-header h2 {
  font-size: 28px;
  color: var(--text-primary);
  margin-bottom: 10px;
}

.login-header p {
  font-size: 16px;
  color: var(--text-secondary);
}

.login-button {
  width: 100%;
  padding: 12px 0;
  font-size: 16px;
  margin-top: 10px;
}

/* 响应式调整 */
@media (max-width: 480px) {
  .login-box {
    width: 90%;
    padding: 30px 20px;
  }
}
</style>
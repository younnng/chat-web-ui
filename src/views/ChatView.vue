<template>
  <div class="chat-container">
    <!-- 顶部导航栏 -->
    <header class="chat-header">
      <div class="header-left">
        <h2>智能文档助手</h2>
      </div>
      <div class="header-right">
        <el-dropdown @command="handleCommand">
          <span class="el-dropdown-link">
            {{ userInfo.username }}<el-icon class="el-icon--right"><arrow-down /></el-icon>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item v-if="isAdmin" command="admin">管理后台</el-dropdown-item>
              <el-dropdown-item command="logout">退出登录</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </div>
    </header>

    <!-- 聊天内容区域 -->
    <main class="chat-main">
      <div class="chat-messages" ref="messagesContainer">
        <div v-for="(message, index) in messages" :key="index" :class="['message', message.role]">
          <div class="message-avatar">
            <el-avatar :icon="message.role === 'user' ? User : ChatDotRound" :size="40" />
          </div>
          <div class="message-content">
            <!-- 思考过程展示 -->
            <div v-if="message.think && message.think.trim()" class="thinking-process">
              <div class="thinking-header">思考过程</div>
              <div class="thinking-content" v-html="formatMessage(message.think)"></div>
            </div>
            <!-- 消息内容 -->
            <div class="message-text" v-html="formatMessage(message.content)"></div>
            <div class="message-time">{{ formatTime(message.timestamp) }}</div>
          </div>
        </div>
        <!-- 添加加载动画 -->
        <div v-if="loading" class="message ai">
          <div class="message-avatar">
            <el-avatar :icon="ChatDotRound" :size="40" />
          </div>
          <div class="message-content">
            <div class="thinking-animation">
              <span class="dot"></span>
              <span class="dot"></span>
              <span class="dot"></span>
              思考中...
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- 输入区域 -->
    <footer class="chat-footer">
      <div class="input-container">
        <el-input
          v-model="inputMessage"
          type="textarea"
          :rows="3"
          placeholder="请输入您的问题..."
          resize="none"
          @keydown.enter.prevent="sendMessage"
        />
        <el-button type="primary" :disabled="!inputMessage.trim() || loading" @click="sendMessage">
          发送 <el-icon><position /></el-icon>
        </el-button>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, nextTick, computed } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import { ArrowDown, User, ChatDotRound, Position } from '@element-plus/icons-vue'
import axios from 'axios'
import { marked } from 'marked'  // 可选：如果需要更复杂的markdown解析

const router = useRouter()
const messagesContainer = ref(null)
const inputMessage = ref('')
const loading = ref(false)
const messages = ref([])

// 获取用户信息
const userInfo = computed(() => {
  try {
    return JSON.parse(localStorage.getItem('user')) || { username: '用户' }
  } catch (e) {
    return { username: '用户' }
  }
})

// 判断是否为管理员
const isAdmin = computed(() => {
  return localStorage.getItem('isAdmin') === 'true'
})

// 格式化消息内容，支持简单的markdown格式
const formatMessage = (content) => {
  if (!content) return ''

  // 先进行HTML转义
  const escapeHtml = (unsafe) => {
    return unsafe.replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;")
                .replace(/"/g, "&quot;")
                .replace(/'/g, "&#039;");
  }
  
  // 将换行符转换为<br>
  let formatted = escapeHtml(content).replace(/<\/?think>/g, '')  // 移除所有think标签
        .replace(/<\/?[a-z]+>/gi, '') // 可选：移除所有其他HTML标签
  
  // 将代码块格式化
  formatted = formatted.replace(/```([\s\S]*?)```/g, '<pre class="code-block">$1</pre>')
  
  // 将粗体文本格式化
  formatted = formatted.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
  
  return formatted
}

// 格式化时间
const formatTime = (timestamp) => {
  if (!timestamp) return ''
  
  const date = new Date(timestamp)
  const hours = date.getHours().toString().padStart(2, '0')
  const minutes = date.getMinutes().toString().padStart(2, '0')
  
  return `${hours}:${minutes}`
}

// 发送消息
const sendMessage = async () => {
  const message = inputMessage.value.trim()
  if (!message || loading.value) return
  
  // 添加用户消息到列表
  messages.value.push({
    role: 'user',
    content: message,
    timestamp: new Date().toISOString()
  })
  
  inputMessage.value = ''
  loading.value = true
  
  // 滚动到底部
  await nextTick()
  scrollToBottom()
  
  try {
    // 发送请求到后端
    // 使用fetch API处理流式响应
    const response = await fetch('/api/chat/stream/message', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${localStorage.getItem('token')}`
      },
      body: JSON.stringify({
        message,
        userId: userInfo.value.id
      })
    })

    // 创建临时消息对象
    const tempMessage = {
      role: 'ai',
      content: '',
      think: '',      // 新增思考过程字段
      timestamp: new Date().toISOString()
    }
    messages.value.push(tempMessage)
    
    // 创建读取器
    const reader = response.body.getReader()
    const decoder = new TextDecoder()
    let isThinking = false

    // 流式数据处理
    while (true) {
      const { done, value } = await reader.read()
      if (done) break
      
      const chunk = decoder.decode(value)
      // 处理思考标签
      // 优化标签处理逻辑
      if (chunk.includes('<think>')) {
        isThinking = true
        tempMessage.think += chunk.replace(/<\/?think>/g, '') // 使用正则移除所有标签
        continue
      }

      if (chunk.includes('</think>')) {
        isThinking = false
        tempMessage.think += chunk.replace(/<\/?think>/g, '')
        // 自动清除空思考内容
        if (tempMessage.think.trim() === '') {
        tempMessage.think = null
      }
        continue
      }
      
      if (isThinking) {
        tempMessage.think += chunk.replace(/<\/?think>/g, '') // 防御性处理
      } else {
        tempMessage.content += chunk.replace(/<\/?think>/g, '') // 清除残留标签
      }
        
      // 触发视图更新
      messages.value = [...messages.value]
      scrollToBottom()
    }
    
    // 滚动到底部
    await nextTick()
    scrollToBottom()
  } catch (error) {
    console.error('发送消息失败:', error)
    ElMessage.error('发送消息失败，请重试')
    
    // 如果是401错误，可能是token过期，重定向到登录页
    if (error.response && error.response.status === 401) {
      ElMessageBox.confirm(
        '登录已过期，请重新登录',
        '提示',
        {
          confirmButtonText: '确定',
          type: 'warning',
          showCancelButton: false
        }
      ).then(() => {
        handleLogout()
      })
    }
  } finally {
    loading.value = false
  }
}

// 滚动到底部
const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

// 处理下拉菜单命令
const handleCommand = (command) => {
  if (command === 'logout') {
    handleLogout()
  } else if (command === 'admin') {
    router.push('/admin')
  }
}

// 处理退出登录
const handleLogout = async () => {
  try {
    await axios.post('/api/auth/logout', {}, {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`
      }
    })
  } catch (error) {
    console.error('退出登录失败:', error)
  } finally {
    // 清除本地存储的用户信息
    localStorage.removeItem('token')
    localStorage.removeItem('user')
    localStorage.removeItem('isAdmin')
    
    // 跳转到登录页
    router.push('/login')
  }
}

// 加载历史消息
const loadHistoryMessages = async () => {
  try {
    const response = await axios.get('/api/chat/history', {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`
      }
    })
    
    if (response.data && Array.isArray(response.data.data)) {
      messages.value = response.data.data
      
      // 滚动到底部
      await nextTick()
      scrollToBottom()
    }
  } catch (error) {
    console.error('加载历史消息失败:', error)
    ElMessage.error('加载历史消息失败')
  }
}

onMounted(() => {
  // 加载历史消息
  loadHistoryMessages()
})
</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  background-color: #f5f7fa;
}

.chat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px;
  background-color: #fff;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
  z-index: 10;
}

.el-dropdown-link {
  cursor: pointer;
  display: flex;
  align-items: center;
  color: var(--el-color-primary);
}

.chat-main {
  flex: 1;
  overflow: hidden;
  padding: 20px;
  position: relative;
}

.chat-messages {
  height: 100%;
  overflow-y: auto;
  padding-right: 10px;
}

.message {
  display: flex;
  margin-bottom: 20px;
  align-items: flex-start;
}

.message.user {
  flex-direction: row-reverse;
}

.message-avatar {
  margin: 0 10px;
}

.message-content {
  max-width: 70%;
  padding: 10px 15px;
  border-radius: 8px;
  position: relative;
}

.message.user .message-content {
  background-color: var(--el-color-primary-light-9);
  color: var(--el-color-primary-dark-2);
  border-top-right-radius: 0;
  margin-right: 5px;
}

.message.ai .message-content {
  background-color: #fff;
  color: #333;
  border-top-left-radius: 0;
  margin-left: 5px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.message-text {
  line-height: 1.5;
  word-break: break-word;
}

.message-time {
  font-size: 12px;
  color: #999;
  margin-top: 5px;
  text-align: right;
}

.code-block {
  background-color: #f5f5f5;
  padding: 10px;
  border-radius: 4px;
  font-family: monospace;
  white-space: pre-wrap;
  margin: 10px 0;
  overflow-x: auto;
}

.chat-footer {
  padding: 15px 20px;
  background-color: #fff;
  border-top: 1px solid #eee;
}

.input-container {
  display: flex;
  align-items: flex-end;
  gap: 10px;
}

.input-container .el-input {
  flex: 1;
}

.input-container .el-button {
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
}

/* 思考过程样式 */
.thinking-process {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 12px;
  border: 1px solid #e9ecef;
}

.thinking-header {
  color: #6c757d;
  font-size: 0.9em;
  margin-bottom: 8px;
  font-weight: 500;
}

.thinking-content {
  color: #495057;
  line-height: 1.6;
  white-space: pre-wrap;
}

/* 加载动画 */
.thinking-animation {
  display: flex;
  align-items: center;
  color: #6c757d;
}

.dot {
  width: 6px;
  height: 6px;
  background: #6c757d;
  border-radius: 50%;
  margin-right: 4px;
  animation: dotPulse 1.4s infinite ease-in-out;
}

.dot:nth-child(2) {
  animation-delay: 0.2s;
}

.dot:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes dotPulse {
  0%, 80%, 100% { transform: scale(0.5); }
  40% { transform: scale(1); }
}

/* 优化代码块样式 */
.code-block {
  position: relative;
  padding: 1em;
  margin: 0.5em 0;
  background: #1e1e1e;
  color: #d4d4d4;
  border-radius: 4px;
  overflow-x: auto;
}

.code-block::before {
  content: 'Code';
  position: absolute;
  top: 0;
  right: 0;
  padding: 2px 8px;
  background: rgba(255,255,255,0.1);
  color: #858585;
  font-size: 0.8em;
  border-bottom-left-radius: 4px;
}
/* 确保空内容不会占用空间 */
.thinking-process:empty {
  display: none;
}
</style>
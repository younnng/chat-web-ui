<template>
  <div class="admin-container">
    <!-- 顶部导航栏 -->
    <header class="admin-header">
      <div class="header-left">
        <h2>智能文档助手管理后台</h2>
      </div>
      <div class="header-right">
        <el-dropdown @command="handleCommand">
          <span class="el-dropdown-link">
            {{ userInfo.username }}<el-icon class="el-icon--right"><arrow-down /></el-icon>
          </span>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item command="chat">返回聊天</el-dropdown-item>
              <el-dropdown-item command="logout">退出登录</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </div>
    </header>

    <!-- 主体内容区域 -->
    <main class="admin-main">
      <el-container>
        <!-- 侧边栏 -->
        <el-aside width="200px">
          <el-menu
            :default-active="activeMenu"
            class="admin-menu"
            @select="handleMenuSelect"
          >
            <el-menu-item index="users">
              <el-icon><user /></el-icon>
              <span>用户管理</span>
            </el-menu-item>
            <el-menu-item index="messages">
              <el-icon><chat-dot-round /></el-icon>
              <span>消息记录</span>
            </el-menu-item>
            <el-menu-item index="settings">
              <el-icon><setting /></el-icon>
              <span>系统设置</span>
            </el-menu-item>
          </el-menu>
        </el-aside>

        <!-- 内容区域 -->
        <el-main>
          <!-- 用户管理 -->
          <div v-if="activeMenu === 'users'" class="content-section">
            <div class="section-header">
              <h3>用户管理</h3>
              <el-button type="primary" @click="showAddUserDialog">添加用户</el-button>
            </div>
            
            <el-table :data="users" style="width: 100%" v-loading="loading.users">
              <el-table-column prop="id" label="ID" width="80" />
              <el-table-column prop="username" label="用户名" width="150" />
              <el-table-column prop="email" label="邮箱" width="200" />
              <el-table-column prop="role" label="角色">
                <template #default="scope">
                  <el-tag :type="scope.row.isAdmin ? 'danger' : 'success'">
                    {{ scope.row.isAdmin ? '管理员' : '普通用户' }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="createdAt" label="创建时间">
                <template #default="scope">
                  {{ formatDate(scope.row.createdAt) }}
                </template>
              </el-table-column>
              <el-table-column label="操作" width="200">
                <template #default="scope">
                  <el-button size="small" @click="editUser(scope.row)">编辑</el-button>
                  <el-button size="small" type="danger" @click="confirmDeleteUser(scope.row)">删除</el-button>
                </template>
              </el-table-column>
              </el-table>
          </div>

          <!-- 消息记录 -->
          <div v-if="activeMenu === 'messages'" class="content-section">
            <div class="section-header">
              <h3>消息记录</h3>
              <el-input
                v-model="messageSearch"
                placeholder="搜索消息内容"
                style="width: 300px"
                clearable
                @clear="loadMessages"
                @keyup.enter="searchMessages"
              >
                <template #append>
                  <el-button @click="searchMessages">
                    <el-icon><search /></el-icon>
                  </el-button>
                </template>
              </el-input>
            </div>
            
            <el-table :data="messages" style="width: 100%" v-loading="loading.messages">
              <el-table-column prop="id" label="ID" width="80" />
              <el-table-column prop="username" label="用户" width="120" />
              <el-table-column prop="role" label="发送者" width="100">
                <template #default="scope">
                  <el-tag :type="scope.row.role === 'user' ? '' : 'info'">
                    {{ scope.row.role === 'user' ? '用户' : 'AI' }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="content" label="内容" show-overflow-tooltip />
              <el-table-column prop="createdAt" label="时间" width="180">
                <template #default="scope">
                  {{ formatDate(scope.row.createdAt, true) }}
                </template>
              </el-table-column>
              <el-table-column label="操作" width="100">
                <template #default="scope">
                  <el-button size="small" type="danger" @click="confirmDeleteMessage(scope.row)">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
            
            <div class="pagination-container">
              <el-pagination
                v-model:current-page="messagePagination.currentPage"
                v-model:page-size="messagePagination.pageSize"
                :page-sizes="[10, 20, 50, 100]"
                layout="total, sizes, prev, pager, next, jumper"
                :total="messagePagination.total"
                @size-change="handleMessageSizeChange"
                @current-change="handleMessageCurrentChange"
              />
            </div>
          </div>

          <!-- 系统设置 -->
          <div v-if="activeMenu === 'settings'" class="content-section">
            <div class="section-header">
              <h3>系统设置</h3>
            </div>
            
            <el-form :model="systemSettings" label-width="120px">
              <el-form-item label="系统名称">
                <el-input v-model="systemSettings.systemName" />
              </el-form-item>
              <el-form-item label="AI模型">
                <el-select v-model="systemSettings.aiModel" style="width: 100%">
                  <el-option label="GPT-3.5" value="gpt-3.5-turbo" />
                  <el-option label="GPT-4" value="gpt-4" />
                  <el-option label="Claude" value="claude" />
                </el-select>
              </el-form-item>
              <el-form-item label="最大上下文长度">
                <el-input-number v-model="systemSettings.maxContextLength" :min="1" :max="20" />
              </el-form-item>
              <el-form-item label="允许用户注册">
                <el-switch v-model="systemSettings.allowRegister" />
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="saveSettings">保存设置</el-button>
                <el-button @click="resetSettings">重置</el-button>
              </el-form-item>
            </el-form>
          </div>
        </el-main>
      </el-container>
    </main>
  </div>

  <!-- 添加/编辑用户对话框 -->
  <el-dialog
    v-model="userDialog.visible"
    :title="userDialog.isEdit ? '编辑用户' : '添加用户'"
    width="500px"
  >
    <el-form :model="userDialog.form" :rules="userDialog.rules" ref="userFormRef" label-width="80px">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="userDialog.form.username" />
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="userDialog.form.email" />
      </el-form-item>
      <el-form-item label="密码" prop="password" v-if="!userDialog.isEdit">
        <el-input v-model="userDialog.form.password" type="password" show-password />
      </el-form-item>
      <el-form-item label="角色" prop="isAdmin">
        <el-select v-model="userDialog.form.isAdmin" style="width: 100%">
          <el-option :value="false" label="普通用户" />
          <el-option :value="true" label="管理员" />
        </el-select>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="userDialog.visible = false">取消</el-button>
        <el-button type="primary" @click="saveUser" :loading="loading.saveUser">确定</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import { ArrowDown, User, ChatDotRound, Setting, Search } from '@element-plus/icons-vue'
import axios from 'axios'

const router = useRouter()
const activeMenu = ref('users')
const userFormRef = ref(null)

// 加载状态
const loading = reactive({
  users: false,
  messages: false,
  saveUser: false,
  settings: false
})

// 获取用户信息
const userInfo = computed(() => {
  try {
    return JSON.parse(localStorage.getItem('user')) || { username: '管理员' }
  } catch (e) {
    return { username: '管理员' }
  }
})

// 用户列表数据
const users = ref([])

// 消息列表数据
const messages = ref([])
const messageSearch = ref('')
</script>
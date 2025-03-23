# AI聊天系统

这是一个基于Vue3和Element Plus的AI聊天Web应用，包含登录页面、聊天对话页面和后台管理页面。

## 功能特点

- 用户登录与认证
- AI智能对话
- 用户管理
- 消息记录查询
- 系统设置

## 技术栈

- Vue 3
- Vue Router
- Pinia
- Element Plus
- Axios

## 项目结构

```
├── public/             # 静态资源
├── src/                # 源代码
│   ├── assets/         # 资源文件
│   ├── components/     # 公共组件
│   ├── router/         # 路由配置
│   ├── views/          # 页面组件
│   ├── App.vue         # 根组件
│   └── main.js         # 入口文件
├── index.html          # HTML模板
└── vite.config.js      # Vite配置
```

## 安装与运行

### 前提条件

- Node.js (推荐v18或更高版本)
- npm或yarn

### 安装依赖

```bash
npm install
# 或
yarn
```

### 开发模式运行

```bash
npm run dev
# 或
yarn dev
```

应用将在 http://localhost:3000 运行

### 构建生产版本

```bash
npm run build
# 或
yarn build
```

## 后端API

本应用需要连接后端API服务，默认配置为连接到 http://localhost:8080

## 页面说明

- `/login` - 登录页面
- `/chat` - AI聊天对话页面
- `/admin` - 后台管理页面（需要管理员权限）
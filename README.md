# 访问统计前端

Vue 3 + Vite 构建的访问统计前端页面，用于展示医学科研助手系统的访问统计数据。

## 功能特性

- 📊 实时显示今日访问用户数
- 📈 显示今日总访问次数
- 📋 展示各导航页面的使用统计（访问次数和独立用户数）
- 🔄 自动刷新（每30秒）
- 🎨 现代化UI设计，渐变背景
- 📱 响应式布局

## 技术栈

- Vue 3
- Vite
- Axios

## 项目结构

```
stats-frontend/
├── src/
│   ├── App.vue          # 主组件
│   └── main.js          # 入口文件
├── index.html           # HTML模板
├── vite.config.js       # Vite配置
├── package.json         # 项目配置
└── README.md           # 说明文档
```

## 快速开始

### 1. 安装依赖

```bash
cd stats-frontend
npm install
```

### 2. 开发模式

```bash
npm run dev
```

访问 http://localhost:3000

### 3. 构建生产版本

```bash
npm run build
```

构建产物在 `dist` 目录。

### 4. 预览生产版本

```bash
npm run preview
```

## 配置

### 后端API地址

在 `vite.config.js` 中配置后端API地址：

```javascript
proxy: {
  '/api': {
    target: 'http://localhost:8080',  // 修改为实际的后端地址
    changeOrigin: true
  }
}
```

如果后端部署在其他地址，修改 `target` 为实际地址。

## 部署

### 方式一：静态文件服务器

1. 执行 `npm run build`
2. 将 `dist` 目录部署到 Nginx、Apache 等静态文件服务器
3. 配置反向代理，将 `/api` 请求转发到后端服务器

### 方式二：集成到Spring Boot

将构建后的 `dist` 目录内容复制到 Spring Boot 项目的 `src/main/resources/static/stats/` 目录，然后访问 `http://your-domain/stats/`

## API接口

前端调用后端接口：`GET /api/stats/today`

返回数据格式：
```json
{
  "todayUsers": 10,
  "todayTotal": 45,
  "date": "2026-03-15",
  "pageCounts": {
    "首页": 5,
    "文献查询": 20,
    "病例报告": 15
  },
  "pageUsers": {
    "首页": 3,
    "文献查询": 8,
    "病例报告": 6
  }
}
```

## 注意事项

1. 确保后端服务已启动并可以访问
2. 确保后端已配置CORS，允许前端域名访问
3. 如果遇到跨域问题，检查 `vite.config.js` 中的代理配置

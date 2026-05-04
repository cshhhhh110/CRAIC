# CRAIC
这是我们的参赛项目
# 非遗文化传承平台

基于 Spring Boot 3 + Vue 3 的非遗文化传承与保护平台,提供非遗项目管理、在线课程、活动报名、非遗商城等功能。

## 项目结构

```
├── backend/                 # 后端项目源码 (Spring Boot 3)
│   ├── src/main/java/       # Java 源代码
│   ├── src/main/resources/  # 配置文件
│   ├── files/               # 上传文件存储
│   └── pom.xml              # Maven 依赖配置
├── frontend/                # 前端项目源码 (Vue 3)
│   ├── src/                 # 前端源代码
│   ├── public/              # 静态资源
│   ├── package.json         # npm 依赖配置
│   └── vite.config.js       # Vite 构建配置
├── database/                # 数据库脚本
│   └── init.sql             # 数据库初始化脚本
├── docs/                    # 项目文档
│   ├── 运行说明.docx
│   ├── AI服务配置说明.txt
│   └── 硅基流动密钥配置说明.doc
└── video/                   # 部署视频教程
    └── 非遗部署视频.mp4
```

## 技术栈

### 后端
- **框架**: Spring Boot 3
- **数据库**: MySQL 8.0
- **ORM框架**: MyBatis-Plus
- **安全框架**: Spring Security + JWT
- **API文档**: Knife4j (Swagger)
- **缓存**: Redis 6.0
- **AI服务**: 硅基流动 AI

### 前端
- **框架**: Vue 3
- **构建工具**: Vite
- **UI组件库**: Element Plus
- **状态管理**: Pinia
- **路由**: Vue Router

## 快速开始

### 环境要求

- JDK 17+
- Maven 3.8+
- Node.js 16+
- MySQL 8.0+
- Redis 6.0+

### 后端运行步骤

1. **导入数据库**
   ```bash
   mysql -u root -p < database/init.sql
   ```

2. **配置数据库连接**
   
   编辑 `backend/src/main/resources/application.yml`,修改数据库连接信息:
   ```yaml
   spring:
     datasource:
       url: jdbc:mysql://localhost:3306/heritage_db
       username: your_username
       password: your_password
   ```

3. **配置 Redis**
   
   在 `application.yml` 中配置 Redis 连接:
   ```yaml
   spring:
     redis:
       host: localhost
       port: 6379
   ```

4. **配置 AI 服务** (可选)
   
   参考 `docs/AI服务配置说明.txt` 和 `docs/硅基流动密钥配置说明.doc`

5. **启动后端**
   ```bash
   cd backend
   mvn spring-boot:run
   ```

6. **访问 API 文档**
   
   启动后访问: http://localhost:8080/doc.html

### 前端运行步骤

1. **安装依赖**
   ```bash
   cd frontend
   npm install
   ```

2. **配置后端接口地址**
   
   编辑 `frontend/.env.development`,修改后端 API 地址:
   ```
   VITE_API_BASE_URL=http://localhost:8080
   ```

3. **启动前端**
   ```bash
   npm run dev
   ```

4. **访问前端页面**
   
   启动后访问: http://localhost:5173

## 功能模块

- 非遗项目管理
- 传承人管理
- 在线课程学习
- 活动报名
- 非遗商城
- AI 智能助手
- 用户管理

## 文档说明

- [运行说明](docs/运行说明.docx) - 项目运行详细步骤
- [AI服务配置说明](docs/AI服务配置说明.txt) - AI 服务配置指南
- [硅基流动密钥配置](docs/硅基流动密钥配置说明.doc) - 硅基流动 AI 密钥配置
- [部署视频教程](video/非遗部署视频.mp4) - 完整部署流程演示

## 许可证

本项目仅供学习交流使用。

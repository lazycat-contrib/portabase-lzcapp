# Portabase - LazyCat 应用

Portabase 是一个功能强大的数据库管理工具，已转换为 LazyCat Cloud 应用格式。

## 📦 应用信息

- **应用名称**: Portabase
- **包名**: cloud.lazycat.app.portabase
- **版本**: 1.0.0
- **最低 LazyCat 系统版本**: 1.3.8
- **许可证**: MIT

## 🚀 快速开始

### 前置要求

1. 安装 LazyCat CLI
   ```bash
   # 参考: https://developer.lazycat.cloud
   ```

2. 确保已登录懒猫应用商店
   ```bash
   lzc-cli appstore login
   ```

### 构建应用

```bash
# 使用自动化脚本
./build.sh
# 选择 1 - 构建应用

# 或手动构建
lzc-cli project build -o portabase-1.0.0.lpk
```

### 安装应用

```bash
# 本地安装测试
lzc-cli app install portabase-1.0.0.lpk

# 或发布到应用商店
./build.sh
# 选择 3 - 发布到应用商店
```

## 🏗️ 应用架构

### 服务组成

```
┌─────────────────────────────────────────┐
│         Portabase Application           │
├─────────────────────────────────────────┤
│  Services:                              │
│  ┌───────────────────────────────────┐  │
│  │  db (PostgreSQL 16-alpine)        │  │
│  │  - 内部服务                        │  │
│  │  - 自动管理密码                    │  │
│  │  - 持久化存储                     │  │
│  │  - 健康检查                       │  │
│  └───────────────────────────────────┘  │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │  portabase-app (Portabase)        │  │
│  │  - 外部服务 (端口 80)              │  │
│  │  - 依赖: db                       │  │
│  │  - 健康检查 (HTTP)                │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### 网络配置

- **子域名**: `portabase`
- **访问地址**: `http://portabase.${LAZYCAT_BOX_DOMAIN}`
- **上游**: `http://portabase-app:80/`

### 存储配置

- **数据库数据**: `/lzcapp/var/postgres`
  - PostgreSQL 数据持久化存储
  - 容器重启后数据保留

## ⚙️ 配置选项

### 部署参数

应用支持以下可选配置（通过部署向导设置）：

#### SMTP 配置（邮件通知）
- **SMTP 服务器**: SMTP 服务器主机名
- **SMTP 端口**: 端口号 (587 为 TLS，465 为 SSL)
- **SMTP 用户名**: 认证用户名
- **SMTP 密码**: 认证密码
- **发件人地址**: 邮件发送地址

#### Google OAuth 配置
- **Google OAuth Client ID**: OAuth 应用客户端 ID
- **Google OAuth Client Secret**: OAuth 应用客户端密钥
- **认证方式**: OAuth 认证方法

### 环境变量

#### 自动生成的内部变量
- `DATABASE_URL`: PostgreSQL 连接字符串（自动包含密码）
- `PROJECT_SECRET`: 应用密钥（自动生成稳定密钥）

#### 用户可配置变量
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `SMTP_FROM`
- `AUTH_GOOGLE_ID`, `AUTH_GOOGLE_SECRET`, `AUTH_GOOGLE_METHOD`

#### 固定变量
- `TZ`: Europe/Paris (巴黎时区)
- `NODE_ENV`: production (生产环境)
- `PROJECT_NAME`: Portabase
- `PROJECT_DESCRIPTION`: Portabase is a powerful database manager
- `RETENTION_CRON`: * * * * * (保留策略定时任务)

## 🔧 技术细节

### 健康检查

#### 数据库服务 (db)
```yaml
test: ["CMD-SHELL", "pg_isready -U devuser -d devdb"]
interval: 10s
timeout: 5s
retries: 5
```

#### 应用服务 (portabase-app)
```yaml
test: ["CMD-SHELL", "curl -f http://localhost:80/ || exit 1"]
interval: 60s
timeout: 10s
retries: 3
start_period: 30s
```

### 资源限制

- **数据库服务**:
  - CPU shares: 512
  - 内存限制: 512M

- **应用服务**:
  - CPU shares: 1024
  - 内存限制: 1024M

### 依赖关系

应用服务依赖于数据库服务：
```yaml
depends_on:
  - db
```

数据库服务启动并健康检查通过后，应用服务才会启动。

## 📁 文件结构

```
portabase-lzcapp/
├── lzc-manifest.yml          # 应用配置文件
├── lzc-build.yml             # 构建配置文件
├── lzc-deploy-params.yml     # 部署参数配置
├── build.sh                  # 自动化构建脚本
├── icon.png                  # 应用图标 (512x512 PNG)
├── README.md                 # 本文件
└── docker-compose.yml        # 原始 Docker Compose 配置
```

## 🛠️ 使用自动化脚本

### 菜单选项

运行 `./build.sh` 后，您将看到以下选项：

1. **📦 构建应用** - 使用 lzc-cli 构建 LPK 包
2. **🔧 镜像复制到懒猫仓库** - 将 Docker 镜像复制到 LazyCat 仓库
3. **📤 发布到应用商店** - 发布应用到懒猫应用商店
4. **🚀 一键构建+镜像复制+发布** - 完整的自动化发布流程
5. **📋 查看应用信息** - 显示应用配置详情
6. **❌ 退出** - 退出脚本

### 一键发布流程

选择选项 4 将执行完整的发布流程：

```
阶段 1: 初始构建（原始镜像）
  ↓ 构建 LPK 包
  ↓
阶段 2: 镜像复制（自动更新 manifest）
  ↓ 复制镜像到 LazyCat 仓库
  ↓ 更新 manifest 中的镜像地址
  ↓
阶段 3: 重新构建（新镜像）
  ↓ 使用新镜像重新构建
  ↓
阶段 4: 发布审核
  ↓ 发布到应用商店
  ↓ 等待审核
```

## 📋 发布流程

### 第一次发布

1. **登录应用商店**
   ```bash
   lzc-cli appstore login
   ```

2. **构建应用**
   ```bash
   ./build.sh
   # 选择 1 - 构建应用
   ```

3. **发布应用**
   ```bash
   ./build.sh
   # 选择 3 - 发布到应用商店
   ```

4. **等待审核**
   - 审核时间：1-3 个工作日
   - 审核通过后，应用将在应用商店中可见

### 更新应用

1. **更新版本号**
   - 编辑 `lzc-manifest.yml`，将 `version` 递增（如 1.0.0 → 1.0.1）

2. **构建并发布**
   ```bash
   ./build.sh
   # 选择 4 - 一键发布
   ```

## 🔄 从 Docker Compose 转换

### 原始配置 (docker-compose.yml)

```yaml
services:
  app:
    image: solucetechnologies/portabase:latest
    ports:
      - '8887:80'
    environment:
      - TZ=Europe/Paris
    env_file:
      - .env

  db:
    image: postgres:16-alpine
    ports:
      - "5433:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=devdb
      - POSTGRES_USER=devuser
      - POSTGRES_PASSWORD=changeme
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U devuser -d devdb" ]
      interval: 10s
      timeout: 5s
      retries: 5
```

### 转换要点

1. **端口映射**
   - 原始：`8887:80` → 转换为 `application.upstreams`
   - LazyCat 使用子域名访问，无需端口映射

2. **数据库服务**
   - 识别为 **内部服务**（有 healthcheck，无外部端口）
   - 密码自动管理：`{{.INTERNAL.db_password}}`
   - 卷映射转换为：`/lzcapp/var/postgres`

3. **应用服务**
   - 重命名为 `portabase-app`（避免使用保留名称 `app`）
   - 环境变量中的 `DATABASE_URL` 自动更新为内部连接
   - 添加健康检查（如果原始配置没有）

4. **环境变量**
   - `.env` 文件中的变量转换为部署参数
   - 敏感信息使用 `{{.U.xxx}}` 或 `{{ stable_secret "xxx" }}`

## ⚠️ 重要说明

### 镜像复制

LazyCat 应用商店要求所有镜像必须存储在 LazyCat 仓库中。在发布前，需要使用 `lzc-cli appstore copy-image` 将镜像复制到仓库。

自动化脚本提供了镜像复制功能，但请注意：
- 镜像必须是公开可访问的
- 每次复制都会重新拉取镜像
- 复制后的镜像地址会自动更新到 manifest

### 审核流程

1. **提交审核**
   - 使用 `lzc-cli appstore publish` 提交
   - 填写应用信息（名称、描述、图标等）

2. **审核时间**
   - 通常需要 1-3 个工作日
   - 审核团队会检查配置、功能、安全性

3. **审核结果**
   - 通过：应用上架到应用商店
   - 不通过：收到修改建议，重新提交

## 📚 参考文档

- **LazyCat 开发者门户**: https://developer.lazycat.cloud
- **应用发布指南**: https://developer.lazycat.cloud/docs/publish-app.html
- **lzc-cli 参考**: https://developer.lazycat.cloud/docs/lzc-cli.html

## 🆘 获取帮助

如果在使用过程中遇到问题：

1. 查看本 README 文档
2. 检查 `build.sh` 脚本输出的错误信息
3. 参考 LazyCat 官方文档
4. 在 GitHub Issues 中提交问题

## 📄 许可证

本项目基于 MIT 许可证开源。

---

**生成时间**: 2026-01-25
**生成工具**: LazyCat App Publisher Skill
# portabase-lzcapp

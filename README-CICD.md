# GitHub CI/CD Docker 镜像构建指南

本文档说明了如何使用GitHub Actions自动构建和推送Docker镜像到GitHub Container Registry (GHCR)。

## CI/CD 流程概述

我们配置了一个自动化的CI/CD流程，当以下事件发生时会触发Docker镜像的构建和推送：

1. 推送代码到 `main` 分支
2. 创建符合语义化版本的标签（如 `v1.0.0`）
3. 针对 `main` 分支创建Pull Request（仅构建，不推送）

## 镜像标签策略

根据不同的触发事件，会生成不同的镜像标签：

### 分支推送
- `ghcr.io/[用户名]/[仓库名]:main` - 基于分支名称

### 标签推送
- `ghcr.io/[用户名]/[仓库名]:1.0.0` - 完整版本号
- `ghcr.io/[用户名]/[仓库名]:1.0` - 主版本和次版本
- `ghcr.io/[用户名]/[仓库名]:1` - 仅主版本

### 默认分支推送
- `ghcr.io/[用户名]/[仓库名]:latest` - 最新版本标签

### Pull Request
- `ghcr.io/[用户名]/[仓库名]:pr-123` - PR编号（仅构建，不推送）

## 如何使用构建的Docker镜像

### 1. 拉取镜像

```bash
# 拉取最新版本
docker pull ghcr.io/[用户名]/[仓库名]:latest

# 拉取特定版本
docker pull ghcr.io/[用户名]/[仓库名]:v1.0.0
```

### 2. 运行容器

```bash
# 基本运行
docker run -p 9090:9090 ghcr.io/[用户名]/[仓库名]:latest

# 带环境变量运行
docker run -p 9090:9090 \
  -e DEFAULT_KEY="your-api-key" \
  -e ZAI_TOKEN="your-zai-token" \
  -e MODEL_NAME="GLM-4.5" \
  -e PORT="9090" \
  ghcr.io/[用户名]/[仓库名]:latest
```

### 3. 使用docker-compose

创建 `docker-compose.yml` 文件：

```yaml
version: '3.8'

services:
  z2api:
    image: ghcr.io/[用户名]/[仓库名]:latest
    ports:
      - "9090:9090"
    environment:
      - DEFAULT_KEY=your-api-key
      - ZAI_TOKEN=your-zai-token
      - MODEL_NAME=GLM-4.5
      - PORT=9090
      - DEBUG_MODE=true
      - DEFAULT_STREAM=true
      - DASHBOARD_ENABLED=true
    restart: unless-stopped
```

然后运行：

```bash
docker-compose up -d
```

## 环境变量配置

应用程序支持以下环境变量：

| 环境变量 | 默认值 | 描述 |
|---------|--------|------|
| `UPSTREAM_URL` | `https://chat.z.ai/api/chat/completions` | 上游API地址 |
| `DEFAULT_KEY` | `sk-123456` | 默认API密钥 |
| `ZAI_TOKEN` | `""` | Z.ai认证令牌 |
| `MODEL_NAME` | `GLM-4.5` | 模型名称 |
| `PORT` | `9090` | 服务端口 |
| `DEBUG_MODE` | `true` | 调试模式 |
| `DEFAULT_STREAM` | `true` | 默认流式响应 |
| `DASHBOARD_ENABLED` | `true` | 是否启用Dashboard |

## 触发构建

### 手动触发构建

1. 登录GitHub仓库
2. 点击 "Actions" 标签
3. 选择 "Build and Push Docker Image" 工作流
4. 点击 "Run workflow" 按钮

### 通过标签触发构建

```bash
# 创建并推送标签
git tag v1.0.0
git push origin v1.0.0
```

## 故障排除

### 常见问题

1. **构建失败**
   - 检查Dockerfile是否正确
   - 确保所有依赖项都已正确配置
   - 查看GitHub Actions日志获取详细错误信息

2. **推送失败**
   - 确保仓库启用了GitHub Packages
   - 检查GITHUB_TOKEN权限
   - 确保镜像名称格式正确

3. **镜像拉取失败**
   - 确保镜像已成功构建和推送
   - 检查镜像名称和标签是否正确
   - 确保已登录到GitHub Container Registry

### 查看构建日志

1. 登录GitHub仓库
2. 点击 "Actions" 标签
3. 点击对应的构建记录
4. 查看各个步骤的详细日志

## 安全注意事项

1. 不要在代码中硬编码敏感信息
2. 使用GitHub Secrets存储敏感数据
3. 定期更新基础镜像以获取安全补丁
4. 限制对Container Registry的访问权限

## 自动化部署

可以将此CI/CD流程与部署流程集成，例如：

1. 构建成功后自动部署到测试环境
2. 标签发布后自动部署到生产环境
3. 使用GitHub Environments管理不同环境的部署

示例部署步骤可以添加到工作流文件中：

```yaml
- name: Deploy to staging
  if: github.ref == 'refs/heads/main'
  run: |
    # 部署到测试环境的命令
    echo "Deploying to staging environment..."
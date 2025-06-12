# Docker 构建流水线配置指南

本项目现在包含两个 GitHub Actions 工作流来自动构建和推送 Docker 镜像。

## 工作流文件

### 1. 完整构建流水线 (`.github/workflows/docker-build.yml`)

这个流水线包含完整的多阶段构建：
- 构建基础镜像 (base2)
- 构建 Go 构建器镜像 (builder-go)
- 构建主要应用镜像 (release)
- 构建无根权限镜像 (rootless)
- 构建夜间版本镜像 (nightly)

### 2. 简化构建流水线 (`.github/workflows/docker-simple.yml`)

这个流水线提供简化的构建过程，只构建主要的应用镜像，适合大多数使用场景。

## 配置要求

### 1. Docker Hub 认证

在 GitHub 仓库的 Settings > Secrets and variables > Actions 中添加以下 secrets：

- `DOCKER_USERNAME`: 你的 Docker Hub 用户名
- `DOCKER_PASSWORD`: 你的 Docker Hub 访问令牌 (推荐) 或密码

### 2. Docker Hub 访问令牌创建步骤

1. 登录 [Docker Hub](https://hub.docker.com/)
2. 点击右上角头像 > Account Settings
3. 选择 Security > Access Tokens
4. 点击 "New Access Token"
5. 输入描述，选择权限 (推荐 Read, Write, Delete)
6. 复制生成的令牌

## 触发条件

### 完整构建流水线

- 推送到 `main` 或 `master` 分支
- 推送标签 (格式: `v*`)
- Pull Request 到 `main` 或 `master` 分支
- 手动触发

### 简化构建流水线

- 推送到 `main` 或 `master` 分支
- 推送版本标签 (格式: `v*.*.*`)
- 手动触发

## 镜像标签

### 完整构建流水线生成的标签

- `latest` (主分支)
- `{branch-name}` (分支推送)
- `{version}` (版本标签)
- `{major}.{minor}` (版本标签)
- `rootless` (无根权限版本)
- `nightly` (夜间版本)

### 简化构建流水线生成的标签

- `latest` (主分支)
- `{branch-name}` (分支推送)
- `{version}` (版本标签)
- `{major}.{minor}` (版本标签)

## 支持的架构

- `linux/amd64`
- `linux/arm64`
- `linux/arm/v7` (仅完整构建流水线)

## 使用建议

1. **首次使用**: 建议使用简化构建流水线 (`docker-simple.yml`)
2. **生产环境**: 可以使用完整构建流水线获得更多镜像变体
3. **测试**: 可以通过 workflow_dispatch 手动触发构建

## 本地测试

在推送到 GitHub 之前，可以本地测试 Docker 构建：

```bash
# 构建主要镜像
docker build -f docker/dockerfile --target release -t uptime-kuma:test .

# 构建无根权限镜像
docker build -f docker/dockerfile --target rootless -t uptime-kuma:rootless-test .

# 运行测试
docker run -p 3001:3001 uptime-kuma:test
```

## 故障排除

### 1. 认证失败

- 确认 Docker Hub 用户名和令牌正确
- 检查 GitHub Secrets 配置

### 2. 构建失败

- 检查 Dockerfile 语法
- 确认所有依赖文件存在
- 查看 GitHub Actions 日志

### 3. 推送失败

- 确认 Docker Hub 仓库存在且有写权限
- 检查镜像名称格式

## 自定义配置

### 修改镜像名称

在工作流文件中修改 `IMAGE_NAME` 环境变量：

```yaml
env:
  REGISTRY: docker.io
  IMAGE_NAME: your-username/your-image-name
```

### 添加其他镜像仓库

可以添加多个镜像仓库，如 GitHub Container Registry：

```yaml
- name: Login to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
``` 
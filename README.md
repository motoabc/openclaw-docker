# OpenClaw Docker Auto Sync

本仓库用于自动监听 OpenClaw 官方 release，并将同版本 tag 的 Docker 镜像发布到 Docker Hub。

## 目标

- 监听上游：`https://github.com/openclaw/openclaw/releases`
- 仅同步正式版 release（非 draft / 非 prerelease）
- 镜像发布到：`docker.io/leopoo/openclaw:<official-tag>`
- 新版本自动构建，无新版本自动跳过

## 镜像运行时约束

- Python `3.10.16`
- Node.js `22.22.1`
- 基础工具：`gh` / `git` / `curl` / `jq`
- SSH 客户端：`openssh-client`
- 中国源优化：APT / pip / npm

## GitHub Actions

工作流文件：`.github/workflows/docker-sync-release.yml`

触发方式：

- `schedule`：每小时轮询（`17 * * * *`）
- `workflow_dispatch`：手动触发

同步逻辑：

1. 读取官方最新正式 release tag
2. 与 `.github/last_built_tag` 比较
3. 若相同则退出
4. 若不同则拉取上游对应 tag 源码并构建
5. 推送镜像到 Docker Hub（tag 与官方一致）
6. 推送成功后更新 `.github/last_built_tag`

## 必要 Secrets

在 GitHub 仓库中配置：

- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

> 建议 Docker Hub token 使用最小权限并定期轮换。

## 本地验证

```bash
docker build -t openclaw-local:test .
docker run --rm openclaw-local:test python --version
docker run --rm openclaw-local:test node --version
docker run --rm openclaw-local:test gh --version
docker run --rm openclaw-local:test git --version
docker run --rm openclaw-local:test curl --version
docker run --rm openclaw-local:test jq --version
docker run --rm openclaw-local:test ssh -V
```

## 状态文件

- `.github/last_built_tag`：记录最近成功发布的官方 tag。
- 仅在镜像 push 成功后更新，避免错误前移。

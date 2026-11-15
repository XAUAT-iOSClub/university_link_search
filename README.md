# 建大导航重置版

## 后端部署

`.github/workflows/backend-ci-cd.yml` 在推送 `master` 或手动运行时构建后端镜像并推送 GHCR；如果配置了 `MIRROR_REGISTRY` / `MIRROR_NAMESPACE` 仓库变量，还会推送镜像仓库，自动部署也优先从镜像仓库拉取。

### 自动部署

在仓库 Settings → Environments 创建 `production`（或使用仓库级 secrets），配置：

- `SSH_HOST`（必需）：部署服务器地址。
- `SSH_USER`（必需）：允许密钥登录的部署用户，需能执行 `docker`。
- `SSH_PRIVATE_KEY`（必需）：部署私钥。
- `SSH_HOST_FINGERPRINT`（建议）：服务器 SSH 主机密钥的 SHA256 指纹，例如 `ssh-keyscan -t ed25519 <host> | ssh-keygen -lf -`。
- `SSH_PASSPHRASE`（按需）：私钥密码。
- `SSH_PORT`（按需）：非 22 端口。
- `MIRROR_USERNAME` / `MIRROR_PASSWORD`（按需）：私有镜像仓库凭证。

缺少三个必需 secrets 时，工作流只推送镜像并输出跳过提示，不会尝试连接服务器。

首次部署前，在服务器准备 `~/universitylink/.env`（可从 `backend/UniversityLink.WebApi/deploy/.env.example` 复制，至少配置 `SQL`、`JWT_SECRET` 和 OAuth 凭据，按需 `APP_PORT`），并确保部署用户可以运行 Docker 和 Docker Compose v2。之后每次部署，工作流会把 `backend/UniversityLink.WebApi/deploy/` 同步到 `~/universitylink/`，以本次提交 SHA 的镜像执行 `build_from_ghcr.sh`，重启容器并确认应用已启动。`.env` 含密钥，始终由运维在服务器维护，不经过 CI。

### 手动部署与回滚

在服务器 `~/universitylink` 中执行：

```sh
IMAGE=ghcr.io/xauat-iosclub/university_link_search-backend:<commit> bash ./build_from_ghcr.sh
```

脚本会输出上一次运行的镜像引用，用它重新执行即可回滚。

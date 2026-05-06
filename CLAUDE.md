# lazy-pansou-web — 维护笔记

懒猫微服 wrapper for [fish2018/pansou-web](https://github.com/fish2018/pansou-web)。

## 仓库结构

```
lazy-pansou-web/
├── docker/
│   └── Dockerfile           单行 FROM ghcr.io/fish2018/pansou-web@sha256:...
├── lazycat/
│   ├── package.template.yml
│   ├── lzc-manifest.template.yml
│   ├── lzc-build.yml
│   ├── appstore.yml
│   ├── icon.png             pansou 占位 logo（部署后换 UI 截图）
│   └── screenshots/         占位（部署后用 playwright 抓真实 UI 替换）
└── .github/workflows/        薄壳，调用 microlazy-apps/lazycat-ci
    ├── release.yml
    └── bootstrap-app.yml
```

## 架构选型：re-tag 而不 vendor

**没有 vendor 上游源代码**，与其他 wrapper 不同。原因：

- pansou-web 仓库**没有 LICENSE 文件**（只有 pansou 后端是 MIT），重分发源码法律不清；
- 上游已通过 GHCR Actions 把镜像 publish 到 `ghcr.io/fish2018/pansou-web:latest`；
- 我们要做的事就是把这个镜像复制到 lazycat registry 然后给它套个 lpk manifest。

所以 `docker/Dockerfile` 仅一行：

```dockerfile
FROM ghcr.io/fish2018/pansou-web@sha256:1857ad...
```

CI 走 `lpk-build.yml@v0.1.0`，docker-context = `./docker`，生成的镜像就是 pansou-web 的 retag 版本，推到 lazycat registry，由 lpk manifest 引用为 `${LAZYCAT_IMAGE}`。

更新上游版本：

```sh
TOKEN=$(curl -sSL 'https://ghcr.io/token?service=ghcr.io&scope=repository:fish2018/pansou-web:pull' | jq -r .token)
curl -sSL -I "https://ghcr.io/v2/fish2018/pansou-web/manifests/latest" \
  -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  -H "Authorization: Bearer $TOKEN" \
  | grep -i docker-content-digest
# 更新 docker/Dockerfile 里的 sha256
```

## 容器内布局

`pansou-web` 镜像基于 `nginx:alpine`，用 `start.sh` 同时跑：

- nginx 监听 :80（serve Vue SPA + reverse-proxy `/api/*` 到后端）
- pansou Go 二进制监听 127.0.0.1:8888

懒猫只需把外部 :80 转给 main 容器，无需进一步多服务编排。

## 健康检查

`http://main:80/api/health` —— pansou 后端的健康端点，nginx 透传。

## 数据持久化

`/lzcapp/var/data:/app/data` —— 缓存 + 日志。所有持久化都在这一个卷下。

## 版本与发布

- 第一次 bootstrap：`gh workflow run bootstrap-app.yml -F version=0.0.1`
- 之后 release：`git tag v0.0.2 && git push origin v0.0.2`
- 上游 image bump：更新 sha256，commit，打新 tag

## 已知风险

- **license 状态不明**：pansou-web 没声明 license。我们只 retag GHCR 镜像不重分发源码以最小化风险，但仍依赖上游"public GHCR push 即默许下载使用"的非正式约定。
- **TG 频道访问**：依赖 box 出网；若被网络拦截 TG 频道，搜索结果会缺失。
- **第三方资源**：本应用只搜索聚合，不存任何资源；点击结果跳转第三方网盘。

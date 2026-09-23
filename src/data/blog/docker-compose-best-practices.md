---
title: "2026 年 Docker Compose 实用最佳实践"
description: "不止是 docker compose up：生产环境配置、密钥、健康检查、Profiles 与多阶段构建。"
pubDatetime: 2026-02-05T10:00:00Z
tags:
  - docker
  - devops
  - containers
  - backend
draft: false
---

`docker compose up` 往往是我们学会的第一条命令。但网络、密钥、健康检查，以及面向不同环境的 Profiles，才是真正区分“能够运行”和“适合生产”的关键。

## Table of contents

## 清晰的基础结构

```yaml file=compose.yml
name: my-app

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: production # 多阶段构建目标 // [!code highlight]
    environment:
      NODE_ENV: production
    env_file: .env.production # 不要硬编码凭据 // [!code highlight]
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy # 等待数据库就绪 // [!code highlight]
    restart: unless-stopped

  db:
    image: postgres:17-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  postgres_data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

## 多阶段构建：更小的体积，更高的安全性

生产环境的 Dockerfile 不应包含开发工具：

```dockerfile file=Dockerfile
# 阶段 1：安装依赖并构建
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci                    # [!code highlight]
COPY . .
RUN npm run build

# 阶段 2：最小化的最终镜像
FROM node:22-alpine AS production  # [!code ++]
WORKDIR /app                       # [!code ++]
                                   # [!code ++]
# 只复制必要内容                   # [!code ++]
COPY --from=builder /app/dist ./dist  # [!code ++]
COPY --from=builder /app/node_modules ./node_modules  # [!code ++]
                                   # [!code ++]
USER node                          # 不以 root 身份运行 // [!code ++]
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

采用多阶段构建后，镜像体积可能从 **600 MB 降低到 80 MB**。

## 使用 Profiles 管理不同环境

通过 `profiles` 可以根据场景启用服务，而不必维护多份 Compose 文件：

```yaml file=compose.yml
services:
  api:
    # 没有 profile，表示始终启用
    build: .

  adminer:
    image: adminer
    profiles: [dev, debug] # 只在开发环境启用 // [!code highlight]
    ports:
      - "8080:8080"

  prometheus:
    image: prom/prometheus
    profiles: [monitoring] # 需要监控时才启用 // [!code highlight]
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

```bash
# 只启动 API 和数据库
docker compose up

# 同时启动开发工具
docker compose --profile dev up

# 启动完整监控栈
docker compose --profile monitoring up
```

## 真正有效的健康检查

基础的 `depends_on` 只会等待容器**启动**，不会等待服务真正**就绪**，二者的差异非常重要：

```yaml file=compose.yml
services:
  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 10
      start_period: 10s # 初始宽限时间 // [!code highlight]

  worker:
    build: .
    depends_on:
      redis:
        condition: service_healthy # 等待健康检查通过 // [!code highlight]
```

## 网络：默认保持隔离

每个 `compose.yml` 都会创建自己的网络。需要进一步隔离前端和后端时，可以这样配置：

```yaml file=compose.yml
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true # 禁止访问互联网 // [!code highlight]

services:
  nginx:
    networks: [frontend, backend] # 唯一同时连接两个网络的服务

  api:
    networks: [backend] # 与外部隔离 // [!code highlight]

  db:
    networks: [backend] # 同样保持隔离
```

## 上线前检查清单

- [ ] 敏感变量放在仓库之外的 `secrets` 或 `.env` 中
- [ ] 已启用多阶段构建
- [ ] 所有关键服务设置 `restart: unless-stopped`
- [ ] 健康检查包含合适的 `start_period`
- [ ] `depends_on` 使用 `condition: service_healthy`
- [ ] 容器使用非 root 用户（如 `USER node`、`USER app`）
- [ ] 持久化数据使用命名卷，生产环境避免 bind mount
- [ ] 根据容器内存配置 `--max-old-space-size`

> 教程里的 `compose.yml` 与生产配置之间的差异，不在于代码行数，而在于是否提前考虑了可能发生的故障。

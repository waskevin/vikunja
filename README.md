# Vikunja for TerraMaster TOS 7

TOS 7 App Center package for [Vikunja](https://vikunja.io).

一个把 [Vikunja](https://vikunja.io) 打包上架到 TOS 7 应用中心的 Docker 应用（单容器、单数据卷、SQLite，零外部依赖）。

---

## What it does / 这是什么

Vikunja is a task manager you host yourself: lists, projects, labels,
priorities, due dates, reminders and calendar feeds - without an account
on somebody else's server.

- Web UI on host port `10046` -> container `3456`
- Tasks, projects, labels, reminders, attachments and list sharing
- Single SQLite database, stored in the application data directory
- Runs as a non-root user, no privileged mode, no host networking

## Package contents / 包内文件

```
vikunja.tar.gz
├── config.ini            # TOS application metadata
├── vikunja.lang           # 14 languages (zh-cn, zh-hk, en-us, ... pt-pt)
├── vikunja.svg            # application icon (SVG, transparent)
└── docker-compose.yml    # container definition
```

## docker-compose.yml

```yaml
version: "3.8"

services:
  vikunja-init:
    image: alpine:3.22
    restart: "no"
    user: "1000:1000"
    volumes:
      - "./data:/data"
    entrypoint: ["/bin/sh", "-c"]
    command:
      - |
        if [ ! -s /data/config.yml ]; then
          secret=$$(head -c 48 /dev/urandom | base64 | tr -d '\n')
          printf 'service:\n  secret: "%s"\n' "$$secret" > /data/config.yml
          chmod 600 /data/config.yml
        fi
        echo "vikunja config ready"

  vikunja:
    image: vikunja/vikunja:2.6
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "10046:3456"
    volumes:
      - "./data:/app/vikunja/files"
    environment:
      - TZ=Asia/Shanghai
      - VIKUNJA_DATABASE_TYPE=sqlite
      - VIKUNJA_DATABASE_PATH=/app/vikunja/files/vikunja.db
      - VIKUNJA_CORS_ENABLE=false
    command: ["--config", "/app/vikunja/files/config.yml"]
    depends_on:
      vikunja-init:
        condition: service_completed_successfully
    healthcheck:
      test: ["CMD", "/app/vikunja/vikunja", "--config", "/app/vikunja/files/config.yml", "healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

x-app-meta:
  web:
    port: 10046
    protocol: http
```

The JWT secret is never hardcoded: the one-shot `vikunja-init` container generates it with `/dev/urandom` into `data/config.yml` (mode 600) before the app starts, and `depends_on` guarantees the order. The main image is distroless, hence the small helper.

## Build / 打包

```sh
./build.sh          # writes vikunja.tar.gz + vikunja.tar.gz.sha256
```

## Data location / 数据位置

TOS places the bind mounts under `/Volume<N>/DockerAppData/vikunja/`; the
`data/` folder holds everything that must survive an update or a
reinstall (database, attachments, configuration).

## Upstream / 上游

- Project: https://github.com/go-vikunja/vikunja
- Docker image: https://hub.docker.com/r/vikunja/vikunja
- License: AGPL-3.0 (see LICENSE / NOTICE in this repository)

## Notes / 注意

Open the web UI and register the first account - the first registered user becomes the administrator. A random JWT signing secret is generated on first start into data/config.yml by the short-lived vikunja-init container, so logins survive restarts. Docker Engine must be installed and enabled from the TOS App Center.

首次使用时请打开网页界面注册第一个账号，首位注册的用户即为管理员。首次启动时由临时的 vikunja-init 容器在数据目录中生成随机 JWT 签名密钥并保存为 config.yml，登录状态重启后依然有效。需先在应用中心安装并启用 DockerEngine。

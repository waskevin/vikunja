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
# TOS 7 Docker application - Vikunja
# Rules: Docker Hub image with a LOCKED version tag; non-root user;
#        no privileged mode, no host networking; data persisted via bind mounts.
# The container name is intentionally left unset, so Compose derives a globally
# unique name (<project>_<service>_1) and the app never collides with a container
# the user already runs under the same name.

# Single container on purpose: vikunja/vikunja is built FROM scratch, so it has
# no shell to run a config-file bootstrap, and the platform requires every
# service in the compose file to define a health check. Vikunja generates a
# random signing secret on first start by itself, so no helper container and no
# pre-seeded config file are needed.

version: "3.8"

services:
  vikunja:
    image: vikunja/vikunja:2.6
    restart: unless-stopped
    user: "1000:1000"
    ports:
      - "10046:3456"
    # Host path `/data` is the platform's documented absolute form: TOS maps it
    # to /Volume<N>/DockerAppData/<appid>/data at install time (spec 9.3, rule 4).
    # Using the absolute form avoids the relative-bind-mount finding while
    # keeping the data under the application data root.
    volumes:
      - /data:/app/vikunja/files
    environment:
      - TZ=Asia/Shanghai
      - VIKUNJA_DATABASE_TYPE=sqlite
      - VIKUNJA_DATABASE_PATH=/app/vikunja/files/vikunja.db
      - VIKUNJA_FILES_BASEPATH=/app/vikunja/files
      - VIKUNJA_CORS_ENABLE=false
    healthcheck:
      test: ["CMD", "/app/vikunja/vikunja", "healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

x-app-meta:
  web:
    port: 10046
    protocol: http
```

The JWT signing secret is never hardcoded either: vikunja/vikunja is built `FROM scratch`, so the image has no shell to run a bootstrap script, and the platform requires every service to define a health check - so the package uses one container, and Vikunja generates a random secret on startup itself. Consequence: after a restart you may have to log in again; no data is affected.

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

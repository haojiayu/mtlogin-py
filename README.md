# MTLogin 管理后台

MTLogin 是一个带 Web 管理后台的 M-Team 登录/刷新服务。后台使用 Flask + SQLite 提供管理 API，前端使用 Vue 3 + Vite 构建单页控制台，支持平台配置、通知渠道、多个登录账户、执行记录和系统设置。

核心能力：

- 多账号独立管理，每个账户绑定一个平台和多个通知渠道
- 按账户配置 M-Team 登录参数、Auth Token、代理、Cookie 策略和 Cron 表达式
- 后台调度线程按账户级 Cron 自动执行，也支持账户行内立即执行
- 账户列表展示最近执行摘要和下次执行时间
- 执行记录持久化，支持按账户、平台、状态和时间范围查询
- Docker 部署可显式指定调度时区，避免容器时区影响 Cron 解释

## 快速启动

安装 Python 依赖：

```bash
pip install -r requirements.txt
```

构建前端并启动 Flask：

```bash
cd frontend
pnpm install
pnpm build
cd ..
python app.py
```

默认访问地址是 `http://127.0.0.1:8000`。默认管理员账号为 `admin`，默认密码为 `admin123456`，首次登录后建议立即在 `系统设置` 中修改。

常用启动参数：

```bash
python app.py \
  --host 0.0.0.0 \
  --port 8000 \
  --db-path ./mtlogin.db \
  --log-file ./mtlogin.log \
  --frontend-dist ./frontend/dist \
  --admin-username admin \
  --admin-password admin123456
```

## Docker 部署

构建镜像：

```bash
docker build -t mtlogin-py .
```

建议使用 volume 持久化 SQLite 数据库和日志，并显式设置调度时区：

```bash
docker volume create mtlogin-data

docker run -d \
  --name mtlogin \
  -p 8000:8000 \
  -e ADMIN_USERNAME="admin" \
  -e ADMIN_PASSWORD="change-me-now" \
  -e SCHEDULER_TIMEZONE="Asia/Shanghai" \
  -v mtlogin-data:/data \
  mtlogin-py \
  python app.py \
    --host 0.0.0.0 \
    --port 8000 \
    --db-path /data/mtlogin.db \
    --log-file /data/mtlogin.log \
    --frontend-dist /app/frontend/dist
```

容器默认把数据库和日志写入 `/app/mtlogin.db`、`/app/mtlogin.log`。如果要持久化数据，建议像上面一样挂载 `/data` 并把 `--db-path`、`--log-file` 指向 `/data`。如果使用 `--rm` 且不挂载持久化目录，容器删除后数据库和日志也会删除。

## 调度时区

账户 Cron 表达式按 `SCHEDULER_TIMEZONE` 指定的 IANA 时区计算下次执行时间和实际调度时间，例如：

```bash
SCHEDULER_TIMEZONE=Asia/Shanghai
```

未设置 `SCHEDULER_TIMEZONE` 时，程序会尝试使用 `TZ`；两者都未设置时，使用当前进程本地时区。Docker 部署建议始终显式设置 `SCHEDULER_TIMEZONE`。

常见 Cron 示例：

```bash
# 每 2 小时的第 2 分钟执行一次
2 */2 * * *

# 每天凌晨 3:30 执行一次
30 3 * * *

# 每 15 分钟执行一次
*/15 * * * *
```

## 后台页面

登录后默认进入 `账户管理` 页面。当前后台包含：

- `账户管理`：维护账户、平台绑定、通知绑定、Cron 计划，并查看最近执行摘要和下次执行时间
- `平台配置`：查看内置 M-Team 平台及启停状态
- `通知管理`：维护 Telegram 通知渠道
- `执行记录`：查询手动或定时执行产生的历史记录
- `系统设置`：修改管理员账号密码，查看运行环境和日志尾部

兼容入口 `/dashboard` 会跳转到账户管理页面。管理前端通过 `/api/admin/**` 调用 Flask JSON API，认证基于 Flask Session。

## 账户配置

每个登录账户支持这些运行参数：

- `M-Team 用户名`、`M-Team 密码`、`TOTP 密钥`
- `M-Team Auth Token`、`M-Team DID`
- `代理`、`Cron 表达式`
- `超时秒数`、`Cookie 模式`、`跳过缓存`
- 一个已启用平台
- 零个或多个已启用通知渠道

密码、TOTP、Auth Token、Telegram Token 等敏感字段在编辑时留空不会覆盖已保存的值。

## 环境变量

`app.py` 支持这些运行环境变量：

| 环境变量 | 默认值 | 说明 |
|---|---|---|
| `HOST` | `0.0.0.0` | Web 服务监听地址 |
| `PORT` | `8000` | Web 服务端口 |
| `DB_PATH` | `./mtlogin.db` | SQLite 数据库路径 |
| `LOG_FILE` | `./mtlogin.log` | 日志文件路径 |
| `FRONTEND_DIST` | `./frontend/dist` | Flask 分发的前端构建目录 |
| `ADMIN_USERNAME` | `admin` | 初始管理员用户名，仅首次初始化时使用 |
| `ADMIN_PASSWORD` | `admin123456` | 初始管理员密码，仅首次初始化时使用 |
| `SECRET_KEY` | 随机生成 | Flask Session 密钥 |
| `SCHEDULER_TIMEZONE` | `TZ` 或进程本地时区 | 账户 Cron 调度使用的 IANA 时区 |
| `TZ` | 空 | 未设置 `SCHEDULER_TIMEZONE` 时的调度时区 fallback |

后台未保存账户配置时，部分 M-Team 默认值仍可从环境变量读取：

- `USERNAME`
- `PASSWORD`
- `TOTPSECRET`
- `CRONTAB`
- `PROXY`
- `M_TEAM_AUTH`
- `M_TEAM_DID`
- `API_HOST`
- `API_REFERER`
- `TGBOT_TOKEN`
- `TGBOT_CHAT_ID`
- `TGBOT_PROXY`
- `TIME_OUT`
- `COOKIE_MODE`

## 开发模式

后端和前端可以分开启动：

```bash
python app.py

cd frontend
pnpm install
pnpm dev
```

默认 Vite 开发地址是 `http://127.0.0.1:3333`。

常用验证命令：

```bash
PYTHONPATH=. python3 tests/test_web_admin.py

cd frontend
pnpm build
```

## 脚本模式

`mtlogin.py` 仍可作为单次或定时脚本单独运行：

```bash
python mtlogin.py --username "站点用户名" --password "站点密码" --totpsecret "TOTP密钥"
```

常用参数：

- `--m-team-auth`
- `--m-team-did`
- `--proxy`
- `--crontab`
- `--skip-cache`
- `--log-file`
- `--db-path`

如果指定 `--crontab`，脚本会按该表达式循环执行；否则只执行一次。

## 安全说明

- 管理员密码以哈希形式保存在 SQLite 中。
- M-Team 密码、TOTP 和 Auth Token 需要可逆使用，仍会保存在本地数据库中。
- HTTP 调试日志会脱敏密码、OTP 和 Authorization，但仍建议限制数据库和日志文件访问权限。

# Sonic 环境变量配置指南

## 概述

Sonic 支持通过环境变量进行配置，环境变量名称为配置路径中的点号（.）替换为下划线（_）。例如：

- `server.host` → `SERVER_HOST`
- `server.port` → `SERVER_PORT`
- `mysql.dsn` → `MYSQL_DSN`

## 完整环境变量列表

### 服务器配置 (server)
| 配置项 | 环境变量 | 默认值 | 描述 |
|--------|----------|--------|------|
| server.host | SERVER_HOST | 0.0.0.0 | 服务器监听地址 |
| server.port | SERVER_PORT | 8080 | 服务器监听端口 |

### 日志配置 (logging)
| 配置项 | 环境变量 | 默认值 | 描述 |
|--------|----------|--------|------|
| logging.filename | LOGGING_FILENAME | sonic.log | 日志文件名 |
| logging.level.app | LOGGING_LEVEL_APP | info | 应用日志级别 (debug,info,warn,error) |
| logging.level.gorm | LOGGING_LEVEL_GORM | warn | GORM日志级别 (info,warn,error,silent) |
| logging.maxsize | LOGGING_MAXSIZE | 10 | 日志文件最大大小（单位：MB） |
| logging.maxage | LOGGING_MAXAGE | 30 | 日志文件保留天数 |
| logging.compress | LOGGING_COMPRESS | false | 是否对旧日志使用gzip进行压缩 |

### 数据库配置
| 配置项 | 环境变量 | 默认值 | 描述 |
|--------|----------|--------|------|
| sqlite3.enable | SQLITE3_ENABLE | true | 是否启用SQLite3 |
| mysql.dsn | MYSQL_DSN | 无 | MySQL连接字符串：username:password@tcp(host:port)/db_name?charset=utf8mb4&parseTime=True&loc=Local&interpolateParams=true |

### Sonic应用配置 (sonic)
| 配置项 | 环境变量 | 默认值 | 描述 |
|--------|----------|--------|------|
| sonic.mode | SONIC_MODE | production | 运行模式 (production/development) |
| sonic.work_dir | SONIC_WORK_DIR | ./ | 工作目录，用来存放日志文件、数据库文件、模板、上传的附件等 |
| sonic.log_dir | SONIC_LOG_DIR | ./logs | 日志目录，不填则使用work_dir路径下的log路径 |
| sonic.admin_url_path | SONIC_ADMIN_URL_PATH | admin | 管理后台URL路径 |

## 注意事项

1. **数据库选择优先级**：如果同时配置了MySQL和SQLite3，优先使用SQLite3
2. **工作目录用途**：work_dir用于存放日志文件、数据库文件、模板、上传的附件等
3. **环境变量格式**：所有布尔值使用true/false，数字使用整数，字符串直接使用
4. **MySQL DSN格式**：需要完整的连接字符串：`用户名:密码@tcp(主机:端口)/数据库名?charset=utf8mb4&parseTime=True&loc=Local&interpolateParams=true`

## Docker使用示例

```bash
# 基本使用（使用SQLite3）
docker run -d \
  --name sonic \
  -p 8080:8080 \
  -v sonic_data:/sonic \
  -e SERVER_HOST=0.0.0.0 \
  -e SERVER_PORT=8080 \
  -e SONIC_MODE=production \
  -e SQLITE3_ENABLE=true \
  -e SONIC_WORK_DIR=/sonic \
  sonic:latest

# 使用MySQL数据库
docker run -d \
  --name sonic \
  -p 8080:8080 \
  -v sonic_data:/sonic \
  -e SERVER_HOST=0.0.0.0 \
  -e SERVER_PORT=8080 \
  -e SONIC_MODE=production \
  -e SQLITE3_ENABLE=false \
  -e MYSQL_DSN="root:password@tcp(mysql:3306)/sonic?charset=utf8mb4&parseTime=True&loc=Local&interpolateParams=true" \
  -e SONIC_WORK_DIR=/sonic \
  gosonic/sonic:latest

# 自定义日志配置
docker run -d \
  --name sonic \
  -p 8080:8080 \
  -v sonic_data:/sonic \
  -e SERVER_HOST=0.0.0.0 \
  -e SERVER_PORT=8080 \
  -e LOGGING_FILENAME="app.log" \
  -e LOGGING_LEVEL_APP="debug" \
  -e LOGGING_LEVEL_GORM="info" \
  -e LOGGING_MAXSIZE=20 \
  -e LOGGING_MAXAGE=60 \
  -e LOGGING_COMPRESS=true \
  -e SONIC_MODE=development \
  -e SQLITE3_ENABLE=true \
  -e SONIC_WORK_DIR=/sonic \
  gosonic/sonic:latest
```

## 配置优先级

1. **环境变量**（最高优先级）- 通过`-e`传递
2. **配置文件**（config.yaml）- 如果存在
3. **默认值**（最低优先级）- 代码中定义的默认值

## 无配置文件运行

通过以上修改，Sonic现在可以完全通过环境变量运行，无需任何配置文件：

```bash
# 完全通过环境变量运行（最小配置）
docker run -d \
  --name sonic \
  -p 9090:9090 \
  -v sonic_data:/sonic \
  -e SERVER_PORT=9090 \
  -e SONIC_MODE=production \
  -e SQLITE3_ENABLE=true \
  sonic:latest

# 完全通过环境变量运行（完整配置）
docker run -d \
  --name sonic \
  -p 8080:8080 \
  -v sonic_data:/sonic \
  -e SERVER_HOST=0.0.0.0 \
  -e SERVER_PORT=8080 \
  -e LOGGING_FILENAME="sonic.log" \
  -e LOGGING_LEVEL_APP="info" \
  -e LOGGING_LEVEL_GORM="warn" \
  -e LOGGING_MAXSIZE=10 \
  -e LOGGING_MAXAGE=30 \
  -e LOGGING_COMPRESS=false \
  -e SQLITE3_ENABLE=true \
  -e SONIC_MODE="production" \
  -e SONIC_WORK_DIR="/sonic" \
  -e SONIC_LOG_DIR="/sonic/logs" \
  -e SONIC_ADMIN_URL_PATH="admin" \
  sonic:latest
```

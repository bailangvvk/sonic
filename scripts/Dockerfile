# 构建阶段 - 精简版
FROM golang:1.19.3-alpine as builder

# 只安装必需的工具
RUN apk --no-cache add git ca-certificates gcc g++

COPY . /go/src/github.com/go-sonic/sonic/
WORKDIR /go/src/github.com/go-sonic/sonic

# 构建应用（精简ldflags，只保留必需的参数）
RUN CGO_ENABLED=1 GOOS=linux go build -o sonic -ldflags="-s -w" -trimpath .

# 准备运行文件并立即清理
RUN mkdir -p /app && \
    cp sonic /app/ && \
    cp -r conf /app/ && \
    cp -r resources /app/ && \
    cp scripts/docker_init.sh /app/ && \
    # 清理构建缓存
    rm -rf /go/pkg /go/src

# 运行时阶段 - 使用scratch最小镜像
FROM scratch

# 复制应用文件和必要的证书
COPY --from=builder /app/ /app/
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# 必要的环境变量
ENV TZ=Asia/Shanghai \
    SONIC_WORK_DIR=/sonic \
    SONIC_LOG_DIR=/sonic/logs

# 创建必要的目录结构
RUN mkdir -p /sonic/logs

VOLUME /sonic
EXPOSE 8080
WORKDIR /sonic

# 使用exec形式启动
CMD ["/app/sonic"]

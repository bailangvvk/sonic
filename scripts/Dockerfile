# 构建阶段
FROM golang:1.19.3-alpine AS builder

WORKDIR /src

# 合并所有构建步骤到一个RUN命令（减少镜像层数）
RUN set -eux && \
    # 安装依赖（包括CGO编译所需工具）
    apk add --no-cache --virtual .build-deps \
    git \
    ca-certificates \
    gcc \
    g++ \
    build-base \
    && \
    # 克隆源代码
    git clone --branch menory --depth 1 https://github.com/bailangvvk/sonic . && \
    # 构建应用
    CGO_ENABLED=1 GOOS=linux go build -o /app/sonic -trimpath . && \
    # 准备运行文件
    mkdir -p /app && \
    cp -r conf /app/ && \
    cp -r resources /app/ && \
    cp scripts/docker_init.sh /app/ && \
    # 确保可执行文件权限
    chmod +x /app/sonic

# 运行阶段
# FROM busybox:musl
FROM alpine:latest AS runpod

# 复制应用文件
COPY --from=builder /app/ /app/

# 设置时区
ENV TZ=Asia/Shanghai

VOLUME /sonic
EXPOSE 8080
WORKDIR /sonic

CMD /app/docker_init.sh && /app/sonic

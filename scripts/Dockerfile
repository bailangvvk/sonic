# 构建阶段
# FROM golang:1.19.3-alpine as builder
FROM golang:1.21-alpine AS builder

# 设置工作目录
WORKDIR /src

# 一次性安装所有依赖并克隆代码、构建应用
RUN set -eux && \
    # 安装构建工具
    apk add --no-cache --virtual .build-deps \
    git \
    ca-certificates \
    gcc \
    g++ && \
    # 浅克隆源代码
    git clone --branch menory --depth 1 https://github.com/bailangvvk/sonic . && \
    # 构建应用（Hugo风格激进优化）
    CGO_ENABLED=1 GOOS=linux go build -o sonic \
        -ldflags="-s -w -extldflags '-static' -X github.com/go-sonic/sonic/consts.SonicVersion=docker" \
        -trimpath . && \
    # 使用strip进一步减小二进制大小
    strip --strip-all sonic && \
    # 准备运行文件
    mkdir -p /app && \
    cp sonic /app/ && \
    cp -r conf /app/ && \
    cp -r resources /app/ && \
    cp scripts/docker_init.sh /app/ && \
    # 清理构建缓存
    rm -rf /go/pkg /src /var/cache/apk/* /tmp/*

# 运行时阶段
FROM alpine:latest

# 复制应用文件
COPY --from=builder /app/ /app/

# 设置时区和清理缓存
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    rm -rf /var/cache/apk/*

# 创建必要的目录结构
RUN mkdir -p /sonic/logs

VOLUME /sonic
EXPOSE 8080

WORKDIR /sonic
CMD /app/docker_init.sh && /app/sonic

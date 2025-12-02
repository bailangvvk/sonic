# 阶段1：构建阶段（合并所有命令减少层数）
FROM golang:1.19.3-alpine AS builder

# 合并：安装所有依赖、设置工作目录、复制代码、构建应用、准备运行文件
RUN set -eux && \
    apk add --no-cache \
        git \
        ca-certificates \
        gcc \
        g++ \
        build-base && \
    mkdir -p /go/src/github.com/go-sonic/sonic

WORKDIR /go/src/github.com/go-sonic/sonic

COPY . .

ARG BUILD_COMMIT
ARG BUILD_TIME
ARG SONIC_VERSION

# 合并：构建应用并准备运行目录
RUN set -eux && \
    CGO_ENABLED=1 GOOS=linux \
    go build -o sonic \
    -ldflags="-s -w \
        -X github.com/go-sonic/sonic/consts.SonicVersion=${SONIC_VERSION} \
        -X github.com/go-sonic/sonic/consts.BuildCommit=${BUILD_COMMIT} \
        -X github.com/go-sonic/sonic/consts.BuildTime=${BUILD_TIME}" \
    -trimpath . && \
    mkdir -p /app/conf /app/resources && \
    cp sonic /app/ && \
    cp -r conf /app/ && \
    cp -r resources /app/ && \
    cp scripts/docker_init.sh /app/ && \
    rm -rf /go/pkg /go/src /var/cache/apk/* /tmp/*

# 阶段2：运行时阶段（使用 busybox:musl，合并命令）
FROM busybox:musl

# 合并：复制应用文件、时区文件、CA证书
COPY --from=builder /app/ /app/
COPY --from=builder /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# 设置时区环境变量
ENV TZ=Asia/Shanghai

VOLUME /sonic
EXPOSE 8080
WORKDIR /sonic

CMD /app/docker_init.sh && /app/sonic
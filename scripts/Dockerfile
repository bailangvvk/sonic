# 构建阶段
FROM golang:1.19.3-alpine as builder

# 安装最小必需构建工具
RUN apk --no-cache add git ca-certificates gcc g++

COPY . /go/src/github.com/go-sonic/sonic/
WORKDIR /go/src/github.com/go-sonic/sonic

# 定义构建参数（提供默认值避免构建失败）
ARG SONIC_VERSION="latest"
ARG BUILD_COMMIT="unknown"
ARG BUILD_TIME="unknown"

# 构建应用（保持ldflags但使用默认值）
RUN CGO_ENABLED=1 GOOS=linux go build -o sonic -ldflags="-s -w \
    -X github.com/go-sonic/sonic/consts.SonicVersion=${SONIC_VERSION} \
    -X github.com/go-sonic/sonic/consts.BuildCommit=${BUILD_COMMIT} \
    -X github.com/go-sonic/sonic/consts.BuildTime=${BUILD_TIME}" \
    -trimpath .

# 准备运行文件并在同一层清理
RUN mkdir -p /app && \
    cp sonic /app/ && \
    cp -r conf /app/ && \
    cp -r resources /app/ && \
    cp scripts/docker_init.sh /app/ && \
    # 清理构建缓存和源代码
    rm -rf /go/pkg /go/src /var/cache/apk/* /tmp/*

# 运行时阶段 - 使用精简的alpine
FROM alpine:latest

# 复制应用文件（只复制必需的文件）
COPY --from=builder /app/ /app/

# 只安装运行时必需的最小包
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone && \
    # 清理apk缓存
    rm -rf /var/cache/apk/*

# 创建必要的目录结构
RUN mkdir -p /sonic/logs

VOLUME /sonic
EXPOSE 8080

WORKDIR /sonic
CMD /app/docker_init.sh && /app/sonic

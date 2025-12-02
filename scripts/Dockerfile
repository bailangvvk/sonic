# 构建阶段
FROM golang:1.19.3-alpine AS builder

WORKDIR /src

# 安装git并克隆源代码
RUN apk add --no-cache git ca-certificates && \
    git clone --branch menory --depth 1 https://github.com/bailangvvk/sonic .

# 构建应用（简化版本）
RUN CGO_ENABLED=1 GOOS=linux go build -o sonic -trimpath .

# 准备运行文件
RUN mkdir -p /app && \
    cp sonic /app/ && \
    cp -r conf /app/ && \
    cp -r resources /app/ && \
    cp scripts/docker_init.sh /app/

# 运行阶段
FROM busybox:musl

# 复制应用文件
COPY --from=builder /app/ /app/

# 设置时区
ENV TZ=Asia/Shanghai

VOLUME /sonic
EXPOSE 8080
WORKDIR /sonic

CMD /app/docker_init.sh && /app/sonic

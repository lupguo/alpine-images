# Build Golang
FROM alpine:latest as builder

# ARG Setting
ARG VERSION=1.13
ARG GOOS=""
ARG GOARCH=""

# ENV Setting
LABEL Description="Build Go on alpine" Version="Go version $VERSION"

# Go source
COPY ./go-source /usr/local/go

# Building
RUN set -eux; \
    # Go 1.4编译的依赖环境安装
    apk add --no-cache --virtual .build-deps \
        bash gcc musl-dev git; \
    # 编译go1.4的版本
    export GOROOT_BOOTSTRAP=/usr/local/go1.4 \
        INSTALL=/usr/local \
        GOROOT=/usr/local/go \
    ; \ 
    cd $INSTALL \
        #&& git clone https://github.com/golang/go.git
        && cp -a go go1.4 && cd go1.4/src \
        && git checkout release-branch.go1.4 \
        && ./make.bash \
    ; \
    # 编译指定版本go
    cd $GOROOT/src \
        && git checkout release-branch.go$VERSION \
        && if [[ "$GOOS" != "" && "$GOARCH" != "" ]]; then \
            GOOS=$GOOS GOARCH=$GOARCH ./bootstrap.bash; else ./make.bash; fi \
        # 移除相关依赖
        && apk del .build-deps \
        && rm -rf $GOROOT_BOOTSTRAP \
        && rm -rf $GOROOT/.git \
        && $GOROOT/bin/go version

# Build a clean Go image
FROM alpine:latest

# 复制构建完成的最新版本Go到当前镜像
COPY --from=builder /usr/local/go /usr/local/go

# Go 编译环境初始化
ENV GOPROXY=https://goproxy.io \
    GOROOT=/usr/local/go \
    GOPATH=/go 
ENV PATH=$GOPATH/bin:$GOROOT/bin:$PATH

# 其他初始化相关
RUN echo "alias ll='ls -al'" >> ~/.profile \
    && echo "export PATH=$GOPATH/bin:$GOROOT/bin:$PATH" >> ~/.profile

# 指定Volume和Workdir
VOLUME ["/data/go"]
WORKDIR "/data/go"

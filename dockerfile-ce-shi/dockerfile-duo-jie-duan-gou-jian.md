# Dockerfile多阶段构建

 Docker 17.05版本以后，新增了Dockerfile多阶段构建。所谓多阶段构建，实际上是允许一个Dockerfile 中出现多个 `FROM` 指令。但最后生成的镜像，仍以最后一条 `FROM` 为准，之前的 `FROM` 会被抛弃。

```
FROM [--platform=<platform>] <image> [AS <name>]
```

例如我们构建一个golang程序，需要用到go语言的编译环境，但把go语言的编译工具和库写到镜像里会十分浪费空间，这时候就可以用到多阶段构建，例如：

```text
# 编译阶段 命名为 base
FROM golang:1.7.3 as base 
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
# 运行阶段
FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=base /go/src/app .
CMD ["./app"]  
```

**使用外部镜像作为stage**

使用多阶段构建时，不仅可以从Dockerfile中创建的镜像中进行复制，还可以使用`COPY –from`指令从单独的image中复制，使用本地image名称，本地或Docker注册表中可用的标记或标记ID。  
  
例如：

```text
COPY --from=nginx:latest /etc/nginx/nginx.conf /usr/local/nginx/nginx.conf
```


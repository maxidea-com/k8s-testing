---
description: '指令参考：https://docs.docker.com/engine/reference/builder/'
---

# Dockerfile入门

**我们先理解一下什么是Dockerfile：**

> Dockerfile 是软件的原材料，Docker 镜像是软件的交付品，而 Docker 容器则可以认为是软件的运行态。从应用软件的角度来看，Dockerfile、Docker 镜像与 Docker 容器分别代表软件的三个不同阶段，Dockerfile 面向开发，Docker 镜像成为交付标准，Docker 容器则涉及部署与运维，三者缺一不可，合力充当 Docker 体系的基石。

Dockerfile 中的每一条命令，都在 Docker 镜像中以一个独立镜像层的形式存在，如下图：

![](../.gitbook/assets/image%20%289%29.png)

## 例子1：

创建一个Dockerfile文件：

```text
FROM alpine:3.11

RUN apk update && apk --no-cache add python3 curl bind-tools iproute2 iptables ipvsadm
RUN pip3 install --no-cache-dir --upgrade pip && \
  pip3 install --no-cache-dir -q Flask==0.12.4 requests==2.21.0

ADD demo.py /usr/local/bin/

ENV DEPLOYENV='Production' RELEASE='Stable' PS1="[\u@\h \w]\\$ "

EXPOSE 80

VOLUME ["/simon-testing/dockerfile/data"]

ENTRYPOINT python3 /usr/local/bin/demo.py
```

在同一目录下创建一个demo.py，使用Flask框架简单搭建一个服务：

```text
#!/usr/bin/python3
#
from flask import Flask, request, abort, Response, jsonify as flask_jsonify, make_response
import argparse 
import sys, os, getopt, socket, json, time

app = Flask(__name__)

start_time = time.time()

@app.route('/')
def index():
        return ('ClientIP: {}, ServerName: {}, ''ServerIP: {}!\n'.format(request.remote_addr, socket.gethostname(),
                                                                socket.gethostbyname(socket.gethostname())))

def main(argv):
    port = 80
    host = '0.0.0.0'
    debug = False

    if os.environ.get('PORT') is not None:
        port = os.environ.get('PORT')

    if os.environ.get('HOST') is not None:
        host = os.environ.get('HOST')

    try:
        opts, args = getopt.getopt(argv,"vh:p:",["verbose","host=","port="])
    except getopt.GetoptError:
        print('server.py -p <portnumber>')
        sys.exit(2)
    for opt, arg in opts:
        if opt in ("-p", "--port"):
            port = arg
        elif opt in ("-h", "--host"):
            host = arg
        elif opt in ("-v", "--verbose"):
            debug = True

    app.run(host=str(host), port=int(port), debug=bool(debug))


if __name__ == "__main__":
    main(sys.argv[1:])
```

然后可以运行指令`docker build . -t demoapp:v0.1`

第一次运行大约需要20分钟，八个步骤对应Dockerfile里面的八层：

```text
Sending build context to Docker daemon  4.608kB
Step 1/8 : FROM alpine:3.11
 ---> f70734b6a266
Step 2/8 : RUN apk update && apk --no-cache add python3 curl bind-tools iproute2 iptables ipvsadm
 ---> Using cache
 ---> 2ebf276753f5
Step 3/8 : RUN pip3 install --no-cache-dir --upgrade pip &&   pip3 install --no-cache-dir -q Flask==0.12.4 requests==2.21.0
 ---> Using cache
 ---> bcf41b3ffeff
Step 4/8 : ADD demo.py /usr/local/bin/
 ---> Using cache
 ---> 8bab90aeff83
Step 5/8 : ENV DEPLOYENV='Production' RELEASE='Stable' PS1="[\u@\h \w]\\$ "
 ---> Using cache
 ---> 2421ecaf4dea
Step 6/8 : EXPOSE 80
 ---> Using cache
 ---> 437beef4b0ef
Step 7/8 : VOLUME ["/simon-testing/dockerfile/data"]
 ---> Using cache
 ---> 31f154aaef03
Step 8/8 : ENTRYPOINT python3 /usr/local/bin/demo.py
 ---> Using cache
 ---> 6d4e0298f516
Successfully built 6d4e0298f516
Successfully tagged demoapp:v0.1
```

镜像打包完成后，用`docker run --name myapp demoapp:v0.1`运行起来，容器内默认会执行`python3 /usr/local/bin/demo.py`

```text
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```

然后打开另一个terminal，查看一下容器ip地址，并用curl访问容器80端口：

```text
# docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' myapp
172.17.0.2
root@09-1:curl 172.17.0.2
ClientIP: 172.17.0.1, ServerName: 48bcba8c4858, ServerIP: 172.17.0.2!
```

其他替换entrypoint和进入容器的操作，请查阅前面的章节。



参考资料：[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)






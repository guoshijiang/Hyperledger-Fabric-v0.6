#gRPC-Go

[gRPC](http://www.grpc.io/)是Go实现的：一个高性能，开源，将移动和HTTP/2放在首位通用的RPC框架， 有关详细信息，请参阅[gRPC快速入门](http://www.grpc.io/docs/)指南。

安装
------------

要安装此软件包，您需要安装Go并在计算机上设置您的Go工作区。 安装库的最简单的方法是运行:

```
$ go get google.golang.org/grpc
```

先决条件
-------------

需要Go 1.5或更高版本。

关于所用版本的使用说明：在基准测试中显着提高性能的grpc-go已经版本升级，从1.5到升级最新1.7.1。

 来自https://golang.org/doc/install 网站, 一种安装最新版本的go方法就是:
```
$ GO_VERSION=1.7.1
$ OS=linux
$ ARCH=amd64
$ curl -O https://storage.googleapis.com/golang/go${GO_VERSION}.${OS}-${ARCH}.tar.gz
$ sudo tar -C /usr/local -xzf go$GO_VERSION.$OS-$ARCH.tar.gz
$ # Put go on the PATH, keep the usual installation dir
$ sudo ln -s /usr/local/go/bin/go /usr/bin/go
$ rm go$GO_VERSION.$OS-$ARCH.tar.gz
```

约束
-----------
grpc包只依赖于标准的Go包和少量的异常. 如果您的贡献引入了不在列表中的新依赖项[list](http://godoc.org/google.golang.org/grpc?imports), 您需要与gRPC-Go作者和顾问讨论一下.

文档
-------------
有关包和API描述，请参阅[API文档](https://godoc.org/google.golang.org/grpc)，并在示例目录中查找示例

状态
------
    GA
    FAQ
---

#### 编译错误, 未定义: grpc.SupportPackageIsVersion

请更新proto包, gRPC 包和重建proto文件:
   
    - go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
    - go get -u google.golang.org/grpc
    - protoc --go_out=plugins=grpc:. *.proto

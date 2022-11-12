---
layout:     post
title:      "抛弃RESTful，在浏览器端使用gRPC"
subtitle:   "gRPC in browser"
date:       2022-03-06 19:00:00
author:     "Reverie"
catalog: false
header-style: text
tags:
- backend
---

> 抛弃RESTful，在浏览器端使用gRPC

## 1. RESTful API vs gRPC

### 1.1 RESTful

RESTFUL是一种网络应用程序的设计风格和开发方式，基于HTTP，可以使用XML格式定义或JSON格式定义。

REST 指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful。

Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态的。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合云计算之类的环境。客户端可以缓存数据以改进性能。

在服务器端，应用程序状态和功能可以分为各种资源。资源是一个有趣的概念实体，它向客户端公开。资源的例子有：应用程序对象、数据库记录、算法等等。每个资源都使用 URI (Universal Resource Identifier) 得到一个唯一的地址。所有资源都共享统一的接口，以便在客户端和服务器之间传输状态。使用的是标准的 HTTP 方法，比如 GET、PUT、POST 和 DELETE。Hypermedia 是应用程序状态的引擎，资源表示通过超链接互联。

#### RESTFUL特点

1. 每一个URI代表1种资源；
2. 客户端使用GET、POST、PUT、DELETE4个表示操作方式的动词对服务端资源进行操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源；
3. 通过操作资源的表现形式来操作资源；
4. 资源的表现形式是XML或者HTML；
5. 客户端与服务端之间的交互在请求之间是无状态的，从客户端到服务端的每个请求都必须包含理解请求所必需的信息。

#### REST 并非适用所有场景

本文给了我们一个更大的视角看待日常开发中的接口问题，对于奋战在一线的前端同学，接触到 90% 的接口都是非 REST 规则的 Http 接口，能真正落实 REST 的团队其实非常少。这其实暴露了一个重要问题，就是 REST 所带来的好处，在整套业务流程中到底占多大的比重？

不仅接口设计方案的使用要分场景，针对某个接口方案的重要性也要再继续细分：在做一个开放接口的项目，提供 Http 接口给第三方使用，这时必须好好规划接口的语义，所以更容易让大家达成一致使用 REST 约定；而开发一个产品时，其实前后端不关心接口格式是否规范，甚至在开发内网产品时，性能和冗余都不会考虑，效率放在了第一位。所以第一点启示是，不要埋冤当前团队业务为什么没有使用某个更好的接口约定，因为接口约定很可能是业务形态决定的，而不是凭空做技术对比从而决定的。

### 1.2 gRPC

#### 1.2.1protoBuf

Protobuf是一种平台无关、语言无关、可扩展且轻便高效的序列化数据结构的协议，可以用于**网络通信**和**数据存储**。

#### 1.2.2 gRPC

gRPC是google于2015年开源的一个RPC框架。它是基于protoBuf和HTTP/2实现的。它是一个现代化、开源、基于HTTP/2协议的 RPC 框架，使用强类型二进制数据（protobuf）进行通讯。目前提供 C、Java 和 Go 语言版本，分别是：grpc, grpc-java, grpc-go. 其中 C 版本支持 C, C++, Node.js, Python, Ruby, Objective-C, PHP 和 C# 等。

gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。

### 1.3 对比 RESTful API和 gRPC

#### 1.3.1 对比表

| 属性      | gRPC                         | REST                            |
| --------- | ---------------------------- | ------------------------------- |
| 名称      | Google Remote Procedure Call | REpresentational State Transfer |
| 数据格式  | Protobuf                     | JSON(典型)                      |
| 可读性    | 不可读二进制                 | 可读                            |
| HTTP      | HTTP/2                       | HTTP 1.1/HTTP/2                 |
| 性能      | 更快                         | 较多性能损耗                    |
| 类型      | 强类型，类型安全             | 弱类型                          |
| 跨语言    | 是                           | 是                              |
| 客户端    | 需要实现客户端               | 不需要实现客户端                |
| HTTP 方法 | 任何方法                     | GET/PUT/DELETE/POST/…           |

因为gRPC是基于protoBuf，所以数据传输是二进制格式，而REST一般采用XML或JSON，所以是text格式。因此对于数据序列化、反序列化，gRPC效率更高，性能自然更好。但是理论上REST也可以采用protoBuf格式，这里只是说的一般情况，因为目前一提到REST，基本就是HTTP/1.1 + JSON。从另一个角度来看，XML或JSON是human-readable，所以更便于调试。

gRPC相比REST具有更高的性能的另一个原因是采用了HTTP/2。HTTP/2的多路复用、server端主动push等功能使gRPC增色不少。当然，REST也可以采用HTTP/2，而且HTTP/2兼容HTTP/1.1。

#### 1.3.2 建议

基于protcbuf对跨语言、跨系统数据模型以及API服务定义的能力，加上二进制通讯在网络的新能优势，建议在各个通讯场景中统一使用protobuf和gRPC。可实现一个服务器，对应多个平台客户端（C/S、Android、iOS、web）统一通讯协议。

## 2. gRPC 的语言支持

gRPC 可生成多种常用语言的服务stub，开发人员可根据stub实现具体业务逻辑。目前支持的语言有： C、Java 和 Go 语言版本，分别是：grpc, grpc-java, grpc-go.

其中 C 版本支持 C, C++, Node.js, Python, Ruby, Objective-C, PHP 和 C# 等.

1. C++

2. C#

3. Dart

4. **golang**

5. **java**

6. **csharp**

7. **php**

8. Python

9. Ruby

10. **swift**

11. **object c**

12. **JavaScript（nodejs）

13. ***web JavaScript (web browser)*** ，

    在浏览器端没有直接的gRPC支持。这也是写这篇文件的初衷，对浏览器端gRPC客户端的实现进行探索对比。

## 3. web JavaScript – gRPC in web browser

在web浏览器端，目前有3个选择，分别是：

| 方案                        | 作者           | 方案简介                                                     | 特点                                                         |
| --------------------------- | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| grpc/grpc-web               | gRPC官方       | 通过envoy进行反向代理，对gRPC服务和web http/1.1进行互相翻译。必须有一个envoy或nginx代理。客户端通过protoc生成js或ts代码 | 官方支持。直接生成commonJS或TS客户端代码。需要独立proxy（envoy或nginx） |
| improbable-eng/grpc-web     | improbable-eng | 嵌入代码在gRPC服务器端，直接把gRPC协议翻译为gRPC-web协议。同时生成浏览器端TypeScript代码，供浏览器直接调用服务。 | 直接生成TSt客户端代码，可在浏览器直接使用。proxy代理内嵌在服务器端。 |
| grpc-ecosystem/grpc-gateway | grpc-ecosystem | 通过protoc插件的方式，对protobuf定义里的annotation进行处理，自动生成反向代理服务器代码以及RESTful API 的swagger 描述信息文件。 | 直接生成RESRful API代理服务器、REST API文档。需要手写浏览器端RESTful client代码。 |

### 3.1 官方 grpc/grpc-web

#### 3.1.1 Overview

https://github.com/grpc/grpc-web

gRPC-Web 提供了一个 JavaScript库，可以让浏览器客户端访问 gRPC 服务。 现在已经足够稳定用在生产环境了。

gRPC-Web 客户端通过一个**特殊的网关代理**连接到gRPC 服务。gRPC-web库当前版本默认使用Envoy作为网关代理，对Envoy的支持已经内置在gRPC-Web 库里面。中间代理可以是nginx或envoy。

protoc 的 gRPC 代码生成器插件下载： https://github.com/grpc/grpc-web/releases

#### 3.1.2 示例

可以从 [Hello World Guide](https://github.com/grpc/grpc-web/blob/master/net/grpc/gateway/examples/helloworld/) 来快速开始体验 gRPC-Web 。根据指南，可以学到如何：

- 使用 protocol buffers 定义服务
- 用 NodeJS 实现一个简单的 gRPC 服务
- 配置 Envoy 代理
- 为客户端生成 protobuf 消息类和客户端服务stub
- 把所有 JS 编译成一个浏览器可以非常容易使用的静态库。

### 3.2 improbable-eng/grpc-web

#### 3.2.1 采用[gRPC-Web协议](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md)

https://github.com/improbable-eng/grpc-web

> [gRPC-Web协议（gRPC官方）](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md) 是一个划时代的作品。它让现代浏览器可以调用gRPC服务。由于浏览器的限制，gRPC-Web协议跟原生的gRPC协议有所不同。gRPC-web协议设计原则是让浏览器和gRPC服务之间的代理服务器可以更容易的对协议进行翻译。

如果你要找 NodeJS 的 gRPC 支持库，这里是 ***[官方 Node.js gRPC 库](https://www.npmjs.com/package/grpc).***。这个包支持 Node.js，但是要求服务器必须有 gRPC-Web 兼容层。

#### 3.2.2 improbable-eng/grpc-web简介

improbable-eng/grpc-web 库基于 Golang 和 TypeScript:

- **[`grpcweb`](https://github.com/improbable-eng/grpc-web/blob/master/go/grpcweb)** - 这是一个可以把现有 grpc.Server 包装成 gRPC-web http.Handler 的 GoLang 包。同时支持HTTP2和HTTP/1.1。
- **[`grpcwebproxy`](https://github.com/improbable-eng/grpc-web/blob/master/go/grpcwebproxy)** - 基于GoLang的独立反向代理服务，为经典 gRPC 服务（比如：java或c++）提供代理，把他们的服务通过 gRPC-web 暴露给现代浏览器。
- **[`ts-protoc-gen`](https://github.com/improbable-eng/ts-protoc-gen)** - protoc (protocol buffers 编译器) 的 TypeScript 代码生成插件，生成 TypeScript 强类型消息类和方法定义代码。
- **[`@improbable-eng/grpc-web`](https://github.com/improbable-eng/grpc-web/blob/master/client/grpc-web)** - 供浏览器和 NodeJS 使用的 TypeScript gRPC-Web 客户端库。

#### 3.2.3 为什么使用 improbable-eng/grpc-web?

对于 gRPC-Web, 要创建定义良好、易于解释的浏览器前端代码和微服务之间的API，是一件及其简单的事情。前端开发有以下意义重大的好处：

- 不再需要到处寻找API文档 —— .proto 就是典型的API规范格式。
- 不在需要手写 JSON 调用对象 —— 所有的请求和响应都是强类型、代码自动生成的，我们在IDE可以方便的使用代码自动提示，提高编程效率。
- 不再需要处理HTTP的method、header、body以及底层网络 —— 所有事情都有 grpc.invoke 处理了。
- 不用需要反复猜测错误代码的含义 —— gRPC 状态码是典型的在 API 中表示问题的方法。
- 不需要为了避免并发链接而采用低效的一次性网络连接 —— gRPC-Web 是基于 HTTP2的，支持一个连接上多路传输多路数据流。
- 没有从服务器读取流式数据的问题 —— gRPC-Web 支持一对一远程调用和一对多数据流请求。
- 处理新二进制时，更少的数据处理错误 —— 前端、后端同时兼容的请求及响应.

总而言之，gRPC-Web 把前端代码和微服务之间的交互从手工编写HTTP请求代码转换成已经定义好的用户业务函数。

#### 3.2.4 客户端侧文档（ grpc-web）

注意：你需要为 gRPC 服务添加 gRPC-Web 兼容能力，可以通过 **[`improbable-eng/grpcweb`](https://github.com/improbable-eng/grpc-web/blob/master/go/grpcweb) 或 [`improbable-eng/grpcwebproxy`](https://github.com/improbable-eng/grpc-web/blob/master/go/grpcwebproxy)** 实现。

另外请参考 [`improbable-eng/grpc-web` 客户端API 文档](https://github.com/improbable-eng/grpc-web/blob/master/client/grpc-web)

#### 3.2.5 实例

1. 创建一个用 TypeScript 前端调用的 Golang gRPC 服务。 参考 [例子](https://github.com/improbable-eng/grpc-web/blob/master/client/grpc-web-react-example)。例子包含了实现功能 的大部门初始化代码，这里是从例子里提取的代码：

   你可以用 `.proto` 文件来定义你的服务。在这个例子里，我定义了一个普通的 RPC (`GetBook`) 服务，以及一个服务端数据流 RPC (`QueryBooks`) 服务：

2. 在 GoLang 里实现业务的服务（或使用其他 gRPC 支持的其他语言）：

3. 你可以用一下 TS 代码在浏览器调用上面的 gRPC 服务了（也可以用转换后的 Javascript 代码）：

#### 3.2.6 与 React 一起使用

[在 React 和 golang 一起使用 gRPC-Web 实例](https://github.com/easyCZ/grpc-web-hacker-news)

### 3.3 grpc-ecosystem/grpc-gateway

#### 3.3.1 简介

https://github.com/grpc-ecosystem/grpc-gateway

这个 grpc-gateway 是Google protocol buffers 编译器[protoc](https://github.com/protocolbuffers/protobuf)的一个插件。它读取 protobuf 服务定义文件，然后生成 一个反向代理服务器，用来“把一个RESTful API调用翻译成gRPC”。这个服务器根据你的定义文件中的 [`google.api.http`](https://github.com/googleapis/googleapis/blob/master/google/api/http.proto#L46) annotation来生成的。这可以让你在服务器里同时提供给RPC和RESTful风格的API服务。

#### 3.3.2 组成

由3个程序组成。

- `protoc-gen-grpc-gateway` —— protoc的插件，用来生成 gRPC gateway 服务器的代码
- `protoc-gen-swagger` —— protoc的插件，用来生成 swagger 的 API 描述文件
- `protoc-gen-go` —— prococ的插件，用来生成 golang 的 message 定义和 gRPC 的 stub 代码

#### 3.3.3 实例

1. #### protobuf 定义文件里添加 import 和 RESTful API 描述

   ```go
   import "google/api/annotations.proto";
   ```

   

2. #### 修改 gRPC 方法定义

   ```protobuf
   service YourService {
     rpc Echo(StringMessage) returns (StringMessage) {
       option (google.api.http) = {
         post: "/v1/example/echo"
         body: "*"
       };
     }
    }
   ```

3. #### 生成 gRPC stub

   基于你的 `path/to/your_service.proto` 文件，用下面的命令行生成 Golang gRPC 代码:

   ```shell
   protoc -I/usr/local/include -I. \
     -I$GOPATH/src \
     -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
     --go_out=plugins=grpc:. \
     path/to/your_service.proto
   ```

   会产生一个 stub 文件：`path/to/your_service.pb.go`。

4. #### 跟平时语言实现你的 gRPC 服务

   1. (可选) 生成其他语言的 gRPC stub。
   2. 添加 googleapis-common-protos gem (或你喜欢的语言) 到项目的依赖项里。
   3. 实现你的 gRPC 服务 stubs。

5. #### 用 `protoc-gen-grpc-gateway`插件生成反向代理

   ```shell
   protoc -I/usr/local/include -I. \
     -I$GOPATH/src \
     -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
     --grpc-gateway_out=logtostderr=true:. \
     path/to/your_service.proto
   ```

   会生成一个反向代理服务器的代码： `path/to/your_service.pb.gw.go`.

6. #### 为 HTTP 反向代理服务写一个服务程序

   ```go
   package main
   
   import (
     "context"  // Use "golang.org/x/net/context" for Golang version <= 1.6
     "flag"
     "net/http"
   
     "github.com/golang/glog"
     "github.com/grpc-ecosystem/grpc-gateway/runtime"
     "google.golang.org/grpc"
   
     gw "path/to/your_service_package"  // Update
   )
   
   var (
     // command-line options:
     // gRPC server endpoint
     grpcServerEndpoint = flag.String("grpc-server-endpoint",  "localhost:9090", "gRPC server endpoint")
   )
   
   func run() error {
     ctx := context.Background()
     ctx, cancel := context.WithCancel(ctx)
     defer cancel()
   
     // Register gRPC server endpoint
     // Note: Make sure the gRPC server is running properly and accessible
     mux := runtime.NewServeMux()
     opts := []grpc.DialOption{grpc.WithInsecure()}
     err := gw.RegisterYourServiceHandlerFromEndpoint(ctx, mux,  *grpcServerEndpoint, opts)
     if err != nil {
       return err
     }
   
     // Start HTTP server (and proxy calls to gRPC server endpoint)
     return http.ListenAndServe(":8081", mux)
   }
   
   func main() {
     flag.Parse()
     defer glog.Flush()
   
     if err := run(); err != nil {
       glog.Fatal(err)
     }
   }
   ```

7. #### (可选) 用`protoc-gen-swagger`生成 swagger 格式的 RESTful API 文档

   ```shell
   protoc -I/usr/local/include -I. \
     -I$GOPATH/src \
     -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
     --swagger_out=logtostderr=true:. \
     path/to/your_service.proto
   ```

### 3.4 浏览器端的 gRPC-web 库选择

从上面的3个库对比来看， improbable-eng/grpc-web 会是更好的选择。原因有三：

1. 没有第三方的 proxy，而是直接内嵌在服务器端内。简化架构
2. 有强类型的浏览器侧 TS 客户端代码，无需手动写 RESTful client

## 4. gRPC 身份认证

### 4.1 [grpc-ecosystem/go-grpc-middleware](https://github.com/grpc-ecosystem/go-grpc-middleware)

gRPC的调用需要考虑安全认证问题。可以使用 [grpc-ecosystem/go-grpc-middleware](https://github.com/grpc-ecosystem/go-grpc-middleware) 实现。

在 gRPC 服务器中可以使用中间件 [参考：用GoLang开发gRPC service中间件](https://medium.com/@matryer/writing-middleware-in-golang-and-how-go-makes-it-so-much-fun-4375c1246e81#.gv7tdlghs) 在用户应用逻辑调用之前，或在客户端调用服务器之前执行相关操作。这是实现用户认证auth, 日志logging, 消息message, 数据验证validation, 重试或性能监控的完美方式。

### 4.2 Interceptors

*Please send a PR to add new interceptors or middleware to this list*

#### 4.1.2 认证Auth

- [`grpc_auth`](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/auth) - 一个可定制的 (通过`AuthFunc`) 用户认证中间件

#### 4.1.3 性能监控 Monitoring

- [`grpc_prometheus`![⚡](https://twemoji.maxcdn.com/v/13.0.0/72x72/26a1.png)](https://github.com/grpc-ecosystem/go-grpc-prometheus) - Prometheus 客户端和服务器端性能监控中间件
- [`otgrpc`![⚡](https://twemoji.maxcdn.com/v/13.0.0/72x72/26a1.png)](https://github.com/grpc-ecosystem/grpc-opentracing/tree/master/go/otgrpc) - [OpenTracing](http://opentracing.io/) 客户端和服务器端 interceptors
- [`grpc_opentracing`](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/tracing/opentracing) - [OpenTracing](http://opentracing.io/) 客户端和服务器端 interceptors，支持streaming 和 handler-returned 标签。

#### 4.1.4 客户端Client

- [`grpc_retry`](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/retry) - 一个通用 gRPC 响应代码重试机制, 客户端中间件。

#### 4.1.5 服务器端Server

- [`grpc_validator`](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/validator) - 从`.proto` 的选项生成入站消息验证中间件代码。
- [`grpc_recovery`](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/recovery) - 把 panics 转换为 gRPC 错误。
- [`ratelimit`](https://github.com/grpc-ecosystem/go-grpc-middleware/blob/master/ratelimit) - 用你自己的计量器对 grpc 进行流量限制。
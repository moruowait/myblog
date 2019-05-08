---
title: gRPC 详解
date: 2019-04-30 18:21:44
toc: true
tags:
- 技术名词
---

## 什么是 gRPC？

### 指南

这篇文档向你介绍了什么是 gRPC 和 protocol buffers。gRPC 可以使用 protocol buffers 作为其接口定义语言（IDL）和底层消息交换格式。如果你是 gRPC 和 protocol buffers 新手，那么读这篇文档可以帮助到你，如果你只是想看看 gRPC 是怎么运作的，直接看[快速开始](https://grpc.io/docs/quickstart/)。

### 概述

在 gRPC 中，Client 应用程序可以直接调用不同机器上的 Server 应用程序上的方法，就像它是一个本地对象一样，这使您更容易创建分布式应用程序和服务。与许多 RPC 系统一样，gRPC 基于定义服务的思想，指定可以使用参数和返回类型远程调用的方法。在 Server 端， Server 实现此接口并运行 gRPC 服务器来处理Client 调用。在Client 端，Client 有一个 Stub (在某些语言中称为 Client)，它提供与 Server 相同的方法。

{% asset_img overview.png This is an image %}

gRPC clients 和 servers 可以在各种环境中运行并相互通信 - 从 Google 中的服务器到你自己的桌面 - 并且可以用 gPRC 支持的任何语言编写。因此，例如，你可以使用 Java 轻松地创建一个 gRPC server，clints 端使用 GO、Python 或者 Ruby。除此之外，最新的 Google APIs 将具有接口的 gRPC 版本，让你可以轻松地将 Google 功能构建到应用程序中去。

### 使用 Protocol Buffers

gRPC 默认使用 [protocol buffers](https://developers.google.com/protocol-buffers/docs/overview)，用于序列化结构化数据的成熟开源机制（尽管它可以与其他数据格式，如：JSON 一起使用）。这是一个如何工作的快速介绍。如果你已经熟悉 protocol buffers，可以随时跳到下一部分。

使用 protocol buffers 的第一步是定义要在 proto 文件中序列化的数据的结构：这是一个带 .proto 扩展名的普通文本文件。 protocol buffers 数据被构造为消息，其中每个消息是包含一系列称为字段的 name-value 对的信息的小逻辑记录。这是一个简单的例子：

```proto
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

然后，一旦指定了数据结构，就可以使用 protocol buffers 编译器 `protoc` 从原型定义生成首选语言的数据访问类。这些为每个字段（如 `name()` 和 `set_name()`）提供了简单的访问器，以及将整个结构序列化/解析为原始字节的方法 - 例如，如果您选择的语言是 C++，则在上面的示例中运行编译器将生成上课了 `Person`。然后，您可以在应用程序中使用此类来填充，序列化和检索 Person protocol buffers 消息。

正如您将在我们的示例中更详细地看到的那样，您可以在普通的 proto 文件中定义 gRPC 服务，并将 RPC 方法参数和返回类型指定为 protocol buffers 消息：

```proto
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

gRPC 还使用 protoc 特殊的 gRPC 插件从 proto 文件生成代码。但是，使用 gRPC 插件，您将获得生成的 gRPC 客户端和服务器代码，以及用于填充，序列化和检索消息类型的常规 protocol buffers 代码。我们将在下面更详细地看一下这个例子。

你可以在 [Protocol Buffers 文档中](https://developers.google.com/protocol-buffers/docs/overview) 找到有关 protocol buffers 的更多信息，并了解如何使用 gRPC 插件快速获取和安装 protoc。

### Protocol Buffers 版本

虽然 protocol buffers 已经供开源用户使用了一段时间，但我们的实例使用了一种新的 protocol buffers 协议，称作 proto3，它具有略微简化的语法，一些有用的新功能并支持更多的语言。目前支持 Java、C++、Python、Object-C、C#、a little-runtime（Android Java）、Ruby、和 JavaScript，这些来自 [protocol buffers GitHub Repo](https://github.com/protocolbuffers/protobuf/releases)。也支持来自 [golang/protobuf GitHub repo](https://github.com/golang/protobuf) 的 Go 语言生成器，还有更多的语言正在开发中。你可以在 [proto3 语言指南](https://developers.google.com/protocol-buffers/docs/proto3)和每种语言的[参考文档](https://developers.google.com/protocol-buffers/docs/reference/overview)中找到更多信息。参考文档还包括 .proto 文件格式的[正式的规范](https://developers.google.com/protocol-buffers/docs/reference/proto3-spec)。

通常情况下，虽然你可以使用 proto2（当前的默认版本），但我们建议你将 proto3 和 gRPC 一起使用，因为它允许你使用全系列的 gRPC 支持的语言，以避免与 proto2 client 与 proto3 server 通信的兼容性问题，反之亦然。

## gRPC 概念

本文档介绍了一些关键的 gRPC 概念，概述了 gRPC 的体系结构和 RPC 生命周期。

### 概览

#### 服务定义

与许多 RPC 系统一样，gRPC 基于定义服务的思想，指定可以使用其参数和返回类型远程调用的方法。默认情况下，gRPC 使用 protocol buffers 作为接口定义语言（IDL）来描述服务接口和有效负载消息的结构。如果需要，可以使用其他替代方案。

```proto
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

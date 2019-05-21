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

```go
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

然后，一旦指定了数据结构，就可以使用 protocol buffers 编译器 `protoc` 从原型定义生成首选语言的数据访问类。这些为每个字段（如 `name()` 和 `set_name()`）提供了简单的访问器，以及将整个结构序列化/解析为原始字节的方法 - 例如，如果您选择的语言是 C++，则在上面的示例中运行编译器将生成上课了 `Person`。然后，您可以在应用程序中使用此类来填充，序列化和检索 Person protocol buffers 消息。

正如您将在我们的示例中更详细地看到的那样，您可以在普通的 proto 文件中定义 gRPC 服务，并将 RPC 方法参数和返回类型指定为 protocol buffers 消息：

```go
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

```go
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

gRPC 允许你定义四种服务方法：

- 一元 RPCs（Unary RPCs），客户端向服务器发送单个请求并返回单个响应，就像正常的函数调用一样。

```go
rpc SayHello(HelloRequest) returns (HelloResponse){
}
```

服务器流式 RPC（Server streaming RPCs），客户端向服务器发送请求并获取流以读取消息序列。客户端从返回的流中读取，直到没有更多消息。gRPC 保证单个 RPC 调用中的消息排序。

```go
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
}
```

客户端流式 RPC（Client streaming RPCs），客户端再次使用提供的流写入一系列消息并将其发送到服务器。一旦客户端写完消息，它就等待服务器读取它们并返回它的响应。gRPC 再次保证在单个 RPC 调用中的消息排序。

```go
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
}
```

双向流式 RPC（Bidirectional streaming RPCs），双方使用读写流发送一系列消息。这两个流独立运行，因此客户端和服务器可以按照自己喜欢的顺序进行读写：例如，服务器可以在写入响应之前等待接收所有客户端消息，或者它可以交替地读取消息然后写入消息，或者其他一些读写组合。保留每个流中的消息顺序。

```go
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
}
```

我们将在下面的 RPC 生命周期部分中更详细地介绍不同类型的RPC。

#### 使用 API

从 .proto 文件中的服务定义开始，gRPC 提供了生成客户端和服务器端代码的 protocol buffer 编译器插件。gRPC 用户通常在客户端调用这些 API，并在服务器端实现相应的 API。

- 在服务器端，服务器实现服务声明的方法，并运行 gRPC 服务器来处理客户端调用。gRPC 基础结构解码传入请求，执行服务方法并对服务响应进行编码
- 在客户端，客户端有一个称为存根的本地对象（stub）（对于某些语言，首选术语是 client），它实现与服务相同的方法。然后，客户端可以在本地对象上调用这些方法，将调用的参数包装在适当的 protocol buffer 消息类型中 - gRPC在将请求发送到服务器并返回服务器的 protocol buffer 响应之后查看。

#### 同步 vs 异步

在响应从服务器到达之前阻塞的同步 RPC 调用最接近 RPC 所期望的过程调用的抽象。另一方面，网络本质上是异步的，在许多情况下，能够在不阻塞当前线程的情况下启动 RPC 非常有用。

大多数语言的 gRPC 编程表面都有同步和异步两种版本。您可以在每种语言的教程和参考文档中找到更多信息（完整的参考文档即将推出）。

### PRC 生命周期

现在让我们仔细看看当 gRPC 客户端调用 gRPC 服务器方法时会发生什么。我们不会查看实现细节，您可以在我们特定语言的页面中找到有关这些内容的更多信息。

#### Unary RPC

首先让我们看一下最简单的 RPC 类型，客户端发送单个请求并返回单个响应。

- 客户端在 stub/client 对象上调用方法后，将通知服务器已使用客户端带着metedata （metadata）调用了产生了一次调用，方法名称和指定的截止时间（如果适用）调用RPC 。

- 然后，服务器可以立即发送回自己的初始metedata （必须在任何响应之前发送），或者等待客户端的请求消息 - 首先发生的是特定于应用程序的消息。

- 一旦服务器具有客户端的请求消息，它就会执行创建和填充其响应所需的任何工作。然后将响应与状态详细信息（状态代码和可选状态消息）以及可选的尾随metedata 一起返回（如果成功）到客户端。

- 如果状态为 OK，则客户端获取响应，从而完成客户端的调用。

#### Server streaming RPC

服务器流 RPC 类似于我们的简单示例，除了服务器在获取客户端的请求消息后发回响应流。在发回所有响应之后，服务器的状态详细信息（状态代码和可选状态消息）和可选的尾随metedata 将被发送回服务器端完成。一旦客户端拥有所有服务器的响应，客户端就会完成。

#### Client streaming RPC

客户端流式 RPC 也类似于我们的简单示例，除了客户端向服务器发送请求流而不是单个请求。服务器发送回单个响应，通常但不一定在收到所有客户端请求后，以及其状态详细信息和可选的尾随metedata 。

#### Bidirectional streaming RPC

在双向流式 RPC 中，再次调用由客户端发起的调用并且服务器端接收客户端的metedata 、方法名称和截止日期。服务器再次可以选择发回其初始metedata 或等待客户端开始发送请求。

接下来会发生什么取决于应用程序，因为客户端和服务器可以按任何顺序读写 - 流完全独立地运行。因此，例如，服务器可以等到它收到所有客户端的消息之后再写入其响应，或者服务器和客户端可以“乒乓”：服务器获取请求，然后发回响应，然后客户端发送另一个基于响应的请求，等等。

#### 截止日期/超时

gRPC 允许客户端指定在 RPC 因错误而终止之前，他们愿意等待 RPC 完成的时间 DEADLINE_EXCEEDED。在服务器端，服务器可以查询特定 RPC 是否已超时，或者剩余多少时间来完成 RPC。

指定截止日期或超时的方式因语言而异 - 例如，并非所有语言都有默认截止日期，某些语言 API 在截止日期（固定时间点）工作，某些语言 API 在超时方面工作（持续时间）。

#### RPC 终止

在 gRPC 中，客户端和服务器都对呼叫的成功进行独立和本地的确定，并且它们的结论可能不匹配。这意味着，例如，您可以在服务器端成功完成 RPC（“我已经发送了所有响应！”），但在客户端失败（“我的截止日期后响应已到达！”）。在客户端发送所有请求之前，服务器也可以决定完成。

#### 取消 RPC

客户端或服务器可以随时取消 RPC。取消立即终止 RPC，以便不再进行进一步的工作。它不是 “撤消”：取消之前所做的更改将不会被回滚。

#### metedata

metedata 是以键值对列表形式的特定 RPC 调用（例如[身份验证详细信息](#身份验证)）的信息，其中键是字符串，值通常是字符串（但可以是二进制数据）。metedata 对 gRPC 本身是不透明的 - 它允许客户端提供与服务器调用相关的信息，反之亦然。

对 metedata 的访问取决于语言。

#### 通道（channels）

gRPC channel 提供与指定主机和端口上的 gRPC 服务器的连接，并在创建客户端 stub（或某些语言中的 “client”）时使用。客户端可以指定 channel 参数来修改gRPC 的默认行为，例如打开和关闭消息压缩。一个 channel 是有状态的，包括 `connected` 和 `idle`。

gRPC 如何处理关闭 channels 与语言有关。某些语言还允许查询 channels 状态。

## 身份验证

### 认证

本文档概述了 gRPC 身份验证，包括我们内置的支持身份验证机制，如何插入您自己的身份验证系统，以及如何在我们支持的语言中使用 gRPC 身份验证的示例。

### 总览

gRPC 旨在与各种身份验证机制配合使用，可以轻松安全地使用 gRPC 与其他系统进行通信。您可以使用我们支持的机制 - 带或不带基于 Google 令牌的身份验证的 SSL / TLS - 或者您可以通过扩展我们提供的代码来插入您自己的身份验证系统。

gRPC 还提供了一个简单的身份验证 API，可让您 `Credentials` 在创建 channel 或调用时提供所有必要的身份验证信息。

### 支持的身份验证机制

gRPC 内置了以下身份验证机制：

- SSL / TLS：gRPC具有SSL / TLS集成，并促进使用SSL / TLS对服务器进行身份验证，并加密客户端和服务器之间交换的所有数据。可选机制可供客户端提供相互身份验证的证书。

- 使用 Google 进行基于令牌的身份验证：gRPC 提供了一种通用机制如（下所述），用于将基于 metedata 的凭据附加到请求和响应。某些身份验证流程提供了在通过 gRPC 访问 Google API 时获取访问令牌（通常是 OAuth2 令牌）的额外支持：您可以在下面的代码示例中看到它的工作原理。通常，必须使用此机制以及通道上的 SSL / TLS - Google 不允许没有 SSL / TLS 的连接，并且大多数 gRPC 语言实现都不允许您在未加密的通道上发送凭据。

警告：Google 凭据只能用于连接 Google 服务。将 Google 发布的 OAuth2 令牌发送到非 Google 服务可能会导致此令牌被盗并用于冒充客户端到 Google 服务。

### 身份验证 API

gRPC 提供了一个基于 Credentials 对象统一概念的简单身份验证 API，可以在创建整个 gRPC channel 或单个 call 时使用。

#### 凭证类型

凭证可以有两种类型：

- Channel credentials，附加到 `Channel`，例如 SSL 凭据。
- Call credentials，附加到 call（或者 C++ 中的 `ClientContext`）。

您还可以将这些组合在一起成为 `CompositeChannelCredentials`，例如，您可以指定 channel 的 SSL 详细信息以及在 channel 上进行的每个 call 的凭据。一个 `CompositeChannelCredentials` 将 `ChannelCredentials` 和 `CallCredentials` 连接到一起，创建一个新的 `ChannelCredentials`。结果将 `CallCredentials` 通过在 channel 上进行的每次调用发送与组合相关的认证数据。

例如，从 `SslCredentials` 和`AccessTokenCredentials` 你可以创建一个 `ChannelCredentials`。应用于 a 的结果 `Channel` 将为此通道上的每个调用发送相应的访问令牌。

单独的 `CallCredentials` 也可以使用 `CompositeCallCredentials`。`CallCredentials` 在调用中使用时产生的结果将触发发送与两者相关联的认证数据。

#### 使用客户端 SSL/TLS

现在让我们看一下如何 Credentials 使用我们支持的 auth 机制之一。这是最简单的身份验证方案，客户端只想验证服务器并加密所有数据。该示例使用的是 C ++，但所有语言的 API 都类似：您可以在下面的示例部分中看到如何在更多语言中启用SSL / TLS。

```go
// Create a default SSL ChannelCredentials object.
auto channel_creds = grpc::SslCredentials(grpc::SslCredentialsOptions());
// Create a channel using the credentials created in the previous step.
auto channel = grpc::CreateChannel(server_name, channel_creds);
// Create a stub on the channel.
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
// Make actual RPC calls on the stub.
grpc::Status s = stub->sayHello(&context, *request, response);
```
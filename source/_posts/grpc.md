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

<!-- more -->

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

```c++
// Create a default SSL ChannelCredentials object.
auto channel_creds = grpc::SslCredentials(grpc::SslCredentialsOptions());
// Create a channel using the credentials created in the previous step.
auto channel = grpc::CreateChannel(server_name, channel_creds);
// Create a stub on the channel.
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
// Make actual RPC calls on the stub.
grpc::Status s = stub->sayHello(&context, *request, response);
```

对于高级用例，例如修改根 CA 或者使用客户端证书，可以在 `SslCredentialsOptions` 传递给工厂方法的参数中设置相应的选项。

使用基于 Google 令牌的身份验证

gRPC 应用程序可以使用简单的 API 创建凭据，该凭据可用于在各种部署方案中与 Google 进行身份验证。同样，我们的示例是在 C++ 中，但你可以在我们的示例部分找到其他语言的示例。

```c++
auto creds = grpc::GoogleDefaultCredentials();
// Create a channel, stub and make RPC calls (same as in the previous example)
auto channel = grpc::CreateChannel(server_name, creds);
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
grpc::Status s = stub->sayHello(&context, *request, response);
```

这个通道凭据对象适用于服务账户里的应用程序以及在 [Google Compute Engine（GCE）中](https://cloud.google.com/compute/)运行的应用程序。在前一种情况下，服务账户的私钥是从环境变量中指定的文件加载的 `GOOGLE_APPLICATION_CREDENTIALS`。秘钥用于生成附加到相应信道上的每个传出 RPC 的承载令牌。

对于在 GCE 中运行的应用程序，可以在 VM 设置期间配置默认服务账户和相应的 OAuth2 范围。在运行时，此凭据处理与身份验证系统的通信以获取 OAuth2 访问令牌，并将它们附加到相应通道上每个传出 RPC。

#### 扩展 gRPC 以支持其他身份验证机制

Credentials 插件 API 允许开发人员插入它们自己的凭据类型。这包括：

- `MetadataCredentialsPlugin` 抽象类，其中包含纯虚 `GetMetadata` 需要由开发者创建的子类来实现的方法。

- `MetadataCredentialsFromPlugin` 函数，它从 `MetadataCredentialsPlugin` 创建了一个 `CallCredentials`。

下面是一个简单的凭证插件的例子，它在自定义头中设置了一个身份验证票据。

```c++
class MyCustomAuthenticator : public grpc::MetadataCredentialsPlugin {
 public:
  MyCustomAuthenticator(const grpc::string& ticket) : ticket_(ticket) {}

  grpc::Status GetMetadata(
      grpc::string_ref service_url, grpc::string_ref method_name,
      const grpc::AuthContext& channel_auth_context,
      std::multimap<grpc::string, grpc::string>* metadata) override {
    metadata->insert(std::make_pair("x-custom-auth-ticket", ticket_));
    return grpc::Status::OK;
  }

 private:
  grpc::string ticket_;
};

auto call_creds = grpc::MetadataCredentialsFromPlugin(
    std::unique_ptr<grpc::MetadataCredentialsPlugin>(
        new MyCustomAuthenticator("super-secret-ticket")));
```

通过在核心级插入 gRPC 凭证实现，可以实现更深入的集成。gRPC 内部还允许使用其他加密机制切换 SSL/TLS。

### 例子

这些认证机制将在所有 gRPC 支持的语言中可用。下面几节将演示上述身份验证和授权特性如何出现在每种语言中:很快就会有更多的语言出现。

#### Go

基本情况 - 没有加密或者身份验证

Client:

```go
conn, _ := grpc.Dial("localhost:50051", grpc.WithInsecure())
// error handling omitted
client := pb.NewGreeterClient(conn)
// ...
```

Server:

```go
s := grpc.NewServer()
lis, _ := net.Listen("tcp", "localhost:50051")
// error handling omitted
s.Serve(lis)
```

使用服务器身份验证 SSL/TLS

Client:

```go
creds, _ := credentials.NewClientTLSFromFile(certFile, "")
conn, _ := grpc.Dial("localhost:50051", grpc.WithTransportCredentials(creds))
// error handling omitted
client := pb.NewGreeterClient(conn)
// ...
```

Server:

```go
creds, _ := credentials.NewServerTLSFromFile(certFile, keyFile)
s := grpc.NewServer(grpc.Creds(creds))
lis, _ := net.Listen("tcp", "localhost:50051")
// error handling omitted
s.Serve(lis)
```

通过 Google 验证

```go
pool, _ := x509.SystemCertPool()
// error handling omitted
creds := credentials.NewClientTLSFromCert(pool, "")
perRPC, _ := oauth.NewServiceAccountFromFile("service-account.json", scope)
conn, _ := grpc.Dial(
	"greeter.googleapis.com",
	grpc.WithTransportCredentials(creds),
	grpc.WithPerRPCCredentials(perRPC),
)
// error handling omitted
client := pb.NewGreeterClient(conn)
// ...
```

#### Ruby

基本情况 - 没有加密或者身份验证

```ruby
stub = Helloworld::Greeter::Stub.new('localhost:50051', :this_channel_is_insecure)
...
```

使用服务器身份验证 SSL/TLS

```ruby
creds = GRPC::Core::Credentials.new(load_certs)  # load_certs typically loads a CA roots file
stub = Helloworld::Greeter::Stub.new('myservice.example.com', creds)
```

通过 Google 验证

```ruby
require 'googleauth'  # from http://www.rubydoc.info/gems/googleauth/0.1.0
...
ssl_creds = GRPC::Core::ChannelCredentials.new(load_certs)  # load_certs typically loads a CA roots file
authentication = Google::Auth.get_application_default()
call_creds = GRPC::Core::CallCredentials.new(authentication.updater_proc)
combined_creds = ssl_creds.compose(call_creds)
stub = Helloworld::Greeter::Stub.new('greeter.googleapis.com', combined_creds)
```

#### C++

基本情况 - 没有加密或者身份验证

```c++
auto channel = grpc::CreateChannel("localhost:50051", InsecureChannelCredentials());
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
...
```

使用服务器身份验证 SSL/TLS

```c++
auto channel_creds = grpc::SslCredentials(grpc::SslCredentialsOptions());
auto channel = grpc::CreateChannel("myservice.example.com", channel_creds);
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
...
```

通过 Google 验证

```c++
auto creds = grpc::GoogleDefaultCredentials();
auto channel = grpc::CreateChannel("greeter.googleapis.com", creds);
std::unique_ptr<Greeter::Stub> stub(Greeter::NewStub(channel));
...
```

#### C#

基本情况 - 没有加密或者身份验证

```c#
var channel = new Channel("localhost:50051", ChannelCredentials.Insecure);
var client = new Greeter.GreeterClient(channel);
...
```

使用服务器身份验证 SSL/TLS

```c#
var channelCredentials = new SslCredentials(File.ReadAllText("roots.pem"));  // Load a custom roots file.
var channel = new Channel("myservice.example.com", channelCredentials);
var client = new Greeter.GreeterClient(channel);

```

通过 Google 验证

```c#
using Grpc.Auth;  // from Grpc.Auth NuGet package
...
// Loads Google Application Default Credentials with publicly trusted roots.
var channelCredentials = await GoogleGrpcCredentials.GetApplicationDefaultAsync();

var channel = new Channel("greeter.googleapis.com", channelCredentials);
var client = new Greeter.GreeterClient(channel);
...
```

验证单个 RPC 调用

```c#
var channel = new Channel("greeter.googleapis.com", new SslCredentials());  // Use publicly trusted roots.
var client = new Greeter.GreeterClient(channel);
...
var googleCredential = await GoogleCredential.GetApplicationDefaultAsync();
var result = client.SayHello(request, new CallOptions(credentials: googleCredential.ToCallCredentials()));
...
```

#### Python

基本情况 - 没有加密或者身份验证

```python
import grpc
import helloworld_pb2

channel = grpc.insecure_channel('localhost:50051')
stub = helloworld_pb2.GreeterStub(channel)
```

使用服务器身份验证 SSL/TLS

Client:

```python
import grpc
import helloworld_pb2

with open('roots.pem', 'rb') as f:
    creds = grpc.ssl_channel_credentials(f.read())
channel = grpc.secure_channel('myservice.example.com:443', creds)
stub = helloworld_pb2.GreeterStub(channel)
```

Server:

```python
import grpc
import helloworld_pb2
from concurrent import futures

server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
with open('key.pem', 'rb') as f:
    private_key = f.read()
with open('chain.pem', 'rb') as f:
    certificate_chain = f.read()
server_credentials = grpc.ssl_server_credentials( ( (private_key, certificate_chain), ) )
# Adding GreeterServicer to server omitted
server.add_secure_port('myservice.example.com:443', server_credentials)
server.start()
# Server sleep omitted
```

使用 JWT 与 Google 进行身份验证

```python
import grpc
import helloworld_pb2

from google import auth as google_auth
from google.auth import jwt as google_auth_jwt
from google.auth.transport import grpc as google_auth_transport_grpc

credentials, _ = google_auth.default()
jwt_creds = google_auth_jwt.OnDemandCredentials.from_signing_credentials(
    credentials)
channel = google_auth_transport_grpc.secure_authorized_channel(
    jwt_creds, None, 'greeter.googleapis.com:443')
stub = helloworld_pb2.GreeterStub(channel)
```

使用 Oauth2 令牌通过 Google 进行身份验证

```python
import grpc
import helloworld_pb2

from google import auth as google_auth
from google.auth.transport import grpc as google_auth_transport_grpc
from google.auth.transport import requests as google_auth_transport_requests

credentials, _ = google_auth.default(scopes=(scope,))
request = google_auth_transport_requests.Request()
channel = google_auth_transport_grpc.secure_authorized_channel(
    credentials, request, 'greeter.googleapis.com:443')
stub = helloworld_pb2.GreeterStub(channel)
```

#### Java

基本情况 - 没有加密或者身份验证

```java
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 50051)
    .usePlaintext(true)
    .build();
GreeterGrpc.GreeterStub stub = GreeterGrpc.newStub(channel);
```

使用服务器身份验证 SSL/TLS

在 Java 中，我们建议你在 TLS 上使用 gRPC 时使用 OpenSSL。您可以在 gRPC Java [安全文档](https://github.com/grpc/grpc-java/blob/master/SECURITY.md#transport-security-tls)中找到关于安装和使用 OpenSSL 以及 Android 和非 Android Java 所需的其他库的详细信息。

要在服务器上启用 TLS，需要以 PEM 格式指定证书链和私钥。这样的私钥不应该使用密码。链中的证书顺序很重要：更具体地说，顶部的证书必须是主机 CA，而最底部的证书必须是根 CA. 标准 TLS 端口是 443，但我们使用下面的8443 以避免需要操作系统的额外权限。

```java
Server server = ServerBuilder.forPort(8443)
    // Enable TLS
    .useTransportSecurity(certChainFile, privateKeyFile)
    .addService(TestServiceGrpc.bindService(serviceImplementation))
    .build();
server.start();
```

如果客户端不知道颁发证书的权限，则应分别正确配置 `SslContext` 或 `SSLSocketFactory` 提供给  `NettyChannelBuilder` 或 `OkHttpChannelBuilder`。

在客户端，使用 SSL/TLS 的服务器身份验证如下所示：

```java
// With server authentication SSL/TLS
ManagedChannel channel = ManagedChannelBuilder.forAddress("myservice.example.com", 443)
    .build();
GreeterGrpc.GreeterStub stub = GreeterGrpc.newStub(channel);

// With server authentication SSL/TLS; custom CA root certificates; not on Android
ManagedChannel channel = NettyChannelBuilder.forAddress("myservice.example.com", 443)
    .sslContext(GrpcSslContexts.forClient().trustManager(new File("roots.pem")).build())
    .build();
GreeterGrpc.GreeterStub stub = GreeterGrpc.newStub(channel);
```

通过 Google 验证

以下代码段显示了如何使用带有服务帐户的 gRPC 调用 [Google Cloud PubSub API](https://cloud.google.com/pubsub/docs/overview)。凭据从存储在众所周知的位置的密钥加载，或者通过检测应用程序在可以自动提供应用程序的环境中运行，例如 Google Compute Engine。虽然此示例特定于 Google 及其服务，但其他服务提供商可以遵循类似的模式。

```java
GoogleCredentials creds = GoogleCredentials.getApplicationDefault();
ManagedChannel channel = ManagedChannelBuilder.forTarget("greeter.googleapis.com")
    .build();
GreeterGrpc.GreeterStub stub = GreeterGrpc.newStub(channel)
    .withCallCredentials(MoreCallCredentials.from(creds));
```

#### Node.js

基本情况 - 没有加密或者身份验证

```javascript
var stub = new helloworld.Greeter('localhost:50051', grpc.credentials.createInsecure());
```

使用服务器身份验证 SSL/TLS

```javascript
var ssl_creds = grpc.credentials.createSsl(root_certs);
var stub = new helloworld.Greeter('myservice.example.com', ssl_creds);
```

通过 Google 验证

```javascript
// Authenticating with Google
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.credentials.createSsl(root_certs);
(new GoogleAuth()).getApplicationDefault(function(err, auth) {
  var call_creds = grpc.credentials.createFromGoogleCredential(auth);
  var combined_creds = grpc.credentials.combineChannelCredentials(ssl_creds, call_creds);
  var stub = new helloworld.Greeter('greeter.googleapis.com', combined_credentials);
});
```

使用 Oauth2 令牌使用 Google 进行身份验证（传统方法）

```javascript
var GoogleAuth = require('google-auth-library'); // from https://www.npmjs.com/package/google-auth-library
...
var ssl_creds = grpc.Credentials.createSsl(root_certs); // load_certs typically loads a CA roots file
var scope = 'https://www.googleapis.com/auth/grpc-testing';
(new GoogleAuth()).getApplicationDefault(function(err, auth) {
  if (auth.createScopeRequired()) {
    auth = auth.createScoped(scope);
  }
  var call_creds = grpc.credentials.createFromGoogleCredential(auth);
  var combined_creds = grpc.credentials.combineChannelCredentials(ssl_creds, call_creds);
  var stub = new helloworld.Greeter('greeter.googleapis.com', combined_credentials);
});

```

#### PHP

基本情况 - 没有加密或者身份验证

```php
$client = new helloworld\GreeterClient('localhost:50051', [
    'credentials' => Grpc\ChannelCredentials::createInsecure(),
]);
...
```

通过 Google 验证

```php
function updateAuthMetadataCallback($context)
{
    $auth_credentials = ApplicationDefaultCredentials::getCredentials();
    return $auth_credentials->updateMetadata($metadata = [], $context->service_url);
}
$channel_credentials = Grpc\ChannelCredentials::createComposite(
    Grpc\ChannelCredentials::createSsl(file_get_contents('roots.pem')),
    Grpc\CallCredentials::createFromPlugin('updateAuthMetadataCallback')
);
$opts = [
  'credentials' => $channel_credentials
];
$client = new helloworld\GreeterClient('greeter.googleapis.com', $opts);
```

使用 Oauth2 令牌使用 Google 进行身份验证（传统方法）

```php
// the environment variable "GOOGLE_APPLICATION_CREDENTIALS" needs to be set
$scope = "https://www.googleapis.com/auth/grpc-testing";
$auth = Google\Auth\ApplicationDefaultCredentials::getCredentials($scope);
$opts = [
  'credentials' => Grpc\Credentials::createSsl(file_get_contents('roots.pem'));
  'update_metadata' => $auth->getUpdateMetadataFunc(),
];
$client = new helloworld\GreeterClient('greeter.googleapis.com', $opts);
```

#### Dart

基本情况 - 没有加密或者身份验证

```dart
final channel = new ClientChannel('localhost',
      port: 50051,
      options: const ChannelOptions(
          credentials: const ChannelCredentials.insecure()));
final stub = new GreeterClient(channel);
```

使用服务器身份验证 SSL/TLS

```dart
// Load a custom roots file.
final trustedRoot = new File('roots.pem').readAsBytesSync();
final channelCredentials =
    new ChannelCredentials.secure(certificates: trustedRoot);
final channelOptions = new ChannelOptions(credentials: channelCredentials);
final channel = new ClientChannel('myservice.example.com',
    options: channelOptions);
final client = new GreeterClient(channel);
```

通过 Google 验证

```dart
// Uses publicly trusted roots by default.
final channel = new ClientChannel('greeter.googleapis.com');
final serviceAccountJson =
     new File('service-account.json').readAsStringSync();
final credentials = new JwtServiceAccountAuthenticator(serviceAccountJson);
final client =
    new GreeterClient(channel, options: credentials.toCallOptions);
```

验证单个 RPC 调用

```dart
// Uses publicly trusted roots by default.
final channel = new ClientChannel('greeter.googleapis.com');
final client = new GreeterClient(channel);
...
final serviceAccountJson =
     new File('service-account.json').readAsStringSync();
final credentials = new JwtServiceAccountAuthenticator(serviceAccountJson);
final response =
    await client.sayHello(request, options: credentials.toCallOptions);
```

## 错误处理和调试

### 错误处理

此页面描述了gRPC如何处理错误，包括gRPC的内置错误代码。可以在[此处](https://github.com/avinassh/grpc-errors)找到不同语言的示例代码。

### 标准错误模型

正如您在我们的概念文档和示例中所看到的，当 gRPC 调用成功完成时，服务器会向客户端返回一个 OK 状态（取决于语言，OK 可能会或可能不会直接在您的代码中使用）。但如果调用不成功会怎样？

如果发生错误，gRPC 会返回其错误状态代码之一，并带有可选的字符串错误消息，该消息提供有关所发生情况的更多详细信息。所有支持的语言中的 gRPC 客户端都可以使用错误信息。

### 更丰富的错误模型

上述错误模型是官方 gRPC 错误模型，受所有 gRPC 客户端/服务器库支持，并且独立于 gRPC 数据格式（无论是 protocol buffers 或者其他内容）。你可能已经注意到它非常有限，并且不包括传达错误详细信息的能力。

如果你在使用 protoco buffers 的数据格式，你不妨考虑使用开发和这里所描述的由谷歌所使用的更丰富的[错误模型](https://cloud.google.com/apis/design/errors#error_model)。这个模型使服务器返回并且客户端能够使用表示为一个或多个 protobuf 消息的错误详细信息。它进一步制定了一组[标准的错误消息类型](https://github.com/googleapis/googleapis/blob/master/google/rpc/error_details.proto)，以满足最常见的需求（例如无效参数，配额违规和堆栈跟踪）。此额外错误信息的 protobuf 二进制编码在响应中作为尾随元数据提供。

这个更丰富的错误模型已经在 C++，Go，Java，Python 和 Ruby 库中得到支持，并且至少 grpc-web 和 Nodes.js 库存在请求支持它的 issue。如果有需求，其他语言库可能会在将来添加支持，因此如果感兴趣，请检查他们的 github 存储库。但请注意，用 C 语言编写的 grpc-core 库不太可能支持它，因为它是有目的的数据格式不可知的。

如果你没有使用 protocol buffers，你可以使用类似的方法（在尾随相应元数据中放置错误详细信息），但你可能需要查找或开发用于访问此数据的库支持，以便在你的实际 API 中使用它。

在决定是否使用这种扩展错误模型时，需要注意一些重要的注意事项，包括：

- 在错误细节有效载荷的要求和期望方面，扩展错误模型的库实现可能在语言之间不一致
- 现有代理，记录器和其他标准 HTTP 请求处理器无法查看错误详细信息，因此无法将其用于监视或其他目的
- 追踪者中的其他错误详细信息会干扰线头阻塞，并且由于更频繁的缓存未命中而会降低 HTTP/2 报头压缩效率
- 较大的错误细节有效负载可能会遇到协议限制（如最大 header 大小），从而有效失去原始错误

### 错误状态代码

gRPC 在各种情况下引发错误，从网络故障到未经认证的连接，每个连接都与特定的状态代码相关联。所有 gRPC 语言都支持以下错误状态代码。

#### 一般错误

| 案件 | 状态代码|
| :--- | :-----|
|客户端应用程序取消请求|GRPC_STATUS_CANCELLED|
|截止日期在服务器返回状态之前到期|GRPC_STATUS_DEADLINE_EXCEEDED|
|在服务器上找不到的方法|GRPC_STATUS_UNIMPLEMENTED|
|服务器关闭|GRPC_STATUS_UNAVAILABLE|
|服务器抛出异常（或者做了除了返回状态代码以终止RPC之外的其他操作）|GRPC_STATUS_UNKNOWN|

#### 网络故障

| 案件 | 状态代码|
| :--- | :-----|
|在截止日期到期之前没有传输数据。也适用于在截止日期到期之前传输某些数据且未检测到其他故障的情况|GRPC_STATUS_DEADLINE_EXCEEDED|
|在连接中断之前传输了一些数据（例如，请求元数据已写入TCP连接）|GRPC_STATUS_UNAVAILABLE|

#### 协议错误

| 案件 | 状态代码|
| :--- | :-----|
|无法解压缩但支持压缩算法|GRPC_STATUS_INTERNAL|
|客户端使用的压缩机制不受服务器支持|GRPC_STATUS_UNIMPLEMENTED|
|达到流量控制资源限制|GRPC_STATUS_RESOURCE_EXHAUSTED|
|流量控制协议违规|GRPC_STATUS_INTERNAL|
|解析返回状态时出错|GRPC_STATUS_UNKNOWN|
|未经身份验证：凭据无法获取元数据|GRPC_STATUS_UNAUTHENTICATED|
|权限元数据中的主机集无效|GRPC_STATUS_UNAUTHENTICATED|
|解析响应协议缓冲区时出错|GRPC_STATUS_INTERNAL|
|解析请求协议缓冲区时出错|GRPC_STATUS_INTERNAL|

## 性能

gRPC 旨在支持多种语言的高性能开源RPC。本文档介绍了性能基准测试工具，测试所考虑的方案以及测试基础架构。

### 概览

gRPC 专为分布式应用的高性能和高生产率设计而设计。持续性能基准测试是 gRPC 开发工作流程的关键部分。针对主分支每小时运行多语言性能测试，并将这些数字报告给仪表板以进行可视化。

- [多语言性能仪表板 @latest_release（最新可能用稳定版）](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5636470266134528)
- [多语言性能仪表板@master（最新开发版）](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5652536396611584)
- [C ++详细性能仪表板@master（最新开发版）](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5685265389584384)

额外的性能测试可以提供有关 CPU 使用情况的细粒度洞察。

- [C ++全栈微基准测试](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5684961520648192)
- [C核心过滤器基准测试](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5740240702537728)
- [C Core共享组件基准测试](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5641826627223552&container=789696829&widget=512792852)
- [C Core HTTP / 2微基准测试](https://performance-dot-grpc-testing.appspot.com/explore?dashboard=5732910535540736)

### 性能测试设计

每种语言都实现了一个实现 gRPC [WorkerService](https://github.com/grpc/grpc/blob/master/src/proto/grpc/testing/worker_service.proto) 的性能测试工作者 。此服务指示工作人员充当实际基准测试的客户端或服务器，表示为 [BenchmarkService](https://github.com/grpc/grpc/blob/master/src/proto/grpc/testing/benchmark_service.proto)。该服务有两种方法：

- UnaryCall - 一个简单请求的一元RPC，它指定在响应中返回的字节数
- StreamingCall - 一种流式RPC，允许重复的 ping-pongs 请求和响应消息类似于 UnaryCall

![image](https://grpc.io/img/testing_framework.png)

这些工作程序由[驱动程序](https://github.com/grpc/grpc/blob/master/test/cpp/qps/qps_json_driver.cc)控制，该驱动程序将该方案描述（采用 JSON 格式）和指定每个工作进程的 hots:port 的环境变量作为输入。

### 正在测试的语言

以下语言作为 master 上的客户端和服务器进行连续性能测试：

- C++
- Java
- Go
- C#
- node.js
- Python
- Ruby

此外，从 C core 派生的所有语言都在每次拉取请求时进行了有限的性能测试（冒烟测试）。

除了作为性能测试的客户端和服务器端运行之外，所有语言都作为针对 C++ 服务器的客户端进行测试，并作为针对 C++ 客户端的服务器进行测试。此测试旨在为给定语言的客户端或服务器实现提供当前的性能上限，而无需测试另一方。

虽然 PHP 或移动环境不支持 gRPC 服务器（我们的性能测试需要），但可以使用另一种语言编写的代理 WorkerService 对其客户端性能进行基准测试。此代码是为 PHP 实现的，但尚未处于连续测试模式。

### 正在测试的场景

有几个重要的方案正在测试中并显示在上面的仪表板中，包括以下内容：

- 无争用延迟 - 只有 1 个客户端使用 StreamingCall 一次发送一条消息时看到的中位数和尾部​​响应延迟
- QPS - 当有 2 个客户端和总共 64 个通道时的消息/秒速率，每个通道使用 StreamingCall 一次发送 100 个未完成的消息
- 可伸缩性（适用于所选语言） - 每个服务器核心的消息数/秒

大多数性能测试都使用安全通信和 protobufs。一些 C++ 测试还使用不安全的通信和通用（非 protobuf）API 来显示峰值性能。将来可能会添加其他方案。

### 测试基础架构

所有性能基准测试都通过我们的 Jenkins 测试基础架构作为 GCE 中的实例运行。除了上面描述的 gRPC 性能方案之外，我们还运行基线 [netperf TCP_RR](http://www.netperf.org/) 延迟数，以便了解底层网络特征。这些数字出现在我们的仪表板上，有时会根据我们的实例在 GCE 中的分配位置而有所不同。

大多数测试实例都是 8 核系统，这些系统用于延迟和 QPS 测量。对于 C++ 和 Java，我们还支持在 32 核系统上进行 QPS 测试。所有 QPS 测试都为每台服务器使用 2 台相同的客户端计算机，以确保 QPS 测量不受客户端限制。
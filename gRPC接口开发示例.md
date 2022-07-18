#### gRPC接口开发示例

示例代码结构如下

```shell
$ tree
├── client
│   └── main.go
├── helloworld
│   ├── helloworld.pb.go
│   └── helloworld.proto
└── server
    └── main.go
```

client 目录存放 Client 端的代码，helloworld 目录用来存放服务的 IDL 定义，server 目录用来存放 Server 端的代码。

##### 1. 定义gRPC服务

首先，需要定义我们的服务。进入 helloworld 目录，新建文件 helloworld.proto：

```shell
$ cd helloworld
$ vi helloworld.proto
```

内容如下

```protobuf
syntax = "proto3";

option go_package = "github.com/marmotedu/gopractise-demo/apistyle/greeter/helloworld";

package helloworld;

// The greeting service definition.
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

在 helloworld.proto 定义文件中，option 关键字用来对.proto 文件进行一些设置，其中 go_package 是必需的设置，而且 go_package 的值必须是包导入的路径。package 关键字指定生成的.pb.go 文件所在的包名。我们通过 service 关键字定义服务，然后再指定该服务拥有的 RPC 方法，并定义方法的请求和返回的结构体类型：

```protobuf
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
```

gRPC 支持定义 4 种类型的服务方法，分别是简单模式、服务端数据流模式、客户端数据流模式和双向数据流模式。

- 简单模式（Simple RPC）：是最简单的 gRPC 模式。客户端发起一次请求，服务端响应一个数据。定义格式为 rpc SayHello (HelloRequest) returns (HelloReply) {}。
- 服务端数据流模式（Server-side streaming RPC）：客户端发送一个请求，服务器返回数据流响应，客户端从流中读取数据直到为空。定义格式为 rpc SayHello (HelloRequest) returns (stream HelloReply) {}。
- 客户端数据流模式（Client-side streaming RPC）：客户端将消息以流的方式发送给服务器，服务器全部处理完成之后返回一次响应。定义格式为 rpc SayHello (stream HelloRequest) returns (HelloReply) {}。
- 双向数据流模式（Bidirectional streaming RPC）：客户端和服务端都可以向对方发送数据流，这个时候双方的数据可以同时互相发送，也就是可以实现实时交互 RPC 框架原理。定义格式为 rpc SayHello (stream HelloRequest) returns (stream HelloReply) {}。

##### 2. 生成客户端和服务器代码

接下来，我们需要根据.proto 服务定义生成 gRPC 客户端和服务器接口。我们可以使用 protoc 编译工具，并指定使用其 Go 语言插件来生成：

```shell
$ protoc -I. --go_out=plugins=grpc:$GOPATH/src helloworld.proto
$ ls
helloworld.pb.go  helloworld.proto
```

你可以看到，新增了一个 helloworld.pb.go 文件。

##### 3. 实现gRPC服务

接着，我们就可以实现 gRPC 服务了。进入 server 目录，新建 main.go 文件：

```shell
$ cd ../server
$ vi main.go
```

Main.go 内容如下

```go
// Package main implements a server for Greeter service.
package main

import (
  "context"
  "log"
  "net"

  pb "github.com/marmotedu/gopractise-demo/apistyle/greeter/helloworld"
  "google.golang.org/grpc"
)

const (
  port = ":50051"
)

// server is used to implement helloworld.GreeterServer.
type server struct {
  pb.UnimplementedGreeterServer
}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
  log.Printf("Received: %v", in.GetName())
  return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func main() {
  lis, err := net.Listen("tcp", port)
  if err != nil {
    log.Fatalf("failed to listen: %v", err)
  }
  s := grpc.NewServer()
  pb.RegisterGreeterServer(s, &server{})
  if err := s.Serve(lis); err != nil {
    log.Fatalf("failed to serve: %v", err)
  }
}
```

上面的代码实现了我们上一步根据服务定义生成的 Go 接口。

我们先定义了一个 Go 结构体 server，并为 server 结构体添加SayHello(context.Context, pb.HelloRequest) (pb.HelloReply, error)方法，也就是说 server 是 GreeterServer 接口（位于 helloworld.pb.go 文件中）的一个实现。

在我们实现了 gRPC 服务所定义的方法之后，就可以通过 net.Listen(...) 指定监听客户端请求的端口；接着，通过 grpc.NewServer() 创建一个 gRPC Server 实例，并通过 pb.RegisterGreeterServer(s, &server{}) 将该服务注册到 gRPC 框架中；最后，通过 s.Serve(lis) 启动 gRPC 服务。

创建完 main.go 文件后，在当前目录下执行 go run main.go ，启动 gRPC 服务。

##### 4. 实现gRPC客户端

打开一个新的 Linux 终端，进入 client 目录，新建 main.go 文件：

```shell
$ cd ../client
$ vi main.go
```

main.go 内容如下：

```go
// Package main implements a client for Greeter service.
package main

import (
  "context"
  "log"
  "os"
  "time"

  pb "github.com/marmotedu/gopractise-demo/apistyle/greeter/helloworld"
  "google.golang.org/grpc"
)

const (
  address     = "localhost:50051"
  defaultName = "world"
)

func main() {
  // Set up a connection to the server.
  conn, err := grpc.Dial(address, grpc.WithInsecure(), grpc.WithBlock())
  if err != nil {
    log.Fatalf("did not connect: %v", err)
  }
  defer conn.Close()
  c := pb.NewGreeterClient(conn)

  // Contact the server and print out its response.
  name := defaultName
  if len(os.Args) > 1 {
    name = os.Args[1]
  }
  ctx, cancel := context.WithTimeout(context.Background(), time.Second)
  defer cancel()
  r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
  if err != nil {
    log.Fatalf("could not greet: %v", err)
  }
  log.Printf("Greeting: %s", r.Message)
}
```

在上面的代码中，我们通过如下代码创建了一个 gRPC 连接，用来跟服务端进行通信：

```go
// Set up a connection to the server.
conn, err := grpc.Dial(address, grpc.WithInsecure(), grpc.WithBlock())
if err != nil {
    log.Fatalf("did not connect: %v", err)
}
defer conn.Close()
```

在创建连接时，我们可以指定不同的选项，用来控制创建连接的方式，例如 grpc.WithInsecure()、grpc.WithBlock() 等。gRPC 支持很多选项，更多的选项可以参考 grpc 仓库下dialoptions.go文件中以 With 开头的函数。

连接建立起来之后，我们需要创建一个客户端 stub，用来执行 RPC 请求c := pb.NewGreeterClient(conn)。创建完成之后，我们就可以像调用本地函数一样，调用远程的方法了。例如，下面一段代码通过 c.SayHello 这种本地式调用方式调用了远端的 SayHello 接口：

```go
r, err := c.SayHello(ctx, &pb.HelloRequest{Name: name})
if err != nil {
    log.Fatalf("could not greet: %v", err)
}
log.Printf("Greeting: %s", r.Message)
```

调用方便：RPC 屏蔽了底层的网络通信细节，使得调用 RPC 就像调用本地方法一样方便，调用方式跟大家所熟知的调用类的方法一致：ClassName.ClassFuc(params)。不需要打包和解包：RPC 调用的入参和返回的结果都是 Go 的结构体，不需要对传入参数进行打包操作，也不需要对返回参数进行解包操作，简化了调用步骤。

最后，创建完 main.go 文件后，在当前目录下，执行 go run main.go 发起 RPC 调用：

```shell
$ go run main.go
2020/10/17 07:55:00 Greeting: Hello world
```

至此，我们用四个步骤，创建并调用了一个 gRPC 服务。接下来我再给大家讲解一个在具体场景中的注意事项。

#### 通过参数决定接口行为

在做服务开发时，我们经常会遇到一种场景：定义一个接口，接口会通过判断是否传入某个参数，决定接口行为。例如，我们想提供一个 GetUser 接口，期望 GetUser 接口在传入 username 参数时，根据 username 查询用户的信息，如果没有传入 username，则默认根据 userId 查询用户信息。

这时候，我们需要判断客户端有没有传入 username 参数。**我们不能根据 username 是否为空值来判断，因为我们不能区分客户端传的是空值，还是没有传 username 参数。这是由 Go 语言的语法特性决定的：如果客户端没有传入 username 参数，Go 会默认赋值为所在类型的零值，而字符串类型的零值就是空字符串**。

那我们怎么判断客户端有没有传入 username 参数呢？**最好的方法是通过指针来判断，如果是 nil 指针就说明没有传入，非 nil 指针就说明传入**，具体实现步骤如下：

##### 1. 编写protobuf定义文件

新建 user.proto 文件，内容如下:

```go
syntax = "proto3";

package proto;
option go_package = "github.com/marmotedu/gopractise-demo/protobuf/user";

//go:generate protoc -I. --experimental_allow_proto3_optional --go_out=plugins=grpc:.

service User {
  rpc GetUser(GetUserRequest) returns (GetUserResponse) {}
}

message GetUserRequest {
  string class = 1;
  optional string username = 2;
  optional string user_id = 3;
}

message GetUserResponse {
  string class = 1;
  string user_id = 2;
  string username = 3;
  string address = 4;
  string sex = 5;
  string phone = 6;
}
```

你需要注意，这里我们在需要设置为可选字段的前面添加了 optional 标识。

##### 2. 使用protoc工具编译protobuf文件

在执行 protoc 命令时，需要传入--experimental_allow_proto3_optional参数以打开 optional 选项，编译命令如下：

```shell
$ protoc --experimental_allow_proto3_optional --go_out=plugins=grpc:. user.proto
```

上述编译命令会生成 user.pb.go 文件，其中的 GetUserRequest 结构体定义如下：

```go
type GetUserRequest struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields

    Class    string  `protobuf:"bytes,1,opt,name=class,proto3" json:"class,omitempty"`
    Username *string `protobuf:"bytes,2,opt,name=username,proto3,oneof" json:"username,omitempty"`
    UserId   *string `protobuf:"bytes,3,opt,name=user_id,json=userId,proto3,oneof" json:"user_id,omitempty"`
}
```

通过 optional + --experimental_allow_proto3_optional 组合，我们可以将一个字段编译为指针类型。

##### 3. 编写gRPC接口实现

新建一个user.go文件，内容如下

```go
package user

import (
    "context"

    pb "github.com/marmotedu/api/proto/apiserver/v1"

    "github.com/marmotedu/iam/internal/apiserver/store"
)

type User struct {
}

func (c *User) GetUser(ctx context.Context, r *pb.GetUserRequest) (*pb.GetUserResponse, error) {
    if r.Username != nil {
        return store.Client().Users().GetUserByName(r.Class, r.Username)
    }

    return store.Client().Users().GetUserByID(r.Class, r.UserId)
}
```

总之，在 GetUser 方法中，我们可以通过判断 r.Username 是否为 nil，来判断客户端是否传入了 Username 参数。

如果有gRPC服务，并且希望该服务同时也能提供RESTful API接口，可以使用[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)
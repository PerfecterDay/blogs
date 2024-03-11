## go 与 GRPC 集成
{docsify-updated}

- [go 与 GRPC 集成](#go-与-grpc-集成)
  - [环境配置](#环境配置)
  - [编译](#编译)
  - [代码](#代码)

### 环境配置

1. 安装 Protocol Buffer Compiler： `brew install protobuf`
2. 安装 Go plugins for the protocol compiler:
   ```
    $ go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
    $ go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
   ```
3. 更新 PATH，以便 protoc 编译器能找到插件: `export PATH="$PATH:$(go env GOPATH)/bin"`


### 编译
```
protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    helloworld/helloworld.proto
```

### 代码
```
package main

import (
	"flag"
	"fmt"
	pb "gjywgitlab.gtja.net/gtja-app-platform/go-trade-gmt/grpc_client"
	"google.golang.org/grpc"
	"google.golang.org/grpc/reflection"
	"log"
	"net"
)

var (
	port = flag.Int("port", 50051, "The server port")
)

// server is used to implement helloworld.GreeterServer.
type server struct {
	pb.UnimplementedGreeterServer
}

func main() {
	//cmd.Execute()

	lis, err := net.Listen("tcp", fmt.Sprintf(":%d", *port))
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	reflection.Register(s)
	pb.RegisterGreeterServer(s, &server{})
	log.Printf("server listening at %v", lis.Addr())
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```
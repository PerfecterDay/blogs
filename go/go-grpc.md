## go 与 GRPC 集成
{docsify-updated}

- [go 与 GRPC 集成](#go-与-grpc-集成)
  - [环境配置](#环境配置)
  - [编译](#编译)

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

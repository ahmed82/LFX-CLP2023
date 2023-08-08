# gRPC
* RPC (Remote Procedure Call Framework) by Google
* Client can execute a remote procedure on the server
* The remote interaction code is handled by gRPC
* APIs and data structure are generated automatically by Protocol Buffer Compiler (protoc)
* Multiple languages supported 

## Steps
1. Define API and data structure ie RPC and request/response structure using protobuf
    ![Alt text](image.png)
2. Generate gRPC stubs for server and client (can be generated in multiple languages):

    ![Alt text](image-1.png)
    ![Alt text](image-2.png)
3. Implement RPC handler on server side
4. Use  generated client stubs and call RPC on the server

## Benefits 
* High performance thanks to HTTP2
* Strong API contract : protobuf RPC definition is strongly typed
* Automatic code generation

## Types of gRPC
1. Unary gRPC : Client sends single request, server responds with single response
2. Client Streaming gRPC : Client sends stream of multiple messages, server sends single response
3. Server streaming gRPC :  Client sends single request, server sends stream of multiple messages in response
4. Bidirectional streaming gRPC :  both client and server keep sending streams of multiple messages
![Alt text](image-3.png)

## gRPC Gateway
A protobuf plugin,which allows us to serve both gRPC and HTTP requests at once
* Transalates HTTP JSON to gRPC
    * In process transaltion :  only for unary
    * Separate Gateway proxy server : both unary and streaming
![Alt text](image-4.png)


## 1. Define gRPC API and generate Go code with protobuf
### Setup
1. Install protocol buffer compiler (protobuf)
    `apt install -y protobuf-compiler`
2. Install Go plugins
```
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.28
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.2
export PATH="$PATH:$(go env GOPATH)/bin"
protoc-gen-go --version #check version
```
4. vscode-proto3 extension

3. Define API in user.proto file
4. Define Request, response types - rpc_create_user.proto, rpc_login_user.proto
5. Define rpc service in service_bank.proto
6. Generate gRPC code : 
```
protoc  --proto_path=proto --go_out=pb --go_opt=paths=source_relative \
    --go-grpc_out=pb --go-grpc_opt=paths=source_relative \
    proto/*.proto
```
8. Now we can see generated files in pb directory

## 2. Run gRPC server using generated Go code

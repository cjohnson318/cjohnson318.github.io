---
layout: post
title:  "Use gRPC to Connect a Go/Gin Server to Python"
date: 2024-11-03 00:00:00 -0700
tags: python go
---

I built a proof of concept to use the Go Gin framework to serve HTTP requests,
and reach out to a Python gRPC server for the results of more specialized data
analysis tasks. I had plans to build a minimal frontend with TypeScript, and
do more than stub out the Python server, but this is a good place to stop at
the moment.

## Python gRPC Server

This was the easiest part. This server listens for a gRPC request, and then
returns a gRPC response.

{% highlight python %}
import grpc

from concurrent import futures
import hello_pb2
import hello_pb2_grpc

class GreeterServicer(hello_pb2_grpc.GreeterServicer):

    def SayHello(self, request, context):
        return hello_pb2.HelloReply(message=f"Hello {request.name}!")

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    hello_pb2_grpc.add_GreeterServicer_to_server(GreeterServicer(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    server.wait_for_termination()

if __name__ == '__main__':
    serve() 
{% endhighlight %}

The `hello_pb2` and `hello_pb2_grpc` files were generated using the `protoc`
protobuf compiler. I had this Hello World protobuf file, `hello.proto`,

{% highlight console %}
syntax = "proto3";

option go_package = "github.com/cjohnson318/motif-backend/pkg/proto";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
{% endhighlight %}

Then after doing some installations, I was able to use the Python wrapper for
protoc to generate some helper code for my protobuf file.

{% highlight console %}
brew install protobuf
python3 -m venv venv
source venv/bin/activate
pip install grpcio protobuf
python -m grpc_tools.protoc -I./proto --python_out=. --pyi_out=. --grpc_python_out=. hello.proto
{% endhighlight %}

Then I wrapped my Python process in a Dockerfile.

{% highlight docker %}
FROM python:3.11-slim-buster

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "server.py"]
{% endhighlight %}

Then I built and ran my Docker container.

{% highlight console %}
docker build -t motif-grpc-server .
docker run -it -p 50051:50051 motif-grpc-server
{% endhighlight %}

That was the easy part.

## Go/Gin gRPC Client

This part was harder. It all came down to the fact that I didn't have the
gRPC plugin installed for Go. I had installed grpc for my system using brew,
but I hadn't installed the gRPC plugin for the go compiler itself. The error
messages complained about not being able to find this or that, and that should
have been my clue that this and that did not in fact exist. Yolo.

{% highlight console %}
brew install grpc
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
{% endhighlight %}

Once that was resolved, I figured out how to compile my protobuf file.

{% highlight console %}
protoc hello.proto --proto_path=. --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative
{% endhighlight %}

I built a Go/Gin server

{% highlight python %}
go mod init github.com/username/reponame
go get -u github.com/gin-gonic/gin
{% endhighlight %}

And then I popped all of my server code and gRPC code into `main.go`.

{% highlight go %}
package main

import (
    "context"
    "fmt"
    "github.com/gin-gonic/gin"
    "google.golang.org/grpc"
    "net/http"
    pb "github.com/cjohnson318/motif-backend/pkg/proto"

)

func main() {
    fmt.Println("Hello")
    r := gin.Default()

    r.GET("/", func(c *gin.Context) {
        conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
        if err != nil {
            panic(err)
        }
        defer conn.Close()

        client := pb.NewGreeterClient(conn)

        ctx := context.Background()
        resp, err := client.SayHello(ctx, &pb.HelloRequest{Name: "Dork"})
        if err != nil {
            panic(err)
        }

        fmt.Println("Greeting:", resp.Message)
        c.JSON(http.StatusOK, gin.H{"data": resp.Message})    
    })

    r.Run(":50052")
}
{% endhighlight %}

To run this, I used,

{% highlight console %}
go run cmd/main.go
{% endhighlight %}

## Conclusion

So that's my minimal proof of concept. The Go HTTP server communicates with the
Python gRPC server running in Docker on port 50051, and it communicates the result 
back to the user. I have code for the Go server at [https://github.com/cjohnson318/motif-backend](https://github.com/cjohnson318/motif-backend)
and the Python server at [https://github.com/cjohnson318/motif-services](https://github.com/cjohnson318/motif-services).

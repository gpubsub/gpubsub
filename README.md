# gPubSub

An opinionated and simplified way to do API first pub/sub.
Support for Apache Kafka, Google Cloud Pub/Sub and Amazon Kinesis.

Current status: thinking about it...

## Protobuf Source

Service definition

```protobuf
package helloworld;

// The greeting publisher definition.
service Greeter {
  // Takes a `HelloMessage` when publishing
  // Returns a `HelloMessage` when subscribing
  rpc SayHello (HelloMessage) returns (HelloMessage);
}

// The request message containing the user's name.
message HelloMessage {
  string name = 1;
}
```

## GO Example

Generate clients and models:

```bash
$ gpubsub -I helloworld/ helloworld/helloworld.proto --go_out=plugins=gpubsub:helloworld
```

Example implementation:

```golang
package main

import (
    "log"

    "golang.org/x/net/context"
    "github.com/gpubsub/kafka"
    pb "helloworld/helloworld"
)

func main() {
    // Set up a connection to the server.
    conn, err := kafka.Dial("localhost:9092")
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()

    msg := pb.HelloMessage{
        Name: "Neo",
    }

    // Publish Message
    p := pb.NewGreeterPub(conn, "/hello_topic")
    err := p.SayHello(context.Background(), &msg)
    if err != nil {
        log.Fatalf("could not publish message: %v", err)
    }

    // Get Messages
    s := pb.NewGreeterSub(conn, "/hello_topic")

    // Message Processor
    processor := func(*HelloMessage) error {
        log.Printf("Hi: %s", hello.Name)
        return nil
    }

    err := s.SayHello(context.Background(), processor)
    if err != nil {
        log.Fatalf("could not fetch message: %v", err)
    }
}
```

## Generated Code

```golang
package helloworld

type HelloMessage struct {
    Name string `protobuf:"bytes,1,opt,name=name" json:"name,omitempty"`
}

// Publisher
func NewGreeterPub(
    cc *gpubsub.ClientConn,
    topic string,
    opts ...gpubsub.PubConnOption
 ) GreeterPub { ... }

type GreeterPub struct { ... }

func (pub *GreeterPub) SayHello(
    ctx context.Context,
    in *HelloRequest,
    opts ...gpubsub.PubOption
) error { ... }

func (pub *GreeterPub) SayHelloBatch(
    ctx context.Context,
     in []*HelloRequest,
     opts ...gpubsub.PubOption
) error { ... }

// Subscriber
func NewGreeterSub(
    cc *gpubsub.ClientConn,
    topic string,
    opts ...gpubsub.SubConnOption
) GreeterSub { ... }

type GreeterSub struct { ... }

func (sub *GreeterSub) SayHelloSub(
    ctx context.Context,
    processor func(*HelloMessage) error,
    opts ...gpubsub.SubOption,
) error { ...  }

func (sub *GreeterSub) SayHelloSubBatch(
    ctx context.Context,
    processor func([]*HelloMessage) error,
    opts ...gpubsub.SubOption,
) error { ...  }
```

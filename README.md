# gPubSub

An opinionated and simplified way to do API first pub/sub. What gRPC is to REST gPubSub is to message queues.
Support for Apache Kafka, Google Cloud Pub/Sub and Amazon Kinesis.

## Protobuf Source

Service definition

```protobuf
package helloworld;

// The greeting pub definition.
pub GreeterPub {
  // Pub a message
  pub SayHello (HelloMessage) {}
  // Pub a batch messages
  pub SayHelloBatch (repeated HelloMessage) {}
}

// The greeting sub definition.
sub GreeterSub {
  // Sub a message
  sub GetHello () returns (HelloMessage) {}
  // Sub a batch of messages
  sub GetHellos () returns (repeated HelloMessage) {}
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
	"github.com/gpubsub/gpubsub"
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

    // Publish Message
    p := pb.NewGreeterPub(conn, "/hello_topic")
    err := p.SayHello(context.Background(), &pb.HelloMessage{Name: "Neo"})
    if err != nil {
        log.Fatalf("could not publish message: %v", err)
    }

    // Get to Messages
    s := pb.NewGreeterSub(conn, "/hello_topic")
    hello, err := s.GetHello(context.Background())
    if err != nil {
        log.Fatalf("could not publish message: %v", err)
    }
    log.Printf("Hi: %s", hello.Name)
}
```

## Generated Code

```golang
package helloworld

type HelloMessage struct {
    Name string `protobuf:"bytes,1,opt,name=name" json:"name,omitempty"`
}

type GreeterPub interface {
    SayHello(ctx context.Context, in *HelloRequest, opts ...gpubsub.PubOption) (error)
    SayHelloBatch(ctx context.Context, in []*HelloRequest, opts ...gpubsub.PubOption) (error)
}

type GreeterSub interface {
    GetHello(ctx context.Context, opts ...grpc.SubOption) (*HelloMessage, error)
    GetHellos(ctx context.Context, batchSize int, opts ...grpc.SubOption) ([]*HelloMessage, error)
}
```

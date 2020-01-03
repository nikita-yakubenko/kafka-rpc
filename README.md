# KRPC

---

## Introduction

KRPC, a RPC protocol that based on [kafka](https://kafka.apache.org/), is meant to provide a swift, stable, reliable remote calling service.

The reason we love about kafka is its fault tolerance, scalability and wicked large throughput.

So if you want a RPC service with kafka features, kRPC is the kind of tool you're looking for.

---

### FAQ

1. What is [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call)?

   RPC is a request–response protocol. An RPC is initiated by the client, which sends a request message to a known remote server to execute a specified procedure with supplied parameters.

   The one that posts the job request is called Client, while the other one that gets the job and responses is called Server.

2. When should I use a RPC?

   When you have to call a function that doesn't belong to local process or computer, but you don't want to build up another complex network framework just to implement restful api or soap, etc.
   RPC is much faster than restful api and easier to use. This is only a very few lines of code to be adjusted to implement a RPC service.

3. Why [krpc](https://github.com/zylo117/krpc/), not the other RPC protocols like [zerorpc](https://github.com/0rpc/zerorpc-python), [grpc](https://github.com/grpc/grpc), [mprpc](https://github.com/studio-ousia/mprpc)?

   Here is the comparison.

   | RPC                                               | MiddleWare | Serialization | Speed(QPS) |                                     Features                                     |
   | ------------------------------------------------- | :--------: | :-----------: | :--------: | :------------------------------------------------------------------------------: |
   | [krpc](https://github.com/zylo117/krpc/)          |   kafka    |    msgpack    |    160+    | dynamic load rebalance, large throughput, data persistence, faster serialization |
   | [zerorpc](https://github.com/0rpc/zerorpc-python) |   zeromq   |    msgpack    |    450+    |  dynamic load rebalance(failed when all server are busy), faster serialization   |
   | [grpc](https://github.com/grpc/grpc)              |  unknown   |   protobuf    | not tested |       dynamic load rebalance, large throughput, support only function rpc        |
   | [mprpc](https://github.com/studio-ousia/mprpc)    |     no     |    msgpack    |   19000+   |                                    lightspeed                                    |

   The only reason that I developed krpc is that zerorpc failed me!

   After months of searching and testing, zerorpc was the best rpc service I'd ever used, but it's bugged!
   Normally, developers won't notice, because most of the time, we use RPC to post the job from client directly to the server.

   But when you develop a distributing system, you will have to post K jobs of different types from N clients to M servers.

   The problem is that you don't really know which server is currently available, or right one for the job.

   And that's where load balancing comes in.

   Zeromq supports that, and zerorpc supports that too. However, sadly, zerorpc doesn't queue up the job, so when you have no server available at all, the jobs will not wait in line but instead, be abandoned.

   That's why I need kafka. Unlike most of the MQ, kafka provides data persistence, so no more job abandon. When all servers is unavailable, the job will queue up and wait in line.

   If the client crash, jobs will still be on disk (kafka features), unharmed, with replicas(can you imagine that).

   Also, krpc supports dynamic scalability, servers can be always added to the cluster or be removed, so jobs will be fairly distribute to all the servers, and will be reroute to another healthy server if assigned server is down.

#### Next Step

Yes, yes, I will try hard to optimize the QPS, maybe rewrite it in cython? Or allow asynchronous calls considering its large throughput advantage?
  
## USAGE

### Assuming you already a object and everything works, like this

#### local_call.py

    # Part1: define a class
    class Sum:
        def add(x, y):
            return x + y

    # Part2: instantiate a class to an object
    s = Sum()

    # Part3: call a method of the object
    result = s.add(1, 2)  # result = 3

### Then you can use RPC to run Part1 and Part2 on process1, and call the method of process 1 from process2

#### krpc_server_demo.py

    from krpc import KRPCServer

    # Part1: define a class
    class Sum:
        def add(x, y):
            return x + y

    # Part2: instantiate a class to an object
    s = Sum()

    # assuming you kafka broker is on 0.0.0.0:9092
    krs = KRPCServer('0.0.0.0', 9092, s, topic_name='sum')
    krs.server_forever()

#### krpc_client_demo.py

    # assuming you kafka broker is on 0.0.0.0:9092
    krc = KRPCClient('0.0.0.0', 9092, topic_name='sum')

    # call method from client to server
    result = krc.add(1, 2)

    # you can find the returned result in result['ret']
    # result = {
    #     'ret': 3,
    #     'tact_time': 0.007979869842529297,  # total process time
    #     'tact_time_server': 0.006567955017089844,  # process time on server side
    #     'server_id': '192.168.1.x'  # client ip
    # }

Additionally, you can enable redis to speed up caching, by adding use_redis=True to KRPCClient, or specify redis port, db and password, like this:

    krc = KRPCClient('0.0.0.0', 9092, topic_name='sum', use_redis=True, redis_port=6379, redis_db=0, redis_password='krpcno1')

### Noted

If use_redis=False, KRPCClient cannot be instantiated more than once

If use_redis=False, KRPCClient cannot be instantiated more than once

If use_redis=False, KRPCClient cannot be instantiated more than once

Do you remember now?
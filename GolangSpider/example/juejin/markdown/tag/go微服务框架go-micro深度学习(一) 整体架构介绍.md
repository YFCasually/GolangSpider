# go微服务框架go-micro深度学习(一) 整体架构介绍 #

产品嘴里的一个小项目，从立项到开发上线，随着时间和需求的不断激增，会越来越复杂，变成一个大项目，如果前期项目架构没设计的不好，代码会越来越臃肿，难以维护，后期的每次产品迭代上线都会牵一发而动全身。项目微服务化，松耦合模块间的关系，是一个很好的选择，随然增加了维护成本，但是还是很值得的。

![](https://user-gold-cdn.xitu.io/2019/2/26/16927b99e2cbcd61?imageView2/0/w/1280/h/960/ignore-error/1)

微服务化项目除了稳定性我个人还比较关心的几个问题：

一: 服务间数据传输的效率和安全性。

二: 服务的动态扩充，也就是服务的注册和发现，服务集群化。

三: 微服务功能的可订制化，因为并不是所有的功能都会很符合你的需求，难免需要根据自己的需要二次开发一些功能。

go-micro是go语言下的一个很好的rpc微服务框架，功能很完善，而且我关心的几个问题也解决的很好：

一：服务间传输格式为protobuf，效率上没的说，非常的快，也很安全。

二：go-micro的服务注册和发现是多种多样的。我个人比较喜欢etcdv3的服务服务发现和注册。

三：主要的功能都有相应的接口，只要实现相应的接口，就可以根据自己的需要订制插件。

业余时间把go-micro的源码系统地读了一遍，越读越感觉这个框架写的好，从中也学到了很多东西。就想整理一系列的帖子，把学习go-micro的心得和大家分享。

### 通信流程 ###

go-micro的通信流程大至如下

![](https://user-gold-cdn.xitu.io/2019/2/26/16927b99e35369d8?imageView2/0/w/1280/h/960/ignore-error/1)

Server监听客户端的调用，和Brocker推送过来的信息进行处理。并且Server端需要向Register注册自己的存在或消亡，这样Client才能知道自己的状态。

Register服务的注册的发现。

Client端从Register中得到Server的信息，然后每次调用都根据算法选择一个的Server进行通信，当然通信是要经过编码/解码，选择传输协议等一系列过程的。

如果有需要通知所有的Server端可以使用Brocker进行信息的推送。

Brocker 信息队列进行信息的接收和发布。

go-micro之所以可以高度订制和他的框架结构是分不开的，go-micro由8个关键的interface组成，每一个interface都可以根据自己的需求重新实现，这8个主要的inteface也构成了go-micro的框架结构。

![](https://user-gold-cdn.xitu.io/2019/2/26/16927ed85e573851?imageView2/0/w/1280/h/960/ignore-error/1)

这些接口go-micir都有他自己默认的实现方式，还有一个go-plugins是对这些接口实现的可替换项。你也可以根据需求实现自己的插件

![](https://user-gold-cdn.xitu.io/2019/2/26/16927b99e39b03d1?imageView2/0/w/1280/h/960/ignore-error/1)

这篇帖子主要是给大家介绍go-micro的主体结构和这些接口的功能，具体细节以后的文章我们再慢慢说：

### Transort ###

服务之间通信的接口。也就是服务发送和接收的最终实现方式，是由这些接口定制的。

源码：

` type Socket interface { Recv(*Message) error Send(*Message) error Close() error } type Client interface { Socket } type Listener interface { Addr() string Close() error Accept(func(Socket)) error } type Transport interface { Dial(addr string, opts ...DialOption) (Client, error) Listen(addr string, opts ...ListenOption) (Listener, error) String() string } 复制代码`

Transport 的Listen方法是一般是Server端进行调用的，他监听一个端口，等待客户端调用。

Transport 的Dial就是客户端进行连接服务的方法。他返回一个Client接口，这个接口返回一个Client接口，这个Client嵌入了Socket接口，这个接口的方法就是具体发送和接收通信的信息。

http传输是go-micro默认的同步通信机制。当然还有很多其他的插件：grpc,nats,tcp,udp,rabbitmq,nats，都是目前已经实现了的方式。在go-plugins里你都可以找到。

### Codec ###

有了传输方式，下面要解决的就是传输编码和解码问题，go-micro有很多种编码解码方式，默认的实现方式是protobuf,当然也有其他的实现方式，json、protobuf、jsonrpc、mercury等等。

源码

` type Codec interface { ReadHeader(*Message, MessageType) error ReadBody(interface{}) error Write(*Message, interface{}) error Close() error String() string } type Message struct { Id uint64 Type MessageType Target string Method string Error string Header map[string]string } 复制代码`

Codec接口的Write方法就是编码过程，两个Read是解码过程。

### Registry ###

服务的注册和发现，目前实现的consul,mdns, etcd,etcdv3,zookeeper,kubernetes.等等，

` type Registry interface { Register(*Service, ...RegisterOption) error Deregister(*Service) error GetService(string) ([]*Service, error) ListServices() ([]*Service, error) Watch(...WatchOption) (Watcher, error) String() string Options() Options } 复制代码`

简单来说，就是Service 进行Register，来进行注册，Client 使用watch方法进行监控，当有服务加入或者删除时这个方法会被触发，以提醒客户端更新Service信息。

默认的是服务注册和发现是consul，但是个人不推荐使用，因为你不能直接使用consul集群

![](https://user-gold-cdn.xitu.io/2019/2/26/16927b99ffaeb4c2?imageView2/0/w/1280/h/960/ignore-error/1)

我个人比较喜欢etcdv3集群。大家可以根据自己的喜好选择。

### Selector ###

以Registry为基础，Selector 是客户端级别的负载均衡，当有客户端向服务发送请求时， selector根据不同的算法从Registery中的主机列表，得到可用的Service节点，进行通信。目前实现的有循环算法和随机算法，默认的是随机算法。

源码：

` type Selector interface { Init(opts ...Option) error Options() Options // Select returns a function which should return the next node Select(service string, opts ...SelectOption) (Next, error) // Mark sets the success/error against a node Mark(service string, node *registry.Node, err error) // Reset returns state back to zero for a service Reset(service string) // Close renders the selector unusable Close() error // Name of the selector String() string } 复制代码`

默认的是实现是本地缓存，当前实现的有blacklist,label,named等方式。

### Broker ###

Broker是消息发布和订阅的接口。很简单的一个例子，因为服务的节点是不固定的，如果有需要修改所有服务行为的需求，可以使服务订阅某个主题，当有信息发布时，所有的监听服务都会收到信息，根据你的需要做相应的行为。

源码

` type Broker interface { Options() Options Address() string Connect() error Disconnect() error Init(...Option) error Publish(string, *Message, ...PublishOption) error Subscribe(string, Handler, ...SubscribeOption) (Subscriber, error) String() string } 复制代码`

Broker默认的实现方式是http方式，但是这种方式不要在生产环境用。go-plugins里有很多成熟的消息队列实现方式，有kafka、nsq、rabbitmq、redis，等等。

### Client ###

Client是请求服务的接口，他封装Transport和Codec进行rpc调用，也封装了Brocker进行信息的发布。

源码

` type Client interface { Init(...Option) error Options() Options NewMessage(topic string, msg interface{}, opts ...MessageOption) Message NewRequest(service, method string, req interface{}, reqOpts ...RequestOption) Request Call(ctx context.Context, req Request, rsp interface{}, opts ...CallOption) error Stream(ctx context.Context, req Request, opts ...CallOption) (Stream, error) Publish(ctx context.Context, msg Message, opts ...PublishOption) error String() string } 复制代码`

当然他也支持双工通信 Stream 这些具体的实现方式和使用方式，以后会详细解说。

默认的是rpc实现方式，他还有grpc和http方式，在go-plugins里可以找到

### Server ###

Server看名字大家也知道是做什么的了。监听等待rpc请求。监听broker的订阅信息，等待信息队列的推送等。

源码

` type Server interface { Options() Options Init(...Option) error Handle(Handler) error NewHandler(interface{}, ...HandlerOption) Handler NewSubscriber(string, interface{}, ...SubscriberOption) Subscriber Subscribe(Subscriber) error Register() error Deregister() error Start() error Stop() error String() string } 复制代码`

默认的是rpc实现方式，他还有grpc和http方式，在go-plugins里可以找到

### Service ###

Service是Client和Server的封装，他包含了一系列的方法使用初始值去初始化Service和Client，使我们可以很简单的创建一个rpc服务。

源码：

` type Service interface { Init(...Option) Options() Options Client() client.Client Server() server.Server Run() error String() string } 复制代码`

具体的细节，我以后的帖子会给大家一一展开，希望这篇帖子，可以帮助你对go-micro的整体框架有个初步了解
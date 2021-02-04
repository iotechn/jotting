#### Netty ChannelGroup

> 接口原意（取至Netty文档）
>
> A thread-safe [`Set`](https://docs.oracle.com/javase/7/docs/api/java/util/Set.html?is-external=true) that contains open [`Channel`](https://netty.io/4.1/api/io/netty/channel/Channel.html)s and provides various bulk operations on them. Using [`ChannelGroup`](https://netty.io/4.1/api/io/netty/channel/group/ChannelGroup.html), you can categorize [`Channel`](https://netty.io/4.1/api/io/netty/channel/Channel.html)s into a meaningful group (e.g. on a per-service or per-state basis.) A closed [`Channel`](https://netty.io/4.1/api/io/netty/channel/Channel.html) is automatically removed from the collection, so that you don't need to worry about the life cycle of the added [`Channel`](https://netty.io/4.1/api/io/netty/channel/Channel.html). A [`Channel`](https://netty.io/4.1/api/io/netty/channel/Channel.html) can belong to more than one [`ChannelGroup`](https://netty.io/4.1/api/io/netty/channel/group/ChannelGroup.html).
>
> 一个安全的集合，包含了打开的Channels 和 为这些Channels提供多样的批量操作. 使用ChannelGroup，你可以将Channels有意义的分类（例如：在每一个服务 或 每一个基础状态或区域）。一个已经关闭的Channel会自动的从这个集合中移除，你无需担心你加进来的Channel的生命周期。一个Channel可以属于多个ChannelGroup。



DefaultChannelGroup相当于一个线程安全的Set集合，可以将一系列Channel放入Group中。

DefaultChannelGroup是Netty默认唯一一个实现了ChannelGroup接口的类；其继承了Java的AbstractSet，其内部封装了两个ConurrentSet<Channel>，分别用于存放ServerChannel 与 客户端的普通 Channel



### ChannelGroup 缺点

显然，他不能通过Key进行路由。可以进行扩展一下，将ConurrentSet转换为ConurrentMap，并将ID作为Set的键，并扩展出一个方法，通过ID获取Channel。这样就不用冗余一个ConurrentMap来存储ID Channel映射表了。



在Netty 4.1 之后已经采用了ConurrentMap，但是并没有开放ID查找的方法



### ChannelGroup自身路由

若是用户处于多个ChannelGroup中，要对特定的ChannelGroup进行广播消息。

在数据库定义好一些固定的ChannelGroupId，例如我在这张图里面，需要接受这张图的广播消息，我则 Map-0001 作为Key，ChannelGroup 作为Value。启动时创建一个空的ChannelGroup，当用户进入这张图，则使用户加入到这个ChannelGroup，因为用户是主动加入到这张图的，肯定有这样图的ID。我用户需要发消息时，就将对这张图发消息。之前可能会做点权限，流量控制之类的。



关于临时的ChannelGroup，比如临时开通的几个人的会话，则不用在数据库中去存，只需要将临时生成一个不重复的ID，并且将几个人放进去就行了。





### Netty消息处理设计

1. 收消息，使用 长度 + 消息体的形式来解决断包粘包的问题。

   Header定义 4byte 消息总长  4byte 消息类型（命令类型） 消息总长byte的消息体（比如序列化好的一个对象）

2. 先读尝试读 4byte，然后尝试再读4 + 消息总长byte。

3. 从第5-8byte中判断出消息类型

4. 通过消息类型，将消息体进行反序列化，变成一个对象（Command？暂定）



调用方案1

将业务层直接提供一个统一接口，接收一个Command的参数，在业务层通过Command携带的消息类型进行路由，分配到各个方法里面去处理。

1. 当连接层反序列对象时会做一次路由

2. 业务层拿到Command的时候又会做一次路由

   但是呢，连接层只关注了消息的收发，并不管调用逻辑，好像要科学一点。

   当增加一个Command时需要重启连接层（以增加Command类）



调用方案2

将接口暴露给连接层，连接层就可以通过接口 + 动态代理 获取到代理实例。将这些接口，按照Map<Int, Method>的方式进行映射，当获取到Command时，直接通过消息类型反序列化为Command，并通过消息类型拿到Method，直接进行RPC。

	1. 增加接口，修改Command都需要重启连接层



调用方案3

连接层获取到数据以后，将数据格式化为

RpcContext {

​	int command;

​	byte[] body;

}

的形式直接进行RPC调用业务层，

业务层拿到命令RpcContext对body进行反序列化，并且进行命令路由，进行处理。



### RPC实现

RPC可以直接集成dubbo，就不用写序列化反序列化 代理相关的逻辑了，也可以使用Netty。

优先使用Netty进行RPC吧。

### 消息发出

连接层允许接受业务层的N种命令，例如T客户下线，向某个客户发送消息，向某个Group发送消息。

业务层如何将命令传达？

1. 用消息队列，好像Netty自己都能发消息了，增加系统运行成本。
2. RPC：循环依赖？
3. 在连接层额外监听一个端口，通过安全组来控制访问权限，这个端口用来接受业务层发过来的命令。

### 框架选择

1. 连接层需要轻量级，只用Netty，不用Spring集成
2. 业务层需要连接数据库，缓存等，相对业务较多，所以就整SpringBoot那一套吧




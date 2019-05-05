网络和消息
========================

Corda uses AMQP/1.0 over TLS between nodes which is currently implemented using Apache Artemis, an embeddable message
queue broker. Building on established MQ protocols gives us features like persistence to disk, automatic delivery
retries with backoff and dead-letter routing, security, large message streaming and so on.

Corda 在 TLS 之上使用 AMQP/1.0 实现节点间的通信，当前是使用 Apache Artemis，一个可嵌入的 message queue broker 来实现的。构建在创建好的 MQ 协议上给了我们诸如持久化到硬盘，带有 backoff 和 dead-letter routing 的自动重试发送，安全和大的消息流等功能。

Artemis is hidden behind a thin interface that also has an in-memory only implementation suitable for use in
unit tests and visualisation tools.

Artemis 隐藏在一个轻接口后边，这个接口还具有一个只在内存中存储的实现，这个在单元测试（unit tests）和可视化工具（visualisation tools）可以使用。

.. note:: A future version of Corda will allow the MQ broker to be split out of the main node and run as a
   separate server. We may also support non-Artemis implementations via JMS, allowing the broker to be swapped
   out for alternative implementations.

.. note:: 未来版本的 Corda 将会允许 MQ broker 可以从主节点中分离出来并且像一个独立的 server 那样运行。我们可能也会支持通过 JMS 的 non-Arteis 实现，允许 broker 能够有其他的实现。

There are multiple ways of interacting with the network. When writing an application you typically won't use the
messaging subsystem directly. Instead you will build on top of the :doc:`flow framework <flow-state-machines>`,
which adds a layer on top of raw messaging to manage multi-step flows and let you think in terms of identities
rather than specific network endpoints.

这有很多种方式跟网络进行互动。当编写一个应用的时候，你通常不会直接地使用一个消息子系统。你会将它构建在 flow 框架之上，这会在原始消息（raw messaging）之上添加一层来管理多步的 flows 并且让你思考关于身份（identities）而不是某个特定的网络 endpoints。

.. _network-map-service:

网络地图服务
-------------------

Supporting the messaging layer is a network map service, which is responsible for tracking public nodes on the network.

网络地图服务支持了消息层，它负责跟踪网络中的公共节点。

Nodes have an internal component, the network map cache, which contains a copy of the network map (which is backed up in the database
to persist that information across the restarts in case the network map server is down). When a node starts up its cache
fetches a copy of the full network map (from the server or from filesystem for development mode). After that it polls on
regular time interval for network map and applies any related changes locally.
Nodes do not automatically deregister themselves, so (for example) nodes going offline briefly for maintenance are retained
in the network map, and messages for them will be queued, minimising disruption.

节点具有一个内部的组件，网络地图缓存，它包括了网络地图的一个副本（只是一个文件）。当一个节点启动的时候，它的缓存会获取整个网络地图的一个副本，并且会请求当网络地图有变化的时候被通知。然后节点会将自己注册到网络地图服务中，这个服务会通知所有订阅的节点有个新的节点加入到网络中来。节点不会自动地将自己从注册中移除，所以当节点由于维护等原因离线之后，他们仍旧会在网络地图中存在，发送给他们的消息会被放入队列中，以最小化造成的损坏。

Additionally, on every restart and on daily basis nodes submit signed ``NodeInfo`` s to the map service. When network map gets
signed, these changes are distributed as new network data. ``NodeInfo`` republishing is treated as a heartbeat from the node,
based on that network map service is able to figure out which nodes can be considered as stale and removed from the network
map document after ``eventHorizon`` time.

另外，在每次重启以及每天，节点都会像这个地图服务提交一个签过名的 ``NodeInfo``。当网络地图得到这些签过名的信息之后，这些变化会作为网络数据被分发出去。``NodeInfo`` 的重新发布就像来自于节点的一次心跳一样，基于它网络地图服务就能够区别出来哪些节点能够被认为是稳定的，哪些可以在 ``eventHorizon`` 时间之后就可以从网络地图文件中移除出去了。

消息队列
--------------

The node makes use of various queues for its operation. The more important ones are described below. Others are used
for maintenance and other minor purposes.

节点运行的时候会使用不同的消息队列。比较重要的都在下边进行了描述。其他的队列会被用来在维护和其他比较小的一些活动中。

:``p2p.inbound.$identity``:
   The node listens for messages sent from other peer nodes on this queue. Only clients who are authenticated to be
   nodes on the same network are given permission to send. Messages which are routed internally are also sent to this
   queue (e.g. two flows on the same node communicating with each other).

   节点会在这个消息队列中监听由其他节点发送过来的消息。只有同一网络中已经授权的节点才会有权限发送消息。在内部路由的消息也会发送到这个队列中（比如一个节点中的两个 flows 互相联系）。

:``internal.peers.$identity``:
   These are a set of private queues only available to the node which it uses to route messages destined to other peers.
   The queue name ends in the base 58 encoding of the peer's identity key. There is at most one queue per peer. The broker
   creates a bridge from this queue to the peer's ``p2p.inbound.$identity`` queue, using the network map service to lookup the
   peer's network address.

   这些是一系列的私有队列，是节点用来向其他节点发送消息的。队列的名字是以 base 58 加密的目标节点的身份密钥的信息作为结尾。这里基本上是每个 peer 节点一个队列。Broker 会通过网络地图服务来查找 peer 节点的网络地址，然后跟对方节点的 p2p.inbound.$identity 队列建立一个桥连接（bridge）。

:``internal.services.$identity``:
   These are private queues the node may use to route messages to services. The queue name ends in the base 58 encoding
   of the service's owning identity key. There is at most one queue per service identity (but note that any one service
   may have several identities). The broker creates bridges to all nodes in the network advertising the service in
   question. When a session is initiated with a service counterparty the handshake is pushed onto this queue, and a
   corresponding bridge is used to forward the message to an advertising peer's p2p queue. Once a peer is picked the
   session continues on as normal.

   这些是节点可能会用来向服务（services）发送消息的私有队列。队列的名字是以 base 58 加密的服务的身份密钥的信息作为结尾。这里每个服务标识（identity）最多是一个队列（但是要注意的是一个服务可能会有多个标识）。Broker 会跟所有的网络中提供服务的节点创建桥连接（bridges）。当跟服务合作方建立一个回话之后，handshake 会被发送到这个队列中来，并且一个对应的桥连接会被用来将消息转发给目标节点的 p2p 队列。当一个 peer 被选择之后，会话将会正常进行。

:``rpc.server``:
   RPC clients send their requests here, and it's only open for sending by clients authenticated as RPC users.

   RPC 客户端通过该队列发送请求，这个队列也仅仅对被授权为 RPC 用户的客户端才可以访问。

:``rpc.client.$user.$random``:
   RPC clients are given permission to create a temporary queue incorporating their username (``$user``) and sole
   permission to receive messages from it. RPC requests are required to include a random number (``$random``) from
   which the node is able to construct the queue the user is listening on and send the response to that. This mechanism
   prevents other users from being able listen in on the responses.

   RPC 客户端被授权使用他们的用户名（``$user``）来创建一个临时的队列，并有权利从这个都列中接收消息。RPC 请求必须要包含一个随机数（``$random``），通过它节点可以构建用户可以监听并且可以向其发送反馈的队列。这个机制能够避免其他的用户能够监听反馈。

安全
--------

Clients attempting to connect to the node's broker fall in one of four groups:

#. Anyone connecting with the username ``SystemUsers/Node`` or ``SystemUsers/NodeRPC`` is treated as the node hosting the brokers, or a logical
   component of the node. The TLS certificate they provide must match the one broker has for the node. If that's the case
   they are given full access to all valid queues, otherwise they are rejected.

#. Anyone connecting with the username ``SystemUsers/Peer`` is treated as a peer on the same Corda network as the node. Their
   TLS root CA must be the same as the node's root CA -- the root CA is the doorman of the network and having the same root CA
   implies we've been let in by the same doorman. If they are part of the same network then they are only given permission
   to send to our ``p2p.inbound.$identity`` queue, otherwise they are rejected.

#. Every other username is treated as a RPC user and authenticated against the node's list of valid RPC users. If that
   is successful then they are only given sufficient permission to perform RPC, otherwise they are rejected.

#. Clients connecting without a username and password are rejected.

客户端尝试连接节点的 broker 会出自四个 groups 之一：

#. 任何使用用户名 ``SystemUsers/Node`` 或者 ``SystemUsers/NodeRPC`` 连接的被认为是运行着 broker 的节点，或者是节点的一个逻辑组件（logical component）。他们所提供的 TLS 证书必须要跟 broker 对于该节点的证书匹配。如果匹配上的话，那么他们会被授权访问所有可用的队列，否则的话会被拒绝访问。
#. 任何使用用户名 ``SystemUsers/Peer`` 连接的被认为是在同一 Corda 网络中的 peer 节点。他们的 TLS root CA 必须同节点的 root CA 相同。Root CA 是网络的 doorman，并且具有相同的 root CA 意味着我们是被相同的 doorman 准入进入到此网络的。如果他们是同一网络的组成节点，那么他们仅仅被授权向我们的 ``p2p.inbound.$identity`` 队列发送消息，否则的话会被拒绝。
#. 任何其他的用户名会被认为是一个 RPC 用户，会根据节点的有效的 RPC 用户列表来进行授权。如果授权成功，那么他们仅仅会给与有效的权力来执行 RPC，否则会被拒绝。
#. 没有用户名和密码的客户端会被拒绝。

Artemis provides a feature of annotating each received message with the validated user. This allows the node's messaging
service to provide authenticated messages to the rest of the system. For the first two client types described above the
validated user is the X.500 subject of the client TLS certificate. This allows the flow framework to authentically determine
the ``Party`` initiating a new flow. For RPC clients the validated user is the username itself and the RPC framework uses
this to determine what permissions the user has.

Artemis 提供了一个功能，为每一个收到的消息田间有效的用户的注解。这个允许节点的消息服务为系统的其他部分提供被授权的消息。对于上边提到的前两种客户端类型，被验证的用户是客户端的 TLS 证书的 X.500 subject。这个允许 flow 框架来判断初始一个新 flow 的 ``Party`` 的权限。对于 RPC 客户端，被验证的用户是指用户名本身，RPC 框架用这个来决定这个用户应该有什么权限。

The broker also does host verification when connecting to another peer. It checks that the TLS certificate subject matches
with the advertised X.500 legal name from the network map service.

当连接其他 peer 的时候，broker 也会进行验证。他会检查 TLS 证书 subject 同网络地图服务中记录的 X.500 legal name 相匹配。

实现的细节
~~~~~~~~~~~~~~~~~~~~~~

The components of the system that need to communicate and authenticate each other are:
   - The Artemis P2P broker (currently runs inside the node's JVM process, but in the future it will be able to run as a separate server):
      * Opens Acceptor configured with the doorman's certificate in the trustStore and the node's SSL certificate in the keyStore.
   - The Artemis RPC broker (currently runs inside the node's JVM process, but in the future it will be able to run as a separate server):
      * Opens "Admin" Acceptor configured with the doorman's certificate in the trustStore and the node's SSL certificate in the keyStore.
      * Opens "Client" Acceptor with the SSL settings configurable. This acceptor does not require SSL client-auth.
   - The current node hosting the brokers:
      * Connects to the P2P broker using the ``SystemUsers/Node`` user and the node's keyStore and trustStore.
      * Connects to the "Admin" Acceptor of the RPC broker using the ``SystemUsers/NodeRPC`` user and the node's keyStore and trustStore.
   - RPC clients (third party applications that need to communicate with the node):
      * Connect to the "Client" Acceptor of the RPC broker using the username/password provided by the node's admin. The client verifies the node's certificate using a trustStore provided by the node's admin.
   - Peer nodes (other nodes on the network):
      * Connect to the P2P broker using the ``SystemUsers/Peer`` user and a doorman signed certificate. The authentication is performed based on the root CA.
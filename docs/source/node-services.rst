节点服务
=============

This document is intended as a very brief introduction to the current 
service components inside the node. Whilst not at all exhaustive it is 
hoped that this will give some context when writing applications and 
code that use these services, or which are operated upon by the internal 
components of Corda.

这篇文章目的是为了非常简单地介绍一下节点内提供的当前版本的服务。这里讲的并不会很详细，我们希望这些内容在你在开发使用这些服务的应用或者代码的时候，能够给你提供更多的上下文信息。

节点内的服务
------------------------

The node services represent the various sub functions of the Corda node. 
Some are directly accessible to contracts and flows through the 
``ServiceHub``, whilst others are the framework internals used to host 
the node functions. Any public service interfaces are defined in the 
``net.corda.core.node.services`` package. The ``ServiceHub`` interface exposes
functionality suitable for flows.
The implementation code for all standard services lives in the ``net.corda.node.services`` package.

节点的服务体现了一个 Corda 节点所具有的多种子功能（sub functions）。一些服务能够通过 ``ServiceHub`` 直接访问 contracts 和 flows，但是还有一些服务是框架内部使用的。任何的公共服务接口会定义在 ``net.corda.core.node.services`` 包中。``ServiceHub`` 接口暴露了适用于 flows 的一些功能。所有标准服务的实现都在 ``net.corda.node.services`` 包中定义的。

All the services are constructed in the ``AbstractNode`` ``start`` 
method. They may also register a shutdown handler during initialisation,
which will be called in reverse order to the start registration sequence when the ``Node.stop`` is called.

所有的服务都是在 ``AbstractNode`` ``start`` 方法中被构建的。他们可能在初始化的时候还会注册一个关闭处理，当 ``Node.stop`` 被调用的时候会被调用。

The roles of the individual services are described below.

每一个服务的角色描述如下。

秘钥管理和身份服务
------------------------------------

InMemoryIdentityService
~~~~~~~~~~~~~~~~~~~~~~~

The ``InMemoryIdentityService`` implements the ``IdentityService`` 
interface and provides a store of remote mappings between ``PublicKey``
and remote ``Parties``. It is automatically populated from the 
``NetworkMapCache`` updates and is used when translating ``PublicKey``
exposed in transactions into fully populated ``Party`` identities. This 
service is also used in the default JSON mapping of parties in the web 
server, thus allowing the party names to be used to refer to other nodes' 
legal identities. In the future the Identity service will be made 
persistent and extended to allow anonymised session keys to be used in 
flows where the well-known ``PublicKey`` of nodes need to be hidden
to non-involved parties.

``InMemoryIdentityService`` 实现了 ``IdentityService`` 接口并且提供了对于 ``PublicKey`` 和远程的 ``Parties`` 的 mapping 的存储。它会从 ``NetworkMapCache`` 更新中自动被抛出来，然后被用来将 transaction 中暴露出来的 ``PublicKey`` 转变为完整的 ``Party`` identities。这个服务也会用来对网络中的 parties 做默认的 JSON mapping，因此允许了 party name 可以被用来作为其他节点的 legal identities 的引用。未来的 Identity 服务会变成持久化的和可扩展的，来允许在对于非相关的 parties 需要隐藏节点的 well-known ``PublicKey`` 的时候，在 flows 中可以使用 anonymised session keys。

PersistentKeyManagementService 和 E2ETestKeyManagementService
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Typical usage of these services is to locate an appropriate 
``PrivateKey`` to complete and sign a verified transaction as part of a 
flow. The normal node legal identifier keys are typically accessed via 
helper extension methods on the ``ServiceHub``, but these ultimately delegate
signing to internal ``PrivateKeys`` from the ``KeyManagementService``. The
``KeyManagementService`` interface also allows other keys to be 
generated if anonymous keys are needed in a flow. Note that this 
interface works at the level of individual ``PublicKey`` and internally
matched ``PrivateKey` pairs, but the signing authority may be represented by a 
``CompositeKey`` on the ``NodeInfo`` to allow key clustering and 
threshold schemes.

通常情况下，使用这些服务是为了找到一个合适的 ``PrivateKey`` 来完成一个 flow 中的已确认的 transaction 并提供签名。常规的节点的 legal identifier keys 通常会使用 ``ServiceHub`` 中的 helper 扩展方法来访问，但是这最终会委托签名给来自于 ``KeyManagementService`` 的内部的 ``PrivateKeys``。``KeyManagementService`` 接口也允许在一个 flow 中需要将签名 anonymous keys 的时候，允许生成其他的 keys。需要注意的是，这个接口是在每个单独的 ``PublickKey`` 的水平上工作的，并且应该同 ``NodeInfo`` 上的 ``PrivateKey`` 配对,但是签名的 authority 可能会以 NodeInfo 的 ``CompositeKey`` 来表示，这样就允许 key clustering 和 threshold schemes。

The ``PersistentKeyManagementService`` is a persistent implementation of 
the ``KeyManagementService`` interface that records the key pairs to a 
key-value storage table in the database. ``E2ETestKeyManagementService`` 
is a simple implementation of the ``KeyManagementService`` that is used 
to track our ``KeyPairs`` for use in unit testing when no database is 
available.

``PersistentKeyManagementService`` 是 ``KeyManagementService`` 接口的一个持久化实现，会将秘钥对存储在数据库中的一个 key-value 存储表中。``E2ETestKeyManagementService`` 是一个 ``KeyManagementService`` 的简单实现，用来跟踪当没有数据库的时候，我们的 ``KeyPairs`` 在单元测试中是如何被使用的。

消息和网络管理服务
-----------------------------------------

ArtemisMessagingServer
~~~~~~~~~~~~~~~~~~~~~~

The ``ArtemisMessagingServer`` service is run internally by the Corda 
node to host the ``ArtemisMQ`` messaging broker that is used for 
reliable node communications. Although the node can be configured to 
disable this and connect to a remote broker by setting the 
``messagingServerAddress`` configuration to be the remote broker 
address. (The ``MockNode`` used during testing does not use this 
service, and has a simplified in-memory network layer instead.) This 
service is not exposed to any CorDapp code as it is an entirely internal 
infrastructural component. However, the developer may need to be aware 
of this component, because the ``ArtemisMessagingServer`` is responsible 
for configuring the network ports (based upon settings in ``node.conf``) 
and the service configures the security settings of the ``ArtemisMQ`` 
middleware and acts to form bridges between node mailbox queues based 
upon connection details advertised by the ``NetworkMapCache``. The
``ArtemisMQ`` broker is configured to use TLS1.2 with a custom 
``TrustStore`` containing a Corda root certificate and a ``KeyStore`` 
with a certificate and key signed by a chain back to this root 
certificate. These keystores typically reside in the ``certificates`` 
sub folder of the node workspace. For the nodes to be able to connect to 
each other it is essential that the entire set of nodes are able to 
authenticate against each other and thus typically that they share a 
common root certificate. Also note that the address configuration 
defined for the server is the basis for the address advertised in the 
``NetworkMapCache`` and thus must be externally connectable by all nodes
in the network.

``ArtemisMessagingServer`` 服务是在 Corda 节点内部运行的承载 ``ArtemisMQ`` 的信息 broker，它用来进行节点间的可信赖的通信。尽管节点可以被配置为不开启这个并且通过将 ``messagingServerAddress`` 配置设置为远程的 broker 地址来链接一个远程的 broker。（在测试节点的过程中所使用的 ``MockNode`` 并不使用这个服务，取而代之的是一个在内存中的简化的网络层）这个服务不会暴露给任何 CorDapp 代码因为它整体上是一个内部的结构组件。然而，开发者可能需要注意一下这个组件，因为 ``ArtemisMessagingServer`` 是负责配置网络的端口（基于 node.conf 中的设置）并且服务还配置了 ``ArtemisMQ`` 中间件的安全设置，并基于在 ``NetworkMapService`` 中发布的连接详细信息作为一个构成结点邮箱队列的桥梁。``ArtemisMQ`` broker 被设置使用 TLS1.2 并带有一个自定义的 ``TrustStore``，其包含了一个 Corda 根证书和一个 ``KeyStore``，这个 ``KeyStore`` 包含了一个证书和一个由能够关联回这个根证书所签名的密钥。这些 KeyStores 通常会贮存在节点的工作路径下的 ``certificates`` 子文件夹下。为了能够使节点间互相连接，整体所有的节点间能够彼此互相验证并且相信他们是在共享一个公用的根证书是非常必要的。并且还要注意的是对于 server 定义的地址信息是 ``NetworkMapService`` 广播出去的地址的基础，所以必须要能够被网络中的所有节点在外部连接的。

P2PMessagingClient
~~~~~~~~~~~~~~~~~~

The ``P2PMessagingClient`` is the implementation of the
``MessagingService`` interface operating across the ``ArtemisMQ`` 
middleware layer. It typically connects to the local ``ArtemisMQ`` 
hosted within the ``ArtemisMessagingServer`` service. However, the 
``messagingServerAddress`` configuration can be set to a remote broker 
address if required. The responsibilities of this service include 
managing the node's persistent mailbox, sending messages to remote peer 
nodes, acknowledging properly consumed messages and deduplicating any 
resent messages. The service also handles the incoming requests from new 
RPC client sessions and hands them to the ``CordaRPCOpsImpl`` to carry 
out the requests.

``P2PMessagingClient`` 是运行在跨 ``ArtemisMQ`` 中间件层的 ``MessagingService`` 接口的实现。它通常连接到运行在 ``ArtemisMessagingServer`` 服务内部的本地 ``ArtemisMQ`` 上。然而，如果需要的话 ``messagingServerAddress`` 配置项可以被设置到一个远程的 broker 地址。这个服务的责任包括管理节点的持久化邮箱，发送信息到远程的 peer 节点，正确地接受被消费的消息和复制任何重新发布的消息。这个服务还能够处理从新的 RPC 客户端对话中的传入的请求并且将他们传递给 ``CordaRPCOpsImpl`` 来处理这个请求。

InMemoryNetworkMapCache
~~~~~~~~~~~~~~~~~~~~~~~

The ``InMemoryNetworkMapCache`` implements the ``NetworkMapCache`` 
interface and is responsible for tracking the identities and advertised 
services of authorised nodes provided by the remote 
``NetworkMapService``. Typical use is to search for nodes hosting 
specific advertised services e.g. a Notary service, or an Oracle 
service. Also, this service allows mapping of friendly names, or 
``Party`` identities to the full ``NodeInfo`` which is used in the 
``StateMachineManager`` to convert between the ``PublicKey``, or
``Party`` based addressing used in the flows/contracts and the 
physical host and port information required for the physical 
``ArtemisMQ`` messaging layer.

``InMemoryNetworkMapCache`` 实现了 ``NetworkMapCache`` 接口，并且负责跟踪由远程的 ``NetworkMapService`` 提供的通过认证的节点的身份信息和提供的服务信息。通常被用来检索带有指定服务的节点，比如一个 Notary service 或者一个 Oracle service。这个服务也允许将友好的名字进行 mapping，或者将一个 ``Party`` 的身份信息变成一个完整的 ``NodeInfo``，这个信息会在 ``StateMachineManager`` 中被用来在 ``PublicKey`` 之间或者在 flows/contracts 中使用的基于 ``Party`` 的地址和物理的 ``ArtemisMQ`` 信息层需要的物理的 host 和端口信息进行转换。

存储和持久化相关的服务
----------------------------------------

DBCheckpointStorage
~~~~~~~~~~~~~~~~~~~

The ``DBCheckpointStorage`` service is used from within the 
``StateMachineManager`` code to persist the progress of flows. Thus 
ensuring that if the program terminates the flow can be restarted 
from the same point and complete the flow. This service should not 
be used by any CorDapp components.

``DBCheckpointStorage`` 服务被使用在从 ``StateMachineManager`` 代码内部到持久化 flows 的流程中。因此如果程序终止了，要确保 flows 能够从相同的时间点重新开始并且完成整个 flow。这个服务不应该被任何的 CorDapp 组件来使用。

DBTransactionMappingStorage 和 InMemoryStateMachineRecordedTransactionMappingStorage
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``DBTransactionMappingStorage`` is used within the 
``StateMachineManager`` code to relate transactions and flows. This 
relationship is exposed in the eventing interface to the RPC clients, 
thus allowing them to track the end result of a flow and map to the 
actual transactions/states completed. Otherwise this service is unlikely 
to be accessed by any CorDapps. The 
``InMemoryStateMachineRecordedTransactionMappingStorage`` service is 
available as a non-persistent implementation for unit tests with no database.

``DBTransactionMappingStorage`` 在 ``StateMachineManager`` 代码中被用来关联 transactions 和 flows 的。这种关系在对 RPC 客户端的事件接口中被暴露出来，因此允许他们能够跟踪一个 flow 的最终结果并且能够匹配到真正结束了的 transactions/states 上。否则的话这个服务可能不会被任何的 CorDapps 所访问。``InMemoryStateMachineRecordedTransactionMappingStorage`` 服务是作为一个使用不需要数据库的单元测试非持久化的一个实现。

DBTransactionStorage
~~~~~~~~~~~~~~~~~~~~

The ``DBTransactionStorage`` service is a persistent implementation of 
the ``TransactionStorage`` interface and allows flows read-only 
access to full transactions, plus transaction level event callbacks. 
Storage of new transactions must be made via the ``recordTransactions`` 
method on the ``ServiceHub``, not via a direct call to this service, so 
that the various event notifications can occur.

``DBTransactionStorage`` 服务是 ``TransactionStorage`` 接口的一个持久化实现，并且允许 flows 以只读权限来访问整体 transactions，加上 transaction 级别的事件回调方法。新的 transactions 的存储必须通过在 ``ServiceHub`` 上的 ``recordTransactions`` 方法来生成，并不是通过对这个服务的直接调用，所以大量的事件通知会发生。

NodeAttachmentService
~~~~~~~~~~~~~~~~~~~~~

The ``NodeAttachmentService`` provides an implementation of the 
``AttachmentStorage`` interface exposed on the ``ServiceHub`` allowing 
transactions to add documents, copies of the contract code and binary 
data to transactions. The service is also interfaced to by the web server,
which allows files to be uploaded via an HTTP post request.

``NodeAttachmentService`` 提供了一个在 ``ServiceHub`` 上暴露的 ``AttachmentStorage`` 接口的实现，它允许 transactions 可以添加文档，contract code 的拷贝和对于 transaction 的二进制数据。这个服务同样也被 web server 作为接口，来允许文件能够通过 HTTP post 请求来上传。

Flow framework 和 event scheduling services
--------------------------------------------

StateMachineManager
~~~~~~~~~~~~~~~~~~~

The ``StateMachineManager`` is the service that runs the active 
flows of the node whether initiated by an RPC client, the web 
interface, a scheduled state activity, or triggered by receipt of a 
message from another node. The ``StateMachineManager`` wraps the 
flow code (extensions of the ``FlowLogic`` class) inside an 
instance of the ``FlowStateMachineImpl`` class, which is a 
``Quasar`` ``Fiber``. This allows the ``StateMachineManager`` to suspend 
flows at all key lifecycle points and persist their serialized state 
to the database via the ``DBCheckpointStorage`` service. This process 
uses the facilities of the ``Quasar`` ``Fibers`` library to manage this 
process and hence the requirement for the node to run the ``Quasar`` 
java instrumentation agent in its JVM.

``StateMachineManager`` 是运行节点的 active flows 的服务，这些 flows 可能是由一个 RPC 客户端， web 接口，一个 scheduled state activity 来初始化的，或者是由通过接收到从其他节点发来的一个消息所出发。``StateMachineManager`` 包装了在一个 ``FlowStateMachineImpl`` 类实例里的 flow 代码（对 ``FlowLogic`` 类的扩展），这个类是一个 ``Quasar`` ``Fiber``。这个允许 ``StateMachineManager`` 可以在生命周期的所有关键时间点挂起 flows，并将他们序列化的 state 通过 ``DBCheckpointStorage`` 服务持久化到数据库中。这个过程使用了 ``Quasar`` ``Fibers`` 类库的协助来管理这个流程，因此这就要求节点需要在它的 JVM 中运行 ``Quasar`` java instrumentation agent。

In operation the ``StateMachineManager`` is typically running an active 
flow on its server thread until it encounters a blocking, or 
externally visible operation, such as sending a message, waiting for a 
message, or initiating a ``subFlow``. The fiber is then suspended 
and its stack frames serialized to the database, thus ensuring that if 
the node is stopped, or crashes at this point the flow will restart 
with exactly the same action again. To further ensure consistency, every 
event which resumes a flow opens a database transaction, which is 
committed during this suspension process ensuring that the database 
modifications e.g. state commits stay in sync with the mutating changes 
of the flow. Having recorded the fiber state the 
``StateMachineManager`` then carries out the network actions as required 
(internally one flow message exchanged may actually involve several 
physical session messages to authenticate and invoke registered 
flows on the remote nodes). The flow will stay suspended until 
the required message is returned and the scheduler will resume 
processing of other activated flows. On receipt of the expected 
response message from the network layer the ``StateMachineManager`` 
locates the appropriate flow, resuming it immediately after the 
blocking step with the received message. Thus from the perspective of 
the flow the code executes as a simple linear progression of 
processing, even if there were node restarts and possibly message 
resends (the messaging layer deduplicates messages based on an id that 
is part of the checkpoint).

``StateMachineManager`` 通常会在它的 server 线程上运行一个 flow 直到遇到一个障碍，或者一个外部可见的操作，比如发送一个消息，等待一个消息或者初始一个 ``subflow``。接下来 fiber 会被挂起，并且它的 stack frames 会被序列化到数据库中，因此确保在节点停止运行或者崩溃的时候，flow 能够完全按照原来要执行的动作来重新启动。为了进一步确保一致性，恢复一个 flow 的每个事件都会打开一个数据库事务，这个事务会在该挂起流程中被提交，来确保对数据库所做的修改（比如state 的提交）始终能够跟 flow 的变化保持同步。有了记录下来的 fiber state，``StateMachineManager`` 会按照需求进行网络活动（内部的一个 flow 信息交换实际上可能要涉及多个物理的对话消息来在远程节点上认证和调用注册的 flows）。flow 会一直保持挂起的状态直到所需的消息返回来，然后 scheduler 会重新启动其他的有效的 flows。当收到来自网络层的期望的反馈信息后，``StateMachineManager`` 会加载相关的 flow，在阻塞的步骤之后带着接收到的消息立即重新启动。因此从 flow 的角度来说，代码作为一个简单流程的线性累加而被执行，即使会有节点的重启和可能出现的重发消息（消息层会基于 checkpoint 的一个 id 来去重）。

The ``StateMachineManager`` service is not directly exposed to the 
flows, or contracts themselves.

``StateMachineManager`` 服务不是直接暴露给 flows 或者 contracts 本身的。

NodeSchedulerService
~~~~~~~~~~~~~~~~~~~~

The ``NodeSchedulerService`` implements the ``SchedulerService`` 
interface and monitors the Vault updates to track any new states that 
implement the ``SchedulableState`` interface and require automatic 
scheduled flow initiation. At the scheduled due time the 
``NodeSchedulerService`` will create a new flow instance passing it 
a reference to the state that triggered the event. The flow can then 
begin whatever action is required. Note that the scheduled activity 
occurs in all nodes holding the state in their Vault, it may therefore 
be required for the flow to exit early if the current node is not 
the intended initiator.

``NodeSchedulerService`` 实现了 ``SchedulerService`` 接口并且会监控 Vault 的更新，以此来跟踪任何的实现了 ``SchedulableState`` 接口的新的 states，并且会请求自动 scheduled flow 的初始化。在预定的结束时间， ``NodeSchedulerService`` 将会创建一个新的 flow 实例，并将触发这次事件的 state 的引用传递给它。这个 flow 接下来可以开始任何所需的动作。注意：这个预约的活动会在所有在他们的 Vault 中存有该 state 的节点上发生， 那么如果该节点不是预期的初始者的话，这可能就会要求这个 flow 会尽早地结束。

Vault 相关的服务
----------------------

NodeVaultService
~~~~~~~~~~~~~~~~

The ``NodeVaultService`` implements the ``VaultService`` interface to 
allow access to the node's own set of unconsumed states. The service 
does this by tracking update notifications from the 
``TransactionStorage`` service and processing relevant updates to delete 
consumed states and insert new states. The resulting update is then 
persisted to the database. The ``VaultService`` then exposes query and 
event notification APIs to flows and CorDapp services to allow them
to respond to updates, or query for states meeting various conditions to 
begin the formation of new transactions consuming them. The equivalent 
services are also forwarded to RPC clients, so that they may show 
updating views of states held by the node.

``NodeVaultService`` 实现了 ``VaultService`` 接口来允许访问节点自己的一系列未被消费的 states。这个服务是通过跟踪从 ``TransactionStorage`` 服务传回的更新通知，然后会进行相关的更新来删除消费掉的 states 并插入新的 states 的方法实现这样的功能的。最终的更新会被持久化到数据库中。``VaultService`` 向 flows 和 CorDapp 服务暴露了查询和事件通知的 APIs，以此来允许他们对更新做出反馈，或者根据不同的查询条件查询 states，来形成新的消费他们的 transactions。同样的服务也会被转发给 RPC 客户端，所以他们会显示该节点所持有的 states 的 updating views。

NodeSchemaService 和 HibernateObserver
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``HibernateObserver`` runs within the node framework and listens for 
vault state updates, the ``HibernateObserver`` then uses the mapping 
services of the ``NodeSchemaService`` to record the states in auxiliary 
database tables. This allows Corda state updates to be exposed to 
external legacy systems by insertion of unpacked data into existing 
tables. To enable these features the contract state must implement the 
``QueryableState`` interface to define the mappings.

``HibernateObserver`` 在节点的 framework 内运行并且会监听 vault state 的更新，然后 ``HibernateObserver`` 会使用 ``NodeSchemaService`` 的 mapping service 来将 states 记录到辅助的数据库表中。这个允许 Corda state 更新通过将未打包的数据插入到已经存在的数据库的表的方式被暴露给外部的系统。要想启用这些功能，contract state 必须要实现 ``QueryableState`` 接口来定义这些 mappings。

Corda Web Server
----------------

A simple web server is provided that embeds the Jetty servlet container.
The Corda web server is not meant to be used for real, production-quality
web apps. Instead it shows one example way of using Corda RPC in web apps
to provide a REST API on top of the Corda native RPC mechanism.

Corda 提供了一个简单的 web server，它内嵌了一个 Jetty servlet 容器。Corda web server 并不意味着适用于真是的生产环境的 web apps。相反的，它只是展示了如何在 web apps 中基于 Corda 自身的 RPC 机制，使用 Corda RPC 来提供一个 REST API。

.. note:: The Corda web server may be removed in future and replaced with
   sample specific webapps using a standard framework like Spring Boot.

.. note:: Corda web server 可能会在未来被移除，并且会使用一个标准的 framework 比如 Spring Boot 的一个例子 webapp 来替代。
节点
=====

.. topic:: 概要

   * *A node is JVM run-time with a unique network identity running the Corda software*
   * *The node has two interfaces with the outside world:*

      * *A network layer, for interacting with other nodes*
      * *RPC, for interacting with the node's owner*

   * *The node's functionality is extended by installing CorDapps in the plugin registry*

   * *一个节点是指运行着 Corda 软件的具有唯一标识的一个 JVM 运行时*
   * *节点对于外部世界包含两个接口：*

      * *网络层，用来同其他的节点通信*
      * *RPC，为了跟节点的所有者通信*

   * *节点的功能是通过在 plugin registry 里安装 CorDapps 方式来扩展的*

.. only:: htmlmode

   Video
   -----
   .. raw:: html

       <p><a href="https://vimeo.com/214168860">Corda Node, CorDapps and Network</a></p>
       <iframe src="https://player.vimeo.com/video/214168860" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
       <p></p>


节点架构
-----------------
A Corda node is a JVM run-time environment with a unique identity on the network that hosts Corda services and
CorDapps.

Corda 中的节点指的是在网络中具有唯一标识的运行着 Corda 服务和 CorDapps 的 JVM 运行时环境。

We can visualize the node's internal architecture as follows:

下边是节点的内部架构图：

.. image:: resources/node-architecture.png
   :scale: 25%
   :align: center

The core elements of the architecture are:

* A persistence layer for storing data
* A network interface for interacting with other nodes
* An RPC interface for interacting with the node's owner
* A service hub for allowing the node's flows to call upon the node's other services
* A cordapp interface and provider for extending the node by installing CorDapps

架构中的核心元素包括：

* 存储数据的持久化层
* 同其他节点沟通的网络接口
* 同节点的所有者进行沟通的 RPC 接口
* 允许节点的 flows 来调用节点其他服务的 service hub
* plugin registry 用来通过安装 CorDapps 来扩展节点

持久层
-----------------
The persistence layer has two parts:

* The **vault**, where the node stores any relevant current and historic states
* The **storage service**, where it stores transactions, attachments and flow checkpoints

持久层包含两部分：

* **Vault**，节点用来存储相关的当前和历史的 states 数据
* **存储服务**，用来存储 transaction, attachment 和 flow checkpoints

The node's owner can query the node's storage using the RPC interface (see below).

节点的所有者可以通过使用 RPC 接口来查询节点的 storage。

网络接口
-----------------
All communication with other nodes on the network is handled by the node itself, as part of running a flow. The
node's owner does not interact with other network nodes directly.

同网络中的其他节点进行沟通是节点自己来处理的，作为运行一个 flow 的一部分。节点的所有者不会直接地同网络中其他的节点进行交互。

RPC 接口
-------------
The node's owner interacts with the node via remote procedure calls (RPC). The key RPC operations the node exposes
are documented in :doc:`api-rpc`.

节点的所有者是通过使用 Remote Procedure Calls(RPC) 来跟节点进行交互的。关键的节点暴露的 RPC 操作可以查看 :doc:`api-rpc`。

The service hub
---------------
Internally, the node has access to a rich set of services that are used during flow execution to coordinate ledger
updates. The key services provided are:

* Information on other nodes on the network and the services they offer
* Access to the contents of the vault and the storage service
* Access to, and generation of, the node's public-private keypairs
* Information about the node itself
* The current time, as tracked by the node

在节点内部，节点可以在 flow 的执行过程中访问丰富的服务来协助更新账本。主要的服务包括：

* 网络中的其他节点以及提供什么服务的信息
* 访问 vault 和存储服务的内容
* 访问和生成节点的公钥私钥对
* 节点本身的信息
* 节点追踪的，当前的时间

CorDapp 提供者
--------------------
The CorDapp provider is where new CorDapps are installed to extend the behavior of the node.

CorDapp 提供者是新的 CorDapps 被安装的地方，来扩展节点的行为。

The node also has several CorDapps installed by default to handle common tasks such as:

* Retrieving transactions and attachments from counterparties
* Upgrading contracts
* Broadcasting agreed ledger updates for recording by counterparties

节点默认会安装一些 CorDapps 来处理一些常见的任务，比如：

* 从合作方那边获得交易和附件信息
* 更新合约
* 向交易其他放广播同意的账本更新信息

.. _draining-mode:

排空节点模式
-------------

In order to operate a clean shutdown of a node, it is important than no flows are in-flight, meaning no checkpoints should
be persisted. The node is able to be put in draining mode, during which:

* Commands requiring to start new flows through RPC will be rejected.
* Scheduled flows due will be ignored.
* Initial P2P session messages will not be processed, meaning peers will not be able to initiate new flows involving the node.
* All other activities will proceed as usual, ensuring that the number of in-flight flows will strictly diminish.

为了执行一次干净的关闭节点操作，没有正在执行的 flows 非常重要，也就是说应该没有任何的 checkpoints 被持久化。节点能够被设置为排空状态，在这个状态中：

* 通过 RPC 要求的启动新的 flows 的命令会被拒绝
* 预约的 flows 会被忽略
* 初始化 P2P 的会话消息将不会被处理，意味着 peers 将不能够初始化新的 flows
* 其他所有的活动还会照常进行，来确保正在执行的 flows 的数量在不断减少。

As their number - which can be monitored through RPC - reaches zero, it is safe to shut the node down.
This property is durable, meaning that restarting the node will not reset it to its default value and that a RPC command is required.

对于他们的数量 - 可以通过 RPC 来进行监控 - 达到0，那么就是安全的了，可以进行关闭节点的操作了。这个属性是持久的，也就是说重新启动这个节点也不会重置这个值到默认和值，并且需要一个 RPC 命令。

The node can be safely shut down via a drain using the shell.

节点可以使用 shell 来被排空然后安全地关闭。
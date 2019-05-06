网络地图
===============

.. contents::

The network map is a collection of signed ``NodeInfo`` objects. Each NodeInfo is signed by the node it represents and
thus cannot be tampered with. It forms the set of reachable nodes in a compatibility zone. A node can receive these
objects from two sources:

1. A network map server that speaks a simple HTTP based protocol.
2. The ``additional-node-infos`` directory within the node's directory.

Corda 中的网络地图是被签名的 ``NodeInfo`` 对象的集合。每个 NodeInfo 会被它所表示的节点进行加密签名，因此是不可篡改的。它形成了在一个兼容区域（compatibility zone）中的一系列的可交互的节点。一个节点可以通过以下两种来源来获得这些对象：

1. 网络地图服务器通过基于 HTTP 协议进行传输
2. 在节点路径下的 ``additional-node-infos`` 目录中

The network map server also distributes the parameters file that define values for various settings that all nodes need
to agree on to remain in sync.

网络地图服务器也会分发一个包含网络参数的文件，这个网络参数文件定义了很多配置的参数值，并且所有的节点需要同意这些配置参数才能够在该网络中同步数据。

.. note:: In Corda 3 no implementation of the HTTP network map server is provided. This is because the details of how
   a compatibility zone manages its membership (the databases, ticketing workflows, HSM hardware etc) is expected to vary
   between operators, so we provide a simple REST based protocol for uploading/downloading NodeInfos and managing
   network parameters. A future version of Corda may provide a simple "stub" implementation for running test zones.
   In Corda 3 the right way to run a test network is through distribution of the relevant files via your own mechanisms.
   We provide a tool to automate the bulk of this task (see below).

.. note:: 在 Corda 3 并没有提供一个 HTTP 网络地图服务器的实现。这是因为一个兼容区域要如何去管理它的成员在不同的操作者之间存在着很大的差异（数据库，问题处理流程，HSM 硬件等），所以我们提供了一个简单的基于 REST 的协议来上传/下载 NodeInfos 和管理网络参数。Corda 未来的版本可能会提供一个简单的 “stub” 的实现来运行测试区域（test zones）。在 Corda 3 的版本中，正确的运行一个测试网络的方式是使用你自己的机制来将相关的文件在不同节点间分发。我们提供了一个工具来自动进行这些任务（下边有详细的描述）。

HTTP 网络地图协议
-------------------------

If the node is configured with the ``compatibilityZoneURL`` config then it first uploads its own signed ``NodeInfo``
to the server at that URL (and each time it changes on startup) and then proceeds to download the entire network map from 
the same server. The network map consists of a list of ``NodeInfo`` hashes. The node periodically polls for the network map 
(based on the HTTP cache expiry header) and any new entries are downloaded and cached. Entries which no longer exist are deleted from the node's cache.

如果节点的配置中配置了 ``compatibilityZoneURL`` 的话，那么当它首先会将自己签名过的 ``NodeInfo`` 文件上传到服务器上（之后每次启动的时候如果 ``NodeInfo`` 有变化的话，也会重新上传），然后会下载整个网络地图。网络地图包含了一个 NodeInfo 哈希的列表。节点会定期地重新获取网络地图（基于 HTTP 缓存的过期 header 设置），新的节点信息会被下载并且加入到缓存中。已经不存在的节点信息会从节点的缓存中被删除。

The set of REST end-points for the network map service are as follows.

网络地图服务的 REST end-points 包括：

+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| Request method | Path                                    | Description                                                                                                                                  |
+================+=========================================+==============================================================================================================================================+
| POST           | /network-map/publish                    | For the node to upload its signed ``NodeInfo`` object to the network map.                                                                    |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| POST           | /network-map/ack-parameters             | For the node operator to acknowledge network map that new parameters were accepted for future update.                                        |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map                            | Retrieve the current signed public network map object. The entire object is signed with the network map certificate which is also attached.  |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/{uuid}                     | Retrieve the current signed private network map object with given uuid. Format is the same as for ``/network-map`` endpoint.                 |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/node-info/{hash}           | Retrieve a signed ``NodeInfo`` as specified in the network map object.                                                                       |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/network-parameters/{hash}  | Retrieve the signed network parameters (see below). The entire object is signed with the network map certificate which is also attached.     |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/my-hostname                | Retrieve the IP address of the caller (and **not** of the network map).                                                                      |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+

Network maps hosted by R3 or other parties using R3's commercial network management tools typically also provide the following endpoints as a convenience to operators and other users

由 R3 或者其他使用 R3 的商业网络管理工具 host 的网络地图通常也会提供下边的 endpoints 来为维护者和其他用户提供方便

.. note:: we include them here as they can aid debugging but, for the avoidance of doubt, they are not a formal part of the spec and the node will operate even in their absence.

.. note:: 我们包含了他们是因为他们能够帮助 debugging，但是为了避免疑问，他们通常不是说明中常规的部分，并且节点即使不存在的话也能够工作。

+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| Request method | Path                                    | Description                                                                                                                                  |
+================+=========================================+==============================================================================================================================================+
| GET            | /network-map/json                       | Retrieve the current public network map formatted as a JSON document.                                                                        |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/json/{uuid}                | Retrieve the current network map for a private network indicated by the uuid parameter formatted as a JSON document.                         |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/json/node-infos            | Retrieve a human readable list of the currently registered ``NodeInfo`` files in the public network formatted as a JSON document.            |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/json/node-infos/{uid}      | Retrieve a human readable list of the currently registered ``NodeInfo`` files in the specified private network map.                          |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| GET            | /network-map/json/node-info/{hash}      | Retrieve a human readable version of a ``NodeInfo`` formatted as a JSON document.                                                            |
+----------------+-----------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+


HTTP is used for the network map service instead of Corda's own AMQP based peer to peer messaging protocol to
enable the server to be placed behind caching content delivery networks like Cloudflare, Akamai, Amazon Cloudfront and so on.
By using industrial HTTP cache networks the map server can be shielded from DoS attacks more effectively. Additionally,
for the case of distributing small files that rarely change, HTTP is a well understood and optimised protocol. Corda's
own protocol is designed for complex multi-way conversations between authenticated identities using signed binary
messages separated into parallel and nested flows, which isn't necessary for network map distribution.

``additional-node-infos`` 目录
---------------------------------------

Alongside the HTTP network map service, or as a replacement if the node isn't connected to one, the node polls the
contents of the ``additional-node-infos`` directory located in its base directory. Each file is expected to be the same
signed ``NodeInfo`` object that the network map service vends. These are automatically added to the node's cache and can
be used to supplement or replace the HTTP network map. If the same node is advertised through both mechanisms then the
latest one is taken.

跟 HTTP 网络地图服务一起，或者说是它的一种替代方案，当节点并没有连接到一个网络地图服务的话，节点会从自己根目录下的 ``additional-node-infos`` 路径下读取网络地图内容。该路径下的每个文件应该同网络地图服务签名加密的 ``NodeInfo`` 文件相同。这些会被自动地加到节点的网络地图缓存中并可以被用来补充或者替换 HTTP 网络地图。如果同一个节点同时在这两种机制中存在，那么 additional-node-infos 中的网络地图会最终被使用。

On startup the node generates its own signed node info file, filename of the format ``nodeInfo-${hash}``. It can also be
generated using the ``generate-node-info`` sub-command without starting the node. To create a simple network
without the HTTP network map service simply place this file in the ``additional-node-infos`` directory of every node that's
part of this network. For example, a simple way to do this is to use rsync.

当节点启动的时候，会生成自己签名过的 node info 文件，文件名的格式为 ``nodeInfo-${hash}```。也可以使用 ``generate-node-info`` 命令行标识不启动节点也可以生成 node info 文件。如果不想使用 HTTP 网络地图服务来创建一个简单的网络的话，简单地将这些文件放在网络中每个节点的 ``additional-node-infos`` 路径下即可。比如一个简单的方式来做这个工作就是使用 rsync。

Usually, test networks have a structure that is known ahead of time. For the creation of such networks we provide a
``network-bootstrapper`` tool. This tool pre-generates node configuration directories if given the IP addresses/domain
names of each machine in the network. The generated node directories contain the NodeInfos for every other node on
the network, along with the network parameters file and identity certificates. Generated nodes do not need to all be
online at once - an offline node that isn't being interacted with doesn't impact the network in any way. So a test
cluster generated like this can be sized for the maximum size you may need, and then scaled up and down as necessary.

通常，测试网络事先都会知道应该具有怎样的一个架构。对于创建这种网络，我们提供了一个 ``network-bootstrapper`` 工具。如果给定了网络中每个节点的 IP 地址/域名的话，这个工具能够生成节点的配置文件路径。生成的节点配置路径中包含了网络中其他节点的 NodeInfos 文件，同时也会包含网络参数文件和身份证书（identity certificates）。生成的节点不需要同时一直保持在线 - 一个不需要进行互动的离线节点不会影响该网络的运行。所以一个这样生成的测试集群可以按照你的需要扩展到最大的 size，然后可以按照需要进行扩大或缩小。

More information can be found in :doc:`network-bootstrapper`.

更多信息可以在 :doc:`network-bootstrapper` 中找到。

网络参数
------------------

Network parameters are a set of values that every node participating in the zone needs to agree on and use to
correctly interoperate with each other. They can be thought of as an encapsulation of all aspects of a Corda deployment
on which reasonable people may disagree. Whilst other blockchain/DLT systems typically require a source code fork to
alter various constants (like the total number of coins in a cryptocurrency, port numbers to use etc), in Corda we
have refactored these sorts of decisions out into a separate file and allow "zone operators" to make decisions about
them. The operator signs a data structure that contains the values and they are distributed along with the network map.
Tools are provided to gain user opt-in consent to a new version of the parameters and ensure everyone switches to them
at the same time.

网络参数是一系列的值，在该网络中的所有节点必须要同意这些参数值才能够彼此正确地进行互通。这个可以被理解为是一个 Corda 部署的所有方面的一个封装，有些节点可能会不同意某些方面。然而其他的区块链/DLT 系统通常会要求一个源代码的分叉（fork）来变更不同的约束（比如在一个加密货币中货币的总量，应该使用的端口号等），在 Corda 中，我们对这些选择进行了改进并将它们放进了一个额外的文件中并且允许 “区域操作者”（zone operators）来对他们进行选择。区域操作者会对包含这些参数的一个数据结构进行签名，然后这个数据结构会同网络地图一起被分发出去。工具被提供用来获得用户对一个新版本的参数的同意，并且确保每个人都会在同一时间切换到新的网络参数。

If the node is using the HTTP network map service then on first startup it will download the signed network parameters,
cache it in a ``network-parameters`` file and apply them on the node.

如果节点使用的是 HTTP 网络地图服务的话，那么在第一次启动的时候它会下载被签名过的网络参数文件，把它缓存在一个 ``network-parameters`` 文件中并且将它应用到了该节点上。

.. warning:: If the ``network-parameters`` file is changed and no longer matches what the network map service is advertising
  then the node will automatically shutdown. Resolution to this is to delete the incorrect file and restart the node so
  that the parameters can be downloaded again.

.. warning:: 如果 ``network-parameters`` 文件被改动了并且同网络地图服务发布的版本不一样了的话，节点就会自动关闭。解决这个问题的方法就是删除这个不正确的文件然后重启节点，所以网络参数就可以被再次地下载。

If the node isn't using a HTTP network map service then it's expected the signed file is provided by some other means.
For such a scenario there is the network bootstrapper tool which in addition to generating the network parameters file
also distributes the node info files to the node directories.

如果节点并没有使用 HTTP 网络地图服务的话，那么这些被签名的文件应该通过另外一种方式被获取。针对于这种情况，这里有一个网络启动器（network bootstrapper）的工具可以用来生成网络参数文件并且连同 node info 文件一同被分发到节点的相关路径下。

The current set of network parameters:

:minimumPlatformVersion: The minimum platform version that the nodes must be running. Any node which is below this will
        not start.

:notaries: List of identity and validation type (either validating or non-validating) of the notaries which are permitted
        in the compatibility zone.

:maxMessageSize: Maximum allowed size in bytes of an individual message sent over the wire. Note that attachments are
            a special case and may be fragmented for streaming transfer, however, an individual transaction or flow message
            may not be larger than this value.

:maxTransactionSize: Maximum allowed size in bytes of a transaction. This is the size of the transaction object and its attachments.

:modifiedTime: The time when the network parameters were last modified by the compatibility zone operator.

:epoch: Version number of the network parameters. Starting from 1, this will always increment whenever any of the
        parameters change.
        
:whitelistedContractImplementations: List of whitelisted versions of contract code.
        For each contract class there is a list of SHA-256 hashes of the approved CorDapp jar versions containing that contract.
        Read more about *Zone constraints* here :doc:`api-contract-constraints`

:eventHorizon: Time after which nodes are considered to be unresponsive and removed from network map. Nodes republish their
        ``NodeInfo`` on a regular interval. Network map treats that as a heartbeat from the node.

:packageOwnership: List of the network-wide java packages that were successfully claimed by their owners.
    Any CorDapp JAR that offers contracts and states in any of these packages must be signed by the owner.
    This ensures that when a node encounters an owned contract it can uniquely identify it and knows that all other nodes can do the same.
    Encountering an owned contract in a JAR that is not signed by the rightful owner is most likely a sign of malicious behaviour, and should be reported.
    The transaction verification logic will throw an exception when this happens.
    Read more about *Package ownership* here :doc:`design/data-model-upgrades/package-namespace-ownership`.

当前的网络参数配置项包括：

:minimumPlatformVersion: 节点运行所需的最小平台版本号。任何运行小于该版本号的节点将不会被启动。

:notaries: 在 compatibility zone 中允许的 notaries 的身份和验证类别（validating 或者是 non-validating）的列表。

:maxMessageSize: Maximum allowed size in bytes of an individual message sent over the wire. Note that attachments are
            a special case and may be fragmented for streaming transfer, however, an individual transaction or flow message
            may not be larger than this value.

:maxTransactionSize: 对于一个 transaction 所允许的最大 size。这个尺寸是指 transaction 对象和它的附件共同的大小。

:modifiedTime: 网络参数被 compatibility zone 操作者最后一个更新的时间。

:epoch: 网络参数的版本号。从1开始，当有任何的参数变化的时候，这个版本号会自动增加。

:whitelistedContractImplementations: 被添加到白名单中的合约代码（contract code）的列表。对于每一个合约的类，这里会有一个经过批准的包含该合约类的 CorDapp jar 的 SHA-256 hashes 的列表。在这里 :doc:`api-contract-constraints` 阅读更多关于 *Zone constraints* 的信息。

:eventHorizon: 代表经过多久之后节点会被认为是没有反应的并会被移除出网络地图。节点可以在固定的一个周期中再次发布他们的 ``NodeInfo``。网络地图会把这种定期的操作看作是来自于节点的心跳。

:packageOwnership: List of the network-wide java packages that were successfully claimed by their owners.
    Any CorDapp JAR that offers contracts and states in any of these packages must be signed by the owner.
    This ensures that when a node encounters an owned contract it can uniquely identify it and knows that all other nodes can do the same.
    Encountering an owned contract in a JAR that is not signed by the rightful owner is most likely a sign of malicious behaviour, and should be reported.
    The transaction verification logic will throw an exception when this happens.
    Read more about *Package ownership* here :doc:`design/data-model-upgrades/package-namespace-ownership`.

More parameters will be added in future releases to regulate things like allowed port numbers, whether or not IPv6
connectivity is required for zone members, required cryptographic algorithms and roll-out schedules (e.g. for moving to post quantum cryptography), parameters related to SGX and so on.

更过的参数会在将来被添加，来规定像被允许的端口号、在一个节点在清除之前他可以保持离线多久、对于 zone 成员是不是必须要求 IPv6 的连接、需要的加密算法和推出的时间安排（比如，换成了 post quantum 加密）、有关 SGX 的参数等等。

Network parameters update process
---------------------------------

Network parameters are controlled by the zone operator of the Corda network that you are a member of. Occasionally, they may need to change
these parameters. There are many reasons that can lead to this decision: adding a notary, setting new fields that were added to enable
smooth network interoperability, or a change of the existing compatibility constants is required, for example.

.. note:: A future release may support the notion of phased roll-out of network parameter changes.

Updating of the parameters by the zone operator is done in two phases:
1. Advertise the proposed network parameter update to the entire network.
2. Switching the network onto the new parameters - also known as a `flag day`.

The proposed parameter update will include, along with the new parameters, a human-readable description of the changes as well as the
deadline for accepting the update. The acceptance deadline marks the date and time that the zone operator intends to switch the entire
network onto the new parameters. This will be a reasonable amount of time in the future, giving the node operators time to inspect,
discuss and accept the parameters.

The fact a new set of parameters is being advertised shows up in the node logs with the message
"Downloaded new network parameters", and programs connected via RPC can receive ``ParametersUpdateInfo`` by using
the ``CordaRPCOps.networkParametersFeed`` method. Typically a zone operator would also email node operators to let them
know about the details of the impending change, along with the justification, how to object, deadlines and so on.

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/messaging/CordaRPCOps.kt
    :language: kotlin
    :start-after: DOCSTART 1
    :end-before: DOCEND 1

Auto Acceptance
```````````````

If the only changes between the current and new parameters are for auto-acceptable parameters then, unless configured otherwise, the new
parameters will be accepted without user input. The following parameters with the ``@AutoAcceptable`` annotation are auto-acceptable:

.. literalinclude:: ../../core/src/main/kotlin/net/corda/core/node/NetworkParameters.kt
    :language: kotlin
    :start-after: DOCSTART 1
    :end-before: DOCEND 1

This behaviour can be turned off by setting the optional node configuration property ``NetworkParameterAcceptanceSettings.autoAcceptEnabled``
to ``false``. For example:

.. sourcecode:: guess

    ...
    NetworkParameterAcceptanceSettings {
        autoAcceptEnabled = false
    }
    ...

It is also possible to switch off this behaviour at a more granular parameter level. This can be achieved by specifying the set of
``@AutoAcceptable`` parameters that should not be auto-acceptable in the optional
``NetworkParameterAcceptanceSettings.excludedAutoAcceptableParameters`` node configuration property.

For example, auto-acceptance can be switched off for any updates that change the ``packageOwnership`` map by adding the following to the
node configuration:

.. sourcecode:: guess

    ...
    NetworkParameterAcceptanceSettings {
        excludedAutoAcceptableParameters: ["packageOwnership"]
    }
    ...

Manual Acceptance
`````````````````

If the auto-acceptance behaviour is turned off via the configuration or the network parameters change involves parameters that are
not auto-acceptable then manual approval is required.

In this case the node administrator can review the change and decide if they are going to accept it. The approval should be done
before the update Deadline. Nodes that don't approve before the deadline will likely be removed from the network map by
the zone operator, but that is a decision that is left to the operator's discretion. For example the operator might also
choose to change the deadline instead.

If the network operator starts advertising a different set of new parameters then that new set overrides the previous set.
Only the latest update can be accepted.

To send back parameters approval to the zone operator, the RPC method ``fun acceptNewNetworkParameters(parametersHash: SecureHash)``
has to be called with ``parametersHash`` from the update. Note that approval cannot be undone. You can do this via the Corda
shell (see :doc:`shell`):

``run acceptNewNetworkParameters parametersHash: "ba19fc1b9e9c1c7cbea712efda5f78b53ae4e5d123c89d02c9da44ec50e9c17d"``

If the administrator does not accept the update then next time the node polls network map after the deadline, the
advertised network parameters will be the updated ones. The previous set of parameters will no longer be valid.
At this point the node will automatically shutdown and will require the node operator to bring it back again.

Cleaning the network map cache
------------------------------

Sometimes it may happen that the node ends up with an inconsistent view of the network. This can occur due to changes in deployment
leading to stale data in the database, different data distribution time and mistakes in configuration. For these unlikely
events both RPC method and command line option for clearing local network map cache database exist. To use them
you either need to run from the command line:

.. code-block:: shell

    java -jar corda.jar clear-network-cache

or call RPC method `clearNetworkMapCache` (it can be invoked through the node's shell as `run clearNetworkMapCache`, for more information on
how to log into node's shell see :doc:`shell`). As we are testing and hardening the implementation this step shouldn't be required.
After cleaning the cache, network map data is restored on the next poll from the server or filesystem.

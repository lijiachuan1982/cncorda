API: RPC 操作
===================
The node's owner interacts with the node solely via remote procedure calls (RPC). The node's owner does not have
access to the node's ``ServiceHub``.

节点的 owner 跟节点进行交互的方式是使用 remote procedure calls（RPC）。节点的 owner 没有访问节点的 ``ServiceHub`` 的权限。

The key RPC operations exposed by the node are:

* ``CordaRPCOps.vaultQueryBy``
    * Extract states from the node's vault based on a query criteria
* ``CordaRPCOps.vaultTrackBy``
    * As above, but also returns an observable of future states matching the query
* ``CordaRPCOps.networkMapFeed``
    * A list of network nodes, and an observable of changes to the network map
* ``CordaRPCOps.registeredFlows``
    * See a list of registered flows on the node
* ``CordaRPCOps.startFlowDynamic``
    * Start one of the node's registered flows
* ``CordaRPCOps.startTrackedFlowDynamic``
    * As above, but also returns a progress handle for the flow
* ``CordaRPCOps.nodeInfo``
    * Returns information about the node
* ``CordaRPCOps.currentNodeTime``
    * Returns the current time according to the node's clock
* ``CordaRPCOps.partyFromKey/CordaRPCOps.wellKnownPartyFromX500Name``
    * Retrieves a party on the network based on a public key or X500 name
* ``CordaRPCOps.uploadAttachment``/``CordaRPCOps.openAttachment``/``CordaRPCOps.attachmentExists``
    * Uploads, opens and checks for the existence of attachments

主要的 RPC 操作包括：

* ``CordaRPCOps.vaultQueryBy``：基于查询条件来从节点的账本中获取 states
* ``CordaRPCOps.vaultTrackBy``：像上边那样，不同的时候同时还会返回满足查询条件的有关未来的states 的观察者（observable）
* ``CordaRPCOps.networkMapFeed``：一个网络节点的列表，和一个关于 network map 的变动的观察者（observable）
* ``CordaRPCOps.registeredFlows``：查看节点的注册测 flows 列表
* ``CordaRPCOps.startFlowDynamic``：开始一个在节点注册的 flow
* ``CordaRPCOps.startTrackedFlowDynamic``：像上边那样，不同的是还会同时返回一个对于 flow 的 progress handler
* ``CordaRPCOps.nodeInfo``：返回关于节点的信息
* ``CordaRPCOps.currentNodeTime``：返回节点对应的当前时间
* ``CordaRPCOps.partyFromKey``/``CordaRPCOps.wellKnownPartyFromX500Name``：根据公钥（public key）或者 X500 名字来获取网络中的某个节点信息
* ``CordaRPCOps.uploadAttachment``/``CordaRPCOps.openAttachment``/``CordaRPCOps.attachmentExists``：上传，打开和检查一个附件是否存在
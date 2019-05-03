API: ServiceHub
===============
Within ``FlowLogic.call``, the flow developer has access to the node's ``ServiceHub``, which provides access to the
various services the node provides. The services offered by the ``ServiceHub`` are split into the following categories:

* ``ServiceHub.networkMapCache``
    * Provides information on other nodes on the network (e.g. notaries…)
* ``ServiceHub.identityService``
    * Allows you to resolve anonymous identities to well-known identities if you have the required certificates
* ``ServiceHub.attachments``
    * Gives you access to the node's attachments
* ``ServiceHub.validatedTransactions``
    * Gives you access to the transactions stored in the node
* ``ServiceHub.vaultService``
    * Stores the node’s current and historic states
* ``ServiceHub.keyManagementService``
    * Manages signing transactions and generating fresh public keys
* ``ServiceHub.myInfo``
    * Other information about the node
* ``ServiceHub.clock``
    * Provides access to the node’s internal time and date

在 ``FlowLogic.call`` 中，flow 开发者能够访问节点的 ``ServiceHub``，其提供了访问很多节点提供的服务。``ServiceHub`` 提供的服务包括以下的类别：

* ``ServiceHub.networkMapCache``
    * 提供了网络中其他节点的信息（比如 notaries）
* ``ServiceHub.identityService``
    * 如果你有所需的认证信息，允许你将匿名的 identities 变为 well-known 的 identities
* ``ServiceHub.attachments``
    * 给你访问节点附件的权限
* ``ServiceHub.validatedTransactions``
    * 给你访问节点存储的 transactions 的权限
* ``ServiceHub.vaultService``
    * 存储了节点的当前和历史的 states
* ``ServiceHub.keyManagementService``
    * 管理签名 transactions 和生成新的公钥
* ``ServiceHub.myInfor``
    * 关于节点的其他信息
* ``ServiceHub.clock``
    * 提供访问节点的内部时间和日期

Additional, ``ServiceHub`` exposes the following properties:

* ``ServiceHub.loadState`` and ``ServiceHub.toStateAndRef`` to resolve a ``StateRef`` into a ``TransactionState`` or
  a ``StateAndRef``
* ``ServiceHub.signInitialTransaction`` to sign a ``TransactionBuilder`` and convert it into a ``SignedTransaction``
* ``ServiceHub.createSignature`` and ``ServiceHub.addSignature`` to create and add signatures to a ``SignedTransaction``

另外，``ServiceHub`` 暴露了以下的属性：

* ``ServiceHub.loadState`` 和 ``ServiceHub.toStateAndRef`` 来将一个 ``StateRef`` 变成一个 ``TransactionState`` 或者一个 ``StateAndRef``
* ``ServiceHub.signInitialTransaction`` 用来给一个 ``TransactionBuilder`` 签名并且将其转换成一个 ``SignedTransaction``
* ``ServiceHub.createSignature`` 和 ``ServiceHub.addSignature`` 用来向 ``SignedTransaction`` 创建和添加签名
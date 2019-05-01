Notaries
========

.. topic:: 概要

   * *Notary clusters prevent "double-spends"*
   * *Notary clusters are also time-stamping authorities. If a transaction includes a time-window, it can only be notarised during that window*
   * *Notary clusters may optionally also validate transactions, in which case they are called "validating" notaries, as opposed to "non-validating"*
   * *A network can have several notary clusters, each running a different consensus algorithm*

   * *Notary 集群避免 “双花”*
   * *Notary 集群也可以是时间戳授权。如果一笔交易包含一个 time-window，那么它只能在这个 time-window 内被公证*
   * *Notary 集群也可以可选地用来验证交易，在这种情况下他们被称为 “用于验证” 的 notaries，相对于 “非验证” 的 notaries*
   * *一个网络中可以有多个 notaries，每一个 notary 运行一个不同的共识算法*

.. only:: htmlmode

    Video
    -----
    .. raw:: html
    
        <iframe src="https://player.vimeo.com/video/214138458" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
        <p></p>


概览
--------
A *notary cluster* is a network service that provides **uniqueness consensus** by attesting that, for a given
transaction, it has not already signed other transactions that consumes any of the proposed transaction's input states.

一个 *notary 集群* 是一个网络服务，通过证明一个给定的交易的 input 是没有被其他的交易消费过的方式提供了 **唯一性共识**。

Upon being sent asked to notarise a transaction, a notary cluster will either:

* Sign the transaction if it has not already signed other transactions consuming any of the proposed transaction's
  input states
* Reject the transaction and flag that a double-spend attempt has occurred otherwise

当被要求为一笔交易进行公证的时候，一个 notary 集群会进行下边两种操作中的一种：

* 如果对于给定的交易中的 input，没有任何其他的交易已经消费该 input 的时候，会提供签名
* 拒绝这笔交易并且标明产生了双花的情况

In doing so, the notary cluster provides the point of finality in the system. Until the notary cluster's signature is
obtained, parties cannot be sure that an equally valid, but conflicting, transaction will not be regarded as the
"valid" attempt to spend a given input state. However, after the notary cluster's signature is obtained, we can be sure
that the proposed transaction's input states have not already been consumed by a prior transaction. Hence, notarisation
is the point of finality in the system.

通过这样做，notary 集群就在系统中提供了一个终结点。在最终获得 notary 集群的签名之前，交易各方并不能确定交易的有效性。但是当收到了 notary 集群的签名之后，我们可以确认的是，交易中的 Input 是没有被其他任何的交易所消费过的。因此公证（notarisation）在系统里是最后的一步。

Every state has an appointed notary cluster, and a notary cluster will only notarise a transaction if it is the
appointed notary cluster of all the transaction's input states.

每个 state 都会有一个指定的 notary 集群，而且一个 notary 集群也只会去公正那些 input 指定它为 notary 集群的 transaction。

共识算法
--------------------
Corda has "pluggable" consensus, allowing notary clusters to choose a consensus algorithm based on their requirements in
terms of privacy, scalability, legal-system compatibility and algorithmic agility.

Corda 拥有一套 “可插拔 pluggable” 的共识，允许 notary 集群根据不同的需求（私有化、扩展性、法律系统的兼容性和算法的便捷性）来选择一种共识算法。

In particular, notary clusters may differ in terms of:

* **Structure** - a notary cluster may be a single node, several mutually-trusting nodes, or several
  mutually-distrusting nodes
* **Consensus algorithm** - a notary cluster may choose to run a high-speed, high-trust algorithm such as RAFT, a
  low-speed, low-trust algorithm such as BFT, or any other consensus algorithm it chooses

特别的，notary 集群可能含有下边的不同：

* **结构** - 一个 notary 集群可能是一个单独的网络节点，或者是互相信任的节点集群，或者是互不信任的节点集群
* **共识算法** - 一个 notary 集群可能会选择运行一个高速，高信任的算法（比如 RAFT），或者一个低速低信任的算法（比如 BFT），又或者是任何其他的选择的共识算法

.. _key_concepts_notaries_validation:

验证
^^^^^^^^^^
A notary cluster must also decide whether or not to provide **validity consensus** by validating each transaction
before committing it. In making this decision, it faces the following trade-off:

* If a transaction **is not** checked for validity (non-validating notary), it creates the risk of "denial of state" attacks, where a node
  knowingly builds an invalid transaction consuming some set of existing states and sends it to the
  notary cluster, causing the states to be marked as consumed

* If the transaction **is** checked for validity (validating notary), the notary will need to see the full contents of the transaction and
  its dependencies. This leaks potentially private data to the notary cluster

一个 notary 集群还需要选择是否在提交之前通过验证每个 transaction 的有效性来提供这种 **有效性共识** 服务。为了做出这个选择，他们需要面对下边的取舍问题：

* 如果一个 transaction **没有** 被检查正确性（非验证 notary），那么这就增加了 “denial of state” 袭击的风险，指的就是某个节点知道这是一个不正确的 transaction 会消费到一些 states，然后该节点还是把这个 transaction 发送给 notary 集群，但是 notary 如果不进行正确性验证的话，会把这个 state 变为历史记录被消费掉，这显然是不正确的

* 如果 transaction **已经** 被验证了正确与否（验证 notary），notary 需要查看该 transaction 的全部内容以及它的所有依赖。这就向 notary 暴露了一些潜在的隐私数据。

There are several further points to keep in mind when evaluating this trade-off. In the case of the non-validating
model, Corda's controlled data distribution model means that information on unconsumed states is not widely shared.
Additionally, Corda's permissioned network means that the notary cluster can store the identity of the party that
created the "denial of state" transaction, allowing the attack to be resolved off-ledger.

当我们考量这些取舍的时候，有一个后续观点需要始终要考虑的。对于非验证模式，Corda 的控制的数据分布模型意味着未被消费的 states 不会被大面积的共享。另外， Corda 的 permissioned network 也意味着 notary 能够存储造成 “denial of state” transaction 的一方的身份信息，这就允许能够在账本外去解决掉这个袭击。

In the case of the validating model, the use of anonymous, freshly-generated public keys instead of legal identities to
identify parties in a transaction limit the information the notary cluster sees.

对于验证模式，对于匿名的使用，使用新生成的公钥而不是使用法律的标识来标记一笔交易的各方也限制了 notary 集群能够看到的信息。

数据的可视性
^^^^^^^^^^^^^^^

Below is a summary of what specific transaction components have to be revealed to each type of notary:

下边是关于哪些特殊的交易组件必须要暴露给每种类型的 notary 的一个总结：

+-----------------------------------+---------------+-----------------------+
| Transaction components            | Validating    | Non-validating        |
+===================================+===============+=======================+
| Input states                      | Fully visible | References only [1]_  |
+-----------------------------------+---------------+-----------------------+
| Output states                     | Fully visible | Hidden                |
+-----------------------------------+---------------+-----------------------+
| Commands (with signer identities) | Fully visible | Hidden                |
+-----------------------------------+---------------+-----------------------+
| Attachments                       | Fully visible | Hidden                |
+-----------------------------------+---------------+-----------------------+
| Time window                       | Fully visible | Fully visible         |
+-----------------------------------+---------------+-----------------------+
| Notary identity                   | Fully visible | Fully visible         |
+-----------------------------------+---------------+-----------------------+
| Signatures                        | Fully visible | Hidden                |
+-----------------------------------+---------------+-----------------------+

Both types of notaries record the calling party's identity: the public key and the X.500 Distinguished Name.

两种类型的 notaries 都会记录调用方的身份信息：公钥以及 X.500 唯一的名字。

.. [1] A state reference is composed of the issuing transaction's id and the state's position in the outputs. It does not
   reveal what kind of state it is or its contents.

.. [1] 一个 state 的引用是由生成它的 transaction 的 id 和这个 state 在 outputs 中的位置共同构成的。它不会暴露它是哪种类型的 state 或者它的内容这类信息。

多个 Notaries
-----------------
Each Corda network can have multiple notary clusters, each potentially running a different consensus algorithm. This
provides several benefits:

* **Privacy** - we can have both validating and non-validating notary clusters on the same network, each running a
  different algorithm. This allows nodes to choose the preferred notary cluster on a per-transaction basis
* **Load balancing** - spreading the transaction load over multiple notary clusters allows higher transaction
  throughput for the platform overall
* **Low latency** - latency can be minimised by choosing a notary cluster physically closer to the transacting parties

每个 Corda 网络可以存在多个 notary 集群，每个 notary 集群可能会运行一套不同的共识算法。这会带来以下的好处：

* **隐私性** - 我们可以在同一个网络中同时拥有验证和非验证的 notary 集群，每个集群运行着不同的算法。这就允许节点针对每个 transaction 来选择更喜欢的不同的 notary。
* **负载平衡** - 将 transaction 的工作分发给多个 notary 集群可以提高平台整体的交易吞吐量
* **低延迟** - 通过选择物理上离交易方最近的 notary 集群来获得最小化的延迟

更换 notaries
^^^^^^^^^^^^^^^^^
Remember that a notary cluster will only sign a transaction if it is the appointed notary cluster of all of the
transaction's input states. However, there are cases in which we may need to change a state's appointed notary cluster.
These include:

* When a single transaction needs to consume several states that have different appointed notary clusters
* When a node would prefer to use a different notary cluster for a given transaction due to privacy or efficiency
  concerns

一个 notary 集群只有当它是这个 transaction 里的所有 input states 指定的 notary 的情况下才可以提供签名。然而下边的情况可能需要换一个 state 的指定的 notary 集群，包括：

* 当一个 transaction 需要消费的 states 中指定了不同的 notary 集群
* 当一个节点因为隐私和效率的考虑希望选择一个不同的 notary 集群

Before these transactions can be created, the states must first all be re-pointed to the same notary cluster. This is
achieved using a special notary-change transaction that takes:

* A single input state
* An output state identical to the input state, except that the appointed notary cluster has been changed

当这样的 transactions 被创建之前，states 必须首先被指定到同一个 notary 集群。这可以通过一个改变 notary 的 transaction 来实现:

* 单一的一个 input state
* 一个 output state 指定到这个 input state，除非指定的 notary 集群被改变了

The input state's appointed notary cluster will sign the transaction if it doesn't constitute a double-spend, at which
point a state will enter existence that has all the properties of the old state, but has a different appointed notary
cluster.

如果该 transaction 不会造成“双花”，这个 input state 指定的 notary 会为该 transaction 提供签名，这种情况下，一个 state 会进入到存在状态，它还有旧的 state 所具有的所有属性，但是会指向一个不同的 notary 集群。
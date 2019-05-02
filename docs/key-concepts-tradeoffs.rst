权衡
==========

.. topic:: 概要

   * *Permissioned networks are better suited for financial use-cases*
   * *Point-to-point communication allows information to be shared need-to-know*
   * *A UTXO model allows for more transactions-per-second*

   * *许可的网络会更好的适合金融的 user-cases*
   * *点对点的通信允许信息是基于需要知道的原则被共享*
   * *UTXO model 允许每秒钟能够处理更多的 transactions*

需要许可 vs 和不需要许可的
-------------------------------
Traditional blockchain networks are *permissionless*. The parties on the network are anonymous, and can join and
leave at will.

传统的 blockchain 是 *不需要许可* 的。网络中的各方都是匿名的，而且可以随时加入或离开。

By contrast, Corda networks are *permissioned*. Each party on the network has a known identity that they use when
communicating with counterparties, and network access is controlled by a doorman. This has several benefits:

* Anonymous parties are inappropriate for most scenarios involving regulated financial institutions
* Knowing the identity of your counterparties allows for off-ledger resolution of conflicts using existing
  legal systems
* Sybil attacks are averted without the use of expensive mechanisms such as proof-of-work

不同的是， Corda 网络是 *需要许可* 的。网络中的每一方都有一个大家都知道的标识，这个会在同其他节点进行沟通的时候使用，并且访问网络是由一个 doorman 来控制的。这有一下的好处：

* 匿名的用户对于大多数跟金融有关的情况都是不适用的
* 知道你的合作方的身份可以允许当出现冲突的时候，可以使用已经存在的法律系统在账本外进行解决
* 女巫攻击（Sybil attacks）可以不通过使用昂贵的机制（比如工作量证明 proof-of-work）来避免

点对点 vs 全局广播
------------------------------------
Traditional blockchain networks broadcast every message to every participant. The reason for this is two-fold:

* Counterparty identities are not known, so a message must be sent to every participant to ensure it reaches its
  intended recipient
* Making every participant aware of every transaction allows the network to prevent double-spends

传统的 blockchain networks 将每一条信息广播给网络上的所有参与者。原因是：

* 合作方的身份是不知道的，所以一条消息需要发给网络上的所有人来确保原本需要收到这条消息的接受者能够接收到
* 让所有参与者知道每一个 transaction 能够允许网络防止“双花”

The downside is that all participants see everyone else's data. This is unacceptable for many use-cases.

不好的地方是所有的参与者都能看到所有其他人的数据。这在很多的 use-cases 是无法接受的。

In Corda, each message is instead addressed to a specific counterparty, and is not seen by any uninvolved third
parties. The developer has full control over what messages are sent, to whom, and in what order. As a result, **data
is shared on a need-to-know basis only**. To prevent double-spends in this system, we employ notaries as
an alternative to proof-of-work.

在 Corda 中，每条消息都会指定一个具体的合作方，而且是不会被任何其他无关方看到的。开发者能够完全掌控什么消息被发送了，发送给了谁，应该按照什么顺序发送。所以 **数据是根据需要知道的原则来共享的**。为了避免“双花”，我们引入了 notaries 来替换掉工作量证明（proof-of-work）。

Corda also uses several other techniques to maximize privacy on the network:

* **Transaction tear-offs**: Transactions are structured in a way that allows them to be digitally signed without
  disclosing the transaction's contents. This is achieved using a data structure called a Merkle tree. You can read
  more about this technique in :doc:`tutorial-tear-offs`.
* **Key randomisation**: The parties to a transaction are identified only by their public keys, and fresh key pairs are
  generated for each transaction. As a result, an onlooker cannot identify which parties were involved in a given
  transaction.

Corda 也是用了其他的一些技术来最大化的包括网络上的隐私：

* **Transaction 隐藏**：Transactions 被结构化成不暴露 transaction 的内容就可以被数字化地签名。这个是通过使用一种叫默克尔树的数据结构来实现的。可以阅读 :doc:`tutorial-tear-offs` 来了解更详细的内容。
* **随机化秘钥**：一个 transaction 的所有参与方是通过他们的公钥进行识别的，并且针对每一个 transaction 都会生成 一个新的 keypairs。所以一个监视者无法识别出来对于一个给定的 transaction 都哪些方参与了。

UTXO vs. 账户模型
----------------------
Corda uses a *UTXO* (unspent transaction output) model. Each transaction consumes a set of existing states to produce
a set of new states.

Corda 使用 *UTXO*（Unspent Transaction Output）model。每个 transaction 都会消费一系列的已经存在的 state 然后再生成一些新的 states。

The alternative would be an *account* model. In an account model, stateful objects are stored on-ledger, and
transactions take the form of requests to update the current state of these objects.

相反的一种方式是 *账户* 模型。在账户模型中，stateful 对象被存在账本上，transaction 会通过请求的方式来对这些对象的当前的 state 进行更新。

The main advantage of the UTXO model is that transactions with different inputs can be applied in parallel,
vastly increasing the network's potential transactions-per-second. In the account model, the number of
transactions-per-second is limited by the fact that updates to a given object must be applied sequentially.

UTXO 模型的主要优点在于含有不同的 inputs 的 transactions 能够并行地被执行，很大程度上地增加了网络中每秒能够处理的 transactions。在账户模型中，每秒钟能够处理的 transactions 数量有限，因为对于一个给定的 object 的更新需要按照给定的顺序来执行。

代码即法律 vs. 既有的法律系统
--------------------------------------
Financial institutions need the ability to resolve conflicts using the traditional legal system where required. Corda
is designed to make this possible by:

* Having permissioned networks, meaning that participants are aware of who they are dealing with in every single
  transaction
* All code contracts should include a ``LegalProseReference`` link to the legal document describing the contract's intended behavior
  which can be relied upon to resolve conflicts

金融体系需要在需要的时候使用传统的法律体系来解决冲突的能力。Corda 被设计用来使这个成为可能：

* 拥有需要准入的网络，意味着所有参与方都能够知道在每一个 transaction 中他们都在跟谁打交道
* 所有代码合约背后都存在有描述着合约意图行为的法律文档，这个文档可以在解决冲突的时候使用

构建 vs. 重用
----------------
Wherever possible, Corda re-uses existing technologies to make the platform more robust platform overall. For
example, Corda re-uses:

* Standard JVM programming languages for the development of CorDapps
* Existing SQL databases
* Existing message queue implementations

任何可能的情况，Corda 会使用 已经存在的技术来让这个平台更加的健壮。比如 Corda 重用了：

* 标准的 JVM 变成语言来开发 CorDapps
* 已经存在的 SQL database
* 已经存在的 消息队列实现
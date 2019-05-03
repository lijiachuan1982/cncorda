.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: States
===========

.. note:: Before reading this page, you should be familiar with the key concepts of :doc:`key-concepts-states`.

.. note:: 在阅读这篇文档之前，你应该已经熟悉了核心概念 :doc:`key-concepts-states`。

.. contents::

ContractState
-------------
In Corda, states are instances of classes that implement ``ContractState``. The ``ContractState`` interface is defined
as follows:

在 Corda 中，states 是那些实现了 ``ContractState`` 的类实例。``ContractState`` 接口定义如下：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/contracts/ContractState.kt
        :language: kotlin
        :start-after: DOCSTART 1
        :end-before: DOCEND 1

``ContractState`` has a single field, ``participants``. ``participants`` is a ``List`` of the ``AbstractParty`` that
are considered to have a stake in the state. Among other things, the ``participants`` will:

* Usually store the state in their vault (see below)

* Need to sign any notary-change and contract-upgrade transactions involving this state

* Receive any finalised transactions involving this state as part of ``FinalityFlow`` / ``ReceiveFinalityFlow``

``ContractState`` 只有一个字段 ``participants``。``participants`` 是一个 ``AbstractParty`` 的 ``List``，代表了同这个 state 有关的节点。``participants`` 将会：

* 通常会将 state 存储到他们的 vault 中
* 需要为任何涉及到该 state 的 notary 变更和合约升级的交易提供签名
* 作为 ``FinalityFlow`` / ``ReceiveFinalityFlow`` 的一部分，接收任何涉及到该 state 的最终交易信息

ContractState 子接口
----------------------------
The behaviour of the state can be further customised by implementing sub-interfaces of ``ContractState``. The two most
common sub-interfaces are:

* ``LinearState``

* ``OwnableState``

state 的行为可以通过实现 ``ContractState`` 的子接口被进一步的定制。最常用的两个子接口包括：

* ``LinearState``
* ``OwnableState``

``LinearState`` models shared facts for which there is only one current version at any point in time. ``LinearState``
states evolve in a straight line by superseding themselves. On the other hand, ``OwnableState`` is meant to represent
assets that can be freely split and merged over time. Cash is a good example of an ``OwnableState`` - two existing $5
cash states can be combined into a single $10 cash state, or split into five $1 cash states. With ``OwnableState``, its
the total amount held that is important, rather than the actual units held.

``LinearState`` 代表了一个在任何时间都是只有一个当前版本的共享的事实。``LinearState`` states 通过替换自己的方式实现一个线性的改变。而 ``OwnableState`` 则代表在任何时候都可以被自由的拆分或者合并的资产。现金就是一个 ``OwnableState`` 的很好的例子 - 两个已经存在 $5 现金 state 可以合并为一个单独的 $10 的现金 state，或者被拆分成 5 个 $1 的现金 state。对于 ``OwnableState``，它的总金额是更重要的，而不是到底有多少份。

We can picture the hierarchy as follows:

我们可以通过下图来表述这个结构：

.. image:: resources/state-hierarchy.png

LinearState
^^^^^^^^^^^
The ``LinearState`` interface is defined as follows:

``LinearState`` 接口定义如下：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/contracts/Structures.kt
        :language: kotlin
        :start-after: DOCSTART 2
        :end-before: DOCEND 2

Remember that in Corda, states are immutable and can't be updated directly. Instead, we represent an evolving fact as a
sequence of ``LinearState`` states that share the same ``linearId`` and represent an audit trail for the lifecycle of
the fact over time.

记住在 Corda 中，states 是不可变的，并且不能直接的更改的。然而，我们可以使用有序的 ``LinearState`` states 来表现一个事实，这些 states 共同分享一个 ``linearId``，并且他们能够代表一个事实的整个生命周期。

When we want to extend a ``LinearState`` chain (i.e. a sequence of states sharing a ``linearId``), we:

* Use the ``linearId`` to extract the latest state in the chain from the vault

* Create a new state that has the same ``linearId``

* Create a transaction with:

  * The current latest state in the chain as an input

  * The newly-created state as an output

当我们想要扩展一个 ``LinearState`` 链（比如使用同一个 ``linearId`` 的 states 的序列）的时候，我们会：

* 使用 ``linearId`` 从账本中获取该 state 链中最新的 state
* 创建一个具有相同 ``linearId`` 的新的 state
* 创建一个包含下边元素的 transaction：
  * 将该 state 链中的当前版本的 state 作为 input
  * 将新创建的 state 作为 output

The new state will now become the latest state in the chain, representing the new current state of the agreement.

新创建的 state 现在就成为了这个 state 链的最新的 state，代表了协议的最新的当前 state。

``linearId`` is of type ``UniqueIdentifier``, which is a combination of:

* A Java ``UUID`` representing a globally unique 128 bit random number
* An optional external-reference string for referencing the state in external systems

``linearId`` 是一种 ``UniqueIdentifier`` 类型，由下边的元素组成：

* 一个 Java ``UUID``，代表了一个全局唯一的 128 bit 的随机数
* 一个可选的外部引用（external-reference） 字符串，作为在外部系统中使用的引用

OwnableState
^^^^^^^^^^^^
The ``OwnableState`` interface is defined as follows:

``OwnableState`` 接口定义如下：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/contracts/Structures.kt
        :language: kotlin
        :start-after: DOCSTART 3
        :end-before: DOCEND 3

Where:

* ``owner`` is the ``PublicKey`` of the asset's owner

* ``withNewOwner(newOwner: AbstractParty)`` creates an copy of the state with a new owner

其中：

* ``owner`` 是该资产的所有者的公钥 ``PublicKey``
* ``withNewOwner(newOwner: AbstractParty)`` 创建了一个具有新的所有者的 state 的副本

Because ``OwnableState`` models fungible assets that can be merged and split over time, ``OwnableState`` instances do
not have a ``linearId``. $5 of cash created by one transaction is considered to be identical to $5 of cash produced by
another transaction.

由于 OwnableState 形成了一个可替换的资产（fungible assets）的模型，这种资产可以合并和拆分，OwnableState 实例没有 linearId。一笔交易产生的 $5 现金和另一笔其他的交易产生的 $5 现金会被看作是同样的 state。

FungibleState
~~~~~~~~~~~~~

``FungibleState<T>`` is an interface to represent things which are fungible, this means that there is an expectation that
these things can be split and merged. That's the only assumption made by this interface. This interface should be
implemented if you want to represent fractional ownership in a thing, or if you have many things. Examples:

* There is only one Mona Lisa which you wish to issue 100 tokens, each representing a 1% interest in the Mona Lisa
* A company issues 1000 shares with a nominal value of 1, in one batch of 1000. This means the single batch of 1000
  shares could be split up into 1000 units of 1 share.

``FungibleState<T>`` 是代表了可替换的事物的一个接口，这意味着这些事物可以被拆分或者合并的例外。那是这个接口唯一的一个假设。如果你想要展示对于一件事物或者多个事物的部分所有权的话，那么久应该实现这个接口。比如：

* 因为只有一个蒙娜丽莎，你想要发行 100 个 tokens 为这个蒙娜丽莎，那么每个 token 就代表着蒙娜丽莎的 1%
* 一家公司发行了 1000 股股票，每股是 1，1次发行完。这意味着这样一次的 1000 股股票的发行可以被拆分为 1000 单位的 1 股股票。

The interface is defined as follows:

接口定义如下：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/contracts/FungibleState.kt
        :language: kotlin
        :start-after: DOCSTART 1
        :end-before: DOCEND 1

As seen, the interface takes a type parameter ``T`` that represents the fungible thing in question. This should describe
the basic type of the asset e.g. GBP, USD, oil, shares in company <X>, etc. and any additional metadata (issuer, grade,
class, etc.). An upper-bound is not specified for ``T`` to ensure flexibility. Typically, a class would be provided that
implements `TokenizableAssetInfo` so the thing can be easily added and subtracted using the ``Amount`` class.

像代码中那样，这个接口使用了一个代表了涉及的可替换的事物的类型参数 ``T``。这个应该描述了这个资产的基本类型，比如 GBP、USD、石油、公司 <X> 的股票，等等。通常，一个类应该被提供来实现 `TokenizableAssetInfo`，所以这个事物可以被简单地使用 ``Amount`` class 进行增加和减少。

This interface has been added in addition to ``FungibleAsset`` to provide some additional flexibility which
``FungibleAsset`` lacks, in particular:

* ``FungibleAsset`` defines an amount property of type ``Amount<Issued<T>>``, therefore there is an assumption that all
  fungible things are issued by a single well known party but this is not always the case.
* ``FungibleAsset`` implements ``OwnableState``, as such there is an assumption that all fungible things are ownable.

除了 ``FungibleAsset`` 外，这个接口被添加用来提供 ``FungibleAsset`` 缺少的额外的灵活性，特别是：

* ``FungibleAsset`` 定义了类型为 ``Amount<Issued<T>>`` 的一个数量属性，因此就有一个所有的可拆分的事物都是由单一一个已知的节点发行的这样一个假设，但是并不是所有情况都是这样的。
* ``FungibleAsset`` 实现了 ``OwnableState``，所以就有了这样一个所有可拆分的事物都是可以被拥有的一个假设。

其他接口
^^^^^^^^^^^^^^^^
You can also customize your state by implementing the following interfaces:

* ``QueryableState``, which allows the state to be queried in the node's database using custom attributes (see
  :doc:`api-persistence`)

* ``SchedulableState``, which allows us to schedule future actions for the state (e.g. a coupon payment on a bond) (see
  :doc:`event-scheduling`)

你也可以通过实现下边的接口来定制你的 state：

* ``QueryableState``，这可以让 state 能够在节点的数据库中通过使用自定义的属性来被查询（查看 :doc:`api-persistence`）
* ``SchedulableState``，可以允许我们对 state 设置一个将来会发生的动作（比如使用优惠券购买债券）（查看 :doc:`event-scheduling`）

用户定义字段
-------------------
Beyond implementing ``ContractState`` or a sub-interface, a state is allowed to have any number of additional fields
and methods. For example, here is the relatively complex definition for a state representing cash:

除了实现 ``ContractState`` 或者子接口外，一个 state 还允许包含任意数量的额外字段和方法。比如下边的代码就定义了一个相对复杂的代表现金 cash 的一个 state：

.. container:: codeset

    .. literalinclude:: ../../finance/contracts/src/main/kotlin/net/corda/finance/contracts/asset/Cash.kt
        :language: kotlin
        :start-after: DOCSTART 1
        :end-before: DOCEND 1

The vault
---------
Whenever a node records a new transaction, it also decides whether it should store each of the transaction's output
states in its vault. The default vault implementation makes the decision based on the following rules:

  * If the state is an ``OwnableState``, the vault will store the state if the node is the state's ``owner``
  * Otherwise, the vault will store the state if it is one of the ``participants``

当一个节点记录了一笔新的交易的时候，它还可以选择是否将交易的每一个 output state 存储到它的 vault 中。默认的 vault 实现让这个决定基于以下的规则：

  *如果 state 是一个 ``OwnableState``，如果该节点是该 state 的 ``owner`` 的时候，账本将会记录该 state
  *如果不是上边的情况，如果节点是该 state 的 ``participants`` 中的一员，那么账本就会记录该 state

States that are not considered relevant are not stored in the node's vault. However, the node will still store the
transactions that created the states in its transaction storage.

不相关的 states 是不会存储到节点的 vault 中的。但是节点还是会将创建该 state 的 transaction 存储到它的 transaction storage 中。

TransactionState
----------------
When a ``ContractState`` is added to a ``TransactionBuilder``, it is wrapped in a ``TransactionState``:

当一个 ``ContractState`` 被添加到一个 ``TransactionBuilder`` 之后，它就被包装成了一个 ``TransactionState``：

.. container:: codeset

   .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/contracts/TransactionState.kt
      :language: kotlin
      :start-after: DOCSTART 1
      :end-before: DOCEND 1

Where:

* ``data`` is the state to be stored on-ledger
* ``contract`` is the contract governing evolutions of this state
* ``notary`` is the notary service for this state
* ``encumbrance`` points to another state that must also appear as an input to any transaction consuming this
  state
* ``constraint`` is a constraint on which contract-code attachments can be used with this state

其中：

* ``data`` 是将被存储到账本中的 state
* ``contract`` 是一个控制 state 转变的合约
* ``notary`` 是这个 state 的 notary service
* ``encumbrance`` 指向了另一个 state，该 state 必须以一个 input 的形式出现在消费此 state 的交易中
* ``constraint`` 是一个该 state 使用的合约代码 contract-code 附件的约束

.. _reference_states:

引用 States
----------------

A reference input state is a ``ContractState`` which can be referred to in a transaction by the contracts of input and
output states but whose contract is not executed as part of the transaction verification process. Furthermore,
reference states are not consumed when the transaction is committed to the ledger but they are checked for
"current-ness". In other words, the contract logic isn't run for the referencing transaction only. It's still a normal
state when it occurs in an input or output position.

一个引用类型 input state 是一个 ``ContractState``，它可以在一个交易中心在 input 和 output states 的合约代码所引用，但是它所对应的合约代码是不会作为交易的验证过程的一部分被执行的。而且，引用类型的 states 在交易被提交到账本的时候是不会被消费掉的，但是他们会被验证 “是否是最新的”。换句话说，合约的逻辑是不会针对引用的交易执行的。当把它放在一个 input 或者 output 的位置的时候，它依旧是一个常规的 state。

Reference data states enable many parties to reuse the same state in their transactions as reference data whilst
still allowing the reference data state owner the capability to update the state. A standard example would be the
creation of financial instrument reference data and the use of such reference data by parties holding the related
financial instruments.

引用类型的数据 states 能够让多方在他们的交易中重用相同的 state 作为引用的数据，然而还能依旧允许这个引用数据 state 的所有者有能力来更新这个 state。一个典型的例子是金融票据引用数据以及由持有这些相关的金融票据的相关方对这些引用数据的使用。

Just like regular input states, the chain of provenance for reference states is resolved and all dependency transactions
verified. This is because users of reference data must be satisfied that the data they are referring to is valid as per
the rules of the contract which governs it and that all previous participants of the state assented to updates of it.

像常规的 input states 一样，关于引用类型的 states 的起源链能够被解决，并且所有依赖的交易也会被验证。这是因为引用数据的用户必须要满意他们引用的数据基于管理它的合约代码定义的规则来说是有效的，并且这个 state 的左右以前的参与方都赞同了它的变更。

**Known limitations:**

**已知的局限性：**

*Notary change:* It is likely the case that users of reference states do not have permission to change the notary
assigned to a reference state. Even if users *did* have this permission the result would likely be a bunch of
notary change races. As such, if a reference state is added to a transaction which is assigned to a
different notary to the input and output states then all those inputs and outputs must be moved to the
notary which the reference state uses.

*Notary 变更：*

If two or more reference states assigned to different notaries are added to a transaction then it follows that this
transaction cannot be committed to the ledger. This would also be the case for transactions not containing reference
states. There is an additional complication for transactions including reference states; it is however, unlikely that the
party using the reference states has the authority to change the notary for the state (in other words, the party using the
reference state would not be listed as a participant on it). Therefore, it is likely that a transaction containing
reference states with two different notaries cannot be committed to the ledger.

如果两个或者多个引用类型的 states 被分配了不同的 notaries，并且他们被添加到了同一个交易的时候，这笔交易是不能够被提交到账本的。这对于不包含引用类型的 states 的情况也是一样的。对于包含引用类型的 states 的情况会具有额外的复杂性；使用这个引用 states 的相关方是没有权限来更改这个 state 对应的 notary 的（换句话说，使用这个引用数据的相关方并没有被添加到这个 state 的参与者列表里）。因此，包含引用类型 states 的交易如果有两个不同的 notaries 的话，是不能够提交到账本的。

As such, if reference states assigned to multiple different notaries are added to a transaction builder
then the check below will fail.

因此，如果引用类型的 states 被分配了多个不同的 notaries 并且被添加到一个 transaction builder 的时候，下边的检查是会失败的。

.. warning:: Currently, encumbrances should not be used with reference states. In the case where a state is
   encumbered by an encumbrance state, the encumbrance state should also be referenced in the same
   transaction that references the encumbered state. This is because the data contained within the
   encumbered state may take on a different meaning, and likely would do, once the encumbrance state
   is taken into account.

.. warning:: 当前，encumbrances 不应该同引用类型的 states 一起使用。对于一个被 encumbrance state 阻碍的 state，encumbrance state 应该在同一个引用这个被阻碍的 state 的交易中被引用。这是因为在被阻碍的 state 中包含的数据可能具有不同的含义，并且一旦 encumbrance state 被考虑的话，会更可能发生。

.. _state_pointers:

State 指针
--------------

A ``StatePointer`` contains a pointer to a ``ContractState``. The ``StatePointer`` can be included in a ``ContractState`` as a
property, or included in an off-ledger data structure. ``StatePointer`` s can be resolved to a ``StateAndRef`` by performing
a look-up. There are two types of pointers; linear and static.

1. ``StaticPointer`` s are for use with any type of ``ContractState``. The ``StaticPointer`` does as it suggests, it always
   points to the same ``ContractState``.
2. The ``LinearPointer`` is for use with LinearStates. They are particularly useful because due to the way LinearStates
   work, the pointer will automatically point you to the latest version of a LinearState that the node performing ``resolve``
   is aware of. In effect, the pointer "moves" as the LinearState is updated.

一个 ``StatePointer`` 包含了一个指向 ``ContractState`` 的指针。``StatePointer`` 可以作为一个属性被包含在一个 ``ContractState`` 里，或者被包含在一个 off-ledger 的数据结构中。``StatePointer`` 可以通过执行一个查询被处理成一个 ``StateAndRef``。有两种类型的指针：linear 和 static。

1. ``StaticPointer`` 可以跟任何类型的 ``ContractState`` 一起使用。``StaticPointer`` 像他建议的那样，总是会指向相同的 ``ContractState``。
2. ``LinearPointer`` 是跟 LinearStates 共同使用的。由于 LinearStates 的工作方式，他们是非常有用的，指针将会自动地将你指向一个节点在执行 ``resolve`` 所了解的 LinearState 的最新版本。因此，这个指针随着 LinearState 的更新而被 “移动”。

State pointers use ``Reference States`` to enable the functionality described above. They can be conceptualized as a mechanism to
formalise a development pattern where one needs to refer to a specific state from another transaction (StaticPointer) or a particular lineage
of states (LinearPointer). In other words, ``StatePointers`` do not enable a feature in Corda which was previously unavailable.
Rather, they help to formalise a pattern which was already possible. In that light, it is worth noting some issues which you may encounter
in its application:

* If the node calling ``resolve`` has not seen any transactions containing a ``ContractState`` which the ``StatePointer``
  points to, then ``resolve`` will throw an exception. Here, the node calling ``resolve`` might be missing some crucial data.
* The node calling ``resolve`` for a ``LinearPointer`` may have seen and stored transactions containing a ``LinearState`` with
  the specified ``linearId``. However, there is no guarantee the ``StateAndRef<T>`` returned by ``resolve`` is the most recent
  version of the ``LinearState``. The node only returns the most recent version that _it_ is aware of.

State 指针使用 ``Reference States`` 来实现上边描述的功能。他们可以被概念化作为一个成为一种开发模式的机制，这种模式中一个 state 需要引用一个从来自于其他的 transaction（StaticPointer）或者一个特定的 states 的系列（LinearPointer）的一个指定的 state。换句话说，``StatePointers`` 不会将 Corda 之前没有的功能变为可能。但是，它会帮助形成一个以前就有可能的模式。从这个角度看，你在它的应用程序中可能遇到的一些问题就没有什么意义了：

* 如果节点调用 ``resolve`` 而没有看到任何的交易包含一个 ``StatePointer`` 所指向的 ``ContractState``，那么 ``resolve`` 将会抛出异常。这里，节点调用 ``resolve`` 的时候可能忘记了一些重要的数据。
* 节点为 ``LinearPointer`` 调用 ``resolve``可能会看到并且存储了包含一个带有指定的 ``linearId`` 的 ``LinearState`` 的交易信息。然而，并没有谁能保证 ``resolve`` 返回的 ``StateAndRef<T>`` 是最新版本的 ``LinearState``。节点只会返回 _它_ 所知道的最新版本。

**Resolving state pointers in TransactionBuilder**

**在 TransactionBuilder 中处理 state 指针**

When building transactions, any ``StatePointer`` s contained within inputs or outputs added to a ``TransactionBuilder`` can
be optionally resolved to reference states using the ``resolveStatePointers`` method. The effect is that the pointed to
data is carried along with the transaction. This may or may not be appropriate in all circumstances, which is why
calling the method is optional.

当构建 transactions 的时候，任何被添加到一个 ``TransactionBuilder`` 的 inputs 或者 outputs 中包含的 ``StatePointer``，可以使用 ``resolveStatePointers`` 方法来被有选择地处理到一个引用的 states。这样做的效果就是被指向的数据会随着 transaction 被带走。这个在所有的情况下可能是合适或者是不合适的，这也就是为什么说这个方法是可选的。
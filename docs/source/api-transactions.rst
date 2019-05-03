.. highlight:: kotlin
.. raw:: html

   <script type="text/javascript" src="_static/jquery.js"></script>
   <script type="text/javascript" src="_static/codesets.js"></script>

API: Transactions
=================

.. note:: Before reading this page, you should be familiar with the key concepts of :doc:`key-concepts-transactions`.

.. note:: 当阅读这里的时候，你应该已经熟悉了核心概念 :doc:`key-concepts-transactions`。

.. contents::

Transaction 生命周期
---------------------
Between its creation and its final inclusion on the ledger, a transaction will generally occupy one of three states:

* ``TransactionBuilder``. A transaction's initial state. This is the only state during which the transaction is
  mutable, so we must add all the required components before moving on.

* ``SignedTransaction``. The transaction now has one or more digital signatures, making it immutable. This is the
  transaction type that is passed around to collect additional signatures and that is recorded on the ledger.

* ``LedgerTransaction``. The transaction has been "resolved" - for example, its inputs have been converted from
  references to actual states - allowing the transaction to be fully inspected.

从它被创建到最终被添加到账本中，每个 transaction 会大体占用 3种状态中的一种：

* ``TransactionBuilder``。这个是 transaction 的初始状态。这也是 transaction 唯一可以被修改的一个状态，所以在进行下一步之前我们必须要确保添加了所有必须的组件。
* ``SignedTransaction``。现在的 transaction 已经有了一个或者更多的数字签名，并且已经是不可修改了。这个会是在不同的节点间传递来获得更多签名的 transaction 类型，也是会最终被记录到账本中的 transaction。
* ``LedgerTransaction``。这个 transaction 已经被“解决”掉了。比如它的 inputs 已经从引用被转换为实际的 states 了 - 允许 transaction 被彻底地检查。

We can visualise the transitions between the three stages as follows:

我们可以用下图来表示 transactions 在三个状态中的转换：

.. image:: resources/transaction-flow.png

Transaction 组件
----------------------
A transaction consists of six types of components:

* 1+ states:

  * 0+ input states
  * 0+ output states
  * 0+ reference input states

* 1+ commands
* 0+ attachments
* 0 or 1 time-window

  * A transaction with a time-window must also have a notary

一个 transaction 包括六种类型的组件：

* 1+ states：

  * 0+ input states
  * 0+ output states
  * 0+ reference input states

* 1+ commands
* 0+ attachments
* 0 或者 1 time-window

  * 带有 time-window 的 transaction 还必须要有一个 notary

Each component corresponds to a specific class in the Corda API. The following section describes each component class,
and how it is created.

每个组件都对应于 Corda API 中的一个指定的类。下边的部分描述了每个组件的类，和他们是如何被创建的。

Input states
^^^^^^^^^^^^
An input state is added to a transaction as a ``StateAndRef``, which combines:

* The ``ContractState`` itself
* A ``StateRef`` identifying this ``ContractState`` as the output of a specific transaction

Input states 是以 ``StateAndRef`` 实例的形式添加进 transaction 的，它包括：

* ``ContractState`` 本身
* 一个 ``StateRef`` 用来识别作为一个指定的 transaction 的 output 的该 ``ContractState``

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 21
        :end-before: DOCEND 21
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 21
        :end-before: DOCEND 21
        :dedent: 12

A ``StateRef`` uniquely identifies an input state, allowing the notary to mark it as historic. It is made up of:

* The hash of the transaction that generated the state
* The state's index in the outputs of that transaction

一个 ``StateRef`` 唯一地识别了一个 input state，允许 notary 可以将它标记为一个历史记录。它由下边的元素组成：

* 产生该 state 的 transaction 的哈希值
* 该 state 在这个 transaction 中的 outputs 列表中的索引值

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 20
        :end-before: DOCEND 20
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 20
        :end-before: DOCEND 20
        :dedent: 12

The ``StateRef`` links an input state back to the transaction that created it. This means that transactions form
"chains" linking each input back to an original issuance transaction. This allows nodes verifying the transaction
to "walk the chain" and verify that each input was generated through a valid sequence of transactions.

``StateRef`` 将一个 input 连接回来产生它的那次 transaction。这就意味着那个 transaction 形成了一个“链条”，这个链条将每个 input 都同产生它的原始 transaction 链接在了一起。这就允许了节点可以回溯整条链来确认一个新的 transaction 并且确保了每个 input 都是通过一个有效的并且有序的 transaction 来产生的。

引用 input states
~~~~~~~~~~~~~~~~~~~~~~

.. warning:: Reference states are only available on Corda networks with a minimum platform version >= 4.

.. warning:: 引用类型的 states 只有在平台版本 >= 4 的 Corda 网络中才可用。

A reference input state is added to a transaction as a ``ReferencedStateAndRef``. A ``ReferencedStateAndRef`` can be
obtained from a ``StateAndRef`` by calling the ``StateAndRef.referenced()`` method which returns a ``ReferencedStateAndRef``.

一个引用类型的 state 是以 ``ReferencedStateAndRef`` 形式被添加到一个交易中的。一个 A ``ReferencedStateAndRef`` 可以通过调用 ``StateAndRef.referenced()`` 方法来从一个 ``StateAndRef`` 获得，这回返回一个 ``ReferencedStateAndRef``。

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 55
        :end-before: DOCEND 55
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 55
        :end-before: DOCEND 55
        :dedent: 12

**处理更新比赛:**

When using reference states in a transaction, it may be the case that a notarisation failure occurs. This is most likely
because the creator of the state (being used as a reference state in your transaction), has just updated it.

当在交易中使用引用类型的 states 的时候，可能会发生公证失败的错误。这很有可能是因为 state 的创建者（在你的交易中被作为一个引用类型的 state 被使用），刚刚更新了它。

Typically, the creator of such reference data will have implemented flows for syndicating the updates out to users.
However it is inevitable that there will be a delay between the state being used as a reference being consumed, and the
nodes using it receiving the update.

通常，这类引用类型的数据的创建者将会具有为用户提供更新的已经实现的 flows。然而，这个必然在正在被消费的作为一个引用的 state 正在被使用和节点正在使用它来接收更新之间会有延迟。

This is where the ``WithReferencedStatesFlow`` comes in. Given a flow which uses reference states, the
``WithReferencedStatesFlow`` will execute the the flow as a subFlow. If the flow fails due to a ``NotaryError.Conflict``
for a reference state, then it will be suspended until the state refs for the reference states are consumed. In this
case, a consumption means that:

1. the owner of the reference state has updated the state with a valid, notarised transaction
2. the owner of the reference state has shared the update with the node attempting to run the flow which uses the
   reference state
3. The node has successfully committed the transaction updating the reference state (and all the dependencies), and
   added the updated reference state to the vault.

这就带来了 ``WithReferencedStatesFlow``。给定一个使用引用类型 states 的一个 flow，``WithReferencedStatesFlow``将会以一个 subflow 的方式执行这个 flow。如果这个 flow 因为对于一个引用的 state 的 ``NotaryError.Conflict`` 原因而失败了的话，那么它就会被挂起，直到引用这个引用类型的 state 的 state 被消费掉。在这个情况下，一个消费代表着：

1. 这个引用 state 的所有者已经使用一个有效的经过公证的交易更新了这个 state
2. 这个引用 state 的所有者已经跟尝试运行使用这个引用 state 的 flow 的节点共享了更新
3. 这个节点已经成功地提交了更新这个引用 state 的交易（以及所有的依赖），并且将这个更新过的引用 state 添加到 vault

At the point where the transaction updating the state being used as a reference is committed to storage and the vault
update occurs, then the ``WithReferencedStatesFlow`` will wake up and re-execute the provided flow.

当更新作为一个引用的 state 的交易被提交并且 vault 的更新发生的时候，``WithReferencedStatesFlow`` 会被唤醒并且会重新执行提供的 flow。

.. warning:: Caution should be taken when using this flow as it facilitates automated re-running of flows which use
             reference states. The flow using reference states should include checks to ensure that the reference data is
             reasonable, especially if the economics of the transaction depends upon the data contained within a reference state.

.. warning:: 当使用这个 flow 的时候要特别小心，因为它会协助使用引用 states 的 flow 自动地重新运行。使用引用 states 的 flow 应该包含一个检查来确保这个引用的数据是有道理的，特别当交易的情况依赖于在一个引用 state 内包含的数据。

Output states
^^^^^^^^^^^^^
Since a transaction's output states do not exist until the transaction is committed, they cannot be referenced as the
outputs of previous transactions. Instead, we create the desired output states as ``ContractState`` instances, and
add them to the transaction directly:

因为一个 transaction 的 output states 在 transaction 被最终提交前是不存在的，所以他们不能够被之前的 transaction 进行引用。相反，我们通过创建 ``ContractState`` 实例的方式创建想要的 output states，并直接把他们添加到 transaction 中：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 22
        :end-before: DOCEND 22
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 22
        :end-before: DOCEND 22
        :dedent: 12

In cases where an output state represents an update of an input state, we may want to create the output state by basing
it on the input state:

当一个 output 会作为一个 input 的更新版本的时候，我们可能会希望基于原始的这个 input state 来创建一个新的 output state：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 23
        :end-before: DOCEND 23
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 23
        :end-before: DOCEND 23
        :dedent: 12

Before our output state can be added to a transaction, we need to associate it with a contract. We can do this by
wrapping the output state in a ``StateAndContract``, which combines:

* The ``ContractState`` representing the output states
* A ``String`` identifying the contract governing the state

当我们的 output state 在能够被添加到一个 transaction 之前，我们需要将它同一个 contract 关联起来。我们可以通过将这个 output state 放入一个 ``StateAndContract`` 中，它将下边两个元素整合在了一起：

* ``ContractState`` 代表了 output state
* 一个 ``String`` 用来识别决定该 state 的 contract

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 47
        :end-before: DOCEND 47
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 47
        :end-before: DOCEND 47
        :dedent: 12

Commands
^^^^^^^^
A command is added to the transaction as a ``Command``, which combines:

* A ``CommandData`` instance indicating the command's type
* A ``List<PublicKey>`` representing the command's required signers

一个 command 是做为 ``Command`` 实例被添加到一个 transaction 中的。Command 包含：

* 一个 ``CommandData`` 实例，它代表了 command 的类型
* 一个 ``List<PublicKey>`` 代表了 command 所要求的签名者的列表

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 24
        :end-before: DOCEND 24
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 24
        :end-before: DOCEND 24
        :dedent: 12

Attachments
^^^^^^^^^^^
Attachments are identified by their hash:

Attachments 附件是通过他们的哈希值来识别的：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 25
        :end-before: DOCEND 25
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 25
        :end-before: DOCEND 25
        :dedent: 12

The attachment with the corresponding hash must have been uploaded ahead of time via the node's RPC interface.

具有相应的哈希值的附件必须要提前通过节点的 RPC 接口上传到 ledger 中。

Time-windows
^^^^^^^^^^^^
Time windows represent the period during which the transaction must be notarised. They can have a start and an end
time, or be open at either end:

Time windows 代表了一个时间区间，transaction 必须要在这个时间区间内被公正。它可以有一个起始和终止时间，或者是一个开放的区间：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 26
        :end-before: DOCEND 26
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 26
        :end-before: DOCEND 26
        :dedent: 12

We can also define a time window as an ``Instant`` plus/minus a time tolerance (e.g. 30 seconds):

我们也可以定义一个包含一个 Instant 和正/负时间差的 time window（比如加/减 30 秒钟）：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 42
        :end-before: DOCEND 42
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 42
        :end-before: DOCEND 42
        :dedent: 12

Or as a start-time plus a duration:

或者包含一个起始时间加上一个时间段：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 43
        :end-before: DOCEND 43
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 43
        :end-before: DOCEND 43
        :dedent: 12

TransactionBuilder
------------------

创建一个 builder
^^^^^^^^^^^^^^^^^^
The first step when creating a transaction proposal is to instantiate a ``TransactionBuilder``.

创建一个 transaction proposal 的第一步是实例化一个 ``TransactionBuilder``。

If the transaction has input states or a time-window, we need to instantiate the builder with a reference to the notary
that will notarise the inputs and verify the time-window:

如果一个 transaction 包含 input states 或者一个 time-window 的话，我们需要实例化这个 builder 并且需要有一个关于 notary 的引用，这个 notary 会对 inputs 进行公正并且验证这个 time-window：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 19
       :end-before: DOCEND 19
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 19
       :end-before: DOCEND 19
       :dedent: 12

We discuss the selection of a notary in :doc:`api-flows`.

我们在 :doc:`api-flows` 讨论了如何选择一个 notary。

If the transaction does not have any input states or a time-window, it does not require a notary, and can be
instantiated without one:

如果一个 transaction 没有任何的 input states 或者 time-window 的话，那就不需要指定 notary 来实例化了：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 46
        :end-before: DOCEND 46
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 46
        :end-before: DOCEND 46
        :dedent: 12

添加 items
^^^^^^^^^^^^
The next step is to build up the transaction proposal by adding the desired components.

下一步就是通过添加期望的组件来构建 transaction。

We can add components to the builder using the ``TransactionBuilder.withItems`` method:

我们可以使用 ``TransactionBuilder.withItems`` 方法来向 builder 中增加组件：

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/transactions/TransactionBuilder.kt
       :language: kotlin
       :start-after: DOCSTART 1
       :end-before: DOCEND 1

``withItems`` takes a ``vararg`` of objects and adds them to the builder based on their type:

* ``StateAndRef`` objects are added as input states
* ``ReferencedStateAndRef`` objects are added as reference input states
* ``TransactionState`` and ``StateAndContract`` objects are added as output states

  * Both ``TransactionState`` and ``StateAndContract`` are wrappers around a ``ContractState`` output that link the
    output to a specific contract

* ``Command`` objects are added as commands
* ``SecureHash`` objects are added as attachments
* A ``TimeWindow`` object replaces the transaction's existing ``TimeWindow``, if any

``withItems`` 使用了一个由对象构成的 ``vararg``，并根据他们的类型向 builder 中添加内容：

* ``StateAndRef`` 对象是作为 input states 被添加
* ``ReferencedStateAndRef`` 对象作为引用类型的 input states 被添加
* ``TransactionState`` 和 ``StateAndContract`` 对象是作为 output states 被添加

  * ``TransactionState`` 和 ``StateAndContract`` 会被 wrapper 成一个 ``ContractState`` output，这就将 output 和一个指定的 contract 链接到了一起

* ``Command`` 对象是作为 commands 被添加
* ``SecureHash`` 对象是作为附件被添加的
* 如果 transaction 中已经存在 ``TimeWindow`` 的话，那么这里的 ``TimeWindow`` 对象会替换掉那个已经存在的 ``TimeWindow``

Passing in objects of any other type will cause an ``IllegalArgumentException`` to be thrown.

传入任何其他类型的对象将会造成一个 ``IllegalArgumentException`` 被抛出。

Here's an example usage of ``TransactionBuilder.withItems``:

下边是一个如何使用 ``TransactionBuilder.withItems`` 的实例代码：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 27
       :end-before: DOCEND 27
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 27
       :end-before: DOCEND 27
       :dedent: 12

There are also individual methods for adding components.

这里也有独立的方法来添加不同的组件。

Here are the methods for adding inputs and attachments:

添加 inputs 和 附件的方法：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 28
        :end-before: DOCEND 28
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 28
        :end-before: DOCEND 28
        :dedent: 12

An output state can be added as a ``ContractState``, contract class name and notary:

一个 output state 可以作为 ``ContractState``，contract 类名和 notary 来添加：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 49
        :end-before: DOCEND 49
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 49
        :end-before: DOCEND 49
        :dedent: 12

We can also leave the notary field blank, in which case the transaction's default notary is used:

我们也可以将 notary 字段留空，那么 transaction 的默认 notary 就会被使用了：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 50
        :end-before: DOCEND 50
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 50
        :end-before: DOCEND 50
        :dedent: 12

Or we can add the output state as a ``TransactionState``, which already specifies the output's contract and notary:

或者我们可以将一个 output state 作为 ``TransactionState`` 来添加，它已经指定了 output 的 contract 和 notary：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 51
        :end-before: DOCEND 51
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 51
        :end-before: DOCEND 51
        :dedent: 12

Commands can be added as a ``Command``:

Commands 可以作为 ``Command`` 被添加：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 52
        :end-before: DOCEND 52
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 52
        :end-before: DOCEND 52
        :dedent: 12

Or as ``CommandData`` and a ``vararg PublicKey``:

或者作为 ``CommandData`` 和一个 ``vararg PublicKey``：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
        :language: kotlin
        :start-after: DOCSTART 53
        :end-before: DOCEND 53
        :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
        :language: java
        :start-after: DOCSTART 53
        :end-before: DOCEND 53
        :dedent: 12

For the time-window, we can set a time-window directly:

对于 time-window，我们可以直接设定 time-window：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 44
       :end-before: DOCEND 44
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 44
       :end-before: DOCEND 44
       :dedent: 12

Or define the time-window as a time plus a duration (e.g. 45 seconds):

或者将 time-window 定义为一个时间加上一个时间差（比如 45 秒钟）：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 45
       :end-before: DOCEND 45
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 45
       :end-before: DOCEND 45
       :dedent: 12

为 builder 签名
^^^^^^^^^^^^^^^^^^^
Once the builder is ready, we finalize it by signing it and converting it into a ``SignedTransaction``.

一旦 builder 准备好了，我们就可以通过签名的方式将它变为一个 ``SignedTransaction``。

We can either sign with our legal identity key:

我们可以使用我们的 legal identity key 来签名：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 29
       :end-before: DOCEND 29
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 29
       :end-before: DOCEND 29
       :dedent: 12

Or we can also choose to use another one of our public keys:

或者也可以选择使用我们的另一个公钥（public key）来签名：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 30
       :end-before: DOCEND 30
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 30
       :end-before: DOCEND 30
       :dedent: 12

Either way, the outcome of this process is to create an immutable ``SignedTransaction`` with our signature over it.

任何的方式，这个流程的输出都会是创建了一个带有我们签名的无法修改的 ``SignedTransaction``。

SignedTransaction
-----------------
A ``SignedTransaction`` is a combination of:

* An immutable transaction
* A list of signatures over that transaction

一个 ``SignedTransaction`` 是下边内容的组合：

* 一个不可修改的 transaction
* 在这个 transaction 上的签名列表

.. container:: codeset

    .. literalinclude:: ../../core/src/main/kotlin/net/corda/core/transactions/SignedTransaction.kt
       :language: kotlin
       :start-after: DOCSTART 1
       :end-before: DOCEND 1

Before adding our signature to the transaction, we'll want to verify both the transaction's contents and the
transaction's signatures.

当提供我们的签名之前，我们会既要确认 transaction 的内容，也有确认 transaction 的签名。

确认 transaction 的内容
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If a transaction has inputs, we need to retrieve all the states in the transaction's dependency chain before we can
verify the transaction's contents. This is because the transaction is only valid if its dependency chain is also valid.
We do this by requesting any states in the chain that our node doesn't currently have in its local storage from the
proposer(s) of the transaction. This process is handled by a built-in flow called ``ReceiveTransactionFlow``.
See :doc:`api-flows` for more details.

如果一个 transaction 含有 inputs 的话，在能够确认 transaction 的内容之前，我们需要取回这个 transaction 依赖的 transaction 链中的所有 states。这是因为只有当依赖链是有效的时候，这个 transaction 才会被认为是有效的。我们可以通过向发起 transaction 的一方来请求任何在当前结点的本地存储中没有 states 来最终验证整个 transaction 依赖链。这个流程是由一个内置的名为 ``ReceiveTransactionFlow`` 的方法来处理的。查看 :doc:`api-flows` 了解详细信息。

We can now verify the transaction's contents to ensure that it satisfies the contracts of all the transaction's input
and output states:

我们现在就可以验证 transaction 的内容来确保它的 input 和 output states 中的 contract code 中定义的约束都能满足：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 33
       :end-before: DOCEND 33
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 33
       :end-before: DOCEND 33
       :dedent: 16

Checking that the transaction meets the contract constraints is only part of verifying the transaction's contents. We
will usually also want to perform our own additional validation of the transaction contents before signing, to ensure
that the transaction proposal represents an agreement we wish to enter into.

检查 transaction 满足合约约束（contract constraints）只是验证 transaction 内容的一部分。通常我们也会在提供签名前，希望进行我们自己指定的额外的验证，来确保 transaction proposal 是我们真正想加入的一个协议。

However, the ``SignedTransaction`` holds its inputs as ``StateRef`` instances, and its attachments as ``SecureHash``
instances, which do not provide enough information to properly validate the transaction's contents. We first need to
resolve the ``StateRef`` and ``SecureHash`` instances into actual ``ContractState`` and ``Attachment`` instances, which
we can then inspect.

但是，``SignedTransaction`` 将它的 inputs 以 ``StateRef`` 实例的形式保留，并且它的附件是作为 ``SecureHash`` 的实例，这并不能提供足够的信息来很好地验证 transaction 的内容。我们首先需要解决的是将 ``StateRef`` 和 ``SecureHash`` 实例化为真正的 ``ContractState`` 和 ``Attachment`` 的实例，然后我们就可以检查了。

We achieve this by using the ``ServiceHub`` to convert the ``SignedTransaction`` into a ``LedgerTransaction``:

我们通过使用 ``ServiceHub`` 来将 ``SignedTransaction`` 转换为一个 ``LedgerTransaction``：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 32
       :end-before: DOCEND 32
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 32
       :end-before: DOCEND 32
       :dedent: 16

We can now perform our additional verification. Here's a simple example:

我们现在就可以进行额外的验证了，下边是示例代码：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 34
       :end-before: DOCEND 34
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 34
       :end-before: DOCEND 34
       :dedent: 16

确认 transaction 的签名
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Aside from verifying that the transaction's contents are valid, we also need to check that the signatures are valid. A
valid signature over the hash of the transaction prevents tampering.

除了确认 transaction 的内容是有效的，我们也要检查签名是有效的。一个建立在 transaction 的哈希值的基础上有效的签名能够防止记录被篡改。

We can verify that all the transaction's required signatures are present and valid as follows:

我们可以验证该 transaction 需要的所有的签名都已经被提供了：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 35
       :end-before: DOCEND 35
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 35
       :end-before: DOCEND 35
       :dedent: 16

However, we'll often want to verify the transaction's existing signatures before all of them have been collected. For
this we can use ``SignedTransaction.verifySignaturesExcept``, which takes a ``vararg`` of the public keys for
which the signatures are allowed to be missing:

然而，在所有的签名被搜集到之前，我们通常也会希望先确认 transaction 里已经有的签名。我们可以使用 ``SignedTransaction.verifySignaturesExcept``，它带有一个公钥的 ``vararg`` 传入参数，它会允许该公钥不需要提供签名：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 36
       :end-before: DOCEND 36
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 36
       :end-before: DOCEND 36
       :dedent: 16

There is also an overload of ``SignedTransaction.verifySignaturesExcept``, which takes a ``Collection`` of the
public keys for which the signatures are allowed to be missing:

这里还有一个对于 ``SignedTransaction.verifySignaturesExcept`` 的重载，它可以传入一个允许不提供签名的公钥的 ``集合``：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 54
       :end-before: DOCEND 54
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 54
       :end-before: DOCEND 54
       :dedent: 16


If the transaction is missing any signatures without the corresponding public keys being passed in, a
``SignaturesMissingException`` is thrown.

如果一个 transaction 没有传入对应的公钥而造成缺少任何的签名的话，一个 ``SignaturesMissingException`` 会被抛出。

We can also choose to simply verify the signatures that are present:

我们也可以选择只是简单地确认一下签名是否提供了：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 37
       :end-before: DOCEND 37
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 37
       :end-before: DOCEND 37
       :dedent: 16

Be very careful, however - this function neither guarantees that the signatures that are present are required, nor
checks whether any signatures are missing.

但是要小心，这个方法既不能保证被展示出来的签名是必须要有的，也不能查出是否缺少了任何的签名。

为 transaction 提供签名
^^^^^^^^^^^^^^^^^^^^^^^
Once we are satisfied with the contents and existing signatures over the transaction, we add our signature to the
``SignedTransaction`` to indicate that we approve the transaction.

一旦我们同意了 transaction 的内容以及 transaction 上已经存在的这些签名，我们就可以将自己的签名附加在这个 ``SignedTransaction`` 上来说明我们同意了这个 transaction。

We can sign using our legal identity key, as follows:

我们可以使用我们的 legal identity key 来签名：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 38
       :end-before: DOCEND 38
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 38
       :end-before: DOCEND 38
       :dedent: 12

Or we can choose to sign using another one of our public keys:

或者可以使用我们的其他的公钥来签名：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 39
       :end-before: DOCEND 39
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 39
       :end-before: DOCEND 39
       :dedent: 12

We can also generate a signature over the transaction without adding it to the transaction directly.

我们也可以通过 transaction 生成一个签名但是不直接地把它添加到 transaction 中。

We can do this with our legal identity key:

我们可以使用我们的 legal identity key 来实现这个：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 40
       :end-before: DOCEND 40
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 40
       :end-before: DOCEND 40
       :dedent: 12

Or using another one of our public keys:

或者使用我们的另外的公钥：

.. container:: codeset

    .. literalinclude:: ../../docs/source/example-code/src/main/kotlin/net/corda/docs/kotlin/FlowCookbook.kt
       :language: kotlin
       :start-after: DOCSTART 41
       :end-before: DOCEND 41
       :dedent: 8

    .. literalinclude:: ../../docs/source/example-code/src/main/java/net/corda/docs/java/FlowCookbook.java
       :language: java
       :start-after: DOCSTART 41
       :end-before: DOCEND 41
       :dedent: 12

公正 和 记录
^^^^^^^^^^^^^^^^^^^^^^^^
Notarising and recording a transaction is handled by a built-in flow called ``FinalityFlow``. See :doc:`api-flows` for
more details.

公正和记录一个 transaction 是由一个内建的名为 ``FinalityFlow`` 的 flow 来处理的。查看 :doc:`api-flows` 了解详细信息。